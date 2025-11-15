# Components

This section defines the major logical components and services across ProCare's full-stack architecture, including frontend applications, backend microservices (7-agent orchestration), and supporting infrastructure services.

## Frontend Components

### Patient Mobile App (React Native + Expo)

**Responsibility:** Primary patient interface for glucose logging, meal tracking, medication reminders, health records, chat, and offline-first functionality.

**Key Interfaces:**
- **Screens:** Onboarding, Home (context-aware), Chat, Glucose Logging, Meal Logging, Health Records, Diet Module, Profile, Trends
- **tRPC Client:** Type-safe API calls to backend via `@trpc/react-query`
- **WatermelonDB Sync:** Background sync for offline data
- **Push Notifications:** Expo Notifications for medication reminders, alerts

**Dependencies:**
- tRPC API Gateway (for server data)
- WatermelonDB (local offline storage)
- MinIO CDN (for health record images)
- Evolution API (for WhatsApp fallback notifications)

**Technology Stack:**
- React Native 0.74+ with Expo SDK 51+
- React Native Reusables (ShadCN components) + NativeWind (TailwindCSS)
- Zustand (app state management)
- TanStack Query + tRPC (server state)
- WatermelonDB (offline database)
- Expo Camera, Expo Image Picker (OCR photo capture)
- Expo Notifications (push notifications)

### Doctor Dashboard (Next.js Web)

**Responsibility:** Doctor interface for patient triage, health record review, decision support, pooled questions, and appointment scheduling.

**Key Interfaces:**
- **Pages:** Login, Patient Buckets (Urgent/Attention/Stable/Call In), Patient Detail, Pooled Questions, Appointments, Onboarding (QR code), Settings
- **tRPC Client:** Type-safe API calls to backend
- **Real-Time Updates:** WebSocket for new alerts (optional Phase 2)

**Dependencies:**
- tRPC API Gateway
- PostgreSQL (via tRPC queries)
- MinIO CDN (for health records display)

**Technology Stack:**
- Next.js 14+ (App Router)
- ShadCN + Radix UI (component library)
- TailwindCSS (styling)
- Zustand + TanStack Query + tRPC (state management)
- Recharts (glucose trend charts)
- NextAuth.js (authentication)

### Caregiver Dashboard (Next.js Web + Mobile App)

**Responsibility:** Caregiver interface for patient monitoring, critical alerts, daily digest, proxy logging, and peace-of-mind dashboard.

**Key Interfaces:**
- **Web Pages:** Login, Patient Status, Alerts, Patient Detail, Messages, Settings
- **Mobile App:** Same functionality as web (built with React Native for on-the-go monitoring)

**Dependencies:**
- tRPC API Gateway
- PostgreSQL (via tRPC)
- Evolution API (for WhatsApp alerts)

**Technology Stack:**
- Same as Doctor Dashboard (Next.js + ShadCN + TailwindCSS)
- Mobile app uses React Native (code sharing with patient app)

## Backend Components (Microservices)

### API Gateway

**Responsibility:** Single entry point for all client requests, handles authentication, rate limiting, routing to tRPC server, and request/response logging.

**Key Interfaces:**
- **HTTP Endpoints:**
  - `POST /trpc/*` - tRPC batch endpoint
  - `GET /health` - Health check for monitoring
  - `POST /webhooks/evolution-api` - WhatsApp webhook receiver
  - `POST /webhooks/razorpay` - Payment webhook
  - `GET /uploads/*` - MinIO proxy for authenticated file access

**Dependencies:**
- tRPC Server (route requests)
- Redis (rate limiting, session storage)
- RabbitMQ (emit events to agents)
- PostgreSQL (user authentication)

**Technology Stack:**
- Node.js 20 LTS with Express.js
- TypeScript
- Helmet (security headers)
- express-rate-limit (rate limiting)
- JWT (authentication)
- Winston (logging)

### Engagement Engine (Orchestrator)

**Responsibility:** Central orchestrator for 7-agent system. Routes patient events to appropriate agents, synthesizes multi-agent responses into single messages, enforces rate limiting (FR15), and maintains patient state.

**Key Interfaces:**
- **Event Listeners (RabbitMQ):**
  - `patient.glucose.logged` → Route to Clinical Monitor, Health Insights
  - `patient.meal.logged` → Route to Lifestyle Coach
  - `patient.question.asked` → Route to Doctor Bridge or Learning Library
  - `patient.medication.missed` → Route to Care Coordinator

- **Event Emitters (RabbitMQ):**
  - `patient.notification.send` → Send via Evolution API / Push Notification
  - `doctor.alert.create` → Create alert in doctor dashboard

**Dependencies:**
- RabbitMQ (event bus)
- Redis (patient state cache, notification count tracking)
- PostgreSQL (patient data, notification logs)
- All 6 other agents (Clinical Monitor, Lifestyle Coach, etc.)

**Technology Stack:**
- Node.js 20 LTS
- TypeScript
- RabbitMQ (amqplib)
- Redis (ioredis)
- Prisma (database ORM)

### Clinical Monitor Agent

**Responsibility:** Tracks glucose, medications, detects emergencies (hypoglycemia/hyperglycemia per FR7), provides immediate instructions, and escalates to caregiver/doctor.

**Key Interfaces:**
- **Event Listeners:**
  - `clinical.glucose.check` → Check emergency thresholds
  - `clinical.medication.check` → Track medication adherence

- **Event Emitters:**
  - `patient.emergency.detected` → Trigger emergency flow
  - `caregiver.alert.send` → Notify caregiver (if management mode = shared/caregiver-primary)

**Dependencies:**
- OpenRouter API (AI-generated emergency instructions)
- PostgreSQL (patient thresholds, medication schedules)
- Redis (recent readings cache)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- Prisma ORM
- OpenRouter SDK

### Lifestyle Coach Agent

**Responsibility:** Analyzes meals, tracks activity, provides behavioral nudges, food swap recommendations (FR22), and calorie management.

**Key Interfaces:**
- **Event Listeners:**
  - `lifestyle.meal.logged` → Analyze meal photo, extract calories
  - `lifestyle.menu.scanned` → Provide restaurant menu recommendations

- **Event Emitters:**
  - `patient.nutrition.advice` → Send food swap recommendation

**Dependencies:**
- OpenRouter API (food recognition, calorie estimation)
- PostgreSQL (meal history, diet preferences)
- MinIO (meal photos)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- OpenRouter SDK (GPT-4 Vision for photo analysis)

### Health Insights Agent

**Responsibility:** Pattern detection (FR5), generates insights, validates predictions (NFR5: 95% accuracy), provides pre-meal warnings (FR6).

**Key Interfaces:**
- **Event Listeners:**
  - `insights.pattern.update` → Update food-glucose correlation patterns
  - `insights.pattern.detect` → Run pattern detection (4+ weeks data)

- **Event Emitters:**
  - `patient.pattern.detected` → Notify patient of new pattern
  - `patient.warning.pre_meal` → Send pre-meal warning (FR6)

**Dependencies:**
- PostgreSQL with TimescaleDB (time-series glucose data)
- OpenRouter API (pattern validation)
- Python microservice (optional: scikit-learn for correlation analysis)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- Prisma ORM
- Node.js statistical libraries or Python subprocess

### Doctor Bridge Agent

**Responsibility:** Routes questions to doctors (FR24), manages doctor alerts, enhances messages with context, pools similar questions, relays doctor responses.

**Key Interfaces:**
- **Event Listeners:**
  - `doctor.question.route` → Classify question (routine vs medical)
  - `doctor.alert.create` → Create alert in doctor dashboard

- **Event Emitters:**
  - `doctor.question.pooled` → Add to pooled questions
  - `patient.answer.relayed` → Send doctor answer to patient

**Dependencies:**
- OpenRouter API (question classification)
- PostgreSQL (pooled questions, doctor preferences)

**Technology Stack:**
- Node.js 20 LTS + TypeScript

### Care Coordinator Agent

**Responsibility:** Manages test appointments (FR10), medication refills, lab integration, payment processing, appointment scheduling (Epic 13).

**Key Interfaces:**
- **Event Listeners:**
  - `care.appointment.book` → Book appointment with doctor
  - `care.test.schedule` → Schedule lab test (2-week reminders)

- **Event Emitters:**
  - `patient.appointment.confirmed` → Send appointment confirmation
  - `payment.initiated` → Trigger Razorpay payment

**Dependencies:**
- Razorpay API (payment gateway)
- Lab Partner APIs (Apollo, Metropolis)
- PostgreSQL (appointment data)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- Razorpay SDK

### Learning Library Agent

**Responsibility:** Context-based health education tips (FR19), triggered by patient behavior patterns (not comprehensive libraries).

**Key Interfaces:**
- **Event Listeners:**
  - `learning.tip.request` → Provide education tip based on context

- **Event Emitters:**
  - `patient.education.send` → Send educational tip

**Dependencies:**
- OpenRouter API (generate contextual tips)
- PostgreSQL (patient history for context)

**Technology Stack:**
- Node.js 20 LTS + TypeScript

## Supporting Services

### Sync Engine

**Responsibility:** Handles offline sync (FR16, FR17), conflict resolution (device timestamp as source of truth), duplicate detection.

**Key Interfaces:**
- tRPC `sync.syncOfflineData` endpoint
- WatermelonDB sync adapter (mobile client)

**Dependencies:**
- PostgreSQL (server data)
- Redis (sync locks)

**Technology Stack:**
- Part of API Gateway (tRPC route)

### OCR Service

**Responsibility:** Extract text from photos (glucometer screens, prescriptions, health records).

**Key Interfaces:**
- Async job processing via RabbitMQ
- MinIO for photo storage/retrieval

**Dependencies:**
- OpenRouter API (GPT-4 Vision for OCR)
- MinIO (photo storage)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- OpenRouter SDK

### Notification Service

**Responsibility:** Send notifications via Evolution API (WhatsApp), Expo Push Notifications (mobile), Email (Twilio SendGrid).

**Key Interfaces:**
- **Event Listeners:**
  - `patient.notification.send` → Send notification
  - `doctor.alert.send` → Send doctor alert
  - `caregiver.alert.send` → Send caregiver alert

**Dependencies:**
- Evolution API (WhatsApp)
- Expo Push Notification Service
- Twilio SendGrid (email)
- Redis (rate limiting)

**Technology Stack:**
- Node.js 20 LTS + TypeScript
- Evolution API SDK
- Expo Server SDK

---
