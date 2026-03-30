# X投稿エージェント実装計画

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Xのアルゴリズムに最適化された投稿案を、マルチエージェント体制で生成するスキルを作成する

**Architecture:** `.claude/commands/x-post.md` にスキル定義を作成。既存の `deep-research.md` と同じマルチエージェントパイプライン構成（Phase 1: 並列ネタ探索 → Phase 2: 評価・選定 → Phase 3: 投稿文生成）。CLAUDE.md のトリガー表に追記。

**Tech Stack:** Claude Code スキル（マークダウン）、WebSearch、WebFetch、Agent tool

---

## Task 1: x-post スキルファイル作成

**Files:**
- Create: `.claude/commands/x-post.md`

- [ ] **Step 1: スキルファイルを作成する**

以下の内容で `.claude/commands/x-post.md` を作成する:

```markdown
---
description: X投稿案を生成する。「X投稿」「ポスト作成」「ツイート」等で起動。
---

# X投稿エージェント

「$ARGUMENTS」をもとに、Xのアルゴリズムに最適化された投稿案を3つ生成する。

## 入力の判定

$ARGUMENTS の内容に応じて動作を分岐する:

| パターン | 判定基準 | 動作 |
|---------|---------|------|
| テーマなし | $ARGUMENTS が空 | トレンドからネタ探索 |
| テーマ指定 | テキストのみ（URLなし） | テーマに沿ったネタ探索 |
| インプット付き | URLまたは長文メモが含まれる | Agent C を追加起動 |

## Phase 1: ネタ探索

### Step 1-1: 並列起動（Agent A + Agent C）

以下のエージェントを **並列で** 起動する。Agent C はインプットがある場合のみ起動する。

#### Agent A: トレンドリサーチャー

WebSearch を使い、直近24-48時間のテック・AI関連トレンドを収集する。

**検索クエリ（すべて実行する）:**
- 日本語: 「AI 最新ニュース 今週」「エンジニアリング トレンド」
- 英語: "AI news today"、"programming trending"

**追加ソース（WebFetch）:**
- https://hacker-news.firebaseio.com/v0/topstories.json の上位5件の詳細を取得
- https://www.reddit.com/r/programming/top.json?t=day&limit=5 を取得

テーマ指定がある場合は、上記に加えてテーマに関連する検索クエリも実行する。

**出力フォーマット:**
```
## トレンドネタ候補
1. [ネタ] — [概要1-2行] (出典: URL)
2. ...
（5-10件）
```

#### Agent C: ユーザーインプット分析（条件付き起動）

ユーザーが渡したURL・メモ・記事を分析し、X投稿に適した切り口を抽出する。

- URLが含まれる場合: WebFetch で記事を取得し内容を読む
- メモ・テキストの場合: そのまま分析する

**分析の観点:**
- この内容の中で、他の人が「へえ」と思うポイントはどこか
- 議論を呼びそうな主張や発見はあるか
- 自分の実体験として語れる要素はあるか

**出力フォーマット:**
```
## インプットからのネタ候補
1. [切り口] — [なぜこの切り口が面白いか]
2. ...
（2-5件）
```

### Step 1-2: Agent Aの結果を受けて並列起動（Agent B + Agent D）

Agent A（+ Agent C）の結果を受け取り、以下の2エージェントを **並列で** 起動する。

#### Agent B: 深掘りリサーチャー

Agent A・Cが収集したネタ候補それぞれについて、WebSearch と WebFetch で深掘りする。

**調査内容:**
- なぜ今これが話題なのか（時系列の文脈）
- 技術的に何が新しい・重要なのか
- 実務への影響（エンジニアの日常にどう関係するか）

**出力フォーマット:**
```
## 深掘り結果
### ネタ1: [タイトル]
- 背景: ...
- 技術的ポイント: ...
- 実務への影響: ...
- 元ネタのURL: ...

### ネタ2: ...
```

#### Agent D: 批判的分析エージェント

Agent A・Cが収集したネタ候補それぞれについて、WebSearch で批判的視点を調査する。

**調査内容:**
- この情報は本当に正しいか？（ソースの信頼性、データの裏付け）
- 誇張されていないか？（実態との乖離）
- 反対意見・別の解釈はないか？
- 見落とされているリスクや副作用はないか？

**出力フォーマット:**
```
## 批判的分析
### ネタ1: [タイトル]
- 信頼性: ★★★★☆（理由: ...）
- 反対意見: ...
- 見落とされている点: ...
- 「実は...」の切り口: ...

### ネタ2: ...
```

## Phase 2: ネタ評価・選定

Agent B・Dの結果を統合し、全ネタを以下の5基準で採点する（各5段階）。

| 基準 | 観点 |
|------|------|
| 会話誘発力 | 問いかけ・議論になりやすいか。リプライ(+75)を狙えるか |
| 独自性 | 他のアカウントと差別化できる切り口があるか |
| テーマ適合度 | テック/AI・プロダクティビティの軸に合っているか |
| 批判的深度 | Agent Dの分析で面白い視点が出たか。「実は...」の切り口があるか |
| 鮮度 | 今投稿する価値があるか（話題のタイミング） |

**選定ルール:**
- 合計スコア上位3件を選定する
- 各ネタに「選定理由」と「推奨する投稿の切り口」を付与する

**出力フォーマット:**
```
## 選定結果
### 1位: [ネタタイトル]（合計: XX/25）
- 会話誘発力: X/5 | 独自性: X/5 | テーマ適合: X/5 | 批判的深度: X/5 | 鮮度: X/5
- 選定理由: ...
- 推奨する切り口: ...

### 2位: ...
### 3位: ...
```

## Phase 3: 投稿文生成

選定された3ネタそれぞれについて、以下のルールで投稿案を生成する。

### 生成ルール（Xアルゴリズム最適化）

- **構造**: 結論 → 理由 → 問いかけ（末尾に問いを置いて会話を誘発する）
- **トーン**: 問いかけ > 実務共有 > 解説 の優先度で、中間〜話し言葉寄り
- **文字数**: 単発は280字以内。深いテーマはスレッド（4-8投稿）も提案する
- **ハッシュタグ**: 0-2個まで（3個以上は-40%ペナルティ）
- **外部リンク**: 本文に含めない（リプライに配置する旨を戦略メモに記載）
- **批判的視点**: Agent Dの分析を活用し、「通説 vs 実態」の対比を織り込む
- **比喩**: 抽象概念は身近なイメージに落とし込む

### 出力フォーマット

以下のフォーマットで3案を生成し、ターミナルに表示する:

```markdown
# X投稿案 — {今日の日付}

---

## 投稿案 1: {ネタタイトル}

### 投稿文
{そのままコピペ可能な投稿文}

### 戦略メモ
- 狙うシグナル: {リプライ、ブックマーク、引用RT等}
- 推奨投稿時間: {曜日・時間帯}
- 投稿後アクション: {リプライ対応方針、リンク補足の有無}
- この形式が有効な理由: {アルゴリズム上の根拠}

---

## 投稿案 2: {ネタタイトル}
...

---

## 投稿案 3: {ネタタイトル}
...
```

## 保存

結果を `output/x-posts/YYYY-MM-DD-{slug}.md` に保存する。
日付フォーマットは `YYYY-MM-DD`、slug はネタ全体を表す英語の短い識別子とする。
```

- [ ] **Step 2: 動作確認のためファイルの構文を確認する**

Run: `cat .claude/commands/x-post.md | head -3`
Expected: フロントマターが正しく出力される
```
---
description: X投稿案を生成する。「X投稿」「ポスト作成」「ツイート」等で起動。
---
```

- [ ] **Step 3: コミット**

```bash
git add .claude/commands/x-post.md
git commit -m "feat: add x-post skill for algorithm-optimized X posting"
```

---

## Task 2: CLAUDE.md にトリガー追記

**Files:**
- Modify: `CLAUDE.md`

- [ ] **Step 1: CLAUDE.md のスキルトリガー表に x-post を追加する**

`CLAUDE.md` の `## スキルトリガー` セクションのテーブルに以下の行を追加する:

```markdown
| 「X投稿」 | /x-post | マルチエージェントでX投稿案を生成 |
```

追加後のテーブル全体:

```markdown
| トリガーワード | コマンド | 動作 |
|--------------|---------|------|
| 「おはよう」 | /daily-schedule | Google Calendar連携で1日のスケジュール作成 |
| 「調べて」 | /deep-research | 6エージェント体制でリサーチチーム起動 |
| 「記事を書いて」 | /write-article | 5フェーズで記事作成チーム起動 |
| 「覚えておいて」 | /agent-memory | AI記憶の分類・保存 |
| 「X投稿」 | /x-post | マルチエージェントでX投稿案を生成 |
```

- [ ] **Step 2: コミット**

```bash
git add CLAUDE.md
git commit -m "docs: add x-post trigger to skill trigger table"
```

---

## Task 3: output ディレクトリ準備

**Files:**
- Create: `output/x-posts/.gitkeep`

- [ ] **Step 1: output/x-posts ディレクトリを作成する**

```bash
mkdir -p output/x-posts
touch output/x-posts/.gitkeep
```

- [ ] **Step 2: .gitignore を確認し、output/x-posts が除外対象か確認する**

`output/` が `.gitignore` に含まれている場合、スキルの出力ファイルはgit管理外になる（これは意図通り — 既存の `output/research/` と同じ扱い）。

- [ ] **Step 3: コミット**

```bash
git add -f output/x-posts/.gitkeep
git commit -m "chore: add output/x-posts directory for x-post skill output"
```

---

## Task 4: 動作確認

- [ ] **Step 1: スキルの起動テスト**

Claude Code で `/x-post` を実行し、以下を確認する:
- スキルが認識されて起動するか
- Phase 1 のエージェントが並列で起動するか
- Phase 2 の評価が正しく動作するか
- Phase 3 の投稿文が生成されるか
- 出力ファイルが `output/x-posts/` に保存されるか

- [ ] **Step 2: テーマ指定での起動テスト**

`/x-post RAGの最新動向` のようにテーマを指定して実行し、テーマに沿ったネタ探索が行われることを確認する。

- [ ] **Step 3: 問題があれば修正してコミット**
