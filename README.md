# Deckhand

Free skills that keep your coding agent's rules honest. Works in Claude Code, Codex, Cursor and any agent that reads skills. No account, no API key, nothing is sent to Harbor.

| Skill | What you get |
| :- | :- |
| [`told-you-twice`](#told-you-twice) | Everything you keep telling your agent, counted from your own session history, drafted as rules |
| [`rules-score`](#rules-score) | A score out of 100 for your `CLAUDE.md` / `AGENTS.md`, the lines that cost points, and a README badge |
| [`context-cost`](#context-cost) | How many tokens your agent reads before you type, and what to cut or move |
| [`write-rules`](#write-rules) | An `AGENTS.md` written from your CI, configs and git history, with a source for every line |
| [`audit-memory`](#audit-memory) | Every line in your rules and memory files that the code proves wrong or out of date |
| [`check-diff`](#check-diff) | Your changes checked against the repo's own rules before you commit |
| [`rules-from-reviews`](#rules-from-reviews) | What your PR reviewers keep asking for, drafted as rules |

**We ran `audit-memory` on 10 popular open-source repos: 8 had lines that send the agent the wrong way.** [See the results.](results/2026-11-ten-repos.md)

## Install

**Any agent** (Claude Code, Cursor, Codex, and others), with the [skills](https://www.npmjs.com/package/skills) CLI:

```sh
npx skills add gethrbr/deckhand
```

**Claude Code**, as a plugin:

```sh
claude plugin marketplace add gethrbr/deckhand && claude plugin install deckhand@gethrbr
```

Installed it as `agent-skills` before the rename? Run `claude plugin uninstall agent-skills@gethrbr`, then the line above.

### Is it safe to install?

A skill runs with your agent's permissions, so read it first. Each skill is one Markdown file in [`skills/`](skills) with no scripts, binaries or install hooks, and every one only reads until you say what to change. `told-you-twice` reads your own session logs and keeps everything on your machine; `rules-from-reviews` reads your PRs through your own signed-in `gh` CLI. Every change to this repo goes through a reviewed pull request, and release tags cannot be moved once published. See [SECURITY.md](SECURITY.md).

## told-you-twice

Every correction you type twice is a rule nobody wrote down.

`told-you-twice` reads your past Claude Code and Codex sessions on this machine (only the messages you typed), finds what you corrected in two or more sessions, and counts it. Then it checks your rules files: is it written nowhere, written but still ignored, or contradicted by another line?

```
1. Use pnpm, never npm or yarn                          23 times in 9 sessions
     "no, pnpm not npm"                     Nov 3
     "i told you, pnpm"                     Nov 9
     written: nowhere
     add to:  AGENTS.md

2. Run the type check before you say it's done          11 times in 7 sessions
     "did you even run tsc?"                Nov 5
     written: CLAUDE.md:12 "Verify your changes", still repeated after it
     fix:     name the command: "Run `pnpm typecheck` before saying done"
```

The most useful finding is usually the second kind: a rule that is already written and the agent still misses, because the line is too vague. Your logs never leave your machine, and the summary at the end has counts and rule titles only, so you can post it.

Use it: ask your agent *what do I keep telling you?*, or in Claude Code run `/deckhand:told-you-twice`. Add *all my projects* to read every repo.

## rules-score

One number for how much your agent can trust its rules, on a fixed rubric: **accurate** (40: paths, commands and tools that exist), **essentials** (25: setup, build, test, one test, lint, layout), **specific** (15), **lean** (10: tokens loaded every session) and **no noise** (10: nothing a linter already enforces, no repeats).

```
Agent rules score: 90/100 (A)

  Accurate     40/40   every path, command and script checked, none wrong
  Essentials   15/25   no install line, no build/run line
  Specific     15/15   18 of 18 lines name something concrete
  Lean         10/10   ~880 tokens loaded every session
  No noise     10/10   nothing restates the formatter config, no repeats
```

That's [trpc/trpc](https://github.com/trpc/trpc), from a real run. Every lost point comes with its `file:line`, proof and fix, and you get a badge for your README:

[![agent rules: 90/100](https://img.shields.io/badge/agent%20rules-90%2F100-brightgreen)](https://github.com/gethrbr/deckhand)

Use it: ask your agent to *score our agent rules*, or in Claude Code run `/deckhand:rules-score`.

## context-cost

Your agent reads your rules, imports, memory index, skill descriptions and MCP tools at the start of every session, before you type a word. `context-cost` measures each one and finds what to cut or move:

```
File                         Loads       Tokens
~/.claude/CLAUDE.md          always       3,900
CLAUDE.md                    always       2,800
docs/conventions.md (@)      always       2,100
skills (31, front matter)    always       1,100
packages/api/CLAUDE.md       on demand      900

-1,600  docs/conventions.md, lines 40-180    one folder only
        the whole "Mobile app" section is about apps/mobile
        move:  apps/mobile/CLAUDE.md (loads only there)
```

It also catches the silent ones: Codex stops reading `AGENTS.md` after 32 KiB, and Claude Code skips `AGENTS.md` entirely when a `CLAUDE.md` exists that does not import it.

Use it: ask your agent *how many tokens do you read before I type?*, or in Claude Code run `/deckhand:context-cost`.

## write-rules

No `AGENTS.md` yet? `write-rules` writes one from what the repo actually does: CI first (that's what "done" means), then scripts, tool configs, folder layout and the last 300 commits. Every line shows its source before anything is written, and it leaves out what your linter already enforces.

```
- Run in this order, as CI does: `pnpm run check`, `pnpm run editorconfig-checker`, `pnpm run build`, `pnpm run test`   [.github/workflows/ci.yml:58-61]
- Tests sit next to the source as `*.test.ts` (125 files in src/)                        [git ls-files src]
- Don't bump adapter versions when releasing hono; adapters are tagged `@hono/<name>@<version>`   [docs/MAINTAINING.md]
```

That's from a real run on [honojs/hono](https://github.com/honojs/hono), which has no rules file.

Use it: ask your agent to *write an AGENTS.md for this repo*, or in Claude Code run `/deckhand:write-rules`.

## audit-memory

Your `CLAUDE.md` was right the day someone wrote it. Then the code moved on.

`audit-memory` reads every agent rules and memory file in the repo and checks each line against the code:

- **Wrong**: it names a path, script or package that no longer exists
- **Out of date**: the repo moved to another tool, version or layout
- **In conflict**: two lines tell the agent opposite things
- **Not followed**: the code breaks the rule in many places
- **Duplicate**: the same rule in two files that will drift apart

Every finding comes with its proof (the file and line, the command output or the commit) and a suggested fix. Before it reports one, it tries to prove itself wrong (hoisted packages, symlinked files, rules scoped to one folder). It never runs your project's scripts.

It reads `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`, `.cursor/rules/`, `.cursorrules`, `.windsurfrules`, `.claude/rules/`, and Claude Code's memory folder for the repo.

```
AGENTS.md:8  Wrong
  says:   `npm run test:e2e` - Run E2E tests with Playwright
  repo:   root package.json has no test:e2e script; it lives in packages/app-tests
  fix:    `npm run test:e2e -w <the app-tests package>`
```

Use it: ask your agent to *audit my agent memory files*, or in Claude Code run `/deckhand:audit-memory`. On a big repo it takes a few minutes.

## check-diff

Your repo wrote its rules down. `check-diff` checks your staged changes (or all uncommitted ones) against them, reading the rules files the way your agent does: the nearest `AGENTS.md` for each changed file, Cursor rules whose globs match, Copilot instructions whose `applyTo` matches.

```
web/src/features/ai-features/server/availability.ts:1
  rule:   AGENTS.md  "Never put internal ticket ids into code comments, ..."
  change: + // Retry once on timeout, see LFE-11234 for the incident.
  fix:    remove the ticket id; describe the incident in plain words
```

It also catches required steps the diff is missing, like a changeset or a migration. A clean diff gets one line saying which rules it was checked against.

Use it: ask your agent to *check my diff against the repo rules* before you commit, or in Claude Code run `/deckhand:check-diff`.

## rules-from-reviews

Your reviewers write the same comment on PR after PR. Each one is a rule nobody wrote down, so the agent breaks it again next week.

`rules-from-reviews` reads the reviews on your recent merged PRs (human reviewers only, no bots, no replies), keeps the asks that came up in at least two PRs, and checks each one against your rules files and your code:

- **Not written down**: a new rule, with the comments that asked for it and the file it belongs in
- **Written but missed**: the file says it, reviewers still had to ask, so the line is too vague or in the wrong folder
- **Reviewers disagree**: two reviewers asked for opposite things, and nothing settles it yet

```
Comments describe the code as it is now. No "fossil" comments that compare old and new behaviour
  asked:   5 times in 3 PRs by 3 reviewers
           #18893 "a 'fossil comment'. Compare and contrast between an old state and new state are typically not needed"
           #19362 "nit: stale comment, images don't use the thumbnail anymore right?"
  written: partly. AGENTS.md keeps comments for a "non-obvious why", nothing on stale ones
  add to:  AGENTS.md, under "Commenting Guidelines"
```

That one is from a real run on [payloadcms/payload](https://github.com/payloadcms/payload). Needs the [GitHub CLI](https://cli.github.com) signed in.

Use it: ask your agent *what do my reviewers keep asking for?*, or in Claude Code run `/deckhand:rules-from-reviews`.

## Keeping the rules current, not just audited

These skills run when you ask. [Harbor](https://gethrbr.com) keeps the rules current all the time: it collects the rules your team settles in PR reviews, Slack and agent sessions, a person approves them, and every agent (Claude Code, Cursor, Codex) gets the ones that apply to the files it is touching. With Harbor connected, `audit-memory` also checks your files against those approved rules.

## License

MIT
