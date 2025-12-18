# 🛠️ TryHackMe: Injectics (Medium)

This repository contains my full write-up and supporting material for the  
**TryHackMe: Injectics** room (difficulty: Medium).

The challenge focuses on chaining classic web vulnerabilities — starting with **SQL Injection**, moving into **Server-Side Template Injection (SSTI)**, and ultimately achieving **Remote Code Execution (RCE)** through Twig template abuse.

---

## 📁 Files in This Directory

- **writeup.md**  
  Full technical write-up describing the complete exploitation path, payloads, and reasoning.

- **screenshots.md**  
  Screenshots taken during the room (source code findings, Burp Suite payloads, SSTI output, flags).

- **README.md**  
  This file — high-level overview and structure of the Injectics challenge.

---

## 🧠 What You’ll Learn

- Identifying and exploiting **SQL Injection** vulnerabilities
- Bypassing weak **client-side input filtering**
- Leveraging exposed logs and source code artifacts
- Detecting and exploiting **Server-Side Template Injection (Twig)**
- Escalating SSTI to full **Remote Code Execution**
- Chaining multiple web vulnerabilities into a complete attack path

---

## 🗺️ High-Level Walkthrough

A full technical breakdown is available in `/writeup/injectics_writeup.md`.  
Below is a condensed overview of the exploitation chain:

1. 🔎 Network reconnaissance using `nmap`
2. 📂 Source code review revealing `mail.log` with default credentials
3. 🎭 Bypassing client-side SQLi filters using Burp Suite
4. 💣 Exploiting SQL Injection to manipulate backend behavior
5. 🔑 Logging into the admin panel → **Flag 1**
6. 🧩 Identifying **SSTI** via Twig template evaluation
7. 🖥️ Escalating SSTI to **Remote Code Execution**
8. 🏁 Reading files from the `flags` directory → **Flag 2**

---

## ⚙️ Tools Used

- **Nmap** — port scanning and service discovery  
- **Burp Suite** — request interception, payload crafting, filter bypass  
- **Browser DevTools** — client-side JavaScript analysis  
- **Source code review** — identifying insecure logic and exposed logs  
- **Twig SSTI payloads** — achieving command execution  

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes**.  
All testing was performed in a controlled, authorized environment provided by TryHackMe.  

❗ Never attempt these techniques on systems you do not own or have explicit permission to test.

---

## 🔗 Useful Links

- [**TryHackMe Room – Injectics**](https://tryhackme.com/room/injectics)
- [**Write-up**](../../challenges/thm-injectics/writeup.md)
- [**Screenshots**](../../challenges/thm-injectics/screenshots.md)

---

