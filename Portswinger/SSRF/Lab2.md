# 🧪 SSRF Exploitation Lab: Scanning Internal Network and Deleting a User

## 📘 Lab Objective
This lab demonstrates a basic **Server-Side Request Forgery (SSRF)** vulnerability in a stock check feature. The goal is to:
- Exploit SSRF to scan the internal `192.168.0.X` IP range.
- Locate the **admin interface** running on port `8080`.
- Use the admin panel to **delete the user `carlos`**.

## 🛠️ Setup Instructions
### 1. Access the Lab Interface
- Launch the lab provided by the platform.
- Open any product page by clicking on its associated post.

### 2. Configure Burp Suite for Interception
- Start **Burp Suite**.
- Enable your browser proxy. You may use **FoxyProxy** or a similar proxy-switching extension.
- Navigate to the stock check feature and click **Check Stock**.
- Go to `Proxy → HTTP History` in Burp Suite to capture the HTTP request.

## 📥 Initial Request Analysis
The captured stock API request will contain a URL-encoded parameter similar to:
```
stockApi=http%3A%2F%2F192.168.0.1%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D3%26storeId%3D1
```
Decoded version:
```
http://192.168.0.1:8080/product/stock/check?productId=3&storeId=1
```

## 🚨 SSRF Attack: Discovering Admin Panel
### 3. Modify the Request for Internal Scanning
Change the request to probe IPs across the `192.168.0.X` range:

```
stockApi=http://192.168.0.x:8080/admin&storeId=1
```

> ⚠️ Do not remove `&storeId=1`. This preserves the request format and ensures it remains valid.

### 4. Use Burp Suite Intruder
1. Send the request to the **Intruder** tab.
2. Highlight `x` in `192.168.0.x` and set it as the payload position.
3. Configure the payload:
   - **Payload Type:** Numbers
   - **Range:** 0 to 255

Start the attack and monitor responses.

## ✅ Identifying the Admin Panel
- Look for any request returning **HTTP Status Code: 200 OK**.
- This indicates the admin panel is accessible at that IP.

## 🧨 Exploitation: Deleting Carlos
### Option A: Direct via Burp Suite
- Send the working request to the **Repeater** tab.
- Inspect the response in **Pretty View**.
- Look for the endpoint or link to delete the user `carlos`.
- Craft and send a DELETE request as needed.

### Option B: Using Graphical Interface
- Copy the discovered admin URL (`200 OK`).
- Paste it into your browser.
- Use the visual admin panel to delete `carlos`.

## 🏁 Lab Completion
Upon deleting `carlos`, the lab will confirm your success.

## ⚠️ Disclaimer
This write-up is intended for educational and ethical research purposes only.
The content demonstrates SSRF exploitation techniques within a controlled lab environment. Unauthorized testing or exploitation of systems without permission is **illegal** and **unethical**. Always obtain explicit authorization before conducting penetration testing or security assessments.
Use this information responsibly.
