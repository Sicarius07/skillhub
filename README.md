# skillhub

A central repository of reusable **Agent Skills** — procedural knowledge that AI coding agents (Claude Code, Cursor, GitHub Copilot, and others) can load on demand to perform engineering tasks more reliably.

Each skill lives in its own folder with a `SKILL.md` file containing YAML frontmatter (`name`, `description`) and a structured body: when to use it, step-by-step instructions, worked examples, and a verification checklist.

## Install

Install every skill in this repo into your agent with a single command:

```bash
npx skills add Sicarius07/skillhub
```

Or reference an individual skill folder directly by copying it into your project's skills directory (e.g. `.claude/skills/`).

## Skills

### Code Quality & Review
- [code-review](./code-review) — review a diff/PR for correctness, clarity, and risk
- [refactor-safely](./refactor-safely) — refactor without changing behavior, guarded by tests
- [simplify-code](./simplify-code) — reduce complexity and remove needless abstraction
- [dead-code-removal](./dead-code-removal) — safely find and delete unused code
- [naming-conventions](./naming-conventions) — choose clear, consistent names
- [error-handling-patterns](./error-handling-patterns) — robust error handling and propagation
- [logging-best-practices](./logging-best-practices) — structured, useful, non-noisy logging
- [dependency-audit](./dependency-audit) — audit and prune project dependencies

### Testing & Debugging
- [tdd](./tdd) — test-driven development red/green/refactor loop
- [write-unit-tests](./write-unit-tests) — focused, fast, deterministic unit tests
- [integration-testing](./integration-testing) — test components working together
- [test-coverage-audit](./test-coverage-audit) — find coverage gaps that matter
- [flaky-test-hunter](./flaky-test-hunter) — diagnose and fix nondeterministic tests
- [root-cause-debugging](./root-cause-debugging) — hypothesis-driven debugging to the true cause
- [bisect-regression](./bisect-regression) — isolate a regression via bisection
- [add-observability](./add-observability) — add metrics, traces, and logs to diagnose issues

### Git & Documentation
- [conventional-commits](./conventional-commits) — write commits in Conventional Commits format
- [clean-git-history](./clean-git-history) — craft a readable, atomic history
- [resolve-merge-conflicts](./resolve-merge-conflicts) — resolve conflicts safely and correctly
- [pr-description-writer](./pr-description-writer) — write clear, reviewable PR descriptions
- [write-readme](./write-readme) — write an effective project README
- [changelog-generator](./changelog-generator) — maintain a human-readable CHANGELOG
- [adr-writer](./adr-writer) — write Architecture Decision Records
- [api-documentation](./api-documentation) — document an API clearly

### Architecture & Data
- [domain-modeling](./domain-modeling) — model a domain with clear boundaries and language
- [rest-api-design](./rest-api-design) — design clean, consistent RESTful APIs
- [database-schema-design](./database-schema-design) — design normalized, evolvable schemas
- [design-patterns](./design-patterns) — apply the right pattern without over-engineering
- [caching-strategy](./caching-strategy) — decide what/where/how to cache and invalidate
- [sql-query-optimization](./sql-query-optimization) — diagnose and optimize slow queries
- [data-migration](./data-migration) — plan safe, reversible data/schema migrations
- [event-driven-design](./event-driven-design) — design event-driven systems

### Performance & Security
- [performance-profiling](./performance-profiling) — profile before optimizing
- [bundle-size-reduction](./bundle-size-reduction) — shrink frontend bundle size
- [security-review](./security-review) — review changes for common vulnerabilities (defensive)
- [secrets-scanning](./secrets-scanning) — detect and remediate leaked secrets
- [input-validation](./input-validation) — validate and sanitize untrusted input
- [dependency-vulnerability-scan](./dependency-vulnerability-scan) — scan deps for known CVEs
- [auth-best-practices](./auth-best-practices) — secure authentication and session handling
- [rate-limiting](./rate-limiting) — design rate limiting and abuse protection

### Frontend & DevOps
- [accessibility-audit](./accessibility-audit) — audit and fix web accessibility (WCAG)
- [responsive-design](./responsive-design) — layouts that work across screen sizes
- [component-design](./component-design) — reusable, composable UI components
- [dockerize-app](./dockerize-app) — clean, small, secure Docker images
- [ci-pipeline-setup](./ci-pipeline-setup) — design a CI pipeline with quality gates
- [github-actions-workflow](./github-actions-workflow) — maintainable GitHub Actions workflows
- [env-config-management](./env-config-management) — manage config and secrets across environments
- [feature-flags](./feature-flags) — introduce and manage feature flags safely

## Skill format

```markdown
---
name: my-skill
description: What this skill does and when to use it.
---

# My Skill

Overview paragraph.

## When to use this skill
- ...

## Instructions
1. ...

## Examples
...

## Checklist
- [ ] ...
```

## Contributing

Add a new skill by creating a folder named after the skill (kebab-case) with a `SKILL.md` inside, following the format above, and add it to the list in this README.

## License

MIT
