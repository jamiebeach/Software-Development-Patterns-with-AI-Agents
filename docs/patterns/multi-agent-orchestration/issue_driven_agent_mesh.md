# Pattern: Issue-Driven Agent Mesh (Git-Tracked Task Units)

* **Category:** Multi-Agent Orchestration / High Autonomy
* **Maturity Level:** Production / Rapid Adoption
* **Primary Actors:** Human Architect / Overseer, Supervisory / Dispatcher Agent ("Mayor"), Ephemeral Worker Agents ("Polecats"), Reviewer / Critic Agents ("Witnesses")
* **Key Implementations:** Steve Yegge's [Beads (`bd`)](https://github.com/gastownhall/gastown), Dolt / Git-backed task graphs, MCP Agent Mail

---

## 1. Intent & Overview

The **Issue-Driven Agent Mesh** (commonly known as the **Beads Pattern**) treats discrete software tasks not as transient chat prompts or fragile markdown checklists, but as **structured, machine-queryable records stored directly within version control**.

Instead of feeding an agent an entire project plan inside its prompt context, the system externalizes project state into a git-tracked directed acyclic graph (DAG) of atomic task units ("beads"). Ephemeral coding agents query the graph deterministically, lock an unblocked task, execute within an isolated git worktree or container, commit code alongside the updated task status, and terminate.

```
       +-------------------------------------------------------------+
       |               PROJECT REPOSITORY (.git / .beads)            |
       |  Atomic Issues (JSONL / Dolt) with Dependency Edges (A -> B) |
       +-------------------------------------------------------------+
                        │                            ▲
                 [Query Unblocked]              [Commit Diff +
                  bd ready --json                Task Status]
                        │                            │
                        ▼                            │
         +-----------------------------+             │
         |     DISPATCH / SUPERVISOR   |             │
         |        ("The Mayor")        |             │
         +-----------------------------+             │
             /          │            \               │
   (Spawn)  /   (Spawn) │     (Spawn) \              │
           ▼            ▼              ▼             │
    [ Worker 1 ]   [ Worker 2 ]   [ Worker 3 ]       │
    (Worktree A)   (Worktree B)   (Worktree C)       │
    Task: bd-101   Task: bd-104   Task: bd-105       │
           \            │              /             │
            \           │             /              │
             +----------┴─────────────┴──────────────+
```

---

## 2. The Problem It Solves

Multi-agent software engineering without a structured, machine-first ledger suffers from three catastrophic failures:

1. **The "50 First Dates" Problem (Memory Amnesia):** When LLM context limits compact or when sessions restart, agents lose situational awareness. They forget what was finished, re-implement solved logic, or drift off-course.
2. **Markdown Checklist Corruption:** Agents updating unstructured `TODO.md` files routinely delete lines, introduce parsing ambiguities, hallucinate completed states, and produce irreconcilable merge conflicts during parallel multi-agent runs.
3. **Context Window Burn on Dependency Graphs:** Asking an LLM to evaluate a 100-item backlog to decide "what to work on next" wastes thousands of tokens per turn and frequently yields incorrect topological ordering.

---

## 3. Core Principles

1. **Machine-First Ledger (Token Economy):** The task store CLI (`bd`) does the heavy graph math in compiled code (e.g., Go) rather than inside the LLM prompt. The agent only receives the pruned, ready-to-run tasks via a compact JSON payload (`bd ready --json`).
2. **Atomic Lifecycles ("Land the Plane and Kill the Session"):** Agents should not live across multiple unrelated tasks. Once a bead is implemented and tests pass, the agent commits the work, updates the bead status, and exits. A clean agent session is spawned for the next task.
3. **Git-Native / Distributed Sync:** Tasks live in the repository (e.g., as append-friendly `.jsonl` or version-controlled relational state like Dolt). Task state travels with branch checkouts, rebases, and PRs, ensuring synchronization between humans and swarms.
4. **Provenance & Audit Trails:** Every bead records origin metadata (which agent spawned it, which parent task or bug discovery triggered it, and which git commit closed it).

---

## 4. Architecture & Data Model

A task unit ("bead") is an immutable or append-tracked record structured specifically for agent parsing:

```json
{
  "id": "bd-8f3a",
  "title": "Add Lua script for atomic token decrement",
  "type": "task",
  "status": "in_progress",
  "priority": 1,
  "blocked_by": ["bd-102a"],
  "discovered_from": "bd-100f",
  "assignee": "agent-worker-04",
  "context": {
    "spec_path": "specs/SPEC-004-rate-limiter.md",
    "target_files": ["src/middleware/rate-limiter/lua.ts"]
  }
}
```

### Deterministic DAG Engine
Instead of dumping all issues into the prompt, the agent runtime calls:
```bash
bd ready --json
```
The binary runs a topological sort over the issue graph, verifies that all `blocked_by` IDs are `status: closed`, and returns only the unblocked candidates:

```json
[
  {
    "id": "bd-8f3a",
    "title": "Add Lua script for atomic token decrement",
    "status": "ready"
  }
]
```

---

## 5. Execution Workflow

```text
 1. PLAN / DECOMPOSE
    Human / Architect Agent breaks feature into atomic beads:
    $ bd create "Implement Redis Token Bucket" -t epic
    $ bd create "Lua Script" -t task --parent bd-01 --deps bd-00

 2. ISOLATE ENVIRONMENT
    Supervisor provisions a fresh Git worktree:
    $ git worktree add ../worktree-bd-8f3a -b feature/bd-8f3a

 3. CLAIM AND LOCK
    Worker launches inside worktree:
    $ bd update bd-8f3a --status in_progress --owner worker-01

 4. EXECUTE & VERIFY
    Worker edits code strictly within the task scope.
    Runs validation suite: $ npm test tests/token-bucket.spec.ts

 5. COMMIT & CLOSE
    Worker marks task done and commits both code and bead update:
    $ bd close bd-8f3a --reason "Lua script verified against mock Redis"
    $ git commit -am "feat: implement token bucket lua script (fixes bd-8f3a)"

 6. TEARDOWN
    Worker session exits. Worktree is merged or submitted as PR.
```

---

## 6. Real-World Multi-Agent Coordination

In advanced swarms (such as **Gas Town** paired with **MCP Agent Mail**):

* **The Mayor:** Monitors the overall bead graph, creates epics, spawns worker agents, and distributes unblocked beads.
* **The Polecats (Workers):** Run in isolated headless tmux panes or Docker sandboxes. They only see their assigned bead and immediate files.
* **Dynamic Discovery:** When an agent hits an unexpected edge case or discovers follow-up work that exceeds its 2-minute horizon, it does **not** try to solve it in the same session. Instead, it runs:
  ```bash
  bd create "Handle Redis cluster reconnect failover" --deps discovered-from:bd-8f3a
  ```
  It finishes its original task cleanly, and the newly discovered bead enters the backlog for future dispatch.
* **The Witness / Referee:** An independent agent that inspects the diff against the bead's original acceptance criteria, runs linter and unit tests, and gives final sign-off.

---

## 7. Anti-Patterns & Failure Modes

| Anti-Pattern | Manifestation | Remediation |
| :--- | :--- | :--- |
| **Monolithic Beads** | Creating beads that take >15 minutes or require 20+ file changes. | Enforce the *2-to-5 minute rule*: tasks must be decomposed until each bead represents an atomic interface or file change. |
| **Markdown Hallucination** | Keeping task lists in `TODO.md` or PR descriptions. | Replace markdown lists with CLI-gated JSONL or database-backed task engines (`bd`). |
| **Context Leaking Across Tasks** | Keeping the same agent session alive for 5 consecutive beads. | Terminate the agent process upon task closure; launch a fresh agent with fresh context for the next bead. |
| **Dangling Blockers** | Circular dependencies in the task graph ($A \rightarrow B \rightarrow A$). | Use a task CLI that checks for cycles at creation time and alerts the supervisory agent. |

---

## 8. Related Patterns

* **[Spec-Driven Development (SDD)](../team-and-enterprise/spec-driven-development.md):** The behavioral contracts and specs authored in SDD supply the raw material that is decomposed into beads.
* **[Hierarchical Supervisor (Gas Town)](../multi-agent-orchestration/hierarchical-supervisor.md):** The organizational model (Mayor, Workers, Witnesses) that operates directly on top of the beads task ledger.
* **[Bounded-Context Sandboxes](../reliability-and-state/bounded-context-sandbox.md):** Running each bead worker inside isolated Git worktrees and containers to prevent concurrent write collisions.