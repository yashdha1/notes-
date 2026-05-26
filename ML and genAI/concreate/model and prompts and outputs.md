--- 
- MODELS.
- PROMPTS. 
- MESSAGES.
- STRUCRURED OUTPUTS.
- UNSTRUCUTRED OUTPUT.
- MEMORY MANAGEMENT.

--- 

## LANGUAGE MODEL

types : 
1. LLM -> general purpose : text genration, summarization, translations, code gen, creative writing.  
2. Chat Model       -> conversational AI : convo, virtual assistant, Ai tutors. 

#### Embedding models 
... 

--- 
## 1. Prompts and messages

static and dynamic prompts. Messaging in langchains. 

```python 
from langchain.messages import SystemMessage, HumanMessage, AIMessage

messages = [
    SystemMessage("You are a poetry expert"),    # system prompt
    HumanMessage("Write a haiku about spring"),  # current users message
    AIMessage("Cherry blossoms bloom...")        # An [`AIMessage`] represents the output of a model invocation.# also has the history for the tool calls. 
]

response = model.invoke(messages)
```
#### messages give the LLM 
- conversation history
- role separation
- tool/function call support
- memory handling
- multi-turn chats
- better control over behavior

---
#### PromptTemplates  
-> to create dynamic prompts in langchain 

```python 
form langchain_core.prompts import PromptTemplate

PromptTemplate({//prompts})
```

Q) why PromptTemplate and not normal fstrings? 
-> *PromptTemplate* gives us some validations of the prompt dynamically in the runtime.
-> *PromptTemplate* Also has the feature to create dynamic prompts json for reusablity and load them back using the **load_prompt()**  
-> tights coupling with the Langchain ecosystem in the chains and graphs and all. 

--- 

## 2. Structured OUTPUTS in LLMs

### Default parsers in Foundation models 
for the tools inputs, api inputs, Databases connectivity, Agents, 
increases **User_Experience**. 
Always get the JSON Output. *->*  Making non-deterministic outputs deterministic.

for the LLM -> machine Interactions. 

foundation models has the abilty to do the -> 
```

model.with_structured_output()
```


```
.with_structured_output()  -> foundation models has this inbuilt with langchain.
in three primary datatypes : 
	
	1. type dicts. -> structured dicts .
	2. Pydantic.   -> prolly best i reckon. offers type safety and all. 
	3. JSON.       -> idk about this one chief. 
```
 
| Feature                       | Plain `dict` | `TypedDict`                     | Pydantic                  |
| ----------------------------- | ------------ | ------------------------------- | ------------------------- |
| Runtime validation            | ❌            | ❌                               | ✅                         |
| Type coercion (e.g., "25"→25) | ❌            | ❌                               | ✅                         |
| Static type checking (mypy)   | ❌            | ✅                               | ✅ (partial)               |
| JSON serialization            | Manual       | Manual                          | Built-in                  |
| IDE autocomplete for keys     | ❌            | ✅                               | ✅                         |
| Performance                   | Fastest      | Fast                            | Good (with v2)            |
| Best for                      | Scripts      | Internal dicts with known shape | APIs, configs, user input |

Decision tree : 
```
					---> DECISION_MAKING <---	
Is your data coming from an external/untrusted source?
  ├─ YES → Use Pydantic (validation + coercion)
  └─ NO  → Is the data a simple dict with known keys?
        ├─ YES → Do you need static type checking only? → TypedDict
        └─ NO  → Is it extremely trivial? → Plain dict + manual code -> JSON
```

--- 
###  Output Parsers [REDACTED]

used only in the langchain_core and the now we use the  
```python 
from langchain_core.output_parsers import StrOutputParser
```

#### All Types of Parsers

| Parser Type                          | Primary Use Case                                               | Output Format                  | Key Advantage                                                                                                                                                                                                                                                                     |
| ------------------------------------ | -------------------------------------------------------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **`StrOutputParser`**                | Simple chat responses, summarization                           | String                         | Simplest, zero configuration[](https://github.com/jsonusuman351/langchain_output_parser)                                                                                                                                                                                          |
| **`JsonOutputParser`**               | Extracting flexible structured data                            | Python `dict` (JSON)           | Good for dynamic schemas, easy API integration[](https://github.com/tahirkorma/langchain-output-parsers)[](https://github.com/jsonusuman351/langchain_output_parser)                                                                                                              |
| **`PydanticOutputParser`**           | Production apps requiring strict data validation               | `Pydantic` Model Object        | Strong validation, type safety, auto-completion[](https://github.com/tahirkorma/langchain-output-parsers)[](https://github.com/jsonusuman351/langchain_output_parser)[](https://lagnchain.readthedocs.io/en/stable/_sources/modules/prompts/output_parsers/getting_started.ipynb) |
| **`StructuredOutputParser`**         | Defining a fixed set of string fields                          | Python `dict` (JSON)           | Explicit, uses `ResponseSchema` for clarity[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://github.com/jsonusuman351/langchain_output_parser)                                                                      |
| **`CommaSeparatedListOutputParser`** | Parsing simple lists (e.g., keywords, items)                   | `List[str]`                    | Perfect for straightforward list extraction[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://blog.csdn.net/u010784605/article/details/154604907)                                                                    |
| **`DatetimeOutputParser`**           | Parsing dates and times                                        | `datetime` object              | Converts LLM outputs to a usable date/time format[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://blog.csdn.net/u010784605/article/details/154604907)                                                              |
| **`EnumOutputParser`**               | Classifying text into a specific set of categories             | `Enum` member                  | Ensures output is one of a predefined list of values[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://blog.csdn.net/u010784605/article/details/154604907)                                                           |
| **`OutputFixingParser`**             | Recovering from minor parsing errors (e.g., missing field)     | Depends on the parser it wraps | Automatically attempts to fix formatting errors[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://python.langchain.com/v0.1/docs/modules/model_io/output_parsers/types/retry/)                                       |
| **`RetryOutputParser`**              | Recovering from major parsing errors (e.g., incomplete output) | Depends on the parser it wraps | More powerful fix by re-prompting with original input[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)[](https://python.langchain.com/v0.1/docs/modules/model_io/output_parsers/types/retry/)                                 |
| **`PandasDataFrameOutputParser`**    | Performing operations on data in DataFrames                    | Pandas DataFrame               | Useful for data manipulation and analysis tasks[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)                                                                                                                              |
| **`XMLOutputParser`**                | Generating or parsing XML data                                 | `dict` of tags                 | Best when working with models strong at XML (e.g., Anthropic)[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)                                                                                                                |
| **`YAMLOutputParser`**               | Returning data in YAML format based on a Pydantic model        | YAML string                    | Alternative to JSON for configurations or specific APIs[](https://www.educative.io/module/page/NxqvGMSPRjvEr6PxW/10370001/5152389308219392/4877331549519872)                                                                                                                      |

> **Prefer Native Structured Outputs**: If you're using a model like OpenAI or Anthropic that supports a native `with_structured_output` method, use it! It's more reliable than prompt-based parsing[](https://github.com/dhar174/tiny_village/issues/162)[](https://js.langchain.ac.cn/docs/troubleshooting/errors/OUTPUT_PARSING_FAILURE/)[](https://docs.langchain.com/oss/javascript/langchain/errors/OUTPUT_PARSING_FAILURE). You can fall back to `PydanticOutputParser` for other models.

--- 

### STRUCTURED_OUTPUT

Modern langchain approach for the output. 

https://docs.langchain.com/oss/python/langchain/structured-output#custom-tool-message-content 














 