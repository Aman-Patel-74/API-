# Types of APIs

APIs can be classified in multiple ways depending on:

- **Access method** (architecture / communication style)
- **Exposure level** (who can access them)
- **Usage context** (where they are used)
- **Communication pattern** (synchronous vs asynchronous)

---

## 1. Types of APIs Based on Access Method (Architecture Style)

### 1.1 REST API (Representational State Transfer)

REST is an architectural style that uses HTTP methods to operate on resources.

**Core Principles**
- Stateless communication
- Resource-based URLs
- Standard HTTP methods
- Uniform interface
- Cacheable responses

**Example**
```
GET    /api/users/101
POST   /api/users
PUT    /api/users/101
DELETE /api/users/101
```

**Data Format**
- Usually JSON
- Can also use XML, HTML, or others

**Advantages**
- Lightweight
- Easy to implement
- Highly scalable
- Works naturally with web infrastructure

**Common Use Cases**
- Web applications
- Mobile backends
- Microservices
- Public APIs

---

### 1.2 SOAP API (Simple Object Access Protocol)

SOAP is a protocol that defines strict rules for message structure and communication.

**Core Characteristics**
- XML-based messaging
- Uses WSDL (Web Services Description Language)
- Supports multiple transport protocols (HTTP, SMTP, TCP)
- Built-in security extensions (WS-Security)
- Supports ACID transactions

**Example SOAP Request**
```xml
<soap:Envelope>
  <soap:Body>
    <GetUser>
      <UserId>101</UserId>
    </GetUser>
  </soap:Body>
</soap:Envelope>
```

**Advantages**
- Strong standardization
- Enterprise-level security
- Reliable message delivery

**Common Use Cases**
- Banking systems
- Enterprise systems
- Government services
- Legacy integrations

---

### 1.3 GraphQL API

GraphQL is a query language for APIs that allows clients to request exactly the data they need.

**Core Characteristics**
- Single endpoint (e.g., `/graphql`)
- Client defines response structure
- Strongly typed schema
- Supports queries, mutations, and subscriptions

**Example Query**
```graphql
query {
  user(id: 101) {
    name
    email
  }
}
```

**Advantages**
- Prevents over-fetching
- Prevents under-fetching
- Efficient data retrieval
- Flexible for frontend development

**Common Use Cases**
- Complex frontend applications
- Mobile apps
- Applications with multiple related data models

---

### 1.4 gRPC API

gRPC is a high-performance RPC framework developed by Google.

**Core Characteristics**
- Uses HTTP/2
- Binary serialization via Protocol Buffers
- Strong typing
- Supports streaming

**Advantages**
- High performance
- Low latency
- Efficient for microservices

**Common Use Cases**
- Internal microservice communication
- High-throughput systems

---

## 2. Types of APIs Based on Exposure Level

### 2.1 Public APIs (Open APIs)
- Available to external developers
- Often require API keys or OAuth

**Example:** Payment gateways, Social media APIs

### 2.2 Private APIs (Internal APIs)
- Used within an organization
- Not exposed to external users

**Common in:** Microservices architecture, Internal dashboards

### 2.3 Partner APIs
- Shared with specific business partners
- Controlled access via contracts and agreements

**Example:** Logistics integration, Banking integrations

---

## 3. Types of APIs Based on Usage Context

### 3.1 Web APIs (HTTP APIs)
- Communicate over HTTP
- Used in web and mobile apps
- Most common modern API type

### 3.2 Operating System APIs
Allow applications to interact with OS-level services.

**Examples:** File system access, Memory management, Process control

**Examples include:** Windows API, POSIX system calls

### 3.3 Library / Framework APIs
Used within programming environments to provide reusable functionality.

**Examples:** Database connectors, Authentication libraries, Logging frameworks

**Example in Python**
```python
import os
os.listdir()
```

---

## Benefits of Using APIs

- **Modularity** – Systems can be divided into independent services.
- **Interoperability** – Different technologies and platforms can communicate.
- **Scalability** – Backend services can scale independently.
- **Security Control** – APIs support:
  - Authentication tokens
  - Role-based access control
  - Rate limiting
  - Logging and monitoring
- **Faster Development** – Developers can integrate third-party services rather than building from scratch.

---

## Differences Between SOAP and REST

| Feature              | SOAP                  | REST                  |
|-----------------------|-----------------------|------------------------|
| Type                  | Protocol              | Architectural style    |
| Data Format           | XML only              | JSON, XML, etc.        |
| Transport             | HTTP, SMTP, TCP       | HTTP                   |
| State                 | Can be stateful       | Stateless              |
| Caching               | Not built-in          | Supported via HTTP     |
| Performance           | Heavier               | Lightweight            |
| Complexity            | High                  | Moderate               |
| Standardization       | Strict WS-* standards | No strict official standard |
| Description Language  | WSDL                  | OpenAPI (optional)     |
| Resource Exposure     | Service-based         | Resource-based URLs    |

---

## REST vs SOAP vs GraphQL vs gRPC ⭐ *(very important)*

| Feature            | REST               | SOAP        | GraphQL        | gRPC             |
|---------------------|---------------------|-------------|-----------------|-------------------|
| Style/Type          | Architectural style | Protocol    | Query language  | RPC framework     |
| Format              | JSON (mostly)       | XML         | JSON            | Protocol Buffers  |
| Performance         | High                 | Moderate    | High            | Very High         |
| Endpoint Structure  | Multiple             | Multiple    | Single          | Multiple          |
| Flexibility         | Medium               | Low         | High            | Medium            |
| Learning Curve      | Easy                 | Complex     | Moderate        | Moderate          |

---

## Summary

**REST**
- Most widely used
- Lightweight and scalable
- Best for web and mobile apps

**SOAP**
- Strict standards
- Enterprise-grade security
- Used in legacy systems

**GraphQL**
- Flexible data querying
- Client-controlled responses
- Efficient for frontend-heavy applications

**gRPC**
- High performance
- Binary protocol
- Ideal for microservices communication
