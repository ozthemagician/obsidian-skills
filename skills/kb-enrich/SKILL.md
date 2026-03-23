---
name: kb-enrich
description: Enrich ingested content in an Obsidian knowledge base vault by leveraging Obsidian CLI commands for backlinks, tags, search, properties, and file operations. Use when the user asks to connect notes, cross-reference content, audit vault quality, fix metadata, find orphans, or run maintenance on their knowledge base.
---

# KB Enrich

Enrich ingested content in an Obsidian knowledge base vault by discovering and creating connections through the vault's own index.

## Enrichment methodology

Always use Obsidian CLI commands as the primary discovery mechanism. Never guess at relationships — discover them through the vault's own index.

### Discovery commands

Use these CLI commands to understand the vault state before making changes:

```bash
# Find what already links TO a note (incoming references)
obsidian backlinks file="Note Name"

# Find notes matching a concept (keyword search across vault)
obsidian search query="consensus" limit=20

# List all tags in use with frequency
obsidian tags sort=count counts

# Read a specific note's full content
obsidian read file="Note Name"

# Read a note at a specific path
obsidian read path="01-books/DDIA/DDIA - Chapter 9.md"

# List files in a folder
obsidian files folder="01-books/"

# List all properties for a note
obsidian properties file="Note Name"

# Read a specific property value
obsidian property:read name="tags" file="Note Name"

# Find unresolved links (wikilinks pointing to non-existent notes)
obsidian unresolved

# Find notes with no incoming links
obsidian orphans

# Find notes with no outgoing links
obsidian deadends

# List outgoing links from a note
obsidian links file="Note Name"

# Search with matching line context
obsidian search:context query="consensus" limit=10
```

### Mutation commands

Use these to make changes:

```bash
# Set or update a frontmatter property
obsidian property:set name="tags" value="distributed-systems, consensus" file="Note Name"

# Append content to an existing note
obsidian append file="Note Name" content="## See also\n- [[Related Note]]"

# Create a new note
obsidian create name="New Note" content="---\ntitle: New Note\n---\n# Content"
```

## Enrichment workflows

### 1. Audit vault quality

**Goal:** Find broken, incomplete, or inconsistent content.

**Steps:**

1. Run `obsidian unresolved` to find broken wikilinks
2. Run `obsidian tags sort=count counts` to find tag inconsistencies
3. Run `obsidian files folder="01-books/"` then for each file run `obsidian properties file="<note>"` to check frontmatter completeness — verify required properties: `title`, `type`, `tags`, `date-added`
4. For each note missing required properties, infer values:
   - Determine type from folder location
   - Extract tags from content
   - Set properties with `obsidian property:set`
5. Report: which notes were fixed, which need manual attention (OCR issues, garbled content)

### 2. Cross-reference a single note

**Goal:** Connect one note to all relevant notes in the vault.

**Steps:**

1. `obsidian read file="Target Note"` — understand the content
2. Extract 3-5 key concepts from the note
3. For each concept, run `obsidian search query="<concept>" limit=10`
4. `obsidian backlinks file="Target Note"` — see what already links here
5. For each search result that is genuinely related and NOT already linked:
   - `obsidian read file="<result>"` to confirm relevance
   - `obsidian append file="Target Note" content="\n- [[Result Note]] — <one-line reason>" `
   - `obsidian append file="<Result Note>" content="\n- [[Target Note]] — <one-line reason>" `
6. Always add links bidirectionally — if A references B, B should reference A

**Important:** Before appending links, read the target note to check if a "See also", "Cross-references", or "Related" section already exists. Append to that section. If none exists, create one with `## See also` heading.

### 3. Cross-reference an entire folder

**Goal:** Connect all notes in a folder (e.g., all chapters of a book) to each other and to the rest of the vault.

**Steps:**

1. `obsidian files folder="<folder>/"` — list all notes in the folder
2. For each note, run workflow #2 (cross-reference a single note)
3. After processing all notes, run `obsidian backlinks` on each to verify bidirectional links were created
4. Check for clusters: if notes A, B, C all discuss the same concept but only A↔B and B↔C are linked, add A↔C

### 4. Connect personal notes to source material

**Goal:** Bridge the gap between theory (books/articles) and practice (personal notes).

**Steps:**

1. `obsidian files folder="02-notes/"` — list personal notes
2. For each personal note:
   - `obsidian read file="<note>"` — extract key concepts
   - `obsidian search query="<concept>"` — find matching book chapters and articles
   - `obsidian backlinks file="<note>"` — check what already links here
   - For each relevant book chapter/article not already linked:
     - Append to personal note under `## Source material`:
       `- [[Book Chapter]] — covers the theory behind this`
     - Append to book chapter under `## Applied in`:
       `- [[Personal Note]] — practical implementation`
3. The section names matter for navigability:
   - Book/article notes get `## Applied in` (pointing to practice)
   - Personal notes get `## Source material` (pointing to theory)
   - This makes the theory↔practice direction clear from either side

### 5. Topic cluster enrichment

**Goal:** Ensure all notes on a shared concept are fully interconnected.

**Steps:**

1. Pick a topic from the vault taxonomy (e.g., "consensus")
2. `obsidian search query="consensus" limit=30` — find all notes mentioning it
3. `obsidian search query="raft" limit=10` — search synonyms and subtopics
4. `obsidian search query="paxos" limit=10` — continue with related terms
5. Deduplicate results into a single list of relevant notes
6. For each note in the cluster:
   - `obsidian backlinks file="<note>"` — check existing connections
   - Identify which other cluster notes are NOT yet linked
   - `obsidian append` bidirectional links for missing connections
7. If the cluster has 5+ notes and no MOC exists for this topic:
   - Suggest creating one (don't auto-create — confirm with user)

### 6. Add structured metadata

**Goal:** Add queryable properties that make future searches more precise.

**Steps:**

1. `obsidian files folder="01-books/"` — iterate book chapters
2. For each note, `obsidian read` and analyze content to determine:
   - `perspective`: theoretical | practical | mixed
   - `questions`: 2-4 questions this note answers, written as a practitioner would ask
3. Set with `obsidian property:set`:
   ```bash
   obsidian property:set name="perspective" value="theoretical" file="Note Name"
   ```
4. For list-type properties like `questions`, read the note's frontmatter, modify it, and rewrite — `property:set` with `type=list` may not handle multi-value lists predictably. Verify behavior first or instruct the user to add list properties manually.

### 7. Find and fix orphans

**Goal:** Eliminate disconnected notes that can't be found through graph traversal.

**Steps:**

1. `obsidian orphans` — find notes with no incoming links
2. `obsidian deadends` — find notes with no outgoing links
3. Notes appearing in BOTH lists are fully disconnected — prioritize these
4. For each orphan:
   - Read its content to understand what it covers
   - `obsidian search` for related notes by key concepts
   - Add at least one outgoing wikilink to a related note
   - Add at least one incoming wikilink from a related note or appropriate MOC
   - If it fits a MOC, append it to that MOC's listing
5. Report: which orphans were connected, and where

### 8. Detect and resolve conflicts

**Goal:** Find notes that discuss the same concept differently and surface the disagreement.

**Steps:**

1. After cross-referencing (workflow #5), look for clusters where source notes disagree
2. `obsidian read` both conflicting notes
3. `obsidian append` a callout to the relevant notes:
   ```
   > [!conflict] Conflicting with [[Other Note]]
   > This note says X, while [[Other Note]] says Y. The difference may be because...
   ```
4. This is especially valuable across book sources — different authors frame the same concept differently, and surfacing that is more useful than hiding it

## Conventions

- **Bidirectional links always.** If you add a link from A→B, always also add B→A. Use `obsidian backlinks` to verify.
- **Section naming matters.** Use consistent section headings for appended links:
  - `## See also` — general cross-references
  - `## Cross-references` — links across different books
  - `## Source material` — on personal notes, pointing to theory
  - `## Applied in` — on book/article notes, pointing to practice
  - `## Related chapters` — within-book links
  - `## Related articles` — book chapters linking to articles
  - `## Deeper reading` — articles linking to book chapters
- **Check before appending.** Always `obsidian read` a note before appending to check if the target section exists and if the link is already there. Never create duplicate links.
- **Files don't open by default.** `create` and `append` don't open the file unless you pass the `open` or `newtab` flag — no special flag needed for batch operations.
- **Respect the taxonomy.** When setting tags, only use values from the taxonomy in CLAUDE.md. If a note needs a topic that doesn't exist, flag it for the user rather than inventing one.
- **Verify with backlinks.** After any enrichment pass, run `obsidian backlinks` on the modified notes to confirm the links registered correctly.
- **Incremental enrichment.** Don't try to enrich the entire vault at once. Work folder by folder, book by book. Each pass makes the next one more effective because there are more links to discover through.

## Common mistakes

| Mistake | Fix |
|---------|-----|
| Adding links without checking existing ones | Always `obsidian read` first to check for duplicate links and existing sections |
| One-directional linking | Every link must be bidirectional — add A→B and B→A |
| Inventing tags not in the taxonomy | Only use tags from CLAUDE.md taxonomy; flag new ones for user approval |
| Enriching entire vault at once | Work incrementally — folder by folder, book by book |
| Guessing at relationships | Use `obsidian search` and `obsidian backlinks` to discover — never assume |
| Creating MOCs without asking | Suggest MOC creation when cluster has 5+ notes, but confirm with user first |
| Manually checking each note for orphans | Use `obsidian orphans` and `obsidian deadends` to find disconnected notes efficiently |
