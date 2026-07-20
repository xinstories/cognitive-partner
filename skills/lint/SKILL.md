---
name: lint
description: Health-check the entire LLM-maintained wiki in knowledge/wikis/. Use when the user says "lint", "check wikis", "clean wikis", "check health", or asks to audit the knowledge base for problems. Scans every wiki page for contradictions, stale claims, orphan pages, broken or full-path wikilinks, missing cross-references, recurring concepts that lack their own page, unattributed outside-vault content, sources not cited inline, and data gaps a web search could fill; proposes fixes and new questions; logs the pass.
---

# Lint

A whole-vault health check on `knowledge/wikis/`. Unlike `ingest` (which only touches what it creates), **lint may read and touch every page**. Read `CLAUDE.md` first for the schema and conventions.

## What to check

Walk all of `wikis/` and flag:

1. **Contradictions** — two pages making incompatible claims. Surface both, propose a reconciliation.
2. **Stale claims** — assertions that time, newer sources, or known facts have overtaken. Flag for update; the `updated:` frontmatter date can help judge staleness.
3. **Orphan pages** — pages nothing links to, and pages that link out to nothing. Wire them into the graph with annotated links, or question whether they should exist.
4. **Broken / full-path wikilinks** — links to nonexistent pages, and any `[[knowledge/wikis/...]]` full-path links. **Normalize all wikilinks to bare** (`[[camus]]`). `grep -rn "\[\[knowledge/wikis/" knowledge/wikis` finds the full-path offenders.
5. **Missing cross-references** — pages that clearly relate but don't link each other. Add annotated `[[links]]` (each with a sentence on *how* they relate).
6. **Bare (unannotated) links** — links dropped without explaining the relationship. Annotate them.
7. **Concepts lacking their own page** — a cross-cutting term explained inline in several pages should be promoted to its own atomic page that the others link to.
8. **Atomicity violations** — sprawling pages covering several ideas; propose a split.
9. **Schema drift** — pages missing the standard sections (definition, `## Core idea`, `## Connections`) or the `type/updated/sources` frontmatter. Note: the current schema has **no `## Key sources` section** — sources are cited inline (see check 14).
10. **Index / log accuracy** — `wikis/primary/index.md` entries that no longer match reality (missing pages, wrong summaries, stale Raw-Sources catalog). Both meta-files (`index.md`, `log.md`) live in **`wikis/primary/`**, not `wikis/` root; a copy in root is a stray duplicate to flag and merge back.
11. **Data gaps a web search could fill** — open questions, "[unknown]" notes, or thin sections where a quick search would add real substance. Note them (and optionally fill them, citing the source).
12. **Duplicate pages across `primary/` and root** — the same page existing in both `wikis/` root and `wikis/primary/` makes Obsidian's bare-link resolution ambiguous. Flag any filename that appears in both; the human's `primary/` copy is canonical, so propose deleting or merging the root duplicate into it (don't move the `primary/` page).
13. **Unattributed outside-vault content** — substantive claims that read as the model's own knowledge rather than anything a vault source supports, but carry no marker. The convention: world knowledge must be denoted `*(outside vault)*` inline at the claim. Flag unmarked external content so the human can tell their own material from model-supplied content; add the marker where clearly warranted, and propose (don't silently apply) the marker where it's a judgment call.
14. **Source-citation drift** — the current convention cites raw sources **inline in `## Core idea`** as `[[links]]` to the raw file, with no trailing `## Key sources` list. Flag legacy pages that still carry a `## Key sources` block, and any raw-source link sitting in `## Connections` (which is wiki-to-wiki only). Migrate them: move each source link to the point in the body where its material is used, then delete the trailing list — this is what makes the raw file's Obsidian backlinks meaningful. Also apply the raw-link disambiguation rule (bare link, or a path like `[[raw/people/nietzsche]]` on a filename collision).

## How to run

1. Build the page inventory: list `wikis/*.md` **and `wikis/primary/*.md`** (the human's starred folder — health-check those pages too), read `wikis/primary/index.md` for the catalog. **Never move pages into or out of `primary/`** — you may fix their content, but placement is the human's call.
2. Read pages (batch related ones) and collect issues by the categories above.
3. **Fix the safe, mechanical things directly**: bare-link normalization, obvious broken links, missing cross-refs, index corrections.
4. **Propose, don't unilaterally rewrite, the judgment calls**: contradictions, page splits, promotions, large content changes — present these to the human with a recommendation.
5. Suggest **new questions and sources**: what the vault is missing, what to read next, what to interrogate further.

## Close out

- Apply the agreed fixes.
- Append a `wikis/primary/log.md` entry headed `## [YYYY-MM-DD] lint | <scope>`: pages scanned, issues found by category, what was auto-fixed, what's pending human decision, and suggested follow-ups.
- Report a concise summary to the human, leading with anything that needs their call.
