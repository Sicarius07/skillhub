---
name: secrets-scanning
description: Detect leaked credentials — API keys, tokens, passwords, private keys, connection strings — in source, config, history, and logs, then safely remediate by rotating and preventing recurrence — use before committing, when auditing a repo, after a suspected leak, or when setting up pre-commit/CI secret scanning.
---

# Secrets Scanning

Committed secrets are one of the most common and damaging leaks. This skill covers finding secrets across the working tree, git history, and CI logs; correctly remediating (rotate first, then purge); and adding automated prevention so they don't come back. The single most important rule: a leaked secret must be considered compromised and rotated — deleting it from the repo is not enough.

## When to use this skill

- Before committing or opening a PR that adds config, infra, or client code.
- Auditing a repository or codebase for exposed credentials.
- Responding to a suspected or confirmed secret leak.
- Setting up pre-commit hooks or CI secret-scanning gates.
- A secret appeared in logs, an error message, or a public artifact.

## Instructions

1. **Scan the working tree.** Run a dedicated scanner over the checkout: `gitleaks detect`, `trufflehog filesystem .`, or `detect-secrets scan`. These use entropy + known-provider patterns (AWS `AKIA…`, GitHub `ghp_…`, Google, Stripe `sk_live_…`, private key headers, JWTs, DB connection strings).
2. **Scan git history, not just HEAD.** Secrets removed in a later commit still live in history. Run `gitleaks detect --log-opts="--all"` or `trufflehog git file://.` over the full history.
3. **Scan CI logs and artifacts** if the secret may have been echoed during builds or printed in stack traces.
4. **Triage each hit.** Confirm real secrets vs. false positives (example values, test fixtures, public keys). Classify by blast radius: what does this credential grant, and where is it valid?
5. **Remediate — in this order:**
   - **Rotate/revoke first.** Issue a new credential and invalidate the exposed one at the provider. Assume it is compromised the moment it hit a shared history.
   - **Remove from code.** Move the value to a secrets manager or environment variable; reference it, never inline it.
   - **Purge from history if needed** using `git filter-repo` or BFG, then force-push and have collaborators re-clone. Note: rotation is still required — purging alone does not undo exposure.
6. **Prevent recurrence.** Add a pre-commit hook (gitleaks/detect-secrets) and a CI job that fails on new findings. Maintain an allowlist/baseline for known false positives. Add sensitive paths to `.gitignore`.
7. **Document** what leaked, when, the rotation performed, and follow-ups.

## Examples

Scanning a repo including full history and outputting a report:

```bash
# Working tree + all history
gitleaks detect --source . --log-opts="--all" --report-path gitleaks-report.json

# Deep verification against providers (validates if a key is live)
trufflehog git file://. --only-verified
```

A pre-commit hook that blocks staged secrets before they are committed:

```yaml
# .pre-commit-config.yaml
repos:
  - repo: https://github.com/gitleaks/gitleaks
    rev: v8.18.0
    hooks:
      - id: gitleaks
```

Replacing an inline secret with an environment reference:

```diff
- STRIPE_KEY = "sk_live_51H8xâ€¦REDACTED"
+ STRIPE_KEY = os.environ["STRIPE_KEY"]   # value stored in the secrets manager
```

## Checklist

- [ ] Working tree scanned with a dedicated secret scanner.
- [ ] Full git history scanned (`--all`), not just current files.
- [ ] CI logs/artifacts checked if exposure there was possible.
- [ ] Every real hit triaged for validity and blast radius.
- [ ] Exposed credentials rotated/revoked at the provider (done first).
- [ ] Secrets moved to a manager/env vars and referenced, not inlined.
- [ ] History purged where warranted (with force-push + re-clone).
- [ ] Pre-commit hook and CI scan added; baseline for false positives set.
