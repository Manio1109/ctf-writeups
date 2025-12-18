# 🛠️ TryHackMe: Injectics  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/injectics)

---

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

#### About This Write-Up
This document is part of my offensive security portfolio.  
My goal is to present structured, professional case studies that demonstrate practical penetration testing skills, clear reporting, and a real-world methodology.

These write-ups are meant to demonstrate practical offensive security skills relevant to junior pentesting, red teaming, and application security roles.

---

## 🧠 Summery
**Injectics** is a medium-difficulty web exploitation room focused on identifying and chaining classic web vulnerabilities into full system compromise.

**Key themes:**
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

### 5. 🔑 Administrative Access & Flag 1

After allowing the database restoration process to complete, the application returned to its default state.  
At this point, the previously exposed credentials from `mail.log` became valid again.

Using the restored **superadmin** credentials, a successful login was performed via the application’s login page.

Once authenticated, access to the administrative interface was granted. This confirmed that:
- Authentication controls had been fully bypassed.
- Administrative privileges were obtained without legitimate authorization.

Within the admin panel, the first flag was located.

**Flag 1:**
```text
THM{REDACTED}
```

**Security impact:**
- Full administrative access was achieved.
- Sensitive application functionality became accessible.
- This access enabled further exploitation paths, including template manipulation and code execution.

At this stage, the attacker had complete control over the web application layer, setting the stage for escalation to system-level access.

---

### 6. 🧩 Identifying Server-Side Template Injection (SSTI)

With administrative access obtained, further analysis of the application’s functionality was performed to identify additional attack vectors.

Within the admin dashboard, a user-controlled input field was discovered in the **profile configuration section**. Changes made to this field were reflected dynamically in server-rendered pages, making it a strong candidate for server-side template injection testing.

To confirm whether template expressions were being evaluated by the backend, the following benign test payload was submitted:

```twig
{{7*7}}
```

The application returned the evaluated result:
```text
49
```

**This response confirmed that:**
- User input was being processed directly by the template engine.
- No sanitization or sandboxing was in place.
- The application was vulnerable to **Server-Side Template Injection (SSTI).**

Further inspection of error messages and rendering behavior indicated that the application was using the **Twig** templating engine.

**Security impact:**
- Server-side template evaluation allowed arbitrary expression execution.
- Twig SSTI can often be escalated to remote code execution.
- This vulnerability enabled escalation beyond the application layer.

The presence of SSTI marked a critical escalation point, as it opened the door to executing arbitrary system commands on the server.

---

### 7. 🖥️ Remote Code Execution via SSTI (Twig)

After confirming the presence of Server-Side Template Injection, the next objective was to escalate this vulnerability to **Remote Code Execution (RCE)**.

An initial attempt was made to directly execute system commands using a common Twig payload:

```twig
{{ _self.constructor.constructor("passthru('ls')")() }}
```

This payload resulted in an error, indicating that direct constructor access was restricted or partially sandboxed.

Further testing led to the discovery of an alternative technique that leverages Twig’s `sort` filter to invoke PHP functions indirectly. The following payload successfully executed a system command:
```twig
{{['id',""]|sort('passthru')}}
```

The server responded with the output of the executed command:
```text
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**This confirmed that:**
- Arbitrary system commands were executed on the server.
- Code execution occurred in the context of the www-data user.
- The SSTI vulnerability had been successfully escalated to full RCE.

**Security impact:**
- Complete compromise of the web server execution context.
- Ability to execute arbitrary shell commands.
- Potential access to sensitive files, credentials, and system resources.

At this stage, the attacker effectively gained command execution capabilities on the underlying operating system.

---

### 8. 🏴 Locating and Extracting the Final Flag

With remote code execution achieved, the final step was to locate sensitive files on the system.

Using the previously established command execution via SSTI, directory enumeration was performed to identify potential locations of the flag files. The following command was executed to list the contents of the `flags` directory:

```twig
{{['ls -la flags',""]|sort('passthru')}}
```

The output revealed a file with a randomized name:
```text
5d8af1dc14503c7e4bdc8e51a3469f48.txt
```

The contents of this file were then read using another SSTI payload:
```twig
{{['cat flags/5d8af1dc14503c7e4bdc8e51a3469f48.txt',""]|sort('passthru')}}
```

This returned the final flag:
```text
THM{REDACTED}
```

Impact:
- Successful read access to sensitive server-side files.
- Confirmation of full exploitation from initial injection to system-level access.
- Completion of the attack chain for the Injectics room.

This step concluded the exploitation process, demonstrating how chained web vulnerabilities can escalate from simple injection flaws to full remote code execution.

---

## 🏁 Flags Obtained

| 🏷️ Flag Type | Technique Used                | Result |
|--------------|-------------------------------|--------|
| Admin Flag   | SQL Injection → Admin login   | `THM{REDACTED}` |
| Root Flag    | SSTI → RCE → File read        | `THM{REDACTED}` |

---

## Room links

📸 [screenshots](../../challenges/thm-injectics/screenshots.md)

🔗 [TryHackMe - Injectics](https://tryhackme.com/room/injectics)

---

## 💭 Reflection

This room demonstrated how classic web vulnerabilities can still be highly effective when combined thoughtfully.  
Rather than relying on obscure or protocol-level flaws, **Injectics** focused on chaining well-known issues into a complete compromise of the application and underlying system.

The attack path progressed through:
- **Information disclosure** via exposed log files
- **SQL Injection** caused by insufficient server-side input validation
- Abuse of **application recovery mechanisms** that restore default credentials
- **Server-Side Template Injection (SSTI)** leading to full **Remote Code Execution (RCE)**

What made this room particularly valuable was how it highlighted the dangers of trusting client-side controls and assuming that “self-healing” mechanisms improve security.

---

### Key Takeaways

- **Client-side filtering provides no real security** and can be trivially bypassed with request manipulation.
- **Exposed logs and configuration files** often contain high-value information such as credentials.
- **Automatic recovery mechanisms** can be abused to force applications back into a predictable, vulnerable state.
- **SSTI vulnerabilities are extremely dangerous**, as they often lead directly to remote code execution.
- Chaining multiple medium-impact issues can result in **full system compromise**.

---

### Lessons Learned

- **All input validation must be enforced server-side**, regardless of client-side restrictions.
- **Sensitive files (logs, backups, configs)** should never be publicly accessible.
- **Template engines must be sandboxed properly** or user input must never be rendered directly.
- **Defense-in-depth is critical**: relying on a single protection layer (e.g., client-side filters) creates a false sense of security.
- Even common vulnerabilities like SQLi remain highly impactful when combined with poor architectural decisions.

This room reinforced the importance of methodical analysis and vulnerability chaining, showing how an attacker can progress from a simple injection flaw to full remote code execution when multiple weaknesses coexist.

