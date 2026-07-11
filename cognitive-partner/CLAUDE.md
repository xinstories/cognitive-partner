# 2nd-brain — Vault Guide

This is a personal Obsidian vault operated as an **LLM-maintained second brain**. You (the LLM) are the maintainer: you read sources, synthesize them, keep the knowledge base interlinked and current, and help the human reflect on their own life. The human curates sources, asks questions, and directs analysis. You do the bookkeeping.

This file is the **schema**: it tells you how the vault is structured, the conventions to follow, and which workflow to run when asked. Read it at the start of every session. Per-operation details live in skills (see *Operations*).

---

## Top-level structure

```
2nd-brain/
├── CLAUDE.md          ← this file (orientation + conventions)
├── knowledge/         ← things I'm learning about (the "wiki" half)
│   ├── raw/           ← immutable source dump — I control what goes here
│   └── wikis/         ← LLM-maintained synthesis — you own this entirely
│       └── primary/   ← my hand-picked "starred" wikis — I curate; you never move pages in/out
├── me/                ← personal context + journaling (the "self" half)
│   ├── context/       ← durable info about who I am
│   ├── journal/       ← daily notes + periodic to-date analyses
│   ├── classes/       ← DISREGARD
│   └── work/          ← DISREGARD
└── images/            ← shared attachment dump for both halves
```

---

## knowledge/ — the wiki

Two layers plus two meta-files.

### `raw/` (source of truth — NEVER modify the body)
My curated dump. **These files are immutable** — you read from them, you never rewrite their content. The only edit you may make is the frontmatter `status` property (see below).

**Subfolder breakdown:**

| Folder | What's in it | Status behavior |
|--------|--------------|-----------------|
| `clip/` | Web clippings of articles & transcripts | `new` → `processed` |
| `concepts/` | My own notes on concepts | `new` → `processed` |
| `media/` | My notes on books / movies / shows | `new` → `processed` |
| `people/` | My notes on figures (thinkers, writers) | `new` → `processed` |
| `rl/` | Reading list — queued, not yet read | **always `ignore`** |

(`raw/hubs/` also exists — legacy map-of-content notes; treat like `concepts/`.)

Every raw file should carry frontmatter:

```yaml
---
title: "..."           # source title (clips also carry source, author, created…)
status: new            # new | processed | ignore
wikis: []              # wiki pages this source fed, e.g. [mortality, presence]
---
```

**`status` values:**
- **`new`** — added but not yet folded into the wikis. This is what `ingest` scans for.
- **`processed`** — already synthesized into `wikis/`; `wikis:` lists the pages it fed.
- **`ignore`** — never ingest. **All of `rl/` is permanently `ignore`** (it's a to-read queue, not source material).

**Re-ingest rule:** if a `processed` raw file is later **edited** (you add new notes to it), flip its status back to **`new`** so the next `ingest` re-synthesizes it.

- Raw files are **never deleted**, even after processing.
- Find work for the next ingest: `grep -rl "^status: new" knowledge/raw`

> Note: `raw/concepts/`, `raw/hubs/`, and `raw/people/` are **legacy hand-written atomic notes** from before this system existed. Their content has already been absorbed into `wikis/`. Treat them as raw sources (don't rewrite), but prefer the `wikis/` versions as the live synthesis. Candidate for relocation to an `archive/` — confirm with the human first.

### `wikis/` (synthesis — you own this layer)
LLM-generated, interlinked markdown. One concept / person / source per page (atomic). You create pages, update them on ingest, maintain cross-references, and keep them consistent. The human reads this; you write it.

**Page schema** (keep pages atomic — one idea per file):
1. `# Title` (the concept, lowercase filename)
2. One-sentence definition up top.
3. `## Core idea` — the synthesis, in depth. Detailed enough that the human never needs to reopen the raw source for a refresher. **Cite sources inline here** (see *Source citation* below).
4. `## Connections` — `[[wikilinks]]` to related wiki pages, each with a sentence on *how* they relate (not a bare link). **Wiki-to-wiki only — never cite a raw source here.**
5. When a cross-cutting term recurs across pages, **give it its own page and link to it** rather than re-explaining it inline (atomic-note principle).

**Source citation — integrate, don't append.** Do **not** collect sources in a trailing `## Key sources` list. Instead, link each raw source **inline, at the exact point in `## Core idea` where its material is used**, with an Obsidian wikilink to the raw file — e.g. "…as [[Elon Musk's Shadow Rule]] documents, a single private actor controlled battlefield connectivity…". Rules:
- **A source is cited only in the atomic note that actually synthesizes it** — its content home. Never in a `## Connections` section, and never re-cited by another page that merely links to this one. This keeps each raw file's *backlinks* pointing to the page(s) that truly draw on it, so opening a raw article in Obsidian shows exactly what it fed.
- A source that genuinely feeds several pages is cited inline on each — wherever its substance actually appears.

**Outside-vault content — mark it.** Synthesis should come from `raw/` sources. Where a page fills a gap with the model's own world knowledge instead (a stub source with no notes, background the sources don't cover), denote that claim inline with a trailing `*(outside vault)*`, so the human can always tell their own material from model-supplied content. `ingest`, `carve`, and `lint` all enforce this.

**Wikilink convention — IMPORTANT:** use **bare links** for wiki-to-wiki references (`[[mortality]]`, `[[camus]]`), never full-path links (`[[knowledge/wikis/camus]]`). Obsidian resolves bare links by filename, and its `alwaysUpdateLinks` setting has historically rewritten some links to fragile full paths — normalize back to bare on any page you touch. **Exception for raw-source citations:** prefer a bare link to the raw file (`[[Elon Musk's Shadow Rule]]`), but if the raw filename collides with a wiki page name (common for legacy `raw/concepts`, `raw/people`, `raw/hubs` notes that share a title with their wiki page), qualify the path just enough to target the raw file (e.g. `[[raw/people/nietzsche]]`) so the backlink lands on the source, not the wiki page.

**Wiki page frontmatter** (enables Dataview / Bases queries — add going forward):
```yaml
---
type: concept        # concept | person | source-summary
updated: 2026-06-05
sources: 3           # count of raw sources feeding this page
---
```

### `primary/` (my starred wikis — human-curated location)
`wikis/primary/` is a **manually controlled "favorites" folder**. I move wiki pages I consider especially good — strong LLM-written pages, or pages produced by `carve` — into it so I can find my go-to references quickly. Pages here are still ordinary wiki pages; the folder just marks them as *mine, curated, canonical*.

Rules for you:
- **Never move pages into or out of `primary/`.** Placement is my decision alone. Don't relocate, promote, or demote anything here.
- **Do keep them current.** You may read and *update the content* of a primary page in place (e.g. `ingest`/`carve` folding in new material) — just don't move the file.
- **A page lives in exactly one place.** If a topic already has a page in `primary/`, update *that* page; never create a second copy of it in `wikis/` root. Watch for duplicate filenames across root and `primary/` (Obsidian link resolution goes ambiguous) — flag them on `lint`.
- Location doesn't affect links: bare `[[wikilinks]]` resolve by filename, so a page works the same in `primary/` or root.

### Meta-files
- **`wikis/index.md`** — content catalog. Every page listed with a one-line summary, grouped by type (Concepts / People / Source Summaries / Raw Sources). **Read this first** when answering a query to find relevant pages, then drill in. Update it on every `ingest` and `carve`.
- **`wikis/log.md`** — append-only chronological record. Prefix every entry `## [YYYY-MM-DD] <op> | <title>` so it stays greppable (`grep "^## \[" log.md | tail -5`). Log every `ingest`, `carve`, and `lint` pass. It also holds the **Interest Evolution (to-date)** visual — a heatmap of the human's *broad* interests (not atomic concepts) across calendar periods, showing how their reading focus shifts over time. **`ingest` maintains it** (and `carve` may nudge the current period's column when it adds a topic): each pass re-weights the current column and adds a new column when the period rolls over.

---

## me/ — context + journaling

The goal: keep enough durable context about the human that you can spot patterns in their thinking, ask sharper follow-up questions, and challenge their decisions and assumptions usefully.

- **`context/`** — durable, slow-changing facts: who they are, interests, situation, values. (Currently holds `blog/` essays; a top-level `profile.md` would help — see backlog.) Read this before any analysis so your questions are grounded.
- **`journal/`** — Obsidian daily notes (`YYYY-MM-DD.md`) where they write whatever is on their mind. **Never edit these.**
- **To-date (TD) analyses** — the **`analyze`** operation: periodically you read all context + journal to date and produce a `YYYY-MM-DD TD analysis.md` file in `journal/`. These are deep, structured, and challenge the human's framing. Start each with a `linked to:` line naming the entries it draws on.

---

## Operations

Run these when asked. Each has (or will have) a dedicated skill with the detailed workflow.

| Operation            | Scope      | What it does                                                                                                                                                                                                                                              |
| -------------------- | ---------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ingest**           | knowledge/ | Read `status: new` raw files, synthesize into new/existing wiki pages, update cross-refs + `index.md`, append to `log.md`, flip raw `status` → `processed`. Only touches pages it creates/updates.                                                        |
| **carve**            | knowledge/ | Top-down counterpart to ingest: build/expand one atomic wiki page on a topic the human names, drawing on all raw (new + processed) + world knowledge. Auto-adds cross-links; *proposes* inline-content migrations. Reminds to run `ingest` if backlog remains. |
| **lint**             | knowledge/ | Health-check *all* wiki pages: contradictions, stale claims, orphan pages, missing cross-refs, concepts lacking their own page, data gaps a web search could fill. Suggest new questions/sources. Log the pass.                                           |
| **analyze**          | me/        | Read all context + journal to date; surface patterns, ask follow-up questions, challenge assumptions; write a TD analysis into `journal/`.                                                                                                                |
| **recommend**        | knowledge/ | Read the wikis (and Interest Evolution) to produce a tight, curated list of new content to consume — books, films, shows, videos, threads, articles, creators — skewed to short/medium-form, each with a one-line why. Log to `wikis/recommendations.md`. |

## Hard rules
- **Never modify the body of anything in `raw/` or any `journal/` entry.** Raw `status` frontmatter is the only exception.
- You write `wikis/`; the human writes `raw/` and `journal/`.
- **`wikis/primary/` is human-curated** — update page *content* there in place, but never move pages in or out of it.
- Bare wikilinks only.
- `ingest` only touches what it creates/updates; `carve` builds one page and links its neighbors but only *proposes* larger content migrations; `lint` may touch everything.
- Ignore `me/classes/` and `me/work/`.
