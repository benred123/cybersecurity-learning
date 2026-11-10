# Project 002 — Web Enumeration with Gobuster

## Scope

Authorised TryHackMe training target: `http://10.129.167.54:8008/`. All observations below are from the user's own lab terminal output.

## Objective

Learn basic directory and file enumeration using Gobuster, interpret HTTP response codes, and distinguish discovered resources from untested possibilities.

## Tool

Gobuster v3.6, directory enumeration mode.

## Command

```bash
gobuster dir -u http://10.129.167.54:8008/ -w /usr/share/wordlists/dirb/common.txt -t 10
```

- `dir`: directory/file enumeration mode.
- `-u`: target URL.
- `-w`: wordlist path.
- `-t 10`: ten concurrent threads.
- Method: GET; negative status code configured as 404.

## Observed results

```text
/index.html           (Status: 200) [Size: 3039]
Progress: 4614 / 4615 (99.98%)
Finished
```

The scan completed using the DIRB `common.txt` wordlist (4,615 listed entries) and reported **one path**, `/index.html`, returning HTTP **200** with size **3,039 bytes**. This matches the content length of the previously inspected coffee page, suggesting it is the same resource, although the file's contents have not yet been compared directly in this project.

## Lessons learned

- A successful scan may report very few paths.
- A `200` response means the request succeeded; it does not itself indicate a vulnerability.
- A single wordlist scan cannot prove that no other resources exist.
- Directory enumeration results should be interpreted and validated, not assumed to reveal every path.

## Next verification

Request `/index.html` with `curl` or a browser to verify its content, then compare it with the homepage. Do not claim further paths exist without evidence.
