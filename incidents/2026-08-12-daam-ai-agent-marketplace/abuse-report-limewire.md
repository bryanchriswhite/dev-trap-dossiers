# Abuse report — LimeWire

**Submission target:** LimeWire abuse / support channel. Attach the original ZIP if the form allows safe malware submission, or provide hashes and the delivery URL.

## Subject

```text
Malicious developer-recruitment ZIP distributed via LimeWire — DAAM AI-agent marketplace trap
```

## Body

```text
I am reporting a LimeWire download link used to distribute a confirmed malicious ZIP archive in a developer-targeted fake-recruitment campaign.

REPORTED URL
https://limewire.com/d/vlTzy#wGXKqgjp2B

ARTIFACT
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d

VERDICT
Confirmed malicious. Static analysis found three independent execution paths in the archive:

1. A bundled .git/hooks/post-checkout hook that downloads and executes attacker-controlled code from iploglab.store when the victim switches/checks out branches.
2. README instructions directing the victim to execute Bitbucket-hosted installer scripts under daam602/ai-agent-marketplace.
3. Normal backend startup imports a 2,901,715-byte obfuscated JavaScript payload disguised as backend/src/config/.svn/.DS_Store.

KEY HASHES
- .git/hooks/post-checkout SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
- Hidden JavaScript payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5

C2 / DROPPER INDICATORS
- iploglab.store
- http://216.126.225.243:8086/upload
- http://216.126.225.243:8085/upload
- http://216.126.225.243:8087

REQUESTED ACTION
Please remove or disable the reported LimeWire download and preserve account/upload metadata for abuse investigation. The ZIP is being used to compromise software developers under a fake interview/pre-interview code-review pretext.

Reported by: [your name / handle]
Date: [today's date]
```
