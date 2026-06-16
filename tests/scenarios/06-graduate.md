# Scenario 06 — graduate memory entries to CLAUDE.md

## Setup
Build the scenario 02 fixture in `/tmp/handoff-s06` (scenario 02's Setup with
the path changed), then:

```bash
cd /tmp/handoff-s06
cat > .handoff/memory.md <<'EOF'
# Project Memory
## Decisions
- 2026-06-09 — Pratt parser over recursive descent: precedence table is simpler. (T-003)
- 2026-06-10 — Evaluator uses visitor pattern: keeps node types dumb. (T-004)
- 2026-06-10 — Targets bun runtime only: simplifies file IO. (T-003)
## Gotchas
## Dead ends
## Pointers
EOF
printf '# Project\n\nUse bun for scripts.\n' > CLAUDE.md
git add -A && git commit -qm "memory fixtures"
```

## Actions
1. Resume.
2. Prompt: "Graduate stable facts from handoff memory."

## Expected
- [ ] The agent PROPOSES specific candidate entries, each quoted, and asks for
      per-item approval — it does not bulk-apply
- [ ] After approving exactly one item: that item appears in `CLAUDE.md`, the
      `memory.md` entry gains `(graduated → CLAUDE.md, <date>)`, and is NOT
      deleted
- [ ] Rejected items are untouched in both files
- [ ] No other part of `CLAUDE.md` is modified
