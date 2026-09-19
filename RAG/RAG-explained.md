# RAG Explained: Retrieval-Augmented Generation

RAG stands for **Retrieval-Augmented Generation**. It is a way to make an AI model answer questions using your own documents, databases, PDFs, websites, test cases, product documentation, or other knowledge instead of relying only on what the model learned during training.

A simple RAG flow looks like this:

```
Documents → Ingestion → Embeddings → Vector Database → Retrieval → Augmentation → LLM Generation → Answer
```

For example, imagine you have 500 QA documents containing test cases, bug reports, API documentation, and regression checklists. You ask:

> "What regression tests should I run for the payment module?"

Instead of guessing, a RAG system searches your documents, finds the relevant payment-related information, gives that context to the LLM, and generates an answer based on it.

---

## Contents

1. [What is Ingestion?](#1-what-is-ingestion)
2. [What is Retrieval?](#2-what-is-retrieval)
3. [What is Augmentation?](#3-what-is-augmentation)
4. [What is Generation?](#4-what-is-generation)
5. [What is an Embedding?](#5-what-is-an-embedding)
6. [Main types of embeddings](#6-main-types-of-embeddings)
7. [Free vs paid embeddings](#7-free-vs-paid-embeddings)
8. [What is a Vector Database?](#8-what-is-a-vector-database)
9. [What normally gets stored in a vector database?](#9-what-normally-gets-stored-in-a-vector-database)
10. [Types of vector databases](#10-types-of-vector-databases)
11. [Free/open-source vector databases](#11-freeopen-source-vector-databases)
12. [Paid/managed vector databases](#12-paidmanaged-vector-databases)
13. [Complete RAG architecture](#13-complete-rag-architecture)

---

## 1. What is Ingestion?

Ingestion means preparing your data so the RAG system can search it later.

Suppose you have:

- PDFs
- Word documents
- HTML pages
- Jira tickets
- Test cases
- API documentation
- GitHub files
- Database records

The ingestion pipeline usually does:

```
Load document
      ↓
Extract text
      ↓
Clean text
      ↓
Split into chunks
      ↓
Create embeddings
      ↓
Store vectors + metadata
      ↓
Vector Database
```

**Example:**

Original document:

> Login functionality allows users to authenticate using email and password. After five failed login attempts the account is temporarily locked.

It might be divided into chunks such as:

**Chunk 1:**

```
Login functionality allows users to authenticate
using email and password.
```

**Chunk 2:**

```
After five failed login attempts,
the account is temporarily locked.
```

Each chunk gets converted into an embedding.

---

## 2. What is Retrieval?

Retrieval means finding the most relevant information for the user's question.

Suppose the user asks:

> "When does the account get locked?"

The system creates an embedding for that question and searches the vector database.

It might retrieve:

```
Chunk:
"After five failed login attempts,
the account is temporarily locked."
```

That is the retrieval stage.

Typically the system retrieves something like:

- Top 3 chunks
- Top 5 chunks
- Top 10 chunks

depending on the application.

---

## 3. What is Augmentation?

This is where the word *Augmented* in RAG comes from.

The retrieved information is inserted into the prompt sent to the LLM.

**Without RAG:**

```
User Question
     ↓
LLM
     ↓
Answer
```

**With RAG:**

```
User Question
     ↓
Retrieve relevant documents
     ↓
Add documents to prompt
     ↓
LLM
     ↓
Answer
```

For example:

```
SYSTEM PROMPT:

Use the following company documentation
to answer the question.

Context:
After five failed login attempts,
the account is temporarily locked.

Question:
When does the account get locked?
```

This extra context is the augmentation.

---

## 4. What is Generation?

Generation is when the LLM produces the final response.

Examples of generators include models such as:

- GPT models
- Claude
- Gemini
- Llama
- Mistral
- Qwen
- DeepSeek

Using the retrieved context, the model might answer:

> The account is temporarily locked after five failed login attempts.

Therefore:

| Stage | What it does |
|---|---|
| **Retrieval** | Find information |
| **Augmentation** | Add information to the prompt |
| **Generation** | LLM writes the answer |

---

## 5. What is an Embedding?

An embedding is a numerical representation of meaning.

For example:

```
"Login failed"
```

might be converted into something conceptually like:

```
[0.21, -0.56, 0.81, 0.12, ...]
```

A real embedding can contain hundreds or thousands of numbers.

For example:

```
Text
"User cannot login"

        ↓

Embedding Model

        ↓

Vector

[0.12, 0.78, -0.33, 0.51, ...]
```

**Why do this?**

Because computers can mathematically compare vectors.

For example:

- "Login failed"

and

- "User cannot sign in"

use different words but have similar meanings.

Their vectors might therefore be located close together.

Conceptually:

```
             User cannot sign in
                    ●
                   /
                  /
      Login failed ●


                                    ● Banana recipe
```

The login sentences are close together because they are semantically related.

---

## 6. Main types of embeddings

There isn't a fixed number of embedding types. Embeddings can be classified in several ways.

A useful classification is:

| Embedding type | Used for |
|---|---|
| Text embeddings | Documents, questions, articles |
| Image embeddings | Image similarity/search |
| Audio embeddings | Speech/audio search |
| Video embeddings | Video understanding/search |
| Code embeddings | Source-code search |
| Multimodal embeddings | Text + images + other modalities |

For RAG, the most common is: **text embedding**.

Examples of embedding model families include:

- OpenAI embeddings
- Cohere Embed
- Google embedding models
- Voyage AI embeddings
- BGE embeddings
- E5 embeddings
- Sentence Transformers
- Jina embeddings
- Nomic embeddings

---

## 7. Free vs paid embeddings

There are broadly two ways to use embedding models.

### Paid API embeddings

You call a hosted API.

Examples include providers such as:

- OpenAI
- Cohere
- Google
- Voyage AI
- Jina AI

The flow is:

```
Your text
   ↓
Embedding API
   ↓
Vector
```

You generally pay according to usage.

**Advantages:**

- Easy setup
- No GPU required
- Managed infrastructure
- Good performance
- Easy scaling

### Free/open-source embeddings

You can run models yourself.

Popular model families include:

- BGE
- E5
- Sentence Transformers
- Nomic
- GTE
- Jina open models

For example, Python developers often use:

```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer("all-MiniLM-L6-v2")
```

**Advantages:**

- No per-request API cost
- Can run locally
- Better data privacy
- Useful for experimentation

But there may still be infrastructure costs if you run them on cloud GPUs/CPUs.

So open-source doesn't necessarily mean zero operational cost.

---

## 8. What is a Vector Database?

A vector database stores embeddings and searches them efficiently.

Traditional databases primarily search things like:

```
ID = 105
name = "John"
status = "failed"
```

A vector database can instead search by semantic similarity.

**Example:**

Stored vectors:

```
Document 1
"Login failure troubleshooting"
→ [0.22, 0.77, ...]

Document 2
"Payment API testing"
→ [0.81, 0.13, ...]

Document 3
"Authentication regression testing"
→ [0.25, 0.72, ...]
```

Question:

> "How do I test login problems?"

gets converted into a vector.

The vector database then finds the vectors nearest to the question vector.

This is commonly called: **Similarity Search** or **Nearest Neighbor Search**.

---

## 9. What normally gets stored in a vector database?

Usually something like:

```
Vector
+
Original text
+
Metadata
```

**Example:**

```json
{
  "id": "TC_LOGIN_001",
  "text": "Verify login with invalid password",
  "vector": [0.23, 0.65, 0.91],
  "metadata": {
    "module": "login",
    "type": "negative",
    "priority": "high"
  }
}
```

Metadata lets you apply filters.

For example:

```
module = login
priority = high
```

before or during semantic retrieval.

---

## 10. Types of vector databases

Again, there isn't one fixed number of types.

A useful way to classify them is:

| Type | Example use |
|---|---|
| Dedicated vector database | Production RAG/search |
| Traditional DB with vector extension | Existing database + AI |
| Search engine with vector search | Hybrid keyword + vector search |
| Local/in-memory vector store | Development/testing |

Examples include:

- Pinecone
- Qdrant
- Weaviate
- Milvus
- Chroma
- FAISS
- PostgreSQL + pgvector
- Elasticsearch
- OpenSearch
- MongoDB Vector Search
- Redis Vector Search
- Azure AI Search

---

## 11. Free/open-source vector databases

Common free or open-source choices include:

- FAISS
- Chroma
- Qdrant
- Weaviate
- Milvus
- pgvector
- OpenSearch

A beginner RAG application commonly looks like:

```
Python
+
Sentence Transformers
+
Chroma
```

or:

```
Python
+
OpenAI embeddings
+
Qdrant
```

---

## 12. Paid/managed vector databases

Many products also provide managed cloud offerings.

Examples include:

- Pinecone
- Qdrant Cloud
- Weaviate Cloud
- Zilliz Cloud
- MongoDB Atlas Vector Search
- Elastic Cloud
- Azure AI Search

They typically handle things such as:

- Infrastructure
- Scaling
- Backups
- Replication
- Monitoring
- Security
- High availability

Many have some combination of free trial, free tier, usage-based pricing, and paid production plans, so the exact free/paid boundary can change over time.

---

## 13. Complete RAG architecture

Here is the whole idea together.

```
                 INGESTION
────────────────────────────────────

PDF / Website / Jira / Test Cases
                │
                ▼
          Document Loader
                │
                ▼
           Text Cleaning
                │
                ▼
             Chunking
                │
                ▼
        Embedding Model
                │
                ▼
           Vector Database


                 QUERY
────────────────────────────────────

User Question
"How should login lockout be tested?"
                │
                ▼
        Embedding Model
                │
                ▼
        Question Vector
                │
                ▼
         Vector Database
                │
                ▼
          RETRIEVAL
      Relevant Documents
                │
                ▼
          AUGMENTATION
Question + Retrieved Context
                │
                ▼
               LLM
                │
                ▼
           GENERATION
                │
                ▼
        Final RAG Answer
```

For a QA/Test Analyst, you could build a RAG system over:

- Requirements
- User stories
- Test cases
- Automation code
- API documentation
- Jira bugs
- Production incidents
- Regression documentation
- Release notes

Then ask questions such as:

- "Generate negative test scenarios for this feature using our previous defects."
- "Which previous production bugs are related to this API?"
- "What regression tests are relevant for this user story?"
- "Find similar defects reported during previous releases."

That is where RAG becomes very useful in QA.

---

## Quick recap

A simple way to remember everything is:

| Term | Meaning |
|---|---|
| **Ingestion** | Prepare knowledge |
| **Embedding** | Convert meaning into numbers |
| **Vector DB** | Store/search those numbers |
| **Retrieval** | Find relevant knowledge |
| **Augmentation** | Give that knowledge to the LLM |
| **Generation** | Produce the final answer |
