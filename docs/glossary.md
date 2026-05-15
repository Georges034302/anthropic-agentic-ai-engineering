# Glossary

Definitions of key terms used throughout the **Anthropic Agentic AI Engineering** course. Terms are organized alphabetically. Where applicable, links point to the module where the concept is introduced.

---

## A

- **Agent** — An AI system that perceives a goal, plans steps, selects tools, and acts autonomously to achieve an outcome, rather than only responding to a single prompt. See [Module 4](../modules/Module_4_Building_AI_Agents.md).
- **Agentic Workflow** — A multi-step process coordinated by one or more agents, typically involving planning, tool use, and reflection. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#11-what-is-agentic-ai).
- **Autonomous Task Execution** — An agent's ability to carry out multi-step tasks with minimal human intervention, including retries and self-correction. See [Module 4](../modules/Module_4_Building_AI_Agents.md#43-autonomous-task-execution).

## C

- **Constitutional AI** — A safety technique where models are trained or guided by a set of explicit principles to produce safer, more aligned responses. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#12-foundations-of-modern-ai-models).
- **Context Engineering** — The practice of selecting, structuring, compressing, and layering information placed in a model's context window to maximize task performance. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#24-context-engineering).
- **Context Window** — The maximum amount of tokens a model can consider at once when generating a response.

## E

- **Enterprise Copilot** — A domain-specialized AI assistant integrated into business workflows to augment knowledge workers. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#14-enterprise-ai-use-cases).
- **Evaluator Prompt** — A prompt designed to assess the quality, correctness, or safety of another model's output. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#22-structured-agent-prompts).

## F

- **Function Calling** — A mechanism allowing a model to invoke external code or APIs by emitting a structured tool-call payload. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#23-tool-use-and-function-calling).

## G

- **Guardrails** — Policies, validations, and runtime controls that constrain agent behavior to keep it safe, on-task, and compliant. See [Module 4](../modules/Module_4_Building_AI_Agents.md#44-agent-guardrails).

## K

- **Knowledge Governance** — Policies and controls over how enterprise knowledge is curated, accessed, and exposed to AI systems. See [Module 6](../modules/Module_6_RAG_and_Enterprise_Knowledge_Agents.md#63-knowledge-governance).

## L

- **Long-Context Processing** — Techniques for handling very large inputs (documents, transcripts, code bases) within an extended context window. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#12-foundations-of-modern-ai-models).

## M

- **MCP (Model Context Protocol)** — An open protocol that standardizes how AI applications connect to external tools, data, and services. See [Module 3](../modules/Module_3_MCP_Model_Context_Protocol.md).
- **MCP Server** — A process that exposes resources, tools, and prompts to MCP-compatible clients via the protocol. See [Module 3](../modules/Module_3_MCP_Model_Context_Protocol.md#33-building-mcp-servers).
- **Memory System** — Storage and retrieval mechanisms (short-term, long-term, episodic, semantic) that let agents persist and reuse information. See [Module 4](../modules/Module_4_Building_AI_Agents.md#42-memory-systems).
- **Multi-Agent System** — An architecture in which multiple specialized agents collaborate, negotiate, or delegate to accomplish a task. See [Module 5](../modules/Module_5_Multi_Agent_Systems.md).

## O

- **Observability** — Logging, tracing, and metrics that make agent behavior transparent and debuggable in production. See [Module 8](../modules/Module_8_Production_Agent_Systems.md#82-observability-and-evaluation).
- **Orchestration** — The coordination of agents, tools, and workflows to deliver an end-to-end business process.

## P

- **Planner Prompt** — A prompt that instructs a model to decompose a goal into ordered, executable steps. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#22-structured-agent-prompts).
- **Production Architecture** — The deployment topology, scaling strategy, and supporting infrastructure for running agents reliably. See [Module 8](../modules/Module_8_Production_Agent_Systems.md#81-production-architecture).
- **Prompt Engineering** — The discipline of crafting model inputs to produce reliable, high-quality outputs. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#21-prompt-engineering-principles).

## R

- **RAG (Retrieval-Augmented Generation)** — A pattern where a model retrieves relevant external knowledge and uses it to ground its responses. See [Module 6](../modules/Module_6_RAG_and_Enterprise_Knowledge_Agents.md#61-enterprise-rag).
- **Agentic RAG** — A RAG variant where an agent dynamically decides when, what, and how to retrieve, often across multiple sources. See [Module 6](../modules/Module_6_RAG_and_Enterprise_Knowledge_Agents.md#62-agentic-rag).
- **Reflection Prompt** — A prompt that asks a model to critique and improve its own previous output. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#22-structured-agent-prompts).
- **Responsible AI** — Practices ensuring AI systems are safe, fair, transparent, and aligned with ethical and regulatory norms. See [Module 8](../modules/Module_8_Production_Agent_Systems.md#83-responsible-ai-operations).

## S

- **Streaming Response** — Token-by-token delivery of a model's output, enabling lower perceived latency and incremental UI updates. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#13-ai-api-fundamentals).
- **Structured Output** — Model output produced according to a defined schema (e.g. JSON), making it machine-consumable. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#13-ai-api-fundamentals).
- **System Prompt** — A high-priority instruction that defines an agent's persona, constraints, and operational rules. See [Module 1](../modules/Module_1_Introduction_to_Agentic_AI.md#13-ai-api-fundamentals).

## T

- **Tool Schema** — A formal description (name, parameters, types) that tells a model how to invoke a tool correctly. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#23-tool-use-and-function-calling).
- **Tool Use** — A model's ability to call external functions or services to extend its capabilities beyond text generation. See [Module 2](../modules/Module_2_Prompt_Engineering_and_Context_Design.md#23-tool-use-and-function-calling).

## W

- **Workflow Agent** — An agent specialized to drive a defined enterprise business process from start to completion. See [Module 5](../modules/Module_5_Multi_Agent_Systems.md#54-enterprise-workflow-agents).
