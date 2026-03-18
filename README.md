> **80-97% cost reduction in AI agent operations.** TLCI is a design philosophy that routes every operation to the cheapest execution tier that can handle it — code for deterministic work, SLMs for classification, frontier LLMs only for judgment. The result: predictable costs, reusable code, and architectures that get cheaper as they scale.
>
> **Reference implementation:** [Career Coach](https://github.com/cgorricho/career-coach) — 19 agents, 88 scripts, $0.16-0.55/day operational cost.

---

# Token-Light, Code-Intensive (TLCI)

**A design philosophy for AI agent automation platforms.**

---

## The Problem TLCI Solves

Every AI automation platform today treats the LLM as the execution engine. User request comes in, LLM processes it, LLM calls tools, LLM formats the response, LLM sends the output. Every step burns tokens. A 5-step workflow that fetches data, filters it, formats it, makes one judgment call, and sends a notification uses the LLM for all 5 steps — even though only step 4 requires intelligence.

The irony is that the LLM is using code to do it. When a frontier model "fetches data" or "formats output," it writes inline code — JavaScript, Python, shell commands — executes it, returns the result, and discards the code. The LLM is paying frontier-model token prices to generate throwaway code that performs deterministic operations. It spends tokens to write code, spends tokens to interpret the result, and then the code is gone. Next execution, it writes the same code again — or more precisely, it writes *some* code that probably does the same thing, since LLM inference is non-deterministic and there is no guarantee that tomorrow's throwaway script behaves identically to today's. You are paying the most expensive tier to do work that a static function could do for free — and then throwing away the work so you can pay for it again tomorrow.

The result: unpredictable costs, invisible spending, disposable code, and architectures that get more expensive as they scale. This is how 40-60% of AI operational costs become hidden from the organizations paying the bills.

TLCI inverts this. AI agents are used only where they are needed. Everything else is code — written once, kept, and reused at zero marginal cost.

---

## The Core Principle

**Token-light, code-intensive. AI agents are used only where they are needed.**

This is not an optimization technique applied after the system is built. It is the governing architecture. Every design decision — from how workflows execute, to how costs are tracked, to how plans are generated — flows from this principle.

---

## Three-Tier Execution Model

Every workflow step is classified into the cheapest tier that can handle it:

### Tier 1: Code Tools ($0)

Deterministic operations that require no intelligence:
- Data fetching (API calls, database queries, file reads)
- Data transformation (filtering, mapping, aggregation, formatting)
- Actions (sending messages, writing files, triggering webhooks)
- Scheduling (cron-based triggers, event-based triggers)

These operations are pure code. They execute instantly, cost nothing in tokens, and produce deterministic results. In a typical workflow, 60-80% of steps are code tools.

### Tier 2: Small Language Models (pennies)

Lightweight inference for tasks that need pattern recognition but not reasoning:
- Classification (routing, categorization, sentiment detection)
- Simple decisions (threshold evaluation, rule application with fuzzy input)
- Extraction (entity recognition, structured data from unstructured input)

SLMs run locally via Ollama, llama.cpp, or ONNX Runtime. Cost per call is fractions of a cent. In a typical workflow, 10-25% of steps are SLM tasks.

### Tier 3: Frontier LLMs (real cost)

Full-capability models for tasks that require genuine reasoning:
- Summarization and synthesis across multiple sources
- Natural language composition (reports, recommendations, analysis)
- Complex planning and decomposition
- Creative generation
- Multi-step reasoning with ambiguity

These are API calls to Claude, GPT-4o, Gemini, or equivalent. They are the expensive tier — and the only tier where real token cost is incurred. In a typical workflow, 5-15% of steps require frontier reasoning.

### The Math

A workflow with 8 steps where 6 are code, 1 is SLM, and 1 is frontier costs ~$0.02-0.05. The same workflow on an all-LLM platform costs ~$0.50-1.00. That is an 80-97% cost reduction — not from cheaper models, but from not using models where they are not needed.

---

## The Planning Layer

TLCI requires a decision-maker that classifies steps before execution. This is the Planning Layer — the component that makes TLCI operational rather than theoretical.

### What the Planning Layer Does

1. **Decomposes intent** — Breaks the user's natural language request into discrete, numbered execution steps
2. **Classifies each step** — Assigns each step to code tool, SLM, or frontier LLM
3. **Decides execution strategy** — Single-agent (simple, linear) or multi-agent (complex, parallel) based on cost/benefit analysis
4. **Estimates total cost** — Calculates expected token usage and dollar cost across all tiers
5. **Presents the plan** — Shows the user what will happen, how much it will cost, and asks for approval before a single token is spent

### Plan Caching

The plan IS the cached skill. First execution: plan + execute. Subsequent executions: execute from the cached plan. A $0.05 planning call that saves $0.50 per execution across 100 runs saves $49.95. The planning cost amortizes to zero over time.

### Multi-Agent as Cost Optimization

Multi-agent is not a product identity — it is an execution strategy chosen by the planning layer when the cost/benefit analysis demands it:

| Workflow Complexity | Strategy | Why |
|---|---|---|
| Simple, linear | Single agent, mostly code | Overhead of coordination exceeds benefit |
| Moderate, branching | Single agent, mixed tiers | One agent can handle branching logic |
| Complex, parallel | Multi-agent, specialized | Parallel execution saves time; specialized agents are more efficient |
| Research-heavy | Multi-agent, iterative | Multiple perspectives produce better synthesis |

The decision is automatic and transparent. The user sees the plan; the platform executes the strategy.

---

## Metering as Architecture

TLCI is only credible if every execution is metered. If you claim 80-97% cost reduction, you must prove it — not with estimates, but with per-execution cost records.

### What Gets Metered

- Every LLM API call: model, tokens (input/output), cost, latency
- Every code tool execution: duration, status, resource usage
- Every SLM inference: model, tokens, cost
- Every workflow execution: total cost across all tiers, step-by-step breakdown

### What Does NOT Get Stored

- Raw prompt content
- Raw response content
- User data passing through the workflow

Execution records contain metadata only: tokens, cost, model, status, duration. This is a deliberate architectural constraint — cost visibility without content surveillance.

### Budget Enforcement

Metering enables enforcement. Three-level budgets (per-execution, per-skill, global) are checked BEFORE execution, not after. Pessimistic locking: if the budget check fails, execution does not start. The user never sees a surprise bill because the platform refuses to create one.

---

## Framework Integration Pattern

TLCI does not mean building everything from scratch. Existing frameworks provide excellent plumbing — but their agent loops are replaced by TLCI's own execution model.

### The Rule

Use framework plumbing. Never use framework brains.

| Framework | Use | How |
|---|---|---|
| LangChain | Tool library | Import tools, document loaders, connectors as code tools. Do not use the agent loop. |
| LlamaIndex | RAG adapter | Import indexing and retrieval pipelines as code tools. Best RAG framework available. |
| LangGraph | Study only | Learn state machine patterns for the router. Do not adopt the runtime. |
| CrewAI | Reference only | Study multi-agent orchestration patterns. Do not adopt as a dependency. |

Frameworks are optional `peerDependencies` behind a code tool interface. The core platform installs and runs without any framework dependency. A user who needs RAG adds the LlamaIndex adapter. A user who does not need RAG never downloads it.

---

## Why TLCI Matters Beyond Cost

### Predictability

All-LLM platforms produce variable costs because LLM inference is non-deterministic in both output and token usage. TLCI reduces the variable component to 5-15% of the workflow. The other 85-95% is deterministic code with fixed, known cost ($0).

### Debuggability

Code tool steps produce deterministic output. When a workflow fails, the failure is either in the code (fixable, reproducible) or in the LLM step (isolatable). You do not debug 8 chained LLM calls to find one hallucination.

### Speed

Code tools execute in milliseconds. SLM inference runs locally with sub-second latency. Only frontier LLM calls have API latency (1-5 seconds). A TLCI workflow is faster than an all-LLM workflow because most steps skip the API entirely.

### Auditability

Every step has a classification label. Every execution record shows which tier processed each step. Compliance teams can verify that sensitive data was processed by code tools (deterministic, auditable) rather than by external LLM APIs (opaque, third-party).

### Arbitrage

TLCI creates 85-95% gross margins on subscription plans. If a user's workflow costs $0.02-0.05 per execution but they pay a flat $25/month subscription, the platform operator captures the margin. All-LLM competitors cannot match this — their per-execution costs are 10-50x higher, making flat-rate subscriptions unprofitable.

---

## Applying TLCI to Your Project

### Step 1: Audit Your Workflow Steps

For each step in your AI workflow, ask: does this step require intelligence, or is it moving/transforming data?

- **Fetching data from an API?** Code tool. Zero tokens needed.
- **Filtering or sorting results?** Code tool.
- **Formatting output for display?** Code tool.
- **Classifying input into categories?** SLM — unless categories are ambiguous and require nuanced judgment.
- **Summarizing a document?** Frontier LLM.
- **Sending a notification?** Code tool.
- **Deciding whether to send a notification?** Depends on the decision complexity. Threshold-based: code. Pattern-based: SLM. Judgment-based: frontier.

### Step 2: Implement the Tiers

You do not need a planning layer on day one. Start with manual classification:

```
// Explicit tier routing
if (step.type === 'fetch' || step.type === 'transform' || step.type === 'send') {
  return executeCodeTool(step);
} else if (step.type === 'classify' || step.type === 'extract') {
  return executeSLM(step);
} else {
  return executeFrontierLLM(step);
}
```

Graduate to automated classification when your step catalog is large enough to warrant it.

### Step 3: Meter Everything

If you cannot measure per-execution cost per tier, you cannot prove TLCI works. Instrument from day one:

- Log every LLM call with model, tokens, and cost
- Log every code tool execution with duration
- Store metadata only — never raw prompts or responses
- Make cost data queryable — users should see their spending in real time

### Step 4: Enforce Budgets

Once you have metering, add budget enforcement:

- Check budget BEFORE execution, not after
- Fail closed — if the budget system is unavailable, do not execute
- Three granularity levels: per-execution, per-workflow, global

---

## Summary

TLCI is a governing philosophy, not an optimization technique. It starts from the premise that AI agents are expensive, non-deterministic, and opaque — and therefore should be used only where their capabilities are required. Everything else should be code: deterministic, free, fast, and auditable.

The three-tier model (code / SLM / frontier) provides the execution framework. The planning layer provides the classification intelligence. Metering provides the proof. Budget enforcement provides the control.

The result: platforms that are 80-97% cheaper, faster, more predictable, more debuggable, and more auditable than all-LLM alternatives — without sacrificing capability where intelligence is genuinely needed.

**AI agents are powerful. Use them only where they are needed.**

---

## Author

**Carlos Gorricho**
cgorricho@carlosgorrichoai.one
+1-470-513-9430
