# learning-harness-state

Private event store for the user's 8Week learning harness.

## Source of truth

`events/YYYY-MM-DD.jsonl` contains the immutable event shard for one Asia/Shanghai calendar day.

Only conversations that actually served as that day's active 8Week learning execution workspace are valid event sources. Learning-related side chats, career discussion, OSS/news browsing, and meta-learning design conversations are excluded unless they were themselves the active execution workspace.

## Invariants

1. Append-only by shard: create one daily shard; never edit an existing shard after a successful commit.
2. Evidence over vibes: do not infer mastery, completion, exact duration, or independence without evidence.
3. Assistance is tracked with A0-A4.
4. Capability evidence is tracked with L1-L6.
5. Large holdouts must come from real university assignments or real OSS tasks; small transfer probes may be authored directly.
6. If the active execution conversation is ambiguous, record no shard and report the ambiguity rather than broad-scanning unrelated chats.
7. If persistence fails, report the exact would-be JSONL and do not claim success.

## Layout

- `schema.json` — event schema and scales
- `events/YYYY-MM-DD.jsonl` — immutable daily event shards
