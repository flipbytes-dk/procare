# API Specification

ProCare uses **tRPC** for end-to-end type-safe API communication. This section defines the tRPC router structure that provides type safety from database to UI without code generation.

## tRPC Router Architecture

**Why tRPC:**
- ✅ End-to-end TypeScript type safety (DB → API → UI)
- ✅ No code generation or schema files (types inferred from code)
- ✅ Autocomplete in IDE for all API calls
- ✅ Compile-time error checking
- ✅ Perfect for monorepo with shared types

**Router Organization:**

```typescript
// Root tRPC Router
export const appRouter = router({
  auth: authRouter,           // Authentication & OTP
  patient: patientRouter,     // Patient data & actions
  glucose: glucoseRouter,     // Glucose logging & retrieval
  medication: medicationRouter, // Medication tracking
  meal: mealRouter,           // Meal logging
  healthRecord: healthRecordRouter, // Health records management
  doctor: doctorRouter,       // Doctor dashboard & actions
  caregiver: caregiverRouter, // Caregiver dashboard & actions
  appointment: appointmentRouter, // Appointment booking
  ai: aiRouter,               // AI question answering
  pattern: patternRouter,     // Pattern detection & insights
  notification: notificationRouter, // Push notifications & alerts
  sync: syncRouter,           // Offline sync resolution
});

export type AppRouter = typeof appRouter;
```

## Authentication Router

**Purpose:** OTP-based authentication for patients, email/password for doctors/caregivers

```typescript
import { z } from 'zod';
import { router, publicProcedure, protectedProcedure } from '../trpc';

export const authRouter = router({
  // Send OTP to phone number
  sendOtp: publicProcedure
    .input(z.object({
      phoneNumber: z.string().regex(/^\+[1-9]\d{1,14}$/), // E.164 format
      language: z.enum(['en', 'hi', 'ta', 'te']).optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Generate OTP, send via Twilio
      // Return: { success: boolean, expiresAt: Date }
    }),

  // Verify OTP and create/login user
  verifyOtp: publicProcedure
    .input(z.object({
      phoneNumber: z.string(),
      otp: z.string().length(6),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify OTP, create JWT token
      // Return: { token: string, user: User, isNewUser: boolean }
    }),

  // Doctor/Caregiver email/password login
  loginWithPassword: publicProcedure
    .input(z.object({
      email: z.string().email(),
      password: z.string().min(8),
    }))
    .mutation(async ({ input, ctx }) => {
      // Verify credentials, return JWT
      // Return: { token: string, user: User }
    }),

  // Get current authenticated user
  me: protectedProcedure
    .query(async ({ ctx }) => {
      // Return current user from JWT
      // Return: User
    }),

  // Logout (invalidate token)
  logout: protectedProcedure
    .mutation(async ({ ctx }) => {
      // Invalidate JWT token in Redis
      // Return: { success: boolean }
    }),
});
```

## Patient Router

**Purpose:** Patient profile, onboarding, and management

```typescript
export const patientRouter = router({
  // Complete onboarding after OTP verification
  completeOnboarding: protectedProcedure
    .input(z.object({
      fullName: z.string(),
      dateOfBirth: z.date(),
      diabetesType: z.enum(['TYPE_1', 'TYPE_2']),
      diagnosedAt: z.date(),
      managementMode: z.enum(['PATIENT_PRIMARY', 'SHARED', 'CAREGIVER_PRIMARY']),
      doctorQrCode: z.string().optional(), // Scanned from doctor's QR
      language: z.enum(['en', 'hi', 'ta', 'te']),
    }))
    .mutation(async ({ input, ctx }) => {
      // Create patient profile, connect to doctor if QR provided
      // Return: { patient: Patient, doctor?: Doctor }
    }),

  // Get patient dashboard data (home screen)
  getDashboard: protectedProcedure
    .query(async ({ ctx }) => {
      // Return patient benchmark, recent readings, pending alerts
      // Return: {
      //   patient: Patient,
      //   lastHbA1c: number,
      //   recentReadings: GlucoseReading[],
      //   pendingAlerts: Alert[],
      //   mostUsedFeatures: string[]
      // }
    }),

  // Get patient profile
  getProfile: protectedProcedure
    .query(async ({ ctx }) => {
      // Return: Patient
    }),

  // Update patient profile
  updateProfile: protectedProcedure
    .input(z.object({
      targetHbA1c: z.number().optional(),
      glucoseUnit: z.enum(['mg/dL', 'mmol/L']).optional(),
      preferredChannel: z.enum(['APP', 'WHATSAPP']).optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Return: Patient
    }),

  // Get trends (all health markers)
  getTrends: protectedProcedure
    .input(z.object({
      period: z.enum(['7_DAYS', '30_DAYS', '3_MONTHS']),
      marker: z.enum(['GLUCOSE', 'HBA1C', 'WEIGHT', 'BP']).optional(),
    }))
    .query(async ({ input, ctx }) => {
      // Return time-series trend data
      // Return: { data: Array<{ timestamp: Date, value: number }> }
    }),

  // Connect to caregiver
  connectCaregiver: protectedProcedure
    .input(z.object({
      caregiverPhoneNumber: z.string(),
      permissions: z.object({
        canUpdateData: z.boolean(),
        canViewHealthRecords: z.boolean(),
      }),
    }))
    .mutation(async ({ input, ctx }) => {
      // Create CaregiverPatientConnection
      // Return: { connection: CaregiverPatientConnection }
    }),
});
```

## Glucose Router

**Purpose:** Glucose logging (multi-input methods) and retrieval

```typescript
export const glucoseRouter = router({
  // Log glucose reading (manual, voice, OCR)
  logReading: protectedProcedure
    .input(z.object({
      value: z.number().min(40).max(600), // mg/dL validation
      state: z.enum([
        'MORNING_FASTING', 'AFTER_BREAKFAST', 'BEFORE_LUNCH',
        'AFTER_LUNCH', 'BEFORE_DINNER', 'AFTER_DINNER', 'BEDTIME', 'RANDOM'
      ]),
      recordedAt: z.date(), // Patient-specified time (not device time)
      source: z.enum(['MANUAL', 'OCR_PHOTO', 'BLUETOOTH', 'VOICE']),
      photoUrl: z.string().optional(), // If source = OCR_PHOTO
      notes: z.string().optional(),
    }))
    .mutation(async ({ input, ctx }) => {
      // Store reading, check emergency thresholds
      // Emit event to RabbitMQ for Clinical Monitor agent
      // Return: { reading: GlucoseReading, isEmergency: boolean }
    }),

  // Get glucose readings (with pagination)
  getReadings: protectedProcedure
    .input(z.object({
      startDate: z.date().optional(),
      endDate: z.date().optional(),
      limit: z.number().default(50),
      offset: z.number().default(0),
    }))
    .query(async ({ input, ctx }) => {
      // Return paginated readings from TimescaleDB
      // Return: { readings: GlucoseReading[], total: number }
    }),

  // Get glucose statistics
  getStats: protectedProcedure
    .input(z.object({
      period: z.enum(['7_DAYS', '30_DAYS', '3_MONTHS']),
    }))
    .query(async ({ input, ctx }) => {
      // Return: {
      //   avg: number,
      //   min: number,
      //   max: number,
      //   inRangePercentage: number,
      //   hypoglycemiaEvents: number,
      //   hyperglycemiaEvents: number
      // }
    }),

  // Process glucometer photo (OCR)
  processGlucometerPhoto: protectedProcedure
    .input(z.object({
      photoUrl: z.string(), // MinIO URL after upload
    }))
    .mutation(async ({ input, ctx }) => {
      // Call OpenRouter API for OCR (GPT-4 Vision)
      // Return: { value: number, confidence: number }
    }),
});
```

*(Additional routers for medication, healthRecord, doctor, AI, sync - see full specification above)*

---
