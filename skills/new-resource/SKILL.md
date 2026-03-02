---
name: new-resource
description: Scaffold all layers for a new API resource. Use when adding support for a new API endpoint group (e.g. Orders, Payments).
---

## Steps

First, determine if the project has **one provider** or **multiple providers** — this controls the folder structure.

**Single provider** (e.g. only one API):
1. Create `src/api/types/{resource}.d.ts`
2. Create `src/api/api-clients/{resource}-api-client.ts`
3. Create `src/api/factories/{resource}.factory.ts`
4. Register the new client in `src/api/fixtures/api.fixture.ts`
5. Scaffold spec at `src/tests/{resource}.api.spec.ts`

**Multiple providers** (e.g. Finnhub + OpenHolidays):
1. Create `src/api/{provider}/types/{resource}.d.ts`
2. Create `src/api/{provider}/api-clients/{resource}-api-client.ts`
3. Create `src/api/{provider}/factories/{resource}.factory.ts`
4. Register the new client in `src/api/fixtures/api.fixture.ts`
5. Scaffold spec at `src/tests/{provider}/{resource}.api.spec.ts`

`src/api/fixtures/`, `src/api/base-api-client.ts`, and `src/api/api-statuses.ts` are always at the root of `src/api/` regardless of provider count.

## File Templates

### `src/api/types/{resource}.d.ts`

```ts
export interface {Resource} {
  id: number | string;
  // Add fields matching the actual API response
}

export interface {Resource}ListResponse {
  data: {Resource}[];
  // pagination, info, etc.
}

export interface {Resource}DetailResponse {
  data: {Resource};
}
```

### `src/api/api-clients/{resource}-api-client.ts`

```ts
import { BaseApiClient } from '@/api/base-api-client';

export class {Resource}ApiClient extends BaseApiClient {
  get{Resource}s(params?: Record<string, string>) {
    return this.get('/api/{resources}', params);
  }

  get{Resource}(id: number) {
    return this.get(`/api/{resource}/${id}`);
  }
}
```

### `src/api/factories/{resource}.factory.ts`

```ts
import { faker } from '@faker-js/faker';

export function {resource}ListParams(overrides: Record<string, string> = {}): Record<string, string> {
  return {
    limit: '10',
    page: '1',
    ...overrides,
  };
}
```

**For time-relative params**, always compute dynamically — never hardcode specific dates:

```ts
export function {resource}DateRangeParams(overrides: Record<string, string> = {}): Record<string, string> {
  const year = new Date().getFullYear().toString();
  return {
    validFrom: `${year}-01-01`,
    validTo: `${year}-12-31`,
    ...overrides,
  };
}
```

### `src/api/fixtures/api.fixture.ts` — add the new client

```ts
import { {Resource}ApiClient } from '@/api/api-clients/{resource}-api-client';

type ApiFixtures = {
  // ... existing fixtures
  {resource}Api: {Resource}ApiClient;
};

export const test = base.extend<ApiFixtures>({
  // ... existing fixtures

  {resource}Api: async ({ playwright }, use) => {
    const context = await playwright.request.newContext({
      baseURL: env.{RESOURCE}_BASE_URL,
      extraHTTPHeaders: {
        'Content-Type': 'application/json',
        // add auth headers specific to this service only
      },
    });
    await use(new {Resource}ApiClient(context));
    await context.dispose();
  },
});
```

**Do not add auth headers to `playwright.config.ts`** — all auth belongs in the per-context fixture setup above.

### `src/tests/{provider}/{resource}.api.spec.ts`

```ts
import { test, expect } from '@/api/fixtures/api.fixture';
import { HTTP_STATUS } from '@/api/api-statuses';
import { {resource}ListParams } from '@/api/factories/{resource}.factory';
import type { {Resource}ListResponse, {Resource}DetailResponse } from '@/api/types/{resource}.d';

test.describe('{Resource}s API', () => {
  test.describe('GET /api/{resources}', () => {
    test('should return a list of {resources}', async ({ {resource}Api }) => {
      const response = await {resource}Api.get{Resource}s({resource}ListParams());

      expect(response.status()).toBe(HTTP_STATUS.OK);
      const body: {Resource}ListResponse = await response.json();
      expect(body.data).toBeDefined();
    });
  });

  test.describe('GET /api/{resource}/:id', () => {
    test('should return a single {resource} by id', async ({ {resource}Api }) => {
      const response = await {resource}Api.get{Resource}(1);

      expect(response.status()).toBe(HTTP_STATUS.OK);
      const body: {Resource}DetailResponse = await response.json();
      expect(body.data).toBeDefined();
    });

    test('should return 404 when {resource} does not exist', async ({ {resource}Api }) => {
      const response = await {resource}Api.get{Resource}(999999);

      expect(response.status()).toBe(HTTP_STATUS.NOT_FOUND);
    });
  });
});
```

**Tier-gated endpoint** — when the endpoint behaves differently based on API plan:

```ts
test.describe('GET /api/{resource}/premium', () => {
  // Note: requires a paid plan; free tier returns 403.

  test('should return 403 on free plan', async ({ {resource}Api }) => {
    const response = await {resource}Api.getPremium();

    expect(response.status()).toBe(HTTP_STATUS.FORBIDDEN);
  });
});
```

## Output Checklist

- [ ] Type interfaces in `src/api/types/{resource}.d.ts`
- [ ] Client class extending `BaseApiClient` in `src/api/api-clients/{resource}-api-client.ts`
- [ ] Factory in `src/api/factories/{resource}.factory.ts`
- [ ] Client registered as `{resource}Api` in `src/api/fixtures/api.fixture.ts`
- [ ] Spec uses `{ {resource}Api }` fixture — no raw `api.get(...)` calls with URL strings
- [ ] Spec has happy and unhappy paths in `src/tests/{provider}/{resource}.api.spec.ts`
- [ ] `HTTP_STATUS.*` used — no raw numbers
- [ ] All `response.json()` calls typed with an interface
- [ ] Auth headers set only in fixture, not in `playwright.config.ts`
