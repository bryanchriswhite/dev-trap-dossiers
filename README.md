# Attack-campaign documentation

A private workspace for analyzing and documenting developer-targeted social-engineering / malware campaigns encountered in the wild. Used as the session anchor for Claude Code Web reviews of suspect repositories.

**Ground rules for this repo:**

- Only *analysis*, *documentation*, and *evidence excerpts* live here.
- The full source trees of suspect/malicious repositories are **not** committed. Inspect them in disposable scratch directories (e.g. `/tmp/<repo>-static-review/`) and quote only the excerpts needed to support a finding.
- Don't commit attacker-controlled binary blobs, payload responses, or anything that contains executable content from the campaigns being studied.
- Each incident gets its own dated file under `incidents/`.

## Layout

```
README.md                                  this file
incidents/
  YYYY-MM-DD-<short-slug>.md               one master write-up per incident
```

A write-up typically covers: engagement context, the malicious mechanism (annotated, with quoted code excerpts), dynamic-analysis findings, campaign attribution, IOCs, recommended actions, and a methodology / audit log so the investigation is reproducible.

## Incidents

| Date       | Slug | Status | Verdict |
|------------|------|--------|---------|
| 2026-05-13 | [ajunaverse-mvp](./incidents/2026-05-13-ajunaverse-mvp.md) | reviewed | confirmed malicious; part of an active multi-org campaign matching the "Contagious Interview" TTP cluster |
