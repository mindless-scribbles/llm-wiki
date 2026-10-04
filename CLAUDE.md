# [Your Domain] Knowledge Base — Schema

## Purpose

<!-- CUSTOMIZE: Replace this with a one-paragraph description of your knowledge domain. -->
<!-- Examples: "machine learning research", "19th-century literature", "competitive landscape for SaaS tools" -->
This is an LLM-maintained knowledge base on [YOUR TOPIC]. The LLM writes and maintains all files under `wiki/`. The human curates raw sources and directs queries. The human never edits wiki files directly.

## Writing Rule (STE-80)

Pages follow the writing rules of ASD-STE100 (Simplified Technical English), about 80% of the way. Use its rules, not its dictionary: STE's approved-word list rejects a domain's own terms.

- Lead with the decision and why. Then give the minimum mechanism.
- One topic per paragraph, at most 6 sentences. Prefer a list or a table.
- A step is a command that starts with a verb: "Set the switch to FK.", not "Switch: FK." One action per step, unless the actions happen together.
- A step is at most 20 words. A descriptive sentence is at most 25. A `code span` or a [[link]] counts as one word.
- Use the active voice in steps. A description can use the passive when the actor does not matter.
- Use the same word for the same thing on every page. The Terminology table below is the list.
- Use plain words: "to", not "in order to"; "before", not "prior to"; "make sure", not "ensure"; "use", not "utilize".

`llm-wiki-site lint .` counts the limits and finds the words. It only reports; the LLM fixes what it flags.

### Terminology

One word for each thing. The lint flags every term in the `Not` column, except inside a code span.

<!-- CUSTOMIZE: /init-wiki drafts these rows from raw/. Replace the placeholder row. -->
| Use | Not | Note |
|---|---|---|
| [term] | [synonyms to avoid] | [why, or where it is defined] |

## Directory Layout

- `raw/` — Immutable source documents (transcripts, articles, notes). Never modify these.
- `wiki/index.md` — Master catalog. Every wiki page must appear here.
- `wiki/log.md` — Append-only activity log.
- `wiki/summaries/` — One summary page per raw source document.
- `wiki/concepts/` — Concept, strategy, and framework pages.
- `wiki/entities/` — Entity pages (people, tools, organizations, products — whatever "things" exist in your domain).
- `wiki/syntheses/` — Comparison tables, decision frameworks, cross-cutting analyses.
- `wiki/journal/` — Research or session journal entries.
- `wiki/presentations/` — Marp slide decks generated from wiki content.
- `wiki/tutorials/` — Step-by-step, timecode-linked tutorials (`type: lesson`) distilled from timecoded transcript pages (mirrors the source page's path under `wiki/`). Each step links to the exact transcript moment — a heading link in Obsidian, a click-to-popover on the site. See the **Add Lesson** workflow.
- `site.config.json` — Branding for the HTML site (title, brand letters, footer, accent color). Optional; see **Branding** below.

**This folder holds markdown and nothing else.** The static HTML site is built by a
separate tool, [`llm-wiki-site`](https://github.com/mindless-scribbles/llm-wiki-site),
which reads this wiki from the outside and writes the HTML somewhere else entirely.
No build script, no widget `.js`, and no generated `site/` folder belongs here — a
wiki usually lives in an Obsidian vault, and Obsidian Sync carries only `*.md`.

## File Naming

- All lowercase, hyphens for word separation: `concept-name.md`
- No spaces, no special characters, no uppercase
- Name should match the page title slug

## Page Format

Every wiki page uses this frontmatter and structure:

```yaml
---
title: "Page Title"
type: concept | entity | summary | synthesis | lesson
tags: [tag1, tag2, tag3]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["raw/filename.txt"]
confidence: high | medium | low
---
```

Tutorial pages (`wiki/tutorials/`) additionally set a `transcript: "<source-target>"` key
naming the timecoded transcript they were distilled from — this is what enables the
clickable-timecode popovers. See the **Add Lesson** workflow.

### Diagrams

Draw structure as a ```` ```mermaid ```` block in the page: never an image file, never ASCII art.
Obsidian renders it natively, and `llm-wiki-site` renders it to inline SVG in the DDC Reel look.
Pick the UML diagram type that matches what the prose describes (modes → state diagram, an order
of calls → sequence diagram, connected parts → component diagram). See the **Add Diagram** workflow.

### Required Sections by Page Type

**Summary pages** (`wiki/summaries/`):
- `## Key Points` — Bulleted list of main claims/ideas
- `## Relevant Concepts` — Links to concept pages this source touches
- `## Source Metadata` — Type of source, author/speaker, date, URL or identifier

**Concept pages** (`wiki/concepts/`):
- `## Definition` — One-paragraph plain-English definition
- `## How It Works` — Mechanics, process, or structure of the concept
- `## Key Parameters` — Important variables, dimensions, or factors
- `## When To Use` — Situations and contexts where this concept applies
- `## Risks & Pitfalls` — Known failure modes, common mistakes, limitations
- `## Related Concepts` — Wiki links to related pages
- `## Sources` — Which raw sources inform this page

**Entity pages** (`wiki/entities/`):
- `## Overview` — What this entity is
- `## Characteristics` — Key properties, attributes, structure
- `## Common Strategies` — Links to concept pages for strategies or methods associated with this entity
- `## Related Entities` — Links to related entity pages

**Lesson pages** (`type: lesson`, for a workshop day or a tutorial): the site renders these as a
lesson, not an article. Write the teaching structure as these plain-markdown markers:
- Before the first phase: `**Goal:**`, `**Result:**`, and `**Open in <app>:**` with a list: say which
  project to open, which assets, and which parts of them today covers.
- `## Phase N — Title (75 min)` for each phase. The minutes set its bar on the phase rail.
  The budget must match the work in the phase. Do not pad it.
- `**Why:**` first in every phase: what the phase teaches, and what breaks without it. A teacher
  reads it as the syllabus line; a learner reads it as the reason to care.
- Teach the why; let a visual carry the how. Show the graph or the screen as `@image[slug]`
  (a capture of the reference) and explain what each part does. Do not list the wiring that
  the capture already shows. Until the capture exists, the `@image` asks for it in review mode.
- Numbered steps. Keep the numbers running across the page.
- `**Ask:**` then `**Decision:**` only for a call the room could get wrong: the wrong answer is
  tempting and costs something you can see. The site hides the decision behind a reveal.
  Otherwise state it as a fact with a bold lead: `**Project, don't copy.** …`.
- `**Predict:**` before a test, then `**Read:**` for what to observe. The site hides the read-off until the reader checks.
- A line that starts with `⚠️` for a trap. A `> quote` for the line to remember.
- One visual per phase where you can: a Mermaid fence, `@image[slug]`, `@video[slug]` or `@viz[slug]`.
  A one-line paragraph ending in `:` before it is its title; one sentence after it is its caption.
- `## Related` last.

**Synthesis pages** (`wiki/syntheses/`):
- `## Comparison` — Table or structured comparison
- `## Analysis` — Cross-cutting insights
- `## Recommendations` — When to prefer which approach
- `## Pages Compared` — Links to all pages involved

## Linking Conventions

- Use Obsidian-style wiki links: `[[concepts/concept-name]]`
- Always use relative paths from wiki root
- Every page must link to at least one other page (no orphans)
- When mentioning a concept that has a page, always link it

## Tagging Taxonomy

<!-- CUSTOMIZE: Replace these placeholder categories with tags relevant to your domain. -->
<!-- Each category should have 3-8 specific tags. -->
<!-- Example for a cooking KB: -->
<!--   Cuisine: italian, japanese, french, mexican -->
<!--   Technique: braising, fermenting, sous-vide, grilling -->
<!--   Ingredient: protein, vegetable, grain, dairy -->

- **Category-A**: `tag-1`, `tag-2`, `tag-3`
- **Category-B**: `tag-4`, `tag-5`, `tag-6`
- **Category-C**: `tag-7`, `tag-8`, `tag-9`
- **Scope**: `foundational`, `advanced`, `experimental`
- **Status**: `well-established`, `emerging`, `speculative`

## Confidence Levels

- **high** — Well-established idea, multiple corroborating sources, demonstrated with concrete examples
- **medium** — Supported by sources but limited examples or single-source
- **low** — Single mention, anecdotal, or speculative

## Workflows

### Ingest

When the user says "ingest [source]" or adds a file to `raw/`:

1. Read the raw source completely
2. Create `wiki/summaries/<source-slug>.md` with full summary
3. Identify all concepts, entities, and strategies mentioned
4. For each concept/entity: create the page if it doesn't exist, or update it with new information if it does. Write to the **Writing Rule**, and use the Terminology table's words. If a source names a thing the table doesn't cover yet, add a row.
5. Where a page describes modes and the switches between them, an order of calls or writes, or parts that connect, add a diagram (**Add Diagram**). Skip it when a numbered list or a table already shows the structure.
6. Add cross-links in both directions between all touched pages
7. Update `wiki/index.md` — add new entries, update summaries of changed pages
8. Append to `wiki/log.md` with timestamp, source name, pages created/updated
9. Flag any contradictions with existing wiki content
10. Run `llm-wiki-site lint .` and fix what it flags on the pages you touched.
11. **Rebuild the HTML site**: run `llm-wiki-site build .` (see Publish). This is a default step of every ingest — the site should never lag the wiki.

### Add Lesson

This wiki has two layers. The reference pages (concepts, entities, theory) explain how things
work; **Ingest** builds them. Lessons (`type: lesson`) teach it in order, as a workshop or a
tutorial. When the user asks for a workshop, a lesson, a tutorial or a step-by-step, run the
global **`add-lesson`** skill. In short:

1. If there is no course overview, draft the syllabus first (who, prerequisites, each lesson's Why
   and outcome) and get the user's OK.
2. Write one pilot lesson to the **Lesson pages** rules above, then review it with the user before
   writing the rest.
3. Evidence for each step is either a reference (snapshot in `raw/`, plus screen captures as
   `@image` slots) or a timecoded transcript (timecode pills; lessons go in `wiki/tutorials/`).
4. Update `index.md` and `log.md`, then rebuild.

### Add Diagram

When a page describes structure in prose, or the user asks for a diagram, run the
**`add-diagram`** skill. In short:

1. Pick the diagram type from what the prose describes: modes → `stateDiagram-v2`; an order of
   calls → `sequenceDiagram`; connected parts → `flowchart` with subgraphs; steps with branches →
   `flowchart TD`; types and fields → `classDiagram`.
2. Lay it out top-down, keep it to about 12 nodes, and label it with the page's own terms.
3. Put it in the section it explains, follow it with a one-sentence caption, and remove any
   prose list or ASCII drawing it replaces.
4. Check it renders (`mmdc`), update `index.md` and `log.md`, then rebuild.

See `.claude/skills/add-diagram/SKILL.md`.

### Review

The human reads the site on the local server and leaves notes on it. Run the global
**`work-notes`** skill when they say "work the notes". In short:

- One server serves every registered wiki: `llm-wiki-site serve`, then
  `http://127.0.0.1:4173/<wiki-id>/`. Notes are always on there. The files in the build output stay clean.
- A **content** note is about what a page says: fix it in this wiki's markdown. Go back to `raw/`,
  or research, when the page cannot answer it. A **design** note is about how every page of a
  kind is laid out: it is fixed in `llm-wiki-site`, never here.
- A pasted screenshot goes to the builder's `sites/<wiki-id>/media/<slug>.png` and onto the page
  as `@image[slug] caption`. Never into this folder.
- Resolve each note with what changed (`llm-wiki-site notes resolve <note-id> -m "..."`), then
  update `index.md` and `log.md` and rebuild.

### Publish

The wiki always has a companion static HTML site, styled with the **DDC Reel** design system
(`ddc-reel` skill; https://claude.ai/artifact/QBTm2jQ8DC1f9bjhwiuDvw),
built **outside this folder** by `llm-wiki-site`. Regenerate it after **any**
change to `wiki/`:

```bash
llm-wiki-site build .            # or, once registered:
llm-wiki-site build --site <wiki-id>
```

- Output goes wherever the builder is pointed — never into this folder. It is
  fully regenerated each run; never hand-edit it.
- If the command is missing, clone and link the builder:
  `git clone https://github.com/mindless-scribbles/llm-wiki-site.git && cd llm-wiki-site && npm link`
- Optionally, give a concept page an interactive visualization by adding
  `sites/<wiki-id>/widgets/<slug>.js` **in the builder repo** (slug = concept
  filename). The build injects it automatically.
- ```` ```mermaid ```` blocks render as inline SVG when `mmdc` (`@mermaid-js/mermaid-cli`) is
  on `PATH`. Without it they stay code blocks on the site, and Obsidian still draws them.
- Requires Node 18+. No dependencies and no network needed to build.

#### Branding

Set these when adapting the template to a new domain: `title`, `brandLetters`
(exactly two), `footer`, `accent` (hex). Keep `accent` at DDC Reel's `#ff3300` unless
the user asks for another; the look itself (fonts, palette, components) comes from the
builder and is the same for every wiki. For any other visual made from this wiki (a Marp
deck, an Artifact, a widget), load the `ddc-reel` skill first.

For a wiki inside an Obsidian vault, put them in the `site:` block of
`wiki/index.md` frontmatter — that is markdown, so it is the only form that
actually syncs:

```yaml
---
title: "Knowledge Base Index"
site:
  title: "Options Trading KB"
  brandLetters: "OT"
  footer: "SYS.TRADING_WIKI / 2026"
  accent: "#ff3300"
---
```

For a wiki under plain git, `site.config.json` at this folder's root works too and
takes precedence over the frontmatter.

### Query

When the user asks a question:

1. Read `wiki/index.md` to find relevant pages
2. Read those pages
3. Synthesize an answer citing specific pages with wiki links
4. If the answer reveals new insight worth preserving:
   - Create a synthesis page in `wiki/syntheses/`
   - Update index and log
   - Rebuild the HTML site (`llm-wiki-site build .`)

### Lint

When the user says "lint" or "health check":

1. Read all wiki pages
2. Check for: orphan pages (no inbound links), stale claims, contradictions between pages, missing cross-links, incomplete sections, low-confidence pages that could be strengthened, and pages that describe modes, call orders or connected parts with no diagram
3. Run `llm-wiki-site lint .` (the Writing Rule's limits and the Terminology table). Rewrite each flagged sentence; keep every name, number and citation.
4. Fix what can be fixed automatically
5. Report issues that need human judgment
6. Suggest new sources or topics to investigate
7. Update log
8. Rebuild the HTML site (`llm-wiki-site build .`) if any page changed

## Rules

- Never modify files in `raw/`
- Always update `index.md` and `log.md` after any wiki change
- Always rebuild the HTML site (`llm-wiki-site build .`) after any wiki change — the site is a default deliverable, not an extra. Never hand-edit generated HTML.
- Never create build scripts, `.js` files, or a `site/` folder inside this wiki. The site is built from the outside, by `llm-wiki-site`.
- Prefer updating existing pages over creating duplicates
- When in doubt about a claim, set confidence to "low" and note the uncertainty
- Keep pages focused — one concept per page, split if a page gets too long
- Use plain English — define jargon on first use in each page
- All dates in ISO 8601 format: YYYY-MM-DD
- When a source provides specific examples, include them with concrete details
