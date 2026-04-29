# Bandit Level 1 > Level 2
**Platform:** OverTheWire: Bandit  
**Date:** April 2026  
**Concept:** Reading files with special character names  

---

## Objective

Log in as `bandit1` using the password from Level 0 and retrieve the password stored in a file named `-` in the home directory.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit1`
- Password: `ZjLjTmM6FvvyRnrb2rfNWOZOTa6ip5If`

---

## Concepts Tested

- Special character filenames and how the shell interprets them
- Using path prefixes to disambiguate file arguments from flags
- Understanding how commands parse stdin vs file input

---

## Methodology

### Step 1: Connect

```bash
ssh bandit1@bandit.labs.overthewire.org -p 2220
```

### Step 2: List the directory

```bash
ls
```

Output:
```
-
```

File is there. One character filename. Already suspicious.

### Step 3: First attempt

```bash
cat -
```

The terminal hung and waited for input. Didn't error out :  just sat there expecting me to type something.

This is the key behavior to understand: the shell interprets `-` as a convention for stdin (standard input), not as a filename. So `cat -` tells cat to read from keyboard input rather than open a file. Most Unix utilities follow this convention :  it's not a `cat` quirk, it's a shell-wide pattern.

`Ctrl+C` to cancel and rethink.

### Step 4: Solution: specify the path explicitly

By prepending `./` we tell the shell this is a path relative to the current directory :  not a flag or stdin indicator:

```bash
cat ./-       OR    cat ~/-
```

Output:
```
263JGJPfgU6LtdEvgfWU1XP5yac29mFx
```

Password for `bandit2`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cat ./-` | Read file named `-` by specifying relative path |
| `Ctrl+C` | Cancel a hanging process |

---

## What I Tried First

Went straight to `cat -` out of habit. The terminal hanging rather than throwing an error was actually the useful signal :  it confirmed the shell was interpreting `-` as stdin, not failing to find the file. That distinction matters: a "command not found" error and a hanging terminal are telling you very different things about what went wrong.

Could also solve this with:
```bash
cat < -
```
Input redirection treats `-` as a literal filename rather than a stdin indicator. Same result, different mechanism :  worth knowing both approaches.

---

## Real-World Pentest Application

**Special character filenames are an evasion technique.** Files named `-`, `--`, or with leading spaces or null bytes are used by attackers to hide malicious files in directories :  they're easy to overlook visually and break naive enumeration scripts that don't handle them properly. During post-exploitation on a Linux host, files that don't behave normally under `cat` or `ls` deserve closer attention, not dismissal.

More broadly, understanding how the shell parses arguments versus filenames is critical when writing automation scripts during an engagement. A script that passes uncleaned user input or filenames directly to shell commands is a command injection vulnerability waiting to happen :  the same parsing behavior that caused `cat -` to hang is the mechanism behind a class of real injection bugs.

---

## Key Takeaway

The shell treats `-` as stdin by convention across almost all Unix utilities. When a file has a name that conflicts with shell syntax, prepend `./` to force path interpretation. This comes up more than you'd expect :  in CTFs, on misconfigured servers, and occasionally in the wild when someone names a log file or output file something that breaks standard tooling.

**Next:** [Bandit Level 2 → 3](./level-02.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) :  OverTheWire Bandit series*
