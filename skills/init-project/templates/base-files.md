# Base File Templates

Verbatim templates for all base project files. Create each file exactly as shown.

---

### `tsconfig.json`

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "rootDir": ".",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@utils/*": ["./utils/*"]
    }
  },
  "include": ["src/**/*", "utils/**/*"],
  "exclude": ["node_modules"]
}
```

---

### `.env.example`

```dotenv
BASE_URL=https://api.example.com
# API_KEY=your_api_key_here
```

Copy to `.env` and fill in real values.

---

### `utils/env.ts`

```typescript
import * as dotenv from 'dotenv';

dotenv.config();

const requireEnv = (name: string): string => {
  const value = process.env[name];
  if (!value) throw new Error(`Missing required environment variable: ${name}`);
  return value;
};

export const env = {
  BASE_URL: requireEnv('BASE_URL'),
  // API_KEY: requireEnv('API_KEY'),
} as const;
```

---

### `playwright.config.ts`

```typescript
import { defineConfig } from '@playwright/test';
import { env } from './utils/env';

export default defineConfig({
  testDir: './src/tests',
  timeout: 30_000,
  use: {
    baseURL: env.BASE_URL,
  },
  reporter: [
    ['list'],
    ['allure-playwright', { outputFolder: 'allure-results', detail: true, suiteTitle: false }],
  ],
});
```

**Do not add auth headers here.** Auth headers belong in `src/api/fixtures/api.fixture.ts` on a per-context basis.

---

### `src/api/api-statuses.ts`

```typescript
export const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  NO_CONTENT: 204,
  BAD_REQUEST: 400,
  UNAUTHORIZED: 401,
  FORBIDDEN: 403,
  NOT_FOUND: 404,
  UNPROCESSABLE_ENTITY: 422,
  INTERNAL_SERVER_ERROR: 500,
} as const;
```

---

### `src/api/base-api-client.ts`

```typescript
import type { APIRequestContext, APIResponse } from '@playwright/test';

export class BaseApiClient {
  constructor(protected readonly api: APIRequestContext) {}

  protected get(path: string, params?: Record<string, string>): Promise<APIResponse> {
    return this.api.get(path, { params });
  }

  protected post(path: string, data?: unknown): Promise<APIResponse> {
    return this.api.post(path, { data });
  }

  protected put(path: string, data?: unknown): Promise<APIResponse> {
    return this.api.put(path, { data });
  }

  protected patch(path: string, data?: unknown): Promise<APIResponse> {
    return this.api.patch(path, { data });
  }

  protected delete(path: string): Promise<APIResponse> {
    return this.api.delete(path);
  }
}
```

---

### `src/api/fixtures/api.fixture.ts`

```typescript
import { test as base } from '@playwright/test';
import { env } from '@utils/env';
import { {Resource}ApiClient } from '@/api/api-clients/{resource}-api-client';

type ApiFixtures = {
  {resource}Api: {Resource}ApiClient;
};

export const test = base.extend<ApiFixtures>({
  {resource}Api: async ({ playwright }, use) => {
    const context = await playwright.request.newContext({
      baseURL: env.BASE_URL,
      extraHTTPHeaders: {
        'Content-Type': 'application/json',
        // 'Authorization': `Bearer ${env.API_KEY}`,
      },
    });
    await use(new {Resource}ApiClient(context));
    await context.dispose();
  },
});

export { expect } from '@playwright/test';
```

Replace `{Resource}` and `{resource}` with the first resource once `/add-test` is run.

---

### `.gitignore`

```gitignore
node_modules/
allure-results/
allure-report/
playwright-report/
test-results/
.env
```
