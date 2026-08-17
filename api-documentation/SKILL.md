---
name: api-documentation
description: Document an API so consumers can use it correctly — covering endpoints, HTTP methods, auth, request parameters and bodies, response schemas, status codes, error formats, rate limits, and copy-pasteable examples — use when writing or improving API reference docs, documenting a new endpoint, drafting an OpenAPI-style spec, or when a user asks how to document an API.
---

# API Documentation

Good API documentation lets a developer integrate without reading your source code. For every operation it answers: how do I call it, what do I send, what do I get back, how do I authenticate, and what happens when it fails. Consistency and runnable examples matter more than prose.

## When to use this skill

- Documenting a new endpoint, resource, or entire API surface.
- Improving sparse or inconsistent API reference docs.
- Writing or reviewing an OpenAPI/Swagger spec, or a Markdown reference.
- A user asks how to document endpoints, parameters, errors, or examples.

## Instructions

1. **Start with orientation.** Document the base URL, supported versions/versioning scheme, content type (e.g. `application/json`), and how authentication works (API key header, OAuth bearer token) with a concrete example.
2. **List endpoints consistently.** For each operation give: HTTP method + path, a one-line description, and whether auth is required.
3. **Document request inputs** in tables:
   - Path parameters, query parameters, and headers — each with name, type, required/optional, default, and description.
   - Request body — the JSON schema with field types, constraints, and which are required.
4. **Document responses.** Show the success status code and a full example response body with field descriptions. Note pagination and any envelope/metadata structure.
5. **Document errors uniformly.** List possible status codes (400/401/403/404/409/422/429/500…), the error response shape (e.g. `{ "error": { "code", "message" } }`), and what each condition means and how to fix it.
6. **Provide copy-pasteable examples** for each endpoint — a `curl` request and at least one language snippet — showing real headers and a real (redacted) body.
7. **Note operational limits.** Rate limits and their headers, timeouts, idempotency, and any side effects.
8. **Keep it in sync.** Prefer generating reference from an OpenAPI spec or annotations so docs cannot drift from the implementation; hand-write only the guides and examples.

## Examples

```markdown
## Create order

`POST /v1/orders` — Creates an order. **Auth required.**

### Request headers
| Header          | Value               | Required |
|-----------------|---------------------|----------|
| Authorization   | `Bearer <token>`    | yes      |
| Content-Type    | `application/json`  | yes      |

### Request body
| Field       | Type    | Required | Description                     |
|-------------|---------|----------|---------------------------------|
| `productId` | string  | yes      | ID of the product to order.     |
| `quantity`  | integer | yes      | Must be >= 1.                   |
| `note`      | string  | no       | Optional customer note.         |

### Example request
```bash
curl -X POST https://api.example.com/v1/orders \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"productId": "prod_123", "quantity": 2}'
```

### Success — `201 Created`
```json
{ "id": "ord_789", "status": "pending", "quantity": 2 }
```

### Errors
| Status | code               | Meaning                                   |
|--------|--------------------|-------------------------------------------|
| 400    | `invalid_quantity` | `quantity` was missing or < 1.            |
| 401    | `unauthorized`     | Missing or invalid bearer token.          |
| 404    | `product_not_found`| `productId` does not exist.               |
| 429    | `rate_limited`     | Too many requests; see `Retry-After`.     |

```json
{ "error": { "code": "invalid_quantity", "message": "quantity must be >= 1" } }
```
```

## Checklist

- [ ] Base URL, versioning, content type, and auth are documented up front.
- [ ] Each endpoint lists method, path, description, and auth requirement.
- [ ] Every parameter and body field has type, required flag, and description.
- [ ] Success responses show status code and an example body.
- [ ] Errors use a consistent shape with codes and remediation guidance.
- [ ] Each endpoint has a runnable `curl` (and ideally SDK) example.
- [ ] Rate limits, pagination, and side effects are noted.
- [ ] Reference stays in sync with the implementation (spec-driven if possible).
