# Agentic RAG with LangGraph

A self-correcting, adaptive Retrieval-Augmented Generation (RAG) system built with LangGraph that goes far beyond traditional linear RAG pipelines.

## Overview

Traditional RAG: `Query → Retrieve → Generate` (linear, one-shot)

**Agentic RAG**: `Query → Route → Retrieve → Grade → Rewrite? → Generate → Verify` (dynamic, self-correcting)

The system uses an LLM-powered agent graph that makes intelligent decisions at each step — filtering bad documents, rewriting poor queries, falling back to web search, and verifying the final answer for hallucinations before returning it.

## Architecture

```
         ┌───────────┐
         │   START   │
         └─────┬─────┘
               │
               ▼
       ┌───────────────┐
       │  Query Router │
       └──────┬────────┘
    AI topic? │  Other?
      ┌───────┘  └───────────┐
      ▼                      ▼
┌──────────┐           ┌────────────┐
│ Retrieve │           │ Web Search │
└────┬─────┘           └──────┬─────┘
     │                        │
     ▼                        │
┌────────────────┐            │
│  Grade Docs    │            │
└──────┬─────────┘            │
  OK?  │  Bad?                │
   ┌───┘  └──────────┐        │
   ▼                 ▼        │
Generate       Rewrite Query  │
   │                 │        │
   │    ┌────────────┘        │
   │    │                     │
   ◄────┘ ◄───────────────────┘
   │
   ▼
┌────────────────────┐
│  Check Hallucation │
│  & Answer Quality  │
└──────┬─────────────┘
  ✅   │   ❌
  ▼    │   └──► Retry
┌─────┐│
│ END ││
└─────┘│
```

## Components

| Component | Role |
|-----------|------|
| **Query Router** | Routes to vectorstore (AI/ML topics) or Tavily web search (current events, other topics) |
| **Document Grader** | Binary relevance scoring — filters out irrelevant chunks before generation |
| **Query Rewriter** | Rewrites vague or low-quality queries to improve retrieval (up to 2 retries) |
| **RAG Generator** | Synthesizes a concise answer from the retrieved context |
| **Hallucination Checker** | Verifies the answer is grounded in the retrieved documents |
| **Answer Grader** | Confirms the answer actually addresses the original question |

## Tech Stack

- **[LangGraph](https://github.com/langchain-ai/langgraph)** — stateful agent graph with conditional edges and cycles
- **[LangChain](https://github.com/langchain-ai/langchain)** — chains, prompts, document loaders
- **[OpenAI GPT-4o-mini](https://platform.openai.com/docs/models)** — LLM for all reasoning steps
- **[Chroma](https://www.trychroma.com/)** — local vector store with OpenAI embeddings
- **[Tavily](https://tavily.com/)** — real-time web search fallback

## Quick Start

### Run in Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YOUR_USERNAME/agentic-rag-langgraph/blob/main/Agentic_RAG_LangGraph_Colab.ipynb)

### Local Setup

```bash
pip install langgraph langchain langchain-openai langchain-community \
    langchain-chroma chromadb tiktoken bs4 tavily-python
```

Set your API keys:

```python
import os
os.environ["OPENAI_API_KEY"] = "your-openai-key"
os.environ["TAVILY_API_KEY"] = "your-tavily-key"
```

Then open `Agentic_RAG_LangGraph_Colab.ipynb` in Jupyter and run all cells.

## Example Runs

**AI topic → routes to vectorstore:**
```
❓ What are the main components of an AI agent?
🚦 ROUTING → vectorstore
🔍 RETRIEVE → grade docs → ✅ 3 relevant
💡 GENERATE → ✅ grounded
💡 Answer: Planning, memory, and tool use are the three core components...
```

**Current events → routes to web search:**
```
❓ What are the latest developments in AI regulation?
🚦 ROUTING → web_search
🌐 WEB SEARCH → generate
💡 Answer: Biden's Executive Order, new employer duties for bias audits...
```

## Knowledge Base

The default vectorstore is built from three Lilian Weng blog posts:
- [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)
- [Prompt Engineering](https://lilianweng.github.io/posts/2023-03-15-prompt-engineering/)
- [Adversarial Tic-Tac-Toe](https://lilianweng.github.io/posts/2023-10-25-adv-tic-tac-toe/)

Swap in any URLs or local documents by editing the `urls` list in Section 2 of the notebook.

## Why Agentic RAG?

Standard RAG fails silently — it retrieves, generates, and returns regardless of quality. Agentic RAG adds:

- **Relevance filtering** — bad documents are dropped, not passed to the LLM
- **Query refinement** — poor queries are rewritten before retry
- **Source diversity** — web search as a fallback when the vectorstore falls short
- **Grounding verification** — hallucinated answers are caught and regenerated
- **Answer validation** — confirms the question was actually answered

## Part of the RAG Series

This notebook is **Part 2** of a hands-on RAG series:

| Part | Topic |
|------|-------|
| Part 1 | Basic RAG with LangChain |
| **Part 2** | **Agentic RAG with LangGraph** ← you are here |
| Part 3 | Coming soon |

## License

MIT
