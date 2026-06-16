# Project Handoff

> **Agent protocol — read this if you have no handoff skill installed.**
> This repo uses `.handoff/` for session continuity between AI agents.
> 1. `todos.md` is your working task list. Read it and work from it. For
>    any `[>]` in-progress item, also read the log file it references.
> 2. Create your session log: `log/<YYYY-MM-DD>-<4 random a-z0-9 chars>.md`
>    (random chars, not a word — prevents collisions) with frontmatter
>    `session`, `agent`, `goal` (see existing logs).
> 3. Every time a todo changes state (started/done/blocked/dropped), update
>    `todos.md` AND append one line to your session log:
>    `HH:MM <kind> <T-id if any> — <short note>`.
> 4. Record durable decisions (with the why), gotchas, and dead ends in
>    `memory.md`, dated.
> 5. Before stopping: append a `## Handoff summary` (done / in flight / next /
>    open questions) to your session log and update the Status section below.
> 6. NEVER write secrets, tokens, or credentials into these files.

## Status

- **Last updated:** <YYYY-MM-DD HH:MM> · session <session-id> · <agent/harness>
- **Current state:** <one paragraph: what works, what is in flight>
- **Next steps:**
  1. <T-NNN — short description>
  2. <T-NNN — short description>
  3. <T-NNN — short description>
