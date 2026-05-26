## 1. Central Tendencies of data
mean                -> avg() 
median             -> .sort()[n//2] 
mode                -> Coutner(arr).sort(lambda x: x[1] , reverse=True) -> (popular one)

geometric mean -> nth root of product of values 
harmonic  mean -> reciprocal of average of reciprocals. 
weighted mean   -> Some vals has more vals than others .

range + midrange  -> general tendency of data

> # remember
> - **Additive data → Mean**
> - **Ordered middle → Median**
> - **Most common → Mode**
> - **Growth → Geometric mean**
> - **Rates → Harmonic mean**
> - **Importance matters → Weighted mean**

--- 
## 2. data dispersions 
variance                -> How far the data points are from the mean, but in **squared form**.
		Population variance and sample variance. 
		-  High variability → data is very different from each other
		- Low variability → data is similar and clustered

Standard Deviation -> avaerage distance of  the from the means. underroot of variance
			low -> tight clustering and low variance
			high -> large diffrences between the vals. 
	

coefficient of variance -> 


---
---
### cost functions : 

^ calculate how far of the predected vals are from the actual vals. 
> Cost function(J)=n1​∑ni​(yi​^​−yi​)2 

### Gradient Descent  
Optimisers -> go towards the proper vals. in the ML. 
- Start with random values for slope and intercept.
- Calculate the error between predicted and actual values.
- Find how much each parameter contributes to the error (gradient).
- Update the parameters in the direction that reduces the error.
- Repeat until the error is as small as possible.

## probablity and statics 

Distributions 
Basic Probability
Covariance, Correlation and causations
bayes theorem 

--- 

## learn_path_in_LANGCHAIN_ROADMAP : 

**Stage 3: Advanced Workflows & Orchestration**
*   **What to Learn**: Use **LangGraph** for complex, stateful, multi-actor applications. This replaces simple chains for agent-like behaviors.
*   **Must Know**: `StateGraph`, nodes and edges, conditional routing. **Key Goal**: Build a customer support flow that routes queries to different RAG pipelines.

**Stage 4: Intelligent Agents & Production**
*   **What to Learn**: Build agents that reason, use tools, and act. Learn monitoring with **LangSmith** and deployment best practices.
*   **Must Know**: `create_openai_tools_agent`, AgentExecutor, tool definitions, tracing, and performance optimization. **Key Goal**: Build an agent that can search the web and perform calculations.

### Tips for Your Journey

*   **Project-Driven Learning**: Don't just read. Each stage should result in a mini-project. The "Key Goal" in the roadmap is your assignment.
*   **Level Up Your RAG**: Start with basic retrieval, then add advanced strategies like **hybrid search (BM25 + vector)** and **re-ranking** to dramatically improve answer quality.
*   **Go Deep on Agents**: Focus on the **ReAct (Reason + Act) framework**. This is the core logic that allows an agent to think, decide on a tool, and observe the result.
*   **Learn Production Skills**: A pro knows how to deploy. Learn to use **LangSmith for tracing and evaluation**, and containerize your app with **Docker**.

Following this path will take you from understanding isolated components to designing complex, intelligent systems.

Hope this roadmap gives you a clear path forward. Which project idea from the roadmap excites you the most?


---
