# Phase 1: Reconnaissance & Attack Surface Mapping

## 1. Objective
Identify exposed endpoints, user-controlled input vectors, and underlying technologies powering the target application without performing destructive actions.

---

## 2. Technology Stack Fingerprinting
By inspecting HTTP response headers, error messages, and frontend assets, the following architecture was identified:

* **Frontend:** Angular SPA (Single Page Application)
* **Backend Framework:** Node.js / Express
* **Database Engine:** SQLite (identified via SQL dialect error patterns)
* **Authentication Mechanism:** JSON Web Tokens (JWT) / Bearer Auth

---

## 3. Attack Surface Identification (Endpoints & Parameters)
The following interactive endpoints were cataloged using Burp Suite Proxy and Site Map analysis:

| Endpoint | HTTP Method | Parameter(s) | Input Type | Potential Vulnerability Vector |
| :--- | :--- | :--- | :--- | :--- |
| `/rest/products/search` | `GET` | `q` | URL Query String | SQL Injection (Data Extraction) |
| `/rest/user/login` | `POST` | `email`, `password` | JSON Body | SQL Injection (Auth Bypass) |
| `/api/Feedbacks` | `POST` | `comment`, `rating` | JSON Body | Stored XSS / SQLi |
| `/rest/basket/:id` | `GET` | `id` (Path Parameter) | Numeric / URL | IDOR / Broken Access Control |

---

## 4. Prioritization for Deep-Dive Testing
Based on data flow analysis, two high-risk entry points were prioritized for vulnerability fuzzing:

1. **`POST /rest/user/login` (`email` field):** Directly handles identity verification and session creation.
2. **`GET /rest/products/search` (`q` field):** Reflects database records directly to the client interface, making it a prime candidate for UNION-based data extraction.
