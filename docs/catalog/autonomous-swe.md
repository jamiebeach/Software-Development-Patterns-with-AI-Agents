# Catalog: Autonomous Software Engineers & Cloud Sandboxes

Autonomous Software Engineers ($L_3 - L_4$) shift the interaction paradigm from synchronous in-editor pairing to **asynchronous task delegation**. Triggered by a GitHub Issue, Slack thread, or Linear ticket, these systems spin up an isolated cloud VM, localize the fault, implement a patch, verify it against test suites, and open a Pull Request.

---

## Architectural Dichotomy: Open-Ended Loops vs. Fixed Pipelines

Research on **[SWE-bench](https://www.swebench.com/)** and **[EvoClaw](https://arxiv.org/abs/2603.13428)** reveals two competing schools of thought in this category:
1. **Unconstrained Sandbox Agents (Devin, OpenHands, SWE-agent):** Give the agent a stateful Linux VM, bash shell, and browser, allowing it to loop dynamically for dozens of steps. High ceiling for complex tasks, but prone to cascading regressions and high token costs.
2. **Structured / Deterministic Pipelines (Agentless, AutoCodeRover):** Restrict the agent to a strict sequence: *AST Fault Localization $\rightarrow$ Context Extraction $\rightarrow$ Candidate Patch Generation $\rightarrow$ Test Filtering*. Much cheaper and more predictable for routine bug fixes, though unable to perform interactive environment debugging.

---

## 1. Devin (by Cognition AI)
* **Link:** [cognition.ai](https://www.cognition.ai/)
* **Taxonomy Profile:** Autonomy: $L_3 - L_4$ | Topology: $1:N$ Asynchronous Cloud Fleet | Patterns: `Autonomous Sandbox Engineer`, `Playbook Memory`

### Architectural Mechanics
Devin runs inside a persistent, cloud-hosted Linux container equipped with a shell, code editor, and headless Chrome browser. 
* **Interactive Planning & Confidence Scoring:** When assigned a ticket, Devin scans the repo and outputs an initial confidence assessment and step-by-step plan before executing code changes.
* **Devin Wiki & Tribal Knowledge:** Automatically indexes repositories to generate living architectural diagrams and documentation, plus reusable `.devin.md` **Playbooks** that codify repeatable team workflows (e.g., how to run integration migrations).
* **Interactive Timeline UI:** Humans can scrub backward and forward through Devin's shell history, browser screenshots, and file diffs, or jump directly into the live container terminal to unstick the agent.

### Failure Modes & Trade-offs
* **Ambiguity Hallucination:** When assigned poorly scoped tickets without acceptance criteria, Devin will frequently invent product requirements to make its test suite pass rather than halting earlier. Best paired with **Spec-Driven Development**.

---

## 2. SWE-agent & SWE-reX (Princeton NLP)
* **Link:** [github.com/SWE-agent/SWE-agent](https://github.com/SWE-agent/SWE-agent)
* **Taxonomy Profile:** Autonomy: $L_3$ | Topology: Single-Task Autonomous Loop | Patterns: `Agent-Computer Interface (ACI)`

### Architectural Mechanics
SWE-agent proved that **the interface presented to the LLM matters as much as the underlying model weights**. Standard Unix tools (`cat`, `sed`, `vim`) are designed for humans, not stateless token windows: running `cat` on a 5,000-line file overflows context, while a bad `sed` regex corrupts syntax.
* **Custom ACI Primitives:** SWE-agent replaces raw file manipulation with guarded, stateful commands: `open <file>` (displays a 100-line viewport with line numbers), `scroll_down`, `search_dir`, and `edit <start>:<end>` (which immediately runs a syntax linter and rejects the edit automatically if a syntax error is introduced).
* **SWE-reX Runtime:** Decouples the agent orchestration loop from the execution sandbox, allowing SWE-agent to run dozens of parallel evaluations across Docker, Modal, or AWS Fargate.

---

## 3. OpenHands (formerly OpenDevin)
* **Link:** [github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)
* **Taxonomy Profile:** Autonomy: $L_3 - L_4$ | Topology: Modular Multi-Agent Runtime | Patterns: `CodeAct Architecture`, `Event-Sourced State Stream`

### Architectural Mechanics
OpenHands is the leading open-source platform for full-stack autonomous software engineering.
* **CodeAct Paradigm:** Rather than exposing dozens of brittle JSON tool schemas, OpenHands consolidates agent actions into executable Python code and Bash commands inside a sandboxed Docker runtime.
* **Deterministic EventStream:** Every agent thought, bash execution, browser DOM snapshot, and observation is recorded as an immutable event on a central pub/sub bus. This allows failed runs to be replayed, audited, or branched from any historical step.

---

## 4. Capy
* **Link:** [capy.ai](https://capy.ai/)
* **Taxonomy Profile:** Autonomy: $L_3 - L_4$ | Topology: $1:N$ Captain + Parallel Cloud VMs | Patterns: `Hierarchical Supervisor`, `Isolated VM Sandboxing`

### Architectural Mechanics
Capy bridges the gap between local CLI tools and cloud swarms using a two-tier cloud architecture:
1. **The Captain Agent:** Reads the repository, decomposes a backlog or epic into discrete specifications, and plans the execution graph.
2. **Parallel Build Agents:** Each specification is handed to an independent Build Agent running inside its own isolated cloud VM (pre-configured with Docker, databases, and toolchains). Five tasks execute simultaneously across five VMs without file lock collisions, passing through an automated review gate before opening GitHub PRs.

---

## 5. Agentless & AutoCodeRover (Structured Non-Agentic Pipelines)
* **Links:** [github.com/OpenAutoCoder/Agentless](https://github.com/OpenAutoCoder/Agentless) | [github.com/nus-apr/auto-code-rover](https://github.com/nus-apr/auto-code-rover)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ (Fixed Pipeline) | Topology: Sequential DAG | Patterns: `AST Fault Localization`

### Architectural Mechanics
Developed by researchers at UIUC and NUS respectively, these systems challenge the need for autonomous bash loops on routine bug fixes:
* **Hierarchical Localization (`Agentless`):** Compresses the repo into a directory tree $\rightarrow$ identifies top-K suspicious files $\rightarrow$ compresses files into class/function skeletons $\rightarrow$ pinpoints exact line ranges $\rightarrow$ generates 40 candidate search/replace patches in parallel and ranks them using regression test reproduction.
* **Spectrum-Based AST Search (`AutoCodeRover`):** Uses Abstract Syntax Tree (AST) APIs (`search_class`, `search_method_in_class`) combined with test-spectrum analysis to locate faults structurally rather than reading raw files, resolving issues at a fraction of the token cost of conversational agents.