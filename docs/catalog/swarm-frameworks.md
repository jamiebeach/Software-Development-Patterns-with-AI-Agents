# Catalog: Swarm Frameworks, Task Graphs & Spec Harnesses

When scaling beyond a single agent ($1:N$ and $M:N$ topologies), raw LLM chat loops collapse under merge conflicts, context drift, and token thrashing (`Co-Coder`, arXiv:2606.00953). The frameworks in this category provide the **externalized memory, dependency graphs (DAGs), and supervisory hierarchies** required to coordinate multiple agents safely.

---

## 1. Gas Town
* **Link:** [github.com/gastownhall/gastown](https://github.com/gastownhall)
* **Taxonomy Profile:** Autonomy: $L_3 - L_4$ | Topology: $1:N$ Industrial Hierarchy | Patterns: `Hierarchical Supervisor`, `Issue-Driven Agent Mesh`

### Architectural Mechanics
Created by Steve Yegge, Gas Town models multi-agent software engineering as an industrial refinery rather than a human chatroom. It coordinates 10–30+ concurrent CLI agents (typically Claude Code or Codex instances) using rigid role separation and Git worktree isolation:
* **The Mayor (Dispatcher):** The top-level coordinator that translates human intent into a dependency graph of atomic tasks (Beads), monitors worker health, and never writes application code directly.
* **Polecats (Ephemeral Workers):** Short-lived coding agents spawned inside isolated Git worktrees (**Rigs**) via `tmux`. Each Polecat claims a single unblocked Bead, implements the change, runs local tests, commits, and terminates.
* **The Witness & The Refinery (Critic & Merge Queue):** Independent verification agents that audit Polecat diffs against acceptance criteria and manage a sequential rebase/merge queue so parallel branches never corrupt `main`.

### Failure Modes & Trade-offs
* **High Coupling Thrashing:** If the Mayor fails to isolate "Structural Hub" files (core shared types/interfaces) into a sequential prerequisite phase before fanning out parallel Polecats, the Refinery will bottleneck on irreconcilable semantic merge conflicts.

---

## 2. Beads & Beadwork (Git-Backed Agent Memory)
* **Links:** [Beads (steveyegge/beads)](https://github.com/steveyegge/beads) | [Beadwork (jallum/beadwork)](https://github.com/jallum/beadwork)
* **Taxonomy Profile:** Autonomy: Infrastructure Layer | Topology: Distributed DAG State | Patterns: `Issue-Driven Agent Mesh`, `Transactional Git Memory`

### Architectural Mechanics
Standard markdown `TODO.md` lists break under multi-agent concurrency because they lack atomic locking, dependency resolution, and context scoping.
* **Beads (`bd` CLI):** Stores micro-issues as structured JSONL/Markdown records inside a version-controlled `.beads/` directory. Agents invoke `bd ready --json` to query the DAG for tasks whose upstream dependencies (`depends_on`) have all reached `completed`, claim the task atomically, and log discoveries.
* **Beadwork:** A high-performance Go implementation of the Beads pattern that stores the task graph and agent state inside a **SQLite database backed by detached Git branches**, preventing working-tree pollution and eliminating JSONL merge conflicts during high-concurrency swarm runs.

---

## 3. GitHub Spec Kit (`specify` CLI)
* **Link:** [github.com/github/spec-kit](https://github.com/github/spec-kit)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ Governance Harness | Topology: Agent-Agnostic Workflow | Patterns: `Spec-Driven Development (Spec-Anchored)`, `Operational Policy Files`

### Architectural Mechanics
Spec Kit is GitHub's open-source toolkit for enforcing **Spec-Anchored Development** across any coding agent (Claude Code, Cursor, Copilot, Codex, Gemini CLI). It scaffolds a `.specify/` governance directory containing:
1. **`memory/constitution.md`:** The immutable architectural principles, testing mandates, and forbidden patterns of the repository.
2. **Slash-Command State Machine:** Injects standardized commands (`/specify`, `/plan`, `/tasks`, `/implement`) into the agent's workflow. Each phase generates validated artifacts inside `specs/<feature-branch>/` (including `spec.md`, `plan.md`, `data-model.md`, and `tasks.md`), ensuring no agent writes code before passing architectural gates.

---

## 4. MetaGPT & ChatDev (Role-Playing SOP Swarms)
* **Links:** [MetaGPT](https://github.com/geekan/MetaGPT) | [ChatDev](https://github.com/OpenBMB/ChatDev)
* **Taxonomy Profile:** Autonomy: $L_3$ | Topology: Sequential Anthropomorphic Pipeline | Patterns: `SOP-Driven Multi-Agent Pipeline`

### Architectural Mechanics
Early pioneers in multi-agent orchestration that model software development after a traditional human org chart (`Product Manager -> Architect -> Project Manager -> Engineer -> QA`):
* **MetaGPT** encodes **Standard Operating Procedures (SOPs)** into the handoffs: the PM agent must output a structured PRD and competitive analysis; the Architect agent must output sequence diagrams and interface definitions before the Engineer agent is invoked.
* **ChatDev** structures development as dyadic (two-agent) chat turns between an Instructor role and an Assistant role across design, coding, and testing phases.

### Failure Modes & Trade-offs
* **Greenfield Only / Brownfield Fragility:** Both frameworks excel at generating self-contained 0-to-1 prototypes (e.g., a 2,000-line web app or game from a single prompt), but struggle on existing enterprise repositories where conversational handoffs suffer severe context loss (`RACE-bench` waterfall degradation).