# Abuse report — hosting provider for `216.126.225.243`

**Submission target:** Current abuse contact for the IP allocation. Re-verify ownership before filing. Passive research during the investigation associated the IP with AS14956 / RouterHosting LLC / Cloudzy-related hosting; a previously identified contact was `abuse-reports@cloudzy.com`, but this must be checked at reporting time.

## Subject

```text
C2 infrastructure for developer-targeting malware — 216.126.225.243 ports 8085/8086/8087
```

## Body

```text
I am reporting C2/exfiltration infrastructure recovered from a confirmed malicious developer-recruitment archive.

REPORTED INFRASTRUCTURE
IP: 216.126.225.243
Ports / URLs recovered from payload:
- http://216.126.225.243:8086/upload
- http://216.126.225.243:8085/upload
- http://216.126.225.243:8087

MALICIOUS ARTIFACT
Filename observed: daam-repo.zip
ZIP SHA-256: b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d
Hidden payload path: backend/src/config/.svn/.DS_Store
Hidden payload SHA-256: fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5
Hidden payload size: 2,901,715 bytes

TECHNICAL BASIS
Static analysis of the hidden JavaScript payload recovered the three URLs above. The endpoints were not contacted during analysis. The payload is imported during normal backend startup by backend/src/data/mockData.js, which deliberately bridges ES modules to CommonJS require() and loads ../config/.svn/.DS_Store.

The same archive also includes a malicious .git/hooks/post-checkout hook that downloads and executes attacker-controlled code from iploglab.store.

CAMPAIGN CORRELATION
Independent public malware research has documented the same IP and ports in developer-targeting recruitment malware, associating:
- 8086 with automatic sensitive-file exfiltration,
- 8085 with on-demand/requested-file exfiltration,
- 8087 with Socket.IO command-and-control.

Those port semantics are external correlation; the direct evidence in this report is that the DAAM payload statically contains these C2 URLs.

REQUESTED ACTION
Please investigate, preserve relevant logs, and disable abusive service on 216.126.225.243 if confirmed. This infrastructure is tied to a fake-recruitment malware campaign targeting software developers.

Reported by: [your name / handle]
Date: [today's date]
```
