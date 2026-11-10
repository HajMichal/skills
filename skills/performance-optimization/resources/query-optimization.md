# Query Optimization

See also: [database-design/resources/query-optimization.md](../../database-design/resources/query-optimization.md) for Prisma-specific patterns.

## N+1 Detection

```typescript
// ❌ N+1 — 1 query for trips + N queries for visits
const trips = await tripsRepo.findAll(userId);
for (const trip of trips) {
  trip.visits = await visitsRepo.findByTrip(trip.id); // N queries
}

// ✅ 2 queries total
const trips = await tripsRepo.findAll(userId);
const tripIds = trips.map((t) => t.id);
const visits = await visitsRepo.findByTripIds(tripIds);
// Map visits back to trips in memory

// ✅ Or 1 query with Prisma include
const trips = await prisma.trip.findMany({
  where: { userId, deletedAt: null },
  include: { regionVisits: { where: { deletedAt: null } } },
});
```

## Pagination Always

Repositories take `skip` / `take` and run `findMany` + `count` in one `$transaction` (see `admin-audit.repository.ts` `listGlobal`):

```typescript
async listGlobal(skip: number, take: number, filters: ListActionLogsQuery) {
  const where: Prisma.AdminActionLogWhereInput = { /* built from filters */ };
  const [items, total] = await this.prisma.$transaction([
    this.prisma.adminActionLog.findMany({ where, orderBy, skip, take }),
    this.prisma.adminActionLog.count({ where }),
  ]);
  return { items, total };
}
```

## Index Verification

Before adding a query with a new WHERE clause, check if an index exists:

```prisma
// Add index to schema if missing
@@index([userId, deletedAt])      // composite index
@@index([createdAt])              // for ORDER BY
```

Then generate the migration with `prisma migrate dev` (see the API app's `CLAUDE.md`) — never hand-write migration SQL.
