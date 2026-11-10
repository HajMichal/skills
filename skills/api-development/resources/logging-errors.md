# Logging & Custom Exceptions

## NestJS Logger Usage

```typescript
@Injectable()
export class TripsService {
  private readonly logger = new Logger(TripsService.name);

  async create(dto: CreateTripDto): Promise<TripResponseDto> {
    this.logger.log(`Creating trip for user ${dto.userId}`);
    try {
      const trip = await this.tripsRepository.create(dto);
      this.logger.log(`Trip created: ${trip.id}`);
      return trip;
    } catch (e) {
      this.logger.error(`Failed to create trip for user ${dto.userId}`, e instanceof Error ? e.stack : e);
      throw e;
    }
  }
}
```

**Log levels:**
- `logger.log()` — general info
- `logger.warn()` — unexpected but handled situations
- `logger.error()` — always pass the stack trace as second arg
- `logger.debug()` — dev-only (disabled in production)

**Never log:** passwords, tokens, national ID numbers, full request bodies with sensitive data.

## Custom HTTP Exceptions

```typescript
export class PolicyExpiredException extends HttpException {
  constructor(policyId: string) {
    super(
      { message: `Policy ${policyId} has expired`, code: 'POLICY_EXPIRED' },
      HttpStatus.UNPROCESSABLE_ENTITY,
    );
  }
}

export class UpstreamSyncException extends HttpException {
  constructor(detail: string) {
    super(
      { message: 'Upstream sync failed', detail },
      HttpStatus.BAD_GATEWAY,
    );
  }
}
```

## Global Exception Filter

```typescript
@Catch()
export class AllExceptionsFilter implements ExceptionFilter {
  private readonly logger = new Logger(AllExceptionsFilter.name);

  catch(exception: unknown, host: ArgumentsHost): void {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const request = ctx.getRequest<Request>();

    const status = exception instanceof HttpException
      ? exception.getStatus()
      : HttpStatus.INTERNAL_SERVER_ERROR;

    if (status >= 500) {
      this.logger.error(`Unhandled exception: ${request.method} ${request.url}`, exception instanceof Error ? exception.stack : String(exception));
    }

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString(),
      path: request.url,
      message: exception instanceof HttpException
        ? exception.getResponse()
        : 'Internal server error',
    });
  }
}
```
