---
name: neam-agents
description: Neam agent design patterns — single agents, multi-agent pipelines, subagents, connectors, memory, world models, and planning.
origin: neam-skills
---

# Neam Agent Patterns

Patterns for designing, composing, and orchestrating agents in Neam — from simple single-agent programs to complex multi-agent pipelines.

## When to Activate

- Designing agent architectures in Neam
- Building multi-agent pipelines or orchestration workflows
- Configuring agent memory, planning, or world models
- Integrating subagents or external connectors
- Choosing LLM providers and models for agents

---

## Core Agent Anatomy

Every Neam agent has a provider, model, and system prompt at minimum. Additional fields enable advanced capabilities.

```neam
agent FullConfig {
  provider: "openai",           // LLM provider
  model: "gpt-4o",              // model name
  system: "You are an expert.", // system prompt
  temperature: 0.3,             // 0.0–1.0 (default 0.7)
  api_key_env: "OPENAI_API_KEY",// env var for API key

  // Advanced capabilities
  skills: [Calculator, WebSearch],
  connected_knowledge: [CompanyDocs],
  guardchains: [SafetyChain],
  budget: AnalysisBudget,
  env: Production,
  memory: SessionMemory,
  world_model: TaskWorld,
  plan: HierarchicalPlanner
}
```

### Supported Providers

| Provider | Value | Auth |
|----------|-------|------|
| Ollama (local) | `"ollama"` | None |
| OpenAI | `"openai"` | `OPENAI_API_KEY` |
| AWS Bedrock | `"bedrock"` | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` |

---

## Pattern 1: Simple Single Agent

Use for straightforward Q&A or task completion.

```neam
agent Assistant {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a helpful assistant. Answer questions clearly and concisely."
}

let question = input();
let answer = Assistant.ask(question);
emit answer;
```

---

## Pattern 2: Multi-Agent Pipeline

Chain agents where each output feeds the next. Keep each agent's role narrow and specialized.

```neam
agent Researcher {
  provider: "openai",
  model: "gpt-4o",
  system: "Research topics thoroughly. Return structured findings with sources."
}

agent Writer {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Transform research notes into clear, engaging articles."
}

agent Editor {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Edit for grammar, flow, and accuracy. Return the polished version only."
}

// Pipeline: research → draft → final
let topic = input();
let research = Researcher.ask("Research this topic: " + topic);
let draft    = Writer.ask("Write a 500-word article from: " + research);
let final    = Editor.ask("Edit this article: " + draft);
emit final;
```

---

## Pattern 3: Cost-Aware Multi-Cloud Pipeline

Use different providers for cost optimization — expensive frontier models for planning, cheaper/local models for execution.

```neam
budget ResearchBudget {
  cost: 25.00,
  tokens: 500000
}

// Expensive model for decomposition — pays off in quality
agent Planner {
  provider: "bedrock",
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",
  system: "Decompose the research topic into 5 focused sub-questions.",
  budget: ResearchBudget
}

// Mid-tier for deep research
agent Researcher {
  provider: "openai",
  model: "gpt-4o",
  system: "Research each sub-question. Provide detailed findings."
}

// Free local model for final synthesis
agent Synthesizer {
  provider: "ollama",
  model: "qwen2.5:14b",
  system: "Synthesize the findings into a coherent report."
}

let topic = "Impact of quantum computing on cryptography";
let plan     = Planner.ask("Create a research plan for: " + topic);
let findings = Researcher.ask("Research these questions:\n" + plan);
let report   = Synthesizer.ask("Write a report from:\n" + findings);
emit report;
```

---

## Pattern 4: Local LLM with Ollama

For privacy-sensitive or offline workloads, route to a local model.

```neam
agent LocalCoder {
  provider: "ollama",
  model: "qwen2.5:14b",
  system: "You are an expert coding assistant. Write clean, commented code.",
  endpoint: "http://localhost:11434"
}

let spec = file_read_string("./requirements.txt");
let code = LocalCoder.ask("Implement this specification:\n" + spec);
file_write_string("./solution.py", code);
emit "Code written to solution.py";
```

---

## Pattern 5: Agent with Memory

Persist conversation state across calls using the `memory` field.

```neam
memory ConversationMemory {
  backend: "redis",
  retention: "session",
  max_events: 10000
}

agent TutoringBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a patient tutor. Remember what the student has learned.",
  memory: ConversationMemory
}

// Memory persists across multiple .ask() calls
let answer1 = TutoringBot.ask("Explain recursion in Python.");
let answer2 = TutoringBot.ask("Give me a harder recursion problem.");
let answer3 = TutoringBot.ask("Why did my solution fail?");
emit answer3;
```

---

## Pattern 6: Agent with Planning

For complex multi-step tasks, attach a planner for hierarchical reasoning.

```neam
plan HierarchicalPlanner {
  pattern: "hierarchical",
  max_depth: 5,
  backtrack: true,
  pruning: "alpha_beta"
}

world_model TaskWorld {
  tier: 1,
  state_schema: "task_state_v1",
  update_frequency: 1000
}

agent StrategicAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Break down complex goals into executable plans. Track progress.",
  world_model: TaskWorld,
  plan: HierarchicalPlanner
}

let goal = "Build a customer support system with RAG and escalation.";
let execution_plan = StrategicAgent.ask("Create a plan for: " + goal);
emit execution_plan;
```

---

## Pattern 7: Subagents for Parallelism

Spawn subagents that inherit capabilities from a parent but have budget limits.

```neam
budget ParentBudget {
  cost: 100.00,
  tokens: 2000000
}

agent MainAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Orchestrate sub-tasks across workers.",
  budget: ParentBudget
}

subagent Worker {
  base_agent: "MainAgent",
  budget_share: 0.25,         // each worker gets 25% of parent budget
  capability_inherit: true    // inherits skills and knowledge
}
```

---

## Pattern 8: External Connectors

Connect agents to external services via the `connector` primitive.

```neam
connector SlackAPI {
  protocol: "http",
  endpoint: "https://slack.com/api",
  auth: env("SLACK_TOKEN")
}

connector GitHubAPI {
  protocol: "http",
  endpoint: "https://api.github.com",
  auth: env("GITHUB_TOKEN")
}

agent DevOpsBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Monitor GitHub and report issues to Slack."
}
```

---

## Pattern 9: Checkpoint and Rewind

Use time-travel debugging for risky agent workflows.

```neam
agent DataProcessor {
  provider: "openai",
  model: "gpt-4o",
  system: "Process and transform datasets."
}

checkpoint "before_transform";

let result = DataProcessor.ask("Transform this dataset: " + data);

if (!result) {
  rewind "before_transform";   // roll back and retry
}

emit result;
```

---

## Choosing Providers and Models

| Use Case | Recommended |
|----------|-------------|
| Complex reasoning, planning | `openai/gpt-4o`, `bedrock/claude-3-5-sonnet` |
| Fast, cheap Q&A | `openai/gpt-4o-mini` |
| Privacy / offline | `ollama/qwen2.5:14b` |
| Cost-controlled production | Mix: planner=frontier, executor=mini |
| Embedding | Use `knowledge` block with `nomic-embed-text` |

---

## Anti-Patterns

```neam
// Bad: Single agent doing research + writing + editing
agent Monolith {
  system: "Research, write, edit, and publish articles on any topic."
}

// Good: Separate concerns into specialized agents
agent Researcher { system: "Research only. Return structured notes." }
agent Writer     { system: "Write only. Take research notes as input." }
agent Editor     { system: "Edit only. Return polished final text." }
```

```neam
// Bad: No error handling for risky operations
let result = Agent.ask(complex_prompt);
emit result;

// Good: Use checkpoint/rewind for resilience
checkpoint "safe";
let result = Agent.ask(complex_prompt);
if (!result) { rewind "safe"; }
emit result;
```

__Remember__: Neam agents are most powerful when each has a single clear responsibility. Compose them like Unix pipes — small, focused, chainable. Always attach budgets in production to prevent runaway costs.
