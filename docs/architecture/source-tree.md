# 12. Unified Project Structure

## 12.1 Turborepo Monorepo Overview

ProCare uses a **Turborepo monorepo** to manage all frontend apps, backend services, and shared packages in a single repository. This enables:

- **Shared TypeScript types** across the entire stack
- **Code reusability** with shared packages
- **Atomic commits** across multiple apps/services
- **Unified versioning** and dependency management
- **Efficient builds** with caching and parallel execution

## 12.2 Complete File Tree

```
procare/
├── .bmad-core/                      # BMAD Core configuration
│   ├── core-config.yaml
│   ├── tasks/
│   └── templates/
│
├── apps/
│   ├── mobile/                      # React Native mobile app (Expo)
│   │   ├── app/                     # Expo Router (file-based routing)
│   │   │   ├── (auth)/
│   │   │   │   ├── login.tsx
│   │   │   │   ├── verify-otp.tsx
│   │   │   │   └── onboarding.tsx
│   │   │   ├── (tabs)/              # Bottom tab navigator
│   │   │   │   ├── _layout.tsx
│   │   │   │   ├── index.tsx        # Home (Today screen)
│   │   │   │   ├── chat.tsx         # AI Chat
│   │   │   │   ├── insights.tsx     # Health Insights
│   │   │   │   └── profile.tsx      # Profile & Settings
│   │   │   ├── glucose/
│   │   │   │   ├── log.tsx          # Log glucose reading
│   │   │   │   ├── history.tsx      # Glucose history
│   │   │   │   └── [id].tsx         # Reading details
│   │   │   ├── meals/
│   │   │   │   ├── log.tsx          # Log meal with photo
│   │   │   │   ├── history.tsx
│   │   │   │   └── [id].tsx
│   │   │   ├── medications/
│   │   │   │   ├── list.tsx
│   │   │   │   └── [id].tsx
│   │   │   ├── lifestyle/
│   │   │   │   ├── activity.tsx
│   │   │   │   ├── sleep.tsx
│   │   │   │   └── stress.tsx
│   │   │   ├── health-records/
│   │   │   │   ├── index.tsx
│   │   │   │   ├── upload.tsx
│   │   │   │   └── [id].tsx
│   │   │   ├── referral/
│   │   │   │   ├── index.tsx        # Doctor referral flow
│   │   │   │   └── success.tsx
│   │   │   ├── trial/
│   │   │   │   ├── index.tsx        # 14-day trial info
│   │   │   │   └── upgrade.tsx      # Subscription upgrade
│   │   │   ├── _layout.tsx          # Root layout
│   │   │   └── +not-found.tsx
│   │   ├── src/
│   │   │   ├── components/          # React Native components
│   │   │   │   ├── ui/              # Reusable UI components
│   │   │   │   │   ├── Button.tsx
│   │   │   │   │   ├── Input.tsx
│   │   │   │   │   ├── Card.tsx
│   │   │   │   │   └── ...
│   │   │   │   ├── GlucoseChart.tsx
│   │   │   │   ├── MealCard.tsx
│   │   │   │   ├── MedicationReminder.tsx
│   │   │   │   └── AIInsightCard.tsx
│   │   │   ├── hooks/               # Custom React hooks
│   │   │   │   ├── useOfflineSync.ts
│   │   │   │   ├── useCamera.ts
│   │   │   │   ├── useNotifications.ts
│   │   │   │   └── useAuth.ts
│   │   │   ├── store/               # Zustand stores
│   │   │   │   ├── authStore.ts
│   │   │   │   ├── syncStore.ts
│   │   │   │   └── settingsStore.ts
│   │   │   ├── lib/                 # Utilities
│   │   │   │   ├── trpc.ts          # tRPC client
│   │   │   │   ├── watermelon.ts    # WatermelonDB setup
│   │   │   │   └── notifications.ts # Expo Notifications
│   │   │   └── models/              # WatermelonDB models
│   │   │       ├── GlucoseReading.ts
│   │   │       ├── Meal.ts
│   │   │       ├── Medication.ts
│   │   │       └── schema.ts
│   │   ├── assets/
│   │   │   ├── images/
│   │   │   └── fonts/
│   │   ├── app.json                 # Expo config
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tailwind.config.js       # NativeWind config
│   │
│   ├── doctor-dashboard/            # Next.js doctor dashboard
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   │   ├── login/
│   │   │   │   │   └── page.tsx
│   │   │   │   └── layout.tsx
│   │   │   ├── (dashboard)/
│   │   │   │   ├── layout.tsx       # Dashboard shell with sidebar
│   │   │   │   ├── page.tsx         # Overview/stats
│   │   │   │   ├── patients/
│   │   │   │   │   ├── page.tsx     # Patient list
│   │   │   │   │   └── [id]/
│   │   │   │   │       ├── page.tsx # Patient details
│   │   │   │   │       ├── glucose/
│   │   │   │   │       ├── meals/
│   │   │   │   │       ├── medications/
│   │   │   │   │       └── health-records/
│   │   │   │   ├── pooled-questions/
│   │   │   │   │   └── page.tsx     # Answer pooled questions
│   │   │   │   ├── analytics/
│   │   │   │   │   └── page.tsx     # Patient analytics
│   │   │   │   └── settings/
│   │   │   │       └── page.tsx
│   │   │   ├── api/
│   │   │   │   └── auth/
│   │   │   │       └── [...nextauth]/
│   │   │   │           └── route.ts # NextAuth.js config
│   │   │   ├── layout.tsx           # Root layout
│   │   │   └── page.tsx             # Landing page
│   │   ├── src/
│   │   │   ├── components/          # React components
│   │   │   │   ├── ui/              # ShadCN components
│   │   │   │   ├── charts/
│   │   │   │   │   ├── GlucoseTrendChart.tsx
│   │   │   │   │   └── PatientStatsChart.tsx
│   │   │   │   ├── PatientCard.tsx
│   │   │   │   ├── EmergencyAlertCard.tsx
│   │   │   │   └── PooledQuestionCard.tsx
│   │   │   ├── lib/
│   │   │   │   ├── trpc.ts          # tRPC client (SSR-compatible)
│   │   │   │   └── utils.ts
│   │   │   └── hooks/
│   │   │       └── usePatientData.ts
│   │   ├── public/
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   ├── tailwind.config.ts
│   │   └── next.config.js
│   │
│   ├── caregiver-dashboard/         # Next.js caregiver dashboard
│   │   ├── app/
│   │   │   ├── (auth)/
│   │   │   │   └── login/
│   │   │   │       └── page.tsx
│   │   │   ├── (dashboard)/
│   │   │   │   ├── layout.tsx
│   │   │   │   ├── page.tsx         # Patient overview
│   │   │   │   ├── patient/
│   │   │   │   │   ├── [id]/
│   │   │   │   │   │   ├── page.tsx
│   │   │   │   │   │   ├── glucose/
│   │   │   │   │   │   ├── meals/
│   │   │   │   │   │   └── medications/
│   │   │   │   ├── insights/
│   │   │   │   │   └── page.tsx     # AI insights summary
│   │   │   │   └── settings/
│   │   │   │       └── page.tsx
│   │   │   └── layout.tsx
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── ui/
│   │   │   │   ├── PatientOverviewCard.tsx
│   │   │   │   └── AlertsWidget.tsx
│   │   │   └── lib/
│   │   │       └── trpc.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tailwind.config.ts
│   │
│   ├── api-gateway/                 # Express.js + tRPC API Gateway
│   │   ├── src/
│   │   │   ├── index.ts             # Express server entry point
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts          # JWT verification
│   │   │   │   ├── rateLimit.ts     # Redis-based rate limiting
│   │   │   │   └── errorHandler.ts
│   │   │   ├── routes/
│   │   │   │   ├── health.ts        # Health check endpoint
│   │   │   │   └── webhooks.ts      # Evolution API, Razorpay webhooks
│   │   │   ├── trpc/
│   │   │   │   ├── index.ts         # tRPC app router
│   │   │   │   ├── context.ts       # Request context
│   │   │   │   └── routers/
│   │   │   │       ├── auth.ts      # OTP send/verify
│   │   │   │       ├── glucose.ts   # Glucose CRUD
│   │   │   │       ├── meals.ts
│   │   │   │       ├── medications.ts
│   │   │   │       ├── lifestyle.ts
│   │   │   │       ├── healthRecords.ts
│   │   │   │       ├── chat.ts      # AI chat sessions
│   │   │   │       ├── insights.ts  # Health insights
│   │   │   │       ├── sync.ts      # Mobile offline sync
│   │   │   │       ├── doctor.ts    # Doctor endpoints
│   │   │   │       ├── caregiver.ts
│   │   │   │       ├── subscription.ts
│   │   │   │       └── referral.ts
│   │   │   └── lib/
│   │   │       ├── redis.ts         # Redis client
│   │   │       └── jwt.ts           # JWT utils
│   │   ├── Dockerfile
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── agent-services/              # 7 AI microservices
│       ├── engagement-engine/       # Central orchestrator
│       │   ├── src/
│       │   │   ├── index.ts         # Service entry point
│       │   │   ├── orchestrator.ts  # Event routing logic
│       │   │   ├── handlers/
│       │   │   │   ├── patientRegistered.ts
│       │   │   │   ├── glucoseLogged.ts
│       │   │   │   ├── mealLogged.ts
│       │   │   │   ├── emergencyDetected.ts
│       │   │   │   └── agentResponse.ts
│       │   │   ├── rateLimiter.ts   # 5 notifications/day logic
│       │   │   └── jobs/
│       │   │       └── medicationReminders.ts # Cron job
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       ├── clinical-monitor/        # Emergency glucose detection
│       │   ├── src/
│       │   │   ├── index.ts
│       │   │   ├── handlers/
│       │   │   │   ├── glucoseCheck.ts      # <70 or >300 detection
│       │   │   │   └── followUpCheck.ts     # 15-minute follow-up
│       │   │   └── lib/
│       │   │       └── generateInstructions.ts # GPT-4 AI instructions
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       ├── health-insights/         # Pattern detection & insights
│       │   ├── src/
│       │   │   ├── index.ts
│       │   │   ├── handlers/
│       │   │   │   ├── analyzePatterns.ts   # FR5: Meal-glucose patterns
│       │   │   │   └── generateInsight.ts   # Weekly insights
│       │   │   └── lib/
│       │   │       ├── statistics.ts        # Correlation analysis
│       │   │       └── preMealWarning.ts    # FR6: Pre-meal warnings
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       ├── lifestyle-coach/         # Diet & lifestyle guidance
│       │   ├── src/
│       │   │   ├── index.ts
│       │   │   ├── handlers/
│       │   │   │   ├── mealAnalysis.ts      # GPT-4 Vision food recognition
│       │   │   │   ├── activityGuidance.ts
│       │   │   │   └── sleepAnalysis.ts
│       │   │   └── lib/
│       │   │       └── foodDatabase.ts      # Indian food nutrition DB
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       ├── pooled-questions/        # Question aggregation & routing
│       │   ├── src/
│       │   │   ├── index.ts
│       │   │   ├── handlers/
│       │   │   │   ├── questionReceived.ts  # Store and analyze similarity
│       │   │   │   ├── routeQuestion.ts     # AI vs doctor routing
│       │   │   │   └── answerDistribution.ts # Send answer to all askers
│       │   │   └── lib/
│       │   │       └── similarity.ts        # Question similarity matching
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       ├── trial-manager/           # 14-day trial & subscription
│       │   ├── src/
│       │   │   ├── index.ts
│       │   │   ├── handlers/
│       │   │   │   ├── trialStarted.ts      # Initialize 14-day countdown
│       │   │   │   ├── trialReminder.ts     # Day 7, 12, 14 reminders
│       │   │   │   └── subscriptionCreated.ts
│       │   │   └── jobs/
│       │   │       └── checkTrials.ts       # Daily cron job
│       │   ├── Dockerfile
│       │   ├── package.json
│       │   └── tsconfig.json
│       │
│       └── revenue-optimizer/       # Revenue sharing & doctor payouts
│           ├── src/
│           │   ├── index.ts
│           │   ├── handlers/
│           │   │   ├── subscriptionCreated.ts # Calculate 40% doctor share
│           │   │   ├── pooledQuestionAnswered.ts # ₹50 per answer
│           │   │   └── monthlyPayout.ts     # Generate payout reports
│           │   └── lib/
│           │       └── razorpayPayout.ts    # Razorpay X payout API
│           ├── Dockerfile
│           ├── package.json
│           └── tsconfig.json
│
├── packages/
│   ├── database/                    # Prisma ORM (shared by all services)
│   │   ├── prisma/
│   │   │   ├── schema.prisma        # Complete database schema
│   │   │   ├── migrations/
│   │   │   └── seed.ts              # Database seeding
│   │   ├── src/
│   │   │   └── index.ts             # Re-export Prisma client
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── shared/                      # Shared utilities & types
│   │   ├── src/
│   │   │   ├── types/
│   │   │   │   ├── api.ts           # tRPC types
│   │   │   │   ├── models.ts        # Domain models
│   │   │   │   └── events.ts        # RabbitMQ event types
│   │   │   ├── lib/
│   │   │   │   ├── rabbitmq.ts      # RabbitMQ client
│   │   │   │   ├── validation.ts    # Zod schemas
│   │   │   │   └── utils.ts         # Common utilities
│   │   │   └── constants/
│   │   │       ├── glucoseStates.ts
│   │   │       ├── managementModes.ts
│   │   │       └── notifications.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── ui/                          # Web UI components (ShadCN)
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── button.tsx
│   │   │   │   ├── input.tsx
│   │   │   │   ├── card.tsx
│   │   │   │   ├── dialog.tsx
│   │   │   │   ├── dropdown-menu.tsx
│   │   │   │   ├── table.tsx
│   │   │   │   └── ...              # All ShadCN components
│   │   │   └── lib/
│   │   │       └── utils.ts         # cn() utility
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tailwind.config.js
│   │
│   ├── ui-native/                   # React Native UI components
│   │   ├── src/
│   │   │   ├── components/
│   │   │   │   ├── Button.tsx
│   │   │   │   ├── Input.tsx
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── BottomSheet.tsx
│   │   │   │   └── ...              # React Native Reusables components
│   │   │   └── lib/
│   │   │       └── utils.ts
│   │   ├── package.json
│   │   ├── tsconfig.json
│   │   └── tailwind.config.js       # NativeWind config
│   │
│   ├── openrouter-client/           # OpenRouter API client
│   │   ├── src/
│   │   │   ├── index.ts             # Main client
│   │   │   ├── types.ts             # Request/response types
│   │   │   └── models.ts            # Model selection logic
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   ├── whatsapp-client/             # Evolution API client
│   │   ├── src/
│   │   │   ├── index.ts             # Main client
│   │   │   ├── sendMessage.ts
│   │   │   ├── sendMedia.ts
│   │   │   └── types.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── config/                      # Shared configurations
│       ├── eslint/
│       │   ├── base.js
│       │   ├── next.js
│       │   └── react-native.js
│       ├── typescript/
│       │   ├── base.json
│       │   ├── next.json
│       │   └── react-native.json
│       └── package.json
│
├── docs/                            # Documentation
│   ├── prd.md                       # Product Requirements Document
│   ├── architecture.md              # This document
│   ├── api.md                       # API documentation
│   └── deployment.md                # Deployment guide
│
├── .github/
│   └── workflows/
│       ├── ci.yml                   # Continuous Integration
│       └── deploy.yml               # Continuous Deployment
│
├── docker-compose.yml               # Local development environment
├── Dockerfile                       # Production Docker image
├── turbo.json                       # Turborepo configuration
├── package.json                     # Root package.json
├── pnpm-workspace.yaml              # PNPM workspace config
├── .gitignore
├── .env.example                     # Environment variables template
└── README.md
```

## 12.3 Key Configuration Files

### 12.3.1 Root `package.json`

```json
{
  "name": "procare",
  "version": "1.0.0",
  "private": true,
  "workspaces": [
    "apps/*",
    "apps/agent-services/*",
    "packages/*"
  ],
  "scripts": {
    "dev": "turbo run dev",
    "dev:mobile": "turbo run dev --filter=mobile",
    "dev:api": "turbo run dev --filter=api-gateway",
    "dev:agents": "turbo run dev --filter=./apps/agent-services/*",
    "build": "turbo run build",
    "test": "turbo run test",
    "lint": "turbo run lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "db:migrate": "pnpm --filter=database prisma migrate dev",
    "db:generate": "pnpm --filter=database prisma generate",
    "db:seed": "pnpm --filter=database prisma db seed",
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down",
    "clean": "turbo run clean && rm -rf node_modules"
  },
  "devDependencies": {
    "@turbo/gen": "^2.3.0",
    "prettier": "^3.4.2",
    "turbo": "^2.3.0",
    "typescript": "^5.7.2"
  },
  "packageManager": "pnpm@9.15.0",
  "engines": {
    "node": ">=20.0.0"
  }
}
```

### 12.3.2 `turbo.json`

```json
{
  "$schema": "https://turbo.build/schema.json",
  "globalDependencies": ["**/.env.*local"],
  "pipeline": {
    "build": {
      "dependsOn": ["^build"],
      "outputs": [".next/**", "!.next/cache/**", "dist/**"]
    },
    "dev": {
      "cache": false,
      "persistent": true
    },
    "test": {
      "dependsOn": ["^build"],
      "outputs": ["coverage/**"]
    },
    "lint": {
      "dependsOn": ["^build"]
    },
    "clean": {
      "cache": false
    }
  }
}
```

### 12.3.3 `pnpm-workspace.yaml`

```yaml
packages:
  - 'apps/*'
  - 'apps/agent-services/*'
  - 'packages/*'
```

### 12.3.4 `.env.example`

```bash
# Database
DATABASE_URL="postgresql://postgres:postgres@localhost:5432/procare"

# Redis
REDIS_URL="redis://localhost:6379"

# RabbitMQ
RABBITMQ_URL="amqp://procare:procare@localhost:5672"

# MinIO (S3-compatible storage)
MINIO_ENDPOINT="localhost"
MINIO_PORT="9000"
MINIO_ACCESS_KEY="procare"
MINIO_SECRET_KEY="procare123"
MINIO_USE_SSL="false"

# JWT
JWT_SECRET="your-secret-key-change-in-production"
JWT_EXPIRES_IN="7d"

# OpenRouter API
OPENROUTER_API_KEY="sk-or-v1-..."
OPENROUTER_APP_NAME="ProCare"
OPENROUTER_APP_URL="https://procare.health"

# Evolution API (WhatsApp)
EVOLUTION_API_URL="http://localhost:8080"
EVOLUTION_API_KEY="your-evolution-api-key"
EVOLUTION_INSTANCE_NAME="procare"

# Razorpay
RAZORPAY_KEY_ID="rzp_test_..."
RAZORPAY_KEY_SECRET="..."

# BunnyCDN
BUNNYCDN_STORAGE_ZONE="procare"
BUNNYCDN_API_KEY="..."
BUNNYCDN_HOSTNAME="procare.b-cdn.net"

# NextAuth.js (for doctor/caregiver dashboards)
NEXTAUTH_URL="http://localhost:3001"
NEXTAUTH_SECRET="your-nextauth-secret"

# Sentry (optional)
SENTRY_DSN="https://..."

# Environment
NODE_ENV="development"
```

## 12.4 Package Dependencies

### 12.4.1 Mobile App (`apps/mobile/package.json`)

```json
{
  "name": "mobile",
  "version": "1.0.0",
  "main": "expo-router/entry",
  "scripts": {
    "dev": "expo start",
    "android": "expo start --android",
    "ios": "expo start --ios",
    "build:android": "eas build --platform android",
    "build:ios": "eas build --platform ios"
  },
  "dependencies": {
    "expo": "^51.0.0",
    "expo-router": "^3.5.0",
    "react-native": "0.74.0",
    "react-native-reusables": "^1.0.0",
    "nativewind": "^4.0.0",
    "@tanstack/react-query": "^5.62.0",
    "@trpc/client": "^10.45.0",
    "@trpc/react-query": "^10.45.0",
    "@watermelondb/react-native": "^0.27.0",
    "zustand": "^4.5.0",
    "expo-camera": "^15.0.0",
    "expo-notifications": "^0.28.0",
    "expo-secure-store": "^13.0.0",
    "react-native-chart-kit": "^6.12.0",
    "database": "workspace:*",
    "shared": "workspace:*",
    "ui-native": "workspace:*"
  },
  "devDependencies": {
    "@types/react": "^18.3.0",
    "typescript": "^5.7.2"
  }
}
```

### 12.4.2 API Gateway (`apps/api-gateway/package.json`)

```json
{
  "name": "api-gateway",
  "version": "1.0.0",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "express": "^4.21.0",
    "helmet": "^8.0.0",
    "cors": "^2.8.5",
    "@trpc/server": "^10.45.0",
    "jsonwebtoken": "^9.0.2",
    "bcryptjs": "^2.4.3",
    "redis": "^4.7.0",
    "database": "workspace:*",
    "shared": "workspace:*",
    "openrouter-client": "workspace:*",
    "whatsapp-client": "workspace:*"
  },
  "devDependencies": {
    "@types/express": "^4.17.21",
    "@types/node": "^22.10.0",
    "tsx": "^4.19.0",
    "typescript": "^5.7.2"
  }
}
```

### 12.4.3 Agent Service (Generic) (`apps/agent-services/*/package.json`)

```json
{
  "name": "engagement-engine",
  "version": "1.0.0",
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "build": "tsc",
    "start": "node dist/index.js"
  },
  "dependencies": {
    "amqplib": "^0.10.4",
    "node-cron": "^3.0.3",
    "database": "workspace:*",
    "shared": "workspace:*",
    "openrouter-client": "workspace:*",
    "whatsapp-client": "workspace:*"
  },
  "devDependencies": {
    "@types/amqplib": "^0.10.5",
    "@types/node": "^22.10.0",
    "@types/node-cron": "^3.0.11",
    "tsx": "^4.19.0",
    "typescript": "^5.7.2"
  }
}
```

### 12.4.4 Shared Package (`packages/shared/package.json`)

```json
{
  "name": "shared",
  "version": "1.0.0",
  "main": "./src/index.ts",
  "types": "./src/index.ts",
  "dependencies": {
    "amqplib": "^0.10.4",
    "zod": "^3.23.0"
  },
  "devDependencies": {
    "@types/amqplib": "^0.10.5",
    "typescript": "^5.7.2"
  }
}
```

## 12.5 TypeScript Configuration

### 12.5.1 Root `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "commonjs",
    "lib": ["ES2022"],
    "moduleResolution": "node",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "strict": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 12.5.2 Next.js App `tsconfig.json`

```json
{
  "extends": "@procare/config/typescript/next.json",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 12.5.3 React Native App `tsconfig.json`

```json
{
  "extends": "expo/tsconfig.base",
  "compilerOptions": {
    "strict": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["**/*.ts", "**/*.tsx", ".expo/types/**/*.ts", "expo-env.d.ts"]
}
```

## 12.6 Build & Deployment Artifacts

When building for production, the following artifacts are generated:

### Apps
- **mobile**: `.apk` (Android), `.ipa` (iOS) via EAS Build
- **doctor-dashboard**: `.next/` build folder (static export)
- **caregiver-dashboard**: `.next/` build folder
- **api-gateway**: `dist/` compiled JavaScript
- **agent-services/***: `dist/` compiled JavaScript for each service

### Docker Images
- `procare/api-gateway:latest`
- `procare/engagement-engine:latest`
- `procare/clinical-monitor:latest`
- `procare/health-insights:latest`
- `procare/lifestyle-coach:latest`
- `procare/pooled-questions:latest`
- `procare/trial-manager:latest`
- `procare/revenue-optimizer:latest`

All Docker images are built using multi-stage builds to minimize final image size (<100MB per service).
