# LocalFlow — LangChain / AutoGen / Ollama Experiments 🔬

**Purpose:** This repository contains a collection of exploratory notebooks and small utilities that demonstrate working with language-model-based tools and agents (LangChain, AutoGen, Ollama), document ingestion and embeddings (Chroma, OllamaEmbeddings), browser automation (Playwright), and automated log analysis. The goal is to keep runnable examples and short, focused notebooks so a reader can understand the intent and reproduce experiments without diving deeply into the source code.

---

## Quick start ✅

1. Install Python (3.8+ recommended). Some notebooks reference Python 3.13 due to package compatibility with `unstructured`/`pdfminer.six`.
2. Create and activate a virtual environment:

```bash
python -m venv .venv
source .venv/bin/activate  # Linux/macOS
```

3. Install common packages used across notebooks (install only what you need):

```bash
pip install -U pip
pip install langchain langchain-ollama langchain-community langchain-classic langchain-core
pip install chromadb langchain-chroma playwright pyautogen autogen python-dotenv
pip install unstructured pdfminer.six>=20231228 pi-heif beautifulsoup4 nest_asyncio lxml
```

4. If you use Playwright: run

```bash
playwright install
```

5. If you use Ollama locally: install and run Ollama, and pull required models:

```bash
# Start ollama server (follow official install instructions first)
ollama serve
# Pull a model used in notebooks (example)
ollama pull gpt-oss:20b-cloud
ollama pull gemma3:1b
```

6. Create a `.env` file for any OpenAI-managed keys (some notebooks expect `OPENAI_API_KEY2`):

```
OPENAI_API_KEY2=sk-...
```

7. Run the notebooks in order or open the ones you are interested in using Jupyter / VS Code.

---

## Project layout (what each folder contains) 📁

- `myapp/` — small example app folder that currently includes an Alembic configuration (`alembic.ini`, `alembic/`) and a sample SQLite file `autogen04202.db`.
    - `alembic/versions/c73813d...` shows a placeholder migration.
- `Section1_installation/` — quick installation notes and a minimal test notebook.
- `Section2_langchain_basics/` — starter LangChain examples (instantiating `ChatOllama`, prompt templates, streaming outputs).
- `Section3_WorkingwithEXTDocs/` — loading external documents (PDFs) using `UnstructuredPDFLoader` and how to handle library compatibility.
- `Section4 _ WorkingwithEmbeddings/` — splitting documents, creating embeddings (OllamaEmbeddings), storing in Chroma vector DB, doing RetrievalQA.
- `Section5_WorkingwithAIagentsandtools/` — example of building agents and tools (simple math tools, using Wikipedia tool, Zero-shot agent patterns).
- `Section6_playwright_toolkit/` — Playwright toolkit usage, browser toolkits, how to create and use `PlayWrightBrowserToolkit` with LangChain agents.
- `Section7_LogReaderAgent/` — tool-wrapped `summarize_logs()` example and an agent that can call it; shows reading logs and creating a small QA DB.
- `Section8_BDDtestcaseagent/` — using a local LLM to generate test cases from user stories; sample prompt templates + agent usage.
- `Section9_AutoGenMulti-AgentSystems/` — AutoGen-based multi-agent examples (AssistantAgent, UserProxyAgent); includes an extended explanation and troubleshooting notes.
- `Section10_Autogenforlogreader/` — a multi-agent AutoGen example that reads logs and uses GroupChat/GroupChatManager to coordinate a LogAnalyst and a Proxy.

---

## Highlights and important decisions 💡

- Ollama (local) — Why: Running models locally with Ollama gives faster iteration, lower latency, and keeps proprietary or sensitive data on-premise. It also reduces API costs compared to cloud-hosted models.
  - Note: when using AutoGen with an Ollama-compatible API, some examples required `base_url` set to `http://localhost:11434/v1` and passing `llm_config` into agent constructors to match OpenAI-style configuration conventions.

- Embeddings (Ollama / nomic) + Chroma — Why: Local, lightweight embeddings (via `OllamaEmbeddings` or `nomic-embed-text`) combined with a local vector store (Chroma) enable fast, private semantic search without external dependencies. Chroma is easy to persist locally which avoids re-embedding on every run and speeds up experiments.

- Document ingestion & chunking — Why: `UnstructuredPDFLoader` handles a wide variety of PDF structures. Chunking (e.g. `chunk_size=1000`, `chunk_overlap=200`) balances context (enough text for semantic understanding) and performance (smaller chunks fit LLM context windows and are cheaper to embed).

- LangChain / AutoGen — Why: LangChain provides composable building blocks (LLM wrappers, prompt templates, retrievers) making rapid prototyping of LLM applications straightforward. AutoGen helps orchestrate multi-agent workflows and role separation when tasks require structured collaboration.

---

## Section-by-section summary (what we did & how to reproduce) 🔍

### Section 1 — Installation
- Purpose: confirm base environment works and record key commands.
- Reproduce: Create venv, install core packages, test the minimal Python cells in the notebook.
- Why: Reproducible development environments reduce "works on my machine" issues. Documenting exact install steps and Python/package compatibility helps new contributors replicate experiments and prevents dependency regressions.

### Section 2 — LangChain basics
- Purpose: show how to create a `ChatOllama` client, call it synchronously and stream responses, and experiment with `ChatPromptTemplate` chaining.
- Key notes: `ChatOllama` config: set `base_url`, `model`, `temperature` as needed. Use `.stream(...)` to stream chunks.
- Why: LangChain abstracts model calls and prompt management, letting us focus on prompts and flow design. Using `ChatOllama` demonstrates local-first development with predictable latency and simple configuration for experimentation.

### Section 3 — Working with external documents
- Purpose: load PDFs using Unstructured-based loaders and prepare documents for chunking.
- Important: `unstructured` and `pdfminer.six` versions matter; notebooks document the versions that worked.
- Reproduce: place PDFs into the `Section3_WorkingwithEXTDocs/Docs` folder and run loader/chunker cells.
- Why: Real documents come in many formats. `unstructured` with `pdfminer.six` supports more file layouts and edge cases than simpler parsers, so we accept the added maintenance complexity in exchange for broader coverage and more reliable text extraction.

### Section 4 — Embeddings & Retrieval
- Purpose: chunk documents, compute embeddings, store them in a persisted Chroma DB (`./chroma_db`), and run RetrievalQA using local LLM.
- How-to: use `RecursiveCharacterTextSplitter` (example: chunk_size=1000, chunk_overlap=200) then `OllamaEmbeddings` + `Chroma.from_documents(...)`.
- Reproduce: run the notebook; the DB will be persisted in `chroma_db` so re-runs can skip re-embedding.
- Why: Embeddings convert text into semantically meaningful vectors that are efficient to search. Persisting a vector DB is crucial for repeatable experiments and speeds up iterative development by avoiding unnecessary recomputation.

### Section 5 — Agents & Tools
- Purpose: demonstrate agent patterns using `langchain_classic` or `langchain` agent factories, using `load_tools` (e.g., `wikipedia`) and custom `@tool` functions.
- Example: small math tools wrapped with `@tool`, then an agent is created with those tools, invoked, and results printed.
- Why: Tools let deterministic code handle tasks that are unambiguous or require external data (e.g., web lookups). This makes LLM outputs more reliable, auditable, and safe by constraining where the model can act.

### Section 6 — Playwright integration
- Purpose: show how to create a browser toolkit and provide browser-based tools to LangChain agents for web browsing tasks.
- Required steps: `pip install playwright` and `playwright install` to fetch browser binaries. Use `create_async_playwright_browser` to build the toolkit and bind it to your LLM.
- Gotcha: Jupyter’s event loop on some platforms (Windows event loop type) breaks Playwright—`nest_asyncio.apply()` helps in notebooks.
- Why: Playwright is robust, cross-browser, and automates real browser interactions, which is necessary for agents that must inspect complex web pages, click elements, or extract structured data reliably.

### Section 7 — Log Reader Agent
- Purpose: implement a `summarize_logs()` tool to read log files, chunk them, embed them in a Chroma DB, and build an agent that can answer questions about logs.
- Important implementation note: ensure the DB location is checked (`if not os.path.exists(db_path)`) and avoid re-embedding every run.
- Why: Log data is large and noisy—semantic chunking + embeddings enable targeted searches and summaries. Persisting embeddings avoids repeated work and provides reproducible analytics.

### Section 8 — BDD Test Case Agent
- Purpose: generate test cases from user stories using an LLM and a custom `generate_TestCase` tool (PromptTemplate-based).
- How-to: create a `@tool` that formats a prompt with the user story, then invoke a small agent calling that tool.
- Why: LLMs are well-suited for producing human-readable test specifications quickly. Wrapping generation in a tool ensures consistent structure (e.g., Gherkin) and lets us validate or run post-processing steps programmatically.

### Section 9 — AutoGen multi-agent tests
- Purpose: create `AssistantAgent` and `UserProxyAgent` to show multi-agent orchestration for tasks like generating test cases.
- Key lessons:
  - Pass `llm_config` into `AssistantAgent` (missing config yields blank outputs).
  - For Ollama compatibility, `base_url` often requires `/v1` when used with OpenAI-style clients.
- Why: Multi-agent frameworks like AutoGen make it easier to model role-based workflows (manager, assistant, proxy), scale tasks across agents, and reproduce collaboration patterns that mimic human team interactions.

### Section 10 — AutoGen Log Reader
- Purpose: an end-to-end AutoGen example: a proxy supplies logs and a LogAnalyst agent analyzes them via `GroupChat` coordinated by `GroupChatManager`.
- Important corrections documented in the notebook: change `logs = []` to `logs = ""` or use `"\n".join(...)` to avoid splitting strings into characters when accumulating into lists.
- Why: GroupChat/GroupChatManager provides structure, turn-taking, and moderation to multi-agent conversations—this avoids race conditions and keeps the analysis deterministic and auditable at scale.

---

## Known issues & troubleshooting ⚠️

- PDF parsing problems (unstructured / pdfminer): if you hit parsing errors, try downgrading `python` or aligning `pdfminer.six` and `unstructured` versions as shown in the notebooks.
- Blank agent output: usually caused by missing `llm_config` in `AssistantAgent` or incorrect `llm_config` keys (e.g., remove `model_name`, keep `model`).
- Playwright not working: run `playwright install` and restart the kernel; use `nest_asyncio.apply()` in notebooks.
- Log-reading bug: initializing `logs = []` and then doing `logs += some_string` will produce lists of characters—initialize logs as a string or append strings to a list and `"\n".join(...)`.
- Ollama connectivity: make sure `ollama serve` is running and the model you call is pulled (`ollama pull <model>`).

---

## How to extend this project 🔧

- Add a `requirements.txt` or `pyproject.toml` to freeze working versions for reproducibility.
- Create small, focused scripts under `scripts/` that expose key flows (doc ingestion, build embeddings, run RetrievalQA, log analysis) so non-notebook users can run them headless.
- Add automated tests for utility functions (log reading, chunking) and sample CI steps if desired.

---

## Final notes ✨

This repository is intentionally exploratory. The notebooks are written to be instructive and show how to solve real-world problems using local LLMs, embeddings, and small agent patterns. Read the notebooks in each `Section*` folder for implementation details and the lessons learned recorded inline.

If you'd like, I can also:
- create a `requirements.txt` with the pinned versions used in the notebooks, and
- add short runnable scripts (non-notebook) for the most important flows (document ingestion -> embeddings -> retriever; log analysis pipeline; BDD generator CLI).

---

Maintainers: Project owner and contributors in repo
License: (Add license if needed)
