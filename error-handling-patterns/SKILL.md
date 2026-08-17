---
name: error-handling-patterns
description: Design robust error handling — validate inputs, fail fast, propagate with context, distinguish recoverable from fatal errors, and clean up resources — across languages; use when adding error handling, fixing swallowed exceptions, hardening a code path, or reviewing failure behavior.
---

# Error Handling Patterns

Robust software behaves predictably when things go wrong. This skill covers how to detect errors early, propagate them with enough context to diagnose, decide where to recover versus fail, and never leak resources or hide failures. The patterns are language-agnostic and apply to exceptions and result/error-value styles alike.

## When to use this skill

- You are adding or hardening error handling on a code path.
- The user reports swallowed errors, silent failures, or unhelpful stack traces.
- You are integrating with I/O, networks, or external services that can fail.
- You are reviewing whether failure modes are handled correctly.

## Instructions

1. Validate at the boundary, fail fast inside. Check and normalize untrusted input where it enters the system; once past the boundary, treat inputs as valid and let internal invariant violations crash loudly rather than limp along.
2. Distinguish error categories and handle each deliberately:
   - **Expected/recoverable** (e.g., not found, validation, transient network): return a typed error or handle locally.
   - **Programmer errors** (bugs, broken invariants): fail loud, do not catch broadly.
   - **Fatal/unrecoverable**: log with full context and terminate the operation cleanly.
3. Propagate with context, don't swallow. Never catch-and-ignore. When rethrowing/wrapping, add what you were doing and the relevant identifiers, and preserve the original cause (wrapped exception / error chaining) so the root cause survives.
4. Catch narrowly and at the right layer. Catch specific error types, not everything. Handle an error at the layer that can actually do something about it (retry, fallback, user message); otherwise let it propagate.
5. Guarantee cleanup. Release resources (files, locks, connections) with `finally`/`defer`/context managers/RAII so they are freed on every path, including the error path.
6. Make retries safe. Only retry idempotent operations; use bounded retries with backoff and jitter; set timeouts on all remote calls; consider a circuit breaker for repeated failures.
7. Don't leak internals to users or logs. Return safe, actionable messages to users; log full detail internally without exposing secrets, tokens, or PII.

## Examples

Wrapping with context instead of swallowing:

```python
# Bad: original cause and context are lost.
try:
    data = load(path)
except Exception:
    return None

# Good: narrow catch, added context, preserved cause.
try:
    data = load(path)
except FileNotFoundError as e:
    raise ConfigError(f"config missing at {path}") from e
```

Guaranteed cleanup plus bounded, backed-off retry:

```python
for attempt in range(3):
    conn = pool.acquire()
    try:
        return conn.fetch(query, timeout=5)   # timeout on remote call
    except TransientError:
        if attempt == 2:
            raise
        time.sleep(2 ** attempt + random.random())  # backoff + jitter
    finally:
        pool.release(conn)                    # freed on every path
```

## Checklist

- [ ] Untrusted inputs are validated at the boundary.
- [ ] Recoverable, programmer, and fatal errors are handled distinctly.
- [ ] No error is silently swallowed; wrapped errors preserve the original cause and context.
- [ ] Catches are narrow and placed at the layer that can act on them.
- [ ] Resources are released on all paths (finally/defer/context manager/RAII).
- [ ] Remote calls have timeouts; retries are bounded, backed off, and idempotent-only.
- [ ] User-facing messages are safe; logs carry detail without secrets or PII.
