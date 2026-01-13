# Portfolio — TryHackMe: *Padelify* (Medium)

**Difficulty:** 🟠 Medium  
**Core areas:** Web exploitation, WAF bypass, Blind XSS, session hijacking, Local File Inclusion (LFI), input encoding evasion.

---

## 1) Nmap scan — Basic network reconnaissance
![Nmap scan](/images/thm_padelify_nmap_scan.png)

**Summary:** Output of a basic `nmap -Pn` scan against the target host.  
**Role / Tools:** Reconnaissance (`nmap`).  
**Result:** Open services discovered on ports 22 (SSH) and 80 (HTTP).  
**Sensitivity:** Low.

---

## 2) Initial visit — registration page (port 80)
![Registration page](/images/thm_padelify_registartiepagina.png)

**Summary:** Initial landing page showing a user registration form.  
**Role / Tools:** Browser.  
**Result:** Account creation requires moderator approval — indicating role-based access control.  
**Sensitivity:** Low.

---

## 3) Login page
![Login page](/images/thm_padelify_inlogpagina.png)

**Summary:** Login interface following the registration process.  
**Role / Tools:** Browser.  
**Result:** Authentication portal identified as primary entry point.  
**Sensitivity:** Low.

---

## 4) Trying default payload — SQLi test
![Trying default payload](/images/thm_padelify_standaar_payload_proberen.png)

**Summary:** Attempted classic SQL injection payload `' OR 1=1 --`.  
**Role / Tools:** Manual injection testing.  
**Result:** Injection failed — no SQLi vulnerability present.  
**Sensitivity:** Low.

---

## 5) Gobuster blocked by firewall
![Gobuster not working](/images/thm_padelify_gobuster_werkt_niet.png)

**Summary:** Initial Gobuster scan blocked by the Web Application Firewall.  
**Role / Tools:** Enumeration (`gobuster`).  
**Result:** WAF actively blocks automated scanner signatures.  
**Sensitivity:** Low.

---

## 6) Curl — firewall detected
![Curl firewall detected](/images/thm_padelify_curl_firewall_detected.png)

**Summary:** Direct HTTP request returns `403 Forbidden`.  
**Role / Tools:** Manual testing (`curl`).  
**Result:** Explicit WAF detection confirmed.  
**Sensitivity:** Low.

---

## 7) Gobuster working — User-Agent bypass
![Gobuster working](/images/thm_padelify_gobuster_werkt.png)

**Summary:** Gobuster scan using a spoofed browser User-Agent.  
**Role / Tools:** Enumeration (`gobuster`, header manipulation).  
**Result:** Directory discovery successful after bypassing WAF signature detection.  
**Sensitivity:** Low.

---

## 8) Firewall blocking `config/app.conf`
![Firewall blocking config](/images/thm_padelify_firewall_blokkeert_config.png)

**Summary:** Attempt to directly access a sensitive configuration file.  
**Role / Tools:** Manual browsing.  
**Result:** Access blocked by WAF keyword filtering.  
**Sensitivity:** Medium. 

---

## 9) Logs — XSS & config path leakage
![Logs error](/images/thm_padelify_logs_error.png)

**Summary:** Application error logs exposing XSS attempts and possible admin information.  
**Role / Tools:** Log file analysis.  
**Result:** Discovery of possible admin information in `/config/app.conf` and evidence of stored XSS.  
**Sensitivity:** High.

---

## 10) Blind XSS — image payload
![Payload image](/images/thm_padelify_payload_img.png)

**Summary:** Injected a simple HTML image payload to test for blind XSS.  
**Role / Tools:** XSS testing, payload crafting.  
**Result:** Payload stored and later executed by a privileged user.  
**Sensitivity:** High.

---

## 11) Python HTTP server — callback received
![Python server](/images/thm_padelify_python_server.png)

**Summary:** Incoming HTTP request received from the victim browser.  
**Role / Tools:** Listener (`python3 -m http.server`).  
**Result:** Confirmation that the blind XSS payload was successfully executed.  
**Sensitivity:** High.

---

## 12) Cookie exfiltration blocked
![Cookie not working](/images/thm_padelify_cookie_werkt_niet.png)

**Summary:** Initial attempt to steal session cookies using `document.cookie`.  
**Role / Tools:** JavaScript injection.  
**Result:** Payload blocked by WAF signature filtering.  
**Sensitivity:** Medium.

---

## 13) Cookie bypass — string concatenation
![Cookie works](/images/thm_padelify_co+okie_werkt_wel.png)

**Summary:** Modified payload using `document["coo"+"kie"]` to bypass WAF.  
**Role / Tools:** WAF evasion, JavaScript obfuscation.  
**Result:** Filter bypass successful; able to exfiltrate session data.  
**Sensitivity:** High.

---

## 14) Cookie received
![Cookie received](/images/thm_padelify_cookie_ontvangen.png)

**Summary:** Moderator session cookie successfully captured via blind XSS.  
**Role / Tools:** Blind XSS, exfiltration listener.  
**Result:** Valid session token obtained for account takeover.  
**Sensitivity:** Very High.

---

## 15) Editing cookie — privilege escalation
![Editing cookie](/images/thm_padelify_cookie_aanpassen.png)

**Summary:** Manual replacement of local cookie with the stolen session token.  
**Role / Tools:** Browser DevTools.  
**Result:** Logged in as moderator without credentials, achieving privilege escalation.  
**Sensitivity:** Very High.

---

## 16) Moderator flag obtained
![Moderator flag](/images/thm_padelify_moderator_flag.png)

**Summary:** Flag visible after successful moderator session hijack.  
**Role / Tools:** Session hijacking.  
**Result:** Moderator privileges confirmed and first flag retrieved.  
**Sensitivity:** Medium. 

---

## 17) Live page — LFI attack surface
![Live page](/images/thm_padelify_live_page.png)

**Summary:** Discovery of a dynamic file inclusion parameter: `live.php?page=`.  
**Role / Tools:** Manual browsing, application analysis.  
**Result:** Potential Local File Inclusion (LFI) attack surface identified.  
**Sensitivity:** Medium.

---

## 18) Gobuster fuzz — parameter discovery
![Gobuster fuzz page](/images/thm_padelify_gobuster_fuzz_page.png)

**Summary:** Fuzzing the `page` parameter to discover accessible files and directories.  
**Role / Tools:** Parameter fuzzing (`gobuster fuzz`).  
**Result:** Internal directories such as `config` and `logs` discovered.  
**Sensitivity:** High.

---

## 19) CyberChef — URL encoding bypass
![CyberChef encoding](/images/thm_padelify_cyberchef_encoding.png)

**Summary:** URL-encoded a blocked file path to evade WAF filtering.  
**Role / Tools:** Encoding (`CyberChef`).  
**Result:** WAF bypass technique prepared for LFI exploitation.  
**Sensitivity:** Medium.

---

## 20) Admin credentials discovered
![Admin credentials found](/images/thm_padelify_admin_creds_gevonden.png)

**Summary:** Sensitive administrator credentials leaked via Local File Inclusion.  
**Role / Tools:** LFI exploitation.  
**Result:** Admin password successfully recovered.  
**Sensitivity:** Very High.


---

## 21) Logging in as admin
![Logged in as admin](/images/thm_padelify_inloggen_als_admin.png)

**Summary:** Authentication attempt using the recovered administrator credentials.  
**Role / Tools:** Browser.  
**Result:** Successful login as admin, granting full administrative access.  
**Sensitivity:** High.

---

## 22) Admin flag found
![Admin flag found](/images/thm_padelify_admin_flag_gevonden.png)

**Summary:** Final flag visible after logging in with administrator privileges.  
**Role / Tools:** Privileged access.  
**Result:** Full system compromise confirmed.  
**Sensitivity:** Medium.

