# TryHackMe – Padelify (Medium)

This repository contains my full write-up and supporting material for the **TryHackMe: Padelify** room (difficulty: Medium).  
The challenge focuses on practical web exploitation techniques including **WAF bypass**, **blind XSS exploitation**, **session hijacking**, and **Local File Inclusion (LFI)** through encoding evasion.

---

## 📁 Files in This Directory

- **writeup.md**  
  Full detailed write-up of the entire exploitation chain, including payloads, methodology, and technical analysis.

- **screenshots.md**  
  Screenshots taken during the challenge to support the write-up (WAF behavior, XSS callbacks, cookie theft, LFI output, etc.).

- **README.md**  
  This file — high‑level summary of the challenge and structure.

---

## 🧠 What You’ll Learn

- **Web Application Firewall (WAF) bypass techniques**
- Exploiting **blind stored XSS**
- **Session hijacking** via cookie exfiltration
- Payload obfuscation to defeat signature-based filters
- **Local File Inclusion (LFI)** exploitation
- Using **URL encoding** to bypass security controls
- Understanding security control parsing discrepancies

---

## 🗺️ High-Level Walkthrough

A full technical write-up is available in `/writeup/padelify_writeup.md`.  
Below is a short overview of the exploitation path:

1. 🔎 Recon using `nmap` and directory enumeration  
2. 🛡️ Identifying and bypassing a signature-based WAF  
3. 🧪 Testing input filtering and payload restrictions  
4. 💉 Injecting a **blind XSS payload**  
5. 📡 Receiving callbacks from a moderator browser  
6. 🍪 Stealing a session cookie using JavaScript obfuscation  
7. 🔓 Hijacking the moderator session → **Flag 1**  
8. 📂 Discovering dynamic file inclusion (`live.php?page=`)  
9. 🧬 Fuzzing parameters to identify internal directories  
10. 🔐 Bypassing WAF with URL encoding  
11. 📜 Reading sensitive config files via LFI  
12. 👑 Logging in as admin → **Flag 2**

---

## ⚙️ Tools Used

- **Nmap** — port and service enumeration  
- **Gobuster** — directory and parameter fuzzing  
- **Curl** — manual HTTP testing  
- **Python 3** — HTTP listener for XSS callbacks  
- **CyberChef** — URL encoding & payload manipulation  
- **Browser DevTools** — cookie editing and session takeover  

---

## 🧪 Included Scripts

The `scripts/` folder contains helper tools used during exploitation.  
Example: a simple Python HTTP server to capture blind XSS callbacks and stolen cookies.

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes** and should only be used within legal, authorized environments such as TryHackMe.  
Never perform these techniques on systems without explicit permission.

---

## 🔗 Useful Links

- [**TryHackMe Room**](https://tryhackme.com/room/padelify)
- [**Write-up**](../../challenges/thm-padelify/writeup.md) 
- [**Screenshots**](../../challenges/thm-padelify/screenshots.md)

