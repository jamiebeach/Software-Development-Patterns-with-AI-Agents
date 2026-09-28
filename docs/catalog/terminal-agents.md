# Catalog: Terminal-First & CLI Agents

Terminal-first agents operate directly inside the developer's shell and local Git environment. Rather than hiding behind a custom GUI, they treat Unix primitives (`git`, `grep`, `ripgrep`, test runners, linters) as their native Agent-Computer Interface (ACI). They excel at rapid $1:1$ pair programming and can be scripted into headless $1:N$ multi-agent swarms via Git worktrees and `tmux`.

---

## Comparative Architecture Matrix

| Tool | Autonomy Tier | Context Strategy | Edit Application Mechanism | Policy File Standard | License |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Claude Code** | $L_2 - L_3$ | Dynamic agentic search (`glob`, `grep`, file reads) | Exact string search/replace tool calls | `CLAUDE.md` / `AGENTS.md` | Proprietary (Anthropic) |
| **OpenAI Codex CLI** | $L_2 - L_3$ | Local directory inspection + reasoning loop | Structured patch / diff application in sandbox | `AGENTS.md` (`/init` native) | Apache 2.0 |
| **Aider** | $L_2$ | Tree-sitter Repo-Map + explicit `/add` context | Configurable (`diff`, `udiff`, `whole`, `architect`) | `.aider.conf.yml` / conventions | Apache 2.0 |
| **Goose (Block)** | $L_2 - L_3$ | Extension-driven via Model Context Protocol (MCP) | MCP filesystem & shell extensions | `.goosehints` / Recipes | Apache 2.0 |
| **Plandex** | $L_2 - L_3$ | Long-context staging sandbox (2M+ token management) | Isolated sandbox branching + atomic diff review | Plan-scoped context files | MIT (Client) |
| **Gemini CLI** | $L_2 - L_3$ | 1M+ token native window + Google Search grounding | Direct shell & file tool calls | `GEMINI.md` / `AGENTS.md` | Apache 2.0 |
| **Sourcegraph Amp** | $L_2 - L_3$ | Sourcegraph cross-repo code intelligence graph | Agentic CLI file edits + automated review | `AGENT.md` | Proprietary |
| **Hermes** | $L_3$ | Persistent cross-session skill memory | Local or remote container execution (SSH/Modal) | Persistent skill store | Open Source |
| **OpenCode** | $L_2 - L_3$ | Client-server session state + LSP integration | Provider-agnostic tool calls (75+ providers) | `AGENTS.md` | MIT |

---

## 1. Claude Code
* **Link:** [docs.anthropic.com/claude-code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ (Interactive) or $1:N$ (Headless `-p` flag) | Patterns: `Operational Policy Files`, `Self-Healing REPL`, `Hierarchical Supervisor` (as worker runtime)

### Architectural Mechanics
Unlike earlier tools that pre-index the repository into a vector database, Claude Code relies on **agentic retrieval**. When given a task, it uses fast local shell primitives (`ripgrep`, `find`, directory listing) to dynamically locate relevant files on demand. It operates a tight `Think -> Act -> Observe` loop with native support for sub-agent spawning (dispatching ephemeral child contexts to investigate deep call stacks without polluting the main conversation window).

### Signature Workflow
* **Interactive Plan-Then-Execute:** Using `Shift+Tab` to enter **Plan Mode**, where the agent investigates the codebase read-only, aligns with the developer on an implementation strategy, and then executes edits and runs test suites autonomously.
* **Headless CI / Swarm Worker:** Because it supports non-interactive execution (`claude -p "fix lint errors"`), it serves as the primary worker engine inside multi-agent orchestrators like Gas Town.

### Failure Modes & Trade-offs
* **Context Compaction Loss:** On long sessions exceeding 150k+ tokens, automatic context compaction (`/compact`) can summarize away subtle architectural constraints unless those invariants are anchored in `CLAUDE.md`.
* **Token Burn on Blind Search:** In massive monorepos without clear directory hints in `CLAUDE.md`, agentic grep searches can burn substantial input tokens exploring dead ends.

---

## 2. Aider
* **Link:** [aider.chat](https://aider.chat/)
* **Taxonomy Profile:** Autonomy: $L_2$ | Topology: $1:1$ Pair Programmer | Patterns: `Self-Healing REPL`, `Architect-Editor Split`

### Architectural Mechanics
Aider pioneered two foundational mechanics in terminal coding agents:
1. **The Tree-sitter Repo-Map:** Aider parses the entire repository using Tree-sitter to extract symbol definitions, class signatures, and call graphs, ranking them with a PageRank algorithm over the dependency graph. This fits a structural map of a large repo into a small token budget.
2. **Architect/Editor Model Split:** In `architect` mode, Aider pairs a high-reasoning model (to propose the structural solution) with a fast, syntax-specialized editing model (to format the exact search/replace diff blocks).Every edit is automatically committed to Git with a descriptive message, making `git reset --hard HEAD~1` a deterministic undo button.

### Signature Workflow
Tight, multi-file refactoring across 2–10 known files where the developer wants explicit control over which files enter the active context window (`/add`, `/drop`) alongside automatic lint/test feedback loops.

### Failure Modes & Trade-offs
* **Manual Context Curation Overhead:** Unlike Claude Code or Codex CLI, Aider is less effective when you don't know where a bug lives; adding too many files manually degrades edit precision.

---

## 3. OpenAI Codex CLI
* **Link:** [github.com/openai/codex](https://github.com/openai/codex)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ / $1:N$ | Patterns: `Operational Policy Files` (`AGENTS.md`), `Bounded Context Sandbox`

### Architectural Mechanics
OpenAI's open-source terminal agent connects reasoning models (`o3`, `o4-mini`, `gpt-5` series) directly to local repositories. It emphasizes **OS-level sandboxing**: on macOS (`sandbox-exec`) and Linux (Docker/iptables), Codex restricts network access and prevents writes outside the active working directory even when running in full-auto mode. Running `/init` automatically scans the repository build scripts and generates a standardized `AGENTS.md` file.

### Signature Workflow
Running in `full-auto` sandboxed mode to iteratively implement a feature or fix a failing test suite with zero risk of the agent accidentally mutating global system files or exfiltrating secrets over the network.

---

## 4. Goose (by Block)
* **Link:** [block.github.io/goose](https://block.github.io/goose/)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ / Team Automation | Patterns: `MCP Tool Mesh`, `Reusable Automation Recipes`

### Architectural Mechanics
Developed and battle-tested internally at Block, Goose is built around an **extension-first architecture** powered entirely by the Model Context Protocol (MCP). Rather than hardcoding tools, every capability (filesystem, Git, Jira, Datadog, Postgres, browser) is an MCP server. Its standout feature is **Goose Recipes**: teams can package a complex multi-step engineering workflow (prompt + required MCP extensions + parameters) into a shareable YAML recipe that any engineer or CI runner can execute deterministically.

### Signature Workflow
Standardizing repeatable team migrations (e.g., upgrading a deprecated internal SDK, updating database schemas, and opening a formatted PR) as a version-controlled Recipe.

---

## 5. Plandex
* **Link:** [plandex.ai](https://plandex.ai/)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ Long-Horizon Planner | Patterns: `Spec-Driven Development`, `Staged Sandbox Branching`

### Architectural Mechanics
Plandex decouples **planning and staging** from the local working tree. When executing a multi-step plan, Plandex applies all intermediate file modifications inside a managed staging sandbox rather than mutating your local files immediately. It includes dedicated version control for the agent's reasoning (`plandex rewind` and `plandex branches`), allowing developers to fork an agent's implementation plan, compare two architectural approaches, and only `plandex apply` once the full diff passes verification.

---

## 6. Google Gemini CLI & Antigravity CLI
* **Link:** [github.com/google-gemini/gemini-cli](https://github.com/google-gemini/gemini-cli)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ / CI Scriptable | Patterns: `Long-Context Monorepo Ingestion`, `Live Search Grounding`

### Architectural Mechanics
Built in TypeScript under Apache 2.0, Gemini CLI (and its agent-first successor, Antigravity CLI) leverages a **1-million-token context window** alongside **native Google Search grounding**. While most agents rely purely on static training weights or local files, Gemini CLI can query live web documentation when encountering breaking changes in newly released third-party libraries, and ingest entire mid-sized repositories or massive log dumps in a single pass.

---

## 7. Sourcegraph Amp
* **Link:** [ampcode.com](https://ampcode.com/)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ + Team Shared Threads | Patterns: `Knowledge-Graph Grounding`

### Architectural Mechanics
Amp is Sourcegraph's dedicated CLI coding agent. It pairs frontier LLM reasoning with Sourcegraph's enterprise **code intelligence graph** (precise cross-repository symbol definitions, references, and dependency trees). It also treats agent transcripts as collaborative artifacts: engineers can share live or completed agent threads across the team to audit how a complex change was architected.

---

## 8. Hermes (by Nous Research) & OpenCode
* **Hermes ([hermes-agent.org](https://hermes-agent.org/)):** An open-source TUI agent featuring a **persistent self-improving memory loop** that synthesizes reusable "skills" from past debugging sessions and dispatches commands across local shells or remote cloud sandboxes (Daytona, Modal, SSH).
* **OpenCode ([opencode.ai](https://opencode.ai/) / [github.com/sst/opencode](https://github.com/sst/opencode)):** A privacy-first, Go/TypeScript terminal agent utilizing a **client-server architecture** with native Language Server Protocol (LSP) integration. Because the engine runs as a local server, developers can attach multiple TUI clients or switch seamlessly across 75+ LLM providers (including local Ollama/vLLM models) without losing session state.