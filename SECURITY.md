# Security

These skills run inside your coding agent with its permissions, so we treat every change to them as security-sensitive.

- Every change to `main` goes through a pull request reviewed by a code owner. Force pushes and branch deletion are blocked.
- Releases are git tags (`v0.1.0`, ...). A tag cannot be moved or deleted once it is published.
- The skills are Markdown only, plus two HTML card templates with no scripts and no web fonts: no binaries, no install hooks, no network calls of their own.
- Every skill only reads until you say what to change. They tell the agent never to run the project's own scripts, never to commit, and never to edit a file you did not pick.
- `told-you-twice` reads your own Claude Code and Codex session logs in your home folder. It copies only the messages you typed into a temp folder, keeps them on your machine, never prints anything that looks like a secret, and its share line holds counts and rule titles only.
- `rules-from-reviews` is the only skill that reaches the network, through your own signed-in `gh` CLI, to read your repo's PR reviews.

## Reporting a vulnerability

Please report it privately through GitHub: **Security → Report a vulnerability** on this repo. Do not open a public issue. We reply within 2 working days.
