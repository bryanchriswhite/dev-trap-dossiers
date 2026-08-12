# Abuse report — npm

**Submission target:** npm package abuse/security reporting flow.

## Subject

```text
Request for npm security review — wallet-modal-223@1.0.0 associated with confirmed malicious developer-recruitment archive
```

## Body

```text
I am requesting npm security review of a package associated with a confirmed malicious developer-recruitment archive.

REPORTED PACKAGE
wallet-modal-223@1.0.0
Publisher/account observed: daam217
Tarball URL from lockfile: https://registry.npmjs.org/wallet-modal-223/-/wallet-modal-223-1.0.0.tgz
Integrity from lockfile: sha512-hoK0UlDkIEG7eYj9rMqfMXWIxqd1N60QbXaKlbNqnnYOSYvEIvEWomboqffjxNh3SIQ3qf6oPX6+c0jVlCG0Ew==
Declared dependencies observed: axios, socket.io-client, toastr

IMPORTANT EVIDENCE BOUNDARY
I did not download or analyze the npm tarball. Please treat this as a qualified report/request for investigation: the package is associated with a confirmed malicious project, but I am not asserting from current evidence that the package itself independently contains malware.

MALICIOUS ARTIFACT CONTEXT
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d
Delivery URL: https://limewire.com/d/vlTzy#wGXKqgjp2B

CONFIRMED MALICIOUS BEHAVIOR IN THE ARCHIVE
1. Bundled .git/hooks/post-checkout downloads and executes attacker-controlled code from iploglab.store.
2. README directs victims to run remote Bitbucket installer scripts under daam602/ai-agent-marketplace.
3. Backend startup imports a hidden 2.9 MB obfuscated JavaScript payload disguised as backend/src/config/.svn/.DS_Store.

KEY HASHES / IOCs
- .git/hooks/post-checkout SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
- Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5
- Dropper: iploglab.store
- C2: 216.126.225.243:8085, :8086, :8087

REQUESTED ACTION
Please review wallet-modal-223@1.0.0 and publisher daam217 for abuse. If the package or account is confirmed malicious or part of this campaign, please remove/suspend accordingly and preserve metadata for investigation.

Reported by: [your name / handle]
Date: [today's date]
```
