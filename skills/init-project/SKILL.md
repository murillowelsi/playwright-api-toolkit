---
name: init-project
description: Scaffold the base project structure for a Playwright API testing suite. Use once when starting a new project.
---

# Init Project

Scaffold the full base structure for a Playwright + TypeScript API testing project. Run this once before creating any spec files.

## 1. Gather Requirements

Ask the user:
- What is the `BASE_URL` for the API?
- Does the API require authentication? If so, what kind? (Bearer token, API key header, Basic auth, none)

## 2. Install Dependencies

```bash
npm init -y
npm install --save-dev @playwright/test typescript @types/node allure-playwright allure-commandline
npm install dotenv
```

> **Note:** `allure-commandline` requires Java to generate HTML reports.
> Install Java before running `npm run report`:
> - macOS: `brew install --cask temurin`
> - Linux: `sudo apt install default-jdk`
> - Windows: download from https://adoptium.net

## 3. Create Directory Structure

**Single provider:**

```bash
mkdir -p src/api/api-clients src/api/types src/api/factories src/api/fixtures src/tests utils
```

**Multiple providers** — create one subdirectory per provider inside `src/api/` and `src/tests/`:

```bash
mkdir -p src/api/{provider}/api-clients src/api/{provider}/types src/api/{provider}/factories
mkdir -p src/api/fixtures src/tests/{provider} utils
```

`src/api/fixtures/`, `src/api/base-api-client.ts`, and `src/api/api-statuses.ts` always stay at the `src/api/` root.

## 4. Create Base Files

Create each file using the exact templates in `templates/base-files.md`:

- `tsconfig.json`
- `.env.example`
- `utils/env.ts`
- `playwright.config.ts`
- `src/api/api-statuses.ts`
- `src/api/base-api-client.ts`
- `src/api/fixtures/api.fixture.ts`
- `.gitignore`

## 5. Configure ESLint and Prettier

Follow the setup in `templates/tooling.md` to install and configure ESLint and Prettier, then add the lint/format scripts to `package.json`.

## 6. Add Scripts to `package.json`

```json
{
  "scripts": {
    "test": "playwright test",
    "report": "allure generate allure-results --clean -o allure-report && allure open allure-report",
    "typecheck": "tsc --noEmit",
    "lint": "eslint src/ utils/",
    "lint:fix": "eslint src/ utils/ --fix",
    "format": "prettier --write src/ utils/",
    "format:check": "prettier --check src/ utils/"
  }
}
```

## 7. Verify Setup

```bash
npx tsc --noEmit
npm run lint
npm run format:check
npm test
```

All four must pass before moving on.

## 8. Next Steps

For each API resource, use the `/new-resource` skill to create:
- `src/api/types/{resource}.d.ts`
- `src/api/factories/{resource}.factory.ts`
- `src/api/api-clients/{resource}-api-client.ts`
- `src/tests/{resource}.api.spec.ts`
