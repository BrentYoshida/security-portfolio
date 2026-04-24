# Bandit Level 5 → Level 6
**Platform:** OverTheWire — Bandit  
**Date:** April 2026  
**Concept:** Using `find` with multiple conditions to isolate a target file  

---

## Objective

Log in as `bandit5` and locate the password file somewhere inside the `inhere` directory. The file has three known properties: human-readable, exactly 1033 bytes in size, and not executable.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit5`
- Password: `4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw`

---

## Concepts Tested

- `find` with chained conditions (`-type`, `-size`, `! -executable`)
- Suppressing permission errors with `2>/dev/null`
- Narrowing search scope to reduce noise
- Understanding why broad searches return unmanageable results

---

## Methodology

### Step 1 — Connect and navigate

```bash
ssh bandit5@bandit.labs.overthewire.org -p 2220
cd inhere
ls
```

Several subdirectories inside `inhere`. No obvious target — too many places to look manually.

### Step 2 — First attempt: search by size alone

```bash
find / -type f -size 1033c
```

Came back with a wall of results pulling from system directories across the entire filesystem. The search worked but the scope was too broad to be useful — hundreds of files at 1033 bytes exist on a typical Linux system.

### Step 3 — Add conditions to eliminate noise

The challenge gives three properties. Stack them all into one `find` command:

- `-type f` — regular file only
- `-size 1033c` — exactly 1033 bytes (`c` = bytes)
- `! -executable` — not executable

Also redirect stderr to `/dev/null` to suppress the permission denied errors from directories we can't read:

```bash
find / -type f -size 1033c ! -executable 2>/dev/null
```

Output:
```
/home/bandit5/inhere/maybehere07/.file2
```

One result. Clean.

### Step 4 — Read the file

```bash
cat maybehere07/.file2
```

Output:
```
HWasnPhtq9AVKe0dmk45nxy20cvUa6EG
```

Password for `bandit6`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `find / -type f -size 1033c` | Find regular files of exact size — too broad alone |
| `find / -type f -size 1033c ! -executable 2>/dev/null` | Same search with executable filter and stderr suppressed |
| `2>/dev/null` | Redirect error output to null — cleans up permission denied noise |
| `cat maybehere07/.file2` | Read the target file |

---

## What I Tried First

Started with `find / -type f -size 1033c` and immediately got buried in results. The fix wasn't a different approach — it was adding the remaining constraints the challenge already provided. The lesson here is to use every piece of information given before assuming a search is broken. The first command worked; it just needed more conditions to become useful.

The `2>/dev/null` addition is worth calling out specifically. Without it, permission denied errors from `/proc`, `/sys`, and other restricted directories flood the output and make real results hard to spot. Redirecting stderr doesn't change what `find` searches — it just keeps the terminal readable.

---

## Real-World Pentest Application

**`find` with chained conditions is a core post-exploitation tool.** After landing on a system, locating files by properties rather than name is frequently more useful than navigating directories manually. Common real-world searches:

```bash
# Find SUID binaries — privilege escalation candidates
find / -type f -perm -4000 2>/dev/null

# Find files owned by root that are writable
find / -type f -user root -writable 2>/dev/null

# Find recently modified files — attacker activity indicator
find / -type f -mmin -10 2>/dev/null

# Find files containing the word "password"
find / -type f -readable 2>/dev/null | xargs grep -l "password" 2>/dev/null
```

Each of these uses the same chained-condition logic from this level applied to a real objective. The `2>/dev/null` pattern appears in all of them for the same reason — permission errors on a production system are noise, not signal.

**Scope matters.** Searching `/` when you only need `inhere` wastes time and generates more log entries on a monitored system. On a live engagement, tightening search scope to relevant directories reduces both runtime and detection risk.

---

## Key Takeaway

`find` becomes powerful when you chain conditions. A single filter often returns too much — stacking `-type`, `-size`, `-executable`, `-user`, `-perm`, and `-mtime` together narrows results to exactly what you need. `2>/dev/null` is not optional in real environments — make it a habit.

**Next:** [Bandit Level 6 → 7](./level-06.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — OverTheWire Bandit series*
