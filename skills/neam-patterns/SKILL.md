---
name: neam-patterns
description: Core Neam language patterns, idioms, and best practices for building robust, expressive AI agent programs.
origin: neam-skills
---

# Neam Language Patterns

Idiomatic Neam patterns and best practices for building robust, efficient AI agent applications with the Neam compiled DSL.

## When to Activate

- Writing new `.neam` source files
- Reviewing or refactoring Neam code
- Designing Neam modules and program structure
- Building any Neam-based application

## How It Works

Neam is a compiled domain-specific language for AI agent systems. Source files (`.neam`) are compiled to bytecode (`.neamb`) by `neamc` and executed by `neam-cli`. The language provides first-class syntax for agents, knowledge bases, skills, tools, guards, budgets, environments, memory, and cloud deployment.

---

## Core Principles

### 1. Agents Are First-Class Citizens

Every Neam program centers on `agent` declarations. Define agents clearly with explicit provider, model, and system prompt.

```neam
// Good: Explicit, purpose-driven agent
agent Support {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a helpful customer support agent. Answer concisely."
}

let reply = Support.ask("How do I reset my password?");
emit reply;
```

```neam
// Bad: Vague system prompt, no clear purpose
agent Bot {
  provider: "openai",
  model: "gpt-4o",
  system: "Help."
}
```

### 2. Use `emit` for Program Output

`emit` is the idiomatic way to output results from a Neam program. Use `print` for debug/logging, `emit` for final output.

```neam
// Good: emit for result, print for debug
let result = Agent.ask(question);
print("Got response from agent");
emit result;

// Bad: print for final output
print(result);
```

### 3. Prefer `const` for Fixed Values

Use `const` for values that never change; use `let` for mutable bindings.

```neam
// Good
const MAX_RETRIES = 3;
const DEFAULT_MODEL = "gpt-4o-mini";

let attempts = 0;
let response = "";
```

### 4. Functions for Reusable Logic

Extract repeated logic into named functions.

```neam
fun build_prompt(topic, context) {
  return "Topic: " + topic + "\nContext: " + context + "\nAnswer:";
}

let prompt = build_prompt("quantum computing", research_notes);
let answer = Agent.ask(prompt);
```

### 5. Explicit Module Declarations

Always declare modules and use explicit imports for clarity.

```neam
module my.app.agents;

import std.list;
import std.map;
import my.app.config as cfg;

pub fun create_pipeline() { }
fun internal_helper() { }
```

---

## Variables and Control Flow

### Variables

```neam
let name = "World";           // mutable
const MAX = 100;              // immutable
let count = 0;
let active = true;
let items = [1, 2, 3];
```

### Conditionals

```neam
// if / else
if (score > 90) {
  emit "Excellent";
} else if (score > 70) {
  emit "Good";
} else {
  emit "Needs improvement";
}
```

### Loops

```neam
// while loop
let i = 0;
while (i < 10) {
  print(i);
  i = i + 1;
}

// for-in loop (idiomatic for collections)
for (item in ["apple", "banana", "cherry"]) {
  print(item);
}
```

### Functions

```neam
fun add(a, b) {
  return a + b;
}

fun greet(name) {
  return "Hello, " + name + "!";
}

emit greet("Neam");  // Hello, Neam!
```

---

## Data and Built-in Functions

### String Operations

```neam
let s = "Hello, World!";
let upper = string_upper(s);          // HELLO, WORLD!
let lower = string_lower(s);          // hello, world!
let length = string_length(s);        // 13
let part = string_slice(s, 0, 5);     // Hello
let found = string_contains(s, "World"); // true
let replaced = string_replace(s, "World", "Neam");
```

### Math

```neam
math_abs(-5);              // 5
math_pow(2, 10);           // 1024
math_sqrt(144);            // 12
math_clamp(150, 0, 100);   // 100
math_random_int(1, 6);     // dice roll
```

### JSON

```neam
let obj = json_parse('{"name": "Alice", "age": 30}');
let text = json_stringify(obj);
```

### File I/O

```neam
let content = file_read_string("./data.txt");
file_write_string("./output.txt", result);
let exists = file_exists("./config.json");
```

### HTTP

```neam
let response = http_get("https://api.example.com/data");
let post_resp = http_request("POST", "https://api.example.com/submit", body, headers);
```

### Time

```neam
let now = time_now();
let formatted = time_format(now, "%Y-%m-%d %H:%M:%S");
time_sleep(1000);    // sleep 1 second
```

---

## Project Structure

### Standard Layout

```
my-neam-project/
├── src/
│   ├── main.neam          # Entry point
│   ├── agents.neam        # Agent declarations
│   ├── knowledge.neam     # Knowledge base config
│   └── skills.neam        # Skill definitions
├── tests/
│   ├── test_agents.neam
│   └── test_skills.neam
├── docs/
│   └── guide.md
├── build/                 # Compiled output (neamb bytecode)
└── README.md
```

### Compiling and Running

```bash
# Compile source to bytecode
neamc src/main.neam -o build/main.neamb

# Run compiled bytecode
neam-cli build/main.neamb

# Interactive REPL for quick testing
neam-cli
neam> 1 + 2
3
```

---

## Anti-Patterns to Avoid

```neam
// Bad: Hardcoded secrets
agent Bot {
  provider: "openai",
  model: "gpt-4o",
  api_key_env: "sk-hardcoded-key-here"   // Never hardcode keys!
}

// Good: Use environment variables
agent Bot {
  provider: "openai",
  model: "gpt-4o",
  api_key_env: "OPENAI_API_KEY"
}
```

```neam
// Bad: One giant agent doing everything
agent DoEverything {
  system: "Research topics, write articles, edit grammar, summarize, translate, answer questions..."
}

// Good: Specialized agents with clear responsibilities
agent Researcher { system: "Research topics thoroughly. Cite sources." }
agent Writer     { system: "Write clear, engaging articles from research notes." }
agent Editor     { system: "Edit for grammar, clarity, and accuracy." }
```

```neam
// Bad: No budget on production agents — can run up unlimited costs
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o"
}

// Good: Always attach a budget in production
budget ProdBudget { cost: 10.00, tokens: 200000 }
agent ProdAgent {
  provider: "openai",
  model: "gpt-4o",
  budget: ProdBudget
}
```

---

## Quick Reference

| Construct | Syntax |
|-----------|--------|
| Variable | `let x = value;` |
| Constant | `const MAX = 100;` |
| Function | `fun name(args) { return val; }` |
| Agent | `agent Name { provider: "...", model: "...", system: "..." }` |
| Ask agent | `let r = Agent.ask("prompt");` |
| Output | `emit value;` |
| Debug | `print(value);` |
| Loop | `for (x in list) { }` / `while (cond) { }` |
| Module | `module my.app.name;` |
| Import | `import std.list;` |
| Test | `test "name" { assert_eq(a, b); }` |

__Remember__: Neam code should be declarative and agent-centric. Express your AI workflows as named, purposeful agents with clear system prompts, explicit budgets, and composable pipelines.
