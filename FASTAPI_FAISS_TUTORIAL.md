# The Ultimate Interview Guide: FAISS & FastAPI

This guide breaks down the two most critical backend technologies in your project from absolute scratch. Read this carefully to dominate the technical rounds.

---

## PART 1: FAISS (Facebook AI Similarity Search)

### 1. The Basics: What is a Vector Database?
Computers cannot understand English. If you ask a computer to find a document about "fixing a car engine", and the document says "automotive motor repair", a standard SQL database (like MySQL) will fail because the exact words don't match.

To solve this, we use an **Embedding Model** (in your project, OpenAI's `text-embedding-3`). This model reads the text and converts its *meaning* into an array of floating-point numbers (a vector). For example, 1536 numbers. 
* "car engine" -> `[0.12, -0.45, 0.89...]`
* "automotive motor" -> `[0.11, -0.44, 0.88...]`

Because their meanings are similar, their numbers are mathematically very close. 

### 2. What is FAISS?
FAISS is an open-source library developed by Facebook AI. It is designed to take thousands of these vectors and find the most similar ones incredibly fast. 

When a user asks a question, we convert the question into a vector, and FAISS calculates the distance between the question's vector and all the document vectors using **Cosine Similarity** (a math formula that measures the angle between two vectors).

### 3. How did we implement it in the project?
In `src/rag/retriever_setup.py`, we implemented a completely **in-memory** FAISS vector store.

```python
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS
from langchain_core.documents import Document

# 1. Initialize the embedding model
embeddings = OpenAIEmbeddings()

# 2. Global variable to hold our FAISS database in memory
_faiss_vectorstore = None

def retriever_chain(chunks: list[Document]):
    global _faiss_vectorstore
    # 3. Create the FAISS database from the chunked PDF text
    vectorstore = FAISS.from_documents(
        documents=chunks, 
        embedding=embeddings
    )
    _faiss_vectorstore = vectorstore
    return True
```

> [!IMPORTANT]
> **How to explain it in the interview:**
> *"When a user uploads a PDF, my system chunks the text and uses OpenAI to convert it into dense vectors. I chose FAISS to store these vectors because it runs entirely in-memory within the Python process. This means I didn't have to spin up an external database server like Pinecone or Qdrant, resulting in zero-latency network overhead. FAISS uses the HNSW algorithm to perform O(log N) similarity search, returning the most semantically relevant document chunks to the LLM."*

---
---

## PART 2: FastAPI (From Zero to Hero)

### 1. What is an API?
An API (Application Programming Interface) is how two programs talk to each other. Your Streamlit Frontend needs to talk to your LangGraph logic. Streamlit sends an HTTP request (like a letter in the mail) to FastAPI, and FastAPI returns a JSON response.

### 2. What is FastAPI?
FastAPI is a modern, ultra-fast web framework for Python. 
It has two massive advantages over older frameworks like Flask or Django:
1. **ASGI (Asynchronous):** It uses `async/await`, meaning it doesn't freeze the server while waiting for slow things (like database reads or OpenAI API calls).
2. **Pydantic Validation:** It automatically checks if the data coming from the frontend is correct.

### 3. Core Concept 1: The App and Routers
You don't put all your code in one massive file. You create the main app in `main.py`, and use `APIRouter` in `routes.py` to keep things organized.

**Your Code (`src/main.py`):**
```python
from fastapi import FastAPI
from src.api.routes import router

# Initialize the main app
app = FastAPI(title="Adaptive RAG API")

# Connect the router (which holds all our endpoints)
app.include_router(router)
```

### 4. Core Concept 2: Endpoints and Pydantic Models
An endpoint is a specific URL that does a specific task. We have an endpoint `/rag/query`. We use Pydantic (the `QueryRequest` class) to force the user to send the exact data we expect.

**Your Code (`src/models/query_request.py`):**
```python
from pydantic import BaseModel

class QueryRequest(BaseModel):
    query: str
    session_id: str
```
If Streamlit forgets to send `session_id`, FastAPI will automatically reject the request with a 422 Error before your code even runs!

**Your Code (`src/api/routes.py`):**
```python
@router.post("/rag/query")
async def rag_query(req: QueryRequest):
    # 'req' is now guaranteed to have req.query and req.session_id
    
    # 1. Fetch memory
    chat_history = ChatHistory.get_session_history(req.session_id)
    
    # 2. The 'await' keyword is crucial! It tells FastAPI to go serve 
    # other users while we wait for MongoDB to respond.
    await chat_history.add_message(HumanMessage(content=req.query))
    messages = await chat_history.get_messages()
    
    # 3. Run the LangGraph logic
    result = builder.invoke({"messages": messages})
    
    return {"result": result["messages"][-1]}
```

### 5. Core Concept 3: Handling File Uploads
When a user uploads a PDF, they aren't sending JSON text. They are sending raw binary file data. FastAPI has special classes `UploadFile` and `File` to handle this safely.

> [!TIP]
> **Why UploadFile?** If a user uploads a 500MB PDF, reading it straight into RAM would crash the server (OOM error). `UploadFile` is smart: it stores small files in RAM, but saves large files to a temporary disk location automatically to protect the server.

**Your Code (`src/api/routes.py`):**
```python
from fastapi import UploadFile, File, Header

@router.post("/rag/documents/upload")
async def upload_file(
    file: UploadFile = File(...), 
    description: str = Header(..., alias="X-Description")
):
    # We pass the safe UploadFile object into our LangChain logic
    status_upload = documents(description, file)
    return {"status": status_upload}
```
*Note: You also used `Header(...)` here. This means the description of the PDF is sent hidden in the HTTP Headers, rather than mixing it with the raw binary file data.*

---

### 🎤 The FastAPI Interview Pitch
If they ask: *"Why FastAPI and what were its key features in your project?"*

You say:
> *"I chose FastAPI for the backend over Flask because an LLM application is heavily I/O bound—we are constantly waiting on OpenAI and MongoDB. FastAPI's native ASGI support allowed me to use `async def` and `await`, meaning my server thread never blocks during API calls and can handle high concurrency. Additionally, I heavily relied on FastAPI's integration with Pydantic for data validation on my `/rag/query` route, and used its `UploadFile` class to safely spool PDF uploads to disk without exhausting server RAM."*
