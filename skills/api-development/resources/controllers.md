# NestJS Controllers

## Standard Template

```typescript
@ApiTags('policies')
@ApiBearerAuth()
@Controller('policies')
export class PoliciesController {
  constructor(private readonly policiesService: PoliciesService) {}

  @Post()
  @HttpCode(HttpStatus.CREATED)
  @ApiOperation({ summary: 'Create a new policy' })
  @ApiResponse({ status: 201, type: PolicyResponseDto })
  create(@Body() dto: CreatePolicyDto): Promise<PolicyResponseDto> {
    return this.policiesService.create(dto);
  }

  @Get(':id')
  @ApiResponse({ status: 200, type: PolicyResponseDto })
  @ApiResponse({ status: 404, description: 'Policy not found' })
  findOne(@Param('id', ParseUUIDPipe) id: string): Promise<PolicyResponseDto> {
    return this.policiesService.findOne(id);
  }

  @Patch(':id')
  update(
    @Param('id', ParseUUIDPipe) id: string,
    @Body() dto: UpdatePolicyDto,
  ): Promise<PolicyResponseDto> {
    return this.policiesService.update(id, dto);
  }

  @Delete(':id')
  @HttpCode(HttpStatus.NO_CONTENT)
  remove(@Param('id', ParseUUIDPipe) id: string): Promise<void> {
    return this.policiesService.remove(id);
  }
}
```

## Key Rules

- Controllers are **thin** — one line per method, delegate to service
- Always use `ParseUUIDPipe` for UUID path params
- Use `@HttpCode()` for non-200 success responses
- Use `@Public()` to opt out of global `JwtAuthGuard`
- Use `@SkipThrottle()` to opt out of rate limiting

## Getting the Current User

```typescript
// Use a custom decorator
@Get('me')
getProfile(@CurrentUser() user: JwtPayload): Promise<UserResponseDto> {
  return this.usersService.findById(user.sub);
}
```

## Query Parameters

```typescript
@Get()
findAll(@Query() query: ListPoliciesDto): Promise<PolicyResponseDto[]> {
  return this.policiesService.findAll(query);
}

// In the DTO:
const listPoliciesSchema = z.object({
  page: z.coerce.number().int().min(1).default(1),
  limit: z.coerce.number().int().min(1).max(100).default(20),
  region: z.enum(TravelRegion).optional(),
});
```
