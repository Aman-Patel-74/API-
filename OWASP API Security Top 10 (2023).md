# OWASP API Security Top 10 (2023)

The **OWASP API Security Top 10** is published by OWASP (Open Web Application Security Project). It highlights the most critical security risks affecting APIs specifically — not traditional web app vulnerabilities.

**Latest stable release:** OWASP API Security Top 10 (2023)

---

## Quick Comparison Table

| Risk | Main Issue | Common Impact |
|------|------------|----------------|
| API1 | Object-level auth | Data exposure |
| API2 | Authentication flaws | Account takeover |
| API3 | Property-level auth | Privilege escalation |
| API4 | Resource abuse | DoS |
| API5 | Function-level auth | Admin compromise |
| API6 | Business logic abuse | Fraud |
| API7 | SSRF | Internal compromise |
| API8 | Misconfiguration | Info leakage |
| API9 | Poor inventory | Hidden attack surface |
| API10 | Unsafe third-party usage | Supply chain risk |

---

## API1: Broken Object Level Authorization (BOLA)

**What it is:**
APIs expose objects (users, orders, accounts). If the API does not properly verify ownership, attackers can access other users' data by manipulating object identifiers.

**Example:**
```http
GET /api/users/124
```
If user `123` can access user `124`'s data just by changing the ID → BOLA.

**Impact:**
- Data leakage
- Account takeover
- Financial fraud

**Prevention:**
- Always validate ownership server-side
- Do not rely on client-side checks
- Use proper authorization middleware

---

## API2: Broken Authentication

**What it is:**
Authentication mechanisms are implemented incorrectly, allowing attackers to compromise tokens or credentials.

**Common issues:**
- Weak JWT secrets
- Missing token expiration
- Brute force without rate limiting
- Accepting unsigned JWTs (`alg: none`)

**Impact:**
- Account takeover
- Privilege escalation

**Prevention:**
- Use strong token secrets
- Enforce expiration
- Implement MFA
- Rate limit login endpoints

---

## API3: Broken Object Property Level Authorization

**What it is:**
Also called a **Mass Assignment** vulnerability. APIs allow modification of properties that should not be user-controlled.

**Example:**
```json
{
  "username": "john",
  "role": "admin"
}
```
If `role` can be changed by the user → privilege escalation.

**Impact:**
- Privilege escalation
- Data corruption

**Prevention:**
- Whitelist allowed fields
- Use DTOs (Data Transfer Objects)
- Do not auto-bind entire objects

---

## API4: Unrestricted Resource Consumption

**What it is:**
APIs do not limit resource usage, allowing abuse.

**Examples:**
- No rate limiting
- Large payload uploads
- Expensive queries
- Infinite pagination

**Impact:**
- Denial of Service (DoS)
- Increased infrastructure cost

**Prevention:**
- Rate limiting
- Throttling
- Payload size limits
- Query depth limiting (GraphQL)

---

## API5: Broken Function Level Authorization

**What it is:**
Improper authorization checks on sensitive functions.

**Example:**
```http
POST /api/admin/deleteUser
```
If regular users can access admin endpoints → broken function-level authorization.

**Impact:**
- Full system compromise
- Data destruction

**Prevention:**
- Role-based access control (RBAC)
- Verify roles server-side
- Separate admin routes properly

---

## API6: Unrestricted Access to Sensitive Business Flows

**What it is:**
Business logic abuse due to lack of controls.

**Examples:**
- Unlimited OTP requests
- Coupon abuse
- Automated ticket booking
- Brute-forcing verification codes

**Impact:**
- Fraud
- Financial loss
- Service abuse

**Prevention:**
- Rate limiting per action
- CAPTCHA
- Anti-automation controls
- Behavioral monitoring

---

## API7: Server Side Request Forgery (SSRF)

**What it is:**
API fetches remote resources without validation, allowing attackers to make the server send requests to internal systems.

**Example:**
```json
{
  "url": "http://internal-service/admin"
}
```

**Impact:**
- Internal network scanning
- Cloud metadata theft
- Remote code execution (sometimes)

**Prevention:**
- Validate and sanitize URLs
- Use allowlists
- Block internal IP ranges
- Disable metadata service access

---

## API8: Security Misconfiguration

**What it is:**
Improper security settings.

**Examples:**
- Debug mode enabled
- Default credentials
- Unnecessary HTTP methods enabled
- Misconfigured CORS

**Impact:**
- Information disclosure
- Unauthorized access

**Prevention:**
- Harden configurations
- Disable unused features
- Secure CORS policies
- Regular security audits

---

## API9: Improper Inventory Management

**What it is:**
Lack of API documentation and version management.

**Examples:**
- Old API versions exposed
- Shadow APIs
- Unpatched legacy endpoints

**Impact:**
- Attackers target outdated endpoints
- Increased attack surface

**Prevention:**
- Maintain API inventory
- Deprecate old versions
- Monitor exposed endpoints
- Use API gateway logging

---

## API10: Unsafe Consumption of APIs

**What it is:**
Trusting third-party APIs without validation.

**Examples:**
- Blindly trusting third-party responses
- Not validating webhook signatures
- Insecure deserialization from external APIs

**Impact:**
- Data tampering
- Injection attacks
- System compromise

**Prevention:**
- Validate all third-party responses
- Verify webhook signatures
- Apply schema validation
- Use timeouts and circuit breakers
