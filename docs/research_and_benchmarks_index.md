# Empirical Research, Benchmarks & Field Studies

While agentic software engineering is evolving rapidly through practitioner experimentation, a growing body of academic research (arXiv, ICLR, FSE) and large-scale industry telemetry has begun testing these workflows empirically.

This page synthesizes the primary findings that ground the architectural patterns in this repository—specifically detailing **where raw coding agents fail** and **why structural scaffolding is required**.

---

## 1. Continuous Evolution & The "Regression Snowball"

Most early coding benchmarks (such as original SWE-bench) evaluated agents on **isolated, single-issue patches**. Recent research reveals a dramatic performance cliff when agents are tasked with evolving a software repository across multiple sequential milestones.

### Key Studies
* **`EvoClaw`**: *Evaluating AI Agents on Continuous Software Evolution* (`arXiv:2603.13428`)
* **`RACE-bench`**: *Benchmarking Coding Agents on Repository-Level Feature Addition* (`arXiv:2603.26337`)

### Empirical Findings
1. **The $80\% \rightarrow 38\%$ Continuous Evolution Cliff:** Frontier models that achieve $>80\%$ resolution rates on isolated single-task benchmarks drop to $\le 38\%$ when executing continuous, multi-milestone evolution on the same repository.
2. **Linear Recall vs. Saturated Precision:** Agents remain effective at generating new code to pass Fail-to-Pass (F2P) feature tests (*Recall*), but fail to preserve existing Pass-to-Pass (P2P) system invariants (*Precision*). Small architectural compromises accumulate across commits until a "regression snowball" renders the codebase unmaintainable.
3. **The Waterfall Degradation of Reasoning (`RACE-bench`):** Across $528$ complex repository-level tasks, researchers evaluated intermediate agent reasoning steps alongside final patch outcomes:
   * **Intent Comprehension:** $>9.2 / 10$ (Near-human understanding of what the user wants).
   * **File & Symbol Localization:** Sharp drop-off (Agents struggle to identify the exact cross-file call graph to modify).
   * **Task Decomposition & Step Execution:** Compounding failure once initial localization is slightly off-target.

```text
Agent Accuracy across the Execution Pipeline (RACE-bench & EvoClaw)
100% |  [██████████] 92%+ Intent Comprehension
 80% |  [████████  ] ~80% Single-Issue Isolated Patch (SWE-bench Verified)
 60% |  [██████    ] ~58% Multi-File Localization & Decomposition
 40% |  [████      ] ≤38% Multi-Milestone Continuous Evolution (EvoClaw)
  0% +--------------------------------------------------------------------
```

* **Repository Pattern Mapping:** Directly motivates [**Spec-Driven Development (SDD)**](patterns/team-and-enterprise/spec-driven-development.md)—specifically the mandatory **Phase 2 Planning & Localization Gate** and separating F2P feature tests from P2P invariant suites.

---

## 2. Multi-Agent Swarms & Cohesion-Aware Partitioning

A common intuition in multi-agent engineering is that spinning up $10$ parallel coding agents will complete a feature $10\times$ faster than a single agent. Graph-theoretic research proves that naive parallelism often degrades both speed and correctness.

### Key Study
* **`Co-Coder`**: *When Parallelism Pays Off: Cohesion-Aware Task Partitioning for Multi-Agent Coding* (`arXiv:2606.00953`)

### Empirical Findings
1. **The Coupled-File Token Thrash:** When two parallel worker agents are assigned tasks that touch tightly coupled modules, the cost of context synchronization, merge conflict resolution, and semantic drift outweighs any parallel speedup.
2. **Graph-Theoretic Task Partitioning:** `Co-Coder` models a codebase as a weighted dependency graph $G = (V, E)$, where vertex weight $w_i$ represents isolated agent execution cost and edge weight $c_{ij}$ represents cross-file coupling. Optimal multi-agent orchestration requires two structural steps before spawning workers:
   * **Structural Hub Isolation:** Core interfaces, shared types, and heavily imported utility files ("hubs") must be isolated and executed in a **sequential Phase 0**.
   * **Community Detection:** Only after hub contracts are committed should the supervisor fan out parallel worker agents across cohesive, low-coupling module clusters.

* **Repository Pattern Mapping:** Grounds the task-slicing rules in [**Issue-Driven Agent Mesh (Beads)**](patterns/multi-agent-orchestration/issue-driven-mesh.md).

---

## 3. Operational Policy Files & Ambiguity Handling

How should repositories communicate rules to coding agents, and how do agents behave when instructions are incomplete?

### Key Studies
* **`Ambig-SWE`**: *Resolving Underspecified Instructions in Software Engineering Agents* (ICLR 2026)
* **Linux Foundation `AGENTS.md` & GitHub 2,500-Repo Field Analysis** (2025–2026)

### Empirical Findings
1. **Silent Ambiguity Failure:** In `Ambig-SWE`, researchers demonstrated that off-the-shelf coding agents almost never ask clarifying questions when given ambiguous or underspecified issues—they silently guess and implement plausible-looking but incorrect behavior. Adding an explicit **Ambiguity Escalation Gate** improves resolution accuracy by up to $74\%$.
2. **Narrative Prose vs. Executable Commands:** Analysis of $2,500+$ repositories using `AGENTS.md` and `CLAUDE.md` found that long-form architectural prose is routinely ignored by models during deep tool-calling loops. High-performing policy files share four structural traits:
   * **Command-First:** Exact, scoped CLI invocations with flags.
   * **Task-Organized:** Rules grouped by workflow stage rather than domain essays.
   * **Closure-Defined:** Explicit exit-code requirements for task completion.
   * **Negative Constraints ("Never" Lists):** Explicit bans on deleting tests, mutating linter configs, or bypassing type checks.

* **Repository Pattern Mapping:** Encapsulated in [**Operational Policy & Constitutional Files**](patterns/reliability-and-state/operational-policy-files.md).

---

## 4. Spec-Driven Maturity & The "Sledgehammer" Anti-Pattern

### Key Study
* **Practitioner Field Analysis:** Birgitta Boeckeler (*MartinFowler.com*: *Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*) & GitHub Spec Kit Telemetry.

### Empirical Findings
1. **The Three Tiers of SDD:** Clarifies the industry split between **Spec-First** (ephemeral specs), **Spec-Anchored** (living `specs/` + `constitution.md`), and **Spec-as-Source** (code as a compiled, untouched build artifact).
2. **The "Sledgehammer on a Nut" Failure Mode:** Applying enterprise Spec-Anchored workflows to small bug fixes creates severe review bottlenecks—turning a 10-line code fix into hundreds of lines of generated user stories, acceptance matrices, and task breakdowns. Effective agentic engineering requires a **Triage Gate** to match workflow ceremony to task complexity.

* **Repository Pattern Mapping:** Integrated into [**Spec-Driven Development (SDD)**](patterns/team-and-enterprise/spec-driven-development.md).

---

## 5. Proof-Carrying Harnesses & Formal Verification

Because empirical unit tests can be incomplete—or actively gamed by reward-seeking agents—researchers are pairing general-purpose coding agents with formal verification kernels (Coq, Lean 4, Dafny).

### Key Studies
* **`Aria`**: *Harnessing Code Agents for Automatic Software Verification* (`arXiv:2607.06341`)
* **`MAGS`**: *Multi-Agent Auto-Formalization Guarantees Safety* (`arXiv:2609.19391`)

### Empirical Findings
* **Kernel-Gated Autonomy:** By wrapping an off-the-shelf coding agent (Claude Code) in a declarative **Harness Hook Language (HHL)** that rejects incomplete proofs (`Admitted`), enforces step timeouts, and feeds back exact kernel error traces across up to $30$ retry loops, `Aria` successfully proved $4,257$ separation-logic lemmas in Iris and $217$ lemmas verifying Rust's standard concurrency primitives (`Arc`, `Mutex`, `RwLock`).
* **"Trust is Free":** When the verification gate is a sound formal kernel rather than a human code reviewer, human review fatigue drops to zero—the kernel mathematically rejects any hallucinated implementation.

---

## 6. Fixed Pipelines vs. Unconstrained Autonomy

### Key Studies
* **`Agentless`**: *Demystifying LLM-Based Software Engineering Agents* (FSE 2025)
* **`Ark`**: *Understanding the Architecture of Coding Agents* (`arXiv:2608.10934`)

### Empirical Findings
* Granting an agent an unconstrained bash shell to loop autonomously ($L_3 \rightarrow L_4$) is often unnecessary and cost-prohibitive for routine maintenance. Deterministic, fixed-topology pipelines (**Localize File $\rightarrow$ Localize Function $\rightarrow$ Generate Candidate Patches $\rightarrow$ Filter via Reproduction Tests**) frequently match or outperform unconstrained autonomous agents at a fraction of the token budget.

---

## 7. Summary Reference Matrix

| Study / Source | Core Focus | Primary Takeaway for Practitioners |
| :--- | :--- | :--- |
| **`EvoClaw`** (`arXiv:2603.13428`) | Continuous repo evolution | Isolate Fail-to-Pass (F2P) feature tests from Pass-to-Pass (P2P) invariant suites to stop regression snowballs. |
| **`RACE-bench`** (`arXiv:2603.26337`) | Multi-step feature reasoning | Always gate agent execution behind an explicit **File Localization & Plan Review** step. |
| **`Co-Coder`** (`arXiv:2606.00953`) | Multi-agent task graphs | Execute shared "hub" interfaces sequentially before fanning out parallel worker swarms. |
| **`Ambig-SWE`** (ICLR 2026) | Underspecified instructions | Add mandatory ambiguity escalation triggers to `AGENTS.md` ($+74\%$ accuracy). |
| **Boeckeler / Martin Fowler** | Spec-Driven Development | Use a Triage Gate to avoid applying heavy SDD ceremony to small bug fixes. |
| **`Aria` & `MAGS`** (`arXiv:2607.06341`) | Formal verification harnesses | Use compiler/proof kernels as uncompromising critic gates for high-assurance concurrency and security code. |