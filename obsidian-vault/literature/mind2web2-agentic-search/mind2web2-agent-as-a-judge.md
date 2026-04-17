---
type: literature
title: "Mind2Web 2: Evaluating Agentic Search with Agent-as-a-Judge"
authors: Xiang Deng, Boyuan Zheng, Ruijia Xu, Yu Gu, Yuheng Huang, Huan Sun
year: 2026
venue: ICML 2026 / NeurIPS 2025
url: https://arxiv.org/abs/2601.05111
notebooklm_id: a3e3507a-ccae-4d3b-a150-20f97cf543d5
date_read: 2026-04-03
rating: 5
status: read
relevance: 5
confidence: 4
topic: agentic-search-evaluation
key_claims:
  - "130-task long-horizon benchmark for agentic search with real-time web browsing"
  - "Agent-as-a-Judge via tree-structured rubrics — critical / non-critical / sequential nodes"
  - "99.03% judge-human alignment; humans made 27 of 35 discrepancies, judge made 7"
  - "OpenAI Deep Research reaches 50-70% human performance in half the time"
  - "Rubrics average 50 nodes; max depth 6 layers, max 603 nodes"
tags: [literature, agent-evaluation, agent-as-a-judge, agentic-search, benchmarks, reward-modeling, process-reward-models]
---

# Mind2Web 2: Evaluating Agentic Search with Agent-as-a-Judge

> [!abstract]
> Mind2Web 2 argues that the rapid rise of long-horizon agentic search (ChatGPT Search, Perplexity Pro, OpenAI Deep Research, Operator) has outpaced static benchmarks built for short, single-site tasks with verifiable string answers. It introduces (1) a **130-task benchmark** of realistic, time-varying web tasks and (2) an **Agent-as-a-Judge** framework that decomposes each task into a **tree-structured rubric** of critical / non-critical / sequential binary leaf nodes, verified by LLM-powered Extractor and Verifier tools. Judge agents hit **99.03% agreement with expert humans** — and humans actually commit 27 of 35 observed errors due to cognitive fatigue. Top frontier systems reach only 50-70% of human performance; open-source deep-research stacks fail structurally.

> [!tldr] Thesis in three bullets
> - Long-horizon agentic search produces thousand-word, time-varying answers that traditional LLM-as-a-Judge cannot reliably grade.
> - The authors exploit **generation-verification asymmetry**: generating comprehensive answers is hard, but checking them against a predefined, hierarchical rubric is tractable and automatable.
> - A rubric tree with critical / non-critical / sequential nodes + tool-augmented verification yields an automated evaluator that beats expert humans on reliability for 99%+ of leaf-level verifications.

---

## Source metadata

- **Paper**: Mind2Web 2: Evaluating Agentic Search with Agent-as-a-Judge
- **Authors**: Xiang Deng, Boyuan Zheng, Ruijia Xu, Yu Gu, Yuheng Huang, Huan Sun (OSU)
- **Venue / year**: NeurIPS 2025 poster → ICML 2026 (dual listing in notebook)
- **arXiv**: [2601.05111](https://arxiv.org/abs/2601.05111)
- **Primary NotebookLM source**: `139f2e67` — NeurIPS Poster PDF
- **Cross-referenced source**: `ea7db974` — Agent-as-a-Judge (Runyang You et al., ICML 2026, DevAI benchmark — closely related but distinct paper also in this notebook)
- **Local PDF**: `sources/` (web sources; no local PDF yet)
- **Leaderboard / code**: The authors release all 130 tasks and evaluation scripts and maintain a public leaderboard for the 120-task test set.[^s1]

[^s1]: §3.5 Benchmark Statistics — "We split our benchmark into a development set (10 tasks), and a test set (120 tasks), and release all the tasks and evaluation scripts. We will maintain a leaderboard for the test set." Source: NotebookLM `139f2e67`.

---

## 1. Problem and thesis

> [!info] Why new evaluation is needed
> Existing benchmarks target tasks of moderate horizon (≤10 actions) with predefined, static-string answers so they can be auto-graded via exact-match. But modern agentic search systems — Deep Research, Operator, Perplexity Pro — return hundreds-to-thousands-of-word answers with time-varying content (prices, hotel availability, catalog changes), often spanning dozens of websites and hundreds of browsing steps. Conventional LLM-as-a-Judge cannot reliably grade these: too long, too multi-faceted, no way to verify claims against real evidence.[^s2]

[^s2]: §1 Introduction + §2 Related Work of `139f2e67`. "The complexity is far beyond what conventional LLM-as-a-Judge [45] methods are used for … necessitating an Agent-as-a-Judge approach."

> [!quote] §1 Introduction — the thesis quote
> "While promising greater efficiency and cognitive offloading, the growing complexity and open-endedness of agentic search have outpaced existing evaluation benchmarks and methodologies, which largely assume short search horizons and static answers."

> [!quote] §1 Introduction — the key technical insight
> "The key insight behind our evaluation methodology lies in the generation-verification asymmetry: while the generated answers can vary substantially across agents, search strategies, or query times, we know a priori what each task is looking for and can design a task-specific rubric to specify the evaluation logic."

### Contributions (what is actually new)

1. **Mind2Web 2 benchmark** — 130 realistic long-horizon tasks requiring live web browsing and extensive synthesis; curated with >1,000 hours of expert human labor.[^s3]
2. **Agent-as-a-Judge evaluation paradigm** for lengthy, time-varying answers, leveraging generation-verification asymmetry.
3. **Tree-structured rubrics** with three node types — *critical, non-critical, sequential* — implementing a "gate-then-average" score aggregation that makes fundamental failures un-maskable by partial credit elsewhere.
4. **Comprehensive systems evaluation** of 10 frontier agentic search systems against human performance, exposing systemic limits on time-varying tasks.

[^s3]: §3.5 Benchmark Statistics of `139f2e67`. 130 = 10 dev + 120 test; task-creation cost reported as ">1,000 hours of expert human labor".

---

## 2. Methods

> [!info] Architecture in one paragraph
> Each task ships with a hand-curated tree-structured rubric. A **task-specific judge agent**, implemented as an agentic Python workflow, takes the evaluated system's answer text + cited URLs as input. It walks the rubric bottom-up: at each leaf, an **Extractor** LLM parses targeted structured info from the answer (item names, prices, URLs), then a **Verifier** LLM compares that information against evidence — the paper emphasises *screenshots of the cited webpages*, not just the text. Leaf verdicts are binary (0/1). Internal nodes aggregate via a "gate-then-average" rule: critical children gate the subtree; non-critical children are averaged; sequential children short-circuit on earlier failure.[^s4]

[^s4]: §3.3 Rubric Tree + §3.4 Rubric-based Judge Agent of `139f2e67`. "The Extractor extracts the corresponding bits of information from the answer, and the Verifier examines the extracted text and the screenshot of the corresponding webpage to determine if the statement is indeed true."

### 2.1 Tree-structured rubric: three node types

| Node type | Behavior | When to use |
|-----------|----------|-------------|
| **Critical** | Mandatory gate. Fails → entire parent subtree scores 0. Critical nodes don't contribute to averaging; they only enable scoring of siblings. | Fundamental task constraints (e.g., "answer is actually about the queried entity") |
| **Non-critical** | Contributes partial credit via averaging over siblings. | Enumerations (e.g., "find 5 matching products" — each found item = 1 point toward 5) |
| **Sequential** | Enforces logical ordering. A later-in-sequence node is auto-failed if any earlier sequential sibling fails. | Multi-step reasoning where step N presupposes step N−1 |

**Aggregation rule ("gate-then-average"):** a parent receives score 1 only if (a) every critical child passed AND (b) the weighted-average of its non-critical children's scores = 1. Critical nodes "warrant the meaningfulness of the evaluation but do not directly contribute to the mathematical averaging process".[^s5]

[^s5]: §3.3–3.4 of `139f2e67`. Explicit statement: "If an internal node consists entirely of critical child nodes, it only receives a score of 1 if every single child passes."

### 2.2 Extractor + Verifier tools

The judge agent's two LLM-based workhorses:
- **Extractor** — parses the (long, messy) answer text and pulls out targeted claims as structured records (item names, URLs, prices, dates, etc.).
- **Verifier** — takes each extracted claim and the cited webpage(s) as evidence, renders or fetches the page, and returns a binary true/false verdict. Cross-references include screenshots of the cited webpages, not just textual content.

> [!example] Worked example from the paper
> Task: find an IKEA product satisfying complex constraints, with a verifiable purchase link. The Extractor pulls `{product_name, price, IKEA_product_URL}` from the answer. The Verifier fetches the URL (and a screenshot), confirms the product exists, the price matches, and the constraints hold. Each check maps to a binary leaf node.

### 2.3 Automated judge-script generation

Manually authoring the rubrics + judge scripts is "prohibitively labor-intensive". The pipeline:

1. Provide the task description + reference answer to an LLM-based code-generation agent.
2. The agent drafts a judge script using a modular Python toolkit (Extractor/Verifier wrappers + tree aggregation).
3. The draft undergoes iterative **self-debug and self-reflection** — the LLM reads its own script, runs it on a reference trajectory, and refines.
4. Two-stage human expert review validates correctness and generalisability.

This is how the authors made 130 task-specific judges tractable — though they emphasise the process remains "highly demanding".[^s6]

[^s6]: §3.4 Rubric-based Judge Agent of `139f2e67`. "These generated scripts then undergo iterative autonomous refinements using self-debug and self-reflection to automatically correct minor or common errors. Finally, the scripts are rigorously validated through a two-stage human refinement process."

### 2.4 Aggregation metrics

Each task produces two headline metrics:
- **Partial Completion** — mean root-node score across all tasks (accepts partial credit).
- **Success Rate** — fraction of tasks achieving root score = 1 (strict).

---

## 3. Benchmark

> [!example] Dataset at a glance
> - **130 tasks total** = 10 dev + 120 test + a 30-task random subset ("Subset-30") used only for the human-performance study.
> - **57 tasks are time-varying** (require live-web interaction — e.g., hotel availability, fluctuating prices, relative dates). The other 73 are time-invariant but still long-horizon.
> - Domains: e-commerce (IKEA, Amazon), travel booking, academic research (paper authorship, fellowships), news/media with nuanced constraints.
> - Rubric size: **mean 50 evaluation nodes per task; max 603 nodes; max depth 6 layers**.
> - Human-effort calibration: solving a task can take **up to 1 hour**, visiting up to **31 websites and 375 webpages**.

> [!info] Why time-varying matters
> Sidestepping time-varying tasks (as concurrent work BrowseComp does by restricting to static-string answers) is a *realism compromise*: most queries that real users issue to Deep Research are exactly the ones that change — prices, availability, news, fresh research results. Mind2Web 2 insists on keeping these and absorbing the evaluation complexity.

---

## 4. Evaluated systems (the "contestants")

Ten frontier agentic search systems plus a human baseline. The paper groups them by architecture:

| Family | System | What it is |
|--------|--------|-----------|
| Search-augmented LLM | **ChatGPT Search** | Quick search-API-backed LLM; few steps |
| | **Perplexity Pro Search** | Similar; commercial retrieval + synthesis |
| Deep Research | **OpenAI Deep Research** | Closed-source, long-horizon, tool-use + citation-backed reports |
| | **Grok DeepSearch / DeeperSearch** | xAI's deep-research family; used to study test-time scaling |
| | **HF Open Deep Research** | Only open-source deep-research system producing reasonable results |
| Browser-interaction agent | **OpenAI Operator** | Direct browser control (click/scroll) in noisy web environments |
| Human baseline | 7 human participants | 3 per task on Subset-30 |

**Baseline judges** compared against Agent-as-a-Judge: (a) conventional LLM-as-a-Judge, (b) human evaluators ("Human-as-a-Judge"), (c) static string-match (the BrowseComp path, contrasted but not directly ran on Mind2Web 2 tasks).

---

## 5. Results

> [!success] Headline numbers

| Category | Metric | Value |
|----------|--------|-------|
| Judge reliability | Judge-human alignment | **99.03%** (7 actual judge errors / 720 leaf verifications) |
| Judge reliability | Rubric agreement | Humans fully agreed with **100% of 15 sampled rubrics** |
| Judge reliability | Comparison | Concurrent automated approaches on simpler web tasks report **<90%** |
| Human baseline | Success rate on Subset-30 | **54%** |
| Human baseline | Time per task | Up to **1 hour**; up to **31 websites / 375 webpages** visited |
| Best agent | Max agent success rate | **~28%** |
| Best agent | OpenAI Deep Research | **50-70% of human performance, in half the time** |
| Benchmark curation | Human labor | **>1,000 hours** |
| Rubric complexity | Mean / max nodes | **50 / 603** |
| Rubric complexity | Max depth | **6 layers** |

> [!quote] §4.4 — the reliability quote that makes the paper
> "Excluding human mistakes, only 7 out of 720 nodes reflect actual errors, indicating an exceptional accuracy of 99.03%. This demonstrates remarkable reliability, particularly when compared to recent automated evaluation approaches for relatively simpler web tasks, where the reported accuracy of the automated evaluation methods typically falls below 90%."

> [!quote] §1 Introduction — the striking finding
> "…even though current systems still underperform humans, the best-performing system, OpenAI Deep Research, can already achieve 50-70% of human performance while spending half the time. It also outperforms humans on some tasks requiring great attention to detail and exhaustiveness in the search."

**Inter-annotator subtlety:** on the 35 discrepancies between the judge and human evaluators, audit found **27 were human errors**, not judge errors. The authors attribute this to cognitive fatigue on long tasks — a pro-automation argument that also applies to the broader evaluation literature.

### Test-time scaling observation

Comparing systems within the same underlying model family (Grok DeepSearch vs. DeeperSearch; Perplexity Pro variants) shows clear gains from extended inference time on long-horizon tasks — consistent with the broader test-time scaling narrative.

---

## 6. Failure modes observed in evaluated agents

> [!warning] Systemic error taxonomy
> The paper catalogues six recurring failure modes across agent systems. These are worth memorising — they're the reward-model signals anyone training such agents needs to target.

| Failure mode | Description | Concrete example |
|--------------|-------------|------------------|
| **Incompleteness — Info Not Found** | Agent explicitly gives up retrieving info. | ChatGPT Search cuts off after a few search steps. |
| **Incompleteness — Partial Missing** | Agent returns fewer items or steps than asked. | "Find 5 products X" → returns 3. |
| **Invalid Attribution** | Fabricated or expired URLs; agent didn't actually visit. | HF Open Deep Research fabricated an Amazon purchase link without ever navigating to Amazon. |
| **Missing Attribution** | Answer relies on parametric memory, no citations. | OpenAI Operator, trained on citation-free web navigation, struggles to cite. |
| **Synthesis error** | Agent misreads a correct page. | Distorts a product's price listed on the actual product page. |
| **Retrieval error** | Agent retrieves an irrelevant page and hallucinates relevance. | Returns unrelated webpage, writes plausible-looking but false supporting detail. |
| **Struggle with time-varying tasks** | Systems without live-browsing fall back on cached / hallucinated info. | Hotel availability on specific dates; relative-date queries. |
| **Criteria violations** | Explicit constraint breaks or factual errors. | Notably: humans commit this *more* than top deep-research systems due to fatigue. |

---

## 7. Limitations of the framework itself

> [!warning] Framework-level constraints (authors' own)
> - **Rubric creation cost**: "prohibitively demanding", even with self-debug pipeline. >1,000 hrs of expert labor to produce 130 tasks + judges.
> - **Residual judge errors**: 99.03% ≠ 100%. 7 actual judge errors out of 720 leaf verifications.
> - **Strict verifiability requirement**: the framework assumes tasks can be decomposed into binary, verifiable leaves. It **cannot reliably evaluate highly subjective or open-ended tasks** (creative writing, social-intelligence interactions) where correctness isn't cross-referenceable to a cited source.

---

## 8. Assumptions

> [!question] Assumptions worth challenging
> - **Generation-verification asymmetry** — the foundational assumption. Probably true for factual web tasks; may not generalise to creative / reasoning-heavy domains.
> - **Rubric completeness** — that a tree of binary leaves can fully capture user intent. For open-ended queries this is implicitly false.
> - **LLM tool-use reliability** — Extractor + Verifier must parse messy text and ground claims without hallucinating. The 7 residual errors likely live here.
> - **Website stability during evaluation** — captured screenshots must accurately reflect the target webpage at query time. Open risk when evaluating long-lived leaderboards against changing websites.
> - **Source-document availability** — agents must provide valid, reachable citations; if a page 404s between agent-run and judge-run, attribution check may falsely flag.
> - **Scope** — explicitly limited to "objective and verifiable" tasks.

---

## 9. Positioning vs. related work

> [!info] How Mind2Web 2 positions itself
> - **Against static web-agent benchmarks (original Mind2Web, WebArena, etc.)**: those focus on ≤10 actions on a single site. M2W2 targets long-horizon, multi-site agentic search.
> - **Against GAIA / BrowseComp**: these sidestep evaluation complexity by restricting to static-string answers; M2W2 refuses that compromise and invests in automated rubric-based evaluation instead.
> - **Against LLM-as-a-Judge**: LLM judges are "passive observers" that rate based on linguistic plausibility. Agent-as-a-Judge replaces intuition with execution — grounding verdicts in actual tool-fetched evidence.
> - **Built on**: tree-structured rubrics (overlaps with PaperBench, concurrent work), Process Reward Model ideas, attribution evaluation literature.

---

## 10. Relationship to this notebook's broader theme (agentic RL + reward modeling)

> [!info] The key insight
> Mind2Web 2 is built as an *evaluator*, but its architecture — tree-structured, tool-augmented, binary-leaf-verified — is exactly what a **Process Reward Model (PRM)** for training an agentic-search policy would need. The paper doesn't claim this use case; the notebook's neighbors (iStar, SPELL, SPICE, CSQ) supply the training framing.

### How Mind2Web 2 interlocks with the notebook's other papers

- **vs. iStar (implicit step rewards via DPO)** [[maps/mind2web2-agentic-search/mindmap|→ mindmap]]
  - iStar *infers* step value without explicit labels.
  - M2W2 provides *explicit*, highly-structured step labels — dense feedback at up to 603 nodes per task.
  - Complementary: iStar for cheap scalable training; M2W2 as an offline high-quality labeler for distilling into a cheaper student reward model.

- **vs. SPELL / SPICE (self-play with verifiers)**
  - SPELL uses a single model's Questioner/Responder/Verifier roles with majority-vote consensus — purely internal grounding.
  - SPICE adds information asymmetry via hidden-document verification.
  - M2W2 pushes further: external tool-grounded verification (screenshots, fetched HTML) instead of LLM introspection.

- **vs. Counterfactual Self-Questioning (CSQ)**
  - CSQ trains a model's "ego critic" to ask "what if this step were wrong?".
  - M2W2 rubrics can provide the deterministic feedback CSQ needs — the ego-critic trains toward the tree-rubric's leaf verdicts.

- **vs. reward-hacking research (EPPO, "Energy Loss" paper, etc.)**
  - The notebook's reward-hacking papers find that reasoning-LLM judges can be fooled by adversarial policies that fabricate constraints.
  - M2W2's **critical-node gating** is a *structural* defense: no amount of partial-credit gaming saves a subtree whose critical gate failed. Combined with tool-grounded verification, it raises the adversarial bar.

> [!note]- Could tree-structured rubrics drive training (not just eval)? — expanded
> **Yes, viable but expensive.** Benefits:
> 1. Granular, interpretable per-step feedback — ideal for pairing with CSQ or PRM training.
> 2. Critical/non-critical gating imposes a logical-AND credit-assignment shape, not a summation. This blunts reward-hacking strategies that accumulate trivial-step rewards.
> 3. Dense enough to replace scalar reward models entirely in structured domains.
>
> Blockers:
> 1. **Compute cost**: invoking Extractor + Verifier per step of every rollout during RL is prohibitive at current prices.
> 2. **Rubric authoring cost**: still ~human-centuries at scale.
>
> Most realistic integration path: use M2W2 judges **offline** to build preference-pair data, then distill into a cheap learned reward model for on-policy RL (GRPO / PPO). The distilled model inherits the gating logic as learned structure.

---

## 11. Open questions

> [!question] Explicitly stated by the authors
> - How to integrate direct live-web browsing into systems currently built around search APIs only.
> - How to post-train models specifically for long-horizon agentic research (beyond chaining off-the-shelf models).

> [!question] Implicit (from failure modes)
> - How to enforce sustained agent effort over long horizons without early termination.
> - How to structurally prevent URL fabrication — context-management + strict evidence-grounding architectures.
> - How to manage long-term memory and noisy action spaces over hundreds of browsing steps.

> [!question] My open questions (pulled from my v1 reading)
> - [ ] Can the framework scale to tasks with hundreds of sequential steps without rubric-tree size exploding past 1000+ nodes?
> - [ ] How robust is judge accuracy against adversarial agent outputs (prompt injection in the agent's answer text aimed at the judge)?
> - [ ] Can tree rubrics be compiled into a differentiable reward (e.g., expected-gate surrogate) for on-policy RL?
> - [ ] How does judge agreement hold up when the cited webpage has changed between agent-run and judge-run (time-drift on time-varying tasks)?

---

## 12. Embedded distilled materials

> [!note]- Briefing doc (transcluded) — click to expand
> ![[distilled/mind2web2-agentic-search/briefing-doc]]

> [!note]- Study guide (transcluded) — click to expand
> ![[distilled/mind2web2-agentic-search/study-guide]]

> [!note]- Quiz (transcluded) — click to expand
> ![[distilled/mind2web2-agentic-search/quiz]]

> [!note]- Flashcards (transcluded) — click to expand
> ![[distilled/mind2web2-agentic-search/flashcards]]

---

## 13. Cross-links

- **Mind map**: [[maps/mind2web2-agentic-search/mindmap]]
- **Related literature** in this notebook:
  - Agent-as-a-Judge (Runyang You et al., ICML 2026) — sibling paper, introduces DevAI, studies component ablations (`ea7db974`)
  - Agentic RL with Implicit Step Rewards (iStar)
  - SPELL — self-play for long-context reasoning
  - SPICE — self-play in corpus environments
  - Counterfactual Self-Questioning
  - Examining Reasoning LLMs-as-Judges — adversarial judge-hacking behaviors
  - Energy Loss Phenomenon in RLHF
- **Related project**: _(none yet — link when started)_
- **Related experiments**: _(none yet)_

---

## 14. Provenance

- Built from 12-probe NotebookLM Q&A run on 2026-04-17. Raw probe JSON: `research-os/scratch/probe{01..12}_*.json`.
- Primary source throughout: NotebookLM source `139f2e67` (NeurIPS poster PDF).
- Quotes verbatim from the paper (§-locations indicated).
- v1 of this note archived at `mind2web2-agent-as-a-judge.v1.md` for comparison.
