# 🛠️ TryHackMe: Hammer  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/hammer)

---

## 🧠 Summary  
**Hammer** is a Medium‑difficulty TryHackMe room focused on analyzing and breaking weak authentication logic.  
The challenge walks you through identifying hidden directories, extracting sensitive information, manipulating JWT tokens, and ultimately escalating privileges to achieve **Remote Code Execution (RCE)**.

This room showcases how small oversights — like exposed logs or hard‑coded secrets — can quickly chain together into full system compromise.

---

## 🧰 Tools & Techniques

| Tool / Technique | Purpose |
|------------------|---------|
| **Nmap**         | Service and port discovery |
| **ffuf**         | Directory enumeration |
| **Python script** | Brute‑forcing 4‑digit PIN codes |
| **JWT.io**        | Inspecting and signing JSON Web Tokens |
| **Burp Suite**   | Intercepting, modifying, and replaying requests |

---

## 🚀 Attack Path

### 1. 🔎 Port Scan

```bash
nmap -sV -p- -T4 10.10.81.17
```

## Open Ports

- **22/tcp** — SSH  
- **1337/tcp** — HTTP service

---

## 📂 Source Code Review

By inspecting the page source, a suspicious directory reference appeared:
```nginx
hmr_directory_name
```
This hinted at where hidden functionality might exist.

---

## 3. 🕵️‍♂️ Directory Fuzzing

**Using a wordlist based on the discovered prefix:**
```bash
ffuf -u http://10.10.51.17:1337/FUZZ/ \
     -w hmr-prefixed.txt -mc 200,301,302,403 -r
```

**Discovered Directory:**
```bash
/hmr_logs
```

---

## 4. 🔍 Log Analysis

**Navigating to the logs revealed:**
```bash
/hmr_logs/error.logs
```

**The file contained a leaked email address:**
```css
tester@hammer.thm
```
This would later be used during the password reset process.

---

## 5. 🔐 Brute‑forcing the PIN

Using the **"Forgot Password?"** mechanism triggered a PIN‑based reset flow.  
A custom Python script was used to brute‑force the **4‑digit PIN**, allowing the password to be reset.

**After logging in with the newly set password, the first flag was obtained:**
```text
THM{REDACTED}
```

---



