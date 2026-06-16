# Scenario 07 — merge of two divergent .handoff states

## Setup
Build the scenario 02 fixture in `/tmp/handoff-s07` (same files, committed on
`main`), then create two diverging session branches. (The `sed -i ''`
invocations are macOS/BSD syntax; on GNU sed drop the empty string: `sed -i`.)

```bash
cd /tmp/handoff-s07
git checkout -qb session-a
sed -i '' 's|- \[>\] T-004 Finish evaluator — see log/2026-06-10-k9x2|- [x] T-004 Finish evaluator|' .handoff/todos.md
printf -- '- [ ] T-006 Add error messages\n' >> .handoff/todos.md
printf -- '---\nsession: 2026-06-11-aa11\nagent: claude-code\ngoal: evaluator\n---\n10:00 done T-004 — evaluator complete\n\n## Handoff summary\n- Done: T-004\n' > .handoff/log/2026-06-11-aa11.md
sed -i '' 's|Parser done; evaluator half-built in src/eval.ts.|Evaluator complete; REPL next.|' .handoff/HANDOFF.md
git add -A && git commit -qm "session a"

git checkout -q main && git checkout -qb session-b
sed -i '' 's|- \[ \] T-005 Add REPL|- [>] T-005 Add REPL — see log/2026-06-11-bb22|' .handoff/todos.md
printf -- '- [ ] T-007 Add history to REPL\n' >> .handoff/todos.md
printf -- '---\nsession: 2026-06-11-bb22\nagent: codex\ngoal: REPL\n---\n11:00 progress T-005 — REPL loop reads input\n\n## Handoff summary\n- In flight: T-005\n' > .handoff/log/2026-06-11-bb22.md
sed -i '' 's|Parser done; evaluator half-built in src/eval.ts.|REPL started; evaluator unchanged.|' .handoff/HANDOFF.md
git add -A && git commit -qm "session b"

git checkout -q session-a && git merge session-b || true   # conflicts expected
```

## Actions
1. Start an agent with the handoff skill on the conflicted merge.
2. Prompt: "Resolve this merge."

## Expected
- [ ] `log/` files from BOTH sessions survive untouched (no conflicts possible
      — different filenames)
- [ ] `todos.md` resolves keeping both sides' line-level changes; T-006 and
      T-007 both present; no duplicate IDs
- [ ] T-004 ends `[x]` done and T-005 ends `[>]` in progress — the merge keeps
      each side's true progress (`grep -qE '^- \[x\] T-004' .handoff/todos.md
      && grep -qE '^- \[>\] T-005' .handoff/todos.md`)
- [ ] `HANDOFF.md` is REGENERATED from todos + latest logs (per the merge
      rule), not hand-merged hunk by hunk
- [ ] Merge commit completes; agent reports what it reconciled
