# HTML Essentials & Cheat Sheet

*Last reviewed: 2026-05*

> A working architect's reference — the elements and attributes I reach for daily, plus the mental models that explain *why* the right tag matters more than the right class name.

HTML is the substrate. It's what screen readers parse, what search engines index, what the browser uses to decide tab order, what the parser turns into a DOM tree before a single byte of CSS or JS runs. Get HTML right and accessibility, SEO, and performance start working in your favor by default. Get it wrong and no amount of ARIA can fully repair it.

---

## Why this matters

Every `<div>` you ship instead of the right element is a tax — paid in accessibility audits, SEO ceilings, keyboard-trap bugs, and the labor of re-implementing browser behavior in JS. The platform ships an enormous amount of behavior for free; the only requirement is that you ask for it by name.

In practice: most HTML mistakes aren't typos — they're someone reaching for `<div>` because it's familiar, when `<button>`, `<details>`, `<dialog>`, or `<output>` would have given them focus, keyboard handling, and a screen-reader announcement at zero cost.

---

## Mindset

- **The element is the contract.** Browsers, assistive tech, and crawlers all key off element semantics. Class names mean nothing to them.
- **Reach for the most specific element that fits.** `<button>` over `<div role="button">`. `<nav>` over `<div class="nav">`. `<time>` over `<span>`.
- **Headings form a tree.** They describe document structure, not visual weight. Style independently.
- **Forms are an OS-native API.** Input types, autocomplete tokens, validation attrs — the browser does the work if you spell the request right.
- **Images are bandwidth contracts.** `srcset`, `sizes`, `loading`, `decoding`, `fetchpriority` are how you tell the browser what to ship and when.
- **The `<head>` is your contract with the platform.** Crawlers, social previews, install prompts, theme color — they all read here.
- **Default to no ARIA.** Native semantics first; ARIA only when the platform genuinely lacks an element. (See the [Web Accessibility](web-accessibility.md) guide for the deep dive.)

---

## The cheat sheet

### Document scaffolding

```html
<!doctype html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Page title — Site name</title>
    <meta name="description" content="One concise sentence." />
    <link rel="canonical" href="https://example.com/page" />
    <link rel="icon" href="/favicon.svg" type="image/svg+xml" />
    <meta name="theme-color" content="#0b0b0b" />
  </head>
  <body>
    <!-- content -->
  </body>
</html>
```

Worth knowing: `<!doctype html>` keeps the browser out of quirks mode. `lang` is required for assistive tech and translation. `viewport` is non-negotiable on mobile. Everything else is layered on top.

### Semantic landmarks

```html
<body>
  <header>
    <a href="/">Logo</a>
    <nav aria-label="Primary">
      <ul>
        <li><a href="/products">Products</a></li>
        <li><a href="/pricing">Pricing</a></li>
      </ul>
    </nav>
    <search>
      <form role="search">
        <label for="q">Search</label>
        <input id="q" type="search" name="q" />
      </form>
    </search>
  </header>

  <main>
    <article>
      <header><h1>Article title</h1></header>
      <section>
        <h2>Section heading</h2>
        <p>...</p>
      </section>
      <aside>Related links</aside>
      <footer>By Ada · 2026-01-01</footer>
    </article>
  </main>

  <footer>© 2026 Example, Inc.</footer>
</body>
```

| Element | Use for |
|---|---|
| `<header>` | Top of page or section — masthead, intro, byline |
| `<nav>` | Major navigation block — primary, secondary, breadcrumbs, TOC |
| `<main>` | The unique-to-this-page content. **Exactly one per page.** |
| `<article>` | Self-contained content (post, card, comment) — could stand alone |
| `<section>` | Thematic grouping with a heading |
| `<aside>` | Tangentially related content — sidebar, callout, pull quote |
| `<footer>` | Bottom of page or section — credits, related links |
| `<search>` *(2024+)* | Search-form region (replaces `role="search"`) |

### Headings

```html
<h1>One per page (or article)</h1>
<h2>Major section</h2>
<h3>Subsection</h3>
```

A reframe: `<h1>` doesn't mean "biggest text on the page" — it means "the topic of this document." Style headings however you want; let the level reflect structure.

### Text content

```html
<p>Paragraphs.</p>

<blockquote cite="https://source.example">
  <p>Quoted material.</p>
  <footer>— <cite>Author</cite></footer>
</blockquote>

<figure>
  <img src="chart.png" alt="Quarterly revenue chart showing 12% growth" />
  <figcaption>Q4 revenue, 2025.</figcaption>
</figure>

<hr />        <!-- thematic break, not a divider line -->
<pre><code>const x = 1;</code></pre>
```

### Inline semantics

| Element | Means |
|---|---|
| `<em>` | Stress emphasis (changes meaning if removed) |
| `<strong>` | Strong importance (warnings, key terms) |
| `<b>` | Stylistically offset, no extra meaning — last-resort |
| `<i>` | Alternate voice — taxonomies, foreign terms, ship names |
| `<code>` | Code fragment |
| `<kbd>` | Keyboard input — `<kbd>Cmd</kbd>+<kbd>K</kbd>` |
| `<samp>` | Sample program output |
| `<var>` | Variable in math/code prose |
| `<mark>` | Highlighted (as if with a marker) |
| `<time datetime="2026-01-01">` | Machine-readable date/time |
| `<abbr title="...">` | Abbreviation with expansion |
| `<dfn>` | Defining instance of a term |
| `<cite>` | Title of a work |
| `<q>` | Inline quotation (browser adds quote marks) |
| `<s>` | No longer accurate (price strikethrough, etc.) |
| `<del>` / `<ins>` | Edits — pair with `cite` and `datetime` for diffs |
| `<sub>` / `<sup>` | Subscript / superscript |
| `<bdi>` / `<bdo>` | Bidirectional text isolation / override |
| `<wbr>` | Word-break opportunity |

### Lists

```html
<!-- Unordered: order doesn't matter -->
<ul>
  <li>Apples</li>
  <li>Oranges</li>
</ul>

<!-- Ordered: sequence matters; supports start/reversed/value -->
<ol start="3" reversed>
  <li>Step</li>
  <li>Step</li>
</ol>

<!-- Description list: term/definition pairs -->
<dl>
  <dt>HTML</dt>
  <dd>HyperText Markup Language</dd>
  <dt>CSS</dt>
  <dd>Cascading Style Sheets</dd>
</dl>
```

A caveat: navigation menus belong inside `<ul>` — screen readers announce list length, which is useful context.

### Tables

```html
<table>
  <caption>Q4 revenue by region</caption>
  <colgroup>
    <col />
    <col span="2" class="num" />
  </colgroup>
  <thead>
    <tr>
      <th scope="col">Region</th>
      <th scope="col">Revenue</th>
      <th scope="col">YoY</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th scope="row">Americas</th>
      <td>$12.4M</td>
      <td>+8%</td>
    </tr>
  </tbody>
  <tfoot>
    <tr>
      <th scope="row">Total</th>
      <td>$48.1M</td>
      <td>+12%</td>
    </tr>
  </tfoot>
</table>
```

Worth knowing: `<caption>` is the accessible name; `scope="col"`/`"row"` is what screen readers use to announce headers when reading cells. Tables for tabular data only — never for layout.

### Forms

#### Input types

```html
<input type="text" />
<input type="email" autocomplete="email" />
<input type="password" autocomplete="new-password" />
<input type="tel" autocomplete="tel" inputmode="tel" />
<input type="url" />
<input type="search" />
<input type="number" min="0" max="100" step="1" />
<input type="date" />
<input type="datetime-local" />
<input type="month" />
<input type="week" />
<input type="time" />
<input type="color" />
<input type="range" min="0" max="11" />
<input type="checkbox" />
<input type="radio" name="plan" />
<input type="file" accept="image/*" multiple />
<input type="hidden" />
<input type="submit" />
```

#### Useful attributes

```html
<input
  type="email"
  name="email"
  required
  autocomplete="email"          <!-- one of the standardized tokens -->
  inputmode="email"             <!-- on-screen keyboard hint -->
  enterkeyhint="send"           <!-- the Enter-key label -->
  spellcheck="false"
  autocapitalize="off"
  placeholder="you@example.com" <!-- supplements, never replaces, a label -->
/>
```

Worth knowing: `autocomplete` tokens are standardized — `email`, `username`, `current-password`, `new-password`, `one-time-code`, `street-address`, `postal-code`, `cc-number`, `bday`, etc. Spell them right; password managers and browser autofill key off them.

#### Labels & grouping

```html
<!-- Always associate label and input -->
<label for="name">Name</label>
<input id="name" name="name" />

<!-- Or wrap (no `for` needed) -->
<label>
  Name
  <input name="name" />
</label>

<!-- Group related controls -->
<fieldset>
  <legend>Shipping address</legend>
  <label>Street <input name="street" /></label>
  <label>City <input name="city" /></label>
</fieldset>
```

#### Validation attributes

```html
<input required />
<input minlength="3" maxlength="20" />
<input type="number" min="1" max="10" step="0.5" />
<input pattern="[A-Z]{3}-\d{4}" />
<input type="email" multiple />
```

Forms validate on submit by default; pair with `novalidate` on the form to take over in JS, or with the [Constraint Validation API](https://developer.mozilla.org/en-US/docs/Web/API/Constraint_validation) to extend.

#### Specialized form elements

```html
<select name="country" autocomplete="country">
  <optgroup label="North America">
    <option value="us">United States</option>
    <option value="ca">Canada</option>
  </optgroup>
</select>

<datalist id="cities">
  <option value="London">
  <option value="Lagos">
</datalist>
<input list="cities" name="city" />

<textarea name="bio" rows="4" maxlength="500"></textarea>

<output name="result" for="a b">0</output>

<progress value="70" max="100">70%</progress>
<meter value="0.6" min="0" max="1" low="0.3" high="0.8" optimum="1">60%</meter>
```

### Media

#### Images

```html
<img
  src="hero-1200.jpg"
  srcset="hero-600.jpg 600w, hero-1200.jpg 1200w, hero-2400.jpg 2400w"
  sizes="(min-width: 60rem) 60rem, 100vw"
  alt="A red bicycle leaning against a brick wall"
  width="1200" height="675"
  loading="lazy"
  decoding="async"
  fetchpriority="high"
/>
```

| Attribute | What it does |
|---|---|
| `srcset` + `sizes` | Browser picks the best size for the viewport + DPR |
| `width`/`height` | Reserves space — prevents layout shift (CLS) |
| `loading="lazy"` | Defer until near viewport. Don't lazy-load LCP images. |
| `decoding="async"` | Decode off the main thread |
| `fetchpriority="high"` | Hoist the LCP image up the queue |
| `alt=""` | Decorative — screen readers skip. Empty, never absent. |

#### `<picture>` for art direction or format negotiation

```html
<picture>
  <source srcset="hero.avif" type="image/avif" />
  <source srcset="hero.webp" type="image/webp" />
  <img src="hero.jpg" alt="..." width="1200" height="675" />
</picture>
```

#### Video & audio

```html
<video
  src="clip.mp4"
  poster="poster.jpg"
  width="800" height="450"
  controls preload="metadata" playsinline
>
  <track kind="captions" src="en.vtt" srclang="en" label="English" default />
  <p>Your browser doesn't support video.</p>
</video>

<audio src="podcast.mp3" controls preload="none"></audio>
```

A caveat: autoplay only works muted (`muted autoplay`) on most browsers. Always provide controls or honor `prefers-reduced-motion` for ambient video.

### Interactive elements

#### Disclosure (`<details>` / `<summary>`)

```html
<details>
  <summary>Frequently asked questions</summary>
  <p>Answer here.</p>
</details>
```

Free expand/collapse with keyboard support and accessible-name semantics. No JS needed.

#### Modal (`<dialog>`)

```html
<dialog id="confirm">
  <form method="dialog">
    <p>Delete this record?</p>
    <button value="cancel">Cancel</button>
    <button value="confirm">Delete</button>
  </form>
</dialog>

<script>
  const d = document.getElementById("confirm");
  d.showModal();    // focus-trapped, ESC-dismissable, top-layer
  d.close("cancel");
</script>
```

`<dialog>` ships focus trap, backdrop, top-layer rendering, and ESC handling. The form's `method="dialog"` returns the clicked button's value — no JS event wiring.

#### Popover (`popover` attribute, 2024+)

```html
<button popovertarget="menu">Menu</button>
<div id="menu" popover>
  <a href="/profile">Profile</a>
  <a href="/logout">Logout</a>
</div>
```

Auto-closes on outside click and ESC. Works without JS for tooltips, dropdowns, and menus.

### Embedded

```html
<iframe
  src="https://maps.example.com/?q=NYC"
  title="Map of NYC"
  width="600" height="400"
  loading="lazy"
  referrerpolicy="no-referrer"
  sandbox="allow-scripts allow-same-origin"
  allow="geolocation"
></iframe>
```

| Attribute | Why |
|---|---|
| `title` | Accessible name — required for screen readers |
| `loading="lazy"` | Defer offscreen iframes |
| `sandbox` | Restrict capabilities; opt back in token by token |
| `allow` | Permissions Policy delegation (camera, geolocation, etc.) |
| `referrerpolicy` | Strip or limit Referer header |

### `<head>` essentials

```html
<!-- SEO + sharing -->
<meta name="description" content="..." />
<link rel="canonical" href="https://example.com/page" />
<meta property="og:title" content="..." />
<meta property="og:description" content="..." />
<meta property="og:image" content="https://example.com/og.png" />
<meta property="og:type" content="article" />
<meta name="twitter:card" content="summary_large_image" />

<!-- App / install -->
<link rel="manifest" href="/manifest.webmanifest" />
<meta name="theme-color" content="#0b0b0b" />
<link rel="apple-touch-icon" href="/icon-180.png" />

<!-- Resource hints -->
<link rel="preload" href="/fonts/inter.woff2" as="font" type="font/woff2" crossorigin />
<link rel="preconnect" href="https://api.example.com" crossorigin />
<link rel="dns-prefetch" href="https://cdn.example.com" />
<link rel="modulepreload" href="/app.js" />
<link rel="prefetch" href="/next-route.js" />

<!-- Structured data: defer to the SEO guide for depth -->
<script type="application/ld+json">{ "@context": "https://schema.org", "@type": "Article", ... }</script>
```

For OG/Twitter strategy, structured data, and `robots`/`sitemap.xml`, see the [SEO guide](seo.md).

### Modern attributes worth knowing

| Attribute | What it does |
|---|---|
| `inert` | Removes element + descendants from focus order, click, AT |
| `popover` | Native popover behavior + top-layer rendering |
| `hidden` | Visually hidden + removed from AT (use over `display:none` for state) |
| `tabindex="-1"` | Programmatic focus only; `0` = natural order; positive values = avoid |
| `contenteditable` | Make any element editable |
| `enterkeyhint` | Label the Enter key on virtual keyboards (`go`, `send`, `search`, ...) |
| `inputmode` | Pick the right virtual keyboard (`numeric`, `email`, `tel`, ...) |
| `autocapitalize` | `off` for usernames/codes, `sentences` for prose |
| `spellcheck` | `false` for codes, IDs, code blocks |
| `translate="no"` | Tell translators to skip (brand names, code) |
| `dir="auto"` | Auto-detect base direction from content |
| `is="..."` | Customize a built-in element (Customized Built-Ins) |
| `slot="..."` | Project content into a Web Component slot |
| `part="..."` | Expose Shadow DOM internals for `::part()` styling |

---

## Mental models

### Semantic vs presentational

The browser doesn't care what your `<div>` looks like; it cares what it *means*. A `<button>` styled as a link is still a button — focusable, Enter/Space-activated, announced as "button" by screen readers. A `<div onclick>` is a div — not focusable, not keyboard-operable, not announced.

A reframe: pick the element for the *behavior and meaning*, then style it however you like.

### Landmarks and the document outline

Assistive tech navigates by landmarks (`header`, `nav`, `main`, `aside`, `footer`, `search`). Screen-reader users routinely jump straight to `<main>` or cycle through landmarks with a single key. If your page is `<div><div><div>`, those shortcuts don't work.

In practice: the rule is one `<main>` per page, and every `<nav>` / `<aside>` / multiple `<header>`/`<footer>` should be distinguishable — give them an `aria-label` if there are more than one of the same kind.

### Headings as a tree

Headings describe document structure. Skipping levels (h1 → h3) confuses outline tools. Multiple `<h1>`s used to be discouraged, but the modern rule is simple: one `<h1>` per *document* (or per `<article>`), and nest h2 → h3 → h4 logically below it.

### Forms are an OS-native API

Spell the request and the platform fulfills it: `type="email"` brings the email keyboard; `autocomplete="one-time-code"` lets iOS auto-paste 2FA codes; `inputmode="numeric"` shows a digit keypad; `enterkeyhint="send"` labels the Enter key. None of these need JS.

Lesson learned: forms broken by JS-only validation are forms that break for keyboard users, password managers, and screen readers. Lean on the platform; reach for JS only where the platform genuinely lacks a primitive.

### Images are bandwidth contracts

`<img src>` alone tells the browser nothing about which size it should ship. `srcset`/`sizes` lets the browser negotiate based on viewport and DPR. `loading="lazy"` defers offscreen images. `width`/`height` reserves space and prevents Cumulative Layout Shift. `fetchpriority="high"` on the LCP image hoists it ahead of the rest.

For LCP and image strategy, see [Core Web Vitals](core-web-vitals.md) and [Web Optimization Techniques](web-optimization-techniques.md).

### The `<head>` is your contract with the platform

The head is read by browsers, search crawlers, social-share bots, OS install handlers, screenshot tools, and feed readers. It's the single most under-loved file region in most repos. Get `<title>`, `<meta description>`, canonical, OG tags, and resource hints right and a lot of downstream things just work.

---

## Div soup → semantic (quick replacements)

| Instead of | Use | Why |
|---|---|---|
| `<div onclick>` | `<button type="button">` | Focus, keyboard, AT all free |
| `<div class="link">` | `<a href>` | Right-click, middle-click, focus, screen-reader announcement |
| `<div class="nav">` | `<nav>` with `<ul>` of links | Landmark + list-length semantics |
| `<div class="header">` | `<header>` | Landmark |
| `<div class="footer">` | `<footer>` | Landmark |
| `<div class="card">` (post/comment/product) | `<article>` | Self-contained content semantic |
| `<div class="sidebar">` | `<aside>` | Tangential-content semantic |
| Floating action button | `<button>` | Same |
| Custom checkbox | `<input type="checkbox">` styled with CSS | Saves implementing focus, keyboard, indeterminate, mixed state |
| Custom select | `<select>` (consider [`<selectlist>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/selectlist) when stable) | Saves a *lot*; native is good enough almost always |
| Custom modal | `<dialog>` + `showModal()` | Focus trap, backdrop, ESC, top-layer — all free |
| Custom dropdown | `popover` attribute | Auto-close, light-dismiss free |
| FAQ accordion | `<details>` / `<summary>` | No JS needed |
| Bold for emphasis | `<strong>` (or `<b>` if visual-only) | Conveys importance to AT |
| Italic for stress | `<em>` (or `<i>` if alternate-voice) | Conveys stress to AT |
| `<span>` for date | `<time datetime="2026-01-01">` | Machine-readable |
| `<span>` for code | `<code>` | Searchability + AT |
| Layout `<table>` | CSS Grid or Flexbox | Tables are tabular data |
| `<br>` for spacing | CSS margin/padding | `<br>` is a line break in prose, not a layout tool |

---

## Anti-patterns

- **`<div role="button">` instead of `<button>`.** The platform did the work; don't pay to redo it.
- **`<a href="#" onclick>`.** Not a link, not a button — works for nobody.
- **Layout tables.** Screen readers announce row/column counts that mean nothing for layout.
- **Skipping heading levels for visual size.** `<h1>` then `<h4>` because the smaller one looked nicer. Use CSS for size; let levels reflect structure.
- **`alt` missing on `<img>`.** Decorative images need `alt=""` (empty, not absent). Content images need a real description.
- **Placeholder as label.** Disappears on focus, low contrast, no AT semantics. Use `<label>`.
- **`disabled` on submit until valid.** Users get no feedback on what's wrong. Let them submit, then show errors.
- **`<select>` with hundreds of options.** Use `<input list>` + `<datalist>` or a typeahead component.
- **Iframes without `title`.** Screen readers announce them as "frame" — useless.
- **Nested `<button>` or interactive-in-interactive.** Invalid, AT chaos.
- **Lazy-loading the LCP image.** Defers the metric you're trying to make fast.
- **`<br><br>` for paragraphs.** Use `<p>`. AT users hear the difference.
- **`autofocus` everywhere.** Disorienting — only the *primary action* of the *current view*.
- **`tabindex="3"`, `tabindex="5"`.** Positive tabindex is almost always a bug; rearrange the DOM instead.
- **Wrapping a whole card in `<a>`.** Selectable text becomes click-to-navigate. Use a single link inside, or use `aria-labelledby` patterns.

---

## Checklist

- [ ] Every page has `<!doctype html>`, `lang`, `charset`, `viewport`, `<title>`, `<meta description>`, canonical.
- [ ] Exactly one `<main>`, one `<h1>` (per document or per article), heading levels nest without skipping.
- [ ] Landmarks (`header`, `nav`, `main`, `aside`, `footer`, `search`) used; multiples disambiguated with `aria-label`.
- [ ] All interactive elements are real interactives — `<button>`, `<a href>`, `<input>`, `<select>`, `<details>`, `<dialog>`.
- [ ] Every `<input>` has an associated `<label>`; related controls grouped with `<fieldset>`/`<legend>`.
- [ ] `autocomplete`, `inputmode`, `enterkeyhint` set on form fields where appropriate.
- [ ] Images have `alt`, `width`, `height`; LCP image has `fetchpriority="high"` and is **not** lazy-loaded.
- [ ] `<picture>` used for format negotiation (AVIF/WebP/JPEG fallback) where it pays off.
- [ ] Tables have `<caption>`, `<thead>`/`<tbody>`, `scope` on header cells.
- [ ] `<dialog>` for modals, `popover` for menus/tooltips, `<details>` for disclosure.
- [ ] Iframes have `title`, `loading="lazy"`, `sandbox`, and `referrerpolicy` where appropriate.
- [ ] Resource hints (`preload`, `preconnect`, `dns-prefetch`, `modulepreload`) added intentionally — not sprayed.
- [ ] OG / Twitter / theme-color / manifest set for sharing and install.
- [ ] No `tabindex` greater than 0; `inert` used to remove offscreen-modal trees from focus.
- [ ] Validate with `lang`-aware tools — [the W3C validator](https://validator.w3.org/) still earns its keep.

---

## Further reading

- [HTML Living Standard (WHATWG)](https://html.spec.whatwg.org/) — the spec; surprisingly readable.
- [MDN — HTML elements reference](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).
- [HTML5 Doctor — semantic element guidance](http://html5doctor.com/) — older but evergreen on `article` vs `section` etc.
- [Open UI](https://open-ui.org/) — where `popover`, `selectlist`, and the next layer of native components are designed.
- [web.dev — HTML & accessibility](https://web.dev/learn/html/).
- [Adam Argyle / Una Kravets / Rachel Andrew posts on web.dev](https://web.dev/) — modern HTML/CSS in practice.
- [The A11y Project](https://www.a11yproject.com/) — patterns for the cases the platform doesn't fully cover.
- This repo: [Web Accessibility](web-accessibility.md), [SEO](seo.md), [Core Web Vitals](core-web-vitals.md), [Web Optimization Techniques](web-optimization-techniques.md), [CSS & Design Systems](css-and-design-systems.md).

---

## Key terms

This guide is the canonical home for these glossary entries: **`<article>`**, **`<aside>`**, **`autocomplete` tokens**, **Constraint Validation API**, **`<dialog>`**, **`<details>` / `<summary>`**, **Landmarks**, **`<main>`**, **`<picture>`**, **`popover`**, **Resource Hints**, **`<search>`**, **`srcset` / `sizes`**, **`<time>`**, **Viewport Meta**, **`inert`**, **`enterkeyhint`**, **`inputmode`**, **`tabindex`**. See the [glossary](glossary.md) for definitions and the rest of the index.

---

## Closing thought

HTML is the cheapest leverage you'll ever buy. Spending five extra seconds picking the right element saves hours of accessibility remediation, SEO debt, and JavaScript reinvention later. The platform has spent thirty years learning what users need from a button, a form, a modal, a video — let it ship that work for you.

What I tell people: write the HTML you'd want to read with the screen reader off, then the screen reader on. If both reads make sense, you've already done 80% of the work.
