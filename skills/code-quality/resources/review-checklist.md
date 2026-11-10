# Code Review Checklist

Run this before opening a PR.

## Correctness
- [ ] All edge cases handled (null/undefined at system boundaries)
- [ ] No off-by-one errors in pagination or time ranges
- [ ] Soft-delete filters (`deletedAt: null`) on all DB reads
- [ ] Error cases return appropriate HTTP status codes

## Security
- [ ] No secrets, tokens, or national ID numbers values in logs
- [ ] No raw Prisma errors exposed to clients
- [ ] No `new PrismaClient()` — only injected `PrismaService`
- [ ] No JWTs or sensitive data in `AsyncStorage`
- [ ] Third-party API bearer tokens never sent to the client

## Code Quality
- [ ] No `any` types — use `unknown` or proper types
- [ ] No `console.log` — use NestJS `Logger`
- [ ] No magic numbers — named constants
- [ ] Functions do one thing (single responsibility)
- [ ] No duplicated logic — extracted to shared function

## API (NestJS)
- [ ] Controller methods are thin (delegate to service immediately)
- [ ] Prisma only called from repositories
- [ ] Response DTOs used (no raw Prisma models returned)
- [ ] `@ApiResponse` decorators present for all response codes
- [ ] New list endpoints are paginated

## Mobile
- [ ] `FlatList` used for all lists (not `ScrollView` + `.map()`)
- [ ] All user-visible strings via `useTranslation()`
- [ ] `expo-secure-store` for sensitive data (never `AsyncStorage`)
- [ ] `isLoading` + `isError` handled for all queries
- [ ] `accessibilityLabel` on interactive elements

## Tests
- [ ] New business logic has unit tests
- [ ] Error scenarios covered
- [ ] No test-only `console.log` left in
