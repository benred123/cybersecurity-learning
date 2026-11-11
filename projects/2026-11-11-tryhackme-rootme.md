# TryHackMe RootMe — guided lab completion (11 November 2026)

## Status and scope
**Completed** the TryHackMe RootMe room in an authorised training environment. This was a **heavily guided walkthrough**, not an independent solve. No flag values are published.

## Reconnaissance
- Ran `nmap -sV -sC 10.128.139.96` against the temporary lab target.
- Found TCP 22 (OpenSSH 8.2p1) and TCP 80 (Apache httpd 2.4.41).
- Browsed the website, then used `gobuster dir -u http://10.128.139.96 -w /usr/share/wordlists/dirb/common.txt`.
- Discovered `/panel/` (upload form) and `/uploads/` (directory listing).

## Upload testing and code execution
- Uploaded a harmless `test.txt` and verified that it was publicly readable under `/uploads/`.
- A PHP test file with extension `.php` was rejected; the same harmless code with extension `.phtml` was accepted.
- Visiting the uploaded `.phtml` executed PHP and printed `Hello from PHP`.
- Used PHP's `shell_exec()` to run `whoami`, `id`, `pwd` and `ls -la`. Confirmed the web-server process ran as `www-data` (UID/GID 33) and the upload path was `/var/www/html/uploads`.
- Found `/var/www/user.txt`, read it and submitted the user flag.

## Privilege escalation
- Enumerated SUID files with `find / -type f -perm -4000 2>/dev/null`.
- Noticed `/usr/bin/python2.7` with root ownership and SUID permissions (`-rwsr-xr-x`).
- Verified the interpreter's effective UID was `0` with Python `os.geteuid()`.
- Listed `/root` using the SUID Python interpreter and identified `root.txt`.
- Edited an existing PHP file using `nano` to use Python's `open(...).read()` to access the protected file. Completed the room's questions and final flag submission.

## What I learned
- A directory discovered by enumeration can reveal functionality not linked from the homepage.
- Accepting an uploaded file and **executing** it are separate security properties; both were tested.
- File-extension filtering alone is inadequate when an alternative executable extension is allowed.
- Linux process identity matters: PHP commands ran as `www-data`, while the SUID Python interpreter ran with effective UID 0.
- `rws` in owner permissions indicates SUID, and a root-owned general-purpose interpreter with SUID can be dangerous.
- Used `nano` to edit a script and learned how PHP string quoting, shell quoting and Python code interact.

## Honest reflection and next steps
This room involved extensive step-by-step assistance, including exact commands and identification of the likely privilege-escalation route. The main gap to address is constructing commands independently rather than copying nested PHP/Python/shell expressions. Next practice should focus on Linux file editing, quoting, command execution, permissions, and gradual hint-based problem solving.

> Training only: all activity was performed on the intentionally vulnerable TryHackMe lab machine.
