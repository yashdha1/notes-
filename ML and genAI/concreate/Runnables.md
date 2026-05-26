
what runnables enables? 

> lego blocks of the langchain world.

**Runnables are the universal interface in modern LangChain** that:
1. Standardize how all components are invoked (sync, async, batch, stream).
2. Enable easy composition using the `|` pipe operator.
3. Automatically provide streaming, batching, and parallel execution.
4. Support advanced patterns like retries, configuration, and schema validation.

> modern LangChain (v0.1+ and v1), **everything is a Runnable**

> in Layman - _"If you implement these 6 methods (invoke, stream, batch, plus their async versions), you can play nicely with everything else in the LangChain ecosystem."_ They're a **shared vocabulary** that lets completely different components understand each other. 

> Runnables enable easier working with all the langchain components ._"This is a Runnable. It has invoke(), stream(), and batch(). I don't care what's inside."_ Thry provide a level of abstraction over the complexions of the langchain and manual connection of the components .

--- 

> ie. No need to be the plumber, be the **developer** 

```python
# PLUMBING CODE (without Runnables)
def connect(a, b):
    output = a.do_something_weird()
    if isinstance(output, dict):
        output = output["value"]
    return b.another_weird_method(output)

# LOGIC CODE (with Runnables)
result = a | b | c  # Just works
```

---

#### Chains in Langchain 

![[Pasted image 20260519061948.png]]

--- 
--- 

## TYPES OF RUNNABLES 

#### TASK SPECIFIC 
- previos versions normal components converted to the runnables in the newere versions of the langchain. 
![[Pasted image 20260519055919.png]]
-- Helps orchestration of the Components Runnables normally.

#### Runnable primitives 
- **Runnable sequence** : is just a Syntactic sugar for the '|' operator. 
- **Runnable parallel**     : must requires the dictionary Defination.
- **Runnable Lambda**    : Convert normal python syntax to the runnable and thus can be used within the chains.
	> In your code, `beutify_response` works **without** `RunnableLambda` because LangChain automatically converts a plain Python callable into a runnable when you use the `|` operator in an LCEL (LangChain Expression Language) chain. 
	
	but we should use the runnable lambda for the inbuild more we get of the runnable classes. - `.invoke()`, `.batch()`, `.stream()`, `.ainvoke()` , `.with_config()` and `.with_retry()`.
- **Runnable Branch**  -> useful in the conditional chaining. 

![[Pasted image 20260519072428.png]]

* LCEL -> Declarative defination to langchain languages. 
--- 

