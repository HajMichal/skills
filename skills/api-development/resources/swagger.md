# Swagger Documentation with nestjs-zod

## Setup (main.ts)

```typescript
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
import { patchNestJsSwagger } from 'nestjs-zod';

patchNestJsSwagger(); // must be called before createDocument

const config = new DocumentBuilder()
  .setTitle('API')
  .setDescription('Public API')
  .setVersion('1.0')
  .addBearerAuth()
  .build();

const document = SwaggerModule.createDocument(app, config);
SwaggerModule.setup('api/docs', app, document);
```

## Controller Decorators

```typescript
@ApiTags('trips')                        // group endpoints
@ApiBearerAuth()                         // show lock icon on all routes
@Controller('trips')
export class TripsController {

  @Post()
  @ApiOperation({ summary: 'Start a new trip' })
  @ApiResponse({ status: 201, type: TripResponseDto })
  @ApiResponse({ status: 400, description: 'Invalid trip data' })
  @ApiResponse({ status: 401, description: 'Unauthorized' })
  create(@Body() dto: CreateTripDto): Promise<TripResponseDto> {
    return this.tripsService.create(dto);
  }

  @Get(':id')
  @ApiParam({ name: 'id', description: 'Trip UUID' })
  findOne(@Param('id', ParseUUIDPipe) id: string): Promise<TripResponseDto> {
    return this.tripsService.findOne(id);
  }
}
```

## Describing Schemas

```typescript
// Add .describe() to fields for Swagger docs
const createTripSchema = z.object({
  userId: z.string().uuid().describe('User ID from the auth JWT'),
  region: z.enum(TravelRegion).describe('Region visited'),
  startsAt: z.coerce.date().describe('Trip start timestamp (ISO 8601)'),
});
```

## Public Routes

```typescript
@Public()                    // skip JwtAuthGuard
@ApiBearerAuth(undefined)    // remove lock icon
@Get('health')
healthCheck() { ... }
```
