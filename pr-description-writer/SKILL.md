---
name: pr-description-writer
description: Write clear, reviewable pull request descriptions that explain what changed, why, how to test, and any risks — use when opening a PR, filling in a PR template, summarizing a branch's changes for reviewers, or when a user asks for help describing or documenting a pull request or merge request.
---

# PR Description Writer

A good pull request description gives reviewers the context they need to review quickly and confidently: what problem it solves, what approach was taken, how to verify it, and what to watch out for. It reduces back-and-forth and becomes durable documentation of the change.

## When to use this skill

- You are opening a pull/merge request or filling in a PR template.
- A branch is ready for review and needs a summary for reviewers.
- A reviewer asks "what does this change and why?"
- You want to standardize PR descriptions across a team.

## Instructions

1. **Write a precise title.** Summarize the change in one line, present tense. If the team uses Conventional Commits, mirror that: `feat(auth): add SSO login`.
2. **Lead with the why.** Open with a short paragraph: the problem, bug, or goal this PR addresses. Link the issue/ticket (`Closes #123`).
3. **Summarize what changed.** Bullet the key changes at a conceptual level — not a file-by-file dump (the diff already shows files). Group related changes.
4. **Explain notable decisions.** Call out design choices, trade-offs, alternatives considered, and anything non-obvious a reviewer might question.
5. **Give test instructions.** State how you verified it and how a reviewer can reproduce: commands to run, steps to reproduce, test coverage added. Include screenshots or recordings for UI changes.
6. **Flag risk and scope.** Note breaking changes, migrations, feature flags, config/env changes, rollout/rollback steps, and anything explicitly *out of scope* or deferred to a follow-up.
7. **Keep PRs small.** If the description is sprawling, the PR is probably too big — consider splitting it.
8. **Guide the reviewer.** Optionally point to the best file to start with, or areas where you specifically want feedback.

## Examples

```markdown
## Why
Users on slow networks saw duplicate orders because the checkout button
stayed enabled during the request. Closes #732.

## What changed
- Disable the checkout button and show a spinner while the order request
  is in flight.
- Add an idempotency key so retried requests do not create duplicates.
- Cover the double-submit path with an integration test.

## How to test
1. `npm run dev` and open /checkout.
2. Throttle the network (DevTools > Slow 3G) and click "Place order" twice.
3. Confirm only one order is created and the button is disabled mid-request.

## Risk & rollout
- No schema change. Idempotency keys are stored in Redis with a 24h TTL.
- Behind no flag; low risk. Rollback = revert this PR.

## Out of scope
- Retry UX for failed payments (tracked in #740).
```

## Checklist

- [ ] Title is a clear one-line summary.
- [ ] The "why" and the linked issue are stated up front.
- [ ] Key changes are summarized conceptually, not as a file dump.
- [ ] Non-obvious decisions and trade-offs are explained.
- [ ] Clear steps for a reviewer to test/verify (plus visuals for UI).
- [ ] Breaking changes, migrations, and rollout/rollback noted.
- [ ] PR is scoped small enough to review in one sitting.
