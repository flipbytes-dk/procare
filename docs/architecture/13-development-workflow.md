# 13. Development Workflow

## 13.1 Prerequisites

Before setting up the ProCare development environment, ensure you have the following installed:

### Required
- **Node.js 20 LTS** or higher
- **pnpm 9.15.0** (package manager)
- **Docker Desktop** (for local infrastructure)
- **Git** (version control)

### Optional (for mobile development)
- **Expo CLI** (installed globally: `npm install -g expo-cli`)
- **Android Studio** (for Android development)
- **Xcode** (for iOS development, macOS only)
- **EAS CLI** (for building: `npm install -g eas-cli`)

### Verify Installation

```bash
node --version    # Should be v20.x.x or higher
pnpm --version    # Should be 9.15.0
docker --version  # Should be 20.x.x or higher
git --version     # Should be 2.x.x or higher
```

## 13.2 Initial Setup

### 13.2.1 Clone Repository

```bash
git clone https://github.com/your-org/procare.git
cd procare
```

### 13.2.2 Install Dependencies

```bash
# Install all workspace dependencies (root + all apps + packages)
pnpm install

# This will install dependencies for:
# - Root workspace
# - All apps (mobile, doctor-dashboard, caregiver-dashboard, api-gateway, agent services)
# - All packages (database, shared, ui, ui-native, etc.)
```

### 13.2.3 Environment Configuration

Create `.env` file in the root directory:

```bash
cp .env.example .env
```

Edit `.env` and configure the following critical variables:

```bash
# Database (will be auto-configured by docker-compose)
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/procare"

# Redis
REDIS_URL="redis://localhost:6379"

# RabbitMQ
RABBITMQ_URL="amqp://procare:procare@localhost:5672"

# JWT (CHANGE THIS!)
JWT_SECRET="your-super-secret-jwt-key-change-me"
JWT_EXPIRES_IN="7d"

# OpenRouter API (REQUIRED - get from https://openrouter.ai)
OPENROUTER_API_KEY="sk-or-v1-your-api-key-here"
OPENROUTER_APP_NAME="ProCare"
OPENROUTER_APP_URL="https://procare.health"

# Evolution API (WhatsApp) - Optional for local dev
EVOLUTION_API_URL="http://localhost:8080"
EVOLUTION_API_KEY="your-evolution-api-key"
EVOLUTION_INSTANCE_NAME="procare"

# Razorpay (use test keys for development)
RAZORPAY_KEY_ID="rzp_test_..."
RAZORPAY_KEY_SECRET="..."

# NextAuth.js
NEXTAUTH_URL="http://localhost:3001"
NEXTAUTH_SECRET="your-nextauth-secret-change-me"

# Environment
NODE_ENV="development"
```

### 13.2.4 Start Infrastructure Services

Start PostgreSQL, Redis, RabbitMQ, and MinIO using Docker Compose:

```bash
pnpm docker:up

# This starts:
# - PostgreSQL (TimescaleDB) on port 5432
# - Redis on port 6379
# - RabbitMQ on ports 5672 (AMQP) and 15672 (Management UI)
# - MinIO on ports 9000 (API) and 9001 (Console)
```

Verify services are running:

```bash
docker ps

# You should see 4 containers running:
# - procare-postgres-1
# - procare-redis-1
# - procare-rabbitmq-1
# - procare-minio-1
```

Access management UIs:
- **RabbitMQ Management**: http://localhost:15672 (username: `procare`, password: `procare`)
- **MinIO Console**: http://localhost:9001 (username: `procare`, password: `procare123`)

### 13.2.5 Database Setup

Generate Prisma client and run migrations:

```bash
# Generate Prisma client (creates TypeScript types)
pnpm db:generate

# Run database migrations
pnpm db:migrate

# Seed the database with sample data (optional)
pnpm db:seed
```

Verify database schema:

```bash
# Connect to PostgreSQL
docker exec -it procare-postgres-1 psql -U postgres -d procare

# List tables
\dt

# You should see all tables: Patient, GlucoseReading, Meal, Medication, etc.
```

## 13.3 Development Commands

### 13.3.1 Run All Apps & Services

```bash
# Start all apps and services in parallel (uses Turborepo)
pnpm dev

# This starts:
# - Mobile app (Expo Dev Server on port 8081)
# - Doctor dashboard (Next.js on port 3001)
# - Caregiver dashboard (Next.js on port 3002)
# - API Gateway (Express on port 3000)
# - All 7 agent services
```

### 13.3.2 Run Specific Apps

```bash
# Mobile app only
pnpm dev:mobile

# API Gateway only
pnpm dev:api

# All agent services only
pnpm dev:agents

# Specific Next.js dashboard
pnpm --filter=doctor-dashboard dev
pnpm --filter=caregiver-dashboard dev

# Specific agent service
pnpm --filter=engagement-engine dev
pnpm --filter=clinical-monitor dev
```

### 13.3.3 Mobile Development

**Start Expo Dev Server:**

```bash
cd apps/mobile
pnpm dev

# Or from root:
pnpm dev:mobile
```

**Run on Physical Device:**

1. Install **Expo Go** app on your phone (iOS/Android)
2. Scan QR code from terminal
3. App will load on your device

**Run on Simulator/Emulator:**

```bash
# iOS Simulator (macOS only)
cd apps/mobile
pnpm ios

# Android Emulator
cd apps/mobile
pnpm android
```

**Build for Production:**

```bash
cd apps/mobile

# Configure EAS (first time only)
eas login
eas build:configure

# Build APK (Android)
pnpm build:android

# Build IPA (iOS)
pnpm build:ios
```

### 13.3.4 Web Dashboard Development

**Doctor Dashboard:**

```bash
cd apps/doctor-dashboard
pnpm dev

# Access at: http://localhost:3001
```

**Caregiver Dashboard:**

```bash
cd apps/caregiver-dashboard
pnpm dev

# Access at: http://localhost:3002
```

### 13.3.5 Backend Development

**API Gateway:**

```bash
cd apps/api-gateway
pnpm dev

# API Gateway will be available at: http://localhost:3000
# tRPC endpoint: http://localhost:3000/trpc
# Health check: http://localhost:3000/health
```

**Test tRPC Endpoints:**

```bash
# Health check
curl http://localhost:3000/health

# Test tRPC endpoint (requires authentication)
curl -X POST http://localhost:3000/trpc/glucose.logReading \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <jwt-token>" \
  -d '{"value": 120, "state": "MORNING_FASTING", "recordedAt": "2025-01-14T08:00:00Z", "source": "MANUAL"}'
```

**Agent Services:**

```bash
# Start all agents
pnpm dev:agents

# Or start individual agents
cd apps/agent-services/engagement-engine
pnpm dev

cd apps/agent-services/clinical-monitor
pnpm dev
```

**Monitor RabbitMQ Events:**

Access RabbitMQ Management UI at http://localhost:15672 to view:
- Queue statistics
- Message rates
- Active consumers
- Published events

## 13.4 Database Workflows

### 13.4.1 Modify Database Schema

When you need to change the database schema:

1. **Edit Prisma schema:**

```bash
# Edit packages/database/prisma/schema.prisma
code packages/database/prisma/schema.prisma
```

2. **Create migration:**

```bash
pnpm db:migrate

# Prisma will:
# 1. Detect schema changes
# 2. Prompt for migration name
# 3. Generate SQL migration file
# 4. Apply migration to database
# 5. Regenerate Prisma client
```

3. **Regenerate Prisma client (if needed):**

```bash
pnpm db:generate
```

4. **Reset database (⚠️ DESTRUCTIVE):**

```bash
# This will:
# - Drop all tables
# - Re-run all migrations
# - Seed database

cd packages/database
pnpm prisma migrate reset
```

### 13.4.2 Seed Database

Add seed data in `packages/database/prisma/seed.ts`:

```typescript
// packages/database/prisma/seed.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create test patient
  const patient = await prisma.patient.create({
    data: {
      phoneNumber: '+919876543210',
      name: 'Test Patient',
      dateOfBirth: new Date('1985-05-15'),
      gender: 'MALE',
      diabetesType: 'TYPE_2',
      managementMode: 'ACTIVE_MANAGEMENT',
      onboardingCompletedAt: new Date(),
      trialEndsAt: new Date(Date.now() + 14 * 24 * 60 * 60 * 1000), // 14 days
    },
  });

  console.log('Created test patient:', patient.id);

  // Create test doctor
  const doctor = await prisma.doctor.create({
    data: {
      phoneNumber: '+919876543211',
      name: 'Dr. Test Doctor',
      email: 'doctor@test.com',
      licenseNumber: 'MCI12345',
      specialization: 'Endocrinology',
    },
  });

  console.log('Created test doctor:', doctor.id);
}

main()
  .catch((e) => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

Run seed:

```bash
pnpm db:seed
```

### 13.4.3 Database Console Access

**Using Prisma Studio (Visual Database Browser):**

```bash
cd packages/database
pnpm prisma studio

# Opens browser at http://localhost:5555
# GUI for viewing/editing database records
```

**Using psql CLI:**

```bash
docker exec -it procare-postgres-1 psql -U postgres -d procare

# Common commands:
\dt               # List all tables
\d Patient        # Describe Patient table schema
SELECT * FROM "Patient" LIMIT 10;  # Query patients
\q                # Quit
```

## 13.5 Testing Workflows

### 13.5.1 Run All Tests

```bash
# Run tests for all workspaces
pnpm test

# Run tests with coverage
pnpm test -- --coverage
```

### 13.5.2 Run Tests for Specific Apps

```bash
# API Gateway tests
pnpm --filter=api-gateway test

# Mobile app tests
pnpm --filter=mobile test

# Specific agent service tests
pnpm --filter=clinical-monitor test
```

### 13.5.3 Watch Mode (for TDD)

```bash
cd apps/api-gateway
pnpm test -- --watch
```

## 13.6 Code Quality

### 13.6.1 Linting

```bash
# Lint all workspaces
pnpm lint

# Lint specific app
pnpm --filter=api-gateway lint

# Auto-fix linting errors
pnpm lint -- --fix
```

### 13.6.2 Type Checking

```bash
# TypeScript type checking for all workspaces
pnpm turbo run type-check

# Type check specific app
cd apps/api-gateway
pnpm tsc --noEmit
```

### 13.6.3 Code Formatting

```bash
# Format all files with Prettier
pnpm format

# Check formatting without making changes
pnpm format:check
```

## 13.7 Git Workflow

### 13.7.1 Branch Naming Conventions

- **Features**: `feature/description` (e.g., `feature/glucose-logging`)
- **Bug fixes**: `fix/description` (e.g., `fix/emergency-notification`)
- **Refactoring**: `refactor/description` (e.g., `refactor/agent-orchestration`)
- **Documentation**: `docs/description` (e.g., `docs/api-endpoints`)

### 13.7.2 Commit Message Conventions

Follow **Conventional Commits** format:

```
<type>(<scope>): <description>

[optional body]

[optional footer]
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring
- `docs`: Documentation changes
- `test`: Adding/updating tests
- `chore`: Maintenance tasks

**Examples:**

```bash
git commit -m "feat(glucose): add OCR photo support for glucose readings"
git commit -m "fix(clinical-monitor): resolve 15-minute follow-up timer issue"
git commit -m "refactor(engagement-engine): optimize rate limiter Redis queries"
git commit -m "docs(architecture): add deployment section"
```

### 13.7.3 Pre-commit Hooks

ProCare uses **Husky** for Git hooks:

```bash
# Install Husky (already configured in package.json)
pnpm prepare

# Hooks run automatically:
# - pre-commit: Runs linter and type checks on staged files
# - commit-msg: Validates commit message format
```

## 13.8 Common Development Tasks

### 13.8.1 Add New tRPC Endpoint

1. **Create router file:**

```bash
# apps/api-gateway/src/trpc/routers/myRouter.ts
import { router, protectedProcedure } from '../trpc';
import { z } from 'zod';

export const myRouter = router({
  myProcedure: protectedProcedure
    .input(z.object({ foo: z.string() }))
    .mutation(async ({ input, ctx }) => {
      // Your logic here
      return { success: true };
    }),
});
```

2. **Register router in app router:**

```typescript
// apps/api-gateway/src/trpc/index.ts
import { myRouter } from './routers/myRouter';

export const appRouter = router({
  // ...existing routers
  myRouter,
});
```

3. **Use in frontend:**

```typescript
// Auto-completion and type safety!
const mutation = trpc.myRouter.myProcedure.useMutation();
await mutation.mutateAsync({ foo: 'bar' });
```

### 13.8.2 Add New Agent Service

1. **Create new agent folder:**

```bash
mkdir -p apps/agent-services/my-agent/src
cd apps/agent-services/my-agent
```

2. **Create `package.json`:**

```bash
pnpm init
# Configure similar to other agent services
```

3. **Create service entry point:**

```typescript
// apps/agent-services/my-agent/src/index.ts
import { consumeEvents } from 'shared';

async function main() {
  console.log('My Agent started');

  // Listen to events
  await consumeEvents('my.event.*', async (event) => {
    console.log('Received event:', event);
    // Handle event
  });
}

main();
```

4. **Add to docker-compose.yml** (optional for local dev)

### 13.8.3 Add New Database Model

1. **Edit Prisma schema:**

```prisma
// packages/database/prisma/schema.prisma
model MyModel {
  id        String   @id @default(cuid())
  name      String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

2. **Create migration:**

```bash
pnpm db:migrate
# Name: add_my_model
```

3. **Use in code:**

```typescript
import { prisma } from 'database';

const myModel = await prisma.myModel.create({
  data: { name: 'Test' },
});
```

### 13.8.4 Debug Backend Services

**VS Code Launch Configuration (`.vscode/launch.json`):**

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Debug API Gateway",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["--filter=api-gateway", "dev"],
      "skipFiles": ["<node_internals>/**"],
      "console": "integratedTerminal"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Debug Clinical Monitor",
      "runtimeExecutable": "pnpm",
      "runtimeArgs": ["--filter=clinical-monitor", "dev"],
      "skipFiles": ["<node_internals>/**"],
      "console": "integratedTerminal"
    }
  ]
}
```

**Add breakpoints in VS Code and press F5 to start debugging.**

## 13.9 Troubleshooting

### 13.9.1 Docker Services Not Starting

```bash
# Stop all containers
pnpm docker:down

# Remove volumes (⚠️ DELETES DATA)
docker-compose down -v

# Restart
pnpm docker:up
```

### 13.9.2 Port Already in Use

```bash
# Find process using port 3000
lsof -i :3000

# Kill process
kill -9 <PID>
```

### 13.9.3 Prisma Client Out of Sync

```bash
# Regenerate Prisma client
pnpm db:generate

# If still issues, clear node_modules
pnpm clean
pnpm install
pnpm db:generate
```

### 13.9.4 RabbitMQ Connection Issues

```bash
# Check RabbitMQ logs
docker logs procare-rabbitmq-1

# Restart RabbitMQ
docker restart procare-rabbitmq-1
```

### 13.9.5 Mobile App Not Loading

```bash
# Clear Expo cache
cd apps/mobile
pnpm expo start -c

# Clear Metro bundler cache
rm -rf node_modules/.cache
```
