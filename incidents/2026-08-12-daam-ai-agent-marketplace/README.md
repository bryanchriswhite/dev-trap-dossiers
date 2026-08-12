# DAAM AI-agent marketplace — incident write-up & technical analysis

| | |
|---|---|
| Subject | DAAM / `ai-agent-marketplace` developer-recruitment archive delivered as `daam-repo.zip` |
| Encounter date | 2026-08-12 |
| Verdict | Confirmed malicious. Developer-targeted social-engineering trap with multiple independent execution paths: bundled Git hook, README remote installer, and normal backend startup importing a hidden obfuscated JavaScript payload. |
| Analysis boundary | Static-only. No Git checkout/switch, package installation, JavaScript execution, README commands, or attacker-controlled URLs were executed or contacted during the analysis summarized here. |

---

## Navigating this case

This file is the master analysis. Sibling files are derivative artifacts for specific audiences:

- [`briefing-for-developers.md`](./briefing-for-developers.md) — short warning for candidates/recruitment targets.
- [`iocs.csv`](./iocs.csv) and [`iocs.json`](./iocs.json) — machine-readable indicators.
- [`detection-rules.md`](./detection-rules.md) — YARA, Sigma concepts, and grep rules.
- Abuse/reporting templates:
  - [`abuse-report-limewire.md`](./abuse-report-limewire.md)
  - [`abuse-report-linkedin.md`](./abuse-report-linkedin.md)
  - [`abuse-report-calendly.md`](./abuse-report-calendly.md)
  - [`abuse-report-bitbucket.md`](./abuse-report-bitbucket.md)
  - [`abuse-report-hosting-c2.md`](./abuse-report-hosting-c2.md)
  - [`abuse-report-iploglab.md`](./abuse-report-iploglab.md)
  - [`abuse-report-npm.md`](./abuse-report-npm.md)
  - [`abuse-report-authorities.md`](./abuse-report-authorities.md)

No full malicious repository source tree, payload body, or executable artifact is committed here.

---

## 0. TL;DR

A recruiter supplied a ZIP archive, `daam-repo.zip`, through LimeWire as pre-interview material for a purported decentralized AI-agent marketplace. Static analysis confirmed the archive is malicious.

The archive contains at least three independent execution paths:

1. A bundled `.git/hooks/post-checkout` hook that downloads and executes attacker-controlled payloads from `iploglab.store` when branch switching occurs.
2. README instructions that tell the candidate to execute remote installer scripts hosted under Bitbucket account/workspace `daam602`.
3. A 2.9 MB obfuscated JavaScript payload disguised as `backend/src/config/.svn/.DS_Store`, imported from normal backend startup via `backend/src/data/mockData.js`.

The hidden payload references Node filesystem, process-execution, network, and crypto modules/APIs, contains strings associated with developer-secret theft, and statically recovered C2 endpoints at `216.126.225.243:8085`, `:8086`, and `:8087`. The same IP/ports overlap with public reporting on Contagious Interview / OtterCookie-related developer-targeting malware.

---

## 1. Recruitment and social-engineering context

The contact originated on LinkedIn. The purported role was a flexible remote engineering position for a "decentralized AI Agent marketplace platform" combining AI agents and blockchain. The pitch included high compensation, part-time or full-time flexibility, and pre-call review of documentation, video, and a "codebase snapshot."

The recruiter supplied:

- Calendly: `https://calendly.com/wallace55/30mins`
- LimeWire delivery URL: `https://limewire.com/d/vlTzy#wGXKqgjp2B`

The contact identified the purported hiring manager as "Wallace" and told the candidate to review the materials and "refer to the README." That instruction matters because the README is itself part of the execution chain.

The AI/Web3 framing, attractive compensation, pre-interview code review, and scheduling pressure are consistent with known developer-targeting recruitment lures. The malicious verdict, however, rests on the supplied artifact's contents, not on the pitch alone.

---

## 2. Primary artifact hashes

- ZIP SHA-256: `b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d`
- `.git/hooks/post-checkout` SHA-256: `9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9`
- Hidden JavaScript payload SHA-256: `fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5`
- Hidden JavaScript payload size: 2,901,715 bytes
- Video SHA-256: `665d013769320c8fe90426b707864f9128fd419107ae1f69525cfece7217044b`

---

## 3. Execution-path matrix

| Developer action | Fires malware? | Why |
|---|---|---|
| Merely possess/download ZIP | Not by the confirmed paths alone | No automatic execution is established from possession of bytes. |
| Use Git to switch/checkout branches in the bundled repo | Yes / attempted RCE | The shipped `.git/hooks/post-checkout` performs OS-specific remote download-and-execute from `iploglab.store`. |
| Follow README installer commands | Yes / attempted RCE | README directs execution of Bitbucket-hosted install scripts. Those live endpoints were not fetched during analysis, so contents are an evidence gap, but the path is unsafe. |
| Start the backend normally | Yes | `mockData.js` imports the hidden obfuscated JS payload at `backend/src/config/.svn/.DS_Store`. |
| Run npm install alone | Not enough to rule safe | Root setup reportedly uses npm `--ignore-scripts`, but that does not prevent runtime import of the hidden payload later. |
| Open in an IDE | Unknown; unsafe | IDEs may invoke Git/workspace automation or inspect/run project tooling. Do not open on a credentialed host. |

---

## 4. Repository and branch structure

Branch refs were recovered directly from loose Git objects without invoking Git:

- `main` → `4408960d10fc8c47668a3f6c6be34232ae08e65a`
- `video` → `0479e2ed93cd049a7cc7094484b43661b6fb03c2`
- `doc` → `1d209cd5d406a551c33fb18fcf79717d2a9f6c89`
- `codebase` → `d29fc332a996157964b9ebb475116d91da79fffe`

The staged social-engineering flow is:

`main → video → documentation → codebase`

The landing branch tells the candidate to switch branches, which creates a natural trigger for the malicious `post-checkout` hook. The complete `.git` directory is distributed inside the ZIP, unlike a normal source archive, allowing hooks to be shipped directly to the victim.

---

## 5. Independent execution paths

### Path A — Git post-checkout hook

The archive includes `.git/hooks/post-checkout`. The hook branches by OS and downloads attacker-controlled code:

- macOS: `https://iploglab.store/api/terminal/bootstrap?os=mac&flag=7` piped to `bash`
- Linux: `https://iploglab.store/api/terminal/bootstrap?os=linux&flag=7` fetched and piped to `bash`
- Windows/MSYS/Cygwin: `https://iploglab.store/api/terminal/windows?flag=7` fetched by PowerShell and passed to `Invoke-Expression`

Output is suppressed. A developer following ordinary branch-switching instructions can trigger this before ever running application code.

### Path B — README remote installer

The `codebase` branch README directs the candidate to execute remote installer scripts:

- `https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install.sh`
- `https://bitbucket.org/daam602/ai-agent-marketplace/raw/main/scripts/install-windows.bat`

Those endpoints were not contacted during this investigation. Report them as installer infrastructure embedded in a confirmed malicious artifact, not as independently analyzed script contents.

### Path C — ordinary backend startup

The codebase contains a 2,901,715-byte obfuscated JavaScript program disguised as:

`backend/src/config/.svn/.DS_Store`

`backend/src/data/mockData.js` deliberately enables CommonJS `require()` inside an ES-module project and loads:

`../config/.svn/.DS_Store`

Multiple controllers/services import the mock-data module. A candidate can avoid the Git hook and README installer yet still run malware simply by starting the backend.

---

## 6. Intentionality and concealment signals

The following signals should be preserved together:

- Complete `.git` directory distributed in the ZIP.
- Landing README encourages branch switching.
- Executable `post-checkout` hook suppresses output.
- Backend `.gitignore` explicitly keeps `.DS_Store` via `!.DS_Store`.
- Executable payload hidden as `.svn/.DS_Store`.
- ES-module project uses deliberate `createRequire()` + `require()` bridge to load the hidden file.
- Root setup uses npm with `--ignore-scripts`, which can appear cautious while not preventing runtime payload execution.
- Multiple independent execution mechanisms target developers with different levels of caution.

---

## 7. Payload static findings

Static inspection of the hidden JavaScript payload confirmed references to:

- Node `os`
- Node `fs`
- Node `path`
- Node `child_process`
- `execSync`
- `spawn`
- `axios`
- `crypto`

Obfuscation characteristics:

- 17,456-entry encoded string table
- Modified Base64 alphabet: `abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=`
- RC4-style decoding layer
- Wrapper functions and dynamic property access
- Table rotation before lookup
- Statically reconstructed rotation: 363 entries

Decoded/static strings included `.ssh`, `.aws`, `seed`, `seed-`, `token`, `Token`, `login`, `Login`, `HOME`, `USER`, `exec`, `execSync`, and `spawn`. These are strong warning signs, but exact credential-stealing semantics should not be inferred from individual strings alone.

Constants observed:

- `u_k = 507`
- `t = 5`

WSL-aware logic referenced:

- `process.env.WSL_DISTRO_NAME`
- `/proc/version`
- Windows-user discovery
- `/mnt/c/Users`

---

## 8. Recovered C2 and campaign correlation

Recovered directly from the hidden payload:

- `http://216.126.225.243:8086/upload`
- `http://216.126.225.243:8085/upload`
- `http://216.126.225.243:8087`

These endpoints were not contacted during the investigation.

Independent public malware research has documented the same IP and ports in developer-targeting recruitment malware, reporting:

- `8086` → automatic sensitive-file exfiltration
- `8085` → on-demand/requested-file exfiltration
- `8087` → Socket.IO command-and-control / remote control

Those port semantics are external campaign correlation unless independently reconstructed from this exact sample. Preferred attribution language:

> Strong technical overlap with, and likely a variant/deployment of infrastructure associated with, Contagious Interview / OtterCookie.

Do not upgrade this into a claim that the visible LinkedIn identity, Git metadata names, or email addresses identify the human operator.

---

## 9. Associated infrastructure and accounts

### Confirmed malicious

- `iploglab.store` — directly referenced by the malicious post-checkout hook.
- `216.126.225.243:8085` — recovered from hidden payload.
- `216.126.225.243:8086` — recovered from hidden payload.
- `216.126.225.243:8087` — recovered from hidden payload.

### Directly associated with delivery/recruitment

- LinkedIn recruiter/contact profile used for initial outreach.
- LimeWire `limewire.com/d/vlTzy#wGXKqgjp2B` — direct artifact delivery.
- Calendly `calendly.com/wallace55/30mins` — scheduling infrastructure supplied in the same flow.
- Bitbucket workspace/user `daam602`.
- Bitbucket repo `daam602/ai-agent-marketplace`.

### Associated / suspicious; report with qualification

- npm package `wallet-modal-223@1.0.0`.
- npm publisher/account `daam217`.

The npm tarball was not downloaded or analyzed. Report it as associated with a confirmed malicious project and request provider investigation; do not state that the package itself is confirmed malware from current evidence.

### Weak provenance only

- `psr511 <andersonalexanderigu505@hotmail.com>`
- `Elizab Campbell <elizab.eth@Elizabs-Mac.local>`
- `unknown <root@SERVER.(none)>`
- `williamherr <williamherr8@gmail.com>`
- `Cursor <cursoragent@cursor.com>`

These may be useful for cross-sample correlation but are trivially forgeable and should not be treated as reliable attribution.

---

## 10. Suspicious npm dependency

The lockfile pins:

- package: `wallet-modal-223@1.0.0`
- tarball: `https://registry.npmjs.org/wallet-modal-223/-/wallet-modal-223-1.0.0.tgz`
- integrity: `sha512-hoK0UlDkIEG7eYj9rMqfMXWIxqd1N60QbXaKlbNqnnYOSYvEIvEWomboqffjxNh3SIQ3qf6oPX6+c0jVlCG0Ew==`

Declared dependencies include `axios`, `socket.io-client`, and `toastr`.

The package itself was not downloaded during this analysis. Treat it as associated/suspicious, not independently confirmed malicious.

---

## 11. Victim-exposure guidance

### Downloaded only

If a victim only downloaded the ZIP and did not invoke Git operations, shell commands, application code, IDE automation, or binaries, the confirmed execution mechanisms described here were not triggered merely by possessing the bytes.

### Switched/checked out branches

Treat the machine as potentially compromised. The bundled hook attempted remote code execution.

### Executed README installer

Treat the machine as potentially compromised. Exact installer contents were not fetched during the investigation.

### Started backend

Treat the machine as potentially compromised because normal backend imports deliberately load the hidden JavaScript payload.

Priority response actions: isolate host, preserve evidence, rotate credentials/tokens/SSH keys, review GitHub/npm/cloud/wallet exposure, audit persistence, and strongly consider reimaging.

---

## 12. Reporting priority

High-confidence abuse reports:

1. LimeWire — direct distribution of confirmed malicious archive.
2. LinkedIn — recruitment/social-engineering account.
3. Calendly — scheduling infrastructure tied to the campaign.
4. Atlassian / Bitbucket — remote installer infrastructure.
5. Hosting provider for `216.126.225.243` — confirmed C2/exfiltration infrastructure. Re-verify current ownership before filing.
6. Registrar/host for `iploglab.store` — confirmed remote-execution infrastructure. Re-verify DNS/registrar/host before filing.

Qualified report/request for investigation:

7. npm — `wallet-modal-223@1.0.0` and publisher `daam217`, associated with confirmed malicious project; package content not independently analyzed.

Preserve but do not report solely from current evidence:

8. Gmail/Hotmail addresses found only in Git metadata.
9. Local Git author identities.

---

## 13. Evidence-preservation checklist

- Original LinkedIn conversation/screenshots/export.
- Recruiter profile URL/screenshots.
- Exact opportunity-pitch text and timestamps.
- Calendly URL.
- LimeWire URL.
- Original ZIP byte-for-byte and SHA-256.
- Hook contents/hash.
- Hidden payload hash/size.
- Branch refs/commit IDs.
- Branch-navigation README.
- Bitbucket installer instructions.
- Source import path into hidden payload.
- npm package/version/integrity metadata.
- C2 endpoints.
- Passive provider/ASN/DNS information at reporting time.
- Provider abuse-report case IDs.
- Later operator messages/replacement infrastructure, if any.

Do not contact attacker infrastructure merely to manufacture additional evidence.
