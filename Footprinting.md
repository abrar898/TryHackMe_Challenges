Complete Notes: Footprinting & Network Services Enumeration
Section 1: Enumeration Methodology
What is Enumeration Methodology?

Enumeration methodology is a standardized, step-by-step approach used during penetration testing to gather information about a target system in an organized way. Without a proper methodology, a penetration tester might miss important parts of the target or waste time going in the wrong direction. Because penetration testing involves many different types of systems, networks, and configurations, having a fixed but flexible structure helps the tester stay on track. The methodology is not a rigid checklist but rather a systematic framework that allows for changes depending on what the tester finds. It is divided into three levels: Infrastructure-based enumeration, Host-based enumeration, and OS-based enumeration. These three levels together cover everything from the internet presence of a company to the internal operating system configuration of its servers.

The 6 Layers of Enumeration

The entire enumeration process is organized into 6 layers, each representing a boundary or "wall" that the tester tries to pass through to get closer to the target. Think of it like a labyrinth where each layer is a ring, and the tester needs to find the gap in each ring to move forward. Each layer has specific goals and information categories that the tester focuses on.

Layer 1 — Internet Presence

This is the outermost layer. Here the tester finds out what the company looks like from the internet. The goal is to find all possible targets such as domains, subdomains, IP addresses, cloud instances, and other infrastructure that is visible on the internet. This layer is especially important in black-box tests where no information is given upfront. The tester uses passive techniques — meaning they do not directly connect to the target — to gather information. Tools like crt.sh, Shodan, and DNS lookups are used at this stage. Information gathered includes domains, subdomains, virtual hosts (vHosts), ASN (Autonomous System Numbers), netblocks, IP addresses, cloud instances, and security measures in place.

Layer 2 — Gateway

Once the internet presence is identified, the next step is to understand how the company protects its infrastructure. This layer focuses on identifying the protective measures between the internet and the internal network. The goal is to understand what kind of security is in place, such as firewalls, DMZ (Demilitarized Zones), IPS/IDS systems, EDR tools, proxies, NAC (Network Access Control), VPNs, and services like Cloudflare. Understanding the gateway helps the tester know what kind of traffic is blocked and what might pass through without triggering alarms.

Layer 3 — Accessible Services

This is the layer that is mostly covered in the main footprinting module. Here the tester identifies all services running on the target systems that can be accessed from outside. Each service has a specific purpose — for example, FTP for file transfer, SSH for remote access, HTTP for web traffic. The tester looks at the service type, its version, its configuration, which port it is running on, and what functionality it provides. The goal is to understand how the service works and how it can be communicated with, which leads to finding ways to exploit it.

Layer 4 — Processes

Every time a command or function is executed on a server, some process runs behind the scenes. This layer focuses on identifying what internal processes are running, what data they handle, and what sources and destinations are involved. For example, a web server process might be processing requests from users and sending them to a database. Understanding these processes helps the tester identify dependencies and potential weak points. Key information includes PID (Process ID), the type of data being processed, tasks being performed, and source/destination details.

Layer 5 — Privileges

Every service on a system runs under a specific user account with specific permissions. Sometimes administrators give services more permissions than needed, or they forget to review permission settings. This layer is about identifying what permissions are attached to each service and each user, and whether those permissions can be misused. This is especially important in Active Directory environments where users might have permissions across many systems. The tester looks at groups, users, permissions, restrictions, and the overall environment.

Layer 6 — OS Setup

This is the innermost layer. At this point, the tester already has internal access to the system and is gathering information about the operating system itself. This includes the OS type (Linux, Windows), the patch level (is it up to date?), network configuration, environment variables, configuration files, and sensitive private files. The goal is to understand how the administrator has set up and maintains the system, and to find any sensitive internal information that can be used for further exploitation.

Section 2: Domain Information
What is Domain Information?

Domain information is one of the most important parts of the passive reconnaissance phase in penetration testing. It is not only about finding subdomains but about understanding the full internet presence of a company — what technologies they use, what services they run, and how their infrastructure is structured. This entire process is done passively, meaning the tester acts like a normal internet user or customer and does not send any direct scans or requests to the target that could alert the company. The information gathered here gives a big-picture view of the target before any active testing begins. Think of it as reading everything that is publicly available about the company without knocking on their door.

Checking the Company Website

The first step is always to visit the company's official website and read the content carefully. The website tells you what kind of services the company offers, what technologies they use, and what their business model is. For example, if a company offers IoT solutions, app development, and cloud hosting, then you know they likely run servers, APIs, and possibly cloud storage. Reading the website with a developer's eye helps you guess what kind of backend infrastructure might be in place. Even small details like "built with Django" or "powered by AWS" in a footer or page source code reveal important technical information.

SSL Certificates and crt.sh
What is crt.sh?

crt.sh is a website that stores Certificate Transparency logs. When any website gets an SSL certificate from a Certificate Authority like Let's Encrypt, a public log entry is created. This log can be searched to find all subdomains that a company has ever used a certificate for. This is extremely useful because companies often have many subdomains — some of which might be forgotten or misconfigured. By searching crt.sh for a domain, you can find subdomains that are not listed anywhere else publicly.

How to Use crt.sh via Command Line

Command:

bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq .

Purpose: This command fetches JSON data from crt.sh about all SSL certificates issued for the domain inlanefreight.com. The jq . part formats the JSON output to make it readable.

To filter only unique subdomains:

bash
curl -s https://crt.sh/\?q\=inlanefreight.com\&output\=json | jq . | grep name | cut -d":" -f2 | grep -v "CN=" | cut -d'"' -f2 | awk '{gsub(/\\n/,"\n");}1;' | sort -u

This command filters out only the subdomain names from the JSON response, removes certificate subject names (CN=), and sorts the results to show only unique entries. The output gives you a clean list of all discovered subdomains.

Finding Company-Hosted Servers
Purpose

After getting a list of subdomains, the next step is to find which ones point to IP addresses that belong to the company itself (not third-party providers). This is important because you can only test hosts that are within the scope of your engagement. Third-party hosted services (like AWS, Cloudflare) require separate permission.

Command:
bash
for i in $(cat subdomainlist); do host $i | grep "has address" | grep inlanefreight.com | cut -d" " -f1,4; done

This loop goes through each subdomain in your list, uses the host command to resolve its IP address, and then filters only those that belong to the target domain. The output shows the subdomain and its resolved IP address.

Using Shodan for IP Intelligence
What is Shodan?

Shodan is a search engine for internet-connected devices. Unlike Google which indexes web content, Shodan indexes open ports, service banners, and device information. It can show what services are running on a given IP address, what software versions are in use, and what ports are open.

Command:
bash
for i in $(cat ip-addresses.txt); do shodan host $i; done

This loops through all discovered IP addresses and queries Shodan for information about each one. The output might show open ports like 22 (SSH), 80 (HTTP), 443 (HTTPS), along with the software running on them (like nginx, Apache), SSL certificate details, and even the city and organization the IP belongs to.

DNS Records with dig
What is dig?

dig (Domain Information Groper) is a command-line tool used to query DNS servers and retrieve DNS records. DNS records are like a phonebook for the internet — they map domain names to IP addresses and other information.

Command:
bash
dig any inlanefreight.com

This retrieves all available DNS record types for the domain. The main record types and what they tell you:

A Records — Map a domain name to an IPv4 address. Shows you which servers the domain points to.

MX Records — Mail exchange records. Tell you which servers handle email for the domain. If Google handles email, this tells you they use Gmail/Google Workspace.

NS Records — Name server records. Show which DNS servers are responsible for the domain. The hosting provider often runs these, so NS records can reveal who is hosting the domain.

TXT Records — Text records used for verification and security purposes. These can contain verification codes for services like Google, Atlassian, and LogMeIn. They also contain SPF (Sender Policy Framework) records which reveal what email services and IP addresses the company uses.

SOA Record — Start of Authority. Contains administrative information about the domain including the primary DNS server and the contact email for the domain.

What TXT Records Reveal

From TXT records, a tester can identify third-party tools and services the company uses. For example:

atlassian-domain-verification → Company uses Atlassian (Jira, Confluence, Bitbucket)
google-site-verification → Company uses Google services; might have open Google Drive links
logmein-verification-code → Company uses LogMeIn for remote access management
v=spf1 include:mailgun.org → Company uses Mailgun for email APIs
include:spf.protection.outlook.com → Company uses Microsoft 365/Azure

Each of these reveals attack surfaces — for example, Mailgun APIs might be vulnerable to IDOR or SSRF attacks, and Azure file storage can be accessed via SMB protocol.

Section 3: Cloud Resources
What are Cloud Resources in Penetration Testing?

Modern companies use cloud services from AWS, Google Cloud (GCP), and Microsoft Azure for storage, computing, and hosting. While cloud providers themselves are very secure, the configurations that companies apply can introduce vulnerabilities. The most common issue is misconfigured storage — such as AWS S3 buckets, Azure Blobs, or GCP Cloud Storage — that are set to public access without authentication. During a penetration test, discovering these misconfigured cloud resources can lead to accessing sensitive files like documents, source code, private keys, and even credentials.

Finding Cloud Storage with Google Dorks
What are Google Dorks?

Google Dorks are advanced Google search operators that narrow down search results to very specific types of pages. In the context of cloud resources, they help find public files stored in AWS or Azure that are indexed by Google.

For AWS S3 Buckets:

intext:companyname inurl:amazonaws.com

For Azure Blobs:

intext:companyname inurl:blob.core.windows.net

These searches can return PDF files, documents, images, and other files that companies have accidentally made public in their cloud storage. Sometimes these files contain sensitive business data.

Finding Cloud Resources in Website Source Code

Companies often load images, JavaScript files, and CSS from their cloud storage to reduce load on web servers. If you inspect the HTML source code of a company's website, you might find links to blob.core.windows.net or s3.amazonaws.com which reveal the cloud storage URLs. These can then be further investigated to see if they allow public access or directory listing.

Third-Party Tools for Cloud Enumeration
Domain.Glass

Domain.Glass is an online tool that provides information about a domain's infrastructure. It can reveal cloud service providers, CDN usage (like Cloudflare), and basic security assessment information. If Cloudflare is detected, it is noted as a security measure in Layer 2 (Gateway) of the enumeration methodology.

GrayHatWarfare

GrayHatWarfare is a specialized search engine for public cloud storage. It indexes publicly accessible files in AWS S3, Azure Blob, and GCP Cloud Storage. A tester can search by company name, filter by file type (PDF, DOCX, XLSX, etc.), and discover what data has been accidentally made public. This is a completely passive approach and requires no interaction with the target's own systems.

Risk: Leaked SSH Private Keys

One of the most dangerous things that can appear in misconfigured cloud storage is leaked SSH private keys. An SSH private key (typically a file named id_rsa) allows direct login to servers without needing a password. If an employee accidentally uploaded their private key to a public S3 bucket, an attacker can download it and use it to log into any server that has the corresponding public key installed. This is a very real risk that happens when employees are under pressure, make mistakes, or do not understand the consequences of sharing certain files.

Section 4: Staff — OSINT on Employees
Why Look at Employees?

Employees of a company are a goldmine of information for penetration testers doing OSINT (Open Source Intelligence). By looking at LinkedIn profiles, GitHub accounts, job postings, and professional websites, you can learn what technologies and tools the company uses, what the internal team structure looks like, what programming languages and frameworks developers work with, and even find leaked credentials or sensitive code in public repositories. This information helps you understand the attack surface and plan your approach.

LinkedIn Job Posts

Job posts reveal a lot about internal technology stacks. For example, if a job requires experience with Django, Flask, PostgreSQL, and Kubernetes, you know the company runs Python web applications on a container infrastructure with a PostgreSQL database. This tells you what technologies to look for vulnerabilities in. A job requiring Atlassian Suite experience confirms the earlier TXT record finding that Atlassian tools are used internally.

LinkedIn Employee Profiles

Individual employee profiles show their skills, past projects, and linked GitHub accounts. From a developer's GitHub profile, you might find open source code they wrote that contains company-specific logic, hardcoded API keys or JWT tokens, database connection strings, or even personal email addresses tied to company systems.

Risk: Hardcoded Credentials in Public Code

A very common finding is that developers push code to GitHub that contains hardcoded secrets. For example, a JWT (JSON Web Token) with a hardcoded secret key found in a public GitHub repo could allow an attacker to forge authentication tokens for the company's web application. This is why code review and secrets scanning are so important in security-conscious organizations.

Section 5: FTP (File Transfer Protocol)
What is FTP?

FTP (File Transfer Protocol) is one of the oldest internet protocols, operating at the application layer of the TCP/IP stack alongside HTTP and POP3. It is used to transfer files between a client and a server. FTP opens two separate connections: a control channel on TCP port 21 (for sending commands and receiving status codes) and a data channel on TCP port 20 (for actual file transfer). If a connection is interrupted during a transfer, FTP can resume the transfer after reconnection.

There are two modes of FTP operation. In Active Mode, the server initiates the data connection back to the client, which can be blocked by firewalls. In Passive Mode, the server tells the client which port to connect to, and the client initiates the connection — this works better through firewalls.

FTP normally sends everything in plain text including passwords, making it vulnerable to sniffing on the network. Some servers offer anonymous FTP where users can connect without credentials, though with limited access.

TFTP (Trivial File Transfer Protocol)

TFTP is a simpler version of FTP that uses UDP instead of TCP, making it faster but less reliable. It has no user authentication and no directory listing. It is used only in local, trusted networks because it has no security. Key TFTP commands include connect (sets remote host), get (downloads a file), put (uploads a file), quit (exits TFTP), status (shows current status), and verbose (shows extra transfer information).

Default FTP Configuration (vsFTPd)

vsFTPd is the most commonly used FTP server on Linux. Its configuration file is at /etc/vsftpd.conf.

How to view the config:

bash
cat /etc/vsftpd.conf | grep -v "#"

Important settings to understand:

anonymous_enable=NO — Disables anonymous login (more secure)
local_enable=YES — Allows local system users to log in
write_enable=YES — Allows upload commands like STOR, DELE, MKD
ssl_enable=NO — SSL encryption is disabled by default

The file /etc/ftpusers lists users who are DENIED FTP access even if they exist on the system.

Dangerous FTP Settings

Some settings make FTP servers insecure and attractive for attackers:

anonymous_enable=YES — Anyone can log in without a password
anon_upload_enable=YES — Anonymous users can upload files
anon_mkdir_write_enable=YES — Anonymous users can create directories
no_anon_password=YES — Anonymous users are not even asked for a password
hide_ids=YES — Hides real user/group IDs in directory listings (shows "ftp" instead)
ls_recurse_enable=YES — Allows recursive listing of all directories at once
Performing FTP Tasks
Connecting Anonymously
bash
ftp 10.129.14.136
# When prompted for name:
Name: anonymous

After login, you get a code 230 meaning "Login successful." You can then run ls to list files.

Checking FTP Server Status
ftp
ftp> status

Shows connection type, transfer mode, and other settings.

Enabling Debug and Trace Mode
ftp
ftp> debug
ftp> trace

These commands make the FTP client show all the raw commands being sent to the server, which is useful for understanding how the protocol works.

Recursive Directory Listing
ftp
ftp> ls -R

Lists all files and folders inside all subdirectories at once. This gives a full picture of the FTP server's file structure in one command.

Downloading a File
ftp
ftp> get "Important Notes.txt"

Downloads the specified file from the FTP server to your local machine.

Downloading All Files at Once
bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

This uses wget to mirror the entire FTP server content to your local machine. The -m flag means mirror (recursive download), and --no-passive forces active mode. After download, files are stored in a folder named after the IP address.

Viewing Downloaded Structure
bash
tree .

Shows the downloaded files in a nice tree format.

Uploading a File
bash
touch testupload.txt  # Create a test file

Then inside FTP:

ftp
ftp> put testupload.txt

The put command uploads the file from your local machine to the FTP server. If upload works, it could mean you can plant malicious files on the server.

Footprinting FTP with Nmap
Update NSE Scripts
bash
sudo nmap --script-updatedb

Updates Nmap's script database to ensure the latest NSE scripts are available.

Find FTP-related NSE Scripts
bash
find / -type f -name ftp* 2>/dev/null | grep scripts

This finds all Nmap scripts related to FTP. Important ones include ftp-anon (checks anonymous login), ftp-brute (brute forces credentials), ftp-syst (gets server status), and ftp-vsftpd-backdoor (checks for a known backdoor).

Full Nmap Scan on FTP
bash
sudo nmap -sV -p21 -sC -A 10.129.14.136
-sV — Detects service version
-p21 — Scans only port 21
-sC — Runs default scripts including ftp-anon and ftp-syst
-A — Aggressive mode (OS detection, version detection, scripts, traceroute)
Nmap with Script Trace
bash
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

Shows all network-level communication between Nmap and the FTP server — every command sent and every response received.

Connecting with Netcat or Telnet
bash
nc -nv 10.129.14.136 21
telnet 10.129.14.136 21

These tools let you manually interact with the FTP server at a raw level, sending commands and reading responses directly.

Connecting to FTP with TLS/SSL
bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp

If the FTP server uses TLS/SSL encryption, you need openssl to connect. The -starttls ftp option tells openssl to first connect in plain text and then upgrade to encrypted. This also reveals the SSL certificate, which contains the hostname and organization name.

Section 6: SMB (Server Message Block)
What is SMB?

SMB is a client-server protocol that controls access to files, directories, printers, routers, and other network resources. It was originally developed for Windows networks but is now used across platforms through the Samba project. SMB uses TCP for communication, which means a three-way handshake is established before any data is shared. Access to shared resources is controlled by Access Control Lists (ACLs), and permissions can be set very specifically per user or group. SMB allows multiple users to access shared folders simultaneously, which makes it a critical protocol in enterprise environments.

Samba and CIFS

Samba is an open-source implementation of SMB for Linux and Unix systems. It uses the CIFS (Common Internet File System) protocol, which is a dialect of SMB version 1. Modern Samba supports SMB 2 and SMB 3, which offer better performance, message signing, and encryption. With Samba version 3, a Linux server can join a Windows Active Directory domain, and with version 4, it can even act as a domain controller. Samba uses two key services: smbd for file and print services, and nmbd for NetBIOS name resolution.

SMB Version History
CIFS (SMB 1.0) — Very old, uses NetBIOS, considered insecure
SMB 2.0 — Better performance, improved message signing (Windows Vista)
SMB 2.1 — Added locking mechanisms (Windows 7)
SMB 3.0 — Multichannel, end-to-end encryption (Windows 8)
SMB 3.1.1 — AES-128 encryption, integrity checking (Windows 10)
Default Samba Configuration
bash
cat /etc/samba/smb.conf | grep -v "#\|\;"

The Samba config has a global section (applies to all shares) and individual share sections. Key settings include workgroup (the domain name), server role (standalone or domain member), and map to guest (what to do with failed logins).

Dangerous SMB Settings
browseable = yes — Lets users (and attackers) see what shares exist
read only = no / writable = yes — Allows creating and modifying files
guest ok = yes — No password required to access the share
create mask = 0777 — New files get full read/write/execute permissions for everyone
magic script = script.sh — Executes a script automatically, very dangerous
Performing SMB Tasks
Restart Samba Service
bash
sudo systemctl restart smbd

Apply configuration changes.

List Available Shares (Anonymous)
bash
smbclient -N -L //10.129.14.128
-N — No password (null session / anonymous access)
-L — List all shares
Connect to a Specific Share
bash
smbclient //10.129.14.128/notes

Prompts for password. Press Enter for anonymous access. Inside the SMB session, type help to see all available commands.

Download a File from SMB
smb
smb: \> get prep-prod.txt

Downloads the file to your local machine.

Run Local Commands from SMB Session
smb
smb: \> !ls
smb: \> !cat prep-prod.txt

The ! prefix runs commands on your local machine without leaving the SMB session.

Check SMB Connection Status (Server Side)
bash
smbstatus

Shows who is connected, which shares they are accessing, which protocol version they are using, and whether encryption and signing are enabled.

Footprinting SMB with Nmap
bash
sudo nmap 10.129.14.128 -sV -sC -p139,445

Scans ports 139 (NetBIOS) and 445 (SMB over TCP). Default scripts will check SMB version, message signing, and time.

RPCclient — Manual SMB Enumeration

RPC (Remote Procedure Call) allows executing functions on a remote system. The rpcclient tool lets you run these functions on an SMB server.

Connect with RPCclient
bash
rpcclient -U "" 10.129.14.128

Connect anonymously (null session). If successful, you get the rpcclient $> prompt.

Useful RPCclient Commands
srvinfo              → Server information and OS version
enumdomains          → List all domains
querydominfo         → Domain, server, user count
netshareenumall      → List all shares with paths
netsharegetinfo notes → Detailed info about a specific share
enumdomusers         → List all users and their RIDs
queryuser 0x3e9      → Detailed info about a specific user by RID
querygroup 0x201     → Info about a specific group
Brute Force User RIDs
bash
for i in $(seq 500 1100); do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done

This loop converts numbers 500-1100 to hexadecimal and queries each one as a user RID. Valid users show their username and group information.

Impacket samrdump.py
bash
samrdump.py 10.129.14.128

A Python tool from the Impacket suite that automatically enumerates users from the SMB server's SAM database, showing usernames, UIDs, password last set times, and account status.

SMBmap and CrackMapExec
SMBmap — Check Share Permissions
bash
smbmap -H 10.129.14.128

Lists all shares and shows what permissions you have (READ, WRITE, or NO ACCESS) for each one.

CrackMapExec — SMB Enumeration
bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''

Connects with empty credentials and lists all shares along with permissions. Also shows the Windows version and SMB signing status.

Enum4Linux-ng — Automated SMB Enumeration
Installation
bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt
Run Full Enumeration
bash
./enum4linux-ng.py 10.129.14.128 -A

The -A flag runs all modules. It automatically checks LDAP, SMB, NetBIOS names, OS information, users, groups, shares, policies, and printers. It is much more comprehensive than running individual tools separately.

Section 7: NFS (Network File System)
What is NFS?

NFS (Network File System) was developed by Sun Microsystems and serves a similar purpose to SMB — it allows remote file systems to be mounted and accessed as if they were local. The key difference is that NFS is designed for Linux and Unix systems, while SMB is for Windows. NFS clients cannot directly communicate with SMB servers and vice versa. NFS uses RPC (Remote Procedure Call) as its underlying communication mechanism and traditionally uses ports 111 (RPC portmapper) and 2049 (NFS service).

The three main versions are NFSv2 (older, UDP-based), NFSv3 (variable file size, better error reporting), and NFSv4 (Kerberos support, firewall-friendly, stateful, uses only port 2049). NFSv4 also includes pNFS (parallel NFS for distributed file access) and multipathing support.

NFS Configuration

The /etc/exports file defines which directories are shared and who can access them.

How to view exports:

bash
cat /etc/exports
Adding an NFS Share
bash
echo '/mnt/nfs 10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs

This shares the /mnt/nfs folder to all hosts in the 10.129.14.0/24 subnet.

Key NFS export options:

rw — Read and write access
ro — Read only
sync — Synchronous writes (safer)
async — Asynchronous writes (faster but riskier)
root_squash — Converts root user access to anonymous (safer)
no_root_squash — Keeps root permissions intact (dangerous)
no_subtree_check — Disables subdirectory checking (faster)
nohide — Makes nested mounted filesystems visible
Footprinting NFS with Nmap
Basic Scan
bash
sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Scans the two key NFS ports. The rpcinfo script lists all currently running RPC services.

NFS-Specific Scripts
bash
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Runs all NFS-related NSE scripts. Results include:

nfs-ls — Lists the contents of the NFS share with permissions
nfs-showmount — Shows which paths are exported and which networks can access them
nfs-statfs — Shows disk usage statistics for the NFS share
Performing NFS Tasks
Show Available NFS Shares on Remote Server
bash
showmount -e 10.129.14.128

Lists all NFS exports and which networks/hosts can mount them.

Mount an NFS Share
bash
mkdir target-NFS
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock
cd target-NFS
tree .

Creates a local folder, mounts the remote NFS share into it, and then navigates into it. After mounting, you can browse the files as if they were on your local machine.

List Files with Usernames
bash
ls -l mnt/nfs/

Shows files with their owner usernames and group names.

List Files with UIDs and GIDs
bash
ls -n mnt/nfs/

Shows files with numeric user IDs (UIDs) and group IDs (GIDs). This is useful because you can then create a local user with the same UID to gain access to files that are owned by that UID.

Unmounting the NFS Share
bash
cd ..
sudo umount ./target-NFS

Always unmount the share when done. Navigate out of the mounted folder first, then unmount.

Section 8: DNS (Domain Name System)
What is DNS?

DNS (Domain Name System) is the internet's phonebook. It translates human-readable domain names like hackthebox.com into IP addresses like 10.10.10.1. DNS does not use a central database — instead, information is distributed across thousands of name servers worldwide. There are different types of DNS servers: Root Servers (13 worldwide, manage top-level domains), Authoritative Nameservers (hold official records for a zone), Non-authoritative Nameservers (collect info via recursive queries), Caching Servers (store results temporarily), Forwarding Servers (forward queries to another server), and Resolvers (local to the computer or router).

DNS is mostly unencrypted, making queries visible to ISPs and network devices. Modern solutions like DNS over TLS (DoT) and DNS over HTTPS (DoH) address this privacy concern.

DNS Record Types
A — Maps domain to IPv4 address
AAAA — Maps domain to IPv6 address
MX — Mail server records (which server handles email)
NS — Name server records (which DNS servers manage the domain)
TXT — Text records (verification codes, SPF, DMARC, DKIM)
CNAME — Alias record (one domain points to another)
PTR — Reverse lookup (IP address → domain name)
SOA — Start of Authority (admin contact, serial number, zone info)
DNS Configuration Files (BIND9)

BIND9 is the most common DNS server software on Linux. Its configuration is split across three files:

named.conf.local — Defines zones (which domains this server is authoritative for)
named.conf.options — Global server settings
named.conf.log — Logging configuration
Zone File
bash
cat /etc/bind/db.domain.com

The zone file maps hostnames to IP addresses. It contains SOA record, NS records, MX records, A records, and CNAME records. Every zone file must have exactly one SOA record and at least one NS record.

Reverse Zone File
bash
cat /etc/bind/db.10.129.14

The reverse zone file maps IP addresses back to hostnames using PTR records.

Footprinting DNS
Query NS Records
bash
dig ns inlanefreight.htb @10.129.14.128

Finds the name servers for the domain. The @10.129.14.128 specifies which DNS server to ask.

Query DNS Version
bash
dig CH TXT version.bind 10.129.120.85

Sends a CHAOS class TXT query to get the DNS server's version string (if configured to respond).

Query All Records
bash
dig any inlanefreight.htb @10.129.14.128

Requests all available record types. Not all servers will return everything, but this gives the broadest view possible.

Zone Transfer (AXFR)
bash
dig axfr inlanefreight.htb @10.129.14.128

An AXFR (Asynchronous Full Transfer Zone) is used to replicate entire DNS zones between servers. If a DNS server is misconfigured to allow zone transfers from any host, an attacker can download the complete zone file — including all hostnames, IP addresses, and record types. This reveals the entire internal DNS structure of a company.

Zone Transfer for Internal Zone:

bash
dig axfr internal.inlanefreight.htb @10.129.14.128

Can reveal internal hostnames like domain controllers, VPN servers, workstations, and WSUS update servers.

Subdomain Brute Force
bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done

Goes through a wordlist of common subdomain names and queries the DNS server for each one. If a subdomain resolves, it is saved to subdomains.txt.

DNSenum — Automated DNS Enumeration
bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb

DNSenum automates the entire DNS enumeration process including NS queries, zone transfer attempts, and subdomain brute forcing.

Section 9: SMTP (Simple Mail Transfer Protocol)
What is SMTP?

SMTP is the protocol used to send emails across IP networks. It works between email clients and outgoing mail servers, and between mail servers themselves. By default, SMTP listens on port 25, but newer implementations also use port 587 (for authenticated users with STARTTLS encryption). SMTP alone is text-based and unencrypted, but ESMTP (Extended SMTP) adds support for TLS encryption via the STARTTLS command.

The email flow works like this: The Mail User Agent (MUA, your email client) sends the email to a Mail Submission Agent (MSA), which validates it and passes it to the Mail Transfer Agent (MTA). The MTA looks up the recipient's mail server in DNS (MX record) and delivers the email. At the destination, the Mail Delivery Agent (MDA) puts it in the recipient's mailbox, accessible via IMAP or POP3.

SMTP has two main security weaknesses: no delivery confirmation system (you might not know if an email was delivered), and no built-in sender authentication (making email spoofing easy). Technologies like SPF, DKIM, and DMARC were developed to partially address these issues.

SMTP Commands
HELO / EHLO — Start the session (EHLO for extended SMTP)
MAIL FROM: — Specify the sender email address
RCPT TO: — Specify the recipient email address
DATA — Begin email body input (end with a line containing only .)
VRFY — Verify if a username exists on the server
EXPN — Expand a mailing list or verify a mailbox
RSET — Reset current mail transaction without closing connection
NOOP — Keep connection alive (no operation)
QUIT — Close the SMTP session
AUTH PLAIN — Authenticate with username and password
Performing SMTP Tasks
Connect to SMTP Server
bash
telnet 10.129.14.128 25

After connecting, you see a 220 banner. Then you send HELO or EHLO to start the session.

HELO / EHLO Interaction
HELO mail1.inlanefreight.htb
→ 250 mail1.inlanefreight.htb (server acknowledges)

EHLO mail1
→ 250-PIPELINING
→ 250-SIZE 10240000
→ 250-VRFY
→ (lists all supported extensions)

EHLO reveals all capabilities the server supports.

User Enumeration with VRFY
VRFY root
→ 252 2.0.0 root

VRFY cry0l1t3
→ 252 2.0.0 cry0l1t3

The VRFY command asks if a username exists. Code 252 means the server won't confirm or deny, but many servers respond differently for real vs. fake usernames. This can be used to enumerate valid usernames.

Sending an Email via Telnet
EHLO inlanefreight.htb
MAIL FROM: <cry0l1t3@inlanefreight.htb>
RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
DATA
From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: Test
Date: Tue, 28 Sept 2021 16:32:51 +0200

Message body here.
.
QUIT
Dangerous SMTP Setting — Open Relay

An Open Relay is an SMTP server that allows anyone to send email through it without authentication. This configuration:

mynetworks = 0.0.0.0/0

means the server accepts email from all IP addresses. Attackers can abuse this to send spam or spoofed phishing emails that appear to come from the company's own domain.

Footprinting SMTP with Nmap
Basic SMTP Scan
bash
sudo nmap 10.129.14.128 -sC -sV -p25

The default script smtp-commands runs EHLO and lists all supported commands.

Check for Open Relay
bash
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

Runs 16 different tests to check if the server is an open relay. If any tests pass, the server can be abused for email spoofing.

Section 10: IMAP / POP3
What are IMAP and POP3?

Both IMAP and POP3 are protocols for receiving emails. IMAP (Internet Message Access Protocol) is the modern standard — it allows managing emails directly on the server with folder structures, works across multiple devices simultaneously, and keeps emails on the server until explicitly deleted. POP3 (Post Office Protocol 3) is simpler — it only supports listing, retrieving, and deleting emails, and typically downloads emails to the local device and removes them from the server.

IMAP uses port 143 (plain) and 993 (SSL/TLS). POP3 uses port 110 (plain) and 995 (SSL/TLS). The higher ports (993, 995) use TLS for encrypted communication.

IMAP Commands
1 LOGIN username password — Authenticate
1 LIST "" * — List all folders/directories
1 CREATE "INBOX" — Create a new mailbox
1 DELETE "INBOX" — Delete a mailbox
1 SELECT INBOX — Select a mailbox to access its messages
1 FETCH <ID> all — Retrieve a specific message
1 LOGOUT — Close the IMAP connection
POP3 Commands
USER username — Identify the user
PASS password — Provide the password
STAT — Get number of emails and total size
LIST — List all emails with sizes
RETR id — Download a specific email by ID
DELE id — Delete a specific email
CAPA — Show server capabilities
QUIT — Close the connection
Footprinting IMAP/POP3
Nmap Scan
bash
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

Scans all four mail ports. The SSL certificate in the output reveals the server's hostname, organization name, and location — very useful information.

Connect with cURL (IMAP over SSL)
bash
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

Lists mailbox folders for the authenticated user.

Verbose cURL Connection
bash
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v

Shows the full TLS handshake, SSL certificate details, TLS version, and the complete IMAP authentication process.

Connect with OpenSSL — POP3
bash
openssl s_client -connect 10.129.14.128:pop3s

Connects to the POP3 server over SSL. The output shows certificate chain details, SSL session information, and the server banner.

Connect with OpenSSL — IMAP
bash
openssl s_client -connect 10.129.14.128:imaps

Same concept but for IMAP. After the TLS handshake, you can manually type IMAP commands.

Dangerous Settings
auth_debug — Logs all authentication debug info (passwords might appear in logs)
auth_debug_passwords — Logs submitted passwords in plain text
auth_verbose — Logs failed authentication attempts (reveals valid usernames)
auth_anonymous_username — Allows anonymous login
Section 11: SNMP (Simple Network Management Protocol)
What is SNMP?

SNMP is a protocol designed for monitoring and managing network devices such as routers, switches, servers, printers, and IoT devices. It uses UDP port 161 for normal communication and UDP port 162 for "traps" (unsolicited notifications from devices to the management server). SNMP exists in three versions: SNMPv1 (no authentication, no encryption — very insecure), SNMPv2c (community string-based, still no encryption), and SNMPv3 (username/password authentication, encryption — most secure but complex to configure).

MIB and OID

The MIB (Management Information Base) is a hierarchical text database that describes all the objects (settings, statistics) that can be queried on a device. Each object has a unique OID (Object Identifier) — a sequence of numbers separated by dots (e.g., 1.3.6.1.2.1.1.5.0). By querying OIDs, you can retrieve specific information from a device.

Community Strings

Community strings are like passwords for SNMP v1 and v2. The default public community string (called "public") is used for read access, and "private" for write access. Many administrators never change these defaults, making it easy for attackers to query device information.

Dangerous SNMP Settings
rwuser noauth — Full OID tree access with no authentication
rwcommunity <string> <IP> — Gives write access to the entire OID tree from specified IP
rwcommunity6 <string> <IPv6> — Same but for IPv6
Performing SNMP Tasks
SNMPwalk — Query All OIDs
bash
snmpwalk -v2c -c public 10.129.14.128
-v2c — Use SNMP version 2c
-c public — Use "public" as the community string

The output shows the entire OID tree of the device, including system description, hostname, contact email, location, uptime, installed packages, running processes, and more.

OneSixtyOne — Brute Force Community Strings
bash
sudo apt install onesixtyone
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128

Uses a wordlist to brute force the community string. If a valid one is found, it shows the system description.

Braa — Fast OID Brute Force
bash
sudo apt install braa
braa public@10.129.14.128:.1.3.6.*

Once you have a valid community string, use braa to rapidly query all OIDs matching the pattern. Much faster than snmpwalk for broad enumeration.

Section 12: MySQL
What is MySQL?

MySQL is an open-source relational database management system (RDBMS) developed by Oracle. It stores data in tables with rows and columns and uses SQL (Structured Query Language) for queries. MySQL works on a client-server model — the MySQL server manages the data, and clients send queries to it. It is commonly used with web applications, especially in the LAMP stack (Linux, Apache, MySQL, PHP). Common use cases include storing website content, user credentials, product information, and configuration data. Sensitive data like passwords should be stored as hashed values, not plain text.

Default MySQL Configuration
bash
sudo apt install mysql-server -y
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'

Default port is 3306. The configuration shows the socket path, data directory, log settings, and other parameters.

Dangerous MySQL Settings
user — Which OS user the MySQL service runs as (if set to root, very dangerous)
password — Plain text password in config file
admin_address — IP that admin interface listens on
debug — Enables verbose debug output (may expose sensitive info to web app users)
sql_warnings — Shows detailed warnings (can leak info through error messages)
secure_file_priv — Controls file import/export operations (if empty, no restrictions)
Footprinting MySQL with Nmap
bash
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

The mysql* wildcard runs all MySQL-related scripts including mysql-brute, mysql-empty-password, mysql-enum, and mysql-info. This reveals the MySQL version, authentication plugin, and may even show valid usernames.

Performing MySQL Tasks
Connect to MySQL (No Password)
bash
mysql -u root -h 10.129.14.132

If the server requires a password, you get an "Access denied" error.

Connect with Password
bash
mysql -u root -pP4SSw0rd -h 10.129.14.128

Note: No space between -p and the password.

Show All Databases
sql
show databases;
Select a Database
sql
use mysql;
Show Tables in Selected Database
sql
show tables;
Show All Columns in a Table
sql
show columns from user;
Select All Data from a Table
sql
select * from user;
Search for Specific String
sql
select * from customers where email = "test@example.com";
Check MySQL Version
sql
select version();
View Connected Hosts
sql
use sys;
select host, unique_users from host_summary;
Section 13: MSSQL (Microsoft SQL Server)
What is MSSQL?

MSSQL is Microsoft's proprietary relational database management system. It was built primarily for Windows systems and integrates deeply with the Windows OS and the .NET framework. MSSQL is very popular in enterprise environments especially those running Windows Server infrastructure. It has strong native support for Windows Authentication, meaning domain credentials can be used to authenticate to the database. MSSQL runs on TCP port 1433 by default.

MSSQL Client Tools
SQL Server Management Studio (SSMS) — GUI-based client, often installed on admin machines
Impacket's mssqlclient.py — Command-line Python tool, very useful for penetration testers
Finding mssqlclient.py
bash
locate mssqlclient
Default MSSQL Databases
master — Tracks all system information for the SQL server instance
model — Template for new databases
msdb — Used by SQL Server Agent for scheduling jobs and alerts
tempdb — Stores temporary objects
resource — Read-only database with system objects
Dangerous MSSQL Configurations
Clients connecting without encryption (credentials sent in plain text)
Self-signed certificates (can be spoofed)
Named pipes enabled (alternative connection method, can bypass firewall rules)
Default sa (System Administrator) account with weak or empty password
Footprinting MSSQL with Nmap
bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

This runs many MSSQL-specific scripts. The output reveals the hostname, instance name, SQL Server version, named pipe path, and whether the server is clustered.

Metasploit — MSSQL Ping
use auxiliary/scanner/mssql/mssql_ping
set rhosts 10.129.201.248
run

Queries the MSSQL server for basic information including server name, instance name, version, and TCP/named pipe information.

Connecting with mssqlclient.py
bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

The -windows-auth flag uses Windows domain authentication. After connection, you get an SQL> prompt.

List Databases
sql
SQL> select name from sys.databases
Section 14: Oracle TNS
What is Oracle TNS?

Oracle TNS (Transparent Network Substrate) is a proprietary communication protocol from Oracle used to connect Oracle databases with client applications. It supports multiple network protocols and provides built-in encryption. TNS is widely used in healthcare, finance, and retail industries for large database management. The TNS listener runs on TCP port 1521 by default and manages incoming connection requests, routing them to the correct database instance.

Two key configuration files exist: tnsnames.ora (client-side, maps service names to network addresses) and listener.ora (server-side, defines what the listener process does). Both are typically in $ORACLE_HOME/network/admin/.

Oracle SID

The SID (System Identifier) uniquely identifies a database instance on a server. When connecting to Oracle, you must specify the SID. If the SID is unknown, it must be discovered through enumeration or brute forcing.

Footprinting Oracle TNS
Nmap Basic Scan
bash
sudo nmap -p1521 -sV 10.129.204.235 --open

Checks if the Oracle TNS listener is running and gets its version.

Nmap SID Brute Force
bash
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

Tries common SID names to discover valid database identifiers.

ODAT — Oracle Database Attacking Tool
bash
./odat.py all -s 10.129.204.235

Runs all ODAT modules to enumerate the Oracle server. Can find valid credentials, database names, versions, processes, and vulnerabilities. In examples, it finds credentials like scott/tiger.

Connect with SQLplus
bash
sqlplus scott/tiger@10.129.204.235/XE

Connects to the Oracle database instance XE using the discovered credentials.

List Tables
sql
SQL> select table_name from all_tables;
Check User Privileges
sql
SQL> select * from user_role_privs;
Connect as SYSDBA (Admin)
bash
sqlplus scott/tiger@10.129.204.235/XE as sysdba

If the user has the right permissions, this gives full database administrator access.

Extract Password Hashes
sql
SQL> select name, password from sys.user$;

Retrieves hashed passwords from the system user table, which can be cracked offline.

Upload a File via ODAT
bash
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

Uploads a local file to the web server directory via Oracle's UTL_FILE package, which could lead to remote code execution.

Section 15: IPMI (Intelligent Platform Management Interface)
What is IPMI?

IPMI is a hardware-level management system that allows administrators to manage servers independently of the operating system. Even if a server is powered off, crashed, or unresponsive, IPMI can still be used to reboot it, check hardware status, read temperature and fan speed, view hardware logs, and even reinstall the operating system. IPMI communicates over UDP port 623 and requires a Baseboard Management Controller (BMC) — a small embedded processor directly connected to the server's motherboard. Common BMC implementations are HP iLO, Dell iDRAC, and Supermicro IPMI.

Footprinting IPMI with Nmap
bash
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Uses UDP scan to check for IPMI on port 623. The output shows the IPMI version and authentication methods supported.

Metasploit — IPMI Version Discovery
use auxiliary/scanner/ipmi/ipmi_version
set rhosts 10.129.42.195
run
Default Credentials

Many BMC devices ship with default passwords that administrators never change:

Dell iDRAC: root / calvin
HP iLO: Administrator / random 8-char string (numbers + uppercase)
Supermicro IPMI: ADMIN / ADMIN
RAKP Protocol Vulnerability

A critical flaw in IPMI 2.0 involves the RAKP (Remote Authenticated Key-Exchange Protocol). During authentication, the server sends a salted SHA1 or MD5 hash of the user's password to the client BEFORE authentication is complete. This allows attackers to capture password hashes for any valid user account without completing the login. The hash can then be cracked offline.

Metasploit — Dump IPMI Hashes
use auxiliary/scanner/ipmi/ipmi_dumphashes
set rhosts 10.129.42.195
run

Attempts to retrieve password hashes from the BMC. If the default password wordlist matches, it shows the plaintext password immediately.

Crack with Hashcat (mode 7300):

bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u
Section 16: Linux Remote Management Protocols
SSH (Secure Shell)

SSH enables encrypted remote access to Linux/Unix systems over TCP port 22. It replaced older insecure protocols like Telnet and R-Services. SSH-2 is more secure than SSH-1 and is not vulnerable to Man-in-the-Middle attacks. OpenSSH supports six authentication methods: password authentication, public-key authentication, host-based authentication, keyboard authentication, challenge-response authentication, and GSSAPI authentication.

Public Key Authentication

A user generates a key pair — private key (kept secret on local machine) and public key (stored on the remote server). When connecting, the server sends a cryptographic challenge encrypted with the public key. Only the private key can decrypt and solve it. This proves identity without sending a password over the network.

Default SSH Configuration
bash
cat /etc/ssh/sshd_config | grep -v "#" | sed -r '/^\s*$/d'
Dangerous SSH Settings
PasswordAuthentication yes — Allows brute force attacks
PermitEmptyPasswords yes — Allows login with no password
PermitRootLogin yes — Root can log in directly (very dangerous)
Protocol 1 — Old vulnerable SSH version
X11Forwarding yes — Allows GUI forwarding (had a command injection vulnerability in 2016)
Footprinting SSH with ssh-audit
bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132

Shows SSH version, encryption algorithms, key exchange methods, and flags weak algorithms.

Verbose SSH Connection
bash
ssh -v cry0l1t3@10.129.14.132

Shows authentication methods the server accepts.

Force Password Authentication
bash
ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

Forces the SSH client to try password authentication, useful for brute-force testing.

Rsync

Rsync is a file synchronization tool that uses delta-transfer algorithm — only sending the parts of files that have changed. It uses port 873 and can run over SSH for secure transfers. Rsync is commonly used for backups.

Scan for Rsync
bash
sudo nmap -sV -p 873 127.0.0.1
Probe for Available Shares
bash
nc -nv 127.0.0.1 873

After connecting, type #list to see available shares.

Enumerate an Open Share
bash
rsync -av --list-only rsync://127.0.0.1/dev

Lists all files in the dev share without downloading them. The -av flag means archive mode + verbose.

Download All Files from Share
bash
rsync -av rsync://127.0.0.1/dev ./local-copy/
R-Services (Legacy Remote Access)

R-Services are old Unix remote access protocols that predate SSH. They operate on ports 512 (rexec), 513 (rlogin), and 514 (rsh/rcp). They transmit everything including passwords in plain text and are vulnerable to MITM attacks. Authentication can be bypassed through /etc/hosts.equiv and .rhosts files which list trusted hosts and users.

Scan for R-Services
bash
sudo nmap -sV -p 512,513,514 10.0.17.2
Login with Rlogin
bash
rlogin 10.0.17.2 -l htb-student

If the .rhosts file allows it, you log in without a password.

List Active Users with Rwho
bash
rwho

Shows all logged-in users across the network, including which machine they are on and how long they have been idle.

Detailed User Listing with Rusers
bash
rusers -al 10.0.17.5

Shows detailed information about logged-in users including username, hostname, TTY, login time, idle time, and remote host.

Section 17: Windows Remote Management Protocols
RDP (Remote Desktop Protocol)

RDP is Microsoft's proprietary protocol for graphical remote access to Windows systems. It transmits screen display and keyboard/mouse inputs over TCP port 3389 (and optionally UDP 3389). RDP has supported TLS/SSL encryption since Windows Vista. Network Level Authentication (NLA) adds an extra layer of security by requiring authentication before the full RDP session is established, preventing unauthorized access to the login screen.

Footprinting RDP with Nmap
bash
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*

Shows the OS version, hostname, NLA support status, and NTLM information.

Check RDP Security Settings
bash
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check
./rdp-sec-check.pl 10.129.201.248

Checks which RDP security layers and encryption methods the server supports.

Connect via RDP from Linux
bash
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248

Opens a full graphical RDP session. You must accept the certificate warning (Y) to proceed.

WinRM (Windows Remote Management)

WinRM is a command-line remote management protocol for Windows based on the WS-Management standard. It uses SOAP (Simple Object Access Protocol) for communication and requires explicit enabling on older Windows systems. Ports used are 5985 (HTTP) and 5986 (HTTPS). WinRM is enabled by default on Windows Server 2012 and later.

Footprinting WinRM
bash
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n
Connect with Evil-WinRM (from Linux)
bash
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!

Provides an interactive PowerShell shell on the remote Windows system.

WMI (Windows Management Instrumentation)

WMI is Microsoft's implementation of WBEM (Web-Based Enterprise Management) and provides nearly universal read and write access to Windows system settings. It is accessed via PowerShell, VBScript, or the command-line tool wmic. WMI communication starts on TCP port 135, then moves to a random port.

Connect with wmiexec.py (Impacket)
bash
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"

Executes a command on the remote Windows system via WMI and returns the output. In this case, it runs hostname and returns the computer name.

These notes cover all sections completely with every command explained — its purpose, usage, and how to perform each task.

FTP (File Transfer Protocol) — Complete Detailed Notes
What is FTP?

FTP (File Transfer Protocol) is one of the oldest protocols on the internet. It was created long before modern security concerns and has been in use for decades for transferring files between computers. FTP operates at the application layer of the TCP/IP protocol stack, sitting at the same layer as HTTP and POP3. This means it works on top of the basic internet communication rules and provides a specific service — file transfer. There are special FTP programs and clients built specifically for FTP, in addition to built-in support in many operating systems. FTP is widely used by web developers, system administrators, and businesses to upload files to servers, share data, and move large files between systems over a network.

How FTP Works — Two Channels

FTP is unique because it uses two separate TCP connections to function, not just one like most protocols.

Channel 1 — Control Channel (TCP Port 21):
This is where all the commands and responses happen. When you connect to an FTP server, this channel is opened first. The client sends commands like LIST, GET, PUT, and the server responds with status codes. This channel stays open for the entire FTP session.

Channel 2 — Data Channel (TCP Port 20):
This is where the actual file data flows. Every time you list a directory or transfer a file, a separate data connection is opened for that specific operation and then closed afterward. This separation of control and data is a key design feature of FTP.

If a connection is interrupted during a file transfer, FTP can resume the transfer after reconnecting — this is another important feature that makes FTP practical for large file transfers.

Active Mode vs Passive Mode

Understanding FTP modes is essential because firewalls affect how FTP works.

Active Mode

In Active Mode, the client connects to the server on port 21 (control channel) and tells the server which port the client is using for data. The server then initiates the data connection back to the client on port 20. The problem here is that if the client is behind a firewall, the firewall may block incoming connections from the server, breaking the data transfer.

Passive Mode

In Passive Mode, after the client connects on port 21, the server announces an available port number. The client then initiates the data connection to that port on the server. Because the client is always the one starting connections, firewalls protecting the client do not block anything. This is why passive mode is the standard for internet FTP connections today.

FTP Authentication

FTP normally requires a username and password to log in. However, many servers are configured to allow anonymous FTP where anyone can log in using "anonymous" as the username (and sometimes any email address as a password). Anonymous FTP is used to share public files but carries security risks because anyone can access those files.

Important security concern: FTP sends all data including usernames and passwords in plain text (unencrypted). If someone is monitoring the network, they can see everything. This is why SFTP (SSH File Transfer Protocol) and FTPS (FTP over SSL) were developed as secure alternatives.

TFTP (Trivial File Transfer Protocol)
What is TFTP?

TFTP is a much simpler version of FTP. It was designed for situations where simplicity is more important than features or security. While FTP uses TCP and provides reliable, ordered delivery, TFTP uses UDP, making it faster but unreliable. TFTP handles data loss through application-layer recovery, meaning it has its own simple mechanism for dealing with lost packets.

TFTP has no authentication at all — no usernames, no passwords. It relies entirely on the file system's read and write permissions to control access. Because of this total lack of security, TFTP should only be used on local, protected, trusted networks — never on the internet. It is commonly used for booting network devices, loading firmware, and distributing configuration files in controlled environments.

TFTP Commands

Each TFTP command has a specific function. Here is a complete breakdown:

connect — Sets the remote host and optionally the port number for file transfers. You must run this before any transfer commands. Example: connect 10.129.14.136

get — Transfers one or more files FROM the remote host TO your local machine. This is used to download files from the TFTP server. Example: get config.txt

put — Transfers one or more files FROM your local machine TO the remote host. This is used to upload files to the TFTP server. Example: put update.bin

quit — Exits the TFTP program and closes the connection cleanly.

status — Shows the current status of the TFTP session, including the transfer mode (ASCII or binary), whether a connection is active, the timeout value, and other session details.

verbose — Turns verbose mode on or off. When verbose is on, extra information is displayed during file transfers including packet counts and timing — useful for debugging.

Important note: Unlike FTP, TFTP does NOT have a directory listing command. You cannot see what files exist on the server — you must already know the exact filename you want to download.

vsFTPd — Default FTP Server on Linux
What is vsFTPd?

vsFTPd (Very Secure FTP Daemon) is the most commonly used FTP server software on Linux-based systems. The name "Very Secure" refers to its design philosophy of security-first. It is the default FTP server on many Linux distributions including Ubuntu and CentOS. Understanding vsFTPd is important because it is what you will encounter most often during penetration tests on Linux systems.

Installing vsFTPd

Command:

bash
sudo apt install vsftpd

Purpose: Installs the vsFTPd FTP server package on your Linux system. The sudo is needed because installing software requires administrator privileges. After installation, the service starts automatically and the configuration file is created at /etc/vsftpd.conf.

Viewing the vsFTPd Configuration File

Command:

bash
cat /etc/vsftpd.conf | grep -v "#"

Purpose: Displays the vsFTPd configuration file, filtering out all comment lines (lines starting with #) so you only see the active settings. The grep -v "#" part means "show everything EXCEPT lines containing #". This gives you a clean view of what settings are actually active.

Key configuration settings and what they mean:

listen=NO — This controls whether vsFTPd runs as a standalone service (YES) or is managed by the inetd/xinetd super-server (NO). Standalone is more common in modern setups.

listen_ipv6=YES — Enables IPv6 listening in addition to IPv4. This allows clients with IPv6 addresses to connect.

anonymous_enable=NO — Controls whether anonymous login is allowed. When set to NO, only authenticated users with valid credentials can log in. Setting this to YES is a dangerous configuration that allows anyone to connect.

local_enable=YES — Allows local Linux system users (users with accounts on the server) to log in to FTP with their system credentials.

dirmessage_enable=YES — When a user enters a directory that has a .message file, the contents of that file are displayed. This can be used for directory-specific messages or warnings.

use_localtime=YES — Uses the server's local timezone when displaying file timestamps in directory listings instead of UTC.

xferlog_enable=YES — Enables logging of all file uploads and downloads. This creates an audit trail of all file transfer activity. Important for security monitoring.

connect_from_port_20=YES — In Active Mode, the server initiates data connections from port 20. This setting enforces that behavior.

secure_chroot_dir=/var/run/vsftpd/empty — Points to an empty directory used for security purposes when the server needs to operate in a chrooted (isolated) environment without filesystem access.

pam_service_name=vsftpd — Specifies the PAM (Pluggable Authentication Module) service configuration file that vsFTPd will use for authentication. PAM is the Linux authentication framework.

rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem — The path to the SSL certificate file used when SSL encryption is enabled.

rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key — The path to the private key that pairs with the SSL certificate.

ssl_enable=NO — Controls whether SSL/TLS encryption is used for FTP connections. When NO, all data including passwords travels in plain text.

The /etc/ftpusers File
What is /etc/ftpusers?

This file is a deny list — it contains usernames of system users who are specifically prohibited from using FTP, even if they have valid system accounts and correct passwords. This is a security feature to prevent certain powerful or sensitive system accounts from accessing the FTP service.

Command to view it:

bash
cat /etc/ftpusers

Example output:

guest
john
kevin

This means even if "guest", "john", and "kevin" exist on the Linux system and know their passwords, they cannot log in through FTP. This is typically used to block system accounts (like root, daemon, nobody) from FTP access as a security measure.

Dangerous FTP Settings

These settings, when enabled, create security vulnerabilities that penetration testers specifically look for. Each one opens a potential attack path.

Anonymous Login Settings

anonymous_enable=YES — Allows anyone to log in without any credentials at all. Just use "anonymous" as the username. This is the most common dangerous setting and is often left enabled by accident.

anon_upload_enable=YES — Allows anonymous users to upload files. This is extremely dangerous because attackers can upload malicious files, web shells, or use the FTP space for illegal content.

anon_mkdir_write_enable=YES — Allows anonymous users to create new directories. Combined with upload, an attacker can organize uploaded malicious content in their own folder structure.

no_anon_password=YES — Anonymous users are not even asked to enter a password. Normally anonymous FTP asks for an email address as a courtesy password. This setting skips even that minimal step.

anon_root=/home/username/ftp — Specifies the root directory for anonymous users. This restricts anonymous users to a specific folder so they cannot browse the entire filesystem.

write_enable=YES — Enables the FTP write commands: STOR (upload), DELE (delete), RNFR/RNTO (rename), MKD (make directory), RMD (remove directory), APPE (append to file), and SITE (site-specific commands). Without this, users can only read and download.

Security-Impacting Display Settings

hide_ids=YES — This setting replaces the actual numeric user IDs and group IDs in directory listings with the generic string "ftp". Normally when you list files, you see who owns them. With hide_ids=YES, everything shows as owned by "ftp". This prevents attackers from learning real usernames on the system. However, it also makes it harder for legitimate administrators to audit file ownership.

ls_recurse_enable=YES — Allows the use of recursive directory listing (ls -R). This lets users see the entire directory tree at once with a single command. For a penetration tester, this is extremely useful because one command reveals the entire FTP file structure. For a server, this can be a privacy/security concern if sensitive directories are accidentally exposed.

Connecting to FTP — Anonymous Login
How to Perform Anonymous Login

Command:

bash
ftp 10.129.14.136

What happens step by step:

The FTP client attempts to connect to the server at the given IP on port 21.
The server responds with a 220 banner — this is the welcome message that often includes the FTP software name and version.
You are prompted for a username. Type anonymous and press Enter.
You may be prompted for a password. For anonymous login, type anything or just press Enter.
If anonymous login is allowed, you receive code 230 meaning "Login successful."
The server tells you the remote system type (usually UNIX) and transfer mode (binary or ASCII).

Full example interaction:

ftp 10.129.14.136

Connected to 10.129.14.136.
220 "Welcome to the HTB Academy vsFTP service."
Name (10.129.14.136:cry0l1t3): anonymous

230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.

Important note: The banner (code 220) often reveals the FTP software name and version. This is very valuable information for a penetration tester because it tells you exactly what software is running, allowing you to search for known vulnerabilities in that specific version.

FTP Commands Inside a Session

Once logged in, you use FTP commands to interact with the server. Here is every important command:

ls — List Directory Contents

Command:

ftp
ftp> ls

Purpose: Lists all files and directories in the current directory on the FTP server. Shows file permissions, owner IDs, file sizes, dates, and names. The status codes shown are 200 (PORT command success) and 150 (about to send data) and 226 (transfer complete).

Example output:

200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-rw-r--    1 1002     1002      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Clients
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Documents
drwxrwxr-x    2 1002     1002         4096 Sep 14 16:50 Employees
-rw-rw-r--    1 1002     1002           41 Sep 14 16:45 Important Notes.txt
226 Directory send OK.

The first column shows file permissions (d means directory, - means file, then rwx for read/write/execute for owner/group/others). Second and third columns show owner UID and GID. Fourth is file size. Then date and filename.

status — Check Current FTP Session Settings

Command:

ftp
ftp> status

Purpose: Displays the current configuration of your FTP session. This is not about the server — it is about how your FTP client is currently set up. It tells you what mode you are in, what your transfer settings are, and what optional features are enabled or disabled.

Example output:

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

This tells you the transfer type is binary (not ASCII), PORT commands are used (Active Mode), and verbose mode is on (you see server responses).

debug — Enable Debug Mode

Command:

ftp
ftp> debug

Purpose: Turns on debugging in the FTP client. When debug mode is on, the client prints every command it sends to the server before sending it. This lets you see the raw FTP protocol communication happening behind the scenes. It is very useful for understanding exactly how FTP works and for troubleshooting connection issues.

After enabling:

Debugging on (debug=1).

Now every command you type will show the raw FTP protocol command being sent, like ---> PORT 10,10,14,4,188,195.

trace — Enable Packet Tracing

Command:

ftp
ftp> trace

Purpose: Turns on packet-level tracing. This is even more detailed than debug mode. With trace enabled, the FTP client shows every network packet being sent and received. This is mainly used for deep protocol analysis and troubleshooting at the network level.

After enabling:

Packet tracing on.
Using debug and trace together — Detailed Output Example

When both debug and trace are enabled, an ls command shows:

ftp
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

Now you can see the raw PORT command being sent (which tells the server where to send the data) and the LIST command that requests the directory listing. This is exactly how the FTP protocol works under the hood.

ls -R — Recursive Directory Listing

Command:

ftp
ftp> ls -R

Purpose: Lists all files and directories recursively — meaning it goes into every subfolder and lists everything inside them too, all in one command. This requires ls_recurse_enable=YES to be set on the server. For a penetration tester, this is a very powerful command because you can see the ENTIRE file structure of the FTP server with one command without having to manually navigate each directory.

Example output:

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

Notice that when hide_ids=YES is set, all owner IDs show as "ftp" instead of real usernames. Also notice that the ./Employees directory is empty — there are no files inside it.

get — Download a Single File

Command:

ftp
ftp> get "Important Notes.txt"

Purpose: Downloads a specific file from the FTP server to your local machine. The file is saved in whatever directory you were in on your local machine when you launched FTP. If the filename has spaces, put it in quotes. When the download completes, FTP shows how many bytes were received and the transfer speed.

Full example:

ftp
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

After exiting FTP, verify the file was downloaded:

bash
ls | grep Notes.txt
'Important Notes.txt'

Note the backslash \ before the space in the filename when not using quotes — this "escapes" the space so the shell doesn't treat it as two separate arguments.

Downloading All Files at Once with wget

Command:

bash
wget -m --no-passive ftp://anonymous:anonymous@10.129.14.136

Purpose: This downloads the entire FTP server content in one command using wget instead of the interactive FTP client. This is very useful when the FTP server has many files spread across many directories and you want everything at once.

Breaking down the command:

wget — The download tool
-m — Mirror mode, recursively downloads everything, preserving directory structure
--no-passive — Forces Active Mode (uses PORT commands instead of PASV)
ftp://anonymous:anonymous@10.129.14.136 — The URL format: protocol://username:password@server

What happens: wget connects to the FTP server, logs in as anonymous, and recursively downloads every file it can access. A directory named after the server's IP address is created, and all files are stored inside it preserving the original folder structure.

Example output:

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

Security warning: Downloading ALL files at once may trigger alerts on the target system because no legitimate user normally downloads everything simultaneously. Use this technique carefully during authorized penetration tests.

Viewing Downloaded Structure with tree

Command:

bash
tree .

Purpose: Displays all downloaded files in a visual tree format. This makes it easy to understand the directory structure at a glance.

Example output:

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
put — Upload a File to FTP Server
Why Uploading Matters in Penetration Testing

If you can upload files to an FTP server that is connected to a web server, you can potentially upload a web shell — a script that lets you run system commands through the browser. This is a common path to gaining Remote Command Execution (RCE). Even FTP logs can be abused — if a web server logs FTP activity and you can inject commands into your FTP username or input, those commands might execute when the log file is processed.

Step 1 — Create a Test File

Command:

bash
touch testupload.txt

Purpose: Creates an empty test file named testupload.txt on your local machine. The touch command creates a new empty file if it does not exist, or updates the timestamp if it does. This is used as a safe, harmless test to check whether the FTP server allows uploads.

Step 2 — Upload the File

Command (inside FTP session):

ftp
ftp> put testupload.txt

Purpose: Uploads the testupload.txt file from your local machine to the current directory on the FTP server. The FTP client sends the STOR command to the server, the server opens a data channel, and the file bytes are transferred.

Full example:

ftp
ftp> put testupload.txt

local: testupload.txt remote: testupload.txt
---> PORT 10,10,14,4,184,33
200 PORT command successful. Consider using PASV.
---> STOR testupload.txt
150 Ok to send data.
226 Transfer complete.

After the upload, verify it appears in the directory listing:

ftp
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

The testupload.txt now appears in the listing with size 0 (empty file). This confirms upload is possible — a major finding in a penetration test.

hide_ids Setting — Impact on Directory Listing
What hide_ids=YES Does

When this setting is active, the FTP server replaces all real user and group ownership information in directory listings with the generic string "ftp". Normally, directory listings show the actual UID and GID numbers of file owners. With hide_ids enabled, all files appear to be owned by "ftp".

Why administrators use it: To prevent attackers from learning real system usernames through FTP directory listings, because those usernames could then be used in brute force attacks against SSH, FTP, or other services.

Example listing WITH hide_ids=YES:

-rw-rw-r--    1 ftp     ftp      8138592 Sep 14 16:54 Calender.pptx
drwxrwxr-x    2 ftp     ftp         4096 Sep 14 17:03 Clients
-rw-rw-r--    1 ftp     ftp           41 Sep 14 16:45 Important Notes.txt
-rw-------    1 ftp     ftp            0 Sep 15 14:57 testupload.txt

Notice everything shows "ftp ftp" as owner and group instead of real values like "1002 1002".

Counter-defense note: Modern systems use fail2ban to detect and block brute force attempts. If too many failed login attempts come from one IP, that IP gets automatically blocked. So even if you discover real usernames through FTP, brute forcing those accounts from the same IP would quickly get blocked.

Footprinting FTP with Nmap
What is Nmap NSE?

Nmap includes the Nmap Scripting Engine (NSE) — a collection of scripts written in Lua that can perform specific enumeration, detection, and even exploitation tasks against target services. For FTP, there are several NSE scripts that automatically check for common vulnerabilities and misconfigurations.

Updating the NSE Script Database

Command:

bash
sudo nmap --script-updatedb

Purpose: Updates the local NSE script database so Nmap knows about all available scripts. You should run this periodically to get the latest scripts. The output confirms when the update is successful.

Example output:

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 13:49 CEST
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.28 seconds
Finding FTP-Related NSE Scripts

Command:

bash
find / -type f -name ftp* 2>/dev/null | grep scripts

Purpose: Searches the entire filesystem for files whose names start with "ftp" and filters results to show only those in a "scripts" directory. The 2>/dev/null redirects error messages (like "permission denied") to null so they don't clutter the output.

Example output:

/usr/share/nmap/scripts/ftp-syst.nse
/usr/share/nmap/scripts/ftp-vsftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-vuln-cve2010-4221.nse
/usr/share/nmap/scripts/ftp-proftpd-backdoor.nse
/usr/share/nmap/scripts/ftp-bounce.nse
/usr/share/nmap/scripts/ftp-libopie.nse
/usr/share/nmap/scripts/ftp-anon.nse
/usr/share/nmap/scripts/ftp-brute.nse

What each script does:

ftp-anon.nse — Checks if the FTP server allows anonymous login. If it does, the script logs in anonymously and lists the root FTP directory contents. This tells you immediately what files are accessible without credentials.

ftp-syst.nse — Sends the STAT command to the FTP server to get detailed status information. This reveals the FTP server type, version, session settings, connection count, and timeout values.

ftp-brute.nse — Attempts to brute force FTP credentials using a username and password list. This is used when you know the service requires authentication.

ftp-vsftpd-backdoor.nse — Checks for a famous backdoor that was planted in vsFTPd version 2.3.4 in 2011. If this backdoor exists, connecting to port 6200 gives a root shell.

ftp-proftpd-backdoor.nse — Similar to above but for ProFTPd server backdoors.

ftp-bounce.nse — Tests if the FTP server is vulnerable to FTP bounce attacks, where you use the server as a proxy to scan other machines.

ftp-vuln-cve2010-4221.nse — Checks for a specific buffer overflow vulnerability in ProFTPd.

ftp-libopie.nse — Tests for OTP (One-Time Password) issues in FTP implementations.

Full Nmap FTP Scan

Command:

bash
sudo nmap -sV -p21 -sC -A 10.129.14.136

Breaking down every flag:

sudo — Run as root (needed for certain scan types)
nmap — The tool name
-sV — Service version detection. Nmap probes the service to determine the exact software and version running
-p21 — Only scan port 21 (FTP port). Without this flag, Nmap scans many ports which takes longer
-sC — Run default scripts. This automatically runs all scripts in Nmap's "default" category for any service detected
-A — Aggressive scan mode. Enables OS detection, version detection, script scanning, and traceroute all together
10.129.14.136 — The target IP address

Example output:

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

What this output tells you:

The ftp-anon script confirmed anonymous FTP login is allowed. It logged in and listed the root directory. Every file shows [NSE: writeable] meaning they can be modified — a serious finding. The ftp-syst script ran the STAT command and learned the exact vsFTPd version (3.0.3), that all connections are plain text (no encryption), the session timeout is 300 seconds, and there are 2 current client connections.

Nmap Script Trace

Command:

bash
sudo nmap -sV -p21 -sC -A 10.129.14.136 --script-trace

Purpose: Adds --script-trace to the normal scan. This makes Nmap show EVERY network interaction that happens during the NSE script execution. You can see all the raw data being sent and received at the network level. This is invaluable for understanding exactly how Nmap's scripts communicate with the FTP server.

Example output showing key parts:

NSOCK INFO [11.4640s] nsock_connect_tcp(): TCP connection requested to 10.129.14.136:21
NSOCK INFO [11.4640s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8
NSOCK INFO [11.4660s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 50
(41 bytes): 220 Welcome to HTB-Academy FTP service...

NSE: TCP 10.10.14.4:54228 < 10.129.14.136:21 | 220 Welcome to HTB-Academy FTP service.

You can see the exact timestamp of each event, the exact bytes received, and the raw server banner. Notice that Nmap opens multiple parallel connections (EID 8, EID 16, EID 24, EID 32) using different local ports (54226, 54228, 54230, 54232) to run multiple scripts simultaneously.

Service Interaction — Direct Connection to FTP
Connecting with Netcat

Command:

bash
nc -nv 10.129.14.136 21

Purpose: Opens a raw TCP connection to the FTP server on port 21 using netcat. This bypasses all FTP client features and lets you communicate with the server directly using raw text commands.

nc — netcat, the network Swiss Army knife
-n — Do not resolve hostnames (use IP directly, faster)
-v — Verbose mode (shows connection status messages)
10.129.14.136 — Target IP
21 — Target port

After connecting, you see the server banner and can type FTP commands manually. This is useful when you want minimal footprint and maximum control over exactly what gets sent.

Connecting with Telnet

Command:

bash
telnet 10.129.14.136 21

Purpose: Similar to netcat — opens a plain text connection to the FTP server. Telnet was originally used for terminal access but can connect to any TCP service. It gives you direct text-based interaction with the FTP server protocol.

Both netcat and telnet are useful when you want to manually test specific FTP commands, check what the server responds to, or communicate with the server in a way that standard FTP clients might not allow.

FTP with TLS/SSL Encryption — Using OpenSSL
When is This Needed?

Some FTP servers are configured with TLS/SSL encryption (called FTPS). Standard FTP clients and netcat/telnet cannot handle encrypted connections. You need a tool that understands SSL/TLS to connect. OpenSSL's s_client tool can do this.

Command:

bash
openssl s_client -connect 10.129.14.136:21 -starttls ftp

Breaking down the command:

openssl — The OpenSSL toolkit
s_client — The SSL/TLS client component of OpenSSL
-connect 10.129.14.136:21 — Connect to this IP and port
-starttls ftp — This is very important. It tells openssl to first connect in plain text, then send the FTP STARTTLS command to upgrade to an encrypted connection. This is how FTPS (explicit mode) works.

What the output reveals:

CONNECTED(00000003)
Can't use SSL_get_servername
depth=0 C = US, ST = California, L = Sacramento, O = Inlanefreight, OU = Dev,
CN = master.inlanefreight.htb, emailAddress = admin@inlanefreight.htb
verify error:num=18:self signed certificate
verify return:1

The SSL certificate in the output tells you extremely valuable information:

C = US, ST = California, L = Sacramento — Country, state, and city of the organization
O = Inlanefreight — Organization name
OU = Dev — Organizational unit (department) — this is the Dev team's certificate
CN = master.inlanefreight.htb — The Common Name, which is the hostname of the server
emailAddress = admin@inlanefreight.htb — The admin's email address

The error "self signed certificate" means the certificate was not issued by a trusted Certificate Authority but was created by the organization itself. This is common in internal environments. It does not prevent the connection — it just means you cannot automatically verify the server's identity.

Why this is useful for penetration testing: Even without breaking the encryption, the certificate itself leaks the server's hostname, the organization name, department, admin email, and in some cases even geographic location. If a company has offices in multiple locations, different servers might have certificates listing different cities. This is all valuable intelligence gathered passively.

These are the complete, detailed notes for the FTP section — every heading, subheading, command, its purpose, usage, and complete demonstration — nothing skipped.

create same notes exacatly like abvoe fro remaing services

Complete Detailed Notes — All Remaining Services
SMB (Server Message Block)
What is SMB?

SMB (Server Message Block) is a client-server network protocol that controls and manages access to shared resources across a network. These shared resources include files, entire directories, printers, routers, and other network interfaces. SMB was originally developed for Windows operating systems and first became widely available as part of the OS/2 network operating system LAN Manager and LAN Server. Beyond file sharing, SMB also handles communication between different system processes, meaning applications on different machines can exchange data using SMB. Today SMB is used in almost every Windows-based network environment, and because of backward compatibility, newer Windows versions can still communicate with very old Windows systems over SMB. This makes SMB both very widespread and a major target for penetration testers.

How SMB Works

SMB works on the client-server model. The client sends a request to access a resource, and the server processes that request and either grants or denies access. Before any data is exchanged, both parties must establish a connection. In IP networks, SMB uses TCP protocol for this, which means a proper three-way handshake (SYN, SYN-ACK, ACK) happens before the session begins. After the connection is established, TCP governs all data transport. The SMB server can share parts of its local file system as "shares" — these shares appear to the client as accessible network folders. Access rights to these shares are controlled by ACLs (Access Control Lists), which can be set very specifically per user or per group with granular permissions like read-only, write, or full access.

Samba — SMB for Linux

Samba is the open-source Linux/Unix implementation of the SMB protocol. It allows Linux and Unix systems to participate in Windows networks and share files with Windows machines. Samba implements CIFS (Common Internet File System), which is a specific dialect of SMB version 1 originally created by Microsoft. Modern Samba supports SMB 2 and SMB 3 as well. Samba uses two important background services: smbd (SMB server daemon) which provides file and print services, and nmbd (NetBIOS Message Block Daemon) which handles NetBIOS name resolution. When Samba communicates with older NetBIOS services, it uses TCP ports 137, 138, and 139. Modern CIFS and SMB use TCP port 445 exclusively.

SMB Version History

Understanding SMB versions is important because older versions have serious known vulnerabilities (like EternalBlue which targeted SMB 1.0):

CIFS (Windows NT 4.0) — The very first version. Communication happens through the NetBIOS interface. Considered completely outdated and insecure today.

SMB 1.0 (Windows 2000) — Introduced direct TCP connection without NetBIOS. Still has major security flaws. The EternalBlue exploit (used in WannaCry ransomware) targeted this version.

SMB 2.0 (Windows Vista, Server 2008) — Major improvements including better performance, improved message signing to detect tampering, and caching features to speed up repeated requests.

SMB 2.1 (Windows 7, Server 2008 R2) — Added file locking mechanisms to prevent conflicts when multiple users access the same file simultaneously.

SMB 3.0 (Windows 8, Server 2012) — Added multichannel connections (multiple network paths for resilience), end-to-end encryption, and remote storage access.

SMB 3.0.2 (Windows 8.1, Server 2012 R2) — Minor updates and bug fixes.

SMB 3.1.1 (Windows 10, Server 2016) — Added integrity checking before authentication and AES-128 encryption for maximum security.

With Samba version 3, the Linux Samba server could join a Windows Active Directory domain as a member. With version 4, Samba can even act as a full Active Directory domain controller.

Default Samba Configuration
How to View Samba Configuration

Command:

bash
cat /etc/samba/smb.conf | grep -v "#\|\;"

Purpose: Reads the main Samba configuration file and filters out all comment lines. The grep -v "#\|\;" part means "exclude lines containing # OR ;". Both characters are used for comments in smb.conf. This gives you only the active configuration lines.

Example output:

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

Understanding the configuration structure:

The [global] section applies settings to all shares. workgroup sets the Windows workgroup or domain name. server string is what appears when clients browse the network. server role = standalone server means this Samba server is not a domain member — it operates independently. map to guest = bad user means failed logins are treated as guest access (this can be dangerous). usershare allow guests = yes allows guest access to user-created shares.

The [printers] and [print$] sections are default shares for printer sharing that are always present.

Key SMB Share Settings and What They Mean

[sharename] — The name in square brackets defines the share name. This is what clients see when they browse the network.

workgroup = WORKGROUP/DOMAIN — Defines which Windows workgroup or domain this server belongs to. Clients in the same workgroup can discover this server automatically.

path = /path/here/ — The actual directory on the Linux filesystem that this share maps to. Clients will see the contents of this directory.

server string = STRING — A description of the server that appears to clients when they connect.

unix password sync = yes — Synchronizes the Unix/Linux password with the SMB password so users only need one password.

usershare allow guests = yes — Allows non-authenticated (guest) users to access shares that are configured for guest access.

map to guest = bad user — When a login fails because the username does not exist, the connection is treated as a guest instead of being rejected. This can allow unauthorized access if guest access is also enabled.

browseable = yes — Makes the share visible in network browsing lists. If set to no, the share is hidden and can only be accessed if you know its exact name.

guest ok = yes — Allows connection to this specific share without any password. Anyone can access it anonymously.

read only = yes — Users can only read files. They cannot create, modify, or delete anything. Set to no to allow modifications.

create mask = 0700 — When new files are created through SMB, they get these Unix permissions. 0700 means only the owner can read, write, execute — no access for group or others.

Dangerous SMB Settings

These settings are the ones penetration testers specifically look for because they create vulnerabilities:

browseable = yes — Makes shares visible to everyone scanning the network. An attacker can see what shares exist without any authentication.

read only = no — Combined with guest access, this allows anonymous users to upload files, modify data, or potentially replace legitimate files with malicious ones.

writable = yes — Same as read only = no but explicitly enables write access.

guest ok = yes — No password required. This is the single most dangerous SMB setting because it allows complete unauthenticated access to the share.

enable privileges = yes — Honors Windows-style SID (Security Identifier) privileges, which can allow privilege escalation within the SMB context.

create mask = 0777 — New files created through SMB get full permissions for everyone — any user on the Linux system can read, write, and execute them.

directory mask = 0777 — New directories created through SMB get full permissions for everyone.

logon script = script.sh — Executes a shell script every time a user logs on. If an attacker controls this script, they can execute arbitrary commands on every user login.

magic script = script.sh — Executes when a specific script file is closed. This is a very dangerous setting that can lead to automatic command execution.

magic output = script.out — Specifies where the output of the magic script is stored.

Creating an Example Share for Testing

Configuration to add to /etc/samba/smb.conf:

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

This creates a share called "notes" that maps to /mnt/notes/, is visible in network browsing, allows read and write access, allows guests without passwords, and gives full permissions to all created files. This is an extremely insecure configuration used here for demonstration purposes.

Restarting Samba to Apply Configuration Changes

Command:

bash
sudo systemctl restart smbd

Purpose: After modifying /etc/samba/smb.conf, you must restart the Samba service for changes to take effect. systemctl is the service management tool on modern Linux systems. restart stops the service and then starts it again. Without restarting, the old configuration remains active.

SMBclient — Listing Available Shares

Command:

bash
smbclient -N -L //10.129.14.128

Purpose: Lists all shared folders available on the target Samba server without using any credentials.

Breaking down the flags:

smbclient — The SMB client tool
-N — Null session, meaning no password is used. This is anonymous access
-L — List mode. Shows all available shares instead of connecting to one
//10.129.14.128 — The target server in UNC path format (double slash + IP)

Example output:

        Sharename       Type      Comment
        ---------       ----      -------
        print$          Disk      Printer Drivers
        home            Disk      INFREIGHT Samba
        dev             Disk      DEVenv
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled -- no workgroup available

You can see five shares. print$ and IPC$ are default shares always present. home, dev, and notes are custom shares. The Type column shows "Disk" for file shares and "IPC" for inter-process communication. The Comment column shows the description set in smb.conf.

SMBclient — Connecting to a Specific Share

Command:

bash
smbclient //10.129.14.128/notes

Purpose: Connects to the specific "notes" share on the target server. You are prompted for a password — pressing Enter attempts anonymous/guest access.

Full interaction example:

Enter WORKGROUP\<username>'s password:
Anonymous login successful
Try "help" to get a list of possible commands.

smb: \>

The smb: \> prompt means you are inside the share and can now navigate and interact with its contents.

help — List Available SMB Commands

Command:

smb: \> help

Purpose: Displays all commands available inside the SMB session. This is very useful when you are unfamiliar with SMBclient commands.

Example output:

?              allinfo        altname        archive        backup
blocksize      cancel         case_sensitive cd             chmod
chown          close          del            deltree        dir
du             echo           exit           get            getfacl
geteas         hardlink       help           history        iosize
lcd            link           lock           lowercase      ls
l              mask           md             mget           mkdir
more           mput           newer          notify         open
posix          posix_encrypt  posix_open     posix_mkdir    posix_rmdir
posix_unlink   posix_whoami   print          prompt         put
pwd            q              queue          quit           readlink
rd             recurse        reget          rename         reput
rm             rmdir          showacls       setea          setmode
scopy          stat           symlink        tar            tarmode
timeout        translate      unlock         volume         vuid
wdel           logon          listconnect    showconnect    tcon
tdis           tid            utimes         logoff         ..
!

Key commands: ls (list files), get (download file), put (upload file), cd (change directory), mkdir (make directory), rm (delete file), ! (run local command).

ls — List Files in SMB Share

Command:

smb: \> ls

Purpose: Lists all files and directories in the current SMB share directory. Shows permissions, size, and last modified date.

Example output:

  .                                   D        0  Wed Sep 22 18:17:51 2021
  ..                                  D        0  Wed Sep 22 12:03:59 2021
  prep-prod.txt                       N       71  Sun Sep 19 15:45:21 2021

                30313412 blocks of size 1024. 16480084 blocks available

The D flag means directory. N means normal file. The number after is the file size in bytes. The last line shows available disk space on the share.

get — Download a File from SMB Share

Command:

smb: \> get prep-prod.txt

Purpose: Downloads the specified file from the SMB share to your local machine. SMBclient transfers the file and shows the transfer size and speed.

Example output:

getting file \prep-prod.txt of size 71 as prep-prod.txt (8,7 KiloBytes/sec)
(average 8,7 KiloBytes/sec)
! — Run Local Commands Without Leaving SMB Session

Command:

smb: \> !ls
smb: \> !cat prep-prod.txt

Purpose: The exclamation mark prefix runs commands on your LOCAL machine without disconnecting from the SMB session. !ls lists your local directory, !cat prep-prod.txt reads the file you just downloaded. This is very convenient during a penetration test because you can download and immediately inspect files without leaving the SMB session.

Example interaction:

smb: \> !ls
prep-prod.txt

smb: \> !cat prep-prod.txt
[] check your code with the templates
[] run code-assessment.py
[] ...
smbstatus — Check Active SMB Connections (Server Side)

Command:

bash
root@samba:~# smbstatus

Purpose: Run on the Samba server itself (requires root), this command shows all currently active SMB connections. It reveals who is connected, from which IP address, which share they are accessing, which SMB protocol version they are using, and whether encryption and signing are active.

Example output:

Samba version 4.11.6-Ubuntu
PID     Username     Group        Machine                          Protocol Version  Encryption  Signing
------------------------------------------------------------------------
75691   sambauser    samba        10.10.14.4 (ipv4:10.10.14.4:45564)  SMB3_11    -           -

Service      pid     Machine       Connected at                     Encryption   Signing
---------------------------------------------------------------------------------------------
notes        75691   10.10.14.4   Do Sep 23 00:12:06 2021 CEST     -            -

No locked files

This shows user "sambauser" connected from 10.10.14.4, is accessing the "notes" share, and is using SMB3_11 (SMB 3.1.1). The - under Encryption and Signing means neither is active — a finding worth noting.

Footprinting SMB with Nmap

Command:

bash
sudo nmap 10.129.14.128 -sV -sC -p139,445

Purpose: Scans the two SMB-related ports. Port 139 is for NetBIOS session service (older SMB), port 445 is for modern SMB over TCP directly. The -sV detects versions, -sC runs default scripts.

Example output:

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

The output shows Samba version 4.6.2. Message signing is enabled but NOT required — meaning signing can be bypassed, which opens the server to relay attacks.

RPCclient — Manual SMB Enumeration
What is RPC?

RPC (Remote Procedure Call) is a technology that allows a program on one computer to execute functions or procedures on another computer as if they were local. In the Windows/Samba world, RPC is used to query all kinds of information from servers. The rpcclient tool implements MS-RPC functions for Linux.

Connecting with RPCclient

Command:

bash
rpcclient -U "" 10.129.14.128

Purpose: Connects to the SMB server using an empty username and no password (null session / anonymous access). The -U "" specifies an empty username. If the server allows null sessions, you get the rpcclient $> prompt.

Example:

Enter WORKGROUP\'s password:
rpcclient $>

Just press Enter when asked for the password. Success means null sessions are allowed — a security misconfiguration.

srvinfo — Get Server Information

Command:

rpcclient $> srvinfo

Purpose: Retrieves basic server information including the server name, OS version, and server type flags.

Example output:

DEVSMB         Wk Sv PrQ Unx NT SNT DEVSM
platform_id     :       500
os version      :       6.1
server type     :       0x809a03

The OS version 6.1 corresponds to Windows 7 / Windows Server 2008 R2. The server type flags tell you what roles the server is performing (workstation, server, print queue server, Unix machine, etc.).

enumdomains — List All Domains

Command:

rpcclient $> enumdomains

Purpose: Lists all Windows domains that are deployed and visible in the network.

Example output:

name:[DEVSMB] idx:[0x0]
name:[Builtin] idx:[0x1]

Shows the main domain (DEVSMB) and the built-in domain (which contains built-in groups like Administrators and Users).

querydominfo — Get Domain Information

Command:

rpcclient $> querydominfo

Purpose: Retrieves detailed information about the domain including the domain name, server name, total users, total groups, and domain server role.

Example output:

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

This reveals the domain is called "DEVOPS", there are 2 users, and this server is the Primary Domain Controller (PDC). Very valuable information for understanding the network structure.

netshareenumall — List All Shares with Paths

Command:

rpcclient $> netshareenumall

Purpose: Enumerates ALL available shares on the server, showing share names, descriptions, and most importantly — the actual filesystem paths on the server. This is more detailed than just listing share names.

Example output:

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

Now you know the real filesystem paths behind each share. For example, the "home" share maps to /home/ on the server, and "notes" maps to /mnt/notes/. This helps you understand the server's directory structure.

netsharegetinfo — Detailed Info About a Specific Share

Command:

rpcclient $> netsharegetinfo notes

Purpose: Gets very detailed information about a specific share including its path, permissions, maximum concurrent users, and the full ACL (Access Control List) in security descriptor format.

Example output:

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

The SID S-1-1-0 is the "Everyone" group — meaning ALL users (including anonymous) have full generic access to this share. Max_uses of -1 means unlimited simultaneous connections.

enumdomusers — List All Domain Users

Command:

rpcclient $> enumdomusers

Purpose: Lists all user accounts on the domain or server, along with their RID (Relative Identifier — similar to a Windows user ID number).

Example output:

user:[mrb3n] rid:[0x3e8]
user:[cry0l1t3] rid:[0x3e9]

This reveals two users: "mrb3n" with RID 0x3e8 (1000 in decimal) and "cry0l1t3" with RID 0x3e9 (1001 in decimal). These usernames can then be used for targeted brute force attacks.

queryuser — Get Detailed User Information by RID

Command:

rpcclient $> queryuser 0x3e9

Purpose: Retrieves complete details about a specific user identified by their RID (in hexadecimal). This reveals the full username, home directory path, profile path, last logon time, password last set time, account flags, and more.

Example output:

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

The home drive path \\devsmb\cry0l1t3 reveals the server name again. The password never needs to change (year 30828 expiry). Zero bad password count and zero logon count suggests this account has never been used yet — or logs are cleared.

querygroup — Get Group Information

Command:

rpcclient $> querygroup 0x201

Purpose: Retrieves information about a specific group identified by its RID.

Example output:

Group Name:     None
Description:    Ordinary Users
Group Attribute:7
Num Members:2

Group 0x201 is the standard user group "None" (equivalent to Domain Users) with 2 members — the same 2 users we found earlier.

Brute Forcing User RIDs with RPCclient
Why Brute Force RIDs?

Windows and Samba assign RIDs to users and groups sequentially starting from 500 for built-in accounts and 1000+ for regular accounts. By querying each RID in sequence, you can discover ALL users on the system even if enumdomusers is restricted.

Command:

bash
for i in $(seq 500 1100); do rpcclient -N -U "" 10.129.14.128 -c "queryuser 0x$(printf '%x\n' $i)" | grep "User Name\|user_rid\|group_rid" && echo ""; done

Purpose: This bash loop goes through numbers 500 to 1100, converts each to hexadecimal using printf '%x\n', then sends a queryuser RPC command for each one. If a user with that RID exists, it prints the username and group information. If not, no output is shown.

Example output:

        User Name   :   sambauser
        user_rid :      0x1f5
        group_rid:      0x201

        User Name   :   mrb3n
        user_rid :      0x3e8
        group_rid:      0x201

        User Name   :   cry0l1t3
        user_rid :      0x3e9
        group_rid:      0x201

Three users discovered. Sambauser has RID 0x1f5 (501 decimal — a built-in account), while mrb3n and cry0l1t3 are regular users starting at 1000.

Impacket samrdump.py — Automated User Enumeration

Command:

bash
samrdump.py 10.129.14.128

Purpose: A Python script from the Impacket toolkit that automatically connects to the SAM (Security Account Manager) database over SMB and enumerates all users. It provides much more detail than manual RID brute forcing and formats the output cleanly.

Example output:

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

This confirms both users are active (not disabled), their passwords expire (good security practice), and neither has a failed login count — suggesting fresh accounts.

SMBmap — Check Share Permissions Quickly

Command:

bash
smbmap -H 10.129.14.128

Purpose: Connects to the SMB server and quickly shows all available shares along with the exact permissions your current user has on each one (READ, WRITE, or NO ACCESS). The -H flag specifies the host IP address.

Example output:

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

In this case, anonymous access has NO ACCESS to everything. If any shares showed READ or READ,WRITE — those would be immediate targets for further investigation.

CrackMapExec — SMB Enumeration with Credentials

Command:

bash
crackmapexec smb 10.129.14.128 --shares -u '' -p ''

Purpose: CrackMapExec (CME) is a powerful post-exploitation tool for SMB enumeration. This command connects with empty username and password (anonymous) and lists all shares with their permissions. Unlike SMBmap, CME also shows the Windows version, hostname, SMB signing status, and whether SMBv1 is enabled.

Example output:

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

Critical finding: signing:False means SMB signing is not required — this allows relay attacks. SMBv1:False is good — SMBv1 is disabled. The "notes" share has READ,WRITE access for anonymous users — a major vulnerability.

Enum4Linux-ng — Complete Automated SMB Enumeration
Installation

Command:

bash
git clone https://github.com/cddmp/enum4linux-ng.git
cd enum4linux-ng
pip3 install -r requirements.txt

Purpose: Clones the enum4linux-ng repository from GitHub, enters the directory, and installs all required Python dependencies. Enum4linux-ng is the modern rewrite of the older enum4linux tool. It automates all SMB enumeration tasks including LDAP checks, SMB dialect detection, NetBIOS information, OS detection, user enumeration, group enumeration, share enumeration, policy enumeration, and printer enumeration.

Running Full Enumeration

Command:

bash
./enum4linux-ng.py 10.129.14.128 -A

Purpose: Runs ALL enumeration modules against the target. The -A flag means "All" — it enables every available check. This is the most comprehensive SMB enumeration you can do with a single command.

Key sections of the output explained:

Service Scan:

[*] Checking LDAP — [-] Could not connect (not a domain controller)
[*] Checking SMB — [+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS — [+] accessible on 139/tcp

NetBIOS Information:

[+] Got domain/workgroup name: DEVOPS
[+] Full NetBIOS names information:
- DEVSMB <00> - Workstation Service
- DEVSMB <20> - File Server Service
- DEVOPS <00> - Domain/Workgroup Name
- DEVOPS <1d> - Master Browser

SMB Dialect Check:

SMB 1.0: false
SMB 2.02: true
SMB 2.1: true
SMB 3.0: true
Preferred dialect: SMB 3.0
SMB signing required: false

RPC Session Check:

[+] Server allows session using username '', password ''
[+] Server allows session using username 'juzgtcsu', password ''

The second line means the server allows ANY random username with no password — very insecure.

Users:

'1000':
  username: mrb3n
  acb: '0x00000010'
'1001':
  username: cry0l1t3
  acb: '0x00000014'

Share Testing:

[*] Testing share home — [+] Mapping: OK, Listing: OK
[*] Testing share notes — [+] Mapping: OK, Listing: OK
[*] Testing share print$ — [+] Mapping: DENIED

Password Policy:

domain_password_information:
  min_pw_length: 5
  DOMAIN_PASSWORD_COMPLEX: false

Minimum password length is only 5 characters and complexity is not required — very weak password policy, making brute force attacks much easier.

NFS (Network File System)
What is NFS?

NFS (Network File System) was developed by Sun Microsystems and serves the same fundamental purpose as SMB — allowing files on a remote server to be accessed as if they were on the local machine. The key difference is that NFS is designed specifically for Linux and Unix systems, while SMB was designed for Windows. NFS clients cannot directly communicate with SMB servers. NFS uses a completely different underlying protocol — ONC-RPC (Open Network Computing Remote Procedure Call), also known as SUN-RPC, which operates on TCP and UDP port 111. For actual file transfer, NFS uses port 2049. Authentication in NFS versions 2 and 3 is based on UNIX UID/GID numbers — this means the server trusts whatever UID the client claims to have, which creates security risks if the server and client have different user mappings. NFSv4 adds Kerberos authentication and firewall-friendly operation using only port 2049.

NFS Version Comparison

NFSv2 — The oldest version, supported by many legacy systems. Originally operated entirely over UDP. Very limited features. Rarely seen in modern environments.

NFSv3 — More commonly seen. Supports variable file sizes and better error reporting. Not fully compatible with NFSv2 clients. Still uses port 111 for portmapping plus dynamic ports for services.

NFSv4 — The modern standard. Includes Kerberos authentication, works through firewalls (uses only port 2049), supports ACLs, uses stateful operations, and provides performance improvements and high security. NFSv4.1 adds parallel NFS (pNFS) for distributed file access and session trunking (NFS multipathing) for redundant connections.

NFS Configuration — /etc/exports
Viewing the NFS Exports File

Command:

bash
cat /etc/exports

Purpose: Reads the NFS exports configuration file. This file defines which local directories are shared (exported) over NFS and who can access them with what permissions. Each line defines one export: the directory path, the allowed hosts/networks, and the options in parentheses.

Example content:

# /etc/exports: the access control list for filesystems which may be exported
# Example for NFSv2 and NFSv3:
# /srv/homes hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
# Example for NFSv4:
# /srv/nfs4  gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)

The comments show the format: directory, then hostname or subnet in parentheses with options.

Adding an NFS Share

Command:

bash
echo '/mnt/nfs  10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports
systemctl restart nfs-kernel-server
exportfs

Purpose of each command:

echo '/mnt/nfs 10.129.14.0/24(sync,no_subtree_check)' >> /etc/exports — Adds a new export entry. This shares the /mnt/nfs directory to all hosts in the 10.129.14.0/24 subnet (all IPs from 10.129.14.1 to 10.129.14.254). The sync option means data is written to disk before the write is confirmed to the client (safer). The no_subtree_check disables checking whether files requested are within the exported subtree (improves reliability).

systemctl restart nfs-kernel-server — Restarts the NFS server to apply the new exports configuration.

exportfs — Displays the currently active NFS exports to confirm the new share is active.

Output of exportfs:

/mnt/nfs        10.129.14.0/24

Confirms the share is now active.

NFS Export Options Explained

rw — Read and Write permissions. Clients can read existing files AND create/modify/delete files.

ro — Read Only permissions. Clients can only read files — cannot make any changes.

sync — Synchronous data transfer. The server confirms writes only after data is physically written to disk. Slightly slower but safer — no data loss if the server crashes.

async — Asynchronous data transfer. The server confirms writes immediately without waiting for disk write. Faster but risks data loss if the server crashes before flushing to disk.

secure — Only allow connections from client ports below 1024. Ports below 1024 require root on the client, so this adds a small layer of security.

insecure — Allows connections from ANY client port including ports above 1024. Removes the port restriction.

no_subtree_check — Disables checking whether a requested file is actually within the exported directory tree. Improves reliability and performance, especially when files can be renamed while in use.

root_squash — This is an important security option. When the client connects as root (UID 0), the server maps that root access to the anonymous user (nobody). This prevents a root user on the client from having root access on the NFS server. This is ON by default in most configurations.

no_root_squash — The opposite of root_squash. Root on the client is treated as root on the server. This is DANGEROUS because anyone who gets root on the NFS client can modify any file on the NFS server including system files.

nohide — If another filesystem is mounted below an exported directory, normally it is hidden from NFS clients. With nohide, it becomes visible and is exported separately.

Dangerous NFS Settings

rw + no_root_squash — Combined, these two settings are extremely dangerous. Any client with root access gets read/write access to the entire NFS share with root privileges on the server side. An attacker who gains root on any client machine can completely compromise the NFS server's data.

insecure — Allows connections from ports above 1024. Since only root can use ports below 1024, the secure option adds a small but meaningful barrier. Removing it with insecure means any process can make NFS connections.

nohide — Can accidentally expose nested filesystems that administrators did not intend to share.

Footprinting NFS with Nmap
Basic NFS Port Scan

Command:

bash
sudo nmap 10.129.14.128 -p111,2049 -sV -sC

Purpose: Scans the two critical NFS ports. Port 111 is the RPC portmapper that tells clients which ports each RPC service is using. Port 2049 is the NFS service itself. The -sV detects versions, -sC runs default scripts including the rpcinfo script.

Example output:

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

The rpcinfo output shows all currently running RPC services and their ports. mountd (port 45837) handles mount requests, nlockmgr handles file locking, and nfs_acl handles ACLs.

NFS-Specific NSE Scripts Scan

Command:

bash
sudo nmap --script nfs* 10.129.14.128 -sV -p111,2049

Purpose: The nfs* wildcard runs ALL Nmap scripts whose names start with "nfs". This includes nfs-ls (list directory contents), nfs-showmount (show exports), and nfs-statfs (disk statistics). Together these scripts give a comprehensive view of the NFS service without needing to mount anything.

Example output:

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

This is extremely valuable. Without even mounting the share, you can see there are SSH private key files (id_rsa and id_rsa.pub) owned by root (UID 0) sitting in the NFS share. The disk is 29% used with 20GB available.

Performing NFS Tasks — Mounting and Accessing Shares
Step 1 — Show Available NFS Shares on Remote Server

Command:

bash
showmount -e 10.129.14.128

Purpose: Queries the remote NFS server for its export list — showing which directories are shared and which clients/networks are allowed to mount them.

Example output:

Export list for 10.129.14.128:
/mnt/nfs 10.129.14.0/24

The /mnt/nfs directory is exported to the 10.129.14.0/24 subnet.

Step 2 — Create a Local Mount Point

Command:

bash
mkdir target-NFS

Purpose: Creates an empty local directory called target-NFS that will serve as the mount point. The NFS share contents will appear inside this directory after mounting.

Step 3 — Mount the NFS Share

Command:

bash
sudo mount -t nfs 10.129.14.128:/ ./target-NFS/ -o nolock

Purpose: Mounts the NFS share from the remote server onto your local directory.

Breaking down each part:

sudo — Root privileges required to mount filesystems
mount — The mount command
-t nfs — Specifies the filesystem type is NFS
10.129.14.128:/ — The remote server IP and the path being mounted (root of the server's NFS exports)
./target-NFS/ — The local directory to mount into
-o nolock — Option to disable NFS file locking (prevents issues when the NFS server doesn't support the lock manager)
Step 4 — Navigate and View Mounted Content

Commands:

bash
cd target-NFS
tree .

Purpose: Enters the mounted NFS directory and displays the file structure in tree format.

Example output:

.
└── mnt
    └── nfs
        ├── id_rsa
        ├── id_rsa.pub
        └── nfs.share

2 directories, 3 files

You now have local access to all these files as if they were on your own machine.

Step 5 — List Files with Usernames and Group Names

Command:

bash
ls -l mnt/nfs/

Purpose: Lists files with their symbolic owner names (username and group name). This requires that the UID/GID numbers on the mounted files match existing users on YOUR local machine. If they do, you see real names.

Example output:

total 16
-rw-r--r-- 1 cry0l1t3 cry0l1t3 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 cry0l1t3 cry0l1t3  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1 root     root     1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1 root     root      348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1 root     root        0 Sep 19 17:22 nfs.share

Files owned by "cry0l1t3" and "root" are visible. If you create a local user with the same UID as cry0l1t3, you can read their private files.

Step 6 — List Files with UIDs and GIDs (Numeric)

Command:

bash
ls -n mnt/nfs/

Purpose: The -n flag shows numeric UIDs and GIDs instead of resolving them to names. This is important for penetration testing because it shows you the exact numeric IDs, which you can use to create matching local users to gain access to the files.

Example output:

total 16
-rw-r--r-- 1 1000 1000 1872 Sep 25 00:55 cry0l1t3.priv
-rw-r--r-- 1 1000 1000  348 Sep 25 00:55 cry0l1t3.pub
-rw-r--r-- 1    0 1000 1221 Sep 19 18:21 backup.sh
-rw-r--r-- 1    0    0 1872 Sep 19 17:27 id_rsa
-rw-r--r-- 1    0    0  348 Sep 19 17:28 id_rsa.pub
-rw-r--r-- 1    0    0    0 Sep 19 17:22 nfs.share

Now you can see the exact UIDs. User "cry0l1t3" is UID 1000. The id_rsa files are owned by root (UID 0). The backup.sh is owned by root (UID 0) but in the cry0l1t3 group (GID 1000) — interesting.

Important note: If root_squash is enabled, even if you are root on your local machine, you cannot modify files owned by root on the NFS share because root_squash maps your root access to nobody.

Step 7 — Unmount the NFS Share

Command:

bash
cd ..
sudo umount ./target-NFS

Purpose: First navigate OUT of the mounted directory (you cannot unmount a filesystem you are currently inside), then unmount it. umount removes the NFS mount and disconnects from the remote server. Always unmount when done to clean up properly and avoid leaving open connections.

DNS (Domain Name System)
What is DNS?

DNS (Domain Name System) is the internet's distributed address book. It translates human-readable domain names like academy.hackthebox.com into the numeric IP addresses that computers use to communicate. DNS has no central database — instead information is spread across thousands of name servers worldwide organized in a hierarchical tree structure. At the top are Root Servers (13 worldwide managed by ICANN), below them are Top-Level Domain servers (.com, .org, .net), then Second-Level Domain servers (hackthebox.com), then subdomain servers (academy.hackthebox.com). DNS is mostly unencrypted, making queries visible to network monitors and ISPs. Modern alternatives like DNS over TLS (DoT) and DNS over HTTPS (DoH) provide encrypted DNS queries.

DNS Server Types

DNS Root Server — The highest level in the DNS hierarchy. There are exactly 13 root servers worldwide coordinated by ICANN. They only respond when all other DNS servers fail to resolve a name. They handle top-level domain queries.

Authoritative Name Server — Holds the official, definitive records for a specific zone (domain). When you register hackthebox.com, you configure authoritative name servers that hold the actual DNS records. Their answers are definitive and binding.

Non-authoritative Name Server — Does not own any zone but collects DNS information by making recursive or iterative queries on behalf of clients. They are middlemen that do the work of finding answers.

Caching DNS Server — Stores (caches) recently resolved DNS answers for a period determined by the TTL (Time To Live) in each record. This speeds up repeat queries. Your ISP's DNS server is usually a caching server.

Forwarding Server — Simply passes DNS queries to another specified DNS server. Often used in corporate environments to route all DNS through a central server.

Resolver — The DNS client built into operating systems. When you visit a website, your OS uses its resolver to send the DNS query to a configured DNS server.

DNS Record Types — Complete Reference

A Record — Maps a domain name to an IPv4 address. The most common record type. Example: hackthebox.com → 104.26.10.78

AAAA Record — Maps a domain name to an IPv6 address. The quad-A record for the newer IPv6 protocol. Example: hackthebox.com → 2606:4700::6817:0a4e

MX Record — Mail Exchange record. Specifies which mail servers handle email for the domain. Has a priority number — lower number means higher priority. Example: mail.hackthebox.com priority 10

NS Record — Name Server record. Specifies which DNS servers are authoritative for the domain. Example: ns1.inwx.net — tells you who hosts the DNS for the domain.

TXT Record — Text records that store arbitrary text data. Used for email security (SPF, DKIM, DMARC), domain ownership verification (for Google, Microsoft, Atlassian), and other purposes. Extremely valuable for OSINT.

CNAME Record — Canonical Name / Alias record. Makes one domain point to another domain instead of an IP address. Example: www.hackthebox.com → hackthebox.com

PTR Record — Pointer record for reverse DNS lookup. Maps an IP address BACK to a domain name. Used for email verification and logging.

SOA Record — Start of Authority. Contains administrative information about the zone: the primary nameserver, admin email address (with @ replaced by .), serial number (version), refresh interval, retry interval, expiry time, and minimum TTL.

DNS Configuration Files — BIND9

BIND9 is the most widely used DNS server software on Linux. It uses three main configuration files:

Local Configuration File

Command:

bash
cat /etc/bind/named.conf.local

Purpose: Defines which zones (domains) this DNS server is responsible for and where their zone files are located.

Example content:

zone "domain.com" {
    type master;
    file "/etc/bind/db.domain.com";
    allow-update { key rndc-key; };
};

This declares that this server is the master (primary authoritative) for domain.com, the zone data is stored in /etc/bind/db.domain.com, and updates are allowed using the rndc-key for secure dynamic updates.

Zone File — Forward Lookup

Command:

bash
cat /etc/bind/db.domain.com

Purpose: The zone file contains all DNS records for a domain. It is the actual phone book for the domain — mapping names to IP addresses and vice versa.

Example content with explanation:

$ORIGIN domain.com
$TTL 86400
@     IN     SOA    dns1.domain.com.     hostmaster.domain.com. (
                    2001062501 ; serial number - increment on every change
                    21600      ; refresh - slaves check every 6 hours
                    3600       ; retry - if refresh fails, retry after 1 hour
                    604800     ; expire - slaves discard records after 1 week
                    86400 )    ; minimum TTL - 1 day

      IN     NS     ns1.domain.com.     ; primary nameserver
      IN     NS     ns2.domain.com.     ; secondary nameserver

      IN     MX     10     mx.domain.com.   ; primary mail server
      IN     MX     20     mx2.domain.com.  ; backup mail server

             IN     A       10.129.14.5      ; domain itself resolves to this IP

server1      IN     A       10.129.14.5      ; server1 subdomain
server2      IN     A       10.129.14.7      ; server2 subdomain
ns1          IN     A       10.129.14.2      ; nameserver 1
ns2          IN     A       10.129.14.3      ; nameserver 2

ftp          IN     CNAME   server1          ; ftp alias to server1
mx           IN     CNAME   server1          ; mx alias to server1
mx2          IN     CNAME   server2          ; mx2 alias to server2
www          IN     CNAME   server2          ; www alias to server2

The $ORIGIN sets the default domain. $TTL 86400 means records are cached for 86400 seconds (1 day). The @ symbol represents the domain itself. Records ending without a dot are relative to the $ORIGIN — those ending WITH a dot are absolute.

Reverse Name Resolution Zone File

Command:

bash
cat /etc/bind/db.10.129.14

Purpose: The reverse zone file maps IP addresses back to hostnames using PTR records. The file is named after the network address in reverse. For network 10.129.14.x, the reverse zone is 14.129.10.in-addr.arpa.

Example content:

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

The "5" in 5 IN PTR server1.domain.com. refers to the last octet of the IP address (10.129.14.5). So this says: IP 10.129.14.5 resolves to server1.domain.com.

Dangerous DNS Settings

allow-query { any; } — Allows ANY host on the internet to send DNS queries to this server. Combined with recursion, this can make the server an amplifier for DDoS attacks.

allow-recursion { any; } — Allows recursive queries from any host. A recursive query makes the DNS server do all the work of finding the answer and returning it. Allowing this from external hosts enables DNS amplification attacks.

allow-transfer { any; } — The most dangerous setting. Allows ANY host to request a full zone transfer (AXFR). This lets anyone download the complete list of ALL DNS records for the domain — revealing every hostname and IP address in the organization.

zone-statistics — Enables collection of query statistics. If accessible, reveals what domains are being queried and how often.

Footprinting DNS — Complete Commands
DIG — NS Query (Find Name Servers)

Command:

bash
dig ns inlanefreight.htb @10.129.14.128

Purpose: Queries the specified DNS server (10.129.14.128) for the NS records of the domain inlanefreight.htb. This reveals which name servers are authoritative for the domain.

Breaking down:

dig — DNS lookup tool
ns — Record type to query
inlanefreight.htb — The target domain
@10.129.14.128 — Which DNS server to ask (the target itself)

Example output:

;; ANSWER SECTION:
inlanefreight.htb.      604800  IN      NS      ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

The answer shows there is one name server: ns.inlanefreight.htb at IP 10.129.34.136. The 604800 is the TTL (604800 seconds = 1 week). Now you know the IP of the authoritative DNS server.

DIG — Version Query (Identify DNS Software Version)

Command:

bash
dig CH TXT version.bind 10.129.120.85

Purpose: Sends a special CHAOS class TXT query asking for the DNS server's version information. Many DNS servers are configured to respond to this query with their software version. This is useful for finding known vulnerabilities in specific DNS software versions.

Breaking down:

CH — The CHAOS class (a special DNS class separate from the normal IN/Internet class)
TXT — Record type
version.bind — Special query name that requests version information from BIND servers

Example output:

;; ANSWER SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1"

;; ADDITIONAL SECTION:
version.bind.       0       CH      TXT     "9.10.6-P1-Debian"

The DNS server is running BIND version 9.10.6-P1 on Debian Linux. This specific version can be searched for known CVEs.

DIG — ANY Query (Retrieve All Record Types)

Command:

bash
dig any inlanefreight.htb @10.129.14.128

Purpose: Requests ALL available record types for the domain in a single query. The server returns everything it is willing to disclose — which may not be everything that exists, but gives the broadest possible view from a single query.

Example output:

;; ANSWER SECTION:
inlanefreight.htb. 604800 IN TXT "v=spf1 include:mailgun.org include:_spf.google.com..."
inlanefreight.htb. 604800 IN TXT "atlassian-domain-verification=t1rKCy68JFszSdCKVpw64A1..."
inlanefreight.htb. 604800 IN TXT "MS=ms97310371"
inlanefreight.htb. 604800 IN SOA inlanefreight.htb. root.inlanefreight.htb. 2 604800...
inlanefreight.htb. 604800 IN NS  ns.inlanefreight.htb.

;; ADDITIONAL SECTION:
ns.inlanefreight.htb.   604800  IN      A       10.129.34.136

From this single query you can learn: the company uses Mailgun and Google for email, they use Atlassian tools, the SOA record shows root@inlanefreight.htb is the admin contact, and the only NS is at 10.129.34.136.

DIG — AXFR Zone Transfer (External Zone)

Command:

bash
dig axfr inlanefreight.htb @10.129.14.128

Purpose: Attempts a full zone transfer (AXFR - Asynchronous Full Transfer Zone) from the target DNS server. If the server is misconfigured to allow zone transfers from any IP, this downloads the COMPLETE zone file — every single DNS record for the domain.

Example output:

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

This reveals ALL hostnames and their IPs: an app server at 10.129.18.15, an internal server at 10.129.1.6, a mail server at 10.129.18.201. The zone transfer exposed the complete internal network map.

DIG — AXFR Zone Transfer (Internal Zone)

Command:

bash
dig axfr internal.inlanefreight.htb @10.129.14.128

Purpose: Attempts zone transfer for the INTERNAL subdomain zone. Internal zones often contain even more sensitive information — domain controllers, VPN servers, internal workstations, and management systems.

Example output:

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

This is a gold mine. Two domain controllers (dc1 at 10.129.34.16, dc2 at 10.129.34.11), a VPN server, two workstations (ws1, ws2), a WSUS (Windows Update) server, and internal mail. This is the complete internal network structure revealed by one DNS misconfiguration.

Subdomain Brute Forcing with Bash Loop

Command:

bash
for sub in $(cat /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt); do dig $sub.inlanefreight.htb @10.129.14.128 | grep -v ';\|SOA' | sed -r '/^\s*$/d' | grep $sub | tee -a subdomains.txt; done

Purpose: Goes through every word in the subdomain wordlist, constructs a DNS query for each potential subdomain, and saves any that resolve to a file.

Breaking down:

cat /opt/useful/seclists/.../subdomains-top1million-110000.txt — Reads a list of 110,000 common subdomain names
for sub in $(...) — Loops through each subdomain name
dig $sub.inlanefreight.htb @10.129.14.128 — Queries for each potential subdomain
grep -v ';\|SOA' — Removes comment lines and SOA records
sed -r '/^\s*$/d' — Removes empty lines
grep $sub — Keeps only lines containing the subdomain name (meaning it resolved)
tee -a subdomains.txt — Writes matching results to file AND displays them

Example output:

ns.inlanefreight.htb.   604800  IN      A       10.129.34.136
mail1.inlanefreight.htb. 604800 IN      A       10.129.18.201
app.inlanefreight.htb.  604800  IN      A       10.129.18.15
DNSenum — Automated DNS Enumeration

Command:

bash
dnsenum --dnsserver 10.129.14.128 --enum -p 0 -s 0 -o subdomains.txt -f /opt/useful/seclists/Discovery/DNS/subdomains-top1million-110000.txt inlanefreight.htb

Purpose: DNSenum automates the complete DNS enumeration process in one command. It performs host address lookup, NS record enumeration, zone transfer attempts, reverse lookups, and subdomain brute forcing.

Breaking down the flags:

--dnsserver 10.129.14.128 — Use this specific DNS server for all queries
--enum — Enable full enumeration mode
-p 0 — No Google scraping (0 Google pages)
-s 0 — No Google scraping subdomains
-o subdomains.txt — Save results to this file
-f /opt/.../subdomains-top1million-110000.txt — Use this wordlist for brute forcing
inlanefreight.htb — Target domain

Example output:

dnsenum VERSION:1.2.6
-----   inlanefreight.htb   -----

Host's addresses:

Name Servers:
ns.inlanefreight.htb.  604800  IN  A  10.129.34.136

Trying Zone Transfers:
Trying Zone Transfer for inlanefreight.htb on ns.inlanefreight.htb...
AXFR record query failed: no nameservers

Brute forcing with subdomains-top1million-110000.txt:
ns.inlanefreight.htb.    604800  IN  A  10.129.34.136
mail1.inlanefreight.htb. 604800  IN  A  10.129.18.201
app.inlanefreight.htb.   604800  IN  A  10.129.18.15
SMTP (Simple Mail Transfer Protocol)
What is SMTP?

SMTP is the standard protocol for sending email across IP networks. It handles email transmission between email clients and outgoing mail servers, and between mail servers themselves. By default, SMTP listens on port 25, but modern implementations also use port 587 for authenticated users with STARTTLS encryption. The email delivery chain works like this: your email client (MUA — Mail User Agent) sends the email to a Mail Submission Agent (MSA), which checks its validity and relays it to a Mail Transfer Agent (MTA). The MTA looks up the recipient's mail server using DNS MX records and delivers the message. At the destination, a Mail Delivery Agent (MDA) places it in the recipient's mailbox for retrieval via IMAP or POP3.

SMTP has two major weaknesses. First, it provides no guaranteed delivery confirmation — you may only get an English-language error message if delivery fails. Second, it has no built-in sender authentication, making email spoofing possible. Technologies like SPF (Sender Policy Framework), DKIM (DomainKeys Identified Mail), and DMARC were developed to address these shortcomings. ESMTP (Extended SMTP) adds support for TLS encryption via the STARTTLS command, securing the connection after initial contact.

Default SMTP Configuration

Command:

bash
cat /etc/postfix/main.cf | grep -v "#" | sed -r "/^\s*$/d"

Purpose: Reads the Postfix mail server configuration (the most common SMTP server on Linux) filtering out comments and blank lines.

Example output:

smtpd_banner = ESMTP Server
biff = no
append_dot_mydomain = no
compatibility_level = 2
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache
myhostname = mail1.inlanefreight.htb
alias_maps = hash:/etc/aliases
mydestination = $myhostname, localhost
mynetworks = 127.0.0.0/8 10.129.0.0/16
mailbox_size_limit = 0
smtp_bind_address = 0.0.0.0
inet_protocols = ipv4
smtpd_helo_restrictions = reject_invalid_hostname
home_mailbox = /home/postfix

Key settings: myhostname reveals the server's hostname. mynetworks shows which networks can relay email (if set too broadly, open relay). smtpd_banner is what the server announces — ESMTP Server is generic which is good for security but less informative for us.

SMTP Commands — Complete Reference

AUTH PLAIN — Authentication extension. Used to authenticate the client with a username and password. AUTH is required by ESMTP to prevent spam relaying.

HELO — The basic greeting command. The client introduces itself with its hostname. HELO mail1.example.com starts a basic SMTP session.

EHLO — Extended Hello. Like HELO but for ESMTP. The server responds with a list of all ESMTP extensions it supports (STARTTLS, AUTH, SIZE, etc.).

MAIL FROM: — Specifies the sender's email address. Example: MAIL FROM: <sender@example.com>

RCPT TO: — Specifies the recipient's email address. Multiple RCPT TO commands can be used for multiple recipients. Example: RCPT TO: <recipient@example.com>

DATA — Signals the beginning of the email body. After sending DATA, you type the email headers (From:, To:, Subject:, Date:) followed by a blank line, then the message body. End with a single line containing only a period (.).

RSET — Reset. Aborts the current mail transaction without closing the connection. Clears the sender, recipient, and data but keeps the session open.

VRFY — Verify. Asks the server if a mailbox/username exists. Example: VRFY username. Some servers respond differently for valid vs invalid users, enabling username enumeration.

EXPN — Expand. Similar to VRFY but also expands mailing lists to show individual members.

NOOP — No Operation. Sends a command that does nothing except keep the connection alive and get a response. Used to prevent timeout disconnections.

QUIT — Terminates the SMTP session cleanly.

Performing SMTP Tasks
Connecting with Telnet

Command:

bash
telnet 10.129.14.128 25

Purpose: Opens a raw TCP connection to the SMTP server on port 25 using telnet, allowing you to manually type SMTP commands and see exact server responses.

HELO/EHLO Interaction

Full example:

telnet 10.129.14.128 25

Trying 10.129.14.128...
Connected to 10.129.14.128.
Escape character is '^]'.
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

What this reveals: EHLO response shows all server capabilities. PIPELINING means multiple commands can be sent without waiting for each response. SIZE 10240000 means max email size is about 10MB. VRFY being listed means user enumeration may be possible. This information helps plan the next steps.

User Enumeration with VRFY

Command:

VRFY root
VRFY cry0l1t3
VRFY testuser
VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa

Purpose: The VRFY command asks the server to verify if a username or mailbox exists. Depending on configuration, the server may respond differently for real vs fake usernames, enabling username discovery.

Example output:

VRFY root
252 2.0.0 root

VRFY cry0l1t3
252 2.0.0 cry0l1t3

VRFY testuser
252 2.0.0 testuser

VRFY aaaaaaaaaaaaaaaaaaaaaaaaaaaa
252 2.0.0 aaaaaaaaaaaaaaaaaaaaaaaaaaaa

In this example, the server returns 252 for everything — even fake users. This means this particular server is configured to not confirm or deny existence. However, some servers return 250 for valid users and 550 for invalid ones, enabling reliable enumeration.

Sending an Email via Telnet

Complete email sending example:

telnet 10.129.14.128 25

220 ESMTP Server

EHLO inlanefreight.htb
250-mail1.inlanefreight.htb
250 CHUNKING

MAIL FROM: <cry0l1t3@inlanefreight.htb>
250 2.1.0 Ok

RCPT TO: <mrb3n@inlanefreight.htb> NOTIFY=success,failure
250 2.1.5 Ok

DATA
354 End data with <CR><LF>.<CR><LF>

From: <cry0l1t3@inlanefreight.htb>
To: <mrb3n@inlanefreight.htb>
Subject: DB
Date: Tue, 28 Sept 2021 16:32:51 +0200

Hey man, I am trying to access our XY-DB but the creds don't work.
Did you make any changes there?
.
250 2.0.0 Ok: queued as 6E1CF1681AB

QUIT
221 2.0.0 Bye
Connection closed by foreign host.

Breaking down each step: EHLO starts extended session. MAIL FROM sets sender. RCPT TO sets recipient with NOTIFY=success,failure requesting delivery notifications. DATA begins body input. The headers (From, To, Subject, Date) are typed first, then blank line, then body. A single . on its own line ends the message. Server responds 250 meaning accepted and queued.

Dangerous SMTP Setting — Open Relay

What is an Open Relay?

An open relay is an SMTP server that accepts and forwards email from anyone, to anyone, without requiring authentication. The dangerous configuration is:

mynetworks = 0.0.0.0/0

This tells Postfix to accept email relay from ALL IP addresses. Attackers can use this to send spam, phishing emails, or spoofed messages that appear to come from the company's legitimate email domain. This damages the company's reputation and can get their domain blacklisted.

Footprinting SMTP with Nmap
Basic SMTP Scan

Command:

bash
sudo nmap 10.129.14.128 -sC -sV -p25

Purpose: Scans port 25 and runs default scripts. The smtp-commands NSE script automatically sends an EHLO command and lists all supported SMTP extensions.

Example output:

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
|_smtp-commands: mail1.inlanefreight.htb, PIPELINING, SIZE 10240000, VRFY,
ETRN, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING,

The banner reveals it is Postfix and the hostname mail1.inlanefreight.htb. The VRFY capability confirms user enumeration may be possible.

Check for Open Relay

Command:

bash
sudo nmap 10.129.14.128 -p25 --script smtp-open-relay -v

Purpose: Runs the smtp-open-relay NSE script which performs 16 different relay tests against the SMTP server to determine if it can be abused as an open relay.

Example output:

PORT   STATE SERVICE
25/tcp open  smtp
| smtp-open-relay: Server is an open relay (16/16 tests)
|  MAIL FROM:<> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@nmap.scanme.org> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@ESMTP> -> RCPT TO:<relaytest@nmap.scanme.org>
|  MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<relaytest@nmap.scanme.org>
...
|_ MAIL FROM:<antispam@[10.129.14.128]> -> RCPT TO:<nmap.scanme.org!relaytest@ESMTP>

All 16 tests passed — this server is a fully open relay that accepts email from any source to any destination. This is a critical finding in any penetration test.

IMAP / POP3
What are IMAP and POP3?

IMAP (Internet Message Access Protocol) and POP3 (Post Office Protocol 3) are both used to RECEIVE emails. IMAP is the modern standard and works by keeping emails on the server — you manage your mailbox directly on the server, and changes sync across all devices. IMAP supports folder structures, simultaneous access by multiple clients, and online email management. It uses port 143 (plain) and 993 (SSL/TLS). POP3 is simpler and older — it only provides three functions: list emails, download emails, and delete emails. It typically downloads emails to one device and removes them from the server. POP3 uses port 110 (plain) and 995 (SSL/TLS). Both protocols can be exploited if misconfigured — particularly if they expose verbose debugging information, allow anonymous access, or use weak authentication.

IMAP Commands — Complete Reference

1 LOGIN username password — Authenticates the user. The "1" is a tag (identifier) that labels this command — the server includes it in its response so you can match responses to commands. Example: 1 LOGIN john password123

1 LIST "" * — Lists all mailbox folders available to the authenticated user. The first "" is the reference name (empty means start from root) and * is a wildcard matching all names.

1 CREATE "INBOX" — Creates a new mailbox folder with the specified name.

1 DELETE "INBOX" — Permanently deletes a mailbox folder and all messages inside it.

1 RENAME "ToRead" "Important" — Renames an existing mailbox folder. First argument is old name, second is new name.

1 LSUB "" * — Returns only folders the user has subscribed to. Subscriptions are a way to organize which folders are actively monitored.

1 SELECT INBOX — Opens a specific mailbox for access. After selecting, you can fetch, search, and manage messages. The server responds with the number of messages and recent messages.

1 UNSELECT INBOX — Exits the currently selected mailbox without closing the connection. Returns to authenticated state.

1 FETCH <ID> all — Retrieves all data (headers, body, flags) associated with a specific message. The ID is the message sequence number.

1 CLOSE — Removes all messages marked with the Deleted flag from the currently selected mailbox and then deselects it.

1 LOGOUT — Properly closes the IMAP connection. The server closes the connection after acknowledging this command.

POP3 Commands — Complete Reference

USER username — Identifies which user is trying to log in. This is the first step of POP3 authentication. Example: USER johnsmith

PASS password — Provides the password for the user identified with USER. Together USER and PASS complete POP3 authentication.

STAT — Requests the number of emails currently in the mailbox and their total size in bytes. Response format: +OK <count> <total-bytes>

LIST — Requests a list of all messages with their individual sizes. Response shows each message number and its size. Can also take a message number to show only that message.

RETR id — Downloads (retrieves) a specific email by its ID number. The complete email including all headers and body is sent. Example: RETR 1 downloads message number 1.

DELE id — Marks a specific message for deletion. Messages are not actually deleted until the session ends with QUIT.

CAPA — Requests the server to list all its capabilities and supported extensions. Similar to EHLO for SMTP.

RSET — Resets the session — unmarks all messages marked for deletion. Useful if you change your mind about deleting messages.

QUIT — Ends the POP3 session. Any messages marked with DELE are permanently deleted at this point.

Dangerous IMAP/POP3 Settings

auth_debug — Enables all authentication debug logging. Every detail of the authentication process is written to logs. If logs are accessible, an attacker can see authentication attempts and potentially usernames and passwords.

auth_debug_passwords — When enabled, the actual submitted passwords (even if wrong) are written to the log file. This means every password attempt — successful or not — is logged in plain text. A serious security risk.

auth_verbose — Logs failed authentication attempts along with the reason for failure. This can reveal valid usernames because "user unknown" and "wrong password" produce different messages.

auth_verbose_passwords — Logs submitted passwords (can be truncated). Similar to auth_debug_passwords but for verbose logging mode.

auth_anonymous_username — Specifies what username is used when logging in with the ANONYMOUS SASL mechanism. If ANONYMOUS auth is enabled, anyone can access email without credentials.

Footprinting IMAP/POP3 with Nmap

Command:

bash
sudo nmap 10.129.14.128 -sV -p110,143,993,995 -sC

Purpose: Scans all four mail protocol ports in one command. Port 110 for POP3 plain, 143 for IMAP plain, 993 for IMAP over SSL, 995 for POP3 over SSL. Default scripts reveal SSL certificates which contain organizational information.

Example output:

PORT    STATE SERVICE  VERSION
110/tcp open  pop3     Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE SASL STLS TOP UIDL RESP-CODES CAPA PIPELINING
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight
| stateOrProvinceName=California/countryName=US
| Not valid before: 2021-09-19T19:44:58
|_Not valid after:  2295-07-04T19:44:58

143/tcp open  imap     Dovecot imapd
|_imap-capabilities: more have post-login STARTTLS LOGIN-REFERRALS LITERAL+
| ssl-cert: Subject: commonName=mail1.inlanefreight.htb/organizationName=Inlanefreight

993/tcp open  ssl/imap Dovecot imapd
|_imap-capabilities: more have post-login AUTH=PLAINA0001 SASL-IR ENABLE IDLE IMAP4rev1

995/tcp open  ssl/pop3 Dovecot pop3d
|_pop3-capabilities: AUTH-RESP-CODE USER SASL(PLAIN) TOP UIDL RESP-CODES CAPA PIPELINING

All four ports are open running Dovecot. The SSL certificate reveals the hostname (mail1.inlanefreight.htb), organization (Inlanefreight), and location (California, US). AUTH=PLAIN on port 993 means plain text authentication over an encrypted connection is possible.

Connecting to IMAP with cURL
Basic IMAP Connection

Command:

bash
curl -k 'imaps://10.129.14.128' --user user:p4ssw0rd

Purpose: Uses cURL to connect to the IMAP server over SSL (imaps://) and authenticate. The -k flag ignores SSL certificate verification errors (useful for self-signed certs). If authentication succeeds, cURL lists the available mailbox folders.

Example output:

* LIST (\HasNoChildren) "." Important
* LIST (\HasNoChildren) "." INBOX

Two mailbox folders exist: "Important" and "INBOX". You can now access emails in these folders.

Verbose IMAP Connection (Showing Full TLS Handshake)

Command:

bash
curl -k 'imaps://10.129.14.128' --user cry0l1t3:1234 -v

Purpose: The -v (verbose) flag shows the complete connection process including the TLS handshake, SSL certificate details, the IMAP authentication exchange, and all IMAP commands and responses. This is extremely detailed and reveals the TLS version, cipher suite, certificate chain, and exact IMAP capabilities.

Key parts of the output:

* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* SSL connection using TLSv1.3 / TLS_AES_256_GCM_SHA384
* Server certificate:
*  subject: C=US; ST=California; L=Sacramento; O=Inlanefreight; OU=Customer Support;
   CN=mail1.inlanefreight.htb; emailAddress=cry0l1t3@inlanefreight.htb
*  expire date: Jul  4 19:44:58 2295 GMT
< * OK [CAPABILITY IMAP4rev1 SASL-IR LOGIN-REFERRALS ENABLE IDLE LITERAL+ AUTH=PLAIN]
  HTB-Academy IMAP4 v.0.21.4
> A002 AUTHENTICATE PLAIN AGNyeTBsMXQzADEyMzQ=
< A002 OK Logged in
> A003 LIST "" *
< * LIST (\HasNoChildren) "." Important
< * LIST (\HasNoChildren) "." INBOX

The certificate reveals the admin email (cry0l1t3@inlanefreight.htb) — this is the email address used when the certificate was created. The IMAP banner shows HTB-Academy IMAP4 v.0.21.4 — a custom version string.

OpenSSL — Connecting to POP3 over SSL

Command:

bash
openssl s_client -connect 10.129.14.128:pop3s

Purpose: Connects to the POP3 over SSL service using OpenSSL. This shows the complete SSL handshake and certificate information, then gives you an interactive POP3 session where you can manually type POP3 commands.

What the output shows:

Complete SSL session details including session ID, cipher, and TLS version
Full certificate chain with subject and issuer information
Session ticket for session resumption
After the handshake: +OK HTB-Academy POP3 Server — the POP3 banner
OpenSSL — Connecting to IMAP over SSL

Command:

bash
openssl s_client -connect 10.129.14.128:imaps

Purpose: Connects to IMAP over SSL (port 993). Similar to the POP3 connection but for IMAP. After the TLS handshake, you get an IMAP session where you can type IMAP commands directly. The IMAP banner at the end shows: * OK [CAPABILITY IMAP4rev1 SASL-IR...] HTB-Academy IMAP4 v.0.21.4

SNMP (Simple Network Management Protocol)
What is SNMP?

SNMP (Simple Network Management Protocol) was created specifically for monitoring and managing network devices remotely. It works with routers, switches, servers, printers, IoT devices, and virtually any networked hardware. SNMP transmits control commands over UDP port 161 and receives unsolicited notifications (called traps) on UDP port 162. Traps are proactive alerts — when something specific happens on a device (like a port going down), the device automatically sends a trap to the management server without being asked. SNMP uses a data model called MIB (Management Information Base) and addresses individual data points using OIDs (Object Identifiers). Without SNMP, network administrators would need to log into each device individually to check status.

MIB — Management Information Base

The MIB is a structured text file that describes all the objects (data points) that can be queried or configured on a device. It is like a dictionary that explains what each OID means. MIB files are written in ASN.1 (Abstract Syntax Notation One) format. Every queryable object in SNMP has at least one OID, a name, a data type, access rights (read-only or read-write), and a description. MIBs do not contain actual data — they define the structure and meaning of data. The actual current values are stored in the device and retrieved using the OID addresses defined in the MIB.

OID — Object Identifier

An OID is a unique address in a hierarchical tree structure that identifies a specific piece of information on a network device. OIDs look like: 1.3.6.1.2.1.1.5.0 — a series of numbers separated by dots. Each number represents a node in the hierarchy. The longer the OID, the more specific the information. For example: 1.3.6.1.2.1.1.1.0 might return the system description, while 1.3.6.1.2.1.1.5.0 returns the hostname. You can look up OIDs in the Object Identifier Registry.

SNMP Versions

SNMPv1 — The original version. No authentication beyond the community string, no encryption. Everything transmitted in plain text. Still used in some small or legacy networks. Anyone on the network can intercept and read all SNMP communications.

SNMPv2c — The "c" stands for community-based. Added bulk retrieval and improved error handling. Security is identical to v1 — community strings still transmitted in plain text, no encryption. The most widely deployed version despite its weaknesses because it is simple to configure.

SNMPv3 — Added proper security: authentication using username and password, and encryption of data using a pre-shared key. Uses either MD5 or SHA for authentication and DES or AES for encryption. Significantly more secure but also more complex to configure, which is why many organizations still use v2c.

Community Strings

Community strings in SNMPv1 and v2c function as simple passwords. There are typically two: the read community string (default "public") which allows reading device information, and the write community string (default "private") which allows modifying device configuration. Because these default values are well-known and many administrators never change them, they are the first things to try when attacking SNMP. Even when changed, since community strings are sent in plain text over the network, they can be intercepted with a packet sniffer.

Default SNMP Configuration

Command:

bash
cat /etc/snmp/snmpd.conf | grep -v "#" | sed -r '/^\s*$/d'

Purpose: Reads the SNMP daemon configuration file filtering out comments and blank lines to show only active settings.

Example output:

sysLocation    Sitting on the Dock of the Bay
sysContact     Me <me@example.org>
sysServices    72
master  agentx
agentaddress  127.0.0.1,[::1]
view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1
rocommunity  public default -V systemonly
rocommunity6 public default -V systemonly
rouser authPrivUser authpriv -V systemonly

sysLocation and sysContact are informational fields visible to SNMP clients — they reveal the physical location and admin contact. rocommunity public default means the read-only community string "public" is accessible from all hosts (default) but limited to the systemonly view. agentaddress 127.0.0.1 limits SNMP to localhost only by default.

Dangerous SNMP Settings

rwuser noauth — Gives read-write access to the full OID tree with NO authentication whatsoever. Anyone who knows the server is running SNMP can read and modify all device settings. Extremely dangerous.

rwcommunity <community string> <IPv4 address> — Grants full read-write access from a specific IP. If set to a broad subnet or "default" (meaning any host), attackers from those networks can modify device configurations.

rwcommunity6 <community string> <IPv6 address> — Same as above but for IPv6. Both should be checked during enumeration.

Performing SNMP Tasks
SNMPwalk — Query All OIDs

Command:

bash
snmpwalk -v2c -c public 10.129.14.128

Purpose: Walks the entire SNMP OID tree of the target device, retrieving every piece of information the device is willing to share under the specified community string.

Breaking down:

snmpwalk — The SNMP enumeration tool
-v2c — Use SNMP version 2c
-c public — Use "public" as the community string
10.129.14.128 — Target IP address

Example output (key parts):

iso.3.6.1.2.1.1.1.0 = STRING: "Linux htb 5.11.0-34-generic #36~20.04.1-Ubuntu SMP..."
iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
iso.3.6.1.2.1.1.3.0 = Timeticks: (5134) 0:00:51.34
iso.3.6.1.2.1.1.4.0 = STRING: "mrb3n@inlanefreight.htb"
iso.3.6.1.2.1.1.5.0 = STRING: "htb"
iso.3.6.1.2.1.1.6.0 = STRING: "Sitting on the Dock of the Bay"
iso.3.6.1.2.1.1.7.0 = INTEGER: 72
iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/boot/vmlinuz-5.11.0-34-generic root=UUID=..."
iso.3.6.1.2.1.25.6.3.1.2.1232 = STRING: "printer-driver-sag-gdi_0.1-7_all"
iso.3.6.1.2.1.25.6.3.1.2.1243 = STRING: "python3_3.8.2-0ubuntu2_amd64"

What this tells you: The first OID gives the OS and kernel version (Linux Ubuntu). OID .1.4.0 reveals the admin's email (mrb3n@inlanefreight.htb). OID .1.5.0 gives the hostname (htb). OID .1.6.0 gives the physical location. The boot image reveals the exact kernel and root partition UUID. The package list (OID .25.6.3.1.2.xxx) shows every installed software package — extremely valuable for finding vulnerable software versions.

OneSixtyOne — Brute Force Community Strings

Installation:

bash
sudo apt install onesixtyone

Command:

bash
onesixtyone -c /opt/useful/seclists/Discovery/SNMP/snmp.txt 10.129.14.128

Purpose: Tries hundreds or thousands of different community string values against the target SNMP service to find valid ones. The -c flag specifies a wordlist of community strings to try.

Breaking down:

onesixtyone — The brute force tool (named after UDP port 161)
-c /opt/useful/seclists/Discovery/SNMP/snmp.txt — Wordlist of community strings to try
10.129.14.128 — Target IP

Example output:

Scanning 1 hosts, 3220 communities
10.129.14.128 [public] Linux htb 5.11.0-37-generic #41~20.04.2-Ubuntu SMP...

Found "public" as a valid community string. The system description is shown confirming successful access. Now you can use this community string with snmpwalk.

Braa — Fast OID Brute Force

Installation:

bash
sudo apt install braa

Command:

bash
braa public@10.129.14.128:.1.3.6.*

Purpose: Once you have a valid community string, braa rapidly queries large ranges of OIDs to enumerate all available information. It is much faster than snmpwalk for broad queries because it sends many requests simultaneously.

Syntax: braa <community string>@<IP>:<OID pattern>

The * wildcard matches all OIDs starting with .1.3.6. — which covers the entire standard SNMP MIB tree.

Example output:

10.129.14.128:20ms:.1.3.6.1.2.1.1.1.0:Linux htb 5.11.0-34-generic...
10.129.14.128:20ms:.1.3.6.1.2.1.1.2.0:.1.3.6.1.4.1.8072.3.2.10
10.129.14.128:20ms:.1.3.6.1.2.1.1.3.0:548
10.129.14.128:20ms:.1.3.6.1.2.1.1.4.0:mrb3n@inlanefreight.htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.5.0:htb
10.129.14.128:20ms:.1.3.6.1.2.1.1.6.0:US
10.129.14.128:20ms:.1.3.6.1.2.1.1.7.0:78

The response time (20ms) shows how fast braa operates. Results confirm the admin email, hostname, and location.

MySQL
What is MySQL?

MySQL is an open-source relational database management system (RDBMS) developed and now owned by Oracle. It organizes data into tables with rows and columns, uses SQL (Structured Query Language) for all operations, and works on a client-server model. MySQL is the most popular database in web development and forms the "M" in the famous LAMP stack (Linux, Apache, MySQL, PHP). The MySQL server manages data storage and distribution. Clients send SQL queries to create, read, update, or delete data. Data stored in MySQL commonly includes website content, user credentials (hashed passwords), customer information, product catalogs, session data, and application configuration. MySQL typically runs on TCP port 3306.

MySQL Databases and Use Cases

MySQL is ideal for web applications requiring fast reads and writes with structured data. Common examples include WordPress (stores all posts, users, passwords in MySQL), e-commerce platforms (products, orders, customers), CRM systems, and content management systems. In web hosting environments, PHP scripts query MySQL to build dynamic web pages. Passwords should always be stored hashed (using bcrypt, SHA256, etc.) but misconfigurations sometimes lead to plain text storage.

MariaDB

MariaDB is a community-developed fork of MySQL created by the original MySQL developers after Oracle acquired MySQL. It is fully compatible with MySQL but adds extra features and maintains 100% open-source commitment. Many Linux distributions ship MariaDB as the default when you install "MySQL." The SQL commands and connection methods are identical, so everything in this section applies to both.

Default MySQL Configuration

Installation:

bash
sudo apt install mysql-server -y

View configuration:

bash
cat /etc/mysql/mysql.conf.d/mysqld.cnf | grep -v "#" | sed -r '/^\s*$/d'

Purpose: Installs MySQL server and reads the active configuration. Key settings visible:

[client]
port        = 3306
socket      = /var/run/mysqld/mysqld.sock

[mysqld]
user        = mysql
pid-file    = /var/run/mysqld/mysqld.pid
socket      = /var/run/mysqld/mysqld.sock
port        = 3306
basedir     = /usr
datadir     = /var/lib/mysql
tmpdir      = /tmp
lc-messages-dir = /usr/share/mysql
symbolic-links=0

Port 3306 is the standard MySQL port. The user = mysql means the service runs as the "mysql" system user (not root). datadir = /var/lib/mysql is where databases are stored on disk.

Dangerous MySQL Settings

user = root — If MySQL runs as the root system user instead of the mysql user, any vulnerability in MySQL could give an attacker OS-level root access. MySQL should NEVER run as root.

password — If a password is stored in the configuration file in plain text, anyone who can read the file gets database access. Config files should be protected with strict permissions.

admin_address — The IP address MySQL listens on for admin connections. If set to 0.0.0.0, it listens on all interfaces including external ones.

debug — Enables debug logging. Debug logs can contain sensitive SQL queries including queries that contain passwords or personal data.

sql_warnings — Causes INSERT statements to return detailed warning messages. These warnings may contain database schema information that helps an attacker craft SQL injection attacks.

secure_file_priv — Controls MySQL's ability to read and write files on the server. If empty (no restriction), MySQL can read any file the mysql user can access — potentially including /etc/passwd or SSH keys.

Footprinting MySQL with Nmap

Command:

bash
sudo nmap 10.129.14.128 -sV -sC -p3306 --script mysql*

Purpose: Scans MySQL port 3306 with all MySQL-related NSE scripts. The mysql* wildcard runs every Nmap script starting with "mysql".

Example output:

PORT     STATE SERVICE     VERSION
3306/tcp open  nagios-nsca Nagios NSCA
| mysql-brute:
|   Accounts:
|     root:<empty> - Valid credentials
|_  Statistics: Performed 45010 guesses in 5 seconds
|_mysql-databases: ERROR: Script execution failed
| mysql-empty-password:
|_  root account has empty password
| mysql-enum:
|   Valid usernames:
|     root:<empty> - Valid credentials
|     netadmin:<empty> - Valid credentials
|     guest:<empty> - Valid credentials
|     user:<empty> - Valid credentials
|     web:<empty> - Valid credentials
|     sysadmin:<empty> - Valid credentials
|     administrator:<empty> - Valid credentials
|     webadmin:<empty> - Valid credentials
|     admin:<empty> - Valid credentials
|     test:<empty> - Valid credentials
|_  Statistics: Performed 10 guesses in 1 seconds
| mysql-info:
|   Protocol: 10
|   Version: 8.0.26-0ubuntu0.20.04.1
|   Thread ID: 13
|   Capabilities flags: 65535
|   Status: Autocommit
|   Auth Plugin Name: caching_sha2_password

The mysql-empty-password script found that root has no password — critical finding. mysql-enum found 10 valid usernames all with empty passwords. The mysql-info script reveals MySQL version 8.0.26 and the authentication plugin name.

Important note: Some Nmap script results may be false positives. Always manually verify findings.

MySQL Connection and Commands
Connecting Without Password (Checking Access)

Command:

bash
mysql -u root -h 10.129.14.132

Purpose: Attempts to connect to MySQL as the "root" user without a password. If the connection is refused with "Access denied," a password is required. If it connects, the server allows passwordless root login — a critical security issue.

Error response (password required):

ERROR 1045 (28000): Access denied for user 'root'@'10.129.14.1' (using password: NO)
Connecting With Password

Command:

bash
mysql -u root -pP4SSw0rd -h 10.129.14.128

Purpose: Connects to MySQL with a known or discovered password. Note there is NO space between -p and the password.

Successful connection:

Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 150165
Server version: 8.0.27-0ubuntu0.20.04.1 (Ubuntu)

MySQL [(none)]>
show databases — List All Databases

Command:

sql
MySQL [(none)]> show databases;

Purpose: Lists all databases on the MySQL server that the current user has permission to see.

Example output:

+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.006 sec)

Four default databases. information_schema contains metadata about all databases and tables. mysql contains user accounts and privileges. performance_schema contains performance monitoring data. sys contains human-readable views of performance data.

use — Select a Database

Command:

sql
MySQL [(none)]> use mysql;

Purpose: Switches the context to a specific database. All subsequent queries will run against this database until you switch again.

show tables — List Tables

Command:

sql
MySQL [mysql]> show tables;

Purpose: Lists all tables in the currently selected database. The output varies enormously based on which database you are in.

Example output (partial):

+------------------------------------------------------+
| Tables_in_mysql                                      |
+------------------------------------------------------+
| columns_priv                                         |
| db                                                   |
| global_grants                                        |
| password_history                                     |
| user                                                 |
+------------------------------------------------------+
37 rows in set (0.002 sec)

The user table in the mysql database is the most important — it contains all user accounts, their privileges, and password hashes.

select version() — Check MySQL Version

Command:

sql
MySQL [(none)]> select version();

Purpose: Returns the exact MySQL version string. Useful for identifying the precise version to search for known vulnerabilities.

Output:

+-------------------------+
| version()               |
+-------------------------+
| 8.0.27-0ubuntu0.20.04.1 |
+-------------------------+
Querying the sys Database

Commands:

sql
mysql> use sys;
mysql> show tables;
mysql> select host, unique_users from host_summary;

Purpose: The sys schema provides human-readable summaries of performance data. host_summary shows which hosts have connected and how many unique users each host has used.

Example output:

+-------------+--------------+
| host        | unique_users |
+-------------+--------------+
| 10.129.14.1 |            1 |
| localhost   |            2 |
+-------------+--------------+

Shows that one external IP (10.129.14.1) connected with 1 user account, and localhost used 2 different accounts — the penetration tester's connection is visible here.

show columns — View Table Structure

Command:

sql
show columns from user;

Purpose: Displays all columns in a table including column names, data types, whether they allow NULL, and their default values. Understanding table structure helps write proper SELECT queries to extract data.

select * from table — Extract All Data

Command:

sql
select * from user;

Purpose: Retrieves all rows and all columns from the specified table. For the user table in the mysql database, this shows all user accounts, hosts, and password hashes — extremely valuable in a penetration test.

Search for Specific Data

Command:

sql
select * from customers where email = "otto@example.com";

Purpose: Searches a specific table for rows matching a condition. The where clause filters results. This is how you look up specific customer records, user credentials, or any other targeted data during a penetration test.

MSSQL (Microsoft SQL Server)
What is MSSQL?

MSSQL (Microsoft SQL Server) is Microsoft's proprietary closed-source relational database management system. Unlike MySQL which runs on any OS, MSSQL was originally built specifically for Windows and integrates deeply with the Windows ecosystem — Active Directory, Windows Authentication, .NET framework, and IIS. It is extremely popular in enterprise environments that run Windows Server infrastructure. MSSQL communicates on TCP port 1433 by default. One key feature is Windows Authentication — instead of database-level usernames and passwords, users can authenticate using their Windows/Active Directory domain credentials. This integration with Active Directory means compromising an AD account can lead directly to database access, and vice versa.

MSSQL Client Tools

SQL Server Management Studio (SSMS) — The official Microsoft GUI tool for managing MSSQL. Commonly installed on administrator workstations and sometimes on the server itself. If found on a compromised machine with saved credentials, it can lead to direct database access.

Impacket's mssqlclient.py — The most useful tool for penetration testers. A Python command-line client that can connect to MSSQL from Linux. Part of the Impacket toolkit pre-installed on most pentesting distributions.

Finding mssqlclient.py:

bash
locate mssqlclient

Output:

/usr/bin/impacket-mssqlclient
/usr/share/doc/python3-impacket/examples/mssqlclient.py

Other clients include mssql-cli, SQL Server PowerShell, HeidiSQL, and SQLPro.

Default MSSQL System Databases

master — The most important system database. Tracks ALL system-level information for the SQL Server instance including user accounts, configuration settings, and the locations of all other databases. Always present.

model — Acts as a template for every new database created. Any settings or objects placed in model are automatically included in all newly created databases. Customizing model affects all future databases.

msdb — Used by the SQL Server Agent service for scheduling automated jobs, alerts, and maintenance plans. Contains job history and scheduling information. Can reveal what automated tasks are configured.

tempdb — Stores all temporary tables, intermediate results, and workspace objects created during query execution. Recreated from scratch every time SQL Server starts. Contains no persistent data.

resource — A read-only hidden database containing all system objects included with SQL Server. Users cannot modify it. Not listed in normal database listings.

Dangerous MSSQL Configurations

No encryption between client and server — Credentials and query data transmitted in plain text. An attacker on the same network can intercept everything with a packet sniffer.

Self-signed certificates — When encryption IS used but with self-signed certificates, the client cannot verify the server's identity. An attacker can perform a man-in-the-middle attack by presenting their own certificate.

Named pipes enabled — Named pipes are an alternative connection method to TCP. If enabled and poorly secured, they can allow authentication bypasses or be used to relay attacks.

Weak or default sa credentials — The sa (System Administrator) account is the built-in MSSQL superuser. Administrators often forget to disable it or set a strong password. Default or empty sa passwords are a common finding.

Footprinting MSSQL with Nmap

Command:

bash
sudo nmap --script ms-sql-info,ms-sql-empty-password,ms-sql-xp-cmdshell,ms-sql-config,ms-sql-ntlm-info,ms-sql-tables,ms-sql-hasdbaccess,ms-sql-dac,ms-sql-dump-hashes --script-args mssql.instance-port=1433,mssql.username=sa,mssql.password=,mssql.instance-name=MSSQLSERVER -sV -p 1433 10.129.201.248

Purpose: Runs a comprehensive set of MSSQL-specific NSE scripts against the target. Each script checks a different aspect of the MSSQL installation.

Script breakdown:

ms-sql-info — Gets server name, instance name, version, named pipe path, clustering status
ms-sql-empty-password — Checks if sa or other accounts have empty passwords
ms-sql-xp-cmdshell — Tests if the dangerous xp_cmdshell stored procedure (allows OS command execution) is enabled
ms-sql-config — Retrieves configuration settings
ms-sql-ntlm-info — Gets NTLM information from the authentication challenge
ms-sql-tables — Lists tables in accessible databases
ms-sql-hasdbaccess — Checks which databases the user has access to
ms-sql-dac — Checks for the Dedicated Administrator Connection
ms-sql-dump-hashes — Attempts to dump password hashes

Example output:

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info:
|   Target_Name: SQL-01
|   NetBIOS_Domain_Name: SQL-01
|   NetBIOS_Computer_Name: SQL-01
|   DNS_Domain_Name: SQL-01
|   DNS_Computer_Name: SQL-01
|_  Product_Version: 10.0.17763
| ms-sql-info:
|   Windows server name: SQL-01
|   10.129.201.248\MSSQLSERVER:
|     Instance name: MSSQLSERVER
|     Version: Microsoft SQL Server 2019 RTM
|     number: 15.00.2000.00
|     TCP port: 1433
|     Named pipe: \\10.129.201.248\pipe\sql\query
|_    Clustered: false

This reveals the server hostname (SQL-01), it is not clustered, SQL Server 2019 version, and the named pipe path for alternative connections.

Metasploit — MSSQL Ping

Commands:

msf6 > use auxiliary/scanner/mssql/mssql_ping
msf6 auxiliary(scanner/mssql/mssql_ping) > set rhosts 10.129.201.248
msf6 auxiliary(scanner/mssql/mssql_ping) > run

Purpose: The mssql_ping Metasploit module sends special MSSQL discovery packets to the target and retrieves key information. It uses the MSSQL browser service protocol to enumerate SQL Server instances.

Example output:

[*] 10.129.201.248:       - SQL Server information for 10.129.201.248:
[+] 10.129.201.248:       -    ServerName      = SQL-01
[+] 10.129.201.248:       -    InstanceName    = MSSQLSERVER
[+] 10.129.201.248:       -    IsClustered     = No
[+] 10.129.201.248:       -    Version         = 15.0.2000.5
[+] 10.129.201.248:       -    tcp             = 1433
[+] 10.129.201.248:       -    np              = \\SQL-01\pipe\sql\query
[*] Scanned 1 of 1 hosts (100% complete)

Confirms the server name, instance name, version (15.0.2000.5 = SQL Server 2019), TCP port, and named pipe path.

Connecting with mssqlclient.py

Command:

bash
python3 mssqlclient.py Administrator@10.129.201.248 -windows-auth

Purpose: Connects to the MSSQL server using Windows Authentication (Active Directory domain credentials). The -windows-auth flag tells the client to use NTLM/Kerberos authentication instead of SQL Server authentication.

Connection process:

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password: [enter password here]
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] INFO(SQL-01): Line 1: Changed database context to 'master'.
[*] ACK: Result: 1 - Microsoft SQL Server (150 7208)
[!] Press help for extra shell commands

SQL>

The server automatically switches to TLS encryption. You are connected to the master database by default.

List All Databases

Command:

sql
SQL> select name from sys.databases

Purpose: Queries the sys.databases system view to list all databases on the SQL Server instance. This is the MSSQL equivalent of show databases; in MySQL.

Example output:

name
----
master
tempdb
model
msdb
Transactions

The first four are default system databases. "Transactions" is a custom database that was added by the organization — a prime target for investigation.

Oracle TNS
What is Oracle TNS?

Oracle TNS (Transparent Network Substrate) is Oracle's proprietary communication protocol for connecting Oracle database clients and applications to Oracle database servers over a network. It supports multiple networking protocols including TCP/IP and IPX/SPX. TNS provides built-in encryption for data in transit, making it suitable for enterprise environments with strict data security requirements. It is widely used in healthcare, finance, and retail industries managing large, complex Oracle databases. The TNS listener is the server-side component that accepts incoming connection requests and routes them to the correct database instance. It runs on TCP port 1521 by default.

Oracle TNS Configuration Files

Two key configuration files manage TNS:

tnsnames.ora — Client-side configuration file. Maps service names (easy to remember names) to network addresses (IP + port + database identifier). When a client application specifies a connection string like "ORCL", the tnsnames.ora file tells it where to find that service on the network.

listener.ora — Server-side configuration file. Defines the TNS listener's properties — which ports it listens on, which database instances it services, and security settings. Located in $ORACLE_HOME/network/admin/.

Example tnsnames.ora
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

This defines a service called "ORCL" at IP 10.129.11.102 on port 1521 using TCP, connecting to the database instance named "orcl" with a dedicated server connection.

Example listener.ora
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

The listener handles connections for the PDB1 instance and listens on both TCP port 1521 and an IPC (Inter-Process Communication) socket.

Oracle SID

The SID (System Identifier) is a unique name that identifies a specific Oracle database instance on the server. A single Oracle server can run multiple database instances, each with its own SID. When connecting, clients must specify the correct SID to reach the right database. Common default SIDs include "ORCL", "XE" (Express Edition), and "PROD". If the SID is unknown, it must be discovered through enumeration or brute forcing.

Setting Up ODAT

Command:

bash
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

Purpose: Installs all dependencies and ODAT (Oracle Database Attacking Tool). ODAT is a comprehensive penetration testing tool for Oracle databases that can enumerate users, brute force credentials, test for vulnerabilities, and exploit misconfigurations. Each step installs required libraries and Python packages.

Verify installation:

bash
./odat.py -h

Shows the help menu with all available modules if installation was successful.

Footprinting Oracle TNS with Nmap
Basic TNS Scan

Command:

bash
sudo nmap -p1521 -sV 10.129.204.235 --open

Purpose: Scans port 1521 for the Oracle TNS listener. The --open flag shows only open ports.

Example output:

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)

Confirms the Oracle TNS listener is running version 11.2.0.2.0. The "(unauthorized)" means no authentication is required to query the listener — the listener itself is publicly accessible.

Nmap SID Brute Force

Command:

bash
sudo nmap -p1521 -sV 10.129.204.235 --open --script oracle-sid-brute

Purpose: Uses the oracle-sid-brute NSE script to try common SID names against the Oracle TNS listener to discover valid database instance identifiers.

Example output:

PORT     STATE SERVICE    VERSION
1521/tcp open  oracle-tns Oracle TNS listener 11.2.0.2.0 (unauthorized)
| oracle-sid-brute:
|_  XE

Found SID "XE" (Oracle Express Edition). Now you can try to connect to this specific database instance.

ODAT — Full Enumeration

Command:

bash
./odat.py all -s 10.129.204.235

Purpose: Runs ALL ODAT modules against the target Oracle server. This attempts credential discovery, SID enumeration, user enumeration, vulnerability checks, and more.

Key output:

[+] Checking if target 10.129.204.235:1521 is well configured for a connection...
[+] According to a test, the TNS listener 10.129.204.235:1521 is well configured.

[!] Notice: 'mdsys' account is locked, so skipping...
[!] Notice: 'oracle_ocm' account is locked, so skipping...
[+] Valid credentials found: scott/tiger. Continue...

ODAT found valid credentials: username scott with password tiger. These are famous default Oracle credentials that have been known for decades but are still found in the wild.

Installing SQLplus

Commands:

bash
sudo apt update
sudo apt upgrade parrot-core
sudo apt install oracle-instantclient-sqlplus

Purpose: Installs the Oracle SQLplus client on Parrot Linux. SQLplus is Oracle's official command-line database client tool. After installation, verify it works:

bash
sqlplus -v

Output:

SQL*Plus: Release 19.0.0.0.0 - Production
Version 19.6.0.0.0

Fixing library error (if it occurs):

bash
sudo sh -c "echo /usr/lib/oracle/12.2/client64/lib > /etc/ld.so.conf.d/oracle-instantclient.conf"
sudo ldconfig

This tells the system where to find the Oracle shared libraries if SQLplus cannot find them automatically.

Connecting to Oracle with SQLplus

Command:

bash
sqlplus scott/tiger@10.129.204.235/XE

Purpose: Connects to the Oracle database using the discovered credentials. Format is username/password@server_IP/SID.

Successful connection:

SQL*Plus: Release 19.0.0.0.0 - Production

ERROR:
ORA-28002: the password will expire within 7 days

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>

Even with a password expiry warning, you are connected. The database is Oracle 11g Express Edition.

Oracle SQL Commands
List All Tables

Command:

sql
SQL> select table_name from all_tables;

Purpose: Lists all tables visible to the current user across all schemas. The all_tables view includes tables the user owns plus tables they have been granted access to.

Partial output:

TABLE_NAME
------------------------------
DUAL
SYSTEM_PRIVILEGE_MAP
TABLE_PRIVILEGE_MAP
HELP
...
Check User Privileges

Command:

sql
SQL> select * from user_role_privs;

Purpose: Shows all roles granted to the current user. Roles bundle multiple privileges together.

Example output:

USERNAME     GRANTED_ROLE    ADM DEF OS_
------------ --------------- --- --- ---
SCOTT        CONNECT         NO  YES NO
SCOTT        RESOURCE        NO  YES NO

Scott has CONNECT (can log in) and RESOURCE (can create objects) roles but no DBA privileges.

Connect as SYSDBA (Database Administrator)

Command:

bash
sqlplus scott/tiger@10.129.204.235/XE as sysdba

Purpose: Attempts to connect with DBA privileges. If scott has been granted the SYSDBA system privilege, this elevates access to full database administrator level. This is similar to sudo in Linux — you use a regular account to gain elevated database privileges.

Successful connection:

Connected to:
Oracle Database 11g Express Edition Release 11.2.0.2.0 - 64bit Production

SQL>
Verify Elevated Privileges

Command:

sql
SQL> select * from user_role_privs;

Output as SYSDBA:

USERNAME  GRANTED_ROLE                   ADM DEF OS_
--------- ------------------------------ --- --- ---
SYS       ADM_PARALLEL_EXECUTE_TASK      YES YES NO
SYS       APEX_ADMINISTRATOR_ROLE        YES YES NO
SYS       AQ_ADMINISTRATOR_ROLE          YES YES NO
SYS       DBA                            YES YES NO
SYS       DBFS_ROLE                      YES YES NO

Now connected as SYS with full DBA privileges and many administrative roles. Complete database control achieved.

Extract Password Hashes

Command:

sql
SQL> select name, password from sys.user$;

Purpose: Queries the internal Oracle user table to retrieve usernames and their password hashes. These hashes can be taken offline and cracked using tools like Hashcat.

Example output:

NAME           PASSWORD
-------------- ------------------------------
SYS            FBA343E7D6C8BC9D
PUBLIC
CONNECT
RESOURCE
SYSTEM         B5073FE1DE351687
OUTLN          4A3BA55E08595C81

SYS and SYSTEM have DES-encrypted password hashes. These old Oracle DES hashes are weak and can be cracked quickly. PUBLIC, CONNECT, RESOURCE are built-in roles with no passwords.

Upload a File via ODAT (Web Shell)

Step 1 — Create test file:

bash
echo "Oracle File Upload Test" > testing.txt

Step 2 — Upload via ODAT:

bash
./odat.py utlfile -s 10.129.204.235 -d XE -U scott -P tiger --sysdba --putFile C:\\inetpub\\wwwroot testing.txt ./testing.txt

Purpose: Uses ODAT's UTL_FILE module to upload a local file to the server's web root directory. If the Oracle server also runs IIS (Windows web server), uploading to C:\inetpub\wwwroot places the file in the publicly accessible web directory.

Breaking down:

utlfile — ODAT module that uses Oracle's UTL_FILE package for file operations
-s 10.129.204.235 — Target server
-d XE — Target database SID
-U scott -P tiger — Credentials
--sysdba — Connect as SYSDBA for elevated file access
--putFile C:\\inetpub\\wwwroot — Destination directory on the server
testing.txt — Destination filename
./testing.txt — Local source file

Output:

[+] The ./testing.txt file was created on the C:\inetpub\wwwroot directory on the 10.129.204.235 server

Step 3 — Verify the upload:

bash
curl -X GET http://10.129.204.235/testing.txt

Output:

Oracle File Upload Test

The file is accessible via HTTP. This confirms you can upload files through Oracle to the web server. The next step in a real penetration test would be uploading a web shell (ASPX/PHP) instead of a text file, enabling remote command execution.

IPMI (Intelligent Platform Management Interface)
What is IPMI?

IPMI is a hardware-level remote management standard that lets administrators monitor and control servers completely independently of the operating system, BIOS, CPU, and firmware. Even if a server is completely powered off, has a crashed OS, or is in an otherwise unresponsive state, IPMI can still be used to power it on or off, check hardware sensor data (temperature, fan speed, voltage), view the system event log, access a remote console, and even reinstall the OS. IPMI works through a dedicated piece of hardware called the BMC (Baseboard Management Controller) — a small ARM processor directly connected to the server's motherboard with its own power supply, network connection, and memory. IPMI communicates over UDP port 623 and was first published by Intel in 1998. Today it is supported by over 200 vendors.

IPMI Components

BMC (Baseboard Management Controller) — The core component. A microcontroller embedded on the server motherboard. It operates independently of the main CPU and OS. Common implementations: HP iLO, Dell iDRAC, Supermicro IPMI.

ICMB (Intelligent Chassis Management Bus) — Allows communication between multiple server chassis, enabling management of chassis components like power supplies and cooling fans.

IPMB (Intelligent Platform Management Bus) — Extends the BMC's reach to other hardware components on the motherboard using an I2C-based bus.

IPMI Memory — Non-volatile storage for the System Event Log (SEL), sensor data repository, and field replacement unit information.

Communications Interfaces — Multiple ways to access the BMC: local system interface (KCS, SMIC, BT), serial interface, LAN interface (port 623 UDP), and ICMB interface.

Default BMC Credentials

Many organizations never change default BMC credentials, making them easy targets:

Dell iDRAC — Username: root, Password: calvin

HP iLO — Username: Administrator, Password: randomly generated 8-character string of numbers and uppercase letters (usually printed on a label on the server)

Supermicro IPMI — Username: ADMIN, Password: ADMIN

These should always be tried first during a penetration test against any BMC. If a factory default password is found, it often indicates the BMC has never been properly hardened.

Footprinting IPMI with Nmap

Command:

bash
sudo nmap -sU --script ipmi-version -p 623 ilo.inlanfreight.local

Purpose: Uses a UDP scan to detect the IPMI service on port 623. The ipmi-version NSE script specifically identifies the IPMI version and supported authentication methods.

Breaking down:

sudo — Required for UDP scanning
-sU — UDP scan (IPMI uses UDP not TCP)
--script ipmi-version — Run the IPMI version detection script
-p 623 — Scan only port 623
ilo.inlanfreight.local — Target hostname

Example output:

PORT    STATE SERVICE
623/udp open  asf-rmcp
| ipmi-version:
|   Version:
|     IPMI-2.0
|   UserAuth:
|   PassAuth: auth_user, non_null_user
|_  Level: 2.0
MAC Address: 14:03:DC:674:18:6A (Hewlett Packard Enterprise)

Confirms IPMI 2.0 is running. The MAC address prefix identifies it as a Hewlett Packard Enterprise server. auth_user and non_null_user indicate that authentication requires a valid username and non-empty password.

Metasploit — IPMI Version Discovery

Commands:

msf6 > use auxiliary/scanner/ipmi/ipmi_version
msf6 auxiliary(scanner/ipmi/ipmi_version) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_version) > show options
msf6 auxiliary(scanner/ipmi/ipmi_version) > run

Purpose: The Metasploit ipmi_version module sends IPMI discovery requests and reports back the version and authentication capabilities of the target BMC.

Example output:

[*] Sending IPMI requests to 10.129.42.195->10.129.42.195 (1 hosts)
[+] 10.129.42.195:623 - IPMI - IPMI-2.0 UserAuth(auth_msg, auth_user, non_null_user)
PassAuth(password, md5, md2, null) Level(1.5, 2.0)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed

This reveals that the BMC supports both IPMI 1.5 and 2.0, accepts password authentication, MD5, MD2, and even null (no password) authentication. The null authentication support is a finding worth investigating.

RAKP Protocol Vulnerability

The RAKP (Remote Authenticated Key-Exchange Protocol) vulnerability is a critical flaw in IPMI 2.0 that affects all implementations. During the IPMI 2.0 authentication process, the server sends a salted SHA1 or MD5 hash of the user's password TO THE CLIENT before authentication is verified. This is backwards from how secure protocols work. An attacker can capture this hash for ANY valid user account on the BMC without completing the login. The hash can then be cracked offline. There is no patch for this because it is part of the IPMI 2.0 specification itself. The only mitigations are using very strong passwords or network segmentation to restrict BMC access.

Metasploit — Dumping IPMI Hashes

Commands:

msf6 > use auxiliary/scanner/ipmi/ipmi_dumphashes
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > set rhosts 10.129.42.195
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > show options
msf6 auxiliary(scanner/ipmi/ipmi_dumphashes) > run

Purpose: Exploits the RAKP vulnerability to retrieve password hashes from the BMC for all valid user accounts. The module also automatically attempts to crack common passwords from a built-in wordlist.

Module options explained:

CRACK_COMMON true — Automatically attempts to crack hashes against common passwords
OUTPUT_HASHCAT_FILE — Optional path to save hashes in Hashcat format for offline cracking
OUTPUT_JOHN_FILE — Optional path to save hashes in John the Ripper format
PASS_FILE — Wordlist of common IPMI passwords for automatic cracking
USER_FILE — List of usernames to try (default IPMI users: ADMIN, admin, Administrator, root)

Example output:

[+] 10.129.42.195:623 - IPMI - Hash found:
ADMIN:8e160d4802040000205ee9253b6b8dac3052c837e23faa631260719fce740d45c3139a7dd4317b9ea
123456789abcdefa123456789abcdef140541444d494e:a3e82878a09daa8ae3e6c22f9080f8337fe0ed7e

[+] 10.129.42.195:623 - IPMI - Hash for user 'ADMIN' matches password 'ADMIN'
[*] Scanned 1 of 1 hosts (100% complete)

The hash was captured AND cracked immediately — the ADMIN account uses the password "ADMIN" (the default). Now you can log into the BMC's web console with these credentials, gaining full hardware-level access to the server.

Cracking manually with Hashcat (if auto-crack fails):

bash
hashcat -m 7300 ipmi.txt -a 3 ?1?1?1?1?1?1?1?1 -1 ?d?u

This uses Hashcat mode 7300 (IPMI2 RAKP HMAC-SHA1) with a mask attack trying all 8-character combinations of digits and uppercase letters — targeting HP iLO factory default password format.

Linux Remote Management Protocols
SSH (Secure Shell)
What is SSH?

SSH (Secure Shell) provides encrypted remote command-line access to Linux and Unix systems over TCP port 22. It replaced completely insecure older protocols like Telnet and R-Services (rlogin, rsh, rexec) by encrypting all data — including passwords, commands, and output — making network interception useless. SSH-1 was the original version but is vulnerable to MITM attacks. SSH-2 is the modern standard with stronger encryption and no MITM vulnerability. OpenSSH is the most widely used SSH implementation and is pre-installed on virtually every Linux distribution and macOS. SSH also supports tunneling, port forwarding, and X11 forwarding (for graphical applications).

SSH Authentication Methods

SSH supports six different ways to verify identity:

Password Authentication — The simplest method. User types their password. The password is sent encrypted over the SSH connection. Vulnerable to brute force if not rate-limited.

Public-Key Authentication — The most secure and recommended method. A key pair is generated: private key (stays on your machine, protected by a passphrase) and public key (stored on the server in ~/.ssh/authorized_keys). During login, the server challenges you with something only the private key can solve.

Host-Based Authentication — Authentication based on the client machine's identity rather than the user's credentials. Trusts specific hosts implicitly.

Keyboard Authentication — Interactive challenge-response authentication where the server sends prompts and the client responds.

Challenge-Response Authentication — Similar to keyboard authentication but used with external authentication systems.

GSSAPI Authentication — Used with Kerberos and other GSSAPI mechanisms, common in enterprise Active Directory environments.

Public Key Authentication — How It Works
You generate a key pair: ssh-keygen -t rsa -b 4096
Your public key (~/.ssh/id_rsa.pub) is added to the server's ~/.ssh/authorized_keys
Your private key (~/.ssh/id_rsa) stays on your machine, protected by a passphrase
When connecting, the server encrypts a random challenge using your public key
Your SSH client decrypts it with the private key and sends back the solution
The server verifies the solution — if correct, access is granted
Your passphrase is never sent over the network — only used locally to unlock the private key
Default SSH Configuration

Command:

bash
cat /etc/ssh/sshd_config | grep -v "#" | sed -r '/^\s*$/d'

Purpose: Reads the OpenSSH server configuration file filtering out comments.

Example output:

Include /etc/ssh/sshd_config.d/*.conf
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem       sftp    /usr/lib/openssh/sftp-server

X11Forwarding yes is enabled by default. This had a command injection vulnerability in OpenSSH 7.2p1 in 2016 (CVE-2016-3115). UsePAM yes uses PAM for authentication which provides flexibility. Subsystem sftp enables the SFTP file transfer subsystem.

Dangerous SSH Settings

PasswordAuthentication yes — Allows password-based login. Enables brute force attacks. Should be disabled in favor of public key authentication only.

PermitEmptyPasswords yes — Allows login with absolutely no password. Any account with an empty password can be accessed by anyone. Critical vulnerability.

PermitRootLogin yes — Allows the root superuser to log in directly via SSH. Even with a strong password, this is dangerous because it provides direct path to full system compromise. Should always be set to no or prohibit-password.

Protocol 1 — Forces use of the older, vulnerable SSH-1 protocol which is susceptible to MITM attacks. Should never appear in modern configurations.

X11Forwarding yes — Allows forwarding graphical applications. Has had vulnerabilities in the past and increases attack surface.

AllowTcpForwarding yes — Allows TCP port forwarding through the SSH tunnel. Attackers who gain SSH access can use this to tunnel traffic to internal network resources.

PermitTunnel — Allows VPN-like tunneling through SSH. Can be used to bypass firewall rules.

DebianBanner yes — Displays a detailed system banner on login, revealing OS information to attackers.

Footprinting SSH with ssh-audit

Installation and usage:

bash
git clone https://github.com/jtesta/ssh-audit.git && cd ssh-audit
./ssh-audit.py 10.129.14.132

Purpose: ssh-audit analyzes the SSH server's configuration and cryptographic settings without requiring authentication. It checks the SSH version, supported key exchange algorithms, host key algorithms, encryption ciphers, and message authentication codes, flagging any that are weak or deprecated.

Example output:

# general
(gen) banner: SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.3
(gen) software: OpenSSH 8.2p1
(gen) compatibility: OpenSSH 7.4+, Dropbear SSH 2018.76+
(gen) compression: enabled (zlib@openssh.com)

# key exchange algorithms
(kex) curve25519-sha256           -- [info] available since OpenSSH 7.4
(kex) ecdh-sha2-nistp256          -- [fail] using weak elliptic curves
(kex) ecdh-sha2-nistp384          -- [fail] using weak elliptic curves
(kex) diffie-hellman-group-exchange-sha256 (2048-bit) -- [info] available

# host-key algorithms
(key) rsa-sha2-512 (3072-bit)     -- [info] available since OpenSSH 7.2
(key) ssh-rsa (3072-bit)          -- [fail] using weak hashing algorithm
(key) ecdsa-sha2-nistp256         -- [fail] using weak elliptic curves
(key) ssh-ed25519                 -- [info] available since OpenSSH 6.5

The banner reveals the exact OpenSSH version (8.2p1) and Ubuntu version. Several key exchange and host key algorithms are flagged as weak (NIST elliptic curves, RSA-SHA1) and should be disabled.

Verbose SSH Connection — Checking Authentication Methods

Command:

bash
ssh -v cry0l1t3@10.129.14.132

Purpose: The -v (verbose) flag shows the full SSH handshake including which authentication methods the server advertises as available. This tells you what attack methods are possible.

Key output line:

debug1: Authentications that can continue: publickey,password,keyboard-interactive

This shows the server accepts three authentication methods. Password authentication being listed means brute force is possible.

Force Specific Authentication Method

Command:

bash
ssh -v cry0l1t3@10.129.14.132 -o PreferredAuthentications=password

Purpose: Forces the SSH client to only try password authentication, even if public key is available. The -o PreferredAuthentications=password sets the authentication method preference. Useful when you want to test password authentication specifically or when performing brute force testing.

Output:

debug1: Authentications that can continue: publickey,password,keyboard-interactive
debug1: Next authentication method: password
cry0l1t3@10.129.14.132's password:

Now the client skips public key and goes straight to password prompt.

Rsync
What is Rsync?

Rsync is a powerful file synchronization tool that uses a delta-transfer algorithm — it only transfers the parts of files that have actually changed rather than whole files every time. This makes it extremely efficient for backups and keeping remote directories synchronized. Rsync uses port 873 by default but can be tunneled over SSH for encryption. During penetration tests, misconfigured Rsync servers may allow unauthenticated access to shared directories containing sensitive files.

Scanning for Rsync

Command:

bash
sudo nmap -sV -p 873 127.0.0.1

Purpose: Checks if the Rsync service is running on port 873.

Example output:

PORT    STATE SERVICE VERSION
873/tcp open  rsync   (protocol version 31)

Rsync is running protocol version 31.

Probing for Available Shares with Netcat

Command:

bash
nc -nv 127.0.0.1 873

Purpose: Opens a raw TCP connection to the Rsync port using netcat. After connecting, you can type #list to see all available Rsync modules (shares).

Full interaction:

(UNKNOWN) [127.0.0.1] 873 (rsync) open
@RSYNCD: 31.0
@RSYNCD: 31.0
#list
dev             Dev Tools
@RSYNCD: EXIT

The server shows one module called "dev" with description "Dev Tools". This is an accessible Rsync share that you can investigate further.

Enumerating Files in an Rsync Share

Command:

bash
rsync -av --list-only rsync://127.0.0.1/dev

Purpose: Lists all files in the "dev" Rsync module without actually downloading them.

Breaking down:

rsync — The sync tool
-a — Archive mode: preserves permissions, timestamps, symbolic links, etc.
-v — Verbose output
--list-only — Only list files, do not transfer anything
rsync://127.0.0.1/dev — The Rsync URL format: rsync://server/module

Example output:

receiving incremental file list
drwxr-xr-x             48 2022/09/19 09:43:10 .
-rw-r--r--              0 2022/09/19 09:34:50 build.sh
-rw-r--r--              0 2022/09/19 09:36:02 secrets.yaml
drwx------             54 2022/09/19 09:43:10 .ssh

sent 25 bytes  received 221 bytes  492.00 bytes/sec

There is a secrets.yaml file and an .ssh directory (likely containing SSH keys). These are immediate targets.

Downloading All Files from Rsync Share

Command:

bash
rsync -av rsync://127.0.0.1/dev ./local-copy/

Purpose: Downloads the entire "dev" module to a local directory called "local-copy". Without --list-only, rsync actually transfers everything.

Rsync Over SSH

Command:

bash
rsync -av -e ssh rsync://127.0.0.1/dev ./local-copy/
# Or for non-standard SSH port:
rsync -av -e "ssh -p2222" rsync://127.0.0.1/dev ./local-copy/

Purpose: The -e ssh flag tells rsync to use SSH as the transport layer, encrypting all data in transit. If the Rsync server requires SSH authentication, this is how you provide it.

R-Services (Legacy Insecure Remote Access)
What are R-Services?

R-Services are a collection of old Unix remote access protocols developed at UC Berkeley in the 1980s. They predate SSH and were designed when security was less of a concern. They transmit ALL data including passwords in plain text and rely on trusting certain hosts rather than strong cryptographic authentication. They have been almost completely replaced by SSH but are occasionally still found in legacy enterprise environments, Solaris, HP-UX, and AIX systems. R-services span ports 512 (rexec), 513 (rlogin), and 514 (rsh/rcp).

R-Commands Overview

rcp (Remote Copy) — Copies files bidirectionally between local and remote hosts. Works like cp but over a network. Does NOT warn about overwriting existing files. Uses port 514 TCP via the rshd daemon.

rsh (Remote Shell) — Opens a shell on a remote machine without going through a login procedure. Authentication is bypassed using trusted host lists. Uses port 514 TCP via rshd.

rexec (Remote Execute) — Runs shell commands on a remote machine. Unlike rsh, it requires username and password — BUT sends them in plain text over an unencrypted connection. Uses port 512 TCP via rexecd.

rlogin (Remote Login) — Logs into a remote Unix host over the network — similar to telnet but Unix-specific. Authentication can be bypassed via trusted host files. Uses port 513 TCP via rlogind.

Trust Files

R-services use two files to define trusted access:

/etc/hosts.equiv — System-wide trust file. Lists trusted hostnames and usernames. Format: <hostname> <username>. Any combination listed here can access the system without a password.

~/.rhosts — Per-user trust file in each user's home directory. Similar format but only affects that specific user.

Viewing /etc/hosts.equiv:

bash
cat /etc/hosts.equiv
# Output:
# pwnbox cry0l1t3

This trusts the user "cry0l1t3" coming from the host "pwnbox" — no password needed.

Example .rhosts file:

htb-student     10.0.17.5
+               10.0.17.10
+               +

The + is a wildcard meaning "any". + 10.0.17.10 trusts any user from that IP. + + trusts ANYONE from ANYWHERE — completely open access, the worst possible configuration.

Scanning for R-Services

Command:

bash
sudo nmap -sV -p 512,513,514 10.0.17.2

Purpose: Scans all three R-services ports simultaneously to detect which ones are running.

Example output:

PORT    STATE SERVICE    VERSION
512/tcp open  exec?
513/tcp open  login?
514/tcp open  tcpwrapped

All three ports are open suggesting R-services are running on this legacy system.

Logging in with Rlogin

Command:

bash
rlogin 10.0.17.2 -l htb-student

Purpose: Attempts to log in to the remote host as the user "htb-student" using rlogin. If the .rhosts file on the remote host trusts your machine for this user, you log in without a password.

Successful login:

Last login: Fri Dec  2 16:11:21 from localhost
[htb-student@localhost ~]$

No password was required. Access granted through the misconfigured .rhosts trust file.

Rwho — List Active Sessions on Network

Command:

bash
rwho

Purpose: The rwho (Remote Who) command queries the rwho daemon running on systems across the local network to discover all currently logged-in users. It broadcasts UDP queries and collects responses.

Example output:

root     web01:pts/0 Dec  2 21:34
htb-student     workstn01:tty1  Dec  2 19:57  2:25

Shows "root" is logged into web01 (on a pseudo-terminal, meaning SSH/remote session) and "htb-student" is logged into workstn01 on a physical terminal with 2 hours and 25 minutes idle time. This reveals active user sessions across the network.

Rusers — Detailed Network User Information

Command:

bash
rusers -al 10.0.17.5

Purpose: Provides more detailed information than rwho for a specific host. Shows username, hostname, TTY (terminal), login date/time, idle time, and remote host they connected from.

Breaking down:

rusers — The remote users command
-a — Show all users including idle ones
-l — Long format (detailed output)
10.0.17.5 — Target host IP

Example output:

htb-student     10.0.17.5:console          Dec 2 19:57     2:25

Shows htb-student logged in via the physical console (not remote) at 10.0.17.5 on December 2nd, has been idle for 2 hours and 25 minutes.

Windows Remote Management Protocols
RDP (Remote Desktop Protocol)
What is RDP?

RDP (Remote Desktop Protocol) is Microsoft's proprietary protocol for providing full graphical remote access to Windows systems. Unlike SSH which is command-line only, RDP gives you complete control of the Windows desktop — seeing the screen and controlling mouse and keyboard — just as if you were sitting in front of it. RDP uses TCP port 3389 and optionally UDP 3389 for improved performance. All screen data, keyboard input, and mouse movements are encrypted using TLS/SSL since Windows Vista. RDP operates at the application layer of the TCP/IP reference model. It is enabled by default on Windows Server systems and is one of the most commonly exposed services on the internet.

Network Level Authentication (NLA) is an additional security layer that requires users to authenticate BEFORE the full RDP session is established. Without NLA, the Windows login screen is visible even without credentials, which increases the attack surface. NLA is strongly recommended but not always enabled.

Footprinting RDP with Nmap

Command:

bash
nmap -sV -sC 10.129.201.248 -p3389 --script rdp*

Purpose: Scans RDP port 3389 with all RDP-related NSE scripts. The rdp* wildcard runs rdp-enum-encryption, rdp-ntlm-info, and other RDP scripts.

Example output:

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-enum-encryption:
|   Security layer
|     CredSSP (NLA): SUCCESS
|     CredSSP with Early User Auth: SUCCESS
|_    RDSTLS: SUCCESS
| rdp-ntlm-info:
|   Target_Name: ILF-SQL-01
|   NetBIOS_Domain_Name: ILF-SQL-01
|   NetBIOS_Computer_Name: ILF-SQL-01
|   DNS_Domain_Name: ILF-SQL-01
|   DNS_Computer_Name: ILF-SQL-01
|   Product_Version: 10.0.17763
|_  System_Time: 2021-11-06T13:46:00+00:00

The rdp-enum-encryption script shows NLA is supported (CredSSP SUCCESS). The rdp-ntlm-info script reveals the hostname (ILF-SQL-01), domain name (ILF-SQL-01), and Windows version (10.0.17763 = Windows Server 2019).

RDP Packet Trace

Command:

bash
nmap -sV -sC 10.129.201.248 -p3389 --packet-trace --disable-arp-ping -n

Purpose: Adds --packet-trace to show all raw network packets exchanged during the scan. This reveals exactly what data is sent and received at the network level, including the RDP cookies used by Nmap.

Important security note from the output:

NSE: TCP 10.10.14.20:36630 > 10.129.201.248:3389 |
Cooki e: mstshash=nmap

The Nmap RDP scripts use mstshash=nmap as an RDP cookie. Security tools, EDR (Endpoint Detection and Response) systems, and threat hunters specifically look for this string in network traffic to identify Nmap scans. On hardened networks, this could trigger alerts and get you blocked.

RDP Security Check with rdp-sec-check.pl

Installation:

bash
sudo cpan
cpan[1]> install Encoding::BER
git clone https://github.com/CiscoCXSecurity/rdp-sec-check.git && cd rdp-sec-check

Command:

bash
./rdp-sec-check.pl 10.129.201.248

Purpose: A Perl script that checks RDP security configurations without authenticating. It tests which security layers and encryption methods the server supports by sending specially crafted RDP handshake packets.

Example output:

[+] Checking supported protocols
[-] Checking if RDP Security (PROTOCOL_RDP) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if TLS Security (PROTOCOL_SSL) is supported...Not supported - HYBRID_REQUIRED_BY_SERVER
[-] Checking if CredSSP Security (PROTOCOL_HYBRID) is supported [uses NLA]...Supported

[+] Summary of protocol support
[-] 10.129.201.248:3389 supports PROTOCOL_SSL   : FALSE
[-] 10.129.201.248:3389 supports PROTOCOL_HYBRID: TRUE
[-] 10.129.201.248:3389 supports PROTOCOL_RDP   : FALSE

[+] Summary of RDP encryption support
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_NONE   : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_40BIT  : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_128BIT : FALSE
[-] 10.129.201.248:3389 supports ENCRYPTION_METHOD_FIPS   : FALSE

This server only supports PROTOCOL_HYBRID (NLA with CredSSP). Plain RDP and TLS without NLA are both rejected. All legacy encryption methods are disabled. This is a well-hardened RDP configuration.

Connecting via RDP from Linux

Command:

bash
xfreerdp /u:cry0l1t3 /p:"P455w0rd!" /v:10.129.201.248

Purpose: Initiates an RDP session from Linux using the xfreerdp client. Opens a full graphical Windows desktop window.

Breaking down:

xfreerdp — The FreeRDP client for Linux (X11 version)
/u:cry0l1t3 — Username
/p:"P455w0rd!" — Password (in quotes to handle special characters)
/v:10.129.201.248 — Target server IP

During connection:

[WARN] Certificate verification failure 'self signed certificate (18)'
[ERROR] WARNING: CERTIFICATE NAME MISMATCH!
The hostname used for this connection (10.129.201.248:3389)
does not match the name given in the certificate:
Common Name (CN): ILF-SQL-01

Do you trust the above certificate? (Y/T/N)

The certificate mismatch warning appears because you are connecting via IP but the certificate is issued to the hostname "ILF-SQL-01". Type Y to accept and proceed. After successful authentication, a Windows desktop window appears.

WinRM (Windows Remote Management)
What is WinRM?

WinRM (Windows Remote Management) is Microsoft's command-line remote management protocol built on the WS-Management (Web Services Management) standard. It uses SOAP (Simple Object Access Protocol) over HTTP/HTTPS to communicate. WinRM is Microsoft's answer to SSH — it provides command-line remote access to Windows systems. WinRM uses TCP port 5985 (HTTP) and TCP port 5986 (HTTPS). It is enabled by default on Windows Server 2012 and later. WinRS (Windows Remote Shell) is the client-side component that allows executing arbitrary commands on remote systems through WinRM. PowerShell remote sessions also use WinRM as their transport.

Footprinting WinRM with Nmap

Command:

bash
nmap -sV -sC 10.129.201.248 -p5985,5986 --disable-arp-ping -n

Purpose: Scans both WinRM ports. The --disable-arp-ping and -n flags disable ARP ping and DNS resolution to speed up the scan.

Example output:

PORT     STATE SERVICE VERSION
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Port 5985 is open (HTTP WinRM). The server header confirms it is Windows. The "Not Found" response to a basic HTTP request is expected — WinRM uses specific SOAP endpoints, not regular web pages.

Connecting with Evil-WinRM

Command:

bash
evil-winrm -i 10.129.201.248 -u Cry0l1t3 -p P455w0rD!

Purpose: Evil-WinRM is a penetration testing tool designed specifically to interact with WinRM from Linux. It provides a feature-rich PowerShell shell on the remote Windows system.

Breaking down:

evil-winrm — The tool
-i 10.129.201.248 — Target IP address
-u Cry0l1t3 — Username
-p P455w0rD! — Password

Successful connection:

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Cry0l1t3\Documents>

You now have a PowerShell prompt inside the remote Windows system. You can run any PowerShell command, upload/download files, and manage the system as if sitting at it.

WMI (Windows Management Instrumentation)
What is WMI?

WMI (Windows Management Instrumentation) is Microsoft's implementation of WBEM (Web-Based Enterprise Management) and CIM (Common Information Model). It provides read and write access to almost ALL settings and information on Windows systems — hardware, software, OS configuration, running processes, network settings, user accounts, installed applications, and much more. WMI is the most powerful management interface available on Windows. It is accessed through PowerShell, VBScript, the WMIC command-line tool, or directly through API calls. WMI communication starts on TCP port 135 (the DCE/RPC endpoint mapper), then moves to a randomly assigned port for the actual data transfer. This makes WMI harder to block with simple firewall rules.

Footprinting WMI with wmiexec.py

Command:

bash
/usr/share/doc/python3-impacket/examples/wmiexec.py Cry0l1t3:"P455w0rD!"@10.129.201.248 "hostname"

Purpose: wmiexec.py from the Impacket toolkit uses WMI to execute commands on remote Windows systems from Linux. It authenticates using provided credentials and runs the specified command, returning the output.

Breaking down:

/usr/share/doc/python3-impacket/examples/wmiexec.py — Full path to the script
Cry0l1t3:"P455w0rD!"@10.129.201.248 — Credentials and target in format username:password@IP
"hostname" — The Windows command to execute remotely

Example output:

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] SMBv3.0 dialect used
ILF-SQL-01

The command hostname was executed on the remote Windows system and returned "ILF-SQL-01" — confirming successful remote command execution via WMI. In a real penetration test, you would run more powerful commands like whoami /all (check privileges), net user (list users), or ipconfig /all (network configuration).

These are the complete, fully detailed notes for every remaining service — SMB, NFS, DNS, SMTP, IMAP/POP3, SNMP, MySQL, MSSQL, Oracle TNS, IPMI, SSH, Rsync, R-Services, RDP, WinRM, and WMI — with every heading, subheading, command, purpose, usage, and complete demonstration included. Nothing skipped.
