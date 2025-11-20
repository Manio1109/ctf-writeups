# 🛠️ TryHackMe: Include  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/include)

> **Disclaimer:**  
> This write-up is for educational and professional demonstration purposes only.  
> All testing was performed in a controlled, authorized environment (TryHackMe).  
> No techniques described here were used against real-world systems.

#### About This Write-Up
This document is part of my offensive security portfolio.  
My goal is to present structured, professional case studies that demonstrate practical penetration testing skills, clear reporting, and a real-world methodology.

These write-ups are meant to demonstrate practical offensive security skills relevant to junior pentesting, red teaming, and application security roles.

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

### 1. 🔎 Port Scan

```bash
nmap -sV -p- -T4 10.10.239.203
```

| Port      | Service               |
| --------- | --------------------- |
| 22/tcp    | SSH                   |
| 25/tcp    | SMTP                  |
| 110/tcp   | POP3                  |
| 143/tcp   | IMAP                  |
| 4000/tcp  | Node.js / Express app |
| 50000/tcp | Apache webserver      |

---

### 2. 👤 Logging into the Node.js App (port 4000)

The application accepted default credentials:
```yaml
guest : guest
```

The dashboard displayed user profiles, each containing an `isAdmin:false` attribute.
The `recommend` feature allowed sending JSON that seemed to influence profile data — a strong indicator of **Prototype Pollution**.

---

### 3. 🧩 Prototype Pollution → Admin Privilege Escalation

By crafting a manipulated JSON structure, it was possible to modify the user object on the server.  
Specifically, adding the field:

```json
{
  "isAdmin": true
}
```

Resulted in my profile being elevated to **admin privileges.**

With administrative access unlocked, the application revealed the **Admin Dashboard API documentation**, exposing internal-only API routes that became essential for the next stage of the attack.

#### Why the Prototype Pollution Worked
The application merged user‑supplied JSON into server‑side objects using a deep merge function without filtering out `__proto__` or inherited properties. As a result, adding `"isAdmin": true` modified the prototype of the user object, causing every new instance of that object to inherit the overridden `isAdmin` value.

---

### 4. 🔑 Internal Admin Dashboard API

With admin privileges enabled, the dashboard exposed the **Admin API documentation**, revealing several internal-only endpoints. These routes were not accessible externally but became visible through the elevated permissions.

Among the most interesting endpoints were:

- `http://127.0.0.1:5000/internal-api`  
  → revealed a `secretKey`
- `http://127.0.0.1:5000/getAllAdmins101099991`  
  → dumped admin usernames and passwords

Since these endpoints were bound to **localhost only**, direct access from the outside was impossible.  
However, the application included a feature that fetched remote URLs for the **banner image**, which opened the door to performing **Server-Side Request Forgery (SSRF)**.

By abusing the banner image URL field, I was able to craft requests to `127.0.0.1`, allowing me to retrieve:

✔️ base64-encoded application credentials  
✔️ system usernames and passwords  
✔️ sensitive configuration data

This information laid the groundwork for gaining full access across multiple services.

---

### 5. 🔓 Decoding Retrieved Credentials & Gaining Application Access

The internal Admin API returned several credential sets, but all of them were encoded in **base64**.  
To extract the usable login information, I decoded the strings using **CyberChef**.

The decoded output revealed the following credentials:

```json
{
  "ReviewAppUsername": "admin",
  "ReviewAppPassword": "[redacted]",
  "SysMonAppUsername": "administrator",
  "SysMonAppPassword": "[redacted]"
}
```

Using these credentials, I successfully logged into the **SysMonApp,** which immediately provided the first room flag.
```text
THM{REDACTED}
```

This confirmed that the SSRF exploitation path was valid and that the internal Admin API fully trusted incoming requests from the manipulated banner image URL.

---

### 6. 📂 Local File Inclusion (LFI) via `profile.php`

While reviewing the page source of the Apache service running on port **50000**, I noticed the following HTML snippet:

```html
<img src="profile.php?img=profile.png"/>
```

This suggested that the value of the `img` parameter was being passed directly to the server-side `profile.php` script — a strong indication of a potential **Local File Inclusion (LFI)** vulnerability.

Initial test request:
```arduino
http://10.10.239.203:50000/profile.php?img=profile.png
```

The script appeared to load files dynamically based on the provided input, making it a suitable target for LFI and **path traversal** techniques in the next phase.

---

### 7. 🔍 LFI Fuzzing with Burp Suite

To confirm and exploit the Local File Inclusion vulnerability in `profile.php`, I used **Burp Suite Intruder** to fuzz the `img` parameter with known LFI payloads.

**Target:**
```bash
/profile.php?img=FUZZ
```

**Payload source:**  
Common LFI/path traversal wordlists from **SecLists**

After testing multiple traversal sequences, a working payload was identified:
```bash
....//....//....//....//....//....//....//....//....//....//etc/passwd
```

This successfully returned the contents of `/etc/passwd`, confirming full LFI exploitation.

From this output, I was able to identify valid system users:

- `joshua`
- `charles`

These usernames became valuable in later stages for SSH access and brute forcing.

#### Why the Traversal Bypass Worked
The application sanitized `../` but did not filter `....//`.  
After filesystem normalization, `....//` resolves to `../`, allowing directory traversal despite the filter:
```bash
....// → ../
```
This made it possible to reach `/etc/passwd` and other system files.

---

### 8. 🛤️ Path Traversal Mechanics

The vulnerable `profile.php` script concatenated the `img` parameter directly into a file path without validation.  
By using repeated traversal sequences such as:

```bash
....//....//....//etc/passwd
```

it became possible to break out of the web directory and read arbitrary system files.

The application attempted to block `../`, but the alternative sequence `....//` bypassed the filter and normalized back into `../` at runtime.

---

### 9. 🔑 SSH Brute Force with Hydra

After extracting valid usernames from `/etc/passwd`, I proceeded with an SSH brute‑force attempt.

**Hydra command used:**
```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://10.10.239.203 -t 4
```

**Results:**
```pgsql
login: joshua  password: REDACTED
login: charles password: REDACTED
```

Both accounts reused extremely weak passwords, allowing successful SSH logins and granting full shell access to the machine.

#### Note on Brute Forcing
Brute forcing was performed only because:
- the room explicitly expects it,
- account lockout was disabled,
- it is part of a safe, controlled CTF environment.

In a real engagement, brute forcing would only be attempted with authorization and usually under strict rate‑limited conditions.

---

### 10. 🏁 Final Flag

Once inside the machine via SSH as **joshua**, I checked the web directory:

```bash
ls -la /var/www/html
```

Inside the directory, I found a file with a hash‑like name:
```bash
505eb0fb8a9f32853b4d955e1f9123ea.txt
```
Reading the file:
```bash
cat /var/www/html/505eb0fb8a9f32853b4d955e1f9123ea.txt
```

Retrieved the flag:
```bash
THM{REDACTED}
```

---

| Flag Type       | Description                        | Value                                   |
| --------------- | ---------------------------------- | --------------------------------------- |
| 🛠️ SSRF Secret | Internal API + decoded credentials | `THM{REDACTED}`             |
| 🔑 Hidden File  | Apache webroot flag                | `THM{REDACTED}` |

---

## Room links

📸 [screenshots](../../challenges/thm-include/screenshots.md)

🔗 [TryHackMe - include](https://tryhackme.com/room/include)

---

### 💭 Reflection

This room showcased how multiple seemingly small vulnerabilities can be chained into a full system compromise.  
Instead of relying on a single critical flaw, the challenge demonstrated the power of layered exploitation:

- Prototype Pollution escalating a standard user into an admin.
- Internal-only Admin APIs becoming reachable through SSRF abuse.
- LFI enabling enumeration of system users and filesystem structure.
- Weak password hygiene allowing successful SSH brute‑forcing.
- Misconfigurations in backups and permissions exposing sensitive data.

Each step reinforced the idea that even minor oversights, when combined, create a clear attack path.

---

### Key Takeaways

- **Prototype Pollution is more impactful than expected** — especially when it manipulates authorization logic.
- **SSRF is extremely dangerous** when applications trust localhost services or expose hidden internal endpoints.
- **LFI is still one of the most effective enumeration techniques**, particularly on misconfigured PHP setups.
- **Password reuse remains one of the most common real‑world mistakes**, even in multi‑user environments.
- **Backup files should be secured**; accessible archives often leak credentials or keys.

---

### Lessons Learned

- Authorization should never rely directly on user‑controlled JSON input structures.
- Localhost‑bound routes must be protected even from internal application components.
- Directory traversal protections should be strict and validated server‑side.
- SSH access should enforce strong password policies and avoid reuse across users.
- Defense in depth matters — one misconfiguration is rarely catastrophic, but several combined always are.

This room is a strong reminder that real‑world compromises often come from *chains* of misconfigurations, not single exploits.

