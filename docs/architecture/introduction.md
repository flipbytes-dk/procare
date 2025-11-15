# Introduction

This document outlines the complete full-stack architecture for **ProCare**, a global AI-powered diabetes management platform that combines native mobile apps (iOS/Android) with WhatsApp as a secondary channel. ProCare addresses critical healthcare gaps through proactive pattern-based interventions, comprehensive health records management, integrated diet/lifestyle support, and doctor-connected care.

This unified architecture combines backend services, frontend implementation, mobile applications, and their integration into a single source of truth for AI-driven development. The platform is designed to support 500,000 registered users and 200,000 paid subscribers in Year 1, with global extensibility as a core principle.

**Key Architectural Highlights:**
- **7-Agent Orchestrated AI System** powered by Openrouter API for flexible LLM access
- **Offline-First Mobile Architecture** with 90-day local caching and intelligent sync
- **Multi-Channel Patient Interface** (Native App primary + WhatsApp secondary)
- **Doctor & Caregiver Dashboards** (Web + Mobile) for comprehensive monitoring
- **Microservices Backend** with event-driven agent orchestration
- **Global/Regional Deployment** with localization support (India MVP, global expansion)

This architecture serves patients (Type 2 diabetes management), doctors (passive monitoring with 10-15 min daily review), and caregivers (peace-of-mind monitoring with critical alerts).

**Rationale:**

1. **AI Model Flexibility via Openrouter:** Using Openrouter API as the gateway for all AI agentic tasks (7-agent orchestration) allows ProCare to dynamically switch between models (GPT-4, Claude, Llama, etc.) without architectural changes. This is critical for:
   - Cost optimization (route simpler queries to cheaper models)
   - Performance tuning (switch models based on task complexity)
   - Regional deployment (use locally-compliant models in different countries)
   - Avoiding vendor lock-in

2. **Offline-First Mobile:** 95% of target users in India have intermittent connectivity. Offline-first architecture ensures 24hr stale data maximum, expanding addressable market to 500M+ users. This is a competitive differentiator.

3. **Dual Interface (App + WhatsApp):** Native app provides comprehensive functionality (health records, diet module, offline support). WhatsApp serves as re-engagement channel (550M+ users in India). This dual approach maximizes reach while ensuring paid subscribers get premium app experience.

4. **7-Agent Orchestration:** Complex healthcare workflows (pattern detection, emergency response, doctor escalation) require specialized agents. Microservices architecture allows independent scaling and evolution of each agent while Engagement Engine maintains consistency.

## Starter Template or Existing Project

**Status:** Greenfield project with curated technology choices

**Technology Foundation:**
- **UI Framework:** ShadCN (component library) + TailwindCSS (utility-first CSS)
- **AI Gateway:** Openrouter API for all LLM interactions

**Recommended Starter Template:**

Given the requirements (monorepo, React Native, Next.js web, shared components, TypeScript), I recommend:

**Option 1: Custom T3 Stack + Turborepo (Recommended)**
- **Repository:** Turborepo monorepo
- **Web App:** Next.js 14+ (App Router) + ShadCN + TailwindCSS
- **Mobile App:** React Native (Expo) with shared UI primitives
- **Backend:** tRPC + Next.js API routes (or separate Node.js/Express service)
- **Shared Packages:** TypeScript types, utilities, ShadCN components adapted for React Native
- **Why:** Type-safe full-stack, excellent DX, code sharing between web/mobile, ShadCN works well with Next.js

**Option 2: Create-T3-Turbo Template**
- **Repository:** https://github.com/t3-oss/create-t3-turbo
- **Includes:** Next.js + Expo + tRPC + Prisma + Tailwind pre-configured
- **Pros:** Battle-tested starter, active community, immediate productivity
- **Cons:** May need modifications for 7-agent architecture, WhatsApp integration

**Option 3: NX Monorepo with Custom Setup**
- **Repository:** NX monorepo with React + React Native
- **Flexibility:** More control over structure, better for microservices backend
- **Complexity:** Requires more initial setup

**Recommendation:** Start with **Create-T3-Turbo** as the foundation and extend it with:
- Separate `apps/agent-services/` for the 7-agent microservices
- WhatsApp Business API integration package
- Offline-first sync engine package
- Openrouter API client package

**Architectural Constraints from Template:**
- TypeScript everywhere (enforced)
- tRPC for type-safe API calls (web ↔ backend)
- Prisma ORM for database (can be supplemented with raw SQL for time-series)
- Tailwind + ShadCN for consistent UI across web/mobile

**Alternative if No Template:**
If you prefer building from scratch without a template, we'll design a custom monorepo structure optimized for the 7-agent architecture and multi-platform deployment.

## Change Log

| Date | Version | Description | Author |
|------|---------|-------------|--------|
| 2025-01-13 | 1.0 | Initial architecture document creation | Architect (Winston) |

---
