# React Memoization

## When to Memoize

**Only memoize when you can measure a real problem.** Premature memoization adds complexity with no benefit.

| Use           | When                                                        |
| ------------- | ----------------------------------------------------------- |
| `React.memo`  | Component is provably expensive AND receives stable props   |
| `useMemo`     | Computation is expensive (e.g. sort/filter of large arrays) |
| `useCallback` | Function is passed as a prop to a memoized component        |

## React.memo

```tsx
// ✅ Justified — TripCard renders a complex UI and appears in a long list
const TripCard = React.memo(({ trip, onPress }: TripCardProps) => {
  return <Pressable onPress={() => onPress(trip.id)}>{/* ... */}</Pressable>;
});

// ❌ Not needed — trivial component, no expensive renders
const Label = React.memo(({ text }: { text: string }) => <Text>{text}</Text>);
```

## useMemo

```tsx
// ✅ Justified — sorting a list on every render
const sortedTrips = useMemo(
  () => [...trips].sort((a, b) => b.startedAt.getTime() - a.startedAt.getTime()),
  [trips] // only re-sort when trips array changes
);

// ❌ Not needed — cheap computation
const fullName = useMemo(() => `${firstName} ${lastName}`, [firstName, lastName]);
// Just write: const fullName = `${firstName} ${lastName}`;
```

## useCallback

```tsx
// ✅ Justified — passed to a memoized child
const handleTripPress = useCallback((tripId: string) => {
  router.push(`/trips/${tripId}`);
}, []); // stable reference

<TripCard
  trip={trip}
  onPress={handleTripPress}
/>;

// ❌ Not needed if the child isn't memoized
// The callback reference stability is irrelevant if TripCard re-renders anyway
```

## React Native Specific

Re-renders cost more on mobile, so prioritize:

1. `FlatList` with `keyExtractor` (and `getItemLayout` for fixed-size items)
2. `memo` on hot, frequently re-rendered components (e.g. `Keypad`, `BiometricRetryButton`)
3. `useCallback` for `renderItem` and handlers passed to list items
