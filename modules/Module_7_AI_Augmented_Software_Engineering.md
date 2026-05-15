# Module 7 — AI-Augmented Software Engineering

> **Module goal.** Apply agentic AI **across the entire software lifecycle** — from coding and review to security, deployment, and operations — without sacrificing engineering rigor.

> **Agentic AI Engineering Focus**
>
> This module is dedicated to agentic AI in software engineering—engineering agents that augment, automate, and secure the software development lifecycle (SDLC). The focus is on agentic workflows for coding, review, and DevOps, not on generic automation or unrelated SD topics.

## Table of Contents

- [7.1 AI-Assisted Development](#71-ai-assisted-development)
- [7.2 AI-Augmented SDLC](#72-ai-augmented-sdlc)
- [7.3 Secure AI Development](#73-secure-ai-development)
- [7.4 AI DevOps and Deployment](#74-ai-devops-and-deployment)

---

## 7.1 AI-Assisted Development

> **Agentic AI in software engineering means building agents that amplify, automate, and secure the SDLC—while keeping humans in the loop.**
>
> In this course, agentic software engineering is about practical, production-grade augmentation, not generic code generation.

```
   Intent ─▶ AI ─▶ Draft ─▶ Engineer reviews ─▶ Tests ─▶ Merge
                              ▲           │
                              └─ iterate ─┘
```

---

### 🧠 The maturity ladder

| Level | What the AI does | Engineer's role |
|---|---|---|
| **L0 Autocomplete** | Single-line suggestions | Type, accept/reject |
| **L1 Snippet** | Multi-line block from comment | Review and edit |
| **L2 Function** | Whole function from spec/test | Validate behavior |
| **L3 Multi-file edit** | Refactors across files | Review the diff |
| **L4 Task agent** | Executes a ticket end-to-end (branch + PR + tests) | Reviews the PR |
| **L5 Autonomous SWE** | Manages a backlog of small tasks | Sets policy, audits |

> **Pragmatic stance.** Most teams live at L1–L3 day-to-day, with L4 task agents handling well-bounded chores (deps upgrades, test gen, lint fixes).

---

### 🛠️ Tooling stack

- **In-IDE assistant** (completion, chat, inline edits).
- **Repo-aware code search** + symbol indexing for grounded answers.
- **Patch & PR primitives** for multi-file edits.
- **Test runner** the agent can invoke (verify-by-execution).
- **Sandbox** for untrusted code execution.
- **Linters & type checkers** as deterministic guardrails.

---

### 🧪 Example — task-agent flow

```
1. Issue: "Add pagination to /orders endpoint"
2. Agent searches the repo, identifies handler + tests
3. Agent drafts a patch, runs the test suite — fails
4. Agent inspects failures, revises patch, runs again — passes
5. Agent runs lint + types — passes
6. Agent opens PR with rationale and links to the failing→passing run
7. Engineer reviews, requests one tweak, merges
```

---

### 📏 Engineering hygiene with AI

- **Specs first.** A clear spec or failing test makes AI 10x more useful.
- **Small diffs.** Force the agent to keep changes scoped and reviewable.
- **Conventional commits.** Make AI-authored history searchable.
- **Diff-only review.** Engineers review the diff, not the prompt.
- **Don't accept what you don't understand.**

---

## 7.2 AI-Augmented SDLC

> Every stage of the SDLC has an AI surface. The teams that win **wire them together**.

```
 Plan ─▶ Design ─▶ Code ─▶ Review ─▶ Test ─▶ Build ─▶ Deploy ─▶ Operate
   │       │       │        │        │       │        │         │
   ▼       ▼       ▼        ▼        ▼       ▼        ▼         ▼
 stories  ADRs   patches  reviews  tests  pipelines  changes   incidents
```

---

### 📋 Stage-by-stage

| Stage | AI surface |
|---|---|
| **Plan** | Story drafting, acceptance-criteria expansion, estimation hints |
| **Design** | ADR drafting, diagram generation, trade-off analysis |
| **Code** | Inline assist, task agents, refactor agents |
| **Review** | PR summary, risk flags, suggested fixes, security checks |
| **Test** | Unit/property/E2E test generation, mutation hints |
| **Build** | Pipeline authoring, error explanation, flake triage |
| **Deploy** | Release-note drafting, rollout plan, change-impact analysis |
| **Operate** | Alert triage, log summarization, incident drafting, postmortem help |

---

### 🧩 Cross-cutting capabilities

- **Repo grounding** — every assist is informed by the actual codebase.
- **Knowledge grounding** — RAG over architecture docs, runbooks, ADRs.
- **Action grounding** — agents take actions via PRs, issues, pipelines — never silent merges.
- **Telemetry** — every AI suggestion that lands in main is traceable.

---

### 🧱 Reference integration

```
   Engineer ◀──▶ IDE assistant ──┐
                                 │
        Issue/PR comments ◀──▶ Repo agent ──┐
                                            │
                Build/Deploy events ◀──▶ Ops agent
                                            │
                                            ▼
                                    Org RAG + tools
```

---

### 📈 What to measure

- **Cycle time** (issue open → merged → deployed).
- **PR throughput** and **review time**.
- **Defect escape rate** (AI-touched vs not).
- **Test coverage** delta and **flake rate**.
- **Acceptance rate** of AI suggestions (in IDE, in PRs).

> **Trap.** Optimizing "AI suggestion acceptance" alone leads to noisier code. Always pair it with quality and defect metrics.

---

## 7.3 Secure AI Development

> Agents that touch code, secrets, and prod are a **new attack surface**. Treat them like any other privileged automation.

---

### 🛡️ Threat model

| Threat | Example |
|---|---|
| **Prompt injection** | Malicious comment in a doc/PR steers the agent |
| **Tool abuse** | Agent tricked into running destructive commands |
| **Secret leakage** | Keys/tokens echoed into prompts or logs |
| **Supply-chain** | Agent installs/uses malicious package |
| **Insider misuse** | Engineer uses AI to bypass review |
| **Data exfiltration** | Agent sends source/secrets to external tools |
| **Hallucinated APIs** | Agent invents a function, breaks at runtime |

---

### 🔐 Controls

- **Least-privilege tools.** Read-only by default; write/exec require explicit grant.
- **Sandboxed execution.** Network egress allow-list; no host access.
- **Secrets out of context.** Pull at runtime via a secrets broker, never inline.
- **PR-only side effects.** Agents never push to main; humans merge.
- **SCA + SBOM.** Scan everything the agent installs.
- **Prompt-injection detectors** on any text from issues, PRs, docs, or web.
- **Audit trail.** Every agent action is logged with principal, tool, input, output.

---

### 🧪 Secure-by-default agent example

- Runs in a container with no network and a writable scratch dir.
- Has tools `read_file`, `apply_patch`, `run_tests`, `open_pr`.
- Cannot run arbitrary shell. Cannot reach external networks.
- Loads secrets only via `secrets://` URIs resolved at call time.
- All actions captured in a structured audit log.

---

### 🧱 Secure code generation

- Run **SAST**, **dependency scanning**, and **license checks** on every AI patch.
- Require **tests for new branches/conditions** in AI-authored code.
- Enforce **code-owner review** on security-sensitive paths regardless of AI authorship.
- Flag **AI-authored** commits in metadata so postmortems can analyze trends.

> **Principle.** *"AI changes who writes the code. It does not change what makes code safe to ship."*

---

## 7.4 AI DevOps and Deployment

> The same agent patterns that build code can **operate** it — if you wire them into your delivery and observability stack.

---

### 🚀 Release engineering with AI

- **Release-note drafting** from merged PRs.
- **Risk scoring** of a release (size, areas touched, test coverage).
- **Rollout planning** (canary %, bake time) suggested per change.
- **Automated rollback proposals** on SLO breach.

---

### 📈 Observability + AI

```
   metrics ─┐
   logs    ─┼─▶  AI triage agent  ─▶  candidate causes  ─▶  on-call review
   traces  ─┘                                  │
                                               ▼
                                       runbook execution
                                       (with approval)
```

---

### 🧩 SRE / on-call augmentation

- **Alert summarization** with linked dashboards and likely owners.
- **Log clustering** to surface the dominant error pattern.
- **Trace narration** — "what happened in this user's request".
- **Runbook execution** — agent runs documented steps under approval.
- **Postmortem drafting** from the incident timeline.

---

### 🧱 Deployment guardrails

- **Progressive delivery** (canary, blue/green) is mandatory for AI-authored changes.
- **SLO-based gates** — promotion blocked on error/latency regressions.
- **Feature flags** so AI-authored behaviour can be killed without rollback.
- **Cost guardrails** — alert on unusual AI spend per service.

---

### 🧪 Example — incident triage agent

```
1. PagerDuty alert fires
2. Agent pulls last 30 min of metrics, top error logs, recent deploys
3. Agent posts a Slack summary: "p95 up 3x; correlates with deploy 2034 to billing-svc"
4. Agent suggests: rollback billing-svc to 2033
5. On-call clicks "approve" → agent runs the rollback runbook
6. Agent monitors recovery, posts confirmation, drafts postmortem skeleton
```

---

### 📊 Operational metrics

- **MTTD / MTTR** with vs without AI triage.
- **Change failure rate** for AI-authored vs human-authored changes.
- **Deployment frequency** — does AI accelerate or destabilize?
- **On-call load** — pages per engineer, time-to-first-action.

---

## 🎯 Module 7 — Takeaways

1. **AI amplifies engineers; specs and tests amplify AI.**
2. **Pick the right level on the assist ladder** — most teams live at L1–L3, with L4 for chores.
3. **Wire AI across the SDLC**, not just into the IDE.
4. **Treat AI agents as privileged automation.** Least privilege, sandboxes, audit.
5. **PR-only side effects** keep humans in the merge loop.
6. **Progressive delivery + SLO gates** are mandatory for AI-authored changes.
7. **Measure outcomes, not suggestion volume** — cycle time, defect escape, MTTR.
8. **AI changes who writes code; it does not change what makes code safe to ship.**

> ⬅️ **Previous:** [Module 6 — RAG and Enterprise Knowledge Agents](Module_6_RAG_and_Enterprise_Knowledge_Agents.md)
> ➡️ **Next:** [Module 8 — Production Agent Systems](Module_8_Production_Agent_Systems.md)
