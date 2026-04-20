<p align="center">English | <a href="README.md">日本語</a></p>

# kohAI — Your AI Kohai (後輩)

A Claude Code workspace where **you are senpai, the AI is your kohai (junior colleague)**.
The name `kohAI` is a double meaning: the Japanese word *kohai* (後輩, "junior") + *AI*. The design intent: **humans stay in the driver's seat, the AI supports**.

Integrates a 3-layer CLAUDE.md architecture, sub-agent orchestration, and an AI memory system to streamline daily workflows.

## Architecture

### 3-Layer CLAUDE.md Design

```
~/.claude/CLAUDE.md              # Global config (work style)
kohAI/CLAUDE.md                  # Project config (structure & triggers)
kohAI/.claude/rules/*.md         # Behavioral rules
```

| Layer | Role | Example |
|-------|------|---------|
| Global | Persona & principles across all projects | Conclusion-first, polite, simplicity-first |
| Project | Directory structure & trigger definitions | Skill trigger table |
| Rules | Auto-applied behavioral norms | "Stop when you feel rushed" |

## Directory Structure

```
kohAI/
├── CLAUDE.md                        # Project config
├── .env                             # External URL config (news sources, etc.)
├── .claude/
│   ├── rules/
│   │   └── behavioral-norms.md      # Behavioral norms
│   ├── config/
│   │   └── news-sources.yaml        # Declarative source defs for /tech-news
│   └── commands/
│       ├── daily-schedule.md        # /daily-schedule
│       ├── tech-news.md             # /tech-news
│       ├── deep-research.md         # /deep-research
│       ├── write-article.md         # /write-article
│       └── agent-memory.md          # /agent-memory
├── 00_context/
│   └── memories/                    # AI memory (personal *.md files are git-ignored)
│       ├── preferences.template.md  # Copy to preferences.md on first use
│       ├── decisions.template.md
│       ├── context-log.template.md
│       └── case-judgment-framework.template.md
├── 01_strategy/                     # Business strategy
└── output/
    ├── research/                    # Research output
    ├── news/                        # Daily tech news digest
    └── articles/                    # Article output
```

## Slash Commands

### `/daily-schedule` — Morning Routine

Trigger: "Good morning", "Today's schedule"

1. Fetch today's events from Google Calendar
2. Load unfinished tasks & preferences from memory
3. Classify tasks by **urgency x importance x cognitive load** (Eisenhower matrix)
4. Generate a 15-min interval schedule (morning = deep work, afternoon = meetings)
5. Register to Google Calendar after approval

### `/tech-news` — Daily Tech News Digest

Trigger: "Tech news", "News"

```
Phase 1 (parallel)  Hacker News / TechCrunch RSS / Reddit r/technology
       ↓
Phase 2             Dedup → Japanese summaries → sort by score → 10-20 items
       ↓
Phase 3             Cross-article analysis (3 insights: tech trends + industry shifts, "What's happening → So what?")
       ↓
Phase 4             Markdown output
```

Output: `output/news/YYYY-MM-DD-tech-news.md`
Config:
- Source declarations: `.claude/config/news-sources.yaml` (add a new source by adding one entry — no command changes needed)
- URL values: `.env` (e.g., `HACKER_NEWS_TOP_STORIES_URL`)
- 3 strategies: `list-then-detail`, `rss`, `json-list` (covers most REST / RSS / JSON APIs)
- 2 transports: `webfetch` (default) / `curl` (bot-block bypass with configurable User-Agent)

### `/deep-research` — 6-Agent Research

Trigger: "Research this", "Look into"

```
Phase 1 (parallel)  Researcher A (latest info)
                     Researcher B (technical background)
                     Researcher C (critical analysis)
       ↓
Phase 2             Synthesizer (integrate & structure)
       ↓
Phase 3             Reviewer (quality check, can reject)
       ↓
Phase 4             Report Writer (final report)
```

Output: `output/research/YYYY-MM-DD-{slug}.md`

### `/write-article` — 5-Phase Article Creation

Trigger: "Write an article", "Write a blog post"

```
Phase 1 (parallel)  Research A (latest info) / Research B (competitor analysis) / Research C (reader needs)
       ↓
Phase 2             Outline design → User approval
       ↓
Phase 3             Editor review (Go / Revise)
       ↓
Phase 4             Writing
       ↓
Phase 5             Final review (typos, fact-check, SEO)
```

Output: `output/articles/YYYY-MM-DD-{slug}.md`

### `/agent-memory` — AI Memory Management

Trigger: "Remember this", "Take a note"

4-step process: auto-classify → dedup check → conflict check → save.

| Category | Storage |
|----------|---------|
| Preferences | `00_context/memories/preferences.md` |
| Decisions | `00_context/memories/decisions.md` |
| Session context | `00_context/memories/context-log.md` |
| Judgment criteria | `00_context/memories/case-judgment-framework.md` |

## Prerequisites

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) installed
- Google Calendar MCP connected

## Setup

On first clone, initialize your local memory files from templates:

```bash
for f in 00_context/memories/*.template.md; do cp "$f" "${f%.template.md}.md"; done
```

Your personal memory (`*.md`) stays local and is git-ignored. Only the templates (`*.template.md`) are tracked.

## Design Principles

| Principle | Description |
|-----------|-------------|
| Layer separation | 3 layers: global, project, rules |
| Information focus | CLAUDE.md contains only decision-relevant info (mechanical rules go to linters) |
| 1:1 principle | 1 agent = 1 task (prevents scattered output) |
| Review gates | Rejectable gates at each phase |
| Context economy | Delegate research to sub-agents |
| Structured memory | Auto-classify, dedup, conflict detection |
