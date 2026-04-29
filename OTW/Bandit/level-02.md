# Bandit Level 2 > Level 3
**Platform:** OverTheWire : Bandit  
**Date:** April 2026  
**Concept:** Handling filenames with spaces  

---

## Objective

Log in as `bandit2` and retrieve the password stored in a file called `spaces in this filename` in the home directory.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit2`
- Password: `263JGJPfgU6LtdEvgfWU1XP5yac29mFx`

---

## Concepts Tested

- How the shell parses arguments separated by spaces
- Quoting and escaping filenames with whitespace
- Tab completion as a practical enumeration habit

---

## Methodology

### Step 1 : Connect

```bash
ssh bandit2@bandit.labs.overthewire.org -p 2220
```

### Step 2 : List the directory

```bash
ls
```

Output:
```
spaces in this filename
```

Four words. The shell is going to interpret each one as a separate argument if we're not careful.

### Step 3 : First attempt

```bash
cat spaces in this filename
```

Output:
```
cat: spaces: No such file or directory
cat: in: No such file or directory
cat: this: No such file or directory
cat: filename: No such file or directory
```

Exactly what you'd expect. The shell split the filename on whitespace and passed four separate arguments to `cat`. None of them exist as individual files.

### Step 4 : Solution: quote the filename

I added the ./ between the cat command and the quotes:   cat ./"--spaces in this filename--"
If you don't put the ./, an error "unrecognized option" occurs because the cat command interprets the dashes (--) 
and spaces at the beginning of the filename as command-line arguments rather than the file name itself.

Wrapping the full filename in quotes tells the shell to treat everything inside as a single argument:

```bash
cat ./"--spaces in this filename--"
```

Output:
```
MNk8KNH3Usiio41PRUEoDFPqfxLPlSmx
```

Password for `bandit3`.

Alternatively, escape each space with a backslash:

```bash
cat spaces\ in\ this\ filename
```

Same result. Both approaches are worth knowing : quotes are more readable, backslash escaping comes in handy when you're building commands programmatically or dealing with nested quoting situations.

Tab completion also handles this automatically:

```bash
cat sp<TAB>
```

The shell completes to `cat spaces\ in\ this\ filename` with escapes inserted. Fast and accurate : useful habit in general.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cat "filename with spaces"` | Read file with spaces using quoting |
| `cat file\ with\ spaces` | Read file with spaces using backslash escaping |
| `<TAB>` | Shell autocomplete : handles special characters automatically |

---

## What I Tried First

Went straight to `cat spaces in this filename` out of habit : got the four-way error immediately. Useful failure though. The error output confirmed exactly what was happening: the shell was treating each word as a separate argument. Once the parsing behavior is clear the fix is obvious.

---

## Real-World Pentest Application

**Spaces in filenames are a real evasion technique.** Filenames like `"system config "` (trailing space) or `" .bashrc"` (leading space) are visually easy to miss and break scripts that don't quote their variables properly. Attackers use this to hide files in directories where a cursory `ls` and `cat` will fail silently.

**Unquoted variables in shell scripts** are one of the most common sources of command injection vulnerabilities. If a script does `cat $filename` instead of `cat "$filename"` and an attacker controls the filename value, the shell splits on whitespace and the extra tokens become additional arguments : or in certain contexts, injected commands. This level is a small demonstration of exactly that parsing behavior. Understanding it at the shell level is what lets you recognize it as a vulnerability in code.

---

## Key Takeaway

The shell splits on whitespace by default. Anything that needs to be treated as a single token : filenames, arguments, variable values : should be quoted. It's a habit that prevents bugs in your own scripts and helps you recognize the class of injection vulnerabilities that stem from the same parsing behavior.

**Next:** [Bandit Level 3 → 4](./level-03.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) : OverTheWire Bandit series*
