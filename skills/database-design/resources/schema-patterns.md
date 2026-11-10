# Prisma Schema Patterns

## Base Model (every model must have this)

```prisma
model Trip {
  id        String    @id @default(uuid())
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  deletedAt DateTime?              // soft delete

  // domain fields...
  userId    String
  region    TravelRegion

  // relations
  user         User          @relation(fields: [userId], references: [id])
  regionVisits RegionVisit[]

  // indexes on all WHERE/ORDER BY fields
  @@index([userId])
  @@index([deletedAt])
  @@index([region, createdAt])
}
```

## Location input vs interpreted state

`LocationEvent` is the raw log of geofence/location signals as received from the device.
`RegionVisit` rows and `Trip` updates are the interpreted result derived from it. Fix
interpretation bugs in the derivation, never by rewriting what the device reported.

## Enum Definition

```prisma
enum TravelRegion {
  NORDICS       @map("1")
  EUROPE        @map("2")
  USA           @map("3")
  REST_OF_WORLD @map("4")
}
```

Use `z.enum(TravelRegion)` in Zod schemas — never redefine as string union.

## Soft Delete Pattern

```typescript
// In repository — always filter deleted records
async findAll(userId: string) {
  return this.prisma.trip.findMany({
    where: { userId, deletedAt: null },
    orderBy: { createdAt: 'desc' },
  });
}

// Soft delete
async remove(id: string) {
  return this.prisma.trip.update({
    where: { id },
    data: { deletedAt: new Date() },
  });
}
```

## Relations

```prisma
// One-to-many
model User {
  id      String   @id @default(uuid())
  trips   Trip[]
  policies Policy[]
}

model Trip {
  userId  String
  user    User    @relation(fields: [userId], references: [id])
}

// Many-to-many (explicit join table for extra fields)
model PolicyAddOn {
  id       String @id @default(uuid())
  policyId String
  addOnId  String
  price    Decimal
  policy   Policy @relation(fields: [policyId], references: [id])
  addOn    AddOn  @relation(fields: [addOnId], references: [id])
  @@unique([policyId, addOnId])
}
```

## Indexing Strategy

```prisma
// Single field
@@index([userId])

// Composite (order matters — most selective first)
@@index([userId, deletedAt])

// Unique constraint
@@unique([userId, policyNumber])
```
