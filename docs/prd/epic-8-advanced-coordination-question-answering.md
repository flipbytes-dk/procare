# Epic 8: Advanced Coordination & Question Answering

## Epic Goal

Complete the 7-agent orchestration system by implementing the remaining three agents: Care Coordinator for managing test appointments (standard tests prescribed by doctor on regular interval, not just default HbA1c test) and medication refills, Doctor Bridge for intelligent question routing and AI-powered answers, and Learning Library for context-based health education. Implement standard test scheduling with reminders and predictions (doctor prescribes standard tests on regular interval via interface or prescription), AI question answering that handles 80%+ of routine questions with pooled questions for doctors, weekly progress reports for patients and doctors, and context-based educational tips triggered by patient behavior patterns. This epic completes the MVP feature set, enabling comprehensive diabetes management support while maintaining the doctor-connected care model where AI supports but doesn't replace medical oversight.

## Story 8.1: Care Coordinator Agent - Standard Test Scheduling

As a patient,
I want reminders for my standard test appointments prescribed by my doctor,
so that I don't forget important health checkups.

**Acceptance Criteria:**
1. Care Coordinator agent service created as independent microservice
2. Standard test prescriptions: Doctor prescribes standard tests on regular interval (not just default HbA1c test) - can be done via interface or prescription
3. Test scheduling: doctor or patient can schedule test appointments (HbA1c, lipid profile, kidney function, liver function, etc.) based on doctor's prescription
4. Appointment tracking: system tracks upcoming test appointments with date, time, lab location, test type
5. Two-week reminder: system sends reminder 2 weeks before test appointment: "Your [test name] test is scheduled for [date]. I'll remind you again closer to the date."
6. One-week reminder: system sends reminder 1 week before: "Your [test name] test is in 1 week. Please confirm your appointment."
7. One-day reminder: system sends reminder 1 day before: "Don't forget: Your [test name] test is tomorrow at [time] at [lab]."
8. Appointment confirmation: patient can confirm or reschedule appointment via app or WhatsApp
9. Appointment storage: appointments stored in database, synced between local and cloud
10. Appointment history: system tracks past appointments and upcoming appointments
11. Appointment integration: appointments integrated with lab partner APIs (Apollo, Metropolis, Orange Health, etc.) for booking
12. Multiple test support: system supports scheduling multiple standard tests as prescribed by doctor

## Story 8.2: Care Coordinator - HbA1c Prediction

As a patient,
I want to know what my HbA1c might be before the test,
so that I'm motivated to improve if needed.

**Acceptance Criteria:**
1. Prediction algorithm: Care Coordinator analyzes patient's glucose readings over past 3 months
2. HbA1c prediction: calculates predicted HbA1c based on average glucose (formula: HbA1c ≈ (average glucose + 46.7) / 28.7)
3. Prediction accuracy: 95%+ prediction accuracy within 0.2% of actual HbA1c (validated against lab results)
4. Prediction delivery: prediction sent with 2-week reminder: "Based on your glucose readings, your predicted HbA1c is 7.2%. Your target is <7.0%. Keep up the good work!"
5. Prediction motivation: if prediction above target, includes encouragement: "You're close to your target! A few small changes can help."
6. Prediction storage: predictions stored in database, compared with actual results for accuracy validation
7. Prediction improvement: prediction algorithm refined based on actual vs predicted comparisons
8. Prediction display: predictions shown in patient weekly progress report and doctor dashboard
9. Prediction timing: predictions calculated 2 weeks before test (when reminder sent)
10. Prediction language: predictions delivered in patient's preferred language (Hindi/English)

## Story 8.3: Care Coordinator - Lab Result Upload & Analysis

As a patient,
I want to upload my blood test results,
so that ProCare can track my progress and share with my doctor.

**Acceptance Criteria:**
1. Result upload: patient can upload blood test result via WhatsApp (photo of lab report) or web dashboard
2. OCR processing: system uses OCR to extract test values from lab report image (supports multiple test types including HbA1c, lipid profile, kidney function, liver function, etc.)
3. Result validation: system validates extracted values based on test type (e.g., HbA1c typical range 4-15%), prompts for correction if invalid
4. Result storage: blood test results stored in database with date, test type, values, lab name, linked to patient
5. Result analysis: system compares actual result with prediction (for tests with predictions like HbA1c), calculates accuracy
6. Result comparison: system compares current result with previous results, shows trend (improving, stable, worsening)
7. Result notification: patient receives WhatsApp: "Your [test name] result: [value] ([comparison to previous]). Great progress! Your doctor will review this."
8. Doctor notification: doctor receives alert in dashboard: "Patient [Name] [test name]: [value] ([predicted if applicable], actual [value]). Trend: [improving/stable/worsening]."
9. Result display: results displayed in patient detail view in doctor dashboard with trend chart
10. Result history: patients and doctors can view blood test history over time for all test types

## Story 8.4: Doctor Bridge Agent - Question Classification

As a patient,
I want to ask questions about my diabetes,
so that I get answers quickly without bothering my doctor unnecessarily.

**Acceptance Criteria:**
1. Doctor Bridge agent service created as independent microservice
2. Question reception: patient questions received via WhatsApp (text or voice)
3. Question classification: Doctor Bridge classifies questions into categories (nutrition, medication, timing, general, medical)
4. Classification accuracy: 90%+ classification accuracy for question types
5. Routine vs medical: questions classified as "routine" (AI can answer) or "medical" (escalate to doctor)
6. Routine categories: nutrition ("Can I eat mango?"), timing ("When should I check glucose?"), general diabetes management
7. Medical categories: medication questions ("Should I increase my dose?"), symptoms ("I feel dizzy"), treatment questions
8. Question storage: all questions stored in database with classification, patient response, outcome
9. Question context: questions include patient context (recent glucose, medications, patterns) for better classification
10. Question logging: all questions logged for analysis and improvement

## Story 8.5: Doctor Bridge - AI Question Answering

As a patient,
I want AI to answer my routine questions,
so that I get immediate help without waiting for my doctor.

**Acceptance Criteria:**
1. AI integration: Doctor Bridge integrates with conversational AI (GPT-4 or Claude) for question answering
2. Context-aware answers: AI answers include patient's specific context (glucose patterns, medications, recent data)
3. Answer generation: AI generates personalized answers based on patient's data and question
4. Answer examples: "Can I eat mango?" → "Based on your patterns, mango usually spikes your glucose to 160. A small piece (50g) might be okay, but check your glucose 2 hours after. Roti keeps you stable at 140."
5. Answer accuracy: 80%+ of routine questions answered correctly by AI (validated through doctor review)
6. Answer delivery: answers sent to patient via WhatsApp within 2 seconds (per NFR2)
7. Answer language: answers delivered in patient's preferred language (Hindi/English)
8. Answer tone: answers warm, supportive, not judgmental (per brand guidelines)
9. Answer validation: answers reviewed by medical advisor, refined based on feedback
10. Answer improvement: AI answers improve over time based on patient feedback and doctor corrections
11. Enhanced AI guidelines for patients without diabetes specialists:
    - AI responses strictly adhere to established medical guidelines (ADA, IDF, Indian diabetes guidelines)
    - Zero tolerance for hallucinations or out-of-bounds responses
    - More conservative approach: AI errs on side of caution, recommends doctor consultation more frequently
    - AI accuracy target: 95%+ for patients without diabetes specialists (per NFR21)
    - AI response validation: All responses for patients without diabetes specialists validated against medical guidelines database
    - Escalation prompt: AI clearly states when patient should consult a diabetes specialist
    - Response boundaries: AI does not provide medication dosage advice, treatment recommendations, or diagnostic suggestions for patients without diabetes specialists
12. Doctor escalation logic:
    - Patients with diabetes specialists: Medical questions escalated to their connected doctor
    - Patients without diabetes specialists: Medical questions cannot be escalated (no doctor connected), AI provides conservative guidance and recommends going to their diabetes doctor and showing ProCare
    - Question classification: System identifies questions requiring doctor input and handles appropriately based on patient's doctor connection status

## Story 8.6: Doctor Bridge - Pooled Questions & Batch Responses

As a doctor,
I want to provide generic answers to pooled questions from multiple patients,
so that I can efficiently help many patients without answering questions one-to-one.

**Acceptance Criteria:**
1. Question pooling: AI pools similar questions from multiple patients into groups
2. Pool identification: system identifies common question patterns (e.g., "Can I eat mango?" asked by 10 patients)
3. Pool display: doctor sees pooled questions in dashboard: "10 patients asked: 'Can I eat mango?'"
4. Generic answer: doctor provides one generic answer to the pooled question
5. Answer relay: generic answer relayed to all patients who asked the question, with doctor's name attached
6. Answer personalization: system can personalize generic answer with patient-specific context when appropriate
7. Pool threshold: questions pooled when 3+ patients ask similar question within time window (e.g., 1 week)
8. Pool management: doctor can review, answer, or dismiss pooled questions
9. Answer tracking: system tracks which patients received answers, patient satisfaction
10. Pool learning: system learns which questions are commonly pooled, improves pooling algorithm
11. Individual escalation: urgent/medical questions still escalated individually (not pooled)
12. Pool history: pooled questions and answers stored for reference and future use
13. Pooled questions only for connected patients: Pooled questions only include patients with connected diabetes specialists (patients without diabetes specialists receive AI-only responses)

## Story 8.11: Reverse Acquisition - Doctor Data Collection

As ProCare,
I want to collect data about doctors viewing patient data,
so that I can contact them to demonstrate value and encourage platform adoption.

**Acceptance Criteria:**
1. Doctor data collection: System tracks when external doctors (not on ProCare platform) view patient data
2. Data collection trigger: When patient shares data with external doctor (via online interface, report download, or data export)
3. Collected information: Doctor name, phone number, treating condition, clinic/hospital name (if provided by patient)
4. Data storage: External doctor information stored in database with patient reference
5. Data privacy: Patient consent required before collecting doctor information (opt-in during data sharing)
6. Doctor identification: System attempts to identify if doctor is already on ProCare platform (by phone number or name)
7. Reverse acquisition queue: External doctors added to reverse acquisition queue for outreach
8. Outreach data package: System prepares data package showing ProCare value for each doctor:
   - Number of their patients using ProCare
   - Patient outcomes and improvements
   - Value proposition for doctor (time savings, better outcomes, revenue opportunity)
9. Outreach channels: System enables offline contact (phone call) or WhatsApp message to external doctors
10. Outreach tracking: System tracks outreach attempts, responses, and conversions
11. Conversion tracking: System tracks when external doctors sign up on ProCare platform
12. Flywheel effect: System measures reverse acquisition impact (doctors acquired → patients acquired → more doctors)

## Story 8.12: Revenue Sharing Between Referring and Diabetes Specialists

As ProCare,
I want to implement revenue sharing between referring doctors and diabetes specialists,
so that both doctors are incentivized to use the platform.

**Acceptance Criteria:**
1. Revenue sharing model: System implements specific revenue share percentages for referring doctors and diabetes specialists
2. Sharing configuration: Revenue shares are configurable, but default percentages are:
   - To start with, 30% of the total amount paid by the patient goes to the referring doctor
   - 60% of the total amount paid will go to the diabetes specialist in case they bring a patient on board
   - In case a referring doctor brings a patient on board and that patient gets attached to a diabetes specialist, the diabetes specialist will get 30% for that patient, as the other 30% is already being taken by the referring doctor
   - In case there is no diabetes specialist, then the total percentage will be 30% to the referring doctor only
   - In case the patient on boards themselves, there will be no referral fees
3. Patient attribution: System tracks which doctor referred patient and which doctor is managing diabetes
4. Revenue calculation: System calculates revenue share based on patient subscription payments
5. Revenue distribution: Revenue shared monthly based on active patient subscriptions
6. Dashboard display: Both doctors see revenue share information in their dashboards
7. Revenue reporting: Doctors receive monthly revenue reports showing their share
8. Payment processing: Revenue shares processed through payment gateway integration
9. Revenue tracking: System tracks revenue shares for accounting and reporting
10. Edge cases: System handles cases where patient switches doctors, or doctor disconnects
11. Revenue transparency: Doctors can see breakdown of revenue from each patient
12. Self-enrollment handling: Patients who onboard themselves (without doctor referral) generate no referral fees

## Story 8.7: Weekly Progress Reports for Patients

As a patient,
I want a weekly summary of my progress,
so that I can see how I'm doing and stay motivated.

**Acceptance Criteria:**
1. Report generation: system generates weekly progress report for each patient every Monday
2. Report content: includes glucose summary (average, range, readings count), medication adherence, meals logged, patterns detected, wins, opportunities
3. Report format: concise, easy-to-read format delivered via WhatsApp: "Your Week in Review: Glucose average 128 mg/dL (great!), Medications 95% adherence (excellent!), 12 meals logged. Win: You chose roti over rice 4 times this week! Opportunity: Try walking 30 minutes after dinner."
4. Report personalization: reports use patient's name, actual data, specific achievements
5. Report timing: reports sent Monday morning for previous week's review
6. Report language: reports delivered in patient's preferred language (Hindi/English)
7. Report storage: reports stored in database, patients can view past reports in dashboard
8. Report comparison: reports include comparison to previous week (trend indicators)
9. Report motivation: reports emphasize wins and progress, not just problems
10. Report opening: 75%+ patients open weekly summary (per NFR14)

## Story 8.8: Weekly Progress Reports for Doctors

As a doctor,
I want weekly summaries of my patients' progress,
so that I can efficiently review multiple patients in one session.

**Acceptance Criteria:**
1. Report generation: system generates weekly progress report for each patient every Monday morning
2. Report content: includes glucose trends, medication adherence, detected patterns, alerts, key insights, intervention effectiveness
3. Report format: concise format displayed in doctor dashboard: "Patient [Name] - Week Summary: Glucose avg 128 mg/dL (stable), Adherence 95% (excellent), Patterns: Rice spikes glucose, roti stable. Alerts: 0. Status: Stable."
4. Report batch view: doctors can view all patient summaries in batch (all patients at once)
5. Report metrics: reports include key metrics (average glucose, adherence, time in range, pattern changes)
6. Report insights: reports include Health Insights agent-generated insights (from Epic 4)
7. Report alerts: reports highlight patients needing attention based on weekly data
8. Report export: doctors can export weekly summaries as PDF or CSV for records
9. Report timing: reports generated Monday morning, available for doctor's weekly review (10-minute session)
10. Report history: doctors can view past weekly summaries for trend analysis

## Story 8.9: Learning Library Agent - Context-Based Education

As a patient,
I want educational tips that are relevant to my situation,
so that I learn what I need to know when I need it.

**Acceptance Criteria:**
1. Learning Library agent service created as independent microservice
2. Context detection: agent detects patient context (high glucose, missed medication, new pattern detected)
3. Content selection: agent selects relevant educational content based on context
4. Content types: simple, actionable tips (not lengthy articles) - "When glucose is high, drink water and avoid food"
5. Content delivery: educational tips included in patient WhatsApp messages (synthesized by Engagement Engine)
6. Content timing: tips delivered when relevant (after high glucose reading, after missed medication)
7. Content personalization: tips personalized based on patient's patterns and data
8. Content library: library of 50-100 educational tips covering common diabetes management topics
9. Content language: tips delivered in patient's preferred language (Hindi/English)
10. Content effectiveness: system tracks if tips lead to behavior change, refines content based on effectiveness

## Story 8.10: Care Coordinator - Medication Refill Tracking

As a patient,
I want reminders when my medications are running low,
so that I don't run out of important medications.

**Acceptance Criteria:**
1. Medication tracking: Care Coordinator tracks medication supply based on prescription and usage
2. Refill calculation: system calculates when medication will run out based on dosage and frequency
3. Refill reminder: system sends WhatsApp reminder 1 week before medication runs out: "Your Metformin is running low. Time to refill! Contact your doctor or pharmacy."
4. Refill tracking: patient can log medication refills via WhatsApp: "refilled metformin" or "got new prescription"
5. Refill history: system tracks medication refill history for adherence and supply management
6. Refill alerts: if medication not refilled and runs out, alert sent to patient and doctor
7. Refill integration: refill reminders integrated with pharmacy APIs (future Phase 2 feature)
8. Refill display: medication supply status displayed in patient and doctor dashboards
9. Refill reporting: medication refill compliance included in weekly progress reports
10. Refill optimization: system learns patient's refill patterns, optimizes reminder timing
