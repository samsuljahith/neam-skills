---
name: neam-rag
description: Neam RAG (Retrieval-Augmented Generation) patterns — knowledge base setup, retrieval strategies, embedding configuration, and grounded agent design.
origin: neam-skills
---

# Neam RAG Patterns

Patterns for building grounded, knowledge-connected agents using Neam's `knowledge` primitive and its 7 retrieval strategies.

## When to Activate

- Connecting agents to document sources
- Building document Q&A or support systems
- Designing knowledge bases with specific retrieval strategies
- Optimizing chunk size, overlap, and top-k for retrieval quality
- Building production RAG pipelines with Neam

---

## The `knowledge` Block

The `knowledge` primitive wires a vector store and embedding model to document sources. Connect it to an agent via `connected_knowledge`.

```neam
knowledge MyDocs {
  vector_store: "usearch",              // vector DB
  embedding_model: "nomic-embed-text",  // embedding model
  chunk_size: 200,                      // tokens per chunk
  chunk_overlap: 50,                    // overlap between chunks
  sources: [
    { type: "file", path: "./docs/guide.md" },
    { type: "file", path: "./docs/faq.md" }
  ]
}

agent DocAgent {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Answer questions strictly based on the provided documentation.",
  connected_knowledge: [MyDocs]
}

let answer = DocAgent.ask("How do I install the CLI?");
emit answer;
```

---

## Retrieval Strategies

Neam supports 7 retrieval strategies. Choose based on your quality and latency requirements.

| Strategy | Description | Best For |
|----------|-------------|----------|
| `"basic"` | Standard vector similarity search | Simple lookups, fast retrieval |
| `"mmr"` | Maximal Marginal Relevance — diversity-aware | Avoiding repetitive results |
| `"hybrid"` | Combined keyword + vector search | Mixed precise/semantic queries |
| `"hyde"` | Hypothetical Document Embeddings | Improving abstract query matching |
| `"self_rag"` | Self-reflective with relevance checks | High-accuracy fact retrieval |
| `"crag"` | Corrective RAG with query decomposition | Complex multi-part questions |
| `"agentic"` | Tool-based retrieval with planning | Deep research, iterative retrieval |

---

## Pattern 1: Basic RAG (Fast Baseline)

Start here. Simple vector similarity, lowest latency.

```neam
knowledge ProductDocs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [
    { type: "file", path: "./docs/product.md" },
    { type: "file", path: "./docs/api.md" }
  ]
}

agent SupportBot {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Answer customer questions using the product documentation. Be concise.",
  connected_knowledge: [ProductDocs]
}

let question = input();
let answer = SupportBot.ask(question);
emit answer;
```

---

## Pattern 2: Hybrid Retrieval (Keyword + Vector)

Best for mixed workloads — users sometimes ask with exact terms (keyword) and sometimes conceptually (semantic).

```neam
knowledge HybridKB {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 300,
  chunk_overlap: 75,
  sources: [
    { type: "file", path: "./data/knowledge.md" }
  ],
  retrieval_strategy: "hybrid",
  top_k: 8
}

agent HybridAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Answer questions using the knowledge base. Cite relevant sections.",
  connected_knowledge: [HybridKB]
}
```

---

## Pattern 3: MMR for Diverse Results

When you need diverse, non-redundant retrieved chunks. Tune `mmr_lambda`: closer to 1.0 = more relevance, closer to 0.0 = more diversity.

```neam
knowledge DiverseSearch {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [
    { type: "file", path: "./corpus/articles.md" }
  ],
  retrieval_strategy: "mmr",
  top_k: 6,
  mmr_lambda: 0.7    // 0.7 = balanced relevance + diversity
}

agent ResearchAgent {
  provider: "openai",
  model: "gpt-4o",
  system: "Synthesize diverse perspectives from the knowledge base.",
  connected_knowledge: [DiverseSearch]
}
```

---

## Pattern 4: Agentic Retrieval (Deep Research)

For complex questions requiring iterative, multi-step retrieval with planning and reflection. Higher latency, highest accuracy.

```neam
knowledge DeepCorpus {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [
    { type: "file", path: "./corpus/volume1.md" },
    { type: "file", path: "./corpus/volume2.md" },
    { type: "file", path: "./corpus/volume3.md" }
  ],
  retrieval_strategy: "agentic",
  top_k: 10,
  max_iterations: 5,
  enable_reflection: true
}

agent DeepResearcher {
  provider: "bedrock",
  model: "anthropic.claude-3-5-sonnet-20241022-v2:0",
  system: "Use all available knowledge to give thorough, accurate answers.",
  connected_knowledge: [DeepCorpus]
}

let answer = DeepResearcher.ask(input());
emit answer;
```

---

## Pattern 5: Multiple Knowledge Bases

Connect an agent to multiple knowledge bases for cross-domain Q&A.

```neam
knowledge LegalDocs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 300,
  chunk_overlap: 75,
  sources: [{ type: "file", path: "./legal/policies.md" }],
  retrieval_strategy: "hybrid",
  top_k: 5
}

knowledge TechDocs {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [{ type: "file", path: "./tech/architecture.md" }],
  retrieval_strategy: "basic",
  top_k: 5
}

agent CompanyBot {
  provider: "openai",
  model: "gpt-4o",
  system: "Answer questions using both legal and technical documentation.",
  connected_knowledge: [LegalDocs, TechDocs]   // multiple KBs
}
```

---

## Pattern 6: Production RAG with FinOps

Full production setup: MMR retrieval, budget enforcement, environment config.

```neam
budget RAGBudget {
  cost: 50.00,
  tokens: 1000000
}

env Production {
  API_URL: "https://api.prod.com",
  DEBUG: "false",
  API_KEY: env("PROD_API_KEY")
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

agent ProductionRAG {
  provider: "openai",
  model: "gpt-4o-mini",
  system: "Answer questions based on the internal documentation. Be accurate and cite sources.",
  connected_knowledge: [InternalDocs],
  budget: RAGBudget,
  env: Production
}

let answer = ProductionRAG.ask(input());
emit answer;
```

---

## Chunk Size Guidelines

| Content Type | Chunk Size | Overlap |
|-------------|------------|---------|
| Short FAQs, Q&A pairs | 100–150 | 20–30 |
| General documentation | 200–300 | 50–75 |
| Long-form articles | 300–500 | 75–100 |
| Technical specs, code | 150–250 | 30–50 |

---

## Choosing top_k

| Scenario | top_k |
|----------|-------|
| Precise single-answer queries | 3–5 |
| General Q&A with context | 5–8 |
| Deep research, synthesis | 8–15 |
| Agentic iterative retrieval | 10+ |

---

## Anti-Patterns

```neam
// Bad: No retrieval strategy specified (defaults to basic — fine for simple use,
// but don't leave complex queries on basic strategy)
knowledge KB {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 1000,     // Too large — poor retrieval precision
  chunk_overlap: 0,     // No overlap — breaks context at boundaries
  sources: [{ type: "file", path: "./docs.md" }]
}

// Good: Tuned configuration for the content type
knowledge KB {
  vector_store: "usearch",
  embedding_model: "nomic-embed-text",
  chunk_size: 200,
  chunk_overlap: 50,
  sources: [{ type: "file", path: "./docs.md" }],
  retrieval_strategy: "hybrid",
  top_k: 5
}
```

```neam
// Bad: Agent with no knowledge for a Q&A use case
agent SupportBot {
  provider: "openai",
  model: "gpt-4o",
  system: "Answer questions about our product."
  // No knowledge — agent will hallucinate!
}

// Good: Always ground agents in documentation
agent SupportBot {
  provider: "openai",
  model: "gpt-4o",
  system: "Answer questions based on the product documentation.",
  connected_knowledge: [ProductDocs]
}
```

__Remember__: Start with `"basic"` retrieval, measure answer quality, then upgrade to `"hybrid"` or `"mmr"` if needed. Reserve `"agentic"` for deep research tasks where latency is acceptable. Always tune chunk size for your content type.
