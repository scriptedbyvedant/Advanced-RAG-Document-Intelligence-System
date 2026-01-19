# API Documentation

Base URL: `http://localhost:8080`

## Health

`GET /health`

Response
```json
{
  "status": "ok",
  "service": "document-portal"
}
```

## Analyze a document

`POST /analyze`

Form-data
- `file` (PDF, required)

Example
```bash
curl -X POST http://localhost:8080/analyze \
  -F "file=@/path/to/document.pdf"
```

Response
```json
{
  "title": "...",
  "summary": "...",
  "author": "...",
  "keywords": ["..."],
  "document_type": "..."
}
```

## Compare two documents

`POST /compare`

Form-data
- `reference` (PDF, required)
- `actual` (PDF, required)

Example
```bash
curl -X POST http://localhost:8080/compare \
  -F "reference=@/path/to/reference.pdf" \
  -F "actual=@/path/to/actual.pdf"
```

Response
```json
{
  "rows": [
    {
      "section": "...",
      "change_type": "...",
      "summary": "...",
      "impact": "..."
    }
  ],
  "session_id": "abc123"
}
```

## Build chat index

`POST /chat/index`

Form-data
- `files` (PDF/DOCX/TXT, required, multi-file)
- `session_id` (string, optional)
- `use_session_dirs` (bool, optional, default: true)
- `chunk_size` (int, optional, default: 1000)
- `chunk_overlap` (int, optional, default: 200)
- `k` (int, optional, default: 5)

Example
```bash
curl -X POST http://localhost:8080/chat/index \
  -F "files=@/path/to/doc1.pdf" \
  -F "files=@/path/to/doc2.pdf" \
  -F "use_session_dirs=true" \
  -F "chunk_size=1000" \
  -F "chunk_overlap=200" \
  -F "k=5"
```

Response
```json
{
  "session_id": "abc123",
  "k": 5,
  "use_session_dirs": true
}
```

## Query chat index

`POST /chat/query`

Form-data
- `question` (string, required)
- `session_id` (string, optional when `use_session_dirs=false`)
- `use_session_dirs` (bool, optional, default: true)
- `k` (int, optional, default: 5)

Example
```bash
curl -X POST http://localhost:8080/chat/query \
  -F "question=Summarize the contract scope" \
  -F "session_id=abc123" \
  -F "use_session_dirs=true" \
  -F "k=5"
```

Response
```json
{
  "answer": "...",
  "session_id": "abc123",
  "k": 5,
  "engine": "LCEL-RAG"
}
```

## Error format

Errors return standard HTTP codes with a JSON payload.

```json
{
  "detail": "Error message"
}
```
