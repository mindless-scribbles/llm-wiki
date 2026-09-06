# [Your Domain] Knowledge Base — Schema

## Purpose

<!-- CUSTOMIZE: Replace this with a one-paragraph description of your knowledge domain. -->
<!-- Examples: "machine learning research", "19th-century literature", "competitive landscape for SaaS tools" -->
This is an LLM-maintained knowledge base on [YOUR TOPIC]. The LLM writes and maintains all files under `wiki/`. The human curates raw sources and directs queries. The human never edits wiki files directly.

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
- `wiki/tutorials/` — Step-by-step, timecode-linked tutorials distilled from timecoded transcript pages (mirrors the source page's path under `wiki/`). Each step links to the exact transcript moment — a heading link in Obsidian, a click-to-popover on the site. See the **Add Tutorial** workflow and the `add-tutorial` skill.
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
type: concept | entity | summary | synthesis
tags: [tag1, tag2, tag3]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: ["raw/filename.txt"]
confidence: high | medium | low
---
```

Tutorial pages (`wiki/tutorials/`) additionally set a `transcript: "<source-target>"` key
naming the timecoded transcript they were distilled from — this is what enables the
clickable-timecode popovers. See the **Add Tutorial** workflow.

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
4. For each concept/entity: create the page if it doesn't exist, or update it with new information if it does
5. Add cross-links in both directions between all touched pages
6. Update `wiki/index.md` — add new entries, update summaries of changed pages
7. Append to `wiki/log.md` with timestamp, source name, pages created/updated
8. Flag any contradictions with existing wiki content
9. **Rebuild the HTML site**: run `llm-wiki-site build .` (see Publish). This is a default step of every ingest — the site should never lag the wiki.

### Add Tutorial

When the user wants a **step-by-step tutorial** from a timecoded transcript (a page whose
headings are `## MM:SS`), or asks to "break this lesson into steps" / "tie each step to a
timecode", run the **`add-tutorial`** skill. In short:

1. Normalize the transcript's timecode headings to have no brackets (`## [MM:SS]` → `## MM:SS`) —
   Obsidian can't link to headings containing `[`/`]`.
2. Write `wiki/tutorials/<same-path-as-source>.md` with a `transcript: "<source-target>"` key and
   phased, numbered steps. Every step opens with a timecode wikilink `[[wiki/<source-target>#MM:SS|MM:SS]]`
   whose `MM:SS` exactly matches a transcript heading (`HH:MM:SS` past the hour).
3. Update `index.md` (a "Tutorials — Step-by-Step" section) and `log.md`, then rebuild.

`llm-wiki-site` renders those timecode wikilinks as pills that pop up the transcript chunk
inline on the site, and they double as heading links in Obsidian. See `.claude/skills/add-tutorial/SKILL.md`.

### Publish

The wiki always has a companion static HTML site (Field Logs journal aesthetic),
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
- Requires Node 18+. No dependencies and no network needed to build.

#### Branding

Set these when adapting the template to a new domain: `title`, `brandLetters`
(exactly two), `footer`, `accent` (hex).

For a wiki inside an Obsidian vault, put them in the `site:` block of
`wiki/index.md` frontmatter — that is markdown, so it is the only form that
actually syncs:

```yaml
---
title: "Knowledge Base Index"
site:
  title: "Trading Field Logs"
  brandLetters: "TF"
  footer: "SYS.TRADING_WIKI / 2026"
  accent: "#33ccff"
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
2. Check for: orphan pages (no inbound links), stale claims, contradictions between pages, missing cross-links, incomplete sections, low-confidence pages that could be strengthened
3. Fix what can be fixed automatically
4. Report issues that need human judgment
5. Suggest new sources or topics to investigate
6. Update log
7. Rebuild the HTML site (`llm-wiki-site build .`) if any page changed

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
