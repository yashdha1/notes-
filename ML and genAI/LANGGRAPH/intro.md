LangGraph is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents.

docs, basics, core benifits,  -> https://docs.langchain.com/oss/python/langgraph/overview

--- 

### CORE DATA STRUCTURES in LangGRAPHS
DS -> https://docs.langchain.com/oss/python/langgraph/graph-api 


At its core, LangGraph models agent workflows as graphs. You define the behavior of your agents using three key components. 

## STATE
- [`State`](https://docs.langchain.com/oss/python/langgraph/graph-api#state): A shared data structure that represents the current snapshot of your application. It can be any data type, but is typically defined using a shared state schema.
- The schema of the `State` will be the input schema to all `Nodes` and `Edges` in the graph, and can be either a `TypedDict` or a `Pydantic` model. All `Nodes` will emit updates to the `State` which are then applied using the specified `reducer` function. 
- **SCHEMAS** -> The main documented way to specify the schema of a graph is by using a [`TypedDict`, `Pydantic modls` or the `dataclasses` . `all graph nodes communicate with a single schema

## NODES
- [`Nodes`](https://docs.langchain.com/oss/python/langgraph/graph-api#nodes): Functions that encode the logic of your agents. They receive the current state as input, perform some computation or side-effect, and return an updated state.
	- input_state -> NODE(manipulate state) -> output_state.

* **START** - The [`START`](https://reference.langchain.com/python/langgraph/constants/START) Node is a special node that represents the node that sends user input to the graph. The main purpose for referencing this node is to determine which nodes should be called first. 
* **END** - The `END` Node is a special node that represents a **terminal node**. This node is referenced when you want to denote which edges have no actions after they are done. Remember this is not the end this is just the terminus cause langgraph is not linear. 

> node caching -> https://docs.langchain.com/oss/python/langgraph/graph-api#node-caching  -> caching the states of the expensive nodes inmemory or out of memory we do not care. 


### Edges
- [`Edges`](https://docs.langchain.com/oss/python/langgraph/graph-api#edges): Functions that determine which `Node` to execute next based on the current state. They can be conditional branches or fixed transitions. 
	- Normal Edges: Go directly from one node to the next.
	- Conditional Edges: Call a function to determine which node(s) to go to next.
	- Entry Point: Which node to call first when user input arrives.
	- Conditional Entry Point: Call a function to determine which node(s) to call first when user input arrives.
	
	A node can have multiple outgoing edges. If a node has multiple outgoing edges, **all** of those destination nodes will be executed in parallel as a part of the next superstep.

### STATEGraph
- The [`StateGraph`](https://reference.langchain.com/python/langgraph/graph/state/StateGraph) class is the main graph class to use. This is parameterized by a user defined `State` object.

### Reducers
- Reducers are key to understanding how updates from nodes are applied to the `State`

### Context Schemas 
When creating a graph, you can specify a `context_schema` for runtime context passed to nodes. This is useful for passing information to nodes that is not part of the graph state. For example, you might want to pass dependencies such as model name or a database connection.

### Super Steps 
One synchronized round of parallel node execution plus state merging before continuing to the next round. When multiple nodes update the same state key in one super-step, LangGraph uses **reducers** to merge values. 
	ESSENTIALLY: 
		- A set of nodes that become runnable at the same time
		- Those nodes executing in parallel
		- Their outputs/state updates being merged
		- A synchronization barrier before moving forward
Nodes within the same super-step do not see each other’s updates while running. 


--- 

setup and shits in langasmith -> https://docs.langchain.com/oss/python/langgraph/local-server 

---
### State vs Checkpointer 
Graph State  = "What the graph currently knows"
Checkpointer = "Where that knowledge is saved"

| Feature        | Graph State             | Graph Checkpointer              |
| -------------- | ----------------------- | ------------------------------- |
| What is it?    | Current workflow data   | System that saves workflow data |
| Purpose        | Hold active information | Persist information             |
| Lifetime       | During execution        | Across executions               |
| Changes often? | Yes                     | Stores snapshots                |
| Example        | messages, variables     | MemorySaver, SQLite saver       |