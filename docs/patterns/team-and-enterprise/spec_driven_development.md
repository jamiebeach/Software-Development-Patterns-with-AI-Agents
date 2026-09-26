# Pattern: Spec-Driven Development (SDD)

* **Category:** Team & Enterprise / High Autonomy
* **Autonomy Level:** $L_2 \rightarrow L_3$ (See [`docs/taxonomy.md`](../../taxonomy.md))
* **Topology:** $1:1$ Synchronous or $1:N$ Swarm
* **Maturity Level:** Production / High Adoption
* **Primary Actors:** Human Architect / Tech Lead, Spec Author Agent, Planner Agent, Task Execution Agent, Evaluator / Test Runner Agent
* **Key Exemplars:** GitHub Spec Kit (`specify`), Amazon Kiro, Tessl, Claude Code, Aider

---

## 1. Intent & Overview

**Spec-Driven Development (SDD)** is an agentic engineering pattern where code generation is strictly decoupled from requirements elicitation through an explicit, version-controlled contract (the *Specification*).

Instead of prompting an agent directly with informal instructions to edit code, the human engineer collaborates with an agent to write machine-verifiable requirements, interface boundaries, and behavioral invariants first. Once the specification and localization plan pass validation gates, execution agents generate code iteratively against that spec until all acceptance criteria and test harnesses pass.

```text
+-----------------------------------------------------------------------------------+
|                        GOVERNANCE LAYER: constitution.md                          |
|         (Immutable architectural principles, layer rules, stack invariants)       |
+-----------------------------------------------------------------------------------+
                                          |
                                          v
+-----------------------------------------------------------------------------------+
|                               PHASE 1: SPECIFY                                    |
|   Human Intent  -->  Spec Author Agent  -->  [spec.md / types / test fixtures]    |
+-----------------------------------------------------------------------------------+
                                          |
                                   [Human Gate 1]
                                          v
+-----------------------------------------------------------------------------------+
|                          PHASE 2: PLAN & LOCALIZE                                 |
|   Spec File     -->  Planner Agent      -->  [File Localization + Task DAG]       |
+-----------------------------------------------------------------------------------+
                                          |
                                   [Human Gate 2]
                                          v
+-----------------------------------------------------------------------------------+
|                                PHASE 3: EXECUTE                                   |
|   +---------------------------------------------------------------------------+   |
|   |  Subtask N  -->  Execution Agent  -->  Run F2P & P2P Test Suites          |   |
|   |                         ^                         |                       |   |
|   |                         |------ [Failures] <------+                       |   |
|   +---------------------------------------------------------------------------+   |
|                                         | [All Pass]                              |
|                                         v                                         |
|                                  Verified Diff / PR                               |
+-----------------------------------------------------------------------------------+
```

---

## 2. The Problem It Solves

Empirical benchmarks (`RACE-bench`, `EvoClaw`) and practitioner field studies show that when autonomous agents are given open-ended feature prompts (e.g., *"Refactor our billing pipeline to support metered seat add-ons"*), failure modes compound rapidly:

1. **The Waterfall Degradation of Reasoning:** `RACE-bench` proved that while frontier models score $>9.2/10$ on high-level intent comprehension, their accuracy drops precipitously during **file localization** and **step decomposition**. Without an explicit intermediate planning gate, early localization errors cascade into broken implementations.
2. **Context Drift & Architectural Hallucination:** Agents make silent, uncoordinated architectural decisions to satisfy local tests, frequently breaking unstated conventions or API contracts.
3. **Review Fatigue:** Humans are forced to review $500+$ line diffs without knowing what intermediate assumptions the agent made.
4. **Sycophantic Test Tampering:** When fixing bug A causes test B to fail, an unconstrained agent will often modify test B or weaken core semantics to make the suite green.

---

## 3. The Three Maturity Tiers of SDD

As documented in industry field analyses (Boeckeler / Martin Fowler, GitHub Spec Kit, Tessl), teams adopt SDD across three distinct maturity tiers depending on the permanence of the specification artifact:

| Tier | Designation | Lifecycle of the Spec | Source of Truth | Best Suited For |
| :--- | :--- | :--- | :--- | :--- |
| **Tier 1** | **Spec-First** | Ephemeral. Drafted before coding to align human and agent in a single session, then discarded or archived once the PR merges. | Application Code | Medium-sized features, isolated refactorings, solo developers. |
| **Tier 2** | **Spec-Anchored** | Persistent. Specs live permanently in `specs/` alongside a project `constitution.md`. Future modifications begin by updating the spec and running `/specify -> /plan -> /tasks`. | Co-equal (`specs/` + `src/` verified via CI) | Multi-developer teams, core domain services, long-lived enterprise repos. |
| **Tier 3** | **Spec-as-Source** | Primary artifact. Humans *only* author and version the specification and test contracts. Application code is treated as a compiled build output that is regenerated on demand and never hand-edited. | The Specification exclusively | Self-contained microservices, deterministic data pipelines, greenfield modules. |

---

## 4. Core Principles

1. **Inverted Authority:** The specification and test harness are immutable during the execution phase. If tests fail, the execution agent is prohibited from altering `spec.md` or test assertions; only the implementation code may change.
2. **Constitutional Grounding:** Every feature spec is evaluated against a repository-wide [`constitution.md`](../reliability-and-state/operational-policy-files.md) so individual specs do not introduce conflicting libraries or architectural drift.
3. **Right-Sizing Gate (Avoiding the Sledgehammer):** Not every change belongs in a 4-stage SDD pipeline. Teams enforce a triage threshold: trivial bug fixes and 1-file tweaks bypass formal spec generation to avoid bureaucratic markdown bloat.
4. **Dual-Suite Verification (F2P + P2P):** Execution is gated by both **Fail-to-Pass (F2P)** acceptance tests (proving the new spec behavior works) and **Pass-to-Pass (P2P)** regression suites (proving existing system invariants remain intact).

---

## 5. Lifecycle & State Machine

```text
               +------------------+
               | 0. TRIAGE GATE   | ---------> [Trivial Fix] --> Direct Edit + P2P Tests
               +------------------+
                        | [Non-Trivial Feature]
                        v
               +------------------+
               | 1. SPECIFY       | <--------- Human intent + constitution.md
               +------------------+
                        |
                [Human Sign-Off]
                        v
               +------------------+
               | 2. PLAN &        | <--------- Agent localizes target files, interfaces,
               |    LOCALIZE      |            and generates atomic Task DAG
               +------------------+
                        |
                 [Plan Review]
                        v
       +-----> +------------------+
       |       | 3. EXECUTE       | <--------- Agent edits src/ to satisfy subtask N
       |       +------------------+
       |                |
 [Fix Error]            v
       |       +------------------+
       +------ | 4. EVALUATE      | <--------- Run static checks, F2P spec tests,
               +------------------+            and P2P regression suite
                        |
                   [All Pass]
                        v
               +------------------+
               | 5. COMPLETED     | <--------- Commit code + spec, open PR
               +------------------+
```

---

## 6. Concrete Walkthrough

### Scenario
Adding an idempotent rate-limiting middleware to a Fastify/Node.js API service based on Redis token buckets.

### Step 1: The Generated Specification (`specs/SPEC-004-rate-limiter.md`)

```markdown
# SPEC-004: Redis-Backed Token Bucket Rate Limiter
* **Constitutional Compliance:** Complies with Article II (explicit Redis timeout budget + OpenTelemetry span).

## 1. Interface Contract
- Function Signature: `createRateLimiter(options: RateLimiterOptions): FastifyPluginAsync`
- Configuration Object:
  ```typescript
  interface RateLimiterOptions {
    redisClient: Redis;
    capacity: number;         // Max tokens in bucket
    refillRatePerSec: number; // Tokens added per second
    timeoutMs?: number;       // Default: 50ms (Constitutional Article II)
    keyGenerator: (req: FastifyRequest) => string;
  }
  ```

## 2. Invariants & Behaviors
- [x] Must return HTTP `429 Too Many Requests` when bucket balance is less than 1.
- [x] Must include headers in all responses:
  - `X-RateLimit-Limit`: Maximum bucket capacity.
  - `X-RateLimit-Remaining`: Floor of currently available tokens.
  - `X-RateLimit-Reset`: Milliseconds until bucket refills to capacity.
- [x] Redis failures or timeouts (> `timeoutMs`) must fail-open (log error, allow request through with `X-RateLimit-Degraded: true`).
- [x] Concurrency requirement: Redis operations must be evaluated atomically via Lua script.

## 3. Test Fixture Contract
- **Fail-to-Pass (F2P) Spec Suite:** `tests/middleware/rate-limiter.spec.ts`
- **Pass-to-Pass (P2P) Regression Suite:** `tests/integration/api-pipeline.spec.ts`
```

---

### Step 2: System Prompting the Execution Agent

```text
SYSTEM:
You are an implementation agent executing against a verified specification.

RULES:
1. You may ONLY modify files in `src/middleware/rate-limiter/`.
2. You CANNOT edit `specs/SPEC-004-rate-limiter.md`, `constitution.md`, or any file in `tests/`.
3. After making changes, run `pnpm vitest run tests/middleware/rate-limiter.spec.ts`.
4. Once F2P tests pass, run `pnpm vitest run tests/integration/api-pipeline.spec.ts` to verify zero P2P regressions.
5. Do not consider the task complete until both suites pass with 0 errors and `pnpm biome check` passes.
```

---

### Step 3: Self-Correction Loop Log

```text
[EXECUTION]: Generating src/middleware/rate-limiter/index.ts...
[SHELL]: pnpm vitest run tests/middleware/rate-limiter.spec.ts

FAIL tests/middleware/rate-limiter.spec.ts
  ● RateLimiter Middleware › Fail-open behavior on timeout
    expect(res.headers['x-ratelimit-degraded']).toBe('true')
    Expected: "true"
    Received: undefined
      at Object.<anonymous> (tests/middleware/rate-limiter.spec.ts:42:54)

[FEEDBACK INGESTED]: Spec invariant #3 violated. Caught Redis timeout exception did not attach degradation header.
[EXECUTION]: Modifying src/middleware/rate-limiter/index.ts catch block...
[SHELL]: pnpm vitest run tests/middleware/rate-limiter.spec.ts

PASS tests/middleware/rate-limiter.spec.ts (4 passed)
[SHELL]: pnpm vitest run tests/integration/api-pipeline.spec.ts
PASS tests/integration/api-pipeline.spec.ts (18 passed)
```

---

## 7. Anti-Patterns & Pitfalls

| Anti-Pattern | Manifestation / Symptom | Remediation |
| :--- | :--- | :--- |
| **The Sledgehammer on a Nut** | Running a full `/specify -> /plan -> /tasks` SDD pipeline for a 5-line bug fix, generating 4 user stories and 1,000 words of markdown overhead. | Enforce a **Step 0 Triage Gate**: changes touching $\le 2$ files with existing test coverage skip straight to execution. |
| **Spec Dilution** | Allowing the execution agent to edit the spec or test file when it gets stuck on an implementation hurdle. | Mount `specs/` and `tests/` as read-only in the agent sandbox or enforce git pre-commit hooks that reject spec checksum changes during execution runs. |
| **Skipping the Localization Plan** | Jumping directly from high-level requirements in `spec.md` to code generation without verifying which files and functions will be touched. | Mandate Phase 2 (`PLAN & LOCALIZE`) so humans can catch wrong file targets before code is generated (`RACE-bench` mitigation). |
| **Monolithic Specification** | Writing a 2,000-line spec covering an entire subsystem at once. | Break specs down into atomic domain slices ($3\text{--}5$ file edits per spec) that map cleanly into task DAGs. |

---

## 8. Half-Life & Future Trajectory

* **Current State:** Spec-First and Spec-Anchored workflows (via tools like GitHub Spec Kit, Kiro, and Claude Code plan modes) are the dominant enterprise guardrails for $L_2 \rightarrow L_3$ coding agents.
* **6-to-12 Month Trajectory:** Natural-language markdown specs still suffer from residual semantic ambiguity. We are seeing a steady shift toward **Hybrid Formal Specifications**—combining brief markdown intent with machine-verified property-based tests, OpenAPI/Protobuf schemas, and lightweight formal verification harnesses (e.g., Dafny, Lean 4) where the compiler kernel guarantees invariant compliance.

---

## 9. Related Patterns

* [**Operational Policy & Constitutional Files**](../reliability-and-state/operational-policy-files.md)**:** Provides the repository-wide `constitution.md` and `AGENTS.md` rules that govern every individual specification.
* [**Issue-Driven Agent Mesh (Beads)**](../multi-agent-orchestration/issue-driven-mesh.md)**:** Phase 2 of SDD decomposes the validated spec into atomic, git-tracked task units ("beads") for parallel swarm execution.
* [**Hierarchical Supervisor (Gas Town)**](../multi-agent-orchestration/hierarchical-supervisor.md)**:** A Mayor or Witness agent uses the spec as the acceptance rubric against which Worker outputs are verified.