# Same-Origin Policy (SOP)

## Definition

A browser-enforced security mechanism that restricts how documents or scripts loaded from one origin can interact with resources from another origin.

---

## What Is an Origin?

An origin is defined by the combination of:

```
Protocol + Domain + Port
```

**Examples:**
- `https://example.com` ≠ `http://example.com` (different protocol)
- `https://example.com` ≠ `https://api.example.com` (different subdomain)
- `https://example.com:443` ≠ `https://example.com:8080` (different port)

---

## What SOP Allows

- Read data from the same origin
- Modify DOM of the same origin
- Access cookies and localStorage of the same origin

## What SOP Blocks

- Reading responses from a different origin
- Accessing DOM of another origin
- Retrieving cookies or storage data from another origin

---

## Exceptions / Allowed Cross-Origin Actions

Even under SOP, certain cross-origin actions are still permitted by the browser — this is where a lot of real-world vulnerabilities (XSS, CSRF, clickjacking) live.

### 1. Loading External Resources

Allowed via:
```html
<script src="https://evil.com/script.js"></script>
<img src="https://target.com/image.png">
<link href="https://cdn.com/style.css">
```
- JavaScript executes, but response content is not directly readable.

### 2. Form Submission (Cross-Origin)

```html
<form action="https://target.com" method="POST">
```
- Requests can be sent to another origin
- Response cannot be read

> This is the foundation of CSRF — the browser will happily send the request, it just won't let the attacker read the response.

### 3. CORS (Cross-Origin Resource Sharing)

Servers can relax SOP using headers:
```
Access-Control-Allow-Origin
```
- Misconfiguration may lead to security issues (e.g. reflecting `Origin` back with `Allow-Credentials: true`).

### 4. Cross-Origin Embedding

Allowed elements:
- `<iframe>`
- `<img>`
- `<video>`

**Restriction:** JavaScript cannot access iframe content if the origin differs.

---

## Pentester Notes

- SOP prevents unauthorized data access between origins.
- Common attack vectors targeting SOP:
  - Cross-Site Scripting (XSS)
  - CORS misconfiguration
  - Cross-Site Request Forgery (CSRF)
  - Clickjacking

---

## Summary

SOP prevents a malicious website from reading sensitive data from another website within a user's browser.
