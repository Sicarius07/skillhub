---
name: security-review
description: Perform a defensive security review of a code change or pull request — systematically check for injection, broken access control, secrets, unsafe deserialization, SSRF, XSS, insecure crypto, and other common vulnerability classes — use before merging security-sensitive code, when reviewing a PR, or when hardening an existing module.
---

# Security Review

A focused, defensive review that reads a diff or module through an attacker-awareness lens and reports concrete, fixable weaknesses. The goal is to find and remediate vulnerabilities, never to exploit them. Prioritize findings by real-world impact and give the author actionable fixes.

## When to use this skill

- Reviewing a pull request that touches auth, input handling, data access, file I/O, or network calls.
- Hardening an existing feature or endpoint before release.
- Someone asks "is this code safe?" or reports a suspected weakness.
- Onboarding a new dependency or integration that handles untrusted data.

## Instructions

1. **Scope the review.** Identify the trust boundaries: where untrusted input enters (HTTP params, headers, uploads, message queues, third-party APIs) and where sensitive actions happen (DB, filesystem, shell, auth, payments).
2. **Walk each input to each sink.** For every untrusted input, trace whether it reaches a dangerous sink without validation or safe encoding. Check these classes:
   - **Injection**: SQL/NoSQL, OS command, LDAP, template. Require parameterized queries / safe APIs, never string concatenation.
   - **XSS**: untrusted data rendered into HTML/JS without contextual output encoding; unsafe `innerHTML`/`dangerouslySetInnerHTML`.
   - **Broken access control**: missing authorization checks, IDOR (object IDs not scoped to the caller), privilege escalation, mass-assignment of protected fields.
   - **SSRF**: server fetching a user-supplied URL without allow-listing.
   - **Path traversal**: user-controlled file paths without normalization/containment.
   - **Unsafe deserialization**: untrusted data into native deserializers (pickle, Java, YAML `load`).
   - **Secrets**: hardcoded keys, tokens, passwords, or secrets logged in plaintext.
   - **Crypto**: weak/rolled-your-own crypto, ECB mode, MD5/SHA1 for passwords, missing salts, predictable randomness for security tokens.
   - **SSTI / open redirect / CSRF / insecure CORS** where relevant.
3. **Check authN/authZ explicitly.** Confirm every state-changing or data-returning path enforces authentication and per-resource authorization.
4. **Review error handling and logging.** Ensure errors don't leak stack traces/secrets to users and that sensitive fields are redacted in logs.
5. **Assess dependencies and config.** Note risky new dependencies, disabled TLS verification, debug mode in prod, or permissive defaults.
6. **Report findings by severity.** For each: location, the vulnerability class, why it is exploitable, and a concrete fix. Use Critical/High/Medium/Low.
7. **Recommend defenses in depth.** Suggest validation, least privilege, and tests that would catch a regression.

## Examples

A SQL injection finding and its fix:

```python
# FINDING (High): SQL injection — user input concatenated into query
cur.execute("SELECT * FROM users WHERE email = '" + email + "'")

# FIX: parameterized query — driver escapes the value
cur.execute("SELECT * FROM users WHERE email = %s", (email,))
```

An IDOR finding — the record is fetched by ID but never scoped to the caller:

```python
# FINDING (High): any authenticated user can read any invoice by guessing id
inv = Invoice.get(request.args["id"])
return inv.to_json()

# FIX: scope the lookup to the authenticated owner
inv = Invoice.get(id=request.args["id"], owner_id=current_user.id)
if inv is None:
    abort(404)
```

## Checklist

- [ ] Trust boundaries and all untrusted-input sources identified.
- [ ] Each input traced to sinks; injection/XSS/SSRF/traversal checked.
- [ ] Authentication and per-resource authorization verified on every sensitive path.
- [ ] No hardcoded secrets; secrets not logged.
- [ ] Cryptography uses vetted algorithms, proper password hashing, and CSPRNG.
- [ ] Errors/logs do not leak sensitive data or stack traces to users.
- [ ] Findings reported with location, impact, severity, and concrete fix.
- [ ] Regression tests or guardrails recommended.
