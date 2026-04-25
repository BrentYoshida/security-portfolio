# HackTheBox — Dancing
**Platform:** HackTheBox — Starting Point, Tier 0  
**Date:** April 2026  
**Difficulty:** Very Easy  
**Concept:** SMB enumeration, null authentication, share access  

---

## Objective

Enumerate SMB shares on the target, identify which ones are accessible without credentials, and retrieve the flag from an open share.

---

## Concepts Tested

- SMB (Server Message Block) — what it is and how it's used
- Null session authentication on SMB shares
- Listing and navigating shares with smbclient
- Understanding share permissions and what each share typically contains

---

## Background — What is SMB?

SMB (Server Message Block) is a network communication protocol used primarily for sharing files, printers, and other resources between nodes on a network. It runs on port 445 (TCP) and is the backbone of Windows file sharing — when you browse a network drive or map a shared folder on a Windows machine, SMB is almost always the protocol doing the work.

SMB also has a significant history in offensive security. It's the protocol exploited by EternalBlue (MS17-010), the vulnerability behind WannaCry ransomware and one of the most impactful exploits ever released. Understanding SMB isn't just useful for file enumeration — it's foundational for understanding a significant class of real-world attacks.

The weakness here: SMB servers can be configured to allow **null sessions** — connections made without any username or password. When null authentication is enabled, anyone on the network can list available shares and potentially access their contents.

---

## Methodology

### Step 1 — Enumerate services

```bash
sudo nmap -sV [target IP]
```

Output:
```
445/tcp open  microsoft-ds
```

Port 445 open — SMB. The `microsoft-ds` service name confirms Windows SMB. Version detection gives enough context to know what we're dealing with before connecting.

### Step 2 — Install smbclient

```bash
sudo apt-get install smbclient
```

smbclient is the standard command-line tool for interacting with SMB shares from Linux. Installed before attempting any connections.

### Step 3 — List available shares

```bash
smbclient -L [target IP]
Password: (none — just pressed Enter)
```

No password required. Null session accepted. Output listed available shares including:
- `ADMIN$`
- `C$`
- `WorkShares`

`ADMIN$` and `C$` are default Windows administrative shares. They require admin credentials. `WorkShares` is a custom share — worth trying without credentials.

### Step 4 — Test share access

```bash
smbclient \\\\[IP]\\ADMIN$
```
Access denied — requires administrator credentials.

```bash
smbclient \\\\[IP]\\C$
```
Access denied — same reason.

```bash
smbclient \\\\[IP]\\WorkShares
```
Connected without a password. Inside the share.

### Step 5 — Navigate and enumerate

```bash
smb: \> ls
```

Two directories visible: `Amy.J` and `James.P`.

```bash
smb: \> cd Amy.J
smb: \Amy.J\> ls
smb: \Amy.J\> get worknotes.txt
smb: \Amy.J\> cd ..
smb: \> cd James.P
smb: \James.P\> ls
smb: \James.P\> get flag.txt
```

Downloaded both files. `worknotes.txt` from Amy's directory, `flag.txt` from James's.

### Step 6 — Read the files

```bash
cat worknotes.txt
cat flag.txt
```

Flag retrieved. `worknotes.txt` contained internal notes — in a real engagement this kind of file is exactly what you'd collect as evidence of data exposure.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `sudo nmap -sV [IP]` | Identify service and version on open ports |
| `sudo apt-get install smbclient` | Install SMB client tool |
| `smbclient -L [IP]` | List available SMB shares — null session |
| `smbclient \\\\[IP]\\ShareName` | Connect to specific share |
| `ls` | List contents of current SMB directory |
| `cd [directory]` | Navigate into directory |
| `get [filename]` | Download file from share to local machine |
| `cat [filename]` | Read downloaded file contents |

---

## What I Tried First

Listed shares first before attempting to connect to any of them — knowing what's available before trying access is cleaner than connecting blind. Attempted `ADMIN$` and `C$` first since they were listed first, got access denied on both as expected. Those are administrative shares that require domain or local admin credentials. `WorkShares` had no access control configured — connected immediately without credentials.

The directory structure inside `WorkShares` gave away that this was a shared drive for individual users. Checked both directories before grabbing anything — `Amy.J` had notes worth collecting, `James.P` had the flag.

---

## Real-World Pentest Application

**SMB enumeration is one of the first things to run on any internal network engagement.** Open shares with null sessions or misconfigured permissions are among the most common findings in corporate environments — particularly on file servers, legacy systems, and any infrastructure that hasn't been through a recent security review.

**Standard SMB enumeration workflow on a real target:**

```bash
# List shares with null session
smbclient -L [IP] -N

# Enumerate shares, permissions, and users with enum4linux
enum4linux -a [IP]

# Use smbmap to check permissions across all shares at once
smbmap -H [IP]

# With credentials
smbmap -H [IP] -u [username] -p [password]

# Connect to a specific share
smbclient \\\\[IP]\\[ShareName] -N

# Download everything accessible recursively
smb: \> recurse ON
smb: \> prompt OFF
smb: \> mget *
```

**What to look for inside accessible shares:**

- Configuration files containing database credentials or API keys
- Password files — even named something obvious like `passwords.txt`
- IT documentation revealing internal network topology
- Backup files containing sensitive data
- Scripts with hardcoded credentials

**The `ADMIN$` and `C$` shares matter even when access is denied.** Their presence confirms the target is a Windows machine and that SMB administrative shares are enabled. If credentials are obtained later in the engagement — through phishing, password spraying, or credential dumping — coming back to these shares with valid admin credentials often provides full filesystem access.

**SMB in the context of real attacks:**

The EternalBlue exploit (MS17-010) targets SMB directly and is still found in unpatched environments. It's also the exploit used in the HTB machine "Blue" — which is on your roadmap for good reason. Understanding SMB null sessions now makes the jump to understanding SMB exploitation significantly easier when you get there.

---

## Key Takeaway

Always list shares before connecting to any of them — `smbclient -L` gives you the full picture upfront. Custom shares (anything that isn't `ADMIN$`, `C$`, `IPC$`) are the first place to test for null or guest access. What looks like a mundane file share in a CTF is the same misconfiguration that exposes internal documents, credentials, and sensitive data in real corporate environments.

**Next:** [Redeemer](./redeemer.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — HackTheBox Starting Point*
