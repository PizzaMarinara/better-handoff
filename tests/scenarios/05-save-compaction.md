# Scenario 05 — normal save with compaction

## Setup
Scenario 02 state, plus 11 old session log files (exceeds the 10-file
threshold), each containing at least one `decision` or `gotcha` checkpoint:

```bash
mkdir -p /tmp/handoff-s05 && cd /tmp/handoff-s05
# First build the scenario 02 fixture here: run scenario 02's Setup block
# with /tmp/handoff-s02 replaced by /tmp/handoff-s05. Then:
for i in $(seq -w 1 11); do
cat > .handoff/log/2026-05-$i-ab$i.md <<EOF
---
session: 2026-05-$i-ab$i
agent: test
goal: filler session $i
---
10:00 decision — decision number $i (→ memory)
10:30 done T-9$i — work item $i

## Handoff summary
- Done: T-9$i
EOF
done
git add -A && git commit -qm "many logs"
```

## Actions
1. Resume; do a small amount of work.
2. Prompt: "Wrap up the session."

## Expected
- [ ] Session log has a full `## Handoff summary` (done / in flight / next /
      open questions)
- [ ] `HANDOFF.md` fully rewritten: fresh timestamp, current state, ≤3 next steps
- [ ] `.handoff/log/` now contains ≤10 files (oldest deleted)
- [ ] Durable facts from deleted logs appear in `memory.md` as dated entries
- [ ] The agent PROPOSED a commit and waited for approval (normal save asks
      first — opposite of scenario 04)
