# Bandit Level 8 > Level 9
**Platform:** OverTheWire : Bandit  
**Date:** April 2026  
**Concept:** Finding unique lines using `sort` and `uniq`  

---

## Objective

Log in as `bandit8` and find the password stored in `data.txt` : it's the only line that appears exactly once.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit8`
- Password: `dfwvzFQi4mU0wfNbFOe9RoWskMLg7eEc`

---

## Concepts Tested

- `sort` for ordering file contents before deduplication
- `uniq -u` for isolating lines that appear exactly once
- Why `uniq` requires sorted input to work correctly
- Chaining pipes to build a precise filtering pipeline

---

## Methodology

### Step 1 : Connect and look around

```bash
ssh bandit8@bandit.labs.overthewire.org -p 2220
ls
```

Output:
```
data.txt
```

Same setup as Level 7. Knowing what `cat data.txt` returns by now : skip it and go straight to filtering.

### Step 2 : Sort, then filter for unique lines

```bash
cat data.txt | sort | uniq -u
```

Output:
```
4CKMh1JI91bUIZZPXDqGanal4xvAg0JM
```

One line. That's the password for `bandit9`.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `sort` | Sorts lines alphabetically : required before `uniq` |
| `uniq -u` | Outputs only lines that appear exactly once |
| `cat data.txt \| sort \| uniq -u` | Full pipeline: read → sort → isolate unique |

---

## What I Tried First

Went straight to the pipeline : the challenge description gave enough information to build it without exploring first. The key insight is that `sort` has to come before `uniq`. `uniq` only compares adjacent lines, so without sorting first, duplicate lines scattered throughout the file wouldn't be detected. Sorting groups all identical lines together so `uniq` can count them correctly.

Running `uniq -u` alone on an unsorted file would return mostly garbage : a detail that's easy to miss and worth understanding rather than just memorizing the command order.

---

## Real-World Pentest Application

**Deduplication pipelines are a log analysis staple.** During an engagement : or when investigating a compromised system : log files frequently contain thousands of repeated entries mixed with a handful of anomalous ones. The `sort | uniq -c | sort -rn` pattern is particularly useful:

```bash
# Count occurrences of each unique line, sort by frequency
cat access.log | sort | uniq -c | sort -rn
```

This turns a web server access log into a frequency-ranked list : the most common requests at the top, rare or suspicious ones at the bottom. Spotting a single request to `/admin/shell.php` buried in ten thousand normal requests is exactly this kind of problem.

Other useful `uniq` variants:

```bash
# Show only lines that appear more than once (duplicates)
sort data.txt | uniq -d

# Count how many times each line appears
sort data.txt | uniq -c

# Case-insensitive deduplication
sort data.txt | uniq -i
```

**Pipes compound.** Level 7 introduced the two-stage pipeline. This level adds a third stage : `sort` feeding into `uniq` feeding into a result. On a real target it's common to build five or six stage pipelines to extract exactly what you need from noisy data without writing a script. Fast, disposable, effective.

---

## Key Takeaway

`uniq` without `sort` is broken : it only catches adjacent duplicates. Always sort first. The `sort | uniq -c | sort -rn` pattern specifically is worth memorizing : it turns any list of repeated data into a ranked frequency table, which is one of the most useful things you can do with raw log output during an investigation.

**Next:** [Bandit Level 9 → 10](./level-09.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) : OverTheWire Bandit series*
