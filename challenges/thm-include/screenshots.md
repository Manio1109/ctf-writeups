# Portfolio — TryHackMe: *Include*

**Difficulty:** 🟠 Medium  
**Core areas:** Reconnaissance, Prototype Pollution, SSRF, API abuse, credential extraction, LFI, brute-force attacks, privilege escalation.

---

## 1) Nmap Scan — Initial Reconnaissance
![Nmap Scan](/images/include_nmap_scan.png)

**Summary:**  
Initial network scan to identify exposed services and open ports.

**Role / Tools:**  
Reconnaissance (`nmap`).

**Result:**  
Discovered the web service and identified the primary attack surface.

**Sensitivity:**  
Low.

---

## 2) Prototype Pollution — `isAdmin: true`
![Prototype Pollution](/images/include_isAdmin_true.png)

**Summary:**  
Exploitation of a prototype pollution vulnerability to inject the `isAdmin: true` property.

**Role / Tools:**  
Web exploitation, JavaScript logic abuse.

**Result:**  
Successfully escalated privileges within the application context.

**Sensitivity:**  
High.

---

## 3) API Dashboard — Admin Access
![API Dashboard](/images/include_api_dashboard.png)

**Summary:**  
Access to the internal API dashboard after achieving admin-level privileges.

**Role / Tools:**  
Browser, API inspection.

**Result:**  
Exposed sensitive API functionality and configuration endpoints.

**Sensitivity:**  
Medium.

---

## 4) SSRF Payload Injection — Settings
![SSRF Payload](/images/include_settings_payload.png)

**Summary:**  
Injected a Server-Side Request Forgery (SSRF) payload via the application settings.

**Role / Tools:**  
SSRF exploitation, request crafting.

**Result:**  
Forced the backend to make requests to internal resources.

**Sensitivity:**  
High.

---

## 5) SSRF Response — Internal Data Disclosure
![SSRF Result](/images/include_settings_uitkomst.png)

**Summary:**  
Captured the response from the SSRF request.

**Role / Tools:**  
SSRF chaining, response analysis.

**Result:**  
Internal data was successfully retrieved from the server.

**Sensitivity:**  
High.

---

## 6) CyberChef Decode — Credential Recovery
![CyberChef Decode](/images/include_cyberchef.png)

**Summary:**  
Decoded encoded credentials using CyberChef.

**Role / Tools:**  
CyberChef, data decoding.

**Result:**  
Recovered valid login credentials.

**Sensitivity:**  
Very High.

---

## 7) SSH Login & First Flag
![SSH Login](/images/include_inloggen_en_flag.png)

**Summary:**  
Authenticated to the target system using the recovered credentials.

**Role / Tools:**  
SSH client.

**Result:**  
Successful shell access and retrieval of the first flag.

**Sensitivity:**  
High.

---

## 8) Source Code Analysis
![Source Code Analysis](/images/include_source_code_analyse.png)

**Summary:**  
Reviewed application source code to identify further vulnerabilities.

**Role / Tools:**  
Manual code review.

**Result:**  
Discovered potential Local File Inclusion (LFI) vectors.

**Sensitivity:**  
Medium.

---

## 9) LFI Fuzzing — Burp Suite
![LFI Fuzzing](/images/include_fuzzing_burp_suite.png)

**Summary:**  
Fuzzed file inclusion parameters to exploit LFI.

**Role / Tools:**  
Burp Suite (Intruder / Repeater).

**Result:**  
Confirmed a working LFI vulnerability.

**Sensitivity:**  
High.

---

## 10) LFI Results — Sensitive Files
![LFI Results](/images/include_uitkomst_fuzzing_burp_suite.png)

**Summary:**  
Successfully retrieved sensitive files from the server.

**Role / Tools:**  
Burp Suite, LFI exploitation.

**Result:**  
Extracted user and configuration data.

**Sensitivity:**  
High.

---

## 11) Hydra Brute Force — User Accounts
![Hydra Brute Force](/images/include_bruteforcen_users.png)

**Summary:**  
Performed brute-force attacks against discovered user accounts.

**Role / Tools:**  
`hydra`.

**Result:**  
Identified valid user credentials.

**Sensitivity:**  
High.

---

## 12) Final Flag — Privilege Escalation Complete
![Final Flag](/images/include_laatste_flag_kraken.png)

**Summary:**  
Used the newly obtained credentials to fully compromise the system.

**Role / Tools:**  
Credential abuse, privilege escalation.

**Result:**  
Successfully retrieved the final flag.

**Sensitivity:**  
High.

---

