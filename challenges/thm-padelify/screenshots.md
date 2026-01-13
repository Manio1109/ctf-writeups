
## 1) Nmap scan — Basic network reconnaissance
![Nmap scan](/images/thm_padelify_nmap_scan.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

## 2) Initial visit — registration page (port 80)
![Registration page](/images/thm_padelify_registartiepagina.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Login page](/images/thm_padelify_inlogpagina.png)
beschrijving van afbeelding: dit is de login pagina wat komt na de registartie pagina
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Trying default payload](/images/thm_padelify_standaar_payload_proberen.png)
beschrijving afbeelding: hier probeer ik de ' OR 1=1 -- payload maar werkt niet
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Gobuster not working](/images/thm_padelify_gobuster_werkt_niet.png)
beschrijving afbeelding: hier kom ik erachter dat gobuster niet werkt door de firewall
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Curl firewall detected](/images/thm_padelify_curl_firewall_detected.png)
beschrijving afbeelding: hier probeer ik de website te curl maar krijg 403 forbidden firewall
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Gobuster working](/images/thm_padelify_gobuster_werkt.png)
beschrijving afbeelding: gobuster werkt met mozilla 5.0 user-agent
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Firewall blocking config](/images/thm_padelify_firewall_blokkeert_config.png)
beschrijving afbeeling: firewall blokkeert config/app.conf
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Logs error](/images/thm_padelify_logs_error.png)
beschrijving afbeelding: hier vind ik xss en admin informatie in de logs/error/log
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Payload image](/images/thm_padelify_payload_img.png)
beschrijving afbeelding: hier probeer ik een simpele img payload om reactie te krijgen op mijn http server
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Python server](/images/thm_padelify_python_server.png)
beschrijving afbeelding: ik krijg een reactie op mijn python http server 
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Cookie not working](/images/thm_padelify_cookie_werkt_niet.png)
beschrijving afbeelding: ik probeer de moderator cookie te stelen maar hier zie je dat ik geen reactie krijg op document.cookie 
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Cookie works](/images/thm_padelify_co+okie_werkt_wel.png)
beschrijving afbeelding: hier zie je dat ik document.coo+ckie probeer
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Cookie received](/images/thm_padelify_cookie_ontvangen.png)
beschrijving afbeelding: hier zie je dat ik een response krijg op mijn python http server met een cookie
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Editing cookie](/images/thm_padelify_cookie_aanpassen.png)
beschrijving afbeelding: ik pas de cookie aan in inspect en dan storage bij cookies
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Moderator flag](/images/thm_padelify_moderator_flag.png)
beschrijving afbeelding: Ik refresh de pagina en heb de 1e flag gepakt omdat ik ben ingelogd als moderator
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Live page](/images/thm_padelify_live_page.png)
beschrijving afbeelding: ik vind op de moderator pagina een live.php?page=mage die interresant is om te fuzzen
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Gobuster fuzz page](/images/thm_padelify_gobuster_fuzz_page.png)
beschrijving afbeeelding: hier gobuster fuzz ik page en vind bijvoorbeeld config
**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![CyberChef encoding](/images/thm_padelify_cyberchef_encoding.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Admin credentials found](/images/thm_padelify_admin_creds_gevonden.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Logged in as admin](/images/thm_padelify_inloggen_als_admin.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 

---

![Admin flag found](/images/thm_padelify_admin_flag_gevonden.png)

**Summary:** 
**Role / Tools:**
**Result:**  
**Sensitivity:** 
