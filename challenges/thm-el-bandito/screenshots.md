# Portfolio — TryHackMe: *El Bandito* (Hard)

**Difficulty:** 🔴 Hard  
**Core areas:** Reconnaissance, SSRF chaining, WebSocket/101 proxy confusion, HTTP/2 → HTTP/1.1 request smuggling (H2.CL).

---

## 1) Nmap scan — Basic network reconnaissance
![Nmap scan](/images/elbandito_nmap.png)

**Summary:** Output of a full `nmap -sV -p- -T4` scan.  
**Role / Tools:** Recon (`nmap`).  
**Result:** Services discovered on ports 22 (SSH), 80 (web), 631 (CUPS), and 8080 (nginx).  
**Sensitivity:** Low.

---

## 2) Initial visit — main page (port 80)
![Main page — nothing to see here](/images/elbandito_pagina_80.png)

**Summary:** Static frontend displaying the text “nothing to see here”.  
**Role / Tools:** Browser / DevTools.  
**Result:** Page appears empty, but references external JavaScript — prompting source code inspection.  
**Sensitivity:** Low.

---

## 3) Page source — `static/messages.js` discovered
![Page source — messages.js reference](/images/elbandito_source_code_script.png)

**Summary:** Reference to `/static/messages.js` found in the page source.  
**Role / Tools:** DevTools / source code inspection.  
**Result:** Location of client-side chat logic and API endpoints identified.  
**Sensitivity:** Low.

---

## 4) `messages.js` — front-end chat logic
![messages.js analysis](/images/elbandito_messages_script.png)

**Summary:** Client-side chat implementation (Jack & Oliver), including endpoints such as `/getMessages` and `/send_message`.  
**Role / Tools:** JavaScript code audit.  
**Result:** Logic and bug observations suggesting potential attack vectors (bot triggering, message handling).  
**Sensitivity:** Low / Medium.

---

## 5) Gobuster (port 80) — `/access` discovered
![Gobuster port 80 results](/images/elbandito_gobuster_port_80.png)

**Summary:** Directory fuzzing revealed the `/access` endpoint.  
**Role / Tools:** `gobuster`.  
**Result:** `/access` appears to be a management/login entry point — relevant for further investigation.  
**Sensitivity:** Low.

---

## 6) `/access` — login required
![Access login page](/images/elbandito_access_login_pagina.png)

**Summary:** Login page requiring credentials.  
**Role / Tools:** Browser.  
**Result:** Indicates a privilege-separated area (useful once credentials are obtained).  
**Sensitivity:** Low.

---

## 7) Burn tokens page — WebSocket hint
![Burn tokens / websocket](/images/elbandito_burn_tokens_websocket.png)

**Summary:** `burn.html` hints at WebSocket functionality, though the connection did not work directly.  
**Role / Tools:** Browser / DevTools.  
**Result:** Suggests a service layer using WebSockets — useful for SSRF/WebSocket chaining concepts.  
**Sensitivity:** Low.

---

## 8) Services status — WebSocket offline, public online
![Services status online/offline](/images/elbandito_websocket_offline.png)

**Summary:** Page displaying service statuses (online/offline).  
**Role / Tools:** UI inspection.  
**Result:** Points to an `isOnline` checker capable of pinging external URLs — a typical SSRF attack surface.  
**Sensitivity:** Low.

---

## 9) Burp intercept — `GET /isOnline?url=...`
![Burp intercept of isOnline request](/images/elbandito_onderscheppen_request_via_burp.png)

**Summary:** Intercepted an `isOnline` request in Burp to observe how the server requests external URLs.  
**Role / Tools:** Burp Suite (intercept).  
**Result:** Insight into request format and headers — essential for reproducing and manipulating SSRF.  
**Sensitivity:** Low.

---

## 10) Trace / diagnostics probing via Burp
![Trace endpoint probes](/images/elbandito_trace_directory_checken_burp.png)

**Summary:** Probing trace/diagnostic endpoints to identify internal information.  
**Role / Tools:** Burp Suite.  
**Result:** Many admin/trace endpoints returned 403 externally, making SSRF interaction particularly interesting.  
**Sensitivity:** Low / Medium.

---

## 11) `GET /admin-creds` — admin credentials discovered (SSRF)
![Admin creds via SSRF](/images/elbandito_admin-creds_checken_burp.png)

**Summary:** Admin credentials leaked via the SSRF path:  
`username: hAckLIEN`  
`password: YouCanCatchUsInYourDreams404`

**Role / Tools:** SSRF chaining, Burp.  
**Result:** Proof that internal data is exposed through the proxy/SSRF chain.  
**Sensitivity:** Very High.

---

## 12) `GET /admin-flag` — Flag 1
![Admin flag via SSRF](/images/elbandito_admin-flag_checken_burp.png)

**Summary:** First flag retrieved via an internal endpoint:  
`THM{:::MY_DECLINATION:+62°_14'_31.4'':::}`

**Role / Tools:** SSRF chaining.  
**Result:** CTF evidence that SSRF and proxy chaining were successful.  
**Sensitivity:** Medium.

---

## 13) Logging in with recovered credentials
![Login with credentials](/images/elbandito_inloggen_met_credentials.png)

**Summary:** Successful login at `/access` using the recovered admin credentials.  
**Role / Tools:** Browser / DevTools.  
**Result:** Access to the admin UI and additional attack surface for deeper analysis.  
**Sensitivity:** High.

---

## 14) Messages page — chat interface (authenticated)
![Messages page logged in](/images/elbandito_messages_pagina.png)

**Summary:** Authenticated chat interface (Jack & Oliver) — the central communication component.  
**Role / Tools:** Browser, chat flow analysis.  
**Result:** Context for the send/get message flow later used in H2 request smuggling tests.  
**Sensitivity:** Low.

---

## 15) Varnish cache indicators
![Varnish cache detection](/images/elbandit0_varnish_cache_server.png)

**Summary:** Response headers indicate the presence of a Varnish cache layer.  
**Role / Tools:** Header inspection, Burp.  
**Result:** Varnish implies a frontend proxy handling HTTP/2 → HTTP/1.1, creating ideal conditions for H2.CL desynchronization.  
**Sensitivity:** Low.

---

## 16) POST manipulation & H2.CL experiments
![POST request experiments](/images/elbandito_post_request_onderschept_en_aangepast.png)

**Summary:** Experiments with malformed `Content-Length` values in HTTP/2 requests to force H2 → H1 desynchronization.  
**Role / Tools:** Burp Suite (HTTP/2 testing, “Update Content-Length” disabled).  
**Result:** Desync confirmed; a successful request-smuggling attack ultimately yielded the second flag.  
**Sensitivity:** Medium.

---

## 17) Flag 2 — H2.CL request smuggling (result)
![Flag 2 — H2.CL exploit](/images/elbandito_get_request_met_flag_ontvangen.png)

**Summary:** Second flag obtained after a successful HTTP/2 → HTTP/1.1 desync exploit.  
**Role / Tools:** H2.CL request smuggling, Burp Suite.  
**Result:** Confirmation that embedded requests reached the backend despite Varnish/proxy logic.  
**Sensitivity:** Medium.

---

## Retrieved flags (CTF)

- **Flag 1 (admin / SSRF):**  
  `THM{:::MY_DECLINATION:+62°_14'_31.4'':::}`

- **Flag 2 (H2.CL smuggling):**  
  `THM{¡!¡RIGHT_ASCENSION_12h_36m_25.46s!¡!}`

