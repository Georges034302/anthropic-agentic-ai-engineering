
# Module 4 — Building AI Agents

> 💡 **Hands-on Labs:** For practical exercises that complement this module, see the [labs/](../labs) directory or start with [Lab 4: Stateful, Guardrailed Agent](../labs/Lab_4_Stateful_Guardrailed_Agent.md).

> **Module goal.** Move from theory to engineering: design a single agent that **plans, remembers, acts autonomously, and stays inside its guardrails** — the building block of every multi-agent system.

> **Agentic AI Engineering Focus**
>
> This module is dedicated to the engineering of agentic AI agents—autonomous, goal-driven systems that plan, act, remember, and operate within guardrails. The focus is on building robust, production-ready agent architectures, not on general automation or unrelated software development topics.

## Table of Contents

- [4.1 Single-Agent Architectures](#41-single-agent-architectures)
- [4.2 Memory Systems](#42-memory-systems)
- [4.3 Autonomous Task Execution](#43-autonomous-task-execution)
- [4.4 Agent Guardrails](#44-agent-guardrails)

---

## 4.1 Single-Agent Architectures

> **Agentic AI agents are engineered as control loops that reason, act, observe, and adapt—enabling autonomy and reliability in real-world workflows.**
>
> In this course, agent architecture means designing the loop, memory, and tool use that make autonomy possible and safe.

```
       ┌────────────┐
       │   Goal     │
       └─────┬──────┘
             ▼
   ┌─────────────────────┐
   │   Reason  ──▶  Act  │
   │     ▲          │    │
   │     └─ Observe ◀┘   │
   └─────────────────────┘
             │
             ▼
       ┌────────────┐
       │  Outcome   │
       └────────────┘
```

---

### 🧠 Core architectural patterns

| Pattern | Shape | Best for |
|---|---|---|
| **ReAct** | Interleaved *Reason → Act → Observe* | General tool-using agents |
| **Plan-and-Execute** | Plan once, then execute steps | Long, predictable tasks |
| **Reflexion** | Act → critique → retry | Quality-sensitive single tasks |
| **Tree-of-Thoughts** | Branch & evaluate alternatives | Search-style problems |
| **Router** | Classify → dispatch to a sub-flow | Mixed-intent assistants |

> **Default.** Start with **ReAct + structured tool calls + a hard step budget**. Move to plan-and-execute when tasks become long or expensive.

---

### 🧱 Anatomy of a single agent

```
  ┌────────────────────────────────────────────┐
  │                System prompt               │   identity, policy, format
  ├────────────────────────────────────────────┤
  │                Tool registry               │   names, schemas, side effects
  ├────────────────────────────────────────────┤
  │                  Memory                    │   short / long / episodic
  ├────────────────────────────────────────────┤
  │             Reasoning engine               │   the foundation model
  ├────────────────────────────────────────────┤
  │              Control loop                  │   limits, retries, stop rules
  ├────────────────────────────────────────────┤
  │                Guardrails                  │   validators, approvals, audit
  └────────────────────────────────────────────┘
```

---

### 🔁 The minimal agent loop

```python
def run_agent(goal, tools, max_steps=12):
    history = [{"role": "user", "content": goal}]
    for step in range(max_steps):
        resp = model.call(history, tools=tools)
        if resp.stop_reason == "end_turn":
            return resp.text
        for call in resp.tool_calls:
            result = dispatch(call.name, call.input)
            history.append(tool_result(call.id, result))
    raise StepBudgetExceeded()
```

> **Engineering rules of thumb.** Always cap steps. Always validate tool inputs. Always log every step.

---

## 4.2 Memory Systems

> Without memory, an agent forgets you between turns. With the *wrong* memory, it remembers things it shouldn't.

```
        ┌──────────────────────────────┐
        │       Working memory         │  current task scratchpad
        ├──────────────────────────────┤
        │      Conversation memory     │  rolling summary + recent turns
        ├──────────────────────────────┤
        │      Episodic memory         │  past sessions, outcomes
        ├──────────────────────────────┤
        │      Semantic memory         │  facts about user / org (RAG)
        ├──────────────────────────────┤
        │      Procedural memory       │  learned skills / playbooks
        └──────────────────────────────┘
```

---

### 🧩 Memory types

| Type | Lifetime | Storage | Example |
|---|---|---|---|
| **Working** | Single turn | Prompt scratchpad | Current plan, intermediate values |
| **Conversation** | Session | Summary + last N turns | "User prefers French replies." |
| **Episodic** | Long-term | Vector DB / log | "Last Tuesday's incident postmortem." |
| **Semantic** | Long-term | Vector / graph DB | "ACME's fiscal year ends March 31." |
| **Procedural** | Long-term | Code / templates | A reusable refund-processing playbook. |

---

### ✍️ Read & write policies

- **Write triggers** — at end of task, on user-confirmed facts, on outcome (win/loss).
- **Read triggers** — on session start, on entity mention, on retrieval.
- **Forgetting** — TTL, decay, explicit user delete (GDPR right-to-erasure).
- **Conflict resolution** — newer facts win unless flagged authoritative.
- **Confidentiality** — tag every memory with sensitivity & access scope.

> **Privacy first.** Memory that you can't audit, redact, or delete is a liability. Build *forget* before you build *remember*.

---

### 🧪 Example — semantic memory entry

```json
{
  "id": "mem_01H...",
  "user_id": "u_42",
  "type": "preference",
  "fact": "Prefers concise, bullet-pointed answers.",
  "source": "explicit_user_statement",
  "confidence": 0.95,
  "created_at": "2026-04-12T10:14:00Z",
  "ttl_days": 365
}
```

---

## 4.3 Autonomous Task Execution

> Autonomy is a spectrum. Pick the right point on it for the **blast radius** of the task.

```
 Suggest  ──▶  Draft  ──▶  Act with approval  ──▶  Act autonomously
 (low risk)                                              (high trust)
```

---

### 🧭 Execution modes

| Mode | Human role | When to use |
|---|---|---|
| **Suggest-only** | Decides & acts | New domain, low trust |
| **Draft & confirm** | Reviews & approves | Customer-facing content, code PRs |
| **Act with approval gate** | Approves high-impact actions | Spend, send, delete operations |
| **Fully autonomous** | Sets policy, audits | Well-bounded, reversible tasks |

---

### 🧱 Building blocks

- **Planner** — decomposes the goal into steps with stop conditions.
- **Executor** — runs each step against tools.
- **Critic** — evaluates intermediate results; can trigger re-planning.
- **Budget manager** — enforces step / token / time / $ limits.
- **State store** — durable record so a crashed run can resume.

---

### ⏯️ Resumability & durability

- Persist agent state after every step (plan, history, tool results).
- Use a **workflow engine** (Temporal, Step Functions) for long-running runs.
- Support **pause/resume** for human-in-the-loop approvals.
- **Idempotent tool calls** so retries are safe.

---

### 🛑 Stop conditions

Every autonomous run needs explicit **stop rules**:

- Goal achieved (model emits a structured "done" signal).
- Step budget reached.
- Cost or time budget reached.
- Repeated tool failure or no-progress detection.
- Critic / evaluator declares "give up — escalate".

> **Failure mode to design out.** *Loop without progress.* Detect repeated tool calls with the same input → break and escalate.

---

### 🧪 Example — execution telemetry

```json
{
  "run_id": "run_01H...",
  "goal": "Reconcile April invoices",
  "steps_taken": 8,
  "tool_calls": 14,
  "tokens": { "in": 42100, "out": 6200 },
  "cost_usd": 0.41,
  "duration_ms": 18342,
  "outcome": "success",
  "approvals": ["finance.controller@acme"]
}
```

---

## 4.4 Agent Guardrails

> Guardrails turn a stochastic model into a **predictable system**. They sit at every boundary the agent crosses.

```
  user ─▶ ┌──────────┐ ─▶ ┌──────────┐ ─▶ ┌──────────┐ ─▶ world
          │  input   │    │ planning │    │  tool    │
          │ guards   │    │ guards   │    │ guards   │
          └──────────┘    └──────────┘    └──────────┘
                                │
                                ▼
                         ┌──────────┐
                         │ output   │
                         │ guards   │
                         └──────────┘
```

---

### 🛡️ Guardrail categories

| Category | Examples |
|---|---|
| **Input** | PII detection, prompt-injection filters, jailbreak classifiers |
| **Planning** | Disallowed actions, scope limits, max sub-tasks |
| **Tool** | Allow-lists, schema validation, rate limits, dry-run, approvals |
| **Output** | Schema/format checks, toxicity & policy classifiers, citation enforcement |
| **Behavioural** | Step / cost / time budgets, no-progress detection |
| **Audit** | Full trace of inputs, decisions, actions, approvals |

---

### 🧱 Implementation patterns

- **Pre-checks** before *every* tool call (auth, schema, business rules).
- **Post-checks** on *every* output (schema, policy classifier, citations).
- **Wrap, don't trust.** Even when the model is "good", validators are deterministic.
- **Fail closed.** On guardrail uncertainty, escalate or refuse — never silently allow.
- **Versioned policies.** Treat guardrails as code; review and test changes.

---

### 🧪 Example — output guardrail

```python
def guard_output(answer, context):
    if not schema.validate(answer):
        return repair(answer, schema)
    if classifier.is_toxic(answer.text):
        return refuse("policy violation: toxicity")
    if not citations_supported(answer.claims, context):
        return repair_with_citations(answer, context)
    return answer
```

---

### 📊 Measuring guardrails

- **Coverage** — what % of runs are checked by which guardrail.
- **True/false positive rate** — measured against a labelled eval set.
- **Refusal rate** — too high = product friction; too low = risk.
- **Time-to-detect** — how fast new failure modes are caught in production.

> **Mantra.** *"Guardrails are not features you ship once. They are a system you operate."*

---

## 🎯 Module 4 — Takeaways

1. **An agent is a control loop, not a single prompt.** Architecture decides reliability.
2. **Start with ReAct + step budget + structured tools.** Graduate to plan-and-execute when needed.
3. **Memory is plural** — working, conversation, episodic, semantic, procedural.
4. **Build *forget* before *remember*.** Privacy and audit are non-negotiable.
5. **Autonomy is a spectrum** — pick the level that matches blast radius.
6. **Make runs durable and resumable** with a real workflow engine.
7. **Guardrails sit at every boundary** — input, planning, tool, output, audit.
8. **Operate guardrails like infrastructure** — measure, iterate, version.

> ⬅️ **Previous:** [Module 3 — MCP (Model Context Protocol)](Module_3_MCP_Model_Context_Protocol.md)
> ➡️ **Next:** [Module 5 — Multi-Agent Systems](Module_5_Multi_Agent_Systems.md)
