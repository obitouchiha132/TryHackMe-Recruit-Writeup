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

## 2. Web Enumeration

### I performed directory enumeration using ffuf.
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

## 3. Information Disclosure

### Mail Log

The `/mail/mail.log` file was publicly accessible and exposed internal deployment information.

![Nmap Scan](images/mail-log.png)

The mail log revealed:

- The **HR username is `hr`**.
- HR login credentials were temporarily stored in the application's **`config.php`** file.
- Administrator credentials were **not stored in application files** and were maintained in the backend database.
- The log also confirmed that the **API documentation page was live**.

This disclosure provided the next step in the attack chain: obtaining the HR credentials from `config.php`.

![Mail Log](images/mail-log_2.png)

## 4.API Documentation Discovery

After reviewing the exposed mail.log, I checked the application's API documentation to understand how the recruitment portal handles candidate CVs.
The API documentation was available at:`/api.php`

The documentation revealed an endpoint for fetching candidate CVs:
```
/file.php?cv=<URL>
```
The important part here is that the application accepts a URL as user-controlled input through the cv parameter. This suggested that the server may be making requests to the supplied resource on behalf of the user.
This became an interesting attack surface because the functionality could potentially be abused to make the server access resources that should not be directly accessible.

![Mail Log](images/api.png)

Next: I tested the cv parameter to determine whether it could be abused for SSRF / local file access.

## 5. SSRF / Local File Access

### The application contained a file endpoint:

![Mail Log](images/file-endpoint.png)

```
/file.php?cv=
```
I first tested:
```
/file.php?cv=config.php
```
The application responded:
Only local files are allowed
This indicated that the parameter was being processed server-side.

I then tested:
```
/file.php?cv=file://config.php
```
The application returned the PHP source code of `config.php`.
This exposed sensitive application configuration and credentials.

![Mail Log](images/config-file.png)

The configuration file contained temporary HR credentials, including the HR password. This was an Information Disclosure / Credential Disclosure vulnerability.
```php
$HR_PASSWORD = (hrpassword123);
```

## 5. HR Login

The credentials obtained from the exposed `config.php` file were used to log in to the HR account successfully.

**Username:** `hr`  
**Password:** `hrpassword123`

![HR Login](images/hr-login.png)


