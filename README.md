# VSF DLKS Search System

> **Knowledge & Retrieval Platform** for Vietnamese travel data.  
> Built as the knowledge layer for AI-powered travel search — crawls, cleans, enriches, indexes, and retrieves hotel data via hybrid search.

---

## Overview

VSF DLKS (V-Source-Framework Distributed Knowledge & Search) is a **complete RAG pipeline** for the Vietnamese tourism domain. It ingests hotel data from OTA websites (Agoda), processes it through a multi-stage pipeline, and exposes retrieval APIs for downstream AI systems.

**Core pipeline:**

```
crawl → clean → chunk → embed → index → hybrid retrieve → rerank → context + citations → API
```

---

## Architecture (8+2 Layers)

```
L1 - Data Sources          crawler/ , data/
L2 - Ingestion             ingestion/
L3 - Knowledge Engineering knowledge_engineering/ , ontology/
L4 - Indexing              indexing/
L6 - Retrieval             retrieval/
L7 - Context Construction  context/
L8 - Platform Services     api/
Cross-cutting              observability/ , evaluation/
```

### Data Flow

```
crawler/ ──▶ ingestion/ ──▶ knowledge_engineering/ ──▶ indexing/
                                                            │
                                                            ▼
api/ ◀── context/ ◀── retrieval/ ◀── (vector + BM25 index)
```

---

## Tech Stack

| Category | Technologies |
|---|---|
| **Backend** | Python 3.11+, FastAPI, Uvicorn |
| **Database** | PostgreSQL 16, SQLAlchemy, Alembic |
| **Vector Search** | Qdrant, SentenceTransformers (BAAI/bge-m3) |
| **Keyword Search** | OpenSearch 2, rank-bm25 |
| **Crawling** | Scrapy, Playwright, BeautifulSoup, httpx |
| **Data Processing** | Pandas, datasketch (MinHash LSH), deep-translator |
| **LLM Integration** | OpenAI, Gemini, Claude, Ollama (multi-provider adapter) |
| **Ontology** | Faceted taxonomy (8 axes), knowledge graph, synonym dictionary |
| **Container** | Docker, docker-compose (5 services) |
| **Observability** | Prometheus, structlog |

---

## Key Features

### Crawler
- **Config-driven** Playwright crawler for Agoda
- 3-stage batch: `crawl_list → resolve_slugs → crawl_detail`
- Recrawl mode for incomplete records
- Review crawler with pagination, 2-pass sorting, dynamic caps

### Data Processing
- Text normalization, HTML stripping, amenity normalization (fuzzy + canonical)
- Vietnamese translation via Google Translate (cached, batch, CJK detection)
- JSON Schema + Pydantic validation with quality metrics
- MinHash LSH near-duplicate detection (Jaccard ≥ 0.85)

### Ontology System
- **8 orthogonal facets:** object_type, location, amenity, setting, price_tier, purpose, style, aspect
- Knowledge graph: `located_in` (admin hierarchy), `near` (landmark distances)
- Core (stable) / Candidate (discovered from reviews) tier system
- Synonym dictionary + query expansion rules

### Indexing
- **Hybrid chunking:** whole / recursive-sentence / atomic strategies
- BM25 via OpenSearch with strict mapping, nested objects, alias-based promotion
- Vector embeddings via bge-m3 (1024-dim, L2 normalized)

### Retrieval
- Lexical search (multi-field BM25, boosted name/description)
- Vector search (semantic, Qdrant-backed)
- Hybrid search via RRF fusion *(planned)*
- Cross-encoder reranking + freshness boost *(planned)*
- Context construction: selection → aggregation → compression → ordering → citations

### ABSA (Aspect-Based Sentiment Analysis)
- Multi-provider LLM analysis on hotel reviews
- 7 ASPECT concepts: cleanliness, service, location, food, room, value, facilities
- Batch processing (20 reviews/request), incremental resume, cost estimation

---

## Quick Start

```bash
# 1. Environment
cp .env.example .env
# Edit .env with your config (DB, LLM keys, etc.)

# 2. Install dependencies
pip install -r requirements.txt

# 3. Start infrastructure (Docker required)
docker compose up -d

# 4. Run the API
uvicorn api.main:app --reload
```

### Services

| Service | Port |
|---|---|
| API | `localhost:8000` |
| PostgreSQL | `localhost:5432` |
| Qdrant | `localhost:6333` |
| OpenSearch | `localhost:9200` |
| OpenSearch Dashboards | `localhost:5601` |

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/health` | Health check |
| `GET` | `/metrics` | Prometheus metrics |
| `POST` | `/search` | BM25 lexical search |
| `POST` | `/context` | LLM-ready context package |
| `GET` | `/documents/{id}` | Knowledge document |
| `GET` | `/metadata/{id}` | Metadata lookup |

---

## Project Structure

```
vsf_dlks_search_system/
├── api/                    # FastAPI app
├── config/                 # YAML configs
├── contracts/              # JSON Schema contracts
├── crawler/                # Scrapy + Playwright crawler
├── data/                   # Raw/cleaned/processed data
├── db/                     # SQLAlchemy models
├── docs/                   # Design documentation
├── frontend/               # React/HTML demo
├── indexing/               # Embedding + BM25 indexing
├── ingestion/              # Cleaning, validation, dedup
├── knowledge_engineering/  # Chunking, ABSA, profiles, chunking
├── migrations/             # Alembic migrations
├── observability/          # Logging, metrics
├── ontology/               # Facets, knowledge graph, synonyms
├── retrieval/              # Search, filtering, reranking
├── scripts/                # Utilities (benchmark, crawl, index)
├── tests/                  # Test suite
├── docker-compose.yml
└── requirements.txt
```

---

## Documentation

Detailed design documents are in [`docs/`](docs/):

| Document | Description |
|---|---|
| `01_problem_scope.md` | Problem scope, DA10 boundary |
| `02_architecture.md` | Architecture overview |
| `06_retrieval_design.md` | Retrieval infrastructure |
| `07_context_construction.md` | Context construction |
| `08_api_contract.md` | API contract definitions |
| `09_evaluation.md` | Evaluation methodology |
| `10_observability.md` | Observability design |
| `11_sprint_plan.md` | Development roadmap |

---

## License

Personal project — All rights reserved.
