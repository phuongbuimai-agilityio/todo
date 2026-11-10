---
name: make-plan
description: Plan a software project, feature, release or migration. Asks questions to pick the plan type, gathers requirements, checks facts, then writes the plan as a markdown content file. Use to create or revise a plan. Does not render HTML; that is a separate step the user chooses.
argument-hint: "[what to plan, or the path of an existing plan content file to revise]"
---

# Make plan

Turn an idea, feature, release or migration into a plan the user can share. The request (or the path of a plan to revise) is: $ARGUMENTS

Work in this order:

1. Pick the plan type.
2. Ask that type's questions.
3. Check the facts that go stale.
4. Write the type's sections.
5. Write the plan to a content file and stop there.

This skill decides what the plan says and works on its own. It does not render HTML and does not call any other skill. If the user wants a styled page, they will ask for it themselves.

## 1. Pick the plan type (always first)

Ask one AskUserQuestion round with these four questions. Skip any the request already answers, and don't ask anything you can infer.

1. **Plan type.** Show the 4 types that fit the request best, with the most likely one first and marked Recommended. The user can type another.
2. **Audience:** just me / my team / stakeholders or management.
3. **Depth:** brief (core sections only) / full plan.
4. **Who builds it:** AI agents (Claude Code) / a human team / both.

| Plan type | Use when | Page title |
|---|---|---|
| Requirements & delivery | A new app or product: what to build and how to ship it | [Name] — Requirements & Delivery Plan |
| Technical design | How to build one feature or system, and why this way | [Name] — Technical Design |
| Release / sprint plan | Scheduling an existing backlog into sprints and a release | [Name] — Release Plan |
| Migration / upgrade | Moving from one stack, host, database or version to another | [Name] — Migration Plan |
| Discovery brief | An early idea with many unknowns, before committing to build | [Name] — Discovery Brief |

How the answers shape the plan:

- **Audience.**
  - Just me: terse, with technical detail.
  - Team: adds conventions and owners by role.
  - Stakeholders: leads with outcomes, dates and risks, and keeps technical tables shorter.
- **Depth.** Brief writes only the type's core sections. Full adds the rest.
- **Who builds it:** picks the delivery workflow variant in section 4.

## 2. The type's questions

Ask in rounds of at most 4 questions, each with 2–4 options and the recommended one first. When the user replies with a short label, take it as the decision. Stop asking once the core sections can be written; whatever is still unknown becomes an open question in the page.

- **Requirements & delivery:** purpose; platform (web, mobile, both); users and accounts; MVP features (propose a list to trim); stack; repo, hosting and budget; the user's approval role; UI languages; deadline; existing design.
- **Technical design:** the feature or system and where its code lives; current state; goals and non-goals; constraints (performance, cost, compatibility); alternatives already considered; rollout limits (downtime, feature flags); deadline.
- **Release / sprint plan:** release goal and date; where the backlog lives; sprint length; who is available (never invent capacity); must-haves vs nice-to-haves; dependencies; who approves the release.
- **Migration / upgrade:** from what to what; why now; what moves (an inventory); downtime tolerance; data volume (ask, never guess); rollback requirement; deadline; who verifies.
- **Discovery brief:** the problem and who has it; how it's solved today; what success would look like; the biggest unknowns; time and budget limits; the decision this brief should enable.

## 3. Check facts that go stale

Run a web search (WebSearch / WebFetch) before writing about anything that changes over time: hosting limits, free tiers, pricing, product features, versions, MCP servers, GitHub Actions. List what you used in the page's Sources section.

Look for **conflicts between the user's choices** and raise them before you build. For example:
- SQLite can't persist on serverless hosts such as Vercel.
- A free plan may not allow commercial use.
- An MCP server may lack an operation the workflow needs, so that step stays manual.

When a conflict changes cost or scope, ask the user to choose.

## 4. Sections per type

"Core" sections are always written. "Full" sections are added when the user picks a full plan. Every page ends with Risks & open questions and Sources.

| Type | Core sections | Full adds |
|---|---|---|
| Requirements & delivery | Summary · Decisions & scope · User stories (US-NN with testable criteria) · Sprint plan · Backlog (one ticket = one issue + one PR) | Non-functional requirements · Architecture (diagram + stack table) · Data, API & screens · Design links · Delivery workflow |
| Technical design | Summary · Goals & non-goals · Proposed design (diagram) | Context & current state · Alternatives (comparison table + the recommendation) · Data & API changes · Rollout · Testing & observability |
| Release / sprint plan | Summary (goal + date) · Scope in/out · Sprint list · Backlog | Dependencies (diagram) · Release checklist · Delivery workflow · Communication |
| Migration / upgrade | Summary · Current vs target (before/after diagram) · Phases with gates · Rollback plan | Inventory · Data migration steps · Verification · Communication |
| Discovery brief | Problem · Users · Goals & success signals (no invented numbers) · Assumptions to test · Next steps | Options compared · Rough phases |

**Delivery workflow variants** (picked by "who builds it"):

- **AI agents:**
  - SDLC phases table (Phase · Agent and where it runs · Output · Exit gate): Requirements → Design → Build → Test → Review → Release → Maintain.
  - Fix loop: at most N rounds after review, then a `needs-human` label and the user decides. Ask for N; the default is 3.
  - Tool access per phase, e.g. GitHub MCP tools.
  - Branches, and a rules file (`AGENTS.md`).
  - Who merges: always the human.
- **Human team:** phases with owners by role, the definition of done, review rules, and the release approval.
- **Both:** humans own requirements, review approval and release; agents own build and test, with the fix-loop limit.

## 5. Write the content file

1. **Write `docs/plans/<name>-plan.md`** in the repo (create the folder if needed), in plain markdown: one `#` title, one `##` per section from step 4 for the chosen type, using tables, lists, code blocks and `- [ ]` checklists.
   - Write the final wording; the file must stand on its own, with everything a reader (or a later renderer) needs.
   - Use the page title from the type table.
   - Open with a one-line description and the key facts (counts, dates, links to designs).
2. **Check the file before handing it off.**
   - Counts and dates match everywhere they appear.
   - Any diagram names its boxes and arrows.
   - Every source has a URL.
3. **Hand off in chat:**
   - The content file's path.
   - One line on what the plan holds.
   - Anything you assumed.
   - Anything that moved dates or scope.
   - At most one open question that blocks the next step.
4. **Keep the content file as the source of truth.** Later revisions start from it. Don't commit, push or open PRs for the plan unless the user asks; if the repo has an `AGENTS.md`, follow its branch rules when they do.
5. **Don't render or open a page.** Don't invoke artifact-page-style or any other skill unless the user asks.

## 6. Revising a plan

- **Update every place a decision touches:** decisions, stories or goals, architecture, workflow, schedule, backlog, risks, sources, and the key facts at the top. Afterwards, search the plan for words from the old decision to catch leftovers.
- **Remove cleanly.** When the user removes something, delete its section, tickets, risks and sources, and fix the counts.
- **Dates.** If a change moves the schedule, ask, or make the call and say so in one line.
- **Source of truth.** Edit the content file. If an HTML page or an uploaded copy (e.g. on claude.ai) was made from it earlier, tell the user it is now out of date; they decide whether to re-render.
- **Changing the plan type** (e.g. a discovery brief growing into a requirements & delivery plan) means writing a new content file from the new type's sections, reusing what still holds.