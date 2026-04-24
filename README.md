# daily-coworker

## Overview

A personal AI assistant workspace built on Claude Code. Schedule management, research, article writing, and memory management are integrated as skills you can launch with trigger words.

For example, saying "good morning" pulls up today's schedule, "look it up" fires off a research team, and "write an article" kicks off the drafting flow.

## Architecture

Built on Claude Code, with a clear separation of three elements: **instructions, skills, and memory**.

```
           User input (trigger word)
                   │
                   ▼
    ┌──────────────────────────────┐
    │  Instructions                │  ── how to behave
    │  (CLAUDE.md + rules/)        │
    └──────────────┬───────────────┘
                   │ matching
                   ▼
    ┌──────────────────────────────┐
    │  Skills                      │  ── what to execute
    │  (.claude/commands/)         │
    └───┬──────────────────────┬───┘
        │                      │
        ▼                      ▼
 ┌──────────────┐    ┌──────────────┐
 │    Memory     │    │    Output     │
 │ (memories/)  │    │  (output/)   │
 │              │    │              │
 │ Read/update  │    │ Save results │
 └──────────────┘    └──────────────┘
```

### Directory Layout

```
daily-coworker/
├── CLAUDE.md                          # Project instructions
├── .claude/
│   ├── rules/
│   │   └── behavioral-norms.md        # Behavioral rules (auto-applied)
│   ├── commands/
│   │   ├── daily-schedule.md          # /daily-schedule  — Morning schedule
│   │   ├── tech-news.md               # /tech-news       — Tech news digest
│   │   ├── deep-research.md           # /deep-research   — Multi-agent research
│   │   ├── write-article.md           # /write-article   — Article writing
│   │   └── agent-memory.md            # /agent-memory    — AI memory management
│   └── config/
│       └── news-sources.yaml          # /tech-news source definitions
├── 00_context/memories/               # AI memory (*.md git-ignored; only templates tracked)
│   ├── preferences.template.md
│   ├── decisions.template.md
│   ├── context-log.template.md
│   └── case-judgment-framework.template.md
├── 01_strategy/                       # Business strategy docs
└── output/
    ├── news/                          # /tech-news daily digests
    ├── research/                      # /deep-research reports
    ├── articles/                      # /write-article outputs
    ├── interview/                     # Interview preparation notes
    └── x-posts/                       # X (Twitter) post drafts
```

### Three-Layer Instruction Hierarchy

| Layer   | Location              | Role |
|---------|-----------------------|------|
| Global  | `~/.claude/CLAUDE.md` | Behavioral principles shared across all projects |
| Project | `daily-coworker/CLAUDE.md` | Project-specific configuration and trigger definitions |
| Rules   | `.claude/rules/`      | Context-activated behavior norms |

Separating the layers localizes the impact of any change.

### Skill Invocation Flow

Trigger word (e.g., "look it up") → the project `CLAUDE.md` matches the corresponding skill → `.claude/commands/{skill}.md` runs. Complex skills parallelize via subagents to keep main-thread context consumption low.

### Memory Handling

`00_context/memories/` accumulates preferences, decisions, and context logs as Markdown. These are automatically referenced at the start of each new session. Personal memory (`*.md`) is git-ignored; only templates (`*.template.md`) are tracked.

## Steps

### Prerequisites

| Tool | Purpose |
|------|---------|
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Runtime |
| Google Calendar MCP | Calendar integration for `/daily-schedule` |
| git | Cloning the repository |

### 1. Install Claude Code

Follow the [official documentation](https://docs.anthropic.com/en/docs/claude-code).

### 2. Clone the repository

```bash
git clone https://github.com/miyata09x0084/daily-coworker.git
cd daily-coworker
```

### 3. Connect Google Calendar MCP

Required only for `/daily-schedule`. Connect Google Calendar through Claude Code's MCP settings.

### 4. Create `.env`

Set news source URLs for `/tech-news`.

```bash
cat > .env <<'EOF'
HACKER_NEWS_TOP_STORIES_URL=https://hacker-news.firebaseio.com/v0/topstories.json
HACKER_NEWS_ITEM_URL_TEMPLATE=https://hacker-news.firebaseio.com/v0/item/{id}.json
TECHCRUNCH_RSS_URL=https://techcrunch.com/feed/
REDDIT_TECH_URL=https://www.reddit.com/r/technology/hot.json
EOF
```

### 5. Initialize memory files

Copy each memory file from its template.

```bash
for f in 00_context/memories/*.template.md; do
  cp "$f" "${f%.template.md}.md"
done
```

Personal memory (`*.md`) is git-ignored; only templates (`*.template.md`) are tracked.

### Usage

Say a trigger word in Claude Code to launch the corresponding skill. The triggers are defined in Japanese because the project targets Japanese-speaking users.

| Trigger (Japanese) | Meaning | Skill |
|--------------------|---------|-------|
| 「おはよう」 / 「今日の予定」   | "good morning" / "today's schedule" | `/daily-schedule` |
| 「テックニュース」 / 「ニュース」 | "tech news" / "news"                | `/tech-news` |
| 「調べて」 / 「リサーチして」   | "look it up" / "research"           | `/deep-research` |
| 「記事を書いて」 / 「ブログ書いて」 | "write an article" / "write a blog" | `/write-article` |
| 「覚えておいて」 / 「メモして」  | "remember this" / "note this"       | `/agent-memory` |

Slash commands (e.g., `/tech-news`) can also be invoked directly.
