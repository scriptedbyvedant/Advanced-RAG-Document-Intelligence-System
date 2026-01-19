# High-Level Design (HLD)

## 1. Executive summary

The Advanced RAG Document Intelligence System is a modular FastAPI service that provides document analysis, comparison, and conversational retrieval. It combines structured parsing with LLM pipelines and FAISS indexing to support enterprise document workflows.

## 2. Scope

### In scope
- PDF analysis for metadata extraction
- PDF comparison for structured change summaries
- Multi-document indexing and conversational Q&A
- Session-based storage and FAISS indexing
- Web UI served by the API

### Out of scope
- Document authoring or editing
- Full DMS/ECM features (versioning, access control, audit trails)
- Multi-tenant identity and billing

## 3. Actors

- End user: Uploads and queries documents via UI or API.
- Operator: Deploys and configures the service.
- Developer: Extends prompts/providers and endpoints.

## 4. System context

- External dependencies: LLM provider(s), embedding provider, FAISS.
- Storage: local filesystem paths for uploads and indices.
- Interface: REST API with JSON responses, optional UI.

## 5. Core components

- API Gateway (FastAPI)
- Document analysis pipeline (LLM + parser)
- Document comparison pipeline (LLM + parser)
- Ingestion & indexing pipeline (chunking + embeddings + FAISS)
- Retrieval pipeline (FAISS retriever + LLM)

## 6. Data flow overview

- Upload -> Parse -> Store -> Process -> Respond
- Index -> Persist -> Query -> Retrieve -> Generate

## 7. Deployment model

- Containerized service (Docker/ECS supported)
- Mounted volumes for `UPLOAD_BASE` and `FAISS_BASE`
- Secrets stored in vault or environment

## 8. Non-functional requirements

- Availability: stateless API with persistent storage for indices
- Scalability: horizontal scale with shared storage
- Security: secret isolation, input validation, CORS restrictions
- Observability: structured logging

## 9. Risks

- LLM output variability
- Large file performance and memory usage
- Provider rate limits and API failures

## 10. Success criteria

- Consistent structured output from analysis and comparison
- Reliable index creation and retrieval per session
- Clear, stable API surface with documentation
