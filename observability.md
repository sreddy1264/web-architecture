# Observability

> A practical guide to making web systems debuggable in production — logs, metrics, traces, real user monitoring, SLOs, alerting, and incident response.

---

## Table of Contents

1. [What is Observability?](#what-is-observability)
2. [Why It Matters](#why-it-matters)
3. [Observability vs. Monitoring](#observability-vs-monitoring)
4. [The Three (Now Four) Pillars](#the-three-now-four-pillars)
5. [Logs](#logs)
6. [Metrics](#metrics)
7. [Traces & Distributed Tracing](#traces--distributed-tracing)
8. [Real User Monitoring (RUM)](#real-user-monitoring-rum)
9. [Frontend Error Tracking](#frontend-error-tracking)
10. [Profiling & Continuous Profiling](#profiling--continuous-profiling)
11. [Synthetic Monitoring](#synthetic-monitoring)
12. [OpenTelemetry — The Modern Standard](#opentelemetry--the-modern-standard)
13. [SLIs, SLOs, SLAs & Error Budgets](#slis-slos-slas--error-budgets)
14. [The Golden Signals & RED & USE](#the-golden-signals--red--use)
15. [Alerting Strategy](#alerting-strategy)
16. [Dashboards That Don't Suck](#dashboards-that-dont-suck)
17. [Incident Response & Postmortems](#incident-response--postmortems)
18. [Cost Management](#cost-management)
19. [Tools Landscape (2026)](#tools-landscape-2026)
20. [Common Anti-Patterns](#common-anti-patterns)
21. [Quick Reference Checklist](#quick-reference-checklist)
22. [Further Reading](#further-reading)

---

## What is Observability?

**Observability** (often abbreviated **o11y**) is the property of a system that lets you **understand its internal state from external outputs** — without having to ship new code to ask new questions.

The term is borrowed from control theory (Rudolf Kálmán, 1960): a system is *observable* if you can determine its state from its outputs alone. Applied to software: can you answer questions about your running production system that you didn't anticipate ahead of time?

> **A useful framing:** the line between monitoring and observability is the *known-unknown* distinction. **Monitoring** answers questions you knew to ask ("CPU > 80%?"). **Observability** lets you ask questions you didn't anticipate ("why did this specific user's checkout take 12 seconds last Tuesday?"). Mature systems need both.

---

## Why It Matters

### 1. Distributed systems are not deterministic
A modern web app is a fleet of services, queues, caches, and dependencies. Bugs emerge from the *interactions*, not from any single component. You can't reproduce them locally — you can only investigate them in production.

### 2. The mean-time-to-resolution (MTTR) game
The DORA research (*Accelerate*) consistently finds that elite teams resolve incidents **>2,000× faster** than low performers. The single biggest differentiator: how quickly engineers can *understand* what's broken. That's observability.

### 3. Detection time = breach severity
The industry-average time to detect a security breach is ~200 days. Mature orgs detect in days or hours. The difference is logging, alerting, and anomaly detection — observability for security.

### 4. User experience is asymmetric
The 50th-percentile user might have a great experience; the 99th-percentile user might be timing out. Aggregate dashboards lie. **Tail latency, error budgets, and per-user tracing are how you find what averages hide.**

### 5. You ship to production blind without it
Code without observability is a black box once deployed. You learn it's broken from customer tickets — the worst possible signal.

---

## Observability vs. Monitoring

| | Monitoring | Observability |
|---|---|---|
| **Question** | Is the system OK? | Why is the system behaving this way? |
| **Style** | Pre-defined dashboards & alerts | Open-ended exploration |
| **Data** | Aggregated metrics | High-cardinality, high-dimensional events |
| **Best for** | Known failure modes | Novel/unknown failure modes |
| **Sufficient when** | Simple, monolithic systems | Distributed, complex systems |

The two are complementary. **Monitoring tells you something is wrong; observability lets you figure out what.**

### Charity Majors' rule
> "If you can't ask a question you didn't ask in advance, that's monitoring, not observability."

Good observability requires:
- **High cardinality** — query by user_id, request_id, build_id, region
- **High dimensionality** — many attributes per event (not just "status: 500")
- **Wide events** — one big structured event per request, with everything attached
- **Indexed for arbitrary queries** — not pre-aggregated

---

## The Three (Now Four) Pillars

```
┌────────────────────────────────────────────────────────────────────┐
│                      THE PILLARS OF O11Y                           │
├────────────────────────────────────────────────────────────────────┤
│                                                                    │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐              │
│   │    LOGS     │   │   METRICS   │   │   TRACES    │              │
│   ├─────────────┤   ├─────────────┤   ├─────────────┤              │
│   │ Discrete    │   │ Aggregated  │   │ Causal      │              │
│   │ events with │   │ time-series │   │ chain across│              │
│   │ context     │   │ numbers     │   │ services    │              │
│   ├─────────────┤   ├─────────────┤   ├─────────────┤              │
│   │ "What       │   │ "How much   │   │ "What       │              │
│   │ happened?"  │   │ / how often?"│  │ caused      │              │
│   │             │   │             │   │ what?"      │              │
│   └─────────────┘   └─────────────┘   └─────────────┘              │
│                                                                    │
│   ┌─────────────┐                                                  │
│   │  PROFILES   │  ← The 4th pillar (modern addition)              │
│   ├─────────────┤                                                  │
│   │ CPU/memory  │                                                  │
│   │ flame graphs│                                                  │
│   │ over time   │                                                  │
│   └─────────────┘                                                  │
│                                                                    │
└────────────────────────────────────────────────────────────────────┘
```

Modern signals also include **events** (RUM, business events) and increasingly **AI-derived signals** (anomaly detection, log clustering).

The three pillars are **not** independent silos. The power comes from **correlation** — clicking from a slow trace to its logs to its metrics to its profile, all linked by `trace_id`. Tools that don't correlate them are stuck in 2015.

---

## Logs

Discrete, time-stamped records of events.

### Structured logging
**Log JSON, not strings.** Structured logs are queryable; unstructured logs are grep targets.

```js
// ❌ Unstructured
console.log(`User ${userId} logged in from ${ip}`);

// ✅ Structured
logger.info({
  event: 'user.login',
  user_id: userId,
  ip,
  user_agent: req.headers['user-agent'],
  trace_id: ctx.traceId,
}, 'user logged in');
```

In your log backend, you can now query:
```
event:"user.login" AND ip:"203.0.113.1" AND _time>1h
```

### Log levels (and when to use each)
| Level | Meaning | Example |
|---|---|---|
| **TRACE** | Extremely detailed; off in prod | Function entry/exit |
| **DEBUG** | Diagnostic; off in prod by default | Variable values during a flow |
| **INFO** | Normal operations | Request received, job completed |
| **WARN** | Unusual but recoverable | Retry succeeded after failure |
| **ERROR** | Operation failed | Exception, 5xx response |
| **FATAL** | Process must exit | Startup config invalid |

> **A heuristic worth applying:** if INFO logs every request, you're paying for noise. Log INFO at *significant business events*, not at every loop iteration. Use sampling for high-volume routes.

### What to log
- Request start/end with duration, status, route, user_id (or hash), trace_id
- Auth events (login, logout, MFA, password reset)
- Authorization decisions (especially denials)
- Sensitive actions (data exports, permission changes, payments)
- Errors with full context (stack trace, inputs, dependent service responses)
- Background jobs (start, completion, failure with retry count)
- Deploys and feature flag changes (as events into the same stream)

### What NOT to log
- **Passwords, tokens, API keys, session IDs**
- Full credit card numbers (PCI-DSS violation)
- Full PII unless legally required and access-controlled
- Request bodies of sensitive endpoints (/login, /signup, /payment)
- Anything that would embarrass you in a screenshot

Implement **automatic redaction** at the logger level — never trust developers to remember.

```js
const logger = pino({
  redact: ['password', 'token', '*.creditCard', 'headers.authorization'],
});
```

### Sampling
At scale, logging every request is impossible. Sample:
- 100% of errors and warnings
- 100% of slow requests (above p99)
- ~1% of normal requests
- 100% of specific user_ids during incidents (dynamic)

This is **head-based sampling** for logs; trace-based sampling is more sophisticated (see Traces).

### Centralization
Logs must be shipped off the host. Common stacks:
- **ELK / OpenSearch** — Elasticsearch + Logstash + Kibana
- **Loki + Grafana** — Prometheus-style log labels; cheap
- **Datadog / Splunk / Sumo Logic / New Relic** — managed, full-featured
- **Cloud-native** — CloudWatch (AWS), Cloud Logging (GCP), Azure Monitor

### Retention
- **Hot** (queryable in seconds): 7–30 days
- **Warm** (slower, cheaper): 30–90 days
- **Cold** (S3 archive): 1+ years for compliance

Compliance regimes (HIPAA, PCI-DSS, SOX) often dictate minimums. Don't assume; check.

---

## Metrics

Aggregated numerical measurements over time.

### Metric types
| Type | Behavior | Example |
|---|---|---|
| **Counter** | Monotonically increasing | `http_requests_total` |
| **Gauge** | Goes up and down | `memory_in_use_bytes`, `active_connections` |
| **Histogram** | Distribution of values in buckets | `http_request_duration_seconds` |
| **Summary** | Pre-computed quantiles | similar to histogram, less flexible |

### Why histograms over averages
**Averages lie.** A service with p50=50ms and p99=10s has the same average as one with p50=200ms and p99=300ms — but very different user experiences. Always track **percentiles (p50, p90, p95, p99, p99.9)**.

```
With averages:                With percentiles:
   "Service is fine,             "p50: 50ms, p99: 8s, p99.9: 30s"
    avg = 200ms"                 → 1% of users having a bad time
```

### Cardinality is everything (and a tax)
**Cardinality** = the number of unique label combinations. Each combination = a separate time series stored in your TSDB.

- `http_requests_total{status="500"}` — low cardinality (5 values for status)
- `http_requests_total{status="500", user_id="u123"}` — high cardinality (millions of users → millions of series)

High cardinality is wonderful for debugging (find that one user's slow request) and **expensive** at scale. Tools like **Prometheus** charge in storage; **Honeycomb / Lightstep / observability-2.0 tools** charge less per dimension. Pick tools that match your cardinality needs.

### What to instrument
The **Four Golden Signals** (Google SRE):
1. **Latency** — duration of requests (p50, p95, p99)
2. **Traffic** — requests per second
3. **Errors** — rate of failed requests
4. **Saturation** — how full the system is (CPU, memory, queue depth)

**RED method** (services):
- **R**ate — requests per second
- **E**rrors — failed requests per second
- **D**uration — distribution of request durations

**USE method** (resources):
- **U**tilization — % time the resource is busy
- **S**aturation — work waiting in queue
- **E**rrors — error count

Apply RED to every service; USE to every resource (CPU, memory, network, disk).

### Business metrics
Tech metrics are necessary but not sufficient. Also track:
- Conversion rate
- Sign-ups per minute
- Revenue per minute
- Cart abandonment
- Search-no-results

A latency regression that doesn't move conversion isn't urgent; one that does, even if metrics look fine, is. **Always tie tech to business signals.**

### Backends
- **Prometheus** — pull-based, time-series, open-source standard
- **VictoriaMetrics** — Prometheus-compatible, more efficient
- **Datadog metrics**, **CloudWatch**, **GCP Monitoring**
- **InfluxDB**, **TimescaleDB** — for non-Prometheus shapes
- **OpenTelemetry Metrics** — vendor-neutral collector

---

## Traces & Distributed Tracing

A **trace** captures the journey of a single request across services. Each unit of work is a **span**.

```
Client request
  │
  ▼
┌─────────────────────────────────────────────────────────────────┐
│   trace_id: abc-123                                             │
│                                                                 │
│   [ web-frontend       350ms ]                                  │
│      [ auth-service    20ms ]                                   │
│      [ api-gateway     320ms ]                                  │
│         [ catalog-svc  50ms ]                                   │
│         [ order-svc    250ms ]                                  │
│            [ db-query  180ms ] ← bottleneck found in 5 seconds  │
│            [ payment   60ms ]                                   │
└─────────────────────────────────────────────────────────────────┘
```

### Why traces are transformative
With logs + metrics, debugging a slow request looks like: see latency spike → grep logs in 12 services → manually correlate timestamps → guess the bottleneck. **With traces, you click and see.**

### Span anatomy
- **`trace_id`** — unique per request; ties spans together
- **`span_id`** — unique per span
- **`parent_span_id`** — builds the tree
- **Operation name** — `GET /api/users/:id`, `db.query`, `redis.get`
- **Duration** — start/end timestamps
- **Attributes** — `http.method`, `db.statement`, `user_id`, custom keys
- **Events** — point-in-time annotations within a span (errors, retries)
- **Status** — OK / ERROR
- **Links** — to other traces (e.g., async work triggered by this request)

### Context propagation
For traces to work across services, **context must propagate** with the request. Standards:
- **W3C Trace Context** — `traceparent` and `tracestate` HTTP headers (the modern default)
- **B3 / Zipkin** — older, still common
- **Datadog / Jaeger / Honeycomb** — vendor-specific propagators (use OpenTelemetry to abstract them)

```
Service A                 Service B
  │                         │
  ├─ HTTP GET /api ─────────►
  │  traceparent: 00-abc... │
  │                         ├─ DB query
  │                         │  (same trace_id)
  │                         ◄─
  │◄────────────────────────┤
```

### Sampling
Tracing every request is expensive. Strategies:
- **Head sampling** — decide at the start (e.g., 1%); cheap but you might miss interesting traces
- **Tail sampling** — buffer all spans, then decide based on outcome (errors, slow requests always kept)
- **Adaptive sampling** — adjust rate based on traffic / SLOs
- **User-controlled** — always trace requests with a debug header

> **In practice:** **always-sample errors and tail latency**. Use head-based sampling for the noise.

### Continuous tracing tooling
- **Jaeger / Zipkin** — open-source UIs
- **Tempo** (Grafana) — high-volume trace storage
- **Honeycomb** — wide-event, high-cardinality observability
- **Lightstep** (now ServiceNow Cloud Observability)
- **Datadog APM**, **New Relic**, **Dynatrace**, **Elastic APM**

---

## Real User Monitoring (RUM)

While server-side observability tells you about your servers, **RUM** tells you about your *users' actual experience* in their browser.

### What RUM captures
- **Core Web Vitals** (LCP, INP, CLS, FCP, TTFB) per session
- **Resource timing** — what loaded, when, how big, from which CDN
- **Navigation timing** — DNS, TCP, TLS, server, render breakdown
- **Errors** — JS exceptions, unhandled rejections, console errors
- **Custom events** — clicks, form submits, business KPIs
- **Session replays** (where consented) — pixel-perfect playback

### Segmentation that matters
Aggregate metrics hide inequality. Always segment:
- **Device class** (low-end Android vs iPhone vs desktop)
- **Network** (4G vs WiFi vs slow-3G fallback)
- **Geography**
- **Browser version**
- **App version / release tag** (so you can detect regressions per deploy)
- **User cohort / role** (new vs returning, free vs paid)

### Implementation
- **`web-vitals` library** — Google's tiny library; collects CWV; ship to your endpoint
- **Vendor RUM** — Datadog RUM, Sentry, New Relic Browser, SpeedCurve, Calibre, DebugBear
- **CrUX** — Chrome User Experience Report; public field data; useful for benchmarks but only Chrome users who opted in

```js
import { onLCP, onINP, onCLS } from 'web-vitals';

const send = (metric) => {
  navigator.sendBeacon('/rum', JSON.stringify({
    name: metric.name,
    value: metric.value,
    id: metric.id,
    rating: metric.rating,
    nav_type: metric.navigationType,
    release: __APP_VERSION__,
    user_id: getUserId(),
    // ...
  }));
};

onLCP(send); onINP(send); onCLS(send);
```

### `sendBeacon` vs `fetch`
Use `navigator.sendBeacon()` for telemetry — fires reliably even on page unload (where `fetch` may be canceled).

### Privacy
RUM ingests user agents, URLs, IP-derived geo. Be transparent in your privacy notice. Anonymize where possible (hash user IDs, drop query strings, redact URLs containing PII like `/users/12345/email/secret`).

---

## Frontend Error Tracking

Server errors get monitored religiously; client errors are often invisible. **Most production bugs your users see, you never know about.**

### Capture
```js
window.addEventListener('error', (e) => {
  reportError({
    message: e.message,
    stack: e.error?.stack,
    file: e.filename,
    line: e.lineno,
    col: e.colno,
    user_agent: navigator.userAgent,
    url: location.href,
    release: __APP_VERSION__,
  });
});

window.addEventListener('unhandledrejection', (e) => {
  reportError({
    message: 'Unhandled promise rejection',
    reason: e.reason?.message ?? String(e.reason),
    stack: e.reason?.stack,
  });
});
```

In React: use **Error Boundaries** to catch render errors per subtree.

### Source maps
Production JS is minified. Without **source maps uploaded to your error tracker**, stacks look like `r.a in chunk-9a2f.js:1:48291` — useless. Upload source maps on every deploy; keep them private (don't ship them publicly).

### Tools
- **Sentry** — the de facto standard
- **Rollbar**, **Bugsnag**, **Honeybadger**
- **Datadog Error Tracking**, **New Relic Errors Inbox**

### Triage strategy
Errors swarm in. To stay sane:
- **Group by signature** (similar errors collapse into one issue)
- **Auto-assign / route** by code owner
- **Set thresholds** for alerting (don't wake someone up for 1 occurrence; 100/min is different)
- **Mark resolved per release** (regressions auto-reopen)
- **Track error budget** (errors per session)

### Filter the noise
Browser extensions, ad blockers, and offline-mode all generate "errors" you can't fix. Filter:
- Errors from extensions (`chrome-extension://`, `safari-extension://`)
- Errors from third-party scripts you don't control
- Network errors (track separately)
- Aborted fetches on navigation

---

## Profiling & Continuous Profiling

The "fourth pillar." A **profile** is a snapshot of where your code is spending time (CPU samples) or memory (heap allocations).

### Continuous profiling
Sample profiles continuously in production at low overhead (~1%). Store, query, diff across deploys.

```
[Pyroscope flame graph]

  main (100%)
    handleRequest (60%)
      fetchUser (45%) ← 45% of CPU here
        jsonParse (30%) ← parsing huge payload
      validateInput (15%)
    renderResponse (40%)
```

### Use cases
- "Why is this service using 80% CPU?"
- "What changed performance after the deploy?"
- Memory leaks (heap profiling over time)

### Tools
- **Pyroscope** (now part of Grafana) — open-source, multi-language
- **Datadog Continuous Profiler**, **New Relic CodeStream**, **Polar Signals**
- **`node --prof`**, Chrome DevTools Performance — for ad-hoc

### When you need it
For services that are CPU-bound, memory-leaky, or have unexplained slowdowns. Skip for low-throughput services where it's overkill.

---

## Synthetic Monitoring

Scripts that simulate user behavior from external locations, on a schedule, regardless of real traffic. Fills the gap when there's no real traffic to measure.

### Use cases
- **Uptime / availability** — "Did the homepage load every minute?"
- **Critical user journeys** — "Did checkout work from London at 03:00?"
- **API contract** — "Does GET /api/health return 200 with the right shape?"
- **Pre-deploy smoke tests** — "Does staging work end-to-end?"
- **Performance baselines** — controlled-network latency over time (independent of real traffic)

### Tools
- **Datadog Synthetics**, **Pingdom**, **Checkly**, **UptimeRobot**, **Better Stack**
- **Playwright** scripts run on a cron in your CI

### What synthetic doesn't replace
RUM. Synthetic monitors don't reflect real user devices, networks, or behavior. **Use both** — synthetic for "is it up?", RUM for "how is it for users?"

---

## OpenTelemetry — The Modern Standard

**OpenTelemetry (OTel)** is the CNCF-graduated, vendor-neutral standard for instrumenting code with logs, metrics, and traces. It's the Linux of observability — you should be on it.

### What it gives you
- **Vendor-neutral instrumentation** — instrument once; switch backends without rewriting
- **Automatic instrumentation** for popular frameworks (Express, Next.js, Fastify, Django, Rails, Spring, .NET)
- **Manual span creation** for business logic
- **Context propagation** standardized (W3C Trace Context)
- **Collector** — a sidecar that batches, samples, transforms, and exports telemetry to one or many backends

### Architecture
```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│   Your services (instrumented with OTel SDK)                 │
│           │                                                  │
│           ▼ OTLP (gRPC or HTTP)                              │
│                                                              │
│   ┌──────────────────────────────────────────┐               │
│   │       OpenTelemetry Collector            │               │
│   │  (receivers → processors → exporters)    │               │
│   │                                          │               │
│   │  • Batch / retry                         │               │
│   │  • Sampling                              │               │
│   │  • Attribute filtering / redaction       │               │
│   │  • Tail sampling                         │               │
│   │  • Multi-backend fanout                  │               │
│   └─────────────┬────────────────────────────┘               │
│                 │                                            │
│      ┌──────────┼──────────┬──────────┐                      │
│      ▼          ▼          ▼          ▼                      │
│   Datadog   Jaeger     Honeycomb   Prometheus                │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Why mature teams default to OTel
- No vendor lock-in
- One agent, one instrumentation, many backends
- Backed by the entire industry (Datadog, AWS, Microsoft, Google all contribute)
- Future-proof — replaces older standards (OpenCensus, OpenTracing)

### Caveat
OTel is a moving target. Early adopters dealt with API churn; in 2026 it's stable for most languages. JavaScript, Go, Java, .NET, Python, Ruby are first-class.

---

## SLIs, SLOs, SLAs & Error Budgets

The Google SRE framework for thinking about reliability *as a budget you spend*.

| Term | Definition |
|---|---|
| **SLI** (Service Level Indicator) | A metric that measures user-perceived service quality (e.g., success rate, latency p99) |
| **SLO** (Service Level Objective) | A target for the SLI (e.g., 99.9% of requests succeed) |
| **SLA** (Service Level Agreement) | A contractual SLO with consequences (refunds, etc.) |
| **Error Budget** | `1 - SLO` = how much "bad" you can afford |

### Picking SLIs
The best SLIs measure what users actually care about:
- **Availability** — % of requests that succeed (definition matters: 4xx are usually user errors, 5xx are yours)
- **Latency** — % of requests under a threshold
- **Throughput** — requests/sec sustained
- **Correctness** — % of business outcomes correct (orders placed without duplicates)
- **Freshness** — data is no more than X seconds stale

### Choosing SLO targets
- **99%** = 7.2 hours/month of downtime — generous
- **99.9%** = 43 minutes/month — most consumer products
- **99.99%** = 4.3 minutes/month — payments, infra
- **99.999%** = 26 seconds/month — telecom-grade; rare

> **Worth knowing:** higher reliability is exponentially more expensive. **Don't promise more than you actually need.** Most consumer products are fine at 99.9%; chasing 99.99% costs 10× more for marginal user value.

### Error budgets in practice
If your SLO is 99.9% and your budget is 0.1%:
- **Budget remaining**: ship features, take risks
- **Budget exhausted**: freeze risky changes, focus on reliability

This **gives engineering teams a quantitative tradeoff** between feature velocity and reliability. No more "we should be more careful" — there's a budget.

### Multi-window, multi-burn-rate alerts
Don't alert on "SLO violated this hour" — alert on **burn rate**:
- A 14× burn rate over 1 hour → page (will exhaust the monthly budget in <2 days)
- A 6× burn rate over 6 hours → ticket (slower drain)
- A 3× burn rate over 3 days → review

Reference: Google SRE Workbook, Chapter 5.

---

## The Golden Signals & RED & USE

Three overlapping mnemonics for "what to instrument." Each is right in its context.

### Four Golden Signals (Google SRE)
For user-facing services:
1. **Latency** — distribution of request durations (success and error separately)
2. **Traffic** — demand (requests/sec, sessions, MB/s)
3. **Errors** — rate of failures
4. **Saturation** — how "full" your service is (CPU, memory, queue, connection pool)

### RED (services)
- **R**ate — requests per second
- **E**rrors — errors per second
- **D**uration — request duration distribution

### USE (resources)
- **U**tilization — % time busy
- **S**aturation — queue / wait
- **E**rrors — error count

> **Pragmatic combo:** RED on every service endpoint; USE on every machine/container; Golden Signals on every user-facing surface.

---

## Alerting Strategy

Alerting that wakes engineers up should be **rare, actionable, and worth the wake-up.** Most teams alert too much and detect too little.

### The three-tier model
1. **Page (P0)** — wake someone up. Reserved for: user impact, SLO burn, hard outages. **Every page should be actionable** ("here's the runbook"); if there's nothing to do, don't page.
2. **Ticket (P1)** — review during business hours. For: warnings, slow trends, capacity ahead-of-time.
3. **Dashboard / log** — informational; not an alert at all.

### Alert on symptoms, not causes
- **Cause-based**: "DB CPU > 80%". Fires often; sometimes the user is fine; sometimes the user is not.
- **Symptom-based**: "Checkout latency p99 > 5s." Fires when *the user* is in pain.

Symptom-based alerts are far more actionable. You don't care that DB CPU is high — you care that users are slow.

### Avoid these alert pitfalls
- **Threshold alerts on noisy metrics** — they flap, get muted, then miss real issues
- **No silence period during deploys** — re-firing per deploy is noise
- **No ownership** — every alert needs a clearly-assigned team or rotation
- **No runbook link** — what does the on-call do? Without a runbook, the page is a quiz
- **"Just FYI" pages** — anything that doesn't require action shouldn't page

### Alert fatigue
The single biggest threat to on-call health. The cure:
- **Audit alerts quarterly** — for each: did it fire? did it page? was it actionable? Was it false?
- **Aggressively delete or downgrade** unactionable alerts
- **Track on-call quality metrics** — pages per shift, time-to-acknowledge, false-positive rate

> **Lesson learned:** if your on-call gets paged more than ~2× per shift on average, your alerting is broken. The job becomes triage-noise instead of incident-response. Every team I've helped through alert-fatigue burnout had this exact pattern. Fix alerts, not headcount.

### Modern alert routing
- **PagerDuty**, **Opsgenie**, **Splunk On-Call**, **incident.io**, **Better Stack**
- Slack/Teams for ticket-tier
- Auto-create a Slack channel + Zoom on P0 (incident.io, FireHydrant pioneer this)

---

## Dashboards That Don't Suck

Most teams have dozens of dashboards nobody reads.

### Tiered dashboards
- **Status / overview** — one screen; team-level health; for execs and stand-ups
- **Service dashboard** — one per service; RED + saturation + dependencies
- **Investigation dashboards** — built for specific incident types; runbook-linked
- **Business dashboards** — conversion, revenue, sign-ups; not just tech metrics

### Principles
- **One dashboard, one purpose.** A dashboard with 40 graphs answers no questions.
- **Above the fold = the key signals.** SLO health, error rate, latency p99.
- **Group by user journey**, not by tech component. Users don't care about your auth service; they care about login.
- **Annotate deploys** — vertical lines marking releases turn unexplained graphs into causal narratives.
- **Time controls and zoom-in are sacred** — broken time pickers ruin investigations.
- **Maintain or delete** — dashboards rot. Annual cleanup; delete what's unread.

### Tools
- **Grafana** (open-source, multi-source) — the standard for self-hosted
- **Datadog**, **New Relic**, **Dynatrace** — managed
- **Honeycomb BubbleUp** — query-driven, not graph-built; different paradigm

---

## Incident Response & Postmortems

Observability without process is wasted. The data needs a workflow.

### Incident lifecycle
```
DETECT  ──►  TRIAGE  ──►  MITIGATE  ──►  RESOLVE  ──►  LEARN
   │           │            │             │            │
  Alert     Severity     Stop the      Root-fix    Postmortem
  fires     declared     bleeding                  Action items
```

### The on-call's first questions (the playbook)
1. **What's the impact?** — users affected, revenue, dollars
2. **When did it start?** — correlate with deploys, traffic, external events
3. **What changed?** — deploys, feature flags, infra changes, vendor incidents
4. **Can I mitigate without understanding?** — rollback, disable a feature flag, scale up, drain a host. **Mitigation > investigation during fire.**
5. **Who else needs to know?** — comms, customer-facing teams, legal (for breaches)

### Severity levels
A common scheme:
- **SEV-1**: full outage / major data loss / security breach. All-hands.
- **SEV-2**: partial outage / important feature broken. Page + war room.
- **SEV-3**: degraded but workable. Ticket; fix in business hours.
- **SEV-4**: minor; backlog.

Different orgs differ; the point is **shared vocabulary**.

### Communication during an incident
- **Status page** — Statuspage, Better Uptime, Instatus. Update every 30 minutes minimum during an incident.
- **Internal channel** — dedicated Slack/Teams channel per incident (auto-created by tooling)
- **Roles** — Incident Commander (decisions), Communications Lead (updates), Subject Matter Experts. Don't let the IC do everything.

### Blameless postmortems
**The single most important practice for high-reliability orgs.** From the Google SRE book:

- **Focus on systems, not individuals.** "What conditions allowed this?", not "who screwed up?"
- **Assume good intent.** Engineers acted reasonably given what they knew.
- **Document timeline, impact, root causes (plural — usually multiple), what went well, what didn't, and action items**
- **Action items have owners and deadlines** — otherwise they don't happen
- **Public-when-possible** — share with the whole company; transparency builds trust
- **Quarterly review** — pick patterns across postmortems; invest in cross-cutting fixes

### Game days / chaos engineering
At maturity, simulate failure deliberately:
- **Game days** — practice incidents (kill a region, disable a service)
- **Chaos engineering** — automated failure injection (Chaos Monkey, Gremlin, Litmus)
- **Tabletop exercises** — paper-walk-through of scenarios

You don't want your team's first major incident to be... their first major incident.

---

## Cost Management

Observability spend can rival cloud bills. Datadog horror stories ("our APM bill was $400K/year") are common. Manage it deliberately.

### Major cost levers
| Lever | Effect |
|---|---|
| **Log volume** | The biggest cost. Reduce via sampling, removing INFO noise, structured logs (compresses better) |
| **Cardinality** | Each unique label combination is a series. High cardinality = high cost. |
| **Retention** | Hot tier costs 10–100× cold storage. Tier aggressively. |
| **Trace sampling rate** | Drop the rate; tail-sample interesting traces. |
| **RUM session sampling** | Don't capture every session; sample at 10% with 100% of errors. |
| **Custom metric pricing** | Some vendors (Datadog) charge per custom metric — audit. |

### Self-hosted vs managed
- **Self-hosted (Prometheus + Grafana + Loki + Tempo)** — cheap on cloud bill, expensive in engineering time
- **Managed (Datadog, New Relic, Honeycomb)** — fast to start, expensive at scale; hidden costs as cardinality grows

> **In practice:** the right answer evolves. Most companies start managed (right tradeoff for small teams), then either negotiate hard with their vendor or move to self-hosted (Grafana stack) at scale. Plan the transition before the bill becomes a board-level conversation.

### Cost observability is observability too
Track o11y spend per service, per team. Make engineers aware of what their logs cost. Surface the largest contributors (top 10 highest-cardinality metrics, log spammers, etc.).

---

## Tools Landscape (2026)

### Full observability platforms (logs + metrics + traces + RUM)
- **Datadog** — most complete; expensive at scale
- **New Relic** — similar; usage-based pricing
- **Dynatrace** — strong APM, AI-driven anomaly detection
- **Elastic Observability** — open-core; flexible
- **Splunk Observability Cloud** (formerly SignalFx)
- **Grafana Cloud** — Loki + Tempo + Mimir + Pyroscope; LGTM stack

### Open-source LGTM stack
- **L**oki — logs
- **G**rafana — visualization
- **T**empo — traces
- **M**imir — metrics (Prometheus-compatible at scale)
- **Pyroscope** — profiles

### Specialized tools
- **Prometheus** — metrics; the de facto standard for Kubernetes
- **Honeycomb** — wide-event observability; high-cardinality SQL-style queries; BubbleUp
- **Lightstep / ServiceNow Cloud Observability** — distributed tracing
- **Jaeger / Zipkin** — open-source tracing UIs
- **Sentry** — error tracking + tracing + RUM (now competing with full APM)
- **Bugsnag**, **Rollbar** — error tracking
- **OpenTelemetry Collector** — vendor-neutral pipeline

### Synthetic
- **Datadog Synthetics**, **Checkly**, **Better Stack**, **Pingdom**, **UptimeRobot**

### Incident management
- **PagerDuty**, **Opsgenie**, **incident.io**, **Splunk On-Call**, **Better Stack**, **FireHydrant**, **Rootly**

### Status pages
- **Statuspage** (Atlassian), **Better Uptime**, **Instatus**, **Cachet** (open-source)

### AI-driven o11y (rising fast)
- **Honeycomb Query Assistant** — LLM-driven query writing
- **Datadog Watchdog** / **Bits AI** — anomaly detection, incident summaries
- **Grafana ML** — forecasting, outliers
- LLMs are increasingly used for incident triage, log summarization, and anomaly explanation. Expect this to be standard within a few years.

---

## Common Anti-Patterns

### 1. Logging instead of tracing
Sprinkling `logger.info` in 12 services and grepping for `request_id` is a primitive form of tracing. Use real distributed tracing.

### 2. Aggregating away the tail
Reporting only averages or median latency. The 99th percentile is where the user pain is.

### 3. Cardinality explosion → bill shock
A label like `user_id` on a Prometheus metric — millions of series, untenable cost. Use traces (high-cardinality friendly) for per-user data.

### 4. Alerting on causes, not symptoms
"DB CPU > 80%" pages while no user is affected. Use SLO-based alerting on user-facing symptoms.

### 5. Logging sensitive data
Tokens, PII, full request bodies in logs. Compliance violation; security incident waiting. Redact at logger level.

### 6. Alerting fatigue
On-call gets 30 pages per shift. Engineers stop reading. Real incidents missed. Audit and prune ruthlessly.

### 7. No source maps in production
Errors are minified gibberish; debugging takes 10× longer. Upload source maps on deploy; keep them private.

### 8. Per-service dashboards, no user-journey view
You can see every service is "green," but checkout is broken because of an unmonitored interaction. Build user-journey dashboards.

### 9. Postmortems that blame individuals
Engineers stop reporting near-misses. Culture decays. Insist on **blameless** postmortems; fix systems, not people.

### 10. Treating o11y as "ops only"
Engineers throw code over the wall; ops figures out logs/metrics/dashboards. Modern practice: **the engineer who writes the code instruments it.** "You build it, you run it."

### 11. No correlation between signals
Logs in one tool, metrics in another, traces in a third — no shared `trace_id`. Investigation is a manual time-correlation exercise. Use a unified backend or rigorously share trace IDs.

### 12. Synthetic monitoring as the only signal
Your synthetic monitor passes; users complain on Twitter. Synthetic doesn't reflect real users. Pair with RUM.

### 13. RUM without segmentation
"Our LCP is 2.8s on average!" That hides the 30% of users above 5s. Always segment by device, network, geo, release.

### 14. Adding o11y after the incident
"We need better dashboards." A week of work, then nothing happens until the next incident. Bake instrumentation into the definition of done. **Untraced services don't go to production.**

### 15. Vendor lock-in instrumentation
Hardcoded calls to `dd-trace.span()` everywhere. Switching backends means rewriting hundreds of files. Use OpenTelemetry; instrument once, export anywhere.

---

## Quick Reference Checklist

### Instrumentation
- [ ] Every service emits structured logs with trace_id, request_id, user_id (hashed if PII)
- [ ] Every service exports metrics (RED) and resource metrics (USE)
- [ ] Every service emits traces with W3C Trace Context propagation
- [ ] OpenTelemetry SDK in use across services
- [ ] Source maps uploaded to error tracker on every deploy
- [ ] RUM captures CWV + custom events; segmented by release/device/network

### SLOs
- [ ] Each service has SLIs (latency, error rate, success rate)
- [ ] SLOs documented and tracked
- [ ] Error budget burn rate alerts (multi-window, multi-rate)
- [ ] SLO dashboard visible to leadership

### Alerting
- [ ] Pages are symptom-based, actionable, runbook-linked
- [ ] On-call rotation with reasonable size; metrics tracked
- [ ] Quarterly alert audit; drift detected
- [ ] No silent dropping of alerts; routing always lands somewhere

### Logs
- [ ] Structured JSON; centralized
- [ ] PII / secrets redacted at logger
- [ ] Sampling for high-volume routes
- [ ] Tiered retention (hot/warm/cold)
- [ ] No log statements at TRACE/DEBUG in production by default

### Errors
- [ ] window error + unhandledrejection captured
- [ ] React Error Boundaries in place
- [ ] Errors tagged with release, user, route
- [ ] Auto-routing/triaging by code owner
- [ ] Source maps uploaded; stack traces readable

### Tracing
- [ ] All services participate in distributed tracing
- [ ] Sampling: 100% errors, 100% slow tail, head-sample 1–10% normal
- [ ] DB queries, cache calls, queue ops as spans

### Dashboards
- [ ] Service dashboard per service
- [ ] User-journey / business KPI dashboard
- [ ] SLO dashboard
- [ ] Deploys annotated on time-series graphs
- [ ] Quarterly cleanup; dead dashboards deleted

### Synthetic + RUM
- [ ] Synthetic monitors for top user journeys (every 1–5 min)
- [ ] Status page connected to monitors
- [ ] RUM with consent; segmented; alerted on regressions per release

### Process
- [ ] Severity levels documented
- [ ] Incident response runbook + roles (IC, Comms, SME)
- [ ] Status page updated on user-facing incidents
- [ ] Blameless postmortems for SEV-1/2; published broadly
- [ ] Action items tracked to completion
- [ ] Game days / chaos exercises quarterly (at maturity)
- [ ] Observability cost monitored & owned

---

## Further Reading

### Foundational
- **Charity Majors, Liz Fong-Jones, George Miranda — *Observability Engineering*** (O'Reilly, 2022) — the modern bible
- **Cindy Sridharan — *Distributed Systems Observability*** — short, brilliant, free PDF on O'Reilly
- **Google — *Site Reliability Engineering*** (free online) — the foundational SRE book
- **Google — *The Site Reliability Workbook*** (free online) — practical companion; SLO chapters are essential
- **Forsgren, Humble, Kim — *Accelerate*** — DORA research; what makes high-performing teams (observability is central)

### SLO-specific
- **Alex Hidalgo — *Implementing Service Level Objectives***
- **The Art of SLOs** — workshop materials by Google SRE; free online

### Specific topics
- **OpenTelemetry docs** — `opentelemetry.io`
- **[Honeycomb O11y Resources](https://www.honeycomb.io/resources)** — Charity Majors' team; deep posts on high-cardinality o11y
- **[Brendan Gregg's blog](https://www.brendangregg.com/)** — performance & profiling legend; flame graphs originator
- **[Increment magazine — On-Call issue](https://increment.com/on-call/)** — incident response philosophy
- **[How Complex Systems Fail (Richard Cook)](https://how.complexsystems.fail/)** — 18-point essay; required reading

### People to follow
- Charity Majors, Liz Fong-Jones, Cindy Sridharan, Brendan Gregg, John Allspaw, Lorin Hochstein (resilience engineering), Niall Murphy (SRE), Ben Sigelman (Lightstep)

### Standards & specs
- **OpenTelemetry** — `opentelemetry.io`
- **W3C Trace Context** — propagation standard
- **Prometheus exposition format** — metrics standard

---

> **Closing thought:** observability is not a tool you buy — it's a *cultural commitment* to treating production as a first-class environment, instrumented from day one, owned by the engineers who build the code. The teams I've watched ship reliably aren't the ones that write the most tests or have the best architectures; they're the ones who, when something breaks, can ask any question of their running system and get an answer in minutes. The teams that can't are always rebuilding the same dashboards in the middle of the next outage. **Build for the next incident — because the next incident is coming, and the only thing you control is how prepared you are when it arrives.**
