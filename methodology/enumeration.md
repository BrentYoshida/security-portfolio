# Enumeration Checklist
**Status:** Living document — updated as methodology develops  
**Last Updated:** April 2026  

---

## Philosophy

Enumeration is where engagements are won or lost. The vulnerability is almost always discovered during this phase — exploitation is just acting on what enumeration revealed. Rushing to exploitation before enumeration is complete is the most common reason people get stuck on machines and miss findings on real engagements.

Run every applicable section before moving to Phase 3. Check each box before declaring enumeration complete.

---

## Phase 1 — Initial Connectivity

```bash
# Confirm host is live before running anything else
ping -c 4 [IP]
```

- [ ] Host responds to ping
- [ ] Target IP confirmed correct
- [ ] VPN connected (HTB/lab environments)

---

## Phase 2 — Port Scanning

### Step 1 — Quick initial scan

```bash
nmap [IP]
```

- [ ] Initial scan complete
- [ ] Open ports noted

### Step 2 — Full port scan (ALWAYS run this)

```bash
nmap -p- [IP]
```

- [ ] Full port scan complete
- [ ] Non-standard ports checked against initial scan
- [ ] Any new ports discovered noted

### Step 3 — Service version and script scan on discovered ports

```bash
nmap -sV -sC -O -p [discovered ports] [IP]
```

- [ ] Service versions identified for all open ports
- [ ] OS detected or inferred
- [ ] Default scripts run
- [ ] Output saved

```bash
# Always save scan output
nmap -sV -sC -oA recon/nmap_[target] [IP]
```

### Step 4 — UDP scan

```bash
sudo nmap -sU --top-ports 100 [IP]
```

- [ ] UDP scan complete
- [ ] Any UDP services noted (DNS/53, SNMP/161, TFTP/69, NTP/123)

---

## Phase 3 — Service Enumeration

Work through every discovered service. Don't skip services that look uninteresting — context from one service frequently makes another exploitable.

### Web (HTTP/HTTPS — ports 80, 443, 8080, 8443)

```bash
# Technology fingerprinting
whatweb http://[IP]

# Directory brute force
gobuster dir -u http://[IP] \
  -w /usr/share/seclists/Discovery/Web-Content/common.txt \
  -x php,html,txt,bak,xml,conf \
  -o recon/gobuster.txt

# Extended wordlist if common.txt returns nothing useful
gobuster dir -u http://[IP] \
  -w /usr/share/seclists/Discovery/Web-Content/big.txt \
  -x php,html,txt,bak

# Virtual host enumeration
gobuster vhost -u http://[domain] \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Vulnerability scan (noisy — use carefully)
nikto -h http://[IP]
```

- [ ] Technology stack identified
- [ ] Directories enumerated
- [ ] Extensions tested (.php, .bak, .conf, .txt, .xml)
- [ ] Virtual hosts checked
- [ ] Login pages identified
- [ ] Input fields noted for injection testing
- [ ] Source code reviewed manually in browser
- [ ] Cookies and headers inspected in Burp Suite
- [ ] Robots.txt and sitemap.xml checked

```bash
curl http://[IP]/robots.txt
curl http://[IP]/sitemap.xml
```

### FTP (port 21)

```bash
# Anonymous login attempt first
ftp [IP]
# Username: anonymous  Password: (blank or anonymous)

ftp> ls -la
ftp> pwd
ftp> get [filename]
```

- [ ] Anonymous login attempted
- [ ] Directory listing obtained
- [ ] All accessible files downloaded and reviewed
- [ ] Write access tested

```bash
ftp> put test.txt
```

### SMB (ports 139, 445)

```bash
# List shares — null session
smbclient -L [IP] -N

# Full enumeration
enum4linux -a [IP]

# Permission mapping
smbmap -H [IP]

# Connect to each accessible share
smbclient \\\\[IP]\\[ShareName] -N
```

Inside each share:
```bash
smb: \> ls
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

- [ ] All shares listed
- [ ] Null session attempted on each share
- [ ] Guest access attempted
- [ ] All accessible files downloaded
- [ ] Write access tested on each share

### SSH (port 22)

```bash
# Version check
ssh -v [user]@[IP]

# Note supported authentication methods
# Check for username enumeration vulnerability if older version
```

- [ ] Version noted
- [ ] Any known CVEs for version checked on searchsploit
- [ ] Default credentials attempted if applicable context exists

### Redis (port 6379)

```bash
redis-cli -h [IP] ping
redis-cli -h [IP] info
redis-cli -h [IP] info keyspace
redis-cli -h [IP] select 0
redis-cli -h [IP] keys *
redis-cli -h [IP] get [key]
```

- [ ] Unauthenticated access confirmed or denied
- [ ] All databases enumerated
- [ ] All keys retrieved and reviewed

### SMTP (port 25)

```bash
# Banner grab
nc [IP] 25

# User enumeration
smtp-user-enum -M VRFY -U /usr/share/seclists/Usernames/top-usernames-shortlist.txt -t [IP]
```

- [ ] Banner captured — software and version noted
- [ ] User enumeration attempted

### SNMP (port 161 UDP)

```bash
snmpwalk -c public -v1 [IP]
snmpwalk -c private -v1 [IP]
```

- [ ] Community strings attempted (public, private)
- [ ] Any returned data reviewed for credentials or system info

### Other Services

For any service not covered above:

```bash
# Banner grab — works on almost any TCP service
nc [IP] [port]

# Searchsploit for identified version
searchsploit [service] [version]
```

- [ ] Banner captured
- [ ] Searchsploit run on service and version
- [ ] Default credentials attempted where applicable

---

## Phase 4 — Exploit Research

Run after all services enumerated.

```bash
# Search for known exploits
searchsploit [service name]
searchsploit [service] [version]

# Copy relevant exploit to working directory
searchsploit -m [exploit/path]
```

- [ ] Searchsploit run on every identified service version
- [ ] CVEs researched for identified versions
- [ ] Metasploit modules checked

```bash
msfconsole
search [service/vulnerability]
```

---

## Enumeration Notes Template

Copy this block into your notes for every machine:

```
TARGET: [IP]
DATE: 
OS: 
OPEN PORTS: 
SERVICES:
  - [port]: [service] [version]
  - [port]: [service] [version]

WEB:
  - Technologies: 
  - Directories found: 
  - Login pages: 
  - Input fields: 

CREDENTIALS ATTEMPTED:
  - anonymous:anonymous — [result]
  - admin:admin — [result]
  - root:(blank) — [result]

INTERESTING FINDINGS:

ATTACK VECTORS TO PURSUE:
```

---

## Rules

**Never skip the full port scan.** Non-standard ports are where critical services hide. This has been demonstrated repeatedly — Redis on 6379 would have been missed by a default scan. Run `-p-` every time without exception.

**Save all scan output.** `nmap -oA recon/nmap_[target]` on every scan. Reference the saved output rather than re-scanning. Re-scanning generates more log entries on monitored systems.

**Attempt default credentials on every applicable service before moving to brute force.** Manual attempts are faster, quieter, and frequently sufficient. Anonymous FTP, null SMB sessions, and passwordless Redis are all findings that require zero tools beyond the service client.

**Check every file for sensitive content.** Configuration files, notes, history files, and backup files contain credentials more often than expected. Everything downloaded during enumeration gets read.

---

*Part of the [Offensive Security Portfolio](../README.md)*  
*Updated: April 2026*
