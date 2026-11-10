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

## Follow-up: Scanning the designated remote TryHackMe target

**Status:** Completed, 10 November 2026 at 15:34 UTC

The designated room target was `10.129.167.54`, different from the AttackBox (`10.129.64.178`). The following command was run within the authorised training environment:

```bash
nmap -sT -sV -p 22,80,443 10.129.167.54
```

### Actual output (relevant lines)

```text
Host is up (0.00067s latency).
PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
80/tcp  closed http
443/tcp closed https
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
Nmap done: 1 IP address (1 host up) scanned in 0.29 seconds
```

### Findings and interpretation

- **22/tcp:** Open, SSH service identified as OpenSSH 9.6p1, Ubuntu package 3ubuntu13.5.
- **80/tcp and 443/tcp:** Closed during this scan.
- Only the three explicitly specified TCP ports were checked; other ports and UDP services were not assessed.
- The AttackBox reported a different OpenSSH Ubuntu package build (`3ubuntu13.20`). A version difference alone is not proof of a vulnerability.

**Next step:** Continue the room's authorised exercises and expand the scan only within the permitted target scope.

## Expanded TCP scan — default 1,000 ports

**Observed:** 10 November 2026 at 15:35 UTC, against the same authorised TryHackMe target.

```bash
nmap -sT 10.129.167.54
```

```text
Host is up (0.00025s latency).
Not shown: 994 closed tcp ports (conn-refused)
PORT     STATE SERVICE
7/tcp    open  echo
9/tcp    open  discard
13/tcp   open  daytime
17/tcp   open  qotd
22/tcp   open  ssh
8008/tcp open  http
Nmap done: 1 IP address (1 host up) scanned in 0.09 seconds
```

### Interpretation

The scan identified **six open ports** among Nmap's default **1,000 commonly scanned TCP ports**, and 994 closed ports. Ports 7 (echo), 9 (discard), 13 (daytime), 17 (quote of the day), 22 (SSH), and 8008 (commonly HTTP) were reported open. These service labels in a scan without `-sV` are based on conventional port mappings and do not verify the running application. The finding on port 8008 illustrates that a potential web service may use a port other than 80 or 443; confirmation requires further probing.

**Next:** Use `nmap -sV -p 7,9,13,17,22,8008 10.129.167.54` to investigate service identities within the authorised lab scope.
