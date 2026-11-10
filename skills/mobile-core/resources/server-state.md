# TanStack Query — Server State

## Basic Query

```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// Always handle isLoading + isError
function TripsScreen() {
  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['trips'],
    queryFn: () => api.trips.getAll(),
  });

  if (isLoading) return <LoadingState />;
  if (isError) return <ErrorState message={error.message} />;

  return <FlatList data={data} ... />;
}
```

## Query Keys Convention

```typescript
// Centralize query keys to avoid typos
export const queryKeys = {
  trips: {
    all: ['trips'] as const,
    detail: (id: string) => ['trips', id] as const,
    byUser: (userId: string) => ['trips', 'user', userId] as const,
  },
  policies: {
    all: ['policies'] as const,
    active: ['policies', 'active'] as const,
  },
};
```

## Mutations

```tsx
function CreateTripButton() {
  const queryClient = useQueryClient();

  const { mutate, isPending } = useMutation({
    mutationFn: (data: CreateTripInput) => api.trips.create(data),
    onSuccess: () => {
      // Invalidate to refetch list
      queryClient.invalidateQueries({ queryKey: queryKeys.trips.all });
    },
    onError: (error) => {
      // Show toast / error message
    },
  });

  return (
    <Button onPress={() => mutate({ region: 'EUROPE' })} disabled={isPending}>
      {isPending ? 'Starting...' : 'Start Trip'}
    </Button>
  );
}
```

## Optimistic Updates

```tsx
const { mutate } = useMutation({
  mutationFn: api.trips.end,
  onMutate: async (tripId) => {
    await queryClient.cancelQueries({ queryKey: queryKeys.trips.all });
    const previous = queryClient.getQueryData(queryKeys.trips.all);
    queryClient.setQueryData(queryKeys.trips.all, (old: Trip[]) =>
      old.filter(t => t.id !== tripId),
    );
    return { previous };
  },
  onError: (err, tripId, context) => {
    queryClient.setQueryData(queryKeys.trips.all, context?.previous);
  },
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.trips.all });
  },
});
```
