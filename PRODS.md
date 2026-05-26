
SANCHITS PROJECT :  
-> An automated, config-driven intelligence digest system that curates and delivers personalized content from multiple sources using LLMs but we will be sending the emails to the smtp servers. 
-> [GitHub - Aasrith-JR/AI-Digest · GitHub](https://github.com/Aasrith-JR/AI-Digest) <- Refer
-> this requires the scrappers I hate that.


LLM based project I really liked. 

**kbase** -> build for reserchers. 

project starting. add docs. assist in building the kb for the project. can be a fork of the previos projects. 

frontend streamlit. 

lets keep it microservice. User. Project service. Notification DB. 
	 flow => create project, add people, actors on the project levels. 
	 features - kb of the entire projects to to build upon these. can be the pdfs, MDs, and all the docs are available for the team. 

FR : 
	-> managing research projects at scale becomes tedious. 
	-> divide tasks. build a KB for the teams. semantic and the FTS search in the membership projects.  
		lets talk genai in this : 
			1. embedding model of the docs and a kb lake building. 
			2. have agents work in the browser over the project. To find the related docs too. 
			3. RAG to see the users uplaoded docs and reiterate over them. 
			4. query based search. semantic + FTS. 
			5. ranking system for the docs over relevence like. LLM based as well as the user approved.
			6. TOOLS : 
				- fetch relevent docs from the internet. To the current researh scope. and suggest and if approved embedd and save in the history as well. 
				- 

tech 

A TUI-based research KB is actually a **killer idea** for researchers. Here's why it works really well and what it would look like:A TUI is honestly a **great fit** for researchers. Here's why and how:

---

## Why TUI works perfectly here

Researchers already live in terminals. They're running jupyter, ssh-ing into clusters, using git — a polished TUI fits the workflow rather than interrupting it. It also means: no browser overhead, keyboard-first speed, scriptable and pipeable, and it can run over SSH on a remote server where the data already lives.

---

## Tech: **Textual** (Python)

[Textual](https://textual.textualize.io/) by Textualize is the right call here — not curses, not urwid. It gives you:

- CSS-like layout system with panels and docks
- Reactive state management (like React, but terminal)
- Rich widgets out of the box (DataTable, Tree, Input, Markdown renderer)
- Async-native — critical for streaming LLM responses and live agent status
- Mouse support, dark/light themes, works in any modern terminal

---

## Layout (what you're seeing above)

Four-panel design:

**Left dock** — workspace nav tree, project switcher, tool toggles. Always visible, keyboard-navigable.

**Center main** — context-switches per mode: search results, document viewer, RAG query+response, knowledge graph, team view. This is where most of the work happens.

**Right dock** — live agent status, doc metadata, and the agent's suggestions queue ("found this paper, approve to embed?").

**Bottom bars** — status (qdrant connection, model, bandwidth) + keybind hints.

---

## What this unlocks architecturally

Since you're going TUI + microservices, the Streamlit idea drops entirely. Instead:

```
kbase-tui (Textual client)
    ↕ websocket / REST
kbase-api (FastAPI gateway)
    ├── search-service (Qdrant + BM25)
    ├── ingest-service (chunking, embedding, async)
    ├── agent-service (LangGraph agents, tool runner)
    ├── rag-service (RAG pipeline, contradiction detector)
    └── graph-service (Neo4j citation + entity graph)
```

The TUI connects to the API gateway. Agent status streams over websocket so the right panel updates live. All microservices stay the same — the TUI is just a different client layer.

---
![[Pasted image 20260517001520.png]]
## TUI-native features to add

- **Fuzzy finder panel** (`^F`) — fzf-style instant filter across all docs, like Telescope in neovim
- **Inline markdown/PDF preview** — render paper abstracts and MD files right in the center panel using Textual's Markdown widget
- **Split view** — two docs side by side for comparison (researchers do this constantly)
- **Agent log stream** — tail agent activity in real time, like `docker logs -f` but for your KB agents
- **Vim-style keybindings mode** — `j/k` navigation, `/` to search, `space` to approve a suggestion
- **Pipe-friendly CLI mode** — `kbase search "attention mechanisms"` returns JSON, so it's scriptable in shell pipelines
- **Offline mode** — queue uploads/searches locally, sync when the API is reachable

---


## Want me to scaffold the project?

I can generate:

- Full Textual app skeleton with the four-panel layout
- FastAPI gateway + service stubs
- Docker Compose for local dev (Qdrant, Neo4j, Postgres, Redis)
- The ingest pipeline (PDF → chunks → embeddings → Qdrant)
- Agent service with the arxiv + web fetch tools

Which piece do you want to start with? 

---

references : 
1. https://machinelearningmastery.com/a-practical-guide-to-building-local-rag-applications-with-langchain/ 
2. 