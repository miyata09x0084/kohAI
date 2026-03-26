<p align="center">English | <a href="README.ja.md">日本語</a></p>

# Coworker — AI Productivity Assistant

A workspace that maximizes Claude Code productivity.
Integrates a 3-layer CLAUDE.md architecture, sub-agent orchestration, and an AI memory system to streamline daily workflows.

## Architecture

### 3-Layer CLAUDE.md Design

```
~/.claude/CLAUDE.md              # Global config (work style)
coworker/CLAUDE.md               # Project config (structure & triggers)
coworker/.claude/rules/*.md      # Behavioral rules
```

| Layer | Role | Example |
|-------|------|---------|
| Global | Persona & principles across all projects | Conclusion-first, polite, simplicity-first |
| Project | Directory structure & trigger definitions | Skill trigger table |
| Rules | Auto-applied behavioral norms | "Stop when you feel rushed" |

## Directory Structure

```
coworker/
├── CLAUDE.md                        # Project config
├── .claude/
│   ├── rules/
│   │   └── behavioral-norms.md      # Behavioral norms
│   └── commands/
│       ├── daily-schedule.md        # /daily-schedule
│       ├── deep-research.md         # /deep-research
│       ├── write-article.md         # /write-article
│       └── agent-memory.md          # /agent-memory
├── 00_context/
│   └── memories/
│       ├── preferences.md           # User preferences
│       ├── decisions.md             # Decision log
│       ├── context-log.md           # Cross-session context
│       └── case-judgment-framework.md # Judgment criteria
├── 01_strategy/                     # Business strategy
└── output/
    ├── research/                    # Research output
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

## Design Principles

| Principle | Description |
|-----------|-------------|
| Layer separation | 3 layers: global, project, rules |
| Information focus | CLAUDE.md contains only decision-relevant info (mechanical rules go to linters) |
| 1:1 principle | 1 agent = 1 task (prevents scattered output) |
| Review gates | Rejectable gates at each phase |
| Context economy | Delegate research to sub-agents |
| Structured memory | Auto-classify, dedup, conflict detection |
