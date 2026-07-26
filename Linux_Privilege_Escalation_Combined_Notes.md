# Linux Privilege Escalation - Combined Notes

This file combines the notes provided in the conversation into a single Markdown document.

\---

# 1\. SUID Abuse

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

## Memory Trick

```text
SUID = Run as Owner
GTFOBins = How to abuse it
```

\---

# 2\. SGID Abuse

## Key Concept

* SGID means a file runs with the permissions of its group.
* User temporarily becomes a member of that group.

## Enumeration

```bash
find / -perm -2000 -type f 2>/dev/null
```

## Memory Trick

```text
SUID = User
SGID = Group
```

\---

# 3\. Sudo Rights Abuse

## Enumeration

```bash
sudo -l
```

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

\---

# 4\. NOPASSWD Abuse

## Key Concept

Command can be executed as root without entering a password.

\---

# 5\. GTFOBins

## Key Concept

GTFOBins is a database of Linux binaries that can be abused.

\---

# 6\. PATH Abuse

## Enumeration

```bash
echo $PATH
```

\---

# 7\. Absolute Path vs Relative Path

## Safe

```bash
/usr/bin/tar
```

## Dangerous

```bash
tar
```

\---

# 8\. Shared Library Hijacking

## Enumeration

```bash
ldd binary
```

\---

# 9\. Wildcard Abuse

Depends on:

```text
\*
```

\---

# 10\. Cron Job Abuse

## Enumeration

```bash
cat /etc/crontab
ls -la /etc/cron.\*
```

\---

# 11\. Custom SUID Binary Analysis

```bash
file binary
strings binary
ldd binary
```

\---

# 12\. Tcpdump Sudo Abuse

Example:

```bash
(root) NOPASSWD: /usr/sbin/tcpdump
```

\---

# 13\. Docker Basics

Docker Client → Docker Daemon → Containers

\---

# 14\. Docker Images vs Containers

Image = Blueprint

Container = Running Instance

\---

# 15\. Docker Shared Directory Abuse

Host ↔ Container volume mounts.

\---

# 16\. Docker Socket Abuse

Typical socket:

```text
/var/run/docker.sock
```

\---

# 17\. Docker Container Escape via Docker Socket

Mount host filesystem and access host files.

\---

# 18\. Docker Group Abuse

Check:

```bash
id
```

Look for:

```text
docker
```

\---

# 19\. Writable Docker Socket

Check:

```bash
ls -la /var/run/docker.sock
```

\---

# 20\. Docker Compose

Look for:

```bash
find / -name "\*compose\*" 2>/dev/null
```

\---

# 21\. Privileged Groups - LXD

Check:

```bash
id
```

Look for:

```text
lxd
```

\---

# 22\. Privileged Groups - Docker

Docker group can control Docker and potentially access host files.

\---

# 23\. Privileged Groups - Disk

Look for:

```text
disk
```

Interesting devices:

```bash
ls -l /dev/sd\*
```

\---

# 24\. Privileged Groups - ADM

Look for:

```text
adm
```

Read logs under:

```text
/var/log
```

\---

# 25\. Linux Capabilities 

capabilities like permission we give to files to proctes aigant unauth user like security 

Enumeration:

```bash
getcap -r / 2>/dev/null
```

\---

# 26\. Capability - cap\_dac\_override

Bypasses file permission checks.

Example:

```text
/usr/bin/vim.basic cap\_dac\_override=eip
```

\---

# 27\. Capability - cap\_setuid

Allows changing UID, potentially to UID 0.

\---

# 28\. Capability - cap\_setgid

Allows changing GID, potentially to GID 0.

\---

# 29\. Capability - cap\_sys\_admin

Often called "The New Root".

\---

# 30\. Capability Values

```text
e = Effective
i = Inheritable
p = Permitted
```

\---

# 31\. /etc/passwd Abuse via cap\_dac\_override

Modify `/etc/passwd` to potentially gain root access.

\---

# Linux PrivEsc Enumeration Checklist

```bash
id
whoami
hostname
sudo -l
find / -perm -4000 -type f 2>/dev/null
find / -perm -2000 -type f 2>/dev/null
getcap -r / 2>/dev/null
cat /etc/crontab
echo $PATH
find / -writable 2>/dev/null
```

