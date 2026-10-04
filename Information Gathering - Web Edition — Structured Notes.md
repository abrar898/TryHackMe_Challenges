# Information Gathering - Web Edition — Structured Notes

## Table of Contents
- [Section 1: Introduction](#section-1-introduction)
  - [1.1 Types of Reconnaissance](#11-types-of-reconnaissance)
  - [1.2 Active Reconnaissance](#12-active-reconnaissance)
  - [1.3 Passive Reconnaissance](#13-passive-reconnaissance)
- [Section 2: WHOIS](#section-2-whois)
  - [2.1 History of WHOIS](#21-history-of-whois)
  - [2.2 Why WHOIS Matters for Web Recon](#22-why-whois-matters-for-web-recon)
- [Section 3: Utilising WHOIS](#section-3-utilising-whois)
  - [3.1 Scenario 1: Phishing Investigation](#31-scenario-1-phishing-investigation)
  - [3.2 Scenario 2: Malware Analysis](#32-scenario-2-malware-analysis)
  - [3.3 Scenario 3: Threat Intelligence Report](#33-scenario-3-threat-intelligence-report)
  - [3.4 Using WHOIS (Practical)](#34-using-whois-practical)
- [Section 4: DNS](#section-4-dns)
  - [4.1 How DNS Works](#41-how-dns-works)
  - [4.2 The Hosts File](#42-the-hosts-file)
  - [4.3 It's Like a Relay Race](#43-its-like-a-relay-race)
  - [4.4 Key DNS Concepts](#44-key-dns-concepts)
  - [4.5 Why DNS Matters for Web Recon](#45-why-dns-matters-for-web-recon)
- [Section 5: Digging DNS](#section-5-digging-dns)
  - [5.1 DNS Tools](#51-dns-tools)
  - [5.2 The Domain Information Groper (dig)](#52-the-domain-information-groper-dig)
  - [5.3 Groping DNS (Practical)](#53-groping-dns-practical)
- [Section 6: Subdomains](#section-6-subdomains)
  - [6.1 Subdomain Enumeration](#61-subdomain-enumeration)
  - [6.2 Active Subdomain Enumeration](#62-active-subdomain-enumeration)
  - [6.3 Passive Subdomain Enumeration](#63-passive-subdomain-enumeration)
- [Section 7: Subdomain Bruteforcing](#section-7-subdomain-bruteforcing)
  - [7.1 The Brute-Force Process](#71-the-brute-force-process)
  - [7.2 Brute-Force Tools](#72-brute-force-tools)
  - [7.3 DNSEnum (Practical)](#73-dnsenum-practical)
- [Section 8: DNS Zone Transfers](#section-8-dns-zone-transfers)
  - [8.1 What is a Zone Transfer](#81-what-is-a-zone-transfer)
  - [8.2 The Zone Transfer Vulnerability](#82-the-zone-transfer-vulnerability)
  - [8.3 Remediation](#83-remediation)
  - [8.4 Exploiting Zone Transfers (Practical)](#84-exploiting-zone-transfers-practical)
- [Section 9: Virtual Hosts](#section-9-virtual-hosts)
  - [9.1 How Virtual Hosts Work: VHosts vs Subdomains](#91-how-virtual-hosts-work-vhosts-vs-subdomains)
  - [9.2 Server VHost Lookup](#92-server-vhost-lookup)
  - [9.3 Types of Virtual Hosting](#93-types-of-virtual-hosting)
  - [9.4 Virtual Host Discovery Tools](#94-virtual-host-discovery-tools)
  - [9.5 Gobuster (Practical)](#95-gobuster-practical)
- [Section 10: Certificate Transparency Logs](#section-10-certificate-transparency-logs)
  - [10.1 What are Certificate Transparency Logs?](#101-what-are-certificate-transparency-logs)
  - [10.2 CT Logs and Web Recon](#102-ct-logs-and-web-recon)
  - [10.3 Searching CT Logs](#103-searching-ct-logs)
  - [10.4 crt.sh Lookup (Practical)](#104-crtsh-lookup-practical)
- [Section 11: Fingerprinting](#section-11-fingerprinting)
  - [11.1 Fingerprinting Techniques](#111-fingerprinting-techniques)
  - [11.2 Banner Grabbing (Practical)](#112-banner-grabbing-practical)
  - [11.3 Wafw00f (Practical)](#113-wafw00f-practical)
  - [11.4 Nikto (Practical)](#114-nikto-practical)
- [Section 12: Crawling](#section-12-crawling)
  - [12.1 How Web Crawlers Work](#121-how-web-crawlers-work)
  - [12.2 Breadth-First Crawling](#122-breadth-first-crawling)
  - [12.3 Depth-First Crawling](#123-depth-first-crawling)
  - [12.4 Extracting Valuable Information](#124-extracting-valuable-information)
  - [12.5 The Importance of Context](#125-the-importance-of-context)
- [Section 13: robots.txt](#section-13-robotstxt)
  - [13.1 What is robots.txt?](#131-what-is-robotstxt)
  - [13.2 How robots.txt Works](#132-how-robotstxt-works)
  - [13.3 Understanding robots.txt Structure](#133-understanding-robotstxt-structure)
  - [13.4 Why Respect robots.txt?](#134-why-respect-robotstxt)
  - [13.5 robots.txt in Web Reconnaissance](#135-robotstxt-in-web-reconnaissance)
  - [13.6 Analyzing robots.txt (Practical)](#136-analyzing-robotstxt-practical)
- [Section 14: Well-Known URIs](#section-14-well-known-uris)
  - [14.1 Web Recon and .well-known](#141-web-recon-and-well-known)
- [Section 15: Creepy Crawlies](#section-15-creepy-crawlies)
  - [15.1 Popular Web Crawlers](#151-popular-web-crawlers)
  - [15.2 Scrapy (Practical)](#152-scrapy-practical)
  - [15.3 ReconSpider (Practical)](#153-reconspider-practical)
  - [15.4 results.json Structure](#154-resultsjson-structure)
- [Section 16: Search Engine Discovery](#section-16-search-engine-discovery)
  - [16.1 Why Search Engine Discovery Matters](#161-why-search-engine-discovery-matters)
  - [16.2 Search Operators](#162-search-operators)
  - [16.3 Google Dorking](#163-google-dorking)
- [Section 17: Web Archives](#section-17-web-archives)
  - [17.1 What is the Wayback Machine?](#171-what-is-the-wayback-machine)
  - [17.2 How Does the Wayback Machine Work?](#172-how-does-the-wayback-machine-work)
  - [17.3 Why the Wayback Machine Matters for Web Reconnaissance](#173-why-the-wayback-machine-matters-for-web-reconnaissance)
  - [17.4 Going Wayback on HTB (Practical)](#174-going-wayback-on-htb-practical)
- [Section 18: Automating Recon](#section-18-automating-recon)
  - [18.1 Why Automate Reconnaissance?](#181-why-automate-reconnaissance)
  - [18.2 Reconnaissance Frameworks](#182-reconnaissance-frameworks)
  - [18.3 FinalRecon (Practical)](#183-finalrecon-practical)
- [Cheat Sheet](#cheat-sheet)

---

## Section 1: Introduction

**Web Reconnaissance** is the foundation of a thorough security assessment — the systematic, meticulous collection of information about a target website or web application before deeper analysis or exploitation begins. It forms a critical part of the "Information Gathering" phase within the broader Penetration Testing Process (Pre-Engagement → Information Gathering → Vulnerability Assessment → Exploitation → Post-Exploitation → Lateral Movement → Proof-of-Concept → Post-Engagement).

The primary goals of web reconnaissance are:
- **Identifying Assets** — uncovering publicly accessible components (web pages, subdomains, IPs, technologies) to build a comprehensive view of the target's online presence.
- **Discovering Hidden Information** — locating accidentally exposed sensitive material like backup files, configs, or internal documentation that reveal potential entry points.
- **Analysing the Attack Surface** — assessing technologies, configurations, and possible exploitation entry points.
- **Gathering Intelligence** — collecting data (key personnel, emails, behavior patterns) useful for further exploitation or social engineering.

Attackers use this information to tailor attacks and bypass defenses; defenders use the same recon process proactively to find and patch weaknesses before attackers can exploit them.

### 1.1 Types of Reconnaissance
Web reconnaissance splits into two fundamental methodologies — **active** and **passive** — each with distinct trade-offs between thoroughness and stealth. Understanding when to use each is essential to effective information gathering, since active methods are more direct but riskier, while passive methods are safer but may be less complete.

### 1.2 Active Reconnaissance
In **active reconnaissance**, the tester directly interacts with the target system to gather information, which provides a more comprehensive view of the target's infrastructure but carries a higher risk of detection by security monitoring.

| Technique | Description | Example | Tools | Risk of Detection |
|---|---|---|---|---|
| Port Scanning | Identifying open ports and running services. | Using Nmap to scan for ports 80/443. | Nmap, Masscan, Unicornscan | High — direct interaction can trigger IDS/firewalls. |
| Vulnerability Scanning | Probing for known vulnerabilities (outdated software, misconfigs). | Running Nessus to check for SQLi/XSS. | Nessus, OpenVAS, Nikto | High — scanners send detectable exploit payloads. |
| Network Mapping | Mapping network topology and connected devices. | Using traceroute to reveal network hops. | Traceroute, Nmap | Medium to High — unusual traffic volume raises suspicion. |
| Banner Grabbing | Retrieving service banners. | Checking an HTTP banner for server software/version. | Netcat, curl | Low — minimal interaction, but can be logged. |
| OS Fingerprinting | Identifying the target's OS. | Nmap's `-O` flag. | Nmap, Xprobe2 | Low — usually passive, but advanced techniques can be detected. |
| Service Enumeration | Determining specific service versions. | Nmap's `-sV` flag. | Nmap | Low — can be logged but rarely triggers alerts. |
| Web Spidering | Crawling the site to map pages/directories/files. | Burp Suite Spider or ZAP Spider. | Burp Suite Spider, OWASP ZAP Spider, Scrapy | Low to Medium — detectable if crawler behavior isn't configured to mimic legitimate traffic. |

### 1.3 Passive Reconnaissance
In **passive reconnaissance**, information is gathered **without** directly interacting with the target, relying instead on publicly available information and resources. This is generally stealthier and less likely to trigger alarms, though it may yield less comprehensive results since it's limited to what's already publicly accessible.

| Technique | Description | Example | Tools | Risk of Detection |
|---|---|---|---|---|
| Search Engine Queries | Using search engines to uncover target info. | Searching "[Target Name] employees" on Google. | Google, DuckDuckGo, Bing, Shodan | Very Low — normal internet activity. |
| WHOIS Lookups | Querying WHOIS databases for domain registration details. | WHOIS lookup for registrant/contact info. | `whois` CLI tool, online lookups | Very Low — legitimate, non-suspicious queries. |
| DNS | Analysing DNS records for subdomains, mail servers, etc. | Using `dig` to enumerate subdomains. | dig, nslookup, host, dnsenum, fierce, dnsrecon | Very Low — essential, routine internet traffic. |
| Web Archive Analysis | Examining historical website snapshots. | Using the Wayback Machine. | Wayback Machine | Very Low — normal activity. |
| Social Media Analysis | Gathering info from social platforms. | Searching LinkedIn for target employees. | LinkedIn, Twitter, Facebook, OSINT tools | Very Low — accessing public profiles isn't intrusive. |
| Code Repositories | Analysing public repos for exposed credentials/vulnerabilities. | Searching GitHub for target-related code. | GitHub, GitLab | Very Low — public repos are meant for public access. |

The module notes it will begin exploring these tools and techniques starting with **WHOIS**, since understanding the WHOIS protocol is a gateway to domain registration, ownership, and infrastructure information that underpins more advanced recon methods later.

---

## Section 2: WHOIS

**WHOIS** is a widely used query/response protocol for accessing databases storing information about registered internet resources — primarily domain names, but also IP address blocks and autonomous systems. It functions essentially like a giant internet phonebook, letting you look up who owns or is responsible for an online asset.

**Command explained:**
```shellsession
whois inlanefreight.com
```
Returns a record typically including:
- **Domain Name** — the domain itself (e.g., `example.com`).
- **Registrar** — the company where the domain was registered (e.g., GoDaddy, Namecheap).
- **Registrant Contact** — the person/organization that registered the domain.
- **Administrative Contact** — the person managing the domain.
- **Technical Contact** — the person handling technical issues.
- **Creation and Expiration Dates** — registration and expiry timestamps.
- **Name Servers** — servers that resolve the domain name to an IP address.

### 2.1 History of WHOIS
WHOIS traces back to **Elizabeth Feinler** and her team at the Stanford Research Institute's Network Information Center (NIC) in the 1970s, who built it to track the growing number of ARPANET network resources. It was formalized in **RFC 812** (1982) by Ken Harrenstien and Vic White, establishing a structured query-response system. As the internet grew, the centralized model became inadequate, leading to the rise of **Regional Internet Registries (RIRs)** in the 1990s (with contributions from Randy Bush and John Postel), distributing WHOIS management regionally for better scalability. In 1998, **ICANN** (co-founded with help from Vint Cerf) took over global DNS/WHOIS policy, standardizing data formats and introducing dispute-resolution frameworks like the **UDRP** for cybersquatting and trademark conflicts. More recently, privacy concerns — especially following the **GDPR** (2018) — have driven the adoption of privacy-masking services and the development of the more privacy-conscious **Registration Data Access Protocol (RDAP)** as WHOIS continues to evolve.

### 2.2 Why WHOIS Matters for Web Recon
WHOIS data is valuable during the reconnaissance phase for several reasons:
- **Identifying Key Personnel** — names, emails, and phone numbers of domain administrators can be leveraged for social engineering or phishing targeting.
- **Discovering Network Infrastructure** — name servers and IP details hint at the target's network setup, revealing potential entry points or misconfigurations.
- **Historical Data Analysis** — services like WhoisFreaks can show historical WHOIS changes (ownership, contact info) over time, useful for tracking a target's digital evolution.

---

## Section 3: Utilising WHOIS

This section illustrates WHOIS's practical investigative value through three real-world-style scenarios before moving into hands-on use of the `whois` command.

### 3.1 Scenario 1: Phishing Investigation
A suspicious email claiming to be from a company's bank prompts a security analyst to run a WHOIS lookup on the linked domain. The record reveals: a **very recent registration date**, **registrant information hidden** behind a privacy service, and **name servers tied to a known bulletproof hosting provider**. Combined, these are strong phishing red flags, prompting the analyst to block the domain and warn employees. Further investigation into the hosting provider's other IPs/domains could reveal more infrastructure used by the same threat actor.

### 3.2 Scenario 2: Malware Analysis
A researcher analysing malware that communicates with a command-and-control (C2) server runs a WHOIS lookup on the C2 domain. The record shows: a registrant using an **anonymous free email service**, an address in a **country with high cybercrime prevalence**, and a registrar with a **history of lax abuse policies**. This leads the researcher to conclude the C2 server is likely hosted on a compromised or "bulletproof" host, and the WHOIS data helps identify and notify the hosting provider.

### 3.3 Scenario 3: Threat Intelligence Report
A cybersecurity firm tracking a threat actor group compiles WHOIS data across multiple campaign-related domains, uncovering patterns: domains **registered in clusters** shortly before attacks, use of **varied aliases/fake identities**, **shared name servers** suggesting common infrastructure, and a **takedown history** indicating past law enforcement action. These patterns feed into a detailed TTP (Tactics, Techniques, Procedures) profile and a set of Indicators of Compromise (IOCs) that other organizations can use defensively.

### 3.4 Using WHOIS (Practical)
**Step-by-step (installing and using whois):**
1. Ensure `whois` is installed:
   ```shellsession
   sudo apt update
   sudo apt install whois -y
   ```
2. Run a lookup against a target domain:
   ```shellsession
   whois facebook.com
   ```

**Command output explained (example: facebook.com):**
- **Domain Registration** — `Registrar: RegistrarSafe, LLC`, `Creation Date: 1997-03-29`, `Registry Expiry Date: 2033-03-30`. A long-active domain with a distant expiry date suggests legitimacy and established presence.
- **Domain Owner** — `Registrant Organization: Meta Platforms, Inc.`, `Registrant Name: Domain Admin`. Identifies the owning organization and point of contact.
- **Domain Status** — flags like `clientDeleteProhibited`, `clientTransferProhibited`, `clientUpdateProhibited`, and their server-side equivalents indicate the domain is locked against unauthorized changes, transfers, or deletions — a sign of strong domain security.
- **Name Servers** — `A.NS.FACEBOOK.COM` through `D.NS.FACEBOOK.COM`, all within the facebook.com domain, showing Meta manages its own DNS infrastructure (common for large organizations wanting full control over DNS resolution).

**Key takeaway:** WHOIS gives domain-level contact/registration info but typically won't directly identify individual employees or specific vulnerabilities — it must be combined with other recon techniques for a complete picture of the target's digital footprint.

---

## Section 4: DNS

The **Domain Name System (DNS)** acts as the internet's "GPS" — translating human-readable domain names (e.g., `www.example.com`) into the numerical IP addresses (e.g., `192.0.2.1`) computers use to communicate, eliminating the need to memorize numeric addresses for every site you want to visit.

### 4.1 How DNS Works
When you type a domain into your browser, the following sequence (a **recursive lookup**) occurs:
1. **DNS Query** — your computer checks its local cache first; if not found, it asks a **DNS resolver** (often provided by your ISP).
2. **Recursive Lookup Begins** — the resolver checks its own cache, then queries a **root name server** if needed.
3. **Root Name Server Points the Way** — doesn't know the exact address but directs the resolver to the correct **Top-Level Domain (TLD) name server** (e.g., for `.com`, `.org`).
4. **TLD Name Server Narrows It Down** — points the resolver to the **authoritative name server** responsible for the specific domain.
5. **Authoritative Name Server Delivers the Address** — holds and returns the actual IP address.
6. **The DNS Resolver Returns the Information** — passes the IP back to your computer and caches it for future requests.
7. **Your Computer Connects** — now able to connect directly to the web server and load the site.

### 4.2 The Hosts File
The **hosts file** is a simple text file that manually maps hostnames to IP addresses, bypassing DNS resolution entirely — useful for development, troubleshooting, or blocking sites.

**Location:**
- Windows: `C:\Windows\System32\drivers\etc\hosts`
- Linux/macOS: `/etc/hosts`

**Format:**
```txt
<IP Address>    <Hostname> [<Alias> ...]
```
Example:
```txt
127.0.0.1       localhost
192.168.1.10    devserver.local
```

**Step-by-step (editing the hosts file):**
1. Open the file with a text editor using administrative/root privileges.
2. Add entries in the `<IP> <Hostname>` format as needed.
3. Save the file — changes apply immediately, no restart required.

**Common uses:**
- Redirecting a domain to a local dev server: `127.0.0.1  myapp.local`
- Testing connectivity to a specific IP: `192.168.1.20  testserver.local`
- Blocking unwanted sites by pointing them to a dead address: `0.0.0.0  unwanted-site.com`

### 4.3 It's Like a Relay Race
The module uses a relay-race metaphor: the domain name starts with your computer, passes to the resolver, then to the root server, then the TLD server, then the authoritative server — each handoff getting closer to the destination — before the final IP address is relayed back down the chain to your computer.

### 4.4 Key DNS Concepts
A **zone** is a distinct part of the domain namespace managed by a specific entity/administrator — e.g., `example.com` and its subdomains typically belong to one zone. The **zone file** is a text file on a DNS server defining the resource records within that zone.

**Example zone file (example.com):**
```dns-zone
$TTL 3600 ; Default Time-To-Live (1 hour)
@       IN SOA   ns1.example.com. admin.example.com. (
                2024060401 ; Serial number (YYYYMMDDNN)
                3600       ; Refresh interval
                900        ; Retry interval
                604800     ; Expire time
                86400 )    ; Minimum TTL

@       IN NS    ns1.example.com.
@       IN NS    ns2.example.com.
@       IN MX 10 mail.example.com.
www     IN A     192.0.2.1
mail    IN A     198.51.100.1
ftp     IN CNAME www.example.com.
```
- `$TTL` — default cache lifetime for records lacking their own TTL.
- `SOA` line — defines the zone's primary name server, admin contact, serial number (for change tracking), and various timing intervals (refresh/retry/expire/minimum TTL).
- `NS` — declares the zone's authoritative name servers.
- `MX 10` — mail server with priority `10` (lower number = higher priority).
- `A`/`CNAME` — map hostnames to IPs or to other hostnames, respectively.

**Core DNS concept glossary:**
| DNS Concept | Description | Example |
|---|---|---|
| Domain Name | A human-readable label for a resource. | `www.example.com` |
| IP Address | A unique numerical device identifier. | `192.0.2.1` |
| DNS Resolver | Translates domain names into IPs. | ISP's server, Google DNS (`8.8.8.8`) |
| Root Name Server | Top-level DNS hierarchy servers. | 13 worldwide, `a.root-servers.net` etc. |
| TLD Name Server | Handles specific TLDs (.com, .org). | Verisign (.com), PIR (.org) |
| Authoritative Name Server | Holds the actual IP for a domain. | Managed by hosting providers/registrars |
| DNS Record Types | Different categories of stored DNS data. | A, AAAA, CNAME, MX, NS, TXT, etc. |

**DNS record types:**
| Record Type | Full Name | Description | Zone File Example |
|---|---|---|---|
| A | Address Record | Maps a hostname to its IPv4 address. | `www.example.com. IN A 192.0.2.1` |
| AAAA | IPv6 Address Record | Maps a hostname to its IPv6 address. | `www.example.com. IN AAAA 2001:db8:85a3::8a2e:370:7334` |
| CNAME | Canonical Name Record | Aliases a hostname to another hostname. | `blog.example.com. IN CNAME webserver.example.net.` |
| MX | Mail Exchange Record | Specifies mail server(s) for the domain. | `example.com. IN MX 10 mail.example.com.` |
| NS | Name Server Record | Delegates a zone to an authoritative server. | `example.com. IN NS ns1.example.com.` |
| TXT | Text Record | Arbitrary text, often for verification/security policies. | `example.com. IN TXT "v=spf1 mx -all"` |
| SOA | Start of Authority Record | Administrative zone info (primary NS, admin email, timers). | `example.com. IN SOA ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400` |
| SRV | Service Record | Defines hostname/port for specific services. | `_sip._udp.example.com. IN SRV 10 5 5060 sipserver.example.com.` |
| PTR | Pointer Record | Reverse DNS lookup — maps an IP to a hostname. | `1.2.0.192.in-addr.arpa. IN PTR www.example.com.` |

The class field **"IN"** stands for "Internet," denoting the standard internet protocol suite — other classes (e.g., CH for Chaosnet, HS for Hesiod) exist but are rarely used today.

### 4.5 Why DNS Matters for Web Recon
DNS is far more than a technical translation protocol — it's a critical infrastructure component that can reveal vulnerabilities and entry points:
- **Uncovering Assets** — subdomains, mail servers, and NS records can be revealed; e.g., a CNAME pointing to an outdated server (`dev.example.com CNAME oldserver.example.net`) could expose a vulnerable system.
- **Mapping the Network Infrastructure** — NS records can reveal the hosting provider, while an A record for `loadbalancer.example.com` can pinpoint a load balancer — helping map system connections and potential choke points/weaknesses.
- **Monitoring for Changes** — tracking DNS over time can reveal new entry points (e.g., a new `vpn.example.com` subdomain appearing) or TXT records hinting at third-party service usage (e.g., `_1password=...`) useful for social engineering/phishing.

---

## Section 5: Digging DNS

This section moves from DNS theory into hands-on tools and techniques for DNS-based web reconnaissance.

### 5.1 DNS Tools
| Tool | Key Features | Use Cases |
|---|---|---|
| dig | Versatile lookup tool supporting many query types and detailed output. | Manual queries, zone transfer attempts, troubleshooting, in-depth analysis. |
| nslookup | Simpler lookup tool, mainly A/AAAA/MX records. | Basic queries, quick domain/mail server checks. |
| host | Streamlined lookup tool with concise output. | Quick A/AAAA/MX checks. |
| dnsenum | Automated enumeration: dictionary attacks, brute-forcing, zone transfers. | Discovering subdomains efficiently. |
| fierce | DNS recon/subdomain enumeration with recursive search, wildcard detection. | User-friendly subdomain/target discovery. |
| dnsrecon | Combines multiple recon techniques, multiple output formats. | Comprehensive enumeration and record gathering. |
| theHarvester | OSINT tool gathering info (including emails) from various sources. | Collecting emails, employee info, and domain-associated data. |
| Online DNS Lookup Services | User-friendly web interfaces for lookups. | Quick checks when CLI tools aren't available. |

### 5.2 The Domain Information Groper (dig)
`dig` (**Domain Information Groper**) is a versatile, powerful DNS query utility known for its flexible, detailed, and customizable output.

**Common dig commands:**
| Command | Description |
|---|---|
| `dig domain.com` | Default A record lookup. |
| `dig domain.com A` | Retrieves the IPv4 (A) record. |
| `dig domain.com AAAA` | Retrieves the IPv6 (AAAA) record. |
| `dig domain.com MX` | Finds mail servers. |
| `dig domain.com NS` | Identifies authoritative name servers. |
| `dig domain.com TXT` | Retrieves TXT records. |
| `dig domain.com CNAME` | Retrieves the CNAME record. |
| `dig domain.com SOA` | Retrieves the SOA record. |
| `dig @1.1.1.1 domain.com` | Queries a specific name server (here, `1.1.1.1`). |
| `dig +trace domain.com` | Shows the full DNS resolution path. |
| `dig -x 192.168.1.1` | Performs a reverse lookup (IP → hostname). |
| `dig +short domain.com` | Gives a short, concise answer only. |
| `dig +noall +answer domain.com` | Displays only the answer section. |
| `dig domain.com ANY` | Retrieves all available record types (many servers ignore this per RFC 8482). |

**Caution:** Excessive DNS queries can be detected and blocked by some servers — always respect rate limits and obtain permission before extensive DNS recon.

### 5.3 Groping DNS (Practical)
**Command explained:**
```shellsession
dig google.com
```
Output breaks into four sections:
- **Header** — `opcode: QUERY, status: NOERROR, id: 16449` (query type, success status, unique ID); `flags: qr rd ad` (`qr`=query response, `rd`=recursion desired, `ad`=authentic data); counts of question/answer/authority/additional entries; may show a warning if recursion was requested but unsupported.
- **Question Section** — restates what was asked (e.g., `;google.com. IN A` = "what is the A record for google.com?").
- **Answer Section** — the actual result (e.g., `google.com. 0 IN A 142.251.47.142`), where the numeric value is the **TTL** (time-to-live, how long the result can be cached).
- **Footer** — query time, which server answered and over what protocol (UDP/TCP), the timestamp, and the response message size in bytes.

An **OPT pseudosection** may also appear due to **EDNS** (Extension Mechanisms for DNS), enabling features like larger message sizes and DNSSEC support.

**Command explained (concise output):**
```shellsession
dig +short hackthebox.com
```
- `+short` — strips all header/footer/metadata, returning only the resolved IP address(es) — useful for quick lookups or scripting.

---

## Section 6: Subdomains

**Subdomains** are extensions of a main domain (e.g., `blog.example.com`, `shop.example.com`, `mail.example.com`), often used to organize different website sections or services. They're significant for web recon because they frequently host valuable, less-visible resources:
- **Development and Staging Environments** — often less securely configured, may expose vulnerabilities or sensitive info.
- **Hidden Login Portals** — admin panels or non-public login pages, attractive targets for unauthorized access attempts.
- **Legacy Applications** — older, forgotten apps possibly running outdated, vulnerable software.
- **Sensitive Information** — subdomains can inadvertently expose confidential documents, internal data, or config files.

### 6.1 Subdomain Enumeration
**Subdomain enumeration** is the systematic process of identifying and listing a domain's subdomains. From a DNS perspective, subdomains are typically represented by **A** (or **AAAA** for IPv6) records mapping the name to an IP, with **CNAME** records sometimes used as aliases pointing to other domains/subdomains.

### 6.2 Active Subdomain Enumeration
This involves directly interacting with the target's DNS servers. One method is attempting a **DNS zone transfer**, which can leak a full subdomain list if the server is misconfigured (rarely successful today due to tightened security). A more common technique is **brute-force enumeration** — systematically testing potential subdomain names from a wordlist against the target domain, automated by tools like `dnsenum`, `ffuf`, and `gobuster`.

### 6.3 Passive Subdomain Enumeration
This relies on external information sources rather than direct target queries. **Certificate Transparency (CT) logs** are a key resource — public SSL/TLS certificate repositories that often list associated subdomains in their Subject Alternative Name (SAN) field. **Search engines** (via operators like `site:`) can also filter results to reveal subdomains, and various online databases aggregate DNS data from multiple sources for subdomain searching without direct target interaction.

**Trade-offs:** Active enumeration offers more control and thoroughness but is more detectable; passive enumeration is stealthier but may miss subdomains. Combining both approaches yields the most thorough, effective strategy.

---

## Section 7: Subdomain Bruteforcing

**Subdomain Brute-Force Enumeration** is a powerful active discovery technique using pre-defined wordlists of potential subdomain names, systematically tested against the target domain to identify valid ones.

### 7.1 The Brute-Force Process
The process breaks into four steps:
1. **Wordlist Selection** — choose a wordlist:
   - **General-Purpose** — broad common names (dev, staging, blog, mail, admin, test); useful when naming conventions are unknown.
   - **Targeted** — focused on the target's industry/technology/naming patterns; more efficient, fewer false positives.
   - **Custom** — built from specific keywords, patterns, or intel gathered elsewhere.
2. **Iteration and Querying** — a script/tool appends each wordlist entry to the main domain (e.g., `dev.example.com`, `staging.example.com`).
3. **DNS Lookup** — a DNS query (typically A/AAAA) is performed for each potential subdomain to check resolution.
4. **Filtering and Validation** — successfully resolving subdomains are added to a valid list; further checks (e.g., browser access) may confirm existence/functionality.

### 7.2 Brute-Force Tools
| Tool | Description |
|---|---|
| dnsenum | Comprehensive DNS enumeration supporting dictionary/brute-force attacks. |
| fierce | User-friendly recursive subdomain discovery with wildcard detection. |
| dnsrecon | Combines multiple recon techniques with customisable output formats. |
| amass | Actively maintained, integrates with other tools and extensive data sources. |
| assetfinder | Simple, lightweight tool for quick subdomain scans. |
| puredns | Powerful, flexible DNS brute-forcing with effective resolving/filtering. |

### 7.3 DNSEnum (Practical)
**dnsenum** is a versatile Perl-based DNS reconnaissance toolkit offering:
- **DNS Record Enumeration** — retrieves A, AAAA, NS, MX, TXT records.
- **Zone Transfer Attempts** — automatically tries AXFR transfers from discovered name servers.
- **Subdomain Brute-Forcing** — wordlist-based testing of potential subdomains.
- **Google Scraping** — scrapes Google results for additional subdomains not in DNS directly.
- **Reverse Lookup** — identifies other domains hosted on the same IP.
- **WHOIS Lookups** — gathers domain ownership/registration info.

**Command explained:**
```bash
dnsenum --enum inlanefreight.com -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```
- `--enum inlanefreight.com` — the target domain, plus a shortcut enabling some tuned default options.
- `-f <path>` — path to the wordlist used for brute-forcing (here, SecLists' top-20000 subdomain list).
- `-r` — enables **recursive** subdomain brute-forcing; if a subdomain is found, dnsenum will also attempt to enumerate sub-subdomains of it.

**Step-by-step:**
1. Identify the target domain (e.g., `inlanefreight.com`).
2. Select an appropriate wordlist (e.g., SecLists' `subdomains-top1million-20000.txt`).
3. Run `dnsenum` with `--enum`, `-f`, and (optionally) `-r`.
4. Review output: dnsenum first shows the domain's host address(es), then systematically brute-forces and reports any subdomains that resolve (e.g., `www.inlanefreight.com`, `support.inlanefreight.com`), finishing with `done.`

---

## Section 8: DNS Zone Transfers

A less invasive, potentially more efficient alternative to brute-forcing for subdomain discovery is exploiting **DNS zone transfers** — a replication mechanism that, if misconfigured, can become a goldmine of information.

### 8.1 What is a Zone Transfer
A **DNS zone transfer** is a wholesale copy of all DNS records within a zone, transferred from one name server to another — essential for maintaining consistency/redundancy across servers, but a serious information leak if not properly secured.

**The zone transfer process:**
1. **Zone Transfer Request (AXFR)** — the secondary server sends a full zone transfer request (type AXFR) to the primary server.
2. **SOA Record Transfer** — the primary server responds with its SOA record, including the serial number used to check if the secondary's data is current.
3. **DNS Records Transmission** — the primary transfers all zone records one-by-one (A, AAAA, MX, CNAME, NS, etc.), defining the domain's full configuration.
4. **Zone Transfer Complete** — the primary signals the transfer is done.
5. **Acknowledgement (ACK)** — the secondary confirms successful receipt, completing the process.

### 8.2 The Zone Transfer Vulnerability
The core issue is weak **access control** over who can initiate a transfer. In the internet's early days, allowing any client to request a zone transfer was common practice — meaning anyone, including attackers, could request a full copy of a zone file. An unauthorised transfer can reveal:
- **Subdomains** — a complete list, including ones not linked from the main site (dev servers, staging, admin panels, etc.).
- **IP Addresses** — associated with each subdomain, useful for further targeting.
- **Name Server Records** — revealing the hosting provider and potential misconfigurations.

### 8.3 Remediation
Awareness of this risk has grown significantly, and most modern DNS servers now restrict zone transfers to **trusted secondary servers only**. However, misconfigurations still occur due to human error or outdated practices — making a zone transfer attempt (with proper authorization) still a worthwhile recon technique, since even a failed attempt can reveal configuration/security posture clues.

### 8.4 Exploiting Zone Transfers (Practical)
**Command explained:**
```shellsession
dig axfr @nsztm1.digi.ninja zonetransfer.me
```
- `axfr` — requests a full zone transfer.
- `@nsztm1.digi.ninja` — specifies the name server to query directly.
- `zonetransfer.me` — the target domain (a service deliberately set up to demonstrate zone transfer risks).

If successful, the output returns every DNS record in the zone — SOA, HINFO, TXT, MX, A, NS, SRV, PTR, AFSDB, and more — effectively exposing the domain's **entire** DNS configuration in one request. The footer shows query time, which server responded, the timestamp, and the total XFR size (records/bytes transferred).

**General step-by-step:**
1. Identify the target domain's authoritative name server(s), e.g. via `dig NS domain.com`.
2. Attempt a zone transfer against each name server: `dig axfr @<nameserver> <domain>`.
3. If the server is misconfigured, you'll receive the full zone file; if properly secured, the request will be refused (which is itself useful confirmation of correct configuration).

---

## Section 9: Virtual Hosts

Once DNS routes traffic to the correct server, **web server configuration** determines how incoming requests are actually handled. Servers like Apache, Nginx, or IIS use **virtual hosting** to host multiple websites/applications on a single server, differentiating between domains, subdomains, or separate sites via the **HTTP Host header** included in every browser request.

### 9.1 How Virtual Hosts Work: VHosts vs Subdomains
- **Subdomains** — extensions of a main domain (e.g., `blog.example.com`), typically with their own DNS records pointing to the same or a different IP; used to organize sections/services of a site.
- **Virtual Hosts (VHosts)** — configurations **within a web server** allowing multiple sites/apps on one server, associated with top-level domains or subdomains, each with its own separate configuration.

If a VHost has no DNS record, it can still be accessed by manually editing the local hosts file to map the domain name to the server's IP, bypassing DNS resolution. Many non-public subdomains/VHosts never appear in DNS records at all — **VHost fuzzing** is the technique of testing hostnames against a known IP to discover these.

VHosts can also map to entirely separate domains, not just subdomains:
```apacheconf
<VirtualHost *:80>
    ServerName www.example1.com
    DocumentRoot /var/www/example1
</VirtualHost>

<VirtualHost *:80>
    ServerName www.example2.org
    DocumentRoot /var/www/example2
</VirtualHost>

<VirtualHost *:80>
    ServerName www.another-example.net
    DocumentRoot /var/www/another-example
</VirtualHost>
```
Here, three unrelated domains share one server; the server uses the Host header to serve the right content for each.

### 9.2 Server VHost Lookup
**How a server resolves which content to serve:**
1. **Browser Requests a Website** — entering a domain initiates an HTTP request to the associated IP.
2. **Host Header Reveals the Domain** — the browser includes the domain name in the request's Host header.
3. **Web Server Determines the Virtual Host** — the server checks the Host header against its virtual host configuration for a match.
4. **Serving the Right Content** — once matched, the server retrieves and returns the corresponding files from that VHost's document root.

In essence, the Host header acts as a switch letting the server dynamically pick which site to serve.

### 9.3 Types of Virtual Hosting
| Type | Description | Pros | Cons |
|---|---|---|---|
| Name-Based | Relies solely on the HTTP Host header to distinguish sites. | Cost-effective, easy setup, widely supported, no extra IPs needed. | Requires server support for name-based hosting; can have SSL/TLS limitations. |
| IP-Based | Each hosted site gets its own unique IP address. | Doesn't rely on Host header; works with any protocol; better isolation. | Requires multiple IPs — expensive and less scalable. |
| Port-Based | Different sites use different ports on the same IP (e.g., 80 vs 8080). | Useful when IPs are limited. | Less common/user-friendly; often requires specifying the port in the URL. |

### 9.4 Virtual Host Discovery Tools
| Tool | Description | Features |
|---|---|---|
| gobuster | Multi-purpose brute-forcing tool, effective for VHost discovery. | Fast, supports multiple HTTP methods, custom wordlists. |
| Feroxbuster | Rust-based, similar to Gobuster, known for speed/flexibility. | Supports recursion, wildcard discovery, filters. |
| ffuf | Fast web fuzzer, usable for VHost discovery by fuzzing the Host header. | Customizable wordlist input and filtering. |

### 9.5 Gobuster (Practical)
**Preparation needed:**
1. **Target Identification** — find the target web server's IP (via DNS lookups or other recon).
2. **Wordlist Preparation** — a wordlist of potential VHost names (pre-compiled, e.g., SecLists, or custom-built).

**Command explained:**
```shellsession
gobuster vhost -u http://<target_IP_address> -w <wordlist_file> --append-domain
```
- `vhost` — selects Gobuster's virtual host discovery mode.
- `-u` — the target URL (use the IP address, not a domain, since you're testing which hostnames respond).
- `-w` — path to the wordlist file.
- `--append-domain` — appends the base domain to each wordlist entry, required in newer Gobuster versions to correctly construct full virtual hostnames (older versions handled this automatically or differently).

**Useful additional flags:**
- `-t` — increases thread count for faster scanning.
- `-k` — ignores SSL/TLS certificate errors.
- `-o` — saves output to a file for later analysis.

**Example command and output:**
```shellsession
gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
```
Output shows the tool's config summary (URL, method, threads, wordlist, user agent, timeout, Append Domain setting), then begins **VHOST enumeration mode**, reporting discovered VHosts (e.g., `Found: forum.inlanefreight.htb:81 Status: 200 [Size: 100]`) as it progresses through the wordlist, finishing with a completion summary.

**Caution:** VHost discovery can generate significant traffic and may be detected by IDS/WAF systems — always obtain proper authorization before scanning.

---

## Section 10: Certificate Transparency Logs

SSL/TLS underpins trust on the internet by encrypting browser-to-website communication via digital certificates. However, certificate issuance isn't foolproof — attackers can exploit rogue/mis-issued certificates to impersonate sites, intercept data, or spread malware. **Certificate Transparency (CT) logs** address this risk.

### 10.1 What are Certificate Transparency Logs?
CT logs are **public, append-only ledgers** recording SSL/TLS certificate issuance. Whenever a Certificate Authority (CA) issues a certificate, it must submit it to multiple CT logs, maintained by independent organizations and open for public inspection. They function as a global, transparent, verifiable certificate registry serving several purposes:
- **Early Detection of Rogue Certificates** — security researchers/owners can spot suspicious/misissued certificates quickly and revoke them before malicious use.
- **Accountability for Certificate Authorities** — CA issuance violations become publicly visible, risking sanctions or loss of trust.
- **Strengthening the Web PKI** — CT logs add public oversight/verification to the trust system (Public Key Infrastructure) underpinning secure online communication.

### 10.2 CT Logs and Web Recon
CT logs offer a unique advantage over brute-forcing/wordlist approaches: instead of guessing subdomain names, they provide a **definitive historical record** of certificates actually issued for a domain and its subdomains — not limited by wordlist scope or brute-force effectiveness. They can also reveal subdomains tied to **old or expired certificates**, which might run outdated, potentially vulnerable software/configurations. In short, CT logs offer a reliable, efficient subdomain discovery method that doesn't depend on exhaustive brute-forcing or wordlist completeness.

### 10.3 Searching CT Logs
| Tool | Key Features | Use Cases | Pros | Cons |
|---|---|---|---|---|
| crt.sh | Simple web interface, search by domain, shows certificate details/SAN entries. | Quick subdomain searches, certificate issuance history checks. | Free, easy, no registration. | Limited filtering/analysis options. |
| Censys | Powerful search engine for internet-connected devices; advanced filtering by domain/IP/cert attributes. | In-depth certificate analysis, misconfiguration/related host discovery. | Extensive data/filtering, API access. | Requires registration (free tier available). |

### 10.4 crt.sh Lookup (Practical)
crt.sh offers both a web interface and an API usable directly from the terminal.

**Command explained (finding all "dev" subdomains of facebook.com):**
```shellsession
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[] | select(.name_value | contains("dev")) | .name_value' | sort -u
```
- `curl -s "https://crt.sh/?q=facebook.com&output=json"` — fetches JSON-formatted certificate data from crt.sh matching `facebook.com`; `-s` silences curl's progress output.
- `jq -r '.[] | select(.name_value | contains("dev")) | .name_value'` — pipes the JSON into `jq`, filtering entries where the `name_value` field contains "dev", then extracting just that field; `-r` outputs raw (unquoted) strings.
- `sort -u` — sorts results alphabetically and removes duplicates.

Example output includes entries like `*.dev.facebook.com`, `dev.facebook.com`, `devvm1958.ftw3.facebook.com`, `secure.dev.facebook.com` — a richer, more targeted result set than blind brute-forcing could easily achieve.

---

## Section 11: Fingerprinting

**Fingerprinting** focuses on extracting technical details about the technologies powering a website — much like a human fingerprint uniquely identifies a person, a web server's digital signatures (software, OS, components) reveal critical infrastructure details and potential weaknesses, letting attackers tailor exploits to specific identified technologies.

Fingerprinting is valuable because it enables:
- **Targeted Attacks** — focusing effort on exploits known to affect the identified technologies, increasing success chances.
- **Identifying Misconfigurations** — exposing outdated software, default settings, or weaknesses other methods might miss.
- **Prioritising Targets** — helping decide which of multiple potential targets is more likely vulnerable or valuable.
- **Building a Comprehensive Profile** — combining fingerprint data with other recon findings for a holistic view of the target's security posture.

### 11.1 Fingerprinting Techniques
- **Banner Grabbing** — analysing banners presented by servers/services, often revealing software and version numbers.
- **Analysing HTTP Headers** — the `Server` header often discloses web server software; `X-Powered-By` may reveal scripting languages/frameworks.
- **Probing for Specific Responses** — crafted requests can elicit unique responses (error messages, behaviors) characteristic of particular software.
- **Analysing Page Content** — page structure, scripts, and elements (e.g., a copyright footer naming specific software) can hint at underlying technologies.

**Automated fingerprinting tools:**
| Tool | Description | Features |
|---|---|---|
| Wappalyzer | Browser extension/online service for tech profiling. | Identifies CMSs, frameworks, analytics tools, and more. |
| BuiltWith | Web tech profiler with detailed stack reports. | Free and paid plans with varying detail levels. |
| WhatWeb | Command-line fingerprinting tool. | Uses a vast signature database. |
| Nmap | Versatile scanner for service/OS fingerprinting. | Extendable via NSE scripts for specialised fingerprinting. |
| Netcraft | Web security services including fingerprinting/reporting. | Detailed reports on tech, hosting, security posture. |
| wafw00f | CLI tool specifically for WAF identification. | Determines WAF presence, type, and configuration. |

### 11.2 Banner Grabbing (Practical)
**Command explained:**
```shellsession
curl -I inlanefreight.com
```
- `-I` (or `--head`) — fetches only the HTTP **headers**, not the full page body, revealing server banner info quickly and with minimal interaction.

**Step-by-step (chasing redirects to fingerprint):**
1. Grab headers from the base domain:
   ```shellsession
   curl -I inlanefreight.com
   ```
   Output example: `Server: Apache/2.4.41 (Ubuntu)`, with a `301 Moved Permanently` redirecting to `https://inlanefreight.com/`.
2. Follow the redirect and grab headers again:
   ```shellsession
   curl -I https://inlanefreight.com
   ```
   Output reveals a further redirect, this time with `X-Redirect-By: WordPress`, pointing to `https://www.inlanefreight.com/` — a clue that WordPress is handling the redirection logic.
3. Grab headers from the final destination:
   ```shellsession
   curl -I https://www.inlanefreight.com
   ```
   Output returns `200 OK` along with `Link` headers referencing `wp-json` — the `wp-` prefix is a strong indicator of a **WordPress** installation.

Each redirect hop can reveal additional fingerprinting clues, so it's important to follow and inspect headers at every stage, not just the final response.

### 11.3 Wafw00f (Practical)
Before deeper fingerprinting, it's important to check for a **Web Application Firewall (WAF)**, since a WAF can interfere with or block recon probes.

**Step-by-step:**
1. Install wafw00f via pip:
   ```shellsession
   pip3 install git+https://github.com/EnableSecurity/wafw00f
   ```
2. Run it against the target domain:
   ```shellsession
   wafw00f inlanefreight.com
   ```
3. Review the result — e.g., `[+] The site https://inlanefreight.com is behind Wordfence (Defiant) WAF`, confirming an additional security layer is present that may need to be accounted for (or evaded) in further testing.

### 11.4 Nikto (Practical)
**Nikto** is an open-source web server scanner whose fingerprinting capabilities, alongside its primary vulnerability-assessment function, reveal insight into a site's technology stack.

**Step-by-step (installing Nikto, if not pre-installed):**
```shellsession
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl
```

**Command explained:**
```shellsession
nikto -h inlanefreight.com -Tuning b
```
- `-h` — specifies the target host.
- `-Tuning b` — restricts the scan to the **Software Identification** modules only (rather than a full vulnerability sweep).

**Output highlights explained:**
- Multiple resolved IPs (IPv4 and IPv6) and SSL certificate info (subject, alt names, cipher, issuer).
- `Server: Apache/2.4.41 (Ubuntu)` — confirms server software/version.
- Missing security headers flagged: no `Strict-Transport-Security`, no `X-Content-Type-Options` — both weakening the site's defense posture.
- `x-redirect-by: WordPress` — an uncommon header confirming WordPress involvement.
- `Content-Encoding: deflate` flagged as a possible **BREACH attack** risk.
- `Apache/2.4.41 appears to be outdated` — version-based vulnerability flag.
- `/license.txt` found — license files can reveal more about installed software components.
- `A Wordpress installation was found`, plus discovery of `/wp-login.php` (the WordPress login page) — both strong indicators for targeted WordPress-specific attack research.
- Cookie flagged as missing the `httponly` flag — a minor security hardening gap.

**Summary of findings from the inlanefreight.com scan:**
- Resolves to both IPv4 (`134.209.24.248`) and IPv6 (`2a03:b0c0:1:e0::32c:b001`) addresses.
- Runs Apache/2.4.41 (Ubuntu).
- Confirmed WordPress installation, including an accessible login page — a likely target for WordPress-specific exploitation.
- Information disclosure risk via `license.txt`.
- Several missing/insecure HTTP headers.

---

## Section 12: Crawling

**Crawling** (or **spidering**) is the automated, systematic process of browsing the web by following links from page to page, collecting information along the way — much like how search engines index pages, but here used for reconnaissance or data analysis purposes.

### 12.1 How Web Crawlers Work
A crawler starts with a **seed URL** (the initial page), fetches it, parses its content, and extracts all links found. These links are queued and crawled in turn, repeating iteratively until the configured scope (an entire site, or a portion of the web) is covered.

**Example progression:**
```txt
Homepage
├── link1
├── link2
└── link3
```
Visiting `link1` reveals further links:
```txt
link1 Page
├── Homepage
├── link2
├── link4
└── link5
```
The crawler continues systematically following links, gathering all accessible pages — this distinguishes crawling (following **actual, discovered** links) from fuzzing (guessing potential, undiscovered paths).

### 12.2 Breadth-First Crawling
**Breadth-first crawling** prioritizes a site's **width** before depth: it crawls all links on the seed page first, then moves to the links found on those pages, and so on. This is useful for getting a broad overview of a site's overall structure and content quickly.

### 12.3 Depth-First Crawling
**Depth-first crawling** prioritizes **depth** over breadth: it follows a single link path as far as possible before backtracking to explore other paths. This is useful for finding specific content or reaching deep into a site's structure rather than surveying it broadly.

The choice between these strategies depends on the specific goals of the crawl — broad mapping versus targeted deep exploration.

### 12.4 Extracting Valuable Information
Crawlers can extract a diverse range of data, each serving a distinct reconnaissance purpose:
- **Links (Internal and External)** — the web's fundamental connective structure; collecting these maps a site's layout, reveals hidden pages, and identifies relationships to external resources.
- **Comments** — blog/forum/page comment sections can be a goldmine, with users inadvertently revealing sensitive details, internal processes, or vulnerability hints.
- **Metadata** — "data about data": page titles, descriptions, keywords, author names, dates — providing valuable context about a page's purpose and relevance.
- **Sensitive Files** — crawlers can be configured to actively hunt for exposed backup files (`.bak`, `.old`), configuration files (`web.config`, `settings.php`), log files (`error_log`, `access_log`), and other files potentially containing passwords, API keys, or other confidential data — a prime source of credentials, encryption keys, or source code snippets.

### 12.5 The Importance of Context
A single data point (e.g., a comment mentioning a software version) may seem insignificant alone — but combined with other findings (an outdated version in metadata, a vulnerable config file elsewhere), it can become a critical vulnerability indicator. The module illustrates this with two examples:
1. A list of extracted links reveals a pattern — several point to a `/files/` directory. Manually visiting it reveals **directory browsing is enabled**, exposing backup archives and internal documents — a finding only possible through contextual pattern recognition, not isolated link review.
2. A seemingly innocuous comment mentioning a "file server" gains new significance once correlated with the `/files/` directory discovery above, reinforcing that the file server may be publicly exposed.

The takeaway: approach data analysis **holistically**, connecting relationships between data points rather than evaluating each piece in isolation.

---

## Section 13: robots.txt

The module frames **robots.txt** with an analogy: like a guest at a party avoiding rooms marked "Private," bots are expected to respect the boundaries this file sets, even though it's not strictly enforceable.

### 13.1 What is robots.txt?
Technically, `robots.txt` is a plain text file placed in a website's **root directory** (e.g., `www.example.com/robots.txt`), following the **Robots Exclusion Standard** — a set of guidelines for how crawlers should behave on a site. It contains "directives" specifying which parts of the site bots can and cannot crawl.

### 13.2 How robots.txt Works
Directives target specific **user-agents** (bot identifiers). Example:
```txt
User-agent: *
Disallow: /private/
```
- `User-agent: *` — applies the rule to all bots (the `*` wildcard).
- `Disallow: /private/` — tells those bots not to access any URL starting with `/private/`.

Other directives can allow access to specific paths, set crawl delays to avoid server overload, or link to sitemaps for more efficient crawling.

### 13.3 Understanding robots.txt Structure
The file consists of "records" separated by blank lines, each with two components:
- **User-agent** — which crawler/bot the rules apply to (`*` = all, or specific like "Googlebot," "Bingbot").
- **Directives** — the specific instructions for that user-agent.

**Common directives:**
| Directive | Description | Example |
|---|---|---|
| Disallow | Paths/patterns the bot should not crawl. | `Disallow: /admin/` |
| Allow | Explicitly permits crawling of a path, even under a broader Disallow. | `Allow: /public/` |
| Crawl-delay | Delay (seconds) between the bot's successive requests. | `Crawl-delay: 10` |
| Sitemap | URL to an XML sitemap for efficient crawling. | `Sitemap: https://www.example.com/sitemap.xml` |

### 13.4 Why Respect robots.txt?
Although `robots.txt` is **not strictly enforceable** (a rogue bot could ignore it entirely), most legitimate crawlers and search engines respect it for several reasons:
- **Avoiding Overburdening Servers** — limiting crawler access prevents excessive traffic that could slow/crash servers.
- **Protecting Sensitive Information** — shields private/confidential content from search engine indexing.
- **Legal and Ethical Compliance** — ignoring directives could violate a site's terms of service or, in some cases, raise legal concerns (especially around copyrighted/private data).

### 13.5 robots.txt in Web Reconnaissance
For security professionals, `robots.txt` is a valuable intelligence source even while respecting its directives:
- **Uncovering Hidden Directories** — disallowed paths often point to directories the owner wants kept out of search engine reach, potentially housing sensitive info, backups, or admin panels.
- **Mapping Website Structure** — analysing allowed/disallowed paths helps build a rudimentary site map, revealing unlinked sections or functionality.
- **Detecting Crawler Traps** — some sites include "honeypot" directories in `robots.txt` to lure malicious bots; spotting these reveals insight into the target's security awareness.

### 13.6 Analyzing robots.txt (Practical)
**Example file:**
```txt
User-agent: *
Disallow: /admin/
Disallow: /private/
Allow: /public/

User-agent: Googlebot
Crawl-delay: 10

Sitemap: https://www.example.com/sitemap.xml
```
**Analysis:**
- All user agents are disallowed from `/admin/` and `/private/`.
- All user agents are explicitly allowed into `/public/`.
- Googlebot specifically must wait 10 seconds between requests.
- A sitemap is provided at the stated URL for easier crawling/indexing.

**Inference:** the site likely has an **admin panel** at `/admin/` and some form of **private content** at `/private/` — both worth further (ethical, authorized) investigation.

---

## Section 14: Well-Known URIs

The **`.well-known`** standard, defined in **RFC 8615**, is a standardized directory within a website's root domain — typically accessible at the `/.well-known/` path — centralizing critical metadata: configuration files and information about a site's services, protocols, and security mechanisms.

This consistent, predictable location simplifies discovery for browsers, applications, and security tools alike, letting clients auto-locate specific configuration files by constructing the appropriate URL (e.g., `https://example.com/.well-known/security.txt`).

**IANA maintains a registry of `.well-known` URIs**, each serving a defined purpose. Notable examples:
| URI Suffix | Description | Status | Reference |
|---|---|---|---|
| security.txt | Contact info for security researchers to report vulnerabilities. | Permanent | RFC 9116 |
| /.well-known/change-password | Standard URL directing users to a password change page. | Provisional | W3C webappsec spec |
| openid-configuration | Configuration details for OpenID Connect (identity layer atop OAuth 2.0). | Permanent | OpenID Connect Discovery spec |
| assetlinks.json | Verifies ownership of digital assets (e.g., apps) tied to a domain. | Permanent | Google Digital Asset Links spec |
| mta-sts.txt | Policy for SMTP MTA Strict Transport Security (email security). | Permanent | RFC 8461 |

This is only a small sample — each registry entry has its own implementation guidelines, ensuring a standardized approach across the many applications of `.well-known`.

### 14.1 Web Recon and .well-known
One particularly useful URI for recon is **openid-configuration**, part of the **OpenID Connect Discovery** protocol. When a client wants to use OpenID Connect for authentication, it retrieves the provider's config at `https://example.com/.well-known/openid-configuration`, returning a JSON document describing the provider's endpoints and capabilities:

```json
{
  "issuer": "https://example.com",
  "authorization_endpoint": "https://example.com/oauth2/authorize",
  "token_endpoint": "https://example.com/oauth2/token",
  "userinfo_endpoint": "https://example.com/oauth2/userinfo",
  "jwks_uri": "https://example.com/oauth2/jwks",
  "response_types_supported": ["code", "token", "id_token"],
  "subject_types_supported": ["public"],
  "id_token_signing_alg_values_supported": ["RS256"],
  "scopes_supported": ["openid", "profile", "email"]
}
```

**What this reveals, field by field:**
- `authorization_endpoint` — the URL where authorization requests are sent.
- `token_endpoint` — where access/refresh tokens are issued.
- `userinfo_endpoint` — where authenticated user info is retrieved.
- `jwks_uri` — the **JSON Web Key Set (JWKS)** location, detailing the server's cryptographic signing keys.
- `scopes_supported` / `response_types_supported` — reveal the supported functionality and limitations of the OpenID Connect implementation.
- `id_token_signing_alg_values_supported` — reveals which signing algorithms are in use, relevant to understanding the implementation's security posture.

The module recommends exploring the full IANA `.well-known` registry and experimenting with various URIs as a valuable, often-overlooked avenue for uncovering structured configuration and endpoint details during reconnaissance.

---

## Section 15: Creepy Crawlies

This section covers dedicated web crawling **tools** that automate the crawling process covered conceptually in Section 12, making it faster and more efficient.

### 15.1 Popular Web Crawlers
- **Burp Suite Spider** — part of the Burp Suite testing platform; excels at mapping web applications, finding hidden content, and uncovering vulnerabilities.
- **OWASP ZAP (Zed Attack Proxy)** — free, open-source security scanner with automated/manual modes and a built-in spider component.
- **Scrapy (Python Framework)** — a versatile, scalable Python framework for building custom crawlers, handling structured data extraction and complex crawling scenarios; ideal for tailored recon tasks.
- **Apache Nutch (Scalable Crawler)** — a highly extensible, scalable Java-based open-source crawler for massive or domain-focused crawls; requires more technical setup but offers strong power/flexibility for large-scale projects.

**Ethical note:** always obtain permission before crawling a website (especially for extensive/intrusive scans), and be mindful of server resource impact to avoid overloading the target.

### 15.2 Scrapy (Practical)
**Command explained:**
```shellsession
pip3 install scrapy
```
- Installs Scrapy and its dependencies via pip, preparing the environment for building/running spiders.

### 15.3 ReconSpider (Practical)
The module provides a pre-built custom Scrapy spider, **ReconSpider**, tailored for reconnaissance.

**Step-by-step:**
1. Download and extract the ReconSpider package:
   ```shellsession
   wget -O ReconSpider.zip https://cdn.services-k8s.prod.aws.htb.systems/content/modules/144/ReconSpider.v1.2.zip
   unzip ReconSpider.zip
   ```
   - `wget -O ReconSpider.zip <url>` — downloads the file and saves it under the specified name.
   - `unzip` — extracts the archive's contents into the current directory.
2. Run the spider against a target domain:
   ```shellsession
   python3 ReconSpider.py http://inlanefreight.com
   ```
   - Replace the URL with your actual target; the spider crawls it and collects reconnaissance data.
3. Review the results, saved automatically to `results.json`.

### 15.4 results.json Structure
**Example structure:**
```json
{
    "emails": ["lily.floid@inlanefreight.com", "cvs@inlanefreight.com", ...],
    "links": ["https://www.themeansar.com", "https://www.inlanefreight.com/index.php/offices/", ...],
    "external_files": ["https://www.inlanefreight.com/wp-content/uploads/2020/09/goals.pdf", ...],
    "js_files": ["https://www.inlanefreight.com/wp-includes/js/jquery/jquery-migrate.min.js?ver=3.3.2", ...],
    "form_fields": [],
    "images": ["https://www.inlanefreight.com/wp-content/uploads/2021/03/AboutUs_01-1024x810.png", ...],
    "videos": [],
    "audio": [],
    "comments": ["<!-- #masthead -->", ...]
}
```
**Key meanings:**
| JSON Key | Description |
|---|---|
| emails | Email addresses found on the domain. |
| links | URLs of links found within the domain. |
| external_files | URLs of external files such as PDFs. |
| js_files | URLs of JavaScript files used by the site. |
| form_fields | Form fields found on the domain. |
| images | URLs of images found on the domain. |
| videos | URLs of videos found on the domain. |
| audio | URLs of audio files found on the domain. |
| comments | HTML comments found in the page source. |

Exploring this structured output can reveal valuable insights into the web application's architecture, content, and points of interest for further manual investigation.

---

## Section 16: Search Engine Discovery

Search engines serve as navigational guides to the internet's vast information, but beyond answering everyday queries, they also hold data invaluable for reconnaissance — a practice known as **search engine discovery** or **OSINT (Open Source Intelligence) gathering**. This involves using search engines and specialised operators to uncover data not readily visible on websites themselves, from employee information and sensitive documents to hidden login pages and exposed credentials.

### 16.1 Why Search Engine Discovery Matters
Key advantages:
- **Open Source** — publicly accessible info, making it a legal and ethical reconnaissance method.
- **Breadth of Information** — search engines index a vast portion of the web.
- **Ease of Use** — no specialised technical skills required.
- **Cost-Effective** — a free, readily available resource.

**Applications:**
- **Security Assessment** — identifying vulnerabilities, exposed data, attack vectors.
- **Competitive Intelligence** — gathering info on competitors' products/strategies.
- **Investigative Journalism** — uncovering hidden connections or unethical practices.
- **Threat Intelligence** — identifying emerging threats and tracking malicious actors.

**Limitation:** search engines don't index everything — some data is deliberately hidden or protected.

### 16.2 Search Operators
Search operators act as "secret codes" unlocking precision in search queries. While exact syntax may vary slightly by engine, the underlying principles are consistent.

| Operator | Description | Example | Example Description |
|---|---|---|---|
| `site:` | Limits results to a specific site/domain. | `site:example.com` | All publicly accessible pages on example.com. |
| `inurl:` | Finds pages with a term in the URL. | `inurl:login` | Search for login pages anywhere. |
| `filetype:` | Searches for a specific file type. | `filetype:pdf` | Find downloadable PDFs. |
| `intitle:` | Finds pages with a term in the title. | `intitle:"confidential report"` | Documents titled "confidential report" or similar. |
| `intext:` / `inbody:` | Searches for a term within page body text. | `intext:"password reset"` | Pages containing "password reset". |
| `cache:` | Shows the cached version of a page. | `cache:example.com` | View example.com's previous cached content. |
| `link:` | Finds pages linking to a specific page. | `link:example.com` | Sites linking to example.com. |
| `related:` | Finds sites related to a given page. | `related:example.com` | Sites similar to example.com. |
| `info:` | Summarises info about a page. | `info:example.com` | Basic details like title/description. |
| `define:` | Provides a word/phrase definition. | `define:phishing` | Definitions of "phishing". |
| `numrange:` | Searches for numbers in a range. | `site:example.com numrange:1000-2000` | Pages with numbers 1000–2000. |
| `allintext:` | Finds pages with all specified words in body text. | `allintext:admin password reset` | Pages with both "admin" and "password reset". |
| `allinurl:` | Finds pages with all specified words in the URL. | `allinurl:admin panel` | Pages with "admin" and "panel" in URL. |
| `allintitle:` | Finds pages with all specified words in the title. | `allintitle:confidential report 2023` | Titles containing all three terms. |
| `AND` | Narrows results — all terms must be present. | `site:example.com AND (inurl:admin OR inurl:login)` | Admin/login pages on example.com specifically. |
| `OR` | Broadens results — any term can match. | `"linux" OR "ubuntu" OR "debian"` | Pages mentioning any of the three. |
| `NOT` | Excludes a term. | `site:bank.com NOT inurl:login` | bank.com pages excluding login pages. |
| `*` (wildcard) | Represents any character/word. | `site:socialnetwork.com filetype:pdf user* manual` | Matches "user guide," "user handbook," etc. |
| `..` (range search) | Finds results within a numeric range. | `site:ecommerce.com "price" 100..500` | Products priced 100–500. |
| `" "` (quotes) | Searches for an exact phrase. | `"information security policy"` | Exact phrase match. |
| `-` (minus) | Excludes a term from results. | `site:news.com -inurl:sports` | news.com articles excluding sports content. |

### 16.3 Google Dorking
**Google Dorking** (or "Google Hacking") uses search operators to uncover sensitive info, security vulnerabilities, or hidden content via Google Search. The module points to the **Google Hacking Database** for further examples. Common dork patterns include:

**Finding Login Pages:**
```
site:example.com inurl:login
site:example.com (inurl:login OR inurl:admin)
```

**Identifying Exposed Files:**
```
site:example.com filetype:pdf
site:example.com (filetype:xls OR filetype:docx)
```

**Uncovering Configuration Files:**
```
site:example.com inurl:config.php
site:example.com (ext:conf OR ext:cnf)
```

**Locating Database Backups:**
```
site:example.com inurl:backup
site:example.com filetype:sql
```

---

## Section 17: Web Archives

Websites constantly change or disappear, leaving only fleeting traces — but the **Internet Archive's Wayback Machine** offers a unique opportunity to revisit the past and explore a site's digital footprint across time.

### 17.1 What is the Wayback Machine?
The **Wayback Machine** is a digital archive of the World Wide Web, founded and maintained by the **Internet Archive** (a non-profit) and archiving websites since **1996**. It lets users "go back in time" and view snapshots ("captures" or "archives") of websites as they appeared at various points in history — including design, content, and functionality.

### 17.2 How Does the Wayback Machine Work?
A three-step process:
1. **Crawling** — automated bots systematically browse the internet, following links and **downloading full copies** of webpages (not just indexing them for search, as traditional search crawlers do).
2. **Archiving** — downloaded pages and their resources (images, stylesheets, scripts) are stored in the archive, each snapshot tied to a specific date/time. Archiving frequency varies — daily, weekly, or monthly — depending on a site's popularity and update frequency.
3. **Accessing** — users enter a URL and select a date to view the site as it appeared then; tools exist for browsing individual pages, searching archived content for specific terms, or downloading entire archived sites for offline analysis.

Not every page is captured — the Wayback Machine prioritizes sites of cultural, historical, or research value, and site owners can request exclusion (though this isn't always guaranteed).

### 17.3 Why the Wayback Machine Matters for Web Reconnaissance
- **Uncovering Hidden Assets and Vulnerabilities** — discovering old pages, directories, files, or subdomains no longer accessible on the live site, potentially exposing sensitive info or security flaws.
- **Tracking Changes and Identifying Patterns** — comparing historical snapshots reveals structural/content/technology evolution and potential vulnerability history.
- **Gathering Intelligence** — archived content is a valuable OSINT source for past activities, marketing strategy, employees, and technology choices.
- **Stealthy Reconnaissance** — accessing archives is passive and doesn't directly interact with the target's live infrastructure, making it a low-detection recon method.

### 17.4 Going Wayback on HTB (Practical)
**Step-by-step:**
1. Go to the Wayback Machine and enter the target page URL.
2. Select the earliest available capture date (for the module's HackTheBox example: **2017-06-10 @ 04h23:01**).
3. Review the archived snapshot — in the HTB example, this reveals the very first version of the HackTheBox homepage (v0.8.7 beta), including its original geometric cube logo and "About" description of the platform — showing how far the site has evolved from its earliest form.

---

## Section 18: Automating Recon

Manual reconnaissance is effective but can be **time-consuming and prone to human error**. Automating recon tasks significantly enhances efficiency and accuracy, allowing information gathering at scale with faster vulnerability identification.

### 18.1 Why Automate Reconnaissance?
- **Efficiency** — automated tools perform repetitive tasks far faster than manual work, freeing time for analysis/decision-making.
- **Scalability** — scales recon efforts across many targets/domains, broadening the scope of discovered information.
- **Consistency** — automated tools follow predefined rules, ensuring consistent, reproducible results and minimizing human error.
- **Comprehensive Coverage** — can be programmed for a wide range of tasks (DNS enumeration, subdomain discovery, crawling, port scanning, etc.) for thorough attack-vector coverage.
- **Integration** — many frameworks integrate easily with other tools/platforms, creating a seamless workflow from recon through vulnerability assessment and exploitation.

### 18.2 Reconnaissance Frameworks
| Framework | Description |
|---|---|
| FinalRecon | Python-based tool with modules for SSL checking, WHOIS gathering, header analysis, and crawling; modular and customisable. |
| Recon-ng | Powerful Python framework with modules for DNS enumeration, subdomain discovery, port scanning, web crawling, and even known-vulnerability exploitation. |
| theHarvester | Specialized for gathering emails, subdomains, hosts, employee names, open ports, and banners from public sources (search engines, PGP key servers, SHODAN). |
| SpiderFoot | Open-source intelligence automation tool integrating many data sources (IPs, domains, emails, social profiles); performs DNS lookups, crawling, port scanning. |
| OSINT Framework | A collection of various OSINT tools/resources spanning social media, search engines, public records, and more. |

### 18.3 FinalRecon (Practical)
**FinalRecon** offers a broad recon feature set: Header Information, Whois Lookup, SSL Certificate Information, a Crawler (HTML/CSS/JS extraction, internal/external links, images, robots.txt, sitemap.xml, JS-embedded links, Wayback Machine data), DNS Enumeration (40+ record types including DMARC), Subdomain Enumeration (via crt.sh, AnubisDB, ThreatMiner, CertSpotter, Facebook API, VirusTotal API, Shodan API, BeVigil API), Directory Enumeration (custom wordlists/extensions), and Wayback Machine URL retrieval (last 5 years).

**Step-by-step (installation):**
```shellsession
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
./finalrecon.py --help
```
- `git clone <url>` — downloads the FinalRecon repository into a new `FinalRecon` directory.
- `cd FinalRecon` — enters the new directory.
- `pip3 install -r requirements.txt` — installs all required Python dependencies.
- `chmod +x ./finalrecon.py` — makes the main script executable.
- `./finalrecon.py --help` — displays the full option/module reference.

**Key options:**
| Option | Argument | Description |
|---|---|---|
| `-h, --help` | — | Show the help message and exit. |
| `--url` | URL | Specify the target URL. |
| `--headers` | — | Retrieve header information. |
| `--sslinfo` | — | Get SSL certificate information. |
| `--whois` | — | Perform a Whois lookup. |
| `--crawl` | — | Crawl the target website. |
| `--dns` | — | Perform DNS enumeration. |
| `--sub` | — | Enumerate subdomains. |
| `--dir` | — | Search for directories. |
| `--wayback` | — | Retrieve Wayback URLs. |
| `--ps` | — | Perform a fast port scan. |
| `--full` | — | Perform a full reconnaissance scan. |

**Extra tuning flags** include `-nb` (hide banner), `-dt`/`-pt` (directory/port scan threads), `-T` (request timeout), `-w` (wordlist path), `-r` (allow redirect), `-s` (toggle SSL verification), `-sp` (SSL port), `-d` (custom DNS servers), `-e` (file extensions), `-o` (export format), `-cd` (export directory), `-k` (add API key, e.g. `-k shodan@key`).

**Example command:**
```shellsession
./finalrecon.py --headers --whois --url http://inlanefreight.com
```
This runs just the header-gathering and WHOIS modules against the target. Output includes the resolved IP, full response headers (server software, caching/link headers, content encoding, etc.), and the full WHOIS record (registrar, registry dates, domain status flags, name servers) — then confirms completion time and the local path where results were exported (e.g., `~/.local/share/finalrecon/dumps/`).

---

## Cheat Sheet

Web reconnaissance is the first step in any security assessment or penetration testing engagement — akin to a detective's initial investigation, gathering clues about a target before formulating a plan of action. In the digital realm, this means accumulating information about a website or web application to identify potential vulnerabilities, misconfigurations, and valuable assets.

**Primary goals of web reconnaissance:**
- **Identifying Assets** — discovering all associated domains, subdomains, and IP addresses to map the target's online presence.
- **Uncovering Hidden Information** — finding directories, files, and technologies not readily apparent that could serve as attacker entry points.
- **Analyzing the Attack Surface** — identifying open ports, running services, and software versions to assess potential vulnerabilities.
- **Gathering Intelligence** — collecting employee info, emails, and technology usage to aid social engineering or technology-specific exploitation.

| Type | Description | Risk of Detection | Examples |
|---|---|---|---|
| Active Reconnaissance | Directly interacts with the target system (probes/requests). | Higher | Port scanning, vulnerability scanning, network mapping |
| Passive Reconnaissance | Gathers information without direct interaction, relying on public data. | Lower | Search engine queries, WHOIS lookups, DNS enumeration, web archive analysis, social media |

### WHOIS
A query/response protocol for retrieving domain name, IP address, and other internet resource information — a directory service detailing domain ownership, registration dates, and contact info.

```bash
whois example.com
```
Returns registrar, registration/expiration dates, nameservers, and owner contact info. Note: WHOIS data can be inaccurate or deliberately obscured by privacy services, so cross-verify from multiple sources.

### DNS
DNS functions as the internet's GPS, translating domain names into IP addresses.

```bash
dig example.com A
```
Queries the A record (IPv4 address) for the domain.

| Record Type | Description |
|---|---|
| A | Maps a hostname to an IPv4 address. |
| AAAA | Maps a hostname to an IPv6 address. |
| CNAME | Creates an alias for a hostname, pointing it to another hostname. |
| MX | Specifies mail servers responsible for handling email for the domain. |
| NS | Delegates a DNS zone to a specific authoritative name server. |
| TXT | Stores arbitrary text information. |
| SOA | Contains administrative information about a DNS zone. |

### Subdomains
Extensions of a primary domain (e.g., `mail.example.com`), valuable for recon since they can expose additional attack surface, hidden services, or internal network clues.

| Approach | Description | Examples |
|---|---|---|
| Active Enumeration | Directly interacts with DNS servers or probing tools. | Brute-forcing, DNS zone transfers |
| Passive Enumeration | Collects subdomain info from public sources without direct interaction. | Certificate Transparency (CT) logs, search engine queries |

### Subdomain Brute-Forcing
Systematically generates and tests potential subdomain names against the target's DNS server.

```bash
dnsenum example.com -f subdomains.txt
```

### Zone Transfers
AXFR (Asynchronous Full Transfer) requests can replicate an entire DNS zone file if a server is misconfigured, exposing subdomains, IPs, and mail server configs.

```bash
dig @ns1.example.com example.com axfr
```
Most servers now restrict zone transfers to authorized secondary servers only, but misconfigured servers may still allow transfer from any source.

### Virtual Hosts
Virtual hosting lets multiple websites share a single IP address, each identified via a unique hostname. Since scanning the IP alone won't reveal all hosted sites, a tool must test different hostnames against it.

```bash
gobuster vhost -u http://192.0.2.1 -w hostnames.txt
```
`-u` specifies the target IP; `-w` specifies the wordlist of candidate hostnames. Gobuster reports any hostname that yields a valid server response.

### Certificate Transparency (CT) Logs
Publicly accessible logs recording SSL/TLS certificates issued for domains and subdomains — a passive reconnaissance goldmine for subdomain discovery.

```bash
curl -s "https://crt.sh/?q=%25.example.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```
Fetches JSON data from crt.sh (`%` is a wildcard), extracts domain names with `jq`, strips wildcard prefixes (`*.`) with `sed`, then sorts and deduplicates results.

### Web Crawling
The automated exploration of a website's structure, following links to map pages and extract embedded information.

`robots.txt` (in a site's root directory) dictates which areas are off-limits to crawlers — analyzing it can reveal hidden/sensitive directories the owner doesn't want indexed.

**Basic Scrapy spider example (extracting links):**
```python
import scrapy

class ExampleSpider(scrapy.Spider):
    name = "example"
    start_urls = ['http://example.com/']

    def parse(self, response):
        for link in response.css('a::attr(href)').getall():
            if any(link.endswith(ext) for ext in self.interesting_extensions):
                yield {"file": link}
            elif not link.startswith("#") and not link.startswith("mailto:"):
                yield response.follow(link, callback=self.parse)
```

**Analysing scraped output:**
```bash
jq -r '.[] | select(.file != null) | .file' example_data.json | sort -u
```
Extracts, sorts, and deduplicates discovered file links from the spider's JSON output for further manual review.

### Search Engine Discovery
Uses search engines' vast indexes to passively uncover target information — a form of Open Source Intelligence (OSINT) gathering — via advanced search operators ("Google Dorks").

| Operator | Description | Example |
|---|---|---|
| `site:` | Restricts results to a specific website. | `site:example.com "password reset"` |
| `inurl:` | Searches for a term in the URL. | `inurl:admin login` |
| `filetype:` | Limits results to a specific file type. | `filetype:pdf "confidential report"` |
| `intitle:` | Searches for a term in the page title. | `intitle:"index of" /backup` |
| `cache:` | Shows the cached version of a webpage. | `cache:example.com` |
| `"search term"` | Searches for an exact phrase. | `"internal error" site:example.com` |
| `OR` | Combines multiple search terms. | `inurl:admin OR inurl:login` |
| `-` | Excludes specific terms from results. | `inurl:admin -intext:wordpress` |

### Web Archives
Digital repositories storing historical snapshots of websites over time — the **Wayback Machine** (by the Internet Archive) is the most comprehensive and accessible of these, archiving the web for over two decades.

| Feature | Description | Use Case in Reconnaissance |
|---|---|---|
| Historical Snapshots | View past versions of websites, including pages, content, and design changes. | Identify past content/functionality no longer available. |
| Hidden Directories | Explore directories/files removed or hidden from the current version. | Discover sensitive info or backups inadvertently left accessible previously. |
| Content Changes | Track changes in website content (text, images, links). | Identify update patterns and assess security posture evolution. |
