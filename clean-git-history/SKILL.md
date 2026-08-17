---
name: clean-git-history
description: Craft a readable, linear git history using atomic commits, interactive rebase, squashing, fixups, and reordering before merging — use when preparing a branch for review or merge, cleaning up messy "WIP" commits, splitting or reordering commits, or when a user asks how to tidy history without losing work.
---

# Clean Git History

A clean history tells the story of *why* the code changed, one reviewable step at a time. This skill covers shaping commits before they land: making them atomic, squashing noise, reordering for logical flow, and rebasing onto the latest base branch — all while protecting your work from loss.

## When to use this skill

- A feature branch has accumulated "wip", "fix typo", or "address review" commits.
- You are about to open or merge a pull request and want a clean, bisectable log.
- You need to split one large commit into several, or combine several into one.
- You want a linear history (rebase) instead of merge bubbles.
- A user asks how to reorder, squash, edit, or drop commits safely.

## Instructions

1. **Back up first.** Before rewriting, note the current tip: `git branch backup/my-feature` or record `git rev-parse HEAD`. You can always return via `git reflog`.
2. **Never rewrite shared history.** Only rebase/squash commits that have not been merged or that others are not building on. If in doubt, ask.
3. **Update your base.** Rebase onto the latest base branch to keep history linear: `git fetch origin` then `git rebase origin/main`. Resolve conflicts commit by commit.
4. **Interactive rebase** to reshape commits: `git rebase -i origin/main`. In the todo list, use:
   - `pick` — keep the commit.
   - `reword` (`r`) — keep changes, edit the message.
   - `edit` (`e`) — pause to amend content or split.
   - `squash` (`s`) — combine into the previous commit, merging messages.
   - `fixup` (`f`) — combine into the previous commit, discarding this message.
   - `drop` (`d`) — remove the commit entirely.
   - Reorder lines to reorder commits.
5. **Use fixup commits during development** so squashing is automatic later: `git commit --fixup=<sha>`, then `git rebase -i --autosquash origin/main` positions them correctly.
6. **Split a commit:** in an `edit` stop, run `git reset HEAD^`, then stage and commit hunks separately (`git add -p`).
7. **Aim for atomic commits:** each compiles, passes tests, and represents one logical change. Keep refactors separate from behavior changes.
8. **Push a rewritten branch** with `git push --force-with-lease` (safer than `--force`; it refuses if the remote moved unexpectedly).

## Examples

Squash three WIP commits into one clean commit and reword it:

```
git rebase -i origin/main
# editor:
pick   a1b2c3  feat(profile): add avatar upload
fixup  d4e5f6  wip
fixup  g7h8i9  fix lint
reword j0k1l2  feat(profile): validate image size
```

Autosquash workflow during development:

```
git commit --fixup=a1b2c3      # marks a fixup for an earlier commit
git rebase -i --autosquash origin/main
git push --force-with-lease
```

Recover if a rebase goes wrong:

```
git reflog                     # find the pre-rebase SHA
git reset --hard HEAD@{5}      # restore that state
```

## Checklist

- [ ] A backup branch or recorded SHA exists before rewriting.
- [ ] Only unshared/unmerged commits were rewritten.
- [ ] Branch is rebased onto the current base.
- [ ] Each commit is atomic and has a meaningful message.
- [ ] No leftover "wip"/"fixup" commits remain.
- [ ] Pushed with `--force-with-lease`, and CI still passes.
