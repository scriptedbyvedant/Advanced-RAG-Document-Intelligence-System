# Architecture

## System overview

```
Client/UI
   |
   v
FastAPI (api/main.py)
   |-- Analyze -> DocumentAnalyzer (LLM + parser)
   |-- Compare -> DocumentComparator + DocumentComparatorLLM
   |-- Chat -> ChatIngestor + FAISS + ConversationalRAG
```

## Modules

### api/

- `api/main.py` exposes HTTP routes, mounts UI assets, and initializes middleware.

### src/document_ingestion/

- `ChatIngestor` handles upload persistence, chunking, embeddings, and FAISS storage.
- `FaissManager` implements a load-or-create pattern with metadata deduplication.
- `DocHandler` saves PDFs for analysis and reads them page-wise.
- `DocumentComparator` saves and combines PDF content for LLM comparison.

### src/document_analyzer/

- `DocumentAnalyzer` loads the LLM and parses structured metadata from documents.
- Uses prompt definitions from `prompt/` and schemas from `model/`.

### src/document_compare/

- `DocumentComparatorLLM` calls the LLM to produce a structured diff result.

### src/document_chat/

- `ConversationalRAG` loads a FAISS index and runs retrieval-augmented answers.

### utils/

- `model_loader.py` centralizes LLM + embeddings provider configuration.
- File I/O helpers manage session IDs and file handling.

## Data flow

1) Upload
- Files are stored under `data/` (configurable) and sessionized if enabled.

2) Chunk + embed
- Documents are split using RecursiveCharacterTextSplitter.
- Embeddings are created via the configured provider.

3) Indexing
- FAISS indices are created under `FAISS_BASE` with `index.faiss/index.pkl`.
- Metadata is stored in `ingested_meta.json` to avoid duplicates.

4) Query
- FAISS retriever returns top-k chunks; LLM forms the final response.

## Storage

- `FAISS_BASE`: persistent vector store path.
- `UPLOAD_BASE`: file upload path.
- `data/document_analysis`: analysis sessions (configurable).
- `data/document_compare`: comparison sessions.

## Extensibility

- Add new providers by extending `utils/model_loader.py`.
- Add new prompts in `prompt/prompt_library.py`.
- Add new endpoints in `api/main.py` that reuse ingestion/retrieval components.
