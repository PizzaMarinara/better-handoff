---
name: handoff
description: Use when a repo contains a .handoff/ directory (always resume from it at session start), when the user asks to resume, hand off, wrap up, checkpoint, or save work, when usage limits or out-of-tokens warnings appear (emergency save), when .handoff/ files have merge conflicts, or when the user wants session memory set up in a project (init). Portable session continuity across agents, machines, and harnesses — a committed work log, decision memory, and todo list any agent can pick up.
license: MIT
---

# Handoff — Portable Session Continuity

Sessions die: context fills up, subscription tokens run out, users switch
machines, accounts, or tools. This skill maintains a durable, harness-agnostic
memory of project work in a committed `.handoff/` directory, so any agent —
in any harness, on any machine — picks up exactly where the last session
stopped, losing at most one milestone of progress.

## Core principle: working files, not mirrors

`.handoff/todos.md` IS your task list for the session — not a copy of one you
keep elsewhere. You work from it, and updating it is how work happens, not an
extra reporting duty. This is what keeps the handoff state truthful even when
your context is long and instructions have faded: maintaining your task list
is a habit you already have.

## State directory

| File | Role | You read it… |
|------|------|--------------|
| `HANDOFF.md` | front page: status + embedded protocol | always, first |
| `todos.md`   | the working task list | always |
| `memory.md`  | decisions+why, gotchas, dead ends, pointers | always |
| `log/<session-id>.md` | per-session journal | latest 1–2 only |

Exact syntax for every file: `references/file-formats.md` (in this skill's
directory). Blank starting files: `references/templates/`.

## Choosing the operation

| Situation | Operation |
|-----------|-----------|
| Repo has `.handoff/` and a session is starting | **resume** |
| User wants handoff set up; no `.handoff/` yet | **init** |
| Todo changed state / decision / gotcha / risky step ahead | **checkpoint** |
| User wraps up, or session is winding down | **save** |
| Out-of-tokens or usage-limit warning | **save (emergency)** |
| Conflicted `.handoff/` files after a merge or pull | **merge** (see Safety rules) |
| User wants memory promoted to CLAUDE.md/AGENTS.md | **graduate** |

If any operation is invoked and no `.handoff/` exists, offer init first —
never scaffold silently.

## init

1. Create `.handoff/` and `.handoff/log/`, then copy `HANDOFF.md`,
   `todos.md`, and `memory.md` from `references/templates/` into `.handoff/`.
   Do NOT copy `session-log.md` — it is instantiated at resume.
2. Seed content — interview the user briefly (project goal, current state,
   first todos), and mine what exists: CLAUDE.md/AGENTS.md, recent git
   history.
3. Fill every `<angle bracket>` placeholder in HANDOFF.md's Status section.
   Keep the agent-protocol blockquote verbatim.
4. Offer — never silently apply — to append the pointer snippet (see
   `references/integration.md`) to whichever instruction files exist:
   CLAUDE.md, AGENTS.md, GEMINI.md, `.cursorrules`.
5. Propose committing `.handoff/`.

## resume

1. Read `HANDOFF.md`, `todos.md`, `memory.md`, and the most recent log file
   (two if the latest is small). If `log/` is empty, this is the first
   working session — skip step 3. If an expected file is missing, recreate
   it from its template, reconstruct what you can from git and the remaining
   files, and note it in the briefing.
2. **Reality check.** Compare claimed state against the repo: `git status`,
   `git log` since the last save, key files mentioned in todos. Reality wins:
   if todos say done but the code disagrees, flag it and fix the files.
3. **Interruption check.** If the latest log has no `## Handoff summary`, the
   previous session died mid-flight (crash, out of tokens). Reconstruct from
   its checkpoint lines plus git evidence, and say so explicitly in the
   briefing. Leave the interrupted log intact.
4. Brief the user in a few sentences: where things stand, what is in flight,
   the recommended next step. No invented facts — everything traceable to the
   files or git.
5. Create this session's log file: `log/<YYYY-MM-DD>-<4 random a-z0-9>.md`
   from the session-log template, frontmatter filled, with a `start`
   checkpoint line. The 4 characters must be random (e.g. `a3f2`) — never a
   word like `init` or `resume`; randomness prevents filename collisions
   between parallel or same-day sessions.
6. Adopt `todos.md` as your working task list for the session, and follow the
   checkpoint rule below for the rest of the session.

## Checkpoint rule (in force for the whole session)

Your task list lives in `todos.md`. Whenever a todo changes state — started,
finished, blocked, dropped — make the change in the file AND append one line
to your session log:

```
HH:MM <kind> <T-id if one applies> — <short note>
```

Kinds: `start done progress blocked decision gotcha deadend risky note`.
Blocked items keep their status mark and record the blocker in the context
(`— blocked by T-NNN`).

Also checkpoint (one line, same format) when:
- you make a decision worth a "why" — record it in `memory.md` too if durable
- you discover a gotcha or a dead end — same
- you are about to do something risky or destructive

Each checkpoint is ONE appended line. No rewriting files, no prose. The
expensive synthesis happens once, at save.

If usage limits are getting near, proactively offer an emergency save. If
the user says they are out of tokens — or the harness issues a hard limit
warning — execute the emergency save immediately, without asking.

## save

**Normal** (user wraps up, or you sense the session winding down — propose it):

1. Reconcile `todos.md` against what actually happened; move `[x]`/`[~]`
   items to `## Done`.
2. Append `## Handoff summary` to the session log: done / in flight /
   recommended next step / open questions.
3. Rewrite `HANDOFF.md` (keep the protocol blockquote; refresh all of Status).
4. **Compaction**, only if `log/` exceeds 10 files: distill durable facts from
   the oldest logs into `memory.md` (dated entries), then delete those logs
   until 10 remain; also prune the corresponding old entries from todos.md's
   `## Done` section. Memory is the long-term store; logs are a buffer —
   history survives in memory and git.
5. Propose a commit of `.handoff/` only — conventional message, e.g.
   `chore(handoff): 2026-06-11 session — timeout config landed`. Ask first.

**Emergency** (user says out-of-tokens / harness warns): one single response,
minimum tokens. Update todo statuses; append a 2–3 line `## Handoff summary`;
patch only HANDOFF.md's Status lines; **commit `.handoff/` files only, without
asking** (stage only `.handoff/` paths — never `-a`/`-A`; same conventional
format). Propose — never execute — a push. Skip
reconciliation, compaction, and synthesis. Done is better than thorough: the
next resume's reality check will catch anything missed.

## graduate

On user request, or suggest it at save time when a memory entry keeps proving
relevant across sessions:

1. Propose specific `memory.md` entries as candidates for CLAUDE.md or
   AGENTS.md, quoting each.
2. Per-item user approval is MANDATORY before touching any instruction file.
3. Copy approved entries over (matching the target file's style); suffix the
   memory entry with `(graduated → <file>, <date>)` — never delete it.

## Safety rules

- **Never write secrets**, tokens, API keys, or credentials into `.handoff/`
  files. They are committed to the repository.
- **Never edit instruction files** (CLAUDE.md, AGENTS.md, GEMINI.md,
  `.cursorrules`) without explicit per-item user approval.
- **Reality wins** over recorded state; reconcile files to match the repo.
- **Merging divergent `.handoff/` state:** `log/` files never conflict (one
  file per session — keep both sides' files). For `todos.md`, take the union
  of both sides' lines; where the same todo differs, the status backed by a
  session log wins (a `done` checkpoint beats a stale `[>]`); if both sides
  are log-backed and disagree, keep the more progressed status and flag it
  in your report. For `memory.md`, keep both sides' bullets (union per
  section). **Never hand-merge `HANDOFF.md`** — regenerate it from the
  merged `todos.md` + latest logs. Report what was reconciled.
- Git autonomy: commits of `.handoff/` are proposed, except during emergency
  save where they are automatic; pushes are always proposed, never automatic.
- No git in the repo? Everything still works as plain files — skip the
  reality check and commit steps.

## Harness notes

This skill is harness-neutral: file operations are described generically.
Use whatever read/write/append/run facilities your environment provides.
Install paths and per-harness setup: `references/integration.md`.
