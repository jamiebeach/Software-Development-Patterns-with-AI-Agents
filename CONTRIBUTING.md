# Contributing to Software Development Patterns with AI Agents

First off, thank you for considering contributing to this field guide. 

Because agentic software engineering is evolving in real time, **this repository relies on practitioners in the trenches.** Whether you are a solo developer orchestrating local CLI agents, an open-source maintainer building swarm tooling, or an enterprise architect rolling out verification guardrails across hundreds of engineers, your field reports make this project valuable.

---

## 1. Our Guiding Philosophy

Before submitting an issue or Pull Request, please keep our core tenets in mind:

1. **Patterns over Products:** A *Pattern* (`docs/patterns/`) must describe a repeatable architectural workflow, state machine, or coordination primitive that can be implemented across multiple tools. Specific tools, commercial platforms, and frameworks belong in the *Catalog* (`docs/catalog/`).
2. **No Marketing Fluff:** We welcome contributions from creators of agentic tools, but entries must read like systems architecture documentation—not a sales pitch. Every contribution **must** honestly document failure modes, token/compute trade-offs, and where the approach breaks down.
3. **Battle-Tested over Theoretical:** We strongly prefer patterns and anti-patterns observed in real software builds over speculative multi-agent architectures that have only been tested on toy benchmarks.
4. **Embrace the 6-Month Half-Life:** Models improve, context windows expand, and scaffolding dissolves. Updating an existing pattern to note that a recent model capability has made part of it obsolete is just as valuable as proposing a brand-new pattern.

---

## 2. Ways to Contribute

### A. Propose a New Pattern (`docs/patterns/`)
If you have identified a distinct workflow or coordination model not yet covered, submit a PR adding a markdown file under the appropriate category directory:
* `docs/patterns/solo-developer/`
* `docs/patterns/multi-agent-orchestration/`
* `docs/patterns/team-and-enterprise/`
* `docs/patterns/reliability-and-state/`

*(See the **Pattern Template** in Section 4 below.)*

### B. Add an Anti-Pattern or Failure Mode
Often, the most helpful contribution is adding a row to the **Anti-Patterns & Pitfalls** table of an existing pattern based on a painful lesson learned in production.

### C. Contribute a Tool / System Case Study (`docs/catalog/`)
Have you dissected how a specific system (e.g., Beads, Gas Town, Devin, SWE-agent, Blitzy, Claude Code, Aider) manages state, context, and verification under the hood? Add or update a case study in `docs/catalog/`.

### D. Submit a "Half-Life / Model Shift" Update
Did a new model release or runtime feature render an existing workaround unnecessary? Open a PR updating the **Maturity & Shelf-Life** section of the affected pattern.

---

## 3. Style & Formatting Guidelines

* **Diagrams:** Use clean ASCII/Unicode box diagrams (`+---+`, `│`, `▼`) or Mermaid flowcharts so architecture and state transitions are immediately legible in standard GitHub Markdown view.
* **Taxonomy Alignment:** Reference the dimensions defined in [`docs/taxonomy.md`](docs/taxonomy.md) (Autonomy levels $L_0 - L_4$, Topologies $1:1$, $1:N$, $M:N$, and Verification Gates).
* **Concrete Artifacts:** Whenever possible, include realistic snippets of actual artifacts (e.g., a sample `spec.md`, a `.beads` JSON payload, a `git worktree` script, or a critic loop terminal log) rather than abstract prose.

---

## 4. Template: New Pattern Specification

Copy and paste the template below when creating a new file in `docs/patterns/<category>/<pattern-name>.md`:

~~~markdown
# Pattern: [Pattern Name]

* **Category:** [Solo Developer | Multi-Agent Orchestration | Team & Enterprise | Reliability & State]
* **Autonomy Level:** [L0 | L1 | L2 | L3 | L4] (See `docs/taxonomy.md`)
* **Topology:** [1:1 Synchronous | 1:N Swarm | M:N Mesh]
* **Maturity Level:** [Experimental | Emerging | Production]
* **Primary Actors:** [e.g., Human Architect, Planner Agent, Execution Agent, Critic Agent]
* **Key Exemplars:** [2-5 tools or open-source projects that embody this pattern]

---

## 1. Intent & Overview

[1-2 paragraphs concisely explaining what this pattern is, why it exists, and the core mechanism by which it coordinates human intent and agent execution.]

```text
[Insert ASCII Architecture or Workflow Diagram Here]
```

---

## 2. The Problem It Solves

[Bullet out the specific failure modes of naive prompting or human-shaped workflows that trigger the need for this pattern—e.g., context drift, merge collisions, review bottlenecks.]

1. **[Failure Mode 1]:** [Explanation]
2. **[Failure Mode 2]:** [Explanation]
3. **[Failure Mode 3]:** [Explanation]

---

## 3. Core Principles

[3-5 foundational rules or invariants that make this pattern work.]

1. **[Principle 1]:** [Explanation]
2. **[Principle 2]:** [Explanation]
3. **[Principle 3]:** [Explanation]

---

## 4. Prerequisites & Environmental Setup

[What must be true about the repository, test suite, sandboxing, or team before adopting this pattern?]

* **[Requirement 1]:** [Details]
* **[Requirement 2]:** [Details]

---

## 5. Lifecycle & Execution Workflow

[Walk through the state transitions or step-by-step operational loop.]

---

## 6. Concrete Walkthrough

[Provide a realistic scenario showing actual prompts, state files, CLI commands, or verification loops.]

---

## 7. Anti-Patterns & Pitfalls

| Anti-Pattern | Manifestation / Symptom | Remediation |
| :--- | :--- | :--- |
| **[Name]** | [What goes wrong in practice] | [How to fix or guard against it] |
| **[Name]** | [What goes wrong in practice] | [How to fix or guard against it] |

---

## 8. Half-Life & Future Trajectory

[How likely is this pattern to be absorbed by future model improvements, or how will it evolve over the next 6-12 months?]

---

## 9. Related Patterns

* **[Related Pattern A](../path/to/pattern.md):** [How they connect or compose]
~~~

---

## 5. Template: Catalog / Tool Case Study

When documenting a specific tool or architecture in `docs/catalog/<tool-name>.md`, use the following structure to keep the analysis objective and architectural:

~~~markdown
# Case Study: [Tool / System Name]

* **Primary Pattern(s) Implemented:** [Link to pattern files]
* **Autonomy & Topology:** [e.g., L3 Asynchronous Swarm (1:N)]
* **Open Source / Commercial:** [License or availability]

## 1. Architectural Summary
[How does the system actually work under the hood? How does it ingest context, manage state, and execute tool calls?]

## 2. State, Memory & Context Strategy
[Where does state live? Ephemeral window, git-tracked files, AST graph, container checkpoints?]

## 3. Verification & Guardrail Loop
[How does the tool verify its own code before handing it to a human?]

## 4. Known Limitations & Trade-Offs
[Where does this tool struggle? Token cost, latency, setup friction, specific codebase sizes?]
~~~

---

## 6. Pull Request Checklist

Before submitting your PR, please verify:

- [ ] Does the contribution clearly separate vendor-agnostic patterns from tool-specific implementations?
- [ ] Are failure modes, trade-offs, and anti-patterns explicitly documented?
- [ ] Are internal links to `docs/taxonomy.md` and related patterns valid?
- [ ] Have you updated the Pattern Index in `README.md` if adding a new file?