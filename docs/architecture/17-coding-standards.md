# 17. Coding Standards

## 17.1 TypeScript Standards

### 17.1.1 Strict Mode

```json
// tsconfig.json
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true
  }
}
```

### 17.1.2 Naming Conventions

- **Files**: camelCase for utilities, PascalCase for components
  - ✅ `userService.ts`, `PatientCard.tsx`
  - ❌ `user_service.ts`, `patient-card.tsx`

- **Variables & Functions**: camelCase
  ```typescript
  const getUserData = async () => { /* ... */ };
  const isEmergency = value < 70 || value > 300;
  ```

- **Types & Interfaces**: PascalCase
  ```typescript
  interface PatientData {
    id: string;
    name: string;
  }

  type GlucoseState = 'MORNING_FASTING' | 'AFTER_BREAKFAST';
  ```

- **Constants**: UPPER_SNAKE_CASE
  ```typescript
  const MAX_GLUCOSE_VALUE = 600;
  const API_BASE_URL = process.env.API_URL;
  ```

- **Enums**: PascalCase (keys UPPER_SNAKE_CASE)
  ```typescript
  enum ManagementMode {
    INDEPENDENT = 'INDEPENDENT',
    CAREGIVER_ASSISTED = 'CAREGIVER_ASSISTED',
    ACTIVE_MANAGEMENT = 'ACTIVE_MANAGEMENT',
  }
  ```

### 17.1.3 Function Guidelines

**Prefer async/await over Promise chains:**

```typescript
// ✅ GOOD
async function getPatientData(patientId: string) {
  const patient = await prisma.patient.findUnique({ where: { id: patientId } });
  const readings = await prisma.glucoseReading.findMany({ where: { patientId } });
  return { patient, readings };
}

// ❌ BAD
function getPatientData(patientId: string) {
  return prisma.patient.findUnique({ where: { id: patientId } })
    .then(patient => {
      return prisma.glucoseReading.findMany({ where: { patientId } })
        .then(readings => ({ patient, readings }));
    });
}
```

**Use early returns:**

```typescript
// ✅ GOOD
function checkGlucoseLevel(value: number): string {
  if (value < 70) return 'LOW';
  if (value > 180) return 'HIGH';
  return 'NORMAL';
}

// ❌ BAD
function checkGlucoseLevel(value: number): string {
  let result = 'NORMAL';
  if (value < 70) {
    result = 'LOW';
  } else if (value > 180) {
    result = 'HIGH';
  }
  return result;
}
```

**Pure functions when possible:**

```typescript
// ✅ GOOD - Pure function
function calculateAverageGlucose(readings: { value: number }[]): number {
  if (readings.length === 0) return 0;
  const sum = readings.reduce((acc, r) => acc + r.value, 0);
  return sum / readings.length;
}

// ❌ BAD - Side effects
let totalGlucose = 0;
function calculateAverageGlucose(readings: { value: number }[]): number {
  totalGlucose = readings.reduce((acc, r) => acc + r.value, 0);
  return totalGlucose / readings.length;
}
```

## 17.2 React/Next.js Standards

### 17.2.1 Component Structure

```tsx
// ✅ GOOD - Functional component with TypeScript
import { FC } from 'react';

interface PatientCardProps {
  patient: {
    id: string;
    name: string;
    latestGlucose: number;
  };
  onSelect?: (patientId: string) => void;
}

export const PatientCard: FC<PatientCardProps> = ({ patient, onSelect }) => {
  const handleClick = () => {
    onSelect?.(patient.id);
  };

  const isEmergency = patient.latestGlucose < 70 || patient.latestGlucose > 300;

  return (
    <Card onClick={handleClick} className={isEmergency ? 'border-red-500' : ''}>
      <h3>{patient.name}</h3>
      <p>Latest: {patient.latestGlucose} mg/dL</p>
      {isEmergency && <Badge variant="destructive">Emergency</Badge>}
    </Card>
  );
};
```

### 17.2.2 Hooks Usage

```tsx
// ✅ GOOD - Custom hook
function useGlucoseHistory(patientId: string) {
  const { data, isLoading, error } = trpc.glucose.getHistory.useQuery({ patientId });

  const averageGlucose = useMemo(() => {
    if (!data) return 0;
    return data.reduce((sum, r) => sum + r.value, 0) / data.length;
  }, [data]);

  return { readings: data, averageGlucose, isLoading, error };
}

// Usage
const { readings, averageGlucose, isLoading } = useGlucoseHistory(patientId);
```

**Hook Rules:**
- Only call hooks at the top level (not in conditions/loops)
- Custom hooks must start with `use`
- Use `useMemo` for expensive computations
- Use `useCallback` for functions passed to child components

## 17.3 React Native Standards

```tsx
// ✅ GOOD - React Native component with NativeWind
import { View, Text, TouchableOpacity } from 'react-native';

interface GlucoseCardProps {
  value: number;
  state: string;
  onPress: () => void;
}

export function GlucoseCard({ value, state, onPress }: GlucoseCardProps) {
  const isHigh = value > 180;
  const isLow = value < 70;

  return (
    <TouchableOpacity onPress={onPress} className="p-4 bg-white rounded-lg shadow-md">
      <Text className="text-2xl font-bold">{value} mg/dL</Text>
      <Text className="text-gray-600">{state}</Text>
      {(isHigh || isLow) && (
        <View className={`mt-2 px-3 py-1 rounded ${isHigh ? 'bg-red-100' : 'bg-yellow-100'}`}>
          <Text className={isHigh ? 'text-red-700' : 'text-yellow-700'}>
            {isHigh ? 'High' : 'Low'}
          </Text>
        </View>
      )}
    </TouchableOpacity>
  );
}
```

## 17.4 Backend Standards

### 17.4.1 tRPC Router Organization

```typescript
// apps/api-gateway/src/trpc/routers/glucose.ts
import { router, protectedProcedure } from '../trpc';
import { z } from 'zod';
import { TRPCError } from '@trpc/server';

export const glucoseRouter = router({
  // Queries (read operations)
  getHistory: protectedProcedure
    .input(z.object({
      limit: z.number().min(1).max(100).default(30),
      startDate: z.date().optional(),
      endDate: z.date().optional(),
    }))
    .query(async ({ input, ctx }) => {
      return prisma.glucoseReading.findMany({
        where: {
          patientId: ctx.user.userId,
          recordedAt: {
            gte: input.startDate,
            lte: input.endDate,
          },
        },
        orderBy: { recordedAt: 'desc' },
        take: input.limit,
      });
    }),

  // Mutations (write operations)
  logReading: protectedProcedure
    .input(glucoseReadingSchema)
    .mutation(async ({ input, ctx }) => {
      const reading = await prisma.glucoseReading.create({
        data: {
          ...input,
          patientId: ctx.user.userId,
        },
      });

      // Check for emergency
      const isEmergency = input.value < 70 || input.value > 300;
      if (isEmergency) {
        await publishEvent('clinical.glucose.check', {
          patientId: ctx.user.userId,
          reading,
          type: input.value < 70 ? 'HYPOGLYCEMIA' : 'HYPERGLYCEMIA',
        });
      }

      return reading;
    }),

  deleteReading: protectedProcedure
    .input(z.object({ id: z.string() }))
    .mutation(async ({ input, ctx }) => {
      // Verify ownership
      const reading = await prisma.glucoseReading.findUnique({
        where: { id: input.id },
      });

      if (!reading || reading.patientId !== ctx.user.userId) {
        throw new TRPCError({ code: 'FORBIDDEN' });
      }

      await prisma.glucoseReading.delete({ where: { id: input.id } });

      return { success: true };
    }),
});
```

### 17.4.2 Error Handling

```typescript
// ✅ GOOD - Proper error handling with tRPC
async function handleRequest() {
  try {
    const data = await fetchExternalAPI();
    return data;
  } catch (error) {
    if (error instanceof ExternalAPIError) {
      throw new TRPCError({
        code: 'BAD_REQUEST',
        message: 'External API error',
        cause: error,
      });
    }

    // Log unexpected errors
    console.error('Unexpected error:', error);
    throw new TRPCError({
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred',
    });
  }
}
```

## 17.5 Database Standards

### 17.5.1 Prisma Best Practices

```typescript
// ✅ GOOD - Use transactions for related operations
await prisma.$transaction(async (tx) => {
  const meal = await tx.meal.create({ data: mealData });

  await tx.notification.create({
    data: {
      patientId,
      type: 'MEAL_LOGGED',
      message: `Meal logged: ${meal.mealType}`,
    },
  });
});

// ✅ GOOD - Use select to avoid over-fetching
const patients = await prisma.patient.findMany({
  select: {
    id: true,
    name: true,
    latestGlucose: true,
  },
});

// ❌ BAD - Fetches all fields
const patients = await prisma.patient.findMany();
```

## 17.6 Code Documentation

```typescript
/**
 * Calculates the average glucose level over a specified time period.
 *
 * @param patientId - The unique identifier of the patient
 * @param startDate - Start of the time period
 * @param endDate - End of the time period
 * @returns The average glucose value in mg/dL
 * @throws {TRPCError} If patient not found or no data available
 */
export async function calculateAverageGlucose(
  patientId: string,
  startDate: Date,
  endDate: Date
): Promise<number> {
  const readings = await prisma.glucoseReading.findMany({
    where: {
      patientId,
      recordedAt: { gte: startDate, lte: endDate },
    },
    select: { value: true },
  });

  if (readings.length === 0) {
    throw new TRPCError({
      code: 'NOT_FOUND',
      message: 'No glucose readings found for the specified period',
    });
  }

  return readings.reduce((sum, r) => sum + r.value, 0) / readings.length;
}
```

## 17.7 ESLint Configuration

```json
// .eslintrc.json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "prettier"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": ["error", { "argsIgnorePattern": "^_" }],
    "@typescript-eslint/no-explicit-any": "warn",
    "@typescript-eslint/explicit-function-return-type": "off",
    "prefer-const": "error",
    "no-console": ["warn", { "allow": ["warn", "error"] }]
  }
}
```

## 17.8 Git Commit Standards

**Conventional Commits:**

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Examples:**

```bash
feat(glucose): add voice input for glucose logging
fix(clinical-monitor): resolve race condition in emergency detection
refactor(api-gateway): extract rate limiting logic to middleware
docs(architecture): add security section
test(glucose-router): add integration tests for emergency flow
chore(deps): upgrade Prisma to v5.8.0
```

---
