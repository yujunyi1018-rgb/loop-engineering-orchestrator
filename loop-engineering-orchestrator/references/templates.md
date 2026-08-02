# Templates

Copy and adapt these when producing project artifacts. Adapt names and language to the existing project; preserve approved paths when revising an active project.

## Document map (directory structure)

```text
project/
├─ README.md                        # new-conversation entry and task routing
├─ project-map.md                   # directory responsibilities and authority
├─ governance/
│  ├─ charter.md
│  ├─ glossary.md
│  ├─ decisions/                    # ADRs and major decisions
│  └─ orchestration/                # coordinator rules, trigger and wave specs
├─ product/
│  ├─ prd/
│  ├─ rules/
│  ├─ templates/
│  └─ examples/
├─ design/
│  ├─ architecture/
│  ├─ data-and-api/
│  ├─ security/
│  └─ implementation-plans/
├─ implementation/
│  ├─ frontend/
│  ├─ backend/
│  ├─ infrastructure/
│  └─ tests/
├─ validation-and-operations/
│  ├─ acceptance/
│  ├─ reports/                      # orchestration state and progress ledgers
│  └─ operations/
├─ project-memory/
│  ├─ current-context.md
│  ├─ work-log/
│  └─ snapshots/
└─ archive/                         # superseded, trace-only material

project-support/                    # outside the repository
├─ test-data/
├─ run-evidence/
└─ learning-and-retrospectives/
```

## Role contract

```text
Role name
Purpose
Required inputs
Allowed write scope
Forbidden scope
Expected deliverables
Acceptance criteria
Receipt format
Escalation and stop conditions
Upstream and downstream roles
```

## Task contract

```text
Task ID: <task-id>
Target role and thread: <role / thread-id>
Input snapshot: <snapshot-id and hashes>
Required reading: <absolute paths>
Write lease: <lease-id and exact paths>
Forbidden scope: <paths and actions>
Deliverables: <specific files or behavior>
Minimal verification: <success path and rejection or boundary path>
Acceptance criteria: <objective checks>
Expected receipt: DELIVERED | BLOCKED | INPUT_STALE
Human gates: <decisions that must stop execution>
```

## Standard receipt

```text
Receipt type: DELIVERED | BLOCKED | INPUT_STALE
Task ID: <task-id>
Input snapshot: <snapshot-id / SHA-256>
Deliverable snapshot: <snapshot-id / SHA-256>
Changed files: <absolute paths>
Commands and results: <reproducible evidence>
Acceptance evidence: <checks performed>
Actual write scope: <paths>
Uncovered risks: <explicit list>
Blockers: <none or precise items>
```

## Orchestration state record (per task)

```text
Task ID
Requirement or milestone
Status
Responsible role
Thread ID
Input snapshot
Deliverable snapshot
Write lease
Dispatch time
Last activity
Receipt source and cursor
Acceptance evidence
Known risks
Next eligible task
Blocker or human gate
```

Keep orchestration state separate from product and architecture authority. State describes what is happening; it must not silently redefine what should be built.

## Automation status record (per configured trigger)

```text
Automation ID
Type (heartbeat / file-watch / other)
Interval or condition
Target thread or conversation
Current status: ACTIVE | PAUSED | DISABLED
Last run
Next expected run
```

A loop with a complete task-record history but a paused or disabled automation is not currently self-advancing — say so plainly rather than describing it as autonomous.
