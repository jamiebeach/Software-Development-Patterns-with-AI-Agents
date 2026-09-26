# Pattern: Spec-Driven Development (SDD)

* **Category:** Team & Enterprise / High Autonomy
* **Maturity Level:** Production / High Adoption
* **Primary Actors:** Human Architect / Tech Lead, Spec Author Agent, Task Execution Agent, Evaluator / Test Runner Agent
* **Key Tools:** Claude Code, GitHub Workspace, Tessl, Aider, Cursor Composer

---

## 1. Intent & Overview

**Spec-Driven Development (SDD)** is an agentic engineering pattern where code generation is strictly decoupled from requirements elicitation through an explicit, version-controlled contract (the *Specification*). 

Instead of prompting an agent directly with informal instructions to edit code, the human engineer collaborates with an agent to write machine-verifiable requirements, interface boundaries, and behavioral invariants first. Once the specification passes validation gates, execution agents generate code iteratively against that spec until all acceptance criteria and test harnesses pass.

```text
+-----------------------------------------------------------------------------------+
|                               PHASE 1: SPECIFY                                    |
|   Human Intent  -->  Spec Author Agent  -->  [spec.md / types / fixtures]         |
+-----------------------------------------------------------------------------------+
                                          |
                                   [Human Gate 1]
                                          v
+-----------------------------------------------------------------------------------+
|                                 PHASE 2: PLAN                                     |
|   Spec File     -->  Planner Agent      -->  [Atomic Task DAG / Work Breakdown]   |
+-----------------------------------------------------------------------------------+
                                          |
                                   [Human Gate 2]
                                          v
+-----------------------------------------------------------------------------------+
|                                PHASE 3: EXECUTE                                   |
|   +---------------------------------------------------------------------------+   |
|   |  Subtask N  -->  Execution Agent  -->  Run Tests/Linter                   |   |
|   |                         ^                     |                           |   |
|   |                         |---- [Failures] <----+                           |   |
|   +---------------------------------------------------------------------------+   |
|                                         | [All Pass]                              |
|                                         v                                         |
|                                  Verified Diff / PR                               |
+-----------------------------------------------------------------------------------+
```

---

## 2. The Problem It Solves

When autonomous agents are given open-ended instructions (e.g., *"Refactor our billing pipeline to support metered seat add-ons"*), failure modes compound rapidly:

1. **Context Drift & Architectural Hallucination:** Agents make silent, uncoordinated architectural decisions to satisfy local tests, frequently breaking unstated conventions or API contracts.
2. **Review Fatigue:** Humans are forced to review 500+ line diffs without knowing what intermediate assumptions the agent made.
3. **Circular Fixes:** When fixing bug A, the agent modifies existing tests or alters core semantics, creating regressions elsewhere.
4. **Moving Goalposts:** Unstructured prompts allow the agent to treat its own generated code as the ground truth rather than adhering to system requirements.

---

## 3. Core Principles

1. **Inverted Authority:** The specification is immutable during execution. If tests fail, the agent is prohibited from altering the spec or test assertions; only the implementation may change.
2. **Contract First, Code Second:** APIs, database schemas, edge-case tables, and acceptance criteria must exist as discrete files in the repository before any application code is touched.
3. **Spec-to-Test Determinism:** Specifications must include or translate directly into executable test fixtures (e.g., Cucumber/Gherkin, Vitest/Pytest suites, OpenAPI schemata).
4. **Bimodal Human Interaction:** High human cognitive load during the *Specification* phase; low-friction review or passive observation during the *Execution* phase.

---

## 4. Prerequisites & Environmental Setup

Before applying SDD in an agentic pipeline:

* **Strict Tool Boundaries:** The execution agent’s tool configuration must restrict write access exclusively to the target source directories and mock folders—explicitly preventing writes to `spec.md` or test contracts.
* **Deterministic Test Harness:** Fast, isolated unit or integration test runners (target execution time < 30 seconds) that produce structured output (`stdout`, JSON test reports).
* **Static Analysis & Linters:** Type checkers (`tsc`, `mypy`, `cargo check`) and linters configured to run via agent CLI calls.
* **Ephemeral Workspaces:** Support for Git worktrees or isolated container runtimes so the agent can execute without contaminating the primary working branch.

---

## 5. Lifecycle & State Machine

```text
               +---------------+
               | 1. DRAFTING   | <--------- Human provides feature intent
               +---------------+
                       |
                       v
               +---------------+
               | 2. VALIDATING | <--------- Agent generates contracts, types, & mocks
               +---------------+
                       |
               [Human Sign-Off]
                       v
               +---------------+
               | 3. PLANNING   | <--------- Agent breaks spec into dependency graph
               +---------------+
                       |
               [Plan Review]
                       v
       +-----> +---------------+
       |       | 4. EXECUTING  | <--------- Agent edits code to satisfy step N
       |       +---------------+
       |               |
 [Fix Error]           v
       |       +---------------+
       +------ | 5. EVALUATING | <--------- Run static checks, linter, & test suite
               +---------------+
                       |
                  [All Pass]
                       v
               +---------------+
               | 6. COMPLETED  | <--------- Git commit, create PR
               +---------------+
```

### State Definitions

1. **`DRAFTING`**: Human provides business context and high-level requirements. The spec-authoring agent generates an initial `spec.md` or RFC draft.
2. **`VALIDATING`**: The agent inspects existing codebase interfaces, generates strict TypeScript/Python types, API schemas, and test matrices. The human approves the contract.
3. **`PLANNING`**: The planner agent analyzes the spec against the codebase and produces an atomic, sequential task list.
4. **`EXECUTING`**: An isolated worker agent takes an assigned subtask, modifies application files, and prepares a candidate diff.
5. **`EVALUATING`**: Automated feedback loop. The agent runs `lint`, `typecheck`, and `test`. If any assertion fails, control loops back to `EXECUTING` with the terminal output appended to context.
6. **`COMPLETED`**: Worktree changes are committed, and the PR references both the implementation diff and the original `spec.md`.

---

## 6. Concrete Walkthrough

### Scenario
Adding an idempotent rate-limiting middleware to a Fastify/Node.js API service based on Redis token buckets.

---

### Step 1: The Generated Specification (`specs/SPEC-004-rate-limiter.md`)

```markdown
# SPEC-004: Redis-Backed Token Bucket Rate Limiter

## 1. Interface Contract
- Function Signature: `createRateLimiter(options: RateLimiterOptions): FastifyPluginAsync`
- Configuration Object:
  ```typescript
  interface RateLimiterOptions {
    redisClient: Redis;
    capacity: number;         // Max tokens in bucket
    refillRatePerSec: number; // Tokens added per second
    keyGenerator: (req: FastifyRequest) => string;
  }
  ```

## 2. Invariants & Behaviors
- [x] Must return HTTP `429 Too Many Requests` when bucket balance is less than 1.
- [x] Must include headers in all responses:
  - `X-RateLimit-Limit`: Maximum bucket capacity.
  - `X-RateLimit-Remaining`: Floor of currently available tokens.
  - `X-RateLimit-Reset`: Milliseconds until bucket refills to capacity.
- [x] Redis failures must fail-open (log error, allow request through with `X-RateLimit-Degraded: true`).
- [x] Concurrency requirement: Redis operations must be evaluated atomically via Lua script.

## 3. Test Fixture Contract
File: `tests/middleware/rate-limiter.spec.ts`
All test assertions in this file must pass without altering the assertions themselves.
```

---

### Step 2: System Prompting the Execution Agent

```text
SYSTEM:
You are an implementation agent executing against a verified specification.

RULES:
1. You may ONLY modify files in `src/middleware/rate-limiter/`.
2. You CANNOT edit `specs/SPEC-004-rate-limiter.md` or any file in `tests/`.
3. After making changes, run `npm test tests/middleware/rate-limiter.spec.ts`.
4. If tests fail, analyze the error output and adjust your code in `src/`.
5. Do not consider the task complete until the test suite passes with 0 errors and `npm run lint` passes.
```

---

### Step 3: Self-Correction Loop Log

```text
[EXECUTION]: Generating src/middleware/rate-limiter/index.ts...
[SHELL]: npm test tests/middleware/rate-limiter.spec.ts

FAIL tests/middleware/rate-limiter.spec.ts
  ● RateLimiter Middleware › Fail-open behavior
    expect(res.headers['x-ratelimit-degraded']).toBe('true')
    Expected: "true"
    Received: undefined
      at Object.<anonymous> (tests/middleware/rate-limiter.spec.ts:42:54)

[FEEDBACK INGESTED]: Spec invariant #3 violated. Caught Redis connection exception did not attach degradation header.
[EXECUTION]: Modifying src/middleware/rate-limiter/index.ts catch block...
[SHELL]: npm test tests/middleware/rate-limiter.spec.ts

PASS tests/middleware/rate-limiter.spec.ts
  ✓ Normal token consumption (12 ms)
  ✓ Rejection on exhaustion (429) (8 ms)
  ✓ Fail-open behavior on Redis error (15 ms)
  ✓ Rate-limit header validation (10 ms)

Test Suites: 1 passed, 1 total
Tests:       4 passed, 4 total
Snapshots:   0 total
Time:        0.842 s
```

---

## 7. Anti-Patterns & Pitfalls

| Anti-Pattern | Description | Remediation |
| :--- | :--- | :--- |
| **Spec Dilution** | Allowing the agent to edit the spec when it gets stuck on an implementation hurdle. | Mount `specs/` and `tests/` as read-only or enforce git pre-commit hooks that fail if spec checksums change during execution runs. |
| **Under-Specified Edge Cases** | Omitting error behavior, time boundaries, or failure modes from the spec. | Mandate an *Invariants and Edge Cases* checklist section in your SDD template. |
| **Monolithic Specification** | Writing a 2,000-line spec covering an entire subsystem at once. | Break specs down into atomic units that can be satisfied in single sessions (max 3–5 file edits per spec). |
| **Non-Deterministic Evaluators** | Testing against live, shared network services where latency spikes trigger test failure. | Require local mocks, ephemeral SQLite/Redis containers, or deterministic contract test fixtures. |

---

## 8. Related Patterns

* **[Issue-Driven Agent Mesh (Beads)](../multi-agent-orchestration/issue-driven-mesh.md):** SDD specs serve as the input definition for atomic task units ("beads").
* **[Hierarchical Supervisor (Gas Town)](../multi-agent-orchestration/hierarchical-supervisor.md):** A Mayor or Supervisor agent uses the spec as the acceptance criteria against which Worker outputs are validated.
* **[Human-in-the-Loop Review Gates](../team-and-enterprise/human-in-the-loop-review.md):** Formalizing the handoff boundary between drafting the specification and running autonomous execution loops.