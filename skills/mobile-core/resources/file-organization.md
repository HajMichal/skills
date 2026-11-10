# Mobile File Organization

## Full Directory Structure

```
apps/mobile/src/
  app/                         ← Expo Router screens (file-based routing)
    _layout.tsx                ← Root layout: fonts, providers, auth redirect
    index.tsx                  ← Boot redirect
    onboarding/                ← Unauthenticated entry
      index.tsx
    auth/
      oauth/
        callback.tsx           ← myapp://auth/oauth/callback
        agreement.tsx
        access-denied.tsx
    (main)/
      _layout.tsx              ← Authenticated layout
      home/
        index.tsx              ← Home / dashboard
      settings/
        index.tsx
      profile/
        index.tsx

  components/
    ui/                        ← General reusable primitives (extend, don't replace)
      button.tsx
      text.tsx
      input.tsx
      card.tsx
      loading-state.tsx
      error-state.tsx

  modules/                     ← Feature modules (hooks + feature-scoped components)
    settings/
      AutomaticCoverageCard.tsx ← Component used only within this feature
      DisableCoverageModal.tsx
    claims/
      components/
      hooks/
    auth/
      AuthBootstrap.tsx

  api/                         ← API client, services, TanStack Query hooks
    client.ts                  ← axios instance, interceptors, token attachment
    services/                  ← per-feature request functions
    queries/                   ← useXxxQuery hooks (e.g. usePolicyQuery.ts)
    mutations/                 ← useXxxMutation hooks (e.g. useOAuthExchangeMutation.ts)

  stores/                      ← Zustand stores
    authStore.ts
    quoteStore.ts

  lib/
    i18n.ts                    ← i18next setup
    cn.ts                      ← Tailwind class merger

packages/locales/              ← shared en.json / no.json (outside apps/mobile)

  assets/                      ← (at apps/mobile/assets/)
    images/
    fonts/
```

## Naming Conventions

| What            | Convention                   | Example           |
| --------------- | ---------------------------- | ----------------- |
| Screen files    | `kebab-case.tsx`             | `trip-detail.tsx` |
| Component files | `PascalCase.tsx`             | `TripCard.tsx`    |
| Hook files      | `camelCase.ts`, `use` prefix | `useTrips.ts`     |
| Store files     | `camelCaseStore.ts`          | `authStore.ts`    |
| Utility files   | `camelCase.ts`               | `formatDate.ts`   |

## Placement Rules

| What                                              | Where                            |
| ------------------------------------------------- | -------------------------------- |
| Reusable UI primitives (Button, Text, Input)      | `components/ui/`                 |
| Feature-scoped components (TripCard, PolicyBadge) | `modules/[feature]/`             |
| TanStack Query hooks                              | `api/queries/`, `api/mutations/` |
| Feature business logic / hooks                    | `modules/[feature]/`             |
| API request functions, axios client               | `api/services/`, `api/client.ts` |
| Screens (routing only, no logic)                  | `app/`                           |
| Global state (auth, UI)                           | `stores/`                        |
