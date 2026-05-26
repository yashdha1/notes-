# igrep User Guide

**i-grep** is a powerful CLI tool for searching images and PDFs using both keyword pattern matching and semantic (AI-powered) search. Extract text from images via OCR and PDFs via text extraction, then search efficiently using a local SQLite database.

---

## Installation

### Option 1: Windows PowerShell (Recommended)
```powershell
irm https://raw.githubusercontent.com/YOU/igrep/main/scripts/install.ps1 | iex
```

### Option 2: Linux/macOS Bash
```bash
curl -LsSf https://raw.githubusercontent.com/YOU/igrep/main/scripts/setup_script.sh | bash
```

### Option 3: pip (Once on PyPI)
```bash
pip install igrep
igrep setup
```

### Manual Installation (Development)
```bash
git clone https://github.com/YOU/igrep.git
cd igrep
uv sync
uv run main.py setup
```

---

## Quick Start (3 Steps)

### 1. Initialize Database & Download Model
This is a one-time setup (~90MB download):
```bash
igrep setup
```
or
```bash
uv run main.py setup
```

**What happens:**
- Creates SQLite database in user data directory
- Downloads sentence-transformer model (`all-MiniLM-L6-v2`) to cache

### 2. Add Folders to Scan
Add folders containing images and PDFs to search:
```bash
igrep --track "C:\Users\YourName\Pictures"
igrep --track "C:\Users\YourName\Documents"
```

View tracked folders:
```bash
igrep --track
```

**Supported formats:** `.png`, `.jpg`, `.jpeg`, `.pdf`

### 3. Index Your Media
Scan tracked folders and extract text:
```bash
igrep sync
```
or
```bash
uv run main.py sync
```

**What happens:**
- Walks through tracked folders
- Extracts text from images via OCR (parallel processing)
- Extracts text from PDFs
- Encodes text into embeddings for semantic search
- Stores everything in local database (no cloud uploads)

---

## Search Commands

### Pattern Search (Exact Match)
Find files with exact text matches:
```bash
igrep "invoice"
igrep "John Doe"
```

**Options:**
- `-i` → Ignore case: `igrep -i "INVOICE"`
- `-c` → Count occurrences: `igrep -c "error"`
- Add limit: `igrep "query" 10` (show 10 results instead of default 5)

**How it works:**
- Uses FTS5 (Full-Text Search) with BM25 ranking
- Shows matching line context
- Fast and accurate for keywords

### Semantic Search (AI-Powered)
Find relevant documents even if exact words don't match:
```bash
igrep -s "financial statement"
igrep -s "business proposal" 10
```

**How it works:**
- Converts query and all indexed documents to embeddings
- Computes semantic similarity using cosine distance
- Returns most relevant results
- Slower than pattern search, but finds conceptually similar content

### Examples
```bash
# Find all mentions of "meeting"
igrep "meeting"

# Find "Meeting" ignoring case, show top 15 results
igrep -i "Meeting" 15

# Count how many times "email" appears across all images/PDFs
igrep -c "email"

# Semantic search: find documents about "recruitment"
igrep -s "recruitment"

# Semantic search: find top 20 documents about "salary"
igrep -s "salary" 20
```

---

## Output Format

### Pattern Search Results
```
Found 3 image(s) with 3 references.
  1 : C:\Users\YourName\Pictures\scan_001.pdf | **invoice** 2024-01-15  file:///C:/Users/YourName/Pictures/scan_001.pdf
  2 : C:\Users\YourName\Pictures\document.png | Total **invoice** amount: $500  file:///C:/Users/YourName/Pictures/document.png
  3 : C:\Documents\receipt.jpg | Invoice number #12345 matches **invoice** criteria  file:///C:/Documents/receipt.jpg
```

- Matched text is **highlighted in bold**
- File URI opens directly in your file explorer/viewer
- Shows the context line where match was found

### Semantic Search Results
```
Found 5 image(s).
  1. [0.845] C:\Users\YourName\Pictures\financial_report.pdf  file:///C:/Users/YourName/Pictures/financial_report.pdf
  2. [0.732] C:\Users\YourName\Pictures\budget_2024.png  file:///C:/Users/YourName/Pictures/budget_2024.png
  3. [0.698] C:\Documents\quarterly_review.pdf  file:///C:/Documents/quarterly_review.pdf
  4. [0.654] C:\Users\YourName\Pictures\expense_summary.jpg  file:///C:/Users/YourName/Pictures/expense_summary.jpg
  5. [0.612] C:\Users\YourName\Pictures\notes.png  file:///C:/Users/YourName/Pictures/notes.png
```

- Score in `[0.0 - 1.0]` indicates relevance (higher = more similar)
- Click file URI to open directly

---

## Architecture

```
User Query
    ↓
+----────────────────────────────────────+
│   Pattern Search (FTS5 + BM25)         │  ← Fast, exact matches
│   OR Semantic Search (Embeddings)      │  ← Slow, conceptual matches
+────────────────────────────────────────+
    ↓
SQLite Database
    ├─ images table: OCR text + embeddings
    └─ images_fts: FTS5 index for keyword search
    ↓
Multi-Process OCR Pipeline
    ├─ Images: pytesseract (14 parallel workers)
    └─ PDFs: PyMuPDF (fitz)
```

---

## Frequently Asked Questions

**Q: Do my files get uploaded anywhere?**
A: No. Everything runs locally on your machine. Database and models are stored in:
- Windows: `%APPDATA%\igrep` and `%LOCALAPPDATA%\igrep\Cache`
- Linux: `~/.local/share/igrep` and `~/.cache/igrep`
- macOS: `~/Library/Application Support/igrep` and `~/Library/Caches/igrep`

**Q: Why is the first semantic search slow?**
A: The embedding model (~90MB) is loaded into memory on first use. Subsequent searches are faster.

**Q: How do I update the index?**
A: Run `igrep sync` again. It skips already-indexed files and only processes new media.

**Q: Can I use igrep for videos?**
A: Currently supports images (PNG, JPG, JPEG) and PDFs only.

**Q: What if OCR is wrong?**
A: Use semantic search to find conceptually similar documents, which is more tolerant of OCR errors.

**Q: How much disk space does the database use?**
A: Depends on how much media you index. Each image/PDF takes ~1-5KB in the database (embeddings + metadata).

---

## Troubleshooting

### "pytesseract not found" error
**Windows:** Download [Tesseract-OCR installer](https://github.com/UB-Mannheim/tesseract/wiki)
**Linux:** `sudo apt-get install tesseract-ocr`
**macOS:** `brew install tesseract`

### "No image path configured" when running `sync`
Add folders first:
```bash
igrep --track "C:\your\folder\path"
```

### Search returns no results
- Check that `igrep sync` completed successfully
- Try semantic search (`-s` flag) if pattern search finds nothing
- Verify files were actually indexed: check database is > 1KB in size

### Model download fails
Ensure you have internet access and sufficient disk space (~100MB).

---

## Advanced Usage

### Using from Python Code
```python
from src.connector.pattern import search_e, search_i
from src.connector.semantic import semantic_search

# Exact pattern search
search_e("invoice")

# Case-insensitive pattern search  
search_i("invoice")

# Semantic search
semantic_search("financial documents", top_k=10)
```

### Custom Configuration
Edit tracked paths in: `%APPDATA%\igrep\paths.txt` (Windows) or `~/.local/share/igrep/paths.txt` (Linux/macOS)

One folder per line:
```
C:\Users\YourName\Pictures
C:\Users\YourName\Documents
/home/user/scans
```

---

## Performance Tips

1. **Batch Indexing:** Run `igrep sync` on a large folder overnight
2. **Pattern Search First:** Use exact search for known keywords (faster)
3. **Limit Results:** Add a number to reduce output: `igrep query 5`
4. **Semantic Search Limit:** Keep top-k under 20 for best relevance

---

## Version & Support

- **Current Version:** 0.1.0 (Alpha)
- **License:** MIT
- **GitHub:** https://github.com/YOU/igrep
- **Python:** >= 3.13

Report issues or request features on GitHub!
