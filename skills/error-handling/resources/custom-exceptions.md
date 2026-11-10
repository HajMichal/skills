# Custom NestJS Exceptions

## Base Pattern

```typescript
import { HttpException, HttpStatus } from '@nestjs/common';

export class PolicyExpiredException extends HttpException {
  constructor(policyId: string) {
    super(
      {
        statusCode: HttpStatus.UNPROCESSABLE_ENTITY,
        error: 'POLICY_EXPIRED',
        message: `Policy ${policyId} has expired`,
      },
      HttpStatus.UNPROCESSABLE_ENTITY,
    );
  }
}
```

## Domain-Specific Exceptions

```typescript
// Errors from an upstream system the API depends on
export class UpstreamApiException extends HttpException {
  constructor(detail: string) {
    super({ error: 'UPSTREAM_ERROR', message: detail }, HttpStatus.BAD_GATEWAY);
  }
}

// Business rule violations
export class ActiveTripExistsException extends HttpException {
  constructor(userId: string) {
    super(
      { error: 'ACTIVE_TRIP_EXISTS', message: 'User already has an active trip' },
      HttpStatus.CONFLICT,
    );
  }
}

export class RegionNotCoveredException extends HttpException {
  constructor(region: string) {
    super(
      { error: 'REGION_NOT_COVERED', message: `Region ${region} is not covered by active policy` },
      HttpStatus.UNPROCESSABLE_ENTITY,
    );
  }
}
```

## Usage in Services

```typescript
async startTrip(userId: string, dto: CreateTripDto): Promise<TripResponseDto> {
  const activeTrip = await this.tripsRepository.findActiveByUser(userId);
  if (activeTrip) throw new ActiveTripExistsException(userId);

  const policy = await this.policiesRepository.findActiveByUser(userId);
  if (!policy) throw new NotFoundException('No active policy found');

  return this.tripsRepository.create({ ...dto, userId });
}
```
