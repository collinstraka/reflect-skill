# reflect-skill

A Claude Code skill for end-of-session capture. Scans a Claude conversation for corrections, insights, milestones, and experiments, then writes structured events to a personal self-learning event store.

## What it does

`/reflect` is a self-learning loop tool. At the end of any Claude session, run it and it will:

1. Scan the full conversation autonomously
2. Identify corrections (where you pushed back or redirected Claude), milestones, insights, and experiments
3. Write structured event files directly — no confirmation needed by default
4. Check for reinforcement: if a new event echoes an existing signal, it updates the original's half-life and links the two

**Core principle:** Claude does the work. You review. You never have to reconstruct what happened from memory — Claude has the full context and uses it.

## Modes

| Invocation | Behavior |
|-----------|----------|
| `/reflect` | Autonomous — scans full session, writes directly |
| `/reflect --quick` | Scopes to most recent exchange only |
| `/reflect --interview` | Walks through 3 questions one at a time |
| `/reflect --correction` | Captures a single correction |
| `/reflect --milestone` | Captures a single milestone |
| `/reflect --insight` | Captures a single insight |

## Event schema

Each event is a markdown file with YAML frontmatter:

```yaml
id: evt-YYYY-MM-DD-NNN
date: YYYY-MM-DD
type: correction|insight|milestone|experiment|reflection|inbox-processed
project: project-slug
title: "Short descriptive title"
signal_type: negative|positive|neutral
confidence: high|medium|low
reinforcement_count: 1
reinforcement_log:
  - date: YYYY-MM-DD
    event_id: evt-YYYY-MM-DD-NNN
    note: "Initial signal"
last_reinforced: YYYY-MM-DD
decay_half_life_days: 30
```

### Decay model

Signals that aren't reinforced fade. Signals that recur get encoded deeply.

`decay_half_life_days = 30 * 2^(reinforcement_count - 1)`

| Reinforcements | Half-life |
|---------------|-----------|
| 1 | 30 days |
| 2 | 60 days |
| 3 | 120 days |
| 4 | 240 days |
| 5 | ~1.3 years |

## Event types

| Type | When | Signal default |
|------|------|---------------|
| `correction` | You pushed back or redirected | negative |
| `insight` | A realization about how you work with Claude | neutral |
| `milestone` | Something shipped or completed | positive |
| `experiment` | New approach or tool tried | neutral |
| `reflection` | End-of-session or periodic reflection | neutral |
| `inbox-processed` | An inbox item ingested | neutral |

## Templates

The `templates/` directory contains frontmatter templates for each event type. The skill reads these when creating events.

## Install

Copy `SKILL.md` to `~/.claude/skills/reflect/SKILL.md`.

Update the paths in `SKILL.md` to point to your event store and templates directory.

## Event store

Events are saved to:
```
events/YYYY/MM/YYYY-MM-DD_<type>_<project>_<slug>.md
```

The event store is separate from this repo — it lives wherever you keep your personal learning system (e.g., Google Drive, Obsidian vault, local folder).
