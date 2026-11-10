# Encapsulation Patterns

## Hide Implementation — Expose Interface

```typescript
// ❌ Leaks Prisma model shape to caller
async findUser(id: string): Promise<User> {  // User is Prisma type
  return this.prisma.user.findUniqueOrThrow({ where: { id } });
}

// ✅ Return the response DTO — callers don't know about Prisma
async findUser(id: string): Promise<UserResponseDto> {
  const user = await this.prisma.user.findUniqueOrThrow({ where: { id } });
  return plainToInstance(UserResponseDto, user);
}
```

## Extract Nested Conditionals

```typescript
// ❌ Nested conditions — hard to read
async processTrip(trip: Trip, user: User): Promise<void> {
  if (trip.status === 'ACTIVE') {
    if (user.policy) {
      if (user.policy.region === trip.region) {
        await this.billUser(user.id, trip);
      }
    }
  }
}

// ✅ Extracted into well-named guard functions
async processTrip(trip: Trip, user: User): Promise<void> {
  if (!this.isTripBillable(trip, user)) return;
  await this.billUser(user.id, trip);
}

private isTripBillable(trip: Trip, user: User): boolean {
  return trip.status === 'ACTIVE'
    && user.policy != null
    && user.policy.region === trip.region;
}
```

## Repository as Encapsulation Boundary

```typescript
// ❌ Service knows about DB internals
const user = await this.prisma.user.findFirst({
  where: { idNumber: { equals: encrypted, mode: 'insensitive' }, deletedAt: null },
});

// ✅ Repository hides the query
const user = await this.usersRepository.findByIdNumber(nin);
// Callers don't know about encryption, soft-delete filter, or case sensitivity
```

## Single Source of Truth

```typescript
// ❌ Magic string duplicated in multiple places
if (region === 'EUROPE') { ... }
if (region === 'EUROPE') { ... }  // somewhere else

// ✅ Defined once
const TRAVEL_REGIONS = {
  NORDICS: 'NORDICS',
  EUROPE: 'EUROPE',
  USA: 'USA',
  REST_OF_WORLD: 'REST_OF_WORLD',
} as const;

// Or better — use Prisma enum
import { TravelRegion } from 'generated/prisma/client';
if (region === TravelRegion.EUROPE) { ... }
```
