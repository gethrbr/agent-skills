# Agent skills by Harbor

Free skills for coding agents. No account, no API key, nothing is sent to Harbor.

## audit-memory

Your `CLAUDE.md` was right the day someone wrote it. Then the code moved on.

`audit-memory` reads every agent rules and memory file in the repo and checks each line against the code:

- **Wrong**: it names a path, script or package that no longer exists
- **Out of date**: the repo moved to another tool, version or layout
- **In conflict**: two lines tell the agent opposite things
- **Not followed**: the code breaks the rule in many places
- **Duplicate**: the same rule in two files that will drift apart

Every finding comes with its proof (the file and line, the command output or the commit) and a suggested fix. It reads only. It never runs your project's scripts and never edits a file until you say which lines.

It reads `CLAUDE.md`, `AGENTS.md`, `GEMINI.md`, `.github/copilot-instructions.md`, `.cursor/rules/`, `.cursorrules`, `.windsurfrules`, `.claude/rules/`, and Claude Code's memory folder for the repo.

### What it found on real repos

Run on two popular open-source repos in November 2026:

```
AGENTS.md:8  Wrong
  says:   `npm run test:e2e` - Run E2E tests with Playwright
  repo:   root package.json has no test:e2e script; it lives in packages/app-tests
  fix:    `npm run test:e2e -w <the app-tests package>`

CLAUDE.md:3  Out of date
  says:   the frontend uses React 18
  repo:   the frontend's package.json has "react": "^19.2.0"
  fix:    change to React 19
```

## Install

**Any agent** (Claude Code, Cursor, Codex, and others), with the [skills](https://www.npmjs.com/package/skills) CLI:

```sh
npx skills add gethrbr/agent-skills
```

**Claude Code**, as a plugin:

```sh
claude plugin marketplace add gethrbr/agent-skills && claude plugin install agent-skills@gethrbr
```

## Use

In your agent, in the repo:

```
audit my agent memory files
```

or, in Claude Code, `/agent-skills:audit-memory`.

On a big repo it takes a few minutes. It ends by asking which findings to fix.

## Keeping the rules current, not just audited

An audit tells you what went stale. [Harbor](https://gethrbr.com) keeps it from going stale: it collects the rules your team settles in PR reviews, Slack and agent sessions, a person approves them, and every agent (Claude Code, Cursor, Codex) gets the ones that apply to the files it is touching. With Harbor connected, `audit-memory` also checks your files against those approved rules.

## License

MIT
