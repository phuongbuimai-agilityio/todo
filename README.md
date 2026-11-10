# Todo

A tiny todo app: add, edit, complete and delete todos. No accounts, one page, one SQLite file.

Todo is also a practice ground for an **AI agent workflow**: Claude Code agents take every ticket through requirements, design, build, test, review and release. See [AGENTS.md](AGENTS.md) for the rules.

## Layout

| Path | What |
|---|---|
| `api/` | FastAPI + SQLAlchemy 2 backend (Python 3.12) |
| `web/` | React + Vite + TypeScript frontend |
| `AGENTS.md` | SDLC phases, fix loop, branches, hard rules |
| `.github/` | Issue and PR templates (CI and review workflows arrive in T02) |
| `.mcp.json` | GitHub MCP connection for Claude Code |

## Running locally

Coming with the scaffolds: the API in T03 (#1), the web app in T05 (#3). Each will start with one command.

## Roadmap

- **Sprint 0** (10–14 Nov): workflow setup
- **Sprint 1** (17–21 Nov): API and list view
- **Sprint 2** (24–28 Nov): add, edit, done, delete, end-to-end test → **v1.0**
- **Next** (from 1 Dec): deploy to Vercel with Turso
