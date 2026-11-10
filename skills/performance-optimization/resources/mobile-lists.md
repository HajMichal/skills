# Mobile List Optimization

The app uses React Native's `FlatList` (no FlashList dependency). Lists are short (notifications, benefits, countries) and several are horizontal paging carousels.

## Fixed-size items — `getItemLayout`

Used by the full-width carousels (`PromoCarousel`, `OnboardingCarousel`, `ActiveClaimsCarousel`, `DemoShowcaseDrawer`). Skips measurement and makes `scrollToIndex` reliable:

```tsx
<FlatList
  data={items}
  horizontal
  pagingEnabled
  keyExtractor={(item) => item.id}
  renderItem={renderItem}
  getItemLayout={(_, index) => ({ length: width, offset: width * index, index })}
/>
```

## Variable-size items

Omit `getItemLayout` (a wrong layout is worse than none) — `keyExtractor` is still required.

## Guidelines

- Stable `keyExtractor` from the item id, never the index
- Define `renderItem` with `useCallback` so item rows don't re-render on parent state changes
- Tune `initialNumToRender` / `windowSize` only if a list is measurably slow — the defaults suit current list sizes
- Add `@shopify/flash-list` only if a genuinely long list appears and FlatList is measured to be the bottleneck
