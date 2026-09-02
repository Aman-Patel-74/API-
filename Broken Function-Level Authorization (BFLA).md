# Broken Function-Level Authorization (BFLA) — API Security Notes

**Target:** crAPI (`192.168.1.28:8888`)
**Category:** Application Programming Interface (API) → Mass Assignment → Broken Function-Level Authorization (BFLA)

---

## Vulnerability Description

**Broken Function Level Authorization (BFLA)** occurs when an API fails to properly enforce authorization checks for specific functions or endpoints. This allows users with lower privileges to access functionality that should be restricted to higher-privileged roles (such as administrators).

If authorization checks are missing or improperly implemented, attackers may invoke privileged operations directly by interacting with backend API endpoints.

This vulnerability is listed in the **OWASP API Security Top 10** as a critical API security risk.

---

## How BFLA Occurs

### 1. Lack of Proper Role Verification
The API does not validate whether the authenticated user has the required role or permission before executing sensitive functions.

### 2. Overly Permissive Endpoints
Endpoints intended for administrators or privileged users are accessible to normal authenticated users.

### 3. Client-Side Authorization Enforcement
The application relies on the frontend to restrict functionality instead of enforcing authorization on the server.

### 4. Exposed Sensitive Endpoints
Administrative endpoints are exposed and accessible if the attacker manually sends requests to them.

---

## Vulnerable Endpoint

```
DELETE /identity/api/v2/admin/videos/52
```

**URL:**
```
http://192.168.1.28:8888/identity/api/v2/admin/videos/52
```

This endpoint appears to be an **administrative API function** used to delete videos from the system.

---

## Proof of Concept (PoC)

The following request was sent using a **regular user JWT token** (role = `user`):

```http
DELETE /identity/api/v2/admin/videos/52 HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <USER_JWT_TOKEN>
Content-Type: application/json
```

**Decoded JWT payload:**
```json
{
  "sub": "ankit+1@gmail.com",
  "role": "user",
  "iat": 1732182096,
  "exp": 1732786896
}
```

Despite the user having the role:
```
role: user
```

the API allowed access to an endpoint intended for **administrative functionality**.

---

## Vulnerability Description (Root Cause)

The endpoint:
```
/identity/api/v2/admin/videos/{id}
```

is part of the **admin API namespace**, indicating that it should only be accessible to users with administrative privileges.

However, the server **does not enforce role-based authorization checks**, allowing a normal user to perform privileged operations.

This represents a **Broken Function Level Authorization (BFLA)** vulnerability.

---

## Impact

An attacker could exploit this vulnerability to:
- Delete videos from the platform
- Perform administrative actions
- Modify or remove sensitive data
- Disrupt platform functionality
- Abuse privileged API operations

If multiple admin endpoints exist, attackers could gain **full administrative capabilities**.

---

## Severity

**High**

**Reasons:**
- Privilege escalation
- Unauthorized administrative actions
- Potential data loss
- Direct API access without restrictions

---

## Indicators of BFLA

Common indicators include:

- API endpoints containing paths like:
```
/admin/
/management/
/internal/
/superuser/
```
- Administrative functions accessible using normal user tokens.
- Missing server-side authorization checks.

---

## Recommended Remediation

### 1. Implement Role-Based Access Control (RBAC)
The API must verify user roles before executing privileged actions.

Example logic:
```python
if user.role != "admin":
    return 403 Forbidden
```

### 2. Enforce Authorization on the Server
Never rely on frontend restrictions. All authorization checks must occur on the **backend API layer**.

### 3. Restrict Access to Admin Endpoints
Endpoints under paths like:
```
/admin/*
```
should only be accessible to **admin roles**.

### 4. Implement Centralized Authorization Middleware
Use middleware to validate permissions for every request before reaching business logic.

### 5. Log and Monitor Privileged Actions
Administrative actions such as deletion should be logged and monitored to detect abuse.

---

## Secure Response Example

If a normal user attempts this request, the API should respond:

```http
HTTP/1.1 403 Forbidden
{
  "error": "Unauthorized access"
}
```

---

## Conclusion

The API fails to enforce proper **function-level authorization checks**, allowing users with insufficient privileges to access administrative functionality. This vulnerability could lead to unauthorized administrative actions, data deletion, and potential system compromise.
