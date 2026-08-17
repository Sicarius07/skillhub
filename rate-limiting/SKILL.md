---
name: rate-limiting
description: Design and implement rate limiting and abuse protection for APIs and endpoints — choose an algorithm (token bucket, sliding window), pick the right key and scope, return correct headers/429s, and layer defenses for login, signup, and expensive operations — use when protecting a service from abuse, brute force, scraping, or overload, or when reviewing an existing throttling design.
---

# Rate Limiting & Abuse Protection

Rate limiting protects availability, controls cost, and blunts abuse like credential stuffing and scraping. This skill helps you pick an algorithm, define what you count and per whom, respond correctly to clients, and combine throttling with other defenses. Good rate limiting is fair to legitimate users, cheap to evaluate, and correct under concurrency and distribution.

## When to use this skill

- Exposing a public or partner API that needs quotas or protection.
- Defending login/signup/reset/OTP endpoints from brute force and enumeration.
- Guarding expensive operations (search, export, LLM calls, uploads) from overload or cost blowups.
- Mitigating scraping, spam, or a single client degrading service for others.
- Reviewing or fixing an existing rate-limiting design.

## Instructions

1. **Define the objective per endpoint.** Distinguish protecting availability (fair capacity sharing) from stopping abuse (brute force, cost). These want different limits and responses.
2. **Choose the identity/key carefully.** Options: authenticated user/API key (preferred — hard to rotate), IP or subnet (weak: NAT/proxies share IPs, attackers rotate), or a composite. For login, key on both account *and* source IP so one attacker can't lock out a victim and a botnet can't slip under a per-IP limit.
3. **Pick an algorithm** matched to the need:
   - **Token bucket** — allows bursts up to a cap, refills at a steady rate. Good general-purpose choice.
   - **Sliding window (log or counter)** — smooth, accurate limits without fixed-window edge bursts.
   - **Fixed window** — simplest but allows 2x bursts at boundaries; use only when approximate is fine.
   - **Concurrency limits** — cap simultaneous in-flight requests for expensive work.
4. **Make it correct when distributed.** Use a shared store (e.g. Redis) with atomic operations so limits hold across instances; avoid per-node counters that multiply the real limit.
5. **Respond properly.** Return HTTP `429 Too Many Requests` with `Retry-After`, and expose `RateLimit-Limit`/`RateLimit-Remaining`/`RateLimit-Reset` headers so good clients can back off. Fail open or closed deliberately if the limiter store is down.
6. **Layer defenses for abuse.** Combine limits with exponential backoff/lockout on repeated auth failures, CAPTCHA or proof-of-work on suspicious traffic, allow/deny lists, and monitoring/alerting on limit breaches. Rate limiting alone is not a full anti-abuse solution.
7. **Tune and observe.** Set limits from real traffic percentiles, provide burst headroom, tier by plan if needed, and monitor 429 rates to catch both attacks and limits that are too tight.

## Examples

Token-bucket check backed by an atomic Redis operation:

```
# Each key holds tokens; refill = rate * elapsed, capped at burst.
allowed = redis.eval(TOKEN_BUCKET_LUA, keys=[f"rl:{user_id}"],
                     args=[rate_per_sec, burst, now, cost=1])
if not allowed:
    return Response(status=429, headers={"Retry-After": "1"})
```

Login endpoint keyed on account + IP to resist both targeted and distributed brute force:

```
key_ip      = f"login:ip:{client_ip}"       # e.g. 20 / 15 min
key_account = f"login:acct:{email}"         # e.g. 5 failures / 15 min -> slow down / CAPTCHA
# Trip on whichever limit is hit first; add exponential backoff on repeated failures.
```

Standard response headers for a throttled client:

```
HTTP/1.1 429 Too Many Requests
Retry-After: 30
RateLimit-Limit: 100
RateLimit-Remaining: 0
RateLimit-Reset: 30
```

## Checklist

- [ ] Objective per endpoint defined (availability vs. abuse) with sensible limits.
- [ ] Limit key chosen deliberately; auth/login keyed on both account and IP.
- [ ] Algorithm fits the need (token bucket / sliding window / concurrency).
- [ ] Limits enforced via a shared atomic store so they hold across instances.
- [ ] Throttled requests return 429 with Retry-After and RateLimit-* headers.
- [ ] Store-failure behavior (fail open/closed) is chosen intentionally.
- [ ] Abuse defenses layered: backoff/lockout, CAPTCHA, allow/deny lists.
- [ ] Limits tuned from real traffic and 429 rates are monitored/alerted.
