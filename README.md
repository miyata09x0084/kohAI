<p align="center"><a href="README.en.md">English</a> | 日本語</p>

# kohAI — あなたの後輩AI

**あなたが先輩、AIは後輩**というスタンスのClaude Codeワークスペース。
名前 `kohAI` は「後輩」+ `AI` の語呂合わせ。設計思想: **人間が主体、AIは支える側**。

CLAUDE.mdの3層設計、サブエージェント活用、AI記憶システムを統合し、日常業務を効率化する。

## アーキテクチャ

### 3層CLAUDE.md設計

```
~/.claude/CLAUDE.md              # グローバル設定（仕事の進め方）
kohAI/CLAUDE.md                  # プロジェクト設定（構成・トリガー）
kohAI/.claude/rules/*.md         # 行動規範ルール
```

| 層 | 役割 | 例 |
|----|------|----|
| グローバル | 全プロジェクト共通の人格・行動原則 | 結論ファースト、敬語、シンプル第一 |
| プロジェクト | ディレクトリ構成・トリガー定義 | スキルトリガーテーブル |
| ルール | 自動適用される行動規範 | 「焦ったら止まれ」 |

## ディレクトリ構成

```
kohAI/
├── CLAUDE.md                        # プロジェクト設定
├── .env                             # ニュースソース等の外部URL設定
├── .claude/
│   ├── rules/
│   │   └── behavioral-norms.md      # 行動規範
│   ├── config/
│   │   └── news-sources.yaml        # /tech-news のソース宣言的定義
│   └── commands/
│       ├── daily-schedule.md        # /daily-schedule
│       ├── tech-news.md             # /tech-news
│       ├── deep-research.md         # /deep-research
│       ├── write-article.md         # /write-article
│       └── agent-memory.md          # /agent-memory
├── 00_context/
│   └── memories/                    # AI記憶（個人の *.md は git管理外）
│       ├── preferences.template.md  # 初回clone時に preferences.md へコピー
│       ├── decisions.template.md
│       ├── context-log.template.md
│       └── case-judgment-framework.template.md
├── 01_strategy/                     # ビジネス戦略
└── output/
    ├── research/                    # リサーチ出力
    ├── news/                        # テックニュース日次ダイジェスト
    └── articles/                    # 記事出力
```

## スラッシュコマンド

### `/daily-schedule` — 朝のルーティン

トリガー: 「おはよう」「今日の予定」

1. Google Calendarから本日の予定を取得
2. メモリから未完了タスク・好みを読み込み
3. タスクを**緊急度×重要度×認知負荷**で分類（アイゼンハワーマトリクス）
4. 15分刻みのスケジュールを生成（午前=集中作業、午後=ミーティング）
5. 承認後、Google Calendarに登録

### `/tech-news` — テックニュース斜め読み

トリガー: 「テックニュース」「ニュース」

```
Phase 1（並列）  Hacker News / TechCrunch RSS / Reddit r/technology
       ↓
Phase 2          重複除去 → 日本語要約 → スコア順ソート → 10〜20件に整形
       ↓
Phase 3          横断分析（技術トレンド + 業界構造で3インサイト、「何が起きているか → So what?」形式）
       ↓
Phase 4          Markdown出力
```

出力先: `output/news/YYYY-MM-DD-tech-news.md`
設定:
- ニュースソース定義は `.claude/config/news-sources.yaml`（宣言的・新規追加はここに entry を足すだけ）
- URL 値は `.env` で管理（`HACKER_NEWS_TOP_STORIES_URL` 他）
- 戦略は 3 種類: `list-then-detail` / `rss` / `json-list`（REST・RSS・JSON API の大半を網羅）
- トランスポートは 2 種類: `webfetch`（デフォルト）/ `curl`（bot ブロック回避用、User-Agent 指定可）

### `/deep-research` — 6エージェントリサーチ

トリガー: 「調べて」「リサーチして」

```
Phase 1（並列）  リサーチャーA（最新情報）
                  リサーチャーB（技術背景）
                  リサーチャーC（批判的分析）
       ↓
Phase 2          シンセサイザー（統合・構造化）
       ↓
Phase 3          レビュアー（品質チェック・差し戻し可）
       ↓
Phase 4          レポートライター（最終レポート作成）
```

出力先: `output/research/YYYY-MM-DD-{slug}.md`

### `/write-article` — 5フェーズ記事作成

トリガー: 「記事を書いて」「ブログ書いて」

```
Phase 1（並列）  調査A（最新情報）/ 調査B（競合分析）/ 調査C（読者ニーズ）
       ↓
Phase 2          構成設計 → ユーザー承認
       ↓
Phase 3          編集長レビュー（Go / 要修正）
       ↓
Phase 4          執筆
       ↓
Phase 5          最終レビュー（誤字脱字・事実確認・SEO）
```

出力先: `output/articles/YYYY-MM-DD-{slug}.md`

### `/agent-memory` — AI記憶管理

トリガー: 「覚えておいて」「メモして」

自動分類 → 重複チェック → 矛盾チェック → 保存の4ステップで記憶を管理。

| カテゴリ | 保存先 |
|---------|--------|
| 好み・設定 | `00_context/memories/preferences.md` |
| 意思決定 | `00_context/memories/decisions.md` |
| セッションコンテキスト | `00_context/memories/context-log.md` |
| 判断基準 | `00_context/memories/case-judgment-framework.md` |

## 前提条件

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) がインストール済み
- Google Calendar MCP が接続済み

## セットアップ

初回 clone 時、テンプレから記憶ファイルを初期化してください:

```bash
for f in 00_context/memories/*.template.md; do cp "$f" "${f%.template.md}.md"; done
```

個人の記憶（`*.md`）はローカル専用・git管理外です。テンプレ（`*.template.md`）のみが追跡対象。

## 設計原則

| 原則 | 説明 |
|------|------|
| 階層分離 | グローバル・プロジェクト・ルールの3層化 |
| 情報絞込 | CLAUDE.mdは意思決定情報のみ（機械的ルールはLinter等に委譲） |
| 1対1原則 | 1エージェント = 1タスク（出力の散漫化を防ぐ） |
| レビュー必須 | 差し戻し可能なゲートを設置 |
| コンテキスト節約 | 調査はサブエージェントに委任 |
| 記憶の構造化 | 自動分類・重複チェック・矛盾チェック |
