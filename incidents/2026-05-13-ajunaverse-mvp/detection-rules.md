# Detection rules

Rules for detecting this campaign across three observation surfaces: source code (YARA on repository contents), system telemetry (Sigma on process / DNS / HTTP events), and ad-hoc analyst use (grep one-liners). All rules are intentionally narrow — the chosen idioms are essentially zero-false-positive in ordinary code, so a single hit is high-signal.

---

## 1. YARA rules — for scanning repositories and source trees

```yara
rule AjunaVerse_NodeJS_RemoteCodeLoader
{
    meta:
        description = "AjunaVerse-family Node loader: POSTs process.env to a base64-obfuscated URL and `new Function`-executes the response"
        author      = "incident 2026-05-13"
        date        = "2026-05-13"
        severity    = "critical"
        reference   = "https://github.com/AjunaWorkHub/AjunaVerse_MVP"

    strings:
        $newfunc_dq      = "new Function(\"require\", response.data)"
        $newfunc_sq      = "new Function('require', response.data)"
        $setapikey_decl  = "const setApiKey = (s) => atob(s)"
        $verify_call     = "verify(setApiKey(process.env.AUTH_API))"
        $exfil_env       = /axios\.post\([^,]{1,80},\s*\{\s*\.\.\.process\.env\s*\}/
        $magic_header    = "x-app-request"
        $decoy_log       = "Aborting mempool scan due to failed API verification"

    condition:
        2 of them
}

rule AjunaVerse_VSCode_TasksJson_AutorunShell
{
    meta:
        description = "VS Code tasks.json with runOn:folderOpen executing curl|bash / wget|sh / curl|cmd silently"
        author      = "incident 2026-05-13"
        date        = "2026-05-13"
        severity    = "critical"

    strings:
        $runon       = /"runOn"\s*:\s*"folderOpen"/
        $silent      = /"reveal"\s*:\s*"silent"/
        $curl_pipe   = /"command"\s*:\s*"[^"]*curl[^"]*\|\s*(bash|sh|cmd)[^"]*"/
        $wget_pipe   = /"command"\s*:\s*"[^"]*wget[^"]*\|\s*(bash|sh)[^"]*"/

    condition:
        $runon and $silent and ($curl_pipe or $wget_pipe)
}

rule AjunaVerse_DotEnv_Base64C2Pointer
{
    meta:
        description = "Committed .env carrying an AUTH_API key whose value is base64-encoded HTTP(S) URL — characteristic loader-pointer"
        author      = "incident 2026-05-13"
        date        = "2026-05-13"
        severity    = "high"

    strings:
        // aHR0c == "http", aHR0cHM6Ly == "https://"
        $auth_api_http  = /^AUTH_API\s*=\s*aHR0c[A-Za-z0-9+\/=]{8,}/
        $auth_api_https = /^AUTH_API\s*=\s*aHR0cHM6Ly[A-Za-z0-9+\/=]{6,}/

    condition:
        filesize < 16KB and ($auth_api_http or $auth_api_https)
}
```

**Usage** (scan a freshly-cloned repo, recursively):

```
yara -r detection-rules.yar /path/to/cloned-repo
```

A hit on any of the three rules warrants treating the repo as untrusted until reviewed.

---

## 2. Sigma rules — for endpoint and network telemetry

These work against process-creation logs (Sysmon, EDR), DNS query logs, and HTTP-proxy logs.

### 2.1 DNS query to a campaign-pattern host

```yaml
title: Campaign DNS query for vscode-settings-* or ip-core-api-* on vercel.app
id: 4a8c9c9f-1a8b-4d4a-8b3a-ajuna1
status: experimental
description: DNS resolution of a developer host for a Vercel-hosted C2 pattern associated with the AjunaVerse / "Contagious Interview" campaign.
references:
  - https://github.com/AjunaWorkHub/AjunaVerse_MVP
author: incident 2026-05-13
date: 2026-05-13
logsource:
  category: dns
  product: any
detection:
  selection:
    QueryName|re: '(?i)^(vscode-settings-[^.]+|ip-core-api-[^.]+)\.vercel\.app$'
  condition: selection
falsepositives:
  - Genuine Vercel applications under similar names (very unlikely given specificity)
level: high
tags:
  - attack.initial-access
  - attack.t1566.003
```

### 2.2 VS Code spawning a shell that pipes curl/wget into bash/sh/cmd

This catches loader A's actual execution on Windows / macOS / Linux, regardless of which C2 domain is used.

```yaml
title: VS Code child shell running curl|bash or wget|sh shortly after folder open
id: 4a8c9c9f-1a8b-4d4a-8b3a-ajuna2
status: experimental
description: VS Code process spawning a shell that executes a piped curl/wget into a shell interpreter. Strongly indicative of a malicious .vscode/tasks.json with runOn:folderOpen.
references:
  - https://github.com/AjunaWorkHub/AjunaVerse_MVP
author: incident 2026-05-13
date: 2026-05-13
logsource:
  category: process_creation
  product: any
detection:
  parent_vscode:
    ParentImage|endswith:
      - '/Code.exe'
      - '\Code.exe'
      - '/Code Helper'
      - '/code'
      - '/code-insiders'
  shell_image:
    Image|endswith:
      - '/bash'
      - '/sh'
      - '/zsh'
      - '\cmd.exe'
      - '\powershell.exe'
  pipe_to_shell:
    CommandLine|re: '(?i)(curl|wget)[^|]{1,200}\|\s*(bash|sh|zsh|cmd)'
  condition: parent_vscode and shell_image and pipe_to_shell
falsepositives:
  - Developers running install scripts manually from a VS Code task (uncommon, audit case-by-case).
level: critical
tags:
  - attack.initial-access
  - attack.execution
  - attack.t1059
```

### 2.3 Node process making an HTTP POST whose body contains process.env keys

This is the network-side signal for loader C. Detection happens at a proxy / EDR with HTTP-body inspection.

```yaml
title: Node POST to *.vercel.app whose body contains shell-environment keys
id: 4a8c9c9f-1a8b-4d4a-8b3a-ajuna3
status: experimental
description: A node process POSTs JSON containing common shell-environment variable names to a *.vercel.app host. Diagnostic of the AjunaVerse-family loader exfiltrating process.env.
references:
  - https://github.com/AjunaWorkHub/AjunaVerse_MVP
author: incident 2026-05-13
date: 2026-05-13
logsource:
  category: proxy
  product: any
detection:
  selection_node:
    UserAgent|contains: 'axios'
  selection_host:
    Host|endswith: '.vercel.app'
  selection_method:
    Method: 'POST'
  selection_body_env:
    BodyContent|contains:
      - 'AWS_SECRET_ACCESS_KEY'
      - 'GITHUB_TOKEN'
      - 'NPM_TOKEN'
      - 'ANTHROPIC_API_KEY'
      - 'OPENAI_API_KEY'
      - 'HOMEBREW_GITHUB_API_TOKEN'
      - 'PATH'
      - 'USER'
      - 'SHELL'
      - 'HOME'
  selection_magic:
    Headers|contains: 'x-app-request: ip-check'
  condition: selection_method and selection_host and ((selection_node and selection_body_env) or selection_magic)
falsepositives:
  - A small number of legitimate Vercel-hosted backend services may POST a configuration object that incidentally includes one of these keys. The magic header `x-app-request: ip-check` is essentially zero-false-positive on its own.
level: critical
tags:
  - attack.exfiltration
  - attack.t1041
```

---

## 3. Grep / one-liner detections — for analysts

Useful as a pre-`npm install` check on a freshly-cloned repo.

```sh
# Catches the loader idiom across the current and earlier generations of this campaign.
# A single hit warrants treating the repo as untrusted.
grep -RIn --color=never -E \
  'new Function\("require",|new Function\(['\''"]require['\''"],|verify\(setApiKey|x-app-request|"runOn"\s*:\s*"folderOpen"' \
  --exclude-dir=node_modules --exclude-dir=.git .
```

Targeted variations:

```sh
# Loader C only (the Node `new Function` RCE primitive)
grep -RIn 'new Function(["'\''"]require' --exclude-dir=node_modules --exclude-dir=.git .

# Loader A only (VS Code auto-run shell payload)
grep -RIn -B2 -A2 '"runOn".*folderOpen' --exclude-dir=node_modules .vscode/

# Loader-pointer base64 in .env
grep -E '^[A-Z_]+=aHR0c' .env .env.* 2>/dev/null

# Decoded loader-pointer URLs from any .env in a tree
find . -name '.env*' -not -path '*/node_modules/*' -print0 \
  | xargs -0 grep -hE '^[A-Z_]+=aHR0c' \
  | sed 's/^[^=]*=//' \
  | while read v; do printf '%s -> ' "$v"; echo "$v" | base64 -d; echo; done
```

---

## 4. Static-blocklist suggestions

For organizations with web proxies or DNS sinkholes:

- Block resolution / outbound to:
  - `vscode-settings-0506.vercel.app`
  - `ip-core-api-one.vercel.app`
- Consider adding wildcard alerts (not blocks — too broad to block) for:
  - `vscode-settings-*.vercel.app`
  - `ip-core-api-*.vercel.app`
- Alert on HTTP request headers containing `x-app-request: ip-check` on outbound traffic from developer workstations.
- Alert on HTTP response bodies containing the exact 21-byte string `Host not in allowlist` paired with the response header `x-deny-reason: host_not_allowed` — this means a host on your network was attempting to reach the C2 from a non-allowlisted IP, almost certainly a sandbox or a researcher probe.

---

## 5. Notes on rule maintenance

- The campaign rotates C2 subdomains (the `-0506` in `vscode-settings-0506` is plausibly date-encoded). The hostname-pattern rules will keep working as new subdomains are deployed; the exact-hostname rules will need updating.
- The code-pattern rules are tied to specific JS idioms. The attacker can rename `setApiKey`, change `AUTH_API` to another key, or wrap `new Function` differently, and the rules will miss the next iteration. The Sigma rules (network and process behavior) are more durable than the YARA rules.
- The `Aborting mempool scan…` log string is a single-author tell. If a new campaign generation changes it, that's a useful artifact to capture; if not, it remains a uniquely diagnostic substring.
- The `x-app-request: ip-check` header is sent client→C2 and is the most durable code-level fingerprint of this loader family across both the current `server/routes/api/auth.js` and earlier `app/controllers/frontController.js` generations.
