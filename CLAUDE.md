# daily-coworker — AI生産性アシスタント

## プロジェクト概要
個人の生産性を最大化するためのAIアシスタントワークスペース。
スケジュール管理、リサーチ、記事作成、記憶管理を統合する。

## ディレクトリ構成

```
00_context/          # プロフィール、AI記憶
  memories/
    preferences.md   # ユーザーの好み・設定
    decisions.md     # 意思決定ログ
    context-log.md   # セッション間コンテキスト
    case-judgment-framework.md  # 判断基準
01_strategy/         # ビジネス戦略
output/              # AI出力ファイル
  research/          # リサーチ結果
  articles/          # 記事
```

## スキルトリガー

| トリガーワード | コマンド | 動作 |
|--------------|---------|------|
| 「おはよう」 | /daily-schedule | Google Calendar連携で1日のスケジュール作成 |
| 「テックニュース」 | /tech-news | 3ソース並列取得＋横断インサイト抽出 |
| 「調べて」 | /deep-research | 6エージェント体制でリサーチチーム起動 |
| 「記事を書いて」 | /write-article | 5フェーズで記事作成チーム起動 |
| 「覚えておいて」 | /agent-memory | AI記憶の分類・保存 |

## コンテキスト管理
- セッション開始時: `00_context/memories/` の関連ファイルを確認する
- セッション終了時: 重要な決定・発見を記録するか確認する
- コンテキストが圧迫されたら正直に宣言して中断する
