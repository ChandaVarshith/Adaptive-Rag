<div align="center">

# 🧠 Adaptive RAG

### An Intelligent, Self-Routing Retrieval-Augmented Generation System

[![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.5.x-FF6B6B?style=for-the-badge&logo=langchain&logoColor=white)](https://langchain-ai.github.io/langgraph/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115.x-009688?style=for-the-badge&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![Streamlit](https://img.shields.io/badge/Streamlit-1.x-FF4B4B?style=for-the-badge&logo=streamlit&logoColor=white)](https://streamlit.io)
[![Groq](https://img.shields.io/badge/Groq-LLaMA3-F54A00?style=for-the-badge&logo=groq&logoColor=white)](https://groq.com)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

</div>

---

## 📖 Overview

**Adaptive RAG** is a production-grade, agentic Retrieval-Augmented Generation system that **intelligently routes user queries** across three execution paths — vector-store retrieval, live web search, and direct LLM response — based on query relevance. Built with **LangGraph**, it implements a stateful, self-correcting pipeline that grades retrieved context and rewrites failing queries before falling back to Tavily web search.

> Built to be **chat-memory-aware**, **document-upload-ready**, and **deployment-friendly** via a FastAPI backend + Streamlit frontend.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         User Query                                  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                    ┌────────▼────────┐
                    │ Query Classifier │  ← Groq LLM + Context
                    └────────┬────────┘
           ┌─────────────────┼─────────────────┐
           │                 │                 │
    ┌──────▼──────┐   ┌──────▼──────┐  ┌──────▼──────┐
    │  Retriever  │   │ General LLM │  │  (Fallback) │
    │  (ReAct +   │   │  (Direct)   │  │             │
    │  Qdrant /   │   └──────┬──────┘  └─────────────┘
    │  FAISS)     │          │
    └──────┬──────┘          │
           │                 │
    ┌──────▼──────┐          │
    │   Grader    │          │
    └──────┬──────┘          │
     Pass  │  Fail           │
           │    └──► ┌───────▼──────┐
           │         │ Query Rewrite │
           │         └───────┬──────┘
           │                 │
           │    ┌────────────▼────────┐
           │    │    Web Search       │
           │    │    (Tavily API)     │
           │    └────────────┬────────┘
           │                 │
    ┌──────▼─────────────────▼──────┐
    │          Generator             │
    └───────────────┬────────────────┘
                    │
            ┌───────▼───────┐
            │  Final Answer  │
            └───────────────┘
```

![Adaptive RAG Flow](adaptive_RAG.png)

---

## ✨ Key Features

| Feature | Description |
|---|---|
| 🔀 **Adaptive Routing** | Classifies each query → vector store, general LLM, or web search |
| 🤖 **ReAct Agent** | Uses a reasoning-and-acting agent for smart document retrieval |
| ✅ **Context Grader** | LLM judges whether retrieved docs actually answer the question |
| 🔄 **Query Rewriter** | Rewrites poor queries to improve retrieval before web fallback |
| 🌐 **Web Search Fallback** | Tavily-powered real-time web search when local knowledge fails |
| 📄 **Document Upload** | Upload PDFs/TXT at runtime and index them instantly |
| 🧠 **Chat Memory** | Persistent conversation history via MongoDB |
| ⚡ **Dual Vector Stores** | Supports both Qdrant (production) and FAISS (local) |
| 🎨 **Streamlit UI** | Login, chat, and document upload — all in one interface |
| 🚀 **FastAPI Backend** | Async REST API ready for production deployment |

---

## 🗂️ Project Structure

```
Adaptive-Rag/
│
├── src/                          # Core backend package
│   ├── api/
│   │   └── routes.py             # FastAPI route definitions
│   ├── config/
│   │   ├── settings.py           # YAML config loader
│   │   └── prompts.yaml          # All LLM prompts (classify, grade, rewrite, generate)
│   ├── core/
│   │   ├── config.py             # App-level configuration constants
│   │   └── logger.py             # Centralized logging setup
│   ├── db/
│   │   └── mongo_client.py       # MongoDB async client singleton
│   ├── llms/
│   │   └── groq_llm.py           # Groq LLaMA3 client initialization
│   ├── memory/
│   │   ├── chat_history_mongo.py      # MongoDB-backed chat history
│   │   └── chathistory_in_memory.py   # In-memory chat history (dev)
│   ├── models/
│   │   ├── state.py              # LangGraph State schema
│   │   ├── grade.py              # Pydantic: relevance grade
│   │   ├── route_identifier.py   # Pydantic: routing decision
│   │   ├── query_request.py      # Pydantic: query API request
│   │   └── verification_result.py # Pydantic: verification output
│   ├── rag/
│   │   ├── graph_builder.py      # Full LangGraph node & edge definitions
│   │   ├── nodes.py              # Individual node logic stubs
│   │   ├── retriever_setup.py    # Qdrant / FAISS retriever factory
│   │   ├── document_upload.py    # Document ingestion pipeline
│   │   └── reAct_agent.py        # ReAct agent executor
│   ├── tools/
│   │   ├── common_tools.py       # Shared utility tools
│   │   └── graph_tools.py        # Routing & grading conditional functions
│   └── main.py                   # FastAPI application entry point
│
├── streamlit_app/                # Frontend package
│   ├── home.py                   # Login / signup page
│   ├── pages/
│   │   └── Chat.py               # Main chat + document upload interface
│   └── utils/
│       └── api_client.py         # HTTP client for FastAPI backend
│
├── .env.example                  # Environment variable template
├── .gitignore                    # Git exclusions
├── requirements.txt              # Python dependencies
├── adaptive_RAG.png              # Architecture diagram
└── README.md                     # This file
```

---

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/ChandaVarshith/Adaptive-Rag.git
cd Adaptive-Rag
```

### 2. Create a Virtual Environment

```bash
python -m venv .venv

# Windows
.venv\Scripts\activate

# macOS/Linux
source .venv/bin/activate
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Configure Environment Variables

```bash
cp .env.example .env
# Now edit .env and fill in your API keys
```

| Variable | Required | Description |
|---|---|---|
| `GROQ_API_KEY` | ✅ | Groq API key for LLaMA3 inference |
| `TAVILY_API_KEY` | ✅ | Tavily API key for web search |
| `MONGO_URI` | ✅ | MongoDB connection URI |
| `QDRANT_URL` | ⚠️ Optional | Qdrant vector store URL (falls back to FAISS) |
| `QDRANT_COLLECTION` | ⚠️ Optional | Qdrant collection name |

### 5. Start the FastAPI Backend

```bash
uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

### 6. Start the Streamlit Frontend

```bash
# In a second terminal
streamlit run streamlit_app/home.py
```

Open **http://localhost:8501** in your browser.

---

## 🔌 API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Health check |
| `POST` | `/rag/query` | Submit a query to the RAG pipeline |
| `POST` | `/rag/documents/upload` | Upload a document (PDF/TXT) |
| `POST` | `/api/init` | Initialize API session |
| `POST` | `/api/create_user` | Create a new user |
| `POST` | `/api/login` | Authenticate a user |

### Example: Query the RAG Pipeline

```bash
curl -X POST http://localhost:8000/rag/query \
  -H "Content-Type: application/json" \
  -d '{"query": "What are the main findings in the uploaded document?", "session_id": "user123"}'
```

### Example: Upload a Document

```bash
curl -X POST http://localhost:8000/rag/documents/upload \
  -H "X-Description: Research paper on transformer models" \
  -F "file=@paper.pdf"
```

---

## 🧩 LangGraph Pipeline

The system is built as a **stateful directed graph** using LangGraph:

```
START
  └─► query_analysis          # Classify: rag / general / web
        ├─► general_llm ─────► END
        ├─► retriever
        │     └─► grade
        │           ├─► generate ──► END   (docs relevant)
        │           └─► rewrite
        │                 └─► retriever    (retry with new query)
        │                        └─► web_search ──► generate ──► END
        └─► (web_search path)
```

### State Schema

```python
class State(TypedDict):
    messages: Annotated[list, add_messages]   # Full conversation history
    route: str                                # Routing decision
    latest_query: str                         # Current/rewritten query
    binary_score: str                         # Grader output: "yes" / "no"
    rewrite_count: int                        # Rewrite loop counter
```

---

## 🛠️ Technology Stack

| Layer | Technology |
|---|---|
| **LLM** | Groq (LLaMA 3.3 70B) |
| **Orchestration** | LangGraph + LangChain |
| **Vector Store** | Qdrant (cloud/local) + FAISS (fallback) |
| **Embeddings** | `sentence-transformers` (HuggingFace) |
| **Web Search** | Tavily API |
| **Backend API** | FastAPI + Uvicorn |
| **Frontend** | Streamlit |
| **Chat Memory** | MongoDB (Motor async driver) |
| **Config** | YAML + Pydantic + python-dotenv |

---

## 📋 Requirements

- Python **3.10+**
- MongoDB (local or Atlas)
- Qdrant (optional — falls back to FAISS automatically)
- Groq API key (free tier available at [console.groq.com](https://console.groq.com))
- Tavily API key (free tier at [tavily.com](https://tavily.com))

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/your-feature`)
3. Commit your changes (`git commit -m 'feat: add your feature'`)
4. Push to your branch (`git push origin feature/your-feature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Varshith Chanda**
- GitHub: [@ChandaVarshith](https://github.com/ChandaVarshith)

---

<div align="center">

⭐ **If you find this project useful, give it a star!** ⭐

</div>
