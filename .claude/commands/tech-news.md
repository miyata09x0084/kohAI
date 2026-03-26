---
description: テックニュースを斜め読みする。「テックニュース」「ニュース」等で起動。
---

# テックニュース斜め読み

## 手順

### 1. ニュース取得（3エージェント並列）

以下の3つのエージェントを **並列で** 起動する。各エージェントは WebFetch でデータを取得し、構造化された結果を返す。

**エージェントA: Hacker News**

```
WebFetch で https://hacker-news.firebaseio.com/v0/topstories.json を取得する。
返ってきたIDリストの先頭10件について、それぞれ https://hacker-news.firebaseio.com/v0/item/{id}.json を取得する。
各記事から以下を抽出して返す:
- title: 記事タイトル
- url: 記事URL
- score: スコア
- source: "HN"
```

**エージェントB: TechCrunch RSS**

```
WebFetch で https://techcrunch.com/feed/ を取得する。
RSSフィードから最新10件を抽出し、各記事から以下を返す:
- title: 記事タイトル（<title>タグ）
- url: 記事URL（<link>タグ）
- score: null
- source: "TC"
```

**エージェントC: Reddit r/technology**

```
WebFetch で https://www.reddit.com/r/technology/top.json?t=day&limit=10 を取得する。
レスポンスの data.children から各記事の以下を抽出して返す:
- title: 記事タイトル（data.title）
- url: 記事URL（data.url）
- score: スコア（data.score）
- source: "Reddit"
```

### 2. 統合・要約・分析

3エージェントの結果を受け取り、以下の処理を行う:

1. **重複除去**: 同じURLの記事、またはタイトルが非常に似ている記事を除外する
2. **日本語要約**: 各記事のタイトルから内容を推測し、1-2行の日本語要約を作成する
3. **ソート**: スコアがある記事はスコア順、ないものは取得順で並べる
4. **件数調整**: 最終的に10-20件のリストにまとめる
5. **横断分析**: ニュース全体を俯瞰し、以下の2軸で3つのインサイトを抽出する:
   - **技術トレンド**: 何が伸びているか、何を学ぶ/採用すべきか、何が廃れつつあるか
   - **業界構造の変化**: 規制・M&A・プラットフォームシフトなど、ビジネスの力学がどう動いているか

   各インサイトは以下の構造で書く:
   - **何が起きているか**（複数記事を横断してパターンを見出す。1記事だけの話はインサイトではない）
   - **So what?**（エンジニア/技術選定の観点で、具体的に何を意味するか）

### 3. 出力

以下のフォーマットでターミナルに表示する:

```markdown
# Tech News Digest — {今日の日付}

1. **[記事タイトル](URL)** — 日本語要約1-2行 `[HN 🔥{score}]`
2. **[記事タイトル](URL)** — 日本語要約1-2行 `[TC]`
3. **[記事タイトル](URL)** — 日本語要約1-2行 `[Reddit ⬆{score}]`
...

---

## Insights

1. **{インサイトのタイトル}**
   {何が起きているか — 関連する記事番号を括弧で参照}
   → {So what? — 技術選定・キャリア・ビジネス判断への示唆}

2. **{インサイトのタイトル}**
   {何が起きているか}
   → {So what?}

3. **{インサイトのタイトル}**
   {何が起きているか}
   → {So what?}

---
Sources: Hacker News, TechCrunch, Reddit r/technology
```

**インサイトのルール:**
- 必ず複数記事を横断して書く（1記事だけのものはインサイトではなくただの要約）
- 「〜が話題です」で終わらせず、**判断・行動につながる示唆**を書く
- 3つに絞る。数を増やすより質を上げる

**ソースタグのルール:**
- Hacker News: `[HN 🔥{score}]`
- TechCrunch: `[TC]`
- Reddit: `[Reddit ⬆{score}]`

### 4. 保存

結果を `output/news/{今日の日付}-tech-news.md` に保存する。
日付フォーマットは `YYYY-MM-DD` とする。
