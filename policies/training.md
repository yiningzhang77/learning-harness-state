# Training Policy

Purpose: store **physical training capability state**, not merely workout volume.

`TrainingState/` is the source history for movement calibration, mobility, strength-control, proprioception, exercise tolerance, recovery-relevant training observations, and progression decisions.

## Write trigger

Training checkpoints are **user-controlled**.

Write only when the user explicitly asks to store/sync/upload the training state or training log. Phrases such as `存`, `记录`, `同步`, or `上传日志` count as explicit triggers when the surrounding context clearly refers to physical training.

On a trigger:
- retrieve the relevant unpersisted training context from the current conversation;
- fold it into one coherent checkpoint per effective date;
- preserve chronological order of observations without inventing exact clock times;
- do not continuously persist every exercise message.

## Record model

Daily source record:

`TrainingState/YYYY-MM-DD.json`

Schema:

`schemas/training-day.schema.json`

A daily record should emphasize **state change**:
- what movement/capability was introduced or clarified;
- what the user could or could not perceive/control;
- what range/load was tolerated;
- what provoked symptoms or compensation;
- what cue/regression improved the movement;
- what should be re-exposed later and under what progression gate.

Do not reduce the record to reps, minutes, calories, or completion status when the meaningful evidence is motor control or body awareness.

## Capability stages

Use the smallest useful vocabulary:

- `not_introduced`
- `learning`
- `rebuilding`
- `stable`
- `ready_to_progress`

Conceptual understanding and embodied control are different evidence. A user may understand a cue while the movement remains `learning` or `rebuilding`.

## Observation vs interpretation

Keep direct user-observed facts separate from assistant interpretation.

Examples of observation:
- low back stayed stable during a heel tap;
- shoulders/upper traps became the first fatigued area;
- posterior pelvic tilt produced central lumbar ache;
- returning to neutral made the ache disappear almost immediately.

Examples of interpretation:
- likely compensation;
- current tolerance in that movement direction appears low;
- proprioceptive discrimination is emerging.

Interpretation must remain provisional and must not be written as diagnosis or structural certainty.

## Pain / symptom handling

Pain is **not** an automatic instruction to train the painful region harder.

When a movement clearly increases pain or other concerning symptoms:
- stop or regress the provoking movement for that session;
- record the response as tolerance evidence;
- prefer comfortable small-range exposure over repeated provocation;
- do not infer tissue damage or medical diagnosis from training observations.

Neurologic symptoms, true fainting/presyncope, chest pain, breathing difficulty, marked weakness, radiating pain, or persistent/worsening symptoms belong outside ordinary progression logic and should be treated as safety context rather than a training target.

## Corrections

Daily training files may be updated later when the user explicitly asks to add newly recovered evidence from the same date. Preserve prior observed facts; do not rewrite them into a cleaner story that changes what was actually reported.

## Guiding rule

TrainingState should answer:

> What can this body currently perceive, control, tolerate, and safely progress — and what evidence supports that state?
