# Glossary

> Quick definitions for terms used throughout the repo, with a back-link to the guide where each one is discussed in depth.

Use the section index below to jump, or `Ctrl-F` to search.

- [Performance & Browser Internals](#performance--browser-internals)
- [Networking](#networking)
- [Rendering Strategies](#rendering-strategies)
- [System Architecture](#system-architecture)
- [Design Patterns](#design-patterns)
- [State, Data & Caching](#state-data--caching)
- [API Design](#api-design)
- [Authentication & Authorization](#authentication--authorization)
- [Security](#security)
- [Testing](#testing)
- [Observability](#observability)
- [CI/CD & Delivery](#cicd--delivery)
- [SEO](#seo)
- [Accessibility](#accessibility)
- [CSS & Design Systems](#css--design-systems)
- [CSS (language)](#css-language)
- [HTML](#html)
- [JavaScript](#javascript)
- [TypeScript](#typescript)
- [React](#react)
- [AI Integration](#ai-integration)

---

## Performance & Browser Internals

- **bfcache** *(back/forward cache)* — browser feature that restores entire pages instantly on back/forward navigation. Disabled by `unload` listeners or `Cache-Control: no-store`. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **CLS** *(Cumulative Layout Shift)* — Core Web Vital measuring unexpected visual movement of content. Target ≤ 0.1. → [Core Web Vitals](core-web-vitals.md).
- **Composite / Compositor Thread** — the GPU-side step that blends painted layers into the final image; CSS `transform` and `opacity` animations run here without touching layout. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **CRP** *(Critical Rendering Path)* — the sequence the browser walks to turn HTML/CSS/JS into pixels: DOM + CSSOM → render tree → layout → paint → composite. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **CrUX** *(Chrome User Experience Report)* — Google's public dataset of real-user Core Web Vitals from opted-in Chrome users. Powers PageSpeed Insights and Search Console. → [Core Web Vitals](core-web-vitals.md).
- **CSSOM** *(CSS Object Model)* — a tree of computed styles built from parsed CSS, joined with the DOM to produce the render tree. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **CWV** *(Core Web Vitals)* — Google's three user-experience metrics: LCP, INP, and CLS. Used as a search-ranking signal. → [Core Web Vitals](core-web-vitals.md).
- **DOM** *(Document Object Model)* — the parsed tree of HTML nodes the browser builds from incoming bytes. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **FCP** *(First Contentful Paint)* — when the first text or image appears. Diagnostic metric, not a Core Web Vital. → [Core Web Vitals](core-web-vitals.md).
- **INP** *(Interaction to Next Paint)* — Core Web Vital measuring the worst latency from user input to next paint. Replaced FID in 2024. Target ≤ 200ms. → [Core Web Vitals](core-web-vitals.md).
- **Layout / Reflow** — the browser step that recomputes element geometry; expensive and cascading. Triggered by changes to width, height, top, font-size, etc. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Layout Thrashing** — the anti-pattern of interleaving DOM writes and reads in a loop, forcing synchronous layout on every iteration. → [Browser Rendering Pipeline](browser-rendering-pipeline.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **LCP** *(Largest Contentful Paint)* — Core Web Vital measuring when the largest visible content paints. Target ≤ 2.5s. Often gated by image load delay. → [Core Web Vitals](core-web-vitals.md).
- **Lighthouse** — Google's lab performance auditing tool; runs in DevTools and CI. Measures lab proxies (TBT) for the field metric INP. → [Core Web Vitals](core-web-vitals.md).
- **Long Task** — any main-thread task lasting more than 50ms; blocks input and degrades INP. → [Core Web Vitals](core-web-vitals.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Paint / Repaint** — the step that rasterizes pixels for visible content. Cheaper than layout but still costs CPU on large layers. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Preload Scanner** — speculative parser that finds resources in HTML ahead of the main parser, even when JS is blocking it. Defeated by CSS-only references and JS-injected scripts. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **rAF** *(`requestAnimationFrame`)* — schedules work just before the next paint; the right hook for animations and pre-paint reads/writes. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Render Tree** — the merged DOM + CSSOM, with `display:none` nodes excluded; the input to layout. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **RUM** *(Real User Monitoring)* — telemetry collected from actual user sessions in production, in contrast to lab metrics. → [Observability](observability.md), [Core Web Vitals](core-web-vitals.md).
- **Speculation Rules API** — newer browser API for prerendering or prefetching likely-next pages with declarative rules. Enables instant navigation. → [Web Optimization Techniques](web-optimization-techniques.md).
- **TBT** *(Total Blocking Time)* — lab proxy for INP; the sum of long-task blocking time after FCP. → [Core Web Vitals](core-web-vitals.md).
- **TTFB** *(Time to First Byte)* — time from request start to first response byte. Caps the achievable LCP. → [Core Web Vitals](core-web-vitals.md).
- **TTI** *(Time to Interactive)* — older metric for "page is responsive"; largely superseded by INP in CWV. → [Core Web Vitals](core-web-vitals.md).
- **View Transitions API** — native API for animating between DOM states (and across documents). Lets MPAs feel like SPAs. → [CSS & Design Systems](css-and-design-systems.md), [Web Rendering Strategies](web-rendering-strategies.md).
- **`will-change`** — CSS hint that promotes an element to its own compositor layer ahead of an animation. Costs GPU memory; use sparingly. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).

---

## Networking

- **0-RTT** — TLS 1.3 / QUIC feature where a returning client can send the first request inside the handshake; only safe for idempotent ops due to replay risk. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Brotli / gzip** — text-compression algorithms. Brotli is ~20% smaller than gzip and the modern default. Pre-compress static assets at level 11. → [Web Optimization Techniques](web-optimization-techniques.md).
- **CDN** *(Content Delivery Network)* — globally distributed cache that serves content from a PoP close to the user. → [Web Optimization Techniques](web-optimization-techniques.md), [Web Architectures](web-architectures.md).
- **DNS** *(Domain Name System)* — name-to-IP lookup, the first step in any network request. Cached at multiple layers. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Edge / Edge Function** — code that runs on CDN nodes physically close to users (Cloudflare Workers, Vercel Edge). Sub-50ms TTFB globally. → [Web Architectures](web-architectures.md), [Web Rendering Strategies](web-rendering-strategies.md).
- **ETag** — opaque versioning identifier on cacheable responses; sent back as `If-None-Match` for conditional revalidation. → [Web Optimization Techniques](web-optimization-techniques.md), [API Design](api-design.md).
- **HSTS** *(HTTP Strict Transport Security)* — header that forces HTTPS for a duration; preloaded HSTS protects the very first request. → [Browser Rendering Pipeline](browser-rendering-pipeline.md), [Web Application Security](web-application-security.md).
- **HTTP/2** — multiplexed protocol over TCP that eliminates head-of-line blocking within a connection. Universal in 2026. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **HTTP/3** — newer protocol over **QUIC** (UDP) with stream-independent multiplexing and 0-RTT resumption. Faster on lossy networks. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **mTLS** *(mutual TLS)* — both client and server present certificates. Used for authenticated service-to-service communication. → [Web Application Security](web-application-security.md).
- **OCSP Stapling** — server attaches a recent certificate-revocation status to its TLS handshake, avoiding a per-client OCSP round-trip. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Preconnect / Preload / Prefetch** — resource hints that warm DNS+TCP+TLS, prioritize a known critical resource, or fetch likely-next content. → [Web Optimization Techniques](web-optimization-techniques.md).
- **QUIC** — UDP-based transport that combines transport + crypto handshake; the foundation of HTTP/3. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **SSE** *(Server-Sent Events)* — one-way server-to-client push over a long-lived HTTP connection. Common for LLM streaming and live feeds. → [API Design](api-design.md), [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **stale-while-revalidate** — `Cache-Control` directive that lets caches serve stale content while refetching in the background. Excellent UX trade-off. → [Web Optimization Techniques](web-optimization-techniques.md), [API Design](api-design.md).
- **TCP 3-way Handshake** — SYN → SYN-ACK → ACK; costs 1 RTT before any data flows. → [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **TLS** *(Transport Layer Security)* — encryption layer beneath HTTPS. TLS 1.3 is the modern default; supports 1-RTT and 0-RTT handshakes. → [Browser Rendering Pipeline](browser-rendering-pipeline.md), [Web Application Security](web-application-security.md).
- **WebSocket** — persistent, full-duplex connection for real-time bidirectional messaging. → [API Design](api-design.md).

---

## Rendering Strategies

- **Client-Side Rendering (CSR)** — server returns a near-empty HTML shell; JS renders the UI in the browser. Worst LCP; weakest SEO. → [Web Rendering Strategies](web-rendering-strategies.md).
- **Edge Rendering** — SSR run on CDN-edge runtimes; sub-50ms TTFB worldwide, with limited Node compatibility. → [Web Rendering Strategies](web-rendering-strategies.md).
- **Hybrid Rendering** — picking a strategy *per route* in the same codebase (marketing static, dashboard SSR, search streamed). → [Web Rendering Strategies](web-rendering-strategies.md).
- **Hydration** — the post-SSR step where client-side React (or similar) attaches event listeners and reactivity to server-rendered HTML. → [Web Rendering Strategies](web-rendering-strategies.md).
- **ISR** *(Incremental Static Regeneration)* — pre-render at build time, then revalidate on a TTL or on-demand. Stale-while-revalidate for static sites. → [Web Rendering Strategies](web-rendering-strategies.md).
- **Islands Architecture** — mostly static HTML with isolated interactive widgets ("islands") that hydrate independently. Pioneered by Astro. → [Web Rendering Strategies](web-rendering-strategies.md).
- **MPA** *(Multi-Page Application)* — each navigation triggers a full document request; the browser owns scroll, focus, and history. → [Web Rendering Strategies](web-rendering-strategies.md).
- **PPR** *(Partial Prerendering)* — Next.js pattern where the static shell is prerendered while dynamic holes stream in at request time. → [Web Rendering Strategies](web-rendering-strategies.md).
- **Resumability** — Qwik's approach: serialize execution state into HTML so the client resumes (no hydration cost). → [Web Rendering Strategies](web-rendering-strategies.md).
- **RSC** *(React Server Components)* — components that render on the server and ship zero JS to the client by default. → [Web Rendering Strategies](web-rendering-strategies.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **SPA** *(Single-Page Application)* — one HTML doc + client routing; subsequent navigations swap UI without full reload. → [Web Rendering Strategies](web-rendering-strategies.md).
- **SSG** *(Static Site Generation)* — pre-render HTML for every route at build time; serve from CDN. Fastest TTFB, cheapest hosting. → [Web Rendering Strategies](web-rendering-strategies.md).
- **SSR** *(Server-Side Rendering)* — render HTML on each request, then hydrate. Per-request personalization with full SEO. → [Web Rendering Strategies](web-rendering-strategies.md).
- **Streaming SSR** — server flushes HTML in chunks via `<Suspense>` boundaries; first byte ships before all data resolves. → [Web Rendering Strategies](web-rendering-strategies.md).

---

## System Architecture

- **BFF** *(Backend-for-Frontend)* — a thin server tailored to one frontend, aggregating microservices and shaping data for that UI. → [Web Architectures](web-architectures.md), [API Design](api-design.md).
- **Clean / Onion Architecture** — concentric rings of dependency from domain core outward; outer rings depend inward only. → [Web Architectures](web-architectures.md).
- **Conway's Law** — "organizations design systems that mirror their communication structure." A sociotechnical iron rule. → [Web Architectures](web-architectures.md).
- **CQRS** *(Command Query Responsibility Segregation)* — separate write models from read models, often paired with event sourcing. → [Web Architectures](web-architectures.md), [Web Design Patterns](web-design-patterns.md).
- **Distributed Monolith** — multiple services that must deploy together and share a database; the worst-of-both-worlds anti-pattern. → [Web Architectures](web-architectures.md).
- **Event Sourcing** — state is the result of replaying an immutable log of events; enables audit trails and time travel. → [Web Architectures](web-architectures.md).
- **Event-Driven Architecture (EDA)** — services communicate by emitting and reacting to events through a broker (Kafka, NATS, SQS). → [Web Architectures](web-architectures.md).
- **Hexagonal / Ports & Adapters** — domain core surrounded by ports (interfaces) implemented by adapters (DB, web, etc.). Highly testable. → [Web Architectures](web-architectures.md).
- **JAMstack** — JavaScript + APIs + Markup. Pre-built markup served from CDN, dynamic via JS calling APIs. → [Web Architectures](web-architectures.md).
- **Micro-frontends** — multiple independently-deployable frontend apps composed at runtime or build time. Heavy operational tax. → [Web Architectures](web-architectures.md).
- **Microservices** — independently deployable services, each owning its own data, communicating over the network. → [Web Architectures](web-architectures.md).
- **Modular Monolith** — a monolith with strict internal module boundaries (private DB tables, public APIs only). The underrated default. → [Web Architectures](web-architectures.md).
- **Monolith** — single deployable containing all business logic. The right default for most teams under 50 engineers. → [Web Architectures](web-architectures.md).
- **Outbox Pattern** — write a message to an outbox table in the same DB transaction as the state change; a separate process publishes it. Reliable at-least-once events. → [Web Architectures](web-architectures.md).
- **Serverless / FaaS** — functions running in a managed runtime that scales to zero; pay per invocation. → [Web Architectures](web-architectures.md).
- **Saga** — long-running distributed transaction implemented as a sequence of local transactions with compensating actions. → [Web Architectures](web-architectures.md).
- **SLSA** *(Supply-chain Levels for Software Artifacts)* — graduated framework for build integrity and provenance. → [Web Application Security](web-application-security.md), [CI/CD](ci-cd.md).
- **SOA** *(Service-Oriented Architecture)* — coarser-grained services around business capabilities, often with an enterprise service bus. → [Web Architectures](web-architectures.md).

---

## Design Patterns

- **Adapter** — wrap an interface to match another. Useful for integrating third-party libraries with your own idioms. → [Web Design Patterns](web-design-patterns.md).
- **Atomic Design** — Atoms → Molecules → Organisms → Templates → Pages. Vocabulary for design systems. → [Web Design Patterns](web-design-patterns.md), [CSS & Design Systems](css-and-design-systems.md).
- **Bulkhead** — isolate resources so one slow dependency can't exhaust pools used by others. → [Web Design Patterns](web-design-patterns.md).
- **Circuit Breaker** — after N consecutive failures, stop calling the failing service; periodically test before reopening. → [Web Design Patterns](web-design-patterns.md), [Web Application Security](web-application-security.md).
- **Compound Components** — a parent component implicitly shares state with its children via context, letting consumers compose freely (e.g., `<Tabs><Tabs.Trigger/><Tabs.Panel/></Tabs>`). → [Web Design Patterns](web-design-patterns.md).
- **Container / Presentational** — separating data-fetching components from pure render components. Less idiomatic with hooks, still a useful mental model. → [Web Design Patterns](web-design-patterns.md).
- **Custom Hook** — encapsulate stateful logic in a named function for reuse. The modern way to share logic in React. → [Web Design Patterns](web-design-patterns.md).
- **Decorator** — wrap an object/function to add behavior without modifying it. → [Web Design Patterns](web-design-patterns.md).
- **Facade** — simplified interface over a complex subsystem. → [Web Design Patterns](web-design-patterns.md).
- **Headless Component** — manages state and behavior but renders minimal markup; the consumer brings the UI (Radix, React Aria, Headless UI). → [Web Design Patterns](web-design-patterns.md), [CSS & Design Systems](css-and-design-systems.md).
- **HOC** *(Higher-Order Component)* — function that takes a component and returns a wrapped one. Mostly superseded by hooks. → [Web Design Patterns](web-design-patterns.md).
- **Mediator** — central coordinator that reduces many-to-many coupling to one-to-many. → [Web Design Patterns](web-design-patterns.md).
- **Observer / Pub-Sub** — subjects emit events; observers subscribe. Loose coupling at the cost of explicit flow. → [Web Design Patterns](web-design-patterns.md).
- **Proxy** — stand-in that controls access to another object. Powers MobX/Vue/Valtio reactivity. → [Web Design Patterns](web-design-patterns.md).
- **Render Props** — component calls a function passed as a prop, providing render arguments. Largely replaced by hooks. → [Web Design Patterns](web-design-patterns.md).
- **Repository** — abstraction over data sources presenting a domain-shaped API. → [Web Design Patterns](web-design-patterns.md).
- **State Machine** — explicitly model states + transitions (XState). Eliminates impossible boolean combinations. → [Web Design Patterns](web-design-patterns.md).
- **Strategy** — interchangeable algorithms behind a common interface. → [Web Design Patterns](web-design-patterns.md).

---

## State, Data & Caching

- **DataLoader** — Facebook's batching/caching library for resolvers; the canonical fix for the N+1 problem. → [API Design](api-design.md).
- **Fetch-on-Render** — component mounts, then fetches; causes waterfalls with nested data. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **N+1 Problem** — fetching a list, then one query per item; cumulatively slow. Fixed with batching, joins, or DataLoader. → [API Design](api-design.md).
- **Optimistic UI** — update UI immediately on user action; reconcile with server response; rollback on error. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Polyglot Persistence** — using different storage engines for different access patterns (Postgres + Redis + Elasticsearch + S3). → [Web Architectures](web-architectures.md).
- **Read Replica** — DB replica serving read-only traffic; offloads the primary at the cost of replication lag. → [Web Architectures](web-architectures.md).
- **Render-as-You-Fetch** — start fetches outside render and read results via Suspense; eliminates waterfalls. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Server State vs Client State** — server state is data cached from the server (lists, profiles); client state is ephemeral UI state (modal open, theme). Treat them differently. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Suspense** — React's primitive for declarative async boundaries; resolves with a fallback while children load. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **TanStack Query / SWR / RTK Query** — server-state libraries that handle caching, deduplication, revalidation, and mutations. → [Web Design Patterns](web-design-patterns.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Virtualization / Windowing** — render only visible list items; recycle DOM nodes as the user scrolls. Mandatory for long lists. → [Web Optimization Techniques](web-optimization-techniques.md).
- **Zustand / Jotai / Valtio** — modern state libraries built on signals or proxies, with finer-grained reactivity than Redux. → [Web Design Patterns](web-design-patterns.md).

---

## API Design

- **BFF Pattern** — see [System Architecture](#system-architecture). → [API Design](api-design.md).
- **Connect / ConnectRPC** — gRPC-compatible protocol that works natively over HTTP/1.1 + HTTP/2 with JSON or Protobuf. Browser-friendly. → [API Design](api-design.md).
- **Cursor Pagination** — opaque token returned with each page; the modern default for large or live datasets. Beats offset/limit at depth. → [API Design](api-design.md).
- **GraphQL** — query language where the client specifies the response shape; one endpoint, strong typing. → [API Design](api-design.md).
- **gRPC** — binary RPC over HTTP/2 with Protobuf schemas. Fast and strongly typed; needs a proxy for browsers. → [API Design](api-design.md).
- **HATEOAS** *(Hypermedia as the Engine of Application State)* — REST level 3; responses carry links to next actions. Rare in modern APIs. → [API Design](api-design.md).
- **Hyrum's Law** — "all observable behaviors of your system will be depended on by somebody." Plan accordingly when changing APIs. → [API Design](api-design.md).
- **Idempotency Key** — client-supplied unique header per logical operation; server deduplicates retries. Essential for safe retries on POST. → [API Design](api-design.md), [Web Application Security](web-application-security.md).
- **JSON Merge Patch** *(RFC 7396)* — partial-update format where `null` deletes; missing fields are unchanged. Simple PATCH semantics. → [API Design](api-design.md).
- **JSON Patch** *(RFC 6902)* — array of explicit `op`/`path`/`value` operations. More expressive, more verbose. → [API Design](api-design.md).
- **OpenAPI** *(formerly Swagger)* — schema spec for REST APIs. Generates docs, SDKs, mocks, and contract tests. → [API Design](api-design.md).
- **Problem Details** *(RFC 9457, formerly RFC 7807)* — standard error shape for HTTP APIs (`type`, `title`, `status`, `detail`, `instance`). → [API Design](api-design.md).
- **REST** — resource-oriented architectural style over HTTP, leaning on verbs and status codes. → [API Design](api-design.md).
- **tRPC** — TypeScript-only end-to-end typed RPC for monorepos. Refactor-safe; not for cross-language consumers. → [API Design](api-design.md).
- **ULID / UUIDv7** — lexicographically sortable, time-prefixed identifiers. Modern alternatives to random UUIDs. → [API Design](api-design.md).
- **Webhook** — outbound HTTP callback from your server to a subscriber URL. Inverse of REST. → [API Design](api-design.md).

---

## Authentication & Authorization

- **ABAC** *(Attribute-Based Access Control)* — decisions based on user, resource, action, and environment attributes. → [Web Application Security](web-application-security.md).
- **AuthN vs AuthZ** — Authentication = who you are. Authorization = what you're allowed to do. Always re-check authz at the resource level. → [Web Application Security](web-application-security.md).
- **Bearer Token** — credential transmitted in `Authorization: Bearer ...`; the holder is granted access. → [Web Application Security](web-application-security.md), [API Design](api-design.md).
- **HttpOnly Cookie** — cookie inaccessible to JavaScript; defends session tokens against XSS exfiltration. → [Web Application Security](web-application-security.md).
- **JWT** *(JSON Web Token)* — self-contained signed token with claims. Powerful but widely overused; revocation is the hard part. → [Web Application Security](web-application-security.md).
- **MFA / 2FA** *(Multi-Factor / Two-Factor Authentication)* — requiring two or more independent factors (knowledge, possession, inherence). → [Web Application Security](web-application-security.md).
- **OAuth 2.0** — authorization framework for delegating access to a third party. Use Authorization Code + PKCE for SPAs and mobile. → [Web Application Security](web-application-security.md).
- **OIDC** *(OpenID Connect)* — identity layer on top of OAuth 2.0; the standard for "Sign in with…". → [Web Application Security](web-application-security.md), [CI/CD](ci-cd.md).
- **Passkey / WebAuthn** — phishing-resistant, public-key-based authentication backed by device biometrics. The new gold standard. → [Web Application Security](web-application-security.md).
- **PKCE** *(Proof Key for Code Exchange)* — OAuth extension that prevents code-interception attacks for public clients (SPAs, mobile). → [Web Application Security](web-application-security.md).
- **RBAC** *(Role-Based Access Control)* — users have roles; roles have permissions. → [Web Application Security](web-application-security.md).
- **ReBAC** *(Relationship-Based Access Control)* — decisions based on relationships ("is Alice a viewer of folder X?"). Inspired by Google Zanzibar. → [Web Application Security](web-application-security.md).
- **SameSite Cookie** — `Strict` / `Lax` / `None` attribute controlling whether cookies are sent on cross-site requests. `Lax` is the modern default; mitigates CSRF. → [Web Application Security](web-application-security.md).
- **TOTP** *(Time-Based One-Time Password)* — authenticator-app codes (RFC 6238). Better than SMS but inferior to passkeys. → [Web Application Security](web-application-security.md).

---

## Security

- **Argon2id** — password-hashing KDF; the modern recommended default. → [Web Application Security](web-application-security.md).
- **BOLA** *(Broken Object Level Authorization)* — OWASP API #1; authenticated user accesses someone else's resource because authz wasn't re-checked. Same idea as IDOR. → [Web Application Security](web-application-security.md), [API Design](api-design.md).
- **Clickjacking** — embedding your site in a hidden iframe to trick users into clicking. Defended with `X-Frame-Options` or `frame-ancestors`. → [Web Application Security](web-application-security.md).
- **COOP / COEP / CORP** — Cross-Origin headers that isolate browsing contexts and restrict embedding. Required for `SharedArrayBuffer`. → [Web Application Security](web-application-security.md).
- **CORS** *(Cross-Origin Resource Sharing)* — browser mechanism that *relaxes* the same-origin policy. Not a server-side security control. → [Web Application Security](web-application-security.md), [API Design](api-design.md).
- **CSP** *(Content Security Policy)* — header that whitelists allowed sources for scripts, styles, frames, etc. Highest-leverage XSS defense. → [Web Application Security](web-application-security.md).
- **CSRF** *(Cross-Site Request Forgery)* — attacker tricks an authenticated browser into sending a state-changing request. Defenses: SameSite cookies + CSRF tokens. → [Web Application Security](web-application-security.md).
- **DAST** *(Dynamic Application Security Testing)* — black-box scanning of a running app (OWASP ZAP, Burp). → [Web Testing Strategy](web-testing-strategy.md), [Web Application Security](web-application-security.md).
- **HMAC** *(Hash-based Message Authentication Code)* — keyed hash for verifying message integrity and authenticity. Used in webhook signing. → [API Design](api-design.md), [Web Application Security](web-application-security.md).
- **IDOR** *(Insecure Direct Object Reference)* — same as BOLA. Most common authorization bug. → [Web Application Security](web-application-security.md).
- **KMS / HSM** *(Key Management Service / Hardware Security Module)* — managed key storage where keys never leave hardware. → [Web Application Security](web-application-security.md).
- **Mass Assignment** — sending request fields like `is_admin: true` and the server blindly applying them. Defense: explicit allowlists / DTOs. → [Web Application Security](web-application-security.md), [API Design](api-design.md).
- **OWASP Top 10** — community-curated list of most critical web app risks. There's also an API-specific Top 10. → [Web Application Security](web-application-security.md).
- **Prototype Pollution** — JS-specific attack that mutates `Object.prototype` via untrusted merge. → [Web Application Security](web-application-security.md).
- **SAST** *(Static Application Security Testing)* — code-analysis scanners (Semgrep, CodeQL, Snyk Code). → [Web Testing Strategy](web-testing-strategy.md), [Web Application Security](web-application-security.md).
- **SBOM** *(Software Bill of Materials)* — inventory of every dependency in an artifact (CycloneDX, SPDX). Required by US EO 14028. → [Web Application Security](web-application-security.md), [CI/CD](ci-cd.md).
- **Sigstore / cosign** — open-source toolchain for signing and verifying artifacts. → [Web Application Security](web-application-security.md), [CI/CD](ci-cd.md).
- **SQLi** *(SQL Injection)* — untrusted input interpreted as SQL. Defense: parameterized queries (and *only* parameterized queries). → [Web Application Security](web-application-security.md).
- **SSRF** *(Server-Side Request Forgery)* — server fetches an attacker-controlled URL, often to access internal services or cloud metadata. → [Web Application Security](web-application-security.md).
- **STRIDE** — Microsoft's threat-modeling framework: Spoofing, Tampering, Repudiation, Information disclosure, DoS, Elevation of privilege. → [Web Application Security](web-application-security.md).
- **Timing-Safe Comparison** — constant-time equality check for secrets to prevent timing-attack leakage (`crypto.timingSafeEqual`). → [Web Application Security](web-application-security.md).
- **Trusted Types** — browser API + CSP directive that requires sanitization at DOM-injection sinks. Strong XSS hardening. → [Web Application Security](web-application-security.md).
- **XSS** *(Cross-Site Scripting)* — attacker injects executable JS into a page another user views. Stored, reflected, and DOM-based variants. → [Web Application Security](web-application-security.md).

---

## Testing

- **ATDD** *(Acceptance TDD)* — tests written from the user's perspective, often in collaboration with PM/QA, as definition-of-done. → [Web Testing Strategy](web-testing-strategy.md).
- **BDD** *(Behavior-Driven Development)* — TDD with collaborative-language descriptions (`Given/When/Then`). → [Web Testing Strategy](web-testing-strategy.md).
- **Component Test** — render a single component with realistic interactions; the cheap-but-confident middle ground. → [Web Testing Strategy](web-testing-strategy.md).
- **Contract Test** — verifies that two services agree on a request/response schema. Pact is the consumer-driven standard. → [Web Testing Strategy](web-testing-strategy.md).
- **Coverage** — % of statements / branches / lines / functions executed by tests. A ceiling, not a floor. → [Web Testing Strategy](web-testing-strategy.md).
- **E2E Test** — drives a real browser against a real backend through the same surface as a user. Slow; reserve for critical journeys. → [Web Testing Strategy](web-testing-strategy.md).
- **Flaky Test** — passes inconsistently. Worse than no test; root-cause every one. → [Web Testing Strategy](web-testing-strategy.md).
- **Integration Test** — exercises multiple units together (component + hook + reducer + mocked API). The trophy-shape sweet spot. → [Web Testing Strategy](web-testing-strategy.md).
- **MSW** *(Mock Service Worker)* — mock HTTP at the network layer (instead of mocking functions). Refactor-safe, realistic. → [Web Testing Strategy](web-testing-strategy.md).
- **Mutation Test** — tooling deliberately changes code (`+` → `-`) and re-runs tests; surviving mutants reveal weak assertions. Stryker is the standard. → [Web Testing Strategy](web-testing-strategy.md).
- **Property-Based Test** — specify invariants; the framework generates random inputs (fast-check, QuickCheck). Finds edge cases you'd never write by hand. → [Web Testing Strategy](web-testing-strategy.md).
- **Snapshot Test** — capture serialized output and compare on re-run. Useful for structured outputs; misused for components. → [Web Testing Strategy](web-testing-strategy.md).
- **TDD** *(Test-Driven Development)* — Red → Green → Refactor. Its real payoff is design feedback, not bug catching. → [Web Testing Strategy](web-testing-strategy.md).
- **Test Doubles** — Dummy / Stub / Spy / Mock / Fake (Meszaros). Mock at the architectural boundary, not at every function call. → [Web Testing Strategy](web-testing-strategy.md).
- **Test Pyramid / Trophy / Honeycomb** — three competing shapes for how to weight unit / integration / E2E. Trophy/honeycomb suit modern frontend; pyramid suits server-heavy systems. → [Web Testing Strategy](web-testing-strategy.md).
- **TIA** *(Test Impact Analysis)* — only run tests affected by a change. Nx, Turborepo, Bazel support this. → [Web Testing Strategy](web-testing-strategy.md), [CI/CD](ci-cd.md).
- **Visual Regression** — screenshot diff testing (Chromatic, Percy, Playwright `toHaveScreenshot`). Best at component level. → [Web Testing Strategy](web-testing-strategy.md).

---

## Observability

- **bfcache** — see [Performance & Browser Internals](#performance--browser-internals).
- **Cardinality** — number of unique label combinations in a metric. High cardinality = high cost in time-series DBs. → [Observability](observability.md).
- **Continuous Profiling** — sampled CPU/memory profiles in production at low overhead (Pyroscope, Datadog). The fourth pillar. → [Observability](observability.md).
- **DORA Metrics** — Deployment Frequency, Lead Time, MTTR, Change Failure Rate. Plus reliability since 2021. → [CI/CD](ci-cd.md), [Observability](observability.md).
- **Error Budget** — `1 - SLO`. The amount of "bad" you can afford in a window; when exhausted, freeze risky changes. → [Observability](observability.md).
- **Four Golden Signals** — Latency, Traffic, Errors, Saturation. Google SRE's user-facing instrumentation primer. → [Observability](observability.md).
- **MTTR** *(Mean Time to Restore)* — average time to recover from incidents. The metric that separates elite from low DORA performers. → [Observability](observability.md), [CI/CD](ci-cd.md).
- **OpenTelemetry (OTel)** — CNCF-graduated, vendor-neutral standard for instrumenting code with logs, metrics, and traces. → [Observability](observability.md).
- **p50 / p95 / p99** — latency percentiles. Always look at the tail; averages hide the users in pain. → [Observability](observability.md).
- **Postmortem (Blameless)** — incident retrospective focused on systems, not individuals. Action items have owners and deadlines. → [Observability](observability.md).
- **RED Method** — for services: Rate, Errors, Duration. → [Observability](observability.md).
- **SLI / SLO / SLA** — Indicator (metric), Objective (internal target), Agreement (contractual). → [Observability](observability.md).
- **Span / Trace** — a span is one unit of work; a trace is the causal chain of spans across services. Connected by a shared `trace_id`. → [Observability](observability.md).
- **Structured Logging** — log JSON, not strings. Enables querying by user_id, trace_id, etc. → [Observability](observability.md).
- **Synthetic Monitoring** — scheduled scripts simulating user journeys from external locations. Complements RUM. → [Observability](observability.md).
- **USE Method** — for resources: Utilization, Saturation, Errors. → [Observability](observability.md).
- **W3C Trace Context** — standard `traceparent` / `tracestate` HTTP headers for cross-service context propagation. → [Observability](observability.md).

---

## CI/CD & Delivery

- **ADR** *(Architecture Decision Record)* — short document recording an architectural decision with context, options, and consequences. → [Web Architectures](web-architectures.md).
- **Argo CD / Flux** — Kubernetes GitOps controllers that reconcile cluster state to a Git repo. → [CI/CD](ci-cd.md).
- **Blue/Green Deployment** — deploy alongside the running version; switch traffic atomically. Instant rollback. → [CI/CD](ci-cd.md).
- **Canary Deployment** — route a small % of traffic to the new version; promote on healthy metrics. → [CI/CD](ci-cd.md).
- **CD** *(Continuous Delivery / Deployment)* — Delivery = every change is deployable on demand. Deployment = every change deploys automatically. → [CI/CD](ci-cd.md).
- **CI** *(Continuous Integration)* — engineers merge to a shared mainline frequently, with automated tests gating each merge. → [CI/CD](ci-cd.md).
- **Distributed Monolith** — see [System Architecture](#system-architecture). → [Web Architectures](web-architectures.md).
- **Expand-and-Contract Migration** — phase a breaking schema change through additive expand → backfill → switchover → contract. Zero downtime. → [CI/CD](ci-cd.md).
- **Feature Flag** — runtime switch that controls whether a feature is on for a request/user. Decouples deploy from release. → [CI/CD](ci-cd.md).
- **GitOps** — desired system state lives in Git; controllers reconcile reality to match. Auditable, self-healing. → [CI/CD](ci-cd.md).
- **IaC** *(Infrastructure as Code)* — infrastructure defined declaratively in version-controlled code (Terraform, Pulumi, CDK, Bicep). → [CI/CD](ci-cd.md).
- **OIDC Federation** *(in CI)* — CI runners assume cloud roles via short-lived OIDC tokens, eliminating long-lived secrets. → [CI/CD](ci-cd.md), [Web Application Security](web-application-security.md).
- **Progressive Delivery** — gradually rolling out a feature via flags, percentages, segments, or canaries with automated rollback on regression. → [CI/CD](ci-cd.md).
- **Rolling Deployment** — replace instances incrementally. Built into Kubernetes; simpler than blue/green but slower to roll back. → [CI/CD](ci-cd.md).
- **Shadow / Mirror Deployment** — duplicate real traffic to a new version while discarding its responses. Real-load test without user risk. → [CI/CD](ci-cd.md).
- **Trunk-Based Development** — short-lived branches merged to `main` daily, behind feature flags. Strongly correlated with elite DORA performance. → [CI/CD](ci-cd.md).
- **12-Factor App** — methodology for portable, scalable SaaS apps (config in env, stateless processes, etc.). → [CI/CD](ci-cd.md).

---

## SEO

- **AI Overviews / GEO** — Google's AI-generated answer panels and the practice of optimizing content to be cited by them. → [SEO](seo.md).
- **Canonical URL** — `<link rel="canonical">` declaring which URL is the authoritative copy when duplicates exist. → [SEO](seo.md).
- **Crawl Budget** — finite number of URLs a search engine will crawl on your site per period. Critical for very large sites. → [SEO](seo.md).
- **E-E-A-T** *(Experience, Expertise, Authoritativeness, Trustworthiness)* — Google's quality framework for content. → [SEO](seo.md).
- **hreflang** — tag declaring language/region variants so engines serve the right one to the right users. → [SEO](seo.md).
- **JSON-LD** — JSON-formatted structured data; the recommended format for Schema.org markup. → [SEO](seo.md).
- **Mobile-First Indexing** — Google indexes the mobile version of your site as primary. → [SEO](seo.md).
- **`robots.txt`** — controls crawling, not indexing. Disallowed pages can still be indexed if linked elsewhere. → [SEO](seo.md).
- **Schema.org** — vocabulary of types (`Article`, `Product`, `FAQPage`) used by search engines for rich results. → [SEO](seo.md).
- **SERP** — Search Engine Results Page. → [SEO](seo.md).
- **Sitemap** — XML file listing your site's URLs for crawlers. → [SEO](seo.md).
- **TTFB** — see [Performance & Browser Internals](#performance--browser-internals). Caps achievable LCP. → [SEO](seo.md).

---

## Accessibility

- **A11y** *(accessibility)* — "a" + 11 letters + "y". The conventional shorthand. → [Web Accessibility](web-accessibility.md).
- **ARIA** *(Accessible Rich Internet Applications)* — spec for adding semantics to dynamic content where HTML alone falls short. First rule: don't use ARIA if a native element suffices. → [Web Accessibility](web-accessibility.md).
- **Focus Trap** — keep keyboard focus inside a modal until it closes. Mandatory for accessible dialogs. → [Web Accessibility](web-accessibility.md).
- **`inert` Attribute** — marks a subtree as non-focusable and hidden from assistive tech, without breaking layout. → [Web Accessibility](web-accessibility.md), [CSS & Design Systems](css-and-design-systems.md).
- **Live Region** — `aria-live="polite|assertive"` element that announces dynamic content changes to screen readers. → [Web Accessibility](web-accessibility.md).
- **POUR Principles** — Perceivable, Operable, Understandable, Robust. The four pillars of WCAG. → [Web Accessibility](web-accessibility.md).
- **Roving tabindex** — keyboard pattern where only one item in a composite widget is tabbable; arrow keys move focus among the rest. → [Web Accessibility](web-accessibility.md).
- **Skip Link** — first-tab-stop link that jumps past navigation to main content. Mandatory for keyboard users. → [Web Accessibility](web-accessibility.md).
- **`sr-only`** — visually-hidden but screen-reader-readable utility class. Implemented with `clip` / `position: absolute`. → [Web Accessibility](web-accessibility.md), [CSS & Design Systems](css-and-design-systems.md).
- **VPAT** *(Voluntary Product Accessibility Template)* — document declaring conformance to accessibility standards. Required by many enterprise buyers. → [Web Accessibility](web-accessibility.md).
- **WCAG** *(Web Content Accessibility Guidelines)* — international accessibility standard; AA is the common target. → [Web Accessibility](web-accessibility.md).

---

## CSS & Design Systems

- **`@layer`** — CSS cascade layers. Explicit ordering replaces specificity wars and `!important`. → [CSS & Design Systems](css-and-design-systems.md).
- **`:has()`** — parent selector. Style an element based on its descendants (`.card:has(img) { padding: 0 }`). → [CSS & Design Systems](css-and-design-systems.md).
- **`:where()`** — same matching as `:is()` but specificity 0; useful for overridable defaults. → [CSS & Design Systems](css-and-design-systems.md).
- **BEM** *(Block, Element, Modifier)* — CSS naming convention: `.block__element--modifier`. → [CSS & Design Systems](css-and-design-systems.md).
- **Cascade Layers** — see [`@layer`](#-layer). → [CSS & Design Systems](css-and-design-systems.md).
- **Container Queries** — style components based on their container's size, not the viewport. → [CSS & Design Systems](css-and-design-systems.md).
- **CSS-in-JS (runtime)** — styled-components, Emotion. Runtime cost has driven the modern shift to zero-runtime alternatives. → [CSS & Design Systems](css-and-design-systems.md).
- **CUBE CSS** — Composition + Utility + Block + Exception methodology. → [CSS & Design Systems](css-and-design-systems.md).
- **Design Tokens** — named visual values (color, spacing, radius). Three-tier hierarchy: primitive → semantic → component. → [CSS & Design Systems](css-and-design-systems.md).
- **`dvh` / `lvh` / `svh`** — dynamic / large / small viewport-height units that account for mobile UI showing/hiding. Replace `vh`. → [CSS & Design Systems](css-and-design-systems.md).
- **`font-display`** — `swap` / `optional` / `block` controls how the browser handles loading fonts. Defends LCP and CLS. → [CSS & Design Systems](css-and-design-systems.md).
- **ITCSS** *(Inverted Triangle CSS)* — layered architecture from low- to high-specificity (Settings → Tools → Generic → Elements → Objects → Components → Utilities). → [CSS & Design Systems](css-and-design-systems.md).
- **`light-dark()`** — CSS function that returns one value for light scheme, another for dark. Pairs with `color-scheme`. → [CSS & Design Systems](css-and-design-systems.md).
- **Logical Properties** — direction-agnostic counterparts (`margin-inline-start`, `padding-block`) that work in LTR and RTL. → [CSS & Design Systems](css-and-design-systems.md).
- **OOCSS** *(Object-Oriented CSS)* — separate structure from skin (`.btn` + `.btn--primary`). → [CSS & Design Systems](css-and-design-systems.md).
- **`oklch` / `oklab`** — perceptually uniform color spaces that produce smooth gradients without dead zones. → [CSS & Design Systems](css-and-design-systems.md).
- **shadcn/ui** — pattern of copying headless components into your repo and customizing with Tailwind + tokens. The 2026 dominant approach. → [CSS & Design Systems](css-and-design-systems.md).
- **SMACSS** *(Scalable & Modular Architecture for CSS)* — five categories: Base, Layout, Module, State, Theme. → [CSS & Design Systems](css-and-design-systems.md).
- **Subgrid** — children inherit their parent's grid tracks; enables aligned, nested layouts. → [CSS & Design Systems](css-and-design-systems.md).
- **Tailwind CSS** — utility-first CSS framework; the dominant 2026 choice for new product apps. → [CSS & Design Systems](css-and-design-systems.md).
- **vanilla-extract / Panda CSS / StyleX** — zero-runtime CSS-in-JS alternatives with type-safe tokens. → [CSS & Design Systems](css-and-design-systems.md).

---

## CSS (language)

- **`@scope`** — limit a stylesheet block to a subtree, with optional lower bound. Cleaner than nested compound selectors. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`@starting-style`** — declare entry-animation start values for transitions on inserted/displayed elements. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Anchor Positioning** *(`anchor-name`, `position-anchor`, `inset-area`)* — pin tooltips/menus to anchor elements without JS. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Block Formatting Context (BFC)** — layout sandbox that contains margin collapse and clears floats. Triggered by `display: flow-root`, `overflow: auto/hidden`, etc. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`box-sizing`** — `border-box` makes `width` include padding + border. The right global default. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Cascade** — pipeline that decides the winner: origin → layer → specificity → order. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md), [CSS & Design Systems](css-and-design-systems.md).
- **`clamp(min, preferred, max)`** — fluid value with floor and ceiling. Workhorse for responsive type and spacing. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`color-mix()`** — interpolate between two colors in a chosen color space. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Containing Block** — the rectangle a positioned/percentage value resolves against. Determined by element type and ancestor chain. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Container Query** *(`@container`)* — style a component based on its container's size, not the viewport. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Container Query Units** *(`cqw`, `cqh`, `cqi`, `cqb`)* — units sized to the nearest query container. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`currentColor`** — keyword that resolves to the element's `color` value; lets borders/SVGs follow text color. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Dynamic Viewport Units** *(`dvh`, `svh`, `lvh`)* — viewport units that account for mobile chrome (address bar). Use `dvh` for full-screen layouts. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`field-sizing: content`** — auto-size form controls to their content. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Flex Formatting Context** — children of a `display: flex` element become flex items, laid out along the main axis. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`@font-face` / `font-display`** — declare a custom font and control fallback timing (`swap`, `optional`, ...). → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Grid Formatting Context** — children of a `display: grid` element become grid items, placed in tracks. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`:has()`** — relational pseudo-class; the parent/sibling-aware selector. Use sparingly on huge trees. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Inheritance** — some properties (`color`, `font-*`, `line-height`) inherit by default; layout/visual ones don't. Force with `inherit`. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`isolation: isolate`** — creates a stacking context without other side effects. The clean way to compose overlays. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`:is()` / `:where()`** — selector grouping. `:is()` takes max specificity of args; `:where()` is always 0. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`light-dark()`** — function that returns one of two values based on `color-scheme`. Cleaner than dual media queries. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Logical Properties** — `padding-block`, `inset-inline-start`, `margin-inline`, etc. Translate for RTL and vertical writing modes. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Margin Collapsing** — adjacent vertical margins between block-level siblings collapse to the larger of the two. Avoid by establishing a BFC. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Native CSS Nesting** — `&` nesting without a preprocessor; widely supported as of 2024. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`oklch()` / `oklab()`** — perceptually uniform color spaces; equal numerical changes look like equal perceptual changes. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`overscroll-behavior`** — control scroll chaining. `contain` stops scrolls from reaching ancestors. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`@property`** — typed custom properties — pick a syntax, default, inheritance behavior; enables interpolation. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`prefers-*` media features** — `reduced-motion`, `color-scheme`, `contrast`, `reduced-transparency`. User preferences exposed to CSS. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md), [Web Accessibility](web-accessibility.md).
- **Scroll-Driven Animations** *(`animation-timeline: scroll(...) | view(...)`)* — animate based on scroll position or element-in-view progress, no JS. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Scroll Snap** *(`scroll-snap-type`, `scroll-snap-align`)* — native snap-to-position scrolling. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Specificity** — four-tuple `(inline, id, class/attr/pseudo-class, element/pseudo-element)`. Compare left-to-right. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Stacking Context** — local `z-index` ordering scope. Created by `position` + `z-index`, `opacity < 1`, `transform`, `filter`, `isolation: isolate`, etc. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **Subgrid** — children inherit their parent grid's tracks; enables aligned, nested layouts. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md), [CSS & Design Systems](css-and-design-systems.md).
- **`text-wrap: balance` / `pretty`** — better line breaking. `balance` for headlines, `pretty` for body. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`:user-valid` / `:user-invalid`** — form validation pseudo-classes that only apply *after* user interaction. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **View Transitions** — `@view-transition` + `view-transition-name` for smooth same-document and cross-document transitions. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md), [CSS & Design Systems](css-and-design-systems.md).
- **`will-change`** — performance hint that promotes an element to its own layer. Use just before animating; remove after. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).
- **`:where()`** — see `:is()` / `:where()`. Specificity 0 — perfect for resets and library defaults. → [CSS Mental Models & Cheat Sheet](css-mental-models-and-cheat-sheet.md).

---

## HTML

- **`<article>`** — self-contained content (post, card, comment) that could stand alone. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<aside>`** — tangentially related content (sidebar, callout, pull quote). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`autocomplete` tokens** — standardized hints (`email`, `username`, `current-password`, `one-time-code`, `cc-number`, ...) password managers and browsers key off. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **Constraint Validation API** — built-in form validation via attributes (`required`, `min`, `pattern`, ...) plus the JS API (`setCustomValidity`, `checkValidity`). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<datalist>`** — typeahead suggestions for an `<input list="...">`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<details>` / `<summary>`** — native disclosure widget with keyboard support. No JS needed. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<dialog>`** — native modal element with focus trap, backdrop, ESC-dismiss, top-layer rendering. Use `showModal()`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<dl>` / `<dt>` / `<dd>`** — description list: term/definition pairs. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`enterkeyhint`** — labels the Enter key on virtual keyboards (`go`, `send`, `search`, `next`, `previous`, `done`). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<fieldset>` / `<legend>`** — group related form controls with an accessible group name. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<figure>` / `<figcaption>`** — self-contained content (image, chart, code) with its caption. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<footer>`** — bottom-of-section content: credits, related links. Landmark when at body level. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<header>`** — top-of-section content: masthead, intro, byline. Landmark when at body level. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`hidden`** — boolean attribute that visually hides + removes from AT. Prefer over `display: none` for state. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`inert`** — removes element + descendants from focus order, click, and assistive tech. Use on offscreen modal trees. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`inputmode`** — hint to the virtual keyboard (`numeric`, `decimal`, `email`, `tel`, `url`, `search`). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<label>`** — accessible name for a form control. Associate via `for`/`id` or by wrapping. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **Landmarks** — `<header>`, `<nav>`, `<main>`, `<aside>`, `<footer>`, `<search>`, plus regions with an accessible name. Screen-reader navigation hooks. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md), [Web Accessibility](web-accessibility.md).
- **`<main>`** — unique-to-this-page content. Exactly one per page. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<meter>`** — scalar measurement within a known range (low/high/optimum). Different from `<progress>`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<nav>`** — major navigation block (primary, secondary, breadcrumbs, TOC). Multiple `<nav>`s should each carry an `aria-label`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<output>`** — result of a calculation or user action; live-region by default. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<picture>`** — wrapper for `<source>`s + fallback `<img>`; used for art direction and format negotiation (AVIF/WebP/JPEG). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`popover`** — boolean attribute that gives any element auto-close, light-dismiss, top-layer popover behavior. Trigger with `popovertarget`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<progress>`** — completion indicator (determinate or indeterminate). → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **Quirks Mode** — legacy rendering mode the browser falls into without `<!doctype html>`. Don't ship pages without a doctype. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **Resource Hints** — `preload`, `preconnect`, `dns-prefetch`, `modulepreload`, `prefetch` — `<link>` directives that nudge browser fetch order. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **`<search>`** *(2024+)* — landmark element for site-search regions. Replaces `role="search"`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<section>`** — thematic grouping, with a heading. Not a substitute for `<div>`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`srcset` / `sizes`** — give the browser candidate image sources at known widths so it can pick the best one for the viewport and DPR. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md), [Core Web Vitals](core-web-vitals.md).
- **`tabindex`** — `0` adds to natural focus order; `-1` is programmatic-only; positive values are almost always a bug. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<time datetime="...">`** — machine-readable date/time for content, calendars, and crawlers. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).
- **`<track>`** — captions, subtitles, descriptions, chapters for `<video>`/`<audio>`. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md), [Web Accessibility](web-accessibility.md).
- **Viewport Meta** — `<meta name="viewport" content="width=device-width, initial-scale=1">`. Required on mobile to disable the default desktop-emulation viewport. → [HTML Essentials & Cheat Sheet](html-essentials-and-cheat-sheet.md).

---

## JavaScript

- **AbortController / AbortSignal** — cancellation primitive accepted by `fetch`, event listeners, observers, and most modern async APIs. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Arrow Function** — `() => ...` shorthand that inherits `this` lexically and has no `arguments`. Not a syntactic shortcut — semantically different. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`async` / `await`** — sugar over Promises; each `await` yields to the event loop and resumes when the promise settles. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Closure** — function plus the lexical environment in which it was created. Backbone of every callback, module, and private state. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **CommonJS (CJS)** — Node's legacy module system: `require` / `module.exports`. Synchronous, dynamic, not tree-shakable. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Destructuring** — `const { a, b: x = 1, ...rest } = obj` — pattern-match into bindings with renames and defaults. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **ESM** *(ECMAScript Modules)* — standard module system: `import` / `export`, static, async-capable, tree-shakable, live bindings. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Event Delegation** — single listener on a parent element that handles events for many children via `e.target.closest(...)`. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Event Loop** — runtime loop that picks one macrotask, drains all microtasks, then maybe renders. Single-threaded. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md), [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Execution Context** — environment a function runs in: lexical environment, `this`, scope chain. Created on each call. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Falsy** — `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`for...of` vs `for...in`** — `for...of` iterates values of iterables; `for...in` iterates enumerable string keys (don't use on arrays). → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Generator** — `function*` that yields lazy values; backbone of iterators and async iteration. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Hoisting** — engine moves `var` and `function` declarations to the top of their scope before running code. `let`/`const` are hoisted but uninitialized (TDZ). → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **IntersectionObserver** — observes element visibility relative to the viewport. The right tool for lazy-loading and infinite scroll. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md), [Web Optimization Techniques](web-optimization-techniques.md).
- **Iterable / Iterator** — anything with `[Symbol.iterator]`; consumed by `for...of`, spread, destructuring. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Layout Thrashing** — interleaving DOM reads (`offsetWidth`, `getBoundingClientRect`) and writes, forcing repeated style/layout recalculation. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md), [Browser Rendering Pipeline](browser-rendering-pipeline.md).
- **Macrotask** — one unit of work pulled from a task queue (timers, I/O, UI events). One macrotask runs per loop tick. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`Map` / `Set` / `WeakMap` / `WeakSet`** — keyed and unique collections; weak variants don't prevent GC of their keys. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Microtask** — promise callback (or `queueMicrotask`); drained completely after every macrotask, before render. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **MutationObserver** — observes DOM tree changes (children, attributes, text). Use over polling. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Nullish Coalescing (`??`)** — returns the right operand only when the left is `null` or `undefined` (unlike `||`, which fires for any falsy). → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Optional Chaining (`?.`)** — short-circuits property access, calls, and indexing on `null`/`undefined`. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Promise** — placeholder for an async result; states are `pending`, `fulfilled`, `rejected`. Settled promises are immutable. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`Promise.all` / `allSettled` / `race` / `any`** — combinators for running promises concurrently with different settlement semantics. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`Promise.withResolvers()`** — returns `{ promise, resolve, reject }` so you can settle a promise from outside its executor. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Prototype Chain** — every object has a `[[Prototype]]`; lookups walk the chain until found or `null`. `class` is sugar over this. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`queueMicrotask`** — schedule a callback to run at the next microtask checkpoint; deterministic, no minimum delay. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`requestAnimationFrame`** — schedule a callback before the next paint; the only correct place to do animation work. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`requestIdleCallback`** — schedule low-priority work for browser-idle time. Chromium-mostly. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **ResizeObserver** — observes element size changes; component-level alternative to `window.resize`. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Scope Chain** — chain of lexical environments the engine walks for identifier lookup. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Spread / Rest** — `...` spreads iterables into args/elements/keys, or collects them in destructuring/parameters. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`structuredClone`** — built-in deep clone supporting `Map`, `Set`, `Date`, typed arrays. The right modern replacement for `JSON.parse(JSON.stringify(...))`. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Symbol** — unique primitive; used for private-ish keys and well-known protocol hooks (`Symbol.iterator`, `Symbol.asyncIterator`). → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Tagged Template** — function call invoked via template-literal syntax: `tag\`...${x}...\``. Used for HTML escaping, SQL builders, etc. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **TDZ** *(Temporal Dead Zone)* — region from the start of a block until a `let`/`const` declaration; reading the binding throws `ReferenceError`. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **`this` Binding** — determined at call time by call shape: method-call, plain-call, `new`, `bind`/`call`/`apply`, or arrow (lexical). → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Top-Level `await`** — `await` allowed at the top of an ESM module; the module becomes async. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).
- **Type Coercion** — implicit conversion done by `==`, `+`, template literals, `Number()`/`String()`. Prefer `===` and explicit conversion. → [JavaScript: Mental Models & Cheat Sheet](javascript-mental-models-and-cheat-sheet.md).

---

## TypeScript

- **`any`** — escape-valve type; disables all checking. Treat each occurrence as a code smell. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`as const`** — assertion that freezes inference to literal types and marks properties `readonly`. The canonical way to derive a union from a JS object. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Assertion Function (`asserts`)** — function that throws if a predicate fails and otherwise narrows the argument's type for the rest of the scope. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Branded / Nominal Type** — phantom-property pattern that simulates nominal typing on top of TS's structural system; used for `UserId` vs `OrderId`, validated values, currency units. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Conditional Type** — type-level if/else (`T extends U ? X : Y`). Distributes over union types unless wrapped in a tuple. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Declaration Merging** — multiple `interface` declarations with the same name combine into one. Foundation for module augmentation. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Discriminated Union** — union where each member carries a literal-typed "tag" field; the compiler narrows on the tag in `switch`/`if` statements. The antidote to boolean-state explosion. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Distributive Conditional Type** — conditional type whose `T` is naked, causing it to apply once per union member. Often subtle. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Exhaustive Check** — pattern of assigning the narrowed value of a discriminated union to `never` in the default branch; build fails if a new member is added without handling. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Generic** — type parameter on a function, type, or class (`function first<T>(arr: T[]): T | undefined`). Often paired with `extends` constraints. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`infer`** — keyword that introduces a type variable inside a conditional, capturing part of the matched type (`T extends Array<infer U> ? U : never`). → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Mapped Type** — type produced by iterating over the keys of another (`{ [K in keyof T]: ... }`). Modifiers: `readonly`, `?`, and the inverses `-readonly`, `-?`. Key remapping with `as` since TS 4.1. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Module Augmentation** — `declare module 'x' { ... }` block that extends third-party types (e.g., adding `req.user` to Express). → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`never`** — bottom type representing values that can't exist. Assignable to every type; nothing is assignable to it. Used for exhaustive checks and unreachable code. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`noUncheckedIndexedAccess`** — strict-mode-adjacent flag that types `arr[i]` as `T \| undefined`. Catches off-by-one and missing-key bugs that `strict` alone misses. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`satisfies`** *(TS 4.9+)* — operator that validates a value matches a type without widening it. The right tool for typed config objects. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Strict Mode** — `tsconfig` umbrella enabling `strictNullChecks`, `noImplicitAny`, `strictFunctionTypes`, and others. The non-negotiable starting point. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Structural Typing** — TypeScript's matching rule: two types are compatible if they have the same shape, regardless of name. The opposite of nominal typing. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Template Literal Type** — string types you can pattern-match and concatenate (`` `on${Capitalize<E>}` ``). Powers typed routing, event names, and CSS-in-TS. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Type Guard / Type Predicate** — function with return type `arg is T`; tells the compiler to narrow on a true result. The compiler trusts the body — false guards introduce silent bugs. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Type Narrowing** — control-flow analysis that reduces a union to one or more specific members based on `typeof`, `instanceof`, `in`, equality, predicates, or assertions. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`unknown`** — type-safe alternative to `any`: holds any value but forbids operations until you narrow. The right type for `JSON.parse`, untrusted input, and `catch (e)` (with `useUnknownInCatchVariables`). → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Utility Types** — built-in generics: `Partial<T>`, `Required<T>`, `Readonly<T>`, `Pick<T,K>`, `Omit<T,K>`, `Record<K,V>`, `ReturnType<F>`, `Parameters<F>`, `Awaited<T>`, `NonNullable<T>`, `Exclude<T,U>`, `Extract<T,U>`. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **`verbatimModuleSyntax`** — `tsconfig` flag that requires explicit `import type` for type-only imports. Cleanest interop with bundlers and Node's ESM. → [TypeScript Deep Dive](typescript-deep-dive.md).
- **Zod / Valibot / ArkType** — runtime schema-validation libraries that double as TS-type sources via `z.infer<typeof schema>`. The right tool at every untrusted boundary. → [TypeScript Deep Dive](typescript-deep-dive.md), [Web Application Security](web-application-security.md).

---

## React

- **Action** *(React 19)* — async function passed to a transition, `<form action>`, or `formAction`; React tracks pending/error/optimistic state around it. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Automatic Batching** — React 18+ batches all state updates in the same tick — including those from timers, promises, and native events. Opt out with `flushSync`. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Commit Phase** — synchronous, uninterruptible phase where React applies DOM mutations, runs `useLayoutEffect`s, then schedules `useEffect`s. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Concurrent Rendering** — umbrella for time-slicing, interruption, transitions, and Suspense-driven boundaries. Opt-in per update. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Double Buffering** — React keeps two fiber trees (`current` and `workInProgress`) and atomically swaps them on commit. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Fiber** — plain JS object representing one unit of work in the tree, linked to parent/child/sibling. The reconciler's data structure. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`forwardRef`** — pre-19 wrapper that let a component receive a `ref` prop. Retired in 19 — `ref` is just a regular prop. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Hooks Linked List** — each fiber stores hook records as a linked list in call order; why hooks must run in the same order every render. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Hydration** — process of attaching React to server-rendered HTML. Selective hydration in 18+ lets the framework prioritize visible/interactive parts. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md), [Web Rendering Strategies](web-rendering-strategies.md).
- **Key** — stable, unique identifier React uses to match list items across renders. Index-as-key is fine for static lists, broken for reorders. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Lanes** — 31-bit bitmask priority model that replaced expiration-time scheduling. Each update is tagged with a lane (Sync, Transition, Idle, etc.). → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`memo`** — HOC that skips re-render when props are referentially equal. Mostly redundant with the React Compiler. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **React Compiler** — Babel/SWC plugin that auto-memoizes by analyzing component dataflow. Ships fine-grained caching the developer would have had to write by hand. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Reconciliation** — React's diffing algorithm: same type → reuse fiber; different type → unmount subtree; lists → match by `key`. O(n) by design. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Render Phase** — pure, interruptible phase where React calls components and builds the work-in-progress tree. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **RSC** *(React Server Components)* — components that render on the server (or at build), serialize over the wire, and ship zero JS. Marked with `"use client"` directive at the boundary. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md), [Web Rendering Strategies](web-rendering-strategies.md).
- **Scheduler** — package (`scheduler`) that time-slices React work into ~5ms chunks and yields to the browser via `MessageChannel`. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Server Action** — async function marked with `"use server"` that can be called directly from forms or client components. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Strict Mode** — React component that double-invokes effects/render in dev to surface impurities and missing cleanup. Not a runtime check. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Suspense** — rendering primitive that pauses subtree until a thrown promise resolves. Backbone of code-splitting, RSC streaming, and `use()`. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **Transition** — non-urgent state update, marked via `useTransition` or `startTransition`; interruptible by higher-priority work. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`use`** *(React 19)* — hook that unwraps a Promise (suspending) or reads a Context. The only hook callable conditionally. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useActionState`** *(React 19)* — hook returning `[state, formAction, isPending]` for forms backed by an Action. Replaces `useFormState`. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useDeferredValue`** — hook that lags a value behind during heavy work, so urgent updates (typing) stay snappy. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useFormStatus`** *(React 19)* — hook that reads the parent `<form>`'s pending state from a nested submit button. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useOptimistic`** *(React 19)* — hook for predicting a state value during a transition, reverting if the action fails. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useSyncExternalStore`** — official hook for binding external stores (Zustand, Redux, etc.) to React's concurrent rendering. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).
- **`useTransition`** — hook returning `[isPending, startTransition]` for marking updates as non-urgent. → [React Internals & Cheat Sheet](react-internals-and-cheat-sheet.md).

---

## AI Integration

- **Agent** — an LLM in a loop with tools, memory, and the ability to plan multi-step tasks. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Chain-of-Thought (CoT)** — prompting technique that asks the model to "think step by step" before answering. Improves reasoning at a token cost. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Chunking** — splitting source documents into pieces for embedding and retrieval. The highest-leverage tuning knob in RAG. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **ColBERT** — late-interaction retrieval method that embeds each token independently. Higher retrieval quality, more compute. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Embedding** — high-dimensional vector representation of text, used for semantic similarity search. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Eval** — automated evaluation of LLM outputs against expected behavior. The test suite of AI engineering. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Few-Shot Prompting** — including 2–5 examples in the prompt for tasks where pattern matching beats instruction. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Function Calling / Tool Use** — provider-supported pattern where the model returns structured tool-call payloads instead of free text. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Generative UI** — streaming UI components rendered by the model (Vercel AI SDK). The UI itself is part of the response. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Hallucination** — model generating plausible-sounding but false content. The default failure mode of LLMs. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **HyDE** *(Hypothetical Document Embeddings)* — embed a hypothetical answer to the query for better retrieval against the actual corpus. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **LLM** *(Large Language Model)* — neural network trained on broad text to generate language. The substrate of modern AI features. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **LLM-as-Judge** — use a model to grade another model's outputs against a rubric. Standard for open-ended eval. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **LLMOps / GenOps** — operational disciplines for LLM-powered systems: prompt versioning, eval, observability, drift detection. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Multimodal** — model that handles multiple input types (text, image, audio, video). → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **On-Device AI** — running models locally in the browser via WebGPU (Transformers.js, WebLLM) or on the device itself. Privacy and latency wins. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Prompt Caching** — provider feature (Anthropic, OpenAI, Gemini) that bills cached prompt prefixes at a fraction of full input cost. 5–10× savings for RAG-heavy apps. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Prompt Injection** — untrusted input that contains adversarial instructions the model follows. The most important AI security topic. → [AI Integration & SDLC](ai-integration-and-sdlc.md), [Web Application Security](web-application-security.md).
- **RAG** *(Retrieval-Augmented Generation)* — retrieve relevant passages, augment the prompt with them, generate a grounded answer. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Re-ranker** — second-stage cross-encoder that re-orders top-k retrieved chunks for higher precision (Cohere Rerank, BGE Rerank). → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Speculative Decoding** — small model drafts; large model verifies. Faster output than always running the large model. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Structured Outputs** — provider-enforced JSON-schema-conformant model outputs. The reliable way to get parseable structure. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **Vector Database** — storage indexed for fast nearest-neighbor search over embeddings (pgvector, Pinecone, Qdrant, Weaviate). → [AI Integration & SDLC](ai-integration-and-sdlc.md).
- **WebGPU** — browser API for general-purpose GPU compute; powers in-browser model inference. → [AI Integration & SDLC](ai-integration-and-sdlc.md).
