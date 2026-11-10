# Naming Examples

## Variables

```typescript
// ❌ Unclear
const d = new Date();
const arr = users.filter(u => u.active);
const x = calculateAmount(base, rate);

// ✅ Reveals intent
const expiresAt = new Date();
const activeUsers = users.filter(u => u.active);
const billedAmount = calculateAmount(basePrice, taxRate);
```

## Booleans

```typescript
// ❌ No is/has prefix
const loading = true;
const error = false;
const premium = user.plan === 'PREMIUM';

// ✅ Reads naturally
const isLoading = true;
const hasError = false;
const isPremiumUser = user.plan === 'PREMIUM';
```

## Functions

```typescript
// ❌ Vague — what does it "do"?
function process(data: Trip): void {}
function handle(event: LocationEvent): void {}
function check(policy: Policy): boolean {}

// ✅ Verb + noun = action
function calculateBillingAmount(trip: Trip): number {}
function recordLocationEvent(event: LocationEvent): void {}
function isPolicyCoveringRegion(policy: Policy, region: TravelRegion): boolean {}
```

## Constants

```typescript
// ❌ Magic numbers
if (nin.length !== 11) throw new Error('Invalid NIN');
const tier = amount > 5000 ? 'premium' : 'standard';

// ✅ Named constants
const NIN_LENGTH = 11;
const PREMIUM_THRESHOLD_EUR = 5000;
if (nin.length !== NIN_LENGTH) throw new Error('Invalid NIN');
```

## Classes and Types

```typescript
// ❌ Too generic
class Manager {}
type Data = Record<string, unknown>;
interface Info {}

// ✅ Domain-specific
class TripBillingCalculator {}
type RegionVisitDuration = { region: TravelRegion; durationMs: number };
interface UpstreamPolicyDetails {}
```
