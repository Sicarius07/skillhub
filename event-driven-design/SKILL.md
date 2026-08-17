---
name: event-driven-design
description: Design event-driven systems with well-named events, appropriate messaging infrastructure, and correct delivery semantics — covering event vs. command, event schemas and versioning, queues vs. logs, idempotent consumers, ordering, dead-letter handling, and the outbox pattern; use when decoupling services, building async workflows, integrating via messaging, or debugging duplicate, lost, or out-of-order messages.
---

# Event-Driven Design

Event-driven architectures decouple producers from consumers, enabling scalability and resilience — at the cost of eventual consistency and harder debugging. This skill helps you model meaningful events, choose the right messaging substrate, and build consumers that are idempotent and tolerant of retries, duplicates, and reordering.

## When to use this skill

- Decoupling services so they can scale and fail independently.
- Building asynchronous workflows (notifications, pipelines, fan-out).
- Integrating systems through a message broker or event log.
- Diagnosing duplicate processing, lost messages, or out-of-order handling.

## Instructions

1. **Distinguish events from commands.** An *event* is an immutable fact about the past (`OrderPlaced`), broadcast to any interested party. A *command* is a request to do something (`PlaceOrder`), directed at one handler. Name events in past tense.
2. **Design the event schema.** Include a unique event ID, event type, timestamp, aggregate/entity ID, a version, and only the data consumers need. Prefer explicit fields over dumping internal models.
3. **Version events from day one.** Treat the schema as a contract. Evolve additively; introduce a new version for breaking changes and support both during transition.
4. **Choose the transport.** Use a *queue* (work distribution, one consumer per message, e.g., SQS/RabbitMQ) for task processing; use a *log* (retained, replayable, multiple independent readers, e.g., Kafka) for event streaming and multiple subscribers.
5. **Pick delivery semantics realistically.** Most systems are at-least-once, so duplicates happen. Design for it rather than assuming exactly-once.
6. **Make consumers idempotent.** Deduplicate by event ID (store processed IDs) or make the operation naturally idempotent (upserts, conditional updates) so reprocessing is safe.
7. **Handle ordering explicitly.** If order matters, partition by the entity key so related events go to the same ordered partition; otherwise design handlers to tolerate reordering.
8. **Publish reliably with the outbox pattern.** Write the event to an outbox table in the same transaction as the state change, then relay it to the broker — avoiding the dual-write problem where the DB commits but the publish fails.
9. **Plan for failure.** Configure retries with backoff and a dead-letter queue for poison messages; alert on DLQ growth and provide a replay path.
10. **Make it observable.** Correlate events with a trace/correlation ID; monitor lag, throughput, and DLQ depth.

## Examples

An event envelope and an idempotent consumer:

```json
{
  "event_id": "evt_01H...",
  "type": "OrderPlaced",
  "version": 1,
  "occurred_at": "2026-08-17T10:00:00Z",
  "aggregate_id": "ord_789",
  "data": { "customer_id": "cus_123", "total_cents": 4000 }
}
```

```python
def handle(event):
    # Idempotency: skip if we've already processed this event_id
    if processed_events.exists(event["event_id"]):
        return
    with db.transaction():
        apply_effect(event)                       # e.g., create shipment
        processed_events.insert(event["event_id"])
```

Transactional outbox (publish reliably):

```sql
BEGIN;
  INSERT INTO orders (...) VALUES (...);
  INSERT INTO outbox (event_id, type, payload, published)
  VALUES ('evt_01H...', 'OrderPlaced', :json, false);
COMMIT;
-- A relay process reads unpublished outbox rows, sends to the broker,
-- and marks them published (retrying safely on failure).
```

## Checklist

- [ ] Events are past-tense facts; commands are separate and directed.
- [ ] Event schema includes ID, type, version, timestamp, and entity ID.
- [ ] Schema evolution is additive and versioned.
- [ ] Transport (queue vs. log) matches the consumption pattern.
- [ ] Consumers are idempotent and safe under at-least-once delivery.
- [ ] Ordering requirements are handled via partitioning or tolerant handlers.
- [ ] Publishing avoids dual-write via an outbox or equivalent.
- [ ] Retries, backoff, and a dead-letter queue with a replay path exist.
- [ ] Events carry correlation IDs; lag and DLQ depth are monitored.
