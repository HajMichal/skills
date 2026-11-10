# Guards, Pipes, Interceptors

## Request Lifecycle Order

```
Request → Middleware → Guards → Interceptors → Pipes → Controller → Service
                                                            ↓
Response ← Exception Filters ← Interceptors ← Controller ←
```

## Guards

```typescript
// Role-based guard
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<Role[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true;

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => user.roles?.includes(role));
  }
}

// Usage
@Roles(Role.ADMIN)
@UseGuards(RolesGuard)
@Get('admin/users')
adminOnly() { ... }
```

## Custom Pipes

```typescript
@Injectable()
export class ParseDatePipe implements PipeTransform {
  transform(value: string): Date {
    const date = new Date(value);
    if (isNaN(date.getTime())) throw new BadRequestException('Invalid date format');
    return date;
  }
}
```

## Interceptors

```typescript
// Response transformation
@Injectable()
export class TransformInterceptor<T> implements NestInterceptor<T, ApiResponse<T>> {
  intercept(context: ExecutionContext, next: CallHandler): Observable<ApiResponse<T>> {
    return next.handle().pipe(map(data => ({ data, timestamp: new Date().toISOString() })));
  }
}

// Logging interceptor
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  private readonly logger = new Logger(LoggingInterceptor.name);

  intercept(context: ExecutionContext, next: CallHandler): Observable<unknown> {
    const req = context.switchToHttp().getRequest();
    const now = Date.now();
    return next.handle().pipe(
      tap(() => this.logger.log(`${req.method} ${req.url} — ${Date.now() - now}ms`)),
    );
  }
}
```

## Global Registration (main.ts / AppModule)

```typescript
app.useGlobalPipes(new ZodValidationPipe());
app.useGlobalInterceptors(new ZodSerializerInterceptor());
app.useGlobalGuards(new JwtAuthGuard(reflector));
app.useGlobalFilters(new AllExceptionsFilter());
```
