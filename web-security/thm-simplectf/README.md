# Simple CTF — CMS Made Simple SQLi to Root

**Category:** Web Exploitation / Credential Cracking / Linux Privilege Escalation
**Full writeup (PDF):** [Simple_CTF.pdf](./Simple_CTF.pdf)

## Overview

A classic beginner-to-intermediate box: an outdated CMS Made Simple install is vulnerable to an authenticated-data-disclosure SQL injection that leaks password hashes, which are cracked and used to pivot to a shell, followed by a well-known GTFOBins privilege escalation.

## Summary of Attack Chain

1. **Recon** — Nmap identified anonymous FTP (21, vsftpd 3.0.3), Apache on port 80, and SSH on the non-standard port 2222.
2. **Directory discovery** — FFUF against port 80 found a `/simple` directory hosting **CMS Made Simple 2.2.8**, a version vulnerable to **CVE-2019-9053** (unauthenticated SQL injection).
3. **Exploitation** — A [public exploit script](https://github.com/Perseus99999/CVE-2019-9053-working-/blob/main/exploit.py) was run against the target with a `--crack` flag pointed at `rockyou.txt`, returning the salt, username, password hash, and the cracked plaintext:
   ```
   Username found: mitch
   Email found: admin@admin.com
   Password cracked: secret
   ```
4. **Initial access** — SSH on port 2222 with the recovered `mitch:secret` credentials returned the user flag.
5. **Enumeration** — The home directory listing revealed a second user, `sunbath`, and `.bash_history` showed `mitch` had recently run commands with `sudo vim`.
6. **Privilege escalation** — Running `sudo vim` inherits root's execution context. From within vim, `:!/bin/sh` spawns a root shell — a textbook [GTFOBins](https://gtfobins.github.io/gtfobins/vim/#sudo) technique — yielding the root flag.

## Skills Demonstrated

- Service/version fingerprinting and matching to public CVEs
- Adapting and running a third-party SQLi exploit script with an integrated cracking workflow
- Offline password cracking against `rockyou.txt`
- Recognising privilege-escalation opportunities from `.bash_history` review
- GTFOBins-style sudo misconfiguration exploitation

## Remediation Notes

- Patch CMS Made Simple to a version beyond 2.2.8, or migrate off unsupported CMS software entirely.
- Enforce strong, non-dictionary passwords — `secret` cracked from a hash almost instantly against `rockyou.txt`.
- Audit `sudoers` entries for editors and other GTFOBins-listed binaries; if `vim` (or similar) must be sudo-permitted, restrict it via `rbash`, command whitelisting, or `sudoedit`.
- Disable anonymous FTP unless explicitly required.

See [Simple_CTF.pdf](./Simple_CTF.pdf) for the full port table, exploit output, and both flags.
