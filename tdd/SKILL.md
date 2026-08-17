---
name: tdd
description: Drive implementation with test-driven development using the red/green/refactor loop — use when starting a new feature, fixing a bug you can reproduce, or when the user asks to "write the test first", practice TDD, or wants a safety net of tests grown alongside the code.
---

# Test-Driven Development (Red / Green / Refactor)

Test-driven development inverts the usual order of work: you write a failing test that specifies the behavior you want, make it pass with the simplest code possible, then clean up. Working in tiny verified increments keeps you focused on one behavior at a time, produces a regression suite as a byproduct, and pressures you toward simpler, more testable designs.

## When to use this skill

- Starting a new function, class, or module where the desired behavior is clear enough to express as an assertion.
- Fixing a bug that you can reproduce — write the failing test that captures the bug first.
- Refactoring risky code and you want a safety net before you touch it.
- The user explicitly asks for TDD, "test first", or "red-green-refactor".
- You want the design of an API to be shaped by how it is actually called.

## Instructions

1. Pick the smallest next behavior. Do not try to specify the whole feature at once. Choose one concrete, observable outcome.
2. RED — write a single failing test that asserts that behavior. Name it after the behavior (e.g. `returns_zero_for_empty_cart`), not the method. Assert on the outcome, not the implementation.
3. Run the test and watch it fail. Confirm it fails for the expected reason (assertion failure), not a typo or missing import. A test you never saw fail proves nothing.
4. GREEN — write the minimum code to make the test pass. Hardcoding or an obvious naive implementation is acceptable here; you are proving the test can pass. Do not add behavior no test demands.
5. Run the full suite. Everything should be green. If an unrelated test broke, address it before moving on.
6. REFACTOR — with tests green, improve names, remove duplication, and clarify structure in both production and test code. Re-run tests after each change. Change structure OR behavior, never both at once.
7. Repeat from step 1 for the next behavior. Let the tests accumulate into the full specification.
8. When done, review the test names top to bottom — they should read as a readable description of what the unit does.

## Examples

A worked loop for a `Stack` that must raise on `pop` when empty.

```text
RED: write the test
  test "pop on empty stack raises Underflow":
    s = Stack()
    expect_raises(Underflow) { s.pop() }
  run -> FAILS (Underflow never raised / no pop method)

GREEN: minimum code
  class Stack:
    def pop(self):
      raise Underflow   # simplest thing that passes THIS test
  run -> PASSES

Next RED: pop returns the last pushed item
  test "pop returns last pushed":
    s = Stack(); s.push(1); s.push(2)
    assert s.pop() == 2
  run -> FAILS

GREEN: now a real implementation is the simplest path
  class Stack:
    def __init__(self): self._items = []
    def push(self, x): self._items.append(x)
    def pop(self):
      if not self._items: raise Underflow
      return self._items.pop()
  run -> PASSES (both tests)

REFACTOR: rename _items -> _elements, extract emptiness check; re-run -> still green
```

Guard against false confidence: temporarily break the production code and confirm the relevant test goes red, then restore it.

## Checklist

- [ ] Each new behavior started as a failing test that I watched fail for the right reason.
- [ ] Production code was only written to satisfy a failing test.
- [ ] The full suite is green after every green/refactor step.
- [ ] Refactoring changed structure only, with no behavior change and no red tests.
- [ ] Test names describe behaviors and read as a specification.
- [ ] No untested branches were added "while I was in there".

## Tip
- Write the failing test first, then the minimal code to pass.
