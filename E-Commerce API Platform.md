# E-Commerce API Platform — Security Assessment Notes

**Target:** `http://192.168.1.5:3000` (local test instance)
**API Version:** v2.3.1 (OAS 3.0), base path `/api/v1`
**Tooling:** Burp Suite Professional v2025.5.6
**Test date:** 2026-03-28
**Test accounts (from Swagger docs):**
- Admin: `admin` / `Admin123!`
- User: `alice` / `User123!`
- Self-registered: `u1` / `SecurePass123!`

> This is a lab/practice environment (RFC1918 address, seeded test credentials published in Swagger UI). Notes below are for personal reference / write-up purposes.

---

## Summary of Findings

| # | Finding | Endpoint | Severity |
|---|---------|----------|----------|
| 1 | Privilege escalation via mass assignment | `PUT /api/v1/auth/profile` | Critical |
| 2 | Broken Object-Level Authorization (BOLA) on order status | `PUT /api/v1/orders/{id}/status` | High |
| 3 | No server-side validation on product price | `PUT /api/v1/products/{id}` | Medium |
| 4 | Order total not recalculated server-side (possible price tampering) | `POST /api/v1/orders` | Medium |
| 5 | Minor data-integrity bugs (malformed timestamp, empty fields accepted) | Various | Low |

---

## 1. Privilege Escalation via Mass Assignment (Critical)

**Endpoint:** `PUT /api/v1/auth/profile`

The profile update endpoint accepts and persists an `isAdmin` field from the request body, even though it's not part of the documented editable profile fields.

**Steps to reproduce:**
1. Register a normal user:
   ```http
   POST /api/v1/auth/register
   { "username":"u1", "password":"SecurePass123!", "email":"aman@example.com", "fullName":"aman", "phone":"1234567890" }
   ```
   → `201 Created`, `userId: 2`

2. Log in and grab the access token:
   ```http
   POST /api/v1/auth/login
   { "username":"u1", "password":"SecurePass123!" }
   ```
   → Response includes `"role":"user"`, `"isAdmin":false`

3. Confirm current profile state:
   ```http
   GET /api/v1/auth/profile
   Authorization: Bearer <token>
   ```
   → `"isAdmin":0`

4. Send a profile update including the extra field:
   ```http
   PUT /api/v1/auth/profile
   { "id":2, "username":"u1", "email":"...", "fullName":"u1", "phone":"...", "role":"user", "isAdmin":1, ... }
   ```
   → `200 OK`, `"message":"Profile updated successfully"`

5. Re-check the profile:
   ```http
   GET /api/v1/auth/profile
   ```
   → `"isAdmin":1` — **confirmed**

6. Re-login — new JWT/session now reflects elevated privileges:
   ```http
   POST /api/v1/auth/login
   ```
   → Response: `"isAdmin":true`

**Impact:** Any authenticated user can grant themselves admin rights with a single PATCH/PUT request. Full account takeover of application privileges.

**Root cause:** Backend deserializes the full request body into the user model without a field allow-list (classic mass-assignment / over-posting vulnerability).

**Remediation:** Whitelist editable fields server-side (`fullName`, `phone`, `email`) and strip/ignore `role` and `isAdmin` from user-controlled input. Privilege changes should only be possible via a dedicated admin-only endpoint.

---

## 2. Broken Object-Level Authorization on Order Status (High)

**Endpoint:** `PUT /api/v1/orders/{id}/status`

Once authenticated (even as a low-privileged/elevated-via-#1 user), the order-status endpoint does not verify that the caller owns the order or has permission to modify it.

**Steps to reproduce:**
1. As the (now admin-flagged) `u1` account, list all orders:
   ```http
   GET /api/v1/orders/my-orders
   ```
   → Returns orders belonging to multiple different `userId`s (e.g. `userId: 2` orders alongside others), suggesting `my-orders` itself may not be scoping correctly, or admin flag is bypassing scoping.

2. Update the status of an order not necessarily created by the current session:
   ```http
   PUT /api/v1/orders/3/status
   { "status":"delivered", "paymentStatus":"paid" }
   ```
   → `200 OK`, `"message":"Order status updated successfully"`

3. Confirm the change persisted:
   ```http
   GET /api/v1/orders/my-orders
   ```
   → Order `id:3` now shows `"status":"delivered"`

**Additional validation observed:**
- Sending an invalid status value (`"completed"`) is rejected: `400 { "error":"Invalid status value" }` — so there **is** an enum check, just no ownership/authorization check.
- Omitting `status` entirely returns `400 { "error":"Status is required" }` — confirms `status` is required, but no auth check occurs before this validation.

**Impact:** Any authenticated user can alter the status/payment status of *any* order in the system (e.g., mark unpaid orders as `paid`, or mark another customer's order `delivered` prematurely).

**Remediation:** Verify `order.userId == requester.id` (or requester has an admin role validated server-side, not client-supplied) before allowing status mutation.

---

## 3. No Server-Side Validation on Product Price (Medium)

**Endpoint:** `PUT /api/v1/products/{id}`

**Steps to reproduce:**
1. Create a product (requires admin — obtained via Finding #1):
   ```http
   POST /api/v1/products
   { "sku":"234","name":"aman","description":"u1","price":100,"salePrice":90,"category":"10","stock":10 }
   ```
   → `201 Created`, `productId: 1`

2. Update the same product with `price` set to `0`:
   ```http
   PUT /api/v1/products/1
   { ..., "price":0, "salePrice":90, ... }
   ```
   → `200 OK`, `"message":"Product updated successfully"`

3. Confirm:
   ```http
   GET /api/v1/products/1
   ```
   → `"price":0` — accepted with no minimum/positive-value check.

**Impact:** Products can be set to $0 (or presumably negative values, untested), enabling free acquisition of goods if price is later trusted at checkout.

**Remediation:** Enforce `price > 0` (and `price >= salePrice` or equivalent business rule) server-side.

---

## 4. Order Total Not Recalculated Server-Side (Medium)

**Endpoint:** `POST /api/v1/orders`

**Observation:**
- Request submitted:
  ```json
  {
    "userId": 1,
    "items": [{ "productId": 1, "quantity": 2, "price": 500 }],
    "totalAmount": 1000,
    "paymentMethod": "online",
    "shippingAddress": "indore"
  }
  ```
- Response: `"totalAmount": 180` (does not match either the submitted `price × quantity` of 1000, nor the submitted `totalAmount` of 1000 — it instead reflects the product's actual `salePrice` at time of order, 90 × 2).

**Interpretation:** The server *does* appear to recompute `totalAmount` from the product's stored `salePrice` rather than trusting client-submitted `price`/`totalAmount` — this is actually the correct behavior and mitigates naive price tampering at order time. However, this should be explicitly confirmed with a few more targeted tests (e.g., submit a wildly different `productId`/`price` combination and diff the recalculated total) since the client-supplied `price` field being accepted in the request at all is a smell worth validating further.

**Follow-up test recommended:** Also validate `POST /orders` rejects orders with an empty `items` array (confirmed: it does — `400 { "error":"Order must contain at least one item" }`).

---

## 5. Minor Data Integrity Issues (Low)

- **Malformed timestamp:** One order record showed `"createdAt":"22026-03-28T17:47:50"` (extra leading `2` — likely a server-side date formatting/typo bug, not security-relevant but worth flagging to the dev team).
- **Empty shipping address accepted:** At least one order was created with `"shippingAddress":""` — no non-empty validation on required-looking fields.
- **Inconsistent `paymentStatus` handling:** Orders can be created/left in `unpaid` state indefinitely while `status` progresses (`pending` → `delivered`) without payment — a business-logic gap rather than a pure security bug, but worth noting given Finding #2 lets any user set `status` freely.

---

## Suggested Remediation Priority

1. **Fix mass assignment on `/auth/profile`** — highest impact, trivial to exploit.
2. **Add ownership/role checks on `/orders/{id}/status`** — depends on #1 not existing, but should be fixed independently (defense in depth).
3. **Add input validation on product price/order fields.**
4. Clean up minor data-integrity bugs.

---

## Appendix: Full Request/Response Flow (chronological)

1. `POST /auth/register` → 201, user `u1` created (id 2)
2. `POST /auth/login` → 200, JWT issued, `isAdmin:false`
3. `POST /auth/reset-password` → 200, password changed via reset token
4. `GET /auth/profile` → 200, confirms `isAdmin:0`
5. `PUT /auth/profile` with `isAdmin:1` injected → 200 (mass assignment)
6. `GET /auth/profile` → 200, confirms `isAdmin:1` now persisted
7. `POST /auth/login` (re-auth) → 200, `isAdmin:true` reflected in session
8. `POST /products` → 201, product created (id 1) using elevated privileges
9. `GET /products/1` → 200, baseline product state
10. `PUT /products/1` with `price:0` → 200 (no validation)
11. `GET /products/1` → 200, confirms `price:0`
12. `POST /orders` (empty items) → 400, validation works as expected
13. `POST /orders` (valid items) → 201, order created; total recalculated server-side
14. `GET /orders/my-orders` → 200, lists orders across the account
15. `PUT /orders/3/status` invalid value → 400 (enum validated)
16. `PUT /orders/3/status` valid value, no ownership check → 200 (BOLA)
17. `GET /orders/my-orders` → 200, confirms status change persisted
18. `PUT /orders/2/status` missing `status` field → 400 (required-field validation only)
