# Web Application Optimization Techniques

> A practical playbook of techniques to make web apps fast — bundle, runtime, network, rendering, and data-layer optimizations, with React-specific patterns.

---

## Table of Contents

1. [The Optimization Mindset](#the-optimization-mindset)
2. [The Optimization Map](#the-optimization-map)
3. [Bundle & Code Optimization](#bundle--code-optimization)
   - [Code Splitting](#code-splitting)
   - [Dynamic Imports](#dynamic-imports)
   - [Tree Shaking & Dead Code Elimination](#tree-shaking--dead-code-elimination)
   - [Bundle Analysis](#bundle-analysis)
   - [Module/Nomodule, Modern Builds](#modulenomodule-modern-builds)
   - [Compression](#compression)
4. [Caching Strategies](#caching-strategies)
   - [The Cache Hierarchy](#the-cache-hierarchy)
   - [HTTP / Browser Caching](#http--browser-caching)
   - [CDN Caching](#cdn-caching)
   - [Service Worker Caching](#service-worker-caching)
   - [In-Memory & Application Caching](#in-memory--application-caching)
   - [API Response Caching](#api-response-caching)
   - [IndexedDB & Local Persistence](#indexeddb--local-persistence)
   - [Cache Invalidation Strategies](#cache-invalidation-strategies)
5. [Rendering Optimization (React)](#rendering-optimization-react)
   - [Why Components Re-render](#why-components-re-render)
   - [Preventing Unnecessary Re-renders](#preventing-unnecessary-re-renders)
   - [State Colocation & Lifting Down](#state-colocation--lifting-down)
   - [Concurrent React: Transitions & Deferred Values](#concurrent-react-transitions--deferred-values)
6. [List & Scrolling Optimization](#list--scrolling-optimization)
   - [Virtualization (Windowing)](#virtualization-windowing)
   - [Infinite Scrolling](#infinite-scrolling)
   - [Pagination Strategies](#pagination-strategies)
7. [Network Optimization](#network-optimization)
   - [Resource Hints](#resource-hints)
   - [Request Deduplication & Batching](#request-deduplication--batching)
   - [HTTP/2, HTTP/3, and Connection Reuse](#http2-http3-and-connection-reuse)
8. [Asset Optimization](#asset-optimization)
   - [Images](#images)
   - [Fonts](#fonts)
   - [Video & Audio](#video--audio)
9. [Runtime & Main-Thread Optimization](#runtime--main-thread-optimization)
   - [Web Workers](#web-workers)
   - [Scheduling APIs](#scheduling-apis)
   - [Avoiding Layout Thrash](#avoiding-layout-thrash)
   - [Debounce, Throttle, Idle](#debounce-throttle-idle)
10. [Data-Layer Optimization](#data-layer-optimization)
11. [Server & Edge Optimization](#server--edge-optimization)
12. [Memory Optimization](#memory-optimization)
13. [Measurement-First Workflow](#measurement-first-workflow)
14. [Common Anti-Patterns](#common-anti-patterns)
15. [Quick Reference Checklist](#quick-reference-checklist)
16. [Further Reading](#further-reading)

---

## The Optimization Mindset

Before tactics, the principles:

### 1. Measure first, optimize second
"My gut says X is slow" is not data. **Profile, identify the actual bottleneck, optimize, re-measure.** Engineers who optimize without measuring usually make things worse — they add complexity to solve the wrong problem.

### 2. Optimize the largest cost
80% of users feel 20% of the wins. The LCP image, the render-blocking script, the synchronous data fetch — these are 10–100× more impactful than micro-optimizations.

### 3. Optimize for the user, not the score
A 95 Lighthouse score on your laptop ≠ a fast experience for a user on a mid-tier Android over 4G. Field metrics (RUM, CrUX) are the source of truth.

### 4. Optimization is a budget, not a tactic
Set per-route performance budgets and enforce in CI. Slow pages don't ship by accident; they ship because nothing stops them. Make slow code a build failure.

### 5. The browser is on your side
Modern browsers do enormous work to optimize bad code (preload scanners, layout incrementalization, GPU compositing, bfcache). Use the platform's primitives instead of fighting them with JS.

### 6. Premature optimization is real
Knuth's full quote — "premature optimization is the root of all evil" — assumes you've measured. Don't optimize speculatively; don't *not* design for performance.

---

## The Optimization Map

Every web app has the same set of levers. The skill is knowing **which lever maps to which symptom**.

```
┌────────────────────────────────────────────────────────────────────┐
│                       OPTIMIZATION MAP                             │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  Symptom              Likely lever                                 │
│  ─────────            ─────────────                                │
│  Slow LCP        ─►   Bundle, network, image, TTFB, render-block   │
│  Slow INP        ─►   Long tasks, hydration, third-party JS        │
│  High CLS        ─►   Reserved space, fonts, lazy content          │
│  Slow nav        ─►   Code splitting, prefetch, bfcache            │
│  Janky scroll    ─►   Layers, passive listeners, virtualization    │
│  Slow data       ─►   API caching, batching, deduplication         │
│  Re-render churn ─►   Memo, state placement, derivation            │
│  Memory growth   ─►   Listener cleanup, ref leaks, large caches    │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

The rest of this document is a catalog of levers, organized by where they sit in the stack.

---

## Bundle & Code Optimization

### Code Splitting

**Goal:** ship only the JavaScript needed for the current page, not the whole app.

```
Without code splitting:               With code splitting:

┌───────────────────────┐           ┌─────────────┐
│   one bundle: 1.2 MB  │           │  main: 80KB │ ← always loaded
│  (all routes)         │           ├─────────────┤
│                       │           │ home: 40KB  │ ← loaded on /
│                       │           ├─────────────┤
│                       │           │ shop: 60KB  │ ← loaded on /shop
│                       │           ├─────────────┤
│                       │           │admin: 200KB │ ← loaded on /admin
└───────────────────────┘           └─────────────┘
```

#### Strategies
- **Route-based splitting** — one chunk per route. The default; biggest win for most apps.
- **Component-based splitting** — heavy components (charts, editors, modals) split independently.
- **Vendor splitting** — separate chunk for `node_modules`. Long-term cacheable across deploys.
- **Library-specific splitting** — pull out chart.js, monaco, three.js into their own chunks.

#### React route-based splitting
```jsx
import { lazy, Suspense } from 'react';

const Dashboard = lazy(() => import('./Dashboard'));
const Reports   = lazy(() => import('./Reports'));

<Suspense fallback={<PageSkeleton />}>
  <Routes>
    <Route path="/dashboard" element={<Dashboard />} />
    <Route path="/reports"   element={<Reports />} />
  </Routes>
</Suspense>
```

In Next.js / Remix / Astro, route-level splitting is the default — you get it for free.

#### Trade-offs
- **More chunks = more requests**, but HTTP/2/3 handles parallelism well
- **Lazy-loading on click** introduces a delay — preload on hover/intent for snappier UX
- **Loading states** must be designed (skeletons, suspense fallbacks)

---

### Dynamic Imports

`import()` is the runtime primitive that powers code splitting.

```js
// Static import — bundled with parent
import { processData } from './heavy-lib';

// Dynamic import — separate chunk, loaded on demand
const { processData } = await import('./heavy-lib');
```

#### Use cases
- **Lazy heavy features** — markdown editor, charting, PDF generator
- **Conditional features** — only load admin code if user is an admin
- **Polyfills** — only load polyfills for browsers that need them
- **A/B variant code** — load only the assigned variant

```js
// Conditional polyfill
if (!('IntersectionObserver' in window)) {
  await import('intersection-observer');
}

// User-tier feature gating
if (user.tier === 'pro') {
  const { AdvancedEditor } = await import('./AdvancedEditor');
  mount(AdvancedEditor);
}
```

#### Preloading dynamic chunks
Don't wait until click — start loading on **intent** (hover, focus, viewport entry).

```js
// On hover — start fetching the chunk
button.addEventListener('mouseenter', () => {
  import(/* webpackPrefetch: true */ './HeavyModal');
});
```

Webpack/Rollup magic comments:
- `/* webpackPrefetch: true */` — load when idle (low priority)
- `/* webpackPreload: true */` — load with parent (high priority)

---

### Tree Shaking & Dead Code Elimination

**Tree shaking** removes unused exports from ES modules at build time.

```js
// utils.js
export const a = () => '...';
export const b = () => '...';

// app.js
import { a } from './utils';
//      ↑ b is dead — bundler removes it from the output
```

#### Requirements
- ES modules (`import`/`export`), not CommonJS (`require`)
- Pure exports (no side effects on import) — mark with `"sideEffects": false` in `package.json`
- Bundler that supports it — Rollup, Vite, esbuild, Webpack all do

#### Common defeats
- **Side-effectful imports** (`import 'polyfill';`) — these stay in the bundle
- **`export *` re-exports** can prevent shaking with some bundlers
- **Dynamic property access** (`utils[methodName]()`) — bundler can't statically analyze

#### Tools that benefit massively
- `lodash` → use `lodash-es` or per-function imports (`import debounce from 'lodash/debounce'`)
- `moment.js` → switch to `date-fns` or `dayjs` (tree-shakeable, smaller)
- Material UI / Ant Design — use named imports, not full-library imports

---

### Bundle Analysis

You cannot optimize what you cannot see. **Analyze every release.**

#### Tools
- `webpack-bundle-analyzer` / `rollup-plugin-visualizer` — interactive treemaps
- `source-map-explorer` — works with any sourcemap-producing build
- `bundlephobia` (web) — check a package's size before adding it
- `import-cost` (VSCode extension) — inline size annotations
- `size-limit` / `bundlewatch` — CI-enforced budgets

#### What to look for
- Duplicate dependencies (multiple versions of the same lib)
- Heavy hitters (anything > 50KB minified that you didn't expect)
- Locale data (moment.js ships ~70KB of locales by default — tree-shake or switch)
- Polyfills shipping to modern browsers
- Source maps accidentally in production bundles

---

### Module/Nomodule, Modern Builds

Ship modern JS to modern browsers; transpile only for legacy.

```html
<script type="module"   src="/app.modern.js"></script>
<script nomodule        src="/app.legacy.js"></script>
```

Modern browsers ignore `nomodule`; legacy browsers ignore `type="module"`. Modern bundles can be 20–40% smaller (no class transpilation, no async/await rewriting, no spread polyfills).

In 2026, most teams have dropped IE11 entirely — a single modern build is fine. Check your analytics before shipping legacy bundles "just in case."

---

### Compression

Always compress text resources at the network layer.

| Format | Compression ratio | Notes |
|---|---|---|
| **Brotli (`br`)** | Best | ~20% smaller than gzip; pre-compress static assets at level 11 |
| **gzip** | Good | Universal fallback |
| **zstd** | Excellent | Some CDNs (Cloudflare); growing support |

Pre-compress static assets at build time (`brotli -11 app.js`); leave dynamic responses to runtime compression at level 4–6.

**Don't double-compress** images, video, fonts (already in WOFF2). Wastes CPU; can make files larger.

---

## Caching Strategies

Caching is the highest-leverage optimization technique. **The fastest request is the one you don't make.**

### The Cache Hierarchy

```
fetch()
   │
   ▼
┌────────────────────────────┐
│   Service Worker           │ ← Custom strategies (offline-first, etc.)
│   (controlled by your JS)  │
├────────────────────────────┤
│   HTTP Cache (Browser)     │ ← Memory cache → Disk cache
│   • Memory: tab session    │
│   • Disk: persistent       │
├────────────────────────────┤
│   CDN Cache (PoP)          │ ← Edge close to user
├────────────────────────────┤
│   Reverse proxy / origin   │ ← Varnish, Nginx, app-level cache
│   cache (Redis/Memcached)  │
├────────────────────────────┤
│   Database / origin        │ ← Last resort
└────────────────────────────┘
```

Each layer adds 1-2ms (memory), 10-50ms (CDN), or hundreds of ms (origin). **Catch the request as high in the stack as possible.**

---

### HTTP / Browser Caching

The HTTP cache is automatic, controlled by response headers, and works in every browser.

```
Cache-Control: public, max-age=31536000, immutable
ETag: "abc123"
Last-Modified: Wed, 30 Apr 2026 10:00:00 GMT
Vary: Accept-Encoding
```

#### Key directives
| Directive | Effect |
|---|---|
| `public` | Any cache (browser + CDN) may store |
| `private` | Only the user's browser |
| `max-age=N` | Fresh for N seconds |
| `s-maxage=N` | Like max-age, but only for shared caches (CDN) |
| `no-cache` | Cache, but **revalidate** every time |
| `no-store` | Never cache (use sparingly — kills bfcache!) |
| `immutable` | Tell browser the content will never change — skip revalidation |
| `stale-while-revalidate=N` | Serve stale for N seconds while refetching in background |
| `stale-if-error=N` | Serve stale if origin errors |

#### The two-tier strategy
- **Hashed assets** (`/app.a1b2c3.js`) → `Cache-Control: public, max-age=31536000, immutable`
  - Hash changes on content change → unique URL → cache forever
- **HTML** → `Cache-Control: no-cache` (or `max-age=0, s-maxage=300, stale-while-revalidate=86400`)
  - HTML must be fresh because it references hashed assets

#### Conditional requests (304 Not Modified)
When `max-age` expires, the browser sends:
```
GET /api/data
If-None-Match: "abc123"
```
Server replies `304 Not Modified` (no body) or `200` with new content. Saves bandwidth, not the round trip.

#### `Vary` matters
`Vary: Accept-Encoding` means the cache should treat gzip and Brotli responses separately. Get this wrong and you serve gzip to clients that asked for Brotli.

---

### CDN Caching

A CDN (Cloudflare, Fastly, Akamai, CloudFront, Vercel Edge) is the single biggest TTFB optimization for global audiences.

#### Strategies
- **Static assets** → cache forever, purge on deploy
- **HTML** → short TTL + stale-while-revalidate (e.g., 60s fresh, 24h stale)
- **API responses** → `s-maxage` for cacheable reads
- **Personalized content** → `Cache-Control: private` or use edge personalization (ESI, edge functions)

#### Cache keys
The CDN keys requests on URL + Vary headers. Watch out for:
- Tracking query params (`?utm_source=...`) — strip them or normalize the cache key
- Cookies — by default, many CDNs bypass cache when a cookie is present (configurable)
- Auth headers — must bypass cache for auth-gated content

#### Purge & invalidation
- **Tag-based purge** (Fastly, Cloudflare Enterprise) — `purge_tag: "post-123"`; perfect for editorial workflows
- **URL purge** — invalidate by exact path
- **Soft purge** — mark stale, serve via `stale-while-revalidate` while refetching

#### Cache key normalization
Aggressively normalize URLs so the same logical resource hits the same cache entry:
- Trailing slash
- Lowercase
- Sorted query params
- Strip tracking params

---

### Service Worker Caching

A Service Worker is a JS proxy between the page and the network, with full programmatic control over the cache. Powers PWAs and offline-first experiences.

```
Page  ──fetch──►  Service Worker  ──►  Cache Storage
                       │                    │
                       └──network────────► Origin
```

#### Common strategies
| Strategy | Behavior | Use case |
|---|---|---|
| **Cache-first** | Try cache; fall back to network | Static assets, fonts |
| **Network-first** | Try network; fall back to cache | HTML, news content |
| **Stale-while-revalidate** | Return cache instantly; update in background | Avatars, semi-static API data |
| **Cache-only** | Cache or fail | Offline-only assets |
| **Network-only** | Bypass cache | Analytics, real-time |

```js
// Stale-while-revalidate
self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.open('v1').then(async (cache) => {
      const cached = await cache.match(event.request);
      const network = fetch(event.request).then((res) => {
        cache.put(event.request, res.clone());
        return res;
      });
      return cached || network;
    })
  );
});
```

#### Tools
- **Workbox** (Google) — battle-tested SW abstractions; the de facto choice
- **vite-plugin-pwa** / **next-pwa** — framework integration

#### Caveats
- **SW updates are tricky** — bad strategies can serve stale forever
- **Debugging is hard** — DevTools → Application → Service Workers
- **Versioning your cache names** is mandatory; clean up old caches on activate
- **Skip-waiting** prematurely can break tabs in flight

> **Lesson learned:** Service Workers are powerful but easy to misuse. The classic failure mode: you ship a buggy SW, and every user is now stuck on it for 24h. The first time this happens to you is a week you don't get back — **always have an "unregister and clear" escape hatch shipped from day one.**

---

### In-Memory & Application Caching

Within the running app, cache derived data and API responses.

#### Memoization
```js
const memo = new Map();
function expensive(key) {
  if (memo.has(key)) return memo.get(key);
  const result = compute(key);
  memo.set(key, result);
  return result;
}
```

For React: `useMemo`, `useCallback`, `React.memo`, `reselect`. Caveat: memoization isn't free — equality checks cost CPU. Memoize **measured hot paths**, not everything.

#### LRU caches
For bounded memory, use an LRU (least-recently-used) cache. Libraries: `lru-cache`, `quick-lru`, `mnemonist/lru-cache`.

```js
const cache = new LRU({ max: 500, ttl: 60_000 });
cache.set('user:42', user);
cache.get('user:42'); // hits
```

---

### API Response Caching

Modern frontends almost universally use a **server-state library** that handles caching, deduplication, revalidation, and optimistic updates:

| Library | Best for |
|---|---|
| **TanStack Query** (React/Vue/Svelte/Solid) | The most popular; cache + dedup + SWR + mutations |
| **SWR** (Vercel) | Lighter than TanStack Query; "stale-while-revalidate" focused |
| **RTK Query** (Redux Toolkit) | Tight Redux integration; codegen from OpenAPI |
| **Apollo Client** / **urql** / **Relay** | GraphQL |

#### What they give you
- Automatic deduplication (10 components asking for the same data → 1 request)
- Caching keyed by query parameters
- Background refetching on window focus, network reconnect, interval
- Optimistic updates with rollback on error
- Pagination + infinite query helpers
- Mutation invalidation (refetch dependent queries after a write)

```jsx
// TanStack Query
const { data, isPending } = useQuery({
  queryKey: ['user', id],
  queryFn: () => fetchUser(id),
  staleTime: 60_000,         // fresh for 1 min
  gcTime: 5 * 60_000,        // keep in cache for 5 min after last use
});
```

#### A reframe worth internalizing: server state vs client state
The single biggest state-management win in modern frontends: **most "global state" was actually server state poorly cached.** Once TanStack Query / SWR took over data fetching, Redux store sizes shrank by ~80%. Stop treating server data as Redux state.

---

### IndexedDB & Local Persistence

For client-side persistence beyond the HTTP cache: **IndexedDB**.

| Storage | Capacity | Sync? | Use |
|---|---|---|---|
| `localStorage` | ~5MB | Sync (blocks main thread!) | Tiny key/value, prefs, tokens |
| `sessionStorage` | ~5MB | Sync | Tab-scoped data |
| `IndexedDB` | GBs | Async | Structured data, offline, large datasets |
| `Cache Storage` | GBs | Async | Service-worker-cached responses |
| `OPFS` (Origin Private File System) | Large | Async | High-perf file I/O (newer) |

**Avoid `localStorage` for anything non-trivial** — it's synchronous and blocks the main thread.

#### Libraries
- **`idb`** (Jake Archibald) — Promise-wrapped IndexedDB; the standard
- **`Dexie.js`** — query API on top of IndexedDB
- **`localForage`** — drop-in for `localStorage` with IndexedDB backing
- **TanStack Query persister** — persist query cache between sessions

#### Use cases
- Offline support (PWAs)
- Large dataset clients (CRMs, analytics dashboards)
- Avoiding refetch after page reload
- Optimistic UI persistence

---

### Cache Invalidation Strategies

> "There are only two hard things in computer science: cache invalidation and naming things." — Phil Karlton

#### Time-based
Set a TTL; cache expires automatically. Simple, but stale until TTL elapses.

#### Event-based
Invalidate when something happens (mutation, webhook, message). Lower staleness, more complexity.

#### Tag-based
Group cached items under tags; invalidate the tag.
```
cache.set('post:42', data, { tags: ['posts'] });
cache.invalidate({ tag: 'posts' });
```

#### Versioned URLs (the easiest)
Don't invalidate — change the URL. Hashed asset names; query-string version bumps. Cache forever; the new content has a new key.

#### Stale-while-revalidate
Serve stale instantly; refetch in background. Excellent UX trade-off. Built into HTTP, CDNs, TanStack Query, SWR.

#### The trap to watch for
The hardest invalidation problems are **derivation chains** — if `A` depends on `B` which depends on `C`, invalidating `C` should cascade. Most caching layers don't model this. Be conservative: when in doubt, invalidate broader. Stale data causes incidents; an extra refetch costs milliseconds.

---

## Rendering Optimization (React)

### Why Components Re-render

A React component re-renders when:
1. Its **state** changes (`useState`, `useReducer`)
2. Its **props** change (parent passed new values)
3. Its **parent re-renders** (default cascade)
4. A **subscribed context** value changes
5. A **subscribed external store** changes (Redux, Zustand, etc.)

Re-rendering is **not** the same as the DOM updating. React diffs the virtual DOM and only commits real changes. But re-rendering still costs CPU — running the function, reconciling children, running effects.

### The cost
A single re-render is cheap. **A re-render of 500 components in response to every keystroke is not.** That's where INP regressions come from.

---

### Preventing Unnecessary Re-renders

#### 1. `React.memo`
Wraps a component; only re-renders when props change (shallow compare).

```jsx
const Row = React.memo(function Row({ item, onClick }) {
  return <li onClick={onClick}>{item.name}</li>;
});
```

**Common defeat:** passing inline objects/functions:
```jsx
// ❌ Object literal is new every render — memo useless
<Row item={item} onClick={() => select(item.id)} />

// ✅ Stable references
const handleClick = useCallback(() => select(item.id), [item.id, select]);
<Row item={item} onClick={handleClick} />
```

#### 2. `useMemo` for derived data
```jsx
const sorted = useMemo(() => heavy(data), [data]);
```
Don't memoize trivially-cheap computations — the equality check + closure overhead exceeds the savings.

#### 3. `useCallback` for stable function identity
Useful when a function is passed to memoized children or used as an effect dep. Otherwise, it's noise.

#### 4. State colocation
Move state **down** to the smallest component that needs it. Top-level state re-renders the whole tree; local state re-renders only the leaf.

```jsx
// ❌ Top-level: every input keystroke re-renders Page
function Page() {
  const [name, setName] = useState('');
  return <>
    <Header />
    <Input value={name} onChange={setName} />
    <Footer />
  </>;
}

// ✅ Localized: only Input re-renders
function Page() {
  return <>
    <Header />
    <NameInput />
    <Footer />
  </>;
}
function NameInput() {
  const [name, setName] = useState('');
  return <Input value={name} onChange={setName} />;
}
```

#### 5. Context splitting
`useContext` causes every consumer to re-render when the context value changes. **Split contexts** so consumers only subscribe to what they need.

```jsx
// ❌ Single context with many fields
<UserContext.Provider value={{ user, theme, locale, notifications }}>

// ✅ Split contexts
<UserContext.Provider value={user}>
  <ThemeContext.Provider value={theme}>
    <LocaleContext.Provider value={locale}>
```

Or use **selector-based stores** (Zustand, Jotai, Redux+useSelector) where consumers subscribe to slices.

#### 6. Stable references everywhere
Memoize props, especially objects/arrays/functions passed to memoized components.

#### 7. Discriminate "real" state from "derived" state
Don't store anything you can compute. State that mirrors props or that you derive in `useEffect` is almost always a bug — compute it during render.

#### 8. Keys
Stable, unique `key` props let React reuse DOM nodes across re-renders. **Never use array index as key** for lists that reorder; insertion in the middle remounts every item below.

#### 9. Profile before optimizing
**Most apps don't have a re-render problem.** Use React DevTools Profiler to find the actual hot path before adding `memo` everywhere — over-memoization adds complexity *and* cost.

---

### State Colocation & Lifting Down

The default React mental model is "lift state up to the closest common ancestor." Mature practice extends it: **lift state up only as far as necessary, and lift it back down when possible.**

When you find:
- A state value used by exactly one subtree
- That state is causing parent-level re-renders unrelated to the rest of the page

→ **lift it down** into the subtree that uses it.

This pairs with **component composition**: pass children as `children` prop so they don't re-render when the parent's state changes.

```jsx
// State change causes <Slow /> to re-render
function Page() {
  const [count, setCount] = useState(0);
  return <>
    <button onClick={() => setCount(c => c + 1)}>{count}</button>
    <Slow />
  </>;
}

// Pass <Slow /> as children — it's stable across count changes
function Page({ children }) {
  const [count, setCount] = useState(0);
  return <>
    <button onClick={() => setCount(c => c + 1)}>{count}</button>
    {children}
  </>;
}
<Page><Slow /></Page>
```

---

### Concurrent React: Transitions & Deferred Values

React 18+ has scheduling primitives that let you mark updates as **non-urgent**, so the browser can interrupt them for user input.

#### `useTransition`
```jsx
const [isPending, startTransition] = useTransition();

const onChange = (e) => {
  setQuery(e.target.value); // urgent (input value)
  startTransition(() => {
    setFilteredList(filter(list, e.target.value)); // non-urgent
  });
};
```
The expensive filter happens **without blocking the input**. INP-saving magic.

#### `useDeferredValue`
```jsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => search(deferredQuery), [deferredQuery]);
```
The input is always responsive; the search lags behind under load and catches up.

#### React Server Components
Heavy work moves entirely off the client. Components that don't need interactivity ship zero JS. Massive INP and bundle wins for content-heavy pages.

---

## List & Scrolling Optimization

### Virtualization (Windowing)

Render only the visible portion of a long list, plus a small buffer (the "window"). Recycle DOM nodes as the user scrolls.

```
Scroll position: 1000px

Without virtualization:           With virtualization:

┌─────────────┐                  ┌─────────────┐
│ Row 1       │ (rendered)       │             │ (not rendered)
│ Row 2       │                  │             │
│ ...         │                  │             │
│ Row 50      │ ← visible        │ Row 50      │ ← visible
│ Row 51      │ ← visible        │ Row 51      │ ← visible
│ Row 52      │ ← visible        │ Row 52      │ ← visible
│ ...         │                  │             │
│ Row 10000   │ (rendered)       │             │ (not rendered)
└─────────────┘                  └─────────────┘
  10,000 nodes                     ~30 nodes
```

#### When to virtualize
- Lists > ~200 items, or
- Tables with many columns, or
- Anywhere render time scales linearly with list length

#### Libraries
- **TanStack Virtual** (modern, framework-agnostic) — the recommended pick
- **react-window** (Brian Vaughn) — lightweight, simple
- **react-virtuoso** — feature-rich, supports variable heights and grouping
- **AG Grid**, **TanStack Table** — for full-featured data grids

#### Trade-offs
- **Browser Ctrl-F** doesn't find virtualized rows (they're not in the DOM)
- **Accessibility** is nuanced — use `aria-rowindex` / `aria-setsize` so screen readers announce position correctly
- **Variable heights** require measurement; can cause scroll jumps if mishandled
- **SEO** — virtualized content isn't crawlable; not a concern for authed apps but a problem for public lists

---

### Infinite Scrolling

Load more items as the user scrolls toward the end.

#### Implementation
Use the **Intersection Observer API** instead of scroll listeners:

```jsx
const sentinelRef = useRef(null);

useEffect(() => {
  const observer = new IntersectionObserver(([entry]) => {
    if (entry.isIntersecting) loadMore();
  });
  if (sentinelRef.current) observer.observe(sentinelRef.current);
  return () => observer.disconnect();
}, [loadMore]);

return <>
  <List items={items} />
  <div ref={sentinelRef} />
</>;
```

Or use TanStack Query's `useInfiniteQuery`:
```js
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({
  queryKey: ['posts'],
  queryFn: ({ pageParam = 0 }) => fetchPosts(pageParam),
  getNextPageParam: (last) => last.nextCursor,
});
```

#### Combine with virtualization
Long infinite lists must be virtualized — without windowing, the DOM grows unbounded as the user scrolls and the page eventually crawls.

#### UX caveats
- **Footer becomes unreachable** — users can't reach a footer if scrolling always loads more
- **Scroll restoration** on back-navigation is hard — store the loaded state
- **Cumulative cognitive load** — users lose track of position
- **Accessibility** — keyboard users struggle; provide a "load more" button as fallback or alternative

#### When to prefer pagination
- SEO-critical (search engines can crawl page links)
- Footer is meaningful (e-commerce, docs)
- User needs to deep-link or share specific pages
- Editorial workflows expecting "page 5 of N"

#### Hybrid: "load more" button
Best of both worlds for many cases — user-controlled fetching, no surprise scroll-jumps.

---

### Pagination Strategies

| Strategy | Pros | Cons |
|---|---|---|
| **Offset/limit** (`?page=3`) | Simple, deep-linkable | Slow on large datasets (DB scans the offset); inconsistent if list changes |
| **Cursor-based** (`?after=xyz`) | Stable across changes; fast | Can't jump to arbitrary page |
| **Keyset** (sort key + cursor) | Fast at any depth; stable | Complex with multi-column sort |

For infinite scrolling and modern APIs, **cursor-based** is the right default.

---

## Network Optimization

### Resource Hints

Tell the browser what to do with future resources:

| Hint | Purpose |
|---|---|
| `<link rel="dns-prefetch">` | Resolve a domain early |
| `<link rel="preconnect">` | DNS + TCP + TLS for an origin |
| `<link rel="preload">` | Fetch a known critical resource ASAP |
| `<link rel="modulepreload">` | Preload an ES module + its deps |
| `<link rel="prefetch">` | Fetch a likely-next resource at low priority |
| `<link rel="prerender">` (deprecated) → Speculation Rules | Prerender a likely next page |
| `fetchpriority="high"` | Prioritize an existing fetch (LCP image) |

```html
<!-- Critical font -->
<link rel="preload" as="font" type="font/woff2" href="/fonts/inter.woff2" crossorigin>

<!-- LCP image -->
<link rel="preload" as="image" href="/hero.webp" fetchpriority="high">

<!-- Likely next page -->
<link rel="prefetch" href="/checkout">

<!-- API host -->
<link rel="preconnect" href="https://api.example.com" crossorigin>
```

#### Speculation Rules API (modern prerender)
```html
<script type="speculationrules">
{
  "prerender": [{ "where": { "href_matches": "/products/*" }, "eagerness": "moderate" }]
}
</script>
```
Chrome prerenders likely-next-page documents — **near-instant** navigation. Combine with View Transitions for stunning UX.

#### A caveat
**Preload everything = preload nothing.** Each preload competes for bandwidth. Reserve for: LCP image, critical fonts, and (rarely) hero JSON.

---

### Request Deduplication & Batching

#### Deduplication
Multiple components asking for the same data → one request.

TanStack Query / SWR handle this automatically. For lower-level control:

```js
const inflight = new Map();
function dedupedFetch(key) {
  if (!inflight.has(key)) {
    inflight.set(key, fetch(key).finally(() => inflight.delete(key)));
  }
  return inflight.get(key);
}
```

#### Batching
Combine many small requests into one.
- **GraphQL** — natural batching (one query, many fields)
- **DataLoader** (Facebook) — server-side batching of N+1 queries
- **REST batch endpoints** — e.g., `GET /users?ids=1,2,3` instead of 3 separate calls
- **JSON-RPC batching** — array of requests in one call

#### Coalescing
Within a tick, queue calls and flush as one:
```js
const queued = new Set();
let flushScheduled = false;

function fetch(id) {
  queued.add(id);
  if (!flushScheduled) {
    flushScheduled = true;
    queueMicrotask(flush);
  }
  return ...; // promise resolved when flush completes
}
```

---

### HTTP/2, HTTP/3, and Connection Reuse

- **HTTP/2** multiplexes many requests over one TCP connection — eliminates head-of-line blocking *within* the connection. Universally supported; turn it on at the CDN.
- **HTTP/3** runs over QUIC (UDP); survives network changes; 0-RTT resumption. ~30% of traffic in 2026. Free perf on lossy networks.
- **Connection reuse** — keep-alive, connection pooling at the origin. A new TCP+TLS handshake costs ~200ms.

---

## Asset Optimization

### Images

The single largest category of bytes on most pages.

#### Format selection
| Format | Use |
|---|---|
| **AVIF** | Best compression; ~50% smaller than JPEG |
| **WebP** | ~30% smaller than JPEG; broad support |
| **JPEG** | Photo fallback |
| **PNG** | Transparent + lossless (icons, screenshots without photos) |
| **SVG** | Vector (logos, icons, charts) — scales infinitely, tiny |

Modern stack: serve AVIF with WebP fallback with JPEG fallback via `<picture>`:
```html
<picture>
  <source srcset="/img.avif" type="image/avif">
  <source srcset="/img.webp" type="image/webp">
  <img src="/img.jpg" alt="..." width="800" height="600" loading="lazy" decoding="async">
</picture>
```

#### Always set width and height
Prevents CLS. Browser computes `aspect-ratio` from the attributes and reserves space.

#### Responsive images
```html
<img
  srcset="/img-400.webp 400w, /img-800.webp 800w, /img-1600.webp 1600w"
  sizes="(max-width: 600px) 100vw, 50vw"
  src="/img-800.webp"
  alt="..."
>
```

#### Lazy loading
- `loading="lazy"` on below-the-fold images
- **Never** lazy-load the LCP image
- `decoding="async"` lets the browser decode off the main thread

#### Image CDNs
Cloudinary, Imgix, Vercel Image, Cloudflare Images, Bunny Optimizer — automatic format negotiation, on-the-fly resize, blur placeholders, smart cropping.

---

### Fonts

Fonts are render-blocking via FOIT (flash of invisible text); poorly handled fonts cause FOUT (flash of unstyled text) → CLS.

#### Best practices
- **WOFF2** only (universal support, smallest size)
- **Subset** — strip glyphs you don't use (a Latin subset is often <50KB; the full font is 500KB+)
- **`font-display: swap`** — show fallback immediately, swap when font loads (FOUT, no FOIT)
- **`font-display: optional`** — use fallback if font isn't ready in 100ms (no shift at all)
- **Preload** critical fonts:
  ```html
  <link rel="preload" as="font" type="font/woff2" href="/inter.woff2" crossorigin>
  ```
- **Match fallback metrics** with `size-adjust`, `ascent-override`, `descent-override` to eliminate CLS:
  ```css
  @font-face {
    font-family: 'Inter';
    src: url('/inter.woff2') format('woff2');
    font-display: swap;
    size-adjust: 100.06%;
    ascent-override: 90%;
  }
  ```
- **Variable fonts** — one file replaces multiple weights; usually a net win

#### Self-host vs Google Fonts
Self-hosting is faster (no extra origin handshake), more privacy-friendly, and gives you more control. Tools like `fontsource` make it trivial.

---

### Video & Audio

- **Modern codecs** — H.265/HEVC, AV1 (royalty-free, ~30% smaller than H.264)
- **Adaptive streaming** — HLS / DASH for variable bitrate based on bandwidth
- **`preload="none"`** for hero videos — don't auto-fetch large MP4s
- **Poster images** — `<video poster="...">` so users see something before play
- **Lazy-load** off-screen video iframes (YouTube/Vimeo) with `loading="lazy"` or facade pattern (image placeholder that swaps to iframe on click)

---

## Runtime & Main-Thread Optimization

### Web Workers

Move CPU-bound work off the main thread.

```js
// main.js
const worker = new Worker('/parser.js', { type: 'module' });
worker.postMessage({ csv: data });
worker.onmessage = (e) => render(e.data.parsed);

// parser.js
self.onmessage = (e) => {
  const parsed = expensiveParse(e.data.csv);
  self.postMessage({ parsed });
};
```

#### Use cases
- Parsing large JSON / CSV / XML
- Search indexing (lunr, FlexSearch)
- Image processing
- Crypto, hashing
- ML inference (TensorFlow.js, ONNX)

#### Trade-offs
- Message-passing overhead (cloning data) — for huge data, use `Transferable` objects (`ArrayBuffer`, `OffscreenCanvas`) to transfer ownership without copying
- No DOM access in workers (by design)
- Bundling for workers requires build-tool support — Vite, webpack, esbuild all do

#### `OffscreenCanvas`
Render canvas content from a worker — keeps animations smooth even when the main thread is busy.

---

### Scheduling APIs

Modern browser APIs to break work up:

#### `scheduler.postTask`
```js
scheduler.postTask(() => doExpensiveThing(), { priority: 'background' });
```
Priorities: `user-blocking`, `user-visible`, `background`.

#### `scheduler.yield`
```js
async function processList(items) {
  for (const item of items) {
    work(item);
    if (someCondition) await scheduler.yield(); // explicit yield to browser
  }
}
```

#### `requestIdleCallback`
```js
requestIdleCallback(() => doNonCriticalWork(), { timeout: 2000 });
```
Run only when the main thread is idle. Don't use for critical work — may never run on busy pages.

#### `requestAnimationFrame`
Run before the next paint (for animations and pre-paint reads/writes).

---

### Avoiding Layout Thrash

Don't read layout properties between writes — that forces synchronous layout recomputation.

```js
// ❌ Thrash: write, read, write, read...
items.forEach(item => {
  item.style.width = '100px';            // dirty layout
  const h = item.offsetHeight;           // FORCED LAYOUT
  item.style.height = h * 2 + 'px';      // dirty layout
  const w = item.offsetWidth;            // FORCED LAYOUT AGAIN
});

// ✅ Batch: read all, then write all
const heights = items.map(i => i.offsetHeight);   // one layout
items.forEach((item, i) => {
  item.style.width = '100px';
  item.style.height = heights[i] * 2 + 'px';     // batched into one layout next frame
});
```

Layout-forcing properties: `offsetHeight`, `clientHeight`, `scrollHeight`, `getBoundingClientRect()`, `getComputedStyle()` (for layout properties), `innerText`.

---

### Debounce, Throttle, Idle

| Pattern | When |
|---|---|
| **Debounce** | Wait for user to stop (search input, autosave) |
| **Throttle** | Cap frequency (scroll, resize, mousemove) |
| **rAF throttle** | Cap to refresh rate (scroll handlers driving DOM) |
| **Idle** | Run only when free (analytics, telemetry) |

```js
// Debounce
function debounce(fn, ms) {
  let t;
  return (...args) => {
    clearTimeout(t);
    t = setTimeout(() => fn(...args), ms);
  };
}

// rAF throttle for scroll
let queued = false;
window.addEventListener('scroll', () => {
  if (queued) return;
  queued = true;
  requestAnimationFrame(() => {
    handleScroll();
    queued = false;
  });
}, { passive: true });
```

#### `passive: true` on scroll/touch listeners
Tells the browser the listener won't `preventDefault()`. The browser can scroll without waiting for your JS — eliminates scroll jank from non-passive listeners.

---

## Data-Layer Optimization

### Reduce payload size
- **Field selection** — GraphQL, sparse fieldsets (`?fields=id,name`), JSON projection
- **Pagination** — never `SELECT *` from a 1M-row table
- **Compression** — gzip/brotli on JSON; binary protocols (Protobuf, MessagePack) for hot paths
- **Schemas** — Protobuf, FlatBuffers for ultra-low overhead

### Optimize the database
- **Index hot queries** — `EXPLAIN` everything that's user-facing
- **Avoid N+1** — use joins, DataLoader, prefetching
- **Read replicas** for read-heavy workloads
- **Materialized views** for expensive aggregations
- **Caching layer** (Redis) for hot reads
- **Connection pooling** (PgBouncer for Postgres)

### API design
- **Aggregation endpoints** — return everything a screen needs in one call
- **GraphQL or BFF** — let frontends ask for exactly what they need
- **HTTP/2 server push or 103 Early Hints** — start sending hints before the response

### Server-side rendering speed
- **Cache rendered HTML** at the edge (s-maxage)
- **Streaming** — flush bytes as soon as available, don't wait for the slowest dependency
- **Partial Prerendering** — static shell + dynamic islands

---

## Server & Edge Optimization

### Edge computing
Move logic close to the user (Cloudflare Workers, Vercel Edge, Deno Deploy):
- Auth checks, A/B routing, geo-redirects → milliseconds, not hundreds of milliseconds
- Caching personalized variants at the edge
- HTML rewriting (HTMLRewriter API)

### Streaming SSR
Flush HTML chunks as data resolves (`<Suspense>`):
```jsx
<Suspense fallback={<Skeleton />}>
  <SlowRecommendations />
</Suspense>
```

### Early Hints (HTTP 103)
Server replies with hints (preload links) before the actual response:
```
HTTP/2 103 Early Hints
Link: </app.css>; rel=preload; as=style
Link: </app.js>; rel=preload; as=script

HTTP/2 200 OK
...
```
The browser starts fetching critical resources while the server is still computing the response. Real-world LCP gains.

### Compression at the edge
Many CDNs offer Brotli at level 11 for static assets — compute-intensive but done once and cached forever. Worth it.

---

## Memory Optimization

SPAs that run for hours accumulate cruft. Memory leaks cause slow degradation.

### Common leaks
- **Forgotten event listeners** — always clean up in `useEffect` return / `componentWillUnmount`
- **Closures holding refs** — large objects captured in long-lived callbacks
- **Detached DOM nodes** — references to removed elements held in JS
- **Timer leaks** — `setInterval` not cleared on unmount
- **Subscription leaks** — observables, sockets not unsubscribed
- **Cache growth** — unbounded `Map`s; use LRU
- **Console logging objects** — DevTools holds them; turn off verbose logging in production

### Tools
- Chrome DevTools → Memory → Heap snapshot, Allocation timeline
- Performance panel → Memory checkbox
- `performance.memory.usedJSHeapSize` (Chrome only) for RUM monitoring
- Long-session smoke testing — keep an SPA tab open overnight; check memory growth

---

## Measurement-First Workflow

Optimize this way, every time:

```
1. Suspect a problem  ─►  Measure (DevTools, RUM, Lighthouse)
2. Identify the actual bottleneck (don't guess!)
3. Pick the lowest-cost lever that addresses it
4. Implement the change
5. Re-measure — both lab and field
6. Set a budget so it doesn't regress
7. Move on
```

### Key tools
- **Chrome DevTools — Performance** — main-thread flame chart, long tasks, frames
- **Performance Insights** — newer, friendlier UI; INP attribution
- **Network panel** — waterfall, request timing, sizes
- **Lighthouse** — opinionated audits + budgets
- **WebPageTest** — best-in-class waterfall + filmstrip
- **`web-vitals`** — RUM library
- **Sentry / Datadog RUM / SpeedCurve / Calibre / DebugBear** — production monitoring

---

## Common Anti-Patterns

### 1. Memoizing everything
`useMemo`/`useCallback`/`memo` everywhere — adds CPU and complexity, often without measurable wins. Memoize **measured hot paths**.

### 2. Loading everything upfront "in case"
A single 800KB bundle with admin code, payment code, settings code, all loaded on every page. Code-split.

### 3. Polling instead of subscribing
`setInterval(fetchData, 1000)` everywhere. Use SSE, WebSockets, or push when available.

### 4. Unbounded caches
A `Map` that grows forever. Use LRU + TTL.

### 5. Synchronous `localStorage` in hot paths
Blocks the main thread; small but adds up.

### 6. `JSON.parse` / `JSON.stringify` of huge payloads on the main thread
Move to a Worker for anything > a few hundred KB.

### 7. Lazy-loading the LCP image
The single most-cited "but I added perf optimizations" mistake. Use `fetchpriority="high"`, never `loading="lazy"` on hero.

### 8. Animating layout properties
`top`/`left`/`width`/`height` animations cost a layout per frame. Use `transform`/`opacity`.

### 9. Long lists without virtualization
1000 items in the DOM with event handlers each → memory + render cost. Virtualize.

### 10. Index-as-key in dynamic lists
`key={index}` on a sortable/filterable list breaks React reconciliation. Use stable IDs.

### 11. Preloading everything
Multiple `<link rel="preload">` competing for bandwidth. Each one slows the others.

### 12. Disabling bfcache accidentally
`unload` listener, `Cache-Control: no-store`, certain WebSocket usage — kills instant back-navigation.

### 13. Bundling node modules into client
Server-only code (e.g., DB drivers) shipped to the browser. Audit your bundle.

### 14. Polyfilling for browsers you don't support
Shipping `core-js` to Chrome 120 for IE11 compatibility you don't actually need.

### 15. Optimizing without a budget
"We'll keep an eye on it" → it regresses next sprint. Budgets in CI are the only sustainable control.

---

## Quick Reference Checklist

### Bundle
- [ ] Routes are code-split
- [ ] Heavy components dynamically imported
- [ ] Tree shaking works (named imports, ES modules)
- [ ] Vendor split for long-term caching
- [ ] No duplicate dependencies
- [ ] Modern build for modern browsers
- [ ] Bundle analysis in CI; size budgets enforced

### Caching
- [ ] Static assets `Cache-Control: immutable, max-age=31536000`
- [ ] HTML short TTL + stale-while-revalidate
- [ ] CDN configured with sensible defaults
- [ ] Service Worker (if PWA) versioned, with kill-switch
- [ ] Server-state library (TanStack Query/SWR) handling API caching
- [ ] LRU + TTL on in-memory caches
- [ ] No `Cache-Control: no-store` on bfcacheable pages

### Rendering (React)
- [ ] State colocated to lowest necessary level
- [ ] Context split by concern; consumers subscribe narrowly
- [ ] `React.memo` on measured hot components
- [ ] Stable keys (not index) on dynamic lists
- [ ] `useTransition` / `useDeferredValue` on expensive interactions
- [ ] No `useEffect` for derived state (compute in render)
- [ ] Profiler used to find actual hotspots

### Lists
- [ ] Lists > 200 items virtualized
- [ ] Infinite scroll uses Intersection Observer
- [ ] Cursor-based pagination on the API
- [ ] Scroll restoration handled on back-navigation

### Network
- [ ] HTTP/2 or /3 enabled
- [ ] Brotli/gzip on text resources
- [ ] DNS prefetch / preconnect to known origins
- [ ] LCP image preloaded with `fetchpriority="high"`
- [ ] Critical fonts preloaded
- [ ] No render-blocking JS in `<head>`

### Assets
- [ ] AVIF/WebP with fallbacks
- [ ] Width/height on all `<img>` and `<video>`
- [ ] WOFF2 fonts; `font-display: swap` or `optional`
- [ ] Subsetted fonts where appropriate
- [ ] Image CDN handling format negotiation

### Runtime
- [ ] Long tasks broken up with `scheduler.yield` or chunking
- [ ] Heavy CPU work in Web Workers
- [ ] Scroll/touch listeners passive
- [ ] No layout thrash (read-write batching)
- [ ] Animations use `transform`/`opacity`

### Process
- [ ] Performance budgets in CI
- [ ] Field RUM collected and segmented
- [ ] Regression alerts on releases
- [ ] Quarterly bundle and dep audits

---

## Further Reading

### Foundational
- **[web.dev — Learn Performance](https://web.dev/learn/performance)**
- **[web.dev — Fast load times](https://web.dev/explore/fast)**
- **[patterns.dev](https://patterns.dev/)** — performance patterns with code
- **Addy Osmani — *Image Optimization*, *JavaScript Loading Priorities*** — articles on web.dev

### Books
- **Ilya Grigorik — *High Performance Browser Networking*** (free online)
- **Steve Souders — *High Performance Web Sites*** (foundational, dated)
- **Tom Barker — *High Performance Images***
- **Smashing Magazine — *Front-End Performance Checklist*** (annual; the most current practical reference)

### React-Specific
- **[React docs — Optimizing Performance](https://react.dev/reference/react/memo)**
- **Kent C. Dodds — *Epic React*** — practical patterns
- **Dan Abramov — Overreacted blog** — deep dives on rendering
- **Mark Erikson — *Practical Redux*** — when state actually warrants a store

### Caching
- **[MDN — HTTP caching](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)**
- **[Workbox docs](https://developer.chrome.com/docs/workbox/)**
- **[Jake Archibald — *Caching best practices*](https://jakearchibald.com/2016/caching-best-practices/)**

### Tools
- **Chrome DevTools — Performance, Performance Insights, Memory, Network**
- **WebPageTest** — `webpagetest.org`
- **Lighthouse CI** — budgets and assertions per PR
- **`web-vitals`** library — RUM
- **`size-limit`**, **`bundlewatch`** — bundle budgets in CI

### People to follow
- Addy Osmani, Jeremy Wagner, Barry Pollard, Annie Sullivan, Harry Roberts (CSS Wizardry), Dan Abramov, Ryan Florence, Tanner Linsley (TanStack), Jake Archibald, Surma

---

> **Closing thought:** optimization is not a checklist; it's a discipline. The edge that separates fast apps from slow ones isn't memorizing tricks — it's **profiling first, choosing the smallest effective change, measuring the result, and writing a budget so it doesn't regress.** Most performance debt accumulates slowly: one un-memoized component, one third-party tag, one un-virtualized list. I've never seen a fast app degrade overnight; it's always one PR at a time. The way to keep apps fast is the same as the way to keep code clean — small, continuous, measured discipline. Do that, and your users will notice without ever knowing why.
