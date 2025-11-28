# 🛡️ SSRF with Blacklist-Based Input Filter — PortSwigger Lab Writeup

## 📌 Lab Overview
This lab from PortSwigger Web Security Academy demonstrates how weak SSRF defenses, specifically blacklist-based filters, can be bypassed to access internal administrative functionality.

### 🧭 Objective
- Exploit SSRF to access the internal administrative interface at `http://localhost/admin`
- Successfully delete user `carlos` via administrative functionality

## 📖 Background: Server-Side Request Forgery (SSRF)
**Server-Side Request Forgery (SSRF)** occurs when an application fetches resources using URLs supplied by the client, without properly validating those URLs. This can allow an attacker to:
- Interact with internal systems not exposed to the public
- Exfiltrate sensitive data (e.g., cloud instance metadata)
- Access administrative endpoints within trusted networks

SSRF defenses often include:
- IP and domain blacklist filtering
- URL path restrictions
- Input sanitization
This lab focuses on the weakness of blacklist filtering.

## 🧪 Lab Initialization
### Prerequisites
- Burp Suite Community or Professional Edition
- Browser configured with Burp Suite proxy

### Steps
1. Access the lab via the provided URL.
2. Click any blog post to reveal the stock check functionality.
3. Launch Burp Suite and ensure **intercept is on** and **proxy is functioning**.
4. Click **Check stock** — Burp Suite captures a POST request to `/product/stock`.
5. Right-click the request and send it to **Repeater** (`Ctrl + R`).

## 🔍 Vulnerability Identification
The intercepted request includes a `stockApi` parameter used for backend resource fetching:
```http
POST /product/stock HTTP/1.1
Host: <lab-host>
Content-Type: application/x-www-form-urlencoded

stockApi=http://stock.weliketoshop.net/product/itemId
```
Attempting to replace this URL with internal targets reveals basic blacklist filtering.

## ❌ Rejected Payloads
Initial attempts to reach internal services using variations of localhost IPs fail:
| URL Attempt                 | Result        |
|----------------------------|---------------|
| http://localhost/admin     | Blocked       |
| http://127.0.0.1/admin     | Blocked       |
| http://127.0.1/admin       | Blocked       |
| http://127.1/admin         | Blocked       |
Although `127.1` is a valid local address (`127.0.0.1 ≡ 127.1.0.0/8`), the lab’s filter blocks common representations of localhost.

## ✅ Exploitation via Case Manipulation
URL filters in this lab do not normalize case sensitivity in paths. Modifying the path to include uppercase characters bypasses the blacklist:
```http
stockApi=http://127.1/Admin
```
📌 **Observation**: Altering case of any character in `/admin` (e.g., `/aDmin`, `/AdmIn`) successfully circumvents the filter.

## 🧑‍💻 Admin Panel Access & User Deletion
### Method 1: Via Burp Suite
- After successful bypass, the response in Repeater contains admin panel HTML.
- Identify the request to delete user `carlos`.
- Manually craft and send a request that targets the deletion endpoint.

### Method 2: Via Browser
- Copy the working admin panel URL from Burp response.
- Paste into browser to interact with the panel GUI.
- Use Burp Interceptor to capture and modify the delete request.
- Ensure successful status code (e.g., `200 OK`) upon deletion.
> 🔁 If deletion fails, retry with valid session parameters and confirm response.

## 🏁 Lab Solved
Successful deletion of `carlos` completes the lab. This demonstrates effective SSRF exploitation by bypassing naive blacklist defenses through case obfuscation.

## 🔍 Key Learnings
- **Blacklist filtering is inadequate** as a primary SSRF defense mechanism.
- Filters must normalize input and validate IP ranges and URL paths.
- Proper SSRF mitigation includes:
  - Whitelisting allowed domains
  - Restricting URL schemas
  - Disabling access to internal networks

## ⚠️ Ethical Disclosure & Disclaimer
> **This report is intended for educational and ethical security research purposes only.**
Any unauthorized exploitation or vulnerability testing on systems without explicit consent may violate legal statutes and ethical guidelines. Always ensure:
- Written authorization from system owners
- Compliance with local laws and organizational policies
- Usage of safe, approved platforms (e.g., PortSwigger Academy, Hack The Box)
Respect boundaries. Ethical research improves security without compromising integrity.

## 📚 References
- [PortSwigger Web Security Academy – SSRF](https://portswigger.net/web-security/ssrf)
- [OWASP SSRF Prevention Guide](https://owasp.org/www-community/Server_Side_Request_Forgery)
- [PayloadsAllTheThings – SSRF Payloads](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
