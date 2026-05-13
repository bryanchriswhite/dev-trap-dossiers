# Abuse report — Calendly

**Submission target:** https://help.calendly.com/hc/en-us/requests/new — Calendly's support form. Categorize the ticket as **Trust & Safety / abuse** if the form exposes that option; otherwise use the closest available "report a user / report abuse" option and explain in the body.

Unlike the GitHub, Vercel, and Gmail reports in this folder, the Calendly report is **inherently case-specific.** Calendly URLs are per-persona, and we don't have a single cluster-wide Calendly handle observed across multiple victims (Calendly URLs aren't surfaced on the malicious GitHub repositories themselves the way the C2 hostnames are). The cluster-wide indicators in this template are the campaign description, the GitHub repos that the Calendly link ultimately leads victims to, and the operator's Gmail-address identity. The case-specific bits — the Calendly URL the filer received, the persona name on the booking page, the event title, plus screenshots if available — are the meat of the filing.

This file is a **copy-paste template**. Find-and-replace the placeholders below before submitting.

### Placeholders to fill in

| Placeholder | What to put | Where to find it |
|---|---|---|
| `[Calendly URL]` | Full URL of the Calendly scheduling page you were sent, e.g. `https://calendly.com/<persona>/<event-name>` | The recruiter's message |
| `[Calendly persona name]` | The name shown on the Calendly booking page (the "host" of the event) | The Calendly page (top-of-page profile) |
| `[event title]` | The Calendly event title shown on the page, e.g. "Web3 MVP intro chat" | The Calendly page |
| `[reported repository URL]` | Full URL of a GitHub repo from the cluster you were ultimately pointed at, e.g. `https://github.com/<org>/<repo>` | The recruiter's follow-up message, the case file, or any of the cluster repos listed in the body |
| `[commit SHA]` | The full 40-char commit SHA you can permalink against on that repo | `git log -1 --format=%H` on a local clone, or the head SHA on the GitHub page |
| `[your name / handle]` | How you want the "Reported by" line to read | (yourself) |

1. Copy the contents of the **Subject** code block below into the ticket's subject / title field.
2. Copy the contents of the **Body** code block below into the ticket's description field. The body is plain text — reads sensibly even though the form doesn't render Markdown.
3. If the form allows attachments, include (a) a screenshot of the Calendly booking page showing the persona name and event title, and (b) a screenshot of the recruiter message that delivered the Calendly link.

> On the rendered GitHub view of this file, hover over each `text` code block to get GitHub's click-to-copy icon — you'll get the verbatim text, not the rendered version. **Do the placeholder fill-in before pasting.**

---

## Subject

```text
Trust & Safety — Calendly persona used as recruiting front for an active developer-targeting malware campaign (~15 GitHub repositories, "Contagious Interview" TTP cluster)
```

## Body

```text
SUMMARY

The Calendly scheduling URL [Calendly URL] (persona "[Calendly persona name]", event "[event title]") is being used as the recruiting-funnel front end for an active, currently-live developer-targeting malware campaign. Victims are pitched a fake "Web3 startup" role, scheduled via Calendly, and -- ahead of the booked call -- pointed at a GitHub repository to clone and run. The repository carries malware that fully compromises the victim's workstation on first run.

The campaign matches the publicly-documented "Contagious Interview" TTP cluster: fake-recruiter cold outreach -> instruction to clone and run a "Web3 MVP" repository ahead of an interview -> compromise on first run. Separate reports have been filed with GitHub Trust & Safety (covering the malicious repositories), Vercel (covering the C2 hosting), and Gmail / Google (covering the operator's email identity).


FULL CASE FILE

A public, regularly-maintained case file for this campaign -- including the annotated technical analysis, machine-readable IOCs (CSV/JSON), and detection rules (YARA, Sigma, grep) -- is at:

  https://github.com/bryanchriswhite/dev-trap-dossiers


REPORTED CALENDLY PAGE

URL:             [Calendly URL]
Persona name:    [Calendly persona name]
Event title:     [event title]


WHAT THE CALENDLY PAGE IS USED FOR

The Calendly booking flow is the operator's mechanism for:

1. Scheduling a video call with the victim to lend the engagement the appearance of a legitimate recruiting process.
2. Establishing a deadline ("the call is at <time>") that pressures the victim into cloning and running the malicious GitHub repository ahead of the call without time for careful review.
3. Capturing victim contact information (email, sometimes phone) through the Calendly intake form.

The Calendly page does not itself host the malware. The malware lives in the GitHub repository the victim is pointed at after booking (or in the recruiter's follow-up message containing the repo link). The Calendly page is the recruiting-funnel front end -- the entry point that walks the victim into the chain.


EVIDENCE THAT THE BOOKING LEADS TO MALWARE

After booking via the Calendly URL above, the filer was directed to the following GitHub repository:

  [reported repository URL]   (at commit [commit SHA])

The repository carries three independent code-execution vectors. Any one of them is sufficient to fully compromise a developer's workstation:

(1) VS Code .vscode/tasks.json with "runOn": "folderOpen" running per-OS curl|bash / wget|sh / curl|cmd against a Vercel-hosted shell-payload distribution host (vscode-settings-*.vercel.app). Opening the repo in VS Code is sufficient to fetch and execute the payload silently.

(2) package.json "prepare" lifecycle hook (and start/build/test/eject scripts) that launches an in-repo Express server during ordinary "npm install" or "npm start".

(3) On server boot, the code at server/routes/api/auth.js POSTs the victim's entire process.env (cloud, GitHub, npm, LLM-provider tokens) to a base64-obfuscated URL committed in .env as AUTH_API, then executes the response body as JavaScript via new Function("require", response.data)(require) -- arbitrary Node RCE as the developer's user.

The same loader is present in at least 14 sibling repositories across three GitHub organizations and several individual accounts -- see the case file at the URL above for the full enumeration. The loader is independently verifiable via GitHub code search on the distinctive strings 'verify(setApiKey(process.env.AUTH_API))' and 'new Function("require", response.data)'.


LINKAGE TO THE BROADER CAMPAIGN

The operator's commit-author identity on the malicious repositories is the Gmail address fatihafariya8+2@gmail.com -- a "+N" alias of the parent inbox fatihafariya8@gmail.com. The Gmail "+N" convention is the operator's bookkeeping for parallel personas off a single inbox; a separate Gmail abuse report has been filed against the parent address.

If the Calendly account behind the page [Calendly URL] is registered to an email address that matches fatihafariya8@gmail.com or any "+N" variant of it, that is a direct linkage between this Calendly persona and the cluster's known operator-controlled email identity. Calendly's internal record of the account's registered email would be the highest-confidence verification.


KNOWN AFFECTED GITHUB REPOSITORIES (the cluster the Calendly booking flow leads victims to)

Current-generation repos (loader at server/routes/api/auth.js):
- https://github.com/AjunaWorkHub/AjunaVerse_MVP
- https://github.com/AetSoftWorkHub/AetSoft_MVP
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


CALENDLY ACCEPTABLE USE POLICY OBSERVATIONS

Using the Calendly scheduling page above as the front end of this recruiting flow appears to violate Calendly's Acceptable Use Policy (https://calendly.com/legal/acceptable-use) on several axes:

- Prohibition on using the service to facilitate fraud or deception -- the recruiting persona is fictitious; the company does not exist; the role does not exist.
- Prohibition on using the service to distribute malware -- the booking flow is the entry point of a chain that delivers malicious code to victims.
- Prohibition on phishing and social engineering -- the recruiting pitch is designed to elicit code execution on the victim's workstation under the pretext of a take-home interview task.


REPORTED BY

[your name / handle]
```

---

## Supporting artifacts (for your reference; do not paste these into the ticket)

- Full technical analysis: [`README.md`](./README.md)
- IOCs in machine-readable form: [`iocs.csv`](./iocs.csv), [`iocs.json`](./iocs.json)
- Companion abuse report to GitHub T&S: [`abuse-report-github.md`](./abuse-report-github.md)
- Companion abuse report to Vercel: [`abuse-report-vercel.md`](./abuse-report-vercel.md)
- Companion abuse report to Gmail / Google: [`abuse-report-gmail.md`](./abuse-report-gmail.md)
