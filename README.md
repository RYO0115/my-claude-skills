# my-claude-skills

[Claude Code](https://claude.com/claude-code) 用の Skill 集です。各スキルは独立した Git リポジトリを **git submodule** として束ねる形で管理しています。

Skill は Claude Code に「特定タスクの進め方」を教える仕組みで、各ディレクトリ直下の `SKILL.md`(YAML frontmatter の `name` / `description` と本文の手順)が本体です。`description` に書かれたトリガー語や状況にマッチすると、Claude が自動的にそのスキルを読み込みます。

---

## 収録スキル一覧

| スキル | 用途 | 主なトリガー |
|--------|------|-------------|
| [`task-triage`](claude-task-triage-skill/) | 着手前にタスク難易度(軽/中/重)を見積もり、見合った実行モードを先に決めてトークンを削減 | 「難易度を見積もって」「トリアージして」「トークンを抑えて進めて」 |
| [`subagent-delegation`](subagent-delegation/) | 高価なモデル(Opus/Fable)作業中に、単純・大量・機械的な作業を安いモデルの subagent へ委譲してトークン消費を抑える | 「これ subagent にやらせて」「安いモデルに投げて」「トークン節約して委譲」 |
| [`opus5-prompting`](claude-opus5-prompting/) | Claude Opus 5 の冗長化・過剰検証などを打ち消し、トークンを抑えつつ精度を上げるプロンプト・テンプレートと effort 選定 | 「Opus 5 のトークンを節約したい」「出力が長い/検証しすぎる」「effort をどう設定」 |
| [`market-research`](market-research/) | 市場調査を体系実行(ヒアリング→計画→並列サブエージェント調査→統合)。信頼度ラベル付きレポートを `research/` に出力 | 「市場調査して」「市場規模を知りたい」「競合を分析して」/ TAM・PEST・3C・Five Forces |
| [`business-planning`](buisiness-planning/) | 市場調査や事業アイデアを入力に、リーン/フル/GTM/新市場参入の各モードで実行可能な事業計画を `research/` に出力 | 「事業計画を作って」「GTM戦略を立てたい」「海外展開の計画」「リーンキャンバスを作って」 |
| [`python-dev-workflow`](python-dev-workflow/) | Python の共通開発ルール集。uv、core/features のレイヤード構成、テスト必須ルール、ハードコード禁止原則 | Python の新規実装・機能追加・リファクタ・レビュー / uv・pyproject.toml・pytest |
| [`cpp-dev-workflow`](cpp-dev-workflow/) | C++ の設計・実装・テスト・静的解析ワークフロー。CMake / Google Test / clang-tidy 前提 | C++ の新規作成・機能追加・バグ修正・リファクタ |
| [`git-dev-workflow`](git-dev-workflow/) | Git-Flow ベースのブランチ運用・コミット・CI・PR ルール。C++ は clang-tidy+GoogleTest、Python は PyLint+pytest 前提 | ブランチを切る/コミット/PR 作成/リリース/ホットフィックス |

> スキル同士は連携します。例:`task-triage` が重タスクの機械的部分を `subagent-delegation` に接続、`business-planning` は `market-research` の出力を入力に使います。

---

## 他プロジェクトへの移植方法

Claude Code はスキルを次の場所から読み込みます。

- **プロジェクト単位**: 対象リポジトリの `.claude/skills/<skill-name>/SKILL.md`
- **ユーザー単位(全プロジェクト共通)**: `~/.claude/skills/<skill-name>/SKILL.md`

各スキルは 1 ディレクトリ = 1 スキルなので、ディレクトリごとコピーするだけで移植できます。用途に応じて以下から選んでください。

### 方法 A: ディレクトリを丸ごとコピー(最も手軽)

特定のスキルだけを別プロジェクトに入れる場合:

```bash
# 例: python-dev-workflow を対象プロジェクトに導入
mkdir -p /path/to/target-project/.claude/skills
cp -R python-dev-workflow /path/to/target-project/.claude/skills/python-dev-workflow
```

全プロジェクトで使いたいスキルはユーザーディレクトリへ:

```bash
mkdir -p ~/.claude/skills
cp -R market-research ~/.claude/skills/market-research
```

> 注意: submodule のまま `cp -R` すると中身がコピーされます(このリポジトリを一度 clone 済みであれば実体が入っています)。`.git` ファイル/ディレクトリは不要なので、コピー後に `rm -rf .claude/skills/<name>/.git` で削除しておくと安全です。

### 方法 B: この集約リポジトリごと clone(全スキルをまとめて使う)

submodule を含めて取得します:

```bash
git clone --recurse-submodules https://github.com/RYO0115/my-claude-skills.git
# 既に clone 済みなら:
git submodule update --init --recursive
```

取得後、必要なスキルを方法 A の要領で対象プロジェクトの `.claude/skills/` へコピーします。

### 方法 C: 個別スキルを submodule として取り込む(更新を追従したい場合)

各スキルは独立リポジトリなので、対象プロジェクトに submodule として追加すれば `git submodule update --remote` で更新を取り込めます:

```bash
cd /path/to/target-project
git submodule add https://github.com/RYO0115/python-dev-workflow.git .claude/skills/python-dev-workflow
```

各スキルの upstream URL は本リポジトリの [`.gitmodules`](.gitmodules) を参照してください。

### 移植後の確認

1. 対象プロジェクトで Claude Code を起動する。
2. `/` を入力し、スキル名(例 `market-research`)が候補に表示されるか確認する。
3. あるいは `description` のトリガー語(例「市場調査して」)を含む依頼を投げ、スキルが自動起動することを確認する。

---

## ライセンス / 更新

各スキルの詳細・ライセンスは、それぞれの upstream リポジトリ([`.gitmodules`](.gitmodules) 記載)を参照してください。
