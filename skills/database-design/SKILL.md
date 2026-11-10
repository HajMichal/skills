---
name: database-design
description: Prisma schema conventions, migrations, and query patterns for a TypeScript monorepo
---

# Database Design — Quick Reference

## Model Conventions

Every model must have:
```prisma
id        String   @id @default(uuid())
createdAt DateTime @default(now())
updatedAt DateTime @updatedAt
deletedAt DateTime?               // user/business records only — see Hard Rules
```

- `@@index` on every field used in `WHERE` clauses
- Enums defined in schema; use `z.enum(PrismaEnum)` (zod v4) in Zod DTOs
- Meaningful domain-driven model names

## Hard Rules

- **Never hard-delete user or business records** — soft delete via `deletedAt`. Tokens, sessions
  and cleanup/audit tables (e.g. refresh tokens, device sessions, password resets) have no
  `deletedAt` and are hard-deleted with `deleteMany`
- **Always filter** `WHERE deletedAt IS NULL` on reads of models that have `deletedAt`
- **Never** call `new PrismaClient()` — only inject `PrismaService`
- **Prisma calls only in repositories** — never in services or controllers

## Migrations

```bash
# from the API app directory
pnpm exec prisma migrate dev --name describe_change   # create + apply
pnpm exec prisma generate                             # regenerate client (always after schema edit)
pnpm exec prisma migrate status
```

- Descriptive migration names (e.g., `add_region_visit_index`)
- Never modify existing migrations
- Review SQL diff before applying in production

## Error Handling in Repositories

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

## Resources

| Topic | File | When to Use |
|-------|------|------------|
| Schema patterns & relations | [`schema-patterns.md`](resources/schema-patterns.md) | New models, FK relations, enums |
| Transactions | [`transactions.md`](resources/transactions.md) | Multi-step atomic operations |
| Query optimization | [`query-optimization.md`](resources/query-optimization.md) | Pagination, N+1, select/include |
| Prisma error codes | [`prisma-errors.md`](resources/prisma-errors.md) | Mapping P-codes to HTTP exceptions |
