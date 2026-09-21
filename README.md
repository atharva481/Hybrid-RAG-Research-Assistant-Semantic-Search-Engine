# 🔬 Hybrid RAG Research Assistant

> A production-oriented **Retrieval-Augmented Generation (RAG) system for academic research**, combining lexical search, semantic retrieval, intelligent document processing, local LLM generation, caching, observability, and agentic workflows.

The **Hybrid RAG Research Assistant** is an end-to-end AI system designed to retrieve, understand, and answer questions from academic research papers.

Instead of relying only on vector similarity, the system combines **BM25 keyword retrieval** with **semantic vector search** to improve retrieval quality across both exact terminology and conceptually related queries.

The project covers the complete RAG lifecycle:

**arXiv ingestion → PDF parsing → chunking → embedding → hybrid retrieval → context construction → LLM generation → caching → observability → agentic reasoning**

---

## ✨ Key Features

* 🔎 **Hybrid Search** — combines BM25 lexical retrieval with semantic vector search
* 📚 **Automated arXiv Pipeline** — discovers and processes academic papers
* 📄 **PDF Processing** — extracts structured content from research papers
* ✂️ **Section-Aware Chunking** — splits papers into retrieval-friendly overlapping chunks
* 🧠 **Semantic Embeddings** — vector representations generated using Jina AI embeddings
* 🔀 **Reciprocal Rank Fusion** — combines lexical and semantic rankings
* 🤖 **Local LLM Generation** — generates grounded answers using Ollama
* ⚡ **Redis Caching** — reduces repeated retrieval and generation overhead
* 📊 **Langfuse Observability** — traces retrieval and generation behavior
* 🔄 **Airflow Orchestration** — automates ingestion and processing workflows
* 🧩 **Agentic RAG** — LangGraph-based adaptive retrieval and document grading
* 💬 **Gradio Interface** — interactive research assistant UI
* 📱 **Telegram Bot** — query the research assistant remotely

---

## 🏗️ Architecture

```text
                         ┌───────────────────┐
                         │     arXiv API     │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │      Airflow      │
                         │ Ingestion Pipeline│
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │   PDF Processing  │
                         │      Docling      │
                         └─────────┬─────────┘
                                   │
                                   ▼
                         ┌───────────────────┐
                         │ Section-Aware     │
                         │     Chunking      │
                         └─────────┬─────────┘
                                   │
                         ┌─────────┴─────────┐
                         ▼                   ▼
                ┌─────────────────┐   ┌─────────────────┐
                │ BM25 / Lexical  │   │ Semantic Vector │
                │     Search      │   │      Search     │
                └────────┬────────┘   └────────┬────────┘
                         │                     │
                         └──────────┬──────────┘
                                    ▼
                         ┌────────────────────┐
                         │ Reciprocal Rank    │
                         │ Fusion / Hybrid    │
                         │     Retrieval      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Retrieved Context  │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │       Ollama       │
                         │     Local LLM      │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Grounded Response  │
                         └────────────────────┘

             Redis ─────────── Caching
             Langfuse ──────── Observability
             PostgreSQL ────── Metadata Storage
             Gradio ────────── Web Interface
             LangGraph ─────── Agentic RAG
             Telegram ──────── Mobile Interface
```

---

## 🧠 Why Hybrid RAG?

Traditional semantic RAG systems primarily depend on vector similarity.

That works well for conceptual questions but can struggle with:

* exact terminology
* model names
* dataset names
* acronyms
* equations
* author names
* domain-specific keywords

BM25 solves many exact-match problems, while semantic retrieval captures meaning even when the wording differs.

This project combines both.

```text
Query
 │
 ├──► BM25 Search ──────────► Lexical Results
 │
 └──► Vector Search ────────► Semantic Results
                                  │
                 ┌────────────────┘
                 ▼
          Reciprocal Rank Fusion
                 │
                 ▼
          Top Relevant Chunks
                 │
                 ▼
              LLM
                 │
                 ▼
         Grounded Answer
```

The result is a retrieval pipeline capable of handling both **keyword-heavy academic queries** and **natural-language research questions**.

---

## 🔍 Retrieval Pipeline

### 1. Paper Ingestion

Research papers are automatically discovered through the **arXiv API**.

The ingestion pipeline handles:

```text
Search arXiv
    ↓
Fetch metadata
    ↓
Download PDFs
    ↓
Parse documents
    ↓
Store metadata
    ↓
Generate chunks
    ↓
Create embeddings
    ↓
Index for retrieval
```

Apache Airflow is used to orchestrate this workflow.

---

### 2. Document Processing

Academic PDFs are processed using **Docling**.

The pipeline is designed to preserve useful document structure before retrieval.

Configurable processing options include:

* maximum PDF pages
* maximum file size
* table structure extraction
* OCR support
* concurrent document processing

---

### 3. Intelligent Chunking

Large papers cannot be sent directly to an LLM context window.

Documents are therefore divided into smaller overlapping chunks.

Default configuration:

```env
CHUNKING__CHUNK_SIZE=600
CHUNKING__OVERLAP_SIZE=100
CHUNKING__MIN_CHUNK_SIZE=100
CHUNKING__SECTION_BASED=true
```

Section-aware chunking helps preserve the logical organization of academic papers while overlap reduces information loss across chunk boundaries.

---

### 4. Semantic Embeddings

Chunks are converted into dense vector representations using **Jina AI embeddings**.

```text
Research Paper
      ↓
   Sections
      ↓
    Chunks
      ↓
Embedding Model
      ↓
1024-D Vectors
      ↓
  OpenSearch
```

Cosine similarity is used to retrieve semantically related chunks.

---

### 5. BM25 Retrieval

The lexical retrieval pipeline uses **BM25** through OpenSearch.

BM25 is particularly useful when the query contains exact research terminology.

Examples:

```text
"Retrieval-Augmented Generation"
"LoRA"
"BERT"
"attention mechanism"
"GPT-4"
```

---

### 6. Hybrid Retrieval

Lexical and semantic retrieval results are combined through a hybrid ranking pipeline.

```text
             Query
            /     \
           /       \
        BM25      Vector
       Search      Search
          \         /
           \       /
        Rank Fusion
             │
             ▼
      Final Ranking
```

This allows documents that perform strongly under either retrieval strategy to remain competitive while boosting documents relevant to both.

---

## 🤖 RAG Generation

After retrieval, the highest-ranked chunks are assembled into context for the language model.

```text
User Question
      ↓
Hybrid Retrieval
      ↓
Relevant Chunks
      ↓
Context Construction
      ↓
Local LLM
      ↓
Grounded Answer
```

The project uses **Ollama** for local model execution, allowing the RAG pipeline to operate without depending entirely on hosted LLM APIs.

Example configuration:

```env
OLLAMA_MODEL=llama3.2:1b
OLLAMA_TIMEOUT=300
```

---

## 🧩 Agentic RAG

The project also extends traditional RAG using **LangGraph**.

Instead of blindly retrieving documents and sending them to the LLM, the agentic workflow can evaluate retrieval quality and adapt its behavior.

```text
User Query
    ↓
Query Analysis
    ↓
Retrieve Documents
    ↓
Document Grading
    ↓
Relevant?
 ┌──┴───┐
 │      │
Yes     No
 │      │
 ▼      ▼
Generate   Rewrite Query
 │             │
 │             └──► Retrieve Again
 ▼
Answer
```

Agentic capabilities include:

* document relevance grading
* adaptive retrieval
* query rewriting
* out-of-domain detection
* retrieval decision-making
* response generation

---

## ⚡ Redis Caching

Redis is used to cache frequently accessed data and reduce unnecessary repeated computation.

Default cache configuration:

```env
REDIS__HOST=redis
REDIS__PORT=6379
REDIS__DB=0
REDIS__TTL_HOURS=6
```

Caching is especially useful for repeated research queries and retrieval operations.

---

## 📊 Observability with Langfuse

RAG systems are difficult to improve without visibility into their internal behavior.

**Langfuse** provides tracing and observability for the pipeline.

It can be used to inspect:

```text
Query
 ↓
Retrieval
 ↓
Retrieved Documents
 ↓
Context
 ↓
LLM Call
 ↓
Response
```

This makes it easier to debug retrieval failures, analyze generation behavior, and identify latency bottlenecks.

---

## 🛠️ Tech Stack

| Layer                  | Technology              |
| ---------------------- | ----------------------- |
| Language               | Python 3.12             |
| API                    | FastAPI                 |
| API Server             | Uvicorn                 |
| Database               | PostgreSQL              |
| ORM                    | SQLAlchemy              |
| Search Engine          | OpenSearch              |
| Lexical Retrieval      | BM25                    |
| Semantic Retrieval     | Vector Search           |
| Embeddings             | Jina AI                 |
| PDF Processing         | Docling                 |
| Local LLM              | Ollama                  |
| Agent Framework        | LangGraph               |
| LLM Framework          | LangChain               |
| Cache                  | Redis                   |
| Workflow Orchestration | Apache Airflow          |
| Observability          | Langfuse                |
| Interface              | Gradio                  |
| Mobile Interface       | Telegram Bot            |
| Containerization       | Docker / Docker Compose |
| Package Management     | uv                      |
| Testing                | Pytest                  |

---

## 📁 Project Structure

```text
Hybrid-RAG-Research-Assistant-Semantic-Search-Engine/
│
├── airflow/                 # Automated ingestion workflows
├── notebooks/               # Experiments and development notebooks
├── src/                     # Core application source code
├── static/                  # Static assets
├── tests/                   # Automated tests
│
├── .env.example             # Example environment configuration
├── .env.test                # Test environment
├── .pre-commit-config.yaml  # Pre-commit configuration
├── Dockerfile               # Application container
├── compose.yml              # Multi-service infrastructure
├── gradio_launcher.py       # Gradio application entry point
├── Makefile                 # Development commands
├── pyproject.toml           # Dependencies and project configuration
├── uv.lock                  # Reproducible dependency lockfile
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

Make sure the following are installed:

* Docker Desktop
* Docker Compose
* Python **3.12**
* `uv`
* Git

Recommended system resources:

```text
RAM:        8 GB+
Disk Space: 20 GB+
```

---

### 1. Clone the Repository

```bash
git clone https://github.com/atharva481/Hybrid-RAG-Research-Assistant-Semantic-Search-Engine.git

cd Hybrid-RAG-Research-Assistant-Semantic-Search-Engine
```

---

### 2. Configure Environment Variables

Copy the example environment file:

```bash
cp .env.example .env
```

On Windows:

```powershell
copy .env.example .env
```

At minimum, configure the external credentials required by the services you plan to use.

For example:

```env
JINA_API_KEY=your_jina_api_key_here

LANGFUSE_PUBLIC_KEY=your_public_key
LANGFUSE_SECRET_KEY=your_secret_key

TELEGRAM__BOT_TOKEN=your_bot_token
```

> **Never commit your `.env` file or real API keys to Git.**

---

### 3. Install Dependencies

The project uses `uv` for dependency management.

```bash
uv sync
```

---

### 4. Start the Infrastructure

```bash
docker compose up --build -d
```

Check running containers:

```bash
docker compose ps
```

---

### 5. Verify the API

```bash
curl http://localhost:8000/api/v1/health
```

Or open:

```text
http://localhost:8000/docs
```

to access the interactive FastAPI documentation.

---

## 🌐 Services

After starting the Docker environment, the project exposes several development services.

| Service               | Address               | Purpose                      |
| --------------------- | --------------------- | ---------------------------- |
| FastAPI               | `localhost:8000`      | RAG backend                  |
| Swagger UI            | `localhost:8000/docs` | API testing                  |
| Gradio                | `localhost:7861`      | Research assistant interface |
| Airflow               | `localhost:8080`      | Pipeline orchestration       |
| OpenSearch            | `localhost:9200`      | Search engine                |
| OpenSearch Dashboards | `localhost:5601`      | Search inspection            |
| Ollama                | `localhost:11434`     | Local LLM inference          |
| Langfuse              | `localhost:3001`      | RAG observability            |
| Redis                 | `localhost:6379`      | Caching                      |

---

## 💬 Example Research Queries

The system is designed for questions such as:

```text
What are the major approaches to retrieval-augmented generation?

How does retrieval augmentation reduce hallucination?

Explain the transformer attention mechanism.

What techniques are used for efficient fine-tuning of large language models?

Compare semantic search and lexical search.

Find papers discussing hybrid retrieval for RAG systems.
```

The retrieval system identifies relevant paper chunks before passing the evidence to the language model.

---

## 🔌 API

The backend is built with FastAPI and exposes REST endpoints for the RAG system.

Core functionality includes:

```text
GET  /api/v1/health
POST /api/v1/ask
POST /api/v1/stream
POST /api/v1/hybrid-search/
POST /api/v1/ask-agentic
POST /api/v1/feedback
```

Interactive documentation is available at:

```text
http://localhost:8000/docs
```

---

## 🧪 Development

Run the test suite:

```bash
uv run pytest
```

Run with coverage:

```bash
uv run pytest --cov=src
```

Lint the project:

```bash
uv run ruff check .
```

Type checking:

```bash
uv run mypy src
```

---

## 🗺️ System Evolution

The project was developed incrementally, with each stage introducing another component of a production RAG architecture.

| Stage | Implementation                                           |
| ----- | -------------------------------------------------------- |
| 1     | Infrastructure — FastAPI, PostgreSQL, OpenSearch, Docker |
| 2     | arXiv ingestion and PDF processing                       |
| 3     | BM25 lexical retrieval                                   |
| 4     | Semantic embeddings and hybrid search                    |
| 5     | Complete RAG generation pipeline                         |
| 6     | Redis caching and Langfuse observability                 |
| 7     | LangGraph agentic RAG and Telegram integration           |

This progression demonstrates the transition from a basic information retrieval system into a complete AI research assistant.

---

## 🎯 What This Project Demonstrates

This repository focuses on more than simply connecting an LLM to a vector database.

It demonstrates concepts across:

**Information Retrieval**

* BM25
* dense vector retrieval
* cosine similarity
* hybrid search
* rank fusion

**RAG Engineering**

* document ingestion
* academic PDF parsing
* chunking strategies
* embeddings
* context construction
* grounded generation

**Backend Engineering**

* FastAPI
* PostgreSQL
* REST APIs
* asynchronous services
* caching

**MLOps / Infrastructure**

* Docker
* Docker Compose
* Apache Airflow
* Langfuse
* Redis
* automated testing

**Agentic AI**

* LangGraph
* document grading
* adaptive retrieval
* query rewriting
* tool-driven workflows

---

## 🔮 Future Improvements

Potential extensions include:

* [ ] Cross-encoder reranking
* [ ] Retrieval evaluation using Recall@K, MRR and NDCG
* [ ] RAGAS-based generation evaluation
* [ ] Citation-level answer grounding
* [ ] Multi-query retrieval
* [ ] HyDE retrieval
* [ ] Query decomposition for complex research questions
* [ ] Knowledge graph integration
* [ ] Multi-paper comparison
* [ ] Conversation memory
* [ ] Improved research-paper recommendation
* [ ] Cloud deployment
* [ ] CI/CD pipeline

---

## ⭐ Support

If you find this project useful or interesting, consider giving the repository a ⭐.

Contributions, suggestions, bug reports, and improvements are welcome.

---

<p align="center">
  <b>Hybrid retrieval. Grounded generation. Smarter research.</b>
</p>
