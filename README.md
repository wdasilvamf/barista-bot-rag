# ☕ Barista Bot RAG — V1.0

Um assistente de perguntas e respostas (Q&A) especializado em café, construído sobre uma arquitetura **RAG (Retrieval-Augmented Generation)** que responde **exclusivamente** com base no conteúdo de um livro de referência.

Este projeto foi pensado como peça de portfólio: o notebook (`notebooks/barista_bot_v1.ipynb`) é didático, comentado célula a célula, e roda de ponta a ponta no Google Colab.

## 🎯 O problema e o detalhe crítico dos dados

A única fonte de verdade deste projeto é um **PDF escaneado** (imagem, não texto pesquisável). Isso significa que bibliotecas tradicionais de extração de texto (`PyPDF2`, `pdfplumber`, etc.) **não funcionam** — elas dependem de uma camada de texto no PDF que simplesmente não existe aqui.

Por isso, o pipeline de ingestão desta V1.0 é construído em torno de **OCR (Optical Character Recognition)**:

1. Cada página do PDF é convertida em uma imagem (`pdf2image`).
2. Cada imagem passa pelo **Tesseract OCR** (`pytesseract`), configurado com o pacote de idioma `por` (português).
3. O texto extraído é anotado com metadado de **número da página**, permitindo rastreabilidade: o usuário final sempre sabe de qual página do livro a resposta foi extraída.

## 🧱 Arquitetura (V1.0)

```
PDF escaneado (/data)
   → pdf2image (páginas → imagens)
   → pytesseract + tesseract-ocr-por (imagens → texto)
   → RecursiveCharacterTextSplitter (chunking)
   → HuggingFace Embeddings (BAAI/bge-small-en-v1.5)
   → FAISS (vector store)
   → Retriever
   → Prompt Template restritivo ("responda só com base no contexto")
   → Ollama (llama3, rodando localmente/local ao Colab)
   → Resposta + página(s) de origem
```

## 📦 Stack utilizada

| Camada              | Ferramenta                          |
|---------------------|--------------------------------------|
| OCR                 | Tesseract OCR + `pytesseract`         |
| Conversão de PDF    | `pdf2image` (poppler)                 |
| Orquestração RAG    | LangChain                             |
| Embeddings          | HuggingFace `BAAI/bge-small-en-v1.5`  |
| Vector Store        | FAISS (`faiss-cpu`)                   |
| LLM                 | Ollama (`llama3`), local ao ambiente  |

## 📁 Estrutura de pastas

```
RAG - coffee/
├── README.md
├── data/                      # Coloque aqui o PDF escaneado do livro
└── notebooks/
    └── barista_bot_v1.ipynb   # Notebook principal (Colab-ready)
```

## ▶️ Como rodar

1. Abra `notebooks/barista_bot_v1.ipynb` no Google Colab.
2. Faça upload do PDF escaneado do livro para a pasta `/data` (dentro do ambiente do Colab).
3. Execute as células em ordem — a primeira célula instala todas as dependências de sistema (Tesseract + idioma `por`) e Python.
4. A célula de setup do Ollama sobe o servidor local em background e baixa o modelo `llama3`.
5. Faça perguntas sobre o conteúdo do livro na célula final de interação.

> ⚠️ Como o OCR de um livro inteiro pode ser lento, o notebook inclui um passo de **cache** do texto extraído (salvo em `/data`), evitando reprocessar o PDF a cada execução.

## 🔮 Roadmap — rumo à V2.0 (agêntica)

Esta V1.0 é intencionalmente um pipeline RAG **linear e determinístico** (retrieval → prompt → geração). Ele estabelece a base de dados confiável (ingestão via OCR com metadados de página) sobre a qual a próxima versão vai evoluir.

Ideias já mapeadas para a V2.0:
- Transformar o bot em um **agente** com ferramentas (tool use), capaz de decidir *quando* consultar o livro vs. quando pedir esclarecimento ao usuário.
- Adicionar uma ferramenta de **re-ranking** dos chunks recuperados antes de montar o contexto final.
- Memória de conversação (multi-turn) com resumo incremental.
- Avaliação automatizada (RAGAS ou similar) da fidelidade das respostas ao texto-fonte.

## 📌 Status

- [x] Ingestão de PDF escaneado via OCR com rastreabilidade de página
- [x] Pipeline RAG completo (FAISS + embeddings + Ollama/llama3)
- [x] Prompt restritivo (sem conhecimento externo ao livro)
- [ ] Agentificação (V2.0)
