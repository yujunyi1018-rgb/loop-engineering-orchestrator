# Auditing and diagnosing the loop

Read this file when a loop is failing, stalling, duplicating work, or drifting silently, and when judging whether a loop can be called established or healthy.

## Failure states

Use explicit states rather than ambiguous prose. The states below are task-level receipt outcomes; `ACCEPTED` is a separate, requirement-level status the coordinator assigns only after unified testing (see `run-loop.md` § 5), so a table full of `DELIVERED` tasks should never be read as an `ACCEPTED` requirement.

| State | Meaning | Required action |
| --- | --- | --- |
| `DELIVERED` | Agent claims a complete delivery | Independently verify before acceptance |
| `BLOCKED` | Agent cannot continue for a specific reason | Preserve lease, diagnose, reroute, or request authority |
| `INPUT_STALE` | Assigned input changed | Reject old downstream use and create a fresh assignment |
| `RECEIPT_INCOMPLETE` | Final message lacks required evidence | Ask the original Agent to complete the receipt |
| `RECEIPT_MISSING` | Completed thread has no standard receipt | Keep the task unaccepted and request a receipt |
| `RECEIPT_DELIVERY_DEGRADED` | Repeated reconciliation cannot read reliable progress | Stop downstream work and notify the user |
| `LEASE_CONFLICT` | Two agents' write scopes overlap or collide | Freeze the affected tasks, redraw lease boundaries, or escalate to a human decision |

## Diagnosing a stalled or misbehaving loop

Work through the causes in this order — each maps to a specific repair rather than a restart:

1. **Nothing is advancing.** Check the automation status record first: a paused or disabled heartbeat means the loop only advances on direct receipts or manual dispatch. Then check for tasks stuck in `BLOCKED` with no owner acting on the blocker, and leases held past any reasonable activity window.
2. **Work is duplicated.** Check snapshot idempotency: the same task and input snapshot must not be processed twice, and delivery events must include the deliverable snapshot in their idempotency key.
3. **Files are being overwritten.** Check for overlapping write leases (`LEASE_CONFLICT`) and for agents writing outside their leased scope; compare receipts' reported write scope against actual file changes.
4. **Results look stale or inconsistent.** Compare current input hashes against the snapshots tasks were dispatched with; input drift that was never marked `INPUT_STALE` invalidates downstream acceptance.
5. **The coordinator keeps waking itself.** Check that coordinator-written state, project memory, reports, archives, Git internals, and automation caches are excluded from file-change triggers.
6. **Progress claims don't match reality.** Re-run the acceptance gate on recent `DELIVERED` tasks: verify hashes, file lists, and reproducible commands rather than trusting the receipts. Check whether requirement-level status was promoted to `ACCEPTED` without unified testing.

## Completion checklist

Before claiming that a loop is established or healthy, confirm:

- the PRD and scope are explicit;
- new conversations have a deterministic reading order;
- every topic has a current authority;
- every role has a clear write scope and acceptance contract;
- the coordinator is separated from execution roles;
- trigger sources and exclusions prevent self-triggering;
- snapshots and idempotency prevent duplicate or stale work;
- leases prevent overlapping writes;
- direct receipts and heartbeat reconciliation use the same acceptance gate;
- transient materials cannot pollute the official repository;
- human decision gates are explicit;
- accepted tasks create recoverable state and enable a defined next action;
- the design states plainly which parts are currently active versus merely designed (e.g., whether triggers are `ACTIVE` or `PAUSED`, whether roles exist only inside conversation prompts or in a formal registry, whether the coordinator is a standing service or a conversation-plus-file mechanism);
- a short, current list of known real-world limitations accompanies any claim that the loop is established or healthy.

If any item is missing, report the gap and the smallest safe correction instead of describing the loop as complete.

## Per-round health questions

In steady-state operation, every dispatch round should be able to answer the ten readiness questions in `run-loop.md` § 2. A round that cannot answer them is itself a diagnostic finding: name the missing answer as the gap.
