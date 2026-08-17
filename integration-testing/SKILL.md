---
name: integration-testing
description: Test multiple components working together across real boundaries (database, HTTP, message queue, filesystem) and decide when to use real dependencies versus test doubles — use when verifying wiring, serialization, transactions, or contracts between modules, or when the user asks for integration/end-to-end tests, testcontainers, or help choosing between mocks and real services.
---

# Integration Testing

Integration tests verify that independently-correct units actually work together across a real boundary — a database, HTTP API, queue, cache, or filesystem. They catch the failures unit tests cannot: wrong SQL, serialization mismatches, misconfigured wiring, broken transactions, and contract drift between services. They run slower than unit tests, so you write fewer of them and aim each at a real seam of risk.

## When to use this skill

- Verifying a repository/DAO issues correct queries and maps rows correctly.
- Testing an HTTP handler end to end: routing, deserialization, business call, status codes, response body.
- Confirming components are wired and configured correctly (dependency injection, migrations, connection strings).
- Checking transactions, retries, idempotency, or message publish/consume flows.
- The user asks for integration or end-to-end tests, testcontainers, contract tests, or advice on mocks vs. real dependencies.

## Instructions

1. Decide the scope: which components and which real boundary are under test. Name the seam of risk you are exercising (e.g. "order service ↔ Postgres transaction").
2. Choose the fidelity of each dependency:
   - Use the REAL thing (ideally an ephemeral container or in-memory equivalent) for the boundary you are actually testing — DB, message broker, your own HTTP handler.
   - Use a TEST DOUBLE (stub/fake/mock) for dependencies that are external, slow, costly, nondeterministic, or out of scope — third-party payment APIs, email senders, clocks.
3. Prefer fakes over mocks at boundaries: an in-memory or containerized real implementation catches integration bugs a hand-written mock will happily hide.
4. Manage the environment: spin up dependencies with a container library or a dedicated test instance; apply real migrations/schema; make setup automatic so the suite is runnable with one command.
5. Isolate state between tests: use transactions rolled back per test, unique schemas/namespaces, or truncate-and-seed. Never depend on data left by another test or on execution order.
6. Seed known fixtures, exercise the flow through the public entry point (not internals), and assert on observable results plus persisted state (row written, message enqueued).
7. Handle async and timing explicitly with polling/awaiting a condition — never fixed sleeps.
8. Keep integration tests separate from unit tests (tag or separate directory) so the fast suite stays fast and CI can run tiers.
9. Verify cleanup: tear down containers/connections, and confirm a repeat run is green (no leaked state).

## Examples

An API-level integration test against a real containerized database.

```python
# Uses a throwaway Postgres container; real migrations applied in a fixture.
def test_create_order_persists_and_returns_201(client, db):
    resp = client.post("/orders", json={"sku": "A1", "qty": 2})

    assert resp.status_code == 201
    order_id = resp.json()["id"]

    # Assert observable API result AND real persisted state.
    row = db.query("SELECT sku, qty FROM orders WHERE id = %s", order_id)
    assert row == ("A1", 2)

def test_create_order_rolls_back_on_payment_failure(client, db, fake_payments):
    fake_payments.will_decline()             # external dependency = test double
    resp = client.post("/orders", json={"sku": "A1", "qty": 2})

    assert resp.status_code == 402
    assert db.count("orders") == 0           # transaction rolled back
```

Choosing fidelity, at a glance.

```text
Under test now (the seam)   -> REAL (container / in-memory real impl)
Your own DB / queue / cache -> REAL
Third-party paid/flaky API  -> DOUBLE (stub canned responses)
Clock / random / UUID       -> DOUBLE (inject deterministic source)
```

## Checklist

- [ ] The seam of risk under test is exercised against a real dependency, not a mock.
- [ ] Out-of-scope, external, or flaky dependencies are replaced with deliberate test doubles.
- [ ] Environment (schema/migrations/containers) is provisioned automatically and runnable with one command.
- [ ] Each test isolates its state; the suite passes regardless of order and on repeat runs.
- [ ] Flows are driven through public entry points; assertions cover both response and persisted state.
- [ ] Async waits poll for a condition instead of sleeping.
- [ ] Integration tests are tiered/tagged separately from unit tests and clean up after themselves.
