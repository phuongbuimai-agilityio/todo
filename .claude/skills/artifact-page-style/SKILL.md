---
name: artifact-page-style
description: House style and renderer for shareable HTML pages (plans, designs, reports, notes). Turns a markdown content file into one self-contained styled HTML page saved next to it. Use when the user asks for a page that looks like their other plan pages. Works on any markdown content file; does not depend on any other skill.
argument-hint: "<path to content file, e.g. docs/plans/todo-plan.md>"
---

# Artifact page style

This skill holds the user's house style for HTML pages. It is a calm, documentation-like look: IBM Plex Sans and Mono, a green accent, a sticky table of contents on the left, one reading column, and automatic dark mode.

Use it whenever a page should look like the user's other pages: plans, technical designs, study notes, reports and summaries. It only decides how a page looks and where the HTML file is written. What goes in the page comes from a markdown content file the user points to (by any author).

## Input: the content file

This skill runs as a separate helper on a lighter model (`context: fork`). The helper starts without the conversation, so everything comes from the **content file** whose path is passed as the argument: `$ARGUMENTS`. It is only run when the user asks for it.

1. **Read the content file.** If no path was given, stop and reply that a content file path is needed (the helper can't see the conversation).
2. **Render exactly what the file says.**
   - Don't rewrite, add or drop content. Layout choices are yours; the words are not.
   - Use the `title` and `page` fields from the file's header (`icon` and `artifact` are ignored locally).
3. **Report problems instead of fixing them.** If something can't be rendered as written, still write what works and list the problem in your reply. Examples: a count that disagrees with another, a diagram that names a box twice, a source without a URL.

### How the file maps to components

| In the content file | On the page |
|---|---|
| Header `title`, `kind`, `updated` | `<title>`, the `.label` kicker ("[kind] · updated [date]") and the h1 (the file's `#` line) |
| Header `pills`, `links` (split on `\|`) | Header `.pill`s; links become `a.pill` with ↗ |
| Line under `#` | The `.muted` description under the h1 |
| `## Title {#id group="…"}` | `<section id>` with `p.label` = group, h2, a contents link under that group, and an `hr.divider` before it |
| First paragraph or `> [!gist]` | `p.gist` (in Summary: the large "In one line" gist) |
| `- card: Title \| text` | `.grid3` (3 cards) or `.grid2` (2 or 4 cards) of `.card` |
| `### ID Title` + "As a …" + bullets | `.stories > .story` (`span.id`, h4, `p.as`, `.list`) |
| `- sprint: …`; `(last)` | `.sprints > .sprint`; `(last)` gets `.last` and the ◆ gate, others get ◇ |
| `filter` + a table whose first column is `Group` | `.filter` buttons (All + each group) and `table#backlog-tbl` with a `tr.mod` row per group; the Group column itself is not shown |
| `[Must]` `[Should]` `[Later]`; `==text==` | `span.badge.must/.should/.later`; `td.hi` |
| `flow: A -> B -> C*` | `.flow`; `*` = `.s.on` |
| `layers:` lines | `.layers > .layer`, the first one `.top` |
| `> [!rule]` / `[!do]` / `[!example]` / `[!warn]` `Tag \| text` | `.co-def` / `.co-do` / `.co-ex` / `.co-warn` with `span.tag` |
| A `diagram` code block | `figure.dg-wrap > svg.dg`: `row` boxes left to right, `highlight` = `.box.ag`, `warn` = `.box.warn`, labelled `arrows`, `caption` = figcaption, `title` = aria-label |
| Other tables, lists, code blocks, `- [ ]` items | `.tbl`, `.list`/`.olist`, `pre > code`, `ul.checks` |
| `## Sources` links | `.list` of `a.link` |

## Reply

When the page is written, reply briefly with:
- the HTML file's path (relative to the repo root)
- whether it is a new file or an overwrite of an existing one
- any problems from step 3

Don't repeat the page's content.

## Page rules

- **One self-contained HTML file.** Keep the `<style>` and `<script>` from the template exactly as they are. Only Google Fonts loads from outside the page; there are no remote images (draw in inline SVG instead).
- **Sections.** Every `<section>` has an `id`, and the table of contents links to every one of them. Group the contents links under small headings (for example Requirements, Design, Delivery, Wrap-up).
- **Header.** A `.label` kicker line ("[Kind of page] · updated [date]"), an h1, a one-line description in `.muted`, then pills for the key facts and links to related artifacts.
- **Spacing.** Put an `hr.divider` between top-level sections.
- **Before writing the file,** check that every tag is closed, every contents link has a matching section, and any number that appears in several places (counts, dates) is the same everywhere.

## Writing rules

- Every section opens with a `p.gist` sentence that states the point, not the topic.
- Use plain words, short sentences and absolute dates.
- Never invent numbers, names or facts. Anything missing becomes an open question.
- Bold carries the key fact; don't bold whole sentences.
- Facts that change over time (prices, limits, versions) come from a source you searched, listed in a Sources section at the end.

## Components

| Need | Use |
|---|---|
| Lead sentence of a section | `p.gist` |
| Small uppercase kicker above an h2 | `p.label` |
| 2–3 parallel facts | `.grid3` / `.grid2` of `.card` (h3 + p or `.list`) |
| Definition, rule or key point | `.co-def` + `span.tag` |
| Steps or "do" list | `.co-do` |
| Example or note | `.co-ex` |
| Warning, out of scope, human decision | `.co-warn` |
| Tables | `.tbl-wrap > table.tbl`. `tr.mod` = group row, `td.hi` = highlight, `td.mono` = ids and paths, `td.num` = numbers |
| Priority or status | `span.badge.must` / `.should` / `.later` |
| Short linear flow | `.flow` with `.s` steps, `.arr` arrows; `.s.on` = the end state |
| Stacked layers (branches, tiers, environments) | `.layers > .layer` (`.top` = the main one) |
| Item cards (stories, requirements, options) | `.stories > .story` with `span.id`, `h4`, `p.as`, `.list` |
| Timeline or phase cards (sprints, phases) | `.sprints > .sprint` with `.label`, h3, `.dates`, p, `.gate`; `.last` = the milestone |
| Filterable table | `.filter` buttons with `data-f` + `table#backlog-tbl` rows with `data-s` (the script handles it) |
| Checklist or open questions | `ul.checks` |
| Code or a file tree | `pre > code` |
| Diagram | `figure.dg-wrap > svg.dg` + `figcaption.legend` |

## Diagrams

- **Frame:** inline SVG, `viewBox="0 0 760 H"`, where H is the content height plus about 24.
- **Boxes:** `rect.box` with `rx=8`, 128–160 wide and 56–64 tall. `.ag` marks the highlighted box (the goal or the human gate); `.warn` marks an escalation or risk. Group boxes inside a `rect.panel` with a `text.cap` label.
- **Text:** `text.t` for the name (centred) and `text.s` for detail lines (`.s.l` for left-aligned edge labels).
  - Text doesn't wrap, so measure each line: at 11.5px a line holds about (box width − 24) / 6.2 characters, and 12.5px bold uses about 7.4 px per character.
- **Arrows:** `path.ln`, with `.ln.m` for the main path, plus the `#ah` / `#ahm` markers.
  - Run arrows in the gaps between boxes, never through a box or a label. Reach a far target with an elbow path.
- **Accessibility and reading:** give the SVG `role="img"` and an `aria-label` that states what the diagram shows. Put one sentence of reading in the figcaption.

## Write and revise

- **Where the page goes.** Use the header's `page` field as the output path when it is set. Otherwise write the HTML next to the content file with the same name and an `.html` extension (`docs/plans/todo-plan.md` → `docs/plans/todo-plan.html`).
- **Revising.** Overwrite the same HTML file. The content file is the source of truth; never patch the HTML by hand.
- **Check it.** After writing, run a quick parse check, e.g. `python3 -c "import html.parser,sys; html.parser.HTMLParser().feed(open(sys.argv[1]).read())" <file>`, and confirm every contents link has a matching `<section id>`.
- **Viewing.** The page is a plain file: open it in a browser (`open <file>` on macOS, `xdg-open <file>` on Linux), or offer to.
- **Sharing.** The page is self-contained, so it can be committed to the repo, attached, or uploaded to claude.ai as an artifact by the user. Don't commit or push it unless asked.
- **After writing,** reply as described in "Reply", plus one line on what the page holds.

## Template

Copy this and replace every `[…]`. Delete any optional block you don't need, and its contents link with it.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1,viewport-fit=cover">
<title>[Name] — [Kind of page]</title>
<link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap">
<style>
  :root {
    --bg: #ffffff; --surface: #f6f8f7; --line: #e2e7e4; --fg: #1c2421; --muted: #55625c; --faint: #7a8781;
    --accent: #16a34a; --accent-soft: #ecf7f0; --warn: #b45309; --warn-soft: #fdf6ec; --code-bg: #f1f4f2;
    --font-body: "IBM Plex Sans", ui-sans-serif, system-ui, -apple-system, "Segoe UI", Roboto, Arial, sans-serif;
    --font-mono: "IBM Plex Mono", ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
    color-scheme: light;
    padding-top: env(safe-area-inset-top, 0px); padding-bottom: env(safe-area-inset-bottom, 0px);
  }
  @media (prefers-color-scheme: dark) {
    :root {
      --bg: #121715; --surface: #1a211e; --line: #2a3430; --fg: #e6ece9; --muted: #a7b3ad; --faint: #85918b;
      --accent: #3ccf72; --accent-soft: #173323; --warn: #f0b35a; --warn-soft: #2c2416; --code-bg: #1d2522;
      color-scheme: dark;
    }
  }
  * { box-sizing: border-box; margin: 0; }
  html { scroll-behavior: smooth; scroll-padding-top: env(safe-area-inset-top, 0px); }
  @media (prefers-reduced-motion: reduce) { html { scroll-behavior: auto; } }
  body { background: var(--bg); color: var(--fg); font-family: var(--font-body); font-size: 15.5px; line-height: 1.7; -webkit-font-smoothing: antialiased; }
  section[id] { scroll-margin-top: 24px; }
  @media (max-width: 1023px) { section[id] { scroll-margin-top: 72px; } }
  .wrap { max-width: 1120px; margin: 0 auto; padding-left: max(16px, env(safe-area-inset-left, 0px)); padding-right: max(16px, env(safe-area-inset-right, 0px)); }
  @media (min-width: 1024px) { .wrap { display: grid; grid-template-columns: 220px minmax(0, 1fr); gap: 56px; } }
  main { min-width: 0; max-width: 800px; padding-block: 40px 64px; }
  h1 { font-size: clamp(1.9rem, 4.5vw, 2.5rem); line-height: 1.15; font-weight: 700; letter-spacing: -0.02em; text-wrap: balance; }
  h2 { font-size: 1.5rem; line-height: 1.3; font-weight: 700; letter-spacing: -0.01em; text-wrap: balance; }
  h3 { font-size: 1rem; font-weight: 600; margin: 28px 0 8px; }
  p, li { max-width: 68ch; }
  .muted { color: var(--muted); }
  strong { color: var(--fg); font-weight: 600; }
  a.link { color: var(--accent); text-decoration: underline; text-underline-offset: 2px; }
  a:focus-visible, button:focus-visible, summary:focus-visible { outline: 2px solid var(--accent); outline-offset: 2px; border-radius: 4px; }
  .label { font-size: 11px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.12em; color: var(--accent); }
  .p { color: var(--muted); margin-bottom: 12px; }
  .list { list-style: disc; padding-left: 20px; color: var(--muted); margin-bottom: 12px; }
  .olist { list-style: decimal; padding-left: 20px; color: var(--muted); margin-bottom: 12px; }
  .list li, .olist li { margin: 3px 0; }
  code { font-family: var(--font-mono); font-size: 0.86em; background: var(--code-bg); color: var(--fg); border-radius: 4px; padding: 1px 5px; }
  pre { font-family: var(--font-mono); font-size: 12.5px; line-height: 1.6; background: var(--surface); border: 1px solid var(--line); border-radius: 8px; padding: 14px 16px; overflow-x: auto; margin: 12px 0; color: var(--fg); white-space: pre; }
  pre code { background: none; padding: 0; font-size: inherit; }
  .src-note { font-size: 13px; color: var(--faint); margin-top: 6px; }
  .divider { border: 0; border-top: 1px solid var(--line); margin: 48px 0; }
  .pill { display: inline-flex; align-items: center; border: 1px solid var(--line); border-radius: 999px; padding: 3px 12px; font-size: 13px; color: var(--muted); background: var(--bg); text-decoration: none; }
  a.pill:hover { border-color: var(--accent); color: var(--accent); }
  .gist { border-left: 4px solid var(--accent); background: var(--surface); border-radius: 0 8px 8px 0; padding: 12px 16px; margin: 14px 0 18px; font-size: 16px; color: var(--fg); max-width: 68ch; }
  .card { background: var(--surface); border: 1px solid var(--line); border-radius: 12px; padding: 18px; min-width: 0; }
  .card h3 { margin: 0 0 6px; font-size: 15px; }
  .card p { font-size: 14.5px; line-height: 1.6; color: var(--muted); }
  .card .list { font-size: 14.5px; line-height: 1.6; margin: 0; }
  .grid3 { display: grid; gap: 14px; }
  @media (min-width: 768px) { .grid3 { grid-template-columns: repeat(3, minmax(0, 1fr)); } }
  .grid2 { display: grid; gap: 12px; }
  @media (min-width: 640px) { .grid2 { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
  .tag { display: block; font-size: 11px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.12em; margin-bottom: 8px; line-height: 1.4; color: var(--faint); }
  .co-def, .co-ex, .co-do, .co-warn { display: block; border-radius: 8px; padding: 14px 18px; margin: 18px 0; font-size: 15px; line-height: 1.7; color: var(--muted); min-width: 0; }
  .co-def { background: var(--bg); border: 1px solid var(--line); border-left: 4px solid var(--accent); }
  .co-def > .tag { color: var(--accent); }
  .co-ex { background: var(--surface); border: 1px solid var(--line); }
  .co-do { background: var(--bg); border: 1px solid var(--line); }
  .co-do > .tag { color: var(--fg); }
  .co-warn { background: var(--warn-soft); border-left: 4px solid var(--warn); }
  .co-warn > .tag { color: var(--warn); }
  .co-def .olist, .co-def .list, .co-do .olist, .co-do .list, .co-ex .list, .co-ex .olist, .co-warn .list { margin-bottom: 0; }
  .tbl-wrap { overflow-x: auto; border: 1px solid var(--line); border-radius: 12px; margin: 16px 0; }
  table.tbl { width: 100%; border-collapse: collapse; font-size: 14.5px; text-align: left; }
  .tbl th { background: var(--surface); font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.04em; color: var(--muted); padding: 10px 14px; border-bottom: 1px solid var(--line); white-space: nowrap; }
  .tbl td { padding: 10px 14px; border-bottom: 1px solid var(--line); vertical-align: top; color: var(--muted); line-height: 1.55; }
  .tbl tr:last-child td { border-bottom: 0; }
  .tbl td:first-child { color: var(--fg); font-weight: 500; }
  .tbl .mod td { background: var(--surface); color: var(--fg); font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.06em; }
  .tbl .num { font-variant-numeric: tabular-nums; white-space: nowrap; }
  .tbl .hi { color: var(--fg); background: var(--accent-soft); }
  .tbl td.mono { font-family: var(--font-mono); font-size: 13px; white-space: nowrap; }
  .badge { display: inline-block; font-size: 11.5px; font-weight: 600; border-radius: 999px; padding: 1px 9px; border: 1px solid var(--line); white-space: nowrap; }
  .badge.must { color: var(--fg); background: var(--accent-soft); border-color: transparent; }
  .badge.should { color: var(--warn); background: var(--warn-soft); border-color: transparent; }
  .badge.later { color: var(--faint); }
  .flow { display: flex; flex-wrap: wrap; align-items: center; gap: 8px; margin: 14px 0; font-size: 14px; }
  .flow .s { border: 1px solid var(--line); border-radius: 8px; padding: 5px 12px; background: var(--bg); color: var(--fg); }
  .flow .s.on { border-color: var(--accent); background: var(--accent-soft); font-weight: 600; }
  .flow .arr { color: var(--faint); }
  .layers { display: grid; gap: 10px; margin: 16px 0; }
  .layer { display: grid; gap: 4px 16px; border: 1px solid var(--line); border-radius: 10px; padding: 12px 16px; background: var(--bg); }
  @media (min-width: 640px) { .layer { grid-template-columns: 170px minmax(0, 1fr); align-items: baseline; } }
  .layer b { font-size: 14px; }
  .layer span { font-size: 14.5px; color: var(--muted); }
  .layer.top { border-left: 4px solid var(--accent); }
  .stories { display: grid; gap: 12px; margin: 14px 0 6px; }
  @media (min-width: 640px) { .stories { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
  .story { border: 1px solid var(--line); border-radius: 10px; padding: 14px 16px; min-width: 0; background: var(--bg); }
  .story .id { font-family: var(--font-mono); font-size: 12px; color: var(--accent); font-weight: 500; }
  .story h4 { font-size: 15px; font-weight: 600; margin: 2px 0 4px; }
  .story .as { font-size: 14px; color: var(--fg); margin-bottom: 6px; }
  .story .list { font-size: 14px; line-height: 1.55; margin: 0; }
  .sprints { display: grid; gap: 12px; margin: 16px 0; }
  @media (min-width: 640px) { .sprints { grid-template-columns: repeat(2, minmax(0, 1fr)); } }
  @media (min-width: 900px) { .sprints { grid-template-columns: repeat(4, minmax(0, 1fr)); } }
  .sprint { border: 1px solid var(--line); border-radius: 10px; padding: 14px 16px; min-width: 0; display: flex; flex-direction: column; gap: 4px; }
  .sprint.last { border: 2px solid var(--accent); background: var(--accent-soft); }
  .sprint h3 { margin: 0; font-size: 15px; }
  .sprint .dates { font-size: 13px; color: var(--faint); font-variant-numeric: tabular-nums; }
  .sprint p { font-size: 14px; color: var(--muted); line-height: 1.55; }
  .sprint .gate { margin-top: auto; padding-top: 8px; border-top: 1px dashed var(--line); font-size: 12.5px; color: var(--fg); font-weight: 500; }
  .filter { display: flex; flex-wrap: wrap; gap: 8px; margin: 10px 0 0; }
  .filter button { border: 1px solid var(--line); background: var(--bg); color: var(--muted); border-radius: 999px; padding: 5px 12px; font: 500 13px var(--font-body); cursor: pointer; }
  .filter button:hover { border-color: var(--accent); color: var(--accent); }
  .filter button[aria-pressed="true"] { border-color: var(--accent); background: var(--accent-soft); color: var(--fg); font-weight: 600; }
  .dg-wrap { margin: 16px 0; border: 1px solid var(--line); border-radius: 12px; background: var(--bg); padding: 12px 12px 10px; overflow-x: auto; }
  .dg { display: block; width: 100%; min-width: 640px; height: auto; font-family: var(--font-body); }
  .dg .panel { fill: var(--surface); stroke: var(--line); stroke-width: 1.25; }
  .dg .box { fill: var(--bg); stroke: var(--line); stroke-width: 1.25; }
  .dg .box.warn { stroke: var(--warn); stroke-width: 2; fill: var(--warn-soft); }
  .dg .box.ag { stroke: var(--accent); stroke-width: 2; fill: var(--accent-soft); }
  .dg .t { fill: var(--fg); font-size: 12.5px; font-weight: 600; text-anchor: middle; }
  .dg .s { fill: var(--muted); font-size: 11.5px; text-anchor: middle; }
  .dg .s.l { text-anchor: start; }
  .dg .cap { fill: var(--faint); font-size: 10.5px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.08em; }
  .dg .ln { stroke: var(--faint); stroke-width: 1.3; fill: none; }
  .dg .ln.m { stroke: var(--accent); stroke-width: 1.6; }
  .dg .ah { fill: var(--faint); }
  .dg .ah.m { fill: var(--accent); }
  .legend { display: flex; flex-wrap: wrap; gap: 6px 18px; margin-top: 8px; font-size: 13px; color: var(--muted); }
  .checks { list-style: none; padding: 0; margin: 0; }
  .checks li { padding-left: 26px; position: relative; margin: 4px 0; }
  .checks li::before { content: ""; position: absolute; left: 0; top: 6px; width: 14px; height: 14px; border: 1.5px solid var(--faint); border-radius: 4px; }
  .toc a { display: block; border-left: 2px solid var(--line); padding: 4px 0 4px 12px; font-size: 14px; color: var(--muted); text-decoration: none; line-height: 1.45; }
  .toc a:hover { color: var(--fg); }
  .toc a.active { border-color: var(--accent); color: var(--accent); font-weight: 600; }
  .toc .grp { font-size: 11px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.1em; color: var(--faint); margin: 14px 0 4px; }
  .toc-head { font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.12em; color: var(--faint); margin-bottom: 10px; }
  .mtoc { position: sticky; top: env(safe-area-inset-top, 0px); z-index: 40; background: var(--bg); border-bottom: 1px solid var(--line); padding-left: max(16px, env(safe-area-inset-left, 0px)); padding-right: max(16px, env(safe-area-inset-right, 0px)); }
  .mtoc summary { list-style: none; display: flex; justify-content: space-between; padding-block: 12px; cursor: pointer; font-size: 12px; font-weight: 600; text-transform: uppercase; letter-spacing: 0.12em; color: var(--fg); }
  .mtoc summary::-webkit-details-marker { display: none; }
  .mtoc nav { padding-bottom: 14px; max-height: 70vh; overflow-y: auto; }
  body::before { content: ""; position: fixed; inset: 0 0 auto; height: env(safe-area-inset-top, 0px); z-index: 50; background: var(--bg); }
  @media (min-width: 1024px) { .mtoc { display: none; } }
  .dtoc { display: none; }
  @media (min-width: 1024px) { .dtoc { display: block; } .dtoc nav { position: sticky; top: 0; max-height: 100vh; overflow-y: auto; padding-block: 44px; } }
</style>
</head>
<body>

<details class="mtoc" id="mobile-toc">
  <summary><span>On this page</span><span aria-hidden="true" id="mobile-toc-icon">+</span></summary>
  <nav aria-label="On this page (mobile)" class="toc" data-toc></nav>
</details>

<div class="wrap">
  <aside class="dtoc">
    <nav aria-label="On this page" class="toc">
      <p class="toc-head">On this page</p>
      <div data-toc-source>
        <a href="#summary">Summary</a>
        <p class="grp">[Group]</p>
        <a href="#section-a">[Section A]</a>
        <a href="#section-b">[Section B]</a>
        <p class="grp">Wrap-up</p>
        <a href="#questions">Risks &amp; open questions</a>
        <a href="#sources">Sources</a>
      </div>
    </nav>
  </aside>

  <main id="main">
    <header>
      <p class="label">[Kind of page] · updated [date]</p>
      <h1 style="margin-top:12px">[Name]</h1>
      <p class="muted" style="margin-top:12px;font-size:17px;max-width:60ch">[One sentence: what this page is and for whom.]</p>
      <div style="display:flex;flex-wrap:wrap;gap:8px;margin-top:20px">
        <span class="pill">[Key fact]</span><span class="pill">[Key fact]</span>
        <a class="pill" href="[url]">[Related artifact] ↗</a>
      </div>
    </header>

    <hr class="divider">

    <section id="summary" aria-labelledby="summary-h">
      <h2 id="summary-h" style="margin-bottom:16px">Summary</h2>
      <div class="gist" style="padding:18px 22px">
        <p class="label" style="margin-bottom:6px">In one line</p>
        <p style="font-size:17px;line-height:1.7"><strong>[The point of the whole page.]</strong></p>
      </div>
      <div class="grid3" style="margin-top:20px">
        <div class="card"><h3>[Fact]</h3><p>[…]</p></div>
        <div class="card"><h3>[Fact]</h3><p>[…]</p></div>
        <div class="card"><h3>[Fact]</h3><ul class="list"><li>[…]</li></ul></div>
      </div>
    </section>

    <hr class="divider">

    <section id="section-a" aria-labelledby="a-h">
      <p class="label">[Group]</p>
      <h2 id="a-h" style="margin-top:4px">[Section A]</h2>
      <p class="gist">[Lead sentence that states the point.]</p>
      <div class="tbl-wrap"><table class="tbl">
        <thead><tr><th>[Column]</th><th>[Column]</th><th>[Column]</th></tr></thead>
        <tbody>
          <tr class="mod"><td colspan="3">[Group row]</td></tr>
          <tr><td>[Item]</td><td class="hi">[Highlighted value]</td><td><span class="badge must">Must</span></td></tr>
        </tbody>
      </table></div>
      <div class="grid2">
        <div class="co-do" style="margin:0"><span class="tag">[Do / in scope]</span><ul class="list"><li>[…]</li></ul></div>
        <div class="co-warn" style="margin:0"><span class="tag">[Watch out / out of scope]</span><ul class="list"><li>[…]</li></ul></div>
      </div>
      <div class="co-def"><span class="tag">[Rule]</span>[Definition or key rule.]</div>
      <div class="stories">
        <div class="story"><span class="id">[ID-01]</span><h4>[Title]</h4><p class="as">[One-line statement.]</p><ul class="list"><li>[Criterion]</li></ul></div>
      </div>
      <div class="flow" aria-label="[Flow name]"><span class="s">[Step]</span><span class="arr">→</span><span class="s">[Step]</span><span class="arr">→</span><span class="s on">[End state]</span></div>
      <div class="layers">
        <div class="layer top"><b>[Main layer]</b><span>[What it is.]</span></div>
        <div class="layer"><b>[Layer]</b><span>[What it is.]</span></div>
      </div>
    </section>

    <hr class="divider">

    <section id="section-b" aria-labelledby="b-h">
      <p class="label">[Group]</p>
      <h2 id="b-h" style="margin-top:4px">[Section B]</h2>
      <p class="gist">[Lead sentence.]</p>
      <figure class="dg-wrap" aria-labelledby="b-cap">
        <svg class="dg" viewBox="0 0 760 200" role="img" aria-label="[What the diagram shows]">
          <defs>
            <marker id="ah" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path class="ah" d="M0 0L10 5L0 10z"/></marker>
            <marker id="ahm" viewBox="0 0 10 10" refX="9" refY="5" markerWidth="6" markerHeight="6" orient="auto-start-reverse"><path class="ah m" d="M0 0L10 5L0 10z"/></marker>
          </defs>
          <rect class="panel" x="24" y="24" width="712" height="140" rx="10"/>
          <text class="cap" x="40" y="46">[Container]</text>
          <rect class="box" x="40" y="72" width="150" height="64" rx="8"/>
          <text class="t" x="115" y="96">[Part]</text><text class="s" x="115" y="113">[detail]</text><text class="s" x="115" y="128">[detail]</text>
          <rect class="box ag" x="222" y="72" width="160" height="64" rx="8"/>
          <text class="t" x="302" y="96">[Highlighted part]</text><text class="s" x="302" y="113">[detail]</text>
          <rect class="box warn" x="414" y="72" width="160" height="64" rx="8"/>
          <text class="t" x="494" y="96">[Escalation]</text><text class="s" x="494" y="113">[detail]</text>
          <path class="ln m" d="M190 104H220" marker-end="url(#ahm)"/>
          <path class="ln" d="M382 104H412" marker-end="url(#ah)"/>
        </svg>
        <figcaption id="b-cap" class="legend"><span>[One sentence of reading.]</span></figcaption>
      </figure>
      <div class="sprints">
        <div class="sprint"><span class="label">[Phase 1]</span><h3>[Theme]</h3><span class="dates">[dates]</span><p>[One line.]</p><span class="gate">◇ [Gate]</span></div>
        <div class="sprint last"><span class="label">[Phase 2]</span><h3>[Milestone]</h3><span class="dates">[dates]</span><p>[One line.]</p><span class="gate">◆ [Milestone gate]</span></div>
      </div>
      <div class="filter" role="group" aria-label="Filter">
        <button type="button" data-f="all" aria-pressed="true">All</button>
        <button type="button" data-f="g1" aria-pressed="false">[Group 1]</button>
        <button type="button" data-f="g2" aria-pressed="false">[Group 2]</button>
      </div>
      <div class="tbl-wrap"><table class="tbl" id="backlog-tbl">
        <thead><tr><th>ID</th><th>Item</th><th>Link</th></tr></thead>
        <tbody>
          <tr class="mod" data-s="g1"><td colspan="3">[Group 1]</td></tr>
          <tr data-s="g1"><td class="mono">[ID]</td><td>[Item]</td><td>[…]</td></tr>
          <tr class="mod" data-s="g2"><td colspan="3">[Group 2]</td></tr>
          <tr data-s="g2"><td class="mono">[ID]</td><td>[Item]</td><td>[…]</td></tr>
        </tbody>
      </table></div>
<pre><code>[code or file tree]</code></pre>
    </section>

    <hr class="divider">

    <section id="questions" aria-labelledby="q-h">
      <p class="label">Wrap-up</p>
      <h2 id="q-h" style="margin-top:4px">Risks and open questions</h2>
      <p class="gist">[Where the main risk sits.]</p>
      <div class="tbl-wrap"><table class="tbl">
        <thead><tr><th>Risk</th><th>Impact</th><th>Mitigation</th></tr></thead>
        <tbody><tr><td>[Risk]</td><td>[Impact]</td><td>[Mitigation]</td></tr></tbody>
      </table></div>
      <div class="co-def"><span class="tag">Open questions</span><ul class="checks"><li>[Question]</li></ul></div>
    </section>

    <section id="sources" aria-labelledby="sources-h" style="margin-top:48px">
      <h2 id="sources-h" style="margin-bottom:12px">Sources</h2>
      <ul class="list"><li><a class="link" href="[url]">[Title]</a>: [what it supports]</li></ul>
    </section>

    <footer style="margin-top:56px;padding-top:20px;border-top:1px solid var(--line);font-size:13px;color:var(--faint)">[Name] · [one-line footer]</footer>
  </main>
</div>

<script>
  (function () {
    var source = document.querySelector('[data-toc-source]');
    var mobileNav = document.querySelector('[data-toc]');
    if (source && mobileNav) mobileNav.innerHTML = source.innerHTML;
    var details = document.getElementById('mobile-toc');
    var icon = document.getElementById('mobile-toc-icon');
    if (details) {
      details.addEventListener('toggle', function () { icon.textContent = details.open ? '−' : '+'; });
      mobileNav.addEventListener('click', function (e) { if (e.target.closest('a')) details.open = false; });
    }
    var links = Array.prototype.slice.call(document.querySelectorAll('.toc a[href^="#"]'));
    var targets = Array.prototype.slice.call(document.querySelectorAll('main section[id]'));
    function setActive(id) { links.forEach(function (a) { a.classList.toggle('active', a.getAttribute('href') === '#' + id); }); }
    if ('IntersectionObserver' in window) {
      var visible = {};
      var io = new IntersectionObserver(function (entries) {
        entries.forEach(function (en) { visible[en.target.id] = en.isIntersecting; });
        for (var i = targets.length - 1; i >= 0; i--) { if (visible[targets[i].id]) { setActive(targets[i].id); break; } }
      }, { rootMargin: '-10% 0px -75% 0px' });
      targets.forEach(function (t) { io.observe(t); });
    }
    if (targets.length) setActive(targets[0].id);

    var btns = document.querySelectorAll('.filter button');
    var rows = document.querySelectorAll('#backlog-tbl tbody tr');
    btns.forEach(function (b) {
      b.addEventListener('click', function () {
        var f = b.getAttribute('data-f');
        btns.forEach(function (x) { x.setAttribute('aria-pressed', x === b ? 'true' : 'false'); });
        rows.forEach(function (r) { r.hidden = !(f === 'all' || r.getAttribute('data-s') === f); });
      });
    });
  })();
</script>
</body>
</html>
```