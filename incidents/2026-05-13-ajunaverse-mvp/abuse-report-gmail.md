# Abuse report — Gmail / Google

**Submission target:** https://support.google.com/mail/contact/abuse — Google's "Report Gmail abuse" form. Phishing-specific reports route to the same form via https://support.google.com/mail/answer/8253.

This file is a **copy-paste template**. Most of it (the cluster-wide operator Gmail address, the `+N` alias convention, the linkage to the malware-loader code, the list of affected GitHub repos) is already filled in — these indicators are the same for any filer in this cluster. The only bits you fill in are case-specific: which repo you cite as evidence, your name, and — optionally — the recruiter-outreach Gmail address from your own inbox if you have one. The body still stands on the cluster-wide commit-author evidence alone if you don't.

### Placeholders to fill in

Placeholders appear in the body as `[descriptive label in square brackets]` — find-and-replace each one before pasting.

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[reported repository URL]` | Full URL of a GitHub repo from the cluster you're citing as evidence, e.g. `https://github.com/<org>/<repo>` | The case file, or any of the cluster repos listed in the body |
| `[commit SHA]` | The full 40-char commit SHA you can permalink against on that repo | `git log -1 --format=%H` on a local clone, or the head SHA on the GitHub page |
| `[recruiter outreach Gmail address — optional]` | A Gmail address that contacted you as part of the recruiting pitch, if you received outreach from a Gmail address (often the host of a Calendly invite or the sender of a direct email). Leave the bracketed placeholder in place if you don't have one — the body still stands on the commit-author email evidence alone. | Your own inbox |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

The operator Gmail address `fatihafariya8@gmail.com` (and the observed `+2` variant `fatihafariya8+2@gmail.com`) is concrete in the body — it's cluster-wide, not a placeholder.

1. Copy the contents of the **Subject** code block below into the form's subject / title field.
2. Copy the contents of the **Body** code block below into the form's description field. The body is plain text — reads sensibly even though the form doesn't render Markdown.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **Do the placeholder fill-in before pasting.**

---

## Why we're filing this — and why it's high-leverage

The operator's identity substrate across the cluster is a single Gmail inbox: `fatihafariya8@gmail.com`. The `+N` alias convention they use (`fatihafariya8+2@gmail.com` is the variant observed in the wild) routes every variant — `+1`, `+2`, `+3`, … — back to the same parent inbox. Action on the parent address simultaneously disables every persona at every `+N` the operator has spun up or may yet spin up off that inbox. This is the single highest-leverage takedown vector in the cluster.

Care taken to file responsibly (mirrors the care taken in the per-account GitHub filings — we don't ask a platform to disable an account on weak signals):

- **Commit-author email on verifiable malware.** `fatihafariya8+2@gmail.com` is the sole commit-author identity on `https://github.com/AjunaWorkHub/AjunaVerse_MVP`, whose loader code is independently verifiable via GitHub code search on the distinctive strings `verify(setApiKey(process.env.AUTH_API))` and `new Function("require", response.data)`.
- **`+N` alias use is atypical of legitimate users.** Legitimate Gmail users rarely use plus-addressing as a persona-numbering substrate across malicious repositories; this pattern is consistent with operator bookkeeping, not normal mail filtering or signup-tracking use of plus-addressing.
- **Case-specific outreach (when present).** If the filer received recruiting outreach from a Gmail address — often the host of a Calendly scheduling link — that address is included under "Additional case-specific recruiter outreach address" below. It's optional; the body stands on the cluster-wide evidence without it.

---

## Subject

```text
Gmail address operating as commit-author identity and operator persona substrate for an active developer-targeting malware campaign (~15 GitHub repositories, "Contagious Interview" TTP cluster)
```

## Body

```text
SUMMARY

The Gmail address fatihafariya8@gmail.com (observed in the wild as the +N-aliased variant fatihafariya8+2@gmail.com) is operating as the operator-identity substrate of an active, currently-live developer-targeting malware campaign. The address appears as the commit-author email on a repository carrying verified malware-loader code, and the Gmail "+N" alias convention is being used to run parallel operator personas off a single inbox. The campaign distributes RCE and credential-theft payloads via at least 15 GitHub repositories across three organizations and several individual accounts. Separate reports have been filed with GitHub Trust & Safety, Vercel (covering the C2 hosting), and Calendly (covering the recruiting-funnel front end).

Because Gmail's "+N" plus-addressing routes all variants to the same parent inbox, action on the parent address fatihafariya8@gmail.com simultaneously disables every operator persona at +1, +2, +3, ... that the operator may already be running or may yet spin up off the same inbox. This is the single highest-leverage takedown vector in the cluster -- it is the reason this report is being filed alongside the GitHub and Vercel reports.


FULL CASE FILE

A public, regularly-maintained case file for this campaign -- including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) -- is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers


REPORTED GMAIL ADDRESS(ES)

Primary -- parent inbox, recommended target for action:
  fatihafariya8@gmail.com

Observed in-the-wild variant -- the operator's "+2" persona:
  fatihafariya8+2@gmail.com

Additional case-specific recruiter outreach address from the filer's own inbox (if known):
  [recruiter outreach Gmail address -- optional]


OBSERVED SIGNALS -- consistent with operator-controlled use, not a legitimate user

(1) COMMIT-AUTHOR EMAIL ON VERIFIED-MALICIOUS CODE.
    The address fatihafariya8+2@gmail.com appears in the git log of
      https://github.com/AjunaWorkHub/AjunaVerse_MVP
    as the sole commit-author identity (GitHub username GitWorkHub9, GitHub user id 272514006). Every commit in that repository -- including the commits that introduce the malware-loader code -- is authored under this Gmail address. The loader is independently verifiable via GitHub code search on the strings 'verify(setApiKey(process.env.AUTH_API))' and 'new Function("require", response.data)'.

(2) "+N" ALIAS CONVENTION AS PERSONA-NUMBERING SUBSTRATE.
    The "+2" suffix on fatihafariya8+2@gmail.com is consistent with the operator's bookkeeping for "this is the N-th persona/repo I'm running off this address." Use of Gmail plus-addressing as a persona-numbering substrate across malicious repositories is atypical of legitimate users (who use plus-addressing for inbox filtering and per-signup tracking, not for identity-segregation across malicious GitHub commits). The "+2" implies parallel personas at +1, +3, +4, etc. that may not have been observed yet in the wild but that share the same parent inbox.

(3) LINKAGE TO FAKE-RECRUITER OUTREACH FUNNEL.
    The campaign delivers its malicious repositories via cold outreach pretending to be recruiters from non-existent companies, frequently using Gmail for the outreach itself or as the backing account for a Calendly scheduling link sent to victims. If the filer received outreach from a Gmail address as part of this campaign, that address appears above under "Additional case-specific recruiter outreach address."


CAMPAIGN MECHANISM (so the reviewer can independently verify the loader is malicious)

Observed in [reported repository URL] at commit [commit SHA]:

The repositories in this campaign carry three independent code-execution vectors. Any one of them is sufficient to fully compromise a developer's workstation:

(1) VS Code .vscode/tasks.json with "runOn": "folderOpen" running per-OS curl|bash / wget|sh / curl|cmd against a Vercel-hosted shell-payload distribution host (vscode-settings-*.vercel.app).

(2) package.json "prepare" lifecycle hook (and start/build/test/eject scripts) that launches an in-repo Express server during ordinary "npm install" or "npm start".

(3) On server boot, the code at server/routes/api/auth.js POSTs the victim's entire process.env (cloud, GitHub, npm, LLM-provider tokens) to a base64-obfuscated URL committed in .env as AUTH_API, then executes the response body as JavaScript via new Function("require", response.data)(require) -- arbitrary Node RCE as the developer's user.

All three vectors are present in the repository whose git log carries fatihafariya8+2@gmail.com as the commit-author identity.


KNOWN AFFECTED GITHUB REPOSITORIES (the cluster this Gmail address is associated with)

Current-generation repos (loader at server/routes/api/auth.js):
- https://github.com/AjunaWorkHub/AjunaVerse_MVP   (fatihafariya8+2@gmail.com appears here as commit-author)
- https://github.com/AetSoftWorkHub/AetSoft_MVP    (sibling org, same-day creation, bit-identical .vscode/tasks.json blob)
- https://github.com/DLabsHungary-Hub9/DLabs-Platform-MVP2
- https://github.com/roamanbuild/OnyxVerse
- https://github.com/khaleb-dev/jackpot
- https://github.com/rony1235/Jp-Soccer
- https://github.com/mspkteam/williampotter

Earlier-generation repos (same loader code at a different file path):
- https://github.com/Andrii-888/0gRollplay
- https://github.com/prahaladbelavadi/CoinLocatorDemo
- https://github.com/sky-cook/tokentradingdapp
- https://github.com/WilliamSuhosky/Property-Voting-DApp
- https://github.com/artemus-jarrett/blockchain-voting-system
- https://github.com/TechByteX/NitroGem
- https://github.com/jamesm-dev/NitroGem
- https://github.com/dappfusion/defi-real-estate
- https://github.com/InvescoHub/defi-real-estate


GMAIL POLICY OBSERVATIONS

The observed use of this Gmail address appears to violate the Gmail Program Policies (https://support.google.com/mail/answer/81126) on:

- Phishing and social engineering -- the operator personas backed by this inbox impersonate employees of non-existent recruiting / Web3-startup companies and use Gmail-borne outreach to walk victims into running malicious code.
- Malware distribution -- the address is the commit-author identity on repositories that deliver remote-code-execution and credential-theft payloads to victims who reach them via the operator's outreach.
- Impersonation -- the recruiting personas are fictitious; the companies they claim to represent do not exist.


REPORTED BY

[your name / handle]
```

---

## Supporting artifacts (for your reference; do not paste these into the form)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
- Companion abuse report to Vercel: [`abuse-report-vercel.md`](./abuse-report-vercel.md)
- Companion abuse report to Calendly: [`abuse-report-calendly.md`](./abuse-report-calendly.md)
