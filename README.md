# playwright-api-toolkit

A Claude Code plugin with skills, commands, and agents for TypeScript + Playwright API test suites.

Enforces a typed client layer, fixture-based architecture, and consistent code quality gates across any REST API testing project.

---

## Install

```shell
/plugin install playwright-api-toolkit
```

Or add to your project's `.claude/settings.json`:

```json
{
  "enabledPlugins": ["playwright-api-toolkit"]
}
```

---

## What's included

### Skills

| Skill | Trigger | Description |
|-------|---------|-------------|
| `init-project` | `/init-project` | Scaffold the full base project structure (run once) |
| `new-resource` | `/new-resource` | Add all layers for a new API resource (client, types, factory, spec) |
| `add-types` | `/add-types` | Create TypeScript interfaces for an API response |
| `diagnose-test` | `/diagnose-test` | Diagnose and fix a failing Playwright API test |
| `install-package` | `/install-package` | Add an npm dependency with correct placement |
| `pre-commit` | `/pre-commit` | Run tests, type check, lint, and format check |

### Commands

| Command | Description |
|---------|-------------|
| `/code-review` | Full diff review before merging (like CodeRabbit) |
| `/create-pr` | Create a well-structured pull request |
| `/audit-types` | Find type smells (`any`, untyped `response.json()`, etc.) |
| `/create-handoff` | Write a session handoff document for continuity |
| `/debug` | Systematic debugger for any issue |

### Agents

| Agent | Description |
|-------|-------------|
| `quality-gate` | Proactive checks for naming, imports, typing, fixture usage |
| `security-auditor` | Scans for exposed secrets, insecure env handling, credential hygiene |
| `documentation-writer` | Keeps CLAUDE.md and README.md in sync with code changes |

### Hooks

- **After editing a `.ts` file** — auto-runs `lint:fix` and `format`
- **Before git commit** — enforces `lint` and `format:check`

---

## Architecture enforced

```
src/
  api/
    api-clients/      ← one typed client per resource, extends BaseApiClient
    types/            ← pure .d.ts interfaces, no functions or constants
    factories/        ← faker-based param builders
    fixtures/         ← api.fixture.ts wires clients into Playwright fixtures
    base-api-client.ts
    api-statuses.ts
  tests/
    {resource}.api.spec.ts
utils/
  env.ts
```

Key constraints:
- All clients extend `BaseApiClient` — no raw `APIRequestContext` in tests
- Every `response.json()` call is typed with an interface — no `any`
- `test`/`expect` imported from `@/api/fixtures/api.fixture`, never from `@playwright/test`
- Path aliases `@/*` and `@utils/*` everywhere — no relative `../../` imports
- Auth headers in fixtures only, never in `playwright.config.ts`
- Pre-commit gate: `npm test && npx tsc --noEmit && npm run lint && npm run format:check`

---

## Quick start

1. Install the plugin
2. Run `/init-project` to scaffold the base structure
3. Run `/new-resource` for each API resource you want to test
4. Run `/pre-commit` before every commit

---

## Requirements

- Node.js 18+
- `@playwright/test`
- `typescript`
- `eslint` + `prettier` (set up by `/init-project`)
