# Learning Policy

Purpose: preserve the difference between **knowing the idea**, **being able to express it in code**, and **being able to generate the full solution without cues**. Learning records should make those boundaries visible instead of compressing them into a success/failure summary.

## What LearningState records

`LearningState/learning-state.jsonl` records chronological learning events: assignments, progress, experiments, checkpoints, test evidence, decisions, and daily snapshots.

A LearningState event should answer questions such as:
- What task was attempted or completed?
- What evidence exists (tests, runtime result, artifact, commit)?
- What capability changed?
- What assistance level was required?
- What remains open or comes next?

LearningState should summarize collisions when useful, but it does not need to preserve every concrete failed line or local mistake in full detail.

## What LearningDiagnostics records

Create/update `LearningDiagnostics/YYYY-MM-DD.json` when the session contains concrete collisions worth preserving.

A collision is worth recording when at least one of these is true:
- the intended behavior was understood but the concrete Python/API expression could not be generated;
- a missing dimension caused the first design to fail (state lifetime, representation boundary, control-flow scope, optional state, process boundary, etc.);
- a small cue caused immediate recovery and therefore reveals a retrieval/automaticity gap;
- the same class of failure has appeared before or seems likely to transfer across contexts;
- the failure explains why an apparently understood concept was not sufficient to produce correct code;
- the failure materially changed the implementation or mental model.

Do not record every typo. Preserve collisions that teach us something about the current learning system.

## Collision evidence model

Keep three layers separate.

### 1. Observed facts

Examples:
- what the user was trying to express;
- the concrete first attempt;
- the observed failure or limitation;
- what cue was supplied;
- whether the user recovered after the cue;
- test/runtime evidence.

### 2. Missing dimension / repair

Describe the mechanism that was absent from the first attempt, for example:
- optional mapping lookup rather than mandatory indexing;
- persistent state across outer-loop iterations;
- bytes versus stream object;
- bytes versus decoded text;
- dict exact lookup rather than scanning with a loop.

### 3. Working hypothesis

Interpretations such as "Python API automaticity is lagging architecture understanding" are hypotheses, not facts. Store them only as `working_hypothesis` and allow later evidence to revise them.

## Preserve `intent_already_known`

This field is important. It distinguishes different failure depths:

- **model gap**: the relevant behavior/dimension was not known or considered;
- **coverage gap**: the main model existed but an edge condition/dimension was omitted;
- **expression gap**: the desired behavior was known but could not be translated into the concrete language/API idiom;
- **retrieval/automaticity gap**: the mechanism was previously learned but did not surface unaided;
- **integration gap**: local pieces were known but their composition/control flow was not generated correctly.

Do not collapse these into "didn't know".

## Assistance and recovery

Record the smallest cue that materially changed the trajectory when possible.

Use the existing A0-A4 scale from LearningState:
- A0: closed-book / no external help
- A1: official docs/search allowed
- A2: hint/cue only
- A3: guided reasoning
- A4: code/solution assistance

For diagnostics, `recovery.outcome` should describe what happened after the cue, for example:
- `immediate-after-cue`
- `recovered-with-guidance`
- `partial-recovery`
- `not-recovered`

Do not upgrade a capability simply because a corrected solution was shown. Prefer evidence from user reconstruction, cold replay, or an unseen transfer.

## Classification and recurrence

`classification` labels are provisional descriptors, not diagnoses. Use them to support later clustering.

`recurrence_key` should name the transferable mechanism rather than the local task when possible.

Prefer:
- `optional-map-access`
- `state-lifetime-scope`
- `bytes-str-boundary`
- `io-object-vs-data`
- `nested-loop-state-machine`

Avoid overly local keys such as `mp7-content-length-bug` unless the issue truly does not generalize.

## When to create systematic review or micro-drills

Do **not** convert every collision into deliberate practice immediately.

Promote a pattern toward systematic review only when evidence supports it, such as:
- recurrence across multiple sessions;
- recurrence across different contexts/projects;
- repeated A2/A3 cue dependence;
- high-cost failures despite conceptual understanding;
- a foundational idiom that blocks many later tasks.

A single local collision normally stays `watch` unless its transfer value is unusually high.

When a drill is eventually created, it belongs to the learning domain (for example future `LearningDiagnostics`, `LearningReview`, or another explicitly named learning folder), **not `TrainingState/`**, which is reserved for physical fitness/training.

## Daily workflow

At the end of a meaningful learning session:

1. Append factual progress/evidence to LearningState when a learning-state event is warranted.
2. Create/update that effective day's LearningDiagnostics file if meaningful collisions occurred.
3. Preserve user-supplied elapsed/completion timing; do not infer missing wall-clock completion times.
4. Cluster only lightly at daily level; mark interpretations as provisional.
5. Do not manufacture drills from one day's data unless recurrence or impact already justifies it.

## Review workflow

During periodic review, examine diagnostics across days and ask:
- Which recurrence keys repeat?
- Which failures happen after the user already states the correct high-level intent?
- Which cues cause immediate recovery?
- Which errors disappear after one correction?
- Which mechanisms recur in different domains?

The goal is not to build a larger mistake list. The goal is to identify the smallest set of transferable mechanisms that still constrain independent generation.

## Guiding principle

A green test result says **the implementation works**. A collision record says **where independent generation stopped working before the implementation became green**. Preserve both.