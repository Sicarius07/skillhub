---
name: changelog-generator
description: Maintain a human-readable CHANGELOG.md in the Keep a Changelog format with an Unreleased section and grouped Added/Changed/Fixed/Removed entries tied to semantic versioning — use when releasing a version, recording notable changes, updating release notes, or when a user asks how to structure or write a changelog.
---

# Changelog Generator

A changelog is a curated, human-readable record of notable changes per release — not a raw git log. The Keep a Changelog convention groups entries by type under dated version headings and keeps an `Unreleased` section at the top so changes are recorded as they land. Version numbers follow Semantic Versioning.

## When to use this skill

- You are cutting a release and need to finalize its notes.
- A notable change (feature, fix, breaking change) just merged and should be recorded.
- A project has no changelog, or an unstructured one, and needs a consistent format.
- A user asks how to write release notes or what goes in a CHANGELOG.

## Instructions

1. **Create or open `CHANGELOG.md`** at the repo root. Start with a short header noting the format and versioning scheme used.
2. **Keep an `## [Unreleased]` section** at the top. Add entries here as changes merge, so nothing is forgotten at release time.
3. **Group entries** under these headings (omit empty ones):
   - `Added` — new features.
   - `Changed` — changes in existing behavior.
   - `Deprecated` — soon-to-be-removed features.
   - `Removed` — features removed now.
   - `Fixed` — bug fixes.
   - `Security` — vulnerability fixes.
4. **Write for humans.** Each entry is a short, user-facing sentence describing the impact — not a commit hash or internal refactor detail. Link issues/PRs where useful.
5. **On release**, rename `[Unreleased]` to the new version with an ISO date: `## [1.4.0] - 2026-08-17`, then start a fresh empty `[Unreleased]` above it.
6. **Choose the version bump** per SemVer: breaking → MAJOR, new backward-compatible feature → MINOR, backward-compatible fix → PATCH.
7. **Order releases newest-first.** Optionally maintain comparison links at the bottom (`[1.4.0]: .../compare/v1.3.0...v1.4.0`).
8. **Automate if desired** by deriving entries from Conventional Commits, but always review the generated text for clarity before publishing.

## Examples

```markdown
# Changelog

All notable changes to this project are documented here.
The format is based on Keep a Changelog, and this project adheres to
Semantic Versioning.

## [Unreleased]

## [1.4.0] - 2026-08-17
### Added
- Fuzzy search so typos still match product names (#482).

### Changed
- Default request timeout raised from 5s to 30s for slow networks.

### Fixed
- Prevent duplicate orders when the checkout button is double-clicked (#732).

## [1.3.0] - 2026-07-02
### Added
- CSV export for the reporting dashboard.

[Unreleased]: https://example.com/compare/v1.4.0...HEAD
[1.4.0]: https://example.com/compare/v1.3.0...v1.4.0
```

## Checklist

- [ ] File follows Keep a Changelog structure with an `Unreleased` section.
- [ ] Entries are grouped under the correct type headings.
- [ ] Each entry is user-facing and readable, not a raw commit message.
- [ ] Released version has a SemVer number and ISO date, newest first.
- [ ] Version bump matches the nature of the changes (major/minor/patch).
- [ ] Breaking changes are clearly called out.
- [ ] Comparison/reference links (if used) are updated.

## Format
- Group entries under Added / Changed / Fixed.
