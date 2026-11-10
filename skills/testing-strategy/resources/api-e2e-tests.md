# API E2E Tests

Requires a running DB. Run with the API's e2e script (e.g. `pnpm --filter <api> test:e2e`)

## Test Setup Template

```typescript
// test/trips.e2e-spec.ts
describe('TripsController (e2e)', () => {
  let app: INestApplication;
  let prisma: PrismaService;
  let authToken: string;

  beforeAll(async () => {
    const moduleRef = await Test.createTestingModule({
      imports: [AppModule],
    }).compile();

    app = moduleRef.createNestApplication();
    app.useGlobalPipes(new ZodValidationPipe());
    await app.init();

    prisma = app.get(PrismaService);
    authToken = await getTestJwt(app); // helper that creates a test user + JWT
  });

  afterAll(async () => {
    await prisma.trip.deleteMany({ where: { userId: TEST_USER_ID } });
    await app.close();
  });

  it('POST /trips — creates a trip', async () => {
    const response = await request(app.getHttpServer())
      .post('/trips')
      .set('Authorization', `Bearer ${authToken}`)
      .send({ region: 'EUROPE' })
      .expect(201);

    expect(response.body).toMatchObject({
      id: expect.any(String),
      region: 'EUROPE',
    });
  });

  it('GET /trips/:id — returns 404 for unknown id', async () => {
    await request(app.getHttpServer())
      .get('/trips/non-existent-id')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(404);
  });

  it('POST /trips — returns 401 without auth', async () => {
    await request(app.getHttpServer())
      .post('/trips')
      .send({ region: 'EUROPE' })
      .expect(401);
  });
});
```

## DB Cleanup Pattern

Always clean up test data — never assume a fresh DB:

```typescript
// Isolated cleanup per test
afterEach(async () => {
  await prisma.regionVisit.deleteMany({ where: { trip: { userId: TEST_USER_ID } } });
  await prisma.trip.deleteMany({ where: { userId: TEST_USER_ID } });
});
```

## Test JWT Helper

```typescript
// test/helpers/auth.ts
export async function getTestJwt(app: INestApplication): Promise<string> {
  const user = await prisma.user.upsert({
    where: { idNumber: TEST_NIN },
    update: {},
    create: { idNumber: encrypt(TEST_NIN), email: 'test@example.com' },
  });
  return jwtService.sign({ sub: user.id });
}
```
