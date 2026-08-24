# Complete Notes: Footprinting & Network Services Enumeration

---

## Section 1: Enumeration Methodology

### What is Enumeration Methodology?

Enumeration methodology is a standardized, step-by-step approach used during penetration testing to gather information about a target system in an organized way. Without a proper methodology, a penetration tester might miss important parts of the target or waste time going in the wrong direction. Because penetration testing involves many different types of systems, networks, and configurations, having a fixed but flexible structure helps the tester stay on track. The methodology is not a rigid checklist but rather a systematic framework that allows for changes depending on what the tester finds. It is divided into three levels: Infrastructure-based enumeration, Host-based enumeration, and OS-based enumeration. These three levels together cover everything from the internet presence of a company to the internal operating system configuration of its servers.

---

### The 6 Layers of Enumeration

The entire enumeration process is organized into 6 layers, each representing a boundary or "wall" that the tester tries to pass through to get closer to the target. Think of it like a labyrinth where each layer is a ring, and the tester needs to find the gap in each ring to move forward. Each layer has specific goals and information categories that the tester focuses on.

---

#### Layer 1 — Internet Presence

This is the outermost layer. Here the tester finds out what the company looks like from the internet. The goal is to find all possible targets such as domains, subdomains, IP addresses, cloud instances, and other infrastructure that is visible on the internet. This layer is especially important in black-box tests where no information is given upfront. The tester uses passive techniques — meaning they do not directly connect to the target — to gather information. Tools like crt.sh, Shodan, and DNS lookups are used at this stage. Information gathered includes domains, subdomains, virtual hosts (vHosts), ASN (Autonomous System Numbers), netblocks, IP addresses, cloud instances, and security measures in place.

---

#### Layer 2 — Gateway

Once the internet presence is identified, the next step is to understand how the company protects its infrastructure. This layer focuses on identifying the protective measures between the internet and the internal network. The goal is to understand what kind of security is in place, such as firewalls, DMZ (Demilitarized Zones), IPS/IDS systems, EDR tools, proxies, NAC (Network Access Control), VPNs, and services like Cloudflare. Understanding the gateway helps the tester know what kind of traffic is blocked and what might pass through without triggering alarms.

---

#### Layer 3 — Accessible Services

This is the layer that is mostly covered in the main footprinting module. Here the tester identifies all services running on the target systems that can be accessed from outside. Each service has a specific purpose — for example, FTP for file transfer, SSH for remote access, HTTP for web traffic. The tester looks at the service type, its version, its configuration, which port it is running on, and what functionality it provides. The goal is to understand how the service works and how it can be communicated with, which leads to finding ways to exploit it.

---

#### Layer 4 — Processes

Every time a command or function is executed on a server, some process runs behind the scenes. This layer focuses on identifying what internal processes are running, what data they handle, and what sources and destinations are involved. For example, a web server process might be processing requests from users and sending them to a database. Understanding these processes helps the tester identify dependencies and potential weak points. Key information includes PID (Process ID), the type of data being processed, tasks being performed, and source/destination details.

---

#### Layer 5 — Privileges

Every service on a system runs under a specific user account with specific permissions. Sometimes administrators give services more permissions than needed, or they forget to review permission settings. This layer is about identifying what permissions are attached to each service and each user, and whether those permissions can be misused. This is especially important in Active Directory environments where users might have permissions across many systems. The tester looks at groups, users, permissions, restrictions, and the overall environment.

---

#### Layer 6 — OS Setup

This is the innermost layer. At this point, the tester already has internal access to the system and is gathering information about the operating system itself. This includes the OS type (Linux, Windows), the patch level (is it up to date?), network configuration, environment variables, configuration files, and sensitive private files. The goal is to understand how the administrator has set up and maintains the system, and to find any sensitive internal information that can be used for further exploitation.

---

## Section 2: Domain Information

### What is Domain Information?

Domain information is one of the most important parts of the passive reconnaissance phase in penetration testing. It is not only about finding subdomains but about understanding the full internet presence of a company — what technologies they use, what services they run, and how their infrastructure is structured. This entire process is done passively, meaning the tester acts like a normal internet user or customer and does not send any direct scans or requests to the target that could alert the company. The information gathered here gives a big-picture view of the target before any active testing begins. Think of it as reading everything that is publicly available about the company without knocking on their door.

---

### Checking the Company Website

The first step is always to visit the company's official website and read the content carefully. The website tells you what kind of services the company offers, what technologies they use, and what their business model is. For example, if a company offers IoT solutions, app development, and cloud hosting, then you know they likely run servers, APIs, and possibly cloud storage. Reading the website with a developer's eye helps you guess what kind of backend infrastructure might be in place. Even small details like "built with Django" or "powered by AWS" in a footer or page source code reveal important technical information.

---

### SSL Certificates and crt.sh

#### What is crt.sh?

crt.sh is a website that stores Certificate Transparency logs. When any website gets an SSL certificate from a Certificate Authority like Let's Encrypt, a public log entry is created. This log can be searched to find all subdomains that a company has ever used a certificate for. This is extremely useful because companies often have many subdomains — some of which might be forgotten or misconfigured. By searching crt.sh for a domain, you can find subdomains that are not listed anywhere else publicly.

#### How to Use crt.sh via Command Line

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .
```

**Purpose:** This command fetches JSON data from crt.sh about all SSL certificates issued for the domain `inlanefreight.com`. The `jq .` part formats the JSON output to make it readable.

To filter only unique subdomains:

```bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u
```

This command filters out only the subdomain names from the JSON response, removes certificate subject names (CN=), and sorts the results to show only unique entries. The output gives you a clean list of all discovered subdomains.

---

### Finding Company-Hosted Servers

#### Purpose

After getting a list of subdomains, the next step is to find which ones point to IP addresses that belong to the company itself (not third-party providers). This is important because you can only test hosts that are within the scope of your engagement. Third-party hosted services (like AWS, Cloudflare) require separate permission.

#### Command

```bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4; done
```

This loop goes through each subdomain in your list, uses the `host` command to resolve its IP address, and then filters only those that belong to the target domain. The output shows the subdomain and its resolved IP address.

---

### Using Shodan for IP Intelligence

#### What is Shodan?

Shodan is a search engine for internet-connected devices. Unlike Google which indexes web content, Shodan indexes open ports, service banners, and device information. It can show what services are running on a given IP address, what software versions are in use, and what ports are open.

#### Command

```bash
for i in $(cat ip-addresses.txt); do shodan host $i; done
```

This loops through all discovered IP addresses and queries Shodan for information about each one. The output might show open ports like 22 (SSH), 80 (HTTP), 443 (HTTPS), along with the software running on them (like nginx, Apache), SSL certificate details, and even the city and organization the IP belongs to.

---

### DNS Records with dig

#### What is dig?

`dig` (Domain Information Groper) is a command-line tool used to query DNS servers and retrieve DNS records. DNS records are like a phonebook for the internet — they map domain names to IP addresses and other information.

#### Command

```bash
dig any inlanefreight.com
```

This retrieves all available DNS record types for the domain. The main record types and what they tell you:

- **A Records** — Map a domain name to an IPv4 address. Shows you which servers the domain points to.
- **MX Records** — Mail exchange records. Tell you which servers handle email for the domain.
- **NS Records** — Name server records. Show which DNS servers are responsible for the domain.
- **TXT Records** — Text records used for verification and security purposes. These can contain verification codes for services like Google, Atlassian, and LogMeIn. They also contain SPF (Sender Policy Framework) records which reveal what email services and IP addresses the company uses.
- **SOA Record** — Start of Authority. Contains administrative information about the domain including the primary DNS server and the contact email for the domain.

#### What TXT Records Reveal

From TXT records, a tester can identify third-party tools and services the company uses. For example:

| TXT Record Value | What It Reveals |
|---|---|
| `atlassian-domain-verification` | Company uses Atlassian (Jira, Confluence, Bitbucket) |
| `google-site-verification` | Company uses Google services; might have open Google Drive links |
| `logmein-verification-code` | Company uses LogMeIn for remote access management |
| `v=spf1 include:mailgun.org` | Company uses Mailgun for email APIs |
| `include:spf.protection.outlook.com` | Company uses Microsoft 365/Azure |

Each of these reveals attack surfaces — for example, Mailgun APIs might be vulnerable to IDOR or SSRF attacks, and Azure file storage can be accessed via SMB protocol.

---

## Section 3: Cloud Resources

### What are Cloud Resources in Penetration Testing?

Modern companies use cloud services from AWS, Google Cloud (GCP), and Microsoft Azure for storage, computing, and hosting. While cloud providers themselves are very secure, the configurations that companies apply can introduce vulnerabilities. The most common issue is misconfigured storage — such as AWS S3 buckets, Azure Blobs, or GCP Cloud Storage — that are set to public access without authentication. During a penetration test, discovering these misconfigured cloud resources can lead to accessing sensitive files like documents, source code, private keys, and even credentials.

---

### Finding Cloud Storage with Google Dorks

#### What are Google Dorks?

Google Dorks are advanced Google search operators that narrow down search results to very specific types of pages. In the context of cloud resources, they help find public files stored in AWS or Azure that are indexed by Google.

For AWS S3 Buckets:
```
intext:companyname inurl:amazonaws.com
```

For Azure Blobs:
```
intext:companyname inurl:blob.core.windows.net
```

These searches can return PDF files, documents, images, and other files that companies have accidentally made public in their cloud storage. Sometimes these files contain sensitive business data.

---

### Finding Cloud Resources in Website Source Code

Companies often load images, JavaScript files, and CSS from their cloud storage to reduce load on web servers. If you inspect the HTML source code of a company's website, you might find links to `blob.core.windows.net` or `s3.amazonaws.com` which reveal the cloud storage URLs. These can then be further investigated to see if they allow public access or directory listing.

---

### Third-Party Tools for Cloud Enumeration

#### Domain.Glass

Domain.Glass is an online tool that provides information about a domain's infrastructure. It can reveal cloud service providers, CDN usage (like Cloudflare), and basic security assessment information. If Cloudflare is detected, it is noted as a security measure in Layer 2 (Gateway) of the enumeration methodology.

#### GrayHatWarfare

GrayHatWarfare is a specialized search engine for public cloud storage. It indexes publicly accessible files in AWS S3, Azure Blob, and GCP Cloud Storage. A tester can search by company name, filter by file type (PDF, DOCX, XLSX, etc.), and discover what data has been accidentally made public. This is a completely passive approach and requires no interaction with the target's own systems.

---

### Risk: Leaked SSH Private Keys

One of the most dangerous things that can appear in misconfigured cloud storage is leaked SSH private keys. An SSH private key (typically a file named `id_rsa`) allows direct login to servers without needing a password. If an employee accidentally uploaded their private key to a public S3 bucket, an attacker can download it and use it to log into any server that has the corresponding public key installed. This is a very real risk that happens when employees are under pressure, make mistakes, or do not understand the consequences of sharing certain files.

---

## Section 4: Staff — OSINT on Employees

### Why Look at Employees?

Employees of a company are a goldmine of information for penetration testers doing OSINT (Open Source Intelligence). By looking at LinkedIn profiles, GitHub accounts, job postings, and professional websites, you can learn what technologies and tools the company uses, what the internal team structure looks like, what programming languages and frameworks developers work with, and even find leaked credentials or sensitive code in public repositories. This information helps you understand the attack surface and plan your approach.

---

### LinkedIn Job Posts

Job posts reveal a lot about internal technology stacks. For example, if a job requires experience with Django, Flask, PostgreSQL, and Kubernetes, you know the company runs Python web applications on a container infrastructure with a PostgreSQL database. This tells you what technologies to look for vulnerabilities in. A job requiring Atlassian Suite experience confirms the earlier TXT record finding that Atlassian tools are used internally.

---

### LinkedIn Employee Profiles

Individual employee profiles show their skills, past projects, and linked GitHub accounts. From a developer's GitHub profile, you might find open source code they wrote that contains company-specific logic, hardcoded API keys or JWT tokens, database connection strings, or even personal email addresses tied to company systems.

---

### Risk: Hardcoded Credentials in Public Code

A very common finding is that developers push code to GitHub that contains hardcoded secrets. For example, a JWT (JSON Web Token) with a hardcoded secret key found in a public GitHub repo could allow an attacker to forge authentication tokens for the company's web application. This is why code review and secrets scanning are so important in security-conscious organizations.

---

## Section 5: FTP (File Transfer Protocol)

### What is FTP?

FTP (File Transfer Protocol) is one of the oldest protocols on the internet. It was created long before modern security concerns and has been in use for decades for transferring files between computers. FTP operates at the application layer of the TCP/IP protocol stack, sitting at the same layer as HTTP and POP3. This means it works on top of the basic internet communication rules and provides a specific service — file transfer. There are special FTP programs and clients built specifically for FTP, in addition to built-in support in many operating systems. FTP is widely used by web developers, system administrators, and businesses to upload files to servers, share data, and move large files between systems over a network.

---

### How FTP Works — Two Channels

FTP is unique because it uses two separate TCP connections to function, not just one like most protocols.

**Channel 1 — Control Channel (TCP Port 21):**
This is where all the commands and responses happen. When you connect to an FTP server, this channel is opened first. The client sends commands like `LIST`, `GET`, `PUT`, and the server responds with status codes. This channel stays open for the entire FTP session.

**Channel 2 — Data Channel (TCP Port 20):**
This is where the actual file data flows. Every time you list a directory or transfer a file, a separate data connection is opened for that specific operation and then closed afterward. This separation of control and data is a key design feature of FTP.

If a connection is interrupted during a file transfer, FTP can resume the transfer after reconnecting — this is another important feature that makes FTP practical for large file transfers.

---

### Active Mode vs Passive Mode

#### Active Mode

In Active Mode, the client connects to the server on port 21 (control channel) and tells the server which port the client is using for data. The server then initiates the data connection back to the client on port 20. The problem here is that if the client is behind a firewall, the firewall may block incoming connections from the server, breaking the data transfer.

#### Passive Mode

In Passive Mode, after the client connects on port 21, the server announces an available port number. The client then initiates the data connection to that port on the server. Because the client is always the one starting connections, firewalls protecting the client do not block anything. This is why passive mode is the standard for internet FTP connections today.

---

### FTP Authentication

FTP normally requires a username and password to log in. However, many servers are configured to allow **anonymous FTP** where anyone can log in using "anonymous" as the username (and sometimes any email address as a password). Anonymous FTP is used to share public files but carries security risks because anyone can access those files.

> **Important security concern:** FTP sends all data including usernames and passwords in **plain text** (unencrypted). If someone is monitoring the network, they can see everything. This is why SFTP (SSH File Transfer Protocol) and FTPS (FTP over SSL) were developed as secure alternatives.

---

### TFTP (Trivial File Transfer Protocol)

#### What is TFTP?

TFTP is a much simpler version of FTP. It was designed for situations where simplicity is more important than features or security. While FTP uses TCP and provides reliable, ordered delivery, TFTP uses **UDP**, making it faster but unreliable. TFTP handles data loss through application-layer recovery, meaning it has its own simple mechanism for dealing with lost packets.

TFTP has **no authentication at all** — no usernames, no passwords. It relies entirely on the file system's read and write permissions to control access. Because of this total lack of security, TFTP should only be used on local, protected, trusted networks — never on the internet. It is commonly used for booting network devices, loading firmware, and distributing configuration files in controlled environments.

#### TFTP Commands

| Command | Description |
|---|---|
| `connect` | Sets the remote host and optionally the port number for file transfers. You must run this before any transfer commands. Example: `connect 10.129.14.136` |
| `get` | Transfers one or more files FROM the remote host TO your local machine. Example: `get config.txt` |
| `put` | Transfers one or more files FROM your local machine TO the remote host. Example: `put update.bin` |
| `quit` | Exits the TFTP program and closes the connection cleanly. |
| `status` | Shows the current status of the TFTP session, including the transfer mode (ASCII or binary), whether a connection is active, the timeout value, and other session details. |
| `verbose` | Turns verbose mode on or off. When verbose is on, extra information is displayed during file transfers including packet counts and timing — useful for debugging. |

> **Important note:** Unlike FTP, TFTP does NOT have a directory listing command. You cannot see what files exist on the server — you must already know the exact filename you want to download.

---

### vsFTPd — Default FTP Server on Linux

#### What is vsFTPd?

vsFTPd (Very Secure FTP Daemon) is the most commonly used FTP server software on Linux-based systems. The name "Very Secure" refers to its design philosophy of security-first. It is the default FTP server on many Linux distributions including Ubuntu and CentOS. Understanding vsFTPd is important because it is what you will encounter most often during penetration tests on Linux systems.

#### Installing vsFTPd

```bash
sudo apt install vsftpd
```

**Purpose:** Installs the vsFTPd FTP server package on your Linux system. The `sudo` is needed because installing software requires administrator privileges. After installation, the service starts automatically and the configuration file is created at `/etc/vsftpd.conf`.

#### Viewing the vsFTPd Configuration File

```bash
cat /etc/vsftpd.conf | grep -v "#"
```

**Purpose:** Displays the vsFTPd configuration file, filtering out all comment lines (lines starting with `#`) so you only see the active settings. The `grep -v "#"` part means "show everything EXCEPT lines containing `#`". This gives you a clean view of what settings are actually active.

Key configuration settings and what they mean:

| Setting | Description |
|---|---|
| `listen=NO` | Controls whether vsFTPd runs as a standalone service (YES) or is managed by the inetd/xinetd super-server (NO). Standalone is more common in modern setups. |
| `listen_ipv6=YES` | Enables IPv6 listening in addition to IPv4. This allows clients with IPv6 addresses to connect. |
| `anonymous_enable=NO` | Controls whether anonymous login is allowed. When set to NO, only authenticated users with valid credentials can log in. Setting this to YES is a dangerous configuration that allows anyone to connect. |
| `local_enable=YES` | Allows local Linux system users (users with accounts on the server) to log in to FTP with their system credentials. |
| `dirmessage_enable=YES` | When a user enters a directory that has a `.message` file, the contents of that file are displayed. |
| `use_localtime=YES` | Uses the server's local timezone when displaying file timestamps in directory listings instead of UTC. |
| `xferlog_enable=YES` | Enables logging of all file uploads and downloads. This creates an audit trail of all file transfer activity. Important for security monitoring. |
| `connect_from_port_20=YES` | In Active Mode, the server initiates data connections from port 20. This setting enforces that behavior. |
| `secure_chroot_dir=/var/run/vsftpd/empty` | Points to an empty directory used for security purposes when the server needs to operate in a chrooted (isolated) environment without filesystem access. |
| `pam_service_name=vsftpd` | Specifies the PAM (Pluggable Authentication Module) service configuration file that vsFTPd will use for authentication. PAM is the Linux authentication framework. |
| `rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem` | The path to the SSL certificate file used when SSL encryption is enabled. |
| `rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key` | The path to the private key that pairs with the SSL certificate. |
| `ssl_enable=NO` | Controls whether SSL/TLS encryption is used for FTP connections. When NO, all data including passwords travels in plain text. |

---

### The /etc/ftpusers File

#### What is /etc/ftpusers?

This file is a **deny list** — it contains usernames of system users who are specifically prohibited from using FTP, even if they have valid system accounts and correct passwords. This is a security feature to prevent certain powerful or sensitive system accounts from accessing the FTP service.

```bash
cat /etc/ftpusers
```

Example output:
```
guest
john
kevin
```

This means even if "guest", "john", and "kevin" exist on the Linux system and know their passwords, they cannot log in through FTP. This is typically used to block system accounts (like root, daemon, nobody) from FTP access as a security measure.

---

### Dangerous FTP Settings

These settings, when enabled, create security vulnerabilities that penetration testers specifically look for:

| Setting | Risk |
|---|---|
| `anonymous_enable=YES` | Allows anyone to log in without any credentials at all. Just use "anonymous" as the username. This is the most common dangerous setting and is often left enabled by accident. |
| `anon_upload_enable=YES` | Allows anonymous users to upload files. This is extremely dangerous because attackers can upload malicious files, web shells, or use the FTP space for illegal content. |
| `anon_mkdir_write_enable=YES` | Allows anonymous users to create new directories. Combined with upload, an attacker can organize uploaded malicious content in their own folder structure. |
| `no_anon_password=YES` | Anonymous users are not even asked to enter a password. Normally anonymous FTP asks for an email address as a courtesy password. This setting skips even that minimal step. |
| `anon_root=/home/username/ftp` | Specifies the root directory for anonymous users. Restricts anonymous users to a specific folder so they cannot browse the entire filesystem. |
| `write_enable=YES` | Enables the FTP write commands: STOR (upload), DELE (delete), RNFR/RNTO (rename), MKD (make directory), RMD (remove directory), APPE (append to file), and SITE (site-specific commands). Without this, users can only read and download. |
| `hide_ids=YES` | Replaces the actual numeric user IDs and group IDs in directory listings with the generic string "ftp". Prevents attackers from learning real usernames on the system through FTP directory listings. |
| `ls_recurse_enable=YES` | Allows the use of recursive directory listing (ls -R). Lets users see the entire directory tree at once with a single command. For a penetration tester, this is extremely useful because one command reveals the entire FTP file structure. |

---

### Connecting to FTP — Anonymous Login

#### How to Perform Anonymous Login

```bash
ftp 10.129.14.136
```

What happens step by step:

1. The FTP client attempts to connect to the server at the given IP on port 21.
2. The server responds with a `220` banner — this is the welcome message that often includes the FTP software name and version.
3. You are prompted for a username. Type `anonymous` and press Enter.
4. You may be prompted for a password. For anonymous login, type anything or just press Enter.
5. If anonymous login is allowed, you receive code `230` meaning "Login successful."
6. The server tells you the remote system type (usually UNIX) and transfer mode (binary or ASCII).

Full example interaction:

```
ftp 10.129.14.136

Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
```

> **Important note:** The banner (code 220) often reveals the FTP software name and version. This is very valuable information for a penetration tester because it tells you exactly what software is running, allowing you to search for known vulnerabilities in that specific version.

---

### FTP Commands Inside a Session

#### ls — List Directory Contents

```
ftp> ls
```

**Purpose:** Lists all files and directories in the current directory on the FTP server. Shows file permissions, owner IDs, file sizes, dates, and names. The status codes shown are `200` (PORT command success), `150` (about to send data), and `226` (transfer complete).

Example output:

```
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

The first column shows file permissions (`d` means directory, `-` means file, then `rwx` for read/write/execute for owner/group/others). Second and third columns show owner UID and GID. Fourth is file size. Then date and filename.

---

#### status — Check Current FTP Session Settings

```
ftp> status
```

**Purpose:** Displays the current configuration of your FTP session. This is not about the server — it is about how your FTP client is currently set up. It tells you what mode you are in, what your transfer settings are, and what optional features are enabled or disabled.

Example output:

```
Connected to 10.129.14.136.
No proxy connection.
Connecting using address family: any.
Mode: stream; Type: binary; Form: non-print; Structure: file
Verbose: on; Bell: off; Prompting: on; Globbing: on
Store unique: off; Receive unique: off
Case: off; CR stripping: on
Quote control characters: on
Ntrans: off
Nmap: off
Hash mark printing: off; Use of PORT cmds: on
Tick counter printing: off
```

---

#### debug — Enable Debug Mode

```
ftp> debug
```

**Purpose:** Turns on debugging in the FTP client. When debug mode is on, the client prints every command it sends to the server before sending it. This lets you see the raw FTP protocol communication happening behind the scenes. It is very useful for understanding exactly how FTP works and for troubleshooting connection issues.

After enabling:
```
Debugging on (debug=1).
```

Now every command you type will show the raw FTP protocol command being sent, like `---> PORT 10,10,14,4,188,195`.

---

#### trace — Enable Packet Tracing

```
ftp> trace
```

**Purpose:** Turns on packet-level tracing. This is even more detailed than debug mode. With trace enabled, the FTP client shows every network packet being sent and received. This is mainly used for deep protocol analysis and troubleshooting at the network level.

After enabling:
```
Packet tracing on.
```

#### Using debug and trace together — Detailed Output Example

```
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
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.
```

Now you can see the raw PORT command being sent (which tells the server where to send the data) and the LIST command that requests the directory listing. This is exactly how the FTP protocol works under the hood.

---

#### ls -R — Recursive Directory Listing

```
ftp> ls -R
```

**Purpose:** Lists all files and directories recursively — meaning it goes into every subfolder and lists everything inside them too, all in one command. This requires `ls_recurse_enable=YES` to be set on the server. For a penetration tester, this is a very powerful command because you can see the ENTIRE file structure of the FTP server with one command without having to manually navigate each directory.

Example output:

```
.:
-rw-rw-r--    1 ftp      ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 ftp      ftp         4096 Sep 14 16:50 Employees

./Clients:
drwx------    2 ftp      ftp          4096 Sep 16 18:04 HackTheBox
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:00 Inlanefreight

./Clients/HackTheBox:
-rw-r--r--    1 ftp      ftp         34872 Sep 16 18:04 appointments.xlsx
-rw-r--r--    1 ftp      ftp        498123 Sep 16 18:04 contract.docx
-rw-r--r--    1 ftp      ftp        478237 Sep 16 18:04 contract.pdf
-rw-r--r--    1 ftp      ftp           348 Sep 16 18:04 meetings.txt

./Clients/Inlanefreight:
-rw-r--r--    1 ftp      ftp         14211 Sep 16 18:00 appointments.xlsx
-rw-r--r--    1 ftp      ftp         37882 Sep 16 17:58 contract.docx
-rw-r--r--    1 ftp      ftp            89 Sep 16 17:58 meetings.txt
-rw-r--r--    1 ftp      ftp        483293 Sep 16 17:59 proposal.pptx

./Documents:
-rw-r--r--    1 ftp      ftp         23211 Sep 16 18:05 appointments-template.xlsx
-rw-r--r--    1 ftp      ftp         32521 Sep 16 18:05 contract-template.docx
-rw-r--r--    1 ftp      ftp        453312 Sep 16 18:05 contract-template.pdf

./Employees:
226 Directory send OK.
```

> Notice that when `hide_ids=YES` is set, all owner IDs show as "ftp" instead of real usernames. Also notice that the `./Employees` directory is empty — there are no files inside it.

---

#### get — Download a Single File

```
ftp> get "Important Notes.txt"
```

**Purpose:** Downloads a specific file from the FTP server to your local machine. The file is saved in whatever directory you were in on your local machine when you launched FTP. If the filename has spaces, put it in quotes. When the download completes, FTP shows how many bytes were received and the transfer speed.

Full example:

```
ftp> ls

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rwxrwxrwx    1 ftp      ftp             0 Sep 16 17:24 Calendar.pptx
drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents
drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees
-rwxrwxrwx    1 ftp      ftp            41 Sep 18 15:58 Important Notes.txt
226 Directory send OK.


ftp> get Important\ Notes.txt

local: Important Notes.txt remote: Important Notes.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for Important Notes.txt (41 bytes).
226 Transfer complete.
41 bytes received in 0.00 secs (606.6525 kB/s)


ftp> exit

221 Goodbye.
```

After exiting FTP, verify the file was downloaded:

```bash
ls | grep Notes.txt
'Important Notes.txt'
```

Note the backslash `\` before the space in the filename when not using quotes — this "escapes" the space so the shell doesn't treat it as two separate arguments.

---

### Downloading All Files at Once with wget

```bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136
```

**Purpose:** This downloads the entire FTP server content in one command using wget instead of the interactive FTP client. This is very useful when the FTP server has many files spread across many directories and you want everything at once.

Breaking down the command:

- `wget` — The download tool
- `-m` — Mirror mode, recursively downloads everything, preserving directory structure
- `--no-passive` — Forces Active Mode (uses PORT commands instead of PASV)
- `ftp://anonymous:anonymous@10.129.14.136` — The URL format: protocol://username:password@server

What happens: wget connects to the FTP server, logs in as anonymous, and recursively downloads every file it can access. A directory named after the server's IP address is created, and all files are stored inside it preserving the original folder structure.

Example output:

```
--2021-09-19 14:45:58--  ftp://anonymous:*password*@10.129.14.136/
           => '10.129.14.136/.listing'
Connecting to 10.129.14.136:21... connected.
Logging in as anonymous ... Logged in!
==> SYST ... done.    ==> PWD ... done.
==> TYPE I ... done.  ==> CWD not needed.
==> PORT ... done.    ==> LIST ... done.

FINISHED --2021-09-19 14:45:58--
Total wall clock time: 0,03s
Downloaded: 15 files, 1,7K in 0,001s (3,02 MB/s)
```

> **Security warning:** Downloading ALL files at once may trigger alerts on the target system because no legitimate user normally downloads everything simultaneously. Use this technique carefully during authorized penetration tests.

---

### Viewing Downloaded Structure with tree

```bash
tree .
```

**Purpose:** Displays all downloaded files in a visual tree format. This makes it easy to understand the directory structure at a glance.

Example output:

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
    │   ├── contract-template.docx
    │   └── contract-template.pdf
    ├── Employees
    └── Important Notes.txt

5 directories, 9 files
```

---

### put — Upload a File to FTP Server

#### Why Uploading Matters in Penetration Testing

If you can upload files to an FTP server that is connected to a web server, you can potentially upload a web shell — a script that lets you run system commands through the browser. This is a common path to gaining Remote Command Execution (RCE). Even FTP logs can be abused — if a web server logs FTP activity and you can inject commands into your FTP username or input, those commands might execute when the log file is processed.

#### Step 1 — Create a Test File

```bash
touch testupload.txt
```

**Purpose:** Creates an empty test file named `testupload.txt` on your local machine. The `touch` command creates a new empty file if it does not exist, or updates the timestamp if it does. This is used as a safe, harmless test to check whether the FTP server allows uploads.

#### Step 2 — Upload the File

```
ftp> put testupload.txt
```

**Purpose:** Uploads the `testupload.txt` file from your local machine to the current directory on the FTP server. The FTP client sends the `STOR` command to the server, the server opens a data channel, and the file bytes are transferred.

Full example:

```
ftp> put testupload.txt

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.
```

After the upload, verify it appears in the directory listing:

```
ftp> ls

---> LIST
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 17:03 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 1002     133             0 Sep 15 14:57 testupload.txt
226 Directory send OK.
```

The `testupload.txt` now appears in the listing with size 0 (empty file). This confirms upload is possible — a major finding in a penetration test.

---

### hide_ids Setting — Impact on Directory Listing

#### What hide_ids=YES Does

When this setting is active, the FTP server replaces all real user and group ownership information in directory listings with the generic string "ftp". Normally, directory listings show the actual UID and GID numbers of file owners. With `hide_ids` enabled, all files appear to be owned by "ftp".

**Why administrators use it:** To prevent attackers from learning real system usernames through FTP directory listings, because those usernames could then be used in brute force attacks against SSH, FTP, or other services.

Example listing WITH `hide_ids=YES`:

```
-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt
```

Notice everything shows "ftp ftp" as owner and group instead of real values like "1002 1002".

> **Counter-defense note:** Modern systems use fail2ban to detect and block brute force attempts. If too many failed login attempts come from one IP, that IP gets automatically blocked. So even if you discover real usernames through FTP, brute forcing those accounts from the same IP would quickly get blocked.

---

### Footprinting FTP with Nmap

#### What is Nmap NSE?

Nmap includes the Nmap Scripting Engine (NSE) — a collection of scripts written in Lua that can perform specific enumeration, detection, and even exploitation tasks against target services. For FTP, there are several NSE scripts that automatically check for common vulnerabilities and misconfigurations.

#### Updating the NSE Script Database

```bash
sudo nmap --script-updatedb
```

**Purpose:** Updates the local NSE script database so Nmap knows about all available scripts. You should run this periodically to get the latest scripts.

Example output:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
```

#### Finding FTP-Related NSE Scripts

```bash
find / -type f -name ftp* 2>/dev/null | grep scripts
```

**Purpose:** Searches the entire filesystem for files whose names start with "ftp" and filters results to show only those in a "scripts" directory. The `2>/dev/null` redirects error messages (like "permission denied") to null so they don't clutter the output.

Example output:

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

What each script does:

| Script | Purpose |
|---|---|
| `ftp-anon.nse` | Checks if the FTP server allows anonymous login. If it does, the script logs in anonymously and lists the root FTP directory contents. |
| `ftp-syst.nse` | Sends the STAT command to the FTP server to get detailed status information. Reveals the FTP server type, version, session settings, connection count, and timeout values. |
| `ftp-brute.nse` | Attempts to brute force FTP credentials using a username and password list. Used when you know the service requires authentication. |
| `ftp-vsftpd-backdoor.nse` | Checks for the famous backdoor planted in vsFTPd version 2.3.4 in 2011. If this backdoor exists, connecting to port 6200 gives a root shell. |
| `ftp-proftpd-backdoor.nse` | Similar to above but for ProFTPd server backdoors. |
| `ftp-bounce.nse` | Tests if the FTP server is vulnerable to FTP bounce attacks, where you use the server as a proxy to scan other machines. |
| `ftp-vuln-cve2010-4221.nse` | Checks for a specific buffer overflow vulnerability in ProFTPd. |
| `ftp-libopie.nse` | Tests for OTP (One-Time Password) issues in FTP implementations. |

#### Full Nmap FTP Scan

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136
```

Breaking down every flag:

- `sudo` — Run as root (needed for certain scan types)
- `nmap` — The tool name
- `-sV` — Service version detection. Nmap probes the service to determine the exact software and version running
- `-p21` — Only scan port 21 (FTP port). Without this flag, Nmap scans many ports which takes longer
- `-sC` — Run default scripts. This automatically runs all scripts in Nmap's "default" category for any service detected
- `-A` — Aggressive scan mode. Enables OS detection, version detection, script scanning, and traceroute all together
- `10.129.14.136` — The target IP address

Example output:

```
Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-16 18:12 CEST
Nmap scan report for 10.129.14.136
Host is up (0.00013s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rwxrwxrwx    1 ftp      ftp       8138592 Sep 16 17:24 Calendar.pptx [NSE: writeable]
| drwxrwxrwx    4 ftp      ftp          4096 Sep 16 17:57 Clients [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 18:05 Documents [NSE: writeable]
| drwxrwxrwx    2 ftp      ftp          4096 Sep 16 17:24 Employees [NSE: writeable]
| -rwxrwxrwx    1 ftp      ftp            41 Sep 16 17:24 Important Notes.txt [NSE: writeable]
|_-rwxrwxrwx    1 ftp      ftp             0 Sep 15 14:57 testupload.txt [NSE: writeable]
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to 10.10.14.4
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
```

What this output tells you: The `ftp-anon` script confirmed anonymous FTP login is allowed. It logged in and listed the root directory. Every file shows `[NSE: writeable]` meaning they can be modified — a serious finding. The `ftp-syst` script learned the exact vsFTPd version (3.0.3), that all connections are plain text (no encryption), the session timeout is 300 seconds, and there are 2 current client connections.

#### Nmap Script Trace

```bash
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace
```

**Purpose:** Adds `--script-trace` to the normal scan. This makes Nmap show EVERY network interaction that happens during the NSE script execution. You can see all the raw data being sent and received at the network level.

Example output showing key parts:

```
NSOCK INFO [11.4640s] nsock_connect_tcp(): TCP connection requested to 10.129.14.136:21
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50
(41 bytes): 220 Welcome to HTB-Academy FTP service...

NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.
```

You can see the exact timestamp of each event, the exact bytes received, and the raw server banner. Notice that Nmap opens multiple parallel connections using different local ports to run multiple scripts simultaneously.

---

### Service Interaction — Direct Connection to FTP

#### Connecting with Netcat

```bash
nc -nv 10.129.14.136 21
```

**Purpose:** Opens a raw TCP connection to the FTP server on port 21 using netcat. This bypasses all FTP client features and lets you communicate with the server directly using raw text commands.

- `-n` — Do not resolve hostnames (use IP directly, faster)
- `-v` — Verbose mode (shows connection status messages)
- `10.129.14.136` — Target IP
- `21` — Target port

After connecting, you see the server banner and can type FTP commands manually. This is useful when you want minimal footprint and maximum control over exactly what gets sent.

#### Connecting with Telnet

```bash
telnet 10.129.14.136 21
```

**Purpose:** Similar to netcat — opens a plain text connection to the FTP server. Telnet was originally used for terminal access but can connect to any TCP service. It gives you direct text-based interaction with the FTP server protocol. Both netcat and telnet are useful when you want to manually test specific FTP commands or check what the server responds to.

---

### FTP with TLS/SSL Encryption — Using OpenSSL

#### When is This Needed?

Some FTP servers are configured with TLS/SSL encryption (called FTPS). Standard FTP clients and netcat/telnet cannot handle encrypted connections. You need a tool that understands SSL/TLS to connect. OpenSSL's `s_client` tool can do this.

```bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp
```

Breaking down the command:

- `openssl` — The OpenSSL toolkit
- `s_client` — The SSL/TLS client component of OpenSSL
- `-connect 10.129.14.136:21` — Connect to this IP and port
- `-starttls ftp` — This is very important. It tells openssl to first connect in plain text, then send the FTP STARTTLS command to upgrade to an encrypted connection. This is how FTPS (explicit mode) works.

What the output reveals:

```
CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev,
CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1
```

The SSL certificate in the output tells you extremely valuable information:

- `C = US, ST = California, L = Sacramento` — Country, state, and city of the organization
- `O = Inlanefreight` — Organization name
- `OU = Dev` — Organizational unit (department) — this is the Dev team's certificate
- `CN = master.inlanefreight.htb` — The Common Name, which is the hostname of the server
- `emailAddress = admin@inlanefreight.htb` — The admin's email address

The error "self signed certificate" means the certificate was not issued by a trusted Certificate Authority but was created by the organization itself. This is common in internal environments. It does not prevent the connection — it just means you cannot automatically verify the server's identity.

> **Why this is useful for penetration testing:** Even without breaking the encryption, the certificate itself leaks the server's hostname, the organization name, department, admin email, and in some cases even geographic location. This is all valuable intelligence gathered passively.

---

## Section 6: SMB (Server Message Block)

### What is SMB?

SMB (Server Message Block) is a client-server network protocol that controls and manages access to shared resources across a network. These shared resources include files, entire directories, printers, routers, and other network interfaces. SMB was originally developed for Windows operating systems and first became widely available as part of the OS/2 network operating system LAN Manager and LAN Server. Beyond file sharing, SMB also handles communication between different system processes, meaning applications on different machines can exchange data using SMB. Today SMB is used in almost every Windows-based network environment, and because of backward compatibility, newer Windows versions can still communicate with very old Windows systems over SMB. This makes SMB both very widespread and a major target for penetration testers.

---

### How SMB Works

SMB works on the client-server model. The client sends a request to access a resource, and the server processes that request and either grants or denies access. Before any data is exchanged, both parties must establish a connection. In IP networks, SMB uses TCP protocol for this, which means a proper three-way handshake (SYN, SYN-ACK, ACK) happens before the session begins. After the connection is established, TCP governs all data transport. The SMB server can share parts of its local file system as "shares" — these shares appear to the client as accessible network folders. Access rights to these shares are controlled by ACLs (Access Control Lists), which can be set very specifically per user or per group with granular permissions like read-only, write, or full access.

---

### Samba — SMB for Linux

Samba is the open-source Linux/Unix implementation of the SMB protocol. It allows Linux and Unix systems to participate in Windows networks and share files with Windows machines. Samba implements CIFS (Common Internet File System), which is a specific dialect of SMB version 1 originally created by Microsoft. Modern Samba supports SMB 2 and SMB 3 as well. Samba uses two important background services: `smbd` (SMB server daemon) which provides file and print services, and `nmbd` (NetBIOS Message Block Daemon) which handles NetBIOS name resolution. When Samba communicates with older NetBIOS services, it uses TCP ports 137, 138, and 139. Modern CIFS and SMB use TCP port 445 exclusively.

---

### SMB Version History

Understanding SMB versions is important because older versions have serious known vulnerabilities (like EternalBlue which targeted SMB 1.0):

| Version | Notes |
|---|---|
| CIFS (Windows NT 4.0) | The very first version. Communication happens through the NetBIOS interface. Considered completely outdated and insecure today. |
| SMB 1.0 (Windows 2000) | Introduced direct TCP connection without NetBIOS. Still has major security flaws. The EternalBlue exploit (used in WannaCry ransomware) targeted this version. |
| SMB 2.0 (Windows Vista, Server 2008) | Major improvements including better performance, improved message signing to detect tampering, and caching features to speed up repeated requests. |
| SMB 2.1 (Windows 7, Server 2008 R2) | Added file locking mechanisms to prevent conflicts when multiple users access the same file simultaneously. |
| SMB 3.0 (Windows 8, Server 2012) | Added multichannel connections (multiple network paths for resilience), end-to-end encryption, and remote storage access. |
| SMB 3.0.2 (Windows 8.1, Server 2012 R2) | Minor updates and bug fixes. |
| SMB 3.1.1 (Windows 10, Server 2016) | Added integrity checking before authentication and AES-128 encryption for maximum security. |

With Samba version 3, the Linux Samba server could join a Windows Active Directory domain as a member. With version 4, Samba can even act as a full Active Directory domain controller.

---

### Default Samba Configuration

#### How to View Samba Configuration

```bash
cat /etc/samba/smb.conf | grep -v "#\|\;"
```

**Purpose:** Reads the main Samba configuration file and filters out all comment lines. The `grep -v "#\|\;"` part means "exclude lines containing `#` OR `;`". Both characters are used for comments in smb.conf. This gives you only the active configuration lines.

Example output:

```
[global]
   workgroup = DEV.INFREIGHT.HTB
   server string = DEVSMB
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```

Understanding the configuration structure: The `[global]` section applies settings to all shares. `workgroup` sets the Windows workgroup or domain name. `server string` is what appears when clients browse the network. `server role = standalone server` means this Samba server is not a domain member — it operates independently. `map to guest = bad user` means failed logins are treated as guest access (this can be dangerous). `usershare allow guests = yes` allows guest access to user-created shares.

#### Key SMB Share Settings and What They Mean

| Setting | Description |
|---|---|
| `[sharename]` | The name in square brackets defines the share name. This is what clients see when they browse the network. |
| `workgroup = WORKGROUP/DOMAIN` | Defines which Windows workgroup or domain this server belongs to. |
| `path = /path/here/` | The actual directory on the Linux filesystem that this share maps to. |
| `server string = STRING` | A description of the server that appears to clients when they connect. |
| `unix password sync = yes` | Synchronizes the Unix/Linux password with the SMB password so users only need one password. |
| `usershare allow guests = yes` | Allows non-authenticated (guest) users to access shares configured for guest access. |
| `map to guest = bad user` | When a login fails because the username does not exist, the connection is treated as a guest instead of being rejected. |
| `browseable = yes` | Makes the share visible in network browsing lists. If set to no, the share is hidden and can only be accessed if you know its exact name. |
| `guest ok = yes` | Allows connection to this specific share without any password. Anyone can access it anonymously. |
| `read only = yes` | Users can only read files. They cannot create, modify, or delete anything. Set to no to allow modifications. |
| `create mask = 0700` | When new files are created through SMB, they get these Unix permissions. 0700 means only the owner can read, write, execute. |

---

### Dangerous SMB Settings

| Setting | Risk |
|---|---|
| `browseable = yes` | Makes shares visible to everyone scanning the network. An attacker can see what shares exist without any authentication. |
| `read only = no` | Combined with guest access, this allows anonymous users to upload files, modify data, or potentially replace legitimate files with malicious ones. |
| `writable = yes` | Same as read only = no but explicitly enables write access. |
| `guest ok = yes` | No password required. This is the single most dangerous SMB setting because it allows complete unauthenticated access to the share. |
| `enable privileges = yes` | Honors Windows-style SID (Security Identifier) privileges, which can allow privilege escalation within the SMB context. |
| `create mask = 0777` | New files created through SMB get full permissions for everyone — any user on the Linux system can read, write, and execute them. |
| `directory mask = 0777` | New directories created through SMB get full permissions for everyone. |
| `logon script = script.sh` | Executes a shell script every time a user logs on. If an attacker controls this script, they can execute arbitrary commands on every user login. |
| `magic script = script.sh` | Executes when a specific script file is closed. This is a very dangerous setting that can lead to automatic command execution. |
| `magic output = script.out` | Specifies where the output of the magic script is stored. |

#### Creating an Example Share for Testing

Configuration to add to `/etc/samba/smb.conf`:

```
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

This creates a share called "notes" that maps to `/mnt/notes/`, is visible in network browsing, allows read and write access, allows guests without passwords, and gives full permissions to all created files. This is an extremely insecure configuration used here for demonstration purposes.

---

### Restarting Samba to Apply Configuration Changes

```bash
sudo systemctl restart smbd
```

**Purpose:** After modifying `/etc/samba/smb.conf`, you must restart the Samba service for changes to take effect. `systemctl` is the service management tool on modern Linux systems. `restart` stops the service and then starts it again. Without restarting, the old configuration remains active.

---

### Performing SMB Tasks

#### SMBclient — Listing Available Shares

```bash
smbclient -N -L //10.129.14.128
```

**Purpose:** Lists all shared folders available on the target Samba server without using any credentials.

Breaking down the flags:

- `smbclient` — The SMB client tool
- `-N` — Null session, meaning no password is used. This is anonymous access
- `-L` — List mode. Shows all available shares instead of connecting to one
- `//10.129.14.128` — The target server in UNC path format (double slash + IP)

Example output:

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

You can see five shares. `print$` and `IPC$` are default shares always present. `home`, `dev`, and `notes` are custom shares. The Type column shows "Disk" for file shares and "IPC" for inter-process communication. The Comment column shows the description set in smb.conf.

#### SMBclient — Connecting to a Specific Share

```bash
smbclient //10.129.14.128/notes
```

**Purpose:** Connects to the specific "notes" share on the target server. You are prompted for a password — pressing Enter attempts anonymous/guest access.

Full interaction example:

```
Enter WORKGROUP\<username>'s password:
Anonymous login successful
Try "help" to get a list of possible commands.

smb: \>
```

The `smb: \>` prompt means you are inside the share and can now navigate and interact with its contents.

#### help — List Available SMB Commands

```
smb: \> help
```

**Purpose:** Displays all commands available inside the SMB session.

Example output (partial):

```
?              allinfo        altname        archive        backup
blocksize      cancel         case_sensitive cd             chmod
chown          close          del            deltree        dir
du             echo           exit           get            getfacl
help           history        iosize         lcd            link
lock           lowercase      ls             l              mask
md             mget           mkdir          more           mput
newer          notify         open           posix          print
prompt         put            pwd            q              queue
quit           readlink       rd             recurse        reget
rename         reput          rm             rmdir          showacls
setea          setmode        scopy          stat           symlink
tar            tarmode        timeout        translate      unlock
volume         vuid           wdel           logon          listconnect
showconnect    tcon           tdis           tid            utimes
logoff         ..             !
```

Key commands: `ls` (list files), `get` (download file), `put` (upload file), `cd` (change directory), `mkdir` (make directory), `rm` (delete file), `!` (run local command).

#### ls — List Files in SMB Share

```
smb: \> ls
```

**Purpose:** Lists all files and directories in the current SMB share directory. Shows permissions, size, and last modified date.

Example output:

```
  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available
```

The `D` flag means directory. `N` means normal file. The number after is the file size in bytes. The last line shows available disk space on the share.

#### get — Download a File from SMB Share

```
smb: \> get prep-prod.txt
```

**Purpose:** Downloads the specified file from the SMB share to your local machine.

Example output:

```
getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec) (average 8,7 KiloBytes/sec)
```

#### ! — Run Local Commands Without Leaving SMB Session

```
smb: \> !ls
smb: \> !cat prep-prod.txt
```

**Purpose:** The exclamation mark prefix runs commands on your LOCAL machine without disconnecting from the SMB session. `!ls` lists your local directory, `!cat prep-prod.txt` reads the file you just downloaded. This is very convenient during a penetration test because you can download and immediately inspect files without leaving the SMB session.

Example interaction:

```
smb: \> !ls
prep-prod.txt

smb: \> !cat prep-prod.txt
[] check your code with the templates
[] run code-assessment.py
[] ...
```

#### smbstatus — Check Active SMB Connections (Server Side)

```bash
root@samba:~# smbstatus
```

**Purpose:** Run on the Samba server itself (requires root), this command shows all currently active SMB connections. It reveals who is connected, from which IP address, which share they are accessing, which SMB protocol version they are using, and whether encryption and signing are active.

Example output:

```
Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                          Protocol Version  Encryption  Signing
------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)  SMB3_11    -           -

Service      pid     Machine       Connected at                     Encryption   Signing
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -

No locked files
```

This shows user "sambauser" connected from 10.10.14.4, is accessing the "notes" share, and is using SMB3_11 (SMB 3.1.1). The `-` under Encryption and Signing means neither is active — a finding worth noting.

---

### Footprinting SMB with Nmap

```bash
sudo nmap 10.129.14.128 -sV -sC -p139,445
```

**Purpose:** Scans the two SMB-related ports. Port 139 is for NetBIOS session service (older SMB), port 445 is for modern SMB over TCP directly. The `-sV` detects versions, `-sC` runs default scripts.

Example output:

```
PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-09-19T13:16:04
|_  start_date: N/A
```

The output shows Samba version 4.6.2. Message signing is enabled but NOT required — meaning signing can be bypassed, which opens the server to relay attacks.

---

### RPCclient — Manual SMB Enumeration

#### What is RPC?

RPC (Remote Procedure Call) is a technology that allows a program on one computer to execute functions or procedures on another computer as if they were local. In the Windows/Samba world, RPC is used to query all kinds of information from servers. The `rpcclient` tool implements MS-RPC functions for Linux.

#### Connecting with RPCclient

```bash
rpcclient -U "" 10.129.14.128
```

**Purpose:** Connects to the SMB server using an empty username and no password (null session / anonymous access). The `-U ""` specifies an empty username. If the server allows null sessions, you get the `rpcclient $>` prompt.

Example:

```
Enter WORKGROUP\'s password:
rpcclient $>
```

Just press Enter when asked for the password. Success means null sessions are allowed — a security misconfiguration.

#### srvinfo — Get Server Information

```
rpcclient $> srvinfo
```

**Purpose:** Retrieves basic server information including the server name, OS version, and server type flags.

Example output:

```
DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
platform_id     :       500
os version      :       6.1
server type     :       0x809a03
```

The OS version `6.1` corresponds to Windows 7 / Windows Server 2008 R2. The server type flags tell you what roles the server is performing (workstation, server, print queue server, Unix machine, etc.).

#### enumdomains — List All Domains

```
rpcclient $> enumdomains
```

**Purpose:** Lists all Windows domains that are deployed and visible in the network.

Example output:

```
name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]
```

Shows the main domain (DEVSMB) and the built-in domain (which contains built-in groups like Administrators and Users).

#### querydominfo — Get Domain Information

```
rpcclient $> querydominfo
```

**Purpose:** Retrieves detailed information about the domain including the domain name, server name, total users, total groups, and domain server role.

Example output:

```
Domain:         DEVOPS
Server:         DEVSMB
Comment:        DEVSM
Total Users:    2
Total Groups:   0
Total Aliases:  0
Sequence No:    1632361158
Force Logoff:   -1
Domain Server State:    0x1
Server Role:    ROLE_DOMAIN_PDC
Unknown 3:      0x1
```

This reveals the domain is called "DEVOPS", there are 2 users, and this server is the Primary Domain Controller (PDC). Very valuable information for understanding the network structure.

#### netshareenumall — List All Shares with Paths

```
rpcclient $> netshareenumall
```

**Purpose:** Enumerates ALL available shares on the server, showing share names, descriptions, and most importantly — the actual filesystem paths on the server. This is more detailed than just listing share names.

Example output:

```
netname: print$
        remark: Printer Drivers
        path:   C:\var\lib\samba\printers
        password:
netname: home
        remark: INFREIGHT Samba
        path:   C:\home\
        password:
netname: dev
        remark: DEVenv
        path:   C:\home\sambauser\dev\
        password:
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
netname: IPC$
        remark: IPC Service (DEVSM)
        path:   C:\tmp
        password:
```

Now you know the real filesystem paths behind each share. For example, the "home" share maps to `/home/` on the server, and "notes" maps to `/mnt/notes/`. This helps you understand the server's directory structure.

#### netsharegetinfo — Detailed Info About a Specific Share

```
rpcclient $> netsharegetinfo notes
```

**Purpose:** Gets very detailed information about a specific share including its path, permissions, maximum concurrent users, and the full ACL (Access Control List) in security descriptor format.

Example output:

```
netname: notes
        remark: CheckIT
        path:   C:\mnt\notes\
        password:
        type:   0x0
        perms:  0
        max_uses:       -1
        num_uses:       1
revision: 1
type: 0x8004: SEC_DESC_DACL_PRESENT SEC_DESC_SELF_RELATIVE
DACL
        ACL     Num ACEs:       1       revision:       2
        ---
        ACE
                type: ACCESS ALLOWED (0) flags: 0x00
                Specific bits: 0x1ff
                Permissions: 0x101f01ff: Generic all access SYNCHRONIZE_ACCESS
                WRITE_OWNER_ACCESS WRITE_DAC_ACCESS READ_CONTROL_ACCESS DELETE_ACCESS
                SID: S-1-1-0
```

The SID `S-1-1-0` is the "Everyone" group — meaning ALL users (including anonymous) have full generic access to this share. `max_uses` of `-1` means unlimited simultaneous connections.

#### enumdomusers — List All Domain Users

```
rpcclient $> enumdomusers
```

**Purpose:** Lists all user accounts on the domain or server, along with their RID (Relative Identifier).

Example output:

```
user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]
```

This reveals two users: "mrb3n" with RID 0x3e8 (1000 in decimal) and "cry0l1t3" with RID 0x3e9 (1001 in decimal). These usernames can then be used for targeted brute force attacks.

#### queryuser — Get Detailed User Information by RID

```
rpcclient $> queryuser 0x3e9
```

**Purpose:** Retrieves complete details about a specific user identified by their RID. This reveals the full username, home directory path, profile path, last logon time, password last set time, account flags, and more.

Example output:

```
User Name   :   cry0l1t3
Full Name   :   cry0l1t3
Home Drive  :   \\devsmb\cry0l1t3
Dir Drive   :
Profile Path:   \\devsmb\cry0l1t3\profile
Logon Script:
Description :
Workstations:
Comment     :
Remote Dial :
Logon Time               :      Do, 01 Jan 1970 01:00:00 CET
Logoff Time              :      Mi, 06 Feb 2036 16:06:39 CET
Kickoff Time             :      Mi, 06 Feb 2036 16:06:39 CET
Password last set Time   :      Mi, 22 Sep 2021 17:50:56 CEST
Password can change Time :      Mi, 22 Sep 2021 17:50:56 CEST
Password must change Time:      Do, 14 Sep 30828 04:48:05 CEST
user_rid :      0x3e9
group_rid:      0x201
acb_info :      0x00000014
bad_password_count:     0x00000000
logon_count:    0x00000000
```

#### querygroup — Get Group Information

```
rpcclient $> querygroup 0x201
```

**Purpose:** Retrieves information about a specific group identified by its RID.

Example output:

```
Group Name:     None
Description:    Ordinary Users
Group Attribute:7
Num Members:2
```

Group 0x201 is the standard user group "None" (equivalent to Domain Users) with 2 members — the same 2 users found earlier.

---

### Brute Forcing User RIDs with RPCclient

#### Why Brute Force RIDs?

Windows and Samba assign RIDs to users and groups sequentially starting from 500 for built-in accounts and 1000+ for regular accounts. By querying each RID in sequence, you can discover ALL users on the system even if `enumdomusers` is restricted.

```bash
for i in $(seq 500 1100); do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done
```

**Purpose:** This bash loop goes through numbers 500 to 1100, converts each to hexadecimal using `printf '%x\n'`, then sends a `queryuser` RPC command for each one. If a user with that RID exists, it prints the username and group information.

Example output:

```
        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201

        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201

        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201
```

Three users discovered. Sambauser has RID 0x1f5 (501 decimal — a built-in account), while mrb3n and cry0l1t3 are regular users starting at 1000.

---

### Impacket samrdump.py — Automated User Enumeration

```bash
samrdump.py 10.129.14.128
```

**Purpose:** A Python script from the Impacket toolkit that automatically connects to the SAM (Security Account Manager) database over SMB and enumerates all users. It provides much more detail than manual RID brute forcing.

Example output:

```
[*] Retrieving endpoint list from 10.129.14.128
Found domain(s):
 . DEVSMB
 . Builtin
[*] Looking up users in domain DEVSMB
Found user: mrb3n, uid = 1000
Found user: cry0l1t3, uid = 1001
mrb3n (1000)/FullName:
mrb3n (1000)/UserComment:
mrb3n (1000)/PrimaryGroupId: 513
mrb3n (1000)/BadPasswordCount: 0
mrb3n (1000)/LogonCount: 0
mrb3n (1000)/PasswordLastSet: 2021-09-22 17:47:59
mrb3n (1000)/PasswordDoesNotExpire: False
mrb3n (1000)/AccountIsDisabled: False
cry0l1t3 (1001)/FullName: cry0l1t3
cry0l1t3 (1001)/PasswordLastSet: 2021-09-22 17:50:56
cry0l1t3 (1001)/PasswordDoesNotExpire: False
cry0l1t3 (1001)/AccountIsDisabled: False
[*] Received 2 entries.
```

---

### SMBmap — Check Share Permissions Quickly

```bash
smbmap -H 10.129.14.128
```

**Purpose:** Connects to the SMB server and quickly shows all available shares along with the exact permissions your current user has on each one (READ, WRITE, or NO ACCESS). The `-H` flag specifies the host IP address.

Example output:

```
[+] Finding open SMB ports....
[+] User SMB session established on 10.129.14.128...
[+] IP: 10.129.14.128:445       Name: 10.129.14.128
        Disk                          Permissions     Comment
        ----                          -----------     -------
        print$                        NO ACCESS       Printer Drivers
        home                          NO ACCESS       INFREIGHT Samba
        dev                           NO ACCESS       DEVenv
        notes                         NO ACCESS       CheckIT
        IPC$                          NO ACCESS       IPC Service (DEVSM)
```

---

### CrackMapExec — SMB Enumeration with Credentials

```bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''
```

**Purpose:** CrackMapExec (CME) is a powerful post-exploitation tool for SMB enumeration. This command connects with empty username and password (anonymous) and lists all shares with their permissions. Unlike SMBmap, CME also shows the Windows version, hostname, SMB signing status, and whether SMBv1 is enabled.

Example output:

```
SMB  10.129.14.128  445  DEVSMB  [*] Windows 6.1 Build 0 (name:DEVSMB) (domain:) (signing:False) (SMBv1:False)
SMB  10.129.14.128  445  DEVSMB  [+] \:
SMB  10.129.14.128  445  DEVSMB  [+] Enumerated shares
SMB  10.129.14.128  445  DEVSMB  Share           Permissions     Remark
SMB  10.129.14.128  445  DEVSMB  -----           -----------     ------
SMB  10.129.14.128  445  DEVSMB  print$                          Printer Drivers
SMB  10.129.14.128  445  DEVSMB  home                            INFREIGHT Samba
SMB  10.129.14.128  445  DEVSMB  dev                             DEVenv
SMB  10.129.14.128  445  DEVSMB  notes           READ,WRITE      CheckIT
SMB  10.129.14.128  445  DEVSMB  IPC$                            IPC Service (DEVSM)
```

> **Critical finding:** `signing:False` means SMB signing is not required — this allows relay attacks. `SMBv1:False` is good — SMBv1 is disabled. The "notes" share has `READ,WRITE` access for anonymous users — a major vulnerability.

---

### Enum4Linux-ng — Complete Automated SMB Enumeration

#### Installation

```bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
```

**Purpose:** Clones the enum4linux-ng repository from GitHub, enters the directory, and installs all required Python dependencies. Enum4linux-ng is the modern rewrite of the older enum4linux tool. It automates all SMB enumeration tasks including LDAP checks, SMB dialect detection, NetBIOS information, OS detection, user enumeration, group enumeration, share enumeration, policy enumeration, and printer enumeration.

#### Running Full Enumeration

```bash
./enum4linux-ng.py 10.129.14.128 -A
```

**Purpose:** Runs ALL enumeration modules against the target. The `-A` flag means "All" — it enables every available check. This is the most comprehensive SMB enumeration you can do with a single command.

Key sections of the output explained:

Service Scan:
```
[*] Checking LDAP — [-] Could not connect (not a domain controller)
[*] Checking SMB — [+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS — [+] accessible on 139/tcp
```

NetBIOS Information:
```
[+] Got domain/workgroup name: DEVOPS
[+] Full NetBIOS names information:
- DEVSMB <00> - Workstation Service
- DEVSMB <20> - File Server Service
- DEVOPS <00> - Domain/Workgroup Name
- DEVOPS <1d> - Master Browser
```

SMB Dialect Check:
```
SMB 1.0: false
SMB 2.02: true
SMB 2.1: true
SMB 3.0: true
Preferred dialect: SMB 3.0
SMB signing required: false
```

RPC Session Check:
```
[+] Server allows session using username '', password ''
[+] Server allows session using username 'juzgtcsu', password ''
```

The second line means the server allows ANY random username with no password — very insecure.

Users:
```
'1000':
  username: mrb3n
  acb: '0x00000010'
'1001':
  username: cry0l1t3
  acb: '0x00000014'
```

Password Policy:
```
domain_password_information:
  min_pw_length: 5
  DOMAIN_PASSWORD_COMPLEX: false
```

Minimum password length is only 5 characters and complexity is not required — very weak password policy, making brute force attacks much easier.


---

## Section 7: NFS (Network File System)

### What is NFS?

NFS (Network File System) was developed by Sun Microsystems and serves a similar purpose to SMB — it allows remote file systems to be mounted and accessed as if they were local. The key difference is that NFS is designed for Linux and Unix systems, while SMB is for Windows. NFS clients cannot directly communicate with SMB servers and vice versa. NFS uses RPC (Remote Procedure Call) as its underlying communication mechanism and traditionally uses ports 111 (RPC portmapper) and 2049 (NFS service). Authentication in NFS versions 2 and 3 is based on UNIX UID/GID numbers — this means the server trusts whatever UID the client claims to have, which creates security risks if the server and client have different user mappings. NFSv4 adds Kerberos authentication and firewall-friendly operation using only port 2049.

---

### NFS Version Comparison

| Version | Notes |
|---|---|
| NFSv2 | The oldest version, supported by many legacy systems. Originally operated entirely over UDP. Very limited features. Rarely seen in modern environments. |
| NFSv3 | More commonly seen. Supports variable file sizes and better error reporting. Not fully compatible with NFSv2 clients. Still uses port 111 for portmapping plus dynamic ports for services. |
| NFSv4 | The modern standard. Includes Kerberos authentication, works through firewalls (uses only port 2049), supports ACLs, uses stateful operations, and provides performance improvements and high security. NFSv4.1 adds parallel NFS (pNFS) for distributed file access and session trunking (NFS multipathing) for redundant connections. |

---

### NFS Configuration — /etc/exports

#### Viewing the NFS Exports File

```bash
cat /etc/exports
```

**Purpose:** Reads the NFS exports configuration file. This file defines which local directories are shared (exported) over NFS and who can access them with what permissions. Each line defines one export: the directory path, the allowed hosts/networks, and the options in parentheses.

Example content:

```
# /etc/exports: the access control list for filesystems which may be exported
# Example for NFSv2 and NFSv3:
# /srv/homes hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
# Example for NFSv4:
# /srv/nfs4  gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
```

The comments show the format: directory, then hostname or subnet in parentheses with options.

#### Adding an NFS Share

```bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs
```

Purpose of each command:

- `echo '/mnt/nfs 10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports` — Adds a new export entry. This shares the `/mnt/nfs` directory to all hosts in the `10.129.14.0/24` subnet. The `sync` option means data is written to disk before the write is confirmed to the client (safer). The `no_subtree_check` disables checking whether files requested are within the exported subtree (improves reliability).
- `systemctl restart nfs-kernel-server` — Restarts the NFS server to apply the new exports configuration.
- `exportfs` — Displays the currently active NFS exports to confirm the new share is active.

Output of exportfs:
```
/mnt/nfs        10.129.14.0/24
```

#### NFS Export Options Explained

| Option | Description |
|---|---|
| `rw` | Read and Write permissions. Clients can read existing files AND create/modify/delete files. |
| `ro` | Read Only permissions. Clients can only read files — cannot make any changes. |
| `sync` | Synchronous data transfer. The server confirms writes only after data is physically written to disk. Slightly slower but safer — no data loss if the server crashes. |
| `async` | Asynchronous data transfer. The server confirms writes immediately without waiting for disk write. Faster but risks data loss if the server crashes before flushing to disk. |
| `secure` | Only allow connections from client ports below 1024. Ports below 1024 require root on the client, so this adds a small layer of security. |
| `insecure` | Allows connections from ANY client port including ports above 1024. Removes the port restriction. |
| `no_subtree_check` | Disables checking whether a requested file is actually within the exported directory tree. Improves reliability and performance. |
| `root_squash` | When the client connects as root (UID 0), the server maps that root access to the anonymous user (nobody). This prevents a root user on the client from having root access on the NFS server. This is ON by default in most configurations. |
| `no_root_squash` | The opposite of root_squash. Root on the client is treated as root on the server. This is DANGEROUS because anyone who gets root on the NFS client can modify any file on the NFS server including system files. |
| `nohide` | If another filesystem is mounted below an exported directory, normally it is hidden from NFS clients. With nohide, it becomes visible and is exported separately. |

---

### Dangerous NFS Settings

- **`rw + no_root_squash`** — Combined, these two settings are extremely dangerous. Any client with root access gets read/write access to the entire NFS share with root privileges on the server side. An attacker who gains root on any client machine can completely compromise the NFS server's data.
- **`insecure`** — Allows connections from ports above 1024. Since only root can use ports below 1024, the `secure` option adds a small but meaningful barrier. Removing it with `insecure` means any process can make NFS connections.
- **`nohide`** — Can accidentally expose nested filesystems that administrators did not intend to share.

---

### Footprinting NFS with Nmap

#### Basic NFS Port Scan

```bash
sudo nmap 10.129.14.128 -p111,2049 -sV -sC
```

**Purpose:** Scans the two critical NFS ports. Port 111 is the RPC portmapper that tells clients which ports each RPC service is using. Port 2049 is the NFS service itself. The `-sV` detects versions, `-sC` runs default scripts including the rpcinfo script.

Example output:

```
PORT    STATE SERVICE VERSION
111/tcp open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100003  3           2049/udp   nfs
|   100003  3,4         2049/tcp   nfs
|   100005  1,2,3      45837/tcp   mountd
|   100005  1,2,3      58830/udp   mountd
|   100021  1,3,4      44629/tcp   nlockmgr
|   100227  3           2049/tcp   nfs_acl
2049/tcp open  nfs_acl 3 (RPC #100227)
```

The rpcinfo output shows all currently running RPC services and their ports. `mountd` (port 45837) handles mount requests, `nlockmgr` handles file locking, and `nfs_acl` handles ACLs.

#### NFS-Specific NSE Scripts Scan

```bash
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049
```

**Purpose:** The `nfs*` wildcard runs ALL Nmap scripts whose names start with "nfs". This includes `nfs-ls` (list directory contents), `nfs-showmount` (show exports), and `nfs-statfs` (disk statistics). Together these scripts give a comprehensive view of the NFS service without needing to mount anything.

Example output:

```
PORT     STATE SERVICE VERSION
111/tcp  open  rpcbind 2-4 (RPC #100000)
| nfs-ls: Volume /mnt/nfs
|   access: Read Lookup NoModify NoExtend NoDelete NoExecute
| PERMISSION  UID    GID    SIZE  TIME                 FILENAME
| rwxrwxrwx   65534  65534  4096  2021-09-19T15:28:17  .
| rw-r--r--   0      0      1872  2021-09-19T15:27:42  id_rsa
| rw-r--r--   0      0      348   2021-09-19T15:28:17  id_rsa.pub
| rw-r--r--   0      0      0     2021-09-19T15:22:30  nfs.share
|_
| nfs-showmount:
|_  /mnt/nfs 10.129.14.0/24
| nfs-statfs:
|   Filesystem  1K-blocks   Used       Available   Use%  Maxfilesize  Maxlink
|_  /mnt/nfs    30313412.0  8074868.0  20675664.0  29%   16.0T        32000
```

> This is extremely valuable. Without even mounting the share, you can see there are SSH private key files (`id_rsa` and `id_rsa.pub`) owned by root (UID 0) sitting in the NFS share. The disk is 29% used with 20GB available.

---

### Performing NFS Tasks — Mounting and Accessing Shares

#### Step 1 — Show Available NFS Shares on Remote Server

```bash
showmount -e 10.129.14.128
```

**Purpose:** Queries the remote NFS server for its export list — showing which directories are shared and which clients/networks are allowed to mount them.

Example output:

```
Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24
```

#### Step 2 — Create a Local Mount Point

```bash
mkdir target-NFS
```

**Purpose:** Creates an empty local directory called `target-NFS` that will serve as the mount point. The NFS share contents will appear inside this directory after mounting.

#### Step 3 — Mount the NFS Share

```bash
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
```

**Purpose:** Mounts the NFS share from the remote server onto your local directory.

Breaking down each part:

- `sudo` — Root privileges required to mount filesystems
- `mount` — The mount command
- `-t nfs` — Specifies the filesystem type is NFS
- `10.129.14.128:/` — The remote server IP and the path being mounted (root of the server's NFS exports)
- `./target-NFS/` — The local directory to mount into
- `-o nolock` — Option to disable NFS file locking (prevents issues when the NFS server doesn't support the lock manager)

#### Step 4 — Navigate and View Mounted Content

```bash
cd target-NFS
tree .
```

**Purpose:** Enters the mounted NFS directory and displays the file structure in tree format.

Example output:

```
.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files
```

You now have local access to all these files as if they were on your own machine.

#### Step 5 — List Files with Usernames and Group Names

```bash
ls -l mnt/nfs/
```

**Purpose:** Lists files with their symbolic owner names (username and group name). This requires that the UID/GID numbers on the mounted files match existing users on YOUR local machine. If they do, you see real names.

Example output:

```
total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share
```

Files owned by "cry0l1t3" and "root" are visible. If you create a local user with the same UID as cry0l1t3, you can read their private files.

#### Step 6 — List Files with UIDs and GIDs (Numeric)

```bash
ls -n mnt/nfs/
```

**Purpose:** The `-n` flag shows numeric UIDs and GIDs instead of resolving them to names. This is important for penetration testing because it shows you the exact numeric IDs, which you can use to create matching local users to gain access to the files.

Example output:

```
total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share
```

Now you can see the exact UIDs. User "cry0l1t3" is UID 1000. The `id_rsa` files are owned by root (UID 0). The `backup.sh` is owned by root (UID 0) but in the cry0l1t3 group (GID 1000).

> **Important note:** If `root_squash` is enabled, even if you are root on your local machine, you cannot modify files owned by root on the NFS share because root_squash maps your root access to nobody.

#### Step 7 — Unmount the NFS Share

```bash
cd ..
sudo umount ./target-NFS
```

**Purpose:** First navigate OUT of the mounted directory (you cannot unmount a filesystem you are currently inside), then unmount it. `umount` removes the NFS mount and disconnects from the remote server. Always unmount when done to clean up properly and avoid leaving open connections.


---

## Section 8: DNS (Domain Name System)

### What is DNS?

DNS (Domain Name System) is the internet's phonebook. It translates human-readable domain names like `hackthebox.com` into IP addresses like `10.10.10.1`. DNS does not use a central database — instead, information is distributed across thousands of name servers worldwide. There are different types of DNS servers: Root Servers (13 worldwide, manage top-level domains), Authoritative Nameservers (hold official records for a zone), Non-authoritative Nameservers (collect info via recursive queries), Caching Servers (store results temporarily), Forwarding Servers (forward queries to another server), and Resolvers (local to the computer or router). DNS is mostly unencrypted, making queries visible to ISPs and network devices. Modern solutions like DNS over TLS (DoT) and DNS over HTTPS (DoH) address this privacy concern.

---

### DNS Record Types

| Record | Description |
|---|---|
| `A` | Maps domain to IPv4 address |
| `AAAA` | Maps domain to IPv6 address |
| `MX` | Mail server records (which server handles email). Has a priority number — lower number means higher priority. |
| `NS` | Name server records (which DNS servers manage the domain) |
| `TXT` | Text records (verification codes, SPF, DMARC, DKIM) |
| `CNAME` | Alias record (one domain points to another) |
| `PTR` | Reverse lookup (IP address → domain name) |
| `SOA` | Start of Authority (admin contact, serial number, zone info) |

---

### DNS Configuration Files (BIND9)

BIND9 is the most common DNS server software on Linux. Its configuration is split across three files:

- `named.conf.local` — Defines zones (which domains this server is authoritative for)
- `named.conf.options` — Global server settings
- `named.conf.log` — Logging configuration

#### Zone File

```bash
cat /etc/bind/db.domain.com
```

The zone file maps hostnames to IP addresses. It contains SOA record, NS records, MX records, A records, and CNAME records. Every zone file must have exactly one SOA record and at least one NS record.

Example content:

```
$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial number - increment on every change
                    21600      ; refresh - slaves check every 6 hours
                    3600       ; retry - if refresh fails, retry after 1 hour
                    604800     ; expire - slaves discard records after 1 week
                    86400 )    ; minimum TTL - 1 day

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.
      IN     MX     10     mx.domain.com.
      IN     MX     20     mx2.domain.com.
             IN     A       10.129.14.5

server1      IN     A       10.129.14.5
server2      IN     A       10.129.14.7
ns1          IN     A       10.129.14.2
ns2          IN     A       10.129.14.3

ftp          IN     CNAME   server1
mx           IN     CNAME   server1
mx2          IN     CNAME   server2
www          IN     CNAME   server2
```

#### Reverse Zone File

```bash
cat /etc/bind/db.10.129.14
```

The reverse zone file maps IP addresses back to hostnames using PTR records.

Example content:

```
$ORIGIN 14.129.10.in-addr.arpa
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial
                    21600      ; refresh
                    3600       ; retry
                    604800     ; expire
                    86400 )    ; minimum TTL

      IN     NS     ns1.domain.com.
      IN     NS     ns2.domain.com.

5    IN     PTR    server1.domain.com.
7    IN     MX     mx.domain.com.
```

---

### Dangerous DNS Settings

| Setting | Risk |
|---|---|
| `allow-query { any; }` | Allows ANY host on the internet to send DNS queries. Combined with recursion, this can make the server an amplifier for DDoS attacks. |
| `allow-recursion { any; }` | Allows recursive queries from any host. A recursive query makes the DNS server do all the work of finding the answer. Enabling this from external hosts enables DNS amplification attacks. |
| `allow-transfer { any; }` | The most dangerous setting. Allows ANY host to request a full zone transfer (AXFR). This lets anyone download the complete list of ALL DNS records — revealing every hostname and IP address in the organization. |
| `zone-statistics` | Enables collection of query statistics. If accessible, reveals what domains are being queried and how often. |

---

### Footprinting DNS

#### DIG — NS Query (Find Name Servers)

```bash
dig ns inlanefreight.htb @10.129.14.128
```

**Purpose:** Queries the specified DNS server for the NS records of the domain `inlanefreight.htb`. This reveals which name servers are authoritative for the domain. The `@10.129.14.128` specifies which DNS server to ask.

Example output:

```
;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
```

The answer shows there is one name server: `ns.inlanefreight.htb` at IP `10.129.34.136`. The 604800 is the TTL (604800 seconds = 1 week). Now you know the IP of the authoritative DNS server.

#### DIG — Version Query (Identify DNS Software Version)

```bash
dig CH TXT version.bind 10.129.120.85
```

**Purpose:** Sends a special CHAOS class TXT query asking for the DNS server's version information. Many DNS servers are configured to respond to this query with their software version. This is useful for finding known vulnerabilities in specific DNS software versions.

Breaking down:

- `CH` — The CHAOS class (a special DNS class separate from the normal IN/Internet class)
- `TXT` — Record type
- `version.bind` — Special query name that requests version information from BIND servers

Example output:

```
;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1"

;; ADDITIONAL SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"
```

The DNS server is running BIND version 9.10.6-P1 on Debian Linux. This specific version can be searched for known CVEs.

#### DIG — ANY Query (Retrieve All Record Types)

```bash
dig any inlanefreight.htb @10.129.14.128
```

**Purpose:** Requests ALL available record types for the domain in a single query. The server returns everything it is willing to disclose.

Example output:

```
;; ANSWER SECTION:
inlanefreight.htb. 604800 IN TXT "v=spf1 include:mailgun.org include:_spf.google.com..."
inlanefreight.htb. 604800 IN TXT "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1..."
inlanefreight.htb. 604800 IN TXT "MS=ms97310371"
inlanefreight.htb. 604800 IN SOA inlanefreight.htb. root.inlanefreight.htb. 2 604800...
inlanefreight.htb. 604800 IN NS  ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
```

From this single query you can learn: the company uses Mailgun and Google for email, they use Atlassian tools, the SOA record shows `root@inlanefreight.htb` is the admin contact, and the only NS is at `10.129.34.136`.

#### DIG — AXFR Zone Transfer (External Zone)

```bash
dig axfr inlanefreight.htb @10.129.14.128
```

**Purpose:** Attempts a full zone transfer (AXFR - Asynchronous Full Transfer Zone) from the target DNS server. If the server is misconfigured to allow zone transfers from any IP, this downloads the COMPLETE zone file — every single DNS record for the domain.

Example output:

```
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800 86400 2419200 604800
inlanefreight.htb.      604800  IN      TXT     "MS=ms97310371"
inlanefreight.htb.      604800  IN      TXT     "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1QksWdXuYFUeSXKU"
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
internal.inlanefreight.htb. 604800 IN   A       10.129.1.6
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
inlanefreight.htb.      604800  IN      SOA     inlanefreight.htb. root.inlanefreight.htb. 2 604800...
;; XFR size: 9 records (messages 1, bytes 520)
```

This reveals ALL hostnames and their IPs: an app server at 10.129.18.15, an internal server at 10.129.1.6, a mail server at 10.129.18.201. The zone transfer exposed the complete internal network map.

#### DIG — AXFR Zone Transfer (Internal Zone)

```bash
dig axfr internal.inlanefreight.htb @10.129.14.128
```

**Purpose:** Attempts zone transfer for the INTERNAL subdomain zone. Internal zones often contain even more sensitive information — domain controllers, VPN servers, internal workstations, and management systems.

Example output:

```
internal.inlanefreight.htb. 604800 IN   SOA     inlanefreight.htb. root.inlanefreight.htb...
internal.inlanefreight.htb. 604800 IN   NS      ns.inlanefreight.htb.
dc1.internal.inlanefreight.htb. 604800 IN A     10.129.34.16
dc2.internal.inlanefreight.htb. 604800 IN A     10.129.34.11
mail1.internal.inlanefreight.htb. 604800 IN A   10.129.18.200
ns.internal.inlanefreight.htb. 604800 IN A      10.129.34.136
vpn.internal.inlanefreight.htb. 604800 IN A     10.129.1.6
ws1.internal.inlanefreight.htb. 604800 IN A     10.129.1.34
ws2.internal.inlanefreight.htb. 604800 IN A     10.129.1.35
wsus.internal.inlanefreight.htb. 604800 IN A    10.129.18.2
;; XFR size: 15 records
```

This is a gold mine. Two domain controllers (dc1 at 10.129.34.16, dc2 at 10.129.34.11), a VPN server, two workstations (ws1, ws2), a WSUS (Windows Update) server, and internal mail. This is the complete internal network structure revealed by one DNS misconfiguration.

#### Subdomain Brute Force with Bash Loop

```bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done
```

**Purpose:** Goes through every word in the subdomain wordlist, constructs a DNS query for each potential subdomain, and saves any that resolve to a file.

Breaking down:

- `cat /opt/useful/seclists/.../subdomains-top1million-110000.txt` — Reads a list of 110,000 common subdomain names
- `for sub in $(...)` — Loops through each subdomain name
- `dig $sub.inlanefreight.htb @10.129.14.128` — Queries for each potential subdomain
- `grep -v ';\|SOA'` — Removes comment lines and SOA records
- `sed -r '/^\s*$/d'` — Removes empty lines
- `grep $sub` — Keeps only lines containing the subdomain name (meaning it resolved)
- `tee -a subdomains.txt` — Writes matching results to file AND displays them

Example output:

```
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
```

#### DNSenum — Automated DNS Enumeration

```bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb
```

**Purpose:** DNSenum automates the complete DNS enumeration process including NS queries, zone transfer attempts, and subdomain brute forcing.

Breaking down the flags:

- `--dnsserver 10.129.14.128` — Use this specific DNS server for all queries
- `--enum` — Enable full enumeration mode
- `-p 0` — No Google scraping (0 Google pages)
- `-s 0` — No Google scraping subdomains
- `-o subdomains.txt` — Save results to this file
- `-f /opt/.../subdomains-top1million-110000.txt` — Use this wordlist for brute forcing
- `inlanefreight.htb` — Target domain
