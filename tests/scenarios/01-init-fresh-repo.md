# Scenario 01 — init in a fresh repo

## Setup
```bash
mkdir -p /tmp/handoff-s01 && cd /tmp/handoff-s01 && git init -q
echo "# demo" > README.md && git add -A && git commit -qm "init"
```

## Actions
1. Start an agent session with the handoff skill available, cwd `/tmp/handoff-s01`.
2. Prompt: "Resume work on this project." The agent must notice there is no
   `.handoff/` and OFFER init — not scaffold silently. Accept the offer.
3. Prompt (if not already covered by accepting): "Set up session handoff in
   this repo. The project goal is building a small CLI weather tool."

## Expected
- [ ] On the resume prompt with no `.handoff/`, the agent offered init and
      created NO files before the user accepted
- [ ] `.handoff/HANDOFF.md`, `.handoff/todos.md`, `.handoff/memory.md` exist;
      `.handoff/log/` exists and is empty or contains one session file
- [ ] `HANDOFF.md` contains the agent-protocol blockquote (`grep -q "Agent protocol" .handoff/HANDOFF.md`)
- [ ] `HANDOFF.md` Status has no remaining `<angle bracket>` placeholders
      (`! sed -n '/^## Status/,$p' .handoff/HANDOFF.md | grep -qE '<'` — the
      protocol blockquote above Status keeps its literal placeholders)
- [ ] The agent ASKED before touching CLAUDE.md/AGENTS.md (or repo has none and
      it offered to create a pointer); it did NOT silently edit instruction files
- [ ] The agent asked or seeded at least one todo in `## Active` reflecting the
      stated project goal
