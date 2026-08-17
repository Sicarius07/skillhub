---
name: design-patterns
description: Choose and apply the right software design pattern for a problem — and just as importantly, recognize when no pattern is needed — covering creational, structural, and behavioral patterns and their trade-offs; use when a design feels rigid or duplicated, when refactoring toward flexibility, during design reviews, or when tempted to add abstraction that may be premature.
---

# Design Patterns

Design patterns are named, reusable solutions to recurring design problems. Their value is in shared vocabulary and proven structure — but misapplied, they add indirection and cost without benefit. This skill helps you diagnose the underlying problem, match it to a pattern (or decide none is warranted), and apply the minimum structure needed.

## When to use this skill

- A class or module is hard to change, test, or extend.
- You see the same conditional or construction logic duplicated.
- Requirements point to variation along a clear axis (behavior, creation, or structure).
- Reviewing a design and judging whether an abstraction earns its keep.
- You feel the urge to add a pattern "to be safe" and want a sanity check.

## Instructions

1. **Name the problem first.** Describe the actual pain (rigidity, duplication, tight coupling, hard-to-test) before reaching for a pattern name.
2. **Check if a pattern is even needed.** If the code has one implementation and no near-term variation, a simple function or class is better than a pattern. Prefer YAGNI.
3. **Identify the axis of change.** Object creation varies → creational (Factory, Builder, Abstract Factory). Composition/interface varies → structural (Adapter, Decorator, Facade, Composite). Behavior/algorithm varies → behavioral (Strategy, Observer, State, Command, Template Method).
4. **Prefer composition over inheritance.** Patterns like Strategy and Decorator let behavior vary without deep class hierarchies.
5. **Program to interfaces.** Depend on abstractions so implementations can be swapped and mocked.
6. **Apply the smallest version.** Introduce only the participants the current problem requires; don't build the full textbook diagram speculatively.
7. **Keep names honest.** If you use a pattern, name the class after its role (`PaymentStrategy`, `OrderRepository`) so readers recognize the intent.
8. **Watch for anti-patterns.** Singletons hiding global state, factories that only wrap `new`, and deep decorator chains often signal over-engineering.
9. **Re-evaluate after the change.** Confirm the pattern actually reduced coupling or duplication; if not, remove it.

## Examples

Replacing a branching conditional with the Strategy pattern:

```python
# Before: rigid, must edit this function for every new method
def fee(method, amount):
    if method == "card":   return amount * 0.029 + 0.30
    elif method == "ach":  return 0.25
    elif method == "wire": return 15.00

# After: open for extension via composition
class FeeStrategy:
    def compute(self, amount): raise NotImplementedError

class CardFee(FeeStrategy):
    def compute(self, amount): return amount * 0.029 + 0.30

class AchFee(FeeStrategy):
    def compute(self, amount): return 0.25

def fee(strategy: FeeStrategy, amount): 
    return strategy.compute(amount)
```

When NOT to use a pattern:

```
Problem: "We might need multiple database backends someday."
Reality: There is one database and no concrete second one planned.
Decision: Skip the Abstract Factory. Use the concrete client directly and
          extract an interface when the second backend actually appears.
```

## Checklist

- [ ] The real problem is stated in plain terms before naming a pattern.
- [ ] There is present or clearly imminent variation to justify the abstraction.
- [ ] The chosen pattern matches the axis of change (creational/structural/behavioral).
- [ ] Composition and interfaces are preferred over deep inheritance.
- [ ] Only the necessary participants were introduced (no speculative structure).
- [ ] Class/role names communicate the pattern's intent.
- [ ] No anti-pattern smell (global singletons, trivial factories, decorator soup).
- [ ] The change measurably reduced coupling or duplication.
