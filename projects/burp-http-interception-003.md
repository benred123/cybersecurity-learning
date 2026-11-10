# Project 003 — HTTP Traffic Inspection with Burp Suite

## Scope

Authorised TryHackMe lab web server at `http://10.129.167.54:8008/`. This project records browser traffic captured using Burp Suite's built-in browser.

## Objective

Understand HTTP requests, headers, and browser-to-server communication before practising request interception.

## Observed activity

- Launched Burp Suite in the TryHackMe AttackBox.
- Opened **Proxy** and used the built-in browser to visit the lab website.
- Located the `GET /` request in **Proxy → HTTP history**.
- Copied the request headers from the **Raw** view.

## Captured HTTP request

```http
GET / HTTP/1.1
Host: 10.129.167.54:8008
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/145.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

## Interpretation

- `GET /`: Requests the homepage using HTTP/1.1.
- `Host`: Identifies the destination host and port.
- `Accept-Language`: Indicates preferred response languages.
- `Upgrade-Insecure-Requests`: Indicates a preference for secure responses where available.
- `User-Agent`: Advertises the browser and operating system.
- `Accept`: Advertises accepted content types.
- `Accept-Encoding`: Advertises supported response compression formats.
- `Connection: keep-alive`: Requests connection reuse.
- No `Cookie` or `Authorization` header appeared in the provided request; this does not establish whether the wider application has authentication.

## Next step

Turn on **Proxy → Intercept**, refresh the authorised lab webpage in Burp's browser, and verify that Burp pauses the outgoing `GET /` request before forwarding. This step has not yet been confirmed.

## Successful HTTP request modification — confirmed

The learner confirmed that Burp Suite's **Proxy → Intercept** paused the outgoing request to the authorised TryHackMe lab website. They replaced the original browser `User-Agent` header with:

```http
User-Agent: Ben-Cybersecurity-Lab
```

They visually confirmed the modified header in Burp before selecting **Forward**, and confirmed the coffee webpage loaded successfully afterwards.

### Outcome and limits

- **Completed:** Capture, edit, and forward a live HTTP request through Burp Suite.
- **Observed:** The website loaded after the header was changed.
- **Not established:** Whether the server logged, stored, or otherwise processed the custom `User-Agent` value beyond receiving the forwarded request.
- **Security lesson:** Client-supplied request headers can be modified; this exercise does not establish a vulnerability.

**Project 003 introductory HTTP interception exercise: complete.**
