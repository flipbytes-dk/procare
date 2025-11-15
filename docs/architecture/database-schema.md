# Database Schema

This section provides the complete Prisma schema definition for ProCare's PostgreSQL database with TimescaleDB extension. The schema implements all data models defined earlier with proper relationships, indexes, and constraints optimized for healthcare data storage and time-series queries.

## Prisma Schema Configuration

**File Location:** `packages/database/prisma/schema.prisma`

```prisma
// Prisma schema for ProCare
// PostgreSQL 16 + TimescaleDB extension

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["postgresqlExtensions"]
}

datasource db {
  provider   = "postgresql"
  url        = env("DATABASE_URL")
  extensions = [timescaledb]
}

// ===== ENUMS =====

enum UserRole {
  PATIENT
  DOCTOR
  CAREGIVER
}

enum Language {
  en // English
  hi // Hindi
  ta // Tamil
  te // Telugu
  bn // Bengali
  mr // Marathi
  gu // Gujarati
}

enum DiabetesType {
  TYPE_1
  TYPE_2
}

enum ManagementMode {
  PATIENT_PRIMARY
  SHARED
  CAREGIVER_PRIMARY
}

enum SubscriptionStatus {
  TRIAL
  ACTIVE
  EXPIRED
  CANCELLED
}

enum GlucoseState {
  MORNING_FASTING
  AFTER_BREAKFAST
  BEFORE_LUNCH
  AFTER_LUNCH
  BEFORE_DINNER
  AFTER_DINNER
  BEDTIME
  RANDOM
}

enum DataSource {
  MANUAL
  OCR_PHOTO
  BLUETOOTH
  VOICE
}

enum MedicationFrequency {
  ONCE_DAILY
  TWICE_DAILY
  THRICE_DAILY
  AS_NEEDED
}

enum MedicationStatus {
  PENDING
  TAKEN
  TAKEN_LATE
  MISSED
  SKIPPED
}

enum MealType {
  BREAKFAST
  LUNCH
  DINNER
  SNACK
}

enum HealthRecordType {
  BLOOD_TEST
  LAB_REPORT
  PRESCRIPTION
  IMAGING
  DOCTOR_NOTE
  OTHER
}

enum HealthRecordCategory {
  LIVER
  KIDNEY
  HEART
  DIABETES
  GENERAL
}

enum AppointmentStatus {
  REQUESTED
  CONFIRMED
  PAYMENT_PENDING
  PAYMENT_COMPLETE
  COMPLETED
  CANCELLED
}

enum AppointmentReason {
  ROUTINE_CHECKUP
  EMERGENCY
  FOLLOW_UP
  TEST_RESULTS
}

enum PatternType {
  FOOD_GLUCOSE_CORRELATION
  ACTIVITY_IMPACT
  MEDICATION_CONSISTENCY
}

enum DoctorSpecialty {
  DIABETES
  CARDIOLOGY
  NEPHROLOGY
  HEPATOLOGY
  OTHER
}

enum AlertFrequency {
  NONE
  IMMEDIATE
  DAILY_DIGEST
}

enum DoctorType {
  DIABETES_SPECIALIST
  REFERRING_DOCTOR
}

enum EmergencyType {
  HYPOGLYCEMIA
  HYPERGLYCEMIA
  MISSED_MEDICATION
}

enum EmergencyStatus {
  ACTIVE
  RESOLVED
  ESCALATED
}

// ===== MODELS =====

// User authentication and base profile
model User {
  id               String    @id @default(uuid())
  phoneNumber      String    @unique // E.164 format: +91XXXXXXXXXX
  email            String?   @unique
  role             UserRole
  language         Language  @default(en)
  profilePictureUrl String?
  isActive         Boolean   @default(true)
  lastLoginAt      DateTime?
  createdAt        DateTime  @default(now())
  updatedAt        DateTime  @updatedAt

  // Polymorphic relationships (one-to-one)
  patient   Patient?
  doctor    Doctor?
  caregiver Caregiver?

  // Authentication
  otpAttempts      Int      @default(0)
  lastOtpSentAt    DateTime?
  passwordHash     String? // For doctors/caregivers

  @@index([phoneNumber])
  @@index([email])
  @@index([role])
  @@map("users")
}

// Patient profile with diabetes information
model Patient {
  id       String @id @default(uuid())
  userId   String @unique
  user     User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Diabetes Information
  diabetesType DiabetesType
  diagnosedAt  DateTime
  lastHbA1c    Float? // e.g., 7.2
  targetHbA1c  Float? // e.g., 7.0 (doctor-set goal)

  // Management Configuration
  managementMode ManagementMode @default(PATIENT_PRIMARY)

  // Subscription
  subscriptionStatus SubscriptionStatus @default(TRIAL)
  trialEndsAt        DateTime?
  subscriptionEndsAt DateTime?

  // Metrics (cached for dashboard performance)
  totalGlucoseReadings         Int   @default(0)
  avgGlucoseLast30Days         Float?
  medicationAdherenceLast30Days Float? // 0-100%

  // Preferences
  glucoseUnit       String  @default("mg/dL") // 'mg/dL' or 'mmol/L'
  preferredChannel  String  @default("APP") // 'APP' or 'WHATSAPP'

  // Dashboard Cached Data (updated every 6 hours)
  healthScore       Float?   @default(0) // 0-100
  lastHealthScoreAt DateTime?
  reviewedByDoctorAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relationships
  doctors          DoctorPatientConnection[]
  caregivers       CaregiverPatientConnection[]
  glucoseReadings  GlucoseReading[]
  meals            Meal[]
  medications      Medication[]
  medicationLogs   MedicationLog[]
  healthRecords    HealthRecord[]
  appointments     Appointment[]
  patterns         Pattern[]
  emergencyEvents  EmergencyEvent[]
  notifications    Notification[]

  @@index([userId])
  @@index([subscriptionStatus])
  @@index([healthScore])
  @@map("patients")
}

// Doctor profile with specialty and revenue sharing
model Doctor {
  id       String @id @default(uuid())
  userId   String @unique
  user     User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  // Professional Information
  fullName            String
  specialty           DoctorSpecialty
  isDiabetesSpecialist Boolean         @default(false)
  licenseNumber       String?
  clinicName          String?
  clinicAddress       String?

  // Revenue Sharing
  revenueSharePercentage Int @default(30) // 30 or 60

  // Alert Configuration
  alertPreferences AlertFrequency @default(IMMEDIATE)

  // QR Code for Patient Onboarding
  onboardingQrCode String @unique // e.g., "DOC-ABC123"

  // Metrics
  totalPatients  Int @default(0)
  activePatients Int @default(0)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relationships
  patients         DoctorPatientConnection[]
  appointments     Appointment[]
  pooledQuestions  PooledQuestion[]
  sentMessages     Message[]

  @@index([userId])
  @@index([onboardingQrCode])
  @@index([specialty])
  @@map("doctors")
}

// Caregiver profile
model Caregiver {
  id       String @id @default(uuid())
  userId   String @unique
  user     User   @relation(fields: [userId], references: [id], onDelete: Cascade)

  fullName              String
  relationshipToPatient String? // 'Son', 'Daughter', 'Spouse', etc.

  // Alert Preferences
  criticalAlertsEnabled Boolean @default(true)
  dailyDigestTime       String? // HH:MM format (e.g., '20:00' for 8 PM)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relationships
  patients CaregiverPatientConnection[]

  @@index([userId])
  @@map("caregivers")
}

// Many-to-many: Doctor <-> Patient with revenue sharing
model DoctorPatientConnection {
  id        String @id @default(uuid())
  doctorId  String
  patientId String
  doctor    Doctor  @relation(fields: [doctorId], references: [id], onDelete: Cascade)
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  doctorType DoctorType

  // Referring Doctor Info
  treatingCondition String? // e.g., 'Heart Disease', 'Kidney Disease'

  // Revenue Share
  revenueSharePercentage Int @default(30) // 30% or 60%

  // Connection Status
  isActive       Boolean   @default(true)
  connectedAt    DateTime  @default(now())
  disconnectedAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([doctorId, patientId])
  @@index([doctorId])
  @@index([patientId])
  @@index([isActive])
  @@map("doctor_patient_connections")
}

// Many-to-many: Caregiver <-> Patient with permissions
model CaregiverPatientConnection {
  id          String    @id @default(uuid())
  caregiverId String
  patientId   String
  caregiver   Caregiver @relation(fields: [caregiverId], references: [id], onDelete: Cascade)
  patient     Patient   @relation(fields: [patientId], references: [id], onDelete: Cascade)

  // Permissions
  canUpdateData         Boolean @default(false)
  canViewHealthRecords  Boolean @default(true)

  isActive    Boolean  @default(true)
  connectedAt DateTime @default(now())

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@unique([caregiverId, patientId])
  @@index([caregiverId])
  @@index([patientId])
  @@index([isActive])
  @@map("caregiver_patient_connections")
}

// Glucose readings (TimescaleDB hypertable)
model GlucoseReading {
  id        String @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  // Core Data
  value      Float        // mg/dL (40-600 range)
  state      GlucoseState
  recordedAt DateTime     // Patient-specified time (NOT createdAt)

  // Metadata
  source   DataSource
  photoUrl String? // If source = OCR_PHOTO
  notes    String?

  // Flags
  isEmergency Boolean @default(false)
  isSynced    Boolean @default(false)

  createdAt DateTime @default(now()) // Device timestamp for sync
  updatedAt DateTime @updatedAt

  // Relationships
  relatedMeal   Meal?   @relation(fields: [relatedMealId], references: [id])
  relatedMealId String?

  @@index([patientId, recordedAt(sort: Desc)]) // Time-series queries
  @@index([patientId, state])
  @@index([isEmergency])
  @@index([recordedAt])
  @@map("glucose_readings")
}

// Medication prescriptions
model Medication {
  id           String              @id @default(uuid())
  patientId    String
  patient      Patient             @relation(fields: [patientId], references: [id], onDelete: Cascade)
  prescribedBy String              // Doctor ID

  name      String
  dosage    String              // e.g., '500mg'
  frequency MedicationFrequency

  // Scheduled Times
  scheduledTimes String[] // ['08:00', '20:00'] for TWICE_DAILY

  // Validity
  startDate DateTime
  endDate   DateTime?
  isActive  Boolean   @default(true)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  // Relationships
  logs MedicationLog[]

  @@index([patientId, isActive])
  @@index([prescribedBy])
  @@map("medications")
}

// Medication adherence logs
model MedicationLog {
  id           String @id @default(uuid())
  medicationId String
  medication   Medication @relation(fields: [medicationId], references: [id], onDelete: Cascade)
  patientId    String
  patient      Patient    @relation(fields: [patientId], references: [id], onDelete: Cascade)

  // Timing
  scheduledTime DateTime
  takenAt       DateTime?
  status        MedicationStatus

  // Logging
  loggedBy String // Patient or Caregiver user ID
  notes    String?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([medicationId, scheduledTime])
  @@index([patientId, status])
  @@index([scheduledTime])
  @@map("medication_logs")
}

// Meal logging with photo and AI analysis
model Meal {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  // Basic Data
  description String?
  photoUrl    String?
  mealType    MealType
  consumedAt  DateTime

  // AI Analysis (populated async after photo upload)
  recognizedFoods   String[] // ['rice', 'dal']
  estimatedCalories Float?

  // Pattern Correlation
  glucoseReadingBefore   GlucoseReading? @relation("BeforeMeal", fields: [glucoseReadingBeforeId], references: [id])
  glucoseReadingBeforeId String?
  glucoseReadingAfter    GlucoseReading[] // Multiple post-meal readings
  glucoseImpact          Float? // Delta between before/after

  // Offline Sync
  isSynced Boolean @default(false)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([patientId, consumedAt(sort: Desc)])
  @@index([mealType])
  @@map("meals")
}

// Health records (lab reports, prescriptions, etc.)
model HealthRecord {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  // File Information
  fileUrl       String
  fileName      String
  fileType      String // 'application/pdf', 'image/jpeg'
  fileSizeBytes Int

  // Classification
  recordType HealthRecordType
  category   HealthRecordCategory

  // Metadata
  recordDate  DateTime // Date of test/report (NOT upload date)
  description String?

  // OCR Extracted Data (for blood tests, prescriptions)
  extractedText    String?
  extractedMarkers Json? // { "HbA1c": 7.2, "Glucose": 128 }

  // Searchable
  tags String[]

  // Access Control
  sharedWithDoctors String[] // Doctor IDs with access

  createdAt DateTime @default(now()) // Upload timestamp
  updatedAt DateTime @updatedAt

  @@index([patientId, recordDate(sort: Desc)])
  @@index([recordType])
  @@index([category])
  @@map("health_records")
}

// Appointment booking with payment
model Appointment {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)
  doctorId  String
  doctor    Doctor  @relation(fields: [doctorId], references: [id], onDelete: Cascade)

  // Scheduling
  requestedDate DateTime
  confirmedDate DateTime?
  duration      Int // Minutes (e.g., 30)

  // Status
  status AppointmentStatus @default(REQUESTED)

  // Payment
  paymentId         String? // Razorpay payment ID
  amount            Float?  // INR
  paymentCompletedAt DateTime?

  // Reason
  reason AppointmentReason
  notes  String?

  // Pre-Appointment Tasks
  preAppointmentTasks String[] // ['Get HbA1c test done', 'Bring previous prescriptions']

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([patientId, status])
  @@index([doctorId, status])
  @@index([requestedDate])
  @@map("appointments")
}

// AI-detected patterns (food-glucose correlations)
model Pattern {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  type PatternType

  // Food-Glucose Pattern
  foodItem            String?
  avgGlucoseIncrease  Float? // +55 mg/dL
  betterAlternative   String? // 'Roti' (+25 mg/dL)

  // Pattern Confidence
  confidence  Float // 0.0-1.0 (only show if >0.7)
  sampleSize  Int // Number of observations

  // Detection
  detectedAt DateTime
  isActive   Boolean  @default(true) // False if pattern no longer holds

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([patientId, type, isActive])
  @@index([confidence])
  @@map("patterns")
}

// Pooled questions for doctors
model PooledQuestion {
  id        String @id @default(uuid())
  doctorId  String
  doctor    Doctor @relation(fields: [doctorId], references: [id], onDelete: Cascade)

  question        String
  patientCount    Int      @default(1) // Number of patients asking similar question
  firstAskedAt    DateTime @default(now())
  lastAskedAt     DateTime @default(now())

  // Answer
  answer       String?
  answeredAt   DateTime?
  answeredBy   String? // Doctor ID

  isAnswered Boolean @default(false)

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([doctorId, isAnswered])
  @@map("pooled_questions")
}

// Emergency events (hypoglycemia, hyperglycemia)
model EmergencyEvent {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  type           EmergencyType
  status         EmergencyStatus @default(ACTIVE)
  glucoseValue   Float? // mg/dL
  instructions   String // AI-generated safety steps

  // Escalation Tracking
  escalatedAt    DateTime?
  resolvedAt     DateTime?
  followUpReadingAt DateTime?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([patientId, status])
  @@index([createdAt(sort: Desc)])
  @@map("emergency_events")
}

// Notifications sent to users
model Notification {
  id        String  @id @default(uuid())
  patientId String
  patient   Patient @relation(fields: [patientId], references: [id], onDelete: Cascade)

  type    String // 'MEDICATION_REMINDER', 'EMERGENCY_ALERT', 'PRE_MEAL_WARNING'
  title   String
  body    String
  channel String // 'PUSH', 'WHATSAPP', 'EMAIL'

  // Tracking
  sentAt     DateTime  @default(now())
  readAt     DateTime?
  isRead     Boolean   @default(false)

  // Rate Limiting
  countsTowardLimit Boolean @default(true) // FR15: max 5/day

  createdAt DateTime @default(now())

  @@index([patientId, sentAt(sort: Desc)])
  @@index([patientId, isRead])
  @@map("notifications")
}

// Messages between doctors and patients
model Message {
  id         String @id @default(uuid())
  senderId   String // User ID
  receiverId String // User ID
  sender     Doctor @relation(fields: [senderId], references: [id])

  subject String?
  body    String
  isRead  Boolean  @default(false)
  readAt  DateTime?

  createdAt DateTime @default(now())

  @@index([senderId, receiverId])
  @@index([receiverId, isRead])
  @@map("messages")
}
```

## TimescaleDB Configuration

**Purpose:** Optimize glucose readings and time-series data queries with automatic partitioning and compression.

**Setup SQL (run after Prisma migration):**

```sql
-- Enable TimescaleDB extension
CREATE EXTENSION IF NOT EXISTS timescaledb;

-- Convert glucose_readings table to TimescaleDB hypertable
SELECT create_hypertable(
  'glucose_readings',
  'recorded_at',
  chunk_time_interval => INTERVAL '7 days',
  if_not_exists => TRUE
);

-- Add compression policy (compress chunks older than 30 days)
ALTER TABLE glucose_readings SET (
  timescaledb.compress,
  timescaledb.compress_segmentby = 'patient_id'
);

SELECT add_compression_policy('glucose_readings', INTERVAL '30 days');

-- Add retention policy (drop chunks older than 5 years)
SELECT add_retention_policy('glucose_readings', INTERVAL '5 years');

-- Create continuous aggregate for daily glucose averages (performance optimization)
CREATE MATERIALIZED VIEW glucose_daily_avg
WITH (timescaledb.continuous) AS
SELECT
  patient_id,
  time_bucket('1 day', recorded_at) AS day,
  AVG(value) AS avg_glucose,
  MIN(value) AS min_glucose,
  MAX(value) AS max_glucose,
  COUNT(*) AS reading_count
FROM glucose_readings
GROUP BY patient_id, time_bucket('1 day', recorded_at);

-- Refresh policy for continuous aggregate (refresh every hour)
SELECT add_continuous_aggregate_policy('glucose_daily_avg',
  start_offset => INTERVAL '1 month',
  end_offset => INTERVAL '1 hour',
  schedule_interval => INTERVAL '1 hour');
```

## Migration Strategy

**File Location:** `packages/database/prisma/migrations/`

**Migration Process:**

1. **Initial Migration:**
   ```bash
   cd packages/database
   npx prisma migrate dev --name init
   ```

2. **Apply TimescaleDB Setup:**
   ```bash
   psql $DATABASE_URL < scripts/setup-timescaledb.sql
   ```

3. **Seed Initial Data:**
   ```bash
   npx prisma db seed
   ```

**Seed Script (`packages/database/prisma/seed.ts`):**

```typescript
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

async function main() {
  // Create demo doctor
  const demoDoctor = await prisma.user.create({
    data: {
      phoneNumber: '+919999999999',
      email: 'doctor@procare.health',
      role: 'DOCTOR',
      language: 'en',
      isActive: true,
      doctor: {
        create: {
          fullName: 'Dr. Demo Diabetes Specialist',
          specialty: 'DIABETES',
          isDiabetesSpecialist: true,
          revenueSharePercentage: 60,
          onboardingQrCode: 'DOC-DEMO-001',
          alertPreferences: 'IMMEDIATE',
        },
      },
    },
  });

  console.log('Created demo doctor:', demoDoctor);

  // Create demo patient
  const demoPatient = await prisma.user.create({
    data: {
      phoneNumber: '+919999999998',
      role: 'PATIENT',
      language: 'hi',
      isActive: true,
      patient: {
        create: {
          diabetesType: 'TYPE_2',
          diagnosedAt: new Date('2023-01-01'),
          managementMode: 'PATIENT_PRIMARY',
          subscriptionStatus: 'TRIAL',
          trialEndsAt: new Date(Date.now() + 30 * 24 * 60 * 60 * 1000),
          glucoseUnit: 'mg/dL',
          preferredChannel: 'APP',
        },
      },
    },
  });

  console.log('Created demo patient:', demoPatient);
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

## Database Indexes Strategy

**Purpose:** Optimize query performance for common access patterns.

**Index Rationale:**

1. **Time-Series Queries:**
   - `glucose_readings(patient_id, recorded_at DESC)` - Patient glucose history
   - `meals(patient_id, consumed_at DESC)` - Patient meal history
   - `health_records(patient_id, record_date DESC)` - Patient health records timeline

2. **Dashboard Queries:**
   - `patients(health_score)` - Doctor dashboard bucket sorting
   - `patients(subscription_status)` - Subscription management
   - `emergency_events(patient_id, status)` - Active emergency tracking

3. **Authentication:**
   - `users(phone_number)` - OTP login lookup
   - `users(email)` - Email login lookup
   - `doctors(onboarding_qr_code)` - QR code scanning

4. **Relationships:**
   - `doctor_patient_connections(doctor_id, patient_id)` - Doctor-patient queries
   - `caregiver_patient_connections(caregiver_id, patient_id)` - Caregiver-patient queries

## Database Constraints

**Data Integrity Rules:**

1. **Referential Integrity:**
   - All foreign keys use `onDelete: Cascade` for automatic cleanup
   - User deletion cascades to Patient/Doctor/Caregiver
   - Patient deletion cascades to all related data (glucose, meals, etc.)

2. **Unique Constraints:**
   - `users.phone_number` - No duplicate phone numbers
   - `users.email` - No duplicate emails
   - `doctors.onboarding_qr_code` - Unique QR codes per doctor
   - `(doctor_id, patient_id)` - One connection per doctor-patient pair

3. **Business Logic Constraints:**
   - `glucose_readings.value` - Validated at application layer (40-600 mg/dL)
   - `patterns.confidence` - Must be 0.0-1.0 (validated in application)
   - `medication_adherence` - Must be 0-100% (validated in application)

## Schema Evolution Best Practices

**Adding New Fields:**

```bash
# 1. Update schema.prisma
# 2. Create migration
npx prisma migrate dev --name add_new_field

# 3. Deploy to production (with downtime)
npx prisma migrate deploy
```

**Adding New Tables:**

```bash
# 1. Add model to schema.prisma
# 2. Create migration with --create-only flag
npx prisma migrate dev --create-only --name add_new_table

# 3. Review generated SQL
# 4. Apply migration
npx prisma migrate dev
```

**Breaking Changes (Renaming/Deleting Columns):**

```typescript
// Use two-step migration for zero-downtime:

// Step 1: Add new column, keep old column
// Migration: 20250113_add_new_column_keep_old.sql
ALTER TABLE patients ADD COLUMN new_column_name TEXT;
UPDATE patients SET new_column_name = old_column_name;

// Deploy Step 1, update application code to use new column

// Step 2: Drop old column (after application deployment)
// Migration: 20250113_drop_old_column.sql
ALTER TABLE patients DROP COLUMN old_column_name;
```

## Database Performance Optimization

**Query Optimization:**

1. **Materialized Views for Aggregations:**
   - `glucose_daily_avg` - Pre-calculated daily glucose statistics
   - Refreshed every hour via TimescaleDB continuous aggregate policy

2. **Connection Pooling:**
   ```typescript
   // packages/database/src/client.ts
   import { PrismaClient } from '@prisma/client';

   const prisma = new PrismaClient({
     datasources: {
       db: {
         url: process.env.DATABASE_URL + '?connection_limit=20&pool_timeout=10',
       },
     },
   });
   ```

3. **Query Batching:**
   ```typescript
   // Use Prisma's batching for bulk operations
   await prisma.$transaction([
     prisma.glucoseReading.createMany({ data: readings }),
     prisma.patient.update({ where: { id }, data: { totalGlucoseReadings: { increment: readings.length } } }),
   ]);
   ```

4. **Read Replicas (Production):**
   ```typescript
   // Separate read/write connections for scale
   const writeClient = new PrismaClient({ datasources: { db: { url: WRITE_URL } } });
   const readClient = new PrismaClient({ datasources: { db: { url: READ_REPLICA_URL } } });
   ```

## Database Monitoring

**Key Metrics to Track:**

1. **Query Performance:**
   - Slow query log (queries >1 second)
   - Average query execution time
   - Index hit ratio (should be >99%)

2. **Connection Pool:**
   - Active connections
   - Connection wait time
   - Connection errors

3. **TimescaleDB Specific:**
   - Chunk compression status
   - Continuous aggregate lag
   - Hypertable size

**Monitoring Query:**

```sql
-- Check slow queries
SELECT
  query,
  calls,
  mean_exec_time,
  max_exec_time
FROM pg_stat_statements
WHERE mean_exec_time > 1000 -- 1 second
ORDER BY mean_exec_time DESC
LIMIT 20;

-- Check index usage
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC
LIMIT 20;

-- TimescaleDB compression status
SELECT
  hypertable_name,
  total_chunks,
  number_compressed_chunks,
  pg_size_pretty(uncompressed_heap_size) AS uncompressed_size,
  pg_size_pretty(compressed_heap_size) AS compressed_size
FROM timescaledb_information.compression_settings;
```

---
