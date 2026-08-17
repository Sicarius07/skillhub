---
name: database-schema-design
description: Design normalized, evolvable relational database schemas — tables, keys, constraints, relationships, indexing, and naming — that enforce data integrity and scale gracefully; use when creating a new schema, adding tables or columns, reviewing a data model, resolving anomalies from denormalized data, or preparing a schema to evolve without downtime.
---

# Database Schema Design

A relational schema is the long-lived backbone of most applications; mistakes here are expensive to fix later. This skill guides you to model data with correct normalization, strong constraints, and thoughtful indexing so the database enforces integrity and remains easy to evolve.

## When to use this skill

- Designing the schema for a new application or feature.
- Adding or restructuring tables, columns, or relationships.
- Reviewing a data model for integrity and performance concerns.
- Fixing update/insert/delete anomalies caused by redundant data.
- Preparing a schema so future changes can ship without downtime.

## Instructions

1. **Start from the domain model.** Turn entities into tables and relationships into foreign keys. Each table should represent one thing.
2. **Choose primary keys deliberately.** Prefer stable surrogate keys (auto-increment or UUID). Use natural keys only when truly immutable and unique.
3. **Normalize to 3NF first.** Ensure every non-key column depends on the key, the whole key, and nothing but the key. Eliminate repeating groups and redundant columns.
4. **Denormalize only with evidence.** Introduce redundancy (e.g., a cached count) only when a measured read pattern demands it, and document how it stays consistent.
5. **Model relationships explicitly.** One-to-many via a foreign key on the child; many-to-many via a join table with its own composite key.
6. **Enforce integrity with constraints.** Use `NOT NULL`, `UNIQUE`, `CHECK`, and `FOREIGN KEY` constraints. Let the database, not just the app, guarantee validity.
7. **Pick precise column types.** Use appropriate numeric/decimal types for money (never floats), timezone-aware timestamps, and enums or lookup tables for fixed sets.
8. **Index for real queries.** Index foreign keys and frequent filter/sort/join columns. Add composite indexes matching multi-column query patterns; avoid indexing everything.
9. **Name consistently.** Pick a convention (e.g., `snake_case`, singular or plural table names) and apply it everywhere, including keys (`customer_id`) and timestamps (`created_at`, `updated_at`).
10. **Design for evolution.** Prefer additive changes; make columns nullable or give defaults when adding; avoid destructive renames in one step. Keep audit columns.
11. **Consider soft vs. hard deletes** and archival strategy for large or regulated tables.

## Examples

A normalized order model with constraints and a join table:

```sql
CREATE TABLE customers (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  email       TEXT NOT NULL UNIQUE,
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE orders (
  id          BIGINT GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
  customer_id BIGINT NOT NULL REFERENCES customers(id),
  status      TEXT NOT NULL CHECK (status IN ('placed','paid','shipped','cancelled')),
  total_cents INTEGER NOT NULL CHECK (total_cents >= 0),
  created_at  TIMESTAMPTZ NOT NULL DEFAULT now()
);
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

CREATE TABLE order_items (
  order_id    BIGINT NOT NULL REFERENCES orders(id) ON DELETE CASCADE,
  product_id  BIGINT NOT NULL REFERENCES products(id),
  quantity    INTEGER NOT NULL CHECK (quantity > 0),
  price_cents INTEGER NOT NULL,
  PRIMARY KEY (order_id, product_id)
);
```

## Checklist

- [ ] Each table models a single, well-named concept.
- [ ] Primary keys are stable; natural keys used only when immutable.
- [ ] Schema is at least 3NF; any denormalization is justified and documented.
- [ ] Foreign keys, NOT NULL, UNIQUE, and CHECK constraints enforce integrity.
- [ ] Money uses integer/decimal types; timestamps are timezone-aware.
- [ ] Indexes match actual query patterns; no blanket over-indexing.
- [ ] Naming conventions are consistent across all objects.
- [ ] Planned changes are additive and can ship without downtime.
