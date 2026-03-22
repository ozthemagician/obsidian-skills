---
name: kb-query
description: Search and reason over an Obsidian knowledge base vault using multi-step agentic retrieval. Use when the user asks a question about their vault content, wants to find information across notes, compare sources, identify coverage gaps, or synthesize knowledge from multiple notes.
---

# KB Query

Multi-step retrieval strategy for answering questions from an Obsidian knowledge base vault. Instead of a single search, iteratively search, read, and follow links to build comprehensive context before answering.

## Retrieval workflow

Follow these steps in order:

1. **Parse the question** — identify key concepts, terms, and synonyms to search for
2. **Search** — run `obsidian search` with targeted queries for each key term
3. **Read top matches** — use `obsidian read` on the most relevant results
4. **Follow wikilinks** — read notes linked via `[[wikilinks]]` in the matched content to pull related context
5. **Check backlinks** — run `obsidian backlinks` on key notes to find referencing notes
6. **Broaden if needed** — if context is insufficient, search with synonyms, check `obsidian tags`, or browse `00-mocs/` for curated overviews
7. **Synthesize** — compose an answer citing specific notes as `[[Note Name]]` wikilinks
8. **Offer follow-up** — if the answer is substantial, offer to create a new note summarizing the findings

## Search strategy

Start narrow, then widen. Run multiple searches with different terms rather than relying on a single query.

```bash
# Targeted keyword search
obsidian search query="linearizability" limit=10

# Broader topic search
obsidian search query="consensus" limit=10

# Check what topics exist
obsidian tags sort=count counts

# Browse MOCs for curated overviews
obsidian read file="Distributed Systems MOC"

# Follow references
obsidian backlinks file="DDIA - Chapter 9 Consistency and Consensus"
```

## Conventions

- **Always cite sources as wikilinks** — e.g. `[[DDIA - Chapter 9 Consistency and Consensus]]` — so results are navigable in Obsidian
- **Distinguish source types** — book content is authoritative reference material, personal notes are contextual and experiential, articles are external perspectives
- **Surface conflicts** — when multiple sources disagree, state the disagreement explicitly and cite both sides
- **Respect the tag taxonomy** — use the vault's `topics` property values when categorizing or filtering results
- **Check MOCs first** — `00-mocs/` contains curated overviews that can shortcut the search process
- **Don't fabricate vault content** — only cite notes that actually exist in the vault. If the vault lacks coverage on a topic, say so
