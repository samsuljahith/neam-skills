---
name: neam-finops
description: Neam FinOps patterns — budget declarations, cost attribution, real-time monitoring, alert configuration, and cost optimization strategies for AI agent systems.
origin: neam-skills
---

# Neam FinOps Patterns

Patterns for managing, monitoring, and optimizing costs in Neam agent systems using the built-in `budget` primitive, FinOps dashboard, and continuous benchmarking.

## When to Activate

- Adding cost controls to Neam agents
- Setting up the FinOps monitoring dashboard
- Designing budget hierarchies for teams or projects
- Optimizing LLM provider selection for cost
- Running continuous benchmarks to detect cost regressions

---

## The `budget` Primitive

Every production agent should have a `budget`. The budget enforces hard limits on time, cost, and tokens per run.

```neam
budget MyBudget {
  time: 5000,       // max milliseconds per agent call
  cost: 0.10,       // max dollars per agent call
  tokens: 5000      // max tokens per agent call
}

agent CheapBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Be brief and efficient.",
  budget: MyBudget
}
```

**Built-in alerts (automatic):**
- Warning at 80% of budget consumed
- Critical at 95% of budget consumed

---

## Pattern 1: Per-Call Micro Budget

For serverless or event-driven agents, cap each invocation tightly.

```neam
budget ServerlessBudget {
  time: 10000,    // 10 seconds max
  cost: 0.01,     // $0.01 max per call
  tokens: 2000    // 2K tokens max
}

agent EventProcessor {
  provider: "bedrock",
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",
  system: "Process events concisely. Return structured JSON.",
  budget: ServerlessBudget
}

let event = input();
let result = EventProcessor.ask("Process: " + event);
emit result;
```

---

## Pattern 2: Daily Team Budget

Set a shared daily budget across a team's agent workloads.

```neam
budget TeamBudget {
  cost: 500.00,          // $500/day max
  tokens: 10000000       // 10M tokens/day max
}

// All agents sharing this budget draw from the same pool
agent ResearchAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Research thoroughly.",
  budget: TeamBudget
}

agent SummaryAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Summarize research notes.",
  budget: TeamBudget
}
```

---

## Pattern 3: Tiered Budget Hierarchy

Use different budgets for different criticality tiers.

```neam
// Tier 1: Critical agents — generous budget
budget CriticalBudget {
  cost: 50.00,
  tokens: 1000000,
  time: 60000
}

// Tier 2: Standard agents — moderate budget
budget StandardBudget {
  cost: 5.00,
  tokens: 100000,
  time: 30000
}

// Tier 3: Background/batch agents — minimal budget
budget BatchBudget {
  cost: 0.50,
  tokens: 10000,
  time: 10000
}

agent CriticalDecider {
  provider: "openai",
  model: "gpt-4o",
  system: "Make critical business decisions with full analysis.",
  budget: CriticalBudget
}

agent StandardWorker {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Process standard requests efficiently.",
  budget: StandardBudget
}

agent BatchProcessor {
  provider: "ollama",             // Free local model for batch
  model: "qwen2.5:14b",
  system: "Process batch items.",
  budget: BatchBudget
}
```

---

## Pattern 4: FinOps Dashboard

Enable real-time cost monitoring with the built-in dashboard.

```neam
env Monitored {
  FINOPS_DASHBOARD: "true",
  FINOPS_PORT: "8080",
  ALERT_WEBHOOK: env("SLACK_WEBHOOK")    // Slack alerts
}

budget TeamBudget {
  cost: 500.00,
  tokens: 10000000
}

agent MonitoredAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Production agent with full cost tracking.",
  budget: TeamBudget,
  env: Monitored
}
```

Access the dashboard at `http://localhost:8080`.

**Dashboard widgets (14 total):**
- Cost over time (line chart)
- Cost by category (compute, LLM API, embedding, vector DB, etc.)
- Cost by agent and provider
- Budget gauges (% consumed, alerts)
- Top spenders
- Latency and throughput trends
- CPU / memory / GPU utilization
- Error rate
- Alerts feed
- Savings opportunities

**Features:** WebSocket live updates, time range presets (1h → 30d), JSON/PDF export, dark mode, embeddable widgets.

---

## Pattern 5: Cost Attribution

Neam tracks costs across 10 categories in real time:

| Category | Description |
|----------|-------------|
| Compute | CPU/memory for running agents |
| LLM API | OpenAI, Bedrock, etc. API calls |
| Embedding API | Embedding model calls |
| Vector DB | usearch query and storage costs |
| Memory | Redis or other memory backend |
| Storage | File storage I/O |
| Network | Egress / ingress costs |
| State backend | World model state updates |
| External APIs | Connector HTTP calls |
| Miscellaneous | Other uncategorized costs |

---

## Pattern 6: Cost-Aware Model Selection

Choose models based on cost-quality tradeoff per task type.

```neam
// Expensive: only for tasks that need it
budget PlannerBudget { cost: 10.00, tokens: 200000 }
agent Planner {
  provider: "bedrock",
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",  // frontier for planning
  system: "Decompose complex tasks into step-by-step plans.",
  budget: PlannerBudget
}

// Cheap: for high-volume execution
budget WorkerBudget { cost: 0.50, tokens: 20000 }
agent Worker {
  provider: "openai",
  model: "gpt-4o-mini",    // mini for execution
  system: "Execute tasks from the plan. Be efficient.",
  budget: WorkerBudget
}

// Free: for summaries and low-stakes tasks
agent Summarizer {
  provider: "ollama",
  model: "qwen2.5:14b",    // local = $0
  system: "Summarize findings concisely."
}

// Pipeline: expensive planning, cheap execution, free summary
let plan    = Planner.ask("Plan this: " + goal);
let result  = Worker.ask("Execute: " + plan);
let summary = Summarizer.ask("Summarize: " + result);
emit summary;
```

---

## Pattern 7: Continuous Benchmarking

Neam includes automated benchmarking with regression detection for CI/CD.

```bash
# Run benchmarks (outputs JUnit XML + GitHub Actions annotations)
neam benchmark --suite full

# Regression thresholds (automatic):
# - Latency increase > 10%  → FAIL
# - Cost increase > 5%      → FAIL
# - Throughput decrease > 10% → FAIL
```

Integrate into CI:

```yaml
# .github/workflows/benchmark.yml
- name: Run Neam benchmarks
  run: |
    neamc src/main.neam -o build/main.neamb
    neam benchmark --suite full --output junit.xml

- name: Upload results
  uses: actions/upload-artifact@v3
  with:
    name: benchmark-results
    path: junit.xml
```

---

## Pattern 8: Full Production FinOps Setup

```neam
budget TeamBudget {
  cost: 500.00,
  tokens: 10000000
}

env Monitored {
  FINOPS_DASHBOARD: "true",
  FINOPS_PORT: "8080",
  ALERT_WEBHOOK: env("SLACK_WEBHOOK")
}

knowledge InternalDocs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [{ type: "file", path: "./docs/" }],
  retrieval_strategy: "mmr",
  top_k: 5,
  mmr_lambda: 0.7
}

agent ProductionAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Production agent with full cost tracking and knowledge base.",
  connected_knowledge: [InternalDocs],
  budget: TeamBudget,
  env: Monitored
}

let answer = ProductionAgent.ask(input());
emit answer;
```

---

## Cost Optimization Strategies

| Strategy | Savings |
|----------|---------|
| Use `gpt-4o-mini` instead of `gpt-4o` for execution | ~10x cheaper per token |
| Use Ollama (local) for summaries and low-stakes tasks | 100% LLM cost savings |
| Use AWS Bedrock spot instances | 60–90% compute savings |
| Use ARM64 for Lambda | ~20% cheaper + faster |
| Set tight `tokens` budgets to fail fast | Prevents runaway costs |
| Use `"mmr"` retrieval to reduce top_k needed | Fewer tokens per query |
| Cache frequent agent responses | Eliminates repeat API calls |

---

## Budget Quick Reference

| Scenario | Budget Suggestion |
|----------|-----------------|
| Serverless / event-driven | `time: 10000, cost: 0.01, tokens: 2000` |
| Standard interactive agent | `cost: 1.00, tokens: 20000` |
| Deep research pipeline | `cost: 25.00, tokens: 500000` |
| Team daily limit | `cost: 500.00, tokens: 10000000` |

---

## Anti-Patterns

```neam
// Bad: No budget in production — unlimited cost exposure
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o"
}

// Good: Always cap cost and tokens
budget ProdBudget { cost: 10.00, tokens: 200000 }
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o",
  budget: ProdBudget
}
```

```neam
// Bad: Using gpt-4o for everything — 10x more expensive than needed
agent BatchSummarizer {
  provider: "openai",
  model: "gpt-4o",    // overkill for summarization
  system: "Summarize this text in 2 sentences."
}

// Good: Right-size the model to the task
agent BatchSummarizer {
  provider: "ollama",
  model: "qwen2.5:14b",   // free local model is sufficient
  system: "Summarize this text in 2 sentences."
}
```

__Remember__: Every production agent needs a `budget`. Use the FinOps dashboard to watch costs in real time. Run continuous benchmarks to catch cost regressions before they hit production. The cheapest viable model for each task is always the right choice.
