# Mobile Component Tests

## Basic Component Test

```tsx
// TripCard.test.tsx
import { render, screen, fireEvent } from '@testing-library/react-native';
import { TripCard } from '@/components/trips/TripCard';

const mockTrip = {
  id: 'trip-1',
  region: 'EUROPE',
  startedAt: new Date('2024-01-01'),
};

describe('TripCard', () => {
  it('renders trip region', () => {
    render(<TripCard trip={mockTrip} onPress={jest.fn()} />);
    expect(screen.getByText('EUROPE')).toBeTruthy();
  });

  it('calls onPress with trip id', () => {
    const onPress = jest.fn();
    render(<TripCard trip={mockTrip} onPress={onPress} />);
    fireEvent.press(screen.getByTestId('trip-card'));
    expect(onPress).toHaveBeenCalledWith('trip-1');
  });
});
```

## Mocking TanStack Query

```tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';

function renderWithQuery(ui: React.ReactElement) {
  const queryClient = new QueryClient({
    defaultOptions: { queries: { retry: false } },
  });
  return render(
    <QueryClientProvider client={queryClient}>{ui}</QueryClientProvider>,
  );
}

// Mock the API call
jest.mock('@/lib/api', () => ({
  trips: { getAll: jest.fn().mockResolvedValue([mockTrip]) },
}));
```

## Mocking Zustand

```tsx
jest.mock('@/stores/auth.store', () => ({
  useAuthStore: () => ({
    token: 'test-token',
    user: { sub: 'user-1' },
    logout: jest.fn(),
  }),
}));
```

## Mocking Expo Router

```tsx
jest.mock('expo-router', () => ({
  router: { push: jest.fn(), back: jest.fn(), replace: jest.fn() },
  useLocalSearchParams: () => ({ id: 'trip-1' }),
  Link: ({ children }: { children: React.ReactNode }) => children,
}));
```

## i18n in Tests

```tsx
import '../../../src/lib/i18n'; // initialize i18n before tests

// Or mock the hook
jest.mock('react-i18next', () => ({
  useTranslation: () => ({ t: (key: string) => key }),
}));
```
