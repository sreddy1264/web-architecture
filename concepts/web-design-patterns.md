# Web Design Patterns

*Last reviewed: 2026-05*

> A practical catalog of patterns used to build modern web applications — what they are, when to reach for them, and what they cost.

---

## Table of Contents

1. [What is a Design Pattern?](#what-is-a-design-pattern)
2. [Why Patterns Matter](#why-patterns-matter)
3. [How to Categorize Web Patterns](#how-to-categorize-web-patterns)
4. [Architectural Patterns](#architectural-patterns)
5. [Component & UI Patterns](#component--ui-patterns)
6. [State Management Patterns](#state-management-patterns)
7. [Data Fetching & Async Patterns](#data-fetching--async-patterns)
8. [Performance Patterns](#performance-patterns)
9. [Module & Structural Patterns (Classic GoF)](#module--structural-patterns-classic-gof)
10. [Communication Patterns](#communication-patterns)
11. [Resilience & Failure Patterns](#resilience--failure-patterns)
12. [Anti-Patterns to Avoid](#anti-patterns-to-avoid)
13. [How to Decide: Choosing the Right Pattern](#how-to-decide-choosing-the-right-pattern)
14. [Pattern Composition Examples](#pattern-composition-examples)
15. [Further Reading](#further-reading)

---

## What is a Design Pattern?

A **design pattern** is a reusable, name-able solution to a recurring problem in software design — codified vocabulary that lets engineers communicate complex ideas in a few words.

> "Each pattern describes a problem which occurs over and over again in our environment, and then describes the core of the solution to that problem, in such a way that you can use this solution a million times over, without ever doing it the same way twice."
> — Christopher Alexander (originator), via *Design Patterns* (Gamma, Helm, Johnson, Vlissides — the "Gang of Four", 1994)

Patterns are **not** templates to copy. They're shapes that emerge from solving similar problems repeatedly. The real skill isn't memorizing patterns — it's recognizing **when a pattern's trade-offs match the problem's constraints**.

---

## Why Patterns Matter

### 1. Shared Vocabulary
Saying *"let's use a Compound Component here"* is faster and more precise than describing the structure from scratch. Patterns are the lingua franca of code review.

### 2. Encoded Trade-offs
Each pattern represents a battle-tested choice between competing forces — flexibility vs. simplicity, decoupling vs. cohesion, performance vs. readability. Using a pattern means inheriting those battle-scars.

### 3. Predictable Onboarding
New engineers ramp faster on a codebase that uses recognized patterns vs. one that's idiosyncratic. "It's a repository pattern" tells them ten things at once.

### 4. Refactoring Targets
Patterns are also *destinations*. When code starts to smell, you often refactor *toward* a pattern (Strategy, State Machine, Repository).

### 5. They Encode Hard-Won Lessons
"Don't put business logic in components" sounds like dogma until you've maintained a 200-component codebase that ignored it. Patterns are scar tissue.

> **A caveat upfront:** patterns are a tool, not a goal. Pattern-itis ("we need a Factory here, and an Observer there") is its own anti-pattern. The right answer is often plain code.

---

## How to Categorize Web Patterns

Patterns operate at different scopes:

| Scope | Examples | Concern |
|---|---|---|
| **Architectural** | MVC, MVVM, Clean, Hexagonal, Micro-frontends | App-wide structure |
| **Component / UI** | Compound Components, Render Props, Hooks, Headless | UI composition |
| **State** | Flux/Redux, Atomic, State Machine, Observer | How state is shaped, mutated, observed |
| **Data fetching** | Render-as-you-fetch, SWR, Repository, BFF | How data flows from server to UI |
| **Performance** | Memoization, Virtualization, PRPL, Code-splitting | Optimizing rendering & loading |
| **Module / Structural (GoF)** | Module, Factory, Singleton, Adapter, Proxy | Code-level organization |
| **Communication** | Pub/Sub, Mediator, Event Bus, Message Queue | Decoupling components |
| **Resilience** | Circuit Breaker, Retry, Bulkhead, Fallback | Handling failure |

A typical app uses ~15 patterns simultaneously, layered. Knowing them lets you read a codebase like a sentence instead of a wall of text.

---

## Architectural Patterns

### 1. MVC — Model-View-Controller
The classic split: **Model** (data + business logic), **View** (UI), **Controller** (orchestration).

**Pros:** clear separation, well-understood, dominant in server-side frameworks (Rails, Django, Laravel, ASP.NET).
**Cons:** in modern client-side apps, "controller" gets fuzzy; views often hold logic; degenerates into "Massive View Controller" without discipline.
**When to use:** server-rendered apps, traditional CRUD systems, monolithic backends.

### 2. MVVM — Model-View-ViewModel
Like MVC, but the **ViewModel** transforms model data into view-shaped data with two-way binding.

**Pros:** clean separation between business and presentation logic; testable view models.
**Cons:** two-way binding can hide complexity; popular in Knockout/Angular 1.x but less idiomatic in modern React.
**When to use:** WPF/Vue/Angular ecosystems; complex forms with derived state.

### 3. Flux / Unidirectional Data Flow
**Action → Dispatcher → Store → View → Action**. State changes flow in one direction; views never mutate state directly.

**Pros:** predictable state, easy to debug, great fit for React.
**Cons:** more boilerplate than two-way binding for simple cases.
**When to use:** medium-to-large apps with shared state across components.

### 4. Clean Architecture (Onion / Hexagonal / Ports & Adapters)
Domain logic at the center; everything else (UI, DB, frameworks) is an outer ring depending *inward*. Outer concerns plug in through *ports* (interfaces) and *adapters*.

**Pros:** business logic is framework-agnostic, deeply testable, swappable infrastructure.
**Cons:** high upfront cost; over-engineered for small apps; lots of indirection.
**When to use:** long-lived business systems, multi-frontend (web + mobile + CLI), regulated domains.

### 5. Layered Architecture
Presentation → Application → Domain → Infrastructure. Each layer only depends on the one beneath it.

**Pros:** easy to reason about; conventional and broadly understood.
**Cons:** strict layering can cause "anemic" middle layers; cross-cutting concerns awkward.
**When to use:** enterprise apps, large teams with junior members who benefit from rigid structure.

### 6. Micro-frontends
Splitting a frontend monolith into independently-deployable apps composed at runtime (Module Federation) or build time.

**Pros:** team autonomy, independent deploys, tech diversity (theoretically).
**Cons:** **massive complexity** — bundle duplication, design inconsistency, cross-app navigation, shared auth, observability split. Rarely worth it under 100 frontend engineers.
**When to use:** very large orgs with strict team boundaries; *not* "we want to use React and Vue together."

### 7. JAMstack
**JavaScript + APIs + Markup**. Pre-built markup served from a CDN, dynamic via JS calling APIs.

**Pros:** speed, security (no server attack surface), cheap hosting, scales infinitely.
**Cons:** real-time / per-user content needs creative API design; build times explode at scale.
**When to use:** content-led sites, marketing, docs, e-commerce with ISR.

### 8. BFF — Backend-for-Frontend
A thin server tailored to a single frontend (web vs. iOS vs. Android), aggregating microservices and shaping data for that UI.

**Pros:** decouples UI from microservice contracts, keeps frontends thin, enables per-platform optimization.
**Cons:** another deployable to operate; can drift from underlying services.
**When to use:** when one shared "API gateway" is forcing every frontend to over-fetch and post-process.

---

## Component & UI Patterns

### 1. Container / Presentational (Smart / Dumb)
**Container** components handle data and state; **presentational** components are pure, props-only, and reusable.

**Pros:** clear separation of concerns; presentational components are trivially testable and storyboard-able.
**Cons:** the rigid split is less idiomatic with hooks (a hook can replace a container). Modern React often uses *colocated* logic instead.
**When to use:** when reusability across data sources matters; a useful mental model even if the strict pattern is fading.

### 2. Compound Components
A parent component implicitly shares state with its children via context, letting consumers compose flexibly.

```jsx
<Tabs defaultValue="a">
  <Tabs.List>
    <Tabs.Trigger value="a">A</Tabs.Trigger>
    <Tabs.Trigger value="b">B</Tabs.Trigger>
  </Tabs.List>
  <Tabs.Panel value="a">…</Tabs.Panel>
  <Tabs.Panel value="b">…</Tabs.Panel>
</Tabs>
```

**Pros:** flexible API, declarative, mirrors how HTML works (`<select><option>`).
**Cons:** state is implicit (via context); harder to type fully; consumers can mis-arrange children.
**When to use:** complex widgets with multiple coordinated parts (Tabs, Menu, Accordion, Disclosure, Dialog). Used heavily by Radix UI, React Aria, Headless UI.

### 3. Render Props
A component calls a function passed as a prop, providing arguments the consumer renders with.

```jsx
<MouseTracker render={({x, y}) => <Cursor x={x} y={y} />} />
```

**Pros:** highly flexible, composable.
**Cons:** "callback hell" with nesting; mostly superseded by hooks in React.
**When to use:** rarely in 2026 React; common in older libraries (Formik v1, Downshift) and outside React.

### 4. Higher-Order Components (HOCs)
A function that takes a component and returns a new wrapped component (`withAuth(Page)`).

**Pros:** reusable cross-cutting behavior (auth gates, analytics, theming).
**Cons:** wrapper hell, prop name collisions, opaque types, debugging stack traces. Mostly replaced by hooks.
**When to use:** sparingly today. Hooks or context providers are usually cleaner.

### 5. Custom Hooks
Encapsulate stateful logic in a named function that components consume (`useAuth()`, `usePagination()`).

**Pros:** the *modern* way to share logic in React; composes naturally; testable in isolation.
**Cons:** hooks have their own rules (always call at top level; only in components/hooks); deep hook chains can hide complexity.
**When to use:** the default. If you're tempted to write an HOC or render prop in React, write a hook instead.

### 6. Headless Components
The component manages **state and behavior** but renders **nothing** (or minimal markup); the consumer brings the UI.

```jsx
const { isOpen, getButtonProps, getMenuProps, getItemProps } = useCombobox({...});
return (
  <>
    <input {...getButtonProps()} />
    <ul {...getMenuProps()}>{items.map(item => <li {...getItemProps({item})}>...</li>)}</ul>
  </>
);
```

**Pros:** gives you accessibility + behavior for free, while letting you fully control markup and styles. Industry-standard now (React Aria, Headless UI, Radix UI primitives, TanStack Table).
**Cons:** more verbose than a styled all-in-one; learning curve.
**When to use:** any time you need a custom-styled but accessible widget. Don't roll your own combobox/menu/dialog.

### 7. Provider Pattern
A context (or DI container) provides values to a subtree without prop drilling.

**Pros:** clean dependency injection; perfect for theme, auth, locale, feature flags.
**Cons:** every consumer re-renders on context change unless split carefully; over-use causes implicit coupling.
**When to use:** truly cross-cutting concerns, not for everyday data.

### 8. Slots Pattern
Named "holes" in a layout that consumers fill (`children`, `<header slot>`, `<aside slot>`).

**Pros:** flexible composition without prop bloat.
**Cons:** ordering/typing harder than a structured props API.
**When to use:** layout components, modal composition, page templates.

### 9. Atomic Design
A methodology for organizing components: **Atoms → Molecules → Organisms → Templates → Pages**.

**Pros:** shared vocabulary for design + engineering; aligns with design systems.
**Cons:** the categorization is fuzzy ("is a CardHeader a molecule or an organism?"); enforced rigidly, becomes bureaucratic.
**When to use:** as a *guideline* in design systems; not a strict folder enforcement.

### 10. Controlled vs. Uncontrolled Components
- **Controlled**: parent owns the state (`<input value={v} onChange={setV} />`).
- **Uncontrolled**: component owns state internally; parent reads via ref or callback.

**Pros (controlled):** predictable, fully integrable with form libraries; single source of truth.
**Pros (uncontrolled):** less re-rendering, simpler for one-off forms.
**When to use:** controlled by default; uncontrolled for performance-sensitive forms or third-party widget integration.

---

## State Management Patterns

### 1. Local Component State
`useState`, `useReducer`. State lives where it's used.

**Pros:** simplest, no global infra, easy to reason about.
**Cons:** prop drilling and lifting state get painful past 2-3 levels.
**When to use:** the default. Don't reach for global state until local stops working.

### 2. Lifted / Lifted + Context
Lift to nearest common ancestor; if too deep, put it in context.

**Pros:** no library; uses framework primitives.
**Cons:** context re-renders everything that consumes it; not built for high-frequency updates.
**When to use:** themes, auth, locale, low-churn shared state.

### 3. Flux / Redux
Single global store, pure reducers, dispatched actions, immutable updates.

**Pros:** predictable, time-travel debugging, mature ecosystem (RTK, Reselect, Redux Toolkit Query), great for complex domains.
**Cons:** boilerplate (less so with Redux Toolkit); a single global store can become a junk drawer.
**When to use:** large apps with complex shared state, undo/redo, audit trails, multi-user collaboration.

### 4. Atomic State (Recoil, Jotai)
State as a graph of small **atoms**; components subscribe to the atoms they need.

**Pros:** fine-grained reactivity (only subscribers re-render), great DX for derived state.
**Cons:** distributed state can be hard to trace; tooling less mature than Redux.
**When to use:** apps with lots of independent, interrelated state (form-heavy apps, complex dashboards).

### 5. Proxy-based / Signals (Zustand, Valtio, Solid signals, Vue refs, Preact signals)
Reactive primitives that track reads automatically; updates only re-render dependents.

**Pros:** minimal boilerplate, fine-grained reactivity, intuitive mental model.
**Cons:** "magic" can confuse; debugging requires understanding the proxy.
**When to use:** modern React apps wanting Redux power without the ceremony (Zustand is hugely popular for this).

### 6. State Machines (XState)
Explicitly model states and transitions as a finite-state machine or statechart.

**Pros:** impossible states become *impossible*; visualizable; great for complex flows (checkout, onboarding, video player, auth).
**Cons:** learning curve; overkill for simple toggles.
**When to use:** any UI with > 4 distinct states or complex transition rules. Once you've used a state machine for a tricky flow, you'll never go back to booleans.

### 7. Server State vs. Client State
A critical modern distinction:
- **Server state**: data that lives on the server, cached on the client (lists, profiles, posts).
- **Client state**: ephemeral UI state (modal open?, form draft, theme).

Treat them differently:
- **Server state** → React Query (TanStack Query), SWR, RTK Query, Apollo, Relay.
- **Client state** → useState, Zustand, Jotai.

**In practice:** the biggest state-management win in the last decade was realizing **most "global state" was actually server state poorly cached**. Once React Query/SWR took over data fetching, Redux store sizes shrank by 80%.

### 8. Event Sourcing
State is the result of replaying an immutable log of events.

**Pros:** complete audit trail, time travel, reactive projections.
**Cons:** complex; rarely needed in pure frontend.
**When to use:** collaborative editors (CRDTs), domain-event-driven backends, audit-heavy systems.

---

## Data Fetching & Async Patterns

### 1. Fetch-on-Render
Component mounts → triggers fetch → loading state → re-renders with data.

**Pros:** simple, colocated.
**Cons:** **waterfalls** — child fetches don't start until parent renders.
**When to use:** simple cases; avoid for nested, dependent data.

### 2. Fetch-Then-Render
Start all fetches before rendering; show UI when data arrives.

**Pros:** no waterfalls, predictable LCP.
**Cons:** users wait for the slowest fetch before seeing anything.
**When to use:** small apps with few fetches.

### 3. Render-as-You-Fetch (Suspense + Streaming)
Fetches start *outside* render; the component reads the result via Suspense; the framework streams partial HTML.

**Pros:** parallel fetches, progressive UI, no waterfalls. The default in modern Next.js, Relay, Remix.
**Cons:** requires Suspense-aware data layer (RSC, Relay, TanStack Query v5+, Apollo).
**When to use:** modern React apps. This is where the ecosystem is heading.

### 4. SWR (Stale-While-Revalidate)
Show cached data instantly; revalidate in background; update UI if changed.

**Pros:** instant perceived performance, automatic deduplication, focus refetch, optimistic UI.
**Cons:** requires understanding cache keys and invalidation.
**When to use:** any client-side data layer. Powers React Query, SWR, RTK Query.

### 5. Repository Pattern
A layer that abstracts data sources behind a domain-shaped API.

```ts
class UserRepository {
  findById(id: string): Promise<User> { /* fetches, caches, transforms */ }
}
```

**Pros:** testable, swappable backend, isolates network code.
**Cons:** an abstraction tax for simple apps.
**When to use:** large apps, multi-source data (REST + GraphQL + cache + local DB).

### 6. Optimistic UI
Update UI immediately on user action; reconcile when server responds; rollback on error.

**Pros:** snappy perceived performance.
**Cons:** rollback UX is hard; conflict resolution non-trivial.
**When to use:** likes, follows, simple writes; not for transactions where wrong state is dangerous.

### 7. Polling vs. Long Polling vs. SSE vs. WebSockets
| Pattern | Use case |
|---|---|
| **Polling** | Simple, infrequent updates; works everywhere |
| **Long Polling** | Real-time-ish without WebSocket infra; legacy fallback |
| **Server-Sent Events (SSE)** | Server → client streams (notifications, live data); simpler than WS |
| **WebSockets** | Full duplex, low-latency (chat, multiplayer, collab editors) |
| **WebTransport** | Modern HTTP/3-based bidi streaming (still maturing) |

### 8. GraphQL
Client specifies the shape of the data; server responds in that shape.

**Pros:** end of over/under-fetching, strong typing, great tooling.
**Cons:** server complexity (N+1s, caching, auth at field level); not a silver bullet.
**When to use:** many heterogeneous clients querying the same domain; large product surface.

---

## Performance Patterns

### 1. Memoization
Cache the result of an expensive computation keyed on inputs (`useMemo`, `React.memo`, reselect, lodash memoize).

**Pros:** avoids redundant work.
**Cons:** memoization isn't free — equality checks cost CPU; over-memoization adds noise without measurable wins.
**When to use:** measurable hot paths. Profile first.

### 2. Virtualization (Windowing)
Render only the visible portion of a long list; recycle DOM nodes as the user scrolls.

**Pros:** handles 100K-row tables without freezing.
**Cons:** breaks Ctrl-F (browser find), accessibility nuances, scroll jank if mis-tuned.
**When to use:** any list > ~200 items. Libraries: TanStack Virtual, react-window, react-virtuoso.

### 3. Code-Splitting
Break the JS bundle into chunks loaded on demand (per route, per feature).

**Pros:** smaller initial download = faster LCP/INP.
**Cons:** lazy-loading mid-interaction can cause perceptible delays; needs preloading strategy.
**When to use:** any non-trivial app. Default for routes; selective for heavy components.

### 4. PRPL Pattern
**Push, Render, Pre-cache, Lazy-load.**
- **Push** critical resources for the initial route
- **Render** the initial route ASAP
- **Pre-cache** remaining routes
- **Lazy-load** on demand

**When to use:** PWAs, performance-critical apps.

### 5. Resource Hints
`<link rel="preload">`, `preconnect`, `dns-prefetch`, `prefetch`, `modulepreload`, `prerender`.

Use when:
- **preload** → critical LCP image, fonts
- **preconnect** → known third-party origins (CDN, analytics, fonts)
- **prefetch** → likely-next-page resources
- **prerender** (via Speculation Rules) → likely-next-page entire document

**Caveat:** preload everything = preload nothing. Be selective.

### 6. Debouncing & Throttling
- **Debounce**: wait until the user stops doing something (search inputs).
- **Throttle**: cap frequency (scroll handlers, resize).

**When to use:** any high-frequency event that triggers expensive work.

### 7. Lazy Loading
`loading="lazy"` for images/iframes; `import()` for components. Defer until needed.

**Caveat:** never lazy-load the LCP image.

### 8. Critical Rendering Path Optimization
Inline critical CSS, defer non-critical, use `font-display`, eliminate render-blocking resources.

**When to use:** always for above-the-fold content.

### 9. Web Workers / OffscreenCanvas
Move CPU-bound work off the main thread.

**Pros:** keeps UI responsive (helps INP).
**Cons:** message-passing overhead, no DOM access in workers.
**When to use:** parsing large JSON/CSV, image processing, crypto, search indexing.

### 10. Service Workers / PWA
A worker that intercepts network requests, enabling offline, caching strategies, push notifications.

**Pros:** offline-first UX, fine-grained caching, app-like installability.
**Cons:** cache-update bugs are notoriously hard to debug; lifecycle complexity.
**When to use:** offline-capable apps, performance-critical content sites (with care), installable PWAs.

---

## Module & Structural Patterns (Classic GoF)

The Gang of Four patterns that still show up daily in JS/TS:

### Creational

#### Factory
Encapsulate object creation behind a function; consumers don't `new` directly.
**Use:** when construction is complex or varies by context (per-environment loggers, per-region clients).

#### Singleton
One instance, globally accessible.
**Use:** sparingly. Module-level constants in JS often serve the same purpose without the testability hit. Singletons are a known testability anti-pattern in OO; treat with suspicion.

#### Builder
Chain method calls to construct a complex object.
**Use:** query builders, fluent APIs (Knex, Kysely, Drizzle).

### Structural

#### Module
Encapsulate state + functions in a closure; expose only the public API. The foundation of JS module systems.

#### Adapter
Wrap an interface to match another. Useful for integrating third-party libraries with your app's idioms.

#### Decorator
Wrap an object/function to add behavior without modifying it. JS class decorators (TC39 Stage 3); HOFs that wrap functions for logging, retry, caching.

#### Facade
A simplified interface over a complex subsystem (a `paymentService.charge()` hiding Stripe + DB + audit + email).

#### Proxy
A stand-in that controls access to another object. JS `Proxy` powers MobX/Vue/Valtio reactivity.

#### Composite
Treat individual and composed objects uniformly (the React tree itself is a composite).

### Behavioral

#### Observer / Pub-Sub
Subjects emit events; observers subscribe. Events, EventEmitter, RxJS, DOM events.

#### Strategy
Encapsulate interchangeable algorithms behind a common interface (sort strategies, pricing strategies, validators).

#### Command
Encapsulate a request as an object — supports undo/redo, queueing, logging.

#### State (Object)
Encapsulate per-state behavior in objects; the host delegates to its current state. Closely related to state machines.

#### Iterator
Generators / `Symbol.iterator` / async iterators — first-class in modern JS.

#### Mediator
A central object coordinates interactions between many components, replacing many-to-many with one-to-many. Redux store, message buses.

#### Chain of Responsibility
Pass a request along a chain until something handles it. Express middleware is the canonical web example.

---

## Communication Patterns

### 1. Pub-Sub (Event Bus)
Components emit events to a bus; others subscribe. Decouples publishers from subscribers entirely.

**Pros:** loose coupling, easy fan-out.
**Cons:** flow becomes implicit and hard to trace; "where does this event come from?" debugging hell.
**When to use:** truly cross-cutting events (analytics, telemetry, feature flag changes). Avoid for app data flow.

### 2. Mediator
A central coordinator instead of point-to-point. Reduces coupling between many components.
**When to use:** complex coordination (chat rooms, multi-step wizards).

### 3. CQRS — Command Query Responsibility Segregation
Separate read models from write models.
**Pros:** independent scaling, optimization.
**Cons:** complexity, eventual consistency.
**When to use:** read-heavy apps with complex aggregation, alongside event sourcing.

### 4. Postal / postMessage / Channel
Cross-frame, cross-window, cross-worker communication via `window.postMessage` or `BroadcastChannel`.
**When to use:** iframe integration, multi-tab synchronization, worker boundaries.

---

## Resilience & Failure Patterns

### 1. Retry with Exponential Backoff
Retry transient failures with increasing delay; add jitter to avoid thundering-herd retries.

```ts
async function retry(fn, { tries = 3, base = 200 } = {}) {
  for (let i = 0; i < tries; i++) {
    try { return await fn(); }
    catch (e) {
      if (i === tries - 1) throw e;
      const delay = base * 2 ** i + Math.random() * 100;
      await new Promise(r => setTimeout(r, delay));
    }
  }
}
```

**Use:** flaky network calls. **Don't:** retry non-idempotent writes without an idempotency key.

### 2. Circuit Breaker
After N consecutive failures, stop calling the failing service; periodically test with one request before reopening.

**Use:** dependent services that may be down; prevents cascading failure.

### 3. Bulkhead
Isolate resources so one slow dependency can't exhaust the pool used by others (separate connection pools, separate worker queues).

**Use:** services with multiple downstream dependencies of varying reliability.

### 4. Timeout
**Always** set a timeout. Default network behavior is "wait forever" — unacceptable in UI.

### 5. Fallback / Graceful Degradation
If primary fails, use cached / stale / simpler data ("Recommendations unavailable; showing trending instead").

**Use:** non-critical UI. Plan the degraded experience deliberately.

### 6. Error Boundaries (React)
Catch render errors in a subtree; show a fallback UI; log to monitoring.

**Use:** wrap risky subtrees (third-party widgets, dynamic content). React 19 / Next.js have route-level error boundaries built-in.

### 7. Idempotency Keys
Client-generated unique key per write; server deduplicates if seen. Critical for payments and any "submit twice" scenario.

### 8. Rate Limiting (Client-side)
Throttle outgoing requests (search-as-you-type, autosave). Reduces backend load and your own bill.

---

## Anti-Patterns to Avoid

### 1. God Component
A 2000-line component that does everything. Symptom of missing abstractions. Decompose by responsibility.

### 2. Prop Drilling Through 6 Levels
Pass-through props that no intermediate component uses. Use context, composition, or move state down.

### 3. Premature Abstraction
Building a "flexible" generic system before you have three concrete use cases. **Rule of three**: extract the abstraction on the third occurrence, not the first.

### 4. Spaghetti useEffect
Multiple `useEffect`s with overlapping deps modifying shared state. Almost always: a state machine wants to be born here.

### 5. Boolean Explosion
`isLoading`, `isError`, `isSuccess`, `isStale`, `isRetrying` — 2^5 = 32 possible states, most invalid. Use a discriminated union or state machine.

```ts
// ❌
const [loading, setLoading] = useState(false);
const [error, setError] = useState(null);
const [data, setData] = useState(null);

// ✅
type State =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: Data }
  | { status: 'error'; error: Error };
```

### 6. Singleton Abuse
Using singletons everywhere because "globals are convenient" — destroys testability. Prefer dependency injection or module-scope factories.

### 7. Magic Strings / Numbers
`if (status === 'a3')` — what's `'a3'`? Use enums or const objects with semantic names.

### 8. Mutable Shared State
Accidentally mutating a Redux store, props, or context value. React's reconciliation can't see mutations; bugs that look like "stale UI" are often this.

### 9. Anemic Domain Model
Models are just bags of properties; all logic lives in services. The opposite of OO. Sometimes fine (functional style), but if business invariants live in scattered services, refactor toward richer models.

### 10. Anchor / Useless Wrapper Divs
`<div><div><div>...` purely for styling needs that flex/grid would solve. Inflates DOM and rendering cost.

### 11. Reinvented Wheels
Custom date pickers, modals, comboboxes, drag-drop, virtual lists. **Use a headless library.** Your handcrafted version will be inaccessible, buggy, and a maintenance burden.

### 12. Cargo-Culting Patterns
"Add a Factory because someone at my last job used Factories." Patterns are tools for problems; identify the problem first.

### 13. Pattern-itis
Over-applying patterns until simple things require five files, three interfaces, and a dependency injection container. The pattern should be invisible compared to the problem it solves.

---

## How to Decide: Choosing the Right Pattern

### Step 1: Diagnose the actual pain
- Is this *really* a pattern problem, or a missing test, missing type, missing abstraction in the code I already have?
- "We're using too much state" vs. "we have one component with 14 useStates that should be a reducer."

### Step 2: Match pattern to forces
Each pattern resolves competing forces. Identify yours:

| If you need... | Reach for... |
|---|---|
| Decoupling event source from listeners | Pub-Sub / Observer |
| Interchangeable algorithms at runtime | Strategy |
| Many states with complex transitions | State Machine |
| Cross-cutting behavior (auth, logging) | HOC / Hook / Decorator |
| Independently-testable business logic | Repository / Clean Architecture |
| Shared logic across components | Custom Hook |
| Coordinated multi-part UI widget | Compound Components |
| Customizable behavior with custom UI | Headless Component |
| Server data caching + invalidation | SWR / React Query |
| Avoiding waterfalls | Render-as-you-fetch + Suspense |
| Surviving flaky dependencies | Circuit Breaker + Retry + Timeout |
| Optimistic UX | Optimistic UI + reconciliation |

### Step 3: Apply the trade-off lens
Every pattern trades simplicity for some other property (flexibility, testability, performance, reusability). Be explicit about *which* property and why it's worth the cost *here*.

### Step 4: Apply the rule of three
Don't extract a pattern on the first occurrence. Wait for the third. Two similar things might be coincidence; three is a pattern worth abstracting.

### Step 5: Pick the lowest-power tool that solves the problem
Local state before context, context before global store, global store before state machine, state machine before event sourcing. Add power only as needed.

### Step 6: Document the *why*
A pattern's rationale fades the moment it ships. Leave a one-paragraph note in the code or ADR (Architecture Decision Record): *what alternatives were considered, why this won, what we'll do if assumptions change*.

---

## Pattern Composition Examples

Real apps stack many patterns. A few examples:

### A modern e-commerce product page
- **Architecture:** Layered (UI → BFF → domain services)
- **Rendering:** ISR (page) + PPR (cart-count island)
- **State:** Local for UI; React Query for product/cart data; Zustand for ephemeral cart-drawer open state
- **Component:** Compound `<ProductGallery>` (Image, Thumbnail, FullScreen)
- **Data:** Render-as-you-fetch with Suspense for recommendations; SWR for inventory polling
- **Performance:** Code-split checkout flow; preload hero image; virtualize review list
- **Resilience:** Circuit breaker on recommendations service; fallback to "trending" if down

### An admin dashboard
- **Architecture:** SPA + BFF
- **Rendering:** CSR shell, SSR for the login wall
- **State:** React Query for entity data; XState for the multi-step editor flow; Zustand for command palette state
- **Component:** Headless table (TanStack Table); Compound filter components; Provider for permissions
- **Performance:** Virtualization for tables; debounced search; lazy routes
- **Resilience:** Optimistic mutations with reconcile-on-error; retry with backoff on transient failures

### A docs site
- **Architecture:** JAMstack + headless CMS
- **Rendering:** SSG + Islands (Astro)
- **State:** Almost none (mostly static)
- **Component:** Slot pattern for layouts; headless search widget
- **Performance:** Critical CSS inlined; preload font; speculation rules to prerender next page

---

## Further Reading

### Foundational
- **Gamma, Helm, Johnson, Vlissides — *Design Patterns* (1994)** — the GoF book; dated examples but timeless ideas
- **Christopher Alexander — *A Pattern Language*** — origin of "pattern language"
- **Eric Evans — *Domain-Driven Design***
- **Robert C. Martin — *Clean Architecture***
- **Vaughn Vernon — *Implementing Domain-Driven Design***

### Web / JavaScript-specific
- **[patterns.dev](https://patterns.dev/)** — Lydia Hallie & Addy Osmani; modern web patterns with code
- **Addy Osmani — *Learning JavaScript Design Patterns* (2nd ed., 2023)** — free online
- **Kent C. Dodds — Epic React, Epic Web** — practical React patterns
- **[The Software Engineer's Guidebook](https://www.engguidebook.com/)** — Gergely Orosz
- **[refactoring.guru](https://refactoring.guru/design-patterns)** — interactive GoF + refactoring catalog

### Architecture
- **Sam Newman — *Building Microservices***, ***Monolith to Microservices***
- **Martin Fowler — [martinfowler.com](https://martinfowler.com)** — Bliki & enterprise patterns
- **Vlad Khononov — *Learning Domain-Driven Design***
- **[microfrontends.com](https://micro-frontends.org/)** — when (and when not) to use them

### State & Data
- **[TanStack Query docs](https://tanstack.com/query)** — modern server-state mental model
- **[XState docs](https://stately.ai/docs)** — state machines for UI
- **David Khourshid's talks** — "No, disabling a button is not app logic" (state machines)

### Resilience
- **Michael Nygard — *Release It!*** — circuit breakers, bulkheads, the foundational text
- **Google SRE Book / Workbook** — free online; resilience at scale

---

## Key terms

This guide is the canonical home for these glossary entries: **Compound Components**, **Render Props**, **Higher-Order Component (HOC)**, **Provider Pattern**, **Observer Pattern**, **Singleton**, **MVC / MVVM**, **State Machine**, **Repository Pattern**, **Strategy Pattern**, **Adapter / Facade**. See the [glossary](glossary.md) for definitions and the rest of the index.

---

> **Closing thought:** patterns are vocabulary, not virtue. The goal isn't to use *more* patterns; it's to use the *right* pattern, *understood at the right depth*, applied where it earns its keep. The engineers I most enjoy working with know fifty patterns and reach for five — and they know exactly when to write plain code that solves the problem, name an exit, and move on. If a pattern is making your code longer than the problem it solves, delete it. The rest of this catalog only earns its place in your codebase one decision at a time.
