# API Testing Notes — Recon (Discovering Attack Surface)

## 1. Why Recon Matters
Before testing an API for vulnerabilities, you need to map out as much
information about it as possible. This process reveals the API's
**attack surface** — every point where it accepts input or can be
interacted with.

## 2. Step 1: Identify API Endpoints

An **endpoint** is a location where the API receives requests about a
specific resource.

Example:
```
GET /api/books HTTP/1.1
Host: example.com
```
- Endpoint: `/api/books`
- Purpose: retrieves a list of books from the library

Endpoints can be more specific, e.g.:
```
GET /api/books/mystery
```
- Retrieves only books in the "mystery" category.

**Takeaway:** Each distinct path represents a resource or sub-resource
the API exposes. Mapping these paths is the first step in building a
picture of what the API can do.

## 3. Step 2: Determine How to Interact With Each Endpoint

Once endpoints are known, figure out *how* to construct valid requests
against them. Key things to identify:

### a) Input Data
- Compulsory (required) parameters
- Optional parameters
- Expected data types/formats for each parameter

### b) Request Types
- Which **HTTP methods** are supported per endpoint
  (e.g., `GET`, `POST`, `PUT`, `DELETE`, `PATCH`)
- Which **media formats** are accepted
  (e.g., `application/json`, `application/xml`, `multipart/form-data`)

### c) Rate Limits & Authentication
- Are there request rate limits? (e.g., X requests per minute)
- What authentication mechanism is used?
  - API keys
  - Bearer tokens / JWT
  - OAuth
  - Basic auth
  - Session cookies

## 4. Quick Recon Checklist
- [ ] List all discovered endpoints
- [ ] Note HTTP methods supported per endpoint
- [ ] Note required vs optional parameters
- [ ] Note accepted content types
- [ ] Identify authentication mechanism(s)
- [ ] Identify rate-limiting behavior
- [ ] Note any versioning in the API (e.g., `/v1/`, `/v2/`)
- [ ] Check for documentation (Swagger/OpenAPI, Postman collections)

## 5. Common Sources for Recon Info
- API documentation (official docs, Swagger/OpenAPI specs)
- JavaScript files (client-side code often reveals endpoints)
- Browser dev tools / proxy tools (e.g., Burp Suite) while using the app
- Error messages (can leak endpoint structure or tech stack)
- robots.txt / sitemap.xml
- Public GitHub repos, API directories (e.g., Postman public workspace)
