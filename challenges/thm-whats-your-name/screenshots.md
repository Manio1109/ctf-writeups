# Portfolio — TryHackMe: *What’s Your Name?* (Medium)

**Difficulty:** 🟠 Medium  
**Core areas:** Reconnaissance, Cross-Site Scripting (XSS) with cookie exfiltration, CSRF-like abuse paths, session hijacking, privilege escalation.

---

## 1) Nmap Scan — Initial Reconnaissance
![Nmap scan — open ports and services](/images/whatsyourname_nmap_scan.png)

**Summary:**  
Network reconnaissance performed to identify open ports and exposed services.

**Role / Tools:**  
Reconnaissance, service discovery (`nmap`).

**Result:**  
Identified web services running on ports 80 and 8081, with a webroot located at `public/html`.

**Sensitivity:**  
Low — no credentials exposed.

---

## 2) Webroot Enumeration — `public/html`
![Public webroot — public/html directory](/images/whatsyourname_public_html.png)

**Summary:**  
Exploration of the web application structure via the exposed webroot.

**Role / Tools:**  
Web enumeration, application mapping.

**Result:**  
Discovered a registration/comment feature that later became the primary attack vector.

**Sensitivity:**  
Low — no sensitive data present.

---

## 3) Cross-Site Scripting (XSS) — Proof of Concept
![XSS payload screenshot (PoC)](/images/whatsyourname_XSS_payload.png)

**Summary:**  
A reflected/stored XSS vulnerability was identified in a client-side input field that lacked proper output encoding.

**Role / Tools:**  
Client-side vulnerability testing, browser-based exploitation.

**Techniques:**  
Browser, DevTools, conceptual XSS payloads (event handlers / `onerror`), *redacted for public release*.

**Result:**  
Successfully executed JavaScript in another user’s context, enabling session cookie exfiltration to a controlled listener.

**Sensitivity:**  
**High** — exploit payloads and exfiltration endpoints must be redacted before public release.

---

## 4) Session Cookie Capture — Moderator Access
![Moderator cookie (anonymized)](/images/whatsyourname_moderator_cookie.png)

**Summary:**  
Captured a non-HttpOnly session cookie belonging to a moderator account via the XSS exploit.

**Role / Tools:**  
Session hijacking, cookie manipulation.

**Result:**  
Injected the stolen session cookie into the browser to impersonate a moderator.

**Sensitivity:**  
**Very High** — session identifiers are fully anonymized (`[REDACTED]`).

---

## 5) Login Page — Credential Context
![Login page / login form](/images/whatsyourname_login_pagina_met_cookie.png)

**Summary:**  
Login interface used to validate access and confirm successful privilege escalation.

**Role / Tools:**  
Browser-based authentication testing.

**Result:**  
Confirmed the location and flow for user, moderator, and admin authentication.

**Sensitivity:**  
Low / Medium — ensure credentials are redacted if shown.

---

## 6) Logged in as Moderator — Privilege Verification
![Logged in as moderator — dashboard view](/images/whatsyourname_moderator_login.png)

**Summary:**  
Moderator dashboard accessed using the hijacked session.

**Role / Tools:**  
Session reuse, privilege validation.

**Result:**  
Successfully accessed moderator-only functionality and retrieved the first flag.

**Sensitivity:**  
Low — no active secrets exposed.

---

## 7) Change Password — Authorization Restriction
![Change password page — insufficient rights](/images/whatsyourname_change_password.png)

**Summary:**  
Attempted to change credentials using standard functionality.

**Role / Tools:**  
Authorization testing.

**Result:**  
Confirmed that password changes require admin privileges, indicating the need for further escalation.

**Sensitivity:**  
Low.

---

## 8) CSRF-like Abuse via Chatbot Feature
![CSRF via chatbot (concept)](/images/whatsyourname_csrf_payload.png)

**Summary:**  
Identified a chatbot/internal request feature capable of dispatching authenticated backend requests.

**Role / Tools:**  
Logic abuse, CSRF-style exploitation.

**Technique (Redacted):**  
A crafted request (using an anonymized `onerror` / `fetch` concept with `credentials: include`) forced an internal POST request to the `change_password` endpoint.

**Result:**  
Admin password was changed in the CTF lab environment, enabling full admin access.

**Sensitivity:**  
**High** — payloads, endpoints, and URLs are intentionally anonymized.

---

## 9) Admin Access — Final Flag
![Admin panel — flag found](/images/whatsyourname_admin_flag.png)

**Summary:**  
Authenticated as administrator using the forced password change.

**Role / Tools:**  
Privilege escalation, admin access validation.

**Result:**  
Retrieved the second (admin-level) flag, completing the room.

**Sensitivity:**  
Medium — flags are harmless in CTF context, but credentials/logs should remain redacted.


