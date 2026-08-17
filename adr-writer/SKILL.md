---
name: adr-writer
description: Write Architecture Decision Records (ADRs) that capture a significant technical decision, its context, the options considered, and consequences, in a numbered immutable log — use when making an architectural or technology choice, documenting why a design was chosen, superseding a past decision, or when a user asks how to record engineering decisions.
---

# ADR Writer

An Architecture Decision Record documents a single significant technical decision: the context that forced a choice, the options weighed, the decision made, and its consequences. ADRs are short, numbered, immutable files stored in the repo so future engineers understand *why* the system is the way it is — and can revisit decisions deliberately.

## When to use this skill

- Making a decision that is costly to reverse: framework, database, protocol, boundary, or major pattern.
- Choosing between competing approaches where the rationale should outlive the discussion.
- Superseding or revisiting a previous architectural decision.
- Onboarding needs, or a reviewer asks "why did we do it this way?"
- A user asks how to document decisions or set up an ADR log.

## Instructions

1. **Store ADRs together**, typically `docs/adr/` (or `docs/decisions/`), one file per decision, numbered sequentially: `0007-use-postgres-for-billing.md`.
2. **Use a consistent template** with these sections: Title, Status, Context, Decision, Consequences. Optionally add Options Considered and References.
3. **Title** it as the decision, not the problem: "Use PostgreSQL for the billing service."
4. **Status** is one of `Proposed`, `Accepted`, `Deprecated`, or `Superseded by ADR-00XX`. Set the date.
5. **Context** — describe the forces at play: requirements, constraints, assumptions, and the problem. State facts, not the decision. This is the most important section.
6. **Decision** — state the choice in active voice: "We will …". Be specific and unambiguous.
7. **Consequences** — list the results, both positive and negative: what becomes easier, what becomes harder, new obligations, and risks accepted. Honesty here is what makes the ADR valuable.
8. **Options Considered** (optional but recommended) — briefly list alternatives with their pros/cons so the reasoning is auditable.
9. **Keep ADRs immutable.** Do not rewrite an accepted decision; instead write a new ADR that supersedes it and update the old one's status with a link.
10. **Keep it short** — one to two pages. If it grows huge, the decision is probably several decisions.

## Examples

```markdown
# 7. Use PostgreSQL for the billing service

Status: Accepted — 2026-08-17

## Context
Billing needs strong transactional guarantees for money movements and
complex relational queries for invoicing. Traffic is moderate (hundreds
of writes/sec). The team already operates PostgreSQL for other services
and has on-call familiarity with it.

## Decision
We will use PostgreSQL as the primary datastore for the billing service,
using serializable transactions for balance-affecting operations.

## Options Considered
- **PostgreSQL** — strong ACID, rich SQL, team expertise. (Chosen)
- **DynamoDB** — great scaling, but weak multi-item transactions and no
  ad-hoc relational queries; higher modeling effort for invoices.
- **MongoDB** — flexible schema, but transactional story is weaker for
  financial invariants.

## Consequences
- Positive: reliable money invariants; reuses existing ops tooling.
- Negative: we own vertical-scaling limits; must plan sharding if volume
  grows 100x. Accepted given current scale.
- We must add read replicas before enabling heavy reporting queries.
```

## Checklist

- [ ] Stored in the ADR directory with a sequential number and slug.
- [ ] Title states the decision, not just the topic.
- [ ] Status and date are set (Proposed/Accepted/Deprecated/Superseded).
- [ ] Context explains the forces without pre-stating the decision.
- [ ] Decision is specific and in active voice ("We will …").
- [ ] Consequences list both benefits and costs/risks honestly.
- [ ] Superseded decisions are linked, not overwritten.
