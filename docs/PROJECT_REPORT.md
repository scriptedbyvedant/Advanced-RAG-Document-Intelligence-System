# Project Report

## 1. Project overview

Advanced RAG Document Intelligence System is a FastAPI-based service for document analysis, comparison, and conversational retrieval. It combines LLM-based parsing with FAISS indexing to provide structured insights and Q&A capabilities over enterprise documents.

## 2. Objectives

- Provide a unified API for document intelligence.
- Ensure modular pipelines for analysis, comparison, and chat.
- Enable local and cloud deployments with persistent storage.

## 3. Functional requirements

- Upload and analyze PDFs into structured metadata.
- Compare two PDF versions and return structured differences.
- Build and query a session-based vector index for RAG.
- Serve a basic UI for interaction.

## 4. Non-functional requirements

- Reliability: stable API responses and predictable error handling.
- Performance: efficient chunking and indexing for multi-doc workloads.
- Portability: support local and containerized deployment.
- Security: secret management and restricted CORS in production.

## 5. Technology stack

- FastAPI
- LangChain
- FAISS
- PyMuPDF
- Pydantic

## 6. Architecture summary

- `api/main.py` orchestrates incoming requests.
- `src/document_ingestion` handles file persistence and FAISS operations.
- `src/document_analyzer` executes LLM-driven metadata extraction.
- `src/document_compare` produces structured diffs.
- `src/document_chat` runs retrieval-augmented generation.

## 7. Key design decisions

- Session-based storage for isolation and reuse.
- Output parsing with fixing to handle LLM variability.
- Local FAISS index storage for fast retrieval and persistence.

## 8. Testing approach

- Unit tests for core utilities and data flows.
- API smoke tests for analyze, compare, and chat endpoints.

## 9. Deployment strategy

- Local: uvicorn with virtualenv.
- Cloud: ECS deployment with ECR + Secrets Manager.

## 10. Risks and mitigations

- LLM output drift: enforce schema parsing and retry logic.
- Large documents: introduce file size limits and streaming.
- Provider availability: allow multiple providers via `model_loader`.

## 11. Future enhancements

- Auth and access control.
- Docx comparison and OCR for scanned PDFs.
- Observability metrics and tracing.
