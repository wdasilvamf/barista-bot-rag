# ☕ Barista Bot RAG — V1.0

A coffee-focused Q&A assistant built on a **RAG (Retrieval-Augmented Generation)** architecture that answers **exclusively** based on the content of a reference book.

This project was designed as a portfolio piece: the notebook (`notebooks/barista_bot_v1.ipynb`) is didactic, commented cell by cell, and runs end-to-end on Google Colab.

## 🎯 The problem and the critical data detail

The single source of truth for this project is a **scanned PDF** (image, not searchable text). This means traditional text-extraction libraries (`PyPDF2`, `pdfplumber`, etc.) **do not work** — they rely on a text layer inside the PDF that simply doesn't exist here.

Because of that, this V1.0's ingestion pipeline is built around **OCR (Optical Character Recognition)**:

1. Each PDF page is converted into an image (`pdf2image`).
2. Each image goes through **Tesseract OCR** (`pytesseract`), configured with the `por` (Portuguese) language pack.
3. The extracted text is annotated with a **page number** metadata field, enabling traceability: the end user always knows exactly which page of the book an answer came from.

## 🧱 Architecture (V1.0)

```
Scanned PDF (/data)
   → pdf2image (pages → images)
   → pytesseract + tesseract-ocr-por (images → text)
   → RecursiveCharacterTextSplitter (chunking)
   → HuggingFace Embeddings (BAAI/bge-small-en-v1.5)
   → FAISS (vector store)
   → Retriever
   → Restrictive Prompt Template ("answer only from the context")
   → Ollama (llama3, running locally in Colab)
   → Answer + source page(s)
```

## 📦 Stack

| Layer               | Tool                                   |
|---------------------|------------------------------------------|
| OCR                 | Tesseract OCR + `pytesseract`             |
| PDF conversion      | `pdf2image` (poppler)                     |
| RAG orchestration   | LangChain                                 |
| Embeddings          | HuggingFace `BAAI/bge-small-en-v1.5`      |
| Vector Store        | FAISS (`faiss-cpu`)                       |
| LLM                 | Ollama (`llama3`), local to the environment |

## 📁 Folder structure

```
RAG - coffee/
├── README.md
├── data/                      # Put the scanned PDF of the book here
└── notebooks/
    └── barista_bot_v1.ipynb   # Main notebook (Colab-ready)
```

## ▶️ How to run

1. Open `notebooks/barista_bot_v1.ipynb` in Google Colab.
2. Upload the scanned PDF of the book to the `/data` folder (inside the Colab environment).
3. Run the cells in order — the first cell installs all system dependencies (Tesseract + `por` language pack) and Python packages.
4. The Ollama setup cell starts the local server in the background and downloads the `llama3` model.
5. Ask questions about the book's content in the final interaction cell.

> ⚠️ Since OCR-ing an entire book can be slow, the notebook includes a **caching** step for the extracted text (saved under `/data`), avoiding reprocessing the PDF on every run.

## 🔮 Roadmap — towards V2.0 (agentic)

This V1.0 is intentionally a **linear, deterministic** RAG pipeline (retrieval → prompt → generation). It establishes the reliable data foundation (OCR ingestion with page-level metadata) on top of which the next version will evolve.

Ideas already mapped for V2.0:
- Turn the bot into an **agent** with tool use, able to decide *when* to consult the book vs. when to ask the user for clarification.
- Add a **re-ranking** tool for retrieved chunks before assembling the final context.
- Multi-turn conversation memory with incremental summarization.
- Automated evaluation (RAGAS or similar) of answer faithfulness to the source text.

## 📌 Status

- [x] Scanned PDF ingestion via OCR with page-level traceability
- [x] Full RAG pipeline (FAISS + embeddings + Ollama/llama3)
- [x] Restrictive prompt (no knowledge outside the book)
- [ ] Agentification (V2.0)
