# Abuse report — Calendly

**Submission target:** https://help.calendly.com/hc/en-us/requests/new — use Trust & Safety / abuse if available.

## Subject

```text
Calendly page used in malicious developer-recruitment funnel — DAAM AI-agent marketplace trap
```

## Body

```text
I am reporting a Calendly scheduling page used as part of a confirmed malware-delivery recruitment funnel targeting software developers.

REPORTED CALENDLY PAGE
https://calendly.com/wallace55/30mins

PERSONA / EVENT DETAILS
Persona name shown: [fill from Calendly page]
Event title shown: [fill from Calendly page]

SUMMARY
A LinkedIn recruiter supplied this Calendly link together with a LimeWire ZIP archive for pre-interview review of a purported decentralized AI-agent marketplace. Static analysis of the ZIP confirms it is malicious.

The Calendly page is not itself hosting the malware, but it is part of the social-engineering chain: it lends legitimacy to the fake interview process and creates deadline pressure for the candidate to inspect or run the malicious code before a call.

MALICIOUS ARTIFACT
Delivery URL: https://limewire.com/d/vlTzy#wGXKqgjp2B
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d

TECHNICAL SUMMARY
The archive contains:
1. A bundled .git/hooks/post-checkout hook that downloads/executes code from iploglab.store.
2. README instructions to run Bitbucket-hosted installer scripts under daam602/ai-agent-marketplace.
3. A hidden obfuscated JavaScript payload at backend/src/config/.svn/.DS_Store imported during normal backend startup.

KEY INDICATORS
- Hook SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
- Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5
- Dropper: iploglab.store
- Payload C2: 216.126.225.243:8085, :8086, :8087

REQUESTED ACTION
Please investigate and disable the reported Calendly page/account if confirmed to be supporting a malware-delivery recruitment campaign. Please preserve account registration and booking metadata for abuse investigation.

Reported by: [your name / handle]
Date: [today's date]
```
