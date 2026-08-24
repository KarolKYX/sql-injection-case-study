# OWASP Juice Shop Security Assessment & CIA Triad Impact

> **Disclaimer:** This repository contains Proof-of-Concept (PoC) materials developed strictly for educational purposes, security research, and vulnerability demonstration in an isolated lab environment. It is not intended for malicious use or unauthorized testing.

---

## 1. Project Overview
This project presents assessment conducted against the intentionally vulnerable **OWASP Juice Shop** web application deployed locally via Docker. The objective is to identify exposed attack vectors, enumerate backend technologies, and demonstrate impact of vulnerabilities by mapping them against the **CIA Triad (Confidentiality, Integrity, Availability)**.

---

## 2. Target Environment & Tooling
* **Target Application:** OWASP Juice Shop (Containerized via Docker)
* **Local Endpoint:** `http://localhost:3000`
* **Assessment Proxy:** Burp Suite Community Edition
* **Identified Stack:** Node.js, Express, SQLite, JSON Web Tokens (JWT)

```bash
# Launch the lab environment using Docker Compose
docker compose up -d
```

---

## 3. Executive Assessment Summary

| Triad Pillar | Target Endpoint | Attack Vector / Weakness | Impact / Demonstrated Result |
| :--- | :--- | :--- | :--- |
| **[C] Confidentiality** | `GET /rest/products/search?q=` | UNION-based SQL Injection (`sqlite_master` enumeration) | Full extraction of user table records, emails, and password hashes. |
| **[I] Integrity** | `POST /rest/user/login` | SQL Injection via Boolean Tautology (`' OR 1=1--`) | Complete authentication bypass and administrative account takeover. |
| **[A] Availability** | `GET /rest/products/search?q=` | Missing Rate Controls (CWE-400) & Unconstrained Queries | Node.js event-loop starvation and SQLite database file lock contention. |

---

## 4. Repository Structure

```text
├── docs/
│   ├── 01_reconnaissance.md   # Phase 1: Attack surface mapping & tech fingerprinting
│   ├── 02_exploitation.md     # Phase 2: Step-by-step CIA Triad exploitation & PoCs
│   └── 03_remediation.md      # Phase 3: Defensive recommendations & secure code fixes
├── images/                    # Burp Suite proof-of-concept captures and logs
├── docker-compose.yml         # Target lab orchestration configuration
└── README.md                  # Project overview and executive summary
```

---

## 5. Assessment Methodology

1. **Phase 1: Reconnaissance & Fingerprinting (`docs/01_reconnaissance.md`)**
   * Passive and active traffic interception via Burp Suite.
   * Enumeration of backend database errors revealing SQLite query structure.
   * Identification of runtime environment (Node.js/Express) and stateless JWT auth.

2. **Phase 2: Exploitation & Impact Demonstration (`docs/02_exploitation.md`)**
   * **Confidentiality:** Automated schema discovery via `sqlite_master` metadata and targeted extraction of credentials.
   * **Integrity:** Subversion of authentication queries to obtain administrator session tokens without credentials.
   * **Availability:** Architectural analysis of missing rate limits (`RateLimit-*`), lack of pagination, and single-threaded resource exhaustion risks.

3. **Phase 3: Defensive Remediation (`docs/03_remediation.md`)**
   * Implementation of parameterized queries (Prepared Statements / ORM binding).
   * Enforcement of API rate limiting (`express-rate-limit`) and mandatory server-side pagination (`LIMIT/OFFSET`).
   * Suppression of internal database error messages in production environments.
