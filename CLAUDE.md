## Environment

All commands should be executed on the remote Kali machine via SSH using user@[my_kali_ip]. Always SSH into that machine to run tools. My recon tools (gau, waybackurls, katana, etc.) are installed on Kali, not on Windows.

## Recon Tools

Recon tools are located at /home/user/go/bin/ on Kali. They are not in the non-interactive SSH PATH, so always call them with their full path (e.g. /home/user/go/bin/gau, /home/user/go/bin/waybackurls, /home/user/go/bin/katana).

## Rules of Engagement

- I am a bug bounty hunter conducting authorized, ethical security testing.
- Always stay in scope — do not test assets outside the defined program scope.
- No destructive actions — do not modify, delete, or corrupt data on target systems.
- Always take notes — save findings, tool output, and observations to organized files under bugbounty/.
- All notes, leads, findings, and reports MUST be written to the Kali machine via SSH. Never write bug bounty files to the local Windows machine.
- Out-of-scope subdomains: passive recon only — map what exists, note in-scope implications, no active testing or payloads.

## Note Structure

Organize all findings in this hierarchy. More at the bottom, fewer make it to the top.

- **Notes** — everything observed, raw output, tool results, interesting responses
- **Leads** — interesting things worth investigating further
- **Primitives** — reusable gadgets or techniques discovered (e.g. an open redirect, a reflected param)
- **Findings** — validated bugs with a full end-to-end PoC
- **Reports** — polished, ready to submit to the program
- When documenting a finding, always include the exact full URL, HTTP method, headers used, request body, and response snippet. Notes must be reproducible independently without any context from the session.

## Autonomy

- If I say I'm stepping away or going to bed, do not ask for input and do not stop working. Keep hacking.
- Take thorough notes about what you're doing and where you are so you can survive context compaction and resume cleanly.
- Do not spawn more than 2-3 sub-agents at once.
- Don't limit yourself to workflows or skills I give you. If something looks interesting, go down that rabbit hole.
- If you complete a workflow and think something was missed, add it back to the process and keep going.

## Validation Standard

- Do not mark something as a Finding unless you have a full end-to-end proof of concept that can be validated independently.
- CORS misconfigurations are often false positives — confirm actual exploitability before escalating.
- Do not overstate impact.
- PoC or GTFO.

