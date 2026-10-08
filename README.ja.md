# readme-writer

[English](README.md) | **日本語**

[![Ask DeepWiki](https://deepwiki.com/badge.svg)](https://deepwiki.com/shimo4228/readme-writer)

readme-writer は、README を書き直したり見直したりして、初めて来た人に何のプロジェクトで自分向けかが分かるページにする Claude Code の skill です。執筆の文脈を持たない別の判定 agent が、ページの記述をその repo のコードと照らし、点数ではなく直すべき文を返します。最後に結果を読んで決めるのはあなたです。

## 何を拾うか

2026-10-07 に、毎朝リサーチのノートを書くコマンドラインのツール [jev-research-pipeline](https://github.com/shimo4228/jev-research-pipeline) の README を、332 行（[改稿前](https://github.com/shimo4228/jev-research-pipeline/blob/6a53b9f/README.md)、英語）から約 190 行（[改稿後](https://github.com/shimo4228/jev-research-pipeline/blob/556e694/README.md)、英語）に書き直しました。ページを縮めると事実が壊れます。草稿では 3 つの事実が崩れました。縮める途中で入った誤った記述、改稿前のページから持ち越した誤った記述、縮める途中で落ちた注記です。判定 agent はコードと改稿前のページを照らし、公開の前に 3 つとも止めました。

- 草稿は「毎朝すべてのテーマのノートを書く」と言っていました。コードは輪番で 1 朝 3 テーマずつ回します（`store/rotation.py`）。改稿前のページはそう書いていて、縮める途中で落ちていました。
- 改稿前のページも、それを引き継いだ草稿も、「各コマンドの前に `uvx` を付ければインストールせずに試せる」と言っていました。`jrp schedule install` だけは、インストール済みでないと動きません（`cli.py`）。
- 草稿は、改稿前のページにあった注記を落としていました。見本のノートを書いたのは Claude Opus で、既定の書き手の GPT-6 Luna は英語と中国語ではまだ測っていない、という注記です。判定 agent はそれを戻すよう求めました。

1 つ目についての判定 agent の報告を縮めたものです。

> **草稿の L17–20 がコードと矛盾。** 「every morning jrp checks … and writes one Markdown note per topic」に対し、`src/jev_research_pipeline/store/rotation.py:24`（`per_tick = 3`）。直し方: 毎朝、固定の輪番で次の 3 テーマを回すと書く。

判定の前には、訪問者の背景を持たせた 3 体の AI の読み手が草稿を毎回読み、どこで読むのをやめたかを答えました。説明の無い有料の API キーと、出力の実物が無いことの 2 つで、どちらも直しました。

## 導入

```bash
git clone https://github.com/shimo4228/readme-writer && cd readme-writer
./install.sh
```

`install.sh` は、skill を `~/.claude/skills/readme-writer` に、判定 agent を `~/.claude/agents/readme-judge.md` に写し、`uv sync` で skill の Python の依存を入れます。すでにある版は `*.bak-<日時>` として残し、`--dry-run` で何をするかだけを見られます。Python 3.11 以上と [uv](https://docs.astral.sh/uv/) が要ります。

GitHub での見え方（パソコンとスマホの幅、明るい表示と暗い表示）も判定 agent に見せるときは、GitHub CLI にログインし（`gh auth login`）、`~/.claude/skills/readme-writer` でブラウザを 1 回入れます（`uv run playwright install chromium`）。これが無ければ、判定は文面だけで行います。[SkillsMP](https://skillsmp.com) のマーケットプレイスから入れる場合（`/skills add shimo4228/readme-writer`）は skill だけが入るので、`agents/readme-judge.md` を `~/.claude/agents/` に自分で写してください。

## 使い方

Claude Code にふつうの言葉で頼めば、skill がモードを選びます。

| 頼み方 | 起きること |
|---|---|
| 「この README を書き直して」 | 全面改稿: 先に節の構成をあなたと決め、書いて、判定し、直し、草稿と描画を通読用に渡す |
| 「この変更に README を追従させて」 | 変更が触れた節だけを書き直し、判定を 1 回 |
| 「この README を見て」 | 判定だけ。書き換えない |
| 「GitHub の About が食い違っている」 | description・topics・homepage の変更案 |

英語と日本語の README に対応し、言語版どうしを揃えます。

## 費用と限界

- **トークン。** 1 回の改稿で判定を 2〜4 回回します。jev-research-pipeline の改稿（約 190 行の英語の README とその日本語版を、中くらいの Python のコードと照合）では、1 回の判定が README・コード・描画を読むのに約 20〜30 万トークンでした。
- **最後の関門はあなたです。** 判定 agent も見落とすので、改稿は必ずあなたの通読で終わります。著者が自分の repo で行った直近 4 回の改稿では、その通読で直すところは見つかりませんでした（[記録](skills/readme-writer/evals/read-through-log.md)）。
- **証拠をこしらえません。** 読者に要るのに repo に無い事実（ベンチマーク、無料枠など）は、言い回しで埋めずにあなたに報告します。

## 著者のほかの仕事

- [llms-txt-writer](https://github.com/shimo4228/llms-txt-writer): AI だけが読むページ（`llms.txt`、FAQ、用語集）を書く対の skill。
- [claude-harness](https://github.com/shimo4228/claude-harness): この skill の出どころの Claude Code の設定一式。判定 agent と、その裏の設計の記録があります。
- [Authorship Strategy](https://github.com/shimo4228/authorship-strategy): README が人と AI の両方に必ず読まれるページである理由と、読者が LLM を通してアイデアに出会う時代に著者の名前を残す方法。
- そのほか: [github.com/shimo4228](https://github.com/shimo4228)。

## ライセンス

MIT

<details>
<summary>ツールと AI アシスタント向けの資料</summary>

**これは何か。** readme-writer は Claude Code の Agent Skill です（Python、MIT、版は 0.2.0 と [CHANGELOG.md](CHANGELOG.md)（英語）にある未リリースの変更）。repo の人間向けの README と、それを要約する GitHub の About を書き、書き直し、見直し、揃えます。AI だけが読むページを書く llms-txt-writer と対になります。

**要るもの。** Claude Code、Python 3.11 以上、uv、`agents/` に同梱した `readme-judge` agent（`install.sh` が skill と agent を一緒に入れます）。任意で、ログイン済みの GitHub CLI と Playwright の Chromium（描画証拠）。Claude Code のプラン以外に有料の API キーは要りません。

**なぜあるか。** README は継ぎ足しで育ちます。リリースのたびに箇条・設計記録の番号・姉妹 repo・造語が増え、初めて来た人には何のプロジェクトか分からなくなります。この skill はそれを戻すためにあり、規則は 1 つです。仕組み・インストール・機能の説明より前に、読者が「この repo は何のためにあるか」を言えること。数えるのはコード、判定するのは LLM、決めるのは人です。

**README の置き分け。** 人は README を上から 1 回読み、分からなくなったところで離れます。URL を取得する AI アシスタントは、畳んだ節もページの末尾も最後まで読みます。そこで、見える本文は人のために使い（何のプロジェクトか、そして使い始めるまでに読者を止めるもの）、LLM がほかのファイルを開かずにプロジェクトを復元するのに要る情報は、この節のように末尾の畳んだ節に置きます。この節を「情報フロア」と呼び、中身は、何であるか、なぜあるか、基本の事実（言語、状態、要る鍵）、具体例 1 つ、深い資料への案内です。

**流れ。**

1. `scripts/readme_evidence.py`（標準ライブラリだけ）が、判定ではなく証拠を JSON で出します。第一画面の長さと新語、内部への参照、造語の候補、`<details>` の中身、前後に文の無い図、切れたリンクとページ内リンク切れ、内部史の行、生の数値、slop 語、日本語の文末、README の字数です。
2. `scripts/readme_render.py`（任意。GitHub CLI と Playwright）が、GitHub 自身の Markdown API で描き、github-markdown-css の写し（GitHub の見た目の近似）を当てて README の列幅に収め、パソコンとスマホの幅、明るい表示と暗い表示の描画、見出しの位置、最初の 1 画面の境目、はみ出す要素を出します。
3. 任意で、人に試してもらうことが目的の改稿では、repo の訪問者の背景を持たせた 3 体の AI の訪問者役が README を読み、どこでやめたか、試すか、何が止めたかを答えます（[references/visitor-read.md](skills/readme-writer/references/visitor-read.md)）。
4. `readme-judge` agent（`agents/` に同梱）が、全言語版を証拠と描画と一緒に 1 回読み、固定のチェックリストに引用付きで答え、記述を repo のコードと照らし、自分の指摘を反証にかけ、名前のついた判定を 1 つ返します。Publishable、Fix（文単位の直し）、Rewrite のどれかです。改稿では、draft、recheck、凍結した本文への拘束力のある最終判定（改稿前との前後比較つき）、最大もう 1 回の recheck の順に回します。
5. 改稿を頼んだ人が結果を通読します。その通読で見つかった指摘の数を、判定 agent の本当の誤り率として記録します。

**例。**

```bash
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence /path/to/README.md
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_evidence --text /path/to/README.md
uv run --directory ~/.claude/skills/readme-writer python -m scripts.readme_render /path/to/README.md --out /tmp/render --surface repo
```

証拠のコマンドは常に 0 で終わります（ファイルが無いか大きすぎるときだけ 2）。描画のコマンドは、ファイル・`gh`・ブラウザのどれかが無いと 2 で終わります。

**さらに読むもの。** [SKILL.md](skills/readme-writer/SKILL.md)（手順）、[references/readme-judge-checklist.md](skills/readme-writer/references/readme-judge-checklist.md)（判定のチェックリスト）、[references/](skills/readme-writer/references/)（図、日本語の文体、About、訪問者役の読み）、[inspiration.md](skills/readme-writer/inspiration.md)（設計の出典、英語）、[llms.txt](llms.txt) と [llms-full.txt](llms-full.txt)（英語）。この skill は [Authorship Strategy](https://github.com/shimo4228/authorship-strategy) の系列の一部です（[DOI 10.5281/zenodo.20263316](https://doi.org/10.5281/zenodo.20263316)）。コードが数え LLM が判定する分担は、[Agent Knowledge Cycle](https://github.com/shimo4228/agent-knowledge-cycle) の ADR-0008 に従います（[DOI 10.5281/zenodo.19200726](https://doi.org/10.5281/zenodo.19200726)）。

</details>
