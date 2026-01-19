# High-Level Design (HLD)

## 1. Overview

Advanced RAG Document Intelligence System is a production-grade portal for document analysis, comparison, and conversational retrieval. It exposes a REST API (FastAPI) and UI, backed by modular ingestion, LLM analysis, and FAISS-based retrieval pipelines.

## 2. Goals and objectives

- Provide a unified API to analyze, compare, and chat over documents.
- Deliver structured, reliable outputs from LLM responses.
- Enable sessionized retrieval with persistent indexing.
- Support local and cloud deployment with CI/CD.

## 3. Scope

### In scope
- PDF analysis with metadata extraction.
- PDF comparison with structured diff results.
- Single and multi-document RAG chat.
- Local persistence of uploads and FAISS indices.
- UI served from the same FastAPI app.

### Out of scope
- Enterprise DMS/ECM features (RBAC, audit trails, multi-tenant billing).
- Full document authoring or editing.
- OCR for scanned images (can be added later).

## 4. Assumptions

- LLM and embedding providers are configured via environment variables.
- Local storage is available and persistent in the target environment.
- Inputs are primarily PDFs; other formats may be supported through ingestion.

## 5. System context

### External systems
- LLM provider(s): OpenAI/Gemini/Groq/Claude/HF/Ollama.
- Embedding provider(s).
- CI/CD via GitHub Actions.
- AWS services for deployment (ECR, ECS/Fargate, Secrets Manager).

### Users and actors
- End users: upload documents, ask questions, compare versions.
- Operators: configure deployments, monitor logs.
- Developers: extend prompts, add endpoints, integrate providers.

## 6. Key features

- Metadata extraction with LLM + schema parsing.
- PDF comparison with structured row output.
- FAISS-based retrieval with sessionized indexing.
- Conversational interface for single or multi-doc contexts.
- UI served via Jinja templates and static assets.

## 7. Functional requirements

- Upload and persist documents for analysis and chat.
- Split documents into chunks for embedding.
- Create and update FAISS indexes with deduplication.
- Perform retrieval and LLM generation for answers.
- Compare two PDFs and return structured differences.
- Provide a health endpoint for service checks.

## 8. Non-functional requirements

- Performance: efficient chunking and indexing for multi-file workloads.
- Reliability: deterministic response schema enforcement where possible.
- Security: secret handling, restrictive CORS in production.
- Observability: structured logging for key actions.
- Portability: local and cloud deployments supported.

## 9. Architecture overview

### High-level flow

- Upload -> Persist -> Parse -> Analyze/Compare/Index -> Respond.
- Index -> Retrieve -> Generate -> Respond.

### Components

- API layer: FastAPI routing and UI serving.
- Ingestion layer: file handling, chunking, FAISS persistence.
- Analysis layer: metadata extraction using LLM and parsers.
- Comparison layer: LLM-based structured diffs.
- Retrieval layer: FAISS retriever + LLM response.

## 10. Data management

### Storage

- `UPLOAD_BASE`: persisted uploads.
- `FAISS_BASE`: FAISS index directories.
- `data/document_analysis`: per-session analysis storage.
- `data/document_compare`: per-session comparison storage.

### Data lifecycle

- Input files are stored per session.
- Indices persist across sessions for reuse.
- Old sessions can be cleaned via a maintenance routine.

## 11. Security considerations

- Validate file extensions and enforce size limits at API level.
- Use Secrets Manager or environment variables for credentials.
- Disable permissive CORS in production.
- Avoid logging sensitive document content.

## 12. Observability

- Central logger used across pipelines.
- Log session IDs, file paths, and step completion.
- CloudWatch logging in AWS deployments.

## 13. Deployment overview

- Local: run via `uvicorn api.main:app`.
- Cloud: Docker build and deploy to ECS/Fargate.
- CI/CD: GitHub Actions workflow to build/push to ECR.
- Secrets: injected at runtime via AWS Secrets Manager.

## 14. Extensibility

- Add new LLM or embedding providers via `utils/model_loader.py`.
- Add new prompts in `prompt/prompt_library.py`.
- Add new endpoints in `api/main.py` and document in `docs/API.md`.

## 15. Risks and mitigations

- LLM output variability: enforce schema parsing and retry logic.
- Large documents: apply size limits and chunking strategies.
- Provider outages: allow multiple provider configs.

## 16. Testing strategy

- Unit tests for ingestion and parsing utilities.
- API smoke tests for analyze/compare/chat endpoints.
- Integration tests for end-to-end upload -> retrieve workflow.

## 17. Compliance and licensing

- Apache License 2.0.
- Third-party dependencies credited in README.
