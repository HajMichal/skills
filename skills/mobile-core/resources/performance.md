# Mobile Performance

## FlatList Optimization

```tsx
<FlatList
  data={trips}
  keyExtractor={(item) => item.id}               // stable, unique key
  renderItem={({ item }) => <TripCard trip={item} />}
  getItemLayout={(_, index) => ({                // skip measurement (fixed height)
    length: TRIP_CARD_HEIGHT,
    offset: TRIP_CARD_HEIGHT * index,
    index,
  })}
  removeClippedSubviews                          // unmount off-screen items
  maxToRenderPerBatch={10}
  windowSize={5}
  initialNumToRender={10}
/>
```

**Common mistakes:**
```tsx
// ❌ Inline function recreated every render
renderItem={({ item }) => <TripCard trip={item} onPress={() => router.push(item.id)} />}

// ✅ Stable callback
const handlePress = useCallback((id: string) => router.push(`/trips/${id}`), []);
renderItem={({ item }) => <TripCard trip={item} onPress={handlePress} />}
```

## Reanimated Animations

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withTiming } from 'react-native-reanimated';

// Always use Reanimated — never the core Animated API
const opacity = useSharedValue(0);

const animatedStyle = useAnimatedStyle(() => ({
  opacity: opacity.value,
}));

// Trigger animation
opacity.value = withTiming(1, { duration: 300 });

return <Animated.View style={animatedStyle} />;
```

## Image Optimization

```tsx
import { Image } from 'expo-image';

// blurhash shows while image loads
<Image
  source={{ uri: imageUrl }}
  placeholder={{ blurhash: 'L6PZfSi_.AyE_3t7t7R**0o#DgR4' }}
  contentFit="cover"
  transition={200}
  cachePolicy="memory-disk"
/>
```

## Memoization (use sparingly)

```tsx
// Only memoize if the component is provably expensive AND receives stable props
const TripCard = React.memo(({ trip, onPress }: TripCardProps) => {
  return <View>...</View>;
});

// useMemo for expensive computations
const sortedTrips = useMemo(() =>
  [...trips].sort((a, b) => new Date(b.startedAt).getTime() - new Date(a.startedAt).getTime()),
  [trips],
);
```
