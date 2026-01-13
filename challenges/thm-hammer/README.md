# TryHackMe – Hammer (Medium)

This repository contains my full write-up and supporting material for the **TryHackMe: Hammer** room (difficulty: Medium).  
The challenge focuses on abusing weaknesses in **authentication logic**, **insecure password‑reset mechanisms**, and **JWT token handling**, eventually leading to **admin privilege escalation** and **remote code execution**.

---

## 📁 Files in This Directory

- **writeup.md**  
  Complete step‑by‑step write-up of the exploitation chain, including enumeration, PIN brute‑forcing, JWT forgery, and RCE.

- **screenshots.md**  
  Supporting screenshots such as ffuf results, leaked logs, crafted JWTs, Burp Suite output, and command‑execution proof.

- **README.md**  
  This file — a high‑level overview of the challenge and repository structure.

---

## 🧠 What You’ll Learn

- Targeted **directory and file enumeration**
- Identifying and exploiting **sensitive log exposure**
- Abusing **insecure password reset flows**
- Performing **PIN brute‑force attacks** with custom scripting
- Understanding and forging **JSON Web Tokens (JWT)**
- Escalating privileges to **admin**
- Achieving **remote code execution** via command injection

---

## 🗺️ High-Level Walkthrough

A full detailed technical write-up is available in `/writeup/hammer_writeup.md`.  
Below is a brief overview of the exploitation path:

1. 🔎 Enumerating directories using `ffuf`  
2. 📓 Accessing exposed log files containing sensitive session information  
3. 🔐 Abusing the 4‑digit PIN password‑reset mechanism using a Python brute‑forcer  
4. 🪪 Extracting and analyzing JWT tokens from the application  
5. 🧵 Recovering the hardcoded JWT secret from server-side code  
6. 🛠️ Forging a new admin‑level JWT to gain privileged access  
7. 💥 Triggering command injection to obtain **remote code execution**  
8. 🎯 Retrieving the final flag from the compromised host

---

## ⚙️ Tools Used

- **Nmap** — initial recon  
- **ffuf** — directory and file enumeration  
- **Burp Suite** — token analysis & request replay  
- **Python 3** — PIN brute‑forcer & helper scripts  
- **jwt.io** — decoding and inspecting JWTs  
- **Linux CLI utilities** — curl, grep, etc.

---

## 🧪 Included Scripts

- a **Python brute‑forcer** for the 4‑digit PIN reset mechanism  
- small utilities for analyzing and forging JWT tokens  

These scripts support the exploitation process described in the write-up.

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes** and should only be used within legal, authorized environments such as TryHackMe.  
Never perform these techniques on systems without explicit permission.

---

## 🔗 Useful Links

- [**TryHackMe Room**](https://tryhackme.com/room/hammer)
- [**Write-up**](../../challenges/thm-hammer/writeup.md)
- [**Screenshots**](../../challenges/thm-hammer/screenshots.md)

---
