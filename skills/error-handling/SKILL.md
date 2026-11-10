---
name: error-handling
description: Error handling patterns per layer — API exceptions, Prisma errors, client boundaries
---

# Error Handling — Quick Reference

## Pattern by Layer

| Layer | Throw | Catch | Never |
|-------|-------|-------|-------|
| Repository | Domain-specific exceptions | `Prisma.PrismaClientKnownRequestError` only for domain-specific cases | Expose raw Prisma errors |
| Service | NestJS HTTP exceptions | Domain-specific | Expose stack traces |
| Controller | — | Global `PrismaExceptionFilter` + Nest's built-in HTTP exception handling | Handle business errors |
| Mobile | — | Error boundaries, try-catch | Unhandled promise rejections |

## Repository Pattern

The global `PrismaExceptionFilter` (`src/core/common/filters/prisma-exception.filter.ts`, registered as `APP_FILTER` in `app.module.ts`) maps `Prisma.PrismaClientKnownRequestError` to HTTP responses (P2002→409, P2025→404, P2003/P2011/P2014/P2015/P2020/P2023→400, others→500 without leaking internals). Don't wrap Prisma calls in try/catch by default — catch locally only when a code needs a domain-specific exception or message.

```typescript
import { Prisma } from 'generated/prisma/client';

// user.repository.ts — P2002 needs a domain-specific response, so it is caught locally
async updateProfile(userId: string, data: Prisma.UserUpdateInput) {
  try {
    return await this.prisma.user.update({ where: { id: userId }, data });
  } catch (error) {
    if (error instanceof Prisma.PrismaClientKnownRequestError && error.code === 'P2002') {
      const target = error.meta?.target;
      const fields = Array.isArray(target) ? target.map(String) : target ? [String(target)] : [];
      throw new ProfileFieldConflictException(fields);
    }
    throw error;
  }
}
```

## API — Standard HTTP Exceptions

```typescript
throw new NotFoundException('Resource not found');
throw new BadRequestException('Invalid input');
throw new ConflictException('Already exists');
throw new UnauthorizedException('Invalid token');
throw new ForbiddenException('Access denied');
```

## Mobile — Error Boundaries

```tsx
// Wrap screens/feature roots
<ErrorBoundary fallback={<ErrorFallback />}>
  <FeatureScreen />
</ErrorBoundary>

// TanStack Query — always handle both states
const { data, isLoading, isError } = useQuery(...)
if (isLoading) return <LoadingState />;
if (isError) return <ErrorState />;
```

## Rules

- Never expose raw Prisma errors or stack traces in API responses
- Always log errors with `Logger` (API) — never `console.log`
- Never log sensitive data (tokens, national ID numbers, passwords)

## Resources

| Topic | File | When to Use |
|-------|------|------------|
| Prisma error codes (P-codes) | [`prisma-errors.md`](resources/prisma-errors.md) | Mapping DB errors to HTTP responses |
| Custom NestJS exceptions | [`custom-exceptions.md`](resources/custom-exceptions.md) | Domain-specific error types |
| Global exception filter | [`exception-filter.md`](resources/exception-filter.md) | Consistent error response format |
| Result type pattern | [`result-pattern.md`](resources/result-pattern.md) | Operations that can fail gracefully |
