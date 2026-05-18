# If you got pointed at `realfraction/realfraction` — read this first

Short briefing for any developer who was sent this repo (or another that looks like it) ahead of an interview, "take-home," or "code review with the team." Treat it as a trap.

**Estimated reading time: 5 minutes. Forwardable to a colleague.**

---

## The pattern

A "recruiter" or "founder" — typically reaching out via LinkedIn DM, Telegram, Discord, or email — describes a contract or full-time role at a "Web3" / blockchain / real-estate / DeFi startup and asks you, ahead of the next call, to:

1. `git clone` the repo
2. `npm install`
3. `npm start` (or open it in VS Code "so we can demo it together")

**The repository is a trap.** It contains a working-looking application on the surface with a code-execution loader grafted into one file. Running it — or starting any of its scripts — boots an Express server whose first act is to fetch JavaScript from an attacker-controlled URL and `eval()` it. That's full RCE on your laptop, with the same privileges as your shell.

This briefing is based on direct review of `https://github.com/realfraction/realfraction`. It applies as-is to that repo and to any other repo that fits the same shape (see [Was your repo one of these?](#was-your-repo-one-of-these)).

---

## What actually runs

The repo has **one** Node-side loader, but it's wired into the import graph such that any of the common npm scripts triggers it. Specifically:

- The file `server/utils/regionChecker.js` contains 10 lines of top-level code. At module load time — i.e., the moment Node `require`s the file — it POSTs to `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` with the header `x-secret-header: secret`, accumulates the response body, and runs `eval(<response>)` unless the body is literally `blocked` or parses to JSON with `{blocked: true}`.

- `server/controllers/userController.js` contains the line `const regionChecker = require("../utils/regionChecker");`. The variable `regionChecker` is never used anywhere in the file. The `require` is there only so the module's top-level code runs.

- `userController.js` is loaded by `userRoute.js`, which is loaded by `app.js`, which is loaded by `server.js`. Therefore the loader fires the moment the Express server starts.

- `package.json`'s scripts all start the Express server via `concurrently`:

  ```
  "start":     "concurrently \"node server/server.js\" \"react-scripts start\"",
  "build":     "concurrently \"node server/server.js\" \"react-scripts build\"",
  "test":      "concurrently \"node server/server.js\" \"react-scripts test\"",
  "deploy":    "concurrently \"node server/server.js\" \"gh-pages -d build\"",
  "predeploy": "concurrently \"node server/server.js\" \"npm run build\"",
  "eject":     "concurrently \"node server/server.js\" \"react-scripts eject\"",
  ```

  So **`npm start`, `npm test`, `npm run build`, `npm run deploy`, `npm run predeploy`, and `npm run eject` all fire the loader.** Even `npm test` runs the Express server, which is unusual and gives the loader many trigger paths.

- `npm install` *alone* does **not** fire the loader — `package.json` has no `preinstall` / `install` / `postinstall` / `prepare` hooks. Opening the folder in VS Code also does **not** fire the loader — there is no `.vscode/tasks.json` autorun in this repo. These are differences from other recent variants of the same broader campaign, where multiple independent loaders are common.

When the C2 returns real code, `eval(<response>)` runs it inside the loader's module scope, where `require`, `process`, `process.env`, `module`, `child_process` (one `require` away), the filesystem, your network, and your credentials are all reachable. A stage-2 payload in this family typically reads `~/.ssh/`, browser-resident wallet data, `.env`s in nearby project directories, shell environment variables (cloud, GitHub, npm, AI-provider tokens), and either exfiltrates them or installs a persistence agent. The C2 is presumably **target-IP gated** — researcher / sandbox IPs are returned `blocked`, the developer the operator is actually interviewing receives the live payload — by analogy with the other campaigns in this cluster that have been observed live.

---

## Visible signs at the GitHub stage — before you clone

Things to look for on the GitHub page *before* cloning. None of these alone is proof; together they're a near-certain trap.

- **Generic Web3 lure theme with no specific identity.** "Blockchain-powered smart real estate platform," "MVP / template stage," "scaffold for production implementation." Real companies pitch their team, their funding, their domain expertise. A README that's a feature-list of buzzwords with no founder names, no specific clients, no specific deployments is doing decoy work.
- **Fresh single-purpose owning account.** The owning org or user has exactly one substantive repo (the one you're being pointed at), no other public activity, no members shown, and a contact email on a brand-new domain. (For `realfraction` specifically: GitHub org with one public repo and `hello@realfraction.xyz`.)
- **`.vscode/` and `package-lock.json` in `.gitignore`.** Either is a yellow flag in a "professional MVP"; both together suggest the operator wants to avoid leaving an auditable record of either editor configuration or dependency hashes. (Note: in this repo, `.env` is also gitignored — different from earlier Contagious Interview campaign variants where `.env` was *committed* as a C2 carrier. Absence of a committed `.env` doesn't mean a repo is safe; the loader can live in source.)
- **Multiple commit-author handles but no live identities behind them.** When you click through to the GitHub profiles of the committers, are they real long-history developer accounts, or single-purpose accounts with no other activity? Cluster operators commonly fork an upstream legitimate codebase and replay its commit history under attribute-rewritten identities, which gives `git log` the *appearance* of a healthy multi-author history.
- **One specific commit whose subject doesn't match its diff.** For `realfraction/realfraction`, look at commit `8cc5120bfa3a64a6af14936ce821092ea08cd78d`: the subject says `"feat(server): add search, pagination, and apiFeatures utils"` but the diff adds only two unrelated files (`server/utils/errorHandler.js` and `server/utils/regionChecker.js`). That subject/diff mismatch is engineered for `git log` plausibility. If you spot one of these, read the actual diff.

## Visible signs at the local stage — after clone, before you run anything

If you've already cloned but not yet installed or opened in an editor, open these files in a text editor that *does not* auto-run anything (e.g., `cat`, `less`, a barebones terminal editor):

- **`server/utils/`** — every file. The malicious file in this repo is named `regionChecker.js`, but the operator can rename it to anything that sounds infrastructural (`featureFlags.js`, `telemetry.js`, `geoGate.js`, `setup.js`). Look for any utility module that contains a top-level `https.request(...)` or `http.request(...)` or `axios.get/post(...)` followed by `eval(...)` or `new Function(...)` on the response.
- **`server/controllers/*.js`** — search each file for `require(...)` statements that assign to a variable that's never used elsewhere in the file. A side-effect-only `require` of a local utility is almost always a smell.
- **`package.json`** `scripts` section — confirm that the scripts you intend to run (typically `start`, `test`, `build`) don't invoke `node server/...` alongside the CRA / Vite / Next runner via `concurrently`. If they do, the server boots whenever you think you're running the frontend.
- **`.vscode/tasks.json`** if present — look for any `"runOn": "folderOpen"` entry. (Absent in this repo, but present in other variants of the same broader campaign.)

A single grep that catches the realfraction-style loader on a freshly-cloned repo:

```
grep -RIn -E \
  "https\.request\([^)]*['\"]x-secret-header['\"]|x-secret-header['\"][[:space:]]*:[[:space:]]*['\"]secret['\"]|eval\(data\)" \
  --exclude-dir=node_modules --exclude-dir=.git .
```

A wider grep that catches both the realfraction-style and the AjunaVerse-style generations of the broader Contagious Interview cluster:

```
grep -RIn -E \
  'new Function\(["'\'']require["'\''],|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"' \
  --exclude-dir=node_modules --exclude-dir=.git .
```

If either returns anything, **do not run the project**.

---

## Was your repo one of these?

If the recruiter pointed you at `https://github.com/realfraction/realfraction`, **this briefing is for you exactly**. Do not clone, install, or start the project.

If your repo is **not** that URL but fits the same shape — Web3 / blockchain / real-estate / DeFi MVP framing, fresh single-purpose owning account, generic team-less README, multiple committers but no real-identity verification, `.vscode/` and `package-lock.json` both gitignored — assume it's the next iteration of the same broader campaign and treat it the same way. Run the wider grep above on a fresh clone; if it hits, the repo is in this family.

There's a parallel set of campaign-shape repos (different loader generations within the same broader Contagious Interview cluster) catalogued at the top-level [`README.md`](../../README.md) of this dossier — worth a glance to recognize sibling lure themes and operator naming conventions.

---

## What to do if you already ran it

Assume the host is compromised. Don't try to clean it in place.

1. **Disconnect from the network** (or at least from any VPN/SSO that authenticates you to shared resources).
2. **Rotate every credential that was in your shell environment** when you ran `npm start` (or any of the other scripts that start the server): cloud (AWS, GCP, Azure), GitHub, npm, OpenAI, Anthropic, any other LLM provider, Stripe, Sentry, etc. Revoke active OAuth tokens.
3. **Rotate SSH keys.** Treat `~/.ssh/` as exfiltrated. Generate a new key, push it to your services, revoke the old one. Audit `~/.ssh/authorized_keys` and `~/.ssh/config` for unfamiliar entries.
4. **Audit persistence locations.** Shell rcs (`.zshrc`, `.bashrc`, `.profile`, `.zprofile`, `.bash_profile`). `crontab -l`. `launchctl list` (macOS) / Windows Task Scheduler / `systemctl --user list-units` (Linux). Browser extensions (especially any new ones you don't remember installing).
5. **Move any browser-resident crypto wallet funds to a fresh wallet on a clean machine.** MetaMask / Phantom / Coinbase Wallet data is in the browser profile; assume the seed has been read.
6. **Review your GitHub account.** Personal access tokens (revoke any not in active use), SSH keys (remove any unfamiliar ones), OAuth apps and integrations, recent push activity for any unfamiliar repos or commits.
7. **Strongly consider reimaging the machine.** A determined RCE payload in this family usually drops a persistence agent. Cleaning in place is an arms race you don't need to fight.
8. **Don't engage with the recruiter further.** Don't tell them you noticed. Don't unblock them. Save the conversation as evidence.
9. **Report.** GitHub Trust & Safety (template in `abuse-report-github.md` in this folder), and the platform the recruiter contacted you on (LinkedIn / Calendly / Discord / Telegram). If you have the bandwidth, also report the C2 domain to the hosting / registrar abuse desk (the C2 host for this repo is `ipregionchecker.com` — registrar not yet identified at the time of analysis; `whois` on the domain will reveal it).

---

## How to share this with a colleague

Forward this paragraph:

> If a recruiter pitches you a Web3 / blockchain / real-estate / DeFi MVP and asks you to clone a repo and `npm install` / `npm start` before a call — pause. Specifically: `https://github.com/realfraction/realfraction` has a 10-line Node loader at `server/utils/regionChecker.js` that fires when the Express server boots, which it does on `npm start`, `npm test`, `npm run build`, `npm run deploy`, `npm run predeploy`, and `npm run eject`. It POSTs to `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` with header `x-secret-header: secret` and `eval()`s the response. Diagnostic strings: `x-secret-header`, `eval(data)`, `ipregionchecker.com`. Sibling campaigns (same broader cluster, different loader generations) use `verify(setApiKey(process.env.AUTH_API))`, `new Function("require", response.data)`, `x-app-request: ip-check`, `"runOn": "folderOpen"`. Full write-up: [share the incident folder, or a screenshot].

---

## Why is this trap shaped this way?

A few notes for the curious — the choice of vectors is deliberate.

- **Loader in `utils/`, not `controllers/` or `routes/`.** An earlier published archetype put `eval(remote)` inside `userController.js` itself, which is now well-known and easily grep-pable. Moving the malicious code into a freshly-named *sibling* utility module that `userController.js` simply `require`s for side-effects keeps the controller looking clean while routing the trigger through it. A reviewer skimming `userController.js` sees a normal `require` of a utility called `regionChecker` — which sounds like a routine geolocation guard, not a code loader.
- **Trigger via `require()` side-effect, not a function call.** `verify(setApiKey(process.env.AUTH_API))` (an earlier generation's trigger) is grep-friendly. Bare `require(...)` followed by an unused variable is not. A reader looking for "where is the loader invoked" finds nothing — because there's no invocation, just an import.
- **Hardcoded URL, not base64-in-`.env`.** The base64-in-`.env` trick is a well-known IOC pattern now. Hardcoding the URL inside a tiny utility module skips the `.env`-shipping step entirely, at the cost of being immediately visible to anyone who opens the file. The operator trades stealth-at-grep for not-having-to-ship-`.env`.
- **Empty POST body.** Earlier generations exfiltrated `process.env` inside the loader's outbound POST. This one doesn't. The trade-off is: smaller, more "normal-looking" traffic for the loader call (it now looks like a routine "is-this-IP-in-region" feature-flag check) at the cost of pushing exfil to stage 2 — which is fine for the operator, because stage 2 has full Node scope including `process.env`.
- **Two negative-gate sentinels.** `data === 'blocked'` and `JSON.parse(data)?.blocked` together let the C2 cheaply respond "you're not the target" without serving any code at all, and let the operator return decoy bodies to researchers without exposing the live payload. From the loader side, both fail closed (the `eval` is wrapped in `try/catch`, so a malformed payload silently does nothing).

The defense is simple: **don't run untrusted code, even when a "recruiter" tells you it's a take-home task**. Real recruiters at real companies don't ask you to install random repos before talking to you. If they really need a take-home, it's a tightly-scoped exercise in a service like CoderPad or a fresh sandbox, not a "run our production MVP locally."
