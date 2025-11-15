# Tech Stack

This is the **definitive technology selection** for ProCare. All development must use these exact technologies and versions.

## Technology Stack Table

| Category | Technology | Version | Purpose | Rationale |
|----------|------------|---------|---------|-----------|
| **Hosting Provider** | Hetzner Cloud | Latest | VPS infrastructure hosting | Cost-effective (€120/month vs AWS $1,500), GDPR-compliant EU data centers, high performance |
| **Deployment Platform** | Coolify | v4.x | Docker-based deployment automation | Open-source Vercel alternative, Git-based CI/CD, automatic SSL, one-click database deployment |
| **Monorepo Tool** | Turborepo | 2.x | Monorepo build system | Industry standard for Next.js + React Native (Vercel-backed), remote caching, incremental builds |
| **Frontend Language** | TypeScript | 5.3+ | Type-safe JavaScript | End-to-end type safety with tRPC, reduced runtime errors, better DX |
| **Mobile Framework** | React Native + Expo | 51+ | Cross-platform iOS/Android | Code sharing with web, fastest mobile development, over-the-air updates, offline-first support |
| **Web Framework** | Next.js | 14.x (App Router) | React framework for dashboards | Server components, edge caching, excellent DX, Coolify native support |
| **UI Library (Web)** | ShadCN + Radix UI | Latest | Accessible component library | Copy-paste components, TailwindCSS integration, WCAG AA compliance, customizable |
| **UI Library (Mobile)** | React Native Reusables | Latest | ShadCN for React Native | Consistent design with web, NativeWind support, accessible mobile components |
| **CSS Framework** | TailwindCSS + NativeWind | 3.4+ | Utility-first styling | Consistent styling across web/mobile, small bundle size, excellent DX |
| **State Management (Web)** | Zustand + TanStack Query | Latest | Client state + server cache | Lightweight (Zustand), optimistic updates (TanStack Query), tRPC integration |
| **State Management (Mobile)** | Zustand + WatermelonDB | Latest | App state + offline database | Simple API (Zustand), offline-first (WatermelonDB), fast queries, lazy loading |
| **Backend Language** | Node.js (TypeScript) | 20 LTS | Microservices runtime | Async I/O for event-driven architecture, TypeScript code sharing, rich ecosystem |
| **Backend Framework** | Express.js | 4.x | HTTP server for microservices | Minimal, flexible, well-documented, middleware ecosystem |
| **API Style** | tRPC | 10.x | End-to-end type-safe RPC | No code generation, type-safe from DB to UI, excellent DX, monorepo-friendly |
| **Database (Primary)** | PostgreSQL (self-hosted) | 16.x | Relational database on VPS | ACID compliance, TimescaleDB support, mature, Coolify one-click deployment |
| **Database (Time-Series)** | TimescaleDB | 2.x | PostgreSQL extension for glucose data | Optimized time-series queries, automatic partitioning, compression, same DB as primary |
| **Database (Mobile Offline)** | WatermelonDB | 0.27+ | React Native offline database | Built on SQLite, lazy loading, 10,000+ records performance, background sync |
| **ORM** | Prisma | 5.x | Type-safe database client | Schema-first, migrations, TypeScript integration, excellent DX |
| **Cache** | Redis | 7.x | Session store + real-time cache | In-memory speed, pub/sub for real-time, self-hostable |
| **Message Queue** | RabbitMQ | 3.13+ | Event-driven agent communication | Reliable message delivery, routing flexibility, proven at scale |
| **File Storage** | MinIO | Latest | S3-compatible object storage | Self-hosted, S3 API compatible, cost-effective, no egress fees |
| **CDN** | BunnyCDN | Latest | Global edge caching | €1/TB (vs CloudFront $85/TB), 114 PoPs, India edge locations |
| **WhatsApp Integration** | Evolution API | Latest | Self-hosted WhatsApp multi-device API | No WhatsApp Business approval wait, unlimited free messaging, multi-device, webhooks |
| **AI Gateway** | OpenRouter API | Latest | Multi-LLM access (GPT-4, Claude, Llama) | 400+ models, cost optimization, regional compliance, no vendor lock-in |
| **Authentication** | NextAuth.js + JWT | 5.x | Auth for doctors/caregivers | OAuth providers, session management, Edge-compatible, customizable |
| **OTP Provider** | Twilio | Latest | Patient SMS OTP | Reliable India delivery, competitive pricing, fall back to email |
| **Frontend Testing** | Vitest + Testing Library | Latest | Unit/integration tests | Fast (Vite-powered), Jest-compatible API, React/React Native support |
| **Backend Testing** | Vitest + Supertest | Latest | API integration tests | TypeScript native, fast, HTTP request testing |
| **E2E Testing** | Playwright | Latest | End-to-end web tests | Multi-browser, auto-wait, debugging tools, CI-friendly |
| **Mobile E2E Testing** | Maestro | Latest | React Native E2E tests | Flow-based, fast, cloud test execution, CI integration |
| **Linting** | ESLint + Prettier | Latest | Code quality | Consistent formatting, catch errors, monorepo support |
| **Type Checking** | TypeScript Compiler | 5.3+ | Static type checking | Catch errors at compile-time, IDE autocomplete |
| **CI/CD** | Coolify Built-in + GitHub Actions | Latest | Automated deployment | Git push → auto-deploy (Coolify), test automation (GitHub Actions) |
| **Infrastructure as Code** | Docker Compose | Latest | Service definitions | Coolify-compatible, reproducible environments, local development |
| **Monitoring** | Grafana + Prometheus | Latest | Metrics + dashboards | Self-hosted observability, alerting, custom dashboards |
| **Logging** | Loki + Promtail | Latest | Centralized logging | Integrates with Grafana, log aggregation, query language |
| **Error Tracking** | Sentry | Latest | Error monitoring + alerting | React/React Native/Node.js SDKs, release tracking, performance monitoring |
| **Tracing** | OpenTelemetry + Tempo | Latest | Distributed tracing | CNCF standard, vendor-neutral, microservices debugging |
| **Backup (Tier 1)** | Hetzner Storage Box + pg_dump | Latest | Fast local backups | 6-hourly backups, 7-day retention, <30min restore time |
| **Backup (Tier 2)** | AWS S3 + Restic | Latest | Off-site disaster recovery | Daily encrypted backups, 30-day retention, geographic redundancy |
| **Backup (Tier 3)** | AWS S3 Glacier Deep Archive | Latest | Long-term compliance archive | Monthly backups, 7-year retention, $1/month for 100GB |
| **Backup (PITR)** | WAL-G + AWS S3 | Latest | Point-in-time recovery | Continuous WAL archiving, restore to any timestamp, critical data protection |
| **Reverse Proxy** | Traefik | 3.x | Auto-configured by Coolify | Automatic SSL (Let's Encrypt), Docker service discovery, load balancing |
| **Payment Gateway** | Razorpay | Latest | Subscription payments | India-focused, UPI/cards/wallets, subscription management, webhooks |

---
