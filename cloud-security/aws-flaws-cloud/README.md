# Flaws.Cloud — AWS Misconfiguration Walkthrough (Levels 1–6)

**Category:** Cloud Security (AWS)
**Full writeup (PDF):** [Flaws_Cloud.pdf](./Flaws_Cloud.pdf)

## Overview

[flaws.cloud](http://flaws.cloud) is a deliberately vulnerable AWS environment designed to teach common cloud misconfigurations. This writeup covers Levels 1 through 6, escalating from a public S3 bucket to full credential theft via a leaked `.git` directory to SSRF against the EC2 instance metadata service.

## Summary by Level

| Level | Vulnerability | Technique |
|---|---|---|
| **1** | S3 bucket permissions set to "Everyone" | `dig` + reverse DNS to find the bucket host, then unauthenticated `aws s3 ls` |
| **2** | S3 bucket permissions set to "Any Authenticated AWS User" | Same enumeration, but requires *any* valid AWS account (`--profile`) rather than anonymous access |
| **3** | Leaked AWS credentials in a public `.git` directory | Synced the bucket's `.git` folder, walked `HEAD` → `refs/heads/master` → `git show`, recovering an access key and secret accidentally committed and later "deleted" (but still present in history) |
| **4** | Publicly accessible EC2 snapshot | Used the Level 3 credentials to enumerate EC2 snapshots owned by the compromised account, created a volume from the snapshot, attached it to an attacker-controlled instance, and read a `setupNginx.sh` script containing HTTP basic-auth credentials |
| **5** | SSRF against the EC2 instance metadata service | The instance ran an HTTP proxy; requesting `169.254.169.254/latest/meta-data/iam/security-credentials/flaws` through it returned temporary IAM credentials for an assumed role |
| **6** | Privilege pivoting via assumed-role credentials | Used the Level 5 credentials to list and access a nested prefix inside the Level 6 bucket, completing the chain |

## Skills Demonstrated

- DNS reconnaissance (`dig`, `nslookup`) to identify cloud-hosted assets
- AWS CLI usage across anonymous, authenticated, and assumed-role contexts
- Git internals (`HEAD`, refs, `git show`) for recovering "deleted" secrets from commit history
- EC2 snapshot/volume forensics for extracting data from a compromised account's infrastructure
- SSRF exploitation against the AWS IMDS (a real-world technique behind several major cloud breaches)

## Remediation Notes

- Set S3 bucket ACLs and policies to deny public/any-authenticated-user access by default; use bucket policies and Block Public Access settings.
- Treat any credential that ever touched version control as compromised — deleting it from a later commit does not remove it from history. Rotate immediately and use tools like `git-secrets` or pre-commit hooks to prevent recurrence.
- Restrict IAM permissions attached to EC2 instance roles to the minimum required, and prefer **IMDSv2** (which requires a session token and mitigates basic SSRF-to-metadata attacks) over IMDSv1.
- Snapshots and volumes should never be shared publicly or left attached to instances with weaker access controls than the source.

See [Flaws_Cloud.pdf](./Flaws_Cloud.pdf) for full command output, screenshots of each level's completion page, and the AWS CLI transcripts.
