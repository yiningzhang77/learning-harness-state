# personal-state OS

> Repository name is historical: `learning-harness-state` started as a learning harness, but the repository is now growing into a broader personal state system.

This README defines the **architecture and domain boundaries** of the repository. Domain-specific behavior belongs in `policies/`; record shapes belong in `schemas/`; actual state belongs in domain state folders.

## Four-layer model

Remember the repository in four sentences:

- **Schema = Shape** — what stored data may look like.
- **Policy = Behavior** — when to read/write, how to route, how to correct.
- **State folders = Data** — the factual/chronological records for each domain.
- **README = Architecture** — how the domains and layers fit together.

`derived/` is a fifth, non-authoritative layer: reproducible views/models folded from source data.

## Domain map

```text
personal-state/
├── README.md
├── schemas/
├── policies/
│
├── LearningState/
├── LearningDiagnostics/
│
├── TrainingState/
├── LifeState/
│
└── derived/
```

### Learning domain

`LearningState/`
- chronological learning history;
- assignment/progress/experiment/checkpoint/daily-snapshot events;
- current active append-only source: `LearningState/learning-state.jsonl`;
- append only; corrections are new events, not rewrites;
- `seq` remains monotonic.

`LearningDiagnostics/`
- concrete places where independent generation failed or became non-automatic;
- preserves first attempt, already-known intent, missing dimension, cue/recovery, provisional classification, and recurrence keys;
- daily filename: `LearningDiagnostics/YYYY-MM-DD.json`;
- diagnostic interpretation does **not** replace chronological LearningState history.

Current learning schemas:

```text
schemas/
├── learning-event.schema.json
├── learning-checkpoint.schema.json
├── learning-current.schema.json
├── learning-collision.schema.json
└── learning-diagnostics-day.schema.json
```

Current learning policies:

```text
policies/
├── ingestion.md
└── learning.md
```

### Training domain

`TrainingState/` is reserved for **physical training / fitness**.

Examples include workouts, running, mobility, strength work, recovery, training load, exercise adherence, and related physical-training state.

Important namespace rule:

> Learning drills, study exercises, model training, and learning weaknesses must never be written to `TrainingState/` merely because they are called “training”.

The training domain may define its own schemas and policies independently. A training-side change must not rewrite learning state, learning schemas, or learning diagnostics as a side effect.

### Life domain

`LifeState/` is for general life-state data that does not belong to a more specific domain. As new domains become substantial, prefer creating a clear domain boundary rather than turning `LifeState/` into a catch-all.

## Routing before writing

Before any write, follow `policies/ingestion.md` and resolve:

1. **Domain** — learning, physical training, life, or another domain.
2. **Record kind** — event, checkpoint, diagnostic, schema/policy change, or derived view.
3. **Source-of-truth role** — primary history/evidence or interpretation/computation.
4. **Mutation mode** — append-only, create-new, or update-in-place.
5. **Effective date / filename** — what time the record actually describes.

If routing is ambiguous, do not guess. Resolve the destination first.

## Cross-domain change contract

To keep parallel work from colliding:

- A domain may freely evolve files that are clearly namespaced to that domain.
- Do not reuse a folder name with a different meaning in another domain.
- Do not mutate another domain's State folder as a side effect of local work.
- Domain-specific schemas use explicit prefixes such as `learning-...`, `training-...`, `life-...`.
- Domain-specific policies should likewise be explicit (`learning.md`, `training.md`, etc.).
- Shared routing/architecture changes belong in `README.md` and/or `policies/ingestion.md`.
- If a future object is genuinely cross-domain, define the shared contract explicitly rather than silently placing it under one domain.

This means learning-side work can continue while training-side work evolves without either side claiming the other's namespace.

## Learning write model

A meaningful learning session may produce **both** kinds of records:

```text
What happened / what passed / what changed?
    -> LearningState/

Where exactly did generation fail, what was already known,
what cue repaired it, and what pattern might this indicate?
    -> LearningDiagnostics/
```

A green test and a collision record are complementary evidence:

- green test: the implementation eventually works;
- collision: where independent generation stopped working before it became green.

See `policies/learning.md` for the full evidence model.

## Evidence rules shared across domains

- Preserve observed facts separately from interpretation.
- Do not invent completion timestamps, durations, measurements, or causes.
- When the user supplies an exact time/duration/measurement, preserve it exactly where the domain schema supports it.
- Derived summaries must remain distinguishable from source records.
- Historical append-only records are corrected by superseding/correction records, not silent edits.

## Current active layout

```text
README.md

schemas/
  learning-event.schema.json
  learning-checkpoint.schema.json
  learning-current.schema.json
  learning-collision.schema.json
  learning-diagnostics-day.schema.json

policies/
  ingestion.md
  learning.md

LearningState/
  learning-state.jsonl

LearningDiagnostics/
  YYYY-MM-DD.json

TrainingState/
  # physical training / fitness domain

LifeState/
  # general life domain

derived/
  # reproducible folded/aggregated views; not source of truth
```

Some domain folders or schemas may be added gradually rather than created empty in advance. The architecture is allowed to grow from real data and real use cases.

## Historical note

Earlier versions of this README described a learning-only architecture based on `raw/YYYY-MM-DD/<workspace_id>/` checkpoints and `events/YYYY-MM-DD.jsonl` shards. That design is historical and is **not the current routing contract**.

For current writes, use this README together with `policies/ingestion.md` and the relevant domain policy/schema. Existing historical artifacts, if present, remain audit/history and should not be silently rewritten.

## Guiding rule

Before adding a file, ask:

> Is this **shape**, **behavior**, **source data**, **diagnostic interpretation**, or a **derived view** — and which domain owns it?

If that answer is clear, the repository can grow without collapsing into one undifferentiated log.