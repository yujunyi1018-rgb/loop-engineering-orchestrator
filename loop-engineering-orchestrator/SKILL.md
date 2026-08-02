---
name: loop-engineering-orchestrator
description: Design, establish, audit, or run a document-driven Codex multi-agent project loop. Use this skill whenever the user mentions a project map, PRD-to-implementation workflow, multiple Agent roles or conversations, a coordinator or orchestrator Agent, scheduled heartbeats, file-change triggers, task routing, write leases, input snapshots, evidence-based acceptance, or asks Codex to keep a project moving autonomously. 中文场景包括“项目地图”“多 Agent/多智能体”“协调 Agent/中控 Agent”“自动任务编排”“定时任务/心跳”“回执”“写入租约”“输入快照”以及“让 Codex 自己跑通这个 Loop”。Also use it when an existing multi-agent loop is duplicating work, overwriting files, losing receipts, using stale inputs, or needs to separate code, test data, evidence, and project memory.
---

# Loop Engineering Orchestrator

## Goal

Turn a project from a collection of chats into a controlled engineering loop in which:

- files hold durable project facts;
- every task has an explicit owner and stable input;
- parallel work has non-overlapping write scopes;
- completion requires reproducible evidence;
- stale input, missing receipts, conflicts, and human decisions stop unsafe downstream work;
- accepted work updates project state and enables the next task.

Do not optimize for the largest number of simultaneous Agents. Optimize for clear ownership, stable inputs, verifiable delivery, and recoverable state.

## Core model

Separate the system into two loops.

### Project establishment loop

```text
User discussion
→ PRD
→ project map and authority structure
→ project memory and new-conversation entry
→ Agent role contracts
→ coordinator and trigger design
→ executable project baseline
```

### Task execution loop

```text
Input change or delivery event
→ stable snapshot
→ prerequisite check
→ task contract and write lease
→ scoped execution and minimal verification
→ standard receipt
→ independent acceptance
→ state update and local checkpoint
→ downstream dispatch
```

## Invariants

Preserve these invariants in every design and execution decision:

1. **Files are authority; conversations are workers.** Important conclusions must be written to the correct project file.
2. **One topic has one current authority.** Do not create competing “latest” or “final” documents.
3. **One controlled write scope has one active writer.** Parallel tasks need disjoint leases.
4. **A task starts from an immutable input snapshot.** Detect and stop stale work.
5. **A natural-language claim is not acceptance evidence.** Verify files, hashes, commands, tests, and scope.
6. **The coordinator controls flow but does not produce business deliverables.** Keep control-plane and execution-plane responsibilities separate.
7. **Major semantic or irreversible decisions remain human gates.** Autonomy does not expand project authority.
8. **Code and official artifacts remain clean.** Keep transient data, logs, screenshots, and learning notes outside the code repository unless the project contract explicitly says otherwise.
9. **Every document declares its own authority scope, including what it is not.** A method note, retrospective, or status summary must state plainly that it is not the PRD, not the code specification, and not the orchestration state source, so a later conversation cannot silently promote it into an authority it was never meant to hold.

## Determine the operating mode, then read the matching reference

First inspect the existing project: read the entry files, current context, PRD, project map, governance rules, recent work log, role definitions, and orchestration state. Use the smallest sufficient context set — do not recursively read every file when the project already defines a routing order. Classify what you find as confirmed current authority, current execution state, historical or replaced material, temporary evidence, or an unresolved human decision. If documents conflict, prefer the explicitly confirmed, newer authority and record the resolution rather than silently blending versions.

Then identify what the user wants and load only the reference you need:

| User intent | Read |
| --- | --- |
| Design a new loop, revise the project contract, define roles, set up the coordinator or triggers | `references/establish-loop.md` |
| Execute the next wave, dispatch tasks, process receipts, accept or reject deliveries, close tasks | `references/run-loop.md` |
| Diagnose a failed, stalled, duplicated, or silently-drifting loop; judge whether a loop is healthy or complete | `references/audit-and-diagnosis.md` |
| Produce any project artifact from a standard structure (document map, role contract, task contract, receipt, state record) | `references/templates.md` |

Establishing a loop and auditing one usually also need `templates.md`; read it alongside. Understanding or explaining the loop generally needs only this file.

Do not create threads or automations merely because they would be useful. Create or modify them only when the user explicitly asks for that action.

## Human decision gates

These apply in every mode. Stop and request direction for:

- PRD or acceptance-scope changes;
- final business labels or irreversible schema;
- final review, rework, relabeling, or export semantics;
- permission exceptions or security-baseline reductions;
- real or sensitive data;
- external accounts, production systems, or paid resources;
- irreversible deletion;
- actions that cannot be recovered through file history or Git.

Do not escalate routine, reversible engineering choices that fit the accepted contract. Never push, deploy, create external accounts, spend money, or handle real data unless the user has authorized those actions.

## Reference files

- `references/establish-loop.md` — Project establishment: project contract, document map, role contracts, coordinator design, trigger design, workspace separation.
- `references/run-loop.md` — Task execution: snapshots, pre-dispatch checks, task contracts, parallel waves, scoped execution, receipts, independent acceptance, task closure.
- `references/audit-and-diagnosis.md` — Failure states, health checks, the completion checklist, and the active-versus-designed distinction.
- `references/templates.md` — Copy-adapt templates for every standard artifact this skill produces.
