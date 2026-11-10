# API Unit Tests

## Service Test Template

```typescript
// trips.service.spec.ts
describe('TripsService', () => {
  let service: TripsService;
  let repository: jest.Mocked<TripsRepository>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      providers: [
        TripsService,
        {
          provide: TripsRepository,
          useValue: {
            create: jest.fn(),
            findById: jest.fn(),
            findAll: jest.fn(),
            remove: jest.fn(),
          },
        },
      ],
    }).compile();

    service = module.get(TripsService);
    repository = module.get(TripsRepository);
  });

  describe('create', () => {
    it('creates a trip and returns response DTO', async () => {
      const dto: CreateTripDto = { userId: 'user-1', region: TravelRegion.EUROPE };
      const created = { id: 'trip-1', ...dto, createdAt: new Date(), deletedAt: null };

      repository.create.mockResolvedValue(created);

      const result = await service.create(dto);

      expect(repository.create).toHaveBeenCalledWith(dto);
      expect(result.id).toBe('trip-1');
    });

    it('propagates repository errors', async () => {
      repository.create.mockRejectedValue(new ConflictException('Already exists'));
      await expect(service.create({ userId: 'u1', region: TravelRegion.USA }))
        .rejects.toThrow(ConflictException);
    });
  });
});
```

## Controller Test Template

```typescript
// trips.controller.spec.ts — keep thin, test routing + delegation only
describe('TripsController', () => {
  let controller: TripsController;
  let service: jest.Mocked<TripsService>;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [TripsController],
      providers: [{
        provide: TripsService,
        useValue: { create: jest.fn(), findOne: jest.fn() },
      }],
    }).compile();

    controller = module.get(TripsController);
    service = module.get(TripsService);
  });

  it('calls service.create with the dto', async () => {
    const dto = { userId: 'u1', region: TravelRegion.EUROPE };
    service.create.mockResolvedValue({ id: 'trip-1', ...dto } as any);
    await controller.create(dto as CreateTripDto);
    expect(service.create).toHaveBeenCalledWith(dto);
  });
});
```

## Mocking Tips

- Mock at the boundary closest to the test — services mock repos, controllers mock services
- Use `jest.fn()` for all injected dependencies
- Reset mocks in `afterEach(() => jest.clearAllMocks())`
