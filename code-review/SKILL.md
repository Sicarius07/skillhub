---
name: code-review
description: Systematically review a diff or pull request for correctness, clarity, security, and risk before it merges; use when asked to review a PR, review changes, give feedback on a diff, or assess whether code is ready to land.
---

# Code Review

This skill guides a thorough, prioritized review of a code change. The goal is to catch defects and risky decisions early, confirm the change does what it claims, and leave feedback that is specific, actionable, and ranked by importance rather than nitpicky and unordered.

## When to use this skill

- The user asks you to review a pull request, diff, patch, or set of staged changes.
- You are asked whether a change is "ready to merge" or "safe to land."
- Someone wants feedback on correctness, security, performance, or readability of code they wrote.
- You are gating a change before deploy and need a risk assessment.

## Instructions

1. Establish intent first. Read the PR title, description, and linked issue. State in one sentence what the change is supposed to do; if you cannot, ask or infer it from the diff before reviewing.
2. Get the full diff, not just snippets. Prefer `git diff <base>...<head>` (three-dot) so you see exactly what the branch adds relative to its merge base. Note the size and which files carry the most churn.
3. Review in passes, highest risk first:
   - **Correctness**: Does the logic match the stated intent? Check edge cases, off-by-one, null/empty inputs, boundary conditions, and error paths.
   - **Security**: Look for injection, unvalidated input, secrets in code, broken authz/authn, unsafe deserialization, and logging of sensitive data.
   - **Concurrency & data**: Race conditions, non-atomic read-modify-write, unbounded growth, migrations that lock or drop data.
   - **API & compatibility**: Breaking changes to public interfaces, wire formats, or DB schema without a migration path.
   - **Tests**: Are new behaviors and bug fixes covered? Do tests actually assert the fix, or just run the code?
   - **Clarity**: Naming, dead branches, misleading comments, needless complexity.
4. Distinguish blocking issues from suggestions. Label each comment: `blocking`, `should-fix`, or `nit`. Do not bury a data-loss bug among style comments.
5. Verify claims you can cheaply verify. Run the tests or a linter if available; trace one representative call path end to end.
6. Summarize: give a clear verdict (approve / approve-with-changes / request-changes), the top 3 concerns, and anything you could not assess.

## Examples

A vague comment versus a useful one:

```diff
- // "this looks wrong"
+ // blocking: `for (i=0; i<=items.length; i++)` reads items[length] on the
+ // last iteration → undefined. Should be `i < items.length`. Add a test with
+ // a 2-element array to lock this in.
```

Structured summary output:

```md
Verdict: request-changes

Blocking
1. Off-by-one in paginate() drops the last page (utils/page.ts:42).
2. User email is logged at INFO in auth handler (auth.ts:88) — PII leak.

Should-fix
- No test covers the empty-cart path added in checkout.ts.

Nits
- `tmp2` → `pendingCharges` in billing.ts:20.

Not assessed: performance of the new N+1 query under load.
```

## Checklist

- [ ] I can state the change's intent in one sentence.
- [ ] I reviewed the complete diff against the correct merge base.
- [ ] Correctness, security, and data-safety passes are done.
- [ ] Each comment is labeled blocking / should-fix / nit.
- [ ] New behavior and bug fixes have real test coverage.
- [ ] I gave a clear verdict and listed the top concerns.
- [ ] I noted anything I could not verify.

## Notes
- Prefer small, focused diffs when reviewing.
