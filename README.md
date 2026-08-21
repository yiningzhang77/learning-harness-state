# learning-harness-state

Private event store for the user's 8Week learning harness.

## Source of truth

`events/YYYY-MM-DD.jsonl` contains the immutable, normalized event shard for one Asia/Shanghai calendar day.

Only conversations that actually served as that day's active 8Week learning execution workspace are valid event sources. Learning-related side chats, career discussion, OSS/news browsing, and meta-learning design conversations are excluded unless they were themselves the active execution workspace.

Raw evidence should be captured **at the source conversation while its full context is still available**, then reconciled into the daily event shard. The midnight task must not depend on reconstructing arbitrary ChatGPT conversations after the fact.

## Learning Log Skill

### Trigger phrases

When the user says any short command with the same intent as:

- `追加今天日志`
- `记录今天学习`
- `今天学完了，写日志`
- `收工，追加日志`

execute this skill without asking the user to restate the logging rules.

The current conversation is authoritative: treat **this chat** as the execution workspace whose learning evidence should be logged. Do not search unrelated chats to make the log look more complete.

### Skill procedure

1. Read this `README.md` and `schema.json` from `yiningzhang77/learning-harness-state` before writing.
2. Resolve the current Asia/Shanghai calendar date.
3. Treat the current conversation as the source workspace. Recover a human-readable title if available, but do not use the title as identity.
4. Resolve or create a stable `workspace_id` in the form `W<week>-YYYYMMDD-NN`. If this chat is continuing a previous workspace, preserve `parent_workspace_id` when known.
5. Inspect existing `raw/YYYY-MM-DD/` checkpoint files to avoid duplicating evidence already committed from this workspace.
6. Extract only new, state-changing learning evidence from the current conversation since the last checkpoint. Do not log every question or explanation.
7. Create a **new immutable checkpoint file** under `raw/YYYY-MM-DD/<workspace_id>/`. Never edit or overwrite an earlier raw checkpoint.
8. If the user indicates the session/day is finished (`今天学完了`, `收工`, etc.), mark the checkpoint as a closing checkpoint and include current open loops / next dependency. Otherwise keep the workspace open.
9. Report the created GitHub path and a concise summary of what was recorded. If persistence fails, show the exact would-be checkpoint and state that it was not persisted.

### What counts as a state-changing checkpoint

Log when at least one of these occurred:

- a real component / assignment section was implemented or completed;
- a meaningful runtime probe, official test, benchmark, or eval produced evidence;
- an important misconception or failure mode was exposed or corrected;
- a blocker was resolved or a new blocker was identified;
- the demonstrated assistance level materially changed;
- an L1-L6 probe or holdout produced evidence;
- the assignment / learning direction changed;
- the conversation is handing off to another execution workspace.

Do **not** create noisy events for ordinary clarifying questions unless they reveal a misconception, transfer signal, or dependency that changes the learning state.

### Raw checkpoint schema

A raw checkpoint should preserve provenance and should normally include:

```json
{
  "date": "YYYY-MM-DD",
  "workspace_id": "W1-YYYYMMDD-01",
  "parent_workspace_id": null,
  "source_conversation": "human-readable title if available",
  "type": "workspace/checkpoint",
  "closing": false,
  "topics": [],
  "tasks": [],
  "evidence": [],
  "assistance": [],
  "capability_evidence": [],
  "misconceptions_or_failures": [],
  "transfer_signals": [],
  "unresolved": [],
  "next_dependency": null,
  "provenance_confidence": "high"
}
```

`evidence` should distinguish claim strength where possible, for example: user statement, code implemented, runtime probe passed, official test passed, unseen transfer task passed.

Capability labels are always **evidence under an assistance condition**, not permanent mastery labels. Prefer language such as `L5 evidence under A3` rather than `capability = L5`.

### Workspace handoff

When the user says something like `这个框接 Week1`, `接着 Week1`, or `换框，接着 Week1`:

- treat the current chat as a new execution workspace;
- read recent raw checkpoints / normalized events to recover the prior open loops;
- create a new `workspace_id` if this is a different chat;
- set `parent_workspace_id` to the prior workspace when recoverable;
- continue from GitHub state rather than relying on cross-chat memory.

The conversation title is only a label. `workspace_id` is the identity.

## Midnight reconciliation

The scheduled midnight logger is a **reconciler, not a conversation retriever**.

For the previous Asia/Shanghai day it should:

1. read `schema.json`;
2. read all raw checkpoints under `raw/YYYY-MM-DD/`;
3. fail closed if there are no valid raw checkpoints or if provenance is ambiguous;
4. deduplicate and normalize the raw evidence;
5. create exactly one immutable `events/YYYY-MM-DD.jsonl` daily shard;
6. never modify an existing daily shard after successful commit;
7. include one final `day/summary` object with mainline progress, spiral-CS exposures/blockers, eval evidence, open loops, and failure taxonomy.

This design intentionally removes the requirement for the midnight task to open or reconstruct arbitrary ChatGPT conversations.

## Invariants

1. Append-only by artifact: raw checkpoints are create-only; daily shards are create-only after commit.
2. Evidence over vibes: do not infer mastery, completion, exact duration, or independence without evidence.
3. Assistance is tracked with A0-A4.
4. Capability evidence is tracked with L1-L6.
5. Large holdouts must come from real university assignments or real OSS tasks; small transfer probes may be authored directly.
6. If provenance is ambiguous, fail closed rather than broad-scanning unrelated chats.
7. If persistence fails, report the exact would-be record and do not claim success.
8. Source conversations produce raw evidence; midnight reconciliation produces normalized daily state.

## Layout

- `README.md` — operating protocol / Learning Log Skill
- `schema.json` — normalized event schema and A0-A4 / L1-L6 scales
- `raw/YYYY-MM-DD/<workspace_id>/*.json` — immutable source-conversation checkpoints
- `events/YYYY-MM-DD.jsonl` — immutable normalized daily event shards
- `LearningState/learning-state.jsonl` — legacy/audit record only; not the active source of truth
