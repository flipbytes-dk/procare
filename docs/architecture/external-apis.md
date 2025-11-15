# External APIs

ProCare integrates with multiple external services for AI capabilities, WhatsApp messaging, payments, lab bookings, and device integrations. This section documents each integration with implementation details.

## OpenRouter API

**Purpose:** Unified gateway for accessing 400+ LLM models (GPT-4, Claude, Llama, etc.) for AI question answering, pattern analysis, OCR, and health insights generation.

**Documentation:** https://openrouter.ai/docs

**Base URL:** `https://openrouter.ai/api/v1`

**Authentication:** Bearer token (API key)

```typescript
// Environment variable
OPENROUTER_API_KEY=sk-or-v1-xxxxx

// Request headers
headers: {
  'Authorization': `Bearer ${process.env.OPENROUTER_API_KEY}`,
  'HTTP-Referer': 'https://procare.health',
  'X-Title': 'ProCare Diabetes Management'
}
```

**Rate Limits:**
- Free tier: 200 requests/day
- Paid tier: Unlimited (pay-per-token)
- Recommended: Cache responses in Redis for 24h for identical questions

**Models Used by ProCare:**

| Use Case | Model | Rationale |
|----------|-------|-----------|
| **Routine Questions** | `meta-llama/llama-3.1-70b-instruct` | Cost-effective ($0.52/M tokens), accurate for general diabetes questions |
| **Medical Questions** | `openai/gpt-4-turbo` | Highest accuracy ($10/M tokens), critical for medical guidance |
| **Pattern Detection** | `anthropic/claude-3.5-sonnet` | Strong reasoning ($3/M tokens), good for correlation analysis |
| **OCR (Glucometer Photos)** | `openai/gpt-4-vision-preview` | Vision capabilities ($10/M tokens), accurate number extraction |
| **OCR (Prescriptions)** | `openai/gpt-4-vision-preview` | Handles handwritten text, medical terminology |
| **Food Recognition** | `openai/gpt-4-vision-preview` | Identifies Indian foods accurately |
| **Question Classification** | `meta-llama/llama-3.1-8b-instruct` | Very cheap ($0.06/M tokens), simple classification task |

## Evolution API (WhatsApp Integration)

**Purpose:** Self-hosted WhatsApp multi-device API for patient messaging, re-engagement, and secondary channel communication.

**Documentation:**
- https://doc.evolution-api.com/
- https://evolution-api.com/postman
- GitHub: https://github.com/EvolutionAPI/evolution-api

**Base URL:** `https://whatsapp.procare.health` (self-hosted on Hetzner VPS via Coolify)

**Authentication:** API key (configured during Evolution API setup)

**Rate Limits:**
- WhatsApp Business Policy: No spam (max ~5 messages/day per patient recommended)
- Evolution API: Unlimited (self-hosted)
- ProCare Rate Limiting: Enforced by Engagement Engine (FR15: max 5/day)

**Key Endpoints Used:**

- `POST /message/sendText/{instance}` - Send text messages
- `POST /message/sendMedia/{instance}` - Send images, PDFs
- Webhook: `POST /webhooks/evolution-api` - Receive incoming messages

## Razorpay API (Payment Gateway)

**Purpose:** Subscription payments, appointment booking payments, revenue sharing calculation (FR35, Epic 13).

**Documentation:** https://razorpay.com/docs/api/

**Base URL:** `https://api.razorpay.com/v1`

**Authentication:** Basic Auth (API Key + Secret)

**Rate Limits:** 600 requests/minute

**Key Endpoints:**
- `POST /orders` - Create payment order
- `POST /subscriptions` - Create monthly subscription
- Webhook signature verification for payment events

## Lab Partner APIs (Apollo, Metropolis)

**Purpose:** Blood test appointment booking, results retrieval (FR10).

**Base URLs:**
- Apollo: `https://api.apollo247.com/partner/v1`
- Metropolis: `https://api.metropolisindia.com/partner/v1`

**Key Features:**
- Book HbA1c and other diabetes-related tests
- Home visit scheduling
- Automatic results upload to ProCare
- 2-week reminder system

## Twilio API (SMS/OTP)

**Purpose:** SMS OTP delivery for patient authentication, backup channel for critical alerts.

**Documentation:** https://www.twilio.com/docs/sms

**Base URL:** `https://api.twilio.com/2010-04-01`

**Key Features:**
- 6-digit OTP generation and delivery
- 5-minute expiry in Redis
- Fallback to email if SMS fails

## External API Summary Table

| Service | Purpose | Authentication | Rate Limits | Cost | Critical? |
|---------|---------|----------------|-------------|------|-----------|
| **OpenRouter** | AI (questions, OCR, patterns) | Bearer token | Unlimited (paid) | $0.06-$10/M tokens | ✅ Yes |
| **Evolution API** | WhatsApp messaging | API key | Unlimited (self-hosted) | Free (self-hosted) | ✅ Yes |
| **Razorpay** | Payments, subscriptions | Basic auth | 600/min | 2% + ₹3 per transaction | ✅ Yes |
| **Twilio** | SMS OTP | Basic auth | 1000/hour | $0.0079/SMS (India) | ✅ Yes |
| **Apollo/Metropolis** | Lab test booking | OAuth 2.0 | Varies | Commission-based | ⚠️ Important |
| **Google Fit** | Device data import | OAuth 2.0 | 10000/day | Free | ⚠️ Phase 2 |
| **Apple HealthKit** | Device data import | Local (no API) | N/A | Free | ⚠️ Phase 2 |

---
