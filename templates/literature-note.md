---
type: literature
title: "{{title}}"
authors:
year:
venue:
url:
notebooklm_id:
date_read: {{date}}
rating:        # 1-5, how well-executed is the paper
status: read   # to-read | reading | read
relevance:     # 1-5, relevance to my current work
confidence:    # 1-5, how well I understood it
topic:         # topic slug, matches parent folder
key_claims:
  -
  -
tags: [literature]
---

# {{title}}

> [!abstract]
> One-paragraph TL;DR — what the paper argues, what it introduces, and the most striking result. Write last, after the rest is filled in.

> [!tldr] Thesis in three bullets
> -
> -
> -

---

## Source metadata

- **Paper**:
- **Authors**:
- **Venue / year**:
- **arXiv / DOI**:
- **Primary NotebookLM source**: `<8-char-source-id>`
- **Local PDF**: `sources/<paper-slug>.pdf`
- **Leaderboard / code**: _(if any)_[^s1]

[^s1]: <section> of `<source-id>`. "verbatim snippet". Source: NotebookLM `<source-id>`.

---

## 1. Problem and thesis

> [!info] Why this work is needed
> What gap does the paper fill? What's the status quo it critiques?[^s2]

[^s2]: <section> of `<source-id>`. "verbatim snippet".

> [!quote] <§ label> — thesis quote
> "Verbatim sentence from the paper that states the central claim."

### Contributions (what is actually new)

1.
2.
3.

---

## 2. Methods

> [!info] Architecture in one paragraph
> Prose summary of how the system works end-to-end.[^s3]

[^s3]: Methods section of `<source-id>`. "verbatim".

### 2.1 <component or sub-method>

### 2.2 <component or sub-method>

> [!example] Worked example from the paper
> Concrete walkthrough, ideally with real numbers or identifiers.

---

## 3. Benchmark / dataset

> [!example] Dataset at a glance
> - Task count, splits, domain coverage
> - Key statistics (size, horizon, difficulty)
> - Anything unusual about construction

---

## 4. Evaluated systems / baselines

| System | What it is |
|--------|-----------|
|        |            |

---

## 5. Results

> [!success] Headline numbers

| Category | Metric | Value |
|----------|--------|-------|
|          |        |       |

> [!quote] <§ label> — the result quote
> "Verbatim result with specific numbers."

---

## 6. Failure modes / error taxonomy

> [!warning]
> Bullet list or small table of observed failures the paper catalogues.

---

## 7. Limitations

> [!warning] Framework-level constraints (authors' own)
> -
> -

---

## 8. Assumptions

> [!question] Assumptions worth challenging
> -
> -

---

## 9. Positioning vs. related work

> [!info]
> How does this paper relate to prior benchmarks / methods / frameworks?

---

## 10. Relationship to this notebook's broader theme

> [!info] How this interlocks with the topic
> If the paper sits inside a themed notebook, name the 2-3 nearest neighbors and the specific relationship.

---

## 11. Open questions

> [!question] Explicitly stated by the authors
> -

> [!question] Implicit (from failure modes)
> -

> [!question] My open questions
> - [ ]

---

## 12. Embedded distilled materials

> [!note]- Briefing doc (transcluded) — click to expand
> ![[distilled/<topic-slug>/briefing-doc]]

> [!note]- Study guide (transcluded) — click to expand
> ![[distilled/<topic-slug>/study-guide]]

> [!note]- Quiz (transcluded) — click to expand
> ![[distilled/<topic-slug>/quiz]]

> [!note]- Flashcards (transcluded) — click to expand
> ![[distilled/<topic-slug>/flashcards]]

---

## 13. Cross-links

- **Mind map**: [[maps/<topic-slug>/mindmap]]
- **Related literature**: [[literature/<topic-slug>/<other-paper>]]
- **Related project**: [[projects/<project-slug>]]
- **Related experiment**: [[experiments/<experiment-slug>]]

---

## 14. Provenance

- Built from N-probe NotebookLM Q&A run on YYYY-MM-DD. Raw probe JSON: `research-os/scratch/probe{01..N}_*.json`.
- Primary source: NotebookLM source `<id>`.
- Quotes verbatim from the paper (§-locations indicated).
