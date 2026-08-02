# Establishing the loop

Read this file when designing a new loop, revising the project contract, defining roles, or configuring the coordinator and its triggers. Templates for every artifact named here are in `templates.md`.

## 1. Establish the project contract

Before implementation routing, ensure the project has:

- a PRD with goals, scope, non-scope, users, flows, requirements, risks, and acceptance;
- a project map defining directory responsibilities and authority;
- a root entry document telling new conversations what to read first;
- a current-context file containing current stage, confirmed decisions, active work, blockers, and next actions;
- a workspace boundary separating official artifacts from transient materials;
- a decision log for choices that affect later implementation.

If any part is missing, propose the smallest set needed to make downstream work unambiguous.

## 2. Build the document map

Use the directory structure in `templates.md` unless the project already has an approved alternative. Adapt names to the existing project language. Preserve existing approved paths when revising an active project.

New conversations must have a deterministic reading order, typically:

```text
README.md
→ project-memory/current-context.md
→ project-map.md
→ the authority files directly relevant to the current task
```

## 3. Define role contracts

Create roles from distinct responsibility domains, not from a desire for more parallelism. Every role contract must fill the role-contract template in `templates.md`.

Typical roles:

- product and business rules;
- architecture and security;
- implementation, test, and delivery planning;
- frontend implementation;
- backend implementation;
- infrastructure implementation;
- integration and end-to-end verification;
- coordinator or orchestrator.

Merge roles when the project is small and write ownership remains clear. Split roles when responsibilities or write scopes would otherwise conflict.

## 4. Define the coordinator as a control plane

The coordinator should:

- detect direct receipts and relevant input changes;
- wait for file stability when an input is being edited;
- create and deduplicate input snapshots;
- verify upstream acceptance gates;
- register tasks, threads, cursors, and write leases;
- dispatch complete task contracts;
- reconcile missing receipts from completed threads;
- independently verify delivery evidence;
- update orchestration state and project memory;
- create local recoverable checkpoints after acceptance when authorized;
- release leases and dispatch eligible downstream work.

The coordinator should not:

- write PRD, product rules, architecture, or business code;
- decide unresolved business semantics;
- infer acceptance from thread completion or file existence;
- modify another Agent's deliverable to make it pass;
- perform remote Git or external production actions without authority.

## 5. Design event-first triggering with a heartbeat fallback

Use direct delivery events as the preferred trigger:

- `DELIVERED` for completed work awaiting acceptance;
- `BLOCKED` for a specific impediment;
- `INPUT_STALE` when the assigned input changed during execution.

Use a heartbeat as a reliability fallback for:

- missing direct receipts;
- completed threads with unread final messages;
- stalled tasks;
- external file changes;
- expired or abandoned leases;
- input drift.

Record the current activation status of every configured trigger — `ACTIVE`, `PAUSED`, or `DISABLED` — using the automation status record in `templates.md`. A documented trigger design is not evidence that it is currently running: state the actual status separately, and treat a paused or disabled trigger as a loop that currently advances only on direct receipts or manual dispatch, not as a fully autonomous one.

When file changes are triggers, apply a quiet window before snapshotting. A simple local project can use two minutes; adjust only when editing patterns make another value clearly safer.

Exclude coordinator-written state, project memory, reports, archives, Git internals, and automation caches from file-change triggers to prevent self-trigger loops.

## 6. Draw the workspace boundary

Keep these in the official repository:

- source code and dependency manifests;
- configuration templates without secrets;
- product, architecture, API, security, and implementation documents;
- automated test source code;
- acceptance standards;
- project memory and formal decisions;
- reviewed, fixed, desensitized fixtures when explicitly allowed.

Keep these in a non-versioned support area:

- user test data and temporary synthetic data;
- run output, logs, screenshots, and browser evidence;
- debugging artifacts;
- learning notes, retrospectives, and experience summaries.

Automated test code is code and lives in the repository; the data, screenshots, and logs a test run produces are run material and live outside it. If a retrospective changes project scope, API, security, or acceptance, move the conclusion into the corresponding authority document while leaving raw process material in the support area.

## 7. Confirm before declaring the loop established

Run the completion checklist in `audit-and-diagnosis.md`. If any item is missing, report the gap and the smallest safe correction instead of describing the loop as complete.
