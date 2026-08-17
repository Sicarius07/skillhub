---
name: conventional-commits
description: Write commit messages that follow the Conventional Commits specification (type(scope): subject, with body and footers) — use when creating a git commit, drafting a commit message, standardizing commit style across a team, enabling automated semantic versioning or changelog generation, or when a user asks how to phrase or format a commit.
---

# Conventional Commits

Conventional Commits is a lightweight convention for commit messages that makes history machine-readable. A structured prefix (`feat`, `fix`, etc.) plus an optional scope and a `!`/`BREAKING CHANGE` marker lets tooling derive semantic version bumps and generate changelogs automatically, while keeping the log easy for humans to scan.

## When to use this skill

- You are about to write or amend a git commit message.
- A team wants a consistent, reviewable commit style.
- You need automated release notes, changelogs, or semantic version bumps (major/minor/patch).
- You are configuring commit linting (e.g. commitlint) or a release tool (e.g. semantic-release).
- Someone asks "how should I word this commit?" or "what type is this change?"

## Instructions

1. Determine the commit **type** from the primary intent of the change:
   - `feat` — a new user-facing feature (triggers a MINOR bump).
   - `fix` — a bug fix (triggers a PATCH bump).
   - `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert` — non-feature/non-fix changes.
2. Optionally add a **scope** in parentheses naming the area affected: `feat(auth):`, `fix(parser):`. Keep scopes short and consistent across the repo.
3. Write the **subject** after `: ` — imperative mood, lower case, no trailing period, ideally under 50 characters. Say what the commit does ("add", "remove", "fix"), not what you did ("added").
4. If the change is breaking, either append `!` before the colon (`feat(api)!:`) and/or add a `BREAKING CHANGE:` footer describing the break and migration path. Either triggers a MAJOR bump.
5. Add a **body** (separated by a blank line) when the *why* is not obvious: motivation, contrast with previous behavior, and any trade-offs. Wrap at ~72 characters.
6. Add **footers** for metadata: issue references (`Closes #123`, `Refs #456`), co-authors, or `BREAKING CHANGE:` details.
7. Keep each commit **atomic** — one logical change per commit so the type is unambiguous.

## Examples

Simple fix:

```
fix(cart): prevent negative quantities at checkout
```

Feature with body and issue footer:

```
feat(search): add fuzzy matching for product names

Exact-match search missed common typos, so users failed to find
products that clearly existed. Uses a trigram index to rank
approximate matches while keeping exact matches first.

Closes #482
```

Breaking change:

```
refactor(api)!: replace positional args with an options object

BREAKING CHANGE: `createUser(name, email)` is now
`createUser({ name, email })`. Update all call sites accordingly.
```

## Checklist

- [ ] Subject starts with a valid type and (if useful) a scope.
- [ ] Subject is imperative, lower case, no trailing period, ~50 chars or fewer.
- [ ] Breaking changes are marked with `!` and/or a `BREAKING CHANGE:` footer.
- [ ] Body explains *why* when the change is non-trivial.
- [ ] Issues/co-authors referenced in footers.
- [ ] The commit contains exactly one logical change.
