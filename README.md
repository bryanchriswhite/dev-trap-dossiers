# Attack-campaign documentation

A growing record of developer-targeted malware campaigns analyzed in detail, paired with **copy-paste-ready artifacts for the different people who need to act on each one** — would-be victims, abuse desks, detection engineers, security researchers.

**Currently tracking:** an active developer-targeting operation matching the publicly-documented **"Contagious Interview" TTP cluster** (fake-recruiter → clone repo → `npm install`/`npm start` → stealer-loader). **≥16 known repository instances** across at least four GitHub organizations and several individual accounts, spanning **at least three distinct loader-code generations** (two AjunaVerse-family + one realfraction-family). Two Vercel-hosted C2 servers plus one non-Vercel C2 host, operator activity observed at least through mid-May 2026.

If you arrived here because of one of the situations below, jump straight to the file that's for you. You don't need to read anything else first.

---

## Are you here because…

### 🚨 A recruiter just sent you a "Web3 / DeFi / metaverse / dApp / crypto-gaming MVP" repo and asked you to clone and run it ahead of an interview

**Stop.** It is very likely a trap.

The campaign covers **at least ~16 known repositories** across multiple GitHub organizations and accounts and **at least three distinct loader generations** — see the [Known campaign repositories](#known-campaign-repositories) table below. If you were pointed at any of them — *or at any repo that fits the same shape* (single-author commit history, "Web3 MVP" framing, fresh GitHub org with one repo, README that pitches a multi-person team or generic-and-team-less platform) — read the developer briefing for the matching generation before doing anything else:

→ **[AjunaVerse-family briefing](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md)** — covers the `*.vercel.app` Vercel-C2 / `verify(setApiKey)` + `new Function("require", response.data)` generations (current and earlier).
→ **[realfraction-family briefing](./incidents/2026-05-18-realfraction/briefing-for-developers.md)** — covers the `ipregionchecker.com` C2 / `x-secret-header: secret` + `eval(data)` generation, where the loader lives in `server/utils/regionChecker.js` and is triggered as a side-effect `require()`.

Both are 5-minute reads. Forwardable to a colleague.

### 📮 You're filing a takedown report against any repo in this campaign

The abuse reports below are **copy-paste templates** that any campaign-affected reporter can use. Fill in your case-specific bits — the repo you were pointed at, the commit you analyzed, your name/handle — from the relevant incident's case file before submitting. The campaign-wide indicators (operator-controlled organizations and user accounts, C2 hostnames, etc.) are already in the templates because they're the same across the cluster.

- **GitHub Trust & Safety** (https://github.com/contact/report-abuse). Pick the template matching the generation of the repo you encountered:
  - AjunaVerse-family repos → **[`abuse-report-github.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md)** (six filings: one repo + three orgs + two users).
  - realfraction-family repos → **[`abuse-report-github.md`](./incidents/2026-05-18-realfraction/abuse-report-github.md)** (two filings: one repo + one org).
  Each template contains a filing checklist with the UI flow per entity type, a signals-based justification of which entities qualify for suspension (vs. compromised legitimate accounts), and templated subject + body code blocks. Includes AUP citations and corroborating-third-party-write-up references.
- **Vercel abuse** (https://vercel.com/help) → **[`abuse-report-vercel.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md)**. Applies to the AjunaVerse-family generation only — the C2 hostnames serve the whole AjunaVerse cluster, so this filing is by nature cluster-wide. The realfraction-family C2 (`ipregionchecker.com`) is not Vercel-hosted; for that, file abuse with the domain's registrar (run `whois ipregionchecker.com` to identify it). Includes a reproducible 30-second curl probe the abuse-desk analyst can run to verify the C2's IP-allowlist gating themselves.

### 🛡 You're a blue-team / detection engineer building rules or feeding a SIEM/TIP

The IOCs and rules below cover the whole cluster, not just one repo.

- **IOCs** in spreadsheet-friendly CSV and tool-friendly JSON (suitable for MISP / STIX / OpenCTI ingestion). Per-generation:
  - AjunaVerse-family → **[`iocs.csv`](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv)** · **[`iocs.json`](./incidents/2026-05-13-ajunaverse-mvp/iocs.json)**
  - realfraction-family → **[`iocs.csv`](./incidents/2026-05-18-realfraction/iocs.csv)** · **[`iocs.json`](./incidents/2026-05-18-realfraction/iocs.json)**
- **Detection rules** — per-generation YARA, Sigma, and grep rules. The realfraction-family ruleset is additive to (not a replacement for) the AjunaVerse-family ruleset; run both:
  - AjunaVerse-family → **[`detection-rules.md`](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md)** (three YARA rules, three Sigma rules, grep one-liners).
  - realfraction-family → **[`detection-rules.md`](./incidents/2026-05-18-realfraction/detection-rules.md)** (two YARA rules, three Sigma rules, grep one-liners including a combined-generation grep).

### 🔍 You're a security researcher or threat-intel analyst who wants the full case file

The master analysis. Engagement context → repo-at-a-glance → execution-path matrix → annotated technical analysis of each loader (with verbatim code excerpts) → dynamic-analysis findings (target-IP allowlist gate confirmed live) → campaign attribution and ~15-sibling-repo footprint → IOCs in prose → reproducibility/methodology audit log of every command run during the investigation:

→ **[`incidents/2026-05-13-ajunaverse-mvp/README.md`](./incidents/2026-05-13-ajunaverse-mvp/README.md)** (~5400 words, structured by section so you can navigate)

### 🧠 You're studying how these traps are constructed — to harden against them, build something similar in a defensive lab, or write a teaching example

Same master file as the previous bullet, but jump straight to **§4 "Annotated technical analysis"** for the reverse-engineering walkthrough. Appendix A has verbatim code with the whitespace obfuscation reformatted out. Appendix B is the command-by-command audit log if you want to reproduce.

---

## Known campaign repositories

All members of the same broader "Contagious Interview" cluster, but spanning **multiple loader-code generations** that differ in idiom, C2 host family, and trigger surface. As of this writing, three generations are documented:

- **AjunaVerse-family, current generation** — loader at `server/routes/api/auth.js`; `verify(setApiKey(process.env.AUTH_API))` + `new Function("require", response.data)(require)`; `x-app-request: ip-check` magic header; Vercel-hosted C2; usually paired with a `.vscode/tasks.json` autorun and a `prepare` lifecycle hook.
- **AjunaVerse-family, earlier generation** — loader at `app/controllers/frontController.js`; same idiom as the current generation; different surrounding scaffold.
- **realfraction-family** — loader at `server/utils/regionChecker.js`; bare `eval(data)` after `https.request(..., {headers: {'x-secret-header': 'secret'}})`; non-Vercel C2 (`ipregionchecker.com`); triggered as a `require()` side-effect from `userController.js`; no `.vscode/tasks.json` and no `prepare`/`postinstall` hooks.

**The artifacts in this repo — briefing, abuse reports, detection rules, IOCs — apply across the whole campaign, but specific filenames are per-generation (linked above and in the per-incident folders).**

The catalog separates two distinct concerns:

- **Repositories** — what victims are sent. Useful for *self-identification* ("was I pointed at one of these?") and for *takedown* (the repos are all malicious; they all warrant removal).
- **Owning accounts and orgs** — the GitHub identities that host or commit to the repos. Useful for *filing decisions* (which entities should be reported for suspension, vs. which are themselves victims of a different attack).

A repo's owning account being operator-owned vs. a compromised legitimate developer doesn't change whether the repo is malicious — the loader is the loader. But it does change whether GitHub should suspend the account or just take down the repo.

### Confidence signals

Each entity below shows which verifiable signals it satisfies. **Multi-signal classifications are more trustworthy than single-signal ones.** Signals come in two groups, because they describe different units:

**Repo-level signals** — observable in the repo itself:

| Code | What it means |
|---|---|
| **L** | **Loader code of any documented generation present in the repo** (verified via direct review or GitHub code search on that generation's distinctive strings). Strongest single observable — the repo is part of the campaign. Per-row, the generation is noted in the table. |
| **T** | **VS Code `.vscode/tasks.json` autorun** on `folderOpen` with piped shell payload is present in the repo. (AjunaVerse-family-current generation only; absent from earlier AjunaVerse and from realfraction.) |
| **E** | **Committed `.env`** carries a base64-encoded `AUTH_API` value pointing at the campaign's Node-loader C2. (AjunaVerse-family generations only; realfraction hardcodes the C2 URL in source instead.) |
| **I** | **Bit-identical artifact** with another known cluster member (e.g. the same git blob SHA for `.vscode/tasks.json`) — proves cross-account operator coordination, not coincidence. |

**Account-level signals** — observable in the owning account/org profile:

| Code | What it means |
|---|---|
| **A** | **Naming matches operator convention** (`*WorkHub*`, `Hub9`, `Hub99`, numeric-`9`-suffix repo-naming pattern) or commit-author email uses the `+N` Gmail-alias persona convention. |
| **S** | **No legitimate-developer activity** — account hosts only campaign-shape repos, or is single-purpose and recently created. |
| **C** | **Cluster-created** with another operator account (same day + adjacent GitHub numeric ID — proves batch creation by one operator). |

Any verified **L** justifies taking down the repo regardless of account status. **A + S** (or **A + S + C**) on the account justifies asking GitHub to suspend the account.

### Repositories

All repos below are confirmed campaign members (L is verified for every row). The "Generation" column refers to the loader-code file path and idiom:

- *AjunaVerse-current* — `server/routes/api/auth.js` + `verify(setApiKey)` / `new Function("require", response.data)` / `x-app-request: ip-check` / Vercel C2.
- *AjunaVerse-earlier* — `app/controllers/frontController.js` + same idiom as AjunaVerse-current; different surrounding scaffold.
- *realfraction* — `server/utils/regionChecker.js` + `eval(data)` / `x-secret-header: secret` / non-Vercel C2 (`ipregionchecker.com`); triggered as a side-effect require from `userController.js`.

| Repository | Lure theme | Generation | Repo signals verified |
|---|---|---|---|
| [AjunaWorkHub/AjunaVerse_MVP](https://github.com/AjunaWorkHub/AjunaVerse_MVP) | Web3 metaverse | AjunaVerse-current | L · T · E · I |
| [AetSoftWorkHub/AetSoft_MVP](https://github.com/AetSoftWorkHub/AetSoft_MVP) | Web3 metaverse | AjunaVerse-current | L · T · I (via bit-identical `tasks.json` blob with AjunaVerse) |
| [DLabsHungary-Hub9/DLabs-Platform-MVP2](https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2) | Generic platform MVP | AjunaVerse-current | L |
| [roamanbuild/OnyxVerse](https://github.com/roamanbuild/OnyxVerse) | Web3 metaverse | AjunaVerse-current | L |
| [khaleb-dev/jackpot](https://github.com/khaleb-dev/jackpot) | Gambling | AjunaVerse-current | L |
| [rony1235/Jp-Soccer](https://github.com/rony1235/Jp-Soccer) | Sports betting | AjunaVerse-current | L |
| [mspkteam/williampotter](https://github.com/mspkteam/williampotter) | (unclear) | AjunaVerse-current | L |
| [Andrii-888/0gRollplay](https://github.com/Andrii-888/0gRollplay) | dApp / gaming | AjunaVerse-earlier | L |
| [prahaladbelavadi/CoinLocatorDemo](https://github.com/prahaladbelavadi/CoinLocatorDemo) | Crypto / locator demo | AjunaVerse-earlier | L |
| [sky-cook/tokentradingdapp](https://github.com/sky-cook/tokentradingdapp) | Token-trading dApp | AjunaVerse-earlier | L |
| [WilliamSuhosky/Property-Voting-DApp](https://github.com/WilliamSuhosky/Property-Voting-DApp) | Voting dApp | AjunaVerse-earlier | L |
| [artemus-jarrett/blockchain-voting-system](https://github.com/artemus-jarrett/blockchain-voting-system) | Voting dApp | AjunaVerse-earlier | L |
| [TechByteX/NitroGem](https://github.com/TechByteX/NitroGem) | (unclear) | AjunaVerse-earlier | L |
| [jamesm-dev/NitroGem](https://github.com/jamesm-dev/NitroGem) | (unclear) | AjunaVerse-earlier | L |
| [dappfusion/defi-real-estate](https://github.com/dappfusion/defi-real-estate) | Real-estate tokenization | AjunaVerse-earlier | L |
| [InvescoHub/defi-real-estate](https://github.com/InvescoHub/defi-real-estate) | Real-estate tokenization | AjunaVerse-earlier | L |
| [realfraction/realfraction](https://github.com/realfraction/realfraction) | Real-estate tokenization | realfraction | L (end-to-end review) |

Note that for most repos only the loader code (**L**) has been directly verified — that's the signal the GitHub code search hit on. The multi-signal rows (AjunaVerse, AetSoft) are the ones we've inspected end-to-end. The rest could have additional signals (**T**, **E**, **I**) but those would need direct inspection of each repo to confirm.

### Owning accounts and orgs

| Account / Org | Type | Verdict | Account signals | Notes |
|---|---|---|---|---|
| [AjunaWorkHub](https://github.com/AjunaWorkHub) | org | **Operator-owned** (suspend) | A · S · C | Org id 276264331, created 2026-04-27 in same-day adjacent-ID cluster with `AetSoftWorkHub`. Owns: `AjunaVerse_MVP`. |
| [AetSoftWorkHub](https://github.com/AetSoftWorkHub) | org | **Operator-owned** (suspend) | A · S · C | Org id 276275397, created same day as `AjunaWorkHub`. Owns: `AetSoft_MVP`. |
| [DLabsHungary-Hub9](https://github.com/DLabsHungary-Hub9) | org | **Operator-owned** (suspend) | A · S | Single-repo single-purpose org. `Hub9` suffix matches operator convention. Owns: `DLabs-Platform-MVP2`. |
| [GitWorkHub9](https://github.com/GitWorkHub9) | user | **Operator-owned** (suspend) | A | User id 272514006. Sole committer to `AjunaWorkHub/AjunaVerse_MVP`. Commit-author email `fatihafariya8+2@gmail.com` — `+N` Gmail-alias persona convention. |
| [GitWorkHub99](https://github.com/GitWorkHub99) | user | **Operator-owned** (suspend) | A · S | User id 213663943. Profile padded with ~20 clones of well-known OSS projects (`llama.cpp`, `prettier`, `angular-cli`, `nuxt.com`, `Xray-core`, …) — the publicly-documented "credibility farming" TTP. Hosts sibling campaign repo `AetSoftVerse`. |
| [roamanbuild](https://github.com/roamanbuild) | user | **Operator-owned** (suspend; not currently in the per-user filing checklist — candidate for addition) | A · S | All 6 account repos are campaign-shape (`OnyxVerse`, `ACN-Verse`, `Japanese-Royal`, plus `*-demo9` variants matching the operator's numeric-`9`-suffix persona convention). All created within a one-week window in May 2026. Owns: `OnyxVerse` + 5 siblings. |
| [khaleb-dev](https://github.com/khaleb-dev) | user | **Likely compromised legitimate** (investigate, don't suspend) | — | 55 repos over 5+ years across PHP/Java/Vue/Dart — clear real-developer portfolio. The `jackpot` repo appears to have been pushed via account compromise. |
| [rony1235](https://github.com/rony1235) | user | **Likely compromised legitimate** (investigate, don't suspend) | — | Account exists since 2017 with ~11 mostly-low-activity repos; three campaign-shape repos (`schooltutorial`, `japan-test`, `Jp-Soccer`) added April–May 2026 suggest recent compromise. |
| [mspkteam](https://github.com/mspkteam) | user | **Likely compromised legitimate** (investigate, don't suspend) | — | 5 mixed repos with the campaign one sandwiched between older and newer legitimate-looking projects (`fitnesssworldadminpanel`, `ETC-Coporative-code`, `specialized_medical`). |
| [Andrii-888](https://github.com/Andrii-888) | user | Uncertain (not investigated) | — | Owns: `0gRollplay`. Earlier-generation lure pattern leans toward "likely compromised legitimate" but not confirmed. |
| [prahaladbelavadi](https://github.com/prahaladbelavadi) | user | Uncertain (not investigated) | — | Owns: `CoinLocatorDemo`. |
| [sky-cook](https://github.com/sky-cook) | user | Uncertain (not investigated) | — | Owns: `tokentradingdapp`. |
| [WilliamSuhosky](https://github.com/WilliamSuhosky) | user | Uncertain (not investigated) | — | Owns: `Property-Voting-DApp`. |
| [artemus-jarrett](https://github.com/artemus-jarrett) | user | Uncertain (not investigated) | — | Owns: `blockchain-voting-system`. |
| [TechByteX](https://github.com/TechByteX) | user/org | Uncertain (not investigated) | — | Owns: `NitroGem`. |
| [jamesm-dev](https://github.com/jamesm-dev) | user | Uncertain (not investigated) | — | Owns: `NitroGem` (duplicate repo name). |
| [dappfusion](https://github.com/dappfusion) | user/org | Uncertain (not investigated) | — | Owns: `defi-real-estate`. |
| [InvescoHub](https://github.com/InvescoHub) | user/org | Uncertain (not investigated) | — | Owns: `defi-real-estate` (duplicate repo name). |
| [realfraction](https://github.com/realfraction) | org | **Operator-owned** (suspend) | A · S | Single-repo, single-purpose GitHub org. Contact email on lure-brand `realfraction.xyz` domain. Owns: `realfraction/realfraction` (realfraction-family generation). |

The detailed signals justifying each operator-owned classification (and the methodology used to verify them) are in each incident's case file. See [Incidents analyzed in this repo](#incidents-analyzed-in-this-repo) below.

### Encountered a repo not on this list?

If a recruiter pointed you at a repository that fits the same shape but isn't above, this diagnostic grep — widened to catch both the AjunaVerse-family and the realfraction-family generations in one shot — will tell you whether it's part of the broader cluster:

```
grep -RIn -E 'new Function\(["'\''"]require["'\''"],|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|ipregionchecker\.com' --exclude-dir=node_modules --exclude-dir=.git .
```

If it hits, please open an issue with the URL (or, if you have push access, add it to the matching incident's `iocs.csv` directly: [AjunaVerse](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) or [realfraction](./incidents/2026-05-18-realfraction/iocs.csv)).

---

## Incidents analyzed in this repo

| Date | Slug | Verdict | Quick links |
|---|---|---|---|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp/) | confirmed malicious; member of the "Contagious Interview" TTP cluster, ≥15 sibling repos (AjunaVerse-family generations) | [case file](./incidents/2026-05-13-ajunaverse-mvp/README.md) · [for devs](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md) · [Vercel abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md) · [IOCs](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) · [rules](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md) |
| 2026-05-18 | [realfraction](./incidents/2026-05-18-realfraction/) | confirmed malicious; sibling generation in the same "Contagious Interview" cluster (realfraction-family loader); single repo so far | [case file](./incidents/2026-05-18-realfraction/README.md) · [for devs](./incidents/2026-05-18-realfraction/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-18-realfraction/abuse-report-github.md) · [IOCs](./incidents/2026-05-18-realfraction/iocs.csv) · [rules](./incidents/2026-05-18-realfraction/detection-rules.md) |

Each incident folder contains the master analysis (with full operator identification, attribution, and the specific permalinks / commit SHAs analyzed) plus the audience-targeted derivatives. Incident-level artifacts apply across the cluster they identified during analysis — the [Known campaign repositories](#known-campaign-repositories) table above is the live catalog of all known cluster members across all incidents.

---

## About this repository

This is a personal workspace for analyzing developer-targeted social-engineering / malware campaigns encountered in the wild — typically delivered via fake recruiting outreach pointing at a malicious GitHub repository. Each case gets a dated directory with one canonical master analysis plus derivative artifacts for the different audiences who need to act on it.

### Ground rules

- Only *analysis*, *documentation*, and *evidence excerpts* live here.
- The full source trees of suspect/malicious repositories are **not** committed. Cases are inspected in disposable scratch directories (e.g. `/tmp/<repo>-static-review/`) and only the excerpts needed to support a finding are quoted.
- No attacker-controlled binary blobs, payload responses, or anything containing executable content from the campaigns being studied is committed.

### Layout

```
README.md                                          this file (audience-first entry point; cluster-level, no per-case specifics)
incidents/
  YYYY-MM-DD-<short-slug>/
    README.md                                      master analysis (the canonical record; contains case-specific specifics)
    briefing-for-developers.md                     short forwardable read for would-be victims
    abuse-report-github.md                         copy-paste template for GitHub T&S filings
    abuse-report-vercel.md                         copy-paste template for Vercel abuse filings
    iocs.csv                                       machine-readable IOCs (spreadsheet-friendly)
    iocs.json                                      machine-readable IOCs (tool-friendly)
    detection-rules.md                             YARA + Sigma + grep rules for blue-team detection
```

### Conventions

- One incident → one directory. Directory name is `YYYY-MM-DD-<slug>` where the date is the encounter date and the slug is the lure / target repo name (not the attacker's chosen branding).
- The master analysis is always `README.md` inside the incident directory, so GitHub renders it when you navigate in. **All per-case specifics — operator org/user names, GitHub IDs, commit-author emails, commit SHAs, dynamic-analysis observations — live here.**
- Abuse-report files are **templates with placeholders** for the case-specific bits (`<YOUR_REPO_URL>`, `<COMMIT_SHA>`, `<YOUR_NAME>`) — the filer fills in from the case file before submitting. Campaign-wide IOCs (operator-controlled org/user names, C2 hostnames, etc.) are kept concrete in the templates because they're the same for any filer in this campaign.
- Derivative artifacts use stable filenames (`briefing-for-developers.md`, `abuse-report-<service>.md`, `iocs.{csv,json}`, `detection-rules.md`) so they're predictable across incidents and audiences know exactly where to look.
- If a derivative type doesn't apply to a given incident (e.g., no Vercel-hosted C2 → no Vercel abuse report), omit the file rather than leaving an empty placeholder.
- The top-level README is the **cluster/campaign view** — campaign-catalog tables, audience routing, repo conventions. It does not name specific operator orgs or user accounts; those go in the incident case file.
