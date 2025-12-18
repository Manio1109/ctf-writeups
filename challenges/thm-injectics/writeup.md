# 🛠️ TryHackMe: Injectics  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/injectics)

---

## ⚠️ Disclaimer

This write-up is intended **strictly for educational and portfolio purposes**.  
All actions described were performed in a **legal, controlled TryHackMe lab environment**.  
No techniques were used against real-world systems without authorization.

---

## 🧠 Overview

**Injectics** is a medium-difficulty web exploitation room focused on identifying and chaining classic web vulnerabilities into full system compromise.

The challenge walks through:
- **SQL Injection** for authentication bypass  
- **Abusing application recovery logic**  
- **Server-Side Template Injection (SSTI)**  
- Escalation to **Remote Code Execution (RCE)**  

The room highlights how seemingly small weaknesses (client-side filtering, exposed logs) can be chained into complete administrative and system-level access.

---

## 🧰 Tools & Techniques

| Tool / Technique | Purpose |
|------------------|---------|
| **Nmap** | Port scanning and service discovery |
| **Source code analysis** | Identifying exposed credentials and logic flaws |
| **Burp Suite** | Intercepting and modifying HTTP requests |
| **SQL Injection** | Authentication bypass & database manipulation |
| **SSTI (Twig)** | Template injection leading to RCE |

---

## 🚀 Attack Walkthrough

### 1. 🔎 Initial Recon — Port Scanning

```bash
nmap -sV -p- -T4 <target-ip>
```

