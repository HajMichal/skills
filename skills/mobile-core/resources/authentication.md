# Mobile Authentication

Example: login through an external OAuth/OIDC provider (opened in the system browser), with the API exchanging the code for its own JWT.

## Auth Flow

```
User taps "Continue with <Provider>"
  → Open the OAuth provider in expo-web-browser
  → The provider redirects back with auth code
  → Exchange code for JWT at the API
  → Store JWT in expo-secure-store
  → Redirect to (main)/ screens
```

## Implementation

```tsx
// modules/auth/useOAuthLogin.ts
import * as WebBrowser from 'expo-web-browser';
import * as Linking from 'expo-linking';
import { useAuthStore } from '@/stores/authStore';

export function useOAuthLogin() {
  const { setToken } = useAuthStore();
  const { mutate: exchangeCode, isPending } = useMutation({
    mutationFn: (code: string) => api.auth.exchangeOAuthCode(code),
    onSuccess: ({ accessToken }) => {
      setToken(accessToken);
      router.replace('/(main)/home');
    },
  });

  const login = async () => {
    const redirectUri = Linking.createURL('/auth/oauth/callback');
    const result = await WebBrowser.openAuthSessionAsync(
      `${API_URL}/auth/oauth?redirect_uri=${encodeURIComponent(redirectUri)}`,
      redirectUri
    );

    if (result.type === 'success') {
      const { code } = Linking.parse(result.url).queryParams ?? {};
      if (typeof code === 'string') exchangeCode(code);
    }
  };

  return { login, isPending };
}
```

## JWT Storage (expo-secure-store only)

```typescript
// stores/authStore.ts
import * as SecureStore from 'expo-secure-store';
import { createJSONStorage, persist } from 'zustand/middleware';

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      setToken: (token) => set({ token, user: token ? decodeJwt<JwtPayload>(token) : null }),
      logout: () => {
        set({ token: null, user: null });
        router.replace('/onboarding');
      },
    }),
    {
      name: 'auth',
      storage: createJSONStorage(() => ({
        getItem: SecureStore.getItemAsync,
        setItem: SecureStore.setItemAsync,
        removeItem: SecureStore.deleteItemAsync,
      })),
    }
  )
);
```

## Route Protection

```tsx
// app/_layout.tsx — redirect unauthenticated users
export default function RootLayout() {
  const { token } = useAuthStore();
  const segments = useSegments();

  useEffect(() => {
    const inAuthGroup = segments[0] === 'auth' || segments[0] === 'onboarding';
    if (!token && !inAuthGroup) {
      router.replace('/onboarding');
    } else if (token && inAuthGroup) {
      router.replace('/(main)/home');
    }
  }, [token, segments]);

  return (
    <QueryClientProvider client={queryClient}>
      <Slot />
    </QueryClientProvider>
  );
}
```

## API Client — Attaching Token

```typescript
// api/client.ts
import axios from 'axios';
import { useAuthStore } from '@/stores/authStore';

export const apiClient = axios.create({ baseURL: process.env.EXPO_PUBLIC_API_URL });

apiClient.interceptors.request.use((config) => {
  const token = useAuthStore.getState().token;
  if (token) config.headers.Authorization = `Bearer ${token}`;
  return config;
});

// Handle 401 — token expired
apiClient.interceptors.response.use(
  (res) => res,
  (error) => {
    if (error.response?.status === 401) {
      useAuthStore.getState().logout();
    }
    return Promise.reject(error);
  }
);
```

## Deep Link Callback Setup

```json
// app.config.ts (expo config)
{
  "scheme": "myapp",
  "plugins": [["expo-router", { "origin": "https://example.com" }]]
}
```

```tsx
// app/auth/oauth/callback.tsx — handles myapp://auth/oauth/callback
export default function AuthCallbackScreen() {
  const params = useLocalSearchParams<{ code?: string; error?: string }>();
  // This screen is rarely visible — redirect happens in the auth hook
  return <LoadingState />;
}
```
