# Bandit Level 4 > Level 5
**Platform:** OverTheWire : Bandit  
**Date:** April 2026  
**Concept:** Identifying file types without extensions  

---

## Objective

Log in as `bandit4` and find the only human-readable file inside the `inhere` directory. Several files are present : only one contains the password.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit4`
- Password: `2WmrDFRmJIq3IPxneAaMGhap0pFhF3NJ`

---

## Concepts Tested

- The `file` command for identifying file types without relying on extensions
- Glob patterns for running a command against multiple files at once
- Why file extensions are unreliable indicators of file content

---

## Methodology

### Step 1 : Connect and navigate

```bash
ssh bandit4@bandit.labs.overthewire.org -p 2220
cd inhere
```

### Step 2 : List the directory

```bash
ls -ahl
```

Output:
```
-file00  -file01  -file02  -file03  -file04
-file05  -file06  -file07  -file08  -file09
```

Ten files, no extensions, all with the same dash-prefix naming convention from Level 1. Opening each one manually with `cat` would work but would also dump binary garbage to the terminal for any non-text files : not ideal.

### Step 3 : Use `file` to identify types across all files at once

The `file` command reads the file header (magic bytes) to determine what a file actually contains regardless of its name or extension. The glob pattern `*` hits all of them in one shot:

```bash
file ./-file0*
```

Output:
```
./-file00: data
./-file01: data
./-file02: data
./-file03: data
./-file04: data
./-file05: data
./-file06: data
./-file07: ASCII text
./-file08: data
./-file09: data
```

`-file07` is the only one returning `ASCII text`. Everything else is binary data.

### Step 4 : Read the file

```bash
cat ./-file07
```

Output:
```
4oQYVPkxZOOEOO5pTW81FB8j8lxXGUQw
```

Password for `bandit5`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ls -ahl` | List all files with hidden, human-readable sizes, and detail |
| `file ./-file0*` | Identify file types using magic bytes : glob across all matches |
| `cat ./-file07` | Read the ASCII text file |

---

## What I Tried First

Went straight to `file ./-file0*` after the `ls` output : previous levels established that these dash-prefixed filenames need the `./` prefix to avoid shell misinterpretation. The glob pattern `*` saved running `file` ten times individually. Only one output line said `ASCII text`, which made the target obvious without opening anything blindly.

---

## Real-World Pentest Application

**File extensions on a target system mean nothing.** Attackers routinely rename malicious executables with innocent extensions (`.txt`, `.jpg`, `.log`) to evade naive detection. Defenders who rely on extension-based filtering get bypassed constantly. The `file` command reads magic bytes : the actual header data embedded in the file itself : which is significantly harder to fake without breaking the file's functionality.

During post-exploitation enumeration, running `file *` across directories of interest is faster and more reliable than making assumptions based on names. Stumbling across a file called `backup.txt` that `file` identifies as an ELF executable or a PKCS private key is exactly the kind of find that changes the direction of an engagement.

**Glob patterns** are worth getting comfortable with early. `file ./-file0*` instead of ten individual commands is a small efficiency here : but the same logic applies when you're writing enumeration scripts or one-liners against large directory trees on a live target. Less noise, faster results, fewer commands in the logs.

---

## Key Takeaway

Never trust filenames or extensions to tell you what something is. `file` reads the actual content and is one of the first commands worth running when you land on a new system and start poking around unfamiliar directories.

**Next:** [Bandit Level 5 → 6](./level-05.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) : OverTheWire Bandit series*
