# Advanced RAG Document Intelligence System

Production-grade document intelligence API for analysis, comparison, and conversational retrieval over uploaded documents. Built on FastAPI with modular ingestion, analysis, and RAG pipelines and a UI served from the same app.

## Why this exists

Organizations need to extract metadata, compare document versions, and query corpora without building bespoke tooling. This system provides:
- Document analysis for metadata/summary extraction.
- PDF comparison with structured output.
- Session-based conversational RAG with FAISS indexing.

## Capabilities

- Analyze PDFs into structured metadata using LLM + parsing.
- Compare two PDFs and return a tabular diff summary.
- Build and query a per-session FAISS index for multi-file Q&A.
- Serve a UI from `templates/` and `static/`.

## Architecture at a glance

- `api/main.py`: FastAPI routes, UI, and orchestration.
- `src/document_ingestion`: file handling, FAISS, session storage.
- `src/document_analyzer`: metadata extraction pipeline.
- `src/document_compare`: LLM-based comparison.
- `src/document_chat`: conversational RAG retrieval.

Detailed docs:
- `docs/OVERVIEW.md`
- `docs/API.md`
- `docs/ARCHITECTURE.md`
- `docs/HLD.md`
- `docs/LLD.md`
- `docs/PROJECT_REPORT.md`
- `docs/MODULE_OVERVIEW.md`
- `docs/USER_GUIDE.md`
- `docs/DEVELOPER_GUIDE.md`
- `docs/DIAGRAMS.md`
- `docs/DEPLOYMENT.md`

## Quick start (local)

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
uvicorn api.main:app --host 0.0.0.0 --port 8080 --reload
```

Open the UI at `http://localhost:8080`.

## Configuration

The app reads environment variables at runtime.

```bash
export FAISS_BASE="faiss_index"
export UPLOAD_BASE="data"
export FAISS_INDEX_NAME="index"
```

Model credentials and provider-specific keys are loaded by `utils/model_loader.py`.

## API surface

- `GET /health`: liveness check.
- `POST /analyze`: analyze a single PDF.
- `POST /compare`: compare two PDFs.
- `POST /chat/index`: build an index from multiple files.
- `POST /chat/query`: query an index.

Request/response examples live in `docs/API.md`.

## Deployment

See `docs/DEPLOYMENT.md` for ECS guidance and operational notes.

## Repository layout

```
api/                  FastAPI entry point and web UI
config/               Configuration helpers
docs/                 Industry-level documentation
exception/            Custom exceptions
logger/               Logging setup
model/                Pydantic models and response schemas
prompt/               Prompt library
src/                  Core pipelines
static/               Web assets
templates/            UI templates
tests/                Test suite
utils/                File and model utilities
```

## Tools and technologies

Badges below are visual references and link to official sources.

[![Python](https://img.shields.io/badge/Python-3.10+-3776AB?logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Framework-009688?logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com/)
[![Uvicorn](https://img.shields.io/badge/Uvicorn-ASGI-121212?logo=uvicorn&logoColor=white)](https://www.uvicorn.org/)
[![LangChain](https://img.shields.io/badge/LangChain-Framework-1C3C3C?logo=langchain&logoColor=white)](https://www.langchain.com/)
[![FAISS](https://img.shields.io/badge/FAISS-Vector%20Store-0047AB?logo=meta&logoColor=white)](https://github.com/facebookresearch/faiss)
[![PyMuPDF](https://img.shields.io/badge/PyMuPDF-Text%20Extraction-2C2C2C?logo=adobe&logoColor=white)](https://pymupdf.readthedocs.io/)
[![Pydantic](https://img.shields.io/badge/Pydantic-Data%20Validation-E92063?logo=pydantic&logoColor=white)](https://docs.pydantic.dev/)
[![Docker](https://img.shields.io/badge/Docker-Containerization-2496ED?logo=docker&logoColor=white)](https://www.docker.com/)
[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-2088FF?logo=github-actions&logoColor=white)](https://github.com/features/actions)
[![AWS ECS](https://img.shields.io/badge/AWS%20ECS-Deployment-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ecs/)
[![AWS Fargate](https://img.shields.io/badge/AWS%20Fargate-Serverless-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/fargate/)
[![AWS ECR](https://img.shields.io/badge/AWS%20ECR-Registry-FF9900?logo=amazon-aws&logoColor=white)](https://aws.amazon.com/ecr/)

## Credits

- FastAPI: https://fastapi.tiangolo.com/
- LangChain: https://www.langchain.com/
- FAISS: https://github.com/facebookresearch/faiss
- PyMuPDF: https://pymupdf.readthedocs.io/
- Pydantic: https://docs.pydantic.dev/
- Uvicorn: https://www.uvicorn.org/
- Docker: https://www.docker.com/
- GitHub Actions: https://github.com/features/actions
- AWS ECS/Fargate/ECR: https://aws.amazon.com/

## Requirements

- Python 3.10+
- One LLM provider configured (OpenAI/Gemini/Groq/Claude/HF/Ollama)
- Embedding provider compatible with your LLM choice

## License

Apache License 2.0. See `LICENSE`.
