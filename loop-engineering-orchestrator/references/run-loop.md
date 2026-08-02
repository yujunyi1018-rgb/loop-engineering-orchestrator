# Running the loop

Read this file when dispatching tasks, processing receipts, accepting or rejecting deliveries, or closing tasks. Templates for the task contract, receipt, and state records are in `templates.md`; failure states and their required actions are in `audit-and-diagnosis.md`.

## 1. Create a stable input snapshot

A snapshot should include:

- task identifier;
- required input files;
- content hashes;
- last modification information when useful;
- upstream acceptance status;
- decisions or unresolved gates that affect the task.

Deduplicate by task and input snapshot. For delivery events, include the deliverable snapshot as part of the idempotency key.

If a required input changes during execution, mark the task `INPUT_STALE`. Do not accept the old result into downstream work unless a fresh comparison proves the change is irrelevant and the project rules explicitly allow that determination.

## 2. Check readiness before dispatch

Before writing the contract, confirm the answer to each of these:

- What is the current authoritative input?
- What is the input snapshot?
- Who owns the task, and under which write lease?
- Which directory or scope may it write to?
- Are the prerequisite tasks accepted?
- What evidence will prove completion?
- How does the task stop if its input changes?
- Who receives a `BLOCKED` report?
- Which downstream task does acceptance unlock?
- Can the result be rolled back through file history or Git?

If any question lacks a clear answer, do not dispatch the task.

## 3. Dispatch a complete task contract

Fill the task-contract template in `templates.md`. An Agent must not infer authorization from a directory name or an old conversation. The current task contract is the execution authority.

## 4. Control parallel waves

Dispatch tasks in the same wave only when:

- all required upstream tasks are accepted;
- they reference the same stable input baseline;
- there is no unfinished read-after-write dependency between them;
- write leases and logical ownership do not overlap;
- each task has independent acceptance and failure handling.

Use a convergence barrier before integration, end-to-end testing, release, or any task that depends on multiple branches.

Freeze only the affected branch when one parallel member fails, unless shared input or write-scope contamination makes the whole wave unsafe.

## 5. Require scoped execution and proportional verification

An execution Agent should:

- reread the assigned authority and snapshot before editing;
- change only the leased scope;
- preserve unrelated user changes;
- avoid inventing unresolved product, security, permission, review, export, or schema semantics;
- run verification proportional to the task and current project phase;
- keep transient run artifacts outside the official repository.

For implementation-first phases, require a minimal reproducible smoke test:

- one critical success path;
- one critical rejection or boundary path.

Record the result as “implementation delivered, pending unified testing” when full integration or acceptance is intentionally deferred. Do not mark the complete requirement accepted early.

Keep the two acceptance tiers distinct in vocabulary as well as in state: a task's receipt can be `DELIVERED` once its own minimal verification passes, but the PRD requirement it belongs to should only move to `ACCEPTED` after every constituent task has passed the unified, full-chain test defined for that requirement. Record both tiers in orchestration state so a fully `DELIVERED` set of tasks is never read as an `ACCEPTED` requirement.

## 6. Require a standard receipt

Every task ends with the receipt template in `templates.md`. The receipt is an event and evidence index, not proof by itself.

## 7. Independently accept or reject delivery

Verify:

- task and active lease match;
- input snapshot is still current;
- actual changes remain inside the lease;
- reported files and actual files match;
- deliverable hashes match;
- commands and tests are reproducible;
- product, security, and acceptance invariants remain intact;
- uncovered risks are recorded at the correct project level.

Use the same gate for direct and reconciled receipts. A completed thread with incomplete evidence becomes `RECEIPT_INCOMPLETE` or `RECEIPT_MISSING`, not accepted.

## 8. Close the task and continue the loop

After acceptance:

- update the orchestration state;
- update current project context and the work log;
- create a scoped local Git checkpoint containing only the artifacts covered by this task's accepted write lease, excluding any other uncommitted changes, when the project authorizes it;
- record known gaps and deferred verification;
- release the write lease;
- dispatch newly eligible downstream tasks.

When acceptance fails, keep the lease held and do not start downstream tasks that depend on the result.
