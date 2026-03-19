---
name: neam-programming
description: Complete Neam language reference for Claude Code. Import this skill to write, debug, and understand Neam — the compiled AI agent programming language. Covers syntax, agents, RAG, skills, tools, guards, budgets, deployment, and built-in functions.
origin: neam-skills
---

# Neam Programming

Neam is a compiled domain-specific language for building AI agent systems. This skill gives Claude full knowledge of Neam syntax and patterns so it can help you write Neam programs from scratch.

## When to Activate

- User is writing a `.neam` file
- User asks how to build an agent in Neam
- User asks about Neam syntax, keywords, or built-in functions
- User wants to connect an agent to a knowledge base (RAG)
- User wants to add skills, guards, budgets, or deploy a Neam agent
- User is new to Neam and needs guidance

---

## Toolchain

```bash
# Build from source
git clone https://github.com/neam-lang/Neam.git
cd Neam && mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel

# Compile a .neam file to bytecode
neamc hello.neam -o hello.neamb

# Run the bytecode
neam-cli hello.neamb

# Interactive REPL
neam-cli
neam> 1 + 2
3
```

---

## Core Syntax

### Hello World

```neam
print("Hello, Neam!");
```

### Variables

```neam
let name = "Alice";        // mutable variable
const MAX = 100;           // immutable constant
let count = 0;
let active = true;
let scores = [10, 20, 30];
```

### Functions

```neam
fun greet(name) {
  return "Hello, " + name + "!";
}

fun add(a, b) {
  return a + b;
}

emit greet("World");   // Hello, World!
```

### Control Flow

```neam
// if / else if / else
if (score > 90) {
  emit "Excellent";
} else if (score > 70) {
  emit "Good";
} else {
  emit "Keep going";
}

// while loop
let i = 0;
while (i < 5) {
  print(i);
  i = i + 1;
}

// for-in loop (use this for lists)
for (item in ["apple", "banana", "cherry"]) {
  print(item);
}
```

### Output

```neam
print("debug message");   // debug / logging
emit result;              // final program output — use this for results
```

---

## Agents

Agents are the core of every Neam program. They wrap an LLM with a system prompt.

### Minimal Agent

```neam
agent Assistant {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a helpful assistant."
}

let answer = Assistant.ask("What is the capital of France?");
emit answer;
```

### Supported Providers

| Provider | Value | Auth |
|----------|-------|------|
| OpenAI | `"openai"` | `OPENAI_API_KEY` env var |
| AWS Bedrock | `"bedrock"` | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` |
| Ollama (local, free) | `"ollama"` | None required |

### Full Agent Configuration

```neam
agent Analyst {
  provider: "openai",
  model: "gpt-4o",
  system: "You are a data analyst. Be precise.",
  temperature: 0.3,
  api_key_env: "OPENAI_API_KEY",
  skills: [Calculator, WebSearch],
  connected_knowledge: [CompanyDocs],
  guardchains: [SecurityChain],
  budget: AnalysisBudget,
  env: Production,
  memory: SessionMemory
}
```

### Multi-Agent Pipeline

```neam
agent Researcher {
  provider: "openai",
  model: "gpt-4o",
  system: "Research topics thoroughly. Return structured findings."
}

agent Writer {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Write clear articles from research notes."
}

agent Editor {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Edit for grammar and clarity."
}

let topic = input();
let research = Researcher.ask("Research: " + topic);
let draft    = Writer.ask("Write from: " + research);
let final    = Editor.ask("Edit: " + draft);
emit final;
```

### Local Agent (Ollama — free, private)

```neam
agent LocalBot {
  provider: "ollama",
  model: "qwen2.5:14b",
  system: "You are a coding assistant.",
  endpoint: "http://localhost:11434"
}
```

---

## Knowledge Bases (RAG)

Connect agents to documents for grounded, accurate answers.

```neam
knowledge ProductDocs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [
    { type: "file", path: "./docs/guide.md" },
    { type: "file", path: "./docs/faq.md" }
  ]
}

agent SupportBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Answer questions using the product documentation.",
  connected_knowledge: [ProductDocs]
}

let answer = SupportBot.ask(input());
emit answer;
```

### Retrieval Strategies

| Strategy | When to use |
|----------|-------------|
| `"basic"` | Simple lookups, fastest |
| `"hybrid"` | Mix of keyword + semantic search |
| `"mmr"` | Need diverse, non-repetitive results |
| `"hyde"` | Abstract or conceptual queries |
| `"self_rag"` | High-accuracy fact retrieval |
| `"crag"` | Complex multi-part questions |
| `"agentic"` | Deep research, iterative retrieval |

```neam
knowledge SmartKB {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [{ type: "file", path: "./data.md" }],
  retrieval_strategy: "hybrid",
  top_k: 5
}
```

---

## Skills

Skills are functions that agents can call as tools.

```neam
skill Calculator {
  description: "Performs arithmetic calculations",
  params: [
    { name: "expression", schema: { "type": "string", "description": "Math expression to evaluate" } }
  ],
  impl: fun(expression) {
    return eval_math(expression);
  }
}

skill WebSearch {
  description: "Search the web for information",
  params: [
    { name: "query", schema: { "type": "string", "description": "Search query" } }
  ],
  impl: fun(query) {
    return http_get("https://api.search.com?q=" + query);
  }
}

agent SmartBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Use your skills to help users.",
  skills: [Calculator, WebSearch]
}
```

---

## Tools

Tools are like skills but with explicit capability-based access control.

```neam
capability FileAccess {
  pattern: "file:./docs/*"
}

tool ReadFile {
  description: "Reads a file from the docs directory",
  capabilities: [FileAccess],
  params: [{ name: "path", type: String }],
  returns: String,
  impl: fun(path) {
    return file_read_string("./docs/" + path);
  }
}
```

---

## Guards and Security

Guards validate inputs and outputs. Chain them for layered security.

```neam
guard InputValidator {
  description: "Validates tool input size",
  handlers: [
    on_tool_input(input) -> Bool {
      if (string_length(input) == 0) { return false; }
      if (string_length(input) > 10000) { return false; }
      return true;
    }
  ]
}

guard OutputSanitizer {
  description: "Removes sensitive data from output",
  handlers: [
    on_tool_output(output) -> Bool {
      if (string_contains(output, "sk-")) { return false; }
      return true;
    }
  ]
}

guardchain SecurityChain {
  guards: [InputValidator, OutputSanitizer]
}

agent SafeAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Safe production agent.",
  guardchains: [SecurityChain]
}
```

---

## Budgets

Enforce cost, token, and time limits. Always use in production.

```neam
budget QuickTask {
  time: 5000,     // max milliseconds
  cost: 0.10,     // max dollars
  tokens: 5000    // max tokens
}

agent CheapBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Be brief and efficient.",
  budget: QuickTask
}
```

---

## Environment Configuration

```neam
env Production {
  API_URL: "https://api.prod.com",
  DEBUG: "false",
  API_KEY: env("PROD_API_KEY")    // always read secrets from env vars
}

agent ProdAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Production agent",
  env: Production
}
```

---

## Memory, World Model, Planning

```neam
memory ConversationMemory {
  backend: "redis",
  retention: "session",
  max_events: 10000
}

world_model TaskWorld {
  tier: 1,
  state_schema: "task_state_v1",
  update_frequency: 1000
}

plan HierarchicalPlanner {
  pattern: "hierarchical",
  max_depth: 5,
  backtrack: true
}

agent StrategicAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Think and plan strategically.",
  memory: ConversationMemory,
  world_model: TaskWorld,
  plan: HierarchicalPlanner
}
```

---

## Checkpoint and Rewind

Time-travel debugging for risky agent operations.

```neam
checkpoint "safe_point";

let result = risky_operation();
if (!result) {
  rewind "safe_point";
}

emit result;
```

---

## Module System

```neam
module my.app.agents;

import std.list;
import std.map;
import my.app.config as cfg;

pub fun public_helper() { }   // pub = exported
fun internal_helper() { }     // no pub = private
```

---

## Testing

```neam
test "addition works" {
  assert_eq(2 + 3, 5);
}

test "string concat" {
  assert_eq("Hello" + " " + "World", "Hello World");
}
```

| Assertion | Description |
|-----------|-------------|
| `assert_eq(a, b)` | Assert `a == b` |
| `assert_ne(a, b)` | Assert `a != b` |
| `assert_true(cond)` | Assert truthy |
| `assert_false(cond)` | Assert falsy |
| `assert_throws(fn)` | Assert throws |

---

## Built-in Functions

### Math
```neam
math_abs(-5)              // 5
math_floor(3.7)           // 3
math_ceil(3.2)            // 4
math_round(3.5)           // 4
math_min(5, 10)           // 5
math_max(5, 10)           // 10
math_clamp(15, 0, 10)     // 10
math_pow(2, 8)            // 256
math_sqrt(16)             // 4
math_random()             // 0.0 to 1.0
math_random_int(1, 100)   // 1 to 100
```

### String
```neam
string_upper("hello")              // "HELLO"
string_lower("HELLO")              // "hello"
string_length("hello")             // 5
string_trim("  hello  ")           // "hello"
string_slice("hello", 0, 3)        // "hel"
string_contains("hello", "ell")    // true
string_replace("hello", "l", "r")  // "herro"
string_split("a,b,c", ",")         // ["a", "b", "c"]
string_join(["a","b","c"], "-")    // "a-b-c"
```

### JSON
```neam
let obj = json_parse('{"name": "Alice"}');
let text = json_stringify(obj);
```

### File I/O
```neam
let content = file_read_string("./data.txt");
file_write_string("./output.txt", "Hello");
let exists = file_exists("./file.txt");
file_copy("./src.txt", "./dst.txt");
```

### HTTP
```neam
let resp = http_get("https://api.example.com/data");
let resp = http_request("POST", "https://api.example.com", body, headers);
```

### Crypto
```neam
crypto_hash("sha256", "data")
crypto_hmac("sha256", "secret", "message")
crypto_uuid_v4()
crypto_base64_encode("hello")
crypto_base64_decode("aGVsbG8=")
```

### Time
```neam
let now = time_now();
let formatted = time_format(now, "%Y-%m-%d");
time_sleep(1000);
```

### Async / Futures
```neam
let resolved = future_resolve(42);
let all = future_all([f1, f2, f3]);
let first = future_race([f1, f2]);
let delayed = future_delay(1000);
```

---

## Deployment

```bash
# Docker
neam deploy --target docker
docker build -t my-agent -f build/deploy/docker/Dockerfile .
docker run -e OPENAI_API_KEY=$OPENAI_API_KEY my-agent

# Kubernetes
neam deploy --target kubernetes --replicas 3 --min-replicas 1 --max-replicas 20

# AWS Lambda
neam deploy --target aws-lambda --memory 512 --timeout 15 --arch arm64

# GCP Cloud Run
neam deploy --target gcp-cloudrun --region us-central1

# Terraform
neam deploy --target terraform
```

---

## Common Patterns — Quick Reference

### Pattern: Simple Q&A Bot
```neam
agent QABot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Answer questions clearly and concisely."
}
emit QABot.ask(input());
```

### Pattern: RAG Document Bot
```neam
knowledge Docs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200, chunk_overlap: 50,
  sources: [{ type: "file", path: "./docs/" }]
}
agent DocBot {
  provider: "openai", model: "gpt-4o-mini",
  system: "Answer from the documentation only.",
  connected_knowledge: [Docs]
}
emit DocBot.ask(input());
```

### Pattern: Multi-Agent Pipeline
```neam
agent A { provider: "openai", model: "gpt-4o",      system: "Step 1 task." }
agent B { provider: "openai", model: "gpt-4o-mini", system: "Step 2 task." }
let out = A.ask(input());
emit B.ask(out);
```

### Pattern: Agent with Budget (Production)
```neam
budget B { cost: 1.00, tokens: 20000 }
agent SafeAgent { provider: "openai", model: "gpt-4o-mini", system: "...", budget: B }
emit SafeAgent.ask(input());
```

---

## Key Rules to Always Follow

1. **Never hardcode API keys** — always use `api_key_env: "ENV_VAR_NAME"`
2. **Always add a `budget`** to production agents
3. **Use `emit`** for output, `print` for debug
4. **Use `const`** for values that never change
5. **One agent, one job** — keep system prompts focused
6. **Add guards** for any agent that receives user input
