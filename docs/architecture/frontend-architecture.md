# Frontend Architecture

This section defines the frontend architecture for ProCare's multi-platform user interfaces: React Native mobile app (patient primary interface), Next.js web dashboards (doctor/caregiver), and shared component libraries.

## Mobile App Architecture (React Native + Expo)

**Primary User Interface:** Patient mobile app with offline-first architecture.

**Technology Stack:**
- React Native 0.74+ with Expo SDK 51+
- React Native Reusables (ShadCN for React Native)
- NativeWind (TailwindCSS for React Native)
- React Navigation 6.x (navigation)
- Zustand (app state)
- TanStack Query + tRPC (server state)
- WatermelonDB (offline database)
- Expo Camera, Image Picker (OCR photo capture)
- Expo Notifications (push notifications)

### Folder Structure

```
apps/mobile/
├── src/
│   ├── app/                    # Expo Router screens
│   │   ├── (auth)/            # Auth flow (grouped route)
│   │   │   ├── login.tsx      # OTP login screen
│   │   │   └── onboarding.tsx # Patient onboarding
│   │   ├── (tabs)/            # Main app tabs (grouped route)
│   │   │   ├── _layout.tsx    # Tab navigator
│   │   │   ├── index.tsx      # Home screen
│   │   │   ├── chat.tsx       # Chat with AI agents
│   │   │   ├── trends.tsx     # Glucose trends
│   │   │   └── profile.tsx    # Patient profile
│   │   ├── glucose/           # Glucose logging screens
│   │   │   ├── log.tsx        # Manual glucose entry
│   │   │   ├── scan.tsx       # OCR photo capture
│   │   │   └── history.tsx    # Glucose history
│   │   ├── meals/             # Meal logging screens
│   │   │   ├── log.tsx        # Meal photo + entry
│   │   │   └── history.tsx    # Meal history
│   │   ├── health-records/    # Health records management
│   │   │   ├── index.tsx      # Records list
│   │   │   ├── upload.tsx     # Upload new record
│   │   │   └── [id].tsx       # Record detail
│   │   ├── diet/              # Diet module
│   │   │   ├── index.tsx      # Diet dashboard
│   │   │   └── menu-scan.tsx  # Restaurant menu scan
│   │   ├── _layout.tsx        # Root layout
│   │   └── +not-found.tsx     # 404 screen
│   ├── components/            # Reusable components
│   │   ├── ui/                # React Native Reusables components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── input.tsx
│   │   │   └── ...
│   │   ├── glucose/           # Glucose-specific components
│   │   │   ├── GlucoseChart.tsx
│   │   │   ├── GlucoseInput.tsx
│   │   │   └── GlucoseStateSelector.tsx
│   │   ├── meal/              # Meal-specific components
│   │   │   ├── MealCard.tsx
│   │   │   └── FoodRecognitionResult.tsx
│   │   └── shared/            # Common components
│   │       ├── EmergencyBanner.tsx
│   │       ├── OfflineIndicator.tsx
│   │       └── LoadingSpinner.tsx
│   ├── hooks/                 # Custom React hooks
│   │   ├── useGlucose.ts      # Glucose data hooks
│   │   ├── useMeal.ts         # Meal data hooks
│   │   ├── useOfflineSync.ts  # Offline sync hook
│   │   └── useNotifications.ts # Push notifications
│   ├── store/                 # Zustand state stores
│   │   ├── authStore.ts       # Authentication state
│   │   ├── glucoseStore.ts    # Local glucose state
│   │   └── syncStore.ts       # Sync queue state
│   ├── db/                    # WatermelonDB schema & models
│   │   ├── schema.ts          # Database schema
│   │   ├── models/            # WatermelonDB models
│   │   │   ├── GlucoseReading.ts
│   │   │   ├── Meal.ts
│   │   │   └── Medication.ts
│   │   └── sync.ts            # Sync adapter
│   ├── utils/                 # Utility functions
│   │   ├── glucose.ts         # Glucose calculations
│   │   ├── datetime.ts        # Date/time helpers
│   │   └── validation.ts      # Form validation
│   ├── lib/                   # External service clients
│   │   ├── trpc.ts            # tRPC client setup
│   │   └── notifications.ts   # Expo notifications setup
│   └── types/                 # TypeScript types
│       └── navigation.ts      # Navigation types
├── assets/                    # Static assets
│   ├── fonts/
│   └── images/
├── app.json                   # Expo configuration
├── package.json
└── tsconfig.json
```

### Screen Organization

**Authentication Flow:**

```typescript
// apps/mobile/src/app/(auth)/login.tsx
import { useState } from 'react';
import { View, Text } from 'react-native';
import { Button, Input } from '@/components/ui';
import { trpc } from '@/lib/trpc';
import { useAuthStore } from '@/store/authStore';

export default function LoginScreen() {
  const [phoneNumber, setPhoneNumber] = useState('');
  const [otp, setOtp] = useState('');
  const [otpSent, setOtpSent] = useState(false);

  const sendOtpMutation = trpc.auth.sendOtp.useMutation();
  const verifyOtpMutation = trpc.auth.verifyOtp.useMutation();
  const setUser = useAuthStore(state => state.setUser);

  const handleSendOtp = async () => {
    await sendOtpMutation.mutateAsync({ phoneNumber });
    setOtpSent(true);
  };

  const handleVerifyOtp = async () => {
    const result = await verifyOtpMutation.mutateAsync({ phoneNumber, otp });
    setUser(result.user, result.token);
    // Navigation handled by auth state change
  };

  return (
    <View className="flex-1 p-4 justify-center">
      <Text className="text-2xl font-bold mb-4">Login to ProCare</Text>
      {!otpSent ? (
        <>
          <Input
            placeholder="+91XXXXXXXXXX"
            value={phoneNumber}
            onChangeText={setPhoneNumber}
            keyboardType="phone-pad"
          />
          <Button onPress={handleSendOtp} loading={sendOtpMutation.isLoading}>
            Send OTP
          </Button>
        </>
      ) : (
        <>
          <Input
            placeholder="Enter 6-digit OTP"
            value={otp}
            onChangeText={setOtp}
            keyboardType="number-pad"
            maxLength={6}
          />
          <Button onPress={handleVerifyOtp} loading={verifyOtpMutation.isLoading}>
            Verify OTP
          </Button>
        </>
      )}
    </View>
  );
}
```

**Home Screen (Context-Aware):**

```typescript
// apps/mobile/src/app/(tabs)/index.tsx
import { View, ScrollView, Text } from 'react-native';
import { Card } from '@/components/ui';
import { GlucoseChart } from '@/components/glucose/GlucoseChart';
import { EmergencyBanner } from '@/components/shared/EmergencyBanner';
import { useGlucose } from '@/hooks/useGlucose';
import { trpc } from '@/lib/trpc';

export default function HomeScreen() {
  const { data: dashboard } = trpc.patient.getDashboard.useQuery();
  const { lastReading, isEmergency } = useGlucose();

  return (
    <ScrollView className="flex-1 bg-gray-50">
      {isEmergency && <EmergencyBanner reading={lastReading} />}

      <View className="p-4">
        {/* Benchmark Card */}
        <Card className="mb-4">
          <Text className="text-lg font-semibold">Your Benchmark</Text>
          <Text className="text-3xl font-bold text-blue-600">
            {dashboard?.lastHbA1c}% HbA1c
          </Text>
          <Text className="text-sm text-gray-600">
            Target: {dashboard?.patient.targetHbA1c}%
          </Text>
        </Card>

        {/* Quick Actions */}
        <View className="grid grid-cols-2 gap-4 mb-4">
          <QuickActionCard icon="droplet" label="Log Glucose" href="/glucose/log" />
          <QuickActionCard icon="utensils" label="Log Meal" href="/meals/log" />
        </View>

        {/* Recent Glucose Trend */}
        <Card className="mb-4">
          <Text className="text-lg font-semibold mb-2">7-Day Glucose Trend</Text>
          <GlucoseChart data={dashboard?.recentReadings} />
        </Card>

        {/* Pending Alerts */}
        {dashboard?.pendingAlerts.map(alert => (
          <AlertCard key={alert.id} alert={alert} />
        ))}
      </View>
    </ScrollView>
  );
}
```

**Glucose Logging Screen:**

```typescript
// apps/mobile/src/app/glucose/log.tsx
import { useState } from 'react';
import { View } from 'react-native';
import { Button, Input, Select } from '@/components/ui';
import { GlucoseStateSelector } from '@/components/glucose/GlucoseStateSelector';
import { useGlucose } from '@/hooks/useGlucose';
import { useOfflineSync } from '@/hooks/useOfflineSync';

export default function LogGlucoseScreen() {
  const [value, setValue] = useState('');
  const [state, setState] = useState<GlucoseState>('MORNING_FASTING');
  const [recordedAt, setRecordedAt] = useState(new Date());

  const { logReading } = useGlucose();
  const { isOnline } = useOfflineSync();

  const handleSubmit = async () => {
    await logReading({
      value: parseFloat(value),
      state,
      recordedAt,
      source: 'MANUAL',
    });
    // Navigate back or show success
  };

  return (
    <View className="flex-1 p-4">
      <Input
        label="Glucose Value (mg/dL)"
        value={value}
        onChangeText={setValue}
        keyboardType="decimal-pad"
        placeholder="120"
      />

      <GlucoseStateSelector value={state} onChange={setState} />

      <DateTimePicker
        label="When did you check?"
        value={recordedAt}
        onChange={setRecordedAt}
      />

      {!isOnline && (
        <Text className="text-amber-600 text-sm mb-2">
          ⚠️ You're offline. Data will sync when online.
        </Text>
      )}

      <Button onPress={handleSubmit}>Log Glucose</Button>
    </View>
  );
}
```

### State Management Patterns

**Zustand Stores:**

```typescript
// apps/mobile/src/store/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  setUser: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      setUser: (user, token) => set({ user, token, isAuthenticated: true }),
      logout: () => set({ user: null, token: null, isAuthenticated: false }),
    }),
    {
      name: 'auth-storage',
      storage: createJSONStorage(() => AsyncStorage),
    }
  )
);
```

**TanStack Query + tRPC:**

```typescript
// apps/mobile/src/lib/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import { httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@procare/api';
import { useAuthStore } from '@/store/authStore';

export const trpc = createTRPCReact<AppRouter>();

export const TRPCProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [queryClient] = useState(() => new QueryClient());
  const token = useAuthStore(state => state.token);

  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: process.env.EXPO_PUBLIC_API_URL + '/trpc',
          headers: () => ({
            authorization: token ? `Bearer ${token}` : undefined,
          }),
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
};
```

**WatermelonDB Offline Storage:**

```typescript
// apps/mobile/src/db/schema.ts
import { appSchema, tableSchema } from '@nozbe/watermelondb';

export const schema = appSchema({
  version: 1,
  tables: [
    tableSchema({
      name: 'glucose_readings',
      columns: [
        { name: 'patient_id', type: 'string', isIndexed: true },
        { name: 'value', type: 'number' },
        { name: 'state', type: 'string' },
        { name: 'recorded_at', type: 'number', isIndexed: true }, // Unix timestamp
        { name: 'source', type: 'string' },
        { name: 'is_synced', type: 'boolean', isIndexed: true },
        { name: 'created_at', type: 'number' },
      ],
    }),
    tableSchema({
      name: 'meals',
      columns: [
        { name: 'patient_id', type: 'string', isIndexed: true },
        { name: 'meal_type', type: 'string' },
        { name: 'photo_url', type: 'string', isOptional: true },
        { name: 'consumed_at', type: 'number', isIndexed: true },
        { name: 'recognized_foods', type: 'string' }, // JSON array
        { name: 'is_synced', type: 'boolean', isIndexed: true },
        { name: 'created_at', type: 'number' },
      ],
    }),
  ],
});

// apps/mobile/src/db/models/GlucoseReading.ts
import { Model } from '@nozbe/watermelondb';
import { field, date, readonly } from '@nozbe/watermelondb/decorators';

export class GlucoseReading extends Model {
  static table = 'glucose_readings';

  @field('patient_id') patientId!: string;
  @field('value') value!: number;
  @field('state') state!: GlucoseState;
  @date('recorded_at') recordedAt!: Date;
  @field('source') source!: DataSource;
  @field('is_synced') isSynced!: boolean;
  @readonly @date('created_at') createdAt!: Date;
}
```

**Offline Sync Hook:**

```typescript
// apps/mobile/src/hooks/useOfflineSync.ts
import { useEffect } from 'react';
import { useNetInfo } from '@react-native-community/netinfo';
import { useDatabase } from '@nozbe/watermelondb/hooks';
import { trpc } from '@/lib/trpc';

export function useOfflineSync() {
  const netInfo = useNetInfo();
  const database = useDatabase();
  const syncMutation = trpc.sync.syncOfflineData.useMutation();

  useEffect(() => {
    if (netInfo.isConnected) {
      syncUnsyncedData();
    }
  }, [netInfo.isConnected]);

  const syncUnsyncedData = async () => {
    // Get all unsynced records
    const glucoseReadings = await database.collections
      .get('glucose_readings')
      .query(Q.where('is_synced', false))
      .fetch();

    const meals = await database.collections
      .get('meals')
      .query(Q.where('is_synced', false))
      .fetch();

    if (glucoseReadings.length === 0 && meals.length === 0) return;

    // Sync to server
    const result = await syncMutation.mutateAsync({
      glucoseReadings: glucoseReadings.map(r => r._raw),
      meals: meals.map(m => m._raw),
    });

    // Mark as synced locally
    await database.write(async () => {
      for (const reading of glucoseReadings) {
        await reading.update(r => { r.isSynced = true; });
      }
      for (const meal of meals) {
        await meal.update(m => { m.isSynced = true; });
      }
    });
  };

  return {
    isOnline: netInfo.isConnected ?? false,
    syncNow: syncUnsyncedData,
  };
}
```

### Navigation Architecture

**Expo Router (File-based routing):**

```typescript
// apps/mobile/src/app/_layout.tsx
import { Stack } from 'expo-router';
import { TRPCProvider } from '@/lib/trpc';
import { DatabaseProvider } from '@nozbe/watermelondb/hooks';
import { database } from '@/db';
import { useAuthStore } from '@/store/authStore';

export default function RootLayout() {
  const isAuthenticated = useAuthStore(state => state.isAuthenticated);

  return (
    <TRPCProvider>
      <DatabaseProvider database={database}>
        <Stack screenOptions={{ headerShown: false }}>
          {!isAuthenticated ? (
            <Stack.Screen name="(auth)" />
          ) : (
            <Stack.Screen name="(tabs)" />
          )}
        </Stack>
      </DatabaseProvider>
    </TRPCProvider>
  );
}

// apps/mobile/src/app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { HomeIcon, ChatIcon, TrendIcon, ProfileIcon } from '@/components/icons';

export default function TabsLayout() {
  return (
    <Tabs>
      <Tabs.Screen
        name="index"
        options={{
          title: 'Home',
          tabBarIcon: ({ color }) => <HomeIcon color={color} />,
        }}
      />
      <Tabs.Screen
        name="chat"
        options={{
          title: 'Chat',
          tabBarIcon: ({ color }) => <ChatIcon color={color} />,
        }}
      />
      <Tabs.Screen
        name="trends"
        options={{
          title: 'Trends',
          tabBarIcon: ({ color }) => <TrendIcon color={color} />,
        }}
      />
      <Tabs.Screen
        name="profile"
        options={{
          title: 'Profile',
          tabBarIcon: ({ color }) => <ProfileIcon color={color} />,
        }}
      />
    </Tabs>
  );
}
```

## Web Dashboards Architecture (Next.js)

**Technology Stack:**
- Next.js 14+ (App Router)
- ShadCN + Radix UI (component library)
- TailwindCSS (styling)
- Zustand + TanStack Query + tRPC (state management)
- Recharts (glucose trend charts)
- NextAuth.js (authentication)

### Folder Structure

```
apps/doctor-dashboard/
├── src/
│   ├── app/                   # Next.js App Router
│   │   ├── (auth)/           # Auth layout group
│   │   │   └── login/
│   │   │       └── page.tsx
│   │   ├── (dashboard)/      # Dashboard layout group
│   │   │   ├── layout.tsx    # Dashboard layout with sidebar
│   │   │   ├── page.tsx      # Dashboard home (patient buckets)
│   │   │   ├── patients/
│   │   │   │   ├── [id]/
│   │   │   │   │   └── page.tsx  # Patient detail
│   │   │   │   └── page.tsx      # Patients list
│   │   │   ├── pooled-questions/
│   │   │   │   └── page.tsx
│   │   │   ├── appointments/
│   │   │   │   └── page.tsx
│   │   │   └── settings/
│   │   │       └── page.tsx
│   │   ├── api/              # API routes
│   │   │   └── trpc/
│   │   │       └── [trpc]/
│   │   │           └── route.ts  # tRPC handler
│   │   ├── layout.tsx        # Root layout
│   │   └── globals.css       # Global styles
│   ├── components/           # Reusable components
│   │   ├── ui/               # ShadCN components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   └── ...
│   │   ├── dashboard/        # Dashboard-specific
│   │   │   ├── PatientBuckets.tsx
│   │   │   ├── PatientCard.tsx
│   │   │   └── Sidebar.tsx
│   │   ├── patient/          # Patient detail components
│   │   │   ├── GlucoseTrendChart.tsx
│   │   │   ├── MedicationAdherence.tsx
│   │   │   └── EmergencyTimeline.tsx
│   │   └── shared/
│   │       └── LoadingSpinner.tsx
│   ├── hooks/                # Custom hooks
│   │   ├── useDoctorDashboard.ts
│   │   └── usePatientDetail.ts
│   ├── store/                # Zustand stores
│   │   └── authStore.ts
│   ├── lib/                  # Utilities
│   │   ├── trpc.ts           # tRPC client
│   │   └── utils.ts
│   └── types/
│       └── index.ts
├── public/
├── next.config.js
├── tailwind.config.ts
└── tsconfig.json
```

### Page Examples

**Dashboard Home (Patient Buckets):**

```typescript
// apps/doctor-dashboard/src/app/(dashboard)/page.tsx
import { PatientBuckets } from '@/components/dashboard/PatientBuckets';
import { trpc } from '@/lib/trpc';

export default async function DashboardPage() {
  // Server-side data fetching with tRPC
  const dashboard = await trpc.doctor.getDashboard.query();

  return (
    <div className="p-8">
      <h1 className="text-3xl font-bold mb-6">Patient Dashboard</h1>
      <PatientBuckets
        urgent={dashboard.urgent}
        attention={dashboard.attention}
        stable={dashboard.stable}
        callIn={dashboard.callIn}
      />
    </div>
  );
}
```

**Patient Detail Page:**

```typescript
// apps/doctor-dashboard/src/app/(dashboard)/patients/[id]/page.tsx
'use client';

import { useParams } from 'next/navigation';
import { trpc } from '@/lib/trpc';
import { GlucoseTrendChart } from '@/components/patient/GlucoseTrendChart';
import { MedicationAdherence } from '@/components/patient/MedicationAdherence';
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

export default function PatientDetailPage() {
  const { id } = useParams();
  const { data: patient, isLoading } = trpc.doctor.getPatientDetail.useQuery({ patientId: id });
  const { data: summary } = trpc.ai.generatePatientSummary.useQuery({ patientId: id });

  if (isLoading) return <LoadingSpinner />;

  return (
    <div className="p-8">
      <div className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold">{patient.user.patient.fullName}</h1>
        <Button onClick={() => /* Send message */}>Send Message</Button>
      </div>

      <div className="grid grid-cols-3 gap-6">
        {/* Left Column - Metrics */}
        <div className="col-span-2 space-y-6">
          <Card>
            <CardHeader>
              <CardTitle>7-Day Glucose Trend</CardTitle>
            </CardHeader>
            <CardContent>
              <GlucoseTrendChart data={patient.recentReadings} />
            </CardContent>
          </Card>

          <Card>
            <CardHeader>
              <CardTitle>Medication Adherence</CardTitle>
            </CardHeader>
            <CardContent>
              <MedicationAdherence data={patient.medicationAdherence} />
            </CardContent>
          </Card>

          {patient.emergencyEvents.length > 0 && (
            <Card className="border-red-500">
              <CardHeader>
                <CardTitle className="text-red-600">Recent Emergencies</CardTitle>
              </CardHeader>
              <CardContent>
                <EmergencyTimeline events={patient.emergencyEvents} />
              </CardContent>
            </Card>
          )}
        </div>

        {/* Right Column - AI Summary */}
        <div>
          <Card>
            <CardHeader>
              <CardTitle>AI Summary</CardTitle>
            </CardHeader>
            <CardContent>
              <p className="text-sm text-gray-700">{summary?.summary}</p>
            </CardContent>
          </Card>
        </div>
      </div>
    </div>
  );
}
```

### Component Patterns

**ShadCN Component Usage:**

```typescript
// apps/doctor-dashboard/src/components/dashboard/PatientBuckets.tsx
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { PatientCard } from './PatientCard';

interface PatientBucketsProps {
  urgent: Patient[];
  attention: Patient[];
  stable: Patient[];
  callIn: Patient[];
}

export function PatientBuckets({ urgent, attention, stable, callIn }: PatientBucketsProps) {
  return (
    <div className="space-y-6">
      {/* Urgent Bucket */}
      <Card className="border-red-500">
        <CardHeader className="bg-red-50">
          <div className="flex items-center justify-between">
            <CardTitle className="text-red-700">Urgent Attention</CardTitle>
            <Badge variant="destructive">{urgent.length}</Badge>
          </div>
        </CardHeader>
        <CardContent className="pt-6">
          <div className="space-y-4">
            {urgent.map(patient => (
              <PatientCard key={patient.id} patient={patient} priority="urgent" />
            ))}
          </div>
        </CardContent>
      </Card>

      {/* Attention Bucket */}
      <Card className="border-amber-500">
        <CardHeader className="bg-amber-50">
          <div className="flex items-center justify-between">
            <CardTitle className="text-amber-700">Needs Attention</CardTitle>
            <Badge variant="warning">{attention.length}</Badge>
          </div>
        </CardHeader>
        <CardContent className="pt-6">
          <div className="space-y-4">
            {attention.map(patient => (
              <PatientCard key={patient.id} patient={patient} priority="attention" />
            ))}
          </div>
        </CardContent>
      </Card>

      {/* Stable Bucket - Collapsed by default */}
      <Collapsible>
        <Card>
          <CardHeader>
            <div className="flex items-center justify-between">
              <CardTitle>Stable ({stable.length})</CardTitle>
              <CollapsibleTrigger asChild>
                <Button variant="ghost">Expand</Button>
              </CollapsibleTrigger>
            </div>
          </CardHeader>
          <CollapsibleContent>
            <CardContent>
              <div className="grid grid-cols-2 gap-4">
                {stable.map(patient => (
                  <PatientCard key={patient.id} patient={patient} priority="stable" compact />
                ))}
              </div>
            </CardContent>
          </CollapsibleContent>
        </Card>
      </Collapsible>
    </div>
  );
}
```

### State Management

**NextAuth.js Authentication:**

```typescript
// apps/doctor-dashboard/src/app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';
import { prisma } from '@procare/database';
import bcrypt from 'bcryptjs';

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        email: { label: "Email", type: "email" },
        password: { label: "Password", type: "password" }
      },
      async authorize(credentials) {
        const user = await prisma.user.findUnique({
          where: { email: credentials.email },
          include: { doctor: true },
        });

        if (!user || !user.passwordHash) return null;

        const isValid = await bcrypt.compare(credentials.password, user.passwordHash);
        if (!isValid) return null;

        return {
          id: user.id,
          email: user.email,
          role: user.role,
          doctor: user.doctor,
        };
      },
    }),
  ],
  session: { strategy: 'jwt' },
  pages: {
    signIn: '/login',
  },
});

export { handler as GET, handler as POST };
```

**tRPC Client Setup:**

```typescript
// apps/doctor-dashboard/src/lib/trpc.ts
import { createTRPCReact } from '@trpc/react-query';
import { httpBatchLink } from '@trpc/client';
import type { AppRouter } from '@procare/api';
import { getSession } from 'next-auth/react';

export const trpc = createTRPCReact<AppRouter>();

export const TRPCProvider: React.FC<{ children: React.ReactNode }> = ({ children }) => {
  const [queryClient] = useState(() => new QueryClient());

  const [trpcClient] = useState(() =>
    trpc.createClient({
      links: [
        httpBatchLink({
          url: process.env.NEXT_PUBLIC_API_URL + '/trpc',
          headers: async () => {
            const session = await getSession();
            return {
              authorization: session?.user ? `Bearer ${session.accessToken}` : undefined,
            };
          },
        }),
      ],
    })
  );

  return (
    <trpc.Provider client={trpcClient} queryClient={queryClient}>
      <QueryClientProvider client={queryClient}>
        {children}
      </QueryClientProvider>
    </trpc.Provider>
  );
};
```

## Shared Frontend Patterns

### Error Boundaries

```typescript
// packages/shared/src/components/ErrorBoundary.tsx
import React from 'react';

interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
}

interface State {
  hasError: boolean;
  error?: Error;
}

export class ErrorBoundary extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
    // Log to error tracking service (Sentry)
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || (
        <div className="p-4 bg-red-50 border border-red-500 rounded">
          <h2 className="text-lg font-semibold text-red-700">Something went wrong</h2>
          <p className="text-sm text-red-600">{this.state.error?.message}</p>
        </div>
      );
    }

    return this.props.children;
  }
}
```

### Form Validation (Zod)

```typescript
// packages/shared/src/validation/glucose.ts
import { z } from 'zod';

export const glucoseLogSchema = z.object({
  value: z.number()
    .min(40, 'Glucose value must be at least 40 mg/dL')
    .max(600, 'Glucose value must be at most 600 mg/dL'),
  state: z.enum([
    'MORNING_FASTING',
    'AFTER_BREAKFAST',
    'BEFORE_LUNCH',
    'AFTER_LUNCH',
    'BEFORE_DINNER',
    'AFTER_DINNER',
    'BEDTIME',
    'RANDOM',
  ]),
  recordedAt: z.date(),
  source: z.enum(['MANUAL', 'OCR_PHOTO', 'BLUETOOTH', 'VOICE']),
  photoUrl: z.string().url().optional(),
  notes: z.string().max(500).optional(),
});

export type GlucoseLogInput = z.infer<typeof glucoseLogSchema>;
```

### Loading States

```typescript
// packages/shared/src/hooks/useAsync.ts
import { useState, useCallback } from 'react';

export function useAsync<T, E = Error>() {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<E | null>(null);
  const [data, setData] = useState<T | null>(null);

  const execute = useCallback(async (promise: Promise<T>) => {
    setIsLoading(true);
    setError(null);

    try {
      const result = await promise;
      setData(result);
      return result;
    } catch (err) {
      setError(err as E);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  return { isLoading, error, data, execute };
}
```

## Performance Optimization

**React Native:**
1. **Image Optimization:** Use `expo-image` for automatic caching and lazy loading
2. **List Virtualization:** Use `FlashList` instead of `FlatList` for glucose/meal history
3. **Code Splitting:** Use `React.lazy()` for heavy screens (health records viewer)
4. **Memoization:** Use `React.memo()` for expensive components (GlucoseChart)

**Next.js:**
1. **Server Components:** Use React Server Components for data fetching
2. **Image Optimization:** Use `next/image` for automatic optimization
3. **Route Prefetching:** Automatic prefetching with Next.js Link
4. **Bundle Analysis:** Use `@next/bundle-analyzer` to identify large dependencies

---
