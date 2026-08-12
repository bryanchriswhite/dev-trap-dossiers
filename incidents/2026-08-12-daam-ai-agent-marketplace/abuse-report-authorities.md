# Report package — appropriate authorities / cybercrime reporting

This is a conservative summary suitable for law-enforcement or cybercrime portals such as IC3 or a national CERT. Adapt to the local reporting channel. Do not include unsupported human attribution claims.

## Subject

```text
Developer-targeted fake-recruitment malware: DAAM AI-agent marketplace archive
```

## Body

```text
I am reporting a developer-targeted malware incident delivered through a fake recruitment process.

SUMMARY
A recruiter contacted me on LinkedIn about a purported decentralized AI-agent marketplace / AI+Web3 engineering role. The recruiter supplied a Calendly scheduling link and a LimeWire ZIP archive as pre-interview material. Static analysis confirms the ZIP archive is malicious and designed to compromise software developers who inspect or run the supplied repository.

RECRUITMENT / DELIVERY DETAILS
- Initial contact platform: LinkedIn
- Calendly URL supplied: https://calendly.com/wallace55/30mins
- LimeWire delivery URL supplied: https://limewire.com/d/vlTzy#wGXKqgjp2B
- Archive filename observed: daam-repo.zip
- ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d

TECHNICAL FINDINGS
The archive contains at least three independent execution paths:

1. Bundled malicious Git hook
   - Path: .git/hooks/post-checkout
   - SHA-256: 9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9
   - Behavior: OS-specific remote download-and-execute from iploglab.store when Git branch checkout/switch occurs.

2. README remote installer instructions
   - https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install.sh
   - https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install-windows.bat
   - The live installer scripts were not contacted or analyzed.

3. Backend runtime payload
   - Hidden payload path: backend/src/config/.svn/.DS_Store
   - Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5
   - Size: 2,901,715 bytes
   - Trigger: backend/src/data/mockData.js enables CommonJS require() inside an ES-module project and imports ../config/.svn/.DS_Store during normal backend startup.

C2 / INFRASTRUCTURE INDICATORS
- Dropper/bootstrap domain: iploglab.store
- Payload C2 URLs recovered statically:
  - http://216.126.225.243:8086/upload
  - http://216.126.225.243:8085/upload
  - http://216.126.225.243:8087
- Bitbucket workspace/user: daam602
- npm package associated with project: wallet-modal-223@1.0.0
- npm publisher/account observed: daam217

EVIDENCE BOUNDARY
The analysis was static-only. No Git commands, branch switches, JavaScript, package installation, README commands, or attacker-controlled URLs were executed or contacted during analysis. The npm package wallet-modal-223@1.0.0 was not downloaded or independently analyzed; it should be treated as associated/suspicious rather than confirmed malicious from current evidence.

CAMPAIGN CORRELATION
The C2 IP and ports overlap with public reporting on developer-targeted recruitment malware associated with the Contagious Interview / OtterCookie cluster. This is technical campaign/infrastructure correlation, not proof of the human operator behind the LinkedIn account.

IMPACT / RISK
A developer following ordinary instructions could trigger remote code execution by switching branches, following README installer commands, or starting the backend. The payload references filesystem, process execution, HTTP communication, and strings associated with developer secrets such as SSH keys, cloud credentials, tokens, and wallet/seed material.

REQUEST
Please record this incident and coordinate with relevant platforms/providers as appropriate. I can provide preserved screenshots, timestamps, the original ZIP, and hashes upon request through a safe evidence-submission channel.

Reported by: [your name / handle]
Date: [today's date]
```
