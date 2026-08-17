---
name: refactor-safely
description: Restructure code without changing its observable behavior, using tests and small reversible steps as a safety net; use when asked to refactor, restructure, extract, rename across a codebase, or clean up code while preserving functionality.
---

# Refactor Safely

Refactoring means improving the internal structure of code while keeping its external behavior identical. This skill enforces a discipline of small, verifiable steps guarded by tests so that a refactor never silently becomes a behavior change or a regression.

## When to use this skill

- The user asks to refactor, restructure, extract a function/module, or reorganize code.
- You need to change internal design (patterns, layering, naming) without altering outputs.
- You are preparing messy code for a feature and want to separate the cleanup from the feature work.
- A large rename or move needs to happen across many files.

## Instructions

1. Confirm the boundary of "behavior." Identify the public surface that must stay constant: return values, side effects, emitted events, logs relied upon, timing guarantees, and error types. Write this down.
2. Establish a safety net before touching anything. Run the existing test suite and record the result. If coverage is thin around the code you will change, add characterization tests that pin current behavior first — including current quirks.
3. Refactor in small, individually reversible steps. Each step should compile and pass tests. Never mix a behavior change into a refactor commit; if you must change behavior, do it in a separate, clearly labeled step.
4. Prefer mechanical, tool-assisted transforms (IDE rename, extract method, automated codemods) over hand edits for renames and signature changes — they are less error-prone.
5. Re-run the full test suite after each meaningful step, not just at the end. Keep the diff green.
6. Keep the diff reviewable. Separate pure moves/renames (which produce large but trivial diffs) from logic-preserving restructures so a reviewer can verify "moved, unchanged."
7. Confirm no behavior drift at the end: same tests pass, same public API, and any performance-sensitive path is still within tolerance.

## Examples

Extracting a function without changing behavior:

```diff
- function total(cart) {
-   let sum = 0;
-   for (const i of cart.items) sum += i.price * i.qty;
-   return sum * (1 - cart.discount);
- }
+ function subtotal(items) {
+   let sum = 0;
+   for (const i of items) sum += i.price * i.qty;
+   return sum;
+ }
+ function total(cart) {
+   return subtotal(cart.items) * (1 - cart.discount);
+ }
```

A characterization test written before refactoring legacy code:

```js
// Pins CURRENT behavior, including the rounding quirk, so the refactor can't drift.
test("total applies discount after summing, truncating to cents", () => {
  const cart = { items: [{ price: 9.99, qty: 3 }], discount: 0.1 };
  expect(total(cart)).toBeCloseTo(26.973, 3);
});
```

## Checklist

- [ ] I documented what "behavior" must remain constant.
- [ ] Tests pass before I start; gaps are covered by characterization tests.
- [ ] Each step compiles and keeps the suite green.
- [ ] No behavior change is hidden inside a refactor step.
- [ ] Renames/moves are separated from logic restructures in the diff.
- [ ] Public API, side effects, and error types are unchanged.
- [ ] Full suite passes at the end and performance is within tolerance.
