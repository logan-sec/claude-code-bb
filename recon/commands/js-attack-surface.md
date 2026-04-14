---
name: js-attack-surface
description: Fetch all JS files for a target URL, analyze them for realistic attack vectors, and print confirmed attack surface findings to chat. Use when you want to map client-side attack surface from a target's JavaScript.
allowed-tools: Bash
---

# JS Attack Surface Analysis

Target: $ARGUMENTS

All recon commands run via SSH: `ssh user@<ip>`

---

## Step 1 — Collect JS File URLs

Use multiple sources to maximize coverage. Run all three in parallel.

```bash
ssh user@<ip> "echo '$ARGUMENTS' | waybackurls | grep '\.js' | grep -v '\.json' | sort -u"
ssh user@<ip> "echo '$ARGUMENTS' | gau --blacklist png,jpg,gif,svg,css,woff,woff2 | grep '\.js' | sort -u"
ssh user@<ip> "katana -u '$ARGUMENTS' -jc -d 3 -silent | grep '\.js' | sort -u"
```

Deduplicate and merge results. Filter out known CDN/third-party domains (e.g., `cdn.`, `googleapis.com`, `cloudflare.com`, `jquery`, `bootstrap`) — focus only on first-party and same-origin JS.

**If zero first-party JS files are found: stop. Note "no first-party JS discovered" and exit.**

---

## Step 2 — Download JS Files

Download each first-party JS file to a working directory on Kali.

```bash
ssh user@<ip> "mkdir -p ~/bugbounty/js-analysis/[target-slug] && cd ~/bugbounty/js-analysis/[target-slug] && cat urls.txt | xargs -I{} sh -c 'curl -s -o \"\$(echo {} | md5sum | cut -d\" \" -f1).js\" \"{}\"'"
```

For each file, also note the source URL — you'll need it to assess same-origin context later.

---

## Step 3 — Extract Signals

Run the following extractions against all downloaded JS. These are the only signal categories that produce findings worth reporting.

### 3a — Endpoints and API Routes

```bash
ssh user@<ip> "grep -rhoP '(\/api\/[a-zA-Z0-9_\-\/\.\?=&%{}:]+)' ~/bugbounty/js-analysis/[target-slug]/ | sort -u"
ssh user@<ip> "grep -rhoP '(https?:\/\/[a-zA-Z0-9_\-\.]+\/[a-zA-Z0-9_\-\/\.\?=&%{}:]+)' ~/bugbounty/js-analysis/[target-slug]/ | sort -u"
```

Flag: any endpoints with path parameters like `/user/{id}`, `/account/{uuid}`, `/v1/resource/{id}` — these are IDOR candidates.

### 3b — Authentication and Authorization Patterns

```bash
ssh user@<ip> "grep -rniP '(authorization|bearer|x-api-key|x-auth-token|access_token|id_token|session|jwt)' ~/bugbounty/js-analysis/[target-slug]/ | grep -v '\.map:' | head -100"
```

Flag: tokens passed as URL params, tokens stored in localStorage, any hint of client-side role checks.

### 3c — DOM Sinks (XSS Surface)

```bash
ssh user@<ip> "grep -rnoP '(innerHTML|outerHTML|__html|document\.write|eval\(|setTimeout\(|setInterval\()' ~/bugbounty/js-analysis/[target-slug]/ | grep -v 'node_modules'"
```

Tier 1 sinks only: `innerHTML`, `outerHTML`, `__html`, `document.write`. Tier 2: `eval`, `setTimeout`, `setInterval`.

For each hit, extract 5 lines of surrounding context to see if user-controlled input reaches the sink.

### 3d — GraphQL

```bash
ssh user@<ip> "grep -rnoP '(query\s+\w+|mutation\s+\w+|gql`[^`]+`)' ~/bugbounty/js-analysis/[target-slug]/ | head -100"
```

Flag: any mutations, any queries that accept an ID or filter parameter without obvious auth enforcement in the JS.

### 3e — Hardcoded Secrets and Keys

```bash
ssh user@<ip> "grep -rnoP '([a-zA-Z0-9_\-]{20,})(key|secret|token|password|pass|pwd|api)([a-zA-Z0-9_\-]{0,20})\s*[:=]\s*[\"'\''][a-zA-Z0-9_\-\.\/+]{8,}[\"'\'']' ~/bugbounty/js-analysis/[target-slug]/ -i | grep -v 'example\|test\|placeholder\|xxxx\|dummy\|fake\|sample'"
ssh user@<ip> "grep -rnoP '(AIza[0-9A-Za-z\-_]{35}|AKIA[0-9A-Z]{16}|sk-[a-zA-Z0-9]{32,})' ~/bugbounty/js-analysis/[target-slug]/"
```

### 3f — SSRF / Open Redirect Candidates

```bash
ssh user@<ip> "grep -rnoP '(url=|redirect=|next=|return=|returnUrl=|callback=|dest=|target=|redir=)' ~/bugbounty/js-analysis/[target-slug]/ | head -50"
```

Flag: any parameter that accepts a full URL or path that is passed directly to a fetch/XHR call or used in a `window.location` assignment.

### 3g — postMessage Handlers

```bash
ssh user@<ip> "grep -rnoP 'addEventListener\([\"'\'']message[\"'\'']' ~/bugbounty/js-analysis/[target-slug]/"
```

For each hit, extract the full handler function. Flag if: `event.origin` is not checked, OR `event.data` reaches a DOM sink or triggers a state change.

---

## Step 4 — Triage Each Signal

For every flagged item from Step 3, apply this filter before adding it to the output:

**Ask for each signal:**
1. Is there a plausible path from attacker-controlled input to this behavior? (Not just "the code exists" — is there a realistic trigger?)
2. What does exploiting this actually yield? (Data exfil, account takeover, SSRF, XSS with session access, etc.)
3. Is this gated behind authentication in a way that would make it reportable only as an authenticated finding — and if so, is that still in scope?

**Drop it if:**
- The sink exists but there is no observable user-controlled data path to it
- The endpoint exists but all parameters are non-enumerable or server-side validated with no bypass path in JS
- The secret looks like a public API key with no meaningful scope (e.g., Google Maps embed key)
- You can't articulate impact beyond "information disclosure of a non-sensitive value"

---

## Step 5 — Output Attack Vectors

Print findings directly to chat. Do not write a file. Do not report — this is a lead list, not a submission.

For each surviving vector, use this format:

---

**Vector: [short name]**
**Class:** [IDOR / XSS / SSRF / Auth bypass / Secret / Open redirect / postMessage / GraphQL / Other]
**Source file:** [filename or URL]
**Evidence:** [exact code snippet or grep line — keep it short]
**Trigger path:** [how attacker-controlled input reaches this — be specific, not hypothetical]
**Impact:** [what an attacker gains if this works]
**Next step:** [the single most valuable manual test to confirm or kill this lead]

---

Only include vectors that pass Step 4 triage. If nothing passes, say so explicitly: "No viable attack vectors identified in first-party JS."

---

## Hard Rules

- **No long-shots.** If you can't write a concrete "Trigger path," drop the vector.
- **No secret-scanner noise.** Only flag credentials that are scoped, active-looking, and would grant meaningful access.
- **No sink-without-source.** A DOM sink existing is not a finding. User-controlled data reaching it is.
- **No "this might be worth checking."** Either the signal passes triage or it doesn't. Vague leads waste hunting time.
- **Do not submit from this output.** Every vector here requires manual validation in Caido/Burp before it becomes a Finding. Run `/triager` before any submission.
