# CLAUDE.md

## Project Overview

**rag-pipeline** is an end-to-end document extraction and RAG (Retrieval-Augmented Generation) pipeline for querying large documents using semantic and hybrid retrieval with reranking. The entire implementation lives in a single Jupyter notebook designed for Google Colab.

## Repository Structure

```
rag-pipeline/
├── .gitignore          # Python project gitignore
├── LICENSE             # MIT License (2025 rexfnguyen)
├── README.md           # Project description
├── CLAUDE.md           # This file
└── notebooks/
    └── rag_pipeline.ipynb   # Complete pipeline implementation
```

## Architecture

The pipeline follows a sequential processing flow:

```
PDF Upload → Text Extraction → Document Loading → Vector Indexing →
Hybrid Retrieval (Vector + BM25) → Reranking → LLM Response
```

### Key Components (in `notebooks/rag_pipeline.ipynb`)

| Cell | Function | Purpose |
|------|----------|---------|
| 1 | pip installs | Install all dependencies at runtime |
| 2 | `upload_pdf()` | Upload PDF via Colab UI, save to `sample_docs/` |
| 3 | (invocation) | Calls `upload_pdf()` |
| 4 | `extract_text_from_pdf()` | Extract text from PDF using PyMuPDF (fitz) |
| 5 | (invocation) | Runs extraction and prints preview |
| 6 | Config | Sets up `nest_asyncio`, Google API key for Gemini |
| 7 | Model init | Initializes Gemini LLM + HuggingFace embeddings |
| 8 | `load_pdf_with_pymupdf()` | Wraps `SimpleDirectoryReader` for PDF loading |
| 9 | `process_and_index_pdf()` | Creates `VectorStoreIndex` from documents |
| 10 | `build_rag_pipeline()` | Builds hybrid retriever + reranker + query engine |
| 11 | (invocation) | Runs the full pipeline and prints response |

### Core RAG Design

- **Hybrid Retriever** (`HybridRetriever` class, extends `BaseRetriever`): Combines vector retrieval (semantic similarity) with BM25 retrieval (keyword matching). Results are deduplicated by `node_id` and sorted by score.
- **Reranker**: Cross-encoder model (`cross-encoder/ms-marco-MiniLM-L-6-v2`) via `SentenceTransformerRerank`. Only enabled when more than 1 node exists.
- **Query Engine**: `RetrieverQueryEngine` combining the hybrid retriever, reranker, and Gemini LLM.
- **Adaptive top_k**: `safe_top_k = min(2, max(1, num_nodes))` prevents errors when few documents exist.

## Dependencies

All dependencies are installed at runtime via `pip install` in the notebook (no `requirements.txt` or `pyproject.toml`):

| Package | Purpose |
|---------|---------|
| `llama-index` | Core RAG framework |
| `llama-index-llms-gemini` | Google Gemini LLM integration |
| `llama-index-embeddings-huggingface` | HuggingFace embedding models |
| `llama-index-retrievers-bm25` | BM25 keyword-based retrieval |
| `pymupdf` (imported as `fitz`) | PDF text extraction |
| `nest_asyncio` | Async event loop patching for notebooks |

**Note**: No versions are pinned. All packages install from latest PyPI.

## Models Used

- **LLM**: Google Gemini (`models/gemini-2.5-flash`)
- **Embeddings**: `sentence-transformers/all-MiniLM-L6-v2` (lightweight, 22M params)
- **Reranker**: `cross-encoder/ms-marco-MiniLM-L-6-v2`

## Environment Requirements

- **Platform**: Google Colab (uses `google.colab.files` for upload)
- **Python**: 3.x (Colab default)
- **API Key**: `GOOGLE_API_KEY` environment variable must be set with a valid Google Gemini API key (currently a placeholder `" "` in cell 6)

## Code Conventions

- **Functions**: `snake_case` (e.g., `upload_pdf`, `extract_text_from_pdf`)
- **Classes**: `PascalCase` (e.g., `HybridRetriever`)
- **Constants**: `UPPER_CASE` (e.g., `GOOGLE_API_KEY`)
- **Docstrings**: Google-style, single-line (`"""Extract text from a PDF file using PyMuPDF."""`)
- **Organization**: One logical step per notebook cell

## Development Workflow

1. Open the notebook in Google Colab (badge link in first cell)
2. Run cell 1 to install dependencies
3. Upload a PDF document via cell 2-3
4. Set your `GOOGLE_API_KEY` in cell 6
5. Run remaining cells sequentially to build and query the pipeline

There are no build scripts, CI/CD pipelines, linters, formatters, or test suites configured.

## Git Conventions

- **Default branch**: `master`
- **Commit style**: Short imperative descriptions (e.g., "Add RAG pipeline notebook")

## Known Limitations

- Colab-only: relies on `google.colab` for file upload
- No persistent storage between Colab sessions
- No error handling beyond basic PDF extension validation
- API key stored as plaintext in notebook
- Unpinned dependency versions may cause reproducibility issues
- No tests or CI/CD
