# If a recruiter sent you the DAAM AI-agent marketplace ZIP — read this first

A recruiter may pitch a "decentralized AI Agent marketplace" or AI/Web3 platform and send a LimeWire ZIP plus Calendly link before an interview. The DAAM archive analyzed here is a confirmed developer trap.

**Do not:**

- switch branches with Git in the supplied repository;
- run README install commands;
- run npm/yarn/pnpm;
- start the backend/frontend;
- open it in an IDE that may run project automation;
- inspect it on a machine containing SSH keys, cloud tokens, browser sessions, wallets, `.env` files, or valuable source repos.

## What is malicious

The ZIP contains three independent execution paths:

1. **Git hook:** `.git/hooks/post-checkout` downloads and executes code from `iploglab.store` when a branch checkout/switch happens.
2. **README installer:** the README tells victims to execute remote Bitbucket scripts under `daam602/ai-agent-marketplace`.
3. **Backend startup:** ordinary backend imports load a 2.9 MB obfuscated JavaScript payload disguised as `backend/src/config/.svn/.DS_Store`.

The hidden payload statically references process execution, filesystem access, HTTP communication, and strings associated with developer secrets. It contains C2 endpoints at `216.126.225.243:8085`, `:8086`, and `:8087`, overlapping with public Contagious Interview / OtterCookie reporting.

## If you already interacted with it

- **Downloaded only:** possession of the ZIP alone is not known to trigger the confirmed execution paths.
- **Switched branches, ran installer commands, or started the backend:** treat the host as potentially compromised.

Recommended response:

1. Disconnect from networks/VPNs.
2. Preserve the original ZIP and recruiter conversation as evidence.
3. Rotate SSH keys, GitHub/npm tokens, cloud credentials, LLM/API keys, and any secrets present in shell environment or local `.env` files.
4. Check wallet/browser-session exposure and move funds from browser wallets using a clean machine.
5. Audit persistence locations or reimage the host.
6. Report the involved platform accounts/resources using the templates in this incident folder.

## Quick indicators

- ZIP SHA-256: `b2545579ec4d36b1ad7d4e30221293ebb1ae472e9d8fb440e80a90a7e7db019d`
- Hook SHA-256: `9f63eb04c513bd8c12c2d7efce7b764d9731990fd434814899bd6621255262e9`
- Hidden payload SHA-256: `fb7f9d7b0b8dbd76feeed437378493c6142f728db8e726417d20cec5735bf9f5`
- Dropper domain: `iploglab.store`
- C2: `216.126.225.243:8085`, `216.126.225.243:8086`, `216.126.225.243:8087`
- Bitbucket: `daam602/ai-agent-marketplace`
- npm: `wallet-modal-223@1.0.0` / publisher `daam217` (associated/suspicious; not independently analyzed)
