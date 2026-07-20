---
name: e2e-testing-patterns
description: Builds reliable end-to-end tests. Use when writing or reviewing browser-level tests (Playwright/Cypress), debugging flaky E2E tests, or deciding what belongs in the E2E suite versus lower test levels.
---

# E2E Testing Patterns

## Overview

E2E tests are the most expensive tests you own — slowest to run, hardest to keep deterministic. They earn their cost only on critical user journeys; everything else belongs lower in the pyramid (see `test-driven-development`). The craft is writing few, high-value tests that fail only when the product is actually broken. Examples below use Playwright; every concept maps 1:1 to Cypress. For exact APIs, follow `source-driven-development`.

## When to Use

- Writing tests for critical user flows (signup, login, checkout, core CRUD)
- Debugging flaky or slow E2E suites
- Reviewing whether a test belongs at the E2E level at all
- Setting up E2E in CI (with `ci-cd-and-automation`)

## What Belongs in E2E — and What Doesn't

**Test with E2E:** critical user journeys, multi-step flows (wizards, checkout), auth flows, interactions that only exist in a real browser (drag-and-drop, file upload).

**Don't test with E2E:** unit-level logic and edge cases (unit tests), API contracts (integration tests), every input-validation permutation (one representative case at most), implementation details.

If a behavior can be verified at a lower level, it must be. E2E covers the wiring, not the logic.

## Selector Strategy

Priority order — each step down is more brittle:

1. **Role and accessible name:** `getByRole('button', { name: 'Submit' })` — tests what users perceive and doubles as an accessibility check
2. **Label / placeholder / text:** `getByLabel('Email')`, `getByText('Welcome')`
3. **Test id:** `getByTestId('task-row')` — for elements with no accessible semantics
4. **Never:** CSS classes, tag chains, `nth-child` — they break on every refactor

## Determinism Rules

Flakiness is almost always a waiting or isolation bug, not "just E2E being E2E":

- **Never use fixed sleeps** (`waitForTimeout(3000)`). Wait for conditions: URL, element state, or a specific network response.
- **Prefer auto-waiting assertions** (`await expect(locator).toBeVisible()`) over manual waits — they retry until timeout.
- When an action triggers a request, register the wait *before* the action (`page.waitForResponse(...)`, then click).
- **Tests are independent:** any test can run alone, in any order, in parallel. No test depends on state another test created.
- **Each test owns its data:** create unique fixtures in setup (e.g. `test-${crypto.randomUUID()}@example.com` — not `Date.now()`, which collides across parallel workers), clean up in teardown. Prefer seeding via API/DB over clicking through the UI to build preconditions.
- Log in via API or a stored auth state, not through the login form in every test — the login *flow* gets its own dedicated test.

## Structural Patterns

**Page objects** — once the suite has multiple tests touching the same screens, wrap each page's locators and actions in a class (`LoginPage.login(email, pw)`) so UI changes are absorbed in one place. Skip the ceremony for a 5-test suite.

**Network mocking** — mock third-party services (payments, email) always, and your own API when testing frontend behavior in isolation (error states, empty states, slow responses):

```typescript
await page.route('**/api/users', route =>
  route.fulfill({ status: 500, body: JSON.stringify({ error: 'boom' }) }));
await page.goto('/users');
await expect(page.getByText('Failed to load users')).toBeVisible();
```

Keep a small set of fully-unmocked journey tests — mocks can't catch contract drift.

**Visual regression** — screenshot assertions (`toHaveScreenshot`) for layout-critical pages only; broad screenshot suites drown you in false diffs.

**Accessibility scan** — run axe (`@axe-core/playwright`) on key pages and assert zero violations; it's a cheap add-on to existing tests.

## CI Configuration Essentials

- Retries in CI only (e.g. 2) — a pass-on-retry is a *flake signal to fix*, not a green light to ignore
- Capture trace/screenshot/video **on failure only**
- `forbidOnly` in CI so a stray `.only` can't silently skip the suite
- Shard across workers when the suite exceeds ~5 minutes
- Run against a production build, not the dev server

## Debugging a Failing E2E Test

1. Read the failure artifact first — trace viewer / screenshot / video usually shows the answer
2. Run the single test headed and step through (`--headed`, `--debug`, `page.pause()`)
3. Distinguish: product bug (fix the product — the test did its job), broken assumption (update the test), or flake (fix the wait/isolation — never add a sleep and move on)
4. For intermittent failures, run the test in a loop (`--repeat-each`) to make it reproducible, then apply `debugging-strategies`

## Red Flags

- Fixed sleeps anywhere in the suite
- Tests that depend on execution order or shared accounts/data
- CSS/nth-child selectors
- E2E tests for logic a unit test could cover
- Flaky tests "fixed" by re-running until green
- Suites that build all test state by clicking through the UI
- No failure artifacts configured in CI

## Verification

After writing or modifying E2E tests:

- [ ] Each test runs green alone, in parallel, and repeated (`--repeat-each=10` for anything previously flaky)
- [ ] Selectors are role/label/test-id based
- [ ] No fixed sleeps; all waits are condition-based
- [ ] Test data is created and cleaned up per test
- [ ] Failure artifacts (trace/screenshot) are produced in CI
- [ ] The suite still covers only critical journeys — nothing that belongs at a lower level
