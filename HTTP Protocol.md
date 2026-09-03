# HTTP Protocol

- HTTP stands for **Hypertext Transfer Protocol**.
- It is the foundational protocol used for communication between clients and servers on the web.

## Key Characteristics

- Application-layer protocol
- Message-based communication model
- Stateless protocol
- Typically runs over TCP (Transmission Control Protocol)
- Default ports:
  - HTTP → 80
  - HTTPS → 443

---

## Core Characteristics of HTTP

### 1. Message-Based Model

HTTP communication is based on a request-response model.

- Client sends a request.
- Server processes it.
- Server sends a response.

> Each interaction is an independent message exchange.

### 2. Stateless Protocol

HTTP is stateless, meaning:
- The server does not retain information about previous requests.
- Each request must contain all necessary information.

State is typically maintained using:
- Cookies
- Session IDs
- JWT tokens
- Local storage (client-side)

### 3. Runs Over TCP

HTTP uses TCP to ensure:
- Reliable delivery
- Ordered transmission
- Error checking

HTTPS adds TLS/SSL encryption on top of TCP.

---

## HTTP Communication Structure

HTTP consists of:
- Request
- Response
- Headers
- Body (optional)

---

## HTTP Request Structure

An HTTP request contains:
1. Request Line
2. Headers
3. Optional Body

**Example:**
```http
GET /api/users/101 HTTP/1.1
Host: example.com
Authorization: Bearer token123
User-Agent: Mozilla/5.0
```

---

## HTTP Response Structure

An HTTP response contains:
1. Status Line
2. Headers
3. Optional Body

**Example:**
```http
HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 45

{
  "id": 101,
  "name": "Rahul"
}
```

---

## HTTP Request Methods

HTTP defines multiple request methods, each representing a specific action.

| Method | Meaning |
|---|---|
| GET | Retrieve a resource |
| POST | Create a new resource |
| PUT | Replace an existing resource |
| DELETE | Remove a resource |
| HEAD | Retrieve headers only |
| OPTIONS | List available communication options |
| TRACE | Perform loop-back test |
| PATCH | Apply partial modifications |
| CONNECT | Establish a tunnel |

### GET
- Used to fetch data.
- Should not modify server state.
- Parameters are usually sent in URL.

**Example:**
```
GET /products?id=10
```

### POST
- Used to submit data.
- Often creates a new resource.
- Data is sent in request body.

**Example:**
```http
POST /users
Content-Type: application/json
```

### PUT
- Replaces the entire resource.
- Idempotent method.

**Example:**
```
PUT /users/101
```

### DELETE
- Removes a specified resource.

**Example:**
```
DELETE /users/101
```

### PATCH
- Partially updates a resource.
- Only specified fields are modified.

**Example:**
```
PATCH /users/101
```

### HEAD
- Same as GET but without response body.
- Used to fetch metadata.

**Example:**
```
HEAD /users/101
```

### OPTIONS
- Returns supported HTTP methods.
- Commonly used in CORS preflight requests.

**Example:**
```
OPTIONS /users
```

### TRACE
- Echoes the received request.
- Used for diagnostic purposes.
- Often disabled due to security risks.

### CONNECT
- Establishes a tunnel to the server.
- Commonly used with HTTPS through proxies.

---

## HTTP Response Status Codes

HTTP responses are categorized into five classes.

### 1XX – Informational

Indicates request received and processing continues.

Examples:
- 100 Continue
- 101 Switching Protocols

### 2XX – Successful

Request successfully processed.

Examples:
- 200 OK
- 201 Created
- 204 No Content

### 3XX – Redirection

Further action required to complete request.

Examples:
- 301 Moved Permanently
- 302 Found
- 304 Not Modified

### 4XX – Client Errors

Error caused by client request.

Examples:
- 400 Bad Request
- 401 Unauthorized
- 403 Forbidden
- 404 Not Found
- 405 Method Not Allowed
- 429 Too Many Requests

### 5XX – Server Errors

Server failed to fulfill valid request.

Examples:
- 500 Internal Server Error
- 502 Bad Gateway
- 503 Service Unavailable
- 504 Gateway Timeout

---

## Important Security Notes (For Pentesting)

### Method Misconfiguration

If dangerous methods are enabled:
```
PUT /uploads/shell.php
```
This may lead to file upload vulnerabilities.

### TRACE Enabled

If TRACE is enabled:
- Can be abused for Cross-Site Tracing (XST).

### Improper Status Codes

Examples:
- Returning 200 instead of 403 for unauthorized actions
- Leaking stack traces in 500 errors

### Missing Security Headers

Important headers to check:
- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options
- Strict-Transport-Security

---

## Summary

HTTP is:
- Stateless
- Message-based
- Built on TCP
- Request-response driven

Understanding HTTP deeply is essential for:
- Web development
- API design
- Web application penetration testing
