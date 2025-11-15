# 14. Deployment Architecture

## 14.1 Infrastructure Overview

ProCare uses a **self-hosted infrastructure** on **Hetzner Cloud** with **Coolify** as the deployment automation platform.

### 14.1.1 Why Hetzner Cloud?

- **Cost-effective**: 60-70% cheaper than AWS/Azure for similar specs
- **Performance**: AMD EPYC processors, NVMe SSD storage
- **Location**: European data centers (can expand to India)
- **Simplicity**: No complex pricing, predictable costs
- **Reliability**: 99.9% uptime SLA

### 14.1.2 Why Coolify?

- **Self-hosted Heroku alternative**: Deploy with Git push
- **Docker-based**: All services run in containers
- **Built-in features**: Automatic SSL, backups, monitoring
- **No vendor lock-in**: Can migrate to any Docker-compatible platform
- **Open source**: Free, community-driven

## 14.2 Server Requirements

### 14.2.1 Production Server Specs

**Recommended Hetzner Cloud Server (CPX41):**
- **vCPUs**: 8 (AMD EPYC)
- **RAM**: 16 GB
- **Storage**: 240 GB NVMe SSD
- **Traffic**: 20 TB/month
- **Cost**: €24.90/month (~₹2,200/month)

**For scaling (after 10,000+ users), upgrade to CCX33:**
- **vCPUs**: 8 dedicated cores
- **RAM**: 32 GB
- **Storage**: 240 GB NVMe SSD
- **Cost**: €79.90/month (~₹7,000/month)

### 14.2.2 Staging Server Specs

**Hetzner Cloud Server (CPX21):**
- **vCPUs**: 3
- **RAM**: 4 GB
- **Storage**: 80 GB SSD
- **Cost**: €7.19/month (~₹630/month)

## 14.3 Coolify Installation

### 14.3.1 Initial Server Setup

**1. Create Hetzner Cloud Server:**

```bash
# Via Hetzner Cloud Console:
# - Create Project: "ProCare Production"
# - Location: Nuremberg (Germany) or Helsinki (Finland)
# - OS: Ubuntu 24.04 LTS
# - SSH Key: Add your public key
# - Server: CPX41
```

**2. Connect to server:**

```bash
ssh root@<server-ip>
```

**3. Install Coolify:**

```bash
curl -fsSL https://cdn.coollabs.io/coolify/install.sh | bash
```

Coolify will:
- Install Docker Engine
- Set up Coolify control panel
- Configure firewall (ports 80, 443, 8000)
- Generate SSL certificates

**4. Access Coolify:**

Open browser: `http://<server-ip>:8000`

Set up admin account (email + password).

### 14.3.2 Configure Coolify

**1. Add GitHub Integration:**

- Navigate to **Settings → Git Providers**
- Connect GitHub account
- Authorize Coolify to access your repository

**2. Add Docker Registry (optional):**

- For private images, configure Docker Hub or GitHub Container Registry

**3. Configure Backups:**

- **Storage Provider**: Hetzner Object Storage (S3-compatible)
- **Backup Schedule**: Daily at 2:00 AM UTC
- **Retention**: 7 daily, 4 weekly, 3 monthly backups

## 14.4 Environment Configuration

### 14.4.1 Staging Environment

**Domain**: `staging.procare.health`

**Environment Variables** (configured in Coolify):

```bash
NODE_ENV=staging

# Database
DATABASE_URL=postgresql://postgres:<password>@postgres:5432/procare_staging

# Redis
REDIS_URL=redis://redis:6379/0

# RabbitMQ
RABBITMQ_URL=amqp://procare:<password>@rabbitmq:5672

# MinIO (Hetzner Object Storage)
MINIO_ENDPOINT=fsn1.your-objectstorage.com
MINIO_PORT=443
MINIO_ACCESS_KEY=<access-key>
MINIO_SECRET_KEY=<secret-key>
MINIO_USE_SSL=true

# JWT
JWT_SECRET=<staging-secret>
JWT_EXPIRES_IN=7d

# OpenRouter API
OPENROUTER_API_KEY=<staging-key>
OPENROUTER_APP_NAME=ProCare Staging
OPENROUTER_APP_URL=https://staging.procare.health

# Evolution API (WhatsApp)
EVOLUTION_API_URL=https://whatsapp.staging.procare.health
EVOLUTION_API_KEY=<staging-key>
EVOLUTION_INSTANCE_NAME=procare-staging

# Razorpay (Test Mode)
RAZORPAY_KEY_ID=rzp_test_...
RAZORPAY_KEY_SECRET=...

# BunnyCDN
BUNNYCDN_STORAGE_ZONE=procare-staging
BUNNYCDN_API_KEY=...
BUNNYCDN_HOSTNAME=staging-cdn.procare.health

# Sentry (Error Tracking)
SENTRY_DSN=https://...
SENTRY_ENVIRONMENT=staging
```

### 14.4.2 Production Environment

**Domain**: `procare.health`

**Environment Variables** (same structure as staging, but with production values):

```bash
NODE_ENV=production

# All credentials use production secrets
DATABASE_URL=postgresql://postgres:<prod-password>@postgres:5432/procare
RAZORPAY_KEY_ID=rzp_live_...
EVOLUTION_INSTANCE_NAME=procare-prod
# etc.
```

## 14.5 Deployment Strategy

### 14.5.1 Services Deployment Order

1. **Infrastructure Services** (PostgreSQL, Redis, RabbitMQ, MinIO)
2. **API Gateway**
3. **Agent Services** (all 7 agents)
4. **Web Dashboards** (doctor-dashboard, caregiver-dashboard)
5. **Mobile App** (EAS Build for App Stores)

### 14.5.2 Deploy Infrastructure Services

**1. PostgreSQL (TimescaleDB):**

In Coolify:
- **Resource Type**: Docker Compose
- **Image**: `timescale/timescaledb:latest-pg16`
- **Persistent Volume**: `/var/lib/postgresql/data` → 100 GB
- **Environment Variables**:
  ```
  POSTGRES_USER=postgres
  POSTGRES_PASSWORD=<secure-password>
  POSTGRES_DB=procare
  ```

**2. Redis:**

- **Image**: `redis:7-alpine`
- **Persistent Volume**: `/data` → 10 GB
- **Configuration**: Append-only file (AOF) enabled

**3. RabbitMQ:**

- **Image**: `rabbitmq:3.13-management-alpine`
- **Persistent Volume**: `/var/lib/rabbitmq` → 20 GB
- **Environment Variables**:
  ```
  RABBITMQ_DEFAULT_USER=procare
  RABBITMQ_DEFAULT_PASS=<secure-password>
  ```

**4. MinIO (Optional - Use Hetzner Object Storage instead):**

Or configure Hetzner Object Storage for S3-compatible storage.

### 14.5.3 Deploy API Gateway

**In Coolify:**

1. **Create New Resource**: Git Repository
2. **Repository**: `https://github.com/your-org/procare`
3. **Branch**: `main` (production) or `staging` (staging)
4. **Build Pack**: Dockerfile
5. **Dockerfile Path**: `apps/api-gateway/Dockerfile`
6. **Port**: 3000
7. **Domain**: `api.procare.health`
8. **Environment Variables**: Copy from production .env
9. **Health Check**: `/health` endpoint

**Build Configuration:**

- **Build Command**: `pnpm install && pnpm --filter=api-gateway build`
- **Start Command**: `pnpm --filter=api-gateway start`

**Auto-deploy on Git Push:**
- Enable "Auto Deploy" in Coolify
- Every push to `main` triggers deployment

### 14.5.4 Deploy Agent Services

**For each agent service (engagement-engine, clinical-monitor, etc.):**

1. **Create New Resource**: Git Repository
2. **Dockerfile Path**: `apps/agent-services/<service-name>/Dockerfile`
3. **Environment Variables**: Same as API Gateway
4. **No public domain** (internal services)

Alternatively, use **Docker Compose** to deploy all agents together:

```yaml
# coolify-agents.yml
version: '3.8'

services:
  engagement-engine:
    build:
      context: .
      dockerfile: apps/agent-services/engagement-engine/Dockerfile
    environment:
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      RABBITMQ_URL: ${RABBITMQ_URL}
    restart: unless-stopped

  clinical-monitor:
    build:
      context: .
      dockerfile: apps/agent-services/clinical-monitor/Dockerfile
    environment:
      DATABASE_URL: ${DATABASE_URL}
      RABBITMQ_URL: ${RABBITMQ_URL}
      OPENROUTER_API_KEY: ${OPENROUTER_API_KEY}
    restart: unless-stopped

  # ... (other 5 agents)
```

### 14.5.5 Deploy Web Dashboards

**Doctor Dashboard:**

1. **Create New Resource**: Git Repository
2. **Build Pack**: Nixpacks (auto-detects Next.js)
3. **Dockerfile Path** (if using Docker): `apps/doctor-dashboard/Dockerfile`
4. **Domain**: `doctor.procare.health`
5. **Environment Variables**:
   ```
   NEXTAUTH_URL=https://doctor.procare.health
   NEXTAUTH_SECRET=<secret>
   NEXT_PUBLIC_API_URL=https://api.procare.health
   ```

**Caregiver Dashboard:**

Same process, domain: `caregiver.procare.health`

### 14.5.6 Deploy Mobile App

**Mobile apps are deployed via EAS Build to App Stores:**

**Android (Google Play Store):**

```bash
cd apps/mobile

# Login to Expo
eas login

# Build production APK
eas build --platform android --profile production

# Submit to Google Play
eas submit --platform android
```

**iOS (Apple App Store):**

```bash
# Build production IPA
eas build --platform ios --profile production

# Submit to App Store
eas submit --platform ios
```

**EAS Configuration (`eas.json`):**

```json
{
  "build": {
    "production": {
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.procare.health"
      },
      "android": {
        "buildType": "apk",
        "gradleCommand": ":app:assembleRelease"
      },
      "ios": {
        "bundleIdentifier": "com.procare.app",
        "buildConfiguration": "Release"
      }
    },
    "staging": {
      "env": {
        "EXPO_PUBLIC_API_URL": "https://api.staging.procare.health"
      }
    }
  }
}
```

## 14.6 CI/CD Pipeline

### 14.6.1 GitHub Actions Workflow

**File**: `.github/workflows/deploy.yml`

```yaml
name: Deploy to Production

on:
  push:
    branches:
      - main

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Install pnpm
        run: npm install -g pnpm@9.15.0

      - name: Install dependencies
        run: pnpm install

      - name: Run tests
        run: pnpm test

      - name: Type check
        run: pnpm turbo run type-check

      - name: Lint
        run: pnpm lint

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Trigger Coolify Deployment
        run: |
          curl -X POST "${{ secrets.COOLIFY_WEBHOOK_URL }}"
```

**Coolify Webhook Setup:**

1. In Coolify, go to each service → **Webhooks**
2. Copy webhook URL
3. Add to GitHub Secrets as `COOLIFY_WEBHOOK_URL`

**Result**: Every push to `main` → runs tests → triggers Coolify deployment

### 14.6.2 Deployment Rollback

**If deployment fails:**

```bash
# Via Coolify UI:
# 1. Go to service
# 2. Click "Deployments"
# 3. Select previous successful deployment
# 4. Click "Redeploy"

# Or via CLI:
curl -X POST "https://coolify.procare.health/api/v1/deploy/rollback" \
  -H "Authorization: Bearer <api-token>" \
  -d '{"service_id": "api-gateway"}'
```

## 14.7 Database Management

### 14.7.1 Migrations in Production

**Apply migrations during deployment:**

```bash
# In API Gateway Dockerfile, add migration step:
CMD ["sh", "-c", "pnpm prisma migrate deploy && pnpm start"]
```

This runs pending migrations before starting the API Gateway.

### 14.7.2 Database Backups

**Automated Backups via Coolify:**

- **Schedule**: Daily at 2:00 AM UTC
- **Storage**: Hetzner Object Storage (S3-compatible)
- **Retention**: 7 daily, 4 weekly, 3 monthly
- **Encryption**: AES-256

**Manual Backup:**

```bash
# SSH into server
ssh root@<server-ip>

# Backup database
docker exec procare-postgres-1 pg_dump -U postgres procare > backup-$(date +%Y%m%d).sql

# Upload to object storage
s3cmd put backup-*.sql s3://procare-backups/
```

### 14.7.3 Database Restore

```bash
# Download backup
s3cmd get s3://procare-backups/backup-20250114.sql

# Restore to database
docker exec -i procare-postgres-1 psql -U postgres procare < backup-20250114.sql
```

## 14.8 Domain & SSL Configuration

### 14.8.1 Domain Setup

**DNS Records (configure at domain registrar):**

```
Type    Name        Value                   TTL
A       @           <server-ip>             3600
A       api         <server-ip>             3600
A       doctor      <server-ip>             3600
A       caregiver   <server-ip>             3600
A       staging     <staging-server-ip>     3600
CNAME   www         procare.health          3600
```

### 14.8.2 SSL Certificates

**Coolify automatically provisions SSL via Let's Encrypt:**

- Certificates auto-renew every 60 days
- Forced HTTPS redirect enabled
- HSTS headers configured

**Manual SSL renewal (if needed):**

```bash
# Via Coolify UI:
# Service → SSL → Renew Certificate
```

## 14.9 Scaling Strategy

### 14.9.1 Vertical Scaling (0-10,000 users)

**Upgrade server specs:**

```
Current: CPX41 (8 vCPU, 16 GB RAM)
  ↓
Upgrade: CCX33 (8 dedicated cores, 32 GB RAM)
```

### 14.9.2 Horizontal Scaling (10,000+ users)

**Add load balancer + multiple app servers:**

1. **Create 2+ app servers** (CPX41 each)
2. **Create load balancer** (Hetzner Cloud Load Balancer: €6.90/month)
3. **Configure health checks**: `/health` endpoint
4. **Distribute traffic**: Round-robin or least connections

**Architecture with Load Balancer:**

```
                    [Load Balancer]
                           |
      ┌────────────────────┼────────────────────┐
      │                    │                    │
 [App Server 1]      [App Server 2]      [App Server 3]
      │                    │                    │
      └────────────────────┼────────────────────┘
                           |
                  [Shared Database + Redis]
```

### 14.9.3 Database Scaling

**PostgreSQL Read Replicas:**

```
[Primary Database]  ←─ Writes
      │
      ├─ [Read Replica 1]  ←─ Reads
      └─ [Read Replica 2]  ←─ Reads
```

**Use Prisma read replicas:**

```typescript
// packages/database/src/index.ts
export const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL, // Primary (writes)
    },
  },
});

// Configure read replica pool
export const prismaRead = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_READ_REPLICA_URL,
    },
  },
});
```

## 14.10 Monitoring & Alerts

### 14.10.1 Coolify Built-in Monitoring

- **Resource usage**: CPU, RAM, disk, network
- **Service health**: Automatic health checks
- **Logs**: Real-time log streaming
- **Alerts**: Email/Discord/Slack notifications

### 14.10.2 Configure Alerts

**In Coolify:**

1. **Settings → Notifications**
2. **Add Email/Discord webhook**
3. **Alert Triggers**:
   - Service down (health check fails 3x)
   - CPU > 80% for 5 minutes
   - RAM > 90% for 5 minutes
   - Disk > 85%

## 14.11 Deployment Checklist

**Before deploying to production:**

- [ ] All tests passing (`pnpm test`)
- [ ] Type checking clean (`pnpm turbo run type-check`)
- [ ] Linting clean (`pnpm lint`)
- [ ] Environment variables configured in Coolify
- [ ] Database migrations tested in staging
- [ ] SSL certificates provisioned
- [ ] Domain DNS configured
- [ ] Backup strategy configured
- [ ] Monitoring alerts enabled
- [ ] Load testing completed (optional)
- [ ] Security headers configured (helmet.js)
- [ ] Rate limiting enabled
- [ ] CORS configured correctly
- [ ] API keys rotated (no test keys in production)
- [ ] Sentry error tracking enabled
- [ ] Mobile app submitted to App Stores
