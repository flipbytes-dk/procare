# User Personas

This document defines the primary user personas for ProCare, providing detailed profiles that guide product design, feature prioritization, and user experience decisions.

---

## Primary Persona: Type 2 Diabetes Patient

### Persona 1: Rajesh Kumar (Patient-Primary Mode)

**Demographics:**
- **Age:** 52 years old
- **Location:** Bangalore, India
- **Occupation:** Middle manager at IT company
- **Family:** Married, 2 adult children
- **Education:** Bachelor's degree
- **Income:** ₹80,000/month (middle class)

**Diabetes Profile:**
- **Type:** Type 2 Diabetes
- **Diagnosis:** 3 years ago
- **Current HbA1c:** 8.2% (target: <7.0%)
- **Medications:** Metformin 500mg (2x daily), Glimepiride 2mg (morning)
- **Complications:** None yet, but worried about future complications

**Technology Proficiency:**
- **Smartphone:** Android (Samsung Galaxy), uses daily
- **Apps Used:** WhatsApp (daily), YouTube (daily), Google Pay (frequently)
- **Tech Comfort:** Medium - comfortable with apps but prefers simple interfaces
- **Internet:** 4G at home and office, occasional connectivity issues during commute

**Goals & Motivations:**
- Reduce HbA1c to <7% to avoid complications
- Understand which foods spike his glucose and which don't
- Remember to take medications on time (currently misses 2-3 times per week)
- Avoid emergency situations that worry his wife
- Feel confident about his diabetes management

**Pain Points & Frustrations:**
- **Anxiety:** "Is this glucose reading okay?" - no one to ask until next doctor visit (3 months away)
- **Food Confusion:** Doesn't know what to eat - "Can I eat mango? Can I have rice?" - conflicting advice from family and internet
- **Medication Adherence:** Forgets evening medication when working late or traveling
- **Information Overload:** Too much conflicting diabetes information on internet, doesn't know what to trust
- **Doctor Unavailability:** Doctor only available every 3 months, emergency calls feel intrusive

**Behaviors & Habits:**
- Checks glucose 2x/day (morning fasting, evening post-dinner) using glucometer
- Uses WhatsApp extensively - prefers WhatsApp over apps for quick interactions
- Logs data occasionally in paper notebook, loses track
- Wife (Priya) helps with diet planning and medication reminders
- Watches YouTube videos about diabetes (Hindi and English)

**User Needs:**
1. Immediate answers to daily questions without waiting for doctor
2. Personalized food recommendations based on his own glucose responses
3. Reliable medication reminders that escalate if he forgets
4. Confidence that someone is monitoring for emergencies
5. Simple, non-judgmental interface that reduces anxiety

**Typical Day:**
- **7:00 AM:** Wake up, check fasting glucose (target: 90-110 mg/dL)
- **7:30 AM:** Take morning medications with breakfast
- **1:00 PM:** Lunch at office canteen - uncertain what to choose
- **7:00 PM:** Evening walk (30 minutes) when possible
- **9:00 PM:** Dinner at home (wife cooks), check post-dinner glucose
- **10:00 PM:** Forgets evening medication 2-3 times/week

**ProCare Usage Pattern:**
- Primary interface: Native mobile app (quick glucose logging with tags)
- Secondary: WhatsApp (re-engagement notifications, summaries)
- Logs glucose 2x/day via app (takes 30 seconds with quick tags)
- Asks 2-3 questions per week via app chat
- Receives 2-3 personalized insights per week after 4 weeks of data
- Wife (caregiver) monitors via web dashboard

---

## Secondary Persona: Diabetes Specialist Doctor

### Persona 2: Dr. Meera Sharma (Diabetes Specialist)

**Demographics:**
- **Age:** 45 years old
- **Location:** Mumbai, India
- **Specialty:** Endocrinologist specializing in diabetes
- **Practice:** Private clinic + hospital consultations
- **Years of Experience:** 18 years
- **Education:** MBBS, MD (Internal Medicine), DM (Endocrinology)

**Professional Profile:**
- **Patient Load:** 200 active diabetes patients
- **Consultation Schedule:** 6-8 hours/day, 6 days/week
- **Consultation Duration:** 15 minutes per patient (too short for thorough review)
- **Patient Reviews:** Quarterly (every 3 months)
- **Emergency Calls:** Receives 4-5 emergency calls per week from anxious patients

**Technology Proficiency:**
- **Devices:** iPhone, MacBook Pro for clinic management
- **Apps Used:** WhatsApp (patient communication), Email, EMR system (Practice Management Software)
- **Tech Comfort:** High - adopts new technology if it saves time
- **Internet:** Reliable WiFi at clinic and hospital

**Goals & Motivations:**
- Help patients achieve HbA1c <7% and avoid complications
- Reduce time spent on routine questions and non-emergencies
- Monitor patients between quarterly visits to catch issues early
- Provide better care without adding more hours to workday
- Scale practice to help more patients without sacrificing quality

**Pain Points & Frustrations:**
- **Time Burden:** Spends 10-15 minutes per patient reviewing status during quarterly visits - wants to reduce to 5 minutes
- **Emergency Calls:** Receives 4-5 emergency calls per week, but 60% are not real emergencies (e.g., "My glucose is 160, is that okay?")
- **No Visibility Between Visits:** Patients come every 3 months, but diabetes management happens daily - no way to monitor in between
- **Routine Question Overload:** 80% of patient questions are routine (food, timing) that don't require doctor expertise
- **Data Entry Burden:** Patients show paper logs or screenshots - manual data entry into EMR system
- **Reactive Medicine:** Only sees patients when problems have already occurred, can't intervene proactively

**Behaviors & Habits:**
- Checks WhatsApp between patients for urgent messages
- Prefers desktop/laptop for patient review (larger screen, faster navigation)
- Reviews patient data in batches (Monday morning for week ahead)
- Uses EMR system for patient records and prescriptions
- Delegates routine tasks to clinic staff when possible

**User Needs:**
1. Quick patient status overview (Urgent/Attention/Stable) for triage
2. Automatic data collection from patients (glucose, medications, meals) without manual entry
3. AI handles routine questions, escalates only medical/medication questions
4. Proactive alerts for emergencies and concerning patterns
5. Batch review capability (10-15 minutes for all patients on Monday morning)
6. Patient onboarding with minimal doctor involvement

**Typical Week:**
- **Monday Morning (10 min):** Review weekly patient summaries, identify patients needing attention
- **Throughout Week (5 min/day):** Quick check of urgent alerts and pooled questions
- **Monthly:** Onboard 3-5 new patients via QR code (30 seconds per patient)
- **Quarterly:** In-person patient consultations with data already in system

**ProCare Usage Pattern:**
- Primary interface: Web dashboard (desktop at clinic)
- Onboards patients via QR code in clinic ("ProCare will take care of you")
- Reviews tiered patient view (Urgent/Attention/Stable) on Monday mornings (10 minutes)
- Responds to pooled questions (AI pools similar questions from multiple patients, doctor provides one generic answer relayed to all)
- Receives configurable alerts for emergencies only (not routine fluctuations)
- Views comprehensive patient data during quarterly consultations (automatically populated from ProCare)

---

## Tertiary Persona: Caregiver

### Persona 3: Priya Kumar (Caregiver for Rajesh)

**Demographics:**
- **Age:** 48 years old
- **Location:** Bangalore, India
- **Occupation:** Homemaker (previously worked as teacher)
- **Relationship:** Wife of Rajesh (patient)
- **Education:** Master's degree
- **Tech Proficiency:** Medium (uses WhatsApp, YouTube, shopping apps)

**Caregiver Profile:**
- **Patient:** Husband Rajesh (52, Type 2 Diabetes for 3 years)
- **Caregiving Duration:** 3 years since diagnosis
- **Involvement Level:** High - manages diet, medication reminders, accompanies to doctor visits
- **Other Responsibilities:** Household management, caring for elderly mother-in-law

**Technology Proficiency:**
- **Smartphone:** Android (Redmi), uses daily
- **Apps Used:** WhatsApp (daily), YouTube (cooking videos), Swiggy/Zomato (occasionally)
- **Tech Comfort:** Medium - comfortable with WhatsApp, needs simple apps
- **Internet:** 4G at home, WiFi when available

**Goals & Motivations:**
- Keep husband healthy and avoid diabetes complications
- Ensure he takes medications on time every day
- Prepare diabetes-friendly meals he enjoys
- Reduce her own anxiety about his health
- Be notified immediately if emergency occurs

**Pain Points & Frustrations:**
- **Anxiety:** Constantly worried about husband's glucose levels and health
- **Medication Adherence:** Husband forgets evening medication when working late or stressed
- **Food Preparation:** Difficult to cook diabetes-friendly food that entire family enjoys
- **Information Overload:** Conflicting advice from doctors, family, internet about what to cook
- **Emergency Fear:** Worries about hypoglycemia when he's alone (at office, traveling)
- **Communication Gap:** Husband doesn't always share glucose readings or symptoms with her

**Behaviors & Habits:**
- Sets phone alarms to remind husband to take medications
- Calls husband at medication times if he's at office
- Plans weekly meals focusing on diabetes-friendly options
- Accompanies husband to doctor visits (takes notes)
- Searches YouTube for "diabetes recipes" (Hindi and English)
- Checks in with husband multiple times daily about how he's feeling

**User Needs:**
1. Real-time alerts for critical situations (hypoglycemia, hyperglycemia)
2. Daily digest of husband's glucose levels and medication adherence (without being intrusive)
3. Simple dashboard showing husband's status (good/attention needed)
4. Ability to help husband log data when he forgets
5. Non-judgmental reminders (avoid amplifying anxiety)

**Typical Day:**
- **7:00 AM:** Ensure husband takes morning medication with breakfast
- **9:00 AM:** Receive daily digest (WhatsApp): "Rajesh's fasting glucose: 105 mg/dL (good), medications taken on time yesterday"
- **Throughout Day:** Check caregiver dashboard 1-2 times for peace of mind
- **7:00 PM:** Cook dinner based on food swap recommendations from ProCare
- **9:00 PM:** Remind husband to check glucose and take evening medication (if ProCare sends nudge)
- **Emergency:** Receive immediate WhatsApp alert if husband's glucose is critically low/high

**ProCare Usage Pattern:**
- Primary interface: WhatsApp (real-time critical alerts, daily digest)
- Secondary: Mobile app (dashboard for detailed view when needed)
- Receives critical alerts immediately (hypoglycemia, hyperglycemia, missed medications)
- Receives daily digest at 9 AM ("Rajesh's glucose yesterday: 2 readings in target range, 1 medication taken late")
- Checks app dashboard 1-2 times daily for reassurance
- Can log data on behalf of husband when he forgets (shared management mode)

---

## Supporting Persona: Referring Doctor (Non-Diabetes Specialist)

### Persona 4: Dr. Arjun Patel (Cardiologist - Referring Doctor)

**Demographics:**
- **Age:** 50 years old
- **Location:** Ahmedabad, India
- **Specialty:** Cardiologist
- **Practice:** Hospital-based + private consultations
- **Years of Experience:** 22 years
- **Education:** MBBS, MD (Cardiology), FACC

**Professional Profile:**
- **Patient Load:** 150 active cardiac patients (many with diabetes as comorbidity)
- **Diabetes Expertise:** Basic understanding, refers to endocrinologists for diabetes management
- **Consultation Focus:** Heart health, blood pressure, cholesterol management
- **Patient Demographics:** 60% have Type 2 Diabetes as comorbidity

**Goals & Motivations:**
- Ensure diabetes patients manage their condition to reduce cardiac risk
- Refer patients to diabetes specialists when needed
- Earn revenue share from patient referrals to diabetes platform
- Provide comprehensive care without becoming diabetes expert

**Pain Points & Frustrations:**
- Many cardiac patients have unmanaged diabetes (increases cardiac risk)
- Doesn't have time to manage their diabetes in detail
- Refers to endocrinologists, but patients often don't follow through
- Wants to help patients but diabetes management is outside specialty

**User Needs:**
1. Easy way to refer patients to diabetes management platform
2. Confidence that patients will get good diabetes care
3. Revenue share for successful patient referrals
4. Minimal involvement after referral (no diabetes management burden)

**ProCare Usage Pattern:**
- Refers patients via QR code: "I'm treating you for your heart condition. For your diabetes management, ProCare will take care of you. Scan this QR code to get started."
- Patient gets 1-month free trial automatically
- Dr. Patel earns 30% revenue share if patient subscribes
- Dr. Patel can view basic patient status in ProCare if needed (optional)
- If patient connects with diabetes specialist later, Dr. Patel still earns 30% revenue share for that patient

---

## Persona Usage in Product Development

### Design Decisions Informed by Personas

1. **Rajesh (Patient):** Mobile app with quick glucose logging tags (not just conversational) because he's time-pressed and wants speed
2. **Dr. Sharma (Diabetes Specialist):** Monday morning batch review feature (10-15 minutes for all patients) instead of daily interruptions
3. **Priya (Caregiver):** WhatsApp as primary caregiver interface (she's already comfortable with WhatsApp, doesn't want another app)
4. **Dr. Patel (Referring Doctor):** QR code onboarding (30 seconds, no form filling) with automatic revenue share calculation

### Feature Prioritization Based on Personas

- **High Priority (MVP):** Features addressing Rajesh's anxiety and Dr. Sharma's time burden
- **Phase 2:** Advanced features like diet module and health records management (enhances Rajesh's experience but not critical for launch)
- **Throughout:** Non-judgmental tone to reduce Priya's anxiety amplification

---

## Document Control

**Version:** 1.0
**Created:** 2025-01-14
**Author:** Sarah (Product Owner)
**Status:** Final
**Related Documents:**
- [Goals and Background Context](./goals-and-background-context.md)
- [User Interface Design Goals](./user-interface-design-goals.md)
- [Requirements](./requirements.md)
- [Epic List](./epic-list.md)
