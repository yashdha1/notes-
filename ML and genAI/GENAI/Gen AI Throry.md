

---
https://www.promptingguide.ai/  
---

#### 1. **Fine tuning**  
-> Taking raw model and fine tune it to your use cases 
- ****Select Base Model:**** Choose a pre-trained model based on our task and compute budget.
- ****Choose Fine-Tuning Method:**** Select the most appropriate method like [****I****nstruction Fine-Tuning](https://www.geeksforgeeks.org/artificial-intelligence/instruction-tuning-for-large-language-models/), [Supervised Fine-Tuning](https://www.geeksforgeeks.org/artificial-intelligence/supervised-fine-tuning-sft-for-llms/), [PEFT](https://www.geeksforgeeks.org/artificial-intelligence/what-is-parameter-efficient-fine-tuning-peft/), [LoRA](https://www.geeksforgeeks.org/deep-learning/what-is-low-rank-adaptation-lora/), [QLoRA](https://www.geeksforgeeks.org/deep-learning/what-is-qlora-quantized-low-rank-adapter/), etc based on the task and dataset.
- ****Prepare Dataset:**** Structure our data for task-specific training, ensuring the format matches the model's requirements.
- ****Training:**** Use frameworks like TensorFlow, PyTorch or high-level libraries like [Transformers](https://www.geeksforgeeks.org/machine-learning/getting-started-with-transformers/) to fine-tune the model.
- ****Evaluate and Iterate:**** Test the model, refine it as necessary and re-train to improve performance.
	1.1. Popular types of **Fine Tuning**  techniques -> Supervised, PEFT (parameter efficient fine tuning) , Reinforcement and RL human feedback (RLHF). 
	
	1.2. RAG vs FineTuning

#### 2. Temperature and Quantization
#### 3. Transformers and Attention 
#### 4. Overfitting underfitting hallucinations

#### 5. Types of Model 
###### User models (prompts, RAG, AI Agents, Vector Databases)  -> SWE shit
	Using Models.
	Frameworks like -> LangChain, LangGraph.
	Prompt Engineering.
	RAG. VectorDB. neo4J. 
	Fine Tuning the responses. 
	Agentic Development. Tools. Hooks. MCPs. 
	LLMOps. Deployment and shi. 
	
###### Builder Models (Fine Tuning)  -> research Wali shit
	Transformers and types 
	Feature Engineering and evaluations 
	Optimization
	Inference
	Fine tune
	Model Deployment

#### 6. system prompt -> defines agents role and the behavior. 

#### 7. Tools and skills and MCP 

- tools : A **tool** is an external function or system the model can call. Does not reason (obv), has deterministic I/O since its just a rule based system. 
	- perform actions. 
	-  LLMs job is to call decide the use of tools and how to use the results. 
	- ex : DB access, api, system handelling api (todo, file readers capablities and etc)
	
- skills (mini-agent ts) : A **skill** is a reusable intelligent behavior. Its just a fucking md file.
	- solve task with array of reason and tools.  **Prompt + logic + tools**
	- contains -> prompts, reasoning steps, context, memory, tools usage and rights, guardrails decision logics. 
	- its a workflow or capability package. 


##### exe 1 : books something, Suppose you build an AI travel assistant. 

TOOLS:  a/c/as -> function calling / MCP servers / APIs 
- Flight API
- Hotel API
- Maps API
- Currency converter
- Weather API
These are raw capabilities. We have wither prebuild or we build this. 

Skill: one skill -> “Plan a 5-day vacation”

The skill may: set of **Prompt + logic + tools** 
1. Ask user preferences
2. Search flights
3. Compare hotels
4. Check weather
5. Optimize itinerary
6. Generate budget
7. Create final travel plan

The skill orchestrates multiple tools plus reasoning. 
also skills can be stateful. 

In langchain the skills are implemented as the **chains**, **reusable agent workflows and custom prompt pipelines**. To do some complex set of tasks. 

#### 8. Agents -> Brain.
An **AI Agent** is an advanced software system that uses a large language model (LLM) as its core reasoning engine to pursue complex goals and execute tasks on your behalf with a high degree of autonomy. **Goal Oriented.** isko bas goal complete krna hai kuch bhi karke. 

**Orchestration Engine** : Manages the entire ffucking thing. 
Agent (Raw reason) -> Planning module (breaks complex reson into actionable steps) based on domains or diffrent skills usage -> array of tools
**paradigm -> ReAct -> reason then act.** 

There are further types of agent. 
- **Simple Reflex Agents**:    Almost a skill. limited. 
- **Goal-Based Agents**:       goal oriented. 
- **Utility-Based Agents**:     best reasoned action oriented. 
- **Multi-Agent Systems**:    combo

**again just a fucking md file.** or (yaml in some contexts) 🤷 

#### 9. Agents and subagents

#### 10. langsmith
	tracing capablities for the langchain workflows.

#### 11. 