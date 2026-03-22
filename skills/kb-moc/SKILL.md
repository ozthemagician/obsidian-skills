---
name: kb-moc
description: Generate and maintain Maps of Content (MOCs) in an Obsidian knowledge base vault. Use when the user wants to create a new MOC, update an existing MOC with recently added notes, identify missing MOCs, or get a curated overview of a topic area in their vault.
---

# KB MOC

Generate and maintain Maps of Content (MOCs) — curated index notes that organize vault content by topic with grouped wikilinks, cross-references, and gap analysis.

## Workflow

### Creating a new MOC

1. **Identify scope** — accept a topic from the user, or identify clusters from `obsidian tags` and folder contents
2. **Search the vault** — find all notes related to the topic using `obsidian search`, `obsidian tags`, and `obsidian backlinks`
3. **Read and classify** — read matched notes to understand their content and group by sub-theme
4. **Generate the MOC** under `00-mocs/` with this structure:
   ```yaml
   ---
   title: "Topic MOC"
   type: moc
   topics:
     - relevant-topic
   date-added: 2026-03-22
   last-updated: 2026-03-22
   ---
   ```
   Followed by:
   - Brief overview of the topic (2-3 sentences)
   - Grouped sections with wikilinks to related notes, each with a one-line description
   - **Related MOCs** section cross-referencing other MOCs where topics overlap
   - **Gaps** section noting areas with thin or missing coverage
5. **Use `silent` flag** when creating to avoid switching focus in Obsidian

### Updating an existing MOC

1. Read the existing MOC with `obsidian read`
2. Search for notes added since `last-updated`
3. Add new notes to the appropriate sub-theme group
4. Update the `last-updated` property
5. Revise the **Gaps** section based on new coverage

## MOC structure example

```markdown
# Consensus algorithms

Overview of consensus mechanisms in distributed systems, covering theory and practical implementations.

## Foundational theory

- [[DDIA - Chapter 9 Consistency and Consensus]] — Kleppmann's treatment of linearizability, ordering, and the FLP impossibility result
- [[Raft Paper Notes]] — Understandable consensus, leader election, log replication

## Practical implementations

- [[etcd Consensus Notes]] — Raft in practice, leader leasing
- [[Kafka KRaft Migration]] — From ZooKeeper to Raft-based metadata

## Related MOCs

- [[Distributed Systems MOC]]
- [[Replication and Consistency MOC]]

## Gaps

- No notes yet on Viewstamped Replication
- Byzantine fault tolerance coverage is thin
```

## Commands

```bash
# Search for notes by topic
obsidian search query="consensus" limit=20
obsidian tags sort=count counts

# Read notes to classify them
obsidian read file="DDIA - Chapter 9 Consistency and Consensus"

# Find related notes via backlinks
obsidian backlinks file="Raft Paper Notes"

# Create the MOC
obsidian create name="Consensus Algorithms MOC" content="..." path="00-mocs/Consensus Algorithms MOC.md" silent

# Update an existing MOC
obsidian append file="Consensus Algorithms MOC" content="- [[New Note]] — description"
```

## Conventions

- **MOCs live in `00-mocs/`** — named as `<Topic> MOC.md`
- **Every entry has a one-line description** — not just a bare wikilink, but context for what the note covers
- **Group by sub-theme** — don't just list notes alphabetically; organize them into meaningful sections
- **Cross-reference other MOCs** — if topics overlap (e.g. consensus and distributed systems), link to related MOCs
- **Always include a Gaps section** — identify areas where vault coverage is thin to guide future ingestion
- **Update `last-updated`** — always set this property when modifying a MOC
- **Distinguish source types in descriptions** — note whether a link points to book content, personal notes, or articles when it adds useful context
