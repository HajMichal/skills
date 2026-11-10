# Result Pattern (No-Throw Operations)

Use when you want callers to handle failure explicitly without try-catch.

## Definition

```typescript
// src/core/common/types/result.type.ts
export type Result<T, E = Error> =
  | { ok: true; value: T }
  | { ok: false; error: E };

export function ok<T>(value: T): Result<T, never> {
  return { ok: true, value };
}

export function err<E>(error: E): Result<never, E> {
  return { ok: false, error };
}
```

## Usage in Services

```typescript
async validateCoverage(
  userId: string,
  region: TravelRegion,
): Promise<Result<Policy, 'NO_POLICY' | 'REGION_NOT_COVERED'>> {
  const policy = await this.policiesRepository.findActiveByUser(userId);
  if (!policy) return err('NO_POLICY');
  if (!policy.coveredRegions.includes(region)) return err('REGION_NOT_COVERED');
  return ok(policy);
}

// Caller handles explicitly
const coverage = await this.validateCoverage(userId, region);
if (!coverage.ok) {
  switch (coverage.error) {
    case 'NO_POLICY': throw new NotFoundException('No active policy');
    case 'REGION_NOT_COVERED': throw new RegionNotCoveredException(region);
  }
}
// TS now knows coverage.value is Policy
```

## When to Use vs Exceptions

| Use Result | Use Exception |
|-----------|--------------|
| Expected failure states (validation, business rules) | Unexpected errors (DB down, integration failure) |
| Caller needs to handle multiple failure cases | Single failure mode |
| Internal service methods | Public API boundaries (controllers) |
