# Module 6 — RAG and Enterprise Knowledge Agents

> **Module goal.** Engineer **trustworthy, governed, agent-driven access to enterprise knowledge** — so answers are grounded, citable, fresh, and safe.

## Table of Contents

- [6.1 Enterprise RAG](#61-enterprise-rag)
- [6.2 Agentic RAG](#62-agentic-rag)
- [6.3 Knowledge Governance](#63-knowledge-governance)
- [6.4 Building Enterprise Knowledge Agents](#64-building-enterprise-knowledge-agents)

---

## 6.1 Enterprise RAG

> RAG turns the model from a *guesser* into a *reader*. Done well, it is the cheapest way to make AI useful on **your** data.

```
   Question ─▶ Retrieve ─▶ Rank ─▶ Augment ─▶ Generate ─▶ Cite
```

---

### 📚 Definition

**Retrieval-Augmented Generation (RAG)** is a pattern where the model **fetches relevant documents at query time** and uses them as grounding context, instead of relying on parametric memory alone.

---

### 🧱 Reference pipeline

```
   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌────────────┐
   │ Ingest     │ ─▶│ Chunk +    │ ─▶│ Embed +    │ ─▶│ Vector +   │
   │ (sources)  │   │ Normalize  │   │ Enrich     │   │ Lexical DB │
   └────────────┘   └────────────┘   └────────────┘   └─────┬──────┘
                                                            │
   ┌────────────┐   ┌────────────┐   ┌────────────┐   ┌─────▼──────┐
   │ Generate + │ ◀─│ Augment    │ ◀─│ Re-rank    │ ◀─│ Retrieve   │
   │ Cite       │   │ context    │   │            │   │ (hybrid)   │
   └────────────┘   └────────────┘   └────────────┘   └────────────┘
```

---

### 🧩 Engineering choices

| Stage | Lever | Why it matters |
|---|---|---|
| **Ingest** | Source connectors, freshness | Stale data → wrong answers |
| **Chunk** | Semantic boundaries, overlap | Bad chunks = bad recall |
| **Enrich** | Titles, tags, ACLs, dates | Filtering and citation quality |
| **Embed** | Model + dimensions | Recall ceiling |
| **Index** | Vector + lexical (BM25) | Hybrid beats either alone |
| **Retrieve** | Top-K, filters, MMR | Precision vs diversity |
| **Re-rank** | Cross-encoder | Big quality jump for small cost |
| **Augment** | Order, labelling, schema | Fight "lost in the middle" |
| **Generate** | Citation enforcement | Trust + auditability |

---

### ⚖️ RAG vs long-context vs fine-tuning

| Use… | When |
|---|---|
| **RAG** | Knowledge changes; corpus is large; you need citations |
| **Long-context** | Corpus fits; you want simplicity over infra |
| **Fine-tuning** | You need *behaviour / style / format* changes, not new facts |

> **Default.** Build RAG first. Add long-context for niche flows. Fine-tune only when prompts and RAG demonstrably can't reach quality targets.

---

### 🧪 Example — citation contract

```text
Answer using ONLY <context>. After every factual sentence, append a citation
like [doc:42#chunk:7]. If <context> does not answer the question, say so.
```

---

## 6.2 Agentic RAG

> Classic RAG retrieves *once*. Agentic RAG decides **whether, what, when, and how** to retrieve — and **iterates** until the answer is good enough.

```
                ┌─────────────┐
   question ──▶ │   Planner   │
                └──┬───────┬──┘
                   │       │
            retrieve?   answer directly?
                   │
                   ▼
             ┌──────────┐
             │ Retriever│  may use multiple sources / tools
             └────┬─────┘
                  ▼
             ┌──────────┐
             │  Critic  │ enough evidence? gaps? conflicts?
             └────┬─────┘
                  ▼
              loop or finalize
```

---

### 🧠 What changes vs classic RAG

- **Multiple retrieval calls** with refined queries.
- **Multiple sources** (vector DB, SQL, APIs, web) chosen by the planner.
- **Self-checking** — the agent verifies citations support each claim.
- **Replanning** on low-confidence answers.

---

### 🧰 Patterns

| Pattern | Idea |
|---|---|
| **Query rewriting** | Expand or decompose the question before retrieval |
| **Multi-hop retrieval** | Use first-hop results to drive second-hop queries |
| **Tool-routed retrieval** | Pick vector vs SQL vs API per sub-question |
| **Self-RAG / critic loop** | Generate → critique citations → retry |
| **HyDE** | Generate a hypothetical answer; embed it; retrieve neighbours |

---

### 🧪 Example — multi-hop research question

> *"Which of our top-10 customers had a price change in the last quarter, and what was the impact on revenue?"*
>
> 1. SQL retriever → top-10 customers by revenue.
> 2. CRM retriever → recent price-change events for those accounts.
> 3. Warehouse retriever → revenue deltas pre/post change.
> 4. Writer agent synthesizes a table with citations to each source.

---

### ⚠️ Failure modes

- **Over-retrieval** — fetching too much, drowning the model.
- **Hallucinated citations** — model invents IDs; *always* validate citations exist in `<context>`.
- **Loop without progress** — refining queries that never converge; cap iterations.

---

## 6.3 Knowledge Governance

> A knowledge agent is only as trustworthy as the **governance around its data**.

---

### 🔐 Access control

- **ACL-aware retrieval.** The retriever filters by the *user's* permissions, not the index's.
- **Tenant isolation** at every layer (index partitioning, query filters, audit).
- **Data classification** tags (Public, Internal, Confidential, Restricted) propagated end-to-end.
- **Default-deny** for unclassified content.

---

### 📚 Source of truth & freshness

- **Authoritative sources** are explicitly listed; retrieval prefers them.
- **Conflict resolution** rules: newer, higher-rank, or human-curated wins.
- **Freshness SLAs** per source (CRM = minutes, policies = daily, archives = weekly).
- **De-duplication** at ingest to prevent the same fact from drowning others.

---

### 🛡️ Safety & compliance

| Concern | Control |
|---|---|
| PII leakage | Detect & redact at ingest; mask in answers |
| Right to erasure (GDPR) | Hard-delete by source ID + re-index sweep |
| Regulated data (HIPAA, PCI) | Separate index, separate model deployment |
| Prompt injection via documents | Tag untrusted sources; restrict tool calls |
| Source attribution | Mandatory citations; refuse without evidence |

---

### 📈 Quality operations

- **Eval set** of golden Q&A pairs per domain.
- **Continuous evaluation** on every index/model change.
- **Human-in-the-loop feedback** captured per answer (👍/👎 + reason).
- **Drift detection** — alert when answer quality or citation rate degrades.

> **Mantra.** *"If you can't audit it, you can't trust it. If you can't trust it, you can't deploy it."*

---

## 6.4 Building Enterprise Knowledge Agents

> A knowledge agent = **RAG infrastructure + an agent loop + governance + UX**.

---

### 🏗️ Reference architecture

```
   ┌──────────────────────────────────────────────────┐
   │                 Knowledge Agent                  │
   │  ┌──────────┐  ┌──────────┐  ┌─────────────┐     │
   │  │ Planner  │─▶│Retriever │─▶│  Synthesizer │    │
   │  └──────────┘  └────┬─────┘  └────┬─────────┘    │
   │                     │              │              │
   │                     ▼              ▼              │
   │             ┌────────────┐   ┌────────────┐       │
   │             │  Critic    │   │  Citations │       │
   │             └────────────┘   └────────────┘       │
   └──────────────────────────────────────────────────┘
                  │              │
                  ▼              ▼
        ┌──────────────┐  ┌──────────────┐
        │ Vector + BM25│  │ Tools (SQL,  │
        │   indices    │  │ APIs, MCP)   │
        └──────────────┘  └──────────────┘
```

---

### 🧩 Build checklist

- [ ] Source inventory with owners, sensitivity, freshness SLAs.
- [ ] Ingestion with chunking, enrichment, ACL propagation.
- [ ] Hybrid index (vector + lexical) per security boundary.
- [ ] Retrieval policy: filters, top-K, re-ranker.
- [ ] Agent loop: planner → retriever → critic → synthesizer.
- [ ] Citation enforcement and verification.
- [ ] Eval harness with golden questions per domain.
- [ ] Feedback loop captured into observability.
- [ ] Refusal behaviour when evidence is insufficient.

---

### 🧱 UX patterns that build trust

- **Inline citations** clickable to the source chunk.
- **"Why this answer?"** view showing retrieved evidence.
- **Confidence indicator** (high / medium / low).
- **"No reliable answer"** when evidence is weak — a *feature*, not a failure.
- **Feedback widget** every answer.

---

### 🧪 Example — answer envelope

```json
{
  "answer_md": "Q3 EMEA churn was **4.2%**, up from 3.6% in Q2 [doc:18#c:3].",
  "citations": [
    { "id": "doc:18#c:3", "title": "Q3 KPI Pack", "url": "...", "score": 0.91 }
  ],
  "confidence": "high",
  "retrieval": { "method": "hybrid+rerank", "top_k": 50, "kept": 5 },
  "policy": { "data_class": "Internal", "user_scope": "EMEA-Sales" }
}
```

---

## 🎯 Module 6 — Takeaways

1. **RAG is the default for "AI on your data".** Cheapest path to grounded, citable answers.
2. **Hybrid retrieval + re-rank** beats vector-only in almost every benchmark and product.
3. **Agentic RAG iterates** — query rewriting, multi-hop, tool routing, self-critique.
4. **Governance is the product.** ACLs, classification, freshness, audit, refusal.
5. **Enforce citations.** No source → no claim.
6. **Eval continuously.** Golden sets, human feedback, drift detection.
7. **UX builds trust** — show citations, confidence, and the courage to say "I don't know".

> ⬅️ **Previous:** [Module 5 — Multi-Agent Systems](Module_5_Multi_Agent_Systems.md)
> ➡️ **Next:** [Module 7 — AI-Augmented Software Engineering](Module_7_AI_Augmented_Software_Engineering.md)
