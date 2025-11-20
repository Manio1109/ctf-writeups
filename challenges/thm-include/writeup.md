# 🛠️ TryHackMe: Include  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/include)

---

## 🧠 Summary  
**Include** is a medium-difficulty web exploitation room focused on chaining multiple vulnerabilities in a Node.js + PHP environment.  
This challenge requires combining **Prototype Pollution**, **SSRF**, **LFI/Path Traversal**, and finally **SSH brute force** to gain full system access.

**Core techniques demonstrated:**
- 🧩 **Prototype Pollution → privilege escalation**  
- 🌐 **SSRF to reach internal-only APIs**  
- 📂 **Local File Inclusion & Path Traversal**  
- 🔑 **Credential decoding & SSH brute forcing**

This write-up is part of my offensive security portfolio and follows a clear, structured pentesting methodology similar to my *El Bandito* documentation.

---

## 🧰 Tools & Techniques

| ⚙️ Tool / Technique     | 📌 Purpose |
|-------------------------|------------|
| **Nmap**               | Network & port scanning |
| **Burp Suite**         | Interception, request tampering, fuzzing |
| **Prototype Pollution** | Modify user objects (`isAdmin:true`) |
| **SSRF**               | Access internal APIs via banner image upload |
| **CyberChef**          | Base64 decoding of leaked credentials |
| **Source Code Review** | Discover hidden PHP files & vulnerable endpoints |
| **LFI / Path Traversal** | Read local system files (`/etc/passwd`) |
| **Hydra**              | SSH brute force using RockYou |

---

## 🚀 Attack Path

---

### 1. 🔎 Port Scan

```bash
nmap -sV -p- -T4 10.10.239.203
```
