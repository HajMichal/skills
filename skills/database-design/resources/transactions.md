# Prisma Transactions

## When to Use

Use transactions when two or more writes must succeed or fail together:

- Creating a policy + initial payment record
- Ending a trip + calculating final billing
- Soft-deleting + archiving related records

## Interactive Transactions (preferred)

```typescript
async createPolicyWithPayment(dto: CreatePolicyWithPaymentDto) {
  return this.prisma.$transaction(async (tx) => {
    // tx is a scoped Prisma client — use instead of this.prisma
    const policy = await tx.policy.create({
      data: { userId: dto.userId, region: dto.region },
    });

    const payment = await tx.payment.create({
      data: { policyId: policy.id, amount: dto.amount, currency: 'EUR' },
    });

    return { policy, payment };
    // if anything throws, entire transaction rolls back
  });
}
```

## Sequential Batch (simpler, no rollback)

```typescript
// Use when queries are independent and you just want a single round-trip
const [users, policies] = await this.prisma.$transaction([
  this.prisma.user.findMany({ where: { deletedAt: null } }),
  this.prisma.policy.findMany({ where: { userId, deletedAt: null } }),
]);
```

## Key Rules

- Inside a transaction, always use the `tx` parameter — not `this.prisma`
- Keep transactions short — long transactions hold DB locks
- Don't call external APIs inside a transaction
- Transactions belong in the repository layer

## Exceptions

External calls inside a transaction are allowed only when the lock is what keeps the external side
effect from running twice. Each exception is documented with its reasoning, and its `timeout` must
outlast the worst case of every external call inside it (the sum of their client timeouts), or a
call that succeeds externally is rolled back locally.

| Flow                                                                   | Why                                                                                                                                                                                    | Doc                                        |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| Example: cancellation withdrawal (`CancellationWithdrawalRepository.runLocked`) | A per-record advisory lock makes a double tap wait instead of calling the external API twice; Postgres releases it if the process dies, so no stale-claim recovery is needed | the feature doc |
