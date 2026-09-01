# Claude Code Skills

Personal [Claude Code](https://claude.com/claude-code) skills. This repository *is* the
skills directory — its root is `~/.claude/skills`, so each top-level folder is one skill
that Claude Code discovers automatically.

The `cc2-*` skills carry software construction standards derived from *Code Complete 2*
(McConnell). The `sec-*` skills carry secure development practice, structured on *Writing
Secure Code 2nd ed.* (Howard & LeBlanc) with the concrete guidance modernized against
current practice — the book is from 2002, so its principles are kept and its platform- and
era-specific recommendations are replaced. Comment discipline, function-argument limits and code layout follow *Clean Code* (Martin),
folded into `cc2-construction` rather than given their own skill. The `architecture` skill
covers structure above the module level, drawing on Clean
Architecture (Martin), ports and adapters (Cockburn) and Modern Software Engineering
(Farley). The `tdd` skill drives the red-green-refactor cycle and uses all of these as the
baseline its refactor step checks against. All are language- and project-agnostic.

## Skills

| Skill | What it does | Requires |
|---|---|---|
| [`gtd`](gtd/SKILL.md) | Runs a GTD system held in Evernote: capture, clarify, organise, surface next actions and waiting-fors, and drive the daily/weekly/quarterly reviews. | `evernote` MCP server |
| [`architecture`](architecture/SKILL.md) | Dependency direction, ports and adapters, layer boundaries, coupling and cohesion, and when a boundary should become a service. | — |
| [`tdd`](tdd/SKILL.md) | Red-green-refactor with a mandatory refactor gate that routes to the `cc2-*` and `sec-*` standards only when a trigger fires. | — |
| [`cc2-construction`](cc2-construction/SKILL.md) | Baseline construction standards applied to all code work: naming, routine and class structure, control flow, scope, comments, refactoring discipline. | — |
| [`cc2-table-driven`](cc2-table-driven/SKILL.md) | Replaces switch statements and nested conditional trees with direct-access, indexed-access, or stair-step lookup tables. | — |
| [`cc2-defensive-design`](cc2-defensive-design/SKILL.md) | Places the barricade between untrusted and trusted data, and separates assertions from error handling. | — |
| [`cc2-pseudocode-design`](cc2-pseudocode-design/SKILL.md) | Designs non-trivial logic as a reviewable outline — purpose, inputs, outputs, pre/postconditions — before any code is written. | — |
| [`cc2-debugging`](cc2-debugging/SKILL.md) | Scientific-method debugging: stabilize, reproduce minimally, hypothesize and test, fix the cause, verify, find similar defects. | — |
| [`cc2-integration`](cc2-integration/SKILL.md) | Assembling modules into a system: integration strategy, stubs and drivers, smoke tests, scaling formality to project size. | — |
| [`cc2-audit`](cc2-audit/SKILL.md) | Read-only construction-quality review of a file, module, or diff. Runs forked, reports ranked findings, changes nothing. | — |
| [`cc2-stakeholder-communication`](cc2-stakeholder-communication/SKILL.md) | Translates technical work into plain language and business consequence for release notes, status updates, and exec summaries. | — |
| [`sec-threat-modeling`](sec-threat-modeling/SKILL.md) | STRIDE threat modeling, trust boundaries, attack surface reduction, secure defaults, and the core security principles. | — |
| [`sec-input-validation`](sec-input-validation/SKILL.md) | Allowlist validation, canonicalization order, path traversal, Unicode attacks, and the resource limits that prevent DoS. | — |
| [`sec-injection-defense`](sec-injection-defense/SKILL.md) | Structural separation of code and data across SQL, shell, HTML, XML, templates, redirects — plus SSRF, XXE and deserialization. | — |
| [`sec-authz-least-privilege`](sec-authz-least-privilege/SKILL.md) | Deny-by-default authorization, object-level checks, multi-tenant isolation, and least privilege for processes, tokens and roles. | — |
| [`sec-crypto-secrets`](sec-crypto-secrets/SKILL.md) | Algorithm selection, password storage, key management, secret handling, and the ways cryptography is misused. | — |
| [`sec-memory-safety`](sec-memory-safety/SKILL.md) | Buffer overruns, integer overflow, lifetime errors and format strings — for C/C++ and unsafe blocks or FFI elsewhere. | — |
| [`sec-review-testing`](sec-review-testing/SKILL.md) | Multi-pass security review, hostile-input testing, supply chain checks, non-leaking errors, and logging for detection. | — |

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
