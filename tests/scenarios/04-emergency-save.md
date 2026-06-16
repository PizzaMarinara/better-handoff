# Scenario 04 — emergency save (out of tokens)

## Setup
Scenario 02 state, mid-session: agent has resumed and made progress on T-004.

## Actions
1. Resume as in scenario 02; let the agent work briefly on T-004.
2. Prompt: "I'm almost out of tokens — handoff NOW."

## Expected
- [ ] The agent completes the save in a SINGLE response (no clarifying
      questions, no multi-turn reconciliation)
- [ ] `todos.md` statuses updated to reflect reality
- [ ] Session log gains a short `## Handoff summary` (2–3 lines acceptable)
- [ ] `HANDOFF.md` Status section updated (full rewrite NOT required)
- [ ] A git commit exists touching ONLY `.handoff/` files, made WITHOUT asking
      (`git log -1 --format=%H -- .handoff/ | xargs git show --stat` lists
      only `.handoff/` paths; message matches conventional format, e.g.
      `chore(handoff): ...`)
- [ ] The agent did NOT push (and at most proposed pushing)
- [ ] No compaction or memory distillation was attempted
