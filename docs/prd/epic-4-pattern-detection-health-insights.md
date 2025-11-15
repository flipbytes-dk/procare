# Epic 4: Pattern Detection & Health Insights

## Epic Goal

Implement the Health Insights agent that analyzes patient data to detect personalized patterns in food-glucose correlations, activity impacts, and medication consistency. After collecting 4 weeks of patient data, the agent generates personalized insights like "Rice usually spikes your glucose to 180. Roti keeps you at 140" that enable proactive interventions (Epic 7). The agent validates predictions against actual outcomes to improve accuracy over time. This epic delivers the core intelligence that differentiates ProCare from generic health apps, providing patients with actionable, personalized insights rather than generic advice. The system works offline-first, storing pattern data locally and syncing when connectivity is available.

## Story 4.1: Health Insights Agent Infrastructure & Data Collection

As a developer,
I want the Health Insights agent to collect and store patient data for pattern analysis,
so that patterns can be detected after sufficient data is collected.

**Acceptance Criteria:**
1. Health Insights agent service created as independent microservice
2. Data collection: agent subscribes to glucose_log, meal_log, medication_log, activity_log events from message bus
3. Data storage: patient data stored in time-series database (InfluxDB or TimescaleDB) for efficient pattern queries
4. Data schema: glucose_readings, meals, medications, activities with timestamps, patient_id, metadata
5. Minimum data requirement: agent tracks data collection progress, requires 4 weeks minimum before pattern detection
6. Data quality validation: agent validates data completeness (glucose readings, meals, medications) before pattern analysis
7. Data retention: stores 90 days of detailed data locally, 1 year of summary data, >1 year stats only
8. Data sync: patient data synced between local and cloud databases for pattern analysis
9. Agent registration: Health Insights agent registers with Engagement Engine and message bus
10. Event processing: agent processes events asynchronously, doesn't block patient interactions

## Story 4.2: Food-Glucose Correlation Analysis

As a patient,
I want to know which foods affect my glucose levels,
so that I can make better food choices.

**Acceptance Criteria:**
1. Correlation algorithm: Health Insights analyzes glucose readings within 2 hours after meals
2. Food identification: matches logged meals (from Story 1.8) with subsequent glucose readings
3. Pattern detection: identifies foods that consistently cause glucose spikes (>180 mg/dL) or maintain stable levels (<140 mg/dL)
4. Confidence scoring: calculates confidence level for each food-glucose correlation (requires 3+ occurrences for confidence)
5. Pattern examples: "Rice usually spikes your glucose to 180 mg/dL" or "Roti keeps you at 140 mg/dL"
6. Pattern storage: detected patterns stored in database with confidence score, frequency, average glucose impact
7. Pattern detection works offline: analyzes local data, stores patterns locally, syncs when online
8. Pattern validation: validates patterns against new data, updates confidence scores over time
9. Pattern reporting: agent generates pattern summary for patient: "I've learned that rice spikes your glucose, but roti keeps you stable"
10. Error handling: handles missing data, invalid correlations, and edge cases gracefully

## Story 4.3: Activity Impact Analysis

As a patient,
I want to know how activity affects my glucose levels,
so that I can plan my exercise better.

**Acceptance Criteria:**
1. Activity logging: patient can log activities via WhatsApp: "walked 30 minutes", "did yoga", "went for run"
2. Activity data collection: Health Insights agent collects activity logs with timestamps
3. Activity-glucose correlation: analyzes glucose readings before and after activities
4. Pattern detection: identifies activities that lower glucose (walking, exercise) or have no impact
5. Impact quantification: calculates average glucose reduction from activities (e.g., "30-minute walk reduces glucose by 20 mg/dL")
6. Timing analysis: identifies optimal timing for activities (before meals, after meals, morning vs evening)
7. Pattern confidence: requires 3+ occurrences for confidence, updates confidence with more data
8. Pattern storage: activity patterns stored in database with impact, timing, confidence score
9. Pattern reporting: agent generates activity insights: "I've noticed that a 30-minute walk after dinner helps keep your glucose stable"
10. Activity recommendations: agent suggests activities based on detected patterns and glucose levels

## Story 4.4: Medication Consistency Pattern Detection

As a patient,
I want to see how consistent medication timing affects my glucose,
so that I can understand the importance of taking medications on time.

**Acceptance Criteria:**
1. Medication timing analysis: Health Insights analyzes medication logs (from Epic 2) and glucose readings
2. Consistency detection: identifies patterns where consistent medication timing correlates with stable glucose
3. Impact analysis: compares glucose levels when medications taken on time vs late/missed
4. Pattern examples: "When you take Metformin on time, your fasting glucose averages 110 mg/dL. When late, it's 140 mg/dL"
5. Pattern confidence: requires 2+ weeks of data with consistent patterns for confidence
6. Pattern storage: medication consistency patterns stored with impact, timing correlation, confidence score
7. Pattern reporting: agent generates medication insights: "Taking your medications on time helps keep your glucose stable"
8. Adherence correlation: correlates medication adherence (from Epic 2) with glucose control
9. Pattern validation: validates patterns against new medication and glucose data
10. Insights integration: medication patterns integrated into weekly progress reports

## Story 4.5: Pattern Detection After 4 Weeks of Data

As a patient,
I want personalized insights after ProCare learns my patterns,
so that I get actionable advice based on my specific data.

**Acceptance Criteria:**
1. Data sufficiency check: Health Insights checks if patient has 4 weeks of data (glucose readings, meals, medications)
2. Pattern analysis trigger: after 4 weeks, agent automatically runs pattern detection analysis
3. Multi-pattern detection: agent detects food-glucose, activity-impact, and medication-consistency patterns simultaneously
4. Pattern confidence threshold: only reports patterns with >70% confidence (requires 3+ occurrences)
5. Pattern summary generation: agent generates personalized pattern summary: "I've learned your patterns: Rice spikes glucose to 180, roti keeps you at 140. 30-minute walk reduces glucose by 20. Taking medications on time helps stability."
6. Pattern notification: patient receives WhatsApp message: "Great news! I've learned your patterns after 4 weeks. Here's what I found: [pattern summary]"
7. Pattern storage: detected patterns stored in patient profile for use in proactive interventions (Epic 7)
8. Pattern updates: agent re-analyzes patterns weekly, updates confidence scores, detects new patterns
9. Pattern accuracy: 90%+ of patients should have detectable patterns within 4 weeks
10. Pattern reporting: patterns displayed in patient weekly progress report and doctor dashboard

## Story 4.6: Personalized Insight Generation

As a patient,
I want insights that are specific to me,
so that I get advice that actually works for my body.

**Acceptance Criteria:**
1. Insight generation: Health Insights generates personalized insights based on detected patterns
2. Insight types: food recommendations, activity suggestions, medication timing advice, glucose trend explanations
3. Insight personalization: insights use patient's name, specific foods, actual glucose values from their data
4. Insight examples: "Your glucose was 180 after rice yesterday. Roti usually keeps you at 140. Try roti instead?" or "Your fasting glucose is high (140). Taking Metformin on time helps - you took it late 3 times this week"
5. Insight timing: insights generated after glucose logging, meal logging, or weekly analysis
6. Insight delivery: insights included in patient WhatsApp messages (synthesized by Engagement Engine)
7. Insight storage: insights stored in database with timestamp, pattern reference, patient response
8. Insight validation: agent tracks patient responses to insights, validates if insights led to behavior change
9. Insight improvement: agent learns which insights are most effective, prioritizes high-impact insights
10. Insight language: insights delivered in patient's preferred language (Hindi/English), warm and supportive tone

## Story 4.7: Prediction Validation & Learning Loop

As a patient,
I want ProCare to learn and improve its predictions over time,
so that the insights become more accurate and helpful.

**Acceptance Criteria:**
1. Prediction generation: Health Insights generates predictions based on patterns (e.g., "If you eat rice, your glucose will likely spike to 180")
2. Outcome tracking: agent tracks actual glucose readings after predictions
3. Validation: compares predicted glucose values with actual values, calculates accuracy
4. Accuracy threshold: 95%+ glucose predictions within 15 mg/dL of actual values
5. Learning loop: agent updates pattern confidence scores based on prediction accuracy
6. Pattern refinement: if predictions inaccurate, agent re-analyzes patterns, adjusts confidence scores
7. False pattern detection: agent identifies and removes false patterns (correlations that don't hold over time)
8. Pattern evolution: agent detects when patterns change (patient's body responds differently), updates patterns
9. Validation reporting: agent reports prediction accuracy to development team for model improvement
10. Continuous learning: agent continuously learns from new data, improves predictions over time

## Story 4.8: Pattern Display in Dashboards

As a doctor,
I want to see my patient's detected patterns,
so that I can provide better guidance during visits.

**Acceptance Criteria:**
1. Pattern display in doctor dashboard: shows patient's detected patterns (food-glucose, activity-impact, medication-consistency)
2. Pattern visualization: patterns displayed as simple text summaries or basic charts (food → glucose impact)
3. Pattern confidence: doctor dashboard shows confidence level for each pattern (high, medium, low)
4. Pattern timeline: shows when patterns were detected, last updated, validation status
5. Pattern insights: doctor dashboard includes agent-generated insights for each patient
6. Pattern in patient detail view: doctor can see patient's patterns when viewing patient detail
7. Pattern in weekly summary: patterns included in doctor's weekly patient summary
8. Pattern export: doctor can export patient patterns for review or sharing
9. Pattern alerts: doctor dashboard highlights patients with new patterns or pattern changes
10. Pattern context: patterns displayed with patient's glucose trends, medication adherence, meal logging frequency
