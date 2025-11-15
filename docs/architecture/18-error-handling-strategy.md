# 18. Error Handling Strategy

## 18.1 Error Types

### 18.1.1 Application Errors

```typescript
// packages/shared/src/errors/ApplicationError.ts
export class ApplicationError extends Error {
  constructor(
    public code: string,
    message: string,
    public statusCode: number = 500,
    public meta?: Record<string, any>
  ) {
    super(message);
    this.name = 'ApplicationError';
  }
}

export class ValidationError extends ApplicationError {
  constructor(message: string, meta?: Record<string, any>) {
    super('VALIDATION_ERROR', message, 400, meta);
    this.name = 'ValidationError';
  }
}

export class NotFoundError extends ApplicationError {
  constructor(resource: string) {
    super('NOT_FOUND', `${resource} not found`, 404);
    this.name = 'NotFoundError';
  }
}

export class UnauthorizedError extends ApplicationError {
  constructor(message: string = 'Unauthorized') {
    super('UNAUTHORIZED', message, 401);
    this.name = 'UnauthorizedError';
  }
}

export class RateLimitError extends ApplicationError {
  constructor() {
    super('RATE_LIMIT_EXCEEDED', 'Too many requests', 429);
    this.name = 'RateLimitError';
  }
}
```

## 18.2 Frontend Error Handling

### 18.2.1 tRPC Error Handling (React)

```tsx
// apps/doctor-dashboard/src/components/PatientList.tsx
import { trpc } from '@/lib/trpc';
import { TRPCClientError } from '@trpc/client';

export function PatientList() {
  const { data, error, isLoading } = trpc.doctor.getPatients.useQuery();

  if (isLoading) return <LoadingSpinner />;

  if (error) {
    // Handle specific error codes
    if (error.data?.code === 'FORBIDDEN') {
      return <ErrorMessage>You don't have permission to view patients.</ErrorMessage>;
    }

    if (error.data?.code === 'UNAUTHORIZED') {
      // Redirect to login
      router.push('/login');
      return null;
    }

    // Generic error
    return <ErrorMessage>Failed to load patients. Please try again.</ErrorMessage>;
  }

  return <div>{/* Render patients */}</div>;
}
```

### 18.2.2 Error Boundary (React)

```tsx
// apps/doctor-dashboard/src/components/ErrorBoundary.tsx
import { Component, ReactNode } from 'react';

interface Props {
  children: ReactNode;
  fallback?: ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error): State {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: any) {
    // Log to error tracking service (Sentry)
    console.error('ErrorBoundary caught:', error, errorInfo);

    if (process.env.NODE_ENV === 'production') {
      // Send to Sentry
      // Sentry.captureException(error, { extra: errorInfo });
    }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="p-8 text-center">
          <h2 className="text-2xl font-bold text-red-600">Something went wrong</h2>
          <p className="mt-2 text-gray-600">Please refresh the page or contact support.</p>
          <button onClick={() => window.location.reload()} className="mt-4 btn-primary">
            Refresh Page
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### 18.2.3 React Native Error Handling

```tsx
// apps/mobile/src/hooks/useErrorHandler.ts
import { useCallback } from 'react';
import { Alert } from 'react-native';

export function useErrorHandler() {
  const handleError = useCallback((error: unknown) => {
    if (error instanceof TRPCClientError) {
      const errorCode = error.data?.code;

      switch (errorCode) {
        case 'UNAUTHORIZED':
          Alert.alert('Session Expired', 'Please log in again.');
          // Navigate to login
          break;
        case 'RATE_LIMIT_EXCEEDED':
          Alert.alert('Too Many Requests', 'Please wait a moment and try again.');
          break;
        default:
          Alert.alert('Error', error.message || 'An unexpected error occurred.');
      }
    } else if (error instanceof Error) {
      Alert.alert('Error', error.message);
    } else {
      Alert.alert('Error', 'An unexpected error occurred.');
    }

    // Log to error tracking
    if (process.env.NODE_ENV === 'production') {
      // Sentry.captureException(error);
    }
  }, []);

  return { handleError };
}
```

## 18.3 Backend Error Handling

### 18.3.1 tRPC Error Middleware

```typescript
// apps/api-gateway/src/trpc/trpc.ts
import { initTRPC, TRPCError } from '@trpc/server';

const t = initTRPC.context<Context>().create({
  errorFormatter({ shape, error }) {
    return {
      ...shape,
      data: {
        ...shape.data,
        // Add custom error fields
        timestamp: new Date().toISOString(),
        path: shape.data.path,
      },
    };
  },
});

// Global error handling middleware
const errorHandlerMiddleware = t.middleware(async ({ next }) => {
  try {
    return await next();
  } catch (error) {
    // Log error
    console.error('[tRPC Error]', error);

    // Send to error tracking (Sentry)
    if (process.env.NODE_ENV === 'production') {
      // Sentry.captureException(error);
    }

    // Re-throw to let tRPC handle the response
    throw error;
  }
});

export const publicProcedure = t.procedure.use(errorHandlerMiddleware);
```

### 18.3.2 Express Error Handler

```typescript
// apps/api-gateway/src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';
import { ApplicationError } from 'shared';

export function errorHandler(err: Error, req: Request, res: Response, next: NextFunction) {
  console.error('[Express Error]', err);

  // Application errors
  if (err instanceof ApplicationError) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        meta: err.meta,
      },
    });
  }

  // Unhandled errors
  if (process.env.NODE_ENV === 'production') {
    // Sentry.captureException(err);
  }

  res.status(500).json({
    error: {
      code: 'INTERNAL_SERVER_ERROR',
      message: 'An unexpected error occurred',
    },
  });
}
```

## 18.4 Agent Service Error Handling

```typescript
// apps/agent-services/clinical-monitor/src/handlers/glucoseCheck.ts
export async function handleGlucoseCheck(event: GlucoseCheckEvent) {
  try {
    await processEmergency(event);
  } catch (error) {
    console.error('Failed to process glucose emergency:', error);

    // Retry logic
    if (error instanceof TemporaryError) {
      // Requeue event for retry
      await publishEvent('clinical.glucose.check.retry', event);
    } else {
      // Send to dead letter queue
      await publishEvent('clinical.glucose.check.failed', {
        ...event,
        error: error instanceof Error ? error.message : 'Unknown error',
      });

      // Alert admins
      await sendAdminAlert({
        type: 'AGENT_ERROR',
        service: 'clinical-monitor',
        error: error instanceof Error ? error.message : 'Unknown error',
      });
    }

    // Don't throw - acknowledge the message to prevent infinite requeuing
  }
}
```

## 18.5 Database Error Handling

```typescript
// packages/shared/src/lib/database-errors.ts
import { Prisma } from '@prisma/client';
import { TRPCError } from '@trpc/server';

export function handlePrismaError(error: unknown): never {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    switch (error.code) {
      case 'P2002':
        // Unique constraint violation
        throw new TRPCError({
          code: 'CONFLICT',
          message: 'A record with this value already exists',
        });

      case 'P2025':
        // Record not found
        throw new TRPCError({
          code: 'NOT_FOUND',
          message: 'Record not found',
        });

      case 'P2003':
        // Foreign key constraint violation
        throw new TRPCError({
          code: 'BAD_REQUEST',
          message: 'Referenced record does not exist',
        });

      default:
        throw new TRPCError({
          code: 'INTERNAL_SERVER_ERROR',
          message: 'Database error',
        });
    }
  }

  throw new TRPCError({
    code: 'INTERNAL_SERVER_ERROR',
    message: 'An unexpected error occurred',
  });
}
```

## 18.6 Error Logging & Monitoring

**Sentry Integration:**

```typescript
// packages/shared/src/lib/sentry.ts
import * as Sentry from '@sentry/node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: process.env.NODE_ENV === 'production' ? 0.1 : 1.0,
  beforeSend(event, hint) {
    // Filter sensitive data
    if (event.request?.headers) {
      delete event.request.headers['authorization'];
    }

    return event;
  },
});

export function captureError(error: Error, context?: Record<string, any>) {
  Sentry.captureException(error, {
    extra: context,
  });
}
```

---
