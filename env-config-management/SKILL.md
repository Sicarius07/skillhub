---
name: env-config-management
description: Manage application configuration and secrets across environments (local, staging, production) using environment variables, validated schemas, a documented .env.example, a secrets manager, and clear separation of config from code; use when setting up config for a new service, handling secrets safely, wiring multi-environment deploys, fixing "works locally but not in prod" config drift, or when users mention env vars, secrets, .env, config, or twelve-factor.
---

# Environment Config Management

This skill provides a disciplined approach to application configuration following twelve-factor principles: keep config in the environment, separate it from code, validate it at startup, and manage secrets through a dedicated system rather than committing them.

## When to use this skill

- Setting up configuration for a new service or app.
- Handling API keys, database URLs, or other secrets safely.
- Supporting multiple environments (local, CI, staging, production).
- Debugging config drift ("works on my machine", missing var in prod).
- Users mention env vars, `.env`, secrets, config, or twelve-factor.

## Instructions

1. **Separate config from code.** Anything that varies by environment (URLs, credentials, feature toggles, resource limits) belongs in the environment, not in source. Code should be identical across environments.
2. **Read config from environment variables.** Load from the process environment; in local dev, populate it from a `.env` file that is git-ignored. Never hard-code secrets.
3. **Provide a committed `.env.example`.** List every variable with a placeholder/dummy value and a comment describing it. This documents the contract and onboards new developers.
4. **Validate config at startup.** Parse and validate all variables against a schema (types, required, allowed values) when the app boots, and fail fast with a clear error if something is missing or malformed.
5. **Centralize access.** Load and validate config once into a typed config object/module; import that everywhere instead of reading `process.env` scattered throughout the codebase.
6. **Manage secrets in a secrets manager.** Use a dedicated store (Vault, AWS/GCP Secrets Manager, SSM, Doppler, cloud CI secrets) for production secrets. Inject them at deploy/runtime, not in images or repos.
7. **Set safe defaults, but not for secrets.** Provide defaults for non-sensitive tuning values; require secrets and environment-specific URLs explicitly so a missing value never silently falls back to a wrong environment.
8. **Keep environments consistent.** Use the same variable names across environments; only the values differ. Document which vars are required where.
9. **Rotate and least-privilege secrets.** Support rotation, scope credentials narrowly, and audit access. Treat any leaked secret as compromised and rotate immediately.
10. **Prevent leaks.** Add `.env*` (except `.env.example`) to `.gitignore`, run secret scanning in CI, and never log secret values.

## Examples

Documented `.env.example`:

```bash
# Server
PORT=3000                       # HTTP port
NODE_ENV=development            # development | production | test

# Database (required, no default)
DATABASE_URL=postgres://user:pass@localhost:5432/app

# Third-party (required in prod)
STRIPE_SECRET_KEY=sk_test_xxx   # from Stripe dashboard
```

Validate at startup and export a typed config (TypeScript + zod):

```ts
import { z } from 'zod';

const schema = z.object({
  PORT: z.coerce.number().default(3000),
  NODE_ENV: z.enum(['development', 'production', 'test']),
  DATABASE_URL: z.string().url(),
  STRIPE_SECRET_KEY: z.string().min(1),
});

const parsed = schema.safeParse(process.env);
if (!parsed.success) {
  console.error('Invalid configuration:', parsed.error.flatten().fieldErrors);
  process.exit(1);
}
export const config = parsed.data; // import this everywhere
```

Gitignore rules:

```
.env
.env.*
!.env.example
```

## Checklist

- [ ] No secrets or environment-specific values hard-coded in source.
- [ ] Config read from environment variables via a single module.
- [ ] `.env.example` committed and documents every variable.
- [ ] Config validated against a schema at startup; fails fast.
- [ ] Production secrets stored in a secrets manager, injected at runtime.
- [ ] Non-sensitive defaults provided; secrets required explicitly.
- [ ] Variable names consistent across all environments.
- [ ] Secret rotation and least-privilege access in place.
- [ ] `.env*` git-ignored; secret scanning enabled; secrets never logged.
