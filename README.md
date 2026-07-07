# 🧠 Adaptive RAG — Agentic AI Chatbot

[![Python 3.9+](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-Latest-green.svg)](https://fastapi.tiangolo.com/)
[![LangGraph](https://img.shields.io/badge/LangGraph-0.5.4-orange.svg)](https://python.langchain.com/langgraph/)
[![FAISS](https://img.shields.io/badge/FAISS-VectorDB-blue.svg)](https://github.com/facebookresearch/faiss)

Adaptive RAG is an intelligent, end-to-end Retrieval-Augmented Generation (RAG) system built on an **Agentic AI architecture**. Using **LangGraph** for workflow orchestration, **FAISS** for local high-performance vector retrieval, and **MongoDB** for chat memory, the system dynamically routes, grades, and refines queries to deliver precise, context-aware answers.

---

## 🎯 Key Capabilities

### 🧠 Intelligent Query Routing
The system automatically classifies incoming queries using an LLM router to choose the optimal processing pathway:
*   **`index`**: Routed to local document search when relevant info is found in indexed PDF/TXT files.
*   **`general`**: Routed directly to the LLM for casual chitchat or general knowledge questions.
*   **`search`**: Routed to real-time search engine when live internet data or niche external lookups are required.

### 🔄 Grading & Self-Correction Loop
Retrieved document chunks are graded for relevance. If the context is deemed insufficient, the query is autonomously rewritten using an optimization loop and retrieved again, preventing generic fallback answers or hallucinations.

### 💾 Async State & Memory
Maintains complete session-based chat history stored asynchronously in MongoDB, ensuring conversational awareness without event-loop blocking.

---

## 🏗️ Architecture & Workflow

```
                        ┌────────────────────────┐
                        │       User Query       │
                        └───────────┬────────────┘
                                    │
                                    ▼
                        ┌────────────────────────┐
                        │   Query Classifier     │
                        └───────────┬────────────┘
                                    │
            ┌───────────────────────┼───────────────────────┐
            │ (index)               │ (general)             │ (search)
            ▼                       ▼                       ▼
┌────────────────────────┐┌──────────────────┐┌────────────────────────┐
│  ReAct Agent (FAISS)   ││   General LLM    ││ Web Search (Tavily)    │
└───────────┬────────────┘└─────────┬────────┘└───────────┬────────────┘
            │                       │                     │
            ▼                       │                     ▼
┌────────────────────────┐          │         ┌────────────────────────┐
│   Relevance Grader     │          │         │    Response Generator  │
└───────────┬────────────┘          │         └───────────┬────────────┘
            │                       │                     │
      [Is Relevant?]                │                     │
       /         \                  │                     │
     (No)       (Yes)               │                     │
     /             \                │                     │
    ▼               ▼               │                     │
┌──────────────┐┌──────────────┐    │                     │
│ Query Rewrite││   Generate   │    │                     │
└──────┬───────┘└──────┬───────┘    │                     │
       │               │            │                     │
       └───────┬───────┴────────────┼─────────────────────┘
               │                    │
               ▼                    ▼
         ┌───────────┴───────────────────────┐
         │       Save to MongoDB & Reply     │
         └───────────────────────────────────┘
```

### LangGraph Workflow Nodes:
1.  **`query_analysis`**: Determines if the query matches indexed document contents.
2.  **`retriever`**: Performs semantic search on the local FAISS index using a ReAct agent.
3.  **`grade`**: Uses LLM structured outputs to verify if retrieved context is relevant.
4.  **`rewrite`**: Refines search queries when grader indicates low relevance.
5.  **`generate`**: Crafts the final answer from retrieved text blocks.
6.  **`web_search`**: Scrapes search engines via Tavily API for fresh/real-time answers.
7.  **`general_llm`**: Handles general conversational queries without database overhead.

---

## 📂 Repository Structure

```
AdaptiveRag/
├── src/                              # Python backend application
│   ├── main.py                       # FastAPI entry point
│   ├── api/                          # REST API routes
│   ├── config/                       # Settings and prompts configuration
│   ├── core/                         # Loggers and environment loading
│   ├── db/                           # MongoDB database client
│   ├── llms/                         # ChatOpenAI models setup
│   ├── memory/                       # MongoDB chat history backend
│   ├── models/                       # Pydantic schemas and Graph states
│   ├── rag/                          # LangGraph definitions and document chunkers
│   └── tools/                        # Graph routers and utility tools
├── streamlit_app/                    # Frontend UI application
│   ├── home.py                       # Login page & backend auth client
│   ├── pages/                        # Multi-page views (Chat UI)
│   └── utils/                        # Frontend API integrations
└── requirements.txt                  # Python dependencies
```

---

## ⚡ Setup & Installation

### Prerequisites
*   **Python 3.9+**
*   **MongoDB** (Local or Atlas Instance)
*   **OpenAI API Key**
*   **Tavily API Key**

### 1. Clone & Install Dependencies
```bash
git clone https://github.com/ChandaVarshith/Adaptive-Rag.git
cd Adaptive-Rag

# Create a virtual environment
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate

# Install required dependencies
pip install -r requirements.txt
```

### 2. Environment Configuration
Create a `.env` file in the root directory:
```env
OPENAI_API_KEY=your_openai_key
TAVILY_API_KEY=your_tavily_key
MONGODB_URL=mongodb://localhost:27017
MONGODB_DB_NAME=adaptive_rag
```

### 3. Run the Services

**Start backend API server:**
```bash
python -m uvicorn src.main:app --reload --host 0.0.0.0 --port 8000
```

**Start frontend UI:**
```bash
streamlit run streamlit_app/home.py
```
Open [http://localhost:8501](http://localhost:8501) in your browser.

---

## 🔌 API Documentation

Detailed interactive Swagger docs are available locally at [http://localhost:8000/docs](http://localhost:8000/docs).

### 1. Document Upload
Upload a document (PDF or TXT) to index it into the local FAISS vector store.

```http
POST /rag/documents/upload
X-Description: Brief description of the uploaded document contents (used by retriever prompt)

Form Data:
  file: <Binary PDF or TXT file>
```
*   **Response (`200 OK`)**:
    ```json
    { "status": true }
    ```

### 2. Ask a Query
Submit a question to get an adaptive response.

```http
POST /rag/query
Content-Type: application/json

{
  "query": "What is the main topic of my uploaded resume?",
  "session_id": "session_user_999"
}
```
*   **Response (`200 OK`)**:
    ```json
    {
      "result": {
        "type": "ai",
        "content": "The uploaded resume highlights deep expertise in Python, FastAPI, and LangGraph..."
      }
    }
    ```

---

## 🛠️ Technology Stack

| Module | Technology | Details |
| :--- | :--- | :--- |
| **Workflow Manager** | LangGraph | Stateful, cyclic agent execution |
| **LLM Provider** | OpenAI | Model: `gpt-4o` |
| **Vector DB** | FAISS | In-memory semantic search |
| **Chat Memory** | MongoDB / Motor | Async BSON message persistence |
| **API Backend** | FastAPI / Uvicorn | ASGI high-performance async endpoints |
| **Frontend UI** | Streamlit | Chat and upload views |
| **Web Search** | Tavily | Real-time web search capabilities |

---

## 👤 Author
*   **Dhruv Singhal** — GitHub: [@dhruvsinghal09](https://github.com/dhruvsinghal09)
*   **Fork Maintainer**: [@ChandaVarshith](https://github.com/ChandaVarshith)
