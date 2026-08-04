# VulnWeb — Multi-Subdomain Web & API Pentest

**Category:** Web Application Penetration Test (Practice Environment)
**Full report (PDF):** [VulnWeb_Pentesting_Report.pdf](./VulnWeb_Pentesting_Report.pdf)

> **Note:** `vulnweb.com` is Acunetix's intentionally vulnerable test environment. This writeup is included to demonstrate breadth of methodology and reporting quality across multiple tech stacks in a single engagement.

## Overview

This report documents a structured black-box assessment across **5 active subdomains** spanning ASP, PHP, ASP.NET, REST, and HTML5 stacks, following the OWASP Testing Guide. The goal was to demonstrate that a single engagement can surface — and consistently document — a wide range of vulnerability classes across heterogeneous technologies, not just one CVE on one target.

## Methodology & Tools

Reconnaissance → Enumeration → Exploitation → Impact Analysis → Reporting, using Wappalyzer, Nuclei, SQLMap, Dirsearch, Subfinder, Httpx, and Burp Suite.

## Key Findings by Subdomain

| Subdomain | Findings |
|---|---|
| `testasp.vulnweb.com` | SQLi auth bypass (`admin' OR 1=1--`), full DB dump via SQLMap, reflected XSS, IDOR on sequential forum IDs |
| `testhtml5.vulnweb.com` | Missing authentication controls (empty password accepted), DOM-based XSS via URL fragment |
| `testphp.vulnweb.com` | Directory listing exposing `create.sql` schema, outdated PHP 5.6.40, full DB dump via SQLMap, stored XSS in guestbook (`javascript:` URI in anchor), plaintext credential transmission |
| `testaspnet.vulnweb.com` | SQLi auth bypass, reflected XSS, outdated ASP.NET 2.0.50727, full DB dump with cracked admin password hash |
| `www.vulnweb.com` | Missing security headers (CSP, HSTS, X-Frame-Options, etc.) via Nuclei scan |

**Highest-priority finding:** Full database compromise via SQL injection was independently reproducible on three separate subdomains (`testasp`, `testphp`, `testaspnet`), each exposing the `users` table with plaintext or weakly-hashed credentials — indicating a systemic failure to parameterise database queries across the whole tech stack, not an isolated bug.

## Risk Prioritisation

The full report includes a consolidated risk matrix (Likelihood × Impact → Priority) across all 12 findings, with SQLi-driven database compromise and the HTML5 auth bypass rated **Immediate**, and XSS/IDOR/outdated-platform findings rated **High**.

## Skills Demonstrated

- Multi-target scoping and subdomain enumeration (Subfinder, Httpx)
- Automated + manual SQL injection testing and full database extraction (SQLMap)
- XSS across three variants: reflected, stored, and DOM-based
- IDOR identification via sequential ID manipulation
- Technology fingerprinting to flag outdated, exploitable platform versions
- Structured, client-ready report writing with risk prioritisation

See [VulnWeb_Pentesting_Report.pdf](./VulnWeb_Pentesting_Report.pdf) for full proof-of-concept requests/responses, SQLMap dump output, and the complete risk matrix.
