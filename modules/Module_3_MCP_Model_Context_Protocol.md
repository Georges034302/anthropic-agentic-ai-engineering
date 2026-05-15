# Module 3 — MCP (Model Context Protocol)

> **Module goal.** Understand MCP as the **USB-C of AI** — a standard protocol that lets any model connect to any tool, data source, or capability without bespoke glue per vendor.

## Table of Contents

- [3.1 Introduction to MCP](#31-introduction-to-mcp)
- [3.2 MCP Components](#32-mcp-components)
- [3.3 Building MCP Servers](#33-building-mcp-servers)
- [3.4 Enterprise MCP Design](#34-enterprise-mcp-design)

---

## 3.1 Introduction to MCP

> Before MCP, every AI app re-implemented "connect model X to system Y". MCP turns that NxM integration matrix into a clean N+M.

```
   Without MCP                         With MCP
   ───────────                         ────────
   Model A ─┬─ Tool 1                  Model A ─┐         ┌─ Tool 1
            ├─ Tool 2                            │         ├─ Tool 2
            └─ Tool 3                  Model B ──┼── MCP ──┼─ Tool 3
   Model B ─┬─ Tool 1                            │         ├─ Tool 4
            ├─ Tool 2                  Model C ─┘         └─ Tool 5
            └─ ...                       (any client)       (any server)
```

---

### 🌐 Why MCP matters

**Definition.** **MCP (Model Context Protocol)** is an open, JSON-RPC-based protocol that standardizes how AI applications expose **tools, data, and prompts** to language models.

**The problem it solves**

- **NxM integration explosion** — every new model × every new system = a custom adapter.
- **Fragmented capability surfaces** — each app reinvents auth, schemas, and discovery.
- **Lock-in** — tools written for one platform don't move to another.

**What MCP gives you**

- A **transport-agnostic protocol** (stdio, HTTP, SSE).
- A **uniform capability surface** — tools, resources, prompts, sampling.
- **Discovery** — clients can list what a server offers at runtime.
- **Portability** — the same MCP server works for any compliant client.

> **Mental model.** MCP is to AI tools what **LSP** is to IDE language features: a small, well-defined protocol that breaks the integration deadlock.

---

### 🏗️ MCP architecture

**Definition.** MCP defines a **client/server** architecture where the **host** application embeds **clients** that connect to one or more **servers** exposing capabilities.

```
   ┌──────────────────────────────────────┐
   │ Host (Claude Desktop, IDE, agent)    │
   │  ┌──────────┐   ┌──────────┐         │
   │  │ Client 1 │   │ Client 2 │   ...   │
   │  └────┬─────┘   └────┬─────┘         │
   └───────┼──────────────┼───────────────┘
           │              │
       JSON-RPC       JSON-RPC
           │              │
   ┌───────▼──────┐ ┌─────▼──────┐
   │  Server A    │ │  Server B  │   ...
   │ (filesystem) │ │  (Jira)    │
   └──────────────┘ └────────────┘
```

**Roles**

| Role | What it is |
|---|---|
| **Host** | The AI application the user interacts with. |
| **Client** | A connection to one server, owned by the host. |
| **Server** | A process exposing tools/resources/prompts over MCP. |
| **Transport** | stdio (local subprocess), HTTP+SSE (remote). |

**Lifecycle**

1. Host launches/connects a server.
2. **Initialize** handshake exchanges capabilities.
3. Host **lists** tools / resources / prompts.
4. Model invokes them via the host; results flow back.
5. Connection closes cleanly on shutdown.

---

## 3.2 MCP Components

> Four primitives. Compose them well and you can model almost any integration.

```
   ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐
   │   Tools    │  │ Resources  │  │  Prompts   │  │  Sampling  │
   │  (verbs)   │  │  (nouns)   │  │ (templates)│  │ (callback) │
   └────────────┘  └────────────┘  └────────────┘  └────────────┘
```

---

### 🛠️ Tools

**Definition.** Server-exposed **functions** the model can call to take action or fetch information.

**Properties**

- Named, typed inputs (JSON Schema).
- Structured outputs (text, JSON, images).
- May have side effects — must be marked clearly.

**Example schema**

```json
{
  "name": "create_jira_issue",
  "description": "Create a Jira issue in a permitted project.",
  "inputSchema": {
    "type": "object",
    "required": ["project", "summary"],
    "properties": {
      "project":  { "type": "string", "enum": ["WEB","API","INFRA"] },
      "summary":  { "type": "string", "maxLength": 120 },
      "priority": { "type": "string", "enum": ["low","medium","high"] }
    }
  }
}
```

---

### 📂 Resources

**Definition.** **Read-only data** the server exposes via stable URIs — files, DB rows, knowledge-base articles.

**Why separate from tools?**

- Resources are **referable** (URI-addressable), **listable**, and **subscribable** for change events.
- Hosts can attach resources to model context **without** invoking a tool call.

**Example**

```
file:///repo/docs/architecture.md
jira://ACME/issue/WEB-1234
db://orders/2026-04
```

---

### 🧾 Prompts

**Definition.** **Reusable, parameterized prompt templates** the server publishes for the host/user to invoke.

**Use cases**

- "Summarize this PR" with a standard rubric.
- "Generate a Terraform module" with org conventions baked in.
- Onboarding workflows packaged as a one-click template.

**Shape**

```json
{
  "name": "review_pull_request",
  "description": "Standard ACME PR review prompt.",
  "arguments": [
    { "name": "repo", "required": true },
    { "name": "pr_number", "required": true }
  ]
}
```

---

### 🔁 Sampling

**Definition.** A protocol feature that lets a **server request the host's model** to generate text on its behalf — enabling agentic servers without bundling their own model.

**Why it matters**

- Servers stay lightweight; the host owns model choice and billing.
- Enables sub-agents and recursive tool flows.
- Keeps user consent in the host (the user sees what the server is asking for).

---

## 3.3 Building MCP Servers

> A good MCP server is a **focused capability surface**, not a kitchen sink.

```
   Pick a domain  ─▶  Define tools/resources  ─▶  Implement  ─▶  Test  ─▶  Distribute
```

---

### 🧱 Server skeleton (TypeScript)

```ts
import { Server } from "@modelcontextprotocol/sdk/server";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio";

const server = new Server(
  { name: "acme-jira", version: "1.0.0" },
  { capabilities: { tools: {}, resources: {} } }
);

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "create_jira_issue",
    description: "Create a Jira issue in a permitted project.",
    inputSchema: { /* ... */ }
  }]
}));

server.setRequestHandler("tools/call", async (req) => {
  if (req.params.name === "create_jira_issue") {
    const result = await jira.create(req.params.arguments);
    return { content: [{ type: "text", text: JSON.stringify(result) }] };
  }
  throw new Error("Unknown tool");
});

await server.connect(new StdioServerTransport());
```

---

### ✅ Design checklist

- **Single responsibility.** One domain per server (Jira, Postgres, S3, Calendar).
- **Stable IDs.** Tool and resource URIs should not change across versions.
- **Schemas as contracts.** Validate every input; return structured errors.
- **Idempotency.** Side-effecting tools accept an idempotency key.
- **Pagination & limits.** Protect against runaway lists.
- **Logging.** Structured logs for every call, with correlation IDs.
- **Versioning.** Semver the server; advertise via the `initialize` response.

---

### 🧪 Testing

| Layer | What you test |
|---|---|
| **Unit** | Each tool handler in isolation. |
| **Protocol** | Replay JSON-RPC fixtures; validate against MCP spec. |
| **Integration** | Run the server against a real client (Claude Desktop, an MCP CLI). |
| **Eval** | Have an agent perform real tasks; measure success rate, cost. |

---

### 📦 Distribution

- **Local subprocess** (stdio) for desktop and dev workflows.
- **Hosted HTTP + SSE** for shared, multi-user deployments.
- **Container image** for reproducible enterprise rollout.
- Publish to internal or public **MCP registries** with clear capability docs.

---

## 3.4 Enterprise MCP Design

> Inside an enterprise, MCP isn't just a protocol — it's the **integration fabric** for agents. Designed well, it gives you reuse, governance, and isolation by default.

---

### 🏢 Reference architecture

```
   ┌──────────────────────────────────────────────┐
   │              Agent / Copilot Host            │
   └───────┬───────────────────────────┬──────────┘
           │                           │
   ┌───────▼────────┐         ┌────────▼────────┐
   │  MCP Gateway   │         │  Local servers  │   (dev, IDE)
   │  (auth, audit, │         └─────────────────┘
   │   policy, rate)│
   └───┬─────┬─────┬┘
       │     │     │
   ┌───▼─┐ ┌─▼──┐ ┌▼────┐
   │ ERP │ │CRM │ │Data │   ... domain MCP servers
   └─────┘ └────┘ └─────┘
```

---

### 🔐 Identity, auth, policy

- **Identity propagation.** The host passes the user's identity to the gateway; the gateway exchanges it for a downstream token (OAuth on-behalf-of).
- **Per-tool authorization.** Allow-lists by user/role/tenant; default-deny for anything destructive.
- **Policy engine** (e.g. OPA) evaluates each call: *who, what, when, why, on what data*.
- **Secrets stay in the gateway/server.** The model never sees credentials.

---

### 📋 Governance

- **Catalog** every server: owner, domain, version, data classification, SLAs.
- **Schema review** for new tools — names, descriptions, side effects, blast radius.
- **Change management** — semver, deprecation windows, telemetry-driven retirement.
- **Cost controls** — per-tool budgets and quotas at the gateway.

---

### 📈 Observability

- **Structured logs** for every tool invocation: principal, tool, inputs (redacted), output size, latency, cost, outcome.
- **Distributed traces** that span host → gateway → server → backend.
- **Audit trail** sufficient for regulatory review (SOX, HIPAA, GDPR).
- **Dashboards** per domain server: usage, errors, p95 latency, cost.

---

### 🛡️ Safety controls

| Risk | Control |
|---|---|
| Prompt injection via resource content | Tag untrusted content; restrict tool calls when present. |
| Over-broad tool access | Per-session capability negotiation; least privilege. |
| Destructive actions | Require confirmation; dry-run mode. |
| Credential exfiltration | Secrets never cross the model boundary. |
| Data leakage across tenants | Tenant ID propagated and enforced server-side. |

> **Principle.** The gateway is your **enforcement point**. Servers should *also* enforce — but never *only* trust the client.

---

## 🎯 Module 3 — Takeaways

1. **MCP collapses the NxM integration problem** into a clean N+M via a standard protocol.
2. **Four primitives** — tools, resources, prompts, sampling — cover most integration needs.
3. **Hosts run clients; servers expose capabilities.** Transports are stdio or HTTP+SSE.
4. **Build focused servers** — one domain, clean schemas, stable IDs, structured errors.
5. **Treat the gateway as your enforcement point** — auth, policy, audit, quotas.
6. **Identity propagates end-to-end.** Secrets never reach the model.
7. **Governance is mandatory at scale** — catalog, version, deprecate, observe.

> ⬅️ **Previous:** [Module 2 — Prompt Engineering and Context Design](Module_2_Prompt_Engineering_and_Context_Design.md)
> ➡️ **Next:** [Module 4 — Building AI Agents](Module_4_Building_AI_Agents.md)
