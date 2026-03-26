<p align="center"><a href="README.md">English</a> | 日本語</p>

# Coworker — AI生産性アシスタント

Claude Codeの生産性を最大化するためのワークスペース。
CLAUDE.mdの3層設計、サブエージェント活用、AI記憶システムを統合し、日常業務を効率化する。

## アーキテクチャ

### 3層CLAUDE.md設計

```
~/.claude/CLAUDE.md              # グローバル設定（仕事の進め方）
coworker/CLAUDE.md               # プロジェクト設定（構成・トリガー）
coworker/.claude/rules/*.md      # 行動規範ルール
```

| 層 | 役割 | 例 |
|----|------|----|
| グローバル | 全プロジェクト共通の人格・行動原則 | 結論ファースト、敬語、シンプル第一 |
| プロジェクト | ディレクトリ構成・トリガー定義 | スキルトリガーテーブル |
| ルール | 自動適用される行動規範 | 「焦ったら止まれ」 |

## ディレクトリ構成

```
coworker/
├── CLAUDE.md                        # プロジェクト設定
├── .claude/
│   ├── rules/
│   │   └── behavioral-norms.md      # 行動規範
│   └── commands/
│       ├── daily-schedule.md        # /daily-schedule
│       ├── deep-research.md         # /deep-research
│       ├── write-article.md         # /write-article
│       └── agent-memory.md          # /agent-memory
├── 00_context/
│   └── memories/
│       ├── preferences.md           # ユーザーの好み・設定
│       ├── decisions.md             # 意思決定ログ
│       ├── context-log.md           # セッション間コンテキスト
│       └── case-judgment-framework.md # 判断基準
├── 01_strategy/                     # ビジネス戦略
└── output/
    ├── research/                    # リサーチ出力
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

## 設計原則

| 原則 | 説明 |
|------|------|
| 階層分離 | グローバル・プロジェクト・ルールの3層化 |
| 情報絞込 | CLAUDE.mdは意思決定情報のみ（機械的ルールはLinter等に委譲） |
| 1対1原則 | 1エージェント = 1タスク（出力の散漫化を防ぐ） |
| レビュー必須 | 差し戻し可能なゲートを設置 |
| コンテキスト節約 | 調査はサブエージェントに委任 |
| 記憶の構造化 | 自動分類・重複チェック・矛盾チェック |
