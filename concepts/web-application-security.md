# Web Application Security

> A practical guide to defending web applications — threat models, common attacks, defenses, and the security mindset.

---

## Table of Contents

1. [What is Web Application Security?](#what-is-web-application-security)
2. [Why It Matters](#why-it-matters)
3. [The Security Mindset](#the-security-mindset)
4. [The CIA Triad and Beyond](#the-cia-triad-and-beyond)
5. [Defense in Depth: Layers of Security](#defense-in-depth-layers-of-security)
6. [Threat Modeling (STRIDE)](#threat-modeling-stride)
7. [Authentication](#authentication)
8. [Authorization](#authorization)
9. [Sessions & Tokens](#sessions--tokens)
10. [The OWASP Top 10 (2021/2025) — Common Attacks](#the-owasp-top-10-20212025--common-attacks)
11. [Cross-Site Scripting (XSS)](#cross-site-scripting-xss)
12. [Cross-Site Request Forgery (CSRF)](#cross-site-request-forgery-csrf)
13. [SQL Injection & Other Injections](#sql-injection--other-injections)
14. [SSRF (Server-Side Request Forgery)](#ssrf-server-side-request-forgery)
15. [Insecure Deserialization & Prototype Pollution](#insecure-deserialization--prototype-pollution)
16. [Clickjacking](#clickjacking)
17. [Open Redirects](#open-redirects)
18. [Security Headers](#security-headers)
19. [CORS — Cross-Origin Resource Sharing](#cors--cross-origin-resource-sharing)
20. [Content Security Policy (CSP)](#content-security-policy-csp)
21. [Transport Security (TLS, HSTS)](#transport-security-tls-hsts)
22. [Cryptography Done Right](#cryptography-done-right)
23. [Secrets Management](#secrets-management)
24. [Supply Chain Security](#supply-chain-security)
25. [Logging, Monitoring, Incident Response](#logging-monitoring-incident-response)
26. [Privacy, GDPR, and Data Protection](#privacy-gdpr-and-data-protection)
27. [Best Practices Checklist](#best-practices-checklist)
28. [Common Pitfalls](#common-pitfalls)
29. [Further Reading](#further-reading)

---

## What is Web Application Security?

**Web application security** is the practice of protecting web applications from threats that exploit weaknesses in code, configuration, infrastructure, dependencies, or human behavior — preserving **confidentiality, integrity, and availability** of data and services.

It spans the full stack:
- **Client** — browser, JS, storage, runtime
- **Network** — TLS, DNS, CDN, WAF
- **Server** — auth, input validation, business logic, secrets
- **Data** — encryption at rest, access control, retention
- **Supply chain** — dependencies, build tools, CI/CD
- **People** — phishing, social engineering, insider threats

Security is **not a feature**. It's a property of the entire system, only as strong as its weakest link.

---

## Why It Matters

### 1. Breaches are catastrophic
The average cost of a data breach in 2024 was **$4.88M** (IBM). Big breaches — Equifax, Target, Capital One, MOVEit, SolarWinds — cost hundreds of millions and erode trust for decades.

### 2. Regulation is everywhere
GDPR (EU), CCPA (California), HIPAA (US healthcare), PCI-DSS (payments), SOC 2 (B2B SaaS), ISO 27001, DPDP Act (India). Non-compliance fines are existential — Meta has been fined €1.2B+; British Airways £20M; Marriott £18M.

### 3. Attacks are commodified
Ransomware-as-a-Service, exploit kits, automated scanners, leaked credential dumps. Even teenagers with no skills can run attacks that took nation-states a decade to develop. **Your app is being scanned every day.**

### 4. Software runs the world
The shift from "websites" to "applications running businesses" means a security failure isn't a defaced page — it's leaked PII, drained bank accounts, halted hospitals, compromised power grids.

### 5. Trust is the moat
Security is brand. One breach can permanently associate your name with "data leak." Customers leave; competitors win.

---

## The Security Mindset

### 1. Assume breach
Plan as though the attacker is already inside. Networks are hostile. Internal services are not "trusted." Compromise of one component shouldn't compromise everything (zero trust).

### 2. Least privilege
Every user, service, token, and process should have the **minimum permissions necessary** to do its job — and nothing more. A compromised component limits the blast radius.

### 3. Defense in depth
Single defenses fail. Stack many: input validation + parameterized queries + WAF + monitoring + alerting. Each layer raises the cost for the attacker.

### 4. Fail securely
When a check fails, fail **closed** (deny) — not open (allow). A failed auth check should never default to "must be admin then."

### 5. Don't roll your own crypto
Use vetted libraries. The history of "we built our own crypto" is the history of breaches. Cryptography is a minefield where your intuition is wrong.

### 6. Validate input, encode output
Untrusted input flows in; sanitize/validate at the boundary. Untrusted output flows out; encode for the destination context (HTML, URL, JS, SQL, shell).

### 7. The attacker only needs to find one
You need to defend everywhere; they need one weakness. **Security is asymmetric.** Compensate with breadth, layering, and continuous testing.

### 8. Security is sociotechnical
Most breaches start with phishing or stolen credentials, not zero-days. Train people. Make secure paths the easy paths.

---

## The CIA Triad and Beyond

```
┌──────────────────────────────────────────────────────┐
│                     SECURITY GOALS                   │
├──────────────────────────────────────────────────────┤
│                                                      │
│        CONFIDENTIALITY                               │
│              │                                       │
│              │      "Only authorized parties         │
│              │       can read."                      │
│              │                                       │
│   INTEGRITY ─┼─ AVAILABILITY                         │
│              │                                       │
│  "Data       │     "Authorized parties               │
│   is correct │      can access when needed."         │
│   & untampered."                                     │
│                                                      │
│   + AUTHENTICITY (who said this?)                    │
│   + NON-REPUDIATION (you can't deny saying it)       │
│   + ACCOUNTABILITY (auditable trail)                 │
│   + PRIVACY (what data, to whom, why, when)          │
│                                                      │
└──────────────────────────────────────────────────────┘
```

Different threats target different goals. A DDoS attacks **availability**; an XSS attacks **confidentiality** and **integrity**; a tampered audit log attacks **non-repudiation**.

---

## Defense in Depth: Layers of Security

```
┌────────────────────────────────────────────────────────────┐
│                  LAYERED DEFENSES                          │
├────────────────────────────────────────────────────────────┤
│                                                            │
│  ┌──────────────────────────────────────────────────┐      │
│  │            People & Process                      │      │
│  │  Training, security reviews, threat modeling,    │      │
│  │  least privilege, code review, IR runbooks       │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Network & Edge                        │      │
│  │  TLS, HSTS, WAF, DDoS, rate limiting, bot mgmt   │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Identity & Access                     │      │
│  │  AuthN (MFA/passkeys), AuthZ (RBAC/ABAC),        │      │
│  │  session management, JIT access                  │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Application                           │      │
│  │  Input validation, output encoding, CSRF tokens, │      │
│  │  CSP, secure deps, error handling                │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Data                                  │      │
│  │  Encryption at rest, key management, masking,    │      │
│  │  tokenization, retention/deletion                │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Infrastructure                        │      │
│  │  Hardened OS, patching, segmentation, secrets    │      │
│  │  mgmt, IaC scanning, container scanning          │      │
│  └──────────────────────────────────────────────────┘      │
│  ┌──────────────────────────────────────────────────┐      │
│  │            Monitoring & Response                 │      │
│  │  Logs, SIEM, alerts, anomaly detection,          │      │
│  │  incident response, post-mortems                 │      │
│  └──────────────────────────────────────────────────┘      │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Each layer should assume the others may fail.**

---

## Threat Modeling (STRIDE)

Before building, model the threats. Microsoft's **STRIDE** framework:

| Letter | Threat | Property violated |
|---|---|---|
| **S** | Spoofing | Authentication |
| **T** | Tampering | Integrity |
| **R** | Repudiation | Non-repudiation |
| **I** | Information disclosure | Confidentiality |
| **D** | Denial of service | Availability |
| **E** | Elevation of privilege | Authorization |

### Process
1. **Decompose** the system — diagram data flows, trust boundaries, components, stores.
2. **Identify threats** — for each component/flow, walk through STRIDE.
3. **Rank** — likelihood × impact.
4. **Mitigate** — pick controls; document accepted risks.
5. **Validate** — pen-test, red-team, automated scanning.

> **In practice:** the value of threat modeling is the *thinking*, not the document. A 30-minute whiteboard session before each major feature catches more bugs than a 200-page document nobody reads.

### Other frameworks
- **PASTA** — Process for Attack Simulation and Threat Analysis
- **DREAD** — Damage, Reproducibility, Exploitability, Affected users, Discoverability (deprecated by MS but still useful)
- **MITRE ATT&CK** — adversary tactics & techniques catalog (post-breach focus)

---

## Authentication

**Authentication (AuthN)** = "Who are you?"

### The factor model
- **Knowledge** — something you know (password, PIN)
- **Possession** — something you have (phone, hardware key)
- **Inherence** — something you are (fingerprint, face)
- **Location** — where you are (IP, GPS — weakest)

**MFA** = ≥ 2 distinct factors. **Adaptive MFA** = require MFA only on risk signals (new device, suspicious location, sensitive action).

### Modern best practice (2026)
1. **Passkeys / WebAuthn** — phishing-resistant, public-key-based, no shared secret. **The new gold standard.** Apple, Google, Microsoft, Amazon all support them.
2. **OAuth / OIDC SSO** — delegate to identity providers (Google, Microsoft, Apple, Okta, Auth0, Clerk).
3. **TOTP** (authenticator apps) — better than SMS.
4. **SMS OTP** — last resort. SIM swap and SS7 attacks make it weak.
5. **Passwords** — still common; treat as legacy and harden them.

### If you must do passwords
- **Enforce strong passwords** — but *measure entropy*, not just length/character classes. NIST SP 800-63B recommends min length 8, no forced rotation, no composition rules; check against breached-password lists (HaveIBeenPwned k-anonymity API).
- **Hash with a slow KDF** — Argon2id (recommended), scrypt, or bcrypt. **Never MD5, SHA-1, SHA-256 alone, or PBKDF2 with low iterations.**
- **Salt** — unique per user (the modern KDFs do this for you).
- **Pepper** — server-side secret added to the password before hashing; protects if DB leaks but server doesn't.
- **Rate-limit login attempts** — per-account (slow-down) and per-IP (block).
- **Generic error messages** — don't reveal whether the username or password was wrong (timing attacks too — use constant-time comparison).
- **Email-based account-recovery** must be rate-limited and time-bound.

### Phishing-resistant flow (passkeys)
```
User      Browser/Authenticator       Server
 │              │                       │
 │ Sign in      │                       │
 ├─────────────►│                       │
 │              │  Request challenge    │
 │              ├──────────────────────►│
 │              │  challenge + RP-ID    │
 │              │◄──────────────────────┤
 │              │                       │
 │ Authorize    │                       │
 │ (biometric)  │                       │
 │◄─────────────┤                       │
 │              │  signed assertion     │
 │              ├──────────────────────►│
 │              │  session              │
 │              │◄──────────────────────┤
```

Why it's phishing-resistant: the authenticator binds the assertion to the **origin (RP-ID)**. A fake site at a different domain can't get a valid signature.

---

## Authorization

**Authorization (AuthZ)** = "What are you allowed to do?"

### Models
| Model | How |
|---|---|
| **DAC** (Discretionary Access Control) | Owners grant access (file system permissions) |
| **MAC** (Mandatory Access Control) | System-enforced rules (military classification) |
| **RBAC** (Role-Based) | Users have roles; roles have permissions |
| **ABAC** (Attribute-Based) | Decisions based on user, resource, action, environment attributes |
| **ReBAC** (Relationship-Based) | Decisions based on relationships (Google Zanzibar; "does Alice own folder X?") |
| **PBAC** (Policy-Based) | OPA/Cedar — declarative policies as code |

### Modern stacks
- **OPA (Open Policy Agent)** — Rego language; cloud-native standard
- **Cedar** (AWS) — designed for app-level authz
- **OSO** — embedded authz library
- **SpiceDB** / **Authzed** — ReBAC inspired by Google Zanzibar
- **Permit.io**, **Aserto**, **Oso Cloud** — managed authz

### Best practices
- **Centralize authz** — don't sprinkle `if (user.role === 'admin')` across 200 files
- **Default deny** — if a permission is unspecified, deny
- **Authorize at every layer** — UI, API, service-to-service, DB row-level (Postgres RLS)
- **Authorize the actual data**, not the route — verify the user can access *this specific resource*
- **Tenant isolation** — for multi-tenant SaaS, every query filters by tenant; RLS enforces it; tested rigorously
- **Audit-log** all sensitive authz decisions

### IDOR — Insecure Direct Object Reference
The most common authz bug. A URL like `/orders/12345` lets you change to `/orders/12346` and see someone else's order.

**Fix:** every resource fetch validates the actor has rights to *this* resource — not "they're logged in." UUIDs help (unguessable IDs) but **are not authorization** — never rely on URL secrecy.

---

## Sessions & Tokens

After authentication, the session establishes who the user is across requests.

### Session cookies (server-side sessions)
- Server stores session state; cookie holds an opaque session ID
- **Pros**: revocable instantly, small token, server controls everything
- **Cons**: requires server-side state (Redis/DB)
- **Use when**: traditional web apps, server-rendered, monolith or with sticky sessions

### JWT (JSON Web Tokens)
- Self-contained, signed (and optionally encrypted) tokens with claims (user ID, expiry, scopes)
- **Pros**: stateless, scales horizontally, works across services
- **Cons**: **revocation is hard** (must wait for expiry or maintain a deny-list); larger than session IDs; secret leakage is catastrophic; algorithm confusion attacks (see below)
- **Use when**: API auth, microservices, mobile apps

> **An unpopular take:** JWTs are widely overused. For most monolithic web apps, **session cookies are simpler and safer**. The only good reason to use JWTs is *stateless* multi-service auth.

### JWT pitfalls
- **`alg: none`** — historic vulnerability where servers accepted unsigned JWTs. Use a library that rejects this.
- **Algorithm confusion** — accepting RS256 vs HS256 means the public key becomes the secret. Pin the algorithm.
- **No revocation** — a stolen JWT is valid until expiry. Use **short access tokens** (5–15 min) + long-lived **refresh tokens** stored in HttpOnly cookies, with revocation lists.
- **Storing in `localStorage`** — readable by any XSS. Always use **HttpOnly Secure SameSite cookies**.

### Cookie attributes (the security ones)
```
Set-Cookie: session=abc123;
            HttpOnly;            ← JS can't read it (defends against XSS-token theft)
            Secure;              ← HTTPS only
            SameSite=Lax;        ← CSRF defense; use Strict for sensitive sites
            Path=/;
            Max-Age=3600;
            __Host- prefix       ← (in name) — required Secure, Path=/, no Domain
```

### OAuth 2.0 / OIDC quick reference
- **OAuth 2.0** — authorization framework (delegate access to a third party)
- **OIDC** — identity layer on top of OAuth (authentication)
- Use **Authorization Code Flow + PKCE** for SPAs and mobile apps. **Implicit flow is deprecated.**
- Use **client_credentials** for service-to-service.

---

## The OWASP Top 10 (2021/2025) — Common Attacks

The most influential security awareness document on the web. Updated every few years by OWASP. The 2021 list (latest stable; 2025 in draft):

| # | Category | Examples |
|---|---|---|
| **A01** | Broken Access Control | IDOR, missing function-level checks, path traversal |
| **A02** | Cryptographic Failures | Weak hashes, hardcoded secrets, plaintext at rest |
| **A03** | Injection | SQL, NoSQL, LDAP, OS command, ORM, template injection, XSS (now grouped here) |
| **A04** | Insecure Design | Missing rate limiting, business logic flaws, no threat model |
| **A05** | Security Misconfiguration | Default creds, verbose errors, unpatched deps, missing headers |
| **A06** | Vulnerable & Outdated Components | Old libraries, unmaintained packages, Log4Shell-class |
| **A07** | Identification & Auth Failures | Weak passwords, broken session mgmt, missing MFA |
| **A08** | Software & Data Integrity Failures | Unsigned updates, deserialization, supply-chain |
| **A09** | Security Logging & Monitoring Failures | No logs, no alerts, can't detect breach |
| **A10** | SSRF (Server-Side Request Forgery) | Server fetches attacker-controlled URLs |

> **Worth knowing:** the Top 10 is awareness, not a checklist. Real apps face threats not on this list (timing attacks, side channels, OAuth misconfig, JWT confusion, prototype pollution, race conditions). Use it as a starting point and go deeper.

---

## Cross-Site Scripting (XSS)

Attacker injects executable JavaScript into a page another user views. Once running, the script can read DOM, cookies (if not HttpOnly), keystrokes, request CSRF tokens — basically own the user's session.

### Types
- **Stored (Persistent)** — payload saved on server (a forum post), served to many users
- **Reflected** — payload in URL/query, reflected back in the response (search results, error messages)
- **DOM-based** — JS in the page reads attacker-controlled input (location, hash, postMessage) and writes it to the DOM unsafely

### Example
```html
<!-- vulnerable -->
<div>Hello, <%= name %></div>     <!-- if name = '<script>...</script>' -->

<!-- DOM-based -->
document.getElementById('out').innerHTML = location.hash.slice(1);
```

### Prevention
1. **Context-aware output encoding** — encode for HTML body, attribute, JS, URL, CSS. Don't paste user input as-is anywhere.
2. **Frameworks help** — React, Vue, Angular auto-escape by default. Use them; don't fight them.
3. **Avoid `dangerouslySetInnerHTML`** / `v-html` / `innerHTML` with user data. If you must, sanitize with **DOMPurify**.
4. **Content Security Policy (CSP)** — even if XSS slips through, CSP can block script execution. **Always deploy CSP.**
5. **HttpOnly cookies** — limits damage by hiding session tokens from JS.
6. **Trusted Types (Chrome)** — `Content-Security-Policy: require-trusted-types-for 'script'` enforces sanitization at sink points.
7. **Input validation** — defense-in-depth, but never the primary control (encoding is).

```js
// React auto-escapes; safe
<div>{userInput}</div>

// Dangerous; only with sanitized HTML
import DOMPurify from 'dompurify';
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(userInput) }} />
```

---

## Cross-Site Request Forgery (CSRF)

Attacker tricks an authenticated user's browser into sending a state-changing request to your site (e.g., transfer funds, change email).

```
User logged into bank.com.
Visits attacker.com, which contains:
   <img src="https://bank.com/transfer?to=evil&amt=10000">
Browser sends the request with the user's bank.com cookies. 💀
```

### Prevention
1. **`SameSite=Lax` cookies (default in modern browsers)** — cookies aren't sent on cross-site sub-requests like image loads or form posts. Lax for most apps; **Strict** for sensitive ops (banking).
2. **CSRF tokens (synchronizer pattern)** — server issues a per-session random token; client sends it on state-changing requests; server verifies. Frameworks (Rails, Django, Laravel, ASP.NET) include this out of the box.
3. **Double-submit cookie** — cookie value must match a request header. Stateless alternative.
4. **Origin / Referer header check** — verify the request came from your domain (bypassable in edge cases; defense-in-depth only).
5. **Re-authentication for sensitive actions** — password / MFA confirmation for changing email, password, transferring funds.

> **In practice:** with `SameSite=Lax` becoming the browser default, classic CSRF is significantly mitigated for cookie-auth sites. But it's not gone — same-site subdomain attacks, GET-state-changing endpoints, and cross-origin auth all still need explicit defense. Don't skip CSRF tokens.

---

## SQL Injection & Other Injections

User input ends up interpreted as code. The classic:
```sql
-- vulnerable
"SELECT * FROM users WHERE name = '" + userInput + "'";
-- userInput = "'; DROP TABLE users; --"
```

### Prevention
1. **Parameterized queries / prepared statements** — input is bound as data, never interpreted as SQL. **The only acceptable defense.**
   ```js
   // ✅
   db.query('SELECT * FROM users WHERE id = ?', [id]);
   ```
2. **ORMs** — most modern ORMs (Prisma, Drizzle, Sequelize, TypeORM, ActiveRecord) parameterize automatically. Watch for raw queries.
3. **Stored procedures** — only if they don't construct dynamic SQL inside.
4. **Input validation** — defense-in-depth.
5. **Least-privilege DB users** — the app's DB user should not be able to `DROP TABLE` or read other databases.

### Other injection classes
- **NoSQL injection** — MongoDB `{ $gt: '' }` operator injection; sanitize input shape
- **OS command injection** — `exec(`ping ${ip}`)` — use spawn with array args, never shell concatenation
- **LDAP injection** — escape special chars; use parameterized libraries
- **XPath / XQuery injection** — same idea
- **Template injection (SSTI)** — never render user input as a template
- **Header injection** — CRLF in headers; never concatenate user input into HTTP headers
- **Log injection** — sanitize before logging; otherwise attackers can forge log entries

### ORM injection: still possible
```js
// ❌ raw query — injectable
prisma.$queryRawUnsafe(`SELECT * FROM users WHERE name = '${name}'`);

// ✅ tagged template — parameterized
prisma.$queryRaw`SELECT * FROM users WHERE name = ${name}`;
```

---

## SSRF (Server-Side Request Forgery)

The server fetches a URL controlled by the attacker — often used to access **internal services** that shouldn't be reachable from the internet (cloud metadata endpoints, internal admin panels, databases).

### Classic attack: AWS metadata
```
GET /fetch?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/
```
The 2019 **Capital One breach** ($190M+) was an SSRF that read AWS IAM credentials, leading to 100M records exposed.

### Prevention
1. **Block requests to internal IPs and metadata endpoints** — `127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`, `::1`, link-local
2. **Use IMDSv2** on AWS — token-based, defeats classic SSRF metadata stealing
3. **Allowlist domains/protocols** — never accept arbitrary URLs; whitelist explicitly
4. **DNS rebinding defenses** — re-resolve after fetching; or fetch through a proxy that validates IPs
5. **Egress controls** — VPC egress firewall blocking unexpected destinations
6. **Disable redirect-following** in HTTP clients, or validate each hop

---

## Insecure Deserialization & Prototype Pollution

### Insecure deserialization
Untrusted data deserialized into objects can trigger RCE in many languages (Java, Python pickle, PHP, .NET, Ruby YAML). Don't deserialize attacker-controlled data with formats that allow arbitrary objects.
- Prefer **JSON** with strict schema validation
- For Node.js, never `eval()` user input
- For YAML, use `safeLoad`

### Prototype pollution (JS)
Attacker-controlled object merge that pollutes `Object.prototype`:
```js
// vulnerable merge
function merge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object') {
      merge(target[key] = target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
}
merge({}, JSON.parse('{"__proto__": {"isAdmin": true}}'));
console.log(({}).isAdmin); // true 💀
```

**Fix:**
- Use `Object.create(null)` for "dictionary" objects without prototype
- Use libraries that block `__proto__`, `constructor`, `prototype` keys (lodash post-4.17.20, modern set-value)
- Validate input shapes (Zod, Yup, Valibot)

---

## Clickjacking

Attacker embeds your site in a hidden iframe and tricks the user into clicking on it. Used to perform actions on your site as the user (transfer money, like a page, change settings).

### Prevention
1. **`X-Frame-Options: DENY`** (or `SAMEORIGIN`) — legacy but still respected by browsers
2. **`Content-Security-Policy: frame-ancestors 'none'`** — modern replacement, more flexible
3. Both? Yes, for defense-in-depth.

```
X-Frame-Options: DENY
Content-Security-Policy: frame-ancestors 'self' https://trusted.com
```

---

## Open Redirects

```
https://yoursite.com/login?redirect=https://evil.com
```
After login, the user is redirected to attacker's site — used in phishing, OAuth abuses (state parameter manipulation), CSRF chaining.

### Prevention
- **Allowlist** redirect destinations (relative paths, or explicitly approved domains)
- Never redirect to user-controlled absolute URLs
- For OAuth `redirect_uri`, the IdP enforces an allowlist; never disable that

---

## Security Headers

Set them at the edge or framework. Test with [securityheaders.com](https://securityheaders.com).

```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-...' 'strict-dynamic'; ...
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
Cross-Origin-Resource-Policy: same-origin
```

| Header | What it does |
|---|---|
| `Strict-Transport-Security` (HSTS) | Force HTTPS for the duration; preload protects first visit |
| `Content-Security-Policy` | Whitelist resources; mitigate XSS — see below |
| `X-Content-Type-Options: nosniff` | Stop MIME sniffing — file is what the server says it is |
| `X-Frame-Options` / `frame-ancestors` | Anti-clickjacking |
| `Referrer-Policy` | Control how much of the URL leaks in `Referer` |
| `Permissions-Policy` | Disable browser features (camera, mic, geo) the app doesn't need |
| `Cross-Origin-Opener-Policy` (COOP) | Isolate browsing context group; defense against Spectre |
| `Cross-Origin-Embedder-Policy` (COEP) | Required for SharedArrayBuffer; cross-origin isolation |
| `Cross-Origin-Resource-Policy` (CORP) | Restrict who can embed this resource |

---

## CORS — Cross-Origin Resource Sharing

CORS is **not a security feature** in the way most people think. The browser's **Same-Origin Policy** is the security; CORS is a *relaxation* of it.

### The model
By default, JS at `a.com` cannot read responses from `b.com`. CORS lets `b.com` opt-in to specific origins.

```
GET /api/data
Origin: https://app.example.com

Response:
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Credentials: true
Access-Control-Allow-Methods: GET, POST
Access-Control-Allow-Headers: Content-Type, Authorization
```

### Common mistakes
- **`Access-Control-Allow-Origin: *` with `Allow-Credentials: true`** — browsers refuse this, but the misconfig itself is a sign of trouble
- **Reflecting `Origin` header without validation** — effectively allows any origin
- **Trusting CORS to protect you** — CORS doesn't prevent the request; it prevents the JS from reading the response. The request still hits your server. Authenticate every request.

### Preflight (OPTIONS)
For "non-simple" requests (custom headers, methods like PUT/DELETE), the browser sends an `OPTIONS` first. Cache it (`Access-Control-Max-Age`) to avoid the per-request RTT.

---

## Content Security Policy (CSP)

A CSP tells the browser what sources are allowed for scripts, styles, images, frames, etc. Even if XSS slips through, a strict CSP can prevent execution.

### A modern, strict CSP
```
Content-Security-Policy:
  default-src 'none';
  script-src 'self' 'nonce-r4nd0m' 'strict-dynamic';
  style-src  'self' 'nonce-r4nd0m';
  img-src    'self' data: https:;
  connect-src 'self' https://api.example.com;
  frame-ancestors 'none';
  base-uri 'none';
  form-action 'self';
  upgrade-insecure-requests;
  report-to default;
```

### Strategies
- **Strict CSP** (recommended) — `script-src 'nonce-...' 'strict-dynamic'`. Per-request random nonce on every inline script.
- **Hash-based** — instead of nonces, hash known inline scripts.
- **Allowlist** (legacy) — `script-src 'self' https://cdn.x.com` — easier to start, harder to maintain, weaker.
- **Avoid `'unsafe-inline'` and `'unsafe-eval'`** — they defeat the purpose.

### Rolling out CSP
1. Start with `Content-Security-Policy-Report-Only` mode → log violations
2. Fix violations (move inline scripts to external, add nonces, allowlist legitimate sources)
3. Switch to enforcing
4. Monitor `report-to` endpoint forever — new violations indicate XSS attempts or regressions

> **Lesson learned:** CSP is one of the highest-leverage defenses you can deploy. The first rollout is painful (lots of inline-script exceptions) — every team I've helped onboard hated me for two weeks — but afterward, every XSS attempt is contained.

---

## Transport Security (TLS, HSTS)

### TLS
- **TLS 1.2 minimum**, prefer **TLS 1.3** (faster handshake, fewer cipher choices, better defaults)
- **No SSLv2/3, no TLS 1.0/1.1** — disabled everywhere
- **Forward secrecy** ciphers (ECDHE) — past sessions stay safe even if private key leaks later
- **Certificate transparency** — required for browser trust
- **Modern cipher suites** — `mozilla-modern` profile
- Test: `ssllabs.com/ssltest`

### HSTS (HTTP Strict Transport Security)
```
Strict-Transport-Security: max-age=63072000; includeSubDomains; preload
```
After first visit, the browser refuses HTTP for the duration. **Preload** submits your domain to a built-in browser list — even the first visit is HTTPS-only. Submit at [hstspreload.org](https://hstspreload.org).

### Mixed content
HTTPS pages must not load HTTP subresources. Browsers block actively-loaded mixed content (scripts) and increasingly block passively-loaded mixed content (images). Audit with `upgrade-insecure-requests` CSP directive.

---

## Cryptography Done Right

### Rules of thumb
- **Don't roll your own crypto.** Use battle-tested libraries: `libsodium` (NaCl), Node `crypto`, Python `cryptography`, AWS KMS, Google Tink.
- **Use high-level primitives.** `crypto.subtle.encrypt` not raw AES; `Tink AEAD` not custom GCM. Lower-level APIs have foot-guns.
- **Authenticated encryption** — always. AES-GCM, ChaCha20-Poly1305, AES-SIV. Never plain CBC without HMAC.
- **Random == secure random.** `crypto.randomBytes`, `crypto.getRandomValues`. Never `Math.random()` for security purposes.
- **Hash for integrity vs. password storage are different.** SHA-256/SHA-3 for integrity; Argon2id/bcrypt/scrypt for passwords.
- **Constant-time comparison** for secrets — `crypto.timingSafeEqual`. Naive `===` leaks via timing.
- **Key sizes:** RSA ≥ 2048 (3072 preferred), ECDSA P-256+ or Ed25519, AES-128 or 256.

### Key management
- **Never** hardcode keys in source. Never commit them. Never log them.
- Use **KMS** (AWS KMS, GCP KMS, Vault Transit) — keys never leave the HSM
- **Rotate** keys on a schedule and after any suspected compromise
- **Envelope encryption** — DEK encrypts data; KEK in KMS encrypts DEK

### What to encrypt
- **At rest** — disks, backups, databases (transparent or column-level)
- **In transit** — TLS everywhere, including internal service-to-service (mTLS)
- **In use** — confidential computing for the highest-sensitivity workloads (TEEs/enclaves)

### Tokenization vs. encryption
For PII (SSNs, credit cards), **tokenization** (replace with a non-sensitive surrogate) is often safer than encryption — no key to leak, no decryption oracle.

---

## Secrets Management

### Principles
- **Don't put secrets in code.** Not in `.env` checked into git, not in Docker images, not in logs.
- **Don't put secrets in environment variables forever.** They leak (process listings, error reports, child processes).
- **Use a secrets manager:** AWS Secrets Manager, GCP Secret Manager, HashiCorp Vault, Doppler, 1Password Secrets, Infisical
- **Rotate** routinely and after suspected exposure
- **Short-lived credentials** — IAM roles, OIDC federation, workload identity. Beats long-lived keys.

### Detection
- **Pre-commit hooks** — `gitleaks`, `detect-secrets`, `truffleHog`
- **CI scanning** — block on detected secrets
- **GitHub secret scanning** — built-in for public repos; partner program for popular tokens
- **Runtime scanning** — DLP, log scrubbing

### When secrets leak (and they will)
- **Rotate immediately** — don't wait
- **Audit logs** — what did the secret access?
- **Notify** — customers, regulators, providers as required
- **Post-mortem** — how did it leak? Process change?

---

## Supply Chain Security

Modern apps depend on hundreds (or thousands) of third-party packages. The supply chain is a major attack surface.

### Famous incidents
- **event-stream (2018)** — npm package `event-stream` had a malicious dependency targeting Bitcoin wallets
- **ua-parser-js (2021)** — compromised npm package mined cryptocurrency
- **Log4Shell (2021)** — Log4j RCE (`${jndi:ldap://...}`) affecting millions of Java apps
- **SolarWinds (2020)** — nation-state inserted backdoor into update mechanism
- **xz utils (2024)** — multi-year nation-state backdoor in upstream open source nearly shipped to all major Linux distros

### Defenses
1. **Lockfiles** — `package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`. Pin transitive deps. Required.
2. **Vulnerability scanning** — Dependabot, Renovate, Snyk, Socket, GitHub Advanced Security. Auto-PR on CVEs.
3. **SBOM (Software Bill of Materials)** — CycloneDX or SPDX format; required by US Executive Order 14028 for federal software.
4. **Provenance / SLSA** — sign build artifacts; verify chain of custody.
5. **Sigstore / cosign** — sign and verify packages and container images.
6. **Restrict install scripts** (`npm install --ignore-scripts`) — many supply-chain attacks abuse postinstall hooks.
7. **Use `--frozen-lockfile`** in CI — fail if lockfile is out of date.
8. **Dependency review** — when adding a new dep, evaluate maintenance, popularity, recent commits, known issues. Use [Socket](https://socket.dev) for behavioral analysis.
9. **Vendor / fork** critical small deps — sometimes copying 30 lines is safer than depending.
10. **Container scanning** — Trivy, Grype on every build.

> **In practice:** modern supply-chain attacks succeed because most teams' "dependency management" is "we updated when something broke." Auto-updates with vulnerability scanning + a real review process is the bare minimum.

---

## Logging, Monitoring, Incident Response

You can't defend what you don't see.

### What to log
- **Authentication events** — successes, failures, MFA challenges, password resets
- **Authorization decisions** — denials especially
- **Sensitive actions** — data exports, permission changes, admin actions
- **Errors and stack traces** — for debugging, but **never log secrets, tokens, full PII**
- **Anomalies** — repeated failures, unusual times, geo-impossible logins ("impossible travel")

### What NOT to log
- Passwords, even hashed
- Full credit card numbers (PCI-DSS violation)
- Auth tokens, session IDs
- API keys
- Full PII unless required and access-controlled
- Full request bodies of sensitive endpoints

### Where logs go
- **Centralized SIEM** — Datadog, Splunk, Elastic, Sumo Logic, Wazuh, Falco
- **Tamper-resistant** — append-only, signed, separate access control from main app
- **Retention** — long enough for forensics (90+ days standard, longer for regulated industries)

### Monitoring & alerts
- Alert on **anomalies**, not just absolute thresholds — sudden spike in 401s, unusual data access patterns, brute force, exfiltration sizes
- **Honeypots / canaries** — fake credentials/files that trigger alerts when accessed
- **WAF rules** — generic + custom for your app

### Incident Response
The **NIST IR lifecycle**: Prepare → Detect → Analyze → Contain → Eradicate → Recover → Lessons learned.

- **Runbooks** for common scenarios — credential leak, ransomware, DDoS, data breach
- **Communication tree** — who calls whom, who talks to legal, who talks to customers
- **Evidence preservation** — don't reboot the compromised host; snapshot, isolate
- **Tabletop exercises** quarterly — walk through scenarios; find the gaps before real incident
- **Post-mortems** — blameless, public-when-possible, improve controls

> **Worth knowing:** the difference between a small incident and a catastrophic one is usually **detection time**. Mature orgs detect breaches in days; immature ones detect in months (industry average is ~200 days). Invest in detection.

---

## Privacy, GDPR, and Data Protection

### Principles (GDPR, but broadly applicable)
- **Lawful basis** — every data use must have a legal basis (consent, contract, legitimate interest, etc.)
- **Data minimization** — collect only what you need, keep only as long as needed
- **Purpose limitation** — data collected for X can't suddenly be used for Y
- **Transparency** — clear privacy notices, no dark patterns
- **Subject rights** — access, rectification, erasure ("right to be forgotten"), portability, objection
- **Data Protection Impact Assessment** for high-risk processing
- **Privacy by Design and by Default**

### Practical engineering
- **Tag PII** at ingest — schemas mark fields as PII so tooling can find/redact
- **Encryption at rest** with KMS
- **Pseudonymization / tokenization** where possible
- **Access logging + DLP** for sensitive data
- **Right-to-erasure pipelines** — every system that holds user data must delete on request, including backups and replicas
- **Cross-border transfers** — Schrems II, SCCs, data residency requirements
- **Breach notification** — GDPR requires within 72 hours of awareness

### Cookies and consent
- Strictly necessary cookies don't need consent; marketing/analytics do
- Use a CMP (Consent Management Platform) — OneTrust, Cookiebot, Iubenda, Klaro
- Don't use dark patterns ("Accept all" prominent vs. "Reject" buried) — increasingly fined under GDPR

---

## Best Practices Checklist

### Application
- [ ] Threat model done; updated when architecture changes significantly
- [ ] All input validated at boundaries; output encoded for context
- [ ] Parameterized queries everywhere; no string-concat SQL
- [ ] Auth uses passkeys / OAuth-OIDC / strong passwords + MFA
- [ ] Sessions use HttpOnly Secure SameSite cookies, short-lived, rotatable
- [ ] CSRF tokens (or strict SameSite) on state-changing endpoints
- [ ] Authorization centralized; default deny; tested for IDOR
- [ ] Rate limiting on auth, password reset, expensive endpoints
- [ ] CSP enforced (nonce + strict-dynamic)
- [ ] Security headers full set (HSTS, COOP/COEP, X-CTO, frame-ancestors)
- [ ] No `dangerouslySetInnerHTML` / `innerHTML` with unsanitized input
- [ ] No `eval`, no `new Function(userInput)`
- [ ] Error pages don't leak stack traces, file paths, or DB details
- [ ] File uploads: validated content-type, scanned, stored outside webroot, served with `Content-Disposition: attachment` where appropriate

### Infrastructure
- [ ] HTTPS everywhere, TLS 1.2+, HSTS preloaded
- [ ] Internal services use mTLS
- [ ] WAF in front of the app
- [ ] DDoS mitigation (CDN + service)
- [ ] Network segmentation, egress controls
- [ ] Secrets in a vault, not env or code
- [ ] No SSH keys lying around; use SSO + IAM
- [ ] Patching automated; SLA for criticals (24-48h)
- [ ] Container/image scanning in CI
- [ ] Backups encrypted, tested with restore drills

### Identity
- [ ] MFA required for all admin / sensitive accounts
- [ ] Passkeys offered; SMS OTP minimized
- [ ] No shared accounts; every action attributable
- [ ] Just-in-time access for elevated permissions
- [ ] Quarterly access reviews
- [ ] Offboarding revokes everything within hours

### Code & supply chain
- [ ] Pre-commit secret scanning
- [ ] CI vulnerability scanning (deps + containers)
- [ ] Lockfiles committed, frozen in CI
- [ ] Dependabot / Renovate enabled with auto-merge for patch
- [ ] SAST (Semgrep, CodeQL) on every PR
- [ ] DAST against staging
- [ ] Annual penetration test

### Operations
- [ ] Centralized logging; PII not logged
- [ ] Alerts on anomalies; runbooks documented
- [ ] Incident response plan; tabletop exercises quarterly
- [ ] Post-mortems blameless; controls improved
- [ ] On-call rotation for security incidents

### Privacy
- [ ] Privacy notice published, accurate, current
- [ ] Data inventory (what, where, why, retention)
- [ ] Subject rights pipelines (access, deletion)
- [ ] DPA / SCC with vendors
- [ ] Cookie consent properly implemented
- [ ] DPO designated where required

---

## Common Pitfalls

### 1. "Security by obscurity"
Hiding the URL, obfuscating the JS, naming the admin endpoint `/sl4ck-admin-94f8/`. None of this is security. **Assume the attacker reads your minified JS, decompiles your binary, and has your sitemap.**

### 2. Trusting the client
Validation in JS is for UX, not security. Every constraint must be enforced server-side. Hidden fields, disabled buttons, client-only checks — all bypassable.

### 3. Logging sensitive data
"Just log everything" → 6 months later, an engineer realizes payment details, passwords, and PII are in CloudWatch in plaintext. **Audit what you log.**

### 4. Trusting the network
"Internal service, no auth needed." Then someone misconfigures a route, an attacker compromises a peer service, or a developer's laptop gets pwned. **Zero trust applies internally too.** Authenticate every request.

### 5. Generic error messages too generous
"Username not found" vs. "Invalid credentials." Reveals which usernames exist. Use generic messages on auth flows.

### 6. Race conditions in business logic
Withdrawing twice from the same balance. Double-spending. Use transactions, locks, idempotency keys, and concurrent-access tests.

### 7. Mass assignment
`User.update(req.body)` lets attackers set `is_admin: true`. Allowlist fields explicitly. Frameworks (Rails strong params, NestJS DTOs) help.

### 8. Insecure defaults in libraries
`jwt.verify(token, secret)` accepts `alg: none` in some versions. Always specify the expected algorithm. Read the docs of every security-relevant library.

### 9. The "we'll fix it later" backlog
Security tickets get deprioritized for features. Fixed by: budgeting time per sprint, treating security bugs like outages, executive sponsorship.

### 10. Ignoring the human layer
Phishing causes >80% of breaches. Train staff. Run simulated phishing. Make reporting suspicious emails easy and praised.

### 11. Excessive permissions for service accounts
"It needs S3, Dynamo, and a few other things — let's give it `*`." Compromise of the service then compromises everything. Per-resource, per-action IAM.

### 12. Forgetting deletion
GDPR/CCPA require deletion. Backups, replicas, search indexes, BI warehouses, log archives, vendor exports — easy to forget; expensive to discover late.

### 13. Trusting "it's behind VPN"
VPNs aren't a security model — they're a transport. Once attackers are inside, they pivot. Use zero trust + identity-aware proxies (BeyondCorp model).

### 14. Public S3 buckets
A perpetual greatest-hits of breaches. Default to private; review every public bucket; use AWS Block Public Access account-wide.

### 15. Treating compliance as security
PCI-DSS / SOC 2 / ISO 27001 are *floors*, not ceilings. You can be compliant and insecure. Compliance is paperwork; security is engineering.

---

## Further Reading

### OWASP (free, foundational)
- **[OWASP Top 10](https://owasp.org/www-project-top-ten/)** — start here
- **[OWASP ASVS (Application Security Verification Standard)](https://owasp.org/www-project-application-security-verification-standard/)** — a comprehensive checklist by maturity level
- **[OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)** — pragmatic per-topic guides; bookmark this
- **[OWASP Threat Modeling](https://owasp.org/www-community/Threat_Modeling)**
- **[OWASP SAMM](https://owaspsamm.org/)** — Software Assurance Maturity Model

### Books
- **Adam Shostack — *Threat Modeling: Designing for Security***
- **Dafydd Stuttard & Marcus Pinto — *The Web Application Hacker's Handbook*** (dated but foundational)
- **Bruce Schneier — *Applied Cryptography***, *Cryptography Engineering* (with Ferguson & Kohno)
- **Michal Zalewski — *The Tangled Web***
- **Tobias Klein — *A Bug Hunter's Diary***
- **Andrew Hoffman — *Web Application Security*** (O'Reilly)

### Specs / Standards
- **NIST SP 800-63** — Digital Identity Guidelines
- **NIST SP 800-53** — Security & Privacy Controls
- **NIST Cybersecurity Framework**
- **CIS Benchmarks** — hardening guides per platform
- **ISO 27001 / 27017 / 27018**
- **PCI-DSS** (payments), **HIPAA** (US health), **GDPR** (EU privacy)

### Web-specific
- **[web.dev — Security](https://web.dev/articles/secure)**
- **[MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)**
- **[Google Web Fundamentals — Security](https://developers.google.com/web/fundamentals/security)**
- **[Mozilla Observatory](https://observatory.mozilla.org/)** — free header check
- **[securityheaders.com](https://securityheaders.com)**
- **[Have I Been Pwned](https://haveibeenpwned.com/)** — breach data + Pwned Passwords API

### Hands-on / training
- **[PortSwigger Web Security Academy](https://portswigger.net/web-security)** — free, world-class, hands-on labs
- **HackTheBox**, **TryHackMe** — CTF-style labs
- **OWASP Juice Shop** — intentionally vulnerable training app

### Blogs / people to follow
- **Troy Hunt** (HaveIBeenPwned, Pluralsight)
- **Scott Helme** (security headers, observatory)
- **Filippo Valsorda** (Go crypto, age, mkcert)
- **Tavis Ormandy** (Project Zero — exploit research)
- **PortSwigger Research** — annual "Top 10 web hacking techniques"
- **Krebs on Security** — investigative breach journalism
- **Schneier on Security** — analysis & policy

---

> **Closing thought:** security is not a product, not a feature, not a sprint. It's a property of *every* line of code, *every* configuration, *every* decision — sustained over years. The security engineers I respect most don't memorize CVEs; they internalize the principles (least privilege, defense in depth, fail securely, validate input, encode output) and apply them everywhere, on every project, without making a fuss about it. There's no such thing as a perfectly secure system, and chasing one is a fool's errand. The goal is to make attacks expensive, detection fast, and recovery quick. **Build for breach — because breach builds for you whether you're ready or not.**
