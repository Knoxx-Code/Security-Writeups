# BookStore — LFI to Root via Werkzeug Debug Console

**Category:** Web Exploitation / Binary Reverse Engineering
**Full writeup (PDF):** [BookStore.pdf](./BookStore.pdf)

## Overview

BookStore is a Flask-based REST API ("Foxy REST API v2.0") serving science-fiction book data. The box chains a local file inclusion bug into remote code execution via Werkzeug's interactive debugger, then requires reversing a SUID binary to escalate to root.

## Summary of Attack Chain

1. **Recon** — Nmap revealed SSH (22), Apache (80), and a Werkzeug dev server (5000). `dirsearch` and endpoint fuzzing on port 5000 surfaced `/api`, `/console`, and `/robots.txt` (which disallowed `/api` — a strong signal the endpoint mattered).
2. **Source hint** — An HTML comment in `login.html` revealed the developer had left the Werkzeug debugger PIN inside a user's `.bash_history` file, hinting at an LFI vector.
3. **LFI discovery** — The documented `v2` API parameters (`id`, `author`, `published`) didn't accept path traversal. Fuzzing an *undocumented* `v1` version of the same API (developers forgot to deprecate it) surfaced a `show` parameter vulnerable to LFI:
   ```
   GET /api/v1/resources/books?show=/../../../../../../home/sid/.bash_history HTTP/1.1
   ```
4. **Debug PIN extraction** — The bash history leaked the Werkzeug debug PIN (`123-321-135`), unlocking the `/console` route — a full Python execution context in the app's process.
5. **RCE via debug console** — A Python reverse-shell one-liner through the console gave a shell as `sid`.
6. **Privilege escalation** — `sid`'s home directory contained a SUID binary (`try-harder`, owned by root). Decompiling it in Ghidra showed a simple XOR check: `input ^ 0x1116 ^ 0x5db3 == 0x5dcd21f4`. Solving for `input` (`0x5dcd21f4 ^ 0x1116 ^ 0x5db3 = 1573743953`) and supplying it as the "magic number" spawned a root shell via `system("/bin/bash -p")`.

## Skills Demonstrated

- API fuzzing and version-discovery (undocumented `v1` endpoint)
- Local File Inclusion exploitation for credential/secret disclosure
- Abuse of Werkzeug's debugger as an RCE primitive
- Static binary analysis and reverse engineering with Ghidra
- Manual XOR cipher reversal for a CTF-style privesc gate

## Remediation Notes

- Never expose the Werkzeug interactive debugger in a production-like environment — it is designed for local development only.
- Deprecated API versions must be fully decommissioned, not just undocumented — leaving `v1` reachable reintroduced a patched vulnerability.
- Secrets (debug PINs, credentials) should never be discoverable via shell history or logs accessible to the application user.
- SUID binaries should be avoided wherever possible; where required, they must not rely on a static, reversible check as an authorization gate.

See [BookStore.pdf](./BookStore.pdf) for full terminal output, Ghidra decompilation screenshots, and both flags.
