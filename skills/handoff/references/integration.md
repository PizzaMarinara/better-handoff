# Installing the handoff skill per harness

The canonical skill folder is `skills/handoff/` in the better-handoff repo.
"Install" always means: make that folder visible to your harness under the
name `handoff`.

## Quick matrix

| Harness | Global install | Project install | Invoke |
|---------|---------------|-----------------|--------|
| Claude Code | `~/.claude/skills/handoff/` or plugin | `.claude/skills/handoff/` | auto by description, or mention "handoff" |
| Codex | `~/.agents/skills/handoff/` | `.agents/skills/handoff/` | `$handoff` or implicit |
| pi | `~/.agents/skills/handoff/` or `~/.pi/agent/skills/handoff/` | `.agents/skills/handoff/` or `.pi/skills/handoff/` | `/skill:handoff` or implicit |

Codex and pi both scan `~/.agents/skills/` and `.agents/skills/` — one copy
serves both. Harnesses discover skills at session start, so begin a new
session after installing. Discovery matches the `name: handoff` frontmatter
field — keep it in sync with the directory name if you rename anything.

```bash
# replace <better-handoff> with the path to your clone
# one-line global install for Codex + pi
mkdir -p ~/.agents/skills && cp -R <better-handoff>/skills/handoff ~/.agents/skills/

# Claude Code (personal skill)
mkdir -p ~/.claude/skills && cp -R <better-handoff>/skills/handoff ~/.claude/skills/
```

Claude Code can also load the repo as a plugin (the repo root carries
`.claude-plugin/plugin.json` and skills are auto-discovered from `skills/`):
add the repo to a marketplace or install directly per Claude Code's plugin
docs.

## Pointer snippet (for harnesses without the skill)

`init` offers to append this to CLAUDE.md / AGENTS.md / GEMINI.md /
`.cursorrules` — it makes the repo self-announcing even where no skill
support exists:

```markdown
## Session continuity

This repo uses `.handoff/` (better-handoff convention) for session handoff
between AI agents. At session start, read `.handoff/HANDOFF.md` and follow
its embedded protocol: work from `.handoff/todos.md`, append a one-line
checkpoint to your session log whenever a todo changes state, and write a
handoff summary before ending. If you are near usage limits, save early.
```

The snippet deliberately carries no canonical URL: repos must stay
self-contained even if this skill's home moves. Revisit once the skill has a
stable published home.

## Publishing targets

| Catalogue | Mechanism |
|-----------|-----------|
| Claude Code plugins | marketplace entry pointing at this repo (`.claude-plugin/plugin.json` present) |
| Codex skills | PR adding the skill folder to `github.com/openai/skills` |
| pi skills | PR adding the skill folder to `github.com/badlogic/pi-skills` |

Before each submission, re-check the target repo's CONTRIBUTING file — folder
layout conventions may have changed since 2026-06-11.

## Degradation ladder

1. **Skill installed** — full behavior. Automatic triggering depends on the
   harness's skill matching — in a repo with no `.handoff/` yet, mention
   "handoff" explicitly; the pointer snippet above makes discovery
   deterministic.
2. **Pointer snippet only** — agent reads HANDOFF.md at start and follows its
   protocol; no init/graduate guidance, but continuity works.
3. **Nothing installed** — the protocol blockquote embedded in HANDOFF.md
   itself instructs any agent that opens it. Continuity still works once
   `.handoff/` exists and the user (or prior context) points the agent at it.
