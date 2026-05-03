# CSS Architecture & Design Systems

> A practical guide to writing, organizing, and shipping CSS at scale — methodology, modern features, design tokens, design systems, performance, and best practices.

---

## Table of Contents

1. [Why CSS Architecture Matters](#why-css-architecture-matters)
2. [The CSS Mindset](#the-css-mindset)
3. [Choosing a Styling Approach](#choosing-a-styling-approach)
4. [CSS Methodologies](#css-methodologies)
5. [The Cascade, Specificity, and `@layer`](#the-cascade-specificity-and-layer)
6. [Modern CSS Features You Should Be Using](#modern-css-features-you-should-be-using)
7. [Layout Systems](#layout-systems)
8. [Responsive Design Beyond Media Queries](#responsive-design-beyond-media-queries)
9. [Design Tokens](#design-tokens)
10. [Theming (Light/Dark and Beyond)](#theming-lightdark-and-beyond)
11. [Design Systems](#design-systems)
12. [Critical CSS](#critical-css)
13. [CSS Performance](#css-performance)
14. [Animation & Motion](#animation--motion)
15. [Accessibility in CSS](#accessibility-in-css)
16. [Internationalization & Logical Properties](#internationalization--logical-properties)
17. [Forms & Print Styles](#forms--print-styles)
18. [CSS Tooling](#css-tooling)
19. [Common Anti-Patterns](#common-anti-patterns)
20. [Quick Reference Checklist](#quick-reference-checklist)
21. [Further Reading](#further-reading)

---

## Why CSS Architecture Matters

CSS is the most globally-scoped, side-effect-laden language most engineers use daily. A class added in one file can break layout in another. A color changed in one component can ripple across the whole app. **Without architecture, CSS rots faster than any other layer.**

### The CSS rot pattern
1. Day 1: clean, semantic, organized.
2. Month 6: a few `!important` declarations, some inline overrides.
3. Year 2: 30,000-line stylesheets, everyone afraid to delete anything, every PR adds 200 lines and removes 0.
4. Year 4: rewrite with the new framework.

### What good CSS architecture buys you
- **Confidence to delete** — knowing a style is unused
- **Predictable cascade** — one source of truth per visual property
- **Refactor safety** — changing one component doesn't break another
- **Onboarding velocity** — new engineers learn the system, not 12 inconsistent patterns
- **Design fidelity at scale** — consistent across teams without endless review

> **The takeaway:** every major CSS methodology (BEM, ITCSS, OOCSS, atomic, utility-first) is a *response to scale*. Pick one, commit fully, enforce with tooling. **Mixing methodologies is worse than picking the wrong one.**

---

## The CSS Mindset

### 1. Embrace the cascade — don't fight it
The cascade and inheritance are CSS's greatest features. Frameworks that "scope every style" (CSS-in-JS with hashed classes) are sometimes solving real problems, sometimes throwing away CSS's leverage.

### 2. Write CSS that survives content changes
A layout that breaks when the heading is one character longer is broken by design. Build for the unknown — variable text, RTL, missing images, long URLs, accessibility zoom.

### 3. Style states, not snapshots
Every interactive element has states: default, hover, focus, focus-visible, active, disabled, selected, error. Designs often show one state; ship all of them.

### 4. Style via tokens, not magic numbers
`margin: 12px` is a mystery. `margin: var(--space-3)` is a system. Tokens turn design into code; magic numbers turn code into a maintenance burden.

### 5. The platform is bigger than you think
Modern CSS does layouts, animations, theming, scroll snapping, and even tabular numerals natively. Reach for JS only when CSS truly can't do it. Each year, that boundary moves toward CSS.

### 6. Fast CSS is small CSS
Selector cost is mostly negligible on modern engines. **What matters is bytes shipped and rendering work.** Audit your bundle; ship critical first; defer the rest.

---

## Choosing a Styling Approach

The 2026 landscape:

| Approach | What it is | Pros | Cons | Best for |
|---|---|---|---|---|
| **Vanilla CSS / SCSS** | Plain stylesheets | Universal, no runtime, full CSS power | Global scope; needs methodology | Content sites, when paired with BEM/ITCSS |
| **CSS Modules** | Scoped class names per file | Scoped, no runtime cost, simple | Naming verbose; less ergonomic in JSX | React/Vue/Svelte components |
| **Tailwind CSS** | Utility-first; classes for everything | Fast iteration; consistency by construction; tiny CSS bundle (purged) | Long classnames; opinion-heavy; learning curve | Most modern web apps in 2026 |
| **CSS-in-JS (runtime)** | Styled-components, Emotion | Dynamic theming; co-located | **Runtime cost (worst INP impact)**; SSR complexity | Legacy / specific dynamic needs |
| **CSS-in-JS (zero-runtime)** | vanilla-extract, Linaria, Panda CSS, StyleX | Type-safe tokens; co-located; no runtime | Build complexity | Type-safe design systems |
| **Headless + Tailwind** | Radix/React Aria + Tailwind | Behavior + a11y free, full style control | Not a styling system itself; style discipline still needed | Custom-designed component libraries |
| **Component libraries** | shadcn/ui, MUI, Chakra, Mantine | Pre-built, themeable | Lock-in; hard to fully customize | Speed-to-market |

### The 2026 default
**Tailwind CSS + Headless components (Radix or React Aria) + Design tokens** has become the dominant stack for new product apps. It pairs:
- Behavior + a11y from headless libraries (you don't reinvent the combobox)
- Style discipline from utility classes (no spelunking through a stylesheet)
- Tokens from CSS variables (theming, dark mode, brand variants)
- Tiny shipped CSS (purged classes only)

### When to *not* use Tailwind
- Email HTML (use inline styles)
- Marketing/blog sites that want semantic class names for copywriters
- Projects with non-engineer contributors editing styles
- Teams that have strong existing CSS practices with high consistency

### The CSS-in-JS retreat
Runtime CSS-in-JS (styled-components, Emotion) was the dominant pattern from 2017–2022. The community is leaving it because:
- **Runtime cost** — measurable INP regression on large pages
- **Streaming SSR friction** — runtime style insertion fights React 18's streaming
- **Bundle size** — the runtime ships with every page

If you're on it, stay if it works. Don't pick it for new code. **Zero-runtime alternatives** (vanilla-extract, Panda, StyleX) preserve the type-safe-tokens win without the runtime cost.

---

## CSS Methodologies

### BEM — Block, Element, Modifier
Naming convention: `.block__element--modifier`. Single-level deep; predictable; widely understood.

```html
<article class="card card--featured">
  <h2 class="card__title">Hello</h2>
  <p class="card__body card__body--lead">…</p>
</article>
```

**Strengths:** explicit, no nesting, easy to grep.
**Weaknesses:** verbose; doesn't address file organization.

### OOCSS — Object-Oriented CSS
Separate **structure** (`.btn`, `.media`) from **skin** (`.btn--primary`, `.media--inverted`). Encourages reusable shapes.

### SMACSS — Scalable & Modular Architecture for CSS
Five categories: Base, Layout, Module, State, Theme. Provides a folder structure.

### ITCSS — Inverted Triangle CSS
Layered specificity from low (settings, tools) to high (utilities). Pairs perfectly with `@layer`:
```
1. Settings  (variables, no output)
2. Tools     (mixins, no output)
3. Generic   (resets, normalize)
4. Elements  (bare HTML element styles)
5. Objects   (layout primitives, like .container, .grid)
6. Components (UI components, like .card, .button)
7. Utilities (single-purpose: .u-mt-2, .hidden)
```
Each layer can override the previous; specificity climbs as you descend.

### Atomic CSS / Utility-First (Tailwind)
Every utility = one property. The "stylesheet" becomes the markup; the CSS bundle stays tiny because you only ship classes you use.

```html
<button class="rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700 focus-visible:ring-2">
```

### CSS Modules
Hash class names per file at build time → automatic scoping. Solves global-scope without runtime.

```css
/* button.module.css */
.primary { background: blue; }
```
```jsx
import styles from './button.module.css';
<button className={styles.primary}>
```

### CUBE CSS
Composition + Utility + Block + Exception. A modern blend favoring native CSS, layers, and tokens.

> **Direct advice:** these aren't religions. **Pick one team-wide; document it; enforce it with linting.** A team using "BEM most of the time" is a team with no methodology.

---

## The Cascade, Specificity, and `@layer`

CSS resolves competing rules using:

1. **Origin & importance** — user-agent < user < author; `!important` reverses direction
2. **Cascade layers** (`@layer`) — explicit ordering of layers
3. **Specificity** — `(inline, IDs, classes/attrs/pseudo-classes, elements)`
4. **Source order** — last declaration wins among equal specificity

### Specificity (in order)
```
.btn           → 0,0,1,0
#submit        → 0,1,0,0
[type="submit"] → 0,0,1,0
:hover         → 0,0,1,0
::before       → 0,0,0,1
:is(.a, #b .c) → takes the highest of its argument
:where(...)    → always 0,0,0,0
:not(.x)       → takes specificity of the inner selector
```

### `:where()` — your friend in the cascade
Wrap selectors in `:where()` to force specificity to 0:
```css
:where(article) h2 { font-size: 1.5rem; }
/* Easy to override; encourages utility / state classes to win */
```

### `@layer` — explicit cascade ordering
The single biggest improvement to CSS architecture in years.

```css
@layer reset, base, components, utilities;

@layer reset { /* normalize */ }
@layer base { h1 { font-size: 2rem; } }
@layer components { .btn { ... } }
@layer utilities { .mt-4 { margin-top: 1rem; } }
```

Layers cascade in the order they're declared. **Utilities defined later but in an earlier layer still lose to components in a later layer.** No more specificity wars; no more `!important` arms races.

### Deprecate `!important`
With `@layer` and `:where()`, you can structure CSS so `!important` is unnecessary. If you reach for it, your architecture has a hole. **Lint against it.**

---

## Modern CSS Features You Should Be Using

CSS in 2026 is wildly more powerful than 2018. Most browsers ship these.

### `:has()` — the parent selector
The single most-anticipated CSS feature for 20 years. Style a parent based on its descendants:

```css
/* Card with an image looks different than one without */
.card:has(img) { padding: 0; }

/* Style a form group with an invalid input */
.form-group:has(:invalid) { color: red; }

/* Highlight rows containing a checked checkbox */
tr:has(input:checked) { background: yellow; }
```

### Container Queries
Style components based on their container, not the viewport:

```css
.card-container { container-type: inline-size; }

@container (min-width: 400px) {
  .card { display: flex; }
}
```

This is what media queries should always have been. Components are now truly reusable across layouts.

### Subgrid
Children inherit their parent's grid tracks. Aligned, nested layouts that were impossible:

```css
.outer { display: grid; grid-template-columns: 1fr 1fr; }
.inner { display: grid; grid-template-columns: subgrid; }
```

### `@scope` — local-first styling
Scope rules to a subtree without naming gymnastics:

```css
@scope (.card) to (.content) {
  h2 { color: blue; }   /* applies only to h2 inside .card, before .content */
}
```

### View Transitions
Native cross-element and cross-document animations:
```css
::view-transition-old(name),
::view-transition-new(name) {
  animation-duration: 0.3s;
}
```

```js
document.startViewTransition(() => updateDOM());
```

Same-document since 2023; cross-document (multi-page apps) since 2024 (Chrome). MPAs feel like SPAs without the JS.

### Cascade Nesting
Sass-style nesting in vanilla CSS:
```css
.card {
  padding: 1rem;
  & .title { font-size: 1.5rem; }
  &:hover { background: #eee; }

  @media (width >= 768px) {
    padding: 2rem;
  }
}
```

### Modern Color
- **`color-mix()`** — `color-mix(in oklch, var(--brand) 70%, white)`
- **`oklch` / `oklab`** — perceptually uniform color spaces; gradients without dead zones
- **`color()`** — wide-gamut (`display-p3`, `rec2020`)
- **Relative color syntax** — `hsl(from var(--brand) h s 95%)`
- **`color-scheme: light dark`** — opt-in to the OS theme

```css
:root {
  color-scheme: light dark;
  --bg: light-dark(white, #111);
  --text: light-dark(#111, #eee);
}
```

### Custom Properties (CSS Variables) — used well
```css
:root {
  --space: 8px;
  --space-2: calc(var(--space) * 2);
  --radius: 6px;
  --shadow-md: 0 4px 6px -1px hsl(220 20% 10% / 0.1);
}

/* Component variables */
.btn {
  --btn-bg: var(--brand-500);
  background: var(--btn-bg);
}
.btn--secondary { --btn-bg: var(--gray-200); }
```

Custom properties are dynamic, inheritable, themeable. They're the foundation of modern design tokens.

### `accent-color` & `caret-color`
Style native form controls with one line:
```css
:root { accent-color: var(--brand); caret-color: var(--brand); }
```

### Logical Properties
Direction-agnostic — work for LTR and RTL:
```css
margin-inline-start: 1rem;   /* not margin-left */
padding-block: 0.5rem;       /* top + bottom */
border-inline-end: 1px solid; /* right in LTR, left in RTL */
```
Use these by default. Your i18n team will thank you.

### Wide-gamut Colors
P3 displays show colors sRGB can't reach. Designs in 2026 increasingly target P3:
```css
.brand-color {
  color: color(display-p3 0.3 0.6 0.9);
  /* Browsers without P3 fall back to closest sRGB */
}
```

### Anchor Positioning (newer)
Native popovers, tooltips, dropdowns positioned relative to anchors without JS:
```css
.tooltip { position: absolute; position-anchor: --btn; top: anchor(bottom); }
```
Chrome 125+; ship behind progressive enhancement.

### Scroll-Driven Animations
Animations tied to scroll position, all in CSS:
```css
@keyframes reveal { from { opacity: 0; } to { opacity: 1; } }
.fade-in {
  animation: reveal linear;
  animation-timeline: view();
  animation-range: entry 0% cover 30%;
}
```

---

## Layout Systems

### Flexbox
1D layout (row OR column). Use for: navbars, button groups, item alignment within a row.

```css
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
}
```

Quick tip: `gap` works in Flexbox now (since 2021 in all browsers). Stop using margins on children.

### Grid
2D layout. Use for: page layouts, card grids, complex forms, dashboards.

```css
.layout {
  display: grid;
  grid-template-columns: minmax(0, 1fr) minmax(0, 3fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;
}
```

### When to use which
- Need to align across rows AND columns? → Grid
- Just one axis? → Flexbox
- Both work? → Either; pick by clarity
- Complex card grids → Grid with `auto-fill`/`auto-fit` + `minmax()`
- Navigation bars → Flexbox

### Modern grid recipes
```css
/* Auto-responsive grid — no media queries */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(min(300px, 100%), 1fr));
  gap: 1rem;
}

/* Holy grail layout */
.app {
  display: grid;
  grid-template:
    "header header" auto
    "nav    main  " 1fr
    "footer footer" auto / 240px 1fr;
  min-height: 100dvh;
}
```

### Don't use float for layout
Float is for floating images within text. Anything else: Grid or Flexbox.

---

## Responsive Design Beyond Media Queries

### Mobile-first (still right)
Default styles for small screens; scale up with `min-width` queries:
```css
.card { padding: 1rem; }

@media (width >= 768px) {
  .card { padding: 2rem; }
}
```

### Container queries — the new responsive
Components respond to *their container's* size, not the viewport's:
```css
.product-card { container-type: inline-size; }

@container (width >= 320px) {
  .product-card .info { grid-column: span 2; }
}
```

A card in a sidebar can render compact; the same card in a wide column renders rich. **Same component, different layouts based on context.**

### Fluid typography & spacing
Smooth scaling between breakpoints with `clamp()`:
```css
:root {
  --font-size-h1: clamp(1.75rem, 1.5rem + 1.5vw, 2.5rem);
  --space-section: clamp(2rem, 5vw, 4rem);
}
```

### Modern viewport units
- `vh` / `vw` — classic; `vh` infamously broken on iOS
- `dvh` / `dvw` — dynamic; accounts for browser UI showing/hiding
- `svh` / `svw` — small (UI showing)
- `lvh` / `lvw` — large (UI hidden)

```css
.hero { min-height: 100dvh; }  /* Replaces 100vh in 2024+ */
```

### Aspect ratio
```css
.thumbnail { aspect-ratio: 16 / 9; }
```
No more padding-bottom hacks.

### `max()` / `min()` / `clamp()`
First-class composition of responsive values:
```css
.container { width: min(100% - 2rem, 1200px); }
.heading   { font-size: clamp(1.5rem, 4vw, 3rem); }
```

---

## Design Tokens

The atomic unit of a design system. **Named values** for every visual decision: color, spacing, typography, radius, shadow, motion, breakpoints.

### The token hierarchy
```
Primitive tokens     →    Semantic tokens     →    Component tokens
(--blue-500)             (--color-brand)            (--button-bg)

Hex codes,               "What does this              "How is it used
spacing scale,           mean?"                       in this component?"
font sizes
```

```css
:root {
  /* Primitives */
  --blue-500: oklch(60% 0.2 250);
  --gray-100: oklch(95% 0 0);
  --space-3: 0.75rem;

  /* Semantic — what they mean */
  --color-brand: var(--blue-500);
  --color-surface: var(--gray-100);
  --space-card-padding: var(--space-3);

  /* Component — how they're used */
  --button-bg: var(--color-brand);
  --button-padding: var(--space-card-padding);
}
```

Three-tier tokens are the pattern that scales. Designers change `--color-brand`; every component using brand updates automatically. Components can override their own variables for variants.

### Token formats
- **CSS custom properties** — runtime, themeable, dynamic
- **JSON** — `style-dictionary` (Amazon) compiles to CSS, JS, iOS, Android
- **W3C Design Tokens Format** — emerging standard
- **Figma Variables** — design source-of-truth syncing to code via Tokens Studio, Style Dictionary

### Tools
- **Style Dictionary** (Amazon) — token compiler
- **Tokens Studio** (Figma plugin) — design ↔ code sync
- **Specify**, **Knapsack**, **Supernova** — managed design ops

---

## Theming (Light/Dark and Beyond)

### Dark mode the modern way
```css
:root {
  color-scheme: light dark;
  --bg:   light-dark(white, #111);
  --text: light-dark(#111, #eee);
}

body { background: var(--bg); color: var(--text); }
```

`light-dark()` (CSS Color 5) lets the browser pick automatically. For user override, swap based on a class:
```css
[data-theme="light"] { color-scheme: light; }
[data-theme="dark"]  { color-scheme: dark; }
```

### `prefers-color-scheme`
```css
@media (prefers-color-scheme: dark) {
  :root { --bg: #111; --text: #eee; }
}
```

### Multi-theme architecture
```css
[data-theme="light"]   { --bg: white; }
[data-theme="dark"]    { --bg: #111; }
[data-theme="sepia"]   { --bg: #f4ecd8; }
[data-theme="contrast"]{ --bg: black; --text: yellow; }
```

Themes are just different sets of token values. Add new themes without touching components.

### Avoid the FOUC on theme load
A user who picked "dark" loading "light" first is jarring. The fix:
```html
<script>
  // Inline, before any CSS, runs synchronously
  const t = localStorage.getItem('theme') || 'system';
  document.documentElement.dataset.theme = t === 'system'
    ? (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light')
    : t;
</script>
```

### `prefers-contrast`, `prefers-reduced-data`, `forced-colors`
Modern users have many preferences:
- `prefers-contrast: more` — bump contrast for visibility
- `prefers-reduced-motion: reduce` — disable animations
- `prefers-reduced-data: reduce` — skip heavy assets
- `forced-colors: active` — Windows High Contrast mode; let user colors win

Respect them all.

---

## Design Systems

A **design system** is the systemized rules + components + documentation that let many teams ship a coherent product.

```
┌────────────────────────────────────────────────────────────────┐
│                   DESIGN SYSTEM ANATOMY                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  Foundations                                                   │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ Tokens · Typography · Color · Spacing · Motion · Iconography│
│  └──────────────────────────────────────────────────────┘      │
│                       ▼                                        │
│  Primitives                                                    │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ Button · Input · Checkbox · Badge · Avatar · Icon    │      │
│  └──────────────────────────────────────────────────────┘      │
│                       ▼                                        │
│  Composed components                                           │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ Card · Form · Dialog · Menu · Toast · DataTable      │      │
│  └──────────────────────────────────────────────────────┘      │
│                       ▼                                        │
│  Patterns / templates                                          │
│  ┌──────────────────────────────────────────────────────┐      │
│  │ Empty states · Login · Pagination · Filter sidebars  │      │
│  └──────────────────────────────────────────────────────┘      │
│                       ▼                                        │
│  Documentation, governance, contributions                      │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Why design systems
- **Consistency at scale** — same login, same modal, same date picker across 50 teams
- **Speed** — teams compose, don't recreate
- **Quality** — accessibility, performance, edge cases solved once
- **Brand discipline** — the brand survives 10× growth

### Build vs. buy
- **Build**: when you have a strong brand identity, multiple products, and ≥10 frontend engineers. Investment pays back in years.
- **Adopt** open-source: shadcn/ui (copy components into your repo), Radix UI / React Aria (headless behavior + a11y) — and add your tokens.
- **Buy** (MUI, Mantine, Chakra, Ant): fastest start; hardest to escape.

> **Worth knowing (2026):** the **shadcn/ui pattern** — copy headless components into your repo, customize with Tailwind + tokens — has become dominant. You own the code; you can change anything; no library upgrade pain.

### Components: behavior vs. style
Behavior + a11y is hard (focus traps, keyboard navigation, ARIA, edge cases). **Don't reimplement it.** Use:
- **Radix UI primitives** (React)
- **React Aria** (Adobe; framework-agnostic adapter via React Aria Components)
- **Headless UI** (Tailwind Labs)
- **Reach UI** (older but solid)

Style on top with tokens. You get accessibility for free; you control every pixel.

### Documentation
- **Storybook** is the de facto choice
- Document: usage, props, accessibility, do's & don'ts, tokens used
- **Real interaction examples** — not just static screenshots
- **Versioning** — track breaking changes; provide codemods

### Governance
A design system without governance becomes 12 inconsistent forks. Set:
- **Contribution model** — RFC for new components; centralized review
- **Component lifecycle** — proposed → in-progress → stable → deprecated
- **Breaking change policy** — major version bumps, deprecation periods, codemods
- **Adoption metrics** — % of components from system vs custom

### Famous examples to study
- **GitHub Primer** — open-source, well-documented
- **Atlassian Design System**
- **Adobe Spectrum** + **React Aria**
- **Material Design** — Google
- **Carbon** — IBM
- **Polaris** — Shopify
- **shadcn/ui** — modern utility-friendly approach

---

## Critical CSS

The CSS needed to render the **above-the-fold** content. Inlining it in `<head>` removes the render-blocking external CSS request for first paint.

### Why
A page with one external `<link rel="stylesheet">` can't paint until the CSS is downloaded and parsed. On a slow connection, that's hundreds of milliseconds — maybe seconds — of blank screen. **Critical CSS slashes LCP.**

### How
```html
<head>
  <style>/* Critical CSS — inlined, ~5–14KB ideally */</style>
  <link
    rel="stylesheet"
    href="/main.css"
    media="print"
    onload="this.media='all'"
  >
  <noscript><link rel="stylesheet" href="/main.css"></noscript>
</head>
```

The `media="print"` trick loads the stylesheet without blocking render; `onload` switches it to `all` after load. Modern alternative: `<link rel="preload" as="style">` + a switch on `onload`.

### Extraction
- **Critters** — Webpack plugin; integrated into Next.js
- **Penthouse** — by Pocket; standalone
- **Beasties** — Next.js / Astro; refined extractor
- **Critical** — Addy Osmani's tool

These tools render the page in a headless browser, find which CSS rules are used above-the-fold, and inline them.

### When critical CSS is overkill
- The whole CSS bundle is < 14KB → just inline all of it
- Already on HTTP/2/3 with a small CSS bundle → marginal gain
- Heavily-cached pages (returning users) → first-load critical CSS isn't reused

### When critical CSS is essential
- Marketing landing pages (cold first-load is the SEO + PSI metric)
- Public content sites with lots of visitors per session
- Users on slow networks (mobile, emerging markets)

### Beware: critical CSS staleness
Critical CSS extraction must run on every deploy. Out-of-date critical CSS shows the page un-styled until the stylesheet loads — worse than no critical CSS. Bake it into the build.

### Single-file critical inline (modern alternative)
```css
@import url('full.css') screen and (min-width: 0);
```
Or just split your CSS into critical + non-critical:
```html
<link rel="stylesheet" href="/critical.css">     <!-- small, blocking -->
<link rel="preload" as="style" href="/rest.css" onload="...">
```

---

## CSS Performance

### Selector cost is mostly a myth
"Avoid descendant selectors" used to matter. Modern engines run selectors **right-to-left**, hash by class, and are wildly fast. Don't optimize selectors prematurely. **Worry about bytes shipped and rendering work.**

### What actually matters
1. **Total CSS shipped** — purge unused (Tailwind's `content` config; PurgeCSS for legacy)
2. **Render-blocking** — minimize via critical CSS
3. **Layout-causing properties** — animating `width`, `height`, `top` triggers reflow
4. **Paint area** — large layers cost more to repaint
5. **Compositing** — promote rarely-changing layers; don't promote everything

### What triggers what
| Change | Pipeline |
|---|---|
| `transform`, `opacity` | Composite only — cheapest |
| `color`, `background-color`, `box-shadow` | Paint |
| `width`, `height`, `top`, `left`, `font-size` | Layout (most expensive) |

### `will-change` — handle with care
```css
.menu { will-change: transform; }
```
Tells the browser "I'm about to animate transform" — promotes the element to its own layer ahead of time. **Don't apply broadly** — each layer costs GPU memory.

### `content-visibility: auto`
Skip rendering off-screen elements until they scroll into view:
```css
.card { content-visibility: auto; contain-intrinsic-size: auto 200px; }
```
Real performance wins on long pages. Pair with `contain-intrinsic-size` to reserve space.

### `contain` — promise the browser
```css
.widget { contain: layout paint; }
```
Tells the engine "changes inside this element don't affect the outside." Enables aggressive optimization.

### Avoid expensive properties
- `box-shadow` with large blur on many elements → paints expensively
- `filter: blur()` over a large area → very expensive
- `backdrop-filter` → expensive but often worth it for the effect
- `border-radius` on huge elements with shadows → expensive

### CSS bundle splitting
Split CSS by route, like JS:
- `core.css` — tokens, base, layout primitives
- `routes/dashboard.css` — page-specific
- `components/heavy-chart.css` — lazy-loaded

Most frameworks (Next.js, Vite) handle this for CSS Modules and Tailwind automatically.

---

## Animation & Motion

### CSS animations vs. JS animations
| | CSS | JS (Web Animations API / GSAP / Framer Motion) |
|---|---|---|
| **Performance** | GPU-accelerated for transform/opacity; smooth | Same when using transform/opacity |
| **Complexity** | Limited orchestration | Sequencing, springs, gestures |
| **Best for** | Hover, simple transitions, scroll-driven | Complex sequences, physics, drag |

For most needs, CSS is enough. For complex motion (drag, springs, choreographed sequences), **Framer Motion** or the **Web Animations API** with **GSAP** wins.

### View Transitions API
Native cross-element animations on state change. Same-document and cross-document.

```js
document.startViewTransition(() => updateDOM());
```
```css
.product { view-transition-name: var(--product-id); }
```
Click a product card; it morphs into the detail page hero. No router-aware JS animation library needed.

### `prefers-reduced-motion`
Respect users with motion sensitivity:
```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### Motion design principles
- **Easing matters** — linear is rare in nature; use easing functions (`cubic-bezier`, the new `linear()` for custom curves)
- **Duration**: 150–300ms for most UI; longer feels sluggish, shorter feels jarring
- **Stagger** lists for hierarchy
- **Anticipation + follow-through** — overshoot, settle (springs)
- **Less is more** — animation should clarify, not distract

---

## Accessibility in CSS

### Visible focus indicators
Never `outline: none` without a replacement. The default outline is plain but accessible — replace with something at least as clear.

```css
*:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
  border-radius: 2px;
}
```

`:focus-visible` shows the ring only on keyboard focus, not mouse clicks — the modern default.

### Hit targets
WCAG 2.2 (2.5.8) requires interactive targets ≥ 24×24 CSS px. Apple HIG recommends 44×44. Pad clickable elements; avoid tightly packed icon-only buttons.

### Contrast
WCAG AA: 4.5:1 for normal text, 3:1 for large text and UI components. Test with browser DevTools, Stark, or `oklch()` lightness math.

### Color isn't the only signal
Errors must communicate beyond red — icons, text, patterns. Pair color with a redundant cue.

### Hidden vs. visible to screen readers
| Technique | Visual | Screen reader | Focusable |
|---|---|---|---|
| `display: none` | Hidden | Hidden | No |
| `visibility: hidden` | Hidden | Hidden | No |
| `hidden` attribute | Hidden | Hidden | No |
| `aria-hidden="true"` | Visible | Hidden | **Still yes — bug** |
| `.sr-only` (clip-path) | Hidden | **Read** | Yes (when focusable) |
| `inert` | Visible | Hidden | No |

```css
/* Visually-hidden, screen-reader accessible */
.sr-only {
  position: absolute;
  width: 1px; height: 1px;
  padding: 0; margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
}
```

### `forced-colors` (Windows High Contrast)
Some users replace colors with system-defined high-contrast palettes. Don't fight it:
```css
@media (forced-colors: active) {
  .icon { forced-color-adjust: auto; }
  /* Use system colors: CanvasText, ButtonFace, LinkText */
}
```

### See [Web Accessibility](web-accessibility.md) for the full a11y picture.

---

## Internationalization & Logical Properties

Every property has a logical equivalent. Use them.

| Physical | Logical |
|---|---|
| `margin-left` | `margin-inline-start` |
| `padding-right` | `padding-inline-end` |
| `border-top` | `border-block-start` |
| `text-align: left` | `text-align: start` |
| `width` | `inline-size` |
| `height` | `block-size` |

```css
/* RTL-friendly by default */
.card {
  padding-inline: 1rem;
  padding-block: 0.5rem;
  border-inline-start: 4px solid var(--accent);
}
```

### `dir="rtl"` and the cascade
Set `dir="rtl"` on `<html>`; logical properties flip automatically; physical ones don't. **Default to logical properties everywhere** to avoid future RTL pain.

### Locale-aware numbers, dates
CSS can't do this; use `Intl` APIs in JS. But:
```css
.numbers {
  font-variant-numeric: tabular-nums;  /* monospaced digits for tables */
  font-feature-settings: "tnum";
}
```

---

## Forms & Print Styles

### Forms
Style native form controls (mostly):
```css
input, button, select, textarea {
  font: inherit;             /* don't inherit Times New Roman */
  color: inherit;
}

input[type="text"], input[type="email"], textarea {
  padding-block: 0.5rem;
  padding-inline: 0.75rem;
  border: 1px solid var(--border);
  border-radius: var(--radius);
}

input:focus-visible {
  outline: 2px solid var(--brand);
  outline-offset: 2px;
}

input:invalid:not(:focus) { border-color: var(--error); }

/* Modern validation styling */
input:user-invalid { border-color: var(--error); }   /* Only after interaction */
```

**`:user-invalid`** — only fires after the user has interacted (better UX than `:invalid` firing immediately on empty inputs).

### Print
Many sites still get printed (invoices, articles, recipes, tickets):
```css
@media print {
  nav, footer, .sidebar, .ads { display: none; }
  body { font-size: 11pt; color: black; background: white; }
  a::after { content: " (" attr(href) ")"; font-size: 0.85em; }
  h1, h2, h3 { break-after: avoid; }
  pre, blockquote { break-inside: avoid; }
}

@page { margin: 1.5cm; }
```

`break-inside: avoid` and friends control pagination. `@page` controls printer margins.

---

## CSS Tooling

### Preprocessors (still relevant?)
- **Sass / SCSS** — was dominant. Modern CSS has variables, nesting, math (`calc`), so Sass is less essential. Still useful for mixins, control flow, build-time logic.
- **PostCSS** — pluggable; powers Tailwind, autoprefixer, modern preset compilers

### Tailwind ecosystem
- **`tailwindcss`** — utilities
- **`@tailwindcss/forms`**, **`@tailwindcss/typography`** — official plugins
- **`prettier-plugin-tailwindcss`** — auto-sorts classes
- **`clsx` / `cn`** — conditional class composition
- **`tailwind-variants`** / **`class-variance-authority` (CVA)** — typed component variants

### Linters
- **Stylelint** — CSS linter; rules for ordering, naming, anti-patterns (no `!important`, no IDs, etc.)
- **eslint-plugin-tailwindcss** — Tailwind class ordering / unknown classes

### Transformers
- **autoprefixer** — vendor prefixes (less needed in 2026)
- **lightningcss** — Rust CSS transformer; modern target compilation; very fast
- **PostCSS Preset Env** — use future CSS today

### Build integration
- Vite / Webpack / Next.js handle CSS modules, Tailwind, PostCSS, Sass out of the box
- **CSS imports are tree-shaken** in modern bundlers if you use named exports / module-level imports

---

## Common Anti-Patterns

### 1. `!important` in source code (not utilities)
A symptom of broken specificity. Use `@layer` and `:where()` to structure cascade; `!important` should be near-zero.

### 2. Deep nesting
```css
/* ❌ */
.app .layout .sidebar .menu .item .link.active { ... }
```
Specificity climbs; refactoring breaks. Use BEM modifiers or utility classes.

### 3. Magic numbers
```css
margin-top: 17px;
```
Where does 17 come from? Tokens replace magic numbers with semantic spacing.

### 4. ID selectors for styling
IDs have specificity 100x higher than classes. Use classes; reserve IDs for fragments / labels / programmatic targets.

### 5. Inline styles for theming
`style="color: red"` defeats your design system. Use classes; tokens in CSS variables let you keep dynamic values without inline styles.

### 6. Animating layout properties
`transition: top 200ms`. Triggers layout per frame — janky. Use `transform: translateY(...)` instead.

### 7. `position: fixed` everywhere
Sticky headers, sticky sidebars, sticky CTAs all stacking → users see no content. Limit fixed elements; use `position: sticky` where appropriate.

### 8. Shadowing native elements
Building a custom checkbox from `<div>`s. You'll lose keyboard, screen reader, indeterminate state, autofill, browser theming. Style the native one with `accent-color` or use a headless library.

### 9. Setting font sizes in `px`
Users override default font size for accessibility; px ignores it. Use `rem` for typography; `em` for relative sizing; `px` for borders and 1px-precision details.

### 10. Forgetting RTL / i18n
Hardcoded `padding-left`, `text-align: left`, arrows pointing right. Default to logical properties; test with `dir="rtl"`.

### 11. Mixing methodologies
BEM here, atomic there, CSS-in-JS over there. Cognitive load × 3; new engineers can't predict where to add a style. Pick one.

### 12. Unscoped global styles in components
`button { ... }` in a component file affects every button on the page. Scope with classes / modules / Tailwind utilities.

### 13. Disabling focus styles
`outline: none` without replacement. Keyboard users can't see what they've focused. Use `:focus-visible` to keep mouse clean and keyboard accessible.

### 14. CSS-in-JS at runtime in 2026
A measurable INP regression on large pages, broken streaming SSR. Move to zero-runtime alternatives (vanilla-extract, Panda, StyleX) or utility-first (Tailwind).

### 15. Pixel-perfect over robust
Designs aren't specs — they're guides. CSS that breaks when copy is 5% longer is brittle. Build for content variance.

---

## Quick Reference Checklist

### Foundation
- [ ] One styling methodology team-wide; documented; linted
- [ ] Cascade layers (`@layer`) used for predictable specificity
- [ ] No `!important` outside utilities
- [ ] Logical properties used by default (i18n-ready)
- [ ] Modern color (oklch / display-p3) where it improves design

### Tokens
- [ ] Three-tier tokens (primitive → semantic → component)
- [ ] CSS custom properties as runtime tokens
- [ ] Style Dictionary or equivalent for cross-platform export
- [ ] Designer ↔ engineer sync workflow

### Components
- [ ] Behavior + a11y from headless library (Radix / React Aria / Headless UI)
- [ ] Tokens drive all visual properties
- [ ] All interactive elements have hover, focus-visible, active, disabled states
- [ ] Stories in Storybook with interaction + a11y addons
- [ ] Visual regression (Chromatic) on stories

### Theming
- [ ] `color-scheme` declared; `light-dark()` used
- [ ] Theme switch persists; no FOUC on load
- [ ] `prefers-color-scheme`, `prefers-contrast`, `prefers-reduced-motion` respected
- [ ] `forced-colors` mode tested

### Performance
- [ ] CSS bundle audited and purged
- [ ] Critical CSS extracted + inlined for marketing/landing
- [ ] Non-critical CSS deferred via preload pattern
- [ ] Animations on `transform` / `opacity` only
- [ ] `content-visibility` / `contain` used on heavy off-screen sections

### Accessibility
- [ ] `:focus-visible` styles on every interactive element
- [ ] Touch targets ≥ 24×24 (WCAG 2.2)
- [ ] Contrast 4.5:1 (AA) — automated test in CI
- [ ] Color paired with non-color cues for errors/states
- [ ] `.sr-only` utility for visually-hidden screen-reader text

### Layout
- [ ] Container queries used for true component-level responsiveness
- [ ] `min(100% - X, max-width)` for fluid containers
- [ ] `dvh` instead of `vh` for full-viewport
- [ ] Subgrid where layouts must align across nesting

### Process
- [ ] Stylelint in CI; fails PR on violations
- [ ] Tailwind class sorting via Prettier plugin
- [ ] Storybook published; design system has changelog
- [ ] Adoption metrics tracked (% of components from system)
- [ ] Design tokens versioned + released

---

## Further Reading

### Books
- **Andy Bell & Heydon Pickering — *Every Layout*** (online book) — composable layout primitives
- **Andrew Clarke — *Hardboiled Web Design***
- **Lea Verou — *CSS Secrets*** — clever CSS techniques (still relevant)
- **Eric Meyer & Estelle Weyl — *CSS: The Definitive Guide* (5th ed.)** — encyclopedic reference
- **Brad Frost — *Atomic Design*** (free online) — design system thinking

### Modern blogs / sites
- **[web.dev — Learn CSS](https://web.dev/learn/css)** — Una Kravets et al.; modern, comprehensive
- **[CSS-Tricks](https://css-tricks.com/)** — still going; recipes & patterns
- **[smashingmagazine.com](https://www.smashingmagazine.com/category/css/)** — annual front-end performance checklists
- **[ishadeed.com](https://ishadeed.com/)** — Ahmad Shadeed; deep dives with great visuals
- **[piccalil.li](https://piccalil.li/)** — Andy Bell; design tokens, modern CSS, CUBE CSS
- **[12daysofweb.dev](https://12daysofweb.dev/)** — annual modern features highlight

### Design systems
- **[Inclusive Components](https://inclusive-components.design/)** — Heydon Pickering
- **[Design Systems Repo](https://designsystemsrepo.com/)** — gallery of public design systems
- **[Awesome Design Systems](https://github.com/alexpate/awesome-design-systems)**
- **GitHub Primer**, **Atlassian Design System**, **Adobe Spectrum**, **Material Design**, **Carbon (IBM)**, **Polaris (Shopify)**, **shadcn/ui**

### People to follow
- Una Kravets (Google CSS), Adam Argyle (Google CSS), Lea Verou (W3C, color), Heydon Pickering (a11y), Andy Bell (modern CSS), Ahmad Shadeed, Sara Soueidan, Stephanie Eckles, Bramus Van Damme, Miriam Suzanne (cascade layers, container queries author), Brad Frost (atomic design)

### Standards / specs
- **CSS Working Group** — W3C
- **caniuse.com** — feature support
- **MDN CSS reference** — the canonical reference

### Tools
- **Storybook**, **Chromatic**
- **Style Dictionary**, **Tokens Studio**
- **Tailwind CSS**, **CVA**, **tailwind-variants**
- **Stylelint**, **Lightning CSS**
- **PostCSS**

---

> **Closing thought:** CSS is the only language in the web stack that runs everywhere — every browser, every device, every assistive tech, every locale, every theme — without you compiling for it. That power comes with sprawl: a stylesheet touches every pixel of every page. The CSS I'm proudest of is always the CSS I didn't write — replaced by a token, a layer, or a modern feature that does the job in two lines instead of fifty. Commit to one methodology, lean on tokens, embrace the cascade with `@layer`, and reach for the platform before the framework. Do that and you get a design system that scales without rotting, components that survive content variance, and pages that feel coherent from the homepage to the deepest admin screen. **Good CSS is invisible; it just feels right.**
