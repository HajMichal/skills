# Prisma Query Optimization

## Pagination

```typescript
// Offset pagination (simple, less efficient for deep pages)
async findAll(page: number, limit: number) {
  return this.prisma.trip.findMany({
    where: { deletedAt: null },
    skip: (page - 1) * limit,
    take: limit,
    orderBy: { createdAt: 'desc' },
  });
}

// Cursor pagination (efficient for infinite scroll)
async findAfterCursor(cursor?: string, limit = 20) {
  return this.prisma.trip.findMany({
    where: { deletedAt: null },
    take: limit,
    ...(cursor && { skip: 1, cursor: { id: cursor } }),
    orderBy: { createdAt: 'desc' },
  });
}
```

## Avoid N+1

```typescript
// ❌ N+1 — queries DB once per trip
const trips = await this.prisma.trip.findMany({ where: { userId } });
for (const trip of trips) {
  trip.visits = await this.prisma.regionVisit.findMany({ where: { tripId: trip.id } });
}

// ✅ Single query with include
const trips = await this.prisma.trip.findMany({
  where: { userId, deletedAt: null },
  include: { regionVisits: true },
});

// ✅ Or batch fetch
const tripIds = trips.map(t => t.id);
const visits = await this.prisma.regionVisit.findMany({
  where: { tripId: { in: tripIds }, deletedAt: null },
});
```

## Select Only Needed Fields

```typescript
// ❌ Over-fetch
const user = await this.prisma.user.findUnique({ where: { id } });

// ✅ Select only what the response DTO needs
const user = await this.prisma.user.findUnique({
  where: { id },
  select: { id: true, email: true, createdAt: true },
});
```

## Count + Data in One Round-Trip

```typescript
const [items, total] = await this.prisma.$transaction([
  this.prisma.policy.findMany({ where, skip, take }),
  this.prisma.policy.count({ where }),
]);
return { items, total, page, limit };
```
