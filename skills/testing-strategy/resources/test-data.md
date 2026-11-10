# Test Data Factories

## Object Factories (no DB)

```typescript
// test/factories/trip.factory.ts
import { TravelRegion } from 'generated/prisma/client';

export function makeTripDto(overrides: Partial<CreateTripDto> = {}): CreateTripDto {
  return {
    userId: 'user-test-1',
    region: TravelRegion.EUROPE,
    ...overrides,
  };
}

export function makeTripResponse(overrides: Partial<TripResponseDto> = {}): TripResponseDto {
  return {
    id: `trip-${Math.random().toString(36).slice(2)}`,
    userId: 'user-test-1',
    region: TravelRegion.EUROPE,
    startedAt: new Date('2024-01-01T10:00:00Z'),
    endedAt: null,
    createdAt: new Date('2024-01-01T10:00:00Z'),
    ...overrides,
  };
}
```

## DB Seeders (for integration / e2e tests)

```typescript
// test/seeders/user.seeder.ts
export async function seedTestUser(prisma: PrismaService, overrides = {}) {
  return prisma.user.upsert({
    where: { idNumber: encrypt(TEST_NIN) },
    update: {},
    create: {
      idNumber: encrypt(TEST_NIN),
      email: 'test@example.com',
      ...overrides,
    },
  });
}

export async function seedTestTrip(prisma: PrismaService, userId: string, overrides = {}) {
  return prisma.trip.create({
    data: {
      userId,
      region: TravelRegion.EUROPE,
      startedAt: new Date(),
      ...overrides,
    },
  });
}
```

## Usage Pattern

```typescript
beforeAll(async () => {
  testUser = await seedTestUser(prisma);
  testTrip = await seedTestTrip(prisma, testUser.id, { region: 'USA' });
});

afterAll(async () => {
  await prisma.trip.deleteMany({ where: { userId: testUser.id } });
  await prisma.user.delete({ where: { id: testUser.id } });
});
```
