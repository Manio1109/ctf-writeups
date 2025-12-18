# Portfolio — TryHackMe: *Hammer*

**Difficulty:** 🟠 Medium  
**Core areas:** Authentication bypass, Python scripting, brute-force attacks, JWT token manipulation, privilege escalation, command execution.

---

## 1) Python exploit — 4-digit PIN brute force
![Python Exploit](/images/pythonscript_exploit.py.png)

**Summary:**  
Custom Python script used to brute-force a 4-digit PIN code by iterating through all possible combinations.

**Role / Tools:**  
Exploit development, Python scripting.

**Result:**  
The correct 4-digit PIN was successfully identified, allowing access to the password reset functionality.

**Sensitivity:**  
Medium.

---

## 2) Password reset & initial access
**Summary:**  
Using the recovered PIN code, the account password was changed via the web application.

**Role / Tools:**  
Browser, application logic abuse.

**Result:**  
Successful login to the target application and retrieval of the first flag.

**Sensitivity:**  
Medium.

---

# JWT Token Manipulation — Privilege Escalation

## 3) JWT token analysis & modification
![JWT Token](/images/jwt-token.png)

**Summary:**  
The issued JSON Web Token (JWT) was decoded and modified using jwt.io to escalate privileges.

**Role / Tools:**  
JWT analysis, token tampering (`jwt.io`).

**Result:**  
The token was altered to grant administrative privileges (e.g. role/claims manipulation).

**Sensitivity:**  
High.

---

## 4) Injecting the modified JWT into Burp Suite
![Burp Suite](/images/Burp-suite-hammer.png)

**Summary:**  
The manipulated JWT token was manually inserted into intercepted HTTP requests.

**Role / Tools:**  
Burp Suite (Repeater / Intercept).

**Result:**  
Authenticated as an administrator by supplying the modified token in the  
`Authorization: Bearer <JWT>` header.

**Sensitivity:**  
High.

---

## 5) Command execution & Flag 2 retrieval
**Summary:**  
With administrative access, system commands could be executed through the web interface.

**Role / Tools:**  
Burp Suite, command execution.

**Result:**  
Executed commands such as:

```bash
cat /home/ubuntu/flag.txt
````

This resulted in the successful retrieval of the second flag.

**Sensitivity:**
High.
