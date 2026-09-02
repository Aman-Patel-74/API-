# API Penetration Testing – Lab Setup

Below are intentionally vulnerable API applications designed for hands-on security testing practice. These labs cover REST, GraphQL, JWT, OAuth, IDOR, SSRF, BOLA, and more.

---

## 1. 🟢 OWASP – crAPI

🔗 [https://github.com/OWASP/crAPI](https://github.com/OWASP/crAPI)

**crAPI (Completely Ridiculous API)** is one of the most comprehensive modern API security labs.

### Focus Areas
- OWASP API Security Top 10
- BOLA (Broken Object Level Authorization)
- Broken Authentication
- Mass Assignment
- JWT issues
- SSRF
- Rate limiting flaws

### 🛠️ Setup
```bash
git clone https://github.com/OWASP/crAPI.git
cd crAPI
docker-compose up
```
Runs via Docker. Web UI + API backend included.

---

## 2. 🟢 vapi

🔗 [https://github.com/roottusk/vapi](https://github.com/roottusk/vapi)

A lightweight vulnerable REST API for beginners.

### Focus Areas
- SQL Injection
- Authentication bypass
- IDOR
- Improper input validation

### Setup
```bash
git clone https://github.com/roottusk/vapi.git
cd vapi
docker-compose up
```

---

## 3. 🟢 VAmPI (Vulnerable API)

🔗 [https://github.com/erev0s/VAmPI](https://github.com/erev0s/VAmPI)

A REST API built in Flask with deliberate vulnerabilities.

### Focus Areas
- JWT weaknesses
- BOLA
- Mass assignment
- Improper rate limiting
- Broken access control

### 🛠️ Setup
```bash
git clone https://github.com/erev0s/VAmPI.git
cd VAmPI
docker-compose up
```

---

## 4. 🟢 Damn Vulnerable Bank

🔗 [https://github.com/rewanthtammana/Damn-Vulnerable-Bank/](https://github.com/rewanthtammana/Damn-Vulnerable-Bank/)
Backend: [BackendServer](https://github.com/rewanthtammana/Damn-Vulnerable-Bank/tree/master/BackendServer)

Full banking simulation with frontend + backend.

### 🎯 Focus Areas
- Business logic flaws
- Transaction manipulation
- IDOR
- Authentication bypass
- Privilege escalation

### 🛠️ Setup

**Backend:**
```bash
cd BackendServer
npm install
npm start
```

**Frontend:**
```bash
cd Frontend
npm install
npm start
```

---

## 5. 🟢 Damn Vulnerable GraphQL Application

🔗 [https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application)

A GraphQL-based intentionally vulnerable lab.

### Focus Areas
- GraphQL introspection abuse
- Authorization bypass
- Query batching attacks
- IDOR in GraphQL
- Injection attacks

### 🛠️ Setup
```bash
docker run -p 5013:5013 dolevf/dvga
```

---

## 🧪 Recommended Lab Environment

Since you're working in a penetration testing setup (Kali, Burp, recon workflow), the following is recommended:

### 💻 Environment
- Kali Linux (Attacker)
- Docker installed
- Burp Suite
- Postman / Insomnia
- ffuf / wfuzz
- sqlmap
- jwt_tool
- mitmproxy (optional)

### Network Setup
- Use **Host-only** or **NAT** networking in VirtualBox/VMware
- Keep labs isolated from production networks

---

## 🎯 Suggested Learning Path (Progressive Difficulty)

1. VAmPI → Basic REST vulnerabilities
2. vapi → Injection practice
3. crAPI → Advanced OWASP API Top 10
4. Damn Vulnerable Bank → Business Logic
5. DVGA → GraphQL-specific attacks

---

## Possible Next Steps

- Map these labs to **OWASP API Security Top 10 (2023)**
- Create a structured **30-day API pentesting roadmap**
- Provide a **Burp Suite testing checklist for APIs**
- Create Docker compose automation script for all labs
- Build reporting template for API findings
