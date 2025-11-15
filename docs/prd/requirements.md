# Requirements

## Functional

FR1: The system shall provide a native mobile app as the primary interface for all patient interactions including glucose logging, meal logging, medication reminders, and question answering. The app shall support flexible UX (chat-based primary interface with direct input options like tags for quick data entry) while maintaining conversational capabilities. WhatsApp shall serve as a secondary channel for re-engagement, templated interactions, summaries, and basic data collection when the app is not being used.

FR2: The system shall support multiple glucose logging input methods: text input (e.g., "128"), voice message (Hindi/English), and photo OCR for glucometer screens.

FR3: The system shall provide smart medication reminders at doctor-specified times with escalation logic: reminder → nudge → caregiver notification → Stored as an input for the doctor to view at the time of visit.

FR4: The system shall support simplified meal logging via photo upload and text description (no detailed nutrition calculations), with basic food recognition (20-30 items offline, full recognition online).

FR5: The system shall detect food-glucose correlations, activity impacts, and medication consistency patterns after 4 weeks of data collection.

FR6: The system shall provide proactive pattern-based interventions after pattern detection, including pre-meal warnings (e.g., "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?").

FR7: The system shall detect emergency glucose levels (around hypoglycemia, hyperglycemia as per the health standards of the country) and provide immediate instructions with escalation to the caregiver & logging it in the doctors dashboard.

FR8: The system shall answer routine patient questions via AI (nutrition, timing, general diabetes management) and escalate medication/medical questions to doctors.

FR9: The system shall generate weekly progress reports for patients (wins, opportunities, insights) and doctors (status, metrics, flags).

FR10: The system shall manage blood test appointments with 2-week reminders & set them up with procare partners, and get the result upload/analysis via OCR & direct upload to the Procare system through APIs.

FR11: The system shall provide a doctor web dashboard with tiered patient view (Urgent/Attention/Stable), weekly summaries, and alert system with context and suggested actions.

FR12: The system shall provide a separate caregiver dashboard (mobile app + WhatsApp + web) with real-time critical alerts, daily digest, and caregiver-primary management option.

FR13: The system shall support doctor-initiated engagement where doctors initiate patient onboarding by entering their number into a system / sending a templated whatsapp to the procare number. They will also set expectations (target HbA1c, medication schedule, lifestyle goals), and ProCare creates personalized programs.

FR14: The system shall implement a 7-agent orchestration system: Engagement Engine (orchestrator), Clinical Monitor, Lifestyle Coach, Health Insights, Doctor Bridge, Care Coordinator, and Learning Library.

FR15: The system shall bundle multiple agent responses into single patient messages and enforce rate limiting (max 5 notifications/day, 2-hour minimum spacing).

FR16: The system shall work fully offline for core features (glucose logging, meal logging, medication reminders) with local database caching 90 days of data.

FR17: The system shall sync offline data when connectivity is restored using event-sourcing model with conflict resolution (device timestamp as source of truth).

FR18: The system shall support patient signup with management mode selection: patient-primary, shared management, or caregiver-primary.

FR19: The system shall deliver context-based health education tips (not comprehensive libraries) triggered by patient behavior patterns.

FR20: The system shall track medication adherence and provide adherence metrics to doctors and caregivers.

FR21: The system shall provide comprehensive health records management enabling patients to upload, store, and query all health records (blood tests, lab reports, medical documents) in a centralized location. Health records shall be automatically available to doctors during every patient meeting and automatically populate doctor EMR systems, eliminating manual data entry.

FR22: The system shall provide a comprehensive diet/lifestyle module that manages calories per day, recommends food swaps, supports specific diet types (low sodium, low potassium, low fat, and more as recommended by doctor), and provides menu recommendations from photos when eating out. This module is critical as 95% of Type 2 diabetes management is diet management.

FR23: The system shall enable patients to upload prescriptions (photos or documents) from which the system extracts medication information, diet type, and other relevant data, reducing manual data entry and ensuring accuracy.

FR24: The system shall support doctor interaction model where doctors provide generic answers to pooled questions (AI pools similar questions from multiple patients), with answers relayed to patients with the doctor's name. Doctors do not answer questions one-to-one but provide batch responses to common questions.

FR25: The system shall support global/regional deployment with regionality support (time zones, local protocols), multi-language support (beyond Hindi/English), and region-specific medical protocols and guidelines.

FR26: The system shall integrate with data collection devices (Bluetooth glucometers, CGM devices, Google Fit, Apple Health), EMR systems, Hospital Management Systems (HMS), payment gateways (payment flow through ProCare), and service providers (medicine providers, test booking agencies like Orange Health).

FR27: The system shall enable patients to view their profile, download summarized reports, and download any data from their history. Doctors shall have a consolidated view of data collected from various patients with drill-down capabilities.

FR28: The system shall provide caregiver access via the mobile app (in addition to web dashboard and WhatsApp), enabling caregivers to monitor and assist with patient management.

FR29: The system shall support doctor-initiated onboarding where doctors onboard patients by saying "ProCare will take care of you," after which patients complete onboarding via app or WhatsApp with the same flow but different interfaces.

FR30: The system shall provide offline functionality primarily through the mobile app (offline data storage, sync when connectivity restored), with WhatsApp remaining conversational-only and requiring connectivity.

FR31: The system shall support referring doctors (non-diabetes specialists such as heart, liver, or kidney specialists) who can onboard patients via QR code. The system shall determine during onboarding whether the doctor is treating the patient for diabetes or another condition. If diabetes, the patient is connected to the doctor. If not, the doctor becomes a referring doctor only, and the patient is onboarded without a diabetes specialist connection.

FR32: The system shall provide a 1-month free trial period for patients referred by non-diabetes specialists. After the trial period, patients must subscribe to maintain full access. Patients who do not subscribe shall lose access to: uploaded health data, prescriptions, online interface for showing data to doctors, diet module recipes and menu scan features, and caregiver module access.

FR33: The system shall enforce enhanced AI guidelines for patients with or without diabetes specialists, ensuring AI responses strictly adhere to medical guidelines, avoid hallucinations, and stay within safe boundaries.

FR34: The system shall support patients without diabetes specialists, where questions cannot be escalated to doctors until a diabetes specialist connects to the patient. The system shall track referring doctor information (name, number, treating condition) for reverse acquisition purposes.

FR35: The system shall implement specific revenue share percentages for referring doctors: 30%, and for diabetes specialists at 60%. In case a patient has been referred by a non diabetes specialist and then a diabetes specialist attaches to the patient, the doctor gets 30% of the revenue share of that patient but 60% of all other patients that they refer. In case no diabetes specialist latches onto the patient, only 30% will be paid out to the referring doctor.

FR36: The system shall support reverse acquisition of diabetes specialists by collecting data about doctors viewing patient data (name, number, treating condition through a simple form) and enabling ProCare to contact these doctors offline or via WhatsApp to demonstrate value and encourage platform adoption.

## Non Functional

NFR1: The system shall achieve 95%+ of patients having <24hr stale data when offline functionality is used.

NFR2: The system shall respond to AI questions within 2 seconds via WhatsApp and load doctor dashboard pages within 500ms.

NFR3: The system shall handle 1,000+ concurrent users with the 7-agent orchestration system.

NFR4: The system shall achieve 98%+ true positive rate for emergency detection with <2% false alarm rate.

NFR5: The system shall achieve 95%+ glucose prediction accuracy within 15 mg/dL for pattern-based interventions.

NFR6: The system shall achieve 80%+ AI question answering accuracy for routine questions (nutrition, timing, general).

NFR7: The system shall encrypt all patient health data with AES-256 at rest and TLS 1.3 in transit.

NFR8: The system shall comply with India data protection laws, HIPAA-equivalent PHI handling, audit logs, and access controls.

NFR9: The system shall support WhatsApp Business API rate limits and handle 10K+ users without degradation.

NFR10: The system shall support multiple Indian languages for MVP including Hindi, English, Tamil, Telugu, and other regional languages as needed. The system should be ready to manage multiple global languages including the interface changes of right to left and beyond.

NFR11: The system shall maintain 90-day full data cache locally (~260MB) with 1-year summary and >1 year stats only.

NFR12: The system shall implement bandwidth-aware sync: WiFi (full), 4G (compressed), 2G/3G (critical only).

NFR13: The system shall provide doctor dashboard accessible via web browsers (Chrome 90+, Safari 14+, Firefox 88+) without requiring mobile app download.

NFR14: The system shall achieve 75%+ Week 1 retention through exceptional first-week experience (2+ personalized insights, quick answers, doctor connection).

NFR15: The system shall achieve 85%+ medication adherence (vs 50% baseline) through smart reminders and caregiver involvement.

NFR16: The system shall reduce emergency calls to doctors by 60% (from 3-5/week to 1-2/week per doctor).

NFR17: The system shall enable doctors to review patient status in 10-15 minutes daily maximum (10 minutes Monday morning, 5 minutes during week).

NFR18: The system shall achieve 2-3 minute onboarding time with doctor-initiated onboarding flow (doctor says "ProCare will take care of you," then patient completes onboarding via app or WhatsApp).

NFR19: The system shall detect patterns for 90%+ of patients within 4 weeks of data collection.

NFR20: The system shall achieve 80%+ doctor dashboard weekly usage and 85%+ caregiver dashboard daily usage.

NFR21: The system shall achieve 95%+ AI response accuracy for patients without diabetes specialists, with strict adherence to medical guidelines and zero tolerance for hallucinations or out-of-bounds responses.

NFR22: The system shall convert 30%+ of trial period patients to paid subscribers within the first month.
