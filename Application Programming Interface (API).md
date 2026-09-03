# Application Programming Interface (API) — Notes

**Path:** Application-Programming-Interface (API) / Application-Programming-Interface (API)

## What is an API?
- A structured interface that allows different software systems to communicate with each other through predefined rules and protocols.
- APIs abstract internal logic and expose only necessary functionality to external systems.

> Think of an API like a restaurant menu — it lists what you can request without exposing how the kitchen prepares it.

### Real-World Analogy
| Concept | Role |
|---|---|
| Client | You placing an order |
| API | The menu (interface) |
| Server | The kitchen (processes request and returns result) |

## Core Components of an API

### 1. Endpoint
An endpoint is a specific URL representing a resource or action.

```
GET https://api.example.com/users/123
```
- `/users` → Resource
- `/123` → Specific object

### 2. HTTP Methods
| Method | Purpose | Example |
|---|---|---|
| GET | Retrieve data | `/products` |
| POST | Create new resource | `/users` |
| PUT | Update resource | `/users/123` |
| PATCH | Partial update | `/users/123` |
| DELETE | Remove resource | `/users/123` |

### 3. Request Structure
A client request may contain:
- Headers (Authorization, Content-Type)
- Query Parameters
- Request Body
- Cookies

**Example:**
```bash
curl -X POST https://api.example.com/login \
     -H "Content-Type: application/json" \
     -d '{"username":"admin","password":"1234"}'
```

### 4. Response Structure
A server response includes:
- Status Code
- Headers
- Body (JSON/XML/etc.)

**Example:**
```json
{
  "id": 123,
  "name": "John Doe",
  "email": "john@example.com"
}
```

### 5. HTTP Status Codes
Grouped into five classes (per MDN / RFC 9110):

1. Informational responses (100–199)
2. Successful responses (200–299)
3. Redirection messages (300–399)
4. Client error responses (400–499)
5. Server error responses (500–599)

| Code | Meaning |
|---|---|
| 200 | Success |
| 201 | Created |
| 204 | No Content |
| 400 | Bad Request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | Not Found |
| 429 | Too Many Requests |
| 500 | Internal Server Error |

Reference: [MDN — HTTP response status codes](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status)

## Common API Use Cases

### 1. Web Applications
- Fetch external services (Maps, Payments, Authentication)
- SPA frameworks using REST APIs
- AJAX-based dynamic content

## Client / Server Architecture (Diagram Notes)

**HTTP Client / User-Agent types** — all connect out to a central API HTTP Server:
- Web
- Android App
- iOS App
- Thick Client App
- Other App Servers

```
Web ─────────┐
Android App ─┤
iOS App ─────┼──►  API / HTTP Server
Thick Client ─┤
Other Servers ┘
```

Each client type sends a request to the API HTTP Server and receives a response back (bidirectional arrows in original diagram) — same underlying request/response model regardless of client platform (web, Android, iOS, thick client).
