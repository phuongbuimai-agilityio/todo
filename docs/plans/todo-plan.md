# Todo — Requirements & Delivery Plan

A deliberately small todo app (one list, no accounts, a SQLite file on your machine) built as the practice ground for an AI agent workflow: Claude Code agents take every ticket through the SDLC, from issue to release.

**Key facts:** 6 user stories · 11 tickets, each one issue and one PR · 3 sprints to v1.0 by 28 November 2026, then T11 deploy · React + Vite + TypeScript web app, FastAPI API, SQLite → Turso on Vercel · no login · built by Claude Code agents, reviewed by the Claude Code GitHub Action · GitHub access through the GitHub MCP only · at most 3 fix rounds per PR.

**Links:** [Design canvas](https://claude.ai/artifact/35nWFrGX4WXsLruzVVJVji) · [Editable doc](https://claude.ai/artifact/H9DJHtQAA9ExfcgKNa34FU) · [Plan artifact](https://claude.ai/artifact/17UhCg3NV7bjarzmBECKJB) · [Repo](https://github.com/phuongbuimai-agilityio/todo)

*Requirements & delivery plan · updated 10 November 2026*

## Summary

The app is the vehicle; the agent workflow is the product. Keep the app tiny (title + done, SQLite, no login) so every one of the 11 tickets runs the full SDLC loop: requirements → design → build → test → review → release, with at most 3 fix rounds before you decide.

- **What v1 does** — One list. Add, edit, complete and delete todos in the browser; they survive a restart because they live in a SQLite file.
- **Where it runs** — On your machine during v1 (data in `todo.db`), then on Vercel with Turso as the hosted SQLite (T11). GitHub holds the code, issues, PRs and CI.
- **v1 is done when** — The app works and keeps data after a restart; every ticket went through the full agent loop; a fresh agent can pick up the next ticket from README + `AGENTS.md`.

## Decisions and scope

v1 is one shared list of todos, each with a title and a done checkbox, stored in SQLite and run locally.

| Topic | Decision |
|---|---|
| Name | Todo |
| Purpose | **Practice building an AI agent workflow; the app stays deliberately small** |
| Accounts | None: no sign-up, login or per-user data |
| Database | SQLite: a local file (`todo.db`) in development, Turso (hosted SQLite) in production |
| Hosting | Local during v1; Vercel right after (T11) |
| Agent tools | **Claude Code sessions run the Analyst, Planner, Implementer and Release roles; the Claude Code GitHub Action reviews every PR** |
| Language | English only |
| Todo fields | Title and done |
| Stack | React + Vite (TypeScript) web app, FastAPI (Python) API |
| Workflow | **SDLC phases per ticket; at most 3 fix rounds after review, then escalate to you** |
| GitHub access | **Only through the GitHub MCP (git for local code); no `gh` CLI** |
| Branch protection | None: the repo is private on GitHub Free, where it isn't enforced. The merge-deny rule and `AGENTS.md` guard `main` (decided 10 November 2026) |
| Your role | Approve and merge the release PR at the end of each sprint |

**In v1:** see all todos, newest first · add a todo · edit a todo's title · mark a todo done or not done · delete a todo · keep todos after restarting the app.

**Out of scope:** accounts and login · multiple lists, due dates, priorities, search · Vietnamese, a mobile app · Vercel hosting (comes right after v1, as T11).

## User stories and acceptance criteria

Six stories cover v1. The Reviewer agent checks every PR against the criteria of the stories it touches.

### US-01 See my todos

As a user, I see all todos when I open the app.

- Todos show newest first, each with a checkbox and its title.
- An empty list shows a short "No todos yet" message.

### US-02 Add a todo

As a user, I type a title and press Enter or click Add.

- Title required: 1–200 characters after trimming; otherwise an inline error.
- The new todo appears at the top without a reload, and the input clears.

### US-03 Edit a todo

As a user, I click a title, change it and press Enter.

- The same title rules apply as when adding.
- Esc cancels the edit.

### US-04 Complete a todo

As a user, one click on the checkbox marks a todo done or not done.

- Done todos show struck through.

### US-05 Delete a todo

As a user, I remove a todo with its delete button.

- It disappears at once.
- Deleting a todo that is already gone shows no error.

### US-06 Keep my data

As a user, my todos are still there after I stop and restart the app.

- Data lives in a SQLite file whose path comes from `DATABASE_URL` (default `./todo.db`).

## Non-functional requirements

These apply to every ticket and are checked by CI or the Reviewer agent.

| Area | Requirement |
|---|---|
| Simplicity | **The smallest code that meets the stories: no accounts, no extra services** |
| Quality | Every PR passes lint, type checks, tests and a build in CI |
| Tests | API: pytest for every endpoint, including validation errors. Web: Vitest for components. One Playwright test for the main flow |
| Accessibility | Every action works by keyboard; the input and buttons are labelled |
| Running locally | One command each starts the API and the web app, documented in the README |
| Data | The SQLite file is git-ignored; the table is created on startup |

## Architecture and tech stack

Everything runs on your machine: a React web app talks to a FastAPI API, which keeps todos in one SQLite file.

```
Your machine (localhost)
+---------------------------------------------------------------------+
|                                                                     |
|  Browser  -->  Web app           -->  API             -->  SQLite   |
|  one page      React + Vite + TS      FastAPI              todo.db  |
|  no login      proxies /api/*         Python 3.12          table:   |
|                                       SQLAlchemy 2         todos    |
|                                                                     |
+---------------------------------------------------------------------+
                                 ^
                                 |  git pull, then run locally
                                 |
                   GitHub repo (issues, PRs, CI checks)
                        where the agents work
```

Boxes: Browser, Web app and API are the three processes; SQLite is the `todo.db` file. Arrows: Browser → Web app → API → SQLite carry each request; GitHub repo → your machine is the `git pull` you run before working locally. Vite's dev server forwards `/api/*` to FastAPI, so no CORS setup is needed. On Vercel (T11), the same API reads `DATABASE_URL` pointing at Turso instead of a local file.

| Layer | Choice | Why |
|---|---|---|
| Web app | React + Vite + TypeScript, plain CSS | Lightest TypeScript setup for one page; no design phase needed |
| API | FastAPI on Python 3.12 | Typed, quick to write, automatic OpenAPI docs |
| Database | **SQLite through SQLAlchemy 2 locally; Turso through sqlalchemy-libsql on Vercel; table created on startup** | One file, nothing to install |
| Tests | pytest + httpx, Vitest + Testing Library, Playwright | Covers the API, components and the main flow |
| Code quality | Ruff + mypy, ESLint + tsc | Strict checks that agents and humans follow |
| CI | GitHub Actions | Runs every check on every PR |

## Data model, API and screen

One table, five endpoints and one screen cover v1.

| Column | Type | Rule |
|---|---|---|
| `id` | integer | Primary key |
| `title` | text | Required, 1–200 characters |
| `done` | boolean | Default false |
| `created_at` | datetime | Set on create |
| `updated_at` | datetime | Set on every change |

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/health` | Health check |
| GET | `/api/todos` | List todos, newest first |
| POST | `/api/todos` | Create a todo from a title; returns 201 and the todo |
| PATCH | `/api/todos/{id}` | Change the title, done, or both |
| DELETE | `/api/todos/{id}` | Delete a todo; returns 204 |

An unknown id returns 404, and an invalid title returns 422 with the reason.

**The one screen:** a header, an input with an Add button, and the list. Each row has a checkbox, the title (click to edit) and a delete button. Empty, loading and error states each get one short message. See the [design](https://claude.ai/artifact/35nWFrGX4WXsLruzVVJVji).

## Web design

The design canvas holds three artboards; the main one is a working prototype you can click through.

- **Web app · interactive** — Add, edit (Enter to save, Esc to cancel), tick done and delete, with inline validation. Press Play to try it.
- **Phone · 390 px** — The same screen at phone width, with a row in edit mode and 44 px touch targets.
- **States** — Empty list, invalid title (add and edit), and the error shown when the API is down.

[Open the design canvas](https://claude.ai/artifact/35nWFrGX4WXsLruzVVJVji) · IBM Plex Sans, a green accent (`#15803D`) that passes 4.5:1 contrast with white text, plain CSS so T06–T09 can copy the styles directly.

## AI agent workflow (SDLC)

Every ticket moves through the same SDLC phases, each owned by one agent with a clear exit gate. After review, a ticket gets at most 3 fix rounds; then it stops and you decide.

```
Requirements --> Design --> Build -------> Test ----> Review ----> Release
 (Analyst)     (Planner)  (Implementer)    (CI)    (GH Action)   (you approve
                             ^   ^          |          |           and merge)
                             |   |          |          |                |
   blocking findings:        |   +----------+          |                |
   fix round 1, 2 or 3 ------+   CI fails:             |                |
                                 same rounds           |                v
                                                       |            Maintain
                                                       v           (bugs become
                                                   Escalate           issues)
                                                 needs-human,            |
                                                  you decide             |
                                              (after round 3)            |
                                                                         v
                                      Requirements <--- new issue -------+
```

Boxes: Requirements, Design, Build, Test, Review, Release, Maintain and Escalate. Arrows: the main path runs left to right; Review → Build and Test → Build are the fix rounds; Review → Escalate fires when a PR is still failing after round 3; Release → Maintain runs after a release, and Maintain → Requirements opens the new issue. Requirements, Design, Build, Test and Review run once per ticket. Release runs once per sprint, and Maintain turns bugs found after a release into new issues.

| Phase | Agent (runs in) | Output | Exit gate |
|---|---|---|---|
| Requirements | Analyst (Claude Code) | Issue with the user story and acceptance criteria | Every criterion is testable; issue labelled `ready` |
| Design | Planner (Claude Code) | Plan comment: files, tests, API or data changes, risks | The plan fits one PR (under 300 changed lines) |
| Build | Implementer (Claude Code) | Feature branch and PR linked to the issue, with code and tests | Every criterion has a test |
| Test | CI (GitHub Actions) | Lint, type checks, unit and API tests, build | All checks green |
| Review | **Reviewer (Claude Code GitHub Action)** | Review comments, each marked blocking or non-blocking | No blocking findings |
| Release | Release agent (Claude Code), then you | Release PR to `main` with notes and a test checklist | **You approve and merge; `main` is tagged vX.Y** |
| Maintain | Analyst (Claude Code) | One new issue per bug or follow-up, linked to the release | The issue enters Requirements |

### Fix loop: at most 3 rounds

A fix round is one push by the Implementer in answer to a failed CI run or a review with blocking findings. Non-blocking findings never start a round; they become follow-up issues.

Round 1 → Round 2 → Round 3 → `needs-human`, you decide.

**Rounds 1 to 3 · agents**

1. The Implementer fixes only the listed findings and pushes.
2. CI and the review run again.
3. The review workflow labels the PR `fix-round-1`, `-2` or `-3`.

**Still failing after round 3 · you**

1. The PR is labelled `needs-human`, the review stops running, and it is assigned to you.
2. You give hints and allow one more round, send it back to Design to re-plan or split it, or take it over.
3. A re-planned ticket starts again at round 0, in a new PR.

**Reviewer · Claude Code GitHub Action.** The Reviewer runs in GitHub, separate from the session that wrote the code, so it never grades its own work. Set it up with `/install-github-app` in Claude Code: it installs the Claude GitHub App, adds the API key as a repo secret and adds the workflow file (`anthropics/claude-code-action@v1`). Its prompt points at the PR's linked issue and `AGENTS.md`, and asks it to mark each finding blocking or non-blocking.

### GitHub access: GitHub MCP only

Local git handles code (branches, commits, push). Claude Code sessions do everything else on GitHub through the GitHub MCP, connected in the repo's `.mcp.json` with your token from the `GITHUB_PAT` env var. There's no `gh` CLI.

| Phase | GitHub MCP toolset · tools |
|---|---|
| Setup (T01) | labels · `create_label` |
| Requirements | issues · `search_issues`, `create_issue`, `update_issue` (label `ready`) |
| Design | issues · `add_issue_comment` (the plan) |
| Build | pull_requests · `create_pull_request` |
| Fix loop | pull_requests · `pull_request_read`; actions · `list_workflow_runs`, `get_job_logs` |
| Release | pull_requests · `create_pull_request` (release PR to `main`) |

**Merging stays yours.** A permission rule in `.claude/settings.json` denies the GitHub MCP's `merge_pull_request`, so only you merge. There is no branch protection on `main` (private repo on GitHub Free), so this rule and the hard rules in `AGENTS.md` are the guard. Inside GitHub Actions, the review action uses its own GitHub access, not your MCP connection.

### Branches

| Branch | Use |
|---|---|
| `main` | Changes only through release PRs you approve and merge. Not technically protected. |
| `sprint/N` | Collects a sprint's tickets. |
| `feat/<issue>-<slug>` | One ticket. |

**`AGENTS.md` · the rulebook every agent reads.** How to run the app and the tests, the coding style, the PR template, the SDLC phases and their exit gates, and three hard rules: **at most 3 fix rounds per PR**, **one ticket per PR**, and **never push to `main`**.

## Sprint plan

A setup sprint gets the agent workflow running first, then two app sprints reach v1.0 on 28 November 2026. Each sprint ends with a release PR for you to approve.

| Sprint | Focus | Dates | Scope | Gate |
|---|---|---|---|---|
| Sprint 0 · T01–T02 | Workflow setup | 10–14 Nov 2026 | Repo, `AGENTS.md`, GitHub MCP, labels, CI and the review action; one dry run of the full loop. | You approve release |
| Sprint 1 · T03–T06 | API and list view | 17–21 Nov 2026 | SQLite todos API, web scaffold, and the todo list on screen. | You approve release |
| Sprint 2 · T07–T10 | Finish and release | 24–28 Nov 2026 | Add, edit, done, delete; end-to-end test; v1.0. | You approve: **v1.0** |
| Next · T11 | Deploy | from 1 Dec 2026 | Vercel with Turso as the hosted SQLite. | You approve |

Sprint 0's dry run takes one throwaway issue through every phase, including a forced review failure, to check the 3-round limit before real tickets start.

## Backlog

11 tickets, each one GitHub issue and one PR.

| Sprint | ID | Ticket | Stories |
|---|---|---|---|
| Sprint 0 · Workflow setup | T01 | Repo: `api/` and `web/` folders, README, `AGENTS.md` with the SDLC phases and rules, issue and PR templates with a phase checklist, `.mcp.json` for the GitHub MCP, labels, the merge-deny permission rule | — |
| Sprint 0 · Workflow setup | T02 | CI (Ruff, mypy, pytest, ESLint, tsc, Vitest, build) and the Claude Code review workflow that counts fix rounds and escalates after 3; dry run of the full loop on a test issue | — |
| Sprint 1 · API and list view | T03 | API scaffold: FastAPI, SQLAlchemy, SQLite file from `DATABASE_URL`, `todos` table, `/api/health` | US-06 |
| Sprint 1 · API and list view | T04 | Todos API: list, create, update and delete with validation and pytest tests | US-01 to US-06 |
| Sprint 1 · API and list view | T05 | Web scaffold: React + Vite + TypeScript, `/api` proxy, typed API client | — |
| Sprint 1 · API and list view | T06 | Todo list view with empty, loading and error states | US-01 |
| Sprint 2 · Finish and release | T07 | Add-todo input with validation | US-02 |
| Sprint 2 · Finish and release | T08 | Inline title editing and the done checkbox | US-03, US-04 |
| Sprint 2 · Finish and release | T09 | Delete button | US-05 |
| Sprint 2 · Finish and release | T10 | Playwright end-to-end test, README run guide, v1.0 release | US-06 |
| Next · Deploy | T11 | Deploy to Vercel: Turso database, `DATABASE_URL` and token as Vercel env vars, preview deploys on PRs | US-06 |

## Risks and open questions

With the app this small, the main risks sit in the agent workflow, not the code.

| Risk | Impact | Mitigation |
|---|---|---|
| Agents drift from the repo's conventions | Inconsistent code, failed reviews | `AGENTS.md`, small tickets, CI as the gatekeeper |
| The GitHub MCP lacks an operation | A step can't be automated | Do it by hand once and record it in `AGENTS.md`; no `gh` fallback |
| No branch protection on `main` | A direct push or merge to `main` isn't blocked by GitHub | Hard rules in `AGENTS.md`; `merge_pull_request` denied in `.claude/settings.json`; you merge release PRs |
| The fix loop never converges | Agents burn time and API usage on one PR | Hard limit of 3 fix rounds, then `needs-human` and your decision |
| The review action approves too easily | Bugs slip through | Review prompt tied to the issue's acceptance criteria; CI must also pass |
| Vercel cannot keep a SQLite file ([Vercel guide](https://vercel.com/kb/guide/is-sqlite-supported-in-vercel)) | Data would vanish between requests | Turso in production; only `DATABASE_URL` changes |
| The Turso SQLAlchemy dialect is marked experimental | T11 may hit driver issues | Spike it early in T11; fallback is Neon Postgres on Vercel |
| Review runs cost API usage | Small monthly bill | Run the review once per PR open and on request, not on every push |

**Open questions:** none. Agent tools, hosting and the name were settled on 9 November 2026; branch protection was dropped on 10 November 2026.

## Sources

- [Todo — Web Design](https://claude.ai/artifact/35nWFrGX4WXsLruzVVJVji): design canvas with the interactive web screen, phone view and states
- [GitHub MCP server](https://github.com/github/github-mcp-server): toolsets (issues, pull_requests, actions, labels…)
- [Claude Code GitHub Actions](https://code.claude.com/docs/en/github-actions): setup with `/install-github-app`; PR review workflows
- [Is SQLite supported in Vercel?](https://vercel.com/kb/guide/is-sqlite-supported-in-vercel): Vercel functions have no persistent file system
- [Turso and Vercel](https://docs.turso.tech/integrations/vercel): hosted SQLite-compatible database for serverless
- [sqlalchemy-libsql](https://pypi.org/project/sqlalchemy-libsql): Turso's SQLAlchemy dialect (experimental)
- [About protected branches](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches): protected branches on GitHub Free are for public repositories only
