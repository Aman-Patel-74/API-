# Excessive Data Exposure

**Excessive Data Exposure** occurs when an API returns **more information than is necessary for the client application**. Instead of filtering sensitive data on the server side, the API sends complete objects and relies on the client to filter the information. This can expose sensitive information to unauthorized users.

In **API Penetration Testing**, this vulnerability is identified by analyzing API responses to determine whether the application discloses **sensitive or unnecessary data fields** such as internal identifiers, authentication tokens, debug information, or personal user details.

If exploited, excessive data exposure may lead to **information disclosure, privacy violations, and further attacks such as privilege escalation or account takeover**.

---

## Importance of Testing for Excessive Data Exposure

APIs often act as a bridge between **frontend applications and backend databases**, making them a critical component in modern architectures. Improper handling of response data can lead to unintended exposure of sensitive information.

Testing for excessive data exposure is important because:

- It may **leak confidential user information** such as personal details, account identifiers, email addresses, or authentication tokens.
- Attackers may use exposed data to **impersonate users, perform privilege escalation, or conduct social engineering attacks**.
- It can allow attackers to **enumerate users or harvest large volumes of sensitive data**, increasing the risk of large-scale data breaches.
- Debug or internal endpoints may reveal **system configuration details, internal APIs, or development artifacts**, which can assist attackers in further exploitation.

---

## Affected Endpoints Identified During Testing

During the security assessment, the following API endpoints were identified as potentially exposing **excessive or sensitive information**:

```
http://192.168.1.28:8888/community/api/v2/community/posts/recent
http://192.168.1.28:5001/users/v1
http://192.168.1.28:5001/users/v1/_debug
http://192.168.1.28:8888/identity/api/v2/user/videos
```

These endpoints were observed returning **more data than required by the client**, including potentially sensitive fields or internal debugging information.

---

## Potential Impact

If exploited, this vulnerability could allow attackers to:

- Access sensitive user information
- Enumerate user accounts
- Obtain internal identifiers and metadata
- Discover debugging information or internal system behavior
- Leverage the exposed data for **further attacks such as account takeover, privilege escalation, or targeted phishing**

---

## Recommendations

To mitigate Excessive Data Exposure vulnerabilities:

1. **Implement Response Filtering**
   - Ensure APIs only return **necessary fields required by the client**.
2. **Use Data Transfer Objects (DTOs)**
   - Avoid returning full database objects directly in API responses.
3. **Disable Debug Endpoints in Production**
   - Internal endpoints such as `/debug` should not be exposed in production environments.
4. **Apply Proper Access Controls**
   - Ensure that sensitive data is only accessible to **authorized users**.
5. **Perform API Response Validation**
   - Regularly review API responses to confirm that **sensitive fields are not exposed**.
6. **Follow the Principle of Least Privilege**
   - Only return data that is strictly required for the requested functionality.

---

## Supporting Evidence

### Service Report Enumeration (crAPI)

Manually incrementing/decrementing the `id` query parameter on the service-report endpoint returned service reports and full owner/mechanic PII belonging to **other users**, confirming excessive data exposure combined with a lack of ownership checks (see the related [BOLA finding](./Broken-Object-Level-Authorization-BOLA.md)):

```
http://192.168.1.5:8888/service-report?id=154   -> logged-in user's own report
http://192.168.1.5:8888/service-report?id=1     -> another user's report:
  Owner: adam007@example.com, 9876895423
  Vehicle VIN: 7ECOX34KJTV359804
  Mechanic: TRAC_JHN, jhon@example.com
  Problem details (free text) contained the owner's phone number and email
```

### Lack of Rate Limiting Enabling Enumeration (Burp Suite Intruder)

A Sniper attack was run against:
```
POST /workshop/api/merchant/contact_mechanic HTTP/1.1
Host: 192.168.1.5:8888
```
using 100 null payloads (no payload markers configured, base request repeated unmodified). All 100 requests returned:
```
HTTP/1.1 200 OK
```
with no throttling, CAPTCHA, or lockout observed — confirming the endpoint has **no rate limiting**, which allows an attacker to mass-enumerate `report_id` / `VIN` values and harvest excessive data at scale (as reflected in the crAPI dashboard showing many duplicate "Service Request" entries generated during testing, annotated "too many requests" with no blocking).
