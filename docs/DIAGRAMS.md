# Diagrams

This document uses Mermaid diagrams. Render in any Markdown viewer that supports Mermaid.

## System context

```mermaid
flowchart LR
  User[End User] --> UI[Web UI]
  User --> API[REST API]
  UI --> API
  API --> LLM[LLM Provider]
  API --> EMB[Embedding Provider]
  API --> FS[Local Storage]
  API --> FAISS[FAISS Index]
```

## High-level data flow

```mermaid
flowchart TB
  Upload[Upload Files] --> Persist[Persist to Storage]
  Persist --> Parse[Parse/Read Content]
  Parse --> Chunk[Chunk Documents]
  Chunk --> Embed[Generate Embeddings]
  Embed --> Index[Build/Update FAISS]
  Index --> Query[User Query]
  Query --> Retrieve[Top-K Retrieval]
  Retrieve --> Generate[LLM Answer]
  Generate --> Response[API Response]
```

## Analyze flow

```mermaid
sequenceDiagram
  autonumber
  participant C as Client
  participant API as FastAPI
  participant DH as DocHandler
  participant LLM as LLM
  participant P as Output Parser

  C->>API: POST /analyze (PDF)
  API->>DH: save_pdf
  API->>DH: read_pdf
  API->>LLM: analyze_document(text)
  LLM-->>P: raw output
  P-->>API: structured metadata
  API-->>C: JSON response
```

## Compare flow

```mermaid
sequenceDiagram
  autonumber
  participant C as Client
  participant API as FastAPI
  participant DC as DocumentComparator
  participant LLM as LLM

  C->>API: POST /compare (ref, actual)
  API->>DC: save_uploaded_files
  API->>DC: combine_documents
  API->>LLM: compare_documents(text)
  LLM-->>API: structured rows
  API-->>C: JSON response
```

## Chat indexing flow

```mermaid
sequenceDiagram
  autonumber
  participant C as Client
  participant API as FastAPI
  participant CI as ChatIngestor
  participant FM as FaissManager

  C->>API: POST /chat/index (files)
  API->>CI: built_retriver
  CI->>FM: load_or_create
  CI->>FM: add_documents
  FM-->>API: retriever
  API-->>C: session_id
```

## Chat query flow

```mermaid
sequenceDiagram
  autonumber
  participant C as Client
  participant API as FastAPI
  participant RAG as ConversationalRAG
  participant FAISS as FAISS Index
  participant LLM as LLM

  C->>API: POST /chat/query (question)
  API->>RAG: load_retriever_from_faiss
  RAG->>FAISS: retrieve top-k
  RAG->>LLM: generate answer
  LLM-->>API: answer
  API-->>C: JSON response
```

## Deployment diagram

```mermaid
flowchart LR
  Client[Browser/Client] --> LB[Ingress/Load Balancer]
  LB --> App[FastAPI Service]
  App --> FS[Mounted Volume]
  App --> FAISS[FAISS Index]
  App --> LLM[LLM Provider]
```

## User journey

```mermaid
flowchart LR
  Start[Start] --> Upload[Upload Documents]
  Upload --> Choice{Task}
  Choice --> Analyze[Analyze]
  Choice --> Compare[Compare]
  Choice --> Chat[Chat]
  Analyze --> ResultA[Structured Metadata]
  Compare --> ResultC[Comparison Rows]
  Chat --> Index[Build Index]
  Index --> Query[Ask Question]
  Query --> ResultQ[Answer]
```

## Component-level sequence diagrams

### DocHandler.save_pdf

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant DH as DocHandler
  participant FS as File System

  API->>DH: save_pdf(uploaded_file)
  DH->>FS: mkdir session path
  DH->>FS: write file bytes
  FS-->>DH: file path
  DH-->>API: saved_path
```

### DocHandler.read_pdf

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant DH as DocHandler
  participant FITZ as PyMuPDF

  API->>DH: read_pdf(pdf_path)
  DH->>FITZ: open(pdf_path)
  FITZ-->>DH: pages
  DH->>DH: extract text per page
  DH-->>API: full_text
```

### DocumentAnalyzer.analyze_document

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant DA as DocumentAnalyzer
  participant LLM as LLM
  participant P as Output Parser

  API->>DA: analyze_document(text)
  DA->>LLM: prompt + text
  LLM-->>DA: raw output
  DA->>P: parse/fix output
  P-->>DA: structured metadata
  DA-->>API: metadata JSON
```

### DocumentComparator.save_uploaded_files

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant DC as DocumentComparator
  participant FS as File System

  API->>DC: save_uploaded_files(reference, actual)
  DC->>FS: create session dir
  DC->>FS: write reference.pdf
  DC->>FS: write actual.pdf
  FS-->>DC: file paths
  DC-->>API: ref_path, act_path
```

### DocumentComparator.combine_documents

```mermaid
sequenceDiagram
  autonumber
  participant DC as DocumentComparator
  participant FITZ as PyMuPDF

  DC->>DC: list session files
  DC->>FITZ: open each PDF
  FITZ-->>DC: page text
  DC->>DC: merge into combined_text
  DC-->>DC: combined_text
```

### DocumentComparatorLLM.compare_documents

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant LLM as LLM
  participant P as Output Parser

  API->>LLM: compare_documents(combined_text)
  LLM-->>P: raw output
  P-->>API: structured rows
```

### ChatIngestor.built_retriver

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant CI as ChatIngestor
  participant IO as File I/O
  participant SPLIT as Splitter
  participant FM as FaissManager

  API->>CI: built_retriver(files, chunk_size, overlap, k)
  CI->>IO: save_uploaded_files
  CI->>IO: load_documents
  CI->>SPLIT: split_documents
  CI->>FM: load_or_create(texts, metas)
  CI->>FM: add_documents(chunks)
  FM-->>CI: retriever
  CI-->>API: retriever
```

### FaissManager.load_or_create

```mermaid
sequenceDiagram
  autonumber
  participant CI as ChatIngestor
  participant FM as FaissManager
  participant FAISS as FAISS

  CI->>FM: load_or_create(texts, metas)
  alt index exists
    FM->>FAISS: load_local(index_dir)
    FAISS-->>FM: vectorstore
  else no index
    FM->>FAISS: from_texts(texts, metas)
    FM->>FAISS: save_local(index_dir)
  end
  FM-->>CI: vectorstore
```

### FaissManager.add_documents

```mermaid
sequenceDiagram
  autonumber
  participant CI as ChatIngestor
  participant FM as FaissManager
  participant FAISS as FAISS
  participant FS as File System

  CI->>FM: add_documents(chunks)
  FM->>FM: fingerprint + dedupe
  FM->>FAISS: add_documents(new_docs)
  FAISS-->>FM: updated index
  FM->>FS: save_local(index_dir)
  FM->>FS: write ingested_meta.json
  FM-->>CI: added_count
```

### ConversationalRAG.load_retriever_from_faiss

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant RAG as ConversationalRAG
  participant FAISS as FAISS

  API->>RAG: load_retriever_from_faiss(index_dir, k)
  RAG->>FAISS: load_local(index_dir, index_name)
  FAISS-->>RAG: retriever
  RAG-->>API: retriever
```

### ConversationalRAG.invoke

```mermaid
sequenceDiagram
  autonumber
  participant API as FastAPI
  participant RAG as ConversationalRAG
  participant RET as Retriever
  participant LLM as LLM

  API->>RAG: invoke(question, history)
  RAG->>RET: get_relevant_documents
  RET-->>RAG: top-k chunks
  RAG->>LLM: generate answer with context
  LLM-->>RAG: answer
  RAG-->>API: answer
```

## Threat model and DFD

### Trust boundaries (DFD Level 0)

```mermaid
flowchart LR
  subgraph TB1[Trusted Zone: VPC]
    API[FastAPI Service]
    FS[Local Storage]
    FAISS[FAISS Index]
    API --> FS
    API --> FAISS
  end
  subgraph TB2[External]
    User[User/Client]
    LLM[LLM Provider]
    EMB[Embedding Provider]
  end
  User --> API
  API --> LLM
  API --> EMB
```

### DFD Level 1

```mermaid
flowchart TD
  User[User] -->|PDFs| API[API Gateway]
  API -->|Persist| Store[(Uploads Storage)]
  API -->|Text| Analyzer[Document Analyzer]
  API -->|Text| Comparator[Document Comparator]
  API -->|Chunks| Ingestor[Chat Ingestor]
  Ingestor -->|Embeddings| Embedder[Embedding Provider]
  Ingestor -->|Index| FAISS[(FAISS Index)]
  API -->|Query| Retriever[Conversational RAG]
  Retriever -->|Top-K| FAISS
  Analyzer -->|Prompt| LLM[LLM Provider]
  Comparator -->|Prompt| LLM
  Retriever -->|Prompt| LLM
  API -->|JSON| User
```

### STRIDE threats (summary)

```mermaid
flowchart TB
  S[spoofing] --> S1[unauthenticated API access]
  T[tampering] --> T1[file upload manipulation]
  R[repudiation] --> R1[missing audit logs]
  I[information disclosure] --> I1[PII in logs or prompts]
  D[denial of service] --> D1[large file uploads]
  E[elevation of privilege] --> E1[overly permissive CORS]
```

## Infra diagram (ECS/VPC)

```mermaid
flowchart LR
  Internet[Internet] --> IGW[Internet Gateway]
  IGW --> ALB[Application Load Balancer]

  subgraph VPC[VPC]
    subgraph Public[Public Subnets]
      ALB
      NAT[NAT Gateway]
    end
    subgraph Private[Private Subnets]
      ECS[ECS Service]
      Task[ECS Task: FastAPI]
      ECS --> Task
    end
    ALB --> ECS
  end

  Task --> ECR[ECR Image]
  Task --> SM[Secrets Manager]
  Task --> CW[CloudWatch Logs]
  Task --> FS[Mounted Volume]
```
