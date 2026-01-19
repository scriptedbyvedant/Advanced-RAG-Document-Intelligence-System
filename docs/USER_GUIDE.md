# User Guide

## Getting started

1) Start the server
```bash
uvicorn api.main:app --host 0.0.0.0 --port 8080 --reload
```

2) Open the UI
- `http://localhost:8080`

## Analyze a PDF

1) Select a single PDF file.
2) Submit for analysis.
3) Review the structured metadata returned by the system.

API example
```bash
curl -X POST http://localhost:8080/analyze \
  -F "file=@/path/to/document.pdf"
```

## Compare two PDFs

1) Select a reference PDF and an actual PDF.
2) Submit for comparison.
3) Review the structured rows summarizing differences.

API example
```bash
curl -X POST http://localhost:8080/compare \
  -F "reference=@/path/to/reference.pdf" \
  -F "actual=@/path/to/actual.pdf"
```

## Build a chat index

1) Upload one or more files.
2) Optional: set `session_id` to reuse an existing index.
3) Submit and store the returned `session_id`.

API example
```bash
curl -X POST http://localhost:8080/chat/index \
  -F "files=@/path/to/doc1.pdf" \
  -F "files=@/path/to/doc2.pdf"
```

## Ask questions

1) Use the `session_id` from the index step.
2) Submit a question to retrieve answers from your documents.

API example
```bash
curl -X POST http://localhost:8080/chat/query \
  -F "question=Summarize the contract scope" \
  -F "session_id=abc123"
```

## Tips

- Use shorter prompts for faster responses.
- For large documents, increase `chunk_size` to reduce chunk count.
- Keep `chunk_overlap` between 100 and 300 for better continuity.
