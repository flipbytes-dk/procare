# Epic List

Epic 1: Foundation & Core Patient Logging - Establish project infrastructure, native mobile app development (iOS and Android), WhatsApp Business API integration (secondary channel), offline-first database architecture, and enable patients to log glucose readings and meals via mobile app (primary) or WhatsApp (secondary) with basic data storage and retrieval.

Epic 2: Medication Management & Reminders - Implement smart medication reminders with doctor-specified schedules, escalation logic (reminder → nudge → caregiver notification → stored for doctor review), and medication adherence tracking.

Epic 3: Agent Orchestration & Emergency Detection - Build the 7-agent orchestration system with Engagement Engine as orchestrator, implement Clinical Monitor agent for emergency detection (hypoglycemia/hyperglycemia), and establish agent communication via message bus.

Epic 4: Pattern Detection & Health Insights - Implement Health Insights agent for food-glucose correlation analysis, activity impact detection, and medication consistency patterns, generating personalized insights after 4 weeks of data collection.

Epic 5: Doctor Dashboard & Patient Management - Build web-based doctor dashboard with tiered patient view (Urgent/Attention/Stable), doctor-initiated patient onboarding (doctor says "ProCare will take care of you"), patient onboarding interface for setting expectations, alert management with context, patient detail views with health records access, and pooled questions interface for batch doctor responses.

Epic 6: Caregiver Features & Alerts - Implement separate caregiver dashboard (mobile app + web + WhatsApp), real-time critical alerts, daily digest summaries, and caregiver-primary management mode option.

Epic 7: Proactive Pattern-Based Interventions - Enable proactive interventions after pattern detection, including pre-meal warnings based on learned patterns, and implement notification bundling with rate limiting (max 5/day, 2-hour spacing).

Epic 8: Advanced Coordination & Question Answering - Implement Care Coordinator agent for blood test scheduling with predictions, Doctor Bridge agent for AI question answering with escalation and pooled questions (AI pools similar questions, doctor provides generic answers relayed with doctor's name), weekly progress reports for patients and doctors, and Learning Library agent for context-based education.

Epic 9: Health Records Management - Implement comprehensive health records management system enabling patients to upload, store, query, and download all health records (blood tests, lab reports, medical documents) in a centralized location. Health records automatically available to doctors during every patient meeting and automatically populate doctor EMR systems, eliminating manual data entry.

Epic 10: Diet/Lifestyle Module - Implement comprehensive diet/lifestyle management module that manages calories per day, recommends food swaps, supports specific diet types (low sodium, low potassium, low fat, and more as recommended by doctor), and provides menu recommendations from photos when eating out. This module is critical as 95% of Type 2 diabetes management is diet management.

Epic 11: Prescription Upload & Data Extraction - Enable patients to upload prescriptions (photos or documents) from which the system extracts medication information, diet type, and other relevant data, reducing manual data entry and ensuring accuracy.

Epic 12: Integrations & Global Support - Integrate with data collection devices (Bluetooth glucometers, CGM devices, Google Fit, Apple Health), EMR systems, Hospital Management Systems (HMS), payment gateways (payment flow through ProCare), and service providers (medicine providers, test booking agencies like Orange Health). Support global/regional deployment with regionality support (time zones, local protocols), multi-language support, and region-specific medical protocols and guidelines.

Epic 13: Appointment Booking & Slot Reservation - Implement booking and reserving a slot for returning patients based on issues seen with the patient and a detailed description of why ProCare is recommending to call the patient. Post that - from the system itself, the doctor can press a button and a message (with approval from the doctor) will go to the WhatsApp / app of the patient - and based on payment, the appointment will be scheduled. ProCare will take 10% of the payment done to secure the patient immediately and ahead of time.
