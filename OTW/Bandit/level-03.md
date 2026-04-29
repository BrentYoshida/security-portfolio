# Bandit Level 3 > Level 4
**Platform:** OverTheWire : Bandit  
**Date:** April 2026  
**Concept:** Hidden files and directory navigation  

---

## Objective

Log in as `bandit3` and retrieve the password stored in a hidden file inside a directory called `inhere`.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit3`
- Password: `MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx`

---

## Concepts Tested

- Hidden files in Linux (dot-prefix convention)
- `ls` flags for revealing non-standard files
- Directory navigation with `cd`

---

## Methodology

### Step 1 : Connect

```bash
ssh bandit3@bandit.labs.overthewire.org -p 2220
```

### Step 2 : List home directory, navigate to inhere

```bash
ls
```

Output:
```
inhere
```

One directory. Move into it.

```bash
cd inhere
```

### Step 3 : First attempt

```bash
ls
```

Output:
```

```

Nothing. Empty directory : or so it appears.

### Step 4 : List hidden files

In Linux, any file or directory prefixed with `.` is hidden from a standard `ls` output. The `-a` flag reveals everything:

```bash
ls -a
```

Output:
```
.  ..  ...Hiding-From-You
```

There it is. `...Hiding-From-You` : three dots as a prefix rather than one, which buries it further from casual inspection.

### Step 5 : Read the file

```bash
cat ...Hiding-From-You
```

Output:
```
2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ
```

Password for `bandit4`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cd directory` | Navigate into a directory |
| `ls -a` | List all files including hidden (dot-prefixed) entries |
| `cat ...Hiding-From-You` | Read the hidden file |

---

## What I Tried First

Standard `ls` inside `inhere` returned nothing : which is itself useful information. An empty-looking directory on a target host is worth a second look with `ls -a` before moving on. The answer here was immediate once the right flag was applied. The filename `...Hiding-From-You` using three dots instead of one is a small detail worth noting : multiple leading dots still triggers the hidden behavior, and visually it blends into directory output even when you know to look for it.

---

## Real-World Pentest Application

**Hidden files are everywhere on real systems** : and not just for CTF flavor. Dotfiles store credentials, session tokens, and shell history that are frequently overlooked during incident response and hardening reviews. On a compromised Linux host, `ls -a` in home directories is a reflex. Common high-value targets:

- `.bash_history` : command history, sometimes contains plaintext credentials
- `.ssh/` : private keys, authorized_keys, known_hosts
- `.aws/credentials` : AWS access keys left by developers
- `.env` : application environment files with API keys and database passwords
- `.git/` : source code repositories accidentally left on production servers

**Attackers also use hidden files for persistence.** Dropping a backdoor as `.sshd` or `...update` in `/tmp` or a home directory is a basic but effective concealment technique. During a post-exploitation review or blue team audit, `ls -la` (hidden files with full detail) in common directories is non-negotiable.

---

## Key Takeaway

`ls` without flags is incomplete enumeration. On any system you're assessing : or investigating : `ls -la` gives you hidden files, permissions, ownership, timestamps, and symlinks in one shot. Build it as a default habit over `ls` alone.

**Next:** [Bandit Level 4 → 5](./level-04.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) : OverTheWire Bandit series*
