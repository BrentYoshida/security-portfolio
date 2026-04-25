# HackTheBox — Meow
**Platform:** HackTheBox — Starting Point, Tier 0  
**Date:** April 2026  
**Difficulty:** Very Easy  
**Concept:** Service enumeration, Telnet, default credentials  

---

## Objective

Gain access to the target machine and retrieve the flag. First machine on HackTheBox — no prior context on what's running or where the flag is.

---

## Concepts Tested

- Network connectivity verification with `ping`
- Service and version detection with `nmap`
- Telnet — what it is, why it's a security risk
- Default and weak credential enumeration
- Basic post-access file enumeration

---

## Methodology

### Step 1 — Verify connectivity

```bash
ping [target IP]
```

Confirmed the target was live and reachable before doing anything else. Simple habit — no point running scans against a host that isn't responding.

### Step 2 — Enumerate services

```bash
sudo nmap -sV [target IP]
```

Output showed port 23/tcp open — Telnet. The `-sV` flag pulls service version information rather than just confirming a port is open. One open port, one obvious attack surface.

### Step 3 — Connect via Telnet

```bash
telnet [target IP]
```

Presented with a `Meow login:` prompt. Time to guess credentials.

### Step 4 — Credential attempts

Tried the obvious first:
- `administrator` — failed
- `admin` — failed
- `root` — no password required. Logged straight in.

No frustration here — this is exactly the right order to try. Most default credential attempts follow a short list of common usernames before anything creative. `root` with no password is a misconfiguration that shows up in real environments more than it should, particularly on embedded devices, IoT hardware, and poorly provisioned servers.

### Step 5 — Look around and grab the flag

```bash
ls
```

`flag.txt` visible in the directory.

```bash
cat flag.txt
```

Flag retrieved. Machine complete.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ping [IP]` | Verify host is live and reachable |
| `sudo nmap -sV [IP]` | Identify open ports and running service versions |
| `telnet [IP]` | Connect to Telnet service on port 23 |
| `ls` | List directory contents post-access |
| `cat flag.txt` | Read the flag |

---

## What I Tried First

First machine on HTB so the initial friction was less about the technical challenge and more about getting oriented — connecting to the VPN, understanding the interface, knowing where to start. Once that clicked, the approach was straightforward: verify the host is up, scan it, connect to whatever is open, try the most obvious credentials first.

The failed attempts on `administrator` and `admin` before `root` worked are worth documenting. Credential guessing without a list should always follow the same priority order — root/admin/administrator first, then service-specific defaults, then anything context-specific from enumeration. Skipping straight to wordlists before trying the obvious is a time waste.

---

## Real-World Pentest Application

**Telnet (port 23) on a real target is an immediate finding.** Telnet transmits everything — credentials, commands, session data — in plaintext over the network. Anyone with a packet capture between the client and server can read the entire session. It's been replaced by SSH for exactly this reason and its presence on a modern system usually means something was misconfigured or never properly hardened.

**Default credentials are one of the highest-yield findings in real engagements.** A significant percentage of networked devices — routers, switches, IP cameras, industrial control systems, IoT hardware — ship with default usernames and passwords that never get changed. `root` with no password, `admin/admin`, `admin/password` — these aren't edge cases. Tools like Hydra and Medusa automate credential stuffing against services, but trying a short manual list first is faster and quieter.

**`nmap -sV`** is the flag that makes version detection worth running. Knowing a port is open is useful. Knowing it's running a specific version of a service is what lets you search for known CVEs and relevant exploits. Building the habit of including `-sV` on initial scans rather than adding it as an afterthought saves time on every engagement.

---

## Key Takeaway

First machine down. The methodology here — verify connectivity, enumerate services, identify attack surface, try obvious credentials before anything complex — is the same sequence that applies to machines ten times harder than this one. The tools get more sophisticated, the vulnerabilities get more obscure, but the thinking stays the same.

Telnet open on a real target means plaintext credentials at risk. Default credentials working means someone never changed the factory settings. Both are findings worth reporting.

**Next:** [Fawn](./fawn.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — HackTheBox Starting Point*
