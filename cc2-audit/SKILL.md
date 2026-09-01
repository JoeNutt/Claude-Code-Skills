---
name: cc2-audit
description: This skill should be used when the user asks for a construction-quality review of existing code against their standards - a whole file, module, package, or diff. It audits naming, routine and class structure, control flow, defensive boundaries, and comments, and reports ranked findings without modifying anything. Trigger phrases include "cc2 audit", "audit this module", "review this against my standards", "code quality review", "is this well constructed", "review this file for quality", "construction review", "where is the complexity here".
context: fork
allowed-tools: Read, Grep, Glob, Bash
---

# Construction Audit

A read-only review of existing code against the construction standards in the user's global
`CLAUDE.md`. This skill runs in a forked context and **never modifies code** — it reports.
Applying fixes is a separate, explicit step the user asks for afterwards.

## Scope

Audit exactly what was named — a file, a module, a package, or a diff. If the target is
ambiguous, audit the smallest defensible unit and say what you covered. Do not expand into
neighbouring modules; note them as out of scope instead.

For a diff, judge the changed lines and the code they directly touch. Pre-existing problems
elsewhere in the file are context, not findings.

## Method

Read the code first, in full. Never report on a file you have only grepped — structural
judgments require having read the structure. Use `grep` and `glob` to find call sites and
usages, not as a substitute for reading.

Work through the dimensions below. `references/audit-checklist.md` holds the full checklist
with the specific thresholds and the smells that indicate each problem — read it at the
start of any audit larger than a single short file.

1. **Complexity** — nesting depth, cyclomatic branching, routine length, conditional trees
   that want to be tables.
2. **Routines and classes** — single responsibility, parameter count and order, cohesion,
   visibility, coupling and Law of Demeter violations.
3. **Naming** — vague identifiers, negated booleans, verb-noun pairing, magic numbers.
4. **Data and scope** — live span, unnecessary breadth of scope, global mutable state,
   numeric type choices where rounding accumulates.
5. **Defensive boundaries** — where untrusted data enters, whether validation is
   consolidated or scattered, assertions used for input, swallowed errors, silent defaults.
6. **Comments and layout** — comments restating syntax, absent *why* on non-obvious
   decisions, commented-out code, stale comments contradicting the code.
7. **Testability** — untested boundaries and failure paths, routines that cannot be tested
   without heavy setup.

## Reporting

Rank findings by cost of leaving them, not by how easy they are to fix. A silently
swallowed error outranks twenty naming nits.

For each finding give:

- **Location** — `path/to/file.ext:LINE`, so it is clickable.
- **What** — the specific defect, in one sentence.
- **Why it costs** — the concrete failure or maintenance burden it produces. If you cannot
  name a consequence, it is a preference, not a finding: drop it.
- **Fix** — the change in a sentence, or a short snippet where the shape is not obvious.

Then a short summary: the two or three themes that account for most of the findings, and
what you would change first.

## Discipline

- **Verify before asserting.** Never report a defect in code you have not read, and never
  claim a call site exists without having found it. Quote the line if in doubt.
- **Cite the rule, not the book.** Say "this routine takes nine parameters; the limit is
  three, with no ordering rationale" — not "McConnell says" or "Martin says". The standard is
  the user's, not an authority's.
- **Say when the code is fine.** A short audit reporting three real findings is more useful
  than thirty padded ones. Reporting nothing significant is a valid, welcome outcome.
- **Respect deliberate deviation.** Idiomatic patterns of the language or framework, and
  anything a project `CLAUDE.md` mandates, are not defects. Where a project rule conflicts
  with the global standard, the project rule wins — note it and move on.
- **Do not restate the code.** The user has it. Report the judgment.
