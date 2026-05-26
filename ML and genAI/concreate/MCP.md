mother link -> https://modelcontextprotocol.io/docs/getting-started/intro 
concepts and arch -> https://modelcontextprotocol.io/docs/learn/architecture 

MCP (Model Context Protocol) is an open-source standard for connecting AI applications to external systems.  
its like a protocol of comminication for the AI LLM systems. to communicate with the world. 
Think of MCP like a USB-C port for AI applications. Just as USB-C provides a standardized way to connect electronic devices, MCP provides a standardized way to connect AI applications to external systems.

![](https://mintcdn.com/mcp/bEUxYpZqie0DsluH/images/mcp-simple-diagram.png?fit=max&auto=format&n=bEUxYpZqie0DsluH&q=85&s=35268aa0ad50b8c385913810e7604550)

-> internet access to the LLMs. Figma, 3d graphics generator, etc. All kinds of jargons can be done via these.
-> MCP provides access to an ecosystem of data sources, tools and apps which will enhance capabilities and improve the end-user experience.
-> uses the **JSON-RPC** for the comminication format. JSON-RPC is extremely simple and can be implemented with any JSON serializer.  Uses the HTTP1.  Defines the structure of the data. 
-> For commuication over the MCPs uses http streams, SSE (http 2.0  based), WebSockets. 

| Transport           | Best For                   | Connection Type                     | Key Strength                             | Official Status                                                                                                                                            |
| ------------------- | -------------------------- | ----------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **stdio**           | Local processes, CLI tools | Bidirectional (over pipes)          | Simple, secure, no network overhead      | **Standard**[](https://modelcontextprotocol.io/specification/2025-06-18/basic/transports.md)                                                               |
| **Streamable HTTP** | Web APIs, Serverless       | Bidirectional (HTTP + optional SSE) | Firewall-friendly, load balancer support | **Standard**[](https://modelcontextprotocol.io/specification/2025-11-25/basic/transports?search=Server-Sent+Events+%28SSE%29+-+Deprecated#streamable-http) |
| **WebSocket**       | Real-time web apps         | Full-Duplex, Persistent             | Low latency, true server push            | **Extension** (In review)[](https://github.com/modelcontextprotocol/modelcontextprotocol/issues/1288)                                                      |

-> we bundle messages and instructions into the JSON RPC. and then ship them over the HTTP 1/2, SSE or sockets. 
-> The Model Context Protocol (MCP) is primarily a **stateful protocol** While the underlying [JSON-RPC 2.0](https://www.jsonrpc.org/specification) standard it uses for message encoding is technically stateless, MCP's implementation requires managing a session lifecycle between a client and a server. 

---

https://modelcontextprotocol.io/docs/sdk 

# MCP sdk -> imp
1. https://github.com/modelcontextprotocol/python-sdk#core-concepts 
2. 
---
### python-sdk MCP

```python 
from mcp.server.fastmcp import Context, FastMCP
from mcp.server.session import ServerSession

mcp = FastMCP("My App", lifespan=app_lifespan)

# Access type-safe lifespan context in tools
@mcp.tool()
def query_db(ctx: Context[ServerSession, AppContext]) -> str:
    """Tool that uses initialized resources."""
    db = ctx.request_context.lifespan_context.db
    return db.query()
```

#### core concepts in MCP
1. **Server** 
2. **Resources/GET**-> glorified get request 
3. **Tools/Post, action, PUT** -> Unlike resources, tools are expected to perform computation and have side effects. 
	1. Tools and Resources returns structured outputs: like the langchain shit. 
4. **Prompts** -> Reusable templates are prompts. 
5. 