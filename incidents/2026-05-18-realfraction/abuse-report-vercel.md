# Abuse report — Vercel (realfraction-family generation)

**Submission target:** https://vercel.com/help — search "Report abuse" or use https://vercel.com/security/abuse if available; otherwise file a support ticket categorized as *abuse / DMCA / legal*.

This file is a **copy-paste template** for the realfraction-family generation. Three Vercel-hosted deployments are involved (one currently live, two already DEPLOYMENT_DISABLED by Vercel) — the body below covers all three so this is a single filing rather than three separate ones. Most of it (the host list, the IPs, the affected repos, the live-probe results) is campaign-wide and already filled in; the only bits you fill in are the URL of *which* GitHub repo you cite as evidence plus your name.

### Placeholders to fill in

Placeholders appear in the body as `[descriptive label in square brackets]` — find-and-replace each one before pasting.

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[reported repository URL]` | Full URL of a GitHub repo from the cluster you're citing as evidence, e.g. `https://github.com/realfraction/realfraction` or any other repo from [README §7.5](./README.md#75-confirmed-sibling-repos-2026-05-18-cluster-expansion-sweep) | The case file |
| `[commit SHA]` | The full 40-char commit SHA you can permalink against on that repo | `git log -1 --format=%H` on a local clone, or the head SHA on the GitHub page |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

This filing is by nature cluster-wide, not specific to any single source repository — the three Vercel deployments serve the entire realfraction-family generation of the broader Contagious Interview cluster (~27 confirmed GitHub repos as of 2026-05-18).

1. Copy the contents of the **Subject** code block below into the ticket's subject / title field.
2. Copy the contents of the **Body** code block below into the ticket's description field.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **Do the placeholder fill-in before pasting.**

---

## Subject

```text
One live and two already-disabled Vercel deployments operating as C2 for an active developer-targeting malware campaign (realfraction-family generation, ~27 GitHub repositories, "Contagious Interview" TTP cluster)
```

## Body

```text
SUMMARY

Three Vercel deployments appear to be operating as command-and-control servers in the realfraction-family generation of an active, currently-live developer-targeting malware campaign. The campaign distributes RCE payloads via at least 27 GitHub repositories across multiple organizations and individual user accounts; the three Vercel hosts are referenced directly from the malicious GitHub repositories and serve the entire generation.

Two of the three are already DEPLOYMENT_DISABLED by Vercel — thank you. The third (isillegalregion.com) is still active and currently serving a ~2.85 MB stage-2 JavaScript payload. This filing requests takedown of the active deployment plus any related-account action.

A separate filing has been submitted with GitHub Trust & Safety covering the repository side, and a separate filing has been submitted with Name.com to revoke the isillegalregion.com domain.


FULL CASE FILE

A public, regularly-maintained case file for this campaign — including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) — is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers

The realfraction-family incident specifically:

  https://github.com/bryanchriswhite/dev-trap-dossiers/tree/main/incidents/2026-05-18-realfraction


VERCEL DEPLOYMENTS INVOLVED

  1. www.isillegalregion.com  — STILL ACTIVE — primary takedown target

     Custom domain on Vercel (NS: ns1.vercel-dns.com / ns2.vercel-dns.com; A: 64.29.17.65, 216.198.79.1)
     Apex domain registered 2026-02-13 at Name.com Inc.
     Endpoint: POST /api/ip-check-encrypted/3aeb34a39
     Role: stage-2 JavaScript loader C2
     Magic header expected on inbound: x-secret-header: secret
     Magic-header-gated response: HTTP 200 with ~2.85 MB single-line obfuscator.io-style obfuscated JS

     Reproducible probe from any IP (returns full payload on 2026-05-18):

       $ curl -sS -D - -o /tmp/payload.bin -X POST \
             -H 'x-secret-header: secret' --max-time 20 \
             'https://www.isillegalregion.com/api/ip-check-encrypted/3aeb34a39'
       HTTP/2 200
       content-type: text/html; charset=utf-8
       server: Vercel
       x-vercel-id: fra1::iad1::<random>
       content-length: 2914396

       $ sha256sum /tmp/payload.bin
       dbd065a1e8d525acf81428bf131240e7ffd2913538052387f83fe3df83659ee0

       $ file /tmp/payload.bin
       JavaScript source, ASCII text, with very long lines (65536), with no line terminators

  2. cookie-xi-seven.vercel.app  — ALREADY DEPLOYMENT_DISABLED — recorded for completeness

     Vercel subdomain (A: 64.29.17.3, 216.198.79.3)
     Endpoint: POST /api/ipcheck-encrypted/6KDisdfjlskjDI837KJH4
     Role: stage-2 loader C2 for sub-shape G (constants+loader template)
     Probe result on 2026-05-18:

       HTTP/2 451
       server: Vercel
       x-vercel-error: DEPLOYMENT_DISABLED

     Thank you for actioning this one. Including it here for cross-referencing the cluster.

  3. ip-check-api.vercel.app  — ALREADY DEPLOYMENT_DISABLED — recorded for completeness

     Vercel subdomain (A: 64.29.17.3, 216.198.79.3 — same edge IPs as #2)
     Endpoint: GET /api/ipcheck-encrypted/3948uf2uhe9r298rh2
     Role: stage-2 loader C2 for sub-shape H (Function.constructor RCE primitive)
     Probe result on 2026-05-18:

       HTTP/2 451
       server: Vercel
       x-vercel-error: DEPLOYMENT_DISABLED

     Same — thank you. Including for cross-referencing.

The shared Vercel edge IPs across #2 and #3 (and the same Vercel hosting on #1) suggest the three deployments are on the same Vercel account (or a small number of related accounts). Account-level action against the underlying Vercel project owner(s) would prevent the operator from re-spinning new deployments under fresh names.


EVIDENCE OF MALICIOUS USE — referenced from malicious GitHub repositories

The three C2 hostnames are referenced directly from the malicious repositories. A representative sample:

- www.isillegalregion.com is hardcoded into the loader at server/controllers/paymentController.js. See:
    https://github.com/slobodanmargetic988/real-world-assets/blob/main/server/controllers/paymentController.js

- cookie-xi-seven.vercel.app is hardcoded into the loader-constants file backend/src/constants/index.js. See:
    https://github.com/fabiolin/schoolmgmt/blob/main/backend/src/constants/index.js
    https://github.com/Paulooo0/go-test/blob/main/backend/src/constants/index.js
    https://github.com/KagiyamaWeb/PyPDFMicroservise/blob/main/backend/src/constants/index.js

- ip-check-api.vercel.app is hardcoded into the loader at backend/src/modules/departments/department-error.js. See:
    https://github.com/Jay-Sojitra/student-management-system/blob/main/backend/src/modules/departments/department-error.js
    https://github.com/sparsh-kr24/Student-Management-System/blob/main/backend/src/modules/departments/department-error.js
    https://github.com/ahmedraza90/test-fullstack/blob/main/backend/src/modules/departments/department-error.js

For the specific repo from which I was personally pointed at (or that I am citing as evidence), see:
    [reported repository URL]/blob/[commit SHA]/...


WHY THIS IS NOT A BENIGN VERCEL APP

The active deployment (#1) at www.isillegalregion.com:

- Is referenced as a loader endpoint by GitHub repos that pitch themselves as developer take-home assignments (e.g., real-estate-tokenization MVP, stock-exchange PoC, blockchain voting system).
- Returns an executable JavaScript body, not an HTML page or an API JSON payload, on the documented endpoint.
- The returned JS body is heavily obfuscated (obfuscator.io-style: object-property obfuscation, hex constants, string-table indirection) — entirely inconsistent with a legitimate web-application response.
- The endpoint is gated by a magic header (x-secret-header: secret) that no real application uses.
- The two sibling deployments (#2, #3) that Vercel has already disabled used identical conventions — magic-header-gated `/api/ipcheck-encrypted/<token>` endpoints serving stage-2 JS payloads. Vercel's prior disablement is itself evidence that #1 belongs in the same category.


KNOWN AFFECTED GITHUB REPOSITORIES (the cluster these C2 hosts serve)

Sub-shape A (regionChecker side-effect; C2 host #1's sibling ipregionchecker.com — currently on registrar client hold at Unstoppable Domains):
- https://github.com/realfraction/realfraction
- https://github.com/chainvisita-protocols/realfraction-mvp

Sub-shape B (inline paymentController; cites C2 host #1 = www.isillegalregion.com):
- https://github.com/slobodanmargetic988/real-world-assets

Sub-shape C (stockx config-driven axios.post; cites ipregionchecker.com):
- https://github.com/LandinLin/stockx_poc_1.03
- https://github.com/devcode8/stock-home-assignment
- https://github.com/0xbrentfi/StockX_PoC_1.03
- https://github.com/0xbrentfi/StockX_PoC_1.04
- https://github.com/Chainbits1/StockX
- https://github.com/Lynqex-Labs/Stockx_PoC_v3

Sub-shape D (mock/users.js verify, env-exfil-at-loader-stage):
- https://github.com/metapulse54/RealEstateDemo
- https://github.com/RockTxoi/DeFi-Estate
- https://github.com/jaiu3d/DeFi-Estate
- https://github.com/kio87j/DeFi-Estate
- https://github.com/ricardomartins9899/SmartPay-Demo

Sub-shape E (settingController verify; cross-generation with AjunaVerse-family-earlier):
- https://github.com/BVSLabs/blockchain-voting-system
- https://github.com/Cortexa-org/NitroGem

Sub-shape F (redis.js verify, env-exfil-at-loader-stage):
- https://github.com/eastmade/web3project-momo-token
- https://github.com/MBhatti26/Purrtal

Sub-shape G (constants+loader template; cites C2 host #2 = cookie-xi-seven.vercel.app):
- https://github.com/fabiolin/schoolmgmt
- https://github.com/sharmapranay38/new_age_blockchain
- https://github.com/shri33/Crypto-Trading-Platform
- https://github.com/Paulooo0/go-test
- https://github.com/KagiyamaWeb/PyPDFMicroservise
- https://github.com/Wilovy09/deby-assignment
- https://github.com/pablodiaz2799/solice-skill-test

Sub-shape H (Function.constructor RCE; cites C2 host #3 = ip-check-api.vercel.app):
- https://github.com/Jay-Sojitra/student-management-system
- https://github.com/sparsh-kr24/Student-Management-System
- https://github.com/ahmedraza90/test-fullstack


REQUESTED ACTION

1. Disable the deployment behind www.isillegalregion.com under Vercel's prohibition on hosting active malware infrastructure.

2. Audit the Vercel account(s) responsible for the three deployments (isillegalregion.com / cookie-xi-seven.vercel.app / ip-check-api.vercel.app); the IP-edge overlap suggests they share an owner. If the same account is responsible for all three, account-level suspension would prevent re-spinning under fresh deployment names.

3. Preserve access logs from all three deployments for forensic purposes — the request logs would carry the set of victim IPs that successfully fetched the stage-2 payload (i.e., developers whose machines are likely already compromised). This information would be highly valuable to the broader security community for victim-notification purposes.


REPORTED BY

[your name / handle]
```

---

## Supporting artifacts (for your reference; do not paste these into the ticket)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
- Companion abuse report to the registrars: [`abuse-report-registrar.md`](./abuse-report-registrar.md)
