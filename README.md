# Web Architecture

> A collection of practical guides covering modern web application development — from how the browser renders a page to how to architect, build, ship, secure, and operate the system around it.

This repo contains 20 guides organized into six themes: **platform fundamentals**, **languages**, **architecture**, **quality cross-cuts**, **frameworks**, and **process and operations**. Each guide is self-contained, opinionated, and built around concrete trade-offs rather than abstract best-practices. They cross-reference each other where it helps; otherwise they stand alone.

Every guide follows the same structure:

- A clear introduction and the *why* behind the topic
- Mindset principles before tools
- The patterns, technologies, and decisions worth knowing — with code examples and diagrams
- Anti-patterns called out by name
- A quick-reference checklist
- Curated further reading
- A short closing reflection

A cross-doc **[Glossary](concepts/glossary.md)** defines the ~440 acronyms and terms used across the guides, with back-links to where each is discussed in depth.

---

## Find by goal

If you're not sure where to start, pick the goal that matches what's actually on your plate.

- **I want to make my site faster** → [Core Web Vitals](concepts/core-web-vitals.md), [Web Optimization Techniques](concepts/web-optimization-techniques.md), [Browser Rendering Pipeline](concepts/browser-rendering-pipeline.md), [Web Rendering Strategies](concepts/web-rendering-strategies.md), [CSS Mental Models](concepts/css-mental-models-and-cheat-sheet.md) (paint vs layout vs composite).
- **I want to make my site accessible** → [Web Accessibility](concepts/web-accessibility.md), [HTML Essentials](concepts/html-essentials-and-cheat-sheet.md), [CSS Mental Models](concepts/css-mental-models-and-cheat-sheet.md) (`:focus-visible`, `prefers-reduced-motion`).
- **I want to ship safely and often** → [CI/CD](concepts/ci-cd.md), [Web Testing Strategy](concepts/web-testing-strategy.md), [Observability](concepts/observability.md).
- **I want to harden the app** → [Web Application Security](concepts/web-application-security.md), [API Design](concepts/api-design.md) (auth, idempotency), [HTML Essentials](concepts/html-essentials-and-cheat-sheet.md) (`sandbox`, CSP-friendly markup).
- **I want to architect a new system** → [Web Architectures](concepts/web-architectures.md), [Web Rendering Strategies](concepts/web-rendering-strategies.md), [Web Design Patterns](concepts/web-design-patterns.md), [API Design](concepts/api-design.md).
- **I want to set up a design system** → [CSS & Design Systems](concepts/css-and-design-systems.md), [CSS Mental Models](concepts/css-mental-models-and-cheat-sheet.md), [Web Accessibility](concepts/web-accessibility.md).
- **I want to add AI features** → [AI Integration & SDLC](concepts/ai-integration-and-sdlc.md), [API Design](concepts/api-design.md) (streaming, tool use), [Web Application Security](concepts/web-application-security.md) (prompt injection, supply chain).
- **I want to debug a frontend bug** → [Browser Rendering Pipeline](concepts/browser-rendering-pipeline.md), [JavaScript: Mental Models](concepts/javascript-mental-models-and-cheat-sheet.md), [React Internals](concepts/react-internals-and-cheat-sheet.md), [CSS Mental Models](concepts/css-mental-models-and-cheat-sheet.md) (cascade, stacking, formatting contexts).
- **I want to learn the language better** → [HTML Essentials](concepts/html-essentials-and-cheat-sheet.md), [CSS Mental Models](concepts/css-mental-models-and-cheat-sheet.md), [JavaScript: Mental Models](concepts/javascript-mental-models-and-cheat-sheet.md), [TypeScript Deep Dive](concepts/typescript-deep-dive.md), [React Internals](concepts/react-internals-and-cheat-sheet.md).
- **I want to rank in search and AI engines** → [SEO](concepts/seo.md), [Web Rendering Strategies](concepts/web-rendering-strategies.md) (JS-rendering pitfalls), [HTML Essentials](concepts/html-essentials-and-cheat-sheet.md) (semantic markup, structured data).

---

## What's in here

### Platform fundamentals
The substrate every web app runs on.

- **[Browser Rendering Pipeline](concepts/browser-rendering-pipeline.md)** — what happens between pressing Enter and seeing pixels: DNS, TCP, TLS, HTTP, the critical rendering path, reflow & repaint, the GPU compositor, and the event loop.

### Languages
The languages we actually write — with the mental models that explain their behavior.

- **[HTML Essentials & Cheat Sheet](concepts/html-essentials-and-cheat-sheet.md)** — semantic landmarks, headings, forms, tables, media (`srcset`/`sizes`/`loading`/`fetchpriority`), `<dialog>` and `popover`, `<head>` essentials, modern attributes (`inert`, `enterkeyhint`, `inputmode`), and a div-soup → semantic replacement table.
- **[CSS Mental Models & Cheat Sheet](concepts/css-mental-models-and-cheat-sheet.md)** — selectors, the box model, Flex and Grid, position, modern features (`:has`, container queries, `@scope`, anchor positioning, `oklch`, scroll-driven animations), plus the four mental models: cascade, specificity, formatting contexts, and stacking contexts.
- **[JavaScript: Mental Models & Cheat Sheet](concepts/javascript-mental-models-and-cheat-sheet.md)** — the syntax that shows up daily, plus the mental models underneath: execution context, hoisting, closures, `this`, prototype chain, coercion, the event loop, microtasks, modules, GC — and a focused DOM cheat sheet.
- **[TypeScript Deep Dive](concepts/typescript-deep-dive.md)** — strict-mode `tsconfig`, generics, conditional types, `infer`, mapped & template literal types, branded types, discriminated unions, React patterns, runtime validation, and TS performance tuning.

### Architecture
Shaping the system before the first line of code.

- **[Web Architectures](concepts/web-architectures.md)** — monolith, modular monolith, microservices, serverless, event-driven, hexagonal, CQRS, and how to pick.
- **[Web Rendering Strategies](concepts/web-rendering-strategies.md)** — CSR, SSR, SSG, ISR, streaming SSR, PPR, islands, resumability — and why the right answer is per route, not per app.
- **[Web Design Patterns](concepts/web-design-patterns.md)** — architectural, component, state, data-fetching, performance, communication, and resilience patterns.
- **[API Design](concepts/api-design.md)** — REST, GraphQL, gRPC, tRPC, webhooks, real-time, idempotency, versioning, error shapes, and API governance.

### Quality cross-cuts
Properties that don't fit on any one page but show up everywhere.

- **[Web Accessibility](concepts/web-accessibility.md)** — WCAG, ARIA, semantic HTML, keyboard and screen-reader support, and how to make inclusive products the default.
- **[SEO](concepts/seo.md)** — technical SEO, content strategy, structured data, JS-rendering pitfalls, and the rise of AI-driven search engines.
- **[Core Web Vitals](concepts/core-web-vitals.md)** — LCP, INP, CLS — measurement, optimization, and treating them as user-frustration metrics rather than Google scores.
- **[Web Optimization Techniques](concepts/web-optimization-techniques.md)** — code splitting, dynamic imports, caching at every layer, virtualization, infinite scrolling, React rerender prevention, and the broader performance playbook.
- **[Web Application Security](concepts/web-application-security.md)** — OWASP Top 10, threat modeling (STRIDE), authentication, authorization, XSS/CSRF/SQLi, security headers, CSP, supply chain, and incident response.
- **[CSS & Design Systems](concepts/css-and-design-systems.md)** — CSS architecture, modern features (`:has`, container queries, `@layer`, view transitions, `oklch`), design tokens, theming, design-system patterns, and critical CSS.

### Frameworks
Deep dives on the libraries we actually ship.

- **[React Internals & Cheat Sheet (React 19)](concepts/react-internals-and-cheat-sheet.md)** — every hook and React 19 API as a quick reference, plus a tour of fibers, reconciliation, lanes, automatic batching, the commit phase, and the React Compiler.

### Process and operations
Once the code is written, the work has barely started.

- **[Web Testing Strategy](concepts/web-testing-strategy.md)** — the test pyramid/trophy/honeycomb, levels of testing (unit → integration → component → e2e → contract → visual → a11y → mutation → property-based), test doubles, flaky-test discipline, and CI integration.
- **[Observability](concepts/observability.md)** — logs, metrics, traces, RUM, profiling, OpenTelemetry, SLIs/SLOs/error budgets, alerting, dashboards, incident response, and postmortems.
- **[CI/CD](concepts/ci-cd.md)** — branching, pipeline anatomy, quality gates, deployment strategies (blue/green, canary, rolling), feature flags, database migrations, IaC, GitOps, supply-chain security, and DORA metrics.
- **[AI Integration & SDLC](concepts/ai-integration-and-sdlc.md)** — LLM integration patterns, prompt engineering, structured outputs, streaming UX, tool use, RAG, agents, on-device AI, evals, observability for AI, and the full AI SDLC.
