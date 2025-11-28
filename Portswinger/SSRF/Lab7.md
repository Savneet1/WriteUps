## 🧪 Lab Objective  
Exploit a **Server-Side Request Forgery (SSRF)** vulnerability in a stock check feature of a simulated web application.

🎯 **Goal:** Access the internal admin interface at `http://localhost/admin` and delete the user `carlos`.
## 🔧 Tools & Environment  
- **Burp Suite** for intercepting and modifying HTTP traffic  
- **Firefox** configured with **FoxyProxy** to route traffic through Burp  
- PortSwigger Web Security Academy Lab environment  

## 🚀 Exploitation Workflow  

### 1️⃣ Initialize Setup  
- Launch Burp Suite and configure Firefox to use it via FoxyProxy.  
- Access the lab and open any product page to trigger the stock check feature.


### 2️⃣ Capture the Target Request  
- In Burp, navigate to `Proxy → HTTP History`.  
- Locate the request to `/product/stock` and send it to **Repeater**.  
  Example request:
  ```
  POST /product/stock HTTP/1.1
  Content-Type: application/x-www-form-urlencoded
  ```
- Note the stock check API uses a URL parameter:
  ```
  stockApi=http://stock.weliketoshop.net/...
  ```
---

### 3️⃣ Bypass Hostname Whitelisting  
The application filters external URLs to prevent SSRF. This is bypassed using clever URL manipulation.

#### 💡 Key Exploit Payload
```text
stockApi=http://127.0.0.1:80%2523@stock.weliketoshop.net/admin&storeId=2
```

🧠 **Explanation:**
- `127.0.0.1` targets localhost (internal service).
- `%2523` is the double-encoded version of `#`.  
  - `%23` = `#`  
  - `%2523` = `%25` (`%`) + `23` (`#`)
- `@stock.weliketoshop.net` tricks hostname filter:
  - The app parses the hostname as `stock.weliketoshop.net`, which is whitelisted.
  - The server actually sends the request to `127.0.0.1`.

✅ This payload passes the SSRF filter while secretly reaching the internal service.

### 4️⃣ Discover Sensitive Endpoint  
- Scroll through the **response** in Burp Repeater’s "Pretty" tab.  
- You’ll find a link:
  ```
  /admin/delete?username=carlos
  ```

This confirms access to the internal admin interface was successful.

### 5️⃣ Trigger Final Exploit  
Send a modified request with the new path injected:

```text
stockApi=http://127.0.0.1:80%2523@stock.weliketoshop.net/admin/delete?username=carlos&storeId=2
```

✅ Upon execution, the internal admin deletes the `carlos` user, solving the lab.

## 🧠 Technical Deep Dive  

### 🔍 Why Port 80 and Not 8080?
- 🔥 Port 80 is the **default HTTP port** for most internal services and production apps.
- 🚫 Port 8080 is commonly used for dev servers and proxies, but it’s **not active** in this lab.
- ✅ Using port 80 ensures the SSRF actually reaches the live internal endpoint.

### 🔓 Why Use `#` Fragment and Double Encoding?  
- 🧩 The `#` symbol is a fragment identifier in URLs. It's **not sent in HTTP requests**, so it can be used to "hide" parts of the path.
- 🔄 **Double encoding** (`%2523`) ensures the app’s initial filter **doesn't decode** and process the fragment prematurely.
- 🕵️ The app sees a valid whitelisted domain `stock.weliketoshop.net`, but the actual request targets `127.0.0.1`.
📦 By exploiting quirks in URL parsing and fragment handling, the SSRF successfully bypasses the security filter.

## 🔐 Summary of Bypass Techniques  

| 🔧 Technique              | 💡 Purpose                                                        |
|--------------------------|-------------------------------------------------------------------|
| `127.0.0.1` as hostname  | Access internal service (localhost)                              |
| `%2523` fragment encoding| Trick URL parsing to bypass filter and mask real destination     |
| `@stock.weliketoshop.net`| Whitelisted domain used to pass host validation check            |
| Port 80 usage            | Target live service (default HTTP port)                          |
| Endpoint discovery       | Reveal hidden internal paths (`/admin/delete`)                   |

## 🛡️ Ethical Disclaimer  
This content is for **educational purposes** only, based on authorized testing within PortSwigger’s Web Security Academy.  
Please use responsible disclosure practices and avoid unauthorized testing in live environments.
