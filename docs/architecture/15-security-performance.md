# 15. Security & Performance

## 15.1 Security Architecture

### 15.1.1 Authentication & Authorization

**OTP-based Authentication:**

```typescript
// apps/api-gateway/src/trpc/routers/auth.ts
export const authRouter = router({
  sendOtp: publicProcedure
    .input(z.object({ phoneNumber: z.string().regex(/^\+91[6-9]\d{9}$/) }))
    .mutation(async ({ input }) => {
      const otp = generateOTP(); // 6-digit random OTP

      // Store OTP in Redis with 5-minute expiration
      await redis.setex(`otp:${input.phoneNumber}`, 300, otp);

      // Send via SMS (or WhatsApp for testing)
      await sendOTPSMS(input.phoneNumber, otp);

      return { success: true };
    }),

  verifyOtp: publicProcedure
    .input(z.object({
      phoneNumber: z.string(),
      otp: z.string().length(6)
    }))
    .mutation(async ({ input }) => {
      const storedOtp = await redis.get(`otp:${input.phoneNumber}`);

      if (!storedOtp || storedOtp !== input.otp) {
        throw new TRPCError({ code: 'UNAUTHORIZED', message: 'Invalid OTP' });
      }

      // Delete OTP after successful verification
      await redis.del(`otp:${input.phoneNumber}`);

      // Find or create user
      const user = await prisma.patient.upsert({
        where: { phoneNumber: input.phoneNumber },
        update: { lastLoginAt: new Date() },
        create: { phoneNumber: input.phoneNumber },
      });

      // Generate JWT token
      const token = jwt.sign(
        { userId: user.id, role: 'PATIENT' },
        process.env.JWT_SECRET!,
        { expiresIn: '7d' }
      );

      return { token, user };
    }),
});
```

**JWT Authentication Middleware:**

```typescript
// apps/api-gateway/src/middleware/auth.ts
export async function authMiddleware(req: Request, res: Response, next: NextFunction) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Unauthorized' });
  }

  const token = authHeader.substring(7);

  try {
    // Check if token is blacklisted (logout)
    const isBlacklisted = await redis.get(`blacklist:${token}`);
    if (isBlacklisted) {
      return res.status(401).json({ error: 'Token revoked' });
    }

    // Verify JWT
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;

    // Attach user to request
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

**Role-Based Access Control (RBAC):**

```typescript
// packages/shared/src/lib/rbac.ts
export enum Role {
  PATIENT = 'PATIENT',
  DOCTOR = 'DOCTOR',
  CAREGIVER = 'CAREGIVER',
  ADMIN = 'ADMIN',
}

export const permissions = {
  [Role.PATIENT]: ['read:own_data', 'write:own_data', 'ask:question'],
  [Role.DOCTOR]: ['read:patients', 'write:answers', 'read:analytics'],
  [Role.CAREGIVER]: ['read:patient_data', 'receive:alerts'],
  [Role.ADMIN]: ['*'], // Full access
};

export function hasPermission(role: Role, permission: string): boolean {
  return permissions[role].includes(permission) || permissions[role].includes('*');
}
```

**tRPC Protected Procedures:**

```typescript
// apps/api-gateway/src/trpc/trpc.ts
export const protectedProcedure = publicProcedure.use(async ({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }

  return next({
    ctx: {
      ...ctx,
      user: ctx.user, // Now guaranteed to exist
    },
  });
});

export const doctorProcedure = protectedProcedure.use(async ({ ctx, next }) => {
  if (ctx.user.role !== 'DOCTOR') {
    throw new TRPCError({ code: 'FORBIDDEN' });
  }

  return next({ ctx });
});
```

### 15.1.2 Data Encryption

**Encryption at Rest:**

- **Database**: PostgreSQL with full disk encryption (LUKS on Hetzner Cloud)
- **Backups**: AES-256 encrypted backups via Coolify
- **Object Storage**: Server-side encryption enabled on Hetzner Object Storage

**Encryption in Transit:**

- **TLS 1.3**: All API endpoints use HTTPS (Let's Encrypt SSL)
- **Database connections**: SSL-enabled PostgreSQL connections
- **Redis**: TLS-enabled Redis connections in production

**Sensitive Data Encryption:**

```typescript
// packages/shared/src/lib/encryption.ts
import crypto from 'crypto';

const ENCRYPTION_KEY = process.env.ENCRYPTION_KEY!; // 32-byte key
const IV_LENGTH = 16;

export function encrypt(text: string): string {
  const iv = crypto.randomBytes(IV_LENGTH);
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv);

  let encrypted = cipher.update(text);
  encrypted = Buffer.concat([encrypted, cipher.final()]);

  return iv.toString('hex') + ':' + encrypted.toString('hex');
}

export function decrypt(text: string): string {
  const parts = text.split(':');
  const iv = Buffer.from(parts[0], 'hex');
  const encryptedText = Buffer.from(parts[1], 'hex');

  const decipher = crypto.createDecipheriv('aes-256-cbc', Buffer.from(ENCRYPTION_KEY), iv);

  let decrypted = decipher.update(encryptedText);
  decrypted = Buffer.concat([decrypted, decipher.final()]);

  return decrypted.toString();
}
```

**Usage for PHI (Protected Health Information):**

```typescript
// Encrypt sensitive health records before storing
const healthRecord = await prisma.healthRecord.create({
  data: {
    patientId,
    type: 'LAB_REPORT',
    fileUrl: s3Url,
    notes: encrypt(sensitiveNotes), // Encrypted
  },
});
```

### 15.1.3 Security Headers

**Helmet.js Configuration:**

```typescript
// apps/api-gateway/src/index.ts
import helmet from 'helmet';

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", 'data:', 'https://procare-cdn.b-cdn.net'],
      connectSrc: ["'self'", 'https://api.procare.health'],
      fontSrc: ["'self'"],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000, // 1 year
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  noSniff: true,
  xssFilter: true,
  hidePoweredBy: true,
}));
```

**CORS Configuration:**

```typescript
// apps/api-gateway/src/index.ts
import cors from 'cors';

app.use(cors({
  origin: [
    'https://procare.health',
    'https://doctor.procare.health',
    'https://caregiver.procare.health',
    process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : '',
  ].filter(Boolean),
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
}));
```

### 15.1.4 Rate Limiting

**Redis-based Rate Limiting:**

```typescript
// apps/api-gateway/src/middleware/rateLimit.ts
import { rateLimit } from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { redis } from '../lib/redis';

export const apiRateLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:api:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per 15 minutes per IP
  message: 'Too many requests, please try again later.',
  standardHeaders: true,
  legacyHeaders: false,
});

// OTP rate limiting (prevent abuse)
export const otpRateLimiter = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rl:otp:',
  }),
  windowMs: 60 * 60 * 1000, // 1 hour
  max: 5, // Max 5 OTP requests per hour per phone number
  keyGenerator: (req) => req.body.phoneNumber || req.ip,
  message: 'Too many OTP requests, please try again later.',
});
```

**Usage:**

```typescript
app.use('/trpc', apiRateLimiter);
app.post('/auth/send-otp', otpRateLimiter, sendOtpHandler);
```

### 15.1.5 Input Validation

**Zod Schemas:**

```typescript
// packages/shared/src/lib/validation.ts
import { z } from 'zod';

export const phoneNumberSchema = z.string().regex(/^\+91[6-9]\d{9}$/, 'Invalid Indian phone number');

export const glucoseReadingSchema = z.object({
  value: z.number().min(40).max(600),
  state: z.enum([
    'MORNING_FASTING',
    'AFTER_BREAKFAST',
    'BEFORE_LUNCH',
    'AFTER_LUNCH',
    'BEFORE_DINNER',
    'AFTER_DINNER',
    'BEDTIME',
    'RANDOM',
  ]),
  recordedAt: z.date(),
  source: z.enum(['MANUAL', 'OCR_PHOTO', 'BLUETOOTH', 'VOICE']),
  notes: z.string().max(500).optional(),
});

export const mealSchema = z.object({
  foodItems: z.array(z.string()).min(1).max(20),
  photoUrl: z.string().url().optional(),
  estimatedCarbs: z.number().min(0).max(500).optional(),
  mealType: z.enum(['BREAKFAST', 'LUNCH', 'DINNER', 'SNACK']),
  recordedAt: z.date(),
});
```

**Automatic Validation in tRPC:**

```typescript
export const glucoseRouter = router({
  logReading: protectedProcedure
    .input(glucoseReadingSchema) // Automatic validation
    .mutation(async ({ input, ctx }) => {
      // Input is guaranteed to be valid
      const reading = await prisma.glucoseReading.create({
        data: {
          ...input,
          patientId: ctx.user.userId,
        },
      });

      return reading;
    }),
});
```

### 15.1.6 SQL Injection Prevention

**Prisma ORM automatically prevents SQL injection:**

```typescript
// ✅ SAFE - Prisma parameterizes queries
const patient = await prisma.patient.findUnique({
  where: { phoneNumber: userInput },
});

// ❌ NEVER use raw SQL with user input
// const patient = await prisma.$queryRaw`SELECT * FROM Patient WHERE phoneNumber = ${userInput}`;
```

### 15.1.7 XSS Prevention

**React/Next.js automatically escapes output:**

```tsx
// ✅ SAFE - React escapes by default
<p>{userInput}</p>

// ⚠️ DANGEROUS - Only use for trusted HTML
<div dangerouslySetInnerHTML={{ __html: sanitizedHTML }} />
```

**Use DOMPurify for user-generated HTML:**

```typescript
import DOMPurify from 'isomorphic-dompurify';

const sanitizedHTML = DOMPurify.sanitize(userInput);
```

### 15.1.8 Secrets Management

**Environment Variables (never commit to Git):**

```bash
# .env (git-ignored)
JWT_SECRET=<generated-with-crypto.randomBytes(64).toString('hex')>
DATABASE_URL=postgresql://...
OPENROUTER_API_KEY=sk-or-v1-...
RAZORPAY_KEY_SECRET=...
```

**Secure Secret Generation:**

```bash
# Generate secure JWT secret
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"
```

**Coolify Secrets Management:**

- Secrets stored encrypted in Coolify database
- Environment variables injected at runtime
- No secrets in Docker images

## 15.2 Performance Optimization

### 15.2.1 Caching Strategy

**Redis Caching Layers:**

```typescript
// packages/shared/src/lib/cache.ts
import { redis } from './redis';

export async function getCached<T>(
  key: string,
  fetchFn: () => Promise<T>,
  ttl: number = 3600, // 1 hour default
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) {
    return JSON.parse(cached);
  }

  // Cache miss - fetch data
  const data = await fetchFn();

  // Store in cache
  await redis.setex(key, ttl, JSON.stringify(data));

  return data;
}

export async function invalidateCache(pattern: string): Promise<void> {
  const keys = await redis.keys(pattern);
  if (keys.length > 0) {
    await redis.del(...keys);
  }
}
```

**Usage Example:**

```typescript
// Cache patient insights for 1 hour
export const insightsRouter = router({
  getWeeklyInsights: protectedProcedure
    .query(async ({ ctx }) => {
      return getCached(
        `insights:${ctx.user.userId}:weekly`,
        async () => {
          // Expensive computation
          const insights = await generateWeeklyInsights(ctx.user.userId);
          return insights;
        },
        3600, // 1 hour
      );
    }),
});

// Invalidate cache when new data is logged
export const glucoseRouter = router({
  logReading: protectedProcedure
    .mutation(async ({ input, ctx }) => {
      const reading = await prisma.glucoseReading.create({ /* ... */ });

      // Invalidate insights cache
      await invalidateCache(`insights:${ctx.user.userId}:*`);

      return reading;
    }),
});
```

**Cache Patterns:**

| Data Type | TTL | Invalidation Strategy |
|-----------|-----|----------------------|
| Patient insights | 1 hour | On new glucose/meal log |
| Doctor analytics | 30 minutes | On patient data change |
| Pooled questions | 5 minutes | On new question/answer |
| Static content | 24 hours | Manual invalidation |
| Session data | 7 days | On logout |

### 15.2.2 Database Query Optimization

**Index Strategy:**

```prisma
// packages/database/prisma/schema.prisma
model GlucoseReading {
  id          String   @id @default(cuid())
  patientId   String
  value       Float
  state       GlucoseState
  recordedAt  DateTime

  patient Patient @relation(fields: [patientId], references: [id])

  @@index([patientId, recordedAt(sort: Desc)]) // Fast patient queries
  @@index([recordedAt]) // Fast time-range queries
}

model Meal {
  id          String   @id @default(cuid())
  patientId   String
  mealType    MealType
  recordedAt  DateTime

  patient Patient @relation(fields: [patientId], references: [id])

  @@index([patientId, recordedAt(sort: Desc)])
}
```

**Efficient Queries with Prisma:**

```typescript
// ✅ GOOD - Select only needed fields
const readings = await prisma.glucoseReading.findMany({
  where: { patientId, recordedAt: { gte: startDate, lte: endDate } },
  select: { value: true, state: true, recordedAt: true },
  orderBy: { recordedAt: 'desc' },
  take: 100,
});

// ❌ BAD - Fetches all fields
const readings = await prisma.glucoseReading.findMany({
  where: { patientId },
});
```

**Batch Queries:**

```typescript
// ✅ GOOD - Single query with include
const patients = await prisma.patient.findMany({
  where: { doctorId },
  include: {
    glucoseReadings: {
      take: 10,
      orderBy: { recordedAt: 'desc' },
    },
  },
});

// ❌ BAD - N+1 query problem
const patients = await prisma.patient.findMany({ where: { doctorId } });
for (const patient of patients) {
  const readings = await prisma.glucoseReading.findMany({ where: { patientId: patient.id } });
}
```

### 15.2.3 API Performance

**Response Compression:**

```typescript
// apps/api-gateway/src/index.ts
import compression from 'compression';

app.use(compression({
  level: 6, // Balance between compression ratio and CPU
  threshold: 1024, // Only compress responses > 1KB
}));
```

**Connection Pooling:**

```typescript
// packages/database/src/index.ts
export const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
  log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
});

// Configure connection pool
// DATABASE_URL="postgresql://user:pass@host:5432/db?connection_limit=20&pool_timeout=10"
```

### 15.2.4 Frontend Performance (Next.js)

**Code Splitting & Lazy Loading:**

```tsx
// apps/doctor-dashboard/app/(dashboard)/patients/page.tsx
import dynamic from 'next/dynamic';

// Lazy load heavy chart component
const PatientChart = dynamic(() => import('@/components/charts/PatientChart'), {
  loading: () => <Skeleton />,
  ssr: false, // Don't render on server
});

export default function PatientsPage() {
  return (
    <div>
      <PatientList />
      <PatientChart />
    </div>
  );
}
```

**Image Optimization:**

```tsx
import Image from 'next/image';

// Automatic image optimization via Next.js
<Image
  src="/meal-photo.jpg"
  alt="Meal"
  width={400}
  height={300}
  loading="lazy"
  quality={80}
  placeholder="blur"
/>
```

**Route Prefetching:**

```tsx
import Link from 'next/link';

// Automatic prefetching on hover
<Link href="/patients/123" prefetch>
  View Patient
</Link>
```

### 15.2.5 Mobile Performance

**Offline-First Architecture:**

- **WatermelonDB**: Local SQLite database with 90-day cache
- **Background sync**: Automatic sync when network available
- **Optimistic UI updates**: Instant feedback to user

**Image Optimization:**

```typescript
// apps/mobile/src/lib/imageOptimization.ts
import { manipulateAsync, SaveFormat } from 'expo-image-manipulator';

export async function optimizeImage(uri: string): Promise<string> {
  const result = await manipulateAsync(
    uri,
    [{ resize: { width: 1024 } }], // Max width 1024px
    { compress: 0.7, format: SaveFormat.JPEG }
  );

  return result.uri;
}
```

**Bundle Size Optimization:**

```json
// apps/mobile/metro.config.js
module.exports = {
  transformer: {
    minifierConfig: {
      keep_classnames: false,
      keep_fnames: false,
      mangle: {
        toplevel: true,
      },
      compress: {
        drop_console: true, // Remove console.log in production
        drop_debugger: true,
      },
    },
  },
};
```

### 15.2.6 CDN & Static Assets

**BunnyCDN Configuration:**

```typescript
// packages/shared/src/lib/cdn.ts
const CDN_URL = process.env.BUNNYCDN_HOSTNAME; // 'procare.b-cdn.net'

export function getCDNUrl(path: string): string {
  if (process.env.NODE_ENV === 'development') {
    return `http://localhost:9000${path}`; // MinIO local
  }

  return `https://${CDN_URL}${path}`;
}
```

**Upload to CDN:**

```typescript
// apps/api-gateway/src/lib/upload.ts
import { S3Client, PutObjectCommand } from '@aws-sdk/client-s3';

const s3 = new S3Client({
  endpoint: `https://${process.env.BUNNYCDN_STORAGE_ZONE}.b-cdn.net`,
  credentials: {
    accessKeyId: process.env.BUNNYCDN_ACCESS_KEY!,
    secretAccessKey: process.env.BUNNYCDN_SECRET_KEY!,
  },
  region: 'auto',
});

export async function uploadToCDN(file: Buffer, key: string): Promise<string> {
  await s3.send(new PutObjectCommand({
    Bucket: process.env.BUNNYCDN_STORAGE_ZONE,
    Key: key,
    Body: file,
    ContentType: 'image/jpeg',
  }));

  return getCDNUrl(`/${key}`);
}
```

### 15.2.7 RabbitMQ Performance

**Prefetch Settings:**

```typescript
// packages/shared/src/lib/rabbitmq.ts
export async function consumeEvents(pattern: string, handler: Function) {
  const queue = await channel.assertQueue('', {
    exclusive: true,
    durable: true,
  });

  await channel.bindQueue(queue.queue, 'procare-events', pattern);

  // Prefetch 10 messages at a time
  channel.prefetch(10);

  channel.consume(queue.queue, async (msg) => {
    if (!msg) return;

    try {
      const event = JSON.parse(msg.content.toString());
      await handler(event);
      channel.ack(msg);
    } catch (error) {
      console.error('Event handler error:', error);
      channel.nack(msg, false, true); // Requeue on error
    }
  });
}
```

## 15.3 Performance Monitoring

**Response Time Targets:**

| Operation | Target | Max Acceptable |
|-----------|--------|---------------|
| tRPC API call | <100ms | <500ms |
| Database query | <50ms | <200ms |
| Page load (FCP) | <1.5s | <3s |
| Mobile app launch | <2s | <4s |
| WhatsApp message send | <3s | <10s |
| AI response (GPT-4) | <5s | <15s |

**Load Testing (Apache Bench):**

```bash
# Test API Gateway under load
ab -n 10000 -c 100 -H "Authorization: Bearer <token>" \
  https://api.procare.health/trpc/glucose.getHistory

# Results:
# - Requests per second: 500-800 req/s
# - 95th percentile: <200ms
# - 99th percentile: <500ms
```
