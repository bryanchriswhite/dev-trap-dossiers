# Detection rules

Rules for detecting the `realfraction/realfraction` loader and its broader generation across three observation surfaces: source code (YARA on repository contents), system telemetry (Sigma on process / DNS / proxy events), and ad-hoc analyst use (grep one-liners).

These are *additive* to — not replacements for — the AjunaVerse-family rules in [`../2026-05-13-ajunaverse-mvp/detection-rules.md`](../2026-05-13-ajunaverse-mvp/detection-rules.md). The two rule sets target different generations of the same broader Contagious Interview cluster. Run both; a hit on either is high-signal.

The idioms targeted below are essentially zero-false-positive in ordinary code, so a single hit is high-signal.

---

## 1. YARA rules — for scanning repositories and source trees

```yara
rule RealFraction_NodeJS_RegionChecker_Loader
{
    meta:
        description = "realfraction-family Node loader: top-level https.request with magic header 'x-secret-header: secret' followed by eval() of the response body, with 'blocked'/{blocked:true} negative-gate sentinels"
        author      = "incident 2026-05-18"
        date        = "2026-05-18"
        severity    = "critical"
        reference   = "https://github.com/realfraction/realfraction"

    strings:
        $magic_header_kv      = /['"]x-secret-header['"]\s*:\s*['"]secret['"]/
        $magic_header         = "x-secret-header"
        $eval_data            = /\beval\s*\(\s*(data|response\.data)\s*\)/
        $func_ctor            = /new\s+Function\.constructor\s*\(\s*['"]require['"]\s*,/
        $neg_gate_str         = /data\s*===\s*['"]blocked['"]/
        $neg_gate_json        = /JSON\.parse\s*\(\s*data\s*\)\s*\?\.\s*blocked/
        $c2_ipregionchecker   = "ipregionchecker.com"
        $c2_isillegalregion   = "isillegalregion.com"
        $c2_cookie_xi_seven   = "cookie-xi-seven.vercel.app"
        $c2_ip_check_api      = "ip-check-api.vercel.app"
        $c2_path_dashed       = "/api/ip-check-encrypted/"
        $c2_path_undashed     = "/api/ipcheck-encrypted/"
        $api_key_g            = "6KDisdfjlskjDI837KJH4"   /* sub-shape G API key */
        $api_key_h            = "3948uf2uhe9r298rh2"      /* sub-shape H API key */

    condition:
        // High-confidence: magic header AND any RCE primitive in the same file.
        // Or: either of the two negative-gate sentinels paired with eval/Function.constructor.
        // Or: any known C2 host, path, or shared API-key token appears anywhere.
        ($magic_header_kv and ($eval_data or $func_ctor)) or
        ($magic_header    and ($eval_data or $func_ctor)) or
        ($neg_gate_str    and ($eval_data or $func_ctor)) or
        ($neg_gate_json   and ($eval_data or $func_ctor)) or
        any of ($c2_*) or
        any of ($api_key_*)
}

rule RealFraction_NodeJS_ConstantsTemplate_SubShapeG
{
    meta:
        description = "realfraction-family sub-shape G constants/index.js template — distinctive multi-line block defining API_HOST + API_SUB_URL + SAMPLE_API_KEY + API_HEADERS + API_URL together. Byte-identical across multiple compromised-victim repos (fabiolin/schoolmgmt, Paulooo0/go-test, KagiyamaWeb/PyPDFMicroservise, Wilovy09/deby-assignment, pablodiaz2799/solice-skill-test, ...). Single hit warrants treating the containing repo as untrusted."
        author      = "incident 2026-05-18 cluster-expansion sweep"
        date        = "2026-05-18"
        severity    = "critical"

    strings:
        $api_host    = /API_HOST\s*=\s*['"]cookie-xi-seven\.vercel\.app['"]/
        $api_sub_url = /API_SUB_URL\s*=\s*['"]ipcheck-encrypted['"]/
        $api_key     = "6KDisdfjlskjDI837KJH4"
        $api_headers = /API_HEADERS\s*=\s*\{\s*['"]x-secret-header['"]\s*:\s*['"]secret['"]/
        $api_url     = /API_URL\s*=\s*`https:\/\/\$\{API_HOST\}/

    condition:
        // The presence of any three of these in the same file is conclusive.
        3 of them
}

rule RealFraction_NodeJS_SideEffectImport_Suspicious_UtilName
{
    meta:
        description = "userController.js (or similar) side-effect-imports a utility module by a name that suggests a guard/region/feature-gate. Loose heuristic; high false-positive on its own. Pair with RealFraction_NodeJS_RegionChecker_Loader."
        author      = "incident 2026-05-18"
        date        = "2026-05-18"
        severity    = "informational"

    strings:
        $imp_regioncheck = /const\s+\w+\s*=\s*require\s*\(\s*['"][.\/\w-]*regionChecker['"][^)]*\)\s*;?/
        $imp_geocheck    = /const\s+\w+\s*=\s*require\s*\(\s*['"][.\/\w-]*geoCheck['"][^)]*\)\s*;?/
        $imp_featflag    = /const\s+\w+\s*=\s*require\s*\(\s*['"][.\/\w-]*featureFlags?['"][^)]*\)\s*;?/

    condition:
        any of them
}
```

**Usage** (scan a freshly-cloned repo, recursively):

```
yara -r detection-rules.yar /path/to/cloned-repo
```

A hit on `RealFraction_NodeJS_RegionChecker_Loader` warrants treating the repo as untrusted until reviewed. A hit on `RealFraction_NodeJS_SideEffectImport_Suspicious_UtilName` alone is informational; pair it with a manual inspection of the imported file's top-level code.

---

## 2. Sigma rules — for endpoint and network telemetry

These work against DNS query logs and HTTP-proxy logs.

### 2.1 DNS query for any realfraction-family C2 host

```yaml
title: Campaign DNS query for any realfraction-family C2 host (ipregionchecker.com / isillegalregion.com / cookie-xi-seven.vercel.app / ip-check-api.vercel.app)
id: 4a8c9c9f-1a8b-4d4a-8b3a-realfr01
status: experimental
description: DNS resolution of a developer host for any known realfraction-family loader C2 host. Updated 2026-05-18 to add three sibling C2 hosts (isillegalregion.com — live; cookie-xi-seven.vercel.app and ip-check-api.vercel.app — both DEPLOYMENT_DISABLED by Vercel as of 2026-05-18). Hits on the disabled hosts indicate a victim host still running an older loader.
references:
  - https://github.com/realfraction/realfraction
  - https://github.com/bryanchriswhite/dev-trap-dossiers/tree/main/incidents/2026-05-18-realfraction
author: incident 2026-05-18 cluster-expansion sweep
date: 2026-05-18
logsource:
  category: dns
  product: any
detection:
  selection:
    QueryName|re: '(?i)^(www\.)?(ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app)$'
  condition: selection
falsepositives:
  - Legitimate use of an unrelated service of the same name (none observed).
level: high
tags:
  - attack.initial-access
  - attack.t1566.003
```

### 2.2 Node process making an HTTP POST with the realfraction-family magic header

```yaml
title: Node process POSTs with x-secret-header:secret to the realfraction-family C2
id: 4a8c9c9f-1a8b-4d4a-8b3a-realfr02
status: experimental
description: A node process POSTs with header x-secret-header:secret. Diagnostic of the realfraction-family loader making its first beacon to the C2.
references:
  - https://github.com/realfraction/realfraction
author: incident 2026-05-18
date: 2026-05-18
logsource:
  category: proxy
  product: any
detection:
  selection_method:
    Method: 'POST'
  selection_header:
    Headers|contains: 'x-secret-header: secret'
  selection_host_apex:
    Host|re: '(?i)(ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app)$'
  selection_path:
    URI|re: '/api/ip-?check-?encrypted/'
  condition:
    selection_method and (selection_header or selection_host_apex or selection_path)
falsepositives:
  - None known. `x-secret-header: secret` is essentially zero-false-positive on outbound HTTP from developer hosts.
level: critical
tags:
  - attack.command-and-control
  - attack.t1071.001
```

### 2.3 Negative-gate sentinel response from the C2 (sandbox / probe detection)

```yaml
title: Inbound response body 'blocked' or '{"blocked":true}' on a path matching the realfraction-family C2
id: 4a8c9c9f-1a8b-4d4a-8b3a-realfr03
status: experimental
description: A small response body whose literal content is 'blocked' or whose JSON contains blocked:true, returned on a path matching the realfraction-family C2. Indicates a host on the network was attempting to reach the C2 from a non-allowlisted IP (a sandbox or a researcher probe) — useful for spotting victim hosts already running the loader.
references:
  - https://github.com/realfraction/realfraction
author: incident 2026-05-18
date: 2026-05-18
logsource:
  category: proxy
  product: any
detection:
  selection_path:
    URI|re: '/api/ip-?check-?encrypted/'
  selection_body_literal:
    BodyContent: 'blocked'
  selection_body_json:
    BodyContent|contains: '"blocked":true'
  condition: selection_path and (selection_body_literal or selection_body_json)
falsepositives:
  - None known.
level: high
tags:
  - attack.command-and-control
```

---

## 3. Grep / one-liner detections — for analysts

Useful as a pre-`npm start` check on a freshly-cloned repo.

```sh
# Catches every known realfraction-family loader idiom (sub-shapes A-H).
# A single hit warrants treating the repo as untrusted.
grep -RIn --color=never -E \
  "x-secret-header['\"][[:space:]]*:[[:space:]]*['\"]secret['\"]|new Function\.constructor\(['\"]require['\"]|ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app|ipcheck-encrypted|ip-check-encrypted|6KDisdfjlskjDI837KJH4|3948uf2uhe9r298rh2" \
  --exclude-dir=node_modules --exclude-dir=.git .
```

Wider grep that catches **both** the AjunaVerse-family and the realfraction-family generations of the broader Contagious Interview cluster. Updated 2026-05-18 to add the sub-shape H `Function.constructor` RCE primitive, the three new realfraction-family C2 hosts (`isillegalregion.com`, `cookie-xi-seven.vercel.app`, `ip-check-api.vercel.app`), and the no-dash path variant (`ipcheck-encrypted`):

```sh
grep -RIn --color=never -E \
  'new Function(\.constructor)?\(["'\'']require["'\''],?|verify\(setApiKey|x-app-request|x-secret-header|"runOn"[[:space:]]*:[[:space:]]*"folderOpen"|ipregionchecker\.com|isillegalregion\.com|cookie-xi-seven\.vercel\.app|ip-check-api\.vercel\.app|ipcheck-encrypted|ip-check-encrypted' \
  --exclude-dir=node_modules --exclude-dir=.git .
```

Targeted variations:

```sh
# Eval-on-remote-response primitive only
grep -RIn -E 'eval\(\s*data\s*\)' --exclude-dir=node_modules --exclude-dir=.git server/ app/ backend/ 2>/dev/null

# Side-effect-only require of a utility (smell, not proof — pair with file inspection)
grep -RIn -E 'const\s+\w+\s*=\s*require\(["'\''][^"'\'']*utils/[^"'\'']*["'\'']\)\s*;?\s*$' \
  --exclude-dir=node_modules --exclude-dir=.git . \
  | while IFS=: read -r f l _; do
      var=$(sed -n "${l}p" "$f" | sed -E 's/^.*const[[:space:]]+([A-Za-z_$][A-Za-z0-9_$]*).*$/\1/')
      # If the variable name never appears elsewhere in the file, the require is side-effect-only.
      count=$(grep -c -E "\\b${var}\\b" "$f")
      [ "$count" -le 1 ] && echo "$f:$l  $var  (side-effect-only)"
    done

# C2 host / path
grep -RIn -E 'ipregionchecker\.com|/api/ip-check-encrypted/' --exclude-dir=node_modules --exclude-dir=.git .
```

---

## 4. Static-blocklist suggestions

For organizations with web proxies or DNS sinkholes:

- Block resolution / outbound to:
  - `www.ipregionchecker.com` and apex `ipregionchecker.com` (already on registrar `client hold` at Unstoppable Domains; blocklist anyway in case the operator regains control)
  - `www.isillegalregion.com` and apex `isillegalregion.com` (**live** as of 2026-05-18; serving stage-2 payload)
  - `cookie-xi-seven.vercel.app` (currently DEPLOYMENT_DISABLED by Vercel)
  - `ip-check-api.vercel.app` (currently DEPLOYMENT_DISABLED by Vercel)
  - `realfraction.xyz` and any `*.realfraction.xyz` (lure-brand domain backing the operator-owned GitHub org's contact email)
- Alert on outbound HTTP request headers containing `x-secret-header: secret` from developer workstations.
- Alert on outbound HTTP requests to any path matching `/api/ip-?check-?encrypted/` (the loader path; both dashed and undashed variants observed; the suffix is high-entropy and may rotate per instance).
- Alert on inbound HTTP response bodies whose first 7 bytes are `blocked` or which contain `"blocked":true` paired with a request path matching the loader path — these are the negative-gate sentinels and their appearance in logs indicates a host on your network was probing the C2 from a non-allowlisted IP. If the host is a developer workstation, it is almost certainly already running the loader.
- Alert on inbound HTTP responses with header `x-vercel-error: DEPLOYMENT_DISABLED` on a request path matching `/api/ip-?check-?encrypted/` — indicates a developer workstation is still running an older loader pointed at a Vercel deployment that has been taken down. The host should be triaged.

---

## 5. Notes on rule maintenance

- The C2 host (`ipregionchecker.com`) and path (`/api/ip-check-encrypted/3aeb34a31`) are stable in the current generation. Both are likely to rotate in future generations — the host name will probably remain in the same "infrastructure-sounding" family (`ipgeocheck.*`, `regionapi.*`, `feature-flags.*`), and the path will remain a high-entropy hex/alphanumeric suffix.
- The magic header `x-secret-header: secret` is the single most durable code-level fingerprint of this generation. It would need to change at the same time as the loader is rewritten; until then, treat it as zero-false-positive.
- The `eval(data)` primitive can be swapped to `new Function(...)(data)` or `vm.runInNewContext(data)` cheaply. A future generation might do so to evade the eval-specific rules. The Sigma rules (network and process behavior) are more durable than the YARA rules.
- The negative-gate sentinels (`blocked`, `{"blocked":true}`) are highly specific to *this* loader's response-side gate. A C2 redesign would change them.
- The side-effect-import shape (`const x = require(...)` with `x` unused) is a generic-enough smell that it will keep being useful even as the loader's contents change. Pair it with the YARA `RealFraction_NodeJS_SideEffectImport_Suspicious_UtilName` rule to surface candidate trigger sites in unfamiliar repos.
