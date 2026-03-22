---
name: kb-ingest
description: Ingest content into an Obsidian knowledge base vault with proper structure and frontmatter. Use when the user wants to add a PDF book, web article, personal note, or existing markdown into their vault with correct metadata, folder placement, and linking.
---

# KB Ingest

Pipeline for bringing new content into an Obsidian knowledge base vault with proper structure, frontmatter, and linking.

## Content types

### PDF book ingestion

1. Convert PDF to markdown (via Marker plugin with Mistral OCR, or Claude reads and restructures)
2. Create a parent book note under `01-books/<book-name>/` with metadata:
   ```yaml
   ---
   title: "Book Title"
   author: "Author Name"
   isbn: "978-..."
   type: book
   topics:
     - relevant-topic
   date-added: 2026-03-22
   status: ingested
   ---
   ```
3. Create one note per chapter, linked from the parent note: `[[Book - Chapter N Title]]`
4. Each chapter note includes frontmatter linking back:
   ```yaml
   ---
   title: "Book - Chapter N Title"
   type: chapter
   parent: "[[Book]]"
   topics:
     - relevant-topic
   date-added: 2026-03-22
   ---
   ```
5. Tag sections with the vault's topic taxonomy

### Web article ingestion

1. Use `defuddle` to extract clean markdown from URL
2. Create note under `03-articles/` with metadata:
   ```yaml
   ---
   title: "Article Title"
   author: "Author Name"
   source: "https://..."
   type: article
   topics:
     - relevant-topic
   date-added: 2026-03-22
   ---
   ```
3. Tag with appropriate topics from the vault taxonomy

### Personal note creation

1. Create note under `02-notes/` with metadata:
   ```yaml
   ---
   title: "Note Title"
   type: note
   topics:
     - relevant-topic
   date-added: 2026-03-22
   ---
   ```
2. Add `[[wikilinks]]` to related vault content

### Existing markdown import

1. Copy markdown into the appropriate vault folder based on content type
2. Add frontmatter if missing (`title`, `type`, `topics`, `date-added` at minimum)
3. Convert any markdown links to internal content into `[[wikilinks]]`

## Commands

```bash
# Create a note with content and frontmatter
obsidian create name="Note Title" content="---\ntitle: ...\n---\n\n# Content" path="01-books/DDIA/DDIA - Chapter 1.md" silent

# Use a template
obsidian create name="Note Title" template="book" silent

# Extract article content from URL
defuddle <url>

# Append content to existing note
obsidian append file="Book Parent Note" content="\n- [[Book - Chapter 5 Title]]"
```

## Conventions

- **Every note must have frontmatter** with at minimum: `title`, `type`, `topics`, `date-added`
- **Use the property schema** — `type` is one of: book, chapter, note, article, reference, moc
- **Topics use kebab-case** — e.g. `distributed-systems`, `event-driven`, `domain-driven-design`
- **Books get one parent note + one note per chapter** — parent links to all chapters, chapters link back via `parent` property
- **Wikilinks for internal, markdown links for external** — internal vault references use `[[wikilinks]]`, external URLs use `[text](url)`
- **Folder placement by type** — books in `01-books/<name>/`, notes in `02-notes/`, articles in `03-articles/`, references in `04-references/`
- **Use `silent` flag** when creating notes to avoid switching focus in Obsidian
