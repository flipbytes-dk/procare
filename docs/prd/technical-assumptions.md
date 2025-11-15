# Technical Assumptions

## Repository Structure: Monorepo

ProCare will use a monorepo approach with a single repository containing multiple services (agents). This enables:
- Shared libraries for common utilities (database models, API clients, event schemas)
- Easier code sharing between agents and frontend/backend
- Simplified dependency management across the 7-agent orchestration system
- Atomic commits across multiple services when needed
- Consistent tooling and CI/CD pipelines

**Rationale:** The 7-agent architecture requires tight coordination and shared data models. A monorepo simplifies development while maintaining service independence. Each agent remains an independent microservice but shares common infrastructure code.

## Service Architecture

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

## Testing Requirements

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

## Additional Technical Assumptions and Requests

**Frontend Technologies:**
- **Native Mobile App (MVP - Primary Patient Interface):** React Native 0.74+ with Expo SDK 51+ for iOS and Android as the primary interface for all patient interactions including glucose logging, meal logging, medication reminders, and question answering. Supports flexible UX (chat-based primary interface with direct input options like tags for quick data entry) while maintaining conversational capabilities.
- **WhatsApp (MVP - Secondary Patient Channel):** WhatsApp Business API integration for re-engagement, templated interactions, summaries, and basic data collection when the app is not being used. Conversational-only interface. Both channels always available - patients can use either at any time.
- **Web Dashboards (MVP):** Next.js 14+ with React 18+ for doctor and caregiver dashboards (desktop-first for doctors, mobile-responsive for caregivers)
- **Phase 2:** Additional web chat interface for patients via browser (Open WebUI or similar) as alternative to mobile app

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
