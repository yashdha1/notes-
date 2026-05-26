
> link -> https://docs.langchain.com/oss/python/integrations/vectorstores

--- 
##### key_features of vector stores 
1. storage.
2. Retrival.
3. semantic search.
4. similarity search.
5. CRUD operations.
6. Indexing. -> Approximate Nearest Neighbor (ANN) indexing. 
--- 
### 1. InMemoryVectorStore :

> https://reference.langchain.com/python/langchain-core/vectorstores/in_memory/InMemoryVectorStore 

---
### 2. VectorStore : 

> https://reference.langchain.com/python/langchain-core/vectorstores/base/VectorStore
 
 remote connect to the :  External : 
>  FAISS 

provides unified apis over all of these things: 
LangChain provides a unified interface for vector stores, allowing you to:
- `add_documents` - Add documents to the store.
- `delete` - Remove stored documents by ID.
- `similarity_search` - Query for semantically similar documents.
This abstraction lets you switch between different implementations without altering your application logic.
![[Pasted image 20260519131657.png]] 

####  VECTOR DATABASE
chroma, pg_vector, elasticserch. mongoDB, Qdrant, pinecone.

> Q. Look diff in vector stores vs vector DB. 
> 
--- 
--- 

####  EMBEDDING MODEL for EMBEDDING THE DATA

Embedding models transform raw text—such as a sentence, paragraph, or tweet—into a fixed-length vector of numbers that captures its **semantic meaning**. These vectors allow machines to compare and search text based on meaning rather than exact words. 
https://docs.langchain.com/oss/python/integrations/embeddings 

---

### 3. RETRIVERS :

A Retriever is  an interface that returns documents given an unstructured query. It is more general than a vector store. A retriever does not need to be able to store documents, only to return (or retrieve) them. Retrievers can be created from vector stores, but are also broad enough to include [Wikipedia search](https://docs.langchain.com/oss/python/integrations/retrievers/wikipedia) and [Amazon Kendra](https://docs.langchain.com/oss/python/integrations/retrievers/amazon_kendra_retriever).

https://docs.langchain.com/oss/python/integrations/retrievers 


Retrievers accept a string query as input and return a list of [`Document`](https://reference.langchain.com/python/langchain-core/documents/base/Document) objects as output.
Reitrievers are Runnables.

- retreiver search the db or the source based on some res and query rather than matching and ranking of the vector DB. 

These retreivers are much more complex and flexible than the vanilla retreival algorithm of the vector DB. 
has many algorithms **retrivers** types built in.  


```python 
from langchain.retreivers.multi_query import ()
```

---



























