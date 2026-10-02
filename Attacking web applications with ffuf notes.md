# Attacking Web Applications with Ffuf — Notes

## Table of Contents

- [1. Introduction](#1-introduction)
- [2. Web Fuzzing](#2-web-fuzzing)
  - [2.1 Overview](#21-overview)
  - [2.2 Fuzzing](#22-fuzzing)
  - [2.3 Wordlists](#23-wordlists)
- [3. Directory Fuzzing](#3-directory-fuzzing)
  - [3.1 Overview](#31-overview)
  - [3.2 Ffuf](#32-ffuf)
  - [3.3 Directory Fuzzing (building the command)](#33-directory-fuzzing-building-the-command)
  - [3.4 Interpreting the Output](#34-interpreting-the-output)
- [4. Page Fuzzing](#4-page-fuzzing)
  - [4.1 Overview](#41-overview)
  - [4.2 Extension Fuzzing](#42-extension-fuzzing)
  - [4.3 Page Fuzzing (finding filenames)](#43-page-fuzzing-finding-filenames)
- [5. Recursive Fuzzing](#5-recursive-fuzzing)
  - [5.1 Overview](#51-overview)
  - [5.2 Recursive Flags](#52-recursive-flags)
  - [5.3 Recursive Scanning (building the command)](#53-recursive-scanning-building-the-command)
- [6. DNS Records](#6-dns-records)
  - [6.1 Overview](#61-overview)
  - [6.2 Why the Connection Fails](#62-why-the-connection-fails)
  - [6.3 Fixing It — Editing /etc/hosts](#63-fixing-it--editing-etchosts)
  - [6.4 Result and Next Step](#64-result-and-next-step)
- [7. Sub-domain Fuzzing](#7-sub-domain-fuzzing)
  - [7.1 Overview](#71-overview)
  - [7.2 Sub-domains](#72-sub-domains)
  - [7.3 Running the Subdomain Scan](#73-running-the-subdomain-scan)
- [8. Vhost Fuzzing](#8-vhost-fuzzing)
  - [8.1 Overview](#81-overview)
  - [8.2 Vhosts vs. Sub-domains](#82-vhosts-vs-sub-domains)
  - [8.3 Vhosts Fuzzing — Method](#83-vhosts-fuzzing--method)
  - [8.4 Interpreting the Output](#84-interpreting-the-output)
- [9. Filtering Results](#9-filtering-results)
  - [9.1 Overview](#91-overview)
  - [9.2 Filtering](#92-filtering)
  - [9.3 Applying the Filter](#93-applying-the-filter)
  - [9.4 Verifying the Finding](#94-verifying-the-finding)
- [10. Parameter Fuzzing - GET](#10-parameter-fuzzing---get)
  - [10.1 Overview](#101-overview)
  - [10.2 GET Request Fuzzing](#102-get-request-fuzzing)
- [11. Parameter Fuzzing - POST](#11-parameter-fuzzing---post)
  - [11.1 Overview](#111-overview)
  - [11.2 POST vs. GET](#112-post-vs-get)
  - [11.3 Building the POST Fuzzing Command](#113-building-the-post-fuzzing-command)
  - [11.4 Verifying with curl](#114-verifying-with-curl)
- [12. Value Fuzzing](#12-value-fuzzing)
  - [12.1 Overview](#121-overview)
  - [12.2 Custom Wordlist](#122-custom-wordlist)
  - [12.3 Value Fuzzing](#123-value-fuzzing)
- [13. Summary of the Overall Workflow](#13-summary-of-the-overall-workflow)

---

<a id="1-introduction"></a>
## 1. Introduction

This module is built around `ffuf`, one of the most widely used and reliable tools for web fuzzing. Fuzzing itself is a broad technique with many possible tools, but the module narrows its focus to `ffuf` because of its speed, flexibility, and simple syntax. The goal of the module is to teach how to discover hidden parts of a web application — directories, files, subdomains, virtual hosts, and parameters — that are not linked anywhere visible on the site. By the end, the skills combine into a full workflow: find a hidden path, find hidden parameters on that path, and find the correct values for those parameters.

The module covers five core topics:
- Fuzzing for directories
- Fuzzing for files and extensions
- Identifying hidden vhosts
- Fuzzing for PHP parameters
- Fuzzing for parameter values

In short, `ffuf` automates the process of testing thousands of possible names (from a wordlist) against a target, and reports back which ones produced a valid response (commonly HTTP 200), so the tester can manually inspect only the promising results instead of guessing one by one.

---

<a id="2-web-fuzzing"></a>
## 2. Web Fuzzing

<a id="21-overview"></a>
### 2.1 Overview
This section introduces the starting scenario: a website with no visible links or navigation, meaning nothing can be discovered by simply clicking around. Since there is no information pointing to other pages, the only practical option left is to fuzz the site to uncover hidden content. This sets up the motivation for everything that follows in the module — without fuzzing, large parts of a web application would remain invisible to a tester, even though they are technically reachable.

<a id="22-fuzzing"></a>
### 2.2 Fuzzing
Fuzzing is a testing technique where varied types of input are sent to an interface in order to observe how the system reacts. The description gives two examples of how fuzzing adapts to context: for SQL injection testing, random special characters are sent to see if the server breaks or reveals errors; for buffer overflow testing, long strings of increasing length are sent to find the point where the binary crashes. For web content discovery (the focus of this module), fuzzing means sending many possible directory/file names to a server and checking which ones exist.

Because web servers generally do not expose a directory listing of all their pages (unless badly configured), testers must guess likely paths and check the server's response. Two examples illustrate this:
- Visiting a path that does not exist (e.g. `/doesnotexist`) returns an HTTP **404 Not Found** along with a "Page Not Found" style page.
- Visiting a path that does exist (e.g. `/login`) returns an HTTP **200 OK** along with the actual page content.

Doing this manually for every possible word would take enormous time, so automated tools send hundreds of requests per second, inspect the HTTP response code for each, and report back which guessed paths are real. This lets a tester quickly narrow down a huge list of possibilities to only the handful worth examining by hand.

<a id="23-wordlists"></a>
### 2.3 Wordlists
A wordlist is a prepared list of likely directory/page names, conceptually similar to a password dictionary used in brute-force attacks (a topic covered later in the module). While a wordlist will never reveal every hidden page — some names are random or unique to a specific application — in practice, common wordlists can uncover up to roughly 90% of pages on many websites, making them a highly efficient first step.

Rather than building a wordlist from scratch, the module points to the **SecLists** GitHub repository, a community-maintained collection of wordlists covering many fuzzing use cases, including common passwords (relevant later for password brute-forcing). On the HTB PwnBox, this repository is pre-installed at `/opt/useful/SecLists`. The specific wordlist used for directory/page fuzzing in this module is `directory-list-2.3-small.txt`, which can be located with the `locate` command shown below.

```
locate directory-list-2.3-small.txt
```
- `locate` — a Linux utility that searches a pre-built filesystem index for files matching the given name, returning their full paths almost instantly (faster than `find`, though it relies on an index that may need updating with `updatedb`).
- `directory-list-2.3-small.txt` — the filename being searched for; this is the specific SecLists wordlist used for directory and page fuzzing in this module.
- **Result:** the command returns the full path `/opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt`, confirming where the wordlist lives on disk.

**Tip given in the content:** The wordlist file contains copyright comment lines at its top, which are not real directory names and will clutter fuzzing results if sent as guesses. The fix (introduced for use in the next section) is to add the `-ic` flag to `ffuf`, which tells it to ignore comment lines in the wordlist.

---

<a id="3-directory-fuzzing"></a>
## 3. Directory Fuzzing

<a id="31-overview"></a>
### 3.1 Overview
With the concept of fuzzing and a usable wordlist established, this section moves into actually running `ffuf` to discover real directories on a live target. It builds the command up step by step: first introducing the tool and its help menu, then assigning the wordlist to a keyword, then placing that keyword into a target URL, and finally executing the full scan and interpreting its output.

<a id="32-ffuf"></a>
### 3.2 Ffuf
`ffuf` comes pre-installed on the PwnBox. For a personal machine, it can be installed via `apt install ffuf -y`, or downloaded directly from its GitHub repository. As a first step with any new tool, the module recommends checking its help menu.

```
ffuf -h
```
- `ffuf` — invokes the fuzzing tool.
- `-h` — displays the help menu, listing every available flag grouped by category (HTTP options, matcher options, filter options, input options, output options, etc.).

Key flags introduced from the help output:
- `-H` — add a custom HTTP header (`"Name: Value"`); can be used multiple times for multiple headers.
- `-X` — specify the HTTP method (defaults to `GET`).
- `-b` — supply cookie data for the request.
- `-d` — supply `POST` data.
- `-recursion` — enables recursive scanning; only works with the `FUZZ` keyword, and the target URL must end with it.
- `-recursion-depth` — sets how many levels deep a recursive scan will go.
- `-u` — the target URL to fuzz.
- `-mc` — match specific HTTP status codes (defaults to `200,204,301,302,307,401,403`), or `all` to match everything.
- `-ms` — match a specific HTTP response size.
- `-fc` — filter out (exclude) specific HTTP status codes.
- `-fs` — filter out (exclude) responses of a specific size.
- `-w` — path to the wordlist, optionally followed by `:KEYWORD` to name it.
- `-o` — write the scan output to a file.

An example usage line from the help output is also shown:
```
ffuf -w wordlist.txt -u https://example.org/FUZZ -mc all -fs 42 -c -v
```
This fuzzes paths from `wordlist.txt` against the target, matches every status code (`-mc all`), filters out responses of size 42 (`-fs 42`), colorizes output (`-c`), and shows verbose output (`-v`).

<a id="33-directory-fuzzing-building-the-command"></a>
### 3.3 Directory Fuzzing (building the command)
The two essential flags for directory fuzzing are `-w` (wordlist) and `-u` (target URL). A wordlist is first bound to a keyword by appending `:FUZZ` after its path:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ
```
- `-w <path>:FUZZ` — loads the given wordlist file and assigns it the placeholder name `FUZZ`, which can then be referenced anywhere in the command.

Next, the `FUZZ` keyword is placed inside the target URL at the position where a directory name would go:

```
ffuf -w <SNIP> -u http://SERVER_IP:PORT/FUZZ
```
- `-u http://SERVER_IP:PORT/FUZZ` — the target URL; `ffuf` will substitute each word from the wordlist in place of `FUZZ` and send a request for each one.

Finally, the full command is run against the live target:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ
```
- This combines the wordlist binding and the target URL: `ffuf` iterates through every word in `directory-list-2.3-small.txt`, substitutes it for `FUZZ` in the URL, sends a GET request, and records the HTTP response for each.

**Step-by-step task:**
1. Confirm the wordlist path using `locate` (from the previous section).
2. Bind the wordlist to the `FUZZ` keyword with `-w <path>:FUZZ`.
3. Build the target URL with `-u`, placing `FUZZ` where the directory name belongs.
4. Run the combined command and wait for it to complete.
5. Review the output table, which lists each discovered path along with its HTTP status, response size, word count, and line count.

<a id="34-interpreting-the-output"></a>
### 3.4 Interpreting the Output
Running the scan tested almost 90,000 URLs in under 10 seconds (scan speed depends on network conditions and whether `ffuf` is run locally or remotely). The output showed one notable hit:
```
blog    [Status: 301, Size: 326, Words: 20, Lines: 10]
```
This means the directory `/blog` exists and returned an HTTP 301 (redirect).

**Speeding up the scan:** The number of concurrent threads can be increased, e.g. `-t 200`, to send more requests per second. However, this is explicitly **not recommended** against remote targets, since it can overload the server (causing a Denial of Service) or saturate the tester's own internet connection.

Visiting `http://SERVER_IP:PORT/blog` directly returns an empty page — but critically, **not** a 404 or 403 error, confirming the directory exists and is accessible, even though it has no visible content yet. This leads naturally into the next section, which fuzzes for files inside this directory.

---

<a id="4-page-fuzzing"></a>
## 4. Page Fuzzing

<a id="41-overview"></a>
### 4.1 Overview
Having confirmed that `/blog` exists but appears empty, this section fuzzes *inside* that directory to find hidden files and pages. Before file names can be guessed, the correct file extension (e.g. `.php`, `.html`, `.aspx`) must first be identified — this section covers both extension discovery and then full filename discovery.

<a id="42-extension-fuzzing"></a>
### 4.2 Extension Fuzzing
One manual approach to guessing a site's language/extension is to inspect the HTTP response headers for the server type (e.g. Apache often pairs with PHP; IIS often pairs with ASP/ASPX). However, this method is unreliable, so the module instead fuzzes for the extension directly using `ffuf`, with a dedicated SecLists wordlist of common web extensions.

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ <SNIP>
```
- `-w .../web-extensions.txt:FUZZ` — loads a wordlist of common file extensions (e.g. `.php`, `.html`, `.asp`) bound to the keyword `FUZZ`.
- `<SNIP>` — placeholder indicating the rest of the command (the target URL) is omitted here and completed in the next step.

Since an extension alone is meaningless without a base filename, the module uses `index.*` — a filename that exists on almost every website — as the base, and fuzzes only the extension part:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ
```
- `-u http://SERVER_IP:PORT/blog/indexFUZZ` — the `FUZZ` keyword is placed directly after `index`, so each extension in the wordlist is appended to `index` (e.g. `index.php`, `index.html`). Note: the wordlist already contains the leading dot, so no extra `.` is added manually.

**Result:** Two hits came back — `.php` with HTTP 200 (file exists and loads) and `.phps` with HTTP 403 (exists but access is denied). Since `.php` returned a valid 200 response, the site is confirmed to run on PHP.

<a id="43-page-fuzzing-finding-filenames"></a>
### 4.3 Page Fuzzing (finding filenames)
With the extension known (`.php`), the same directory-list wordlist used earlier for directory fuzzing is reused — this time to guess filenames inside `/blog/`, with `.php` appended as a fixed suffix.

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php
```
- `-u http://SERVER_IP:PORT/blog/FUZZ.php` — each word from the wordlist is substituted for `FUZZ`, and `.php` is appended afterward, so `ffuf` tests names like `index.php`, `admin.php`, `login.php`, etc.

**Step-by-step task:**
1. Identify the server's likely file extension by fuzzing `index.FUZZ` with the web-extensions wordlist.
2. Confirm the working extension from the HTTP 200 result (here, `.php`).
3. Reuse the directory wordlist, this time fuzzing filenames with the confirmed extension appended (`FUZZ.php`).
4. Run the scan against the target directory (`/blog/`).
5. Review results: distinguish between pages that exist but are empty (size 0) versus pages that return real content (non-zero size).

**Result:** Two hits returned HTTP 200 — `index.php` (size 0, confirming it is empty, matching what was seen earlier in the browser) and a second redacted file with a non-zero size, meaning it has real content. Visiting that second file in a browser revealed a message indicating an admin panel had been relocated, setting up the next stage of the walkthrough (finding where it moved to).

---

<a id="5-recursive-fuzzing"></a>
## 5. Recursive Fuzzing

<a id="51-overview"></a>
### 5.1 Overview
Manually repeating the "fuzz a directory, then fuzz inside it" process for every discovered directory does not scale once a site has many nested subdirectories. This section introduces recursive fuzzing, which automates that repetition so `ffuf` continues fuzzing newly found directories on its own.

<a id="52-recursive-flags"></a>
### 5.2 Recursive Flags
When recursive scanning is enabled, `ffuf` automatically starts a new fuzzing job under every newly discovered directory, continuing until the entire site tree (within limits) has been explored. Because deeply nested directory trees (e.g. `/login/user/content/uploads/...`) could make a recursive scan take a very long time, it is strongly advised to set a maximum depth so the scan doesn't run unbounded. A common workflow is to run a shallow recursive scan first, then pick the most interesting discovered directories and run a focused follow-up scan on them.

Key flags:
- `-recursion` — enables recursive scanning.
- `-recursion-depth` — sets how many directory levels deep the recursion will go. For example, `-recursion-depth 1` fuzzes the main directories and their direct subdirectories only; it will not continue into subdirectories discovered inside those (e.g. if `/login/user` is found, it is not fuzzed further).
- `-e .php` — specifies an extension to apply during recursive fuzzing (since the same extension typically applies site-wide).
- `-v` — outputs full/verbose URLs for every result, which is necessary in recursive scans so it is clear which directory each discovered `.php` file belongs to.

<a id="53-recursive-scanning-building-the-command"></a>
### 5.3 Recursive Scanning (building the command)
The module reuses the original directory-fuzzing command and adds the recursion flags plus the `.php` extension:

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v
```
- `-w ...:FUZZ` — same directory wordlist as before, bound to the keyword `FUZZ`.
- `-u http://SERVER_IP:PORT/FUZZ` — base target URL with the fuzzing position marked.
- `-recursion` — turns on automatic recursive fuzzing into newly discovered directories.
- `-recursion-depth 1` — limits recursion to one level deep (main directories plus their immediate subdirectories only).
- `-e .php` — appends `.php` to fuzzed names so files are tested alongside directories.
- `-v` — prints full URLs in the output so results from different directories are distinguishable.

**Step-by-step task:**
1. Start from the base directory-fuzzing command used in Section 3.
2. Add `-recursion` to enable automatic scanning of newly found directories.
3. Add `-recursion-depth 1` to cap how deep the automatic recursion goes.
4. Add `-e .php` so filenames are also tested with the known extension.
5. Add `-v` so each result shows its full path, making nested results easy to read.
6. Run the scan and observe that it automatically queues new jobs (e.g. for `/forum/FUZZ`) as new directories are found.

**Result:** The scan took considerably longer and sent about six times more requests than the earlier scan, because the wordlist was effectively doubled (tested both with and without the `.php` extension) and multiple directories were automatically queued. Despite the longer runtime, it successfully reproduced every result found manually in earlier sections — directories like `/blog`, files like `/blog/index.php`, and the site root `index.php` — all from a single command.

---

<a id="6-dns-records"></a>
## 6. DNS Records

<a id="61-overview"></a>
### 6.1 Overview
After visiting the discovered `/blog/index.php`-adjacent page, a message appeared stating the admin panel had moved to `academy.htb`. However, attempting to visit `academy.htb` directly in a browser failed to connect. This section explains why that happens and how to fix it by editing the local hosts file.

<a id="62-why-the-connection-fails"></a>
### 6.2 Why the Connection Fails
Browsers only know how to connect to IP addresses, not domain names directly. When a URL is entered, the browser needs to translate it into an IP address, which it does by checking two sources in order: the local `/etc/hosts` file, and then the public Domain Name System (DNS). If the domain is found in neither place, the browser cannot determine where to connect and the request fails.

Since `academy.htb` is a private lab domain (not a publicly registered website), it has no public DNS record. It also hasn't yet been added to the local `/etc/hosts` file, so the browser has no way to resolve it — hence the "can't connect to the server" error, even though the server is actually reachable by its IP address.

<a id="63-fixing-it--editing-etchosts"></a>
### 6.3 Fixing It — Editing /etc/hosts
To resolve `academy.htb` locally, it must be manually mapped to the target's IP address inside `/etc/hosts`.

```
sudo sh -c 'echo "SERVER_IP  academy.htb" >> /etc/hosts'
```
- `sudo` — runs the command with administrator privileges, required because `/etc/hosts` is a protected system file.
- `sh -c '...'` — runs the quoted string as a shell command; this wrapping is needed here because a plain `sudo echo ... >> /etc/hosts` would not work as expected (the redirection `>>` would run with the original user's permissions, not root's, due to how shell redirection is processed before `sudo` takes effect).
- `echo "SERVER_IP  academy.htb"` — outputs a line pairing the target's IP address with the hostname `academy.htb`.
- `>> /etc/hosts` — appends that line to the end of the `/etc/hosts` file (rather than overwriting the file, which a single `>` would do).

**Step-by-step task:**
1. Identify the target's IP address (`SERVER_IP`) from the running exercise.
2. Run the `sudo sh -c 'echo "SERVER_IP academy.htb" >> /etc/hosts'` command, substituting the real IP.
3. Confirm the entry was added by checking `/etc/hosts` (e.g. with `cat /etc/hosts`).
4. Revisit `http://academy.htb:PORT` in the browser (including the correct port) — it should now load successfully.

<a id="64-result-and-next-step"></a>
### 6.4 Result and Next Step
After the hosts file update, `academy.htb` loaded — but it turned out to be the exact same website already being tested (confirmed by successfully visiting `/blog/index.php` again). A full recursive scan on this domain found no admin panel or related content. This implies the admin panel lives under a **subdomain** of `academy.htb` rather than on the root domain itself, which is the problem tackled in the next section.

---

<a id="7-sub-domain-fuzzing"></a>
## 7. Sub-domain Fuzzing

<a id="71-overview"></a>
### 7.1 Overview
This section covers how to discover subdomains of a target domain (e.g. `photos.google.com` as a subdomain of `google.com`) using `ffuf`, by checking whether each guessed subdomain has a valid public DNS record pointing to a working server.

<a id="72-sub-domains"></a>
### 7.2 Sub-domains
A subdomain is a website that exists "under" a parent domain. The example given is `https://photos.google.com`, which is the `photos` subdomain of `google.com`. Discovering subdomains works by testing many candidate names and checking whether each one resolves via public DNS to an actual server — if it does, a response (such as a redirect or page content) comes back; if not, the request fails.

Two things are needed before scanning: a wordlist of likely subdomain names, and a target domain. SecLists provides dedicated subdomain wordlists under `/opt/useful/seclists/Discovery/DNS/`; this module uses the shorter `subdomains-top1million-5000.txt` (a larger list can be substituted for a more thorough scan).

<a id="73-running-the-subdomain-scan"></a>
### 7.3 Running the Subdomain Scan
The target used for demonstration is `inlanefreight.com`, with the `FUZZ` keyword placed where the subdomain portion of the URL would go.

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
```
- `-w .../subdomains-top1million-5000.txt:FUZZ` — loads the subdomain wordlist, bound to `FUZZ`.
- `-u https://FUZZ.inlanefreight.com/` — the target URL with `FUZZ` placed as the subdomain prefix, so `ffuf` tests values like `www.inlanefreight.com`, `blog.inlanefreight.com`, etc.

**Step-by-step task:**
1. Choose a subdomain wordlist from SecLists (e.g. the 5000-word list for a quicker scan).
2. Set the target domain and place `FUZZ` in the subdomain position of the URL.
3. Run the scan and observe hits — status codes like 301 (redirect) or 200 (success) indicate a real subdomain was found.
4. Repeat the same command against a different domain if needed, to compare results.

**Result on `inlanefreight.com`:** Several subdomains were found, including `support`, `ns3`, `blog`, `my`, and `www` — the latter returning a full HTTP 200 page.

**Result on `academy.htb`:** Running the identical scan against `academy.htb` produced **zero hits** — every single request failed (shown as errors in the progress summary). This does not mean no subdomains exist; it means `academy.htb` has no entries in the *public* DNS system, since it is a private lab domain. Even though `academy.htb` itself was manually added to `/etc/hosts` earlier, that entry only covers the exact domain typed in — it does not automatically cover subdomains, so `ffuf`'s DNS lookups for guessed subdomains still fail. This limitation motivates the next section's technique: VHost fuzzing, which works without relying on public DNS at all.

---

<a id="8-vhost-fuzzing"></a>
## 8. Vhost Fuzzing

<a id="81-overview"></a>
### 8.1 Overview
Since subdomain fuzzing failed for `academy.htb` due to the lack of public DNS records, this section introduces an alternative method — Virtual Host (VHost) fuzzing — that can discover hidden subdomains/virtual sites even when there is no DNS record for them at all.

<a id="82-vhosts-vs-sub-domains"></a>
### 8.2 Vhosts vs. Sub-domains
The core distinction: a VHost is effectively a "subdomain" that is served from the *same* physical server and IP address, meaning a single server can host multiple distinct websites differentiated only by the hostname requested. Crucially, VHosts may or may not have a public DNS record — many organizations run internal or staging sites as VHosts specifically so they are not publicly discoverable via DNS.

Because subdomain fuzzing (Section 7) relies entirely on public DNS resolution, it can only ever find *public* subdomains. VHost fuzzing instead targets a server's IP directly and manipulates the HTTP request itself, allowing both public and non-public VHosts to be discovered on a known IP, without needing any DNS record at all.

<a id="83-vhosts-fuzzing--method"></a>
### 8.3 Vhosts Fuzzing — Method
Since the target IP is already known, there's no need to populate `/etc/hosts` with the entire wordlist. Instead, `ffuf` fuzzes the `Host:` HTTP header directly using the `-H` flag, with the `FUZZ` keyword embedded inside the header's value.

```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'
```
- `-w .../subdomains-top1million-5000.txt:FUZZ` — the same subdomain wordlist used previously, bound to `FUZZ`.
- `-u http://academy.htb:PORT/` — the request is always sent to the same known IP/domain and port; only the header changes between requests.
- `-H 'Host: FUZZ.academy.htb'` — overrides the `Host` header of each HTTP request with a guessed subdomain name. Since web servers can host multiple sites on one IP, the server uses this header to decide which site's content to return — this is exactly how VHost-based hosting works.

**Step-by-step task:**
1. Confirm the target's known IP address and port (already resolved via `/etc/hosts`).
2. Choose a subdomain-style wordlist.
3. Build the command sending every request to the same base URL, but fuzzing the `Host:` header instead of the URL path.
4. Run the scan and inspect response sizes, since status codes alone won't be useful here (explained below).

<a id="84-interpreting-the-output"></a>
### 8.4 Interpreting the Output
Every single request returned **HTTP 200**, including nonexistent VHosts like `mail2`, `dns2`, `ns3`, etc. This happens because the server simply serves its *default* site whenever it receives a `Host:` header it doesn't recognize — so a 200 response by itself does not confirm a VHost actually exists. However, if a guessed VHost *does* exist, it will likely serve genuinely different content, which should produce a noticeably different response size. This sets up the need for filtering by response size, which is addressed directly in the next section.

---

<a id="9-filtering-results"></a>
## 9. Filtering Results

<a id="91-overview"></a>
### 9.1 Overview
Up to this point, `ffuf`'s default filtering (based on status code, excluding 404s) was enough to narrow results. But as seen in the previous VHost scan, every request returned HTTP 200 regardless of whether the VHost was real, making status-code filtering useless there. This section covers filtering by other criteria — specifically response size — to cut through that noise.

<a id="92-filtering"></a>
### 9.2 Filtering
`ffuf` supports both *matching* (keep only responses meeting a condition) and *filtering* (exclude responses meeting a condition), based on HTTP status code, response size, line count, word count, or a regex pattern. These are visible in the help menu:

- `-mc` — match specific HTTP status codes, or `all` for everything (default: `200,204,301,302,307,401,403`).
- `-ml` — match a specific number of lines in the response.
- `-mr` — match a given regex pattern in the response.
- `-ms` — match a specific response size.
- `-mw` — match a specific word count in the response.
- `-fc` — filter out specific status codes (comma-separated list/ranges).
- `-fl` — filter out responses with a specific line count (comma-separated list/ranges).
- `-fr` — filter out responses matching a regex.
- `-fs` — filter out responses with a specific size (comma-separated list/ranges).
- `-fw` — filter out responses with a specific word count (comma-separated list/ranges).

In this scenario, matching can't be used because the correct VHost's response size isn't known in advance. However, the *incorrect* (default/catch-all) response size **is** known from the previous scan — it was consistently 900 bytes — so filtering that size out is the practical approach.

<a id="93-applying-the-filter"></a>
### 9.3 Applying the Filter
```
ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900
```
- This is the same VHost-fuzzing command from Section 8, with one addition:
- `-fs 900` — filters out (hides) any response whose size is exactly 900 bytes, which was identified as the size of the generic "no such VHost" default page. Only responses with a *different* size will now be shown.

**Step-by-step task:**
1. Run an initial VHost (or directory/subdomain) scan without filters to observe the "default"/incorrect response size.
2. Identify that repeated, uninteresting size value (here, 900 bytes).
3. Re-run the same scan, adding `-fs <size>` to exclude that known default size.
4. Review the much shorter list of remaining results — these are the genuinely different (and therefore interesting) responses.

**Result:** With the filter applied, a single meaningful hit appeared: `admin` with a size of 0 — meaning a VHost `admin.academy.htb` exists and returns a distinct (empty) page, different from the generic default.

<a id="94-verifying-the-finding"></a>
### 9.4 Verifying the Finding
Visiting `https://admin.academy.htb:PORT/` confirmed the VHost works, showing an empty page (as expected from the size-0 result). Two notes accompany this step: first, `admin.academy.htb` must also be manually added to `/etc/hosts` before the browser can resolve it; second, if the lab exercise instance is restarted, the port number may change and should be rechecked.

Further confirming this is a distinct VHost (not the same site as before): visiting `https://admin.academy.htb:PORT/blog/index.php` returns a **404 Not Found**, proving that `admin.academy.htb` is a genuinely separate site from the one hosting `/blog`. The section ends with a suggested exercise: run a recursive scan against `admin.academy.htb` to discover its pages — a task picked up in the next section.

---

<a id="10-parameter-fuzzing---get"></a>
## 10. Parameter Fuzzing - GET

<a id="101-overview"></a>
### 10.1 Overview
A recursive scan against `admin.academy.htb` surfaces `admin/admin.php`. Visiting this page directly shows some form of access-control message, implying the page checks for a specific identifying value before revealing its content (in this case, a flag). Since no login or cookie was used, the page likely expects a secret key passed as a URL or form **parameter**. This section fuzzes for the *name* of that parameter using GET requests.

<a id="102-get-request-fuzzing"></a>
### 10.2 GET Request Fuzzing
GET parameters are appended to a URL after a `?` symbol, in the form `?param1=key`. To discover a valid parameter name, `param1` is replaced with the `FUZZ` keyword, and a dedicated parameter-name wordlist from SecLists is used: `/opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt`.

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx
```
- `-w .../burp-parameter-names.txt:FUZZ` — loads a wordlist of common parameter names (drawn from Burp Suite's known parameter list), bound to `FUZZ`.
- `-u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key` — sends a GET request where the parameter *name* itself is fuzzed (the value is fixed as the placeholder `key`); `ffuf` tries each wordlist entry as the parameter name (e.g. `?id=key`, `?user=key`, `?debug=key`, etc.).
- `-fs xxx` — filters out the known default/uninteresting response size (the placeholder `xxx` represents whatever size was identified as the "no effect" baseline response, following the same filtering approach introduced in Section 9).

**Step-by-step task:**
1. Identify the target page requiring a parameter (`admin/admin.php`) via a recursive scan.
2. Select the Burp parameter-names wordlist from SecLists.
3. Build the URL with `FUZZ` as the parameter name and a placeholder value (`key`).
4. Determine the baseline response size for an incorrect/nonexistent parameter and filter it out with `-fs`.
5. Run the scan and review any remaining (non-filtered) hits.
6. Manually visit the URL using the discovered parameter name to test its effect.

**Result:** One hit was returned. Visiting the page with that discovered parameter name set (`?REDACTED=key`) did not reveal the flag — instead, it indicated that this particular parameter was **deprecated** and no longer functional, signaling the need to look elsewhere (covered in the next section, via POST requests).

---

<a id="11-parameter-fuzzing---post"></a>
## 11. Parameter Fuzzing - POST

<a id="111-overview"></a>
### 11.1 Overview
Having found that the only GET parameter result was deprecated, this section pivots to testing **POST** parameters instead, which are not visible in the URL and must be fuzzed differently.

<a id="112-post-vs-get"></a>
### 11.2 POST vs. GET
Unlike GET parameters, POST parameters are sent in the request's data/body, not appended to the URL after a `?`. The module points to a separate "Web Requests" module for deeper background on HTTP request structure.

<a id="113-building-the-post-fuzzing-command"></a>
### 11.3 Building the POST Fuzzing Command
To fuzz POST data, the `-d` flag carries the data payload (with `FUZZ` embedded inside it), and `-X POST` sets the HTTP method explicitly. A tip notes that PHP backends generally expect POST data in the `application/x-www-form-urlencoded` content type, so that header should be set explicitly too.

```
ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
- `-w .../burp-parameter-names.txt:FUZZ` — same parameter-name wordlist as the GET fuzzing step.
- `-u http://admin.academy.htb:PORT/admin/admin.php` — target URL; no query string needed since POST data goes in the body.
- `-X POST` — forces `ffuf` to send HTTP POST requests instead of the default GET.
- `-d 'FUZZ=key'` — the POST body; `FUZZ` is substituted with each wordlist entry as the parameter *name*, while `key` remains the placeholder value.
- `-H 'Content-Type: application/x-www-form-urlencoded'` — sets the content type header so the PHP backend correctly parses the POST body.
- `-fs xxx` — filters out the known baseline/uninteresting response size, same approach as before.

**Step-by-step task:**
1. Reuse the same parameter-name wordlist used for GET fuzzing.
2. Switch the HTTP method to POST with `-X POST`.
3. Move the fuzzed parameter into the POST body using `-d 'FUZZ=key'`.
4. Add the correct `Content-Type` header so the server parses the data properly.
5. Filter out the baseline response size with `-fs`.
6. Run the scan and inspect any hits.

**Result:** Two hits returned this time — the same previously-found (deprecated) parameter, plus a new one: `id`. This indicates `id` is a genuine, currently-active parameter accepted by the page.

<a id="114-verifying-with-curl"></a>
### 11.4 Verifying with curl
To manually confirm the `id` parameter's effect, a direct POST request is sent using `curl`.

```
curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'
```
- `curl` — command-line tool for making HTTP requests directly.
- `http://admin.academy.htb:PORT/admin/admin.php` — the target URL.
- `-X POST` — sends the request as an HTTP POST.
- `-d 'id=key'` — sends `id=key` as the POST body, testing the `id` parameter with the placeholder value `key`.
- `-H 'Content-Type: application/x-www-form-urlencoded'` — ensures the server correctly interprets the POST body format.

**Result:** The response changed from the earlier access-denied style message to `"Invalid id!"` — confirming that `id` is a real, actively-checked parameter, but the value `key` is not valid. This sets up the final step: finding the *correct* value for `id`, covered in the next section.

---

<a id="12-value-fuzzing"></a>
## 12. Value Fuzzing

<a id="121-overview"></a>
### 12.1 Overview
With the `id` parameter confirmed as real but requiring a correct value, this final practical section covers fuzzing for parameter *values* rather than parameter names — including how to build a custom wordlist when no pre-made one fits the situation.

<a id="122-custom-wordlist"></a>
### 12.2 Custom Wordlist
Pre-made wordlists don't exist for every possible parameter value, since the expected value type varies per parameter (e.g. usernames, IDs, tokens). For some cases — like usernames — a suitable pre-made wordlist may already exist in SecLists, or one can be built from known/likely users of the target site. For other cases — like this custom `id` parameter — a wordlist may need to be generated manually.

Since `id` is likely a numeric value (and such IDs are often sequential, e.g. 1 through 1000 or more), the module builds a simple numeric wordlist using a Bash loop.

```
for i in $(seq 1 1000); do echo $i >> ids.txt; done
```
- `for i in $(seq 1 1000); do ... done` — a Bash `for` loop that iterates the variable `i` over every number from 1 to 1000, generated by the `seq 1 1000` command.
- `echo $i` — prints the current number.
- `>> ids.txt` — appends that number as a new line into the file `ids.txt` (rather than overwriting it each time, which a single `>` would do).
- **Net effect:** after the loop finishes, `ids.txt` contains the numbers 1 through 1000, one per line — a ready-made numeric wordlist.

Verifying the file's contents:
```
cat ids.txt
```
- `cat` — prints the full contents of the file to the terminal, used here simply to confirm the wordlist was generated correctly (showing `1, 2, 3, 4, 5, 6, ...` and so on).

<a id="123-value-fuzzing"></a>
### 12.3 Value Fuzzing
With the numeric wordlist ready, the same POST-fuzzing structure from the previous section is reused — except this time `FUZZ` is placed in the parameter's *value* position instead of its name, using the known `id` parameter name.

```
ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx
```
- `-w ids.txt:FUZZ` — loads the custom numeric wordlist just created, bound to `FUZZ`.
- `-u http://admin.academy.htb:PORT/admin/admin.php` — the same target page confirmed earlier to accept the `id` parameter.
- `-X POST` — sends requests as HTTP POST, matching how the parameter was discovered.
- `-d 'id=FUZZ'` — this time, the parameter *name* (`id`) is fixed, and `FUZZ` fuzzes the *value*, testing every number from the wordlist as a candidate ID.
- `-H 'Content-Type: application/x-www-form-urlencoded'` — required content type for the PHP backend to parse the POST data correctly.
- `-fs xxx` — filters out the known baseline response size corresponding to an invalid ID (e.g. the "Invalid id!" response size), so only genuinely different (likely correct) responses remain visible.

**Step-by-step task:**
1. Determine the expected data type for the target parameter (here, numeric, based on context).
2. Generate a custom wordlist of candidate values using a Bash loop (`for i in $(seq 1 1000); do echo $i >> ids.txt; done`).
3. Verify the wordlist was created correctly with `cat ids.txt`.
4. Reuse the POST fuzzing command structure, but swap `FUZZ` into the parameter's *value* slot while fixing the parameter name (`id=FUZZ`).
5. Filter out the known "invalid" response size with `-fs` to surface only meaningful hits.
6. Run the scan and identify any value that produces a different response.
7. Confirm manually with a direct `curl` POST request using the discovered correct value, to retrieve the final flag content.

**Result:** A hit was returned almost immediately — an `id` value that produced a response different from the filtered "invalid" baseline. The final step (left to the reader) is to send one more `curl` POST request using that confirmed `id` value, mirroring the verification method used in the previous section, in order to retrieve and collect the flag.

---

<a id="13-summary-of-the-overall-workflow"></a>
## 13. Summary of the Overall Workflow

Across the twelve sections, the module builds one continuous investigation using `ffuf`:
1. **Directory fuzzing** found `/blog`.
2. **Extension + page fuzzing** found `/blog/index.php` (empty) and a second PHP file revealing the admin panel had moved to `academy.htb`.
3. **Recursive fuzzing** automated steps 1–2 across an entire site tree in one command.
4. **`/etc/hosts` editing** made the private domain `academy.htb` resolvable locally.
5. **Subdomain fuzzing** worked for public domains but failed for the private `academy.htb`, since it depends on public DNS.
6. **VHost fuzzing** (fuzzing the `Host:` header) succeeded where subdomain fuzzing failed, discovering `admin.academy.htb` without needing any DNS record.
7. **Response-size filtering** was necessary to cut through false-positive 200 responses in the VHost scan.
8. **GET parameter fuzzing** found one parameter, but it was deprecated.
9. **POST parameter fuzzing** found a working parameter, `id`.
10. **Custom wordlist generation + value fuzzing** found the correct numeric value for `id`, ultimately leading to the flag.
