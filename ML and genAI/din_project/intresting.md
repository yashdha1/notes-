1. Tools registeryusing custom decorators

```python
from collections.abc import Callable
from typing import Any
from langchain_core.tools import BaseTool, tool

registry: dict[str, BaseTool] = {}

def register_tool(func: Callable | None = None, **kwargs: Any) -> Callable:
    """
	    Decorator that wraps a function with langchain's ``@tool`` and
	    registers it in the global registry keyed by function name.
	    Supports both ``@register_tool`` and ``@register_tool()`` usage.
    """
    def decorator(f: Callable) -> BaseTool:
        wrapped = tool(f, **kwargs)
        registry[wrapped.name] = wrapped
        return wrapped
    return decorator(func) if func else decorator

def get_tools(tool_names: list[str]) -> list[BaseTool]:
    return [registry[name] for name in tool_names if name in registry]
```

--- 

2. 