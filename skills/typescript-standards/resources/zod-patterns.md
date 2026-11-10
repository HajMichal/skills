# Zod Schema Patterns

## Basic Schema (conventions)

```typescript
// Schema name: camelCase + Schema suffix
const createTripSchema = z.object({
  userId: z.string().uuid(),
  region: z.enum(TravelRegion),       // z.enum accepts Prisma enums (zod v4)
  startedAt: z.coerce.date(),                // coerce string → Date
  notes: z.string().max(500).optional(),
});

// Derive type — never duplicate
type CreateTripInput = z.infer<typeof createTripSchema>;

// DTO for NestJS
export class CreateTripDto extends createZodDto(createTripSchema) {}
```

## Composition

```typescript
// Base schema
const baseTripSchema = z.object({ region: z.enum(TravelRegion) });

// Create = base + required fields
const createTripSchema = baseTripSchema.extend({ userId: z.string().uuid() });

// Update = all optional
const updateTripSchema = createTripSchema.partial();

// Response = pick specific fields
const tripResponseSchema = createTripSchema.pick({ region: true, userId: true })
  .extend({ id: z.string().uuid(), createdAt: z.date() });
```

## Custom Validation

```typescript
const ninSchema = z.string()
  .length(11, 'NIN must be exactly 11 characters')
  .refine(val => /^\d+$/.test(val), 'NIN must be numeric');

// Cross-field validation
const dateRangeSchema = z.object({
  startDate: z.coerce.date(),
  endDate: z.coerce.date(),
}).refine(data => data.endDate > data.startDate, {
  message: 'End date must be after start date',
  path: ['endDate'],
});
```

## Transforms

```typescript
const normalizedEmailSchema = z.string()
  .email()
  .transform(val => val.toLowerCase().trim());

const amountSchema = z.number()
  .positive()
  .transform(val => Math.round(val * 100)); // store as cents
```

## Swagger Descriptions

```typescript
const userSchema = z.object({
  email: z.string().email().describe('User email address'),
  nin: z.string().length(11).describe('National Identity Number (encrypted at rest)'),
});
```
