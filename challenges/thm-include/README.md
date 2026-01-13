# TryHackMe – Include (Medium)

This repository contains my full write-up and supporting material for the **TryHackMe: Include** room (difficulty: Medium).  
The challenge revolves around a multi‑stage exploitation chain including **Prototype Pollution**, **SSRF**, **Local File Inclusion**, and credential harvesting leading to **SSH compromise**.

---

## 📁 Files in This Directory

- **writeup.md**  
  Full detailed walk-through of the entire attack chain, including payloads, analysis, screenshots, and methodology.

- **screenshots.md**  
  Supporting screenshots taken during the challenge (Burp Suite traffic, SSRF responses, LFI output, Hydra results, etc.).

- **README.md**  
  This file — a high‑level summary of the challenge and the repository structure.

---

## 🧠 What You’ll Learn

- How Prototype Pollution can escalate privileges inside Node.js applications  
- Abusing **SSRF** to interact with internal‑only admin APIs  
- Performing **Local File Inclusion** using PHP path traversal  
- Identifying and decoding leaked credentials  
- Conducting controlled **SSH brute forcing** in a CTF environment  
- Chaining medium‑impact vulnerabilities into a full system compromise  

---

## 🗺️ High-Level Walkthrough

A full technical write-up is available in `/writeup/include_writeup.md`.  
Below is a short overview of the exploitation path:

1. 🔎 Reconnaissance using `nmap` across ports 22–50000  
2. 👤 Logging in using default credentials (`guest:guest`)  
3. 🧩 Privilege escalation via Prototype Pollution (`isAdmin:true`)  
4. 🌐 Abusing the banner-image feature for **SSRF** to reach `127.0.0.1` admin APIs  
5. 🔐 Extracting and decoding base64 internal credentials → **Flag 1**  
6. 📂 Discovering `profile.php` and exploiting **LFI** through traversal bypass  
7. 📝 Enumerating system users from `/etc/passwd`  
8. 🔑 Gaining shell access via **Hydra SSH brute forcing**  
9. 🏁 Retrieving the final hidden flag from the Apache webroot  

---

## ⚙️ Tools Used

- **Nmap** — port & service enumeration  
- **Burp Suite** — request manipulation, prototype pollution testing, and Intruder LFI fuzzing  
- **CyberChef** — decoding leaked base64 credentials  
- **Hydra** — SSH brute forcing  
- **SecLists** — LFI/traversal payload lists  
- **Linux standard utilities** — system inspection after gaining shell access  

---

## 🧪 Included Scripts / Payloads

- Useful LFI payloads used during fuzzing  
- Sample SSRF requests for internal API probing  
- Wordlists or user lists used during SSH brute forcing  

(These are provided for educational purposes and to reproduce the methodology.)

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes** and should only be used within legal, authorized environments such as TryHackMe.  
Never perform these techniques on systems without explicit permission.

---

## 🔗 Useful Links

- [**TryHackMe Room**](https://tryhackme.com/room/include)  
- [**Write-up**](../../challenges/thm-include/writeup.md)  
- [**Screenshots**](../../challenges/thm-include/screenshots.md)

---

