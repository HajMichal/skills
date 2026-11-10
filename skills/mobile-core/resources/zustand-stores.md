# Zustand Stores

## Store Location

`apps/mobile/src/stores/`

## Auth Store (core pattern)

```typescript
// stores/authStore.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import * as SecureStore from 'expo-secure-store';

interface AuthState {
  token: string | null;
  user: JwtPayload | null;
  setToken: (token: string | null) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      token: null,
      user: null,
      setToken: (token) => set({ token, user: token ? decodeJwt(token) : null }),
      logout: () => set({ token: null, user: null }),
    }),
    {
      name: 'auth-storage',
      // Use expo-secure-store for sensitive data
      storage: createJSONStorage(() => ({
        getItem: SecureStore.getItemAsync,
        setItem: SecureStore.setItemAsync,
        removeItem: SecureStore.deleteItemAsync,
      })),
    }
  )
);
```

## UI State Store (no persistence needed)

```typescript
// stores/ui.store.ts
interface UIState {
  isOnboarding: boolean;
  activeTrip: string | null;
  setActiveTrip: (tripId: string | null) => void;
}

export const useUIStore = create<UIState>()((set) => ({
  isOnboarding: false,
  activeTrip: null,
  setActiveTrip: (tripId) => set({ activeTrip: tripId }),
}));
```

## Rules

- Zustand for auth state and UI state that spans screens
- Sensitive values (tokens) → always `expo-secure-store` storage adapter
- Non-sensitive UI state → default in-memory (no persistence needed)
- Server/API data → TanStack Query (not Zustand)
