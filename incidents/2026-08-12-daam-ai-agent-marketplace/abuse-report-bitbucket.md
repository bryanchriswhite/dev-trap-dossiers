# Abuse report — Atlassian / Bitbucket

**Submission target:** Atlassian abuse / Bitbucket report-abuse flow.

## Subject

```text
Bitbucket installer infrastructure embedded in confirmed malicious developer-recruitment archive — daam602/ai-agent-marketplace
```

## Body

```text
I am reporting Bitbucket-hosted installer paths embedded in a confirmed malicious developer-recruitment archive.

REPORTED BITBUCKET ACCOUNT / REPOSITORY
Workspace/user: daam602
Repository: daam602/ai-agent-marketplace

REPORTED URLS
- https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install.sh
- https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install-windows.bat

IMPORTANT EVIDENCE BOUNDARY
I did not fetch or execute the live Bitbucket installer scripts. This report is based on the fact that these URLs are embedded in the README of a ZIP archive that is independently confirmed malicious through static analysis.

MALICIOUS ARTIFACT CONTEXT
Filename observed: daam-repo.zip
Delivery URL: https://limewire.com/d/vlTzy#wGXKqgjp2B
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d

CONFIRMED MALICIOUS BEHAVIOR IN THE ARCHIVE
1. The archive ships .git/hooks/post-checkout, which downloads and executes attacker-controlled code from iploglab.store.
2. The backend imports a hidden 2.9 MB obfuscated JavaScript payload disguised as backend/src/config/.svn/.DS_Store during normal startup.
3. The hidden payload contains C2 endpoints at 216.126.225.243:8085, :8086, and :8087.

KEY HASHES
- .git/hooks/post-checkout SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
- Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5

REQUESTED ACTION
Please investigate the daam602 account/repository and disable any malicious installer content. The URLs are part of a fake-recruitment flow designed to compromise developers asked to run pre-interview materials.

Reported by: [your name / handle]
Date: [today's date]
```
