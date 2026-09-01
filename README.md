# Claude Code Skills

Personal [Claude Code](https://claude.com/claude-code) skills. This repository *is* the
skills directory — its root is `~/.claude/skills`, so each top-level folder is one skill
that Claude Code discovers automatically.

## Skills

| Skill | What it does | Requires |
|---|---|---|
| [`gtd`](gtd/SKILL.md) | Runs a GTD system held in Evernote: capture, clarify, organise, surface next actions and waiting-fors, and drive the daily/weekly/quarterly reviews. | `evernote` MCP server |
| [`cc2-construction`](cc2-construction/SKILL.md) | Baseline construction standards applied to all code work: naming, routine and class structure, control flow, scope, comments, refactoring discipline. | — |
| [`cc2-table-driven`](cc2-table-driven/SKILL.md) | Replaces switch statements and nested conditional trees with direct-access, indexed-access, or stair-step lookup tables. | — |
| [`cc2-defensive-design`](cc2-defensive-design/SKILL.md) | Places the barricade between untrusted and trusted data, and separates assertions from error handling. | — |
| [`cc2-pseudocode-design`](cc2-pseudocode-design/SKILL.md) | Designs non-trivial logic as a reviewable outline — purpose, inputs, outputs, pre/postconditions — before any code is written. | — |
| [`cc2-debugging`](cc2-debugging/SKILL.md) | Scientific-method debugging: stabilize, reproduce minimally, hypothesize and test, fix the cause, verify, find similar defects. | — |
| [`cc2-integration`](cc2-integration/SKILL.md) | Assembling modules into a system: integration strategy, stubs and drivers, smoke tests, scaling formality to project size. | — |
| [`cc2-audit`](cc2-audit/SKILL.md) | Read-only construction-quality review of a file, module, or diff. Runs forked, reports ranked findings, changes nothing. | — |
| [`cc2-stakeholder-communication`](cc2-stakeholder-communication/SKILL.md) | Translates technical work into plain language and business consequence for release notes, status updates, and exec summaries. | — |

## Installing on another machine

The repository root must land *as* `~/.claude/skills`, not inside it:

```sh
git clone https://github.com/JoeNutt/Claude-Code-Skills.git ~/.claude/skills
```

Nothing else to run. Everything this repository provides lives inside it, so a clone is a
complete install on any machine.

If that directory already exists, clone elsewhere and move the contents in, or point an
existing checkout at this remote:

```sh
cd ~/.claude/skills
git init
git remote add origin https://github.com/JoeNutt/Claude-Code-Skills.git
git pull origin main
```

New skill directories are picked up during a running session, so no restart is needed after
cloning or adding one. Ask "what skills do you have?" to confirm they loaded.

## Layout

Claude Code finds a skill by looking for `SKILL.md` one level down. Anything else in the
folder is only read when `SKILL.md` points to it, which keeps the always-loaded footprint
to the frontmatter alone.

```
<skill-name>/
  SKILL.md        # frontmatter (name + description) and the instructions
  references/     # detail loaded on demand, not at startup
```

The `description` in the frontmatter is the only thing Claude sees when deciding whether a
skill applies, so it carries the trigger phrases.

## Adding a skill

Create a folder, write a `SKILL.md` with `name` and `description` frontmatter, and commit. Behavioural instructions belong in `SKILL.md` and its references — this README
describes the repository, and deliberately does not restate how any skill works.
