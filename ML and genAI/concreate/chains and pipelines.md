https://milvus.io/ai-quick-reference/what-are-chains-in-langchain-and-how-do-they-function 

> chain = step1 | step2 | step3
> 	 works with both. 
	chain.invoke()
	chain.stream()

--- 
* Save visual for the chain
```python 
g = chain.get_graph()
png_data = g.draw_mermaid_png()
with open("chain_graph1.png", "wb") as f:
    f.write(png_data)
```

--- 
* ascii visual for the chain
```python 
print(chain.get_graph().print_ascii()
```
--- 

#### PARALLEL CHAINS

We need **runnable_parallel** for this : -> EXECUTES chains parallelly  on multiple CPUs. 
the prallel chains works literally runs in the parallel. 

```python 
from langchain_core.prompts import  PromptTemplate
from langchain_google_genai import ChatGoogleGenerativeAI
from langchain_core.output_parsers import StrOutputParser
from langchain_core.runnables import RunnableParallel
from config.conf import settings

def create_gemini_model() -> ChatGoogleGenerativeAI:
    gemini_agent = ChatGoogleGenerativeAI(
        model=settings.gemini_model,
        temperature=settings.gemini_temperature,
        api_key=settings.gemini_api_key
    )
    return gemini_agent

prompt1 = PromptTemplate(
    template=(
        "Generate top concise topics to learn on this subject: {sub}\n"
    ),
    input_variables=["sub"]
)
prompt2 = PromptTemplate(
    template=(
        "Generate top resource BOOKS to learn on these topic :  {sub}\n"
        ),
    input_variables=["sub"],
)
prompt3 = PromptTemplate(
    template=(
        "MERGE THE INCOMMING subs and resources, create a roadmap on this information comming from the sources.\n "
        "here are the subject on the topic : {subjects}\n"
        "here are the resources on the topic : {resources}\n"
    ),
    input_variables=["subjects", "resources"],
)

parser = StrOutputParser()

def beutify_response(res: str) -> str:
    return res.replace("\n", "\n- ")


TOPIC = "MACHINE LEARNING"

# Parallel chains
gemini_model = create_gemini_model()

# chaining_explanation
chain1 = prompt1 | gemini_model | parser
chain2 = prompt2 | gemini_model | parser

# needs the dict
parallel_chain = RunnableParallel({  
    "subjects": chain1,
    "resources": chain2
})
# on merge pipeline
merge = prompt3 | gemini_model | parser

final_chain = parallel_chain | merge | beutify_response
res = final_chain.invoke({"sub": TOPIC})
print(res)

g = final_chain.get_graph()
png_data = g.draw_mermaid_png()
with open("chain_graph1.png", "wb") as f:
        f.write(png_data)

```

![C:\Users\YashDhadod\Desktop\lngchain\starter\chain_graph1.png](file:///c%3A/Users/YashDhadod/Desktop/lngchain/starter/chain_graph1.png)

--- 

#### CONDITIONAL CHAINs 

using the Runnables again. 