# RAG API with FastAPI (Local Prototype)

This project is a Retrieval-Augmented Generation (RAG) API built with FastAPI.  
It answers questions by retrieving relevant context from a vector database and using an LLM to generate grounded responses.

## Components
- **FastAPI** – API framework and endpoints
- **Uvicorn** – ASGI server to run the API
- **ChromaDB** – Vector store for embeddings + similarity search
- **Ollama** – Local LLM inference
- **Swagger UI** – Auto API documentation and testing

## How it works (RAG flow)
1. Content is embedded and stored in ChromaDB
2. A user sends a question to `/query`
3. Chroma retrieves the most relevant context
4. The LLM generates an answer using the retrieved context
5. The API returns the response

## Cloud mapping (how this becomes production)
In production, this same pattern maps to cloud services like:
- Object storage (S3/GCS) for documents
- Managed embeddings + model endpoints (Bedrock/Vertex/etc.)
- Managed vector search (OpenSearch / vector DB)
- API Gateway + container/serverless runtime
- CI/CD for automated updates and governance

## Run locally
```bash
pip install -r requirements.txt
uvicorn app:app --reload
