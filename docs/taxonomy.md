# Taxonomy of AI Agent Software Engineering Patterns

A foundational classification framework for categorizing how AI agents collaborate, execute, and interface with human developers and codebases.

---

## 1. Classification Dimensions

Every agentic pattern in this repository is evaluated across five fundamental operational axes:

```
        Autonomy Spectrum [L0 -> L4]
                     ▲
                     │
Coupling & Concurrency ──┼── State & Memory Persistence
                     │
                     ▼
           Verification Rigor [Human -> Gated CI]
```

1. **Autonomy Spectrum ($L_0 - L_4$):** How much initiative and decision-making authority the agent holds before requiring human intervention.
2. **Coupling & Concurrency:** The topology of execution—from isolated solo loops to hierarchical swarms and mesh networks.
3. **State & Memory Persistence:** How context, working memory, and operational state survive across session boundaries.
4. **Verification & Feedback Rigor:** The mechanism through which an agent validates that its changes are correct.
5. **Context Ingestion Strategy:** How the codebase is represented and retrieved (e.g., file-tree dumping, RAG, AST knowledge graphs).

---

## 2. The Autonomy Spectrum ($L_0 - L_4$)

| Level | Designation | Human Role | Agent Role | Execution Boundary | Exemplar |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **$L_0$** | **Inline Auto-complete** | Direct author; reviews character-by-character | Predicts next tokens / line-level completion | Single file, cursor position | GitHub Copilot (ghost text) |
| **$L_1$** | **Interactive REPL / Vibe** | Pilot; steering conversation in continuous prompt loops | Generates diffs on request; suggests commands | Single module or visual component | Cursor Chat, v0, Lovable |
| **$L_2$** | **Contract-Bounded Executor** | Architect; drafts specifications and validates PRs | Autonomous implementation against strict test/spec boundaries | Multi-file feature branch | Spec-Driven Dev (Aider, Claude Code) |
| **$L_3$** | **Asynchronous Swarm Worker** | Product owner / Supervisor; sets goals and resolves escalated conflicts | Plans sub-tasks, claims issues from DAG, resolves dependencies | Branch per issue; git worktrees | Beads, Steve Yegge's Gas Town |
| **$L_4$** | **Full Repository Custodian** | Stakeholder; monitors metrics and budget | Discovers bugs, upgrades dependencies, re-architects subsystems | Full repository, CI/CD pipelines | Cognition Devin, Factory Droids |

---

## 3. Organizational & Collaboration Topologies

### 3.1. Topology A: Single Developer + Local Companion (1:1 Synchronous)
* **Mechanics:** The human drives the editor; the agent operates synchronously within the active context window.
* **Failure Mode:** Context saturation, lack of long-horizon tracking, human fatigue from constant prompt iteration.
* **Primary Patterns:** Vibe Coding, Self-Healing REPL.

```
[ Human Developer ] <====== (Chat / Inline Prompt) ======> [ Local Agent ]
         │                                                        │
         └───────────── Modifies Working Tree ────────────────────┘
```

---

### 3.2. Topology B: Single Developer + Multi-Agent Swarm (1:N Asynchronous)
* **Mechanics:** The human defines a high-level goal. A supervisory agent breaks work into a Directed Acyclic Graph (DAG) of micro-tasks. Ephemeral worker agents spin up in isolated git worktrees or containers, execute individual tasks, and submit PRs.
* **Failure Mode:** Merge conflicts, duplicate work, cascading errors across dependent tasks.
* **Primary Patterns:** Issue-Driven Agent Mesh (Beads), Hierarchical Supervisor (Gas Town).

```
                            [ Human Lead ]
                                  │
                          (Defines Goals/PRs)
                                  ▼
                        [ Supervisor / Mayor ]
                       /          │           \
             (Dispatches)   (Dispatches)   (Dispatches)
                   ▼              ▼              ▼
              [ Worker A ]   [ Worker B ]   [ Worker C ]
             (Worktree 1)   (Worktree 2)   (Worktree 3)
                   │              │              │
                   └───────► [ Critic/CI ] ◄─────┘
```

---

### 3.3. Topology C: Multi-Developer + Multi-Agent Enterprise (M:N Mesh)
* **Mechanics:** Human engineers and AI agents are peers in the issue tracker. Agents triage incoming alerts, attempt bug reproduction, prepare draft PRs, and run automated architectural evaluations.
* **Failure Mode:** Noise pollution in pull requests, hallucinated architectural patterns, degradation of codebase idioms.
* **Primary Patterns:** Knowledge-Graph Grounding (Blitzy), Contract-Gated PR Reviews.

---

## 4. State & Memory Strategies

How agents remember past decisions, track progress, and avoid infinite retry loops:

### 4.1. In-Context Ephemeral (Session Window)
* Context is stored exclusively in the LLM conversational message history.
* **Trade-off:** Fast to iterate, but brittle. Context degrades rapidly beyond ~50–100 turns.

### 4.2. File-Backed / Git-Persisted State (The "Beads" Paradigm)
* Tasks, dependencies, blockers, and completed execution logs are committed into git as discrete markdown/JSON metadata (e.g., `.beads/`).
* **Trade-off:** Completely portable, surviving context window resets and agent restarts; requires disciplined schema design.

### 4.3. External Knowledge Graphs & AST Indexes
* The repository is parsed ahead-of-time into an Abstract Syntax Tree (AST), call graph, and relational database. Agents query interface shapes and call hierarchies via custom tools.
* **Trade-off:** Scales to enterprise codebases (10M+ LOC); requires indexing infrastructure.

---

## 5. Verification & Safety Guardrails

Every agentic development pattern must define its verification boundary:

```
+-------------------------------------------------------------------------------+
|                            VERIFICATION GATES                                 |
+-------------------------------------------------------------------------------+
| 1. Syntax / Static Checks | Typecheckers, compilers, formatters, linters      |
| 2. Behavioral Checks      | Unit, integration, and property-based test suites |
| 3. Contract Checks        | OpenAPI schema, invariant assertions, mock suites |
| 4. Security & Sandbox     | Docker/gVisor isolation, egress firewalls, secrets |
| 5. Human Sign-Off         | Final diff review, manual smoke testing           |
+-------------------------------------------------------------------------------+
```

* **Weak Gate (Vibe / L1):** Visual inspection by the human in a browser preview or simulator.
* **Medium Gate (SDD / L2):** Agent-driven feedback loop running test commands until returncode is `0`.
* **Strong Gate (Enterprise / L3-L4):** Dual-agent verification where a distinct Critic Agent evaluates the Execution Agent's diff against the original spec before committing.