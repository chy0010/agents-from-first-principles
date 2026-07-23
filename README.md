# 🤖 Agents from First Principles

> A from-scratch implementation of the core mechanics behind LLM agents — tool calling, agentic loops, MCP, and RAG — built directly against the OpenAI API with no framework in between.

![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)
![OpenAI](https://img.shields.io/badge/OpenAI-API-412991?logo=openai&logoColor=white)
![MCP](https://img.shields.io/badge/MCP-Model%20Context%20Protocol-orange)
![pgvector](https://img.shields.io/badge/pgvector-PostgreSQL-336791?logo=postgresql&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)

---

## What This Demonstrates

- Calling the OpenAI API directly — chat, memory, streaming, system-prompt personas
- **Function/tool calling** implemented by hand: schemas, `tool_calls` parsing, the `role: tool` round-trip
- An **agentic loop** — the model keeps calling tools until it decides it's done
- **Multi-tool and parallel-tool orchestration** across several APIs
- Reading and reasoning over **CSV and PDF files**
- Building and connecting to an **MCP server** for modular, reusable tools
- **RAG from scratch** — chunking, embeddings, cosine similarity — with no vector library
- Scaling RAG with a **pgvector PostgreSQL** vector database

No frameworks (LangChain, the Agents SDK, etc.) — everything is written directly against the raw OpenAI API so the mechanics underneath those frameworks are fully visible.

---

## Quick Start

```bash
# 1. Clone and enter the repo
git clone https://github.com/chy0010/agents-from-first-principles.git
cd agents-from-first-principles

# 2. Set up your environment
cp .env.example .env
# → Add your OpenAI API key (and PostgreSQL credentials for RAG examples)

# 3. Install dependencies
pip install openai python-dotenv requests pandas pypdf mcp anyio numpy psycopg2-binary

# 4. Run your first agent
python agent.py
```

> Type `quit` to exit any chat-loop example.

---

## Implementation Map

Each script is a self-contained example of one concept, in increasing order of complexity.

| # | File | Concept |
|---|------|---------|
| 1 | `agent.py` | Single-turn chat with the OpenAI API |
| 2 | `memory_agent.py` | Multi-turn memory via message history |
| 3 | `personality_bot.py` | System prompts and persona shaping |
| 4 | `streaming_bot.py` | Streaming responses token-by-token |
| 5 | `calculator_agent.py` | Function calling — first tool integration |
| 6 | `weather_agent.py` | Calling a real external API via tool use |
| 7 | `stock_agent.py` | Multiple tools in one agent |
| 8 | `multi_tool_agent.py` | Model chooses between calculator, weather & stocks |
| 9 | `file_reader_agent_csv.py` | Reading & summarising CSV data with pandas |
| 10 | `file_reader_agent_pdf.py` | Extracting text from PDFs and Q&A over them |
| 11 | `orchestration_agent.py` | Chaining tool calls to combine results |
| 12 | `mcp_server.py` | Building a reusable MCP tool server |
| 13 | `mcp_client.py` | Connecting to the MCP server directly |
| 14 | `mcp_agent.py` | Routing OpenAI tool calls through MCP |
| 15 | `agent_loop.py` | Agentic loop — model runs until no more tool calls |
| 16 | `rag_agent.py` | RAG from scratch: chunking, embeddings, cosine similarity |
| 17 | `ingest_pdf_pgvector.py` | Ingesting a PDF into PostgreSQL with pgvector |
| 18 | `rag_pgvector.py` | RAG with pgvector — vector search via SQL |

---

## Architecture Overview

```
Basic Chat ──► Memory ──► Personality ──► Streaming
                                              │
                                              ▼
                            Tool Calling (calculator / weather / stocks)
                                              │
                                              ▼
                               File Readers (CSV / PDF)
                                              │
                                              ▼
                              Orchestration Agent (multi-tool)
                                              │
                                              ▼
                         MCP Server ◄──── MCP Agent ◄──── Agent Loop
                                              │
                                              ▼
                    RAG (in-memory) ──► pgvector (PostgreSQL)
                    chunk → embed → similarity search → answer
```

---

## RAG Setup (pgvector)

The RAG examples require a PostgreSQL database with the pgvector extension.

```sql
-- Run once in PostgreSQL
CREATE EXTENSION IF NOT EXISTS vector;

CREATE TABLE document_chunks (
    id SERIAL PRIMARY KEY,
    chunk_text TEXT,
    embedding vector(1536)
);
```

Then ingest your PDF and query it:

```bash
# Terminal 1 — ingest the PDF into the database
python ingest_pdf_pgvector.py

# Terminal 2 — ask questions
python rag_pgvector.py
```

---

## Sample Data

| File | Used by |
|------|---------|
| `Fact_Sales_1.csv` | `file_reader_agent_csv.py` |
| `Meta.pdf` | `file_reader_agent_pdf.py`, `rag_agent.py`, `ingest_pdf_pgvector.py` |
| `sample.txt` | General file-reading practice |

---

## Setup Details

### Environment Variables

Copy `.env.example` to `.env` and fill in your keys:

```
OPENAI_API_KEY=sk-...

# For pgvector RAG examples
PG_HOST=localhost
PG_PORT=5432
PG_DATABASE=rag_db
PG_USER=postgres
PG_PASSWORD=your_password
```

### Running the MCP Examples

The MCP agent requires the server to be running first:

```bash
# Terminal 1 — start the server
python mcp_server.py

# Terminal 2 — run the agent or client
python mcp_agent.py
python agent_loop.py
```

---

## Notes

- The `calculator_agent.py` uses Python `eval()` — fine for demos, but swap it for a proper math parser in production.
- All agents default to `gpt-4o-mini` to keep API costs low while learning.
- `rag_agent.py` keeps embeddings in memory — fast for small docs, but doesn't persist between runs. Use `rag_pgvector.py` for persistence.
- No frameworks (LangChain, etc.) are used on purpose — everything is written against the raw OpenAI API so you can see exactly what's happening.
