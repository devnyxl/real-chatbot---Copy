# 🏝️ Indonesian Legal & Cultural Assistant

A RAG-powered chatbot that helps **international tourists, expats, and digital nomads** navigate Indonesian law and Balinese cultural norms — so they stay safe, informed, and respectful.

> Built for people living or traveling in Indonesia who need reliable answers about visa rules, drone usage, alcohol regulations, temple etiquette, overstay risks, and more.

---

## ✨ Features

- **Dual-domain RAG** — queries both a **law** corpus and a **Balinese culture** corpus simultaneously, returning two focused answers per question
- **Hybrid Retrieval** — combines dense vector search (ChromaDB) with sparse BM25 keyword matching for high-recall candidate generation
- **Cross-Encoder Reranking** — BAAI/bge-reranker-v2-m3 scores and reranks candidates for precision
- **Direct Article Lookup** — exact metadata match when users reference a specific article/pasal number (e.g. "Article 458")
- **Local LLM** — runs fully offline via [Ollama](https://ollama.com/) (default: `qwen3:4b`)
- **Flask Web UI** — clean browser-based chat interface

---

## 🗂️ Project Structure

```
.
├── app.py               # Flask web server — serves index.html + /ask endpoint
├── ragWeb.py            # Production RAG: dual-domain retrieval, LLM call, response cleaning
├── vector.py            # All retrieval logic: ChromaDB, BM25, reranker, HybridRetriever
├── templates/
│   └── index.html       # Frontend chat UI
├── data/
│   ├── data.json        # Indonesian Penal Code (law corpus source)
│   └── bali.json        # Balinese ceremony & taboo data (culture corpus source)
├── vectorstore_law/     # Persisted ChromaDB — law domain
├── vectorstore_culture/ # Persisted ChromaDB — culture domain
└── pdf/                 # Source PDF documents
```

---

## ⚙️ Tech Stack

| Layer | Component |
|---|---|
| **LLM** | [Ollama](https://ollama.com/) · `qwen3:4b` |
| **Embeddings** | Ollama · `bge-m3` |
| **Vector Store** | [ChromaDB](https://www.trychroma.com/) (persisted locally) |
| **Sparse Retrieval** | BM25Okapi via `rank_bm25` |
| **Reranker** | `BAAI/bge-reranker-v2-m3` via `sentence-transformers` |
| **Orchestration** | [LangChain](https://www.langchain.com/) (`langchain-core`, `langchain-ollama`, `langchain-chroma`) |
| **Web Server** | [Flask](https://flask.palletsprojects.com/) |
| **Frontend** | Vanilla HTML / CSS / JS |

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- [Ollama](https://ollama.com/) installed and running locally
- Required Ollama models pulled:

```bash
ollama pull qwen3:4b
ollama pull bge-m3
```

### Installation

```bash
# 1. Clone the repo
git clone <your-repo-url>
cd <repo-folder>

# 2. Create and activate virtual environment
python -m venv venv

# Windows
venv\Scripts\activate

# macOS / Linux
source venv/bin/activate

# 3. Install dependencies
pip install flask langchain langchain-core langchain-ollama langchain-chroma \
            chromadb rank_bm25 sentence-transformers numpy
```

### Run the Web App

```bash
python app.py
```

Open your browser at **http://127.0.0.1:5000**

---

## 🔍 How It Works

### Retrieval Pipeline

```
User Question
     │
     ▼
┌──────────────────────────────────────────┐
│   Direct Article Lookup (if article/     │
│   pasal number detected in query)        │
└──────────────────────┬───────────────────┘
                       │ (fallback if no exact match)
                       ▼
┌──────────────────────────────────────────┐
│   Hybrid Candidate Generation            │
│   ├─ Vector Search   (ChromaDB top-10)   │
│   └─ BM25 Search     (BM25Okapi top-10)  │
│   → Dedup by canonical article ID        │
└──────────────────────┬───────────────────┘
                       ▼
┌──────────────────────────────────────────┐
│   Cross-Encoder Reranker                 │
│   BAAI/bge-reranker-v2-m3 → top-5 docs  │
└──────────────────────┬───────────────────┘
                       ▼
           Build Context (≤16 000 chars)
                       │
                       ▼
┌──────────────────────────────────────────┐
│   LLM (Ollama qwen3:4b)                  │
│   ├─ Law answer   (from law context)     │
│   └─ Culture answer (from culture ctx)   │
└──────────────────────────────────────────┘
```

Each question is answered from **both** domains independently. The frontend displays both law and culture perspectives side by side.

### Domains

| Domain | Contents |
|---|---|
| **Law** | Indonesian Penal Code articles (`data.json`) — criminal penalties, sentences, complaint rights, etc. |
| **Culture** | Balinese ceremonies & taboos (`bali.json`) — Nyepi, Galungan, Ngaben, temple dress code, offerings etiquette, etc. |

---

## 🖥️ API

### `POST /ask`

**Request:**
```json
{ "question": "Can I fly a drone near a temple in Bali?" }
```

**Response:**
```json
{
  "answer_law": "...",
  "answer_culture": "...",
  "docs_law": 5,
  "docs_culture": 5,
  "time": 3.21
}
```

---

## 🔧 Configuration

Key constants to tune:

**`vector.py`**
```python
EMBED_MODEL    = "bge-m3"                  # Ollama embedding model
VECTOR_TOP_K   = 10                        # Candidates from vector search
BM25_TOP_K     = 10                        # Candidates from BM25
RERANK_TOP_K   = 5                         # Final docs after reranking
RERANKER_MODEL = "BAAI/bge-reranker-v2-m3"
```

**`ragWeb.py`**
```python
LLM_MODEL         = "qwen3:4b"             # Ollama chat model
TEMPERATURE       = 0.1                    # Low temp for factual answers
MAX_CONTEXT_CHARS = 16000                  # Context window limit
```

---

## 🎨 Brand & Design

Premium, calming feel — inspired by boutique Balinese resorts. Principles:

- **Understated authority** — serious legal topics, serene presentation
- **Accessible warmth** — approachable, not corporate
- **Restrained locality** — subtle Indonesian cultural cues, no touristy clip-art
- **Clarity above all** — legibility, clear status, unambiguous answers
- WCAG 2.1 AA contrast; supports light/dark mode; mobile-first (min 44px tap targets)

---

## 📄 License

Educational and research use.

---

## 🙏 Acknowledgements

- [Ollama](https://ollama.com/) — local LLM inference
- [BAAI](https://huggingface.co/BAAI) — `bge-m3` embeddings & `bge-reranker-v2-m3`
- [LangChain](https://www.langchain.com/) — orchestration
- [ChromaDB](https://www.trychroma.com/) — vector persistence
