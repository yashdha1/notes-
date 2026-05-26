Defines all the operations in the langchain that we can do :  

1. Tools definations and format. 

```python 
from langchain.tools import tool

@tool 
 def tool_name(params) -> output_format: 
	...
```

--- 
2. Memory Mounting to the agents.  
--> for the fast testing and prototyping purposes. 
in production we mount a PG server to this bitch. 

```python
from langgraph.checkpoint.memory import InMemorySaver
checkpointer = InMemorySaver()
```

--- 
3. output streaming: 
	   rather than sending a single response, most of the model api provide the streaming of the llm outputs. So that we get the procedural outputs. 
	- .stream() -> stream of output 
	- .invoke() -> give the output in the single block of the output. 

```python 
for chunk in model.stream("Why do parrots have colorful feathers?"):
    print(chunk.text, end="|", flush=True) # returns the ans in chunks. 
```

--- 
4. tools definations and binding 
```python
from langchain.tools import tool

@tool
def get_weather(location: str) -> str:
    """Get the weather at a location."""
    return f"It's sunny in {location}."

model_with_tools = model.bind_tools([get_weather]) # bind tools to specific models 

response = model_with_tools.invoke("What's the weather like in Boston?")
for tool_call in response.tool_calls:
    # View tool calls made by the model
    print(f"Tool: {tool_call['name']}")
    print(f"Args: {tool_call['args']}")
```

--- 
5. model.profile -> model_attributes
6. Prompt caching -> 
7. 