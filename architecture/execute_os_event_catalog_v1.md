# Event Index

| Event | Category | Producer |
|------|------|------|
system_generated | lifecycle | Generation Service |
system_activated | lifecycle | Orchestrator |
system_adapted | lifecycle | Adaptation Service |
session_created | execution | Execution Service |
session_started | execution | Execution Service |
task_completed | execution | Execution Service |
phase_completed | execution | Execution Service |
system_completed | execution | Execution Service |
instance_migrated | lifecycle | Orchestrator |
system_published | ecosystem | Publication Service |
system_forked | ecosystem | Publication Service |
system_remixed | ecosystem | Generation Service |
proof_submitted | ecosystem | Execution API |
proof_verified | ecosystem | Curator Service |
proof_rejected | ecosystem | Curator Service |

# Execution OS — Event Catalog (v1)

This document defines the **canonical event log contract** for Execution OS.

Events represent the **immutable execution history of the platform**.

They are append-only and must **never be modified or deleted**.

Events are emitted **only after commands are validated and committed by the Orchestrator**.


# Event Categories

Events fall into three categories.

## Execution Telemetry

Generated during runtime execution.

Examples:

* session_created
* session_started
* task_completed
* task_blocked
* phase_completed
* system_completed

Used for:

* progress computation
* execution replay
* analytics
* pattern extraction


## Lifecycle Events

Generated when the platform lifecycle changes.

Examples:

* system_generated
* system_activated
* system_adapted
* instance_migrated

Used for:

* system evolution tracking
* migration tracking
* lifecycle auditing


## Ecosystem Events

Generated from public platform actions.

Examples:

* system_published
* system_forked
* system_remixed
* proof_submitted
* proof_verified

Used for:

* library indexing
* reputation signals
* ecosystem analytics


# Canonical Event Structure

All events follow the same structure.

```json
{
  "event_id": "UUID",
  "event_type": "STRING",
  "schema_version": 1,

  "entity_type": "STRING",
  "entity_id": "UUID",

  "instance_id": "UUID|null",
  "session_id": "UUID|null",

  "actor_id": "UUID|null",

  "event_sequence": "INTEGER|null",
  "event_timestamp": "TIMESTAMP",

  "dedupe_key": "STRING|null",

  "event_payload": {}
}
```

Field descriptions:

| Field           | Purpose                      |
| --------------- | ---------------------------- |
| event_id        | globally unique identifier   |
| event_type      | event name                   |
| schema_version  | schema evolution control     |
| entity_type     | entity affected              |
| entity_id       | entity identifier            |
| instance_id     | execution instance reference |
| session_id      | execution session reference  |
| actor_id        | actor responsible            |
| event_sequence  | per-session ordering         |
| event_timestamp | event creation time          |
| dedupe_key      | optional dedupe protection   |
| event_payload   | structured event data        |


# Event Producer Authority

Each event has a **single canonical producer service**.

| Event                        | Producer            |
| ---------------------------- | ------------------- |
| session_created              | Execution Service   |
| session_started              | Execution Service   |
| task_completed               | Execution Service   |
| task_blocked                 | Execution Service   |
| task_skipped                 | Execution Service   |
| phase_completed              | Execution Service   |
| session_completed            | Execution Service   |
| instance_stalled             | Pattern Service     |
| system_generated             | Generation Service  |
| system_activated             | Orchestrator        |
| system_adapted               | Adaptation Service  |
| instance_migration_requested | Orchestrator        |
| instance_migration_accepted  | Orchestrator        |
| instance_migrated            | Orchestrator        |
| system_completed             | Execution Service   |
| system_published             | Publication Service |
| system_forked                | Publication Service |
| system_remixed               | Generation Service  |
| proof_submitted              | Execution API       |
| proof_verified               | Curator Service     |
| proof_rejected               | Curator Service     |

Only the designated producer may emit that event.


# Event Ordering

Execution OS uses two ordering scopes.

## Session Ordering

Execution events must be ordered by:

```
(session_id, event_sequence)
```

Rules:

* `event_sequence` must be monotonic per session
* replay must follow this ordering

This guarantees deterministic session reconstruction.


## Global Ordering

System and ecosystem events use:

```
(event_timestamp, event_id)
```

Used for:

* system lifecycle replay
* ecosystem analytics
* public system indexing


# Event Storage Model

Events are stored in an append-only event log.

Example schema:

```
execution_events

event_id UUID PRIMARY KEY
event_type TEXT
schema_version INT

entity_type TEXT
entity_id UUID

instance_id UUID
session_id UUID

actor_id UUID

event_sequence INT

event_timestamp TIMESTAMP

dedupe_key TEXT

event_payload JSONB
```

Recommended indexes:

```
session_id
instance_id
entity_id
event_type
event_timestamp
```

Rules:

* events must not be updated
* events must not be deleted
* corrections require compensating events


# Execution Events

## session_created

Producer
Execution Service

Payload

```
{
  "instance_id": "UUID",
  "user_id": "UUID"
}
```


## session_started

Producer
Execution Service

Payload

```
{
  "instance_id": "UUID"
}
```


## task_completed

Producer
Execution Service

Payload

```
{
  "phase_index": "INTEGER",
  "task_index": "INTEGER"
}
```


## task_blocked

Producer
Execution Service

Payload

```
{
  "phase_index": "INTEGER",
  "task_index": "INTEGER",
  "blocker_reason": "STRING"
}
```


## task_skipped

Producer
Execution Service

Payload

```
{
  "phase_index": "INTEGER",
  "task_index": "INTEGER"
}
```


## phase_completed

Producer
Execution Service

Payload

```
{
  "phase_index": "INTEGER"
}
```


## session_completed

Producer
Execution Service

Payload

```
{
  "instance_id": "UUID"
}
```


## instance_stalled

Producer
Pattern Service

Payload

```
{
  "stall_window_hours": "INTEGER"
}
```


# Lifecycle Events

## system_generated

Producer
Generation Service

Payload

```
{
  "system_id": "UUID",
  "version_id": "UUID"
}
```


## system_activated

Producer
Orchestrator

Payload

```
{
  "system_id": "UUID"
}
```


## system_adapted

Producer
Adaptation Service

Payload

```
{
  "system_id": "UUID",
  "new_version_id": "UUID"
}
```


## instance_migration_requested

Producer
Orchestrator

Payload

```
{
  "instance_id": "UUID",
  "target_version_id": "UUID"
}
```


## instance_migration_accepted

Producer
Orchestrator

Payload

```
{
  "instance_id": "UUID"
}
```


## instance_migrated

Producer
Orchestrator

Payload

```
{
  "instance_id": "UUID",
  "version_id": "UUID"
}
```


## system_completed

Producer
Execution Service

Payload

```
{
  "instance_id": "UUID"
}
```


# Ecosystem Events

## system_published

Producer
Publication Service

Payload

```
{
  "system_id": "UUID",
  "public_system_id": "UUID"
}
```


## system_forked

Producer
Publication Service

Payload

```
{
  "source_public_system_id": "UUID",
  "forked_system_id": "UUID"
}
```


## system_remixed

Producer
Generation Service

Payload

```
{
  "source_public_system_id": "UUID",
  "new_system_id": "UUID"
}
```


# Proof Events

## proof_submitted

Producer
Execution API

Payload

```
{
  "proof_id": "UUID",
  "instance_id": "UUID"
}
```


## proof_verified

Producer
Curator Service

Payload

```
{
  "proof_id": "UUID"
}
```


## proof_rejected

Producer
Curator Service

Payload

```
{
  "proof_id": "UUID"
}
```


# Event Immutability

Events must follow these rules:

1. append-only
2. immutable
3. never updated
4. never deleted

Corrections must be represented using **compensating events**.

Example:

```
task_completed
task_completion_reversed
```


# Consumer Idempotency

Event consumers must be idempotent.

Workers must store:

```
last_processed_event_id
```

Reprocessing the same event must not produce inconsistent projections.


# Event Replay Guarantee

Execution sessions must be replayable using:

```
(session_id, event_sequence)
```

Replay must reconstruct:

* task completion
* phase progression
* session lifecycle
* execution state


# Schema Evolution

Every event includes:

```
schema_version
```

Rules:

* additive schema changes only
* new fields optional
* breaking changes require new event type


# Derived Data Rules

Derived metrics must never mutate core entities.

Examples:

* progress_percent
* execution_streak
* system_success_rate
* ranking_score
* stalled_status

Derived values must be computed asynchronously from events.


# Final Runtime Contract

Execution OS runtime telemetry guarantees:

* append-only history
* deterministic replay
* strict producer authority
* schema versioning
* idempotent consumption

This event catalog defines the **immutable execution log contract** for the platform.
