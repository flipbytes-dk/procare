# Epic 6: Caregiver Features & Alerts

## Epic Goal

Implement comprehensive caregiver features that enable family members to monitor and support elderly diabetic patients, reducing caregiver anxiety while improving patient outcomes. Build separate caregiver dashboard (web + WhatsApp) with real-time critical alerts for hypoglycemia and missed medications, daily digest summaries showing patient status, and caregiver-primary management mode option where caregivers manage the patient's ProCare account. This epic addresses the unique India market need for family involvement in healthcare, delivering 2.3x medication adherence improvement (validated by WellDoc research) and providing peace of mind to working professionals managing elderly parents. The caregiver experience is designed to reduce anxiety, not amplify it, with setup primarily on the caregiver's device rather than requiring elderly parent to use technology.

## Story 6.1: Caregiver Registration & Patient Linking

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

## Story 6.2: Caregiver Web Dashboard - Patient Status Overview

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

## Story 6.3: Real-Time Critical Alerts via WhatsApp

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

## Story 6.4: Daily Digest Summary via WhatsApp

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

## Story 6.5: Caregiver Dashboard - Patient Detail View

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

## Story 6.6: Caregiver-Primary Management Mode

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

## Story 6.7: Caregiver-Patient Messaging via Dashboard

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

## Story 6.8: Caregiver Dashboard Performance & Anxiety Reduction

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
