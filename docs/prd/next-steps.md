# Next Steps

## UX Expert Prompt

Create the UX design architecture for ProCare based on this PRD. Focus on:

1. **WhatsApp Conversational Interface:** Design the conversational flows for patient interactions (glucose logging, meal logging, medication reminders, question answering). Ensure the experience feels natural, supportive, and reduces anxiety.

2. **Doctor Dashboard UX:** Design the tiered patient view (Urgent/Attention/Stable), patient detail views, alert management, and weekly summary interfaces. Optimize for 10-15 minute daily review time.

3. **Caregiver Dashboard UX:** Design the caregiver dashboard and WhatsApp alerts with focus on reducing anxiety, not amplifying it. Ensure setup is simple and primarily on caregiver's device.

4. **Offline-First UX:** Design visual indicators for sync status, offline mode, and connectivity states. Ensure users understand when data is synced vs pending.

5. **Accessibility:** Ensure all designs meet WCAG AA standards, with particular attention to elderly users (60+) and low-tech-literacy users.

6. **India-Specific Considerations:** Design for Hindi/English bilingual support, cultural sensitivity, and WhatsApp-native experience.

Use the PRD requirements, user personas, and core screens identified in the UI Design Goals section as your foundation. Create detailed user journey maps, wireframes, and interaction specifications.

## Architect Prompt

Create the technical architecture for ProCare based on this PRD. Focus on:

1. **7-Agent Microservices Architecture:** Design the microservices architecture for the 7-agent orchestration system (Engagement Engine, Clinical Monitor, Lifestyle Coach, Health Insights, Doctor Bridge, Care Coordinator, Learning Library). Define agent communication patterns, message bus design, and orchestration logic.

2. **Offline-First Architecture:** Design the offline-first data architecture with local SQLite storage, sync engine, conflict resolution, and event-sourcing model. Specify sync algorithms, conflict resolution strategies, and bandwidth-aware sync logic.

3. **WhatsApp Integration Architecture:** Design the WhatsApp Business API integration, webhook handling, message queuing, and rate limiting system.

4. **Database Architecture:** Design the database schema (PostgreSQL for cloud, SQLite for local), time-series database for glucose readings, and data sync mechanisms.

5. **Security & Compliance:** Design authentication/authorization, data encryption, HIPAA-equivalent compliance, and India data protection compliance.

6. **Scalability & Performance:** Design for 1,000+ concurrent users (scaling to 10K+), ensure <2s AI response times, <500ms dashboard loads, and handle WhatsApp API rate limits.

Use the Technical Assumptions section, functional/non-functional requirements, and epic stories as your foundation. Address the technical risks identified in the Checklist Results Report, particularly offline sync complexity and 7-agent orchestration scalability.

