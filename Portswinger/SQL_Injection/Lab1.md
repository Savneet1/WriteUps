# 🛡️ SQL Injection in WHERE Clause Allowing Retrieval of Hidden Data

## 1. 📋 Lab Overview
**Title:** SQL injection vulnerability in WHERE clause allowing retrieval of hidden data  
**Objective:** Exploit a SQL injection vulnerability in the product category filter to retrieve unreleased products from the database.  
**Vulnerability Type:** In-band SQL Injection (Boolean-based, tautology injection)  
**Impact:** 🚨 Unauthorized disclosure of hidden product data.

## 2. 🖥️ Application Behavior
When a user selects a product category, the application executes a SQL query similar to:

```sql
SELECT * FROM products 
WHERE category = 'Gifts' AND released = 1;
```

- The `category` parameter is taken directly from the URL query string and inserted into the SQL statement without proper sanitization.
- The `released = 1` condition ensures only publicly available products are shown.

## 3. 🛠️ Tools Used
- **Burp Suite** (Proxy & Repeater modules) for intercepting and modifying HTTP requests.

## 4. 🚀 Exploitation Steps

### Step 1: 🔍 Intercept the Request
1. Enable Burp Suite Proxy and configure the browser to route traffic through it.
2. Launch the lab and select any category (e.g., *Lifestyle*).
3. In Burp Suite → **Proxy → HTTP history**, locate the captured request:
   ```http
   GET /filter?category=Lifestyle HTTP/1.1
   ```

### Step 2: 📤 Send to Repeater
- Right-click the request → **Send to Repeater** for controlled testing.

### Step 3: 🧪 Craft the Injection Payload
- The goal is to bypass the `released = 1` condition by injecting a tautology (`OR 1=1`).
- Modify the request to:
   ```http
   GET /filter?category=Lifestyle' OR 1=1-- HTTP/2
   ```
**Explanation:**
- `'` closes the original string literal.
- `OR 1=1` always evaluates to `TRUE`.
- `--` comments out the remainder of the SQL query, removing the `AND released = 1` restriction.

### Step 4: 📡 Send the Exploit
- Send the modified request from Repeater.
- The server responds with a product list that now includes unreleased items.

## 5. ✅ Result
- **Lab Status:** ✔️ Solved  
- **Observed Behavior:** The product listing now contains items that were previously hidden, confirming successful exploitation.

## 6. 🧐 Root Cause Analysis
- **Cause:** Lack of input validation and parameterized queries.
- **Risk:** Attackers can manipulate SQL queries to access unauthorized data, potentially leading to full database compromise.

## 7. 🔐 Remediation Recommendations
- Implement **prepared statements** or **parameterized queries**.
- Apply **server-side input validation** and escaping.
- Use **least privilege** for database accounts to limit damage from SQL injection.

## 8. 💡 Key Takeaways
- Even simple tautology-based injections can bypass business logic restrictions.
- Comment sequences (`--`) are effective for truncating unwanted query parts.
- Always test both the functional and security implications of user-controlled parameters.

## ⚠️ Disclaimer
This write-up is provided **for educational purposes only**. The techniques described here must **only** be used in authorized environments such as PortSwigger’s labs or systems you have explicit permission to test.  
Unauthorized testing or exploitation of systems is **illegal** and may result in severe legal consequences.
