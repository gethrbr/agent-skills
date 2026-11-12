---
name: check-diff
description: Check the current changes against this repo's own agent rules (CLAUDE.md, AGENTS.md, Cursor rules, Copilot instructions) before committing. Use when the user asks to check, review or verify their changes or diff against the repo's rules or conventions, or before a commit or pull request.
---
# Check the diff against the repo's rules

Find every place where the current changes break a rule this repo wrote down for its agents. Report only. Never edit, stage or commit anything during the check.

## 1. Get the diff

From the repo root (`git rev-parse --show-toplevel`):

- Staged changes first: `git diff --cached`.
- If nothing is staged: `git diff HEAD` (all uncommitted changes).
- If the user named a branch or a range, use that instead, for example `git diff main...HEAD`.

If there are no changes, say so in one line and stop. Skip lockfiles, generated files and vendored code.

## 2. Collect the rules that apply

Rules files are read the way the agents read them, nearest folder wins:

- `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` at the root and in **every folder between the root and each changed file** (`packages/api/AGENTS.md` applies to `packages/api/src/x.ts`, not to `web/`)
- `.cursor/rules/*.mdc`: a rule applies when its `globs` match the changed file, or it has `alwaysApply: true`
- `.claude/rules/**/*.md`, `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` (match their `applyTo`), `.cursorrules`, `.windsurfrules`
- Files these import with an `@path` line

A symlinked rules file is one file: read it once.

Keep only rules a diff can break: conventions ("named exports only", "no default exports"), bans ("never import X directly"), required steps ("add a changeset for changes to `packages/`", "update the migration when you change `schema.prisma`"), naming and placement ("services live in `services/`"). Skip rules about how to talk to the user or how to run tools.

## 3. Check each changed hunk

For every added or changed line, ask which collected rule it breaks. Look at the hunk with its surrounding code (`git diff -U10` or read the file) so you judge it in context.

Also check the **required steps** against the whole diff: a rule like "add a changeset when you change `packages/`" is broken by what is missing, not by a line.

Before you report a finding, try to prove yourself wrong: the rule may be scoped to another folder, to new code only, or the repo may plainly allow an exception right next to it. If you are unsure, leave it out.

## 4. Report

Lead with one line: `Checked 7 files against 23 rules from 4 files: 2 findings.`

Then each finding:

```
src/api/users.ts:42
  rule:   AGENTS.md:18  "Use named exports only"
  change: + export default function getUser(...)
  fix:    export function getUser(...), and update the 2 imports in src/routes/
```

Missing required steps go last:

```
(missing)  .changeset/
  rule:   AGENTS.md:31  "Add a changeset for any change to packages/"
  change: packages/core/src/index.ts changed, no changeset added
  fix:    pnpm changeset
```

No findings: say which rules files you checked against, in one line, and stop. Do not pad.

End by asking whether to fix the findings. Change only what the user agrees to.
