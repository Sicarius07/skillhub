---
name: write-readme
description: Write an effective project README that helps users understand, install, and use a project quickly — covering the one-liner, badges, install, quick start, usage, configuration, contributing, and license — use when creating or overhauling a README, documenting a new repo, or when a user asks how to structure project documentation.
---

# Write README

The README is the front door of a project. A strong one answers, in order: what is this, why should I care, how do I get it running, and where do I go next. It is optimized for someone scanning in 60 seconds, then progressively reveals depth for those who need it.

## When to use this skill

- Starting a new repository or open-source project.
- An existing README is thin, outdated, or disorganized.
- You are publishing a library, CLI, or service others will consume.
- A user asks how to document a project or what a README should contain.

## Instructions

1. **Open with a title and one-liner.** Name the project and describe what it does in a single sentence a newcomer understands. Avoid jargon.
2. **State the value / why.** One short paragraph on the problem it solves and who it is for. Optionally add badges (build status, version, license) directly under the title.
3. **Show it fast.** Include a minimal usage snippet, screenshot, or GIF near the top so readers see the payoff before install details.
4. **Installation.** Give copy-pasteable commands for the common environment(s). List prerequisites (runtime versions, system deps).
5. **Quick start / usage.** Provide a runnable minimal example, then a few common tasks. Prefer real, correct commands over prose.
6. **Configuration.** Document key options, environment variables, and config files in a table (name, description, default).
7. **Link out for depth.** Point to fuller docs, API reference, examples, or a wiki rather than inlining everything.
8. **Contributing & support.** Explain how to report bugs, request features, run tests, and submit PRs. Link a CONTRIBUTING guide and code of conduct if present.
9. **License & credits.** State the license and acknowledge major dependencies or contributors.
10. **Keep it current.** Ensure every command actually works; a broken quick start is worse than none.

## Examples

Skeleton of a strong README:

```markdown
# Fizzbuzz CLI

A tiny, fast command-line tool that turns any range of numbers into a
FizzBuzz sequence — for teaching, benchmarking, and demos.

[![build](https://img.shields.io/badge/build-passing-green)]() [![license: MIT](https://img.shields.io/badge/license-MIT-blue)]()

## Quick start
```bash
npm install -g fizzbuzz-cli
fizzbuzz 1 15
# 1 2 Fizz 4 Buzz Fizz 7 8 Fizz Buzz 11 Fizz 13 14 FizzBuzz
```

## Installation
Requires Node.js 18+.
```bash
npm install -g fizzbuzz-cli
```

## Configuration
| Option      | Description                    | Default |
|-------------|--------------------------------|---------|
| `--fizz`    | Divisor for "Fizz"             | 3       |
| `--buzz`    | Divisor for "Buzz"             | 5       |

## Contributing
See [CONTRIBUTING.md](CONTRIBUTING.md). Run `npm test` before opening a PR.

## License
MIT © 2026 Example Authors
```

## Checklist

- [ ] Title plus a one-sentence description of what the project does.
- [ ] The "why" / value is clear within the first screen.
- [ ] A working quick-start example appears early.
- [ ] Install steps list prerequisites and are copy-pasteable.
- [ ] Configuration/options are documented (table where helpful).
- [ ] Contributing, support, and license are covered.
- [ ] Every command in the README was verified to work.
