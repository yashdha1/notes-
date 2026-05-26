> fine tuning is expensive thus we cheap out


Doc retreival -> provide to the LLM. 
query -> retreival -> feed LLM info and the retreival.

> mother link : https://www.promptingguide.ai/research/rag  


> RAG takes input and retrieves a set of relevant/supporting documents given a source (e.g., Wikipedia). The documents are concatenated as context with the original input prompt and fed to the text generator which produces the final output. This makes RAG adaptive for situations where facts could evolve over time. This is very useful as LLMs's parametric knowledge is static. RAG allows language models to bypass retraining, enabling access to the latest information for generating reliable outputs via retrieval-based generation.

### Typical RAG WORKFLOW

![[rag-process.webp]]

--- 

### TYPES OF RAGS

![[rag-paradigms.webp]]


--- 
--- 

TOPICS IMPORTANT FOR RAG.

--- 
## DOCUMENT LOADERS
## TEXT SPLITERS 
## VECTOR DB
## RETRIEVERS

---
---
## with RAG 

1. RAGAS  (RAG and Monotoring and Assesment)->  https://medium.com/data-science/evaluating-rag-applications-with-ragas-81d67b0ee31a 
2. Reciprocal Rank Fusion, or RRF. Instead of combining raw scores, it combines rankings. If a document consistently appears near the top across multiple retrieval systems, it gets rewarded regardless of how each model calculates similarity internally.
