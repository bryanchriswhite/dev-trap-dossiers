# Abuse report — LinkedIn

**Submission target:** LinkedIn profile/reporting flow or LinkedIn Trust & Safety. Attach screenshots/export of the recruiter conversation and profile.

## Subject

```text
LinkedIn account used to deliver malicious developer-recruitment archive — DAAM AI-agent marketplace
```

## Body

```text
I am reporting a LinkedIn account used as the social-engineering front for a confirmed developer-targeting malware campaign.

REPORTED PROFILE
[LinkedIn profile URL]

SUMMARY
The reported account contacted me with a purported opportunity for a decentralized AI-agent marketplace / AI+Web3 engineering role. The account supplied a Calendly scheduling URL and a LimeWire ZIP archive for pre-interview review. The ZIP has been statically analyzed and confirmed malicious.

RECRUITMENT INFRASTRUCTURE SUPPLIED IN THE CONVERSATION
- Calendly: https://calendly.com/wallace55/30mins
- LimeWire: https://limewire.com/d/vlTzy#wGXKqgjp2B

MALICIOUS ARTIFACT
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d

TECHNICAL SUMMARY
The archive contains multiple independent execution paths:
1. .git/hooks/post-checkout downloads and executes attacker-controlled code from iploglab.store.
2. README instructions direct the victim to execute Bitbucket-hosted installer scripts under daam602/ai-agent-marketplace.
3. Ordinary backend startup imports a 2.9 MB obfuscated JavaScript payload disguised as backend/src/config/.svn/.DS_Store.

KEY INDICATORS
- Hook SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
- Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5
- Dropper: iploglab.store
- Payload C2: 216.126.225.243:8085, :8086, :8087

ATTRIBUTION NOTE
This report is about the LinkedIn account's use in a malicious recruitment flow. I am not claiming the visible profile identity is the real-world operator; the account may be fake, compromised, or otherwise misrepresented.

REQUESTED ACTION
Please investigate and disable the LinkedIn account if confirmed to be using LinkedIn to deliver malware to developers. Please preserve conversation/profile metadata for abuse investigation.

Reported by: [your name / handle]
Date: [today's date]
```
