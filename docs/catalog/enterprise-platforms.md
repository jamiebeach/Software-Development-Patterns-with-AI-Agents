# Catalog: Enterprise Autonomous Platforms

Enterprise platforms ($L_3 - L_4$) operate at the organizational and monorepo scale (1M to 50M+ lines of code). Unlike local developer tools, they are engineered around three non-negotiable enterprise constraints:
1. **Deep Structural Ingestion:** Replacing naive RAG with full Abstract Syntax Tree (AST), call-graph, and historical PR knowledge bases.
2. **Zero-Trust Security & Compliance:** Self-hosted VPC deployments, air-gapped LLM routing, SOC2/HIPAA auditability, and strict Role-Based Access Control (RBAC) inherited from the invoking engineer.
3. **Organizational Workflow Embedding:** Triggering directly from Jira, Linear, PagerDuty, Datadog, and Slack rather than requiring an engineer to babysit an IDE.

---

## 1. Blitzy
* **Link:** [blitzy.com](https://blitzy.com/)
* **Taxonomy Profile:** Autonomy: $L_4$ (System 2 Batch Execution) | Topology: $M:N$ Enterprise Factory | Patterns: `Knowledge-Graph Grounding`, `Spec-Driven Development`

### Architectural Mechanics
Blitzy is engineered for massive enterprise modernization, feature build-outs, and cross-cutting refactors where context windows fail:
* **Multi-Dimensional Codebase Ingestion:** Before writing a line of code, Blitzy ingests the entire monorepo—parsing ASTs, cross-service dependency graphs, documentation, and historical git blame into a dynamic architectural index.
* **Long-Horizon System 2 Compute:** Rather than returning a response in 30 seconds, Blitzy orchestrates thousands of specialized micro-agents over **8 to 12+ hours of background compute**. Agents generate a comprehensive technical specification file first, validate it with human architects, and then execute, compile, and recursively test code inside isolated staging environments.
* **Audit-Ready Merge Gate:** Delivers cohesive Pull Requests accompanied by architectural decision logs, test coverage diffs, and blast-radius reports so human reviewers can verify massive changes without reading every line cold.

---

## 2. Factory (Factory.ai)
* **Link:** [factory.ai](https://factory.ai/)
* **Taxonomy Profile:** Autonomy: $L_3 - L_4$ | Topology: Specialized Organizational Fleet | Patterns: `Cross-System Event-Driven Agents`

### Architectural Mechanics
Factory embeds specialized autonomous agents—called **Droids**—directly into the enterprise's operational telemetry and ticket lifecycle:
* **Code & Migration Droids:** Connect to Jira/Linear and GitHub to turn technical specifications or language upgrade tickets into tested PRs across multiple microservice repositories.
* **Reliability & Incident Droids:** Integrate natively with Datadog, Sentry, PagerDuty, and cloud logs. When a production alert fires, the Reliability Droid autonomously correlates the stack trace against recent deployments, drafts a Root Cause Analysis (RCA) in Slack, and opens a candidate hotfix PR before the on-call engineer finishes triaging.

---

## 3. Tessl
* **Link:** [tessl.io](https://tessl.io/)
* **Taxonomy Profile:** Autonomy: $L_4$ | Topology: Spec-Centric Compilation | Patterns: `Spec-Driven Development (Spec-as-Source)`

### Architectural Mechanics
Tessl represents the most radical point on the SDD maturity curve: **Spec-as-Source**. 
* **Inverted Artifact Hierarchy:** In Tessl, human engineers only author, review, and version-control specifications, behavioral contracts, and test invariants. The underlying application code (TypeScript, Python, Java) is treated like compiled machine bytecode—generated, maintained, and regenerated autonomously by the platform.
* **Self-Healing Dependency & Security Maintenance:** When a CVE is disclosed in a framework or an upstream API changes, Tessl recompiles the implementation layer from the immutable specification and validates it against the contract test harness without manual code intervention.

### Failure Modes & Trade-offs
* **Abstraction Leaky Seams:** Requires complete organizational buy-in. If engineers manually hotfix the generated implementation code without updating the upstream specification, the next spec compilation pass will overwrite their manual changes.