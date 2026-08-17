---
name: test-coverage-audit
description: Assess where a codebase's tests are thin and prioritize the tests that would actually reduce risk — use when coverage numbers are unclear or gamed, before a risky refactor or release, when the user asks "what should we test next", or to turn a raw coverage report into a ranked, meaningful test plan rather than chasing a percentage.
---

# Test Coverage Audit

A coverage audit answers "where is our testing weakest relative to risk?" — not "what percentage did we hit?". Line coverage is a weak signal: code can be executed without any assertion, and 90% coverage can leave the most dangerous branch untested. This skill uses coverage data as one input among several to produce a prioritized list of meaningful tests to add.

## When to use this skill

- Before a risky refactor, migration, or release and you want to know where the net has holes.
- The coverage number looks high but bugs still ship, suggesting assertions are missing.
- Onboarding to an unfamiliar codebase and deciding where testing effort pays off.
- The user asks "what should we test next", "is our coverage good", or wants a test backlog.
- Coverage gates feel gamed and you want to measure risk instead of a metric.

## Instructions

1. Generate a coverage report with branch (not just line) coverage enabled, and produce a per-file/per-function breakdown you can sort.
2. Map risk independently of coverage. Rank modules by: blast radius if broken (money, auth, data integrity), change frequency (churn from version control history), complexity (many branches/conditions), and recent bug history.
3. Build a risk × coverage matrix. The priority quadrant is HIGH risk + LOW coverage. High risk + high coverage may still need better assertions; low risk + low coverage can usually wait.
4. Inspect the covered-but-unasserted code: search for tests that execute a path without asserting its outcome (e.g. call then no assert, or asserting only "did not throw"). These inflate coverage without protecting behavior.
5. Find untested branches, not just untested lines: error handling, early returns, boundary conditions, null/empty inputs, and the "impossible" else. These are where real defects hide.
6. Check for missing test types: pure logic without unit tests, boundaries without integration tests, and critical flows without any end-to-end coverage.
7. Produce a ranked backlog. For each gap, write: the file/behavior, why it is risky, the specific test to add, and the test type. Order by risk-reduction per unit of effort.
8. Recommend guardrails, not vanity targets: consider requiring coverage on changed lines in CI rather than a global percentage, and protecting the high-risk modules explicitly.
9. If asked, write the top-priority tests first and re-measure to confirm the gap closed with real assertions.

## Examples

A ranked gap report distilled from a coverage run plus risk analysis.

```text
Priority backlog (highest risk-reduction first):

1. payments/refund.py :: partial_refund()   [HIGH risk, 0% branch cov]
   Risk: touches money; refund > original amount path unguarded.
   Add: unit tests for full, partial, over-refund (expect error), zero.

2. auth/session.py :: validate()            [HIGH risk, lines covered / no asserts]
   Risk: test calls validate() but never asserts expiry rejection.
   Add: assert expired + tampered tokens are rejected.

3. import/csv_loader.py :: parse_row()       [MED risk, 40% branch cov]
   Risk: malformed row / missing column branches untested.
   Add: table test for empty file, extra column, bad encoding.

Deferred: reporting/format.py (LOW risk, cosmetic) — leave for now.
```

Spotting a coverage-but-no-assertion smell.

```python
# Counts as "covered" but asserts nothing meaningful:
def test_process():
    process(order)          # executes the line, verifies no behavior

# Meaningful replacement:
def test_process_marks_order_shipped():
    result = process(order)
    assert result.status == "shipped"
    assert order.shipped_at is not None
```

## Checklist

- [ ] Report includes branch coverage and a per-file/function breakdown.
- [ ] Modules ranked by risk (blast radius, churn, complexity, bug history) independent of coverage.
- [ ] High-risk + low-coverage gaps identified via a risk × coverage matrix.
- [ ] Covered-but-unasserted code flagged separately from truly uncovered code.
- [ ] Untested branches and error paths called out, not just line gaps.
- [ ] Output is a ranked, actionable backlog naming the specific test and type for each gap.
- [ ] Recommendation favors changed-lines/high-risk guardrails over a vanity percentage.
