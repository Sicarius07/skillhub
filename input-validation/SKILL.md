---
name: input-validation
description: Design and implement robust validation and sanitization of untrusted input at trust boundaries — enforce type, range, format, and size with allow-lists, reject or normalize bad data, and encode for the destination context — use when building or reviewing API endpoints, form handlers, file uploads, message consumers, or anywhere external data enters the system.
---

# Input Validation

Most injection and data-integrity bugs trace back to trusting input that should not be trusted. This skill establishes a consistent, allow-list-first approach: validate structure and content as early as possible, reject what does not conform, normalize what you keep, and encode for output at the point of use. Validation is for correctness and safety; encoding is context-specific and separate.

## When to use this skill

- Building or reviewing an HTTP/RPC endpoint, form handler, or webhook.
- Accepting file uploads, query parameters, headers, or request bodies.
- Consuming messages from a queue or data from a third-party API.
- Any place external, user-controlled, or cross-service data enters.

## Instructions

1. **Identify the trust boundary and every field.** List each input, its source, and its intended type, format, range, and required/optional status. Anything crossing from outside is untrusted — including data from other internal services if they can be influenced by users.
2. **Prefer allow-lists over deny-lists.** Define what is *valid* (an enum, a numeric range, a strict regex, a known set of MIME types) rather than trying to enumerate everything bad. Deny-lists are routinely bypassed.
3. **Validate with a schema at the edge.** Use a declarative validator (JSON Schema, zod, Pydantic, Joi, bean validation) to enforce types, bounds, lengths, patterns, and structure in one place before business logic runs.
4. **Enforce limits.** Cap string lengths, array sizes, numeric ranges, request/body size, and upload size to prevent resource exhaustion and overflow.
5. **Normalize/canonicalize before comparing or storing.** Apply Unicode normalization, trim, lowercase where appropriate, and canonicalize paths/URLs — then re-validate. Do this consistently to avoid bypasses.
6. **Fail closed.** On invalid input, reject with a clear, non-leaky error (HTTP 400/422) and do not partially process. Never "fix up" security-relevant input silently.
7. **Encode for the destination, separately.** Validation does not replace context-aware output encoding: HTML-encode for HTML, use parameterized queries for SQL, escape for shell, encode for URLs. Match the encoding to where the value is used.
8. **Test the boundaries.** Add tests for valid, invalid, empty, oversized, wrong-type, boundary, and adversarial inputs (unexpected Unicode, nested structures, injection payloads).

## Examples

Schema-first validation at an API edge (rejects before business logic):

```ts
import { z } from "zod";

const CreateUser = z.object({
  email: z.string().email().max(254),
  age: z.number().int().min(13).max(120),
  role: z.enum(["viewer", "editor"]),          // allow-list, not free text
  bio: z.string().max(500).optional(),
});

const parsed = CreateUser.safeParse(req.body);
if (!parsed.success) return res.status(422).json({ error: parsed.error.issues });
const user = parsed.data; // typed, validated, safe to use
```

Validation is not encoding — both are needed:

```python
# Validate: is this an allowed value at all?
if category not in ALLOWED_CATEGORIES:
    abort(422)

# Encode at the sink: parameterized query prevents SQL injection regardless
cur.execute("SELECT * FROM items WHERE category = %s", (category,))
```

## Checklist

- [ ] Every untrusted input and its expected type/format/range is enumerated.
- [ ] Validation uses allow-lists, not deny-lists.
- [ ] A schema validator enforces structure at the trust boundary, before logic.
- [ ] Length, size, and range limits cap resource use and overflow.
- [ ] Inputs are normalized/canonicalized, then re-validated.
- [ ] Invalid input fails closed with a clear, non-leaky error.
- [ ] Output is context-encoded at each sink (HTML/SQL/shell/URL) separately.
- [ ] Tests cover valid, invalid, empty, oversized, and adversarial cases.
