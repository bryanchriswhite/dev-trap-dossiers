# Attack-campaign documentation

A private workspace for analyzing and documenting developer-targeted social-engineering / malware campaigns encountered in the wild. Used as the session anchor for Claude Code Web reviews of suspect repositories.

**Ground rules for this repo:**

- Only *analysis*, *documentation*, and *evidence excerpts* live here.
- The full source trees of suspect/malicious repositories are **not** committed. Inspect them in disposable scratch directories (e.g. `/tmp/<repo>-static-review/`) and quote only the excerpts needed to support a finding.
- Don't commit attacker-controlled binary blobs, payload responses, or anything that contains executable content from the campaigns being studied.
- Each incident gets its own dated directory under `incidents/`, containing the master analysis plus audience-targeted derivative artifacts.

## Layout

```
README.md                                          this file
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

The master file is the canonical record — engagement context, annotated technical analysis, dynamic-analysis findings, campaign attribution, IOCs in prose, recommended actions, and a reproducible methodology/audit log. Each derivative is generated from the master and serves one specific audience without forcing them to read the whole document.

Conventions:
- One incident → one directory. Directory name is `YYYY-MM-DD-<slug>` where the date is the encounter date and the slug is the lure / target repo (not the attacker's chosen branding).
- Master analysis is always `README.md` in the incident directory, so GitHub renders it when you navigate in.
- Derivative artifacts use stable filenames (`briefing-for-developers.md`, `abuse-report-<service>.md`, `iocs.{csv,json}`, `detection-rules.md`) so they're predictable across incidents.
- If a derivative type doesn't apply to a given incident (e.g., no Vercel-hosted C2 → no Vercel abuse report), omit the file rather than leaving an empty placeholder.

## Incidents

| Date       | Slug | Status | Verdict |
|------------|------|--------|---------|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp/) | reviewed | confirmed malicious; part of an active multi-org campaign matching the "Contagious Interview" TTP cluster |
