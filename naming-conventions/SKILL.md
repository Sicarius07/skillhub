---
name: naming-conventions
description: Choose clear, consistent, intention-revealing names for variables, functions, types, and files that follow the surrounding codebase's conventions; use when naming new identifiers, renaming unclear code, reviewing names, or establishing naming standards.
---

# Naming Conventions

Names are the primary interface between a reader and code. Good names reveal intent and let readers skip the implementation; poor names force everyone to re-read the body every time. This skill gives concrete rules for picking names and matching a codebase's existing style.

## When to use this skill

- You are introducing new variables, functions, types, modules, or files.
- The user asks to rename unclear identifiers or improve readability.
- You are reviewing code and names like `data`, `tmp`, `mgr`, or `flag2` appear.
- A team needs a consistent naming standard across a codebase.

## Instructions

1. Match the codebase first. Detect and follow existing conventions: case style (`camelCase`, `snake_case`, `PascalCase`, `kebab-case` for files), pluralization, and domain vocabulary. Consistency beats personal preference.
2. Name for intent, not implementation. `activeUsers` beats `filteredList`; `retryCount` beats `n`. The name should say what the value means, not how it was produced.
3. Make length proportional to scope. A loop index in three lines can be `i`; a module-level constant needs a full, descriptive name. Wide scope demands more specificity.
4. Use grammatical roles consistently:
   - Functions/methods: verb phrases (`fetchUser`, `isExpired`, `hasPermission`). Booleans read as predicates (`is`, `has`, `can`, `should`).
   - Variables/fields: noun phrases (`orderTotal`, `pendingJobs`).
   - Collections: plural nouns (`orders`, not `orderList`).
5. Avoid noise and misinformation. Drop redundant type suffixes (`userObject`, `dataArray`), meaningless words (`Manager`, `Helper`, `Util` used as a dumping ground), and abbreviations that aren't standard in the domain. Never let a name lie (e.g., `list` holding a set).
6. Keep units and domain terms explicit. Encode units when ambiguous (`timeoutMs`, `sizeBytes`). Use one canonical term per concept across the codebase — don't mix `customer`, `client`, and `user` for the same thing.
7. Name files/modules by their responsibility and follow the platform's file convention. One clear concept per file where practical.

## Examples

Before and after:

```diff
- function proc(d, f) {
-   const l = d.filter(x => x.a > f);
-   return l;
- }
+ function selectOrdersAbove(orders, minTotal) {
+   return orders.filter(order => order.total > minTotal);
+ }
```

Encoding units and boolean intent:

```diff
- let timeout = 30;        // 30 what?
- let done = user.state;   // not obviously boolean
+ let timeoutMs = 30_000;
+ let isOnboardingComplete = user.state === "complete";
```

## Checklist

- [ ] Names follow the codebase's existing case and file conventions.
- [ ] Each name reveals intent, not implementation detail.
- [ ] Name specificity is proportional to scope.
- [ ] Functions use verbs; booleans use is/has/can/should; collections are plural.
- [ ] No redundant type suffixes, filler words, or misleading names.
- [ ] Units are explicit where ambiguous; one canonical term per concept.
- [ ] Renames were applied consistently across all references.
