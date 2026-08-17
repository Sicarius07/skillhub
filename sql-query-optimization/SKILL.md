---
name: sql-query-optimization
description: Diagnose and speed up slow SQL queries using execution plans, indexing, query rewrites, and schema-aware tuning — covering EXPLAIN/ANALYZE reading, missing and misused indexes, N+1 patterns, sargability, joins, and pagination; use when a query or report is slow, a database is CPU/IO-bound, timeouts appear under load, or you need to review a query before it ships.
---

# SQL Query Optimization

Slow queries usually stem from missing indexes, non-sargable predicates, bad join order, or fetching far more data than needed. This skill provides a systematic method: measure with the execution plan, find the dominant cost, and apply the smallest change that removes it — then verify the improvement.

## When to use this skill

- A specific query, endpoint, or report is slow or timing out.
- The database is CPU- or IO-bound and you need to find the culprits.
- Latency degrades as data volume or concurrency grows.
- Reviewing a query in a PR before it reaches production data sizes.

## Instructions

1. **Reproduce and measure.** Capture the actual query and run `EXPLAIN ANALYZE` (or the engine's equivalent) against representative data. Note total time and rows.
2. **Read the plan for the bottleneck.** Look for sequential/full scans on large tables, high `rows` estimates vs. actual, expensive sorts, nested-loop joins over big row counts, and steps consuming most of the time.
3. **Check for missing indexes.** If a large table is scanned to satisfy a `WHERE`, `JOIN`, or `ORDER BY`, add an index on the filtered/joined columns. Use composite indexes ordered to match the query's equality-then-range predicates.
4. **Make predicates sargable.** Avoid wrapping indexed columns in functions or type casts (`WHERE date(created_at) = ...`), leading wildcards (`LIKE '%x'`), and implicit conversions — they defeat indexes. Rewrite to compare the raw column to a computed bound.
5. **Select only what you need.** Replace `SELECT *` with required columns; a covering index can then satisfy the query without touching the table.
6. **Fix N+1 access.** Replace per-row queries in application loops with a single set-based join or `IN` query.
7. **Optimize joins.** Ensure join columns are indexed and typed identically. Filter early to shrink intermediate results. Reconsider join order only if the planner misestimates.
8. **Paginate efficiently.** Prefer keyset/seek pagination (`WHERE id > :last ORDER BY id LIMIT n`) over large `OFFSET`, which scans and discards rows.
9. **Refresh statistics.** Run `ANALYZE`/update stats so the planner has accurate row estimates; stale stats cause bad plans.
10. **Verify and guard.** Re-run `EXPLAIN ANALYZE` to confirm the plan changed and time dropped. Add the query to a regression check if it is critical.

## Examples

Turning a non-sargable filter into an index-friendly range:

```sql
-- Slow: function on the column prevents index use
SELECT id, total FROM orders
WHERE date(created_at) = '2026-08-17';

-- Fast: half-open range hits an index on created_at
SELECT id, total FROM orders
WHERE created_at >= '2026-08-17' AND created_at < '2026-08-18';

CREATE INDEX idx_orders_created_at ON orders(created_at);
```

Composite index matching an equality + range + sort query:

```sql
-- Query: filter by customer, range on date, newest first
SELECT id FROM orders
WHERE customer_id = 42 AND created_at >= '2026-01-01'
ORDER BY created_at DESC
LIMIT 20;

-- Index ordered: equality column first, then the range/sort column
CREATE INDEX idx_orders_cust_created ON orders(customer_id, created_at DESC);
```

Keyset pagination instead of large OFFSET:

```sql
-- Instead of: ... ORDER BY id LIMIT 20 OFFSET 100000
SELECT * FROM orders
WHERE id > :last_seen_id
ORDER BY id
LIMIT 20;
```

## Checklist

- [ ] The slow query was measured with EXPLAIN ANALYZE on realistic data.
- [ ] The dominant cost step in the plan was identified.
- [ ] Large-table scans backing WHERE/JOIN/ORDER BY have supporting indexes.
- [ ] Predicates are sargable (no functions/casts/leading wildcards on indexed columns).
- [ ] The query selects only needed columns.
- [ ] Application-side N+1 loops were replaced with set-based queries.
- [ ] Pagination uses keyset seeking for large datasets.
- [ ] Planner statistics are current.
- [ ] The improvement was verified by re-running the plan.
