---
name: new-target
description: Run a full first-time recon on a new bug bounty target program. Use when starting on a brand new program for the first time. Reads scope from ~/bugbounty_real/[target]/scope.md on Kali.
allowed-tools: Bash
---

# New Target Recon

Program: $ARGUMENTS

All commands run via SSH: `ssh user@<ip>`
Tools at `/home/user/go/bin/` — use full paths.
Save all output to `~/bugbounty_real/$ARGUMENTS/recon/` on Kali.

**Do not attempt to exploit anything. Recon and mapping only.**

---

## Pre-flight

Verify scope file exists and tools are available:

```bash
ssh user@<ip> "cat ~/bugbounty_real/$ARGUMENTS/scope.md"
```

```bash
ssh user@<ip> "which /home/user/go/bin/subfinder /home/user/go/bin/httpx /home/user/go/bin/gau /home/user/go/bin/waybackurls /home/user/go/bin/katana 2>&1"
```

If `scope.md` is missing or empty — stop and tell the user to create it before continuing.
If any tool is missing — stop and notify. Do not proceed.

Create output directory:

```bash
ssh user@<ip> "mkdir -p ~/bugbounty_real/$ARGUMENTS/recon"
```

---

## Scope Parsing

Read `scope.md` and split into two lists:

- **Root domains** — lines with no wildcard (e.g. `example.com`)
- **Wildcard domains** — lines starting with `*.` (e.g. `*.example.com`) — strip the `*.` to get the root for enumeration

Combine both into a single deduplicated root domain list and save it:

```bash
ssh user@<ip> "grep -v '^\s*$' ~/bugbounty_real/$ARGUMENTS/scope.md | sed 's/^\*\.//' | sort -u > ~/bugbounty_real/$ARGUMENTS/recon/scope-roots.txt && cat ~/bugbounty_real/$ARGUMENTS/recon/scope-roots.txt"
```

All recon phases run against every root in `scope-roots.txt`.

---

## Phase 1 — Subdomain Enumeration (15 min)

Run subfinder, gau, and waybackurls against every root domain in scope:

```bash
ssh user@<ip> "while read domain; do
  /home/user/go/bin/subfinder -d \$domain -silent
  echo \$domain | /home/user/go/bin/gau --subs 2>/dev/null | grep -oP 'https?://\K[^/]+'
  echo \$domain | /home/user/go/bin/waybackurls 2>/dev/null | grep -oP 'https?://\K[^/]+'
done < ~/bugbounty_real/$ARGUMENTS/recon/scope-roots.txt | sort -u > ~/bugbounty_real/$ARGUMENTS/recon/01-subdomains.txt && wc -l ~/bugbounty_real/$ARGUMENTS/recon/01-subdomains.txt"
```

---

## Phase 2 — Live Host Discovery (10 min)

```bash
ssh user@<ip> "cat ~/bugbounty_real/$ARGUMENTS/recon/01-subdomains.txt | /home/user/go/bin/httpx -silent -status-code -title -tech-detect -o ~/bugbounty_real/$ARGUMENTS/recon/02-live-hosts.txt && wc -l ~/bugbounty_real/$ARGUMENTS/recon/02-live-hosts.txt"
```

Flag any hosts returning 200 with interesting titles. Note anything that looks like admin panels, APIs, or internal tooling.

---

## Phase 3 — Tech Stack Fingerprinting (10 min)

Extract technology signals from httpx output:

```bash
ssh user@<ip> "grep -oP '\[[^\]]+\]' ~/bugbounty_real/$ARGUMENTS/recon/02-live-hosts.txt | sort | uniq -c | sort -rn | head -40"
```

Analyze for:
- Server, framework, CDN, WAF indicators
- Version numbers in headers or response bodies
- Frontend framework signals from JS bundle names

Save all identified technologies and versions to `03-tech-stack.md`. Every version number feeds Phase 6.

---

## Phase 4 — Endpoint Mapping (15 min)

Run katana against all live hosts, and gau/waybackurls against all scope roots:

```bash
ssh user@<ip> "awk '{print \$1}' ~/bugbounty_real/$ARGUMENTS/recon/02-live-hosts.txt | /home/user/go/bin/katana -silent -d 3 -o ~/bugbounty_real/$ARGUMENTS/recon/katana.txt 2>/dev/null"
```

```bash
ssh user@<ip> "while read domain; do
  echo \$domain | /home/user/go/bin/gau --blacklist png,jpg,gif,svg,css,woff,woff2 2>/dev/null
  echo \$domain | /home/user/go/bin/waybackurls 2>/dev/null
done < ~/bugbounty_real/$ARGUMENTS/recon/scope-roots.txt | sort -u > ~/bugbounty_real/$ARGUMENTS/recon/07-historical-urls.txt"
```

Combine and filter for interesting patterns:

```bash
ssh user@<ip> "cat ~/bugbounty_real/$ARGUMENTS/recon/katana.txt ~/bugbounty_real/$ARGUMENTS/recon/07-historical-urls.txt | sort -u | grep -iE '(api|admin|auth|token|key|secret|internal|debug|v1|v2|graphql|swagger|config|reset|password|upload|export|download|backup|\.json|\.xml|\.env)' > ~/bugbounty_real/$ARGUMENTS/recon/04-endpoints.txt && wc -l ~/bugbounty_real/$ARGUMENTS/recon/04-endpoints.txt"
```

---

## Phase 5 — JS Recon (10 min)

Extract first-party JS file URLs:

```bash
ssh user@<ip> "cat ~/bugbounty_real/$ARGUMENTS/recon/07-historical-urls.txt ~/bugbounty_real/$ARGUMENTS/recon/katana.txt | grep '\.js' | grep -v '\.json' | grep -v -iE '(jquery|bootstrap|cdn\.|cloudflare|googleapis|facebook|twitter|analytics)' | sort -u > ~/bugbounty_real/$ARGUMENTS/recon/js-files.txt && wc -l ~/bugbounty_real/$ARGUMENTS/recon/js-files.txt"
```

Run LinkFinder and SecretFinder on first-party JS:

```bash
ssh user@<ip> "head -30 ~/bugbounty_real/$ARGUMENTS/recon/js-files.txt | while read url; do
  echo \"=== \$url ===\"
  python3 ~/tools/LinkFinder/linkfinder.py -i \$url -o cli 2>/dev/null
done | sort -u > ~/bugbounty_real/$ARGUMENTS/recon/js-endpoints.txt"
```

```bash
ssh user@<ip> "head -30 ~/bugbounty_real/$ARGUMENTS/recon/js-files.txt | while read url; do
  echo \"=== \$url ===\"
  python3 ~/tools/SecretFinder/SecretFinder.py -i \$url -o cli 2>/dev/null
done > ~/bugbounty_real/$ARGUMENTS/recon/05-js-secrets.md"
```

If LinkFinder or SecretFinder are not at `~/tools/`, skip and note the path needs updating.

---

## Phase 6 — CVE Lookup (5 min)

Run targeted nuclei CVE templates against live hosts — critical and high only:

```bash
ssh user@<ip> "awk '{print \$1}' ~/bugbounty_real/$ARGUMENTS/recon/02-live-hosts.txt | /home/user/go/bin/nuclei -t ~/nuclei-templates/cves/ -severity critical,high -silent -o ~/bugbounty_real/$ARGUMENTS/recon/08-cves.md 2>/dev/null"
```

Do not run full nuclei scans. CVE templates only.

---

## Phase 7 — Summary (5 min)

Write `~/bugbounty_real/$ARGUMENTS/recon/00-summary.md`:

```
# Recon Summary — $ARGUMENTS
Date: [date]

## Scope
[list all in-scope roots from scope.md]

## Numbers
- Root domains in scope: [count]
- Subdomains discovered: [count]
- Live hosts: [count]
- Endpoints mapped: [count]
- JS files analyzed: [count]

## Tech Stack
[identified technologies and versions]

## Top Endpoints for Manual Testing
[5-10 most interesting, with reason for each]

## Secrets / Keys Found
[any hardcoded credentials, tokens, API keys — with source file]

## CVEs
[any relevant CVEs for identified versions]

## Suggested Starting Points
[specific hypotheses for BugBounty_Hacking — IDOR surfaces, auth endpoints, etc.]

## Notes
[anything unusual, out of scope items encountered, rate limiting, etc.]
```

---

## Hard Rules

- Do not brute force anything
- Do not send more than normal recon-level traffic
- Do not run nuclei full scans — targeted CVE templates, critical/high only
- If you find what looks like a confirmed vulnerability, note it in the summary and stop — do not investigate further. That belongs in BugBounty_Hacking
- Only test hosts that match the original scope in scope.md — do not follow links off-scope
- Use full paths for all tools: `/home/user/go/bin/`
