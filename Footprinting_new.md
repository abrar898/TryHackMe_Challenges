# Network Services — Complete Penetration Testing Notes

> **Rules for these notes:**
> - Simple English throughout
> - Every heading has minimum 6-7 lines of description
> - Every command explained with purpose, usage, and full demonstration
> - Nothing skipped — complete coverage of every topic

---

# Table of Contents

1. [Enumeration Methodology](#1-enumeration-methodology)
2. [Domain Information](#2-domain-information)
3. [Cloud Resources](#3-cloud-resources)
4. [Staff — OSINT on Employees](#4-staff--osint-on-employees)
5. [FTP (File Transfer Protocol)](#5-ftp-file-transfer-protocol)
6. [SMB (Server Message Block)](#6-smb-server-message-block)
7. [NFS (Network File System)](#7-nfs-network-file-system)
8. [DNS (Domain Name System)](#8-dns-domain-name-system)
9. [SMTP (Simple Mail Transfer Protocol)](#9-smtp-simple-mail-transfer-protocol)
10. [IMAP / POP3](#10-imap--pop3)
11. [SNMP (Simple Network Management Protocol)](#11-snmp-simple-network-management-protocol)
12. [MySQL](#12-mysql)
13. [MSSQL (Microsoft SQL Server)](#13-mssql-microsoft-sql-server)
14. [Oracle TNS](#14-oracle-tns)
15. [IPMI (Intelligent Platform Management Interface)](#15-ipmi-intelligent-platform-management-interface)
16. [Linux Remote Management Protocols](#16-linux-remote-management-protocols)
17. [Windows Remote Management Protocols](#17-windows-remote-management-protocols)

---

# 1. Enumeration Methodology

## What is Enumeration Methodology?

Enumeration methodology is a standardized, step-by-step approach used during penetration testing to gather information about a target system in an organized way. Without a proper methodology, a penetration tester might miss important parts of the target or waste time going in the wrong direction. Because penetration testing involves many different types of systems, networks, and configurations, having a fixed but flexible structure helps the tester stay on track. The methodology is not a rigid checklist but rather a systematic framework that allows for changes depending on what the tester finds. It is divided into three levels: Infrastructure-based enumeration, Host-based enumeration, and OS-based enumeration. These three levels together cover everything from the internet presence of a company to the internal operating system configuration of its servers.

## The Three Levels of Enumeration

| Level | Focus |
|-------|-------|
| Infrastructure-based | Internet presence, gateways, accessible services |
| Host-based | Processes, privileges running on the host |
| OS-based | Internal OS setup, configuration files, sensitive data |

---

## The 6 Layers of Enumeration

The entire enumeration process is organized into 6 layers, each representing a boundary or "wall" that the tester tries to pass through to get closer to the target. Think of it like a labyrinth where each layer is a ring, and the tester needs to find the gap in each ring to move forward. Each layer has specific goals and information categories that the tester focuses on.

| Layer | Description | Information Categories |
|-------|-------------|----------------------|
| 1. Internet Presence | Identify internet presence and externally accessible infrastructure | Domains, Subdomains, vHosts, ASN, Netblocks, IP Addresses, Cloud Instances, Security Measures |
| 2. Gateway | Identify security measures protecting external and internal infrastructure | Firewalls, DMZ, IPS/IDS, EDR, Proxies, NAC, Network Segmentation, VPN, Cloudflare |
| 3. Accessible Services | Identify accessible interfaces and services hosted externally or internally | Service Type, Functionality, Configuration, Port, Version, Interface |
| 4. Processes | Identify internal processes, sources, and destinations associated with services | PID, Processed Data, Tasks, Source, Destination |
| 5. Privileges | Identify internal permissions and privileges to accessible services | Groups, Users, Permissions, Restrictions, Environment |
| 6. OS Setup | Identify internal components and systems setup | OS Type, Patch Level, Network config, OS Environment, Configuration files, Sensitive private files |

### Layer 1 — Internet Presence

This is the outermost layer. Here the tester finds out what the company looks like from the internet. The goal is to find all possible targets such as domains, subdomains, IP addresses, cloud instances, and other infrastructure that is visible on the internet. This layer is especially important in black-box tests where no information is given upfront. The tester uses passive techniques — meaning they do not directly connect to the target — to gather information. Tools like crt.sh, Shodan, and DNS lookups are used at this stage.

### Layer 2 — Gateway

Once the internet presence is identified, the next step is to understand how the company protects its infrastructure. This layer focuses on identifying the protective measures between the internet and the internal network. The goal is to understand what kind of security is in place — firewalls, DMZ zones, IPS/IDS systems, EDR tools, proxies, NAC, and VPNs. Understanding the gateway helps the tester know what kind of traffic is blocked and what might pass through without triggering alarms.

### Layer 3 — Accessible Services

This is the layer that is mostly covered in the main footprinting module. Here the tester identifies all services running on the target systems that can be accessed from outside. Each service has a specific purpose — FTP for file transfer, SSH for remote access, HTTP for web traffic. The tester looks at the service type, version, configuration, port, and functionality. The goal is to understand how the service works so it can be communicated with and potentially exploited.

### Layer 4 — Processes

Every time a command or function is executed on a server, some process runs behind the scenes. This layer focuses on identifying what internal processes are running, what data they handle, and what sources and destinations are involved. For example, a web server process might be processing requests from users and sending them to a database. Understanding these processes helps the tester identify dependencies and potential weak points. Key information includes PID, type of data being processed, tasks, and source/destination details.

### Layer 5 — Privileges

Every service on a system runs under a specific user account with specific permissions. Sometimes administrators give services more permissions than needed, or they forget to review permission settings. This layer is about identifying what permissions are attached to each service and each user, and whether those permissions can be misused. This is especially important in Active Directory environments where users might have permissions across many systems.

### Layer 6 — OS Setup

This is the innermost layer. At this point, the tester already has internal access to the system and is gathering information about the operating system itself. This includes the OS type, patch level, network configuration, environment variables, configuration files, and sensitive private files. The goal is to understand how the administrator has set up and maintains the system, and to find any sensitive internal information that can be used for further exploitation.

---

# 2. Domain Information

## What is Domain Information?

Domain information is one of the most important parts of the passive reconnaissance phase in penetration testing. It is not only about finding subdomains but about understanding the full internet presence of a company — what technologies they use, what services they run, and how their infrastructure is structured. This entire process is done passively, meaning the tester acts like a normal internet user and does not send any direct scans or requests to the target that could alert the company. The information gathered here gives a big-picture view of the target before any active testing begins. Think of it as reading everything that is publicly available about the company without knocking on their door.

---

## Checking the Company Website

The first step is always to visit the company's official website and read the content carefully. The website tells you what kind of services the company offers, what technologies they use, and what their business model is. For example, if a company offers IoT solutions, app development, and cloud hosting, then you know they likely run servers, APIs, and possibly cloud storage. Reading the website with a developer's eye helps you guess what kind of backend infrastructure might be in place. Even small details like "built with Django" or "powered by AWS" in a footer or page source code reveal important technical information.

---

## SSL Certificates and crt.sh

### What is crt.sh?

crt.sh is a website that stores Certificate Transparency logs. When any website gets an SSL certificate from a Certificate Authority like Let's Encrypt, a public log entry is created. This log can be searched to find all subdomains that a company has ever used a certificate for. This is extremely useful because companies often have many subdomains — some of which might be forgotten or misconfigured. By searching crt.sh for a domain, you can find subdomains that are not listed anywhere else publicly.

### Query crt.sh via Command Line

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

**Purpose:** Fetches JSON data from crt.sh about all SSL certificates issued for the target domain. The `jq .` part formats the JSON output to make it readable. Each entry shows the certificate's common name, issuer, validity dates, and serial number.

### Filter Unique Subdomains Only

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

**Purpose:** Filters only subdomain names from the JSON response. Removes certificate CN fields, extracts clean subdomain names, and sorts them uniquely.

**Example output:**
```
account.ttn.inlanefreight.com
blog.inlanefreight.com
matomo.inlanefreight.com
shop.inlanefreight.com
www.inlanefreight.com
```

---

## Finding Company-Hosted Servers

### Purpose

After getting a list of subdomains, the next step is to find which ones resolve to IP addresses that belong to the company itself and not third-party providers. This matters because you can only test hosts within your authorized scope. Third-party hosted services require separate permission.

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4; done
```

**Purpose:** Loops through each subdomain, resolves its IP address using the `host` command, and filters only those belonging to the target domain. Shows subdomain + IP pairs.

**Example output:**
```
blog.inlanefreight.com 10.129.24.93
inlanefreight.com 10.129.27.33
matomo.inlanefreight.com 10.129.127.22
www.inlanefreight.com 10.129.127.33
s3-website-us-west-2.amazonaws.com 10.129.95.250
```

---

## Using Shodan for IP Intelligence

### What is Shodan?

Shodan is a search engine for internet-connected devices. Unlike Google which indexes web content, Shodan indexes open ports, service banners, and device information. It reveals what services are running on a given IP, what software versions, and what ports are open — all without directly scanning the target yourself.

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f4 >> ip-addresses.txt; done
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

**Purpose:** First command builds a list of IP addresses from the subdomains. Second command queries Shodan for each IP, returning open ports, software versions, SSL details, and geographic information.

**Example Shodan output:**
```
10.129.27.33
City:                    Berlin
Country:                 Germany
Organization:            InlaneFreight
Number of open ports:    3
Ports:
     22/tcp OpenSSH (7.6p1 Ubuntu-4ubuntu0.3)
     80/tcp nginx
    443/tcp nginx
        |-- SSL Versions: TLSv1.2
```

---

## DNS Records with dig

### What is dig?

`dig` (Domain Information Groper) is a command-line tool used to query DNS servers and retrieve DNS records. DNS records are like a phonebook for the internet — they map domain names to IP addresses and other information.

```bash
dig any inlanefreight.com
```

**Purpose:** Retrieves ALL available DNS record types for the domain in one query.

### DNS Record Types Explained

| Record | Purpose | What It Reveals |
|--------|---------|----------------|
| A | Maps domain to IPv4 | Server IP addresses |
| MX | Mail servers | Who handles company email (Google, Exchange, etc.) |
| NS | Name servers | Who hosts the DNS (reveals hosting provider) |
| TXT | Text records | Third-party service verifications, SPF, DMARC, DKIM |
| SOA | Zone authority | Admin email, zone settings |

### What TXT Records Reveal

```
atlassian-domain-verification=...   → Company uses Jira/Confluence/Bitbucket
google-site-verification=...        → Uses Google services; possible open GDrive links
logmein-verification-code=...       → Uses LogMeIn for remote access management
v=spf1 include:mailgun.org         → Uses Mailgun email API (check for IDOR/SSRF)
include:spf.protection.outlook.com  → Uses Microsoft 365/Azure (check Azure storage)
MS=ms92346782372                    → Microsoft account verification
```

---

# 3. Cloud Resources

## What are Cloud Resources in Penetration Testing?

Modern companies use cloud services from AWS, Google Cloud (GCP), and Microsoft Azure for storage, computing, and hosting. While cloud providers themselves are very secure, the configurations that companies apply can introduce vulnerabilities. The most common issue is misconfigured storage — such as AWS S3 buckets, Azure Blobs, or GCP Cloud Storage — that are set to public access without authentication. During a penetration test, discovering these misconfigured cloud resources can lead to accessing sensitive files like documents, source code, private keys, and even credentials. Finding cloud resources is always done passively — you use public search engines and databases rather than directly attacking anything.

---

## Finding Cloud Storage with Google Dorks

### What are Google Dorks?

Google Dorks are advanced Google search operators that narrow down search results to very specific types of pages. They help find public files stored in AWS or Azure that have been indexed by Google.

**For AWS S3 Buckets:**
```
intext:companyname inurl:amazonaws.com
```

**For Azure Blobs:**
```
intext:companyname inurl:blob.core.windows.net
```

**Purpose:** These searches return PDF files, documents, images, and other files that companies have accidentally made public in their cloud storage. The results often reveal sensitive business documents.

---

## Finding Cloud Resources in Website Source Code

Companies often load images, JavaScript files, and CSS from their cloud storage to reduce load on web servers. If you inspect the HTML source code of a company's website (Ctrl+U in browser), you might find links to `blob.core.windows.net` or `s3.amazonaws.com` which reveal cloud storage URLs. These can then be investigated to see if they allow public access or directory listing.

---

## Third-Party Tools for Cloud Enumeration

### Domain.Glass

Domain.Glass is an online tool that provides information about a domain's infrastructure. It can reveal cloud service providers, CDN usage (like Cloudflare), and basic security assessment information. If Cloudflare is detected, it is noted as a security measure in Layer 2 (Gateway) of the enumeration methodology. Visit: `https://domain.glass`

### GrayHatWarfare

GrayHatWarfare is a specialized search engine for public cloud storage. It indexes publicly accessible files in AWS S3, Azure Blob, and GCP Cloud Storage. A tester can search by company name, filter by file type (PDF, DOCX, XLSX, etc.), and discover what data has been accidentally made public. Visit: `https://grayhatwarfare.com`

---

## Risk: Leaked SSH Private Keys

One of the most dangerous things that can appear in misconfigured cloud storage is leaked SSH private keys. An SSH private key (typically named `id_rsa`) allows direct login to servers without needing a password. If an employee accidentally uploaded their private key to a public S3 bucket, an attacker can download it and use it to log into any server that has the corresponding public key installed. This is a very real risk that happens when employees are under pressure, make mistakes, or do not understand the consequences of sharing certain files.

---

# 4. Staff — OSINT on Employees

## Why Look at Employees?

Employees of a company are a goldmine of information for penetration testers doing OSINT (Open Source Intelligence). By looking at LinkedIn profiles, GitHub accounts, job postings, and professional websites, you can learn what technologies and tools the company uses, what the internal team structure looks like, what programming languages and frameworks developers work with, and even find leaked credentials or sensitive code in public repositories. This information helps you understand the attack surface and plan your approach before any active testing begins.

---

## LinkedIn Job Posts

Job posts reveal a lot about internal technology stacks. For example, if a job requires experience with Django, Flask, PostgreSQL, and Kubernetes, you know the company runs Python web applications on a container infrastructure with a PostgreSQL database.

**Example technologies revealed by job posts:**
- Programming languages: Java, C#, C++, Python, Ruby, PHP, Perl
- Databases: PostgreSQL, MySQL, Oracle
- Frameworks: Flask, Django, ASP.NET, Spring
- Tools: Atlassian Suite (Jira/Confluence), Git, Docker, Kubernetes
- Cloud: AWS, Azure, GCP

---

## LinkedIn Employee Profiles

Individual employee profiles show their skills, past projects, and linked GitHub accounts. From a developer's GitHub profile, you might find:
- Open source code containing company-specific logic
- Hardcoded API keys or JWT tokens
- Database connection strings
- Personal email addresses tied to company systems

---

## Risk: Hardcoded Credentials in Public Code

A very common finding is that developers push code to GitHub that contains hardcoded secrets. For example, a JWT (JSON Web Token) with a hardcoded secret key found in a public GitHub repo could allow an attacker to forge authentication tokens for the company's web application. Key indicators to look for:
- `.env` files committed by accident
- Database passwords in configuration files
- AWS access keys in code
- Private API keys in JavaScript files

---

# 5. FTP (File Transfer Protocol)

## What is FTP?

FTP (File Transfer Protocol) is one of the oldest internet protocols, operating at the application layer of the TCP/IP stack alongside HTTP and POP3. It is used to transfer files between a client and a server. FTP opens two separate connections: a control channel on TCP port 21 (for sending commands and receiving status codes) and a data channel on TCP port 20 (for actual file transfer). If a connection is interrupted during a file transfer, FTP can resume the transfer after reconnecting — this is an important feature that makes FTP practical for large file transfers. FTP normally sends everything including passwords in plain text, making it vulnerable to sniffing on the network.

---

## How FTP Works — Two Channels

**Channel 1 — Control Channel (TCP Port 21):**
This is where all the commands and responses happen. When you connect to an FTP server, this channel is opened first. The client sends commands like LIST, GET, PUT, and the server responds with status codes. This channel stays open for the entire FTP session.

**Channel 2 — Data Channel (TCP Port 20):**
This is where the actual file data flows. Every time you list a directory or transfer a file, a separate data connection is opened for that specific operation and then closed afterward. This separation of control and data is a key design feature of FTP.

---

## Active Mode vs Passive Mode

### Active Mode
The server **initiates** the data connection back to the client after the client tells it which port to use. Problem: firewalls protecting the client may block incoming connections from the server, breaking the data transfer.

### Passive Mode
The server announces an available port and the **client initiates** the data connection. Because the client is always the one starting connections, firewalls do not block anything. This is the standard for internet FTP today.

---

## TFTP (Trivial File Transfer Protocol)

### What is TFTP?

TFTP is a much simpler version of FTP. It uses UDP instead of TCP, making it faster but unreliable (handles lost packets at application layer). TFTP has no authentication at all — no usernames, no passwords. It relies entirely on file system read/write permissions. Because of this total lack of security, TFTP should only be used on local, protected, trusted networks — never on the internet. It is commonly used for booting network devices, loading firmware, and distributing configuration files.

### TFTP Commands

| Command | Description | Usage |
|---------|-------------|-------|
| `connect` | Sets the remote host and optionally port for transfers | `connect 10.129.14.136` |
| `get` | Downloads file(s) from remote host to local machine | `get config.txt` |
| `put` | Uploads file(s) from local machine to remote host | `put update.bin` |
| `quit` | Exits TFTP | `quit` |
| `status` | Shows current TFTP session status (mode, connection, timeout) | `status` |
| `verbose` | Toggles verbose mode showing extra transfer info | `verbose` |

> **Important:** Unlike FTP, TFTP has NO directory listing command. You must know the exact filename before downloading.

---

## vsFTPd — Default FTP Server on Linux

### What is vsFTPd?

vsFTPd (Very Secure FTP Daemon) is the most commonly used FTP server software on Linux-based systems. The name "Very Secure" refers to its design philosophy of security-first. It is the default FTP server on many Linux distributions including Ubuntu and CentOS.

### Installing vsFTPd

```bash
sudo apt install vsftpd
```

**Purpose:** Installs the vsFTPd FTP server package. After installation, the service starts automatically and the configuration file is created at `/etc/vsftpd.conf`.

---

## Viewing the vsFTPd Configuration File

```bash
cat /etc/vsftpd.conf | grep -v "#"
```

**Purpose:** Displays the vsFTPd configuration file filtering out all comment lines so you only see the active settings. The `grep -v "#"` means "show everything EXCEPT lines containing #".

### Key Configuration Settings

| Setting | Description |
|---------|-------------|
| `listen=NO` | Run from inetd (NO) or as standalone daemon (YES) |
| `listen_ipv6=YES` | Enable IPv6 listening |
| `anonymous_enable=NO` | Allow anonymous login without credentials |
| `local_enable=YES` | Allow local system users to log in |
| `dirmessage_enable=YES` | Show .message file contents when entering directories |
| `use_localtime=YES` | Use server's local timezone for file timestamps |
| `xferlog_enable=YES` | Log all file uploads and downloads |
| `connect_from_port_20=YES` | Use port 20 for active mode data connections |
| `secure_chroot_dir=/var/run/vsftpd/empty` | Empty directory for security chroot |
| `pam_service_name=vsftpd` | PAM authentication configuration name |
| `rsa_cert_file=...` | Path to SSL certificate for encrypted connections |
| `ssl_enable=NO` | Enable/disable SSL/TLS encryption |

---

## The /etc/ftpusers File

### What is /etc/ftpusers?

This file is a **deny list** — it contains usernames of system users who are specifically prohibited from using FTP, even if they have valid system accounts and correct passwords. This is a security feature to prevent certain powerful or sensitive system accounts from accessing the FTP service.

```bash
cat /etc/ftpusers
```

**Example output:**
```
guest
john
kevin
```

These users cannot log into FTP even with correct passwords.

---

## Dangerous FTP Settings

These settings create security vulnerabilities that penetration testers specifically look for:

| Setting | Risk | Description |
|---------|------|-------------|
| `anonymous_enable=YES` | HIGH | Anyone can log in without credentials |
| `anon_upload_enable=YES` | CRITICAL | Anonymous users can upload files |
| `anon_mkdir_write_enable=YES` | HIGH | Anonymous users can create directories |
| `no_anon_password=YES` | HIGH | No password asked for anonymous login |
| `anon_root=/home/username/ftp` | MEDIUM | Sets root directory for anonymous users |
| `write_enable=YES` | MEDIUM | Enables upload/delete/rename FTP commands |
| `hide_ids=YES` | LOW | Hides real user/group IDs (shows "ftp") |
| `ls_recurse_enable=YES` | MEDIUM | Allows recursive directory listing |

---

## Connecting to FTP — Anonymous Login

```bash
ftp 10.129.14.136
```

**What happens step by step:**
1. FTP client connects to server on port 21
2. Server responds with **220 banner** (welcome message with software name/version)
3. Prompted for username — type `anonymous` and press Enter
4. Prompted for password — press Enter (anything works for anonymous)
5. If allowed: receive code **230** meaning "Login successful"
6. Server reports remote system type and transfer mode

**Full example:**
```
Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

> **Note:** The 220 banner often reveals FTP software name and version — valuable for finding known vulnerabilities.

---

## FTP Commands Inside a Session

### ls — List Directory Contents

```ftp
ftp> ls
```

**Purpose:** Lists all files and directories in the current directory. Shows permissions, owner IDs, file sizes, dates, and names.

**Example output:**
```
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

Column meanings: permissions, links, owner-UID, group-GID, size-in-bytes, date, filename. `d` = directory, `-` = file.

---

### status — Check Current FTP Session Settings

```ftp
ftp> status
```

**Purpose:** Displays the current configuration of your FTP client session — what mode you are in, transfer type (binary/ASCII), and what optional features are enabled.

**Example output:**
```
Connected to 10.129.14.136.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Hash mark printing: off; Use of PORT cmds: on
```

---

### debug — Enable Debug Mode

```ftp
ftp> debug
```

**Purpose:** Turns on debugging in the FTP client. When debug mode is on, the client prints every raw command it sends to the server before sending it. This lets you see the actual FTP protocol communication happening behind the scenes — useful for understanding the protocol and troubleshooting.

**After enabling:**
```
Debugging on (debug=1).
```

Now every command shows the raw protocol being sent, like `---> PORT 10,10,14,4,188,195`.

---

### trace — Enable Packet Tracing

```ftp
ftp> trace
```

**Purpose:** Turns on packet-level tracing — even more detailed than debug mode. Shows every network packet being sent and received. Used for deep protocol analysis and troubleshooting at the network level.

**After enabling:**
```
Packet tracing on.
```

---

### debug + trace — Seeing Raw Protocol Communication

```ftp
ftp> debug
Debugging on (debug=1).

ftp> trace
Packet tracing on.

ftp> ls
---> PORT 10,10,14,4,188,195
200 PORT command successful. Consider using PASV.
---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
226 Directory send OK.
```

Now you can see the PORT command (tells server where to send data) and the LIST command (requests directory listing) being sent as raw FTP protocol commands.

---

### ls -R — Recursive Directory Listing

```ftp
ftp> ls -R
```

**Purpose:** Lists ALL files and directories recursively — goes into every subfolder and lists everything in one command. Requires `ls_recurse_enable=YES` on the server. For penetration testing, this gives you the ENTIRE file structure in one command.

**Example output:**
```
.:
-rw-rw-r--    1 ftp  ftp  8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp  ftp     4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp  ftp     4096 Sep 14 16:50 Documents

./Clients:
drwx------    2 ftp  ftp     4096 Sep 16 18:04 HackTheBox
drwxrwxrwx    2 ftp  ftp     4096 Sep 16 18:00 Inlanefreight

./Clients/HackTheBox:
-rw-r--r--    1 ftp  ftp    34872 Sep 16 18:04 appointments.xlsx
-rw-r--r--    1 ftp  ftp   498123 Sep 16 18:04 contract.docx
-rw-r--r--    1 ftp  ftp   478237 Sep 16 18:04 contract.pdf
```

> When `hide_ids=YES` is set, all owner IDs show as "ftp" instead of real UIDs.

---

### get — Download a Single File

```ftp
ftp> get "Important Notes.txt"
```

**Purpose:** Downloads a specific file from the FTP server to your local machine. Use quotes or backslash-escape spaces in filenames.

**Full example:**
```ftp
ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)

ftp> exit
221 Goodbye.
```

**Verify download:**
```bash
ls | grep Notes.txt
'Important Notes.txt'
```

---

## Downloading All Files at Once with wget

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

**Purpose:** Downloads the entire FTP server content in one command.

**Flag breakdown:**
- `-m` — Mirror mode: recursively downloads everything, preserving directory structure
- `--no-passive` — Forces Active Mode (uses PORT commands)
- `ftp://anonymous:anonymous@10.129.14.136` — Format: `protocol://username:password@server`

**Example output:**
```
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/
Connecting to 10.129.14.136:21... connected.
Logging in as anonymous ... Logged in!
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```

> **Warning:** Downloading ALL files may trigger security alerts. Use carefully during authorized tests.

### View Downloaded File Structure

```bash
tree .
```

**Output:**
```
.
└── 10.129.14.136
    ├── Calendar.pptx
    ├── Clients
    │   └── Inlanefreight
    │       ├── appointments.xlsx
    │       ├── contract.docx
    │       ├── meetings.txt
    │       └── proposal.pptx
    ├── Documents
    │   ├── appointments-template.xlsx
    │   └── contract-template.docx
    └── Important Notes.txt
```

---

## put — Upload a File to FTP Server

### Why Uploading Matters

If you can upload files to an FTP server connected to a web server, you can potentially upload a web shell — a script that lets you run system commands through the browser. This is a common path to gaining Remote Command Execution (RCE).

### Step 1 — Create a Test File

```bash
touch testupload.txt
```

**Purpose:** Creates an empty test file locally. Used as a safe, harmless test to check whether the FTP server allows uploads.

### Step 2 — Upload the File

```ftp
ftp> put testupload.txt

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.
```

**Purpose:** `put` uploads a local file to the current directory on the FTP server. The FTP server sends the STOR command. A successful upload (code 226) is a major finding in a penetration test.

### Step 3 — Verify the Upload

```ftp
ftp> ls
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
```

The `testupload.txt` now appears — upload confirmed.

---

## Footprinting FTP with Nmap

### Update NSE Script Database

```bash
sudo nmap --script-updatedb
```

**Purpose:** Updates Nmap's NSE script database to ensure the latest scripts are available.

**Output:**
```
NSE: Updating rule database.
NSE: Script Database updated successfully.
```

---

### Find FTP-Related NSE Scripts

```bash
find / -type f -name ftp* 2>/dev/null | grep scripts
```

**Purpose:** Finds all Nmap scripts whose names start with "ftp". The `2>/dev/null` hides permission errors.

**Output:**
```
/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse
```

| Script | Purpose |
|--------|---------|
| `ftp-anon` | Checks if anonymous login is allowed; lists root directory |
| `ftp-syst` | Runs STAT command to get FTP server status and version |
| `ftp-brute` | Brute forces FTP credentials |
| `ftp-vsftpd-backdoor` | Checks for vsFTPd 2.3.4 backdoor (gives root shell) |
| `ftp-proftpd-backdoor` | Checks for ProFTPd backdoors |
| `ftp-bounce` | Tests for FTP bounce attack vulnerability |
| `ftp-vuln-cve2010-4221` | Checks ProFTPd buffer overflow vulnerability |

---

### Full Nmap FTP Scan

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

**Flag breakdown:**
- `-sV` — Detect service version
- `-p21` — Scan only port 21
- `-sC` — Run default scripts (includes ftp-anon, ftp-syst)
- `-A` — Aggressive: OS detection + version + scripts + traceroute

**Example output:**
```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx  1 ftp ftp  8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx  4 ftp ftp     4096 Sep 16 17:57 Clients [NSE: writeable]
| -rwxrwxrwx  1 ftp ftp       41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx  1 ftp ftp        0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

**What this reveals:** Anonymous login allowed, all files are writeable (critical!), exact vsFTPd version (3.0.3), plain text connections (no encryption).

---

### Nmap Script Trace

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

**Purpose:** Shows EVERY network interaction during NSE script execution at the raw packet level. Reveals exact bytes sent/received, connection timings, and raw server responses.

**Key output:**
```
NSOCK INFO [11.4660s] Callback: READ SUCCESS for EID 50 [10.129.14.136:21]
(41 bytes): 220 Welcome to HTB-Academy FTP service...
NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

---

### Connecting with Netcat

```bash
nc -nv 10.129.14.136 21
```

**Purpose:** Opens a raw TCP connection to the FTP server using netcat. Bypasses all FTP client features and lets you communicate with raw text commands.

- `-n` — No hostname resolution (use IP directly)
- `-v` — Verbose (shows connection status)

---

### Connecting with Telnet

```bash
telnet 10.129.14.136 21
```

**Purpose:** Opens a plain text connection to the FTP server using Telnet. Similar to netcat but uses the Telnet protocol. Both netcat and Telnet are useful when you want direct text-based interaction with the FTP server protocol.

---

### FTP with TLS/SSL — Using OpenSSL

```bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

**Purpose:** Connects to an FTPS (FTP over SSL) server. The `-starttls ftp` flag tells OpenSSL to first connect in plain text, then send the FTP STARTTLS command to upgrade to encrypted.

**What the SSL certificate reveals:**
```
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev,
CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
```

- **O = Inlanefreight** → Organization name
- **OU = Dev** → Department (Development team)
- **CN = master.inlanefreight.htb** → Server hostname
- **emailAddress = admin@inlanefreight.htb** → Admin email address

Even without breaking encryption, the SSL certificate leaks the hostname, organization, department, and admin email.

---

# 6. SMB (Server Message Block)

## What is SMB?

SMB (Server Message Block) is a client-server network protocol that controls and manages access to shared resources across a network. These shared resources include files, entire directories, printers, routers, and other network interfaces. SMB was originally developed for Windows operating systems and first became widely available as part of the OS/2 network operating system LAN Manager and LAN Server. Beyond file sharing, SMB also handles communication between different system processes, meaning applications on different machines can exchange data using SMB. Today SMB is used in almost every Windows-based network environment. Because of backward compatibility, newer Windows versions can still communicate with very old Windows systems over SMB. Access rights to shares are controlled by ACLs (Access Control Lists) which can be set very granularly per user or per group.

---

## Samba — SMB for Linux

Samba is the open-source Linux/Unix implementation of the SMB protocol. It allows Linux and Unix systems to participate in Windows networks and share files with Windows machines. Samba implements CIFS (Common Internet File System), which is a dialect of SMB version 1. Modern Samba supports SMB 2 and SMB 3 as well. Samba uses two important background services: `smbd` (file and print services) and `nmbd` (NetBIOS name resolution). When communicating with older NetBIOS services, Samba uses TCP ports 137, 138, and 139. Modern SMB uses TCP port 445 exclusively.

---

## SMB Version History

| Version | OS Support | Features |
|---------|-----------|---------|
| CIFS | Windows NT 4.0 | Communication via NetBIOS interface |
| SMB 1.0 | Windows 2000 | Direct TCP connection |
| SMB 2.0 | Windows Vista, Server 2008 | Performance upgrades, improved message signing, caching |
| SMB 2.1 | Windows 7, Server 2008 R2 | Locking mechanisms |
| SMB 3.0 | Windows 8, Server 2012 | Multichannel, end-to-end encryption, remote storage |
| SMB 3.0.2 | Windows 8.1, Server 2012 R2 | Minor updates |
| SMB 3.1.1 | Windows 10, Server 2016 | Integrity checking, AES-128 encryption |

---

## Default Samba Configuration

```bash
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

**Purpose:** Reads the Samba configuration file filtering out all comment lines. Both `#` and `;` are comment characters in smb.conf.

**Example output:**
```
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   server role = standalone server
   map to guest = bad user
   usershare allow guests = yes

[printers]
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes

[print$]
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
```

---

## Key SMB Share Settings

| Setting | Description |
|---------|-------------|
| `[sharename]` | Share name visible to clients |
| `workgroup = WORKGROUP` | Windows workgroup or domain name |
| `path = /path/here/` | Actual filesystem path on the server |
| `server string = STRING` | Server description shown to clients |
| `unix password sync = yes` | Sync Unix password with SMB password |
| `usershare allow guests = yes` | Allow non-authenticated users |
| `map to guest = bad user` | Treat failed logins as guest |
| `browseable = yes` | Show share in network browse lists |
| `guest ok = yes` | Allow access without password |
| `read only = yes` | Users can only read, not modify |
| `create mask = 0700` | Permissions for newly created files |

---

## Dangerous SMB Settings

| Setting | Risk |
|---------|------|
| `browseable = yes` | Attackers can see all shares without authentication |
| `read only = no` + `guest ok = yes` | Anonymous write access — can upload malicious files |
| `writable = yes` | Enable write access |
| `guest ok = yes` | No password required — anyone can connect |
| `enable privileges = yes` | SID-based privilege escalation possible |
| `create mask = 0777` | All created files get full permissions for everyone |
| `directory mask = 0777` | All created directories get full permissions |
| `logon script = script.sh` | Executes script on every user login |
| `magic script = script.sh` | Executes when script file is closed — auto execution |

---

## Creating a Test Share

Add to `/etc/samba/smb.conf`:
```ini
[notes]
    comment = CheckIT
    path = /mnt/notes/
    browseable = yes
    read only = no
    writable = yes
    guest ok = yes
    enable privileges = yes
    create mask = 0777
    directory mask = 0777
```

---

## Restarting Samba

```bash
sudo systemctl restart smbd
```

**Purpose:** Applies configuration changes to the running Samba service. Always restart after editing smb.conf.

---

## SMBclient — Listing Available Shares

```bash
smbclient -N -L //10.129.14.128
```

**Purpose:** Lists all shared folders available on the target Samba server anonymously.

- `-N` — Null session (no password)
- `-L` — List all shares
- `//10.129.14.128` — Target server in UNC path format

**Example output:**
```
        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available
```

---

## SMBclient — Connecting to a Specific Share

```bash
smbclient //10.129.14.128/notes
```

**Purpose:** Connects to the "notes" share. Press Enter when prompted for password to attempt anonymous/guest access.

```
Enter WORKGROUP\<username>'s password:
Anonymous login successful
smb: \>
```

---

## help — List All Available SMB Commands

```
smb: \> help
```

**Purpose:** Shows all commands available inside the SMB session.

**Key commands:**
- `ls` — List files
- `get` — Download file
- `put` — Upload file
- `cd` — Change directory
- `mkdir` — Create directory
- `rm` — Delete file
- `!` — Run local command without leaving session

---

## ls — List Files in SMB Share

```
smb: \> ls
```

**Example output:**
```
  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

`D` = directory, `N` = normal file. Number after type = file size in bytes.

---

## get — Download a File from SMB

```
smb: \> get prep-prod.txt

getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec)
```

**Purpose:** Downloads the specified file from the SMB share to your local machine.

---

## ! — Run Local Commands Without Leaving SMB

```
smb: \> !ls
prep-prod.txt

smb: \> !cat prep-prod.txt
[] check your code with the templates
[] run code-assessment.py
```

**Purpose:** The `!` prefix runs commands on your LOCAL machine without disconnecting from the SMB session. Very convenient for immediately inspecting downloaded files.

---

## smbstatus — Check Active SMB Connections (Server Side)

```bash
root@samba:~# smbstatus
```

**Purpose:** Run on the Samba server (requires root). Shows all currently active connections: username, IP address, share being accessed, protocol version, and whether encryption/signing is active.

**Example output:**
```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                          Protocol Version  Encryption  Signing
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)  SMB3_11    -           -

Service      pid     Machine       Connected at
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST
```

The `-` under Encryption and Signing means neither is active — a security finding.

---

## Footprinting SMB with Nmap

```bash
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

**Purpose:** Scans both SMB ports. Port 139 = NetBIOS session service (older SMB), port 445 = modern SMB over TCP.

**Example output:**
```
PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
```

`Message signing enabled but not required` → signing can be bypassed → relay attacks possible.

---

## RPCclient — Manual SMB Enumeration

### What is RPC?

RPC (Remote Procedure Call) allows a program on one computer to execute functions on another computer as if they were local. The `rpcclient` tool implements MS-RPC functions for Linux, enabling deep enumeration of Windows/Samba systems.

### Connecting with RPCclient

```bash
rpcclient -U "" 10.129.14.128
```

**Purpose:** Connects anonymously (null session). `-U ""` = empty username. If the server allows null sessions, you get the `rpcclient $>` prompt.

```
Enter WORKGROUP\'s password:    [press Enter]
rpcclient $>
```

---

### srvinfo — Get Server Information

```
rpcclient $> srvinfo
```

**Example output:**
```
DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
platform_id     :       500
os version      :       6.1
server type     :       0x809a03
```

OS version 6.1 = Windows 7 / Windows Server 2008 R2.

---

### enumdomains — List All Domains

```
rpcclient $> enumdomains
```

**Example output:**
```
name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]
```

---

### querydominfo — Get Detailed Domain Information

```
rpcclient $> querydominfo
```

**Example output:**
```
Domain:         DEVOPS
Server:         DEVSMB
Total Users:    2
Total Groups:   0
Server Role:    ROLE_DOMAIN_PDC
```

Reveals domain name, server name, user count, and server role.

---

### netshareenumall — List All Shares with Filesystem Paths

```
rpcclient $> netshareenumall
```

**Example output:**
```
netname: home
        remark: INFREIGHT Samba
        path:   C:\home\
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
netname: IPC$
        path:   C:\tmp
```

Reveals the actual filesystem paths behind each share name.

---

### netsharegetinfo — Detailed Info About a Specific Share

```
rpcclient $> netsharegetinfo notes
```

**Example output:**
```
netname: notes
        path:   C:\mnt\notes\
        max_uses:       -1
        num_uses:       1
DACL
        ACE
                type: ACCESS ALLOWED (0)
                Permissions: Generic all access
                SID: S-1-1-0
```

SID `S-1-1-0` = "Everyone" group → ALL users including anonymous have full access.

---

### enumdomusers — List All Domain Users

```
rpcclient $> enumdomusers
```

**Example output:**
```
user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]
```

Reveals usernames and their RIDs (Relative Identifiers).

---

### queryuser — Get Detailed User Info by RID

```
rpcclient $> queryuser 0x3e9
```

**Example output:**
```
User Name   :   cry0l1t3
Home Drive  :   \\devsmb\cry0l1t3
Password last set Time   :   Mi, 22 Sep 2021 17:50:56 CEST
Password must change Time:   Do, 14 Sep 30828 04:48:05 CEST
user_rid :      0x3e9
group_rid:      0x201
bad_password_count:     0x00000000
logon_count:    0x00000000
```

---

### querygroup — Get Group Information

```
rpcclient $> querygroup 0x201
```

**Example output:**
```
Group Name:     None
Description:    Ordinary Users
Num Members:2
```

---

## Brute Forcing User RIDs

### Why Brute Force RIDs?

Windows assigns RIDs to users sequentially starting from 500 (built-in) and 1000+ (regular). By querying each RID in sequence, you can discover ALL users even if `enumdomusers` is restricted.

```bash
for i in $(seq 500 1100); do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done
```

**Purpose:** Loops through numbers 500-1100, converts each to hexadecimal, sends a queryuser RPC command for each. Valid users print their information.

**Example output:**
```
        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201

        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201
```

---

## Impacket samrdump.py — Automated User Enumeration

```bash
samrdump.py 10.129.14.128
```

**Purpose:** Automatically connects to the SAM database over SMB and enumerates all users with detailed information.

**Example output:**
```
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
[*] Received 2 entries.
```

---

## SMBmap — Check Share Permissions

```bash
smbmap -H 10.129.14.128
```

**Purpose:** Quickly shows all available shares and exact permissions (READ, WRITE, or NO ACCESS).

**Example output:**
```
        Disk                          Permissions     Comment
        print$                        NO ACCESS       Printer Drivers
        home                          NO ACCESS       INFREIGHT Samba
        notes                         READ,WRITE      CheckIT
        IPC$                          NO ACCESS       IPC Service
```

`notes` has READ,WRITE for anonymous users — critical finding.

---

## CrackMapExec — Comprehensive SMB Enumeration

```bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

**Purpose:** Connects with empty credentials and lists all shares with permissions. Also shows Windows version, hostname, SMB signing status, and SMBv1 status.

**Example output:**
```
SMB  10.129.14.128  445  DEVSMB  [*] Windows 6.1 (name:DEVSMB) (signing:False) (SMBv1:False)
SMB  10.129.14.128  445  DEVSMB  Share       Permissions  Remark
SMB  10.129.14.128  445  DEVSMB  notes       READ,WRITE   CheckIT
```

`signing:False` → SMB signing not required → relay attacks possible.

---

## Enum4Linux-ng — Full Automated Enumeration

### Installation

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
```

### Run Full Enumeration

```bash
./enum4linux-ng.py 10.129.14.128 -A
```

**Purpose:** The `-A` flag runs ALL modules: LDAP check, SMB dialect detection, NetBIOS info, OS info, users, groups, shares, policies, and printers.

**Key output sections:**
```
[+] SMB Dialect: SMB 3.0 (Preferred)
[+] SMB signing required: false
[+] Server allows session using username '', password ''

Users:
'1000': username: mrb3n
'1001': username: cry0l1t3

Shares:
[+] home — Mapping: OK, Listing: OK
[+] notes — Mapping: OK, Listing: OK

Password Policy:
min_pw_length: 5
DOMAIN_PASSWORD_COMPLEX: false
```

Minimum 5-character passwords with no complexity = very weak, easy to brute force.

---

# 7. NFS (Network File System)

## What is NFS?

NFS (Network File System) was developed by Sun Microsystems and serves the same fundamental purpose as SMB — allowing files on a remote server to be accessed as if they were on the local machine. The key difference is that NFS is designed specifically for Linux and Unix systems, while SMB was designed for Windows. NFS clients cannot directly communicate with SMB servers. NFS uses ONC-RPC (Open Network Computing Remote Procedure Call) on TCP and UDP port 111 for service registration and port 2049 for actual file operations. Authentication in NFS v2 and v3 is based on UNIX UID/GID numbers — the server trusts whatever UID the client claims, which creates security risks. NFSv4 adds Kerberos authentication and uses only port 2049.

---

## NFS Version Comparison

| Version | Features |
|---------|---------|
| NFSv2 | Oldest, UDP-based, limited features |
| NFSv3 | Variable file size, better error reporting, not backward compatible with v2 |
| NFSv4 | Kerberos auth, firewall-friendly (port 2049 only), ACLs, stateful protocol, high security |
| NFSv4.1 | Parallel NFS (pNFS) for distributed access, session trunking (multipathing) |

---

## NFS Configuration — /etc/exports

### Viewing the NFS Exports File

```bash
cat /etc/exports
```

**Purpose:** Shows which directories are shared and who can access them with what permissions. Format: `directory host/network(options)`.

---

### Adding an NFS Share

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs
```

**Command 1:** Adds a new export entry sharing `/mnt/nfs` to the entire `10.129.14.0/24` subnet.
**Command 2:** Restarts NFS server to apply changes.
**Command 3:** Displays currently active exports to confirm.

**Output of exportfs:**
```
/mnt/nfs        10.129.14.0/24
```

---

## NFS Export Options

| Option | Description |
|--------|-------------|
| `rw` | Read and write permissions |
| `ro` | Read only permissions |
| `sync` | Write to disk before confirming (safer, slightly slower) |
| `async` | Confirm before writing to disk (faster, risk of data loss) |
| `secure` | Only allow connections from ports below 1024 |
| `insecure` | Allow connections from any port including above 1024 |
| `no_subtree_check` | Disable subdirectory checking (improves reliability) |
| `root_squash` | Map root (UID 0) to anonymous — prevents remote root from having local root |
| `no_root_squash` | Root on client = root on server — DANGEROUS |
| `nohide` | Make nested mounted filesystems visible |

---

## Dangerous NFS Settings

| Setting Combination | Risk |
|--------------------|----|
| `rw + no_root_squash` | Remote root has full server root access — critical |
| `insecure` | Any process can connect, not just privileged ones |
| `nohide` | Accidentally exposes nested filesystems |

---

## Footprinting NFS with Nmap

### Basic NFS Port Scan

```bash
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

**Purpose:** Scans ports 111 (RPC portmapper) and 2049 (NFS). The `rpcinfo` script lists all running RPC services and their ports.

**Example output:**
```
PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   100003  3,4  2049/tcp   nfs
|   100005  1,2,3  45837/tcp  mountd
|   100021  1,3,4  44629/tcp  nlockmgr
|   100227  3    2049/tcp   nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
```

---

### NFS-Specific NSE Scripts

```bash
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

**Purpose:** Runs all NFS-related scripts: `nfs-ls` (list files), `nfs-showmount` (show exports), `nfs-statfs` (disk stats).

**Example output:**
```
| nfs-ls: Volume /mnt/nfs
| PERMISSION  UID  GID  SIZE  TIME                 FILENAME
| rwxrwxrwx   65534 65534 4096 2021-09-19T15:28:17  .
| rw-r--r--   0     0    1872 2021-09-19T15:27:42  id_rsa
| rw-r--r--   0     0     348 2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0     0       0 2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount:
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs:
|_  /mnt/nfs  30313412.0  8074868.0  20675664.0  29%  16.0T  32000
```

SSH private key files visible without even mounting — critical finding.

---

## Performing NFS Tasks

### Step 1 — Show Available NFS Shares

```bash
showmount -e 10.129.14.128
```

**Purpose:** Queries the remote NFS server for its export list.

**Output:**
```
Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

---

### Step 2 — Create Local Mount Point

```bash
mkdir target-NFS
```

**Purpose:** Creates an empty local directory to serve as the mount point.

---

### Step 3 — Mount the NFS Share

```bash
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
```

**Flag breakdown:**
- `sudo` — Root privileges needed
- `mount` — Mount command
- `-t nfs` — Filesystem type is NFS
- `10.129.14.128:/` — Remote server and path
- `./target-NFS/` — Local mount point
- `-o nolock` — Disable NFS file locking

---

### Step 4 — Navigate and View Files

```bash
cd target-NFS
tree .
```

**Output:**
```
.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share
```

---

### Step 5 — List Files with Usernames

```bash
ls -l mnt/nfs/
```

**Output:**
```
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
```

---

### Step 6 — List Files with UIDs and GIDs (Numeric)

```bash
ls -n mnt/nfs/
```

**Output:**
```
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

**Purpose:** `-n` shows numeric IDs. Create a local user with UID 1000 to gain access to cry0l1t3's private files.

---

### Step 7 — Unmount the NFS Share

```bash
cd ..
sudo umount ./target-NFS
```

**Purpose:** First navigate OUT of the mounted directory, then unmount. Always clean up when done.

---

# 8. DNS (Domain Name System)

## What is DNS?

DNS (Domain Name System) is the internet's distributed address book. It translates human-readable domain names like `hackthebox.com` into numeric IP addresses that computers use. DNS has no central database — information is spread across thousands of name servers worldwide organized in a hierarchical tree. DNS is mostly unencrypted, making queries visible to network monitors and ISPs. Modern alternatives like DNS over TLS (DoT) and DNS over HTTPS (DoH) encrypt DNS queries.

---

## DNS Server Types

| Server Type | Description |
|------------|-------------|
| DNS Root Server | 13 worldwide, managed by ICANN, handle top-level domain queries |
| Authoritative Nameserver | Holds official records for a specific zone — answers are definitive |
| Non-authoritative Nameserver | Collects DNS info via recursive queries, acts as middleman |
| Caching DNS Server | Stores recent DNS answers for TTL period to speed up repeat queries |
| Forwarding Server | Passes queries to another specified DNS server |
| Resolver | DNS client built into the OS, sends queries to configured DNS server |

---

## DNS Record Types

| Record | Description |
|--------|-------------|
| A | Maps domain to IPv4 address |
| AAAA | Maps domain to IPv6 address |
| MX | Mail Exchange — specifies mail servers for the domain |
| NS | Name Server — which DNS servers are authoritative |
| TXT | Text records — email security, domain verification |
| CNAME | Canonical Name / Alias — one domain points to another |
| PTR | Reverse lookup — IP address to domain name |
| SOA | Start of Authority — admin info, serial number, zone settings |

---

## DNS Configuration Files — BIND9

### Local Configuration File

```bash
cat /etc/bind/named.conf.local
```

**Example content:**
```
zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};
```

Defines zones this server is authoritative for, where their files are, and who can update them.

---

### Zone File — Forward Lookup

```bash
cat /etc/bind/db.domain.com
```

**Example content with explanations:**
```
$ORIGIN domain.com
$TTL 86400
@  IN  SOA  dns1.domain.com.  hostmaster.domain.com. (
               2001062501 ; serial — increment on every change
               21600      ; refresh — slaves check every 6 hours
               3600       ; retry — retry after 1 hour if refresh fails
               604800     ; expire — discard records after 1 week
               86400 )    ; minimum TTL — 1 day

   IN  NS   ns1.domain.com.         ; primary nameserver
   IN  NS   ns2.domain.com.         ; secondary nameserver
   IN  MX   10  mx.domain.com.      ; primary mail server
   IN  A    10.129.14.5             ; domain itself resolves here

server1  IN  A      10.129.14.5     ; server1 subdomain
ftp      IN  CNAME  server1         ; ftp is alias to server1
www      IN  CNAME  server2         ; www is alias to server2
```

---

### Reverse Zone File

```bash
cat /etc/bind/db.10.129.14
```

**Example content:**
```
$ORIGIN 14.129.10.in-addr.arpa
$TTL 86400
@  IN  SOA  dns1.domain.com.  hostmaster.domain.com. ( ... )
   IN  NS   ns1.domain.com.

5  IN  PTR  server1.domain.com.
7  IN  MX   mx.domain.com.
```

`5 IN PTR server1.domain.com.` = IP 10.129.14.**5** resolves to server1.domain.com.

---

## Dangerous DNS Settings

| Option | Risk |
|--------|------|
| `allow-query { any; }` | Any host can query — enables DNS amplification attacks |
| `allow-recursion { any; }` | Recursive queries from external hosts — DDoS amplifier |
| `allow-transfer { any; }` | Anyone can download the complete zone file — most dangerous |
| `zone-statistics` | Reveals query patterns if accessible |

---

## Footprinting DNS — Complete Commands

### DIG — NS Query

```bash
dig ns inlanefreight.htb @10.129.14.128
```

**Purpose:** Queries the specified DNS server for NS records. Reveals which name servers are authoritative for the domain.

**Example output:**
```
;; ANSWER SECTION:
inlanefreight.htb.  604800  IN  NS  ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.  604800  IN  A  10.129.34.136
```

---

### DIG — Version Query

```bash
dig CH TXT version.bind 10.129.120.85
```

**Purpose:** Sends a CHAOS class TXT query to retrieve the DNS server's version string (if configured to respond).

**Example output:**
```
;; ANSWER SECTION:
version.bind.  0  CH  TXT  "9.10.6-P1"
version.bind.  0  CH  TXT  "9.10.6-P1-Debian"
```

BIND version 9.10.6-P1 on Debian — searchable for known CVEs.

---

### DIG — ANY Query

```bash
dig any inlanefreight.htb @10.129.14.128
```

**Purpose:** Requests ALL available record types in one query.

**Example output:**
```
inlanefreight.htb. 604800 IN TXT "v=spf1 include:mailgun.org..."
inlanefreight.htb. 604800 IN TXT "atlassian-domain-verification=..."
inlanefreight.htb. 604800 IN SOA inlanefreight.htb. root.inlanefreight.htb. ...
inlanefreight.htb. 604800 IN NS  ns.inlanefreight.htb.
ns.inlanefreight.htb.  604800  IN  A  10.129.34.136
```

---

### DIG — AXFR Zone Transfer (External Zone)

```bash
dig axfr inlanefreight.htb @10.129.14.128
```

**Purpose:** Attempts a full zone transfer. If the server is misconfigured, this downloads the COMPLETE zone file — every single DNS record.

**Example output:**
```
inlanefreight.htb.       604800  IN  SOA   inlanefreight.htb. root...
inlanefreight.htb.       604800  IN  NS    ns.inlanefreight.htb.
app.inlanefreight.htb.   604800  IN  A     10.129.18.15
internal.inlanefreight.htb. 604800 IN A   10.129.1.6
mail1.inlanefreight.htb. 604800  IN  A     10.129.18.201
ns.inlanefreight.htb.    604800  IN  A     10.129.34.136
;; XFR size: 9 records
```

---

### DIG — AXFR Zone Transfer (Internal Zone)

```bash
dig axfr internal.inlanefreight.htb @10.129.14.128
```

**Purpose:** Zone transfer for the INTERNAL zone — reveals even more sensitive internal infrastructure.

**Example output:**
```
dc1.internal.inlanefreight.htb.  604800 IN A  10.129.34.16
dc2.internal.inlanefreight.htb.  604800 IN A  10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A 10.129.18.200
vpn.internal.inlanefreight.htb.  604800 IN A  10.129.1.6
ws1.internal.inlanefreight.htb.  604800 IN A  10.129.1.34
ws2.internal.inlanefreight.htb.  604800 IN A  10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A  10.129.18.2
;; XFR size: 15 records
```

Complete internal network map: 2 domain controllers, VPN server, 2 workstations, WSUS update server.

---

### Subdomain Brute Forcing with Bash Loop

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

**Purpose:** Loops through 110,000 common subdomain names, queries each one, saves those that resolve to `subdomains.txt`.

**Example output:**
```
ns.inlanefreight.htb.    604800  IN  A  10.129.34.136
mail1.inlanefreight.htb. 604800  IN  A  10.129.18.201
app.inlanefreight.htb.   604800  IN  A  10.129.18.15
```

---

### DNSenum — Automated DNS Enumeration

```bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

**Flag breakdown:**
- `--dnsserver 10.129.14.128` — Use this specific DNS server
- `--enum` — Enable full enumeration mode
- `-p 0` — No Google scraping
- `-s 0` — No Google subdomain scraping
- `-o subdomains.txt` — Save results to file
- `-f ...` — Wordlist for brute forcing
- `inlanefreight.htb` — Target domain

**DNSenum automates:** NS queries, zone transfer attempts, subdomain brute forcing — all in one command.

---

# 9. SMTP (Simple Mail Transfer Protocol)

## What is SMTP?

SMTP is the standard protocol for sending email across IP networks. It handles email transmission between email clients and outgoing mail servers, and between mail servers themselves. By default, SMTP listens on port 25, but modern implementations also use port 587 for authenticated users with STARTTLS encryption. The email delivery chain: MUA (email client) → MSA (submission agent) → MTA (transfer agent — looks up DNS MX records) → MDA (delivery agent) → recipient mailbox. SMTP has two key weaknesses: no guaranteed delivery confirmation, and no built-in sender authentication — making email spoofing possible. ESMTP adds TLS encryption via STARTTLS, and SPF/DKIM/DMARC address spoofing.

---

## Default SMTP Configuration

```bash
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"
```

**Example key settings:**
```
myhostname = mail1.inlanefreight.htb
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
smtpd_helo_restrictions = reject_invalid_hostname
```

`myhostname` reveals the server hostname. `mynetworks` shows which IPs can relay — if set too broadly creates an open relay.

---

## SMTP Commands

| Command | Description | Example |
|---------|-------------|---------|
| `AUTH PLAIN` | Authenticate client | `AUTH PLAIN` |
| `HELO` | Basic session start, client introduces itself | `HELO mail1.example.com` |
| `EHLO` | Extended Hello — server responds with all ESMTP capabilities | `EHLO mail1` |
| `MAIL FROM:` | Specify sender address | `MAIL FROM: <sender@example.com>` |
| `RCPT TO:` | Specify recipient address | `RCPT TO: <recipient@example.com>` |
| `DATA` | Begin email body input, end with `.` on own line | `DATA` |
| `RSET` | Reset current transaction without closing connection | `RSET` |
| `VRFY` | Verify if username/mailbox exists | `VRFY username` |
| `EXPN` | Expand mailing list to show members | `EXPN mailinglist` |
| `NOOP` | Keep connection alive, do nothing | `NOOP` |
| `QUIT` | Terminate session | `QUIT` |

---

## Performing SMTP Tasks

### Connecting with Telnet

```bash
telnet 10.129.14.128 25
```

**Purpose:** Opens raw TCP connection to SMTP server on port 25. Lets you manually type SMTP commands.

---

### HELO/EHLO Interaction

```
telnet 10.129.14.128 25

Connected to 10.129.14.128.
220 ESMTP Server

HELO mail1.inlanefreight.htb
250 mail1.inlanefreight.htb

EHLO mail1
250-mail1.inlanefreight.htb
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250-DSN
250-SMTPUTF8
250 CHUNKING
```

EHLO reveals all server capabilities. `VRFY` being listed means user enumeration may be possible.

---

### User Enumeration with VRFY

```
VRFY root
252 2.0.0 root

VRFY cry0l1t3
252 2.0.0 cry0l1t3

VRFY nonexistentuser
252 2.0.0 nonexistentuser
```

**Purpose:** VRFY asks if a username exists. Some servers return 250 for valid and 550 for invalid — enabling reliable enumeration. Others return 252 for everything (like here) which is less useful.

---

### Sending an Email via Telnet

```
telnet 10.129.14.128 25

EHLO inlanefreight.htb
250 CHUNKING

MAIL FROM: <cry0l1t3@inlanefreight.htb>
250 2.1.0 Ok

RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
250 2.1.5 Ok

DATA
354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: Test Message
Date: Tue, 28 Sept 2021 16:32:51 +0200

Message body here.
.
250 2.0.0 Ok: queued as 6E1CF1681AB

QUIT
221 2.0.0 Bye
```

The `NOTIFY=success,failure` on RCPT TO requests delivery notifications. A single `.` on its own line ends the email body.

---

## Dangerous SMTP Setting — Open Relay

```
mynetworks = 0.0.0.0/0
```

**What this means:** Accept email relay from ALL IP addresses. Attackers can send spam, phishing, or spoofed emails through this server. Gets the domain blacklisted.

---

## Footprinting SMTP with Nmap

### Basic SMTP Scan

```bash
sudo nmap 10.129.14.128 -sC -sV -p25
```

**Example output:**
```
PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY,
ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING,
```

Banner reveals Postfix and hostname. VRFY capability confirms user enumeration possible.

---

### Check for Open Relay

```bash
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v
```

**Purpose:** Runs 16 different relay tests. If any pass, the server can be abused for email spoofing.

**Example output (open relay found):**
```
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@ESMTP> -> RCPT TO:<relaytest@nmap.scanme.org>
...
```

All 16 tests passed — critical finding.

---

# 10. IMAP / POP3

## What are IMAP and POP3?

IMAP (Internet Message Access Protocol) and POP3 (Post Office Protocol 3) are both protocols for RECEIVING emails. IMAP is the modern standard — keeps emails on the server, supports folders, works across multiple devices simultaneously, and syncs changes everywhere. Uses port 143 (plain) and 993 (SSL/TLS). POP3 is simpler — download and optionally delete from server, no folder structure, one device. Uses port 110 (plain) and 995 (SSL/TLS). Both protocols can be exploited if misconfigured, particularly if they expose verbose debug information, allow anonymous access, or use weak authentication.

---

## IMAP Commands

| Command | Description |
|---------|-------------|
| `1 LOGIN username password` | Authenticate user (tag "1" matches response) |
| `1 LIST "" *` | List all mailbox folders |
| `1 CREATE "INBOX"` | Create new mailbox folder |
| `1 DELETE "INBOX"` | Delete a mailbox folder permanently |
| `1 RENAME "ToRead" "Important"` | Rename existing folder |
| `1 LSUB "" *` | List subscribed folders only |
| `1 SELECT INBOX` | Open mailbox for message access |
| `1 UNSELECT INBOX` | Exit selected mailbox |
| `1 FETCH <ID> all` | Retrieve all data for a specific message |
| `1 CLOSE` | Delete flagged messages and deselect mailbox |
| `1 LOGOUT` | Close IMAP connection |

---

## POP3 Commands

| Command | Description |
|---------|-------------|
| `USER username` | Identify which user is logging in |
| `PASS password` | Provide authentication password |
| `STAT` | Get count and total size of emails |
| `LIST` | List all messages with individual sizes |
| `RETR id` | Download specific email by ID number |
| `DELE id` | Mark specific email for deletion |
| `CAPA` | Show server capabilities and extensions |
| `RSET` | Unmark all messages marked for deletion |
| `QUIT` | End session; permanently delete marked messages |

---

## Dangerous Settings

| Setting | Risk |
|---------|------|
| `auth_debug` | All authentication details written to logs |
| `auth_debug_passwords` | Submitted passwords written to logs in plain text |
| `auth_verbose` | Failed auth attempts logged with reasons (reveals valid usernames) |
| `auth_verbose_passwords` | Passwords logged in verbose mode |
| `auth_anonymous_username` | Allows anonymous login with no credentials |

---

## Footprinting IMAP/POP3 with Nmap

```bash
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC
```

**Purpose:** Scans all four mail ports. Default scripts reveal SSL certificates containing organizational information.

**Example output:**
```
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE SASL STLS TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight
|   stateOrProvinceName=California/countryName=US

143/tcp open  imap     Dovecot imapd
|_imap-capabilities: STARTTLS LOGIN-REFERRALS LITERAL+ AUTH=PLAIN

993/tcp open  ssl/imap Dovecot imapd
995/tcp open  ssl/pop3 Dovecot pop3d
```

SSL certificate reveals hostname, organization name, and location.

---

## Connecting to IMAP with cURL

### Basic IMAP Connection

```bash
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd
```

**Purpose:** Connects to IMAP over SSL. `-k` ignores SSL certificate errors. Lists available mailbox folders on success.

**Output:**
```
* LIST (\HasNoChildren) "." Important
* LIST (\HasNoChildren) "." INBOX
```

---

### Verbose IMAP Connection

```bash
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v
```

**Purpose:** `-v` shows complete TLS handshake, SSL certificate details, TLS version, cipher suite, and all IMAP authentication commands.

**Key output:**
```
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* Server certificate:
*  subject: C=US; ST=California; L=Sacramento; O=Inlanefreight;
   CN=mail1.inlanefreight.htb; emailAddress=cry0l1t3@inlanefreight.htb
< * OK [CAPABILITY IMAP4rev1 AUTH=PLAIN] HTB-Academy IMAP4 v.0.21.4
> A002 AUTHENTICATE PLAIN AGNyeTBsMXQzADEyMzQ=
< A002 OK Logged in
> A003 LIST "" *
< * LIST (\HasNoChildren) "." Important
< * LIST (\HasNoChildren) "." INBOX
```

Certificate reveals admin email. IMAP banner shows custom version string `HTB-Academy IMAP4 v.0.21.4`.

---

## OpenSSL — Connecting to POP3 over SSL

```bash
openssl s_client -connect 10.129.14.128:pop3s
```

**Purpose:** Connects to POP3 over SSL (port 995). Shows complete certificate chain and SSL session details. After handshake, gives interactive POP3 session.

**End of output:**
```
+OK HTB-Academy POP3 Server
```

---

## OpenSSL — Connecting to IMAP over SSL

```bash
openssl s_client -connect 10.129.14.128:imaps
```

**Purpose:** Connects to IMAP over SSL (port 993). Same as POP3 connection but for IMAP. After TLS handshake, you can type IMAP commands manually.

**End of output:**
```
* OK [CAPABILITY IMAP4rev1 SASL-IR AUTH=PLAIN] HTB-Academy IMAP4 v.0.21.4
```

---

# 11. SNMP (Simple Network Management Protocol)

## What is SNMP?

SNMP (Simple Network Management Protocol) was created specifically for monitoring and managing network devices remotely. It works with routers, switches, servers, printers, IoT devices, and virtually any networked hardware. SNMP transmits control commands over UDP port 161 and receives unsolicited notifications (traps) on UDP port 162. Traps are proactive alerts — when something specific happens (like a port going down), the device automatically sends a trap without being asked. SNMP uses a data model called MIB (Management Information Base) where each piece of information is addressed by an OID (Object Identifier). Without SNMP, network administrators would need to log into each device individually to check status — SNMP makes centralized monitoring possible.

---

## MIB and OID

**MIB (Management Information Base):** A structured text database that describes all queryable objects on a device. Like a dictionary — defines what each OID means, its data type, access rights, and description. Does NOT contain actual data — defines the structure for retrieving it.

**OID (Object Identifier):** A unique address in a hierarchical tree that identifies a specific piece of information. Looks like: `1.3.6.1.2.1.1.5.0`. Longer OID = more specific information.

---

## SNMP Versions

| Version | Security | Notes |
|---------|----------|-------|
| SNMPv1 | None | No authentication, no encryption, plain text |
| SNMPv2c | Community strings only | Still plain text — widely deployed despite flaws |
| SNMPv3 | Username/password + encryption | Secure but complex, uses MD5/SHA + DES/AES |

---

## Community Strings

Community strings in SNMPv1 and v2c function as simple passwords:
- **"public"** — Default read community string (read device info)
- **"private"** — Default write community string (modify device config)

Because these defaults are well-known and many admins never change them, they are the first things to try. Even when changed, community strings are sent in plain text and can be intercepted.

---

## Default SNMP Configuration

```bash
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'
```

**Example output:**
```
sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
```

`sysLocation` and `sysContact` are visible to all SNMP clients — reveal physical location and admin contact. `rocommunity public default` means "public" community string is accessible from all hosts but limited to the `systemonly` view.

---

## Dangerous SNMP Settings

| Setting | Risk |
|---------|------|
| `rwuser noauth` | Full OID tree read-write access with NO authentication |
| `rwcommunity <string> <IP>` | Full read-write access from specified IP (if broad, critical) |
| `rwcommunity6 <string> <IPv6>` | Same for IPv6 |

---

## SNMPwalk — Query All OIDs

```bash
snmpwalk -v2c -c public 10.129.14.128
```

**Flag breakdown:**
- `snmpwalk` — SNMP enumeration tool
- `-v2c` — Use SNMP version 2c
- `-c public` — Use "public" as community string

**Example output (key parts):**
```
iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu..."
iso.3.6.1.2.1.1.4.0 = STRING: "mrb3n@inlanefreight.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "htb"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/boot/vmlinuz-5.11.0-34-generic root=UUID=..."
iso.3.6.1.2.1.25.6.3.1.2.1243 = STRING: "python3_3.8.2-0ubuntu2_amd64"
```

Reveals: OS and kernel version, admin email, hostname, physical location, boot parameters, all installed packages.

---

## OneSixtyOne — Brute Force Community Strings

### Installation

```bash
sudo apt install onesixtyone
```

### Running the Attack

```bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128
```

**Purpose:** Tries hundreds of different community string values from a wordlist against the target SNMP service.

- `-c` — Specifies the wordlist of community strings to try

**Example output:**
```
Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic #41~20.04.2-Ubuntu...
```

Found "public" as valid. System description confirms access.

---

## Braa — Fast OID Brute Force

### Installation

```bash
sudo apt install braa
```

### Running Braa

```bash
braa public@10.129.14.128:.1.3.6.*
```

**Syntax:** `braa <community string>@<IP>:<OID pattern>`

**Purpose:** Once you have a valid community string, braa rapidly queries large ranges of OIDs simultaneously. Much faster than snmpwalk for broad enumeration.

**Example output:**
```
10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic...
10.129.14.128:20ms:.1.3.6.1.2.1.1.4.0:mrb3n@inlanefreight.htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.5.0:htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.6.0:US
```

20ms response time shows speed advantage over snmpwalk.

---

# 12. MySQL

## What is MySQL?

MySQL is an open-source relational database management system (RDBMS) owned by Oracle. It organizes data into tables with rows and columns, uses SQL (Structured Query Language) for all operations, and works on a client-server model. MySQL is the most popular database in web development and forms the "M" in the LAMP stack (Linux, Apache, MySQL, PHP). The MySQL server manages data storage and distribution while clients send SQL queries. MySQL typically runs on TCP port 3306. Data commonly stored includes website content, user credentials (hashed), customer information, product catalogs, session data, and application configuration. MariaDB is a fully compatible open-source fork of MySQL created by original MySQL developers after Oracle acquisition.

---

## Default MySQL Configuration

### Installation

```bash
sudo apt install mysql-server -y
```

### View Configuration

```bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'
```

**Key settings:**
```
port        = 3306
user        = mysql
datadir     = /var/lib/mysql
socket      = /var/run/mysqld/mysqld.sock
```

Port 3306 is standard. `user = mysql` means service runs as non-root system user (secure). `datadir` is where database files are stored.

---

## Dangerous MySQL Settings

| Setting | Risk |
|---------|------|
| `user = root` | MySQL runs as OS root — any MySQL exploit = OS root |
| `password` | Plain text password in config file |
| `admin_address = 0.0.0.0` | Admin interface exposed on all network interfaces |
| `debug` | Debug logs may contain sensitive SQL queries with passwords |
| `sql_warnings` | Detailed warnings expose schema to SQL injection attackers |
| `secure_file_priv` (empty) | MySQL can read any file — /etc/passwd, SSH keys, etc. |

---

## Footprinting MySQL with Nmap

```bash
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*
```

**Purpose:** Scans MySQL port 3306 with all MySQL-related NSE scripts.

**Example output:**
```
PORT     STATE SERVICE  VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-empty-password:
|_  root account has empty password
| mysql-enum:
|   Valid usernames:
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
| mysql-info:
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Auth Plugin Name: caching_sha2_password
```

Root has empty password — critical. Multiple accounts with empty passwords found.

---

## MySQL Connection and Commands

### Connecting Without Password

```bash
mysql -u root -h 10.129.14.132
```

**If refused:** `ERROR 1045 (28000): Access denied for user 'root'@'...' (using password: NO)`

---

### Connecting With Password

```bash
mysql -u root -pP4SSw0rd -h 10.129.14.128
```

> **Note:** No space between `-p` and the password.

**Successful connection:**
```
Welcome to the MariaDB monitor.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)
MySQL [(none)]>
```

---

### show databases — List All Databases

```sql
MySQL [(none)]> show databases;
```

**Output:**
```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```

`mysql` database contains user accounts and privileges. `information_schema` contains metadata.

---

### use — Select a Database

```sql
MySQL [(none)]> use mysql;
```

---

### show tables — List Tables

```sql
MySQL [mysql]> show tables;
```

The `user` table in the mysql database contains all accounts, privileges, and password hashes.

---

### select version() — Check MySQL Version

```sql
MySQL [(none)]> select version();
```

**Output:**
```
+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
```

---

### show columns — View Table Structure

```sql
show columns from user;
```

---

### select * — Extract All Data from Table

```sql
select * from user;
```

---

### Search for Specific Data

```sql
select * from customers where email = "otto@example.com";
```

---

### Querying the sys Database

```sql
mysql> use sys;
mysql> select host, unique_users from host_summary;
```

**Output:**
```
+-------------+--------------+
| host        | unique_users |
+-------------+--------------+
| 10.129.14.1 |            1 |
| localhost   |            2 |
+-------------+--------------+
```

Shows which hosts connected and how many distinct accounts they used.

---

## MySQL Command Summary

| Command | Description |
|---------|-------------|
| `mysql -u <user> -p<password> -h <IP>` | Connect to MySQL server |
| `show databases;` | List all databases |
| `use <database>;` | Select a database |
| `show tables;` | List all tables in selected database |
| `show columns from <table>;` | Show table structure |
| `select * from <table>;` | Get all data from table |
| `select * from <table> where <col> = "<value>";` | Search with filter |

---

# 13. MSSQL (Microsoft SQL Server)

## What is MSSQL?

MSSQL (Microsoft SQL Server) is Microsoft's proprietary closed-source relational database management system. Originally built specifically for Windows, it integrates deeply with the Windows ecosystem — Active Directory, Windows Authentication, .NET framework, and IIS. It is extremely popular in enterprise environments running Windows Server infrastructure. MSSQL communicates on TCP port 1433. A key feature is Windows Authentication — users can log into the database using their Windows/Active Directory domain credentials. This integration means compromising an Active Directory account can lead directly to database access, and database access can lead to domain compromise.

---

## MSSQL Client Tools

| Tool | Description |
|------|-------------|
| SQL Server Management Studio (SSMS) | Official Microsoft GUI tool, often on admin workstations |
| Impacket's mssqlclient.py | Best for penetration testers, Python CLI from Linux |
| mssql-cli | Microsoft's cross-platform CLI client |
| SQL Server PowerShell | Windows PowerShell MSSQL module |
| HeidiSQL / SQLPro | GUI clients |

### Finding mssqlclient.py

```bash
locate mssqlclient
```

**Output:**
```
/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py
```

---

## Default MSSQL System Databases

| Database | Description |
|---------|-------------|
| `master` | Tracks all system information for the SQL Server instance |
| `model` | Template for every new database — changes here affect all future databases |
| `msdb` | SQL Server Agent uses this for scheduled jobs, alerts, maintenance |
| `tempdb` | Stores temporary tables and intermediate results — recreated on restart |
| `resource` | Read-only hidden database with system objects |

---

## Dangerous MSSQL Configurations

| Issue | Risk |
|-------|------|
| No encryption between client/server | Credentials and data in plain text |
| Self-signed certificates | Can be spoofed for MITM attacks |
| Named pipes enabled | Alternative connection method, can bypass firewall rules |
| Weak/default `sa` credentials | SA = System Administrator — database superuser |
| xp_cmdshell enabled | Allows OS command execution from SQL — critical |

---

## Footprinting MSSQL with Nmap

```bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248
```

**Script purposes:**

| Script | Purpose |
|--------|---------|
| `ms-sql-info` | Server name, instance, version, named pipe path |
| `ms-sql-empty-password` | Check if sa or other accounts have empty passwords |
| `ms-sql-xp-cmdshell` | Test if OS command execution is enabled |
| `ms-sql-ntlm-info` | Get NTLM authentication information |
| `ms-sql-dump-hashes` | Attempt to dump password hashes |

**Example output:**
```
PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   Product_Version: 10.0.17763
| ms-sql-info:
|   Windows server name: SQL-01
|   Version: Microsoft SQL Server 2019 RTM
|   TCP port: 1433
|   Named pipe: \\10.129.201.248\pipe\sql\query
|_  Clustered: false
```

---

## Metasploit — MSSQL Ping

```
msf6 > use auxiliary/scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
msf6 auxiliary(scanner/mssql/mssql_ping) > run
```

**Example output:**
```
[+] 10.129.201.248:  -    ServerName      = SQL-01
[+] 10.129.201.248:  -    InstanceName    = MSSQLSERVER
[+] 10.129.201.248:  -    IsClustered     = No
[+] 10.129.201.248:  -    Version         = 15.0.2000.5
[+] 10.129.201.248:  -    tcp             = 1433
[+] 10.129.201.248:  -    np              = \\SQL-01\pipe\sql\query
```

---

## Connecting with mssqlclient.py

```bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth
```

**Purpose:** Connects using Windows Domain Authentication. `-windows-auth` uses NTLM/Kerberos instead of SQL Server authentication.

**Connection output:**
```
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation
Password: [enter password]
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208)
SQL>
```

---

### List All Databases

```sql
SQL> select name from sys.databases
```

**Output:**
```
master
tempdb
model
msdb
Transactions
```

"Transactions" is a custom database — investigate it.

---

# 14. Oracle TNS

## What is Oracle TNS?

Oracle TNS (Transparent Network Substrate) is Oracle's proprietary communication protocol for connecting Oracle database clients and applications to Oracle database servers. It supports multiple networking protocols and provides built-in encryption for data in transit, making it suitable for enterprise environments. TNS is widely used in healthcare, finance, and retail industries managing large Oracle databases. The TNS listener is the server-side component accepting incoming connection requests and routing them to correct database instances. It runs on TCP port 1521 by default.

---

## Oracle TNS Configuration Files

**tnsnames.ora** — Client-side. Maps service names to network addresses (IP + port + database SID). When an app specifies "ORCL", this file tells it where to find that service.

**listener.ora** — Server-side. Defines the listener's properties — ports it listens on, which database instances it services, security settings. Located in `$ORACLE_HOME/network/admin/`.

### Example tnsnames.ora

```
ORCL =
  (DESCRIPTION =
    (ADDRESS_LIST =
      (ADDRESS = (PROTOCOL = TCP)(HOST = 10.129.11.102)(PORT = 1521))
    )
    (CONNECT_DATA =
      (SERVER = DEDICATED)
      (SERVICE_NAME = orcl)
    )
  )
```

### Example listener.ora

```
SID_LIST_LISTENER =
  (SID_LIST =
    (SID_DESC =
      (SID_NAME = PDB1)
      (ORACLE_HOME = C:\oracle\product\19.0.0\dbhome_1)
      (GLOBAL_DBNAME = PDB1)
    )
  )

LISTENER =
  (DESCRIPTION_LIST =
    (DESCRIPTION =
      (ADDRESS = (PROTOCOL = TCP)(HOST = orcl.inlanefreight.htb)(PORT = 1521))
      (ADDRESS = (PROTOCOL = IPC)(KEY = EXTPROC1521))
    )
  )
```

---

## Oracle SID

The SID (System Identifier) uniquely identifies a specific Oracle database instance. A server can run multiple database instances, each with its own SID. Clients must specify the correct SID to reach the right database. Common default SIDs: `ORCL`, `XE` (Express Edition), `PROD`.

---

## Setting Up ODAT

```bash
sudo apt-get install -y build-essential python3-dev libaio1
git clone https://github.com/quentinhardy/odat.git
cd odat/
pip install python-libnmap
git submodule init
git submodule update
sudo apt-get install python3-scapy -y
sudo pip3 install colorlog termcolor passlib python-libnmap
sudo apt-get install build-essential libgmp-dev -y
pip3 install pycryptodome openpyxl
```

**Verify installation:**
```bash
./odat.py -h
```

---

## Footprinting Oracle TNS

### Basic TNS Scan

```bash
sudo nmap -p1521 -sV 10.129.204.235 --open
```

**Output:**
```
PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
```

---

### Nmap SID Brute Force

```bash
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute
```

**Output:**
```
| oracle-sid-brute:
|_  XE
```

Found SID "XE" (Oracle Express Edition).

---

## ODAT — Full Enumeration

```bash
./odat.py all -s 10.129.204.235
```

**Purpose:** Runs ALL ODAT modules — credential discovery, SID enumeration, user enumeration, vulnerability checks.

**Key output:**
```
[+] Valid credentials found: scott/tiger. Continue...
```

Famous default Oracle credentials `scott/tiger` still in use.

---

## Installing SQLplus

```bash
sudo apt update
sudo apt install oracle-instantclient-sqlplus
```

**Verify:**
```bash
sqlplus -v
```

**Output:**
```
SQL*Plus: Release 19.0.0.0.0 - Production
Version 19.6.0.0.0
```

**Fix library error if needed:**
```bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig
```

---

## Connecting to Oracle with SQLplus

```bash
sqlplus scott/tiger@10.129.204.235/XE
```

**Successful connection:**
```
Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production
SQL>
```

---

## Oracle SQL Commands

### List All Tables

```sql
SQL> select table_name from all_tables;
```

---

### Check User Privileges

```sql
SQL> select * from user_role_privs;
```

**Output:**
```
USERNAME     GRANTED_ROLE    ADM
SCOTT        CONNECT         NO
SCOTT        RESOURCE        NO
```

No DBA privileges.

---

### Connect as SYSDBA (Database Admin)

```bash
sqlplus scott/tiger@10.129.204.235/XE as sysdba
```

**Verify elevated privileges:**
```sql
SQL> select * from user_role_privs;
```

**Output:**
```
USERNAME  GRANTED_ROLE         ADM
SYS       DBA                  YES
SYS       EXECUTE_CATALOG_ROLE YES
```

Full DBA access achieved.

---

### Extract Password Hashes

```sql
SQL> select name, password from sys.user$;
```

**Output:**
```
NAME      PASSWORD
SYS       FBA343E7D6C8BC9D
SYSTEM    B5073FE1DE351687
OUTLN     4A3BA55E08595C81
```

Old Oracle DES hashes — crackable offline.

---

## Upload a File via ODAT

### Step 1 — Create Test File

```bash
echo "Oracle File Upload Test" > testing.txt
```

### Step 2 — Upload via ODAT

```bash
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt
```

**Flag breakdown:**
- `utlfile` — Uses Oracle's UTL_FILE package for file operations
- `-s 10.129.204.235` — Target server
- `-d XE` — Target SID
- `-U scott -P tiger` — Credentials
- `--sysdba` — Connect as SYSDBA for elevated access
- `--putFile C:\\inetpub\\wwwroot` — Destination on Windows web server
- `testing.txt` — Destination filename
- `./testing.txt` — Local source file

**Output:**
```
[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory
```

### Step 3 — Verify Upload

```bash
curl -X GET http://10.129.204.235/testing.txt
```

**Output:**
```
Oracle File Upload Test
```

File accessible via HTTP — next step would be uploading a web shell for RCE.

---

# 15. IPMI (Intelligent Platform Management Interface)

## What is IPMI?

IPMI (Intelligent Platform Management Interface) is a hardware-level remote management standard that lets administrators monitor and control servers completely independently of the operating system, BIOS, CPU, and firmware. Even if a server is completely powered off, has a crashed OS, or is unresponsive, IPMI can still power it on/off, check temperature/fan/voltage sensors, view system event logs, access a remote console, and reinstall the OS. IPMI works through a BMC (Baseboard Management Controller) — a small ARM processor directly connected to the server motherboard with its own power supply and network connection. IPMI communicates over UDP port 623. Common BMC implementations: HP iLO, Dell iDRAC, Supermicro IPMI.

---

## IPMI Components

| Component | Description |
|-----------|-------------|
| BMC (Baseboard Management Controller) | Core component — micro-controller on server motherboard |
| ICMB (Intelligent Chassis Management Bus) | Allows communication between multiple server chassis |
| IPMB (Intelligent Platform Management Bus) | Extends BMC's reach to other hardware components |
| IPMI Memory | Stores System Event Log, sensor data repository |
| Communications Interfaces | Local, serial, LAN (port 623 UDP), and ICMB interfaces |

---

## Default BMC Credentials

| Product | Username | Password |
|---------|----------|----------|
| Dell iDRAC | root | calvin |
| HP iLO | Administrator | Random 8-char (numbers + uppercase, printed on server label) |
| Supermicro IPMI | ADMIN | ADMIN |

Always try default credentials first during penetration tests.

---

## Footprinting IPMI with Nmap

```bash
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local
```

**Flag breakdown:**
- `sudo` — Required for UDP scanning
- `-sU` — UDP scan (IPMI uses UDP not TCP)
- `--script ipmi-version` — IPMI version detection script
- `-p 623` — Scan only port 623

**Example output:**
```
PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)
```

MAC prefix identifies as HP server. IPMI 2.0 confirmed.

---

## Metasploit — IPMI Version Discovery

```
msf6 > use auxiliary/scanner/ipmi/ipmi_version
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > run
```

**Output:**
```
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user)
PassAuth(password, md5, md2, null) Level(1.5, 2.0)
```

Supports both 1.5 and 2.0. `null` authentication support is a finding worth investigating.

---

## RAKP Protocol Vulnerability

The RAKP (Remote Authenticated Key-Exchange Protocol) vulnerability is a critical flaw in IPMI 2.0. During authentication, the server sends a salted SHA1 or MD5 hash of the user's password TO THE CLIENT before authentication is complete. An attacker can capture this hash for ANY valid user without completing the login. The hash can then be cracked offline. **There is no patch** — this is part of the IPMI 2.0 specification itself.

---

## Metasploit — Dumping IPMI Hashes

```
msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run
```

**Module options:**
- `CRACK_COMMON true` — Automatically attempts to crack with common passwords
- `OUTPUT_HASHCAT_FILE` — Save hashes for Hashcat offline cracking
- `OUTPUT_JOHN_FILE` — Save hashes for John the Ripper
- `PASS_FILE` — Wordlist for automatic cracking
- `USER_FILE` — Usernames to try (ADMIN, admin, Administrator, root)

**Example output:**
```
[+] 10.129.42.195:623 - IPMI - Hash found:
ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260...

[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
```

Hash captured AND cracked — ADMIN/ADMIN default credentials confirmed.

### Cracking Manually with Hashcat (for HP iLO factory defaults)

```bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
```

Mode 7300 = IPMI2 RAKP HMAC-SHA1. Tries all 8-character combinations of digits and uppercase letters.

---

# 16. Linux Remote Management Protocols

## SSH (Secure Shell)

### What is SSH?

SSH (Secure Shell) provides encrypted remote command-line access to Linux and Unix systems over TCP port 22. It replaced completely insecure older protocols like Telnet and R-Services by encrypting all data — including passwords, commands, and output — making network interception useless. SSH-1 is the original but vulnerable to MITM attacks. SSH-2 is the modern standard with stronger encryption. OpenSSH is pre-installed on virtually every Linux distribution and macOS. SSH also supports tunneling, port forwarding, and X11 forwarding for graphical applications.

---

### SSH Authentication Methods

| Method | Description |
|--------|-------------|
| Password authentication | User types password — vulnerable to brute force |
| Public-key authentication | Key pair: private key (local) + public key (server) — most secure |
| Host-based authentication | Authentication based on client machine identity |
| Keyboard authentication | Interactive challenge-response |
| Challenge-response authentication | Used with external auth systems |
| GSSAPI authentication | Kerberos and other GSSAPI mechanisms — common in enterprise AD |

### How Public Key Authentication Works

1. Generate key pair: `ssh-keygen -t rsa -b 4096`
2. Public key added to server's `~/.ssh/authorized_keys`
3. Private key stays local, protected by passphrase
4. Server encrypts a challenge using your public key
5. SSH client decrypts it with the private key and sends back the solution
6. Server verifies — if correct, access granted
7. **Your passphrase never travels over the network**

---

### Default SSH Configuration

```bash
cat /etc/ssh/sshd_config | grep -v "#" | sed -r '/^\s*$/d'
```

**Example output:**
```
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
Subsystem       sftp    /usr/lib/openssh/sftp-server
```

---

### Dangerous SSH Settings

| Setting | Risk |
|---------|------|
| `PasswordAuthentication yes` | Enables brute force attacks |
| `PermitEmptyPasswords yes` | Login with no password — critical |
| `PermitRootLogin yes` | Root can log in directly — very dangerous |
| `Protocol 1` | Old vulnerable SSH-1 protocol, MITM attacks |
| `X11Forwarding yes` | GUI forwarding — had command injection vulnerability (CVE-2016-3115) |
| `AllowTcpForwarding yes` | Attackers can tunnel traffic to internal resources |
| `PermitTunnel` | VPN-like tunneling through SSH — bypass firewall |
| `DebianBanner yes` | Reveals OS information in login banner |

---

### Footprinting SSH with ssh-audit

```bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132
```

**Purpose:** Analyzes SSH server configuration and cryptographic settings without authentication. Checks SSH version, key exchange algorithms, host key algorithms, encryption ciphers, and flags weak ones.

**Example output:**
```
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1

(kex) curve25519-sha256           -- [info] available since OpenSSH 7.4
(kex) ecdh-sha2-nistp256          -- [fail] using weak elliptic curves
(kex) ecdh-sha2-nistp384          -- [fail] using weak elliptic curves

(key) ssh-rsa (3072-bit)          -- [fail] using weak hashing algorithm
(key) ecdsa-sha2-nistp256         -- [fail] using weak elliptic curves
(key) ssh-ed25519                 -- [info] available since OpenSSH 6.5
```

Banner reveals exact OpenSSH version. Several algorithms flagged as weak.

---

### Verbose SSH Connection — Check Auth Methods

```bash
ssh -v cry0l1t3@10.129.14.132
```

**Key output line:**
```
debug1: Authentications that can continue: publickey,password,keyboard-interactive
```

Shows all available authentication methods. Password auth listed = brute force possible.

---

### Force Specific Authentication Method

```bash
ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password
```

**Purpose:** Forces password authentication only, skipping public key. Useful for targeted brute force testing.

**Output:**
```
debug1: Next authentication method: password
cry0l1t3@10.129.14.132's password:
```

---

## Rsync

### What is Rsync?

Rsync is a file synchronization tool that uses a delta-transfer algorithm — only transfers the parts of files that have actually changed. This makes it extremely efficient for backups and keeping remote directories synchronized. Rsync uses port 873 by default but can be tunneled over SSH for encryption. During penetration tests, misconfigured Rsync servers may allow unauthenticated access to directories containing sensitive files.

---

### Scanning for Rsync

```bash
sudo nmap -sV -p 873 127.0.0.1
```

**Output:**
```
PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)
```

---

### Probing for Available Shares

```bash
nc -nv 127.0.0.1 873
```

**Interaction:**
```
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev             Dev Tools
@RSYNCD: EXIT
```

Found "dev" share with description "Dev Tools".

---

### Enumerate Files in Rsync Share

```bash
rsync -av --list-only rsync://127.0.0.1/dev
```

**Purpose:** Lists all files without downloading.

- `-a` — Archive mode (preserve permissions, timestamps)
- `-v` — Verbose
- `--list-only` — Only list, do not transfer

**Output:**
```
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh
```

`secrets.yaml` and `.ssh` directory — immediate targets.

---

### Download All Files from Rsync Share

```bash
rsync -av rsync://127.0.0.1/dev ./local-copy/
```

---

### Rsync Over SSH

```bash
rsync -av -e ssh rsync://127.0.0.1/dev ./local-copy/
# Non-standard SSH port:
rsync -av -e "ssh -p2222" rsync://127.0.0.1/dev ./local-copy/
```

**Purpose:** `-e ssh` uses SSH as transport layer for encryption.

---

## R-Services (Legacy Insecure Remote Access)

### What are R-Services?

R-Services are old Unix remote access protocols developed at UC Berkeley in the 1980s, predating SSH. They transmit ALL data including passwords in plain text and rely on trusting certain hosts rather than strong cryptographic authentication. Almost completely replaced by SSH but occasionally still found in legacy enterprise environments (Solaris, HP-UX, AIX). R-services span ports 512 (rexec), 513 (rlogin), 514 (rsh/rcp).

---

### R-Commands Overview

| Command | Port | Description |
|---------|------|-------------|
| rcp | 514 TCP | Copy files bidirectionally — no overwrite warnings |
| rsh | 514 TCP | Remote shell without login procedure |
| rexec | 512 TCP | Run commands on remote machine — sends password in plain text |
| rlogin | 513 TCP | Remote login, Unix-only, similar to Telnet |

---

### Trust Files

`/etc/hosts.equiv` — System-wide trust. Lists trusted hostnames and users. Format: `<hostname> <username>`.

`~/.rhosts` — Per-user trust. Same format but only affects that specific user.

**Viewing /etc/hosts.equiv:**
```bash
cat /etc/hosts.equiv
# pwnbox cry0l1t3
```

**Example .rhosts (dangerous):**
```
htb-student     10.0.17.5
+               10.0.17.10
+               +
```

`+ +` = trust ANYONE from ANYWHERE — completely open.

---

### Scanning for R-Services

```bash
sudo nmap -sV -p 512,513,514 10.0.17.2
```

**Output:**
```
PORT    STATE SERVICE    VERSION
512/tcp open  exec?
513/tcp open  login?
514/tcp open  tcpwrapped
```

---

### Logging in with Rlogin

```bash
rlogin 10.0.17.2 -l htb-student
```

**Successful login (no password required):**
```
Last login: Fri Dec  2 16:11:21 from localhost
[htb-student@localhost ~]$
```

Access granted through misconfigured `.rhosts` trust file.

---

### Rwho — List Active Sessions on Network

```bash
rwho
```

**Output:**
```
root     web01:pts/0 Dec  2 21:34
htb-student     workstn01:tty1  Dec  2 19:57  2:25
```

Shows all logged-in users across the network — usernames, which machine, terminal type, login time, and idle time.

---

### Rusers — Detailed Network User Information

```bash
rusers -al 10.0.17.5
```

**Purpose:** More detailed than rwho. Shows username, hostname, TTY, login time, idle time, and remote host.

- `-a` — Show all users including idle
- `-l` — Long format (detailed output)

**Output:**
```
htb-student     10.0.17.5:console          Dec 2 19:57     2:25
```

---

# 17. Windows Remote Management Protocols

## RDP (Remote Desktop Protocol)

### What is RDP?

RDP (Remote Desktop Protocol) is Microsoft's proprietary protocol for providing full graphical remote access to Windows systems. Unlike SSH which is command-line only, RDP gives complete control of the Windows desktop — seeing the screen and controlling mouse and keyboard — just as if sitting in front of it. RDP uses TCP port 3389 and optionally UDP 3389 for improved performance. All data including passwords is encrypted using TLS/SSL since Windows Vista. Network Level Authentication (NLA) is an additional security layer requiring authentication BEFORE the full RDP session is established — preventing unauthorized access to the Windows login screen.

---

### Footprinting RDP with Nmap

```bash
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*
```

**Purpose:** Scans RDP port 3389 with all RDP-related NSE scripts: `rdp-enum-encryption`, `rdp-ntlm-info`.

**Example output:**
```
PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-enum-encryption:
|   Security layer
|     CredSSP (NLA): SUCCESS
|     CredSSP with Early User Auth: SUCCESS
|_    RDSTLS: SUCCESS
| rdp-ntlm-info:
|   Target_Name: ILF-SQL-01
|   NetBIOS_Computer_Name: ILF-SQL-01
|   DNS_Computer_Name: ILF-SQL-01
|   Product_Version: 10.0.17763
|_  System_Time: 2021-11-06T13:46:00+00:00
```

Hostname: ILF-SQL-01, Windows Server 2019 (10.0.17763), NLA required.

---

### RDP Packet Trace

```bash
nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n
```

**Purpose:** Shows all raw network packets during the scan.

**Security note — from the output:**
```
NSE: TCP 10.10.14.20:36630 > 10.129.201.248:3389 |
Cookie: mstshash=nmap
```

Nmap uses `mstshash=nmap` as an RDP cookie. EDR systems and security tools specifically look for this to identify Nmap scans. Can trigger alerts and get you blocked on hardened networks.

---

### RDP Security Check with rdp-sec-check.pl

### Installation

```bash
sudo cpan
cpan[1]> install Encoding::BER
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
```

### Running the Check

```bash
./rdp-sec-check.pl 10.129.201.248
```

**Purpose:** Checks RDP security configurations without authenticating. Tests which security layers and encryption methods the server supports.

**Example output:**
```
[+] Checking supported protocols
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Not supported
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Not supported
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Supported

[+] Summary of protocol support
[-] 10.129.201.248:3389 supports PROTOCOL_SSL   : FALSE
[-] 10.129.201.248:3389 supports PROTOCOL_HYBRID: TRUE
[-] 10.129.201.248:3389 supports PROTOCOL_RDP   : FALSE
```

Only NLA (CredSSP) is supported — well-hardened configuration.

---

### Connecting via RDP from Linux

```bash
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248
```

**Flag breakdown:**
- `xfreerdp` — FreeRDP client for Linux (X11 version)
- `/u:cry0l1t3` — Username
- `/p:"P455w0rd!"` — Password (quotes handle special characters)
- `/v:10.129.201.248` — Target server IP

**During connection:**
```
[WARN] Certificate verification failure 'self signed certificate'
[ERROR] WARNING: CERTIFICATE NAME MISMATCH!
Common Name (CN): ILF-SQL-01

Do you trust the above certificate? (Y/T/N) Y
```

Type `Y` to accept. Full Windows desktop window opens after authentication.

---

## WinRM (Windows Remote Management)

### What is WinRM?

WinRM (Windows Remote Management) is Microsoft's command-line remote management protocol built on the WS-Management (Web Services Management) standard. It uses SOAP (Simple Object Access Protocol) over HTTP/HTTPS for communication. WinRM is Microsoft's answer to SSH — providing command-line remote access to Windows systems. It uses TCP port 5985 (HTTP) and TCP port 5986 (HTTPS). WinRM is enabled by default on Windows Server 2012 and later. PowerShell remote sessions also use WinRM as their transport. WinRS (Windows Remote Shell) allows executing arbitrary commands on remote systems through WinRM.

---

### Footprinting WinRM with Nmap

```bash
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
```

**Example output:**
```
PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Port 5985 open (HTTP WinRM). The "Not Found" response is expected — WinRM uses specific SOAP endpoints, not regular web pages.

---

### Connecting with Evil-WinRM

```bash
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!
```

**Flag breakdown:**
- `-i 10.129.201.248` — Target IP
- `-u Cry0l1t3` — Username
- `-p P455w0rD!` — Password

**Successful connection:**
```
Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Cry0l1t3\Documents>
```

Full PowerShell prompt on the remote Windows system.

---

## WMI (Windows Management Instrumentation)

### What is WMI?

WMI (Windows Management Instrumentation) is Microsoft's implementation of WBEM (Web-Based Enterprise Management) and CIM (Common Information Model). It provides read and write access to almost ALL settings and information on Windows systems — hardware, software, OS configuration, processes, network settings, user accounts, and installed applications. WMI is the most powerful management interface available on Windows. WMI communication starts on TCP port 135 (the DCE/RPC endpoint mapper), then moves to a randomly assigned port for actual data transfer. This makes WMI harder to block with simple firewall rules since the data port is dynamic.

---

### Footprinting WMI with wmiexec.py

```bash
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"
```

**Purpose:** Uses WMI to execute commands on remote Windows systems from Linux.

**Format:** `wmiexec.py username:password@target_IP "command_to_run"`

**Example output:**
```
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] SMBv3.0 dialect used
ILF-SQL-01
```

The `hostname` command executed on the remote system and returned "ILF-SQL-01" — confirming successful remote command execution via WMI.

**Useful follow-up commands to run:**
```bash
# Check who you are and privileges
"whoami /all"

# List all users
"net user"

# Get network information
"ipconfig /all"

# List running processes
"tasklist"
```

---

# Quick Reference — Ports Summary

| Service | Port(s) | Protocol | Notes |
|---------|---------|----------|-------|
| FTP | 20, 21 | TCP | 21=control, 20=data |
| TFTP | 69 | UDP | No auth, LAN only |
| SSH | 22 | TCP | Encrypted remote access |
| Telnet | 23 | TCP | Unencrypted, avoid |
| SMTP | 25, 587 | TCP | 25=server-to-server, 587=auth clients |
| DNS | 53 | TCP/UDP | UDP for queries, TCP for zone transfers |
| HTTP | 80 | TCP | Web traffic |
| POP3 | 110, 995 | TCP | 110=plain, 995=SSL |
| NFS | 111, 2049 | TCP/UDP | 111=portmap, 2049=NFS |
| IMAP | 143, 993 | TCP | 143=plain, 993=SSL |
| HTTPS | 443 | TCP | Encrypted web |
| SMB | 139, 445 | TCP | 139=NetBIOS, 445=Direct |
| MSSQL | 1433 | TCP | Microsoft SQL Server |
| Oracle TNS | 1521 | TCP | Oracle database listener |
| MySQL | 3306 | TCP | MySQL/MariaDB database |
| RDP | 3389 | TCP/UDP | Windows Remote Desktop |
| SNMP | 161, 162 | UDP | 161=queries, 162=traps |
| Rsync | 873 | TCP | File synchronization |
| IPMI | 623 | UDP | Hardware management |
| WinRM | 5985, 5986 | TCP | 5985=HTTP, 5986=HTTPS |

---

# Quick Reference — Key Tools

| Tool | Purpose | Example |
|------|---------|---------|
| `nmap` | Port scanning and service detection | `nmap -sV -sC -p21 10.x.x.x` |
| `ftp` | FTP client | `ftp 10.x.x.x` |
| `smbclient` | SMB client | `smbclient -N -L //10.x.x.x` |
| `rpcclient` | SMB/RPC enumeration | `rpcclient -U "" 10.x.x.x` |
| `crackmapexec` | SMB enumeration | `crackmapexec smb 10.x.x.x --shares -u '' -p ''` |
| `smbmap` | SMB permission check | `smbmap -H 10.x.x.x` |
| `enum4linux-ng` | Full SMB enumeration | `./enum4linux-ng.py 10.x.x.x -A` |
| `showmount` | NFS share listing | `showmount -e 10.x.x.x` |
| `dig` | DNS queries | `dig axfr domain.htb @10.x.x.x` |
| `dnsenum` | DNS enumeration | `dnsenum --dnsserver 10.x.x.x domain.htb` |
| `telnet` | Raw protocol interaction | `telnet 10.x.x.x 25` |
| `openssl` | SSL/TLS connections | `openssl s_client -connect 10.x.x.x:443` |
| `snmpwalk` | SNMP OID querying | `snmpwalk -v2c -c public 10.x.x.x` |
| `onesixtyone` | SNMP community string brute force | `onesixtyone -c wordlist.txt 10.x.x.x` |
| `mysql` | MySQL client | `mysql -u root -pPassword -h 10.x.x.x` |
| `mssqlclient.py` | MSSQL client | `python3 mssqlclient.py user@10.x.x.x -windows-auth` |
| `sqlplus` | Oracle client | `sqlplus user/pass@10.x.x.x/XE` |
| `odat.py` | Oracle attack tool | `./odat.py all -s 10.x.x.x` |
| `ssh-audit` | SSH security analysis | `./ssh-audit.py 10.x.x.x` |
| `evil-winrm` | WinRM shell from Linux | `evil-winrm -i 10.x.x.x -u user -p pass` |
| `xfreerdp` | RDP client for Linux | `xfreerdp /u:user /p:pass /v:10.x.x.x` |
| `wmiexec.py` | WMI remote execution | `wmiexec.py user:pass@10.x.x.x "whoami"` |
| `wget` | Download files | `wget -m --no-passive ftp://anon:anon@10.x.x.x` |
| `curl` | HTTP/IMAP/other requests | `curl -k 'imaps://10.x.x.x' --user user:pass` |

---

*End of Notes — All services covered completely with every command, its purpose, usage, and full demonstration.*
