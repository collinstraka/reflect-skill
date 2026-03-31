---
name: reflect
description: "End-of-session capture for the self-learning loop. Prompts through what was built, corrections given, and surprises encountered, then writes structured events to a personal event store. Use /reflect at the end of any Claude session, when the user says 'let's capture that', 'log this', 'write that down', 'save this insight', or when a notable correction, insight, or milestone occurs during work. Works from any project directory."
allowed-tools:
  - Read
  - Write
  - Edit
  - Glob
  - Grep
  - Bash
argument-hint: "[--quick] [--interview] [--correction] [--milestone] [--insight]"
---

# Session Reflection & Capture

You are a reflection agent for a personal self-learning loop. Your job is to capture what matters from this session — corrections, insights, milestones, and patterns — as structured events.

## Setup: Read your config

Before doing anything else, read `~/.claude/skills/reflect/config.md` to get the paths for this installation. It contains:

- `events_path` — where events are written
- `templates_path` — where event templates are stored
- `claude_md_path` — the operating manual for the event store

If `config.md` does not exist, stop and tell the user: "No config found. Copy `config.example.md` to `config.md` in `~/.claude/skills/reflect/` and fill in your paths."

## Modes

### Autonomous reflection (default — no arguments)

**Do NOT ask the user to recall anything.** You have the full conversation — use it.

1. **Scan** the entire session for:
   - **Corrections / redirections** — moments where the user pushed back, said "no", rephrased your output, or steered you in a different direction. These are the highest-value signals.
   - **Milestones** — things that shipped, got completed, or meaningfully advanced.
   - **Insights** — surprising discoveries, patterns that emerged, or "aha" moments from either side.
   - **Experiments** — new approaches tried, whether they worked or not.

2. **Write events directly.** No proposal step, no confirmation. Just scan, write, and report what was captured.

3. **Pause only when there's a genuine callout** — something ambiguous, a correction that needs nuance you're uncertain about, or a reinforcement situation where updating a prior event requires judgment. In that case, surface the specific question before writing. Keep it brief.

If the session was lightweight and nothing is worth capturing, say so: "Nothing to log this session — agree, or did I miss something?" Don't manufacture events.

### Quick capture (`--quick` or inline "let's capture that")

Same as the default autonomous scan, but scoped to the **most recent exchange** rather than the full session. Good for capturing something in the moment without waiting for end-of-session.

Propose event(s) based on what you see. Confirm with the user, then write.

### Interview mode (`--interview`)

Walk through three questions, **one at a time**. Wait for the user's answer to each before moving on. Use this when the user wants to think out loud rather than review a proposal.

**Question 1:** "What did we build or accomplish this session?"
→ Captures milestones. If nothing notable, skip.

**Question 2:** "Did you correct me or push back on anything I generated?"
→ Captures corrections. Probe gently — corrections often feel minor in the moment but reveal important preferences. If the user says "not really," accept it. Don't fish.

**Question 3:** "Anything surprising, or something worth remembering for next time?"
→ Captures insights, experiments, or observations. Open-ended by design.

After all three questions, summarize what you'll capture and confirm before writing events.

### Typed capture (`--correction`, `--milestone`, `--insight`)

Capture a specific type of event. Ask one targeted question:
- `--correction`: "What did I get wrong, and what should I have done instead?"
- `--milestone`: "What was shipped or completed?"
- `--insight`: "What's the insight worth remembering?"

Then write the event directly.

## Writing Events

### Step 1: Determine today's next ID

```bash
grep -r "^id:" "<events_path>" 2>/dev/null | grep "$(date +%Y-%m-%d)" | sort | tail -1
```

Extract the NNN from the last `evt-YYYY-MM-DD-NNN` for today and increment. If none found, start at 001.

### Step 2: Determine the project context

Check the current working directory to infer the project. Use a short lowercase kebab-case slug (e.g., `general`, `website-redesign`, `data-pipeline`). If unclear, ask the user: "Which project does this relate to?"

### Step 3: Create the event file

Read the appropriate template from `templates_path`. Fill in all required fields:

```yaml
---
id: evt-YYYY-MM-DD-NNN
date: YYYY-MM-DD
type: correction|insight|milestone|experiment|reflection
project: project-slug
source_session: folder-name
title: "Short descriptive title"
tags: []

signal_type: negative|positive|neutral
confidence: high|medium|low

reinforcement_count: 1
reinforcement_log:
  - date: YYYY-MM-DD
    event_id: evt-YYYY-MM-DD-NNN
    note: "Initial signal"
last_reinforced: YYYY-MM-DD
decay_half_life_days: 30

related_events: []
triggered_by: null
supersedes: null
parent_pattern: null
---
```

**For corrections specifically:**
- `signal_type` should be `negative`
- Body must include: what Claude did wrong, what the user wanted instead, and why it matters for future alignment
- Check if this reinforces an existing correction — if so, update the original event's reinforcement fields

### Step 4: Save the event

Path: `<events_path>/YYYY/MM/YYYY-MM-DD_<type>_<project>_<slug>.md`

All segments lowercase kebab-case. Create date directories if needed.

### Step 5: Check for reinforcement

Scan recent events (last 30 days) for related signals:
```
Glob for events in the current and previous month's directories
```

If the new event echoes an existing signal:
1. Update the original event: increment `reinforcement_count`, add to `reinforcement_log`, update `last_reinforced`, recalculate `decay_half_life_days = 30 * 2^(reinforcement_count - 1)`
2. Set `related_events` in both events to cross-link them

### Step 6: Confirm

After writing, report what was captured:
> "Captured: [event title] (evt-YYYY-MM-DD-NNN, type: [type], project: [project])"

## Tone

Be conversational, not clinical. This is a reflection, not a form. If the user gives you a one-word answer, that's fine — write a focused event. If they want to think out loud, let them — capture the essence, not a transcript.

**You do the work, the user edits.** The default should feel like reviewing a 10-second summary, not recalling a 60-minute session from memory. Claude has the full context — use it. Never make the user reconstruct what happened when you can reconstruct it yourself.

## Key Rules

1. **Write first, report after.** Don't propose — just capture and confirm what was written. Pause only for genuine callouts (ambiguous corrections, uncertain reinforcement).
2. **Corrections are gold.** Treat them with care. Capture the why, not just the what.
3. **One event per insight.** Don't bundle multiple unrelated things into one event.
4. **Check for reinforcement.** The decay model depends on it.
5. **Brevity over completeness.** A captured signal is infinitely more valuable than a lost one. Short events are fine.
