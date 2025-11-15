# 19. Monitoring and Observability

## 19.1 Monitoring Stack

**ProCare Monitoring Architecture:**

- **Prometheus**: Metrics collection
- **Grafana**: Visualization dashboards
- **Loki**: Log aggregation
- **Sentry**: Error tracking
- **Uptime Kuma**: Uptime monitoring

## 19.2 Metrics Collection

### 19.2.1 Prometheus Metrics (Express)

```typescript
// apps/api-gateway/src/lib/metrics.ts
import promClient from 'prom-client';

// Register default metrics (CPU, memory, etc.)
promClient.collectDefaultMetrics({ timeout: 5000 });

// Custom metrics
export const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_ms',
  help: 'Duration of HTTP requests in ms',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [10, 50, 100, 200, 500, 1000, 2000, 5000],
});

export const glucoseReadingsTotal = new promClient.Counter({
  name: 'glucose_readings_total',
  help: 'Total number of glucose readings logged',
  labelNames: ['patient_id', 'source'],
});

export const emergencyEventsTotal = new promClient.Counter({
  name: 'emergency_events_total',
  help: 'Total number of emergency events detected',
  labelNames: ['type'],
});

export const activePatients = new promClient.Gauge({
  name: 'active_patients',
  help: 'Number of active patients (logged in last 7 days)',
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

### 19.2.2 Middleware for Request Tracking

```typescript
// apps/api-gateway/src/middleware/metricsMiddleware.ts
import { Request, Response, NextFunction } from 'express';
import { httpRequestDuration } from '../lib/metrics';

export function metricsMiddleware(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;

    httpRequestDuration
      .labels(req.method, req.route?.path || req.path, res.statusCode.toString())
      .observe(duration);
  });

  next();
}
```

## 19.3 Logging Strategy

### 19.3.1 Structured Logging

```typescript
// packages/shared/src/lib/logger.ts
import winston from 'winston';
import LokiTransport from 'winston-loki';

export const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: process.env.SERVICE_NAME || 'procare',
    environment: process.env.NODE_ENV,
  },
  transports: [
    // Console logging
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      ),
    }),

    // Loki logging (production only)
    ...(process.env.NODE_ENV === 'production'
      ? [
          new LokiTransport({
            host: process.env.LOKI_URL || 'http://localhost:3100',
            labels: { app: 'procare' },
            json: true,
          }),
        ]
      : []),
  ],
});

// Usage
logger.info('Patient registered', { patientId: '123', phoneNumber: '+91...' });
logger.error('Failed to send WhatsApp message', { error: err.message, patientId: '123' });
```

### 19.3.2 Request Logging

```typescript
// apps/api-gateway/src/middleware/requestLogger.ts
import { logger } from 'shared';

export function requestLogger(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();

  res.on('finish', () => {
    logger.info('HTTP Request', {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: Date.now() - start,
      userId: req.user?.userId,
    });
  });

  next();
}
```

## 19.4 Grafana Dashboards

### 19.4.1 API Gateway Dashboard

**Metrics to track:**

- Request rate (req/s)
- Response time (p50, p95, p99)
- Error rate (%)
- Active connections
- Memory usage
- CPU usage

**PromQL Queries:**

```promql
# Request rate
rate(http_request_duration_ms_count[5m])

# P95 response time
histogram_quantile(0.95, rate(http_request_duration_ms_bucket[5m]))

# Error rate
rate(http_request_duration_ms_count{status_code=~"5.."}[5m]) / rate(http_request_duration_ms_count[5m])
```

### 19.4.2 Application Metrics Dashboard

```promql
# Glucose readings per minute
rate(glucose_readings_total[1m]) * 60

# Emergency events per hour
rate(emergency_events_total[1h]) * 3600

# Active patients
active_patients
```

## 19.5 Alerting Rules

### 19.5.1 Prometheus Alerts

```yaml
# prometheus/alerts.yml
groups:
  - name: procare-alerts
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: rate(http_request_duration_ms_count{status_code=~"5.."}[5m]) / rate(http_request_duration_ms_count[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} for the last 5 minutes"

      # High response time
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_ms_bucket[5m])) > 1000
        for: 10m
        labels:
          severity: warning
        annotations:
          summary: "High API response time"
          description: "P95 response time is {{ $value }}ms"

      # Service down
      - alert: ServiceDown
        expr: up == 0
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Service {{ $labels.instance }} is down"

      # High memory usage
      - alert: HighMemoryUsage
        expr: process_resident_memory_bytes / 1024 / 1024 > 1024
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High memory usage"
          description: "Memory usage is {{ $value }}MB"
```

### 19.5.2 Alert Notifications

Configure Prometheus Alertmanager to send alerts to:

- **Discord** (dev team channel)
- **Email** (for critical alerts)
- **PagerDuty** (for production emergencies)

## 19.6 Health Checks

```typescript
// apps/api-gateway/src/routes/health.ts
import express from 'express';
import { prisma } from 'database';
import { redis } from '../lib/redis';

const router = express.Router();

router.get('/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    services: {
      database: 'unknown',
      redis: 'unknown',
      rabbitmq: 'unknown',
    },
  };

  try {
    // Check database connection
    await prisma.$queryRaw`SELECT 1`;
    health.services.database = 'healthy';
  } catch (error) {
    health.services.database = 'unhealthy';
    health.status = 'unhealthy';
  }

  try {
    // Check Redis connection
    await redis.ping();
    health.services.redis = 'healthy';
  } catch (error) {
    health.services.redis = 'unhealthy';
    health.status = 'unhealthy';
  }

  const statusCode = health.status === 'healthy' ? 200 : 503;
  res.status(statusCode).json(health);
});

export default router;
```

## 19.7 Uptime Monitoring

**Uptime Kuma Configuration:**

- **API Gateway**: https://api.procare.health/health (check every 60s)
- **Doctor Dashboard**: https://doctor.procare.health (check every 60s)
- **Caregiver Dashboard**: https://caregiver.procare.health (check every 60s)
- **WhatsApp API (Evolution)**: https://whatsapp.procare.health/health (check every 120s)

**Notifications**: Discord webhook + Email for downtime alerts

---
