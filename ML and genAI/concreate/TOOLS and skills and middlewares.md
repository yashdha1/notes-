
# TOOLS 

https://docs.langchain.com/oss/python/langchain/tools 

tools are callable functions with well-defined inputs and outputs that get passed to a chat model. Agents decides when to call them with whatever params they decide it to do? 
from langchain.tools import tool

> 1.  Type hints are **required** as they define the tool’s input schema. 
> 2. The docstring should be informative and concise to help the model understand the tool’s purpose. 
> 3. doc strings are important too as they allow the LLM to understand and evaluate the tools properly. 
> 4. Also has pydantic support for custom and limited outputs. 

```python

from langchain.tools import tool

# syntax for the tools. 
@tool('Custome_name_of_the_tool', description='...', args_schema='pydantic_schema_name') 
def search_database(query: str, limit: int = 10) -> str:
    """Search the customer database for records matching the query.

    Args:
        query: Search terms to look for
        limit: Maximum number of results to return
    """
    return f"Found {limit} results for '{query}'"
```

###### Tools Binding to the agents or the skills

Using this tech the LLM can use these tools. LLMs does not run the tool, it does not has the ability to run the tool, it just uses the current runtime to choose and suggest when to run according to their convinience. LLM suggest the tool to call we approve the tool calling. 

```python
from langchain.tools import tool
from langchain_openai import ChatOpenAI

@tool
def multiply(a: int, b: int) -> int:
    """Multiply two numbers."""
    return a * b

llm = ChatOpenAI(model="gpt-4o-mini")

llm_with_tools = llm.bind_tools([multiply])

response = llm_with_tools.invoke("What is 12 times 7?")
print(response)
```

##### Tool Runtime 

Tools are most powerful when they can access runtime information like conversation history, user data, and persistent memory. This section covers how to access and update this information from within your tools. you inject these in the tools arguments. Also this is hidden from the LLMs. 

| Component          | Description                                                                                                                 | Use case                                                    |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **State**          | *Short-term memory* - mutable data that exists for the current conversation (messages, counters, custom fields)             | Access conversation history, track tool call counts         |
| **Context**        | Immutable configuration passed at invocation/start time (user IDs, session info)                                            | Personalize responses based on user identity                |
| **Store**          | Long-term memory - persistent data that survives across conversations                                                       | Save user preferences, maintain knowledge base              |
| **Stream Writer**  | Emit real-time updates during tool execution                                                                                | Show progress for long-running operations                   |
| **Execution Info** | Identity and retry information for the current execution (thread ID, run ID, attempt number)                                | Access thread/run IDs, adjust behavior based on retry state |
| **Server Info**    | Server-specific metadata when running on LangGraph Server (assistant ID, graph ID, authenticated user)                      | Access assistant ID, graph ID, or authenticated user info   |
| **Config**         | [`RunnableConfig`](https://reference.langchain.com/python/langchain-core/runnables/config/RunnableConfig) for the execution | Access callbacks, tags, and metadata                        |
| **Tool Call ID**   | Unique identifier for the current tool invocation                                                                           | Correlate tool calls for logs and model invocations         |

you can manage array of things in the tools using these params. 

--- 
#### Custom tools types:

* @tools decorator 
* `BaseTool` -> class inherit this and make tools.  
* Structured Tool and pydantic models. -> Most secured and best acc possibly. 

cereating async tools using inheriting the BaseTool class. or @tool?  

--- 
---

#### TOOLKIT 
clubbing tools together for reusablity. of related context.  

### Tools calling essentials 
##### 1. Enabling Parallel Tool Calls
First, ensure your model supports parallel tool calling. Most modern LLMs (OpenAI, Anthropic Claude, Gemini) do. When binding tools, explicitly enable the feature:

```python
@tool
def get_weather(location: str) -> str:
    """Get current weather in a location."""
    return f"Weather in {location}: Sunny, 72°F"

@tool  
def get_news(topic: str) -> str:
    """Get news about a topic."""
    return f"News about {topic}: Major breakthrough announced"

model = ChatAnthropic(model="claude-sonnet-4-20250514")
model_with_tools = model.bind_tools(
    [get_weather, get_news],
    parallel_tool_calls=True  # Enable parallel calling
)
```

The `parallel_tool_calls` parameter is key—set it to `True` to allow the model to invoke multiple tools in a single response.

##### 2. Handling Multi-Tool Responses with Annotations

When the model returns multiple `tool_calls`, your execution loop processes all of them before returning to the model. Here's the pattern using annotated injection for state management:

```python
from typing import Annotated, List
from langchain_core.messages import HumanMessage, ToolMessage
from langchain.tools import InjectedToolArg

def execute_tool_calls(response, tools_map: dict):
    """Execute multiple tool calls and return results."""
    tool_messages = []
    
    for tool_call in response.tool_calls:
        tool_name = tool_call["name"]
        tool_args = tool_call["args"]
        tool_call_id = tool_call["id"]
        
        # Execute the specific tool
        if tool_name in tools_map:
            result = tools_map[tool_name].invoke(tool_args)
            tool_messages.append(
                ToolMessage(content=str(result), tool_call_id=tool_call_id)
            )
    
    return tool_messages

# Usage
messages = [HumanMessage("What's the weather in Boston and any news about AI?")]
response = model_with_tools.invoke(messages)

if response.tool_calls:
    tools_map = {"get_weather": get_weather, "get_news": get_news}
    tool_results = execute_tool_calls(response, tools_map)
    messages.extend([response] + tool_results)
    final_response = model_with_tools.invoke(messages)
```

##### 3. Progressive Tool Disclosure with Middleware
For scenarios with many tools (30+), use middleware injection to progressively reveal tools based on conversation context. This prevents context bloat while maintaining access to all tools:
```python
from langchain.agents.middleware import wrap_model_call, ModelRequest, ModelResponse
from langchain.tools import tool
from typing import Callable

ALL_TOOLS = [get_weather, get_news, get_stock_price, search_database]  # Many tools
CORE_TOOLS = {"get_weather", "get_news"}  # Always visible

@tool  
def search_tools(query: str) -> str:
    """Search for specialized tools when needed."""
    matches = [f"- {t.name}: {t.description}" for t in ALL_TOOLS 
               if query.lower() in t.name.lower()]
    return "\n".join(matches) if matches else "No matching tools"

@wrap_model_call
def progressive_tool_disclosure(
    request: ModelRequest,
    handler: Callable[[ModelRequest], ModelResponse],
) -> ModelResponse:
    """Middleware that dynamically injects tools based on conversation."""
    active_tools = set(CORE_TOOLS)
    
    # Check history for search_tool calls to discover more tools
    for msg in request.state.get("messages", []):
        if hasattr(msg, "tool_calls"):
            for tc in msg.tool_calls:
                if tc["name"] == "search_tools":
                    # Add discovered tools to active set
                    query = tc["args"].get("query", "")
                    for tool in ALL_TOOLS:
                        if query.lower() in tool.name.lower():
                            active_tools.add(tool.name)
    
    # Filter tools to only active ones for this call
    filtered_tools = [t for t in request.tools if t.name in active_tools]
    return handler(request.override(tools=filtered_tools))
```

The middleware intercepts each model call, checks what tools have been discovered via `search_tools`, and injects only relevant tools into the current request context.
##### 4. Injected State Pattern for Tool Execution
For tools that need access to shared state or metadata, use annotated injection:

```python
from typing import Annotated, Any
from langgraph.types import InjectedState, InjectedToolCallId

@tool
def process_with_state(
    task: str,
    state: Annotated[Any, InjectedState],
    tool_call_id: Annotated[str, InjectedToolArg],
) -> str:
    """Tool with injected state and tool call ID."""
    # Access conversation state
    previous_results = state.get("results", [])
    
    # The tool_call_id links back to the model's request
    result = f"Processed: {task}, previous: {len(previous_results)}"
    
    return result
```

The `InjectedState` and `InjectedToolArg` parameters are automatically populated by LangChain, giving your tools access to conversation context without manual passing.

##### 5. Complete Working Example
Here's a production-ready implementation using the multi-turn loop pattern:

```python
from langchain_core.tools import tool
from langchain_core.messages import HumanMessage, ToolMessage
from langchain_openai import ChatOpenAI

@tool
def calculate(expression: str) -> str:
    """Calculate a mathematical expression."""
    try:
        return str(eval(expression))
    except:
        return "Calculation error"

@tool
def get_temperature(city: str) -> str:
    """Get temperature in a city."""
    temps = {"boston": "72°F", "nyc": "75°F", "sf": "68°F"}
    return temps.get(city.lower(), "Temperature data unavailable")

# Setup
model = ChatOpenAI(model="gpt-4o-mini")
tools = [calculate, get_temperature]
model_with_tools = model.bind_tools(tools, parallel_tool_calls=True)

# Multi-step conversation with parallel tools
messages = [HumanMessage(
    "Calculate 15 * 23 and get the temperature in Boston and NYC"
)]

MAX_ROUNDS = 3
for _ in range(MAX_ROUNDS):
    response = model_with_tools.invoke(messages)
    messages.append(response)
    
    if not response.tool_calls:
        break
        
    # Execute all tool calls concurrently
    for tool_call in response.tool_calls:
        tool_name = tool_call["name"]
        tool_args = tool_call["args"]
        
        if tool_name == "calculate":
            result = calculate.invoke(tool_args)
        elif tool_name == "get_temperature":
            result = get_temperature.invoke(tool_args)
        else:
            result = f"Unknown tool: {tool_name}"
            
        messages.append(ToolMessage(content=result, tool_call_id=tool_call["id"]))

print(f"Final response: {response.content}")
```

##### Key Takeaways
- **Enable parallel calls** with `parallel_tool_calls=True` when binding tools
- **Use annotated injection** (`InjectedState`, `InjectedToolArg`) for tools needing context
- **Implement middleware** with `@wrap_model_call` for progressive tool disclosure on large tool sets
- **Process all tool_calls** in a loop before returning to the model for true parallelism
- **Always include tool_call_id** in ToolMessage responses to maintain conversation coherence

---
---


### Langchain Middleware 

https://docs.langchain.com/oss/python/langchain/middleware/overview 


Middleware provides a way to more tightly control what happens inside the agent. Middleware is useful for the following:

Simple usage : -> 

```python 
langchain.agents.middleware  # <- all is here
```
```python 
from langchain.agents import create_agent
from langchain.agents.middleware import SummarizationMiddleware, HumanInTheLoopMiddleware

agent = create_agent(
    model="gpt-5.4",
    tools=[...],
    middleware=[
        SummarizationMiddleware(...),
        HumanInTheLoopMiddleware(...)
    ],
)
```

prebuilt:   https://docs.langchain.com/oss/python/langchain/middleware/built-in 
Custome: https://docs.langchain.com/oss/python/langchain/middleware/custom 

![[Pasted image 20260518103130.png]]

--- 
### Langchain Hooks 

https://docs.langchain.com/oss/python/langchain/middleware/custom#node-style-hooks 

##### **Node-style hooks**

hooks as to when the middlwares are executed? before_agent , before_model , after_agent,
after_model.  Run sequentially at specific execution points. Use for *logging, validation, and state updates.*

|Hook|When it runs|
|---|---|
|`before_agent`|Before agent starts (once per invocation)|
|`before_model`|Before each model call|
|`after_model`|After each model response|
|`after_agent`|After agent completes (once per invocation)|
##### **Wrap-style hooks** 
run around each call, giving you control over execution. Intercept *execution and control when the handler is called. Use for retries, caching, and transformation.*

|Hook|When it runs|
|---|---|
|`wrap_model_call`|Around each model call|
|`wrap_tool_call`|Around each tool call|
