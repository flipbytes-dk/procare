# Goals and Background Context

## Goals

- Enable Type 2 diabetes patients in India to reduce HbA1c to <7.0% through personalized, AI-powered guidance
- Achieve 85%+ medication adherence through smart reminders and caregiver involvement
- Provide immediate, personalized feedback to reduce patient anxiety ("Is this glucose reading okay?")
- Enable doctors to save 5-10 minutes per patient while improving patient outcomes through passive monitoring
- Reduce emergency calls to doctors by 60% (from 3-5/week to 1-2/week) through proactive pattern-based interventions
- Achieve 75%+ Week 1 retention through exceptional first-week experience (2+ personalized insights, quick answers, doctor connection)
- Support 500,000 registered users and 200,000 paid subscribers in Year 1
- Establish native mobile app as primary interface with WhatsApp as secondary channel for re-engagement (app-first architecture, 2-3 minute onboarding)
- Enable offline-first functionality so 95%+ patients have <24hr stale data, expanding addressable market to 500M+ users
- Detect 90%+ of patients' food-glucose patterns within 4 weeks to enable proactive interventions
- Achieve 80%+ AI question answering accuracy (routine questions) with 20% escalation to doctors
- Create a product that people 'want' to have, because it offers value across various health parameters including centralised storage of data, 24x7 assistance, diet support for all types of diets - personalised meal by meal, caregiver support and more
- Make a product that is easily extendable across the globe to any part of the world, using the same methodology
- Help reduce the HbA1c of patients by 1 point within 5 months of them using the system

## Background Context

This is a global problem, and we are going after places where the doctor is the one who will push the service to patients. The payment could happen through the patient, the caregiver, the company, the TPA or the insurance agency. The problem of reducing HbA1c to the required numbers is what we are solving for. We will launch with India - but the product should be applicable for any location by just setting up the system as per the laws of the country.

ProCare addresses a critical healthcare gap in diabetes management globally. The problem stems from reactive medicine—doctors see patients every 3 months with no visibility into behavior between visits, while patients experience daily anxiety with no one to ask "Is this okay?" or "Can I eat mango?"

Existing solutions fail because they don't provide comprehensive health records management (critical value prop), lack integrated diet/lifestyle management (95% of Type 2 diabetes management is diet), don't automatically populate doctor EMR systems, provide generic advice rather than personalized insights, lack doctor connection (patients want oversight), and use judgmental tones that increase guilt. International platforms aren't designed for unique regional needs: dual interface (app + WhatsApp), health records centralization, prescription-based data extraction, family involvement in care, and affordability.

ProCare solves this through a 7-agent orchestrated platform that combines native mobile app (primary interface with flexible UX) and WhatsApp (secondary channel for re-engagement and templated interactions), comprehensive health records management (upload, store, query, auto-populate doctor EMR), integrated diet/lifestyle module (food swaps, calorie management, specific diets, menu recommendations), offline-first architecture (works without internet), doctor-connected care (not AI-only), family/caregiver involvement (2.3x adherence improvement), and proactive pattern-based interventions (preventive vs reactive). The platform learns each patient's unique patterns over 4 weeks, then provides pre-meal warnings like "Rice usually spikes your glucose to 180. Roti keeps you at 140. What are you having?"—preventing problems rather than responding after they occur. Patients pay for the service, making app download a natural expectation, while WhatsApp serves as a re-engagement channel when the app is not being used.

ProCare employs a dual doctor acquisition strategy: (1) Direct onboarding where diabetes specialists onboard their patients directly, and (2) Reverse acquisition where non-diabetes specialists (heart, liver, kidney specialists) refer patients who then discover ProCare's value and introduce it to their diabetes specialists, creating a flywheel effect. Patients referred by non-diabetes specialists receive a 1-month free trial, after which they must subscribe to maintain full access. This strategy expands the addressable market beyond diabetes specialists while creating natural pathways for diabetes specialist acquisition.

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-11-07 | 1.0 | Initial PRD creation from Project Brief | PM (John) |
| 2025-11-08 | 1.1 | Added web chat interface (Open WebUI) as alternative communication channel alongside WhatsApp | PM (John) |
| 2025-11-09 | 1.2 | Major platform shift: Native mobile app (primary) + WhatsApp (secondary), health records management, diet/lifestyle module, doctor-initiated onboarding, prescription upload, doctor pooled questions, global/regional support, comprehensive integrations | PM (John) |
| 2025-11-09 | 1.3 | Enhanced mobile app UX (context-aware home screen, trends, most used features), expanded Indian language support, improved WhatsApp interactions (24-hour window, proactive templates), enhanced doctor interface (search, decision support bot, daily patient buckets), caregiver app access, ABHA integration planning, glucose logging time verification, photo input recognition, sequential medication nudges, regional protocol-based thresholds, configurable alerts, doctor self-registration, handwritten prescription support, meal inference from glucose, standard test prescriptions, location packs for global deployment | PM (John) |
| 2025-11-10 | 1.4 | Added referring doctor workflow (non-diabetes specialists can refer patients), trial period management (1 month free for referred patients), subscription restrictions (restricted access after trial), enhanced AI guidelines for patients without diabetes specialists, revenue sharing model (referring doctor + diabetes specialist), reverse acquisition strategy for diabetes specialists | PM (John) |
| 2025-11-12 | 1.5 | Updated goals (removed baseline references, changed time savings to per-patient, updated user targets, added global extensibility goals), revised background context for global focus, updated FR3 (medication escalation), FR7 (country-specific emergency thresholds), FR10 (generic blood tests), FR13 (doctor onboarding process), FR33 (AI guidelines scope), FR35/FR36 (revenue sharing details), NFR10 (global language support), added Epic 13 (appointment booking), updated Stories 1.5-1.10 (UI/UX requirements), Story 1.13 (subscription restrictions), Story 2.3 (medication reminder flow), Story 5.2 (appointment bucket visibility), Story 5.4 (referring doctor prescriptions), Story 5.5 (onboarding and revenue sharing), Story 8.3 (generic blood tests), Story 8.5 (AI guidance), Story 8.12 (revenue sharing percentages), added Stories 1.4a (caregiver enrollment) and 1.4b (self enrollment) | PM (John) |
