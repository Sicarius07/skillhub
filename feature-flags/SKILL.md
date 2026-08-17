---
name: feature-flags
description: Introduce and manage feature flags safely for gradual rollouts, kill switches, A/B tests, and trunk-based development — covering flag naming, evaluation, default-off safety, targeting, cleanup of stale flags, and testing both states; use when hiding unfinished work behind a toggle, rolling out a risky change progressively, decoupling deploy from release, or when users mention feature flags, toggles, rollout, kill switch, or experiments.
---

# Feature Flags

This skill covers using feature flags (feature toggles) to decouple deployment from release, roll changes out gradually, and disable features instantly when something goes wrong. It emphasizes safe defaults, clear ownership, and disciplined cleanup so flags do not become permanent technical debt.

## When to use this skill

- Merging unfinished work to the main branch behind a toggle (trunk-based development).
- Rolling out a risky feature progressively (percentage or cohort ramp).
- Needing an instant kill switch for a feature in production.
- Running A/B tests or experiments.
- Users mention feature flags, toggles, rollout, canary, kill switch, or experiments.

## Instructions

1. **Classify the flag by purpose.** Decide whether it is a release toggle (temporary, ship-behind), an ops/kill switch (long-lived, operational), an experiment (A/B, time-boxed), or a permission/entitlement flag. Purpose drives lifespan and cleanup expectations.
2. **Name flags clearly and consistently.** Use a descriptive, prefixed convention (e.g., `release_new_checkout`, `ops_disable_search`) so intent and type are obvious. Record owner, creation date, and intended removal date.
3. **Default to off / safe.** New behavior is disabled by default; the fallback path must be the current, known-good behavior. If flag evaluation fails, fall back to the safe default rather than erroring.
4. **Centralize evaluation.** Evaluate flags through a single service/wrapper so targeting, logging, and defaults are consistent. Avoid scattering ad-hoc environment checks.
5. **Keep the flagged surface small.** Branch at a single well-defined point (a function, route, or component boundary), not sprinkled across many files, so removal is easy later.
6. **Roll out gradually.** Enable for internal users, then a small percentage, then ramp up while watching metrics and error rates. Keep the kill switch reachable at every step.
7. **Support targeting when needed.** Allow enabling by environment, user segment, tenant, or percentage bucket using a stable hash of a user/tenant id for consistent assignment.
8. **Test both states.** Write/verify tests with the flag on and off so neither path rots. In CI, exercise the default and, where feasible, the enabled path.
9. **Instrument and monitor.** Log flag exposure and tie flag state to dashboards/alerts so you can correlate a rollout with regressions.
10. **Clean up stale flags.** Once a release flag is fully rolled out (or an experiment concludes), remove the flag and the dead branch promptly. Track flags in a registry and review regularly.

## Examples

Safe evaluation wrapper with a default-off fallback:

```ts
function isEnabled(flag: string, ctx: Context): boolean {
  try {
    return flagClient.evaluate(flag, ctx, /* default */ false);
  } catch {
    return false; // never break the request on flag errors
  }
}

if (isEnabled('release_new_checkout', { userId: user.id })) {
  return renderNewCheckout();
}
return renderCheckout(); // known-good fallback
```

Deterministic percentage rollout via stable hashing:

```ts
function inRollout(userId: string, percent: number): boolean {
  const bucket = hashToInt(userId) % 100; // stable per user
  return bucket < percent;
}
```

Flag metadata for the registry:

```yaml
release_new_checkout:
  type: release
  owner: payments-team
  created: 2026-08-01
  remove_by: 2026-10-01
  description: New checkout flow; remove after 100% rollout.
```

## Checklist

- [ ] Flag classified by type with owner and removal date recorded.
- [ ] Clear, prefixed, consistent name.
- [ ] Defaults to off; failure falls back to safe/current behavior.
- [ ] Evaluation goes through a centralized wrapper/service.
- [ ] Branching confined to a small, well-defined surface.
- [ ] Rollout is gradual with a reachable kill switch.
- [ ] Targeting uses stable hashing for consistent assignment.
- [ ] Both on and off states are tested.
- [ ] Flag exposure logged and monitored.
- [ ] Stale flags scheduled for cleanup and removed after rollout.
