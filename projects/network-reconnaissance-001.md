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

## Service detection on all six discovered ports

**Observed:** 10 November 2026 at 15:36 UTC on the authorised TryHackMe target.

```bash
nmap -sV -p 7,9,13,17,22,8008 10.129.167.54
```

| Port | Nmap service | Version / evidence |
| --- | --- | --- |
| 7/tcp | echo | No version identified |
| 9/tcp | discard? | Identification uncertain |
| 13/tcp | daytime? | Returned UTC date/time strings |
| 17/tcp | qotd? | Returned quotation text |
| 22/tcp | ssh | OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 |
| 8008/tcp | http | lighttpd 1.4.74 |

**Scan duration:** 157.48 seconds. Nmap reported two unrecognised service fingerprints despite receiving data, corresponding to ports 13 and 17. The question marks indicate uncertain identification. Returned text strongly supports Daytime and Quote of the Day functionality, but Nmap did not confidently match those services to known signatures. The HTTP service on port 8008 demonstrates why scanning only 80 and 443 can miss a web server.

**Next investigation:** In the authorised lab, open `http://10.129.167.54:8008/` in the AttackBox browser and inspect the page, or request its HTTP headers with `curl -I http://10.129.167.54:8008/`. Record actual responses before making further claims. A service version alone does not prove a vulnerability.

## HTTP header inspection — Port 8008

**Observed:** 10 November 2026 at 15:56:57 GMT, within the authorised TryHackMe lab.

```bash
curl -I http://10.129.167.54:8008/
```

```http
HTTP/1.1 200 OK
Content-Type: text/html
ETag: "3115179486"
Last-Modified: Wed, 09 Oct 2024 07:50:58 GMT
Content-Length: 3039
Accept-Ranges: bytes
Date: Tue, 10 Nov 2026 15:56:57 GMT
Server: lighttpd/1.4.74
```

### Interpretation

- **200 OK:** The server successfully handled the HTTP HEAD request made by `curl -I`.
- **Content-Type: text/html:** The resource is identified as HTML.
- **Content-Length: 3039:** The server reports a 3,039-byte representation; `curl -I` did not download the page body.
- **ETag / Last-Modified:** Metadata used for cache validation; the server reports a last-modified timestamp of 9 October 2024.
- **Accept-Ranges: bytes:** The server advertises byte-range request support.
- **Date:** The server reports its response timestamp.
- **Server: lighttpd/1.4.74:** Consistent with Nmap's earlier version detection. This is an advertised version, not proof of a vulnerability.

**Next:** Request the page body or inspect the page in the AttackBox browser, and record the actual visible content before making conclusions.

## Web content inspection and TryHackMe flag discovery

**Observed:** 10 November 2026, authorised room target at `10.129.167.54:8008`.

```bash
curl http://10.129.167.54:8008/
```

The HTTP response body contained an HTML document titled **Types and Styles of Coffee**. Near the beginning of the `<body>`, the page displayed a heading `Flag` and a `<code>` element containing:

```text
THM{SECRET_PAGE_38B9P6}
```

### What this demonstrates

1. Nmap identified an HTTP service on nonstandard port **8008** even though ports 80 and 443 were closed.
2. `curl -I` inspected HTTP response headers, while `curl` without `-I` retrieved the HTML body.
3. Reading the returned HTML revealed the training-room flag embedded directly in page content.
4. This was **content discovery**, not exploitation or an authentication bypass. The flag was present in the response available to the client.

**Next:** Submit the flag to the corresponding TryHackMe question and confirm whether it is accepted. The flag has been found but room acceptance has not yet been verified.
