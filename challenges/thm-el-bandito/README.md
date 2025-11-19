# TryHackMe – El Bandito (Hard)

This repository contains my full write-up and supporting material for the **TryHackMe: El Bandito** room (difficulty: Hard).  
The challenge involves chaining several advanced web exploitation techniques, including **SSRF abuse through WebSocket handling** and a rare **HTTP/2 → HTTP/1.1 (H2.CL) request smuggling attack**.

---

## 📁 Files in This Directory

- **writeup.md**  
  Full detailed write-up of the entire exploitation chain, including payloads, methodology, and analysis.

- **screenshots.md**  
  Screenshots taken during the challenge to support the write-up (Burp Suite traffic, SSRF responses, smuggling output, etc.).

- **README.md**  
  This file — high‑level summary of the challenge and structure.

---

## 🧠 What You’ll Learn

- Advanced **SSRF exploitation**
- **WebSocket upgrade abuse** and proxy misdirection
- Techniques for bypassing internal network restrictions
- Understanding **HTTP/2 to HTTP/1.1 downgrade behavior**
- Performing **H2.CL request smuggling**
- Recognizing weaknesses in reverse proxy and caching layers (e.g., Varnish)

---

## 🗺️ High-Level Walkthrough

A full technical write-up is available in `/writeup/el_bandito_writeup.md`.  
Below is a short overview of the exploitation path:

1. 🔎 Recon using `nmap` and directory enumeration  
2. 🔍 Analysis of client-side JavaScript and WebSocket structure  
3. 📡 Abuse of `/isOnline` endpoint for SSRF  
4. 🧩 Triggering a fake WebSocket upgrade via a custom `101 Switching Protocols` HTTP server  
5. 🔐 Extracting internal admin credentials → **Flag 1**  
6. 🔀 Testing HTTP/2 messages for downgrade behavior  
7. 💥 Performing an **H2.CL desync attack**  
8. 🎯 Extracting the second flag through smuggled backend requests

---

## ⚙️ Tools Used

- **Nmap** — port and service enumeration  
- **Gobuster** — folder & endpoint brute-forcing  
- **Burp Suite (with HTTP/2 support)** — interception, request manipulation, smuggling  
- **Python 3** — for the custom 101-switch server  
- **Browser DevTools** — client-side logic analysis  

---

## 🧪 Included Scripts

The `scripts/` folder contains helper scripts used during the exploitation.  
Example: a minimal server returning a `101 Switching Protocols` response, used to fool the backend proxy during SSRF testing.

---

## ⚠️ Disclaimer

This repository is intended **strictly for educational purposes** and should only be used within legal, authorized environments such as TryHackMe.  
Never perform these techniques on systems without explicit permission.

---

## 🔗 Useful Links

- **TryHackMe Room:** https://tryhackme.com/room/elbandito  
- **Write-up:** `/challenges/thm-el-bandito/writeup.md`  
- **Screenshots:** `/challenges/thm-el-bandito/screenshots.md`

---

