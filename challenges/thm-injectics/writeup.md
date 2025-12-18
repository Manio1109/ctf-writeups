# 🛠️ TryHackMe: Injectics  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/injectics)

---

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

---

## 🧠 Overview

**Injectics** is a medium-difficulty web exploitation room focused on identifying and chaining classic web vulnerabilities into full system compromise.

The challenge walks through:
- **SQL Injection** for authentication bypass  
- **Abusing application recovery logic**  
- **Server-Side Template Injection (SSTI)**  
- Escalation to **Remote Code Execution (RCE)**  

The room highlights how seemingly small weaknesses (client-side filtering, exposed logs) can be chained into complete administrative and system-level access.

---

## 🧰 Tools & Techniques

| Tool / Technique | Purpose |
|------------------|---------|
| **Nmap** | Port scanning and service discovery |
| **Source code analysis** | Identifying exposed credentials and logic flaws |
| **Burp Suite** | Intercepting and modifying HTTP requests |
| **SQL Injection** | Authentication bypass & database manipulation |
| **SSTI (Twig)** | Template injection leading to RCE |

---

## 🚀 Attack Walkthrough

### 1. 🔎 Initial Recon — Port Scanning

```bash
nmap -sV -p- -T4 <target-ip>
```
**Open ports identified:**
22/tcp — SSH
80/tcp — HTTP

The attack surface is limited to a web application running on port 80.

---

### 2. 📂 Source Code & Log Analysis

After identifying the web service on port 80, the next step was to analyze the application’s publicly accessible pages and client-side assets.

During this inspection, a reference was found indicating that application-related emails were stored in a file named **`mail.log`**. This log file was directly accessible from the web root, representing a serious information disclosure issue.

Upon reviewing `mail.log`, it became clear that the file contained **default user credentials** used by the application. Even more critically, the application logic revealed that these credentials were **automatically restored at regular intervals**, meaning that any changes to user passwords were temporary.

**Exposed credentials discovered:**

| Email | Password |
|------|----------|
| superadmin@injectics.thm | superSecurePasswd101 |
| dev@injectics.thm | devPasswd123 |

Further analysis of the login functionality revealed client-side input filtering implemented in `script.js`:

```javascript
const invalidKeywords = ['or', 'and', 'union', 'select', '"', "'"];
```

This filtering only occurred on the client side and could easily be bypassed by intercepting and modifying requests before they reached the server.

**Security impact:**
- Sensitive credentials were exposed via a publicly accessible log file.
- Automatic credential restoration weakened account security.
- Client-side-only filtering provided no real protection against injection attacks.

This combination of issues strongly suggested that authentication controls could be bypassed and that the login mechanism was a promising attack surface for further exploitation.

---

### 3. 🎭 Client-Side Filter Bypass (SQL Injection)

With the client-side filtering mechanism identified, the next step was to determine whether the backend properly validated user input.

Using **Burp Suite**, the login request was intercepted and sent to **Repeater** for manual testing. Because the filtering was implemented only in JavaScript, modifying the request before it reached the server completely bypassed these restrictions.

Multiple payloads were tested to assess backend behavior. A successful authentication bypass was achieved with the following SQL injection payload:

```sql
username=a' || 1=1-- -&password=test&function=login
```

This payload forces the SQL query to always evaluate as true, granting access without valid credentials.

**Observed behavior:**
- The server accepted the injected payload without sanitization.
- Authentication was bypassed successfully.
- This confirmed that the backend database queries were vulnerable to SQL injection.

**Security impact:**
- Client-side filtering provided a false sense of security.
- The lack of server-side input validation allowed direct SQL injection.
- Authentication controls could be fully bypassed by an unauthenticated attacker.

This vulnerability served as the initial foothold and enabled further interaction with privileged application functionality.

---

### 4. 💣 Database Manipulation — Dropping Tables

After confirming that the backend was vulnerable to SQL injection, further testing was performed to understand the database behavior and potential impact.

Using the previously identified injection point, a payload was submitted to intentionally remove the `users` table:

```sql
1;DROP TABLE users;
```

Instead of permanently breaking the application, the server responded with a message indicating that a recovery mechanism was in place:
*"Seems like database or some important table is deleted. InjecticsService is running to restore it. Please wait for 1–2 minutes."*

This response revealed that the application automatically restores critical database tables when they are missing.

**Key observations:**
- The backend allowed stacked SQL queries.
- Destructive SQL commands were executed without restriction.
- A background service restored the database state automatically.

**Security impact:**
- The ability to execute destructive SQL queries confirmed full database-level injection.
- Automatic restoration logic introduced predictable default states.
- Default credentials were reliably reintroduced, enabling repeated administrative access.

This behavior significantly lowered the attack complexity, as it allowed an attacker to force the application back into a known, exploitable configuration at any time.

---

