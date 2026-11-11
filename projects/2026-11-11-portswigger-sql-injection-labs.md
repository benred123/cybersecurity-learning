# PortSwigger Web Security Academy — SQL Injection Labs

**Date:** 11 November 2026  
**Platform:** PortSwigger Web Security Academy  
**Environment:** Deliberately vulnerable training labs, accessed using a TryHackMe AttackBox and Burp Suite Community Edition  
**Status:** 3 labs completed

> All testing was conducted against authorised educational lab instances. These notes intentionally exclude session cookies and unique lab hostnames.

## Lab 1 — SQL injection vulnerability in WHERE clause allowing retrieval of hidden data

**Objective:** Display unreleased products by changing a product-category filter.

### Method

1. Opened a product category and identified the user-controlled `category` query parameter in `/filter?category=Gifts`.
2. Added a single quote (`Gifts'`), which produced an error and suggested unsafe SQL query construction.
3. Tested `Gifts'--`; the page loaded, suggesting the trailing SQL condition could be commented out.
4. Submitted `Gifts' OR 1=1--`, causing the query's filtering condition to evaluate as true and exposing hidden products.
5. Confirmed PortSwigger marked the lab solved.

### Key learning

SQL injection occurs when user-controlled data is incorporated into an SQL statement as executable query syntax. `OR 1=1` introduces an always-true condition, while `--` comments out trailing SQL in this lab. Parameterised queries are the primary defence.

## Lab 2 — SQL injection UNION attack, determining the number of columns returned by the query

**Objective:** Identify the column count of the product query and return an additional row of NULL values.

### Setup and troubleshooting

- Started TryHackMe's browser-based AttackBox and opened Burp Suite Community Edition.
- The AttackBox could reach `example.com` and `portswigger-labs.net`, but a request to `portswigger.net:443` failed.
- Confirmed the actual lab hostname was reachable and subsequently loaded the shopping website in Burp's browser.
- Used Proxy HTTP history to capture a product-category request and sent it to Repeater.

### Method and observed results

| Test | Payload (category value) | HTTP response |
| --- | --- | --- |
| 1 column | `Gifts' UNION SELECT NULL--` | 500 |
| 2 columns | `Gifts' UNION SELECT NULL,NULL--` | 500 |
| 3 columns | `Gifts' UNION SELECT NULL,NULL,NULL--` | 200 |

The third test indicated the underlying query returned **three columns**, but the lab was not yet marked solved.

Submitted the lab-completion payload:

```text
'+UNION+SELECT+NULL,NULL,NULL--
```

PortSwigger then confirmed the lab was **solved**.

### Key learning

A UNION combines the results of SELECT queries when their column counts match and corresponding data types are compatible. `NULL` is useful when probing column counts because it is compatible with many SQL column types. HTTP 200 alone does not guarantee that a lab's completion condition has been met.

## Lab 3 — SQL injection UNION attack, finding a column containing text

**Objective:** Find a column accepting text and display the lab's assigned string.

### Testing in Burp Repeater

| Test | Result |
| --- | --- |
| `' UNION SELECT NULL,NULL,NULL--` | 200 OK |
| `' UNION SELECT 'hello',NULL,NULL--` | 500 error |
| `' UNION SELECT NULL,'hello',NULL--` | 200 OK; `hello` found in raw HTTP response |

I replaced `hello` in column two with the lab's exact assigned string and PortSwigger marked the lab solved.

### Lessons

The UNION queries must have matching column counts and compatible data types. Test each column with a string while leaving the others NULL. Inspect raw HTTP responses, not just rendered pages. HTTP 200 alone does not prove a lab is solved.

## Skills practised

- HTTP query parameters and request/response inspection
- Burp Suite Proxy and Repeater
- SQL comment syntax, boolean conditions and UNION queries
- Inferring database query structure from response differences
- Connectivity troubleshooting with `curl -I`

## Next planned exercise

Next: another authorised PortSwigger web security lab.

## Reflection

The most useful progression was moving from editing URLs in the browser to modifying and resending captured HTTP requests in Burp Repeater. The failed one- and two-column UNION tests were informative, and the three-column test demonstrated how response comparison helps investigate query structure.
