# Prisma Error Handling

See [database-design/resources/prisma-errors.md](../../database-design/resources/prisma-errors.md)
for the P-code → HTTP mapping applied by the global `PrismaExceptionFilter`.

## Default: Let It Bubble

```typescript
// ✅ No try/catch — P2025 becomes 404, P2002 becomes 409 via the global filter
async findByIdOrThrow(id: string): Promise<Trip> {
  return this.prisma.trip.findUniqueOrThrow({ where: { id, deletedAt: null } });
}
```

## Catch Locally Only for Domain-Specific Cases

```typescript
import { Prisma } from 'generated/prisma/client';

// user.repository.ts — the client needs to know which field conflicted
try {
  return await this.prisma.user.update({ where: { id: userId }, data });
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError && error.code === 'P2002') {
    const target = error.meta?.target;
    const fields = Array.isArray(target) ? target.map(String) : target ? [String(target)] : [];
    throw new ProfileFieldConflictException(fields);
  }
  throw error; // unhandled codes → global filter
}
```

- Never wrap every call in try/catch just to re-throw a generic HTTP exception — the filter already does that
- Always re-throw errors you don't handle unchanged
