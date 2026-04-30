# CI/CD — Continuous Integration & Continuous Delivery

> A practical guide to designing pipelines that ship code safely, frequently, and quickly — from PR to production.

---

## Table of Contents

1. [What is CI/CD?](#what-is-cicd)
2. [Why It Matters](#why-it-matters)
3. [CI vs. CD vs. CD — The Three Terms](#ci-vs-cd-vs-cd--the-three-terms)
4. [The CI/CD Mindset](#the-cicd-mindset)
5. [Pipeline Anatomy](#pipeline-anatomy)
6. [Branching Strategies](#branching-strategies)
7. [Continuous Integration in Detail](#continuous-integration-in-detail)
8. [Quality Gates](#quality-gates)
9. [Artifacts & Registries](#artifacts--registries)
10. [Environments](#environments)
11. [Deployment Strategies](#deployment-strategies)
12. [Progressive Delivery & Feature Flags](#progressive-delivery--feature-flags)
13. [Database Migrations in CI/CD](#database-migrations-in-cicd)
14. [Infrastructure as Code (IaC)](#infrastructure-as-code-iac)
15. [GitOps](#gitops)
16. [Secrets in CI/CD](#secrets-in-cicd)
17. [Supply Chain Security](#supply-chain-security)
18. [Rollback Strategies](#rollback-strategies)
19. [Pipeline Performance](#pipeline-performance)
20. [Monorepo CI/CD](#monorepo-cicd)
21. [DORA Metrics & Measuring Pipeline Health](#dora-metrics--measuring-pipeline-health)
22. [Compliance, Audit & Change Management](#compliance-audit--change-management)
23. [Tools Landscape (2026)](#tools-landscape-2026)
24. [Common Anti-Patterns](#common-anti-patterns)
25. [Quick Reference Checklist](#quick-reference-checklist)
26. [Further Reading](#further-reading)

---

## What is CI/CD?

**CI/CD** is the practice (and the tooling) that automates the path from "I committed code" to "users are running it in production." It combines:

- **Continuous Integration (CI)** — engineers integrate their work into a shared mainline frequently (multiple times per day), with each integration verified by automated tests.
- **Continuous Delivery / Deployment (CD)** — every successful change can be (Delivery) or is automatically (Deployment) released to production.

Done well, CI/CD turns "shipping software" from a quarterly event into a quiet hourly activity. Done badly, it's a flaky pipeline that nobody trusts and everyone bypasses.

> **A reframe worth keeping:** the value of CI/CD is **not automation for its own sake** — it's **shrinking the gap between writing code and learning whether it works.** Every minute you save in feedback is a minute of compounded learning across the team. Pipelines optimize for *learning velocity*.

---

## Why It Matters

### 1. The DORA findings are unambiguous
The State of DevOps Research (now part of the *Accelerate* book by Forsgren, Humble, and Kim) measured tens of thousands of teams across many years. **Elite performers** vs **low performers**:

| Metric | Elite | Low |
|---|---|---|
| **Deployment frequency** | On-demand (multiple/day) | Once per 1–6 months |
| **Lead time for changes** | < 1 hour | 1–6 months |
| **Mean time to restore (MTTR)** | < 1 hour | 1 week – 6 months |
| **Change failure rate** | 0–15% | 46–60% |

The gap is often **>1,000× across all four metrics** simultaneously. Strong CI/CD is the technical backbone of that performance.

### 2. Small batches lower risk
Counterintuitive but well-supported: **deploying smaller changes more often is safer than deploying larger changes less often.** Smaller batches mean smaller blast radius, easier rollback, faster identification of regressions. Big-bang quarterly releases are the highest-risk way to ship.

### 3. CI/CD enables every other practice
Continuous testing, continuous security scanning, progressive delivery, feature flags, observability-driven development — all assume an automated pipeline you trust. Without it, you're stuck in batch-mode.

### 4. Manual deploys are expensive
A 30-minute manual deploy ritual times 50 engineers times 100 deploys/year is many person-years lost. Automated deploys reclaim that time.

### 5. Developer experience compounds
Fast pipelines mean engineers don't context-switch while tests run. Slow ones fragment focus and erode flow. Pipeline performance is engineering productivity in disguise.

---

## CI vs. CD vs. CD — The Three Terms

These get conflated; precision matters in design discussions.

```
┌───────────────────────────────────────────────────────────┐
│                                                           │
│   CONTINUOUS INTEGRATION (CI)                             │
│   ──────────────────────────                              │
│   Engineers merge to main daily.                          │
│   Every merge runs automated tests + checks.              │
│   Goal: catch integration bugs in minutes, not weeks.     │
│                                                           │
│   ──────────────────────────────────────────              │
│                                                           │
│   CONTINUOUS DELIVERY (CD)                                │
│   ─────────────────────                                   │
│   Every change that passes CI is automatically            │
│   built into a deployable artifact and is one             │
│   button-press away from production.                      │
│   Production deploys are still manual approvals.          │
│                                                           │
│   ──────────────────────────────────────────              │
│                                                           │
│   CONTINUOUS DEPLOYMENT (CD)                              │
│   ──────────────────────                                  │
│   Every change that passes CI deploys                     │
│   automatically to production with no human in            │
│   the loop. Possible only with strong testing,            │
│   monitoring, and rollback automation.                    │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

Most production-facing companies practice **continuous delivery** — automated to staging, gated to production. **Continuous deployment** (no human approval) is achievable but requires deep maturity in testing, feature flags, observability, and rollback. Etsy, GitHub, Netflix, and Stripe deploy to production hundreds of times per day on continuous deployment.

---

## The CI/CD Mindset

### 1. Trunk-based development is the foundation
Long-lived feature branches kill CI's value. The longer a branch lives away from main, the more it diverges, the more painful the merge. **Merge to main daily, behind feature flags.**

### 2. Every commit is a release candidate
The artifact built on `main` is what could go to production. No "release branches that get extra polish." If main isn't deployable, fix main *first*.

### 3. Pipelines are code
CI/CD configuration belongs in version control next to the source. Click-and-configure CI tools that store config in their own DBs (legacy Jenkins) are a graveyard of unreproducible setups.

### 4. Speed is a feature
A 45-minute pipeline doesn't get run. A 5-minute pipeline gets run constantly. **Aggressively cache, parallelize, and shard.** Fast CI is engineering culture.

### 5. Tests must be reliable, not just present
A flaky test suite is **worse than no suite** — it teaches engineers to ignore failures. Quarantine flakes; never "rerun until it passes."

### 6. Mean Time to Repair > Mean Time Between Failures
Optimizing only for "no failures" is brittle. Optimizing for "fast detection + fast rollback" is anti-fragile. **Build for the inevitable failure; recover quickly when it comes.**

### 7. Production is the final test environment
No matter how much staging you build, production is unique — real traffic, real data, real users. Practices like canary deploys, feature flags, and observability turn production into a safe place to learn.

### 8. Trust the pipeline, but verify
Pipelines fail silently. Mark green builds that didn't actually run all tests. **Audit your gates** — what runs, what doesn't, what's been disabled "temporarily."

---

## Pipeline Anatomy

A modern pipeline is a directed graph of jobs, gated by conditions.

```
┌─────────────────────────────────────────────────────────────────────┐
│                       END-TO-END PIPELINE                           │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  PR opened / pushed                                                 │
│        │                                                            │
│        ▼                                                            │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ FAST FEEDBACK     (target: < 5 min)                      │       │
│  │   ► Lint  ► Format  ► Type check                         │       │
│  │   ► Unit + component tests (sharded)                     │       │
│  │   ► Bundle size budget                                   │       │
│  │   ► Secret scan  ► Dep vulnerabilities                   │       │
│  └────────────────────────┬─────────────────────────────────┘       │
│                           │                                         │
│                           ▼                                         │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ DEEP CHECKS       (target: < 15 min)                     │       │
│  │   ► Integration tests                                    │       │
│  │   ► Visual regression (Storybook)                        │       │
│  │   ► A11y axe scans                                       │       │
│  │   ► SAST (Semgrep / CodeQL)                              │       │
│  │   ► Container build + scan (Trivy)                       │       │
│  │   ► Preview env deployed (per-PR URL)                    │       │
│  └────────────────────────┬─────────────────────────────────┘       │
│                           │                                         │
│                Code review approval ──────┐                         │
│                           │               │                         │
│                           ▼               │                         │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ MERGE TO MAIN                                            │       │
│  │   Squash merge, signed commit                            │       │
│  └────────────────────────┬─────────────────────────────────┘       │
│                           │                                         │
│                           ▼                                         │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ POST-MERGE      (target: < 20 min total)                 │       │
│  │   ► Build production artifact (with provenance)          │       │
│  │   ► Sign artifact (Sigstore / cosign)                    │       │
│  │   ► Push to registry                                     │       │
│  │   ► Deploy to staging                                    │       │
│  │   ► E2E tests on staging (Playwright)                    │       │
│  │   ► Lighthouse CI                                        │       │
│  │   ► DAST scan                                            │       │
│  └────────────────────────┬─────────────────────────────────┘       │
│                           │                                         │
│                           ▼                                         │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ DEPLOY TO PROD     (manual or auto)                      │       │
│  │   ► Canary 1% → 5% → 25% → 100%                          │       │
│  │   ► Monitor SLOs at each step                            │       │
│  │   ► Auto-rollback on regression                          │       │
│  │   ► Smoke tests in production                            │       │
│  │   ► Synthetic monitor catches regressions                │       │
│  └────────────────────────┬─────────────────────────────────┘       │
│                           │                                         │
│                           ▼                                         │
│  ┌──────────────────────────────────────────────────────────┐       │
│  │ POST-DEPLOY                                              │       │
│  │   ► RUM regression watch (per release tag)               │       │
│  │   ► Error tracker scoped to release                      │       │
│  │   ► Auto-create incident if SLO burns                    │       │
│  └──────────────────────────────────────────────────────────┘       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### Key principles
- **Build once; deploy many.** The artifact built post-merge is what runs in staging *and* production. **Never rebuild for prod.** Different binaries = different bugs.
- **Fast lane → slow lane.** Short feedback first; expensive checks later. Engineers don't wait 20 minutes for a typo.
- **Required vs. advisory checks.** Some checks block merges; some report and inform.
- **Idempotent jobs.** Re-running a job should produce the same result. No "well it works the second time."
- **Artifact promotion.** Move the same binary through stages; don't rebuild.

---

## Branching Strategies

The way you branch determines how CI/CD operates.

### Trunk-Based Development (TBD)
**The default that scales.** All work merged to `main` daily, gated by quality checks and feature flags.

```
   main:  ──●──●──●──●──●──●──●──→
            ↑   ↑   ↑   ↑
           short branches (hours, not days)
```

**Benefits:**
- Real CI — everyone integrates with everyone constantly
- No merge hell
- Releases on demand (every commit potentially shippable)
- Strongly correlated with elite DORA performance

**Required:**
- Strong test coverage at all levels
- Feature flags for in-progress work
- Discipline (no "works on my branch")

### GitHub Flow
A lightweight variant: short-lived branches → PR → main → deploy. Effectively trunk-based for small teams.

### GitFlow (legacy)
Long-lived `develop`, `feature/*`, `release/*`, `hotfix/*` branches. Heavyweight; designed for shrink-wrapped software with quarterly releases. **Most modern web teams should not use this.**

### Release Branches (when needed)
For products that ship versions to customers (mobile apps, on-prem software, OS distros), maintain `release/v*` branches alongside `main`. Cherry-pick fixes from main; tag releases.

> **A challenge to consider:** if your team uses GitFlow, ask why. The most common reason is "we always have." Migration to trunk-based + feature flags pays back almost immediately in lead time and merge pain.

---

## Continuous Integration in Detail

### What CI actually means
"We use Jenkins" ≠ CI. CI requires:
1. **Single shared mainline** that everyone integrates into
2. **Frequent integration** — at minimum daily, ideally hourly
3. **Automated build** triggered on every commit
4. **Automated tests** that gate the build
5. **Fast feedback** to the engineer who broke it
6. **Broken builds are stopped first** — fixing main is everyone's priority

If any of those are missing, you have automated tests, not CI.

### "Stop the line"
Toyota's manufacturing principle, applied to code: when the build is broken, don't merge new work that compounds the problem. Fix the build first. Some teams enforce this via branch protection (no merges while main is red).

### PR-based workflow
The dominant pattern:
```
1. Engineer creates branch from main
2. Pushes commits; PR is opened (or draft)
3. CI runs on every push
4. Code review
5. All required checks pass
6. Merge to main (squash usually preferred)
7. Branch deleted
```

### Pre-commit hooks
Catch issues before push. Tools: **husky**, **lefthook**, **pre-commit** (Python), **lint-staged** for staged-only files.

```yaml
# lefthook.yml
pre-commit:
  parallel: true
  commands:
    lint:
      glob: "*.{ts,tsx}"
      run: pnpm eslint --fix {staged_files}
    format:
      glob: "*.{ts,tsx,md,json}"
      run: pnpm prettier --write {staged_files}
```

Don't go too heavy — slow pre-commit hooks are skipped (`git commit --no-verify`).

### Pre-push hooks
Run quick tests, type checks. Tradeoff: catches issues earlier; slows down `git push`.

### CI per platform — the modern shape
- **GitHub Actions** — most popular for OSS and small-medium teams
- **GitLab CI** — if you're on GitLab; tightly integrated
- **Buildkite** — strong for self-hosted runners and custom workflows
- **CircleCI**, **Jenkins**, **Azure Pipelines**, **Drone**, **TeamCity**
- **Argo Workflows** — Kubernetes-native, for complex DAG pipelines

---

## Quality Gates

What blocks a merge to main? Document and enforce.

### Required (block merge)
- ✅ Lint (ESLint, etc.) — zero errors
- ✅ Format check (Prettier / Biome)
- ✅ Type check (`tsc --noEmit`)
- ✅ Unit + component tests pass
- ✅ Bundle size within budget
- ✅ No new secrets detected (gitleaks, TruffleHog, GitHub secret scanning)
- ✅ No new vulnerabilities above severity threshold (Dependabot, Snyk)
- ✅ No new SAST findings (Semgrep, CodeQL)
- ✅ At least 1 (often 2) reviewer approvals
- ✅ Branch up-to-date with main

### Advisory (informational, don't block)
- 🟡 Visual regression diffs (review, may be intentional)
- 🟡 Coverage delta (warn on drops, don't block)
- 🟡 Performance budgets (warn first, enforce later)

### Quality gate maturity
Start lenient; tighten over time. **A strict policy that gets bypassed daily is worse than a lenient one that's followed.** Earn enforcement.

---

## Artifacts & Registries

### What's an artifact?
The thing that runs in production: a Docker image, a static site bundle, a JAR, a tarball.

### Build-once principle
Pipelines that rebuild for each environment introduce drift. **Build once on merge to main; tag with commit SHA; promote the same artifact through environments.**

```
main commit abc123
   │
   ▼
[ build artifact ] ─→ tag: abc123
   │
   ▼
[ push to registry ]
   │
   ▼
[ deploy to dev ]    ← same artifact
[ deploy to staging] ← same artifact
[ deploy to prod ]   ← same artifact
```

### Artifact registries
- **Docker images** — Docker Hub, GitHub Container Registry (GHCR), AWS ECR, GCP Artifact Registry, Azure ACR, Harbor (self-hosted)
- **JS packages** — npm, GitHub Packages, JFrog
- **JVM** — Maven Central, JFrog Artifactory, GitHub Packages
- **Generic** — JFrog Artifactory, AWS S3 with versioning

### Tagging strategy
- **Commit SHA** (`abc123def`) — immutable, traceable; the **default for production**
- **Semver** (`v1.4.2`) — for libraries, public artifacts
- **`latest`** — convenient but dangerous; never deploy `latest` to production
- **Date-based** (`2026.04.30.1430`) — useful for daily snapshots

Tag every artifact with **multiple tags** so you can find it by SHA, version, and date.

### Provenance
Modern supply-chain security demands you can prove an artifact came from a specific commit + build. See [Supply Chain Security](#supply-chain-security).

---

## Environments

A typical environment progression:

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│  LOCAL  →  PR PREVIEW  →  STAGING  →  CANARY  →  PRODUCTION    │
│                                                                │
│  Each environment:                                             │
│  - more production-like (data, scale, traffic)                 │
│  - fewer engineers can deploy to it                            │
│  - more automation between it and the next                     │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Local
Developer machine. Should run a representative slice of the system (Docker Compose, devcontainers, mise/asdf for tool versions).

### PR preview / ephemeral environments
A dedicated environment per pull request, spun up on each push. Reviewers click a URL; the change is live, isolated, with seeded data. Tear down on merge or stale.

**Tools:** Vercel previews, Netlify deploy previews, Render preview environments, Heroku Review Apps, Coherence, Architect, Bunnyshell, ephemeral K8s namespaces.

**Why mature teams love them:** product reviewers, designers, PMs, and stakeholders see real changes in real environments before merge. Closes the feedback loop dramatically.

### Staging / pre-production
Production-shaped (similar infra, anonymized prod-like data, similar traffic synthetic). E2E tests run here; smoke checks before promoting.

**Common pitfall:** staging that diverges from prod (different data, different secrets, different scale) — bugs ship to prod that staging would have caught. Keep them as similar as possible.

### Production
The real thing. Deploys here are gated, monitored, and reversible.

### Disaster recovery / failover
Often a separate region; tested via game days.

### A challenge to your ladder
The classic "dev → test → staging → UAT → preprod → prod" ladder is mostly cargo-culted. Each environment costs money and engineer time. **Justify every environment** with a question: *what does this environment let us learn that the next one can't?*

---

## Deployment Strategies

How you put a new version in front of users. Each has different risk/cost tradeoffs.

### 1. Recreate / Big Bang
Stop old; start new. Downtime; high risk; only for cases where downtime is acceptable.

### 2. Rolling
Replace instances one (or a few) at a time. Eventually all are updated.
- ✅ Zero downtime
- ✅ Built into Kubernetes
- ❌ Slow rollback (must roll back instance by instance)
- ❌ Both versions live simultaneously (need backward compatibility)

### 3. Blue/Green
Deploy new version (green) alongside old (blue). Switch traffic. Keep blue as instant rollback.
```
[Load Balancer]
       │
       ├─→ Blue (current)  ← all traffic
       └─→ Green (new)     ← idle, warming
                                ↓ (switch)
[Load Balancer]
       │
       ├─→ Blue (current)  ← idle, ready to roll back
       └─→ Green (new)     ← all traffic
```
- ✅ Instant rollback
- ✅ Easy to test green before switch
- ❌ Doubles infrastructure cost briefly
- ❌ Database migrations are still hard (shared DB)

### 4. Canary
Send a small % of traffic (1%, 5%, 25%) to the new version; watch metrics; promote or roll back.
```
                       100%
                        │
                ┌───────┴───────┐
              99% (current)   1% (canary)
                                │
              SLOs OK? ─────────┴────► Promote
              SLOs bad? ──────────► Rollback
```
- ✅ Real-traffic validation with limited blast radius
- ✅ Gradual confidence
- ❌ Requires routing infra (service mesh, load balancer with weighted routing)
- ❌ Requires *good* observability to detect regressions per percentile

**Modern tools:** Argo Rollouts, Flagger, Spinnaker, Linkerd / Istio for traffic splitting.

### 5. Shadow / Mirror
Real production traffic is duplicated to the new version, but its responses are discarded. Lets you test against real load without risk.
- ✅ Real-traffic load test before users see anything
- ✅ Find performance / DB / cache issues
- ❌ Complex routing
- ❌ Side effects (writes) need to be neutered

### 6. A/B Testing / Feature Flags
A subset of users see the new behavior, controlled by flag rules (random %, user attributes, geo, etc.). See [Progressive Delivery](#progressive-delivery--feature-flags).

### Choosing
| Need | Pick |
|---|---|
| Zero-downtime, simple | Rolling |
| Instant rollback | Blue/Green or Canary |
| De-risk a major change | Canary + feature flags |
| Test against real load | Shadow |
| User-targeted release | Feature flags |

---

## Progressive Delivery & Feature Flags

The idea: **decouple deployment from release.** Code goes to production all the time; *features* go to users when ready, at the speed and audience you choose.

### Feature flag basics
```ts
if (flags.isEnabled('new-checkout', user)) {
  return <NewCheckout />;
}
return <OldCheckout />;
```

The flag system decides per-request whether the flag is on, based on:
- Boolean (everyone on/off)
- % rollout (10% of users)
- User attributes (paid plan only)
- Cohort / segment
- Geographic region
- Device type

### Why mature teams ship behind flags
1. **Decouple deploy from release** — ship code mid-feature; flip it on later
2. **Roll out gradually** — 1% → 5% → 25% → 100% with monitoring at each step
3. **Targeted releases** — beta program, internal-only, geographic rollouts
4. **Kill switches** — instant disable on incident
5. **A/B testing** — measure impact, not vibes
6. **Trunk-based development** — half-finished features can ship safely behind off flags

### Types of flags
- **Release flags** — temporary, controlling rollout. Should be deleted after rollout.
- **Ops flags** — kill switches for ops to disable misbehaving features
- **Permission flags** — entitlement-based (paid features)
- **Experiment flags** — A/B test variants

### Flag debt is real
Flags accumulate. A 2-year-old flag still in code is a maintenance burden, often dead weight, sometimes a security hole. **Set TTLs and clean them up.** Tools (LaunchDarkly, Unleash) surface stale flags.

### Tools
- **LaunchDarkly** — market leader, comprehensive
- **Statsig** — strong experiment + flag combo
- **GrowthBook** — open-source experimentation + flags
- **Unleash** — open-source, self-hostable
- **Flagsmith** — open-source, multi-tenant
- **Cloudflare Workers / Vercel Edge Config** — edge-evaluated flags
- **OpenFeature** — vendor-neutral spec; instrument once, swap providers

### The kill-switch principle
Anything that could go wrong should have an off-switch. New endpoints, new dependencies, new third-party integrations — all behind flags so ops can disable them in seconds, not deploys.

---

## Database Migrations in CI/CD

Schema changes are the single hardest part of CI/CD. Get them wrong, and downtime, data loss, or rollback impossibility ensues.

### The expand-and-contract pattern (a.k.a. parallel change)
Never make a breaking schema change in one step. Decompose:

1. **Expand** — add the new column (nullable, defaulted) without breaking the old code
2. **Backfill** — populate the new column from the old one, asynchronously
3. **Migrate** — application reads/writes both old and new
4. **Switchover** — application reads/writes only the new
5. **Contract** — drop the old column

Each step is independently shippable and reversible. **Multi-week migration; zero downtime.**

### Examples
**Renaming a column:**
1. Add `new_email`; copy from `email` on writes
2. Backfill `new_email` for existing rows
3. Application reads `new_email`, falling back to `email`
4. Application reads/writes only `new_email`
5. Drop `email`

**Splitting a table:** similar dance — write to both, backfill, switch reads, drop old.

### Tools
- **Prisma Migrate**, **Drizzle Migrate**, **Knex migrations** (JS/TS)
- **Flyway**, **Liquibase** (Java; widely used in polyglot)
- **Alembic** (Python / SQLAlchemy)
- **ActiveRecord migrations** (Rails)
- **Atlas** (Ariga) — declarative, multi-DB

### Rules
- **Migrations are forward-only in production.** Down migrations are theoretical; you'll fix forward.
- **Migrations run before app deploy** — new code shouldn't run against old schema.
- **Long-running migrations need care.** Adding a column with a default on a 50M-row table can lock it for hours; use `ALTER TABLE ... ADD COLUMN ... DEFAULT NULL` (instant in modern Postgres) and backfill separately.
- **Index creation should be `CONCURRENTLY`** in Postgres to avoid table locks.
- **Test migrations against production-shaped data.** A migration that runs in 1s on dev may run in 1 hour on prod.

### Schema review
Reviewing migrations is its own skill. Mature teams require:
- DBA / SRE review on schema PRs
- Tooling that flags risky migrations (table locks, full-table rewrites)
- **Squawk** (`squawk lint`) for Postgres — catches dangerous migrations

---

## Infrastructure as Code (IaC)

Infrastructure defined declaratively in version-controlled code. **Click-ops doesn't scale.**

### Why
- **Reproducibility** — disaster? Re-create the environment from code
- **Review** — infrastructure changes go through PR review like code
- **History** — what changed, when, by whom
- **Multi-environment parity** — staging and prod from the same templates
- **Disaster recovery** — bootstrap a new region in hours, not weeks

### Tools

| Tool | Style | Best for |
|---|---|---|
| **Terraform / OpenTofu** | Declarative HCL; provider-rich | Multi-cloud, the de facto standard |
| **Pulumi** | Real programming language (TS, Python, Go) | Teams that want loops/conditionals/types |
| **AWS CDK** | TS/Python on top of CloudFormation | AWS-only; ergonomic |
| **CloudFormation** | YAML/JSON; AWS-only | Pure AWS; mature |
| **Bicep** | Azure-native DSL | Azure |
| **Ansible** | Procedural; configuration management | Server config (less for infra provisioning) |
| **Crossplane** | K8s-native IaC | Cloud resources via K8s manifests |
| **Helm** | K8s package manager | Kubernetes app deployment |

### State management
Terraform/Pulumi maintain *state* — what's currently deployed. Lose state and you've lost infrastructure metadata. Store remotely (S3 + DynamoDB lock, Terraform Cloud, Pulumi Cloud); never commit local state.

### Drift detection
Reality drifts from code (someone clicked something in the console). Run regular drift detection (`terraform plan`); alert; resolve.

### IaC scanning
- **Checkov**, **tfsec**, **Snyk IaC**, **Trivy** — find misconfigurations (open S3 buckets, missing encryption)
- **OPA / Conftest** — policy-as-code over IaC
- **Terraform Cloud / Spacelift / Atlantis** — managed IaC pipelines

### A sociotechnical observation
IaC is sociotechnical: it forces explicit decisions and review. Teams that adopt it discover all the ways their click-ops process was breaking down silently.

---

## GitOps

A specific deployment philosophy: **the desired state of the system lives in Git; operators reconcile reality to match.**

```
┌────────────────────────────────────────────────────────────────┐
│                                                                │
│   git push  ─►  Git repo (desired state)                       │
│                       │                                        │
│                       ▼                                        │
│             [ GitOps controller ]                              │
│             (ArgoCD, Flux)                                     │
│                       │                                        │
│                       ▼                                        │
│            Compare desired vs actual                           │
│                       │                                        │
│            Reconcile (apply)                                   │
│                       │                                        │
│                       ▼                                        │
│              Kubernetes / cloud                                │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Properties
- **Pull-based** — controllers run inside the cluster; pull config from Git. No CI server with cluster credentials.
- **Declarative** — desired state in YAML/HCL, not imperative scripts
- **Auditable** — every change is a commit with author + reason
- **Self-healing** — drift is automatically reconciled
- **Disaster recovery** — rebuild from Git

### Tools
- **ArgoCD** — Kubernetes GitOps; rich UI
- **Flux** — Kubernetes GitOps; CNCF
- **Atlantis** — Terraform GitOps via PR comments
- **Fleet** (Rancher) — multi-cluster

### When GitOps wins
- Kubernetes-heavy organizations
- Multiple clusters / regions
- Strict audit / compliance requirements
- Strong separation of CI (build) from CD (deploy)

### When it doesn't fit
- Small teams without K8s
- Serverless-heavy stacks (Vercel, Netlify, AWS Lambda) — their own deploy model is simpler
- Apps that need imperative deploy logic GitOps doesn't model well

---

## Secrets in CI/CD

Pipelines need secrets (DB passwords, API keys, signing keys). Handling them poorly is a leading breach cause.

### Rules
- **Never** commit secrets to Git. Use pre-commit secret scanning (gitleaks, TruffleHog).
- **Never** print secrets in logs. Most CI tools mask known secrets, but custom output can leak.
- **Never** pass secrets in command-line args (visible in `ps`).
- **Never** hardcode secrets in Dockerfiles or CI YAML.

### Where secrets should live
- **Cloud secrets manager** — AWS Secrets Manager, GCP Secret Manager, Azure Key Vault, HashiCorp Vault
- **CI provider secret store** — GitHub Actions secrets, GitLab CI variables (encrypted, scoped)
- **Doppler, 1Password Connect, Infisical** — managed secret platforms

### Short-lived credentials > long-lived
**OIDC federation** is the modern way. CI assumes a cloud role via OIDC for the duration of the job; no long-lived keys stored anywhere.

```yaml
# GitHub Actions example
- uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: arn:aws:iam::123:role/deploy
    aws-region: us-east-1
# No AWS_ACCESS_KEY_ID stored anywhere — token minted just-in-time.
```

Supported by GitHub Actions, GitLab CI, CircleCI → AWS, GCP, Azure, HashiCorp Vault. **Adopt this pattern.**

### Rotation
Even best-managed secrets get exposed. Rotate:
- On a schedule (every 30/60/90 days for high-value)
- On staff offboarding
- After any suspected compromise

Automated rotation is mature in cloud secrets managers — wire it up.

### Per-environment scoping
Production secrets only available to the production deploy job. Don't let a typo in a dev script reveal prod credentials.

---

## Supply Chain Security

You inherit the security of every dependency. The 2020s have been a decade of supply-chain attacks: SolarWinds, Codecov, Log4Shell, ua-parser-js, xz utils.

### SLSA — Supply-chain Levels for Software Artifacts
A graduated framework for build integrity:
- **Level 1** — automated build, provenance generated
- **Level 2** — version-controlled, hosted CI, authenticated provenance
- **Level 3** — hardened build platform, non-falsifiable provenance
- **Level 4** — two-party review, hermetic, reproducible builds

Most companies should target **SLSA 2** or **SLSA 3**.

### Sigstore / cosign
Sign artifacts cryptographically; verify on deploy. Free, open-source.

```bash
cosign sign ghcr.io/example/app:abc123
cosign verify ghcr.io/example/app:abc123 --certificate-identity ...
```

### SBOM (Software Bill of Materials)
Inventory of every dependency in the artifact. **CycloneDX** or **SPDX** format. Required by US Executive Order 14028 for federal software.

Generate at build time; store with artifact:
```bash
syft packages dir:. -o cyclonedx-json > sbom.json
```

### Vulnerability scanning
- **Dependencies** — Dependabot, Renovate, Snyk, OSV-scanner
- **Container images** — Trivy, Grype, Snyk
- **Source code** — Semgrep, CodeQL, Snyk Code

Block builds on critical CVEs; auto-PR for fixes.

### Lockfile discipline
- **Always commit lockfiles** (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`)
- **Use `--frozen-lockfile`** in CI to fail if it's out of date
- **Auditable upgrades** — Renovate / Dependabot grouped PRs

### Reproducible builds
The same source should produce byte-identical artifacts on different machines. Hard to achieve in JS land (timestamps in tarballs); achievable in Go, Rust, Java with discipline. Ideal but rarely strict requirement outside high-security domains.

### Pinned base images
```dockerfile
# ❌
FROM node:20

# ✅
FROM node:20.11.1-alpine3.19@sha256:abc123def...
```
The digest pin prevents supply-chain swaps.

---

## Rollback Strategies

Every deployment is a hypothesis. Be ready to undo.

### Forward vs. backward
- **Backward rollback** — redeploy the previous artifact. Fast for stateless services.
- **Forward rollback** — ship a fix forward. Often required when DB migrations have run.

### When you can roll back cleanly
- No schema changes
- No data shape changes
- New version is a strict superset of behavior of the old

### When you can't (and what to do)
- After a forward-only DB migration
- When a feature flag needs to be off (instead of redeploy, **flip the flag** — instant rollback without deploy)
- When external integrations are now using new APIs

**The flag-driven rollback is faster than any deploy.** Design features with "off" as a real option from day one.

### Automatic rollback triggers
- SLO error budget burn rate spikes
- Error rate above threshold
- Latency p99 above threshold
- Specific high-priority alerts firing

Tools: Argo Rollouts, Flagger, Spinnaker have automated rollback on metric regressions.

### Rollback-by-default
Some teams (e.g., Google) have an opinion: **assume rollback unless proven safe**. Promotion gates require explicit pass; failure rolls back automatically.

### Test the rollback
Untested rollback procedures fail at the worst time. **Game-day rollbacks** quarterly: deploy to prod, intentionally roll back, time how long it takes, fix the issues you find.

---

## Pipeline Performance

A 30-minute pipeline trains engineers to context-switch. Optimize.

### Targets
- **PR feedback** — < 10 minutes for required checks
- **Post-merge to staging** — < 20 minutes
- **Staging to prod** — < 30 minutes
- **Total: PR open → prod** — < 1 hour for elite teams

### Tactics

#### Caching
- **Dependencies** — npm cache, pnpm store, Docker layer cache
- **Build cache** — Vite/Webpack/esbuild caches; **Turborepo / Nx / Bazel remote cache** for monorepos
- **Test runner cache** — `vitest --cache`
- **CI image cache** — pre-built containers with dependencies installed

#### Parallelism
- Run independent jobs in parallel (lint, type-check, test all at once)
- Shard test suites across workers (`--shard=1/4`, `--shard=2/4`, ...)
- Matrix builds for multiple OS/runtime targets

#### Test impact analysis
Run only tests affected by the changed files:
- **Nx affected**, **Turborepo filter**
- **GitHub Actions paths-filter**, **Bazel test**
- **Wallaby**, **Jest's `--changedSince`**

#### Self-hosted runners (sometimes)
Public CI runners are shared and slow. Self-hosted on beefier hardware can be 2-5× faster, but you trade ops burden. Worth it for very large monorepos and Docker-heavy builds.

#### Skip what you can
- Documentation-only changes — skip tests
- README changes — skip everything except link checks
- Smart enough to detect by file paths

#### Don't run E2E on every PR
Slow, flaky, expensive. Run a subset on PRs (smoke); full suite on main; full + extended on schedule.

### Flaky tests destroy pipeline performance
Re-running flaky tests doubles or triples pipeline time. **Quarantine flakes**; fix or delete; never tolerate.

---

## Monorepo CI/CD

Monorepos (Nx, Turborepo, Bazel, Pants, Lerna, pnpm workspaces) bring CI/CD challenges:

### The naive trap
Running every test on every PR is slow when the monorepo has 50 packages. **Affected-only** is the answer.

### Affected detection
Tools detect which packages changed, then run tests/builds only for those plus their dependents.

```bash
# Nx
nx affected:test --base=main

# Turborepo
turbo run test --filter='[main...HEAD]'

# Bazel
bazel test //... # but with sophisticated dependency caching
```

### Remote caching
The most leveraged CI optimization in monorepos: **share build/test cache across machines**. A test that passes for one engineer is cached; everyone else benefits.

- **Turborepo Remote Cache** (Vercel)
- **Nx Cloud** — managed remote cache
- **Bazel remote build execution**

### Parallel builds
With proper dependency graph awareness, monorepo tools can build packages in parallel, only after their deps complete.

### Pipeline shape
```
Detect affected packages (Nx affected / Turborepo filter)
        │
        ▼
For each affected package, in parallel where possible:
  ┌─────────────────────┐
  │  Lint + type + test │
  └─────────────────────┘
        │
        ▼
Build only affected publishable packages
        │
        ▼
Deploy only affected services
```

### A warning from experience
A monorepo without affected-only CI is a productivity disaster waiting to scale. **Set this up before you need it.**

---

## DORA Metrics & Measuring Pipeline Health

The four DORA metrics are the north stars for CI/CD effectiveness.

### The Four Keys
1. **Deployment Frequency** — how often you deploy to production
2. **Lead Time for Changes** — commit → production
3. **Mean Time to Restore (MTTR)** — incident detected → resolved
4. **Change Failure Rate** — % of deploys causing incidents

### Performance bands (DORA 2023)
| Metric | Elite | High | Medium | Low |
|---|---|---|---|---|
| **Deploy frequency** | On demand | Daily–weekly | Weekly–monthly | < monthly |
| **Lead time** | < 1 hour | 1 day – 1 week | 1 week – 1 month | 1–6 months |
| **MTTR** | < 1 hour | < 1 day | 1 day – 1 week | 1 week – 6 months |
| **Change failure rate** | 0–15% | 16–30% | 16–30% | 46–60% |

### Beyond DORA — adding reliability
DORA's 5th metric (added 2021) is **reliability** — meeting SLOs. (See `observability.md` for SLOs.)

### Measuring DORA
- **GitHub data** for deploy frequency and lead time
- **Incident tracker** (PagerDuty, incident.io) for MTTR and CFR
- **Tools**: Sleuth, LinearB, Faros AI, Jellyfish, DX, Allstacks
- **DIY** with the right Prometheus / data warehouse setup

### Other useful pipeline metrics
- **Pipeline duration** (p50, p95, p99) — fast feedback
- **Pipeline success rate** — flake indicator
- **Build queue time** — capacity indicator
- **Deploy success rate** — pipeline + production health
- **Time to merge** — code review and pipeline efficiency

### A caution about metrics
Metrics drive behavior. **Optimize for "what we want" not "what we measure."** A team gaming "deployment frequency" by deploying micro-changes 100× a day looks elite but is shipping nothing of value. Pair DORA with business outcomes.

---

## Compliance, Audit & Change Management

Regulated environments (finance, healthcare, government) layer compliance on top of CI/CD.

### Common requirements
- **Separation of duties** — author can't approve their own change
- **Documented approvals** — who approved, when, why
- **Change management records** — every prod deploy has a ticket
- **Audit trail** — what was deployed, when, by whom; immutable
- **Production access logs** — every command, recorded
- **Build integrity** — provable artifact provenance (SLSA)

### How modern CI/CD meets these
- **Required reviewers** in branch protection — separation of duties
- **Signed commits** — author authentication
- **Merge events as audit log** — Git itself is the record
- **GitOps** — Git history is the change-management ledger
- **Provenance + SBOM** — SLSA-compliant
- **CI logs** — retained per audit policy
- **Production deploys via tooling** — no SSH; commands logged

### "DevOps in regulated industries"
Misconception: regulation forbids fast deploys. Reality: regulation requires *traceability*, not slowness. **You can deploy 100×/day in a SOX-compliant environment** if every deploy is traced, approved, tested, and reversible.

### CAB (Change Advisory Board) reviews
Heavyweight; legacy ITIL. Modern equivalent: **automated change records** generated from PR metadata. Reviewers approve in PR; the system creates the change record automatically.

---

## Tools Landscape (2026)

### CI platforms
- **GitHub Actions** — most popular for OSS and small-medium teams; native ecosystem
- **GitLab CI/CD** — tightly integrated; strong for self-hosted GitLab
- **CircleCI** — fast, mature; good caching
- **Buildkite** — strong for self-hosted runners and custom workflows
- **Jenkins** — legacy but powerful; enterprise still
- **Drone**, **Woodpecker** — lightweight, container-native
- **TeamCity** — JetBrains; enterprise
- **Azure Pipelines** — strong on Azure; multi-platform
- **AWS CodePipeline / CodeBuild** — AWS-native (somewhat clunky)
- **Argo Workflows** — Kubernetes-native, complex DAGs

### CD / progressive delivery
- **Argo CD** + **Argo Rollouts** — K8s-native GitOps + canary
- **Flux** — alternative GitOps for K8s
- **Spinnaker** — Netflix's CD platform; sophisticated
- **Flagger** — progressive delivery on K8s service meshes
- **Octopus Deploy** — Windows / .NET strong

### Feature flags
- **LaunchDarkly** — market leader
- **Statsig**, **Unleash**, **GrowthBook**, **Flagsmith**
- **OpenFeature** — vendor-neutral spec

### IaC
- **Terraform / OpenTofu**, **Pulumi**, **AWS CDK**, **Bicep**, **Crossplane**

### Container & registries
- **Docker / OCI**, **Buildah**, **Kaniko** (rootless build)
- **GHCR**, **AWS ECR**, **GCP Artifact Registry**, **Harbor**, **JFrog Artifactory**

### Supply chain
- **Sigstore / cosign**, **SLSA tools**, **Syft / Grype** (SBOM, scan), **Trivy**

### Secrets
- **HashiCorp Vault**, **AWS Secrets Manager**, **GCP Secret Manager**, **Doppler**, **Infisical**, **1Password Connect**

### Pipeline analytics / DORA
- **Sleuth**, **LinearB**, **Jellyfish**, **DX**, **Faros AI**

### PR previews / ephemeral envs
- **Vercel**, **Netlify**, **Render**, **Heroku Review Apps**, **Coherence**, **Bunnyshell**

### Compliance
- **Vanta**, **Drata**, **Secureframe** — automate SOC 2 / ISO / HIPAA evidence collection from pipelines

---

## Common Anti-Patterns

### 1. Long-lived feature branches
Branches that live for weeks → merge hell, divergent CI signal, big-bang releases. Migrate to trunk + flags.

### 2. Manual deploy steps
"Just SSH in and `git pull`" → unrepeatable, unaudited, untested. Automate.

### 3. Different artifact per environment
Building separately for staging vs prod = different bugs. Build once; promote.

### 4. Slow pipelines tolerated
A 60-minute pipeline silently destroys productivity. Treat pipeline speed as a P1 concern.

### 5. Tests skipped because they're flaky
Skipping creates blind spots. Quarantine, fix, or delete — never `.skip` indefinitely.

### 6. Secrets in env files committed to Git
The single most common breach vector. Pre-commit scanning + secrets manager + OIDC.

### 7. Click-ops infrastructure
Console clicks → drift, no review, no recovery. IaC everything.

### 8. CI/CD as the security gate
Squashing all security into CI = false sense of security. Defense in depth: static + dynamic + runtime + monitoring.

### 9. Production deploys without monitoring
Deploy and pray. Always pair deploys with release-tagged metrics, errors, RUM.

### 10. No rollback rehearsal
First time you roll back is during an incident. Practice quarterly.

### 11. "We'll add tests later"
Code without tests in trunk-based CI is unsafe. Tests are the contract that makes "merge to main" safe.

### 12. Branch protection bypassed by admins
Every "admin can bypass" rule gets used "this once"; soon every PR is bypassed. Make protection apply to admins too (or restrict bypass to break-glass).

### 13. Build artifacts not signed
Cosign / sigstore are free; not using them is a 2026-grade neglect.

### 14. CI-only runs the tests engineers wrote
Tests must run locally too. If `pnpm test` doesn't reproduce CI, your CI is fragile.

### 15. Database migrations done manually
Schema PRs → automated migrations → automated CI verification. Manual migration in production is a disaster waiting to happen.

### 16. Treating CD as an afterthought to CI
"We have CI; we'll figure out deploys later." Deploys are the harder half. Design them together.

### 17. No cleanup of feature flags
Flags pile up; code branches multiply; nobody dares delete one. Set TTLs.

### 18. Production access via SSH for "emergencies"
The "emergency" becomes routine. Move all prod actions through tooling with audit logs.

### 19. Compliance theater
A 200-page change-management process that gets bypassed in real incidents. Real compliance is automated and painless.

### 20. Pipeline as an engineering org no-man's-land
"Platform team owns CI; nobody owns the slow tests." Pipeline performance and reliability need clear ownership.

---

## Quick Reference Checklist

### Source control
- [ ] Trunk-based development
- [ ] Branch protection: required reviews, required checks, no admin bypass for prod branches
- [ ] Signed commits required (or at least encouraged)
- [ ] Squash merges as default

### CI fast lane
- [ ] PR feedback < 10 minutes
- [ ] Lint + format + type check
- [ ] Unit + component tests, sharded
- [ ] Bundle size budget enforced
- [ ] Secret scanning
- [ ] Dep vulnerability scanning
- [ ] Pre-commit hooks for trivial issues

### CI deep lane
- [ ] Integration tests
- [ ] Visual regression (Storybook + Chromatic)
- [ ] A11y axe scans
- [ ] SAST (Semgrep / CodeQL)
- [ ] Container scan (Trivy)
- [ ] PR preview environment URL

### Build & artifacts
- [ ] Build once on merge to main
- [ ] Tag with commit SHA
- [ ] Push to registry
- [ ] Signed (cosign / Sigstore)
- [ ] SBOM generated and stored
- [ ] Provenance attested (SLSA)

### Deploy
- [ ] Automated to staging
- [ ] E2E tests on staging (Playwright)
- [ ] Lighthouse CI on staging
- [ ] DAST scan
- [ ] Deploy to prod via canary or blue/green
- [ ] Monitor SLOs at each step
- [ ] Auto-rollback on regression
- [ ] Smoke tests in production
- [ ] Deploy event tagged in observability tools

### Feature flags
- [ ] In-progress features behind flags
- [ ] Kill switches for new features
- [ ] Flag TTLs and stale-flag cleanup
- [ ] Flag changes audited

### Database
- [ ] Migrations versioned in repo
- [ ] Forward-only in prod
- [ ] Expand-and-contract for breaking changes
- [ ] Migration linter (Squawk for Postgres)

### Security & supply chain
- [ ] Secrets in cloud secret manager / vault
- [ ] OIDC federation, no long-lived keys in CI
- [ ] Pinned base images (digest)
- [ ] Lockfiles committed; `--frozen-lockfile` in CI
- [ ] Dependency auto-update (Dependabot / Renovate)

### Infra
- [ ] All infra in IaC (Terraform / Pulumi)
- [ ] Remote state with locking
- [ ] Drift detection on schedule
- [ ] IaC scanning (Checkov / tfsec)
- [ ] PR previews / ephemeral envs

### Observability
- [ ] Deploy events sent to APM / logs
- [ ] Release tags propagate through error tracker
- [ ] RUM metrics segmented by release
- [ ] SLO burn-rate alerts
- [ ] Synthetic monitors after deploy

### Process
- [ ] DORA metrics tracked
- [ ] Pipeline duration p95 monitored
- [ ] Flaky test dashboard; quarantine policy
- [ ] Rollback rehearsals quarterly
- [ ] Post-incident reviews feed back into pipeline improvements

---

## Further Reading

### Foundational
- **Jez Humble & David Farley — *Continuous Delivery* (2010)** — the book that defined the field
- **Jez Humble, Joanne Molesky, Barry O'Reilly — *Lean Enterprise***
- **Forsgren, Humble, Kim — *Accelerate* (2018)** — DORA research; required reading
- **Gene Kim et al. — *The Phoenix Project*, *The Unicorn Project*** — narrative DevOps
- **Charity Majors et al. — *Database Reliability Engineering***

### Practical
- **Mike Long — *DevOps for the Modern Enterprise***
- **Jennifer Davis & Ryn Daniels — *Effective DevOps***
- **Brendan Burns — *Designing Distributed Systems***
- **Site Reliability Engineering (Google, free online)** — chapters on release engineering, canary, postmortems

### Specs / standards
- **SLSA** — `slsa.dev`
- **Sigstore** — `sigstore.dev`
- **CycloneDX**, **SPDX** — SBOM formats
- **OpenFeature** — `openfeature.dev`
- **OpenTelemetry** — for pipeline-emitted telemetry

### Blogs & resources
- **DORA / DevOps Research** — `dora.dev`
- **CD Foundation** — `cd.foundation`
- **Spotify Engineering**, **Stripe Engineering**, **Etsy Engineering**, **Netflix Tech Blog**, **Shopify Engineering** — model CI/CD writeups
- **Octopus Deploy blog** — practical CD content
- **Charity Majors' blog (`charity.wtf`)** — observability + deploy philosophy
- **Continuous Delivery YouTube channel** — Dave Farley

### People to follow
- Dave Farley, Jez Humble, Charity Majors, Liz Fong-Jones, Adrian Cockcroft, Kelsey Hightower, Bridget Kromhout, Patrick Debois (DevOps coined), Nicole Forsgren

---

> **Closing thought:** CI/CD is not "the pipeline." It's the engineering culture of **shipping in small, safe, frequent, reversible steps** — built on automation that engineers trust enough to use as a default. The technical substrate (runners, registries, GitOps, canaries) is the easy part; **the cultural shift to trust automation, embrace small batches, and treat production as a first-class environment** is the hard part. The work I'm proudest of is rarely the YAML — it's shaping the conditions where small, safe, fast deploys are *easier than the alternative*, so engineers default to them without thinking. When you build that, you ship daily without fear, recover from incidents in minutes instead of hours, and turn "shipping software" from an event into a habit. That's where elite teams live — and the on-ramp is the pipeline.
