# ⚡ Flash — AI-Powered Document Q&A & Flashcard Generator

Flash is a locally-running AI web application that lets you upload documents (PDF, DOCX, PPT) and instantly:
- **Chat with your document** using a RAG-powered chatbot
- **Generate flashcards** from your document's content

Everything runs 100% locally using [Ollama](https://ollama.com) — no API keys, no cloud, no data leaving your machine.

---

## ✨ Features

| Feature | Description |
|---|---|
| 📄 Document Upload | Upload PDF, DOCX, PPTX files (up to 32 MB) |
| 🔍 RAG Chatbot | Ask questions; answers are grounded strictly in your document |
| 🃏 Flashcard Generator | Auto-generate study flashcards from document content |
| 🔒 100% Local | All AI inference runs locally via Ollama |
| ⚡ OCR Support | Optionally extract text from scanned/image-based PDFs |

---

## 🏗️ Architecture

```
User Browser
     │
     ▼
Flask Web App (app.py)
     │
     ├── POST /api/upload     → Extract text → Chunk → Embed → ChromaDB
     ├── POST /api/chat       → Embed query → Retrieve chunks → Ollama LLM → Answer
     └── POST /api/generate   → Pass context → Ollama LLM → Flashcards
                                        │
                              ┌─────────┴─────────┐
                              │                   │
                         ChromaDB            Ollama (local)
                      (vector store)    nomic-embed-text + gemma2:2b
```

---

## 🛠️ Requirements

- Python 3.10+
- [Ollama](https://ollama.com/download) installed and running

---

## 🚀 Setup & Installation

### 1. Clone the repository

```bash
git clone <your-repo-url>
cd flash
```

### 2. Install Python dependencies

```bash
pip install -r requirements.txt
```

> **Note:** This project requires `numpy<2` due to a compatibility issue with `chromadb`. If you see a `np.float_` error, run:
> ```bash
> pip install "numpy<2"
> ```

### 3. Install and start Ollama

Download Ollama from [https://ollama.com/download](https://ollama.com/download), then pull the required models:

```bash
# Chat/generation model
ollama pull gemma2:2b

# Embedding model (required for RAG chatbot)
ollama pull nomic-embed-text
```

### 4. Configure environment variables

Copy the example env file and edit if needed:

```bash
cp .env.example .env
```

`.env` defaults:

```env
OLLAMA_BASE_URL=http://127.0.0.1:11434
OLLAMA_MODEL=gemma2:2b
```

> ⚠️ **macOS users:** Use `127.0.0.1` instead of `localhost` — on macOS, `localhost` resolves to IPv6 (`::1`) which Ollama does not listen on, causing 404 errors.

### 5. Run the app

```bash
python3 app.py
```

The app will be available at **http://127.0.0.1:5005**

---

## 📁 Project Structure

```
flash/
├── app.py                    # Flask app entry point
├── config.py                 # Environment config (Ollama URLs, paths)
├── requirements.txt
├── .env                      # Local environment variables
│
├── routes/
│   ├── upload.py             # POST /api/upload — file ingestion & indexing
│   ├── chatbot.py            # POST /api/chat — RAG question answering
│   └── flashcards.py         # POST /api/generate — flashcard generation
│
├── utils/
│   ├── rag_pipeline.py       # Core RAG logic (index + query)
│   ├── vector_store.py       # ChromaDB wrapper (store & retrieve embeddings)
│   ├── embeddings.py         # Text → vector via nomic-embed-text
│   ├── file_parser.py        # Text extraction from PDF/DOCX/PPTX
│   ├── chunker.py            # Split text into overlapping chunks
│   └── flashcard_generator.py # LLM-based flashcard creation
│
├── static/
│   ├── css/style.css
│   └── js/app.js
│
├── templates/
│   └── index.html
│
├── chroma_db/                # Persistent ChromaDB vector store (auto-created)
└── uploads/                  # Temporary file storage (auto-created, auto-cleaned)
```

---

## 🔌 API Reference

### `POST /api/upload`
Upload and index a document.

**Form Data:**
| Field | Type | Description |
|---|---|---|
| `file` | File | PDF, DOCX, or PPTX file |
| `ocr_mode` | `"true"/"false"` | Enable OCR for scanned documents |

**Response:**
```json
{
  "message": "File processed successfully.",
  "filename": "example.pdf",
  "chunk_count": 42,
  "context": "..."
}
```

---

### `POST /api/chat`
Ask a question about the uploaded document.

**Request body:**
```json
{ "question": "What is the main topic of the document?" }
```

**Response:**
```json
{
  "answer": "The document discusses...",
  "context": ["chunk 1...", "chunk 2...", "..."]
}
```

---

### `POST /api/generate`
Generate flashcards from document context.

**Request body:**
```json
{ "context": "<document text>" }
```

**Response:**
```json
{
  "flashcards": [
    { "question": "...", "answer": "..." }
  ]
}
```

---

## ⚙️ Configuration

All settings can be overridden via `.env`:

| Variable | Default | Description |
|---|---|---|
| `OLLAMA_BASE_URL` | `http://127.0.0.1:11434` | Ollama server URL |
| `OLLAMA_MODEL` | `gemma2:2b` | Default Ollama model |
| `OLLAMA_CHAT_MODEL` | `gemma2:2b` | Model used for chat/Q&A |
| `OLLAMA_EMBED_MODEL` | `nomic-embed-text` | Model used for embeddings |

---

## 🐛 Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `ModuleNotFoundError: flask_cors` | Dependencies not installed | Run `pip install -r requirements.txt` |
| `np.float_ was removed in NumPy 2.0` | NumPy version too new for chromadb | Run `pip install "numpy<2"` |
| `Failed to generate embedding` | `nomic-embed-text` not pulled | Run `ollama pull nomic-embed-text` |
| `404 Not Found` on Ollama calls | Using `localhost` on macOS (IPv6) | Set `OLLAMA_BASE_URL=http://127.0.0.1:11434` in `.env` |
| `Connection refused` on Ollama | Ollama not running | Start Ollama app or run `ollama serve` |

---

## 📄 License

MIT
