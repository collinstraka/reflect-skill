---
id: evt-YYYY-MM-DD-NNN                # Unique, sortable ID (NNN = sequence within day, start 001)
date: YYYY-MM-DD                       # Date of the event
time: "HH:MM"                         # Optional, for ordering within a day
type: inbox-processed
project: project-slug                  # Lowercase kebab-case project identifier
source_session: folder-name            # Which Claude session/folder generated this
title: "Short descriptive title"       # Brief, scannable title
tags: []                               # Freeform tags for slicing and querying

# Alignment signal
signal_type: neutral                   # positive | negative | neutral
confidence: medium                     # high | medium | low — how clear the signal was

# Reinforcement tracking (decay model)
reinforcement_count: 1                 # Starts at 1, incremented on each reinforcement
reinforcement_log:                     # Entry 1: event_id is this event itself (self-ref)
  - date: YYYY-MM-DD                   # Subsequent entries: event_id is the reinforcing event
    event_id: evt-YYYY-MM-DD-NNN
    note: "Initial signal"
last_reinforced: YYYY-MM-DD            # Drives decay calculation
decay_half_life_days: 30               # Formula: 30 * 2^(reinforcement_count - 1)
                                       # 1x=30d, 2x=60d, 3x=120d, 4x=240d, 5x=480d, 6x=960d

# Graph-ready fields (optional — for future knowledge graph)
related_events: []                     # IDs of related events
triggered_by: null                     # ID of event that caused this one
supersedes: null                       # ID of event this replaces/evolves
parent_pattern: null                   # Links to a synthesis/patterns/ file

# Inbox provenance
source_file: "original-filename.ext"   # Original filename from inbox
source_type: article                   # article | repo | tweet | experiment | reference
---

## What was ingested
[Description of the source material — what it is, where it came from]

## Key takeaways
[The important insights, ideas, or reference points extracted]

## Events generated
[List of event IDs created from this ingestion, if any]
