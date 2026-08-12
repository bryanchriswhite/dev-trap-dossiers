# Abuse report — registrar/host for `iploglab.store`

**Submission target:** Current registrar and/or hosting provider for `iploglab.store`. Re-verify DNS, registrar, and hosting immediately before filing.

## Subject

```text
iploglab.store used as download-and-execute bootstrap in malicious Git hook
```

## Body

```text
I am reporting iploglab.store as remote-execution infrastructure used by a malicious Git hook in a developer-targeted fake-recruitment archive.

REPORTED DOMAIN
iploglab.store

REPORTED URLS OBSERVED IN THE MALICIOUS HOOK
- https://iploglab.store/api/terminal/bootstrap?os=mac&flag=7
- https://iploglab.store/api/terminal/bootstrap?os=linux&flag=7
- https://iploglab.store/api/terminal/windows?flag=7

MALICIOUS ARTIFACT
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d
Hook path: .git/hooks/post-checkout
Hook SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9

TECHNICAL SUMMARY
The ZIP archive ships a complete .git directory including an executable post-checkout hook. The hook branches by OS and downloads attacker-controlled code from iploglab.store. On Linux/macOS it is piped to a shell; on Windows it is fetched by PowerShell and passed to Invoke-Expression. Output is suppressed.

This means a candidate following README instructions to switch branches can trigger remote code execution before running package installation or application code.

ADDITIONAL CONFIRMED MALWARE IN THE SAME ARCHIVE
The archive also contains a hidden 2.9 MB obfuscated JavaScript payload at backend/src/config/.svn/.DS_Store, imported during normal backend startup. Payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5. Static analysis recovered C2 endpoints at 216.126.225.243:8085, :8086, and :8087.

REQUESTED ACTION
Please investigate and suspend/disable iploglab.store if confirmed abusive. Preserve DNS/hosting/account logs where possible; this domain is directly used for download-and-execute payload delivery in a developer-targeting malware campaign.

Reported by: [your name / handle]
Date: [today's date]
```
