# 🧪 PortSwigger Lab Writeup: OS Command Injection – Simple Case

## 📘 Lab Summary
This lab demonstrates a classic OS command injection vulnerability in a web application's stock checking feature. The server-side logic constructs a shell command using user-supplied input from the `productId` and `storeId` parameters, without proper sanitization. The objective is to exploit this flaw to execute the `whoami` command and retrieve the name of the current system user.

## 🔍 Technical Analysis

### 🔧 Vulnerable Endpoint
```
POST /product/stock HTTP/2
```

### 📥 Parameters
- `productId`
- `storeId`

The application embeds these parameters directly into a shell command, likely resembling:
```bash
./stockreport.sh [productId] [storeId]
```
This approach is inherently unsafe if the input is not properly sanitized, as it allows attackers to inject arbitrary shell commands.

## 🛠️ Exploitation Procedure

### 1. 🧭 Environment Setup
- **Browser:** Firefox
- **Proxy Tool:** Burp Suite (configured via FoxyProxy extension)

### 2. 🔗 Accessing the Lab
- Navigate to the lab URL provided by PortSwigger.
- Open any product page.

### 3. 📡 Capturing the Vulnerable Request
- Click the **"Check stock"** button.
- In Burp Suite, go to **Proxy → HTTP History**.
- Locate the POST request to `/product/stock`.
- Send the request to **Repeater** for manual testing.

### 4. 🧬 Identifying the Injection Point
The original request body appears as:
```
productId=1&storeId=1
```

### 5. 💣 Crafting the Injection Payload
To exploit the vulnerability, inject the `whoami` command into the `productId` parameter using shell syntax:
```
productId=whoami|1&storeId=1
```

This payload attempts to execute `whoami`, pipe its output to `1` (which is invalid but harmless), and continue processing `storeId=1`.

### 6. 🚀 Executing the Payload
- Send the modified request via Burp Repeater.
- Observe the server response.

## 📊 Response & Outcome

The server responds with:
```
/home/peter-FQSGlE/stockreport.sh: line 5: $2: unbound variable
sh: 1: 1: not found
```

Despite the error messages, the lab is marked as **solved**, confirming that the `whoami` command was successfully executed. The output was processed by the server, and the vulnerability was exploited.

## ⚠️ Security Impact

This vulnerability allows attackers to:
- Execute arbitrary OS-level commands on the server.
- Potentially access sensitive system information.
- Escalate privileges or pivot to other parts of the infrastructure.
- Compromise server integrity and availability.

## 🛡️ Remediation Strategies

To mitigate OS command injection risks:
- **Avoid shell invocation** with user input. Use safe APIs that do not rely on shell commands.
- **Sanitize and validate** all user-supplied data rigorously.
- **Use allowlists** for expected input values.
- **Employ parameterized functions** or command execution libraries that isolate input from execution logic.
- **Restrict privileges** of the executing process to minimize impact in case of exploitation.

## 📝 Conclusion

This lab highlights the dangers of unsanitized input in shell command execution. By injecting `whoami` into the `productId` parameter, we demonstrated successful exploitation of an OS command injection vulnerability. The exercise reinforces the importance of secure coding practices, especially when interfacing with the operating system.

## 📄 Disclaimer
This writeup is intended solely for educational and ethical penetration testing purposes. All testing was conducted in a controlled environment provided by PortSwigger Web Security Academy, which is designed for safe security research and learning. Unauthorized testing or exploitation of systems without explicit permission is illegal and unethical. Always obtain proper authorization before performing any security assessments.
