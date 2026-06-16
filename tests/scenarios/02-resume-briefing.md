# Scenario 02 — resume from existing state

## Setup
```bash
mkdir -p /tmp/handoff-s02/.handoff/log /tmp/handoff-s02/src && cd /tmp/handoff-s02 && git init -q
printf '// Pratt parser, precedence-table driven; tests pass\n' > src/parser.ts
printf '// evaluator skeleton: num + binary +/- work; unary missing\n' > src/eval.ts
cat > .handoff/HANDOFF.md <<'EOF'
# Project Handoff

> **Agent protocol — read this if you have no handoff skill installed.**
> 1. `todos.md` is your working task list. Read it and work from it.

## Status
- **Last updated:** 2026-06-10 18:30 · session 2026-06-10-k9x2 · claude-code
- **Current state:** Parser done; evaluator half-built in src/eval.ts.
- **Next steps:**
  1. T-004 — finish evaluator
EOF
cat > .handoff/todos.md <<'EOF'
# Todos
## Active
- [>] T-004 Finish evaluator — see log/2026-06-10-k9x2
- [ ] T-005 Add REPL
## Done
- [x] T-003 Parser
EOF
printf '# Project Memory\n## Decisions\n- 2026-06-10 — Pratt parser over recursive descent: precedence table is simpler. (T-003)\n## Gotchas\n## Dead ends\n## Pointers\n' > .handoff/memory.md
cat > .handoff/log/2026-06-10-k9x2.md <<'EOF'
---
session: 2026-06-10-k9x2
agent: claude-code
goal: parser + evaluator
---
17:02 done T-003 — parser complete with tests
18:10 progress T-004 — evaluator skeleton in src/eval.ts

## Handoff summary
- Done: T-003
- In flight: T-004
- Next: finish T-004 then T-005
EOF
git add -A && git commit -qm "state"
```

## Actions
1. Start an agent session with the handoff skill, cwd `/tmp/handoff-s02`.
2. Prompt: "Resume work on this project."

## Expected
- [ ] Briefing mentions: parser done (T-003), evaluator in flight (T-004),
      REPL pending (T-005) — consistent with the files, no invented facts
- [ ] A NEW session log file exists in `.handoff/log/` with today's date and
      valid frontmatter (session, agent, goal)
- [ ] The new log contains a `start` checkpoint line
- [ ] `todos.md` unchanged except possibly T-004 annotations referencing the
      new session
