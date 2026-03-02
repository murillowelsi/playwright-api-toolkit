---
name: debug-test
description: Diagnose and fix a failing Playwright API test. Use when a spec is erroring, returning unexpected status codes, or failing schema validation.
---

## Steps

1. Read the error output — check for status code mismatch first
2. Inspect the client method: verify path, HTTP method, and payload shape in `src/api/api-clients/`
3. Check fixture setup in `src/api/fixtures/api.fixture.ts` — confirm the correct `baseURL` and auth headers are set for that client's context
4. Verify `.env` has the correct values for the target environment
5. Run in debug mode to step through: `npx playwright test --debug <spec>`

## Common Causes

| Symptom | Likely Cause |
|---|---|
| `401 Unauthorized` | Token missing or expired — check `FINNHUB_API_KEY` (or equivalent) in `.env` and in the fixture's `extraHTTPHeaders` |
| `403 Forbidden` | Endpoint requires a paid API plan — add a `// Note:` comment and assert the 403 explicitly |
| `404 Not Found` | Path typo in the client method — check `src/api/api-clients/{resource}-api-client.ts` |
| `429 Too Many Requests` | Rate limit hit (common on free API tiers with parallel workers) — add `retries: 1` to `playwright.config.ts` or run with `--workers=1` |
| Fails on body assertions but status is 200 | API response shape changed — update the type interface in `src/api/types/{resource}.d.ts` |
| Passes locally, fails in CI | Env var not set in CI — check `.env.example` and CI secrets config |
| Flaky / intermittent failure | Rate limit or network timeout — increase `timeout` in `playwright.config.ts` or add `test.slow()` |
| Unexpected green when should be red | Test is tier-gated and asserting the wrong status — check if a plan upgrade changed the expected response |

## Tier-Gated Test Failures

If tests start failing after an API plan change (e.g. upgrading from free to paid tier):

- Tests asserting `403` or redirect behavior for gated endpoints will flip to red
- Update the test to assert the paid-tier response, or guard with an env var:

```ts
test.skip(env.API_PLAN === 'paid', 'Tier-gated: behaviour differs on paid plan');
```

## Fixture Debug Checklist

- Is the correct fixture (`{ stockApi }`, `{ cryptoApi }`, etc.) being destructured in the test?
- Does the fixture in `api.fixture.ts` use the right `baseURL` for this service?
- Are auth headers set in the fixture context, not in `playwright.config.ts`?
- Is the client class actually instantiated in the fixture (`new StockApiClient(context)`) and not left as a raw context?
