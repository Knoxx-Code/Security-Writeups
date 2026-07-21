# BeetleBug CTF - Android Mobile Penetration Testing

**Category:** Mobile Security (Android)
**Full writeup (PDF):** [Winston_BeetleBug_CTF.pdf](./Winston_BeetleBug_CTF.pdf)

## Overview

BeetleBug is an intentionally vulnerable Android application (`app.beetlebug`) built to teach the OWASP Mobile Top 10 through capture-the-flag style challenges. This writeup covers static analysis (Jadx), dynamic analysis (`adb`), and the exported-component attacks that make up most real-world Android app assessments.

## Summary of Challenges Completed

| Category | Vulnerability | Technique |
|---|---|---|
| **Embedded Secrets** | Hardcoded data in `strings.xml` | Static analysis in Jadx located an obfuscated flag and a hardcoded promo code (`beetle_bug_shop_promo_code`) 
directly in decompiled source |
| **Data Storage** | Shared Preferences, External Storage, SQLite | Used `adb shell` to read `/data/data/app.beetlebug/shared_prefs/`, `/storage/emulated/0/Documents/`, 
and pulled a SQLite `user.db` for offline inspection — all stored credentials in plaintext |
| **Web View** | Weak URL validation & JS injection | Jadx revealed `setAllowUniversalAccessFromFileURLs(true)` and `setJavaScriptEnabled(true)` on a WebView; 
exploited via a crafted `adb` intent and confirmed with an XSS `<script>alert()</script>` payload |
| **Insecure Databases** | SQL Injection, Misconfigured Firebase | Classic `admin' OR 1=1--` returned all DB rows including credentials; a hardcoded Firebase 
Realtime Database URL in `strings.xml` was queryable unauthenticated via `curl`, leaking user tokens |
| **Android Components** | Unprotected Activity, Vulnerable Service, Vulnerable Content Provider | Enumerated `AndroidManifest.xml` for `android:exported="true"` 
components and launched them directly via `adb shell am start`/`startservice`, and queried an exported `ContentProvider` via `adb shell content query` |
| **Sensitive Info Disclosure** | Insecure logging, Clipboard exposure | `logcat` revealed full credit card numbers logged in plaintext; a "Copy & Pay" button 
copied full card details (including CVV) to the system clipboard |
| **Biometric Bypass** | Weak deep link handling | An activity meant to gate account access behind fingerprint auth accepted an unvalidated deep link intent via
`adb`, bypassing biometric checks entirely |
| **Binary Patching** | Client-side authorization check | Decompiled the APK with `apktool`, edited a disabled button's `android:enabled` attribute to `true` directly 
in the extracted layout XML, demonstrating how client-side gating can be bypassed by anyone with the APK |

## Skills Demonstrated

- Static analysis and decompilation with Jadx (string/resource search, manifest review, WebView/Activity/Service/Provider auditing)
- Dynamic analysis via `adb shell` (file system access, intent crafting, content provider querying)
- Identification of the OWASP Mobile Top 10 categories in a hands-on setting: insecure storage, weak cryptography/hardcoding, improper platform usage, insufficient authentication
- APK patching workflow with `apktool` (decompile → edit smali/resources → rebuild → sign)

## Remediation Themes

- Never hardcode secrets, API keys, or database URLs in application resources — use a backend-mediated secrets store.
- Set `exported="false"` on any Activity, Service, or ContentProvider that isn't intentionally part of the app's public inter-app contract.
- Disable `setAllowUniversalAccessFromFileURLs` and treat WebView JavaScript execution as untrusted by default.
- Never persist sensitive data (credentials, card numbers) in Shared Preferences, external storage, or logs without encryption.
- Client-side checks (button enablement, deep-link gating) are not security boundaries — they must be enforced server-side.

See [Winston_BeetleBug_CTF.pdf](./Winston_BeetleBug_CTF.pdf) for full screenshots of each captured flag and the Jadx/apktool output.
