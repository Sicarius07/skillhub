---
name: dependency-audit
description: Audit a project's third-party dependencies for security vulnerabilities, unused or duplicate packages, outdated versions, and license/maintenance risk, then prune and update them safely; use when asked to audit dependencies, reduce bundle/install size, fix vulnerable packages, or clean up the dependency tree.
---

# Dependency Audit

Third-party dependencies are attack surface, maintenance burden, and weight. This skill inventories what a project depends on, flags what is vulnerable, unused, duplicated, outdated, or poorly maintained, and prunes or upgrades it without breaking the build. It works across ecosystems (npm, pip/Poetry, Cargo, Go modules, Maven/Gradle, etc.).

## When to use this skill

- The user asks to audit, review, or clean up dependencies.
- There are security advisories, CVEs, or a failing vulnerability scan.
- Install size, bundle size, or build time has grown and needs trimming.
- You are assessing supply-chain, license, or maintenance risk before a release.

## Instructions

1. Build an accurate inventory. Identify the manifest and lockfile (`package.json`/`package-lock.json`, `pyproject.toml`/`poetry.lock`, `Cargo.toml`/`Cargo.lock`, `go.mod`/`go.sum`, etc.). Separate direct dependencies from transitive, and runtime from dev/build/test.
2. Scan for known vulnerabilities. Run the ecosystem's audit tool (`npm audit`, `pip-audit`, `cargo audit`, `govulncheck`, `osv-scanner`). Triage by severity and whether the vulnerable path is actually reachable, not just present.
3. Find unused dependencies. Use tooling where available (e.g., `depcheck`, `deptry`, `cargo-udeps`) and cross-check with a repo-wide import/usage search. Confirm a package is truly unused before removing it — watch for dynamic imports, plugins, and build-only tools.
4. Detect duplicates and bloat. Inspect the resolved tree for multiple versions of the same package and heavy transitive dependencies. Deduplicate/hoist where the ecosystem allows, and prefer lighter alternatives for oversized single-purpose packages.
5. Assess update posture and maintenance risk. List outdated packages and separate patch/minor (low risk) from major (breaking). Note unmaintained, deprecated, single-maintainer, or recently transferred packages as supply-chain risk.
6. Check licenses. Flag licenses incompatible with the project's distribution model (e.g., copyleft in proprietary software) and any missing/unknown licenses.
7. Change safely and incrementally. Update or remove in small commits — ideally one concern per commit. After each change, reinstall from the lockfile, run the full test suite and build, and commit the updated lockfile. Pin/lock versions and prefer reproducible installs.

## Examples

A minimal audit pass in an npm project:

```bash
npm audit --omit=dev                 # vulnerabilities in runtime deps
npx depcheck                         # unused deps and missing deps
npm ls --all | grep -A1 "lodash"     # duplicate versions in the tree
npm outdated                         # patch/minor/major update posture
```

Removing a confirmed-unused dependency safely:

```bash
grep -rn "from 'moment'" src/        # confirm no imports (incl. dynamic)
npm remove moment
npm ci && npm test && npm run build  # reinstall from lock, verify green
git add package.json package-lock.json && git commit -m "deps: drop unused moment"
```

## Checklist

- [ ] Direct vs transitive and runtime vs dev dependencies are inventoried.
- [ ] A vulnerability scan ran and findings are triaged by severity and reachability.
- [ ] Unused packages are confirmed (tooling + usage search) before removal.
- [ ] Duplicate versions and oversized packages are identified and addressed.
- [ ] Outdated packages are split into safe minor/patch vs breaking major updates.
- [ ] Licenses are checked for compatibility; unmaintained packages are flagged.
- [ ] Each change is small; tests, build, and lockfile are green and committed.
