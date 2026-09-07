# TryHackMe-Recruit-Writeup
Detailed write-up for the TryHackMe Recruit Web Challenge, covering reconnaissance, enumeration, SSRF/local file access, credential disclosure, SQL injection, and privilege escalation.

# TryHackMe - Recruit Writeup

## Overview

Recruit is a web security challenge from TryHackMe focused on identifying and exploiting vulnerabilities in a web application.

In this challenge, the attack chain involved:

- Reconnaissance
- Web Enumeration
- Information Disclosure
- SSRF / Local File Access
- Credential Disclosure
- SQL Injection
- Credential Extraction
- Administrative Access

---

## Attack Chain

Nmap
   ↓
Web Enumeration
   ↓
mail.log
   ↓
SSRF / Local File Access
   ↓
config.php
   ↓
HR Credentials
   ↓
HR Login
   ↓
SQL Injection
   ↓
users Table
   ↓
Admin Credentials
   ↓
Admin Access
   ↓
Flag

---

## 1. Reconnaissance

### Nmap

```bash
nmap -sC -sV <TARGET-IP>
```
![Nmap Scan](images/nmap.png)

The scan was performed to identify open ports and running services.
Findings
- Port 22 - SSH
- Port 53 - DNS
- Port 80 - HTTP
The web server on port 80 became the primary attack surface.

The scan identified the available services running on the target. 
Port 80 exposed the web application, which became the primary attack surface.

##2. Web Enumeration

###I performed directory enumeration using ffuf.
```
ffuf -u http://<TARGET-IP>/FUZZ \
-w <WORDLIST>
```
![Nmap Scan](images/directory-enumeration.png)

Interesting endpoints discovered included:
- /assets
- /mail
- /javascript
- /phpmyadmin
- /server-status
The /mail directory was particularly interesting.


