---
name: auth-best-practices
description: Implement or review secure authentication and session/token handling — password storage, login flows, session cookies, JWTs, refresh/rotation, MFA, and logout — use when building or auditing sign-in, session management, API tokens, OAuth/OIDC integration, or password reset, or when hardening an existing auth system defensively.
---

# Authentication Best Practices

Authentication is high-value attack surface; small mistakes lead to account takeover. This skill covers the defensive fundamentals of proving identity and maintaining sessions securely: hashing credentials correctly, issuing and storing session artifacts safely, expiring and rotating them, and closing common gaps like enumeration and fixation. Prefer a vetted library or identity provider over building primitives yourself.

## When to use this skill

- Building or reviewing login, signup, password reset, or logout flows.
- Designing session cookies, API keys, or JWT/refresh-token handling.
- Integrating OAuth 2.0 / OIDC or an external identity provider.
- Adding MFA or hardening an existing authentication system.

## Instructions

1. **Store passwords with a slow, salted hash.** Use `argon2id`, `bcrypt`, or `scrypt` with sensible cost parameters and a unique per-user salt (built into these algorithms). Never store plaintext, never use fast hashes (MD5/SHA-256) for passwords, never roll your own.
2. **Make login safe against enumeration and guessing.** Return the same generic error for unknown user vs. wrong password, use constant-time comparison, and apply rate limiting / lockout / CAPTCHA on repeated failures. Log auth events for monitoring.
3. **Choose sessions vs. tokens deliberately:**
   - **Server-side sessions:** store a random opaque ID; keep state server-side. Easy to revoke.
   - **JWTs:** stateless but hard to revoke — keep them short-lived, verify signature and `alg` (reject `none`), validate `exp`/`aud`/`iss`.
4. **Protect session cookies.** Set `HttpOnly`, `Secure`, and `SameSite` (Lax/Strict). Scope `Path`/`Domain` tightly. Use CSRF protection (token or SameSite) for cookie-based auth.
5. **Manage token lifecycle.** Use short-lived access tokens plus longer-lived refresh tokens; **rotate refresh tokens on use** and detect reuse (a replayed old refresh token means theft — revoke the family). Regenerate the session ID on login to prevent fixation.
6. **Support MFA and strong recovery.** Offer TOTP/WebAuthn. Make password reset use single-use, expiring, unpredictable tokens and invalidate existing sessions on password change.
7. **Logout and expiry that work.** On logout, invalidate the server session / revoke tokens (not just clear the cookie). Enforce idle and absolute session timeouts.
8. **Prefer proven building blocks.** Delegate to a maintained library or IdP (OAuth/OIDC) rather than hand-rolling crypto, and follow least privilege for scopes.

## Examples

Hashing and verifying a password with a modern KDF:

```python
from argon2 import PasswordHasher
ph = PasswordHasher()                 # argon2id with sane defaults

hashed = ph.hash(password)            # store this; salt is embedded
# On login — constant-time verify, generic error on failure
try:
    ph.verify(hashed, submitted)
except Exception:
    raise AuthError("Invalid email or password")  # same message for all failures
```

Hardened session cookie and refresh-token rotation:

```
Set-Cookie: sid=<opaque-random>; HttpOnly; Secure; SameSite=Lax; Path=/; Max-Age=3600
```

```
POST /token/refresh  ->  issue new access + new refresh token,
                         invalidate the presented refresh token.
If a already-used refresh token is presented again -> revoke the whole token family (theft).
```

## Checklist

- [ ] Passwords hashed with argon2id/bcrypt/scrypt and per-user salt; no fast hashes.
- [ ] Login resists enumeration (generic errors) and guessing (rate limit/lockout).
- [ ] Session vs. JWT chosen deliberately; JWTs short-lived with verified alg/exp/aud.
- [ ] Cookies set HttpOnly, Secure, SameSite; CSRF protection in place.
- [ ] Session ID regenerated on login; refresh tokens rotate with reuse detection.
- [ ] MFA available; password reset uses single-use expiring tokens; reset revokes sessions.
- [ ] Logout revokes server-side session/tokens; idle and absolute timeouts enforced.
- [ ] Vetted library/IdP used instead of hand-rolled crypto.
