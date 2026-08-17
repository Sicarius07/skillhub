---
name: root-cause-debugging
description: Debug systematically with a hypothesis-driven method that finds the true cause instead of patching symptoms — use when facing a confusing bug, an intermittent or hard-to-reproduce failure, a production incident, or when the user asks why something is broken, wants a real root cause, or keeps applying fixes that do not stick.
---

# Root-Cause Debugging

Symptom-patching makes a bug disappear from view without removing its cause, so it returns in a new disguise. Root-cause debugging is a disciplined loop: reproduce reliably, form a specific falsifiable hypothesis, run the cheapest experiment that could disprove it, and follow the evidence to the actual defect. You change one variable at a time and let observations — not guesses — drive each step.

## When to use this skill

- A bug is confusing, intermittent, or resists the obvious fix.
- A production incident needs a real cause, not just mitigation.
- Previous fixes did not stick, or the same class of bug keeps returning.
- The user asks "why is this happening", "what's the root cause", or is stuck guessing.
- Behavior differs between environments (works locally, fails in prod) and you need to know why.

## Instructions

1. Nail down the symptom precisely: exact error, inputs, environment, frequency, and when it started. Vague reports ("it's slow") must be made concrete before proceeding.
2. Reproduce reliably. Find the smallest, fastest reproduction you can — a failing test is ideal. If it only happens sometimes, hunt for the hidden variable (data, timing, order, load, config) that flips it.
3. Establish what changed. Correlate the onset with deploys, config, data, dependency, or traffic changes. Recent change is the highest-yield suspect (see the bisect-regression skill to isolate a commit).
4. Form ONE specific hypothesis, phrased so it can be proven false: "the cache returns a stale value because the key omits the tenant id" — not "something's wrong with caching".
5. Design the cheapest experiment that discriminates between "hypothesis true" and "false": add a targeted log/breakpoint, inspect a value, disable one component, or diff good vs. bad inputs. Change one variable at a time.
6. Observe and update. If the experiment disproves the hypothesis, that is progress — discard it and form the next. Keep a short written log of hypotheses tried and what each ruled out, so you do not loop.
7. Bisect the search space. Confirm where the data is still correct and where it is first wrong; the cause lives between those two points. Halve the space each step (in code path, time, or data).
8. Reach the true cause: the point where correct input first becomes incorrect output, and you can explain the mechanism. Prove it by toggling it on and off — the bug should follow.
9. Fix the cause, then add a regression test that fails without the fix. Ask whether the same root cause exists elsewhere.
10. Do a brief retrospective for significant bugs: why it happened, why it was not caught, and what guardrail prevents the class.

## Examples

A hypothesis log that converges on the cause.

```text
Symptom: ~1% of checkout totals are off by a few cents, only for multi-item carts.

H1: floating-point rounding in line-item sum.
  Experiment: log raw item prices + running total for a failing order.
  Observation: totals correct until the discount step. -> H1 FALSE, but narrowed.

H2: discount applied per-item then re-summed, rounding each time.
  Experiment: compute discount on the pre-rounded subtotal instead.
  Observation: discrepancy disappears for the repro order. -> H2 SUPPORTED.

Proof: toggle rounding location on/off -> bug follows the toggle.
Cause: rounding each discounted line item accumulates error.
Fix: round once at the total; add regression test with the failing cart.
```

Bisecting where correct data first becomes wrong.

```text
input OK ----> parse OK ----> transform OK ----> serialize BAD ----> output BAD
                                          ^ correct here     ^ wrong here
=> the defect is inside serialize; instrument its input vs output next.
```

## Checklist

- [ ] Symptom is concrete: exact error, inputs, environment, frequency, first-seen.
- [ ] There is a reliable (ideally minimal/automated) reproduction.
- [ ] Recent changes were checked as the leading suspect.
- [ ] Each step tested ONE falsifiable hypothesis with the cheapest discriminating experiment.
- [ ] One variable changed at a time, with a log of what each experiment ruled out.
- [ ] The true cause is identified and proven by toggling it on/off.
- [ ] A regression test fails without the fix and passes with it.
- [ ] Checked whether the same root cause exists elsewhere; noted a preventing guardrail.
