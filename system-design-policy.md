# System Design Progression Policy

This file defines how the Learning Daily Controller decides **when to introduce which system-design questions and how deep the user's answer is expected to go**.

The controller must gate by demonstrated evidence in `raw/` checkpoints, not by calendar week, confidence, or topic familiarity alone.

## Exposure modes

Each candidate system-design question must be assigned exactly one mode:

- `HOLD` — prerequisite evidence is too weak; do not assign the question yet. State the exact missing evidence.
- `PREVIEW` — the question may be shown for a 5–10 minute architecture sketch. The goal is vocabulary and component recognition, not a complete design.
- `PRACTICE` — assign a 20–30 minute structured design. Hints may be allowed, but the user should own the main decomposition and tradeoffs.
- `INTERVIEW` — assign a 35–45 minute largely independent answer under A0/A1 conditions, followed by critique.

Exposure is monotonic only when supported by evidence. A later failure may temporarily lower the expected mode for a specific topic without lowering all system-design ability.

## Answer-depth rubric

Use the following answer-depth levels independently from L1–L6 capability labels:

### SD-D0 — Request-flow sketch
Expected answer:
- identify actors and one end-to-end request flow;
- name the main service/process boundaries;
- identify where state lives;
- identify the protocol/API boundary;
- state one obvious failure or bottleneck.

Do not require capacity math, partitioning, consistency models, or production SLOs.

### SD-D1 — Single-service design
Expected answer:
- clarify functional requirements and a small set of non-functional requirements;
- propose API endpoints or message contracts;
- propose a data model and state ownership;
- draw/read the request lifecycle through service + storage;
- discuss basic cache/index/queue choices when relevant;
- identify obvious bottlenecks and one or two failure paths.

### SD-D2 — Distributed scaling design
Expected answer adds:
- rough capacity reasoning when useful;
- horizontal scaling and stateless/stateful boundaries;
- cache strategy and invalidation implications;
- async queue/event use and backpressure where relevant;
- partitioning/sharding/replication choices;
- consistency and availability tradeoffs;
- retries, idempotency, deduplication, timeout behavior;
- observability signals and failure isolation.

### SD-D3 — Production reliability design
Expected answer adds:
- explicit SLO/SLA framing;
- failure domains and degradation strategy;
- overload handling/load shedding/backpressure;
- recovery, replay, poison-message or retry-storm considerations where relevant;
- operational observability, alerting and capacity headroom;
- migration/versioning or rollout considerations;
- security/abuse/privacy constraints when relevant;
- defend tradeoffs against at least one credible alternative.

### SD-D4 — AI/Agent system design
Expected answer adds the relevant AI-specific dimensions:
- model/tool/workflow orchestration;
- context construction and memory/state boundaries;
- retrieval/data freshness and document-processing path when relevant;
- sandbox/tool-permission boundaries;
- latency, token/cost and concurrency budgeting;
- eval/observability and failure taxonomy;
- fallback/retry/model-routing behavior;
- safety/guardrail/human-escalation boundaries;
- online/offline feedback loop and rollout strategy.

`SD-D4` does not replace D1–D3. A strong AI/Agent design must still satisfy the appropriate backend/distributed/reliability depth.

## Progression gates

### Stage SD0 — Local request lifecycle
Typical questions:
- trace/design one HTTP request through a tiny service;
- design the internal shape of a minimal web framework or request router;
- where should parsing, serialization, routing and state ownership live?

Default target: `PREVIEW/PRACTICE` at `SD-D0`.

Unlock evidence should include several of:
- concrete request-lifecycle reasoning;
- client/server and protocol-boundary understanding;
- serialization/parsing evidence;
- async task/request lifetime reasoning;
- code-level evidence from a real assignment or system.

### Stage SD1 — Small service / storage design
Typical questions:
- URL shortener;
- paste/text sharing service;
- simple file metadata service;
- basic rate limiter;
- webhook receiver;
- small job submission API.

Default target: `PRACTICE` at `SD-D1`.

Unlock from SD0 when evidence shows the user can independently or near-independently reason about:
- API/request flow;
- state ownership and a basic data model;
- at least one storage/cache/index tradeoff;
- basic failure paths;
- not merely name components.

### Stage SD2 — Distributed primitives
Typical questions:
- scalable chat/message delivery;
- notification service;
- distributed job queue;
- feed/timeline fanout;
- scalable file upload/processing pipeline;
- metrics/log ingestion pipeline.

Default target: `PRACTICE`, later `INTERVIEW`, at `SD-D2`.

Unlock when evidence spans multiple real primitives, preferably from assignments/OSS/projects:
- concurrency/networking;
- persistent storage/data modeling;
- cache and/or queue semantics;
- retries/timeouts/idempotency or equivalent failure-control reasoning;
- ability to debug or explain a multi-component failure path.

### Stage SD3 — Production distributed reliability
Typical questions:
- webhook delivery platform;
- payment/event processing pipeline;
- distributed scheduler;
- high-volume notification system;
- multi-region API/service;
- observability/eval ingestion platform.

Default target: `INTERVIEW` at `SD-D3` only after repeated D2 evidence.

Unlock when the logs show repeated evidence of:
- unfamiliar-system debugging at roughly L4 or stronger under low assistance;
- design/build evidence across more than one subsystem;
- explicit tradeoff reasoning rather than component listing;
- failure recovery/backpressure/overload/observability reasoning;
- at least one substantial real system, university assignment, or OSS contribution where these concerns mattered.

### Stage SD4 — AI / Agent infrastructure
Typical questions:
- production RAG service;
- agent execution/runtime platform;
- coding-agent harness;
- evaluation platform for LLM applications;
- long-term memory/context service;
- tool-calling/sandbox execution platform;
- multi-model routing and fallback service.

Default target can begin as `PREVIEW` before SD3 is complete when the user's AI application experience makes the question useful, but a full `INTERVIEW` answer should require backend/distributed evidence at approximately SD-D2/SD-D3 depth.

AI-specific unlock evidence should include several of:
- real RAG/agent implementation evidence;
- evaluation/experimentation evidence;
- backend/API integration evidence;
- tool-calling/context/memory or retrieval-system reasoning;
- latency/cost/failure-mode reasoning;
- real OSS or production-like debugging evidence.

## Controller rules

The Learning Daily Controller must maintain a separate `System Design` lane every day, even when it assigns no system-design question.

The lane should report:

- `current_stage`: SD0–SD4 (stage can be multi-dimensional; e.g. backend SD1 while AI-specific preview is SD4-PREVIEW);
- `current_answer_depth`: SD-D0–SD-D4;
- `exposure_mode`: HOLD / PREVIEW / PRACTICE / INTERVIEW;
- `evidence_basis`: concrete raw checkpoints supporting the rating;
- `next_unlock`: the smallest missing evidence needed for the next stage/depth;
- `candidate_question`: at most one question when useful;
- `expected_answer_contract`: exactly what the user is expected to cover now, and what is explicitly *not* required yet;
- `assistance_cap`: A0–A4 appropriate to the mode;
- `promotion_test`: what evidence would justify increasing answer depth or moving to INTERVIEW mode.

Do not assign a system-design question merely because one is famous or commonly asked. Prefer questions whose primitives intersect with the user's current real work, so system design consolidates engineering experience instead of becoming vocabulary memorization.

Do not equate familiarity with readiness. Do not treat a polished GPT-assisted answer as independent interview evidence.
