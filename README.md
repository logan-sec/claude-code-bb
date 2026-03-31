# claude-code-bb

CLAUDE.md configurations and custom skills I use for bug bounty hunting 
with Claude Code. Built and documented as part of the LoganSec bug bounty journey.

## What's in here

| File | What it does |
|------|-------------|
| `CLAUDE.md` | Main config — primes Claude Code to think like a bug bounty hunter from session start |
| `skills/triager.md` | Pre-submission critique tool — simulates a real Bugcrowd triager, flags N/A patterns, and protects account reputation before anything goes out |
| `skills/report-draft.md` | Formats findings into clean, submission-ready reports |
| `skills/hypothesis.md` | Generates attack hypotheses for a given target surface |

## How to use

Drop `CLAUDE.md` into the root of whatever directory you're running Claude Code from. 
Skills go in a `/skills` folder in the same directory.

Adapt everything. These are my configs — your hunting style will be different.

## More

📺 [LoganSec](https://www.youtube.com/@Logan-sec)  
𝕏 [@LoganOpSec](https://x.com/LoganOpSec)  
🐛 One hunter's bug bounty journey; documented as it happens.
