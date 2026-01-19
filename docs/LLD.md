# Low-Level Design (LLD)

## 1. API layer

### Routes

- `GET /` serves the UI (`templates/index.html`).
- `GET /health` returns service health.
- `POST /analyze` accepts PDF and returns metadata JSON.
- `POST /compare` accepts two PDFs and returns comparison rows.
- `POST /chat/index` accepts files and builds FAISS index.
- `POST /chat/query` queries the index for an answer.

### Request/response handling

- FastAPI performs parameter parsing and basic validation.
- Errors are raised as `HTTPException` or wrapped from domain exceptions.

## 2. Document analysis

### Classes

- `DocHandler`
  - `save_pdf(uploaded_file) -> str`
  - `read_pdf(pdf_path) -> str`
- `DocumentAnalyzer`
  - `analyze_document(document_text) -> dict`

### Flow

1) Persist file to `data/document_analysis/<session_id>`.
2) Read text page-wise using `fitz`.
3) Run LLM chain with prompt `document_analysis`.
4) Parse response using `JsonOutputParser` and `OutputFixingParser`.

## 3. Document comparison

### Classes

- `DocumentComparator`
  - `save_uploaded_files(reference_file, actual_file) -> (Path, Path)`
  - `combine_documents() -> str`
- `DocumentComparatorLLM`
  - `compare_documents(combined_docs) -> DataFrame`

### Flow

1) Persist reference and actual PDFs to `data/document_compare/<session_id>`.
2) Read both PDFs and combine into a single text payload.
3) Invoke comparison prompt, parse result into DataFrame.
4) Return `rows` and `session_id`.

## 4. Chat (RAG)

### Classes

- `ChatIngestor`
  - `built_retriver(uploaded_files, chunk_size, chunk_overlap, k)`
- `FaissManager`
  - `load_or_create(texts, metadatas)`
  - `add_documents(docs)`
- `ConversationalRAG`
  - `load_retriever_from_faiss(index_dir, k, index_name)`
  - `invoke(question, chat_history)`

### Flow

Index:
1) Save files to `UPLOAD_BASE/<session_id>`.
2) Load docs into `langchain` `Document` objects.
3) Chunk using `RecursiveCharacterTextSplitter`.
4) Embed and store in FAISS.
5) Deduplicate entries via `ingested_meta.json`.

Query:
1) Validate session and index presence.
2) Load FAISS index with `FAISS.load_local()`.
3) Retrieve top-k chunks.
4) Generate answer via LLM chain.

## 5. Configuration

- `FAISS_BASE`: base path for indices.
- `UPLOAD_BASE`: base path for uploads.
- `FAISS_INDEX_NAME`: index file prefix (default `index`).

## 6. Error handling

- Domain-specific failures raise `DocumentPortalException`.
- API returns `500` with `detail` for internal errors.
- `404` for missing FAISS index when querying.

## 7. Logging

- `logger/GLOBAL_LOGGER` used across the app.
- Key events include session IDs and file paths.

## 8. Threading and concurrency

- FastAPI handles concurrency; model calls are synchronous.
- File I/O is blocking; consider async IO for large-scale workloads.

## 9. Data schemas

- Response schemas defined in `model/models.py`.
- Prompt templates in `prompt/prompt_library.py`.
