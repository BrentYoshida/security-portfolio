# HackTheBox : Redeemer
**Platform:** HackTheBox : Starting Point, Tier 0  
**Date:** April 2026  
**Difficulty:** Very Easy  
**Concept:** Redis enumeration, unauthenticated access, in-memory database extraction  

---

## Objective

Identify and connect to an exposed Redis instance on a non-standard port, enumerate the database contents, and retrieve the flag stored as a key-value pair.

---

## Concepts Tested

- What Redis is and why it's a security risk when exposed
- Full port scanning with `-p-` to find non-standard services
- Connecting to Redis without authentication
- Redis CLI commands for database enumeration and data extraction

---

## Background : What is Redis?

Looked this up before starting. Redis (REmote DIctionary Server) is an open-source, in-memory data store used as a database, cache, and message broker. Everything Redis stores lives in RAM : which makes it extremely fast but means data is lost on restart unless persistence is configured.

Redis is a NoSQL key-value store, meaning data is stored and retrieved by key name rather than SQL queries. It's widely used in production applications for session storage, real-time leaderboards, queuing systems, and caching database query results.

The security implication: **Redis has no authentication enabled by default.** It was designed to run on internal networks behind a firewall, trusting that nothing external would ever reach it. When a Redis instance is exposed to the internet or an internal attacker without authentication configured, the entire database is accessible to anyone who can connect to port 6379.

---

## Methodology

### Step 1 : Verify connectivity

```bash
ping [target IP]
```

Host live. Standard first step.

### Step 2 : Full port scan with version detection

```bash
nmap -p- -sV [target IP]
```

Output:
```
6379/tcp open  redis  Redis key-value store 5.0.7
```

The `-p-` flag is what found this. A default nmap scan only checks the 1000 most common ports : port 6379 isn't in that list. Without scanning all 65535 ports this service would have been completely missed. This is exactly why full port scans matter on every target.

Version 5.0.7 noted. Older Redis versions have known vulnerabilities : worth keeping in mind for future engagements.

### Step 3 : Install Redis CLI and check usage

```bash
sudo apt install redis-tools
redis-cli --help
```

Installed the client before connecting. Checked the help output to confirm the `-h` flag for specifying a remote host.

### Step 4 : Connect to Redis

```bash
redis-cli -h [target IP]
```

Connected directly with no password prompt. Unauthenticated access confirmed.

### Step 5 : Enumerate the database

```bash
10.10.10.127:6379> info
```

The `info` command returns server configuration, statistics, and keyspace information. Scrolling to the `#Keyspace` section:

```
# Keyspace
db0:keys=4,expires=0,avg_ttl=0
```

Database 0 contains 4 keys. That's the target.

### Step 6 : Select the database and list keys

```bash
10.10.10.127:6379> select 0
10.10.10.127:6379> keys *
```

`select 0` switches to database index 0. `keys *` returns all key names : the `*` is a wildcard matching everything. Four key names returned, one of them clearly the flag.

### Step 7 : Retrieve the flag

```bash
10.10.10.127:6379> get flag
```

Flag value returned directly. Redis stores data as key-value pairs : `get [key]` retrieves whatever is stored at that key, no further steps required.

---

## Commands Used

| Command | Purpose |
|---------|---------|
| `ping [IP]` | Verify host is live |
| `nmap -p- -sV [IP]` | Full port scan with version detection : found non-standard port |
| `sudo apt install redis-tools` | Install Redis CLI client |
| `redis-cli -h [IP]` | Connect to remote Redis instance |
| `info` | Dump server info including keyspace |
| `select 0` | Switch to database index 0 |
| `keys *` | List all keys in selected database |
| `get [key]` | Retrieve value stored at a key |

---

## What I Tried First

Ran a full `-p-` scan from the start based on the pattern established in previous machines : never assume default ports are the whole picture. Found Redis on 6379 immediately. Connected without credentials, ran `info` to understand the database structure before touching any keys, then listed everything with `keys *`. The flag key name was obvious in the output. One `get` command and done.

The `info` step before `keys *` is worth keeping as a habit : it shows the full server configuration including which databases exist and how many keys each contains. On a real target with hundreds of keys, knowing the structure before dumping everything saves time.

---

## Real-World Pentest Application

**Exposed Redis instances are a critical finding in real engagements.** Shodan and Censys regularly index tens of thousands of Redis servers exposed to the internet with no authentication. Attackers use these for data theft, cryptocurrency mining, and in some configurations : remote code execution.

**Standard Redis enumeration on a real target:**

```bash
# Connect and check if authentication is required
redis-cli -h [IP] ping
# Response: PONG = no auth required

# Dump all server information
redis-cli -h [IP] info

# List all databases with key counts
redis-cli -h [IP] info keyspace

# Select a database and dump all keys
redis-cli -h [IP] select 0
redis-cli -h [IP] keys *

# Get all values : use with caution on large databases
redis-cli -h [IP] --scan | xargs redis-cli -h [IP] get
```

**What attackers look for in exposed Redis:**

- Session tokens : Redis is commonly used for web application session storage. Extracting session tokens allows account takeover without credentials
- API keys and secrets : developers cache sensitive configuration data in Redis assuming it's internal-only
- Personally identifiable information : user data cached for performance
- Application logic : queued jobs often contain serialized objects with sensitive parameters

**The RCE path from Redis:** Redis has a `CONFIG SET` command that can change where it writes its persistence file. If Redis is running as root (common in misconfigured deployments), an attacker can write an SSH public key to `/root/.ssh/authorized_keys` or a cron job to `/etc/cron.d/` : turning unauthenticated database access into full system compromise. This is a well-documented attack path and the reason exposed Redis instances are treated as critical severity findings.

**Why `-p-` matters:** This machine is the clearest demonstration so far of why full port scans are non-negotiable. Redis on 6379 would have been completely invisible to a default nmap scan. On real engagements, services on non-standard ports : databases, admin panels, development servers : are frequently where the most sensitive data lives precisely because administrators assume "nobody knows it's there."

---

## Key Takeaway

Always run `-p-`. Non-standard ports hide critical services that default scans miss entirely. Unauthenticated Redis is a critical finding : not just for data extraction but as a potential path to full system compromise depending on how the service is configured. The `info` > `select` > `keys *` > `get` sequence is the complete Redis enumeration workflow.

**Tier 0 complete.**

---

*Part of the [Offensive Security Portfolio](../../README.md) : HackTheBox Starting Point*
