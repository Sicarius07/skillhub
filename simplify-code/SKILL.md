---
name: simplify-code
description: Reduce accidental complexity by removing needless abstraction, flattening control flow, and deleting speculative generality while preserving behavior; use when asked to simplify, clean up, make code more readable, reduce nesting, or cut over-engineering.
---

# Simplify Code

This skill targets accidental complexity — the incidental difficulty we add ourselves through over-abstraction, deep nesting, premature generalization, and indirection that hides intent. The aim is code that a new reader understands quickly, without changing what it does.

## When to use this skill

- The user asks to simplify, streamline, or make code easier to read.
- Code has deep nesting, long functions, or many layers of indirection for a single caller.
- You spot speculative "flexibility" (config, plugins, base classes) with only one concrete use.
- A function is hard to name because it does several unrelated things.

## Instructions

1. Understand before cutting. Read the code and its tests; confirm what behavior must be preserved. Simplification is behavior-preserving — treat it like a refactor and lean on tests.
2. Attack nesting with guard clauses. Return/throw early for error and edge cases so the happy path is flat and left-aligned.
3. Collapse needless indirection. Inline single-use helpers, wrappers, or interfaces that have exactly one implementation and no test seam justifying them. Remove pass-through layers that only forward arguments.
4. Delete speculative generality. Remove parameters, hooks, and extension points that no caller uses. Choose the concrete solution over the "future-proof" one until a second real use appears.
5. Prefer clear data flow over clever expressions. Replace dense ternary chains and nested comprehensions with named intermediate values. Optimize for reading, not character count.
6. Reduce state and mutation. Prefer pure functions and immutable values where practical; shrink variable scope so each name has one meaning.
7. Verify equivalence. Run tests after each change; keep changes small so a reviewer sees that simplification did not alter behavior.

## Examples

Guard clauses replacing nested conditionals:

```diff
- function handle(req) {
-   if (req.user) {
-     if (req.user.active) {
-       return process(req);
-     } else { return deny("inactive"); }
-   } else { return deny("no user"); }
- }
+ function handle(req) {
+   if (!req.user) return deny("no user");
+   if (!req.user.active) return deny("inactive");
+   return process(req);
+ }
```

Removing speculative generality:

```diff
- // A "strategy" interface with exactly one implementation, ever.
- class Formatter { format(x) { throw new Error("abstract"); } }
- class JsonFormatter extends Formatter { format(x) { return JSON.stringify(x); } }
- const f = new JsonFormatter();
- send(f.format(payload));
+ send(JSON.stringify(payload));
```

## Checklist

- [ ] Behavior is preserved and tests still pass.
- [ ] The happy path is flat; error/edge cases use guard clauses.
- [ ] Single-use wrappers and one-implementation interfaces are inlined or removed.
- [ ] Unused parameters, hooks, and extension points are gone.
- [ ] Dense expressions are replaced with clearly named values.
- [ ] Variable scope and mutation are minimized.
- [ ] The diff is small enough that the simplification is self-evidently behavior-preserving.
