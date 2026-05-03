# API Design

> A practical guide to designing APIs that are durable, evolvable, performant, secure, and pleasant to use.

---

## Table of Contents

1. [What is API Design?](#what-is-api-design)
2. [Why It Matters](#why-it-matters)
3. [The API Design Mindset](#the-api-design-mindset)
4. [Choosing an API Style](#choosing-an-api-style)
5. [REST — Done Properly](#rest--done-properly)
6. [HTTP Methods, Idempotency, Safety](#http-methods-idempotency-safety)
7. [Status Codes That Mean Something](#status-codes-that-mean-something)
8. [URL & Resource Design](#url--resource-design)
9. [Request & Response Design](#request--response-design)
10. [Pagination](#pagination)
11. [Filtering, Sorting, Sparse Fieldsets, Expansion](#filtering-sorting-sparse-fieldsets-expansion)
12. [Versioning](#versioning)
13. [Error Handling](#error-handling)
14. [Idempotency Keys](#idempotency-keys)
15. [Caching](#caching)
16. [Rate Limiting](#rate-limiting)
17. [Authentication & Authorization](#authentication--authorization)
18. [GraphQL](#graphql)
19. [gRPC](#grpc)
20. [tRPC & End-to-End Type Safety](#trpc--end-to-end-type-safety)
21. [Real-Time: WebSockets, SSE, Webhooks](#real-time-websockets-sse-webhooks)
22. [Long-Running Operations](#long-running-operations)
23. [Webhooks Done Right](#webhooks-done-right)
24. [API Security (Top 10)](#api-security-top-10)
25. [API Gateway & BFF Patterns](#api-gateway--bff-patterns)
26. [Documentation, OpenAPI & SDKs](#documentation-openapi--sdks)
27. [Performance & Scalability](#performance--scalability)
28. [Backward Compatibility & Deprecation](#backward-compatibility--deprecation)
29. [API Lifecycle & Governance](#api-lifecycle--governance)
30. [Common Anti-Patterns](#common-anti-patterns)
31. [Quick Reference Checklist](#quick-reference-checklist)
32. [Further Reading](#further-reading)

---

## What is API Design?

An **API (Application Programming Interface)** is the contract between two systems. **API design** is the discipline of shaping that contract so it's:

- **Predictable** — consumers can guess how things work without re-reading the docs
- **Stable** — you can evolve it without breaking everyone downstream
- **Performant** — efficient over the network and at the database
- **Secure** — protects data, authn/authz, rate limits, audit trails
- **Discoverable** — documented, versioned, browsable
- **Pleasant** — the things you do most often are the easiest

> **A reframe worth keeping:** an API is a **product** even if you never sell it. Internal APIs have users (other engineers, future-you), and ergonomic decisions compound. Treat API design with the same intentionality as UX design.

---

## Why It Matters

### 1. APIs are the longest-lived part of any system
You'll rewrite the database, swap the framework, redo the UI — but the API contract is what your customers, mobile apps, integrations, and partners have wired into their code. **Breaking it is breaking them.**

### 2. The cost of bad API decisions compounds
A clumsy URL, an inconsistent error shape, a missing pagination token — every consumer works around it. Years later, you can't fix it without breaking thousands of downstream integrations. **Get it right early.**

### 3. APIs are the integration surface
In a world of microservices, mobile, web, partner integrations, AI agents, and webhooks, your API is how the company is *wired together*. A coherent API is a coherent business.

### 4. Developer experience drives adoption
For public APIs (Stripe, Twilio, GitHub, OpenAI), excellent design is competitive moat. Engineers pick services partly on how easy the API feels in the first 30 minutes.

### 5. AI agents are now first-class consumers
LLMs read your OpenAPI schemas, call your endpoints via tool use, and generate integration code. **APIs designed for humans tend to also work well for agents** — clear names, consistent shapes, semantic errors. APIs designed badly fail both.

---

## The API Design Mindset

### 1. Design from the outside in
Start by writing the *example calls* you wish your users could make. Then build the API to satisfy them. This is "API-first" or "schema-first" — the inverse of "we'll add an API to the database later."

### 2. Be conservative in what you accept, generous in what you provide
Postel's Law (the robustness principle), with a caveat: **be strict on the way in** (validate, reject malformed requests, demand explicit shapes), **predictable on the way out** (consistent format, no surprises, additive change).

### 3. Make breaking changes expensive
Every breaking change costs every consumer. Design defaults that **make the right thing additive**: optional fields, new endpoints, new versions — not changing the meaning of old ones.

### 4. Be explicit, not magical
Magic ("we'll figure out what you meant") is great until it isn't. Explicit defaults, explicit pagination, explicit fields, explicit errors. Good APIs explain themselves.

### 5. Optimize for reading
APIs are read 10,000× for every time they're written. The names, the shapes, the error messages, the docs — all optimize for the reader years from now.

### 6. The API is the product, the database is not
Don't expose your DB schema. Internal table changes shouldn't break external clients. Design the API around *use cases*, not schemas.

### 7. Backwards compatibility is non-negotiable
For public APIs, treat compatibility as a hard constraint. For internal APIs, treat it as the default and document the exceptions. **Breaking changes erode trust faster than slow features.**

---

## Choosing an API Style

The 2026 landscape:

| Style | Use case | Pros | Cons |
|---|---|---|---|
| **REST (HTTP+JSON)** | Public APIs, CRUD, broad compatibility | Universal, cacheable, debuggable, web-native | Verbose for graph data, no schema by default, over/under-fetching |
| **GraphQL** | Many heterogeneous clients, complex relations | Single endpoint, client-driven shape, strong typing, introspection | Server complexity (N+1, depth limits, caching), auth at field level |
| **gRPC** | Internal service-to-service, polyglot, perf-critical | Binary, streaming, strong contracts (Protobuf), fast | Browser support requires gRPC-Web proxy; less human-debuggable |
| **tRPC** | TS-monorepo, full-stack apps with one team | End-to-end type safety, no codegen, instant feedback | TS-only, tightly couples client and server |
| **WebSocket / SSE** | Real-time, push, live updates | Persistent connections, low latency | Not request/response; harder to load-balance and cache |
| **Webhooks** | Event-driven integrations with third parties | Simple, inverted (server pushes to consumer) | Retries, idempotency, security each integration's problem |
| **Async messaging (Kafka, SQS)** | Decoupled fan-out across services | Resilient, scalable, replay | Eventual consistency; not request/response |

### The pragmatic 2026 default
- **Public-facing API** → **REST + OpenAPI** (broadest compat) or **GraphQL** (if many client shapes; you have the team to operate it)
- **Internal microservice ↔ microservice** → **gRPC** (perf, contracts) or **REST** (simplicity)
- **Same-team monorepo (BFF + frontend)** → **tRPC** or REST with shared TS types
- **Real-time** → **WebSocket** for bidi, **SSE** for one-way push, **Webhooks** for cross-org events

> **In practice:** the styles are not religions. Most large products use **all of them at once** — REST for public, gRPC internal, GraphQL for one product surface, webhooks for integrations, WebSockets for live updates. Pick per use case.

---

## REST — Done Properly

REST (Roy Fielding's 2000 dissertation) is **resource-oriented**, stateless, cacheable, and uses HTTP semantics. Most "REST APIs" in the wild are really HTTP+JSON, missing key REST traits (HATEOAS, hypermedia). That's fine — be honest about what you're shipping.

### The Richardson Maturity Model
Levels of "REST-ness":
- **L0** — One URL, POST everything (RPC over HTTP)
- **L1** — Multiple resources (`/users`, `/orders`)
- **L2** — Use HTTP verbs and status codes properly
- **L3** — HATEOAS (responses include links to next actions)

> Most modern APIs sit at **L2**. L3 (HATEOAS) is rarely seen outside specific domains (Spring HAL, FHIR for healthcare). Don't agonize about reaching L3 unless you have a clear reason.

### Resource thinking
Frame your API around **nouns (things)**, not verbs (actions):

```
✅  GET    /orders
✅  POST   /orders
✅  GET    /orders/123
✅  PATCH  /orders/123
✅  DELETE /orders/123

❌  POST /createOrder
❌  POST /getOrderById
❌  POST /updateOrder
```

When an action doesn't fit a noun, model it as a **sub-resource** or use a **verb endpoint** sparingly:
```
POST /orders/123/cancellations    ← treat the cancellation as a thing
POST /orders/123:cancel           ← Google AIP convention; explicit verb
```

### Stateless requests
Every request carries everything needed to process it (auth token, params). The server doesn't store session state in memory — enables horizontal scaling, simpler failover, easier debugging.

### Consistent shapes
Every list response uses the same envelope; every error uses the same shape; every timestamp uses ISO 8601 UTC. **Predictable structures are the API's most important UX.**

---

## HTTP Methods, Idempotency, Safety

### The verbs and their guarantees

| Method | Safe? | Idempotent? | Cacheable? | Use for |
|---|---|---|---|---|
| **GET** | ✅ | ✅ | ✅ | Read a resource |
| **HEAD** | ✅ | ✅ | ✅ | Like GET, no body |
| **OPTIONS** | ✅ | ✅ | – | Preflight; capabilities |
| **POST** | ❌ | ❌ (usually) | sometimes | Create; non-idempotent action |
| **PUT** | ❌ | ✅ | ❌ | Replace whole resource |
| **PATCH** | ❌ | ❌ (usually) | ❌ | Partial update |
| **DELETE** | ❌ | ✅ | ❌ | Remove |

- **Safe** — does not modify state (purely read).
- **Idempotent** — calling N times has the same effect as calling once. *Critical for retries.*

### POST vs PUT vs PATCH
- **POST `/orders`** — server assigns ID; not idempotent (each call creates new). Use **idempotency keys** for at-most-once semantics.
- **PUT `/orders/123`** — full replacement; client knows the ID. Idempotent.
- **PATCH `/orders/123`** — partial update; semantics depend on body format (JSON Merge Patch RFC 7396 vs JSON Patch RFC 6902).

### JSON Merge Patch (the easy one)
```http
PATCH /users/42
Content-Type: application/merge-patch+json

{ "email": "new@example.com", "phone": null }
```
`null` deletes the field; missing fields are unchanged.

### JSON Patch (the explicit one)
```http
PATCH /users/42
Content-Type: application/json-patch+json

[
  { "op": "replace", "path": "/email", "value": "new@example.com" },
  { "op": "remove",  "path": "/phone" }
]
```

Pick one and document it. Most APIs choose Merge Patch for simplicity.

---

## Status Codes That Mean Something

Use HTTP status codes as the first-line indicator of outcome.

### 2xx — Success
| Code | Meaning |
|---|---|
| **200 OK** | Generic success with body |
| **201 Created** | Resource created (return it in body or `Location` header) |
| **202 Accepted** | Async work started; check back later |
| **204 No Content** | Success, no body (after DELETE, sometimes PUT) |
| **206 Partial Content** | Range request fulfilled |

### 3xx — Redirection (rarely in APIs)
| Code | Meaning |
|---|---|
| **301 Moved Permanently** | Resource permanently moved |
| **304 Not Modified** | Conditional GET; client cache is fresh |

### 4xx — Client errors
| Code | Meaning |
|---|---|
| **400 Bad Request** | Malformed request (bad JSON, missing required) |
| **401 Unauthorized** | No or invalid credentials (poorly named — actually means *unauthenticated*) |
| **403 Forbidden** | Authenticated but not allowed |
| **404 Not Found** | Resource doesn't exist (or you don't have rights to see it) |
| **405 Method Not Allowed** | The method doesn't apply to this resource |
| **406 Not Acceptable** | Content negotiation failed |
| **409 Conflict** | State conflict (version mismatch, unique constraint) |
| **410 Gone** | Permanently removed (vs 404 = "we don't know") |
| **412 Precondition Failed** | `If-Match` / `If-None-Match` failed |
| **415 Unsupported Media Type** | Bad `Content-Type` |
| **422 Unprocessable Entity** | Syntactically valid but semantically wrong (validation failed) |
| **429 Too Many Requests** | Rate-limited (include `Retry-After`) |

### 5xx — Server errors
| Code | Meaning |
|---|---|
| **500 Internal Server Error** | Unhandled exception — generic |
| **501 Not Implemented** | Method not implemented yet |
| **502 Bad Gateway** | Upstream returned an invalid response |
| **503 Service Unavailable** | Temporarily down (often with `Retry-After`) |
| **504 Gateway Timeout** | Upstream timed out |

### Nuances worth knowing
- **400 vs 422** — both are validation failures, but `400` is for malformed (couldn't parse), `422` for parsed-but-invalid. Pick one convention; document it.
- **401 vs 403** — `401` if missing/invalid auth; `403` if auth is fine but the user lacks permission. Don't return `401` to a user with valid creds.
- **404 vs 403 (information leak)** — sometimes you return `404` instead of `403` to avoid revealing that a resource exists. Document the convention.
- **5xx is your fault** — never use for client problems. If you're tempted to return `500` for a bad request, use `4xx`.
- **Don't invent codes** — `499`, `420` are non-standard or specific (Cloudflare, Twitter rate limit). Stick with the IANA registry.

---

## URL & Resource Design

### Naming
- **Plural nouns for collections**: `/users`, `/orders`, `/products`
- **Lowercase, hyphenated**: `/order-items`, `/payment-methods`
- **No file extensions**: `/users` not `/users.json` (use `Accept` header)
- **No verbs in URLs** (mostly)
- **Predictable nesting**: `/users/42/orders` for orders belonging to user 42

### Hierarchies
```
GET /users                       → list users
GET /users/42                    → get user 42
GET /users/42/orders             → list orders for user 42
GET /users/42/orders/9001        → get order 9001 (scoped to user)
```

Or shallow with filters:
```
GET /orders?user_id=42           → list orders, filtered
GET /orders/9001                 → flat addressing
```

> **A heuristic that holds up:** flat URLs with filters scale better than deep nesting. Reserve nesting for true containment relationships ("an order belongs to one user"); use filters for everything else.

### IDs
- **Don't expose database integer IDs** for resources visible to other tenants. Predictable, enumerable IDs enable IDOR attacks (`/orders/9000`, `/orders/9001`...).
- **Use opaque IDs** — UUIDs, ULIDs, or prefixed identifiers (Stripe-style: `cus_abc123`, `ch_xyz789`). Prefixes are self-documenting and let you validate types at the boundary.
- **ULIDs** (lexicographically sortable, time-prefixed) and **UUIDv7** (similar) are excellent modern choices.

### Query parameters
```
GET /orders?status=pending&created_after=2026-01-01&sort=-created_at&limit=50
```
- **Filter**: `?status=pending`, `?created_after=...`
- **Sort**: `?sort=created_at` (asc) / `?sort=-created_at` (desc; minus = descending)
- **Pagination**: `?limit=50&cursor=...`
- **Field selection**: `?fields=id,name,email`
- **Expansion**: `?expand=customer,items` (include related)

### `:` for actions on resources (Google AIP)
```
POST /orders/123:cancel
POST /messages:batchSend
```
Action is unambiguous; doesn't conflict with sub-resources.

### Versioning in the URL (the practical default)
```
/v1/orders
/v2/orders
```
See [Versioning](#versioning) for tradeoffs.

---

## Request & Response Design

### Content negotiation
```
Accept: application/json
Content-Type: application/json
```
JSON is the default; support gzip/Brotli compression.

### Consistent response shapes
Pick a convention. Two common ones:

**Plain (the resource at the top level)**:
```json
{ "id": "ord_123", "status": "paid", "total": 9999 }
```

**Envelope (with metadata)**:
```json
{
  "data": { "id": "ord_123", "status": "paid", "total": 9999 },
  "meta": { "request_id": "req_abc" }
}
```

For lists, *always* use an envelope so you have somewhere to put pagination metadata:
```json
{
  "data": [{ ... }, { ... }],
  "page": { "next_cursor": "abc...", "has_more": true },
  "meta": { "request_id": "req_abc" }
}
```

### Naming conventions
- **Snake_case** vs **camelCase** — pick one and never mix. JSON in the JS world tilts toward camelCase; broader public APIs (Stripe, GitHub) often use snake_case. **What matters is consistency, not the choice.**
- **Booleans named `is_x` / `has_x` / `can_x`** — predictable and self-explanatory
- **Timestamps as ISO 8601 with timezone**: `"2026-04-30T14:30:00Z"` (UTC). Never Unix epoch unless absolutely required for size.
- **Money** — use **integers in minor units** (cents) + currency code; never floats. Floating point + money = bugs.
  ```json
  { "amount": 1099, "currency": "USD" }   // $10.99
  ```
- **Enums** — strings with stable values: `"status": "pending"` — never magic numbers
- **IDs** — strings, even if they're numeric internally (avoids JS bigint issues)

### Headers
Common request headers:
```
Authorization: Bearer eyJ...
Content-Type: application/json
Accept: application/json
Accept-Encoding: br, gzip
X-Request-Id: req_abc123        ← client-generated; echoed in response & logs
Idempotency-Key: 7f4a...        ← for safe retries on POST
If-Match: "etag-v3"             ← optimistic locking on PUT/PATCH/DELETE
```

Common response headers:
```
Content-Type: application/json
ETag: "v4"
Cache-Control: private, max-age=60
X-Request-Id: req_abc123
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 875
X-RateLimit-Reset: 1735689600
```

---

## Pagination

Three approaches; pick deliberately.

### 1. Offset / Limit
```
GET /orders?offset=100&limit=20
```
**Pros:** simple, jump to any page.
**Cons:** **slow at depth** (DB scans the offset); inconsistent if list changes during paging (skip / duplicate items).
**Use when:** small datasets; admin UIs where consistency isn't critical.

### 2. Cursor (opaque token)
```
GET /orders?limit=20
Response: { "data": [...], "next_cursor": "eyJpZCI6IjEyMyJ9" }

GET /orders?limit=20&cursor=eyJpZCI6IjEyMyJ9
```
**Pros:** stable across changes; fast at any depth; pagination tokens are opaque so you can change implementation.
**Cons:** can't jump to arbitrary "page N"; only forward (or with explicit `prev_cursor`).
**Use when:** **the modern default**. Most large APIs (Stripe, GitHub, Slack) use cursors.

### 3. Keyset / seek
```
GET /orders?limit=20&after_id=ord_123&sort=created_at
```
**Pros:** fast; index-friendly.
**Cons:** sort key must be unique + indexed; tricky with multi-column sort.
**Use when:** very large, append-only lists.

### Pagination response shape (recommended)
```json
{
  "data": [...],
  "page": {
    "limit": 20,
    "next_cursor": "abc...",
    "has_more": true
  }
}
```

### Nuances worth knowing
- **Tell consumers there's more** with `has_more` — avoid having to call again to know
- **Never expose the offset** in cursor APIs; it leaks implementation
- **Cap `limit`** server-side (e.g., max 100); reject larger
- **Stable sort key** — without one, identical sort values cause skipped/duplicated items

---

## Filtering, Sorting, Sparse Fieldsets, Expansion

### Filtering
- **Exact match**: `?status=paid`
- **Multiple values**: `?status=paid,pending` (comma-separated)
- **Range**: `?created_after=...&created_before=...`
- **Operators**: `?amount[gte]=100&amount[lte]=500` (bracketed)
- **Full-text**: `?q=keyword`

Document the schema; reject unknown fields. Don't accept arbitrary SQL-like predicates from users (NoSQL injection risk).

### Sorting
```
?sort=created_at         → asc
?sort=-created_at        → desc
?sort=-priority,created_at  → multi-key
```

### Sparse fieldsets
Let clients ask for only what they need:
```
GET /users/42?fields=id,name,email
```
Reduces bandwidth on bandwidth-constrained clients (mobile). Inspired by JSON:API spec.

### Expansion / Inclusion
```
GET /orders/123?expand=customer,items.product
```
Server returns related resources inline. Saves N+1 round-trips on the client. **Cap depth** server-side.

### Field selection vs. GraphQL
At extreme: GraphQL gives every consumer arbitrary shape control. REST with sparse fields is a middle ground. If you find yourself shipping more than ~20 expansion permutations, consider GraphQL.

---

## Versioning

**Plan for v2 from day 1.** Even if you swear you won't break it, you will.

### Strategies
| Strategy | Example | Pros | Cons |
|---|---|---|---|
| **URI versioning** | `/v1/users` | Visible, browser-friendly, easy to route | Doesn't match REST purity ("the resource is the same"); copies routes |
| **Header versioning** | `Accept: application/vnd.example.v1+json` | Pure REST | Less discoverable; harder to test in browser; breaks naive caching |
| **Query parameter** | `/users?version=1` | Simple | Cluttered; hard to enforce |
| **Date-based** | `Stripe-Version: 2025-04-15` | Granular; per-customer pinning | More complex routing logic |

### The pragmatic choice
**URI versioning** for most public APIs. **Date-based pinning** (Stripe model) when you can afford to maintain many concurrent versions for a long time.

### Stripe's model — gold standard
- Every API call has a default version (the version the user signed up with)
- Customers can opt into newer versions explicitly
- Each version is documented; old ones run for years
- Internal "version transformer" pipeline rewrites requests/responses to match the requested version
- Most changes are *additive* and need no version bump

### Compatibility rules
**Backward-compatible (no version bump):**
- Adding new optional fields to responses
- Adding new optional request parameters with sensible defaults
- Adding new endpoints
- Adding new enum values *that clients are told to ignore unknowns*
- Adding new HTTP status codes only in new conditions
- Loosening validation

**Breaking (requires version bump):**
- Removing or renaming fields
- Changing field types or semantics
- Making optional fields required
- Removing endpoints
- Tightening validation
- Changing default values
- Changing error shapes

### A reality check
"Just add `v2`" sounds easy. Maintaining v1 + v2 in parallel for 18 months is **not** easy. Plan deprecation:
- Announce it loudly (changelog, email, in-product warnings)
- Set a sunset date (`Sunset` HTTP header)
- Track usage per version
- Help large customers migrate (sometimes 1:1)

---

## Error Handling

Errors are 50% of the API's surface area. Take them seriously.

### A consistent error shape
The standard is **RFC 9457 Problem Details for HTTP APIs** (formerly RFC 7807):

```json
{
  "type": "https://api.example.com/errors/insufficient-funds",
  "title": "Insufficient funds",
  "status": 422,
  "detail": "The account balance ($45.00) is less than the requested amount ($60.00).",
  "instance": "/transfers/abc123",
  "code": "insufficient_funds",
  "request_id": "req_xyz",
  "fields": {
    "amount": ["must be ≤ available balance"]
  }
}
```

```
Content-Type: application/problem+json
```

### What every error should include
- **HTTP status** — first-line outcome
- **`code`** — machine-readable, stable string (`"insufficient_funds"`)
- **`message`** / `detail` — human-readable explanation, **safe to display**
- **`request_id`** — so users can paste it in support requests
- **Field-level errors** for validation (so the client can highlight the form)

### What errors should NOT include
- Stack traces (security leak)
- Internal IDs / DB queries
- "Object reference not set to an instance of an object." (.NET nightmare)

### Error code design
Codes are forever. Once you ship `"validation_failed"`, you can't change it without breaking clients. Pick a flat namespace; document each code; group categories by prefix:
```
auth.invalid_credentials
auth.token_expired
billing.insufficient_funds
billing.card_declined
validation.required_field
validation.invalid_format
```

### Distinguishing user errors from system errors
| Type | Status | What client should do |
|---|---|---|
| Validation | 4xx | Show user; let them fix and retry |
| Business rule | 4xx | Show user; usually no retry |
| Auth | 401/403 | Re-authenticate or escalate |
| Rate limit | 429 | Back off, retry later |
| Transient server | 503/504 | Retry with backoff |
| Permanent server | 500 | Don't retry; report bug |

Make this clear in the error: *"This is a transient issue, please retry"* vs *"This is a permanent error, please contact support."*

---

## Idempotency Keys

The most important reliability primitive most APIs miss.

### The problem
You POST `/payments` to charge a customer. Network hiccups; you don't get a response. Retry? You might double-charge. Don't retry? Maybe the first one didn't go through. **Lose-lose.**

### The solution
Client generates a unique key per logical operation, sends it as a header:
```
POST /payments
Idempotency-Key: 7f4a8b2e-...
```

Server stores `(key → result)` for some retention window (24h is common). On retry with the same key:
- If a result exists → return it (replay the previous response, including status and body)
- If still in-flight → return `409 Conflict` or wait

### Storage
- DB row keyed by `idempotency_key` + `request_hash`
- Save the response body + status
- Expire after retention window
- Hash the request body and **reject** if the key matches but the body differs (mismatched retry — clearly a bug)

### When to require it
- All non-idempotent state changes (POST that creates something)
- Anything that costs money or sends external messages
- Public APIs where clients have flaky networks

Stripe has an excellent [idempotency design doc](https://stripe.com/docs/api/idempotent_requests) — required reading.

---

## Caching

(See [Web Optimization Techniques](web-optimization-techniques.md) for the broader caching picture.)

### HTTP cache headers
```
Cache-Control: public, max-age=3600
ETag: "v42"
Last-Modified: Wed, 30 Apr 2026 14:30:00 GMT
Vary: Accept-Encoding
```

### Conditional requests (304)
```http
GET /users/42
If-None-Match: "v42"

→ 304 Not Modified  (no body)
```

### Optimistic locking (412)
```http
PATCH /orders/123
If-Match: "v3"
Content-Type: application/merge-patch+json

{ "status": "shipped" }

→ 412 Precondition Failed  (someone else updated to v4)
```

This is how you implement safe concurrent edits without explicit locks.

### `Cache-Control` directives
- `private` — only the user's browser
- `public` — any cache (browser + CDN)
- `no-cache` — cache, but revalidate every time
- `no-store` — never cache
- `max-age=N` — fresh for N seconds
- `stale-while-revalidate=N` — serve stale while refetching in background
- `must-revalidate` — once stale, must check before serving

### What to cache
- **Read-only public data** — public profiles, feature flags, content
- **Per-user data** with `private` + short TTL
- **Authenticated endpoints** generally don't cache at CDN; your in-process cache may
- **Lists** — cache per-query-string; invalidate on write

---

## Rate Limiting

Protect your service from accidental DDoS and abuse.

### Algorithms
| Algorithm | How it works | Tradeoffs |
|---|---|---|
| **Fixed window** | N requests per minute, reset at the boundary | Simple; allows 2N at boundaries |
| **Sliding window log** | Keep timestamps; count those in window | Most accurate; memory-heavy |
| **Sliding window counter** | Approximate sliding window with two fixed buckets | Good balance |
| **Token bucket** | Bucket fills at rate R, capped at C; each request takes a token | **Most popular**; allows burst |
| **Leaky bucket** | Like token bucket but constant outflow | Smooths bursts |

### Headers (standard convention)
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 873
X-RateLimit-Reset: 1735689600

429 Too Many Requests
Retry-After: 30
```

There's a **draft RFC** (`RateLimit` headers, RFC 9234-track) standardizing this. Use it if you want to be future-friendly:
```
RateLimit: limit=1000, remaining=873, reset=30
```

### Tiering
Different limits for:
- Anonymous (lowest)
- Authenticated user (medium)
- Higher tier / paid plan (high)
- Internal service-to-service (very high or unlimited with mTLS)

### What to limit on
- IP (anonymous)
- User ID / API key
- Endpoint (specific limits on expensive operations like search)
- Combination thereof

### Where to enforce
- **API gateway** (Kong, Envoy, Cloudflare, AWS API Gateway) — first line of defense
- **In-process** for fine-grained per-endpoint limits
- **Redis-backed** sliding window for accurate distributed counting

---

## Authentication & Authorization

(See [Web Application Security](web-application-security.md) for the full security picture.)

### Authentication mechanisms

| Mechanism | When |
|---|---|
| **API keys** | Server-to-server; simple; **must be transmitted only over HTTPS, in headers** |
| **Bearer tokens (JWT)** | Stateless; multi-service; short-lived access tokens + long-lived refresh tokens |
| **OAuth 2.0 + OIDC** | Third-party access on behalf of a user; the standard |
| **mTLS** | Service-to-service inside a trusted network |
| **Session cookies** | Browser-only first-party APIs; HttpOnly + Secure + SameSite |
| **HMAC signatures** | Webhooks (provider signs payload; consumer verifies) |

### Where to put credentials
**Always in headers**, never in URLs (URLs are logged):
```
Authorization: Bearer eyJ...
Authorization: Basic <base64>          ← only for HTTPS
X-Api-Key: sk_live_abc...              ← legacy; prefer Authorization
```

Avoid:
- Query parameters (`?api_key=...` — logged in CDN, server, browser history)
- POST body (works but loses HTTP semantic — Authorization header is right)

### Authorization patterns
- **RBAC** — user has roles; roles have permissions
- **ABAC** — attributes (user, resource, action, environment)
- **ReBAC** — relationships (Google Zanzibar; "is Alice a viewer of folder X?")

Centralize authorization. Authorize the **specific resource**, not just the route — see **IDOR / BOLA** in the API security section.

---

## GraphQL

A query language and runtime for APIs. Clients ask for the exact shape they need; servers respond.

### When GraphQL wins
- **Many heterogeneous clients** (web + mobile + partners) needing different shapes
- **Deep relational data** where REST would over-fetch or chain calls
- **Strong typing** as a hard requirement
- **Schema-first development** with multiple teams

### When GraphQL loses
- **Simple CRUD** — REST is less ceremony
- **File uploads / binary** — awkward in GraphQL
- **HTTP caching** — much harder than REST (POST + body-keyed)
- **Public APIs** — broad-compat tooling (curl, Postman) is REST-native

### Schema design
```graphql
type Order {
  id: ID!
  status: OrderStatus!
  total: Money!
  items: [OrderItem!]!
  customer: User!
  createdAt: DateTime!
}

enum OrderStatus { PENDING PAID SHIPPED CANCELLED }

type Query {
  order(id: ID!): Order
  orders(first: Int, after: String, status: OrderStatus): OrderConnection!
}

type Mutation {
  cancelOrder(id: ID!): Order!
}
```

### Cursor connections (Relay spec)
The standard for GraphQL pagination:
```graphql
type OrderConnection {
  edges: [OrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int
}
type OrderEdge { node: Order!, cursor: String! }
type PageInfo { hasNextPage: Boolean!, endCursor: String, ... }
```

### The N+1 problem
```graphql
{ orders { id  customer { name } } }
```
Naive resolver: 1 query for orders, then N queries for customers. Use **DataLoader** (Facebook) — batches and caches per-request:
```js
const loader = new DataLoader(ids => batchGetUsers(ids));
// resolver
customer: (order) => loader.load(order.customerId);
```

### Other concerns
- **Depth limits** — bounded query depth (5–10) to prevent DoS via deep recursion
- **Complexity scoring** — assign costs to fields; reject queries above a budget
- **Field-level authorization** — every field needs auth; centralize via directives
- **Persisted queries** — pre-register queries; allow only known IDs in production (smaller payloads, security)
- **Subscriptions** — GraphQL over WebSocket for real-time
- **Federation (Apollo Federation, Schema Stitching)** — combine multiple GraphQL services into one supergraph; useful at scale

### Tools
- **Apollo Server / Client**, **GraphQL Yoga**, **Mercurius**, **Hasura**
- **Relay** (Facebook) — opinionated client with strong patterns
- **GraphQL Codegen** — generate TS types and hooks from schema
- **Pothos**, **TypeGraphQL** — code-first schema in TypeScript

---

## gRPC

Google's RPC framework over HTTP/2 with Protocol Buffers (Protobuf).

### Strengths
- **Binary, very fast** — small payloads, fast serialization
- **Strong contracts** — `.proto` files generate client and server code in many languages
- **Streaming** — unary, server-streaming, client-streaming, bidi
- **HTTP/2 multiplexing** — many concurrent calls on one connection

### Weaknesses
- **Browser support requires gRPC-Web** — proxy needed; not native
- **Less human-readable** than JSON; debugging requires tooling
- **Versioning is rigid** — Protobuf field numbers are forever; must follow strict rules

### Use cases
- **Internal microservice communication** — perfect fit
- **Polyglot services** — Go, Java, Python, Rust, etc., all compatible from one schema
- **Mobile apps** with battery/bandwidth constraints

### `.proto` design
```protobuf
syntax = "proto3";
package orders.v1;

service OrderService {
  rpc GetOrder(GetOrderRequest) returns (Order);
  rpc StreamOrderUpdates(StreamRequest) returns (stream OrderUpdate);
}

message Order {
  string id = 1;
  string status = 2;
  int64 total_minor = 3;
  string currency = 4;
  // Field numbers are forever; never reuse.
}
```

Field numbers ≤ 15 use 1 byte on the wire; reserve those for hot fields.

### Connect / ConnectRPC
A modern alternative — gRPC-compatible but works natively over HTTP/1.1 + HTTP/2 with JSON or Protobuf. Excellent for browsers without a proxy. Worth considering for new projects.

---

## tRPC & End-to-End Type Safety

For TypeScript monorepos where the same team owns frontend and backend.

```ts
// Server
const appRouter = router({
  user: router({
    getById: publicProcedure
      .input(z.object({ id: z.string() }))
      .query(async ({ input }) => {
        return db.user.findUnique({ where: { id: input.id } });
      }),
  }),
});
export type AppRouter = typeof appRouter;

// Client
const user = await trpc.user.getById.query({ id: '42' });
//      ↑ fully typed from the server
```

### Strengths
- **Zero codegen** — types flow via TypeScript imports
- **Refactor safety** — rename a procedure on the server, the client breaks at compile time
- **Excellent DX** — autocomplete, jump-to-definition

### Weaknesses
- **TS-only** — not for polyglot
- **Tightly couples** client and server (won't work if they're different repos / versions)
- **Public API** is awkward (no language-agnostic schema)

Consider **tRPC for internal**, **OpenAPI/REST or GraphQL for external**.

---

## Real-Time: WebSockets, SSE, Webhooks

### WebSocket
Full-duplex, persistent connection.
```js
const ws = new WebSocket('wss://api.example.com/v1/orders/stream');
ws.onmessage = (e) => console.log(JSON.parse(e.data));
```

**Use:** chat, multiplayer, collaborative editing, live dashboards with bi-directional updates.
**Trade-offs:** load balancing harder, no HTTP semantics, auth is per-connection.

### Server-Sent Events (SSE)
One-way push, plain HTTP, native browser API.
```js
const es = new EventSource('/v1/orders/stream');
es.onmessage = (e) => console.log(JSON.parse(e.data));
```

**Use:** notifications, live feeds, AI streaming responses (very common with LLMs).
**Trade-offs:** server → client only; reconnection logic baked in; works through HTTP proxies.

### Long polling (legacy fallback)
Client requests; server holds open until data is available; client immediately re-requests. Use only when WebSocket / SSE isn't available.

### Choosing
| Need | Pick |
|---|---|
| Bi-directional, low-latency | WebSocket |
| Server → client only | SSE |
| Cross-org event delivery | Webhook |
| LLM streaming responses | SSE |

---

## Long-Running Operations

Some operations take seconds or minutes (export a CSV, generate a video, train a model). Don't make the client wait synchronously.

### The async pattern
```http
POST /exports
→ 202 Accepted
{
  "operation_id": "op_abc123",
  "status": "running",
  "self": "/operations/op_abc123"
}

GET /operations/op_abc123
→ 200 OK
{
  "operation_id": "op_abc123",
  "status": "succeeded",
  "result": { "url": "https://..." }
}
```

### Statuses
`pending → running → succeeded | failed | cancelled`

### Progress
Optionally include `progress: 0.42` and `estimated_completion_at` for UX.

### Cancellation
`POST /operations/op_abc123:cancel` (or `DELETE`).

### Notification on completion
Better than polling: webhook or SSE/WebSocket. Especially for long jobs (>1 min).

### Standards
- **Google AIP-151** Long-running operations — solid reference

---

## Webhooks Done Right

Inverse of REST — your server pushes events to consumers.

### Event design
```json
{
  "id": "evt_abc",
  "type": "order.shipped",
  "created_at": "2026-04-30T14:30:00Z",
  "data": {
    "order_id": "ord_123",
    "tracking_number": "..."
  },
  "api_version": "2026-01-01"
}
```

### Subscription model
- Consumer registers a URL with the events they care about
- Provider sends POST to that URL when events occur
- Consumer responds 2xx for success; non-2xx triggers retries

### Reliability requirements (this is where most webhook systems fail)
1. **At-least-once delivery** — make consumers idempotent (use the event `id` as idempotency key)
2. **Retry with exponential backoff** — common policy: 5s, 15s, 1m, 5m, 30m, 1h, 6h, 24h, then dead-letter
3. **Signed payloads** — HMAC-SHA256 of body + timestamp; consumer verifies
   ```
   X-Signature: t=1735689600,v1=abc...
   ```
4. **Replay protection** — include timestamp; reject events older than ~5 minutes
5. **Dead-letter queue** — surface failures to the consumer dashboard

### Provider-side best practices
- **Async dispatch** — never let webhook delivery block your main flow
- **Per-consumer queue** — one slow consumer can't back up others
- **Tenant isolation** — burst from one consumer doesn't affect others
- **Public IP allowlist support** — let consumers firewall to known sources
- **mTLS option** for high-security integrations

### Signing example
```js
// Provider:
const ts = Date.now();
const body = JSON.stringify(payload);
const sig = hmacSha256(secret, `${ts}.${body}`);
headers['X-Signature'] = `t=${ts},v1=${sig}`;

// Consumer:
const [t, v1] = parseHeader(req.headers['x-signature']);
if (Date.now() - t > 5 * 60 * 1000) reject('too old');
const expected = hmacSha256(secret, `${t}.${rawBody}`);
if (!timingSafeEqual(v1, expected)) reject('bad signature');
```

### Standards
- **Standard Webhooks** (`standardwebhooks.com`) — emerging spec for header conventions, signing
- **CloudEvents** — CNCF spec for event metadata; broader than webhooks but compatible

---

## API Security (Top 10)

OWASP's API Security Top 10 (2023) — the API-specific equivalent of the web Top 10:

| # | Category | Examples |
|---|---|---|
| **API1** | Broken Object Level Authorization (BOLA) | IDOR — `/orders/{id}` without checking ownership |
| **API2** | Broken Authentication | Weak tokens, no rotation, no revocation |
| **API3** | Broken Object Property Level Auth | Mass assignment; updating fields you shouldn't be able to |
| **API4** | Unrestricted Resource Consumption | No rate limit; expensive queries; large payloads |
| **API5** | Broken Function Level Authorization (BFLA) | Calling admin endpoints as a regular user |
| **API6** | Unrestricted Access to Sensitive Business Flows | Bot scraping; credential stuffing |
| **API7** | Server-Side Request Forgery (SSRF) | API fetches arbitrary URLs |
| **API8** | Security Misconfiguration | Verbose errors, default creds, missing headers |
| **API9** | Improper Inventory Management | Forgotten v0 endpoints, undocumented APIs |
| **API10** | Unsafe Consumption of APIs | Trusting third-party API responses without validation |

### BOLA in detail (the #1 cause)
```
GET /orders/9001
Authorization: Bearer <Alice's token>
```
Server returns the order — but it belongs to Bob. The auth check confirmed *who Alice is*, not *whether she can see this order*. **Always authorize the specific resource.**

### Mass assignment defense
```
PATCH /users/me
{ "email": "...", "role": "admin" }   ← user smuggling role escalation
```
Defense: explicit allowlists. Don't `User.update(req.body)`. Use DTOs / strong params / Zod schemas / explicit field mapping.

### Excessive data exposure
Don't return everything from the DB; return only what the client needs. The pattern that scales: dedicated **view models** per endpoint, not raw DB rows.

---

## API Gateway & BFF Patterns

### API Gateway
A single front door for many backend services. Handles:
- TLS termination
- Authentication / authorization
- Rate limiting
- Request routing
- Response caching
- Logging / metrics
- API key management

**Tools:** Kong, Apigee, AWS API Gateway, Cloudflare API Gateway, Envoy/Istio, Traefik.

### Backend-for-Frontend (BFF)
A thin server tailored to one frontend (web vs iOS vs Android), aggregating microservices and shaping data per client.

```
                        ┌─→ [Web BFF]   ─→ Tailored for browser
                        │
[Web/iOS/Android] ─────┤   ┌─→ [iOS BFF]   ─→ Tailored for iOS
                        │   │
                        └───┼─→ [Android BFF]  ─→ Tailored for Android
                            │
                  Backend microservices
```

### When to use BFF
- Multiple frontends with different needs
- Microservices forcing every client to call N services
- Frontends should not depend on backend microservice contracts directly

### A caveat
Adding a BFF adds an operational layer. For a single web client, just use the API gateway. Bring in BFF when the cost of NOT having one (frontends doing complex aggregation, over-fetching, microservice coupling) exceeds the cost of running it.

---

## Documentation, OpenAPI & SDKs

### OpenAPI (formerly Swagger)
The de facto standard for documenting REST APIs. JSON or YAML schema:

```yaml
openapi: 3.1.0
info:
  title: Orders API
  version: 1.0.0
paths:
  /orders/{id}:
    get:
      operationId: getOrder
      parameters:
        - name: id
          in: path
          required: true
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Order' }
components:
  schemas:
    Order:
      type: object
      required: [id, status]
      properties:
        id: { type: string }
        status: { type: string, enum: [pending, paid, shipped] }
```

### What you get from OpenAPI
- **Auto-generated docs** (Swagger UI, Redoc, Stoplight, Mintlify, Scalar)
- **Client SDKs** in any language (`openapi-generator`, `Speakeasy`, `Stainless`)
- **Server validation** — middleware enforces the schema
- **Mock servers** — `Prism`, `Mockoon`
- **Contract tests** — verify implementation matches schema

### Schema-first vs. code-first
- **Schema-first** — write the OpenAPI/Protobuf/GraphQL schema first; generate code. Best for cross-team contracts, public APIs.
- **Code-first** — generate the schema from server code (e.g., NestJS decorators, FastAPI). Faster iteration; risks schema drift.

**Recommendation:** schema-first for public/cross-team APIs; code-first for team-internal.

### Documentation that doesn't suck
- **Working code examples** in multiple languages
- **Authentication setup** is the *first* thing
- **Quickstart** (do something useful in 5 minutes)
- **Common errors** documented with what causes them
- **Changelogs** — versioned, searchable
- **Interactive try-it** with sandbox credentials
- **SDKs** for top languages (TS, Python, Go, Java, Ruby)
- **Postman / Insomnia collections** if your audience uses them

### Modern docs tools
- **Mintlify**, **Stoplight**, **Redocly**, **Scalar**, **Bump.sh** — beautiful, modern OpenAPI-driven docs
- **Stripe**, **Twilio**, **GitHub** docs — the gold standards to study

### SDK strategy
Auto-generated SDKs from OpenAPI are fine for internal use; for public APIs, hand-curate or use `Stainless`/`Speakeasy` (which produce idiomatic, hand-quality SDKs from OpenAPI).

---

## Performance & Scalability

### Reduce payload
- **Field selection** / sparse fieldsets
- **Compression** — Brotli > gzip; always for JSON
- **Pagination defaults** — sensible limits, never unbounded
- **Don't over-include** — let clients ask for what they need

### HTTP/2 and HTTP/3
- HTTP/2 multiplexes requests on one connection — eliminates connection overhead for chatty clients
- HTTP/3 (QUIC) is faster on lossy networks (mobile, distant geographies)

### Connection management
- Keep-alive on by default
- Tune connection pool sizes
- Close idle connections after a sensible timeout

### Batching
- Allow `POST /batch` with multiple operations
- Reduces round-trips for bulk imports / chatty clients

### N+1 prevention
- Use DataLoader or request-scoped caches
- Eager-load related data in lists
- Document and audit queries per endpoint

### CDN for read APIs
- Cache GET responses at the CDN with `s-maxage`
- Use `stale-while-revalidate` for resilience
- Tag-based invalidation (Fastly, Cloudflare Enterprise) for editorial workflows

### Async for slow operations
Don't make clients wait synchronously for slow work. Long-running operation pattern (above).

### Capacity planning
- Track p95/p99 latency per endpoint
- Set error budgets and SLOs per API
- Auto-scale; load-test before launches

---

## Backward Compatibility & Deprecation

### The Hyrum's Law reality
> *"With a sufficient number of users of an API, it does not matter what you promise in the contract: all observable behaviors of your system will be depended on by somebody."*

You will be surprised by what consumers depend on. Plan accordingly.

### Deprecation playbook
1. **Announce loudly** — changelog, email, in-product banner, dev portal
2. **Set a sunset date** at least 6 months out (12+ for big public APIs)
3. **Mark deprecated** in OpenAPI / docs
4. **`Sunset` HTTP header** — tells clients programmatically:
   ```
   Sunset: Wed, 30 Apr 2026 00:00:00 GMT
   Deprecation: true
   Link: <https://api.example.com/changelog>; rel="deprecation"
   ```
5. **Track usage** per deprecated endpoint, per customer
6. **Reach out** to top consumers; help them migrate
7. **Final warning** with date confirmed
8. **Remove** — return 410 Gone with a helpful message

### Compatibility classification
On every change, label it:
- **Additive** — safe; ship anytime
- **Soft-breaking** — affects edge cases; communicate; phased rollout
- **Hard-breaking** — version bump or sunset cycle required

---

## API Lifecycle & Governance

For organizations with many APIs:

### API design review
- Every new public endpoint goes through a design review
- Style guide (URL conventions, error shapes, pagination) is enforced
- New API? Spec proposed before implementation
- Tooling: linting OpenAPI specs (Spectral), API style guides

### API catalog
- A single source listing every API, its owner, status, version, docs link, contact
- Tools: **Backstage** (Spotify), Kong API Catalog, Stoplight, Postman

### API style guide
Every mature org has one. Examples to study:
- **Google AIP** (`google.aip.dev`) — comprehensive
- **Microsoft REST API Guidelines** (open-source)
- **Zalando API Guidelines** (open-source)
- **PayPal API Standards**
- **JSON:API** spec

### API consumer feedback loop
- Sandbox environment with real-feeling data
- Public roadmap
- Office hours / direct support for top consumers
- Issue tracker accepting public bug reports
- Changelog as a public document

---

## Common Anti-Patterns

### 1. Verbs in URLs
`POST /createUser`, `POST /getUserById`. Use HTTP methods + nouns: `POST /users`, `GET /users/42`.

### 2. Returning 200 for errors
```json
{ "status": "error", "code": 500 }   ← HTTP 200 OK
```
Status codes exist; use them. Clients write `if (response.ok)` and your "error" passes.

### 3. Inconsistent error shapes
Each endpoint returns a different error structure. Clients write 12 different parsers. Pick one (RFC 9457), apply everywhere.

### 4. Exposing DB primary keys
Auto-incrementing integer IDs leak: how many users you have, when records were created, enable IDOR. Use UUIDs / ULIDs / prefixed opaque IDs.

### 5. Returning the kitchen sink
Every endpoint returns every field on the entity. Mobile clients suffer; security gets harder. Define view models per endpoint.

### 6. No pagination by default
`GET /users` returns 200,000 rows. Page or fail. Always cap.

### 7. Vague status codes
500 for everything, including validation errors. Lazy and unhelpful. Match the cause to the code.

### 8. Mass assignment
`User.update(req.body)`. Allowlist explicit fields. Frameworks (Rails strong params, NestJS DTOs, Zod schemas) help.

### 9. Versioning by branch / fork
"We forked the v1 codebase for v2." Both rot. Use a single codebase that handles multiple versions via a transformer pipeline (Stripe model).

### 10. Authentication in URL
`GET /users?api_key=...`. Logged everywhere; visible in browser history. Use headers.

### 11. Trusting client IDs
`POST /orders` with `{ "user_id": "anyone" }` and the server uses it. Always derive identity from auth, not from request body.

### 12. Synchronous slow operations
Endpoints that return after 30 seconds. Mobile clients abandon; gateways time out. Async with operation tracking.

### 13. Public endpoints that change shape silently
A field renamed; a type changed; a default flipped. Every change is a breaking change to *someone*. Treat compat seriously.

### 14. No rate limiting
First time you go viral, you go down. Rate limit from day one (even if generous).

### 15. Documentation drift
API ships; docs lag by months. Generate docs from schema; treat doc updates as part of "done."

### 16. Treating internal APIs as throwaway
"It's just internal, doesn't need quality." Internal APIs become long-lived; bad ones rot for years. Apply discipline.

### 17. Field-level chaos
`createdAt` here, `created_at` there, `creation_time` over there. Pick a convention; lint it; never deviate.

### 18. Reinventing OAuth
"Let's roll our own auth flow." You'll get it wrong. Use OIDC / OAuth 2.1 / passkeys.

### 19. No request IDs
Customer says "this failed at 3pm." With no request ID, you have nothing to correlate. Always echo a `X-Request-Id` (generate if absent).

### 20. Silently dropping fields
Client sends `{ "foo": "bar", "baz": 1 }`; server only knows about `foo`; ignores `baz` silently. Reject unknown fields in strict mode (or warn) — a typo gone silent is a bug gone silent.

---

## Quick Reference Checklist

### Design
- [ ] API style chosen deliberately (REST / GraphQL / gRPC / tRPC) per use case
- [ ] Resource-oriented, noun-based URLs
- [ ] Plural collections; opaque IDs
- [ ] Logical properties (snake_case OR camelCase, consistent)
- [ ] Money in minor units + currency code; never floats
- [ ] Timestamps in ISO 8601 UTC

### HTTP
- [ ] Methods used semantically; idempotent ones are
- [ ] Status codes accurate (no 200 for errors)
- [ ] Content negotiation (`Accept`, `Content-Type`)
- [ ] Compression on responses
- [ ] HTTP/2 or /3 enabled

### Pagination & filtering
- [ ] Cursor-based pagination on all lists
- [ ] Server-capped `limit`
- [ ] `has_more` / `next_cursor` in response
- [ ] Filter / sort / sparse fields documented

### Errors
- [ ] RFC 9457 problem-details (or consistent equivalent)
- [ ] Stable machine-readable `code`
- [ ] Field-level errors for validation
- [ ] `request_id` in every response (and error)
- [ ] No stack traces / internal info leaked

### Idempotency
- [ ] `Idempotency-Key` supported on all non-idempotent state changes
- [ ] Stored with retention; replays return original response
- [ ] Mismatched body on same key → reject

### Caching
- [ ] `Cache-Control` set deliberately per endpoint
- [ ] `ETag` / `Last-Modified` on cacheable resources
- [ ] `If-Match` for optimistic concurrency on writes

### Rate limiting
- [ ] Default limits documented
- [ ] Per-tier limits (anon, authed, paid)
- [ ] `RateLimit*` headers on every response
- [ ] `Retry-After` on 429

### Security
- [ ] Auth via headers (never URL)
- [ ] Authorization at the resource level (not just route)
- [ ] Mass-assignment guards
- [ ] CORS restricted to known origins for browser APIs
- [ ] Webhooks signed; replay protection
- [ ] Rate limiting; bot mitigation
- [ ] OWASP API Top 10 reviewed

### Versioning
- [ ] Strategy chosen (URI / date-based / header)
- [ ] Compatibility rules documented
- [ ] Deprecation playbook in place
- [ ] `Sunset` / `Deprecation` headers used

### Documentation
- [ ] OpenAPI / GraphQL schema source-of-truth
- [ ] Auto-generated, always current docs
- [ ] Quickstart (5-minute first call)
- [ ] Changelogs published
- [ ] SDKs in top languages (or Stainless/Speakeasy generated)
- [ ] Sandbox / mock environment

### Observability
- [ ] Request IDs throughout
- [ ] Structured logs per request
- [ ] Distributed tracing (OpenTelemetry)
- [ ] RED metrics per endpoint
- [ ] SLOs defined per critical endpoint

### Process
- [ ] API design review before implementation
- [ ] Style guide enforced (Spectral lint on OpenAPI)
- [ ] API catalog with ownership
- [ ] Consumer feedback channel

---

## Further Reading

### Foundational
- **Roy Fielding's dissertation (2000)** — the original REST architectural style
- **Mark Massé — *REST API Design Rulebook***
- **Mike Amundsen — *RESTful Web Clients*** / ***API Traffic Management 101***
- **Arnaud Lauret — *The Design of Web APIs*** — practical, modern, excellent
- **Brandur Leach's blog (`brandur.org`)** — Stripe's API engineering deep dives

### Specs
- **OpenAPI Specification** — `spec.openapis.org`
- **JSON:API** — `jsonapi.org` — opinionated REST conventions
- **AsyncAPI** — for event-driven APIs
- **CloudEvents** — CNCF spec for event metadata
- **Standard Webhooks** — `standardwebhooks.com`
- **RFC 9457** — Problem Details for HTTP APIs
- **Google AIP** — `google.aip.dev`

### Style guides to study
- **Google AIP** — comprehensive; the gold standard
- **Microsoft REST API Guidelines** (open-source on GitHub)
- **Zalando API Guidelines** (open-source)
- **PayPal API Standards**
- **Stripe API** — go read every doc page

### GraphQL
- **`graphql.org` learn**
- **Apollo Docs** — patterns and federation
- **Marc-André Giroux — *Production Ready GraphQL***

### Books
- **Sam Newman — *Building Microservices*** — communication patterns
- **Susan Fowler — *Production-Ready Microservices***
- **Gregor Hohpe — *Enterprise Integration Patterns*** — async messaging foundation

### People to follow
- Brandur Leach (Stripe), Marc-André Giroux (GraphQL), Phil Sturgeon (APIs You Won't Hate), Kin Lane (API Evangelist), Arnaud Lauret (API Handyman), Mark Nottingham (HTTP standards)

### Watch / listen
- "How to Design Great APIs" — many talks at API Days conferences
- Stripe Engineering Blog — model API design articles
- Architecture decision records from public companies (Twilio, GitHub, Slack)

---

> **Closing thought:** an API is a promise. The promise lives forever; the implementation behind it can change a thousand times. I obsess over API design because I've watched what happens when teams don't — **what you ship today is what every consumer of your API depends on for years**, and breaking changes are paid for in customer trust, not in engineering hours. The goal isn't to design a clever API; the goal is to design a **boring, predictable, durable, evolvable** one — where the right thing is the easy thing, the wrong thing is hard, errors are clear, and tomorrow's needs can be added without breaking yesterday's clients. **Good APIs are invisible; they just feel right.**
