# Patentis

Discover what hasn't been patented yet. Patentis maps an existing patent landscape, identifies genuine innovation gaps, generates novel product ideas, and guides you through development — all while ensuring you don't infringe on existing patents.

## How it works

1. **Search** — Enter your Groq API key and describe a technology area. Patentis extracts keywords and queries USPTO, EPO, arXiv, and Semantic Scholar in parallel.
2. **Review** — See all retrieved patents and papers. Select which ones go into the analysis.
3. **Analyze** — An LLM reads the documents and produces a patent landscape map: covered territory, gaps, and research directions.
4. **Ideas** — Generates novel ideas filling the identified gaps. Select one to trigger an infringement check.
5. **Develop** — A chat interface with full patent context guides you through development.

## Prerequisites

- **Python 3.11+**
- **Node.js 18+**
- **Groq API key** — free at [console.groq.com](https://console.groq.com)

## Quick start

```bash
git clone https://github.com/Natheoah/patentisv1.git
cd patentisv1
./start.sh
```

Open **http://localhost:5173**, enter your Groq API key on the home screen, and search.

The script will:
- Create a Python virtual environment and install dependencies
- Install Node packages
- Start both the backend (port 8000) and frontend (port 5173)

> First run downloads the sentence-transformer embedding model (~90 MB). Subsequent runs are instant.

## Manual setup

### Backend

```bash
cd backend
python3 -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

## Groq API key

You can provide your key in two ways:

1. **In the app** — enter it on the home screen (easiest, no config needed)
2. **In `backend/.env`** — set `GROQ_API_KEY=your_key` to pre-fill it server-side

Get a free key at [console.groq.com/keys](https://console.groq.com/keys).

## EPO credentials (optional)

Register for free at [ops.epo.org](https://ops.epo.org) to add European patent data. Add to `backend/.env`:

```
EPO_CLIENT_ID=your_id
EPO_CLIENT_SECRET=your_secret
```

Without EPO credentials, Patentis uses USPTO + arXiv + Semantic Scholar.

## Architecture

```
patentisv1/
├── backend/
│   ├── main.py                   # FastAPI app
│   ├── api/
│   │   ├── deps.py               # Groq key extraction (header → env fallback)
│   │   └── routes/
│   │       ├── search.py         # POST /api/search
│   │       ├── session.py        # GET/PATCH /api/session/{id}
│   │       ├── analysis.py       # POST /api/session/{id}/analyze  (SSE)
│   │       ├── ideas.py          # POST /api/session/{id}/generate-ideas
│   │       │                     # POST /api/session/{id}/select-idea  (SSE)
│   │       └── chat.py           # POST /api/session/{id}/chat  (SSE)
│   ├── services/
│   │   ├── llm.py                # Groq API client + model routing
│   │   ├── patent_search.py      # USPTO PatentsView + EPO OPS
│   │   ├── paper_search.py       # arXiv + Semantic Scholar
│   │   ├── rag.py                # ChromaDB + sentence-transformers
│   │   └── session_store.py      # In-memory session state
│   └── models/schemas.py         # Pydantic models
└── frontend/
    └── src/
        ├── store.ts              # Zustand global state
        ├── api/client.ts         # Fetch + SSE helpers
        ├── pages/                # SearchPage → ReviewPage → AnalysisPage → IdeasPage → GuidancePage
        └── components/           # PatentCard, PaperCard, IdeaCard, ChatMessage, ThinkingBlock
```

## Model routing

| Task | Model |
|---|---|
| Keyword extraction | llama-3.3-70b-versatile |
| Patent gap analysis | qwen/qwen3-32b |
| Idea generation | qwen/qwen3-32b |
| Infringement check | qwen/qwen3-32b |
| Development guidance chat | llama-3.3-70b-versatile |

## Data sources (all free)

| Source | What it provides |
|---|---|
| USPTO PatentsView API | US patents — title, abstract, inventors, assignee |
| EPO OPS | European patents (requires free registration) |
| arXiv API | Academic preprints |
| Semantic Scholar API | Broader academic paper search |
