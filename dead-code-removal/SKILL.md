---
name: dead-code-removal
description: Safely find and delete code that is never reached or referenced — unused functions, imports, files, flags, and branches — with evidence and reversibility; use when asked to remove dead code, clean up unused code, prune stale features, or reduce codebase clutter.
---

# Dead Code Removal

Dead code is code that can never execute or is never referenced: orphaned functions, unreachable branches, unused imports and exports, retired feature flags, and abandoned files. This skill removes it safely, requiring evidence that code is truly unused before deletion and keeping every removal easy to revert.

## When to use this skill

- The user asks to remove dead, unused, orphaned, or stale code.
- You are cleaning up after a feature was retired or a flag was fully rolled out.
- Static analysis, coverage, or linters flag unreachable or unreferenced symbols.
- The codebase has accumulated commented-out blocks or "TODO: delete" files.

## Instructions

1. Gather evidence, do not guess. For each candidate, confirm it is unused:
   - Search the whole repo for references, including tests, config, string-based lookups, reflection/dynamic dispatch, and DI registrations.
   - Use language tooling where available (dead-code linters, unused-export detectors, coverage reports from a full test/e2e run).
2. Beware of hidden entry points. Public API/exported library symbols, framework callbacks, serialized class names, CLI commands, cron jobs, and reflection targets may have no in-repo caller yet still be used. When in doubt, keep or deprecate rather than delete.
3. Check dynamic references explicitly. Grep for the symbol name as a string, not just as an identifier, to catch reflection, event names, template bindings, and config keys.
4. Delete in small, isolated commits. Group by feature or module so each removal can be reverted independently. Do not bundle dead-code deletion with behavior changes.
5. Remove the whole chain. When you delete a function, also remove now-orphaned imports, tests, fixtures, translations, and config it depended on. Re-run the unused-symbol check to catch newly-orphaned code.
6. Verify the build and tests. Compile, run the full suite, and run any type checker. A clean build after removal is your primary safety signal.
7. Prefer deprecation for uncertain public surface. Mark it deprecated with a removal date rather than deleting immediately when external consumers may exist.

## Examples

Confirming a function is unused before deleting:

```bash
# Identifier references (excluding its own definition file):
grep -rn "calculateLegacyTax" src/ test/ config/
# String-based / dynamic references:
grep -rn "calculateLegacyTax" . --include="*.json" --include="*.yaml"
# If both return only the definition, it is a safe candidate.
```

Removing an unreachable branch left after a flag rollout:

```diff
- if (features.newCheckout) {
-   return renderNewCheckout();
- } else {
-   return renderOldCheckout();  // flag has been 100% on for 6 months
- }
+ return renderNewCheckout();
// also delete renderOldCheckout() and its tests once no references remain
```

## Checklist

- [ ] Each candidate has evidence of being unused (identifier + string search + tooling).
- [ ] Hidden entry points (public API, reflection, framework hooks) are ruled out.
- [ ] Orphaned imports, tests, fixtures, and config are removed with the code.
- [ ] A second unused-symbol pass found no newly-orphaned code.
- [ ] Removals are in small, independently revertible commits.
- [ ] Build, type check, and full test suite pass after removal.
- [ ] Uncertain public surface is deprecated rather than hard-deleted.
