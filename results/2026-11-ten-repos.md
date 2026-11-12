# We audited the agent rules files of 10 popular open-source repos

November 2026. We ran `audit-memory` on 10 well-known open-source repos that ship agent instructions (`CLAUDE.md`, `AGENTS.md`, Cursor rules, Copilot instructions), then checked every finding below by hand against the repo at the commit listed.

**8 of 10 had at least one line that would send an agent the wrong way.** These are good, well-run projects. That's the point: rules files go stale in every repo, because nothing tells you when the code moves on.

| Repo | Commit | Confirmed findings |
| :- | :- | :- |
| [payloadcms/payload](#payload) | `7a80820ce` | 4 |
| [calcom/cal.com](#calcom) | `ef7954d16` | 4 |
| [triggerdotdev/trigger.dev](#triggerdev) | `7a2a7dc80` | 8 |
| [novu/novu](#novu) | `b2420a4e9` | 4 |
| [langfuse/langfuse](#langfuse) | `72399b76e` | 2 |
| [documenso/documenso](#documenso) | `2d68c47da` | 4 |
| [twentyhq/twenty](#twenty) | `444ad378c` | 2 |
| [excalidraw/excalidraw](#excalidraw) | `701b901a0` | 2 |
| [formbricks/formbricks](#formbricks) | `f61a6d476` | 0 confirmed |
| [trpc/trpc](#trpc) | `8eff9ff0d` | 0, clean |

What kept coming up:

- **Paths that moved.** A folder gets renamed or deleted; the rules file still points at it. The agent searches for it, finds nothing, and guesses.
- **Tools that changed.** eslint to oxlint, Remix to React Router, React 18 to 19. The rules file still names the old one.
- **Two files that disagree.** `AGENTS.md` says one thing, `.cursor/rules/` or a nested `AGENTS.md` says the opposite. Which one the agent follows depends on which it read last.

## payload

- `AGENTS.md:120` says auto-login is disabled with `PAYLOAD_PUBLIC_DISABLE_AUTO_LOGIN=false` or a `--no-auto-login` flag. `test/buildConfigWithDefaults.ts:182` only disables it when the variable is `'true'`, and nothing reads that flag.
- `AGENTS.md:395` says `pnpm run lint` runs all linters. It runs `oxlint packages tools` only; stylelint never runs there.
- `AGENTS.md:289`: `packages/richtext-lexical` has no `translateNewKeys` script.
- `packages/create-payload-app/AGENTS.md:46-47` documents `addDatabaseAdapter` and `addStorageAdapter` with positional arguments; both take one object now.

## cal.com

- `AGENTS.md:130`, `agents/knowledge-base.md:91` and `agents/rules/reference-file-locations.md:29` point at `packages/features/ee/workflows/lib/constants.ts`. `packages/features/ee` has no tracked files.
- Three rules files point at `packages/features/calendar-cache-sql`, which is gone; the cache lives in `packages/features/calendar-subscription/lib/cache`.
- `agents/rules/quality-imports.md:18-23` marks a named `AppleCalendarService` import as good. The file is `export default function BuildCalendarService`, and 20 app-store files follow that pattern, so the example teaches the opposite of the code.
- Two rules name `apps.browser-addon.generated.tsx` / `apps.browser-*.generated.tsx`; the only file is `apps.browser.generated.tsx`.

## trigger.dev

- Six paths that moved or don't exist: `app/v3/eventRepository.server.ts` (now `app/v3/eventRepository/`), `app/v3/presenters/**` (now `app/presenters/v3/`), run-engine `src/locking.ts` (now `src/engine/locking.ts`), the `@trigger.dev/core/v3/utils` import (not an export), `legacyRunEngineWorker.server.ts` (gone), `docs/documentation/` (doesn't exist).
- `packages/cli-v3/AGENTS.md:38` says to update `rules/` and `.claude/skills/trigger-dev-tasks/` when SDK features change; three other files say not to unless asked.
- `.claude/rules/sdk-packages.md:8` says to always add a changeset for `packages/`; `AGENTS.md` says to skip it when the change isn't user-facing.

## novu

- `.cursor/rules/clickhouse.mdc:19` says to filter ClickHouse queries by `_environmentId` and `_organizationId`. The columns are `environment_id` and `organization_id`.
- `AGENTS.md:9` lists `packages/` and `enterprise/` as needing a rebuild; `libs/` (consumed from `dist/` and `build/`) needs it too, as `.cursor/rules/infrastructure.mdc` says.
- `apps/api/src/ee/` and `apps/dashboard/src/ee/` are listed as enterprise folders; neither exists.
- `.cursor/rules/dashboard.mdc:42` points at `src/components/ui/`; the components are in `src/components/primitives/`.

## langfuse

- `.agents/AGENTS.md:225-226` says every package lints with eslint `--max-warnings 0`. All four run oxlint since the migration in #19613, and `packages/shared` has no eslint dependency.
- `.agents/AGENTS.md` and `ee/AGENTS.md` say `web` (and `worker`) consume `@langfuse/ee`. No `package.json` outside `ee/` depends on it and nothing in `web/src` or `worker/src` imports it.

## documenso

- `AGENTS.md:8` says `npm run test:e2e` runs the E2E tests. The root `package.json` has no such script; it lives in `packages/app-tests`.
- `.cursorrules:18-19`: "prefer types over interfaces", then "use functional components with TypeScript interfaces". The code uses `type` about ten times as often.
- `.cursorrules:47` points to `TeamCreateDialog.tsx`; the file is `team-create-dialog.tsx`.
- `.cursorrules:1` and `:35` describe Remix and Server Actions; the app runs on React Router 7 and has no `'use server'` directives.

## twenty

- `CLAUDE.md:3` says `twenty-front` is on React 18; its `package.json` has `"react": "^19.2.0"`.
- 15 app docs (`AGENTS.md`, `CLAUDE.md`, `AGENT.md`, `LLMS.md`, copied from the app template) list as a pitfall that a view with no `navigationMenuItem` is "**available** on the left sidebar"; the copies in call-recorder and granola say "unavailable".

## excalidraw

- `CLAUDE.md:23` calls `yarn test:update` "Run all tests". It runs vitest with `--update` only; the full check is `yarn test:all`.
- `CLAUDE.md:9` lists four core packages; `packages/` also has `fractional-indexing` and `laser-pointer`.

## formbricks

The audit flagged three lines. We could not confirm any of them by hand (the main one, a `node_modules` path, very likely resolves through hoisting), so none are counted.

## trpc

Clean: 23 of 30 claims checkable, all of them true.

---

Run it on your own repo: [`audit-memory`](../README.md#install). It's free and reads only. We found false alarms too (a symlinked `AGENTS.md` reported as a duplicate, a test command reported as hanging); each was dropped here, and the skill now checks for them.
