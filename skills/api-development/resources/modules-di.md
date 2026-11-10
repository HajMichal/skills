# NestJS Modules & Dependency Injection

## Feature Module Template

```typescript
@Module({
  imports: [PrismaModule],          // add shared modules needed
  controllers: [UsersController],
  providers: [UsersService, UsersRepository],
  exports: [UsersService],          // export only if other modules need it
})
export class UsersModule {}
```

## Shared Modules Pattern

Register global modules once in `AppModule`:

```typescript
@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    PrismaModule,                    // inject PrismaService anywhere
    ThrottlerModule.forRoot([{ ttl: 60000, limit: 10 }]),
  ],
})
export class AppModule {}
```

## Dependency Injection

```typescript
@Injectable()
export class UsersService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly configService: ConfigService,
  ) {}
}
```

Rules:
- Constructor injection only — never property injection
- Never `new SomeService()` — let the DI container handle it
- Use `forwardRef()` only as a last resort for circular deps

## Dynamic Modules (forRootAsync)

```typescript
// In a module that needs async config
UpstreamApiModule.forRootAsync({
  useFactory: (config: ConfigService) => ({
    baseUrl: config.get('UPSTREAM_API_BASE_URL'),
    apiKey: config.get('UPSTREAM_API_KEY'),
  }),
  inject: [ConfigService],
})
```

## Custom Providers

```typescript
// Interface token
const PAYMENT_GATEWAY = Symbol('PAYMENT_GATEWAY');

// In module providers:
{
  provide: PAYMENT_GATEWAY,
  useFactory: (config: ConfigService) => new PaymentGateway(config.get('PAYMENT_PROVIDER_SECRET')),
  inject: [ConfigService],
}

// Inject by token:
constructor(@Inject(PAYMENT_GATEWAY) private gateway: IPaymentGateway) {}
```
