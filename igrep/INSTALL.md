# igrep - Installation & Distribution Guide

## For End Users

### Quick Installation (Recommended)

1. **Download** `igrep-setup.exe` from the releases page
2. **Double-click** the executable
3. Follow the on-screen instructions
4. The installer will:
   - Install Python dependencies
   - Download the ML model (~90-100MB)
   - Create a system command `igrep`
   - Add it to your PATH

> **Note**: First run will take a few minutes while downloading models.

### Installation Methods

#### Method 1: Using the Setup Executable (Easiest)
```
1. Download igrep-setup.exe
2. Double-click to run
3. Follow prompts
```

#### Method 2: Using PowerShell
```powershell
# Right-click PowerShell and select "Run as Administrator"
powershell -NoProfile -ExecutionPolicy Bypass -File "install.ps1"
```

#### Method 3: Using Command Prompt
```batch
install.bat
```

#### Method 4: Manual Setup (For Developers)
```bash
# Requires: Python 3.13+ and uv installed
uv sync                 # Install dependencies
uv run main.py setup    # Download models and initialize DB
```

### System Requirements

- **Windows 7 or later** (Windows 10/11 recommended)
- **Internet connection** (for downloading models on first run)
- **~500MB free disk space** (for models and database)
- **Administrator privileges** (recommended for PATH configuration)

---

## For Developers & Distributors

### Building the Executable

#### Prerequisites
```bash
# Install Python 3.13+
# Install uv (https://github.com/astral-sh/uv)
```

#### Build Steps

1. **Clone the repository**
   ```bash
   git clone https://github.com/yourusername/igrep.git
   cd igrep
   ```

2. **Build the .exe**
   ```bash
   # Option A: Using batch script
   .\build_exe.bat
   
   # Option B: Using PowerShell
   powershell -ExecutionPolicy Bypass -File .\build_exe.ps1
   
   # Option C: Manual build
   pip install pyinstaller
   pyinstaller --onefile --console --name="igrep-setup" setup_gui.py
   ```

3. **Find the executable**
   ```
   dist/igrep-setup.exe
   ```

### Distribution Methods

#### Option 1: GitHub Releases (Recommended)
```bash
# 1. Create a GitHub release
# 2. Upload dist/igrep-setup.exe as an asset
# 3. Users download and run directly
```

#### Option 2: Windows Package Manager (winget)
Create a `winget.yaml` manifest:

```yaml
PackageIdentifier: YourName.igrep
PackageVersion: 0.1.0
PackageName: igrep
Publisher: Your Name
PackageUrl: https://github.com/yourusername/igrep
License: MIT
ShortDescription: Search images and PDFs with OCR and semantic search
Description: A lightweight CLI tool for pattern and semantic search in image and PDF collections

Installers:
  - Architecture: x64
    InstallerType: zip
    InstallerUrl: https://github.com/yourusername/igrep/releases/download/v0.1.0/igrep-setup.exe
    InstallerSha256: <SHA256_HASH>
    InstallerSwitches:
      Silent: /quiet
      SilentWithProgress: /passive
```

#### Option 3: Self-Hosted Web Download
- Host `igrep-setup.exe` on your website
- Users download and run

#### Option 4: Python Package (PyPI)
```bash
# Not recommended for GUI setup, but possible
pip install igrep
```

---

## Command Reference

After installation, use `igrep` from any directory:

### Pattern Search
```bash
# Exact match
igrep "search term"

# Ignore case
igrep -i "search term"

# Count occurrences
igrep -c "search term"
```

### Semantic Search
```bash
# Top 5 results (default)
igrep -s "search description"

# Top 10 results
igrep -s "search description" 10
```

### Track Folders
```bash
# Add a folder to track
igrep --track "C:\path\to\folder"

# Add multiple folders
igrep --track "C:\Documents\Images"
igrep --track "C:\Downloads\PDFs"

# List all tracked folders
igrep --track
```

### Sync Files
```bash
# Index all files in tracked folders
igrep sync
```

### Setup & Help
```bash
# Initialize database and download models
igrep setup

# Show help
igrep --help
```

---

## Troubleshooting

### "igrep command not found"
- Restart your Command Prompt/PowerShell window
- If still not working, the PATH wasn't added properly
- **Solution**: Manually add the installation folder to your PATH:
  1. Win+X → System
  2. Advanced System Settings
  3. Environment Variables
  4. Edit PATH and add the igrep installation directory

### "Failed to install dependencies"
- Check your internet connection
- Run installer as Administrator
- Try manual installation: `uv sync`

### "Tesseract not found"
- Install tesseract-ocr from: https://github.com/UB-Mannheim/tesseract/wiki
- Set `TESSERACT_CMD` environment variable if needed

### Models not downloading
- Check internet connection
- Ensure ~100MB free disk space
- Models are cached in `models/all-MiniLM-L6-v2/`

### Database errors
- Database is stored in `data/database.db`
- Delete it and run `igrep setup` to rebuild

---

## File Structure

```
igrep/
├── install.bat              # Batch installer
├── install.ps1              # PowerShell installer
├── setup_gui.py             # GUI setup (for .exe)
├── build_exe.bat            # Build script (batch)
├── build_exe.ps1            # Build script (PowerShell)
├── main.py                  # CLI entry point
├── paths.txt                # Tracked folder paths (user-editable)
├── pyproject.toml           # Project configuration
├── data/
│   └── database.db          # SQLite database (created on first run)
├── models/                  # ML models (cached)
└── src/
    ├── lib/
    │   ├── db.py            # Database setup
    │   ├── llm.py           # ML model loading
    │   └── path_manager.py   # Folder tracking
    └── service/             # Business logic
```

---

## Advanced Configuration

### Customize Tesseract Path
Edit `main.py` or set environment variable:
```batch
set TESSERACT_CMD=C:\Program Files\Tesseract-OCR\tesseract.exe
```

### Use Different OCR Engine
Edit `src/service/extractor.py` to use different models or engines.

### Disable Semantic Search
Remove sentence-transformers from dependencies if not needed.

---

## License
[Your License Here]

## Support
- GitHub Issues: https://github.com/yourusername/igrep/issues
- Discussions: https://github.com/yourusername/igrep/discussions
