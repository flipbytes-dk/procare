# Data Models

Based on the PRD requirements, here are the core data models that will be shared between frontend and backend. These TypeScript interfaces serve as the foundation for database schema (Prisma), API contracts (tRPC), and UI components.

## User & Authentication Models

**Purpose:** Base authentication and user management for all user types (patients, doctors, caregivers)

**Key Attributes:**
- `id`: UUID primary key
- `phoneNumber`: Primary identifier (OTP authentication for patients)
- `email`: Secondary identifier (doctors/caregivers)
- `role`: User type (PATIENT, DOCTOR, CAREGIVER)
- `language`: Preferred language (Hindi, English, Tamil, Telugu, etc.)
- `createdAt`, `updatedAt`: Audit timestamps

### TypeScript Interface

```typescript
enum UserRole {
  PATIENT = 'PATIENT',
  DOCTOR = 'DOCTOR',
  CAREGIVER = 'CAREGIVER'
}

enum Language {
  ENGLISH = 'en',
  HINDI = 'hi',
  TAMIL = 'ta',
  TELUGU = 'te'
}

interface User {
  id: string; // UUID
  phoneNumber: string; // E.164 format: +91XXXXXXXXXX
  email?: string;
  role: UserRole;
  language: Language;
  profilePictureUrl?: string;
  isActive: boolean;
  lastLoginAt?: Date;
  createdAt: Date;
  updatedAt: Date;

  // Polymorphic relationships
  patient?: Patient;
  doctor?: Doctor;
  caregiver?: Caregiver;
}
```

### Relationships

- **One-to-One:** User → Patient (if role = PATIENT)
- **One-to-One:** User → Doctor (if role = DOCTOR)
- **One-to-One:** User → Caregiver (if role = CAREGIVER)

## Patient Model

**Purpose:** Core patient entity with diabetes information, management mode, and subscription status

**Key Attributes:**
- `diabetesType`: Type 1 or Type 2
- `diagnosedAt`: Diagnosis date
- `lastHbA1c`: Latest HbA1c reading (benchmark metric)
- `managementMode`: Patient-primary, Shared, or Caregiver-primary
- `subscriptionStatus`: Active, Trial, Expired

### TypeScript Interface

```typescript
enum DiabetesType {
  TYPE_1 = 'TYPE_1',
  TYPE_2 = 'TYPE_2'
}

enum ManagementMode {
  PATIENT_PRIMARY = 'PATIENT_PRIMARY',
  SHARED = 'SHARED',
  CAREGIVER_PRIMARY = 'CAREGIVER_PRIMARY'
}

enum SubscriptionStatus {
  TRIAL = 'TRIAL',          // 1-month free trial (referred patients)
  ACTIVE = 'ACTIVE',        // Paid subscription
  EXPIRED = 'EXPIRED',      // Trial ended, no payment
  CANCELLED = 'CANCELLED'   // User cancelled
}

interface Patient {
  id: string;
  userId: string; // Foreign key to User

  // Diabetes Information
  diabetesType: DiabetesType;
  diagnosedAt: Date;
  lastHbA1c?: number; // e.g., 7.2
  targetHbA1c?: number; // e.g., 7.0 (doctor-set goal)

  // Management Configuration
  managementMode: ManagementMode;

  // Subscription
  subscriptionStatus: SubscriptionStatus;
  trialEndsAt?: Date;
  subscriptionEndsAt?: Date;

  // Metrics (cached for dashboard performance)
  totalGlucoseReadings: number;
  avgGlucoseLast30Days?: number;
  medicationAdherenceLast30Days?: number; // 0-100%

  // Preferences
  glucoseUnit: 'mg/dL' | 'mmol/L';
  preferredChannel: 'APP' | 'WHATSAPP';

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  user: User;
  doctors: DoctorPatientConnection[];
  caregivers: CaregiverPatientConnection[];
  glucoseReadings: GlucoseReading[];
  meals: Meal[];
  medications: Medication[];
  healthRecords: HealthRecord[];
}
```

### Relationships

- **Many-to-Many:** Patient ↔ Doctor (via DoctorPatientConnection)
- **Many-to-Many:** Patient ↔ Caregiver (via CaregiverPatientConnection)
- **One-to-Many:** Patient → GlucoseReading[]
- **One-to-Many:** Patient → Meal[]
- **One-to-Many:** Patient → HealthRecord[]

## Doctor Model

**Purpose:** Diabetes specialists and referring doctors with revenue sharing

**Key Attributes:**
- `specialty`: Diabetes specialist or other (cardiology, nephrology, etc.)
- `isDiabetesSpecialist`: Boolean flag for revenue share calculation
- `revenueSharePercentage`: 30% (referring) or 60% (diabetes specialist)
- `alertPreferences`: Configurable alert settings

### TypeScript Interface

```typescript
enum DoctorSpecialty {
  DIABETES = 'DIABETES',
  CARDIOLOGY = 'CARDIOLOGY',
  NEPHROLOGY = 'NEPHROLOGY',
  HEPATOLOGY = 'HEPATOLOGY',
  OTHER = 'OTHER'
}

enum AlertFrequency {
  NONE = 'NONE',           // No alerts
  IMMEDIATE = 'IMMEDIATE', // Real-time push notifications
  DAILY_DIGEST = 'DAILY_DIGEST' // Daily summary email
}

interface Doctor {
  id: string;
  userId: string; // Foreign key to User

  // Professional Information
  fullName: string;
  specialty: DoctorSpecialty;
  isDiabetesSpecialist: boolean;
  licenseNumber?: string;
  clinicName?: string;
  clinicAddress?: string;

  // Revenue Sharing
  revenueSharePercentage: number; // 30 or 60

  // Alert Configuration
  alertPreferences: AlertFrequency;

  // QR Code for Patient Onboarding
  onboardingQrCode: string; // Unique doctor ID for QR scanning

  // Metrics
  totalPatients: number;
  activePatients: number;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  user: User;
  patients: DoctorPatientConnection[];
  appointments: Appointment[];
  pooledQuestions: PooledQuestion[];
}
```

### Relationships

- **Many-to-Many:** Doctor ↔ Patient (via DoctorPatientConnection)
- **One-to-Many:** Doctor → Appointment[]
- **One-to-Many:** Doctor → PooledQuestion[]

## Caregiver Model

**Purpose:** Family members monitoring patient health with peace-of-mind dashboard

### TypeScript Interface

```typescript
interface Caregiver {
  id: string;
  userId: string; // Foreign key to User

  fullName: string;
  relationshipToPatient?: string; // 'Son', 'Daughter', 'Spouse', etc.

  // Alert Preferences
  criticalAlertsEnabled: boolean; // Hypoglycemia, missed meds
  dailyDigestTime?: string; // HH:MM format (e.g., '20:00' for 8 PM)

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  user: User;
  patients: CaregiverPatientConnection[];
}
```

### Relationships

- **Many-to-Many:** Caregiver ↔ Patient (via CaregiverPatientConnection)

## GlucoseReading Model (Time-Series)

**Purpose:** Patient glucose readings with time/state verification (core time-series data)

**Key Attributes:**
- `value`: Glucose value (40-600 mg/dL validated)
- `state`: Morning fasting, After lunch, etc. (required for context)
- `recordedAt`: Patient-specified timestamp (NOT device timestamp)
- `source`: Manual entry, OCR photo, Bluetooth device

### TypeScript Interface

```typescript
enum GlucoseState {
  MORNING_FASTING = 'MORNING_FASTING',
  AFTER_BREAKFAST = 'AFTER_BREAKFAST',
  BEFORE_LUNCH = 'BEFORE_LUNCH',
  AFTER_LUNCH = 'AFTER_LUNCH',
  BEFORE_DINNER = 'BEFORE_DINNER',
  AFTER_DINNER = 'AFTER_DINNER',
  BEDTIME = 'BEDTIME',
  RANDOM = 'RANDOM'
}

enum DataSource {
  MANUAL = 'MANUAL',           // Typed in app/WhatsApp
  OCR_PHOTO = 'OCR_PHOTO',     // Glucometer photo
  BLUETOOTH = 'BLUETOOTH',     // Connected glucometer
  VOICE = 'VOICE'              // Voice message
}

interface GlucoseReading {
  id: string;
  patientId: string;

  // Core Data (stored in TimescaleDB for optimized queries)
  value: number; // mg/dL (40-600 range)
  state: GlucoseState;
  recordedAt: Date; // Patient-specified time (NOT createdAt)

  // Metadata
  source: DataSource;
  photoUrl?: string; // If source = OCR_PHOTO
  notes?: string;

  // Flags
  isEmergency: boolean; // True if hypoglycemia/hyperglycemia
  isSynced: boolean; // False if logged offline, True after sync

  createdAt: Date; // Device timestamp for sync conflict resolution
  updatedAt: Date;

  // Relationships
  patient: Patient;
  relatedMeal?: Meal; // If post-meal reading
}
```

### Relationships

- **Many-to-One:** GlucoseReading → Patient
- **One-to-One (optional):** GlucoseReading → Meal (for post-meal correlation)

## Medication & MedicationLog Models

**Purpose:** Prescribed medications and adherence tracking

### TypeScript Interface

```typescript
enum MedicationFrequency {
  ONCE_DAILY = 'ONCE_DAILY',
  TWICE_DAILY = 'TWICE_DAILY',
  THRICE_DAILY = 'THRICE_DAILY',
  AS_NEEDED = 'AS_NEEDED'
}

interface Medication {
  id: string;
  patientId: string;
  prescribedBy: string; // Doctor ID

  name: string; // e.g., 'Metformin'
  dosage: string; // e.g., '500mg'
  frequency: MedicationFrequency;

  // Scheduled Times
  scheduledTimes: string[]; // ['08:00', '20:00'] for TWICE_DAILY

  // Validity
  startDate: Date;
  endDate?: Date;
  isActive: boolean;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  patient: Patient;
  logs: MedicationLog[];
}

enum MedicationStatus {
  TAKEN = 'TAKEN',
  TAKEN_LATE = 'TAKEN_LATE', // Logged within 2 hours
  MISSED = 'MISSED',
  SKIPPED = 'SKIPPED' // Intentionally skipped
}

interface MedicationLog {
  id: string;
  medicationId: string;
  patientId: string;

  // Timing
  scheduledTime: Date;
  takenAt?: Date;
  status: MedicationStatus;

  // Logging
  loggedBy: string; // Patient or Caregiver user ID
  notes?: string;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  medication: Medication;
  patient: Patient;
}
```

### Relationships

- **Many-to-One:** Medication → Patient
- **One-to-Many:** Medication → MedicationLog[]

## Meal Model

**Purpose:** Simplified meal logging with photo and basic food recognition

### TypeScript Interface

```typescript
interface Meal {
  id: string;
  patientId: string;

  // Basic Data
  description?: string; // 'Rice, dal, chapati'
  photoUrl?: string;
  mealType: 'BREAKFAST' | 'LUNCH' | 'DINNER' | 'SNACK';
  consumedAt: Date;

  // AI Analysis (populated async after photo upload)
  recognizedFoods?: string[]; // ['rice', 'dal']
  estimatedCalories?: number;

  // Pattern Correlation
  glucoseReadingBefore?: string; // Glucose reading ID
  glucoseReadingAfter?: string;
  glucoseImpact?: number; // Delta between before/after

  // Offline Sync
  isSynced: boolean;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  patient: Patient;
}
```

### Relationships

- **Many-to-One:** Meal → Patient
- **One-to-One (optional):** Meal → GlucoseReading (before/after)

## HealthRecord Model

**Purpose:** Comprehensive health records management (critical value prop per PRD)

### TypeScript Interface

```typescript
enum HealthRecordType {
  BLOOD_TEST = 'BLOOD_TEST',
  LAB_REPORT = 'LAB_REPORT',
  PRESCRIPTION = 'PRESCRIPTION',
  IMAGING = 'IMAGING',
  DOCTOR_NOTE = 'DOCTOR_NOTE',
  OTHER = 'OTHER'
}

enum HealthRecordCategory {
  LIVER = 'LIVER',
  KIDNEY = 'KIDNEY',
  HEART = 'HEART',
  DIABETES = 'DIABETES',
  GENERAL = 'GENERAL'
}

interface HealthRecord {
  id: string;
  patientId: string;

  // File Information
  fileUrl: string; // MinIO S3 URL
  fileName: string;
  fileType: string; // 'application/pdf', 'image/jpeg'
  fileSizeBytes: number;

  // Classification
  recordType: HealthRecordType;
  category: HealthRecordCategory;

  // Metadata
  recordDate: Date; // Date of test/report (NOT upload date)
  description?: string;

  // OCR Extracted Data (for blood tests, prescriptions)
  extractedText?: string;
  extractedMarkers?: Record<string, any>; // { "HbA1c": 7.2, "Glucose": 128 }

  // Searchable
  tags?: string[];

  // Access Control
  sharedWithDoctors: string[]; // Doctor IDs with access

  createdAt: Date; // Upload timestamp
  updatedAt: Date;

  // Relationships
  patient: Patient;
}
```

### Relationships

- **Many-to-One:** HealthRecord → Patient

## Appointment Model

**Purpose:** Doctor appointment booking with payment integration (Epic 13 - FR13)

### TypeScript Interface

```typescript
enum AppointmentStatus {
  REQUESTED = 'REQUESTED',     // Patient requested, awaiting doctor confirmation
  CONFIRMED = 'CONFIRMED',     // Doctor confirmed
  PAYMENT_PENDING = 'PAYMENT_PENDING',
  PAYMENT_COMPLETE = 'PAYMENT_COMPLETE',
  COMPLETED = 'COMPLETED',     // Appointment happened
  CANCELLED = 'CANCELLED'
}

interface Appointment {
  id: string;
  patientId: string;
  doctorId: string;

  // Scheduling
  requestedDate: Date;
  confirmedDate?: Date;
  duration: number; // Minutes (e.g., 30)

  // Status
  status: AppointmentStatus;

  // Payment
  paymentId?: string; // Razorpay payment ID
  amount?: number; // INR
  paymentCompletedAt?: Date;

  // Reason
  reason: 'ROUTINE_CHECKUP' | 'EMERGENCY' | 'FOLLOW_UP' | 'TEST_RESULTS';
  notes?: string;

  // Pre-Appointment Tasks (doctor-requested)
  preAppointmentTasks?: string[]; // ['Get HbA1c test done', 'Bring previous prescriptions']

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  patient: Patient;
  doctor: Doctor;
}
```

### Relationships

- **Many-to-One:** Appointment → Patient
- **Many-to-One:** Appointment → Doctor

## Pattern Model

**Purpose:** AI-detected food-glucose correlations (FR5, FR6 - 4+ weeks data)

### TypeScript Interface

```typescript
enum PatternType {
  FOOD_GLUCOSE_CORRELATION = 'FOOD_GLUCOSE_CORRELATION',
  ACTIVITY_IMPACT = 'ACTIVITY_IMPACT',
  MEDICATION_CONSISTENCY = 'MEDICATION_CONSISTENCY'
}

interface Pattern {
  id: string;
  patientId: string;

  type: PatternType;

  // Food-Glucose Pattern
  foodItem?: string; // 'Rice'
  avgGlucoseIncrease?: number; // +55 mg/dL
  betterAlternative?: string; // 'Roti' (+25 mg/dL)

  // Pattern Confidence
  confidence: number; // 0.0-1.0 (only show if >0.7)
  sampleSize: number; // Number of observations

  // Detection
  detectedAt: Date;
  isActive: boolean; // False if pattern no longer holds

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  patient: Patient;
}
```

### Relationships

- **Many-to-One:** Pattern → Patient

## Connection Models

**Purpose:** Many-to-many relationships between patients, doctors, and caregivers

### TypeScript Interface

```typescript
enum DoctorType {
  DIABETES_SPECIALIST = 'DIABETES_SPECIALIST',
  REFERRING_DOCTOR = 'REFERRING_DOCTOR'
}

interface DoctorPatientConnection {
  id: string;
  doctorId: string;
  patientId: string;

  doctorType: DoctorType;

  // Referring Doctor Info (if doctorType = REFERRING_DOCTOR)
  treatingCondition?: string; // 'Heart Disease', 'Kidney Disease'

  // Revenue Share
  revenueSharePercentage: number; // 30% or 60%

  // Connection Status
  isActive: boolean;
  connectedAt: Date;
  disconnectedAt?: Date;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  doctor: Doctor;
  patient: Patient;
}

interface CaregiverPatientConnection {
  id: string;
  caregiverId: string;
  patientId: string;

  // Permissions
  canUpdateData: boolean; // Proxy logging
  canViewHealthRecords: boolean;

  isActive: boolean;
  connectedAt: Date;

  createdAt: Date;
  updatedAt: Date;

  // Relationships
  caregiver: Caregiver;
  patient: Patient;
}
```

---
