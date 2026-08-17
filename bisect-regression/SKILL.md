---
name: bisect-regression
description: Isolate the exact change that introduced a regression using binary search over history (git bisect) or over inputs/config — use when something worked before and is now broken, you have a known-good and known-bad point, or the user asks which commit caused a bug, wants to bisect, or needs to narrow a large suspect range fast.
---

# Bisect a Regression

When behavior worked at one point and is broken now, binary search finds the culprit in log-two steps instead of reading every change. `git bisect` automates this over commit history: you mark a known-good and known-bad commit, and it checks out the midpoint repeatedly until it names the first bad commit. The same halving strategy applies to inputs, config, feature flags, or data when the cause is not in code.

## When to use this skill

- A feature/test that used to pass now fails, and you have (or can find) a commit where it worked.
- The suspect range is large — many commits, a big config file, or a huge input — and reading it all is impractical.
- The user asks "which commit broke this", "when did this regress", or wants to bisect.
- Performance regressed and you need the change that introduced it.
- The regression is in data/config rather than code and you want to halve the search space.

## Instructions

1. Define a crisp, automatable test for "bad": a single command that exits 0 when good and non-zero when bad. Ambiguity here poisons the whole bisect. Make it deterministic (see the flaky-test-hunter skill if not).
2. Find bounds. Identify a known-BAD ref (usually current HEAD) and a known-GOOD ref (a commit/tag/date where it worked). If unsure of good, jump back further until the test passes.
3. Start the bisect and mark the bounds:
   ```bash
   git bisect start
   git bisect bad                 # current HEAD is broken
   git bisect good v1.4.0         # last known-good ref
   ```
4. At each checkout, run the test and mark the result: `git bisect good` or `git bisect bad`. Repeat; each answer halves the remaining range.
5. Prefer automation with `git bisect run`, which drives the whole search from your test command:
   ```bash
   git bisect run ./scripts/repro.sh
   ```
   Ensure the script's exit codes are correct (0 = good, 1–124 = bad, 125 = skip/untestable).
6. Handle untestable commits (won't build, unrelated breakage) with `git bisect skip` so they are excluded rather than mismarked.
7. Read the result: git reports the first bad commit. Inspect its diff (`git show <sha>`) to understand the mechanism — the diff points you at the change, but confirm cause via the root-cause-debugging skill.
8. Clean up: `git bisect reset` to return to your original HEAD.
9. If the cause is not in commits, bisect the other axis: halve a config file, disable half the feature flags/plugins, or shrink the input by half repeatedly until the minimal trigger remains.
10. Close the loop: fix the cause and add a regression test that would have failed at the bad commit, so the bisect never needs repeating.

## Examples

Fully automated bisect with a repro script.

```bash
# repro.sh — exit 0 if good, 1 if bad, 125 if this commit can't be tested
#!/usr/bin/env bash
set -e
make build || exit 125          # skip commits that don't build
./run-check --case regression-42 # exits non-zero when the bug is present

# Drive the search:
git bisect start
git bisect bad HEAD
git bisect good v1.4.0
git bisect run ./repro.sh
# ... git prints: "<sha> is the first bad commit"
git bisect reset
```

Bisecting non-code causes.

```text
Config bisect:   comment out half the config -> retest -> keep halving the half that reproduces.
Plugin bisect:   disable half the plugins/extensions -> retest -> narrow to one.
Input bisect:    delete half the input rows -> retest -> shrink to the minimal failing row.
Dependency:      pin to a midpoint version between last-good and first-bad release.
```

## Checklist

- [ ] "Bad" is defined as a single deterministic command with correct exit codes.
- [ ] A genuine known-good and known-bad bound were established before starting.
- [ ] Untestable commits were `skip`ped, not guessed as good/bad.
- [ ] The search was automated with `git bisect run` where possible.
- [ ] The reported first-bad commit's diff was inspected and the mechanism confirmed.
- [ ] `git bisect reset` returned the tree to its original state.
- [ ] A regression test was added that fails at the bad commit and passes after the fix.
