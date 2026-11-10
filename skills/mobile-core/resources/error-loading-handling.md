# Error & Loading Handling

## TanStack Query — Always Handle Both States

```tsx
function TripsScreen() {
  const { data, isLoading, isError, error, refetch } = useQuery({
    queryKey: queryKeys.trips.all,
    queryFn: () => api.trips.getAll(),
  });

  if (isLoading) return <LoadingState />;
  if (isError)
    return (
      <ErrorState
        message={error.message}
        onRetry={refetch}
      />
    );

  return <TripList trips={data} />;
}
```

## Reusable State Components

```tsx
// components/ui/loading-state.tsx
export function LoadingState({ message }: { message?: string }) {
  const { t } = useTranslation();
  return (
    <View className='flex-1 items-center justify-center gap-3'>
      <ActivityIndicator
        size='large'
        color='#2db0a9'
      />
      <Text className='text-sm text-primary/60'>{message ?? t('common.loading')}</Text>
    </View>
  );
}

// components/ui/error-state.tsx
export function ErrorState({ message, onRetry }: { message?: string; onRetry?: () => void }) {
  const { t } = useTranslation();
  return (
    <View className='flex-1 items-center justify-center gap-4 px-8'>
      <Text className='text-center text-primary'>{message ?? t('common.errorGeneric')}</Text>
      {onRetry && (
        <Button
          onPress={onRetry}
          variant='outline'
        >
          <Text>{t('common.retry')}</Text>
        </Button>
      )}
    </View>
  );
}

// components/ui/empty-state.tsx
export function EmptyState({ message }: { message: string }) {
  return (
    <View className='flex-1 items-center justify-center px-8'>
      <Text className='text-center text-primary/60'>{message}</Text>
    </View>
  );
}
```

## Error Boundaries

Wrap feature roots — not individual components:

```tsx
// app/(main)/_layout.tsx
import { ErrorBoundary } from 'react-error-boundary';

export default function AppLayout() {
  return (
    <ErrorBoundary FallbackComponent={AppErrorFallback}>
      <Slot />
    </ErrorBoundary>
  );
}

function AppErrorFallback({ error, resetErrorBoundary }: FallbackProps) {
  const { t } = useTranslation();
  return (
    <SafeAreaView className='flex-1 items-center justify-center gap-4 px-8'>
      <Text className='text-center font-albertsans-semibold text-xl text-primary'>
        {t('error.unexpectedTitle')}
      </Text>
      <Text className='text-center text-sm text-primary/60'>{error.message}</Text>
      <Button onPress={resetErrorBoundary}>
        <Text>{t('common.tryAgain')}</Text>
      </Button>
    </SafeAreaView>
  );
}
```

## Inline Loading in Lists

```tsx
<FlatList
  data={trips}
  renderItem={({ item }) => <TripCard trip={item} />}
  ListFooterComponent={isFetchingNextPage ? <ActivityIndicator className='py-4' /> : null}
  ListEmptyComponent={isLoading ? <LoadingState /> : <EmptyState message={t('trips.empty')} />}
  onEndReached={fetchNextPage}
  onEndReachedThreshold={0.5}
/>
```

## Mutation Error Handling (Toast Pattern)

```tsx
const { mutate } = useMutation({
  mutationFn: api.trips.end,
  onError: (error) => {
    // Show non-blocking toast
    Toast.show({ type: 'error', text1: t('trips.endFailed'), text2: error.message });
  },
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: queryKeys.trips.all });
    router.back();
  },
});
```
