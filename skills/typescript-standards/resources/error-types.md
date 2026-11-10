# TypeScript Error Types

## Custom Domain Errors

```typescript
export class DomainError extends Error {
  constructor(
    message: string,
    public readonly code: string,
  ) {
    super(message);
    this.name = this.constructor.name;
  }
}

export class InvalidNINError extends DomainError {
  constructor(nin: string) {
    super(`Invalid NIN format: ${nin}`, 'INVALID_NIN');
  }
}
```

## Typed Catch Clauses

```typescript
try {
  await processPayment(amount);
} catch (e) {
  if (e instanceof Prisma.PrismaClientKnownRequestError) {
    // handle DB error
  } else if (e instanceof InvalidNINError) {
    // handle domain error
  } else {
    throw e; // re-throw unknown errors
  }
}
```

## Result Type (no-throw pattern)

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

// Callers don't need try-catch
async function validateNIN(nin: string): Promise<Result<string, InvalidNINError>> {
  if (nin.length !== 11) return { ok: false, error: new InvalidNINError(nin) };
  return { ok: true, value: nin };
}

// Usage
const result = await validateNIN(input);
if (!result.ok) {
  // handle error without try-catch
}
```

## Error in async contexts

```typescript
// Always handle unhandled rejections explicitly
async function riskyOperation(): Promise<void> {
  const result = await someAsyncThing().catch((e: unknown) => {
    if (e instanceof KnownError) return null;
    throw e;
  });
}
```
