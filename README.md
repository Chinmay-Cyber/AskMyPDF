# 🤖 RAG Chatbot PDF

A production-ready Retrieval-Augmented Generation (RAG) chatbot that lets you have a conversation with your PDF or text documents — right inside a Jupyter notebook. Upload a document, ask questions, and get accurate, cited answers powered by Groq's LLaMA model.

---

## ✨ Features

- **Conversational Q&A** — multi-turn dialogue with memory of previous questions
- **Source citations** — every answer references the exact file and page number it drew from
- **Redundancy filtering** — contextual compression pipeline removes duplicate retrieved chunks before answering
- **Security-first design** — filename sanitization, file size validation, extension allowlisting, and API key masking
- **Interactive Jupyter UI** — fully widget-based interface; no web server required
- **Reset & reload** — swap documents mid-session without restarting the kernel

---

## 🏗️ Architecture

```
┌─────────────┐     ┌───────────────────┐     ┌──────────────────────┐
│  RAGInterface│────▶│    RAGSystem       │────▶│  DocumentProcessor   │
│  (ipywidgets)│     │  (Orchestrator)    │     │  PyPDF / TextLoader  │
└─────────────┘     └───────────────────┘     │  RecursiveTextSplit  │
                             │                 └──────────────────────┘
                             │                 ┌──────────────────────┐
                             ├────────────────▶│  VectorStoreManager  │
                             │                 │  HuggingFace Embeds  │
                             │                 │  FAISS + Compression │
                             │                 └──────────────────────┘
                             │                 ┌──────────────────────┐
                             └────────────────▶│       RAGChain       │
                                               │  Groq LLaMA 3.1      │
                                               │  ConversationMemory  │
                                               └──────────────────────┘
```

**Component breakdown:**

- `RAGConfig` — frozen dataclass holding all tunable parameters (chunk size, model names, retrieval K, etc.)
- `SecurityValidator` — sanitizes filenames, validates file size and extension, masks API keys in logs
- `DocumentProcessor` — loads PDF/TXT files, enriches metadata (source file, page number, chunk index), splits into overlapping chunks
- `VectorStoreManager` — generates embeddings with `sentence-transformers/all-MiniLM-L6-v2`, indexes with FAISS, applies a redundancy filter at retrieval time
- `RAGChain` — wraps Groq's LLaMA 3.1 8B Instant with a custom prompt, conversation buffer memory, and a `ConversationalRetrievalChain`
- `RAGSystem` — orchestrates all components, handles temporary file lifecycle with a context manager
- `RAGInterface` — ipywidgets UI layer for the full notebook experience

---

## 🚀 Quick Start

### 1. Clone the repository

```bash
git clone https://github.com/your-username/rag-chatbot-pdf.git
cd rag-chatbot-pdf
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Get a Groq API key

Sign up at [console.groq.com](https://console.groq.com) and create a free API key.

### 4. Launch the notebook

```bash
jupyter notebook rag_chatbot_pdf.ipynb
```

Run all cells. The interactive UI will appear at the bottom of the notebook.

### 5. Use the chatbot

1. Paste your **Groq API key** into the password field
2. **Upload** a PDF or `.txt` file (up to 50 MB)
3. Click **🚀 Build Pipeline** and wait for the success message
4. Type your question and hit **➤ Ask** (or press Enter)
5. Click **🔄 Reset** to start over with a new document

---

## 📦 Requirements

| Package | Purpose |
|---|---|
| `langchain` | RAG chain, memory, retrieval |
| `langchain-community` | Document loaders, FAISS, embeddings, transformers |
| `langchain-groq` | Groq LLM integration |
| `sentence-transformers` | `all-MiniLM-L6-v2` embedding model |
| `faiss-cpu` | Vector similarity search |
| `pypdf` | PDF parsing |
| `ipywidgets` | Notebook UI |
| `jupyter` | Notebook runtime |

Install everything at once:

```bash
pip install langchain langchain-community langchain-groq sentence-transformers faiss-cpu pypdf ipywidgets jupyter
```

---

## ⚙️ Configuration

All parameters live in the `RAGConfig` dataclass at the top of the notebook. Edit them before running to customise behaviour:

| Parameter | Default | Description |
|---|---|---|
| `chunk_size` | `1000` | Characters per text chunk |
| `chunk_overlap` | `150` | Overlap between adjacent chunks |
| `embedding_model` | `all-MiniLM-L6-v2` | HuggingFace sentence embedding model |
| `llm_model` | `llama-3.1-8b-instant` | Groq model identifier |
| `temperature` | `0.2` | LLM creativity (lower = more factual) |
| `top_k_retrieval` | `10` | Chunks fetched before compression |
| `top_k_compression` | `5` | Chunks passed to the LLM after filtering |
| `max_tokens` | `4096` | Maximum tokens in LLM response |
| `max_file_size_mb` | `50` | Upload size limit |

---

## 🔒 Security Notes

- API keys are **never logged** in plaintext — only a masked version (`gsk_...1234`) appears in logs
- Uploaded filenames are **sanitized** and given a short SHA-256 hash prefix to prevent path traversal and collision attacks
- Files are written to a **temporary directory** and deleted automatically after processing, regardless of success or failure
- Only `.pdf` and `.txt` extensions are accepted

---

## 📁 Project Structure

```
rag-chatbot-pdf/
├── rag_chatbot_pdf.ipynb   # Main notebook (all code + UI)
└── README.md
```

---

## 🛠️ How It Works

1. **Ingestion** — the uploaded file is validated, written to a temp path, and loaded page-by-page
2. **Chunking** — text is split recursively on paragraph, sentence, and word boundaries with overlap to preserve context
3. **Embedding** — each chunk is converted to a 384-dimension vector using MiniLM
4. **Indexing** — vectors are stored in an in-memory FAISS index
5. **Retrieval** — at query time the top-K most similar chunks are fetched, then a redundancy filter removes near-duplicates
6. **Generation** — the compressed context plus conversation history is fed to LLaMA 3.1 via Groq's API, which returns an answer with inline source citations
7. **Memory** — `ConversationBufferMemory` keeps the full dialogue history so follow-up questions are handled naturally

---

## 🙌 Acknowledgements

- [LangChain](https://www.langchain.com/) for the RAG framework
- [Groq](https://groq.com/) for fast LLaMA inference
- [HuggingFace](https://huggingface.co/) for the `sentence-transformers` embedding models
- [Meta AI](https://ai.meta.com/) for the LLaMA 3.1 model family
