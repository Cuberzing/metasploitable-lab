# 💀 Metasploitable 2 — Penetration Testing Lab

A full penetration test performed against Metasploitable 2, an intentionally vulnerable virtual machine, using Metasploit on Windows. This documents the methodology, exploitation, post-exploitation, and lessons learned.

---

## 🛠 Tools Used

- **Metasploit Framework** (running on Windows)
- **VirtualBox** — hosting Metasploitable 2 VM
- **Netcat** — manual shell access via backdoor port
- **John the Ripper** — offline password cracking
- **rockyou.txt** — wordlist for dictionary attack
- OS: Windows (host) / Linux (Metasploitable 2 target)

---

## 🎯 Objective

Simulate a real-world penetration test against a vulnerable machine. Gain initial access, escalate to root, extract credentials, and crack password hashes — documenting every step.

---

## 🖥 Lab Setup

| Component | Details |
|---|---|
| Host Machine | Windows PC |
| Hypervisor | VirtualBox |
| Target | Metasploitable 2 (Linux) |
| Network Mode | Host-only / NAT (isolated from internet) |
| Attacker Tool | Metasploit Framework on Windows |

Both machines ran on the same Windows host via VirtualBox, with networking configured so they could communicate while remaining isolated.

---

## 🔍 Phase 1 — Reconnaissance

Before exploiting anything, identified the target's open services:

```bash
nmap -A <target-ip>
```

Key finding: **vsftpd 2.3.4** running on port 21 — a version known to contain a backdoor vulnerability.

---

## 💥 Phase 2 — Exploitation (vsFTPd 2.3.4 Backdoor)

**CVE:** CVE-2011-2523

**What the vulnerability is:**
A malicious backdoor was introduced into the vsFTPd 2.3.4 source code. When a username containing `:)` is sent during login, the backdoor triggers and opens a root shell on port 6200 — no password required.

**Metasploit commands:**
```bash
msfconsole
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS <target-ip>
run
```

**Result:** Root Meterpreter session — full control of the target machine.

---

## 🔓 Phase 3 — Post Exploitation

### Backdoor Port Discovery
During enumeration, discovered **port 1524** open — a known Metasploitable backdoor that drops directly into a root shell via Netcat:

```bash
nc <target-ip> 1524
```

This gave an immediate root shell without any exploit — highlighting why unpatched services are dangerous.

### Password Hash Extraction
From the root shell, manually read the shadow file containing hashed passwords:

```bash
cat /etc/shadow
```

Extracted 7 password hashes and saved them locally for cracking.

### Hash Cracking with John the Ripper
```bash
john --wordlist=rockyou.txt hashes.txt
```

**Results — 2 out of 7 cracked:**

| Account | Password | Type |
|---|---|---|
| sys | batman | System account |
| service | service | Service account |

Weak, default, and reused passwords are extremely common in real environments. `batman` being a system account password is a perfect example of why password policies matter.

---

## ⚠️ Vulnerabilities Identified

| Vulnerability | Severity | Details |
|---|---|---|
| vsFTPd 2.3.4 Backdoor | Critical | Remote root access via malicious FTP version |
| Open Backdoor Port 1524 | Critical | Unauthenticated root shell via Netcat |
| Weak Password — sys | High | Password: "batman" |
| Weak Password — service | High | Password matches account name |

---

## 📚 What I Learned

- Full penetration testing methodology: recon → exploit → post-exploit
- How supply chain attacks work (backdoored software at the source)
- Why version detection during recon is critical
- Manual file reading as an alternative when automated tools fail
- How dictionary attacks work and why weak passwords fall instantly
- The difference between a Meterpreter session and a raw shell

---

## 🔗 Related Projects

- [Home Network Scan](../home-network-scan) — Nmap reconnaissance on a live network
- [Web Security Notes](../web-security-notes) — Burp Suite HTTP interception and analysis

---

*This lab was performed in an isolated virtual environment for educational purposes only. Never perform these techniques on systems you do not own or have explicit permission to test.*
