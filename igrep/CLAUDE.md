# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
uv sync

# One-time setup: init SQLite DB + download embedding model (~90MB)
uv run main.py setup

# Index images/PDFs from paths listed in paths.txt
uv run main.py sync

# Pattern search (exact, ignore-case, count)
uv run main.py "query"
uv run main.py -i "query"
uv run main.py -c "query"

# Semantic search (default top 5, optional count arg)
uv run main.py -s "query"
uv run main.py -s "query" 10
```

No linting tools or automated tests are configured. `test.py` is an empty placeholder.

## Architecture

**igrep** is a CLI tool for searching images and PDFs using OCR-extracted text. It supports keyword (FTS5) and semantic (embedding) search over a local SQLite database.

### Layers

```
main.py                   # argparse CLI entry point
src/connector/            # Orchestration: ties together lib + service per command
src/service/              # Business logic: OCR, text extraction, DB writes, search, output
src/lib/                  # Infrastructure: DB engine, ORM session, model loading
src/models/Image.py       # SQLAlchemy ORM model + FTS5 sync event hooks
```

### Indexing pipeline (`sync`)

1. Read directories from `paths.txt`
2. Walk for `.png/.jpg/.jpeg/.pdf` files, skip already-indexed paths
3. **PDFs**: extract text via PyMuPDF (`fitz`) — no OCR needed
4. **Images**: OCR via `pytesseract` using `ProcessPoolExecutor` (up to 14 workers)
5. Batch-encode extracted text with `all-MiniLM-L6-v2` sentence-transformer
6. Bulk INSERT into `images` table; backfill `images_fts` FTS5 virtual table

### Database schema

- **`images`**: `id`, `image_loc` (relative path), `words` (OCR text), `embeddings` (JSON float32 array), `created_at`
- **`images_fts`**: FTS5 virtual table mirroring `words`; kept in sync via SQLAlchemy ORM event hooks in `src/models/Image.py`

### Search modes

- **Pattern search** (`src/service/search_pattern.py`): FTS5 `MATCH` query with BM25 ranking + `snippet()`; results filtered by `src/service/filters.py` to extract matching line context
- **Semantic search** (`src/lib/llm.py`): loads all embeddings from DB, encodes query, computes cosine similarity via numpy, returns top-k

### Key files

| File | Purpose |
|---|---|
| `paths.txt` | Directories to scan (one per line) |
| `data/database.db` | SQLite DB (gitignored) |
| `models/all-MiniLM-L6-v2/` | Cached sentence-transformer model (gitignored) |
| `src/service/extractor.py` | pytesseract (images) + fitz (PDFs) extraction |
| `src/service/save_image.py` | Multi-process OCR + batch embedding + bulk insert |
| `src/lib/llm.py` | Model loading, encode, cosine similarity |
| `src/lib/db.py` | SQLAlchemy engine + FTS5 table setup |

# Linux
curl -LsSf https://raw.githubusercontent.com/YOU/igrep/main/scripts/setup_script.sh | bash

# Windows
irm https://raw.githubusercontent.com/YOU/igrep/main/scripts/install.ps1 | iex

# Or via pip (once on PyPI)
pip install igrep
igrep setup

