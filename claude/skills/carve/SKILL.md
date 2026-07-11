---
name: carve
description: Commission a wiki page on a specific topic the human names, top-down — the counterpart to ingest (which is bottom-up from new sources). Use when the user says "carve", "carve out a page/wiki on X", "create a wiki on X", "make a page about X", "write a wiki about X", or "zoom into X" (a concept, a person, or a specific book). Gathers everything in the vault touching that topic plus world knowledge, builds or expands one atomic page for it, auto-adds cross-links from related pages, and proposes (does not perform) migrating inline content onto the new page. Reminds the human to run ingest if new raw remains.
---

# Carve

Direct, top-down page creation. Where `ingest` starts from new sources and asks "what pages should these become?", **`carve` starts from a topic the human names and asks "what page should exist for this, and what in the vault feeds it?"** You own the `wikis/` layer; `raw/` and `journal/` stay immutable except raw `status`/`wikis` frontmatter.

Read `CLAUDE.md` first — it is the vault schema. This skill is the per-operation workflow.

## Input

A topic the human names, optionally scoped:
- a **concept** — e.g. "the American Dream" → a `concept` page.
- a **person** — e.g. "carve out Hannah Arendt" → a `person` page.
- a **source** — e.g. "zoom into *The Stranger*" → a `source-summary` page (go deep on one book/film/essay).

If the topic is ambiguous or could be scoped several ways, ask one clarifying question before building.

## Step 1 — Gather evidence

Assemble everything the page should be built from, in this order:

1. **Existing wikis** — is there already a page (or a thin stub) for this topic? If so, this is an *expansion*, not a fresh create. Search `wikis/` root **and `wikis/primary/`** (the human's starred folder); if the page lives in `primary/`, expand *that* page in place — never create a duplicate in root, and never move it out of `primary/`. Also find every page that mentions the topic inline (`grep -ril "<topic>" knowledge/wikis`) — these are both evidence and reconciliation candidates for Step 3.
2. **Raw sources** — search all of `raw/`, both `status: new` **and** `status: processed` (`grep -ril "<topic>" knowledge/raw`). Carve is allowed to pull from already-processed sources; that's how it amends past ingests. Skip `rl/` (ignore).
3. **World knowledge** — fill the gaps the vault doesn't cover so the page is self-sufficient. Stay consistent with the framing the human's sources have established. **Clause — always mark world knowledge as external:** wherever a page draws on the model's own knowledge rather than a vault source, make that explicit and traceable. Where a substantive claim in `## Core idea` rests on world knowledge rather than a source the human fed in, denote it inline with a trailing `*(outside vault)*`. The human must always be able to tell which parts of a carved page came from their own material and which the model supplied.

## Step 2 — Build the page

Create the new page (or substantially expand the existing thin one) following the wiki page schema in `CLAUDE.md`:
1. `# Title` (lowercase filename; source pages use the `(summary)` suffix convention).
2. One-sentence definition.
3. `## Core idea` — the synthesis, in depth, self-sufficient enough that the human never needs the raw source to refresh. **Cite each raw source inline here** with an Obsidian wikilink to the raw file (`[[Elon Musk's Shadow Rule]]`) at the point its material is used — no trailing `## Key sources` list. Cite a source only where its substance appears, never in `## Connections`, so the raw file gets clean backlinks. Prefer a bare link; qualify the path (`[[raw/people/nietzsche]]`) only on a filename collision. See *Source citation* in CLAUDE.md.
4. `## Connections` — annotated `[[wikilinks]]` to related wiki pages (each with a sentence on *how* it relates; wiki-to-wiki only, no raw-source links here).

Add frontmatter: `type` (concept | person | source-summary), `updated:` (today), `sources:` (count of raw sources feeding it). Bare wikilinks only.

## Step 3 — Reconcile the graph (link automatically, propose migrations)

The atomic-note payoff: a cross-cutting topic should live on *its own* page, with other pages linking to it rather than re-explaining it inline.

- **Do automatically:** add an annotated `[[link]]` to the new page from every related page found in Step 1 (and add reciprocal links in the new page's `## Connections`). This is safe, additive work.
- **Propose only, do not perform:** where another page currently explains this topic *inline*, identify the specific passages that now belong on the new page and should be trimmed to a link. **List these as concrete proposals** (page, the passage, what it'd be replaced with) and let the human approve before you move anything. Never silently gut an existing page.

## Step 4 — Close the loop on raw

For any `status: new` raw file you actually drew from, set it to `status: processed` and record the pages in its `wikis:` frontmatter. Leave raw you didn't touch alone. Never edit a raw body.

## Step 5 — Update the meta-files

- **`index.md`** — add the new page (or update its summary) in the right section.
- **`log.md`** — append `## [YYYY-MM-DD] carve | <topic>`: what was created/expanded, which sources fed it, links added, and the migrations you proposed.
- **`log.md` → Interest Evolution table** — if the carved topic reflects a live interest, nudge the current period's weighting (same as ingest).

## Step 6 — Backlog check

After carving, check for remaining work: `grep -rl "^status: new" knowledge/raw`. If any `status: new` files remain that you didn't consume, **tell the human they still have a backlog and suggest running `ingest`** to sweep the rest as usual. Carve deliberately does not ingest the remainder itself.

## Step 7 — Report

Summarize: the page created/expanded, what fed it (vault sources vs. world knowledge), the cross-links added, the **migration proposals awaiting approval**, and whether an `ingest` is still owed for leftover backlog.
