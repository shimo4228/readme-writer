# readme-writer

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/readme-writer)

An [Agent Skill](https://agentskills.io/specification) for writing and repairing the **README**: the page a visitor opens first, and the one surface an LLM is guaranteed to ground on when a repo URL is pasted into a chat or surfaced by AI search.

**Why it exists.** READMEs grow by accretion. Every release adds a bullet, an ADR number, a sibling repo, a coined term, until a first-time visitor cannot tell what the project is for, and an LLM reading only the README cannot reconstruct it. This skill exists to reverse that, and it starts from one condition: **before any mechanism, install step, or feature list, the reader must be able to say what the repository is for.** If the purpose is not standing, nothing that follows lands. The skill then keeps the page short and scannable for people while preserving a small, non-negotiable **information floor** that lets an LLM recover the project from the README alone.

It is the human-surface counterpart to [`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer) (AI-only surface).

## How It Works

Code counts; an LLM judges; a human decides. Three parts:

1. **`scripts/readme_evidence.py`** (stdlib Python) extracts **evidence, not a verdict**: first-screen length and new terms, ADR / sibling-repo / docs references, coined-term candidates, what `<details>` blocks hide, whether every figure has prose after it, internal-history lines, raw numeric claims, slop words and em-dashes. JSON out. No score, no threshold, no failing exit code.
2. **`readme-judge`**, a fresh-context judge agent, reads the README once with that JSON as one input, answers a fixed checklist with one-line quoted evidence, runs a refutation pass against its own findings, and returns a **named verdict**: Publishable / Fix / Rewrite. Fix findings are span-level only. The checklist is in [`references/readme-judge-checklist.md`](skills/readme-writer/references/readme-judge-checklist.md).
3. **Revision loop with a hard cap of two rounds**, then a review panel (`readme-reviewer`, `readme-clarity-reviewer`, optionally a cross-model `codex-review`), a binding re-judgment on the frozen candidate, and the author's read-through as the final gate. Round three is where voice starts normalizing toward corporate About-page prose, so the loop stops before it.

```bash
# Evidence JSON for a README (exit 0 always; 2 only if the file is missing or too large)
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence /path/to/README.md

# Human-readable summary
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence --text /path/to/README.md
```

## What the Skill Enforces

- **Purpose first.** The floor's second element is *why this exists*. This is a condition, not a section template: it may live in the identity sentence or the lead. Projects that are not tools (experiments, observation apparatus, research instruments) say so and name their output (a history, a dataset, a record) rather than describing an audience "for whom this is a tool". A project that runs on an external platform says whether the platform is the purpose or the field.
- **Promises get destinations.** If the lead promises an artifact ("the history of how the constitution changed"), the README must offer a reachable link, section, or dated count in the same change.
- **Coined-term budget** of roughly six per README, each glossed at first use, never two in one sentence.
- **References are pointers, not explanations.** Deleting an ADR number or sibling-repo link must leave a sentence that still carries meaning.
- **Every figure has a one-sentence text equivalent.** Mermaid over raster; nothing load-bearing inside images or collapsed blocks.
- **No word-count targets, no quality score.** A `Lead: 6/10` changes nothing; "the lead never says who it is for" changes the next edit.
- **Japanese READMEs use ですます.** A deliberate departure from the だ/である register used for essays: a README is a first meeting.

## Install

The skill depends on three judge / reviewer agents that live in [`claude-harness`](https://github.com/shimo4228/claude-harness): `readme-judge`, `readme-reviewer`, `readme-clarity-reviewer`. Install the skill and the agents together.

```bash
# Skill
cp -r skills/readme-writer ~/.claude/skills/readme-writer

# Agents (from the claude-harness checkout)
cp /path/to/claude-harness/agents/readme-judge.md \
   /path/to/claude-harness/agents/readme-reviewer.md \
   /path/to/claude-harness/agents/readme-clarity-reviewer.md \
   ~/.claude/agents/
```

`/skills add shimo4228/readme-writer` (SkillsMP) installs the skill only; add the agents by hand.

## When to Use

- Creating or improving `README.md` / `README.ja.md`
- A README that has grown by accretion and no longer reads on first contact
- Moving long rationale, ADR references, and internal history out of the README into `docs/`
- Aligning the GitHub About (description / topics) with what the README claims

**Not for:** `llms.txt` / `llms-full.txt` / FAQ ([`llms-txt-writer`](https://github.com/shimo4228/llms-txt-writer)); `graph.jsonld` ([`jsonld-knowledge-graph`](https://github.com/shimo4228/jsonld-knowledge-graph)); cross-surface drift repair ([`context-sync`](https://github.com/shimo4228/context-sync)); articles and essays ([`claude-skill-writing-ecosystem`](https://github.com/shimo4228/claude-skill-writing-ecosystem)).

## Requirements

Python >= 3.11, no runtime dependencies. `uv` for the tests:

```bash
cd skills/readme-writer && uv run pytest -v
```

## About this skill

A component skill of the [Authorship Strategy](https://github.com/shimo4228/authorship-strategy) line ([DOI 10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)) by [@shimo4228](https://github.com/shimo4228). Authorship Strategy's ADR-0006 requires an LLM-facing dual entry point (prose navigator + concept graph) for every governed artifact; this skill owns the third, human-facing surface and keeps its facts consistent with the machine layer. The code-counts / LLM-judges split follows AKC ADR-0008 "Code-LLM Layering" ([Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle), [DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)). Design sources are in [`inspiration.md`](skills/readme-writer/inspiration.md).

## License

MIT

---

## 日本語

README（repo を開いた人が最初に見るページ。AI 検索やチャットに URL を貼ったとき LLM が必ず前提にする唯一の面でもあります）を書く・直すためのスキルです。

**なぜあるのか。** README は継ぎ足しで育ちます。リリースのたびに bullet・ADR 番号・姉妹 repo・造語が増え、初めて来た人には何のプロジェクトか分からず、README だけを読む LLM にも復元できなくなります。このスキルはそれを戻すためにあり、出発点は 1 つの条件です。**仕組み・インストール・機能の説明に入る前に、読者が「この repo は何のためにあるか」を言えること。** 目的が立っていなければ、後に続く記述は何も入ってきません。そのうえで、人向けに短く走査しやすく保ちながら、LLM が README 一枚から project を復元できる最小の情報フロアを残します。

動かし方は「code が数え、LLM が判定し、人が決める」の 3 段です。`readme_evidence.py` が証拠だけを JSON で出し（verdict も閾値も持ちません）、fresh context の判定器 `readme-judge` が固定チェックリストに引用付きで答えて Publishable / Fix / Rewrite を返し、改稿は上限 2 ラウンドで止め、panel と binding 再判定を経て著者の通読が最後のゲートになります。判定器・reviewer の 3 agent は [`claude-harness`](https://github.com/shimo4228/claude-harness) 側にあります。

日本語 README の地の文はですます調です（エッセイの だ/である から意図的に分けています。README は初対面の案内だからです）。詳細は [`skills/readme-writer/SKILL.md`](skills/readme-writer/SKILL.md) を参照してください。
