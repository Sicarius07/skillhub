---
name: resolve-merge-conflicts
description: Resolve git merge or rebase conflicts safely and correctly by understanding both sides, using conflict markers and diff tools, verifying with tests, and aborting cleanly when needed — use when a merge, rebase, cherry-pick, or stash pop reports conflicts, or when a user asks how to handle "<<<<<<< HEAD" markers.
---

# Resolve Merge Conflicts

Merge conflicts happen when two branches change the same lines (or one edits what another deletes) and git cannot auto-combine them. Resolving them well means understanding the *intent* of both sides — not just picking one blindly — then verifying the result builds and behaves correctly.

## When to use this skill

- `git merge`, `git rebase`, `git cherry-pick`, or `git stash pop` reports conflicts.
- You see `<<<<<<<`, `=======`, `>>>>>>>` markers in files.
- A pull request shows "This branch has conflicts that must be resolved."
- You need to decide whether to keep yours, keep theirs, or combine both.

## Instructions

1. **See the scope.** Run `git status` to list conflicted files and `git diff` to view the conflicting hunks. Understand what you were doing (merge vs rebase) — this affects which side is "ours".
2. **Know which side is which.** In a merge, `<<<<<<< HEAD` is your current branch and `>>>>>>>` is the incoming branch. In a **rebase these are swapped** (HEAD is the branch you are rebasing *onto*), so read carefully.
3. **Understand both changes** before editing. Use `git log --merge -p <file>` to see the commits that touched the conflicting region on each side. Ask *why* each change was made.
4. **Edit to the correct combined intent.** Remove all `<<<<<<<`, `=======`, `>>>>>>>` markers and write code that honors both sides' goals. Do not reflexively "accept ours/theirs" unless one side is genuinely obsolete.
5. **Use tooling** for complex conflicts: `git mergetool`, an IDE's three-way merge view, or enable `git config merge.conflictstyle zdiff3` to also show the common ancestor for context.
6. **Stage resolved files** with `git add <file>`. Repeat until `git status` shows no unmerged paths.
7. **Verify before continuing:** build the project and run the relevant tests. A conflict resolution that removes markers but breaks logic is worse than the conflict.
8. **Continue the operation:** `git merge --continue` / `git rebase --continue` / `git cherry-pick --continue`.
9. **Abort if stuck:** `git merge --abort` or `git rebase --abort` returns you to the pre-operation state with no harm done. Prefer this over guessing.

## Examples

A conflict hunk before resolution:

```
<<<<<<< HEAD
const timeout = 30_000; // ours: bumped for slow CI
=======
const timeout = 5_000;  // theirs: lowered for faster local runs
>>>>>>> feature/fast-tests
```

Resolved to honor both intents (configurable, defaulting to the CI-safe value):

```
const timeout = Number(process.env.TEST_TIMEOUT ?? 30_000);
```

Typical flow:

```
git rebase origin/main
# ... conflicts reported ...
git status                     # list conflicted files
git mergetool                  # or edit by hand
git add src/config.ts
npm test                       # verify
git rebase --continue
```

## Checklist

- [ ] Every conflict marker (`<<<<<<<`, `=======`, `>>>>>>>`) is removed.
- [ ] The resolution reflects the intent of *both* sides, not a blind pick.
- [ ] Merge vs rebase side-mapping was accounted for.
- [ ] Project builds and relevant tests pass.
- [ ] All previously conflicted files are staged.
- [ ] The merge/rebase/cherry-pick was continued (or cleanly aborted).
