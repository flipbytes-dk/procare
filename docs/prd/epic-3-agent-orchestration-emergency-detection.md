# Epic 3: Agent Orchestration & Emergency Detection

## Epic Goal

Build the foundational 7-agent orchestration system that enables ProCare to provide intelligent, coordinated responses to patient actions. Implement the Engagement Engine as the central orchestrator that routes all patient events to appropriate agents, manages patient state, and synthesizes multi-agent responses into single messages. Implement the Clinical Monitor agent to detect emergency glucose levels (hypoglycemia <70 mg/dL, hyperglycemia >250 mg/dL) and provide immediate safety instructions with automatic doctor escalation. Establish agent communication via message bus (RabbitMQ or Redis) and implement priority hierarchy where emergencies override all other notifications. This epic establishes the technical architecture that enables all subsequent intelligent features while ensuring patient safety through emergency detection.

## Story 3.1: Message Bus Infrastructure & Agent Communication

As a developer,
I want a message bus system for agent-to-agent communication,
so that agents can communicate asynchronously and independently.

**Acceptance Criteria:**
1. Message bus infrastructure set up (RabbitMQ or Redis) with queues for each agent
2. Event schema defined: event_type, patient_id, timestamp, data (JSON), source_agent, priority
3. Agent registration system: each agent registers itself and subscribes to relevant event types
4. Event publishing: agents can publish events to message bus
5. Event consumption: agents can consume events from their queues
6. Error handling: failed message processing retried with exponential backoff, dead letter queue for failed messages
7. Message persistence: events stored for replay and debugging
8. Monitoring: message bus health checks, queue depth monitoring, agent status tracking

## Story 3.2: Engagement Engine - Core Orchestrator

As a patient,
I want ProCare to intelligently route my actions to the right agents,
so that I get coordinated, helpful responses.

**Acceptance Criteria:**
1. Engagement Engine service created as central orchestrator
2. Event routing logic: patient action → Engagement Engine → determines which agents need to process event
3. Agent selection: Engagement Engine routes events to relevant agents based on event type (glucose_log → Clinical Monitor + Health Insights, meal_log → Lifestyle Coach + Health Insights)
4. Patient state management: Engagement Engine tracks patient state (onboarding, active, inactive, emergency)
5. Channel routing: Engagement Engine routes responses to the channel where patient last interacted (if patient logged glucose via WhatsApp, response goes to WhatsApp; if via web chat, response goes to web chat)
6. Multi-channel support: Engagement Engine supports sending messages via both WhatsApp and web chat - patient can use either channel at any time, both are always available
7. Channel detection: Engagement Engine detects which channel patient is using based on incoming message source (WhatsApp webhook vs web chat API)
8. Event aggregation: Engagement Engine collects responses from multiple agents
9. Response synthesis: Engagement Engine combines multiple agent responses into single patient message
10. Rate limiting enforcement: Engagement Engine enforces max 5 notifications/day, 2-hour minimum spacing (applies across both channels - total notifications, not per channel)
11. Notification queuing: Engagement Engine queues notifications when rate limit reached, sends when allowed
12. Priority hierarchy: Emergency events override rate limiting and other notifications
13. Logging: Engagement Engine logs all routing decisions, channel detection, and agent responses for debugging

## Story 3.3: Clinical Monitor Agent - Emergency Detection Logic

As a patient,
I want ProCare to detect dangerous glucose levels immediately,
so that I get help when I need it most.

**Acceptance Criteria:**
1. Clinical Monitor agent service created as independent microservice
2. Emergency detection rules: Hypoglycemia and hyperglycemia thresholds picked up from regional protocols (not hardcoded). System supports region-specific medical protocols and guidelines for emergency thresholds.
3. Real-time monitoring: Clinical Monitor processes glucose readings as they're logged
4. Emergency classification: detects hypoglycemia (low), hyperglycemia (high), or normal
5. Emergency priority: emergencies marked as highest priority, override all other notifications
6. Emergency detection works offline: Clinical Monitor processes local glucose readings even when offline
7. Emergency detection accuracy: 98%+ true positive rate (no missed emergencies)
8. False alarm prevention: validates glucose reading (not duplicate, not test strip error) before triggering emergency
9. Emergency context: Clinical Monitor includes patient's recent glucose history, medication status, meal timing in emergency alert
10. Emergency logging: all emergency detections logged with timestamp, glucose value, patient response

## Story 3.4: Clinical Monitor - Emergency Response & Instructions

As a patient experiencing hypoglycemia or hyperglycemia,
I want immediate, clear instructions on what to do,
so that I can take action to stay safe.

**Acceptance Criteria:**
1. Hypoglycemia response: Threshold determined by regional protocols. Immediate message via channel where patient last interacted (WhatsApp or app) with instructions: "URGENT: Your glucose is [value] mg/dL (low). Eat 15g fast-acting sugar (glucose tablets, fruit juice, candy). Check again in 15 minutes. If still low, contact your doctor immediately."
2. Hyperglycemia response: Threshold determined by regional protocols. Immediate message via channel where patient last interacted: "URGENT: Your glucose is [value] mg/dL (high). Drink water, avoid food, check for ketones if you have strips. Contact your doctor if this persists or you feel unwell."
3. Channel routing: Engagement Engine routes emergency messages to the channel where patient last interacted (if patient last used WhatsApp, emergency goes to WhatsApp; if web chat, goes to web chat). If patient hasn't interacted recently, message sent to both channels.
4. Emergency instructions personalized: includes patient's name, recent medication timing, meal timing
5. Emergency instructions in Hindi and English based on patient preference
6. Follow-up check: Clinical Monitor sends follow-up message 15 minutes after hypoglycemia via same channel: "Please check your glucose again. How are you feeling?"
7. Emergency escalation: if patient doesn't respond to emergency or glucose worsens, escalate to doctor immediately (Story 3.5)
8. Emergency instructions work offline: stored locally, sent immediately when connectivity available
9. Emergency acknowledgment: patient can respond "ok", "feeling better", "need help" via their active channel to update emergency status

## Story 3.5: Clinical Monitor - Configurable Doctor Alerts for Emergencies

As a doctor,
I want to configure whether I receive emergency alerts,
so that I can control how I'm notified about patient emergencies.

**Acceptance Criteria:**
1. Alert configuration: Doctor can configure alert preferences (want alerts y/n, if y - immediate/report daily)
2. Default setting: Default setting is N (no alerts) for doctor
3. Alert trigger: If doctor has alerts enabled, emergency detected (thresholds from regional protocols) AND patient doesn't respond within 15 minutes OR glucose worsens
4. Escalation priority: highest priority (overrides all other notifications) when alerts enabled
5. Escalation alert created in doctor dashboard (if alerts enabled): patient name, emergency type, glucose value, timestamp, patient response status
6. Escalation includes context: patient's recent glucose history, medications, caregiver status
7. Doctor notification: If immediate alerts enabled, WhatsApp or SMS sent to doctor: "URGENT: Patient [Name] has [hypoglycemia/hyperglycemia] - glucose [value] mg/dL at [time]. Please check dashboard."
8. Daily report option: If doctor selects "report daily", emergency alerts batched and sent in daily summary
9. Doctor can change alert preferences anytime via dashboard settings
10. Escalation history: all emergency escalations logged for tracking and compliance (even if alerts disabled)
11. Escalation effectiveness: system tracks if doctor responded and patient outcome (when alerts enabled)
12. Escalation only sent if emergency persists or worsens (not for single emergency reading that resolves)

## Story 3.6: Agent Response Synthesis & Notification Bundling

As a patient,
I want a single, coordinated message from ProCare,
so that I'm not overwhelmed with multiple notifications.

**Acceptance Criteria:**
1. Response collection: Engagement Engine collects responses from all agents processing an event
2. Response synthesis: Engagement Engine combines multiple agent responses into single, coherent message
3. Message formatting: synthesized message includes all relevant information without redundancy
4. Example synthesis: "Glucose logged: 128 mg/dL. Looking good! [Clinical Monitor] Your glucose is in target range. [Health Insights] This is similar to your readings after roti meals. [Lifestyle Coach] Keep up the good work!"
5. Notification bundling: if multiple events occur within 2-hour window, bundle into single notification
6. Priority ordering: emergency responses always sent first, then other notifications
7. Message length: synthesized messages kept concise (max 500 characters) for WhatsApp readability
8. Language consistency: synthesized messages maintain consistent tone and language (Hindi/English)
9. Error handling: if agent response fails, Engagement Engine still sends available responses
10. Response logging: all synthesized messages logged for quality improvement

## Story 3.7: Priority Hierarchy & Emergency Override

As a patient,
I want emergency notifications to always get through,
even if I've already received my daily limit of messages.

**Acceptance Criteria:**
1. Priority levels defined: Emergency (highest), Urgent, Attention, Normal (lowest)
2. Emergency override: emergency notifications bypass rate limiting (max 5/day) and send immediately
3. Priority enforcement: Engagement Engine checks priority before applying rate limiting
4. Emergency queue: emergency notifications queued separately from normal notifications
5. Emergency delivery: emergency notifications sent immediately, normal notifications queued if rate limit reached
6. Priority in message bus: emergency events marked with highest priority, processed first
7. Agent priority: Clinical Monitor agent has highest priority, can interrupt other agent processing
8. Emergency notification: patient receives emergency message even if already received 5 notifications today
9. Normal notification deferral: if emergency sent, normal notifications deferred until next allowed window
10. Priority logging: all priority decisions logged for monitoring and debugging

## Story 3.8: Agent Health Monitoring & Error Recovery

As a developer,
I want to monitor agent health and recover from failures,
so that ProCare remains reliable even if individual agents fail.

**Acceptance Criteria:**
1. Agent health checks: each agent exposes health check endpoint (returns status, last processed event, queue depth)
2. Health monitoring: Engagement Engine monitors agent health, detects failures
3. Agent failure detection: if agent doesn't respond to health check within timeout, mark as failed
4. Graceful degradation: if agent fails, Engagement Engine continues with other agents, logs failure
5. Error recovery: failed agents automatically restarted, queued events reprocessed
6. Circuit breaker: if agent fails repeatedly, circuit breaker prevents further requests until recovery
7. Fallback responses: if agent fails, Engagement Engine sends generic response or skips that agent's contribution
8. Error alerting: agent failures logged and alerted to development team
9. Agent status dashboard: monitoring dashboard shows agent health, queue depths, error rates
10. Recovery testing: system tested for agent failure scenarios, graceful degradation verified
