### 1. build it memory blocks 

TEMPORARY
1. **ConversationBufferMemory** -> keep the entire conversational history. 
2. **ConversationBufferWindowMemory** -> keep the entire last k  history. 
3. **ConversationSummaryMemory** -> Keeps a summary and fast version of the entire memory block. 
4. **ConversationSummaryBufferMemory** -> summary + last k conversational blocks are kept.   
```python
from langchain.memory import ( 
								ConversationBufferMemory,
								ConversationBufferWindowMemory,
								ConversationSummaryMemory, 
								ConversationSummaryBufferMemory
							) 
# 
``` 

--- 

###  2. Entity / Knowledge Memories 

1. **ConversationEntityMemory** -> Extracts entities from conversation. used in the personalized assistants and bots. 
2. **ConversationKGMemory**   -> builds relationships based on the covnersations. and between entities. used in reasoning systems. 
--- 
### 3. Vector DB 
LONG TERM and durable. huge memory scale. 

1. **VectorStoreRetrieverMemory** -> Stores the memories in external db. 
	 ex : Chroma. Pinecone. 

--- 
--- 

# Modern MEMEORY MANAGEMENT IN LANGCHAIN AND LANGRAPH 


read about this from the website directly.  https://docs.langchain.com/oss/python/langchain/short-term-memory?utm_source=chatgpt.com 

**checkpointers** :   https://docs.langchain.com/oss/python/integrations/checkpointers 

## Short term Memory management
Thread/session scoped. stores the current conversations, recent messages, working state
Checkpoint-based. 

> LangChain agent manages short-term memory as a part of your agent’s state.  

- By storing these in the graph’s state, the agent can access the full context for a given conversation while maintaining separation between different threads. 
- State is persisted to a database (or memory) using a checkpointer so the thread can be resumed at any time. 
- Short-term memory updates when the agent is invoked or a step (like a tool call) is completed, and the state is read at the start of each step.

1.  **Mounting a memory block to the agent in memory.  raw.** 
```python 
from langchain.agents import create_agent
from langgraph.checkpoint.memory import InMemorySaver  

agent = create_agent(
    "gpt-5.4",
    tools=[get_user_info],
    checkpointer=InMemorySaver(),
)

agent.invoke(
    {"messages": [{"role": "user", "content": "Hi! My name is Bob."}]},
    {"configurable": {"thread_id": "1"}},
)
```

2. **Formatted memory mount using the 'AgentMemory()'** 
```python
from langchain.agents import create_agent, AgentState
from langgraph.checkpoint.memory import InMemorySaver

class CustomAgentState(AgentState):
    user_id: str
    preferences: dict

agent = create_agent(
    "gpt-5.4",
    tools=[get_user_info],
    state_schema=CustomAgentState, # stores memory in certain states .
    checkpointer=InMemorySaver(),
)
```

```
STRATEGIES TO HAVE MORE EFFICIENT AND ECONOMICAL MEMORY MANAGEMENT.  

https://docs.langchain.com/oss/python/langchain/short-term-memory?utm_source=chatgpt.com#common-patterns 

trim messages, delete messages and summarize messages. Using the middleware before the messages hit the middlewares. 
```

#### Short term memory access in Langchain

#### Access memory
managed using the - 
> ToolRunTime 
```python 
from langchain.tools import tool, ToolRuntime

@tool
def get_user_info(
    runtime: ToolRuntime
) -> str:
    """Look up user info."""
    user_id = runtime.state["user_id"]
    return "User is John Smith" if user_id == "user_123" else "Unknown user"

agent = create_agent(
    model="gpt-5-nano",
    tools=[get_user_info],
    state_schema=CustomState,
)
```

> READ  for memory management
> @before_model -> decorater
> @after_model    -> decorater 

---
- Build interactive frontend for these systems is of the utmost Priority. 
- Fully compatible with the JS libs.  react, vue ,svelte and angular. 
---
--- 

