# Adaptive RAG - Agentic AI Chatbot

[![Python 3.9+](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Latest-green.svg)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.5.4-orange.svg)](https://python.langchain.com/langgraph/)
[![FAISS](https://img.shields.io/badge/FAISS-VectorDB-blue.svg)](https://github.com/facebookresearch/faiss)
[![MongoDB](https://img.shields.io/badge/MongoDB-Database-green.svg)](https://www.mongodb.com/)
[![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o-black.svg)](https://openai.com/)

## 📋 Overview

**Adaptive RAG** is an intelligent, end-to-end Retrieval-Augmented Generation (RAG) system powered by agentic AI architecture. It combines dynamic query routing, intelligent document retrieval, and advanced LLM capabilities to provide accurate, context-aware answers to user queries.

The system intelligently adapts its retrieval strategy based on query type — utilizing indexed documents, general knowledge, or real-time web search — to generate comprehensive and faithful responses. Built with a modular architecture using **LangGraph** for workflow orchestration and **FAISS** for high-speed in-memory vector retrieval.

---

## 🎯 Key Features

### 🧠 Intelligent Query Routing
- **Adaptive Classification**: Automatically routes queries to the most appropriate processing pipeline
- **Three Query Types**:
  - **`index`** — Queries answerable from uploaded documents
  - **`general`** — Queries answerable with general LLM knowledge
  - **`search`** — Queries requiring real-time web search

### 📚 Advanced RAG Pipeline
- **Document Processing**: Intelligent chunking and embedding of uploaded PDFs and TXTs
- **Vector Search**: Fast semantic similarity search using FAISS (in-memory, zero-latency)
- **Relevance Grading**: LLM-based evaluation of retrieved document chunks
- **Query Rewriting**: Automatic query optimization loop for better retrieval results

### 🤖 Agentic AI Architecture
- **ReAct Framework**: Reasoning and Acting pattern for autonomous decision-making
- **Self-Correction Loop**: Retries retrieval with a rewritten query if context is irrelevant
- **Tool Integration**: Seamless integration with FAISS retriever and Tavily web search

### 💾 Persistent Memory
- **MongoDB Backend**: Fully persistent chat history and session management
- **Async Driver**: Non-blocking database ops via Motor (AsyncIOMotorClient)
- **Session Tracking**: Per-user conversation context retained across queries

### 🎨 User Interface
- **Streamlit Web App**: Interactive chat interface with sidebar document upload
- **File Support**: PDF and TXT document uploads
- **Real-time Feedback**: Live chat with instant responses

### ⚡ API-First Architecture
- **FastAPI Backend**: High-performance ASGI REST API
- **Async Operations**: Non-blocking database and LLM API calls
- **Pydantic Validation**: Automatic request/response schema validation

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Interface                          │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  Streamlit Web Application                               │  │
│  │  • Chat Interface    • Document Upload (PDF, TXT)        │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────┘
                                ↓
┌─────────────────────────────────────────────────────────────────┐
│                       FastAPI Backend                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │  POST /rag/query          POST /rag/documents/upload      │  │
│  └──────────────────────────────────────────────────────────┘  │
└───────────────────────────────┬─────────────────────────────────┘
                                ↓
┌─────────────────────────────────────────────────────────────────┐
│                    LangGraph Orchestration                      │
│  ┌────────────┐   ┌──────────────┐   ┌───────────────────────┐ │
│  │  Query     │ → │   Classify   │ → │    Conditional Router  │ │
│  │  Received  │   │   Query      │   │ index / general/search │ │
│  └────────────┘   └──────────────┘   └───────────────────────┘ │
└───────────────────┬───────────────────────┬─────────────────────┘
        ↓                   ↓                       ↓
   ┌─────────┐       ┌──────────┐          ┌────────────┐
   │ ReAct   │       │ General  │          │ Web Search │
   │ (FAISS) │       │  LLM     │          │ (Tavily)   │
   └────┬────┘       └────┬─────┘          └─────┬──────┘
        │                 │                      │
        ▼                 │                      │
   ┌─────────┐            │                      │
   │  Grade  │            │                      │
   └────┬────┘            │                      │
   (yes)|  (no)           │                      │
        |    ↓            │                      │
        |  ┌──────────┐   │                      │
        |  │ Rewrite  │   │                      │
        |  │  Query   │   │                      │
        |  └────┬─────┘   │                      │
        |       └→ RetriEVER again               │
        ▼                 │                      │
   ┌─────────┐            │                      │
   │Generate │ ←──────────┘ ←────────────────────┘
   │ Answer  │
   └────┬────┘
        ↓
   ┌─────────────────────────────────┐
   │  Save to MongoDB & Return User  │
   └─────────────────────────────────┘
```

### Graph Nodes

| Node | Description |
|------|-------------|
| **query_analysis** | Classifies the incoming query and fetches initial FAISS context |
| **retriever** | ReAct agent performs semantic search on the FAISS vector index |
| **grade** | LLM grades whether retrieved chunks are relevant |
| **rewrite** | Rewrites the query to improve retrieval on next attempt |
| **generate** | Generates the final readable answer from retrieved context |
| **web_search** | Performs real-time web search via Tavily when needed |
| **general_llm** | Handles general knowledge / conversational queries |

---

## 📦 Project Structure

```
AdaptiveRag/
├── src/                              # Main source code
│   ├── main.py                       # FastAPI application entry point
│   ├── api/                          # API routes and endpoints
│   │   └── routes.py                 # RAG query and document upload endpoints
│   ├── config/                       # Configuration management
│   │   ├── settings.py               # Application settings loader
│   │   └── prompts.yaml              # All LLM prompts and system messages
│   ├── core/                         # Core utilities
│   │   ├── config.py                 # Environment variable loader
│   │   └── logger.py                 # Logging setup
│   ├── db/                           # Database layer
│   │   └── mongo_client.py           # Async MongoDB client (Motor)
│   ├── llms/                         # Language model integrations
│   │   └── openai.py                 # OpenAI GPT-4o initialization
│   ├── memory/                       # Chat memory management
│   │   ├── chat_history_mongo.py     # MongoDB-backed persistent chat history
│   │   └── chathistory_in_memory.py  # In-memory fallback chat history
│   ├── models/                       # Data models and schemas
│   │   ├── state.py                  # LangGraph state definition
│   │   ├── query_request.py          # Query request Pydantic schema
│   │   ├── grade.py                  # Relevance grade Pydantic model
│   │   ├── route_identifier.py       # Route classification model
│   │   └── verification_result.py    # Answer faithfulness model
│   ├── rag/                          # RAG pipeline implementation
│   │   ├── graph_builder.py          # LangGraph workflow builder
│   │   ├── retriever_setup.py        # FAISS vector store and retriever setup
│   │   ├── document_upload.py        # Document loading, chunking, and indexing
│   │   └── reAct_agent.py            # ReAct agent setup and AgentExecutor
│   └── tools/                        # Utility tools and functions
│       ├── common_tools.py           # LLM description enhancer
│       └── graph_tools.py            # Graph routing and decision tools
│
├── streamlit_app/                    # Streamlit web application
│   ├── home.py                       # Authentication and login page
│   ├── pages/
│   │   └── chat.py                   # Chat interface and document upload
│   └── utils/
│       └── api_client.py             # FastAPI backend HTTP client
│
├── README.md                         # Project documentation
├── requirements.txt                  # Python dependencies
├── .env.example                      # Example environment configuration
└── LICENSE                           # MIT License
```

---

## 🔌 API Endpoints

**Base URL:** `http://localhost:8000`

### 1. Query Endpoint

```http
POST /rag/query
Content-Type: application/json

{
  "query": "What is the main topic of the document?",
  "session_id": "user_session_123"
}
```

**Response:**
```json
{
  "result": {
    "type": "ai",
    "content": "Based on the document, the main topic is..."
  }
}
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `query` | string | ✅ | User's question or query |
| `session_id` | string | ✅ | Unique session ID for conversation tracking |

---

### 2. Document Upload Endpoint

```http
POST /rag/documents/upload
X-Description: Brief description of the document

Form Data:
  file: <PDF or TXT file>
```

**Response:**
```json
{ "status": true }
```

| Header | Required | Description |
|--------|----------|-------------|
| `X-Description` | ✅ | Document description used to scope the retriever tool |

**Supported Formats:** `.pdf` · `.txt`

---

## 📖 Setup Guide

### Prerequisites

- Python 3.9+
- MongoDB (local or [MongoDB Atlas](https://www.mongodb.com/atlas))
- OpenAI API Key
- Tavily API Key

### 1. Clone & Install

```bash
git clone https://github.com/ChandaVarshith/Adaptive-Rag.git
cd Adaptive-Rag

python -m venv .venv
source .venv/bin/activate       # Windows: .venv\Scripts\activate

pip install -r requirements.txt
```

### 2. Configure Environment

Create a `.env` file in the project root:

```env
# OpenAI
OPENAI_API_KEY=your_openai_api_key_here

# Tavily Search
TAVILY_API_KEY=your_tavily_api_key_here

# MongoDB
MONGO_URI=mongodb://localhost:27017
MONGO_DB_NAME=adaptive_rag
```

> **Note**: FAISS runs fully in-memory — no external vector database credentials needed.

### 3. Run the Application

```bash
# Terminal 1 — FastAPI backend
python -m uvicorn src.main:app --reload --host 0.0.0.0 --port 8000

# Terminal 2 — Streamlit frontend
streamlit run streamlit_app/home.py
```

| Service | URL |
|---------|-----|
| Web Interface | http://localhost:8501 |
| API Swagger Docs | http://localhost:8000/docs |
| ReDoc API Docs | http://localhost:8000/redoc |

---

## 🧪 Testing the API

### Example Queries

```bash
# Upload a document
curl -X POST http://localhost:8000/rag/documents/upload \
  -H "X-Description: Resume of Varshith Chanda" \
  -F "file=@resume.pdf"

# Query the system
curl -X POST http://localhost:8000/rag/query \
  -H "Content-Type: application/json" \
  -d '{"query": "What skills are mentioned in the resume?", "session_id": "user_1"}'
```

### Test Cases

| Test | Query | Expected Route |
|------|-------|----------------|
| Casual chat | `"Hello, how are you?"` | `general` |
| Document query | `"What is covered in the uploaded file?"` | `index` |
| Factual lookup | `"What is the latest GPT model?"` | `search` |

---

## 📊 Technical Parameters

| Parameter | Value | Reason |
|-----------|-------|--------|
| `chunk_size` | 1000 chars | Optimal context window balance |
| `chunk_overlap` | 150 chars | Prevents meaning loss at chunk boundaries |
| `max_iterations` | 2 | Limits ReAct agent loops to prevent cost blowup |
| `embedding_model` | `text-embedding-3` | OpenAI's dense semantic embedding model |
| `llm_model` | `gpt-4o` | Best structured output and reasoning capabilities |

---

## 📚 Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| **LLM Framework** | LangChain | ~0.3.27 |
| **Workflow Orchestration** | LangGraph | ~0.5.4 |
| **Web Framework** | FastAPI | Latest |
| **ASGI Server** | Uvicorn | Latest |
| **UI Framework** | Streamlit | Latest |
| **Vector Database** | FAISS (in-memory) | Latest |
| **Chat Database** | MongoDB + Motor | Latest |
| **LLM Provider** | OpenAI GPT-4o | ~0.3.28 |
| **Web Search** | Tavily | Latest |
| **Data Validation** | Pydantic | ~2.11.7 |

---

## 🔐 Security Considerations

- Store all API keys in `.env` — **never commit secrets**
- Use HTTPS in production deployments
- Implement rate limiting for public-facing APIs
- Validate and sanitize all user file inputs
- Secure MongoDB with proper auth credentials

---

## 📈 Project Status

- ✅ Core Adaptive RAG pipeline implemented
- ✅ Document upload and FAISS indexing
- ✅ Three-way query routing (index / general / search)
- ✅ LLM-based relevance grading and self-correction loop
- ✅ Async MongoDB chat history persistence
- ✅ Streamlit web interface
- ✅ ReAct agentic retriever
- 🚀 Production ready

---

## 🗺️ Roadmap

- [ ] Enhanced multi-document context management
- [ ] Multi-language support
- [ ] Persistent FAISS index (disk save/load)
- [ ] Extended LLM provider support (Groq, Gemini)
- [ ] Advanced user authentication system
- [ ] Real-time streaming responses
- [ ] Analytics and usage dashboard
- [ ] Docker containerized deployment

---

## 🤝 Contributing

Contributions are welcome! Follow these steps:

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/YourFeature`
3. Follow `CODE_STYLE_GUIDE.md` standards
4. Commit with descriptive messages: `git commit -m 'feat: Add YourFeature'`
5. Push: `git push origin feature/YourFeature`
6. Open a Pull Request

---

## 📄 License

This project is licensed under the **MIT License** — see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Varshith Chanda**
- GitHub: [@ChandaVarshith](https://github.com/ChandaVarshith)
- Project: [Adaptive RAG](https://github.com/ChandaVarshith/Adaptive-Rag)

---

**Last Updated**: July 2026 | **Status**: ✅ Production Ready
