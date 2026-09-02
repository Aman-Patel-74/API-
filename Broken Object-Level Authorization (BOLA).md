# Broken Object-Level Authorization (BOLA) — API Security Notes

**Target:** crAPI (`192.168.1.5:8888` / `192.168.1.28:8888`)
**Category:** Application Programming Interface (API) → Broken Object-Level Authorization (BOLA)

---

## Vulnerability Description

Broken Object-Level Authorization (BOLA) occurs when an application fails to properly enforce authorization checks on objects referenced by user-supplied identifiers. This allows attackers to manipulate object identifiers (such as `vehicle_id`, `order_id`, or `report_id`) to access or modify resources belonging to other users.

Several API endpoints expose object identifiers in URLs or parameters but **do not validate whether the authenticated user owns the requested resource**. As a result, an authenticated user can modify identifiers and access **sensitive data belonging to other users**.

---

## Affected Endpoints

### 1. Vehicle Location API

**Endpoint:**
```
GET /identity/api/v2/vehicle/{vehicle_id}/location
```

**Example request:**
```http
GET /identity/api/v2/vehicle/4bae9968-ec7f-4de3-a3a0-ba1b2ab5e5e5/location HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <user_token>
```

**Steps to reproduce:**
1. Login as **User A**.
2. Navigate to the dashboard: `http://192.168.1.28:8888/dashboard`
3. Capture the request for the vehicle location:
   `/identity/api/v2/vehicle/<vehicle_id>/location`
4. Replace the `vehicle_id` with another valid UUID belonging to **User B**:
   `/identity/api/v2/vehicle/fa8cca43-d125-4a0c-8872-8501bcb30145/location`
5. Send the request.

**Result:** The server returns the **location data of another user's vehicle**.

**Expected result:** The application should return `403 Forbidden` or `Access Denied` if the vehicle does not belong to the authenticated user.

---

### 2. Orders API

**Vulnerable endpoint:**
```
GET /workshop/api/shop/orders/{order_id}
```

Example:
```
GET /workshop/api/shop/orders/1
GET /workshop/api/shop/orders/9
```

**Steps to reproduce:**
1. Login as **User A**.
2. Capture the request: `GET /workshop/api/shop/orders/1`
3. Modify the order ID: `GET /workshop/api/shop/orders/9`
4. Send the request.

**Result:** The application returns **order details belonging to another user**.

---

### 3. Mechanic Reports

**Vulnerable endpoint:**
```
GET /workshop/api/mechanic/mechanic_report?report_id={id}
```

**Confirmed via Burp Suite Repeater** (report_id enumeration):
- `report_id=7` → returned mechanic report for vehicle VIN `562UT57U39MCTD86R`, owner `aman@gmail.com`
- `report_id=1` → returned mechanic report for vehicle VIN `7EC0X34KJTV359804`, owner `adam007@example.com`, including owner's phone number and full problem-details text

Both requests succeeded (`HTTP/1.1 200 OK`) despite the reports belonging to different, unrelated users — confirming no ownership check is performed on `report_id`.

**Related UI confirmation:** `crAPI` service-report page (`/service-report?id=6`) shows report details (mechanic code, mechanic email, vehicle VIN, owner email/phone) for a report not necessarily created by the logged-in user (`aman`).

---

## Additional Testing Notes (Burp Suite)

- **Password reset OTP brute-force observed in Intruder run:** payload set `0000`–`0020` against endpoint tied to `/forgot-password`, targeting body `{"email":"aman@gmail.com","otp":"<PAYLOAD>","password":"Azlan@1234"}`.
  - Payload `0083` returned **HTTP 200** while all others (`0000`–`0020` etc.) returned **HTTP 500**, indicating the correct OTP was identified via brute force — a related **Broken User Authentication** issue compounding the BOLA impact (account takeover path).

---

## Security Impact

Successful exploitation allows an attacker to access sensitive information including:
- Vehicle location data
- Other users' orders
- Mechanic diagnostic reports
- Potential personally identifiable information (PII)

This vulnerability could enable:
- Privacy violations
- Vehicle tracking
- Data leakage
- Business logic abuse

---

## Risk Severity

**High**

**CVSS v3.1 Example:**
```
CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:L/A:N
Score: 7.5 (High)
```

---

## Root Cause

The backend API does not verify resource ownership when processing requests.

Instead of checking:
```
Does this user own this object?
```

The system only checks:
```
Is the user authenticated?
```

---

## Recommended Remediation

### 1. Enforce Object-Level Authorization
Verify ownership of every object before returning data.

Example:
```python
if order.user_id != current_user.id:
    return 403
```

### 2. Implement Access Control Middleware
Centralize authorization checks for all sensitive resources.

### 3. Use Indirect Object References
Avoid exposing predictable IDs. Use secure references or enforce strict authorization checks.

### 4. Apply Authorization at the Database Layer
Example:
```sql
SELECT * FROM orders
WHERE order_id = ?
AND user_id = current_user_id;
```

### 5. Log and Monitor Access
Log abnormal access patterns such as:
- Multiple ID enumeration attempts
- Requests for objects belonging to other users

---

## Reference

OWASP crAPI official documentation (`docs/challenges.md`) confirms these as intended training challenges:

**Challenge 1 – Access details of another user's vehicle**
- Vehicle IDs are GUIDs, not sequential — requires finding a way to expose another user's vehicle ID.
- Find an API endpoint that receives a vehicle ID and returns information about it.

**Challenge 2 – Access mechanic reports of other users**
- crAPI lets vehicle owners contact mechanics via a "contact mechanic" form.
- Analyze the report submission process.
- Find a hidden API endpoint that exposes mechanic report details.
- Change the report ID to access other users' reports.
