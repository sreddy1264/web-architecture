# Web Rendering Strategies & Application Architectures

> A practical guide to choosing how (and where) your web pages get rendered.

---

## Table of Contents

1. [Why This Matters](#why-this-matters)
2. [The Two Orthogonal Axes](#the-two-orthogonal-axes)
3. [Application Shapes: MPA vs. SPA](#application-shapes-mpa-vs-spa)
4. [Rendering Strategies](#rendering-strategies)
   - [CSR — Client-Side Rendering](#1-csr--client-side-rendering)
   - [SSR — Server-Side Rendering](#2-ssr--server-side-rendering)
   - [SSG — Static Site Generation](#3-ssg--static-site-generation)
   - [ISR — Incremental Static Regeneration](#4-isr--incremental-static-regeneration)
   - [Streaming SSR](#5-streaming-ssr)
   - [Edge Rendering](#6-edge-rendering)
   - [PPR — Partial Prerendering](#7-ppr--partial-prerendering)
   - [Islands Architecture](#8-islands-architecture)
   - [Resumability](#9-resumability)
5. [Side-by-Side Comparison](#side-by-side-comparison)
6. [How to Decide: A Framework](#how-to-decide-a-framework)
7. [Hybrid & Mixed Architectures](#hybrid--mixed-architectures)
8. [Framework Landscape (2026)](#framework-landscape-2026)
9. [Migration Paths](#migration-paths)
10. [Precautions and Common Pitfalls](#precautions-and-common-pitfalls)
11. [Decision Cheat Sheet](#decision-cheat-sheet)
12. [Further Reading](#further-reading)

---

## Why This Matters

The "render strategy" decision is the single most consequential architectural choice in a modern web app. It determines:

- **Performance ceiling** — what LCP, INP, and TTFB you can realistically hit
- **SEO outcomes** — whether crawlers and AI engines see your content
- **Hosting & infrastructure cost** — static CDN vs. always-on servers
- **Developer experience** — local-only vs. server runtime complexity
- **Operational complexity** — caching layers, revalidation, edge runtimes
- **Time-to-interactive on slow devices** — hydration is expensive
- **Personalization capabilities** — auth-aware vs. anonymous rendering

> **Worth knowing:** there is no "best" strategy. There is only the right strategy for *this page, this audience, this content cadence, this team's operational maturity*. The most sophisticated apps use **multiple strategies in the same codebase** — marketing pages static, dashboard SSR, search results streamed, modals client-only.

---

## The Two Orthogonal Axes

People conflate "SPA vs MPA" with "CSR vs SSR." They're independent decisions:

| Axis | Question | Options |
|---|---|---|
| **Application shape** | After first load, do route changes reload the document? | **MPA** (yes) / **SPA** (no) |
| **Render location & timing** | Where and when is HTML produced? | **Build time** (SSG) / **Request time on server** (SSR) / **Request time at edge** (Edge) / **Client at runtime** (CSR) |

A Next.js App Router app is technically a *hybrid SPA with server-rendered initial HTML* — server-rendered first response, then client-side navigation. Astro is the opposite — *MPA with optional client islands*. The dichotomies blur, which is why precise vocabulary matters.

---

## Application Shapes: MPA vs. SPA

### MPA — Multi-Page Application
Each navigation triggers a full document request. The server returns HTML; the browser parses and renders it from scratch.

**Examples:** classic Rails/Django/Laravel apps, WordPress, Astro, MPAs built with HTMX, marketing sites.

**Benefits:**
- Simple mental model — every URL is a fresh request
- Excellent first-load performance per page (no JS framework needed)
- Ideal for content-heavy, SEO-critical sites
- Browser handles back/forward, scroll restoration, focus naturally
- Smaller JS payloads — only what each page needs
- Easier accessibility (browser does most of the work)

**Trade-offs:**
- Full reload on each navigation feels slower (mitigated by **View Transitions API** + browser bfcache)
- Can't preserve state across navigation (audio players, sidebars) without effort
- Harder to build app-like, highly interactive flows
- Repeated parse of shared header/footer/CSS on each page (cacheable, but still)

### SPA — Single-Page Application
The browser loads one HTML document and one JS bundle; subsequent "navigations" are client-side route changes that swap UI without reloading.

**Examples:** Gmail, Linear, Figma, most React/Vue/Angular apps without SSR, internal dashboards.

**Benefits:**
- App-like feel — instant transitions, preserved state
- Rich interactivity (drag-drop, real-time updates, optimistic UI)
- Decoupled frontend from backend (separate deploys, BFF pattern)
- Once loaded, lower server load — most ops are API calls
- Best fit for authenticated, behind-login experiences

**Trade-offs:**
- Big initial JS bundle = slow first paint, especially on mobile
- SEO requires extra effort (JS rendering by crawlers is unreliable for non-Google engines)
- Routing, focus management, accessibility, and analytics are all *your problem*
- Memory leaks accumulate over long sessions
- Bundling complexity — code-splitting becomes critical
- "White screen until JS loads" experience without SSR fallback

> **Lesson learned:** the "SPA tax" is real. Choosing SPA means committing to ongoing investment in route-aware focus management, error boundaries, scroll restoration, head metadata per route, and analytics page tracking. None of these are free, and all are easy to ship broken — every project I've seen underestimate this paid for it later.

### MPA + View Transitions = the Best of Both
The View Transitions API (Chrome 111+, Safari 18+) lets MPAs animate between pages smoothly. Combined with **Speculation Rules API** for prefetching/prerendering, modern MPAs can feel as fast as SPAs without the JS tax.

```html
<!-- Same-document transition (SPA) -->
<script>
  document.startViewTransition(() => updateDOM());
</script>

<!-- Cross-document transition (MPA) — Chrome 126+ -->
<meta name="view-transition" content="same-origin" />
```

---

## Rendering Strategies

### 1. CSR — Client-Side Rendering

The server returns a near-empty HTML shell (`<div id="root"></div>`); JavaScript downloaded by the browser fetches data and renders the UI.

**Example pipeline:**
```
GET /  →  <html><body><div id="root"></div><script src="bundle.js"/></body></html>
        →  browser downloads JS  →  hydrates  →  fetches API  →  renders UI
```

**Use when:**
- Authenticated dashboard / internal tool with no SEO requirement
- Highly interactive, app-like experiences (Figma-style canvases, IDEs, design tools)
- The first screen is gated behind auth anyway

**Benefits:**
- Cheapest hosting (static files on a CDN; no server runtime)
- Simple deployment pipeline
- Snappy after initial load — all subsequent interactions are client-side
- Backend is just an API; clean separation of concerns

**Trade-offs:**
- **Worst initial LCP/FCP** of any strategy
- **SEO challenges** — Googlebot will render but with delay; many other crawlers won't render at all
- **Open Graph / Twitter / link previews** don't render JS; you'll get blank previews unless you SSR or pre-render those routes
- Slow on low-end devices (CPU-bound hydration & framework overhead)
- "White screen" until JS loads — bad on flaky networks

**Verdict:** acceptable for authenticated apps; **do not use for public, indexable pages**.

---

### 2. SSR — Server-Side Rendering

The server renders the full HTML on every request and returns it. The browser shows content immediately; JS hydrates the page to make it interactive.

**Example pipeline:**
```
GET /product/123  →  server fetches data  →  renders React to HTML  →  ships HTML + JS
                  →  browser shows content  →  hydration begins  →  interactive
```

**Use when:**
- Content varies per user, per request (auth-aware, personalized, A/B tested)
- Real-time / highly dynamic data (price, stock, scores)
- SEO matters and content can't be pre-built
- E-commerce product pages, search results, news, social feeds

**Benefits:**
- Fast LCP — content arrives in the first byte stream
- Full SEO support — crawlers see real HTML
- Per-request personalization
- Single source of truth for routing/data (no client-only logic gaps)

**Trade-offs:**
- **Server cost** — every request burns CPU on render
- **TTFB depends on render speed** — slow renders = slow LCP
- **Hydration cost** — JS still ships and runs to attach event listeners; mobile devices feel it
- More operational complexity (runtime, scaling, caching layers)
- Cache invalidation is genuinely hard

**Verdict:** the default for personalized, dynamic, SEO-critical pages. Pair with caching aggressively (CDN + per-user fragments).

---

### 3. SSG — Static Site Generation

HTML is generated at **build time** for every route and served as static files from a CDN.

**Example pipeline:**
```
[build]  →  framework iterates all routes  →  pre-renders HTML  →  uploads to CDN
[user]   →  GET /  →  CDN returns static HTML in <50ms globally
```

**Use when:**
- Content changes infrequently (marketing sites, docs, blogs, landing pages)
- The same HTML is fine for every visitor
- You want the lowest cost and highest reliability

**Benefits:**
- **Fastest possible TTFB** — served from the CDN edge, no compute
- Cheapest hosting (S3 + CloudFront, Vercel, Netlify, Cloudflare Pages)
- Resilient — no server to crash; CDN is your runtime
- Excellent SEO and Core Web Vitals
- Trivial to scale (CDN is built for it)
- Atomic, immutable deploys — easy rollbacks

**Trade-offs:**
- **Stale until rebuild** — content updates require redeploy
- **Build time scales with route count** — 100K+ routes can mean 30+ min builds
- No per-request personalization (need client-side fetches or edge middleware for that)
- Auth-gated content needs a different strategy

**Verdict:** the best choice for content that doesn't change per user or per minute. Combine with ISR or client-side hydration for dynamic islands.

---

### 4. ISR — Incremental Static Regeneration

A hybrid: HTML is statically generated, but **revalidated** on a TTL or on-demand. Stale pages are served instantly while a fresh version is rebuilt in the background (stale-while-revalidate semantics).

**Example pipeline:**
```
[build]   →  pre-render top N routes
[user 1]  →  GET /post/abc  →  stale HTML returned instantly
                              →  background regeneration triggered
[user 2]  →  GET /post/abc  →  fresh HTML (or near-fresh)
```

**Use when:**
- Large catalogs (e-commerce, news, blogs) with tens of thousands to millions of pages
- Content updates periodically but doesn't need to be real-time
- You want SSG performance with SSR-like freshness

**Benefits:**
- CDN-fast responses with eventual consistency
- Builds stay short (only top routes pre-rendered; rest generated on first visit)
- Reduced server cost vs full SSR
- Per-page revalidation control

**Trade-offs:**
- The first user after invalidation may see a stale page (or a cold-build delay, depending on framework)
- Requires platform support (Next.js, Gatsby, Nuxt, Astro with adapters)
- Cache invalidation logic is non-trivial — debugging stale content is painful
- Not for per-user content (use SSR for that)

**Verdict:** the unsung hero of large content sites. If you have >5K pages and content updates periodically, ISR is usually the right answer.

---

### 5. Streaming SSR

A variant of SSR where the server **streams** the HTML response as it's being rendered, sending chunks the browser can paint progressively. Slow data dependencies (e.g., a recommendation block) are wrapped in `<Suspense>` boundaries that resolve later in the stream.

**Example pipeline:**
```
GET /  →  server flushes shell + nav immediately
       →  streams hero section as soon as data resolves
       →  streams below-the-fold content last (or with placeholders)
```

**Use when:**
- Dynamic pages have a fast core (header, hero) and slower secondary data (recommendations, comments)
- You want SSR's SEO + personalization without TTFB penalty for the slowest data
- Modern React (18+), Next.js App Router, Remix, Solid Start

**Benefits:**
- **Excellent TTFB and LCP** — first byte arrives before all data is fetched
- Per-section loading states (Suspense) without client roundtrips
- Backpressure and progressive rendering for large pages

**Trade-offs:**
- Requires framework + infrastructure that supports streaming responses (some CDNs/proxies buffer)
- Harder to reason about (concurrent rendering, async boundaries)
- Error handling is more nuanced (errors in late chunks)

**Verdict:** the modern default for dynamic React apps. If you're on Next.js App Router, you're already getting it.

---

### 6. Edge Rendering

SSR (or middleware) executed on **CDN-edge runtimes** physically close to the user (Cloudflare Workers, Vercel Edge, Deno Deploy, AWS Lambda@Edge).

**Use when:**
- Latency-sensitive personalization (geo-routing, A/B tests, auth checks, redirects)
- Global audiences where origin server latency dominates
- Lightweight rendering — you don't need a full Node.js runtime

**Benefits:**
- Sub-50ms TTFB worldwide
- Cheap per-request (V8 isolates, ms billing)
- Often paired with CDN cache for hybrid speed

**Trade-offs:**
- Limited runtime — no full Node API, restricted package compatibility, often no native modules
- Cold-start behavior varies by platform
- Database access from edge requires edge-friendly drivers (Hyperdrive, PlanetScale, Neon serverless, Turso) or you lose the latency benefit
- Harder to debug (distributed logs across regions)
- Per-region data residency / compliance can complicate routing

**Verdict:** powerful for middleware (auth, redirects, rewrites, geo) and lightweight pages. Less ideal for heavy DB-bound rendering — origin SSR + CDN cache often wins there.

---

### 7. PPR — Partial Prerendering

A newer hybrid (Next.js, Astro, others exploring): the **static shell** is prerendered at build time, while **dynamic holes** stream in at request time. Combines SSG's TTFB with SSR's freshness within a single page.

**Example pipeline:**
```
GET /shop  →  static shell (header, nav, hero) served instantly from CDN
            →  user-specific cart count + recommendations stream in
```

**Use when:**
- Pages have a mostly-static shell with a few dynamic islands (cart count, personalized hero)
- You're on a framework that supports it (Next.js 14+ with App Router, opt-in)

**Benefits:**
- Best-of-both performance: instant shell + fresh dynamic data
- Single mental model per page (no CSR fallback for one widget)

**Trade-offs:**
- Framework-coupled, still maturing as of 2026
- Debugging hydration mismatches across static/dynamic boundaries
- Requires Suspense-friendly data fetching

**Verdict:** the direction modern React frameworks are heading. Adopt cautiously while it stabilizes; it will likely be the default within a few years.

---

### 8. Islands Architecture

The page is **mostly static HTML**, with small **interactive "islands"** (widgets) that hydrate independently. Most of the page ships zero JS; only the interactive parts get a runtime.

**Pioneered by:** Astro, Marko, Fresh (Deno), Eleventy + Is-land.

**Use when:**
- Content-heavy sites with isolated interactivity (blog with a search widget, docs with a like button, marketing with a pricing calculator)
- You want the simplicity of MPA + the interactivity of SPA

**Benefits:**
- **Tiny JS payloads** — orders of magnitude less than typical SPA
- Excellent CWV out of the box
- Per-island lazy hydration / load priority
- Mix frameworks (React island here, Vue island there) — though you usually shouldn't

**Trade-offs:**
- Not great for highly interactive *application* shells (you'll be reaching for a SPA)
- Cross-island state sharing is awkward (often needs a small store like nanostores)
- Routing is MPA-style (full reloads) unless layered with view transitions

**Verdict:** the sweet spot for content sites with interactive flourishes. If you're shipping a full SPA for a blog, you've probably over-engineered.

---

### 9. Resumability

A more radical approach: instead of *hydrating* (re-running framework code on the client to attach handlers), the server serializes execution state into the HTML so the client can simply **resume** where the server left off. **No hydration cost**, regardless of page complexity.

**Pioneered by:** Qwik (and Qwik City), exploring in some experimental React variants.

**Use when:**
- You need the absolute lowest INP / TTI on first interaction
- Page is large/complex enough that hydration cost is meaningful
- You're willing to bet on a smaller ecosystem

**Benefits:**
- O(1) interactivity — same time-to-interactive regardless of app size
- Tiny initial JS — handlers fetched on-demand per interaction
- Excellent CWV at scale

**Trade-offs:**
- Smaller ecosystem (fewer libraries, smaller hiring pool)
- Different mental model for state and component lifecycle
- Tooling/debugging less mature than React/Vue

**Verdict:** technically impressive; viable for greenfield content/commerce. For most teams, RSC + streaming SSR + PPR is "good enough" without leaving the React ecosystem.

---

## Side-by-Side Comparison

| Strategy | TTFB | LCP | JS Sent | SEO | Cost | Personalization | Best for |
|---|---|---|---|---|---|---|---|
| **CSR** | Fastest (static shell) | **Worst** (waits for JS) | Heavy | Weak | $ | Easy (post-load) | Internal dashboards |
| **SSR** | Slowest (renders per req) | Good | Heavy (hydration) | **Strong** | $$$ | Native | Personalized dynamic pages |
| **SSG** | **Fastest** (CDN) | **Best** | Light | **Strong** | $ | None (without extras) | Marketing, docs, blogs |
| **ISR** | Fastest (CDN) | Best | Light | Strong | $$ | Cohort, not per-user | Large content catalogs |
| **Streaming SSR** | Fast (first chunk) | **Best** (progressive) | Heavy | Strong | $$$ | Native | Modern dynamic apps |
| **Edge SSR** | **Fastest** globally | Excellent | Heavy | Strong | $$ | Geo + auth | Global, edge-friendly apps |
| **PPR** | Fastest (shell) | **Best** | Medium | Strong | $$ | Per-island | Mixed-static/dynamic pages |
| **Islands** | Fast (CDN) | **Best** | Tiny | Strong | $ | Per-island | Content + interactive flair |
| **Resumability** | Fast | **Best** | **Tiniest** | Strong | $ | Native | Performance-critical greenfield |

> **Cost** column reflects typical infrastructure spend, not engineering cost. Engineering cost is roughly inverse: simple strategies (CSR, SSG) are cheaper to build; sophisticated ones (PPR, Streaming SSR) are cheaper to operate at scale.

---

## How to Decide: A Framework

Walk through these questions in order. The first decisive answer wins.

### Q1. Does the page need to be discoverable by search engines or AI engines?
- **Yes** → exclude pure CSR. You need rendered HTML on the first response (SSG / SSR / ISR / Streaming / Islands / Edge / PPR).
- **No** (auth-gated, internal) → CSR is on the table.

### Q2. Does the content vary per user / per request?
- **Yes, per user** → SSR, Streaming SSR, Edge SSR, or PPR with dynamic holes.
- **Yes, but cohort-level (geo, A/B, role)** → Edge middleware + cached variants, or PPR.
- **No** → SSG or ISR.

### Q3. How fresh does the content need to be?
| Freshness need | Strategy |
|---|---|
| Real-time (seconds) | SSR / Streaming SSR (no cache, or short cache) |
| Minutes | ISR with short revalidate, or SSR + short CDN cache |
| Hours / on-publish | ISR with on-demand revalidation, or SSG + scheduled rebuild |
| Build cycle (days) | SSG |

### Q4. How many unique URLs are there?
- **< 10,000** → SSG works fine.
- **10,000 – 1M** → ISR (pre-render top routes, generate the rest on demand).
- **> 1M, dynamic** → SSR + aggressive caching, or ISR with on-demand-only.

### Q5. What's the interactivity profile?
- **Mostly static + a few widgets** → Islands (Astro) or PPR.
- **App shell with rich interactions** → SPA over SSR (Next.js, Remix).
- **Highly interactive single experience** (canvas editor, IDE) → CSR SPA, possibly with a server-rendered shell.

### Q6. What's your team's operational maturity?
- **Small team, infra-light** → SSG or CSR SPA.
- **Comfortable with serverless / edge runtimes** → SSR / Streaming / PPR / Edge.
- **Existing backend monolith** → consider keeping it MPA (Rails/Django/Laravel + Hotwire/HTMX/Turbo); new SPA infra is a big commitment.

### Q7. What's the budget profile?
- **Cost-sensitive, low/spiky traffic** → SSG / ISR / Islands (CDN economics).
- **Always-on, predictable load** → SSR is fine; the cost is amortized.
- **Global audience** → Edge to reduce origin egress and per-region latency.

### Q8. What does the rest of the stack already look like?
- React shop with Next.js → App Router with Streaming + PPR (default), opt into SSG/ISR per route.
- Content + marketing focus → Astro with islands.
- Backend-heavy team, Rails/Django → MPA + Hotwire/HTMX/Turbo.
- Heavy real-time collaboration → CSR SPA with WebSocket/CRDT backbone.

---

## Hybrid & Mixed Architectures

Real applications use **multiple strategies in the same codebase**, picked per route:

```
Marketing pages       →  SSG       (rebuild on content change)
Blog                  →  SSG / ISR (revalidate every hour)
Product detail pages  →  ISR       (revalidate on inventory change)
Search & category     →  Streaming SSR
Cart / checkout       →  SSR       (uncached, per-user)
Account dashboard     →  CSR (post-login SPA) or SSR with auth
Admin / internal      →  CSR
```

This is the dominant pattern in modern frameworks. Next.js, Nuxt, SvelteKit, Remix, SolidStart, and Astro all let you choose per route.

### The "BFF" pattern
**Backend-For-Frontend** — a thin server (often the same Next.js / Remix process) sits between the browser and the underlying microservices, aggregating, transforming, and authenticating on behalf of the UI. It's not a render strategy itself but is often what makes SSR/Edge possible.

### Micro-frontends
Splitting a large frontend into independently-deployed apps composed at runtime (Module Federation) or build time (single-spa, web components). Powerful for very large orgs; **massive complexity** for everyone else. The performance, consistency, and ops cost is rarely worth it for teams under ~100 frontend engineers.

### Headless + JAMstack
Content lives in a headless CMS (Contentful, Sanity, Strapi, Storyblok); the frontend (often SSG/ISR) consumes it via API. Excellent separation of concerns; works beautifully with islands or RSC.

---

## Framework Landscape (2026)

| Framework | Default Strategy | Strengths | When to pick |
|---|---|---|---|
| **Next.js** (App Router) | Streaming SSR + RSC, opt-in SSG/ISR/PPR | Most flexible, biggest ecosystem | Default React choice for production apps |
| **Remix / React Router 7** | SSR + nested routing | Web-platform mindset, simpler mental model | Server-rendered apps that lean on web fundamentals |
| **Astro** | SSG + Islands, opt-in SSR | Best-in-class for content sites | Marketing, docs, blogs, content-heavy sites |
| **SvelteKit** | SSR / SSG / CSR per route | Smaller bundles, ergonomic DX | Teams that prefer Svelte; full-stack apps |
| **Nuxt** | SSR / SSG / Hybrid | Vue ecosystem | Vue shops needing flexibility |
| **SolidStart** | Streaming SSR / Islands | Fine-grained reactivity, fast | Performance-obsessed greenfield |
| **Qwik / Qwik City** | Resumability | Tiny JS, instant interactivity | Greenfield content/commerce, perf-critical |
| **Eleventy (11ty)** | SSG | Simple, no-runtime, framework-agnostic | Static sites where you don't want a framework |
| **Hotwire / HTMX / Turbo** | MPA + partial updates | Server-driven UI, no SPA tax | Rails/Django/Laravel teams; CRUD-heavy apps |

> **In practice:** the React-vs-everyone-else story matters less than it did. RSC + streaming has narrowed React's gap on bundle size; islands frameworks show that "less JS" is winning ground. The right answer is increasingly *the framework whose model fits your content shape*, not "React because everyone uses React."

---

## Migration Paths

Common migrations and what to watch for:

### CSR SPA → SSR (Next.js / Remix)
- Audit every `useEffect`-driven render — many become server-side data fetches
- Watch for `window`/`document` access — must guard for server runtime
- Hydration mismatches will surface immediately (different render server vs client)
- Cookies and auth move from client to server; rework auth flow accordingly
- **Wins:** much better LCP, real SEO, better DX once you're past the migration hump

### MPA → SPA
- Build a routing layer (React Router, TanStack Router, or framework router)
- Re-implement scroll restoration, focus management, head metadata per route
- Set up analytics page-tracking on route change
- Establish error boundaries, loading states, route-level code splitting
- **Watch out for:** the "SPA tax" — accessibility, SEO, and UX bugs that the browser handled for free in MPA mode

### SPA → MPA + Islands
- Identify which interactive parts truly need a JS framework
- Migrate static content to the islands framework (Astro, Fresh)
- Keep complex interactive flows as islands or separate SPA areas
- **Wins:** dramatic CWV improvements, smaller bundles, simpler hosting

### SSG → ISR
- Identify routes where staleness causes problems
- Set sensible `revalidate` TTLs per content type
- Add on-demand revalidation hooks for editorial publishes
- **Watch out for:** the first-after-revalidation user seeing a stale page (educate stakeholders)

### SSR → Edge SSR
- Audit dependencies for edge-runtime compatibility (no native Node modules)
- Replace traditional DB clients with edge-friendly drivers
- Re-evaluate data residency / compliance constraints
- **Wins:** dramatic global TTFB improvements; **risk:** subtle Node API gaps

---

## Precautions and Common Pitfalls

### 1. Don't pick the strategy before understanding the content
"We're using Next.js" isn't a strategy. Each route has a render strategy; pick deliberately.

### 2. Don't ship a SPA for a content site
A blog or marketing site does not need React Router and 300KB of JS. The bundle tax will haunt your CWV forever. Use SSG/Islands.

### 3. Don't ship pure CSR for public, indexable pages
Googlebot will eventually render, but with delay and inconsistency. Other crawlers (Bing, social previewers, AI engines, regional engines) often won't. You'll lose half your link previews and a chunk of your organic traffic.

### 4. Don't forget hydration is not free
Even SSR ships JS to make the page interactive. Hydration is CPU-heavy on mid-tier mobile. Strategies that reduce hydration (Islands, RSC, Resumability) are increasingly the answer for content-leaning sites.

### 5. Don't cache what shouldn't be cached
Auth-gated, per-user content cached at the CDN is a security incident waiting to happen. Use `Cache-Control: private, no-store` for personalized responses or render them outside cache boundaries.

### 6. Don't underestimate ISR cache invalidation
"We'll just bump the revalidate TTL" works until editorial wants instant publishes, or your top product changes price. Plan on-demand invalidation from day one if content cadence is editorial-driven.

### 7. Don't rely on Googlebot rendering for SEO
Crawl + render is delayed (sometimes by days) and inconsistent. Anything important — title, description, canonical, structured data, primary content — must be in the **initial HTML response**.

### 8. Don't conflate framework choice with architecture
Next.js can do CSR, SSR, SSG, ISR, Streaming, PPR, and Edge. The framework gives you the menu; *you* still pick the meal per route.

### 9. Don't migrate everything at once
Big-bang rewrites fail. Migrate route-by-route or section-by-section behind a reverse proxy. Validate each migrated section with real user metrics before moving on.

### 10. Don't chase the new shiny without measurement
RSC, PPR, Resumability — every couple of years there's a new paradigm. Most teams' performance problems are *image weight, third-party scripts, and untracked bundle bloat*, not their render strategy. Fix those first; they're 10x the leverage.

### 11. Don't forget the "long session" SPA problem
SPAs run for hours; memory leaks, stale tokens, and accumulated DOM nodes degrade performance over time. SSR/MPA pages are inherently fresh — every navigation is a clean slate. Test SPAs with multi-hour sessions before shipping.

### 12. Don't overlook bfcache
Browsers' back/forward cache restores pages instantly — effectively zero LCP/INP on back navigation. SPAs can break bfcache trivially (e.g., `unload` listeners, `Cache-Control: no-store`, certain WebSocket usage). Audit with Lighthouse's bfcache report.

### 13. Don't pick edge for everything
Edge runtimes have real limits — package compatibility, DB driver compatibility, runtime API surface. For DB-heavy SSR, origin region with a global CDN cache often beats edge SSR in practice.

### 14. Don't confuse "rendered on server" with "fast"
A 4-second SSR render produces a 4-second TTFB and a worse LCP than CSR. SSR speed is a function of *your data layer* — slow APIs make slow SSR. Fix the data layer first.

### 15. Don't mix too many frameworks
"Astro with React + Vue + Svelte islands" is technically possible, rarely a good idea. Each framework adds runtime, build-time, hiring, and maintenance cost. Pick one and stick with it unless you have a *very* specific reason.

---

## Decision Cheat Sheet

```
                                    Public + indexable?
                              ┌──────────┴──────────┐
                             No                    Yes
                              │                     │
                          [ CSR SPA ]          Per-user content?
                                            ┌──────┴──────┐
                                           No            Yes
                                            │             │
                                  Many URLs / changes?    │
                                  ┌────────┴────────┐     │
                                Often              Rarely │
                                  │                 │     │
                            [ ISR / SSR ]       [ SSG ]   │
                                                          │
                                              Mostly static shell?
                                              ┌────┴────┐
                                            Yes        No
                                              │         │
                                         [ PPR /     [ SSR /
                                         Islands ]   Streaming SSR ]
```

---

## Quick Reference: When to Use What

| Page type | Recommended strategy |
|---|---|
| Marketing landing page | SSG or PPR |
| Blog post | SSG with ISR |
| Documentation | SSG / Islands |
| Product detail (e-commerce) | ISR with on-demand revalidation, or PPR |
| Category / search results | Streaming SSR |
| Pricing page | SSG (with client-side currency switcher) |
| User dashboard (authed) | SSR or CSR SPA with shell SSR |
| Admin panel | CSR SPA |
| Real-time collaboration tool | CSR SPA + WebSocket |
| Online IDE / editor | CSR SPA |
| Cart / checkout | SSR (uncached, per-user) |
| News article | ISR or SSG |
| Personalized homepage | Streaming SSR or PPR |
| Geo-targeted variant | Edge SSR or Edge middleware |
| API endpoint | n/a (this is a server route, not a render decision) |

---

## Further Reading

- **[web.dev — Rendering on the Web](https://web.dev/articles/rendering-on-the-web)** — Jason Miller's foundational article; required reading
- **[patterns.dev](https://patterns.dev/)** — Lydia Hallie & Addy Osmani; visual guides to rendering patterns
- **[Next.js Docs — Rendering](https://nextjs.org/docs/app/building-your-application/rendering)**
- **[Astro — Why Islands?](https://docs.astro.build/en/concepts/islands/)**
- **[Remix Philosophy](https://remix.run/docs/en/main/discussion/introduction)**
- **[Qwik — Resumability vs Hydration](https://qwik.dev/docs/concepts/resumable/)**
- **[The Cost of Hydration](https://www.builder.io/blog/hydration-is-pure-overhead)** — Misko Hevery (Qwik creator's perspective)
- **[A Visual Guide to React Server Components](https://www.joshwcomeau.com/react/server-components/)** — Josh W. Comeau
- **[View Transitions API](https://developer.chrome.com/docs/web-platform/view-transitions)** — for MPA-as-SPA UX
- **[Speculation Rules API](https://developer.chrome.com/docs/web-platform/prerender-pages)**
- Books: *Web Application Architecture* (Shklar & Rosen), *Building Micro-Frontends* (Luca Mezzalira)

---

> **Closing thought:** the rendering strategy debate has shifted. A decade ago, SPA was the answer to everything; five years ago, SSR was the rebuttal. Today the question isn't "SPA or SSR?" — it's "which strategy *per route*, and how do they compose?" The habit I try to build on every team I work with is the **per-route decision**: this page is static, this one streams, this one is an island, this one is client-only — and we should be able to defend each choice in one sentence. The framework is a tool; the architecture is the choice. Once you stop treating "the framework's default" as the answer, you stop fighting the platform and start using it.
