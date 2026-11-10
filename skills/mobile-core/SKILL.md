---
name: mobile-core
description: React Native / Expo navigation, state management, security, and performance for a TypeScript monorepo
---

# Mobile Core — Quick Reference

## State Management Rules

| What                         | Tool                | Where                          |
| ---------------------------- | ------------------- | ------------------------------ |
| Server / API data            | TanStack Query      | Always `isLoading` + `isError` |
| Auth state & cross-screen UI | Zustand             | `stores/` directory            |
| JWTs & sensitive values      | `expo-secure-store` | **Never** `AsyncStorage`       |
| Local/ephemeral UI state     | `useState`          | Inside the component           |

## Feature Module Template

One layout for every `src/modules/[feature]/`. Apply it to a module when you touch it; don't
move untouched modules in bulk.

```
src/modules/[feature]/
  index.ts              ← public surface; code outside the module imports only from here
  [feature].types.ts    ← types and Zod schemas
  [feature].const.ts    ← constants
  [feature].api.ts      ← request functions (apiClient calls), no React
  [feature].rules.ts    ← pure business rules: no React, no I/O, unit-tested
  [feature]Store.ts     ← Zustand store, only for state shared across screens
  hooks/                ← TanStack Query hooks (use[X]Query / use[X]Mutation) and other hooks
  components/           ← the module's components and screens
  __tests__/
```

- Route files in `src/app/` stay thin: read params, guard, render the module's screen. No data
  fetching or business logic in a route file
- Business rules live in the module's `.rules.ts`, not in `src/utils/`. `utils/` is for
  feature-free helpers (formatting, dates)
- `src/api/` and `src/stores/` hold only what several features share (API client, query client,
  auth/session stores). A feature's queries and store move into its module when it is touched

## Navigation (Expo Router)

```
src/app/
  onboarding/      ← unauthenticated entry
  auth/oauth/      ← OAuth login callback, access-denied
  (main)/          ← authenticated screens (home, settings, profile, …)
```

- `router.push` / `router.replace` / `router.back` with TypeScript-typed params
- Configure screens with `Stack.Screen` options
- Deep linking via `expo-linking`
- If a root hook (e.g. `useInitialRoute`) navigates whenever session/onboarding state changes, a
  screen that changes that state must not also `router.replace` to the same target — two mounts of
  one route crash Android Fabric (`addViewAt`)

## Gotchas

- No `RefreshControl` / pull-to-refresh on a `@gorhom/bottom-sheet` scrollable (`DrawerSheet`):
  it crashes with a `scrollTo` recursion on Reanimated v4. Use a refresh button or query refetch
- If the mobile app has no ESLint, never run `npx expo lint` — it installs and configures ESLint
  (package.json, lockfile). Verify with jest + `tsc --noEmit` instead
- `eas credentials` doesn't load the profile's EAS environment, so a dynamic `app.config.ts`
  resolves the prod bundle id even for a preview profile. Set the env var the config reads (e.g.
  `EXPO_PUBLIC_APP_ENV=preview eas credentials -p ios`) and check the Bundle Identifier first

## Performance Rules

- **Lists**: always `FlatList` — never `ScrollView` + `.map()`
- **Animations**: `react-native-reanimated` — never the core `Animated` API
- **Images**: `expo-image` with `contentFit` and `placeholder` blurhash
- **Heavy modules**: lazy-load with dynamic imports

## i18n

- All user-visible strings via `useTranslation()` — **no hardcoded copy**
- Add keys to every locale file at the same time
- Key format: `screen.section.element`
- Currency unit lives in the locale string, never in code (e.g. `"{{amount}} kr"` in one locale,
  `"{{amount}} NOK"` in another)

## Accessibility

- `accessibilityLabel` on all interactive elements
- `accessibilityRole` for semantic meaning
- `expo-haptics` for button feedback

## Security

- Secrets in `expo-secure-store` only
- Environment variables for config — never commit secrets
- Validate all user inputs at the form level

## Resources

| Topic                   | File                                                               | When to Use                                                |
| ----------------------- | ------------------------------------------------------------------ | ---------------------------------------------------------- |
| File & folder structure | [`file-organization.md`](resources/file-organization.md)           | Where to place new screens, components, hooks              |
| Authentication (OAuth)  | [`authentication.md`](resources/authentication.md)                 | Auth flow, JWT storage, route protection, API interceptors |
| Error & loading states  | [`error-loading-handling.md`](resources/error-loading-handling.md) | LoadingState, ErrorState, ErrorBoundary, mutation errors   |
| Expo Router patterns    | [`navigation.md`](resources/navigation.md)                         | Complex routing, params, deep links                        |
| TanStack Query setup    | [`server-state.md`](resources/server-state.md)                     | Query keys, mutations, optimistic updates                  |
| Zustand store patterns  | [`zustand-stores.md`](resources/zustand-stores.md)                 | Store slices, persist middleware                           |
| Performance checklist   | [`performance.md`](resources/performance.md)                       | FlatList optimization, Reanimated worklets                 |
| Forms & keyboard        | [`forms.md`](resources/forms.md)                                   | KeyboardAvoidingView, input types, validation              |
