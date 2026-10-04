# Network Enumeration with Nmap — HTB Academy Structured Notes

> **Module:** Network Enumeration with Nmap | **Sections:** 1–12 | **Source:** HTB Academy

---

## Table of Contents

1. [Section 1 — Enumeration](#section-1)
2. [Section 2 — Introduction to Nmap](#section-2)
3. [Section 3 — Host Discovery](#section-3)
4. [Section 4 — Host and Port Scanning](#section-4)
5. [Section 5 — Saving the Results](#section-5)
6. [Section 6 — Service Enumeration](#section-6)
7. [Section 7 — Nmap Scripting Engine](#section-7)
8. [Section 8 — Performance](#section-8)
9. [Section 9 — Firewall and IDS/IPS Evasion](#section-9)
10. [Section 10 — Firewall and IDS/IPS Evasion — Easy Lab](#section-10)
11. [Section 11 — Firewall and IDS/IPS Evasion — Medium Lab](#section-11)
12. [Section 12 — Firewall and IDS/IPS Evasion — Hard Lab](#section-12)
13. [Nmap Cheatsheet — Quick Reference](#cheatsheet)

---

<a name="section-1"></a>
## Section 1 — Enumeration

### What Is Enumeration?

Enumeration is the most critical phase in any penetration test. The goal is **not** simply to gain access to the target — the real objective is to identify **all possible ways** we could attack a target. Enumeration is about actively interacting with each individual service to discover what information it provides and what possibilities it offers. It is not just about running tools — tools alone will never replace knowledge and attention to detail. Without understanding how services work and what syntax they use, the results from tools mean very little.

---

### Why Enumeration Matters More Than Tools

Most scanning tools simplify and speed up the process, but they have limits. A tool relies on timeouts: if a port doesn't respond within a specific window, the tool marks it as `closed`, `filtered`, or `unknown`. A port mistakenly marked as `closed` might contain a critical service that offers a path into the system — and we'd waste hours before discovering it manually. The difference between finding an attack path quickly and getting stuck for days usually comes down to understanding **how services work**, not which tools were run. The goal of enumeration is collecting as much information as possible — the more we know, the easier it is to find attack vectors.

---

### Two Key Ways to Find Attack Vectors

Every piece of useful information we gather falls into one of two categories:

1. **Functions or resources** that allow direct interaction with the target or provide additional information (e.g., an exposed FTP with anonymous login, a public web application, an open admin panel).
2. **Information that leads to more information** — data that, when combined with other findings, opens new doors (e.g., a version number that leads to a known CVE, a username that leads to a password spray).

When scanning and inspecting, we look exactly for these two types. Most of the information we find comes from misconfigurations or a poor security mindset — such as an administrator relying solely on a firewall, Group Policy Objects, and updates without verifying actual service security.

---

### The Car Keys Analogy

A useful mental model for understanding the importance of detail in enumeration: if you call a partner to ask where your car keys are and they say "in the living room" — that's not very helpful. If they say "in the living room, on the white shelf, next to the TV, in the third drawer" — you find the keys immediately. Enumeration works the same way. Vague, shallow scanning leaves us searching blindly. Deep, detailed enumeration with full understanding of what each service is telling us is what makes the difference between finding the path to access and wasting hours.

---

### Manual Enumeration vs. Automated Tools

Automated tools are valuable but incomplete. They are designed for general use and cannot always bypass security measures or interpret service behaviour correctly. Manual enumeration — such as directly connecting to a service with Netcat, reading banners, and understanding protocols — often reveals information that automated tools miss. The key lesson is that tools are supplements to knowledge, not replacements for it. Understanding the service's syntax, purpose, and expected behavior is what makes findings meaningful.

---

<a name="section-2"></a>
## Section 2 — Introduction to Nmap

### What Is Nmap?

**Network Mapper (Nmap)** is an open-source network analysis and security auditing tool written in C, C++, Python, and Lua. It is designed to scan networks, identify active hosts, and discover open ports, services, and application versions using raw packet construction and analysis. Nmap can also detect operating systems, check firewall and IDS configurations, and interact with target services through its scripting engine. It is one of the most widely used tools by both network administrators and penetration testers. Nmap is available on Linux, Windows, and macOS.

---

### Use Cases

Nmap serves a wide range of purposes in both offensive and defensive security contexts:

- **Auditing network security** — identifying exposed services and open ports
- **Simulating penetration tests** — mapping attack surfaces
- **Checking firewall and IDS configurations** — verifying which ports are reachable and how traffic is handled
- **Network mapping** — discovering all active hosts and their relationships
- **Response analysis** — studying how services and hosts respond to different packet types
- **Vulnerability assessment** — identifying known weaknesses in detected service versions
- **Identifying open ports** — finding all TCP/UDP entry points on a host

---

### Nmap Architecture

Nmap's functionality is organized into five core scanning categories:

| Category | Description |
|---|---|
| **Host Discovery** | Determines which hosts are alive on a network using ICMP, ARP, or TCP/UDP probes |
| **Port Scanning** | Identifies open, closed, or filtered ports using various TCP/UDP techniques |
| **Service Enumeration & Detection** | Determines the name, version, and details of services running on open ports |
| **OS Detection** | Fingerprints the target's operating system based on TCP/IP stack behavior |
| **NSE (Nmap Scripting Engine)** | Allows scriptable interaction with target services using Lua-based scripts |

---

### Syntax

The basic Nmap command structure is:

```bash
nmap <scan types> <options> <target>
```

**Examples:**
- `nmap 10.129.2.28` — basic scan of a single host
- `nmap 10.129.2.0/24` — scan an entire /24 subnet
- `nmap -sS -sV -p- 10.129.2.28` — SYN scan with version detection on all ports

The `<target>` can be a single IP, a range (`10.129.2.18-20`), a subnet (`10.129.2.0/24`), or a hostname. Options are added with `-` for short flags or `--` for long flags.

---

### Scan Techniques

Nmap provides a comprehensive list of scan techniques, viewable with `nmap --help`:

```
-sS/sT/sA/sW/sM    TCP SYN/Connect/ACK/Window/Maimon scans
-sU                  UDP Scan
-sN/sF/sX           TCP Null, FIN, and Xmas scans
--scanflags <flags>  Customize TCP scan flags
-sI <zombie host>    Idle scan
-sY/sZ               SCTP INIT/COOKIE-ECHO scans
-sO                  IP protocol scan
-b <FTP relay>       FTP bounce scan
```

The most commonly used technique is the **TCP SYN scan (`-sS`)**, which is the default when run as root. It is also called a "half-open scan" because it sends a SYN packet but never completes the three-way TCP handshake.

---

### TCP-SYN Scan Explained

The **TCP SYN scan (`-sS`)** works as follows:

- Sends a TCP packet with the **SYN flag** to the target port
- If the target replies with **SYN-ACK** → port is **open**
- If the target replies with **RST** → port is **closed**
- If Nmap receives **no response** → port is **filtered**

This method is fast (can scan thousands of ports per second) and stealthy because it never establishes a full connection, making it less likely to appear in application-level logs. However, modern IDS systems can still detect half-open scans.

---

### Basic SYN Scan Example

**Command:**
```bash
sudo nmap -sS localhost
```

**Example Output:**
```
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1
```

**Explanation:**  
The scan reveals four open TCP ports on localhost. The `-sS` flag is the default when running as root. Without `sudo`, Nmap falls back to a TCP Connect scan (`-sT`) because raw packet crafting requires root/administrator privileges.

---

<a name="section-3"></a>
## Section 3 — Host Discovery

### Overview

Before port scanning, the first step in any penetration test is determining **which hosts are alive** on the network. For an internal network assessment, we need to map the entire IP range first. Nmap provides several host discovery methods, with ICMP echo requests being the most commonly used. It is always recommended to **save every scan result** — different tools and different scan methods may produce different results, and having documented output makes it easier to compare findings, write reports, and spot discrepancies over time.

---

### Scan Network Range

**Command:**
```bash
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5
```

**What Each Part Does:**

| Option | Description |
|---|---|
| `10.129.2.0/24` | Target network range (all 256 addresses in this /24 subnet) |
| `-sn` | Disables port scanning — only performs host discovery (ping scan) |
| `-oA tnet` | Saves results in **all three formats** (`.nmap`, `.gnmap`, `.xml`) with the base name `tnet` |
| `grep for` | Filters the output to show only lines containing "for" (which appear in "Nmap scan report for ...") |
| `cut -d" " -f5` | Cuts each filtered line by space delimiter and extracts the 5th field (the IP address) |

**Example Output:**
```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

**Explanation:**  
Seven hosts responded to the host discovery scan. Hosts that did not respond are not listed — they may be down or have firewalls blocking ICMP.

---

### Scan IP List

During internal penetration tests, clients often provide a pre-defined list of hosts to test. Nmap can read this list with `-iL`.

**Creating/Viewing a host list:**
```bash
cat hosts.lst
```
```
10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

**Command to scan the list:**
```bash
sudo nmap -sn -oA tnet -iL hosts.lst | grep for | cut -d" " -f5
```

**What Each Part Does:**

| Option | Description |
|---|---|
| `-sn` | Disables port scanning |
| `-oA tnet` | Saves results in all formats |
| `-iL hosts.lst` | Reads target hosts from the specified file (`hosts.lst`) instead of the command line |

**Example Output:**
```
10.129.2.18
10.129.2.19
10.129.2.20
```

**Explanation:**  
Only 3 of the 7 hosts from the list responded. The remaining 4 hosts likely have ICMP echo requests blocked by their firewall configuration — Nmap receives no reply and marks them as inactive. This does not mean they are down; it means they are not reachable via the default ICMP method.

---

### Scan Multiple IPs

When testing a small subset of a network, specific IP addresses can be listed directly on the command line.

**Method 1 — List each IP:**
```bash
sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20
```

**Method 2 — Define a range using a dash:**
```bash
sudo nmap -sn -oA tnet 10.129.2.18-20
```

Both commands produce identical results — they scan the three hosts `10.129.2.18`, `10.129.2.19`, and `10.129.2.20`. The dash notation `18-20` tells Nmap to scan the range from `.18` to `.20` in the last octet. This is a convenient shorthand when scanning consecutive IP addresses.

---

### Scan Single IP

Before port scanning a target, confirm it is alive with a host discovery scan.

**Command:**
```bash
sudo nmap 10.129.2.18 -sn -oA host
```

**What Each Part Does:**

| Option | Description |
|---|---|
| `10.129.2.18` | Single target IP address |
| `-sn` | Disables port scanning (host discovery only) |
| `-oA host` | Saves results in all formats with the base name `host` |

**Example Output:**
```
Host is up (0.087s latency).
MAC Address: DE:AD:00:00:BE:EF
```

**Explanation:**  
When `-sn` is used, Nmap automatically uses **ARP ping** first (on local networks) — sending an ARP request to determine the MAC address. If the host replies with an ARP reply, it is marked as alive. On remote networks where ARP is not available, ICMP Echo Requests are used instead.

---

### Packet Trace During Host Discovery

To see exactly what packets Nmap is sending and receiving, use `--packet-trace`:

**Command:**
```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace
```

**What New Flags Do:**

| Option | Description |
|---|---|
| `-PE` | Explicitly requests ICMP Echo Request ping (instead of ARP) |
| `--packet-trace` | Shows all packets sent (`SENT`) and received (`RCVD`) during the scan |

**Example Output:**
```
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
```

**Explanation:**  
Even though `-PE` was specified (ICMP Echo Request), Nmap sends an **ARP request first** when the target is on the same local network. ARP is faster and more reliable for local network host discovery. The `SENT` line shows the ARP broadcast asking who has `10.129.2.18`, and `RCVD` shows the ARP reply confirming the host is alive and revealing its MAC address (`DE:AD:00:00:BE:EF`).

---

### Using --reason to Understand Host Detection

**Command:**
```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --reason
```

**What `--reason` Does:**  
Adds an explanation column to the output showing **why** Nmap determined a host is up or a port is in a certain state.

**Example Output:**
```
Host is up, received arp-response (0.028s latency).
```

**Explanation:**  
The `--reason` flag shows `received arp-response` — confirming the host was determined alive because it responded to an ARP request, not an ICMP echo. This is crucial for understanding what method actually triggered the "host up" result.

---

### Disabling ARP Ping to Force ICMP Echo Requests

**Command:**
```bash
sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping
```

**What `--disable-arp-ping` Does:**  
Prevents Nmap from using ARP for host discovery. Forces it to use the method specified by `-PE` (ICMP Echo Request).

**Example Output:**
```
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128]
```

**Explanation:**  
Now Nmap sends a true **ICMP Echo Request** (type 8, code 0) and receives an **ICMP Echo Reply** (type 0, code 0). The TTL value in the reply (`ttl=128`) can also be used for OS fingerprinting: Windows typically returns TTL=128, Linux typically returns TTL=64.

---

<a name="section-4"></a>
## Section 4 — Host and Port Scanning

### Overview

Once we confirm that a host is alive, the next step is detailed scanning to determine what services are running and what information they expose. We need to gather: open ports and services, service versions, information provided by the services, and the operating system. Understanding how Nmap handles each possible port state is essential for correctly interpreting scan results.

---

### The Six Port States

Nmap can report six different states for any scanned port:

| State | Description |
|---|---|
| `open` | A connection was successfully established to the port. Can be TCP, UDP, or SCTP. |
| `closed` | The target sent back a TCP packet with the **RST** flag — the port is accessible but no service is listening. Also used to confirm a host is alive. |
| `filtered` | Nmap cannot determine if the port is open or closed — no response received, or received an ICMP error code. Usually indicates a firewall is dropping packets. |
| `unfiltered` | Port is accessible but open/closed status cannot be determined. Only seen in TCP ACK scans (`-sA`). |
| `open\|filtered` | No response received — could be open (service not responding to empty probes) or filtered. Common result in UDP scans. |
| `closed\|filtered` | Only seen in IP ID idle scans. Cannot determine if the port is closed or filtered by a firewall. |

---

### Discovering Open TCP Ports

**Default Nmap behavior:**
- Default scan: top **1000 TCP ports** using SYN scan (`-sS`) when run as root
- If NOT root: TCP Connect scan (`-sT`) is used instead
- Port selection options:

| Method | Example | Description |
|---|---|---|
| Single port | `-p 22` | Scan only port 22 |
| Multiple ports | `-p 22,25,80` | Scan ports 22, 25, and 80 |
| Port range | `-p 22-445` | Scan all ports from 22 to 445 |
| Top ports | `--top-ports=10` | Scan the 10 most frequent ports |
| All ports | `-p-` | Scan all 65,535 ports |
| Fast scan | `-F` | Scan top 100 ports |

---

### Scanning Top 10 TCP Ports

**Command:**
```bash
sudo nmap 10.129.2.28 --top-ports=10
```

**What It Does:**  
Scans only the 10 most commonly used TCP ports (as ranked in Nmap's internal database) and reports their state.

**Example Output:**
```
PORT     STATE    SERVICE
21/tcp   closed   ftp
22/tcp   open     ssh
23/tcp   closed   telnet
25/tcp   open     smtp
80/tcp   open     http
110/tcp  open     pop3
139/tcp  filtered netbios-ssn
443/tcp  closed   https
445/tcp  filtered microsoft-ds
3389/tcp closed   ms-wbt-server
```

**Explanation:**  
The scan shows three distinct states: `open` (service is running and accepting connections), `closed` (no service listening, but host is responding), and `filtered` (firewall is blocking — no response). Ports 139 and 445 (NetBIOS/SMB) are `filtered`, suggesting a firewall rule is in place for those ports.

---

### Nmap — Trace the Packets

To understand exactly how Nmap determined a port state, use `--packet-trace` with ICMP, DNS, and ARP disabled for clean output:

**Command:**
```bash
sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping
```

**What Each Flag Does:**

| Flag | Description |
|---|---|
| `-p 21` | Scan only port 21 (FTP) |
| `--packet-trace` | Show all sent and received packets |
| `-Pn` | Disable ICMP Echo requests (assume host is up) |
| `-n` | Disable DNS resolution (faster, no DNS lookups) |
| `--disable-arp-ping` | Disable ARP ping (force IP-based scan) |

**Example Output:**
```
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 seq=0 win=0
PORT   STATE  SERVICE
21/tcp closed ftp
```

---

### Understanding the SENT/RCVD Output

**Request (SENT line):**

| Field | Description |
|---|---|
| `SENT (0.0429s)` | Time the packet was sent |
| `TCP` | Protocol used |
| `10.10.14.2:63090` | Our IP address and source port |
| `10.129.2.28:21` | Target IP and port |
| `S` | **SYN flag** — initiating TCP connection |
| `ttl=56` | Time-to-live value |
| `seq=1699105818` | TCP sequence number |
| `win=1024 mss 1460` | Window size and maximum segment size |

**Response (RCVD line):**

| Field | Description |
|---|---|
| `RCVD (0.0573s)` | Time the response was received |
| `TCP` | Protocol used |
| `10.129.2.28:21` | Source: target IP and port |
| `10.10.14.2:63090` | Destination: our IP and port |
| `RA` | **RST + ACK flags** — port is closed; ACK acknowledges our SYN, RST terminates the session |

---

### Connect Scan (-sT)

The **TCP Connect Scan** completes the full three-way TCP handshake to determine port state:

1. **SYN** → sent to target port
2. **SYN-ACK** → received if port is open
3. **ACK** → sent to complete the handshake
4. Then immediately sends RST to close the connection

**Command:**
```bash
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```

**What `-sT` Does:**  
Performs a **full TCP connection** (three-way handshake). More accurate than SYN scan but less stealthy — the full connection is logged by most services and creates connection logs. Useful when SYN scanning is blocked, or when we need to interact cleanly with services without disruption.

**Example Output:**
```
CONN (0.0385s) TCP localhost > 10.129.2.28:443 => Operation now in progress
CONN (0.0396s) TCP localhost > 10.129.2.28:443 => Connected
PORT    STATE SERVICE REASON
443/tcp open  https   syn-ack
```

**Key Difference: SYN vs. Connect Scan:**

| Aspect | SYN Scan (-sS) | Connect Scan (-sT) |
|---|---|---|
| Handshake | Half-open (SYN only) | Full three-way handshake |
| Stealth | More stealthy | Less stealthy (logged by services) |
| Privilege required | Root/admin | Not required |
| Speed | Faster | Slightly slower |
| IDS detection | Harder to detect | Easily detected |

---

### Filtered Ports — Dropped vs. Rejected

When Nmap marks a port as `filtered`, there are two possible reasons:

**Case 1 — Packet Dropped (no response):**

```bash
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ...
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ...
PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
```

**Explanation:**  
Nmap sent **two SYN packets** but received no response. The firewall is silently **dropping** the packets (not acknowledging them at all). The scan took ~2 seconds because Nmap waited for a response before sending the retry. The default retry count (`--max-retries`) is 10.

**Case 2 — Packet Rejected (ICMP error response):**

```bash
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
```

```
SENT (0.0388s) TCP ... > 10.129.2.28:445 S ...
RCVD (0.0487s) ICMP [10.129.2.28 > ... Port 445 unreachable (type=3/code=3)]
PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
```

**Explanation:**  
Here the firewall actively **rejects** the packet by responding with an **ICMP Type 3, Code 3** message ("Port Unreachable"). The response is almost immediate (< 50ms). Both behaviours result in `filtered`, but rejection gives feedback confirming the firewall is actively handling the traffic.

---

### Discovering Open UDP Ports

UDP is a **stateless protocol** — it does not use a three-way handshake. Nmap sends empty UDP datagrams and waits for a response. Because there is no connection establishment, timeouts are much longer, making UDP scans significantly slower than TCP scans.

**Command (fast scan of top 100 UDP ports):**
```bash
sudo nmap 10.129.2.28 -F -sU
```

| Option | Description |
|---|---|
| `-F` | Fast scan (top 100 ports) |
| `-sU` | Performs a UDP scan |

**Example Output:**
```
PORT     STATE         SERVICE
68/udp   open|filtered dhcpc
137/udp  open          netbios-ns
138/udp  open|filtered netbios-dgm
631/udp  open|filtered ipp
5353/udp open          zeroconf
```

**Understanding UDP Port States:**

| State | Meaning |
|---|---|
| `open` | Application responded to the UDP probe (e.g., port 137/udp returned a response) |
| `open\|filtered` | No response — could be open but application ignoring empty probes, or filtered by firewall |
| `closed` | ICMP Type 3, Code 3 (Port Unreachable) was received — no service listening |

**Tracing a Confirmed Open UDP Port (port 137):**

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason
```

```
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ...
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ...
PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
```

NetBIOS Name Service on port 137 replied with UDP data, confirming it is open.

**Tracing a Closed UDP Port (port 100):**

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason
```

```
SENT UDP ... > 10.129.2.28:100 ...
RCVD ICMP [... Port unreachable (type=3/code=3)]
PORT    STATE  SERVICE REASON
100/udp closed unknown port-unreach ttl 64
```

ICMP Type 3/Code 3 confirms port 100 UDP is closed.

**Tracing an open|filtered UDP Port (port 138):**

```bash
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason
```

```
SENT UDP ... > 10.129.2.28:138 ...
SENT UDP ... > 10.129.2.28:138 ...   (retry)
PORT    STATE         SERVICE
138/udp open|filtered netbios-dgm no-response
```

No response received at all — Nmap resent the probe (retry) and still got nothing. Port is marked `open|filtered`.

---

### Version Scan (-sV)

The `-sV` flag tells Nmap to probe open ports and attempt to identify the **service name, version, and extra details**.

**Command:**
```bash
sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason -sV
```

**What `-sV` Does:**  
After detecting that a port is open, Nmap connects to it and sends specific probes. It compares responses against its database of known service signatures. If a banner is returned, it reads it directly; if not, it tries multiple probe types until a match is found. This significantly increases scan duration but provides rich service information.

**Example Output:**
```
PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu
```

**Explanation:**  
The version scan reveals the port is running **Samba smbd** (versions 3.X - 4.X) on an **Ubuntu** Linux host, in a WORKGROUP environment. This information directly enables searching for vulnerabilities specific to this Samba version range on Ubuntu.

---

<a name="section-5"></a>
## Section 5 — Saving the Results

### Overview of Output Formats

Saving scan results is essential for: documentation, report writing, comparing results from different scans or tools, and future reference. Nmap supports three output formats. Always save results — never rely on terminal output alone. Using `-oA` saves all three formats simultaneously, which is the recommended practice.

---

### Different Formats

| Format Option | Extension | Description |
|---|---|---|
| `-oN filename` | `.nmap` | Normal (human-readable) output, similar to terminal output |
| `-oG filename` | `.gnmap` | Grepable output — easy to parse with `grep`, `awk`, `cut` |
| `-oX filename` | `.xml` | XML output — machine-parseable, can be converted to HTML |
| `-oA filename` | `.nmap` + `.gnmap` + `.xml` | Saves in **all three formats** simultaneously |

---

### Saving in All Formats

**Command:**
```bash
sudo nmap 10.129.2.28 -p- -oA target
```

| Option | Description |
|---|---|
| `10.129.2.28` | Target to scan |
| `-p-` | Scan **all 65,535 ports** |
| `-oA target` | Save results in all formats; each file starts with `target` |

**Resulting files:**
```bash
ls
target.gnmap  target.xml  target.nmap
```

---

### Normal Output (.nmap)

**Command to view:**
```bash
cat target.nmap
```

**Example Content:**
```
# Nmap 7.80 scan initiated Tue Jun 16 12:14:53 2020
Nmap scan report for 10.129.2.28
Host is up (0.053s latency).
PORT   STATE SERVICE
22/tcp open  ssh
25/tcp open  smtp
80/tcp open  http
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)
# Nmap done at Tue Jun 16 12:15:03 2020 -- 1 IP address (1 host up) scanned in 10.22 seconds
```

**Explanation:**  
The `.nmap` file is human-readable, formatted like the terminal output, and includes timestamps and scan metadata. Best format for manual review and inclusion in reports as plain text.

---

### Grepable Output (.gnmap)

**Command to view:**
```bash
cat target.gnmap
```

**Example Content:**
```
Host: 10.129.2.28 ()    Status: Up
Host: 10.129.2.28 ()    Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http///
```

**Explanation:**  
The `.gnmap` format puts all information for each host on a single line. This makes it easy to extract specific fields using tools like `grep`, `awk`, and `cut`. For example: `grep "open" target.gnmap | cut -d" " -f2` to extract all IP addresses with open ports. Widely used in scripts that process Nmap output.

---

### XML Output (.xml)

**Command to view:**
```bash
cat target.xml
```

**Example Content (simplified):**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<nmaprun scanner="nmap" args="nmap -p- -oA target 10.129.2.28">
  <host>
    <address addr="10.129.2.28" addrtype="ipv4"/>
    <ports>
      <port protocol="tcp" portid="22">
        <state state="open" reason="syn-ack"/>
        <service name="ssh"/>
      </port>
    </ports>
  </host>
</nmaprun>
```

**Explanation:**  
XML provides structured, machine-parseable output. It is ideal for importing into other tools, writing scripts that process scan data, or storing results in databases. XML output is also the basis for generating HTML reports.

---

### Style Sheets — Converting XML to HTML

**Command:**
```bash
xsltproc target.xml -o target.html
```

**What It Does:**  
`xsltproc` is an XSLT processor that converts XML documents using a stylesheet. Nmap's XML output references a built-in stylesheet (`nmap.xsl`) that transforms the XML into a clean, structured HTML report viewable in any browser. This is especially useful for sharing scan results with non-technical stakeholders or including in formal pentest reports.

**Explanation of Output:**  
The HTML report shows all open ports in a clearly formatted table with columns for port number, state, service, and version. It includes scan metadata (date, time, command used) and is suitable for distribution to clients.

---

<a name="section-6"></a>
## Section 6 — Service Enumeration

### Service Version Detection

Knowing the exact **application name and version** running on an open port is critical for identifying known CVEs and planning exploits. The recommended workflow is: run a **quick port scan first** (less traffic, less detectable) to get a list of open ports, then run a **targeted version scan** (`-sV`) on those specific ports. Running version detection on all ports from the start generates more traffic and takes longer. Accurate version information enables searching for **precise exploits** that match both the service and the OS of the target.

---

### Full Port Scan with Version Detection

**Command:**
```bash
sudo nmap 10.129.2.28 -p- -sV
```

| Option | Description |
|---|---|
| `10.129.2.28` | Target IP |
| `-p-` | Scan all 65,535 ports |
| `-sV` | Detect service name and version on open ports |

A full port scan takes significant time. During the scan, pressing **[Space Bar]** shows real-time progress:

```
Stats: 0:00:03 elapsed; 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 3.64% done; ETC: 19:45 (0:00:53 remaining)
```

---

### Monitoring Scan Progress with --stats-every

**Command:**
```bash
sudo nmap 10.129.2.28 -p- -sV --stats-every=5s
```

**What `--stats-every=5s` Does:**  
Automatically prints a progress update every 5 seconds (can also use `5m` for minutes). Shows percentage complete and estimated time of completion (ETC). Useful for long-running full-port scans where you want regular updates without pressing Space Bar manually.

---

### Increasing Verbosity with -v

**Command:**
```bash
sudo nmap 10.129.2.28 -p- -sV -v
```

**What `-v` Does:**  
Increases verbosity — Nmap immediately prints each open port as it is **discovered in real time**, rather than waiting for the full scan to complete. This is particularly useful in large scans where you want to start researching the first open ports while the rest of the scan continues.

**Example real-time output:**
```
Discovered open port 995/tcp on 10.129.2.28
Discovered open port 80/tcp on 10.129.2.28
Discovered open port 993/tcp on 10.129.2.28
```

For even more detail, use `-vv` (double verbose).

---

### Banner Grabbing

After a full `-sV` scan completes, the output shows all TCP ports with their service names and versions:

```
PORT      STATE    SERVICE      VERSION
22/tcp    open     ssh          OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
25/tcp    open     smtp         Postfix smtpd
80/tcp    open     http         Apache httpd 2.4.29 (Ubuntu)
110/tcp   open     pop3         Dovecot pop3d
139/tcp   filtered netbios-ssn
143/tcp   open     imap         Dovecot imapd (Ubuntu)
445/tcp   filtered microsoft-ds
993/tcp   open     ssl/imap     Dovecot imapd (Ubuntu)
995/tcp   open     ssl/pop3     Dovecot pop3d
Service Info: Host: inlane; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

**Explanation:**  
Nmap reads service **banners** — identification strings that services send immediately after a connection is established. When banners are unavailable, it uses **signature-based matching** which takes longer. The output reveals the OS (Ubuntu Linux), mail server stack (Postfix + Dovecot), and SSH version — all directly searchable for vulnerabilities.

---

### Manual Banner Grabbing with tcpdump and Netcat

Nmap does not always show the complete banner. Manual interaction with a service often reveals more information.

**Step 1 — Start traffic capture with tcpdump:**
```bash
sudo tcpdump -i eth0 host 10.10.14.2 and 10.129.2.28
```

**What It Does:**  
`tcpdump` captures all network packets on interface `eth0` between our host (`10.10.14.2`) and the target (`10.129.2.28`). The `-i eth0` specifies the network interface to capture on. `host A and B` filters to only show traffic between the two specified hosts.

**Step 2 — Connect to SMTP with Netcat:**
```bash
nc -nv 10.129.2.28 25
```

**What It Does:**  
`nc` (Netcat) connects to port 25 (SMTP) on the target. `-n` disables DNS resolution; `-v` enables verbose mode. After connecting, the SMTP server sends its banner:

```
220 inlane ESMTP Postfix (Ubuntu)
```

**Explanation:**  
The full banner `220 inlane ESMTP Postfix (Ubuntu)` is visible, revealing the Linux distribution (Ubuntu). Nmap showed only `Postfix smtpd` in its output — the OS detail was in the banner but Nmap didn't display it.

---

### Reading tcpdump — The Three-Way Handshake

The tcpdump output shows the complete TCP conversation:

```
1. [SYN]     18:28:07.128564  10.10.14.2.59618 > 10.129.2.28.smtp: Flags [S]
2. [SYN-ACK] 18:28:07.255151  10.129.2.28.smtp > 10.10.14.2.59618: Flags [S.]
3. [ACK]     18:28:07.255281  10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.]
4. [PSH-ACK] 18:28:07.319306  10.129.2.28.smtp > 10.10.14.2.59618: Flags [P.] length 35: SMTP: 220 inlane ESMTP Postfix (Ubuntu)
5. [ACK]     18:28:07.319426  10.10.14.2.59618 > 10.129.2.28.smtp: Flags [.]
```

**Explanation of Each Step:**

| Step | Flags | Description |
|---|---|---|
| 1 | `[S]` — SYN | We initiate the TCP connection request |
| 2 | `[S.]` — SYN-ACK | Target acknowledges and accepts the connection |
| 3 | `[.]` — ACK | We acknowledge — three-way handshake complete |
| 4 | `[P.]` — PSH-ACK | Target **pushes** data (the banner) to us; `PSH` means "send this data now" |
| 5 | `[.]` — ACK | We acknowledge receipt of the banner data |

The `PSH` (Push) flag in step 4 indicates the server is actively sending data. This is the moment the SMTP banner is transmitted.

---

<a name="section-7"></a>
## Section 7 — Nmap Scripting Engine

### Overview of NSE

The **Nmap Scripting Engine (NSE)** extends Nmap's capabilities beyond simple port scanning. NSE allows running Lua-based scripts against target hosts and services to perform tasks ranging from authentication testing to vulnerability identification. There are **14 script categories** built into NSE, each serving a different purpose. Scripts can be found in `/usr/share/nmap/scripts/` on Linux. The NSE is what makes Nmap a comprehensive security assessment tool rather than just a port scanner.

---

### NSE Script Categories

| Category | Description |
|---|---|
| `auth` | Tests for authentication credentials and authentication mechanisms |
| `broadcast` | Discovers hosts by broadcasting on the network; discovered hosts can be automatically added to the scan |
| `brute` | Attempts to brute-force login credentials for the scanned service |
| `default` | Scripts that run when `-sC` is used — a curated set of safe, commonly useful scripts |
| `discovery` | Evaluates and enumerates accessible services and network information |
| `dos` | Checks services for denial-of-service vulnerabilities (use with caution — can harm services) |
| `exploit` | Attempts to exploit known vulnerabilities on the scanned port |
| `external` | Uses external services (databases, APIs) for additional processing |
| `fuzzer` | Sends different/malformed fields to identify unexpected packet handling and potential vulnerabilities |
| `intrusive` | More aggressive scripts that could negatively affect the target system |
| `malware` | Checks if the target system is infected with malware or backdoors |
| `safe` | Defensive, non-intrusive scripts that do not perform destructive access |
| `version` | Extension to service detection for determining more precise version information |
| `vuln` | Identifies specific known vulnerabilities in the detected services |

---

### Running NSE Scripts

**Method 1 — Default Scripts:**
```bash
sudo nmap <target> -sC
```
Runs all scripts in the `default` category — a curated set of safe, informative scripts.

**Method 2 — Specific Category:**
```bash
sudo nmap <target> --script <category>
```
Example: `--script vuln` runs all vulnerability detection scripts.

**Method 3 — Named Scripts:**
```bash
sudo nmap <target> --script <script-name>,<script-name>,...
```
Example: `--script banner,smtp-commands` runs only the banner and smtp-commands scripts.

---

### Nmap — Specifying Scripts

**Command:**
```bash
sudo nmap 10.129.2.28 -p 25 --script banner,smtp-commands
```

| Option | Description |
|---|---|
| `-p 25` | Scan only port 25 (SMTP) |
| `--script banner,smtp-commands` | Run the `banner` and `smtp-commands` NSE scripts |

**Example Output:**
```
PORT   STATE SERVICE
25/tcp open  smtp
|_banner: 220 inlane ESMTP Postfix (Ubuntu)
|_smtp-commands: inlane, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8
```

**Explanation:**
- `banner` script: Read the service banner — reveals the Linux distribution (Ubuntu) and mail server (Postfix)
- `smtp-commands` script: Queried the SMTP server for supported commands. The `VRFY` command (visible in the output) can be used to verify whether specific email addresses / usernames exist on the system — a potential enumeration vector

---

### Nmap — Aggressive Scan (-A)

The `-A` flag combines multiple scan types into one comprehensive scan:

**Command:**
```bash
sudo nmap 10.129.2.28 -p 80 -A
```

**What `-A` Includes:**

| Technique | Flag | Description |
|---|---|---|
| Service Detection | `-sV` | Identifies service name and version |
| OS Detection | `-O` | Fingerprints the operating system |
| Traceroute | `--traceroute` | Maps the network path to the target |
| Default Scripts | `-sC` | Runs all default NSE scripts |

**Example Output:**
```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-generator: WordPress 5.3.4
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: blog.inlanefreight.com

Aggressive OS guesses: Linux 2.6.32 (96%), Linux 3.2 - 4.9 (96%)
```

**Explanation:**  
The aggressive scan revealed: web server is Apache 2.4.29 on Ubuntu, hosting WordPress 5.3.4 at `blog.inlanefreight.com`. The OS is likely Linux (96% confidence). This information enables targeted vulnerability research for WordPress 5.3.4, Apache 2.4.29, and the identified Ubuntu version.

---

### Vulnerability Assessment — Vuln Category

**Command:**
```bash
sudo nmap 10.129.2.28 -p 80 -sV --script vuln
```

| Option | Description |
|---|---|
| `-p 80` | Scan only port 80 (HTTP) |
| `-sV` | Detect service version |
| `--script vuln` | Run all vulnerability-related NSE scripts against the target |

**Example Output:**
```
| http-enum:
|   /wp-login.php: Possible admin folder
|   /readme.html: Wordpress version: 2
|   /: WordPress version: 5.3.4
|_  /wp-login.php: Wordpress login page.
| http-wordpress-users:
| Username found: admin
| vulners:
|   cpe:/a:apache:http_server:2.4.29:
|       CVE-2019-0211   7.2  https://vulners.com/cve/CVE-2019-0211
|       CVE-2018-1312   6.8  https://vulners.com/cve/CVE-2018-1312
```

**Explanation:**  
The vuln scripts found: the WordPress admin login page, WordPress version 5.3.4, a discovered username (`admin`), and multiple CVEs for Apache 2.4.29 (the most critical being CVE-2019-0211 with CVSS score 7.2). This gives a direct actionable list of vulnerabilities to research and potentially exploit.

---

<a name="section-8"></a>
## Section 8 — Performance

### Overview

Scanning performance is critical when dealing with large networks or low-bandwidth environments. Nmap provides multiple tuning options to control how fast scans run, how long timeouts are, and how many packets are sent simultaneously. However, speed always comes at a cost — faster scans may miss hosts or ports. The right balance depends on the network environment and whether being detected is a concern.

---

### Timeouts

**Round-Trip-Time (RTT)** is the time it takes for a packet to travel from Nmap to the target and the response to return. Nmap's default initial RTT timeout is **100ms**. If the RTT is consistent, lowering the timeout speeds up the scan significantly.

**Default Scan (no RTT tuning):**
```bash
sudo nmap 10.129.2.0/24 -F
```
```
Nmap done: 256 IP addresses (10 hosts up) scanned in 39.44 seconds
```

**Optimized RTT Scan:**
```bash
sudo nmap 10.129.2.0/24 -F --initial-rtt-timeout 50ms --max-rtt-timeout 100ms
```

| Option | Description |
|---|---|
| `--initial-rtt-timeout 50ms` | Sets the starting RTT timeout to 50ms (default: 100ms) |
| `--max-rtt-timeout 100ms` | Sets the maximum RTT timeout to 100ms |

```
Nmap done: 256 IP addresses (8 hosts up) scanned in 12.29 seconds
```

**Trade-off:**  
The optimized scan ran in ~12 seconds vs. ~39 seconds. However, **2 fewer hosts were found**. Setting RTT too low means Nmap gives up waiting for slow responses — hosts that respond slowly are missed. Always verify findings from optimized scans with a slower, more thorough scan.

---

### Max Retries

By default, Nmap retries each unresponsive port up to **10 times** (`--max-retries 10`). Reducing retries speeds up scans but risks missing ports on slow or inconsistently responding hosts.

**Default Scan — Count Open Ports:**
```bash
sudo nmap 10.129.2.0/24 -F | grep "/tcp" | wc -l
```
```
23
```

**Reduced Retries (0 retries):**
```bash
sudo nmap 10.129.2.0/24 -F --max-retries 0 | grep "/tcp" | wc -l
```
```
21
```

**Explanation:**  
With `--max-retries 0`, Nmap does not resend any packet that didn't get a response — it just skips that port. Result: 2 fewer open ports found. Useful for very quick overview scans but should not be used for thorough assessments.

---

### Rates — Controlling Packet Speed

During **white-box penetration tests** (where we are whitelisted), we can control the packet sending rate to dramatically speed up scans without worrying about detection.

**Default Scan:**
```bash
sudo nmap 10.129.2.0/24 -F -oN tnet.default
```
```
Nmap done: 256 IP addresses (10 hosts up) scanned in 29.83 seconds
```

**Optimized with --min-rate:**
```bash
sudo nmap 10.129.2.0/24 -F -oN tnet.minrate300 --min-rate 300
```

| Option | Description |
|---|---|
| `--min-rate 300` | Sends a minimum of 300 packets per second |
| `-oN tnet.minrate300` | Save results in normal format to `tnet.minrate300` |

```
Nmap done: 256 IP addresses (10 hosts up) scanned in 8.67 seconds
```

**Result Comparison:**
```bash
cat tnet.default | grep "/tcp" | wc -l       # 23 ports
cat tnet.minrate300 | grep "/tcp" | wc -l    # 23 ports
```

Both found the same 23 open ports, but the rate-optimized scan completed in ~8.67 seconds vs ~29.83 seconds. In white-box tests, `--min-rate` is safe to use aggressively.

---

### Timing Templates (-T)

For situations where manual tuning is not practical (e.g., black-box tests), Nmap provides **six predefined timing templates**:

| Template | Name | Use Case |
|---|---|---|
| `-T 0` | `paranoid` | Extremely slow; evades most IDS; suited for stealth |
| `-T 1` | `sneaky` | Very slow; avoids some IDS detection |
| `-T 2` | `polite` | Slow; reduces bandwidth usage |
| `-T 3` | `normal` | **Default** — balanced speed and reliability |
| `-T 4` | `aggressive` | Fast; assumes reliable network; may miss some ports |
| `-T 5` | `insane` | Very fast; sacrifices accuracy; may get blocked |

**Default Scan vs. Insane Scan:**
```bash
sudo nmap 10.129.2.0/24 -F -oN tnet.default        # 32.44 seconds, 23 ports
sudo nmap 10.129.2.0/24 -F -oN tnet.T5 -T 5        # 18.07 seconds, 23 ports
```

**Explanation:**  
`-T 5` (insane) completed in ~18 seconds vs ~32 seconds for the default, while finding the same 23 ports. In environments where detection is not a concern (authorized tests with whitelisting), `-T 4` or `-T 5` significantly speeds up large-scale network scanning.

---

<a name="section-9"></a>
## Section 9 — Firewall and IDS/IPS Evasion

### Firewalls

A **firewall** is a security measure designed to block unauthorized connection attempts from external networks. Firewalls work at both the software and hardware level and inspect network traffic as it passes through. Based on configured rules, a firewall decides to: **pass** (allow) the packet, **drop** (silently discard) the packet, or **reject** (explicitly deny and send an error response) the packet. Dropped packets leave no trace on the sender's side, while rejected packets generate an ICMP or TCP RST response. Understanding this difference helps identify firewall behavior during scans.

---

### IDS/IPS

- **IDS (Intrusion Detection System):** A software-based passive monitoring system that analyzes network traffic for suspicious patterns and alerts administrators when potential attacks are detected. IDS does not block traffic — it only detects and reports.
- **IPS (Intrusion Prevention System):** Complements IDS by actively taking defensive measures when a potential attack is detected — such as automatically blocking a source IP or dropping specific connections. IPS works based on pattern matching and signatures (e.g., detecting a port scan pattern and blocking the source).

---

### Determine Firewalls and Their Rules

When a port is `filtered`, the firewall may be **dropping** or **rejecting** packets:

**Dropped** — Nmap sends multiple retries with no response (slow scan):
```bash
sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn
# Port 139/tcp filtered — two SYN packets sent, no response
```

**Rejected** — Nmap receives an ICMP error immediately:
```bash
sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn
# RCVD ICMP Port 445 unreachable (type=3/code=3)
# Port 445/tcp filtered — fast response with ICMP rejection
```

**ICMP Error Types That Indicate Rejection:**
- `Net Unreachable` — Network not reachable
- `Net Prohibited` — Network access prohibited
- `Host Unreachable` — Target host not reachable
- `Host Prohibited` — Host access prohibited
- `Port Unreachable` (Type 3, Code 3) — Specific port not reachable (most common)
- `Proto Unreachable` — Protocol not supported

---

### SYN-Scan vs. ACK-Scan

The TCP ACK scan (`-sA`) is harder for firewalls and IDS/IPS to detect than SYN or Connect scans, because ACK packets look like responses to legitimate established connections.

**SYN-Scan:**
```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sS -Pn -n --disable-arp-ping --packet-trace
```

```
PORT   STATE    SERVICE
21/tcp filtered ftp       (ICMP Port Unreachable received)
22/tcp open     ssh       (SYN-ACK received)
25/tcp filtered smtp      (no response — dropped)
```

**ACK-Scan:**
```bash
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n --disable-arp-ping --packet-trace
```

```
PORT   STATE      SERVICE
21/tcp filtered   ftp     (ICMP Port Unreachable — firewall rejecting)
22/tcp unfiltered ssh     (RST received — port exists, accessible)
25/tcp filtered   smtp    (no response — firewall dropping)
```

**Key Difference:**
- In the ACK scan, port 22 shows as **`unfiltered`** (not `open`) — ACK scans can only determine if a port is accessible, not if a service is listening. The RST response from port 22 confirms the firewall is not blocking ACK packets to that port.
- Firewalls typically block incoming SYN packets (new connection attempts) but may pass ACK packets (which appear to be part of existing connections).

---

### Detect IDS/IPS

Unlike firewalls, IDS/IPS systems are passive monitors — they do not block by default (IDS) or block automatically based on rules (IPS). Detecting them requires behavioral inference:

- **Use multiple VPS (Virtual Private Servers) with different IPs** to scan from. If one IP gets blocked mid-engagement, the IDS/IPS has detected activity and the administrator acted.
- **Aggressive scanning behavior** (like rapid port scanning of a single port) can trigger IPS rules and result in a ban.
- **Monitor response consistency** — if previously reachable services suddenly stop responding, an IPS may have auto-blocked your IP.
- **Quieter, slower scans** (`-T 1`, `-T 2`) reduce the chance of triggering rate-based IDS/IPS rules.

---

### Decoys (-D)

Decoy scanning disguises the true source of the scan by inserting **fake (spoofed) source IP addresses** alongside the real IP address in packet headers. The target sees scan packets from multiple IPs, making it difficult to identify which one is the real attacker.

**Command:**
```bash
sudo nmap 10.129.2.28 -p 80 -sS -Pn -n --disable-arp-ping --packet-trace -D RND:5
```

| Option | Description |
|---|---|
| `-D RND:5` | Generate **5 random decoy IP addresses** alongside the real IP |

**Example SENT packets:**
```
SENT TCP 102.52.161.59:59289 > 10.129.2.28:80 S   (decoy 1)
SENT TCP 10.10.14.2:59289 > 10.129.2.28:80 S       (REAL IP — position 2)
SENT TCP 210.120.38.29:59289 > 10.129.2.28:80 S    (decoy 2)
SENT TCP 191.6.64.171:59289 > 10.129.2.28:80 S     (decoy 3)
SENT TCP 184.178.194.209:59289 > 10.129.2.28:80 S  (decoy 4)
SENT TCP 43.21.121.33:59289 > 10.129.2.28:80 S     (decoy 5)
```

**Important Notes:**
- Decoys must be **live/reachable IP addresses** — if decoys are unreachable, SYN-flood protection on the target may kick in
- ISPs and routers may **filter spoofed packets** from different network ranges
- Decoys work with SYN, ACK, ICMP, and OS detection scans

---

### Testing Firewall Rule with Different Source IP (-S)

When specific subnets are blocked, using a different source IP may bypass the restriction.

**Blocked (default source IP):**
```bash
sudo nmap 10.129.2.28 -n -Pn -p445 -O
```
```
PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
```

**Bypassed (different source IP from same network):**
```bash
sudo nmap 10.129.2.28 -n -Pn -p 445 -O -S 10.129.2.200 -e tun0
```

| Option | Description |
|---|---|
| `-S 10.129.2.200` | Spoof source IP address to `10.129.2.200` |
| `-e tun0` | Send all packets through the `tun0` interface (VPN tunnel) |
| `-O` | OS detection scan |

```
PORT    STATE SERVICE
445/tcp open  microsoft-ds
Aggressive OS guesses: Linux 2.6.32 (96%)
```

**Explanation:**  
By sourcing the scan from `10.129.2.200` (a different address in the same network), the firewall rule blocking port 445 from our real IP was bypassed. The port now shows as `open`. This technique is useful when the firewall has IP-based whitelist/blacklist rules.

---

### DNS Proxying

By default, Nmap makes DNS reverse-resolution queries over **UDP port 53**. Modern networks increasingly also use **TCP port 53** for DNS (due to IPv6, DNSSEC, and larger response sizes). This is important because many firewalls allow DNS traffic (port 53) through without much scrutiny.

**Two DNS Proxying techniques:**

**1. Custom DNS Server (`--dns-server`):**
```bash
sudo nmap <target> --dns-server <company DNS server>
```
Using the target company's internal DNS server (found during enumeration) routes DNS queries through the trusted internal infrastructure, sometimes bypassing external DNS filtering.

**2. Source Port 53 (`--source-port`):**

**Filtered port with normal source:**
```bash
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace
```
```
PORT      STATE    SERVICE
50000/tcp filtered ibm-db2
```

**Same port bypassed using DNS source port:**
```bash
sudo nmap 10.129.2.28 -p50000 -sS -Pn -n --disable-arp-ping --packet-trace --source-port 53
```
```
PORT      STATE SERVICE
50000/tcp open  ibm-db2
```

**What `--source-port 53` Does:**  
Sets the **source port** of all scan packets to **53** (DNS). Some firewall rules explicitly allow inbound DNS responses (port 53 source) without strict inspection. If the administrator configured a rule like "allow traffic from source port 53," our scan packets masquerade as DNS responses and pass through.

---

### Connect To The Filtered Port Using Ncat

After discovering that `--source-port 53` bypasses the firewall, we can use Ncat (the enhanced Netcat) to interact with the open service:

**Command:**
```bash
ncat -nv --source-port 53 10.129.2.28 50000
```

| Option | Description |
|---|---|
| `ncat` | Ncat — enhanced version of Netcat from the Nmap project |
| `-n` | Disable DNS resolution |
| `-v` | Verbose output |
| `--source-port 53` | Use port 53 as the source port for the connection |
| `10.129.2.28 50000` | Connect to the target on port 50000 |

**Example Output:**
```
Ncat: Version 7.80
Ncat: Connected to 10.129.2.28:50000.
220 ProFTPd
```

**Explanation:**  
The connection succeeded using source port 53. The banner `220 ProFTPd` reveals the service behind port 50000 is **ProFTPd** (an FTP server). This information would not have been accessible without bypassing the firewall rule. The `--source-port` trick works because the firewall trusts port-53-sourced traffic (treating it as DNS-related).

---

<a name="section-10"></a>
## Section 10 — Firewall and IDS/IPS Evasion — Easy Lab

### Scenario

A company hired us to test their IT security defenses, including their IDS and IPS systems. The client will make improvements to their IDS/IPS after each successful test. The lab provides a machine protected by IDS/IPS systems. A status page is accessible at:

```
http://<target>/status.php
```

This page shows the current **number of alerts triggered**. If we generate too many alerts, our IP will be banned — we must scan as **quietly as possible**. The key to this lab is using the evasion techniques from Section 9: low timing templates (`-T 1` or `-T 2`), source port spoofing (`--source-port 53`), decoys (`-D`), and disabling unnecessary discovery methods. Monitor the alert count at the status page between scans to gauge detection impact.

---

<a name="section-11"></a>
## Section 11 — Firewall and IDS/IPS Evasion — Medium Lab

### Scenario

This lab increases the difficulty by implementing stricter IDS/IPS rules than the Easy Lab. The client has improved their security systems based on findings from the previous test. The same status page (`http://<target>/status.php`) is used to monitor alert counts. More advanced evasion techniques are required — such as packet fragmentation, slower timing templates, and combining decoys with source port spoofing. The goal is to identify specific services on the target without triggering a ban. Use everything learned in Section 9 and adapt based on which techniques trigger alerts.

---

<a name="section-12"></a>
## Section 12 — Firewall and IDS/IPS Evasion — Hard Lab

### Scenario

The hardest lab — the client has implemented the most aggressive IDS/IPS rules based on the previous two tests. Every scan technique must be carefully chosen and combined to stay under the alert threshold. Techniques to consider: extreme timing (`-T 0`), source port DNS (`--source-port 53`), specific IP decoys (not random, to ensure decoys are alive), disabling all automatic discovery (`-Pn -n --disable-arp-ping`), and scanning one port at a time. The status page (`http://<target>/status.php`) remains the real-time feedback mechanism. Patience and methodical, slow enumeration is the only way to succeed without being banned.

---

<a name="cheatsheet"></a>
## Nmap Cheatsheet — Quick Reference

### Scanning Options

| Nmap Option | Description |
|---|---|
| `10.10.10.0/24` | Target network range |
| `-sn` | Disables port scanning (host discovery only) |
| `-Pn` | Disables ICMP Echo Requests (skip host discovery, treat all hosts as up) |
| `-n` | Disables DNS resolution |
| `-PE` | Performs ping scan using ICMP Echo Requests |
| `--packet-trace` | Shows all packets sent and received during the scan |
| `--reason` | Displays the reason for a specific result (why a port has a certain state) |
| `--disable-arp-ping` | Disables ARP ping requests |
| `--top-ports=<num>` | Scans the specified number of most frequent ports |
| `-p-` | Scan all 65,535 ports |
| `-p 22-110` | Scan all ports between 22 and 110 |
| `-p 22,25` | Scan only the specified ports 22 and 25 |
| `-F` | Scans the top 100 ports (fast scan) |
| `-sS` | Performs TCP SYN scan (half-open, stealthy, requires root) |
| `-sT` | Performs TCP Connect scan (full handshake, no root required) |
| `-sA` | Performs TCP ACK scan (for firewall rule determination) |
| `-sU` | Performs UDP scan |
| `-sV` | Scans open ports for service names and versions |
| `-sC` | Runs default NSE scripts (equivalent to `--script=default`) |
| `--script <script>` | Runs specified NSE script(s) by name or category |
| `-O` | Performs OS detection scan |
| `-A` | Performs OS detection + service detection + traceroute + default scripts |
| `-D RND:5` | Uses 5 random decoy IP addresses to disguise the real scan source |
| `-e <interface>` | Specifies the network interface to use for the scan |
| `-S 10.10.10.200` | Spoofs the source IP address to `10.10.10.200` |
| `-g 53` | Sets source port to 53 (short form of `--source-port`) |
| `--source-port 53` | Uses port 53 as the source port (bypasses DNS-allowing firewall rules) |
| `--dns-server <ns>` | Uses the specified name server for DNS resolution |

---

### Output Options

| Nmap Option | Description |
|---|---|
| `-oA filename` | Saves results in **all three formats** (`.nmap`, `.gnmap`, `.xml`) |
| `-oN filename` | Saves results in **normal** (human-readable) format as `.nmap` |
| `-oG filename` | Saves results in **grepable** format as `.gnmap` |
| `-oX filename` | Saves results in **XML** format as `.xml` |
| `xsltproc target.xml -o target.html` | Converts XML output to a readable HTML report |

---

### Performance Options

| Nmap Option | Description |
|---|---|
| `--max-retries <num>` | Sets the maximum number of retries for unresponsive ports (default: 10) |
| `--stats-every=5s` | Prints scan progress every 5 seconds |
| `-v` | Increases verbosity (shows open ports as discovered) |
| `-vv` | Double verbose — shows even more detail |
| `--initial-rtt-timeout 50ms` | Sets the initial RTT timeout (lower = faster, may miss slow hosts) |
| `--max-rtt-timeout 100ms` | Sets the maximum RTT timeout |
| `--min-rate 300` | Sends at least 300 packets per second |
| `-T 0` | Timing: **paranoid** — extremely slow, maximum stealth |
| `-T 1` | Timing: **sneaky** — very slow, avoids some IDS |
| `-T 2` | Timing: **polite** — slow, reduces bandwidth impact |
| `-T 3` | Timing: **normal** — default balanced setting |
| `-T 4` | Timing: **aggressive** — fast, assumes reliable network |
| `-T 5` | Timing: **insane** — very fast, may miss ports |

---

### Useful Command Combinations

```bash
# Quick host discovery of a full subnet
sudo nmap 10.129.2.0/24 -sn -oA subnet_hosts

# Full port scan + version detection + save output
sudo nmap 10.129.2.28 -p- -sV -oA full_scan

# Aggressive all-in-one scan
sudo nmap 10.129.2.28 -A -oA aggressive_scan

# Stealth SYN scan with packet trace (debugging)
sudo nmap 10.129.2.28 -sS -Pn -n --disable-arp-ping --packet-trace

# Firewall evasion with source port 53
sudo nmap 10.129.2.28 -sS -Pn -n --disable-arp-ping --source-port 53

# Decoy scan with 5 random decoys
sudo nmap 10.129.2.28 -sS -D RND:5 -Pn -n

# NSE vulnerability scan on specific port
sudo nmap 10.129.2.28 -p 80 -sV --script vuln

# UDP scan of top 100 ports
sudo nmap 10.129.2.28 -sU -F

# Scan from hosts list file
sudo nmap -sn -oA scan_results -iL hosts.lst

# ACK scan to detect firewall rules
sudo nmap 10.129.2.28 -p 21,22,25 -sA -Pn -n

# Rate-optimized scan for white-box tests
sudo nmap 10.129.2.0/24 -F --min-rate 300 -oN fast_scan.nmap

# Convert XML output to HTML report
xsltproc scan.xml -o report.html

# Connect to filtered port using DNS source port bypass
ncat -nv --source-port 53 10.129.2.28 50000
```

---

*End of Network Enumeration with Nmap — HTB Academy Structured Notes*

> **Sections covered:** 1 through 12 | **Total sections:** 12/12 | All headings, subheadings, commands, and steps preserved.
