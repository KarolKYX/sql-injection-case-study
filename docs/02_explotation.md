# Phase 2: Vulnerability Exploitation & CIA Triad Impact

## 1. Objective
Demonstrate impact of identified vulnerabilities by mapping successful exploits against the CIA Triad: Confidentiality, Integrity, and Availability.

---

## 2. [C] Confidentiality: Data Exfiltration via UNION SQLi

* **Vulnerable Endpoint:** `GET /rest/products/search?q=`
* **Target Data:** User credentials table (`Users`)
* **Payload:** 
  ```sql
  apple')) UNION SELECT id, email, password, 'admin', 5, 6, 7, 8, 9 FROM Users--
  ```

### Technical Execution
The product search query directly concatenates user input. By matching the original query's column count (9 columns), an adversary can force the database to append user credentials to the public catalog search results.

![Confidentiality Breach PoC](../images/05_confidentiality_poc.png)

---

## 3. [I] Integrity: Authentication Bypass

* **Vulnerable Endpoint:** `POST /rest/user/login`
* **Impact:** Administrative Identity Takeover
* **Payload:**
  ```json
  {
    "email": "' OR 1=1--",
    "password": "x"
  }
  ```

### Technical Execution
The login authentication logic parses the raw string without parameter binding. The payload forces the condition `1=1` to evaluate to TRUE, while `--` truncates the password verification entirely, logging the attacker into the first database record (`admin@juice-sh.op`).

![Integrity Breach PoC](../images/06_integrity_poc.png)

---

## 4. [A] Availability: Service Disruption via Heavy Query / Lock

* **Vulnerable Endpoint:** `GET /rest/products/search?q=`
* **Impact:** Database lock / Denial of Service
* **Payload Concept:** Intensive recursive query or table lock payload causing elevated backend latency and resource exhaustion.

![Availability Disruption PoC](../images/07_availability_poc.png)
