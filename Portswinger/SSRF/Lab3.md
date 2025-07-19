## 📘 Lab Overview
This lab demonstrates a **Blind Server-Side Request Forgery (SSRF)** vulnerability that relies on **out-of-band (OAST)** interaction verification. The target web application uses an analytics service that fetches the URL specified in the `Referer` header when a product page is loaded.
The goal is to:
- Exploit this behavior to force the server to interact with a Burp Collaborator endpoint.
- Confirm the SSRF vulnerability by observing DNS or HTTP requests in the Collaborator client.

## 🛠️ Lab Setup and Prerequisites

### ✅ Required Tools
- **Burp Suite Professional**
- **Burp Collaborator Client**
- **Web Browser** (e.g., Firefox or Chrome)
- **Proxy Switch Extension** (e.g., FoxyProxy)

### 🔧 Environment Configuration
1. Launch Burp Suite and ensure it is listening on `127.0.0.1:8080`.
2. Configure your browser to use Burp's proxy, either manually or via FoxyProxy.
3. Ensure SSL certificates are installed in your browser to intercept HTTPS traffic.
4. Access the PortSwigger lab: **Blind SSRF with Out-of-Band Detection**.

## 🎯 Exploitation Workflow

### 1. Access the Vulnerable Endpoint
- Navigate to any blog post or product page in the lab (e.g., `/product?productId=1`).
- This will automatically trigger an analytics request using the `Referer` header.

### 2. Capture the Request
- In Burp Suite, open the **Proxy → HTTP History** tab.
- Locate and inspect the relevant request:
  ```
  GET /product?productId=1 HTTP/2
  Host: <target-lab-host>
  ```
- Right-click → **Send to Repeater**.

### 3. Generate Collaborator Payload
- In Burp, open the **Burp Collaborator Client** tab.
- Click **Copy to Clipboard** to copy your unique payload:
  ```
  e.g., rkf5u61apud26uapu2nafw4c93fu3kr9.oastify.com
  ```
### 4. Modify the Referer Header
- In the Repeater request, locate the `Referer` header.
- Replace the existing value with your Burp Collaborator URL:
  ```http
  Referer: https://rkf5u61apud26uapu2nafw4c93fu3kr9.oastify.com/
  ```
- Send the modified request.

## 📡 Verifying SSRF via Collaborator

### 5. Monitor for Out-of-Band Interactions
- Return to the **Burp Collaborator Client** tab.
- Wait approximately 5 seconds.
- If no interaction appears, click **Pull Now**.

### ✅ Successful Indicators
If the SSRF is successful, you should observe:
- **DNS lookups** to your payload domain
- Possibly **HTTP requests** from the analytics system

This confirms that the server issued an outbound request to your controlled endpoint.

## 🏁 Lab Completion
Once Burp detects an interaction in the Collaborator tab, the lab is successfully solved.

## 📘 Technical Notes
- **Blind SSRF** means you cannot directly observe the response; detection relies on external evidence.
- **Burp Collaborator** acts as a listener for DNS/HTTP interactions triggered by SSRF payloads.
- The `Referer` header is trusted by the backend analytics system and is therefore a valid injection point.

## ⚠️ Disclaimer
This documentation is intended solely for educational and ethical research purposes.
The described techniques are demonstrated in a controlled lab environment provided by PortSwigger. Any unauthorized testing, exploitation, or tampering of third-party systems is strictly prohibited and may violate applicable laws. Always perform security research responsibly and with proper authorization.
