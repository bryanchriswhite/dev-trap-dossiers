# Attack-campaign documentation

A growing record of developer-targeted malware campaigns analyzed in detail, paired with **copy-paste-ready artifacts for the different people who need to act on each one** — would-be victims, abuse desks, detection engineers, security researchers.

If you arrived here because of one of the situations below, jump straight to the file that's for you. You don't need to read anything else first.

---

## Are you here because…

### 🚨 A recruiter just sent you a "Web3 / DeFi / metaverse / dApp / crypto-gaming MVP" repo and asked you to clone and run it ahead of an interview

**Stop.** It is very likely a trap. Read this before doing anything else:

→ **[`incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md`](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md)** — 5-minute read. Tells you what the repo would actually do if you ran it, how to spot the trap on GitHub before cloning, how to inspect a freshly-cloned copy safely, what to do if you already ran it, and a single grep that catches the current campaign generation. Forwardable to a colleague.

### 📮 You're filing a takedown report

Two pre-filled ticket bodies, copy-paste straight in:

- **GitHub Trust & Safety** (https://github.com/contact/report-abuse) → **[`incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md)**. Includes permalinks at specific commit SHAs, Acceptable-Use-Policy citations, the list of ~15 sibling repos, and a corroborating-third-party-write-ups list to satisfy "weaponized in the wild" requests.
- **Vercel abuse** (https://vercel.com/help) → **[`incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md`](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md)**. Includes hostnames + IPs + a reproducible 30-second curl probe the analyst can run to verify the C2's IP-allowlist gating themselves.

### 🛡 You're a blue-team / detection engineer building rules or feeding a SIEM/TIP

- **IOCs** in spreadsheet-friendly CSV and tool-friendly JSON (suitable for MISP / STIX / OpenCTI ingestion):
  → **[`incidents/2026-05-13-ajunaverse-mvp/iocs.csv`](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv)**
  → **[`incidents/2026-05-13-ajunaverse-mvp/iocs.json`](./incidents/2026-05-13-ajunaverse-mvp/iocs.json)**
- **Detection rules** — three YARA rules (source-code scanning), three Sigma rules (process / DNS / proxy telemetry), four grep one-liners for analyst use, plus DNS/proxy blocklist guidance and a "rule maintenance" section explaining which rules are durable vs. fragile when the campaign rotates idioms:
  → **[`incidents/2026-05-13-ajunaverse-mvp/detection-rules.md`](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md)**

### 🔍 You're a security researcher or threat-intel analyst who wants the full case file

The master analysis. Engagement context → repo-at-a-glance → execution-path matrix → annotated technical analysis of each loader (with verbatim code excerpts) → dynamic-analysis findings (target-IP allowlist gate confirmed live) → campaign attribution and ~15-sibling-repo footprint → IOCs in prose → reproducibility/methodology audit log of every command run during the investigation:

→ **[`incidents/2026-05-13-ajunaverse-mvp/README.md`](./incidents/2026-05-13-ajunaverse-mvp/README.md)** (~5400 words, structured by section so you can navigate)

### 🧠 You're studying how these traps are constructed — to harden against them, build something similar in a defensive lab, or write a teaching example

Same master file as the previous bullet, but jump straight to **§4 "Annotated technical analysis"** for the reverse-engineering walkthrough. Appendix A has verbatim code with the whitespace obfuscation reformatted out. Appendix B is the command-by-command audit log if you want to reproduce.

---

## Incidents

| Date | Slug | Verdict | Quick links |
|---|---|---|---|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp/) | confirmed malicious; member of the "Contagious Interview" TTP cluster, ≥15 sibling repos | [master](./incidents/2026-05-13-ajunaverse-mvp/README.md) · [for devs](./incidents/2026-05-13-ajunaverse-mvp/briefing-for-developers.md) · [GH abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-github.md) · [Vercel abuse](./incidents/2026-05-13-ajunaverse-mvp/abuse-report-vercel.md) · [IOCs](./incidents/2026-05-13-ajunaverse-mvp/iocs.csv) · [rules](./incidents/2026-05-13-ajunaverse-mvp/detection-rules.md) |

---

## About this repository

This is a personal workspace for analyzing developer-targeted social-engineering / malware campaigns encountered in the wild — typically delivered via fake recruiting outreach pointing at a malicious GitHub repository. Each case gets a dated directory with one canonical master analysis plus derivative artifacts for the different audiences who need to act on it.

### Ground rules

- Only *analysis*, *documentation*, and *evidence excerpts* live here.
- The full source trees of suspect/malicious repositories are **not** committed. Cases are inspected in disposable scratch directories (e.g. `/tmp/<repo>-static-review/`) and only the excerpts needed to support a finding are quoted.
- No attacker-controlled binary blobs, payload responses, or anything containing executable content from the campaigns being studied is committed.

### Layout

```
README.md                                          this file (audience-first entry point)
incidents/
  YYYY-MM-DD-<short-slug>/
    README.md                                      master analysis (the canonical record)
    briefing-for-developers.md                     short forwardable read for would-be victims
    abuse-report-github.md                         copy-paste-ready ticket body for GitHub T&S
    abuse-report-vercel.md                         copy-paste-ready ticket body for Vercel abuse
    iocs.csv                                       machine-readable IOCs (spreadsheet-friendly)
    iocs.json                                      machine-readable IOCs (tool-friendly)
    detection-rules.md                             YARA + Sigma + grep rules for blue-team detection
```

### Conventions

- One incident → one directory. Directory name is `YYYY-MM-DD-<slug>` where the date is the encounter date and the slug is the lure / target repo name (not the attacker's chosen branding).
- The master analysis is always `README.md` inside the incident directory, so GitHub renders it when you navigate in.
- Derivative artifacts use stable filenames (`briefing-for-developers.md`, `abuse-report-<service>.md`, `iocs.{csv,json}`, `detection-rules.md`) so they're predictable across incidents and audiences know exactly where to look.
- If a derivative type doesn't apply to a given incident (e.g., no Vercel-hosted C2 → no Vercel abuse report), omit the file rather than leaving an empty placeholder.
