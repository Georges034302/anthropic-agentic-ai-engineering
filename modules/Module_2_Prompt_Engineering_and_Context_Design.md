# Module 2 — Prompt Engineering and Context Design

> **Module goal.** Treat prompts and context as **engineered artifacts**: design them with structure, test them like code, and shape them to make models reliable, tool-using collaborators rather than unpredictable text generators.

## Table of Contents

- [2.1 Prompt Engineering Principles](#21-prompt-engineering-principles)
- [2.2 Structured Agent Prompts](#22-structured-agent-prompts)
- [2.3 Tool Use and Function Calling](#23-tool-use-and-function-calling)
- [2.4 Context Engineering](#24-context-engineering)

---

## 2.1 Prompt Engineering Principles

> A prompt is not a wish. It is a **specification** the model executes against — and like any specification, ambiguity costs you.

The shift from "prompt hacking" to **prompt engineering** is the shift from clever phrasing to disciplined design: clear intent, explicit structure, deliberate reasoning, and curated context.

```
        Goal
          │
          ▼
   ┌─────────────┐
   │  Intent     │  what we want
   ├─────────────┤
   │  Structure  │  how the answer must look
   ├─────────────┤
   │  Reasoning  │  how the model should think
   ├─────────────┤
   │  Context    │  what it can rely on
   └─────────────┘
          │
          ▼
       Output
```

---

### ✍️ Clear instruction design

**Definition.** Writing prompts that are **specific, unambiguous, and verifiable** — so the model and the engineer agree on what "done" means.

**Discipline**

| Bad prompt | Better prompt |
|---|---|
| "Summarize this." | "Summarize the document in **5 bullet points**, each ≤ 20 words, focused on **financial risk**." |
| "Make this code better." | "Refactor `parse_orders()` for readability **without** changing behavior; preserve the public signature; add type hints." |
| "Translate this." | "Translate the text to **French (France)**, formal register, preserving Markdown structure." |

**Engineering rules**

- **Name the audience and the format.** "For a CFO, in Markdown, with a `Summary` and `Numbers` section."
- **State the constraints positively.** "Use only the data in `<context>`" beats "don't make things up".
- **Quantify wherever possible.** Counts, lengths, ranges, schemas.
- **Show, don't only tell.** One concrete example outperforms three paragraphs of instructions.

> **Heuristic.** If two senior engineers can disagree on whether the output meets the prompt, the prompt is underspecified.

**Example**

```text
You are a release-notes writer.

Task: From the <commits> block, produce release notes for engineers.
Output: Markdown with three sections — "Features", "Fixes", "Breaking Changes".
Rules:
- Each item ≤ 15 words, present tense, imperative voice.
- Group by component (`api`, `web`, `infra`); omit empty groups.
- Cite commit SHAs in parentheses.
```

---

### 🧱 Structured prompting

**Definition.** Organizing prompts with **semantic delimiters** (XML tags, Markdown sections, JSON blocks) so the model can reliably distinguish *instructions*, *context*, *examples*, and *user input*.

**Why it works.** Foundation models are trained on web-scale structured text; tagged regions activate stronger attention to "this is the rule" vs "this is the data".

**A reusable prompt skeleton**

```xml
<role>You are a senior compliance analyst.</role>

<task>
  Identify clauses in the contract that deviate from our playbook.
</task>

<rules>
  - Cite the clause number for every finding.
  - Classify severity as one of: low | medium | high.
  - Output JSON conforming to <schema/>.
</rules>

<schema>
  { "findings": [ { "clause": "string", "severity": "low|medium|high", "explanation": "string" } ] }
</schema>

<examples>
  <example> ... </example>
</examples>

<context>
  <playbook> ... </playbook>
  <contract> ... </contract>
</context>
```

**Conventions worth adopting**

- Use **lowercase XML tags** with semantic names (`<task>`, `<context>`, `<rules>`).
- Put **untrusted input inside tags** and tell the model to treat tag contents as *data*.
- Keep **instructions at the top and the bottom** for reliable recall in long prompts.

---

### 🧩 Deliberate reasoning

**Definition.** Techniques that prompt the model to **think step-by-step** before answering — improving accuracy on multi-step problems at the cost of tokens and latency.

**The toolkit**

- **Chain-of-thought (CoT).** "Think through this step by step before answering."
- **Plan-then-execute.** First produce a numbered plan; then execute each step.
- **Decomposition.** Split the task into sub-questions; answer each, then combine.
- **Self-consistency.** Generate N reasoning paths; take the majority answer.
- **Reflection.** After answering, critique the answer against the rules; revise if needed.

> ⚠️ **Don't show internal reasoning to the end user by default.** Keep it as scratchpad; expose only the final, polished output.

**When to use what**

| Task | Technique |
|---|---|
| Arithmetic / multi-hop logic | CoT, decomposition |
| Long-running agent task | Plan-then-execute |
| Subjective judgement | Self-consistency |
| High-stakes draft | Reflection / evaluator pass |

**Example**

```text
Think step by step inside <scratchpad>...</scratchpad>.
Then output ONLY the final JSON answer outside the scratchpad.
```

---

### 🗂️ Context structuring

**Definition.** The deliberate **selection, ordering, and labelling** of supporting information placed in the model's context window.

**Levers**

- **Relevance.** Retrieve or curate; do not stuff.
- **Order.** Most-important content **first or last** — middles get lost.
- **Labels.** Tag each block (`<policy>`, `<email>`, `<ticket>`) so the model can reference it.
- **Compression.** Summarize history; keep verbatim only what is decision-critical.

**Example layout**

```
<system>          stable identity, scope, format
<rules>           non-negotiable constraints
<context>         retrieved evidence, labelled
<conversation>    rolling summary + last N turns
<user>            the current request
```

---

## 2.2 Structured Agent Prompts

> Agents need more than one prompt. They need a **prompt portfolio** — one for each cognitive role the agent plays in the loop.

```
   ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
   │ Planner  │ ──▶ │  Router  │ ──▶ │  Worker  │ ──▶ │Reflector │
   └──────────┘     └──────────┘     └──────────┘     └──────────┘
                                                          │
                                                          ▼
                                                    ┌──────────┐
                                                    │Evaluator │
                                                    └──────────┘
```

---

### 🗺️ Planner prompts

**Definition.** Prompts that ask the model to **decompose a goal** into an ordered, executable plan before any tool is called.

**Anatomy**

- **Goal restatement** — force the model to confirm what it heard.
- **Constraints** — budgets, time limits, forbidden actions.
- **Tool inventory** — names, purposes, costs.
- **Output schema** — a structured plan, not prose.

**Example output schema**

```json
{
  "goal": "string",
  "assumptions": ["string"],
  "steps": [
    { "id": 1, "action": "tool_name", "input": {...}, "expected": "string" }
  ],
  "stop_conditions": ["string"]
}
```

> **Tip.** Have the planner output **stop conditions** explicitly. Agents that don't know when to stop, don't.

---

### 🔀 Tool-routing prompts

**Definition.** Prompts that let the model **pick the right tool** (or no tool) for the current step.

**Design rules**

- Describe each tool with **purpose, inputs, outputs, cost, and side effects**.
- Include a **"none of the above"** option — over-eager tool use is a real failure mode.
- For ambiguous routing, ask the model to **state its reasoning** before selecting.

**Example**

```text
Available tools:
- web_search(query)        → recent public information; cost: $$
- internal_kb(query)       → ACME documents; cost: $
- run_sql(query)           → finance warehouse; cost: $; SIDE EFFECTS: none
- send_email(to, subject)  → SIDE EFFECTS: irreversible — requires approval

Pick exactly one tool, or "none" if no tool is needed.
Return JSON: { "tool": "...", "reason": "..." }
```

---

### 🔁 Reflection prompts

**Definition.** Prompts that ask the model to **critique its own previous output** and produce an improved version.

**Pattern**

1. Generate draft.
2. Evaluate draft against the rules / rubric.
3. Rewrite if any rule fails. Repeat up to N times.

**Example**

```text
Here is your draft answer:
<draft>{{draft}}</draft>

Critique it against these rules:
- Every numerical claim cites a source from <context>.
- No sentence exceeds 25 words.
- Tone is neutral and analytical.

If any rule fails, output a corrected version. Otherwise, return the draft unchanged.
```

> **Caution.** Reflection improves quality but compounds latency and cost. Cap iterations.

---

### 🧪 Evaluator prompts

**Definition.** Prompts (often run by a *separate, smaller* model) that **score** an output for quality, correctness, or safety — used in evals and as runtime guardrails.

**What to evaluate**

- **Correctness** — does the answer match the ground truth?
- **Faithfulness** — is every claim supported by `<context>`?
- **Format compliance** — does it match the schema?
- **Safety** — does it violate any policy?

**Example — faithfulness evaluator**

```json
{
  "supported": true,
  "unsupported_claims": [],
  "score": 1.0
}
```

> **Best practice.** Use the **same evaluator prompts** in your offline test suite *and* as runtime guardrails. What you measure, you can defend.

---

## 2.3 Tool Use and Function Calling

> Tool use is what turns an LLM into an **agent**. Treat tools as a **public API surface** the model is calling — design them with the same care.

```
  model  ─── tool_use ──▶  your code  ─── tool_result ──▶  model
                              │
                              ▼
                      external system
```

---

### 📐 Tool schemas

**Definition.** A formal description — name, description, parameters, types — that tells the model **how to invoke a tool correctly**.

**Schema design rules**

- **Name.** Verb-led, snake_case (`search_orders`, `create_ticket`).
- **Description.** One paragraph: *what it does, when to use it, when **not** to*.
- **Parameters.** JSON Schema; mark required fields; use `enum` where bounded.
- **Outputs.** Document the result shape — the model uses this to plan next steps.
- **Side effects.** Make them explicit in the description.

**Example**

```json
{
  "name": "create_ticket",
  "description": "Create a Jira ticket. Use ONLY when the user explicitly asks to file an issue. Irreversible.",
  "input_schema": {
    "type": "object",
    "properties": {
      "project": { "type": "string", "enum": ["WEB", "API", "INFRA"] },
      "summary": { "type": "string", "maxLength": 120 },
      "priority": { "type": "string", "enum": ["low", "medium", "high"] }
    },
    "required": ["project", "summary"]
  }
}
```

---

### 🔧 Function calling

**Definition.** The runtime mechanism by which the model emits a **structured tool call** that your code executes, returning a `tool_result` for the next turn.

**The loop**

```
1. Send messages + tools  ──▶  model
2. Model returns: text  OR  tool_use(name, input)
3. Your code executes the tool; capture the result
4. Send tool_result back  ──▶  model
5. Repeat until model returns a final assistant text
```

**Engineering checklist**

- **Validate** every tool input against the schema **before** executing.
- **Time-box** every tool call; treat timeouts as recoverable errors.
- **Return structured results**, not raw blobs the model has to re-parse.
- **Log** each call with inputs, outputs, latency, and cost.

**Example — Python sketch**

```python
while True:
    resp = client.messages.create(model=..., tools=TOOLS, messages=history)
    if resp.stop_reason == "end_turn":
        break
    for block in resp.content:
        if block.type == "tool_use":
            result = dispatch(block.name, block.input)
            history.append({"role": "assistant", "content": resp.content})
            history.append({
                "role": "user",
                "content": [{"type": "tool_result", "tool_use_id": block.id, "content": result}]
            })
```

---

### 🌐 External API orchestration

**Definition.** Wrapping third-party APIs (CRM, payments, search, weather, …) as tools the agent can compose.

**Production-grade wrappers**

- **Auth boundary.** The wrapper holds credentials; the model never sees them.
- **Scoping.** Pass the calling user's identity through; enforce permissions server-side.
- **Rate limits & retries.** Exponential backoff with jitter, circuit breakers.
- **Schema mapping.** Convert messy vendor JSON into a clean, model-friendly shape.
- **Cost & quota tracking.** Per-tool, per-tenant, per-run.

**Example.** A `search_customers(query)` tool wraps Salesforce SOQL — the model sees a clean `[{id, name, segment, mrr}]` array; the wrapper handles auth, pagination, and field mapping.

---

### 🛡️ Safe tool execution

**Definition.** The defensive layer between the model's **intent** to call a tool and the **actual side effect** in the world.

**Layered defense**

```
 model intent
     │
     ▼
 ┌─────────────────┐   schema + business-rule validation
 │  Pre-check      │
 └────────┬────────┘
          ▼
 ┌─────────────────┐   allow-list, rate-limit, identity
 │  Authorization  │
 └────────┬────────┘
          ▼
 ┌─────────────────┐   sandbox · dry-run · simulate
 │  Execution      │
 └────────┬────────┘
          ▼
 ┌─────────────────┐   diff preview · human approval (high-impact)
 │  Confirmation   │
 └─────────────────┘
```

**Non-negotiables for any side-effecting tool**

- **Idempotency keys** so retries don't double-charge.
- **Dry-run / preview** mode for destructive operations.
- **Human approval gate** for irreversible actions above a threshold.
- **Full audit log** — who, what, when, why, with what input.

> **Mantra.** *"Stochastic decisions, deterministic actions."* The LLM may decide; the tool layer must execute predictably.

---

## 2.4 Context Engineering

> If prompt engineering is "what we tell the model", **context engineering is what we let the model see** — and how.

The window is finite, latency scales with it, and quality degrades when it is poorly organized. Context is a budget; spend it deliberately.

---

### 🪟 Long-context optimization

**Definition.** Practices that maximize **effective recall and reasoning quality** within a long context window — without paying for tokens that don't earn their keep.

**Techniques**

- **Place the question last.** End the prompt with the question and required output format.
- **Place the most important evidence first or last** — fight "lost in the middle".
- **Tag every block** so the model can reference it (`<doc id="3">`).
- **Prune aggressively.** Boilerplate, repeated headers, navigation chrome.
- **Summarize the rest.** Older history → rolling summary; long docs → section abstracts.

**Cost intuition**

> Doubling input tokens roughly doubles cost and latency. Long context is a *tool*, not a *default*.

---

### 🔎 Retrieval strategies

**Definition.** Techniques that fetch the **most relevant slices** of a corpus into context, instead of stuffing everything.

**Retrieval choices**

| Method | When to use |
|---|---|
| Lexical (BM25) | Exact terms, codes, IDs |
| Dense (embeddings) | Semantic similarity, paraphrase |
| Hybrid (lexical + dense + re-rank) | Production default |
| Structured (SQL / graph) | Tabular, relational, entity questions |
| Tool-driven (live API) | Freshness-sensitive data |

**Quality levers**

- **Chunking** — semantic, not naive byte windows.
- **Metadata filters** — tenant, date, doc type, ACLs.
- **Re-rankers** — a cross-encoder over the top-K candidates.
- **Citations** — return chunk IDs the agent must cite.

**Example.** A support copilot uses hybrid retrieval over the help-center corpus, filters by product version, re-ranks the top 50 to top 5, and requires the answer to cite each chunk used.

---

### 🗜️ Compression

**Definition.** Reducing context size while **preserving decision-relevant information**.

**Patterns**

- **Rolling conversation summary.** Keep the last N turns verbatim; summarize everything older.
- **Hierarchical summaries.** Section → chapter → document abstracts.
- **Selective extraction.** Pull only the entities/figures the task needs.
- **Reference, don't repeat.** Store the artifact once; pass an ID, not the body.

**Trade-off**

> Compression saves tokens but **drops nuance**. Validate that compressed context still passes your task-specific evals.

---

### 🥞 Context layering

**Definition.** Composing context from **multiple sources** in a stable, predictable order so the agent always knows where to look.

**A reference layering**

```
┌──────────────────────────┐
│ 1. Identity & policy     │  stable across the session
├──────────────────────────┤
│ 2. Tool inventory        │  changes rarely
├──────────────────────────┤
│ 3. Long-term memory      │  user / org facts (RAG)
├──────────────────────────┤
│ 4. Task context          │  retrieved evidence for THIS task
├──────────────────────────┤
│ 5. Conversation state    │  rolling summary + recent turns
├──────────────────────────┤
│ 6. Current request       │  user message
└──────────────────────────┘
```

**Why ordering matters**

- The **top** is high-trust, slow-changing — the model treats it as ground truth.
- The **bottom** is the active request — most recent, highest attention.
- The **middle** is where retrieved evidence and memory live; tag everything.

> **Principle.** A good context layering is **inspectable** — for any output, an engineer can trace which block influenced which claim.

---

## 🎯 Module 2 — Takeaways

1. **Prompts are specifications.** Specific, structured, testable, versioned.
2. **Use semantic structure** — XML tags and labelled sections — to separate rules, context, and input.
3. **Deliberate reasoning beats clever phrasing** for hard tasks; cap it to control cost.
4. **Agents need a prompt portfolio** — planner, router, worker, reflector, evaluator.
5. **Treat tools as a public API** — clean schemas, validated inputs, structured outputs.
6. **Stochastic decisions, deterministic actions.** Guard every side effect.
7. **Context is a budget** — retrieve, compress, order, and tag what you spend it on.
8. **Layer context predictably** so behavior is debuggable and reproducible.

> ⬅️ **Previous:** [Module 1 — Introduction to Agentic AI](Module_1_Introduction_to_Agentic_AI.md)
> ➡️ **Next:** [Module 3 — MCP (Model Context Protocol)](Module_3_MCP_Model_Context_Protocol.md)
