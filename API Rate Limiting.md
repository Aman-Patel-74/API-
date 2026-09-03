# API Rate Limiting

## What Is Rate Limiting

**Rate limiting** is a technique used to control the number of requests a client can send to an API within a specified time window. It helps protect the system from abuse, excessive traffic, and automated attacks such as brute-force attempts or denial-of-service (DoS) attacks.

By restricting request frequency, rate limiting ensures **API availability, stability, and fair usage among clients.**

---

## Why Rate Limiting Is Important

1. **Prevents DoS/DDoS Attacks**
   Without proper rate limiting, attackers may flood an API with thousands of requests per second, exhausting server resources and causing downtime.

2. **Protects Backend Resources**
   APIs often interact with databases, file systems, or external services. Excessive requests can overload these resources.

3. **Ensures Fair Usage**
   Rate limiting prevents one client from consuming all available API resources.

4. **Improves System Stability**
   Controlled request flow ensures the API remains responsive and available to legitimate users.

5. **Mitigates Brute Force Attacks**
   Login endpoints and sensitive operations become safer when repeated attempts are restricted.

---

## How Rate Limiting Works

Rate limiting is typically implemented based on:

### 1. Request Threshold
Maximum number of requests allowed.

Example:
- 100 requests per minute
- 10 requests per second

### 2. Time Window
The duration over which requests are counted.

Examples:
- Per second
- Per minute
- Per hour
- Per day

### 3. Identification Mechanism
Rate limiting may be applied based on:
- IP Address
- API Key
- User Account
- Session Token
- JWT Token

---

## Case Study: Client-Controlled Repeat Parameter (Rate-Limit Bypass Risk)

**Example API endpoint:**
```http
POST /workshop/api/merchant/contact_mechanic HTTP/1.1
Host: 192.168.1.28:8888
```

**Request body:**
```json
{
  "mechanic_code": "TRAC_JHN",
  "problem_details": "test",
  "vin": "4CVZR19NKXF615223",
  "mechanic_api": "http://192.168.1.28:8888/workshop/api/mechanic/receive_report",
  "repeat_request_if_failed": true,
  "number_of_repeats": 100
}
```

### Security Risk Identified

The parameter:
```
number_of_repeats: 100
```
may allow the client to **trigger multiple backend requests automatically**, potentially bypassing server-side rate limiting.

If the backend processes this parameter directly, an attacker could:
- Generate **massive internal requests**
- Trigger **resource exhaustion**
- Cause **internal service flooding**

This is effectively a client-controlled loop that lets a single external request fan out into many internal requests — sidestepping any rate limit applied at the edge/gateway layer, since only the *initial* request is rate-limited, not the resulting internal fan-out.

---

## Rate Limiting Testing Methodology

During API security testing, the following steps are commonly performed:

### 1. Automated Request Flooding
Send multiple requests in a short time.

Example tools:
- Burp Suite Intruder
- Turbo Intruder
- ffuf
- Python scripts

**Goal:** Check if the API blocks excessive requests.

### 2. Observe Server Response
Indicators of rate limiting include responses such as:
```
HTTP 429 Too Many Requests
```
or headers like:
```
X-RateLimit-Limit
X-RateLimit-Remaining
```

### 3. Test Client-Controlled Loop Parameters
Test parameters such as:
```
number_of_repeats
repeat_request_if_failed
```
to check if they trigger backend loops.

---

## Impact

If rate limiting is missing or weak, attackers may:
- Perform **API abuse**
- Conduct **brute force attacks**
- Trigger **DoS conditions**
- Exhaust backend services
- Generate excessive internal API calls

This can lead to:
- Service degradation
- API downtime
- Increased infrastructure costs

---

## Recommended Mitigations

### 1. Implement Server-Side Rate Limiting
Example limits:
- 10 requests per second
- 100 requests per minute per user

### 2. Enforce Global and Per-User Limits
Rate limits should apply to:
- IP address
- User account
- API key

### 3. Ignore Client-Controlled Loop Parameters
Do not allow client parameters such as:
```
number_of_repeats
repeat_request_if_failed
```
to control backend request behavior.

### 4. Implement Request Throttling
Return response:
```
HTTP/1.1 429 Too Many Requests
```
with headers:
```
Retry-After: 60
```

### 5. Use API Gateways or WAF
Examples include:
- API gateways
- Reverse proxies
- Web application firewalls

These can enforce rate limits before requests reach the backend.

---

## Conclusion

Rate limiting is a critical security control for protecting APIs from abuse, brute force attacks, and denial-of-service conditions. Proper implementation ensures fair resource usage and maintains API stability and availability.

## Possible Follow-Up Sections (for a fuller pentest report)

- Rate Limit Bypass Techniques
- Burp Suite / Turbo Intruder testing script
- Professional CVSS scoring for this issue
