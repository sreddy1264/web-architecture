# Web Application Testing Strategy

> A practical guide to testing web applications — what to test, how much, with which tools, and how to keep the suite fast, reliable, and trustworthy.

---

## Table of Contents

1. [Why Testing Matters](#why-testing-matters)
2. [The Testing Mindset](#the-testing-mindset)
3. [The Shape of a Test Suite](#the-shape-of-a-test-suite)
4. [Test Pyramid vs. Trophy vs. Honeycomb](#test-pyramid-vs-trophy-vs-honeycomb)
5. [The Levels of Testing](#the-levels-of-testing)
   - [Static Analysis](#1-static-analysis)
   - [Unit Tests](#2-unit-tests)
   - [Integration Tests](#3-integration-tests)
   - [Component Tests](#4-component-tests)
   - [End-to-End Tests](#5-end-to-end-tests-e2e)
   - [Contract Tests](#6-contract-tests)
   - [Visual Regression Tests](#7-visual-regression-tests)
   - [Accessibility Tests](#8-accessibility-tests)
   - [Performance Tests](#9-performance-tests)
   - [Security Tests](#10-security-tests)
   - [Mutation Tests](#11-mutation-tests)
   - [Property-Based Tests](#12-property-based-tests)
   - [Snapshot Tests](#13-snapshot-tests)
   - [Smoke & Production Tests](#14-smoke--production-tests)
6. [Test Doubles: Mocks, Stubs, Fakes, Spies](#test-doubles-mocks-stubs-fakes-spies)
7. [TDD, BDD, ATDD](#tdd-bdd-atdd)
8. [Testing Methodology — What to Actually Test](#testing-methodology--what-to-actually-test)
9. [Test Architecture & Code Organization](#test-architecture--code-organization)
10. [Test Data Management](#test-data-management)
11. [Flaky Tests — The Perpetual Battle](#flaky-tests--the-perpetual-battle)
12. [Performance of the Test Suite](#performance-of-the-test-suite)
13. [Coverage — Use & Misuse](#coverage--use--misuse)
14. [CI/CD Integration](#cicd-integration)
15. [Tools Landscape (2026)](#tools-landscape-2026)
16. [React-Specific Testing Patterns](#react-specific-testing-patterns)
17. [Common Anti-Patterns](#common-anti-patterns)
18. [Quick Reference Checklist](#quick-reference-checklist)
19. [Further Reading](#further-reading)

---

## Why Testing Matters

### 1. Confidence to change
Tests aren't about catching bugs in the code you just wrote — they're about giving you the confidence to change code months later without re-deriving every assumption. **A test suite is institutional memory.**

### 2. Living documentation
Well-written tests describe how a system is *supposed* to behave. New engineers read tests to understand intent; old engineers read them to remember why a weird branch exists.

### 3. Design pressure
Code that's hard to test is usually hard to use. The discipline of writing tests pushes you toward smaller functions, clearer interfaces, fewer hidden dependencies, explicit contracts. **TDD's secret value is design feedback, not bug catching.**

### 4. Faster delivery, not slower
Counterintuitive but consistently measured: teams with strong test suites ship faster, not slower. The DORA research (*Accelerate*, Forsgren et al.) found "continuous testing" as one of the strongest predictors of high software delivery performance. Slow teams skip tests; fast teams rely on them.

### 5. Risk-tiered insurance
Different parts of an app carry different blast radius. Your payment flow needs more tests than your About page. Testing strategy is **risk allocation**, not uniform coverage.

> **A reframe:** the goal isn't 100% coverage. The goal is **confidence proportional to risk**. A small business-critical module deserves 95% coverage and mutation testing; a marketing page deserves a smoke test. Unequal investment is correct.

---

## The Testing Mindset

### 1. Test behavior, not implementation
Tests should survive refactors. If you renamed a variable and 30 tests broke, your tests are coupled to implementation details. **Test what the user (or caller) observes**, not how it's accomplished.

### 2. Trust correlates with cost
Kent C. Dodds' famous principle: *"The more your tests resemble the way your software is used, the more confidence they can give you."* But realistic tests are slower and harder to write. **Choose the cheapest test that gives the confidence you need** — and no cheaper.

### 3. Fast tests get run; slow tests get skipped
A 30-minute suite is no better than no suite — engineers will commit without running it. Optimize ruthlessly for speed. Sub-second feedback for unit tests, sub-minute for the full unit/integration tier.

### 4. Flaky tests poison the well
A test that passes 95% of the time is *worse* than no test — it trains the team to ignore failures. **Flake is a P0 bug.** Quarantine, fix, or delete. Never "rerun until it passes."

### 5. Tests are code; the same standards apply
Duplication, magic numbers, opaque setup, copy-paste assertions — these compound in tests faster than in source. Refactor tests aggressively. **Bad test code creates more bugs than good test code prevents.**

### 6. The strongest tests fail loudly and locally
A test that fails with "expected true to be false" is useless; a test that fails with `expected user.email to equal "alice@example.com" but got null` tells you the bug. Invest in error messages, custom matchers, focused assertions.

### 7. Coverage is a ceiling, not a floor
You can have 100% coverage and 0% confidence. You can't have 95% confidence with 30% coverage. Use coverage to find untested risky code, never as a goal.

---

## The Shape of a Test Suite

A healthy test suite is **layered, weighted, and intentional**:

```
┌────────────────────────────────────────────────────────────────┐
│                    A HEALTHY TEST SUITE                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│   E2E (browser, real backend)                                  │
│      [▓▓▓]   ~5%       ~minutes        critical user journeys  │
│                                                                │
│   Integration / API                                            │
│      [▓▓▓▓▓▓▓▓▓▓]  ~20%   ~seconds    service boundaries       │
│                                                                │
│   Component / Module                                           │
│      [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓]  ~30%   ~ms     UI behavior         │
│                                                                │
│   Unit / Pure logic                                            │
│      [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓]  ~30%  ~ms              │
│                                                                │
│   Static (TS, lint, format) — runs in seconds, finds bugs free │
│      [▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓]  ~15%        │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

Numbers are illustrative; ratios depend on app shape. The key principle: **more tests at faster, cheaper layers; fewer at slower, expensive ones.**

---

## Test Pyramid vs. Trophy vs. Honeycomb

Three popular shapes; each fits a different system.

### 1. The Test Pyramid (Mike Cohn, 2009)
```
           ▲
          ╱ ╲
         ╱E2E╲          few, slow, high-value
        ╱─────╲
       ╱integ. ╲        moderate
      ╱─────────╲
     ╱   unit    ╲      many, fast, narrow
    ─────────────
```
**Best for:** server-heavy systems where logic lives in pure functions / services.
**Limitation:** in modern frontends, many bugs live at the integration layer (component + DOM + library + browser API), not the unit layer.

### 2. The Testing Trophy (Kent C. Dodds, 2018)
```
              ▲
            ╱ E2E ╲       few
           ╱───────╲
          ╱integration╲   many ←── most value
         ╱─────────────╲
        ╱     unit       ╲ moderate
       ─────────────────────
       ╱      static       ╲ free, run constantly
      ───────────────────────
```
**Best for:** modern frontend apps. Static analysis (TypeScript, ESLint) catches a huge class of bugs for free; integration tests give the most confidence per minute. Unit tests are still useful, but for pure functions.

### 3. The Honeycomb / Diamond (Spotify, 2021)
```
        ┌─────────────┐
        │   small     │
        └──┬───────┬──┘
           │       │
        ┌──┴───────┴──┐
        │ INTEGRATION │  ← bulk of suite
        │   (medium)  │
        └──┬───────┬──┘
           │       │
        ┌──┴───────┴──┐
        │   small     │
        └─────────────┘
```
**Best for:** distributed systems, microservices, full-stack features. Most tests are integration-shaped; unit and E2E are minimized at the edges.

> **Worth knowing:** the shapes are heuristics. The right answer is **whatever distribution gives the most confidence per unit of test maintenance cost for your system.** Most modern web apps lean trophy/honeycomb; old monoliths lean pyramid.

---

## The Levels of Testing

### 1. Static Analysis
The cheapest "test" — runs without executing code.

- **Type checking (TypeScript)** — catches whole classes of bugs (typos, wrong arguments, null/undefined access). The single highest-leverage tool in modern frontend.
- **Linting (ESLint)** — code style, common bugs, framework-specific rules (`react-hooks/exhaustive-deps`, `jsx-a11y`).
- **Formatting (Prettier, dprint, Biome)** — eliminates style debates entirely.
- **Stricter sets** — `strictNullChecks`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, `noImplicitAny`. Turn them all on.
- **Dead code detection** — `ts-unused-exports`, `knip`.
- **Dependency cruiser / Madge** — enforce architectural boundaries (e.g., "UI layer cannot import from infrastructure").

**Why mature teams obsess over static analysis:** it runs constantly, finds bugs before tests run, and demands no test maintenance.

---

### 2. Unit Tests
Test a single function, class, or module **in isolation**, with all dependencies replaced by test doubles.

#### What they're good for
- Pure functions (formatters, validators, transformers, reducers, selectors)
- Algorithms with many branches and edge cases
- Business logic with rich invariants

#### What they're bad for
- Code whose value comes from integration (a component using context, hooks, APIs)
- Anything where mocking obscures the test ("I tested that I called the mock")

#### Example
```ts
// price.ts
export function applyDiscount(price: number, percent: number): number {
  if (percent < 0 || percent > 100) throw new RangeError('invalid');
  return Math.round(price * (1 - percent / 100) * 100) / 100;
}

// price.test.ts
describe('applyDiscount', () => {
  it('rounds to 2 decimal places', () => {
    expect(applyDiscount(99.99, 10)).toBe(89.99);
  });
  it('rejects negative percentages', () => {
    expect(() => applyDiscount(100, -5)).toThrow(RangeError);
  });
  it('rejects > 100%', () => {
    expect(() => applyDiscount(100, 150)).toThrow(RangeError);
  });
  it('handles 0%', () => {
    expect(applyDiscount(100, 0)).toBe(100);
  });
  it('handles 100%', () => {
    expect(applyDiscount(100, 100)).toBe(0);
  });
});
```

#### When unit tests go wrong
- Mocking everything in sight — you're testing the mock, not the behavior
- Asserting `toHaveBeenCalledWith(...)` instead of asserting the result
- Re-implementing the function under test in the test (just running it twice)

> **A heuristic worth memorizing:** if your unit test passes after deleting the function under test (because the mock matches), it's not a test.

---

### 3. Integration Tests
Test how multiple units work together — often a slice of the system: component + hook + reducer + API client (with the API mocked at the network layer, not at the function call).

This is where most modern frontend test value lives.

#### Example: an integration test of a checkout component
```jsx
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { http, HttpResponse } from 'msw';
import { server } from './test/server';
import { Checkout } from './Checkout';

test('user can submit valid order', async () => {
  server.use(
    http.post('/api/orders', () =>
      HttpResponse.json({ id: 'ord_123' }, { status: 201 })
    )
  );

  render(<Checkout cart={mockCart} />);

  await userEvent.type(screen.getByLabelText(/email/i), 'a@b.com');
  await userEvent.click(screen.getByRole('button', { name: /place order/i }));

  expect(await screen.findByText(/order ord_123 confirmed/i)).toBeVisible();
});
```

This single test covers: form rendering, label/input association (a11y), state updates, button activation logic, API request shape, success rendering. **One integration test, ten potential unit tests' worth of confidence.**

#### Tools
- **Vitest** / **Jest** + **@testing-library/react** — the dominant stack
- **MSW (Mock Service Worker)** — mock at the *network* layer, not the function level. The single most important tool in modern frontend testing.
- **happy-dom** / **jsdom** — DOM environment for Node-side tests

---

### 4. Component Tests
A subset of integration tests that focus on a single component's behavior, rendered in isolation but with realistic interactions.

```jsx
test('disabled button does not fire onClick', async () => {
  const onClick = vi.fn();
  render(<Button disabled onClick={onClick}>Submit</Button>);
  await userEvent.click(screen.getByRole('button', { name: /submit/i }));
  expect(onClick).not.toHaveBeenCalled();
});
```

#### Storybook + interaction tests
Storybook (with `@storybook/test`) lets you write component stories that double as interaction tests:
```jsx
export const SubmitWorks = {
  play: async ({ canvasElement }) => {
    const canvas = within(canvasElement);
    await userEvent.click(canvas.getByRole('button'));
    await expect(canvas.getByText(/success/i)).toBeVisible();
  },
};
```
Now your component's documentation, visual testing, and behavior tests live in one artifact. Highly leveraged.

---

### 5. End-to-End Tests (E2E)
Drive a real browser against a real (or near-real) backend through the same surface a user touches.

#### What they catch
- Routing bugs
- Auth flows
- Real-network timing issues
- Third-party integration drift
- Anything caused by gluing systems together

#### What they cost
- Slow (seconds to minutes per test)
- Flaky (network, timing, third parties)
- Hard to debug ("it failed in CI")

#### Strategic principle: **Critical user journeys only**
Don't try to E2E-test every feature. Pick 5–15 flows that, if broken, would page someone:
- Sign-up + first-time login
- Core "happy path" purchase / submission
- Password reset
- Critical admin / dangerous-state flows
- Billing / payment

#### Modern stack
- **Playwright** — current SOTA. Cross-browser (Chromium, Firefox, WebKit), built-in tracing, auto-wait, parallelization, network mocking. **The default in 2026.**
- **Cypress** — popular, opinionated, great DX; weaker on multi-tab, multi-origin scenarios.
- **WebdriverIO** — for Selenium-style flexibility.
- **Puppeteer** — lower-level; useful for scraping/automation more than testing.

#### Practices that pay off
- **Auth via API**, not UI. Logging in via the UI in every test is slow and brittle. Use API-based fixtures with cookies/tokens; only one E2E tests the actual login flow.
- **Test data is owned by the test.** Don't depend on shared seed data. Use API fixtures or DB factories with unique IDs.
- **Avoid sleeping.** Use auto-wait selectors (`page.waitForSelector`, role-based queries).
- **Run against staging, not production** — but smoke-test production after deploys.
- **Quarantine flakes** — separate suite, fix or delete; never tolerate.
- **Trace on failure** — Playwright's trace viewer is the gold standard; ship traces to CI artifacts.

---

### 6. Contract Tests
Verify that two services agree on a contract (request/response shape). Critical for microservices and BFF patterns.

#### Why?
You can mock the API in your frontend tests — but if the real API changes, the frontend breaks despite green tests.

#### Approaches
- **Consumer-Driven Contracts (Pact)** — consumer publishes expectations; provider verifies them in CI.
- **Schema-driven (OpenAPI / GraphQL / Protobuf)** — generate contracts from a shared spec; validate both sides against it.
- **Type-shared monorepos** (tRPC) — one TypeScript type bridges client and server; impossibility-by-construction.

#### When you need them
- Multiple teams own client and server independently
- API changes break consumers in production
- You're shipping public APIs

#### A nuance to keep in mind
For tightly-coupled monorepos, **shared types** beat contract tests. For loosely-coupled distributed teams, Pact or schema validation is invaluable. Don't bring contract testing without organizational pain to solve — it's significant infrastructure.

---

### 7. Visual Regression Tests
Capture screenshots; compare to a baseline; fail on pixel diffs.

#### Why
Catch CSS regressions, layout shifts, font issues, theme drift. The kind of bugs assertions can't easily express ("this is misaligned by 4 pixels").

#### Tools
- **Chromatic** (Storybook) — gold standard for component-level visual tests
- **Percy** — full-page snapshots
- **Playwright `toHaveScreenshot()`** — built-in, free, viewport-aware
- **Loki**, **BackstopJS** — open-source options

#### Pitfalls
- **Pixel diffs are noisy** — anti-aliasing, fonts, dynamic content. Use perceptual diff thresholds.
- **Dynamic content kills baselines** — mock dates, animations, randomization, ads.
- **Massive baseline drift on every redesign** — automate baseline updates; review diffs in PR like code.
- **Browser/OS variance** — pin to one rendering target (CI image), or use cloud rendering.

> **A heuristic that's stuck:** visual regression is great at the **component** level (Storybook + Chromatic). At the page level, it's high-maintenance. Most teams don't need full-page visual tests; component-level catches 90% of design regressions.

---

### 8. Accessibility Tests
Automated checks for WCAG conformance. (See [Web Accessibility](web-accessibility.md) for the full a11y picture.)

#### Tools
- **axe-core** — the engine behind every major a11y tool
- **jest-axe** / **@axe-core/playwright** — assertions in tests
- **eslint-plugin-jsx-a11y** — static-analysis level
- **Storybook a11y addon** — per-story
- **Pa11y CI** — page-level audits

```jsx
import { axe, toHaveNoViolations } from 'jest-axe';
expect.extend(toHaveNoViolations);

test('no a11y violations', async () => {
  const { container } = render(<Form />);
  expect(await axe(container)).toHaveNoViolations();
});
```

#### Reality check
Automated tools catch ~30–40% of issues. The rest — focus order, semantic correctness, screen reader experience — needs **manual testing with assistive tech**. Automated a11y testing is a floor, not a ceiling.

---

### 9. Performance Tests
Lab and field perf checks gated in CI.

#### Lab (synthetic)
- **Lighthouse CI** — assertions per route (LCP < 2.5s, JS bundle < 170KB)
- **WebPageTest API** — deeper waterfalls
- **Bundle size budgets** — `size-limit`, `bundlewatch`, `bundle-analyzer` PR comments
- **Custom microbenchmarks** — `vitest bench`, `tinybench` for hot algorithms

#### Field (RUM)
- Track Core Web Vitals per release tag
- Alert on p75 regressions within hours of deploy
- Don't block deploys on field metrics (too slow); block on lab regressions and monitor field

> **In practice:** budget enforcement in CI is the only sustainable way to keep apps fast. "We'll keep an eye on it" never works — performance debt accumulates one un-noticed PR at a time.

---

### 10. Security Tests
(See [Web Application Security](web-application-security.md) for the threat picture.)

- **SAST** (Static Application Security Testing) — Semgrep, CodeQL, Snyk Code, SonarQube
- **DAST** (Dynamic) — OWASP ZAP, Burp Suite, Nuclei
- **Dependency scanning** — Dependabot, Snyk, npm audit, OSV-scanner
- **Container scanning** — Trivy, Grype
- **Secret scanning** — gitleaks, truffleHog, GitHub secret scanning
- **IaC scanning** — Checkov, tfsec
- **Pen tests** — annual third-party engagement
- **Red team / bug bounty** — at scale

Run SAST on every PR; DAST nightly against staging; dependency scanning continuously; pen-test annually.

---

### 11. Mutation Tests
The "tests for the tests." A mutation tool changes your code (`+` → `-`, `>` → `>=`, deletes a line) and runs the suite. If tests still pass, the mutation **survived** — meaning your tests don't actually verify that behavior.

#### Tools
- **Stryker** (JS/TS) — the standard
- **PIT** (Java)

#### Why mature teams use it
Coverage tools tell you which lines were *executed*; mutation testing tells you which lines were *meaningfully tested*. A 95% coverage suite with 40% mutation score has assertion gaps.

#### Caveats
- Slow (runs the suite many times). Use sparingly — on critical modules, in nightly CI, not PR.
- Some mutants are equivalent (semantically identical to the original) — false positives.

> **A signal of test maturity:** a team that does mutation testing on critical paths has a different relationship with their test suite than one that doesn't.

---

### 12. Property-Based Tests
Instead of writing examples (`expect(reverse([1,2,3])).toEqual([3,2,1])`), specify invariants and let the framework generate hundreds of random inputs.

#### Tools
- **fast-check** (JS/TS) — the standard
- **QuickCheck** (Haskell, the original)

```ts
import fc from 'fast-check';

test('reversing twice yields the original', () => {
  fc.assert(
    fc.property(fc.array(fc.anything()), (arr) => {
      expect(reverse(reverse(arr))).toEqual(arr);
    })
  );
});
```

#### When it shines
- Pure functions with mathematical properties (sort, reverse, parse/format roundtrips)
- Reducers, validators, parsers
- Algorithms with edge cases you'd never think to write

#### Why experienced engineers love it
Finds the inputs you didn't think of: empty arrays, NaN, very long strings, Unicode, negative zero, the array `[null, undefined]`. The framework shrinks failures to a minimal counterexample. **It's a different category of confidence.**

---

### 13. Snapshot Tests
Capture serialized output; compare to stored baseline.

#### Useful for
- API response shapes (after schema changes you want to see the diff)
- Configuration objects
- Compiler / generator output
- AST transformations

#### Misused for
- Component rendering (replaced by Testing Library + visual regression)
- Anything where the snapshot is huge and unreviewable
- "Update without thinking" — the cardinal sin

#### A pragmatic approach
Use **inline snapshots** for small assertions (they live in the test file, easy to read in PR). Use file snapshots only when the output is truly large and structured. Always review snapshot diffs as carefully as code diffs.

---

### 14. Smoke & Production Tests
Tests that run *after* deploy against the real production environment.

- **Smoke tests** — does the homepage load? Does login work? Can a user place a test order? 5 minutes; runs after every deploy.
- **Synthetic monitoring** (Datadog Synthetics, Pingdom, Checkly) — runs critical paths every N minutes; pages on failure.
- **Canary checks** during rollout — abort the deploy if smoke tests fail before full rollout.

These complement (not replace) staging E2E. Production is a different environment with different data, different load, different bugs.

---

## Test Doubles: Mocks, Stubs, Fakes, Spies

Critical vocabulary that engineers confuse constantly. Gerard Meszaros's taxonomy:

| Type | Purpose |
|---|---|
| **Dummy** | Filler argument — never actually used |
| **Stub** | Returns canned data when called |
| **Spy** | A stub that records its calls for assertion |
| **Mock** | A spy with pre-programmed expectations (assertions baked in) |
| **Fake** | A working alternative implementation (in-memory DB instead of Postgres) |

### When to use which
- **Pure logic** — no doubles
- **External I/O** (network, FS, time) — fake or stub at the **boundary**
- **Verifying interaction** (sent an email, called the API) — spy
- **Strict expectations** — mock (used carefully; over-specified mocks are fragile)

### The big rule: mock at the architectural boundary
Don't mock `userRepository.findById` if the real boundary is the database. Mock the **network or process boundary** — typically with **MSW** for HTTP, with an in-memory test database for DB. Tests stay realistic; refactors don't break them.

```jsx
// ❌ Mocking the function — couples test to implementation
vi.mock('./userRepository');
vi.mocked(findUser).mockResolvedValue(user);

// ✅ Mocking the network — refactor-safe
server.use(http.get('/api/users/:id', () => HttpResponse.json(user)));
```

---

## TDD, BDD, ATDD

### TDD — Test-Driven Development
Kent Beck's red-green-refactor cycle:
1. **Red** — write a failing test
2. **Green** — write the simplest code to make it pass
3. **Refactor** — improve design while tests stay green

#### What it actually delivers
Not "tests for free" — *better designs*. Code written test-first tends to be smaller, more orthogonal, more composable. The pressure of "make this testable" forces good interfaces.

#### When it works
- Pure logic, well-understood requirements
- Refactoring or fixing bugs (write a failing test first)
- Algorithms with clear input/output

#### When it doesn't
- Exploratory UI work (you don't know what you want yet)
- Performance / experimental code
- Throwaway prototypes

> **The reality:** very few teams do strict TDD. Most experienced engineers write tests "just-in-time" — sometimes before the code, sometimes after, but always before merging. The discipline that matters is "no untested production code," not "test must precede code."

### BDD — Behavior-Driven Development
TDD with collaborative-language test descriptions (`Given/When/Then`):
```
Given a user with $50 in the cart
When they apply a 10% discount code
Then the cart total is $45.00
```

Tools: **Cucumber**, **Playwright Test** (its `test()` blocks already read like BDD).

Useful for cross-functional teams; overkill for engineering-only contexts.

### ATDD — Acceptance Test-Driven Development
Tests are written from the *user* perspective, often before development begins, in collaboration with PM/QA. Acceptance tests become the definition-of-done.

---

## Testing Methodology — What to Actually Test

### Risk-based test design
Not every test is worth writing. Prioritize by:
- **Frequency of use** — runs millions of times → test thoroughly
- **Blast radius if broken** — payment / auth → test thoroughly
- **Likelihood of regression** — branches with subtle logic → test thoroughly
- **Cost to test** — easy to test → test it

### What to test (the pragmatic version)
1. **Happy path** — the most common use case works
2. **Boundaries** — empty, one, many, off-by-one
3. **Error paths** — what happens when X fails?
4. **Edge cases** — null, undefined, NaN, very large, very small, Unicode, RTL
5. **Concurrency / ordering** — race conditions, double-submits, stale closures
6. **Security-relevant** — input validation, auth checks, IDOR
7. **Accessibility** — keyboard, screen reader, focus management
8. **Regressions** — every bug fix gets a test that would have caught it

### What NOT to test
- Trivial code (`const id = (x) => x.id`)
- Framework internals (React's reconciliation)
- Third-party libraries (test your *integration*, not their library)
- Implementation details (private methods, internal state)
- Generated code (types, RPC clients)

### The "behavior contract" framing
For each function/component, ask:
- **What does it promise to its caller?** Test that.
- **What does it depend on?** Test integration at boundaries.
- **What can go wrong inside?** Test those branches.

---

## Test Architecture & Code Organization

### Naming
Test names should read like specifications:
```
✅ describe('createOrder')
   it('rejects an empty cart')
   it('charges the customer in their billing currency')
   it('emits an order.placed event')

❌ describe('OrderService')
   it('test1')
   it('works')
```

### AAA — Arrange / Act / Assert
```ts
test('updates the cart total when an item is added', () => {
  // Arrange
  const cart = createCart({ items: [{ price: 10 }] });

  // Act
  cart.add({ price: 5 });

  // Assert
  expect(cart.total()).toBe(15);
});
```

### One assertion per test (loosely)
Not literally one `expect`, but one *concept* per test. If a test name has "and" in it, it's two tests.

### Test Data Builders / Object Mothers
Stop copy-pasting fixtures:

```ts
// ❌ Copy-paste hell
const user = { id: 1, name: 'Alice', email: 'a@b.com', role: 'user', ... };

// ✅ Builder pattern
const user = aUser().withRole('admin').build();
const oldUser = aUser().createdAt(daysAgo(365)).build();
```

Or use libraries: **Fishery**, **rosie**, **factory-girl** for JS.

### Custom matchers / domain assertions
Push complex assertions into reusable matchers:
```ts
expect(response).toBeValidOrder();
expect(html).toMatchA11yRules();
```

### Folder structure
- **Co-located** (`Foo.tsx` + `Foo.test.tsx`) — easy to find, encourages writing tests
- **Mirror tree** (`/src` + `/test` shadows) — useful for large monorepos, integration tests, e2e
- **By type** (`/unit`, `/integration`, `/e2e`) — clear separation; great for CI parallelization

---

## Test Data Management

### Strategies
- **Builders / factories** — generate per-test
- **Fixtures** — committed JSON/YAML; OK for read-only references
- **Seeded snapshots** — clear DB, run migrations, seed, run test, rollback (transactional tests)
- **Per-test isolation** — each test starts fresh (database transaction rolled back, in-memory DB recreated)

### The big rule
**Tests must not depend on order or shared state.** A test that passes in isolation but fails in the suite (or vice versa) is a flaky test waiting to happen.

### Production-like vs synthetic
- **Synthetic data** — full control, fast, no PII risk; doesn't catch edge cases real data has
- **Anonymized prod snapshots** — realistic, finds real bugs; complex to maintain, GDPR concerns
- **Pact-recorded fixtures** — record real API responses once, replay forever

---

## Flaky Tests — The Perpetual Battle

Flaky tests destroy trust faster than missing tests. Common causes:

### 1. Timing
- `await sleep(100)` — magic numbers; replace with explicit waits (`waitFor`, `findByRole`)
- Animations during tests — disable with `prefers-reduced-motion` or CSS overrides
- Debounced/throttled handlers — fast-forward fake timers

### 2. Order dependence
Tests share state through globals, the DOM, modules with caches, the file system. Reset between tests; use `beforeEach`/`afterEach`; clear all caches.

### 3. Network non-determinism
- Real network in tests → flaky. Mock with MSW.
- Even mocked: clock skew, port races, DNS. Pin everything.

### 4. Concurrency
Parallel tests stepping on shared resources (ports, files, DB rows). Use unique IDs per test, separate ports, transactional DB.

### 5. Date/time
Code that depends on `new Date()` is flaky around midnights, DST, year-ends. **Always inject the clock**:
```ts
function isWeekend(d = new Date()) { ... }   // testable
const d = new Date();                         // not testable
```
Or use `vi.setSystemTime()` / `MockDate`.

### 6. Random
Same as time. Inject the source of randomness; seed it in tests.

### 7. Network latency in E2E
Use auto-wait selectors, not sleeps. Configure retries strategically — but a test that needs 3 retries to pass is broken, not flaky.

### Process: how to handle flake
1. **Detect** — track flake rate per test in CI; surface dashboards
2. **Quarantine** — move flaky tests to a separate suite that doesn't block PRs
3. **Fix or delete** within a sprint — never leave indefinitely
4. **Post-mortem** — what design choice caused the flake? Apply to other tests.

> **A heuristic that's served me well:** every flaky test is a **design** issue (in the code or the test). The cure is not retries — it's removing the source of non-determinism.

---

## Performance of the Test Suite

A slow suite is the path to "we don't run tests locally."

### Targets (for a healthy modern app)
- **Unit + component**: total < 60s
- **Integration**: total < 5 min
- **E2E (CI parallelized)**: total < 15 min
- **PR feedback**: < 10 min total

### Tactics
- **Parallel execution** — Vitest, Jest both parallelize across workers; Playwright across shards
- **Sharding in CI** — split tests across N runners
- **Watch mode locally** — re-run only affected tests on file change
- **Test-impact analysis** — Nx affected, Turborepo, GitHub Actions paths-filter
- **Avoid global setup overhead** — DB schema once, transactions per test
- **Profile slow tests** — `--reporter verbose` or `--logHeapUsage`; ten 5-second tests bottleneck the suite
- **Prefer integration over E2E** for most behavior — 100x faster
- **Beware of `beforeEach` heavy with DOM rendering** — render once per describe if possible

---

## Coverage — Use & Misuse

### What coverage tells you
- **Statement coverage** — % of statements executed
- **Branch coverage** — % of `if`/`else`/`switch` branches taken
- **Function coverage** — % of functions called
- **Line coverage** — % of source lines executed (least useful)

### What it doesn't tell you
- Whether assertions are meaningful
- Whether edge cases are covered
- Whether bugs would be caught

### How to use it
- **Find untested risky code** — sort by file, look for low coverage on critical modules
- **Set per-module thresholds**, not global ones — `payment/` 95%, `marketing/` 50%
- **Review trends, not absolutes** — coverage shouldn't drop on PRs; numbers in isolation are noise
- **Pair with mutation testing** for true confidence

### The 100%-coverage trap
Pursuing 100% drives engineers to write trivial tests of trivial code. Quality drops. Productivity drops. **Coverage is a smoke detector, not a goal.**

---

## CI/CD Integration

### Pipeline shape
```
PR opened
   │
   ▼
┌──────────────────────────────────────────────────────────┐
│ FAST FEEDBACK (< 5 min)                                  │
│   • Lint + format                                        │
│   • Type check                                           │
│   • Unit + component tests (sharded)                     │
│   • Bundle size budget                                   │
└──────────────────────────────────────────────────────────┘
   │
   ▼ (parallel)
┌──────────────────────────────────────────────────────────┐
│ DEEP CHECKS (< 15 min)                                   │
│   • Integration tests                                    │
│   • Visual regression (component-level)                  │
│   • Accessibility                                        │
│   • SAST + dep scan                                      │
│   • Build all targets                                    │
└──────────────────────────────────────────────────────────┘
   │
   ▼
   PR can merge
   │
   ▼
┌──────────────────────────────────────────────────────────┐
│ POST-MERGE                                               │
│   • E2E on staging                                       │
│   • Lighthouse CI                                        │
│   • Deploy preview                                       │
└──────────────────────────────────────────────────────────┘
   │
   ▼
   Release
   │
   ▼
┌──────────────────────────────────────────────────────────┐
│ POST-DEPLOY                                              │
│   • Smoke tests in production                            │
│   • Synthetic monitoring (continuous)                    │
│   • RUM regression watch                                 │
└──────────────────────────────────────────────────────────┘
```

### Patterns
- **Required checks** — block merges on the must-pass set
- **Auto-merge on green** — Mergify, GitHub native
- **Test impact analysis** — only run tests affected by the PR
- **Caching** — node_modules, build cache, browser binaries
- **Artifact retention** — failed test traces, screenshots, logs
- **Flaky test reporting** — automated dashboards, GitHub action surfaces flake history
- **Parallelism shape** — split by file, not by test, to balance worker time

### Branch protection
- Required reviewers
- Required status checks
- Linear history (squash merges)
- Signed commits (in security-sensitive orgs)
- Forbid force-push to protected branches

---

## Tools Landscape (2026)

### Test runners
- **Vitest** — modern, fast, ESM-native, Vite-aligned. **The default in 2026 for Vite-based apps.**
- **Jest** — still dominant in legacy and Node-side; slower than Vitest but mature ecosystem
- **Node test runner** (`node --test`) — built-in; minimal but production-ready
- **Bun test** — extremely fast; Bun ecosystem only
- **Mocha + Chai** — flexible, older

### Component / UI
- **@testing-library/react** (and Vue/Svelte/Angular variants) — query by accessible role; the de facto choice
- **@testing-library/user-event** — realistic interactions
- **Storybook + @storybook/test** — stories as tests
- **Enzyme** — deprecated; do not use in new projects

### Network / API
- **MSW (Mock Service Worker)** — mock at the network layer; works in tests, dev, and Storybook
- **nock** — Node HTTP mocking (Node-only)
- **Pact** — consumer-driven contracts

### E2E
- **Playwright** — current SOTA
- **Cypress** — popular alternative
- **Puppeteer** — automation/scraping
- **WebdriverIO** — Selenium-based flexibility

### Visual / Accessibility
- **Chromatic** — Storybook visual regression
- **Percy** — page-level
- **axe-core** family — `@axe-core/playwright`, `jest-axe`

### Performance
- **Lighthouse CI**
- **WebPageTest API**
- **size-limit** / **bundlewatch**
- **vitest bench** / **tinybench**

### Security
- **Semgrep**, **CodeQL**, **Snyk**
- **OWASP ZAP**
- **gitleaks**, **truffleHog**

### Mutation
- **Stryker**

### Property
- **fast-check**

### Coverage
- **c8** (V8 native; used by Vitest)
- **istanbul** / **nyc** (Jest)

### CI
- **GitHub Actions**, **GitLab CI**, **CircleCI**, **Buildkite**
- **Nx**, **Turborepo** for affected-only and caching

---

## React-Specific Testing Patterns

### Query priority (Testing Library)
Use queries in this order — top is most accessible-aligned:
1. `getByRole` (with name) — what screen readers see
2. `getByLabelText` — form fields
3. `getByPlaceholderText` — fallback for unlabeled inputs (avoid)
4. `getByText` — non-interactive content
5. `getByDisplayValue`, `getByAltText`, `getByTitle`
6. `getByTestId` — last resort, when nothing semantic fits

### Avoid
- `container.querySelector('.btn-primary')` — couples to implementation
- `wrapper.find('Button')` (Enzyme-style) — replaced by accessible queries

### Testing custom hooks
```ts
import { renderHook, act } from '@testing-library/react';

test('useCounter', () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

### Testing async data
```jsx
test('shows users when loaded', async () => {
  render(<UserList />);
  expect(screen.getByText(/loading/i)).toBeVisible();
  expect(await screen.findByText(/alice/i)).toBeVisible();
});
```
`findBy*` queries auto-retry until the element appears or times out — built-in async wait.

### Testing context / providers
Wrap in test providers; create a custom render helper:
```jsx
function renderWithProviders(ui, { user, theme = 'light' } = {}) {
  return render(
    <QueryClientProvider client={new QueryClient()}>
      <ThemeProvider value={theme}>
        <UserContext.Provider value={user}>
          {ui}
        </UserContext.Provider>
      </ThemeProvider>
    </QueryClientProvider>
  );
}
```

### Testing TanStack Query / SWR
Disable retries in tests (otherwise failed queries retry 3× and slow tests):
```ts
new QueryClient({ defaultOptions: { queries: { retry: false } } });
```

### Testing Server Components
React Server Components are async functions returning JSX. Test them as functions, asserting on returned JSX (`react-test-renderer` or rendering them in a client-side test harness). Tooling here is still maturing in 2026.

### Testing forms
Prefer integration-style tests with `userEvent.type`, `userEvent.click`, real submit. Don't unit-test individual `onChange` handlers — too implementation-coupled.

---

## Common Anti-Patterns

### 1. Testing implementation details
Asserting on internal state, private methods, specific function call counts. Tests break on refactor; bugs slip through.

### 2. Mocking everything
Mocks return mocked data; "tests" pass; production breaks. Mock at the **boundary**, not at every function.

### 3. Snapshot tests of large components
Massive snapshots that nobody reviews. Just rubber-stamped on update. Switch to behavior-based tests.

### 4. The "and" test
`it('renders, fetches data, and shows the list')` — three tests masquerading as one. Split.

### 5. Sleeps to "fix" timing
`await sleep(500)` patches symptoms. Use auto-waiting queries or fake timers.

### 6. No test data isolation
Tests pollute a shared database; pass in isolation, fail together. Transactional rollback, unique IDs, fresh fixtures.

### 7. Tests as documentation gone wrong
A 3000-line `setup.ts` that nobody understands. Test setup is code; refactor it.

### 8. Coverage as the goal
"We need 80% coverage" → engineers add trivial tests of trivial code → confidence does not increase. Use coverage to find gaps in *risky* code.

### 9. Skipping flaky tests forever
`.skip()` with no follow-up = the test never runs again. Quarantine with a TTL; delete or fix.

### 10. E2E for everything
Trying to E2E-test every feature. Suite balloons to 90 minutes; flake explodes. Push tests down to integration / component layer.

### 11. Re-implementing the function under test
```ts
test('formats date', () => {
  const expected = new Date(input).toISOString().slice(0, 10);
  expect(formatDate(input)).toBe(expected);
});
```
You're testing nothing. Use a known, fixed expected value.

### 12. Testing the framework
"Test that React renders the component" — React's job, not yours. Test *your* logic.

### 13. Asserting on `console.log` or implementation details to "make sure code ran"
Useless. Either test the observable behavior or don't bother.

### 14. Not running tests in CI
Local-only test culture is no test culture. Tests run on every PR or they don't exist.

### 15. Treating tests as second-class code
Untyped, undocumented, copy-pasted, never refactored. Test code is production code; treat it accordingly.

---

## Quick Reference Checklist

### Strategy
- [ ] Test pyramid/trophy/honeycomb shape decided per app
- [ ] Critical user journeys identified (E2E coverage targets)
- [ ] Risk-tier per module (high-stakes get more tests)
- [ ] Coverage thresholds per module (not global)
- [ ] Flaky test policy documented; tracked dashboards

### Static
- [ ] TypeScript with `strict: true` (and `noUncheckedIndexedAccess`)
- [ ] ESLint + framework-specific rules; CI fails on errors
- [ ] Prettier / Biome formatting; no style debates in PR
- [ ] Pre-commit hooks (lint-staged, husky, lefthook)

### Unit / Component
- [ ] Pure logic has unit tests (formatters, reducers, validators)
- [ ] Components use Testing Library + accessible queries
- [ ] Custom hooks tested with `renderHook`
- [ ] Test data via builders / factories, not copy-paste

### Integration
- [ ] MSW (or equivalent) mocks HTTP at the network layer
- [ ] Real router, real providers in test renders
- [ ] Forms tested end-to-end with `userEvent`

### E2E
- [ ] Playwright with cross-browser matrix
- [ ] Critical user journeys only — 5–15 tests
- [ ] Auth via API, not UI
- [ ] Trace + screenshot on failure in CI artifacts
- [ ] Quarantine list for flakes

### A11y
- [ ] axe-core in component tests
- [ ] Storybook a11y addon
- [ ] Manual screen reader testing per release

### Visual
- [ ] Chromatic / Percy on Storybook
- [ ] Dynamic content mocked (dates, animations)

### Performance
- [ ] Lighthouse CI with budgets per route
- [ ] Bundle size budgets in PR
- [ ] RUM regression alerting per release

### Security
- [ ] SAST (Semgrep / CodeQL) on PR
- [ ] Dependency scanning continuous
- [ ] DAST nightly against staging
- [ ] Annual pen test

### Process
- [ ] PR feedback < 10 min
- [ ] Required checks block merges
- [ ] Smoke tests run post-deploy
- [ ] Synthetic monitoring with paging
- [ ] Test results & flake history dashboards

---

## Further Reading

### Foundational
- **Kent C. Dodds — [Testing JavaScript](https://testingjavascript.com/)** / [Epic Web](https://www.epicweb.dev/) — the modern frontend testing curriculum
- **Kent Beck — *Test-Driven Development: By Example*** — the TDD primer
- **Gerard Meszaros — *xUnit Test Patterns*** — the test-double taxonomy and patterns reference
- **Lisa Crispin & Janet Gregory — *Agile Testing*** / ***More Agile Testing***
- **Mark Winteringham & Gwen Diagram — *Software Testing Folklore: 50 Misconceptions Debunked***
- **Yoni Goldberg — [JavaScript Testing Best Practices](https://github.com/goldbergyoni/javascript-testing-best-practices)** — comprehensive checklist

### Specific topics
- **[Testing Library docs](https://testing-library.com/)** — read the "Guiding Principles" and "Common Mistakes" pages
- **[Playwright docs](https://playwright.dev/docs/best-practices)** — official best practices
- **[MSW docs](https://mswjs.io/)**
- **[Stryker mutation testing](https://stryker-mutator.io/)**
- **[fast-check](https://fast-check.dev/)** — property-based testing
- **[Storybook test](https://storybook.js.org/docs/writing-tests)**

### Industry research
- **DORA / Accelerate** — Forsgren, Humble, Kim — empirical research on what makes high-performing software teams; testing is central
- **Google Testing Blog** — historical archives still gold
- **Continuous Delivery** — Humble & Farley — how testing fits into the deploy pipeline

### People to follow
- Kent C. Dodds, Yoni Goldberg, Michael Bolton, James Bach (testing schools), Maaret Pyhäjärvi, Lisa Crispin, Janet Gregory, Dan North (BDD originator)

---

> **Closing thought:** testing is not about catching bugs — it's about **enabling change**. A great test suite makes a codebase malleable; a bad one (or no suite) makes it brittle. The teams I've watched ship the fastest don't optimize for "tests exist" — they optimize for *the right tests at the right level, with realistic boundaries, fast feedback, low flake, and assertions that mean something*. The goal is a suite where a green run means "this works," a red run tells you exactly what broke, and either result arrives within minutes. Build that, and you can ship daily without fear. Skip it, and every change is a gamble. I've watched both outcomes play out across enough projects to be very sure which one I'd rather work in.
