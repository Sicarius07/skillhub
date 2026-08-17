---
name: ci-pipeline-setup
description: Design a fast, reliable CI pipeline with clear stages for lint, type-check, test, build, and security scanning, plus caching, parallelism, and merge gates; use when setting up CI for a repo, speeding up or stabilizing a flaky pipeline, defining required checks and branch protection, or when users mention CI, build pipeline, quality gates, test stages, or continuous integration.
---

# CI Pipeline Setup

This skill describes how to design a continuous integration pipeline that gives fast, trustworthy feedback and enforces quality before code merges. It is platform-agnostic (GitHub Actions, GitLab CI, CircleCI, Jenkins) and focuses on stage design, caching, and gating.

## When to use this skill

- Setting up CI for a new repository or service.
- An existing pipeline is slow, flaky, or lets broken code merge.
- Defining required status checks and branch protection rules.
- Users mention CI, pipeline stages, quality gates, caching, or continuous integration.

## Instructions

1. **Define triggers.** Run the full pipeline on pull requests and on pushes to the default branch. Consider lighter jobs for draft PRs and scheduled jobs for slow security scans.
2. **Order stages fast-to-slow.** Put cheap, high-signal checks first (lint, format, type-check) so obvious failures fail fast, then unit tests, then integration/e2e, then build and package.
3. **Make jobs independent and parallel.** Split checks into separate jobs that can run concurrently; fan out test suites by shard or workspace to shorten wall-clock time.
4. **Cache aggressively but correctly.** Cache dependency directories and build artifacts keyed on lockfile hashes. Restore caches at the start of jobs and invalidate them when inputs change.
5. **Pin and reproduce the environment.** Pin language/tool versions and action/image versions. Install exactly from lockfiles (`npm ci`, `pip install -r`, `go mod download`) so CI matches locally.
6. **Enforce quality gates.** Fail the build on lint errors, type errors, failing tests, or coverage below threshold. Add security scanning (dependency audit, SAST, secret scanning) as gates or advisories.
7. **Produce and share artifacts.** Upload build output, test reports, and coverage so results are inspectable; pass built artifacts between jobs instead of rebuilding.
8. **Set required checks + branch protection.** Mark the essential jobs as required for merge, require up-to-date branches, and block direct pushes to the default branch.
9. **Keep it fast and stable.** Target a short PR feedback loop (aim under ~10 minutes). Quarantine or fix flaky tests promptly; retries should be a stopgap, not a strategy.
10. **Add observability.** Surface timings, failure reasons, and trends so you can spot regressions in pipeline health.

## Examples

Recommended stage flow:

```
lint + format + type-check   (parallel, fail fast)
        |
     unit tests               (sharded, parallel)
        |
integration / e2e tests
        |
   build + package
        |
 security scan (audit/SAST)
        |
   required checks -> merge
```

Cache key strategy (pseudocode, applies to most CI systems):

```yaml
cache:
  key: deps-${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
  paths:
    - ~/.npm
    - node_modules
```

Coverage gate as a hard failure:

```bash
# Fail CI if line coverage drops below 80%
npm test -- --coverage --coverageThreshold='{"global":{"lines":80}}'
```

## Checklist

- [ ] Pipeline triggers on PRs and default-branch pushes.
- [ ] Stages ordered fast-to-slow; cheap checks fail first.
- [ ] Jobs run in parallel / test suites sharded.
- [ ] Dependencies and build outputs cached with correct keys.
- [ ] Tool versions pinned; installs come from lockfiles.
- [ ] Lint, type, test, coverage, and security gates enforced.
- [ ] Artifacts and reports uploaded and reused across jobs.
- [ ] Required checks + branch protection configured.
- [ ] PR feedback loop is fast; flaky tests tracked and fixed.
