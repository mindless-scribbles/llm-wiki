# LLM Wiki

An open-source template for building LLM-powered knowledge bases, following [Andrej Karpathy's "LLM Wiki" pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

You provide raw sources. The LLM reads them, writes structured wiki pages, cross-links everything, and maintains it over time. You never edit the wiki directly — you curate sources and ask questions.

## How It Works

The system has three layers:

```
raw/              Sources you collect (articles, transcripts, notes, PDFs)
wiki/             LLM-written & maintained pages (summaries, concepts, entities, syntheses)
CLAUDE.md         Schema that tells the LLM how to structure everything
```

Three operations drive the workflow:

| Operation | Trigger | What happens |
|-----------|---------|--------------|
| **Ingest** | "ingest raw/my-source.txt" | LLM reads the source, creates a summary page, creates/updates concept and entity pages, adds cross-links, updates the index and log, and rebuilds the HTML site |
| **Query** | Ask any question | LLM searches the wiki, synthesizes an answer with citations, optionally creates a synthesis page for novel insights |
| **Lint** | "lint" or "health check" | LLM audits all pages for orphans, contradictions, missing links, incomplete sections, and low-confidence claims, runs `llm-wiki-site lint` on the writing — fixes what it can, reports the rest |

## Quick Start

1. **Clone this repo**

   ```bash
   git clone https://github.com/YOUR_USERNAME/llm-wiki.git my-knowledge-base
   cd my-knowledge-base
   ```

2. **Customize CLAUDE.md** for your domain
   - Update the Purpose section with your topic
   - Replace the placeholder tagging taxonomy with your own categories
   - Adjust confidence level descriptions if needed
   - Set your site branding in `site.config.json` (title, brand letters, footer, accent)
   - Everything else (workflows, page formats, linking rules) works as-is

3. **Drop sources into `raw/`**
   - Text files, transcripts, articles, notes — any plain text
   - These are immutable once added; the LLM never modifies them

4. **Tell the LLM to ingest**

   ```
   ingest raw/my-first-source.txt
   ```

   The LLM will create summary pages, concept pages, entity pages, cross-links, and update the index.

5. **Ask questions**

   ```
   What are the key differences between X and Y?
   ```

   The LLM answers from the wiki, citing specific pages.

6. **Run health checks**

   ```
   lint
   ```

   The LLM audits the wiki and fixes issues.

## Directory Structure

```
.
├── CLAUDE.md                      # Schema — the LLM's instructions
├── site.config.json               # HTML site branding (title, accent, ...)
├── raw/                           # Your source documents (immutable)
└── wiki/
    ├── index.md                   # Master catalog of all pages
    ├── log.md                     # Append-only activity log
    ├── dashboard.md               # Dataview dashboard (Obsidian)
    ├── analytics.md               # Charts View analytics (Obsidian)
    ├── flashcards.md              # Spaced repetition cards
    ├── summaries/                 # One page per source document
    ├── concepts/                  # Concept and framework pages
    ├── entities/                  # People, tools, organizations, etc.
    ├── syntheses/                 # Cross-cutting analyses and comparisons
    ├── journal/                   # Research/session journal entries
    │   └── template.md            # Journal entry template
    └── presentations/             # Marp slide decks
```

## Enhancements

This template includes several extras beyond the core wiki pattern:

### Static HTML Site ([`llm-wiki-site`](https://github.com/mindless-scribbles/llm-wiki-site))

Every wiki has a companion static website styled with the **DDC Reel** design
system ([artifact](https://claude.ai/artifact/QBTm2jQ8DC1f9bjhwiuDvw)): dark, monochrome, Syne / Space Mono / Hanken Grotesk,
one orange accent, a numbered catalog sidebar, typographic hero, and cross-linked pages.
The LLM regenerates it after every ingest, so the site never lags the wiki.

The generator is a **separate repo**, because a wiki normally lives in an Obsidian
vault and Obsidian Sync carries only `*.md`. Build machinery would never sync, and
thousands of generated HTML files have no business in a vault. So this folder stays
pure markdown, and the builder reads it from the outside:

```bash
git clone https://github.com/mindless-scribbles/llm-wiki-site.git
cd llm-wiki-site && npm link

llm-wiki-site build /path/to/this/wiki --out ~/sites/my-wiki
```

Branding (name, brand letters, footer, accent color) lives in `site.config.json`,
or — for a synced vault — in a `site:` block in `wiki/index.md` frontmatter, which
is the only form that travels with the notes. Concept pages can also carry bespoke
**interactive canvas visualizations**, kept in the builder repo alongside the site
registry. No dependencies, Node 18+, no network needed to build.

### Dataview Dashboard (`wiki/dashboard.md`)

Live queries that surface low-confidence pages, recent updates, concepts by tag, and pages with the most sources. Requires the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) Obsidian plugin.

### Charts View Analytics (`wiki/analytics.md`)

Visual analytics with pie charts, bar charts, and word clouds. Requires the [Charts View](https://github.com/caronchen/obsidian-chartsview-plugin) Obsidian plugin.

### Writing Rule (STE-80)

Pages follow the writing rules of [ASD-STE100](https://www.asd-ste100.org/) (Simplified Technical English), about 80% of the way: short steps that start with a verb, sentence and paragraph limits, plain words, and one word for each thing (a per-wiki Terminology table in `CLAUDE.md`). `llm-wiki-site lint` measures it and reports; the LLM fixes. The STE dictionary is deliberately not used: it rejects a domain's own terms.

### Diagrams (`add-diagram` skill)

Structure is drawn as Mermaid code blocks, using the UML diagram types: state diagrams for modes, sequence diagrams for an order of calls, component diagrams for connected parts. Obsidian renders them natively; `llm-wiki-site` renders them to inline SVG in the DDC Reel look when [`mmdc`](https://github.com/mermaid-js/mermaid-cli) is installed. The `add-diagram` skill (`.claude/skills/add-diagram/`) holds the rules: which type, top-down layout, the page's own words, one caption.

### Marp Slides (`wiki/presentations/`)

Create slide decks from markdown using [Marp](https://marp.app/). Drop presentation files in this directory.

### Research Journal (`wiki/journal/`)

Track your research sessions, experiments, or applied work with the included template. The LLM can reference journal entries when answering queries.

### Spaced Repetition (`wiki/flashcards.md`)

Flashcards in the format used by the [Spaced Repetition](https://github.com/st3v3nmw/obsidian-spaced-repetition) Obsidian plugin. Ask the LLM to generate flashcards from any wiki page.

### MCP Server

This repo works with Claude Code's MCP server capabilities. Point an MCP-compatible client at this repo and the LLM can read/write the wiki programmatically.

## Customizing for Your Domain

The schema in `CLAUDE.md` is domain-agnostic. To adapt it:

1. **Purpose** — Describe your knowledge domain in one paragraph
2. **Tagging taxonomy** — Replace placeholder categories with your own (e.g., for a cooking KB: `cuisine`, `technique`, `ingredient`, `equipment`)
3. **Confidence levels** — Adjust the descriptions to match your domain's evidence standards
4. **Entity types** — Update the entity page description to match what entities mean in your domain (people, tools, companies, etc.)
5. **Journal template** — Customize `wiki/journal/template.md` for your workflow

Everything else — page format, linking conventions, workflows, rules — is universal and works across domains.

## Example Domains

This template works for any knowledge-intensive topic:

- **Research notes** — papers, experiments, methodologies
- **Book analysis** — themes, characters, author techniques
- **Competitive analysis** — companies, products, market trends
- **Course notes** — lectures, readings, key concepts
- **Personal development** — frameworks, habits, book summaries
- **Technical documentation** — APIs, architectures, design patterns
- **Hobby deep-dives** — any subject you want to master

## License

MIT
