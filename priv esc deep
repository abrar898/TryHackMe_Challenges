Linux Privilege Escalation - My Notes
6. Privileged Groups - LXD
Key Concept
LXD is a Linux container manager.
Users in the lxd group can create containers.
A privileged container can access the host filesystem.
This can lead to root access.
Enumeration
id

Look for:

lxd

Example:

uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)
What I Do Next
Check if lxc is installed.
which lxc
Import a container image.
Create a privileged container.
lxc init alpine r00t -c security.privileged=true
Mount host filesystem.
lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true
Start container.
lxc start r00t
Get shell.
lxc exec r00t /bin/sh
Browse host files.
cd /mnt/root
Why It Works
Normal Container
Root ≠ Host Root

Privileged Container
Root = Host Root
Memory Trick
lxd Group
     ↓
Create Container
     ↓
Mount Host Filesystem
     ↓
Read/Modify Host Files
     ↓
Root
7. Privileged Groups - Docker
Key Concept
Docker users can create containers.
Containers can mount host directories.
Mounted directories can expose root files.
Enumeration
id

Look for:

docker
Example

Mount host root directory:

docker run -v /root:/mnt -it ubuntu

Inside container:

cd /mnt

Now viewing:

Host's /root directory
Interesting Files
/root/.ssh/id_rsa
/root/.ssh/authorized_keys
/etc/shadow
Memory Trick
Docker Group
     ↓
Create Container
     ↓
Mount Host Folder
     ↓
Access Root Files
8. Privileged Groups - Disk
Key Concept
Disk group can directly access disks.
Members can read filesystem data from disk devices.
Enumeration
id

Look for:

disk
Interesting Devices
ls -l /dev/sd*

Examples:

/dev/sda
/dev/sda1
Why Dangerous
Normal User
      ↓
Permissions
      ↓
Filesystem

Disk Group User
      ↓
Direct Disk Access
      ↓
Bypass Filesystem Restrictions
Common Tool
debugfs
Memory Trick
disk Group
     ↓
Access Disk Directly
     ↓
Read Sensitive Files
9. Privileged Groups - ADM
Key Concept
Members can read logs in:
/var/log
Usually not direct root.
Useful for credential hunting.
Enumeration
id

Look for:

adm
Interesting Logs
/var/log/auth.log
/var/log/syslog
/var/log/apache2/
/var/log/nginx/
Look For
Usernames
Passwords
API Keys
Cron Jobs
Internal Paths
Database Credentials
Memory Trick
adm Group
     ↓
Read Logs
     ↓
Find Secrets
     ↓
Possible Privilege Escalation
10. Linux Capabilities
Key Concept

Linux capabilities split root privileges into smaller permissions.

Instead of:

Root Access

Linux gives:

Specific Privilege

to a program.

Example

Allow a program to bind to privileged ports:

sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
Why Dangerous

A dangerous capability on a powerful binary may lead to privilege escalation.

Enumeration

Check one binary:

getcap /usr/bin/vim.basic

Check all binaries:

find / -type f -exec getcap {} \; 2>/dev/null

or

getcap -r / 2>/dev/null
Memory Trick
Capabilities
      ↓
Small Pieces Of Root
      ↓
Dangerous Binary
      ↓
Privilege Escalation
11. Capability - cap_dac_override
Key Concept

Bypasses Linux file permissions.

Normally:

cat /etc/shadow

Result:

Permission denied

With:

cap_dac_override

Permission checks are ignored.

Enumeration
getcap -r / 2>/dev/null

Example:

/usr/bin/vim.basic cap_dac_override=eip
What I Do Next
Find binary with capability.
Check if binary can edit/read files.

Example:

/usr/bin/vim.basic /etc/passwd
Attempt to read or modify restricted files.
Why Dangerous
File Permission Checks
          ↓
Ignored
          ↓
Read/Write Root Files
Memory Trick
DAC Override
      ↓
Ignore Permissions
      ↓
Read/Write Anything
12. Capability - cap_setuid
Key Concept

Allows a process to change its User ID.

Why Dangerous

Can change:

UID=1000

to:

UID=0

(root)

Memory Trick
cap_setuid
      ↓
Change UID
      ↓
Become Root
13. Capability - cap_setgid
Key Concept

Allows a process to change Group ID.

Why Dangerous

Can change:

GID=1000

to:

GID=0

(root group)

Memory Trick
cap_setgid
      ↓
Change GID
      ↓
Root Group Access
14. Capability - cap_sys_admin
Key Concept

One of the most powerful capabilities.

Provides many administrative privileges.

Examples
Mount Filesystems
System Administration
Namespace Operations
Configuration Changes
Why Dangerous

Often called:

"The New Root"

because it provides many root-like actions.

Memory Trick
cap_sys_admin
       ↓
Mini Root
       ↓
Many Privileged Actions
15. Capability Values
Key Concept

Capability values determine how the capability behaves.

"="
Capability Exists
No Privileges Granted

Example:

cap_net_bind_service=
"+ep"
e = Effective

Capability active immediately.

p = Permitted

Program allowed to use capability.

Example:

cap_dac_override=ep
"+ei"
e = Effective

Active immediately.

i = Inheritable

Child processes inherit capability.

"+p"

Only permitted.

Not automatically effective.

Memory Trick
e = Effective
i = Inheritable
p = Permitted
16. /etc/passwd Abuse via cap_dac_override
Key Concept

If a binary can bypass permissions, it may edit:

/etc/passwd
Original Entry
root:x:0:0:root:/root:/bin/bash
Meaning Of x
Password Stored In
/etc/shadow
Attack Idea

Change:

root:x:0:0:root:/root:/bin/bash

to:

root::0:0:root:/root:/bin/bash
Result

Root account has no password.

Then:

su root

may grant root access.

Attack Flow
Find Capability
      ↓
Binary Can Edit Files
      ↓
Modify /etc/passwd
      ↓
su root
      ↓
Root Shell
Quick Exam Revision
LXD
id
↓
lxd Group?
↓
Privileged Container
↓
Mount Host Filesystem
↓
Root
Docker
id
↓
docker Group?
↓
Mount Host Folder
↓
Access Root Files
Disk
id
↓
disk Group?
↓
Direct Disk Access
↓
Sensitive Files
ADM
id
↓
adm Group?
↓
Read Logs
↓
Find Secrets
Capabilities
getcap -r /
↓
Dangerous Capability?
↓
Abuse Binary
↓
Privilege Escalation
cap_dac_override
Ignore Permissions
↓
Read/Write Protected Files
cap_setuid
Change UID
↓
UID 0
↓
Root
cap_setgid
Change GID
↓
GID 0
cap_sys_admin
Mini Root
↓
Administrative Actions

# Linux Privilege Escalation - My Notes

---

# 1. SUID Abuse

## Key Concept

* SUID means a file runs with the permissions of its owner.
* Most of the time the owner is `root`.

## Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

## What I Do Next

1. Find SUID files.
2. Check owner:

```bash
ls -l /path/to/file
```

3. Search the binary on GTFOBins.
4. If GTFOBins has a SUID technique, test it.
5. If owner is root and exploit works → root privileges.

## If Not On GTFOBins

Check:

```bash
file binary
strings binary
ldd binary
```

Look for:

* PATH Hijacking
* Shared Library Hijacking
* Command Execution
* Writable Files
* Known Vulnerabilities

## Memory Trick

```text
SUID = Run as Owner
GTFOBins = How to abuse it
```

---

# 2. SGID Abuse

## Key Concept

* SGID means a file runs with the permissions of its group.
* User temporarily becomes a member of that group.

## Enumeration

```bash
find / -perm -2000 -type f 2>/dev/null
```

or

```bash
find / -uid 0 -perm -6000 -type f 2>/dev/null
```

## What I Do Next

1. Find SGID binaries.
2. Check group owner.

```bash
ls -l binary
```

3. Search on GTFOBins.
4. Look for file access or command execution.

## Memory Trick

```text
SUID = User
SGID = Group
```

---

# 3. Sudo Rights Abuse

## Key Concept

A user can run specific commands as root.

## Enumeration

```bash
sudo -l
```

## What I Do Next

1. Check allowed commands.
2. Search command on GTFOBins.
3. Look for Sudo section.
4. If command allows shell execution, file write, command execution, etc. → privilege escalation.

## Example

```bash
(root) NOPASSWD: /usr/sbin/tcpdump
```

Search:

```text
GTFOBins → tcpdump
```

Check for shell execution methods.

## Memory Trick

```text
sudo -l
    ↓
Allowed command
    ↓
GTFOBins
    ↓
Root shell?
```

---

# 4. NOPASSWD Abuse

## Key Concept

Command can be executed as root without entering password.

## Example

```bash
(root) NOPASSWD: /usr/bin/vim
```

## What I Do Next

1. Run:

```bash
sudo -l
```

2. Find NOPASSWD entries.
3. Search GTFOBins.
4. Test privilege escalation technique.

## Memory Trick

```text
NOPASSWD
    ↓
No password required
    ↓
GTFOBins
```

---

# 5. GTFOBins

## Key Concept

GTFOBins is a database of Linux binaries that can be abused.

## Common Uses

* Shell Spawn
* Sudo Abuse
* SUID Abuse
* File Read
* File Write
* Reverse Shell

## What I Do Next

When I find:

```text
find
vim
less
awk
python
tar
bash
nmap
tcpdump
```

Search:

```text
GTFOBins + binary name
```

## Memory Trick

```text
Interesting Binary
      ↓
GTFOBins
      ↓
Privilege Escalation?
```

---

# 6. PATH Abuse

## Key Concept

Linux searches directories in PATH from left to right.

## Enumeration

```bash
echo $PATH
```

## Look For

Scripts running as root containing:

```bash
tar
cp
find
chmod
chown
```

instead of:

```bash
/usr/bin/tar
/bin/cp
```

## Then Check

```bash
ls -ld directory
```

for each PATH directory.

## Vulnerable If

```text
Command uses NO absolute path
+
Writable PATH directory
```

## Memory Trick

```text
Check Script
     ↓
No Full Path?
     ↓
Check PATH
     ↓
Writable Directory?
     ↓
PATH Abuse
```

---

# 7. Absolute Path vs Relative Path

## Safe

```bash
/usr/bin/tar
```

Linux executes exactly:

```text
/usr/bin/tar
```

No searching.

## Dangerous

```bash
tar
```

Linux searches PATH.

Example:

```text
/tmp
/usr/bin
/bin
```

First match wins.

## Memory Trick

```text
Absolute Path
     ↓
Safe

No Absolute Path
     ↓
Search PATH
     ↓
First Match Wins
```

---

# 8. Shared Library Hijacking

## Key Concept

Program loads external libraries.

## Enumeration

```bash
ldd binary
```

Look for:

```text
not found
```

or writable library paths.

## What I Do Next

1. Check loaded libraries.

```bash
ldd binary
```

2. Find writable location.
3. Replace or create malicious library.
4. Execute binary.

## Memory Trick

```text
Binary
   ↓
Loads Library
   ↓
Can I Control Library?
   ↓
Library Hijack
```

---

# 9. Wildcard Abuse

## Key Concept

Depends on:

```text
*
```

(Asterisk Wildcard)

## Look For

```bash
tar *
zip *
rsync *
```

## Requirements

```text
Wildcard (*)
+
Writable directory
+
Runs as root
```

## Attack Idea

Create files whose names look like command-line options.

When `*` expands, program treats filenames as options.

## Memory Trick

```text
Wildcard Abuse
=
Asterisk (*)
+
Writable Directory
```

---

# 10. Cron Job Abuse

## Key Concept

A cron job automatically runs commands/scripts.

## Enumeration

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.*
```

Useful:

```bash
pspy64
```

## Look For

```text
Writable script
Writable cron file
Misconfigured cron.d entry
```

## Example

```bash
backup.sh
```

runs as root every minute.

If writable:

```bash
echo "my command" >> backup.sh
```

Root executes it.

## Memory Trick

```text
Find Cron
     ↓
Runs As Root?
     ↓
Script Writable?
     ↓
Cron Abuse
```

---

# 11. Custom SUID Binary Analysis

## Key Concept

Custom binaries usually won't be on GTFOBins.

## Enumeration

```bash
file binary
```

```bash
strings binary
```

```bash
ldd binary
```

## Look For

```text
system()
popen()
exec()
tar
cp
chmod
chown
```

Also:

```text
/tmp
/home
config files
```

## Memory Trick

```text
Custom Binary
      ↓
strings
      ↓
Interesting Commands?
      ↓
Exploit Path
```

---

# 12. Tcpdump Sudo Abuse

## Key Concept

Tcpdump can execute a command using:

```bash
-z
```

option.

## Enumeration

```bash
sudo -l
```

Example:

```bash
(root) NOPASSWD: /usr/sbin/tcpdump
```

## What I Do Next

1. Search GTFOBins.
2. Check tcpdump options.
3. Look for command execution.
4. Abuse postrotate-command.

## Memory Trick

```text
sudo tcpdump
      ↓
-z option
      ↓
Execute Command
      ↓
Root Shell
```

---

# Linux PrivEsc Enumeration Checklist

## Identity

```bash
id
whoami
hostname
```

---

## Sudo

```bash
sudo -l
```

---

## SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

---

## SGID

```bash
find / -perm -2000 -type f 2>/dev/null
```

---

## Capabilities

```bash
getcap -r / 2>/dev/null
```

---

## Cron Jobs

```bash
cat /etc/crontab
```

```bash
ls -la /etc/cron.*
```

---

## PATH

```bash
echo $PATH
```

---

## Writable Files

```bash
find / -writable 2>/dev/null
```

---

## Interesting Binaries

```bash
find
vim
less
nano
awk
perl
python
tar
cp
bash
nmap
tcpdump
screen
```

Search all on:

```text
GTFOBins
```

---

# Quick Exam Revision

## SUID

```text
Find SUID
↓
Check Owner
↓
GTFOBins
```

## SGID

```text
Find SGID
↓
Check Group
↓
GTFOBins
```

## Sudo

```text
sudo -l
↓
GTFOBins
↓
Abuse Allowed Command
```

## PATH

```text
No Absolute Path
+
Writable PATH Directory
```

## Shared Library

```text
Binary
↓
Library
↓
Writable?
↓
Hijack
```

## Wildcard

```text
*
+
Writable Directory
```

## Cron

```text
Writable Script
+
Runs As Root
```

## GTFOBins

```text
Interesting Binary
↓
GTFOBins
↓
Privilege Escalation
```# Linux Privilege Escalation - My Notes

## 1. SUID Abuse

### Key Concept

* SUID means a file runs with the permissions of its owner.
* Most of the time the owner is `root`.

### Enumeration

```bash
find / -perm -4000 -type f 2>/dev/null
```

### What I Do Next

1. Find SUID files.
2. Check owner:

```bash
ls -l /path/to/file
```

3. Search the binary on GTFOBins.
4. If GTFOBins has a SUID technique, test it.
5. If owner is root and exploit works → root privileges.

### Memory Trick

```text
SUID = Run as Owner
GTFOBins = How to abuse it
```

---

## 2. Sudo Rights Abuse

### Key Concept

A user can run specific commands as root.

### Enumeration

```bash
sudo -l
```

### What I Do Next

1. Check allowed commands.
2. Search command on GTFOBins.
3. Look for Sudo section.
4. If command allows shell execution, file write, command execution, etc. → privilege escalation.

### Memory Trick

```text
sudo -l
    ↓
Allowed command
    ↓
GTFOBins
    ↓
Root shell?
```

---

## 3. PATH Abuse

### Key Concept

Linux searches directories in PATH from left to right.

### Enumeration

```bash
echo $PATH
```

### Look For

Scripts running as root containing:

```bash
tar
cp
find
chmod
chown
```

instead of:

```bash
/usr/bin/tar
/bin/cp
```

### Then Check

```bash
ls -ld /path/directory
```

for every directory in PATH.

### Vulnerable If

```text
Command uses NO absolute path
+
Writable PATH directory
```

### Memory Trick

```text
Check script
     ↓
Command without full path?
     ↓
Check PATH
     ↓
Writable directory?
     ↓
PATH Abuse
```

### About "." and "/tmp"

```bash
export PATH=.:$PATH
```

or

```bash
export PATH=/tmp:$PATH
```

Used to demonstrate PATH hijacking.

Only gives root if ROOT executes the fake binary.

---

## PATH Abuse - Absolute Path vs No Absolute Path

### What is an Absolute Path?

An absolute path gives the complete location of a file.

Example:

```bash
/usr/bin/tar
```

Linux knows exactly which file to execute.

```text
Root
 │
 ▼
/usr/bin/tar
```

No searching happens.

✅ Safe

---

### What is a Non-Absolute Path?

Example:

```bash
tar
```

Linux does not know where `tar` is located.

It must search the PATH variable.

Check PATH:

```bash
echo $PATH
```

Example output:

```text
/tmp:/usr/bin:/bin
```

Linux searches from left to right:

```text
1. /tmp/tar
2. /usr/bin/tar
3. /bin/tar
```

First match wins.

---

### Why is this dangerous?

Suppose a root script contains:

```bash
tar -cf backup.tar /data
```

and PATH is:

```text
/tmp:/usr/bin:/bin
```

If I can write to `/tmp`, I can create:

```text
/tmp/tar
```

Now when root runs:

```bash
tar -cf backup.tar /data
```

Linux searches:

```text
1. /tmp/tar  ← Found First
2. /usr/bin/tar
3. /bin/tar
```

Linux executes:

```text
/tmp/tar
```

instead of:

```text
/ usr/bin/tar
```

My fake binary runs with root's privileges.

---

### PATH Abuse Checklist

```text
Find Script
    ↓
Command uses:
tar, cp, find, chmod
(No absolute path)
    ↓
Check PATH
    ↓
Any writable PATH directory?
    ↓
YES
    ↓
Create fake command
    ↓
PATH Abuse
```

### Memory Trick

```text
Absolute Path
/usr/bin/tar
     ↓
Safe

No Absolute Path
tar
 ↓
Search PATH
 ↓
First Match Wins
 ↓
Possible PATH Abuse
```

---

## 4. Wildcard Abuse

### Key Concept

Depends on:

```text
*
```

(Asterisk Wildcard)

### Look For

```bash
tar *
zip *
rsync *
```

### Requirements

```text
Wildcard (*)
+
Writable directory
+
Runs as root
```

### Attack Idea

Create files whose names look like command-line options.

When `*` expands, program treats filenames as options.

### Memory Trick

```text
Wildcard Abuse
=
Asterisk (*)
+
Writable directory
```

---

## 5. Cron Job Abuse

### Key Concept

A cron job automatically runs commands/scripts.

### Enumeration

```bash
cat /etc/crontab
ls -la /etc/cron.*
```

Useful:

```bash
pspy64
```

### Look For

```text
Writable script
Writable cron file
Misconfigured cron.d entry
```

### Example

```bash
backup.sh
```

is run by root every few minutes.

If editable:

```bash
echo "my command" >> backup.sh
```

Root executes it when cron runs.

### Memory Trick

```text
Find Cron
     ↓
Runs as Root?
     ↓
Script Writable?
     ↓
Cron Job Abuse
```

---

# Quick Exam Revision

### SUID

```text
Find SUID
↓
Check Owner
↓
Check GTFOBins
```

### Sudo

```text
sudo -l
↓
GTFOBins
↓
Abuse Allowed Command
```

### PATH

```text
No Absolute Path
+
Writable PATH Directory
```

### Wildcard

```text
*
+
Writable Directory
```

### Cron

```text
Writable Script
+
Runs Automatically As Root
```Linux Privilege Escalation - My Notes
1. Docker Basics
Key Concept

Docker allows applications to run inside isolated environments called containers.

A container contains:

Application code
Libraries
Dependencies
Configurations

Think:

Docker Image
      ↓
Creates
      ↓
Docker Container
Docker Architecture

Docker consists of:

Docker Client

Used by users to issue commands.

Examples:

docker ps
docker run ubuntu
docker images
Docker Daemon

Backend service that:

Creates containers
Starts containers
Stops containers
Removes containers
Downloads images
Memory Trick
Docker Client
      ↓
Sends Commands
      ↓
Docker Daemon
      ↓
Performs Actions
2. Docker Images vs Containers
Key Concept
Docker Image

Blueprint/template used to create containers.

Example:

ubuntu:20.04

Image is:

Read Only
Docker Container

Running instance of an image.

Example:

docker run ubuntu

Container is:

Running
Writable
Memory Trick
Image
=
Blueprint

Container
=
Actual Running House
3. Docker Shared Directory Abuse
Key Concept

Docker can share directories between:

Host
↕
Container

This is called:

Volume Mount

Example:

-v /home:/data

Meaning:

Host
/home

Container
/data
Why Is It Dangerous?

If sensitive directories are mounted:

/hostsystem/home/user

We may access:

.ssh
.bash_history
config files

Example:

cat ~/.ssh/id_rsa

May reveal SSH private keys.

Enumeration

Look for unusual directories:

ls /

Possible findings:

/host
/hostsystem
/data
/mnt

Check mounted filesystems:

mount

or

df -h
Vulnerable If
Container Access
+
Sensitive Host Directory Mounted
Memory Trick
Container
     ↓
Find Mounted Folder
     ↓
Access Host Files
     ↓
Possible Escape
4. Docker Socket Abuse
Key Concept

Docker communicates using:

docker.sock

Usually:

/var/run/docker.sock

This file allows communication with the Docker daemon.

Think of it as:

Remote Control
For Docker
Enumeration

Search for socket:

find / -name docker.sock 2>/dev/null

Check permissions:

ls -la /var/run/docker.sock
Why Is It Dangerous?

If we can access:

docker.sock

we may:

Create containers
Stop containers
Mount host filesystem
Control Docker

Which often means:

Control Host
Vulnerable If
User Can Access docker.sock
Memory Trick
docker.sock
      ↓
Talk To Docker
      ↓
Control Containers
      ↓
Possible Root
5. Docker Container Escape via Docker Socket
Key Concept

If docker.sock is accessible, we may create a new container that mounts:

/

(host root filesystem)

inside the container.

Example idea:

Host /
      ↓
Mounted
      ↓
/hostsystem
What Happens?

We gain access to:

/hostsystem/root
/hostsystem/etc
/hostsystem/home

Possible targets:

/root/.ssh/id_rsa
/etc/shadow
Requirements
Access To docker.sock

or

Docker Privileges
Memory Trick
Docker Socket
      ↓
Create Container
      ↓
Mount Host /
      ↓
Access Host Files
6. Docker Group Abuse
Key Concept

Users inside:

docker

group can control Docker.

Check:

id

Example:

uid=1000(user)
groups=1000(user),116(docker)
Why Is It Dangerous?

Docker users can:

Create containers
Mount host filesystem
Run privileged containers

Often resulting in:

Root Access
Enumeration
id

Look for:

docker

group membership.

Vulnerable If
User ∈ docker group
Memory Trick
docker group
      ↓
Control Docker
      ↓
Control Host
      ↓
Root
7. Writable Docker Socket
Key Concept

Normally:

root

or

docker group

can access:

docker.sock

Sometimes permissions are weak.

Example:

ls -la /var/run/docker.sock

Output:

srw-rw-rw-

Anyone can write to it.

Why Is It Dangerous?

Even without being in:

docker group

you can communicate with Docker.

Which means:

Privilege Escalation
Enumeration
ls -la /var/run/docker.sock

Check permissions carefully.

Memory Trick
Not In Docker Group?
          ↓
docker.sock Writable?
          ↓
Still Dangerous
8. Docker Compose
Key Concept

Docker Compose manages multiple containers together.

Uses:

docker-compose.yml

or

compose.yaml
What It Defines
Services
Networks
Volumes
Environment Variables
Why Pentesters Care

Sensitive information may exist:

Passwords
API Keys
Database Credentials

inside:

docker-compose.yml
Enumeration
find / -name "*compose*" 2>/dev/null

Check:

cat docker-compose.yml
Memory Trick
Compose File
      ↓
Container Configuration
      ↓
Possible Secrets
Docker Privilege Escalation Checklist
Inside Container?
        ↓
Check Mounted Directories
        ↓
Check docker.sock
        ↓
Check Docker Group
        ↓
Check Writable docker.sock
        ↓
Check Docker Compose Files
        ↓
Look For Host Access
Quick Exam Revision
Docker Image
Blueprint
Docker Container
Running Instance
Docker Client
Issues Commands
Docker Daemon
Executes Commands
Shared Directory Abuse
Mounted Host Folder
+
Sensitive Files
Docker Socket Abuse
docker.sock
+
Access
Docker Group Abuse
User In Docker Group
Writable Docker Socket
docker.sock Writable
Docker Escape
Mount Host /
+
Access Host Files
One-Line Memory Formula
Docker Enumeration

Container?
   ↓
Mounts?
   ↓
docker.sock?
   ↓
Docker Group?
   ↓
Writable Socket?
   ↓
Host Access?
   ↓
Privilege Escalation 

create a single marakable file with all taht ntoes no working change aombine thme in ascending order  no remove of coanotnetn not conepet hcange 

