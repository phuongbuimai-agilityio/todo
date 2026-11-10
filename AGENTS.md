# AGENTS.md

The rulebook for every agent (and human) working on **Todo**. Read it before you touch an issue or a branch.

Todo is a tiny todo app. It exists to practise an AI agent workflow: Claude Code agents take every ticket through the SDLC below. Keep the app small; keep the process strict.

## Hard rules

1. **At most 3 fix rounds per PR.** After round 3, stop and hand over (`needs-human`).
2. **One ticket per PR.** One issue, one branch, one PR. Don't fold in unrelated changes.
3. **Never push to `main`.** `main` changes only through a release PR the owner approves.

Also: never merge a PR. Merging is the owner's job, and `.claude/settings.json` denies the GitHub MCP's `merge_pull_request`.

## SDLC phases

Every ticket moves through these phases in order. Each has one owner and an exit gate; don't start a phase until the previous gate is met.

| Phase | Owner | Output | Exit gate |
|---|---|---|---|
| Requirements | Analyst (Claude Code) | Issue with the user story and acceptance criteria | Every criterion is testable; issue labelled `ready` |
| Design | Planner (Claude Code) | Plan comment on the issue: files, tests, API or data changes, risks | The plan fits one PR (under 300 changed lines) |
| Build | Implementer (Claude Code) | Branch `feat/<issue>-<slug>` and a PR linked to the issue, with code and tests | Every acceptance criterion has a test |
| Test | CI (GitHub Actions) | Lint, type checks, unit and API tests, build | All checks green |
| Review | Reviewer (Claude Code GitHub Action) | Review comments, each marked **blocking** or **non-blocking** | No blocking findings |
| Release | Release agent (Claude Code), then the owner | Release PR from `sprint/N` to `main` with notes and a test checklist | Owner approves; `main` is tagged `vX.Y` |
| Maintain | Analyst (Claude Code) | One new issue per bug or follow-up, linked to the release | The issue enters Requirements |

Requirements → Build run once per ticket. Release runs once per sprint.

## Fix loop

A **fix round** is one push by the Implementer that answers a failed CI run or a review with blocking findings.

- The Implementer fixes **only** the listed findings, then pushes. CI and the review run again.
- The review workflow labels the PR `fix-round-1`, `fix-round-2`, `fix-round-3`.
- Still failing after round 3: the PR gets `needs-human`, is assigned to the owner, and the review stops. The owner gives hints and allows one more round, sends the ticket back to Design to re-plan or split, or takes it over.
- A re-planned ticket starts again at round 0, in a new PR.
- Non-blocking findings never start a round. Open a follow-up issue (`type: follow-up`) for each instead.

## Branches

| Branch | Use |
|---|---|
| `main` | Protected. Changes only through approved release PRs. |
| `sprint/N` | Collects one sprint's tickets. Feature PRs target it. |
| `feat/<issue>-<slug>` | One ticket, e.g. `feat/1-api-scaffold`. |

Commit messages: imperative, short subject, reference the issue (`Add health endpoint (#1)`).

## Labels

| Label | Meaning |
|---|---|
| `sprint-0`, `sprint-1`, `sprint-2`, `next` | Which sprint the ticket belongs to |
| `type: feature`, `type: chore`, `type: bug`, `type: follow-up` | Kind of work |
| `area: api`, `area: web`, `area: workflow` | Where the change lives |
| `ready` | Requirements done; the Planner can start |
| `fix-round-1..3` | Set by the review workflow |
| `needs-human` | Fix loop exhausted; owner decides |

## GitHub access

- **Code:** local `git` (branch, commit, push).
- **Everything else:** the GitHub MCP from `.mcp.json`. Don't use the `gh` CLI.

| Phase | Tools |
|---|---|
| Requirements | `search_issues`, `create_issue`, `update_issue` (add `ready`) |
| Design | `add_issue_comment` (the plan) |
| Build | `create_pull_request` |
| Fix loop | `pull_request_read`, `list_workflow_runs`, `get_job_logs` |
| Release | `create_pull_request` (release PR to `main`) |

Setup: create a fine-grained personal access token for this repo (Issues, Pull requests, Actions: read/write; Contents: read) and export it as `GITHUB_PAT` before starting Claude Code. Never commit it.

## Requirements and design sources

- Plan (owner only): https://claude.ai/artifact/17UhCg3NV7bjarzmBECKJB — user stories have anchors `#us-01` … `#us-06`.
- Issues carry the story text and acceptance criteria, so the issue is the source of truth for review.

## Stack and coding style

| Layer | Choice | Checks |
|---|---|---|
| API (`api/`) | Python 3.12, FastAPI, SQLAlchemy 2, SQLite via `DATABASE_URL` | Ruff, mypy (strict), pytest + httpx |
| Web (`web/`) | React + Vite + TypeScript (strict), plain CSS | ESLint, `tsc --noEmit`, Vitest + Testing Library, `vite build` |
| E2E | Playwright (T10) | One test of the main flow |

- Write the smallest code that meets the stories: no accounts, no extra services, no UI library.
- Every endpoint has pytest tests, including validation errors. Every component has Vitest tests.
- Every action works by keyboard; inputs and buttons are labelled.
- Design: IBM Plex Sans, accent `#15803D`.

## Running and testing

_Filled in as the scaffolds land._

| What | Command | Added by |
|---|---|---|
| Start the API | TBD | T03 (#1) |
| API tests and checks | TBD | T03 (#1) |
| Start the web app | TBD | T05 (#3) |
| Web tests and checks | TBD | T05 (#3) |
