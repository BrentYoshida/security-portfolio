# Bandit Level 10 > Level 11
**Platform:** OverTheWire : Bandit  
**Date:** April 2026  
**Concept:** Base64 encoding and decoding  

---

## Objective

Log in as `bandit10` and decode the base64 encoded contents of `data.txt` to retrieve the password.

**Credentials:**
- Host: `bandit.labs.overthewire.org`
- Port: `2220`
- Username: `bandit10`
- Password: `FGUW5ilLVJrxX9kMYMmlN4MgbpfMiqey`

---

## Concepts Tested

- What base64 encoding actually is and why it exists
- `base64 -d` for decoding on the command line
- Why encoded data is not the same as encrypted data

---

## Background : What is Base64?

Before running anything I looked this one up. Base64 is a binary-to-text encoding scheme that translates raw binary data into a printable ASCII string format. It uses 64 characters : uppercase A–Z, lowercase a–z, digits 0–9, plus `+` and `/` : to represent data that would otherwise contain unprintable or unsafe bytes.

The reason it exists: many systems (email protocols, URLs, HTTP headers) were designed to handle text only. Binary data doesn't survive transport through those systems intact. Base64 solves that by converting anything into a safe, predictable character set.

Important distinction: **base64 is encoding, not encryption.** It's a transformation for compatibility, not confidentiality. Anyone with the string and one command can read what's inside.

---

## Methodology

### Step 1 : Connect and look around

```bash
ssh bandit10@bandit.labs.overthewire.org -p 2220
ls
```

Output:
```
data.txt
```

### Step 2 : Read the file

```bash
cat data.txt
```

Output:
```
VGhlIHBhc3N3b3JkIGlzIGR0UjE3M2ZaS2IwUlJzREZTR3NnMlJXbnBOVmozcVJyCg==
```

The character set and `==` padding at the end confirm base64 immediately. The `=` padding is added to make the total length a multiple of 4 : a structural requirement of the encoding. Once you've seen it a few times it's recognizable at a glance.

### Step 3 : Decode it

```bash
cat data.txt | base64 -d
```

Output:
```
The password is dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr
```

Password for `bandit11`: `dtR173fZKb0RRsDFSGsg2RWnpNVj3qRr`

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `cat data.txt` | Read file : confirmed base64 encoding from character set and padding |
| `base64 -d` | Decode base64 encoded input |
| `cat data.txt \| base64 -d` | Pipe file contents into decoder |

---

## What I Tried First

Straight to `base64 -d` once the encoding was confirmed visually. No dead ends here : the challenge description and the `==` padding made the approach obvious. The time went into understanding *what base64 is* before running the command, which felt more useful than just executing the solution.

---

## Real-World Pentest Application

**Base64 is not encryption : and that matters constantly on real engagements.** Developers frequently store credentials in base64 under the mistaken assumption that it obscures them. It doesn't. Common places base64 credentials appear in the wild:

```bash
# HTTP Basic Authentication headers : trivially decoded
# Authorization: Basic dXNlcjpwYXNzd29yZA==
echo "dXNlcjpwYXNzd29yZA==" | base64 -d
# Output: user:password

# Kubernetes secrets stored in YAML manifests
kubectl get secret mysecret -o yaml | grep -A1 "data:" | base64 -d

# Docker registry credentials in config.json
cat ~/.docker/config.json | python3 -c "import sys,json,base64; d=json.load(sys.stdin); [print(base64.b64decode(v['auth']).decode()) for v in d['auths'].values()]"

# Environment variables and .env files
grep -r "BASE64\|_KEY\|_TOKEN" /var/www/ 2>/dev/null | awk -F= '{print $2}' | base64 -d 2>/dev/null
```

The HTTP Basic Auth case is the most immediate. An Authorization header intercepted in Burp Suite or a packet capture that looks like `Basic dXNlcjpwYXNzd29yZA==` decodes to plaintext credentials in one command. No cracking, no tools : just `base64 -d`.

**Recognize it on sight:** limited character set (A–Z, a–z, 0–9, +, /), length always a multiple of 4, trailing `=` or `==` padding. When you find what looks like a random string in a config file, environment variable, or HTTP header : run it through `base64 -d` before moving on. It's credentials more often than you'd expect.

---

## Key Takeaway

Base64 is a compatibility tool, not a security control. The moment you can identify it visually : character set, padding, length : the decode is one command. Never assume encoded means protected.

**Next:** [Bandit Level 11 → 12](./level-11.md)

---

*Part of the [Offensive Security Portfolio](../../README.md) : OverTheWire Bandit series*
