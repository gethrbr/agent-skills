# Security

These skills run inside your coding agent with its permissions, so we treat every change to them as security-sensitive.

- Every change to `main` goes through a pull request reviewed by a code owner. Force pushes and branch deletion are blocked.
- Releases are git tags (`v0.1.0`, ...). A tag cannot be moved or deleted once it is published.
- The skills are Markdown only: no scripts, no binaries, no install hooks, no network calls of their own.
- `audit-memory` only reads. It tells the agent never to run the project's own scripts and never to edit a file until you say which lines.

## Reporting a vulnerability

Please report it privately through GitHub: **Security → Report a vulnerability** on this repo. Do not open a public issue. We reply within 2 working days.
