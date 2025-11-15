# ProCare

AI-powered diabetes management platform for India, connecting patients, doctors, and caregivers to provide continuous monitoring, personalized guidance, and early intervention.

## Overview

ProCare helps patients manage chronic diseases (starting with diabetes) between doctor visits. It connects patients, doctors, and AI to provide continuous monitoring, personalized guidance, and early intervention.

## Key Features

### MVP Priorities

1. **Native Mobile App (Primary)** - React Native app for comprehensive patient experience
2. **WhatsApp Integration (Secondary)** - Re-engagement and basic interactions (550M+ users in India)
3. **Doctor Web Dashboard** - Tiered view for efficient patient triage (10-15 min/day vs 2 hours)
4. **Multiple Input Methods** - Text, photo, OCR for glucose and meal logging
5. **AI-Powered Pattern Detection** - Personalized insights after 4 weeks of data

### Core Capabilities

- Real-time glucose monitoring with emergency detection
- Personalized meal and activity guidance based on patient's own data
- Medication adherence tracking with smart reminders
- Pattern detection and predictive insights (after 4 weeks of data)
- Efficient doctor dashboard with priority-based alerts
- Care coordination (tests, refills, appointments)
- Educational content delivery
- AI-powered question answering (80%+ routine questions)

## Documentation

- **[Requirements Document](context/ProCare_Requirements_Plain_English.docx.txt)** - Complete functional requirements in plain English
- **[Brainstorming Session Results](docs/brainstorming-session-results.md)** - Comprehensive brainstorming output including:
  - 150+ questions across 15 categories
  - Stakeholder role-playing (Patient, Doctor, Caregiver)
  - Critical scenarios (WhatsApp-only, Offline-first)
  - SCAMPER feature exploration
  - Convergence & synthesis with action plan

## Key Insights

### India-Specific Requirements (Core Focus)
- Native mobile app with WhatsApp integration for broad reach
- Local storage with background sync capability for connectivity challenges
- Multiple input methods: text, photo OCR for accessibility
- Doctor-connected care model for trust and clinical oversight

### First Week/Month Experience Determines Adoption
- Patient: First week must show personalized insights, reduce anxiety
- Doctor: First month must save time, show measurable improvement
- Caregiver: First week must reduce anxiety, not amplify it

### Technical Moat
- 7-agent orchestration architecture creates 18-24 month competitive advantage
- AI-powered pattern detection based on patient's own data
- Doctor-connected care model with clinical oversight
- Type-safe end-to-end development with tRPC

## Development Roadmap

### Phase 1: MVP Development (Weeks 1-24)
- Weeks 1-4: Foundation (Infrastructure, mobile app, database, basic logging)
- Weeks 5-8: Core Features (Medication management, agent orchestration, emergency detection)
- Weeks 9-16: Advanced Features (Pattern detection, doctor dashboard, AI Q&A)
- Weeks 17-24: Integration & Testing (Cross-platform testing, pilot validation, launch preparation)

### Phase 2: Launch & Iteration (Months 6-8)
- Soft launch with 100-500 patients, 5-10 doctors
- Wizard of Oz testing (humans behind AI)
- Refine AI based on learnings
- Add caregiver features and alerts

### Phase 3: Scale (Months 9-12)
- Regional expansion to 3-5 cities
- Add advanced integrations (Bluetooth devices, EMR systems)
- Comprehensive diet and lifestyle module
- Target: 100K+ users, 30K+ paid subscribers

## Success Metrics

### Patient Health Outcomes
- HbA1c reduction: Average 0.5-1.0% improvement in 3 months
- Time in range: 70%+ of glucose readings in 70-180 mg/dL range
- Medication adherence: 85%+ patients taking meds as prescribed

### Patient Engagement
- Daily active users: 80%+ of patients logging at least once per day
- 30-day retention: 85%+ patients still active after first month
- Response rate: 60%+ patients respond to AI prompts within 2 hours

### Doctor Satisfaction
- Time saved: 15-20 minutes per patient per month
- Dashboard usage: Doctors checking 3x per week minimum
- Retention: 90%+ of doctors stay on platform after 6 months

## Business Model

### Revenue Streams
- **B2C:** Patient subscriptions (₹99-999/month)
- **B2B:** Doctor commissions (₹3,600/patient/year)
- **B2B2C:** TPA/institutional contracts
- **Data:** Research, analytics, risk scoring

### Pricing Tiers
- **Free Tier (Mobile Basic):** ₹0 - Basic logging, reminders, Q&A
- **Paid Tier (Mobile Premium):** ₹99-199/month - Enhanced features, doctor consultation
- **Professional Tier (Mobile + Web + Caregiver):** ₹499-999/month - Full feature suite

## Technology Stack

- **Mobile App:** React Native + Expo (primary patient interface)
- **WhatsApp Integration:** Evolution API for secondary channel
- **Web Dashboard:** Next.js 14 + ShadCN (for doctors)
- **Backend:** Node.js microservices with 7-agent orchestration
- **Database:** PostgreSQL + TimescaleDB (cloud), WatermelonDB (local storage)
- **AI/ML:** OpenRouter API gateway for 400+ LLMs
- **Infrastructure:** Self-hosted on Hetzner Cloud with Coolify
- **API Style:** tRPC for end-to-end type safety

## Contributing

This is a private repository. For access or contributions, please contact the repository owner.

## License

Proprietary - All rights reserved

---

**Last Updated:** November 7, 2025

