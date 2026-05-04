# From URL to Pixels: How Browsers Render Web Pages

*Last reviewed: 2026-05*

> A practical walkthrough of everything that happens between pressing Enter and seeing pixels — the network stack, the rendering pipeline, the critical rendering path, reflow & repaint, and the modern composited compositor.

---

## Table of Contents

1. [The Big Picture](#the-big-picture)
2. [Stage 1 — Address Bar Input & URL Parsing](#stage-1--address-bar-input--url-parsing)
3. [Stage 2 — DNS Resolution](#stage-2--dns-resolution)
4. [Stage 3 — TCP Connection](#stage-3--tcp-connection)
5. [Stage 4 — TLS Handshake](#stage-4--tls-handshake)
6. [Stage 5 — HTTP Request & Response](#stage-5--http-request--response)
7. [Stage 6 — The Critical Rendering Path (CRP)](#stage-6--the-critical-rendering-path-crp)
8. [Stage 7 — JavaScript Execution Model](#stage-7--javascript-execution-model)
9. [The Browser Architecture](#the-browser-architecture)
10. [Reflow (Layout) and Repaint](#reflow-layout-and-repaint)
11. [Compositing — The GPU Pipeline](#compositing--the-gpu-pipeline)
12. [Frame Lifecycle: How 60fps Actually Works](#frame-lifecycle-how-60fps-actually-works)
13. [Browser Caching](#browser-caching)
14. [Common Performance Pitfalls](#common-performance-pitfalls)
15. [Mental Models](#mental-models)
16. [Further Reading](#further-reading)

---

## The Big Picture

When you type `https://example.com` and press Enter, dozens of subsystems collaborate across multiple machines, networks, processes, and threads to produce pixels on your screen — usually in under a second.

```
┌─────────────────────────────────────────────────────────────────────┐
│                  THE END-TO-END JOURNEY                             │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  Address bar                                                        │
│       │                                                             │
│       ▼                                                             │
│  [URL parsing]                                                      │
│       │                                                             │
│       ▼                                                             │
│  [DNS lookup]──► IP address                                         │
│       │                                                             │
│       ▼                                                             │
│  [TCP handshake]   (or QUIC for HTTP/3)                             │
│       │                                                             │
│       ▼                                                             │
│  [TLS handshake]                                                    │
│       │                                                             │
│       ▼                                                             │
│  [HTTP request]──► Server                                           │
│       ▲                                                             │
│       │                                                             │
│  [HTTP response] ◄── HTML bytes                                     │
│       │                                                             │
│       ▼                                                             │
│  ┌─────────────────────────────────────────────────────────────┐    │
│  │                  RENDERING PIPELINE                         │    │
│  │                                                             │    │
│  │  Bytes ─► HTML Parser ─► DOM                                │    │
│  │  Bytes ─► CSS Parser  ─► CSSOM                              │    │
│  │                              │                              │    │
│  │  DOM + CSSOM ─► Render Tree ─► Layout ─► Paint ─► Composite │    │
│  │                                                             │    │
│  │  JS engine runs concurrently, can mutate DOM/CSSOM,         │    │
│  │  triggering reflow/repaint.                                 │    │
│  └─────────────────────────────────────────────────────────────┘    │
│       │                                                             │
│       ▼                                                             │
│  Pixels on screen                                                   │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

Each stage is worth understanding because **performance bugs hide in stage transitions** — a slow DNS, a TLS misconfiguration, a render-blocking script, a forced layout from a JS query — these are the boundaries where things break.

---

## Stage 1 — Address Bar Input & URL Parsing

Before any network traffic, the browser:

1. **Decides if it's a URL or a search query.** `react docs` becomes a Google search; `react.dev` becomes a navigation.
2. **Checks for HSTS preload list** — for sites like google.com, the browser refuses HTTP and forces HTTPS *before* any DNS happens. This protects against SSL stripping.
3. **Checks the bfcache (back/forward cache).** If the URL was previously visited and the page is in bfcache, the browser **restores the entire page in milliseconds** — no network, no rendering. This is why "back" can feel instant.
4. **Service Worker check.** If a Service Worker is registered for this scope, it intercepts the navigation request and may respond from cache without touching the network.
5. **URL parsing.** The URL is split into `scheme`, `userinfo`, `host`, `port`, `path`, `query`, and `fragment` per RFC 3986. The browser also normalizes the URL (lowercasing the host, percent-encoding, etc.).

> **Worth knowing:** the bfcache is one of the most powerful (and easily-broken) browser features. Things that disable it: `unload` event listeners, `Cache-Control: no-store`, certain WebSocket usage. Check `chrome://discards` or Lighthouse's bfcache audit.

---

## Stage 2 — DNS Resolution

The browser needs an IP address. It walks a cache hierarchy:

```
┌──────────────────────────────────────────────────────────┐
│  1. Browser DNS cache         (in-process, seconds-mins) │
│  2. OS DNS cache              (system-wide)              │
│  3. Router DNS cache                                     │
│  4. ISP recursive resolver    (e.g., 8.8.8.8, 1.1.1.1)   │
│  5. Root DNS servers          (.)                        │
│  6. TLD DNS servers           (.com, .org, etc.)         │
│  7. Authoritative name server (the domain's own NS)      │
└──────────────────────────────────────────────────────────┘
```

The first hit returns. A cold lookup walks the full chain (10–200ms typical).

### Modern DNS
- **DoH (DNS over HTTPS)** and **DoT (DNS over TLS)** encrypt DNS queries — the browser may use these instead of plain UDP/53.
- **DNS prefetch hints** (`<link rel="dns-prefetch">`) start lookups for known origins early.
- **`preconnect`** goes further — DNS + TCP + TLS warm-up.

```html
<link rel="dns-prefetch" href="//cdn.example.com">
<link rel="preconnect" href="https://api.example.com" crossorigin>
```

### Why DNS matters for performance
A 200ms DNS lookup is 200ms of LCP you can't get back. CDNs like Cloudflare and Akamai run anycasted resolvers globally to keep this near-zero, but third-party origins (analytics, fonts, ads) often add cold lookups. **Every distinct origin on a page is a potential DNS round-trip.**

---

## Stage 3 — TCP Connection

For HTTP/1.1 and HTTP/2 (over TCP), the browser performs the **3-way handshake**:

```
Client                              Server
  │                                    │
  │ ──────── SYN (seq=x) ─────────►    │
  │                                    │
  │ ◄────── SYN-ACK (ack=x+1) ─────    │
  │                                    │
  │ ──────── ACK (ack=y+1) ────────►   │
  │                                    │
  │  Connection established            │
```

This costs **1 round-trip time (RTT)** before any data flows. On a 100ms RTT (e.g., transcontinental), that's 100ms before the request even leaves.

### HTTP/3 / QUIC: Eliminating the Handshake
HTTP/3 runs over **QUIC** (UDP-based). QUIC combines transport + crypto handshake, supports **0-RTT resumption** (returning visitors send the request *with* the handshake), and survives network changes (Wi-Fi → 4G) without reconnecting.

### Connection Reuse
- **HTTP/1.1 keep-alive** — reuse a TCP connection for multiple requests, but only serially (head-of-line blocking).
- **HTTP/2 multiplexing** — many parallel streams over one TCP connection. Eliminates the need for domain sharding (an HTTP/1 hack).
- **HTTP/3 multiplexing** — same as HTTP/2, but stream-independent — one slow packet doesn't block others.

> **Worth knowing:** in 2026, ~30% of web traffic is HTTP/3. If your origin doesn't speak it, you're leaving real performance on the table for users on lossy networks (mobile, satellite, distant geographies).

---

## Stage 4 — TLS Handshake

For HTTPS, after TCP, the browser performs a TLS handshake to establish encrypted communication.

### TLS 1.3 (the modern default)
1. **ClientHello** — supported ciphers, extensions, key share
2. **ServerHello** — chosen cipher, certificate, key share
3. **Finished** — handshake complete

That's **1-RTT**. Combined with TCP's 1-RTT, you've spent 2 RTTs before the first HTTP byte (~200ms on a 100ms link).

### TLS 1.3 0-RTT
For repeat connections, TLS 1.3 supports **0-RTT resumption** — the client sends data with the first handshake message. *Much* faster, but with replay-attack caveats — only safe for idempotent requests (GETs).

### Certificate validation
The server's cert must:
- Chain to a trusted root CA
- Match the hostname (SAN extension)
- Be unexpired
- Not be revoked (OCSP / CRL checks — modern browsers prefer OCSP stapling)

**Slow OCSP** is a classic source of 1-3s TTFB spikes; OCSP stapling fixes it.

### What's encrypted
TLS encrypts the HTTP payload + headers (in HTTP/2/3). It does **not** encrypt the destination IP, the SNI hostname (until ECH ships universally), or the timing/size of packets — relevant for privacy, not for correctness.

---

## Stage 5 — HTTP Request & Response

### Request
```
GET / HTTP/2
Host: example.com
User-Agent: Mozilla/5.0 ...
Accept: text/html,application/xhtml+xml...
Accept-Encoding: gzip, deflate, br
Cookie: session=abc123
If-None-Match: "v42"
```

### Response
```
HTTP/2 200 OK
Content-Type: text/html; charset=utf-8
Content-Encoding: br
Cache-Control: public, max-age=0, must-revalidate
ETag: "v43"
Server-Timing: db;dur=53, render;dur=120

<!doctype html>
<html>...
```

### TTFB (Time to First Byte)
The interval between the request leaving and the first byte of the response arriving. It includes:
- Network round-trip
- Server queue time
- Server processing (DB queries, template rendering)
- Network return time

**A 1-second TTFB caps your LCP at >1 second.** You can't optimize what comes after if the server is slow.

### Streaming
HTML responses are **streamed** — the browser starts parsing the first bytes while the rest are still arriving. This is why **server-rendered apps feel faster than CSR**: the browser can start building the DOM before the response completes.

### Compression
- **Brotli (`br`)** — modern, ~20% better than gzip
- **gzip** — universal fallback

Always serve compressed text (HTML, CSS, JS, JSON, SVG). Never compress already-compressed (images, video, fonts already in WOFF2).

---

## Stage 6 — The Critical Rendering Path (CRP)

The **Critical Rendering Path** is the sequence of steps the browser takes to convert HTML, CSS, and JS into pixels on screen. Optimizing CRP = optimizing first paint.

```
┌─────────────────────────────────────────────────────────────────────┐
│                   THE CRITICAL RENDERING PATH                       │
└─────────────────────────────────────────────────────────────────────┘

  HTML bytes                 CSS bytes
      │                         │
      ▼                         ▼
  [Tokenizer]              [Tokenizer]
      │                         │
      ▼                         ▼
  [Parser]                 [Parser]
      │                         │
      ▼                         ▼
  ┌─────┐                  ┌──────┐
  │ DOM │                  │CSSOM │
  └──┬──┘                  └──┬───┘
     │                        │
     └───────────┬────────────┘
                 │
                 ▼
          ┌──────────────┐
          │ Render Tree  │   (DOM nodes + computed styles,
          │              │    excludes display:none)
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │   Layout     │   (a.k.a. Reflow — positions & sizes)
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │    Paint     │   (raster pixels into layers)
          └──────┬───────┘
                 │
                 ▼
          ┌──────────────┐
          │  Composite   │   (assemble layers on GPU)
          └──────┬───────┘
                 │
                 ▼
              Pixels
```

### Step 1 — Build the DOM
The HTML parser tokenizes bytes, then constructs the **DOM** (Document Object Model) — a tree of nodes representing the document structure.

```html
<html>
  <head><title>Hi</title></head>
  <body><p>Hello</p></body>
</html>
```

Becomes:
```
Document
└── html
    ├── head
    │   └── title — "Hi"
    └── body
        └── p — "Hello"
```

The HTML parser is **resumable** and **streaming**: it builds the DOM incrementally as bytes arrive.

### Step 2 — Build the CSSOM
CSS bytes are parsed into the **CSSOM** (CSS Object Model), a tree mirroring the DOM with computed styles per node.

> **CSS is render-blocking.** The browser will not paint until CSSOM is built — otherwise the user would see unstyled content (FOUC). This is why **render-blocking CSS in the head is the single biggest first-paint risk** for many pages.

### Step 3 — Render Tree
The browser combines DOM + CSSOM into the **render tree** — only nodes that will be painted (`display:none` is excluded; `visibility:hidden` is included with empty layout).

### Step 4 — Layout (Reflow)
The browser computes the geometry of every render-tree node — position, size, dimensions. This is **layout** (also called **reflow**).

Layout is a global computation: a width change at the top of the page can cascade to every descendant. Browsers optimize aggressively, but layout is one of the most expensive things they do.

### Step 5 — Paint
The browser fills in pixels for each visible element — text, colors, images, borders, shadows. Paint produces **layers** (think Photoshop).

### Step 6 — Composite
The GPU composites layers together according to z-index, transforms, opacity, and produces the final framebuffer the OS displays.

### How CSS and JS block the CRP

| Resource | Default behavior | Blocks parsing? | Blocks rendering? |
|---|---|---|---|
| `<link rel="stylesheet">` in head | Render-blocking | No | **Yes** |
| `<script>` (sync) in head | Parser-blocking | **Yes** | Yes |
| `<script defer>` | Loads in parallel, runs after parse | No | No |
| `<script async>` | Loads in parallel, runs ASAP | Briefly when executing | Briefly |
| `<script type="module">` | Like `defer` by default | No | No |
| Inline `<script>` | Synchronous | **Yes** | Yes |

> **In practice:** the *most common* reason for slow LCP is a render-blocking stylesheet or sync script in `<head>`. Audit your `<head>` ruthlessly.

### Optimization tactics
- **Inline critical CSS** (the styles needed for above-the-fold) directly in `<head>`; load the rest with `<link rel="stylesheet" media="print" onload="...">` or similar deferred patterns.
- **Defer non-critical JS** (`defer` or `async`).
- **Preload critical assets** — `<link rel="preload" as="font" href="..." crossorigin>` for fonts; `<link rel="preload" as="image" fetchpriority="high">` for the LCP image.
- **Preconnect** to known origins (CDN, fonts, analytics).
- **Minify and compress** all text resources.

### The Preload Scanner
Modern browsers run a **preload scanner** alongside the main HTML parser. It speculatively scans the bytes ahead, finds resources (`<img>`, `<link>`, `<script>`), and starts fetching them in parallel — **even if the main parser is blocked by a sync script**.

This is one of the most underrated performance features of modern browsers. **Things that defeat the preload scanner:**
- CSS-only references (background images aren't found by the scanner)
- JS-injected resources (`document.createElement('script')` — too late)
- Resources behind dynamic imports

If your LCP image is a CSS background, the preload scanner can't help — it's discovered late, after CSSOM. Use `<img>` or preload it explicitly.

---

## Stage 7 — JavaScript Execution Model

JavaScript runs on the browser's **main thread** — the same thread that handles parsing, layout, paint, input events, and timers. **JS that runs blocks everything else.**

### The Event Loop

```
┌─────────────────────────────────────────────────────────────┐
│                       MAIN THREAD                           │
│                                                             │
│       ┌──────────────────────────────────────────┐          │
│       │              Call Stack                  │          │
│       │  (executes one frame at a time)          │          │
│       └──────────────────────────────────────────┘          │
│                          │                                  │
│                          ▼                                  │
│  When stack empty, event loop pulls from queues:            │
│                                                             │
│     ┌─────────────────────────────────────────┐             │
│     │     Microtask Queue (highest priority)  │ ← Promise   │
│     │                                         │   callbacks │
│     │     ──────────────────────────────────  │             │
│     │                                         │             │
│     │     Animation Frames (rAF)              │ ← before    │
│     │                                         │   render    │
│     │     ──────────────────────────────────  │             │
│     │                                         │             │
│     │     Render Steps (style/layout/paint)   │             │
│     │     ──────────────────────────────────  │             │
│     │                                         │             │
│     │     Macrotask Queue                     │ ← timers,   │
│     │     (one per loop iteration)            │   IO,       │
│     │                                         │   events    │
│     └─────────────────────────────────────────┘             │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Per "tick" of the loop, roughly:
1. Run one **macrotask** (e.g., a `setTimeout` callback or an event handler) to completion
2. Drain the **microtask queue** (Promises, `queueMicrotask`, MutationObserver)
3. If it's time to render: run `requestAnimationFrame` callbacks → style → layout → paint
4. Repeat

### Long Tasks Block Everything
A task > **50ms** is a "long task." During a long task:
- User input is queued, not processed → bad **INP**
- `rAF` callbacks don't run → animations jank
- Layout/paint don't happen → frame drops

Modern frameworks (React 18+, Solid, Svelte) and APIs (`scheduler.yield`, `scheduler.postTask`, `requestIdleCallback`) help break work into smaller chunks.

### Single-Threaded ≠ One Thread Total
The browser is **multi-process** and **multi-thread**, but JS itself is single-threaded **per page**. Workers (`Web Worker`, `Service Worker`, `OffscreenCanvas`) let you run JS on separate threads, with message-passing isolation.

---

## The Browser Architecture

A modern browser is not one process. Chrome's architecture (similar in Firefox, Safari):

```
┌────────────────────────────────────────────────────────────────────┐
│                       BROWSER ARCHITECTURE                         │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│  ┌──────────────────┐   ┌──────────────────┐   ┌────────────────┐  │
│  │ Browser Process  │   │ Renderer Process │   │  GPU Process   │  │
│  │ ───────────────  │   │ ───────────────  │   │ ─────────────  │  │
│  │ • UI (chrome)    │   │ • One per tab    │   │ • Compositing  │  │
│  │ • Tabs           │   │   (or per site)  │   │ • Raster       │  │
│  │ • Networking     │   │ • Main thread    │   │ • WebGL        │  │
│  │ • Storage        │   │ • Compositor     │   │                │  │
│  │ • Permissions    │   │   thread         │   │                │  │
│  │                  │   │ • Worker threads │   │                │  │
│  └──────────────────┘   │ • V8 (JS engine) │   └────────────────┘  │
│                         │ • Blink (engine) │                       │
│  ┌──────────────────┐   └──────────────────┘   ┌────────────────┐  │
│  │ Network Process  │                          │  Utility       │  │
│  │ ───────────────  │   ┌──────────────────┐   │  Processes     │  │
│  │ • Sockets        │   │ Plugin Process   │   │ ─────────────  │  │
│  │ • TLS            │   │                  │   │ • Audio        │  │
│  │ • HTTP cache     │   │                  │   │ • Storage      │  │
│  │ • CORS           │   │                  │   │ • etc.         │  │
│  └──────────────────┘   └──────────────────┘   └────────────────┘  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

### Why multi-process?
- **Security**: a renderer crash or exploit is sandboxed; can't access other tabs or the OS.
- **Stability**: one tab crashing doesn't kill the browser.
- **Performance**: parallelism across cores; freezing one tab doesn't freeze others.
- **Site Isolation**: cross-origin pages run in separate processes (defense against Spectre-class side channels).

### Threads inside the renderer
- **Main thread** — parsing, JS, style, layout, paint *commands*
- **Compositor thread** — independent of main; handles scroll, transform/opacity animations, layers (this is why scrolling stays smooth even when JS is busy)
- **Raster threads** — convert paint commands to bitmaps for layers
- **Worker threads** — Web Workers, Service Workers

> **Worth knowing:** the compositor thread is why CSS `transform` and `opacity` animations stay smooth during heavy JS — they're driven by the compositor independently. Animations of `top`, `left`, `width`, etc., must go through the main thread (layout!) and will jank.

---

## Reflow (Layout) and Repaint

These are the most-misunderstood parts of the rendering pipeline.

### Reflow (Layout)
Recomputing the geometry of part (or all) of the page.

**Triggers:**
- Adding/removing visible DOM nodes
- Changing element size (width, height, padding, border, margin)
- Changing position properties (top, left, etc., when they affect layout)
- Changing fonts (font size, font family — affects text metrics)
- Resizing the window
- Reading certain layout properties (see "forced synchronous layout" below)

**Cost:** O(n) where n is roughly the affected subtree. A reflow at the document root is the most expensive possible operation.

### Repaint
Re-rasterizing pixels for an element whose appearance changed but whose geometry didn't.

**Triggers:**
- Changing `color`, `background-color`, `visibility`
- Changing `box-shadow`, `border-radius`, `outline`
- Element becomes hidden via `visibility: hidden`

**Cost:** less than reflow — no geometry math — but still expensive on large layers.

### Composite-only changes (the cheapest!)
- `transform: translate/rotate/scale`
- `opacity`
- `filter` (in modern browsers)

These don't trigger layout *or* paint — only the compositor thread re-blends layers. **This is why animation libraries always recommend `transform` and `opacity`.**

### The Trigger Pyramid

```
                       ┌────────────────┐
                       │   Layout       │   ← Most expensive
                       │   (Reflow)     │     (e.g., width, top, font-size)
                       └────────────────┘
                              │
                              ▼
                     ┌──────────────────┐
                     │     Paint        │   ← Less expensive
                     │   (Repaint)      │     (e.g., color, background)
                     └──────────────────┘
                              │
                              ▼
                  ┌────────────────────────┐
                  │     Composite          │   ← Cheapest
                  │  (transform, opacity)  │     (GPU only)
                  └────────────────────────┘
```

Reference: [csstriggers.com](https://csstriggers.com/) lists every CSS property and what stage it triggers.

### Forced Synchronous Layout (Layout Thrashing)

The trap that catches everyone:

```js
// Each iteration: writes invalidate layout, then reads force recomputation
for (const item of items) {
  item.style.width = '100px';            // write: invalidates layout
  const w = item.offsetWidth;            // read: forces synchronous layout
  item.style.height = w + 'px';          // write: invalidates again
  const h = item.offsetHeight;           // read: forces again
}
```

This is **layout thrashing** — alternating writes (which dirty the layout) and reads (which force recomputation). Each iteration triggers a full layout. A 100-item loop becomes a 100-layout disaster.

### The fix: batch reads, then writes
```js
// Read phase — single layout
const sizes = items.map(item => item.offsetWidth);

// Write phase — invalidations batch into one layout next frame
items.forEach((item, i) => {
  item.style.width = '100px';
  item.style.height = sizes[i] + 'px';
});
```

### Properties that force synchronous layout (when read)
- `offsetWidth`, `offsetHeight`, `offsetTop`, `offsetLeft`, `offsetParent`
- `clientWidth`, `clientHeight`, `clientTop`, `clientLeft`
- `scrollWidth`, `scrollHeight`, `scrollTop`, `scrollLeft`
- `getBoundingClientRect()`, `getClientRects()`
- `getComputedStyle()` (for layout properties)
- `innerText` (forces layout to determine visibility)

> **Lesson learned:** React, Vue, and other frameworks batch DOM updates within a single tick to minimize layout. The classic way to defeat this is calling `ref.offsetHeight` inside a render or effect — an instant layout thrash that I've debugged in more codebases than I'd care to admit. Use `requestAnimationFrame` to defer reads when needed.

---

## Compositing — The GPU Pipeline

After paint, the browser has a set of **layers** (think Photoshop layers). The compositor's job: combine them into the final image.

### Why layers?
- Smooth scrolling — the page can scroll without re-painting if it's a single layer
- Cheap animations — transform/opacity changes don't require repaint, just re-composite
- Independent updates — only the changed layer needs re-rasterization

### What promotes an element to its own layer?
The browser auto-promotes elements that:
- Have a 3D `transform` (`translate3d`, `translateZ`) — historically the "GPU acceleration" trick
- Have `will-change: transform` or `will-change: opacity`
- Are `<video>`, `<canvas>` (with WebGL), `<iframe>`
- Have `position: fixed` (in some browsers)
- Have `filter`, `backdrop-filter`, `mask`
- Are in a `transform: translate3d(0,0,0)` ancestor

### The `will-change` hint
```css
.menu {
  will-change: transform;  /* Tells browser: I'm about to animate this */
}
```

`will-change` is a **promise**, not a free upgrade. Overusing it (e.g., on every element) wastes GPU memory and can degrade performance. Apply only to elements about to animate, and remove after.

### Layer explosion
Promoting too many elements creates "layer explosion" — each layer costs GPU memory (a 1920×1080 layer at 32-bit color = 8MB). Mobile devices have limited GPU memory; overflow causes terrible performance.

> Use Chrome DevTools → Rendering → "Layer borders" to visualize layers. If you see hundreds, you have a problem.

---

## Frame Lifecycle: How 60fps Actually Works

To hit 60fps, the browser has **16.67ms per frame**. To hit 120fps (high-refresh displays), **8.33ms**. Within that budget:

```
┌────────────────────── 16.67ms (60fps) ──────────────────────┐
│                                                             │
│  Input  ─►  JS  ─►  Style  ─►  Layout  ─►  Paint  ─► Comp.  │
│   ~1ms     ~5ms     ~1ms      ~3ms        ~3ms       ~2ms   │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

Miss the budget once → dropped frame (jank). Miss repeatedly → users feel it.

### The frame as work units
1. **Input handling** — process queued events (clicks, scrolls, touch)
2. **JS execution** — run timers, promises, `requestAnimationFrame` callbacks
3. **Style recalculation** — apply CSS rules to changed elements
4. **Layout** — recompute geometry if needed
5. **Paint** — generate paint commands
6. **Composite** — assemble final image (often on the compositor thread, in parallel with the next frame's main-thread work)

### `requestAnimationFrame` vs `setTimeout`
`rAF` runs **just before** the browser repaints — synchronized with the display refresh rate. `setTimeout(fn, 16)` is *not* aligned with the frame loop and causes jank.

```js
// ❌ unsynced
setInterval(updatePosition, 16);

// ✅ synced to refresh rate
function animate() {
  updatePosition();
  requestAnimationFrame(animate);
}
requestAnimationFrame(animate);
```

### Modern alternatives
- **`scheduler.postTask`** — explicit task priority (`user-blocking`, `user-visible`, `background`)
- **`scheduler.yield`** — explicitly yield mid-task to let the browser breathe
- **`requestIdleCallback`** — run work only when the browser is idle (don't use for critical work)

---

## Browser Caching

Caching is a multi-tier system. Each layer can short-circuit the network entirely.

```
Request flow:

  fetch()
     │
     ▼
  Service Worker  ──► (custom cache strategy)
     │ (passes through if no SW)
     ▼
  Memory Cache    ──► (in-process, very fast, cleared on tab close)
     │
     ▼
  Disk Cache      ──► (on-disk HTTP cache)
     │
     ▼
  Network         ──► CDN ─► Origin
```

### HTTP caching headers

| Header | Purpose |
|---|---|
| `Cache-Control` | The primary directive (`public`, `private`, `max-age`, `no-cache`, `no-store`, `immutable`, `stale-while-revalidate`) |
| `ETag` | Versioning identifier; server responds 304 if unchanged |
| `Last-Modified` / `If-Modified-Since` | Older revalidation mechanism |
| `Vary` | Cache key includes specified request headers |

### Strategies
- **Static assets with hashed filenames** (`/app.a1b2c3.js`) — `Cache-Control: public, max-age=31536000, immutable`
- **HTML** — `Cache-Control: no-cache` (must revalidate, but allow conditional caching) or `max-age=0, s-maxage=300` (browser fresh, CDN caches 5 min)
- **API responses** — case by case; often `private, max-age=0`

### `stale-while-revalidate`
```
Cache-Control: max-age=60, stale-while-revalidate=300
```
Browser uses cached response for 60s, then **serves stale** for up to 5 minutes while revalidating in the background. Excellent UX for slowly-changing data.

### The 304 Not Modified
On expiry, the browser sends a conditional request (`If-None-Match: "etag"`). If unchanged, the server replies `304` with no body — saves bandwidth, not the round trip.

---

## Common Performance Pitfalls

### 1. Render-blocking CSS in `<head>`
Every byte of CSS in `<head>` delays first paint. Audit and inline only what's critical.

### 2. Synchronous scripts in `<head>`
A 200KB sync `<script>` blocks parsing for ~50ms on a fast device, much longer on mobile. Use `defer` or `async`.

### 3. Layout thrashing
Read-write-read-write loops over the DOM. Batch reads, then writes.

### 4. Animating `top`/`left`/`width`/`height`
Triggers layout every frame. Use `transform` and `opacity`.

### 5. Long tasks on the main thread
Anything > 50ms blocks input. Break up with `scheduler.yield`, Web Workers, or chunking.

### 6. CSS background as the LCP image
Preload scanner can't find it. Use `<img>` with `fetchpriority="high"`.

### 7. Unoptimized fonts (FOIT/FOUT/CLS)
Use `font-display: swap` or `optional`, preload critical fonts, match fallback metrics.

### 8. Many distinct origins
Each origin = DNS + TCP + TLS. Consolidate where possible; preconnect the rest.

### 9. Missing compression
Brotli or gzip on all text. Surprisingly common to find uncompressed JSON or SVG.

### 10. Polling instead of events
Setting up `setInterval` to poll instead of using events, observers, or push channels.

### 11. Forgetting bfcache
A single `unload` listener disables bfcache and costs you instant back-navigation.

### 12. No HTTP/2 (or /3)
Still serving HTTP/1.1? You're paying for serialized requests on a single connection.

### 13. Memory leaks from forgotten event listeners
SPAs that don't clean up listeners or observers grow unbounded; tab eventually slows to a crawl.

### 14. Ignoring the network panel
Most performance bugs are visible in DevTools → Network → Waterfall in 30 seconds. Look at it before guessing.

---

## Mental Models

### 1. "Bytes flow, the compositor blends, the main thread is the bottleneck"
Network delivers bytes. The main thread serializes parsing → JS → layout → paint. The compositor blends layers on the GPU. **Most performance work is reducing main-thread serialization.**

### 2. "Every render-blocking byte is LCP latency"
There's a 1:1 relationship between render-blocking resources and first-paint delay. Inline critical, defer the rest.

### 3. "Reads dirty layout when paired with writes"
Don't read layout properties between writes — that's layout thrashing. Either batch (read all, then write all) or `requestAnimationFrame` to defer.

### 4. "Promote layers, don't paint them"
Animations of `transform` and `opacity` skip layout and paint entirely. Animations of layout properties cost a frame budget.

### 5. "The browser is on your side"
Modern browsers do **enormous** work to hide bad code: preload scanners, speculative parsing, layout incrementalization, paint caching, GPU compositing, bfcache, lazy hydration support. **Don't fight the browser — let it help.** That means using the standard APIs (`<img>`, semantic HTML, `loading="lazy"`, `fetchpriority`) instead of bespoke JS solutions.

### 6. "TTFB caps everything"
LCP ≥ TTFB. INP is bounded by main-thread freedom. CLS depends on layout stability. **You can't optimize past your TTFB.**

### 7. "Performance is a budget, not a tactic"
Set per-route budgets (LCP < 2.5s, INP < 200ms, CLS < 0.1, JS < 170KB). Enforce in CI. Investigate every regression. The discipline of "we don't ship over budget" prevents 90% of slow sites.

### 8. "Lab is fiction; field is truth"
Lighthouse on your laptop is a useful tool, not a north star. Real users on mid-tier Android over 4G — that's the bar. Collect RUM (`web-vitals` library) and align lab thresholds to field reality.

### 9. "Cold cache, slow CPU, far away"
Always test as if the user has none of your cache, a 4-year-old phone, and is 200ms from your server. If it's fast there, it's fast everywhere.

### 10. "The pipeline is older than your framework"
The CRP existed before React, Vue, Angular. Frameworks layer on top; they don't replace it. **Understand the platform first; the framework second.** Engineers who know the browser ship faster sites in any framework.

---

## Further Reading

### Foundational
- **[web.dev — Inside look at modern web browser](https://developer.chrome.com/blog/inside-browser-part1/)** — Mariko Kosaka's 4-part series; the best free deep-dive
- **[How Browsers Work (Tali Garsiel)](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)** — slightly dated but still gold; the foundational article
- **[web.dev — Critical Rendering Path](https://web.dev/articles/critical-rendering-path)** — Ilya Grigorik
- **Ilya Grigorik — *High Performance Browser Networking*** — free online; *the* book on browser networking

### Performance Internals
- **[csstriggers.com](https://csstriggers.com/)** — every CSS property and what it triggers
- **[Layout Thrashing — Wilson Page](https://gist.github.com/paulirish/5d52fb081b3570c81e3a)**
- **[Avoid Large, Complex Layouts](https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing)**
- **[Browser Rendering Performance (Paul Lewis et al., web.dev)](https://web.dev/articles/rendering-performance)**

### Chrome Internals
- **[Chromium Design Docs](https://www.chromium.org/developers/design-documents/)** — actual source-of-truth from the engine team
- **[V8 Blog](https://v8.dev/blog)** — JS engine deep-dives
- **[Jake Archibald — In the Loop (JSConf.asia 2018)](https://www.youtube.com/watch?v=cCOL7MC4Pl0)** — *the* talk on the event loop. Required viewing.
- **[Tasks, microtasks, queues and schedules — Jake Archibald](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/)**

### Networking
- **[High Performance Browser Networking](https://hpbn.co/)** — Ilya Grigorik (free online)
- **[HTTP/3 Explained](https://http3-explained.haxx.se/)** — Daniel Stenberg (curl maintainer)
- **[Cloudflare Learning Center](https://www.cloudflare.com/learning/)** — accessible explainers on TLS, HTTP/2, HTTP/3, QUIC

### Tools
- **Chrome DevTools — Performance panel** — flame charts, frame breakdown, long tasks
- **Chrome DevTools — Performance Insights** — newer, easier-to-read perf UI
- **Chrome DevTools — Rendering tab** — Layer borders, FPS meter, paint flashing
- **WebPageTest** — best-in-class waterfall + filmstrip + connection emulation
- **Lighthouse** — opinionated audits, including CRP analysis

### Books
- **Steve Souders — *High Performance Web Sites*** — the original "rules" book; dated but foundational
- **Tom Barker — *High Performance Images***
- **Smashing Magazine — *Front-end Performance*** (annual checklist by Vitaly Friedman) — the most current practical reference

---

## Key terms

This guide is the canonical home for these glossary entries: **Critical Rendering Path**, **DOM**, **CSSOM**, **Render Tree**, **Layout (Reflow)**, **Paint**, **Composite**, **Event Loop**, **GPU Compositor**, **bfcache**, **DNS**, **TCP**, **TLS**, **HTTP/2**, **HTTP/3 / QUIC**. See the [glossary](glossary.md) for definitions and the rest of the index.

---

> **Closing thought:** the entire journey from URL to pixels is a marvel of engineering — DNS resolution, TLS handshakes, HTTP streaming, parallel parsing, speculative resource discovery, layered compositing, GPU rasterization, all coordinated in milliseconds across multiple processes. I don't try to keep every detail in my head; what matters is the **mental map of the pipeline** so that when something is slow, I know *which stage* to suspect, *which tool* to reach for, and *which lever* to pull. The browser is the most sophisticated piece of software most users ever interact with — and the more deeply you understand it, the more leverage you have over every framework that runs on top of it. The investment pays dividends forever.
