# Silentium – Hack The Box Writeup

## 🧠 Introduction

Silentium is a challenging Hack The Box machine focused on web exploitation, API abuse, internal service pivoting, and privilege escalation.

The attack chain involves:

- Password reset token disclosure leading to account takeover  
- API key exposure enabling interaction with a Flowise AI instance  
- Remote Code Execution via Flowise Custom MCP  
- Internal service discovery and pivoting via SSH tunneling  
- Privilege escalation through a vulnerable GOGS instance  

---

## 🎯 Target


IP: 10.129.21.246
Domains: silentium.htb / staging.silentium.htb


---

## 🔎 Enumeration

Initial port scan:

```bash
nmap -sC -sV 10.129.21.246
Open ports:
22/tcp – OpenSSH
80/tcp – nginx (redirects to silentium.htb)

The main website presented a corporate landing page with no direct attack surface.

However, user enumeration revealed:

User: Ben
Email pattern: ben@silentium.htb
🌐 Subdomain Discovery

Virtual host enumeration revealed an additional service:

gobuster vhost -u http://silentium.htb -w <wordlist> --append-domain
Found:
staging.silentium.htb
⚙️ Staging Application – Flowise AI

The staging environment exposed a Flowise AI platform, used for building LLM workflows.

Key functionality identified:

Password reset endpoint
🔓 Password Reset Token Disclosure (Account Takeover)

The password reset functionality was vulnerable and returned a temporary token in the HTTP response.

Exploitation flow:
Request password reset for a valid user
Server returns reset token in response
Token used to set a new password
Account takeover achieved
👤 Account Access

After resetting the password, login was successful.

Inside the user dashboard:

An API key was exposed, allowing interaction with internal APIs
💥 Remote Code Execution – Flowise Custom MCP

A vulnerable feature in Flowise Custom MCP allowed remote code execution via crafted API requests.

Steps:
Reverse shell listener started locally
Exploit executed using API key
Reverse shell obtained from target
🧪 Container Enumeration

Inside the compromised environment, sensitive configuration data was discovered via environment variables:

SMTP configuration (credentials redacted)
JWT secrets (redacted)
Internal service configuration

One of the credentials was reused for system access.

🔑 SSH Access

Reused credentials allowed SSH login:

ssh ben@silentium.htb

User flag was obtained.

🧭 Local Enumeration

Local services discovered:

127.0.0.1:3000
127.0.0.1:3001
127.0.0.1:1025
127.0.0.1:8025

SSH tunneling was used to access internal services:

ssh -L 3001:127.0.0.1:3001 ben@silentium.htb
🛠️ Internal Service – GOGS

A local GOGS instance was discovered on port 3001.

This service was vulnerable to remote code execution via repository manipulation.

🚀 Privilege Escalation

The exploit chain resulted in execution as root, achieving full system compromise.

uid=0(root)

Root flag obtained successfully.

📌 Key Takeaways
Password reset flows must never expose tokens in responses
API key exposure can significantly escalate impact
Internal services are often high-value attack surfaces
SSH tunneling is critical for pivoting
Credential reuse increases compromise impact
DevOps tools like GOGS can introduce serious risks if misconfigured
🧾 Conclusion

Silentium demonstrates a full attack chain:

Web vulnerability → Account takeover → API abuse → RCE → Internal pivot → Root

A realistic scenario showing how multiple low/medium severity issues can combine into full system compromise.
