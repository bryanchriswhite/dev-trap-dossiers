# If a recruiter just sent you this kind of repo — read this first

Short briefing for developers who get pitched a "Web3 / DeFi / metaverse / dApp / crypto-gaming MVP" by a recruiter and asked to clone and run a repo before an interview.

**Estimated reading time: 5 minutes. Forwardable to a colleague.**

---

## The pattern

A "recruiter" reaches out — LinkedIn DM, Calendly link, Telegram, Discord, sometimes email — for a friendly-sounding contract or full-time role at a "Web3 startup." The project is described as a metaverse, a DeFi platform, a play-to-earn game, a sports-betting dApp, a real-estate-tokenization platform, or similar. You're sent a GitHub repository link and asked, ahead of the next call, to:

1. `git clone` the repo
2. `npm install`
3. `npm start` (or open it in VS Code so they can "screenshare your changes live")

**The repository is a trap.** It contains a working-looking application on the surface, with three independent code-execution payloads grafted on. The instant you do any of the steps above, your laptop is compromised.

This briefing is based on one current instance — `https://github.com/AjunaWorkHub/AjunaVerse_MVP` — and ~15 sibling repositories that share the same loader code. The campaign matches the publicly-documented "Contagious Interview" cluster (often attributed to DPRK-aligned actors), but the same delivery pattern is used by other groups too.

---

## What actually runs

The repo carries three independent loaders. **Any one of them is enough.** They're independent so that even if you happen to neutralize one, the others still fire.

1. **Open the folder in VS Code.** A `.vscode/tasks.json` file in the repo declares an OS-specific `curl … | bash` / `wget … | sh` / `curl … | cmd` task with `runOn: "folderOpen"` and all output suppressed. The moment VS Code finishes opening the folder, this task fetches a per-OS shell script from a Vercel-hosted server the attacker controls and executes it as your user, silently. (A *second* task in the same file silently runs `npm install`, which triggers loader 2.)

2. **`npm install`.** `package.json` contains `"prepare": "node server/server.js"`. The `prepare` lifecycle hook runs automatically after `npm install` finishes — it's like `postinstall`, but less notorious, so security tools and reviewers tend to skim past it. Running it boots the in-repo Express server, which fires loader 3.

3. **`npm start`, `npm run build`, `npm test`, `npm run eject`.** Each script begins with `node server/server.js | react-scripts …`. The pipe means the Express server runs no matter which command you type. The server's auth module is hardcoded so that the moment it's loaded, it (a) POSTs **your entire shell environment** — including any cloud / GitHub / npm / OpenAI / Anthropic / AWS tokens you have exported, plus any `.env` content — to a Vercel-hosted URL the attacker controls, and (b) executes the response body as Node code with full `require` access. That's full RCE on your laptop, plus credential theft.

The URL for (3) is committed in `.env` as a base64-encoded string under the key `AUTH_API`, so it looks like just another API key in the file. Decoded, it's a `vercel.app` subdomain run by the attacker.

---

## Visible signs at the GitHub stage — before you click

Things to look for on the GitHub page *before* cloning. None of these alone is proof; together they're a near-certain trap.

- **Single-author commit history despite a "team" README.** The README claims a multi-person team, but `git log` (or the Contributors graph on GitHub) shows one identity. Often the email is a `+N`-aliased Gmail (`foo+2@gmail.com`, `foo+3@gmail.com`).
- **`.env` and `.env.local` are committed.** Real projects gitignore these. A repo where they're in the tree is either careless (rare in a project polished enough to fool a recruiter pitch) or deliberately shipping a baked-in C2 pointer.
- **Recent commits dominated by `Update .env` and `Update tasks.json`.** Rotating the C2 pointer and the loader is the attacker's operational tempo. Real projects don't update `.env` every few weeks with a single-line commit message.
- **Brand-new org with one repo.** Orgs created within the last few months with exactly one public repo, no members, no other activity. Sometimes paired with a sibling org with similar naming, created the same day.
- **Generic Web3 buzzword stack in the README that doesn't match the code.** README claims Three.js / Babylon.js / Unity / Polygon / Hardhat / Python analytics; code is a CRUD app with a generic React frontend. The decoy and the pitch don't match.
- **No GitHub Actions, no Dockerfile, no CI configuration.** A "professional MVP" with zero CI is implausible.

## Visible signs at the local stage — after clone, before you run anything

If you've already cloned but not yet installed or opened in an editor, these files are worth opening (in a text editor that *does not* auto-run anything, e.g., `cat` or a barebones editor in a terminal):

- **`.vscode/tasks.json`** — look for any `"runOn": "folderOpen"` entry. Look hard at any per-OS `osx`/`linux`/`windows` blocks; the attacker may pad them with hundreds of spaces so the malicious `curl|bash` command sits far off the right edge of a single line. Wrap the file or read it through `cat -A` to expose padding.
- **`package.json` scripts section** — look for `prepare`, `preinstall`, `install`, `postinstall`, `prepublish` running anything other than test/build tooling. Treat `node …` or `bash …` in any of these as a red flag.
- **Committed `.env` files** — read every line. Any base64-looking value (`aHR0c…`, `aHR0a…` — these are the b64 prefixes for `http://` / `htta…`) that isn't supposed to be base64 (Stripe keys aren't, Alchemy keys aren't) deserves a `base64 -d` on it. If it decodes to a URL, that URL is the C2.
- **`server/` (or `backend/`, `app/`) controllers and routes** — search for `new Function(`, `eval(`, `atob(`, `Buffer.from(..., 'base64')`, `child_process`, `axios.post(.*process\.env)`, `axios.post(.*\.\.\.process\.env)`. Any one of these in a file that doesn't obviously need them is a flag.

Single grep that catches the current generation of this campaign on a freshly-cloned repo:

```
grep -RIn -E 'new Function\(["'\'']require["'\''],|verify\(setApiKey|x-app-request|runOn["'\''" ]*:[" ]*["'\'']folderOpen' .
```

If that returns anything, **do not run the project**.

---

## What to do if you already ran it

Assume the host is compromised. Don't try to clean it in place.

1. **Disconnect from network** (or at least from any VPN/SSO that authenticates you to shared resources).
2. **Rotate every credential that was in your shell environment** when you ran the install/start: cloud (AWS, GCP, Azure), GitHub, npm, OpenAI, Anthropic, any other LLM provider, Stripe, Sentry, etc. Revoke active OAuth tokens.
3. **Rotate SSH keys.** Treat `~/.ssh/` as exfiltrated. Generate a new key, push it to your services, revoke the old one. Audit `~/.ssh/authorized_keys` and `~/.ssh/config` for unfamiliar entries.
4. **Audit persistence locations.** Shell rcs (`.zshrc`, `.bashrc`, `.profile`, `.zprofile`, `.bash_profile`). `crontab -l`. `launchctl list` (macOS) / Windows Task Scheduler / `systemctl --user list-units` (Linux). Browser extensions (especially any new ones you don't remember installing).
5. **Move any browser-resident crypto wallet funds to a fresh wallet on a clean machine.** MetaMask / Phantom / Coinbase Wallet data is in the browser profile; assume the seed has been read.
6. **Review your GitHub account.** Personal access tokens (revoke any not in active use), SSH keys (remove any unfamiliar ones), OAuth apps and integrations, recent push activity for any unfamiliar repos or commits, recent fork/star activity.
7. **Strongly consider reimaging the machine.** A determined RCE payload in this family usually drops a persistence agent. Cleaning in place is an arms race you don't need to fight.
8. **Don't engage with the recruiter further.** Don't tell them you noticed. Don't unblock them. Save the conversation as evidence.
9. **Report.** GitHub Trust & Safety, Vercel abuse, and the platform the recruiter contacted you on (LinkedIn / Calendly / Discord / Telegram). Pre-written reports for GitHub and Vercel are in this incident folder; they take 60 seconds to file.

---

## How to share this with a colleague

Forward this paragraph:

> If a recruiter pitches you a "Web3 MVP" / "metaverse" / "DeFi dApp" job and asks you to clone a repo and `npm install` / `npm start` before a call — pause. Current campaigns hide loaders in `.vscode/tasks.json` (auto-runs when you open the folder in VS Code), the `prepare` script in `package.json` (runs on `npm install`), and in `server/routes/api/auth.js` / `app/controllers/frontController.js` (boots a remote-code loader from a base64-encoded URL in `.env`). Diagnostic strings: `verify(setApiKey(process.env.AUTH_API))`, `new Function("require", response.data)`, `"runOn": "folderOpen"`, `AUTH_API=aHR0c…`. C2 commonly on `vscode-settings-*.vercel.app` and `ip-core-api-*.vercel.app`. **Do not clone, install, or open in an editor** without inspecting `.vscode/tasks.json`, `package.json`'s scripts section, and any committed `.env`. Full write-up: [share the incident folder, or a screenshot].

---

## Why is this trap shaped this way?

A few notes for the curious — the choice of vectors is deliberate.

- **VS Code `tasks.json` with `runOn: folderOpen`.** This is the only place in modern dev tooling where opening a directory in your editor can run arbitrary shell commands. Most developers don't know it exists. The default-after-once-accepted prompt means many people have it implicitly enabled.
- **`prepare` over `postinstall`.** Security tools and code reviewers look hardest at `postinstall`. `prepare` runs in the same situations (after `npm install`) but is associated with publishing workflows, so it slips past attention.
- **`new Function("require", response.data)(require)` over `eval(response.data)`.** Both work; the former hands the attacker `require` so they can pull in `fs`, `child_process`, `os`, etc. It's also less likely to flag a JS lint rule that bans `eval`.
- **`{ ...process.env }` over a specific key list.** It's lazy from a stealth perspective (a heavier POST is more visible) but maximizes yield: in a successful run, the attacker doesn't have to know in advance which keys the victim exports.
- **Vercel `*.vercel.app` C2 with IP-allowlist gating.** Free, instantly-deployable hosting, no domain registration to leave a paper trail. Target-IP gating means automated scanners and security researchers get a 403; only the specific developer being interviewed sees the live payload.

The defense is simple: **don't run untrusted code, even when a "recruiter" tells you it's a take-home task**. Real recruiters at real companies don't ask you to install random repos before talking to you. If they really need a take-home, it's a tightly-scoped exercise in a service like CoderPad or a fresh sandbox, not a "run our production MVP locally."
