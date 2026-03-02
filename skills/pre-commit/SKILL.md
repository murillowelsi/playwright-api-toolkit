---
name: pre-commit
description: Run all tests, TypeScript type check, lint, and format check. Use before commits and PRs.
---

# Full Check

Run comprehensive quality checks before committing. **Mandatory** before any PR.

## Steps

1. **Run Playwright tests**
2. **Run TypeScript type check**
3. **Run lint**
4. **Run format check**

```bash
npm test && npx tsc --noEmit && npm run lint && npm run format:check
```

## Individual Commands

```bash
# Run a specific spec file
npx playwright test src/tests/books.api.spec.ts

# TypeScript check only
npx tsc --noEmit

# Lint only
npm run lint

# Auto-fix lint and format
npm run lint:fix && npm run format

# Show HTML report after test run
npx playwright show-report

# Run in debug mode
npx playwright test --debug
```

## Interpreting Results

### All Tests Pass

```
18 passed (3.6s)
```

### Test Failures

```
✗  src/tests/books.api.spec.ts:36 › Books API › GET /api/book/:id › should return a single book by id
  Error: expect(received).toBe(expected)
  Expected: 200
  Received: 404
```

Fix: Check the endpoint URL and API behavior. See `/debug-test` for a full diagnosis checklist.

### TypeScript Errors

```
src/tests/books.api.spec.ts:12:5 - error TS2345:
Argument of type 'unknown' is not assignable to parameter of type 'string'.
```

Fix: Add proper type annotation or update the type definition.

## Common Issues

### API Returns 404 or Times Out

- Check `BASE_URL` in `.env`
- Check if the API server is running (free tier may cold-start)
- Increase timeout in `playwright.config.ts`:
  ```typescript
  use: {
    actionTimeout: 30000,
  }
  ```

### TypeScript Errors After Type Changes

Run `npx tsc --noEmit` to see all errors, then fix field types.

## Rules

- Must pass before every commit and PR
- Never use `--no-verify` or comment out failing tests to force a pass
- Fix the root cause — if a test fails due to an API change, update the assertion or types
