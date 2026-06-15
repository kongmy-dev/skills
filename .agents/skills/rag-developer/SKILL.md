---
name: rag-developer
description: Build and implement traditional single-shot Retrieval-Augmented Generation (RAG) pipelines. Use this skill when the user asks to build a simple RAG application, set up a vector database with embeddings, or implement basic semantic search and generation. If the user wants a more complex, multi-step RAG setup (Agentic RAG) with planning and grading, use the agentic-rag-developer skill instead.
---

# Traditional RAG Developer Skill

This skill guides the implementation of a traditional, single-shot Retrieval-Augmented Generation (RAG) pipeline. This is suitable for simpler use cases where performance and latency are prioritized over multi-step reasoning, or when a naive approach is sufficient.

## Core Workflow

When tasked with building a traditional RAG pipeline, follow these steps:

1. **Ingestion & Indexing:**
   - Extract text from documents.
   - Chunk the text into manageable pieces with overlap to preserve context.
   - Embed the chunks using an embedding model.
   - Store the embeddings and metadata in a Vector Database.

2. **Retrieval:**
   - Receive the user's query.
   - Embed the query using the same embedding model.
   - Perform a similarity search (e.g., Cosine Similarity, dot product) in the Vector Database to retrieve the top $K$ most relevant chunks.

3. **Generation:**
   - Construct a prompt containing the user's original query and the retrieved context chunks.
   - Pass the augmented prompt to an LLM to generate the final response.

## Key Considerations

- **Simplicity:** Keep the architecture straightforward. Avoid unnecessary feedback loops or complex agentic routing unless the user's requirements evolve (at which point, pivot to Agentic RAG).
- **Chunking Strategy:** Consider the chunk size and overlap carefully based on the expected document types (e.g., smaller chunks for dense factual texts, larger chunks for narrative).
- **Metadata:** Store useful metadata (e.g., source URL, page number, date) along with embeddings to allow for metadata filtering during retrieval.

## Example Code Snippet (Framework-Agnostic Concepts)

```python
# 1. Retrieval Phase
query_embedding = embed_model.embed(user_query)
retrieved_docs = vector_db.similarity_search(query_embedding, top_k=5)

# 2. Augmentation Phase
context = "\n".join([doc.text for doc in retrieved_docs])
prompt = f"Use the following context to answer the query.\n\nContext:\n{context}\n\nQuery:\n{user_query}"

# 3. Generation Phase
response = llm.generate(prompt)
print(response)
```

## When to Upgrade to Agentic RAG

Traditional RAG has limitations (e.g., struggling with multi-part questions, relying solely on single-shot retrieval, lacking self-correction). If the user mentions any of the following, recommend upgrading to **Agentic RAG**:
- Multi-step reasoning or complex queries.
- High hallucination rates.
- Need for the system to verify its own answers.
- Utilizing multiple different data sources or tools (e.g., a SQL database + Vector DB).
