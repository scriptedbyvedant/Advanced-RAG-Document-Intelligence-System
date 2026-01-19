# Architecture

## System overview

```
Client/UI
   |
   v
FastAPI (api/main.py)
   |-- Analyze -> DocHandler -> DocumentAnalyzer -> LLM + Output Parser
   |-- Compare -> DocumentComparator -> DocumentComparatorLLM -> LLM + Parser
   |-- Chat -> ChatIngestor -> FAISS -> ConversationalRAG -> LLM
```

## Component responsibilities

### api/

- `api/main.py` handles HTTP routes, UI rendering, CORS, and request validation.
- Orchestrates workflows by delegating to ingestion, analysis, and retrieval services.

### src/document_ingestion/

- `DocHandler`: saves uploaded PDFs for analysis, reads page-wise text.
- `DocumentComparator`: persists reference/actual PDFs and produces merged content.
- `ChatIngestor`: stores uploads, splits content, creates or loads FAISS index.
- `FaissManager`: load-or-create index with metadata deduplication.

### src/document_analyzer/

- `DocumentAnalyzer`: LLM-based structured metadata extraction.
- Uses prompts from `prompt/` and schemas from `model/`.
- Applies JSON parsing with fixing to reduce malformed output.

### src/document_compare/

- `DocumentComparatorLLM`: LLM chain that returns structured comparison rows.
- Formats response into a DataFrame for API return.

### src/document_chat/

- `ConversationalRAG`: retrieval-augmented generation with FAISS-backed retriever.
- Loads index by session and executes LLM response with context.

### utils/

- `model_loader.py`: LLM/embedding provider integration and credential handling.
- `document_ops.py`: FastAPI file adapters and PDF read helpers.
- `file_io.py`: session IDs and file persistence helpers.

## Runtime sequence: analyze

1) Client uploads PDF to `POST /analyze`.
2) `DocHandler.save_pdf()` persists to `data/document_analysis/<session>`.
3) `DocHandler.read_pdf()` returns concatenated text.
4) `DocumentAnalyzer.analyze_document()` runs LLM chain and parser.
5) API returns JSON metadata.

## Runtime sequence: compare

1) Client uploads reference + actual PDFs to `POST /compare`.
2) `DocumentComparator.save_uploaded_files()` persists to `data/document_compare/<session>`.
3) `DocumentComparator.combine_documents()` builds a single combined text.
4) `DocumentComparatorLLM.compare_documents()` invokes LLM chain.
5) API returns comparison rows with `session_id`.

## Runtime sequence: chat

Indexing:
1) Client uploads files to `POST /chat/index`.
2) `ChatIngestor` saves files under `UPLOAD_BASE/<session>`.
3) Documents split into chunks and embedded.
4) `FaissManager.load_or_create()` creates/loads FAISS index.
5) `FaissManager.add_documents()` deduplicates and persists new chunks.

Query:
1) Client calls `POST /chat/query` with `question`.
2) `ConversationalRAG.load_retriever_from_faiss()` loads index.
3) Retriever gets top-k chunks.
4) LLM generates answer with retrieved context.

## Data storage layout

```
data/
  document_analysis/<session_id>/*.pdf
  document_compare/<session_id>/*.pdf
faiss_index/
  <session_id>/index.faiss
  <session_id>/index.pkl
  <session_id>/ingested_meta.json
```

## Error handling

- All core modules raise `DocumentPortalException` for domain errors.
- API converts exceptions to HTTP 500 with a `detail` payload.
- Input validation occurs at the API boundary (e.g., missing session_id).

## Observability

- Central logger `logger/GLOBAL_LOGGER` used across modules.
- Key actions log session ID, file paths, and pipeline steps.

## Performance considerations

- Chunking and embedding drive latency; tune `chunk_size` and `chunk_overlap`.
- FAISS persistence reduces re-indexing costs across sessions.
- Large PDFs can be memory heavy; consider limiting file sizes at API level.

## Security considerations

- Only file extensions are validated; consider content sniffing for hardening.
- For production, restrict CORS, use auth, and enforce rate limits.
- Store secrets in a vault (AWS Secrets Manager) rather than env files.

## Extensibility

- Add new providers by extending `utils/model_loader.py`.
- Add new prompts in `prompt/prompt_library.py`.
- Add new endpoints in `api/main.py` that reuse ingestion/retrieval components.
