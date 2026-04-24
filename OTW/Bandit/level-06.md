# Bandit Level 6 → Level 7
**Platform:** OverTheWire — Bandit  
**Date:** April 2026  
**Concept:** Finding files by ownership and group across the entire filesystem  

---

## Objective

Log in as `bandit6` and find the password file stored *somewhere on the server*. The file has three known properties: owned by user `bandit7`, owned by group `bandit6`, and exactly 33 bytes in size.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit6`
- Password: `HWasnPhtq9AVKe0dmk45nxy20cvUa6EG`

---

## Concepts Tested

- `find` with `-user` and `-group` flags for ownership-based searches
- Searching outside the home directory — full filesystem scope
- `2>/dev/null` to suppress permission errors at scale
- Understanding Linux file ownership and its security implications

---

## Methodology

### Step 1 — Connect and look around

```bash
ssh bandit6@bandit.labs.overthewire.org -p 2220
ls
ls -alh
```

Home directory is empty. Nothing hidden either. The challenge said *somewhere on the server* — the home directory was never the right place to look.

### Step 2 — Search the filesystem by ownership and size

The challenge gives three properties. All three go into a single `find` command. Searching from `/` covers the entire filesystem:

```bash
find / -group bandit6 -user bandit7 -size 33c 2>/dev/null
```

Output:
```
/var/lib/dpkg/info/bandit7.password
```

One result. The `2>/dev/null` is doing a lot of work here — without it, permission denied errors from restricted system directories would bury this single line.

### Step 3 — Read the file

```bash
cat /var/lib/dpkg/info/bandit7.password
```

Output:
```
morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj
```

Password for `bandit7`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ls -alh` | Confirm home directory is empty including hidden files |
| `find / -group bandit6 -user bandit7 -size 33c 2>/dev/null` | Search full filesystem by group owner, user owner, and size |
| `cat /var/lib/dpkg/info/bandit7.password` | Read the located file |

---

## What I Tried First

Checked the home directory first with `ls` and `ls -alh` — nothing there. The challenge specifying *somewhere on the server* was the signal to go wide. The `find` command came together cleanly using the three properties given. The path that came back — `/var/lib/dpkg/info/` — is a package management directory that wouldn't come up in normal manual enumeration. That's exactly the point: ownership-based searches surface files in places you'd never think to navigate to by hand.

---

## Real-World Pentest Application

**Ownership-based searches are a privilege escalation reflex.** After landing on a system as a low-privilege user, finding files owned by higher-privilege users or sensitive groups is one of the first things worth running:

```bash
# Files owned by root but writable by current user
find / -user root -writable -type f 2>/dev/null

# Files owned by current user's group with elevated permissions
find / -group $(id -gn) -type f 2>/dev/null

# SUID files — run as owner regardless of who executes them
find / -user root -perm -4000 -type f 2>/dev/null

# SGID files — run as group owner
find / -perm -2000 -type f 2>/dev/null
```

The `/var/lib/dpkg/info/` path is also worth filing away. Package management directories store installation scripts, file lists, and configuration data for every package on the system. On a real target, browsing `/var/lib/dpkg/info/` occasionally surfaces credentials and configuration files dropped by poorly written install scripts — the kind of thing that doesn't show up in obvious places but has been sitting there since the package was first installed.

**`2>/dev/null` at filesystem scope** is non-negotiable on a real engagement. A Linux system with normal permissions will generate thousands of permission denied errors when you search from `/` as a non-root user. Suppressing stderr keeps output actionable and avoids the kind of terminal noise that slows down time-sensitive work.

---

## Key Takeaway

When a challenge or engagement tells you a file is *somewhere on the system*, ownership and size constraints fed into `find` from `/` is the move — not manual directory browsing. The combination of `-user`, `-group`, and `-size` narrowed an entire filesystem down to a single result in one command.

**Next:** [Bandit Level 7 → 8](./level-07.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — OverTheWire Bandit series*
