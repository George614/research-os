# CLAUDE.md — research-os

Project-level instructions auto-loaded by Claude Code when working in this directory.

## Purpose of this repo

This repo is the **system** behind the research-os workflow — templates, skills, vault configuration, and a single worked example (Mind2Web 2). It is **not** a storage location for the user's ongoing literature notes, projects, experiments, or NotebookLM outputs. That content stays local; only system improvements get pushed.

## Auto-sync rule

**After any change to a system file (list below), commit and push to `origin/main` before ending the turn.**

Never commit user-data paths (list below). The `.gitignore` enforces this, but do a last-pass diff check before every commit anyway.

### System paths (track + sync)

| Path | What it is |
|------|-----------|
| `CLAUDE.md`, `README.md`, `.gitignore` | Repo meta |
| `templates/*.md` | Literature / project / experiment note templates |
| `skills/notebooklm-research/SKILL.md` | Mirror of the live `~/.claude/skills/notebooklm-research/SKILL.md` |
| `skills/obsidian-research-vault/SKILL.md` | Mirror of the live `~/.claude/skills/obsidian-research-vault/SKILL.md` |
| `obsidian-vault/.obsidian/{app,appearance,community-plugins,core-plugins}.json` | Vault config — documents required plugins |
| `obsidian-vault/.obsidian/plugins/*/manifest.json` | Plugin manifests (but **not** main.js/styles.css/data.json) |
| `obsidian-vault/literature/mind2web2-agentic-search/` | The worked-example dossier (kept as a demo) |
| `obsidian-vault/maps/mind2web2-agentic-search/` | Worked-example mindmap |
| `obsidian-vault/distilled/mind2web2-agentic-search/` | Worked-example distilled artifacts |
| `obsidian-vault/_index.md` | Dataview-driven MOC (content is query-only; no user data) |
| `scripts/**` | Any helpers |

### User-data paths (NEVER commit)

| Path | Why |
|------|-----|
| `obsidian-vault/literature/*/` (except `mind2web2-agentic-search/`) | Personal paper notes |
| `obsidian-vault/maps/*/` (except the demo) | Per-paper mindmaps |
| `obsidian-vault/distilled/*/` (except the demo) | Per-paper distilled content |
| `obsidian-vault/projects/*.md` | Personal research projects |
| `obsidian-vault/experiments/*.md` | Personal experiments |
| `sources/*` | Raw PDFs |
| `exports/*` (new files — worked-example exports are already tracked) | Slide decks, infographics, CSVs |
| `scratch/*` (new files — worked-example probes are already tracked) | NotebookLM Q&A JSON from future runs |
| `obsidian-vault/.obsidian/workspace*.json` | Per-user pane layout |
| `obsidian-vault/.obsidian/plugins/*/main.js`, `styles.css`, `data.json` | Plugin binaries / local state |

## Skill-file sync

The canonical "live" skills run from `~/.claude/skills/{notebooklm-research,obsidian-research-vault}/SKILL.md`. The repo carries mirror copies under `skills/<id>/SKILL.md`.

**When editing a skill, edit both locations:**

- Live version (what Claude actually loads):
  - `~/.claude/skills/notebooklm-research/SKILL.md`
  - `~/.claude/skills/obsidian-research-vault/SKILL.md`
- Repo mirror (what gets pushed):
  - `skills/notebooklm-research/SKILL.md`
  - `skills/obsidian-research-vault/SKILL.md`

Preferred method: edit the live version first (so runtime behavior updates immediately), then copy the file into the repo mirror and commit. A one-liner to keep them in sync:

```bash
cp ~/.claude/skills/notebooklm-research/SKILL.md  skills/notebooklm-research/SKILL.md
cp ~/.claude/skills/obsidian-research-vault/SKILL.md  skills/obsidian-research-vault/SKILL.md
```

## Commit workflow (follow exactly)

1. **Stage only system files:** after editing, inspect `git status` and ensure nothing from the user-data table is in "Changes not staged" or "Untracked".
2. **Diff-check for leaks:** run
   ```bash
   git diff --cached --name-only | \
     grep -E '^(obsidian-vault/(literature|maps|distilled)/|obsidian-vault/(projects|experiments)/|sources/|scratch/|exports/)' | \
     grep -vE 'mind2web2-agentic-search' || echo "OK — no user-data paths staged"
   ```
   If the output is non-empty (and not the tracked Mind2Web demo), abort and unstage those files — they're user data.
3. **Commit with a descriptive message** naming the system change (e.g. `feat(skill): add 12-probe Q&A pattern`, `docs(template): add Dataview frontmatter fields`, `fix(vault-config): enable Mindmap NextGen by default`).
4. **Push to origin**: `git push origin main`.
5. Before ending the turn, confirm `git status` is clean.

## Safety rails

- **Never `git push --force` / `--force-with-lease`** unless the user explicitly asks — history rewrites are user-visible actions.
- **Never commit** anything matching `.env`, `*.secret`, API keys, tokens, or auth cookies (the `.gitignore` covers common cases; still grep before pushing).
- **Never bypass hooks** with `--no-verify`.
- If the user is mid-debugging and the system is in a known-broken state, **hold commits until they confirm** — don't push a broken state.
- If unsure whether a file is "system" or "user data", **ask before committing**.

## When NOT to sync

- User explicitly says "don't commit this" or "just test locally"
- User is iterating rapidly (5+ edits in quick succession) — batch into one commit at a logical pause point
- The only change is to a user-data file — skip the commit; nothing to sync
- You're in the middle of reproducing a user-reported bug and haven't fixed it yet

## Example commit messages

```
feat(skill): add 12-probe Q&A pattern to notebooklm-research
docs(template): add key_claims + confidence fields to literature-note
fix(vault): document Mindmap NextGen heading-only format
chore(gitignore): exclude .obsidian plugin binaries
refactor(readme): clarify plugin install steps
```

Keep subject ≤72 chars, imperative mood, prefix with `feat|fix|docs|chore|refactor|style|test`.
