# MultiDocChat — RAG-Powered Multi-Document Chat API

[![CI/CD to ECS Fargate](https://github.com/Yashgiradkar/RAG_MultiDoc_Chat/actions/workflows/aws.yml/badge.svg)](https://github.com/Yashgiradkar/RAG_MultiDoc_Chat/actions/workflows/aws.yml)

---

## Table of Contents

- [Project Description](#project-description)
- [Tech Stack](#tech-stack)
- [How to Run](#how-to-run)
- [API Endpoints](#api-endpoints)
- [Build & Deployment](#build--deployment)

---

## Project Description

**MultiDocChat** is a conversational Retrieval-Augmented Generation (RAG) API that allows users to upload multiple documents (PDF, DOCX, TXT, and more) and chat with their contents via a context-aware Q&A interface. Uploaded documents are chunked, embedded using Google's Gemini embedding model, and persisted in a session-scoped FAISS vector index. Subsequent chat queries use Maximal Marginal Relevance (MMR) retrieval and an LCEL-based LangChain chain to generate grounded, concise answers while maintaining full conversation history. The project is containerized with Docker and ships with a complete CI/CD pipeline targeting AWS ECS Fargate.

---

## Tech Stack

| Category             | Technology / Library                                                                               |
|----------------------|----------------------------------------------------------------------------------------------------|
| **Backend**          | Python 3.14, FastAPI 0.115.6, Uvicorn 0.32.1, Jinja2 3.1.4                                       |
| **AI / LLM**         | LangChain 0.3.27, LangChain-Google-GenAI 2.1.8 (Gemini), LangChain-Groq 0.3.6, LangSmith 0.10.10|
| **Vector Store**     | FAISS (`faiss-cpu`)                                                                                |
| **Embeddings**       | Google Gemini Embedding (`models/gemini-embedding-001`)                                            |
| **Document Parsing** | PyPDF 5.0.0 (PDF), docx2txt 0.9 (DOCX)                                                           |
| **Config**           | PyYAML, python-dotenv 1.1.1                                                                        |
| **Logging**          | structlog 25.4.0                                                                                   |
| **Testing**          | pytest, FastAPI TestClient                                                                         |
| **DevOps**           | Docker, GitHub Actions, AWS ECR, AWS ECS Fargate, AWS Secrets Manager                             |
| **Package Manager**  | uv                                                             |

---

## Prerequisites

| Requirement    | Version / Notes                                  |
|----------------|--------------------------------------------------|
| Python         | `>= 3.14` (`.python-version` pin: `3.14`)        |
| uv             | Latest — used as the primary package manager     |
| Docker         | Required for container-based runs and deployment |
| Google API Key | Gemini embedding + LLM access                    |
| Groq API Key   | Alternative LLM provider (Groq-hosted models)    |

---

## Installation

```bash
# 1. Clone the repository
git clone https://github.com/Yashgiradkar/RAG_MultiDoc_Chat.git
cd RAG_MultiDoc_Chat

pip install -r requirements.txt
```

---

## How to Run

### Local Development

```bash
# Using uvicorn directly (auto-reload enabled)
uvicorn main:app --host 0.0.0.0 --port 8000 --reload

# OR using the Python entrypoint
python main.py
```

The application will be available at `http://localhost:8000`.

### Production (Uvicorn with multiple workers)

```bash
uvicorn main:app --host 0.0.0.0 --port 8080 --workers 4
```

---

## API Endpoints

| Method | Endpoint  | Description                                                                                         |
|--------|-----------|-----------------------------------------------------------------------------------------------------|
| `GET`  | `/`       | Serves the HTML chat interface (`templates/index.html`)                                             |
| `GET`  | `/health` | Health check — returns `{"status": "ok"}`                                                           |
| `POST` | `/upload` | Accepts one or more files; indexes them into FAISS and returns a `session_id`                       |
| `POST` | `/chat`   | Accepts a `session_id` and a user `message`; returns a grounded answer from the indexed documents  |


---

## Build & Deployment

### Docker Build

```bash
docker build -t multidocchat:latest .
```

### AWS Infrastructure

| Resource       | Value                        |
|----------------|------------------------------|
| Region         | `us-east-1`                  |
| ECR Repository | `rag_multidoc_chat_repo`     |
| ECS Cluster    | `rag_multidoc_chat-cluster`  |
| ECS Service    | `rag_multidoc_chat-service`  |
| Container Port | `8080`                       |
| Task CPU       | 256 vCPU                     |
| Task Memory    | 512 MB                       |
