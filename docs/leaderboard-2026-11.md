# Agent rules leaderboard, November 2026

`rules-score` on the agent rules (`CLAUDE.md`, `AGENTS.md`, Cursor and Copilot rules) of 20 popular open-source repos, scored on 2026-11-13 at the commit shown. Two runs of the skill on the same commit land within about 3 points; most of that drift is in Specific and No noise. Every wrong-path deduction below was checked against the repo by hand.

Scoped rules files (in subfolders) count for Accurate, so a dead path in `extensions/foo/AGENTS.md` costs as much as one in the root file: the agent loads both.

| # | Repo | Commit | Score | Accurate /40 | Essentials /25 | Specific /15 | Lean /10 | No noise /10 | Biggest loss |
| -: | :- | :- | -: | -: | -: | -: | -: | -: | :- |
| 1 | getsentry/sentry | `5565711b` | 89 | 32 | 25 | 14 | 10 | 8 | `static/AGENTS.md` points images at `static/app/images`, which does not exist |
| 2 | twentyhq/twenty | `f92e6d41` | 88 | 31 | 25 | 14 | 10 | 8 | Two app rules files disagree on one pitfall; says React 18, the repo is on 19 |
| 3 | nuxt/nuxt | `850bce0` | 88 | 40 | 20 | 12 | 10 | 6 | No way to run a single test file |
| 4 | n8n-io/n8n | `d919f0f1` | 86 | 40 | 25 | 14 | 5 | 2 | 24.6 KB loaded every session; generic and repeated rules |
| 5 | tldraw/tldraw | `b16ccbe` | 84 | 40 | 25 | 11 | 8 | 0 | Generic rules; sentence case stated 4 times |
| 6 | vercel/next.js | `6f2463fa` | 83 | 40 | 25 | 13 | 5 | 0 | 31.5 KB loaded every session; 5 rules stated twice |
| 7 | trpc/trpc | `e2e07e7` | 82 | 32 | 15 | 15 | 10 | 10 | Points at `next/` on the v12 branch, renamed to `v12/` |
| 8 | browser-use/browser-use | `48d9fe0` | 80 | 35 | 20 | 15 | 10 | 0 | `CLAUDE.md` is a full copy of `AGENTS.md` instead of importing it |
| 9 | supabase/supabase | `d374842f` | 80 | 36 | 15 | 13 | 8 | 8 | No install step or one-test command; Node version out of date |
| 10 | grafana/grafana | `5eb1773a` | 78 | 32 | 25 | 13 | 8 | 0 | Alerting `AGENTS.md` lists a `templates/` folder that does not exist |
| 11 | sveltejs/svelte | `0156c5f` | 75 | 40 | 0 | 15 | 10 | 10 | Setup, build, test and lint live only in a linked `CONTRIBUTING.md` |
| 12 | astral-sh/uv | `9b4fe90` | 74 | 40 | 0 | 14 | 10 | 10 | All five essentials live only in `CONTRIBUTING.md` |
| 13 | withastro/astro | `f9a1062` | 73 | 32 | 20 | 11 | 8 | 2 | `errors-data.ts` path is missing `src/internal/` |
| 14 | denoland/deno | `e7b39ebd` | 72 | 32 | 25 | 10 | 5 | 0 | `cargo test spec::` matches nothing; the tests are `specs::` |
| 15 | excalidraw/excalidraw | `82d5847` | 66 | 35 | 10 | 11 | 10 | 0 | No install, run or one-test commands; `CLAUDE.md` skips `AGENTS.md` |
| 16 | storybookjs/storybook | `631be15` | 53 | 16 | 15 | 12 | 10 | 0 | `.cursorrules` names a deleted `vitest.workspace.ts`, a missing test script and Cursor tasks that do not exist |
| 17 | microsoft/vscode | `6f2eddfc` | 52 | 8 | 25 | 11 | 8 | 0 | 4 dead paths in the Copilot extension and agent host rules |
| 18 | zed-industries/zed | `48741fd` | 44 | 16 | 5 | 11 | 8 | 4 | Crash prompt files deleted in October; `script/crash-to-prompt` never existed |
| 19 | oven-sh/bun | `f84773225` | 39 | 0 | 20 | 14 | 5 | 0 | `src/shell/`, `src/bake/`, `src/deps/` and `websocket_client/` moved; `pr::comments` is `pr:comments` |
| 20 | calcom/cal.com | `ef7954d` | 35 | 7 | 20 | 8 | 0 | 0 | 41 rules files (~102 KB, ~25k tokens) load every session; 3 dead paths |

Score your own: `npx skills add gethrbr/deckhand`, then ask your agent to *score our agent rules*. If a score here is wrong, open an issue with the line and we will re-check it.
