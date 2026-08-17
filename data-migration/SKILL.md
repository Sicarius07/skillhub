---
name: data-migration
description: Plan and execute safe, reversible schema and data migrations with zero or minimal downtime — covering expand/contract (parallel change), backfills, backward-compatible deploys, rollback plans, and verification; use when altering tables in production, renaming or dropping columns, changing types, moving or transforming large datasets, or splitting/merging services' data.
---

# Data Migration

Production migrations are risky because the schema, the running application code, and existing data must stay mutually compatible at every moment. This skill applies the expand/contract (parallel change) approach so migrations ship incrementally, remain reversible, and avoid locking or breaking live traffic.

## When to use this skill

- Altering a production table: adding/removing/renaming columns or changing types.
- Backfilling or transforming a large existing dataset.
- Moving data between tables, databases, or services.
- Any schema change that must ship without downtime or a maintenance window.

## Instructions

1. **Define the target and the invariant.** State the desired end state and what must remain true throughout (no data loss, no downtime, reversible at each step).
2. **Assess blast radius.** Identify table size, lock behavior of the operation, replication lag impact, and every reader/writer of the affected data.
3. **Use expand/contract.** Split the change into phases that are each independently deployable and backward compatible:
   - **Expand:** add the new structure (nullable column, new table) without removing the old. Deploy code that can read both.
   - **Migrate:** dual-write to old and new; backfill existing rows in batches.
   - **Contract:** switch reads to the new structure, stop writing the old, then drop the old structure in a later release.
4. **Make each migration reversible.** Write and test a `down`/rollback for every step, or ensure the step is safe to leave in place. Never combine an irreversible drop with the change that depends on it.
5. **Backfill in batches.** Update large tables in bounded chunks with throttling to avoid long locks, replication lag, and load spikes. Make backfills idempotent and resumable.
6. **Avoid blocking DDL.** Prefer non-locking operations (add nullable column, create index concurrently). Add NOT NULL / defaults in a separate step after backfill.
7. **Deploy code and schema in a compatible order.** Additive schema first, then code that uses it; remove code before removing schema it references.
8. **Verify at each phase.** Compare row counts and checksums between old and new; monitor error rates, latency, and lag. Have a go/no-go gate before contract.
9. **Keep a rollback runbook.** Document exact steps and the point of no return; ensure backups/snapshots exist before destructive steps.
10. **Clean up.** Only after the new path is proven in production, run the contract step to drop old columns/tables.

## Examples

Renaming a column with zero downtime via expand/contract:

```sql
-- Phase 1 (Expand): add new column, no lock, keep old
ALTER TABLE users ADD COLUMN full_name TEXT;   -- nullable, non-blocking

-- Phase 2 (Migrate): app dual-writes name -> full_name.
-- Backfill existing rows in batches:
UPDATE users SET full_name = name
WHERE full_name IS NULL AND id BETWEEN :lo AND :hi;   -- loop over ranges

-- Phase 3 (Contract): after reads switched to full_name and verified
ALTER TABLE users DROP COLUMN name;            -- ship in a later release
```

Batched, resumable backfill loop (pseudocode):

```
last_id = 0
loop:
  rows = SELECT id FROM users WHERE id > last_id AND full_name IS NULL
         ORDER BY id LIMIT 1000
  if rows empty: break
  UPDATE ... for those ids            # idempotent
  last_id = max(rows.id)
  sleep(short)                        # throttle to limit load/lag
```

## Checklist

- [ ] End state and safety invariants are written down.
- [ ] Change is split into expand / migrate / contract phases, each deployable alone.
- [ ] Each phase is backward compatible with the currently running code.
- [ ] Every step has a tested rollback or is safe to leave in place.
- [ ] DDL avoids long locks (nullable adds, concurrent index builds).
- [ ] Backfills run in throttled, idempotent, resumable batches.
- [ ] Schema and code deploy in a compatible order.
- [ ] Row counts/checksums and live metrics are verified before contracting.
- [ ] A backup/snapshot exists before any destructive step.
- [ ] Old structures are dropped only after the new path is proven.
