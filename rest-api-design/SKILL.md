---
name: rest-api-design
description: Design clean, consistent, and predictable RESTful HTTP APIs — resource naming, HTTP methods and status codes, pagination, filtering, versioning, error formats, and idempotency; use when creating a new API, reviewing an endpoint design, standardizing conventions across a team, or fixing an inconsistent or hard-to-consume API.
---

# REST API Design

A well-designed REST API is predictable: once a consumer learns a few endpoints, they can guess the rest. This skill provides conventions for modeling resources, choosing methods and status codes, handling collections, versioning, and reporting errors so that APIs stay consistent and easy to evolve.

## When to use this skill

- Designing a new HTTP API or a new set of endpoints.
- Reviewing API design in a PR or design doc.
- Establishing or documenting team-wide API conventions.
- An existing API is inconsistent, awkward to consume, or hard to version.

## Instructions

1. **Model resources as nouns.** Use plural nouns for collections (`/orders`, `/customers`). Avoid verbs in paths — the HTTP method is the verb.
2. **Use HTTP methods correctly.** `GET` (read, safe), `POST` (create/append), `PUT` (full replace, idempotent), `PATCH` (partial update), `DELETE` (remove, idempotent).
3. **Nest for relationships, shallowly.** `/customers/{id}/orders` to list a customer's orders. Avoid nesting more than one level deep; link by ID instead.
4. **Return correct status codes.** `200` OK, `201` Created (with `Location` header), `204` No Content, `400` bad request, `401` unauthenticated, `403` forbidden, `404` not found, `409` conflict, `422` validation error, `429` rate limited, `5xx` server errors.
5. **Standardize errors.** Return a consistent JSON error shape with a machine-readable code, human-readable message, and optional field-level details. Never leak stack traces.
6. **Paginate all collections.** Prefer cursor-based pagination for large or changing datasets; offset/limit is acceptable for small stable ones. Always return pagination metadata.
7. **Support filtering, sorting, sparse fields via query params.** e.g. `?status=paid&sort=-created_at&fields=id,total`.
8. **Version deliberately.** Put the major version in the path (`/v1/...`) or a header. Only bump for breaking changes; add fields additively otherwise.
9. **Make writes idempotent where possible.** Support an `Idempotency-Key` header on `POST` so retries don't double-create.
10. **Be consistent with formats.** Use `snake_case` or `camelCase` everywhere (pick one), ISO-8601 UTC timestamps, and return IDs as strings.
11. **Document with examples.** Provide request/response samples and an OpenAPI/schema definition.

## Examples

Creating and reading a resource:

```
POST /v1/orders
Idempotency-Key: 8f3c-1a...
Content-Type: application/json

{ "customer_id": "cus_123", "items": [{ "sku": "ABC", "qty": 2 }] }

201 Created
Location: /v1/orders/ord_789
{ "id": "ord_789", "status": "placed", "total": "40.00", "created_at": "2026-08-17T10:00:00Z" }
```

Consistent error response:

```
422 Unprocessable Entity
{
  "error": {
    "code": "validation_failed",
    "message": "One or more fields are invalid.",
    "details": [
      { "field": "items[0].qty", "issue": "must be greater than 0" }
    ]
  }
}
```

Paginated collection:

```
GET /v1/orders?status=paid&limit=20&cursor=eyJpZCI6...

200 OK
{
  "data": [ /* ... */ ],
  "pagination": { "next_cursor": "eyJpZCI6...", "has_more": true }
}
```

## Checklist

- [ ] Paths are plural nouns; verbs live in HTTP methods.
- [ ] Methods and status codes follow their semantics.
- [ ] A single, documented error format is used everywhere.
- [ ] Every collection endpoint paginates and states its metadata.
- [ ] Filtering/sorting conventions are consistent across endpoints.
- [ ] Breaking changes are gated behind a version bump; new fields are additive.
- [ ] Unsafe writes accept an idempotency key.
- [ ] Naming, casing, and timestamp formats are uniform.
- [ ] An OpenAPI/schema doc with examples exists.
