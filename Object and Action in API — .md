# Object and Action in API — Notes

**Path:** Application-Programming-Interface (API) / Object-and-Action-in-API

## Overview
- In APIs, **objects** and **actions** define how resources are structured and manipulated.
- **Object** → What the system manages (noun)
- **Action** → What you do to that object (verb)
- Understanding this distinction is critical when testing APIs for authorization and logic flaws.

## 1. Object in API
An **object** represents a resource/entity in the system. Objects are typically nouns and appear in the URL path.

### Common API Objects
`users`, `accounts`, `orders`, `products`, `invoices`, `payments`, `tickets`, `comments`, `files`, `projects`

### Example 1: User Object
```
GET /api/users/101
```
- Object → `users`
- Object ID → `101`

**Example Response:**
```json
{
  "id": 101,
  "username": "rahul",
  "email": "rahul@example.com",
  "role": "user"
}
```
This JSON represents a **User object**.

### Example 2: Order Object
```
GET /api/orders/9001
```
**Response:**
```json
{
  "order_id": 9001,
  "user_id": 101,
  "amount": 1500,
  "status": "shipped"
}
```
Here, the object is `order`.

### Example 3: Nested Object
```
GET /api/users/101/orders
```
- Parent Object → `users`
- Child Object → `orders`

This retrieves all orders belonging to user `101`.

## 2. Action in API
An **action** defines what operation is performed on an object. Actions are expressed using:
- HTTP methods (REST style)
- Action-based endpoints (RPC style)

### A) RESTful Actions (Using HTTP Methods)
| Method | Action | Example |
|---|---|---|
| GET | Read | `/users/101` |
| POST | Create | `/users` |
| PUT | Full update | `/users/101` |
| PATCH | Partial update | `/users/101` |
| DELETE | Remove | `/users/101` |

**Example 1: Create User**
```
POST /api/users
```
Body:
```json
{
  "username": "testuser",
  "email": "test@example.com"
}
```
- Object → `users`
- Action → `POST` (create)

**Example 2: Update Order**
```
PUT /api/orders/9001
```
- Object → `orders`
- Action → `update`

**Example 3: Delete Comment**
```
DELETE /api/comments/555
```
- Object → `comments`
- Action → `delete`

### B) Action-Based Endpoints (Non-REST Style)
Some APIs define actions explicitly in the URL.

**Example 1: Reset Password**
```
POST /api/users/007/reset-password
```
- Object → `users`
- Action → `reset-password`

**Example 2: Cancel Order**
```
POST /api/orders/9001/cancel
```
- Object → `orders`
- Action → `cancel`

**Example 3: Transfer Funds**
```
POST /api/accounts/2001/transfer
```
- Object → `accounts`
- Action → `transfer`

**Example 4: Approve Invoice**
```
POST /api/invoices/300/approve
```
- Object → `invoices`
- Action → `approve`

## 3. Complex Object + Action Examples

### Banking API Example
```
GET    /accounts/5001
POST   /accounts
POST   /accounts/5001/withdraw
POST   /accounts/5001/deposit
POST   /accounts/5001/close
```
- Objects: `accounts`
- Actions: `withdraw`, `deposit`, `close`

### SaaS Application Example
```
GET    /projects/10
POST   /projects
POST   /projects/10/archive
POST   /projects/10/add-member
DELETE /projects/10
```
- Objects: `projects`
- Actions: `archive`, `add-member`, `delete`

## 4. Object vs Action (Clear Comparison)
| Aspect | Object | Action |
|---|---|---|
| Type | Noun | Verb |
| Represents | Resource | Operation |
| Example | `user`, `order` | `create`, `delete`, `cancel` |
| Security Risk | BOLA | Broken Function Level Auth |

## 5. Security Perspective (Pentesting View)
Understanding object and action separation helps identify:

### 🔴 Object-Level Vulnerabilities
```
GET /api/users/102
```
If changing `007` → `102` gives access to another user's data:
→ Broken Object Level Authorization (BOLA)
→ Also known as IDOR

### 🔴 Action-Level Vulnerabilities
```
POST /api/users/102/delete
```
If a normal user can delete another user:
→ Broken Function Level Authorization

### 🔴 Mass Assignment via Object Fields
*(section started but not fully captured in these screenshots — revisit slide/doc for full detail)*

## Summary
- **Object** = What the API manages
- **Action** = What operation is performed
- Objects are nouns.
- Actions are verbs.
- Security testing requires validating both:
  - Access to the object
  - Permission to perform the action
