# DTOs with Zod & nestjs-zod

## Standard Pattern

```typescript
import { createZodDto } from 'nestjs-zod';
import { z } from 'zod';

// 1. Define schema (camelCase + Schema suffix)
const createPolicySchema = z.object({
  userId: z.string().uuid(),
  region: z.enum(TravelRegion).describe('Region visited'),
  startsAt: z.coerce.date(),
});

// 2. Derive type
type CreatePolicyInput = z.infer<typeof createPolicySchema>;

// 3. Extend createZodDto for NestJS
export class CreatePolicyDto extends createZodDto(createPolicySchema) {}
```

## Three DTO Types Per Feature

```typescript
// create-policy.dto.ts
export class CreatePolicyDto extends createZodDto(createPolicySchema) {}

// update-policy.dto.ts
const updatePolicySchema = createPolicySchema.partial();
export class UpdatePolicyDto extends createZodDto(updatePolicySchema) {}

// policy-response.dto.ts — control what is exposed to clients
const policyResponseSchema = z.object({
  id: z.string().uuid(),
  region: z.enum(TravelRegion),
  startsAt: z.date(),
  createdAt: z.date(),
  // ⚠️ never include: raw NIN, third-party API tokens, internal IDs
});
export class PolicyResponseDto extends createZodDto(policyResponseSchema) {}
```

## Enum Validation

```typescript
import { TravelRegion } from 'generated/prisma/client';

// Always z.enum(PrismaEnum) for Prisma enums (zod v4; nativeEnum is deprecated) — gives Swagger enum docs automatically
region: z.enum(TravelRegion)

// z.enum for string literals
status: z.enum(['ACTIVE', 'EXPIRED', 'CANCELLED'])
```

## Schema Composition

```typescript
// Build on top of existing schemas
const tripWithVisitsSchema = tripResponseSchema.extend({
  regionVisits: z.array(regionVisitResponseSchema),
});

// Pick a subset
const tripSummarySchema = tripResponseSchema.pick({ id: true, region: true, startsAt: true });

// Merge two schemas
const fullUserSchema = userSchema.merge(addressSchema);
```
