# Linux Investigation #001 — Users, Directories and Permissions

**Date:** 10 November 2026  
**Environment:** Authorised browser-based Linux training VM  
**Status:** Completed (introductory exercise)

## Objective
Practise basic Linux navigation, identify the current user and groups, inspect directory contents, create a practice file and interpret its permissions.

## Commands and observed results

| Command | Observation |
| --- | --- |
| `pwd` | `/root` |
| `id` | `uid=0(root) gid=0(root) groups=0(root),987(docker)` |
| `ls -la` | Listed hidden files, owners, groups, sizes and permissions in `/root` |
| `cd /tmp` | Changed to the temporary directory |
| `mkdir ben-linux-lab` | Created a practice directory |
| `cd ben-linux-lab` | Entered the practice directory |
| `touch notes.txt` | Created an empty file |
| `ls -l` | Displayed the new file and its permissions |

### Evidence from the practice directory

```text
root@training-vm:/tmp/ben-linux-lab# ls -l
total 0
-rw-r--r-- 1 root root 0 Nov 10 15:11 notes.txt
```

The host identifier has been replaced with a generic training-VM label.

## What the results mean

- `/root` is the root user's home directory.
- UID 0 identifies the root account, and GID 0 identifies the primary root group.
- The user also belongs to group `docker` (GID 987). On some configurations Docker access can confer extensive privileges; group membership alone does not prove a specific vulnerability.
- `ls -la` displayed hidden configuration directories, including ones associated with Burp Suite, OWASP ZAP and Metasploit. Their presence does not prove the applications are running.
- `-rw-r--r--` corresponds to standard permission mode `644`: the owner can read/write, while group and others can read.
- `root root` indicates the file's owner and group; `0` indicates its size in bytes.

## Security relevance

Understanding identity, group membership and file permissions is foundational for system hardening, access control and investigating potential privilege escalation. Commands here were executed only within an authorised training VM.

## Reflection and next steps

This exercise confirmed how to identify the active user, navigate directories and inspect file permissions. Next: practise `chmod`, `stat`, file content commands and the principle of least privilege using a disposable practice file.

*This write-up records an introductory exercise, not a penetration test or a discovered vulnerability.*

## Follow-up exercise — Changing file permissions

**Status:** Completed, 10 November 2026

Within the same authorised training VM and the practice directory `/tmp/ben-linux-lab`, I changed permissions on the empty file `notes.txt` and checked the result:

```text
$ chmod 600 notes.txt
$ ls -l notes.txt
-rw------- 1 root root 0 Nov 10 15:11 notes.txt
$ chmod 644 notes.txt
$ stat -c '%a %U %G %n' notes.txt
644 root root notes.txt
```

### Explanation

- `600` grants the **owner** read (4) and write (2), and gives group and others no permissions.
- `644` grants the owner read/write and grants group and others read only.
- `ls -l` showed the symbolic permissions after the first change.
- `stat` confirmed the restored numeric mode, owner, group and filename.

**Security lesson:** Permissions implement access controls, but the exercise ran as UID 0 (`root`), which can normally bypass standard discretionary file permissions. Testing actual access restrictions requires a non-root account in a controlled lab.
