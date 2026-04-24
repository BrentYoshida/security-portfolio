# Bandit Level 7 → Level 8
**Platform:** OverTheWire — Bandit  
**Date:** April 2026  
**Concept:** Searching large files with `grep` and piping output  

---

## Objective

Log in as `bandit7` and find the password stored in `data.txt` next to the word "millionth".

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit7`
- Password: `morbNTDkSW6jIlUc0ymOdMaLnOlFVAaj`

---

## Concepts Tested

- `grep` for pattern matching in large files
- Piping (`|`) to chain command output into another command
- Why reading large files raw is the wrong first instinct

---

## Methodology

### Step 1 — Connect and look around

```bash
ssh bandit7@bandit.labs.overthewire.org -p 2220
ls
```

Output:
```
data.txt
```

One file. Straightforward enough.

### Step 2 — First attempt

```bash
cat data.txt
```

Terminal flooded with thousands of lines — word/password pairs scrolling faster than anything useful could be pulled from them. `Ctrl+C` to stop it. `cat` on an unknown file before checking its size is a habit worth breaking.

### Step 3 — Pipe into grep to isolate the target line

The challenge says the password is next to the word "millionth." That's a `grep` job:

```bash
cat data.txt | grep "millionth"
```

Output:
```
millionth	dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc
```

One line. Done.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cat data.txt` | Read file — returned unusable volume of output |
| `cat data.txt \| grep "millionth"` | Pipe file contents into grep to find the matching line |
| `grep "pattern" file` | Equivalent direct form — no cat required |

---

## What I Tried First

Opened the file raw with `cat` — immediately obvious that wasn't going to work. The volume of output was the signal to stop and think about filtering before reading. Once the approach shifted to "search for the known string" rather than "read the file," the answer was one command away. Worth noting: `grep "millionth" data.txt` achieves the same result without the `cat` — passing the file directly to `grep` is slightly cleaner and skips the unnecessary pipe.

---

## Real-World Pentest Application

**Large file searching is constant work during real engagements.** After getting access to a system, log files, configuration files, and database dumps are frequently enormous — manually reading them isn't an option. `grep` is the primary tool for making them useful:

```bash
# Search for passwords across an entire file
grep -i "password" data.txt

# Search recursively through a directory
grep -r "api_key" /var/www/ 2>/dev/null

# Show line numbers for context
grep -n "password" config.txt

# Show surrounding lines for context (-A after, -B before, -C both)
grep -C 3 "password" config.txt

# Search for multiple patterns at once
grep -E "password|secret|token|api_key" config.txt

# List only filenames that contain the pattern
grep -rl "password" /etc/ 2>/dev/null
```

**Piping is a force multiplier.** The `|` operator is one of the most powerful concepts in Linux — it lets you chain simple commands into precise workflows without writing a script. `cat file | grep pattern | sort | uniq` takes a massive input and progressively filters it down to exactly what's needed. Every level of the pipe adds a constraint. The result at the end is surgical.

On a live target, combining `find` from Level 5 and 6 with `grep` creates a two-stage search that's genuinely powerful:

```bash
# Find readable files then search their contents for credentials
find / -type f -readable 2>/dev/null | xargs grep -l "password" 2>/dev/null
```

That one-liner scans every accessible file on the system for the word "password" and returns a list of files worth reading — without opening a single one manually.

---

## Key Takeaway

`cat` on an unknown file is the wrong first move. Check size first (`ls -lh`) or pipe directly into a filter. `grep` turns an unreadable wall of text into a single relevant line — and that same skill scales directly to log analysis, credential hunting, and config file review on real systems.

**Next:** [Bandit Level 8 → 9](./level-08.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — OverTheWire Bandit series*
