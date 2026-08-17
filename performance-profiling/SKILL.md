---
name: performance-profiling
description: Measure and profile an application to locate real, data-backed performance bottlenecks (CPU hotspots, slow queries, memory pressure, latency tails) before changing code — use whenever something is "slow", latency/throughput regressed, cost is spiking, or someone proposes an optimization without measurements.
---

# Performance Profiling

Optimization without measurement is guessing. This skill establishes a disciplined loop: reproduce, measure, find the dominant cost, fix that one thing, then re-measure. It steers you away from micro-optimizing code that isn't hot and toward the small number of changes that actually move the needle.

## When to use this skill

- A user reports the app, endpoint, page, job, or query is "slow" or "laggy".
- Latency (p50/p95/p99), throughput, or infrastructure cost has regressed.
- Someone wants to optimize code but has no profile proving where time goes.
- CPU, memory, or I/O usage is unexpectedly high.
- You need to set or verify a performance budget before shipping.

## Instructions

1. **Define the target metric and goal.** State exactly what you are optimizing (e.g. "p95 latency of `GET /search`") and the target ("under 200ms"). Without a number you cannot know when you are done.
2. **Establish a reproducible baseline.** Create a repeatable scenario (a load test, a benchmark, a representative request). Run it several times and record the metric. Note the environment (prod vs. dev, dataset size, cache state).
3. **Measure, don't guess.** Attach a profiler appropriate to the runtime:
   - CPU: sampling profiler / flame graph (e.g. `perf`, `py-spy`, Node `--prof`, `pprof`, async-profiler).
   - Memory: heap snapshots and allocation profiles.
   - I/O and DB: query logs, `EXPLAIN ANALYZE`, slow-query logs, distributed traces.
4. **Find the dominant cost.** Read the flame graph or trace top-down. Rank costs by total time. Apply Amdahl's law: a 90% speedup of something that is 3% of runtime is nearly worthless.
5. **Form one hypothesis and one change.** Isolate a single bottleneck (N+1 query, unbounded allocation, missing index, sync call in a hot loop, needless serialization). Change only that.
6. **Re-measure against the baseline.** Confirm the metric improved and that you did not regress correctness or another metric. Keep or revert based on data.
7. **Repeat** until the goal is met or remaining costs are not worth the complexity.
8. **Lock it in.** Add a benchmark or budget check to CI so the regression cannot silently return, and document what was slow and why.

## Examples

Finding an N+1 query hidden behind a slow endpoint using a trace summary:

```
$ curl -s localhost:8080/orders?trace=1 | jq '.spans | group_by(.name) | map({name: .[0].name, count: length, total_ms: (map(.ms) | add)}) | sort_by(-.total_ms)'
[
  { "name": "db.query users", "count": 213, "total_ms": 1840 },   # <-- 213 identical queries: N+1
  { "name": "http.handler",   "count": 1,   "total_ms": 2010 },
  { "name": "db.query orders", "count": 1,   "total_ms": 45 }
]
```

The fix is to batch the 213 user lookups into one query, then re-run the same trace to confirm `count` drops to 1.

Recording a CPU flame graph before touching code:

```
# Node.js
node --prof server.js        # run load, then:
node --prof-process isolate-*.log > profile.txt   # inspect "ticks" by function

# Linux native / JVM
perf record -F 99 -g -- ./app && perf script | flamegraph.pl > flame.svg
```

## Checklist

- [ ] A single, numeric target metric and goal are written down.
- [ ] A reproducible baseline was measured (multiple runs, known environment).
- [ ] A profiler/trace — not intuition — identified the dominant cost.
- [ ] Only one bottleneck was changed per measurement cycle.
- [ ] The metric was re-measured and improvement confirmed with data.
- [ ] Correctness and other metrics were not regressed.
- [ ] A benchmark or performance budget guards against regression in CI.
