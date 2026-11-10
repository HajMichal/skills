---
name: api-development
description: NestJS feature modules, controllers, services, repositories, and DTOs for a NestJS API (Zod DTOs, Prisma repositories)
---

# API Development — Quick Reference

## `src/` Layout

```
src/
  core/          infrastructure every feature may use (common, email, file-storage, health, …)
  integrations/  external systems, one folder each (clients + wire DTOs), e.g. payments/, crm/
  modules/       domain features (user, order, payment, …)
  admin/         admin API (if any), one folder per screen
  scheduler/     cron jobs; wiring layer like app.module, may import from any group
```

Dependency direction: `admin → modules → integrations / core`. `core` and `integrations` never
import `modules` or `admin`; `modules` never imports `admin`. Enforce it with a boundary test
(e.g. `architecture.boundary.spec.ts`). A module that only reads another
feature's data imports its `*DataModule` (repositories only), not the feature module.

## Feature Slice Structure

```
src/modules/[feature]/
  [feature].module.ts
  [feature].controller.ts    ← thin; no business logic
  [feature].service.ts       ← all business logic
  [feature].repository.ts    ← all Prisma calls
  dto/
    create-[feature].dto.ts
    update-[feature].dto.ts
    [feature]-response.dto.ts
  index.ts                   ← barrel export
```

## Key Rules

- **Controllers** — route/HTTP only; delegate to service immediately
- **Services** — business logic only; never call Prisma directly
- **Repositories** — all Prisma calls; let Prisma errors reach the global `ApiExceptionFilter`; catch locally only for domain-specific cases
- **Never** `new PrismaClient()` — inject `PrismaService`

## DTOs with Zod (nestjs-zod)

```typescript
// schemas name start with lowercase
const createUserSchema = z.object({
  email: z.string().email().describe('User email'),
  nin: z.string().min(11).max(11),
});

// DTO extends createZodDto
export class CreateUserDto extends createZodDto(createUserSchema) {}

// Response DTO — always create; never expose raw Prisma models
export class UserResponseDto extends createZodDto(userResponseSchema) {}
```

## Global Providers (auto-applied)

| Provider | Purpose | Override |
|----------|---------|---------|
| `PrismaExceptionFilter` (`APP_FILTER`) | Map Prisma errors to HTTP responses | local try/catch |
| `ZodValidationPipe` | Validate + coerce request bodies | — |
| `ZodSerializerInterceptor` | Strip non-DTO fields from responses | `@SerializeOptions` |
| `ThrottlerGuard` | Rate limiting (`short` + `long`) | `@SkipAllThrottles()` — bare `@SkipThrottle()` is a no-op with named throttlers |
| `JwtAuthGuard` | JWT on all routes | `@Public()` decorator |
| `MaintenanceGuard` | Block requests during maintenance mode | `@AllowDuringMaintenance()` |
| `AppVersionGateGuard` | Block outdated mobile app versions | `@Public()` routes skip it |

`AppLoggerMiddleware` is applied to all routes in `AppModule.configure`.

## Resources

| Topic | File | When to Use |
|-------|------|------------|
| Module setup & DI | [`modules-di.md`](resources/modules-di.md) | New feature module, shared modules |
| Controllers & HTTP | [`controllers.md`](resources/controllers.md) | Route decorators, versioning, guards |
| DTOs & Zod | [`dtos-zod.md`](resources/dtos-zod.md) | Schema composition, transforms, enums |
| Guards, Pipes, Interceptors | [`middleware-stack.md`](resources/middleware-stack.md) | Auth, RBAC, cross-cutting concerns |
| Swagger docs | [`swagger.md`](resources/swagger.md) | Documenting endpoints with nestjs-zod |
| Logging & errors | [`logging-errors.md`](resources/logging-errors.md) | NestJS Logger, custom exceptions |
