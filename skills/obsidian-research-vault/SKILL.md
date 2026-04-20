---
name: obsidian-research-vault
description: >
  Maintain and organize the Obsidian research vault in the research-os project. Use this skill whenever
  the user wants to: create a literature note, project note, or experiment note; update an existing note;
  link notes together; generate an index or MOC (map of content); check vault health (orphan notes,
  broken links, missing fields); or organize research artifacts in the vault. Also trigger when the user
  mentions Obsidian, vault, research notes, literature notes, project notes, experiment notes, or asks
  to "write up" or "document" research findings.
---

# Obsidian Research Vault Skill

This skill manages the Obsidian vault inside `research-os/obsidian-vault/`. It enforces a consistent note structure, links knowledge across papers/projects/experiments, and keeps the vault navigable as it grows.

## Vault Structure

Each notebook/research topic gets its own subfolder under `literature/`, `maps/`, and `distilled/`. Projects and experiments stay flat since they're cross-cutting.

```
obsidian-vault/
  literature/
    <topic-slug>/         # One subfolder per notebook/research topic
      <paper-slug>.md     # One note per paper
  projects/               # Flat -- cross-cutting across topics
    <project-slug>.md
  experiments/            # Flat -- cross-cutting across topics
    <experiment-slug>.md
  maps/
    <topic-slug>/
      mindmap.json        # Raw JSON from NotebookLM
      mindmap.md          # Mermaid rendering for Obsidian
  distilled/
    <topic-slug>/
      flashcards.md       # Short names -- folder gives context
      flashcards.json
      study-guide.md
      briefing-doc.md
      quiz.md
  _index.md               # Master index / MOC
```

### Topic Slug

Derive from the notebook title. Keep it short, descriptive, lowercase, hyphen-separated:
- "Mind2Web 2: The Agent-as-a-Judge Evaluation Framework" -> `mind2web2-agentic-search`
- "KV Cache Optimization and Scalable LLM Inference" -> `kv-cache-inference`

Templates live in `research-os/templates/`:
- `literature-note.md`
- `project-note.md`
- `experiment-note.md`

## Working with notebooklm-research Skill

When both skills are used together (the most common workflow), the notebooklm-research skill handles artifact generation and this skill handles vault organization. The handoff:

1. **notebooklm-research** runs the **12-probe Q&A set** with `notebooklm ask --json` (thesis, methods, datasets, baselines, numerical results, ablations, failure modes, assumptions, related work, verbatim quotes, open questions, topic relationship) and saves raw JSON to `research-os/scratch/probe{01..12}_*.json`, plus `scratch/source_index.json` from `notebooklm source list --json`
2. **notebooklm-research** generates and downloads artifacts (mind map, flashcards, quiz, study guide, briefing doc, etc.) to the right directories
3. **This skill** creates the literature note as a **grounded dossier** (callouts + footnote source anchors + `![[...]]` transclusion) using the dossier template — see §"Grounded-Dossier Literature Note" below
4. **This skill** creates the mind map Markdown companion from the JSON
5. **This skill** links everything together and updates the index

The 2-batched-Q&A pattern is deprecated; it produces thin notes that lose to the NotebookLM web UI.

### Mind Map JSON -> Markdown Conversion

The mind map JSON from NotebookLM has this structure:
```json
{"name": "Root Topic", "children": [{"name": "Child", "children": [...]}]}
```

Convert it to a **heading-only** Markdown file for the **Mindmap NextGen** community plugin (plugin id: `obsidian-mindmap-nextgen`). Mermaid `mindmap` is deprecated for this vault — it renders as a flat image and feels worse than the NotebookLM web UI.

> [!warning] Heading-only, NOT a ` ```markmap ` fenced block
> Mindmap NextGen has two render paths with different parsers:
> - **Pane view** (Command Palette → *Mindmap NextGen: Open unpinned mindmap* / *Open pinned mindmap*) reads only the document's **heading tree** — `#`, `##`, `###`, bullet lists.
> - **Inline render** (Reading mode) reads fenced ` ```markmap ` blocks embedded inside a prose document.
>
> For files whose sole purpose is to be a mindmap, **use heading-only form**. Users expect to open these in the pane view (full pane width, zoom, fold, pop-out-to-window). Wrapping content in a ` ```markmap ` fence makes the pane view show only the wrapper — a near-empty map. See `memory/feedback_mindmap_format.md` for the full reasoning from a 2026-04-17 debugging session.

**Correct template** — raw headings, no fence:

```markdown
---
type: map
source_paper: "[[literature/<topic-slug>/<paper-slug>]]"
generated: YYYY-MM-DD
updated: YYYY-MM-DD
renderer: markmap
markmap:
  maxWidth: 0            # unlimited node-text width
  colorFreezeLevel: 2    # consistent subtree colors
  initialExpandLevel: 2  # fold beyond depth 2 on first open
tags: [map, <topic-tag>]
---

> [!info] How to view
> Command Palette → **Mindmap NextGen: Open unpinned mindmap**. Right-click the mindmap pane's tab → *Move to new window* for OS-level fullscreen. Pinch / Cmd-scroll to zoom, click nodes to fold.

# Root Topic

## Branch 1
- Leaf A
- Leaf B

## Branch 2
### Sub-branch
- Leaf

---

<!-- source links below the --- and without headings so they don't pollute the mindmap tree -->
**Related:** [[literature/<topic-slug>/<paper-slug>|source paper]] · `maps/<topic-slug>/mindmap.json` (canonical JSON — don't hand-edit in parallel)
```

**Mapping rule:** Depth 1 of the JSON tree → `#` heading (root). Depth 2 → `##`. Depth 3 → `###`. Leaves → `-` bullets. Preserve the JSON's canonical `mindmap.json` unchanged when you update the rendering; the `.md` is a derived view.

**Avoid these pitfalls**:
- **Don't wrap content in ` ```markmap `** — kills the pane view.
- **Don't add `## Source` or `## References` headings after the mindmap content** — they'd show up as extra branches in the tree. Use a `---` rule + plain paragraph instead.
- **Don't add both an outer wrapper H1 like "Mind Map: <title>" and an inner H1 root** — you get two roots. Pick one: either the H1 is the root topic, or delete it and let the H2s be the top level.
- **Only two pane commands exist** in Mindmap NextGen: `Mindmap NextGen: Open unpinned mindmap` (follows active file) and `Mindmap NextGen: Open pinned mindmap` (locks to one file across sessions). Don't hallucinate others. Pins persist in `.obsidian/plugins/obsidian-mindmap-nextgen/data.json`; delete that file to reset stuck pins.
- **Before recommending any plugin command**, verify it exists by `grep`ping the plugin's `main.js` (for Mindmap NextGen 1.16.0 the i18n commands table sits around offset 2447400 with the exact strings). README prose often lags the binary.

## Creating Notes

### Grounded-Dossier Literature Note

When the user reads a paper, build the literature note as a rich dossier — not a thin template skeleton. Target 300-600 lines where every non-trivial claim is source-anchored.

1. Read the template from `templates/literature-note.md` (dossier skeleton with callouts + footnote placeholders).
2. **If a prior version of this note exists, archive it as `<paper-slug>.v1.md` before overwriting** so the user can diff before/after in Obsidian.
3. Replace `{{title}}` and `{{date}}` (use `YYYY-MM-DD`, matches existing `date_read:`).
4. Fill the frontmatter completely — required fields: `type`, `title`, `authors`, `year`, `venue`, `url`, `notebooklm_id`, `date_read`, `rating` (1-5), `status` (`to-read`/`reading`/`read`), `relevance` (1-5), `confidence` (1-5), `topic` (slug), `key_claims` (list of 3-5 claims), `tags`.
5. Walk each probe → section using the 12-probe mapping from the notebooklm-research skill:

| Probe | Section | Callout type |
|-------|---------|--------------|
| 1 thesis | `> [!abstract]` + `> [!tldr]` + §1 Contributions | abstract, tldr |
| 2 methods | §2 Methods + subsections | info, example |
| 3 datasets | §3 Benchmark | example |
| 4 baselines | §4 Evaluated systems table | — |
| 5 numerical results | §5 Results | success, quote |
| 6 ablations | §5 subsection or §6 | — |
| 7 limitations | §6 failure-mode taxonomy + §7 | warning |
| 8 assumptions | §8 | question |
| 9 related work | §9 | info |
| 10 verbatim quotes | Populate `> [!quote]` callouts throughout | quote |
| 11 open questions | §11 | question |
| 12 topic relationship | §10 | info, note |

6. For every non-trivial claim (number, specific technique, named contribution), add a footnote `[^s1]` with the form:
   ```markdown
   Claim in body text with an anchor[^s1].
   ...
   [^s1]: §<section-label> of `<8-char-source-id>`. "verbatim snippet". Source: NotebookLM `<source-id>`.
   ```
7. Promote 1-3 `cited_text` snippets per probe to `> [!quote]` callouts with section labels. Filter out `"Show more"` noise. Strip-quote verbatim; never paraphrase.
8. Embed distilled artifacts inside collapsed callouts via `![[...]]` transclusion — don't leave them orphaned:
   ```markdown
   > [!note]- Briefing doc (transcluded) — click to expand
   > ![[distilled/<topic-slug>/briefing-doc]]
   ```
9. Slugify the title for the filename; save to `obsidian-vault/literature/<topic-slug>/<paper-slug>.md`.
10. Add a §14 "Provenance" block listing probe JSON paths, primary source ID, run date.
11. Verify in Obsidian: callouts expand/collapse, footnotes hover-preview, transclusions resolve (not broken links).

### Callout palette

| Callout | Use for |
|---------|---------|
| `> [!abstract]` | One-paragraph TL;DR at top of note |
| `> [!tldr]` | Thesis as 3 bullets |
| `> [!info]` | Methods / architecture / positioning prose |
| `> [!example]` | Datasets, numbers, worked examples |
| `> [!quote]` | Verbatim quotes from the paper (always with section label in title) |
| `> [!warning]` | Limitations, failure modes |
| `> [!question]` | Assumptions, open questions (both author's and mine) |
| `> [!success]` | Headline results / win conditions |
| `> [!note]-` | Collapsed container for `![[...]]` transclusions or expanded side-analysis |

The trailing `-` after the callout type makes it collapsed by default. `+` makes it open by default.

### Frontmatter schema (extended — supports Dataview queries)

```yaml
type: literature
title: "<paper title>"
authors: <comma-separated>
year: <int>
venue: <string>
url: <arxiv or DOI>
notebooklm_id: <uuid>
date_read: YYYY-MM-DD
rating: 1-5              # how well-executed
status: to-read | reading | read
relevance: 1-5           # to my current work
confidence: 1-5          # how well I understood it
topic: <slug>            # matches parent folder
key_claims:              # 3-5 one-line claims
  - "..."
tags: [literature, <topic-tags>]
```

### Literature Note cross-links (inline, throughout body, not just a Links section)

Link aggressively with topic-subfolder paths. The dossier has a §13 "Cross-links" section but you should also sprinkle `[[maps/<topic>/mindmap|→ mindmap]]` inside §10 (topic-relationship) prose where it aids navigation. Typical link set:

```markdown
- **Mind map**: [[maps/<topic-slug>/mindmap]]
- **Related literature**: [[literature/<topic-slug>/other-paper]]
- **Related project**: [[projects/project-slug]]
- **Related experiment**: [[experiments/experiment-slug]]
```

Do NOT repeat the distilled-file links in the Cross-links section — they already appear as `![[...]]` transclusions in §12.

### Project Note

When the user starts a new research project or direction:

1. Read the template from `templates/project-note.md`
2. Replace `{{title}}` and `{{date}}`
3. Fill in the goal and initial state
4. Save to `obsidian-vault/projects/<slug>.md`
5. Link to relevant literature and experiment notes

### Experiment Note

When the user runs or plans an experiment:

1. Read the template from `templates/experiment-note.md`
2. Replace `{{title}}` and `{{date}}`
3. Fill in hypothesis, setup, and command
4. Link to the parent project note
5. Save to `obsidian-vault/experiments/<slug>.md`
6. After the experiment completes, update metrics, interpretation, and next step

## Updating Notes

When updating an existing note:
- Read the current note first
- Preserve all existing content
- Add new information in the appropriate section
- For project notes, append to the Log section with today's date
- For experiment notes, fill in metrics/interpretation if they were empty
- Update the `status` field in frontmatter when applicable (e.g., `planned` -> `running` -> `completed`)

## Linking Notes

Obsidian uses `[[wiki-links]]`. Follow these linking conventions:

| From | Link to | How |
|------|---------|-----|
| Literature note | Related literature | `[[literature/<topic>/other-paper]]` in Links section |
| Literature note | Mind map | `[[maps/<topic>/mindmap]]` in Links section |
| Literature note | Flashcards / study guide | `[[distilled/<topic>/flashcards]]` in Links section |
| Project note | Literature | `[[literature/<topic>/paper]]` in Links to Literature |
| Project note | Experiments | `[[experiments/slug]]` in Next Experiments or Log |
| Experiment note | Project | `[[projects/slug]]` in frontmatter `project` field |
| Experiment note | Literature | `[[literature/<topic>/paper]]` in Links section |

### Backlinks

When creating a new note that references an existing note, also update the existing note to link back. For example, when creating an experiment note for a project, add a link to the experiment in the project note's "Next Experiments" or "Log" section.

## Index / Map of Content

Maintain `obsidian-vault/_index.md` as a master index. The preferred form is **Dataview-driven** (auto-populates from frontmatter) with a hand-curated fallback section for when the plugin isn't installed.

### Dataview-driven index (preferred)

Install the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) community plugin. The index then needs zero hand-maintenance as long as notes carry the queryable frontmatter fields documented below. The standard `_index.md` ships queries for: Active Projects, Recent Literature, High-Relevance Reading List, To-Read Queue, Recent Experiments, Running/Open Experiments, Maps, Literature by Topic. See `obsidian-vault/_index.md` for the canonical version.

### Queryable frontmatter fields (must be present for Dataview to work)

Literature notes (beyond the standard metadata): `rating` (1-5), `status` (`to-read` | `reading` | `read`), `relevance` (1-5), `confidence` (1-5), `topic` (slug), `key_claims` (list), `date_read` (YYYY-MM-DD).

Project notes: `status` (`active` | `paused` | `completed` | `archived`), `priority` (1-5), `last_active` (YYYY-MM-DD — bump when you add a log entry), `topic` (optional slug).

Experiment notes: `status` (`planned` | `running` | `completed` | `abandoned`), `outcome` (`success` | `fail` | `inconclusive` | null), `created`, `completed` (YYYY-MM-DD, filled when status flips to completed), `project` (wikilink), `topic` (optional slug).

Map notes: `type: map`, `source_paper` (wikilink), `renderer: markmap`, `generated`, `updated`, `tags`.

### Fallback hand-curated listing

`_index.md` also keeps a "Fallback listing" section at the bottom for users without Dataview. Update it whenever a new note is created. Keep most-recent first.

### When updating the index

With Dataview installed, the only thing you need to touch when new notes arrive is:
1. Bump `updated:` in `_index.md` frontmatter to today's date.
2. If adding a one-off curated highlight, append to the Fallback section.

Without Dataview, treat the Fallback section as the actual index and keep it current.

## Vault Health Checks

When asked to check vault health or "tidy up" the vault:

### 1. Orphan Detection
Find notes that have no incoming links from other notes:

```bash
# Find all .md files in the vault
# For each, check if its filename appears as [[link]] in any other file
```

Report orphans and suggest where they should be linked from.

### 2. Broken Link Detection
Find `[[wiki-links]]` that point to non-existent files:

```bash
# Extract all [[links]] from all vault files
# Check each target exists as a file
```

Report broken links with the file they appear in.

### 3. Missing Fields
Check that required template fields are filled in:

- Literature notes: source, thesis, methods must not be empty
- Project notes: goal must not be empty
- Experiment notes: hypothesis must not be empty

Report notes with missing required fields.

### 4. Stale Projects
Find project notes with `status: active` that have no log entries in the last 30 days. Suggest the user update or archive them.

## Filename Conventions

- All filenames and folder names are lowercase, hyphen-separated slugs
- No spaces, no special characters beyond hyphens
- **Topic folders**: `<short-topic-name>/` (e.g., `mind2web2-agentic-search/`, `kv-cache-inference/`)
- **Literature**: `<topic-slug>/<paper-slug>.md` (e.g., `efficient-transformers/linformer.md`)
- **Projects**: `<project-name>.md` (flat, e.g., `efficient-rl-for-llms.md`)
- **Experiments**: `<experiment-name>.md` (flat, e.g., `grpo-lr-sweep-2026-04.md`)
- **Maps**: `<topic-slug>/mindmap.md` + `mindmap.json` (short names, folder gives context)
- **Distilled**: `<topic-slug>/flashcards.md`, `study-guide.md`, `briefing-doc.md`, `quiz.md`

## Bulk Operations

### Import Papers from sources/

When the user has PDFs in `sources/` without corresponding literature notes:

```bash
# List PDFs in sources/ that don't have matching literature notes
```

For each, create a skeleton literature note with the filename derived from the PDF name, source path filled in, and other fields left as placeholders for the user to complete.

### Generate Project Summary

Compile a summary from a project note and all its linked experiments and literature:

1. Read the project note
2. Follow links to all experiments and literature
3. Produce a Markdown summary with: goal, key findings from experiments, relevant literature insights, current blockers, next steps

## Best Practices

- **One note per concept**: Don't combine multiple papers into one literature note. Each paper gets its own note. Cross-reference instead.
- **Link aggressively**: Every note should link to at least one other note. Isolated notes lose context.
- **Fill templates completely**: Empty fields are technical debt. If information isn't available yet, write "TBD" rather than leaving blank.
- **Update, don't duplicate**: If a note exists for a paper/project/experiment, update it rather than creating a new one.
- **Date your log entries**: Project note logs should always have a date header.
- **Use frontmatter tags**: Tags enable Obsidian's search and graph view. Use `[literature]`, `[project]`, `[experiment]`, `[map]`, plus topic-specific tags.
