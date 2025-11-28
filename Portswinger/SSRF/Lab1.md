## 🧪 PortSwigger SSRF Lab: Exploiting SSRF to Access Internal Services

This document provides a comprehensive walkthrough for solving the PortSwigger lab titled **"Basic SSRF against the local server."** The objective is to exploit a Server-Side Request Forgery (SSRF) vulnerability in a stock check feature to access internal administrative functionality and remove the user `carlos`.

## 📘 Lab Overview

The target application includes a product stock checker that performs server-side HTTP requests. When the stock API URL is influenced by unvalidated user input, it opens the door to SSRF attacks. In this scenario, we redirect the server's request to an internal endpoint (`http://localhost/admin`) and use that access to perform privileged operations.

## 🎯 Objective

- Exploit SSRF through manipulation of the `stockApi` parameter.
- Access the internal admin interface via localhost.
- Delete the user `carlos` from the system.

## ⚙️ Requirements

To complete this lab, ensure the following tools and configurations are available:

- [Burp Suite Community Edition](https://portswigger.net/burp)
- A browser configured with a proxy extension (e.g., [FoxyProxy](https://getfoxyproxy.org/))
- An active PortSwigger Lab account

## 🔍 Exploitation Procedure

### 1. Initialize the Lab

- Launch the lab via your PortSwigger dashboard.
- Navigate to any blog post or product page within the application.

### 2. Intercept the Stock Check Request

- Open Burp Suite and ensure your browser proxy is active.
- Click **"Check Stock"** on any product.
- In Burp, go to `Proxy > HTTP History` and locate the intercepted request:
  ```
  POST /product/stock HTTP/1.1
  ```
- Right-click the request and send it to the **Repeater** tab for editing.

### 3. Modify the `stockApi` Parameter

In Repeater, identify the URL-encoded `stockApi` value:
```
stockApi=http%3A%2F%2Fstock.weliketoshop.net%3A8080%2Fproduct%2Fstock%2Fcheck%3FproductId%3D1%26storeId%3D2
```

Replace it with:
```
stockApi=http://localhost/admin
```

- Send the modified request.
- ✅ The response should contain the HTML content for the admin panel.

## 🧨 Deleting User `carlos`

You now have two options to complete the objective:

### ✅ Method 1: Manual Deletion

- Right-click the response → *Request in browser → In current browser session*.
- Copy and open the URL in your browser.
- Locate the user `carlos` and use the interface to delete them.

### ✅ Method 2: Direct SSRF Trigger

- In the response, locate the hyperlink for deleting `carlos`:
  ```html
  <a href="/admin/delete?username=carlos">
  ```
- Update your `stockApi` parameter to:
  ```
  stockApi=http://localhost/admin/delete?username=carlos
  ```
- Submit the updated request.
- Right-click → *Request in browser → In current browser session*.
- Open the link in your browser to execute the deletion.

✅ Success — user `carlos` has been removed, and the lab is complete.

## 🧠 Security Insights

- SSRF vulnerabilities arise when server-side functionality can be manipulated by external user input.
- Internal services accessible via `localhost` or internal IP ranges can often expose sensitive endpoints.
- Validating and sanitizing URLs in user-controlled parameters is essential to mitigate SSRF risks.

## ⚠️ Disclaimer
This walkthrough is intended for educational use only within legal and authorized environments such as PortSwigger Labs. Unauthorized testing against real-world systems without consent is strictly prohibited.
