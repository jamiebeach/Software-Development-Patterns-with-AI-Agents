# Catalog: IDE-Integrated & Protocol-Driven Agents

IDE-Integrated Agents embed autonomous planning, terminal execution, and multi-file diff orchestration directly inside the graphical editor. The category has bifurcated into **AI-Native Editor Forks** (Cursor, Windsurf, Kiro), **Autonomous Editor Extensions** (Cline, Roo Code), and **Open Protocol Clients** (Zed via ACP).

---

## Comparative Architecture Matrix

| Tool | Form Factor | Core Orchestration Engine | Context & Verification Innovation | Primary Pattern |
| :--- | :--- | :--- | :--- | :--- |
| **Cursor** | VS Code Fork | Composer / Background Agents | **Shadow Workspace** (hidden LSP/linter verification before rendering diffs) | `Vibe Coding` / `Compiler-Gated Critic` |
| **Windsurf** | VS Code Fork | Cascade Engine | **Implicit Flow Tracking** (tracks human cursor, terminal, and clipboard state in real time) | `Human-AI Shared State` |
| **Amazon Kiro** | VS Code Fork | Dual Vibe / Spec Engine | **Native Spec-Anchored Pipeline** (`requirements.md` $\rightarrow$ `design.md` $\rightarrow$ `tasks.md`) | `Spec-Driven Development (SDD)` |
| **Cline** | VS Code Extension | Plan & Act Loop | **Strict Stepwise Human-in-the-Loop Gate** + Headless Browser & MCP integration | `Human-in-the-Loop Review Gate` |
| **Roo Code** | VS Code Extension | Role-Based Multi-Mode Engine | **Persona Segregation** (Architect, Code, Debug, Ask, Orchestrator Boomerang modes) | `Role-Segregated Sub-Agents` |
| **Zed (via ACP)** | Native Rust IDE | Agent Client Protocol (ACP) | **Decoupled Editor-Agent Protocol** (plug Claude Code, Codex, or Gemini CLI into native UI) | `Protocol-Decoupled ACI` |
| **Copilot Workspace** | Cloud / Web IDE | Issue-to-PR Pipeline | **Issue-First Lifecycle** (starts from GitHub Issue $\rightarrow$ Spec $\rightarrow$ Plan $\rightarrow$ Cloud Sandbox) | `Issue-Driven Autonomy` |

---

## 1. Cursor
* **Link:** [cursor.com](https://www.cursor.com/)
* **Taxonomy Profile:** Autonomy: $L_1 - L_3$ | Topology: $1:1$ + Background Cloud Agents | Patterns: `Vibe Coding`, `Compiler-Gated Critic`, `Operational Policy Files` (`.cursor/rules/*.mdc`)

### Architectural Mechanics
Cursor combines fast speculative edits (custom small models predicting where your cursor will move next and applying multi-line diffs) with its **Composer Agent**. Two architectural choices set it apart:
1. **The Shadow Workspace:** When an agent generates code, Cursor spawns an invisible background window running the language server (LSP). The agent queries kernel/LSP diagnostics silently and fixes type errors *before* the human ever sees the proposed diff.
2. **Scoped `.mdc` Rules:** Instead of a single monolithic prompt file, Cursor supports path-scoped rule files (`.cursor/rules/backend.mdc` triggered only when touching `src/api/**/*.ts`), keeping context pollution low.

### Failure Modes & Trade-offs
* **Over-Eager "Vibe" Drift:** In long Composer sessions without a spec file, rapid "Accept All" clicking leads to the classic `EvoClaw` regression snowball—fixing immediate UI behavior while silently breaking underlying domain boundaries.

---

## 2. Amazon Kiro
* **Link:** [kiro.dev](https://kiro.dev/)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ Spec-Guided | Patterns: `Spec-Driven Development (Spec-Anchored)`, `Event-Driven Agent Hooks`

### Architectural Mechanics
Amazon Kiro is the flagship IDE built explicitly around the **Spec-Anchored** maturity tier of Spec-Driven Development. On initialization, Kiro generates persistent steering files (`product.md`, `tech.md`, `structure.md`). When building a feature in **Spec Mode**, Kiro enforces a three-artifact state machine before touching application code:
1. `requirements.md`: Translates user prompts into EARS (Easy Approach to Requirements Syntax) acceptance criteria (`WHEN <trigger> THE SYSTEM SHALL <response>`).
2. `design.md`: Generates sequence diagrams, data models, and interface contracts.
3. `tasks.md`: Produces an interactive checklist of discrete engineering tasks tied back to specific requirements.
Kiro also introduces **Agent Hooks**—background triggers that automatically run an agent pass (e.g., updating unit tests or documentation) whenever a developer saves a file.

### Failure Modes & Trade-offs
* **The "Sledgehammer on a Nut" Problem:** As documented in field studies, running Kiro's full 3-document Spec Mode on minor bug fixes creates substantial markdown review overhead. Developers must actively switch between Kiro's *Vibe Mode* (for small fixes) and *Spec Mode* (for architectural features).

---

## 3. Cline & Roo Code (The Open-Source VS Code Ecosystem)
* **Links:** [cline.bot](https://cline.bot/) | [roocode.com](https://roocode.com/)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: $1:1$ / $1:N$ Sub-Task Boomerang | Patterns: `Human-in-the-Loop Gate`, `Role-Segregated Sub-Agents`

### Architectural Mechanics
While both live in the VS Code sidebar and share a common open-source lineage, they have evolved into two distinct architectural philosophies:
* **Cline (Auditable Plan & Act):** Focuses on enterprise governance and transparent two-phase execution (**Plan Mode** vs. **Act Mode**). Every terminal command, file edit, and headless browser interaction (capturing screenshots and console logs to debug visual UI regressions) can be gated behind explicit developer approval.
* **Roo Code (Role-Segregated Multi-Mode Orchestration):** Forked from Cline to prioritize modular workflow automation. Roo restricts tool permissions by **Mode**:
  * *Architect Mode* can ONLY read code and write `.md` spec files (physically prevented from editing source code).
  * *Code Mode* has full write and terminal permissions.
  * *Orchestrator (Boomerang) Mode* breaks a complex goal into subtasks, spawns isolated child sessions in the appropriate mode, and receives only a concise completion summary back—preventing parent context window exhaustion.

---

## 4. Windsurf (Cascade Engine)
* **Link:** [windsurf.com](https://windsurf.com/)
* **Taxonomy Profile:** Autonomy: $L_2$ | Topology: $1:1$ Tightly Coupled | Patterns: `Shared Human-Agent State`

### Architectural Mechanics
Windsurf's **Cascade** engine models development as a single shared timeline between the human and the agent. If the human manually edits three lines of a function or runs a failing command in the terminal, Cascade automatically ingests that action as an implicit state update without requiring the developer to explain what they just changed or `@mention` the modified file.

---

## 5. Zed & The Agent Client Protocol (ACP)
* **Link:** [zed.dev/acp](https://zed.dev/acp)
* **Taxonomy Profile:** Autonomy: $L_2 - L_3$ | Topology: Modular Client-Server | Patterns: `Protocol-Decoupled ACI`

### Architectural Mechanics
Just as the Language Server Protocol (LSP) decoupled compilers from editors, and MCP decoupled external tools from LLMs, Zed created the open-source **Agent Client Protocol (ACP)** (now co-supported by JetBrains). Instead of locking developers into a proprietary editor agent, ACP allows any external CLI agent—such as **Claude Code**, **OpenAI Codex CLI**, or **Gemini CLI**—to run as a background subprocess and render rich, multi-file diffs, tool calls, and review panels natively inside Zed's high-performance Rust UI.