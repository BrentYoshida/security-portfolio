# Bandit Level 9 → Level 10
**Platform:** OverTheWire — Bandit  
**Date:** April 2026  
**Concept:** Extracting readable strings from binary data  

---

## Objective

Log in as `bandit9` and find the password stored in `data.txt` — a mostly binary file containing a few human-readable strings. The password is preceded by several `=` characters.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit9`
- Password: `4CKMh1JI91bUIZZPXDqGanal4xvAg0JM`

---

## Concepts Tested

- `strings` for extracting printable text from binary files
- Chaining `grep` to filter `strings` output by known pattern
- Refining grep patterns when initial results are too broad
- Recognizing when a tool is wrong for the job before going further

---

## Methodology

### Step 1 — Connect and look around

```bash
ssh bandit9@bandit.labs.overthewire.org -p 2220
ls
```

Output:
```
data.txt
```

### Step 2 — First attempt: treat it like previous levels

```bash
cat data.txt | sort | uniq -u
```

Terminal filled with binary garbage — scrambled characters, control sequences, noise. This isn't a text file. The pipeline from Level 8 doesn't apply here. Different file type, different approach.

### Step 3 — Use `strings` to extract readable content

`strings` scans a binary file and pulls out any sequence of printable characters above a minimum length (default 4). It ignores everything else:

```bash
strings data.txt
```

Returns readable fragments mixed in with the binary content — closer, but still too much output to identify the password manually.

### Step 4 — Filter by the known pattern

The challenge says the password follows several `=` characters. Pipe into `grep`:

```bash
strings data.txt | grep "="
```

Still noisy — single `=` characters appear in enough places to muddy the results. Tighten the pattern:

```bash
strings data.txt | grep "=="
```

Output:
```
========== the
2========== password
========== is
========== FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey
```

Password for `bandit10`: `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cat data.txt \| sort \| uniq -u` | First attempt — confirmed file is binary, not text |
| `strings data.txt` | Extract printable character sequences from binary file |
| `strings data.txt \| grep "="` | Filter for `=` — too broad |
| `strings data.txt \| grep "=="` | Filter for `==` — isolated the target lines |

---

## What I Tried First

Ran the Level 8 pipeline out of habit — immediately clear from the output that this was binary data and not a text file. That failed attempt was actually useful: it confirmed the file type without needing to run `file data.txt` first. Switched to `strings`, which pulled readable fragments but still needed filtering. Starting with `grep "="` was close but returned too many matches. Tightening to `==` cut the noise to four lines and the answer was obvious.

The refinement step — going from `=` to `==` — is worth calling out. Grep patterns are as specific as you make them. When the first filter returns too much, narrowing the pattern is faster than reading through the output manually.

---

## Real-World Pentest Application

**`strings` is a standard first pass on unknown binaries.** During post-exploitation or malware analysis, running `strings` on an executable or unknown file frequently reveals hardcoded credentials, API keys, URLs, internal hostnames, error messages, and debug output left by developers. It takes seconds and regularly surfaces high-value information without needing a disassembler:

```bash
# Common targets for strings analysis
strings suspicious_binary | grep -i "password"
strings suspicious_binary | grep -i "http"
strings suspicious_binary | grep -i "key\|token\|secret"
strings /usr/bin/someapp | grep -E "[0-9]{1,3}\.[0-9]{1,3}"  # IP addresses
```

Developers routinely leave database connection strings, internal API endpoints, and plaintext credentials inside compiled binaries assuming nobody will look. `strings` is the tool that proves them wrong in about thirty seconds.

**Iterative grep refinement** is the practical skill here. On a real target, the first filter is rarely tight enough — useful signal is usually buried in noise. The habit of progressively narrowing the pattern (`=` → `==`) rather than reading raw output manually is what keeps analysis fast and systematic.

**Binary vs text awareness** matters before running commands. Running a text-processing pipeline on binary data wastes time and can crash poorly written scripts. Checking file type with `file filename` before deciding on an approach is a clean habit — this level demonstrated exactly why.

---

## Key Takeaway

`strings` turns binary files into something searchable. Pipe it into `grep` with a pattern derived from whatever context you have about the target — then tighten the pattern if the first pass returns too much. The combination works on executables, firmware dumps, memory captures, and any other binary artifact you encounter on a real system.

**Next:** [Bandit Level 10 → 11](./level-10.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) — OverTheWire Bandit series*
