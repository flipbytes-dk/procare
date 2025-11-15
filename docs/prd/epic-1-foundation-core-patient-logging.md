# Epic 1: Foundation & Core Patient Logging

## Epic Goal

Establish the foundational infrastructure for ProCare including project setup, Git repository, CI/CD pipelines, and core database architecture. Develop native mobile apps (iOS and Android) as the primary interface with flexible UX (chat-based primary with direct input options). Integrate WhatsApp Business API as secondary channel for re-engagement and templated interactions. Implement offline-first database architecture with local SQLite storage and sync engine foundation (primarily in mobile app). Enable patients to log glucose readings and meals via mobile app (primary) or WhatsApp (secondary) using multiple input methods (text, voice, photo, direct tags). Mobile app provides flexible UX with direct input options (e.g., glucose tags for quick logging), while WhatsApp remains conversational-only. Data stored locally and synced to cloud when connectivity is available. Both app and WhatsApp are always available - patients can use either channel at any time. Same onboarding flow but different interfaces (app vs WhatsApp). Messages route to the channel where patient last interacted. This epic delivers immediate patient value (working logging functionality) while establishing the technical foundation for all subsequent features.

## Story 1.1: Project Foundation & Infrastructure Setup

As a developer,
I want a properly configured project repository with Git, CI/CD, and development environment,
so that the team can collaborate effectively and deploy code reliably.

**Acceptance Criteria:**
1. Git repository initialized with appropriate .gitignore for Node.js/Python, Docker, and environment files
2. CI/CD pipeline configured (GitHub Actions or similar) with automated testing, linting, and build steps
3. Monorepo structure established with directories for: services (agents), frontend (web dashboard), shared libraries, and infrastructure
4. Docker and Docker Compose configuration for local development environment
5. Environment variable management system (.env files, secrets management)
6. Basic health check endpoint or canary page to verify deployment
7. README with setup instructions, architecture overview, and development workflow
8. Code formatting and linting rules configured (ESLint/Prettier for JavaScript, Black/Flake8 for Python)

## Story 1.1a: Infrastructure as Code Setup

As a DevOps engineer,
I want infrastructure defined as code,
so that environments can be reproduced reliably and deployments can be automated.

**Acceptance Criteria:**
1. IaC tool selection: Terraform or OpenTofu chosen for infrastructure provisioning
2. Hetzner Cloud provider configuration: Terraform provider configured for Hetzner Cloud API
3. VPS provisioning code: Terraform configuration created for production (CPX41) and staging (CPX21) servers
4. Server configuration includes: Ubuntu 24.04 LTS, SSH key setup, firewall rules (ports 80, 443, 8000)
5. Coolify installation automation: Ansible playbook or Terraform provisioner script for automated Coolify installation
6. DNS configuration: Terraform code for DNS records (procare.health, api.procare.health, doctor.procare.health, caregiver.procare.health, staging.procare.health)
7. State management: Terraform state stored securely (Terraform Cloud or S3-compatible backend with state locking)
8. Environment separation: Separate Terraform workspaces or directories for staging and production
9. Infrastructure documentation: README with setup instructions, variable definitions, and deployment process
10. Secrets management: Sensitive variables (API keys, passwords) stored in environment variables or secret management system (not in Terraform code)
11. Rollback capability: Infrastructure changes can be rolled back using Terraform state history
12. Alternative approach documented: If IaC not implemented initially, document manual Coolify setup steps as accepted approach with plan to add IaC in Phase 2

## Story 1.1b: Third-Party Service Setup

As a DevOps/Admin user,
I want to set up all required third-party service accounts,
so that API keys are available before development begins.

**Acceptance Criteria:**
1. OpenRouter account creation: Create OpenRouter account for AI/LLM access
2. OpenRouter API key obtained: Generate API key from OpenRouter dashboard (test mode for development)
3. Evolution API deployment: Deploy self-hosted Evolution API instance on development environment (Docker container on Hetzner VPS)
4. Evolution API configuration: Configure Evolution API with webhook URL, generate API key
5. Razorpay merchant account: Create Razorpay merchant account (test mode for MVP development)
6. Razorpay API credentials: Obtain API Key ID and Secret from Razorpay dashboard
7. Twilio account creation: Create Twilio account for SMS/OTP delivery
8. Twilio credentials obtained: Obtain Account SID, Auth Token, and phone number from Twilio console
9. WhatsApp Business API application: Submit WhatsApp Business API application (2-4 week approval timeline - apply early!)
10. WhatsApp API webhook setup: Configure webhook URL for WhatsApp Business API (Evolution API endpoint)
11. BunnyCDN account: Create BunnyCDN account for static asset hosting and edge caching
12. BunnyCDN storage zone: Create storage zone and obtain API key
13. Hetzner Object Storage: Create Hetzner Object Storage bucket for backups (S3-compatible)
14. Service account documentation: Document all account creation steps, credentials location (.env.example), and renewal/billing information
15. Development environment configuration: All API keys configured in development .env files
16. Test API calls: Verify each service account with test API call (OpenRouter completion, Twilio SMS send, Razorpay test payment)

## Story 1.1c: External API Resilience & Circuit Breakers

As a developer,
I want resilient external API integrations,
so that the system handles API failures gracefully without cascading failures.

**Acceptance Criteria:**
1. Circuit breaker pattern implemented: Circuit breaker library integrated (e.g., opossum for Node.js, pybreaker for Python)
2. Circuit breaker for OpenRouter API: Protects against OpenRouter API failures (AI backbone - critical)
3. Circuit breaker for Razorpay API: Protects against payment gateway failures
4. Circuit breaker for Twilio API: Protects against SMS/OTP delivery failures
5. Circuit breaker for lab partner APIs: Protects against Apollo/Metropolis API failures
6. Circuit breaker configuration: Configurable thresholds (failure rate, timeout, volume) per service
7. Circuit breaker states: Closed (normal), Open (failing, reject requests), Half-Open (testing recovery)
8. Retry logic with exponential backoff: Transient failures retried with exponential backoff (1s, 2s, 4s, 8s, 16s)
9. OpenRouter response caching: AI responses cached in Redis for 24h (identical questions) to reduce API dependency
10. Graceful degradation messages: When OpenRouter unavailable, show cached responses or "AI temporarily unavailable, your question has been escalated to your doctor"
11. API health monitoring: Circuit breaker state logged to monitoring system (Sentry)
12. Failure alerts: Circuit breaker opens → alert sent to DevOps team via monitoring system
13. Automatic recovery: Circuit breaker automatically transitions to Half-Open state after cooldown period (30 seconds)
14. Request queuing: Failed requests queued for processing when API recovers (for non-time-critical operations)
15. Fallback strategies documented: Fallback behavior defined for each critical external API failure scenario

## Story 1.2: Database Architecture & Offline-First Foundation

As a patient,
I want my data stored locally so I can use ProCare even without internet connectivity,
so that I'm not blocked by intermittent connectivity issues.

**Acceptance Criteria:**
1. PostgreSQL database schema designed and implemented for cloud storage (users, patients, doctors, caregivers, glucose_readings, meals, medications, events)
2. SQLite local database schema implemented matching PostgreSQL structure for offline storage
3. Event sourcing model designed: all patient actions stored as immutable events with device timestamp
4. Database migration system configured (e.g., Alembic for Python, Knex for Node.js)
5. Local database initialization logic: creates SQLite database on first launch, stores 90-day data cache
6. Basic sync engine foundation: detects online/offline status, queues events for sync when offline
7. Conflict resolution strategy documented: device timestamp as source of truth, last-write-wins for settings
8. Database connection pooling and error handling implemented

## Story 1.3: Native Mobile App Development (iOS & Android)

As a patient,
I want to use a native mobile app as my primary interface for ProCare,
so that I have a flexible UX with chat-based primary interface and direct input options for quick data entry.

**Acceptance Criteria:**
1. Native iOS app developed (Swift/SwiftUI or React Native) with flexible UX design
2. Native Android app developed (Kotlin/Jetpack Compose or React Native) with flexible UX design
3. Chat-based primary interface with conversational capabilities
4. Direct input options: tags below glucose value for quick logging without follow-up questions (e.g., "Sugar: [tag]" → direct number entry)
5. Photo upload for meal logging, glucometer readings, prescriptions, and health records
6. Natural language input (text, voice) for conversational interactions when preferred
7. Offline-first functionality with local SQLite storage and sync when connectivity restored
8. Authentication system (phone + OTP) integrated
9. Push notifications for reminders and alerts
10. Integration with device features (camera, Bluetooth for device integration, Google Fit/Apple Health)
11. App store deployment setup (App Store Connect, Google Play Console)
12. Error handling and offline state management
13. Basic logging and monitoring for app interactions

## Story 1.3a: WhatsApp Business API Integration (Secondary Channel)

As a patient,
I want to interact with ProCare via WhatsApp as a secondary channel,
so that I can re-engage and receive templated interactions when the app is not being used.

**Acceptance Criteria:**
1. WhatsApp Business API account created and approved (or test environment configured)
2. Webhook endpoint implemented to receive incoming WhatsApp messages (text, voice, images)
3. Message sending functionality implemented via WhatsApp Business API
4. Media handling: receive and process images (meal photos, glucometer screens)
5. Voice message handling: receive and process audio messages (Hindi/English)
6. Conversational-only interface (no direct input options like app)
7. Templated interactions: summaries, questions, basic data collection
8. Message queuing system for rate limit compliance (max 5 notifications/day, 2-hour spacing)
9. Error handling for API failures, rate limits, and invalid messages
10. Webhook signature verification for security
11. Requires connectivity (no offline functionality)
12. Used primarily for re-engagement when app is not being used
13. Basic logging and monitoring for WhatsApp API interactions

## Story 1.4: Doctor-Initiated Patient Onboarding & Authentication

As a new patient,
I want to be onboarded by my doctor who says "ProCare will take care of you," then complete onboarding via app or WhatsApp,
so that I can start managing my diabetes immediately with doctor guidance.

**Acceptance Criteria:**
1. Doctor-initiated onboarding: Doctor says "ProCare will take care of you" to patient, initiating onboarding process
2. Patient registration flow via mobile app or WhatsApp: phone number verification (OTP via SMS/WhatsApp)
3. Same onboarding flow for both app and WhatsApp but different interfaces (app: flexible UX, WhatsApp: conversational-only)
4. Doctor type determination: During onboarding, system determines what condition the doctor is treating the patient for (diabetes, hypertension, kidney, liver, or other)
5. Doctor connection logic:
   - If doctor is treating for diabetes: Patient is connected to doctor as diabetes specialist, full doctor-patient relationship established
   - If doctor is treating for other condition: Doctor becomes referring doctor only, patient onboarded without diabetes specialist connection
6. Patient profile creation: basic info (name, age, diabetes type, diagnosis date, referring doctor info if applicable)
7. Doctor connection: patient links to doctor via QR code scan (doctor already initiated)
8. Management mode selection: patient-primary, shared management, or caregiver-primary
9. Health records upload option: patient can upload health records during onboarding or anytime later
10. Trial period activation: If patient referred by non-diabetes specialist, 1-month free trial automatically activated
11. Patient data stored in both local (SQLite - app only) and cloud (PostgreSQL) databases
12. Authentication token generation (JWT) for app, web dashboard, and WhatsApp access
13. Multi-channel access: patient can use both app and WhatsApp immediately after onboarding (no channel selection required)
14. Onboarding completion confirmation message sent to channel used for registration (app or WhatsApp)
15. Patient can start using either channel at any time - both are always available
16. Error handling for invalid doctor codes, duplicate registrations, and incomplete profiles
17. Referring doctor information stored: name, phone number, treating condition for reverse acquisition purposes

## Story 1.4a: Caregiver Enrollment Without Doctor

As a caregiver,
I want to enroll in ProCare and pay for the service for my parent,
so that I can manage their diabetes care even if they don't have a connected doctor.

**Acceptance Criteria:**
1. Caregiver registration flow: Caregiver can register via mobile app or WhatsApp with credentials (name, email, phone, relationship to patient)
2. Patient linking: Caregiver links to patient account via patient phone number and verification code (provided by patient)
3. Payment by caregiver: Caregiver pays subscription fees for patient's ProCare account
4. Value proposition for caregiver: System provides value specifically for caregiver (peace of mind, monitoring, alerts, insights)
5. Caregiver dashboard access: Caregiver gains full access to caregiver dashboard with patient monitoring capabilities
6. No doctor requirement: Patient does not need to have a connected doctor for caregiver enrollment
7. Caregiver-primary mode: System supports caregiver-primary management mode where caregiver manages patient account
8. Caregiver authentication: OAuth 2.0 or JWT-based authentication for secure login
9. Caregiver data storage: Caregiver profiles stored in PostgreSQL database, linked to patient
10. Access control: Caregivers can only access linked patient's data, not other patients
11. Caregiver payment tracking: System tracks caregiver payments and subscription status
12. Caregiver value features: Dashboard highlights caregiver-specific value (alerts, monitoring, peace of mind, insights)

## Story 1.4b: Self Enrollment

As a patient,
I want to enroll in ProCare without a doctor referral,
so that I can start managing my diabetes independently.

**Acceptance Criteria:**
1. Self-registration flow: Patient can register via mobile app or WhatsApp without doctor referral
2. Patient registration: phone number verification (OTP via SMS/WhatsApp), basic info (name, age, diabetes type, diagnosis date)
3. No doctor connection: Patient onboarded without diabetes specialist connection (same as referral doctor workflow)
4. Trial period: Self-enrolled patients receive 1-month free trial automatically
5. Payment by patient: Patient pays subscription fees directly (no revenue share to doctors)
6. Self-enrollment identification: System identifies self-enrolled patients and applies appropriate workflows
7. Patient profile creation: basic info stored in database without doctor connection
8. Management mode selection: patient-primary, shared management, or caregiver-primary
9. Health records upload option: patient can upload health records during onboarding or anytime later
10. Patient data stored in both local (SQLite - app only) and cloud (PostgreSQL) databases
11. Authentication token generation (JWT) for app, web dashboard, and WhatsApp access
12. Multi-channel access: patient can use both app and WhatsApp immediately after onboarding
13. Onboarding completion confirmation: "Welcome to ProCare! You have 1 month free trial. Upload your prescription to get started."
14. No revenue share: Self-enrolled patients generate no referral fees (as stated in Story 8.12)
15. Doctor connection later: Patient can connect to diabetes specialist later via QR code scan, which will then activate revenue sharing if applicable

## Story 1.5: Glucose Logging via WhatsApp (Text Input)

As a patient,
I want to log my glucose reading by typing the number in WhatsApp,
so that I can quickly record my readings without friction.

**Acceptance Criteria:**
1. Natural language parsing for glucose values: recognizes "128", "sugar 150", "glucose is 140", "Mera sugar 150 hai", "109, Morning fasting"
2. Time verification: System requests time/state for glucose readings - cannot use system time directly as patient might have taken reading in morning but logging in evening. System prompts: "Please include when you took the reading, like '109, Morning fasting' or '150, After lunch'"
3. Glucose reading validation: accepts values 40-600 mg/dL, rejects invalid ranges with helpful error message
4. Timestamp handling: uses explicitly provided time/state from patient (e.g., "Morning fasting", "After lunch", "Evening"), not system timestamp
5. Glucose reading stored as event in local SQLite database (offline-first) with verified timestamp
6. Glucose reading synced to cloud PostgreSQL when connectivity available
7. Confirmation message sent to patient via WhatsApp: "Glucose logged: 128 mg/dL (Morning fasting). Looking good!"
8. Support for pre-meal, post-meal, fasting, and random glucose types (detected from context or explicitly specified like "Morning fasting")
9. Error handling for unparseable input with helpful guidance: "I couldn't understand that. Please send your glucose reading with time, like '128, Morning fasting' or 'sugar 150, After lunch'"

## Story 1.5a: Glucose Logging via Mobile App (Text Input with Pre-configured Nudges)

As a patient,
I want to log my glucose reading via mobile app with quick entry options,
so that I can log readings quickly without follow-up questions.

**Acceptance Criteria:**
1. Same natural language parsing logic as WhatsApp (Story 1.5) - shared parsing service
2. Pre-configured nudge: When number entered or button clicked for glucose logging, app shows pre-configured options like "Morning fasting", "After lunch", "Evening", etc.
3. Quick entry: Patient can select pre-configured time/state or type custom time/state
4. Glucose reading validation: accepts values 40-600 mg/dL, rejects invalid ranges with helpful error message
5. Timestamp handling: uses explicitly provided time/state from patient (e.g., "Morning fasting", "After lunch"), not system timestamp
6. Glucose reading stored as event in local SQLite database (offline-first) with verified timestamp
7. Glucose reading synced to cloud PostgreSQL when connectivity available
8. Confirmation message sent to patient via app: "Glucose logged: 128 mg/dL (Morning fasting). Looking good!"
9. Support for pre-meal, post-meal, fasting, and random glucose types (detected from context or explicitly specified)
10. Error handling for unparseable input with helpful guidance: "I couldn't understand that. Please select a time option or type your reading with time, like '128, Morning fasting'"
11. Consistent experience: app glucose logging provides same functionality as WhatsApp but with enhanced UX (pre-configured options)

## Story 1.6: Glucose Logging via WhatsApp (Voice Input)

As a patient with low literacy or tech comfort,
I want to log my glucose reading by speaking into WhatsApp,
so that I don't need to type numbers.

**Acceptance Criteria:**
1. Voice message reception via WhatsApp webhook
2. Voice-to-text conversion using speech recognition API (Google Speech-to-Text or similar) supporting Hindi and English
3. Parsed text processed through same glucose parsing logic as text input (Story 1.5) including time verification
4. Time verification: System requests time/state for glucose readings - cannot use system time directly. System prompts: "Please include when you took the reading, like '109, Morning fasting' or '150, After lunch'"
5. Glucose reading stored and synced same as text input with verified timestamp
6. Confirmation message sent: "Glucose logged: 128 mg/dL (Morning fasting). Thank you!"
7. Error handling for unclear audio with retry suggestion: "I couldn't understand your voice message. Please try again or type the number with time, like '128, Morning fasting'"
8. Support for Hindi voice input: "Mera sugar 150 hai, subah ke baad" → parsed to 150 mg/dL (Morning fasting)
9. Voice message transcription stored for debugging and improvement

## Story 1.7: Glucose Logging via WhatsApp (Photo OCR)

As a patient,
I want to log my glucose reading by taking a photo of my glucometer screen,
so that I don't need to manually type the number.

**Acceptance Criteria:**
1. Image reception via WhatsApp webhook (glucometer screen photos)
2. OCR processing using cloud OCR service (Google Vision API or similar) to extract glucose value from image
3. Image preprocessing: rotation correction, contrast enhancement, noise reduction
4. Glucose value extraction: recognizes common glucometer displays (various brands, formats)
5. Time verification: System requests time/state for glucose readings - cannot use system time directly. System prompts: "I see your glucose is [value] mg/dL. When did you take this reading? (Morning fasting, After lunch, Evening, etc.)"
6. Extracted value validated (40-600 mg/dL range) and stored with verified timestamp
7. Confirmation message sent: "Glucose logged: 128 mg/dL (Morning fasting) from your photo. Great job tracking!"
8. Error handling for unreadable images: "I couldn't read the number from your photo. Please make sure the screen is clear and well-lit, or type the number with time."
9. Image stored securely for future reference and OCR model improvement

## Story 1.8: Meal Logging via WhatsApp (Photo + Text)

As a patient,
I want to log my meals by sending a photo and optional description,
so that ProCare can learn what foods affect my glucose.

**Acceptance Criteria:**
1. Meal photo reception via WhatsApp webhook
2. Basic food recognition using TensorFlow Lite model (offline) or cloud AI (online) - recognizes 20-30 common Indian foods offline
3. Text description parsing: patient can send "rice and dal" or "roti with sabzi" along with photo
4. Meal logged as event with: photo, recognized foods, text description, timestamp
5. Meal data stored in local SQLite (offline-first) and synced to cloud when online
6. Confirmation message sent: "Meal logged: Rice and Dal. I'm learning what works for you!"
7. Support for multiple meals per day: breakfast, lunch, dinner, snacks
8. Error handling for unrecognizable photos with suggestion: "I couldn't identify the food. Can you describe what you're eating?"

## Story 1.8a: Meal Logging via Web Chat (Photo + Text)

As a patient,
I want to log my meals by uploading a photo and optional description via web chat,
so that I can use the web interface for meal logging.

**Acceptance Criteria:**
1. Meal photo upload via web chat interface (file upload or drag-and-drop)
2. Same food recognition logic as WhatsApp (Story 1.8) - shared recognition service
3. Text description parsing: patient can type "rice and dal" or "roti with sabzi" along with photo
4. Meal logged as event with: photo, recognized foods, text description, timestamp
5. Meal data stored in local SQLite (offline-first) and synced to cloud when online
6. Confirmation message sent via web chat: "Meal logged: Rice and Dal. I'm learning what works for you!"
7. Support for multiple meals per day: breakfast, lunch, dinner, snacks
8. Error handling for unrecognizable photos with suggestion: "I couldn't identify the food. Can you describe what you're eating?"
9. Consistent experience: web chat meal logging provides same functionality and responses as WhatsApp

## Story 1.9: Photo Input Recognition & Automatic Routing

As a patient,
I want to upload photos of various types (BP reading, health tracker reading, food photo, glucometer reading, prescription),
so that the system automatically recognizes the type and takes appropriate action.

**Acceptance Criteria:**
1. Photo upload interface: patients can upload photos via mobile app or WhatsApp
2. Photo type detection: system automatically discerns between BP reading, health tracker reading, food photo, glucometer reading, prescription
3. BP reading recognition: system recognizes BP monitor screens and extracts systolic/diastolic values
4. Health tracker reading recognition: system recognizes health tracker displays (steps, heart rate, etc.) if not integrated
5. Food photo recognition: system recognizes food photos and routes to meal logging (Story 1.8)
6. Glucometer reading recognition: system recognizes glucometer screens and routes to glucose logging (Story 1.7)
7. Prescription recognition: system recognizes prescription documents and routes to prescription upload (Epic 11)
8. Automatic routing: system automatically routes photo to appropriate module based on recognition
9. Appropriate action: system takes appropriate action for each photo type (stores BP reading, logs meal, logs glucose, extracts prescription data)
10. Storage: each photo type stored in relevant place (BP readings in health records, meals in meal log, etc.)
11. Error handling: if photo type cannot be determined, system prompts user to specify type
12. Photo metadata: system stores photo metadata (type, timestamp, source) for reference

## Story 1.10: Meal Inference from Glucose Reading

As a patient,
I want the system to help me understand what might have caused a high glucose reading,
so that I can learn from my patterns even if I forgot to log a meal.

**Acceptance Criteria:**
1. Meal inference trigger: if previous meal data not logged and current blood glucose reading is high/unusual
2. Glucose analysis: system analyzes current glucose reading against patient's usual patterns
3. Inference prompt: system prompts patient: "Sugar is 210. That's higher than usual. What could be the reason? Larger portion / different food or Missed medication?"
4. Inference options: system provides multiple inference options (larger portion, different food, missed medication, other)
5. Patient response: patient can select inference option or provide additional context
6. Learning: system learns from patient responses to improve inference accuracy
7. Pattern correlation: system correlates inferred meals with glucose outcomes to improve future inferences
8. Meal logging reminder: system reminds patient to log meals to improve accuracy
9. Inference display: inferred meals shown in meal history with "inferred" tag
10. Inference accuracy: system tracks inference accuracy and improves over time

**Note for Stories 1.5-1.10:** The UI and UX of the app has to be new age and personalised. There might be a better way to log glucose / meals etc and maybe give hot cards that keep changing with time of day / previous usage / next actions / pending actions etc - which can be used on the home page. The interactions can happen via these hot buttons in a faster and simpler UX and the result can be put on the chat interface for further discussions.

## Story 1.11: Data Sync Engine (Offline to Online)

As a patient,
I want my offline logs automatically synced when I get internet,
so that my doctor can see my data and I don't lose any information.

**Acceptance Criteria:**
1. Connectivity detection: monitors online/offline status
2. Event queue: stores all patient actions (glucose logs, meal logs) locally when offline
3. Automatic sync trigger: when connectivity restored, sync engine processes queued events
4. Conflict resolution: device timestamp used as source of truth, duplicate detection prevents re-syncing same events
5. Sync status feedback: patient receives confirmation "All your data has been synced!" after successful sync
6. Partial sync support: critical data (emergencies) synced first, then full data sync
7. Error handling: failed syncs retried with exponential backoff, events remain in queue until successful
8. Sync progress tracking: logs sync status for monitoring and debugging

## Story 1.12: Trial Period Management for Referred Patients

As a patient referred by a non-diabetes specialist,
I want a 1-month free trial to experience ProCare,
so that I can evaluate the service before subscribing.

**Acceptance Criteria:**
1. Trial period activation: Patients referred by non-diabetes specialists automatically receive 1-month free trial
2. Trial start date: Trial starts from onboarding completion date
3. Trial duration tracking: System tracks trial days remaining and displays to patient
4. Trial status display: Patient sees trial status in app and receives trial expiration reminders (7 days, 3 days, 1 day before expiration)
5. Full access during trial: Patients have full access to all features during trial period (glucose logging, meal logging, health records, diet module, caregiver access, AI questions)
6. Trial expiration notification: Patient receives notification when trial expires: "Your 1-month free trial has ended. Subscribe now to continue using ProCare."
7. Trial-to-paid conversion: Patient can subscribe anytime during trial or after expiration
8. Trial extension: System can extend trial period for special cases (admin override)
9. Trial analytics: System tracks trial conversion rates, trial usage patterns, and conversion triggers
10. Trial reminder: Patient receives reminder 7 days before expiration with subscription options

## Story 1.13: Subscription Restrictions After Trial Period

As a patient who hasn't subscribed after trial,
I want to understand what access I lose,
so that I can make an informed decision about subscribing.

**Acceptance Criteria:**
1. Subscription check: System checks subscription status before allowing access to restricted features
2. Restricted features after trial expiration (if not subscribed):
   - Uploaded health data: Patient can access previously uploaded health records only 2 times a year (configurable). This includes online viewing by the doctor.
   - Prescriptions: Patient cannot view or download prescriptions, but can upload
   - Online interface: Patient can use online interface to show data to doctors - subject to the earlier point about restricted number of views of overall data
   - Diet module recipes: Patient cannot access recipe recommendations
   - Food swap ideas are to be restricted
   - Food insights become generic
   - Menu scan feature: Patient cannot use menu scan feature for restaurant recommendations
   - Caregiver module: Caregiver loses access to patient data & the caregiver also gets warning about subscriptions with details about what all will be lost
3. Free features (always available):
   - Basic glucose logging (stays unlimited)
   - Basic meal logging (remains unlimited)
   - Basic AI questions (limited to 3 a week)
4. Access restriction messaging: Clear messaging when patient tries to access restricted feature: "This feature requires a subscription. Subscribe now to continue using ProCare."
5. Subscription prompt: Patient sees subscription options when accessing restricted features
6. Data retention: Patient data retained for 90 days after trial expiration, then archived (can be restored upon subscription)
7. Subscription restoration: When patient subscribes, all previously restricted features and data are immediately restored
8. Grace period: 7-day grace period after trial expiration where patient can still view data but cannot add new data
9. Subscription upgrade flow: Patient can upgrade to paid subscription from any restricted feature screen
10. Access logging: System logs all restricted feature access attempts for analytics

## Story 1.14: Payment Gateway Integration

As a patient,
I want to pay for ProCare subscription through the app,
so that I can easily manage my subscription and maintain access after the trial period.

**Acceptance Criteria:**
1. Payment gateway integration: System integrates with Razorpay payment gateway (test mode for MVP development, production mode for launch)
2. Payment flow: Payment flow happens through ProCare app (not external redirect) using Razorpay SDK
3. Subscription plans: Three subscription tiers available (₹99/month basic, ₹299/month standard, ₹999/month premium)
4. Payment methods: Supports credit cards, debit cards, UPI, net banking, wallets via Razorpay
5. Payment security: All payments processed securely with PCI compliance, encryption in transit (TLS 1.3)
6. Payment confirmation: Patients receive payment confirmation via app notification and WhatsApp receipt
7. Subscription status display: Patients can view subscription status (active/expired), current plan, renewal date in app profile
8. Subscription management: Patients can upgrade plan (immediate effect), downgrade plan (next billing cycle), view payment history
9. Auto-renewal: Subscriptions auto-renew with payment on file, reminder sent 3 days before renewal
10. Payment failure handling: Handles payment failures (expired cards, insufficient funds) with retry logic and grace period (3 days)
11. Revenue sharing calculation: System calculates and tracks revenue sharing for referring doctors (30%) and diabetes specialists (60%) per FR35
12. Payment reporting: Payment data tracked for business analytics (conversion rates, churn, revenue by plan)
13. Razorpay webhook integration: Webhook endpoint handles payment success, failure, refund events
14. Refund support: Admin interface for processing refunds when needed
15. Payment timing: Integration must be complete before trial period expiration (Story 1.12) to enable subscription conversion

## Story 1.15: MVP Validation Pilot Program

As a product owner,
I want to validate MVP assumptions with real users before full launch,
so that we can measure baseline metrics and validate technical performance.

**Acceptance Criteria:**
1. Pilot program scope: 50 participants total (25 patients, 10 doctors, 5 caregivers, 10 referring doctors)
2. Pilot duration: 4 weeks minimum (allows pattern detection validation per Epic 4)
3. Pilot recruitment: Recruit through personal networks, partner clinics, diabetes communities
4. Baseline metric measurement: Establish baselines for NFRs before pilot:
   - Current medication adherence rate (target: establish <50% baseline to measure 85%+ improvement)
   - Current doctor time per patient (target: establish baseline to measure 10-15 minute reduction)
   - Current emergency calls per doctor per week (target: establish 3-5/week baseline to measure 60% reduction)
5. AI accuracy validation: Measure AI question answering accuracy against medical advisor review (target: 80%+ for routine questions per NFR6)
6. Pattern detection validation: Measure pattern detection success rate (target: 90%+ of patients have detectable patterns within 4 weeks per NFR19)
7. HbA1c prediction validation: Measure HbA1c prediction accuracy against lab results (target: 95%+ within 0.2% per Epic 8 Story 8.2 AC#3)
8. Emergency detection validation: Measure emergency detection true positive rate and false alarm rate (target: 98%+ true positive, <2% false alarm per NFR4)
9. Retention metric validation: Measure Week 1 retention rate (target: 75%+ per NFR14)
10. Technical performance validation: Load test 7-agent orchestration at 1,000 concurrent users (per NFR3)
11. Offline sync validation: Test offline sync scenarios with multiple devices, validate conflict resolution, measure sync success rate
12. User feedback collection: Conduct user interviews (patients, doctors, caregivers) to validate UX, identify pain points, gather feature requests
13. Pilot success criteria: Define success criteria for MVP launch readiness:
    - 80%+ AI accuracy achieved
    - 90%+ pattern detection within 4 weeks
    - 98%+ emergency detection accuracy
    - <2 second AI response time
    - <500ms dashboard load time
    - Zero critical bugs in pilot
14. Pilot report: Comprehensive pilot report with metrics, findings, recommendations, and launch readiness assessment
15. Launch decision: Go/no-go decision for MVP launch based on pilot results and success criteria
