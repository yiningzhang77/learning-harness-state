# Ingestion Policy

Purpose: decide **where a new record belongs before writing it**. This repo is a personal state system; folder names are domains, schemas define shape, policies define behavior, and state folders store source data.

## Pre-write routing check

Before every write, answer these five questions:

1. **Domain** — Is this learning, physical training/fitness, general life, or another domain?
2. **Record kind** — Is this a raw event, a daily checkpoint, a diagnostic interpretation, a schema/policy change, or a derived view?
3. **Source of truth** — Is the record primary evidence/history or something computed/interpreted from that history?
4. **Mutation mode** — Append-only, create-new-daily-file, or replace/update?
5. **Date/name** — What effective date does the record describe, and what filename convention applies?

Do not write until the destination is resolved.

## Routing table

### LearningState/

Use for chronological learning events and checkpoints that are part of the append-only learning history.

Current source-of-truth file:

`LearningState/learning-state.jsonl`

Rules:
- append new events only;
- never delete or rewrite prior event lines to correct history;
- if an old event is inaccurate, append a correction/superseding event;
- continue `seq` monotonically from the current latest event;
- preserve user-supplied completion times/elapsed times exactly when available;
- never invent a completion timestamp when only an elapsed duration is known.

### LearningDiagnostics/

Use for detailed records of **where learning failed or became non-automatic**, including concrete collisions, cue/recovery behavior, provisional classifications, and daily pattern summaries.

Filename convention:

`LearningDiagnostics/YYYY-MM-DD.json`

Rules:
- one diagnostic document per effective day;
- collisions should preserve the concrete observed attempt/failure, not only the final lesson;
- interpretation belongs in `working_hypothesis`, not in factual observation fields;
- diagnostics may reference LearningState event seqs but do not replace the LearningState history;
- a diagnostic file may be updated if additional collisions from the same effective day are discovered later, but observed facts must not be silently rewritten into cleaner narratives.

### TrainingState/

Reserved for **physical training / fitness state**. Do not route learning drills, learning weaknesses, model training, or study exercises here.

### LifeState/

Use for general life-state records that do not belong to a more specific domain.

### schemas/

Use only for contracts that define what stored data may look like. Schemas are not logs and must not contain daily state as examples that could be mistaken for source data.

### policies/

Use only for behavior/routing rules: when to read, when to write, how to decide, how to correct, and how to preserve evidence.

### derived/

Use for computed/folded views such as current-state snapshots, recurrence summaries, clusters, dashboards, or other reproducible projections. Derived data is not the source of truth.

## Learning-specific decision tree

When a learning session produces information:

- "What happened / what was completed / what passed / what changed?" -> `LearningState/`
- "Where exactly did generation fail, what was already known, what cue repaired it, and what pattern might this indicate?" -> `LearningDiagnostics/`
- "What fields should those records have?" -> `schemas/`
- "When should we record or interpret them?" -> `policies/`
- "What does the history currently imply after folding/aggregation?" -> `derived/`

A single session may legitimately write both LearningState and LearningDiagnostics because they answer different questions.

## Corrections and ambiguity

If destination is ambiguous, prefer **no write** over a guessed write. Resolve the domain and record kind first.

If a wrong destination was previously used:
- do not erase append-only history;
- add a correction/migration note where needed;
- create the correctly routed record;
- preserve enough linkage to explain why both records exist.

## Naming conventions

- schemas: lowercase kebab-case ending in `.schema.json`
- policies: lowercase kebab-case Markdown
- daily diagnostic files: `YYYY-MM-DD.json`
- append-only learning event file: retain the existing `learning-state.jsonl` path and monotonic `seq`

The routing principle is simple: **history goes to State, failure analysis goes to Diagnostics, shape goes to schemas, behavior goes to policies, computed views go to derived.**