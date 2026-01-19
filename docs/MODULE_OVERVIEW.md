# Module Overview: Document Analysis, Chat & Comparison Portal

## Summary

This module delivers a production-grade document intelligence portal with advanced RAG, LLMOps practices, and end-to-end deployment. It combines ingestion, indexing, semantic search, and conversational interfaces for single and multi-document use cases. It also covers optimization, evaluation, CI/CD, and AWS deployment with FastAPI + Streamlit.

## Outcomes

By the end of this module, teams can:
- Build a robust RAG pipeline with metadata-aware indexing.
- Implement document analysis, comparison, and multi-doc chat.
- Optimize performance with caching, local LLMs, and reranking.
- Deploy to AWS via GitHub Actions and Fargate.

## Topic coverage

### RAG pipeline foundation
- RAG architecture: ingestion, retriever, generator.
- Chunking strategies using LangChain splitters.
- Embeddings via Hugging Face / OpenAI / BGE.
- Vector store selection: FAISS vs Chroma vs Pinecone.
- Metadata indexing (title, page, source ID).

### Advanced RAG enhancements
- Query rewriting and context condensation (RAG Fusion / ReAct).
- Reranking with MMR.
- Similarity metrics: L2, cosine, Euclidean, Jaccard.

### Chat and comparison features
- Session memory and chat history management.
- Document comparison logic (side-by-side, diff detection).
- Document-specific chat (single doc QnA).
- Multi-doc chat (retriever across all sources).

### Performance and optimization
- Local LLMs with vLLM, Groq, or quantized models.
- Cache-augmented generation (CAG) for repeated queries.

### Text processing tools
- PyMuPDF, PDFMiner, Unstructured, python-docx.

### Evaluation and testing
- Evaluate RAG pipelines.
- Final integration test: Upload -> Ask -> Compare -> View Sources.

### Frontend and backend development
- UI with Streamlit or Gradio.
- FastAPI backend for model serving.

### CI/CD and deployment on AWS
- GitHub Actions workflows for automated deployment.
- SonarQube integration for static analysis.
- AWS Secrets Manager for configuration.
- IAM roles, ECR, ECS cluster setup.
- Docker build and push to ECR.
- Deployment on Fargate with env injection.
- Safe rollout and verification using AWS CLI.
