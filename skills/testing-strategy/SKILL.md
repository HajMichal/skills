---
name: testing-strategy
description: Test strategies, tooling, and coverage targets for all app layers of a TypeScript monorepo
---

# Testing Strategy — Quick Reference

## Test Matrix

| Layer               | Unit Tests                                                         | Integration Tests | E2E Tests           |
| ------------------- | ------------------------------------------------------------------ | ----------------- | ------------------- |
| API Services        | Jest — mock repos                                                  | Jest — real DB    | supertest + live DB |
| API Controllers     | Jest — mock services                                               | —                 | supertest           |
| Repositories        | —                                                                  | Jest — real DB    | —                   |
| Mobile Components   | RNTL                                                               | —                 | —                   |
| Mobile Screens      | RNTL + Zustand                                                     | —                 | Maestro (optional)  |
| Web app (React)     | Vitest browser mode (Playwright Chromium) + `vitest-browser-react` | —                 | —                   |

## Key Commands

```bash
pnpm test                          # all unit tests (via Turbo)
pnpm --filter <api> test           # one app's unit tests
pnpm --filter <api> test:e2e       # API e2e (needs live DB)
npx jest path/to/test.spec.ts      # single file
pnpm --filter <web-app> test       # web app: Vitest browser mode, headless Chromium
npx jest --coverage    # with coverage
```

## Rules

- **API tests**: inject mocked repos via `Test.createTestingModule()`; never call Prisma directly in unit tests
- **Repository tests**: use real DB — mocking Prisma masks migration/query bugs
- **Mobile**: React Native Testing Library; test user interactions, not implementation
- **Mobile queries**: `getByTestId`, never `getByText` on translated strings —
  `getByText` only for untranslated data values
- **Maestro**: keep platform and flake gotchas in a README next to the flows
- **iOS location permission**: delete + reinstall does not reset the grant. Use
  `xcrun simctl privacy booted reset location <bundleId>` (simulator) or Reset Location & Privacy
  (device)
- Target **> 80%** coverage on services

## Test File Naming

```
[feature].service.spec.ts       ← unit
[feature].repository.spec.ts    ← integration
[feature].controller.spec.ts    ← unit (thin; keep small)
test/[feature].e2e-spec.ts      ← e2e
```

## Resources

| Topic                  | File                                               | When to Use                                |
| ---------------------- | -------------------------------------------------- | ------------------------------------------ |
| API unit tests         | [`api-unit-tests.md`](resources/api-unit-tests.md) | Mocking deps, DI setup, assertion patterns |
| API e2e tests          | [`api-e2e-tests.md`](resources/api-e2e-tests.md)   | supertest, DB seeding, cleanup             |
| Mobile component tests | [`mobile-tests.md`](resources/mobile-tests.md)     | RNTL, mocking TanStack Query / Zustand     |
| Test data factories    | [`test-data.md`](resources/test-data.md)           | Builder factories, seed helpers            |
