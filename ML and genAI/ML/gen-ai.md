# 🧠 LLM · GenAI · Agentic AI — End-to-End Learning Roadmap

> From absolute beginner → production-ready expert

---

## How to Use This Roadmap

- Follow phases **in order** — each builds on the last
- Each phase has a **goal**, **topics**, **resources**, and a **checkpoint project**
- Time estimates assume 8–12 hrs/week of focused study
- Mark checkpoints ✅ before moving to the next phase

---

## Phase 1 — Machine Learning Foundations (4–6 weeks)

**Goal:** Understand how models learn — the engine under every LLM.

### Topics

- Supervised vs unsupervised learning
- Loss functions, gradient descent, backpropagation
- Overfitting, regularization, train/val/test splits
- Neural networks: perceptrons → MLPs → CNNs → RNNs

### Resources

- [fast.ai Practical Deep Learning for Coders](https://course.fast.ai/) ← best starting point
- [Andrew Ng's ML Specialization (Coursera)](https://www.coursera.org/specializations/machine-learning-introduction)
- _Deep Learning_ (Goodfellow, Bengio, Courville) — reference text

### Tools

- PyTorch (preferred) or TensorFlow
- Jupyter Notebooks / Google Colab

### ✅ Checkpoint

Train an image classifier on CIFAR-10 from scratch using PyTorch. Achieve > 70% accuracy. Explain what each layer does.

---

## Phase 2 — The Transformer & LLM Core (5–7 weeks)

**Goal:** Deeply understand how modern LLMs are built.

### Topics

#### Transformers

- Attention mechanism (self-attention, multi-head attention)
- Positional encoding
- Encoder-decoder vs decoder-only architectures
- Layer norm, residual connections, FFN layers

#### Language Modeling

- Tokenization (BPE, WordPiece, SentencePiece)
- Next-token prediction, causal masking
- Perplexity as a metric

#### Landmark Models (understand architecture, not just names)

- BERT (encoder-only, bidirectional)
- GPT family (decoder-only, autoregressive)
- T5 (encoder-decoder)
- LLaMA, Mistral, Gemma (open-weight models)

### Resources

- [_Attention Is All You Need_](https://arxiv.org/abs/1706.03762) — read this paper
- [The Illustrated Transformer (Jay Alammar)](https://jalammar.github.io/illustrated-transformer/)
- [Andrej Karpathy — Let's Build GPT from Scratch](https://www.youtube.com/watch?v=kCc8FmEb1nY) ← essential
- [Hugging Face NLP Course](https://huggingface.co/learn/nlp-course) — free

### ✅ Checkpoint

Implement a mini-GPT (character-level) from scratch following Karpathy's tutorial. Train it on a text corpus of your choice. Understand every line.

---

## Phase 4 - Fine-Tuning & Customization (5–6 weeks)

**Goal:** Adapt pre-trained LLMs to specific tasks or domains.
### Topics
#### Fine-Tuning Concepts
- When to fine-tune vs prompting (trade-offs)
- Full fine-tuning vs parameter-efficient methods
- Instruction tuning, RLHF (Reinforcement Learning from Human Feedback)
- DPO (Direct Preference Optimization) — simpler RLHF alternative

#### Parameter-Efficient Fine-Tuning (PEFT)
- LoRA (Low-Rank Adaptation) — the dominant method
- QLoRA — LoRA + quantization for consumer GPUs
- Adapters, prefix tuning (know conceptually)

#### Datasets & Training
- Dataset preparation: instruction formats (Alpaca, ChatML, ShareGPT)
- Hugging Face `datasets`, `trl`, `peft` libraries
- Training on Google Colab (free GPU) or RunPod / Lambda Labs

#### Evaluation
- BLEU, ROUGE (traditional NLP metrics)
- LLM-as-judge (use GPT-4/Claude to evaluate outputs)
- Benchmarks: MMLU, HellaSwag, ARC

### Resources
- [Hugging Face PEFT Docs](https://huggingface.co/docs/peft)
- [Axolotl](https://github.com/axolotl-ai-cloud/axolotl) — fine-tuning framework
- [Tim Dettmers — QLoRA Paper](https://arxiv.org/abs/2305.14314)
- [Unsloth](https://github.com/unslothai/unsloth) — fast LoRA fine-tuning

 
### ✅ Checkpoint

Fine-tune a 7B model (e.g., Mistral or LLaMA) using QLoRA on a custom instruction dataset (≥ 500 examples). Evaluate before/after on held-out examples. using Unsloth. 

---

## Phase 5 — RAG & Knowledge Augmentation (4–5 weeks)

**Goal:** Give LLMs access to your data without fine-tuning.
### Topics

#### RAG Architecture (Retrieval-Augmented Generation)

- The core pipeline: Embed → Store → Retrieve → Generate
- Chunking strategies (fixed-size, semantic, recursive)
- Embedding models (text-embedding-3, BGE, E5)
- Vector databases: Pinecone, Weaviate, ChromaDB, pgvector
- Similarity search: cosine similarity, ANN (approximate nearest neighbors)

#### Advanced RAG

- Hybrid search (vector + keyword BM25)
- Re-ranking (cross-encoders, Cohere Rerank)
- HyDE (Hypothetical Document Embedding)
- Parent-child chunking, multi-vector retrieval
- Evaluation: RAGAS framework

#### Other Knowledge Tools

- Structured data: Text-to-SQL
- Web retrieval: Tavily, Brave Search API
- Document parsers: Unstructured, LlamaParse

### Resources

- [LangChain RAG Tutorial](https://python.langchain.com/docs/tutorials/rag/)
- [LlamaIndex Docs](https://docs.llamaindex.ai/)
- [RAGAS — RAG Evaluation](https://ragas.io/)
- [Pinecone Learning Center](https://www.pinecone.io/learn/)

### ✅ Checkpoint

Build a RAG pipeline over a collection of PDFs (e.g., research papers or company docs). Users should be able to ask questions and get cited, accurate answers. Measure retrieval precision.

---

## Phase 6 — Agentic AI & Multi-Agent Systems (6–8 weeks)

**Goal:** Build autonomous AI systems that plan, use tools, and collaborate.

### Topics

#### Agent Fundamentals

- The agent loop: perceive → think → act → observe
- ReAct (Reasoning + Acting) pattern
- Tool use / function calling
- Memory types: in-context, external (RAG), episodic, semantic

#### Tool Use & Function Calling

- Defining tools (JSON schema)
- Parsing and executing tool calls
- Error handling in agentic loops
- OpenAI / Anthropic tool-use APIs

#### Planning & Reasoning

- Chain-of-Thought (CoT), Tree-of-Thought (ToT)
- Plan-and-Execute agents
- Reflection and self-correction loops

#### Multi-Agent Systems

- Orchestrator → specialist agent patterns
- Agent communication and handoffs
- Parallelism: fan-out tasks to multiple agents
- Supervisor patterns

#### Agentic Frameworks

- **LangGraph** — graph-based stateful agents (recommended)
- **CrewAI** — role-based multi-agent teams
- **AutoGen** (Microsoft) — conversational multi-agent
- **Pydantic AI** — production-grade, type-safe agents
- **Claude's Computer Use API** — GUI automation

#### Safety & Reliability

- Prompt injection in agentic systems
- Human-in-the-loop (HITL) patterns
- Guardrails, output validation (Pydantic, Instructor)
- Monitoring agents: LangSmith, LangFuse

### Resources

- [LangGraph Docs & Tutorials](https://langchain-ai.github.io/langgraph/)
- [Anthropic — Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- [Andrew Ng — Agentic AI Patterns](https://www.deeplearning.ai/the-batch/issue-242/)
- [CrewAI Docs](https://docs.crewai.com/)

### ✅ Checkpoint

Build a research agent that: takes a question, searches the web, reads relevant pages, synthesizes findings, and produces a structured report with citations. It should self-correct if a tool fails.

---

## Phase 7 — Production & MLOps for LLMs (4–6 weeks)

**Goal:** Deploy, monitor, and maintain LLM applications at scale.

### Topics

#### Serving & Inference Optimization

- vLLM — high-throughput serving with PagedAttention
- TGI (Text Generation Inference) by Hugging Face
- Batching strategies, throughput vs latency trade-offs
- Speculative decoding, flash attention

#### Deployment

- Containerization: Docker, Kubernetes basics
- Cloud deployment: AWS Bedrock, GCP Vertex AI, Azure OpenAI
- Serverless inference: Modal, Replicate, Beam

#### Observability & Monitoring

- Logging LLM inputs/outputs
- Latency, cost, and error rate dashboards
- Drift detection — when model behavior changes
- Tools: LangSmith, LangFuse, Helicone, Weights & Biases

#### Cost Optimization

- Model routing (use cheap model when possible, escalate to expensive)
- Caching (semantic caching with GPTCache)
- Prompt compression (LLMLingua)

#### Security

- PII detection and redaction
- Prompt injection defenses
- Rate limiting and abuse prevention

### Resources

- [vLLM Docs](https://docs.vllm.ai/)
- [Full Stack LLM Bootcamp (Berkeley)](https://fullstackdeeplearning.com/llm-bootcamp/)
- [LangSmith Docs](https://docs.smith.langchain.com/)
- [Modal — Serverless GPU](https://modal.com/docs)

### ✅ Checkpoint

Deploy your Phase 5 RAG app or Phase 6 agent behind a FastAPI server. Add request logging, latency tracking, and a simple dashboard. Estimate cost per query.

---

## Phase 8 — Specialization & Cutting Edge (ongoing)

**Goal:** Go deep in one area and stay current with the field.

### Pick Your Focus Track

#### Track A — Research & Pretraining

- Read papers: Chinchilla scaling laws, MoE (Mixture of Experts), state space models (Mamba)
- Study pretraining pipelines: data curation, deduplication, tokenizer training
- Resources: [EleutherAI](https://www.eleuther.ai/), [Papers With Code](https://paperswithcode.com/)

#### Track B — Advanced Agents & Reasoning

- Test-time compute scaling (o1/o3/DeepSeek-R1 reasoning models)
- Long-context models and memory architectures
- Formal verification of agent behavior
- Resources: DeepMind, Anthropic, OpenAI research blogs

#### Track C — Applied / Product Engineering

- Multi-modal models (vision, audio, video): GPT-4V, Gemini, LLaVA
- Voice interfaces: Whisper, TTS, real-time audio APIs
- Code generation & AI dev tools: Cursor, GitHub Copilot internals
- Resources: Simon Willison's blog, Latent Space podcast

#### Track D — Safety & Alignment

- Constitutional AI, RLHF deep-dive
- Mechanistic interpretability (Anthropic's circuits work)
- Red-teaming and adversarial robustness
- Resources: [AI Safety Fundamentals](https://aisafetyfundamentals.com/), Anthropic research

### Stay Current

- Follow: [@karpathy](https://x.com/karpathy), [@simonw](https://x.com/simonw), [@swyx](https://x.com/swyx), [@goodside](https://x.com/goodside)
- Podcasts: _Latent Space_, _Practical AI_, _The TWIML AI Podcast_
- Newsletters: _The Batch_ (deeplearning.ai), _Import AI_, _Last Week in AI_
- Papers: arXiv cs.CL, cs.AI — read 2–3 papers/week

---

## Summary Timeline

|Phase|Topic|Duration|
|---|---|---|
|0|Prerequisites (Math + Python)|2–4 weeks|
|1|Machine Learning Foundations|4–6 weeks|
|2|Transformers & LLM Core|5–7 weeks|
|3|Working with LLMs (APIs, Prompting)|4–5 weeks|
|4|Fine-Tuning & Customization|5–6 weeks|
|5|RAG & Knowledge Augmentation|4–5 weeks|
|6|Agentic AI & Multi-Agent Systems|6–8 weeks|
|7|Production & MLOps|4–6 weeks|
|8|Specialization (ongoing)|∞|
|**Total to expertise**||**~9–12 months**|

---

## The Golden Rule

> **Build something at every phase.** Reading and watching alone won't make you an expert. Every concept becomes real only when you've debugged it at 1am.

---

_Last updated: May 2026_