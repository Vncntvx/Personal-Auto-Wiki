
<h1 align="center">Personal Auto Wiki</h1>

<p align="center" style="color: #888; font-size: 0.95em; margin-top: -0.5em; margin-bottom: 0.5em;">
  <a href="README_CN.md">中文</a> | <b>English</b>
</p>

Personal knowledge-base system that encapsulates core workflows as Skills to automatically transform reusable source materials into a structured Wiki. Paired with the `qmd` MCP search engine, it enables automated ingestion, intelligent querying, and health maintenance for long-term knowledge preservation and efficient retrieval.

> Project article: https://blog.wenjiexu.site/en/posts/llm-personal-wiki/

## Overview

### Core capabilities

- 🔄 **Automatic sync detection** — detect new, modified and deleted files under the `ObsidianRaw/03_Resources/` directory
- 📥 **Intelligent ingestion** — extract concepts and entities from source materials, produce concise summaries, and build indexes
- 🔍 **Hybrid retrieval** — combine lexical (keyword) search and vector (semantic) search for accurate recall
- 📊 **Health checks** — detect orphan pages, concept gaps, contradiction annotations, cognitive quality, and sync issues

---

## Architecture

### Directory structure

```text
PersonalWiki/
├── ObsidianRaw/                  # Raw source materials (organized with the PARA method, read-only)
│   ├── 00_Inbox/                 # Inbox (temporary, not processed)
│   ├── 01_Projects/              # Projects (action-oriented, not processed)
│   ├── 02_Areas/                 # Areas (ongoing responsibilities, not processed)
│   ├── 03_Resources/             # Resources ← the only ingestion source
│   └── 04_Archive/               # Archive (historical records, not processed)
├── Wiki/                         # The knowledge layer (maintained by the AI Agent)
│   ├── Index.md                  # Navigation index for the Wiki
│   ├── Log.md                    # Operation log (machine-parsable)
│   ├── Concepts/                 # Concept pages (theories, frameworks)
│   ├── Entities/                 # Entity pages (people, organizations, tools)
│   ├── Sources/                  # Source summaries (tracks original file status)
│   └── Outputs/                  # Analytical outputs (high-value synthesis from queries)
├── .claude/skills/               # Skills definitions (implemented workflows)
│   ├── syncing-wiki/             # Full sync workflow
│   ├── ingesting-resources/      # Ingesting resources workflow
│   ├── querying-wiki/            # Querying the Wiki workflow
│   ├── checking-wiki-health/     # Health checking workflow
│   └── detecting-resources-sync/ # Resources sync detection workflow
├── AGENTS.md                     # Comprehensive operational specification
├── VERSION                       # Architecture specification version
└── README.md                     # This file
```

### Ingestion scope

**Only files under `ObsidianRaw/03_Resources/` are ingested.**

| Folder | Purpose | Processed? |
|--------|---------|------------|
| `00_Inbox/` | Temporary inbox, needs curation | ❌ |
| `01_Projects/` | Action-oriented project notes | ❌ |
| `02_Areas/` | Ongoing responsibility notes | ❌ |
| `03_Resources/` | Reusable reference materials (target for ingestion) | ✅ |
| `04_Archive/` | Historical records | ❌ |

### Exclusions

The ingestion pipeline skips:

- The `Personal/` folder (sensitive data)
- Files that contain only images (no text)
- Shopping lists or configuration dumps (no knowledge value)

---

## Quick start

### 1. Full sync (recommended)

```text
sync and process
```

Runs the complete workflow in one pass: detect → confirm → process → verify → log → report.

### 2. Detect new/changed files

```text
detect new files
```

Produces a sync report comparing `ObsidianRaw/03_Resources/` and `Wiki/Sources/`:

- New files — queued for ingestion
- Changed files — require re-ingestion
- Deleted files — require handling in Wiki

### 3. Ingest resources

```bash
# Ingest a single file
ingest Academic/NewTopic.md

# Ingest a directory
ingest "Academic/Research Notes/Theory/"

# Ingest all new files
ingest --all-new
```

### 4. Query the Wiki

Ask a question and the system will search the Wiki and answer:

```text
What are the development stages of resilience theory?
What are typical applications of Bayesian networks?
```

### 5. Run health checks

```text
run health check
```

Generates a Wiki health report: orphan pages, concept gaps, contradiction annotations, cognitive quality, and unsynced sources.

---

## Workflows

### First-time setup

```text
1. sync and process    → detect and ingest in one pass (recommended)
2. run health check    → validate the result
```

### Ongoing maintenance

```text
1. regularly sync and process → discover updates and rebuild the Wiki
2. weekly health check        → keep the Wiki healthy
```

### Querying for knowledge

```text
1. Ask a question             → querying-wiki searches the Wiki
2. If the result is high-value → consider backfilling to Wiki/Outputs/
```

---

## Available commands

| Trigger phrases | Skill | Function |
|-----------------|-------|----------|
| `sync and process`, `full sync`, `detect and process` | `syncing-wiki` | Full workflow (recommended) |
| `detect new files`, `sync status`, `check updates` | `detecting-resources-sync` | Scan Resources and report sync status |
| `ingest`, `process file`, `update wiki` | `ingesting-resources` | Ingest source files into the Wiki |
| `health check`, `check status`, `diagnose` | `checking-wiki-health` | Run Wiki health checks |
| Free-form question | `querying-wiki` | Search the Wiki and answer |

---

## Technical details

### File format

All pages use standard YAML frontmatter:

```yaml
---
title: Page title
created: YYYY-MM-DD
updated: YYYY-MM-DD
type: concept | entity | source | output
tags: [tag1, tag2]
---
```

### Source tracking fields

Each Source page includes tracking fields for sync detection:

```yaml
source_path: Academic/Research Notes/Theory/resilience.md
source_hash: 45ea3a15        # first 8 chars of SHA256
source_mtime: 2026-04-08T15:00:05Z
```

These fields allow the system to detect moved, changed or deleted source files.

### Page references

Pages link to each other using `[[wiki-links]]`.

### Naming conventions

| Type | Path | Example |
|------|------|---------|
| Concept | `Wiki/Concepts/{ConceptName}.md` | `Concepts/Resilience.md` |
| Entity  | `Wiki/Entities/{EntityName}.md`  | `Entities/Bayesian_Network.md` |
| Source  | `Wiki/Sources/{SourceId}.md`     | `Sources/Resilience_Paper.md` |
| Output  | `Wiki/Outputs/{YYYY-MM-DD}-{topic}.md` | `Outputs/2026-04-08-resilience-analysis.md` |

---

## Search engine

The system uses the local `qmd` MCP tool for Markdown search and supports:

- Lexical search (`lex`) — exact keyword matching
- Vector search (`vec`) — semantic similarity matching
- Hybrid search — combine both strategies (recommended)
- Precise retrieval — fetch by path or docid

`intent` can be used to disambiguate queries when needed.

---

## Dependencies

- An AI agent runtime (e.g., Claude Code, Codex)
- `qmd` cli and MCP tools, local Markdown search engine
- `openssl`, for computing file hashes

---

## References

- [Karpathy: LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — original inspiration
- [PARA method](https://fortelabs.com/blog/para/) — organization framework for raw sources
- [Obsidian](https://obsidian.md/) — local Markdown knowledge browser
- [qmd](https://github.com/nickthecat/qmd) — local Markdown search engine
