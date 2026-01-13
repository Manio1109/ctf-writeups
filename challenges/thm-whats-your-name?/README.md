# TryHackMe – What's Your Name? (Medium)

This repository contains my full write‑up and supporting material for the **TryHackMe: What's Your Name?** room (difficulty: Medium).  
The challenge focuses on chaining **client‑side vulnerabilities**, **weak session handling**, and **missing server‑side validation** to escalate privileges all the way from a normal user to **administrator**.

---

## 📁 Files in This Directory

- **writeup.md**  
  Complete, detailed write‑up of the entire attack chain — from recon to XSS exploitation, CSRF‑style privilege escalation, and final admin compromise.

- **screenshots.md**  
  All screenshots taken during the challenge to support the write‑up (payload execution, cookie exfiltration, moderator/admin dashboards, etc.).

- **README.md**  
  This file — high‑level summary of the challenge structure and what you’ll learn.

---

## 🧠 What You’ll Learn

- Identifying and exploiting **Stored XSS**  
- Executing **DOM‑based XSS** through vulnerable UI components  
- Performing **session hijacking** via insecure cookie attributes  
- Abusing **CSRF‑like weaknesses** to force privileged actions  
- Understanding **defense‑in‑depth failures** in modern web apps  
- Chaining multiple "low impact" bugs into a full **admin takeover**

---

## 🗺️ High‑Level Walkthrough

A full technical write‑up is available in `/writeup/whats_your_name_writeup.md`.  
Below is a short overview of the exploitation path:

1. 🔎 Initial recon using `nmap` on ports 22, 80, and 8081  
2. 💣 Stored XSS in profile field → moderator cookie exfiltration  
3. 🔁 Session hijacking using the stolen `PHPSESSID` → **Flag 1**  
4. 🤖 DOM‑XSS in chatbot due to unsafe `innerHTML` usage  
5. 🎯 Forced POST request (CSRF‑style) to backend `/change_password.php`  
6. 🔐 Overwriting admin password  
7. 🚪 Logging in as admin and retrieving **Flag 2**

---

## ⚙️ Tools Used

- **Nmap** — port & service enumeration  
- **Browser DevTools** — cookie modification & DOM inspection  
- **Netcat / simple HTTP listeners** — cookie exfiltration  
- **Burp Suite** — request analysis  
- **JavaScript payloads** — XSS, forced POST requests  
- **Python (optional)** — custom listeners or small helpers  

---

## 🧪 Included Scripts

- XSS cookie‑exfiltration snippet  
- JavaScript‑based forced POST actions for privilege escalation  

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes** and should only be used within authorized environments such as TryHackMe.  
Never execute these techniques on systems you do not own or without explicit permission.

---

## 🔗 Useful Links

- [**TryHackMe Room**](https://tryhackme.com/room/whatsyourname)  
- [**Write‑up**](../../challenges/thm-whats-your-name/writeup.md)  
- [**Screenshots**](../../challenges/thm-whats-your-name/screenshots.md)

---

