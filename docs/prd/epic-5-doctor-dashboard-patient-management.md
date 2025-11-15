# Epic 5: Doctor Dashboard & Patient Management

## Epic Goal

Build a web-based doctor dashboard that enables doctors to efficiently manage their patients, review patient data, and provide guidance without adding significant time burden. Implement tiered patient view (Urgent/Attention/Stable) for quick triage, patient onboarding interface for setting expectations (target HbA1c, medication schedule, lifestyle goals), alert management system with context and suggested actions, and comprehensive patient detail views showing glucose trends, medication adherence, patterns, and alerts. The dashboard saves doctors 10-15 minutes daily through batch review (Monday morning weekly summaries) and enables the doctor-initiated engagement model where doctors set expectations and ProCare creates personalized programs. The dashboard is desktop-first, accessible via web browsers without requiring mobile app download.

## Story 5.1: Doctor Self-Registration & Dashboard Foundation

As a doctor,
I want to self-register and set myself up with minimal involvement from ProCare,
so that I can start using the system directly without waiting for manual setup.

**Acceptance Criteria:**
1. Doctor self-registration: doctors can self-register via web form or mobile app with credentials (name, email, phone, medical license number, specialty)
2. Self-setup process: doctor can log on, understand how to activate patients, get business model, see examples, and start using system directly
3. Doctor authentication: OAuth 2.0 or JWT-based authentication for secure login
4. Login page: simple, clean login interface accessible via web browser and mobile app
5. Session management: secure session handling with automatic timeout after inactivity
6. Doctor profile: doctors can view and edit their profile (name, contact info, preferences)
7. Dashboard landing page: after login, doctors see main dashboard with patient overview
8. Responsive design: dashboard works on desktop browsers (Chrome 90+, Safari 14+, Firefox 88+) and mobile app
9. Error handling: clear error messages for login failures, session expiration, access denied
10. Security: password requirements, account lockout after failed attempts, secure password reset
11. Doctor data storage: doctor profiles stored in PostgreSQL database with encrypted sensitive data
12. Minimal ProCare involvement: system designed for self-service setup with minimal support needed

## Story 5.2: Patient Search & Tiered Patient View

As a doctor,
I want to search for a specific patient and see my patients organized by priority,
so that I can quickly find and identify who needs immediate attention.

**Acceptance Criteria:**
1. Patient search: doctor can search for specific patient by ID or phone number to access their profile
2. Search interface: search bar prominently displayed in dashboard header
3. Search results: shows matching patients with name, ID, phone number, last activity
4. Quick access: clicking search result takes doctor directly to patient profile
5. Patient tiering logic: patients automatically categorized into Urgent, Attention, Stable, and "Call in for appointment" buckets
6. Daily patient buckets: System updates daily (not weekly aggregate) with most important bucket being "Call in for appointment" with reason for doctor's judgment call
7. Urgent tier: patients with emergencies (hypoglycemia/hyperglycemia from Epic 3), critical alerts, or urgent medication issues
8. Attention tier: patients with missed medications, high glucose trends, new patterns detected, or questions requiring response
9. Stable tier: patients with good glucose control, consistent medication adherence, no alerts
10. "Call in for appointment" bucket: patients with dramatically wrong situations that need in-person visit, with reason displayed. This bucket is only visible to registered diabetes specialists.
11. Tiered view interface: dashboard shows tabs or sections (Urgent, Attention, Stable, Call in for appointment - only visible to diabetes specialists)
12. Patient list in each tier: shows patient name, last glucose reading, adherence percentage, alert count, last activity, reason for bucket assignment
13. Patient count badges: shows number of patients in each tier (e.g., "Urgent (3)", "Attention (12)", "Stable (45)", "Call in (2)")
14. Auto-refresh: patient tiers update automatically as new data arrives (daily updates, not weekly)
15. Tier filtering: doctors can filter patients within each tier by name, date, or alert type
16. Tier sorting: patients within each tier sorted by priority (most urgent first) or alphabetically

## Story 5.3: Patient Detail View with Health Record Trends & Decision Support

As a doctor,
I want to see comprehensive patient information including health record trends and decision support,
so that I can quickly understand their status and get guidance on what to look at and possible solutions.

**Acceptance Criteria:**
1. Patient detail page: accessible by clicking patient name from tiered view or search results
2. Patient overview: shows patient name, age, diabetes type, diagnosis date, doctor connection date
3. Health record trends: All types of health markers viewable as trends (based on various health records uploaded), searchable and bucketed under headers like Liver, Kidney, Heart, etc.
4. Health marker organization: markers organized by category (Liver, Kidney, Heart, Blood Sugar, etc.) for easy navigation
5. Trend visualization: each marker shows trend over time (line chart or table) with ability to view different time periods
6. Glucose trends: displays glucose readings over time (last 7 days, 30 days) as simple line chart or table
7. Medication adherence: shows adherence percentage (daily, weekly, monthly) with trend indicator (shown as trend, not on every miss/delay)
8. Diet trends and patterns: shows patient's diet patterns, food-glucose correlations, meal logging frequency
9. Exercise data: shows patient's exercise/activity data and its impact on glucose
10. Last prescription: displays most recent prescription with extracted medication and diet information
11. Decision support bot: Provides advice on what to look at and what could be possible solutions - important for doctors who don't understand what all can be done for patient in various situations
12. Decision support context: bot analyzes patient's health record trends, glucose patterns, medication adherence, diet, exercise to provide recommendations
13. Recent alerts: displays recent alerts (emergencies, missed medications, pattern changes) with timestamps
14. Detected patterns: shows patient's food-glucose, activity-impact, medication-consistency patterns (from Epic 4)
15. Recent activity: shows last glucose log, meal log, medication log with timestamps
16. Caregiver status: shows if patient has caregiver linked, caregiver contact info (if permitted)
17. Patient summary: quick summary card showing key metrics (average glucose, adherence, alert count) with summarised data from system
18. Navigation: easy navigation back to tiered view, to other patients, or to patient messaging

## Story 5.4: Doctor-Initiated Patient Onboarding with QR Code

As a doctor,
I want to initiate patient onboarding by providing a personalized QR code,
so that patients can easily join ProCare and most data comes from prescription upload.

**Acceptance Criteria:**
1. Doctor onboarding interface: Doctor initiates onboarding by saying "ProCare will take care of you" to patient
2. Personalized QR code generation: System generates unique QR code for each doctor that patients can scan
3. QR code display: Doctor can view and print/download their personalized QR code from dashboard or mobile app
4. QR code scanning: Patient scans QR code via mobile app to start onboarding process
5. Doctor specialty detection: System determines doctor's specialty during registration (diabetes specialist, heart specialist, liver specialist, kidney specialist, or other)
6. Condition determination: During patient onboarding, system determines what condition doctor is treating patient for (diabetes, hypertension, kidney, liver, or other)
7. Doctor connection logic:
   - If doctor is diabetes specialist AND treating for diabetes: Patient connected to doctor as diabetes specialist, full doctor-patient relationship established
   - If doctor is non-diabetes specialist OR treating for other condition: Doctor becomes referring doctor only, patient onboarded without diabetes specialist connection, 1-month trial activated
8. Minimal doctor input: Doctor will NOT do detailed setup in beginning - will at best say 'scan my personalized QR code' and get onto ProCare
9. Prescription-based data: Remaining data will mostly come from prescription upload (some doctors might provide details, but most won't)
10. Onboarding flow: Patient completes onboarding via app or WhatsApp after scanning QR code (phone number, doctor connection via QR code, condition being treated, management mode selection, option to upload health records)
11. Prescription upload: Patient uploads prescription during or after onboarding, system extracts medication info, diet type, and other data. If referring doctor - they need to upload their diabetes prescriptions. Not just the prescriptions of the doctor who has referred them.
12. Onboarding completion: Patient receives confirmation via app or WhatsApp:
    - If diabetes specialist: "Welcome to ProCare! Dr. [Name] has connected you. Upload your prescription to get started."
    - If referring doctor: "Welcome to ProCare! Dr. [Name] referred you. You have 1 month free trial. Upload your prescription to get started."
13. Onboarding data storage: Patient connection to doctor stored in database, linked via QR code, doctor type (diabetes specialist vs referring doctor) and treating condition stored
14. QR code accessibility: QR code accessible on app as well as WhatsApp (doctor can share via WhatsApp)
15. Standard test prescriptions: Doctor can prescribe standard tests on regular interval (not just default HbA1c test) - can be done via interface or prescription
16. Referring doctor information: System stores referring doctor information (name, phone number, treating condition) for reverse acquisition purposes

## Story 5.5: Diabetes Specialist Connection to Referred Patients

As a diabetes specialist,
I want to connect to patients who were referred by other doctors,
so that I can manage their diabetes care through ProCare.

**Acceptance Criteria:**
1. Patient connection: During onboarding - The patient will anyways be connected to the doctor using the QR code, which will be specific to the doctor. In which case the notification to connect with the patient is not needed.
2. Patient search: Doctor can search for patients by phone number, name, or patient ID
3. Connection establishment: When patient scans QR code from diabetes specialist, diabetes specialist becomes their connected doctor
4. Doctor dashboard update: Both doctors see patient in their dashboard (referring doctor sees as "referred patient", diabetes specialist sees as "connected patient")
5. Full access: Once diabetes specialist connected, patient gains full access to doctor features (pooled questions, alerts, etc.)
6. Connection history: System tracks connection history (referring doctor → patient → diabetes specialist)
7. Revenue share calculation: System calculates revenue share based on connection date and subscription payments. Revenue shares are fixed percentages as stated in FR35 and Story 8.12 - there is no configurable revenue sharing.
8. Patient data access: Diabetes specialist gains full access to patient's ProCare data (glucose logs, medications, patterns, health records)

## Story 5.6: Alert Management & Appointment Scheduling

As a doctor,
I want to manage patient alerts and schedule appointments,
so that I can bring patients back for meetings when needed without changing medicines online.

**Acceptance Criteria:**
1. Alert display: alerts shown in patient detail view and tiered view with alert type, timestamp, priority
2. Alert context: each alert includes relevant context (patient's recent glucose history, medication timing, meal timing, caregiver status)
3. Alert management: Cannot ask doctor to change medicines online - most alerts bring patient back for another meeting
4. Dramatically wrong situations: If something dramatically wrong, patient goes to "Schedule appointment" bucket with reason
5. Appointment scheduling: Doctor can request additional activity before appointment (e.g., "Come in on Nov 20th, get HbA1c check done before coming")
6. Patient acceptance: Patient accepts appointment request, and formal appointment is setup by doctor/clinic at the time
7. Appointment calendar: System has calendar to view all appointments scheduled on system
8. Ancillary approach: If doctor doesn't adopt ProCare completely, ancillary approach available (email to assistant to call/confirm and block appointment time, WhatsApp message to assistant to book appointment for xyz at abc time)
9. Alert types: emergency alerts (from Epic 3), medication alerts (from Epic 2), pattern alerts (from Epic 4), question alerts
10. Alert prioritization: alerts sorted by priority (Urgent > Attention > Normal) and timestamp
11. Alert filtering: doctors can filter alerts by type, date, priority, or patient
12. Alert actions: doctors can acknowledge alerts, mark as resolved, or schedule appointment
13. Alert history: doctors can view alert history for each patient (resolved and active alerts)
14. Alert reporting: alert summary included in daily patient summary

## Story 5.7: Daily Patient Summary with Appointment Bucket

As a doctor,
I want daily summaries of my patients with focus on appointment scheduling,
so that I can efficiently identify patients who need to come in for appointments.

**Acceptance Criteria:**
1. Daily summary generation: System generates daily patient summaries (not weekly aggregate) - updates changing on daily basis
2. Most important bucket: "Call in for appointment" bucket is most important, with reason why for doctor to take judgment call
3. Summary content: includes glucose trends, medication adherence, detected patterns, alerts, key insights, intervention effectiveness
4. Summary format: concise format displayed in doctor dashboard: "Patient [Name] - Daily Summary: Glucose avg 128 mg/dL (stable), Adherence 95% (excellent), Patterns: Rice spikes glucose, roti stable. Alerts: 0. Status: Stable. Action: [Reason for appointment if applicable]"
5. Summary batch view: doctors can view all patient summaries in batch (all patients at once)
6. Summary metrics: summaries include key metrics (average glucose, adherence, time in range, pattern changes)
7. Summary insights: summaries include Health Insights agent-generated insights (from Epic 4)
8. Summary alerts: summaries highlight patients needing attention based on daily data
9. Appointment bucket: Patients in "Call in for appointment" bucket shown prominently with reason
10. Summary export: doctors can export daily summaries as PDF or CSV for records
11. Summary timing: summaries generated daily, available for doctor's review
12. Summary history: doctors can view past daily summaries for trend analysis

## Story 5.8: Doctor Communication via ProCare Relay (No Direct Conversation)

As a doctor,
I want to tell ProCare what to tell my patients,
so that ProCare relays the message without me conversing directly with patients.

**Acceptance Criteria:**
1. Doctor communication interface: doctor can compose messages for ProCare to relay to patients via dashboard or mobile app
2. Message relay: Doctors tell ProCare what to tell patient, and ProCare relays message to patient (no direct doctor-patient conversation)
3. Message types: doctors can send text messages, medication reminders, appointment reminders, general guidance, instructions
4. Message delivery: messages relayed to patient via app or WhatsApp (integrated with WhatsApp API and app push notifications)
5. Message personalization: ProCare can personalize doctor's message with patient context when appropriate
6. Message history: doctors can view message history with each patient (what ProCare relayed on doctor's behalf)
7. Patient responses: patient responses come back to ProCare, doctor can see responses in dashboard
8. Message templates: doctors can save and reuse message templates for common scenarios
9. Message scheduling: doctors can schedule messages to be relayed at specific times
10. Message status: doctors can see if message was delivered and read by patient
11. Message context: messages include patient context (recent glucose, medications, patterns) for doctor reference
12. No direct conversation: Doctors will NOT converse directly with patient - all communication goes through ProCare relay

## Story 5.9: Doctor Dashboard Performance & Usability

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
