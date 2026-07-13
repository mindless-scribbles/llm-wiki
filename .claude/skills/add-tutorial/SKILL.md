---
name: add-tutorial
description: Turn a timecoded transcript/lecture page into an actionable step-by-step tutorial whose every step links to the exact transcript timecode — clickable as a heading link in Obsidian and a click-to-popover on the HTML site. Use when the user wants to break a transcript/lesson into steps, make a walkthrough or step-by-step, or "tie each step to a timecode".
---

# Add Tutorial (timecode-linked step-by-step)

Distill a **timecoded transcript** into an actionable, phased **step-by-step tutorial**
whose every step is anchored to the exact moment in the transcript. Each timecode is a
real Obsidian heading wikilink, so it jumps to that spot in Obsidian **and** renders as a
click-to-popover pill on the generated HTML site (the popover shows the transcript chunk
inline). The verbatim transcript stays untouched as the reference layer.

## When this applies

The source must be a page with **timecoded headings** — lines like `## 00:30` or
`## [00:30]` (and `## HH:MM:SS` past the hour), each followed by the text spoken at that
moment. These are usually lecture/video transcripts (often under `wiki/summaries/` or a
section folder). If a source has no timecoded headings, this skill does not apply.

Requires a `build-site.mjs` that supports timecode popovers (ships with the llm-wiki
template — look for `buildTimecodeIndex` / `TC_SCRIPT`). If it's missing, the tutorial
still works as plain text; copy the timecode feature from the template's `build-site.mjs`.

## The timecode contract (do not deviate)

1. **Transcript headings carry no brackets.** Obsidian cannot link to a heading containing
   `[` or `]` (reserved), so normalize `## [MM:SS]` → `## MM:SS` in the source transcript
   before linking. This is cosmetic and idempotent; site anchor slugs (`00-30`) are unchanged.
2. **Each step opens with one or more timecode wikilinks** of the exact form:
   `[[wiki/<transcript-target>#MM:SS|MM:SS]]`
   where `<transcript-target>` is the source page's path from the wiki root without `.md`
   (e.g. `summaries/lesson-04` or `Section02-.../04-Starting...`). Use `HH:MM:SS` past the hour.
   - The `MM:SS` after `#` **must exactly match** a transcript heading (same digits, same
     `MM:SS` vs `HH:MM:SS` form). A mismatch renders as a dead `tc-missing` pill.
   - The `wiki/` prefix disambiguates the transcript from the same-named tutorial file in
     Obsidian; `build-site.mjs` strips it when resolving. Use it on **all** links to the transcript.
3. **The tutorial declares its source** via a `transcript:` frontmatter key naming the same
   `<transcript-target>`. `build-site.mjs` only renders pills on pages that have this key.

## Procedure — one lesson

1. **Locate the source** transcript page and note its wiki-root target. Read it fully.
2. **De-bracket** its timecode headings in place: `## [MM:SS]` → `## MM:SS` (skip if already plain).
3. **Write the tutorial** at `wiki/tutorials/<same-relative-path-as-source>.md` (mirror the
   source's location under `wiki/`, same basename). Frontmatter:
   ```yaml
   ---
   title: "<NN — Title> · Step-by-Step"
   type: summary
   tags: [<domain tags>, tutorial]
   created: YYYY-MM-DD
   updated: YYYY-MM-DD
   sources: ["wiki/<transcript-target>.md"]
   transcript: "<transcript-target>"
   confidence: high
   ---
   ```
4. **Body structure:**
   - H1 title.
   - A short blockquote linking the transcript (`[[wiki/<transcript-target>|… full transcript]]`)
     and noting that each timecode is clickable (Obsidian heading link + site popover).
   - `**Goal:**` one line and `**Result:**` one line, then a `---`.
   - `## Phase N — …` sections grouping ~2–6 transcript chunks each. Every step is a numbered
     item starting with the timecode wikilink(s), then an imperative instruction. Bold key
     UI/menu names. Mark ⚠️ gotchas and `(optional)` / `(advanced/skippable)` where the source says so.
   - `## Related` with the full-transcript link plus Previous / Next tutorial where they exist.
5. **Be comprehensive and faithful** — cover the whole lesson in order, click-by-click. Ignore
   trailing filler chunks (repeated "See you next time", promo/subscribe lines, subtitle credits).
   Fix obvious ASR garbles in the prose, but keep the **timecodes** exact so the popover shows the raw source.
6. **Update `wiki/index.md`** — add/extend a "Tutorials — Step-by-Step" section linking the new
   tutorial (`[[tutorials/<relative-path>|…]]`), and prefix transcript links with `wiki/` for Obsidian.
7. **Append to `wiki/log.md`** and **rebuild**: `node build-site.mjs`.

## Procedure — many lessons at once

Do lesson-01 yourself as the **gold template**, confirm the format with the user, then fan out
one subagent per remaining lesson, each given: the source path, the gold-template path to copy,
the exact frontmatter values, and prev/next links. Instruct every agent to use only exact
transcript timecodes and to re-scan the transcript to verify each token. Then run the validation
below across all files and rebuild once.

## Validation (run before rebuilding)

Confirm every timecode token matches a real transcript heading:

```bash
python3 - <<'PY'
import re, os, glob
tut="wiki/tutorials"; src="wiki"   # adjust src root if transcripts live elsewhere
bad=0; tok=0
for f in glob.glob(tut+"/**/*.md", recursive=True):
    t=open(f,encoding="utf8").read()
    m=re.search(r'^transcript:\s*"?([^"\n]+?)"?\s*$', t, re.M)
    if not m: continue
    tr=os.path.join(src, m.group(1)+".md")
    heads=set(re.findall(r'^##\s+\[?(\d{1,2}:\d{2}(?::\d{2})?)\]?\s*$', open(tr,encoding="utf8").read(), re.M))
    toks=re.findall(r'\[\[wiki/[^#\]]+#(\d{1,2}:\d{2}(?::\d{2})?)\|', t)
    b=[x for x in toks if x not in heads]; tok+=len(toks); bad+=len(b)
    if b: print(f"  BAD in {os.path.basename(f)}: {sorted(set(b))[:10]}")
print(f"tokens={tok} bad={bad}")
PY
```

After `node build-site.mjs`, confirm zero dead pills:
`grep -ro 'tc-missing' site/tutorials | wc -l` → should be `0`.

## Notes

- The compact `@[MM:SS]` form is also accepted by `build-site.mjs` (site-only), but prefer the
  `[[wiki/…#MM:SS|MM:SS]]` wikilink so timecodes are clickable in Obsidian too.
- Never modify `raw/`. The transcript pages under `wiki/` are editable (only heading brackets change).
