---
name: ingest
description: Read new raw sources in the knowledge/ half of the vault and synthesize them into the LLM-maintained wiki. Use when the user says "ingest", "read new sources", "process new sources", "update the wikis", or otherwise asks to fold freshly-added clippings, notes, books, or articles into wikis/. Scans raw files with status:new, writes/updates atomic wiki pages, maintains cross-links, refreshes the catalog and log (index.md and log.md, both in wikis/primary/), and flips processed sources to status:processed.
---

# Ingest

Turn `status: new` raw sources into durable, interlinked synthesis in `knowledge/wikis/`. You own the `wikis/` layer; `raw/` is immutable except for its `status`/`wikis` frontmatter.

Read `CLAUDE.md` first — it is the vault schema. This skill is the per-operation workflow.

## Step 0 — Clean & update the status property (do this FIRST)

Before reading anything for synthesis, get the work queue right. The valid values are **`new`**, **`processed`**, and **`ignore`**.

1. **Normalize missing status.** Newly-added raw files arrive with no `status` property. Sweep every file under `knowledge/raw/` and give each a status:
   - A file under `raw/rl/` (the reading list) with no/null `status` → set `status: ignore`. All of `rl/` is permanently `ignore`.
   - Any other file with no/null `status` → set `status: new`.
   - Find offenders: files lacking a `status:` line. e.g. `for f in $(grep -rL "^status:" knowledge/raw --include=*.md); do echo "$f"; done` (those under `raw/rl/` get `ignore`, the rest get `new`).
2. Find candidates: `grep -rl "^status: new" knowledge/raw`
3. Re-ingest sweep — a `processed` file that the human has since **edited** should be re-synthesized. If a file's body was modified after the date in its last ingest (check `git status`/`git diff` if available, or the human will tell you), flip its `status: processed` → `status: new` so it re-enters the queue.
4. Confirm `rl/` is untouched — every file under `raw/rl/` is permanently `status: ignore`. Never ingest it.
5. The resulting set of `status: new` files is your work list for this pass.

Only the `status` (and later `wikis:`) frontmatter may be edited on a raw file. **Never touch the body.**

## Step 1 — Read each new source

For every `status: new` file, read it in full. Note: the core claims, who/what it's about, which existing wiki concepts it touches, and any genuinely new concept that deserves its own page.

## Step 2 — Synthesize into wiki pages (evergreen-note discipline)

Write `wikis/` pages as **Andy Matuschak-style evergreen notes**. These properties are the quality bar:

- **Atomic** — one concept per page. If a source spans five ideas, it feeds five pages, not one dumping-ground note. Give a recurring cross-cutting term its *own* page and link to it rather than re-explaining it inline.
- **Concept-oriented, not source-oriented** — title and organize by *idea*, not by where it came from. A book touching ten concepts updates ten concept pages; the book itself gets one `(summary)` page. Don't file knowledge under its source.
- **Densely and meaningfully linked** — every page connects to related pages, and **each link carries a sentence on *how* they relate**, never a bare drop. Links are the value; the act of choosing them is the thinking.
- **Titles as claims/handles** — prefer titles that state an idea you can build on. A good title is one you'd be happy to link to from many places.
- **Self-sufficient depth** — the `## Core idea` must be detailed enough that the human never needs to reopen the raw source for a refresher.
- **Write for your future self** — favor clarity and findability over cleverness; assume the reader has forgotten the context.

**Cite sources inline, not in a trailing list.** Link each raw source at the point in `## Core idea` where you use its material, with an Obsidian wikilink to the raw file (`[[Elon Musk's Shadow Rule]]`). No `## Key sources` section. A source is cited only in the atomic note that synthesizes it — never in `## Connections`, never re-cited by a page that merely links here — so the raw file's Obsidian backlinks point to exactly the pages it fed. Prefer a bare link to the raw file; qualify the path (e.g. `[[raw/people/nietzsche]]`) only when the raw filename collides with a wiki page name. See *Source citation* in CLAUDE.md.

**Mark outside-vault content as external.** Synthesis should come from the raw sources. Where you fill a gap with the model's own knowledge (e.g. a stub source with no notes, or background the sources don't cover), make it explicit and traceable — the same convention `carve` and `lint` use: denote the substantive Core-idea claim inline with a trailing `*(outside vault)*`. The human must always be able to tell their own material from model-supplied content.

**Page schema** (see CLAUDE.md):
1. `# Title` (lowercase filename)
2. One-sentence definition.
3. `## Core idea` — the synthesis, in depth, with raw sources cited inline as `[[wikilinks]]` to the raw files.
4. `## Connections` — annotated `[[wikilinks]]` to related wiki pages (wiki-to-wiki only; no raw-source links here).

Add wiki-page frontmatter going forward:
```yaml
---
type: concept        # concept | person | source-summary
updated: <today>
sources: <n>         # count of raw sources feeding this page
---
```

**Create vs. update:** prefer updating an existing page over spawning a near-duplicate. Create a new page only for a genuinely distinct atomic concept, person, or source-summary. **Check `wikis/primary/` as well as `wikis/` root** — if the page already exists in `primary/` (the human's starred folder), update *that* page in place. Never create a second copy in root, and never move a page into or out of `primary/`; placement there is the human's call.

**Wikilinks:** bare only — `[[mortality]]`, never `[[knowledge/wikis/mortality]]`. Normalize any full-path links you encounter back to bare.

## Step 3 — Close the loop on each source

For every file you finished synthesizing:
- Set `status: processed`.
- Fill `wikis: [page-a, page-b]` with the pages it fed.

## Step 4 — Update the meta-files

> **Location:** both meta-files live in **`knowledge/wikis/primary/`** — `wikis/primary/index.md` and `wikis/primary/log.md` — *not* in `wikis/` root. Edit them in place (updating content in `primary/` is allowed; only *moving* pages in/out of `primary/` is forbidden). If a copy ever appears in `wikis/` root, it's a stray duplicate — merge it into the `primary/` copy and delete the root one.

- **`wikis/primary/index.md`** — add any new pages with one-line summaries in the right section (Concepts / People / Source Summaries / Raw Sources). Move newly-processed raw sources into the catalog.
- **`wikis/primary/log.md`** — append an entry headed `## [YYYY-MM-DD] ingest | <short title>` describing scope, pages created/updated, cross-refs, and notable stubs or deferrals.
- **`wikis/primary/log.md` → Interest Evolution table** — re-weight the current calendar period's column to reflect what this batch of sources was about (broad interests, not atomic concepts). If the calendar period has rolled over since the last column, add a new column. Update the **Trajectory** line if the trend shifted.

## Step 5 — Report

Summarize for the human: what was ingested, which pages were created vs. updated, any sources you skipped and why, and any new questions or gaps the sources surfaced.
