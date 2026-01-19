# Overview

## Purpose

Advanced RAG Document Intelligence System provides a single API for document analysis, comparison, and conversational retrieval over uploaded content. It supports sessionized storage and FAISS-based indexing for efficient retrieval, with LLM-backed analysis and comparison.

## Core workflows

1) Analyze
- Upload one PDF and receive structured metadata and summaries.
- Backed by a prompt-driven LLM chain with output parsing.

2) Compare
- Upload two PDFs (reference vs actual).
- Receive a structured comparison summary with rows suitable for tables.

3) Chat (RAG)
- Upload multiple documents to build a FAISS index, optionally per session.
- Query the index with natural language questions.

## Key concepts

- Session ID: Used to segment storage and FAISS indexes per user/session.
- FAISS index: Stored locally on disk under `FAISS_BASE`.
- Chunking: Documents are split before embedding using RecursiveCharacterTextSplitter.

## Runtime components

- FastAPI app orchestrating routes and UI.
- LLM provider configured via `utils/model_loader.py`.
- Prompt registry under `prompt/`.
- Pydantic response models under `model/`.

## Non-goals

- Not a document authoring system.
- Not a full DMS or ECM replacement.

## Supported file types

- Analysis and comparison endpoints currently accept PDFs.
- Chat indexing supports PDF, DOCX, and TXT as defined in ingestion utilities.

## Operational profile

- Stateless API with local storage for indices and uploads.
- Suitable for containerized deployments with mounted volumes.
