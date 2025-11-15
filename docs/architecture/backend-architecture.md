# Backend Architecture

This section defines the backend architecture for ProCare's microservices-based event-driven system, including the API Gateway, 7-agent orchestration, and supporting services.

## Service Architecture Overview

ProCare's backend consists of:
1. **API Gateway** - Single entry point for all client requests
2. **tRPC Server** - Type-safe API layer
3. **7 AI Agent Services** - Specialized microservices for healthcare workflows
4. **RabbitMQ** - Event bus for inter-service communication
5. **Redis** - Caching and session management
6. **PostgreSQL** - Primary data store

## API Gateway

**Responsibility:** Entry point for all HTTP requests, handles authentication, rate limiting, routing, and request/response logging.

### Folder Structure

```
apps/api-gateway/
├── src/
│   ├── index.ts              # Express app entry point
│   ├── server.ts             # HTTP server setup
│   ├── middleware/           # Express middleware
│   │   ├── auth.ts          # JWT authentication
│   │   ├── rateLimit.ts     # Rate limiting
│   │   ├── cors.ts          # CORS configuration
│   │   ├── logging.ts       # Request logging
│   │   └── errorHandler.ts  # Global error handler
│   ├── routes/              # REST endpoints
│   │   ├── health.ts        # Health check
│   │   └── webhooks/        # Webhook handlers
│   │       ├── evolution.ts # WhatsApp webhooks
│   │       └── razorpay.ts  # Payment webhooks
│   ├── trpc/                # tRPC setup
│   │   ├── context.ts       # Request context
│   │   ├── router.ts        # Root router
│   │   └── routers/         # Feature routers
│   │       ├── auth.ts
│   │       ├── patient.ts
│   │       ├── glucose.ts
│   │       ├── doctor.ts
│   │       └── ...
│   ├── lib/                 # Utilities
│   │   ├── prisma.ts        # Prisma client
│   │   ├── redis.ts         # Redis client
│   │   ├── rabbitmq.ts      # RabbitMQ publisher
│   │   └── logger.ts        # Winston logger
│   └── types/               # TypeScript types
│       └── express.d.ts
├── Dockerfile
├── package.json
└── tsconfig.json
```

### Implementation

**Express Server Setup:**

```typescript
// apps/api-gateway/src/index.ts
import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import { createExpressMiddleware } from '@trpc/server/adapters/express';
import { appRouter } from './trpc/router';
import { createContext } from './trpc/context';
import { authMiddleware } from './middleware/auth';
import { rateLimitMiddleware } from './middleware/rateLimit';
import { loggingMiddleware } from './middleware/logging';
import { errorHandler } from './middleware/errorHandler';
import healthRoutes from './routes/health';
import webhookRoutes from './routes/webhooks';
import { logger } from './lib/logger';

const app = express();
const PORT = process.env.PORT || 3000;

// Security middleware
app.use(helmet());
app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  credentials: true,
}));

// Request parsing
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));

// Logging
app.use(loggingMiddleware);

// Health check (no auth required)
app.use('/health', healthRoutes);

// Webhooks (verify signatures, not JWT)
app.use('/webhooks', webhookRoutes);

// tRPC endpoint (with auth)
app.use(
  '/trpc',
  authMiddleware, // JWT verification
  rateLimitMiddleware, // Rate limiting
  createExpressMiddleware({
    router: appRouter,
    createContext,
  })
);

// Global error handler
app.use(errorHandler);

app.listen(PORT, () => {
  logger.info(`API Gateway listening on port ${PORT}`);
});
```

**Authentication Middleware:**

```typescript
// apps/api-gateway/src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';
import { prisma } from '../lib/prisma';

export interface AuthRequest extends Request {
  user?: {
    id: string;
    role: 'PATIENT' | 'DOCTOR' | 'CAREGIVER';
  };
}

export async function authMiddleware(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  const authHeader = req.headers.authorization;

  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as {
      userId: string;
      role: string;
    };

    // Check if token is blacklisted (logout)
    const isBlacklisted = await redis.get(`blacklist:${token}`);
    if (isBlacklisted) {
      return res.status(401).json({ error: 'Token has been revoked' });
    }

    // Verify user still exists and is active
    const user = await prisma.user.findUnique({
      where: { id: decoded.userId },
      select: { id: true, role: true, isActive: true },
    });

    if (!user || !user.isActive) {
      return res.status(401).json({ error: 'User not found or inactive' });
    }

    req.user = user;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}
```

**Rate Limiting Middleware:**

```typescript
// apps/api-gateway/src/middleware/rateLimit.ts
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';
import { redis } from '../lib/redis';

export const rateLimitMiddleware = rateLimit({
  store: new RedisStore({
    client: redis,
    prefix: 'rate-limit:',
  }),
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Limit each user to 100 requests per window
  keyGenerator: (req: AuthRequest) => req.user?.id || req.ip,
  handler: (req, res) => {
    res.status(429).json({
      error: 'Too many requests, please try again later.',
    });
  },
});
```

**tRPC Context:**

```typescript
// apps/api-gateway/src/trpc/context.ts
import { CreateExpressContextOptions } from '@trpc/server/adapters/express';
import { AuthRequest } from '../middleware/auth';
import { prisma } from '../lib/prisma';
import { redis } from '../lib/redis';
import { publishEvent } from '../lib/rabbitmq';

export function createContext({ req }: CreateExpressContextOptions) {
  const authReq = req as AuthRequest;

  return {
    user: authReq.user,
    prisma,
    redis,
    publishEvent, // For emitting events to RabbitMQ
  };
}

export type Context = Awaited<ReturnType<typeof createContext>>;
```

**tRPC Router:**

```typescript
// apps/api-gateway/src/trpc/router.ts
import { initTRPC, TRPCError } from '@trpc/server';
import { Context } from './context';
import { authRouter } from './routers/auth';
import { patientRouter } from './routers/patient';
import { glucoseRouter } from './routers/glucose';
import { doctorRouter } from './routers/doctor';

const t = initTRPC.context<Context>().create();

// Middleware to check authentication
const isAuthed = t.middleware(({ ctx, next }) => {
  if (!ctx.user) {
    throw new TRPCError({ code: 'UNAUTHORIZED' });
  }
  return next({ ctx: { ...ctx, user: ctx.user } });
});

// Protected procedure (requires authentication)
export const protectedProcedure = t.procedure.use(isAuthed);
export const publicProcedure = t.procedure;
export const router = t.router;

// Root router
export const appRouter = router({
  auth: authRouter,
  patient: patientRouter,
  glucose: glucoseRouter,
  medication: medicationRouter,
  meal: mealRouter,
  healthRecord: healthRecordRouter,
  doctor: doctorRouter,
  caregiver: caregiverRouter,
  appointment: appointmentRouter,
  ai: aiRouter,
  pattern: patternRouter,
  notification: notificationRouter,
  sync: syncRouter,
});

export type AppRouter = typeof appRouter;
```

**Example Router (Glucose):**

```typescript
// apps/api-gateway/src/trpc/routers/glucose.ts
import { z } from 'zod';
import { router, protectedProcedure } from '../router';
import { TRPCError } from '@trpc/server';

export const glucoseRouter = router({
  logReading: protectedProcedure
    .input(z.object({
      value: z.number().min(40).max(600),
      state: z.enum([
        'MORNING_FASTING', 'AFTER_BREAKFAST', 'BEFORE_LUNCH',
        'AFTER_LUNCH', 'BEFORE_DINNER', 'AFTER_DINNER', 'BEDTIME', 'RANDOM'
      ]),
      recordedAt: z.date(),
      source: z.enum(['MANUAL', 'OCR_PHOTO', 'BLUETOOTH', 'VOICE']),
      photoUrl: z.string().optional(),
      notes: z.string().optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify user is a patient
      const patient = await ctx.prisma.patient.findUnique({
        where: { userId: ctx.user.id },
      });

      if (!patient) {
        throw new TRPCError({
          code: 'FORBIDDEN',
          message: 'Only patients can log glucose'
        });
      }

      // Store reading in database
      const reading = await ctx.prisma.glucoseReading.create({
        data: {
          patientId: patient.id,
          value: input.value,
          state: input.state,
          recordedAt: input.recordedAt,
          source: input.source,
          photoUrl: input.photoUrl,
          notes: input.notes,
          isSynced: true,
        },
      });

      // Check for emergency
      const isEmergency = input.value < 70 || input.value > 300;

      if (isEmergency) {
        // Emit event to Clinical Monitor agent
        await ctx.publishEvent('clinical.glucose.check', {
          patientId: patient.id,
          reading: {
            id: reading.id,
            value: input.value,
            state: input.state,
            recordedAt: input.recordedAt,
          },
          type: input.value < 70 ? 'HYPOGLYCEMIA' : 'HYPERGLYCEMIA',
        });
      } else {
        // Still emit event for tracking
        await ctx.publishEvent('patient.glucose.logged', {
          patientId: patient.id,
          reading,
        });
      }

      return { reading, isEmergency };
    }),

  getReadings: protectedProcedure
    .input(z.object({
      startDate: z.date().optional(),
      endDate: z.date().optional(),
      limit: z.number().default(50),
      offset: z.number().default(0),
    }))
    .query(async ({ input, ctx }) => {
      const patient = await ctx.prisma.patient.findUnique({
        where: { userId: ctx.user.id },
      });

      if (!patient) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      const readings = await ctx.prisma.glucoseReading.findMany({
        where: {
          patientId: patient.id,
          recordedAt: {
            gte: input.startDate,
            lte: input.endDate,
          },
        },
        orderBy: { recordedAt: 'desc' },
        take: input.limit,
        skip: input.offset,
      });

      const total = await ctx.prisma.glucoseReading.count({
        where: { patientId: patient.id },
      });

      return { readings, total };
    }),
});
```

## Microservices Architecture (7 Agents)

**Common Service Structure:**

```
apps/agent-services/[agent-name]/
├── src/
│   ├── index.ts              # Service entry point
│   ├── handlers/             # Event handlers
│   │   └── [event-name].ts
│   ├── services/             # Business logic
│   │   └── [feature].ts
│   ├── lib/                  # Utilities
│   │   ├── openrouter.ts    # OpenRouter client
│   │   ├── prisma.ts
│   │   └── logger.ts
│   └── types/
├── Dockerfile
├── package.json
└── tsconfig.json
```

### Engagement Engine (Orchestrator)

**Responsibility:** Central orchestrator, routes events, synthesizes responses, enforces rate limiting.

```typescript
// apps/agent-services/engagement-engine/src/index.ts
import { consumeEvents, publishEvent } from './lib/rabbitmq';
import { handlePatientEvent } from './handlers/patientEvent';
import { handleAgentResponse } from './handlers/agentResponse';
import { logger } from './lib/logger';

async function main() {
  logger.info('Starting Engagement Engine...');

  // Listen for patient events
  await consumeEvents('patient.*', async (event) => {
    await handlePatientEvent(event);
  });

  // Listen for agent responses
  await consumeEvents('agent.*.response', async (event) => {
    await handleAgentResponse(event);
  });

  logger.info('Engagement Engine ready');
}

main().catch((error) => {
  logger.error('Failed to start Engagement Engine', error);
  process.exit(1);
});
```

**Event Handler:**

```typescript
// apps/agent-services/engagement-engine/src/handlers/patientEvent.ts
import { publishEvent } from '../lib/rabbitmq';
import { checkRateLimit } from '../services/rateLimit';
import { routeToAgents } from '../services/router';
import { logger } from '../lib/logger';

export async function handlePatientEvent(event: any) {
  const { type, patientId, data } = event;

  logger.info(`Received patient event: ${type}`, { patientId });

  // Route event to appropriate agents
  const targetAgents = await routeToAgents(type, data);

  for (const agent of targetAgents) {
    await publishEvent(`${agent}.${type}`, {
      patientId,
      data,
      timestamp: Date.now(),
    });
  }
}
```

**Rate Limiting Service:**

```typescript
// apps/agent-services/engagement-engine/src/services/rateLimit.ts
import { redis } from '../lib/redis';
import { prisma } from '../lib/prisma';

const MAX_NOTIFICATIONS_PER_DAY = 5;

export async function checkRateLimit(patientId: string): Promise<boolean> {
  const today = new Date().toISOString().split('T')[0];
  const key = `notifications:${patientId}:${today}`;

  const count = await redis.get(key);
  const currentCount = count ? parseInt(count, 10) : 0;

  if (currentCount >= MAX_NOTIFICATIONS_PER_DAY) {
    return false; // Rate limit exceeded
  }

  return true;
}

export async function incrementNotificationCount(patientId: string): Promise<void> {
  const today = new Date().toISOString().split('T')[0];
  const key = `notifications:${patientId}:${today}`;

  await redis.incr(key);
  await redis.expire(key, 86400); // Expire after 24 hours
}
```

### Clinical Monitor Agent

**Responsibility:** Emergency detection, medication tracking, immediate safety instructions.

```typescript
// apps/agent-services/clinical-monitor/src/handlers/glucoseCheck.ts
import { publishEvent } from '../lib/rabbitmq';
import { generateEmergencyInstructions } from '../services/emergencyResponse';
import { prisma } from '../lib/prisma';
import { logger } from '../lib/logger';

export async function handleGlucoseCheck(event: any) {
  const { patientId, reading, type } = event;

  logger.info(`Glucose check: ${type}`, { patientId, value: reading.value });

  // Create emergency event
  const emergencyEvent = await prisma.emergencyEvent.create({
    data: {
      patientId,
      type,
      glucoseValue: reading.value,
      status: 'ACTIVE',
      instructions: '', // Will be updated with AI instructions
    },
  });

  // Generate AI instructions
  const instructions = await generateEmergencyInstructions(
    type,
    reading.value,
    patientId
  );

  // Update emergency event with instructions
  await prisma.emergencyEvent.update({
    where: { id: emergencyEvent.id },
    data: { instructions },
  });

  // Emit emergency event
  await publishEvent('patient.emergency.detected', {
    patientId,
    emergencyId: emergencyEvent.id,
    type,
    glucoseValue: reading.value,
    instructions,
  });

  // Start 15-minute follow-up timer
  setTimeout(async () => {
    await checkFollowUp(emergencyEvent.id, patientId);
  }, 15 * 60 * 1000);
}

async function checkFollowUp(emergencyId: string, patientId: string) {
  const emergency = await prisma.emergencyEvent.findUnique({
    where: { id: emergencyId },
    include: {
      patient: {
        include: {
          glucoseReadings: {
            where: {
              recordedAt: {
                gte: new Date(Date.now() - 15 * 60 * 1000),
              },
            },
            orderBy: { recordedAt: 'desc' },
            take: 1,
          },
        },
      },
    },
  });

  if (!emergency) return;

  if (emergency.status !== 'ACTIVE') {
    // Already resolved
    return;
  }

  const latestReading = emergency.patient.glucoseReadings[0];

  if (!latestReading) {
    // No new reading - escalate
    await publishEvent('patient.emergency.escalation', {
      patientId,
      emergencyId,
      reason: 'NO_FOLLOW_UP_READING',
    });
  } else if (latestReading.value < 70 || latestReading.value > 300) {
    // Still abnormal - escalate
    await publishEvent('patient.emergency.persistent', {
      patientId,
      emergencyId,
      latestValue: latestReading.value,
    });
  } else {
    // Resolved
    await prisma.emergencyEvent.update({
      where: { id: emergencyId },
      data: { status: 'RESOLVED', resolvedAt: new Date() },
    });

    await publishEvent('patient.emergency.resolved', {
      patientId,
      emergencyId,
    });
  }
}
```

**Emergency Instructions Generation:**

```typescript
// apps/agent-services/clinical-monitor/src/services/emergencyResponse.ts
import { openrouterClient } from '../lib/openrouter';

export async function generateEmergencyInstructions(
  type: 'HYPOGLYCEMIA' | 'HYPERGLYCEMIA',
  glucoseValue: number,
  patientId: string
): Promise<string> {
  const prompt = type === 'HYPOGLYCEMIA'
    ? `Patient has hypoglycemia with blood glucose ${glucoseValue} mg/dL (below 70).
       Provide immediate safety instructions in 3-4 bullet points.
       Be clear, concise, and actionable.`
    : `Patient has hyperglycemia with blood glucose ${glucoseValue} mg/dL (above 300).
       Provide immediate safety instructions in 3-4 bullet points.
       Be clear, concise, and actionable.`;

  const response = await openrouterClient.chat.completions.create({
    model: 'openai/gpt-4-turbo',
    messages: [
      {
        role: 'system',
        content: 'You are a diabetes care assistant providing emergency safety instructions.',
      },
      {
        role: 'user',
        content: prompt,
      },
    ],
    max_tokens: 200,
  });

  return response.choices[0]?.message?.content || 'Please contact your doctor immediately.';
}
```

### Health Insights Agent

**Responsibility:** Pattern detection, pre-meal warnings, glucose predictions.

```typescript
// apps/agent-services/health-insights/src/handlers/patternUpdate.ts
import { analyzePatterns } from '../services/patternAnalysis';
import { publishEvent } from '../lib/rabbitmq';
import { prisma } from '../lib/prisma';

export async function handlePatternUpdate(event: any) {
  const { patientId } = event;

  // Check if patient has enough data (4+ weeks)
  const fourWeeksAgo = new Date(Date.now() - 28 * 24 * 60 * 60 * 1000);

  const mealCount = await prisma.meal.count({
    where: {
      patientId,
      consumedAt: { gte: fourWeeksAgo },
    },
  });

  if (mealCount < 8) {
    // Not enough data yet
    return;
  }

  // Analyze food-glucose patterns
  const patterns = await analyzePatterns(patientId);

  for (const pattern of patterns) {
    if (pattern.confidence > 0.7 && pattern.sampleSize >= 8) {
      // Pattern is significant - save it
      await prisma.pattern.upsert({
        where: {
          patientId_foodItem: {
            patientId,
            foodItem: pattern.foodItem,
          },
        },
        create: {
          patientId,
          type: 'FOOD_GLUCOSE_CORRELATION',
          foodItem: pattern.foodItem,
          avgGlucoseIncrease: pattern.avgIncrease,
          betterAlternative: pattern.alternative,
          confidence: pattern.confidence,
          sampleSize: pattern.sampleSize,
          detectedAt: new Date(),
        },
        update: {
          avgGlucoseIncrease: pattern.avgIncrease,
          betterAlternative: pattern.alternative,
          confidence: pattern.confidence,
          sampleSize: pattern.sampleSize,
          isActive: true,
        },
      });

      // Emit pattern detected event
      await publishEvent('patient.pattern.detected', {
        patientId,
        pattern,
      });
    }
  }
}
```

**Pattern Analysis:**

```typescript
// apps/agent-services/health-insights/src/services/patternAnalysis.ts
import { prisma } from '../lib/prisma';

interface FoodPattern {
  foodItem: string;
  avgIncrease: number;
  alternative: string;
  confidence: number;
  sampleSize: number;
}

export async function analyzePatterns(patientId: string): Promise<FoodPattern[]> {
  // Get meals with before/after glucose readings
  const meals = await prisma.meal.findMany({
    where: {
      patientId,
      glucoseImpact: { not: null },
      recognizedFoods: { isEmpty: false },
    },
    include: {
      glucoseReadingBefore: true,
      glucoseReadingAfter: true,
    },
  });

  // Group by food item
  const foodGroups = new Map<string, number[]>();

  for (const meal of meals) {
    for (const food of meal.recognizedFoods) {
      if (!foodGroups.has(food)) {
        foodGroups.set(food, []);
      }
      foodGroups.get(food)!.push(meal.glucoseImpact!);
    }
  }

  // Calculate patterns
  const patterns: FoodPattern[] = [];

  for (const [food, impacts] of foodGroups.entries()) {
    if (impacts.length < 3) continue; // Need at least 3 samples

    const avgIncrease = impacts.reduce((a, b) => a + b, 0) / impacts.length;
    const stdDev = Math.sqrt(
      impacts.reduce((sum, val) => sum + Math.pow(val - avgIncrease, 2), 0) / impacts.length
    );

    // Confidence based on sample size and consistency
    const confidence = Math.min(
      (impacts.length / 15) * (1 - stdDev / avgIncrease),
      1.0
    );

    patterns.push({
      foodItem: food,
      avgIncrease,
      alternative: await findAlternative(food),
      confidence,
      sampleSize: impacts.length,
    });
  }

  return patterns;
}

async function findAlternative(foodItem: string): Promise<string> {
  // Hardcoded alternatives for common Indian foods
  const alternatives: Record<string, string> = {
    'white rice': 'brown rice',
    'potato curry': 'green beans',
    'naan': 'roti',
    'white bread': 'whole wheat bread',
  };

  return alternatives[foodItem.toLowerCase()] || 'healthier alternative';
}
```

### Lifestyle Coach Agent

**Responsibility:** Meal analysis, food recognition, dietary recommendations.

```typescript
// apps/agent-services/lifestyle-coach/src/handlers/mealLogged.ts
import { analyzeMealPhoto } from '../services/foodRecognition';
import { publishEvent } from '../lib/rabbitmq';
import { prisma } from '../lib/prisma';

export async function handleMealLogged(event: any) {
  const { patientId, mealId, photoUrl } = event;

  if (!photoUrl) return; // No photo to analyze

  // Analyze meal photo with GPT-4 Vision
  const analysis = await analyzeMealPhoto(photoUrl);

  // Update meal with recognized foods
  await prisma.meal.update({
    where: { id: mealId },
    data: {
      recognizedFoods: analysis.foods,
      estimatedCalories: analysis.calories,
    },
  });

  // Check for active patterns
  const patterns = await prisma.pattern.findMany({
    where: {
      patientId,
      isActive: true,
      foodItem: { in: analysis.foods },
    },
  });

  if (patterns.length > 0) {
    // Emit pre-meal warning
    await publishEvent('patient.warning.pre_meal', {
      patientId,
      mealId,
      patterns,
    });
  }
}
```

**Food Recognition:**

```typescript
// apps/agent-services/lifestyle-coach/src/services/foodRecognition.ts
import { openrouterClient } from '../lib/openrouter';

interface FoodAnalysis {
  foods: string[];
  calories: number;
}

export async function analyzeMealPhoto(photoUrl: string): Promise<FoodAnalysis> {
  const response = await openrouterClient.chat.completions.create({
    model: 'openai/gpt-4-vision-preview',
    messages: [
      {
        role: 'system',
        content: 'You are a nutrition expert specializing in Indian cuisine. Identify food items and estimate calories.',
      },
      {
        role: 'user',
        content: [
          {
            type: 'text',
            text: 'Identify all food items in this image. Return a JSON object with:\n1. foods: array of food names\n2. calories: total estimated calories',
          },
          {
            type: 'image_url',
            image_url: { url: photoUrl },
          },
        ],
      },
    ],
    response_format: { type: 'json_object' },
  });

  const result = JSON.parse(response.choices[0]?.message?.content || '{}');

  return {
    foods: result.foods || [],
    calories: result.calories || 0,
  };
}
```

## Background Job Processing

**Cron Jobs for Scheduled Tasks:**

```typescript
// apps/agent-services/care-coordinator/src/jobs/medicationReminders.ts
import cron from 'node-cron';
import { prisma } from '../lib/prisma';
import { publishEvent } from '../lib/rabbitmq';
import { logger } from '../lib/logger';

// Run every minute to check for medication reminders
cron.schedule('* * * * *', async () => {
  const now = new Date();
  const currentTime = `${now.getHours().toString().padStart(2, '0')}:${now.getMinutes().toString().padStart(2, '0')}`;

  // Find medications scheduled for this time
  const medications = await prisma.medication.findMany({
    where: {
      isActive: true,
      scheduledTimes: { has: currentTime },
    },
    include: { patient: true },
  });

  for (const medication of medications) {
    // Create medication log
    const log = await prisma.medicationLog.create({
      data: {
        medicationId: medication.id,
        patientId: medication.patientId,
        scheduledTime: now,
        status: 'PENDING',
        loggedBy: medication.patientId, // Self-logging
      },
    });

    // Emit reminder event
    await publishEvent('care.medication.reminder', {
      patientId: medication.patientId,
      medicationId: medication.id,
      logId: log.id,
      medication: {
        name: medication.name,
        dosage: medication.dosage,
      },
    });

    logger.info(`Medication reminder sent`, {
      patientId: medication.patientId,
      medication: medication.name,
    });
  }
});
```

## Service Communication Patterns

**RabbitMQ Event Publisher:**

```typescript
// packages/shared/src/lib/rabbitmq.ts
import amqp from 'amqplib';
import { logger } from './logger';

let connection: amqp.Connection;
let channel: amqp.Channel;

export async function connectRabbitMQ() {
  connection = await amqp.connect(process.env.RABBITMQ_URL!);
  channel = await connection.createChannel();

  // Declare exchange for event routing
  await channel.assertExchange('procare-events', 'topic', { durable: true });

  logger.info('Connected to RabbitMQ');
}

export async function publishEvent(routingKey: string, data: any) {
  if (!channel) {
    throw new Error('RabbitMQ not connected');
  }

  const message = JSON.stringify({
    ...data,
    publishedAt: Date.now(),
  });

  channel.publish('procare-events', routingKey, Buffer.from(message), {
    persistent: true,
  });

  logger.debug(`Published event: ${routingKey}`, data);
}

export async function consumeEvents(
  pattern: string,
  handler: (event: any) => Promise<void>
) {
  if (!channel) {
    throw new Error('RabbitMQ not connected');
  }

  const queue = await channel.assertQueue('', { exclusive: true });

  await channel.bindQueue(queue.queue, 'procare-events', pattern);

  channel.consume(queue.queue, async (msg) => {
    if (!msg) return;

    try {
      const event = JSON.parse(msg.content.toString());
      await handler(event);
      channel.ack(msg);
    } catch (error) {
      logger.error('Event handler error', error);
      // Requeue or send to dead letter queue
      channel.nack(msg, false, false);
    }
  });

  logger.info(`Consuming events: ${pattern}`);
}
```

## External Service Integration

**OpenRouter Client:**

```typescript
// packages/shared/src/lib/openrouter.ts
import OpenAI from 'openai';

export const openrouterClient = new OpenAI({
  baseURL: 'https://openrouter.ai/api/v1',
  apiKey: process.env.OPENROUTER_API_KEY,
  defaultHeaders: {
    'HTTP-Referer': 'https://procare.health',
    'X-Title': 'ProCare Diabetes Management',
  },
});

// Model selection based on task
export const MODELS = {
  ROUTINE: 'meta-llama/llama-3.1-70b-instruct', // $0.52/M tokens
  MEDICAL: 'openai/gpt-4-turbo', // $10/M tokens
  PATTERN: 'anthropic/claude-3.5-sonnet', // $3/M tokens
  OCR: 'openai/gpt-4-vision-preview', // $10/M tokens
  CLASSIFICATION: 'meta-llama/llama-3.1-8b-instruct', // $0.06/M tokens
};
```

**Evolution API Client:**

```typescript
// packages/shared/src/lib/evolutionApi.ts
import axios from 'axios';

const evolutionClient = axios.create({
  baseURL: process.env.EVOLUTION_API_URL,
  headers: {
    'apikey': process.env.EVOLUTION_API_KEY,
  },
});

export async function sendWhatsAppMessage(
  phoneNumber: string,
  message: string
) {
  await evolutionClient.post(`/message/sendText/${process.env.EVOLUTION_INSTANCE}`, {
    number: phoneNumber,
    text: message,
  });
}

export async function sendWhatsAppMedia(
  phoneNumber: string,
  mediaUrl: string,
  caption?: string
) {
  await evolutionClient.post(`/message/sendMedia/${process.env.EVOLUTION_INSTANCE}`, {
    number: phoneNumber,
    mediaUrl,
    caption,
  });
}
```

## Docker Configuration

**API Gateway Dockerfile:**

```dockerfile
# apps/api-gateway/Dockerfile
FROM node:20-alpine AS base

# Install dependencies
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# Build application
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

# Production image
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 expressjs

COPY --from=builder --chown=expressjs:nodejs /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./package.json

USER expressjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "dist/index.js"]
```

**Docker Compose for Local Development:**

```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: timescale/timescaledb:latest-pg16
    environment:
      POSTGRES_DB: procare
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data

  rabbitmq:
    image: rabbitmq:3.13-management-alpine
    environment:
      RABBITMQ_DEFAULT_USER: procare
      RABBITMQ_DEFAULT_PASS: procare
    ports:
      - "5672:5672"   # AMQP
      - "15672:15672" # Management UI
    volumes:
      - rabbitmq-data:/var/lib/rabbitmq

  minio:
    image: minio/minio:latest
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: procare
      MINIO_ROOT_PASSWORD: procare123
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio-data:/data

  api-gateway:
    build:
      context: ./apps/api-gateway
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/procare
      REDIS_URL: redis://redis:6379
      RABBITMQ_URL: amqp://procare:procare@rabbitmq:5672
      JWT_SECRET: your-secret-key
      PORT: 3000
    ports:
      - "3000:3000"
    depends_on:
      - postgres
      - redis
      - rabbitmq

  engagement-engine:
    build:
      context: ./apps/agent-services/engagement-engine
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/procare
      REDIS_URL: redis://redis:6379
      RABBITMQ_URL: amqp://procare:procare@rabbitmq:5672
    depends_on:
      - postgres
      - redis
      - rabbitmq

  clinical-monitor:
    build:
      context: ./apps/agent-services/clinical-monitor
      dockerfile: Dockerfile
    environment:
      DATABASE_URL: postgresql://postgres:postgres@postgres:5432/procare
      RABBITMQ_URL: amqp://procare:procare@rabbitmq:5672
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
    depends_on:
      - postgres
      - rabbitmq

volumes:
  postgres-data:
  redis-data:
  rabbitmq-data:
  minio-data:
```
