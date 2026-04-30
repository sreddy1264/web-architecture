# Core Web Vitals (CWV)

> A practical engineering guide to measuring and optimizing real-user performance.

---

## Table of Contents

1. [What are Core Web Vitals?](#what-are-core-web-vitals)
2. [Why are They Important?](#why-are-they-important)
3. [The Three Core Metrics](#the-three-core-metrics)
4. [Other Web Vitals (Diagnostic Metrics)](#other-web-vitals-diagnostic-metrics)
5. [How They're Measured: Lab vs. Field](#how-theyre-measured-lab-vs-field)
6. [How to Implement & Optimize](#how-to-implement--optimize)
7. [Core Web Vitals in React](#core-web-vitals-in-react)
8. [Precautions and Common Pitfalls](#precautions-and-common-pitfalls)
9. [Measurement & Tooling](#measurement--tooling)
10. [Performance Budgets & Process](#performance-budgets--process)
11. [Quick Reference Checklist](#quick-reference-checklist)
12. [Further Reading](#further-reading)

---

## What are Core Web Vitals?

**Core Web Vitals (CWV)** are a set of standardized, user-centric performance metrics defined by Google to measure **real-world page experience** — how fast a page loads, how quickly it responds to input, and how stable it is visually.

They are part of a larger family called **Web Vitals**, but the "Core" set is what Google uses as a **ranking signal** and what most teams should track first.

As of 2026, the three Core Web Vitals are:

| Metric | Measures | Good threshold |
|---|---|---|
| **LCP** — Largest Contentful Paint | Loading | ≤ **2.5s** |
| **INP** — Interaction to Next Paint | Interactivity | ≤ **200ms** |
| **CLS** — Cumulative Layout Shift | Visual stability | ≤ **0.1** |

> **Important historical note:** **INP replaced FID** (First Input Delay) as a Core Web Vital on **March 12, 2024**. INP is a much stricter, more honest measure of responsiveness — it captures the *worst* interaction latency across the entire page lifecycle, not just the first.

The thresholds are based on the **75th percentile** of page loads across mobile and desktop — meaning your slowest 25% of users still need to be within "good" range to pass.

---

## Why are They Important?

### 1. They Reflect Actual User Experience
Unlike older metrics (load time, DOMContentLoaded), CWV are designed around **what users perceive**. LCP measures when the page *looks* loaded; INP measures whether the page *feels* responsive; CLS measures whether content jumps around as you read.

### 2. They Are a Google Ranking Signal
Since 2021, page experience (which includes CWV) is part of Google's ranking algorithm. It's a tiebreaker rather than a primary factor — but at scale, tiebreakers matter. Pages that pass CWV thresholds get a small but meaningful ranking advantage.

### 3. They Move Business Metrics
Performance has direct, measured commercial impact:

- **Vodafone**: 31% improvement in LCP → **+8% sales**
- **NDTV**: 55% reduction in CLS → **+50% bounce rate improvement**
- **Rakuten 24**: better CWV → **+53.4% revenue per visitor**
- **Amazon (classic)**: 100ms latency cost ~1% of sales
- **Google**: 500ms slower SERP cost ~20% of traffic

Every 100ms shaved off LCP or INP shows up in conversion rates, retention, and revenue.

### 4. They Predict Frustration and Churn
Slow, janky, shifting interfaces erode trust. Mobile users on flaky networks are the most sensitive. CWV is one of the few engineering metrics that **directly predicts user behavior**.

### 5. They Are Vendor-Neutral and Field-Measurable
You can collect CWV from your own users with the [`web-vitals`](https://github.com/GoogleChrome/web-vitals) JS library and pipe them anywhere — Datadog, BigQuery, your own warehouse. This is rare for UX metrics.

---

## The Three Core Metrics

### 1. LCP — Largest Contentful Paint

**What it measures:** the render time of the largest visible content element (image, video poster, or block-level text) within the viewport.

**Thresholds:**
| Good | Needs Improvement | Poor |
|---|---|---|
| ≤ 2.5s | ≤ 4.0s | > 4.0s |

**Why it matters:** LCP is a proxy for "when does the page feel loaded?" The user is staring at the screen waiting for the hero to appear.

**Common LCP elements:**
- Hero image (`<img>`)
- Background image (via CSS) — often the LCP element on landing pages
- `<video>` poster
- The first big block of text (`<h1>` or paragraph)

**The four sub-phases of LCP** (this is the framework to debug it):

| Phase | What's happening | Typical % of LCP |
|---|---|---|
| **TTFB** (Time to First Byte) | Server processing + network | 10–40% |
| **Resource Load Delay** | Time between TTFB and when the LCP resource starts downloading | 0–40% (usually the biggest fixable chunk) |
| **Resource Load Time** | The actual download of the LCP resource | 10–40% |
| **Render Delay** | Time after the LCP resource is loaded until it paints | 10–30% |

> **In practice:** most LCP problems aren't slow servers or huge images — they're **load delay**. The browser doesn't even *start* fetching the LCP image until it's parsed deep into the HTML, after CSS, after fonts, after some JS. Fix that first.

### 2. INP — Interaction to Next Paint

**What it measures:** the latency of all interactions (clicks, taps, key presses) on a page, reporting (roughly) the worst one. Specifically: from input → next visual update painted.

**Thresholds:**
| Good | Needs Improvement | Poor |
|---|---|---|
| ≤ 200ms | ≤ 500ms | > 500ms |

**Why it's stricter than FID was:**
- FID only measured the *first* input's delay
- FID only measured input *delay*, not full processing + paint
- INP measures the **whole interaction**: input delay + processing time + presentation delay
- INP captures the **worst** interaction across the page lifecycle (with some smoothing for outliers)

**The three phases of an interaction:**

```
[input]──┐
         │  Input delay (main thread busy?)
         ▼
   [event handlers run]
         │  Processing time (your JS)
         ▼
   [rendering]
         │  Presentation delay (layout, paint, compositor)
         ▼
   [next frame painted]
```

INP failures usually come from:
- Long tasks blocking the main thread (third-party scripts, hydration, heavy event handlers)
- Synchronous layout thrash inside handlers
- Large React re-renders triggered by every click/keystroke
- `setState` cascades that fan out across the tree

### 3. CLS — Cumulative Layout Shift

**What it measures:** unexpected visual movement of content. Computed as `impact fraction × distance fraction` for each shift, summed within **session windows** (max 5s wide, 1s gap), reporting the **worst window**, not the lifetime sum.

**Thresholds:**
| Good | Needs Improvement | Poor |
|---|---|---|
| ≤ 0.1 | ≤ 0.25 | > 0.25 |

**Why it matters:** layout shift causes mis-clicks ("I tapped Buy! No, Cancel!"), reading interruption, and a feeling of cheapness.

**Top causes:**
- Images/videos without explicit `width`/`height` (or `aspect-ratio`)
- Web fonts swapping in (FOUT/FOIT) and shifting text dimensions
- Ads, embeds, iframes with unreserved space
- Banners injected at the top of the page
- Animations of properties that affect layout (`top`, `left`, `width`) instead of `transform`

> **Worth knowing:** CLS is the easiest CWV to fix. If you're failing it, you have low-hanging fruit. If you're passing it, prevent regressions with lint rules and tests — once shipped, CLS regressions are easy to introduce by accident (a third-party tag, a new ad slot).

---

## Other Web Vitals (Diagnostic Metrics)

These aren't "Core" but you'll need them to debug.

| Metric | What | Good |
|---|---|---|
| **TTFB** — Time to First Byte | Server response speed | ≤ 800ms |
| **FCP** — First Contentful Paint | First *anything* painted | ≤ 1.8s |
| **TBT** — Total Blocking Time | Sum of long-task blocking time (lab proxy for INP) | ≤ 200ms |
| **SI** — Speed Index | How quickly content visually populates | varies |

> Lighthouse "Performance" score is a weighted average of LCP, TBT, CLS, FCP, and SI — useful in lab, but **the field metrics (LCP/INP/CLS) are what Google uses for ranking**. Don't optimize for the Lighthouse score in isolation.

---

## How They're Measured: Lab vs. Field

### Lab Data
Synthetic, controlled tests (Lighthouse, WebPageTest, PSI's "Performance" tab).
- ✅ Reproducible, easy to debug
- ❌ One device, one network, one moment — doesn't reflect real users
- ❌ **Cannot measure INP accurately** — synthetic tools have no real interactions; they show TBT instead

### Field Data (RUM — Real User Monitoring)
Anonymized metrics collected from actual visits, aggregated at the 75th percentile.
- ✅ Reflects your real user mix (devices, networks, geographies)
- ✅ The data Google uses for ranking
- Sources:
  - **CrUX (Chrome User Experience Report)** — public, free, 28-day rolling, only Chrome users who opted in. Powers PSI, Search Console, BigQuery `chrome-ux-report`.
  - **Your own RUM** — using `web-vitals` JS library, sent to your analytics. Gives per-page, per-segment data CrUX can't provide.

**Recommendation:** use both. CrUX is your "ranking signal" view; your own RUM is your "debugging" view (per route, per device class, per release).

---

## How to Implement & Optimize

### Optimizing LCP

#### 1. Identify the LCP element
```js
new PerformanceObserver((list) => {
  const entries = list.getEntries();
  console.log('LCP element:', entries[entries.length - 1].element);
}).observe({ type: 'largest-contentful-paint', buffered: true });
```
Or use Chrome DevTools → Performance → see the "LCP" marker.

#### 2. Reduce TTFB
- Server-render or pre-render — avoid waiting on client JS for first paint
- Cache HTML at the CDN edge (with appropriate `Cache-Control` / stale-while-revalidate)
- Use a fast origin (or move logic to edge functions)
- HTTP/2 or HTTP/3, Brotli compression
- Avoid expensive synchronous DB queries in the request path

#### 3. Eliminate Resource Load Delay (the #1 lever)
Make sure the LCP resource is discovered and prioritized as early as possible:

```html
<!-- Preload the hero image so it starts downloading immediately -->
<link
  rel="preload"
  as="image"
  href="/hero.webp"
  fetchpriority="high"
  imagesrcset="/hero-800.webp 800w, /hero-1600.webp 1600w"
  imagesizes="100vw"
/>

<!-- Or, on the img itself -->
<img src="/hero.webp" alt="..." fetchpriority="high" />
```

- **Don't lazy-load the LCP image.** `loading="lazy"` on the hero is a self-inflicted wound.
- **Don't hide LCP behind JS** — server-render the hero markup.
- Use `fetchpriority="high"` (Chrome 102+, Safari 17.2+).
- Avoid CSS background-images for the LCP — they're discovered late. Use `<img>` instead, or preload them.

#### 4. Reduce Resource Load Time
- Modern formats: **AVIF** (~50% smaller than JPEG), **WebP** (~30%) with JPEG fallback
- Responsive images: `srcset` + `sizes` so mobile doesn't fetch desktop assets
- Compress aggressively (most hero images can be 60–80% smaller with no perceptible loss)
- Use an image CDN (Cloudinary, Imgix, Vercel/Next Image) with automatic format negotiation
- Serve from a CDN close to your users
- Preconnect to image origins: `<link rel="preconnect" href="https://cdn.example.com">`

#### 5. Reduce Render Delay
- Inline critical CSS, defer the rest
- Avoid render-blocking JS in `<head>` — use `defer` or `async`
- Use `font-display: swap` or `optional` so text doesn't wait on fonts
- Preload critical fonts: `<link rel="preload" as="font" type="font/woff2" crossorigin>`

### Optimizing INP

#### 1. Find the slow interactions
Use the **`web-vitals`** library with attribution:
```js
import { onINP } from 'web-vitals/attribution';
onINP((metric) => {
  console.log(metric.attribution.eventTarget,
              metric.attribution.eventType,
              metric.attribution.loadState,
              metric.value);
});
```
Or Chrome DevTools → Performance → Interactions track.

#### 2. Break up long tasks
Anything > 50ms blocks the main thread.

```js
// Yield to the browser between chunks
async function processLargeList(items) {
  for (const item of items) {
    doWork(item);
    if (navigator.scheduling?.isInputPending()) {
      await new Promise(r => setTimeout(r));
    }
  }
}

// Or use the modern scheduler API
await scheduler.yield();
```

Tools:
- `scheduler.postTask()` — run lower-priority work
- `scheduler.yield()` — explicitly yield mid-task
- `requestIdleCallback()` — defer until idle

#### 3. Move expensive work off the main thread
- **Web Workers** for parsing, sorting, crypto, image processing
- **OffscreenCanvas** for heavy rendering
- Server-side computation when possible

#### 4. Reduce JS payload
- Code-split by route
- Lazy-load components below the fold (`React.lazy`, dynamic `import()`)
- Tree-shake aggressively — audit with `source-map-explorer` or `bundle-analyzer`
- Remove unused polyfills (ship modern JS to modern browsers via `<script type="module">`)
- Replace heavy deps (moment.js → date-fns, lodash → individual functions)

#### 5. Optimize event handlers
- Debounce/throttle input handlers
- Avoid `setState` in mousemove/scroll without throttling
- Don't do synchronous layout-reading work in handlers (`getBoundingClientRect()` after a write)

#### 6. Optimize hydration (SSR/SSG)
- Stream HTML so users see content before hydration completes
- Use **selective/progressive hydration** (React 18+)
- Use **partial hydration / islands** (Astro, Qwik) for content-heavy sites
- Replace full-page hydration with **resumability** (Qwik) for the most extreme cases

#### 7. Tame third-party scripts
Third-party tags (analytics, A/B testing, chat, ads) are a leading cause of INP failures.
- Load with `async` or `defer`, never blocking
- Lazy-load chat widgets on user intent (scroll, idle, click)
- Use a tag manager with strict policies — *don't* let marketing add anything they want
- Audit periodically; remove dead tags

### Optimizing CLS

#### 1. Reserve space for media
```html
<!-- Always set width and height — browser computes aspect ratio -->
<img src="/photo.jpg" alt="..." width="1600" height="900" />

<!-- For responsive images using CSS -->
<style>
  img { aspect-ratio: 16 / 9; width: 100%; height: auto; }
</style>
```

#### 2. Reserve space for ads, embeds, iframes
```css
.ad-slot {
  min-height: 250px; /* Reserve before fetch */
}
```

#### 3. Avoid inserting content above existing content
Banners, cookie notices, "new version available" toasts — show them at the bottom or as overlays, not pushed into the document flow.

#### 4. Manage fonts
- `font-display: optional` — eliminates shift entirely (uses fallback if font isn't ready in 100ms)
- `font-display: swap` + matching fallback metrics (`size-adjust`, `ascent-override`) — modern browsers can match fallbacks to web fonts almost perfectly
- Preload critical fonts

```css
@font-face {
  font-family: 'Inter';
  src: url('/inter.woff2') format('woff2');
  font-display: swap;
  size-adjust: 100.06%;
  ascent-override: 90%;
}
```

#### 5. Animate with `transform` and `opacity`, not layout
```css
/* ❌ causes layout, paints, and CLS */
.menu { transition: top 200ms; top: -100px; }

/* ✅ no layout, GPU-accelerated, no CLS */
.menu { transition: transform 200ms; transform: translateY(-100px); }
```

#### 6. Use `content-visibility: auto` carefully
It's a powerful rendering optimization but can cause shifts when off-screen content enters the viewport. Pair with `contain-intrinsic-size` to reserve space.

---

## Core Web Vitals in React

Since this is a React codebase, specific patterns matter.

### Server-render or pre-render
A pure CSR React app has terrible LCP — the browser downloads JS, hydrates, fetches data, then paints. Use:
- **Next.js** (App Router with RSC + streaming)
- **Remix / React Router 7**
- **Astro** (with React islands for content sites)

### Use Next.js `<Image>` (or equivalent)
It handles `srcset`, `sizes`, lazy loading, modern formats, and `width`/`height` automatically.

```jsx
import Image from 'next/image';

<Image
  src="/hero.jpg"
  alt="..."
  width={1600} height={900}
  priority   // marks LCP image — disables lazy load, sets fetchpriority="high"
/>
```

### Optimize hydration cost
- Server Components for non-interactive UI (zero JS shipped)
- `<Suspense>` boundaries to stream and parallelize
- `next/dynamic` with `ssr: false` for heavy client-only widgets (charts, editors)
- Avoid putting heavy logic in top-level layouts that hydrate on every navigation

### Tame re-renders (helps INP)
- `React.memo` on expensive children that receive stable props
- `useMemo` / `useCallback` only when there's a measured win — they have their own cost
- Co-locate state (lift only when needed) to avoid wide re-renders
- Use **transition APIs** for non-urgent updates:

```jsx
const [isPending, startTransition] = useTransition();
startTransition(() => setHeavyState(next));
```

This marks the update as low priority — the browser can interrupt it for user input, dramatically improving INP on filter/search inputs.

### Use `useDeferredValue` for expensive derived state
```jsx
const deferredQuery = useDeferredValue(query);
const results = useMemo(() => search(deferredQuery), [deferredQuery]);
```

### Code-split by route, lazy-load below the fold
```jsx
const HeavyChart = React.lazy(() => import('./HeavyChart'));
<Suspense fallback={<Skeleton/>}><HeavyChart/></Suspense>
```

### Measure in production
```jsx
// app/layout.tsx (Next.js)
import { useReportWebVitals } from 'next/web-vitals';

useReportWebVitals((metric) => {
  // send to your analytics
  navigator.sendBeacon('/rum', JSON.stringify(metric));
});
```

---

## Precautions and Common Pitfalls

### 1. Don't optimize lab scores while field metrics suffer
Lighthouse on your laptop with a fast connection and no extensions ≠ a real user on a mid-tier Android over 4G. **Always validate with field data (CrUX or your own RUM).** A site can have a 100 Lighthouse score and fail CWV in the field.

### 2. Don't lazy-load the LCP image
`loading="lazy"` defers loading until near-viewport — but the LCP element *is* in the viewport. This is a top cause of bad LCP. Use `fetchpriority="high"` instead.

### 3. Don't use `loading="lazy"` on `<iframe>` for above-the-fold maps/videos
Same problem.

### 4. Don't preload everything
Preload is a hammer; using it on too many resources delays the actually-critical ones. Reserve preload for: hero LCP image, critical fonts, and (rarely) hero JSON data.

### 5. Don't ignore TTFB
You can't have a 2.5s LCP if your TTFB is 1.5s — you're starting the race two-thirds late. Slow servers cap your achievable LCP.

### 6. Don't trust pre-aggregated CrUX without segmentation
75th-percentile CrUX hides issues. A page can pass overall while utterly failing on Android 4G. Segment by device, network, country, and route in your own RUM.

### 7. Don't measure INP in Lighthouse and call it done
Lighthouse reports **TBT**, not INP. TBT is a *lab proxy* — correlation is moderate. Real INP requires real interactions; collect it from RUM.

### 8. Don't optimize for the average user
The 75th percentile target means your slow users matter most. The average might be 1.8s LCP while p75 is 4.5s — and Google sees the p75. Look at distributions, not averages.

### 9. Don't count CLS only on initial load
CLS is measured for the **entire page lifecycle**, including post-load shifts from infinite scrollers, lazy content, ads rotating, modals appearing. Test interactions, not just page load.

### 10. Don't blame the framework before measuring
React/Next/Vue can all hit "Good" CWV. Most performance problems are app-level: heavy hero images, render-blocking third parties, hydration of unnecessarily-interactive components, oversized JS bundles. Profile first.

### 11. Don't ignore long-tail third-party tags
Tag managers accumulate cruft: an A/B test from 2022, a chat widget marketing forgot about, a heatmap on a page that doesn't need it. Audit `_dataLayer` quarterly.

### 12. Beware Single Page Apps and CWV reporting
Each route change is **not** a new "page" by browser standards — it's the same document. Soft navigations have new metrics being standardized but support is partial. Use the latest `web-vitals` library which supports soft-navigation INP/LCP-style metrics.

### 13. Don't micro-optimize JS when the LCP image is 2MB
80% of LCP wins come from images, fonts, and TTFB. Optimize there before chasing JS milliseconds.

### 14. Don't forget about the back/forward cache (bfcache)
Chrome and Safari restore pages from bfcache instantly (effectively zero LCP/INP). Things that disable bfcache: `unload` event listeners, `Cache-Control: no-store`, certain WebSocket usage. Check `chrome://discards` or Lighthouse's bfcache audit. Free wins.

### 15. Don't ship without a regression alert
Performance regressions are silent. A new analytics tag, an unminified dep, a forgotten `console.log` in a render path — none break tests, all break CWV. Alert on field-metric regressions.

---

## Measurement & Tooling

### Field (RUM)
- **[Google Search Console — Core Web Vitals report](https://search.google.com/search-console)** — your CrUX-based view per origin
- **[PageSpeed Insights](https://pagespeed.web.dev/)** — CrUX field data + Lighthouse lab data side-by-side
- **[CrUX Dashboard (Looker Studio)](https://g.co/chromeuxdash)** — historical trends from public CrUX
- **[CrUX BigQuery dataset](https://developer.chrome.com/docs/crux/bigquery)** — query-level access to public field data
- **`web-vitals` npm package** — collect your own RUM
- **Vendors**: Datadog RUM, New Relic Browser, Sentry Performance, SpeedCurve, Calibre, DebugBear, Akamai mPulse, Cloudflare Web Analytics

### Lab
- **Lighthouse** (DevTools, CLI, CI)
- **WebPageTest** — best-in-class waterfall analysis
- **Chrome DevTools Performance panel** — flame charts, Long Tasks, INP attribution
- **Performance Insights panel** (Chrome) — newer, friendlier perf UI

### Synthetic monitoring
- **Lighthouse CI** — assertions and baselines per PR
- **SpeedCurve / Calibre / DebugBear** — scheduled runs, budgets, alerts
- **Sitespeed.io** — open-source, self-hostable

### Profiling tooling
- **Chrome DevTools** — Performance, Performance Insights, Coverage, Network
- **`react-scan`** / **React DevTools Profiler** — re-render sources
- **`source-map-explorer`** / **`webpack-bundle-analyzer`** — what's in your JS

---

## Performance Budgets & Process

Performance is an organizational practice, not a one-time fix.

### 1. Set explicit budgets
Per-route budgets that block CI on regression:

```json
// lighthouserc / budget.json (example)
{
  "resourceSizes": [
    { "resourceType": "script", "budget": 170 },
    { "resourceType": "image", "budget": 250 },
    { "resourceType": "total", "budget": 500 }
  ],
  "timings": [
    { "metric": "largest-contentful-paint", "budget": 2500 },
    { "metric": "cumulative-layout-shift", "budget": 0.1 },
    { "metric": "total-blocking-time", "budget": 200 }
  ]
}
```

### 2. Wire it into CI
- Lighthouse CI on every PR, asserting against budgets
- Block merges on regression beyond a threshold
- Track bundle size diffs in the PR (e.g., `size-limit`, Bundlephobia, `bundlewatch`)

### 3. Monitor field metrics on every release
- Tag RUM events with release/commit
- Alert on p75 regressions per route within 24–48h of deploy
- Investigate every regression before it ships to 100%

### 4. Make performance a definition-of-done item
- New components have a Storybook story + perf check
- Designs show with realistic content and image sizes
- "Does this add a third-party script?" is a PR review question

### 5. Track the right metrics for your team
- Engineering team: CWV at p75 per route
- Product: conversion lift per CWV bucket
- Leadership: % of sessions in "Good" CWV across the funnel

---

## Quick Reference Checklist

### LCP
- [ ] LCP element identified and server-rendered (not appearing post-hydration)
- [ ] Hero image has `fetchpriority="high"` or is preloaded
- [ ] Hero image is **not** `loading="lazy"`
- [ ] Modern image format (AVIF/WebP) with responsive `srcset`
- [ ] Critical fonts preloaded with `crossorigin`
- [ ] `font-display: swap` or `optional`
- [ ] Render-blocking JS minimized; non-critical JS `defer`/`async`
- [ ] Critical CSS inlined; rest deferred
- [ ] CDN-cached HTML where possible (TTFB ≤ 800ms)
- [ ] Preconnect to image/font origins

### INP
- [ ] Long tasks (> 50ms) identified and broken up
- [ ] Heavy work moved to Web Workers where possible
- [ ] React `useTransition` / `useDeferredValue` used for expensive updates
- [ ] Third-party scripts audited; chat/widgets lazy-loaded on intent
- [ ] Unused JS eliminated (Coverage tab clean)
- [ ] Code split by route, components lazy-loaded below the fold
- [ ] Hydration scoped (RSC, islands, or selective)

### CLS
- [ ] All `<img>` and `<video>` have `width`/`height` or `aspect-ratio`
- [ ] Ad slots, embeds, iframes have reserved space
- [ ] No content injected above existing content (banners go bottom or overlay)
- [ ] Animations use `transform`/`opacity`, not layout properties
- [ ] Web fonts: `font-display` set, fallback metrics tuned

### Process
- [ ] Field RUM collecting LCP/INP/CLS, segmented by route + device
- [ ] CrUX / Search Console reviewed monthly
- [ ] Performance budgets enforced in CI
- [ ] Regression alerts wired to release tags
- [ ] bfcache not disabled (no `unload`, no `Cache-Control: no-store`)

---

## Further Reading

- **[web.dev — Learn Performance](https://web.dev/learn/performance)**
- **[web.dev — Core Web Vitals](https://web.dev/articles/vitals)**
- **[web-vitals JS library](https://github.com/GoogleChrome/web-vitals)** — official RUM collector
- **[Chrome DevTools Performance docs](https://developer.chrome.com/docs/devtools/performance)**
- **[INP deep dive](https://web.dev/articles/inp)** by Jeremy Wagner
- **[Optimize LCP](https://web.dev/articles/optimize-lcp)** — the four sub-phases framework
- **[Optimize CLS](https://web.dev/articles/optimize-cls)**
- **[CrUX Dashboard](https://g.co/chromeuxdash)** — public field data
- **[Smashing Magazine — A Guide to Core Web Vitals](https://www.smashingmagazine.com/2021/04/complete-guide-core-web-vitals/)**
- **Books**: *High Performance Browser Networking* (Ilya Grigorik), *Web Performance in Action* (Jeremy Wagner)
- **People to follow**: Addy Osmani, Jeremy Wagner, Barry Pollard, Annie Sullivan, Harry Roberts (CSS Wizardry)

---

> **Closing thought:** Core Web Vitals aren't a Google checklist — they're a **quantification of frustration**. LCP measures waiting, INP measures sluggishness, CLS measures jank. The shift I want every team I work with to make is to stop optimizing for the score and start optimizing for *the moment a real user hits your page on a slow phone in a parking lot*. Once you internalize that frame, the rest of this document becomes obvious — it's just the cheapest ways to remove that user's pain. Hit 75% green in CrUX and your site won't just rank better; it will feel faster, and your users will stay longer without ever knowing why.
