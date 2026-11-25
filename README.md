# RAG Project

A Python-based retrieval-augmented generation (RAG) demo built with LangChain and LangGraph. It combines a Milvus vector database, dense and sparse retrieval, LLM-based answer grading, query rewriting, and optional Tavily web search.

## Features

- Markdown document parsing and semantic chunking
- Hybrid Milvus retrieval with BM25 sparse vectors and BGE dense vectors
- Agent-style RAG workflow with document relevance checks and query rewriting
- Advanced LangGraph workflow with:
  - question routing between vector search and web search
  - document relevance grading
  - hallucination and answer grading
  - automatic query transformation and retry
- Conversation history support in the simple agent demo
- Workflow visualization export through Mermaid

## Project Structure

```text
.
├── agent/                 # Tool-calling agent with conversation history
├── datas/md/              # Source Markdown documents
├── datas/output/          # Example parsed JSON output
├── documents/             # Markdown parsing and Milvus ingestion
├── graph/                 # Agent and retrieval LangGraph workflow
├── graph2/                # Graded RAG workflow with web-search routing
├── llm_models/            # Chat model and embedding configuration
├── logs/                  # Runtime log output
├── search_tool/           # Tavily search experiments
├── test_* /               # Milvus and loading examples
├── tools/                 # LangChain retriever tool
└── utils/                 # Environment and logging utilities
```

## Requirements

- Python 3.10 or later
- A running Milvus server
- An OpenAI-compatible API key
- Optional: Tavily API key for web search
- Optional: DeepSeek API key if switching the chat model configuration

The project uses packages including LangChain, LangGraph, PyMilvus, LangChain Milvus, Hugging Face Transformers, and Unstructured. Install the dependencies manually if they are not already present:

```bash
python -m pip install python-dotenv \
  langchain langchain-core langchain-community langchain-openai \
  langgraph langchain-milvus langchain-huggingface \
  langchain-experimental pymilvus sentence-transformers unstructured
```

You may need additional system or Python dependencies for `unstructured` Markdown parsing. Use the project's error output to install the exact missing parser packages.

## Configuration

Create a `.env` file in the project root:

```dotenv
OPENAI_API_KEY=your_openai_compatible_api_key
TAVILY_API_KEY=your_tavily_api_key
# Optional, for the DeepSeek example
DEEPSEEK_API_KEY=your_deepseek_api_key
```

Milvus and collection settings are currently configured in [utils/env_utils.py](/Users/olivia/Desktop/RAG_PROJECT/utils/env_utils.py):

```python
MILVUS_URI = "http://1.95.116.112:19530"
COLLECTION_NAME = "t_collection01"
```

Update these values for your own Milvus deployment. The chat model also uses an OpenAI-compatible endpoint in [llm_models/all_llm.py](/Users/olivia/Desktop/RAG_PROJECT/llm_models/all_llm.py). Change the `base_url`, model name, or API provider there if needed.

## Data Ingestion

Markdown files in `datas/md/` are parsed by [documents/markdown_parser.py](/Users/olivia/Desktop/RAG_PROJECT/documents/markdown_parser.py). The parser:

1. Loads Markdown elements.
2. Merges title hierarchy with associated content.
3. Applies semantic chunking to large documents.
4. Adds source metadata.

Before running ingestion, update the input directory in [documents/write_milvus.py](/Users/olivia/Desktop/RAG_PROJECT/documents/write_milvus.py). Its default value is a Windows path and is not portable.

The ingestion script recreates the configured collection, so it deletes existing indexes and collection data before rebuilding it.

```bash
python documents/write_milvus.py
```

The same operation can also be run for one file through [documents/milvus_db.py](/Users/olivia/Desktop/RAG_PROJECT/documents/milvus_db.py), but first update its example Windows paths.

## Run the Demos

From the project root:

```bash
# Tool-calling agent with stored chat history
python agent/rag_agent.py

# LangGraph RAG workflow with relevance checks and rewriting
python graph/graph1.py

# Advanced RAG workflow with routing, web search, and answer grading
python graph2/graph_2.py
```

Each workflow is interactive and prompts for Chinese questions in the terminal:

```text
用户：什么是EUV光刻机？
```

Type `q`, `exit`, or `quit` to stop the demo.

## Workflow Overview

### Graph 1

```text
Agent -> Retrieval tool -> Document relevance grade
                                |
                                +--> Generate answer
                                |
                                +--> Rewrite query -> Agent
```

[graph_rag1.png](/Users/olivia/Desktop/RAG_PROJECT/graph_rag1.png) and [graph/graph_rag1-2.png](/Users/olivia/Desktop/RAG_PROJECT/graph/graph_rag1-2.png) contain rendered workflow diagrams.

### Graph 2

```text
Question
   |
   +--> Vector store -> Retrieve -> Grade documents
   |                                  |
   |                                  +--> Generate -> Verify answer
   |                                  |
   |                                  +--> Transform query -> Retry
   |
   +--> Tavily web search -> Generate -> Verify answer
```

See [graph_rag2.png](/Users/olivia/Desktop/RAG_PROJECT/graph_rag2.png) for the rendered workflow.

## Notes

- The retriever defaults to the `content` category and returns the top four documents.
- Dense embeddings use `BAAI/bge-small-zh-v1.5`; OpenAI embeddings are used for semantic Markdown chunking.
- The Milvus collection uses separate sparse BM25 and dense HNSW indexes.
- `main.py` is only a PyCharm sample and is not the RAG entry point.
- Do not commit real API keys or credentials to `.env`, Git, or source files.
