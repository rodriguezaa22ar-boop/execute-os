# Execution OS — State Transition Matrix (v1)

This document defines the canonical lifecycle transitions for the Execution OS runtime.

All lifecycle state mutations **must be committed by the Orchestrator**.

Services may only **propose transitions**.

---

# ExecutionSystem Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| draft | GenerateExecutionSystem | Generation Service | Orchestrator | tenant valid, goal defined | create first version | generated |
| generated | ActivateSystem | Orchestrator | Orchestrator | version exists | initialize runtime | active |
| active | PauseSystem | Execution Service | Orchestrator | system active | pause instances | paused |
| paused | ResumeSystem | Execution Service | Orchestrator | system paused | resume instances | active |
| active | SystemBlocked | Execution Service | Orchestrator | repeated blockers | record blocker | blocked |
| active | SystemCompleted | Execution Service | Orchestrator | phases complete | emit system_completed | completed |
| active | AdaptationApproved | Adaptation Service | Orchestrator | proposal approved | create new version | active |
| any | ArchiveSystem | Orchestrator | Orchestrator | owner permission | archive system | archived |

---

# ExecutionInstance Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| ready | CreateExecutionSession | Execution Service | Orchestrator | valid instance | create session | ready |
| ready | ActivateExecutionSession | Execution Service | Orchestrator | session started | mark running | running |
| running | TaskBlockedThreshold | Execution Service | Orchestrator | repeated blockers | emit blocked signal | blocked |
| running | StallDetected | Pattern Service | Orchestrator | inactivity threshold | mark stalled | stalled |
| blocked | ResumeExecution | Execution Service | Orchestrator | blocker resolved | reopen session | running |
| stalled | ResumeExecution | Execution Service | Orchestrator | new session started | emit recovery | running |
| running | PhaseCompleted | Execution Service | Orchestrator | tasks completed | increment phase pointer | running |
| running | SystemCompleted | Execution Service | Orchestrator | final phase done | finalize execution | completed |
| running | AbandonSystem | User | Orchestrator | owner confirmation | close sessions | abandoned |

---

# ExecutionSession Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| started | ActivateSession | Execution Service | Orchestrator | instance running | emit session_started | active |
| active | TaskCompleted | Execution Service | Orchestrator | valid task | append event | active |
| active | TaskBlocked | Execution Service | Orchestrator | blocker recorded | append event | active |
| active | TaskSkipped | Execution Service | Orchestrator | valid task | append event | active |
| active | PhaseCompleted | Execution Service | Orchestrator | tasks finished | append event | active |
| active | SessionComplete | Execution Service | Orchestrator | final task | append event | completed |
| active | SessionInterrupted | Execution Service | Orchestrator | inactivity timeout | emit interruption | interrupted |
| active | SessionFailed | Execution Service | Orchestrator | system error | emit failure | failed |

---

# PublicSystem Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| private | PublishSystem | Publication Service | Orchestrator | completed system | create public artifact | published |
| published | ArchivePublicSystem | Publication Service | Orchestrator | creator permission | remove from index | archived |

---

# Proof Verification Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| self_reported | SubmitProof | User | Orchestrator | valid instance | append proof | pending_review |
| pending_review | VerifyProof | Curator | Orchestrator | verification rules | update ranking | verified |
| pending_review | RejectProof | Curator | Orchestrator | moderation rules | flag proof | rejected |

---

# Instance Migration Lifecycle

| Current State | Trigger | Proposed By | Committed By | Validations | Side Effects | Next State |
|---|---|---|---|---|---|---|
| no_update | NewVersionCreated | Adaptation Service | Orchestrator | structural change detected | notify owner | update_available |
| update_available | SuggestMigration | Orchestrator | Orchestrator | compatible version | notify user | migration_suggested |
| migration_suggested | AcceptMigration | User | Orchestrator | owner approval | update version pointer | migration_accepted |
| migration_accepted | MigrationComplete | Orchestrator | Orchestrator | validation passed | emit migration event | migration_completed |

---

# Global Runtime Rules

- Only **Orchestrator commits state transitions**
- Services may **propose transitions only**
- All transitions emit **immutable events**
- Events are **append-only**
- Structural changes require **new system version**
- Publishing is a **projection, not a system mutation**
