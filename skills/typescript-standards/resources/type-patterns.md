# TypeScript Type Patterns

## Discriminated Unions

Use a literal `type` field to narrow safely:

```typescript
type ApiResponse<T> =
  | { status: 'success'; data: T }
  | { status: 'error'; message: string; code: number };

function handle(res: ApiResponse<User>) {
  if (res.status === 'success') {
    console.log(res.data); // TS knows data exists
  } else {
    console.log(res.message); // TS knows message exists
  }
}
```

## Result Type

For operations that can fail without throwing:

```typescript
type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

async function parseNIN(raw: string): Promise<Result<string>> {
  if (raw.length !== 11) return { ok: false, error: new Error('Invalid NIN length') };
  return { ok: true, value: raw };
}
```

## Generics

```typescript
// Constrained generic
function getById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}

// Utility types
type PartialUser = Partial<User>;
type RequiredConfig = Required<Config>;
type ReadonlyPolicy = Readonly<Policy>;
type UserKeys = keyof User;
```

## Type Guards

```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

function isPrismaError(e: unknown): e is Prisma.PrismaClientKnownRequestError {
  return e instanceof Prisma.PrismaClientKnownRequestError;
}
```

## Mapped Types

```typescript
type Nullable<T> = { [K in keyof T]: T[K] | null };
type Optional<T> = { [K in keyof T]?: T[K] };
```
