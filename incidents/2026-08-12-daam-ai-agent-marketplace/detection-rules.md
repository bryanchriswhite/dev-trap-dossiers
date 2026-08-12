# Detection rules — DAAM AI-agent marketplace trap

Rules for detecting the DAAM developer-trap archive across source trees, endpoint telemetry, and ad-hoc analyst checks. These are additive to the existing AjunaVerse and realfraction-family rules.

The source rules intentionally target loader structure, paths, and IOCs. They do not include full malware payload bytes.

---

## 1. YARA rules — repository/source scanning

```yara
rule DAAM_Git_PostCheckout_Downloader_Hook
{
    meta:
        description = "DAAM malicious Git post-checkout hook: OS-specific bootstrap from iploglab.store"
        author = "dev-trap-dossiers incident 2026-08-12"
        date = "2026-08-12"
        severity = "critical"

    strings:
        $domain = "iploglab.store"
        $mac = "/api/terminal/bootstrap?os=mac&flag=7"
        $linux = "/api/terminal/bootstrap?os=linux&flag=7"
        $windows = "/api/terminal/windows?flag=7"
        $hook_name = "post-checkout"
        $iex = "Invoke-Expression"
        $bash = "bash"

    condition:
        $domain and 2 of ($mac, $linux, $windows, $iex, $bash)
}

rule DAAM_Hidden_DSStore_JS_Payload_Import
{
    meta:
        description = "DAAM backend imports hidden JavaScript payload disguised as .svn/.DS_Store"
        author = "dev-trap-dossiers incident 2026-08-12"
        date = "2026-08-12"
        severity = "critical"

    strings:
        $hidden_path = "../config/.svn/.DS_Store"
        $svn_dsstore = ".svn/.DS_Store"
        $create_require = "createRequire"
        $require_call = /require\s*\(\s*['\"][.][.][\/\\]config[\/\\][.]svn[\/\\][.]DS_Store['\"]\s*\)/

    condition:
        ($create_require and ($hidden_path or $require_call)) or $svn_dsstore
}

rule DAAM_Obfuscated_Node_Payload_Static_IOCs
{
    meta:
        description = "DAAM hidden obfuscated Node payload static IOCs: modified base64 alphabet, large string table traits, C2 endpoints"
        author = "dev-trap-dossiers incident 2026-08-12"
        date = "2026-08-12"
        severity = "critical"

    strings:
        $c2_8086 = "http://216.126.225.243:8086/upload"
        $c2_8085 = "http://216.126.225.243:8085/upload"
        $c2_8087 = "http://216.126.225.243:8087"
        $alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/="
        $execsync = "execSync"
        $spawn = "spawn"
        $ssh = ".ssh"
        $aws = ".aws"
        $wsl = "WSL_DISTRO_NAME"

    condition:
        any of ($c2_*) or
        ($alphabet and 3 of ($execsync, $spawn, $ssh, $aws, $wsl))
}

rule DAAM_Backend_Gitignore_DSStore_Exception
{
    meta:
        description = "Suspicious .DS_Store retention signal used by DAAM to preserve hidden payload"
        author = "dev-trap-dossiers incident 2026-08-12"
        date = "2026-08-12"
        severity = "informational"

    strings:
        $keep_dsstore = "!.DS_Store"
        $dsstore = ".DS_Store"

    condition:
        $keep_dsstore and $dsstore
}
```

---

## 2. Sigma concepts — endpoint and network telemetry

### 2.1 Git hook or shell downloads from `iploglab.store`

```yaml
title: DAAM Git Hook Bootstrap From iploglab.store
id: 3c42e6f1-DAAM-0001
status: experimental
description: Detects shell/PowerShell download-and-execute behavior to DAAM hook bootstrap infrastructure.
references:
  - https://github.com/bryanchriswhite/dev-trap-dossiers/tree/main/incidents/2026-08-12-daam-ai-agent-marketplace
author: dev-trap-dossiers incident 2026-08-12
logsource:
  category: process_creation
  product: any
detection:
  selection_domain:
    CommandLine|contains: 'iploglab.store'
  selection_hook:
    CommandLine|contains: 'post-checkout'
  selection_exec:
    CommandLine|contains:
      - 'bash'
      - 'Invoke-Expression'
      - 'powershell'
      - 'wget'
      - 'curl'
  condition: selection_domain and selection_exec
falsepositives:
  - None expected in normal developer workflows.
level: critical
tags:
  - attack.execution
  - attack.t1059
```

### 2.2 Node process contacting DAAM C2 IP/ports

```yaml
title: DAAM Node Process Contacting 216.126.225.243 C2 Ports
id: 3c42e6f1-DAAM-0002
status: experimental
description: Detects outbound connections to C2 endpoints statically recovered from the DAAM hidden payload.
author: dev-trap-dossiers incident 2026-08-12
logsource:
  category: network_connection
  product: any
detection:
  selection_ip:
    DestinationIp: '216.126.225.243'
  selection_ports:
    DestinationPort:
      - 8085
      - 8086
      - 8087
  condition: selection_ip and selection_ports
falsepositives:
  - Unknown legitimate services on the same IP/ports; treat hits from developer workstations as high severity.
level: critical
tags:
  - attack.command-and-control
  - attack.exfiltration
```

### 2.3 Access to hidden payload path in developer worktree

```yaml
title: DAAM Hidden .svn .DS_Store JavaScript Loaded By Node
id: 3c42e6f1-DAAM-0003
status: experimental
description: Detects Node or shell tooling opening a suspicious hidden payload path used by DAAM.
author: dev-trap-dossiers incident 2026-08-12
logsource:
  category: file_event
  product: any
detection:
  selection_path:
    TargetFilename|contains: 'backend/src/config/.svn/.DS_Store'
  selection_proc:
    Image|endswith:
      - '/node'
      - '\\node.exe'
  condition: selection_path
falsepositives:
  - Legitimate macOS metadata files named .DS_Store exist, but JavaScript/backend access to .svn/.DS_Store in a source tree is highly suspicious.
level: high
tags:
  - attack.defense-evasion
  - attack.execution
```

---

## 3. Grep / one-liner detections

Use these before running any unsolicited candidate repository.

```sh
# DAAM-specific high-signal source indicators.
grep -RIn --color=never -E \
  'iploglab\.store|/api/terminal/bootstrap\?os=|/api/terminal/windows\?flag=7|\.svn/\.DS_Store|216\.126\.225\.243:(8085|8086|8087)|wallet-modal-223|daam602/ai-agent-marketplace|createRequire\(|!\.DS_Store' \
  --exclude-dir=node_modules .
```

```sh
# Hidden/system-looking JavaScript payloads in source trees.
find . -path '*/node_modules' -prune -o \
  \( -path '*/.svn/.DS_Store' -o -name '.DS_Store' \) -type f -print
```

```sh
# Git hooks shipped inside an archive; inspect with a pager, do not execute Git operations.
find .git/hooks -type f -maxdepth 1 -print 2>/dev/null
```

A hit on `iploglab.store`, `.svn/.DS_Store`, `216.126.225.243`, or the Bitbucket installer paths warrants treating the repository as malicious until proven otherwise.

---

## 4. Blocklist / alert suggestions

- Block/alert on DNS or HTTP/S to `iploglab.store`.
- Block/alert on outbound traffic to `216.126.225.243` ports `8085`, `8086`, and `8087`.
- Alert on developer workstations executing shell/PowerShell command lines containing `/api/terminal/bootstrap?os=` or `/api/terminal/windows?flag=7`.
- Alert on Node processes reading `backend/src/config/.svn/.DS_Store` or any `.svn/.DS_Store` in an application source tree.
- Alert on source repositories that ship non-sample executable Git hooks in `.git/hooks/` inside a distributed archive.

---

## 5. Maintenance notes

The exact domains and C2 IP may rotate. The more durable behavioral signals are:

- shipped Git hooks in unsolicited ZIPs;
- branch-switching instructions that naturally trigger hooks;
- hidden executable JavaScript under system-looking paths such as `.svn/.DS_Store`;
- ES-module projects using `createRequire()` to load hidden/system-looking files;
- runtime imports from mock-data modules into obfuscated payloads;
- README instructions to run remote installer scripts before an interview.
