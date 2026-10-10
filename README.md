# readme-writer

**English** | [日本語](README.ja.md)

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/readme-writer)

readme-writer is a Claude Code skill that rewrites or reviews a README so that a first-time visitor can
tell what the project is and whether it is for them. A separate judge, a Claude Code subagent that
starts with no memory of the writing session and answers a fixed checklist, checks the page against the
repository's own code and returns fixes for specific sentences, not a score. You read the result last
and decide. The author's other work is listed under [More from the author](#more-from-the-author).

## What it catches

On 2026-10-07 it rewrote the README of
[jev-research-pipeline](https://github.com/shimo4228/jev-research-pipeline), a command-line tool
that writes a research note each morning, from 332 lines
([before](https://github.com/shimo4228/jev-research-pipeline/blob/6a53b9f/README.md)) to about 190
([after](https://github.com/shimo4228/jev-research-pipeline/blob/556e694/README.md)). Three facts went
wrong in the drafts, and the judge, checking the code and the old page, caught all three before
publication:

- A draft said every topic gets a note every morning. The code runs three topics a morning in
  rotation (`store/rotation.py`); the old page had said so, and the shortening lost it.
- The old page, and the drafts after it, said to put `uvx` in front of every command to try the tool
  without installing. One command, `jrp schedule install`, refuses to run without an installed copy
  (`cli.py`).
- A draft dropped the old page's note that jrp's sample notes were written by Claude Opus while the
  model jrp writes with by default, GPT-6 Luna, had not been measured in English or Chinese. The judge
  asked for it back.

This is the judge's report on the first one, condensed:

> **Draft L17–20 contradicts the code.** "every morning jrp checks … and writes one Markdown note per
> topic" against `src/jev_research_pipeline/store/rotation.py:24` (`per_tick = 3`). Fix: state that
> each morning runs the next three topics in a fixed rotation.

The published page reads: "Every morning jrp takes the next three topics in turn (a setting), checks
new papers and repositories … and writes one Markdown note per topic."

Before the judge, three AI readers, each given the background of a likely visitor, read every draft and
said where they stopped reading and why: an unexplained paid API key and no example of the output, both
fixed.

## Install

```bash
git clone https://github.com/shimo4228/readme-writer && cd readme-writer
./install.sh
```

`install.sh` copies the skill to `~/.claude/skills/readme-writer` and the judge agent to
`~/.claude/agents/readme-judge.md`, then installs the skill's Python dependencies with `uv sync`. An
existing copy that differs is moved to `~/.claude/backups/install-<timestamp>/`; `--dry-run` shows
what it would do. You need Python 3.11 or later and [uv](https://docs.astral.sh/uv/). A rewrite
runs the judge two to four times, at about 200,000 to 300,000 tokens a pass in the example above
([Cost and limits](#cost-and-limits)).

To let the judge also look at the page the way GitHub draws it (desktop and mobile, light and dark),
sign in to the GitHub CLI (`gh auth login`) and install the browser once, from
`~/.claude/skills/readme-writer`: `uv run playwright install chromium`. That step sends the README's
text to GitHub's Markdown API to draw it. Without these the judge works from the text alone.

## Use

Ask Claude Code in plain words; the skill picks the mode.

| you say | what happens |
|---|---|
| "rewrite this README" | full rewrite: agrees with you first which sections the page will have, then writes, judges, fixes, and hands you the draft (with screenshots, if set up) to read |
| "update the README for this change" | rewrites only the sections the change touched, then one judgment |
| "review this README" | judgment only; nothing is rewritten |
| "the GitHub About doesn't match" | a proposal for the description, topics and homepage in the repository page's About box |

It works on English and Japanese READMEs and keeps the language versions in step. A rewritten README
keeps what people need in the visible text and puts the facts an AI assistant needs in a collapsed
section at the end, like "For tools and AI assistants" at the bottom of this page.

## Cost and limits

- **Tokens.** The figure of 200,000 to 300,000 tokens a judge pass comes from the jev-research-pipeline
  rewrite: an English README of about 190 lines plus its Japanese version, checked against a mid-sized
  Python codebase, with the READMEs, the code and the screenshots read in each pass.
- **You are the last gate.** The judge can miss things, so a rewrite always ends with your
  read-through. The [log](skills/readme-writer/evals/read-through-log.md) (in Japanese) records what
  that read-through found after each final judgment. As of 2026-10-09 it lists twelve rewrites, all on
  the author's own repositories. Ten have been read through: six had nothing left to fix and four had
  one to five issues (overstated facts in the earliest, before claims were checked against the code;
  choices of structure and framing in the later three). The other two still await a read-through.
- **It will not invent evidence.** If the reader would need a fact the repository does not have (a
  benchmark, a free tier), the skill reports it to you instead of writing around it.

## More from the author

- **[Is a README for Humans or for LLMs?](https://dev.to/shimo4228/is-a-readme-for-humans-or-for-llms-2206)**
  ([日本語](https://zenn.dev/shimo4228/articles/readme-human-llm-fold)): why the visible text is for
  people and the facts for LLMs sit in a collapsed section at the end; all five AI assistants that
  fetched the author's test README, about 26,000 characters long, read the collapsed text and the end.
- **[llms-txt-writer](https://github.com/shimo4228/llms-txt-writer)**: the companion skill for pages
  only AI reads (`llms.txt`, `llms-full.txt`, FAQ, glossary).
- **[jsonld-knowledge-graph](https://github.com/shimo4228/jsonld-knowledge-graph)**: the companion
  skill for a `graph.jsonld` beside `llms.txt` that states a project's concepts and their relations as
  schema.org triples.
- **[claude-harness](https://github.com/shimo4228/claude-harness)**: the Claude Code setup this skill
  comes from, with the judge agent and the design records behind it.
- **[Authorship Strategy](https://github.com/shimo4228/authorship-strategy)**: why a README is the one
  page both people and AI are sure to read, and how an author stays visible when readers meet ideas
  through LLMs.
- **[shimo4228](https://github.com/shimo4228/shimo4228)**: the author's hub, with Authorship Strategy
  next to the author's other long-running projects and their DOIs.

## License

MIT

<details>
<summary>For tools and AI assistants</summary>

**What it is.** readme-writer is an Agent Skill for Claude Code (Python, MIT, version 0.2.0 plus
unreleased changes listed in [CHANGELOG.md](CHANGELOG.md)) that writes, rewrites, reviews and keeps
in step the human-facing README of a repository, and the GitHub About fields that summarize it. It is
the human-facing counterpart of llms-txt-writer, which writes pages meant only for AI.

**Requirements.** Claude Code, Python 3.11 or later and uv; the `readme-judge` agent, bundled in
`agents/` (`install.sh` installs the skill and the agent together); optionally an authenticated GitHub CLI and Playwright's Chromium for render evidence.
No paid API key beyond the Claude Code plan. Status: active; the skill is synced one way from the
author's Claude Code harness (`scripts/sync-from-local.sh`, which never commits), so the harness copy
can be ahead between syncs.

**Why it exists.** READMEs grow by accretion: each release adds a bullet, a design-record number, a
sibling repository or a coined term, until a first-time visitor cannot tell what the project is for.
The skill reverses that under one rule: before any mechanism, install step or feature list, the reader
must be able to say what the repository is for. Code counts, an LLM judges, and a human decides.

**How a README is laid out.** People read a README once from the top and leave when it stops making
sense; an AI assistant that fetches the URL reads everything, including collapsed sections and the end
of the page. So the visible text serves people (what the project is, then whatever stands between the
reader and a first run), and the information an LLM needs to reconstruct the project without opening
any other file sits in a collapsed section at the end, like this one. That section is the
"information floor": identity, why it exists, canonical facts (language, status, required keys), one
concrete example, and pointers to deeper documents.

**Pipeline.**

1. `scripts/readme_evidence.py` (standard library only) emits evidence as JSON, never a verdict:
   first-screen length and new terms, internal references, coined-term candidates, what the collapsed
   (details) blocks hold, figures without adjacent prose, broken links and anchors, internal-history
   lines, raw numeric claims, stock AI phrasing (slop words), Japanese sentence endings, and the
   README's size.
2. `scripts/readme_render.py` (optional; GitHub CLI and Playwright) renders the README through GitHub's
   own Markdown API, styles it with a copy of the github-markdown-css stylesheet (an approximation of
   GitHub's look) at the README column's width, desktop and mobile, light and dark,
   and reports screenshots, heading positions, the fold and overflowing elements.
3. Optional, for rewrites meant to get people to try a project: three simulated first-time visitors
   (AI readers, each given the background of a likely visitor to the repository) read the README cold and report where they stopped, whether they would try it, and
   what pushed them away ([references/visitor-read.md](skills/readme-writer/references/visitor-read.md),
   in Japanese).
4. The `readme-judge` agent (bundled in `agents/`) reads every language version once with the evidence
   and screenshots, answers a fixed checklist with quoted evidence, checks each claim against the
   repository's code, tries to refute its own findings, and returns one named verdict rather than a
   score: Publishable, Fix (with fixes for specific sentences) or Rewrite. A rewrite runs a draft
   judgment, a recheck only when that verdict is Fix, a binding final judgment on the frozen text
   (with a before/after comparison when the old page was rendered), and one more recheck only when the
   final verdict is Fix.
5. The person who asked for the rewrite reads the whole result; the number of issues that read-through
   finds is recorded as the judge's real error rate.

**Example.**

```bash
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence /path/to/README.md
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence --text /path/to/README.md
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_render /path/to/README.md --out /tmp/render --surface repo
```

The evidence command always exits 0 (2 only for a missing or oversized file); the render command exits
2 when the file, `gh` or the browser is missing.

**Where to read more.** In Japanese: [SKILL.md](skills/readme-writer/SKILL.md) (the procedure),
[references/readme-judge-checklist.md](skills/readme-writer/references/readme-judge-checklist.md)
(the judge's checklist), [references/](skills/readme-writer/references/) (diagrams, Japanese register,
About fields, visitor read, tagline eval). In English: [inspiration.md](skills/readme-writer/inspiration.md) (design sources),
[llms.txt](llms.txt) and [llms-full.txt](llms-full.txt). The skill is part of the
[Authorship Strategy](https://github.com/shimo4228/authorship-strategy) line
([DOI 10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)); its code-counts,
LLM-judges split follows ADR-0008 of
[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle)
([DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)).

</details>
