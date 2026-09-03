# Vulnerable E-Commerce API Platform — Lab Setup Notes

**Path:** Application-Programming-Interface (API) / API-Penetration-Testing-Lab-Setup / Vulnerable-E-Commerce-API-Platform

> ⚠️ **Warning:** Contains intentional security vulnerabilities for educational/pentest purposes only. Do not deploy to production or use real data.

## Project Overview
A realistic, production-like e-commerce API with intentional vulnerabilities, built for security training and pen-testing practice. Simulates a mid-scale e-commerce platform with:

- User authentication and authorization (JWT)
- Product catalog management
- Shopping cart operations
- Order processing with payment simulation
- Review and rating system
- Admin panel functionality
- File upload capabilities
- Internal microservices simulation
- Webhook integrations
- Coupon / discount system

## Features

**Secure implementations (present):**
- Password hashing with bcrypt
- JWT-based authentication
- Input validation on selected endpoints
- Proper RBAC on some admin endpoints
- Parameterized SQL queries (in most places)

**Intentional vulnerabilities:**
- See `docs/VULNERABILITY_GUIDE.md` in the project for details

## Prerequisites
- Node.js 18+
- npm or yarn
- Docker (optional)

## Installation — Method 1: Local Setup

1. **Clone / unzip and move project**
   ```bash
   mv -v vulnerable-ecommerce-api /opt/
   cd /opt/vulnerable-ecommerce-api
   ```

2. **Install dependencies**
   ```bash
   npm install sqlite3@latest
   npm install
   ```

3. **Configure environment**
   ```bash
   cp .env.example .env
   # Edit .env if needed — defaults work fine for testing
   ```

4. **Seed the database**
   ```bash
   npm run seed
   ```

5. **Start the server**
   ```bash
   npm start
   ```

6. **Access the API**
   - API Base URL: `http://localhost:3000/api/v1`
   - API Documentation: `http://localhost:3000/api-docs`
   - Health Check: `http://localhost:3000/health`

*(Method 2: Docker Setup also available — not fully captured in these notes.)*

## Project Structure (from zip extraction)
```
vulnerable-ecommerce-api/
├── src/
│   ├── controllers/
│   │   ├── productsController.js
│   │   ├── servicesController.js
│   │   ├── cartController.js
│   │   ├── reviewsController.js
│   │   ├── debugController.js
│   │   ├── authController.js
│   │   └── ordersController.js
│   ├── models/
│   │   └── database.js
│   └── config/
│       ├── index.js
│       └── logger.js
├── docs/
│   ├── swagger.yaml
│   ├── SETUP_GUIDE.md
│   ├── VULNERABILITY_GUIDE.md
│   └── FILE_INDEX.md
├── scripts/
│   └── seed.js
├── .env.example
├── .gitignore
├── .dockerignore
├── docker-compose.yml
├── Dockerfile
├── package.json
├── PROJECT_SUMMARY.md
└── README.md
```

## API Documentation (Swagger / OpenAPI 3.0)
Accessed at: `http://<host>:3000/api-docs/`

- **Version:** 2.3.1 (OAS 3.0)
- **Base URL:** `/api/v1`
- **Auth:** Most endpoints require JWT Bearer token

**Test accounts:**
| Role  | Username | Password  |
|-------|----------|-----------|
| Admin | `admin`  | `Admin123!` |
| User  | `alice`  | `User123!`  |

**Key endpoint groups seen:**
- **Authentication**
  - `POST /auth/register` — Register new user
  - `POST /auth/login` — Login user
  - `POST /auth/refresh` — Refresh access token
  - `GET /auth/profile` 🔒 — Get current user profile
  - `PUT /auth/profile` 🔒 — Update user profile
- **Products**
  - `GET /products` — Get all products
  - `POST /products` 🔒 — Create product (Admin only)

## Notes / To-Do
- Review `VULNERABILITY_GUIDE.md` for the full list of intentional flaws before testing.
- `debugController.js` stands out as worth investigating first — debug endpoints are common vuln injection points.
- Confirm which endpoints lack RBAC/input validation per the "Secure Implementations" caveats ("on **selected** endpoints", "on **some** admin endpoints", "in **most** places").
