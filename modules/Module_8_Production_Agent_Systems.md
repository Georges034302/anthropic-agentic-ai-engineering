# Module 8 — Production Agent Systems

> **Module goal.** Run agents in production like any other **mission-critical service** — with architecture, observability, governance, and a path to scale.

## Table of Contents

- [8.1 Production Architecture](#81-production-architecture)
- [8.2 Observability and Evaluation](#82-observability-and-evaluation)
- [8.3 Responsible AI Operations](#83-responsible-ai-operations)
- [8.4 Scaling Enterprise Agents](#84-scaling-enterprise-agents)

---

## 8.1 Production Architecture

> A demo runs an agent. A product runs **many agents, for many users, against many systems, under SLAs**.

```
 Client ─▶ API GW ─▶ Agent Runtime ─▶ Model Gateway ─▶ Models
                       │                  │
                       ├─▶ Tool layer (MCP, internal APIs)
                       ├─▶ Memory + RAG indices
                       ├─▶ Workflow engine (durable runs)
                       └─▶ Observability + Audit
```

---

### 🧱 Core components

| Component | Responsibility |
|---|---|
| **API gateway** | AuthN/Z, rate limit, schema, tenancy |
| **Agent runtime** | Loop execution, state, retries, budgets |
| **Model gateway** | Provider abstraction, routing, caching, cost |
| **Tool layer** | MCP servers + internal API wrappers |
| **Memory + RAG** | Vector + lexical indices, governance |
| **Workflow engine** | Durable, resumable long-running runs |
| **Observability** | Logs, traces, metrics, evals, audit |
| **Policy engine** | Authorization, content & action policy |

---

### 🧭 Architectural choices

- **Stateless runtime, durable state.** Agent runtime is horizontally scalable; durable state lives in a workflow engine and DBs.
- **Model gateway in front of providers.** Switch models, A/B, fail over, cache, enforce quotas.
- **Tools behind a gateway.** MCP gateway handles auth, audit, and policy uniformly.
- **Tenancy first.** Every request carries a tenant ID; isolation is enforced top-to-bottom.
- **Multi-region** for latency and resilience; pin sensitive tenants per residency rules.

---

### 🚦 Reliability patterns

- **Timeouts** on every model and tool call.
- **Retries with backoff & jitter**, capped.
- **Circuit breakers** on flaky upstreams.
- **Idempotency keys** for side-effecting tools.
- **Bulkheads** — separate pools per tenant or workload class.
- **Graceful degradation** — fall back to a smaller model or a deterministic path.

---

### 💰 Cost architecture

- **Per-request budget** carried through the run.
- **Model routing by step** — small for cheap steps, large for hard ones.
- **Prompt & response caching** keyed by stable inputs.
- **Batching** for bulk transformations.
- **Quota & throttling** per tenant, workload, or feature.

> **Principle.** *"Latency, cost, and quality are a triangle. Pick two per step, and make the trade-off explicit."*

---

## 8.2 Observability and Evaluation

> You cannot operate what you cannot see. Agents need **tracing, metrics, audit, and continuous evaluation** as first-class infrastructure.

---

### 🔭 What to instrument

| Layer | Signals |
|---|---|
| **Run** | Goal, principal, tenant, duration, outcome, cost |
| **Step** | Type (model / tool), inputs, outputs, latency, tokens |
| **Model** | Provider, model, prompt hash, cache hit, finish reason |
| **Tool** | Name, args (redacted), result, side effects, error |
| **Guardrail** | Triggered policy, action (allow/repair/refuse) |
| **Approval** | Approver, decision, latency, comment |

---

### 🧱 Tracing

- Each **run = a trace**; each step = a span.
- Use OpenTelemetry conventions; add agent-specific attributes (`agent.role`, `model.name`, `tool.name`).
- Render runs as **trees** for inspection (planner → workers → tools).
- Capture **inputs/outputs** at sampled fidelity for replay.

---

### 📊 Core metrics

- **Quality** — eval pass rate, citation rate, refusal rate.
- **Reliability** — success rate, retry rate, timeout rate.
- **Latency** — p50/p95/p99 per step type.
- **Cost** — $ per run, per tenant, per feature.
- **Safety** — guardrail trigger rate, policy violations caught vs escaped.
- **User feedback** — 👍/👎, edit-after-acceptance rate.

---

### 🧪 Continuous evaluation

```
   Golden sets ─┐
   Replay logs ─┼─▶  Eval harness  ─▶  Scorecards per release / model / prompt
   Synthetic   ─┘                              │
                                               ▼
                                       block / promote / rollback
```

- **Golden sets** of high-value tasks per domain.
- **Replay** real (redacted) production runs against new prompts/models.
- **LLM-as-judge** for subjective quality, *paired with* deterministic checks.
- **Pre-deploy gates** on evals; **online evals** on a sampled stream.

---

### 🧩 Audit & forensics

- **Immutable logs** for regulated workloads.
- **Per-action provenance** — which prompt version, which model, which tool, which approver.
- **Replayable runs** — given a run ID, reconstruct exactly what the agent saw and did.

> **Mantra.** *"If you can't replay it, you can't defend it."*

---

## 8.3 Responsible AI Operations

> Responsible AI is not a checklist you finish. It is a **practice you run** — across people, process, and platform.

---

### 🧭 Principles in practice

| Principle | Operational realization |
|---|---|
| **Helpfulness** | Eval rubrics, user feedback loops |
| **Honesty** | Citation enforcement, refusal when unsupported |
| **Harmlessness** | Policy classifiers, allow-lists, human gates |
| **Privacy** | Data minimization, redaction, retention, right-to-erasure |
| **Fairness** | Disaggregated eval across cohorts; bias monitoring |
| **Transparency** | "Why this answer?", model cards, change logs |
| **Accountability** | Owner per agent, per tool, per policy |

---

### 🏛️ Governance structures

- **AI risk council** — reviews new agents, tools, and high-impact changes.
- **Standards & playbooks** — system-prompt reviews, eval requirements, rollout gates.
- **Model & prompt registry** — versioned, owned, deprecation policy.
- **Incident process** specific to AI failures (hallucinations, bad actions, policy breaches).

---

### 🛡️ Safety operations

- **Red-teaming** as a continuous activity, not a launch milestone.
- **Abuse monitoring** — anomaly detection on prompts, tool use, output content.
- **Kill switches** per agent, per tool, per tenant.
- **Customer & regulator-facing transparency** for material changes.

---

### ⚖️ Compliance touch points

- **Data residency** — region pinning per tenant.
- **Sectoral regimes** — HIPAA, PCI, SOX, GDPR, AI Act — mapped to controls.
- **Vendor risk** — model & infra providers evaluated and monitored.
- **DPIAs** for new high-risk use cases.

> **Principle.** *"Govern the system, not the model. Models change quarterly; the controls must outlive them."*

---

## 8.4 Scaling Enterprise Agents

> Scaling is **not just more traffic**. It is more agents, more tools, more tenants, more teams — without losing control.

---

### 📈 Dimensions of scale

| Dimension | What changes |
|---|---|
| **Traffic** | Throughput, concurrency, burst handling |
| **Tenants** | Isolation, quotas, residency, customization |
| **Agents** | Discovery, dependency, lifecycle |
| **Tools** | Catalog, governance, deprecation |
| **Teams** | Ownership, paved paths, golden templates |
| **Knowledge** | Index size, freshness, ACL complexity |
| **Cost** | Per-feature accountability, optimization loops |

---

### 🛣️ The platform play

```
   ┌───────────────────────────────────────────┐
   │              AI Platform                  │
   │  Model GW · MCP GW · Eval · Observability │
   │  Memory/RAG · Workflow · Policy · Cost    │
   └───────────────────────────────────────────┘
                    ▲
                    │ paved-path SDKs / templates
                    │
   ┌──────────┐ ┌──────────┐ ┌──────────┐
   │ Sales AI │ │  Ops AI  │ │  Eng AI  │   product teams build on top
   └──────────┘ └──────────┘ └──────────┘
```

- **One platform**, many product agents.
- Product teams **own behaviour**; the platform owns **substrate, safety, and economics**.
- Paved paths (templates, SDKs, evals, dashboards) make the secure default the easy default.

---

### 💸 FinOps for agents

- **Unit economics** — cost per completed task, per tenant, per feature.
- **Cost dashboards** broken down by model, tool, prompt version.
- **Caching, batching, distillation, routing** as continuous optimization loops.
- **Budgets and alerts** at the team & tenant level.

---

### 🗺️ Maturity model

| Level | Characteristics |
|---|---|
| **L1 Pilot** | One agent, one team, manual ops |
| **L2 Productized** | SLAs, observability, basic governance |
| **L3 Multi-agent** | Multiple agents, MCP gateway, eval harness |
| **L4 Platform** | Internal AI platform, paved paths, FinOps |
| **L5 Strategic** | AI woven into core processes; governance & safety operationalized |

---

### 🧪 Example — quarterly platform health review

- Adoption: # active agents, # tools, # tenants.
- Quality: pass rates per agent vs target; trend.
- Reliability: error budgets per agent.
- Cost: $/task, top spenders, savings from optimizations.
- Safety: guardrail triggers, incidents, time-to-mitigate.
- Roadmap: deprecations, new platform features, research bets.

---

## 🎯 Module 8 — Takeaways

1. **Production agents are services.** Architect them with the same rigor as any tier-1 system.
2. **Stateless runtime + durable state + workflow engine** is the reliable shape.
3. **Model and MCP gateways** centralize control of cost, safety, and policy.
4. **Observability is a feature, not a luxury.** Trace, measure, evaluate continuously.
5. **Responsible AI is operational** — councils, registries, red teams, kill switches.
6. **Govern the system, not the model.** Controls must outlive providers and weights.
7. **Scale via a platform** — paved paths, FinOps, multi-tenant safety by default.
8. **Pick two of latency, cost, quality per step** — and make the trade-off explicit.

> ⬅️ **Previous:** [Module 7 — AI-Augmented Software Engineering](Module_7_AI_Augmented_Software_Engineering.md)
> 🏁 **End of course.** Return to the [Course Index](../docs/index.md).
