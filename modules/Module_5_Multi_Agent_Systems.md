
# Module 5 — Multi-Agent Systems

> 💡 **Hands-on Labs:** For practical exercises that complement this module, see the [labs/](../labs) directory or start with [Lab 5: Multi-Agent Workflow](../labs/Lab_5_Multi_Agent_Workflow.md).

> **Module goal.** Compose multiple specialized agents into a **system that solves problems no single agent can** — while keeping coordination, communication, and cost under control.

> **Agentic AI Engineering Focus**
>
> This module is dedicated to engineering multi-agent systems—composing specialized agentic AI components to solve complex problems collaboratively. The focus is on agentic architectures, coordination, and safety, not on general distributed systems or unrelated software topics.

## Table of Contents

- [5.1 Multi-Agent Concepts](#51-multi-agent-concepts)
- [5.2 Agent Collaboration](#52-agent-collaboration)
- [5.3 Multi-Agent Frameworks](#53-multi-agent-frameworks)
- [5.4 Enterprise Workflow Agents](#54-enterprise-workflow-agents)

---

## 5.1 Multi-Agent Concepts

> **Multi-agent systems in agentic AI are engineered organizations—where agents specialize, collaborate, and coordinate to achieve goals beyond the reach of any single agent.**
>
> In this course, multi-agent means designing agentic workflows, roles, and topologies for reliability and value.

```
                  ┌───────────────┐
                  │   Supervisor  │
                  └───┬───────┬───┘
                      │       │
              ┌───────▼─┐   ┌─▼───────┐
              │ Worker A│   │ Worker B│   specialists
              └────┬────┘   └────┬────┘
                   │             │
                   ▼             ▼
                 Tools         Tools
```

---

### 🧠 Why multi-agent

- **Specialization** — narrow prompts and tools beat one generalist.
- **Parallelism** — independent sub-tasks run concurrently.
- **Separation of concerns** — planner, retriever, writer, critic each evolve independently.
- **Safety** — a critic agent reviews a worker's output before action.

> **Caveat.** Multi-agent adds **latency, cost, and failure modes**. Use it when the value of specialization clearly beats the coordination tax.

---

### 🧱 Topologies

| Topology | Shape | Best for |
|---|---|---|
| **Supervisor → Workers** | Hub & spoke | Decomposable goals |
| **Pipeline** | Linear chain | Stage-based processing |
| **Debate / Critic** | Agent + adversary | Quality-sensitive answers |
| **Blackboard** | Shared workspace | Loosely coupled experts |
| **Market / Auction** | Bid-based dispatch | Heterogeneous capability pools |

---

### 🧩 Roles in a multi-agent system

- **Supervisor / Orchestrator** — owns the goal, plans, delegates.
- **Specialist Workers** — narrow domains, narrow tools.
- **Critic / Evaluator** — independent quality and safety checks.
- **Memory / Knowledge agent** — owns RAG and long-term recall.
- **Tool agents** — wrap a specific external system.

---

## 5.2 Agent Collaboration

> Collaboration only works when **communication is structured** and **accountability is explicit**.

---

### 💬 Communication patterns

| Pattern | How it works |
|---|---|
| **Direct messaging** | Agents send typed messages with sender/receiver IDs. |
| **Shared blackboard** | Agents read/write a common store keyed by topic. |
| **Event bus** | Agents publish/subscribe to events; loose coupling. |
| **Function calls** | Supervisor invokes a worker as if it were a tool. |

---

### 📋 Message contract

```json
{
  "id": "msg_01H...",
  "from": "supervisor",
  "to": "finance_worker",
  "type": "task_request",
  "task": "pull_gl_balances",
  "input": { "period": "2026-04" },
  "deadline": "2026-05-02T17:00:00Z",
  "reply_to": "msg_01H..."
}
```

> **Rule.** Every inter-agent message is **typed, traced, and rate-limited**. Free-form chatter between agents is a debugging nightmare.

---

### 🤝 Coordination primitives

- **Task contracts** — explicit goal, inputs, outputs, deadline, success criteria.
- **Handoffs** — one agent delegates *and* transfers context, not just intent.
- **Voting / consensus** — multiple agents propose, an evaluator picks.
- **Negotiation** — agents request resources or capabilities from each other.
- **Escalation** — workers escalate to the supervisor, supervisor escalates to a human.

---

### 🧪 Example — supervisor handoff

```python
plan = supervisor.plan(goal)
for step in plan.steps:
    worker = registry.get(step.role)
    result = worker.run(
        task=step.task,
        context=memory.scoped(step.context_keys),
        budget=step.budget,
    )
    memory.write(step.output_key, result)
    if critic.reject(result):
        plan = supervisor.replan(goal, memory)
```

---

## 5.3 Multi-Agent Frameworks

> Frameworks accelerate the obvious 80%. They become a liability when they hide what your agents are doing.

---

### 🛠️ Landscape

| Framework | Strength | Watch out for |
|---|---|---|
| **LangGraph** | Explicit graph of states & transitions; durable | Verbosity for simple flows |
| **AutoGen** | Conversational multi-agent patterns | Cost of free-form chatter |
| **CrewAI** | Role-based "crews" with simple APIs | Limited control of low-level loop |
| **OpenAI Agents SDK / Anthropic SDK** | Vendor-aligned, lean | Less ecosystem than LC family |
| **Custom** | Total control | You build everything |

---

### 🧭 How to choose

- **Pick a framework when** you need multi-agent state machines, durable runs, or built-in tracing.
- **Stay framework-light when** the system is one agent + a few tools — direct SDK calls are clearer.
- **Always own** the prompts, the tool schemas, and the eval harness — frameworks come and go.

---

### 🧱 Reference shape — LangGraph-style state machine

```
            ┌──────────┐
   start ─▶ │  Plan    │
            └────┬─────┘
                 ▼
            ┌──────────┐
            │  Route   │
            └─┬──────┬─┘
              │      │
        ┌─────▼┐  ┌──▼─────┐
        │Worker│  │Worker B│
        │  A   │  └────┬───┘
        └──┬───┘       │
           └─────┬─────┘
                 ▼
            ┌──────────┐
            │  Critic  │
            └────┬─────┘
                 ▼
              done?
```

---

### 📈 Operational concerns

- **Tracing.** A run = a tree of agent calls; you need a tracer that renders it.
- **Determinism knobs.** Seed, temperature, model pinning per agent.
- **Cost telemetry.** Per-agent, per-tool, per-run.
- **Replays.** Capture inputs/outputs so failed runs can be re-executed offline.

---

## 5.4 Enterprise Workflow Agents

> The unit of value in the enterprise is not "an agent" — it is a **completed business process**.

---

### 🏢 Pattern: agentized business process

```
   Trigger ─▶ Plan ─▶ Specialist agents ─▶ Approvals ─▶ Side effects ─▶ Audit
                                ▲                            │
                                └──── retry / replan ────────┘
```

---

### 🧩 Building blocks

- **Workflow engine** (Temporal, Step Functions, Camunda) — durability, retries, timers, signals.
- **Agent layer** — planners, workers, critics; live as workflow activities.
- **Tool layer** — MCP servers wrapping enterprise systems.
- **Policy layer** — auth, allow-lists, approval gates, audit.
- **Human-in-the-loop tasks** — first-class steps in the workflow.

---

### 🧪 Example — claims processing

| Step | Owner |
|---|---|
| Intake validation | Tool (deterministic) |
| Coverage check | Specialist agent + RAG over policy docs |
| Fraud signal review | Specialist agent + ML scoring tool |
| Decision draft | Writer agent |
| Adjuster approval | Human task |
| Payout | Tool (idempotent) |
| Audit emit | Tool (deterministic) |

> **Design rule.** Anything **regulated, irreversible, or high-cost** is a deterministic step or a human task — *not* an autonomous LLM decision.

---

### 📊 What to measure

- **Cycle time** vs the pre-agent baseline.
- **Auto-completion rate** (no human touch).
- **Override rate** (human edited the agent's output).
- **Quality** (sampled human review against a rubric).
- **Cost per completed process.**
- **Incident rate** (policy violations, bad actions, escalations).

---

### 🧱 Rollout playbook

1. **Shadow mode.** Agent runs alongside humans; outputs compared, never executed.
2. **Co-pilot mode.** Agent drafts; human approves every action.
3. **Gated autonomy.** Agent acts on low-risk cases; humans handle exceptions.
4. **Full autonomy with audit.** Only after the metrics justify it.

---

## 🎯 Module 5 — Takeaways

1. **Multi-agent = an organization of specialists.** Use it only when specialization beats coordination cost.
2. **Pick a topology that matches the work** — supervisor, pipeline, debate, blackboard, market.
3. **Communication must be typed, traced, and rate-limited.** No free-form chatter.
4. **Frameworks help, but you must own** prompts, tool schemas, and evals.
5. **The enterprise unit of value is the completed process**, not the agent.
6. **Hybridize.** Deterministic steps and human tasks for high-stakes; LLM steps where flexibility wins.
7. **Roll out in stages** — shadow → copilot → gated → autonomous.

> ⬅️ **Previous:** [Module 4 — Building AI Agents](Module_4_Building_AI_Agents.md)
> ➡️ **Next:** [Module 6 — RAG and Enterprise Knowledge Agents](Module_6_RAG_and_Enterprise_Knowledge_Agents.md)
