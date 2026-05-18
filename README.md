# Attack-campaign documentation

A growing record of developer-targeted malware campaigns analyzed in detail, paired with **copy-paste-ready artifacts for the different people who need to act on each one** — would-be victims, abuse desks, detection engineers, security researchers.

**Currently tracking:** an active developer-targeting operation matching the publicly-documented **"Contagious Interview" TTP cluster** (fake-recruiter → clone repo → `npm install`/`npm start` → stealer-loader). **≥43 confirmed repository instances** (16 AjunaVerse-family + 27 realfraction-family) across at least 11 operator-owned GitHub organizations plus several operator-owned and likely-compromised individual accounts. Loader code spans **at least three distinct generations** (two AjunaVerse-family + one realfraction-family with eight internal sub-shapes A–H), and the realfraction-family is now known to share Vercel hosting and an RCE primitive with the AjunaVerse-family (a cross-generation operator overlap; see incident 2026-05-18-realfraction §7.5 / §7.10). Five C2 hosts identified — two Vercel-hosted *.vercel.app deployments for AjunaVerse, plus for realfraction: `ipregionchecker.com` (registrar-frozen at Unstoppable Domains), `isillegalregion.com` (Vercel-hosted; live, serving stage-2 payload), `cookie-xi-seven.vercel.app` and `ip-check-api.vercel.app` (both DEPLOYMENT_DISABLED by Vercel). Operator activity observed at least through mid-May 2026.

If you arrived here because of one of the situations below, jump straight to the file that's for you. You don't need to read anything else first.

---

## Are you here because…

### 🚨 A recruiter just sent you a "Web3 / DeFi / metaverse / dApp / crypto-gaming MVP" repo and asked you to clone and run it ahead of an interview

**Stop.** It is very likely a trap.

The campaign covers **at least ~43 known repositories** across multiple GitHub organizations and accounts and **at least three distinct loader generations** (the realfraction-family generation alone has eight internal sub-shapes A–H) — see the [Known campaign repositories](#known-campaign-repositories) table below. If you were pointed at any of them — *or at any repo that fits the same shape* (single-author commit history, "Web3 MVP" framing, fresh GitHub org with one repo, README that pitches a multi-person team or generic-and-team-less platform) — read the developer briefing for the matching generation before doing anything else:

→ **[AjunaVerse-family briefing](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md)** — covers the `*.vercel.app` Vercel-C2 / `verify(setApiKey)` + `new Function("require", response.data)` generations (current and earlier).
→ **[realfraction-family briefing](./incidents/2026-05-18-realfraction/briefing-for-developers.md)** — covers the `ipregionchecker.com` C2 / `x-secret-header: secret` + `eval(data)` generation, where the loader lives in `server/utils/regionChecker.js` and is triggered as a side-effect `require()`.

Both are 5-minute reads. Forwardable to a colleague.

### 📮 You're filing a takedown report against any repo in this campaign

The abuse reports below are **copy-paste templates** that any campaign-affected reporter can use. Fill in your case-specific bits — the repo you were pointed at, the commit you analyzed, your name/handle — from the relevant incident's case file before submitting. The campaign-wide indicators (operator-controlled organizations and user accounts, C2 hostnames, etc.) are already in the templates because they're the same across the cluster.

- **GitHub Trust & Safety** (https://github.com/contact/report-abuse). Pick the template matching the generation of the repo you encountered:
  - AjunaVerse-family repos → **[`abuse-report-github.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md)** (six filings: one repo + three orgs + two users).
  - realfraction-family repos → **[`abuse-report-github.md`](./incidents/2026-05-18-realfraction/abuse-report-github.md)** (41-row filing checklist: 30 confirmed-malicious repo takedowns + 11 account-level filings against operator-owned orgs/users).
  Each template contains a filing checklist with the UI flow per entity type, a signals-based justification of which entities qualify for suspension (vs. compromised legitimate accounts), and templated subject + body code blocks. Includes AUP citations and corroborating-third-party-write-up references.
- **Vercel abuse** (https://vercel.com/help):
  - AjunaVerse-family → **[`abuse-report-vercel.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md)**. Cluster-wide; covers the two AjunaVerse-family *.vercel.app C2s.
  - realfraction-family → **[`abuse-report-vercel.md`](./incidents/2026-05-18-realfraction/abuse-report-vercel.md)**. Cluster-wide; covers three Vercel-hosted realfraction-family C2s — one currently live (`www.isillegalregion.com`) plus two already-DEPLOYMENT_DISABLED Vercel deployments. Includes a reproducible curl probe that returns a ~2.85 MB stage-2 payload from the live C2.
- **Registrar abuse** (per-registrar):
  - realfraction-family → **[`abuse-report-registrar.md`](./incidents/2026-05-18-realfraction/abuse-report-registrar.md)**. Two filings: Name.com (for `isillegalregion.com`, the live C2 apex) and Namecheap (for `realfraction.xyz`, the lure-brand domain). Note: the apex of the realfraction-family's first C2 (`ipregionchecker.com`, registered at Unstoppable Domains) is already on registrar `client hold` — no filing needed.
- **Gmail / Google** (https://support.google.com/mail/contact/abuse) → **[`abuse-report-gmail.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-gmail.md)**. AjunaVerse-family. The operator's commit-author Gmail (`fatihafariya8+2@gmail.com`) sits on a single parent inbox (`fatihafariya8@gmail.com`) that the `+N` alias convention routes every persona's mail back to. Action on the parent address simultaneously disables every operator persona at `+1`, `+2`, `+3`, … off that inbox — the single highest-leverage takedown vector in the AjunaVerse cluster. The case-specific recruiter-outreach Gmail address (if the filer received one in their inbox) is an optional placeholder; the body stands on the cluster-wide commit-author evidence without it. No realfraction-family equivalent yet — the realfraction trojan-commit authorship was re-attributed to real-developer handles rather than persona Gmails (see incident 2026-05-18-realfraction §7.2).
- **Calendly** (https://help.calendly.com/hc/en-us/requests/new) → **[`abuse-report-calendly.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-calendly.md)**. AjunaVerse-family. Applies when the recruiting outreach included a Calendly booking link. Inherently case-specific — Calendly URLs are per-persona and aren't surfaced on the malicious GitHub repos themselves — so the Calendly URL, persona name, and event title from the filer's recruiter message are the meat of the filing; the cluster-wide GitHub repo + Gmail-identity linkage is prefilled. AUP citation included.

### 🛡 You're a blue-team / detection engineer building rules or feeding a SIEM/TIP

The IOCs and rules below cover the whole cluster, not just one repo.

- **IOCs** in spreadsheet-friendly CSV and tool-friendly JSON (suitable for MISP / STIX / OpenCTI ingestion). Per-generation:
  - AjunaVerse-family → **[`iocs.csv`](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv)** · **[`iocs.json`](./incidents/2026-05-13-ajunaverse-mvp/iocs.json)**
  - realfraction-family → **[`iocs.csv`](./incidents/2026-05-18-realfraction/iocs.csv)** · **[`iocs.json`](./incidents/2026-05-18-realfraction/iocs.json)**
- **Detection rules** — per-generation YARA, Sigma, and grep rules. The realfraction-family ruleset is additive to (not a replacement for) the AjunaVerse-family ruleset; run both:
  - AjunaVerse-family → **[`detection-rules.md`](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md)** (three YARA rules, three Sigma rules, grep one-liners).
  - realfraction-family → **[`detection-rules.md`](./incidents/2026-05-18-realfraction/detection-rules.md)** (three YARA rules including a sub-shape-G constants-template detector, three Sigma rules updated 2026-05-18 to cover all four realfraction-family C2 hosts, and grep one-liners including a combined-generation grep with the Function.constructor RCE primitive).

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
- **realfraction-family** — `x-secret-header: secret` magic header across all sub-shapes; RCE via `eval(...)` or `new Function.constructor("require", ...)`; C2 host varies (`ipregionchecker.com`, `isillegalregion.com`, `cookie-xi-seven.vercel.app`, `ip-check-api.vercel.app`); loader file varies (eight known sub-shapes A–H: `server/utils/regionChecker.js`, `server/controllers/paymentController.js`, `backend/src/compliance/complianceService.js`, `server/mock/users.js`, `app/controllers/settingController.js` — cross-gen with AjunaVerse-earlier, `backend/src/utils/redis.js`, `backend/src/constants/index.js`, `backend/src/modules/departments/department-error.js`). Some sub-shapes (D/E/F) exfil `process.env` at loader stage; others (A/B/C/G/H) do not. Vercel hosts three of the four known C2 hosts. See [`incidents/2026-05-18-realfraction/README.md`](./incidents/2026-05-18-realfraction/README.md) §7.5 for the per-sub-shape breakdown.

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
- *AjunaVerse-earlier* — `app/controllers/frontController.js` + same idiom as AjunaVerse-current; different surrounding scaffold. Several rows in this generation **also carry the realfraction-family loader** at `app/controllers/settingController.js` (sub-shape E) — cross-generation dual-loader repos noted in the table.
- *realfraction* — `x-secret-header: secret` magic header across all sub-shapes A–H; loader file path and RCE primitive vary per sub-shape (see [`incidents/2026-05-18-realfraction/README.md`](./incidents/2026-05-18-realfraction/README.md) §7.5). C2 hosts: `ipregionchecker.com` (registrar-frozen), `isillegalregion.com` (Vercel; live), `cookie-xi-seven.vercel.app` and `ip-check-api.vercel.app` (both DEPLOYMENT_DISABLED).

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
| [realfraction/realfraction](https://github.com/realfraction/realfraction) | Real-estate tokenization | realfraction (sub-shape A) | L (end-to-end review) |
| [chainvisita-protocols/realfraction-mvp](https://github.com/chainvisita-protocols/realfraction-mvp) | Real-estate tokenization | realfraction (sub-shape A) | L · I (byte-identical loader to realfraction/realfraction except for C2 suffix) — formerly listed as `ChainVisitaTech/realfraction-mvp`; org was renamed |
| [slobodanmargetic988/real-world-assets](https://github.com/slobodanmargetic988/real-world-assets) | Real-estate tokenization | realfraction (sub-shape B) | L |
| [LandinLin/stockx_poc_1.03](https://github.com/LandinLin/stockx_poc_1.03) | StockX PoC matching-engine | realfraction (sub-shape C) | L |
| [devcode8/stock-home-assignment](https://github.com/devcode8/stock-home-assignment) | StockX PoC | realfraction (sub-shape C) | L |
| [0xbrentfi/StockX_PoC_1.03](https://github.com/0xbrentfi/StockX_PoC_1.03) | StockX PoC | realfraction (sub-shape C) | L |
| [0xbrentfi/StockX_PoC_1.04](https://github.com/0xbrentfi/StockX_PoC_1.04) | StockX PoC | realfraction (sub-shape C; inferred) | (inferred sibling on confirmed-operator-owned account; loader presence not directly verified) |
| [Chainbits1/StockX](https://github.com/Chainbits1/StockX) | StockX PoC | realfraction (sub-shape C) | L |
| [Lynqex-Labs/Stockx_PoC_v3](https://github.com/Lynqex-Labs/Stockx_PoC_v3) | StockX PoC | realfraction (sub-shape C) | L |
| [Lynqex-Labs/gas-optimization](https://github.com/Lynqex-Labs/gas-optimization) | StockX-adjacent | realfraction (sub-shape C; inferred) | (inferred sibling on confirmed-operator-owned org; loader presence not directly verified) |
| [metapulse54/RealEstateDemo](https://github.com/metapulse54/RealEstateDemo) | Real-estate tokenization | realfraction (sub-shape D) | L |
| [RockTxoi/DeFi-Estate](https://github.com/RockTxoi/DeFi-Estate) | Real-estate tokenization | realfraction (sub-shape D) | L |
| [jaiu3d/DeFi-Estate](https://github.com/jaiu3d/DeFi-Estate) | Real-estate tokenization | realfraction (sub-shape D) | L |
| [kio87j/DeFi-Estate](https://github.com/kio87j/DeFi-Estate) | Real-estate tokenization | realfraction (sub-shape D) | L |
| [kio87j/defi-estate-latest](https://github.com/kio87j/defi-estate-latest) | Real-estate tokenization | realfraction (sub-shape D; inferred) | (inferred sibling on confirmed-operator-owned org) |
| [ricardomartins9899/SmartPay-Demo](https://github.com/ricardomartins9899/SmartPay-Demo) | Payments demo | realfraction (sub-shape D) | L |
| [BVSLabs/blockchain-voting-system](https://github.com/BVSLabs/blockchain-voting-system) | Blockchain voting | realfraction (sub-shape E; cross-gen with AjunaVerse-earlier file path) | L |
| [Cortexa-org/NitroGem](https://github.com/Cortexa-org/NitroGem) | NitroGem (generic MVP) | realfraction (sub-shape E; cross-gen) | L |
| [eastmade/web3project-momo-token](https://github.com/eastmade/web3project-momo-token) | Token / meme trading | realfraction (sub-shape F) | L |
| [MBhatti26/Purrtal](https://github.com/MBhatti26/Purrtal) | Web3 meme trading | realfraction (sub-shape F) | L |
| [fabiolin/schoolmgmt](https://github.com/fabiolin/schoolmgmt) | School management | realfraction (sub-shape G) | L |
| [sharmapranay38/new_age_blockchain](https://github.com/sharmapranay38/new_age_blockchain) | Blockchain platform | realfraction (sub-shape G) | L |
| [shri33/Crypto-Trading-Platform](https://github.com/shri33/Crypto-Trading-Platform) | Crypto trading | realfraction (sub-shape G/H hybrid) | L |
| [Paulooo0/go-test](https://github.com/Paulooo0/go-test) | Skill-test scaffold | realfraction (sub-shape G) | L |
| [KagiyamaWeb/PyPDFMicroservise](https://github.com/KagiyamaWeb/PyPDFMicroservise) | Skill-test scaffold | realfraction (sub-shape G) | L |
| [Wilovy09/deby-assignment](https://github.com/Wilovy09/deby-assignment) | Skill-test scaffold | realfraction (sub-shape G) | L |
| [pablodiaz2799/solice-skill-test](https://github.com/pablodiaz2799/solice-skill-test) | Skill-test scaffold | realfraction (sub-shape G) | L |
| [Jay-Sojitra/student-management-system](https://github.com/Jay-Sojitra/student-management-system) | Student management | realfraction (sub-shape H) | L |
| [sparsh-kr24/Student-Management-System](https://github.com/sparsh-kr24/Student-Management-System) | Student management | realfraction (sub-shape H) | L |
| [ahmedraza90/test-fullstack](https://github.com/ahmedraza90/test-fullstack) | Skill-test scaffold | realfraction (sub-shape H) | L |

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
| [realfraction](https://github.com/realfraction) | org | **Operator-owned** (suspend) | A · S | Org id 279572783; created 2026-04-27. Single-repo single-purpose GitHub org. Contact email on lure-brand `realfraction.xyz` domain. Owns: `realfraction/realfraction` (realfraction-family sub-shape A). |
| [chainvisita-protocols](https://github.com/chainvisita-protocols) | org | **Operator-owned** (suspend) | A · S | Org id 266603464; created 2026-03-09. **Renamed from `ChainVisitaTech`** (the original 404 row has been resolved). Hosts `realfraction-mvp` (byte-identical loader to `realfraction/realfraction` except for C2 suffix) plus 11 same-day forks of well-known crypto/blockchain projects (TON SDK / iotex-core / wallet-kit / etc.) — classic credibility-farming TTP. |
| [Chainbits1](https://github.com/Chainbits1) | org | **Operator-owned** (suspend) | A · S | Org id 258889381; created 2026-02-02. Single-repo single-purpose org. Owns: `StockX` (sub-shape C). |
| [Lynqex-Labs](https://github.com/Lynqex-Labs) | org | **Operator-owned** (suspend) | A · S | Org id 254089140; created 2026-01-10. 7 repos: 2 campaign-shape (`Stockx_PoC_v3` + `gas-optimization`) plus 5 same-day forks of well-known crypto trading bots — credibility-farming TTP. |
| [metapulse54](https://github.com/metapulse54) | org | **Operator-owned** (suspend) | A · S | Org id 283720861; created 2026-05-11. Single-repo single-purpose org. Owns: `RealEstateDemo` (sub-shape D). |
| [RockTxoi](https://github.com/RockTxoi) | org | **Operator-owned** (suspend) | A · S · C | Org id 279846053; created 2026-04-27 **same day as `realfraction`**. Single-repo single-purpose org. Owns: `DeFi-Estate` (sub-shape D). |
| [jaiu3d](https://github.com/jaiu3d) | org | **Operator-owned** (suspend) | A · S | Org id 274630995; created 2026-04-08. Single-repo single-purpose org. Owns: `DeFi-Estate` (sub-shape D — duplicate repo name with kio87j/DeFi-Estate). |
| [kio87j](https://github.com/kio87j) | org | **Operator-owned** (suspend) | A · S | Org id 274335121; created 2026-04-07 (one day before jaiu3d). 2 repos both campaign-shape (`DeFi-Estate` + `defi-estate-latest`). |
| [BVSLabs](https://github.com/BVSLabs) | org | **Operator-owned** (suspend) | A · S | Org id 278779276; created 2026-04-23. Single-repo single-purpose org. Owns: `blockchain-voting-system` (sub-shape E; cross-gen file path with AjunaVerse-earlier). |
| [Cortexa-org](https://github.com/Cortexa-org) | user | **Operator-owned** (suspend) | A · S | User id 230719000; created 2025-09-06. Despite the `-org` suffix this is a User account. 6 repos all single-purpose vapor-MVP-shape (`NitroGem` + `EHR-Demo` + `intelhealthcare` + `intel-healthcare` + `Neura-MVP` + `neura-frontend`) — multi-lure-theme persona. |
| [0xbrentfi](https://github.com/0xbrentfi) | user | **Operator-owned** (suspend) | A · S | User id 263011287; created 2026-02-21. 6 repos: 2 confirmed campaign-shape (StockX_PoC_1.03 / _1.04) plus 4 small forks. |
| [ricardomartins9899](https://github.com/ricardomartins9899) | user | Uncertain — leans operator-owned | A · S | User id 241361229; created 2025-10-31. 1 repo (`SmartPay-Demo`). Single-purpose; harder to distinguish from a single-purpose alt. |
| [sparsh-kr24](https://github.com/sparsh-kr24) | user | Uncertain — leans operator-owned | S | User id 174685180; created 2024-07-04. 1 repo (`Student-Management-System`). Single-purpose but creation date is older than typical operator throwaways. |
| [slobodanmargetic988](https://github.com/slobodanmargetic988) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 25 repos since 2018; mixed real-developer history. Hosts sub-shape B realfraction-family loader at `server/controllers/paymentController.js`. |
| [LandinLin](https://github.com/LandinLin) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 9 repos since 2018. Hosts sub-shape C loader. |
| [devcode8](https://github.com/devcode8) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 62 repos since 2022. Hosts sub-shape C loader. |
| [eastmade](https://github.com/eastmade) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 20 repos since 2017. Hosts sub-shape F loader. |
| [MBhatti26](https://github.com/MBhatti26) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 10 repos since 2025-03. Hosts sub-shape F loader on `Purrtal`. |
| [fabiolin](https://github.com/fabiolin) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 10 repos since 2014. Hosts sub-shape G template on `schoolmgmt`. |
| [sharmapranay38](https://github.com/sharmapranay38) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 69 repos since 2020. Hosts sub-shape G loader. |
| [shri33](https://github.com/shri33) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 72 repos since 2020. Hosts sub-shape G/H hybrid loader. |
| [Paulooo0](https://github.com/Paulooo0) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 41 repos since 2022. Hosts sub-shape G template. |
| [KagiyamaWeb](https://github.com/KagiyamaWeb) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 14 repos since 2020. Hosts sub-shape G template. |
| [Wilovy09](https://github.com/Wilovy09) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 70 repos since 2020. Hosts sub-shape G template. |
| [pablodiaz2799](https://github.com/pablodiaz2799) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 17 repos since 2018. Hosts sub-shape G template. |
| [Jay-Sojitra](https://github.com/Jay-Sojitra) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 75 repos since 2021. Hosts sub-shape H loader (Function.constructor RCE). |
| [ahmedraza90](https://github.com/ahmedraza90) | user | **Likely compromised legitimate / candidate-fooled** (investigate, don't suspend) | — | 36 repos since 2020. Hosts sub-shape H loader. |

The detailed signals justifying each operator-owned classification (and the methodology used to verify them) are in each incident's case file. See [Incidents analyzed in this repo](#incidents-analyzed-in-this-repo) below.

### Encountered a repo not on this list?

If a recruiter pointed you at a repository that fits the same shape but isn't above, this diagnostic grep — widened to catch both the AjunaVerse-family and the realfraction-family generations in one shot — will tell you whether it's part of the broader cluster:

```
grep -RIn -E 'new Function(\.constructor)?\(["'\''"]require["'\''"],?|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app|ipcheck-encrypted|ip-check-encrypted' --exclude-dir=node_modules --exclude-dir=.git .
```

If it hits, please open an issue with the URL (or, if you have push access, add it to the matching incident's `iocs.csv` directly: [AjunaVerse](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) or [realfraction](./incidents/2026-05-18-realfraction/iocs.csv)).

---

## Incidents analyzed in this repo

| Date | Slug | Verdict | Quick links |
|---|---|---|---|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp/) | confirmed malicious; member of the "Contagious Interview" TTP cluster, ≥15 sibling repos (AjunaVerse-family generations) | [case file](./incidents/2026-05-13-ajunaverse-mvp/README.md) · [for devs](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md) · [Vercel abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md) · [IOCs](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) · [rules](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md) |
| 2026-05-18 | [realfraction](./incidents/2026-05-18-realfraction/) | confirmed malicious; realfraction-family generation in the same "Contagious Interview" cluster. As of 2026-05-18 cluster-expansion sweep: 27+ confirmed sibling repos across 4 C2 hosts in 8 loader-idiom sub-shapes (A–H). Cross-generation operator overlap with AjunaVerse confirmed | [case file](./incidents/2026-05-18-realfraction/README.md) · [for devs](./incidents/2026-05-18-realfraction/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-18-realfraction/abuse-report-github.md) · [Vercel abuse](./incidents/2026-05-18-realfraction/abuse-report-vercel.md) · [registrar abuse](./incidents/2026-05-18-realfraction/abuse-report-registrar.md) · [IOCs](./incidents/2026-05-18-realfraction/iocs.csv) · [rules](./incidents/2026-05-18-realfraction/detection-rules.md) |

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
    abuse-report-registrar.md                      copy-paste template for domain-registrar abuse filings (where the abuse path is domain revocation)
    abuse-report-gmail.md                          copy-paste template for Gmail / Google abuse filings
    abuse-report-calendly.md                       copy-paste template for Calendly Trust & Safety filings
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
