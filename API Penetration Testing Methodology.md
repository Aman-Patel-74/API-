# API Penetration Testing Methodology

A structured methodology ensures complete coverage of authentication, authorization, data handling, and business logic vulnerabilities in APIs.

This guide aligns with the **OWASP API Security Top 10** and modern API attack techniques.

---

## 1. Reconnaissance & Discovery

### 1.1 Identify Base URL

- `https://api.target.com`
- `https://target.com/api`
- Check versions:
  - `/v1/`
  - `/v2/`
  - `/beta/`

### 1.2 Discover Documentation

- `/swagger`
- `/swagger-ui`
- `/v3/api-docs`
- `/openapi.json`
- `/docs`

Check for:
- Public OpenAPI specs
- GraphQL endpoints (`/graphql`)
- Postman collections

---

## 2. Authentication Analysis

Identify how the API authenticates users.

### 2.1 Common Authentication Mechanisms

- Session Cookies
- Bearer Tokens
- JWT
- API Keys
- OAuth 2.0
- mTLS

If JWT is used:
- Decode token
- Check:
  - `alg` field
  - Expiry (`exp`)
  - Role claims
- Test:
  - Algorithm confusion
  - Signature bypass
  - Token tampering
  - None algorithm

---

## 3. Authorization Testing (Broken Access Control)

One of the most critical API vulnerabilities.

### 3.1 BOLA (Broken Object Level Authorization)

**Example:**
```
GET /api/users/1001
```
Change:
```
GET /api/users/1002
```
Check if you can access another user's data.

### 3.2 Vertical Privilege Escalation

- Access `/admin`
- Modify roles
- Access restricted endpoints

### 3.3 Horizontal Privilege Escalation

- Access other user resources
- Modify other user profiles

---

## 4. Endpoint Enumeration

### 4.1 Methods

- Analyze JS files
- Review mobile app traffic
- Check API documentation
- Fuzz endpoints
- Use wordlists

Look for:
- Hidden admin endpoints
- Debug endpoints
- Backup versions
- Deprecated APIs

### 4.2 HTTP Method Testing

Test all methods:
- GET
- POST
- PUT
- PATCH
- DELETE
- OPTIONS

Check for:
- Method override headers
- Improperly restricted methods
- CORS misconfiguration

---

## 5. Input Validation Testing

Test all input parameters.

### 5.1 Injection Attacks

- SQL Injection
- NoSQL Injection
- Command Injection
- LDAP Injection
- Template Injection

### 5.2 JSON Attacks

- Nested JSON manipulation
- Unexpected properties (Mass Assignment)
- Type confusion
- Large payloads

**Example:**
```json
{
  "username": "user1",
  "role": "admin"
}
```
Check if backend improperly assigns role.

---

## 6. Business Logic Testing

APIs often fail in logic enforcement.

Test:
- Price manipulation
- Quantity tampering
- Workflow bypass
- Race conditions
- Replay attacks
- Refund abuse
- Coupon reuse

**Example:**
- Modify price before checkout
- Skip payment verification step

---

## 7. Rate Limiting & Brute Force

Test for:
- Login brute force
- OTP brute force
- Password reset abuse
- Enumeration attacks

Check:
- Rate limiting headers
- Lockout mechanisms
- CAPTCHA enforcement

---

## 8. Data Exposure Testing

Look for:
- Excessive data exposure
- Debug messages
- Stack traces
- Internal IP addresses
- API keys in responses
- Hidden fields in JSON

**Example:**
```json
{
  "id": 101,
  "email": "user@email.com",
  "passwordHash": "...",
  "internalNotes": "VIP customer"
}
```

---

## 9. API Configuration Testing

### 9.1 CORS Misconfiguration

Check:
- `Access-Control-Allow-Origin: *`
- Allow credentials with wildcard

### 9.2 Security Headers

- HSTS
- CSP
- X-Content-Type-Options

### 9.3 HTTP Response Codes

- Improper status codes
- 200 for failed requests

---

## 10. GraphQL Testing (If Applicable)

Check:
- Introspection enabled
- Query depth abuse
- Query batching
- Field suggestion attacks

**Example:**
```json
{
  __schema {
    types {
      name
    }
  }
}
```

---

## 11. File Upload Testing

Test:
- Extension bypass
- MIME type bypass
- Large file DoS
- Path traversal in filename

---

## 12. Automation & Fuzzing

Use:
- ffuf
- Postman
- curl

Fuzz:
- IDs
- Parameters
- Headers
- JSON keys

---

## 13. Logging & Monitoring Weaknesses

Test:
- Invalid login attempts logged?
- Token misuse detected?
- Admin access alerts?

---

## 14. API Pentest Execution Flow (Practical Workflow)

1. Recon & Documentation discovery
2. Intercept traffic
3. Map endpoints
4. Identify authentication
5. Test authentication flaws
6. Test authorization (BOLA/IDOR)
7. Test input validation
8. Test business logic
9. Test rate limiting
10. Test misconfigurations
11. Validate findings
12. Document impact

---

## API Pentest Worksheet Template

### Target Information

```
Target URL:
Authentication Type:
API Version:
Role Tested:
```

### Endpoint Mapping Table

| Resource | Method | Endpoint | Parameters | Auth Required | Role Required | Notes |
|----------|--------|----------|------------|----------------|----------------|-------|
|          |        |          |            |                |                |       |
|          |        |          |            |                |                |       |

### Vulnerability Tracking Table

| Vulnerability | Endpoint | Severity | Impact | Status |
|----------------|----------|----------|--------|--------|
|                |          |          |        |        |
|                |          |          |        |        |

---

## Reporting Structure

When reporting, include:

1. Endpoint
2. Request
3. Modified Request
4. Response
5. Impact
6. Proof of Concept
7. Remediation
