# Epic 2: Medication Management & Reminders

## Epic Goal

Implement comprehensive medication management system that enables doctors to set medication schedules during patient onboarding, sends smart reminders to patients at doctor-specified times, tracks medication adherence, and escalates missed doses through a multi-tier system (reminder → nudge → caregiver notification → doctor alert). This epic addresses the critical pain point of 50% medication adherence, targeting 85%+ adherence through timely reminders and caregiver involvement. The system works offline-first, storing medication data locally and syncing when connectivity is available.

## Story 2.1: Medication Data Model & Storage

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

## Story 2.2: Doctor Sets Medication Schedule During Patient Onboarding

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

## Story 2.3: Basic Medication Reminders via WhatsApp/Web Chat

As a patient,
I want to receive medication reminders at the times my doctor specified,
so that I don't forget to take my medications.

**Acceptance Criteria:**
1. Reminder scheduler: checks medication schedules and sends reminders at specified times
2. Escalation flow: This starts with a push notification, then WhatsApp, then general nudge to caregiver over WhatsApp
3. Push notification: First reminder sent as push notification to mobile app: "Time for your medication! Metformin 500mg. Tap to log."
4. WhatsApp reminder: If no response to push notification after 15 minutes, send WhatsApp message: "Time for your medication! Metformin 500mg. Reply 'taken' when you've taken it."
5. Caregiver nudge: If no response to WhatsApp reminder after 30 minutes and patient has caregiver linked, send general nudge to caregiver over WhatsApp: "Friendly reminder: [Patient Name] hasn't logged Metformin 500mg yet. Did they take it?"
6. Channel routing: Engagement Engine routes reminders to the channel where patient last interacted. If patient hasn't interacted recently, reminder sent to both channels to ensure delivery.
7. Reminder respects rate limiting: max 5 notifications/day, 2-hour minimum spacing (if multiple medications, applies across both channels)
8. Reminder works offline: scheduled reminders stored locally, sent when connectivity available
9. Patient can confirm medication taken: replies "taken", "yes", "done", "le liya" (Hindi) via their active channel
10. Medication log entry created when patient confirms: medication_id, taken_at timestamp, logged_via "reminder_response"
11. Confirmation message sent via same channel: "Great! I've logged your medication. Keep it up!"
12. Reminder not sent if patient already logged medication within 30 minutes of scheduled time

## Story 2.4: Medication Logging by Patient (Manual Entry)

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

## Story 2.5: Missed Medication Detection & Sequential Nudges

As a patient,
I want a gentle nudge if I miss a medication,
so that I can still take it if I forgot.

**Acceptance Criteria:**
1. Missed medication detection: if medication not logged within 1 hour of scheduled time, mark as missed
2. Sequential nudges: Nudges sent sequentially (not simultaneously) - first via app, then via WhatsApp if no response
3. First nudge (app): send app notification 1 hour after missed time: "Friendly reminder: You haven't logged Metformin 500mg yet. Did you take it? Tap to log."
4. Second nudge (WhatsApp): if no response to app nudge after 30 minutes, send WhatsApp message: "Friendly reminder: You haven't logged Metformin 500mg yet. Did you take it? Reply 'taken' if yes."
5. Patient can still log medication after nudge (within 2 hours of scheduled time)
6. If patient logs medication after nudge, mark as "taken_late" in medication log
7. If medication still not logged 2 hours after scheduled time, escalate to caregiver (Story 2.6)
8. Nudge respects rate limiting: doesn't send if already sent 5 notifications today
9. Nudge works offline: missed medication detected locally, nudge queued for sending
10. Adherence calculation updated: missed medications reduce adherence percentage

## Story 2.6: Medication Escalation to Caregiver (No Doctor Escalation)

As a caregiver,
I want to be notified if my parent misses a medication,
so that I can help ensure they take it.

**Acceptance Criteria:**
1. Caregiver escalation trigger: if medication not logged 2 hours after scheduled time AND patient has caregiver linked
2. WhatsApp message sent to caregiver: "Alert: [Patient Name] hasn't logged Metformin 500mg (scheduled 8:00 AM). Please check if they took it."
3. Caregiver can respond: "taken" or "will remind" or "not taken"
4. If caregiver confirms "taken", medication log entry created with logged_via "caregiver_confirmation"
5. Medication adherence shown as trend (not on every miss/delay) - caregiver can see adherence trends over time
6. Caregiver highlighted when medications are being missed and that will create issues
7. Caregiver escalation only for patients with caregiver-primary or shared management mode
8. Caregiver escalation respects patient privacy: only sends for critical medications or after multiple misses
9. Caregiver notification includes patient context: recent glucose readings, medication history
10. NO escalation to doctor: Doctors will NOT get into every patient's details of whether they have taken their daily medication. Medication adherence shown as trend only, not on every miss/delay.

## Story 2.7: Medication Adherence Trend (No Doctor Escalation)

As a doctor,
I want to see medication adherence trends for my patients,
so that I can understand adherence patterns during patient visits.

**Acceptance Criteria:**
1. Medication adherence shown as trend in doctor dashboard (not on every miss/delay)
2. Adherence metrics: weekly/monthly adherence percentage calculated and displayed
3. Adherence trends: visual trend showing adherence over time (improving, stable, worsening)
4. Adherence context: adherence data includes patient's recent glucose readings, medication history
5. Adherence included in patient detail view and weekly summaries
6. NO escalation to doctor: Doctors will NOT get into every patient's details of whether they have taken their daily medication
7. Adherence data available during patient visits for discussion
8. Caregiver highlighted when medications are being missed and that will create issues (caregiver gets alerts, doctor sees trends)

## Story 2.8: Medication Adherence Tracking & Reporting

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
