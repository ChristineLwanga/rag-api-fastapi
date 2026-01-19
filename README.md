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
## Failure Testing & Hardening

### Failure: Empty Vector Search Result

I intentionally tested the API with a query that had no matching documents in ChromaDB.
This caused an `IndexError` because the application assumed that search results would
always return at least one document.

### Root Cause
ChromaDB can return an empty list when no relevant documents are found.  
The original implementation accessed nested list indexes without validating their existence.

### Fix
I added defensive checks to safely handle empty search results:

```python
docs = results.get("documents") or []
context = docs[0][0] if docs and docs[0] else ""
```

## Failure Testing & Hardening
1️⃣ Two Different Ways Knowledge Enters the System

This RAG API supports two separate knowledge ingestion paths, both of which store data in the same vector database (Chroma), but behave differently.

📁 File-based ingestion (k8s.txt)

k8s.txt contains static, developer-provided knowledge

Example content:

Kubernetes is a container orchestration platform used to manage containers at scale.
My name is Tina
I come from Kenya


This file is read by embed.py

The contents are embedded and stored in Chroma only when embed.py is executed

📌 Editing the file alone does not update the knowledge base.

🌐 API-based ingestion (POST /add-knowledge)

Swagger UI was used to inject runtime knowledge

Example input:

My name is Christine Lwanga


This text is immediately embedded and stored in Chroma

No files are modified on disk

📌 This knowledge exists only in the vector database, not in k8s.txt.

2️⃣ Why the Answers Didn’t Match the File Contents

After updating k8s.txt with:

My name is Tina
I come from Kenya


The API still responded:

Christine Lwanga

✅ Root Cause

Chroma already contained a document added earlier via /add-knowledge

The ingestion script originally used collection.add() with a fixed ID:

collection.add(documents=[text], ids=["k8s"])


Chroma does not overwrite existing IDs

As a result, edits to k8s.txt were ignored until the ingestion logic was corrected

🔧 Fix Applied

The ingestion logic was updated to use upsert():

collection.upsert(documents=[text], ids=["k8s"])


This ensures:

The document is updated if it already exists

File edits are correctly reflected in the vector database

3️⃣ Why Some Responses Appeared Inconsistent or “Funny”

When querying:

Where do I come from?


The API returned varying responses, including:

An “ambiguous” explanation

An inferred answer

A blended response referencing Kubernetes and identity information

🧠 Explanation

This is expected RAG behavior when:

The question is ambiguous

Retrieved context is weak or mixed

The prompt allows the LLM to infer missing information

In this project:

All file content was embedded as one large document

Identity data and technical definitions existed in the same chunk

The model attempted to be helpful by combining context

📌 This is not a bug — it highlights the importance of:

Prompt constraints

Clear chunking strategies

Explicit grounding instructions

4️⃣ Key Technical Takeaways

Editing source files does not affect a RAG system unless embeddings are regenerated

Vector databases require explicit update logic (upsert)

Runtime and static knowledge can coexist but must be managed carefully

Ambiguous questions lead to under-constrained generation

Prompt discipline is essential for reliable RAG behavior

5️⃣ Why This Matters (Real-World Relevance)

This behavior mirrors real production RAG systems:

Documentation vs live updates

User-injected vs curated knowledge

The need for controlled grounding in regulated environments

Understanding this distinction is critical before containerizing and deploying the system.

6️⃣ Evidence (Screenshots)

The following screenshots demonstrate:

Editing k8s.txt in VS Code

Running embed.py to upsert embeddings

Querying the API via terminal and Swagger UI

Observing differences in responses before and after re-embedding
<img width="1459" height="451" alt="image" src="https://github.com/user-attachments/assets/67c56eb9-9fb4-4641-8478-e462ba7e1dcb" />
<img width="892" height="323" alt="image" src="https://github.com/user-attachments/assets/bcbf10f0-6dac-463c-bbdd-b8853c10a6ae" />
<img width="1912" height="1033" alt="image" src="https://github.com/user-attachments/assets/6713cb4b-de15-474b-ac9f-6e477ffc43ad" />
<img width="1852" height="892" alt="image" src="https://github.com/user-attachments/assets/d053af31-c25c-4667-b6e5-6f9eb486553b" />
<img width="1855" height="900" alt="image" src="https://github.com/user-attachments/assets/e799fb1b-936a-4d51-a8da-8cf341b2b55b" />
<img width="1838" height="810" alt="image" src="https://github.com/user-attachments/assets/f5f8812f-5286-453f-9754-240966094dd8" />
<img width="1802" height="866" alt="image" src="https://github.com/user-attachments/assets/7b305431-2ded-402f-8d11-870058f9605a" />
<img width="1813" height="972" alt="image" src="https://github.com/user-attachments/assets/b066c887-7ef0-482f-9a33-91a2e1a376f9" />
