<div align="center">

# 🤝 better-handoff

**Portable session handoff for AI coding agents.**

One skill + one file convention that gives any agent — Claude Code, Codex, pi, or
anything that can read markdown — a durable shared memory of your project: what
was done, why, what was tried and abandoned, and what comes next.

[![License: MIT](https://img.shields.io/badge/license-MIT-yellow.svg)](LICENSE)
[![Format: Agent Skills](https://img.shields.io/badge/format-Agent_Skills-blue.svg)](skills/handoff/SKILL.md)
![Harnesses: Claude Code · Codex · pi](https://img.shields.io/badge/harnesses-Claude_Code_%C2%B7_Codex_%C2%B7_pi-8A2BE2.svg)
![Runtime dependencies: none](https://img.shields.io/badge/runtime_dependencies-none-success.svg)
[![PRs welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg)](#-contributing)

</div>

---

## 🧩 The problem

Agentic sessions die: context fills up, subscription tokens run out, you
switch machines, accounts, or harnesses. Each death loses the session's
working memory — and most memory tools don't actually solve this, because
they keep state on the machine the session died on, inside the harness it
died in.

## 🎯 Opinions

better-handoff is opinionated. Five choices define it:

1. **State is committed — git is the sync layer.** `.handoff/` lives in your
   repo and travels with every clone. Switch machine, account, or harness:
   `git pull` is the entire sync story. Most memory tools keep state in your
   home directory; the repo-local skills gitignore it or leave committing up
   to you. Project memory belongs to the project.
2. **Checkpointing is a side effect of working, not a duty.** `.handoff/todos.md`
   IS the agent's task list — updating it is how work happens, so the record
   stays truthful even when long-context instruction-following decays. Tools
   that rely on "summarize at the end" lose everything when the session dies
   before the end.
3. **Emergency saves exist because token cutoffs exist.** Say "out of tokens,
   handoff": one single response updates state and auto-commits `.handoff/`.
   Worst case — no warning at all — the one-line milestone checkpoints are
   the recovery record.
4. **The state explains itself.** `HANDOFF.md` embeds its own protocol, so
   the receiving agent needs nothing installed — no skill, no instruction-file
   edits, no MCP server, no daemon, no database. Open the file, follow it.
5. **Session memory is not configuration.** Stable facts move to
   CLAUDE.md/AGENTS.md only through the `graduate` flow — per item, with your
   explicit approval. Your instruction files stay yours.

## 🗂️ How it works

The skill maintains a committed `.handoff/` directory in your repo:

```
.handoff/
├── HANDOFF.md      # front page: status + embedded protocol for skill-less agents
├── todos.md        # THE working task list (stable IDs)
├── memory.md       # decisions+why, gotchas, dead ends, pointers
└── log/            # one journal file per session
```

Each checkpoint is a one-line append; the expensive synthesis happens once,
at save time. Five operations: `init` (scaffold), `resume` (reality-checked
briefing + interruption recovery), the always-on checkpoint rule, `save`
(normal + single-shot emergency mode), and `graduate` — plus merge rules for
divergent state.

## ⚖️ How it differs

|  | better-handoff | claude-mem | native memory (Claude Code / Codex) | agent-handoff-skill | handover |
|---|---|---|---|---|---|
| State location | repo, **committed** | `~/.claude-mem/` (SQLite + Chroma) | home dir | repo, not committed by default | repo, gitignored |
| Survives machine/account switch | yes (git) | no | no | only if you commit it yourself | no |
| Cross-harness | yes | yes | no | yes (CC + Codex) | yes (CC + Codex) |
| Runtime deps | none | Bun worker + DB | none | none | none |
| Mid-session checkpoints | yes (working-files mechanic) | hook capture | automatic | no (end-of-session) | no |
| Out-of-tokens emergency mode | yes | no | no | no | no |
| Receiving end needs install | **no** | yes | n/a | yes | yes |

Different tools for different jobs: if you want semantic search over months
of transcripts, use claude-mem. If you want a dependency-graph issue tracker
in git, use beads. If you want the next session — on any machine, any
harness, any account — to start exactly where this one died, use
better-handoff.

## 📦 Install

| Harness | Command |
|---------|---------|
| Claude Code | `mkdir -p ~/.claude/skills && cp -R skills/handoff ~/.claude/skills/` (or install as plugin) |
| Codex | `mkdir -p ~/.agents/skills && cp -R skills/handoff ~/.agents/skills/` |
| pi | same as Codex — pi reads `~/.agents/skills/` too |

Project-local installs and the pointer snippet for skill-less harnesses:
see [`skills/handoff/references/integration.md`](skills/handoff/references/integration.md).

## 🆘 Surviving an out-of-tokens cutoff

Mid-task and the limit warning appears? Say **"out of tokens, handoff"** — the
agent does a single-response emergency save and commits `.handoff/` (accept
the proposed push if your next session is on another machine). Open any
other harness on the same repo and say **"resume"**. If the session died with
no warning at all, the per-milestone checkpoint lines are the recovery record:
the next resume detects the missing handoff summary and reconstructs.

## 🧪 Tests

Behaviour is pinned by runnable acceptance scenarios in
[`tests/scenarios/`](tests/scenarios/). Each is a Setup / Actions / Expected
checklist that a human — or a sub-agent with the skill installed — can execute
in a throwaway directory to verify a single operation (init, resume,
interrupted recovery, emergency save, compaction, graduate, merge).

## 🤝 Contributing

Issues and pull requests are welcome. If you change the skill, please re-run
the relevant scenario(s) in `tests/scenarios/` and note the outcome in your
PR. Keep the `name: handoff` frontmatter in sync with the skill directory
name, and keep `SKILL.md` harness-neutral (no tool names — only generic
read/write/append/run verbs).

## 📄 License

[MIT](LICENSE) © Enrico Fantini
