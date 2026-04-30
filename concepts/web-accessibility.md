# Web Accessibility (a11y)

> A practical guide to building inclusive, accessible web applications.

---

## Table of Contents

1. [What is Web Accessibility?](#what-is-web-accessibility)
2. [Why is it Important?](#why-is-it-important)
3. [The Standards: WCAG, ARIA, and the Law](#the-standards-wcag-aria-and-the-law)
4. [The POUR Principles](#the-pour-principles)
5. [How to Implement Accessibility](#how-to-implement-accessibility)
6. [Precautions and Common Pitfalls](#precautions-and-common-pitfalls)
7. [Testing & Tooling](#testing--tooling)
8. [Accessibility in React](#accessibility-in-react)
9. [Building an Accessibility Culture](#building-an-accessibility-culture)
10. [Further Reading](#further-reading)

---

## What is Web Accessibility?

**Web accessibility (commonly abbreviated as `a11y` — "a", 11 letters, "y")** is the practice of designing and developing websites, tools, and technologies so that **people with disabilities can perceive, understand, navigate, interact with, and contribute to the web**.

It encompasses all disabilities that affect access to the web, including:

| Category | Examples |
|---|---|
| **Visual** | Blindness, low vision, color blindness |
| **Auditory** | Deafness, hard of hearing |
| **Motor** | Inability to use a mouse, limited fine motor control, tremors |
| **Cognitive** | Learning disabilities, dyslexia, ADHD, memory loss |
| **Neurological** | Seizure disorders (e.g., photosensitive epilepsy) |
| **Speech** | Difficulty using voice interfaces |
| **Situational** | Bright sunlight, noisy environment, broken arm, slow network |

> Accessibility is **not** a niche concern — it benefits **everyone**. Captions help in noisy cafes. Keyboard navigation helps power users. High contrast helps in sunlight. This is the **curb-cut effect**: features built for disability help all users.

---

## Why is it Important?

### 1. It's a Human Right
Roughly **1.3 billion people** (~16% of the global population) live with a significant disability (WHO, 2023). Excluding them from the web is exclusion from modern life — banking, healthcare, education, employment.

### 2. Legal & Regulatory Compliance
Inaccessible products invite real legal exposure:

- **United States** — Americans with Disabilities Act (ADA), Section 508 (federal), Title III (private businesses). Lawsuits have surged: ~4,000+ ADA web lawsuits filed annually in U.S. federal courts.
- **European Union** — European Accessibility Act (EAA) — **enforced June 28, 2025** for most consumer-facing digital products and services.
- **United Kingdom** — Equality Act 2010, PSBAR for public sector.
- **Canada** — AODA (Ontario), ACA (federal).
- **Australia** — Disability Discrimination Act (DDA).

Notable cases: *Domino's Pizza v. Robles* (2019, U.S. Supreme Court declined to hear — ADA applies to websites), *Target* class action ($6M settlement).

### 3. Business Value
- **Larger market.** Disability + aging population = a major segment with significant disposable income (the "purple pound" in the UK is ~£274B).
- **SEO benefits.** Semantic HTML, alt text, and proper headings overlap heavily with what search crawlers want.
- **Better UX for everyone.** Clear focus states, readable typography, robust keyboard support → improved usability metrics across the board.
- **Lower maintenance cost.** Accessible code is usually more semantic, more testable, and more durable.

### 4. Brand & Reputation
Accessibility failures show up in social media, press, and procurement RFPs. Many enterprise buyers now require a **VPAT (Voluntary Product Accessibility Template)** before signing.

---

## The Standards: WCAG, ARIA, and the Law

### WCAG (Web Content Accessibility Guidelines)
The international standard, published by the W3C's WAI.

| Version | Year | Status |
|---|---|---|
| WCAG 2.0 | 2008 | Foundation, ISO standard |
| WCAG 2.1 | 2018 | Adds mobile, low-vision, cognitive |
| WCAG 2.2 | 2023 | Adds focus appearance, dragging alternatives, target size |
| WCAG 3.0 | Draft | Major restructure, scoring-based — not yet stable |

**Conformance levels:**
- **A** — Minimum (must-have, but insufficient alone)
- **AA** — **The legal/industry target** for most products
- **AAA** — Highest, often impractical for entire sites

### ARIA (Accessible Rich Internet Applications)
A spec for adding semantics to dynamic content where HTML alone falls short — `role`, `aria-*` attributes, and live regions.

> **First Rule of ARIA: Don't use ARIA.** If a native HTML element gives you the semantics for free (`<button>`, `<nav>`, `<dialog>`), use it. ARIA does not change behavior; it only changes how assistive technology *announces* an element. Misused ARIA is often worse than no ARIA.

### Other Specs Worth Knowing
- **ATAG** — Authoring Tool Accessibility Guidelines (CMS, IDEs)
- **UAAG** — User Agent (browser) Accessibility Guidelines
- **EPUB Accessibility 1.1**, **PDF/UA** — for documents

---

## The POUR Principles

WCAG is organized around four foundational principles:

### 1. **P**erceivable
Information must be presentable in ways users can perceive.
- Text alternatives for non-text content (`alt`, captions, transcripts)
- Adaptable content (semantic structure that doesn't depend on visual layout)
- Distinguishable: sufficient contrast, resizable text, no audio that auto-plays

### 2. **O**perable
Interface components must be operable.
- Full keyboard accessibility — no keyboard traps
- Enough time to read and interact
- No content that causes seizures (no flashing >3 times/second)
- Navigable: skip links, page titles, focus order, link purpose

### 3. **U**nderstandable
Information and operation must be understandable.
- Readable language (`lang` attribute, plain language)
- Predictable behavior (no surprise context changes)
- Input assistance (clear error messages, labels, hints)

### 4. **R**obust
Content must be robust enough for current and future assistive tech.
- Valid, parseable HTML
- Name, role, value exposed correctly to AT
- Status messages communicated programmatically

---

## How to Implement Accessibility

### 1. Start with Semantic HTML
The single highest-leverage thing you can do.

```html
<!-- ❌ Bad: div soup -->
<div class="button" onclick="submit()">Submit</div>

<!-- ✅ Good: semantic, focusable, keyboard-operable, announced as a button -->
<button type="submit">Submit</button>
```

Use the right element for the job:
- `<button>` for actions, `<a href>` for navigation
- `<nav>`, `<main>`, `<header>`, `<footer>`, `<aside>`, `<section>`, `<article>` for landmarks
- `<h1>`–`<h6>` in a logical, non-skipping hierarchy
- `<ul>`, `<ol>`, `<dl>` for lists
- `<label>` always associated with form controls

### 2. Provide Text Alternatives

```html
<!-- Informative image -->
<img src="chart.png" alt="Q4 revenue grew 23% year-over-year" />

<!-- Decorative image -->
<img src="flourish.svg" alt="" role="presentation" />

<!-- Functional image (icon button) -->
<button aria-label="Close dialog">
  <svg aria-hidden="true">...</svg>
</button>
```

Rules of thumb:
- **Alt text describes purpose, not appearance.**
- Decorative images get `alt=""` (empty, not missing).
- Don't start with "Image of…" — screen readers already say that.
- For complex images (charts), provide a long description nearby or via `aria-describedby`.

### 3. Color & Contrast

| WCAG 2.2 Level AA | Minimum ratio |
|---|---|
| Normal text | **4.5:1** |
| Large text (18pt+ or 14pt bold) | **3:1** |
| UI components & graphical objects | **3:1** |
| Focus indicator | **3:1** against adjacent colors |

**Never rely on color alone.** Pair color with icons, text, or patterns.

```html
<!-- ❌ Color-only error indication -->
<input style="border-color: red" />

<!-- ✅ Color + icon + text -->
<input aria-invalid="true" aria-describedby="email-err" />
<span id="email-err"><Icon/> Please enter a valid email</span>
```

### 4. Keyboard Accessibility
Every interactive element must be:
- **Reachable** via Tab
- **Operable** via Enter/Space (buttons), arrow keys (menus, sliders, tabs)
- **Visible** when focused (never `outline: none` without a replacement)
- **Escapable** — Esc closes dialogs, menus, popovers

Standard keyboard patterns (see [WAI-ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/)):

| Component | Keys |
|---|---|
| Tabs | ←/→ to move, Home/End to jump |
| Menu | ↑/↓ to move, Esc to close, Enter to activate |
| Dialog | Esc closes, focus trapped inside, returns to trigger on close |
| Combobox | ↑/↓ to navigate options, Enter to select |

### 5. Focus Management
- **Visible focus rings.** If you remove the default, replace it with something equally clear (3:1 contrast).
- **Logical tab order.** Match DOM order to visual order. Avoid `tabindex` values > 0.
- **Manage focus on route change** in SPAs — move focus to the new page's `<h1>` or main landmark.
- **Trap focus in modals** while open; **restore focus** to the trigger on close.
- **Skip links** at the top: `<a href="#main">Skip to main content</a>`.

### 6. Forms

```html
<label for="email">Email address</label>
<input
  id="email"
  type="email"
  required
  autocomplete="email"
  aria-describedby="email-hint email-err"
  aria-invalid="false"
/>
<p id="email-hint">We'll never share your email.</p>
<p id="email-err" role="alert"></p>
```

Checklist:
- Every input has a programmatically associated `<label>` (or `aria-label` / `aria-labelledby` as fallback)
- Group related controls with `<fieldset>` + `<legend>`
- Use `autocomplete` tokens — huge win for cognitive and motor disabilities
- Errors are announced (use `role="alert"` or `aria-live="assertive"` for new errors)
- Don't disable submit until valid — let users try and learn what's wrong
- Server errors return focus to the first invalid field

### 7. ARIA — When You Truly Need It

```html
<!-- Live region for async status updates -->
<div role="status" aria-live="polite">
  3 results loaded.
</div>

<!-- Toggle button -->
<button aria-pressed="false" aria-label="Mute">🔊</button>

<!-- Disclosure -->
<button aria-expanded="false" aria-controls="faq-1">What is a11y?</button>
<div id="faq-1" hidden>...</div>
```

**ARIA principles:**
- Don't change native semantics (don't put `role="button"` on a `<button>`)
- All interactive ARIA controls must be keyboard accessible
- Don't use `role="presentation"` or `aria-hidden="true"` on a focusable element
- Visible label text should be in the accessible name (avoid `aria-label` overriding visible text — confuses voice control users)

### 8. Motion, Animation & Time
- Respect `prefers-reduced-motion` — disable or tone down animation.
- No content flashing more than **3 times per second** (seizure risk).
- Auto-advancing carousels need pause/stop controls.
- Sessions that time out must warn the user and let them extend.

```css
@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 9. Responsive & Zoom
- Page must work at **400% zoom** without horizontal scroll (WCAG 1.4.10 Reflow).
- Use relative units (`rem`, `em`) — never lock font sizes in `px` for body text.
- Touch targets: minimum **24×24 CSS pixels** (WCAG 2.2 — 2.5.8), 44×44 recommended (Apple HIG).

### 10. Internationalization
- Set `<html lang="en">` (and `lang="…"` on inline language switches).
- Use logical CSS properties (`margin-inline-start` not `margin-left`) for RTL.

---

## Precautions and Common Pitfalls

### 1. Don't Reinvent Native Controls Without Cause
Custom selects, checkboxes, and date pickers are notorious for being inaccessible. If you must build one, follow the [ARIA Authoring Practices](https://www.w3.org/WAI/ARIA/apg/) **completely** — partial patterns are usually worse than none.

### 2. Don't Hide Focus
```css
/* ❌ Catastrophic for keyboard users */
*:focus { outline: none; }

/* ✅ Better */
*:focus-visible {
  outline: 2px solid currentColor;
  outline-offset: 2px;
}
```

### 3. Don't Trap Keyboard Users
Modals must have an Esc handler. Custom widgets must let Tab escape when appropriate. **Test every flow with the keyboard alone.**

### 4. Don't Misuse `aria-hidden`
- Never put `aria-hidden="true"` on a focusable element (creates "ghost" focus).
- Don't `aria-hidden` your entire app while a modal is open — use the `inert` attribute instead.

### 5. Don't Confuse `display: none` with Screen-Reader-Only
| Technique | Visual | Screen Reader | Focusable |
|---|---|---|---|
| `display: none` | Hidden | Hidden | No |
| `visibility: hidden` | Hidden | Hidden | No |
| `hidden` attr | Hidden | Hidden | No |
| `aria-hidden="true"` | Visible | Hidden | **Still yes — bug risk** |
| `.sr-only` (clip class) | Hidden | **Read** | Yes (if focusable) |
| `inert` | Visible | Hidden | No |

### 6. Don't Auto-Play Media or Spawn Dialogs Unprompted
Both are WCAG violations and disorient screen reader users. If you must auto-play, mute by default and provide controls.

### 7. Don't Rely Solely on Automated Tests
Tools like axe catch ~**30–40% of issues**. The rest — meaningful alt text, logical reading order, sensible focus management — needs manual testing. **You must test with a screen reader.**

### 8. Don't Use Placeholder as Label
Placeholder disappears on input, has poor contrast, and is missed by many users. Always use a real `<label>`.

### 9. Don't Make Everything a `<div>`
Div-soup destroys semantics. Even if you style it perfectly, screen readers see nothing structural.

### 10. Don't Treat Accessibility as a Final QA Step
Retrofitting a11y costs **5–10x** more than building it in. Bake it into design reviews, ticket acceptance criteria, and PR templates.

### 11. Don't Forget Documents
PDFs, Word docs, and exported reports are part of your product. They need their own accessibility (tagged PDFs, alt text, reading order).

### 12. Don't Conflate Accessibility with Overlay Widgets
Third-party "accessibility overlays" (accessiBe, UserWay, etc.) **do not make a site WCAG compliant** and have been the subject of [a public statement signed by ~700 a11y experts](https://overlayfactsheet.com/). They can actively harm users. Fix the underlying code.

---

## Testing & Tooling

### Automated
- **[axe DevTools](https://www.deque.com/axe/)** (browser extension, CI integration via `@axe-core/*`)
- **Lighthouse** (Chrome DevTools, also CI)
- **WAVE** by WebAIM (browser extension)
- **Pa11y**, **jest-axe**, **eslint-plugin-jsx-a11y**, **@axe-core/playwright**
- **Storybook a11y addon** for component-level checks

### Manual
- **Keyboard-only test** — unplug your mouse for an hour
- **Screen reader test:**
  - macOS: **VoiceOver** (Cmd+F5) — pair with Safari
  - Windows: **NVDA** (free) — pair with Firefox or Chrome; **JAWS** for enterprise
  - iOS: VoiceOver; Android: TalkBack
- **Zoom test** — 200% and 400%
- **Color contrast** — Stark, Contrast (figma plugins), browser DevTools
- **Reduced motion** — toggle OS setting and re-test
- **Forced colors / Windows High Contrast** mode

### Recommended CI Setup
```yaml
# Example: axe in Playwright
- name: a11y scan
  run: npx playwright test --grep @a11y
```
Block PRs on critical/serious violations; allow review for moderate.

---

## Accessibility in React

React-specific guidance, since this is a React codebase:

### Use `eslint-plugin-jsx-a11y`
Catches many issues at lint time — `alt` text, `tabindex`, label association, valid ARIA props.

### Prefer Native Elements
```jsx
// ❌
<div onClick={handleClick} role="button" tabIndex={0} onKeyDown={...}>

// ✅
<button onClick={handleClick}>
```

### Manage Focus Across Routes
SPAs don't reload — the browser doesn't reset focus or announce the new page.

```jsx
useEffect(() => {
  const heading = document.querySelector('h1');
  heading?.focus(); // requires tabIndex={-1} on the heading
}, [pathname]);
```

Or use `react-router`'s scroll-restoration patterns + a "skip to content" link.

### Headless / Accessible Component Libraries
Battle-tested primitives that handle the tricky ARIA + keyboard logic for you:

- **[React Aria](https://react-spectrum.adobe.com/react-aria/)** (Adobe) — the gold standard
- **[Radix UI](https://www.radix-ui.com/)** — primitives for popovers, dialogs, menus
- **[Headless UI](https://headlessui.com/)** (Tailwind Labs)
- **[Reach UI](https://reach.tech/)** (older but still useful)

Use these instead of building custom comboboxes, menus, dialogs, or date pickers from scratch.

### Common React A11y Bugs
- Forgetting `htmlFor` on `<label>` (note: not `for` in JSX)
- Spreading props that overwrite `aria-*` attributes silently
- Conditional rendering with `&&` showing `0` or stray text to screen readers
- Toast/notification components without `role="status"` or `aria-live`
- Portals (modals) without focus management

### Testing in React
```jsx
import { render } from '@testing-library/react';
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

test('no a11y violations', async () => {
  const { container } = render(<MyComponent />);
  expect(await axe(container)).toHaveNoViolations();
});
```

Pair with **@testing-library/react** which encourages querying by accessible name (`getByRole`, `getByLabelText`) — if your tests can find elements, screen readers can too.

---

## Building an Accessibility Culture

Sustainable accessibility is **organizational**, not just technical:

### Process
- **Definition of Done** includes a11y acceptance criteria for every story.
- **Design review** checks contrast, focus order, error states before handoff.
- **PR template** has an a11y checkbox: "I tested keyboard navigation and ran axe."
- **Component library** is the source of truth — fix it once, fix it everywhere.

### People
- Designate an **accessibility champion** per team.
- Train designers, PMs, content writers, and QA — not just engineers.
- Where possible, **involve users with disabilities** in research and usability testing. Nothing replaces lived experience.

### Documentation
- Maintain a public **accessibility statement** with conformance level, known issues, and a contact path.
- Publish a **VPAT** if you sell to enterprise/government.
- Keep an internal **a11y pattern library** documenting your team's accessible solutions.

### Metrics That Matter
- % of components with axe-clean tests
- # of a11y bugs in backlog (track separately, monitor age)
- Time-to-fix for keyboard/screen-reader regressions
- User-reported issues from assistive technology users

---

## Quick Reference: 12-Item Pre-Ship Checklist

- [ ] All interactive elements reachable and operable by keyboard
- [ ] Visible focus indicator (3:1 contrast minimum)
- [ ] Logical heading hierarchy (`<h1>` → `<h2>` → …)
- [ ] All images have meaningful `alt` (or `alt=""` if decorative)
- [ ] All form inputs have associated labels
- [ ] Error messages are programmatically associated and announced
- [ ] Color contrast meets WCAG AA (4.5:1 text, 3:1 UI)
- [ ] No information conveyed by color alone
- [ ] Page works at 400% zoom without horizontal scroll
- [ ] `prefers-reduced-motion` respected
- [ ] Tested with at least one screen reader (VoiceOver or NVDA)
- [ ] Automated axe scan passes with zero critical/serious issues

---

## Further Reading

- **[W3C WAI](https://www.w3.org/WAI/)** — official specs, tutorials, techniques
- **[WCAG 2.2 Quick Reference](https://www.w3.org/WAI/WCAG22/quickref/)**
- **[ARIA Authoring Practices Guide (APG)](https://www.w3.org/WAI/ARIA/apg/)** — patterns with code examples
- **[WebAIM](https://webaim.org/)** — articles, screen reader surveys, contrast checker
- **[A11y Project](https://www.a11yproject.com/)** — community-maintained checklist
- **[Inclusive Components](https://inclusive-components.design/)** by Heydon Pickering
- **[Deque University](https://dequeuniversity.com/)** — paid courses, free reference
- **WebAIM Screen Reader User Survey** — published every 1–2 years; invaluable real-world data on AT usage

---

> **Closing thought:** accessibility is a **practice**, not a feature. The teams I've seen ship genuinely inclusive products aren't the ones that hit 100 on Lighthouse — they're the ones who consistently ask, *"Who might we be excluding, and how do we fix that?"* Every checklist on this page is downstream of that question. Treat the question as the contract, and the rest takes care of itself over time.
