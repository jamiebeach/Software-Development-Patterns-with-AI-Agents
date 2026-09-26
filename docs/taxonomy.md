# Taxonomy of AI Agent Software Engineering Patterns

A foundational classification framework for categorizing how AI agents collaborate, execute, and interface with human developers and codebases.

---

## 1. Classification Dimensions

Every agentic pattern in this repository is evaluated across five fundamental operational axes:

```text
              Autonomy Spectrum [L0 -> L4]
                           ▲
                           │
Coupling & Graph Topology ─┼─ State, Memory & Policy Persistence
                           │
                           ▼
        Verification Rigor [Human Visual -> Formal Kernel]
```

1. **Autonomy Spectrum ($L_0 \rightarrow L_4$):** How much initiative and decision-making authority the agent holds before requiring human intervention.
2. **Coupling & Graph Topology:** The organizational structure of execution—from isolated solo loops to cohesion-partitioned swarms and enterprise meshes.
3. **State, Memory & Policy Persistence:** How context, working memory, architectural laws, and task state survive across session boundaries.
4. **Verification & Feedback Rigor:** The deterministic mechanism through which an agent validates both new behavior (Fail-to-Pass) and system stability (Pass-to-Pass).
5. **Context Ingestion Strategy:** How the codebase is represented and retrieved (e.g., file-tree dumping, scoped localization, AST knowledge graphs).

---

## 2. The Autonomy Spectrum ($L_0 \rightarrow L_4$)

| Level | Designation | Human Role | Agent Role | Execution Boundary | Exemplar |
| :--- | :--- | :--- | :--- | :--- | :--- |
| $L_0$ | **Inline Auto-complete** | Direct author; reviews character-by-character | Predicts next tokens / line-level completion | Single file, cursor position | GitHub Copilot (ghost text) |
| $L_1$ | **Interactive REPL / Vibe** | Pilot; steering conversation in continuous prompt loops | Generates diffs on request; suggests commands | Single module or visual component | Cursor Chat, v0, Lovable |
| $L_2$ | **Contract-Bounded Executor** | Intent Architect; drafts specifications and validates plans | Autonomous implementation against strict test/spec boundaries | Multi-file feature branch | Spec-Driven Dev (Spec Kit, Kiro, Claude Code) |
| $L_3$ | **Asynchronous Swarm Worker** | Fleet Coordinator; sets goals and resolves escalated conflicts | Plans sub-tasks, claims issues from DAG, resolves dependencies | Branch per issue; git worktrees | Beads, Steve Yegge's Gas Town |
| $L_4$ | **Full Repository Custodian** | Outcome Auditor; monitors metrics, invariants, and budget | Discovers bugs, upgrades dependencies, re-architects subsystems | Full repository, CI/CD pipelines | Cognition Devin, Blitzy, Factory |

---

## 3. Organizational & Collaboration Topologies

### 3.1. Topology A: Single Developer + Local Companion ($1:1$ Synchronous)
* **Mechanics:** The human drives the editor; the agent operates synchronously within the active context window.
* **Failure Mode:** Context saturation, lack of long-horizon tracking, human fatigue from constant prompt iteration.
* **Primary Patterns:** Vibe Coding, Self-Healing REPL.

```text
[ Human Developer ] <====== (Chat / Inline Prompt) ======> [ Local Agent ]
         │                                                        │
         └───────────── Modifies Working Tree ────────────────────┘
```

### 3.2. Topology B: Single Developer + Multi-Agent Swarm ($1:N$ Asynchronous)
* **Mechanics:** The human defines a high-level goal. A supervisory agent breaks work into a Directed Acyclic Graph (DAG) of micro-tasks. Ephemeral worker agents spin up in isolated git worktrees or containers, execute individual tasks, and submit PRs.
* **Empirical Constraint (Cohesion-Aware Partitioning):** As demonstrated in `Co-Coder` (`arXiv:2606.00953`), modelling a codebase as a coupling graph $G = (V, E)$ shows that assigning tightly coupled files to parallel agents causes severe token thrashing and merge conflicts. Effective $1:N$ swarms require **Structural Hub Isolation** (modifying shared interfaces sequentially in Phase 0) before fanning out parallel workers across decoupled module clusters.
* **Primary Patterns:** Issue-Driven Agent Mesh (Beads), Hierarchical Supervisor (Gas Town).

```text
                            [ Human Lead ]
                                  │
                          (Defines Goals/Specs)
                                  ▼
                        [ Supervisor / Mayor ]
                                  │
                     [Phase 0: Sequential Hub Edit]
                                  │
                       /──────────┼──────────\
             (Dispatches)   (Dispatches)   (Dispatches)
                   ▼              ▼              ▼
              [ Worker A ]   [ Worker B ]   [ Worker C ]
             (Module Cluster) (Module Cluster) (Module Cluster)
                   │              │              │
                   └───────► [ Critic/CI ] ◄─────┘
```

### 3.3. Topology C: Multi-Developer + Multi-Agent Enterprise ($M:N$ Mesh)
* **Mechanics:** Human engineers and AI agents are peers in the issue tracker. Agents triage incoming alerts, attempt bug reproduction, prepare draft PRs, and run automated architectural evaluations.
* **Failure Mode:** Noise pollution in pull requests, regression snowballs across milestones (`EvoClaw`), degradation of codebase idioms.
* **Primary Patterns:** Knowledge-Graph Grounding (Blitzy), Spec-Anchored Governance.

---

## 4. State, Memory & Policy Persistence

How agents remember past decisions, obey repository laws, and survive context resets:

### 4.1. In-Context Ephemeral (Session Window)
* Context is stored exclusively in the LLM conversational message history.
* **Trade-off:** Fast to iterate, but brittle. Context degrades rapidly beyond $50\text{--}100$ turns.

### 4.2. Operational Policy & Constitutional Files (`AGENTS.md` & `constitution.md`)
* Repository-level operational mechanics (`AGENTS.md`) and immutable architectural invariants (`constitution.md`) are auto-injected at the start of every session.
* **Trade-off:** Eliminates repetitive prompt engineering and prevents destructive recovery loops, provided instructions are written as deterministic CLI commands rather than narrative prose.

### 4.3. Specification Maturity Tiers (`specs/`)
* **Spec-First:** Ephemeral specs used to guide initial generation, then discarded.
* **Spec-Anchored:** Living specification files maintained in lockstep with code alongside `constitution.md`.
* **Spec-as-Source:** Specifications serve as the primary human-authored artifact; application code is treated as a regenerable build output.

### 4.4. File-Backed / Git-Persisted Task Graphs (The "Beads" Paradigm)
* Tasks, dependencies, blockers, and completed execution logs are committed into git as discrete JSONL or Dolt records (`.beads/`).
* **Trade-off:** Completely portable across session restarts; prevents the "50 First Dates" amnesia problem.

### 4.5. External Knowledge Graphs & AST Indexes
* The repository is parsed ahead-of-time into an Abstract Syntax Tree (AST), call graph, and relational database. Agents query interface shapes and call hierarchies via custom tools.
* **Trade-off:** Scales to enterprise codebases ($10\text{M+}$ LOC); requires indexing infrastructure.

---

## 5. Verification & Safety Guardrails

Empirical benchmarks (`EvoClaw`, `arXiv:2603.13428`) show that agent accuracy drops from $>80\%$ on isolated patches to $\le 38\%$ over continuous multi-milestone evolution due to accumulating regressions. Every agentic pattern must define its verification boundary across five progressive gates:

```text
+-----------------------------------------------------------------------------------+
|                              VERIFICATION GATES                                   |
+-----------------------------------------------------------------------------------+
| Gate 1: Static & Syntactic  | Typecheckers (tsc/mypy), compilers, AST linters     |
| Gate 2: Dual Behavioral     | Fail-to-Pass (F2P) feature tests +                  |
|                             | Pass-to-Pass (P2P) continuous regression suites     |
| Gate 3: Constitutional      | Layer isolation checks, read-only test harnesses    |
| Gate 4: Formal / Kernel     | Proof-carrying harnesses (Lean 4, Coq, Dafny, TLA+) |
| Gate 5: Human Outcome Audit | Spec/Plan sign-off, provenance-backed PR review     |
+-----------------------------------------------------------------------------------+
```

* **Weak Gate (Vibe / $L_1$):** Visual inspection by the human in a browser preview or simulator.
* **Medium Gate (SDD / $L_2$):** Read-only F2P and P2P test suites + CLI closure gates in `AGENTS.md` requiring exit code `0`.
* **Strong Gate (Swarm & Enterprise / $L_3\text{--}L_4$):** Dual-agent adversarial verification (Generator + Critic Witness) or formal verification kernels (`Aria` / `MAGS`) where unsound implementations are mathematically rejected before a human ever reviews the PR.