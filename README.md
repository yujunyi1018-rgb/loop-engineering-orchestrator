# Loop Engineering Orchestrator

**English** | [简体中文](README.zh-CN.md)

**An agent skill that turns multi-agent chaos into a controlled engineering loop.**

No code. No framework. No SDK. Just a set of markdown files that teach your AI agent how to run a document-driven, evidence-based, multi-agent project — with input snapshots, write leases, standard receipts, independent acceptance, and human decision gates.

Distilled from a real production project: a customer-service conversation annotation platform built end-to-end by multiple Codex agent conversations coordinated through this exact loop.

---

## The problem

Anyone who has tried running several AI agent conversations on one project has met these failure modes:

- Two agents edit the same file and silently overwrite each other.
- An agent says "done!" — but nothing verifiable was actually delivered.
- An agent works for an hour on a spec that changed ten minutes in.
- The same task gets done twice because nobody tracked what was dispatched.
- A "coordinator" agent starts writing business code itself and becomes both referee and player.
- Three documents all claim to be the "final" version.
- The project state lives in chat history, and dies when the chat closes.

More agents ≠ more progress. Without engineering constraints, more agents just means faster chaos.

## The idea

This skill does **not** try to make agents smarter. It makes the project **structurally unable to go wrong in the usual ways**, borrowing ideas that distributed systems solved decades ago:

| Failure mode | Mechanism |
| --- | --- |
| Stale input | **Immutable input snapshots** (SHA-256) — work stops with `INPUT_STALE` when the input drifts |
| Duplicate work | **Idempotency keys** (task + snapshot) — the same work is never dispatched twice |
| Overwritten files | **Write leases** — one controlled scope, one active writer, ever |
| Fake "done" claims | **Evidence-based acceptance** — hashes, file lists, and reproducible commands; natural language is never proof |
| Lost messages | **Event-first triggering with heartbeat reconciliation** — direct receipts preferred, polling as fallback, same acceptance gate for both |
| Runaway autonomy | **Human decision gates** — irreversible or semantic decisions always stop and ask |
| Chat-dependent state | **Files are authority; conversations are workers** — durable facts live in documents, not in context windows |

One sentence: **documents hold facts, roles hold responsibility, snapshots freeze inputs, leases isolate writes, evidence decides acceptance, and humans keep the final say.**

## Installation

This skill follows the standard `SKILL.md` agent-skill format.

**Claude Code** — copy the skill folder into your skills directory:

```bash
# personal (all projects)
git clone https://github.com/yujunyi1018-rgb/loop-engineering-orchestrator.git
cp -r loop-engineering-orchestrator/loop-engineering-orchestrator ~/.claude/skills/

# or project-level
cp -r loop-engineering-orchestrator/loop-engineering-orchestrator .claude/skills/
```

The path must be `~/.claude/skills/loop-engineering-orchestrator/SKILL.md` — not nested one level deeper. Start a new session and the skill is picked up automatically.

**Claude.ai** — upload the packaged `.skill` file (or a zip of the skill folder) under **Settings → Capabilities/Features → skills**.

**Codex CLI / other agents** — most agents that support the `SKILL.md` format read from `.agents/skills/` as a vendor-neutral path:

```bash
cp -r loop-engineering-orchestrator/loop-engineering-orchestrator .agents/skills/
```

## Usage

You don't invoke this skill with a command. It triggers on intent. Say things like:

> "Set up a project map and multi-agent roles for this PRD."
>
> "Design a coordinator agent that dispatches tasks and verifies deliveries."
>
> "My agents keep overwriting each other's files — audit this loop."
>
> "Run the next wave of tasks."

The skill routes each request to one of three operating modes and loads only the guidance it needs:

```
loop-engineering-orchestrator/
├─ SKILL.md                          # always loaded: core model, 9 invariants,
│                                    # mode routing, human decision gates
└─ references/                       # loaded on demand
   ├─ establish-loop.md              # designing/revising a loop: project contract,
   │                                 # document map, role contracts, coordinator, triggers
   ├─ run-loop.md                    # executing: snapshots, pre-dispatch checks,
   │                                 # task contracts, parallel waves, receipts, acceptance
   ├─ audit-and-diagnosis.md         # failure states, symptom→repair diagnosis,
   │                                 # completion checklist
   └─ templates.md                   # copy-adapt skeletons: directory tree, role contract,
                                     # task contract, receipt, state records
```

Safety-critical content (the nine invariants and the human decision gates) deliberately lives in the always-loaded `SKILL.md`, never in an on-demand file.

## How the loop works

Two nested loops:

**Establishment loop** (runs once, and on major revisions) — turns an idea into a project operating system:

```
User discussion → PRD → project map & authority structure
→ project memory & new-conversation entry → agent role contracts
→ coordinator & trigger design → executable baseline
```

**Execution loop** (runs continuously) — turns events into verified progress:

```
Input change or delivery event → stable snapshot → prerequisite check
→ task contract & write lease → scoped execution & minimal verification
→ standard receipt → independent acceptance → state update & local checkpoint
→ downstream dispatch
```

Before any task is dispatched, the coordinator must answer ten readiness questions (what is the authoritative input? who holds the lease? what evidence proves completion? can the result be rolled back? …). Any unanswered question blocks dispatch — ambiguity is resolved at the control plane, never pushed down for an execution agent to improvise.

Acceptance is two-tiered: a task can be `DELIVERED` after its own smoke test, but the requirement it belongs to only becomes `ACCEPTED` after full-chain unified testing. A board full of green `DELIVERED` tags is not a finished requirement, and the skill forces that distinction into the recorded state.

## What this skill is not

- **Not an autonomous "AI runs everything" system.** Major semantic and irreversible decisions always stop at a human gate. Autonomy never expands project authority.
- **Not a parallelism maximizer.** It explicitly optimizes for clear ownership and verifiable delivery, not for the largest number of simultaneous agents.
- **Not software.** There is nothing to run. The "orchestrator" is a coordinator conversation plus state files plus an optional heartbeat — the skill is honest about that boundary and requires the loop's own status records to be honest about it too (a paused heartbeat must be recorded as `PAUSED`, not described as "autonomous").

## Origin

This skill was extracted from a real project: an annotation platform for AI customer-service conversations, built by multiple Codex agent conversations (product rules, architecture, backend, frontend, infrastructure, testing) coordinated by a non-coding coordinator agent — with every mechanism here (quiet windows, lease conflicts, receipt reconciliation, two-tier acceptance) added because its absence caused a real failure first.

It is young. Expect rough edges. That's what the issues tab is for.

## Contributing & feedback

Feedback from real multi-agent projects is worth more than anything else right now. Especially useful:

- A failure mode you hit that the loop didn't catch → open an issue with the symptom.
- A gate that felt like bureaucracy in practice → open an issue; over-constraint is also a bug.
- Adaptations for other agents (Cursor, Gemini CLI, etc.) → PRs welcome.

## License

MIT
