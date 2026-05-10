# Agentic RAG with LangGraph

> Part 2 of the RAG Series — building a self-correcting, adaptive RAG pipeline that knows when it's wrong and fixes itself.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/agentic-rag-langgraph/blob/main/notebooks/Agentic_RAG_LangGraph_Colab.ipynb)
![Python](https://img.shields.io/badge/Python-3.10+-blue)
![LangGraph](https://img.shields.io/badge/LangGraph-0.2+-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## What is Agentic RAG?

Traditional RAG is a straight line:

```
Query → Retrieve → Generate
```

It retrieves documents and generates an answer — no matter how bad the documents are, no matter if the answer is hallucinated.

**Agentic RAG** turns that into a loop:

```
Query → Route → Retrieve → Grade Docs → [Rewrite?] → Generate → Verify → Done
```

The system actively checks its own work at each step and retries until the answer is actually good.

---

## Project Structure

```
agentic-rag-langgraph/
├── notebooks/
│   └── Agentic_RAG_LangGraph_Colab.ipynb   # Main notebook (runs in Colab)
├── .env.example                              # API key template
├── .gitignore
├── requirements.txt
└── README.md
```

---

## Agent Architecture

```
              ┌─────────┐
              │  START  │
              └────┬────┘
                   │
                   ▼
          ┌─────────────────┐
          │   Query Router  │
          └────────┬────────┘
         AI topic? │  Other topic?
         ┌─────────┘  └──────────────┐
         ▼                           ▼
   ┌───────────┐               ┌────────────┐
   │  Retrieve │               │ Web Search │
   └─────┬─────┘               └──────┬─────┘
         │                            │
         ▼                            │
  ┌─────────────┐                     │
  │ Grade Docs  │                     │
  └──────┬──────┘                     │
  OK? ───┤ Not relevant?              │
         │       └──► Rewrite Query ──┘
         ▼               (max 2 retries, then web search)
   ┌──────────┐
   │ Generate │◄───────────────────────┘
   └─────┬────┘
         │
         ▼
  ┌───────────────────────┐
  │ Hallucination Check   │
  │ + Answer Quality Check│
  └──────────┬────────────┘
     Good?   │   Bad?
     ▼       │    └──► Rewrite & retry
  ┌─────┐    │
  │ END │    │
  └─────┘    │
```

---

## Components

| Node | What it does |
|------|-------------|
| **Query Router** | Routes to vectorstore for AI/ML topics; Tavily web search for everything else |
| **Retrieve** | Fetches top-4 chunks from Chroma vectorstore |
| **Grade Documents** | Binary relevance check — filters out irrelevant chunks |
| **Query Rewriter** | Rewrites vague queries for better retrieval (up to 2 retries) |
| **Web Search** | Tavily fallback when vectorstore can't answer |
| **Generate** | Synthesizes a concise answer from filtered context |
| **Hallucination Checker** | Verifies the answer is grounded in the source documents |
| **Answer Grader** | Confirms the answer actually addresses the original question |

---

## Tech Stack

| Tool | Role |
|------|------|
| [LangGraph](https://github.com/langchain-ai/langgraph) | Stateful agent graph with conditional edges and cycles |
| [LangChain](https://github.com/langchain-ai/langchain) | Chains, prompts, loaders |
| [OpenAI GPT-4o-mini](https://platform.openai.com/docs/models) | LLM powering all reasoning steps |
| [Chroma](https://www.trychroma.com/) | Local vector store |
| [Tavily](https://tavily.com/) | Real-time web search |

---

## Quickstart

### Option 1 — Google Colab (recommended)

Click the badge at the top. Add `OPENAI_API_KEY` and `TAVILY_API_KEY` to Colab Secrets and run all cells.

### Option 2 — Local

```bash
git clone https://github.com/YOUR_USERNAME/agentic-rag-langgraph.git
cd agentic-rag-langgraph

pip install -r requirements.txt

cp .env.example .env
# Edit .env and add your API keys

jupyter notebook notebooks/Agentic_RAG_LangGraph_Colab.ipynb
```

---

## Example Outputs

**AI topic → vectorstore path:**
```
❓ What are the main components of an AI agent?
🚦 ROUTING  → vectorstore
🔍 RETRIEVE → 4 chunks fetched
📊 GRADE    → 3 relevant, 1 filtered
💡 GENERATE → answer created
🔎 CHECK    → ✅ grounded, ✅ addresses question
Answer: Planning, memory, and tool use are the three core components...
```

**Current events → web search path:**
```
❓ What are the latest developments in AI regulation?
🚦 ROUTING  → web_search
🌐 SEARCH   → 3 results fetched
💡 GENERATE → answer created
🔎 CHECK    → ✅ grounded, ✅ addresses question
Answer: Biden's Executive Order, new employer duties for bias audits...
```

---

## Knowledge Base

The default vectorstore is built from three [Lilian Weng](https://lilianweng.github.io/) blog posts:

- [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)
- [Prompt Engineering](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)
- [Adversarial Tic-Tac-Toe](https://lilianweng.github.io/posts/2023-10-25-adv-tic-tac-toe/)

Swap in your own documents by editing the `urls` list in Section 2 of the notebook.

---

## RAG Series

| Part | Topic | Status |
|------|-------|--------|
| Part 1 | Basic RAG with LangChain | ✅ Published |
| **Part 2** | **Agentic RAG with LangGraph** | ✅ **This repo** |
| Part 3 | Coming soon | 🔜 |

---

## License

MIT
