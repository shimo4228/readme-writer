# readme-writer

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/readme-writer) [![GitMCP](https://img.shields.io/endpoint?url=https://gitmcp.io/badge/shimo4228/readme-writer)](https://gitmcp.io/shimo4228/readme-writer) [![View Code Wiki](https://assets.codewiki.google/readme-badge/static.svg)](https://codewiki.google/github.com/shimo4228/readme-writer)

An [Agent Skill](https://agentskills.io/specification) that writes and improves **human-facing READMEs** — the single canonical entrance where humans, search, and AI Overviews all land. It is the human-surface counterpart to [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer): that skill optimizes the AI surface (`llms.txt` / `llms-full.txt`), this one owns the human surface.

The skill splits README quality into two kinds of property and gives each its proper owner, following **AKC ADR-0008 "Code-LLM Layering"**:

- **Structural hygiene** (single H1, no skipped heading levels, image alt-text, local-link resolution) is **100%-deterministic** — checked by `readme_lint.py`.
- **Semantic quality** (does the lead grab a human, is the value proposition clear, does the narrative flow) is a **judgment** — handled by a holistic LLM review. **No score is produced.**

## Install

### Claude Code

```bash
# Copy skill into your global skills directory (no runtime deps — stdlib only)
cp -r skills/readme-writer ~/.claude/skills/readme-writer
```

### SkillsMP

```bash
/skills add shimo4228/readme-writer
```

## How It Works

The workflow is the AKC layering pattern: **code filter → LLM review → human gate.**

1. **Code filter — `readme_lint.py`** (deterministic, structural only). Emits concrete issues, not a score.
2. **LLM holistic review** (rubric-as-lens, never scored). Claude reads the README and returns direct findings plus small `y/n` diffs.
3. **Fact-consistency check** is delegated to [`context-sync`](https://github.com/shimo4228/context-sync) — the README must not diverge from `llms.txt` / `graph.jsonld` (diverging content is cloaking).
4. **Human gate** — diffs are approved and applied by a human.

```bash
# Structural lint (exit code is the code-owned gate: 0=clean / 1=issue / 2=not found)
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_lint /path/to/README.md

# JSON output for machine consumption
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_lint --json /path/to/README.md
```

## Structural Checks (`readme_lint.py`)

Every check is structural and reports a concrete issue — never a quality score.

| Check | What it verifies | Why |
|-------|------------------|-----|
| `single_h1` | Exactly one `#` H1 | Document outline / SEO title clarity |
| `heading_levels` | No skipped levels (e.g. H2 → H4) | Accessible, parseable outline |
| `alt_text` | Every image has alt text | Accessibility + image SEO |
| `local_link` | Local relative links / image `src` resolve to real files | No broken links on the landing page |

## Why No Score?

A `geo_check.py`-style static scorer was deliberately **rejected** for READMEs. `geo_check`'s section-ratio metrics rest on empirical LLM-citation research; README "quality" (human hook, value proposition, narrative) is a semantic judgment with **no equivalent deterministic base**. Per **signal-first / scaffold-dissolution** (AKC), the skill emits only what changes the next action — `Lead: 6/10` changes nothing, "the lead never says who it's for" does. A quality score that no code gate consumes is scaffolding that compresses (and kills) a high-capability model's judgment. Importing AI-surface metrics (ski-ramp, entity density) into a human README is an anti-pattern that degrades readability.

## When to Use

- Creating or improving `README.md` / `README.ja.md`
- Tuning the page a human lands on first for a repo or project
- Fixing a README that reads "machine-ish and half-baked for humans"

**Do not use for:**
- `llms.txt` / `llms-full.txt` / FAQ / glossary → [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer)
- `graph.jsonld` design → [`jsonld-knowledge-graph`](https://github.com/shimo4228/jsonld-knowledge-graph)
- Cross-surface drift detection / sync → [`context-sync`](https://github.com/shimo4228/context-sync)
- Articles / essays / blog posts → [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)

## Requirements

- Python >= 3.11
- No runtime dependencies (standard library only). `uv` optional, for running tests.

## Tests

```bash
cd skills/readme-writer && uv run pytest -v  # 59 tests
```

## Related skills (siblings)

- [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) — the AI-surface counterpart (writes `llms.txt` / `llms-full.txt` with the research-backed `geo_check.py` scorer)
- [`jsonld-knowledge-graph`](https://github.com/shimo4228/jsonld-knowledge-graph) — companion JSON-LD graph for the concept surface
- [`context-sync`](https://github.com/shimo4228/context-sync) — audits cross-surface fact consistency (README ↔ machine layer)
- [`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem) — orchestrator for long-form human prose (articles, essays)

## About this skill

This skill is a **component skill of the [Authorship Strategy](https://github.com/shimo4228/authorship-strategy) research line** ([DOI 10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)) maintained by [@shimo4228](https://github.com/shimo4228). Authorship Strategy's [ADR-0006](https://github.com/shimo4228/authorship-strategy/blob/main/docs/adr/0006-llm-first-ingest-dual-entry-points.md) normatively requires an LLM-facing dual entry point (prose navigator + concept graph) for every framework-governed artifact; this skill covers the **third, human-facing surface** that those two do not — keeping the README a clean human entrance whose facts stay consistent with the machine layer (divergence would be cloaking). Its design rationale traces to **AKC ADR-0008 "Code-LLM Layering"** ([Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle), [DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)).

## License

MIT

---

## 日本語

人間（と、人間が着地する検索 / AI Overviews）に向けた README を書く・改善するスキルです。AI surface（`llms.txt` / `llms-full.txt`）を担う [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) に対し、本 skill は **人間 surface の単一正準入口**を担います。

README 品質には 2 種類の property が混在します。**AKC ADR-0008「Code-LLM Layering」**に従い所有者を分けます:

- **構造的衛生**（H1 が 1 個・見出しレベル飛ばし無し・画像 alt-text・ローカルリンク解決）は決定論的な `readme_lint.py` が 100% 精度で検査。
- **意味的品質**（lead が人を掴むか・価値提案の明快さ・物語の流れ）は LLM のホリスティック review が担当。**スコアは付けません**。

スコアを作らない理由は signal-first / scaffold-dissolution（AKC）です。「次の行動を変える情報」だけを出す — `Lead: 6/10` は行動を変えませんが、「lead が誰向けか言っていない」は変えます。AI surface の指標（ski-ramp / entity density）を人間 README に流用するのは可読性を下げる anti-pattern です。

詳細は [`skills/readme-writer/SKILL.md`](skills/readme-writer/SKILL.md) を参照してください。
