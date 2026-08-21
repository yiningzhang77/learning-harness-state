# learning-harness-state

Private event store for the user's 8Week learning harness.

## Source of truth

The primary source of truth is the set of immutable source-conversation checkpoints under:

`raw/YYYY-MM-DD/<workspace_id>/*.json`

Each active learning conversation writes its own evidence **while that conversation still has the full local context**. This removes any dependency on a scheduled task reconstructing arbitrary ChatGPT conversations later.

`events/YYYY-MM-DD.jsonl` is a normalized/derived daily shard when one exists. Existing daily shards remain valid historical artifacts, but future controllers must be able to fold raw checkpoints directly and must not require a midnight reconciliation step.

Only conversations that actually served as an active 8Week learning execution workspace are valid sources. Learning-related side chats, career discussion, OSS/news browsing, and meta-learning design conversations are excluded unless they themselves became an execution workspace.

## Learning Log Skill

### Trigger phrases

When the user says any short command with the same intent as:

- `追加今天日志`
- `记录今天学习`
- `今天学完了，写日志`
- `收工，追加日志`

execute this skill without asking the user to restate the logging rules.

The **current conversation is authoritative**. Treat this chat as the execution workspace whose new learning evidence should be persisted. Do not search unrelated chats to make the log look more complete.

### Skill procedure

1. Read this `README.md` and `schema.json` from `yiningzhang77/learning-harness-state` before writing.
2. Resolve the current Asia/Shanghai calendar date.
3. Treat the current conversation as the source workspace. Recover a human-readable title if available, but never use the title as identity.
4. Resolve or create a stable `workspace_id` in the form `W<week>-YYYYMMDD-NN`. If this chat is continuing a previous workspace, preserve `parent_workspace_id` when known.
5. Inspect existing checkpoints under `raw/YYYY-MM-DD/<workspace_id>/` and extract only evidence that has not already been persisted.
6. Extract only **state-changing learning evidence** from the current conversation. Do not log every question or explanation.
7. Create a NEW immutable checkpoint file under `raw/YYYY-MM-DD/<workspace_id>/`. Never edit or overwrite an earlier checkpoint.
8. If the command indicates the current session is finished (`今天学完了`, `收工`, etc.), set `closing=true` and include the current open loops and next dependency. The short command `追加今天日志` may be treated as a normal end-of-session checkpoint unless the surrounding conversation clearly indicates learning will continue immediately.
9. Report the created GitHub path and a concise summary of what was recorded. If persistence fails, show the exact would-be checkpoint and state that it was not persisted.

No midnight logger is required. Persistence happens at the source conversation.

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

Capability labels are always **evidence under an assistance condition**, not permanent mastery labels. Prefer `L5 evidence under A3` over `capability = L5`.

### Workspace handoff

When the user says something like `这个框接 Week1`, `接着 Week1`, or `换框，接着 Week1`:

- treat the current chat as a new execution workspace;
- read recent raw checkpoints and any normalized events to recover prior open loops;
- create a new `workspace_id` if this is a different chat;
- set `parent_workspace_id` to the prior workspace when recoverable;
- continue from GitHub state rather than relying on cross-chat memory.

The conversation title is only a label. `workspace_id` is the identity.

### Multiple workspaces in one day

Multiple learning chats in one Asia/Shanghai day are allowed. Each chat writes its own immutable checkpoints. Downstream consumers fold all valid checkpoints for the date in checkpoint/commit order.

If one workspace stops and another continues the same work, use `parent_workspace_id` to preserve the handoff chain. Do not merge evidence from different chats into a single checkpoint because provenance matters for assistance and capability evaluation.

## Downstream consumers

The 08:00 Learning Daily Controller and the OSS Capability Radar should:

1. read `README.md` and `schema.json`;
2. fold committed `raw/` checkpoints in date/workspace/checkpoint order;
3. also read any historical normalized `events/YYYY-MM-DD.jsonl` shards when useful, while avoiding duplicate evidence already represented in raw checkpoints;
4. derive current state from evidence, not from chat titles or broad cross-chat retrieval;
5. never modify raw checkpoints.

## Invariants

1. Append-only by artifact: raw checkpoints are create-only.
2. Evidence over vibes: do not infer mastery, completion, exact duration, or independence without evidence.
3. Assistance is tracked with A0-A4.
4. Capability evidence is tracked with L1-L6.
5. Large holdouts must come from real university assignments or real OSS tasks; small transfer probes may be authored directly.
6. If provenance is ambiguous, fail closed rather than broad-scanning unrelated chats.
7. If persistence fails, report the exact would-be record and do not claim success.
8. Source conversations produce the authoritative evidence stream.
9. Conversation titles are labels; `workspace_id` is identity.
10. No scheduled midnight conversation-retrieval/reconciliation step is required.

## Layout

- `README.md` — operating protocol / Learning Log Skill
- `schema.json` — A0-A4 / L1-L6 scales and normalized schema
- `raw/YYYY-MM-DD/<workspace_id>/*.json` — authoritative immutable source-conversation checkpoints
- `events/YYYY-MM-DD.jsonl` — optional/historical normalized daily shards
- `LearningState/learning-state.jsonl` — legacy/audit record only; not the active source of truth
