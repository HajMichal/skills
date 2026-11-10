# Expo Router Navigation

## File Structure

```
src/app/
  _layout.tsx              ← root layout (fonts, providers, auth check)
  index.tsx                ← boot redirect (e.g. to /onboarding)
  onboarding/
    index.tsx              ← unauthenticated entry
  auth/
    oauth/
      callback.tsx         ← myapp://auth/oauth/callback
  (main)/
    _layout.tsx            ← authenticated layout
    home/
      index.tsx            ← home screen
    settings/
      index.tsx
    profile/
      index.tsx
```

## Navigation

```tsx
import { router, useLocalSearchParams } from 'expo-router';

// Navigate
router.push('/settings');
router.push({ pathname: '/policy/cancel', params: { policyId: policy.id } });
router.replace('/onboarding');
router.back();

// Type-safe params
const { id } = useLocalSearchParams<{ id: string }>();
```

## Stack.Screen Options

```tsx
// In _layout.tsx
<Stack>
  <Stack.Screen
    name='trips/[id]'
    options={{
      title: 'Trip Details',
      headerShown: true,
      presentation: 'modal',
    }}
  />
</Stack>
```

## Auth Guard Pattern

```tsx
// app/_layout.tsx
export default function RootLayout() {
  const { token } = useAuthStore();

  useEffect(() => {
    if (!token) router.replace('/onboarding');
  }, [token]);

  return <Slot />;
}
```

## Deep Linking

```typescript
// app.json
{
  "scheme": "myapp",
  "plugins": [["expo-router", { "origin": "https://example.com" }]]
}
// Deep link: myapp://trips/123
```
