# 16. Testing Strategy

## 16.1 Testing Pyramid

ProCare follows the testing pyramid approach:

```
           /\
          /E2E\          ~5% (End-to-End Tests)
         /------\
        /  INT   \       ~15% (Integration Tests)
       /----------\
      /   UNIT     \     ~80% (Unit Tests)
     /--------------\
```

## 16.2 Unit Testing

**Framework**: **Vitest** (faster than Jest, compatible with existing Jest tests)

### 16.2.1 Backend Unit Tests

```typescript
// apps/api-gateway/src/lib/cache.test.ts
import { describe, it, expect, beforeEach, vi } from 'vitest';
import { getCached, invalidateCache } from './cache';
import { redis } from './redis';

vi.mock('./redis');

describe('Cache utilities', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('should return cached value if exists', async () => {
    const mockData = { foo: 'bar' };
    vi.mocked(redis.get).mockResolvedValue(JSON.stringify(mockData));

    const fetchFn = vi.fn();
    const result = await getCached('test-key', fetchFn);

    expect(result).toEqual(mockData);
    expect(fetchFn).not.toHaveBeenCalled();
    expect(redis.get).toHaveBeenCalledWith('test-key');
  });

  it('should fetch and cache if not in cache', async () => {
    const mockData = { foo: 'bar' };
    vi.mocked(redis.get).mockResolvedValue(null);
    const fetchFn = vi.fn().mockResolvedValue(mockData);

    const result = await getCached('test-key', fetchFn, 3600);

    expect(result).toEqual(mockData);
    expect(fetchFn).toHaveBeenCalled();
    expect(redis.setex).toHaveBeenCalledWith('test-key', 3600, JSON.stringify(mockData));
  });

  it('should invalidate cache by pattern', async () => {
    vi.mocked(redis.keys).mockResolvedValue(['key1', 'key2', 'key3']);

    await invalidateCache('test:*');

    expect(redis.keys).toHaveBeenCalledWith('test:*');
    expect(redis.del).toHaveBeenCalledWith('key1', 'key2', 'key3');
  });
});
```

**tRPC Router Testing:**

```typescript
// apps/api-gateway/src/trpc/routers/glucose.test.ts
import { describe, it, expect, beforeEach } from 'vitest';
import { appRouter } from '../index';
import { createContext } from '../context';
import { prisma } from 'database';

describe('Glucose Router', () => {
  const caller = appRouter.createCaller(await createContext({
    user: { userId: 'test-patient-id', role: 'PATIENT' },
  }));

  beforeEach(async () => {
    // Clean up test data
    await prisma.glucoseReading.deleteMany({ where: { patientId: 'test-patient-id' } });
  });

  it('should log glucose reading', async () => {
    const input = {
      value: 120,
      state: 'MORNING_FASTING' as const,
      recordedAt: new Date(),
      source: 'MANUAL' as const,
    };

    const result = await caller.glucose.logReading(input);

    expect(result.value).toBe(120);
    expect(result.state).toBe('MORNING_FASTING');
    expect(result.patientId).toBe('test-patient-id');
  });

  it('should reject invalid glucose value', async () => {
    const input = {
      value: 1000, // Invalid
      state: 'MORNING_FASTING' as const,
      recordedAt: new Date(),
      source: 'MANUAL' as const,
    };

    await expect(caller.glucose.logReading(input)).rejects.toThrow();
  });

  it('should get glucose history', async () => {
    // Create test data
    await prisma.glucoseReading.createMany({
      data: [
        { patientId: 'test-patient-id', value: 110, state: 'MORNING_FASTING', recordedAt: new Date('2025-01-10'), source: 'MANUAL' },
        { patientId: 'test-patient-id', value: 140, state: 'AFTER_BREAKFAST', recordedAt: new Date('2025-01-11'), source: 'MANUAL' },
        { patientId: 'test-patient-id', value: 95, state: 'BEDTIME', recordedAt: new Date('2025-01-12'), source: 'MANUAL' },
      ],
    });

    const result = await caller.glucose.getHistory({ limit: 10 });

    expect(result.length).toBe(3);
    expect(result[0].recordedAt.getTime()).toBeGreaterThan(result[1].recordedAt.getTime()); // Descending order
  });
});
```

### 16.2.2 Frontend Unit Tests (React Components)

```tsx
// apps/doctor-dashboard/src/components/PatientCard.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import PatientCard from './PatientCard';

describe('PatientCard', () => {
  const mockPatient = {
    id: '1',
    name: 'John Doe',
    diabetesType: 'TYPE_2',
    managementMode: 'ACTIVE_MANAGEMENT',
    latestGlucose: 125,
  };

  it('should render patient information', () => {
    render(<PatientCard patient={mockPatient} />);

    expect(screen.getByText('John Doe')).toBeInTheDocument();
    expect(screen.getByText('Type 2')).toBeInTheDocument();
    expect(screen.getByText('125 mg/dL')).toBeInTheDocument();
  });

  it('should show emergency indicator for high glucose', () => {
    render(<PatientCard patient={{ ...mockPatient, latestGlucose: 350 }} />);

    expect(screen.getByTestId('emergency-indicator')).toBeInTheDocument();
  });
});
```

**Mobile Component Testing (React Native):**

```tsx
// apps/mobile/src/components/GlucoseChart.test.tsx
import { describe, it, expect } from 'vitest';
import { render } from '@testing-library/react-native';
import GlucoseChart from './GlucoseChart';

describe('GlucoseChart', () => {
  const mockData = [
    { date: '2025-01-10', value: 110 },
    { date: '2025-01-11', value: 140 },
    { date: '2025-01-12', value: 95 },
  ];

  it('should render chart with data', () => {
    const { getByTestId } = render(<GlucoseChart data={mockData} />);

    expect(getByTestId('glucose-chart')).toBeTruthy();
  });

  it('should show empty state when no data', () => {
    const { getByText } = render(<GlucoseChart data={[]} />);

    expect(getByText('No glucose data available')).toBeTruthy();
  });
});
```

## 16.3 Integration Testing

**Test Database Setup:**

```typescript
// apps/api-gateway/src/test/setup.ts
import { PrismaClient } from '@prisma/client';
import { beforeAll, afterAll, beforeEach } from 'vitest';

const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_TEST_URL, // Separate test database
    },
  },
});

beforeAll(async () => {
  // Run migrations on test database
  await prisma.$executeRaw`CREATE EXTENSION IF NOT EXISTS timescaledb;`;
});

beforeEach(async () => {
  // Clean all tables before each test
  await prisma.$transaction([
    prisma.glucoseReading.deleteMany(),
    prisma.meal.deleteMany(),
    prisma.medication.deleteMany(),
    prisma.patient.deleteMany(),
  ]);
});

afterAll(async () => {
  await prisma.$disconnect();
});
```

**Integration Test Example:**

```typescript
// apps/api-gateway/src/integration/emergency-flow.test.ts
import { describe, it, expect } from 'vitest';
import { appRouter } from '../trpc';
import { prisma } from 'database';
import { publishEvent } from 'shared';

describe('Emergency Glucose Flow', () => {
  it('should trigger emergency when glucose is critically low', async () => {
    // Create test patient
    const patient = await prisma.patient.create({
      data: {
        phoneNumber: '+919876543210',
        name: 'Test Patient',
        managementMode: 'ACTIVE_MANAGEMENT',
      },
    });

    const caller = appRouter.createCaller({ user: { userId: patient.id, role: 'PATIENT' } });

    // Mock RabbitMQ event listener
    const eventsReceived: any[] = [];
    publishEvent = vi.fn().mockImplementation((routingKey, data) => {
      eventsReceived.push({ routingKey, data });
    });

    // Log critically low glucose reading
    await caller.glucose.logReading({
      value: 55, // Critical
      state: 'RANDOM',
      recordedAt: new Date(),
      source: 'MANUAL',
    });

    // Verify emergency event was published
    expect(eventsReceived).toContainEqual(
      expect.objectContaining({
        routingKey: 'clinical.glucose.check',
        data: expect.objectContaining({
          type: 'HYPOGLYCEMIA',
        }),
      })
    );

    // Verify emergency event was created in database
    const emergencyEvent = await prisma.emergencyEvent.findFirst({
      where: { patientId: patient.id },
    });

    expect(emergencyEvent).toBeTruthy();
    expect(emergencyEvent?.type).toBe('HYPOGLYCEMIA');
  });
});
```

## 16.4 End-to-End Testing

**Framework**: **Playwright** (for web dashboards)

```typescript
// apps/doctor-dashboard/e2e/patient-management.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Doctor Dashboard - Patient Management', () => {
  test.beforeEach(async ({ page }) => {
    // Login as doctor
    await page.goto('http://localhost:3001/login');
    await page.fill('input[name="email"]', 'doctor@test.com');
    await page.fill('input[name="password"]', 'test123');
    await page.click('button[type="submit"]');
    await expect(page).toHaveURL('/dashboard');
  });

  test('should view patient list', async ({ page }) => {
    await page.goto('/patients');

    await expect(page.locator('h1')).toContainText('My Patients');
    await expect(page.locator('[data-testid="patient-card"]')).toHaveCount(5);
  });

  test('should view patient details and glucose history', async ({ page }) => {
    await page.goto('/patients');
    await page.click('[data-testid="patient-card"]:first-child');

    await expect(page).toHaveURL(/\/patients\/.+/);
    await expect(page.locator('[data-testid="glucose-chart"]')).toBeVisible();
    await expect(page.locator('[data-testid="glucose-table"]')).toBeVisible();
  });

  test('should answer pooled question', async ({ page }) => {
    await page.goto('/pooled-questions');

    const questionCard = page.locator('[data-testid="question-card"]').first();
    await questionCard.click();

    await page.fill('textarea[name="answer"]', 'This is a test answer from the doctor.');
    await page.click('button[type="submit"]');

    await expect(page.locator('text=Answer submitted successfully')).toBeVisible();
  });
});
```

**Mobile E2E Testing (Detox for React Native):**

```typescript
// apps/mobile/e2e/glucose-logging.e2e.ts
import { device, element, by, expect as detoxExpect } from 'detox';

describe('Glucose Logging Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });

  beforeEach(async () => {
    await device.reloadReactNative();
  });

  it('should log glucose reading', async () => {
    // Navigate to glucose logging screen
    await element(by.id('tab-glucose')).tap();
    await element(by.id('log-reading-button')).tap();

    // Fill in glucose value
    await element(by.id('glucose-value-input')).typeText('120');

    // Select glucose state
    await element(by.id('glucose-state-picker')).tap();
    await element(by.text('Morning Fasting')).tap();

    // Submit
    await element(by.id('submit-button')).tap();

    // Verify success message
    await detoxExpect(element(by.text('Glucose reading logged successfully'))).toBeVisible();
  });

  it('should show emergency alert for low glucose', async () => {
    await element(by.id('tab-glucose')).tap();
    await element(by.id('log-reading-button')).tap();

    await element(by.id('glucose-value-input')).typeText('55');
    await element(by.id('glucose-state-picker')).tap();
    await element(by.text('Random')).tap();
    await element(by.id('submit-button')).tap();

    // Verify emergency alert appears
    await detoxExpect(element(by.id('emergency-alert'))).toBeVisible();
    await detoxExpect(element(by.text('Your glucose is critically low'))).toBeVisible();
  });
});
```

## 16.5 Agent Service Testing

**Event Handler Testing:**

```typescript
// apps/agent-services/clinical-monitor/src/handlers/glucoseCheck.test.ts
import { describe, it, expect, vi } from 'vitest';
import { handleGlucoseCheck } from './glucoseCheck';
import { prisma } from 'database';
import { publishEvent } from 'shared';

vi.mock('shared');
vi.mock('openrouter-client');

describe('Clinical Monitor - Glucose Check', () => {
  it('should create emergency event for hypoglycemia', async () => {
    const event = {
      patientId: 'test-patient-id',
      reading: { id: 'reading-id', value: 60 },
      type: 'HYPOGLYCEMIA',
    };

    await handleGlucoseCheck(event);

    // Verify emergency event created
    const emergencyEvent = await prisma.emergencyEvent.findFirst({
      where: { patientId: 'test-patient-id', type: 'HYPOGLYCEMIA' },
    });

    expect(emergencyEvent).toBeTruthy();

    // Verify notification event published
    expect(publishEvent).toHaveBeenCalledWith(
      'patient.emergency.detected',
      expect.objectContaining({
        patientId: 'test-patient-id',
        type: 'HYPOGLYCEMIA',
      })
    );
  });

  it('should generate AI instructions for emergency', async () => {
    const mockOpenRouter = await import('openrouter-client');
    vi.mocked(mockOpenRouter.generateText).mockResolvedValue('Eat 15g of fast-acting carbs immediately...');

    const event = {
      patientId: 'test-patient-id',
      reading: { id: 'reading-id', value: 60 },
      type: 'HYPOGLYCEMIA',
    };

    await handleGlucoseCheck(event);

    expect(mockOpenRouter.generateText).toHaveBeenCalled();

    const emergencyEvent = await prisma.emergencyEvent.findFirst({
      where: { patientId: 'test-patient-id' },
    });

    expect(emergencyEvent?.instructions).toContain('Eat 15g of fast-acting carbs');
  });
});
```

## 16.6 Test Coverage Requirements

**Minimum Coverage Targets:**

| Component | Unit Tests | Integration Tests | E2E Tests |
|-----------|-----------|------------------|-----------|
| API Gateway (tRPC routers) | 80% | 60% | N/A |
| Agent Services | 70% | 50% | N/A |
| Web Dashboards | 60% | 30% | Critical flows |
| Mobile App | 60% | 30% | Critical flows |
| Shared Packages | 90% | N/A | N/A |

## 16.7 Running Tests

```bash
# Run all tests
pnpm test

# Run tests in watch mode
pnpm test -- --watch

# Run tests with coverage
pnpm test -- --coverage

# Run specific test file
pnpm test apps/api-gateway/src/lib/cache.test.ts

# Run E2E tests (Playwright)
cd apps/doctor-dashboard
pnpm test:e2e

# Run mobile E2E tests (Detox)
cd apps/mobile
detox test --configuration ios.sim.debug
```

## 16.8 CI/CD Test Integration

```yaml
# .github/workflows/ci.yml
name: CI Tests

on:
  pull_request:
  push:
    branches: [main, staging]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: timescale/timescaledb:latest-pg16
        env:
          POSTGRES_PASSWORD: postgres
        ports:
          - 5432:5432
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379

    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm install -g pnpm@9.15.0
      - run: pnpm install
      - run: pnpm db:migrate
      - run: pnpm test -- --coverage
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
```

---
