# Bandit Level 0 → Level 1
**Platform:** OverTheWire — Bandit  
**Date:** April 2026  
**Concept:** SSH authentication and remote host connection  

---

## Objective

Connect to the Bandit game server via SSH using provided credentials and retrieve the password stored in a file called `readme` in the home directory.

**Given:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit0`
- Password: `bandit0`

---

## Concepts Tested

- SSH (Secure Shell) — encrypted remote terminal protocol
- Connecting on non-standard ports
- Basic home directory navigation
- Reading file contents from the command line

---

## Methodology

### Step 1 — Connect via SSH

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

The non-standard port is the only thing worth pausing on here. Default SSH runs on 22 — this one is 2220. Easy to miss if you're moving fast and default to `ssh bandit0@host` without reading the spec.

Accept the host key fingerprint (`yes`) and enter the password `bandit0` when prompted.

### Step 2 — Look around

```bash
ls
```

Output:
```
readme
```

One file. Exactly what the challenge described. Good sign.

### Step 3 — Read it

```bash
cat readme
```

Output:
```
NH2SXQwcBdpmTEzi3bvBHMM9H66vVXjL
```

Password for `bandit1`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ssh user@host -p PORT` | Connect to remote host on non-default port |
| `ls` | List current directory contents |
| `cat filename` | Print file contents to terminal |

---

## What I Tried First

Straightforward level — no wrong turns. The only thing to note is that typing `ssh bandit0@bandit.labs.overthewire.org` without `-p 2220` throws a connection refused error. Not a big deal here, but it reinforces the habit of checking port specifications before connecting rather than assuming defaults.

---

## Real-World Pentest Application

**Non-standard ports matter more than people think.** Administrators routinely move SSH off port 22 as a low-effort hardening measure — it cuts down on automated scanning noise but doesn't stop a deliberate attacker. During a real engagement, running `nmap` with only default port ranges misses these entirely. Full-range scans (`nmap -p-`) are slower but ensure nothing gets overlooked. Finding SSH on port 2220, 2222, or 22222 on a target is common.

**`cat` for file enumeration** is one of those tools that sounds trivial until you're doing post-exploitation on a live host. Home directories, web roots, and config paths (`/etc/passwd`, `.bash_history`, `.ssh/authorized_keys`) are standard first stops after gaining access. The habit of listing a directory and reading what's there without overthinking it is worth building early.

---

## Key Takeaway

Level 0 is about establishing the connection pattern that every subsequent level builds on. The `-p` flag for non-standard ports is the one thing to internalize — it's a small detail that comes up constantly in both CTFs and real engagements.

**Next:** [Bandit Level 1 → 2](./level-01.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — OverTheWire Bandit series*

