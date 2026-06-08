# Changelog

All notable changes to this project will be documented in this file.

## [0.1.0] — 2026-06-08

Initial public release.

### What it does

An [Agent Skill](https://agentskills.io/specification) that writes and improves
human-facing READMEs — the single canonical entrance where humans, search, and
AI Overviews land. The human-surface counterpart to
[claude-skill-llms-txt-writer](https://github.com/shimo4228/claude-skill-llms-txt-writer).

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
