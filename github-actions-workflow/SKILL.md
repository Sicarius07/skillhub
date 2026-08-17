---
name: github-actions-workflow
description: Author maintainable, secure GitHub Actions workflows with pinned actions, least-privilege permissions, matrix builds, dependency caching, concurrency control, reusable workflows, and OIDC-based deploys; use when creating or refactoring a .github/workflows file, debugging Actions failures, hardening workflow security, or when users mention GitHub Actions, workflow YAML, jobs, runners, or CI on GitHub.
---

# GitHub Actions Workflow

This skill covers writing GitHub Actions workflows that are readable, fast, and secure. It complements ci-pipeline-setup by focusing on GitHub-specific mechanics: triggers, permissions, caching, matrices, reuse, and secrets handling.

## When to use this skill

- Creating a new workflow under `.github/workflows/`.
- Refactoring a workflow that is slow, duplicated, or over-permissioned.
- Debugging failing or flaky Actions runs.
- Hardening workflow security (secrets, third-party actions, deploy credentials).
- Users mention GitHub Actions, workflow YAML, jobs, runners, or matrix builds.

## Instructions

1. **Scope triggers precisely.** Use `on:` with the events you need (`pull_request`, `push` to specific branches, `workflow_dispatch`, `schedule`). Add `paths`/`branches` filters to avoid unnecessary runs.
2. **Set least-privilege permissions.** Add a top-level `permissions:` block defaulting to `contents: read`, and elevate per-job only where required (e.g., `id-token: write` for OIDC, `pull-requests: write` for comments).
3. **Add concurrency control.** Use a `concurrency` group with `cancel-in-progress: true` to cancel superseded runs on the same ref and save minutes.
4. **Pin third-party actions.** Reference actions by full commit SHA (or at least a version tag) rather than a moving branch, to prevent supply-chain surprises.
5. **Cache dependencies.** Use `actions/cache` or the built-in caching in setup actions, keyed on lockfile hashes, to speed up installs.
6. **Use matrix builds for coverage.** Test across OS/runtime versions with `strategy.matrix`; set `fail-fast: false` when you want all combinations to report.
7. **Factor out reusable workflows.** Extract shared logic into `workflow_call` reusable workflows or composite actions instead of copy-pasting steps across files.
8. **Handle secrets safely.** Reference `${{ secrets.X }}` only in trusted contexts, avoid printing them, and prefer OIDC federation over long-lived cloud keys for deploys.
9. **Guard `pull_request_target`.** Avoid running untrusted PR code with elevated permissions; prefer `pull_request` for fork PRs and never checkout+build untrusted code with secrets.
10. **Name and observe.** Give jobs/steps clear `name:`s, upload artifacts and test reports, and use step summaries so failures are easy to diagnose.

## Examples

A lean, hardened CI workflow:

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        node: [18, 20, 22]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
          cache: npm
      - run: npm ci
      - run: npm run lint
      - run: npm test -- --coverage
```

OIDC-based deploy (no long-lived cloud secrets):

```yaml
  deploy:
    needs: test
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - uses: actions/checkout@v4
      - uses: aws-actions/configure-aws-credentials@v4
        with:
          role-to-assume: arn:aws:iam::123456789012:role/deploy
          aws-region: us-east-1
      - run: ./scripts/deploy.sh
```

## Checklist

- [ ] Triggers scoped with branch/path filters.
- [ ] Top-level `permissions` default to read; elevated only per-job.
- [ ] `concurrency` cancels superseded runs.
- [ ] Third-party actions pinned to SHA or version.
- [ ] Dependencies cached by lockfile hash.
- [ ] Matrix used for multi-version/OS coverage where relevant.
- [ ] Shared logic extracted into reusable/composite workflows.
- [ ] Secrets never logged; OIDC preferred over static cloud keys.
- [ ] No untrusted code runs with elevated permissions.
- [ ] Jobs/steps named; artifacts and summaries uploaded.
