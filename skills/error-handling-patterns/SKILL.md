---
name: error-handling-patterns
description: Designs robust error handling. Use when implementing error paths, designing how an API reports failures, adding retries or fallbacks for external services, or improving reliability of existing code.
---

# Error Handling Patterns

## Overview

Error handling is API design: every function's failure behavior is part of its contract. Decide deliberately how errors are represented, where they are handled, and what the caller can rely on — instead of sprinkling try/catch until the stack traces stop. These patterns are language-agnostic; express them with your language's native mechanism (exceptions, `Result`, error returns).

## When to Use

- Implementing any code path that can fail (I/O, network, parsing, user input)
- Designing how an API or module reports failures to its consumers
- Integrating external services that can be slow, down, or wrong
- Debugging swallowed errors, duplicate logs, or unhelpful failure messages

## Choose the Mechanism

| Mechanism | Use for | Notes |
|---|---|---|
| **Exceptions / thrown errors** | Unexpected conditions the immediate caller can't fix | Keep the async chain intact; never swallow |
| **Result / error returns** | Expected, recoverable failures (validation, parsing, lookups) | Makes failure part of the type signature; caller must handle it |
| **Crash / panic** | Programming bugs and unrecoverable states (corrupt invariants, OOM) | Fail fast and loud; don't "handle" bugs into silence |

Pick one convention per boundary and keep it consistent — if some endpoints throw, others return null, and others return `{ error }`, no consumer can predict behavior (see `api-and-interface-design` for the error-response contract).

## Design an Error Taxonomy

Give the application a small hierarchy of typed errors instead of raw strings:

```typescript
class AppError extends Error {
  constructor(
    message: string,
    public code: string,            // machine-readable: "VALIDATION_ERROR"
    public details?: unknown,       // context for logs/debugging
    options?: { cause?: unknown },  // preserve the original error
  ) {
    super(message, options);
    this.name = this.constructor.name;
  }
}
class ValidationError extends AppError { /* code: VALIDATION_ERROR */ }
class NotFoundError extends AppError { /* code: NOT_FOUND */ }
class ExternalServiceError extends AppError { /* code + which service */ }
```

**Taxonomy rules:**
- One base type; a handful of subtypes matching how callers *react* differently — not one class per message.
- Every error carries a machine-readable `code`, a human-readable message, and enough context (ids, inputs) to debug.
- **Wrap external errors at the boundary** (`cause`/`from e`/`%w`): callers see your taxonomy, logs keep the original chain.
- Error messages say what happened and what to do about it. "Error occurred" is not a message.

## Handling Rules

1. **Fail fast.** Validate input at the boundary and reject early; don't let bad data travel.
2. **Handle where you can act.** Catch an error only at the level that can meaningfully respond (retry, fallback, translate, report). Everything else lets it propagate.
3. **Never swallow.** An empty catch block turns a bug into a mystery. Minimum viable handling is log-with-context and rethrow — but not both repeatedly: log an error **once**, at the top-level handler, not at every layer it passes through.
4. **Always clean up.** Use `finally` / context managers / `defer` so files, connections, and transactions are released on both paths. Transactions: commit on success, roll back on any error.
5. **Catch narrowly.** `except Exception` / bare `catch` hides bugs (and can eat cancellation signals). Catch the types you expect; let the rest crash visibly.
6. **Don't leak internals.** User-facing errors get the code and a safe message; stack traces and internal details go to logs only.
7. **Handle async failures.** Every promise/future needs an owner — unhandled rejections and fire-and-forget tasks are silent data loss.

## Resilience Patterns for External Dependencies

**Retry with exponential backoff (+ jitter)** — for transient failures (timeouts, 429/503). Retry only idempotent operations, cap attempts (~3), back off exponentially, add jitter to avoid thundering herds. Never retry validation or auth failures.

**Circuit breaker** — when a dependency is down, stop hammering it. Track consecutive failures; after a threshold, fail fast for a cooldown period (open), then let one probe request test recovery (half-open) before resuming (closed). Use a maintained library rather than hand-rolling in production.

**Graceful degradation** — define what the feature does when a dependency fails: serve cached/stale data, a default value, or a reduced experience. An explicit ordered fallback chain (`primary → secondary → cache → default`) beats an error page — but log every fallback activation, or outages become invisible.

**Error aggregation** — when validating many fields or processing batches, collect all failures and report them together (e.g. `AggregateError`) instead of stopping at the first. One round-trip per fix is miserable UX.

## Red Flags

- Empty catch blocks, or `catch` that only logs and continues as if it succeeded
- The same error logged at three different layers
- Broad catches (`except Exception`, bare `catch`) around large blocks
- Error messages without context ("something went wrong", no ids/inputs)
- External-service errors surfacing raw to users, or internals/stack traces in API responses
- Retries on non-idempotent operations, or unbounded retries
- Resources (connections, files, transactions) not released on the failure path
- Unhandled promise rejections / ignored error returns

## Verification

After implementing error handling:

- [ ] Every failure path has a defined behavior (propagate, translate, retry, fallback) — none swallowed
- [ ] Errors carry machine-readable codes and debugging context; external errors are wrapped with cause preserved
- [ ] Each error is logged exactly once, with context
- [ ] User-facing responses expose no internals
- [ ] Resources are cleaned up on failure paths (test this, don't assume)
- [ ] Retries are bounded, backed off, and idempotent-only; fallbacks are logged
- [ ] Tests cover the error paths, not just the happy path
