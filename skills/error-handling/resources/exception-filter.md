# Global Exception Filter

## What Is Registered

`app.module.ts` registers one global filter:

```typescript
{ provide: APP_FILTER, useClass: PrismaExceptionFilter },
```

`PrismaExceptionFilter` (`src/core/common/filters/prisma-exception.filter.ts`) is
`@Catch(Prisma.PrismaClientKnownRequestError)` only. It maps known codes via `PRISMA_ERROR_MAP`
(logged as `warn`) and returns 500 `Database operation failed` for anything else (logged as
`error` with the stack). See
[database-design/resources/prisma-errors.md](../../database-design/resources/prisma-errors.md)
for the code table.

`HttpException`s (and subclasses) thrown from services/repositories are handled by NestJS's
built-in exception layer; unknown errors become a generic 500.

## Prisma Error Response Shape

```json
{
  "statusCode": 404,
  "message": "Record not found",
  "timestamp": "2024-01-01T10:00:00.000Z",
  "path": "/trips/trip-123"
}
```

## Adding a Mapping

Add the code to `PRISMA_ERROR_MAP` rather than catching it in every repository. Use a local
catch only when the response must be domain-specific.
