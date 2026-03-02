# Rules

Conventions that apply to every task without being asked.

---

## File Placement

The directory structure scales with the number of API providers.

### Single provider

```
src/
  api/
    api-clients/          ← one file per resource
    types/                ← one .d.ts per resource
    factories/            ← one factory per resource
    fixtures/
    base-api-client.ts
    api-statuses.ts
  tests/
    {resource}.api.spec.ts
utils/
  env.ts
```

### Multiple providers

```
src/
  api/
    {provider}/
      api-clients/        ← one file per resource
      types/              ← one .d.ts per resource
      factories/          ← one factory per resource
    fixtures/             ← shared across all providers
    base-api-client.ts
    api-statuses.ts
  tests/
    {provider}/
      {resource}.api.spec.ts
utils/
  env.ts
```

| Concern | Single provider | Multiple providers |
|---|---|---|
| API client wrappers | `src/api/api-clients/` | `src/api/{provider}/api-clients/` |
| TypeScript types/interfaces | `src/api/types/` | `src/api/{provider}/types/` |
| Faker-based factories | `src/api/factories/` | `src/api/{provider}/factories/` |
| Test specs | `src/tests/` | `src/tests/{provider}/` |
| Custom Playwright fixtures | `src/api/fixtures/` | `src/api/fixtures/` (always shared) |
| Reusable utilities / env | `utils/` | `utils/` |
| HTTP status constants | `src/api/api-statuses.ts` | `src/api/api-statuses.ts` |
| Base API client | `src/api/base-api-client.ts` | `src/api/base-api-client.ts` |

---

## Naming

- Clients: `{resource}-api-client.ts` — class named `{Resource}ApiClient`
- Types: `{resource}.d.ts` — interfaces in PascalCase, **no `I` prefix** (e.g. `User`, `UserListResponse`)
- Factories: `{resource}.factory.ts` — exports `{resource}ListParams`, `{resource}SearchParams`, etc.
- Specs: `{resource}.api.spec.ts` (e.g. `books.api.spec.ts`)
- Fixture file: `api.fixture.ts`
- Fixture names: `{resource}Api` (e.g. `stockApi`, `cryptoApi`, `openHolidaysApi`) — consistent `{resource}Api` pattern for every service
- Utils: `{name}.ts` (e.g. `env.ts`)

**Imports** — always use path aliases:
- `@/*` for anything inside `src/` (e.g. `@/api/api-clients/books-api-client`)
- `@utils/*` for anything inside `utils/` (e.g. `@utils/env`)
- Never use relative `../../` paths across layers.

---

## Client Layer (`src/api/api-clients/`)

- All clients extend `BaseApiClient` from `src/api/base-api-client.ts`
- `BaseApiClient` owns: base URL, auth header injection, typed request methods
- Client methods return the raw `APIResponse` — never throw on 4xx/5xx; let tests assert
- Do not put assertions inside client methods
- Every client must be wired into `src/api/fixtures/api.fixture.ts` — clients that are not in the fixture are dead code

```ts
import { BaseApiClient } from '@/api/base-api-client';

export class BooksApiClient extends BaseApiClient {
  getBooks(params?: Record<string, string>) {
    return this.get('/api/books', params);
  }

  getBook(id: number) {
    return this.get(`/api/book/${id}`);
  }
}
```

---

## Type Files (`src/api/types/`)

- Pure TypeScript interfaces and type aliases only — no functions, no constants, no imports
- Every `response.json()` call must be typed: `const body: BookListResponse = await response.json()`
- Nullable fields: `string | null`; optional/absent fields: `field?: string`
- Always use `import type` when importing from `.d.ts` files

```ts
import type { BookListResponse, BookDetailResponse } from '@/api/types/book.d';
```

---

## Test Structure

- Use `test.describe` blocks grouped by resource then endpoint
- Every spec imports `test` and `expect` from `@/api/fixtures/api.fixture` — never from `@playwright/test`
- Follow **Arrange → Act → Assert** with a blank line between each phase
- All test names must start with `should`
- Every resource must have at least one unhappy-path (4xx) test
- Tests use the typed client fixture (e.g. `{ booksApi }`) — never call `api.get(...)` with raw URL strings

```ts
import { test, expect } from '@/api/fixtures/api.fixture';
import { HTTP_STATUS } from '@/api/api-statuses';
import type { BookListResponse } from '@/api/types/book.d';

test.describe('Books API', () => {
  test.describe('GET /api/books', () => {
    test('should return a list of books', async ({ booksApi }) => {
      const response = await booksApi.getBooks();           // Act

      expect(response.status()).toBe(HTTP_STATUS.OK);       // Assert
      const body: BookListResponse = await response.json();
      expect(body.data).toBeDefined();
    });

    test('should return 404 when book does not exist', async ({ booksApi }) => {
      const response = await booksApi.getBook(999999);

      expect(response.status()).toBe(HTTP_STATUS.NOT_FOUND);
    });
  });
});
```

---

## Assertions

- Assert `response.status()` before body — fail fast on wrong status code
- Use `HTTP_STATUS.*` constants from `src/api/api-statuses.ts`, never raw numbers
- Never use untyped `response.json()` — always annotate with an interface
- Negative cases must assert both status code and response shape

---

## Fixtures (`src/api/fixtures/`)

- `api.fixture.ts` — exposes one typed client per API service
- Each fixture entry creates its own `APIRequestContext` with the correct `baseURL` and `extraHTTPHeaders`, then wraps it in the typed client class
- Fixture names follow the `{resource}Api` convention — e.g. `stockApi`, `cryptoApi`, `openHolidaysApi`
- Tests destructure the typed client fixture — never instantiate `APIRequestContext` directly in tests

```ts
import { test as base } from '@playwright/test';
import { env } from '@utils/env';
import { BooksApiClient } from '@/api/api-clients/books-api-client';

type ApiFixtures = {
  booksApi: BooksApiClient;
};

export const test = base.extend<ApiFixtures>({
  booksApi: async ({ playwright }, use) => {
    const context = await playwright.request.newContext({
      baseURL: env.BASE_URL,
      extraHTTPHeaders: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${env.API_KEY}`,
      },
    });
    await use(new BooksApiClient(context));
    await context.dispose();
  },
});

export { expect } from '@playwright/test';
```

**Auth headers belong in the fixture only.** Never set auth headers in `playwright.config.ts` `extraHTTPHeaders` — the config-level headers apply globally to all contexts including services that don't require that auth scheme.

---

## Environment & Config

- All secrets and base URLs live in `.env` — never hardcoded
- All env vars accessed through `utils/env.ts` via `@utils/env` — never `process.env` directly
- `utils/env.ts` uses `requireEnv()` for required variables (fails fast if missing)
- `.env.example` must be updated whenever a new env var is introduced
- `playwright.config.ts` must NOT include auth headers in `extraHTTPHeaders` — auth is injected per-context in `api.fixture.ts`

---

## Factories (`src/api/factories/`)

- Use `@faker-js/faker` for random data
- Factories must not make API calls — they only produce plain data objects
- Time-relative params (date ranges, unix timestamps) must be computed dynamically from `Date.now()` — never hardcode specific past dates
- Factories are most valuable for complex or dynamic params; for simple single-value calls prefer passing the value directly to the typed client method

---

## Pre-Commit

Run before every commit and before opening a PR — never commit with failing tests, type errors, lint violations, or formatting issues:

```bash
npm test && npx tsc --noEmit && npm run lint && npm run format:check
```

---

## Tier-Gated Endpoints

When testing an endpoint that behaves differently based on API plan/tier:

- Add a `// Note:` comment above the `test.describe` block explaining the tier constraint
- Name the test to reflect what the current tier receives, not the endpoint's intended behavior
- If the project may upgrade to a paid plan, guard with an env var: `test.skip(env.API_PLAN === 'paid', '...')`

```ts
test.describe('GET stock/candle', () => {
  // Note: /stock/candle requires a paid Finnhub plan; free tier returns 403.

  test('should return 403 when accessing candles on free plan', async ({ stockApi }) => {
    const response = await stockApi.getCandles(stockCandleParams());
    expect(response.status()).toBe(HTTP_STATUS.FORBIDDEN);
  });
});
```


