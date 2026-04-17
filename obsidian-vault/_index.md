---
type: index
updated: 2026-04-17
---

# Research Vault Index

> [!info] Rendering
> This index uses the [Dataview](https://github.com/blacksmithgu/obsidian-dataview) community plugin. Install "Dataview" from Obsidian → Settings → Community plugins. Without it, you'll see the raw code blocks instead of the generated tables — scroll to the **Fallback listing** section at the bottom for a hand-curated view.

## Active Projects

```dataview
TABLE status, priority, last_active
FROM "projects"
WHERE type = "project" AND status != "archived"
SORT priority DESC, last_active DESC
```

## Recent Literature

```dataview
TABLE year, venue, rating, relevance, status, topic
FROM "literature"
WHERE type = "literature"
SORT date_read DESC
LIMIT 20
```

## High-Relevance Reading List

```dataview
TABLE year, venue, relevance, rating, topic
FROM "literature"
WHERE type = "literature" AND relevance >= 4
SORT relevance DESC, rating DESC
```

## To-Read Queue

```dataview
LIST FROM "literature"
WHERE type = "literature" AND status = "to-read"
SORT relevance DESC
```

## Recent Experiments

```dataview
TABLE status, outcome, project, created, completed
FROM "experiments"
WHERE type = "experiment"
SORT created DESC
LIMIT 20
```

## Running / Open Experiments

```dataview
TABLE status, project, created
FROM "experiments"
WHERE type = "experiment" AND (status = "running" OR status = "planned")
SORT created DESC
```

## Maps

```dataview
LIST source_paper
FROM "maps"
WHERE type = "map"
SORT file.name ASC
```

## Literature by Topic

```dataview
TABLE rows.file.link AS Papers
FROM "literature"
WHERE type = "literature"
GROUP BY topic
SORT topic ASC
```

---

## Fallback listing (no plugin required)

Hand-curated; update when new notes land:

- **Literature**
  - [[literature/mind2web2-agentic-search/mind2web2-agent-as-a-judge]] — Mind2Web 2: Evaluating Agentic Search with Agent-as-a-Judge (2026)
- **Maps**
  - [[maps/mind2web2-agentic-search/mindmap]] — Agentic RL & Reward Modeling mindmap
- **Distilled**
  - [[distilled/mind2web2-agentic-search/study-guide]], [[distilled/mind2web2-agentic-search/briefing-doc]], [[distilled/mind2web2-agentic-search/flashcards]], [[distilled/mind2web2-agentic-search/quiz]]
