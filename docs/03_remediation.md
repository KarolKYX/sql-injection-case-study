# Phase 3: Defensive Remediation & Security Controls

## 1. Objective
Provide straightforward, industry-standard defensive measures to fix the vulnerabilities identified in Phase 1 and Phase 2, restoring security across all three pillars of the CIA Triad.

---

## 2. Parameterized Queries (Fix for SQL Injection)

* **Vulnerability Addressed:** SQL Injection in `/rest/products/search` and `/rest/user/login` (Confidentiality & Integrity).
* **The Problem:** The backend concatenates raw user strings directly into SQL statements, allowing attackers to manipulate query logic.
* **The Fix:** Use parameterized queries (prepared statements) or ORM built-in query helpers. The database treats user input strictly as data, never as executable code.

### Code Comparison (Node.js / Sequelize)

* **Vulnerable (Raw String Concatenation):**
  ```javascript
  // INSECURE: Input is directly added to SQL
  const query = `SELECT * FROM Products WHERE name LIKE '%${req.query.q}%'`;
  sequelize.query(query);
  ```

* **Remediated (Parameterized / ORM Methods):**
  ```javascript
  // SECURE: Sequelize automatically sanitizes and parameterizes the query
  const products = await Product.findAll({
    where: {
      name: {
        [Op.like]: `%${req.query.q}%`
      }
    }
  });
  ```

---

## 3. Rate Limiting & Server-Side Pagination (Fix for Resource Exhaustion)

* **Vulnerability Addressed:** Lack of rate controls and unconstrained database queries (Availability).
* **The Problem:** Clients can send unlimited requests and retrieve full database tables in one call, spiking CPU and memory usage.
* **The Fix:** Implement standard throttling middleware and enforce mandatory page limits.

### Implementation Examples

1. **API Rate Limiting (`express-rate-limit`):**
   ```javascript
   const rateLimit = require('express-rate-limit');

   const searchLimiter = rateLimit({
     windowMs: 15 * 60 * 1000, // 15-minute window
     max: 100,                 // Limit each IP to 100 requests per window
     message: { error: "Too many search requests, please try again later." }
   });

   app.use('/rest/products/search', searchLimiter);
   ```

2. **Mandatory Pagination (`LIMIT / OFFSET`):**
   ```javascript
   // Enforce a maximum of 20 items per request
   const page = parseInt(req.query.page) || 1;
   const limit = 20;
   const offset = (page - 1) * limit;

   const products = await Product.findAll({
     limit: limit,
     offset: offset
   });
   ```

---

## 4. Error Handling & Information Leakage Suppression

* **Vulnerability Addressed:** Stack trace and database error leakage (Reconnaissance).
* **The Problem:** The server returns raw database error messages (`SQLITE_ERROR`) and stack traces to clients, revealing internal file structures and database engines.
* **The Fix:** Implement a global error-handling middleware that logs technical details internally and returns a generic error message to users.

### Generic Error Handler (Express.js)

```javascript
// Generic production error middleware
app.use((err, req, res, next) => {
  // Log the detailed error internally on the server
  console.error(err.stack);

  // Send a sanitized, generic response to the user
  res.status(500).json({
    status: "error",
    message: "An internal error occurred. Please try again later."
  });
});
```

---

## 5. Summary of Controls & CIA Mapping

| Security Control | Implementation Method | CIA Triad Protected |
| :--- | :--- | :--- |
| **Parameterized Queries** | Sequelize ORM operators / Prepared Statements | **[C] Confidentiality & [I] Integrity** |
| **Rate Limiting** | `express-rate-limit` middleware | **[A] Availability** |
| **Server-Side Pagination** | Enforced `limit` & `offset` on queries | **[A] Availability** |
| **Sanitized Error Handling** | Global Express catch-all middleware | **[C] Confidentiality** |
