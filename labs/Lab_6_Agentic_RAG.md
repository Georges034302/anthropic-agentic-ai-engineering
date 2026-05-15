
# Lab 6: Implement Agentic RAG (Pro Version)

## Objective
Build an agent that retrieves, grounds, and cites enterprise knowledge using a vector database and RAG pipeline, enforcing citation and governance. This lab will guide you through:
- Setting up a vector database
- Ingesting and indexing documents
- Implementing a RAG agent with citation enforcement
- Logging and governance

---

## Step 1: Set Up Vector Database

We'll use FAISS for simplicity. Install dependencies:
```bash
pip install faiss-cpu sentence-transformers
```

```python
# Import libraries
import faiss
from sentence_transformers import SentenceTransformer
import numpy as np
```

```python
# Sample documents to index
documents = [
  "OpenAI released a new model in 2026.",
  "Vector databases enable fast similarity search.",
  "RAG pipelines combine retrieval and generation."
]
```

```python
# Embed documents and build FAISS index
model = SentenceTransformer('all-MiniLM-L6-v2')
embeddings = model.encode(documents)
index = faiss.IndexFlatL2(embeddings.shape[1])
index.add(np.array(embeddings))
```

---

## Step 2: Implement RAG Agent

```python
# RAG agent retrieves, grounds, and cites
def rag_agent(query, top_k=1):
  # Embed the query
  query_vec = model.encode([query])
  # Retrieve top_k relevant docs
  D, I = index.search(np.array(query_vec), top_k)
  retrieved = [documents[i] for i in I[0]]
  # Ground answer in retrieved content
  answer = f"Answer: {retrieved[0]} (Source: doc#{I[0][0]})"
  # Log retrieval and citation
  print(f"[LOG] Query: {query}\n[LOG] Retrieved: {retrieved}\n[LOG] Citation: doc#{I[0][0]}")
  return answer
```

---

## Step 3: Governance and Citation Enforcement

```python
# Governance: enforce citation in every answer
def enforce_citation(answer):
  if "Source:" not in answer:
    raise ValueError("Citation missing in answer!")
  return True
```

---

## Step 4: Run and Log

```python
if __name__ == "__main__":
  query = "What did OpenAI release?"
  answer = rag_agent(query)
  try:
    enforce_citation(answer)
    print("\nFinal Answer with Citation:\n", answer)
  except Exception as e:
    print("[GOVERNANCE]", e)
```

---

## Sample Output
```
[LOG] Query: What did OpenAI release?
[LOG] Retrieved: ['OpenAI released a new model in 2026.']
[LOG] Citation: doc#0

Final Answer with Citation:
 Answer: OpenAI released a new model in 2026. (Source: doc#0)
```

---

## Explanation
- **Vector DB:** Stores and retrieves relevant knowledge chunks.
- **RAG Agent:** Grounds answers in retrieved content and enforces citation.
- **Governance:** Ensures every answer is cited.
- **Logs:** Show retrieval, grounding, and citation steps.

---

## Pro Tips
- Use more documents and chunking for real-world RAG.
- Add more governance (e.g., allowed sources, citation format).
- Integrate with LLMs for answer generation.

---

**Proceed to Lab 7 when you have a working RAG agent and run log.**
