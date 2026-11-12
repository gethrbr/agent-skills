# Agent skills by Harbor

Free skills for coding agents. No account, no API key, nothing is sent to Harbor.

| Skill | What it does |
| :- | :- |
| [`audit-memory`](#audit-memory) | Checks your `CLAUDE.md`, `AGENTS.md`, Cursor rules and memory files against the code, and finds the lines that went stale |
| [`check-diff`](#check-diff) | Checks your changes against the repo's own agent rules before you commit |
| [`rules-from-reviews`](#rules-from-reviews) | Reads your merged PR reviews, finds what reviewers keep asking for, and drafts it as rules your agent will follow |

**We ran `audit-memory` on 10 popular open-source repos: 8 had lines that send the agent the wrong way.** [See the results.](results/2026-11-ten-repos.md)

## Install

**Any agent** (Claude Code, Cursor, Codex, and others), with the [skills](https://www.npmjs.com/package/skills) CLI:

```sh
npx skills add gethrbr/agent-skills
```

**Claude Code**, as a plugin:

```sh
claude plugin marketplace add gethrbr/agent-skills && claude plugin install agent-skills@gethrbr
```

### Is it safe to install?

A skill runs with your agent's permissions, so read it first. Each skill is one Markdown file ([audit-memory](skills/audit-memory/SKILL.md), [check-diff](skills/check-diff/SKILL.md), [rules-from-reviews](skills/rules-from-reviews/SKILL.md)) with no scripts, binaries or install hooks, and all three only read: they never edit a file until you say so. `rules-from-reviews` reads your PRs through your own signed-in `gh` CLI; nothing goes anywhere else. Every change to this repo goes through a reviewed pull request, and release tags cannot be moved once published. See [SECURITY.md](SECURITY.md).

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

Use it: ask your agent to *audit my agent memory files*, or in Claude Code run `/agent-skills:audit-memory`. On a big repo it takes a few minutes.

## check-diff

Your repo wrote its rules down. `check-diff` checks your staged changes (or all uncommitted ones) against them, reading the rules files the way your agent does: the nearest `AGENTS.md` for each changed file, Cursor rules whose globs match, Copilot instructions whose `applyTo` matches.

```
web/src/features/ai-features/server/availability.ts:1
  rule:   AGENTS.md  "Never put internal ticket ids into code comments, ..."
  change: + // Retry once on timeout, see LFE-11234 for the incident.
  fix:    remove the ticket id; describe the incident in plain words
```

It also catches required steps the diff is missing, like a changeset or a migration. A clean diff gets one line saying which rules it was checked against.

Use it: ask your agent to *check my diff against the repo rules* before you commit, or in Claude Code run `/agent-skills:check-diff`.

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

Use it: ask your agent *what do my reviewers keep asking for?*, or in Claude Code run `/agent-skills:rules-from-reviews`.

## Keeping the rules current, not just audited

These skills run once, when you ask. [Harbor](https://gethrbr.com) keeps the rules current all the time: it collects the rules your team settles in PR reviews, Slack and agent sessions, a person approves them, and every agent (Claude Code, Cursor, Codex) gets the ones that apply to the files it is touching. With Harbor connected, `audit-memory` also checks your files against those approved rules.

## License

MIT
