# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Added
- `agents/readme-judge.md` is bundled, with an idempotent `install.sh` that installs the skill and
  the agent into `~/.claude` and runs `uv sync` (previously the agent had to be fetched from
  claude-harness).
- `scripts/readme_render.py`: renders the README through GitHub's Markdown API, styled with a copy of github-markdown-css (an
  approximation of GitHub's look), at
  the README column's width (desktop and mobile, light and dark) and reports screenshots, heading
  positions, the fold and overflowing elements. The judge answers a visual section (§V) from them.
  Needs Playwright (now the one runtime dependency) and an authenticated GitHub CLI; optional.
- Visitor read (`references/visitor-read.md`): for rewrites meant to get people to try a project,
  three first-time visitors chosen for the repository read each draft and report where they stopped,
  whether they would try it, and what pushed them away.
- Overview diagram guidance (`references/overview-diagram.md`) for repositories whose mechanism a
  reader must understand before starting.
- The reader-cut agreement test (`scripts/reader_cut*.py`, `evals/reader-cut/`), whose first result
  stopped the sentence-level arm.

### Changed
- Layout: the visible text serves people (what the project is, then whatever stands between the reader
  and a first run); the LLM-read information floor moves to a collapsed section at the end of the
  README. The judge asks for docs/ once the raw README passes about 15,000 characters.
- One judge: `readme-judge` now also checks each claim against the repository's code, and runs a
  before/after comparison in the final judgment. The review panel (`readme-reviewer`,
  `readme-clarity-reviewer`, `codex-review`) is retired; a cross-model opinion is optional through the
  `codex@openai-codex` plugin.
- `readme_evidence.py` is split into sibling modules and reports the README's size.

### Fixed
- A `<details>` tag named in inline code no longer counts as a collapsed block.

## [0.2.0] — 2026-08-20

The lint-based design is replaced by an evidence + fresh-context judge design.

### Added
- `scripts/readme_evidence.py` — deterministic evidence extractor (stdlib only).
  Emits JSON (or `--text`): first-screen length and new terms, identity lead,
  ADR / sibling-repo / docs references, coined-term candidates, `<details>`
  contents, figures with prose-after check, badges, slop words and em-dashes,
  internal-history lines, numeric claims, DOI / citation presence. No verdict,
  no threshold, exit 0 always (2 on missing / oversized file).
- `references/readme-judge-checklist.md` — the fixed checklist read by the
  `readme-judge` agent: R1–R14 (first screen, sentence density, context budget)
  and K1–K6 generalized from author findings. K5 (missing purpose: the reader
  must be able to say what the repo is for before any detail; not a heading
  template) and K6 (lead promises need a reachable destination) were added
  from the contemplative-agent README rewrite on 2026-08-20.
- `references/visual.md`, `references/ja-register.md`, `references/about.md`.
- `evals/fixtures/` — pre-rewrite real READMEs with expected judge findings.
- `tests/test_readme_evidence.py`.
- SKILL.md: the information floor now names *why the project exists* as a
  floor element (non-tool repos say so and name their output; platform =
  purpose or field), the 11-step workflow (evidence → judge draft gate →
  two-round cap → panel → binding re-judgment → other languages → read-only
  fact check → About proposal → author read-through), ですます register for
  Japanese READMEs, no word-count targets.

### Removed
- `scripts/readme_lint.py` and `tests/test_readme_lint.py`. The four
  structural checks moved into the evidence extractor's `structure` block;
  the exit-code gate is gone because the judge, not a code gate, owns the
  verdict.

### Changed
- Verdicts are named and non-aggregated (Publishable / Fix / Rewrite), issued
  by a fresh-context agent rather than an in-session holistic review.
- Judge and reviewer agents (`readme-judge`, `readme-reviewer`,
  `readme-clarity-reviewer`) are published in
  [claude-harness](https://github.com/shimo4228/claude-harness); this repo
  ships the skill only.
- Repository renamed to drop the `claude-skill-` prefix. This skill follows the
  open Agent Skills standard and is not Claude-specific; the old name implied
  otherwise. The previous URL redirects to the new one.
- Added a `compatibility` frontmatter field (per the Agent Skills spec).
- Updated sibling cross-references to the renamed sibling repositories.

## [0.1.0] — 2026-06-08

Initial public release.

### What it does

An [Agent Skill](https://agentskills.io/specification) that writes and improves
human-facing READMEs — the single canonical entrance where humans, search, and
AI Overviews land. The human-surface counterpart to
[llms-txt-writer](https://github.com/shimo4228/llms-txt-writer).

It splits README quality into two properties with separate owners, per
**AKC ADR-0008 "Code-LLM Layering"**:

1. **Structural hygiene** — deterministic, owned by `readme_lint.py`.
2. **Semantic quality** — a judgment, owned by a holistic LLM review. No score.

### Components

- `SKILL.md` — when-to-use boundaries, the code-filter → LLM → human-gate
  workflow, structural-vs-semantic ownership table, anti-patterns.
- `scripts/readme_lint.py` — deterministic structural linter (stdlib only).
  Checks `single_h1`, `heading_levels`, `alt_text`, `local_link`. Emits concrete
  issues (never a score). Text or `--json` output; exit code 0/1/2 as the
  code-owned gate.
- `tests/test_readme_lint.py` — 59 passing unit + integration tests.
- `fixtures/sample_clean.md` (0 issues), `fixtures/sample_issues.md` (multiple
  issues) — worked examples.
- `inspiration.md` — origin story and canonical pointers, kept out of `SKILL.md`
  for portability.

### Design decisions

- **No scorer.** A `geo_check.py`-style static analyzer was rejected for
  READMEs: section-ratio metrics rest on empirical LLM-citation research, but
  README "quality" is a semantic judgment with no equivalent deterministic base.
  Grounded in signal-first / scaffold-dissolution (AKC).
- **Structural / semantic split** decided via the `when-code-when-llm` axis
  ("can the same byte string mean different things depending on context?").

### Requirements

- Python >= 3.11
- No runtime dependencies (standard library only)

### Tests

```bash
cd skills/readme-writer && uv run pytest -v  # 59 tests
```
