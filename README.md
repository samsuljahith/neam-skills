# Neam Skills

Claude Code skills for the [Neam programming language](https://github.com/neam-lang/Neam) — the compiled DSL for building AI agent systems.

---

## What is Neam?

Neam is a compiled domain-specific language for building AI agent systems. It provides first-class syntax for:

- **Agents** — wrap LLM providers with system prompts and call them via `.ask()`
- **Knowledge Bases** — connect agents to documents via RAG with 7 retrieval strategies
- **Skills & Tools** — extend agents with callable functions
- **Guards & Guardchains** — runtime validation pipelines for security
- **Budgets** — enforce cost, time, and token limits
- **Cloud Deployment** — compile to Docker, Kubernetes, Lambda, Cloud Run, and more

```neam
agent Assistant {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "You are a helpful assistant."
}

let answer = Assistant.ask("What is the capital of France?");
emit answer;
```

---

## Skills

| Skill | Description |
|-------|-------------|
| [neam-patterns](./skills/neam-patterns/SKILL.md) | Core Neam syntax, idioms, variables, functions, built-ins, and project structure |
| [neam-agents](./skills/neam-agents/SKILL.md) | Agent design — single agents, multi-agent pipelines, subagents, memory, planning |
| [neam-rag](./skills/neam-rag/SKILL.md) | RAG patterns — knowledge bases, retrieval strategies, chunk tuning, grounded agents |
| [neam-testing](./skills/neam-testing/SKILL.md) | Testing — `test` blocks, assertions, test organization, integration tests |
| [neam-security](./skills/neam-security/SKILL.md) | Security — guards, capabilities, guardchains, prompt injection defense |
| [neam-deployment](./skills/neam-deployment/SKILL.md) | Deployment — Docker, Kubernetes, Lambda, Cloud Run, Helm, Terraform |
| [neam-finops](./skills/neam-finops/SKILL.md) | FinOps — budgets, cost attribution, dashboard, model selection, benchmarking |

---

## Installation

### Manual

Copy the skill files into your Claude Code skills directory:

```bash
# Clone the repo
git clone https://github.com/samsuljahith/neam-skills.git
cd neam-skills

# Copy all skills
cp -r skills/* ~/.claude/skills/
```

### Install a Single Skill

```bash
cp -r skills/neam-patterns ~/.claude/skills/
```

---

## Usage

Once installed, reference a skill in your `CLAUDE.md` or activate it directly in Claude Code:

```markdown
# CLAUDE.md
Use the neam-patterns skill when writing Neam code.
Use the neam-agents skill when designing agent architectures.
Use the neam-rag skill when building knowledge-connected agents.
Use the neam-security skill when adding guards or capabilities.
Use the neam-deployment skill when deploying Neam agents.
Use the neam-finops skill when configuring budgets or cost controls.
```

---

## Neam Quick Reference

```bash
# Build from source
git clone https://github.com/neam-lang/Neam.git
cd Neam && mkdir -p build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
cmake --build . --parallel

# Compile and run
neamc hello.neam -o hello.neamb
neam-cli hello.neamb

# REPL
neam-cli

# Deploy
neam deploy --target kubernetes --replicas 3
neam deploy --target aws-lambda --memory 512 --arch arm64
neam deploy --target docker
```

---

## Skills Map

```
Neam Skills
├── neam-patterns      ← Start here. Language fundamentals.
├── neam-agents        ← Designing agent systems.
│   ├── neam-rag       ← Adding knowledge / RAG.
│   └── neam-security  ← Adding guards and access control.
├── neam-testing       ← Writing tests for Neam code.
├── neam-deployment    ← Shipping to cloud.
└── neam-finops        ← Controlling cost.
```

---

## Contributing

1. Fork the repo
2. Add or improve a skill in `skills/<skill-name>/SKILL.md`
3. Follow the frontmatter format: `name`, `description`, `origin: neam-skills`
4. Open a pull request

---

## License

MIT
