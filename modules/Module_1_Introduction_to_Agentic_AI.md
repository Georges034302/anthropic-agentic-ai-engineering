
# Module 1 — Introduction to Agentic AI

> 💡 **Hands-on Labs:** For practical exercises that complement this module, see the [labs/](../labs) directory or start with [Lab 1: Minimal Agentic Loop](../labs/Lab_1_Minimal_Agentic_Loop.md).

> **Module goal.** Build a precise mental model of what agentic AI *is*, what makes it different from prior generations of AI, and where it creates measurable value in the enterprise.

> **Agentic AI Engineering Focus**
> 
> This course is dedicated to the engineering of agentic AI systems—autonomous, goal-driven software agents that plan, act, and adapt in real-world workflows. It is not about general software development or generative image models (e.g., Stable Diffusion), but about building, deploying, and operating agentic AI architectures for enterprise and production use.

## Table of Contents

- [1.1 What is Agentic AI](#11-what-is-agentic-ai)
- [1.2 Foundations of Modern AI Models](#12-foundations-of-modern-ai-models)
- [1.3 AI API Fundamentals](#13-ai-api-fundamentals)
- [1.4 Enterprise AI Use Cases](#14-enterprise-ai-use-cases)

---

## 1.1 What is Agentic AI

> **Agentic AI moves software from answering questions to completing work.**
>
> In this course, "agentic" means engineering systems that can autonomously pursue goals, use tools, and adapt to changing information—far beyond simple API calls or static prompts. Our focus is on the practical design and deployment of such agentic workflows.

A foundation model on its own is a powerful function: text in, text out. An **agent** wraps that function in a loop, gives it tools, gives it memory, and points it at a goal. The result is a system that can plan, act, observe, and adapt — autonomously.

```
        ┌──────────────┐
        │     Goal     │
        └──────┬───────┘
               ▼
   ┌──────────────────────┐
   │  Plan  →  Act  →  Observe  │  ← loop until "done" or stopped
   └──────────────────────┘
               │
               ▼
        ┌──────────────┐
        │   Outcome    │
        └──────────────┘
```

---

### 🧠 Autonomous reasoning systems

**Definition.** A system that, given a goal, decomposes it, chooses actions, executes them, evaluates the result, and decides what to do next — without a human approving every step.

**Core ideas**

| Concept | What it means |
|---|---|
| Agent loop | The *plan → act → observe* cycle (often implemented as ReAct). |
| Planning horizon | Short-horizon (one tool call) vs long-horizon (multi-day workflows). |
| Policy | The strategy mapping *state → next action*; mostly encoded in prompts and tool schemas. |
| Self-correction | Re-plan after a failed call, an invalid output, or a critic's feedback. |

**Where it fits.** Anywhere the work requires **composition** (search + compute + write), **branching** (decisions on intermediate results), or **recovery** (retrying when the first approach fails).

**Example — Research agent**

> *Goal: "Summarize Q3 competitor product launches."*
>
> 1. Query news APIs and the internal CRM.
> 2. Deduplicate and cluster by competitor.
> 3. Extract launch dates and pricing.
> 4. Draft a one-page brief with citations.
> 5. If a query returns nothing → broaden keywords and retry.

---

### 🤖 AI agents vs chatbots

**Definition.** A **chatbot** is reactive and conversational. An **AI agent** is proactive and outcome-oriented — it may converse, but it is judged by *task completion*, not response quality alone.

| Dimension | Chatbot | AI Agent |
|---|---|---|
| **Trigger** | User message | Message *or* event / schedule |
| **State** | Stateless / short memory | Persistent memory + working state |
| **Tools** | Few or none | First-class, often many |
| **Output** | Text reply | Side effects (files, APIs, tickets) **+** reply |
| **Success metric** | Helpfulness, CSAT | Task completion rate, time-to-resolution |
| **Failure mode** | Wrong answer | Wrong answer **or** wrong action |

**Rule of thumb**

> Use a **chatbot** when the goal is a good answer.
> Use an **agent** when the goal is a finished task.

**Example.** A support **chatbot** explains *how* to reset a password. A support **agent** verifies identity, triggers the reset workflow, opens a ticket, and confirms back to the user.

---

### 🏢 Enterprise AI agents

**Definition.** Agents deployed inside the organization's systems-of-record (CRM, ERP, ITSM, code, data) under enterprise-grade constraints: identity, audit, residency, policy.

**Non-negotiables**

- **Identity-bound execution** — the agent acts *on behalf of* an authenticated principal; permissions are inherited, never elevated.
- **Policy enforcement points** — pre-action checks, post-action validation, human approval for high-impact moves.
- **Auditability** — every plan, prompt, tool call, and result is logged and queryable.
- **Data boundaries** — PII/PHI handling, tenant isolation, retention controls.

**Example — Finance agent**

> Reads open invoices in the ERP → flags duplicates → drafts a remediation memo.
> Auto-posts journal corrections **below $5k**; routes anything **above** to a controller for approval.

---

### 🔁 Agentic workflows

**Definition.** Multi-step processes where one or more agents coordinate tools, data, and sometimes other agents to deliver a defined business outcome.

**Common orchestration patterns**

- **Sequential pipeline** — fixed order of LLM/tool steps.
- **Router** — an LLM picks a branch; each branch is a sub-workflow.
- **Parallel fan-out / fan-in** — split work, run concurrently, merge.
- **Supervisor + workers** — a planner agent delegates to specialist agents.
- **Plan-and-execute** — produce a full plan first, then execute step by step.

**Design tension**

> **Determinism vs autonomy.** Hard-code the steps that must be auditable.
> Let the model drive the steps where flexibility creates value.

**Example — Monthly board pack**

> A *supervisor* agent decomposes the goal and delegates:
> - **Finance worker** pulls GL data via SQL tool.
> - **Narrative worker** drafts commentary using RAG over past packs.
> - **Editor worker** enforces tone and consistency.
>
> The supervisor assembles the deck and routes it for sign-off.

---

## 1.2 Foundations of Modern AI Models

> You cannot design good agents without a working model of how the *underlying model* behaves — what it is good at, where it breaks, and how it is kept safe.

---

### 🧬 Model families and capabilities

**Definition.** A *model family* is a coordinated set of models from a vendor — typically a small/fast variant, a balanced variant, and a flagship — sharing training methodology and tool-use conventions.

**Capability axes that matter for agents**

- Reasoning depth and instruction following
- Code, math, and structured output reliability
- Multimodality (vision, audio)
- Tool use and function calling
- Long-context comprehension
- Latency and cost per million tokens

**Selection heuristic**

| Step in the agent | Typical model choice | Why |
|---|---|---|
| Routing / classification | Small, fast | High volume, low complexity |
| Planning / reasoning | Flagship | Quality of decisions dominates cost |
| Bulk transformation | Small or mid | Cost dominates quality |
| Final synthesis | Flagship | User-visible, brand-critical |

> ⚠️ **Benchmarks are not evals.** MMLU and SWE-bench tell you *general* capability. Only **task-specific evals on your data** tell you fitness for your workload.

**Example.** A document pipeline uses a small model to classify the document, a vision model to OCR scans, and a flagship model to extract structured fields validated against a JSON Schema.

---

### 📚 Long-context processing

**Definition.** The model's ability to attend to very large inputs — tens or hundreds of thousands of tokens — within a single request.

**What actually matters**

- **Window size ≠ effective context.** Recall and reasoning quality degrade in the middle of long inputs ("lost in the middle").
- **Placement is leverage.** Put critical instructions and the most relevant evidence near the **start or end**.
- **Structure earns attention.** Use headings, XML tags, and Markdown delimiters to make the document navigable.
- **Cost scales linearly** with input tokens — long context shifts spend from retrieval engineering to inference.

**When to choose long-context over RAG**

> When the source set is small enough to fit, **and** the cost of stuffing it is lower than the cost of building and maintaining retrieval.

**Example.** A legal review agent loads a 200-page contract **plus** the firm's 50-page playbook into a single request and returns a structured table of clause-level deviations.

---

### 📜 Constitutional and responsible AI

**Definition.** Techniques — most notably Anthropic's *Constitutional AI* — that shape model behavior using an explicit, written set of principles ("a constitution"), applied during training and at inference.

**How it works**

1. Define principles in plain language (honesty, harmlessness, helpfulness, domain rules).
2. Train the model to **critique** its own drafts against those principles.
3. Train it to **revise** the draft until the critique passes.
4. At inference, supply *additional* organization-specific principles via system prompts or evaluator agents.

**Why it matters for agents**

> Agents act in the world. A constitution gives the agent — and your evaluators — a stable, inspectable reference for what "acceptable behavior" means.

**Example.** Every customer-facing response is passed through an evaluator agent that checks the draft against the company's tone, compliance, and brand-voice constitution and rewrites on violation.

---

### 🛡️ Safety-oriented architectures

**Definition.** System-level designs combining model choice, prompts, tools, validators, and human oversight to make agent behavior **predictable, bounded, and recoverable**.

**Defense in depth**

```
 user / event
     │
     ▼
 ┌─────────────────┐   input validation
 │  Policy prompt  │   tool allow-list
 └────────┬────────┘
          ▼
 ┌─────────────────┐   sandboxed execution
 │   Agent loop    │   rate limits
 └────────┬────────┘
          ▼
 ┌─────────────────┐   schema / regex / unit-test validators
 │ Output validator│
 └────────┬────────┘
          ▼
 ┌─────────────────┐   human approval for high-impact actions
 │  Action gate    │
 └─────────────────┘
```

**Principles**

- **Least privilege.** Give the agent only the tools and permissions it needs.
- **Reversibility first.** Prefer dry-run, simulate, or stage; require confirmation for irreversible operations.
- **Deterministic guardrails around stochastic models.** Schemas, regexes, sandboxes, rate limits.
- **Assume prompt injection.** Treat content from tools and untrusted users as *data*, never as *instructions*.

**Example.** A DevOps agent may read logs and propose Kubernetes manifest changes — but applying them requires schema validation, `kubectl --dry-run`, and an explicit human approval comment on the generated PR.

---

## 1.3 AI API Fundamentals

> Agents are built on top of model APIs. Master the request/response contract, prompt roles, streaming, and structured output before anything else.

---

### 💬 Messages and interaction APIs

**Definition.** A modern LLM API exposes a **messages** endpoint that takes an ordered list of role-tagged messages — `system`, `user`, `assistant`, `tool` — and returns the next assistant message.

**What to internalize**

- **Roles encode trust.** `system` is highest, `tool`/`user` are lowest.
- **State is client-managed.** The API is stateless; *you* resend the relevant history each turn.
- **Tokens drive cost and latency.** Truncation and summarization strategies are part of the design, not an afterthought.
- **Production concerns.** Idempotency keys, retries with backoff, rate-limit handling.

**Example — minimal call**

```python
response = client.messages.create(
    model="claude-...-latest",
    system="You are a careful financial analyst.",
    messages=[
        {"role": "user", "content": "Summarize the Q3 revenue drivers."},
    ],
    max_tokens=1024,
)
print(response.content[0].text)
```

---

### 🧭 System prompts

**Definition.** A high-priority instruction block that defines the agent's **identity, scope, constraints, output format, and policies** for the entire session.

**A reusable layout**

```
1. Identity         — who the agent is
2. Capabilities     — what it can do
3. Constraints      — what it must not do
4. Tools            — when and how to use each tool
5. Output contract  — format, sections, citation rules
6. Examples         — 1–3 high-quality demonstrations
```

**Discipline**

- **Keep the system prompt stable** across turns. Per-turn changes belong in user/tool messages.
- **Treat prompts as code.** Review, version, and test them.
- **Resist prompt injection.** State explicitly that text from tools and users is data, not instruction.

**Example**

```text
You are "FinBot", an internal financial-analysis assistant for ACME Corp.

- Use only data provided in <context> blocks; never invent figures.
- If asked to make a trade, refuse and direct the user to Treasury.
- Respond in Markdown with three sections: "Summary", "Numbers", "Caveats".
```

---

### ⚡ Streaming responses

**Definition.** A delivery mode where the model emits output **incrementally** over an open connection, instead of returning one final payload.

**Why it matters**

- **UX.** Lower perceived latency — users see progress immediately.
- **Engineering cost.** The client must buffer, render, cancel, and recover from mid-stream errors.
- **Tool use over streams.** Tool arguments arrive as a streamed JSON delta; accumulate before parsing.

**Event types you will handle**

- `content_block_delta` — incremental text
- `tool_use` start / input deltas / stop
- `message_stop` — final usage, stop reason

**Example**

```python
with client.messages.stream(model=..., messages=...) as stream:
    for text in stream.text_stream:
        print(text, end="", flush=True)
    final = stream.get_final_message()
```

---

### 📦 Structured outputs

**Definition.** Techniques and API features that constrain model output to a **machine-parseable schema** — usually JSON — so downstream code can consume it deterministically.

**Three approaches, ranked by reliability**

1. **Tool / function calling** — the model returns a tool call whose arguments conform to your JSON Schema.
2. **Schema-constrained decoding / `response_format`** — the API enforces the schema on the final message.
3. **Prompted JSON + validator** — instruct, then validate (`pydantic`, `zod`, JSON Schema) and self-repair on failure.

**Schema design rules**

- Prefer **flat, well-named** fields.
- Mark optional fields explicitly.
- Use `enum` wherever values are bounded — it dramatically reduces drift.
- Always validate; on failure, send the validator error back to the model for repair.

**Example — invoice extraction**

```json
{
  "invoice_number": "INV-2026-0431",
  "vendor": "ACME Logistics",
  "issue_date": "2026-04-12",
  "due_date":   "2026-05-12",
  "currency":   "USD",
  "total":      12450.00,
  "line_items": [
    { "description": "Freight, Sydney → Singapore", "qty": 1, "amount": 8200.00 },
    { "description": "Customs handling",            "qty": 1, "amount": 4250.00 }
  ]
}
```

---

## 1.4 Enterprise AI Use Cases

> Four agent patterns that consistently produce measurable ROI in production.

---

### 🔬 Research agents

**Definition.** Agents that autonomously **gather, evaluate, synthesize, and cite** information from internal and external sources to answer open-ended questions or produce briefs.

**Architecture**

```
Search ──▶ Read ──▶ Summarize ──▶ Cite
   ▲                                  │
   └──────────  critic: "complete?" ──┘
```

**Engineering levers**

- **Source credibility scoring** and **mandatory citations**.
- **Cost control** — cap iterations, depth, tokens per source.
- **Freshness** — combine real-time search with internal RAG.

**Example.** *"Compare the top 3 vector databases for our latency/cost profile."*
The agent queries vendor docs, benchmark blogs, and the internal cost calculator, then produces a comparison table with footnoted sources and explicitly noted gaps.

---

### 👨‍💻 Coding assistants

**Definition.** Agents embedded in the developer workflow — IDE, CLI, CI — that read and write code, run tests, and iterate to satisfy a specification.

**What makes them work**

- **Repository awareness** — code search, symbol indexing, dependency graphs.
- **Edit primitives** — file diffs, patches, multi-file refactors.
- **Verify by execution** — tests, type-checkers, linters; iterate on failure.
- **PR-centric workflow** — the agent proposes a branch + PR; humans review and merge.
- **Sandboxing** — contained execution, network egress controls, secret scrubbing.

**Example.** A failing test is reported. The agent locates the offending function, proposes a patch, runs the suite, iterates until green, runs the linter, and opens a PR with a written rationale linking the failing → passing test runs.

---

### 🧑‍💼 Enterprise copilots

**Definition.** Domain-specialized AI assistants embedded in line-of-business apps — sales, support, HR, finance — combining RAG, tool use, and policy enforcement to assist or act on behalf of employees.

**Maturity ladder**

```
 Answer  ──▶  Answer + Draft  ──▶  Answer + Draft + Execute (with approval)
```

**What good copilots share**

- **Grounded answers** — every claim cites a retrieved enterprise source.
- **Identity-aware** — role, region, and permissions shape responses *and* actions.
- **Native UX** — embedded in the system of record, not a separate tab.
- **Undo / redo** for any action the agent performs.

**Example.** A sales copilot in the CRM summarizes account history, drafts a tailored outreach email citing recent product news, and — on approval — schedules the send and creates a follow-up task.

---

### 🔀 Workflow orchestration

**Definition.** The coordination of multiple steps, tools, and (optionally) agents to deliver an end-to-end business process with **reliability, observability, and recovery**.

**What you actually need in production**

- **Durable execution** — workflow engines (Temporal, Step Functions, Airflow, Prefect) provide retries, timeouts, checkpoints, visibility. LLM calls become individual steps.
- **Hybrid orchestration** — deterministic control flow on compliance-critical paths; LLM-driven decisions where flexibility wins.
- **Human-in-the-loop tasks** — pause, request approval, resume on signal.
- **Idempotency** — exactly-once semantics for side-effecting tool calls.
- **Observability** — trace every LLM call, tool call, and decision; correlate by workflow run ID.

**Example — Employee onboarding**

```
HRIS event
   │
   ▼
 Agent gathers role profile
   │
   ▼
 Tools create accounts (IdP, Slack, GitHub)
   │
   ▼
 Agent drafts personalized welcome doc (RAG)
   │
   ▼
 Manager approves
   │
   ▼
 Send doc · schedule orientation · emit audit record
```

---

## 🎯 Module 1 — Takeaways

1. **An agent is a loop, not a model.** Plan → act → observe → repeat.
2. **Choose chatbot vs agent by goal.** Good answer → chatbot. Finished task → agent.
3. **Enterprise agents inherit identity, audit, and policy** from the systems they touch.
4. **Pick the right model for the right step** — small for routing, flagship for reasoning.
5. **Long context is powerful but not magic** — placement and structure decide quality.
6. **Safety is architectural** — defense in depth, least privilege, reversibility.
7. **Master the API primitives** — messages, system prompts, streaming, structured output.
8. **The four high-ROI patterns** are research, coding, copilots, and workflow orchestration.

> ➡️ **Next:** [Module 2 — Prompt Engineering and Context Design](Module_2_Prompt_Engineering_and_Context_Design.md)
