# CMSpit - Cockpit CMS RCE Chain (CVE-2020-35846 → CVE-2021-22204)

**Category:** Web Exploitation / CVE Chaining / Linux Privilege Escalation
**Full writeup (PDF):** [CMSpit.pdf](./CMSpit.pdf)

## Overview

CMSpit targets a Cockpit CMS 0.11.1 instance. The engagement chains a known username-enumeration/auth-bypass CVE into an authenticated file upload RCE, then escalates privileges through a second, unrelated CVE affecting a sudo-permitted binary.

## Summary of Attack Chain

1. **Recon & version fingerprinting** — Nmap found ports 80/22. Aggressive `dirsearch`/`feroxbuster` enumeration surfaced dozens of framework paths, and `package.json` leaked the exact Cockpit version: **0.11.1**.
2. **User enumeration & auth bypass — CVE-2020-35846** — Intercepting a failed login in DevTools revealed a `POST /auth/check` endpoint that leaks whether a username exists. This version of Cockpit is vulnerable to CVE-2020-35846; a Metasploit module (`multi/http/cockpit_cms_rce`) automated user enumeration, returning `admin`, `darkStar7471`, `skidy`, and `ekoparty`.
3. **Account takeover** — Using the same Metasploit module against the `skidy` account recovered their email and reset their password via the `/auth/resetpassword` token flow, giving authenticated access.
4. **File upload RCE** — With admin-level access to the Cockpit finder, a malicious PHP reverse shell was uploaded directly through the CMS file manager and triggered by browsing to it, returning a `www-data` shell.
5. **Lateral discovery** — A `.dbshell` history file in `/home/stux` revealed raw MongoDB commands including a plaintext password (`p4ssw0rdhack3d!123`) for the `stux` account, allowing SSH access and the user flag.
6. **Privilege escalation — CVE-2021-22204** — `sudo -l` showed `stux` could run `exiftool` as root. Exiftool ≤ 12.23 is vulnerable to CVE-2021-22204, a command injection triggered via a crafted DjVu file (built with `djvumake`). Running the [public PoC](https://github.com/UNICORDev/exploit-CVE-2021-22204) generated a malicious JPEG; running `sudo exiftool` against it while listening on a port returned a root shell.

## Skills Demonstrated

- CVE research and matching to fingerprinted software versions
- Metasploit module usage for both enumeration and exploitation
- Credential harvesting from application/database shell history
- Public exploit adaptation (fixing Python string-formatting issues in a third-party PoC)
- Chaining two independent CVEs across the application layer and the OS layer

## Remediation Notes

- Upgrade Cockpit CMS well past 0.11.1 and monitor CVE feeds for CMS dependencies.
- Never persist credentials in database shell history files (`.dbshell`, `.mongorc.js`, etc.).
- Restrict `sudo` grants to the minimum required binaries, and keep utilities like `exiftool` patched — a sudo allowance for a vulnerable binary is equivalent to a root shell.
- Disable unauthenticated file upload/execution paths in CMS file managers, or at minimum block execution of uploaded scripts from the upload directory.

See [CMSpit.pdf](./CMSpit.pdf) for full request/response captures, Metasploit output, and both flags.
