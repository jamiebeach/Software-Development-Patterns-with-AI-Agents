# Catalog: Autonomous Critic/Review Agents & Execution Sandboxes

Empirical industry telemetry (Google DORA & GitHub Octoverse) reveals a critical bottleneck in agentic engineering: while coding agents increase PR volume per developer by **20%+**, post-merge production incidents rise when human review queues become overwhelmed. 

This catalog covers the two infrastructure pillars that keep high-autonomy coding safe: **Autonomous Critic/Review Agents** (acting as the independent "Witness" at the PR gate) and **Agent Execution Sandboxes** (isolating untrusted agent compute).

---

## Part 1: Autonomous PR Critic & Verification Agents

| Tool | Review Scope | Core Architectural Differentiator | Best Fit |
| :--- | :--- | :--- | :--- |
| **Greptile** | Full Repository Graph | Indexes the entire codebase into an AST/dependency graph to catch cross-file regressions and broken downstream consumers. | Catching deep architectural & cross-layer bugs |
| **Qodo 2.0** | Multi-Repo & Ticket Compliance | Multi-agent review engine that validates PR diffs against linked Jira/Linear specs and cross-repo organization rules. | Enterprise governance, spec compliance, & test generation |
| **CodeRabbit** | Diff + Static Analysis | Combines LLM diff walkthroughs, sequence diagrams, and 40+ deterministic linters/SAST scanners inside PR comments. | Fast first-pass triage & high-volume PR summaries |

### 1. Greptile
* **Link:** [greptile.com](https://www.greptile.com/)
* **Patterns Implemented:** `Knowledge-Graph Grounding`, `Independent Critic Gate`
* **How It Works:** Standard PR bots only read the `git diff`, meaning they miss bugs where changing a default parameter in `auth.ts` silently breaks a consumer in `billing.ts` (which wasn't touched in the PR). Greptile maintains a full, continuously updated index of your repository (and supports VPC self-hosting). During PR review, it traverses the call graph to trace side effects across untouched files, flags missing environment prerequisites, and generates call-order sequence diagrams.

### 2. Qodo 2.0 (formerly CodiumAI)
* **Link:** [qodo.ai](https://www.qodo.ai/)
* **Patterns Implemented:** `Spec-to-PR Verification`, `Pass-to-Pass Regression Gate`
* **How It Works:** Qodo deploys a specialized multi-agent review pipeline at the PR gate. Beyond checking code quality, it connects directly to Jira, Linear, or repository rule files to perform **Requirement Gap Analysis**—verifying whether the agent-generated PR actually satisfies every acceptance criterion in the originating ticket, and generating missing unit tests directly in the PR thread.

### 3. CodeRabbit
* **Link:** [coderabbit.ai](https://www.coderabbit.ai/)
* **Patterns Implemented:** `Compiler-Gated Critic`, `Automated Triage`
* **How It Works:** Installed on over 2M+ repositories, CodeRabbit acts as a hybrid deterministic + LLM critic. Before invoking the LLM, it runs over 40 static analysis and SAST linters on the diff, feeding those deterministic signals into the review context to provide one-click commit fixes and conversational `@coderabbitai` follow-ups.

---

## Part 2: Agent Execution Sandboxes & Compute Runtimes

When building custom $L_3 - L_4$ coding agents or running parallel swarms, executing LLM-generated bash commands directly on a developer's host machine or shared CI server is a severe security and stability risk.

### 1. E2B (Code Interpreter & Agent Sandbox Cloud)
* **Link:** [e2b.dev](https://e2b.dev/)
* **Patterns Implemented:** `Bounded Context Sandbox`, `Ephemeral MicroVM Isolation`
* **How It Works:** E2B provides open-source, cloud-hosted **Firecracker microVMs** purpose-built as computers for AI agents. Unlike standard Docker containers that share a host kernel, Firecracker microVMs boot in ~150ms with hardware-level virtualization. Orchestrators can programmatically spin up a sandbox via Python/TS SDKs, run Claude Code, Codex, or OpenHands inside it, snapshot the filesystem and memory state mid-task (`pause`/`resume`), and tear it down safely.

### 2. Daytona
* **Link:** [daytona.io](https://www.daytona.io/)
* **Patterns Implemented:** `Standardized Dev Container Runtime`
* **How It Works:** Provides programmatic, sub-second workspace creation from standard `devcontainer.json` specifications, allowing multi-agent swarms (like Capy, Hermes, or custom Gas Town setups) to spin up identical, pre-warmed development environments with databases and toolchains ready to execute tests immediately.