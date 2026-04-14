# BugBounty_Recon - Claude Code Configuration

## Environment
All commands run via SSH on Kali Linux at user@<ip>
Tools are located at /home/user/go/bin/ and must be called with full paths

## Caido
URL: http://127.0.0.1:8080
PAT auth is set via environment variable CAIDO_PAT.
When using caido-mode, always filter to in-scope target traffic only.

## Session Start Protocol
Before doing anything else, run:
ssh user@<ip> "ls ~/bugbounty_real/"

Then ask the user which target they want to work on.

Once a target is selected, run:
ssh user@<ip> "find ~/bugbounty_real/[selected_target] -type f | sort && echo '===' && cat ~/bugbounty_real/[selected_target]/00-summary.md 2>/dev/null || echo 'No summary yet'"

Use the output to brief yourself on current state before taking any action.

## Rules
- Recon and mapping only. Do not exploit anything.
- Do not submit reports or create findings without explicit instruction.
- All output saves to ~/bugbounty_real/[target]/recon/ on Kali.
- When returning to a target, read existing notes before running any tools — avoid duplicating work already done.
- If multiple targets exist and none is specified, list them and ask. Do not assume or default to the most recent.
