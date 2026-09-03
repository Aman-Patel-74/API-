## API Security & Penetration Testing Notes

Personal study notes on API security concepts, the OWASP API Security Top 10, and hands-on findings from intentionally vulnerable lab applications (VAmPI, crAPI, a custom vulnerable e-commerce API, etc).

> ⚠️ **Disclaimer:** All testing documented here was performed against local/lab instances on private (RFC1918) IP addresses using intentionally vulnerable, publicly available training applications. Nothing here targets production systems. Use this material only against systems you own or are authorized to test.

---

## Table of Contents

- [1. API Fundamentals](#1-api-fundamentals)
- [2. Types of APIs](#2-types-of-apis)
- [3. API Authentication](#3-api-authentication)
- [4. OWASP API Security Top 10 (2023)](#4-owasp-api-security-top-10-2023)
- [5. Vulnerability Deep Dives](#5-vulnerability-deep-dives)
- [6. Lab Environments](#6-lab-environments)
- [7. Tooling & Learning Path](#7-tooling--learning-path)
- [8. Notes / Cleanup Log](#8-notes--cleanup-log)

---

## 1. API Fundamentals

An API (Application Programming Interface) is a structured interface that lets software systems communicate through predefined rules. It abstracts internal logic and exposes only the functionality clients need.

**Analogy:** an API is a restaurant menu — it tells you what you can order without exposing how the kitchen works.

| Concept | Role |
|---|---|
| Client | Places the request |
| API | The interface/menu |
| Server | Processes the request and returns a result |

### Core Components
- **Endpoint** — a URL representing a resource or action, e.g. `GET /users/123`
- **HTTP Methods** — GET, POST, PUT, PATCH, DELETE
- **Request** — headers, query params, body, cookies
- **Response** — status code, headers, body (JSON/XML)

### HTTP Status Code Classes
| Range | Meaning |
|---|---|
| 1xx | Informational |
| 2xx | Success |
| 3xx | Redirection |
| 4xx | Client error |
| 5xx | Server error |

### Object vs. Action
APIs are built around **objects** (nouns — `users`, `orders`, `accounts`) and **actions** (verbs — create, read, update, delete, or custom actions like `/cancel`, `/transfer`).

| Aspect | Object | Action |
|---|---|---|
| Type | Noun | Verb |
| Represents | Resource | Operation |
| Related vuln class | BOLA / IDOR | Broken Function-Level Authorization |

Understanding this split matters for testing: you need to check **both** whether a user can reach an object *and* whether they're allowed to perform the action on it.

### HTTP Protocol Notes
- Stateless, message-based, request/response, runs over TCP (HTTPS adds TLS).
- Dangerous methods enabled (e.g. `PUT` on an upload path) can lead to file upload vulnerabilities.
- `TRACE` enabled → potential Cross-Site Tracing (XST).
- Watch for missing security headers: `Content-Security-Policy`, `X-Frame-Options`, `X-Content-Type-Options`, `Strict-Transport-Security`.

---

## 2. Types of APIs

| Style | Description | Typical Use |
|---|---|---|
| **REST** | Stateless, resource-based, standard HTTP methods, usually JSON | Web/mobile backends, microservices |
| **SOAP** | XML-based protocol, WSDL, strict standards, WS-Security | Banking, enterprise, legacy |
| **GraphQL** | Single endpoint, client defines the response shape | Complex frontends, mobile |
| **gRPC** | HTTP/2 + Protocol Buffers, binary, high performance | Internal microservice comms |

**By exposure:** Public/Open APIs, Private/Internal APIs, Partner APIs.
**By context:** Web APIs, OS-level APIs, Library/Framework APIs.

---

## 3. API Authentication

*Authentication* = "who are you?" *Authorization* = "what are you allowed to do?"

| Method | How it works | Good for | Watch out for |
|---|---|---|---|
| **Basic Auth** | Base64 `user:password` in the `Authorization` header | Internal/testing use only | Must be HTTPS-only; credentials sent every request |
| **API Keys** | Static key in header/query/body | Service-to-service | No real identity, easy to leak in client-side code/logs |
| **OAuth 2.0** | Authorization framework issuing scoped, expiring access tokens | Third-party/delegated access | Complex to configure correctly; open redirect / state-validation bugs |
| **JWT** | Self-contained `Header.Payload.Signature` token | Stateless microservices | Hard to revoke; weak/hardcoded signing secrets; `alg: none` acceptance |
| **mTLS** | Client + server certificates | High-security, internal comms | Certificate lifecycle/revocation overhead |
| **HMAC** | Shared-secret request signing | Cloud/financial APIs (AWS-style) | Secret management, replay attacks without timestamps |
| **Session-based (cookies)** | Server-side session, cookie holds session ID | Traditional web apps | CSRF, session fixation/hijacking; not ideal for pure REST |

### Cookie- vs. Token-Based Auth

| Feature | Cookie-Based | Token-Based |
|---|---|---|
| Server state | Stateful | Stateless |
| Storage | Browser cookie | Header / client storage |
| Scalability | Limited | High |
| Revocation | Easy | Harder |
| CSRF risk | Higher | Lower (if header-based) |
| XSS risk | Lower (`HttpOnly`) | Higher (if `localStorage`) |

**Best practices:** HTTPS everywhere, short expirations, `HttpOnly`/`Secure`/`SameSite` cookies, signature + issuer/audience validation on tokens, rotate on login, invalidate on logout.

### Choosing a Method
| Scenario | Recommended |
|---|---|
| Internal APIs | mTLS or HMAC |
| Public APIs | OAuth 2.0 + JWT |
| Traditional web apps | Session + cookies |
| Microservices | JWT or mTLS |
| High-security environments | mTLS + OAuth |

---

## 4. OWASP API Security Top 10 (2023)

| # | Risk | Main Issue | Typical Impact |
|---|------|------------|-----------------|
| API1 | Broken Object Level Authorization (BOLA) | No ownership check on object IDs | Data exposure |
| API2 | Broken Authentication | Weak tokens/secrets, no rate limiting | Account takeover |
| API3 | Broken Object Property Level Authorization | Mass assignment of restricted fields | Privilege escalation |
| API4 | Unrestricted Resource Consumption | No rate limits, huge payloads/queries | Denial of service |
| API5 | Broken Function Level Authorization (BFLA) | Admin functions reachable by normal users | Admin compromise |
| API6 | Unrestricted Access to Sensitive Business Flows | No anti-automation on business logic | Fraud/abuse |
| API7 | Server-Side Request Forgery (SSRF) | Server fetches attacker-controlled URLs | Internal network compromise |
| API8 | Security Misconfiguration | Debug mode, default creds, bad CORS | Info leakage |
| API9 | Improper Inventory Management | Shadow/legacy API versions | Hidden attack surface |
| API10 | Unsafe Consumption of APIs | Blind trust in third-party responses | Supply-chain risk |

---

## 5. Vulnerability Deep Dives

Each subsection below is a condensed write-up from lab testing, kept in a consistent format: **what it is → how it was found → impact → fix.**

### 5.1 Broken Object Level Authorization (BOLA / IDOR)

**What:** The API doesn't verify that the authenticated user actually owns the object referenced by an ID/UUID in the request.

**Findings (crAPI):**
- `GET /identity/api/v2/vehicle/{vehicle_id}/location` — swapping in another user's vehicle UUID returns their live location.
- `GET /workshop/api/shop/orders/{order_id}` — sequential order IDs return other users' full order + partial payment/card data.
- `GET /workshop/api/mechanic/mechanic_report?report_id={id}` — sequential `report_id` values leak other users' vehicle VIN, mechanic contact info, and phone numbers.
- Combined with a **brute-forceable password-reset OTP** (4-digit OTP, no lockout — correct OTP found after ~83 attempts in Burp Intruder), this chain enables account takeover.

**Findings (custom e-commerce API):**
- `PUT /api/v1/orders/{id}/status` — any authenticated user can change any order's `status`/`paymentStatus`, e.g. mark someone else's unpaid order as `paid`/`delivered`.

**Fix:** Always check `resource.user_id == current_user.id` server-side before returning/modifying data; use access-control middleware; consider non-sequential/opaque IDs as defense-in-depth (not a substitute for authorization checks); log ID-enumeration patterns.

### 5.2 Broken Function Level Authorization (BFLA)

**What:** Sensitive/admin-only endpoints are reachable by low-privileged users because the server doesn't check role, only authentication.

**Finding (crAPI):** `DELETE /identity/api/v2/admin/videos/{id}` succeeded using a JWT with `"role": "user"`, allowing any authenticated user to delete platform videos.

**Fix:** Enforce RBAC server-side on every privileged route (`if user.role != "admin": return 403`), never rely on the frontend hiding admin UI, centralize checks in middleware, log privileged actions.

### 5.3 Mass Assignment (Broken Object Property-Level Authorization)

**What:** The API binds the entire request body to an internal model without an allow-list, so extra/hidden fields (`role`, `isAdmin`, `admin`, `credit`) get persisted.

**Findings:**
- **VAmPI:** injecting `"role":"admin"` into `POST /users/v1/register` to test whether it's silently accepted.
- **crAPI:** `PUT /workshop/api/shop/orders/{id}` with `"status":"returned"` triggers refund logic and inflates balance; negative `quantity` on order creation increases available credit instead of decreasing it (`$80 → $11,080` observed); `PUT /identity/api/v2/user/videos/{id}` exposes an internal `conversion_params` field that, if passed to a shell command (e.g. ffmpeg), could enable command injection.
- **Custom e-commerce API:** `PUT /api/v1/auth/profile` accepts and persists an `isAdmin` field not in the documented schema — a normal user can self-promote to admin in one request, and the elevated privilege is reflected on next login.

**Fix:** Whitelist editable fields server-side per endpoint; use DTOs instead of binding raw request bodies to models; require a separate, admin-only endpoint for role/privilege changes; validate business invariants server-side (e.g. `quantity > 0`, `price > 0`).

### 5.4 Excessive Data Exposure

**What:** The API returns full internal objects and relies on the client to filter, leaking fields like password hashes, secret keys, or debug data.

**Findings:**
- `/users/v1/_debug` (VAmPI) — debug endpoint exposed.
- `/service-report?id=N` (crAPI) — incrementing `id` returns other users' mechanic reports including PII (email, phone) with no ownership check — same root cause as the BOLA finding in 5.1, compounded by **no rate limiting** (100/100 requests to `contact_mechanic` returned `200 OK` with no throttling), which makes bulk enumeration trivial.

**Fix:** Return only the fields the client needs (DTOs), disable debug endpoints in production, apply least privilege to response payloads.

### 5.5 Unrestricted Resource Consumption / Rate Limiting

**What:** No limits on request volume, letting attackers brute-force, scrape, or cause resource exhaustion.

**Finding (crAPI):** `POST /workshop/api/merchant/contact_mechanic` accepts a client-supplied `number_of_repeats` field intended to control retries — if the backend trusts it, a single external request can fan out into many internal requests, bypassing edge/gateway rate limiting entirely (only the *initial* request is throttled, not the resulting internal loop).

**Testing methodology:** flood with Burp Intruder/Turbo Intruder/ffuf, look for `429 Too Many Requests` and `X-RateLimit-*` headers, specifically test any client-controlled "repeat"/"retry" parameters.

**Fix:** Server-side rate limiting per IP/user/API key, ignore client-supplied loop-control parameters, return `429` + `Retry-After`, enforce limits at an API gateway/WAF layer before requests reach application code.

### 5.6 SQL Injection

**Finding (crAPI):** `coupon_code` in `POST /workshop/api/shop/apply_coupon` is injectable — a time-based payload (`test';SELECT PG_SLEEP(5)--`) confirmed the injection via Burp Repeater. Full exploitation performed with `sqlmap` (`--dbs`, `--tables`, `--columns`, `--dump`), extracting user credentials and role data from a PostgreSQL backend.

**Finding (VAmPI):** the `name` path parameter on `GET /users/v1/<name>` is injectable against the underlying SQLite database; `sqlmap` was used to dump user credentials and other tables.

**Fix:** Parameterized queries / prepared statements only, strict input validation and typing, least-privilege DB accounts, don't leak DB errors to clients.

### 5.7 NoSQL Injection

**Finding (crAPI):** `POST /community/api/v2/coupon/validate-coupon` passes `coupon_code` directly into a MongoDB query. Sending `{"coupon_code":{"$gt":""}}` instead of a string causes the `$gt` operator to match *any* coupon, leaking a valid, unused coupon code that can then be redeemed for account credit.

**Fix:** Enforce strict types (reject objects where a string is expected), strip/block Mongo operator keys (`$gt`, `$ne`, `$regex`, …), apply schema validation, least-privilege DB roles.

### 5.8 Server-Side Request Forgery (SSRF)

**Finding (crAPI):** `mechanic_api` in `POST /workshop/api/merchant/contact_mechanic` is a user-controlled URL that the backend fetches server-side. Pointing it at an internal service (`http://<internal-ip>:8025`) returned the full HTML of an internal MailHog instance inside the API's own response — confirmed SSRF with real internal-service disclosure. The same parameter could plausibly be used to hit cloud metadata endpoints, scan internal ports, or exfiltrate data to an attacker-controlled host (untested in this lab, listed as a follow-up).

**Fix:** Never fetch a raw user-supplied URL; allowlist destination domains; block RFC1918/loopback/link-local ranges (including `169.254.169.254`); restrict to `http(s)://` only; validate the resolved IP post-DNS lookup; use IMDSv2 in cloud environments.

---

## 6. Lab Environments

| Lab | Style | Difficulty | Focus |
|---|---|---|---|
| [VAmPI](https://github.com/erev0s/VAmPI) | Monolithic Flask REST API | Low–Medium | JWT flaws, BOLA, mass assignment, rate limiting |
| [vapi](https://github.com/roottusk/vapi) | Lightweight REST API | Low | SQLi, auth bypass, IDOR |
| [OWASP crAPI](https://github.com/OWASP/crAPI) | Microservices | High | Full OWASP API Top 10, SSRF, business logic |
| [Damn Vulnerable Bank](https://github.com/rewanthtammana/Damn-Vulnerable-Bank/) | Full-stack banking sim | High | Business logic, transaction manipulation, IDOR |
| [Damn Vulnerable GraphQL App](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application) | GraphQL | Medium–High | Introspection abuse, GraphQL-specific IDOR/injection |
| Custom Vulnerable E-Commerce API (local project) | Node.js/Express REST | Medium | Mass assignment, BOLA, business-logic price/order flaws |

### Quick Setup Reference

**VAmPI**
```bash
git clone https://github.com/erev0s/VAmPI.git
cd VAmPI
docker-compose up -d
# or run the container directly:
docker run -d -e vulnerable=1 -e tokentimetolive=300 -p 5000:5000 erev0s/vampi:latest
```
Swagger UI: `http://<host>:5000/ui/` · Init DB: `http://<host>:5000/createdb`

**vapi**
```bash
git clone https://github.com/roottusk/vapi.git
cd vapi
docker-compose up
```

**crAPI**
```bash
git clone https://github.com/OWASP/crAPI.git
cd crAPI
docker-compose up
```

**Damn Vulnerable GraphQL App**
```bash
docker run -p 5013:5013 dolevf/dvga
```

**Damn Vulnerable Bank**
```bash
cd BackendServer && npm install && npm start
cd Frontend && npm install && npm start
```

**Custom E-Commerce API**
```bash
cd /opt/vulnerable-ecommerce-api
npm install
cp .env.example .env
npm run seed
npm start
# API: http://localhost:3000/api/v1  |  Docs: http://localhost:3000/api-docs
```
Test accounts: `admin / Admin123!` (admin), `alice / User123!` (user).

> **Note on IPs:** examples throughout this repo use lab-only RFC1918 addresses (`192.168.1.5`, `192.168.1.28`, `192.168.1.103`) from different testing sessions — they refer to the same lab running at different times/DHCP leases, not different targets. Swap in your own lab's IP.

### Suggested Progression
1. **VAmPI** — core REST vulnerabilities
2. **vapi** — injection practice
3. **crAPI** — full OWASP API Top 10, business logic, SSRF
4. **Damn Vulnerable Bank** — business logic at "production" complexity
5. **DVGA** — GraphQL-specific attack surface

### Recommended Attacker Environment
- Kali Linux, Docker
- Burp Suite Professional
- Postman / Insomnia
- `ffuf` / `wfuzz`, `sqlmap`, `jwt_tool`, `mitmproxy`
- Isolated host-only/NAT network — never mix lab traffic with production networks

---

## 7. Tooling & Learning Path

- **Recon:** Swagger/OpenAPI enumeration, Postman collections, endpoint diffing
- **Auth testing:** `jwt_tool`, manual JWT decode/re-sign, Burp Repeater for OTP brute force
- **Injection:** `sqlmap`, manual payloads (see `PayloadsAllTheThings` NoSQL/SQLi cheat sheets)
- **Fuzzing/rate-limit testing:** Burp Intruder / Turbo Intruder, `ffuf`
- **Reporting:** map every finding back to an OWASP API Top 10 category + CVSS score

### Possible Next Steps
- Map all findings above into a single OWASP API Top 10 tracking sheet
- Build a 30-day structured API pentesting roadmap
- Write a reusable Burp Suite API testing checklist
- Add a docker-compose file that spins up all labs together
- Standardize a per-finding report template (Description / PoC / Impact / CVSS / Remediation)

---

## 8. Notes / Cleanup Log

Things fixed or worth double-checking while consolidating the raw notes into this README:

- **Removed duplicate content:** two near-identical "API Penetration Testing Lab Setup" documents (VAmPI + crAPI write-ups) were merged into one; the VAmPI install/setup steps that were repeated a third time inside the Mass Assignment notes were also deduplicated.
- **Fixed a token-lifetime inconsistency:** one note ran `docker run -e tokentimetolive=3000 ...` but then explained the variable as "`300` → 5 minutes." 3000 seconds ≠ 5 minutes — the setup command above uses `300` (5 min) to match the explanation; confirm which value you actually intended before relying on it.
- **Inconsistent ports for VAmPI:** the same guide lists the service on port `5000` in most places but `5001` in the "Access URLs" table and the Hydra brute-force example. Pick one and update all references — otherwise commands will silently target the wrong port.
- **Placeholder IPs vary across notes** (`192.168.1.5`, `.28`, `.103`) — harmless if these are the same lab across sessions, but worth normalizing to one IP (or `<TARGET_IP>`) so copy-pasted commands work without editing.
- **Off-topic material left out of this README:** the raw notes also included general (non-API-pentest) material — a basic "What is an API" primer, Same-Origin Policy, and Uncomplicated Firewall (UFW) usage. These are fine topics but don't fit an *API pentesting* repo; consider a separate `general-security-notes` repo/section for them rather than mixing them in here.
- **Unverified/"smell" item kept as a caveat, not a confirmed finding:** the e-commerce API's order-total recalculation looked correct in testing (server recomputed the total from `salePrice` rather than trusting the client), but the original notes flagged that the client-supplied `price` field being accepted *at all* is worth a follow-up test — that caveat is preserved above rather than presented as a solved issue.
- **Trimmed for the README (kept in case you want them back in a separate `raw-notes/` folder):** the full literal `sqlmap` command chains with every table name, the complete raw `PayloadsAllTheThings` NoSQL payload dump, and the full chronological 18-step appendix from the e-commerce assessment — all valuable as raw evidence, but too granular for a top-level README. Consider a `findings/` folder per lab with the full request/response evidence, and let this README stay a summary + index.
