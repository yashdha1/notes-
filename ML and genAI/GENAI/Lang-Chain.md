- opensource framework that allows us to work with all the major LLM providers in chatbots, QA systems, RAG systems, autonoumous agents and a lot more. 
```
Why??
	1. LLM ki variety. 
	2. Simplifies the LLM workflows. 
	3. Major tool support. 
	4. Free.
	5. majorly support all the usecases with langchain.
```


Q] **Why need of langchain?**
-> it has chains. get it? 
-> can create complex pipeline in this sause.
-> Model agnostic devleopment.
-> Complete ecosystem. 
-> Memory and state handeling. at each chain intersection. Also concept of the global memory shitz.
-> Pipeline creations are really expressive. 
-> great in RAG. Doc loading and shit. 

--- 
Q] **what can you do with Langchain?** 
-> chat bot wali shit. Conversational wali shit.
-> Knowledge assistant with specialised usecases. 
-> RAGs.
-> AI Agents ->LLM + Tools (can do something in real worlds)
-> MCP use to handel diffrent shits. 
-> 


--- 

# langchain Components


> **MODEL | PROMPTS | MEMORY | INDEXEX | AGENTS | CHAINS**

--- 
### Models

> https://docs.langchain.com/oss/python/langchain/models 

interface through which we interact with AL models.  koi bhi model provider se convo krne liye. Anthropic, openai, grok any thing is simple. Just a simple interface for the comminications.  
We have *Language Models* and the *Embedding Model* look into them. 

- **Your Question:** "What did my company's CEO say about Q3 earnings?" 
	- **Embedding Model (The Librarian):** Takes your question and converts it into a vector. It then searches through a database of your company's documents (which are also stored as vectors) and **retrieves** the top 3 most relevant paragraphs about Q3 earnings. Returns the vectorised outputs. 
	- **Language Model (The Author):** Takes your original question **and** the 3 retrieved paragraphs. It reads the retrieved context and then **generates** a clear, concise answer citing the CEO's actual statements. returns actual language. 

- **Language Models** are usually **decoder-only** Transformers (like GPT). They have a "causal mask" so they can only see past tokens, not future ones.
- **Embedding Models** are usually **encoder-only** Transformers (like BERT) or specially trained decoder models. They have bidirectional attention, allowing them to understand the full context of every word in the input.

> **Embedding models** measure meaning; **language models** generate language

--- 
### Prompts 
input to LLMs. langchain provides the PromptTemplate to create templates of prompts. 
role based prompt. 
```python 
from langchain_core.prompts import PromptTemplates
```


- **system prompt**    -> highest level to set up the LLMs. sets up the personality of AI. decides rules, personality. 
- **normal prompt**    -> normal conversational prompts. 
- **Few Shot prompt** -> prompting with examples. also can add templates here. like a soft db for the user.  ye dekh ke jawab de lawde. 

--- 
### Chains 

> in LangChains ~ get it? its in the name. 

- To build pipelines. 
- Help so that we don't have to manually pass the data and the information from one state to the next. 
- neo4j dependency resolution and relationships finding shit.

> parallel chaining
> conditional chaining
> simple chaining
> topological chaining. 

--- 
### Indexer
the **knowledge source** 

> Doc loader. -> Text processing. ->  Vector DB. ->  Retreivers. 

-> just the vector storage based upon embedding. to search based upon the all the shits that we enable it to do. 

--- 
### Memory

LLMs are stateless by design. OBVIOSLY.  So we need to store the memory somehow here. 
ways? 
--- 
### Agents

> kaam karrvao agent se. 
> chatbot on oids. 
> just a fucking md file. not even a maintained configuration most of the times. 

*Orchestration* and *Skills* and *Tools* and *MCP* and *Orchestration Layer*. 

---
### Middleware  

prebuilt:   https://docs.langchain.com/oss/python/langchain/middleware/built-in 
Custome: https://docs.langchain.com/oss/python/langchain/middleware/custom 

#### custome middlewares 
Build custom middleware by implementing hooks that run at specific points in the agent execution flow.  

--- 

### runnables 
yes these are runnables too. 

|Method|Purpose|Return Type|Use Case|
|---|---|---|---|
|**`invoke(input)`**|Single synchronous execution|`Output`|One-off requests|
|**`ainvoke(input)`**|Single async execution|`Output`|Async one-off requests|
|**`stream(input)`**|Streaming execution|`Iterator<Output>`|Real-time responses|
|**`astream(input)`**|Async streaming|`AsyncIterator<Output>`|Async real-time|
|**`batch(inputs)`**|Batch processing|`List<Output>`|Multiple inputs at once|
|**`abatch(inputs)`**|Async batch processing|`List<Output>`|Async multiple inputs|