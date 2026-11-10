---
name: typescript-standards
description: TypeScript naming conventions, type system rules, and essential patterns for a TypeScript monorepo
---

# TypeScript Standards — Quick Reference

## Naming Conventions

| What                | Case          | Example                             |
| ------------------- | ------------- | ----------------------------------- |
| Type / Interface    | PascalCase    | `UserProfile`, `CreateTripInput`    |
| Variable / Function | camelCase     | `userData`, `isLoading`, `hasError` |
| Constant            | UPPER_CASE    | `MAX_RETRIES`, `TRAVEL_REGIONS`     |
| Zod Schema          | Schema suffix | `createTripSchema`                  |
| Component props     | Props suffix  | `ButtonProps`, `TripCardProps`      |
| Enum (Prisma)       | PascalCase    | `TravelRegion.EUROPE`               |

## Type System Rules

- **Interface** for object shapes (extendable, readable)
- **Type** for unions, intersections, mapped types
- Never `any` — use `unknown` for truly unknown values
- Derive TS types from Zod schemas: `type T = z.infer<typeof tSchema>`

```typescript
// ✅ Interface for shapes
interface CreateTripInput {
  userId: string;
  startedAt: Date;
}

// ✅ Type for unions
type TravelRegion = 'NORDICS' | 'EUROPE' | 'USA' | 'REST_OF_WORLD';

// ✅ Zod schema — derive type, never duplicate
const createTripSchema = z.object({ userId: z.string(), startedAt: z.coerce.date() });
type CreateTripInput = z.infer<typeof createTripSchema>;
```

## Key Patterns

- **Prefer `readonly`** for immutable properties
- **Prefer `async/await`** over `.then()` chains
- **Type guards** over assertions (`as`)
- **Explicit return types** on all exported functions

## Resources

| Topic                  | File                                             | When to Use                                    |
| ---------------------- | ------------------------------------------------ | ---------------------------------------------- |
| Advanced type patterns | [`type-patterns.md`](resources/type-patterns.md) | Discriminated unions, generics, Result types   |
| Zod schema patterns    | [`zod-patterns.md`](resources/zod-patterns.md)   | Composition, `.refine()`, `z.enum(PrismaEnum)` |
| Error types            | [`error-types.md`](resources/error-types.md)     | Custom domain errors, typed catch              |
