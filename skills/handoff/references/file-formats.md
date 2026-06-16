# .handoff/ File Formats

The complete specification of the handoff state directory. All files are
markdown, human-readable, and hand-editable. Formats are designed to be
merge-tolerant in git: one line per fact, stable IDs, per-session log files.

```
.handoff/
├── HANDOFF.md      # front page: status snapshot + embedded mini-protocol
├── todos.md        # THE working task list
├── memory.md       # decisions, gotchas, dead ends, pointers
└── log/
    └── YYYY-MM-DD-xxxx.md   # one file per session
```

## Session IDs

`YYYY-MM-DD-xxxx` where `xxxx` is 4 random lowercase alphanumeric characters
(e.g. `2026-06-11-a3f2`). Random means random — never a word like `init` or
`resume`; two same-day sessions must not collide. Generate once per session;
it names the log file and appears in cross-references.

## HANDOFF.md — front page

The only file an agent MUST read to start. About 20 lines. Fully rewritten at
each normal save; emergency saves only minimally patch lines inside the
`## Status` section and touch nothing else.

Required sections, in order:

1. A blockquote **agent protocol** at the top — instructions sufficient for an
   agent with no handoff skill installed (see references/templates/HANDOFF.md
   in the handoff skill). Never remove it.
2. `## Status` with three bullets:
   - `**Last updated:**` timestamp (`YYYY-MM-DD HH:MM`), session id,
     agent/harness (free text)
   - `**Current state:**` one paragraph, plain prose
   - `**Next steps:**` numbered list of at most 3 items, each referencing a
     todo ID

Merge rule: NEVER hand-merge a conflicted HANDOFF.md. Regenerate it from
`todos.md` + the latest log files, then mark the conflict resolved.

## todos.md — working task list

One line per item:

```
- [STATUS] T-NNN Title — optional context (blockers, links, reasons)
```

Statuses:

| Mark  | Meaning      | Rule |
|-------|--------------|------|
| `[ ]` | pending      | |
| `[>]` | in progress  | append `— see log/<session-id>` to the context |
| `[x]` | done         | |
| `[~]` | dropped      | reason is REQUIRED after the dash |

Rules:
- IDs are `T-` + zero-padded increasing number (`T-001`, `T-002` …; three
  digits minimum, grow as needed). Never
  reuse an ID, even after deletion.
- Two sections: `## Active` and `## Done`. At save time, move `[x]`/`[~]`
  items to `## Done`. Compaction (run at save when `log/` exceeds 10 files;
  procedure in SKILL.md) deletes old `## Done` entries;
  their history survives in session logs, memory, and git history.
- New todos may be added at any time during a session — append to `## Active`
  with the next free ID.
- There is no "blocked" status mark: blocked items keep their current mark
  and record the blocker in the context (`— blocked by T-NNN`).

## memory.md — durable session-borne knowledge

Four sections: `## Decisions`, `## Gotchas`, `## Dead ends`, `## Pointers`.
Every entry is one bullet, starting with an ISO date:

```
- 2026-06-11 — Native fetch wrapper instead of ky: bundle size. (T-003)
```

- Decisions must include the **why**, not just the what.
- Gotchas are surprising behaviors the next session must know to avoid wasted
  time.
- Pointers are key files, commands, and URLs discovered during work.
- Dead ends must include what was tried and why it failed, so no future
  session retries it.
- Entries promoted to an instruction file are suffixed, not deleted:
  `(graduated → CLAUDE.md, 2026-06-15)`.

## log/ — per-session journals

Filename: `log/<session-id>.md`. Structure:

```markdown
---
session: 2026-06-11-a3f2
agent: claude-code            # free text, best effort
goal: implement timeout + retry for api client
---
14:02 start — resumed from T-005
14:32 done T-005 — timeout config in client.ts; retries need jitter (→ memory)
15:10 decision — native fetch wrapper, not ky (→ memory)
15:41 risky — rewriting client.ts error handling

## Handoff summary
- Done: T-005 timeout config
- In flight: T-007 retry logic, half-implemented in client.ts
- Next: finish T-007, then T-008
- Open: should retry budget be configurable?
```

Checkpoint lines: `HH:MM <kind> <ref> — <note>` — one line, no wrapping. The
ref is a todo ID when one applies and is omitted otherwise (no literal
brackets, as in the examples above). Kinds: `start`, `done`, `progress`,
`blocked`, `decision`, `gotcha`, `deadend`, `risky`, `note`.

The `## Handoff summary` section is appended ONLY at save time. **Its absence
marks an interrupted session** — resume logic depends on this signal.

## Global rules

- Never write secrets, tokens, API keys, or credentials into any `.handoff/`
  file. These files are committed to the repository.
- Markdown only, no HTML. The prescribed separators (`·` in Status, `→` in
  graduated suffixes, `—` in checkpoint lines and contexts) are part of the
  formats — keep them.
- Timestamps are local time, `HH:MM`, dates ISO `YYYY-MM-DD`.
