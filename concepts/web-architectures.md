# Web Architectures

> A practical guide to system-level architectural styles for web applications — what they are, what they cost, and how to choose.

---

## Table of Contents

1. [What is Web Architecture?](#what-is-web-architecture)
2. [Why Architecture Matters](#why-architecture-matters)
3. [The Three Architectural Layers](#the-three-architectural-layers)
4. [Backend / System Architectures](#backend--system-architectures)
   - [Monolith](#1-monolith)
   - [Modular Monolith](#2-modular-monolith)
   - [Microservices](#3-microservices)
   - [Service-Oriented Architecture (SOA)](#4-service-oriented-architecture-soa)
   - [Serverless / FaaS](#5-serverless--faas)
   - [Event-Driven Architecture](#6-event-driven-architecture)
   - [CQRS + Event Sourcing](#7-cqrs--event-sourcing)
   - [Hexagonal / Ports & Adapters](#8-hexagonal--ports--adapters)
   - [Clean / Onion Architecture](#9-clean--onion-architecture)
   - [Layered Architecture](#10-layered-architecture)
   - [Space-Based Architecture](#11-space-based-architecture)
5. [Frontend Architectures](#frontend-architectures)
6. [Communication Architectures](#communication-architectures)
7. [Data Architectures](#data-architectures)
8. [Deployment & Distribution Architectures](#deployment--distribution-architectures)
9. [Cross-Cutting Concerns](#cross-cutting-concerns)
10. [How to Decide: The Architectural Decision Framework](#how-to-decide-the-architectural-decision-framework)
11. [Architectural Maturity & Evolution](#architectural-maturity--evolution)
12. [Common Pitfalls](#common-pitfalls)
13. [Decision Cheat Sheet](#decision-cheat-sheet)
14. [Further Reading](#further-reading)

---

## What is Web Architecture?

**Web architecture** is the high-level organization of how a web application's components — clients, servers, databases, services, networks — are structured, deployed, and communicate.

It answers questions like:
- Is this one process or many?
- Where does state live?
- How do components talk?
- How do we scale, deploy, fail, recover?
- Where do team boundaries map onto code boundaries?

> **Architecture is what's hard to change later.** A render strategy can be swapped per route in a sprint. A microservices migration takes years. The distinguishing trait of architectural decisions is **cost-of-reversal** — they shape every line of code that follows.

---

## Why Architecture Matters

### 1. It's the Largest Cost Lever
The wrong architecture costs orders of magnitude more than wrong code. A premature microservices split has bankrupted startups; an unsplittable monolith has cratered scale-ups.

### 2. It Encodes Team Topology
**Conway's Law** ("organizations design systems that mirror their communication structure") is not a metaphor — it's an iron rule. Architecture is sociotechnical, not just technical. Choosing one means choosing how teams will work for years.

### 3. It Sets Performance Ceilings
You can't tune your way out of an N+1 architecture or a chatty network boundary. The shape determines what's possible.

### 4. It Determines Failure Modes
Monoliths fail loudly and totally. Microservices fail partially and confusingly. Each architecture has a *failure signature*, and you'll spend a lot of nights with it.

### 5. It Shapes Hiring
The architecture you ship is the talent you need. Distributed systems engineers, SREs, platform teams, frontend specialists — different shapes need different orgs.

> **Lesson learned:** the most expensive architectural mistake is **over-engineering for scale you don't have**. The second most expensive is **under-engineering and hitting a wall too late**. The judgment that actually matters is sensing which side you're on — and that judgment only comes from shipping, breaking things, and rebuilding.

---

## The Three Architectural Layers

Real architecture decisions cut across three independent axes:

| Axis | Question | Examples |
|---|---|---|
| **Structural** | How is the code organized? | Monolith / Modular / Microservices |
| **Logical** | How is responsibility divided? | Layered / Hexagonal / Clean / Event-Driven / CQRS |
| **Physical / Deployment** | Where and how does it run? | Single VM / Containers / Serverless / Edge |

Mistaking these for the same thing causes endless confusion. A *modular monolith* using *hexagonal architecture* deployed as *serverless functions* is a coherent, sensible choice — but it's three independent decisions.

---

## Backend / System Architectures

### 1. Monolith

A single deployable unit containing all business logic, served by one (replicated) process.

```
[Client] ── HTTP ──► [App Server (one binary)] ── SQL ──► [Database]
```

**Benefits:**
- **Fastest to build** — one repo, one deploy, one runtime
- **Simple operations** — one log stream, one metric set, one rollback
- **No network between modules** — function calls, not RPCs; no serialization overhead
- **ACID transactions** trivially span features
- **Refactoring is local** — rename across the whole codebase atomically
- Hugely underrated: most successful companies started here and stayed for years

**Trade-offs:**
- Scaling means scaling *everything*, even if only one feature is hot
- Deploys are all-or-nothing — one bad change blocks everyone
- Tech-stack lock-in (one language, one framework)
- Eventually grows into a "big ball of mud" without discipline

**When to use:** ~95% of new applications, especially under 50 engineers. **Default to monolith. Earn the right to split.**

> **The Shopify thesis:** Shopify is a multi-billion-dollar Rails monolith. Stack Overflow ran on a monolith. GitHub was a monolith for 15+ years. "Monolith" is not a slur.

---

### 2. Modular Monolith

A monolith with **strict internal module boundaries** — public APIs, no cross-module DB access, enforced via tooling (linters, architecture tests, separate folders/packages).

```
[App Server]
├── module/billing       (private DB tables, public API only)
├── module/catalog
├── module/identity
└── module/notifications
```

**Benefits:**
- All the **operational simplicity** of a monolith
- All the **logical separation** of microservices
- **Future-proofs** for extraction — modules with clean contracts can become services later
- **No distributed-systems tax** unless you opt in
- Better cognitive load than a "ball of mud" monolith

**Trade-offs:**
- Discipline-dependent — "private" is a convention unless enforced by tooling
- Doesn't solve build-time / deploy-time scaling (still one binary)
- Can't have language/runtime diversity per module

**When to use:** the right answer for most teams that have outgrown a flat monolith but aren't ready to operate distributed systems. **The single most underrated architecture in 2026.**

---

### 3. Microservices

The application is decomposed into independently deployable services, each owning its own data, communicating over the network.

```
[Client] ─► [API Gateway] ─► [Identity Service]   ─► [Identity DB]
                          ├─► [Catalog Service]    ─► [Catalog DB]
                          ├─► [Order Service]      ─► [Order DB]
                          └─► [Billing Service]    ─► [Billing DB]
```

**Benefits:**
- **Independent deploys** — teams ship without coordination
- **Independent scaling** — scale only the hot services
- **Tech diversity** (theoretically) — Go for hot paths, Python for ML, Node for web
- **Failure isolation** — billing outage doesn't kill catalog
- **Team autonomy** at scale (mirrors "two-pizza teams")

**Trade-offs:**
- **Distributed systems are hard** — partial failures, network errors, retries, idempotency, eventual consistency, distributed tracing
- **Operational overhead** — 30 services means 30 deploy pipelines, 30 dashboards, 30 sets of secrets
- **Local development** is painful (need 12 services running just to test one)
- **Cross-service transactions** require sagas, outboxes, or distributed transactions — expensive
- **Data duplication and synchronization**
- **Latency** of network calls vs. function calls (1000x slower)
- Most teams discover *too late* that they don't have the platform maturity to operate them

**When to use:** at scale (typically 50+ engineers, or independent product domains), **after** you've outgrown a modular monolith and have a platform team to support it. Not the default.

> **Sam Newman's rule of thumb:** "Don't start with microservices. You'll regret it."

---

### 4. Service-Oriented Architecture (SOA)

Coarser-grained than microservices — a handful of large services around business capabilities, often with a shared **enterprise service bus (ESB)** or canonical data model.

**Benefits:** more autonomy than monolith, less operational overhead than microservices.
**Trade-offs:** ESBs become bottlenecks; canonical data models are notoriously brittle; the term itself is associated with 2000s enterprise IT pain.
**When to use:** rarely greenfield in 2026 — but the term is making a quiet comeback as "macro-services" or "right-sized services" for teams that found microservices too granular.

---

### 5. Serverless / FaaS

Code runs as discrete functions in a managed runtime (AWS Lambda, Cloudflare Workers, Vercel Functions, GCP Cloud Run / Cloud Functions, Azure Functions).

```
[Event source] ─► [Function (cold-starts, scales to zero)] ─► [Managed services]
```

**Benefits:**
- **Zero server management** — no patching, no capacity planning
- **Scale-to-zero** — pay only for invocations
- **Excellent for spiky / event-driven workloads**
- **Fast experimentation** — ship a function in minutes
- **Tight integration** with cloud event sources (S3, queues, schedules)

**Trade-offs:**
- **Cold starts** — first request after idle can be slow (mitigated by provisioned concurrency, edge runtimes, snapstart)
- **Vendor lock-in** — IAM, event shapes, runtime quirks
- **Local development** harder than a regular server
- **Long-running work** is awkward (timeouts, no persistent connections in classic Lambda)
- **State management** — every invocation is potentially a new instance
- **Cost cliffs** — at high steady traffic, serverless is *more* expensive than VMs
- **Observability** requires extra effort (distributed tracing across 50 functions)

**When to use:** event-driven workloads, glue/automation, scheduled jobs, low-traffic APIs, edge middleware, and modern web frameworks (Next.js, Remix on Vercel/Netlify) where the trade-offs are well-managed.

> **Worth knowing:** "Serverless" is a deployment model, not an architecture. You can build a monolith on Lambda (one function, many handlers) or microservices on Lambda (many functions). Don't confuse the two.

---

### 6. Event-Driven Architecture (EDA)

Services communicate by emitting and reacting to **events** through a message broker (Kafka, RabbitMQ, AWS SQS/SNS, NATS, Redis Streams).

```
[Order Service] ──emits──► [order.placed event]
                                   │
                          ┌────────┼────────┐
                          ▼        ▼        ▼
                  [Inventory]  [Email]  [Analytics]
                  consumer    consumer   consumer
```

**Benefits:**
- **Loose coupling** — producers don't know about consumers
- **Easy to add new consumers** without touching producers
- **Natural fit for async workflows** — onboarding, notifications, ETL
- **Resilience** — a slow/down consumer doesn't block the producer
- **Auditability** — events are an immutable record

**Trade-offs:**
- **Hard to reason about flow** — "what happens after a user signs up?" requires tracing across topics and consumers
- **Eventual consistency** — UI may show stale data until events propagate
- **Schema evolution** is hard at scale (tools: Avro, Protobuf with schema registries)
- **Duplicate / out-of-order events** are the norm — consumers must be idempotent
- **Operational complexity** of running a broker (or paying for managed Kafka)

**When to use:** async workflows that span services, fan-out scenarios (one event, many consumers), audit-critical domains, real-time pipelines.

#### Variants
- **Event notification** — "something happened, check it if you care"
- **Event-carried state transfer** — events carry the full state needed by consumers (reduces back-calls)
- **Event sourcing** — see below

---

### 7. CQRS + Event Sourcing

**CQRS** — Command Query Responsibility Segregation — separates write models (commands) from read models (queries).
**Event sourcing** — state is the result of replaying an immutable log of events (not stored as a row).

```
[Command]  ─► [Aggregate]  ─► [Event Log] ──► [Read Model 1: SQL projection]
                                            ├► [Read Model 2: search index]
                                            └► [Read Model 3: cache]
```

**Benefits:**
- **Complete audit trail** — every state change is an event
- **Time travel** — replay events to any past state, perfect for debugging
- **Independent read scaling** — denormalize for each query pattern
- **Temporal queries** — "what did this look like on March 1?"
- **Rebuild projections** — change read model schema without migration

**Trade-offs:**
- **Significant complexity** — event versioning, idempotency, projections, eventual consistency
- **Hard to onboard** — most engineers haven't built event-sourced systems
- **Long event logs** require snapshotting strategies
- **Not a great fit** for simple CRUD

**When to use:** financial systems, audit-heavy domains, collaborative editors with undo/redo, complex domain models with rich invariants. **Don't use for blogs.**

---

### 8. Hexagonal / Ports & Adapters

Domain logic is at the center. The outside world (UI, DB, message brokers, third-party APIs) connects through **ports** (interfaces) implemented by **adapters**.

```
                [Web Adapter]   [CLI Adapter]   [Test Adapter]
                       │             │              │
                       └─────────────┼──────────────┘
                                     │
                              [Application Core]
                                     │
                       ┌─────────────┼──────────────┐
                       │             │              │
                [SQL Adapter]   [Kafka Adapter]  [HTTP Adapter]
```

**Benefits:**
- **Domain logic is framework-agnostic** — swap databases, queues, web frameworks
- **Deeply testable** — substitute fake adapters in unit tests
- **Long-lived investment** — protects business logic from infrastructure churn

**Trade-offs:**
- **Indirection tax** — every external call goes through a port + adapter
- **Over-engineered for simple CRUD apps**
- **Hard to onboard** — junior engineers see "where's my Express handler?"

**When to use:** long-lived systems with rich domain logic, regulated domains, products with multiple interfaces (web + mobile + CLI + API).

---

### 9. Clean / Onion Architecture

A close cousin of hexagonal — concentric rings of dependency: **Entities → Use Cases → Interface Adapters → Frameworks**. Dependencies point *inward*; the inner core knows nothing about the outer rings.

**Benefits:** same as hexagonal — testability, framework-independence, long-term maintainability.
**Trade-offs:** same indirection tax; rigid layering can make simple changes touch 5 files.
**When to use:** popularized by Robert C. Martin (Uncle Bob); strong fit for enterprise software, less so for lean web products. Often over-applied; a modular monolith with clear module boundaries gets you 80% of the benefit with 20% of the structure.

---

### 10. Layered Architecture

The classic 3-tier (or 4-tier): **Presentation → Application → Domain → Infrastructure**. Each layer depends only on the one below.

**Benefits:** simple, well-understood, taught in every CS curriculum.
**Trade-offs:** strict layering causes "anemic" middle layers; cross-cutting concerns (logging, auth, transactions) are awkward.
**When to use:** the classic default for enterprise apps; perfectly fine for many CRUD systems.

---

### 11. Space-Based Architecture

Built for extreme load: in-memory data grids replicate state across processing units; persistence is async to a separate data writer. Used in trading platforms, ad-tech, large-scale gaming.

**Benefits:** massive horizontal scale without DB bottleneck.
**Trade-offs:** highly specialized; complex consistency guarantees.
**When to use:** 100K+ TPS workloads with strict latency SLAs. Most teams will never need this.

---

## Frontend Architectures

(See also [Web Rendering Strategies](web-rendering-strategies.md) for SSR/SSG/CSR specifics.)

### 1. Server-Rendered MPA (Classic)
Server emits HTML per route; minimal JS sprinkled on top.
**Use:** content sites, traditional CRUD, Rails/Django/Laravel. Modern via Hotwire, HTMX, Turbo.

### 2. SPA (Single-Page Application)
One HTML doc, client-side routing, JSON APIs.
**Use:** authenticated apps, dashboards, app-like experiences.

### 3. Universal / Isomorphic (SSR + Hydration)
Same JS code runs on server and client; SSR for first paint, hydration for interactivity.
**Use:** SEO-critical apps with rich interactivity (Next.js, Remix, Nuxt, SolidStart).

### 4. Islands Architecture
Mostly static HTML with isolated interactive widgets ("islands").
**Use:** content-heavy sites with selective interactivity (Astro, Fresh).

### 5. Micro-Frontends
Multiple independently-deployable frontend apps composed at runtime or build time.

```
┌──────────────────────────────────────────────┐
│              Container Shell                 │
├──────────────┬──────────────┬───────────────┤
│  Catalog MFE │  Cart MFE    │  Checkout MFE │
│  (Team A)    │  (Team B)    │  (Team C)     │
└──────────────┴──────────────┴───────────────┘
```

**Benefits:**
- **Team autonomy** — independent deploys, tech choices, release cadence
- **Smaller individual codebases**
- **Incremental migration** — strangle a legacy SPA piece by piece

**Trade-offs:**
- **Bundle bloat** — multiple framework runtimes shipped to the browser
- **Cross-app state, navigation, auth** require careful design
- **Design system drift** between teams
- **Performance penalty** — running multiple framework instances
- **Operational complexity** — composition, versioning, contracts, integration tests
- Often **a sociotechnical Band-Aid** rather than a real solution

**When to use:** large orgs (100+ frontend engineers), strict team boundaries, *after* you've exhausted simpler options. **Not** "we want to use React and Vue together."

### 6. JAMstack (Static + APIs + JS)
Pre-built markup, dynamic via APIs, served from CDN.
**Use:** content + light commerce; pairs naturally with headless CMS.

### 7. PWA — Progressive Web App
Installable, offline-capable, push-notification-able web apps via service workers + web app manifest.
**Use:** apps that benefit from app-like installability or offline support; alternative to native apps for many use cases.

---

## Communication Architectures

How clients and services talk:

### 1. REST
Resource-oriented, stateless, HTTP verbs, JSON.
**Pros:** universal, cacheable, simple, well-understood.
**Cons:** over/under-fetching, multiple round trips for related data.
**Use:** the default for public APIs and most service-to-service.

### 2. GraphQL
Client specifies the shape of the response.
**Pros:** no over-fetching, strong types, single endpoint, great DX with codegen.
**Cons:** server complexity (N+1, depth limits, caching), auth at field level, harder to cache than REST.
**Use:** many heterogeneous clients, large product surface, mobile apps with bandwidth constraints.

### 3. gRPC
Binary protocol over HTTP/2 with Protobuf schemas.
**Pros:** very fast, strongly typed, streaming built-in.
**Cons:** browser support requires gRPC-Web proxy; less human-debuggable.
**Use:** internal service-to-service, performance-critical RPCs.

### 4. tRPC
End-to-end type-safe RPC for TypeScript monorepos.
**Pros:** zero-codegen type safety from server to client.
**Cons:** TS-only, ties client and server tightly.
**Use:** internal apps where the same team owns frontend + backend (Next.js + tRPC is a popular combo).

### 5. WebSockets / SSE / WebTransport
Persistent connections for real-time streams.
**Use:** chat, multiplayer, live dashboards, notifications.

### 6. Messaging (Async)
Brokers like Kafka, RabbitMQ, SQS, NATS.
**Use:** decoupled async workflows; cross-service events.

### 7. Webhooks
Outbound HTTP callbacks to subscribers.
**Pros:** simple, universal.
**Cons:** retry/idempotency/security each integration's problem.
**Use:** integrating with external systems (Stripe, GitHub, Slack).

### 8. API Gateway / BFF
A facade in front of multiple services.
**API Gateway** — generic; handles auth, rate-limiting, routing.
**BFF (Backend-for-Frontend)** — tailored to one frontend, aggregating + shaping data.

---

## Data Architectures

### 1. Single Database (Shared)
Every service queries the same DB.
**Pros:** simple, ACID transactions across the app.
**Cons:** schema becomes a coupling point; can't evolve independently.
**Use:** monoliths and modular monoliths.

### 2. Database-per-Service
Each microservice owns its DB; no cross-service queries allowed.
**Pros:** services can pick storage best for them; independent evolution.
**Cons:** cross-service queries require API calls or async replication; no distributed transactions.
**Use:** microservices done right.

### 3. Polyglot Persistence
Different storage for different access patterns: PostgreSQL for relational, Redis for cache/queues, Elasticsearch for search, S3 for blobs, ClickHouse for analytics, Neo4j for graph.
**Use:** mature applications with varied access patterns.

### 4. CQRS Read Models
Separate optimized read models built from the write side via events.
**Use:** read/write asymmetry, complex queries, real-time analytics.

### 5. Data Lake / Lakehouse / Warehouse
Separate analytical store fed by ETL/CDC.
- **Data warehouse** (Snowflake, BigQuery, Redshift) — structured, fast queries
- **Data lake** (S3 + Parquet) — unstructured, cheap storage
- **Lakehouse** (Databricks, Iceberg) — best of both
**Use:** anything BI/analytics; never run analytics on the OLTP database in production.

### 6. Event Log as Source of Truth
Kafka or similar; databases are projections.
**Use:** event sourcing, real-time pipelines, ML feature stores.

### 7. Sharding / Partitioning
Splitting a large table across multiple physical databases by key (user_id, region).
**Use:** when one DB can't handle the load — but try vertical scaling and read replicas first; modern Postgres on a big box goes very far.

### 8. Read Replicas
Replicas serve read traffic while the primary handles writes.
**Use:** read-heavy workloads. Watch for replication lag in user-facing flows ("I just edited my profile but it shows the old name").

### 9. Caching Layers
- **Browser cache** — `Cache-Control` + ETags
- **CDN cache** — static + cached HTML
- **Reverse proxy cache** — Varnish, Nginx
- **Application cache** — Redis, Memcached
- **In-process cache** — LRU in memory

> **In practice:** "There are only two hard things in computer science: cache invalidation and naming things." — Phil Karlton. Most production incidents are cache invalidation bugs in disguise.

---

## Deployment & Distribution Architectures

### 1. Single VM / Bare Metal
**Use:** small projects, cost-sensitive, predictable load. Often underrated.

### 2. Containerized (Docker)
**Use:** the modern default. Consistent environments dev/staging/prod.

### 3. Container Orchestration (Kubernetes, ECS, Nomad)
**Pros:** automated scheduling, scaling, self-healing.
**Cons:** **massive operational complexity** — most teams don't need it.
**Use:** at scale, with a platform team. **Don't run K8s for 3 services.**

### 4. Serverless / FaaS
**Use:** event-driven, spiky, low-traffic APIs, edge logic.

### 5. PaaS (Heroku, Render, Fly.io, Railway, Vercel, Netlify)
**Pros:** push-to-deploy, no infra to manage, sane defaults.
**Cons:** less flexibility, can become expensive at scale.
**Use:** the right answer for most early-stage teams. The "boring" choice that ships product.

### 6. Edge-First
Code runs on CDN edge nodes globally close to users.
**Use:** latency-sensitive content, geo-routing, auth middleware, A/B routing.

### 7. Multi-Region
Active-active or active-passive deployments across regions.
**Pros:** low latency globally, regional failure resilience.
**Cons:** **distributed data is genuinely hard** — write conflicts, CRDTs, regional consistency.
**Use:** global products with regional latency SLAs or compliance requirements.

### 8. Multi-Cloud / Hybrid
Running across multiple clouds (AWS + GCP + Azure) or cloud + on-prem.
**Pros:** vendor independence, regulatory compliance.
**Cons:** **enormous complexity tax**; lowest common denominator services; networking and IAM are nightmares.
**Use:** strict regulatory requirements; rarely worth it as pure "avoid lock-in" insurance.

---

## Cross-Cutting Concerns

Architectures only succeed if these are designed in, not bolted on:

### Observability
- **Logs** (structured, centralized) — Loki, ELK, Datadog
- **Metrics** (time-series) — Prometheus, Datadog
- **Traces** (distributed) — OpenTelemetry, Jaeger, Honeycomb
- **Errors** — Sentry, Rollbar
- **RUM** (real user monitoring) — for frontend perf
- **Synthetic monitoring** — uptime + critical-path checks

> Distributed systems without distributed tracing are unmaintainable. Add it on day one.

### Security
- TLS everywhere (HTTPS, mTLS service-to-service)
- AuthN (OIDC, OAuth, passkeys) and AuthZ (RBAC, ABAC, OPA, Cedar)
- Secrets management (Vault, AWS Secrets Manager, Doppler, 1Password)
- Network segmentation (VPCs, security groups, service mesh)
- Defense in depth — assume any one layer can fail

### Reliability
- SLOs (objectives) and error budgets — Google SRE practice
- Circuit breakers, bulkheads, timeouts
- Health checks (liveness, readiness, startup)
- Graceful shutdown
- Chaos engineering (Gremlin, Chaos Monkey) at scale

### CI/CD
- Trunk-based development for high-velocity teams
- Automated tests (unit + integration + e2e + contract)
- Progressive delivery — canary, blue/green, feature flags
- Infrastructure as Code (Terraform, Pulumi, CDK)
- GitOps (ArgoCD, Flux) for K8s

### Data Governance
- Schema migration discipline (Liquibase, Flyway, Prisma Migrate)
- Backups + tested restores (an untested backup is not a backup)
- PII handling, retention, deletion (GDPR, CCPA)
- Lineage and quality monitoring for data pipelines

### Cost Management
- Tag everything for attribution
- Track unit economics (cost per user, per transaction)
- Spot instances, reserved capacity, savings plans
- Egress is the silent killer — minimize cross-region/cross-cloud transfers
- The "FinOps" practice is real and necessary at scale

---

## How to Decide: The Architectural Decision Framework

Walk through these questions in order. The first decisive answer wins.

### Q1. What's your team size?
- **1–10 engineers** → Monolith. Period.
- **10–50** → Modular monolith. Start enforcing module boundaries.
- **50–200** → Selectively extract services along the *seams that hurt most* — usually domains with different scaling, deploy cadence, or compliance needs.
- **200+** → Microservices justified, but only with platform investment.

### Q2. What's your traffic profile?
- **< 10 RPS sustained** → Monolith on PaaS or single VM is fine.
- **10–1000 RPS** → Monolith + caching + CDN handles this.
- **1000+ RPS, uneven** → Probably need to split *some* hot paths.
- **Bursty / event-driven** → Serverless makes sense.
- **Steady high load** → VMs/containers are cheaper than serverless.

### Q3. What's the domain shape?
- **Single product, tight integration** → Monolith.
- **Multiple loosely-related products** → Service-per-product (or modular monolith with strict boundaries).
- **Real-time / collaborative** → Event-driven, possibly CQRS.
- **Heavy audit/compliance** → Hexagonal + event sourcing.
- **Content-driven** → Static + APIs.

### Q4. What's your operational maturity?
- **No SRE / platform team** → Stay simple. Avoid microservices, K8s, multi-region.
- **Has platform team + SRE practices** → Distributed architectures become viable.
- **Mature platform** (golden paths, CI/CD, observability, runbooks) → You've earned the right to scale up.

### Q5. What's the change rate?
- **Code changes daily** → Trunk-based, fast deploys. Monolith ships faster than microservices for small teams.
- **Domains evolve at very different rates** → Microservices win (independent deploys).

### Q6. What's the data shape?
- **Transactional, ACID-required** → Single relational DB. Don't fragment unless forced.
- **Eventually consistent OK** → Microservices + events viable.
- **Read-dominated, complex queries** → CQRS + read replicas / projections.
- **Time-series / analytics** → Polyglot persistence with a separate analytical store.

### Q7. What does the org look like?
- **One team owns everything** → Monolith.
- **Multiple autonomous teams** → Architecture should mirror team boundaries (Conway's Law).
- **Many product squads** → Each squad owns its services + frontend slice.

### Q8. What's the budget?
- **Cost-sensitive, low traffic** → PaaS + monolith + managed DB.
- **Steady high load** → VMs / containers (serverless gets expensive).
- **Spiky, unpredictable** → Serverless wins on cost.

### Q9. What's the failure tolerance?
- **Internal tools** → Monolith is fine; reboots are acceptable.
- **Customer-facing, business-critical** → Need redundancy, graceful degradation, multi-AZ.
- **Life-critical / financial** → Multi-region active-active, formal SLOs, chaos testing.

---

## Architectural Maturity & Evolution

Most companies traverse this path. The question is *how fast* and *how deliberately*.

### Stage 1: PMF Quest (1–10 engineers)
- **Stack:** Monolith on PaaS (Render, Fly.io, Railway, Heroku, Vercel)
- **Database:** Managed Postgres
- **Frontend:** Next.js or similar, mostly SSR + SSG
- **Goal:** ship fast, learn, survive
- **Don't:** introduce microservices, K8s, Kafka, or "future-proofing"

### Stage 2: Early Scale (10–50 engineers)
- **Stack:** Modular monolith; introduce module boundaries
- **Database:** Add read replicas, Redis cache, search index
- **Async:** Background workers, simple queue (SQS, BullMQ)
- **Goal:** preserve shippability while adding structure
- **Don't:** prematurely split into microservices

### Stage 3: Functional Decomposition (50–200 engineers)
- **Stack:** Extract services along the *seams that hurt* — a hot path, a compliance boundary, a 3rd-party integration domain
- **Platform:** Build (or buy) a platform team. Standardize observability, secrets, deploy
- **Async:** Real event broker (Kafka or managed equivalent)
- **Goal:** team autonomy without operational chaos
- **Risk:** "distributed monolith" — services that can't deploy independently. The worst of both worlds.

### Stage 4: Microservices at Scale (200+ engineers)
- **Stack:** Many services, service mesh, mature platform
- **Data:** Polyglot persistence, CQRS where needed, event sourcing for select domains
- **Operations:** Full SRE, error budgets, chaos engineering, multi-region
- **Org:** Strong platform/infrastructure org; clear service ownership

### When to Consolidate (yes, this happens)
Not all evolution is forward. Many companies have **re-merged** services back into modular monoliths after discovering the operational tax outweighed the team-autonomy benefit (Segment, Amazon Prime Video famously, Uber's monolith-of-microliths). **Going back is allowed.**

---

## Common Pitfalls

### 1. Microservices Before Modules
You can't split what you don't understand. Most "microservice migrations" fail because the team didn't first achieve clean *internal* boundaries. **Refactor to modular monolith first; extract services from clear modules later.**

### 2. The Distributed Monolith
Multiple services that must deploy together, share a database, and call each other synchronously in tight chains. You paid the distributed-systems tax and got the monolith's coupling. **The worst architecture.**

### 3. Resume-Driven Development
Adopting Kubernetes, Kafka, GraphQL, microservices because someone on the team wants them on their CV — not because the problem requires them.

### 4. Premature Optimization for Scale
Designing for 1M users when you have 100. The architecture you'd build for 1M is wrong for 100; the iterations you'd have shipped to learn what users want are gone.

### 5. Ignoring Conway's Law
Building an architecture that doesn't match team structure. If two teams have to coordinate every deploy, your "microservices" are functionally a monolith.

### 6. CAP Theorem Denial
You cannot have **C**onsistency, **A**vailability, and **P**artition tolerance simultaneously. In any distributed system across a network, you must choose: in a partition, do you serve stale data (AP) or refuse writes (CP)?

### 7. Ignoring the Fallacies of Distributed Computing
Peter Deutsch's fallacies — "the network is reliable, latency is zero, bandwidth is infinite, the network is secure, topology doesn't change, there is one administrator, transport cost is zero, the network is homogeneous." Every microservice architecture that ignores these fails on each one.

### 8. Sync-by-Default Service Calls
Services synchronously calling other services in long chains turn one user request into a cascading failure waiting to happen. Prefer async messaging or aggressive timeouts/circuits.

### 9. Shared Database in Microservices
Two services writing to the same tables is microservices in name only. The DB is the coupling.

### 10. Over-Engineering Domain Models
Spending six months on a Domain-Driven Design model for a CRUD app. **DDD is for complex domains.** Most apps don't have one.

### 11. Underestimating Data Migration Pain
Splitting a database is *vastly* harder than splitting code. You're rewriting code that works; meanwhile, dual-writes, backfills, and consistency checks consume the team for months.

### 12. Architecture Astronauting
Endless whiteboard diagrams, RFC documents, and "design phases" that delay shipping. Architecture is **discovered through delivery**, not designed in advance. Iterate.

### 13. Ignoring Observability Until It Hurts
"We'll add tracing later" — you won't. By the time you need it, the system is too complex to instrument retroactively. **Observability is foundational, not a feature.**

### 14. Forgetting That Architecture Has a Cost
Every layer, every service, every abstraction has a tax — cognitive, operational, financial. Be explicit about which tax buys what value.

### 15. Treating Architecture as Static
Architectures evolve. The right architecture for 2024-you might be wrong for 2026-you. Build with the assumption that pieces *will* be rewritten or moved. Reversibility is a feature.

---

## Decision Cheat Sheet

```
                                  Team size?
                       ┌──────────┴──────────┐
                    < 50                    ≥ 50
                       │                     │
                  Single domain?       Independent domains?
                  ┌────┴────┐          ┌─────┴─────┐
                Yes        No          Yes         No
                 │          │           │           │
            [Monolith]  [Modular   [Microservices  [Modular
                         Monolith]  + Platform]    Monolith]


                                Workload shape?
                       ┌──────────┴──────────┐
                  Steady load              Spiky / event-driven
                       │                          │
                [Containers/VMs]              [Serverless]


                                Domain complexity?
                       ┌──────────┴──────────┐
                  CRUD-ish                 Rich invariants
                       │                          │
              [Layered architecture]     [Hexagonal + DDD]
                                              │
                                       Audit-critical?
                                       ┌────┴────┐
                                     Yes        No
                                       │         │
                                  [+ Event   [Hexagonal
                                   Sourcing] alone]
```

---

## Quick Reference: Architecture by Use Case

| Use case | Recommended starting point |
|---|---|
| MVP / prototype | Monolith on PaaS |
| Early-stage SaaS | Modular monolith + Postgres + Redis |
| Content site / docs | Static (SSG/Islands) + headless CMS |
| E-commerce | Modular monolith → extract checkout/payments later |
| Real-time collab (Figma-like) | CSR SPA + WebSocket + CRDT/event-sourced backend |
| Internal admin tool | CSR SPA + simple REST monolith |
| High-throughput pipeline | Event-driven (Kafka) + stream processors |
| Financial / audit-heavy | Hexagonal + event sourcing + CQRS read models |
| Global low-latency app | Edge rendering + multi-region data |
| ML / data product | Pipeline (batch or streaming) + warehouse + serving layer |
| IoT ingest | Serverless ingest + queue + stream processor + warehouse |

---

## Further Reading

### Foundational
- **Sam Newman — *Building Microservices* (2nd ed., 2021)** — the definitive book; covers when *not* to use them
- **Sam Newman — *Monolith to Microservices*** — practical migration patterns
- **Gregor Hohpe & Bobby Woolf — *Enterprise Integration Patterns*** — async messaging foundation
- **Robert C. Martin — *Clean Architecture***
- **Eric Evans — *Domain-Driven Design***
- **Vaughn Vernon — *Implementing Domain-Driven Design***

### Distributed Systems
- **Martin Kleppmann — *Designing Data-Intensive Applications*** — required reading; the best book on data architectures
- **Brendan Burns — *Designing Distributed Systems***
- **Google — *Site Reliability Engineering* and *The SRE Workbook*** — free online; how Google operates at scale

### Practical / Pragmatic
- **Mark Richards & Neal Ford — *Fundamentals of Software Architecture*** — broad architectural styles
- **Mark Richards & Neal Ford — *Software Architecture: The Hard Parts*** — distributed architecture trade-offs
- **Gregor Hohpe — *The Software Architect Elevator***
- **Michael Nygard — *Release It!*** — production-readiness, circuit breakers, bulkheads
- **Jez Humble & David Farley — *Continuous Delivery***

### Web-Specific
- **Ilya Grigorik — *High Performance Browser Networking*** — free online
- **[microservices.io](https://microservices.io/)** by Chris Richardson — pattern catalog
- **[martinfowler.com](https://martinfowler.com)** — Bliki, enterprise patterns, microservices articles
- **[The Twelve-Factor App](https://12factor.net/)** — methodology for SaaS
- **[Backend Engineer Roadmap](https://roadmap.sh/backend)**

### Talks
- Sam Newman — "When to Use Microservices (and When Not To)"
- Mary Poppendieck — "The Tyranny of the Plan"
- Adrian Cockcroft (ex-Netflix) — anything on microservices
- Charity Majors (Honeycomb) — observability, on-call, distributed systems sanity

---

> **Closing thought:** there is no universally correct architecture — only the architecture that fits *this team, this domain, this stage, this constraint set, right now*. After enough projects, I've stopped memorizing architectures and started feeling the **forces** each one resolves; the work is recognizing which forces dominate today. The mark of architectural maturity isn't building the most sophisticated system — it's building the **simplest system that solves today's problem while remaining open to tomorrow's**. I've been wrong about architecture decisions more times than I'd care to count, and the architectures I'm proudest of are the ones I could change once I knew I was wrong. Plan to be wrong; build for reversibility.
