---
name: notebooklm-research
description: >
  Automate research paper reading, analysis, and study material generation using Google NotebookLM
  via the notebooklm-py CLI, integrated with the research-os directory structure. Use this skill
  whenever the user wants to: read or analyze research papers, create mind maps from papers or topics,
  generate flashcards for studying, create quizzes for comprehension testing, produce audio overviews
  (podcasts) of papers, build study guides or briefing documents, discover related research, or ask
  questions about loaded papers. Also trigger when the user mentions NotebookLM, arXiv papers, paper
  summaries, literature reviews, or research workflows. Even if the user just says "summarize this
  paper" or "help me study this", this skill applies.
---

# NotebookLM Research Skill

This skill uses the `notebooklm` CLI (from `notebooklm-py`) to automate research paper workflows. The CLI wraps Google NotebookLM's capabilities, giving programmatic access to paper ingestion, AI-powered Q&A, and study material generation -- including features the web UI doesn't expose (structured JSON exports, batch downloads, editable PPTX, etc.).

**Important context**: This uses undocumented Google APIs via browser-session auth. It's ideal for personal research and prototyping, not production systems. Operations like audio/video generation take 5-45 minutes and are rate-limited.

## Research-OS Directory Structure

All operations use the `research-os` project directory. Determine the project root by looking for the directory containing `sources/`, `exports/`, `obsidian-vault/`, `templates/`, and `scripts/`. Typically `/Users/zhizhuo/Projects/research-os`.

```
research-os/
  sources/          # Downloaded PDFs, raw paper files, input materials
  exports/          # Generated artifacts the user requests to export (slides, audio, reports, CSVs)
  obsidian-vault/   # Mind maps, literature notes, distilled knowledge, interlinked notes
    literature/
      <topic-slug>/   # One subfolder per notebook/research topic
        <paper>.md    # One note per paper within that topic
    projects/         # Project notes (flat -- cross-cutting)
    experiments/      # Experiment notes (flat -- cross-cutting)
    maps/
      <topic-slug>/   # One subfolder per topic
        mindmap.json
        mindmap.md
    distilled/
      <topic-slug>/   # One subfolder per topic
        flashcards.md
        study-guide.md
        briefing-doc.md
        quiz.md
    _index.md         # Master index / MOC
  templates/          # Obsidian note templates (literature, project, experiment)
  scripts/            # Automation scripts
```

### Topic Slug Convention

Each notebook maps to a **topic slug** -- a short, descriptive, hyphenated name derived from the notebook title. Examples:

| Notebook title | Topic slug |
|---|---|
| Mind2Web 2: The Agent-as-a-Judge Evaluation Framework | `mind2web2-agentic-search` |
| KV Cache Optimization and Scalable LLM Inference | `kv-cache-inference` |
| Literature Review: Efficient Transformers | `efficient-transformers` |

The topic slug is used as the subfolder name under `literature/`, `maps/`, and `distilled/`. Files within a topic folder use **short names** since the folder provides context (e.g., `flashcards.md` not `mind2web2-flashcards.md`). Literature notes keep their paper-specific slug (e.g., `mind2web2-agent-as-a-judge.md`) since a topic can contain multiple papers.

### Routing Rules

| Content type | Destination |
|---|---|
| Downloaded PDFs / raw papers | `sources/` |
| Mind maps (JSON + Markdown rendering) | `obsidian-vault/maps/<topic-slug>/` |
| Literature notes (per-paper) | `obsidian-vault/literature/<topic-slug>/` |
| Study guides, briefing docs, flashcard decks, quizzes | `obsidian-vault/distilled/<topic-slug>/` |
| Slide decks (.pptx, .pdf) | `exports/` |
| Audio overviews (.mp3) | `exports/` |
| Video overviews (.mp4) | `exports/` |
| Infographics (.png) | `exports/` |
| Data tables (.csv) | `exports/` |
| Any artifact the user explicitly asks to "export" | `exports/` |

Exports stay flat (prefixed with topic slug) since they're consumed outside Obsidian.

### Auto-create Subdirectories

Before writing any file, derive the topic slug from the notebook title, then ensure directories exist:

```bash
TOPIC="<topic-slug>"
mkdir -p sources/ exports/ \
  obsidian-vault/literature/$TOPIC \
  obsidian-vault/maps/$TOPIC \
  obsidian-vault/distilled/$TOPIC \
  obsidian-vault/{projects,experiments}
```

## Prerequisites

The user must have `notebooklm-py` installed and authenticated:

```bash
# Check if installed
notebooklm --version

# If not installed
pip install "notebooklm-py[browser]"
playwright install chromium

# Authenticate (opens browser for Google login)
notebooklm login

# Verify auth works
notebooklm auth check --test
```

If auth fails, suggest `notebooklm login` first. Credentials persist at `~/.notebooklm/storage_state.json`.

## Critical CLI Behavior Notes

These were discovered through real usage and override any assumptions:

### Notebook IDs Must Be Full UUIDs
The `notebooklm list` table display truncates IDs. Truncated IDs fail with RPC errors. **Always** use `notebooklm list --json` to get full UUIDs before running `notebooklm use` or `-n`.

```bash
# BAD: truncated ID from table display
notebooklm use a3e3507a-ccae-4d3b-a150-2…   # FAILS

# GOOD: full UUID from --json output
notebooklm list --json   # get full IDs
notebooklm use a3e3507a-ccae-4d3b-a150-20f97cf543d5
```

### `--wait` Has a Hard 300s Internal Timeout
The CLI's `--wait` flag times out after 300 seconds regardless of shell timeout. This is fine for fast artifacts (flashcards, quiz, mind map, report, infographic, data table) but **will fail for slow artifacts** (slide decks, audio, video).

For slow artifacts, use **non-blocking generation + polling**:

```bash
# Start generation (returns a task ID immediately)
notebooklm generate slide-deck -n <notebook_id>
# Output: Started: <task_id>

# Poll until complete (no --wait flag exists on poll)
notebooklm artifact poll <task_id> -n <notebook_id>

# Once status shows completed, download
notebooklm download slide-deck -n <notebook_id> exports/<slug>-slides.pptx
```

### Never Retry Generation Blindly
Each `notebooklm generate` call creates a **new server-side task**. Retrying because of a timeout creates duplicate tasks (visible in the NotebookLM UI). Instead:
1. Save the task ID from the original `generate` output
2. Poll that task ID until it completes
3. Only retry if the task actually failed (not just timed out locally)

### Multiple Reports Need `--name` to Download
When generating multiple report formats (e.g., study-guide then briefing-doc), `notebooklm download report` defaults to the **latest** report. Use `--name` with a partial match to download a specific one. There is no `--index` flag.

```bash
# Generate two report types
notebooklm generate report --format study-guide --wait
notebooklm generate report --format briefing-doc --wait

# Download each by name (partial match works)
notebooklm download report --name "Study Guide" obsidian-vault/distilled/<slug>-study-guide.md
notebooklm download report --name "Strategic" obsidian-vault/distilled/<slug>-briefing-doc.md

# Without --name, you get the most recently generated report
notebooklm download report obsidian-vault/distilled/<slug>-latest-report.md
```

### Artifact Speed Tiers
Plan generation order accordingly -- kick off slow artifacts first (non-blocking), then fast ones (with `--wait`), then poll/download slow ones last.

| Tier | Artifacts | Typical Time | Strategy |
|------|-----------|-------------|----------|
| Fast (<60s) | mind-map, flashcards, quiz, report, data-table | 5-30s | Use `--wait`, generate in parallel |
| Medium (1-5min) | infographic, slide-deck | 1-10min | Generate non-blocking, poll, download later |
| Slow (5-45min) | audio, video | 5-45min | Generate non-blocking, poll periodically, download last |

## Core Workflows

### 1. Paper Ingestion Pipeline

The foundation of every research workflow: create a notebook and load papers into it.

```bash
# Create a notebook for the research topic
notebooklm create "Attention Is All You Need - Analysis"

# The create command outputs a notebook ID and sets it as current context.
# All subsequent commands use this notebook automatically.

# Add papers as sources (auto-detects type)
notebooklm source add "https://arxiv.org/abs/1706.03762"         # arXiv URL
notebooklm source add "./sources/transformer_paper.pdf"            # Local PDF from sources/
notebooklm source add "https://youtube.com/watch?v=..."            # YouTube lecture
notebooklm source add "Key insight: self-attention replaces recurrence" --title "My Notes"  # Inline notes

# Wait for source processing (important before generating content)
notebooklm source wait

# Verify sources loaded
notebooklm source list
```

**Source types supported**: URLs, PDFs, Markdown, Word docs, text files, YouTube videos, Google Drive docs, pasted text.

**Why waiting matters**: NotebookLM indexes source content asynchronously. Generating artifacts before indexing completes produces shallow or incomplete results. Always wait for sources to finish processing.

**Saving source PDFs**: When the user provides a URL to a paper, also download the PDF to `sources/` for local archival:

```bash
# Example: download arXiv PDF
curl -L "https://arxiv.org/pdf/1706.03762" -o sources/attention-is-all-you-need.pdf
```

### 2. Research Discovery

Find and import related papers automatically:

```bash
# Web search -- finds relevant sources and adds them
notebooklm source add-research "transformer attention mechanisms" --mode deep --import-all

# For non-blocking deep research (takes longer but more thorough)
notebooklm source add-research "self-attention alternatives 2024" --mode deep --no-wait
notebooklm research status    # Check progress
notebooklm research wait --import-all  # Wait and import results

# Google Drive search (if papers are stored there)
notebooklm source add-research "project docs" --from drive
```

### 3. Interactive Paper Q&A

Ask questions grounded in the loaded papers -- answers include citations:

```bash
# Ask about the paper
notebooklm ask "What is the key innovation of the transformer architecture?"

# Scope to specific sources
notebooklm ask -s <source_id> "What are the limitations discussed in section 5?"

# Get structured output with source references
notebooklm ask "Compare the computational complexity of self-attention vs recurrence" --json

# Save a particularly good answer as a notebook note
notebooklm ask "Summarize the main contributions" --save-as-note --note-title "Key Contributions"
```

### 4. Mind Map Generation (-> obsidian-vault/maps/<topic-slug>/)

Create hierarchical concept maps from papers:

```bash
# Generate mind map from all sources
notebooklm generate mind-map

# From specific sources only
notebooklm generate mind-map -s <source_id>

# Download JSON to topic subfolder
notebooklm download mind-map obsidian-vault/maps/<topic-slug>/mindmap.json
```

The JSON output contains a hierarchical tree of concepts. After downloading, convert it to a Markdown rendering for Obsidian:

Write a companion `mindmap.md` in the same topic folder that renders as a Mermaid diagram, with YAML frontmatter:

```markdown
---
type: map
source_paper: "[[literature/<topic-slug>/<paper-slug>]]"
generated: YYYY-MM-DD
tags: [map]
---
# Mind Map: Paper Title

(Mermaid mindmap diagram here)
```

### 5. Flashcard Generation (-> obsidian-vault/distilled/<topic-slug>/)

Create study cards with configurable difficulty:

```bash
# Standard flashcards
notebooklm generate flashcards --wait

# Customized generation
notebooklm generate flashcards "focus on key equations and theorems" --quantity more --difficulty hard --wait

# Scope to specific sources
notebooklm generate flashcards -s <source_id> "vocabulary and definitions only" --wait

# Download to topic subfolder
notebooklm download flashcards obsidian-vault/distilled/<topic-slug>/flashcards.json
notebooklm download flashcards obsidian-vault/distilled/<topic-slug>/flashcards.md
```

**Difficulty levels**: easy, medium, hard
**Quantity**: fewer, standard, more

### 6. Quiz Generation (-> obsidian-vault/distilled/<topic-slug>/)

Test comprehension with auto-generated quizzes:

```bash
# Generate quiz
notebooklm generate quiz "test understanding of multi-head attention" --difficulty hard --quantity more --wait

# Download to topic subfolder
notebooklm download quiz obsidian-vault/distilled/<topic-slug>/quiz.json
notebooklm download quiz --format markdown obsidian-vault/distilled/<topic-slug>/quiz.md
```

### 7. Audio Overview (-> exports/)

Generate podcast-style audio discussions of papers:

```bash
# Deep dive podcast (default, most thorough)
notebooklm generate audio "focus on the methodology and results" --format deep-dive --wait

# Quick briefing
notebooklm generate audio --format brief --length short --wait

# Debate format (two perspectives)
notebooklm generate audio "debate the practical implications" --format debate --wait

# Critical analysis
notebooklm generate audio --format critique --wait

# Download to exports
notebooklm download audio exports/<slug>-podcast.mp3
```

**Formats**: deep-dive, brief, critique, debate
**Lengths**: short, default, long
**Note**: Audio generation takes 5-20 minutes. Use `--wait` to block, or omit it and check with `notebooklm artifact poll`.

### 8. Study Guide & Reports (-> obsidian-vault/distilled/)

Generate comprehensive study materials. **Important**: when generating multiple report types, each becomes a separate artifact. You must use `--name` to download specific reports (see "Multiple Reports Need `--name`" above).

```bash
# Study guide (structured for learning)
notebooklm generate report --format study-guide --wait

# Briefing document (executive summary style)
notebooklm generate report --format briefing-doc --wait

# Blog post (accessible explanation)
notebooklm generate report --format blog-post --wait

# Custom report with specific instructions
notebooklm generate report "Create a literature review comparing this paper to recent works on efficient attention" --format custom --wait

# Download EACH report by name (partial match) to topic subfolder
notebooklm download report --name "Study Guide" obsidian-vault/distilled/<topic-slug>/study-guide.md
notebooklm download report --name "Briefing" obsidian-vault/distilled/<topic-slug>/briefing-doc.md
# Without --name, defaults to latest generated report
```

### 9. Additional Artifacts (-> exports/)

```bash
# Slide deck (for presentations about the paper)
notebooklm generate slide-deck --wait
notebooklm download slide-deck exports/<slug>-slides.pptx
notebooklm download slide-deck exports/<slug>-slides.pdf

# Infographic
notebooklm generate infographic --orientation landscape --wait
notebooklm download infographic exports/<slug>-infographic.png

# Data table (extract structured data)
notebooklm generate data-table "compare model sizes, training costs, and BLEU scores" --wait
notebooklm download data-table exports/<slug>-comparison.csv

# Video overview
notebooklm generate video --style whiteboard --wait
notebooklm download video exports/<slug>-explainer.mp4
```

## Complete Research Paper Workflow

End-to-end workflow for deeply studying a paper, integrated with research-os. Optimized for speed: kicks off slow artifacts first, runs fast ones in parallel, downloads slow ones last.

```bash
# Step 0: Setup
mkdir -p sources/ exports/ obsidian-vault/{literature,projects,experiments,maps,distilled}
notebooklm list --json   # get full notebook UUIDs

# Step 1: Download and ingest paper
curl -L "<pdf_url>" -o sources/<slug>.pdf
notebooklm create "Paper: [Title]"
notebooklm source add "sources/<slug>.pdf"
notebooklm source wait

# Step 2: Kick off SLOW artifacts first (non-blocking, no --wait)
notebooklm generate slide-deck -n <notebook_id>
# Save the task ID from output for later polling

# Step 3: Q&A probe set for grounded dossier (12 probes with --json for source anchors)
# Run independent probes in parallel (background). Each returns {answer, references[], conversation_id, ...}.
# IMPORTANT: redirect stderr separately; CLI may print "No answer extracted" warnings that break JSON parsing.
mkdir -p scratch
notebooklm ask --json -n <id> "What is the central thesis and 3-5 key contributions? Be specific." > scratch/probe01_thesis.json 2> scratch/probe01.stderr &
notebooklm ask --json -n <id> "Describe the methods/architecture/evaluation pipeline in detail." > scratch/probe02_methods.json 2> scratch/probe02.stderr &
notebooklm ask --json -n <id> "What benchmarks/datasets are used, and what are their key statistics?" > scratch/probe03_datasets.json 2> scratch/probe03.stderr &
notebooklm ask --json -n <id> "Which systems/baselines are evaluated? List each briefly." > scratch/probe04_baselines.json 2> scratch/probe04.stderr &
notebooklm ask --json -n <id> "Report all key numerical results as a markdown table." > scratch/probe05_numbers.json 2> scratch/probe05.stderr &
notebooklm ask --json -n <id> "What ablations and robustness analyses are run? Quote percentages." > scratch/probe06_ablations.json 2> scratch/probe06.stderr &
notebooklm ask --json -n <id> "What failure modes and limitations does the paper document?" > scratch/probe07_limitations.json 2> scratch/probe07.stderr &
notebooklm ask --json -n <id> "What assumptions -- especially implicit ones -- does the methodology rest on?" > scratch/probe08_assumptions.json 2> scratch/probe08.stderr &
notebooklm ask --json -n <id> "How does the paper position itself against prior work? Which lines of work does it build on?" > scratch/probe09_related.json 2> scratch/probe09.stderr &
notebooklm ask --json -n <id> "Give 5-7 verbatim quotes that best capture thesis, methodology, and striking results. Include section/subsection for each." > scratch/probe10_quotes.json 2> scratch/probe10.stderr &
notebooklm ask --json -n <id> "What open questions and future work does the paper name, explicitly and implicitly?" > scratch/probe11_openq.json 2> scratch/probe11.stderr &
notebooklm ask --json -n <id> "How does this paper relate to the other sources in this notebook? Name specific neighbors and relationships." > scratch/probe12_topic.json 2> scratch/probe12.stderr &
wait

# Record the source index so citation UUIDs can be mapped to papers
notebooklm source list --json > scratch/source_index.json

# Step 4: Generate FAST artifacts in parallel (all use --wait, safe under 300s)
notebooklm generate mind-map -n <id>
notebooklm generate flashcards "key concepts and methods" --quantity more --difficulty hard -n <id> --wait
notebooklm generate quiz "test comprehension" --difficulty hard -n <id> --wait
notebooklm generate report --format study-guide -n <id> --wait
notebooklm generate report --format briefing-doc -n <id> --wait
notebooklm generate infographic --orientation landscape -n <id> --wait
notebooklm generate data-table "compare methods, results, metrics" -n <id> --wait

# Step 5: Download fast artifacts
notebooklm download mind-map -n <id> obsidian-vault/maps/<slug>-mindmap.json
notebooklm download flashcards -n <id> obsidian-vault/distilled/<slug>-flashcards.md
notebooklm download flashcards -n <id> obsidian-vault/distilled/<slug>-flashcards.json
notebooklm download quiz --format markdown -n <id> obsidian-vault/distilled/<slug>-quiz.md
notebooklm download report --name "Study Guide" -n <id> obsidian-vault/distilled/<slug>-study-guide.md
notebooklm download report --name "Strategic" -n <id> obsidian-vault/distilled/<slug>-briefing-doc.md
notebooklm download infographic -n <id> exports/<slug>-infographic.png
notebooklm download data-table -n <id> exports/<slug>-comparison.csv

# Step 6: Convert mind map JSON to Mermaid .md for Obsidian
# Read the JSON, build a Mermaid mindmap diagram, save as obsidian-vault/maps/<slug>-mindmap.md

# Step 7: Build grounded dossier literature note (see "Auto-generating Literature Notes" below)
# Read templates/literature-note.md (the dossier template with callouts + footnotes).
# Read scratch/probe{01..12}_*.json + scratch/source_index.json.
# Fill every section; embed verbatim quotes as `> [!quote]` callouts with section anchors.
# Ground non-trivial claims with footnotes `[^s1]` naming the source ID (8-char prefix).
# Transclude distilled files via `![[distilled/<topic>/briefing-doc]]` inside collapsed `> [!note]-` callouts.
# Save to obsidian-vault/literature/<topic-slug>/<paper-slug>.md
# If a prior version exists, archive it as <paper-slug>.v1.md before overwriting.

# Step 8: Update vault index (_index.md)

# Step 9: Poll and download slow artifacts
notebooklm artifact poll <task_id> -n <id>   # repeat until completed
notebooklm download slide-deck -n <id> exports/<slug>-slides.pptx
```

## Auto-generating Literature Notes

After ingesting a paper, build a grounded dossier (not a thin template). The output target is a 300-600 line literature note where every non-trivial claim is source-anchored. Two-probe Q&A is **deprecated** — it produces thin notes that read worse than the NotebookLM web UI.

### The 12-probe Q&A set

Run these in parallel using `notebooklm ask --json` (see Step 3 of the workflow above). Each `--json` response carries `{answer, references[], conversation_id, turn_number, is_follow_up}`. The `references[]` list is the gold — each entry has `source_id`, `citation_number`, `cited_text`, `start_char`, `end_char`, `chunk_id`.

| # | Probe | Purpose |
|---|-------|---------|
| 1 | Central thesis + 3-5 key contributions | `> [!abstract]` + `> [!tldr]` + Contributions section |
| 2 | Methods / architecture / evaluation pipeline | §2 Methods with `> [!info]` + subsection callouts |
| 3 | Benchmarks / datasets + statistics | §3 with `> [!example]` |
| 4 | Evaluated systems / baselines | §4 comparison table |
| 5 | Numerical results (table-ready) | §5 with `> [!success]` results table |
| 6 | Ablations + robustness | §5 subsection or §6 |
| 7 | Failure modes + limitations | §6 taxonomy + §7 `> [!warning]` |
| 8 | Assumptions (implicit and explicit) | §8 `> [!question]` |
| 9 | Positioning vs. prior work | §9 `> [!info]` |
| 10 | 5-7 verbatim quotes with section anchors | Populate `> [!quote]` callouts |
| 11 | Open questions + future work | §11 `> [!question]` |
| 12 | Relationship to notebook's other sources | §10 cross-paper interlock |

**Redirect stderr separately** — the CLI occasionally writes `WARNING [notebooklm._chat] No answer extracted` to stderr, which breaks JSON parsing if mixed into stdout. If a probe returns `{"answer": ""}`, retry that probe alone with a simpler phrasing.

Also capture `notebooklm source list --json > scratch/source_index.json` so you can map 8-char source UUIDs to paper titles when composing footnotes.

### Composing the dossier

1. Read `research-os/templates/literature-note.md` (the dossier template — callouts, footnotes, transclusion scaffolding).
2. If a prior note exists, archive it as `<paper-slug>.v1.md` before overwriting.
3. Fill the frontmatter: `rating`, `status`, `relevance`, `confidence`, `topic` (slug), `key_claims` (list), plus the usual metadata fields.
4. Walk each probe → section:
   - Paste the probe answer as the section body.
   - Extract 1-3 verbatim snippets from `cited_text` (filter out "Show more" noise) and promote them to `> [!quote] §<section-label>` callouts. Strip-quote the verbatim snippet; don't paraphrase.
   - For each non-trivial numeric claim or named technique, add a footnote `[^s1]` naming the paper section + the 8-char source-id prefix.
5. Wrap distilled files in collapsed callouts:
   ```markdown
   > [!note]- Briefing doc (transcluded) — click to expand
   > ![[distilled/<topic-slug>/briefing-doc]]
   ```
   Do the same for study-guide, quiz, flashcards.
6. Add a §14 "Provenance" block listing the probe JSON paths, the primary source ID, and the run date.
7. Save to `obsidian-vault/literature/<topic-slug>/<paper-slug>.md`.
8. Verify the file renders in Obsidian: callouts expand/collapse, footnotes hover-preview, transclusions resolve to their targets.

## Working with Multiple Papers

For literature reviews or comparative studies:

```bash
# Create a themed notebook
notebooklm create "Literature Review: Efficient Transformers"

# Download and add multiple papers
curl -L "https://arxiv.org/pdf/2009.06732" -o sources/linformer.pdf
curl -L "https://arxiv.org/pdf/2006.04768" -o sources/performer.pdf
curl -L "https://arxiv.org/pdf/2004.05150" -o sources/longformer.pdf

notebooklm source add "sources/linformer.pdf"
notebooklm source add "sources/performer.pdf"
notebooklm source add "sources/longformer.pdf"
notebooklm source wait

# Cross-paper analysis
notebooklm ask "Compare the approaches to reducing attention complexity across these papers"
notebooklm generate data-table "compare methods, complexity, performance tradeoffs"
notebooklm download data-table exports/efficient-transformers-comparison.csv

notebooklm generate report "Write a comparative literature review" --format custom --wait
notebooklm download report obsidian-vault/distilled/efficient-transformers-review.md

# Generate individual literature notes for each paper
# Then create a project note linking them all together
```

## Notebook Management

```bash
# List all notebooks (table view -- IDs are truncated!)
notebooklm list

# ALWAYS use --json to get full UUIDs before switching
notebooklm list --json

# Switch to a notebook (MUST use full UUID, not truncated)
notebooklm use <full-notebook-uuid>

# Check current context
notebooklm status

# Rename
notebooklm rename <notebook_id> "Better Title"

# Delete (destructive -- confirm with user first)
notebooklm delete <notebook_id>
```

## Best Practices

### Rate Limiting
Audio, video, quiz, flashcard, infographic, and slide deck generation are rate-limited by Google. If you hit a rate limit, wait 5-10 minutes before retrying. Use `--retry 3` for automatic exponential backoff.

### Source Processing
Always run `notebooklm source wait` after adding sources and before generating any artifacts. Generating content from unprocessed sources produces incomplete results.

### Output Routing
Follow the routing rules table above. Never dump all output into a single flat directory. Mind maps and knowledge artifacts belong in `obsidian-vault/`, presentation artifacts belong in `exports/`.

### JSON for Downstream Processing
When flashcards or quizzes will be imported into Anki, Quizlet, or other tools, always download as JSON -- it preserves structure. Markdown is better for human reading and Obsidian integration.

### Parallel-Safe Commands
When using this skill from subagents, always pass `-n <notebook_id>` explicitly rather than relying on `notebooklm use` context, which can cause conflicts.

### Mind Map Post-Processing
The mind map JSON can be converted to various visual formats. Always create a companion Markdown file in `obsidian-vault/maps/` that renders the mind map as a Mermaid diagram or indented bullet list so it's browsable in Obsidian.

### Literature Note Hygiene
Every paper ingested should produce a literature note in `obsidian-vault/literature/`. Use the template. Link to related maps, distilled content, projects, and experiments.

## Troubleshooting

| Issue | Fix |
|-------|-----|
| "Not authenticated" | Run `notebooklm login` |
| "RPC GET_NOTEBOOK failed" on `use` | ID is truncated. Run `notebooklm list --json` to get full UUID |
| `--wait` times out (300s) | Normal for slide decks/audio/video. Use non-blocking generation + `artifact poll` instead |
| `download report` gets wrong report | Multiple reports exist. Use `--name "<partial_title>"` to select specific one |
| `download report --index N` fails | `--index` doesn't exist. Use `--name` instead |
| `artifact poll --wait` fails | `--wait` flag doesn't exist on poll. Must poll manually in a loop |
| Duplicate artifacts in UI | Caused by retrying `generate`. Poll the original task ID instead of re-generating |
| Source stuck processing | Run `notebooklm source refresh <id>` then `source wait` |
| Generation fails with rate limit | Wait 5-10 min, retry with `--retry 3` |
| Empty/shallow results | Ensure sources finished processing (`source wait`) |
| Auth expired | Run `notebooklm login` again |
| Playwright issues on Linux | See `notebooklm` troubleshooting docs |
| Wrong output directory | Check routing rules table; mind maps go to vault, slides go to exports |
