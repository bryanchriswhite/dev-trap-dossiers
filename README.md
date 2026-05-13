# Attack-campaign documentation

A growing record of developer-targeted malware campaigns analyzed in detail, paired with **copy-paste-ready artifacts for the different people who need to act on each one** — would-be victims, abuse desks, detection engineers, security researchers.

**Currently tracking:** an active developer-targeting operation matching the publicly-documented **"Contagious Interview" TTP cluster** (fake-recruiter → clone repo → `npm install`/`npm start` → stealer-loader). **≥15 known repository instances** across at least three GitHub organizations and several individual accounts. Two Vercel-hosted C2 servers, operator activity observed at least through mid-May 2026.

If you arrived here because of one of the situations below, jump straight to the file that's for you. You don't need to read anything else first.

---

## Are you here because…

### 🚨 A recruiter just sent you a "Web3 / DeFi / metaverse / dApp / crypto-gaming MVP" repo and asked you to clone and run it ahead of an interview

**Stop.** It is very likely a trap.

The campaign covers **at least ~15 known repositories** across multiple GitHub organizations and accounts — see the [Known campaign repositories](#known-campaign-repositories) table below. If you were pointed at any of them — *or at any repo that fits the same shape* (single-author commit history, "Web3 MVP" framing, committed `.env`, fresh GitHub org with one repo, README claims a multi-person team that the commit history doesn't support) — read the developer briefing before doing anything else:

→ **[`briefing-for-developers.md`](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md)** — 5-minute read. Tells you what the repo would actually do if you ran it, how to spot the trap on GitHub before cloning, how to inspect a freshly-cloned copy safely, what to do if you already ran it, and a single grep that catches the current campaign generation. **Applies to all known instances in the table below.** Forwardable to a colleague.

### 📮 You're filing a takedown report against any repo in this campaign

The abuse reports below are **copy-paste templates** that any campaign-affected reporter can use. Fill in your case-specific bits — the repo you were pointed at, the commit you analyzed, your name/handle — from the relevant incident's case file before submitting. The campaign-wide indicators (operator-controlled organizations and user accounts, C2 hostnames, etc.) are already in the templates because they're the same across the cluster.

- **GitHub Trust & Safety** (https://github.com/contact/report-abuse) → **[`abuse-report-github.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md)**. The campaign warrants multiple filings through GitHub's three abuse flows: one against the repository you encountered, plus per-entity filings against each operator-controlled organization and user account named in the case file. The template contains a filing checklist with the UI flow per entity type, a signals-based justification of which entities qualify for suspension (vs. compromised legitimate accounts), and templated subject + body code blocks for both the main report and the per-entity filings. Includes AUP citations and corroborating-third-party-write-up references.
- **Vercel abuse** (https://vercel.com/help) → **[`abuse-report-vercel.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md)**. The C2 hostnames serve the entire campaign, so this filing is by nature cluster-wide. Includes a reproducible 30-second curl probe the abuse-desk analyst can run to verify the C2's IP-allowlist gating themselves.

### 🛡 You're a blue-team / detection engineer building rules or feeding a SIEM/TIP

The IOCs and rules below cover the whole cluster, not just one repo.

- **IOCs** in spreadsheet-friendly CSV and tool-friendly JSON (suitable for MISP / STIX / OpenCTI ingestion):
  → **[`iocs.csv`](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv)**
  → **[`iocs.json`](./incidents/2026-05-13-ajunaverse-mvp/iocs.json)**
- **Detection rules** — three YARA rules (source-code scanning), three Sigma rules (process / DNS / proxy telemetry), four grep one-liners for analyst use, plus DNS/proxy blocklist guidance and a "rule maintenance" section explaining which rules are durable vs. fragile when the campaign rotates idioms:
  → **[`detection-rules.md`](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md)**

### 🔍 You're a security researcher or threat-intel analyst who wants the full case file

The master analysis. Engagement context → repo-at-a-glance → execution-path matrix → annotated technical analysis of each loader (with verbatim code excerpts) → dynamic-analysis findings (target-IP allowlist gate confirmed live) → campaign attribution and ~15-sibling-repo footprint → IOCs in prose → reproducibility/methodology audit log of every command run during the investigation:

→ **[`incidents/2026-05-13-ajunaverse-mvp/README.md`](./incidents/2026-05-13-ajunaverse-mvp/README.md)** (~5400 words, structured by section so you can navigate)

### 🧠 You're studying how these traps are constructed — to harden against them, build something similar in a defensive lab, or write a teaching example

Same master file as the previous bullet, but jump straight to **§4 "Annotated technical analysis"** for the reverse-engineering walkthrough. Appendix A has verbatim code with the whitespace obfuscation reformatted out. Appendix B is the command-by-command audit log if you want to reproduce.

---

## Known campaign repositories

All members of the same multi-org operation. Each carries the same `verify(setApiKey(process.env.AUTH_API))` + `new Function("require", response.data)` Node-loader idiom (or its earlier-generation equivalent). **The artifacts in this repo — briefing, abuse reports, detection rules, IOCs — apply across the whole campaign, not just to any one repo.**

### Confidence signals

A repo's "confidence" is the convergence of independent verifiable signals. **Multi-signal classifications are more trustworthy than single-signal ones**, and the breakdown matters because account suspension is only appropriate when the account itself looks operator-owned — a compromised legitimate developer's account that happens to host a campaign repo is the *victim* of a different attack, not the perpetrator.

| Code | What it means |
|---|---|
| **L** | **Loader code present in the repo.** The verifiable malware-loader idiom is in the repo's source (verified via GitHub code search). Strongest single observable — if **L** is verified, the repo itself should be taken down regardless of account status. |
| **T** | **VS Code `.vscode/tasks.json` autorun** on `folderOpen` with piped shell payload to `vscode-settings-*.vercel.app` is present in the repo. |
| **E** | **Committed `.env`** carries a base64-encoded `AUTH_API` value pointing at the campaign's Node-loader C2. |
| **I** | **Bit-identical artifact** with another known cluster member (e.g. the same git blob SHA for `.vscode/tasks.json`) — proves cross-account operator coordination, not coincidence. |
| **A** | **Account/org naming matches operator convention** (`*WorkHub*`, `Hub9`, `Hub99`, numeric-`9`-suffix persona pattern) or commit-author email uses the `+N` Gmail-alias persona convention. |
| **S** | **Owning account shows no legitimate-developer activity** — hosts only campaign-shape repos, or is single-purpose and recently created. |
| **C** | **Cluster-created** with another operator account (created same day + adjacent GitHub numeric ID — proves batch creation by one operator). |

Signals **L T E I** describe the **repo** itself. Signals **A S C** describe the **owning account**. **L** alone justifies taking down the repo; **A + S** (or **A + S + C**) on the account justifies asking GitHub to suspend the account.

The "Account verdict" column below distinguishes:
- **Operator-owned** — account is part of the campaign; suspend it.
- **Likely compromised legitimate** — repo is malicious but account belongs to a real developer who's a victim of the attack; take down the repo, investigate (don't suspend) the account.
- **Uncertain** — not investigated in depth; classification deferred.

### Current-generation loader (`server/routes/api/auth.js`)

| Repository | Lure theme | Signals | Account verdict |
|---|---|---|---|
| [AjunaWorkHub/AjunaVerse_MVP](https://github.com/AjunaWorkHub/AjunaVerse_MVP) | Web3 metaverse | **L · T · E · I · A · S · C** | Operator-owned (multi-signal; primary case file) |
| [AetSoftWorkHub/AetSoft_MVP](https://github.com/AetSoftWorkHub/AetSoft_MVP) | Web3 metaverse | **L · T · I · A · S · C** | Operator-owned (bit-identical `tasks.json` blob with AjunaVerse + same-day cluster creation with adjacent org ID) |
| [DLabsHungary-Hub9/DLabs-Platform-MVP2](https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2) | Generic platform MVP | **L · A · S** | Operator-owned (`Hub9`-naming match + single-repo single-purpose org) |
| [roamanbuild/OnyxVerse](https://github.com/roamanbuild/OnyxVerse) | Web3 metaverse | **L · A · S** | Operator-owned (all 6 account repos are campaign-shape: `OnyxVerse`, `ACN-Verse`, `Japanese-Royal`, plus `*-demo9` variants matching the operator's numeric-`9`-suffix persona convention; no legitimate activity) |
| [khaleb-dev/jackpot](https://github.com/khaleb-dev/jackpot) | Gambling | **L** | **Likely compromised legitimate** — owning account has 55 repos over 5+ years across PHP/Java/Vue/Dart, consistent with a real developer's portfolio. Repo should be taken down; account should be investigated for compromise rather than suspended. |
| [rony1235/Jp-Soccer](https://github.com/rony1235/Jp-Soccer) | Sports betting | **L** | **Likely compromised legitimate** — owning account exists since 2017 with ~11 mostly-low-activity repos; three campaign-shape repos (`schooltutorial`, `japan-test`, `Jp-Soccer`) added in April–May 2026 suggest recent compromise. Repo takedown only. |
| [mspkteam/williampotter](https://github.com/mspkteam/williampotter) | (unclear) | **L** | **Likely compromised legitimate** — owning account hosts older legitimate-looking repos (`fitnesssworldadminpanel`, `ETC-Coporative-code`, `specialized_medical`) sandwiching the campaign one. Repo takedown only. |

### Earlier-generation loader (`app/controllers/frontController.js`)

Same loader code, different scaffold. The accounts hosting these repos look like **compromised legitimate developers** more often than the current-generation set — varied project portfolios, older creation dates, mostly low-activity. The lure delivery to a victim still works either way.

| Repository | Lure theme | Signals | Account verdict |
|---|---|---|---|
| [Andrii-888/0gRollplay](https://github.com/Andrii-888/0gRollplay) | dApp / gaming | **L** | Likely compromised legitimate |
| [prahaladbelavadi/CoinLocatorDemo](https://github.com/prahaladbelavadi/CoinLocatorDemo) | Crypto / locator demo | **L** | Likely compromised legitimate |
| [sky-cook/tokentradingdapp](https://github.com/sky-cook/tokentradingdapp) | Token-trading dApp | **L** | Likely compromised legitimate |
| [WilliamSuhosky/Property-Voting-DApp](https://github.com/WilliamSuhosky/Property-Voting-DApp) | Voting dApp | **L** | Likely compromised legitimate |
| [artemus-jarrett/blockchain-voting-system](https://github.com/artemus-jarrett/blockchain-voting-system) | Voting dApp | **L** | Likely compromised legitimate |
| [TechByteX/NitroGem](https://github.com/TechByteX/NitroGem) | (unclear) | **L** | Uncertain (not investigated) |
| [jamesm-dev/NitroGem](https://github.com/jamesm-dev/NitroGem) | (unclear) | **L** | Uncertain (not investigated) |
| [dappfusion/defi-real-estate](https://github.com/dappfusion/defi-real-estate) | Real-estate tokenization | **L** | Uncertain (not investigated) |
| [InvescoHub/defi-real-estate](https://github.com/InvescoHub/defi-real-estate) | Real-estate tokenization | **L** | Uncertain (not investigated) |

The "Uncertain (not investigated)" entries are accounts whose profiles haven't been individually checked — the loader is verified in their repos via code search, but we haven't analyzed their owning-account histories. These could be either operator-owned or compromised legitimate.

The operator-controlled organizations and user accounts and the per-entity signals justifying their classification (creation dates, GitHub IDs, naming patterns, commit-author email conventions, etc.) are catalogued in detail in each incident's case file. See the case file linked under [Incidents analyzed in this repo](#incidents-analyzed-in-this-repo) below.

### Encountered a repo not on this list?

If a recruiter pointed you at a repository that fits the same shape but isn't above, the briefing's diagnostic grep — `grep -RIn -E 'new Function\(["'\''"]require["'\''"],|verify\(setApiKey|x-app-request|"runOn":[[:space:]]*"folderOpen"' .` — will tell you in one shot whether it's the same campaign. If it hits, please open an issue with the URL (or, if you have push access, [add it to the IOCs](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) directly).

---

## Incidents analyzed in this repo

| Date | Slug | Verdict | Quick links |
|---|---|---|---|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp/) | confirmed malicious; member of the "Contagious Interview" TTP cluster, ≥15 sibling repos | [case file](./incidents/2026-05-13-ajunaverse-mvp/README.md) · [for devs](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md) · [Vercel abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md) · [IOCs](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) · [rules](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md) |

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
