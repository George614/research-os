# research-os

A personal research operating system that turns NotebookLM notebooks into a searchable, linkable, source-grounded Obsidian knowledge vault.

## What it is

An opinionated pipeline for academic/industry literature review that glues together:

- **NotebookLM** (via the `notebooklm-py` CLI) for paper ingestion, Q&A, and artifact generation (mind maps, briefing docs, study guides, quizzes, flashcards, slide decks)
- **Obsidian** as the primary reading and linking surface, with:
  - [Dataview](https://github.com/blacksmithgu/obsidian-dataview) for auto-generated indices and reading queues
  - [Mindmap NextGen](https://github.com/james-tindal/obsidian-mindmap-nextgen) for interactive, zoomable concept maps
- **Claude Code skills** (`notebooklm-research`, `obsidian-research-vault`) that codify the end-to-end workflow so new papers are ingested with a single prompt

## Directory layout

```
research-os/
├── obsidian-vault/              ← open this in Obsidian
│   ├── _index.md                    Dataview-driven MOC
│   ├── literature/<topic>/          one sub-folder per topic; one .md per paper
│   ├── maps/<topic>/                mindmap.md (heading-only) + mindmap.json
│   ├── distilled/<topic>/           briefing-doc / study-guide / quiz / flashcards
│   ├── projects/                    flat; cross-cutting research threads
│   └── experiments/                 flat; cross-cutting experiment logs
├── templates/                   ← used by the obsidian-research-vault skill
│   ├── literature-note.md           dossier template (callouts, footnotes, transclusion)
│   ├── project-note.md              with Dataview-queryable frontmatter
│   └── experiment-note.md
├── sources/                     ← raw PDFs downloaded for NotebookLM ingestion
├── exports/                     ← slide decks, infographics, CSVs (consumed outside Obsidian)
├── scratch/                     ← raw probe JSON from NotebookLM Q&A runs (provenance)
└── scripts/                     ← future helpers
```

## Core idea: grounded dossiers, not skeletons

Each paper becomes a **300–600 line literature-note dossier** in `obsidian-vault/literature/<topic>/`. The dossier uses:

- Obsidian callouts (`> [!abstract]`, `> [!quote]`, `> [!warning]`, `> [!question]`, etc.) for visual hierarchy
- Footnote-style source anchors (`[^s1]` → paper section + NotebookLM source UUID) for every non-trivial claim
- `![[...]]` transclusion of distilled artifacts (flashcards, quizzes, study guides) inside collapsible callouts
- Rich frontmatter (`rating`, `status`, `relevance`, `confidence`, `key_claims`, `topic`) so Dataview queries can surface "unread high-relevance papers in topic X"

The Mind2Web 2 paper under `obsidian-vault/literature/mind2web2-agentic-search/` is the worked example — compare `*.md` (current dossier) against `*.v1.md` (the thin prior template) for the before/after.

## Getting started

### 1. Install Obsidian and open the vault

1. Install [Obsidian](https://obsidian.md/).
2. Open the `obsidian-vault/` directory as an Obsidian vault.
3. Settings → Community plugins → Turn on → install **Dataview** and **Mindmap NextGen** (both already listed in `.obsidian/community-plugins.json`). Trust community plugins on first prompt.

### 2. Install the NotebookLM CLI (optional — only if you want to regenerate)

The vault content was generated via the `notebooklm-py` community CLI. Install from its project page, then run `notebooklm login` to authenticate with a Google account that has NotebookLM access.

### 3. Hook up the Claude Code skills (optional)

If you use [Claude Code](https://docs.claude.com/en/docs/claude-code/overview), the pipeline is automated via two skills under `~/.claude/skills/`:

- `notebooklm-research` — NotebookLM ingestion, Q&A (12-probe `ask --json` pattern), and artifact generation
- `obsidian-research-vault` — vault organization (dossier assembly, Dataview-queryable frontmatter, Markmap-compatible heading-only mindmaps)

Tell Claude Code: *"Ingest `<arxiv-url>` into the `<topic-slug>` notebook"* and it will ingest the paper, run the 12-probe Q&A, generate artifacts, and drop a full dossier into `obsidian-vault/literature/<topic>/`.

## Conventions

- **Topic slug** — lowercase-hyphenated, short, descriptive. e.g., `mind2web2-agentic-search`, `kv-cache-inference`.
- **Literature, maps, distilled** — nested under the topic slug.
- **Projects, experiments** — flat, since they tend to span topics.
- **Exports** — flat, prefixed with the topic slug: `exports/mind2web2-slides.pptx`.

See the skill files (`~/.claude/skills/notebooklm-research/SKILL.md`, `~/.claude/skills/obsidian-research-vault/SKILL.md`) for the full rules.

## License

Personal research workspace. No license granted; do not redistribute paper content.
