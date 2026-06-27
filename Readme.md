# ⚖️ Law-Lens — AI-Powered Legal Document Analysis

**Law-Lens** is an intelligent legal document analysis backend that extracts, interprets, and visualizes key insights from legal PDFs. It combines legal-domain NLP models, LLMs, vector search, and OCR to help legal professionals quickly understand and navigate complex documents.

---

## Key Features

| Feature | Description |
|---|---|
| 📄 **Smart PDF Extraction** | Extracts text from any PDF — born-digital (fast path) or scanned (OCR fallback) |
| 🔑 **Legal Keyword Extraction** | Domain-aware keyword extraction using **LegalBERT** + **KeyBERT** |
| 📖 **Keyword Definitions** | Plain-English, one-sentence meanings for every legal term via LLM |
| ⚖️ **Indian Case Law Mapping** | Maps keywords to IPC/Act sections with **IndianKanoon** search links |
| 🎯 **Paragraph Importance Scoring** | Hybrid semantic filter + LLM scoring — tags paragraphs as High / Medium / Low |
| 🖍️ **Highlighted PDF Export** | Color-annotated PDF (red = high, yellow = medium) with bounding-box precision |
| 💬 **RAG-Powered Chatbot** | Ask questions about any uploaded document — grounded in its content via **FAISS** retrieval |
| 🚀 **FastAPI Backend** | Clean REST API with interactive Swagger docs at `/docs` |

---

## Tech Stack

```
Python 3.8+  |  FastAPI  |  Uvicorn
────────────────────────────────────
AI / NLP:    LegalBERT · KeyBERT · SentenceTransformers · Groq Llama 3.1
Vector:      FAISS (IndexFlatL2)
OCR:         Tesseract · pdf2image · Poppler
PDF:         pypdfium2 · PyMuPDF · PyPDF2 · Pillow
API:         FastAPI · Pydantic · python-multipart
UI (Alt):    Streamlit
```

---

## Architecture

```
┌─────────────┐     ┌──────────────────┐     ┌──────────────────────┐
│  FastAPI     │────▶│  Service Layer   │────▶│   Domain Modules     │
│  Routes      │     │  (Orchestration) │     │  (Pure Logic)        │
│              │     │                  │     │                      │
│  health.py   │     │  pdf_service.py  │     │  pdf_processor.py    │
│  pdf.py      │     │  chat_service.py │     │  keyword.py          │
│  chat.py     │     │  json_utils.py   │     │  keyword_meaning.py  │
└─────────────┘     └──────────────────┘     │  semantic_importance │
                                              │  vector_store.py     │
                                              │  chatbot.py          │
                                              │  highlight_pdf.py    │
                                              │  case_law_fetcher.py │
                                              │  ocr.py              │
                                              └──────────────────────┘
```

---

## Processing Pipeline

```
PDF Upload
    │
    ▼
┌──────────────────────────────────────────────┐
│ 1. Text Extraction                           │
│    Fast: pypdfium2  │  Fallback: PyPDF2      │
│    OCR:  Tesseract (if < 50 chars extracted) │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 2. Legal Keyword Extraction (LegalBERT)      │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 3. Keyword Meanings  (Groq Llama, JSON mode) │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 4. Case Law Mapping  (Groq → IndianKanoon)   │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 5. Paragraph Importance Scoring              │
│    Hybrid: SentenceTransformer filter        │
│           + Groq Llama batch scoring         │
│           + Keyword-count fallback           │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 6. FAISS Indexing (sliding-window chunks)    │
└──────────────────────────────────────────────┘
    │
    ▼
┌──────────────────────────────────────────────┐
│ 7. PDF Highlighting (PyMuPDF color annot.)   │
└──────────────────────────────────────────────┘
    │
    ▼
 Response with session_id + analysis data
```

---

## Quick Start

### Prerequisites

- Python 3.8+
- [Tesseract OCR](https://github.com/tesseract-ocr/tesseract)
- [Poppler](https://poppler.freedesktop.org/)
- A [Groq API key](https://console.groq.com)

### Setup

```bash
# Clone and navigate
cd Law-Lens

# Install dependencies
pip install -r requirements.txt

# Create .env file
cat > .env << EOF
GROQ_API_KEY=your_groq_api_key
TESSERACT_PATH=C:/path/to/tesseract.exe
POPPLER_BIN_PATH=C:/path/to/poppler/bin
EOF

# Start the API server
python run_api.py
```

The server starts at **http://localhost:8000**. Interactive docs at **http://localhost:8000/docs**.

### Streamlit UI (Alternative)

```bash
streamlit run app.py
```

---

## API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/` | Root health check |
| `GET` | `/health` | Server status + model load state |
| `POST` | `/pdf/upload` | Upload & analyze a PDF document |
| `GET` | `/pdf/session/{session_id}` | Retrieve analysis data for a session |
| `DELETE` | `/pdf/session/{session_id}` | Delete session & cleanup files |
| `GET` | `/pdf/download/highlighted/{session_id}` | Download highlighted PDF |
| `GET` | `/pdf/download/keywords/{session_id}` | Download keywords as `.txt` |
| `GET` | `/pdf/download/text/{session_id}` | Download raw extracted text |
| `POST` | `/chat/` | Ask a question about a document |

---

## Project Structure

```
Law-Lens/
├── main.py                        # FastAPI entry point
├── app.py                         # Streamlit UI entry point
├── run_api.py                     # Server launcher
├── requirements.txt               # Python dependencies
├── config.toml                    # Streamlit file-watcher config
├── .env                           # API keys & system paths
│
├── routes/                        # HTTP layer
│   ├── health.py                  # Health check endpoints
│   ├── pdf.py                     # PDF upload / download / session
│   └── chat.py                    # Chat endpoint
│
├── services/                      # Business logic & orchestration
│   ├── pdf_service.py             # PDF pipeline orchestrator + state
│   ├── chat_service.py            # Chat orchestration
│   └── json_utils.py              # NumPy-safe JSON serialization
│
├── modules/                       # Domain logic (no HTTP awareness)
│   ├── pdf_processor.py           # PDF text extraction & cleaning
│   ├── keyword.py                 # LegalBERT keyword extraction
│   ├── keyword_meaning.py         # Groq-based keyword definitions
│   ├── semantic_importance.py     # Hybrid paragraph scoring
│   ├── vector_store.py            # FAISS index creation & search
│   ├── chatbot.py                 # RAG Q&A pipeline
│   ├── highlight_pdf.py           # PDF color annotation
│   ├── case_law_fetcher.py        # Indian legal section mapper
│   ├── ocr.py                     # Tesseract OCR pipeline
│   └── utils/
│       └── text_cleaner.py        # Keyword normalization
│
└── test_new_api.py                # Integration test suite
```

---

## Design Highlights

- **Resilient fallbacks** — PDF extraction degrades gracefully from fast → slow → OCR; paragraph scoring falls back to keyword heuristics if LLM unavailable; highlighting skips gracefully on failure
- **API cost optimization** — semantic filter reduces LLM calls by ~60-80% by pre-filtering non-legal paragraphs before Groq scoring
- **Robust PDF matching** — token overlap + fuzzy ratio heuristics handle layout variations in legal PDFs
- **Lazy model loading** — heavy ML libraries (PyTorch, Transformers) load on-demand, not at import time
- **NumPy-safe serialization** — recursive converter ensures FAISS/Numpy types don't break JSON responses

---

## Testing

```bash
# Start the server first, then:
python test_new_api.py
```

The integration test validates: health check → PDF upload → chat → downloads → session management.

---

## Future Roadmap

- Persistent storage (PostgreSQL + Redis) for production scalability
- Asynchronous LLM calls for parallel pipeline stages
- WebSocket-based progress streaming
- Docker containerization with multi-stage build
- CI/CD pipeline (GitHub Actions → cloud deploy)
- JWT authentication & rate limiting
- Unit test coverage with mocked LLM responses
