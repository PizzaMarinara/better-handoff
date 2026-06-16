# Scenario 03 — recovery from an interrupted session

## Setup
Like scenario 02, but the latest log has checkpoints and NO `## Handoff
summary`, and the working tree has an uncommitted change:

```bash
mkdir -p /tmp/handoff-s03/.handoff/log /tmp/handoff-s03/src && cd /tmp/handoff-s03 && git init -q
printf '// Pratt parser, precedence-table driven; tests pass\n' > src/parser.ts
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
printf '# Project Memory\n## Decisions\n## Gotchas\n## Dead ends\n## Pointers\n' > .handoff/memory.md
cat > .handoff/log/2026-06-11-p4q1.md <<'EOF'
---
session: 2026-06-11-p4q1
agent: codex
goal: finish evaluator
---
09:12 start — resumed from T-004
09:55 progress T-004 — binary ops evaluate; unary ops still failing
EOF
echo "// half-finished unary ops" > src/eval.ts
# src/eval.ts is intentionally uncommitted — do NOT use `git add -A`
git add .handoff src/parser.ts && git commit -qm "state"
```

## Actions
1. Start an agent session with the handoff skill, cwd `/tmp/handoff-s03`.
2. Prompt: "Resume work on this project."

## Expected
- [ ] Briefing discloses that the previous session did not complete a handoff
      (interrupted/died mid-flight/work possibly unsaved — semantics matter,
      not the literal word)
- [ ] Briefing reconstructs state from checkpoint lines: binary ops done,
      unary ops failing
- [ ] Briefing mentions the uncommitted change in `src/eval.ts` (reality check
      via git status)
- [ ] The interrupted log is left intact (not edited); a new session log is
      created
