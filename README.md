# Software Development Patterns with AI Agents

> *An evolving field guide to the architectures, workflows, and coordination primitives emerging at the frontier of agentic software engineering.*

---

## 1. The Great Refactoring: From Human-Shaped to Agent-Shaped Development

For the past fifty years, every practice in software engineering was designed around a single, immutable constraint: **the shape of the human mind.**

Think about the foundational artifacts of modern software development:
* **Two-week Agile sprints and daily standups** exist because humans operate on circadian rhythms and need periodic synchronization to stay aligned.
* **Jira tickets and Markdown checklists** are written in ambiguous natural language because human engineers bring years of implicit institutional context to fill in the blanks.
* **Small Pull Requests (< 400 lines)** are mandated because human working memory degrades rapidly during code review.
* **Clean Code, DRY, and deep folder hierarchies** optimize for human navigability and cognitive load.

In a remarkably short span of time, we have introduced a fundamentally different kind of actor into the software lifecycle. AI coding agents do not get fatigued at 4:00 PM, can spin up twenty parallel instances in isolated git worktrees in milliseconds, and can synthesize boilerplate across fifty files in seconds. Yet they also suffer from failure modes no human engineer experiences: **sudden context-window amnesia, silent architectural drift, sycophantic test tampering, and compounding hallucination loops.**

Trying to force autonomous agents into *human-shaped* workflows—handing an agent a vague Jira ticket, letting it run in a single endless chat session, or asking a human team to manually review 15,000 lines of AI-generated diffs a day—breaks the system. 

We are witnessing a rapid, industry-wide transition toward **agent-shaped development**: new ways of structuring state, specifications, verification gates, and concurrency designed around the strengths and failure modes of machines, steered by human intent.

---

## 2. Why This Repository Exists

Right now, the software industry is in the middle of a massive, decentralized phase of live experimentation. There is no settled orthodoxy. There is no definitive "Gang of Four" textbook for building software with agents—because the underlying models, context windows, and tool-use capabilities are shifting beneath our feet every few weeks.

Across startups, open-source communities, and enterprise engineering orgs, builders are discovering entirely different paradigms through trial and error:
* Solo developers are **"vibe coding"** full-stack prototypes in tight visual REPL loops.
* Systems engineers are building **"Gas Towns"** and **"Beads"**—running swarms of 10 to 30 concurrent agents coordinated by supervisory "Mayors" over git-backed issue DAGs.
* Principal architects are retreating from chat prompts entirely, practicing strict **Spec-Driven Development (SDD)** where agents are locked in read-only test harnesses until deterministic contracts pass.
* Enterprise platforms are ingesting 20-million-line monorepos into **AST Knowledge Graphs** to execute multi-hour asynchronous refactorings before a human ever sees a diff.

**The goal of this repository is to take stock.** 

Rather than promoting a single tool or declaring a premature standard, this project aims to capture, categorize, and distill the key architectural insights and repeatable patterns that are actually working in the wild today.

---

## 3. Human-Shaped vs. Agent-Shaped Engineering

To understand the patterns in this repository, it helps to contrast the assumptions of traditional engineering with the emerging realities of agentic workflows:

| Dimension | Human-Shaped Development | Agent-Shaped Development |
| :--- | :--- | :--- |
| **Primary Bottleneck** | Typing speed, implementation time, context switching | Verification rigor, specification clarity, human review bandwidth |
| **State & Memory** | Implicit institutional knowledge + long-term human memory | Ephemeral session windows + explicit, git-persisted state graphs (`.beads/`, `spec.md`) |
| **Task Granularity** | Multi-day user stories ("Build the billing settings page") | Atomic, 2-to-5 minute micro-tasks with strict input/output boundaries |
| **Concurrency Model** | 1 branch per developer; long-lived feature branches | $N$ isolated Git worktrees or containers per developer, merged continuously |
| **Communication Protocol** | Natural language prose, Slack threads, meetings | Machine-queryable JSON/DAGs, strict type schemas, executable test fixtures |
| **Quality Assurance** | Peer human code review + post-push CI pipelines | Pre-commit compiler/linter critic loops + dual-agent adversarial verification |
| **Failure Recovery** | Debugging in place; incremental hotfixes | **"Land the plane or kill the session"**—discarding tainted context and respawning clean |

---

## 4. How to Use This Repository

We have decoupled **Patterns** (timeless or structural workflows, state machines, and mental models) from the **Catalog** (specific tools, CLIs, and commercial platforms implementing them).

1. **Start with the [Taxonomy (`docs/taxonomy.md`)](docs/taxonomy.md):** Understand the five core axes—Autonomy ($L_0 \rightarrow L_4$), Concurrency Topologies ($1:1$, $1:N$, $M:N$), Memory Persistence, Verification Rigor, and Context Grounding.
2. **Identify Your Operating Context:** Are you a solo builder looking for velocity, a staff engineer orchestrating parallel local agents, or an enterprise lead introducing guardrails across a 100-person org?
3. **Adopt the Guardrails Before the Autonomy:** A recurring lesson across every pattern here is that **autonomy without a deterministic verification harness is just accelerated technical debt.**

---

## 5. Repository Structure & Pattern Index

```text
├── README.md                                     # Philosophy, context, and master index
├── CONTRIBUTING.md                               # Pattern RFC template & submission guide
└── docs/
    ├── taxonomy.md                               # The 5 axes of agentic development
    ├── patterns/
    │   ├── solo-developer/
    │   │   ├── vibe-coding.md                    # High-velocity conversational REPL loops
    │   │   └── self-healing-repl.md              # Automated compiler/runtime feedback loops
    │   ├── multi-agent-orchestration/
    │   │   ├── issue-driven-mesh.md              # Git-tracked task DAGs (The "Beads" pattern)
    │   │   ├── hierarchical-supervisor.md        # Mayor / Worker / Witness swarms ("Gas Town")
    │   │   └── adversarial-critic-loop.md        # Dual-agent generator + verifier gates
    │   ├── team-and-enterprise/
    │   │   ├── spec-driven-development.md        # Contract-first execution & immutable specs
    │   │   ├── knowledge-graph-grounding.md      # AST / call-graph indexing for massive repos
    │   │   └── human-in-the-loop-review.md       # Triage gates & provenance-backed PRs
    │   └── reliability-and-state/
    │       ├── ephemeral-worktree-isolation.md   # Preventing write collisions in parallel runs
    │       └── context-compaction-checkpoints.md # Surviving long-horizon tasks without amnesia
    └── catalog/                                  # Implementations & case studies
        ├── gastown-and-beads.md
        ├── devin-and-swe-agent.md
        ├── blitzy-and-factory.md
        └── claude-code-and-aider.md
```

### Quick Pattern Matrix

| Pattern | Topology | Autonomy | Best Suited For | Core Mechanism |
| :--- | :--- | :--- | :--- | :--- |
| **[Vibe Coding](docs/patterns/solo-developer/vibe-coding.md)** | $1:1$ Sync | $L_1$ | UI prototypes, greenfield exploration, throwaway spikes | Tight human-in-the-loop visual/REPL steering |
| **[Spec-Driven Development (SDD)](docs/patterns/team-and-enterprise/spec-driven-development.md)** | $1:1$ or $1:N$ | $L_2$ | Production features, strict APIs, complex business logic | Immutable `spec.md` + read-only test suites gating execution |
| **[Issue-Driven Agent Mesh](docs/patterns/multi-agent-orchestration/issue-driven-mesh.md)** | $1:N$ Swarm | $L_3$ | Multi-step features, backlog burning, parallel execution | Git-backed atomic task graphs (`.beads`) + short-lived workers |
| **[Hierarchical Supervisor](docs/patterns/multi-agent-orchestration/hierarchical-supervisor.md)** | $1:N$ Swarm | $L_3$ | High-throughput solo/small-team multipliers | Dispatcher ("Mayor") assigning worktrees to Workers & Critics |
| **[Knowledge-Graph Grounding](docs/patterns/team-and-enterprise/knowledge-graph-grounding.md)** | $M:N$ Mesh | $L_3 - L_4$ | Legacy modernization, 1M+ LOC enterprise monorepos | Deep AST/dependency graph ingestion + asynchronous batching |

---

## 6. A Note on Imperfection & The 6-Month Half-Life

**This repository is intentionally incomplete, occasionally contradictory, and guaranteed to age rapidly.**

Many of the patterns documented here—such as aggressive context-window pruning, external CLI task ledgers, or elaborate multi-agent supervisory hierarchies—exist partially as scaffolding around current model limitations. As reasoning models deepen, native context windows become truly lossless at millions of tokens, and RL-trained coding environments mature, some of today's essential workarounds will dissolve into the model layer.

In six months, parts of this repository will look like historical artifacts, while new patterns we haven't yet named will take center stage. We believe there is immense value in documenting the territory *while* we are crossing it, rather than waiting for the dust to settle years from now.

---

## 7. Looking Ahead: Where Is This Going?

If we extrapolate from the patterns emerging today, several long-term trajectories become visible:

1. **Specifications Become the Primary Source Code:** Just as high-level languages abstracted away assembly, structured specifications, behavioral invariants, and formal verification harnesses are beginning to abstract away implementation syntax. Code is increasingly treated as a compiled, regenerable artifact of the spec.
2. **Codebases Optimized for Machine Legibility:** We will begin architecting repositories not for how easily a human can browse them in an IDE, but for how deterministically an agent swarm can parse, isolate, test, and mutate them—favoring strict static typing, hermetic sub-packages, instant local test execution, and explicit dependency graphs.
3. **Continuous Background Maintenance (Self-Healing Repos):** Rather than batching dependency upgrades, dead-code removal, or performance optimizations into quarterly human sprints, background custodian agents ($L_4$) will continuously propose, verify, and land micro-improvements while humans focus on product topology and domain modeling.
4. **The Rise of the "Fleet Architect":** The core skill of the senior software engineer is shifting from *writing syntax* to *designing the factory*—crafting the system prompts, verification harnesses, DAG decomposes, and critic gates that allow a fleet of agents to build reliably.

---

## 8. Contributing

Because no single person or team has all the answers, this repository relies on field reports from engineers building in the trenches.

* Have you discovered a failure mode in one of our documented patterns?
* Are you running a multi-agent workflow or enterprise guardrail that isn't captured here?
* Has a new model release rendered one of these patterns obsolete?

Please open an issue or submit a Pull Request. See [CONTRIBUTING.md](CONTRIBUTING.md) for our pattern template and architectural guidelines.