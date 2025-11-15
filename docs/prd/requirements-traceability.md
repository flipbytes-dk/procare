# Requirements Traceability Matrix

This document maps each Functional Requirement (FR) and Non-Functional Requirement (NFR) to the implementing Epic(s) and Story(ies), ensuring complete coverage and traceability from requirements to implementation.

---

## Functional Requirements Traceability

| Req ID | Requirement Summary | Implementing Epic(s) | Implementing Story(ies) | Status |
|--------|-------------------|---------------------|------------------------|--------|
| **FR1** | Native mobile app (primary) + WhatsApp (secondary) for patient interactions | Epic 1 | Story 1.3 (Native Mobile App), Story 1.3a (WhatsApp Integration) | ✅ Covered |
| **FR2** | Multiple glucose logging input methods (text, voice, photo OCR) | Epic 1 | Story 1.5 (Text), Story 1.5a (App Text with Tags), Story 1.6 (Voice), Story 1.7 (Photo OCR) | ✅ Covered |
| **FR3** | Smart medication reminders with escalation logic | Epic 2 | Story 2.3 (Reminders), Story 2.5 (Nudges), Story 2.6 (Caregiver Escalation), Story 2.7 (Adherence Trend) | ✅ Covered |
| **FR4** | Simplified meal logging via photo + text | Epic 1 | Story 1.8 (WhatsApp Meal Logging), Story 1.8a (Web Chat Meal Logging) | ✅ Covered |
| **FR5** | Pattern detection (food-glucose, activity, medication consistency) after 4 weeks | Epic 4 | Story 4.2 (Food-Glucose), Story 4.3 (Activity Impact), Story 4.4 (Medication Consistency), Story 4.5 (4-Week Detection) | ✅ Covered |
| **FR6** | Proactive pattern-based interventions (pre-meal warnings) | Epic 7 | Story 7.1 (Pre-Meal Timing), Story 7.2 (Warning Generation), Story 7.3 (Effectiveness Tracking) | ✅ Covered |
| **FR7** | Emergency glucose detection with escalation to caregiver + doctor dashboard | Epic 3 | Story 3.3 (Emergency Detection), Story 3.4 (Emergency Response), Story 3.5 (Configurable Doctor Alerts) | ✅ Covered |
| **FR8** | AI question answering (routine) with escalation to doctors (medical) | Epic 8 | Story 8.4 (Question Classification), Story 8.5 (AI Answering), Story 8.6 (Pooled Questions) | ✅ Covered |
| **FR9** | Weekly progress reports for patients and doctors | Epic 8 | Story 8.7 (Patient Reports), Story 8.8 (Doctor Reports) | ✅ Covered |
| **FR10** | Blood test appointment management with 2-week reminders | Epic 8 | Story 8.1 (Standard Test Scheduling), Story 8.2 (HbA1c Prediction), Story 8.3 (Lab Result Upload) | ✅ Covered |
| **FR11** | Doctor web dashboard with tiered patient view and alerts | Epic 5 | Story 5.1 (Dashboard Foundation), Story 5.2 (Tiered View), Story 5.3 (Patient Detail), Story 5.6 (Alert Management) | ✅ Covered |
| **FR12** | Caregiver dashboard (mobile app + WhatsApp + web) with alerts and digest | Epic 6 | Story 6.1 (Registration), Story 6.2 (Web Dashboard), Story 6.3 (WhatsApp Alerts), Story 6.4 (Daily Digest), Story 6.5 (Detail View) | ✅ Covered |
| **FR13** | Doctor-initiated engagement (onboarding, set expectations, personalized programs) | Epic 1, Epic 5 | Story 1.4 (Onboarding), Story 5.4 (Doctor-Initiated Onboarding with QR Code) | ✅ Covered |
| **FR14** | 7-agent orchestration system | Epic 3, Epic 4, Epic 7, Epic 8 | Story 3.1 (Message Bus), Story 3.2 (Engagement Engine), Story 3.3-3.8 (Clinical Monitor), Story 4.1-4.8 (Health Insights), Story 8.1-8.12 (Care Coordinator, Doctor Bridge, Learning Library) | ✅ Covered |
| **FR15** | Response bundling and rate limiting (max 5/day, 2-hour spacing) | Epic 3, Epic 7 | Story 3.6 (Response Synthesis), Story 7.4 (Notification Rate Limiting) | ✅ Covered |
| **FR16** | Offline-first functionality (glucose, meal, medication logging) | Epic 1 | Story 1.2 (Database Architecture), Story 1.11 (Sync Engine) | ✅ Covered |
| **FR17** | Offline data sync with conflict resolution (device timestamp as source of truth) | Epic 1 | Story 1.11 (Data Sync Engine) | ✅ Covered |
| **FR18** | Patient signup with management mode selection (patient/shared/caregiver-primary) | Epic 1 | Story 1.4 (Onboarding), Story 1.4a (Caregiver Enrollment), Story 1.4b (Self Enrollment) | ✅ Covered |
| **FR19** | Context-based health education tips triggered by patient behavior | Epic 8 | Story 8.10 (Learning Library - Context-Based Tips) | ✅ Covered |
| **FR20** | Medication adherence tracking with metrics to doctors/caregivers | Epic 2 | Story 2.7 (Adherence Trend), Story 2.8 (Adherence Tracking & Reporting) | ✅ Covered |
| **FR21** | Comprehensive health records management (upload, store, query, auto-populate EMR) | Epic 9 | Story 9.1 (Health Records Upload), Story 9.2 (Query/Search), Story 9.3 (Auto-Populate EMR) | ✅ Covered |
| **FR22** | Diet/lifestyle module (calories, food swaps, specific diets, menu recommendations) | Epic 10 | Story 10.1 (Calorie Management), Story 10.2 (Food Swaps), Story 10.3 (Specific Diets), Story 10.4 (Menu Recommendations) | ✅ Covered |
| **FR23** | Prescription upload with OCR extraction (medication, diet type, data) | Epic 11 | Story 11.1 (Prescription Upload & OCR), Story 11.2 (Data Extraction & Verification) | ✅ Covered |
| **FR24** | Pooled questions (AI pools similar questions, doctor provides batch responses) | Epic 8 | Story 8.6 (Pooled Questions for Doctors) | ✅ Covered |
| **FR25** | Global/regional deployment (time zones, languages, regional protocols) | Epic 12 | Story 12.5 (Global/Regional Support with Location Packs) | ✅ Covered |
| **FR26** | Device integrations (Bluetooth glucometers, CGM, Google Fit, Apple Health, EMR, HMS, payment, service providers) | Epic 12 | Story 12.1 (Bluetooth Devices), Story 12.2 (Google Fit/Apple Health), Story 12.3/1.14 (Payment Gateway), Story 12.4 (Service Providers) | ✅ Covered |
| **FR27** | Patient profile viewing, report downloads, consolidated doctor view | Epic 5, Epic 8 | Story 5.3 (Patient Detail View), Story 8.7 (Patient Reports), Story 8.8 (Doctor Reports) | ✅ Covered |
| **FR28** | Caregiver access via mobile app (in addition to web + WhatsApp) | Epic 6 | Story 6.1 (Registration), Story 6.8 (Performance & Anxiety Reduction) | ✅ Covered |
| **FR29** | Doctor-initiated onboarding (same flow, different interfaces for app vs WhatsApp) | Epic 1 | Story 1.4 (Doctor-Initiated Patient Onboarding & Authentication) | ✅ Covered |
| **FR30** | Offline functionality in mobile app, WhatsApp conversational-only requiring connectivity | Epic 1 | Story 1.2 (Database Architecture), Story 1.3 (Native Mobile App), Story 1.3a (WhatsApp) | ✅ Covered |
| **FR31** | Referring doctor workflow (non-diabetes specialists onboard patients, determine condition during onboarding) | Epic 1, Epic 5 | Story 1.4 (Onboarding with Doctor Type Determination), Story 5.4 (Doctor-Initiated Onboarding with QR Code) | ✅ Covered |
| **FR32** | 1-month free trial for referred patients, subscription restrictions after trial | Epic 1 | Story 1.12 (Trial Period Management), Story 1.13 (Subscription Restrictions After Trial) | ✅ Covered |
| **FR33** | Enhanced AI guidelines for patients with/without diabetes specialists (strict adherence to medical guidelines, no hallucinations) | Epic 8 | Story 8.5 (AI Question Answering with Enhanced Guidelines - AC#11) | ✅ Covered |
| **FR34** | Support patients without diabetes specialists (questions cannot be escalated until specialist connects, track referring doctor info) | Epic 5, Epic 8 | Story 5.4 (Referring Doctor Info), Story 5.5 (Diabetes Specialist Connection), Story 8.5 (AI Answering) | ✅ Covered |
| **FR35** | Revenue share percentages (30% referring doctor, 60% diabetes specialist) | Epic 1, Epic 5 | Story 1.14 (Payment Gateway - AC#11 Revenue Sharing Calculation), Story 5.5 (Revenue Share Calculation - AC#7) | ✅ Covered |
| **FR36** | Reverse acquisition tracking (collect doctor data viewing patient info for offline outreach) | Epic 5 | Story 5.4 (Referring Doctor Information - AC#16) | ⚠️ Partially Covered (tracking mentioned but no dedicated implementation story) |

---

## Non-Functional Requirements Traceability

| Req ID | Requirement Summary | Implementing Epic(s) | Validation Method | Status |
|--------|-------------------|---------------------|-------------------|--------|
| **NFR1** | 95%+ patients have <24hr stale data when offline | Epic 1 | Story 1.15 (MVP Validation Pilot - AC#11 Offline Sync Validation) | ✅ Covered |
| **NFR2** | <2 second AI response time, <500ms dashboard load time | Epic 5, Epic 8 | Story 1.15 (MVP Validation Pilot - AC#13 Performance Validation), Story 5.9 (Dashboard Performance - AC#1-2) | ✅ Covered |
| **NFR3** | Handle 1,000+ concurrent users with 7-agent orchestration | Epic 3 | Story 1.15 (MVP Validation Pilot - AC#10 Load Testing), Story 3.8 (Agent Health Monitoring) | ✅ Covered |
| **NFR4** | 98%+ emergency detection accuracy, <2% false alarm rate | Epic 3 | Story 1.15 (MVP Validation Pilot - AC#8 Emergency Detection Validation), Story 3.3 (Emergency Detection - AC#7-8) | ✅ Covered |
| **NFR5** | 95%+ glucose prediction accuracy within 15 mg/dL | Epic 4, Epic 8 | Story 1.15 (MVP Validation Pilot - AC#7 HbA1c Prediction Validation), Story 4.7 (Prediction Validation) | ✅ Covered |
| **NFR6** | 80%+ AI question answering accuracy for routine questions | Epic 8 | Story 1.15 (MVP Validation Pilot - AC#5 AI Accuracy Validation), Story 8.5 (AI Answering - AC#5) | ✅ Covered |
| **NFR7** | AES-256 encryption at rest, TLS 1.3 in transit | Epic 1 | Architecture: security-performance.md defines encryption standards | ✅ Covered |
| **NFR8** | India data protection compliance, HIPAA-equivalent PHI handling, audit logs, access controls | Epic 1 | Architecture: security-performance.md defines compliance requirements | ✅ Covered |
| **NFR9** | WhatsApp Business API rate limits, handle 10K+ users without degradation | Epic 1, Epic 3 | Story 1.3a (WhatsApp - AC#8 Rate Limit Compliance), Story 3.2 (Engagement Engine - AC#10 Rate Limiting) | ✅ Covered |
| **NFR10** | Multiple Indian languages for MVP (Hindi, English, Tamil, Telugu), global language readiness | Epic 12 | Story 12.5 (Global/Regional Support with Location Packs - AC#6-7) | ✅ Covered |
| **NFR11** | 90-day full data cache locally (~260MB), 1-year summary, >1 year stats only | Epic 1 | Story 1.2 (Database Architecture - AC#5), Story 4.1 (Health Insights Data Collection - AC#7) | ✅ Covered |
| **NFR12** | Bandwidth-aware sync: WiFi (full), 4G (compressed), 2G/3G (critical only) | Epic 1 | Story 1.11 (Data Sync Engine - AC#5 Bandwidth-Aware Sync Logic) | ✅ Covered |
| **NFR13** | Doctor dashboard accessible via web browsers (Chrome 90+, Safari 14+, Firefox 88+) | Epic 5 | Story 5.9 (Dashboard Performance - AC#5 Browser Compatibility) | ✅ Covered |
| **NFR14** | 75%+ Week 1 retention | Epic 1, Epic 4, Epic 8 | Story 1.15 (MVP Validation Pilot - AC#9 Retention Metric Validation) | ✅ Covered |
| **NFR15** | 85%+ medication adherence (vs 50% baseline) | Epic 2 | Story 1.15 (MVP Validation Pilot - AC#4 Baseline Measurement), Story 2.8 (Adherence Tracking) | ✅ Covered |
| **NFR16** | Reduce emergency calls to doctors by 60% (from 3-5/week to 1-2/week) | Epic 3, Epic 7 | Story 1.15 (MVP Validation Pilot - AC#4 Baseline Measurement) | ✅ Covered |
| **NFR17** | Enable doctors to review patient status in 10-15 minutes daily maximum | Epic 5 | Story 1.15 (MVP Validation Pilot - AC#4 Baseline Measurement), Story 5.7 (Daily Patient Summary), Story 5.9 (Dashboard Performance - AC#8) | ✅ Covered |
| **NFR18** | 2-3 minute onboarding time | Epic 1 | Story 1.4 (Doctor-Initiated Patient Onboarding - AC#10) | ✅ Covered |
| **NFR19** | 90%+ patients have detectable patterns within 4 weeks | Epic 4 | Story 1.15 (MVP Validation Pilot - AC#6 Pattern Detection Validation), Story 4.5 (Pattern Detection After 4 Weeks - AC#9) | ✅ Covered |
| **NFR20** | 80%+ doctor dashboard weekly usage, 85%+ caregiver dashboard daily usage | Epic 5, Epic 6 | Story 5.9 (Dashboard Performance - AC#10 Usage Tracking), Story 6.8 (Caregiver Dashboard Performance - AC#9) | ✅ Covered |
| **NFR21** | 95%+ AI response accuracy for patients without diabetes specialists (strict adherence, zero hallucinations) | Epic 8 | Story 8.5 (AI Question Answering - AC#11 Enhanced AI Guidelines) | ✅ Covered |
| **NFR22** | 30%+ trial period patients convert to paid subscribers within first month | Epic 1 | Story 1.12 (Trial Period Management - AC#9 Trial Analytics) | ✅ Covered |

---

## Coverage Summary

### Functional Requirements
- **Total FRs:** 36
- **Fully Covered:** 35 (97.2%)
- **Partially Covered:** 1 (2.8%) - FR36 (Reverse acquisition tracking needs dedicated implementation story)
- **Not Covered:** 0 (0%)

### Non-Functional Requirements
- **Total NFRs:** 22
- **Fully Covered:** 22 (100%)
- **Not Covered:** 0 (0%)

### Overall Traceability Score: **98.3%** (57/58 requirements fully covered)

---

## Gaps & Recommendations

### Gap: FR36 - Reverse Acquisition Tracking

**Current Coverage:** Partially covered - referring doctor information is tracked (Story 5.4 AC#16), but no dedicated story for displaying this data to ProCare team or automating outreach.

**Recommendation:** Add Story 5.10 to Epic 5:

**Story 5.10: Reverse Acquisition Dashboard**

As a ProCare business development team member,
I want to view diabetes specialists who are viewing patient data without being connected,
so that I can contact them offline to encourage platform adoption.

**Acceptance Criteria:**
1. Admin dashboard displays list of doctors viewing patient data who are not yet connected diabetes specialists
2. Doctor information collected via form when viewing patient data: name, phone number, treating condition
3. Dashboard shows: doctor name, phone number, treating condition, number of patients viewed, date last viewed
4. Export functionality: Export doctor list to CSV for offline outreach campaigns
5. Contact log: Track outreach attempts (WhatsApp message sent, call made, email sent) with status
6. Conversion tracking: Track when doctor signs up as connected diabetes specialist after outreach
7. Analytics: Measure reverse acquisition conversion rate (doctors contacted → diabetes specialists onboarded)

---

## Document Control

**Version:** 1.0
**Created:** 2025-01-14
**Author:** Sarah (Product Owner)
**Status:** Final
**Related Documents:**
- [Requirements](./requirements.md)
- [Epic List](./epic-list.md)
- [MVP Scope Matrix](./mvp-scope-matrix.md)
