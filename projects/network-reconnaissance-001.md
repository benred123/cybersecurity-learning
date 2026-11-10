# Network Reconnaissance #001 — First Nmap Scans

**Date:** 10 November 2026  
**Environment:** Authorised TryHackMe AttackBox  
**Status:** Completed introductory local-host scan; separate target investigation pending

## Objective
Learn how to check an Nmap installation, run a TCP connect scan, and identify service versions.

## Scope and important observation
The terminal prompt identified the AttackBox as `ip-10-129-64-178`. The scanned address was also `10.129.64.178`, so this exercise scanned the **AttackBox itself**, not a separate TryHackMe target.

## Commands
```bash
nmap --version
nmap -sT -p 22,80,443 10.129.64.178
nmap -sV -p 22,80,443 10.129.64.178
```

## Actual findings
- Nmap version: **7.94SVN**
- Host reported up
- **22/tcp open:** SSH; version scan identified `OpenSSH 9.6p1 Ubuntu 3ubuntu13.20`
- **80/tcp closed:** HTTP's conventional port was not accepting connections
- **443/tcp closed:** HTTPS's conventional port was not accepting connections
- TCP connect scan duration: **0.13 seconds**
- Service version scan duration: **2.06 seconds**

## Interpretation
`-sT` uses the operating system's TCP connect mechanism. `-sV` adds probes to identify running services and versions. An open port is not itself evidence of a vulnerability; closed ports do not rule out services on other ports.

## Lessons learned
- Verify the target IP before scanning: an AttackBox and a deployed room target are different machines.
- Read the `PORT`, `STATE`, `SERVICE`, and `VERSION` columns separately.
- Record observations without claiming vulnerabilities that have not been verified.
- Scan only systems owned or explicitly authorised for testing.

## Next step
Deploy the room's designated target, confirm its IP in the TryHackMe interface, and repeat a scoped scan against that authorised target.
