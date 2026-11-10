# Prisma Error Codes → HTTP Responses

Prisma errors are mapped globally by `PrismaExceptionFilter`
(e.g. `src/core/common/filters/prisma-exception.filter.ts`, registered as `APP_FILTER` in
`app.module.ts`). Repositories don't need try/catch for these by default.

## Global Mapping (`PRISMA_ERROR_MAP`)

| Code | Meaning | HTTP status |
|------|---------|-------------|
| `P2002` | Unique constraint violation | 409 Conflict |
| `P2003` | Foreign key constraint violation | 400 Bad Request |
| `P2011` | Null constraint violation | 400 Bad Request |
| `P2014` | Relation violation | 400 Bad Request |
| `P2015` | Related record not found | 400 Bad Request |
| `P2020` | Value out of range | 400 Bad Request |
| `P2023` | Inconsistent column data | 400 Bad Request |
| `P2025` | Record not found (`findUniqueOrThrow`, `update`, `delete`) | 404 Not Found |
| any other | — | 500 `Database operation failed` (logged with stack) |

Responses use generic messages — raw Prisma errors never reach the client.

## Local Catch — Domain-Specific Cases Only

Catch locally only when the caller needs a domain-specific exception or message
(e.g. which profile field conflicted). Import `Prisma` from the generated client:

```typescript
import { Prisma } from 'generated/prisma/client';

// user.repository.ts
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

Always re-throw what you don't handle so it reaches the global filter.
