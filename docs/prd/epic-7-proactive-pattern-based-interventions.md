# Epic 7: Proactive Pattern-Based Interventions

## Epic Goal

Enable proactive, pattern-based interventions that prevent problems before they occur, rather than reacting after glucose spikes or medication misses. After patterns are detected (Epic 4), implement pre-meal warnings that alert patients before they eat problematic foods (e.g., "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"), enabling patients to make better choices in real-time. Implement notification bundling where multiple agent responses are combined into single messages, and enforce rate limiting (max 5 notifications/day, 2-hour minimum spacing) to avoid overwhelming patients. This epic delivers the "don't forget umbrella" vs "you're wet, here's a towel" experience, where prevention is easier than correction. The system works offline-first, storing intervention logic locally and triggering interventions based on learned patterns.

## Story 7.1: Pre-Meal Timing Detection

As a patient,
I want ProCare to warn me before I eat something that will spike my glucose,
so that I can make a better choice.

**Acceptance Criteria:**
1. Meal timing detection: system detects when patient is likely about to eat based on patterns (typical meal times, meal logging frequency)
2. Pattern-based triggers: interventions triggered when patient's typical meal time approaches AND patterns detected (from Epic 4)
3. Pre-meal window: system detects meal timing within 30 minutes before typical meal time
4. Pattern availability check: system verifies that food-glucose patterns are available for patient (requires Epic 4 completion)
5. Intervention eligibility: patient must have 4+ weeks of data and detectable patterns for pre-meal interventions
6. Timing detection works offline: meal timing patterns stored locally, detection works without connectivity
7. Multiple meal times: system supports multiple meal times per day (breakfast, lunch, dinner, snacks)
8. Pattern confidence: only triggers interventions for patterns with >70% confidence (from Epic 4)
9. Intervention frequency: pre-meal interventions sent max once per meal time per day
10. Timing detection logging: system logs meal timing detections for pattern refinement

## Story 7.2: Pre-Meal Warning Generation

As a patient,
I want personalized warnings about foods that affect my glucose,
so that I can choose better options.

**Acceptance Criteria:**
1. Warning generation: when meal time detected, system generates pre-meal warning based on detected patterns
2. Warning content: includes problematic food, expected glucose impact, better alternative, personalized message
3. Warning example: "Rice usually spikes your glucose to 180 mg/dL. Roti keeps you at 140 mg/dL. What are you having for lunch?"
4. Pattern selection: system selects most impactful pattern (highest glucose spike) for warning
5. Alternative suggestion: system suggests better alternative based on patterns (food that keeps glucose stable)
6. Warning personalization: warnings use patient's name, actual glucose values from their data, specific foods they eat
7. Warning language: warnings delivered in patient's preferred language (Hindi/English), warm and supportive tone
8. Warning timing: warnings sent 15-30 minutes before typical meal time (not too early, not too late)
9. Warning storage: warnings stored in database with timestamp, pattern reference, patient response
10. Warning effectiveness: system tracks if patient changed behavior after warning (logged different food)

## Story 7.3: Pre-Meal Warning Delivery via WhatsApp

As a patient,
I want to receive pre-meal warnings via WhatsApp,
so that I get them at the right time without checking an app.

**Acceptance Criteria:**
1. Warning delivery: pre-meal warnings sent to patient via WhatsApp (integrated with WhatsApp API)
2. Delivery timing: warnings sent at optimal time (15-30 minutes before meal) based on patient's patterns
3. Warning format: concise WhatsApp message with food comparison and question: "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"
4. Patient response: patient can respond with food choice, system logs response and tracks glucose outcome
5. Response tracking: system tracks if patient chose better alternative (roti) or problematic food (rice)
6. Outcome validation: system validates warning effectiveness by comparing predicted vs actual glucose after meal
7. Warning frequency: max one pre-meal warning per meal time per day (respects rate limiting)
8. Warning opt-out: patient can opt out of pre-meal warnings if they find them annoying
9. Warning preferences: patient can customize warning timing or frequency
10. Warning integration: warnings integrated with Engagement Engine for rate limiting and notification bundling

## Story 7.4: Behavior Change Tracking & Validation

As a patient,
I want ProCare to learn if its warnings help me make better choices,
so that the advice becomes more effective over time.

**Acceptance Criteria:**
1. Behavior tracking: system tracks patient responses to pre-meal warnings (chose better alternative vs problematic food)
2. Glucose outcome tracking: system tracks actual glucose readings after patient's meal choice
3. Behavior change detection: system detects if patient changed behavior (chose roti instead of rice after warning)
4. Change validation: compares glucose outcomes when patient chose better alternative vs problematic food
5. Effectiveness calculation: calculates warning effectiveness (% of times patient chose better alternative after warning)
6. Pattern refinement: if warnings effective, system increases pattern confidence; if not, refines warning approach
7. Success metrics: tracks behavior change rate (target: 30%+ patients change behavior after warning)
8. Response rate tracking: tracks patient response rate to warnings (target: 70%+ respond to warnings)
9. Long-term tracking: system tracks behavior change over time (sustained change vs one-time change)
10. Reporting: behavior change metrics included in weekly progress reports and doctor dashboard

## Story 7.5: Notification Bundling Logic

As a patient,
I want a single, coordinated message from ProCare,
so that I'm not overwhelmed with multiple notifications.

**Acceptance Criteria:**
1. Bundling detection: Engagement Engine detects when multiple agents have responses for same event or time window
2. Bundling window: responses within 2-hour window are bundled into single message
3. Response collection: Engagement Engine collects responses from all relevant agents (Clinical Monitor, Health Insights, Lifestyle Coach, etc.)
4. Message synthesis: Engagement Engine combines multiple responses into single, coherent WhatsApp message
5. Synthesis example: "Glucose logged: 128 mg/dL. Looking good! [Clinical Monitor] Your glucose is in target range. [Health Insights] This is similar to your readings after roti meals. [Lifestyle Coach] Keep up the good work!"
6. Priority ordering: emergency responses always sent first, then bundled normal responses
7. Message length: bundled messages kept concise (max 500 characters) for WhatsApp readability
8. Redundancy removal: Engagement Engine removes redundant information from multiple agent responses
9. Language consistency: bundled messages maintain consistent tone and language (Hindi/English)
10. Bundling logging: all bundled messages logged for quality improvement and debugging

## Story 7.6: Rate Limiting Enforcement

As a patient,
I want ProCare to respect my notification limits,
so that I'm not overwhelmed with messages.

**Acceptance Criteria:**
1. Rate limit rules: max 5 notifications per day, minimum 2-hour spacing between notifications
2. Rate limit tracking: Engagement Engine tracks notification count and timing for each patient
3. Notification queuing: if rate limit reached, notifications queued for next allowed window
4. Emergency override: emergency notifications (from Clinical Monitor) bypass rate limiting
5. Priority queuing: urgent notifications queued separately from normal notifications, sent first when allowed
6. Queue management: queued notifications prioritized by urgency, oldest first within same priority
7. Queue notification: patient receives notification when queue processed: "Here are your updates: [bundled message]"
8. Rate limit display: patient can check notification count via WhatsApp: "You've received 3 notifications today. 2 remaining."
9. Rate limit preferences: patient can adjust rate limits (increase or decrease) based on preferences
10. Rate limit logging: all rate limit decisions logged for monitoring and optimization

## Story 7.7: Proactive Medication Reminder Optimization

As a patient,
I want medication reminders that adapt to my patterns,
so that I get reminders at times when I'm most likely to take medications.

**Acceptance Criteria:**
1. Medication timing patterns: system analyzes when patient typically takes medications (from Epic 2 logs)
2. Optimal timing detection: identifies optimal reminder timing based on patient's actual medication-taking patterns
3. Timing optimization: adjusts reminder timing to 15 minutes before patient's typical medication time
4. Pattern-based reminders: reminders personalized based on patient's medication-taking patterns
5. Reminder effectiveness: tracks if optimized reminders improve medication adherence vs fixed-time reminders
6. Reminder refinement: system refines reminder timing based on patient response and adherence outcomes
7. Multiple medications: optimizes timing for each medication separately based on individual patterns
8. Pattern updates: reminder timing updates as patient patterns change over time
9. Reminder integration: optimized reminders integrated with medication management system (Epic 2)
10. Reminder reporting: reminder effectiveness metrics included in weekly progress reports

## Story 7.8: Intervention Effectiveness Reporting

As a doctor,
I want to see if proactive interventions are helping my patients,
so that I can understand the impact of ProCare.

**Acceptance Criteria:**
1. Intervention metrics: system tracks intervention effectiveness (pre-meal warnings, optimized reminders)
2. Behavior change metrics: tracks % of patients who changed behavior after interventions
3. Glucose outcome metrics: tracks glucose improvements after interventions (reduced spikes, better control)
4. Adherence improvement: tracks medication adherence improvements after optimized reminders
5. Intervention reporting: intervention metrics displayed in doctor dashboard for each patient
6. Aggregate reporting: doctor dashboard shows aggregate intervention effectiveness across all patients
7. Weekly summary integration: intervention effectiveness included in weekly patient summaries
8. Trend analysis: system shows intervention effectiveness trends over time (improving or declining)
9. Patient-specific reporting: doctor can see which interventions work best for each patient
10. Export functionality: doctors can export intervention effectiveness data for analysis
