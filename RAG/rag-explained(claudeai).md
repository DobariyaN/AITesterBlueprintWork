# RAG Explained

## What is RAG?

**Retrieval-Augmented Generation (RAG)** is a pattern where an LLM looks up relevant information from an external knowledge source before answering, instead of relying only on what it learned in training. It helps with private or up-to-date data (your documents, manuals, databases), reduces hallucination, and lets you show sources without retraining the model.

The four stages below make up the pipeline.

---

## 1. Ingestion (offline, done ahead of time)

This is how you prepare your knowledge base:

1. **Load** documents (PDFs, Word files, web pages, database rows).
2. **Clean and parse** them (remove noise, extract tables and text).
3. **Chunk** them into smaller pieces (typically a few hundred tokens, often with some overlap).
4. **Embed** each chunk into a vector.
5. **Store** the vectors and metadata (source, page, date, permissions) in a vector database.

> Chunking quality has a huge effect on final answer quality.

---

## 2. Retrieval (at query time)

When the user asks a question:

1. Embed the question with the same embedding model used in ingestion.
2. Search the vector database for the top-k most similar chunks.
3. Optionally improve results with **hybrid search** (vector plus keyword/BM25), **metadata filters**, and a **reranker** that re-scores the top results.

---

## 3. Augmentation

Here you build the prompt: the user's question, the retrieved chunks as context, and instructions such as "answer only from the context; say so if the answer isn't there; cite sources." The model's input is "augmented" with your data.

---

## 4. Generation

The LLM reads the augmented prompt and writes the final answer, grounded in the retrieved context. You can also add citations and guardrails at this stage.

---

## What is an Embedding?

An embedding is a list of numbers (for example, 768 or 1536 of them) that represents the *meaning* of a piece of data. Texts with similar meaning end up close together in this vector space, so "reset my password" and "I forgot my login" land near each other even though they share few words.

Closeness is measured with cosine similarity, dot product, or Euclidean distance.

---

## What is a Vector Database?

A database built to store embeddings plus metadata and quickly find the nearest vectors to a query vector. Comparing a query against millions of vectors one by one is too slow, so vector databases use **approximate nearest neighbor (ANN)** indexes such as HNSW, IVF, or product quantization. They also support metadata filtering, updates and deletes, and scaling.

---

## How Many Types of Embeddings Are There, Free or Paid?

There is no fixed number. The useful way to count is by category.

### By modality

Text, image, audio, video, code, and multimodal (text and images in the same space, like CLIP-style models).

### By representation

- **Dense** vectors: the standard kind, capturing semantic meaning.
- **Sparse** vectors: keyword-weighted (TF-IDF, BM25, SPLADE).
- **Multi-vector / late interaction**: one vector per token (ColBERT).
- **Classic word-level**: Word2Vec, GloVe, FastText. These are older and rarely used for RAG now.

### By cost

| | Free / open-source (self-host) | Paid (API) |
|---|---|---|
| **Examples** | sentence-transformers (all-MiniLM, all-mpnet), BGE, E5, GTE, Nomic Embed, Qwen3-Embedding, Snowflake Arctic Embed, EmbeddingGemma | OpenAI text-embedding-3, Cohere Embed, Voyage AI, Google Gemini/Vertex embeddings, Amazon Titan, Mistral Embed |
| **Trade-off** | No per-call cost and data stays local, but you need your own compute | Easy and strong quality, but with per-token cost and data leaving your environment |

Some "free" models have license restrictions (for example non-commercial), so check before using one in a product. New models appear constantly, so check a leaderboard such as MTEB for current rankings.

---

## How Many Types of Vector Databases Are There, Free or Paid?

Again, no fixed number (there are dozens). They fall into four groups.

### 1. Purpose-built vector databases

- **Pinecone**: managed, paid (with a free tier)
- **Weaviate, Qdrant, Milvus** (Zilliz Cloud is the managed version): open-source, plus paid cloud offerings
- **Chroma, LanceDB**: open-source, lightweight or embedded, good for prototypes
- **Vespa**: open-source, strong for large-scale hybrid search

### 2. Vector support added to existing databases

- PostgreSQL with **pgvector** (free)
- Elasticsearch and OpenSearch
- MongoDB Atlas Vector Search
- Redis, Cassandra, SingleStore, Oracle, and others

### 3. Libraries (not full databases)

- FAISS, hnswlib, Annoy, ScaNN. These are free and fast, but you handle persistence, filtering, and scaling yourself.

### 4. Cloud-provider services

- Azure AI Search, Vertex AI Vector Search, Amazon OpenSearch and S3 Vectors. These are paid, usage-based, and tied to their clouds.

### Rule of thumb

For a prototype or small dataset, start with **Chroma, FAISS, or pgvector**. If you already run PostgreSQL, pgvector is often enough. For large-scale production, consider **Qdrant, Weaviate, Milvus, or Pinecone**.
