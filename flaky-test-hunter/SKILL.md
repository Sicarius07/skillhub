---
name: flaky-test-hunter
description: Diagnose and fix nondeterministic tests that pass and fail without code changes — use when CI is red intermittently, a test only fails "sometimes" or in parallel, a retry makes it pass, or the user asks to stabilize flaky tests, quarantine them, or track down timing/ordering/shared-state issues.
---

# Flaky Test Hunter

A flaky test passes and fails without any change to the code under test, eroding trust in the suite until people ignore real failures. Flakiness is a defect — usually in the test, sometimes in the code — caused by hidden nondeterminism: timing, ordering, shared state, unseeded randomness, real network, or concurrency. This skill reproduces the flake, classifies its cause, and removes the nondeterminism at the source.

## When to use this skill

- CI fails intermittently and re-running the same commit turns it green.
- A test fails only in parallel, only in CI, only under load, or only in a certain order.
- Someone added `@retry`, `sleep`, or "just run it again" to a test.
- The user asks to stabilize, de-flake, or quarantine tests, or to find the source of intermittent failures.

## Instructions

1. Confirm and reproduce. Run the suspect test many times in a loop; run it in isolation and within the full suite; try randomized order and parallel execution. A flake you cannot reproduce cannot be verified fixed.
2. Capture the failing run's details: exact assertion, values, timestamps, seed, thread, and order. Compare a passing and failing run side by side.
3. Classify the root cause. Common families:
   - Timing/async: fixed `sleep`, race between action and assertion, waiting on wall-clock.
   - Order dependence: relies on state set by another test, or on iteration order of a hash/set/map.
   - Shared mutable state: global singletons, static caches, DB rows, files, ports not reset between tests.
   - Nondeterministic inputs: unseeded random, `now()`, UUIDs, locale/timezone, floating-point equality.
   - Concurrency: real threads/goroutines with unsynchronized access.
   - External dependencies: real network, third-party services, DNS, time-sensitive fixtures.
4. Fix at the source, not the symptom. Replace sleeps with polling/awaiting an explicit condition; inject deterministic clocks/random/ID generators; reset or isolate shared state per test; sort or assert on sets rather than ordered comparisons; stub external calls.
5. Remove the crutches: delete retries, arbitrary sleeps, and order assumptions once the real cause is fixed.
6. Verify hard: run the previously-flaky test hundreds of times, in random order, and in parallel. It must be green every time.
7. If a fix is not immediate, quarantine deliberately: tag/skip the test so it stops blocking CI, file a tracked issue with the reproduction, and set a deadline — quarantine is temporary, not a graveyard.
8. Prevent recurrence: enable randomized test ordering and a fixed-but-logged seed in CI so future flakes surface early.

## Examples

Turning a timing-based flake into a deterministic wait.

```python
# Flaky: assumes the worker finished within 100ms.
def test_job_completes():
    submit(job)
    time.sleep(0.1)                    # sometimes not enough -> flaky
    assert job.status == "done"

# Stable: poll for the condition with a timeout, no wall-clock assumption.
def test_job_completes():
    submit(job)
    wait_until(lambda: job.status == "done", timeout=5)
    assert job.status == "done"
```

Killing order dependence and unstable ordering.

```python
# Flaky: dict/set iteration order and leftover global state.
assert list(get_tags()) == ["a", "b", "c"]   # order not guaranteed
# Stable:
assert set(get_tags()) == {"a", "b", "c"}

# Reproduce order-dependence:
#   pytest -p no:randomly -p randomly --randomly-seed=1234
#   run the single test 500x:  for i in $(seq 500); do pytest test_x.py -q || break; done
```

## Checklist

- [ ] The flake was reproduced reliably (loop, isolation, random order, parallel) before fixing.
- [ ] A failing vs passing run was compared and the root-cause family identified.
- [ ] The nondeterminism is removed at its source (clock/random/state injected or isolated).
- [ ] All sleeps, retries, and order assumptions added as crutches are gone.
- [ ] The test passed hundreds of consecutive runs in random order and in parallel.
- [ ] Any test that could not be fixed now is quarantined with a tracked issue and owner.
- [ ] CI runs tests in randomized order with a logged seed to catch future flakes.
