# RAG 101

A hands-on curriculum for learning Retrieval-Augmented Generation, going from the basic pipeline all the way to self-corrective LangGraph systems. Every concept lives in a runnable notebook ("playbook"), with short inline notes explaining what each stage does and why.

## Stack

- **LLM & Embeddings**: Ollama (`gemma3:4b` + `nomic-embed-text`), fully local, no API keys needed. 
  - To use your own provider instead, Just swap the `ChatOllama` / `OllamaEmbeddings` calls for your provider's classes (e.g. `ChatOpenAI`, `HuggingFaceEmbeddings`) and set the model you like; everything else stays the same
- **Framework**: LangChain / LCEL, LangGraph for agentic control flows
- **Vector stores**: Chroma (persistent), plus in-memory stores
- **Observability**: LangSmith tracing on every notebook (`LANGSMITH_TRACING=true`)
- **Late interaction**: PyLate (ColBERT) with PLAID index
- **Web fallback**: Tavily

## Setup

```bash
# 1. Install Ollama and pull the models
ollama pull gemma3:4b
ollama pull nomic-embed-text

# 2. Install dependencies
uv sync   # or: pip install -e .

# 3. Create .env (never commit it) with your LangSmith + Tavily keys:
#    LANGSMITH_API_KEY=...
#    TAVILY_API_KEY=...
```

Each notebook's first cell turns on tracing via `dotenv` + env vars. API keys get picked up from `.env` automatically.

## Learning Path

The notebooks build on each other: document-side optimization → query-side optimization → orchestration → self-corrective graphs. Go in order within each group.

### Stage 0: Foundation

| Playbook | Concepts |
| --- | --- |
| `rag_basic.ipynb` | The full pipeline by hand: load → split → embed → store → retrieve → generate. VectorStore vs Retriever (Runnables), search variants, first RAG chain |

### Stage 1: Document-side Optimization *(what you embed)*

| Playbook | Concepts |
| --- | --- |
| `rag_chunking.ipynb` | Fixed-size vs structure-aware vs context-aware splitting, and the trade-offs of each |
| `rag_indexing.ipynb` | Multi-vector/proposition indexing (embed summaries, retrieve parents); ColBERT late interaction with PyLate/PLAID |

### Stage 2: Query-side Optimization *(what you search with)*

| Playbook | Concepts |
| --- | --- |
| `rag_query_translation.ipynb` | Multi-Query, RAG Fusion (RRF), Decomposition, Step-back prompting, HyDE |
| `rag_query_construction.ipynb` | Turning natural language into structured metadata filters via `with_structured_output`; zero-shot vs few-shot extraction against SQLite |

### Stage 3: Orchestration *(deciding where queries go)*

| Playbook | Concepts |
| --- | --- |
| `rag_routing.ipynb` | Logical routing (LLM + structured output) vs semantic routing (embedding similarity); dynamic persona prompting |

### Stage 4: Self-Corrective Graphs *(LangGraph)*

Same corpus and graders across all three, only the **control flow** changes. Read them together; the architecture IS the technique here.

| Playbook | Concepts |
| --- | --- |
| `rag_corrective_rag.ipynb` | CRAG: grade docs once, terminal web-search fallback. Linear, loop-free |
| `rag_self_rag.ipynb` | Prompted reflection tokens (ISREL / ISSUP / ISUSE): grade inputs AND your own output; cycles by design |
| `rag_adaptive_rag.ipynb` | The union: router at START + CRAG grading + generation graders + bounded retries with fallback |

## Source & Credits

This repo started as a follow-along of LangChain's RAG from scratch series, then grew into its own thing.

**Source material:**

- Video series: [RAG from scratch](https://www.youtube.com/watch?v=sVcwVQRHIc8) (LangChain, YouTube)
- Reference code: [langchain-ai/rag-from-scratch](https://github.com/langchain-ai/rag-from-scratch)

**What changed from the original:**

- Most examples are replicated from that material, not copied blind
- Everything reworked to run fully local on Ollama (the originals lean on OpenAI)
- Updated to the latest LangChain APIs; several things from the videos are deprecated or moved by now (e.g. RAGatouille → PyLate, deprecated built-in retrievers rewritten maunally)

## How to Use This Repo

1. **Run notebooks top to bottom**. The markdown cells explain each task inline (Problem → Idea → flow), so they work as revision notes too, no re-running needed.
2. **Watch the traces in LangSmith**. Every chain/graph is traced per project. Compare grader decisions across runs.
3. **Compare, don't just run**. Try the same question through multi-query vs HyDE, or CRAG vs Adaptive RAG paths.
4. **Notebooks reference each other** instead of repeating explanations. That's intentional; follow the links when a cell points you somewhere.
5. **Known sharp edges** (on purpose, good learning material):
   - Small local LLMs are inconsistent graders. See the retry/fallback ladder in `rag_adaptive_rag`
   - Self-RAG's graph can spin until LangGraph's recursion limit kicks in. The bounded-retry pattern fix is documented there
   - Re-running DB seed cells can duplicate rows