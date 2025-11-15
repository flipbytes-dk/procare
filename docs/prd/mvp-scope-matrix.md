# MVP Scope & Phasing Matrix

## Executive Summary

This document clarifies which features are included in the MVP (Minimum Viable Product) versus Phase 2, provides rationale for scope decisions, and defines recommended timelines.

**MVP Timeline Recommendation:** 20-24 weeks (extended from original 12-week estimate)
**MVP Core Value:** AI-powered diabetes management with doctor-connected care, emergency detection, and personalized pattern-based interventions

---

## MVP (Phase 1): Core Diabetes Management Platform

**Duration:** 20-24 weeks
**Goal:** Deliver core diabetes management loop with AI guidance, doctor oversight, and pattern-based interventions

### Epics Included in MVP

| Epic | Title | Stories | Rationale | Priority |
|------|-------|---------|-----------|----------|
| **Epic 1** | Foundation & Core Patient Logging | 13 stories | Essential infrastructure, mobile app, WhatsApp, glucose/meal logging | CRITICAL |
| **Epic 2** | Medication Management & Reminders | 8 stories | Core adherence improvement (85%+ target), smart reminders with escalation | CRITICAL |
| **Epic 3** | Agent Orchestration & Emergency Detection | 8 stories | Patient safety (emergency detection), foundational 7-agent architecture | CRITICAL |
| **Epic 4** | Pattern Detection & Health Insights | 8 stories | Core differentiator - personalized insights after 4 weeks of data | CRITICAL |
| **Epic 5** | Doctor Dashboard & Patient Management | 9 stories | Doctor oversight, tiered patient view, pooled questions, QR onboarding | CRITICAL |
| **Epic 8** | Advanced Coordination & Question Answering | 12 stories | AI Q&A (80%+ routine questions), Care Coordinator, weekly reports, test scheduling | HIGH |

**Total MVP Stories:** ~58 stories

### MVP Feature Highlights

✅ **Patient Experience:**
- Native mobile app (iOS + Android) as primary interface
- WhatsApp as secondary channel for re-engagement
- Glucose logging (text, voice, photo OCR)
- Meal logging (photo + text, simplified)
- Medication reminders with escalation
- AI question answering (routine questions)
- Pattern-based insights after 4 weeks
- Proactive pre-meal warnings (requires Epic 4 + 4 weeks data)
- Offline-first functionality
- Trial period management (1 month for referred patients)

✅ **Doctor Experience:**
- Self-registration and dashboard
- Tiered patient view (Urgent/Attention/Stable/"Call in")
- Patient search
- Doctor-initiated onboarding with QR code
- Referring doctor workflow (non-diabetes specialists)
- Pooled questions (AI pools similar questions, doctor provides batch answers)
- Alert management with appointment scheduling
- Daily patient summaries
- HbA1c prediction
- Blood test scheduling and reminders

✅ **AI & Agents:**
- 7-agent orchestration system (Engagement Engine, Clinical Monitor, Health Insights, Doctor Bridge, Care Coordinator)
- Emergency detection (hypoglycemia/hyperglycemia)
- Pattern detection (food-glucose, activity impact, medication consistency)
- AI question classification and answering
- Notification bundling and rate limiting

✅ **Technical Foundation:**
- PostgreSQL (cloud) + SQLite (local) databases
- Offline-first architecture with sync engine
- RabbitMQ message bus for agent communication
- tRPC for type-safe API
- React Native + Expo for mobile
- Next.js for web dashboards

---

## Phase 2: Comprehensive Health Platform

**Duration:** 8-12 weeks (after MVP launch)
**Goal:** Add comprehensive features that differentiate from competitors and expand market reach

### Epics Deferred to Phase 2

| Epic | Title | Stories | Rationale for Deferral | Dependencies |
|------|-------|---------|------------------------|--------------|
| **Epic 6** | Caregiver Features & Alerts | 8 stories | Important but not critical for MVP - can launch with patient+doctor only | Requires Epic 2, 3 foundation |
| **Epic 7** | Proactive Pattern-Based Interventions | 8 stories | Requires Epic 4 + 4 weeks of patient data - naturally deferred | **Depends on Epic 4** |
| **Epic 9** | Health Records Management | 5 stories | Major value prop but can be added post-launch to increase stickiness | None (independent) |
| **Epic 10** | Diet/Lifestyle Module | 4 stories | Critical for T2D management (95% is diet) but can enhance MVP post-launch | None (independent) |
| **Epic 11** | Prescription Upload & Data Extraction | 2 stories | Streamlines medication entry but manual entry acceptable for MVP | Requires Epic 2 |
| **Epic 12** | Integrations & Global Support | 6 stories | Bluetooth devices, EMR, HMS, global language packs - can add incrementally | None (independent) |
| **Epic 13** | Appointment Booking & Slot Reservation | Stories TBD | Payment integration and appointment automation - can be manual initially | Requires payment gateway |

**Total Phase 2 Stories:** ~37 stories

### Phase 2 Feature Highlights

✅ **Caregiver Features (Epic 6):**
- Separate caregiver dashboard (mobile app + web)
- Real-time critical alerts
- Daily digest summaries
- Caregiver-primary management mode

✅ **Advanced Interventions (Epic 7):**
- Pre-meal timing detection
- Pre-meal warning generation
- Warning effectiveness tracking
- Medication timing optimization

✅ **Health Records (Epic 9):**
- Comprehensive health records upload/storage
- Query and download capabilities
- Auto-populate doctor EMR
- Health records viewer

✅ **Diet Module (Epic 10):**
- Calorie management
- Food swaps recommendations
- Specific diet types (low sodium, low potassium, low fat)
- Menu recommendations from photos

✅ **Prescription OCR (Epic 11):**
- Prescription upload (photo/document)
- OCR extraction (medication, diet type, dosage)
- Automatic medication schedule creation

✅ **Advanced Integrations (Epic 12):**
- Bluetooth glucometer integration
- Google Fit / Apple Health integration
- EMR/HMS system integration
- Lab partner APIs (Apollo, Metropolis)
- Payment gateway (Razorpay) - **Note: Basic payment tracking in Epic 1.14**
- Global location packs (languages, protocols, time zones)

✅ **Appointment Automation (Epic 13):**
- Automated appointment booking
- Slot reservation
- Payment integration (ProCare takes 10% for securing appointment)

---

## Rationale for MVP Scope Decisions

### Why These 6 Epics in MVP?

**Epic 1 (Foundation):** Non-negotiable - required for everything else

**Epic 2 (Medication):** Core adherence improvement (85%+ target vs 50% baseline) - immediate measurable impact

**Epic 3 (Agent Orchestration & Emergency):** Patient safety is critical - emergency detection cannot be deferred

**Epic 4 (Pattern Detection):** Core differentiator - personalized insights distinguish ProCare from generic apps

**Epic 5 (Doctor Dashboard):** Doctor oversight enables doctor-connected care model - essential for trust and acquisition

**Epic 8 (AI Q&A & Care Coordinator):** Reduces doctor burden (60% fewer emergency calls), enables 80%+ routine question answering

### Why Defer These Epics to Phase 2?

**Epic 6 (Caregiver):** Important but not critical - can launch with patient+doctor only, add caregivers to increase retention

**Epic 7 (Proactive Interventions):** Requires Epic 4 + 4 weeks of patient data - natural deferral, enables "before/after" marketing

**Epic 9 (Health Records):** Major value prop but can be added post-launch to drive re-engagement and increase stickiness

**Epic 10 (Diet Module):** Critical for long-term T2D management but simplified meal logging in MVP is acceptable initially

**Epic 11 (Prescription OCR):** Streamlines data entry but manual entry is workable for MVP

**Epic 12 (Advanced Integrations):** Can be added incrementally - Bluetooth devices, EMR, global packs enhance but not block MVP

**Epic 13 (Appointments):** Manual appointment scheduling acceptable for MVP, automation adds convenience

---

## Core Value Propositions by Phase

### MVP Core Value Props

1. ✅ **AI-powered personalized guidance** (Epic 4, 8) - "Rice spikes your glucose to 180, roti keeps you at 140"
2. ✅ **Doctor-connected care** (Epic 5) - Not AI-only, doctor oversight and guidance
3. ✅ **Emergency detection and safety** (Epic 3) - Immediate critical alerts with instructions
4. ✅ **Medication adherence improvement** (Epic 2) - Smart reminders with 85%+ adherence target
5. ✅ **Offline-first reliability** (Epic 1) - Works without internet, 95%+ patients <24hr stale data
6. ✅ **Dual interface flexibility** (Epic 1) - Native app (primary) + WhatsApp (secondary)

### Phase 2 Enhanced Value Props

1. ⏭️ **Comprehensive health records management** (Epic 9) - Centralized storage, auto-populate EMR
2. ⏭️ **Integrated diet/lifestyle management** (Epic 10) - Food swaps, calorie tracking, specific diets
3. ⏭️ **Family/caregiver involvement** (Epic 6) - 2.3x adherence improvement with caregiver support
4. ⏭️ **Advanced device integrations** (Epic 12) - Bluetooth glucometers, CGM, health apps
5. ⏭️ **Proactive pattern-based interventions** (Epic 7) - Pre-meal warnings prevent problems
6. ⏭️ **Global scalability** (Epic 12) - Location packs for any country

---

## Timeline & Resource Considerations

### MVP Timeline (20-24 weeks)

**Weeks 1-4:** Epic 1 (Foundation, Infrastructure, Mobile App, WhatsApp, Database)
**Weeks 5-8:** Epic 2 (Medication Management) + Epic 3 (Agent Orchestration & Emergency)
**Weeks 9-12:** Epic 4 (Pattern Detection) - requires 4 weeks of data collection to validate
**Weeks 13-16:** Epic 5 (Doctor Dashboard)
**Weeks 17-20:** Epic 8 (AI Q&A, Care Coordinator)
**Weeks 21-24:** Integration, testing, bug fixes, MVP validation pilot

### Why 20-24 Weeks Instead of 12 Weeks?

**Original Estimate:** 12 weeks for 13 epics (~70+ stories)
**Realistic Estimate:** 2-3 weeks per epic for complex features
**Calculation:** 6 epics × 3.5 weeks average = 21 weeks + 3 weeks buffer = 24 weeks

**Factors:**
- Native mobile app development (iOS + Android) - complex
- 7-agent orchestration architecture - complex
- Offline-first sync engine - complex
- WhatsApp Business API integration + approval (2-4 weeks)
- Pattern detection algorithms requiring validation
- Healthcare compliance and testing requirements

### Phase 2 Timeline (8-12 weeks)

Post-MVP launch, add remaining 7 epics incrementally based on customer feedback and priorities.

---

## Success Criteria for MVP Launch

### Minimum Launch Requirements

✅ **Functional:**
- Native mobile app live on iOS App Store + Google Play Store
- WhatsApp Business API approved and integrated
- Glucose logging (text, voice, photo) working
- Medication reminders with escalation working
- Emergency detection (hypoglycemia/hyperglycemia) working
- Doctor dashboard with tiered patient view working
- AI question answering (80%+ routine questions) working
- Offline-first functionality validated

✅ **Technical:**
- 98%+ emergency detection accuracy (NFR4)
- <2 second AI response time (NFR2)
- <500ms dashboard load time (NFR2)
- 95%+ patients <24hr stale data offline (NFR1)
- Security: AES-256 encryption, HIPAA-equivalent compliance

✅ **Validation:**
- 50-patient pilot completed (Story 1.15)
- Baseline metrics established
- AI accuracy validated (80%+ for routine questions)
- Pattern detection validated (90%+ patients have patterns within 4 weeks)

### Phase 2 Trigger Criteria

Launch Phase 2 when:
- MVP achieves 75%+ Week 1 retention (NFR14)
- MVP achieves 85%+ medication adherence (NFR15)
- 500+ active patients using MVP
- Customer feedback prioritizes specific Phase 2 features

---

## Document Control

**Version:** 1.0
**Created:** 2025-01-14
**Author:** Sarah (Product Owner)
**Status:** Final
**Related Documents:**
- [Goals and Background Context](./goals-and-background-context.md)
- [Requirements](./requirements.md)
- [Epic List](./epic-list.md)
- [Checklist Results Report](./checklist-results-report.md)
