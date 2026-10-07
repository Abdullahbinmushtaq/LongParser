<p align="center">
  <img src="https://raw.githubusercontent.com/ENDEVSOLS/LongParser/main/docs/assets/logo.png" alt="LongParser" width="320">
  <p align="center"><strong>Privacy-first document intelligence engine for production RAG pipelines.</strong></p>
  <p align="center">
    Parse PDFs, DOCX, PPTX, XLSX &amp; CSV → validated, AI-ready chunks with HITL review.
  </p>
  <p align="center">
    <a href="https://github.com/ENDEVSOLS/LongParser/actions/workflows/ci.yml">
      <img src="https://github.com/ENDEVSOLS/LongParser/actions/workflows/ci.yml/badge.svg" alt="CI">
    </a>
    <a href="https://pypi.org/project/longparser/">
      <img src="https://img.shields.io/pypi/v/longparser.svg?label=pypi&color=0078d4" alt="PyPI">
    </a>
    <a href="https://pepy.tech/project/longparser">
      <img src="https://static.pepy.tech/badge/longparser" alt="Total Downloads">
    </a>
    <a href="https://pepy.tech/project/longparser">
      <img src="https://static.pepy.tech/badge/longparser/month" alt="Monthly Downloads">
    </a>
    <a href="https://www.python.org/">
      <img src="https://img.shields.io/badge/python-3.10%20%7C%203.11%20%7C%203.12%20%7C%203.13-blue.svg" alt="Python">
    </a>
    <a href="LICENSE-THIRD-PARTY.md">
      <img src="https://img.shields.io/badge/License-MIT-brightgreen.svg" alt="MIT License">
    </a>
    <a href="https://endevsols.github.io/LongParser/">
      <img src="https://img.shields.io/badge/docs-online-indigo.svg" alt="Docs">
    </a>
  </p>
</p>

---

## What LongParser does

LongParser reads documents, extracts text and structure, and turns them into smaller chunks with source-page references. An embedding model converts those chunks into vectors. When someone asks a question, retrieval finds relevant chunks and supplies them to an LLM to generate an answer.

```text
Document → Extract text and structure → Chunk → Embed → Retrieve → Answer
```

Scanned pages need OCR to read text from images. Documents with usable text can be parsed directly; full-page OCR is optional. The default PDF extractor uses Docling with Tesseract CLI OCR when enabled.

## Version and release status

Package and runtime versions remain **0.1.5**. The next planned release is **0.1.6**, focused on regression tests, development checks and targeted fixes. All five implementation milestones are complete and tested locally; remote CI and release validation remain pending. These changes are recorded under [Unreleased in the changelog](CHANGELOG.md#unreleased--planned-016).

| Area | Changes planned for 0.1.6 |
|---|---|
| Development checks | Enforce configured Ruff rules, strengthen import/license isolation tests, and run informational mypy checks. |
| Coverage | Measure all production modules and enforce a 95% statement-coverage minimum. |
| Chunking | Add hybrid and semantic chunking regressions, plus optional real embedding tests; fix source-page references for carried equation context. |
| Pipeline | Test routing, PII, references, summaries and utilities; restore PyMuPDF construction and automatic fallback when its dependency is unavailable. |
| Extractors | Test deterministic helpers, optional dependency guards and LaTeX OCR setup without loading OCR models. |

Marker compatibility and official support remain deferred beyond this release.


## Features

| Feature | Detail |
|---------|--------|
| **Multi-format extraction** | PDF, DOCX, PPTX, XLSX and CSV via Docling; optional PyMuPDF for native PDFs |
| **Hybrid chunking** | Estimated token budgets, heading hierarchy, table rows, lists, equations and optional overlap |
| **Semantic chunking** | Optional cosine-similarity boundaries using SentenceTransformer embeddings; default model `all-MiniLM-L6-v2` |
| **Cross-referencing** | Deterministic linking of explicit and implicit charts/figures |
| **Quality scoring** | Zero-ML heuristic scoring with dictionary & fastText validation |
| **PII redaction** | Optional regex and spaCy NER redaction; originals retained in block metadata for authorized HITL review |
| **Summary chunks** | Async ARQ worker generating hierarchical LLM section summaries |
| **HITL review** | Human-in-the-Loop block & chunk editing before embedding |
| **LangGraph HITL** | `approve / edit / reject` workflow with LangGraph `interrupt()` and MongoDB checkpointer |
| **3-layer memory** | Short-term turns + rolling summary + long-term facts |
| **Multi-provider LLM** | OpenAI, Gemini, Groq, OpenRouter |
| **Multi-backend vectors** | Chroma, FAISS, Qdrant |
| **Production-ready API** | FastAPI + Motor (MongoDB) + ARQ + Redis (Queue & Rate Limiting) |
| **Enterprise Security** | Tenant isolation, Role-Based Access Control (RBAC), and CORS |
| **Integration adapters** | LangChain document loader and retriever; LlamaIndex document reader |
| **Local document processing** | Extraction and local Hugging Face embeddings can run on your infrastructure; hosted embedding/LLM providers receive content sent to them |

---

## Installation

### Full GPU bundle

```bash
pip install "longparser[gpu]"
```

Includes the server, SentenceTransformer embeddings, FAISS GPU, Chroma, pix2tex, PPTX helpers, LangChain and LlamaIndex. Qdrant, NER and optional extraction backends are separate extras. Choose the CPU bundle below for CPU-only environments.

### Core SDK (without server extras)

```bash
pip install longparser
```

Includes Docling and the declared core dependencies. Conversion dependencies may include torch and other model packages. Basic package/schema imports are lazy and do not initialize extraction or OCR models.

For the default PDF OCR path, install the Tesseract executable and the language data you need on the host or in your container.

### Pick only what you need

| Extra | What it adds |
|---|---|
| `server` | FastAPI + MongoDB + Redis + LangChain chat |
| `embeddings`, `embeddings-gpu`, `embeddings-cpu` | `sentence-transformers`; installed PyTorch determines CPU/GPU support |
| `faiss-gpu` | FAISS GPU vector store |
| `faiss-cpu` | FAISS CPU vector store |
| `chroma` | ChromaDB |
| `qdrant` | Qdrant |
| `latex-ocr`, `latex-ocr-gpu`, `latex-ocr-cpu` | `pix2tex` equation OCR; installed PyTorch determines CPU/GPU support |
| `pptx` | PowerPoint helpers for indentation detection |
| `docx-equations` | DOCX/PPTX equation extraction helpers |
| `mfd` | `pix2text` formula detection; requires local model setup |
| `ner` | spaCy for contextual PII redaction; its language model is installed separately |
| `pymupdf` | Native PDF extraction through PyMuPDF4LLM (AGPL component) |
| `marker` | Experimental Marker backend (GPL component); compatibility validation deferred |
| `langchain` | LangChain core adapter |
| `llamaindex` | LlamaIndex reader adapter |
| `gpu` | Server + GPU embedding/FAISS/OCR extras + Chroma + PPTX + integration adapters |
| `cpu`, `all` | Corresponding CPU bundle; preinstall CPU PyTorch as shown below |
| `dev` | pytest, coverage, Ruff, mypy and packaging checks |

### CPU-only install

For Docker images, edge devices, or CI environments where CUDA isn't needed:

```bash
# Step 1 — CPU PyTorch
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu

# Step 2 — LongParser CPU bundle
pip install "longparser[cpu]"
```

For optional backend licensing and import isolation, see [LICENSE-THIRD-PARTY.md](LICENSE-THIRD-PARTY.md).

---


## Quick Start

### Python SDK

```python
from longparser import ChunkingConfig, DocumentPipeline, ProcessingConfig

config = ProcessingConfig()
pipeline = DocumentPipeline(config)
result = pipeline.process_file("document.pdf", config=config)
chunks = pipeline.chunk(result, ChunkingConfig())

print(f"Pages: {result.document.metadata.total_pages}")
print(f"Chunks: {len(chunks)}")
if chunks:
    print(chunks[0].text)
```

`process_file()` extracts the document; `chunk()` creates and stores `result.chunks`. Embedding, indexing and retrieval are separate stages. To enable semantic boundaries, pass `ChunkingConfig(use_semantic_chunking=True)` and install an embedding extra.

`max_tokens` is an estimated packing target, calculated from word counts. Indivisible blocks/rows, small-chunk merging and added context can exceed it. See [the chunking test guide](docs/chunking-tests.md#existing-token-budget-policy-dec-04) for the existing policy.

### REST API

```bash
# 1. Copy and edit configuration
cp .env.example .env

# 2. Start the API, background worker, MongoDB and Redis
docker compose up -d --build

# 3. Upload a document
curl -X POST http://localhost:8000/jobs \
  -H "X-API-Key: your-key" \
  -F "file=@document.pdf"

# 4. Check job status; wait for ready_for_review
curl http://localhost:8000/jobs/{job_id} -H "X-API-Key: your-key"

# 5. Finalize and enqueue embedding
curl -X POST http://localhost:8000/jobs/{job_id}/finalize \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"finalize_policy": "approve_all_pending"}'

curl -X POST http://localhost:8000/jobs/{job_id}/embed \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"provider": "huggingface", "model": "BAAI/bge-base-en-v1.5", "vector_db": "chroma"}'

# 6. After job status is indexed, chat with the document
curl -X POST http://localhost:8000/chat/sessions \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"job_id": "your-job-id"}'

curl -X POST http://localhost:8000/chat \
  -H "X-API-Key: your-key" \
  -H "Content-Type: application/json" \
  -d '{"session_id": "...", "job_id": "...", "question": "What is the refund policy?"}'
```

---

## Architecture

```
Document → Extract → Validate → Chunk → HITL Review → Finalize → Embed → Index
                                                                         ↓
                                         Question → Retrieve → LLM → Answer
```

### Pipeline Stages

1. **Extract** — Docling converts documents into structured `Block` objects; optional PyMuPDF handles native PDFs. In `auto` mode, eligible native PDFs use PyMuPDF when available and fall back to Docling when its optional dependency is missing.
2. **Validate** — Per-page confidence scoring and RTL detection
3. **Chunk** — `HybridChunker` builds chunks with section hierarchy and source references; semantic boundaries are optional
4. **HITL Review and finalize** — Human approves/edits/rejects blocks and chunks via the API before embedding
5. **Embed and index** — Hugging Face or OpenAI embedding vectors are stored in Chroma/FAISS/Qdrant
6. **Retrieve and chat** — Relevant chunks provide context for an LLM answer, with 3-layer memory and citation validation

PII redaction is disabled by default. When enabled, it runs before chunking; originals remain in `Block.pii_redactions` for review. Redaction does not delete or encrypt those preserved values.

---

## Project Structure

```
src/longparser/
├── schemas.py           ← core Pydantic models (Document, Block, Chunk, …)
├── extractors/          ← Docling, optional PDF backends, LaTeX OCR
├── chunkers/            ← HybridChunker, semantic boundaries, quality scorer
├── pipeline/            ← DocumentPipeline, PII, references, summary helpers
├── integrations/        ← LangChain loader & LlamaIndex reader
├── utils/               ← shared helpers (RTL detection, …)
└── server/              ← REST API layer
    ├── app.py           ← FastAPI application (all routes)
    ├── db.py            ← Motor async MongoDB
    ├── queue.py         ← ARQ/Redis job queue
    ├── worker.py        ← ARQ background worker
    ├── embeddings.py    ← HuggingFace / OpenAI embedding engine
    ├── vectorstores.py  ← Chroma / FAISS / Qdrant adapters
    └── chat/            ← RAG chat engine
        ├── engine.py    ← ChatEngine (LCEL + 3-layer memory)
        ├── graph.py     ← LangGraph HITL workflow
        ├── schemas.py   ← chat Pydantic models
        ├── retriever.py ← LangChain BaseRetriever adapter
        ├── llm_chain.py ← multi-provider LLM factory
        └── callbacks.py ← observability callbacks
```

---

## LangChain Integration

```python
from longparser.integrations.langchain import LongParserLoader

loader = LongParserLoader("report.pdf")
docs = loader.load()  # list[langchain_core.documents.Document]
```

## LlamaIndex Integration

```python
from longparser.integrations.llamaindex import LongParserReader

reader = LongParserReader()
docs = reader.load_data("report.pdf")
```

---

## Configuration

Copy `.env.example` to `.env` and set:

| Variable | Default | Description |
|----------|---------|-------------|
| `LONGPARSER_MONGO_URL` | `mongodb://localhost:27017` | MongoDB connection |
| `LONGPARSER_REDIS_URL` | `redis://localhost:6379` | Redis for job queue & rate limits |
| `LONGPARSER_LLM_PROVIDER` | `openai` | LLM provider |
| `LONGPARSER_LLM_MODEL` | `gpt-5.3` | Model name |
| `LONGPARSER_EMBED_PROVIDER` | `huggingface` | Embedding provider |
| `LONGPARSER_VECTOR_DB` | `chroma` | Vector store backend |
| `LONGPARSER_CORS_ORIGINS` | `*` | Allowed CORS origins |
| `LONGPARSER_RATE_LIMIT` | `60` | Max RPM per tenant |
| `LONGPARSER_ADMIN_KEYS` | (empty) | Comma-separated admin API keys; empty retains the existing all-users-admin behavior |
| `LONGPARSER_DO_OCR` | `true` | Worker text OCR toggle |
| `LONGPARSER_FORMULA_OCR` | `true` | Worker equation OCR toggle, independent of text OCR |
| `LONGPARSER_GENERATE_SUMMARIES` | `false` | Enqueue LLM summary enrichment after extraction |

---

## Running with Docker

```bash
cp .env.example .env
docker compose up --build
```

API available at `http://localhost:8000` · Docs at `http://localhost:8000/docs`

---

## Testing

Run these commands from the repository root. The default suite uses controlled external dependencies and does not need live MongoDB/Redis services, provider calls or OCR/model inference.

```bash
# Install the same development/server extras used by CI
python -m pip install -e '.[dev,server]'

# Run the full default suite with whole-production coverage
python -m pytest tests/ --cov=longparser --cov-report=term-missing

# Enforce configured lint rules
ruff check .

# Informational type check (currently non-blocking in CI)
mypy src/longparser/

# Run a focused test group
python -m pytest tests/unit/test_extractors.py
```

### Optional real embedding tests

These six tests run actual CPU inference with a pinned `sentence-transformers/all-MiniLM-L6-v2` snapshot. Install the embedding dependencies and cache the required revision using [the preparation instructions](docs/chunking-tests.md#prepare-the-optional-real-model-environment), then run:

```bash
OMP_NUM_THREADS=1 MKL_NUM_THREADS=1 python -m pytest \
  tests/integration/test_real_embeddings.py --run-real-embeddings --no-cov
```

They run offline, are deselected in default CI, and fail an explicit run if dependencies or local assets are missing. The small labelled fixture checks model integration and boundary decisions; it does not measure general retrieval or extraction accuracy.

### Completed Milestone 5 verification (historical)

| Check | Result after Milestone 5 |
|---|---|
| Full default suite, Python 3.13 | **287 passed**; 6 real embedding tests deselected |
| Optional real embedding suite | **6 passed** separately |
| Focused extractor tests | **62 passed** on Python 3.10, 3.11, 3.12 and 3.13, including minimal-dependency environments |
| Whole-production statement coverage | **47.53%** across 40 modules; enforced minimum **45%** |
| Ruff | Passed |
| Mypy | 91 existing findings; informational and non-blocking |
| Package build and metadata | Wheel and source distribution built; both pass Twine checks |
| Documentation build | Passed with `mkdocs build --strict` |
| Clean core install and real parse | Built wheel installed in a fresh environment; real DOCX extraction and explicit chunking passed |
| Remote CI | Pending; the full Python 3.10–3.13 suite and packaging checks still need remote verification |

Coverage includes all production modules and measures code execution, not document/model accuracy. See the [combined validation report](docs/combined-validation.md), [coverage policy](docs/coverage-policy.md), [chunking tests](docs/chunking-tests.md), [pipeline tests](docs/pipeline-tests.md) and [extractor tests](docs/extractor-tests.md) for scope, evidence and remaining validation.

---

### Full-project testing expansion

The expanded local suite reaches **95.97% whole-production coverage**, and the enforced minimum is now **95%**. All **448 default tests pass**, including the repaired search-scope, Docling conversion-state and wide-table regressions. All six real embedding tests pass separately. See the [full-project test report](docs/full-project-testing.md) for scope, module coverage and resolved defects. Remote CI and release acceptance remain pending.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for development setup and PR guidelines.

## Security

See [SECURITY.md](SECURITY.md) for vulnerability reporting.

## License

MIT — Copyright © 2026 ENDEVSOLS. See [third-party licenses](LICENSE-THIRD-PARTY.md) for optional backend terms.
