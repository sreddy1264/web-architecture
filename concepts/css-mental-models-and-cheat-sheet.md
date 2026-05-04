# CSS Mental Models & Cheat Sheet

*Last reviewed: 2026-05*

> A working architect's reference for the CSS *language* — selectors, the box model, layout primitives, modern features — and the mental models (cascade, specificity, formatting contexts, stacking contexts) that explain why it behaves the way it does.

This guide is the language-level companion to [CSS & Design Systems](css-and-design-systems.md), which covers architecture, tokens, theming, `@layer` strategy, and design-system patterns. Read this one to write CSS; read that one to organize it.

---

## Why this matters

Most CSS bugs trace back to four mental models: the cascade, specificity, formatting contexts, and stacking contexts. The properties are easy to look up; the underlying rules are what determine why two declarations that look right next to each other don't both apply, or why an absolutely positioned element keeps placing itself somewhere you didn't expect.

In practice: a developer who knows `flex` and `grid` cold but can't tell you what creates a stacking context will spend their afternoon hunting a `z-index` bug. The cheat sheet keeps the syntax close; the mental models keep the behavior predictable.

---

## Mindset

- **The cascade is a pipeline, not a collision.** Origin → layer → specificity → order. Knowing *which* stage decides the winner saves hours of overrides.
- **Composition over override.** Add a class, don't pile on `!important`. Use `@layer` and utility composition; don't fight the cascade.
- **Logical properties by default.** `padding-inline`, `margin-block`, `inset` — they translate for RTL, vertical writing modes, and future you.
- **Layout is a primitive choice.** Reach for Grid for two-dimensional layout, Flex for one-dimensional. Don't fight either by re-implementing the other.
- **Animate cheap properties.** `transform` and `opacity` are GPU-composited; `top`/`left`/`width` trigger layout. Pick the cheap path.
- **Media queries describe environment; container queries describe component.** Use container queries when the component should react to *its slot*, not the viewport.
- **System color scheme is data.** `prefers-color-scheme`, `prefers-reduced-motion`, `prefers-contrast` are inputs; honor them before opinions.

---

## The cheat sheet

### Selectors

```css
/* Basics */
*               /* universal */
.class
#id
elem
elem.class
elem,elem2      /* selector list */

/* Combinators */
A B             /* descendant */
A > B           /* direct child */
A + B           /* adjacent sibling */
A ~ B           /* general sibling */

/* Attribute */
[type]
[type="text"]
[href^="https"]   /* starts with */
[href$=".pdf"]    /* ends with */
[class*="card"]   /* contains substring */
[lang|="en"]      /* exact or starts with "en-" */
[data-state="open" i]  /* case-insensitive */

/* Pseudo-classes — state */
:hover :active :focus :focus-visible :focus-within
:disabled :enabled :checked :indeterminate :default
:required :optional :valid :invalid
:user-valid :user-invalid    /* only after interaction */
:placeholder-shown :read-only :read-write
:target :defined

/* Pseudo-classes — structural */
:root :empty :has(...)
:first-child :last-child :only-child
:first-of-type :last-of-type :only-of-type
:nth-child(2n+1) :nth-last-child(...) :nth-of-type(...)

/* Logical / functional */
:is(h1, h2, h3)         /* matches any; takes max specificity inside */
:where(h1, h2, h3)      /* same, but specificity is 0 — perfect for resets */
:not(.disabled)         /* negation */
:has(> img)             /* parent selector — the long-awaited one */

/* Pseudo-elements */
::before ::after
::first-line ::first-letter
::placeholder ::selection ::marker
::file-selector-button ::backdrop
::details-content        /* for <details> */
::cue                    /* WebVTT captions */
```

Worth knowing: `:has()` is the parent selector. `.card:has(img)` styles cards that contain an image. Use it sparingly — it can defeat browser style-engine optimizations on big trees.

### Specificity at a glance

| Selector kind | Weight |
|---|---|
| Inline `style=""` | 1,0,0,0 |
| `#id` | 0,1,0,0 |
| `.class`, `[attr]`, `:pseudo-class` | 0,0,1,0 |
| `elem`, `::pseudo-element` | 0,0,0,1 |
| `*`, combinators, `:where(...)` | 0,0,0,0 |
| `!important` | wins origin/layer fight, then by specificity |

`:is()` and `:not()` take the **maximum** specificity of their argument list. `:where()` is always 0. Use `:where()` in resets and base layers so consumers can override without an arms race.

### Box model

```css
.box {
  box-sizing: border-box;     /* width includes padding + border */
  width: 240px;
  padding: 1rem;
  border: 2px solid;
  margin: 1rem;
}

/* Logical equivalents — preferred */
.box {
  padding-block: 1rem;        /* top + bottom in horizontal-tb */
  padding-inline: 1.5rem;     /* left + right */
  margin-block: 1rem;
  border-inline-start: 2px solid;
}
```

A reframe: `box-sizing: border-box` should be a global default. Most teams set it on `*, *::before, *::after`.

### Display

```css
display: block;
display: inline;
display: inline-block;
display: flex;
display: inline-flex;
display: grid;
display: inline-grid;
display: contents;     /* the element itself disappears, children promote */
display: flow-root;    /* establishes a new BFC; clears floats; isolates margins */
display: none;         /* removed from flow + AT */
```

`display: contents` is great for letting a wrapper participate in the parent's grid/flex without consuming a track. `display: flow-root` is the modern way to clear floats and contain margin collapse — no clearfix hack needed.

### Position

```css
position: static;       /* default */
position: relative;     /* offsets via top/right/bottom/left, no escape from flow */
position: absolute;     /* removed from flow; positioned to nearest positioned ancestor */
position: fixed;        /* positioned to the viewport (unless transformed ancestor) */
position: sticky;       /* relative until threshold, then fixed-within-container */
```

```css
/* Sticky needs an ancestor with overflow: visible (or none) and a threshold */
.toc {
  position: sticky;
  top: 1rem;
}

/* Logical positioning */
.tooltip {
  position: absolute;
  inset-block-start: 100%;
  inset-inline-start: 0;
}
```

A caveat: `position: fixed` is positioned relative to the viewport *unless* an ancestor has `transform`, `filter`, or `will-change: transform` set — then that ancestor becomes the containing block. Most "fixed isn't fixed" bugs are this.

### Flexbox

```css
.row {
  display: flex;
  flex-direction: row | row-reverse | column | column-reverse;
  flex-wrap: nowrap | wrap | wrap-reverse;
  gap: 1rem;                 /* shorthand: row-gap column-gap */

  justify-content: flex-start | center | space-between | space-around | space-evenly;
  align-items: stretch | flex-start | center | flex-end | baseline;
  align-content: ...;        /* multi-line cross-axis distribution */
}

.item {
  flex: 1 1 auto;            /* grow shrink basis */
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: 200px;
  align-self: center;
  order: 2;
}
```

| Shorthand | Means |
|---|---|
| `flex: 1` | `1 1 0%` — fill space evenly |
| `flex: auto` | `1 1 auto` — fill, but respect content |
| `flex: none` | `0 0 auto` — don't grow or shrink |
| `flex: 0 1 200px` | Don't grow, may shrink, start at 200px |

Worth knowing: `gap` works in flex (not just grid) — drop `margin` hacks for spacing children.

### Grid

```css
.grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-template-rows: auto 1fr auto;
  gap: 1rem;

  /* Named areas */
  grid-template-areas:
    "header header header"
    "side   main   aside"
    "footer footer footer";
}

.header { grid-area: header; }
.main   { grid-area: main; }

/* Auto-fit / auto-fill — responsive grids */
.cards {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(16rem, 1fr));
  gap: 1rem;
}

/* Track placement on items */
.item {
  grid-column: 1 / 3;        /* span columns 1→3 */
  grid-column: span 2;
  grid-row: 2 / -1;          /* second row to last */
  justify-self: center;
  align-self: end;
}

/* Subgrid — children inherit parent's tracks */
.row {
  display: grid;
  grid-template-columns: subgrid;
  grid-column: 1 / -1;
}
```

A reframe: Grid is for two-dimensional layout (rows *and* columns matter together). Flex is for one dimension at a time. Combining them is normal — Grid for the page, Flex for the toolbar.

### Units

| Unit | Means |
|---|---|
| `px` | Absolute pixels (CSS pixels, not device) |
| `rem` | Root `font-size` — the safe scaling unit |
| `em` | Parent `font-size` (or own, for `line-height`) |
| `%` | Percentage of containing block (varies by property) |
| `vw` / `vh` | 1% of viewport width/height |
| `svh` / `lvh` / `dvh` | Small / large / dynamic viewport height (mobile address bar aware) |
| `svw` / `lvw` / `dvw` | Same for width |
| `vmin` / `vmax` | Smaller/larger of viewport dims |
| `ch` | Width of "0" glyph — great for text-column widths |
| `ex` / `cap` / `ic` | Font-metric units |
| `cqw` / `cqh` / `cqi` / `cqb` / `cqmin` / `cqmax` | Container-query units |
| `fr` | Fractional unit (Grid only) |

Lesson learned: use `dvh` for full-screen layouts on mobile — `100vh` overshoots when the address bar is visible. Use `svh` to get the *smallest* possible viewport height.

### Colors

```css
/* Classic */
color: #ff8800;
color: rgb(255 136 0);
color: rgb(255 136 0 / 0.5);     /* alpha via slash */
color: hsl(30 100% 50%);

/* Modern, perceptually uniform */
color: oklch(70% 0.18 50);       /* lightness chroma hue */
color: oklab(70% 0.1 0.05);
color: lab(70% 30 40);
color: lch(70% 50 40);

/* Color functions */
color: color-mix(in oklch, var(--brand) 70%, white);
color: light-dark(black, white); /* respects color-scheme */

/* Wide gamut */
color: color(display-p3 1 0 0);

/* Currentcolor — the magic keyword */
border: 1px solid currentColor;
```

Worth knowing: `oklch` solves the perceptual-uniformity problem of `hsl` — equal numerical changes look like equal perceptual changes. Use it for design tokens when you can.

### Typography

```css
.body {
  font: 400 1rem/1.5 system-ui, -apple-system, sans-serif;
  /*    weight size/line-height family */

  letter-spacing: -0.01em;
  word-spacing: 0;
  text-wrap: balance;        /* headlines */
  text-wrap: pretty;         /* paragraphs */
  hyphens: auto;
  hanging-punctuation: first allow-end last;
  text-rendering: optimizeLegibility;
  font-feature-settings: "ss01", "tnum";
  font-variant-numeric: tabular-nums;
  font-optical-sizing: auto;
  font-synthesis: none;      /* don't fake bold/italic */
}
```

A caveat: `text-wrap: balance` is great for headlines (typically up to ~6 lines). For body copy, use `text-wrap: pretty`, which balances only the last few lines and is much cheaper.

### Backgrounds, borders, gradients, shadows

```css
.card {
  background:
    linear-gradient(to bottom, #fff, #f5f5f5)
    fixed
    no-repeat;
  background-image: url(img.jpg), linear-gradient(...);  /* layered */
  background-size: cover;
  background-clip: padding-box;
  background-origin: border-box;

  border: 1px solid;
  border-radius: 12px;
  border-image: linear-gradient(...) 1;

  box-shadow:
    0 1px 0 hsl(0 0% 0% / 0.04),
    0 4px 16px hsl(0 0% 0% / 0.08);
}

/* Conic + radial */
background: conic-gradient(from 0deg, red, yellow, lime, blue, red);
background: radial-gradient(circle at 30% 30%, white, black);

/* Inset / outset shadows */
box-shadow: inset 0 0 0 1px var(--ring);
```

### Transforms

```css
.thing {
  transform: translate(10px, 20px) rotate(15deg) scale(1.05);
  transform-origin: top left;
  transform-style: preserve-3d;
  perspective: 800px;
  backface-visibility: hidden;
}

/* Modern individual transforms — easier to compose */
.thing {
  translate: 10px 20px;
  rotate: 15deg;
  scale: 1.05;
}
```

In practice: `translate`/`rotate`/`scale` as standalone properties compose better than the `transform` shorthand — you can transition them independently.

### Transitions & animations

```css
.button {
  transition:
    background-color 200ms ease,
    transform 150ms ease-out;
}

@keyframes pulse {
  from { transform: scale(1); }
  50%  { transform: scale(1.05); }
  to   { transform: scale(1); }
}

.dot {
  animation: pulse 1.6s ease-in-out infinite;
}

/* Scroll-driven animations (2024+) */
.indicator {
  animation: progress linear;
  animation-timeline: scroll(root);
}

@keyframes progress {
  to { scale: 1 1; }
}

/* Honor reduced motion */
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

Worth knowing: easing is what separates "professional" from "twitchy" animation. Default `ease` is fine; explicit cubic-béziers (`cubic-bezier(.2, .8, .2, 1)`) feel even better.

### Media queries

```css
@media (min-width: 60rem) { ... }
@media (max-width: 40rem) { ... }
@media (60rem <= width <= 100rem) { ... }   /* range syntax */

/* Capability and preference */
@media (hover: hover) and (pointer: fine) { ... }
@media (prefers-color-scheme: dark) { ... }
@media (prefers-reduced-motion: reduce) { ... }
@media (prefers-contrast: more) { ... }
@media (prefers-reduced-transparency: reduce) { ... }
@media (forced-colors: active) { ... }
@media (display-mode: standalone) { ... }    /* installed PWA */
@media (orientation: landscape) { ... }
```

### Container queries

```css
.card-host {
  container-type: inline-size;       /* opt in */
  container-name: card;
}

@container card (width > 30rem) {
  .card { display: grid; grid-template-columns: 1fr 2fr; }
}

/* Container query units */
.title { font-size: clamp(1rem, 4cqw, 2rem); }
```

A reframe: media queries describe the *page environment*; container queries describe the *component's slot*. The same component placed in a sidebar and a main column should style itself based on its slot, not on the viewport.

### `z-index` & stacking contexts

A new stacking context is created by *any* of:

- `position: relative | absolute | fixed | sticky` with `z-index` set
- `opacity < 1`
- `transform`, `filter`, `perspective`, `clip-path`, `mask` (not `none`)
- `will-change: transform` (or any GPU-promoting property)
- `isolation: isolate`
- `contain: paint | layout | strict | content`
- The root element

Inside a stacking context, `z-index` is local — a child with `z-index: 9999` cannot escape its parent's stacking context to sit above an unrelated sibling.

```css
/* When you want to "promote" a layer without other side effects */
.modal-host { isolation: isolate; }
```

### Logical properties

| Physical | Logical |
|---|---|
| `margin-top` | `margin-block-start` |
| `margin-bottom` | `margin-block-end` |
| `margin-left` | `margin-inline-start` |
| `margin-right` | `margin-inline-end` |
| `padding-top` | `padding-block-start` |
| `width` | `inline-size` |
| `height` | `block-size` |
| `top`, `left` | `inset-block-start`, `inset-inline-start` |
| `text-align: left` | `text-align: start` |

```css
.callout {
  padding-block: 1rem;
  padding-inline: 1.5rem;
  margin-block: 2rem;
  inset-inline-start: 0;
}
```

### Functions

```css
width: calc(100% - 2rem);
font-size: clamp(1rem, 2.5vw, 1.5rem);   /* min, preferred, max */
gap: max(1rem, 2vw);
gap: min(2rem, 5vw);
color: var(--brand, dodgerblue);          /* with fallback */
content: attr(data-tooltip);              /* read attribute */
background: color-mix(in oklch, var(--brand) 70%, white);
```

`clamp(min, preferred, max)` is the workhorse for fluid type and spacing.

### Overflow & scrolling

```css
.wrap {
  overflow: auto;
  overflow-x: hidden;
  overflow-y: auto;
  overflow: clip;                  /* like hidden, but no scroll port */

  scroll-behavior: smooth;
  overscroll-behavior: contain;    /* stop the bounce-through to body */
  scrollbar-gutter: stable both-edges;
}

/* Scroll-snap */
.gallery {
  scroll-snap-type: x mandatory;
  overflow-x: auto;
}
.gallery > * { scroll-snap-align: start; }
```

A caveat: `overscroll-behavior: contain` on modals/sidebars stops the scroll chain from reaching the document — the difference between feeling crisp and feeling broken on touch devices.

### Filters & blend modes

```css
.thumb {
  filter: blur(2px) saturate(1.2) drop-shadow(0 4px 8px black);
  backdrop-filter: blur(8px);
}

.over {
  mix-blend-mode: multiply;        /* on the element */
}
.layer {
  background-blend-mode: overlay;  /* between background layers */
}
```

### Modern features worth knowing

| Feature | What it does |
|---|---|
| `:has(...)` | Parent / sibling-aware selection |
| `:is(...)` / `:where(...)` | Group selectors; `:where` is specificity 0 |
| Container queries + units (`cqw`, etc.) | Style by slot, not viewport |
| Cascade layers (`@layer`) | Predictable origin grouping; see [CSS & Design Systems](css-and-design-systems.md) |
| `@scope` | Limit a stylesheet block to a subtree, with optional lower bound |
| `@starting-style` | Define entry-animation start values for transitions |
| Anchor positioning (`anchor-name`, `position-anchor`, `inset-area`) | Pin tooltips/menus to anchors without JS |
| View Transitions (`@view-transition`, `view-transition-name`) | Cross-document and same-document animated transitions |
| `field-sizing: content` | Auto-size form controls to content |
| `:user-valid` / `:user-invalid` | Validation pseudo-classes that only apply *after* interaction |
| `light-dark(...)` | One declaration that respects `color-scheme` |
| `text-wrap: balance` / `pretty` | Better line breaking for headlines/body |
| Subgrid | Inherit parent's tracks |
| `@property` | Typed custom properties (interpolatable + with default + inheritance) |
| `color-mix()`, `color()`, `oklch()` | Modern color math |
| Nesting | Native CSS nesting — no preprocessor needed |

### Native nesting

```css
.card {
  padding: 1rem;

  & .title { font-weight: 600; }

  &:hover {
    background: var(--surface-2);
  }

  @media (min-width: 60rem) {
    padding: 2rem;
  }
}
```

The leading `&` is optional in modern syntax for class/element selectors but required when starting with an element name (`& div { ... }`).

---

## Mental models

### The cascade

The cascade decides which declaration wins, in this order:

1. **Origin & importance** — UA → user → author → animation; `!important` flips the order within origin.
2. **Cascade layer** — within author origin, layers (`@layer reset, base, components, utilities`) are applied in declaration order; later layers win. Unlayered styles win over layered ones.
3. **Specificity** — see arithmetic below.
4. **Order of appearance** — last declaration wins.

In practice: if two rules look like they should both apply, walk the four steps in order. The answer is always at the first step that differs.

### Specificity arithmetic

Specificity is a four-tuple: `(inline, id, class/attr/pseudo-class, element/pseudo-element)`. Compare left-to-right.

```
.btn.primary           → 0,0,2,0
button.primary         → 0,0,1,1
#submit                → 0,1,0,0
#submit.primary        → 0,1,1,0
:is(.a, #b)            → 0,1,0,0    (max of args)
:where(.a, #b)         → 0,0,0,0    (always 0)
:not(.a)               → 0,0,1,0    (specificity of arg)
```

Lesson learned: `:where()` exists so library/reset CSS can declare defaults at specificity 0 — consumers can override with a single class.

### Inheritance

| Inherits | Doesn't inherit |
|---|---|
| `color`, `font-*`, `line-height`, `text-*`, `letter-spacing`, `word-spacing`, `visibility`, `cursor`, `direction`, `writing-mode` | `display`, `position`, `width`/`height`, `margin`, `padding`, `border`, `background`, `box-shadow`, `transform`, `opacity` |

You can force inheritance with the `inherit` keyword, reset to initial with `initial`, or reset to inherited-or-initial with `unset`. `revert` rolls back to UA defaults; `revert-layer` rolls back through the current layer.

### Box model + `box-sizing`

```
+--------------------------------+   ← margin (transparent, can collapse)
|   +------------------------+   |
|   |        border          |   |
|   |  +------------------+  |   |
|   |  |     padding      |  |   |
|   |  |  +------------+  |  |   |
|   |  |  |  content   |  |  |   |
|   |  |  +------------+  |  |   |
|   |  +------------------+  |   |
|   +------------------------+   |
+--------------------------------+
```

- `box-sizing: content-box` — `width` is content only (default).
- `box-sizing: border-box` — `width` includes padding + border. Almost always what you want.

**Margin collapsing** between block-level siblings: only the larger of two adjacent vertical margins applies. Establish a new BFC (`display: flow-root`) or use padding to opt out.

### Formatting contexts

A formatting context is a layout sandbox. CSS has several:

| Context | Triggered by | Notes |
|---|---|---|
| Block (BFC) | `display: flow-root`, `overflow: auto/hidden`, floats, abs-pos, table cells, flex/grid children | Contains margin collapse, clears floats |
| Inline (IFC) | Default flow of inline content | Line boxes; baseline alignment |
| Flex | `display: flex` | Children become flex items |
| Grid | `display: grid` | Children become grid items |

A reframe: most "this layout breaks" issues are someone treating an inline element as block, or expecting margin to behave outside a BFC. Pick the right context and the rest follows.

### Containing blocks

Each element has a **containing block** — what its `width` percentage, `top`/`left` offsets, and so on resolve against.

| Element | Containing block |
|---|---|
| Static / relative | Nearest block-level ancestor |
| Absolute | Nearest **positioned** ancestor (any non-`static`) |
| Fixed | Viewport — *unless* an ancestor has `transform`/`filter`/`will-change: transform`, in which case that ancestor wins |
| Inside a Grid | Grid track or named area |

This is the source of most `position: fixed` "isn't fixed" bugs.

### Stacking contexts & `z-index`

`z-index` only orders elements *within the same stacking context*. A child can never escape its parent's stacking context. So a `z-index: 9999` deep in a tree will lose to a `z-index: 1` sibling of an ancestor stacking context.

To audit: open DevTools → Layers panel. Look at how things are nested; rearrange the DOM (or use `isolation: isolate`) to compose the right contexts.

### Flex vs Grid — when to reach for which

| Reach for | When |
|---|---|
| Flex | One dimension: a row of buttons, a navbar, a form field with a label |
| Grid | Two dimensions: a page layout, a card grid, a calendar |
| Both | A grid of cards (Grid for the layout) where each card has a Flex toolbar inside |

If you find yourself writing `flex-wrap: wrap` plus complicated child widths to simulate a grid, switch to Grid. If you find yourself defining one row of `grid-template-columns`, switch to Flex.

### Logical vs physical

- **Block axis** = the writing-mode's "line stacking" direction. In `horizontal-tb` (English), block is top-to-bottom.
- **Inline axis** = the line-flow direction. In English, inline is left-to-right.
- `start` / `end` map to the writing mode + direction.

In practice: writing logical from day one costs nothing. Switching back when you need RTL costs days.

### Paint, layout, composite — what's cheap to animate

| Property changes | Triggers |
|---|---|
| `transform`, `opacity`, `filter` | Composite only — cheapest |
| `color`, `background-color`, `box-shadow` | Paint |
| `width`, `height`, `top`, `left`, `padding`, `margin`, `border-width`, `font-size` | Layout (most expensive) |

Hint the browser to promote with `will-change: transform` (or `opacity`) — but only on elements you're *about to* animate, and remove the hint after. Permanent `will-change` is a memory leak.

For deeper detail on what each phase costs and how the browser decides, see [Browser Rendering Pipeline](browser-rendering-pipeline.md).

---

## Anti-patterns

- **`!important` to win specificity fights.** A symptom of unstructured CSS. Use `@layer` and selectors instead — see [CSS & Design Systems](css-and-design-systems.md).
- **`z-index: 9999`.** It loses to a `z-index: 1` in the right stacking context. Find the *real* containing context.
- **Animating `top`/`left`/`width`/`height` instead of `transform`.** Layout per frame is the path to jank.
- **Hard-coded `100vh` for full-screen mobile layouts.** Use `100dvh` — `vh` overshoots when the address bar is up.
- **Permanent `will-change`.** Allocates GPU memory forever. Add it just before animating, remove it after.
- **`px` for `font-size` (or `rem` for borders).** `rem` for typography (respects user zoom), `px` for borders/widths where exactness matters.
- **`* { margin: 0 }` and `* { box-sizing: border-box }` together as universal selectors.** Universal selector inheritance with `*` can be expensive on huge trees and has surprising specificity (`0,0,0,0`). Scope it.
- **Nesting deeper than three levels.** A class on the right element beats a deeply-nested chain.
- **`overflow: hidden` to fix layout overflow you didn't understand.** Find the runaway child instead.
- **`display: none` for state when `hidden` or `inert` is right.** `display: none` removes from layout, AT, and tab order — sometimes too aggressive, sometimes not enough; pick consciously.
- **Custom-styling form controls that ignore validation states.** Style `:focus-visible`, `:user-invalid`, `:placeholder-shown`, etc. — built-in states are accessible by default.
- **Media queries when container queries are right.** A `Card` component shouldn't care about viewport width; it should care about its own slot.
- **Using `:hover` for interactivity on touch devices.** Pair with `(hover: hover) and (pointer: fine)` so it doesn't stick on tap.
- **Letting `color-scheme: dark` happen by accident.** Set `color-scheme: light dark` on `:root` so form controls and scrollbars adapt.
- **Skipping `prefers-reduced-motion`.** Vestibular disorders are real; `@media (prefers-reduced-motion: reduce)` is a thirty-second fix.

---

## Checklist

- [ ] `box-sizing: border-box` set globally.
- [ ] `:root { color-scheme: light dark; }` so native UI matches the theme.
- [ ] Logical properties (`padding-block`, `inset-inline-start`, etc.) used by default.
- [ ] Type uses `rem`; container/component sizing uses `clamp()` for fluid scaling.
- [ ] `100dvh` (not `100vh`) for full-viewport heights on mobile.
- [ ] Animations use `transform`/`opacity`; `prefers-reduced-motion` honored.
- [ ] `:focus-visible` styled distinctly; never `outline: none` without a replacement.
- [ ] Container queries used where the component's behavior depends on its slot.
- [ ] No `!important` outside the utilities layer (or third-party-overrides layer).
- [ ] `:where()` used in resets so consumers can override at specificity 0.
- [ ] `light-dark()` (or layer-based theming) used for color adaptation.
- [ ] `text-wrap: balance` on headings, `text-wrap: pretty` on body.
- [ ] Form pseudo-classes (`:user-valid`/`:user-invalid`) used over JS-driven state.
- [ ] Stacking contexts audited; modal/overlay roots use `isolation: isolate`.

---

## Further reading

- [MDN — CSS reference](https://developer.mozilla.org/en-US/docs/Web/CSS/Reference) — the canonical lookup.
- [W3C CSS specs](https://www.w3.org/Style/CSS/current-work) — current state of every module.
- [web.dev — Learn CSS](https://web.dev/learn/css/) — the best beginners-to-intermediate course.
- [smolcss.dev](https://smolcss.dev/) — minimal modern-CSS recipes.
- [defensive CSS](https://defensivecss.dev/) — patterns that don't break under real content.
- [Josh Comeau — CSS for JS Devs](https://css-for-js.dev/) — paid but thorough.
- [Ahmad Shadeed](https://ishadeed.com/) — the gold standard for visual CSS explainers.
- [Una Kravets / Adam Argyle / Bramus on web.dev](https://web.dev/) — modern CSS in production.
- This repo: [CSS & Design Systems](css-and-design-systems.md) for architecture, tokens, theming, `@layer` strategy, and design-system patterns; [Browser Rendering Pipeline](browser-rendering-pipeline.md) for how paint/layout/composite phases work.

---

## Key terms

This guide is the canonical home for these glossary entries: **Cascade**, **Specificity**, **Stacking Context**, **Containing Block**, **Block Formatting Context (BFC)**, **Container Query**, **`:has()`**, **`:is()` / `:where()`**, **Logical Properties**, **`oklch`**, **Subgrid**, **`box-sizing`**, **Margin Collapsing**, **Inheritance**, **`text-wrap: balance` / `pretty`**. See the [glossary](glossary.md) for definitions and the rest of the index.

---

## Closing thought

CSS got dramatically better between 2020 and 2026 — `:has`, container queries, cascade layers, `oklch`, view transitions, scroll-driven animations, anchor positioning. Most of what we used to need preprocessors, libraries, and JS for is now native. The skill that pays off isn't memorizing properties; it's internalizing the four mental models — cascade, specificity, formatting contexts, stacking contexts — and trusting the platform to ship the rest.

What I tell people: when CSS surprises you, it's almost never a bug in CSS. It's a containing block you didn't see, a stacking context you didn't notice, or a margin that collapsed. Learn the rules once, and the language stops fighting you.
