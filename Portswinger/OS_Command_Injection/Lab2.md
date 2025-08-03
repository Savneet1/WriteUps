## 🧪 **Lab Title:** PortSwigger – Blind OS Command Injection (Time Delay Exploitation)

### 🎯 Objective  
Demonstrate the exploitation of a blind OS command injection vulnerability by crafting a time-based payload that manipulates server-side command execution. The goal is to induce a controlled delay in the application's response time, thereby verifying the presence of the vulnerability despite the absence of direct output.

### 🧭 Lab Description  
This PortSwigger lab showcases a blind OS command injection vulnerability within the feedback submission functionality of a web application. The server executes a shell command using user-supplied input, but the output is not returned in the HTTP response. To confirm the vulnerability, a time-based payload is used to induce a measurable delay, proving successful command execution without relying on output reflection.

### 🛠️ Environment Setup

1. **Proxy Configuration**
   - Launch **Burp Suite**.
   - Configure your browser (e.g., **Firefox**) to route traffic through Burp using **FoxyProxy** or a similar extension.

2. **Access the Lab**
   - Navigate to the lab URL provided by PortSwigger.
   - Locate and open the **Submit Feedback** form.

### 📝 Exploitation Steps

#### 1. Submit Initial Feedback
Populate the form with arbitrary values:
```
Name: abc  
Email: abc@g.co  
Subject: abcd  
Message: abcde
```
Click **Submit Feedback**.

#### 2. Intercept the Request
- In Burp Suite, go to **Proxy > HTTP History**.
- Locate the `POST /feedback/submit` request.
- Send the request to **Repeater**.

#### 3. Establish Baseline Response Time
- Send the original request from Repeater.
- Observe the response time (typically around **190 milliseconds**).

#### 4. Inject Time-Based Payload
Modify the `email` parameter to include a shell command that introduces a delay:
```
x||ping -c 10 127.0.0.1||
```

This payload uses the `ping` command to send 10 ICMP packets to localhost, introducing an approximate 10-second delay.

##### 🔧 Modified Request Parameters:
```
csrf=vpE0MceAOKwyO4b5hXYEkQny4ZfnYxGi
name=abc
email=x||ping+-c+10+127.0.0.1||abc%40g.co
subject=abc
message=abc
```

> ⚠️ Ensure proper URL encoding where necessary (e.g., `@` becomes `%40`).

#### 5. Send the Tampered Request
- Send the modified request via Repeater.
- Observe the response time (approximately **9400 milliseconds**), confirming the delay.

### ✅ Outcome & Validation

The significant increase in response time confirms that the injected command was executed on the server, validating the presence of a **blind OS command injection** vulnerability.

### 🧨 Security Impact

- **Remote Code Execution (RCE):** Attackers can execute arbitrary system commands.
- **Data Exfiltration:** Sensitive data can be extracted using chained commands.
- **System Compromise:** Attackers may pivot to other parts of the infrastructure.
- **Denial of Service:** Time-based payloads can be used to degrade performance or crash services.


### 🛡️ Remediation Strategies

- **Input Validation:** Sanitize and validate all user inputs before processing.
- **Use Safe APIs:** Avoid using shell commands directly; use language-native libraries for system operations.
- **Command Whitelisting:** If shell commands must be used, restrict inputs to a safe subset.
- **Least Privilege:** Run web applications with minimal system permissions.
- **Security Testing:** Regularly perform code reviews and penetration testing to identify injection points.

### 📌 Key Takeaways

- Blind command injection can be verified using **time-based techniques** when output is suppressed.
- Always test input fields that interact with server-side logic, especially those that trigger system-level operations.
- Tools like Burp Suite and Repeater are essential for controlled testing and timing analysis.

### ⚠️ Disclaimer

This lab is part of the **PortSwigger Web Security Academy** and is intended for **educational and ethical testing purposes only**. Unauthorized testing or exploitation of systems without explicit permission is illegal and unethical. Always conduct security research in controlled environments or with proper authorization.
