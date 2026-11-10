---
name: performance-optimization
description: Performance optimization checklist for API, database, and mobile layers
---

# Performance Optimization — Quick Reference

## Checklist by Layer

### API

- [ ] Pagination on list endpoints (`skip` + `take`, with `count` in the same `$transaction`)
- [ ] `select` only needed fields — avoid over-fetching
- [ ] No N+1 queries — use `include` or batch with `findMany` + `where: { id: { in: ids } }`
- [ ] No caching layer is in use (`@nestjs/cache-manager` is installed but unwired) — don't add one without a measured need
- [ ] Heavy / periodic work runs in `@nestjs/schedule` crons, side effects in `@nestjs/event-emitter` listeners — see [`background-work.md`](resources/background-work.md)

### Database

- [ ] `@@index` on all `WHERE` / `ORDER BY` fields
- [ ] Use transactions for multi-step writes
- [ ] Use `findUnique` / `findUniqueOrThrow` over `findFirst` when possible

### Mobile

- [ ] `FlatList` for lists — never `ScrollView` + `.map()` for unbounded data
- [ ] `keyExtractor` always; `getItemLayout` when item size is fixed (e.g. full-width carousels)
- [ ] `react-native-reanimated` worklets for animations (UI thread)
- [ ] `expo-image` with `placeholder={{ blurhash: DEFAULT_IMAGE_BLURHASH }}`
- [ ] `useMemo` / `useCallback` / `memo` only for provably expensive values — avoid premature memoization

### Web (React)

- [ ] `React.memo` for expensive pure components
- [ ] `useMemo` / `useCallback` for stable references passed to memoized children
- [ ] Proper `key` props in lists (stable, not index)

## Resources

| Topic                    | File                                                       | When to Use                            |
| ------------------------ | ---------------------------------------------------------- | -------------------------------------- |
| Background work          | [`background-work.md`](resources/background-work.md)       | Crons, advisory locks, domain events   |
| Query optimization       | [`query-optimization.md`](resources/query-optimization.md) | Prisma select/include, N+1, pagination |
| React memoization        | [`react-memoization.md`](resources/react-memoization.md)   | When and when NOT to memoize           |
| Mobile list optimization | [`mobile-lists.md`](resources/mobile-lists.md)             | FlatList config, fixed-size carousels  |
