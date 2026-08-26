# Claude Code Skills

Personal [Claude Code](https://claude.com/claude-code) skills. This repository *is* the
skills directory — its root is `~/.claude/skills`, so each top-level folder is one skill
that Claude Code discovers automatically.

## Skills

| Skill | What it does | Requires |
|---|---|---|
| [`gtd`](gtd/SKILL.md) | Runs a GTD system held in Evernote: capture, clarify, organise, surface next actions and waiting-fors, and drive the daily/weekly/quarterly reviews. | `evernote` MCP server |

## Installing on another machine

The repository root must land *as* `~/.claude/skills`, not inside it:

```sh
git clone https://github.com/JoeNutt/Claude-Code-Skills.git ~/.claude/skills
```

If that directory already exists, clone elsewhere and move the contents in, or point an
existing checkout at this remote:

```sh
cd ~/.claude/skills
git init
git remote add origin https://github.com/JoeNutt/Claude-Code-Skills.git
git pull origin main
```

Skills are picked up at the start of a session, so restart Claude Code after cloning. Run
`/help` or ask "what skills do you have?" to confirm they loaded.

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

Create a folder, write a `SKILL.md` with `name` and `description` frontmatter, restart, and
commit. Behavioural instructions belong in `SKILL.md` and its references — this README
describes the repository, and deliberately does not restate how any skill works.
