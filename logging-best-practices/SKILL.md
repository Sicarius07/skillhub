---
name: logging-best-practices
description: Produce structured, leveled, low-noise logs that are useful for debugging and observability without leaking secrets or drowning signal; use when adding or cleaning up logging, choosing log levels, introducing structured logging, or reducing log noise.
---

# Logging Best Practices

Logs exist to answer "what happened and why" during an incident, and to feed metrics, tracing, and alerts. This skill helps produce logs that are structured, appropriately leveled, correlatable, and quiet in the normal case — the opposite of unstructured, secret-leaking, high-cardinality noise that hides the one line that matters.

## When to use this skill

- You are adding logging to new code or cleaning up existing log statements.
- The user wants structured/JSON logging, better log levels, or less noise.
- You are debugging an issue where logs were unhelpful or missing context.
- You are wiring up observability (correlation IDs, tracing, metrics from logs).

## Instructions

1. Log structured key-value pairs, not string soup. Emit events as structured records (JSON or your logger's structured fields) so they are queryable. Keep the message a stable, low-cardinality string and put the variables in fields.
2. Use levels with clear, consistent meaning:
   - `ERROR`: an operation failed and needs attention.
   - `WARN`: unexpected but handled; may need attention if frequent.
   - `INFO`: significant business/lifecycle events (startup, request handled, job done).
   - `DEBUG`: detailed diagnostic info, off in production by default.
   - `TRACE`: very fine-grained, temporary.
3. Add correlation and context. Include request/trace IDs, user or tenant IDs (non-sensitive), and operation names so logs can be tied together across services. Use context propagation rather than threading IDs by hand.
4. Never log secrets or sensitive data. Exclude passwords, tokens, API keys, full card/SSN numbers, and PII. Redact or mask at the logging layer; assume logs may be widely readable and retained.
5. Keep the normal path quiet. Avoid logging inside tight loops or per-item in a large batch; log aggregates or sample instead. Don't log the same failure at every layer — log it once, where it's handled, with full context.
6. Make messages actionable and self-contained. State what happened and the identifiers needed to investigate. Log the error with its cause/stack, not just its message. Prefer past-tense factual events over vague notes.
7. Log for machines and humans. Ensure timestamps are present and in UTC/ISO-8601, and that the format is consistent so it parses cleanly in your aggregation system. Configure levels via environment, not code changes.

## Examples

Unstructured and noisy versus structured and useful:

```diff
- console.log("processing user " + user.email + " token=" + token);
- console.log("done");
+ log.info("order.processed", {
+   orderId: order.id,
+   userId: user.id,        // stable id, not email/PII
+   amountCents: order.total,
+   durationMs: elapsed,
+   traceId: ctx.traceId,
+ });
// token is never logged; a single event carries queryable fields.
```

Logging an error once, with cause, at the handling layer:

```python
try:
    charge(order)
except PaymentError as e:
    log.error("payment.failed", extra={
        "orderId": order.id, "provider": "stripe", "traceId": ctx.trace_id,
    }, exc_info=e)   # includes stack/cause; not re-logged upstream
    raise
```

## Checklist

- [ ] Logs are structured with a stable message and variable fields.
- [ ] Levels follow consistent, documented meanings.
- [ ] Correlation/trace IDs and operation context are attached.
- [ ] No secrets, credentials, or PII are logged; sensitive fields are redacted.
- [ ] The normal path is quiet; no per-item loop spam or duplicate error logging.
- [ ] Errors are logged once, at the handling layer, with cause/stack.
- [ ] Timestamps are UTC/ISO-8601 and levels are configurable at runtime.
