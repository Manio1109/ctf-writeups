# Portfolio — TryHackMe: *Injectics*

**Difficulty:** 🟠 Medium  
**Core areas:** Source code analysis, authentication bypass, SQL injection, SSTI, template engine abuse (Twig), remote command execution (RCE).

---

## 1) Source Code Analysis — Credential Discovery
![Source Code Analysis](/images/injectics_mail.log_file_.png)

**Summary:**  
While reviewing the application source code, a reference to the file `mail.log` was discovered.  
This file contained default login credentials used by the application.

**Role / Tools:**  
Manual source code review.

**Result:**  
Recovered valid credentials that could later be used to authenticate.

**Sensitivity:**  
High.

---

## 2) Login Bypass — Burp Suite Payload Manipulation
![Burp Suite Payload](/images/injectics_burp_suite_inloggen.png)

**Summary:**  
Client-side input validation was bypassed by intercepting and modifying HTTP requests directly.

**Role / Tools:**  
Burp Suite (Intercept / Repeater).

**Result:**  
Successfully bypassed the login mechanism by submitting a crafted request.

**Sensitivity:**  
High.

---

## 3) SQL Injection — Dropping the `users` Table
![Dropping Tables](/images/injectics_drop_table_users.png)

**Summary:**  
A SQL injection payload was tested that dropped the `users` table from the database.

**Role / Tools:**  
SQL injection testing, Burp Suite.

**Result:**  
The application triggered an automatic recovery mechanism that recreated the table, restoring default data.

**Sensitivity:**  
High.

---

## 4) Admin Login — Default Credentials Restored
![Logged in as Admin](/images/injectics_ingelogd_als_admin.png)

**Summary:**  
After the database recovery, the previously discovered default admin credentials became valid again.

**Role / Tools:**  
Browser authentication.

**Result:**  
Logged in as an administrator, gained access to the admin panel, and retrieved the first flag.

**Sensitivity:**  
High.

---

## 5) Server-Side Template Injection (SSTI) Detection
![SSTI Confirmation](/images/injectics_template_bevestigen.png)

**Summary:**  
Injected the payload `{{7*7}}` into a template-rendered input field.

**Role / Tools:**  
SSTI testing.

**Result:**  
The application returned `49`, confirming a Server-Side Template Injection vulnerability.

**Sensitivity:**  
High.

---

## 6) Template Engine Identification — Twig
![Twig Confirmation](/images/injectics_twig_bevestiging.png)

**Summary:**  
The application’s response revealed that the Twig template engine was in use.

**Role / Tools:**  
Response analysis, framework fingerprinting.

**Result:**  
Twig was confirmed, enabling escalation from SSTI to Remote Code Execution (RCE).

**Sensitivity:**  
High.

---

## 7) Twig Payload — Remote Command Execution
![Flag Payload](/images/injectics_flag_uitlezen.png)

**Summary:**  
A crafted Twig payload was used to execute shell commands on the server.

**Role / Tools:**  
Twig SSTI exploitation.

**Result:**  
Successfully executed commands and accessed files within the `flags` directory.

**Sensitivity:**  
Critical.

---

## 8) Final Flag — Full Compromise
![Flag Retrieved](/images/injectics_flag_gekraakt.png)

**Summary:**  
Using RCE via Twig, the contents of the flag file were read from the `flags` directory.

**Role / Tools:**  
Remote command execution.

**Result:**  
Retrieved the second and final flag, completing the room.

**Sensitivity:**  
Critical.


