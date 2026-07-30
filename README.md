# ResearchMind — Multi-Agent Deep Research System

A production-grade autonomous research system powered by **LangGraph**, featuring a multi-agent pipeline that transforms any query into a structured, citation-backed research report.

## Architecture Overview

For the complete LangGraph node map, routing rules, shared-state contract, and architecture assessment, see [LangGraph Architecture](docs/langgraph-architecture.md).

```
User Query
    │
    ▼
┌─────────────────────────────────────────────┐
│           Interface Layer                   │
│   CLI Terminal │ FastAPI REST │ WebSocket   │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│         Orchestration Layer (LangGraph)     │
│                                             │
│  START → intent → plan → web/local search   │
│       → deep_dive → analyze → reflect?      │
│       → write → END                         │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│              Agent Layer                     │
│  IntentRouter │ Planner │ WebScout           │
│  LocalScout   │ EvidenceJudge │ Analyst      │
│  Reflect      │ Writer                       │
└─────────────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────────────┐
│              Service Layer                   │
│  Bocha Search │ Milvus │ PostgreSQL │ Redis  │
│         Memory Service (Short/Long Term)     │
└─────────────────────────────────────────────┘
```

## Key Features

- **Multi-Agent Orchestration** — 8 specialized agents coordinated by LangGraph state machine, each with a distinct role in the research pipeline
- **Adaptive Reflection Loop** — The Analyst agent evaluates evidence completeness and triggers supplementary searches when gaps are detected
- **Evidence Audit** — EvidenceJudge scores sources by credibility (official: 0.9 / media: 0.7 / community: 0.6), deduplicates, and detects conflicts before analysis
- **Dual-path Retrieval** — Parallel web search (Bocha AI) and local vector search (Milvus RAG) with unified evidence pool
- **Persistent Memory** — Three-tier memory system: short-term (Redis/PostgreSQL), long-term (PostgreSQL), and semantic retrieval (Milvus)
- **Streaming API** — Real-time report streaming via WebSocket and Server-Sent Events
- **Citation-backed Reports** — Every claim in the final report is traceable to a source with credibility metadata

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Agent Orchestration | LangGraph |
| LLM | DeepSeek (OpenAI-compatible API) |
| Web Framework | FastAPI + Uvicorn |
| Web Search | Bocha AI API |
| Vector Database | Milvus |
| Relational Database | PostgreSQL |
| Cache / Short-term Memory | Redis |
| Frontend | Vue 3 + Vite |

## Quick Start

### Prerequisites

- Python 3.10 / 3.11
- Docker Desktop

### 1. Start infrastructure

```bash
# Redis
docker run -d --name redis -p 6379:6379 -e REDIS_PASSWORD=yourpassword bitnami/redis:latest

# PostgreSQL
docker run -d --name postgres -p 5432:5432 \
  -e POSTGRES_USER=root -e POSTGRES_PASSWORD=yourpassword \
  -e POSTGRES_DB=mydb postgres:16
```

### 2. Configure environment

```bash
cp .env.example .env
# Fill in your API keys in .env
```

Required keys:

```env
DASHSCOPE_API_KEY=your_deepseek_api_key
BOCHA_API_KEY=your_bocha_key
REDIS_URL=redis://:yourpassword@127.0.0.1:6379
POSTGRES_DSN=postgresql://root:yourpassword@127.0.0.1:5432/mydb
ENABLE_MILVUS= true
```

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

### 4. Run

```bash
# API server (recommended)
python app/app_main.py

# CLI mode
python main.py
```

Visit `http://localhost:8000/docs` to access the interactive API documentation.

## API Usage

```bash
curl -X POST http://localhost:8000/api/v1/research/run \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What are the best AI products in 2026?",
    "user_id": "default_user",
    "thread_id": "default_thread",
    "tenant_id": "default_tenant",
    "max_iterations": 2,
    "enable_memory": true
  }'
```

## Agent Pipeline

| Agent | Role |
|-------|------|
| **IntentRouter** | Classifies query as `direct` (simple Q&A) or `multiagent` (deep research) |
| **Planner** | Decomposes query into sub-questions and generates a search plan |
| **WebScout** | Executes parallel web searches via Bocha AI |
| **LocalScout** | Queries local knowledge base via Milvus vector search |
| **EvidenceJudge** | Scores, deduplicates, and audits all retrieved evidence |
| **Analyst** | Synthesizes findings, identifies gaps, decides if more research is needed |
| **Reflect** | Generates supplementary queries to fill identified gaps |
| **Writer** | Produces the final Markdown report with citations |

## Project Structure

```
deep_research/
├── app/
│   ├── app_main.py              # FastAPI application entry point
│   ├── backend/
│   │   ├── router/              # API routes (REST + WebSocket)
│   │   ├── service/             # Workflow orchestration service
│   │   └── schemas/             # Request/response models
│   └── mult_agents/
│       ├── graph.py             # LangGraph workflow definition
│       ├── nodes.py             # Agent node implementations
│       ├── state.py             # Shared ResearchState definition
│       ├── prompts.py           # LLM prompts for each agent
│       ├── tools.py             # Search and retrieval tools
│       ├── config.py            # Configuration management
│       ├── memory/              # Memory system (short/long term)
│       └── rag/                 # Local RAG pipeline
├── main.py                      # CLI entry point
├── config.json                  # Default configuration
├── requirements.txt
└── .env.example
```

## License

MIT
