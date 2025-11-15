# User Interface Design Goals

## Overall UX Vision

ProCare's user experience centers on reducing anxiety and friction for diabetes management. The primary interface is a native mobile app—providing flexible UX with chat-based primary interface and direct input options (e.g., tags for quick glucose logging without follow-up questions). The app supports comprehensive health records management, diet/lifestyle module, profile viewing, and report downloads. WhatsApp serves as a secondary channel for re-engagement, templated interactions (summaries, questions, basic data collection), and conversational interactions when the app is not being used. Both interfaces are available—patients can use either the app or WhatsApp, with the same onboarding flow but different interfaces. The app provides flexible UX (chat + direct inputs), while WhatsApp remains conversational-only. The system anticipates needs (proactive interventions) rather than reacting after problems occur. Same account, same data across both channels. For doctors and caregivers, web dashboards and mobile apps provide at-a-glance visibility with minimal time investment (10-15 minutes daily for doctors, peace-of-mind checks for caregivers). The offline-first architecture (primarily in the app) ensures the experience never breaks, even with intermittent connectivity—users can log glucose, meals, and medications seamlessly whether online or offline.

## Key Interaction Paradigms

**Native Mobile App (Primary Interface):**
- Chat-based primary interface with conversational capabilities
- Flexible UX with direct input options (e.g., tags below glucose value for quick logging without follow-up questions)
- Context-aware home screen: Dynamically changes first screen based on most relevant action needed (Trends → Pending & Important → Most Used by end user)
- Most used features tracking: System tracks most frequently used features and proactively displays them on home screen
- Trends display: Shows trends across various health markers (not just glucose, but all health record markers)
- Health records recovery: Dedicated area to recover all health records including prescriptions, blood tests, and other medical documents
- Natural language input (text, voice) for conversational interactions
- Photo upload for meal logging, glucometer readings, prescriptions, health records, BP readings, health tracker readings - system automatically discerns photo type and takes appropriate action
- Context-aware responses that feel like talking to a knowledgeable friend
- Proactive messages (not just reactive) with pattern-based interventions
- Rate-limited notifications (max 5/day, 2-hour spacing) to avoid overwhelm
- Comprehensive health records management (upload, store, query, download)
- Diet/lifestyle module with food swaps, calorie management, specific diets, menu recommendations
- Profile viewing and report downloads
- Patient diabetes information benchmark: Shows key information like last HbA1c to help benchmark where to start
- Offline-first functionality with local data storage and sync
- Works seamlessly online and offline

**WhatsApp Interface (Secondary Channel):**
- Conversational-only interface for re-engagement when app is not being used
- Proactive templates and re-engagement: Led by ProCare proactively to stay within WhatsApp policy guidelines
- 24-hour conversation window: If patient wants to engage, system keeps conversation active within 24-hour period
- Direct patient queries: Patient can ask anything directly on WhatsApp to ProCare and get an answer
- Natural language input (text, voice) for all interactions
- Photo sharing for meal logging, glucometer readings, BP readings, health tracker readings - system automatically discerns photo type
- Context-aware responses that feel like talking to a knowledgeable friend
- Proactive messages with pattern-based interventions
- Rate-limited notifications (max 5/day, 2-hour spacing) to avoid overwhelm
- Requires connectivity (no offline functionality)
- Used primarily for re-engagement and templated interactions

**Tiered Information Architecture (Doctor Dashboard):**
- Urgent/Attention/Stable patient categorization for quick triage
- Daily patient buckets: System updates daily (not weekly aggregate) with most important bucket being "Call in for appointment" with reason for doctor's judgment call
- Patient search: Doctor can search for specific patient by ID or phone number to access their profile
- Patient profile: Comprehensive view including health record trends (all markers, not just HbA1c), diet, exercise, last prescription, decision support bot
- Decision support bot: Provides advice on what to look at and what could be possible solutions - important for doctors who don't understand what all can be done for patient in various situations
- Health record trends: All types of health markers viewable as trends (based on various health records uploaded), searchable and bucketed under headers like Liver, Kidney, Heart, etc.
- Context-rich alerts with suggested actions (not just data dumps)
- Minimal clicks to understand patient status and take action

**Caregiver Peace-of-Mind Model:**
- Real-time critical alerts (hypoglycemia, missed meds) for immediate awareness
- Daily digest summaries for routine monitoring
- Separate caregiver view (not patient's view) to respect independence
- Setup and management primarily on caregiver's device (not elderly parent's)

**Offline-First Interaction:**
- All core actions (logging, reminders) work identically offline
- Visual indicators for sync status (synced, syncing, offline)
- Graceful degradation (full features offline, enhanced features online)
- Seamless sync when connectivity returns (no user action required)

## Core Screens and Views

**Patient Experience (Mobile App - Primary):**
- Doctor-initiated onboarding: Doctor says "ProCare will take care of you," then patient completes onboarding (2-3 minutes: phone number, doctor connection, management mode selection, option to upload health records)
- Context-aware home screen: Dynamically shows most relevant action (Trends → Pending & Important → Most Used)
- Patient diabetes information benchmark: Shows key information like last HbA1c to benchmark where to start
- Flexible logging interface: Chat-based primary with direct input options (e.g., glucose tag for quick entry with pre-configured nudge for "Morning fasting" when number entered/button clicked, meal tags, medication tags)
- Glucose logging with time verification: System requests time/state for glucose readings (e.g., "109, Morning fasting") - cannot use system time directly as patient might log reading later
- Conversational logging interface (glucose, meals, medications via natural language when preferred)
- Photo input recognition: System automatically discerns between BP reading, health tracker reading, food photo, glucometer reading, prescription, and takes appropriate action/stores in relevant place
- Health records management: Upload, view, query, download all health records (blood tests, lab reports, medical documents)
- Trends display: Shows trends across various health markers (all health record markers, not just glucose)
- Diet/lifestyle module: Food swaps, calorie management, specific diets (low sodium, low potassium, low fat), menu recommendations from photos
- Prescription upload: Upload prescription photos/documents (including handwritten), system extracts medication info, diet type, and other data with verification
- Meal inference: If previous meal data not logged, system uses current blood glucose reading to infer what was eaten last ("Sugar is 210. That's higher than usual. What could be the reason? Larger portion / different food or Missed medication?")
- Question-answering interface (AI responses with doctor escalation option)
- Profile viewing and report downloads (summarized reports, full history)
- Weekly progress summary
- Offline functionality with sync when connectivity restored

**Patient Experience (WhatsApp - Secondary):**
- Doctor-initiated onboarding: Doctor says "ProCare will take care of you," then patient completes onboarding via WhatsApp (same flow as app: phone number, doctor connection, management mode selection, option to upload health records)
- Proactive templates and re-engagement: Led by ProCare proactively to stay within WhatsApp policy
- 24-hour conversation window: System keeps conversation active within 24-hour period if patient wants to engage
- Direct queries: Patient can ask anything directly on WhatsApp to ProCare and get an answer
- Conversational logging interface (glucose, meals, medications via natural language)
- Glucose logging with time verification: System requests time/state (e.g., "109, Morning fasting") - cannot use system time directly
- Photo input recognition: System automatically discerns between BP reading, health tracker reading, food photo, glucometer reading, and takes appropriate action
- Question-answering interface (AI responses with doctor escalation option)
- Weekly progress summary (delivered via WhatsApp message)
- Templated interactions: Summaries, questions, basic data collection
- Re-engagement channel when app is not being used

**Doctor Web Dashboard & Mobile App:**
- Doctor self-registration: Doctor can self-register and set up with minimal involvement from ProCare - can log on, understand how to activate patients, get business model, see examples, and start using system directly
- Login/Authentication screen
- Patient search: Doctor can search for specific patient by ID or phone number to access their profile
- Main dashboard with tiered patient view (Urgent/Attention/Stable tabs)
- Daily patient buckets: System updates daily (not weekly aggregate) with most important bucket being "Call in for appointment" with reason for doctor's judgment call
- Patient detail view: Comprehensive view including:
  - Health record trends: All types of health markers viewable as trends (based on various health records uploaded), searchable and bucketed under headers like Liver, Kidney, Heart, etc.
  - Diet trends and patterns
  - Exercise data
  - Last prescription
  - Medication adherence (shown as trend, not on every miss/delay)
  - Blood sugar readings
  - Decision support bot: Provides advice on what to look at and what could be possible solutions
- Alert management interface: Cannot ask doctor to change medicines online—most alerts bring the patient back for another meeting. If something is dramatically wrong, the patient goes to the "Schedule appointment" bucket on the doctors interface with appropriate reason. Doctor can request additional activity before appointment (e.g., "Come in on Nov 20th, get HbA1c check done before coming")
- Appointment scheduling: The patient gets a calendar view / whatsapp link to a calendar view to book an appointment and then Procare also gives a link to make the payment for the doctors appointment, and confirms an appointment on the calendar interface as well as sends an intimation to the doctor online (in the calendar), or sends a mail to a designated email ID.
- Appointment calendar: View all appointments scheduled on the system, with an ancillary approach if the doctor doesn't adopt ProCare completely (email/WhatsApp to the assistant to call/confirm and block appointment time)
- Patient onboarding interface: Doctor initiates onboarding by saying "ProCare will take care of you" and provides personalized QR code for patient to scan. Most data comes from prescription upload (some doctors might provide details, but most won't)
- Standard test prescriptions: Doctor prescribes standard tests on regular interval (not just default HbA1c test) - can be done via interface or prescription
- Pooled questions interface: AI pools similar questions from multiple patients, doctor provides generic answers that are relayed to patients with doctor's name via ProCare (doctor does NOT converse directly with patient)
- Health records view: Automatic access to patient health records during every meeting, auto-populated EMR data
- Configurable alerts: Doctor can configure alert preferences (want alerts y/n, if y - immediate/report daily). Default setting is N (no alerts) for doctor
- Doctor communication: Doctors tell ProCare what to tell patient, and ProCare relays message to patient (no direct doctor-patient conversation)

**Caregiver Mobile App:**
- Caregiver can download app and get access similar to patient - context changes but functionality broadly remains same
- Login/Authentication screen
- Main dashboard with patient status overview (current status, recent alerts, adherence metrics)
- Real-time alert feed (critical alerts: hypoglycemia, missed medications)
- Daily digest view (glucose summary, medication adherence, meal patterns)
- Patient detail view (trends, patterns, doctor notes)
- Caregiver-specific elements: Shows "missed", "alerts", "state of mind", "things to watch out for", trends, and suggestions on what to discuss with patient
- No "spying" on patient - respectful monitoring and support
- Question interface: Can ask questions to doctor on behalf of patient or query AI assistant to get answer
- Data update by proxy: Can update data by proxy for patient
- Offline functionality with sync when connectivity restored

**Caregiver Web Dashboard:**
- Login/Authentication screen
- Main dashboard with patient status overview (current status, recent alerts, adherence metrics)
- Real-time alert feed (critical alerts: hypoglycemia, missed medications)
- Daily digest view (glucose summary, medication adherence, meal patterns)
- Patient detail view (trends, patterns, doctor notes)
- Question interface: Can ask questions to doctor on behalf of patient or query AI assistant
- Data update by proxy: Can update data by proxy for patient

**Caregiver WhatsApp Interface:**
- Critical alert notifications (hypoglycemia, missed meds)
- Daily digest summary (delivered via WhatsApp message)
- Question interface: Can ask questions to doctor on behalf of patient or query AI assistant

**Shared Components:**
- Offline status indicator (visible when offline, sync status when online)
- Language selector (Multiple Indian languages for MVP: Hindi, English, Tamil, Telugu, and other regional languages as needed)

## Accessibility: WCAG AA

ProCare must meet WCAG AA standards to serve India's diverse user base, including elderly patients (60+) and users with varying tech literacy. Key accessibility requirements:
- Voice input/output support for low-literacy users
- High contrast text and clear visual hierarchy
- Keyboard navigation for web dashboards
- Screen reader compatibility for web interfaces
- Simple, clear language (avoid medical jargon)
- Large touch targets for mobile web interfaces
- Photo-based input (glucometer screens, meals) reduces text input barriers

## Branding

ProCare's brand identity emphasizes support, not judgment. Tone is warm, encouraging, and personalized—like a trusted friend who understands diabetes management. Visual design should feel:
- **Medical but approachable:** Professional healthcare credibility without clinical coldness
- **Indian context-aware:** Colors, imagery, and language that resonate with Indian users
- **Accessibility-first:** High contrast, clear typography, simple layouts
- **App-native:** Feels natural within the mobile app interface with flexible UX options
- **Offline-resilient:** Visual cues that reinforce reliability (sync status, offline indicators)

No specific brand guidelines provided yet—this will be developed during design phase with focus on reducing anxiety and increasing trust.

## Target Device and Platforms: Cross-Platform

**Primary Platform: Native Mobile Apps (iOS 13+, Android 8+)**
- Native iOS and Android apps as primary interface
- Flexible UX with chat-based primary interface and direct input options
- Comprehensive health records management, diet/lifestyle module, profile viewing, report downloads
- Offline-first functionality with local data storage and sync
- Patients download app as they will be paying for the service
- Enhanced data visualization and offline AI models
- Integration with Bluetooth devices (glucometers, CGM), Google Fit, Apple Health

**Secondary Platform: WhatsApp (iOS 13+, Android 8+)**
- Conversational-only interface for re-engagement when app is not being used
- Templated interactions: summaries, questions, basic data collection
- Accessible to 550M+ WhatsApp users in India
- Requires connectivity (no offline functionality)
- Used primarily for re-engagement and templated interactions

**Tertiary Platform: Web Portal (Responsive)**
- Doctor dashboard: Desktop-first (Chrome 90+, Safari 14+, Firefox 88+)
- Caregiver dashboard: Mobile-responsive (iOS 13+, Android 8+)
- Consolidated patient data view with drill-down capabilities for doctors
- Health records access and EMR integration

**Cross-Platform Considerations:**
- Consistent experience across app and WhatsApp (same data, same insights)
- Patients can use either app or WhatsApp - both channels are always available
- Same onboarding flow but different interfaces (app vs WhatsApp)
- App provides flexible UX (chat + direct inputs), WhatsApp remains conversational-only
- Messages route to the channel where patient last interacted
- Same account, same data across both channels - seamless multi-channel support
- Offline-first architecture primarily in app (WhatsApp requires connectivity)
- Responsive design ensures usability on wide range of screen sizes (4" to desktop)
- Global/regional support: time zones, local protocols, multi-language, region-specific medical guidelines
