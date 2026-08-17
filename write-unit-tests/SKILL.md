---
name: write-unit-tests
description: Write focused, fast, deterministic unit tests for a single unit of behavior — use when adding tests for a new or existing function/class/module, hardening a bug fix, or when the user asks for unit tests, better test coverage of pure logic, or help structuring assertions and edge cases.
---

# Writing Unit Tests

A unit test verifies one behavior of one unit (a function, method, or small class) in isolation, runs in milliseconds, and gives the same result every time. Good unit tests are precise enough that a failure points almost directly at the defect, and readable enough that they double as documentation of intended behavior.

## When to use this skill

- Adding tests for a newly written pure function or class.
- Backfilling tests around existing logic before changing it.
- Locking in a bug fix with a regression test.
- The user asks for "unit tests", more coverage of business logic, or help with edge cases and assertions.
- Reviewing tests and wanting to tighten flaky, slow, or over-broad ones.

## Instructions

1. Identify the unit and its observable contract: inputs, outputs, side effects, and error conditions. Test the contract, not private internals.
2. Structure each test as Arrange–Act–Assert: set up inputs, call the unit once, assert on the result. Keep the act step to a single call.
3. Give each test one reason to fail. Prefer many small tests over one test with a dozen assertions. Name tests after the behavior and condition (e.g. `parses_negative_amounts`).
4. Enumerate cases deliberately: the happy path, boundaries (0, 1, max, empty, off-by-one), invalid input, and error/exception paths. Consider a table/parameterized test for many similar cases.
5. Keep it deterministic. Inject or freeze clocks, randomness, and IDs. No real network, filesystem, DB, sleeps, or dependence on test execution order.
6. Isolate collaborators only when they are slow or nondeterministic. Prefer real simple objects; use a fake/stub for the awkward dependency rather than mocking everything.
7. Assert on meaningful values, not incidental ones. Avoid asserting exact strings that will churn; assert the parts that carry meaning.
8. Make failure messages informative — assert on the whole object or use descriptive matchers so a failure shows expected vs actual clearly.
9. Run the test in isolation and as part of the suite. Confirm it fails when the behavior is broken.

## Examples

Parameterized happy-path plus boundary and error cases for a `discount(price, pct)` function.

```python
import pytest

@pytest.mark.parametrize("price, pct, expected", [
    (100, 0,   100),   # no discount
    (100, 50,  50),    # half off
    (100, 100, 0),     # fully discounted (boundary)
])
def test_discount_applies_percentage(price, pct, expected):
    assert discount(price, pct) == expected

def test_discount_rejects_pct_over_100():
    with pytest.raises(ValueError):
        discount(100, 150)

def test_discount_does_not_mutate_inputs():
    p = Money(100)
    discount(p, 10)
    assert p == Money(100)   # no side effects
```

Freezing nondeterminism instead of sleeping or using the real clock.

```python
def test_token_expires_after_one_hour():
    clock = FakeClock(at="2026-01-01T00:00:00Z")
    token = issue_token(clock=clock)
    clock.advance(minutes=61)
    assert token.is_expired(now=clock.now()) is True
```

## Checklist

- [ ] Each test targets one behavior and has one clear reason to fail.
- [ ] Tests follow Arrange–Act–Assert with a single act step.
- [ ] Happy path, boundaries, invalid input, and error paths are covered.
- [ ] No real I/O, sleeps, wall-clock, or unseeded randomness; deterministic in any order.
- [ ] Test names describe the behavior and condition.
- [ ] Assertions target meaningful values and produce clear failure messages.
- [ ] I confirmed each test fails when its behavior is broken.
