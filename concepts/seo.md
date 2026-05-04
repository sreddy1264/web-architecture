# Search Engine Optimization (SEO)

*Last reviewed: 2026-05*

> A practical engineering guide to building search-friendly web applications.

---

## Table of Contents

1. [What is SEO?](#what-is-seo)
2. [Why is SEO Important?](#why-is-seo-important)
3. [How Search Engines Actually Work](#how-search-engines-actually-work)
4. [The Three Pillars of SEO](#the-three-pillars-of-seo)
5. [How to Implement SEO](#how-to-implement-seo)
6. [SEO for JavaScript / SPAs](#seo-for-javascript--spas)
7. [Core Web Vitals & Performance](#core-web-vitals--performance)
8. [Precautions and Common Pitfalls](#precautions-and-common-pitfalls)
9. [Measurement & Tooling](#measurement--tooling)
10. [SEO in the Age of AI Search](#seo-in-the-age-of-ai-search)
11. [Quick Reference Checklist](#quick-reference-checklist)
12. [Further Reading](#further-reading)

---

## What is SEO?

**Search Engine Optimization (SEO)** is the practice of improving a website's visibility in **organic (non-paid) search results** so that the right users can discover it for the right queries.

It is the intersection of three disciplines:

| Discipline | Focus |
|---|---|
| **Technical SEO** | Crawlability, indexability, performance, structured data, site architecture |
| **On-page SEO** | Content quality, keywords, semantic HTML, metadata, internal linking |
| **Off-page SEO** | Backlinks, brand mentions, authority, reputation |

> SEO is **not** about gaming algorithms. It's about making your site **understandable to machines and valuable to humans** — those goals have converged over the last decade.

---

## Why is SEO Important?

### 1. Organic Search is the Largest Traffic Source on the Web
- Roughly **53%** of all website traffic comes from organic search (BrightEdge research).
- The **#1 result in Google captures ~27–40%** of clicks; results past page 1 get under 1%.
- Search drives **10x more traffic** than organic social media for most B2B and content sites.

### 2. Compounding ROI
Unlike paid ads, organic rankings keep working after you stop investing. A well-optimized page can drive traffic for years. The cost of SEO is largely up-front; the return is amortized.

### 3. High-Intent, High-Quality Traffic
Search users are **actively looking** for something. Conversion rates from organic search consistently outperform display, social, and many paid channels.

### 4. Trust & Credibility
Users trust organic results more than ads. Ranking high signals authority and increases brand perception.

### 5. Defensive Necessity
If you're not visible, your competitors are. Losing rankings on branded queries is a leak; losing rankings on category queries is existential.

### 6. SEO and AI/LLM Search Overlap
Generative engines (Google AI Overviews, ChatGPT search, Perplexity, Claude) cite the same authoritative, well-structured content that ranks in classic search. **Good SEO is now also good GEO** (Generative Engine Optimization).

---

## How Search Engines Actually Work

Understanding the pipeline lets you debug ranking issues precisely.

### 1. **Crawling**
Bots (Googlebot, Bingbot, etc.) follow links and sitemaps to discover URLs. They have a **crawl budget** — a finite number of requests per site.

### 2. **Rendering** (for JS sites)
Modern crawlers render JavaScript in a headless Chrome. Google uses **evergreen Googlebot** (latest stable Chrome). Rendering happens in a **second wave** that can be **delayed by hours to days** behind the initial HTML crawl.

### 3. **Indexing**
The rendered content is parsed, deduplicated (canonicalized), and stored in the index. **A page must be indexed to rank.** "Crawled but not indexed" and "Discovered but not crawled" are the two most common visibility killers.

### 4. **Ranking**
For each query, the engine scores indexed pages using hundreds of signals — relevance, authority, freshness, quality, intent match, user signals, and contextual factors (location, language, device, history).

### 5. **Serving (SERP)**
Results are assembled with classic links, featured snippets, knowledge panels, sitelinks, image/video carousels, "People Also Ask," and increasingly **AI Overviews**.

> **Implication:** A ranking problem is almost always a problem at one of these stages. Diagnose by walking the pipeline: *Is it crawled? Rendered? Indexed? If indexed, why isn't it ranking?*

---

## The Three Pillars of SEO

### 1. Technical SEO — *Can search engines access your site?*
- Crawlable URLs (no auth walls, no infinite spaces)
- Clean `robots.txt` and XML sitemaps
- Fast, mobile-friendly, secure (HTTPS)
- Correct status codes (200, 301, 404, 410, 503)
- Proper canonicalization, hreflang, pagination
- Structured data (Schema.org)

### 2. On-Page SEO — *Does your content match user intent?*
- Keyword research aligned to **search intent** (informational, navigational, commercial, transactional)
- Semantic HTML structure
- Compelling titles & meta descriptions
- Internal linking with descriptive anchor text
- Multimedia with alt text and transcripts

### 3. Off-Page SEO — *Do others trust your content?*
- Earning quality backlinks (digital PR, content marketing, partnerships)
- Brand mentions across the web
- Reviews, citations, profiles (especially for local SEO)
- Topical authority — being known for a subject

---

## How to Implement SEO

### 1. Information Architecture
Plan before you build. A flat, logical URL hierarchy ranks better than a deep, chaotic one.

```
✅ /blog/web-accessibility
✅ /products/running-shoes/men
❌ /p/?id=12345&cat=run&g=m
❌ /blog/2024/03/27/some-very-long-auto-generated-slug-with-stop-words-in-it
```

URL guidelines:
- Lowercase, hyphen-separated, short
- Descriptive of content, not implementation
- Stable — don't change without 301 redirects
- Avoid URL parameters for canonical content

### 2. The HTML Head — Get the Basics Right

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!-- Title: 50–60 chars, keyword + brand, unique per page -->
    <title>Web Accessibility: A Practical Engineering Guide | YourBrand</title>

    <!-- Meta description: 140–160 chars, compelling, includes intent -->
    <meta
      name="description"
      content="A practical engineering guide to web accessibility — WCAG, ARIA, testing, and React patterns. Learn why a11y matters and how to ship inclusive products."
    />

    <!-- Canonical: prevents duplicate content -->
    <link rel="canonical" href="https://example.com/blog/web-accessibility" />

    <!-- Open Graph (LinkedIn, Slack, Facebook) -->
    <meta property="og:title" content="Web Accessibility: A Practical Engineering Guide" />
    <meta property="og:description" content="..." />
    <meta property="og:image" content="https://example.com/og/a11y.png" />
    <meta property="og:url" content="https://example.com/blog/web-accessibility" />
    <meta property="og:type" content="article" />

    <!-- Twitter / X -->
    <meta name="twitter:card" content="summary_large_image" />
    <meta name="twitter:title" content="..." />

    <!-- hreflang for multilingual sites -->
    <link rel="alternate" hreflang="en" href="https://example.com/en/..." />
    <link rel="alternate" hreflang="es" href="https://example.com/es/..." />
    <link rel="alternate" hreflang="x-default" href="https://example.com/en/..." />
  </head>
</html>
```

### 3. Semantic HTML & Heading Hierarchy
Search engines weight semantic structure heavily.

```html
<header><nav>...</nav></header>
<main>
  <article>
    <h1>One H1 per page — primary topic</h1>
    <p>Lead paragraph that summarizes the article.</p>
    <h2>Major section</h2>
    <h3>Subsection</h3>
    <h2>Another major section</h2>
  </article>
</main>
<footer>...</footer>
```

Rules:
- **Exactly one `<h1>`** per page, near the top
- Don't skip heading levels (`h2` → `h4`)
- Use real headings, not styled `<div>`s
- Place primary keywords naturally in `<h1>` and the first 100 words

### 4. Content That Matches Search Intent
Modern SEO is **intent matching**, not keyword stuffing.

For each target query, ask: *what does the user actually want?*

| Intent | Format that wins |
|---|---|
| **Informational** ("what is SEO") | Long-form guide, definitions, examples |
| **Navigational** ("github login") | Brand homepage, fast load |
| **Commercial** ("best running shoes 2026") | Comparison, reviews, pros/cons |
| **Transactional** ("buy nike pegasus 41") | Product page, clear CTA, price, stock |

Then build content that satisfies the intent **better than the current top results** — more depth, fresher data, clearer structure, original insight.

#### E-E-A-T (Google's quality framework)
- **Experience** — first-hand experience with the topic
- **Expertise** — credentials, depth
- **Authoritativeness** — recognized in the space
- **Trustworthiness** — accurate, transparent, secure

Demonstrate E-E-A-T with: clear authorship, bios, citations, original data, case studies, "About" / "Contact" pages, HTTPS, accurate site info.

### 5. Internal Linking
- Link to related pages with **descriptive anchor text** (not "click here")
- Build **topic clusters** — a pillar page + supporting articles all linking to each other
- Surface deep pages from your homepage and high-traffic posts (passes "link equity")
- Fix orphan pages (no internal links → poor crawl/rank)

### 6. Images & Media

```html
<img
  src="/img/running-shoes.webp"
  alt="Nike Pegasus 41 men's running shoe in blue"
  width="800" height="600"
  loading="lazy"
  decoding="async"
/>
```

- **Modern formats**: WebP, AVIF (40–60% smaller than JPEG)
- **Responsive images**: `srcset`, `sizes`, `<picture>`
- **Always set width/height** to prevent CLS
- **Lazy-load** below-the-fold images (`loading="lazy"`) — but never the LCP image
- **Descriptive filenames**: `running-shoes-blue.webp`, not `IMG_4427.jpg`
- **Alt text** doubles as accessibility + image search SEO

### 7. Structured Data (Schema.org)
JSON-LD is Google's recommended format. It powers rich results — star ratings, FAQ accordions, recipe cards, product info, breadcrumbs.

```html
<script type="application/ld+json">
{
  "@context": "https://schema.org",
  "@type": "Article",
  "headline": "Web Accessibility: A Practical Engineering Guide",
  "author": {
    "@type": "Person",
    "name": "Jane Doe",
    "url": "https://example.com/authors/jane"
  },
  "datePublished": "2026-04-30",
  "dateModified": "2026-04-30",
  "publisher": {
    "@type": "Organization",
    "name": "YourBrand",
    "logo": {"@type": "ImageObject", "url": "https://example.com/logo.png"}
  },
  "image": "https://example.com/og/a11y.png"
}
</script>
```

Common useful types:
- `Article` / `BlogPosting` / `NewsArticle`
- `Product` + `Offer` + `AggregateRating` + `Review`
- `FAQPage` (note: visibility reduced in 2023, but still useful)
- `HowTo`, `Recipe`, `Event`, `LocalBusiness`, `Organization`
- `BreadcrumbList`
- `VideoObject`, `Course`, `JobPosting`

Validate with **[Google's Rich Results Test](https://search.google.com/test/rich-results)** and **[Schema Markup Validator](https://validator.schema.org/)**.

### 8. Sitemaps & robots.txt

**robots.txt** — controls *crawling*, not indexing:
```
User-agent: *
Disallow: /admin/
Disallow: /api/
Allow: /

Sitemap: https://example.com/sitemap.xml
```

**sitemap.xml** — tells crawlers what to find:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/</loc>
    <lastmod>2026-04-30</lastmod>
    <changefreq>weekly</changefreq>
    <priority>1.0</priority>
  </url>
</urlset>
```

- One sitemap per content type (pages, posts, products, images, videos)
- Use a sitemap index for >50K URLs or >50MB
- Submit to Google Search Console and Bing Webmaster Tools

### 9. Canonicalization
Tell engines which URL is the "real" one when multiple paths show the same content.

```html
<!-- On every variant of /shoes?color=blue, /shoes?ref=email, etc. -->
<link rel="canonical" href="https://example.com/shoes" />
```

Also normalize:
- `http` → `https`
- `www` ↔ non-`www` (pick one, 301 the other)
- Trailing slash consistency
- Uppercase → lowercase URLs

### 10. Redirects & Status Codes
- **301** — permanent move (passes ~100% of link equity now; "lose 15%" is outdated)
- **302** — temporary move (don't use for permanent changes)
- **404** — gone, no replacement
- **410** — gone permanently (faster de-indexing than 404)
- **503** — temporary unavailability (use during maintenance — Googlebot will retry)

Avoid redirect chains (`A → B → C`) — collapse to single hops. Never redirect a deep page to your homepage; it's a "soft 404" signal.

### 11. International SEO
- **Subdirectory** (`/es/`) is usually better than country TLDs unless you have country-specific operations
- **`hreflang`** on every alternate version, including a `x-default`
- Translate content — don't just translate metadata
- Localize: currencies, dates, units, examples, idioms

### 12. Mobile-First Indexing
Google indexes the **mobile version** of your site as primary. Ensure:
- Mobile and desktop content is **equivalent** (don't hide content on mobile)
- Structured data is on both
- Images and links are reachable on mobile
- Tap targets are large enough, viewport is set

---

## SEO for JavaScript / SPAs

Since this is a React codebase, this section matters.

### The Core Problem
A vanilla React SPA renders to `<div id="root"></div>`. Googlebot **does** execute JS — but rendering is queued, delayed, expensive, and not perfectly reliable. Other crawlers (Bing, social previews, AI scrapers) may render less well or not at all.

### Rendering Strategies

| Strategy | How | Best for |
|---|---|---|
| **Static Site Generation (SSG)** | Pre-render HTML at build time | Marketing sites, blogs, docs — content that changes infrequently |
| **Server-Side Rendering (SSR)** | Render HTML on each request | Personalized or frequently-updated content |
| **Incremental Static Regeneration (ISR)** | Pre-render, revalidate on a TTL | E-commerce, large content sites |
| **Client-Side Rendering (CSR)** | JS in the browser only | Authenticated dashboards (no SEO need) |
| **Edge Rendering** | SSR at the CDN edge | Latency-sensitive global content |

**Modern recommendation:** for any public, indexable content, ship **server-rendered HTML on the first response**. The user (and Googlebot) gets meaningful content immediately; client hydration enriches it.

### Frameworks
- **Next.js** — App Router (RSC + streaming SSR), SSG, ISR — the de facto React SEO framework
- **Remix / React Router 7** — server-first, nested routing
- **Astro** — partial hydration ("islands"), excellent for content sites
- **Gatsby** — SSG (older but stable)

### Per-Route Metadata
Every route needs unique `<title>`, `<meta>`, canonical, and OG tags. Don't ship the same meta on every page.

```jsx
// Next.js App Router
export const metadata = {
  title: 'Web Accessibility Guide | YourBrand',
  description: '...',
  alternates: { canonical: 'https://example.com/blog/web-accessibility' },
  openGraph: { /* ... */ },
};
```

### Common JS-SEO Mistakes
- Rendering content only after `useEffect` runs → crawlers may miss it
- Using `<a>` without `href` (e.g., `<a onClick>`) → not crawlable as a link
- Client-side redirects via `useEffect` → use server `301`/`302` instead
- Hash-based routing (`/#/about`) → engines treat as one URL
- Lazy-loading critical content behind interactions → invisible to crawlers
- `noindex` accidentally shipped to production from a staging template

### Verify Rendering
- **URL Inspection** in Google Search Console — see what Google actually rendered
- `curl -A "Googlebot" https://yoursite.com/page` — see the raw HTML response
- `view-source:` — see what arrives before JS runs

---

## Core Web Vitals & Performance

Google uses **Core Web Vitals** as a ranking signal. They're also a UX win on their own.

| Metric | What it measures | Good | Needs work | Poor |
|---|---|---|---|---|
| **LCP** (Largest Contentful Paint) | When the main content paints | ≤ 2.5s | ≤ 4.0s | > 4.0s |
| **INP** (Interaction to Next Paint) | Responsiveness to input | ≤ 200ms | ≤ 500ms | > 500ms |
| **CLS** (Cumulative Layout Shift) | Visual stability | ≤ 0.1 | ≤ 0.25 | > 0.25 |

> **INP replaced FID** as a Core Web Vital in March 2024. It measures the *worst* latency across all interactions, not just the first. Pay attention.

### High-Leverage Optimizations
- **LCP**: preload the hero image, serve from CDN, use modern formats, eliminate render-blocking CSS/JS, server-render
- **INP**: break up long tasks, debounce expensive handlers, defer non-critical JS, avoid heavy work in event handlers
- **CLS**: set explicit `width`/`height` on images and embeds, reserve space for ads/banners, avoid injecting content above existing content, use `font-display: optional` or preload fonts

### Other performance wins (indirect SEO benefit)
- HTTP/2 or HTTP/3
- Brotli/gzip compression
- Long-cache static assets, immutable hashes
- `<link rel="preload">` for fonts, hero images
- `<link rel="preconnect">` for third-party origins
- Image CDNs with automatic format negotiation
- Code-split by route, lazy-load below-the-fold components
- Eliminate unused JS (audit with Coverage tab in DevTools)

---

## Precautions and Common Pitfalls

### 1. Don't Block Resources Googlebot Needs
Don't `Disallow:` your CSS or JS in `robots.txt`. Google needs to render the page. (Common in old WordPress/CMS setups.)

### 2. Don't Confuse `noindex`, `disallow`, and `canonical`
| Directive | Effect |
|---|---|
| `robots.txt: Disallow` | Don't crawl. Page **may still be indexed** if linked elsewhere (just shows no snippet). |
| `<meta name="robots" content="noindex">` | Crawler must **see** it (so don't also Disallow). Removes from index. |
| `<link rel="canonical">` | Hint about the preferred URL. Not a hard rule. |
| `301 redirect` | Strong consolidation. Old URL replaced by new in index. |

A page disallowed in `robots.txt` **cannot be de-indexed via meta noindex** because the crawler can't read the meta tag. Allow crawl + use noindex.

### 3. Don't Ship `noindex` to Production
Staging environments often set `<meta name="robots" content="noindex,nofollow">` site-wide. If that template ships unchanged, your traffic dies overnight. Add a CI check that fails the build if production HTML contains `noindex`.

### 4. Don't Cloak
Showing different content to bots vs users is against guidelines and earns manual penalties. The safest rule: **Googlebot should see exactly what users see.**

### 5. Don't Chase Algorithm Updates with Tricks
Algorithm volatility (March 2024 core update, helpful content updates, spam updates) is the new normal. Sites built on quality, intent match, and user value recover; sites built on tricks don't.

### 6. Don't Buy Backlinks
Paid link schemes are the most common cause of manual penalties. Use `rel="sponsored"` for paid placements, `rel="ugc"` for user content, `rel="nofollow"` for untrusted links.

### 7. Don't Duplicate Content
- Same content on multiple URLs → canonicalize
- Boilerplate-heavy thin pages (programmatic SEO done badly) → consolidate or noindex
- Syndicated content → require canonical back to your URL

### 8. Don't Over-Optimize Anchor Text
A natural backlink profile has varied anchors (brand, URL, generic, exact match). 100 inbound links all reading "best running shoes" is a manipulation signal.

### 9. Don't Build Massive Programmatic Pages Without Quality
"pSEO" works only when the generated pages are genuinely useful. Thin templates × millions of URLs = a Helpful Content Update casualty.

### 10. Don't Migrate Sites Without a Plan
Site migrations (domain change, replatform, redesign) are the #1 cause of self-inflicted traffic drops. Mandatory: full URL inventory, 301 map, structured data preserved, sitemap updated, Search Console property added, post-launch monitoring for 30+ days.

### 11. Don't Forget About `https`, Security, and HSTS
HTTPS is a (small) ranking factor and a trust signal. Mixed content, expired certs, or insecure forms erode rankings and conversion.

### 12. Don't Treat SEO as a One-Off Project
Search results, competition, and algorithms shift constantly. Quarterly content audits, monthly Search Console reviews, and continuous link earning are baseline operations.

### 13. Don't Underestimate Crawl Budget on Big Sites
Sites with millions of URLs need:
- `noindex` on faceted-search permutations
- Parameter handling (Search Console, or canonical)
- Robust sitemaps that exclude low-value pages
- Server logs analysis to see what Googlebot actually crawls

### 14. Don't Hide Important Content Behind Tabs/Accordions/Click-to-Expand on Mobile
Google indexes hidden content but may weight it less. Critical content should be visible by default.

### 15. Beware AI-Generated Mass Content
Google's stance: AI content is fine **if it's helpful, accurate, and demonstrates E-E-A-T**. AI-generated thin content scaled to game search has been hit hard by spam updates. Use AI to assist, not to replace, expert content.

---

## Measurement & Tooling

### Free, Essential Tools
- **[Google Search Console](https://search.google.com/search-console)** — performance, indexing, errors, sitemaps. Non-negotiable.
- **[Bing Webmaster Tools](https://www.bing.com/webmasters)** — same for Bing
- **[Google Analytics 4](https://analytics.google.com/)** — traffic, conversions
- **[PageSpeed Insights](https://pagespeed.web.dev/)** — Core Web Vitals (lab + field/CrUX data)
- **[Lighthouse](https://developer.chrome.com/docs/lighthouse/)** — built into Chrome DevTools
- **[Rich Results Test](https://search.google.com/test/rich-results)** — validate structured data
- **[Mobile-Friendly Test](https://search.google.com/test/mobile-friendly)**
- **[Screaming Frog SEO Spider](https://www.screamingfrog.co.uk/)** — free up to 500 URLs; the industry-standard crawler

### Paid Platforms
- **Ahrefs** — backlinks, keyword research, content gap analysis
- **Semrush** — competitor research, position tracking
- **Sistrix** — visibility index, especially strong in Europe
- **Botify** / **OnCrawl** — enterprise log file analysis & crawl management
- **Conductor**, **BrightEdge** — enterprise SEO platforms

### Metrics That Matter
| Stage | Metric |
|---|---|
| **Indexing health** | Pages indexed vs submitted, coverage errors |
| **Visibility** | Impressions, average position, share of voice |
| **Engagement** | CTR by query, bounce rate, time on page |
| **Outcome** | Organic conversions, organic revenue, assisted conversions |
| **Technical** | Core Web Vitals (field data, not lab), crawl stats, server errors |

---

## SEO in the Age of AI Search

The SERP is evolving. Google's **AI Overviews**, **ChatGPT search**, **Perplexity**, **Claude**, and **Bing Copilot** synthesize answers and cite sources.

### What Changes
- **Zero-click queries are growing** — users get answers without clicking
- **Citations matter more than rank** — being one of 3 cited sources beats being #5 in classic results
- **Brand recall and direct traffic become more valuable** — defensive moats

### What Stays the Same (and Gets More Important)
- Authoritative, well-structured, factually accurate content
- Clear semantic HTML and structured data — easier for LLMs to parse
- Topical authority — being the recognized voice on a subject
- Original research, data, and quotes — citation-worthy assets

### Practical Adjustments — "GEO" (Generative Engine Optimization)
- Lead with a clear, concise answer (LLMs grab the summary)
- Use Q&A and definition formats; FAQ sections still help
- Maintain consistent entity information (Schema.org `Organization`, `Person`, sameAs links to LinkedIn/Wikipedia/Wikidata)
- Get cited in places LLMs trust — Reddit, GitHub, Wikipedia, established publications
- Robots files: decide whether to allow `GPTBot`, `Google-Extended`, `ClaudeBot`, `PerplexityBot` — opting out reduces visibility in AI answers but protects content. There is no consensus best answer here.

---

## Quick Reference Checklist

### Pre-launch
- [ ] Unique `<title>` and meta description on every indexable page
- [ ] Single `<h1>` per page, logical heading hierarchy
- [ ] Canonical tags on every page
- [ ] XML sitemap submitted to Search Console
- [ ] `robots.txt` correct, no accidental site-wide block
- [ ] Verified property in Google Search Console + Bing Webmaster Tools
- [ ] HTTPS with valid cert, HSTS enabled
- [ ] Structured data validated (Rich Results Test)
- [ ] Open Graph + Twitter card metadata
- [ ] 404 page returns real 404; soft-404s eliminated
- [ ] Mobile-friendly test passes
- [ ] Core Web Vitals "Good" on key templates
- [ ] No `noindex` on production pages
- [ ] Internal linking from homepage to key pages
- [ ] Image alt text and modern formats

### Ongoing
- [ ] Monthly Search Console review (queries, coverage, CWV)
- [ ] Quarterly content audit (refresh / consolidate / prune)
- [ ] Backlink monitoring (toxic link disavow if needed)
- [ ] Competitor SERP tracking on priority keywords
- [ ] Server log analysis on large sites
- [ ] Regression checks on releases (titles, canonicals, structured data, status codes)

---

## Further Reading

- **[Google Search Central](https://developers.google.com/search)** — official documentation
- **[Google Search Quality Rater Guidelines (PDF)](https://services.google.com/fh/files/misc/hsw-sqrg.pdf)** — how human raters evaluate quality; reveals what Google optimizes for
- **[web.dev — Learn SEO](https://web.dev/learn/seo)**
- **[Schema.org](https://schema.org/)**
- **[Ahrefs Blog](https://ahrefs.com/blog)** / **[Backlinko](https://backlinko.com/)** / **[SEO by the Sea](https://www.seobythesea.com/)**
- **[Search Engine Journal](https://www.searchenginejournal.com/)** / **[Search Engine Land](https://searchengineland.com/)**
- **[Aleyda Solis — SEO Newsletters](https://www.aleydasolis.com/)**
- Books: *The Art of SEO* (Enge et al.), *SEO for Growth*, *Product-Led SEO* (Eli Schwartz)

---

## Key terms

This guide is the canonical home for these glossary entries: **Canonical**, **Structured Data**, **`robots.txt`**, **`sitemap.xml`**, **Hreflang**, **Crawl Budget**, **E-E-A-T**, **Core Web Vitals**, **AEO** *(Answer Engine Optimization)*, **GEO** *(Generative Engine Optimization)*. See the [glossary](glossary.md) for definitions and the rest of the index.

---

> **Closing thought:** SEO is not a checklist of tags — it's a discipline of aligning **technical excellence, user intent, and editorial judgment** so the right people find the right page at the right time. The mistake I see most often is treating SEO as marketing's problem; the teams that win treat it as an architectural concern from day one and ship structures that compound for years. Get the foundations right early, and ranking becomes a side effect of building genuinely useful things.
