# Pattern: Operational Policy & Constitutional Files (`AGENTS.md` / `constitution.md`)

* **Category:** Reliability & State / Team & Enterprise
* **Autonomy Level:** $L_1 \rightarrow L_4$ (Foundational across all tiers; see [`docs/taxonomy.md`](../../taxonomy.md))
* **Topology:** $1:1$ Synchronous, $1:N$ Swarm, $M:N$ Mesh
* **Maturity Level:** Production / Industry Standard
* **Primary Actors:** Human Maintainer, Coding Agent Runtime, Reviewer / Linter Agent
* **Key Exemplars:** Linux Foundation `AGENTS.md` Standard, Claude Code (`CLAUDE.md`), GitHub Spec Kit (`constitution.md`), Cursor (`.cursor/rules`)

---

## 1. Intent & Overview

**Operational Policy & Constitutional Files** externalize repository-specific operational commands, architectural invariants, escalation boundaries, and closure criteria into version-controlled files automatically ingested at the root of every agent session.

Rather than relying on human developers to repeatedly paste build flags, directory conventions, or safety rules into chat prompts, repositories expose a machine-oriented operational contract (`AGENTS.md` for execution mechanics and `constitution.md` for non-negotiable architectural invariants).

```text
+-------------------------------------------------------------------------+
|                        REPOSITORY ROOT / SUB-PACKAGE                    |
|                                                                         |
|  +-----------------------------------+  +----------------------------+  |
|  |            AGENTS.md              |  |      constitution.md       |  |
|  | - Exact build/test/lint commands  |  | - Architectural invariants |  |
|  | - Task-scoped rules (When X...)   |  | - Forbidden dependencies   |  |
|  | - Ambiguity escalation gates      |  | - Domain boundaries        |  |
|  | - Destructive recovery bans       |  | - Security requirements    |  |
|  +-----------------------------------+  +----------------------------+  |
+-------------------------------------------------------------------------+
                     |                                   |
                     +-----------------+-----------------+
                                       |
                        [Auto-Injected at Session Boot]
                                       v
+-------------------------------------------------------------------------+
|                          AGENT EXECUTION LOOP                           |
|  1. Check Ambiguity Gate  --> Stop & Ask if requirements underspecified |
|  2. Execute Scoped Edits  --> Respect constitutional module boundaries  |
|  3. Run Closure Commands  --> Verify exit code 0 before commit/PR       |
+-------------------------------------------------------------------------+
```

---

## 2. The Problem It Solves

Empirical studies of over 2,500 repositories and agent benchmarks (`Ambig-SWE`, ICLR 2026) reveal three critical failure modes when agents operate without structured operational policy files:

1. **The Prose Blindness Problem:** Developers frequently write `AGENTS.md` or `CLAUDE.md` files like human onboarding docs—filled with narrative paragraphs about company history or vague advice (*"be careful when touching the billing module"*). Benchmarks demonstrate that narrative prose in policy files produces **zero measurable improvement** in task resolution while burning context tokens.
2. **Silent Assumption Hallucination:** When handed an underspecified prompt, coding agents almost never pause to ask clarifying questions by default; they silently hallucinate architectural assumptions. Explicitly defining ambiguity escalation gates in policy files improves task accuracy by up to $74\%$.
3. **Destructive Recovery Loops:** When an agent encounters a stubborn test failure, type error, or lockfile conflict, its default reward-seeking behavior is to take the path of least resistance—deleting the failing test assertion, adding `@ts-ignore`, modifying linter configs, or wiping `package-lock.json`.

---

## 3. Core Principles

1. **Command-First over Narrative Prose:** Every instruction that can be expressed as an exact, copy-pasteable CLI invocation (`pnpm --filter @app/billing test:unit --run`) must be written as a command rather than a prose explanation.
2. **Task-Scoped Conditional Activation:** Organize instructions by operational phase (`WHEN LOCALIZING`, `WHEN IMPLEMENTING`, `WHEN VERIFYING`) or use directory-scoped policy files (`src/payments/AGENTS.md`) so agents only load rules relevant to their active sub-tree.
3. **Explicit Ambiguity Escalation:** Mandate exact conditions under which an agent must halt execution and query the human (or supervisor agent) rather than guessing.
4. **Deterministic Closure Definitions:** Define what "done" means in terms of zero-exit-code commands, not subjective code completeness.
5. **Hard "Never" Invariants (Anti-Tampering):** Explicitly forbid destructive recovery behaviors (e.g., mutating linter rules, skipping tests, or altering migrations).

---

## 4. Separation of Concerns: `AGENTS.md` vs. `constitution.md`

Mature agentic repositories split operational mechanics from architectural governance:

| Artifact | Purpose | Typical Contents | Primary Consumer |
| :--- | :--- | :--- | :--- |
| **`AGENTS.md`** (or `CLAUDE.md`) | **Operational Mechanics:** *How to build, test, navigate, and verify this repo.* | CLI commands, test flags, directory map, closure checklists, escalation triggers. | Execution Agents, Test Runners |
| **`constitution.md`** | **Architectural Governance:** *Non-negotiable system laws and invariants.* | Layer boundaries, forbidden libraries, state management laws, security invariants. | Spec Author Agents, Planner Agents, Critic Agents |

---

## 5. Concrete Walkthrough

### Artifact 1: A High-Signal, Command-First `AGENTS.md`

```markdown
# AGENTS.md — Operational Contract

## 1. Environment & Fast-Feedback Commands
Do NOT run the full test suite during iterative edits. Use scoped commands:
- **Typecheck single package:** `pnpm --filter <pkg-name> typecheck`
- **Run single test file:** `pnpm vitest run <path/to/file.test.ts>`
- **Lint & format check:** `pnpm biome check --write <path/to/file.ts>`
- **Regenerate DB types:** `pnpm db:generate` (Run after any `schema.prisma` edit)

## 2. Ambiguity & Escalation Gate (MANDATORY)
Before modifying any code, evaluate the prompt/task against these triggers. HALT and ask for clarification if:
1. A new API endpoint or schema change does not specify error response codes or auth scope.
2. Fixing a bug requires altering a public interface exported in `packages/sdk/src/index.ts`.
3. More than one valid state-management pattern exists for the target feature and no spec file is linked.

## 3. Forbidden Recovery Actions ("NEVER" List)
When a test, build, or linter command fails, you are strictly PROHIBITED from:
- Adding `// @ts-ignore`, `// @ts-expect-error`, or `any` casts to silence compiler errors.
- Modifying `biome.json`, `tsconfig.json`, or `.eslintrc` to pass a check.
- Deleting, `.skip`-ing, or weakening existing assertions in `*.test.ts` or `*.spec.ts`.
- Running `git push --force` or deleting lockfiles (`pnpm-lock.yaml`).

## 4. Definition of Done (Closure Gate)
A task is ONLY complete when all three commands return exit code `0`:
1. `pnpm turbo run typecheck --filter=...[origin/main]`
2. `pnpm turbo run lint --filter=...[origin/main]`
3. `pnpm turbo run test:unit --filter=...[origin/main]`
```

---

### Artifact 2: A Project `constitution.md` (Spec-Anchored Governance)

```markdown
# Project Constitution — Architectural Invariants

All specifications (`specs/*.md`), implementation plans, and code diffs must obey these invariants. Any violation will be rejected by the Critic/CI gate.

## Article I: Domain & Layer Isolation
1. `packages/domain/` is pure TypeScript. It MUST NOT import from `packages/infra/`, HTTP frameworks (`fastify`), or ORM clients (`prisma`).
2. All database mutations must occur inside explicit unit-of-work transaction wrappers defined in `packages/infra/db/tx.ts`.

## Article II: Error Handling & Observability
1. Never throw raw `Error` instances across service boundaries. All domain functions must return `Result<T, DomainError>` from `neverthrow`.
2. Every external I/O call (Redis, Stripe, Postgres) must specify an explicit timeout budget and emit structured OpenTelemetry spans.

## Article III: Dependency Discipline
1. Zero new runtime dependencies (`dependencies` in `package.json`) may be added without an approved ADR or explicit authorization in the task spec.
```

---

## 6. Anti-Patterns & Pitfalls

| Anti-Pattern | Manifestation / Symptom | Remediation |
| :--- | :--- | :--- |
| **The Encyclopedia `AGENTS.md`** | A 2,500-line file explaining every microservice, historical migration, and coding style preference. | Keep root `AGENTS.md` under $150$ lines. Move package-specific rules into nested `packages/<name>/AGENTS.md` files. |
| **Vague Admonitions** | Writing rules like *"Write clean, modular, well-tested code and follow SOLID principles."* | Replace subjective adjectives with deterministic linters, structural AST tests (e.g., `dependency-cruiser`), or exact CLI gates. |
| **Stale Command Drift** | Repository migrates from `jest` to `vitest`, but `AGENTS.md` still instructs agents to run `npm run jest`. | Validate commands listed in `AGENTS.md` inside CI so broken documentation fails the build. |
| **Missing Negative Constraints** | Telling the agent how to run tests, but failing to forbid test modification. | Always pair test-execution instructions with an explicit ban on altering existing Pass-to-Pass (P2P) test assertions. |

---

## 7. Half-Life & Future Trajectory

* **Current State:** `AGENTS.md` has rapidly consolidated as the cross-vendor standard (stewarded under the Linux Foundation's Agentic AI Foundation), replacing fragmented vendor files.
* **6-to-12 Month Trajectory:** Static markdown policy files are beginning to evolve into **Executable Policy Hooks** (such as declarative Harness Hook Languages and pre-tool-call interceptors) where the agent runtime automatically blocks forbidden file edits or shell commands at the system-call boundary rather than relying on prompt compliance.

---

## 8. Related Patterns

* [**Spec-Driven Development (SDD)**](../team-and-enterprise/spec-driven-development.md)**:** Uses `constitution.md` as the immutable rulebook against which feature specifications and plans are validated.
* [**Issue-Driven Agent Mesh (Beads)**](../multi-agent-orchestration/issue-driven-mesh.md)**:** Ephemeral workers rely on `AGENTS.md` to boot up with zero prior conversational history and immediately know how to build and verify their assigned bead.