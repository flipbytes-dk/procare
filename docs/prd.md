# ProCare Product Requirements Document (PRD)

## Goals and Background Context

### Goals

- Enable Type 2 diabetes patients in India to reduce HbA1c from 8.5% to <7.0% through personalized, AI-powered guidance
- Achieve 85%+ medication adherence (vs 50% baseline) through smart reminders and caregiver involvement
- Provide immediate, personalized feedback to reduce patient anxiety ("Is this glucose reading okay?")
- Enable doctors to save 10-15 minutes daily while improving patient outcomes through passive monitoring
- Reduce emergency calls to doctors by 60% (from 3-5/week to 1-2/week) through proactive pattern-based interventions
- Achieve 75%+ Week 1 retention through exceptional first-week experience (2+ personalized insights, quick answers, doctor connection)
- Support 100,000 registered users and 30,000 paid subscribers in Year 1
- Generate ₹30L monthly recurring revenue (MRR) from B2C subscriptions + ₹10.8L monthly from doctor commissions
- Establish WhatsApp-first architecture as primary channel (70%+ users start on WhatsApp, 2-3 minute onboarding)
- Enable offline-first functionality so 95%+ patients have <24hr stale data, expanding addressable market to 500M+ users
- Detect 90%+ of patients' food-glucose patterns within 4 weeks to enable proactive interventions
- Achieve 80%+ AI question answering accuracy (routine questions) with 20% escalation to doctors

### Background Context

ProCare addresses a critical healthcare gap in India's diabetes management landscape. With 77 million Type 2 diabetes patients, India faces a crisis where only 50% adhere to medications, 30% check glucose regularly, and average HbA1c levels sit at 8.5% (well above the 7.0% target). The problem stems from reactive medicine—doctors see patients every 3 months with no visibility into behavior between visits, while patients experience daily anxiety with no one to ask "Is this okay?" or "Can I eat mango?"

Existing solutions fail because they require app downloads (barrier for 70% of market), don't work offline (excludes 500M+ potential users), provide generic advice rather than personalized insights, lack doctor connection (patients want oversight), and use judgmental tones that increase guilt. International platforms aren't designed for India's unique needs: WhatsApp-first communication, intermittent connectivity, family involvement in care, and affordability (₹500-1,000/month vs ₹2,000-5,000).

ProCare solves this through a 7-agent orchestrated platform that combines WhatsApp-first accessibility (550M+ users, zero friction), offline-first architecture (works without internet), doctor-connected care (not AI-only), family/caregiver involvement (2.3x adherence improvement), and proactive pattern-based interventions (preventive vs reactive). The platform learns each patient's unique patterns over 4 weeks, then provides pre-meal warnings like "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"—preventing problems rather than responding after they occur.

### Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2024-11 | 1.0 | Initial PRD creation from Project Brief | PM (John) |
| 2024-11 | 1.1 | Added web chat interface (Open WebUI) as alternative communication channel alongside WhatsApp | PM (John) |

## Requirements

### Functional

FR1: The system shall provide a WhatsApp-first conversational interface for all patient interactions including glucose logging, meal logging, medication reminders, and question answering.

FR2: The system shall support multiple glucose logging input methods: text input (e.g., "128"), voice message (Hindi/English), and photo OCR for glucometer screens.

FR3: The system shall provide smart medication reminders at doctor-specified times with escalation logic: reminder → nudge → caregiver notification → doctor alert.

FR4: The system shall support simplified meal logging via photo upload and text description (no detailed nutrition calculations), with basic food recognition (20-30 items offline, full recognition online).

FR5: The system shall detect food-glucose correlations, activity impacts, and medication consistency patterns after 4 weeks of data collection.

FR6: The system shall provide proactive pattern-based interventions after pattern detection, including pre-meal warnings (e.g., "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?").

FR7: The system shall detect emergency glucose levels (hypoglycemia <70 mg/dL, hyperglycemia >250 mg/dL) and provide immediate instructions with automatic doctor escalation.

FR8: The system shall answer routine patient questions via AI (nutrition, timing, general diabetes management) and escalate medication/medical questions to doctors.

FR9: The system shall generate weekly progress reports for patients (wins, opportunities, insights) and doctors (status, metrics, flags).

FR10: The system shall manage HbA1c test appointments with 2-week reminders, HbA1c predictions, lab appointment booking, and result upload/analysis via OCR.

FR11: The system shall provide a doctor web dashboard with tiered patient view (Urgent/Attention/Stable), weekly summaries, and alert system with context and suggested actions.

FR12: The system shall provide a separate caregiver dashboard (WhatsApp + web) with real-time critical alerts, daily digest, and caregiver-primary management option.

FR13: The system shall support doctor-initiated engagement where doctors set expectations (target A1C, medication schedule, lifestyle goals) and ProCare creates personalized programs.

FR14: The system shall implement a 7-agent orchestration system: Engagement Engine (orchestrator), Clinical Monitor, Lifestyle Coach, Health Insights, Doctor Bridge, Care Coordinator, and Learning Library.

FR15: The system shall bundle multiple agent responses into single patient messages and enforce rate limiting (max 5 notifications/day, 2-hour minimum spacing).

FR16: The system shall work fully offline for core features (glucose logging, meal logging, medication reminders) with local database caching 90 days of data.

FR17: The system shall sync offline data when connectivity is restored using event-sourcing model with conflict resolution (device timestamp as source of truth).

FR18: The system shall support patient signup with management mode selection: patient-primary, shared management, or caregiver-primary.

FR19: The system shall deliver context-based health education tips (not comprehensive libraries) triggered by patient behavior patterns.

FR20: The system shall track medication adherence and provide adherence metrics to doctors and caregivers.

FR21: The system shall provide a web-based chat interface (accessible via browser without app download) as an alternative communication channel for patients, supporting the same core functionality as WhatsApp (glucose logging, meal logging, medication reminders, question answering). Both WhatsApp and web chat are always available—patients can use either channel at any time without selection or switching. Messages route to the channel where patient last interacted, with consistent conversational experience across both channels.

### Non Functional

NFR1: The system shall achieve 95%+ of patients having <24hr stale data when offline functionality is used.

NFR2: The system shall respond to AI questions within 2 seconds via WhatsApp and load doctor dashboard pages within 500ms.

NFR3: The system shall handle 1,000+ concurrent users with the 7-agent orchestration system.

NFR4: The system shall achieve 98%+ true positive rate for emergency detection with <2% false alarm rate.

NFR5: The system shall achieve 95%+ glucose prediction accuracy within 15 mg/dL for pattern-based interventions.

NFR6: The system shall achieve 80%+ AI question answering accuracy for routine questions (nutrition, timing, general).

NFR7: The system shall encrypt all patient health data with AES-256 at rest and TLS 1.3 in transit.

NFR8: The system shall comply with India data protection laws, HIPAA-equivalent PHI handling, audit logs, and access controls.

NFR9: The system shall support WhatsApp Business API rate limits and handle 10K+ users without degradation.

NFR10: The system shall support Hindi and English languages for MVP (Tamil, Telugu in Phase 2).

NFR11: The system shall maintain 90-day full data cache locally (~260MB) with 1-year summary and >1 year stats only.

NFR12: The system shall implement bandwidth-aware sync: WiFi (full), 4G (compressed), 2G/3G (critical only).

NFR13: The system shall provide doctor dashboard accessible via web browsers (Chrome 90+, Safari 14+, Firefox 88+) without requiring mobile app download.

NFR14: The system shall achieve 75%+ Week 1 retention through exceptional first-week experience (2+ personalized insights, quick answers, doctor connection).

NFR15: The system shall achieve 85%+ medication adherence (vs 50% baseline) through smart reminders and caregiver involvement.

NFR16: The system shall reduce emergency calls to doctors by 60% (from 3-5/week to 1-2/week per doctor).

NFR17: The system shall enable doctors to review patient status in 10-15 minutes daily maximum (10 minutes Monday morning, 5 minutes during week).

NFR18: The system shall achieve 70%+ of users starting on WhatsApp with 2-3 minute onboarding time.

NFR19: The system shall detect patterns for 90%+ of patients within 4 weeks of data collection.

NFR20: The system shall achieve 80%+ doctor dashboard weekly usage and 85%+ caregiver dashboard daily usage.

## User Interface Design Goals

### Overall UX Vision

ProCare's user experience centers on reducing anxiety and friction for diabetes management. The primary interface is WhatsApp—familiar, accessible, and requiring zero app downloads. Patients can also use a web-based chat interface (Open WebUI or similar) accessible via browser at any time. Both channels are always available—patients can use either WhatsApp or web chat without any selection or switching. Both channels provide the same conversational experience—supportive and personalized, never judgmental. The system routes messages to the channel where patient last interacted, ensuring continuity. The system anticipates needs (proactive interventions) rather than reacting after problems occur. Same account, same data across both channels. For doctors and caregivers, web dashboards provide at-a-glance visibility with minimal time investment (10-15 minutes daily for doctors, peace-of-mind checks for caregivers). The offline-first architecture ensures the experience never breaks, even with intermittent connectivity—users can log glucose, meals, and medications seamlessly whether online or offline.

### Key Interaction Paradigms

**Conversational Interface (WhatsApp):**
- Natural language input (text, voice) for all patient interactions
- Photo sharing for meal logging and glucometer readings
- Context-aware responses that feel like talking to a knowledgeable friend
- Proactive messages (not just reactive) with pattern-based interventions
- Rate-limited notifications (max 5/day, 2-hour spacing) to avoid overwhelm

**Web Chat Interface (Browser-Based):**
- Web-based chat UI accessible via browser without app download (using Open WebUI or similar framework)
- Same conversational experience as WhatsApp with consistent functionality
- Natural language input (text) for all patient interactions
- Photo upload for meal logging and glucometer readings
- Context-aware responses matching WhatsApp experience
- Proactive messages and pattern-based interventions
- Works offline-first with local data storage and sync
- Mobile-responsive design for smartphone browsers

**Tiered Information Architecture (Doctor Dashboard):**
- Urgent/Attention/Stable patient categorization for quick triage
- Weekly batch summaries (Monday morning review) vs real-time alerts
- Context-rich alerts with suggested actions (not just data dumps)
- Minimal clicks to understand patient status and take action

**Caregiver Peace-of-Mind Model:**
- Real-time critical alerts (hypoglycemia, missed meds) for immediate awareness
- Daily digest summaries for routine monitoring
- Separate caregiver view (not patient's view) to respect independence
- Setup and management primarily on caregiver's device (not elderly parent's)

**Offline-First Interaction:**
- All core actions (logging, reminders) work identically offline
- Visual indicators for sync status (synced, syncing, offline)
- Graceful degradation (full features offline, enhanced features online)
- Seamless sync when connectivity returns (no user action required)

### Core Screens and Views

**Patient Experience (WhatsApp):**
- Onboarding flow (2-3 minutes: phone number, doctor connection, management mode selection)
- Conversational logging interface (glucose, meals, medications via natural language)
- Question-answering interface (AI responses with doctor escalation option)
- Weekly progress summary (delivered via WhatsApp message)

**Patient Experience (Web Chat Interface):**
- Web chat login/authentication (phone number + OTP)
- Onboarding flow via web chat (same as WhatsApp: phone number, doctor connection, management mode selection)
- Conversational logging interface (glucose, meals, medications via natural language)
- Question-answering interface (AI responses with doctor escalation option)
- Weekly progress summary (delivered via web chat)
- Chat history and message persistence
- Photo upload interface for meal logging and glucometer readings

**Doctor Web Dashboard:**
- Login/Authentication screen
- Main dashboard with tiered patient view (Urgent/Attention/Stable tabs)
- Patient detail view (glucose trends, medication adherence, alerts, weekly summary)
- Alert management interface (urgent alerts with context and suggested actions)
- Patient onboarding interface (set target A1C, medication schedule, lifestyle goals)

**Caregiver Web Dashboard:**
- Login/Authentication screen
- Main dashboard with patient status overview (current status, recent alerts, adherence metrics)
- Real-time alert feed (critical alerts: hypoglycemia, missed medications)
- Daily digest view (glucose summary, medication adherence, meal patterns)
- Patient detail view (trends, patterns, doctor notes)

**Caregiver WhatsApp Interface:**
- Critical alert notifications (hypoglycemia, missed meds)
- Daily digest summary (delivered via WhatsApp message)

**Shared Components:**
- Offline status indicator (visible when offline, sync status when online)
- Language selector (Hindi/English for MVP)

### Accessibility: WCAG AA

ProCare must meet WCAG AA standards to serve India's diverse user base, including elderly patients (60+) and users with varying tech literacy. Key accessibility requirements:
- Voice input/output support for low-literacy users
- High contrast text and clear visual hierarchy
- Keyboard navigation for web dashboards
- Screen reader compatibility for web interfaces
- Simple, clear language (avoid medical jargon)
- Large touch targets for mobile web interfaces
- Photo-based input (glucometer screens, meals) reduces text input barriers

### Branding

ProCare's brand identity emphasizes support, not judgment. Tone is warm, encouraging, and personalized—like a trusted friend who understands diabetes management. Visual design should feel:
- **Medical but approachable:** Professional healthcare credibility without clinical coldness
- **Indian context-aware:** Colors, imagery, and language that resonate with Indian users
- **Accessibility-first:** High contrast, clear typography, simple layouts
- **WhatsApp-native:** Feels natural within WhatsApp's interface (not like an external app)
- **Offline-resilient:** Visual cues that reinforce reliability (sync status, offline indicators)

No specific brand guidelines provided yet—this will be developed during design phase with focus on reducing anxiety and increasing trust.

### Target Device and Platforms: Cross-Platform

**Primary Platform: WhatsApp (iOS 13+, Android 8+)**
- Conversational interface accessible to 550M+ WhatsApp users in India
- Zero app download required
- Works on any smartphone with WhatsApp installed

**Secondary Platform: Web Portal (Responsive)**
- **Patient Web Chat Interface:** Browser-based chat UI (Open WebUI or similar) accessible via mobile or desktop browsers, supporting same functionality as WhatsApp
- Doctor dashboard: Desktop-first (Chrome 90+, Safari 14+, Firefox 88+)
- Caregiver dashboard: Mobile-responsive (iOS 13+, Android 8+)
- Progressive Web App (PWA) capabilities for app-like experience without app store

**Future Platform: Native Mobile Apps (Phase 2)**
- iOS and Android native apps for power users
- Enhanced data visualization and offline AI models
- Target: 5% of users (tech-savvy, newer smartphones)

**Cross-Platform Considerations:**
- Consistent experience across WhatsApp and web chat (same data, same insights, same conversational flow)
- Patients can use either WhatsApp or web chat at any time - both channels are always available (no selection required)
- Messages route to the channel where patient last interacted, or patient can use either channel simultaneously
- Same account, same data across both channels - seamless multi-channel support
- Offline-first architecture works identically across all platforms
- Responsive design ensures usability on wide range of screen sizes (4" to desktop)
- Web chat interface provides alternative for users who prefer browser-based interaction or don't use WhatsApp

## Technical Assumptions

### Repository Structure: Monorepo

ProCare will use a monorepo approach with a single repository containing multiple services (agents). This enables:
- Shared libraries for common utilities (database models, API clients, event schemas)
- Easier code sharing between agents and frontend/backend
- Simplified dependency management across the 7-agent orchestration system
- Atomic commits across multiple services when needed
- Consistent tooling and CI/CD pipelines

**Rationale:** The 7-agent architecture requires tight coordination and shared data models. A monorepo simplifies development while maintaining service independence. Each agent remains an independent microservice but shares common infrastructure code.

### Service Architecture

ProCare will use a **Microservices architecture** with the following structure:

**Core Services:**
- **Engagement Engine (Orchestrator):** Central service that routes all patient events, manages notifications, tracks patient state, and synthesizes multi-agent responses into single messages
- **7 Agent Services (Independent Microservices):**
  - Clinical Monitor (tracks glucose, medications, detects emergencies)
  - Lifestyle Coach (analyzes meals, tracks activity, behavioral nudges)
  - Health Insights (pattern detection, generates insights, validates predictions)
  - Doctor Bridge (routes questions, manages doctor alerts, enhances messages with context)
  - Care Coordinator (test appointments, medication refills, lab integration)
  - Learning Library (targeted health education, context-based content)

**Supporting Infrastructure:**
- **API Gateway:** Single entry point for all external requests (WhatsApp webhooks, web dashboard APIs), handles rate limiting and authentication
- **Message Bus:** Event-driven communication between agents (RabbitMQ or Redis)
- **Sync Engine:** Handles offline-first data synchronization with conflict resolution

**Communication Pattern:**
- Event-driven architecture: Patient action → Engagement Engine → Route to agents → Synthesize response → Patient
- Priority hierarchy: Emergencies override everything (Clinical Monitor takes control)
- Notification bundling: Multiple agent responses → Single patient message
- Rate limiting: Max 5 notifications/day, 2-hour minimum spacing

**Rationale:** Microservices enable independent scaling of each agent, fault isolation, and technology flexibility. The Engagement Engine acts as orchestrator to maintain consistency while allowing agents to evolve independently. This architecture supports the complex 7-agent orchestration while maintaining scalability.

### Testing Requirements

ProCare will implement **Full Testing Pyramid** with the following levels:

**Unit Tests:**
- Individual agent logic (pattern detection algorithms, emergency detection rules, question classification)
- Data transformation functions (glucose parsing, meal recognition, sync conflict resolution)
- Business logic validation (medication adherence calculation, pattern confidence scoring)
- Target: 80%+ code coverage for critical paths

**Integration Tests:**
- Agent-to-agent communication via message bus
- Engagement Engine orchestration flows (patient action → agent routing → response synthesis)
- Database operations (CRUD, queries, sync operations)
- WhatsApp API integration (webhook handling, message sending, media uploads)
- Offline sync engine (conflict resolution, duplicate detection, event replay)

**End-to-End Tests:**
- Complete patient workflows (glucose logging → pattern detection → proactive intervention)
- Doctor dashboard workflows (patient onboarding → alert review → patient messaging)
- Caregiver workflows (alert receipt → dashboard check → patient status)
- Emergency detection and escalation flows
- Offline-to-online sync scenarios

**Manual Testing:**
- WhatsApp conversational flows (natural language variations, voice messages, photo uploads)
- Multi-language support (Hindi/English) with native speakers
- Accessibility testing (WCAG AA compliance, screen readers, keyboard navigation)
- User acceptance testing with real patients, doctors, and caregivers

**Rationale:** Healthcare applications require rigorous testing due to patient safety implications. Unit tests ensure individual components work correctly. Integration tests validate agent orchestration and data flow. E2E tests ensure complete workflows function end-to-end. Manual testing validates user experience, accessibility, and real-world usage patterns. The offline-first architecture adds complexity requiring extensive sync testing.

### Additional Technical Assumptions and Requests

**Frontend Technologies:**
- **Web Portal:** React 18+ for component reusability and large ecosystem
- **Patient Web Chat Interface:** Open WebUI or similar web-based chat framework for browser-accessible conversational interface
- **Mobile Web:** Progressive Web App (PWA) for app-like experience without app store
- **Future Native App:** React Native for code sharing with web (Phase 2)

**Backend Technologies:**
- **API Layer:** Node.js/Express or Python/FastAPI (fast development, good AI integration)
- **Message Queue:** RabbitMQ or Redis for agent communication and async processing
- **API Gateway:** Kong or AWS API Gateway for rate limiting, authentication, routing

**Database Technologies:**
- **Primary Database:** PostgreSQL for structured data, ACID compliance, doctor dashboards
- **Local Database (Mobile/Offline):** SQLite for offline-first 90-day cache
- **Cache Layer:** Redis for session management and real-time data
- **Time-Series Database:** InfluxDB or TimescaleDB for glucose readings and pattern analysis

**AI/ML Technologies:**
- **Conversational AI:** OpenAI GPT-4 or Anthropic Claude for question answering and natural language processing
- **Image Recognition:** Custom TensorFlow model (TensorFlow Lite for offline, cloud for online)
- **Pattern Detection:** Scikit-learn or PyTorch for correlation analysis and predictions
- **On-Device ML:** TensorFlow Lite for offline food recognition and basic pattern detection

**Hosting/Infrastructure:**
- **Cloud Provider:** AWS or GCP with India regions for low latency
- **Containerization:** Docker + Kubernetes for scalable agent isolation
- **CDN:** CloudFront or Cloudflare for static assets and global distribution
- **Monitoring:** Datadog or New Relic for agent performance and user analytics

**Integration Requirements:**
- **WhatsApp Business API:** Webhook handling, message sending, media uploads (approval required, 2-4 weeks)
- **Web Chat Interface:** Open WebUI or similar framework for browser-based chat, WebSocket/SSE for real-time messaging
- **Lab Partners:** Apollo, Metropolis APIs for appointment booking and results
- **Payment Gateway:** Razorpay or Stripe for subscription management
- **SMS/Email:** Twilio or SendGrid for backup notifications and OTP

**Security/Compliance:**
- **Data Encryption:** AES-256 at rest, TLS 1.3 in transit
- **Authentication:** JWT tokens for API authentication, OAuth 2.0 for doctors
- **Compliance:** HIPAA-equivalent PHI handling, audit logs, access controls, India data protection laws
- **Regulatory:** CDSCO compliance if classified as medical device (positioning as "health information service")

**Offline-First Architecture:**
- **Event Sourcing:** Every action is immutable event (device timestamp = source of truth)
- **Sync Engine:** Conflict resolution (last-write-wins for settings, merge for additive data), duplicate detection
- **Local Database:** 90-day full data, 1-year summary, >1 year stats only (~260MB storage requirement)
- **Bandwidth-Aware Sync:** WiFi (full sync), 4G (compressed), 2G/3G (critical data only)

**Performance Requirements:**
- **Response Time:** <2 seconds for AI responses (WhatsApp), <500ms for dashboard loads
- **Sync Time:** <5 seconds for critical data, <30 seconds for full sync
- **Concurrent Users:** Handle 1,000+ concurrent users with 7-agent orchestration (scaling to 10K+ in Phase 2)
- **Offline Functionality:** 100% core features available offline

**Browser/OS Support:**
- **Web Portal:** Chrome 90+, Safari 14+, Firefox 88+ (last 2 versions)
- **Mobile Web:** iOS 13+, Android 8+ (covers 95% of Indian smartphones)
- **WhatsApp:** iOS 13+, Android 8+ (WhatsApp requirement)

**Language Support:**
- **MVP:** Hindi and English (voice recognition, text responses, UI translation)
- **Phase 2:** Tamil, Telugu, Bengali, Marathi (covers 90% of India)

## Epic List

Epic 1: Foundation & Core Patient Logging - Establish project infrastructure, WhatsApp Business API integration, offline-first database architecture, and enable patients to log glucose readings and meals via WhatsApp with basic data storage and retrieval.

Epic 2: Medication Management & Reminders - Implement smart medication reminders with doctor-specified schedules, escalation logic (reminder → nudge → caregiver → doctor), and medication adherence tracking.

Epic 3: Agent Orchestration & Emergency Detection - Build the 7-agent orchestration system with Engagement Engine as orchestrator, implement Clinical Monitor agent for emergency detection (hypoglycemia/hyperglycemia), and establish agent communication via message bus.

Epic 4: Pattern Detection & Health Insights - Implement Health Insights agent for food-glucose correlation analysis, activity impact detection, and medication consistency patterns, generating personalized insights after 4 weeks of data collection.

Epic 5: Doctor Dashboard & Patient Management - Build web-based doctor dashboard with tiered patient view (Urgent/Attention/Stable), patient onboarding interface for setting expectations, alert management with context, and patient detail views.

Epic 6: Caregiver Features & Alerts - Implement separate caregiver dashboard (web + WhatsApp), real-time critical alerts, daily digest summaries, and caregiver-primary management mode option.

Epic 7: Proactive Pattern-Based Interventions - Enable proactive interventions after pattern detection, including pre-meal warnings based on learned patterns, and implement notification bundling with rate limiting (max 5/day, 2-hour spacing).

Epic 8: Advanced Coordination & Question Answering - Implement Care Coordinator agent for HbA1c test scheduling with predictions, Doctor Bridge agent for AI question answering with escalation, weekly progress reports for patients and doctors, and Learning Library agent for context-based education.

## Epic 1: Foundation & Core Patient Logging

### Epic Goal

Establish the foundational infrastructure for ProCare including project setup, Git repository, CI/CD pipelines, and core database architecture. Integrate WhatsApp Business API and web chat interface (Open WebUI or similar) to enable conversational patient interactions via multiple channels. Implement offline-first database architecture with local SQLite storage and sync engine foundation. Enable patients to log glucose readings and meals via WhatsApp or web chat using multiple input methods (text, voice, photo), with data stored locally and synced to cloud when connectivity is available. Both WhatsApp and web chat are always available - patients can use either channel at any time without selection or switching. Messages route to the channel where patient last interacted. This epic delivers immediate patient value (working logging functionality) while establishing the technical foundation for all subsequent features.

### Story 1.1: Project Foundation & Infrastructure Setup

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

### Story 1.2: Database Architecture & Offline-First Foundation

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

### Story 1.3: WhatsApp Business API Integration

As a patient,
I want to interact with ProCare via WhatsApp,
so that I don't need to download a separate app.

**Acceptance Criteria:**
1. WhatsApp Business API account created and approved (or test environment configured)
2. Webhook endpoint implemented to receive incoming WhatsApp messages (text, voice, images)
3. Message sending functionality implemented via WhatsApp Business API
4. Media handling: receive and process images (meal photos, glucometer screens)
5. Voice message handling: receive and process audio messages (Hindi/English)
6. Message queuing system for rate limit compliance (max 5 notifications/day, 2-hour spacing)
7. Error handling for API failures, rate limits, and invalid messages
8. Webhook signature verification for security
9. Basic logging and monitoring for WhatsApp API interactions

### Story 1.3a: Web Chat Interface Integration

As a patient,
I want to interact with ProCare via a web-based chat interface,
so that I have an alternative to WhatsApp if I prefer browser-based interaction.

**Acceptance Criteria:**
1. Web chat interface framework integrated (Open WebUI or similar) for browser-based conversational interface
2. WebSocket or Server-Sent Events (SSE) connection for real-time messaging
3. Chat interface accessible via web browser (mobile and desktop) without app download
4. Message sending and receiving functionality via web chat API
5. Photo upload handling: receive and process images (meal photos, glucometer screens) via web interface
6. Chat history persistence: messages stored and displayed in chat interface
7. Message queuing system for rate limit compliance (max 5 notifications/day, 2-hour spacing) - same as WhatsApp
8. Error handling for connection failures, timeouts, and invalid messages
9. Authentication integration: web chat uses same authentication system as WhatsApp (phone + OTP)
10. Consistent experience: web chat provides same functionality and conversational flow as WhatsApp
11. Offline support: web chat works offline with local message storage and sync when connectivity restored
12. Basic logging and monitoring for web chat interactions

### Story 1.4: Patient Onboarding & Authentication

As a new patient,
I want to sign up for ProCare via WhatsApp or web chat,
so that I can start managing my diabetes immediately.

**Acceptance Criteria:**
1. Patient registration flow via WhatsApp or web chat: phone number verification (OTP via SMS/WhatsApp)
2. Patient profile creation: basic info (name, age, diabetes type, diagnosis date)
3. Doctor connection: patient can link to doctor via doctor code or phone number
4. Management mode selection: patient-primary, shared management, or caregiver-primary
5. Patient data stored in both local (SQLite) and cloud (PostgreSQL) databases
6. Authentication token generation (JWT) for web dashboard and web chat access
7. Multi-channel access: patient can use both WhatsApp and web chat immediately after onboarding (no channel selection required)
8. Onboarding completion confirmation message sent to channel used for registration (WhatsApp or web chat)
9. Patient can start using either channel at any time - both are always available
10. Error handling for invalid doctor codes, duplicate registrations, and incomplete profiles

### Story 1.5: Glucose Logging via WhatsApp (Text Input)

As a patient,
I want to log my glucose reading by typing the number in WhatsApp,
so that I can quickly record my readings without friction.

**Acceptance Criteria:**
1. Natural language parsing for glucose values: recognizes "128", "sugar 150", "glucose is 140", "Mera sugar 150 hai"
2. Glucose reading validation: accepts values 40-600 mg/dL, rejects invalid ranges with helpful error message
3. Timestamp handling: uses message timestamp if provided, otherwise current device time
4. Glucose reading stored as event in local SQLite database (offline-first)
5. Glucose reading synced to cloud PostgreSQL when connectivity available
6. Confirmation message sent to patient via WhatsApp: "Glucose logged: 128 mg/dL at 8:30 AM. Looking good!"
7. Support for pre-meal, post-meal, fasting, and random glucose types (detected from context or explicitly specified)
8. Error handling for unparseable input with helpful guidance: "I couldn't understand that. Please send your glucose reading as a number, like '128' or 'sugar 150'"

### Story 1.5a: Glucose Logging via Web Chat (Text Input)

As a patient,
I want to log my glucose reading by typing the number in the web chat interface,
so that I can use the web interface if I prefer it over WhatsApp.

**Acceptance Criteria:**
1. Same natural language parsing logic as WhatsApp (Story 1.5) - shared parsing service
2. Glucose reading validation: accepts values 40-600 mg/dL, rejects invalid ranges with helpful error message
3. Timestamp handling: uses message timestamp if provided, otherwise current device time
4. Glucose reading stored as event in local SQLite database (offline-first)
5. Glucose reading synced to cloud PostgreSQL when connectivity available
6. Confirmation message sent to patient via web chat: "Glucose logged: 128 mg/dL at 8:30 AM. Looking good!"
7. Support for pre-meal, post-meal, fasting, and random glucose types (detected from context or explicitly specified)
8. Error handling for unparseable input with helpful guidance: "I couldn't understand that. Please send your glucose reading as a number, like '128' or 'sugar 150'"
9. Consistent experience: web chat glucose logging provides same functionality and responses as WhatsApp

### Story 1.6: Glucose Logging via WhatsApp (Voice Input)

As a patient with low literacy or tech comfort,
I want to log my glucose reading by speaking into WhatsApp,
so that I don't need to type numbers.

**Acceptance Criteria:**
1. Voice message reception via WhatsApp webhook
2. Voice-to-text conversion using speech recognition API (Google Speech-to-Text or similar) supporting Hindi and English
3. Parsed text processed through same glucose parsing logic as text input (Story 1.5)
4. Glucose reading stored and synced same as text input
5. Confirmation message sent: "Glucose logged: 128 mg/dL. Thank you!"
6. Error handling for unclear audio with retry suggestion: "I couldn't understand your voice message. Please try again or type the number."
7. Support for Hindi voice input: "Mera sugar 150 hai" → parsed to 150 mg/dL
8. Voice message transcription stored for debugging and improvement

### Story 1.7: Glucose Logging via WhatsApp (Photo OCR)

As a patient,
I want to log my glucose reading by taking a photo of my glucometer screen,
so that I don't need to manually type the number.

**Acceptance Criteria:**
1. Image reception via WhatsApp webhook (glucometer screen photos)
2. OCR processing using cloud OCR service (Google Vision API or similar) to extract glucose value from image
3. Image preprocessing: rotation correction, contrast enhancement, noise reduction
4. Glucose value extraction: recognizes common glucometer displays (various brands, formats)
5. Extracted value validated (40-600 mg/dL range) and stored same as text input
6. Confirmation message sent: "Glucose logged: 128 mg/dL from your photo. Great job tracking!"
7. Error handling for unreadable images: "I couldn't read the number from your photo. Please make sure the screen is clear and well-lit, or type the number."
8. Image stored securely for future reference and OCR model improvement

### Story 1.8: Meal Logging via WhatsApp (Photo + Text)

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

### Story 1.8a: Meal Logging via Web Chat (Photo + Text)

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

### Story 1.9: Data Sync Engine (Offline to Online)

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

## Epic 2: Medication Management & Reminders

### Epic Goal

Implement comprehensive medication management system that enables doctors to set medication schedules during patient onboarding, sends smart reminders to patients at doctor-specified times, tracks medication adherence, and escalates missed doses through a multi-tier system (reminder → nudge → caregiver notification → doctor alert). This epic addresses the critical pain point of 50% medication adherence, targeting 85%+ adherence through timely reminders and caregiver involvement. The system works offline-first, storing medication data locally and syncing when connectivity is available.

### Story 2.1: Medication Data Model & Storage

As a developer,
I want a medication data model that stores medication schedules, dosages, and adherence tracking,
so that the medication management system can track and remind patients effectively.

**Acceptance Criteria:**
1. Database schema designed for medications table: medication_id, patient_id, medication_name, dosage, frequency, times_per_day, specific_times (array), start_date, end_date, doctor_id
2. Medication schedule table: links medications to specific times (e.g., "Metformin 500mg at 8:00 AM and 8:00 PM")
3. Medication log table: tracks when medications were taken (medication_id, taken_at, logged_via, confirmed)
4. Medication reminder queue table: stores pending reminders with scheduled_time, status, escalation_level
5. Schema supports multiple medications per patient with different schedules
6. Schema supports one-time medications and recurring medications
7. Data stored in both PostgreSQL (cloud) and SQLite (local) with sync capability
8. Database indexes optimized for querying: patient medications, upcoming reminders, adherence calculations

### Story 2.2: Doctor Sets Medication Schedule During Patient Onboarding

As a doctor,
I want to set my patient's medication schedule when they onboard,
so that ProCare can send reminders at the correct times.

**Acceptance Criteria:**
1. Doctor web interface (or WhatsApp) to add medications during patient onboarding: medication name, dosage, frequency
2. Time selection: doctor can specify exact times (e.g., "8:00 AM and 8:00 PM") or frequency-based (e.g., "twice daily")
3. Medication schedule stored in database linked to patient and doctor
4. Confirmation sent to patient via WhatsApp: "Dr. Verma has set your medication schedule: Metformin 500mg at 8:00 AM and 8:00 PM. I'll remind you!"
5. Support for multiple medications with different schedules
6. Medication schedule editable by doctor (add, modify, remove medications)
7. Medication schedule changes synced to patient's local database
8. Error handling for invalid schedules, duplicate medications, and missing required fields

### Story 2.3: Basic Medication Reminders via WhatsApp/Web Chat

As a patient,
I want to receive medication reminders at the times my doctor specified,
so that I don't forget to take my medications.

**Acceptance Criteria:**
1. Reminder scheduler: checks medication schedules and sends reminders at specified times
2. Message sent to patient via channel where patient last interacted (WhatsApp or web chat): "Time for your medication! Metformin 500mg. Reply 'taken' when you've taken it."
3. Channel routing: Engagement Engine routes reminders to the channel where patient last interacted. If patient hasn't interacted recently, reminder sent to both channels to ensure delivery.
4. Reminder respects rate limiting: max 5 notifications/day, 2-hour minimum spacing (if multiple medications, applies across both channels)
5. Reminder works offline: scheduled reminders stored locally, sent when connectivity available
6. Patient can confirm medication taken: replies "taken", "yes", "done", "le liya" (Hindi) via their active channel
7. Medication log entry created when patient confirms: medication_id, taken_at timestamp, logged_via "reminder_response"
8. Confirmation message sent via same channel: "Great! I've logged your medication. Keep it up!"
9. Reminder not sent if patient already logged medication within 30 minutes of scheduled time

### Story 2.4: Medication Logging by Patient (Manual Entry)

As a patient,
I want to log when I take my medication,
so that I can track my adherence even if I miss a reminder.

**Acceptance Criteria:**
1. Patient can log medication via WhatsApp: "took metformin", "medication taken", "medicine le liya" (Hindi)
2. Natural language parsing: recognizes medication names, dosages, and "taken" confirmation
3. Medication log entry created: medication_id, taken_at timestamp, logged_via "manual_entry"
4. Support for logging multiple medications: "took metformin and glipizide"
5. Data stored locally (SQLite) and synced to cloud when online
6. Confirmation message: "Medication logged: Metformin 500mg at 2:30 PM. Thank you!"
7. Error handling for unrecognized medication names: "I couldn't find that medication. Please check your schedule or contact your doctor."
8. Support for logging past medications: "took metformin yesterday at 8am"

### Story 2.5: Missed Medication Detection & First Escalation (Nudge)

As a patient,
I want a gentle nudge if I miss a medication,
so that I can still take it if I forgot.

**Acceptance Criteria:**
1. Missed medication detection: if medication not logged within 1 hour of scheduled time, mark as missed
2. First escalation (nudge): send WhatsApp message 1 hour after missed time: "Friendly reminder: You haven't logged Metformin 500mg yet. Did you take it? Reply 'taken' if yes."
3. Patient can still log medication after nudge (within 2 hours of scheduled time)
4. If patient logs medication after nudge, mark as "taken_late" in medication log
5. If medication still not logged 2 hours after scheduled time, escalate to next level (Story 2.6)
6. Nudge respects rate limiting: doesn't send if already sent 5 notifications today
7. Nudge works offline: missed medication detected locally, nudge queued for sending
8. Adherence calculation updated: missed medications reduce adherence percentage

### Story 2.6: Medication Escalation to Caregiver

As a caregiver,
I want to be notified if my parent misses a medication,
so that I can help ensure they take it.

**Acceptance Criteria:**
1. Caregiver escalation trigger: if medication not logged 2 hours after scheduled time AND patient has caregiver linked
2. WhatsApp message sent to caregiver: "Alert: [Patient Name] hasn't logged Metformin 500mg (scheduled 8:00 AM). Please check if they took it."
3. Caregiver can respond: "taken" or "will remind" or "not taken"
4. If caregiver confirms "taken", medication log entry created with logged_via "caregiver_confirmation"
5. If caregiver responds "not taken", escalate to doctor (Story 2.7)
6. Caregiver escalation only for patients with caregiver-primary or shared management mode
7. Caregiver escalation respects patient privacy: only sends for critical medications or after multiple misses
8. Caregiver notification includes patient context: recent glucose readings, medication history

### Story 2.7: Medication Escalation to Doctor

As a doctor,
I want to be alerted if my patient consistently misses medications,
so that I can intervene before it affects their health.

**Acceptance Criteria:**
1. Doctor escalation trigger: medication not logged 4 hours after scheduled time OR caregiver confirms "not taken" OR patient misses same medication 3+ times in a week
2. Alert created in doctor dashboard: patient name, medication, scheduled time, missed duration, adherence history
3. Alert includes context: patient's recent glucose readings, medication history, caregiver status
4. Alert categorized as "Attention" tier in doctor dashboard (not "Urgent" unless combined with other issues)
5. Doctor can view patient detail and send message via dashboard or WhatsApp
6. Doctor escalation notification: "Patient [Name] missed Metformin 500mg (scheduled 8:00 AM). Adherence this week: 60%."
7. Escalation respects doctor preferences: some doctors may want immediate alerts, others prefer daily summary
8. Medication adherence metrics updated: weekly adherence percentage calculated and displayed to doctor

### Story 2.8: Medication Adherence Tracking & Reporting

As a doctor,
I want to see my patient's medication adherence percentage,
so that I can identify patients who need help with medication compliance.

**Acceptance Criteria:**
1. Adherence calculation: percentage of medications taken on time (within 1 hour of scheduled time) vs total scheduled medications
2. Daily adherence: calculated each day for each patient
3. Weekly adherence: average of daily adherence percentages for the week
4. Monthly adherence: average of weekly adherence percentages for the month
5. Adherence displayed in doctor dashboard: patient list shows adherence percentage, patient detail shows adherence trend chart
6. Adherence displayed in caregiver dashboard: shows parent's adherence percentage and trend
7. Adherence data stored in database: daily, weekly, monthly aggregates for reporting
8. Adherence metrics synced between local and cloud databases
9. Adherence alerts: doctor dashboard highlights patients with <80% adherence
10. Patient-facing adherence: weekly progress report includes adherence percentage and encouragement message

## Epic 3: Agent Orchestration & Emergency Detection

### Epic Goal

Build the foundational 7-agent orchestration system that enables ProCare to provide intelligent, coordinated responses to patient actions. Implement the Engagement Engine as the central orchestrator that routes all patient events to appropriate agents, manages patient state, and synthesizes multi-agent responses into single messages. Implement the Clinical Monitor agent to detect emergency glucose levels (hypoglycemia <70 mg/dL, hyperglycemia >250 mg/dL) and provide immediate safety instructions with automatic doctor escalation. Establish agent communication via message bus (RabbitMQ or Redis) and implement priority hierarchy where emergencies override all other notifications. This epic establishes the technical architecture that enables all subsequent intelligent features while ensuring patient safety through emergency detection.

### Story 3.1: Message Bus Infrastructure & Agent Communication

As a developer,
I want a message bus system for agent-to-agent communication,
so that agents can communicate asynchronously and independently.

**Acceptance Criteria:**
1. Message bus infrastructure set up (RabbitMQ or Redis) with queues for each agent
2. Event schema defined: event_type, patient_id, timestamp, data (JSON), source_agent, priority
3. Agent registration system: each agent registers itself and subscribes to relevant event types
4. Event publishing: agents can publish events to message bus
5. Event consumption: agents can consume events from their queues
6. Error handling: failed message processing retried with exponential backoff, dead letter queue for failed messages
7. Message persistence: events stored for replay and debugging
8. Monitoring: message bus health checks, queue depth monitoring, agent status tracking

### Story 3.2: Engagement Engine - Core Orchestrator

As a patient,
I want ProCare to intelligently route my actions to the right agents,
so that I get coordinated, helpful responses.

**Acceptance Criteria:**
1. Engagement Engine service created as central orchestrator
2. Event routing logic: patient action → Engagement Engine → determines which agents need to process event
3. Agent selection: Engagement Engine routes events to relevant agents based on event type (glucose_log → Clinical Monitor + Health Insights, meal_log → Lifestyle Coach + Health Insights)
4. Patient state management: Engagement Engine tracks patient state (onboarding, active, inactive, emergency)
5. Channel routing: Engagement Engine routes responses to the channel where patient last interacted (if patient logged glucose via WhatsApp, response goes to WhatsApp; if via web chat, response goes to web chat)
6. Multi-channel support: Engagement Engine supports sending messages via both WhatsApp and web chat - patient can use either channel at any time, both are always available
7. Channel detection: Engagement Engine detects which channel patient is using based on incoming message source (WhatsApp webhook vs web chat API)
8. Event aggregation: Engagement Engine collects responses from multiple agents
9. Response synthesis: Engagement Engine combines multiple agent responses into single patient message
10. Rate limiting enforcement: Engagement Engine enforces max 5 notifications/day, 2-hour minimum spacing (applies across both channels - total notifications, not per channel)
11. Notification queuing: Engagement Engine queues notifications when rate limit reached, sends when allowed
12. Priority hierarchy: Emergency events override rate limiting and other notifications
13. Logging: Engagement Engine logs all routing decisions, channel detection, and agent responses for debugging

### Story 3.3: Clinical Monitor Agent - Emergency Detection Logic

As a patient,
I want ProCare to detect dangerous glucose levels immediately,
so that I get help when I need it most.

**Acceptance Criteria:**
1. Clinical Monitor agent service created as independent microservice
2. Emergency detection rules: hypoglycemia <70 mg/dL, hyperglycemia >250 mg/dL
3. Real-time monitoring: Clinical Monitor processes glucose readings as they're logged
4. Emergency classification: detects hypoglycemia (low), hyperglycemia (high), or normal
5. Emergency priority: emergencies marked as highest priority, override all other notifications
6. Emergency detection works offline: Clinical Monitor processes local glucose readings even when offline
7. Emergency detection accuracy: 98%+ true positive rate (no missed emergencies)
8. False alarm prevention: validates glucose reading (not duplicate, not test strip error) before triggering emergency
9. Emergency context: Clinical Monitor includes patient's recent glucose history, medication status, meal timing in emergency alert
10. Emergency logging: all emergency detections logged with timestamp, glucose value, patient response

### Story 3.4: Clinical Monitor - Emergency Response & Instructions

As a patient experiencing hypoglycemia or hyperglycemia,
I want immediate, clear instructions on what to do,
so that I can take action to stay safe.

**Acceptance Criteria:**
1. Hypoglycemia response (<70 mg/dL): immediate message via channel where patient last interacted (WhatsApp or web chat) with instructions: "URGENT: Your glucose is [value] mg/dL (low). Eat 15g fast-acting sugar (glucose tablets, fruit juice, candy). Check again in 15 minutes. If still low, contact your doctor immediately."
2. Hyperglycemia response (>250 mg/dL): immediate message via channel where patient last interacted: "URGENT: Your glucose is [value] mg/dL (high). Drink water, avoid food, check for ketones if you have strips. Contact your doctor if this persists or you feel unwell."
3. Channel routing: Engagement Engine routes emergency messages to the channel where patient last interacted (if patient last used WhatsApp, emergency goes to WhatsApp; if web chat, goes to web chat). If patient hasn't interacted recently, message sent to both channels.
4. Emergency instructions personalized: includes patient's name, recent medication timing, meal timing
5. Emergency instructions in Hindi and English based on patient preference
6. Follow-up check: Clinical Monitor sends follow-up message 15 minutes after hypoglycemia via same channel: "Please check your glucose again. How are you feeling?"
7. Emergency escalation: if patient doesn't respond to emergency or glucose worsens, escalate to doctor immediately (Story 3.5)
8. Emergency instructions work offline: stored locally, sent immediately when connectivity available
9. Emergency acknowledgment: patient can respond "ok", "feeling better", "need help" via their active channel to update emergency status

### Story 3.5: Clinical Monitor - Doctor Escalation for Emergencies

As a doctor,
I want to be immediately alerted if my patient has a dangerous glucose level,
so that I can provide medical assistance.

**Acceptance Criteria:**
1. Doctor escalation trigger: emergency detected (hypoglycemia <70 or hyperglycemia >250) AND patient doesn't respond within 15 minutes OR glucose worsens
2. Urgent alert created in doctor dashboard: patient name, glucose value, emergency type, timestamp, patient response status
3. Alert categorized as "Urgent" tier (highest priority) in doctor dashboard
4. Alert includes context: patient's recent glucose history, medication timing, meal timing, caregiver status
5. Doctor notification: WhatsApp or SMS sent to doctor: "URGENT: Patient [Name] has [hypoglycemia/hyperglycemia] - glucose [value] mg/dL at [time]. Please check dashboard."
6. Doctor can view patient detail and send message via dashboard or WhatsApp
7. Doctor escalation respects doctor preferences: some doctors want immediate alerts, others prefer during business hours only
8. Emergency status tracking: Clinical Monitor tracks emergency resolution (patient responded, doctor contacted, resolved)
9. Emergency follow-up: Clinical Monitor sends follow-up to doctor if emergency not resolved within 1 hour
10. Emergency reporting: all emergencies logged for doctor review and patient safety analysis

### Story 3.6: Agent Response Synthesis & Notification Bundling

As a patient,
I want a single, coordinated message from ProCare,
so that I'm not overwhelmed with multiple notifications.

**Acceptance Criteria:**
1. Response collection: Engagement Engine collects responses from all agents processing an event
2. Response synthesis: Engagement Engine combines multiple agent responses into single, coherent message
3. Message formatting: synthesized message includes all relevant information without redundancy
4. Example synthesis: "Glucose logged: 128 mg/dL. Looking good! [Clinical Monitor] Your glucose is in target range. [Health Insights] This is similar to your readings after roti meals. [Lifestyle Coach] Keep up the good work!"
5. Notification bundling: if multiple events occur within 2-hour window, bundle into single notification
6. Priority ordering: emergency responses always sent first, then other notifications
7. Message length: synthesized messages kept concise (max 500 characters) for WhatsApp readability
8. Language consistency: synthesized messages maintain consistent tone and language (Hindi/English)
9. Error handling: if agent response fails, Engagement Engine still sends available responses
10. Response logging: all synthesized messages logged for quality improvement

### Story 3.7: Priority Hierarchy & Emergency Override

As a patient,
I want emergency notifications to always get through,
even if I've already received my daily limit of messages.

**Acceptance Criteria:**
1. Priority levels defined: Emergency (highest), Urgent, Attention, Normal (lowest)
2. Emergency override: emergency notifications bypass rate limiting (max 5/day) and send immediately
3. Priority enforcement: Engagement Engine checks priority before applying rate limiting
4. Emergency queue: emergency notifications queued separately from normal notifications
5. Emergency delivery: emergency notifications sent immediately, normal notifications queued if rate limit reached
6. Priority in message bus: emergency events marked with highest priority, processed first
7. Agent priority: Clinical Monitor agent has highest priority, can interrupt other agent processing
8. Emergency notification: patient receives emergency message even if already received 5 notifications today
9. Normal notification deferral: if emergency sent, normal notifications deferred until next allowed window
10. Priority logging: all priority decisions logged for monitoring and debugging

### Story 3.8: Agent Health Monitoring & Error Recovery

As a developer,
I want to monitor agent health and recover from failures,
so that ProCare remains reliable even if individual agents fail.

**Acceptance Criteria:**
1. Agent health checks: each agent exposes health check endpoint (returns status, last processed event, queue depth)
2. Health monitoring: Engagement Engine monitors agent health, detects failures
3. Agent failure detection: if agent doesn't respond to health check within timeout, mark as failed
4. Graceful degradation: if agent fails, Engagement Engine continues with other agents, logs failure
5. Error recovery: failed agents automatically restarted, queued events reprocessed
6. Circuit breaker: if agent fails repeatedly, circuit breaker prevents further requests until recovery
7. Fallback responses: if agent fails, Engagement Engine sends generic response or skips that agent's contribution
8. Error alerting: agent failures logged and alerted to development team
9. Agent status dashboard: monitoring dashboard shows agent health, queue depths, error rates
10. Recovery testing: system tested for agent failure scenarios, graceful degradation verified

## Epic 4: Pattern Detection & Health Insights

### Epic Goal

Implement the Health Insights agent that analyzes patient data to detect personalized patterns in food-glucose correlations, activity impacts, and medication consistency. After collecting 4 weeks of patient data, the agent generates personalized insights like "Rice usually spikes your glucose to 180. Roti keeps you at 140" that enable proactive interventions (Epic 7). The agent validates predictions against actual outcomes to improve accuracy over time. This epic delivers the core intelligence that differentiates ProCare from generic health apps, providing patients with actionable, personalized insights rather than generic advice. The system works offline-first, storing pattern data locally and syncing when connectivity is available.

### Story 4.1: Health Insights Agent Infrastructure & Data Collection

As a developer,
I want the Health Insights agent to collect and store patient data for pattern analysis,
so that patterns can be detected after sufficient data is collected.

**Acceptance Criteria:**
1. Health Insights agent service created as independent microservice
2. Data collection: agent subscribes to glucose_log, meal_log, medication_log, activity_log events from message bus
3. Data storage: patient data stored in time-series database (InfluxDB or TimescaleDB) for efficient pattern queries
4. Data schema: glucose_readings, meals, medications, activities with timestamps, patient_id, metadata
5. Minimum data requirement: agent tracks data collection progress, requires 4 weeks minimum before pattern detection
6. Data quality validation: agent validates data completeness (glucose readings, meals, medications) before pattern analysis
7. Data retention: stores 90 days of detailed data locally, 1 year of summary data, >1 year stats only
8. Data sync: patient data synced between local and cloud databases for pattern analysis
9. Agent registration: Health Insights agent registers with Engagement Engine and message bus
10. Event processing: agent processes events asynchronously, doesn't block patient interactions

### Story 4.2: Food-Glucose Correlation Analysis

As a patient,
I want to know which foods affect my glucose levels,
so that I can make better food choices.

**Acceptance Criteria:**
1. Correlation algorithm: Health Insights analyzes glucose readings within 2 hours after meals
2. Food identification: matches logged meals (from Story 1.8) with subsequent glucose readings
3. Pattern detection: identifies foods that consistently cause glucose spikes (>180 mg/dL) or maintain stable levels (<140 mg/dL)
4. Confidence scoring: calculates confidence level for each food-glucose correlation (requires 3+ occurrences for confidence)
5. Pattern examples: "Rice usually spikes your glucose to 180 mg/dL" or "Roti keeps you at 140 mg/dL"
6. Pattern storage: detected patterns stored in database with confidence score, frequency, average glucose impact
7. Pattern detection works offline: analyzes local data, stores patterns locally, syncs when online
8. Pattern validation: validates patterns against new data, updates confidence scores over time
9. Pattern reporting: agent generates pattern summary for patient: "I've learned that rice spikes your glucose, but roti keeps you stable"
10. Error handling: handles missing data, invalid correlations, and edge cases gracefully

### Story 4.3: Activity Impact Analysis

As a patient,
I want to know how activity affects my glucose levels,
so that I can plan my exercise better.

**Acceptance Criteria:**
1. Activity logging: patient can log activities via WhatsApp: "walked 30 minutes", "did yoga", "went for run"
2. Activity data collection: Health Insights agent collects activity logs with timestamps
3. Activity-glucose correlation: analyzes glucose readings before and after activities
4. Pattern detection: identifies activities that lower glucose (walking, exercise) or have no impact
5. Impact quantification: calculates average glucose reduction from activities (e.g., "30-minute walk reduces glucose by 20 mg/dL")
6. Timing analysis: identifies optimal timing for activities (before meals, after meals, morning vs evening)
7. Pattern confidence: requires 3+ occurrences for confidence, updates confidence with more data
8. Pattern storage: activity patterns stored in database with impact, timing, confidence score
9. Pattern reporting: agent generates activity insights: "I've noticed that a 30-minute walk after dinner helps keep your glucose stable"
10. Activity recommendations: agent suggests activities based on detected patterns and glucose levels

### Story 4.4: Medication Consistency Pattern Detection

As a patient,
I want to see how consistent medication timing affects my glucose,
so that I can understand the importance of taking medications on time.

**Acceptance Criteria:**
1. Medication timing analysis: Health Insights analyzes medication logs (from Epic 2) and glucose readings
2. Consistency detection: identifies patterns where consistent medication timing correlates with stable glucose
3. Impact analysis: compares glucose levels when medications taken on time vs late/missed
4. Pattern examples: "When you take Metformin on time, your fasting glucose averages 110 mg/dL. When late, it's 140 mg/dL"
5. Pattern confidence: requires 2+ weeks of data with consistent patterns for confidence
6. Pattern storage: medication consistency patterns stored with impact, timing correlation, confidence score
7. Pattern reporting: agent generates medication insights: "Taking your medications on time helps keep your glucose stable"
8. Adherence correlation: correlates medication adherence (from Epic 2) with glucose control
9. Pattern validation: validates patterns against new medication and glucose data
10. Insights integration: medication patterns integrated into weekly progress reports

### Story 4.5: Pattern Detection After 4 Weeks of Data

As a patient,
I want personalized insights after ProCare learns my patterns,
so that I get actionable advice based on my specific data.

**Acceptance Criteria:**
1. Data sufficiency check: Health Insights checks if patient has 4 weeks of data (glucose readings, meals, medications)
2. Pattern analysis trigger: after 4 weeks, agent automatically runs pattern detection analysis
3. Multi-pattern detection: agent detects food-glucose, activity-impact, and medication-consistency patterns simultaneously
4. Pattern confidence threshold: only reports patterns with >70% confidence (requires 3+ occurrences)
5. Pattern summary generation: agent generates personalized pattern summary: "I've learned your patterns: Rice spikes glucose to 180, roti keeps you at 140. 30-minute walk reduces glucose by 20. Taking medications on time helps stability."
6. Pattern notification: patient receives WhatsApp message: "Great news! I've learned your patterns after 4 weeks. Here's what I found: [pattern summary]"
7. Pattern storage: detected patterns stored in patient profile for use in proactive interventions (Epic 7)
8. Pattern updates: agent re-analyzes patterns weekly, updates confidence scores, detects new patterns
9. Pattern accuracy: 90%+ of patients should have detectable patterns within 4 weeks
10. Pattern reporting: patterns displayed in patient weekly progress report and doctor dashboard

### Story 4.6: Personalized Insight Generation

As a patient,
I want insights that are specific to me,
so that I get advice that actually works for my body.

**Acceptance Criteria:**
1. Insight generation: Health Insights generates personalized insights based on detected patterns
2. Insight types: food recommendations, activity suggestions, medication timing advice, glucose trend explanations
3. Insight personalization: insights use patient's name, specific foods, actual glucose values from their data
4. Insight examples: "Your glucose was 180 after rice yesterday. Roti usually keeps you at 140. Try roti instead?" or "Your fasting glucose is high (140). Taking Metformin on time helps - you took it late 3 times this week"
5. Insight timing: insights generated after glucose logging, meal logging, or weekly analysis
6. Insight delivery: insights included in patient WhatsApp messages (synthesized by Engagement Engine)
7. Insight storage: insights stored in database with timestamp, pattern reference, patient response
8. Insight validation: agent tracks patient responses to insights, validates if insights led to behavior change
9. Insight improvement: agent learns which insights are most effective, prioritizes high-impact insights
10. Insight language: insights delivered in patient's preferred language (Hindi/English), warm and supportive tone

### Story 4.7: Prediction Validation & Learning Loop

As a patient,
I want ProCare to learn and improve its predictions over time,
so that the insights become more accurate and helpful.

**Acceptance Criteria:**
1. Prediction generation: Health Insights generates predictions based on patterns (e.g., "If you eat rice, your glucose will likely spike to 180")
2. Outcome tracking: agent tracks actual glucose readings after predictions
3. Validation: compares predicted glucose values with actual values, calculates accuracy
4. Accuracy threshold: 95%+ glucose predictions within 15 mg/dL of actual values
5. Learning loop: agent updates pattern confidence scores based on prediction accuracy
6. Pattern refinement: if predictions inaccurate, agent re-analyzes patterns, adjusts confidence scores
7. False pattern detection: agent identifies and removes false patterns (correlations that don't hold over time)
8. Pattern evolution: agent detects when patterns change (patient's body responds differently), updates patterns
9. Validation reporting: agent reports prediction accuracy to development team for model improvement
10. Continuous learning: agent continuously learns from new data, improves predictions over time

### Story 4.8: Pattern Display in Dashboards

As a doctor,
I want to see my patient's detected patterns,
so that I can provide better guidance during visits.

**Acceptance Criteria:**
1. Pattern display in doctor dashboard: shows patient's detected patterns (food-glucose, activity-impact, medication-consistency)
2. Pattern visualization: patterns displayed as simple text summaries or basic charts (food → glucose impact)
3. Pattern confidence: doctor dashboard shows confidence level for each pattern (high, medium, low)
4. Pattern timeline: shows when patterns were detected, last updated, validation status
5. Pattern insights: doctor dashboard includes agent-generated insights for each patient
6. Pattern in patient detail view: doctor can see patient's patterns when viewing patient detail
7. Pattern in weekly summary: patterns included in doctor's weekly patient summary
8. Pattern export: doctor can export patient patterns for review or sharing
9. Pattern alerts: doctor dashboard highlights patients with new patterns or pattern changes
10. Pattern context: patterns displayed with patient's glucose trends, medication adherence, meal logging frequency

## Epic 5: Doctor Dashboard & Patient Management

### Epic Goal

Build a web-based doctor dashboard that enables doctors to efficiently manage their patients, review patient data, and provide guidance without adding significant time burden. Implement tiered patient view (Urgent/Attention/Stable) for quick triage, patient onboarding interface for setting expectations (target A1C, medication schedule, lifestyle goals), alert management system with context and suggested actions, and comprehensive patient detail views showing glucose trends, medication adherence, patterns, and alerts. The dashboard saves doctors 10-15 minutes daily through batch review (Monday morning weekly summaries) and enables the doctor-initiated engagement model where doctors set expectations and ProCare creates personalized programs. The dashboard is desktop-first, accessible via web browsers without requiring mobile app download.

### Story 5.1: Doctor Authentication & Dashboard Foundation

As a doctor,
I want to securely log into the ProCare dashboard,
so that I can access my patient data safely.

**Acceptance Criteria:**
1. Doctor registration: doctors can register via web form with credentials (name, email, phone, medical license number, specialty)
2. Doctor authentication: OAuth 2.0 or JWT-based authentication for secure login
3. Login page: simple, clean login interface accessible via web browser
4. Session management: secure session handling with automatic timeout after inactivity
5. Doctor profile: doctors can view and edit their profile (name, contact info, preferences)
6. Dashboard landing page: after login, doctors see main dashboard with patient overview
7. Responsive design: dashboard works on desktop browsers (Chrome 90+, Safari 14+, Firefox 88+)
8. Error handling: clear error messages for login failures, session expiration, access denied
9. Security: password requirements, account lockout after failed attempts, secure password reset
10. Doctor data storage: doctor profiles stored in PostgreSQL database with encrypted sensitive data

### Story 5.2: Tiered Patient View (Urgent/Attention/Stable)

As a doctor,
I want to see my patients organized by priority,
so that I can quickly identify who needs immediate attention.

**Acceptance Criteria:**
1. Patient tiering logic: patients automatically categorized into Urgent, Attention, or Stable tiers
2. Urgent tier: patients with emergencies (hypoglycemia/hyperglycemia from Epic 3), critical alerts, or urgent medication issues
3. Attention tier: patients with missed medications, high glucose trends, new patterns detected, or questions requiring response
4. Stable tier: patients with good glucose control, consistent medication adherence, no alerts
5. Tiered view interface: dashboard shows three tabs or sections (Urgent, Attention, Stable)
6. Patient list in each tier: shows patient name, last glucose reading, adherence percentage, alert count, last activity
7. Patient count badges: shows number of patients in each tier (e.g., "Urgent (3)", "Attention (12)", "Stable (45)")
8. Auto-refresh: patient tiers update automatically as new data arrives (every 5 minutes or on-demand)
9. Tier filtering: doctors can filter patients within each tier by name, date, or alert type
10. Tier sorting: patients within each tier sorted by priority (most urgent first) or alphabetically

### Story 5.3: Patient Detail View

As a doctor,
I want to see comprehensive patient information in one place,
so that I can quickly understand their status and provide guidance.

**Acceptance Criteria:**
1. Patient detail page: accessible by clicking patient name from tiered view
2. Patient overview: shows patient name, age, diabetes type, diagnosis date, doctor connection date
3. Glucose trends: displays glucose readings over time (last 7 days, 30 days) as simple line chart or table
4. Medication adherence: shows adherence percentage (daily, weekly, monthly) with trend indicator
5. Recent alerts: displays recent alerts (emergencies, missed medications, pattern changes) with timestamps
6. Detected patterns: shows patient's food-glucose, activity-impact, medication-consistency patterns (from Epic 4)
7. Recent activity: shows last glucose log, meal log, medication log with timestamps
8. Caregiver status: shows if patient has caregiver linked, caregiver contact info (if permitted)
9. Patient summary: quick summary card showing key metrics (average glucose, adherence, alert count)
10. Navigation: easy navigation back to tiered view, to other patients, or to patient messaging

### Story 5.4: Patient Onboarding Interface - Setting Expectations

As a doctor,
I want to set my patient's treatment goals and medication schedule during onboarding,
so that ProCare can create a personalized program for them.

**Acceptance Criteria:**
1. Patient onboarding interface: doctor can access onboarding form for new patients
2. Target A1C setting: doctor can set target HbA1c (e.g., <7.0%) for patient
3. Medication schedule setup: doctor can add medications with dosages and times (links to Epic 2 Story 2.2)
4. Lifestyle goals: doctor can set lifestyle goals (e.g., "walk 30 minutes daily", "reduce rice intake")
5. Glucose target ranges: doctor can set target glucose ranges (fasting, post-meal, random)
6. Onboarding completion: doctor submits onboarding form, patient receives WhatsApp confirmation
7. Personalized program creation: ProCare creates personalized program based on doctor's expectations
8. Onboarding data storage: patient goals and expectations stored in database, linked to patient and doctor
9. Onboarding editing: doctor can update patient goals and expectations after onboarding
10. Onboarding confirmation: patient receives WhatsApp: "Dr. Verma has set your goals: Target A1C <7.0%, Metformin 500mg twice daily, walk 30 minutes daily. I'll help you achieve these!"

### Story 5.5: Alert Management System with Context

As a doctor,
I want to see patient alerts with context and suggested actions,
so that I can quickly understand the issue and take appropriate action.

**Acceptance Criteria:**
1. Alert display: alerts shown in patient detail view and tiered view with alert type, timestamp, priority
2. Alert context: each alert includes relevant context (patient's recent glucose history, medication timing, meal timing, caregiver status)
3. Suggested actions: alerts include suggested actions (e.g., "Review medication schedule", "Check glucose pattern", "Contact patient")
4. Alert types: emergency alerts (from Epic 3), medication alerts (from Epic 2), pattern alerts (from Epic 4), question alerts
5. Alert prioritization: alerts sorted by priority (Urgent > Attention > Normal) and timestamp
6. Alert filtering: doctors can filter alerts by type, date, priority, or patient
7. Alert actions: doctors can acknowledge alerts, mark as resolved, or escalate
8. Alert history: doctors can view alert history for each patient (resolved and active alerts)
9. Alert notifications: doctors receive email or WhatsApp notification for urgent alerts (based on preferences)
10. Alert reporting: alert summary included in weekly patient summary

### Story 5.6: Weekly Patient Summary Generation

As a doctor,
I want to see weekly summaries of my patients' progress,
so that I can efficiently review multiple patients in one session.

**Acceptance Criteria:**
1. Weekly summary generation: system generates weekly summary for each patient every Monday morning
2. Summary content: includes glucose trends, medication adherence, detected patterns, alerts, key insights
3. Summary display: doctors can view weekly summaries in batch (all patients) or individual patient view
4. Summary metrics: shows average glucose, time in range, adherence percentage, pattern changes
5. Summary insights: includes Health Insights agent-generated insights (from Epic 4)
6. Summary alerts: highlights patients needing attention based on weekly data
7. Summary export: doctors can export weekly summaries as PDF or CSV for records
8. Summary timing: summaries generated Monday morning, available for doctor's weekly review (10-minute session)
9. Summary notification: doctors receive email notification when weekly summaries are ready
10. Summary history: doctors can view past weekly summaries for trend analysis

### Story 5.7: Doctor-Patient Messaging via Dashboard

As a doctor,
I want to send messages to my patients via the dashboard,
so that I can provide guidance without switching to WhatsApp.

**Acceptance Criteria:**
1. Messaging interface: doctor dashboard includes messaging interface for sending messages to patients
2. Message composition: doctors can type messages, select from templates, or use suggested responses
3. Message delivery: messages sent to patient via WhatsApp (integrated with WhatsApp API)
4. Message history: doctors can view message history with patients (sent and received)
5. Message templates: doctors can create and use message templates for common scenarios
6. Suggested responses: dashboard suggests responses based on patient alerts or questions
7. Message context: messages include patient context (recent glucose, adherence, patterns) for doctor reference
8. Message scheduling: doctors can schedule messages to be sent later
9. Message status: doctors can see if message was delivered and read by patient
10. Message integration: patient messages appear in doctor dashboard, doctor responses appear in patient WhatsApp

### Story 5.8: Doctor Dashboard Performance & Usability

As a doctor,
I want the dashboard to load quickly and be easy to use,
so that I can review patients efficiently without frustration.

**Acceptance Criteria:**
1. Page load performance: dashboard pages load within 500ms (per NFR2)
2. Data loading: patient data loads incrementally (tiered view loads first, detail view loads on demand)
3. Responsive design: dashboard works on desktop browsers, tablet-friendly layout
4. Intuitive navigation: clear navigation between tiered view, patient detail, alerts, messaging
5. Search functionality: doctors can search patients by name, phone number, or alert type
6. Keyboard shortcuts: common actions accessible via keyboard shortcuts for power users
7. Accessibility: dashboard meets WCAG AA standards (keyboard navigation, screen reader support)
8. Error handling: clear error messages, graceful degradation if data unavailable
9. Help documentation: inline help tooltips and documentation for dashboard features
10. User testing: dashboard tested with 5-10 doctors, usability feedback incorporated

## Epic 6: Caregiver Features & Alerts

### Epic Goal

Implement comprehensive caregiver features that enable family members to monitor and support elderly diabetic patients, reducing caregiver anxiety while improving patient outcomes. Build separate caregiver dashboard (web + WhatsApp) with real-time critical alerts for hypoglycemia and missed medications, daily digest summaries showing patient status, and caregiver-primary management mode option where caregivers manage the patient's ProCare account. This epic addresses the unique India market need for family involvement in healthcare, delivering 2.3x medication adherence improvement (validated by WellDoc research) and providing peace of mind to working professionals managing elderly parents. The caregiver experience is designed to reduce anxiety, not amplify it, with setup primarily on the caregiver's device rather than requiring elderly parent to use technology.

### Story 6.1: Caregiver Registration & Patient Linking

As a caregiver,
I want to register and link to my parent's ProCare account,
so that I can monitor their diabetes management.

**Acceptance Criteria:**
1. Caregiver registration: caregivers can register via web form or WhatsApp with credentials (name, email, phone, relationship to patient)
2. Patient linking: caregiver can link to patient account via patient phone number and verification code (provided by patient)
3. Patient consent: patient must explicitly consent to caregiver access (during onboarding or later)
4. Relationship verification: system verifies caregiver-patient relationship (phone number match, patient approval)
5. Caregiver profile: caregivers can view and edit their profile (name, contact info, notification preferences)
6. Multiple caregivers: patient can link multiple caregivers (e.g., both children, spouse)
7. Caregiver authentication: OAuth 2.0 or JWT-based authentication for secure login
8. Caregiver data storage: caregiver profiles stored in PostgreSQL database, linked to patient
9. Access control: caregivers can only access linked patient's data, not other patients
10. Caregiver removal: patient or doctor can remove caregiver access at any time

### Story 6.2: Caregiver Web Dashboard - Patient Status Overview

As a caregiver,
I want to see my parent's current status at a glance,
so that I know they're okay without calling them constantly.

**Acceptance Criteria:**
1. Caregiver dashboard landing page: shows patient status overview after login
2. Current status indicator: shows patient's current status (OK, Alert, Emergency) with color coding
3. Last glucose reading: displays last glucose reading with timestamp and status (normal, high, low)
4. Medication adherence: shows today's medication adherence percentage and status
5. Recent activity: shows last glucose log, meal log, medication log with timestamps
6. Alert summary: shows count of active alerts (critical, attention, normal)
7. Status timeline: shows patient activity timeline for today (glucose checks, medications, meals)
8. Quick actions: caregivers can send message to patient or view detailed patient information
9. Dashboard refresh: status updates automatically every 5 minutes or on-demand
10. Mobile-responsive design: dashboard works on mobile devices (caregiver's phone) and desktop

### Story 6.3: Real-Time Critical Alerts via WhatsApp

As a caregiver,
I want immediate alerts for critical situations,
so that I can help my parent quickly if needed.

**Acceptance Criteria:**
1. Critical alert types: hypoglycemia (<70 mg/dL), hyperglycemia (>250 mg/dL), missed critical medications
2. Alert delivery: critical alerts sent to caregiver via WhatsApp immediately (not batched)
3. Alert content: includes patient name, alert type, glucose value (if applicable), timestamp, suggested action
4. Alert example: "URGENT: [Parent Name] has hypoglycemia - glucose 65 mg/dL at 2:30 PM. Please check on them immediately."
5. Alert acknowledgment: caregiver can respond "ok", "checking", "on my way" to acknowledge alert
6. Alert escalation: if caregiver doesn't respond within 15 minutes, alert escalated to doctor
7. Alert history: caregivers can view alert history in dashboard (all critical alerts with timestamps)
8. Alert preferences: caregivers can set alert preferences (all alerts, critical only, daily digest only)
9. Alert frequency: critical alerts sent immediately, not rate-limited (emergency override)
10. Alert integration: alerts integrated with Clinical Monitor agent (from Epic 3) and medication escalation (from Epic 2)

### Story 6.4: Daily Digest Summary via WhatsApp

As a caregiver,
I want a daily summary of my parent's diabetes management,
so that I can monitor their progress without constant checking.

**Acceptance Criteria:**
1. Daily digest generation: system generates daily digest for each patient with caregiver at end of day (8 PM)
2. Digest content: includes glucose summary (average, range, readings count), medication adherence, meals logged, alerts summary
3. Digest delivery: daily digest sent to caregiver via WhatsApp message
4. Digest format: concise, easy-to-read format: "Daily Summary for [Parent Name]: Glucose average 128 mg/dL (3 readings), Medications taken 100%, Meals logged 2, No alerts. Looking good!"
5. Digest timing: digest sent at consistent time (8 PM) for routine monitoring
6. Digest preferences: caregivers can customize digest timing or opt out
7. Digest history: caregivers can view past daily digests in dashboard
8. Digest comparison: digest includes comparison to previous day or week (trend indicators)
9. Digest actions: digest includes quick actions (view dashboard, send message to parent)
10. Digest language: digest delivered in caregiver's preferred language (Hindi/English)

### Story 6.5: Caregiver Dashboard - Patient Detail View

As a caregiver,
I want to see detailed information about my parent's diabetes management,
so that I can understand their patterns and provide better support.

**Acceptance Criteria:**
1. Patient detail page: accessible from caregiver dashboard, shows comprehensive patient information
2. Glucose trends: displays glucose readings over time (last 7 days, 30 days) as simple chart or table
3. Medication adherence: shows adherence percentage (daily, weekly, monthly) with trend indicator
4. Detected patterns: shows patient's food-glucose, activity-impact patterns (from Epic 4) if available
5. Recent alerts: displays recent alerts (emergencies, missed medications) with timestamps and resolution status
6. Meal logging: shows recent meals logged with timestamps (if patient logs meals)
7. Caregiver actions: shows caregiver's recent actions (alerts acknowledged, messages sent)
8. Patient summary: quick summary card showing key metrics (average glucose, adherence, alert count)
9. Data privacy: caregiver sees patient data but cannot modify patient settings (read-only access)
10. Navigation: easy navigation back to dashboard overview or to patient messaging

### Story 6.6: Caregiver-Primary Management Mode

As a caregiver managing an elderly parent,
I want to manage their ProCare account myself,
so that my parent doesn't need to use technology.

**Acceptance Criteria:**
1. Management mode selection: patient can select caregiver-primary mode during onboarding or later
2. Caregiver-primary setup: caregiver sets up patient account, manages medications, receives all notifications
3. Patient experience: patient receives minimal notifications (only critical alerts), caregiver handles routine management
4. Caregiver permissions: caregiver can log glucose, meals, medications on behalf of patient
5. Caregiver logging: caregiver can log patient data via WhatsApp or web dashboard
6. Patient independence: patient can still log data themselves if they want, but caregiver is primary manager
7. Mode switching: patient or doctor can switch between patient-primary, shared, and caregiver-primary modes
8. Mode indicators: system clearly indicates current management mode in caregiver and patient dashboards
9. Mode documentation: clear documentation explains caregiver-primary mode and its implications
10. Mode testing: caregiver-primary mode tested with elderly patients and caregivers, feedback incorporated

### Story 6.7: Caregiver-Patient Messaging via Dashboard

As a caregiver,
I want to send messages to my parent via the dashboard,
so that I can provide encouragement and reminders.

**Acceptance Criteria:**
1. Messaging interface: caregiver dashboard includes messaging interface for sending messages to patient
2. Message composition: caregivers can type messages, select from templates, or use suggested messages
3. Message delivery: messages sent to patient via WhatsApp (integrated with WhatsApp API)
4. Message history: caregivers can view message history with patient (sent and received)
5. Message templates: caregivers can create and use message templates for common scenarios
6. Suggested messages: dashboard suggests messages based on patient alerts or status (e.g., "Great job taking your medication today!")
7. Message context: messages include patient context (recent glucose, adherence) for caregiver reference
8. Message scheduling: caregivers can schedule messages to be sent later (e.g., medication reminders)
9. Message status: caregivers can see if message was delivered and read by patient
10. Message integration: patient messages appear in caregiver dashboard, caregiver responses appear in patient WhatsApp

### Story 6.8: Caregiver Dashboard Performance & Anxiety Reduction

As a caregiver,
I want the dashboard to be simple and reassuring,
so that I feel less anxious, not more.

**Acceptance Criteria:**
1. Dashboard design: clean, simple design that reduces anxiety (not overwhelming with data)
2. Status indicators: clear visual indicators (green/yellow/red) for patient status
3. Positive messaging: dashboard emphasizes positive progress, not just problems
4. Alert filtering: caregivers can filter alerts to see only critical ones (reduce anxiety from minor alerts)
5. Dashboard performance: dashboard loads quickly (<500ms), data updates smoothly
6. Mobile optimization: dashboard optimized for mobile devices (caregiver's primary device)
7. Help documentation: clear help documentation explains dashboard features and what alerts mean
8. False alarm prevention: system minimizes false alarms (validates alerts before sending)
9. Anxiety reduction metrics: dashboard tracks caregiver anxiety levels (surveys), design iterated based on feedback
10. User testing: dashboard tested with 10-15 caregivers, anxiety reduction validated through surveys

## Epic 7: Proactive Pattern-Based Interventions

### Epic Goal

Enable proactive, pattern-based interventions that prevent problems before they occur, rather than reacting after glucose spikes or medication misses. After patterns are detected (Epic 4), implement pre-meal warnings that alert patients before they eat problematic foods (e.g., "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"), enabling patients to make better choices in real-time. Implement notification bundling where multiple agent responses are combined into single messages, and enforce rate limiting (max 5 notifications/day, 2-hour minimum spacing) to avoid overwhelming patients. This epic delivers the "don't forget umbrella" vs "you're wet, here's a towel" experience, where prevention is easier than correction. The system works offline-first, storing intervention logic locally and triggering interventions based on learned patterns.

### Story 7.1: Pre-Meal Timing Detection

As a patient,
I want ProCare to warn me before I eat something that will spike my glucose,
so that I can make a better choice.

**Acceptance Criteria:**
1. Meal timing detection: system detects when patient is likely about to eat based on patterns (typical meal times, meal logging frequency)
2. Pattern-based triggers: interventions triggered when patient's typical meal time approaches AND patterns detected (from Epic 4)
3. Pre-meal window: system detects meal timing within 30 minutes before typical meal time
4. Pattern availability check: system verifies that food-glucose patterns are available for patient (requires Epic 4 completion)
5. Intervention eligibility: patient must have 4+ weeks of data and detectable patterns for pre-meal interventions
6. Timing detection works offline: meal timing patterns stored locally, detection works without connectivity
7. Multiple meal times: system supports multiple meal times per day (breakfast, lunch, dinner, snacks)
8. Pattern confidence: only triggers interventions for patterns with >70% confidence (from Epic 4)
9. Intervention frequency: pre-meal interventions sent max once per meal time per day
10. Timing detection logging: system logs meal timing detections for pattern refinement

### Story 7.2: Pre-Meal Warning Generation

As a patient,
I want personalized warnings about foods that affect my glucose,
so that I can choose better options.

**Acceptance Criteria:**
1. Warning generation: when meal time detected, system generates pre-meal warning based on detected patterns
2. Warning content: includes problematic food, expected glucose impact, better alternative, personalized message
3. Warning example: "Rice usually spikes your glucose to 180 mg/dL. Roti keeps you at 140 mg/dL. What are you having for lunch?"
4. Pattern selection: system selects most impactful pattern (highest glucose spike) for warning
5. Alternative suggestion: system suggests better alternative based on patterns (food that keeps glucose stable)
6. Warning personalization: warnings use patient's name, actual glucose values from their data, specific foods they eat
7. Warning language: warnings delivered in patient's preferred language (Hindi/English), warm and supportive tone
8. Warning timing: warnings sent 15-30 minutes before typical meal time (not too early, not too late)
9. Warning storage: warnings stored in database with timestamp, pattern reference, patient response
10. Warning effectiveness: system tracks if patient changed behavior after warning (logged different food)

### Story 7.3: Pre-Meal Warning Delivery via WhatsApp

As a patient,
I want to receive pre-meal warnings via WhatsApp,
so that I get them at the right time without checking an app.

**Acceptance Criteria:**
1. Warning delivery: pre-meal warnings sent to patient via WhatsApp (integrated with WhatsApp API)
2. Delivery timing: warnings sent at optimal time (15-30 minutes before meal) based on patient's patterns
3. Warning format: concise WhatsApp message with food comparison and question: "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"
4. Patient response: patient can respond with food choice, system logs response and tracks glucose outcome
5. Response tracking: system tracks if patient chose better alternative (roti) or problematic food (rice)
6. Outcome validation: system validates warning effectiveness by comparing predicted vs actual glucose after meal
7. Warning frequency: max one pre-meal warning per meal time per day (respects rate limiting)
8. Warning opt-out: patient can opt out of pre-meal warnings if they find them annoying
9. Warning preferences: patient can customize warning timing or frequency
10. Warning integration: warnings integrated with Engagement Engine for rate limiting and notification bundling

### Story 7.4: Behavior Change Tracking & Validation

As a patient,
I want ProCare to learn if its warnings help me make better choices,
so that the advice becomes more effective over time.

**Acceptance Criteria:**
1. Behavior tracking: system tracks patient responses to pre-meal warnings (chose better alternative vs problematic food)
2. Glucose outcome tracking: system tracks actual glucose readings after patient's meal choice
3. Behavior change detection: system detects if patient changed behavior (chose roti instead of rice after warning)
4. Change validation: compares glucose outcomes when patient chose better alternative vs problematic food
5. Effectiveness calculation: calculates warning effectiveness (% of times patient chose better alternative after warning)
6. Pattern refinement: if warnings effective, system increases pattern confidence; if not, refines warning approach
7. Success metrics: tracks behavior change rate (target: 30%+ patients change behavior after warning)
8. Response rate tracking: tracks patient response rate to warnings (target: 70%+ respond to warnings)
9. Long-term tracking: system tracks behavior change over time (sustained change vs one-time change)
10. Reporting: behavior change metrics included in weekly progress reports and doctor dashboard

### Story 7.5: Notification Bundling Logic

As a patient,
I want a single, coordinated message from ProCare,
so that I'm not overwhelmed with multiple notifications.

**Acceptance Criteria:**
1. Bundling detection: Engagement Engine detects when multiple agents have responses for same event or time window
2. Bundling window: responses within 2-hour window are bundled into single message
3. Response collection: Engagement Engine collects responses from all relevant agents (Clinical Monitor, Health Insights, Lifestyle Coach, etc.)
4. Message synthesis: Engagement Engine combines multiple responses into single, coherent WhatsApp message
5. Synthesis example: "Glucose logged: 128 mg/dL. Looking good! [Clinical Monitor] Your glucose is in target range. [Health Insights] This is similar to your readings after roti meals. [Lifestyle Coach] Keep up the good work!"
6. Priority ordering: emergency responses always sent first, then bundled normal responses
7. Message length: bundled messages kept concise (max 500 characters) for WhatsApp readability
8. Redundancy removal: Engagement Engine removes redundant information from multiple agent responses
9. Language consistency: bundled messages maintain consistent tone and language (Hindi/English)
10. Bundling logging: all bundled messages logged for quality improvement and debugging

### Story 7.6: Rate Limiting Enforcement

As a patient,
I want ProCare to respect my notification limits,
so that I'm not overwhelmed with messages.

**Acceptance Criteria:**
1. Rate limit rules: max 5 notifications per day, minimum 2-hour spacing between notifications
2. Rate limit tracking: Engagement Engine tracks notification count and timing for each patient
3. Notification queuing: if rate limit reached, notifications queued for next allowed window
4. Emergency override: emergency notifications (from Clinical Monitor) bypass rate limiting
5. Priority queuing: urgent notifications queued separately from normal notifications, sent first when allowed
6. Queue management: queued notifications prioritized by urgency, oldest first within same priority
7. Queue notification: patient receives notification when queue processed: "Here are your updates: [bundled message]"
8. Rate limit display: patient can check notification count via WhatsApp: "You've received 3 notifications today. 2 remaining."
9. Rate limit preferences: patient can adjust rate limits (increase or decrease) based on preferences
10. Rate limit logging: all rate limit decisions logged for monitoring and optimization

### Story 7.7: Proactive Medication Reminder Optimization

As a patient,
I want medication reminders that adapt to my patterns,
so that I get reminders at times when I'm most likely to take medications.

**Acceptance Criteria:**
1. Medication timing patterns: system analyzes when patient typically takes medications (from Epic 2 logs)
2. Optimal timing detection: identifies optimal reminder timing based on patient's actual medication-taking patterns
3. Timing optimization: adjusts reminder timing to 15 minutes before patient's typical medication time
4. Pattern-based reminders: reminders personalized based on patient's medication-taking patterns
5. Reminder effectiveness: tracks if optimized reminders improve medication adherence vs fixed-time reminders
6. Reminder refinement: system refines reminder timing based on patient response and adherence outcomes
7. Multiple medications: optimizes timing for each medication separately based on individual patterns
8. Pattern updates: reminder timing updates as patient patterns change over time
9. Reminder integration: optimized reminders integrated with medication management system (Epic 2)
10. Reminder reporting: reminder effectiveness metrics included in weekly progress reports

### Story 7.8: Intervention Effectiveness Reporting

As a doctor,
I want to see if proactive interventions are helping my patients,
so that I can understand the impact of ProCare.

**Acceptance Criteria:**
1. Intervention metrics: system tracks intervention effectiveness (pre-meal warnings, optimized reminders)
2. Behavior change metrics: tracks % of patients who changed behavior after interventions
3. Glucose outcome metrics: tracks glucose improvements after interventions (reduced spikes, better control)
4. Adherence improvement: tracks medication adherence improvements after optimized reminders
5. Intervention reporting: intervention metrics displayed in doctor dashboard for each patient
6. Aggregate reporting: doctor dashboard shows aggregate intervention effectiveness across all patients
7. Weekly summary integration: intervention effectiveness included in weekly patient summaries
8. Trend analysis: system shows intervention effectiveness trends over time (improving or declining)
9. Patient-specific reporting: doctor can see which interventions work best for each patient
10. Export functionality: doctors can export intervention effectiveness data for analysis

## Epic 8: Advanced Coordination & Question Answering

### Epic Goal

Complete the 7-agent orchestration system by implementing the remaining three agents: Care Coordinator for managing test appointments and medication refills, Doctor Bridge for intelligent question routing and AI-powered answers, and Learning Library for context-based health education. Implement HbA1c test scheduling with 2-week reminders and glucose-based predictions, AI question answering that handles 80%+ of routine questions with 20% escalation to doctors, weekly progress reports for patients and doctors, and context-based educational tips triggered by patient behavior patterns. This epic completes the MVP feature set, enabling comprehensive diabetes management support while maintaining the doctor-connected care model where AI supports but doesn't replace medical oversight.

### Story 8.1: Care Coordinator Agent - HbA1c Test Scheduling

As a patient,
I want reminders for my HbA1c test appointments,
so that I don't forget important checkups.

**Acceptance Criteria:**
1. Care Coordinator agent service created as independent microservice
2. Test scheduling: doctor or patient can schedule HbA1c test appointments (every 3 months typical)
3. Appointment tracking: system tracks upcoming test appointments with date, time, lab location
4. Two-week reminder: system sends WhatsApp reminder 2 weeks before test appointment: "Your HbA1c test is scheduled for [date]. I'll remind you again closer to the date."
5. One-week reminder: system sends reminder 1 week before: "Your HbA1c test is in 1 week. Please confirm your appointment."
6. One-day reminder: system sends reminder 1 day before: "Don't forget: Your HbA1c test is tomorrow at [time] at [lab]."
7. Appointment confirmation: patient can confirm or reschedule appointment via WhatsApp
8. Appointment storage: appointments stored in database, synced between local and cloud
9. Appointment history: system tracks past appointments and upcoming appointments
10. Appointment integration: appointments integrated with lab partner APIs (Apollo, Metropolis) for booking

### Story 8.2: Care Coordinator - HbA1c Prediction

As a patient,
I want to know what my HbA1c might be before the test,
so that I'm motivated to improve if needed.

**Acceptance Criteria:**
1. Prediction algorithm: Care Coordinator analyzes patient's glucose readings over past 3 months
2. HbA1c prediction: calculates predicted HbA1c based on average glucose (formula: HbA1c ≈ (average glucose + 46.7) / 28.7)
3. Prediction accuracy: 95%+ prediction accuracy within 0.2% of actual HbA1c (validated against lab results)
4. Prediction delivery: prediction sent with 2-week reminder: "Based on your glucose readings, your predicted HbA1c is 7.2%. Your target is <7.0%. Keep up the good work!"
5. Prediction motivation: if prediction above target, includes encouragement: "You're close to your target! A few small changes can help."
6. Prediction storage: predictions stored in database, compared with actual results for accuracy validation
7. Prediction improvement: prediction algorithm refined based on actual vs predicted comparisons
8. Prediction display: predictions shown in patient weekly progress report and doctor dashboard
9. Prediction timing: predictions calculated 2 weeks before test (when reminder sent)
10. Prediction language: predictions delivered in patient's preferred language (Hindi/English)

### Story 8.3: Care Coordinator - Lab Result Upload & Analysis

As a patient,
I want to upload my HbA1c test results,
so that ProCare can track my progress and share with my doctor.

**Acceptance Criteria:**
1. Result upload: patient can upload HbA1c test result via WhatsApp (photo of lab report) or web dashboard
2. OCR processing: system uses OCR to extract HbA1c value from lab report image
3. Result validation: system validates extracted value (typical range 4-15%), prompts for correction if invalid
4. Result storage: HbA1c results stored in database with date, value, lab name, linked to patient
5. Result analysis: system compares actual result with prediction, calculates accuracy
6. Result comparison: system compares current result with previous results, shows trend (improving, stable, worsening)
7. Result notification: patient receives WhatsApp: "Your HbA1c result: 7.1% (down from 7.5%!). Great progress! Your doctor will review this."
8. Doctor notification: doctor receives alert in dashboard: "Patient [Name] HbA1c: 7.1% (predicted 7.2%, actual 7.1%). Trend: improving."
9. Result display: results displayed in patient detail view in doctor dashboard with trend chart
10. Result history: patients and doctors can view HbA1c history over time

### Story 8.4: Doctor Bridge Agent - Question Classification

As a patient,
I want to ask questions about my diabetes,
so that I get answers quickly without bothering my doctor unnecessarily.

**Acceptance Criteria:**
1. Doctor Bridge agent service created as independent microservice
2. Question reception: patient questions received via WhatsApp (text or voice)
3. Question classification: Doctor Bridge classifies questions into categories (nutrition, medication, timing, general, medical)
4. Classification accuracy: 90%+ classification accuracy for question types
5. Routine vs medical: questions classified as "routine" (AI can answer) or "medical" (escalate to doctor)
6. Routine categories: nutrition ("Can I eat mango?"), timing ("When should I check glucose?"), general diabetes management
7. Medical categories: medication questions ("Should I increase my dose?"), symptoms ("I feel dizzy"), treatment questions
8. Question storage: all questions stored in database with classification, patient response, outcome
9. Question context: questions include patient context (recent glucose, medications, patterns) for better classification
10. Question logging: all questions logged for analysis and improvement

### Story 8.5: Doctor Bridge - AI Question Answering

As a patient,
I want AI to answer my routine questions,
so that I get immediate help without waiting for my doctor.

**Acceptance Criteria:**
1. AI integration: Doctor Bridge integrates with conversational AI (GPT-4 or Claude) for question answering
2. Context-aware answers: AI answers include patient's specific context (glucose patterns, medications, recent data)
3. Answer generation: AI generates personalized answers based on patient's data and question
4. Answer examples: "Can I eat mango?" → "Based on your patterns, mango usually spikes your glucose to 160. A small piece (50g) might be okay, but check your glucose 2 hours after. Roti keeps you stable at 140."
5. Answer accuracy: 80%+ of routine questions answered correctly by AI (validated through doctor review)
6. Answer delivery: answers sent to patient via WhatsApp within 2 seconds (per NFR2)
7. Answer language: answers delivered in patient's preferred language (Hindi/English)
8. Answer tone: answers warm, supportive, not judgmental (per brand guidelines)
9. Answer validation: answers reviewed by medical advisor, refined based on feedback
10. Answer improvement: AI answers improve over time based on patient feedback and doctor corrections

### Story 8.6: Doctor Bridge - Question Escalation to Doctor

As a doctor,
I want to see questions that need my attention,
so that I can provide medical guidance when AI isn't appropriate.

**Acceptance Criteria:**
1. Escalation triggers: medical questions, medication questions, symptom questions automatically escalated
2. Escalation accuracy: 90%+ of escalated questions require doctor attention (validated)
3. Escalation display: escalated questions appear in doctor dashboard with patient context
4. Escalation priority: medical questions marked as "Attention" tier, urgent symptoms as "Urgent"
5. Escalation notification: doctor receives alert: "Patient [Name] asked: [question]. This requires your attention."
6. Doctor response: doctor can respond via dashboard or WhatsApp, response sent to patient
7. Response tracking: system tracks doctor response time, patient satisfaction with answers
8. Escalation history: doctors can view escalation history for each patient
9. Escalation learning: system learns which questions to escalate based on doctor responses
10. Escalation reporting: escalation metrics included in weekly patient summaries

### Story 8.7: Weekly Progress Reports for Patients

As a patient,
I want a weekly summary of my progress,
so that I can see how I'm doing and stay motivated.

**Acceptance Criteria:**
1. Report generation: system generates weekly progress report for each patient every Monday
2. Report content: includes glucose summary (average, range, readings count), medication adherence, meals logged, patterns detected, wins, opportunities
3. Report format: concise, easy-to-read format delivered via WhatsApp: "Your Week in Review: Glucose average 128 mg/dL (great!), Medications 95% adherence (excellent!), 12 meals logged. Win: You chose roti over rice 4 times this week! Opportunity: Try walking 30 minutes after dinner."
4. Report personalization: reports use patient's name, actual data, specific achievements
5. Report timing: reports sent Monday morning for previous week's review
6. Report language: reports delivered in patient's preferred language (Hindi/English)
7. Report storage: reports stored in database, patients can view past reports in dashboard
8. Report comparison: reports include comparison to previous week (trend indicators)
9. Report motivation: reports emphasize wins and progress, not just problems
10. Report opening: 75%+ patients open weekly summary (per NFR14)

### Story 8.8: Weekly Progress Reports for Doctors

As a doctor,
I want weekly summaries of my patients' progress,
so that I can efficiently review multiple patients in one session.

**Acceptance Criteria:**
1. Report generation: system generates weekly progress report for each patient every Monday morning
2. Report content: includes glucose trends, medication adherence, detected patterns, alerts, key insights, intervention effectiveness
3. Report format: concise format displayed in doctor dashboard: "Patient [Name] - Week Summary: Glucose avg 128 mg/dL (stable), Adherence 95% (excellent), Patterns: Rice spikes glucose, roti stable. Alerts: 0. Status: Stable."
4. Report batch view: doctors can view all patient summaries in batch (all patients at once)
5. Report metrics: reports include key metrics (average glucose, adherence, time in range, pattern changes)
6. Report insights: reports include Health Insights agent-generated insights (from Epic 4)
7. Report alerts: reports highlight patients needing attention based on weekly data
8. Report export: doctors can export weekly summaries as PDF or CSV for records
9. Report timing: reports generated Monday morning, available for doctor's weekly review (10-minute session)
10. Report history: doctors can view past weekly summaries for trend analysis

### Story 8.9: Learning Library Agent - Context-Based Education

As a patient,
I want educational tips that are relevant to my situation,
so that I learn what I need to know when I need it.

**Acceptance Criteria:**
1. Learning Library agent service created as independent microservice
2. Context detection: agent detects patient context (high glucose, missed medication, new pattern detected)
3. Content selection: agent selects relevant educational content based on context
4. Content types: simple, actionable tips (not lengthy articles) - "When glucose is high, drink water and avoid food"
5. Content delivery: educational tips included in patient WhatsApp messages (synthesized by Engagement Engine)
6. Content timing: tips delivered when relevant (after high glucose reading, after missed medication)
7. Content personalization: tips personalized based on patient's patterns and data
8. Content library: library of 50-100 educational tips covering common diabetes management topics
9. Content language: tips delivered in patient's preferred language (Hindi/English)
10. Content effectiveness: system tracks if tips lead to behavior change, refines content based on effectiveness

### Story 8.10: Care Coordinator - Medication Refill Tracking

As a patient,
I want reminders when my medications are running low,
so that I don't run out of important medications.

**Acceptance Criteria:**
1. Medication tracking: Care Coordinator tracks medication supply based on prescription and usage
2. Refill calculation: system calculates when medication will run out based on dosage and frequency
3. Refill reminder: system sends WhatsApp reminder 1 week before medication runs out: "Your Metformin is running low. Time to refill! Contact your doctor or pharmacy."
4. Refill tracking: patient can log medication refills via WhatsApp: "refilled metformin" or "got new prescription"
5. Refill history: system tracks medication refill history for adherence and supply management
6. Refill alerts: if medication not refilled and runs out, alert sent to patient and doctor
7. Refill integration: refill reminders integrated with pharmacy APIs (future Phase 2 feature)
8. Refill display: medication supply status displayed in patient and doctor dashboards
9. Refill reporting: medication refill compliance included in weekly progress reports
10. Refill optimization: system learns patient's refill patterns, optimizes reminder timing

## Checklist Results Report

### Executive Summary

**Overall PRD Completeness:** 92% - The PRD is comprehensive and well-structured, covering all essential aspects of the ProCare MVP.

**MVP Scope Appropriateness:** Just Right - The scope is appropriately sized for MVP, with clear boundaries between MVP and Phase 2 features. All 8 epics deliver incremental value and are logically sequenced.

**Readiness for Architecture Phase:** Ready - The PRD provides sufficient technical guidance, clear requirements, and well-defined epics/stories for the Architect to begin system design.

**Most Critical Gaps or Concerns:**
- Some technical implementation details may need clarification during architecture phase (offline sync conflict resolution specifics)
- Payment/subscription management requirements not explicitly detailed (implied but not specified)
- User authentication flows could be more detailed (OTP, password reset, etc.)

### Category Analysis

| Category                         | Status | Critical Issues |
| -------------------------------- | ------ | --------------- |
| 1. Problem Definition & Context  | PASS   | None - Well-defined problem, clear user personas, quantified impact |
| 2. MVP Scope Definition          | PASS   | None - Clear MVP boundaries, eliminated features documented, rationale provided |
| 3. User Experience Requirements  | PASS   | Minor - Core screens identified, but detailed user flows could be expanded |
| 4. Functional Requirements       | PASS   | None - 20 functional requirements clearly defined, testable, user-focused |
| 5. Non-Functional Requirements   | PASS   | None - 20 NFRs covering performance, security, compliance, scalability |
| 6. Epic & Story Structure        | PASS   | None - 8 epics, 70+ stories, logical sequencing, appropriate sizing |
| 7. Technical Guidance            | PASS   | Minor - Architecture direction provided, but some implementation details deferred |
| 8. Cross-Functional Requirements | PASS   | Minor - Data requirements clear, integrations identified, operational needs specified |
| 9. Clarity & Communication       | PASS   | None - Well-structured, consistent terminology, clear documentation |

### Top Issues by Priority

**BLOCKERS:** None identified - PRD is ready for architecture phase.

**HIGH Priority:**
1. **Payment/Subscription Management:** While business model is clear (₹99-999/month tiers, doctor commissions), specific payment gateway integration, subscription management, and billing requirements should be added as functional requirements.
2. **User Authentication Details:** OTP flow, password reset, session management details could be more explicit in Epic 1 stories.

**MEDIUM Priority:**
1. **Detailed User Flows:** While core screens are identified, detailed user journey maps for key workflows (onboarding, glucose logging, emergency response) would enhance UX clarity.
2. **Error Handling Specifications:** While error handling is mentioned in stories, comprehensive error scenarios and recovery flows could be more detailed.
3. **Data Migration Strategy:** For existing patients or data imports, migration approach should be documented.

**LOW Priority:**
1. **Visual Design Guidelines:** While branding direction is provided, specific design system details can be deferred to UX phase.
2. **Analytics Requirements:** While metrics are defined, specific analytics implementation requirements could be detailed.

### MVP Scope Assessment

**Features Appropriately Scoped:**
- ✅ WhatsApp-first interface (non-negotiable, correctly prioritized)
- ✅ Offline-first functionality (essential for India market)
- ✅ 7-agent orchestration (complex but core differentiator)
- ✅ Pattern detection (4-week requirement is realistic)
- ✅ Doctor and caregiver dashboards (both essential for value delivery)

**Features That Could Be Deferred (Already Out of Scope):**
- ❌ Native mobile apps (correctly deferred to Phase 2)
- ❌ CGM integration (correctly deferred to Phase 2)
- ❌ Multi-language beyond Hindi/English (correctly deferred to Phase 2)
- ❌ Detailed meal nutrition calculations (correctly simplified per requirements)

**Complexity Concerns:**
- **Offline Sync Engine:** High complexity but essential - conflict resolution strategy documented, but implementation details will need architect deep-dive
- **7-Agent Orchestration:** High complexity but core differentiator - architecture well-defined, but coordination logic needs detailed design
- **Pattern Detection:** Medium complexity - 4-week learning period is realistic, algorithm details can be refined during development

**Timeline Realism:**
- 8 epics with 70+ stories is substantial but achievable for MVP
- Epic sequencing is logical and allows parallel development where possible
- 12-week MVP timeline (from brief) is ambitious but feasible with focused team

### Technical Readiness

**Clarity of Technical Constraints:**
- ✅ Platform requirements clearly specified (WhatsApp, web, offline-first)
- ✅ Technology stack preferences documented (React, Node.js/Python, PostgreSQL, SQLite)
- ✅ Architecture approach defined (microservices, monorepo, event-driven)
- ✅ Integration requirements identified (WhatsApp API, lab partners, payment gateways)

**Identified Technical Risks:**
1. **WhatsApp Business API Approval:** 2-4 week approval process could delay MVP - mitigation: apply early, parallel development
2. **Offline Sync Complexity:** Conflict resolution and sync engine are complex - mitigation: event-sourcing model documented, extensive testing planned
3. **AI Accuracy:** 80%+ question answering accuracy requires validation - mitigation: human-in-loop pilot, medical advisor review
4. **Pattern Detection Accuracy:** 95%+ prediction accuracy within 15 mg/dL requires validation - mitigation: 4-week learning period, continuous validation

**Areas Needing Architect Investigation:**
1. **Offline Sync Conflict Resolution:** Specific algorithms and edge cases
2. **Message Bus Scalability:** RabbitMQ/Redis performance at 10K+ users
3. **AI API Costs:** Cost optimization for conversational AI at scale
4. **Time-Series Database:** InfluxDB vs TimescaleDB performance comparison
5. **Event Sourcing Implementation:** Event store design, replay mechanisms

### Recommendations

**Immediate Actions:**
1. **Add Payment Requirements:** Create FR21-FR23 for payment gateway integration, subscription management, and billing
2. **Expand Authentication Stories:** Add detailed OTP flow and password reset stories to Epic 1
3. **Document Error Scenarios:** Add comprehensive error handling acceptance criteria to key stories

**Before Architecture Phase:**
1. **Technical Deep-Dive:** Architect should review offline sync requirements and propose specific conflict resolution algorithms
2. **Integration Planning:** Detailed integration specifications for WhatsApp API, lab partners, payment gateways
3. **Performance Modeling:** Load testing requirements and performance benchmarks for 7-agent orchestration

**During Architecture Phase:**
1. **Detailed User Flows:** UX Expert should create detailed journey maps for key workflows
2. **Design System:** Establish visual design guidelines and component library
3. **API Specifications:** Detailed API contracts for agent-to-agent communication

### Final Decision

**READY FOR ARCHITECT:** The PRD and epics are comprehensive, properly structured, and ready for architectural design. The identified gaps are minor and can be addressed during the architecture phase or through follow-up clarifications. The MVP scope is appropriate, requirements are clear and testable, and technical guidance is sufficient for the Architect to begin system design.

## Next Steps

### UX Expert Prompt

Create the UX design architecture for ProCare based on this PRD. Focus on:

1. **WhatsApp Conversational Interface:** Design the conversational flows for patient interactions (glucose logging, meal logging, medication reminders, question answering). Ensure the experience feels natural, supportive, and reduces anxiety.

2. **Doctor Dashboard UX:** Design the tiered patient view (Urgent/Attention/Stable), patient detail views, alert management, and weekly summary interfaces. Optimize for 10-15 minute daily review time.

3. **Caregiver Dashboard UX:** Design the caregiver dashboard and WhatsApp alerts with focus on reducing anxiety, not amplifying it. Ensure setup is simple and primarily on caregiver's device.

4. **Offline-First UX:** Design visual indicators for sync status, offline mode, and connectivity states. Ensure users understand when data is synced vs pending.

5. **Accessibility:** Ensure all designs meet WCAG AA standards, with particular attention to elderly users (60+) and low-tech-literacy users.

6. **India-Specific Considerations:** Design for Hindi/English bilingual support, cultural sensitivity, and WhatsApp-native experience.

Use the PRD requirements, user personas, and core screens identified in the UI Design Goals section as your foundation. Create detailed user journey maps, wireframes, and interaction specifications.

### Architect Prompt

Create the technical architecture for ProCare based on this PRD. Focus on:

1. **7-Agent Microservices Architecture:** Design the microservices architecture for the 7-agent orchestration system (Engagement Engine, Clinical Monitor, Lifestyle Coach, Health Insights, Doctor Bridge, Care Coordinator, Learning Library). Define agent communication patterns, message bus design, and orchestration logic.

2. **Offline-First Architecture:** Design the offline-first data architecture with local SQLite storage, sync engine, conflict resolution, and event-sourcing model. Specify sync algorithms, conflict resolution strategies, and bandwidth-aware sync logic.

3. **WhatsApp Integration Architecture:** Design the WhatsApp Business API integration, webhook handling, message queuing, and rate limiting system.

4. **Database Architecture:** Design the database schema (PostgreSQL for cloud, SQLite for local), time-series database for glucose readings, and data sync mechanisms.

5. **Security & Compliance:** Design authentication/authorization, data encryption, HIPAA-equivalent compliance, and India data protection compliance.

6. **Scalability & Performance:** Design for 1,000+ concurrent users (scaling to 10K+), ensure <2s AI response times, <500ms dashboard loads, and handle WhatsApp API rate limits.

Use the Technical Assumptions section, functional/non-functional requirements, and epic stories as your foundation. Address the technical risks identified in the Checklist Results Report, particularly offline sync complexity and 7-agent orchestration scalability.

