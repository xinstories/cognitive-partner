---
name: recommend
description: Produce a tight, curated list of new content for the human to consume, derived from what the wikis show they care about. Use when the user says "recommend", "recommendations", "what should I read", "what should I watch", "suggest content", or asks for things to read/watch/follow next. Draws on knowledge/wikis/ to surface books, films, shows, YouTube videos, X/Twitter threads, articles, and creators worth following — weighted toward short/medium-form over books, toward recent/relevant content, and toward wikis that are recently updated or mature — each a piece that genuinely shaped its industry's discourse from a significant, respected creator, with a multi-line why, and logs the batch to knowledge/wikis/recommendations.md.
---

# Recommend

Turn what the wikis reveal about the human's interests into a **tight, high-signal list** of new things to consume. Quality over quantity — a short list they'll actually act on beats a long one they won't.

Read `CLAUDE.md` first for the vault schema.

## Step 1 — Read the interest signal

1. Read `knowledge/wikis/primary/index.md` for the catalog of concepts/people/sources.
2. Read `knowledge/wikis/primary/log.md` → the **Interest Evolution** table to see what's *currently* hot vs. fading. Weight recommendations toward the live edge of their interests, not just the all-time set.
3. **Rank the wikis by maturity & recency.** Prefer pages that are either recently updated or well-developed — these are where the human's attention actually is and where a recommendation lands best. Judge by:
   - the page's `updated:` frontmatter (recent = live interest),
   - its `sources:` count and link density (high = a mature, serious interest worth feeding),
   - corroboration from the Interest Evolution table (a cluster trending up beats one fading out),
   - **placement in `wikis/primary/`** — a page the human has manually starred into their primary folder is the strongest explicit signal they care about that topic; weight those most heavily.
   Bias the batch toward these high-maturity / recently-updated / starred pages. Thin stubs and stale pages get little or no weight.
4. Skim those top-ranked pages to ground recommendations in specifics (a real argument they engaged with, a thinker they returned to, an open question the page leaves).
5. Read `knowledge/wikis/recommendations.md` (if it exists) so you **don't repeat** anything already recommended.

## Step 2 — Choose recommendations

Assemble a tight batch (aim for **~6–10 items**, not an exhaustive dump). Each must be a real, identifiable piece of content the human hasn't already engaged with in the vault.

**Allowed types:** books, films, shows, YouTube videos, X/Twitter threads or articles, blog posts / essays / articles, podcasts, and creators worth following on any platform.

**Form-factor balance — important:** skew toward **short and medium-form** (videos, threads, essays, a creator to follow, one podcast episode). Books are allowed but should be the **minority** of the list — at most ~2–3 — since they're the highest-cost ask.

**Selection principles:**
- **Anchor each pick to a high-maturity / recently-updated wiki interest** — every recommendation should trace to something concrete in the vault, biased toward the pages ranked highest in Step 1.3.
- **Recent and relevant first** — skew hard toward current, of-the-moment content: a 2025–2026 essay, a recent talk or episode, an active creator publishing now. An older piece earns its place only if it's genuinely canonical for the theme and nothing current covers it as well. **Use web search to find what's new** in the relevant space, not just what you already know.
- **Content that has genuinely shaped the discourse** — this is the core bar. Recommend only pieces that have actually created or contributed to the conversation in their relevant industry or field: work that originated an idea, framed a debate, set a reference point others respond to, or moved the field — not derivative summaries, reaction content, explainers, or commentary on someone else's work. The piece itself must be a real contribution, not just *about* one.
- **Only significant, respected creators** — and from people who are industry-relevant, established, and well-regarded in the domain (a recognized researcher, journalist, author, founder, or operator with real standing). No anonymous blogs, no thin-credibility hot-takes, no content-mill output. If you can't say who the creator is, why they have standing, and how this specific piece contributed to the field's discourse, don't include it.
- **Stretch, don't just mirror** — mix confirming picks (deepen a known interest) with adjacent ones (a bridge into territory they're trending toward, per the Interest Evolution table). Avoid recommending the exact authors/sources already summarized; go one step outward.
- **Be specific** — name the actual video/essay/episode, not just "watch some Veritasium." Include the creator/author and platform so it's findable.
- **Verify currency** — confirm via web search that the piece exists, who made it, and that it's recent/accurately attributed before listing it. Don't invent titles or attributions.
- **Diversity** — vary platform, format, and viewpoint within the batch.

## Step 3 — Write the why (a few lines each)

For each item write a **short multi-line explanation** (roughly 3–5 sentences), not a one-liner. Cover three things:

1. **Who the creator is** — name them and their standing in one phrase (e.g. "Princeton CS professor and co-author of *AI Snake Oil*"), so the human knows why this voice is worth their time.
2. **The theme / main point** — what the piece actually argues or covers, in a couple of sentences. Enough that the human can decide to engage without opening it first.
3. **How it fits the vault** — name the specific wiki page(s) it connects to and the *relationship*: does it **deepen**, **supplement**, or **counter** what's already there? (e.g. "supplies the credible middle position that [[ai futures]] is missing between Amodei's optimism and the AI 2027 doom case.")

Keep it tight and substantive — informative, not padded.

## Step 4 — Log to recommendations.md

Maintain `knowledge/wikis/recommendations.md` as the running record (create it if absent).

- Prepend a new dated batch at the top (newest-first), headed `## [YYYY-MM-DD] recommendations`.
- Within a batch, group by form factor (Short/medium-form first, then Books) or list in priority order.
- Each entry: type, title, creator/author + their standing, platform/link if known, and the multi-line why from Step 3 (creator → theme → how it fits/supplements/counters a vault page).

**Bold the content type — never bracket it.** Write `**Video** —`, not `**[Video]**`: Obsidian reads `[text]` as a link and will spawn phantom pages.

Suggested entry format:
```
## [YYYY-MM-DD] recommendations

### Short & medium-form
- **Video** — "Title" — Creator (YouTube)
  Who: one phrase on the creator's standing. Theme: what it argues. Fits: deepens/supplements/counters [[wiki page]] by …
- **Essay** — "Title" — Author (site/publication)
  Who: … Theme: … Fits: …
- **Creator** — @handle (platform)
  Who: … What they cover: … Fits: why worth following given [[wiki page]].

### Books (fewer)
- **Book** — *Title* — Author (year)
  Who: … Theme: … Fits: …
```

If the file is new, start it with a short `# Recommendations` header and a one-line note that it's maintained by the `recommend` skill.

## Step 5 — Report

Present the batch to the human in chat (same content as logged), leading with the 2–3 you'd most push them toward and why. Note that it's saved to `recommendations.md`.
