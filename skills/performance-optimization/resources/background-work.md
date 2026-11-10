# Background Work

No queue (no Bull/Redis). Two mechanisms, both in-process:

## Scheduled jobs — `@nestjs/schedule`

All crons live in `src/scheduler/*.scheduler.ts`, registered in `scheduler.module.ts`. A scheduler is thin: log, delegate to a domain service, log one aggregate outcome line.

```typescript
@Injectable()
export class MonthlyChargeScheduler {
  private readonly logger = new Logger(MonthlyChargeScheduler.name);

  constructor(private readonly paymentService: PaymentService) {}

  @Cron(CronExpression.EVERY_DAY_AT_7AM)
  async createMonthlyCharges(): Promise<void> {
    const outcomes = await this.paymentService.createMonthlyChargesForAllActive();
    this.logger.log(`Monthly charge job finished: scanned=${outcomes.length} ...`);
  }
}
```

- Pass `{ timeZone: '<business time zone>' }` when the time is user-facing (emails, pushes, billing dates)
- Multiple API instances run every cron — guard jobs that must run once with `AdvisoryLockRepository.runExclusive(lockKey, fn)` (Postgres advisory lock)
- Admin-configurable notification sweeps go through `GuardedSweepRunner` (`runScheduled` / `runNow`), which combines schedule resolution + advisory lock + `SweepOutcome`

## Domain events — `@nestjs/event-emitter`

Use for side effects that must not block or fail the originating request (emails, CRM sync, invoices, admin alerts).

```typescript
// emitter (service)
this.eventEmitter.emit('trip.departed', { userId, tripId, ... });

// listener (*.listener.ts or service)
@OnEvent('trip.departed')
async handleTripDeparted(payload: TripDepartedEvent): Promise<void> { ... }
```

- Event names are `domain.past-tense` (`policy.issued`, `charge.succeeded`, `user.deleted`)
- Listeners catch and log their own errors — a throwing listener must not surface to the emitter
