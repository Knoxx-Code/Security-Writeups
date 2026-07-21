# Security-Writeups
This repo is an index of technical writeups from CTF rooms, vulnerable-by-design labs, and authorised real-world engagements. Each writeup documents the methodology, the exact commands/payloads used, and the remediation I'd recommend if this were a live production system.

Full PDF reports (with screenshots and terminal captures) are linked from each folder — Obsidian's image-embed syntax doesn't render on GitHub, so the source-of-truth for evidence lives in the PDF while the README carries the narrative and technical detail.

## Real-World Engagements

| Writeup | Target Type | Highlights |
|---|---|---|
| [Web & API Pentest - Event Management & Ticketing Platform](./pentest-report-web-api) | Private client, black-box | 9 findings (3 Critical, 4 High, 2 Medium) — mass assignment, privilege escalation, payment bypass, IDOR |

## Web Pentesting

| Writeup | Target | Vulnerability Class |
|---|---|---|
| [BookStore - LFI to Root](./web-pentesting/thm-bookstore) | Flask/Werkzeug REST API | Local File Inclusion → Werkzeug Debug PIN → RCE → Binary Reversing (Ghidra) |
| [CMSpit - Cockpit CMS](./web-pentesting/thm-cmspit) | Cockpit CMS 0.11.1 | CVE-2020-35846 (User Enum/Auth Bypass) → File Upload RCE → CVE-2021-22204 (exiftool privesc) |
| [Simple CTF - CMS Made Simple](./web-pentesting/thm-simple-ctf) | CMS Made Simple 2.2.8 | CVE-2019-9053 (SQLi) → Credential Cracking → GTFOBins privesc (vim) |
| [VulnWeb - Multi-Subdomain Assessment](./web-pentesting/vulnweb) | 5 subdomains, mixed stacks | SQLi, Stored/Reflected/DOM XSS, IDOR, Broken Auth, Outdated Platforms |

## Cloud Security

| Writeup | Target | Vulnerability Class |
|---|---|---|
| [Flaws.Cloud - AWS Misconfigurations](./cloud-security/aws-flaws-cloud) | AWS (S3, EC2, IAM) | Public S3 buckets, exposed .git credentials, EC2 snapshot abuse, SSRF via metadata service |

## Mobile Security

| Writeup | Target | Vulnerability Class |
|---|---|---|
| [BeetleBug CTF - Android Pentesting](./mobile-security/beetlebug-ctf-android) | Android APK | Hardcoded secrets, insecure storage, WebView misconfig, SQLi, exported components, biometric bypass |

## Tools I Use

Burp Suite · OWASP ZAP · PostMan · Nmap · Nuclei · SQLMap · Dirsearch/Feroxbuster · FFUF · Subfinder · Jadx · Ghidra · Metasploit · AWS CLI · ADB
