---
name: add-observability
description: Instrument a service with metrics, traces, and structured logs so production issues can be diagnosed from telemetry alone — use when a system is a black box in prod, incidents take too long to diagnose, you are adding a new critical path, or the user asks for observability, monitoring, tracing, structured logging, SLIs/alerts, or better production visibility.
---

# Add Observability

Observability is the ability to answer new questions about a running system without shipping new code. You achieve it with three complementary signals: metrics (cheap aggregate numbers for "how much/how often/how fast"), traces (the causal path of one request across services), and logs (rich per-event detail for the specifics). This skill instruments a critical path with all three, tied together so you can pivot from a bad metric to the exact traces and logs behind it.

## When to use this skill

- A service is a black box in production and incidents take too long to diagnose.
- You are building or hardening a critical path (checkout, auth, ingestion) and want it observable from day one.
- Alerts are noisy or missing, or you cannot answer "is it the DB, the network, or us?".
- The user asks for metrics, tracing, structured/JSON logging, dashboards, SLIs/SLOs, or production visibility.

## Instructions

1. Start from the questions you need to answer in an incident ("which endpoint is slow", "which dependency is failing", "what is the error rate for tenant X"). Instrument to answer those, not to collect everything.
2. Prefer an open standard (e.g. OpenTelemetry) so metrics, traces, and logs share one instrumentation layer and can be exported to any backend without lock-in.
3. Metrics — cover the RED signals for each service (Rate, Errors, Duration) and USE for resources (Utilization, Saturation, Errors). Use histograms for latency (so you get p50/p95/p99), counters for events, gauges for levels. Keep label/tag cardinality bounded — never put user IDs or raw URLs in labels.
4. Traces — create a span per meaningful operation and propagate context across service and async boundaries so a request stays one trace. Record key attributes on spans (route, status, tenant, downstream target) and mark error spans.
5. Logs — make them structured (JSON) with consistent fields, not free-form strings. Always include the trace/span id, a request/correlation id, level, timestamp, service, and event-specific fields. Log at the right level; reserve ERROR for actionable failures.
6. Correlate the three: inject the same trace id into logs and expose it on metric exemplars so you can jump metric -> trace -> logs for one request.
7. Add context, not noise: attach tenant, request id, version/build, and region as consistent attributes. Never log secrets, tokens, PII, or full payloads.
8. Control cost and overhead: sample traces (head or tail) sensibly, aggregate high-volume logs, and set retention. Ensure instrumentation is low-overhead on the hot path.
9. Make it actionable: build dashboards around the RED/USE signals, define SLIs and alert on symptoms (error rate, latency SLO burn) rather than causes (CPU), and route alerts with enough context to start debugging.
10. Verify by using it: trigger a failure in staging and confirm you can detect it via metrics, find the trace, and read the logs — end to end — without adding new code.

## Examples

A structured, trace-correlated log line plus RED-style metrics around a handler.

```python
# One instrumented request path (pseudo-OpenTelemetry).
with tracer.start_as_current_span("checkout", attributes={"route": "/checkout", "tenant": tenant}) as span:
    start = clock.now()
    try:
        result = process_checkout(cart)
        requests_total.inc(labels={"route": "/checkout", "status": "200"})
    except PaymentError as e:
        span.record_exception(e); span.set_status("error")
        requests_total.inc(labels={"route": "/checkout", "status": "402"})
        raise
    finally:
        latency_seconds.observe(clock.now() - start, labels={"route": "/checkout"})  # histogram

# Structured log correlated to the trace:
log.info("checkout_completed", extra={
    "trace_id": span.trace_id, "request_id": rid, "tenant": tenant,
    "amount_cents": cart.total, "items": len(cart.items),
})   # -> {"level":"INFO","event":"checkout_completed","trace_id":"...","tenant":"..."}
```

Symptom-based alert and the signal choices behind it.

```text
Alert: p95 latency of /checkout > 800ms for 5m  (symptom, user-visible) — page.
Alert: error rate of /checkout > 2% for 5m — page.
Signal choices:
  latency  -> histogram (need percentiles)   labels: route  (NOT user id)
  requests -> counter    labels: route,status
  in-flight-> gauge
Pivot on fire: dashboard -> exemplar trace id -> logs filtered by that trace_id.
```

## Checklist

- [ ] Instrumentation is driven by concrete incident questions, using an open standard where possible.
- [ ] RED metrics per service and USE metrics per resource; latency as histograms; label cardinality bounded.
- [ ] Spans cover meaningful operations and context propagates across service/async boundaries.
- [ ] Logs are structured with consistent fields including trace/request ids and levels.
- [ ] Metrics, traces, and logs are correlated so you can pivot between them for one request.
- [ ] No secrets, tokens, PII, or full payloads in logs, labels, or span attributes.
- [ ] Sampling, aggregation, and retention keep cost and hot-path overhead in check.
- [ ] Dashboards exist and alerts fire on symptoms (SLI/SLO), with enough context to debug.
- [ ] Verified end to end by triggering a failure and diagnosing it from telemetry alone.
