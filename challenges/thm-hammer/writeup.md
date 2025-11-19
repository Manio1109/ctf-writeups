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
```text
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
```text
### Final Modified Token:
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6Ii92YXIvd3d3L2h0bWwvMTg4YWRlMS5rZXkifQ.eyJpc3MiOiJodHRwOi8vaGFtbWVyLnRobSIsImF1ZCI6Imh0dHA6Ly9oYW1tZXIudGhtIiwiaWF0IjoxNzUzODE3NjM5LCJleHAiOjE3NTM4MjEyMzksImRhdGEiOnsidXNlcl9pZCI6MSwiZW1haWwiOiJ0ZXN0ZXJAaGFtbWVyLnRobSIsInJvbGUiOiJhZG1pbiJ9fQ.89bKAVq7f0ytB5aFIXtvrK2_I0wV9vJSR83ihrS40O0
```
This token granted full administrative access.

---

## 8. 📬 Token Abuse for RCE

With the newly forged admin JWT, privileged functionality became accessible.  
Using **Burp Suite**, an authenticated request was intercepted and modified to execute system commands.

The following command was injected to retrieve the final flag:

```bash
cat /home/ubuntu/flag.txt
```
Retrieved Flag:
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

